# Source: https://github.com/frankzh0330/query-agent/blob/master/service/session_manager.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# session\_manager.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/service/session_manager.py)

Open commit details

History

269 lines (225 loc) · 9.73 KB

## FilesExpand file tree

master

/

# session\_manager.py

Copy path

Top

## File metadata and controls

- Code

- Blame

269 lines (225 loc) · 9.73 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/session_manager.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""会话管理器 支持 JSONL 持久化和文件恢复，集成长期记忆。 架构： - Working Memory: 内存中的 SessionContext（带 JSONL 持久化） - Long-Term Memory: 记忆文件（纠正/约束） 优先级链：explicit user input > session state (last\_query\_state) > memory corrections """ from \_\_future\_\_ import annotations import logging import uuid from datetime import datetime from typing import Any, Dict, Optional from memory.long\_term\_memory import LongTermMemory from memory.storage.memory\_file import SessionStorage from service.session\_models import Message, QueryState, SessionContext logger = logging.getLogger(\_\_name\_\_) class SessionManager: """会话管理器（JSONL 持久化 + 文件恢复） - Session 数据追加写入 JSONL 文件，服务重启后可恢复 - 集成 LongTermMemory 读取纠正/约束记忆 """ def \_\_init\_\_(self, data\_path: str = "data", max\_messages: int = 20): self.\_sessions: Dict\[str, SessionContext\] = {} self.\_max\_messages = max\_messages self.storage = SessionStorage(f"{data\_path}/sessions") self.long\_term = LongTermMemory(f"{data\_path}/memory") def create\_or\_get( self, session\_id: str | None, user\_id: str | None, project\_id: int ) -> SessionContext: """创建或恢复会话""" # 内存命中 if session\_id and session\_id in self.\_sessions: ctx = self.\_sessions\[session\_id\] ctx.last\_active = datetime.now() logger.debug(f"Retrieved existing session: {session\_id}") return ctx # 尝试从 JSONL 文件恢复 if session\_id: ctx = self.\_load\_from\_storage(session\_id, user\_id, project\_id) if ctx: self.\_sessions\[session\_id\] = ctx logger.info(f"Restored session from file: {session\_id}") return ctx # 创建新会话 new\_id = session\_id or str(uuid.uuid4()) ctx = SessionContext( session\_id=new\_id, user\_id=user\_id, project\_id=project\_id ) self.\_sessions\[new\_id\] = ctx logger.info( f"Created new session: {new\_id} (user\_id={user\_id}, project\_id={project\_id})" ) return ctx def \_load\_from\_storage( self, session\_id: str, user\_id: str | None, project\_id: int ) -> Optional\[SessionContext\]: """从 JSONL 文件恢复会话""" records = self.storage.read\_tail(session\_id, n=self.\_max\_messages) if not records: return None ctx = SessionContext( session\_id=session\_id, user\_id=user\_id, project\_id=project\_id ) for r in records: if r.get("type") == "message": try: msg = Message( role=r\["role"\], content=r\["content"\], timestamp=datetime.fromisoformat(r\["timestamp"\]), metadata=r.get("metadata", {}), ) ctx.messages.append(msg) if msg.role == "user": ctx.last\_user\_query = msg.content ctx.turn\_index += 1 except (KeyError, ValueError) as e: logger.warning(f"Skipping invalid message record: {e}") elif r.get("type") == "query\_state": try: ctx.last\_query\_state = QueryState(\*\*r\["data"\]) ctx.last\_turn\_type = r.get("turn\_type") or ctx.last\_query\_state.turn\_type except (TypeError, ValueError) as e: logger.warning(f"Skipping invalid query\_state record: {e}") elif r.get("type") == "session\_meta": ctx.pending\_task\_id = r.get("pending\_task\_id") ctx.last\_turn\_type = r.get("last\_turn\_type", ctx.last\_turn\_type) ctx.last\_user\_query = r.get("last\_user\_query", ctx.last\_user\_query) ctx.turn\_index = r.get("turn\_index", ctx.turn\_index) ctx.last\_active = datetime.now() return ctx def add\_message( self, session\_id: str, role: str, content: str, metadata: Dict\[str, Any\] | None = None, ) -> None: """添加消息到会话（内存 + 持久化）""" if session\_id not in self.\_sessions: logger.warning(f"Session not found: {session\_id}") return ctx = self.\_sessions\[session\_id\] ctx.add\_message(role, content, metadata) # 持久化到 JSONL self.storage.append( session\_id, { "type": "message", "role": role, "content": content, "timestamp": datetime.now().isoformat(), "metadata": metadata or {}, }, ) # 截断内存中的消息（文件保留完整记录） if len(ctx.messages) > self.\_max\_messages: removed = len(ctx.messages) - self.\_max\_messages ctx.messages = ctx.messages\[-self.\_max\_messages :\] logger.debug(f"Trimmed {removed} old messages from session: {session\_id}") def update\_query\_state(self, session\_id: str, query\_state: QueryState) -> None: """更新最近一次查询状态（内存 + 持久化）""" if session\_id not in self.\_sessions: logger.warning(f"Session not found: {session\_id}") return ctx = self.\_sessions\[session\_id\] ctx.last\_query\_state = query\_state ctx.last\_turn\_type = query\_state.turn\_type self.storage.append( session\_id, { "type": "query\_state", "data": query\_state.to\_dict(), "turn\_type": query\_state.turn\_type, "timestamp": datetime.now().isoformat(), }, ) logger.debug(f"Updated last\_query\_state for session {session\_id}") def update\_pending\_task(self, session\_id: str, task\_id: str | None) -> None: """更新 session 的 pending\_task\_id（内存 + 持久化）。""" if session\_id not in self.\_sessions: logger.warning(f"Session not found: {session\_id}") return ctx = self.\_sessions\[session\_id\] ctx.pending\_task\_id = task\_id self.storage.append( session\_id, { "type": "session\_meta", "pending\_task\_id": task\_id, "last\_turn\_type": ctx.last\_turn\_type, "last\_user\_query": ctx.last\_user\_query, "turn\_index": ctx.turn\_index, "timestamp": datetime.now().isoformat(), }, ) logger.debug(f"Updated pending\_task\_id for session {session\_id}: {task\_id}") def get\_query\_state(self, session\_id: str) -> Optional\[QueryState\]: """获取最近一次查询状态""" ctx = self.\_sessions.get(session\_id) return ctx.last\_query\_state if ctx else None def get\_context(self, session\_id: str) -> Dict\[str, Any\]: """获取会话上下文（用于 LLM）""" if session\_id not in self.\_sessions: return {} ctx = self.\_sessions\[session\_id\] recent = ctx.get\_recent\_messages(limit=5) result: Dict\[str, Any\] = { "recent\_queries": \[m.content for m in recent if m.role == "user"\], "project\_id": ctx.project\_id, "last\_turn\_type": ctx.last\_turn\_type, "last\_user\_query": ctx.last\_user\_query, "turn\_index": ctx.turn\_index, } if ctx.last\_query\_state: result\["last\_query\_state"\] = ctx.last\_query\_state.to\_dict() return result def get\_session(self, session\_id: str) -> SessionContext | None: """获取会话""" return self.\_sessions.get(session\_id) def list\_sessions(self) -> list\[SessionContext\]: """列出所有会话""" return list(self.\_sessions.values()) def delete\_session(self, session\_id: str) -> bool: """删除会话（内存 + 磁盘文件）""" if session\_id not in self.\_sessions: return False del self.\_sessions\[session\_id\] self.storage.delete(session\_id) logger.info("Deleted session: %s", session\_id) return True def cleanup\_inactive(self, max\_age\_minutes: int = 60) -> int: """清理不活跃的会话""" now = datetime.now() to\_remove = \[\] for sid, ctx in self.\_sessions.items(): age = (now - ctx.last\_active).total\_seconds() / 60 if age > max\_age\_minutes: to\_remove.append(sid) for sid in to\_remove: self.delete\_session(sid) if to\_remove: logger.info("Cleaned up %d inactive sessions", len(to\_remove)) return len(to\_remove) # ============ 增强上下文（集成长期记忆）============ def get\_enhanced\_context(self, session\_id: str, query\_text: str | None = None) -> Dict\[str, Any\]: """获取增强上下文（会话 + 记忆文件） Returns: 包含短期上下文（会话历史 + last\_query\_state） 和长期记忆（纠正/约束）的字典 """ session = self.\_sessions.get(session\_id) if not session: return {} context = self.get\_context(session\_id) # 按 project\_id 分桶加载记忆文件内容（纠正/约束） memory\_content = self.long\_term.load\_memory\_context( project\_id=session.project\_id, query\_text=query\_text, last\_query\_state=context.get("last\_query\_state"), ) if memory\_content: context\["memory\_corrections"\] = memory\_content logger.debug(f"Enhanced context for {session\_id}: injected {len(memory\_content)} chars memory") return context

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

224

225

226

227

228

229

230

231

232

233

234

235

236

237

238

239

240

241

242

243

244

245

246

247

248

249

250

251

252

253

254

255

256

257

258

259

260

261

262

263

264

265

266

267

268

269

"""会话管理器

支持 JSONL 持久化和文件恢复，集成长期记忆。

架构：

\- Working Memory: 内存中的 SessionContext（带 JSONL 持久化）

\- Long-Term Memory: 记忆文件（纠正/约束）

优先级链：explicit user input > session state (last\_query\_state) > memory corrections

"""

from \_\_future\_\_ import annotations

import logging

import uuid

from datetime import datetime

from typing import Any, Dict, Optional

from memory.long\_term\_memory import LongTermMemory

from memory.storage.memory\_file import SessionStorage

from service.session\_models import Message, QueryState, SessionContext

logger \= logging.getLogger(\_\_name\_\_)

class SessionManager:

"""会话管理器（JSONL 持久化 + 文件恢复）

\- Session 数据追加写入 JSONL 文件，服务重启后可恢复

\- 集成 LongTermMemory 读取纠正/约束记忆

"""

def \_\_init\_\_(self, data\_path: str \= "data", max\_messages: int \= 20):

self.\_sessions: Dict\[str, SessionContext\] \= {}

self.\_max\_messages \= max\_messages

self.storage \= SessionStorage(f"{data\_path}/sessions")

self.long\_term \= LongTermMemory(f"{data\_path}/memory")

def create\_or\_get(

self, session\_id: str | None, user\_id: str | None, project\_id: int

) \-> SessionContext:

"""创建或恢复会话"""

\# 内存命中

if session\_id and session\_id in self.\_sessions:

ctx \= self.\_sessions\[session\_id\]

ctx.last\_active \= datetime.now()

logger.debug(f"Retrieved existing session: {session\_id}")

return ctx

\# 尝试从 JSONL 文件恢复

if session\_id:

ctx \= self.\_load\_from\_storage(session\_id, user\_id, project\_id)

if ctx:

self.\_sessions\[session\_id\] \= ctx

logger.info(f"Restored session from file: {session\_id}")

return ctx

\# 创建新会话

new\_id \= session\_id or str(uuid.uuid4())

ctx \= SessionContext(

session\_id\=new\_id, user\_id\=user\_id, project\_id\=project\_id

)

self.\_sessions\[new\_id\] \= ctx

logger.info(

f"Created new session: {new\_id} (user\_id={user\_id}, project\_id={project\_id})"

)

return ctx

def \_load\_from\_storage(

self, session\_id: str, user\_id: str | None, project\_id: int

) \-> Optional\[SessionContext\]:

"""从 JSONL 文件恢复会话"""

records \= self.storage.read\_tail(session\_id, n\=self.\_max\_messages)

if not records:

return None

ctx \= SessionContext(

session\_id\=session\_id, user\_id\=user\_id, project\_id\=project\_id

)

for r in records:

if r.get("type") \== "message":

try:

msg \= Message(

role\=r\["role"\],

content\=r\["content"\],

timestamp\=datetime.fromisoformat(r\["timestamp"\]),

metadata\=r.get("metadata", {}),

)

ctx.messages.append(msg)

if msg.role \== "user":

ctx.last\_user\_query \= msg.content

ctx.turn\_index += 1

except (KeyError, ValueError) as e:

logger.warning(f"Skipping invalid message record: {e}")

elif r.get("type") \== "query\_state":

try:

ctx.last\_query\_state \= QueryState(\*\*r\["data"\])

ctx.last\_turn\_type \= r.get("turn\_type") or ctx.last\_query\_state.turn\_type

except (TypeError, ValueError) as e:

logger.warning(f"Skipping invalid query\_state record: {e}")

elif r.get("type") \== "session\_meta":

ctx.pending\_task\_id \= r.get("pending\_task\_id")

ctx.last\_turn\_type \= r.get("last\_turn\_type", ctx.last\_turn\_type)

ctx.last\_user\_query \= r.get("last\_user\_query", ctx.last\_user\_query)

ctx.turn\_index \= r.get("turn\_index", ctx.turn\_index)

ctx.last\_active \= datetime.now()

return ctx

def add\_message(

self,

session\_id: str,

role: str,

content: str,

metadata: Dict\[str, Any\] | None \= None,

) \-> None:

"""添加消息到会话（内存 + 持久化）"""

if session\_id not in self.\_sessions:

logger.warning(f"Session not found: {session\_id}")

return

ctx \= self.\_sessions\[session\_id\]

ctx.add\_message(role, content, metadata)

\# 持久化到 JSONL

self.storage.append(

session\_id,

{

"type": "message",

"role": role,

"content": content,

"timestamp": datetime.now().isoformat(),

"metadata": metadata or {},

},

)

\# 截断内存中的消息（文件保留完整记录）

if len(ctx.messages) \> self.\_max\_messages:

removed \= len(ctx.messages) \- self.\_max\_messages

ctx.messages \= ctx.messages\[\-self.\_max\_messages :\]

logger.debug(f"Trimmed {removed} old messages from session: {session\_id}")

def update\_query\_state(self, session\_id: str, query\_state: QueryState) \-> None:

"""更新最近一次查询状态（内存 + 持久化）"""

if session\_id not in self.\_sessions:

logger.warning(f"Session not found: {session\_id}")

return

ctx \= self.\_sessions\[session\_id\]

ctx.last\_query\_state \= query\_state

ctx.last\_turn\_type \= query\_state.turn\_type

self.storage.append(

session\_id,

{

"type": "query\_state",

result: Dict\[str, Any\] \= {

"recent\_queries": \[m.content for m in recent if m.role \== "user"\],

"project\_id": ctx.project\_id,

"last\_turn\_type": ctx.last\_turn\_type,

"last\_user\_query": ctx.last\_user\_query,

"turn\_index": ctx.turn\_index,

}

if ctx.last\_query\_state:

result\["last\_query\_state"\] \= ctx.last\_query\_state.to\_dict()

return result

def get\_session(self, session\_id: str) \-> SessionContext | None:

"""获取会话"""

return self.\_sessions.get(session\_id)

def list\_sessions(self) \-> list\[SessionContext\]:

"""列出所有会话"""

return list(self.\_sessions.values())

def delete\_session(self, session\_id: str) \-> bool:

"""删除会话（内存 + 磁盘文件）"""

if session\_id not in self.\_sessions:

return False

del self.\_sessions\[session\_id\]

self.storage.delete(session\_id)

logger.info("Deleted session: %s", session\_id)

return True

def cleanup\_inactive(self, max\_age\_minutes: int \= 60) \-> int:

"""清理不活跃的会话"""

now \= datetime.now()

to\_remove \= \[\]

for sid, ctx in self.\_sessions.items():

age \= (now \- ctx.last\_active).total\_seconds() / 60

if age \> max\_age\_minutes:

to\_remove.append(sid)

for sid in to\_remove:

self.delete\_session(sid)

if to\_remove:

logger.info("Cleaned up %d inactive sessions", len(to\_remove))

return len(to\_remove)

\# ============ 增强上下文（集成长期记忆）============

def get\_enhanced\_context(self, session\_id: str, query\_text: str | None \= None) \-> Dict\[str, Any\]:

"""获取增强上下文（会话 + 记忆文件）

Returns:

包含短期上下文（会话历史 + last\_query\_state）

和长期记忆（纠正/约束）的字典

"""

session \= self.\_sessions.get(session\_id)

if not session:

return {}

context \= self.get\_context(session\_id)

\# 按 project\_id 分桶加载记忆文件内容（纠正/约束）

memory\_content \= self.long\_term.load\_memory\_context(

project\_id\=session.project\_id,

query\_text\=query\_text,

last\_query\_state\=context.get("last\_query\_state"),

)

if memory\_content:

context\["memory\_corrections"\] \= memory\_content

logger.debug(f"Enhanced context for {session\_id}: injected {len(memory\_content)} chars memory")

return context