# Source: https://github.com/frankzh0330/query-agent/blob/master/service/session_models.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# session\_models.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add multi-turn follow-up query pipeline and extract QueryOrches…](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4)

Open commit details

May 1, 2026

[3560e95](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4) · May 1, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/service/session_models.py)

Open commit details

History

202 lines (164 loc) · 6.82 KB

## FilesExpand file tree

master

/

# session\_models.py

Copy path

Top

## File metadata and controls

- Code

- Blame

202 lines (164 loc) · 6.82 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/session_models.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""会话数据模型""" from \_\_future\_\_ import annotations from dataclasses import dataclass, field from datetime import datetime from typing import Any, Dict, Optional @dataclass class QueryState: """最近一次查询的完整语义状态（替代旧的 resolved\_entities） 覆盖查询的所有维度：event, metric, time, region, group\_by, filters, chart\_type。 每次成功执行查询后更新，作为下一轮对话的默认补全来源。 优先级：explicit user input > session state (last\_query\_state) > preference memory """ project\_id: int = 55 # 主体 event: Optional\[str\] = None metric: Optional\[str\] = None # 时间 time\_range: Optional\[Dict\[str, Any\]\] = None # {"type": "last\_n\_days", "n": 7} # 维度 / 过滤 region\_filter: list\[str\] = field(default\_factory=list) group\_by: list\[str\] = field(default\_factory=list) filters: list\[Dict\[str, Any\]\] = field(default\_factory=list) # filters 格式: \[{"field": "platform", "op": "=", "value": "ios"}, ...\] # 展示 / 交互 chart\_type: Optional\[str\] = None interaction\_mode: Optional\[str\] = None # 调试 confidence: Optional\[float\] = None explicit\_fields: list\[str\] = field(default\_factory=list) inherited\_fields: list\[str\] = field(default\_factory=list) field\_sources: Dict\[str, str\] = field(default\_factory=dict) turn\_type: Optional\[str\] = None def to\_dict(self) -> Dict\[str, Any\]: return { "project\_id": self.project\_id, "event": self.event, "metric": self.metric, "time\_range": self.time\_range, "region\_filter": self.region\_filter, "group\_by": self.group\_by, "filters": self.filters, "chart\_type": self.chart\_type, "interaction\_mode": self.interaction\_mode, "confidence": self.confidence, "explicit\_fields": self.explicit\_fields, "inherited\_fields": self.inherited\_fields, "field\_sources": self.field\_sources, "turn\_type": self.turn\_type, } @dataclass class TaskContext: """单次查询任务的上下文 当用户需要确认（如多个候选 event）时，创建 TaskContext 追踪状态。 与 Session 解耦：通过 task\_id 索引，不挂在 session 上。 TTL 30 分钟，过期自动失效。 """ task\_id: str session\_id: str raw\_query: str user\_id: Optional\[str\] = None # LLM 提取结果 extraction: Dict\[str, Any\] = field(default\_factory=dict) # 正在构建中的 QueryState（尚未确认/执行） partial\_query\_state: Optional\[QueryState\] = None # 候选项: {"event": \[{"value": "payment\_submit", "score": 0.88}, ...\]} candidates: Dict\[str, list\[Dict\[str, Any\]\]\] = field(default\_factory=dict) # 用户已确认的选择: {"event": "payment\_success"} user\_selection: Dict\[str, str\] = field(default\_factory=dict) # 状态：waiting\_confirmation | confirmed | completed | cancelled | expired status: str = "waiting\_confirmation" turn\_type: str = "confirmation" resume\_count: int = 0 last\_prompt: Optional\[str\] = None created\_at: datetime = field(default\_factory=datetime.now) updated\_at: datetime = field(default\_factory=datetime.now) def is\_expired(self, ttl\_minutes: int = 30) -> bool: """检查任务是否过期""" age = (datetime.now() - self.created\_at).total\_seconds() / 60 return age > ttl\_minutes def to\_dict(self) -> Dict\[str, Any\]: return { "task\_id": self.task\_id, "session\_id": self.session\_id, "user\_id": self.user\_id, "raw\_query": self.raw\_query, "extraction": self.extraction, "partial\_query\_state": self.partial\_query\_state.to\_dict() if self.partial\_query\_state else None, "candidates": self.candidates, "user\_selection": self.user\_selection, "status": self.status, "turn\_type": self.turn\_type, "resume\_count": self.resume\_count, "last\_prompt": self.last\_prompt, "created\_at": self.created\_at.isoformat(), "updated\_at": self.updated\_at.isoformat(), } @dataclass class Message: """单条消息""" role: str # "user" or "assistant" content: str timestamp: datetime metadata: Dict\[str, Any\] = field(default\_factory=dict) def to\_dict(self) -> Dict\[str, Any\]: return { "role": self.role, "content": self.content, "timestamp": self.timestamp.isoformat(), "metadata": self.metadata, } @dataclass class SessionContext: """会话上下文""" session\_id: str user\_id: Optional\[str\] = None project\_id: int = 55 messages: list\[Message\] = field(default\_factory=list) # 最近一次"已确认 / 已执行成功"的查询状态 last\_query\_state: Optional\[QueryState\] = None # 最近一次结果摘要（可选） last\_result\_summary: Optional\[str\] = None # 当前等待用户确认的任务 ID pending\_task\_id: Optional\[str\] = None last\_turn\_type: Optional\[str\] = None last\_user\_query: Optional\[str\] = None turn\_index: int = 0 created\_at: datetime = field(default\_factory=datetime.now) last\_active: datetime = field(default\_factory=datetime.now) def add\_message(self, role: str, content: str, metadata: Dict\[str, Any\] | None = None) -> None: """添加消息到会话""" msg = Message( role=role, content=content, timestamp=datetime.now(), metadata=metadata or {} ) self.messages.append(msg) self.last\_active = datetime.now() if role == "user": self.last\_user\_query = content self.turn\_index += 1 def get\_recent\_messages(self, limit: int = 5) -> list\[Message\]: """获取最近的消息""" return self.messages\[-limit:\] if len(self.messages) > limit else self.messages def to\_dict(self) -> Dict\[str, Any\]: return { "session\_id": self.session\_id, "user\_id": self.user\_id, "project\_id": self.project\_id, "messages": \[m.to\_dict() for m in self.messages\], "last\_query\_state": self.last\_query\_state.to\_dict() if self.last\_query\_state else None, "last\_result\_summary": self.last\_result\_summary, "pending\_task\_id": self.pending\_task\_id, "last\_turn\_type": self.last\_turn\_type, "last\_user\_query": self.last\_user\_query, "turn\_index": self.turn\_index, "created\_at": self.created\_at.isoformat(), "last\_active": self.last\_active.isoformat(), } @dataclass class DailyMemory: """每日记忆（类似 OpenClaw 的 memory/YYYY-MM-DD.md）""" date: str entries: list\[str\] = field(default\_factory=list)

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

"""会话数据模型"""

from \_\_future\_\_ import annotations

from dataclasses import dataclass, field

from datetime import datetime

from typing import Any, Dict, Optional

@dataclass

class QueryState:

"""最近一次查询的完整语义状态（替代旧的 resolved\_entities）

覆盖查询的所有维度：event, metric, time, region, group\_by, filters, chart\_type。

每次成功执行查询后更新，作为下一轮对话的默认补全来源。

优先级：explicit user input > session state (last\_query\_state) > preference memory

"""

project\_id: int \= 55

\# 主体

event: Optional\[str\] \= None

metric: Optional\[str\] \= None

\# 时间

time\_range: Optional\[Dict\[str, Any\]\] \= None \# {"type": "last\_n\_days", "n": 7}

\# 维度 / 过滤

region\_filter: list\[str\] \= field(default\_factory\=list)

group\_by: list\[str\] \= field(default\_factory\=list)

filters: list\[Dict\[str, Any\]\] \= field(default\_factory\=list)

\# filters 格式: \[{"field": "platform", "op": "=", "value": "ios"}, ...\]

\# 展示 / 交互

chart\_type: Optional\[str\] \= None

interaction\_mode: Optional\[str\] \= None

\# 调试

confidence: Optional\[float\] \= None

explicit\_fields: list\[str\] \= field(default\_factory\=list)

inherited\_fields: list\[str\] \= field(default\_factory\=list)

field\_sources: Dict\[str, str\] \= field(default\_factory\=dict)

turn\_type: Optional\[str\] \= None

def to\_dict(self) \-> Dict\[str, Any\]:

return {

"project\_id": self.project\_id,

"event": self.event,

"metric": self.metric,

"time\_range": self.time\_range,

"region\_filter": self.region\_filter,

"group\_by": self.group\_by,

"filters": self.filters,

"chart\_type": self.chart\_type,

"interaction\_mode": self.interaction\_mode,

"confidence": self.confidence,

"explicit\_fields": self.explicit\_fields,

"inherited\_fields": self.inherited\_fields,

"field\_sources": self.field\_sources,

"turn\_type": self.turn\_type,

}

@dataclass

class TaskContext:

"""单次查询任务的上下文

当用户需要确认（如多个候选 event）时，创建 TaskContext 追踪状态。

与 Session 解耦：通过 task\_id 索引，不挂在 session 上。

TTL 30 分钟，过期自动失效。

"""

task\_id: str

session\_id: str

raw\_query: str

user\_id: Optional\[str\] \= None

\# LLM 提取结果

extraction: Dict\[str, Any\] \= field(default\_factory\=dict)

\# 正在构建中的 QueryState（尚未确认/执行）

partial\_query\_state: Optional\[QueryState\] \= None

\# 候选项: {"event": \[{"value": "payment\_submit", "score": 0.88}, ...\]}

candidates: Dict\[str, list\[Dict\[str, Any\]\]\] \= field(default\_factory\=dict)

\# 用户已确认的选择: {"event": "payment\_success"}

user\_selection: Dict\[str, str\] \= field(default\_factory\=dict)

\# 状态：waiting\_confirmation | confirmed | completed | cancelled | expired

status: str \= "waiting\_confirmation"

turn\_type: str \= "confirmation"

resume\_count: int \= 0

last\_prompt: Optional\[str\] \= None

created\_at: datetime \= field(default\_factory\=datetime.now)

updated\_at: datetime \= field(default\_factory\=datetime.now)

def is\_expired(self, ttl\_minutes: int \= 30) \-> bool:

"""检查任务是否过期"""

age \= (datetime.now() \- self.created\_at).total\_seconds() / 60

return age \> ttl\_minutes

def to\_dict(self) \-> Dict\[str, Any\]:

return {

"task\_id": self.task\_id,

"session\_id": self.session\_id,

"user\_id": self.user\_id,

"raw\_query": self.raw\_query,

"extraction": self.extraction,

"partial\_query\_state": self.partial\_query\_state.to\_dict() if self.partial\_query\_state else None,

"candidates": self.candidates,

"user\_selection": self.user\_selection,

"status": self.status,

"turn\_type": self.turn\_type,

"resume\_count": self.resume\_count,

"last\_prompt": self.last\_prompt,

"created\_at": self.created\_at.isoformat(),

"updated\_at": self.updated\_at.isoformat(),

}

@dataclass

class Message:

"""单条消息"""

role: str \# "user" or "assistant"

content: str

timestamp: datetime

metadata: Dict\[str, Any\] \= field(default\_factory\=dict)

def to\_dict(self) \-> Dict\[str, Any\]:

return {

"role": self.role,

"content": self.content,

"timestamp": self.timestamp.isoformat(),

"metadata": self.metadata,

}

@dataclass

class SessionContext:

"""会话上下文"""

session\_id: str

user\_id: Optional\[str\] \= None

project\_id: int \= 55

messages: list\[Message\] \= field(default\_factory\=list)

\# 最近一次"已确认 / 已执行成功"的查询状态

last\_query\_state: Optional\[QueryState\] \= None

\# 最近一次结果摘要（可选）

last\_result\_summary: Optional\[str\] \= None

\# 当前等待用户确认的任务 ID

pending\_task\_id: Optional\[str\] \= None

last\_turn\_type: Optional\[str\] \= None

last\_user\_query: Optional\[str\] \= None

turn\_index: int \= 0

created\_at: datetime \= field(default\_factory\=datetime.now)

last\_active: datetime \= field(default\_factory\=datetime.now)

def add\_message(self, role: str, content: str, metadata: Dict\[str, Any\] | None \= None) \-> None:

"""添加消息到会话"""

msg \= Message(

role\=role,

content\=content,

timestamp\=datetime.now(),

metadata\=metadata or {}

)

self.messages.append(msg)

self.last\_active \= datetime.now()

if role \== "user":

self.last\_user\_query \= content

self.turn\_index += 1

def get\_recent\_messages(self, limit: int \= 5) \-> list\[Message\]:

"""获取最近的消息"""

return self.messages\[\-limit:\] if len(self.messages) \> limit else self.messages

def to\_dict(self) \-> Dict\[str, Any\]:

return {

"session\_id": self.session\_id,

"user\_id": self.user\_id,

"project\_id": self.project\_id,

"messages": \[m.to\_dict() for m in self.messages\],

"last\_query\_state": self.last\_query\_state.to\_dict() if self.last\_query\_state else None,

"last\_result\_summary": self.last\_result\_summary,

"pending\_task\_id": self.pending\_task\_id,

"last\_turn\_type": self.last\_turn\_type,

"last\_user\_query": self.last\_user\_query,

"turn\_index": self.turn\_index,

"created\_at": self.created\_at.isoformat(),

"last\_active": self.last\_active.isoformat(),

}

@dataclass

class DailyMemory:

"""每日记忆（类似 OpenClaw 的 memory/YYYY-MM-DD.md）"""

date: str

entries: list\[str\] \= field(default\_factory\=list)