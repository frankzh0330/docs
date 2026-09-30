# Source: https://github.com/frankzh0330/query-agent/blob/master/service/llm_extractions.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# llm\_extractions.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[update docs and removed unused codes](https://github.com/frankzh0330/query-agent/commit/28cd5d550776847f844dead59a860a776081b9b4)

May 1, 2026

[28cd5d5](https://github.com/frankzh0330/query-agent/commit/28cd5d550776847f844dead59a860a776081b9b4) · May 1, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/service/llm_extractions.py)

Open commit details

History

452 lines (356 loc) · 16 KB

## FilesExpand file tree

master

/

# llm\_extractions.py

Copy path

Top

## File metadata and controls

- Code

- Blame

452 lines (356 loc) · 16 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/llm_extractions.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import asyncio import hashlib import json import logging import os from typing import Any, Dict, List, Optional from dotenv import load\_dotenv from openai import OpenAI from pydantic import BaseModel, Field, ValidationError load\_dotenv() logger = logging.getLogger(\_\_name\_\_) # ==================== 缓存机制 ==================== \_query\_cache: Dict\[str, dict\] = {} def \_query\_hash(text: str) -> str: """生成查询的哈希值用于缓存""" return hashlib.md5(text.encode()).hexdigest() def \_is\_cache\_enabled() -> bool: """检查是否启用缓存""" return os.getenv("ENABLE\_LLM\_CACHE", "true").lower() == "true" # ==================== 会话上下文 ==================== def \_build\_context\_str(session\_context: Optional\[Dict\[str, Any\]\]) -> str: """构建动态会话上下文字符串（注入到提示词） 只保留有价值的动态上下文： 1. QueryState — 上下文继承（最高价值） 2. 最近用户消息 — 辅助理解 不再注入 favorite\_regions/metrics/events（LLM + Catalog 已够用） 记忆文件纠正内容在 \_build\_messages 中单独注入。 """ if not session\_context: return "" parts = \[\] # QueryState — 最高价值 if session\_context.get("last\_query\_state"): qs = session\_context\["last\_query\_state"\] state\_parts = \[\] if qs.get("region\_filter"): state\_parts.append(f"地区={','.join(qs\['region\_filter'\])}") if qs.get("metric"): state\_parts.append(f"指标={qs\['metric'\]}") if qs.get("event"): state\_parts.append(f"事件={qs\['event'\]}") if qs.get("time\_range"): tr = qs\["time\_range"\] state\_parts.append(f"时间=近{tr.get('n', '?')}天") if qs.get("group\_by"): state\_parts.append(f"分组={','.join(qs\['group\_by'\])}") if qs.get("filters"): filter\_strs = \[f"{f\['field'\]}{f\['op'\]}{f\['value'\]}" for f in qs\["filters"\]\] state\_parts.append(f"过滤={','.join(filter\_strs)}") if state\_parts: parts.append(f"上次查询: {', '.join(state\_parts)}") # 最近用户消息 — 辅助理解 recent = session\_context.get("recent\_queries", \[\]) if recent: history\_lines = \[f"Q{i}: {q}" for i, q in enumerate(recent\[-3:\], 1)\] parts.append("=== 最近对话 ===\\n" + "\\n".join(history\_lines) + "\\n=== 历史结束 ===") return "\\n".join(parts) if parts else "" def \_is\_followup\_turn(session\_context: Optional\[Dict\[str, Any\]\]) -> bool: """是否处于更可能是 follow-up 的轮次。""" if not session\_context: return False return bool(session\_context.get("last\_query\_state")) and bool(session\_context.get("turn\_index", 0)) def \_build\_followup\_instruction(session\_context: Optional\[Dict\[str, Any\]\]) -> str: """为多轮 follow-up 场景追加更偏 patch extraction 的指令。""" if not \_is\_followup\_turn(session\_context): return "" return ( "\\n\\n=== 多轮查询补充规则 ===\\n" "如果当前问题看起来是在补充、修改或缩写上一轮查询，请优先抽取本轮明确提到的新信息。\\n" "不要为了补全而重复输出用户本轮没有明确说出的 event、metric、group\_by。\\n" "如果本轮只说了时间、地区、指标或分组变化，就只抽取这些变化。\\n" "只有当用户本轮明确提到 event 时，才填充 event\_extractions。\\n" "=== 规则结束 ===\\n" ) def \_extract\_json(content: str) -> str: """ 从内容中提取第一个完整的 JSON 对象 处理 LLM 可能返回的额外数据： 1. 移除 markdown 代码块标记 2. 提取第一个完整的 JSON 对象 3. 忽略 JSON 后的额外内容 """ content = content.strip() # 移除 markdown 代码块标记 if content.startswith("\`\`\`"): end\_marker = content.find("\\n\`\`\`", 3) if end\_marker != -1: content = content\[3:end\_marker\].strip() else: lines = content.split("\\n", 1) if len(lines) > 1: content = lines\[1\].strip() if content.startswith("json"): content = content\[4:\].strip() if content.startswith("\`\`\`json"): content = content\[7:\].strip() elif content.startswith("\`\`\`"): content = content\[3:\].strip() start = content.find("{") if start == -1: start = content.find("\[") if start == -1: return content if content\[start\] == "{": depth = 0 in\_string = False escape\_next = False for i in range(start, len(content)): char = content\[i\] if escape\_next: escape\_next = False continue if char == "\\\\": escape\_next = True continue if char == '"' and not escape\_next: in\_string = not in\_string continue if in\_string: continue if char == "{": depth += 1 elif char == "}": depth -= 1 if depth == 0: return content\[start:i + 1\] return content\[start:\] # ==================== 数据模型 ==================== class Extraction(BaseModel): text: str class ExtractionsJson(BaseModel): metric\_extractions: List\[Extraction\] = Field(default\_factory=list) time\_extractions: List\[Extraction\] = Field(default\_factory=list) event\_extractions: List\[Extraction\] = Field(default\_factory=list) # 区域过滤：LLM 直接返回 \["EUTTP"\], \["USTTP"\], 或 \["ROW"\] # 注意：region\_filter 不能为空，默认为 \["ROW"\] region\_filter: List\[str\] = Field(default\_factory=lambda: \["ROW"\]) group\_by\_extractions: List\[Extraction\] = Field(default\_factory=list) # ==================== Prompt ==================== FEW\_SHOT\_EXAMPLES = r""" 示例: 德国 PV -> {"metric\_extractions":\[{"text":"PV"}\],"event\_extractions":\[\],"region\_filter":\["EUTTP"\]} 加州 UV -> {"metric\_extractions":\[{"text":"UV"}\],"event\_extractions":\[\],"region\_filter":\["USTTP"\]} 欧洲 app\_launch的PV -> {"metric\_extractions":\[{"text":"PV"}\],"event\_extractions":\[{"text":"app\_launch"}\],"region\_filter":\["EUTTP"\]} 欧洲 近7天 app\_launch -> {"metric\_extractions":\[{"text":"PV"}\],"time\_extractions":\[{"text":"近7天"}\],"event\_extractions":\[{"text":"app\_launch"}\],"region\_filter":\["EUTTP"\]} """ \_BASE\_SYSTEM\_PROMPT = r"""你是"提取器"。只做：从用户问题中抽取结构化片段（extractions）。 抽取规则： - metric\_extractions：从用户问题中提取指标 - 明确提到 PV/浏览量/访问量/点击量 → "PV" - 明确提到 UV/独立访客/访客数/用户数 → "UV" - 没有明确提到指标 → 默认提取为 "PV"（这是最常用的指标） - metric\_extractions 不能为空，必须有值 - event\_extractions：从用户问题中提取事件名称（重要！） - app\_launch/启动/应用启动/打开app → "app\_launch" - click/点击/点击事件 → "click" - view/浏览/浏览事件 → "view" - 其他事件名 → 直接提取原文 - 注意：如果提到 app launch、click、view 等，这些是事件，不是时间！ - 只有用户明确提到事件才提取，不要猜测或默认填充 - time\_extractions：从用户问题中提取时间范围 - 近7天/最近7天/7天内 → "近7天" - 昨天/昨天一天 → "昨天" - 今天/当天 → "今天" - 本周/这周 → "本周" - 本月/这个月 → "本月" - 注意：只有明确的时间描述才是 time，app\_launch 不是时间！ - region\_filter：直接返回区域代码列表，注意不是国家名而是区域代码！ - 欧洲/欧盟国家（德国、法国、意大利等）→ \["EUTTP"\] - 美国/美国州（加州、纽约等）→ \["USTTP"\] - 其他/未知/新加坡等 → \["ROW"\] - 无区域信息 → \["ROW"\] - group\_by\_extractions：从用户问题中提取分组维度 """ + FEW\_SHOT\_EXAMPLES # Prompt-based 专用 prompt（含格式指令） \_BASE\_SYSTEM\_PROMPT\_TEXT = \_BASE\_SYSTEM\_PROMPT + r""" 输出要求： - 不要输出任何解释文字 - 不要使用 Markdown 代码块（不要出现 \`\`\`） - 直接输出 JSON，必须以 { 开头，以 } 结尾 """ # ==================== Tool Schema (Function Calling) ==================== EXTRACT\_ENTITIES\_TOOL = { "type": "function", "function": { "name": "extract\_entities", "description": "从用户自然语言查询中提取结构化实体", "parameters": { "type": "object", "properties": { "metric\_extractions": { "type": "array", "items": { "type": "object", "properties": {"text": {"type": "string"}}, "required": \["text"\], }, "description": "指标提取，如 PV、UV", }, "time\_extractions": { "type": "array", "items": { "type": "object", "properties": {"text": {"type": "string"}}, "required": \["text"\], }, "description": "时间范围提取，如 近7天、昨天", }, "event\_extractions": { "type": "array", "items": { "type": "object", "properties": {"text": {"type": "string"}}, "required": \["text"\], }, "description": "事件名提取，如 app\_launch、click", }, "region\_filter": { "type": "array", "items": {"type": "string"}, "description": "区域代码列表，如 EUTTP、USTTP、ROW", }, "group\_by\_extractions": { "type": "array", "items": { "type": "object", "properties": {"text": {"type": "string"}}, "required": \["text"\], }, "description": "分组维度提取", }, }, "required": \["metric\_extractions", "region\_filter"\], }, }, } # ==================== LLM Client ==================== \_llm\_client: OpenAI | None = None def get\_llm\_client() -> OpenAI: """获取 OpenAI 兼容客户端（模块级单例，复用连接池）""" global \_llm\_client if \_llm\_client is not None: return \_llm\_client backend = os.getenv("LLM\_BACKEND", "zhipu").lower() if backend == "ollama": base\_url = os.getenv("OLLAMA\_BASE\_URL", "http://localhost:11434/v1") \_llm\_client = OpenAI( api\_key="ollama", base\_url=base\_url, ) else: # zhipu (默认) api\_key = os.getenv("ZHIPU\_API\_KEY", os.getenv("ZHIPUAI\_API\_KEY", "")) base\_url = os.getenv("ZHIPU\_BASE\_URL", "https://open.bigmodel.cn/api/coding/paas/v4") \_llm\_client = OpenAI(api\_key=api\_key, base\_url=base\_url) return \_llm\_client def \_get\_model\_name() -> str: """获取模型名称""" backend = os.getenv("LLM\_BACKEND", "zhipu").lower() if backend == "ollama": return os.getenv("OLLAMA\_MODEL", "glm4") return os.getenv("ZHIPU\_MODEL", "glm-4") def \_supports\_tool\_calling() -> bool: """检查是否启用 function calling 默认 true，所有 OpenAI 兼容后端均可使用。 设 TOOL\_CALLING\_ENABLED=false 可强制走 prompt-based 路径。 """ return os.getenv("TOOL\_CALLING\_ENABLED", "true").lower() == "true" # ==================== 构建消息 ==================== def \_build\_messages(query: str, session\_context: Optional\[Dict\[str, Any\]\], use\_tool\_calling: bool) -> list\[dict\[str, str\]\]: """构建 OpenAI 兼容的 messages 列表 注入结构（按顺序追加到 system prompt）： 1. 记忆文件内容（纠正/约束，借鉴 cc\_python section 9 的内容注入方式） 2. 动态上下文（QueryState + 最近消息） """ system\_prompt = \_BASE\_SYSTEM\_PROMPT if use\_tool\_calling else \_BASE\_SYSTEM\_PROMPT\_TEXT # 层 1：记忆文件内容（纠正/约束） if session\_context and session\_context.get("memory\_corrections"): corrections = session\_context\["memory\_corrections"\] system\_prompt += ( f"\\n\\n=== 已知约束和纠正 ===\\n" f"{corrections}\\n" f"=== 约束结束 ===\\n" f"请遵循上述约束处理用户查询。\\n" ) # 层 2：动态上下文（QueryState + 最近消息） context\_str = \_build\_context\_str(session\_context) if context\_str: system\_prompt += "\\n\\n请参考上述上下文理解用户当前问题。\\n" + context\_str # 层 3：follow-up patch extraction 指令 followup\_instruction = \_build\_followup\_instruction(session\_context) if followup\_instruction: system\_prompt += followup\_instruction messages = \[ {"role": "system", "content": system\_prompt}, {"role": "user", "content": f"用户问题：\\n{query}"}, \] if not use\_tool\_calling: messages\[-1\]\["content"\] += "\\n\\n返回 JSON：" return messages # ==================== 提取逻辑 ==================== def \_parse\_result(data: dict) -> ExtractionsJson: """解析并验证提取结果""" return ExtractionsJson.model\_validate(data) def extract\_llm(query: str, session\_context: Optional\[Dict\[str, Any\]\] = None) -> ExtractionsJson: """LLM 提取实体（支持会话上下文） Args: query: 用户查询文本 session\_context: 会话上下文，包含历史对话和已解析实体 """ # 检查缓存（暂不支持带上下文的缓存） if \_is\_cache\_enabled() and not session\_context: cache\_key = \_query\_hash(query) if cache\_key in \_query\_cache: logger.info(f"extract\_llm: cache hit for query: {query}") return ExtractionsJson(\*\*\_query\_cache\[cache\_key\]) client = get\_llm\_client() model = \_get\_model\_name() use\_tool\_calling = \_supports\_tool\_calling() messages = \_build\_messages(query, session\_context, use\_tool\_calling) kwargs: dict\[str, Any\] = { "model": model, "messages": messages, "temperature": 0.2, } logger.debug(f"LLM extract: model={model}, tool\_calling={use\_tool\_calling}, query={query\[:80\]}") if use\_tool\_calling: kwargs\["tools"\] = \[EXTRACT\_ENTITIES\_TOOL\] kwargs\["tool\_choice"\] = {"type": "function", "function": {"name": "extract\_entities"}} resp = client.chat.completions.create(\*\*kwargs) choice = resp.choices\[0\] message = choice.message # Function Calling 路径：从 tool\_calls 提取结构化参数 if use\_tool\_calling and message.tool\_calls: args = json.loads(message.tool\_calls\[0\].function.arguments) logger.debug(f"LLM extract (tool\_call): {json.dumps(args, ensure\_ascii=False)\[:200\]}") result = \_parse\_result(args) else: # Prompt-based 路径（fallback 或 Ollama） content = message.content if hasattr(message, "content") else str(message) content = \_extract\_json(content) logger.debug(f"LLM extract (prompt): raw={content\[:200\]}") try: data = json.loads(content) result = \_parse\_result(data) except (json.JSONDecodeError, ValidationError) as e: logger.error(f"LLM extract parse failed: {e}, raw={content\[:300\]}") raise ValueError(f"LLM extractions 输出不合法: {e}\\nRaw:\\n{content}") # 保存到缓存（仅无上下文时） if \_is\_cache\_enabled() and not session\_context: \_query\_cache\[cache\_key\] = result.model\_dump() return result async def extract\_llm\_async(query: str, session\_context: Optional\[Dict\[str, Any\]\] = None) -> ExtractionsJson: """extract\_llm 的异步包装，通过 asyncio.to\_thread 避免阻塞事件循环""" return await asyncio.to\_thread(extract\_llm, query, session\_context)

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89

90

91

92

93

94

95

96

97

98

99

100

101

102

103

104

105

106

107

108

109

110

111

112

113

114

115

116

117

118

119

120

121

122

123

124

125

126

127

128

129

130

131

132

133

134

135

136

137

138

139

140

141

142

143

144

145

146

147

148

149

150

151

152

153

154

155

379

380

381

382

383

384

385

386

387

388

389

390

391

392

393

394

395

396

397

398

399

400

401

402

403

404

405

406

407

408

409

410

411

412

413

414

415

416

417

418

419

420

421

422

423

424

425

426

427

428

429

430

431

432

433

434

435

436

437

438

439

440

441

442

443

444

445

446

447

448

449

450

451

452

from \_\_future\_\_ import annotations

import asyncio

import hashlib

import json

import logging

import os

from typing import Any, Dict, List, Optional

from dotenv import load\_dotenv

from openai import OpenAI

from pydantic import BaseModel, Field, ValidationError

load\_dotenv()

logger \= logging.getLogger(\_\_name\_\_)

\# ==================== 缓存机制 ====================

\_query\_cache: Dict\[str, dict\] \= {}

def \_query\_hash(text: str) \-> str:

"""生成查询的哈希值用于缓存"""

return hashlib.md5(text.encode()).hexdigest()

def \_is\_cache\_enabled() \-> bool:

"""检查是否启用缓存"""

return os.getenv("ENABLE\_LLM\_CACHE", "true").lower() \== "true"

\# ==================== 会话上下文 ====================

def \_build\_context\_str(session\_context: Optional\[Dict\[str, Any\]\]) \-> str:

"""构建动态会话上下文字符串（注入到提示词）

只保留有价值的动态上下文：

1\. QueryState — 上下文继承（最高价值）

2\. 最近用户消息 — 辅助理解

不再注入 favorite\_regions/metrics/events（LLM + Catalog 已够用）

记忆文件纠正内容在 \_build\_messages 中单独注入。

"""

if not session\_context:

return ""

parts \= \[\]

\# QueryState — 最高价值

if session\_context.get("last\_query\_state"):

qs \= session\_context\["last\_query\_state"\]

state\_parts \= \[\]

if qs.get("region\_filter"):

state\_parts.append(f"地区={','.join(qs\['region\_filter'\])}")

if qs.get("metric"):

state\_parts.append(f"指标={qs\['metric'\]}")

if qs.get("event"):

state\_parts.append(f"事件={qs\['event'\]}")

if qs.get("time\_range"):

tr \= qs\["time\_range"\]

state\_parts.append(f"时间=近{tr.get('n', '?')}天")

if qs.get("group\_by"):

state\_parts.append(f"分组={','.join(qs\['group\_by'\])}")

if qs.get("filters"):

filter\_strs \= \[f"{f\['field'\]}{f\['op'\]}{f\['value'\]}" for f in qs\["filters"\]\]

state\_parts.append(f"过滤={','.join(filter\_strs)}")

if state\_parts:

parts.append(f"上次查询: {', '.join(state\_parts)}")

\# 最近用户消息 — 辅助理解

recent \= session\_context.get("recent\_queries", \[\])

if recent:

history\_lines \= \[f"Q{i}: {q}" for i, q in enumerate(recent\[\-3:\], 1)\]

parts.append("=== 最近对话 ===\\n" + "\\n".join(history\_lines) + "\\n\=== 历史结束 ===")

return "\\n".join(parts) if parts else ""

def \_is\_followup\_turn(session\_context: Optional\[Dict\[str, Any\]\]) \-> bool:

"""是否处于更可能是 follow-up 的轮次。"""

if not session\_context:

return False

return bool(session\_context.get("last\_query\_state")) and bool(session\_context.get("turn\_index", 0))

def \_build\_followup\_instruction(session\_context: Optional\[Dict\[str, Any\]\]) \-> str:

"""为多轮 follow-up 场景追加更偏 patch extraction 的指令。"""

if not \_is\_followup\_turn(session\_context):

return ""

return (

"\\n\\n\=== 多轮查询补充规则 ===\\n"

"如果当前问题看起来是在补充、修改或缩写上一轮查询，请优先抽取本轮明确提到的新信息。\\n"

"不要为了补全而重复输出用户本轮没有明确说出的 event、metric、group\_by。\\n"

"如果本轮只说了时间、地区、指标或分组变化，就只抽取这些变化。\\n"

"只有当用户本轮明确提到 event 时，才填充 event\_extractions。\\n"

"=== 规则结束 ===\\n"

)

def \_extract\_json(content: str) \-> str:

"""

从内容中提取第一个完整的 JSON 对象

处理 LLM 可能返回的额外数据：

1\. 移除 markdown 代码块标记

2\. 提取第一个完整的 JSON 对象

3\. 忽略 JSON 后的额外内容

"""

content \= content.strip()

\# 移除 markdown 代码块标记

if content.startswith("\`\`\`"):

end\_marker \= content.find("\\n\`\`\`", 3)

if end\_marker != \-1:

content \= content\[3:end\_marker\].strip()

else:

lines \= content.split("\\n", 1)

if len(lines) \> 1:

content \= lines\[1\].strip()

if content.startswith("json"):

content \= content\[4:\].strip()

if content.startswith("\`\`\`json"):

content \= content\[7:\].strip()

elif content.startswith("\`\`\`"):

content \= content\[3:\].strip()

start \= content.find("{")

if start \== \-1:

start \= content.find("\[")

if start \== \-1:

return content

if content\[start\] \== "{":

depth \= 0

in\_string \= False

escape\_next \= False

for i in range(start, len(content)):

char \= content\[i\]

if escape\_next:

escape\_next \= False

continue

if char \== "\\\\":

escape\_next \= True

continue

if char \== '"' and not escape\_next:

in\_string \= not in\_string

continue

if in\_string:

if not use\_tool\_calling:

messages\[\-1\]\["content"\] += "\\n\\n返回 JSON："

return messages

\# ==================== 提取逻辑 ====================

def \_parse\_result(data: dict) \-> ExtractionsJson:

"""解析并验证提取结果"""

return ExtractionsJson.model\_validate(data)

def extract\_llm(query: str, session\_context: Optional\[Dict\[str, Any\]\] \= None) \-> ExtractionsJson:

"""LLM 提取实体（支持会话上下文）

Args:

query: 用户查询文本

session\_context: 会话上下文，包含历史对话和已解析实体

"""

\# 检查缓存（暂不支持带上下文的缓存）

if \_is\_cache\_enabled() and not session\_context:

cache\_key \= \_query\_hash(query)

if cache\_key in \_query\_cache:

logger.info(f"extract\_llm: cache hit for query: {query}")

return ExtractionsJson(\*\*\_query\_cache\[cache\_key\])

client \= get\_llm\_client()

model \= \_get\_model\_name()

use\_tool\_calling \= \_supports\_tool\_calling()

messages \= \_build\_messages(query, session\_context, use\_tool\_calling)

kwargs: dict\[str, Any\] \= {

"model": model,

"messages": messages,

"temperature": 0.2,

}

logger.debug(f"LLM extract: model={model}, tool\_calling={use\_tool\_calling}, query={query\[:80\]}")

if use\_tool\_calling:

kwargs\["tools"\] \= \[EXTRACT\_ENTITIES\_TOOL\]

kwargs\["tool\_choice"\] \= {"type": "function", "function": {"name": "extract\_entities"}}

resp \= client.chat.completions.create(\*\*kwargs)

choice \= resp.choices\[0\]

message \= choice.message

\# Function Calling 路径：从 tool\_calls 提取结构化参数

if use\_tool\_calling and message.tool\_calls:

args \= json.loads(message.tool\_calls\[0\].function.arguments)

logger.debug(f"LLM extract (tool\_call): {json.dumps(args, ensure\_ascii\=False)\[:200\]}")

result \= \_parse\_result(args)

else:

\# Prompt-based 路径（fallback 或 Ollama）

content \= message.content if hasattr(message, "content") else str(message)

content \= \_extract\_json(content)

logger.debug(f"LLM extract (prompt): raw={content\[:200\]}")

try:

data \= json.loads(content)

result \= \_parse\_result(data)

except (json.JSONDecodeError, ValidationError) as e:

logger.error(f"LLM extract parse failed: {e}, raw={content\[:300\]}")

raise ValueError(f"LLM extractions 输出不合法: {e}\\nRaw:\\n{content}")

\# 保存到缓存（仅无上下文时）

if \_is\_cache\_enabled() and not session\_context:

\_query\_cache\[cache\_key\] \= result.model\_dump()

return result

async def extract\_llm\_async(query: str, session\_context: Optional\[Dict\[str, Any\]\] \= None) \-> ExtractionsJson:

"""extract\_llm 的异步包装，通过 asyncio.to\_thread 避免阻塞事件循环"""

return await asyncio.to\_thread(extract\_llm, query, session\_context)