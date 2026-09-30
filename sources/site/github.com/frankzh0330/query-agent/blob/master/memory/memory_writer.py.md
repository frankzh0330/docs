# Source: https://github.com/frankzh0330/query-agent/blob/master/memory/memory_writer.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# memory\_writer.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/memory/memory_writer.py)

Open commit details

History

223 lines (181 loc) · 7.97 KB

## FilesExpand file tree

master

/

# memory\_writer.py

Copy path

Top

## File metadata and controls

- Code

- Blame

223 lines (181 loc) · 7.97 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/memory/memory_writer.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""LLM 驱动的自动记忆写入 在查询成功后，用一次轻量 LLM 调用判断本轮是否有值得记住的信息。 如果有，追加到 project 的 auto\_learned.md 文件中。 设计： - 异步非阻塞，不影晌主流程响应速度 - 追加到一个文件（auto\_learned.md），FIFO 淘汰 - 写入前检查去重 - 自动维护 MEMORY.md 索引 """ from \_\_future\_\_ import annotations import asyncio import hashlib import json import logging import re from pathlib import Path from typing import Any, Dict, Optional from memory.judge\_prompt import JUDGE\_SYSTEM\_PROMPT, JUDGE\_TOOL\_SCHEMA, JUDGE\_USER\_TEMPLATE logger = logging.getLogger(\_\_name\_\_) \_MAX\_AUTO\_LINES = 30 # auto\_learned.md 最大行数 class MemoryWriter: """LLM 驱动的记忆写入器""" def \_\_init\_\_(self, data\_path: str = "data/memory"): self.data\_path = Path(data\_path) async def maybe\_save( self, project\_id: int, user\_query: str, extraction: Dict\[str, Any\], resolver\_explain: Dict\[str, Any\], current\_state: Dict\[str, Any\], prev\_state: Optional\[Dict\[str, Any\]\] = None, confirmed\_selection: Optional\[Dict\[str, str\]\] = None, ) -> None: """LLM 判断是否写入记忆，是则追加到 auto\_learned.md 非阻塞：失败只记日志，不影响主流程。 """ try: project\_dir = self.data\_path / f"project\_{project\_id}" project\_dir.mkdir(parents=True, exist\_ok=True) # 读取已有记忆（用于去重 + 给 LLM 参考） existing = self.\_read\_existing(project\_dir) logger.debug(f"MemoryWriter: project={project\_id}, existing\_len={len(existing)}, query={user\_query\[:60\]}") # 如果有确认选择，注入到 resolver\_explain explain = dict(resolver\_explain) if confirmed\_selection: explain\["user\_confirmed"\] = confirmed\_selection # 调用 LLM 判断 logger.debug(f"MemoryWriter: calling judge for project={project\_id}") result = await self.\_judge( user\_query=user\_query, extraction=extraction, resolver\_explain=explain, current\_state=current\_state, prev\_state=prev\_state, existing\_memory=existing, ) if not result or not result.get("should\_save"): logger.debug("Memory judge: nothing to save") return content = result.get("content", "").strip() category = result.get("category", "correction") if not content: return # 去重检查 if self.\_is\_duplicate(project\_dir, content): logger.debug(f"Memory judge: duplicate, skip: {content\[:50\]}") return # 写入 self.\_append\_memory(project\_dir, category, content) logger.info(f"Memory saved \[{category}\]: {content\[:80\]}") except Exception as e: logger.warning(f"Memory writer error (non-critical): {e}") async def \_judge( self, user\_query: str, extraction: Dict\[str, Any\], resolver\_explain: Dict\[str, Any\], current\_state: Dict\[str, Any\], prev\_state: Optional\[Dict\[str, Any\]\], existing\_memory: str, ) -> Optional\[Dict\[str, Any\]\]: """调用 LLM 判断""" from service.llm\_extractions import get\_llm\_client, \_get\_model\_name, \_supports\_tool\_calling, \_extract\_json user\_content = JUDGE\_USER\_TEMPLATE.format( user\_query=user\_query, extraction=json.dumps(extraction, ensure\_ascii=False), resolver\_explain=json.dumps(resolver\_explain, ensure\_ascii=False), current\_state=json.dumps(current\_state, ensure\_ascii=False), prev\_state=json.dumps(prev\_state, ensure\_ascii=False) if prev\_state else "无（首次查询）", existing\_memory=existing\_memory if existing\_memory else "无", ) messages = \[ {"role": "system", "content": JUDGE\_SYSTEM\_PROMPT}, {"role": "user", "content": user\_content}, \] client = get\_llm\_client() model = \_get\_model\_name() use\_tool = \_supports\_tool\_calling() kwargs: Dict\[str, Any\] = { "model": model, "messages": messages, "temperature": 0.1, } if use\_tool: kwargs\["tools"\] = \[JUDGE\_TOOL\_SCHEMA\] kwargs\["tool\_choice"\] = {"type": "function", "function": {"name": "judge\_memory"}} # 异步执行（to\_thread 避免阻塞事件循环） resp = await asyncio.to\_thread( client.chat.completions.create, \*\*kwargs ) choice = resp.choices\[0\] message = choice.message if use\_tool and message.tool\_calls: return json.loads(message.tool\_calls\[0\].function.arguments) else: content = message.content if hasattr(message, "content") else str(message) content = \_extract\_json(content) try: return json.loads(content) except json.JSONDecodeError: logger.warning(f"Memory judge returned invalid JSON: {content\[:200\]}") return None def \_append\_memory(self, project\_dir: Path, category: str, content: str) -> None: """追加一条记忆到 auto\_learned.md""" auto\_file = project\_dir / "auto\_learned.md" # 确保索引文件包含 auto\_learned.md self.\_ensure\_index(project\_dir) # 追加一行 line = f"- \[{category}\] {content}\\n" if auto\_file.exists(): with open(auto\_file, "a", encoding="utf-8") as f: f.write(line) else: auto\_file.write\_text(line, encoding="utf-8") # FIFO 淘汰：超过行数限制时裁剪 self.\_trim\_if\_needed(auto\_file) def \_ensure\_index(self, project\_dir: Path) -> None: """确保 MEMORY.md 索引包含 auto\_learned.md""" index\_file = project\_dir / "MEMORY.md" auto\_entry = "- \[Auto Learned\](auto\_learned.md)\\n" if not index\_file.exists(): index\_file.write\_text(auto\_entry, encoding="utf-8") return content = index\_file.read\_text(encoding="utf-8") if "auto\_learned.md" not in content: # 追加到末尾 if not content.endswith("\\n"): content += "\\n" content += auto\_entry index\_file.write\_text(content, encoding="utf-8") def \_trim\_if\_needed(self, auto\_file: Path) -> None: """超过 \_MAX\_AUTO\_LINES 时保留最新的""" lines = auto\_file.read\_text(encoding="utf-8").strip().split("\\n") if len(lines) > \_MAX\_AUTO\_LINES: trimmed = "\\n".join(lines\[-\_MAX\_AUTO\_LINES:\]) + "\\n" auto\_file.write\_text(trimmed, encoding="utf-8") def \_is\_duplicate(self, project\_dir: Path, content: str) -> bool: """检查是否已有相似记忆（hash + 归一化全文去重）""" auto\_file = project\_dir / "auto\_learned.md" if not auto\_file.exists(): return False content\_hash = hashlib.md5(content.encode("utf-8")).hexdigest() existing = auto\_file.read\_text(encoding="utf-8") # 逐行 hash 去重 for line in existing.splitlines(): if hashlib.md5(line.strip().encode("utf-8")).hexdigest() == content\_hash: return True # 归一化全文包含检查 normalized\_existing = re.sub(r"\[^\\w\\u4e00-\\u9fff\]", "", existing) normalized\_content = re.sub(r"\[^\\w\\u4e00-\\u9fff\]", "", content) return normalized\_content in normalized\_existing def \_read\_existing(self, project\_dir: Path) -> str: """读取已有的 auto\_learned.md 内容""" auto\_file = project\_dir / "auto\_learned.md" if not auto\_file.exists(): return "" try: return auto\_file.read\_text(encoding="utf-8").strip() except OSError: return ""

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

156

157

158

159

160

161

162

163

164

165

166

167

168

169

170

171

172

173

174

175

176

177

178

179

180

181

182

183

184

185

186

187

188

189

190

191

192

193

194

195

196

197

198

199

200

201

202

203

204

205

206

207

208

209

210

211

212

213

214

215

216

217

218

219

220

221

222

223

"""LLM 驱动的自动记忆写入

在查询成功后，用一次轻量 LLM 调用判断本轮是否有值得记住的信息。

如果有，追加到 project 的 auto\_learned.md 文件中。

设计：

\- 异步非阻塞，不影晌主流程响应速度

\- 追加到一个文件（auto\_learned.md），FIFO 淘汰

\- 写入前检查去重

\- 自动维护 MEMORY.md 索引

"""

from \_\_future\_\_ import annotations

import asyncio

import hashlib

import json

import logging

import re

from pathlib import Path

from typing import Any, Dict, Optional

from memory.judge\_prompt import JUDGE\_SYSTEM\_PROMPT, JUDGE\_TOOL\_SCHEMA, JUDGE\_USER\_TEMPLATE

logger \= logging.getLogger(\_\_name\_\_)

\_MAX\_AUTO\_LINES \= 30 \# auto\_learned.md 最大行数

class MemoryWriter:

"""LLM 驱动的记忆写入器"""

def \_\_init\_\_(self, data\_path: str \= "data/memory"):

self.data\_path \= Path(data\_path)

async def maybe\_save(

self,

project\_id: int,

user\_query: str,

extraction: Dict\[str, Any\],

resolver\_explain: Dict\[str, Any\],

current\_state: Dict\[str, Any\],

prev\_state: Optional\[Dict\[str, Any\]\] \= None,

confirmed\_selection: Optional\[Dict\[str, str\]\] \= None,

) \-> None:

"""LLM 判断是否写入记忆，是则追加到 auto\_learned.md

非阻塞：失败只记日志，不影响主流程。

"""

try:

project\_dir \= self.data\_path / f"project\_{project\_id}"

project\_dir.mkdir(parents\=True, exist\_ok\=True)

\# 读取已有记忆（用于去重 + 给 LLM 参考）

existing \= self.\_read\_existing(project\_dir)

logger.debug(f"MemoryWriter: project={project\_id}, existing\_len={len(existing)}, query={user\_query\[:60\]}")

\# 如果有确认选择，注入到 resolver\_explain

explain \= dict(resolver\_explain)

if confirmed\_selection:

explain\["user\_confirmed"\] \= confirmed\_selection

\# 调用 LLM 判断

logger.debug(f"MemoryWriter: calling judge for project={project\_id}")

result \= await self.\_judge(

user\_query\=user\_query,

extraction\=extraction,

resolver\_explain\=explain,

current\_state\=current\_state,

prev\_state\=prev\_state,

existing\_memory\=existing,

)

if not result or not result.get("should\_save"):

logger.debug("Memory judge: nothing to save")

return

content \= result.get("content", "").strip()

category \= result.get("category", "correction")

if not content:

return

\# 去重检查

if self.\_is\_duplicate(project\_dir, content):

logger.debug(f"Memory judge: duplicate, skip: {content\[:50\]}")

return

\# 写入

self.\_append\_memory(project\_dir, category, content)

logger.info(f"Memory saved \[{category}\]: {content\[:80\]}")

except Exception as e:

logger.warning(f"Memory writer error (non-critical): {e}")

async def \_judge(

self,

user\_query: str,

extraction: Dict\[str, Any\],

resolver\_explain: Dict\[str, Any\],

current\_state: Dict\[str, Any\],

prev\_state: Optional\[Dict\[str, Any\]\],

existing\_memory: str,

) \-> Optional\[Dict\[str, Any\]\]:

"""调用 LLM 判断"""

from service.llm\_extractions import get\_llm\_client, \_get\_model\_name, \_supports\_tool\_calling, \_extract\_json

user\_content \= JUDGE\_USER\_TEMPLATE.format(

user\_query\=user\_query,

extraction\=json.dumps(extraction, ensure\_ascii\=False),

resolver\_explain\=json.dumps(resolver\_explain, ensure\_ascii\=False),

current\_state\=json.dumps(current\_state, ensure\_ascii\=False),

prev\_state\=json.dumps(prev\_state, ensure\_ascii\=False) if prev\_state else "无（首次查询）",

existing\_memory\=existing\_memory if existing\_memory else "无",

)

messages \= \[

{"role": "system", "content": JUDGE\_SYSTEM\_PROMPT},

{"role": "user", "content": user\_content},

\]

client \= get\_llm\_client()

model \= \_get\_model\_name()

use\_tool \= \_supports\_tool\_calling()

kwargs: Dict\[str, Any\] \= {

"model": model,

"messages": messages,

"temperature": 0.1,

}

if use\_tool:

kwargs\["tools"\] \= \[JUDGE\_TOOL\_SCHEMA\]

kwargs\["tool\_choice"\] \= {"type": "function", "function": {"name": "judge\_memory"}}

\# 异步执行（to\_thread 避免阻塞事件循环）

resp \= await asyncio.to\_thread(

client.chat.completions.create, \*\*kwargs

)

choice \= resp.choices\[0\]

message \= choice.message

if use\_tool and message.tool\_calls:

return json.loads(message.tool\_calls\[0\].function.arguments)

else:

content \= message.content if hasattr(message, "content") else str(message)

content \= \_extract\_json(content)

try:

return json.loads(content)

except json.JSONDecodeError:

logger.warning(f"Memory judge returned invalid JSON: {content\[:200\]}")

return None

def \_append\_memory(self, project\_dir: Path, category: str, content: str) \-> None:

"""追加一条记忆到 auto\_learned.md"""

auto\_file \= project\_dir / "auto\_learned.md"

\# 确保索引文件包含 auto\_learned.md

self.\_ensure\_index(project\_dir)

\# 追加一行

line \= f"- \[{category}\] {content}\\n"

if auto\_file.exists():

with open(auto\_file, "a", encoding\="utf-8") as f:

f.write(line)

else:

auto\_file.write\_text(line, encoding\="utf-8")

\# FIFO 淘汰：超过行数限制时裁剪

self.\_trim\_if\_needed(auto\_file)

def \_ensure\_index(self, project\_dir: Path) \-> None:

"""确保 MEMORY.md 索引包含 auto\_learned.md"""

index\_file \= project\_dir / "MEMORY.md"

auto\_entry \= "- \[Auto Learned\](auto\_learned.md)\\n"

if not index\_file.exists():

index\_file.write\_text(auto\_entry, encoding\="utf-8")

return

content \= index\_file.read\_text(encoding\="utf-8")

if "auto\_learned.md" not in content:

\# 追加到末尾

if not content.endswith("\\n"):

content += "\\n"

content += auto\_entry

index\_file.write\_text(content, encoding\="utf-8")

def \_trim\_if\_needed(self, auto\_file: Path) \-> None:

"""超过 \_MAX\_AUTO\_LINES 时保留最新的"""

lines \= auto\_file.read\_text(encoding\="utf-8").strip().split("\\n")

if len(lines) \> \_MAX\_AUTO\_LINES:

trimmed \= "\\n".join(lines\[\-\_MAX\_AUTO\_LINES:\]) + "\\n"

auto\_file.write\_text(trimmed, encoding\="utf-8")

def \_is\_duplicate(self, project\_dir: Path, content: str) \-> bool:

"""检查是否已有相似记忆（hash + 归一化全文去重）"""

auto\_file \= project\_dir / "auto\_learned.md"

if not auto\_file.exists():

return False

content\_hash \= hashlib.md5(content.encode("utf-8")).hexdigest()

existing \= auto\_file.read\_text(encoding\="utf-8")

\# 逐行 hash 去重

for line in existing.splitlines():

if hashlib.md5(line.strip().encode("utf-8")).hexdigest() \== content\_hash:

return True

\# 归一化全文包含检查

normalized\_existing \= re.sub(r"\[^\\w\\u4e00-\\u9fff\]", "", existing)

normalized\_content \= re.sub(r"\[^\\w\\u4e00-\\u9fff\]", "", content)

return normalized\_content in normalized\_existing

def \_read\_existing(self, project\_dir: Path) \-> str:

"""读取已有的 auto\_learned.md 内容"""

auto\_file \= project\_dir / "auto\_learned.md"

if not auto\_file.exists():

return ""

try:

return auto\_file.read\_text(encoding\="utf-8").strip()

except OSError:

return ""