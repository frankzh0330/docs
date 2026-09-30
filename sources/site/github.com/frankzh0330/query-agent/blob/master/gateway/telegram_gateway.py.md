# Source: https://github.com/frankzh0330/query-agent/blob/master/gateway/telegram_gateway.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# telegram\_gateway.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add TaskManager confirmation flow and rebuild test suite (138 t…](https://github.com/frankzh0330/query-agent/commit/68792e59e016c780e1c83c89e4205a0ab5e4fafb)

Open commit details

Apr 14, 2026

[68792e5](https://github.com/frankzh0330/query-agent/commit/68792e59e016c780e1c83c89e4205a0ab5e4fafb) · Apr 14, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/gateway/telegram_gateway.py)

Open commit details

History

193 lines (156 loc) · 6.6 KB

## FilesExpand file tree

master

/

# telegram\_gateway.py

Copy path

Top

## File metadata and controls

- Code

- Blame

193 lines (156 loc) · 6.6 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/gateway/telegram_gateway.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import asyncio import json import logging import os from typing import Any, Dict, Optional import httpx from bus.message\_schema import BusMessage from gateway.base import BaseGateway from ingress.telegram\_adapter import TelegramAdapter logger = logging.getLogger(\_\_name\_\_) class TelegramGateway(BaseGateway): """ Telegram Bot Gateway 使用 long polling 方式接收消息。 通过 MessageBus 将消息传递给 AgentWorker 处理，不再直接调用业务逻辑。 """ def \_\_init\_\_(self, bot\_token: Optional\[str\] = None, bus=None): super().\_\_init\_\_(channel="telegram", bus=bus) self.bot\_token = bot\_token or os.getenv("TELEGRAM\_BOT\_TOKEN", "") self.base\_url = f"https://api.telegram.org/bot{self.bot\_token}" self.\_running = False self.\_offset: int = 0 # 用于 long polling async def start(self) -> None: """启动 Gateway，开始轮询消息""" if not self.bot\_token: logger.warning("Telegram bot token not configured, gateway will not start") return self.\_running = True logger.info("TelegramGateway started") while self.\_running: try: await self.\_poll\_updates() except Exception as e: logger.error(f"Error polling updates: {e}") await asyncio.sleep(5) async def stop(self) -> None: """停止 Gateway""" self.\_running = False logger.info("TelegramGateway stopped") async def \_poll\_updates(self) -> None: """轮询获取更新""" url = f"{self.base\_url}/getUpdates" params = { "offset": self.\_offset + 1, "timeout": 30, } async with httpx.AsyncClient() as client: response = await client.get(url, params=params, timeout=35) data = response.json() if not data.get("ok"): logger.error(f"Failed to get updates: {data}") return updates = data.get("result", \[\]) for update in updates: self.\_offset = update.get("update\_id", self.\_offset) await self.handle\_message(update) async def handle\_message(self, update: Dict\[str, Any\]) -> None: """处理消息：Ingress 清洗 → 入队""" adapter = TelegramAdapter() std\_msg = adapter.adapt(update) # 检查是否重复 if std\_msg.is\_duplicate: logger.info(f"Duplicate message ignored: {std\_msg.message\_id}") return # 检查清洗后是否为空 if not std\_msg.text: logger.info(f"Empty message after cleaning: {std\_msg.raw\_text}") return logger.info( f"Received message: user={std\_msg.user\_id}, " f"chat={std\_msg.chat\_id}, text={std\_msg.text}" ) # 构建统一消息并通过 Bus 发送 bus\_msg = BusMessage( channel=self.channel, chat\_id=std\_msg.chat\_id, user\_id=std\_msg.user\_id, text=std\_msg.text, project\_id=55, ) if self.bus: await self.bus.enqueue\_request(bus\_msg) else: logger.error("No bus configured, message dropped") def format\_response(self, nl2dsl\_result: Any, query\_result: Any = None) -> str: """格式化响应消息为 Telegram 文本""" status = nl2dsl\_result.get("status", "success") # 确认流：返回候选列表供用户选择 if status == "needs\_confirmation": return self.\_format\_confirmation(nl2dsl\_result) # 正常结果 semantic = nl2dsl\_result.get("semantic", {}) metric = semantic.get("metric", {}) event = semantic.get("event", {}) time\_range = semantic.get("time\_range", {}) lines = \[ "\\U0001F4CA 查询结果", f"地区: {', '.join(semantic.get('region\_filter', \[\]))}", f"指标: {metric.get('metric\_id', 'N/A')}", f"事件: {event.get('event\_name', 'N/A')}", f"时间: 近 {time\_range.get('n', 0)} 天", \] if query\_result and query\_result.get("success"): data = query\_result.get("result", query\_result) mock\_data = data.get("data", {}) records = mock\_data.get("result", mock\_data.get("data", \[\])) if records: lines.append("\\n\\U0001F4C8 数据结果") for record in records\[:5\]: parts = \[\] for k, v in record.items(): if k != "date": parts.append(f"{k}={v}") date\_val = record.get('date', 'N/A') if parts: lines.append(f"{date\_val}: {', '.join(parts)}") else: lines.append(f"{date\_val}: {json.dumps(record, ensure\_ascii=False)}") if len(records) > 5: lines.append(f"...(还有 {len(records) - 5} 条记录)") else: lines.append("\\n暂无数据") elif query\_result and query\_result.get("error"): lines.append(f"\\n\\U0000274C 查询失败: {query\_result.get('error')}") else: lines.append("\\n\\U000026A0\\uFE0F 未获取到查询结果") return "\\n".join(lines) async def send\_response(self, recipient: str, response: Dict\[str, Any\]) -> None: """发送响应""" url = f"{self.base\_url}/sendMessage" payload = { "chat\_id": recipient, "text": response.get("text", ""), } async with httpx.AsyncClient() as client: resp = await client.post(url, json=payload) data = resp.json() if not data.get("ok"): logger.error(f"Failed to send message: {data}") else: logger.info(f"Message sent to {recipient}") def \_format\_confirmation(self, nl2dsl\_result: Any) -> str: """格式化确认请求消息""" message = nl2dsl\_result.get("message", "") candidates = nl2dsl\_result.get("candidates", {}) if not candidates: return message lines = \[\] for field\_name, cands in candidates.items(): field\_display = {"event": "事件", "metric": "指标"}.get(field\_name, field\_name) lines.append(f"请选择{field\_display}:") for i, c in enumerate(cands\[:5\], 1): lines.append(f" {i}. {c\['value'\]} (匹配度 {c\['score'\]:.0f}%)") lines.append("回复编号或名称即可") return "\\n".join(lines)

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

from \_\_future\_\_ import annotations

import asyncio

import json

import logging

import os

from typing import Any, Dict, Optional

import httpx

from bus.message\_schema import BusMessage

from gateway.base import BaseGateway

from ingress.telegram\_adapter import TelegramAdapter

logger \= logging.getLogger(\_\_name\_\_)

class TelegramGateway(BaseGateway):

"""

Telegram Bot Gateway

使用 long polling 方式接收消息。

通过 MessageBus 将消息传递给 AgentWorker 处理，不再直接调用业务逻辑。

"""

def \_\_init\_\_(self, bot\_token: Optional\[str\] \= None, bus\=None):

super().\_\_init\_\_(channel\="telegram", bus\=bus)

self.bot\_token \= bot\_token or os.getenv("TELEGRAM\_BOT\_TOKEN", "")

self.base\_url \= f"https://api.telegram.org/bot{self.bot\_token}"

self.\_running \= False

self.\_offset: int \= 0 \# 用于 long polling

async def start(self) \-> None:

"""启动 Gateway，开始轮询消息"""

if not self.bot\_token:

logger.warning("Telegram bot token not configured, gateway will not start")

return

self.\_running \= True

logger.info("TelegramGateway started")

while self.\_running:

try:

await self.\_poll\_updates()

except Exception as e:

logger.error(f"Error polling updates: {e}")

await asyncio.sleep(5)

async def stop(self) \-> None:

"""停止 Gateway"""

self.\_running \= False

logger.info("TelegramGateway stopped")

async def \_poll\_updates(self) \-> None:

"""轮询获取更新"""

url \= f"{self.base\_url}/getUpdates"

params \= {

"offset": self.\_offset + 1,

"timeout": 30,

}

async with httpx.AsyncClient() as client:

response \= await client.get(url, params\=params, timeout\=35)

data \= response.json()

if not data.get("ok"):

logger.error(f"Failed to get updates: {data}")

return

updates \= data.get("result", \[\])

for update in updates:

self.\_offset \= update.get("update\_id", self.\_offset)

await self.handle\_message(update)

async def handle\_message(self, update: Dict\[str, Any\]) \-> None:

"""处理消息：Ingress 清洗 → 入队"""

adapter \= TelegramAdapter()

std\_msg \= adapter.adapt(update)

\# 检查是否重复

if std\_msg.is\_duplicate:

logger.info(f"Duplicate message ignored: {std\_msg.message\_id}")

return

\# 检查清洗后是否为空

if not std\_msg.text:

logger.info(f"Empty message after cleaning: {std\_msg.raw\_text}")

return

logger.info(

f"Received message: user={std\_msg.user\_id}, "

f"chat={std\_msg.chat\_id}, text={std\_msg.text}"

)

\# 构建统一消息并通过 Bus 发送

bus\_msg \= BusMessage(

channel\=self.channel,

chat\_id\=std\_msg.chat\_id,

user\_id\=std\_msg.user\_id,

text\=std\_msg.text,

project\_id\=55,

)

if self.bus:

await self.bus.enqueue\_request(bus\_msg)

else:

logger.error("No bus configured, message dropped")

def format\_response(self, nl2dsl\_result: Any, query\_result: Any \= None) \-> str:

"""格式化响应消息为 Telegram 文本"""

status \= nl2dsl\_result.get("status", "success")

\# 确认流：返回候选列表供用户选择

if status \== "needs\_confirmation":

return self.\_format\_confirmation(nl2dsl\_result)

\# 正常结果

semantic \= nl2dsl\_result.get("semantic", {})

metric \= semantic.get("metric", {})

event \= semantic.get("event", {})

time\_range \= semantic.get("time\_range", {})

lines \= \[

"\\U0001F4CA 查询结果",

f"地区: {', '.join(semantic.get('region\_filter', \[\]))}",

f"指标: {metric.get('metric\_id', 'N/A')}",

f"事件: {event.get('event\_name', 'N/A')}",

f"时间: 近 {time\_range.get('n', 0)} 天",

\]

if query\_result and query\_result.get("success"):

data \= query\_result.get("result", query\_result)

mock\_data \= data.get("data", {})

records \= mock\_data.get("result", mock\_data.get("data", \[\]))

if records:

lines.append("\\n\\U0001F4C8 数据结果")

for record in records\[:5\]:

parts \= \[\]

for k, v in record.items():

if k != "date":

parts.append(f"{k}\={v}")

date\_val \= record.get('date', 'N/A')

if parts:

lines.append(f"{date\_val}: {', '.join(parts)}")

else:

lines.append(f"{date\_val}: {json.dumps(record, ensure\_ascii\=False)}")

if len(records) \> 5:

lines.append(f"...(还有 {len(records) \- 5} 条记录)")

else:

lines.append("\\n暂无数据")

elif query\_result and query\_result.get("error"):

lines.append(f"\\n\\U0000274C 查询失败: {query\_result.get('error')}")

else:

lines.append("\\n\\U000026A0\\uFE0F 未获取到查询结果")

return "\\n".join(lines)

async def send\_response(self, recipient: str, response: Dict\[str, Any\]) \-> None:

"""发送响应"""

url \= f"{self.base\_url}/sendMessage"

payload \= {

"chat\_id": recipient,

"text": response.get("text", ""),

}

async with httpx.AsyncClient() as client:

resp \= await client.post(url, json\=payload)

data \= resp.json()

if not data.get("ok"):

logger.error(f"Failed to send message: {data}")

else:

logger.info(f"Message sent to {recipient}")

def \_format\_confirmation(self, nl2dsl\_result: Any) \-> str:

"""格式化确认请求消息"""

message \= nl2dsl\_result.get("message", "")

candidates \= nl2dsl\_result.get("candidates", {})

if not candidates:

return message

lines \= \[\]

for field\_name, cands in candidates.items():

field\_display \= {"event": "事件", "metric": "指标"}.get(field\_name, field\_name)

lines.append(f"请选择{field\_display}:")

for i, c in enumerate(cands\[:5\], 1):

lines.append(f" {i}. {c\['value'\]} (匹配度 {c\['score'\]:.0f}%)")

lines.append("回复编号或名称即可")

return "\\n".join(lines)