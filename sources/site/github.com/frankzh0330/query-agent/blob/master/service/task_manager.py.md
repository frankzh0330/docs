# Source: https://github.com/frankzh0330/query-agent/blob/master/service/task_manager.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# task\_manager.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/service/task_manager.py)

Open commit details

History

252 lines (211 loc) · 8.77 KB

## FilesExpand file tree

master

/

# task\_manager.py

Copy path

Top

## File metadata and controls

- Code

- Blame

252 lines (211 loc) · 8.77 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/task_manager.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""任务管理器：独立于 Session，管理查询任务的创建/确认/过期""" from \_\_future\_\_ import annotations import logging import uuid from datetime import datetime from typing import Any, Dict, Optional from memory.storage.memory\_file import TaskStorage from service.session\_models import TaskContext, QueryState logger = logging.getLogger(\_\_name\_\_) class TaskManager: """任务管理器 管理查询任务的完整生命周期：创建 → 确认 → 完成 → 过期。 关键设计： - 与 Session 解耦：通过 task\_id 索引，不挂在 session 上 - TTL 机制：默认 30 分钟过期 - 幂等：同一 task\_id 多次 confirm 不会重复执行 """ DEFAULT\_TTL\_MINUTES = 30 def \_\_init\_\_(self, ttl\_minutes: int = DEFAULT\_TTL\_MINUTES, storage: TaskStorage | None = None): self.\_tasks: Dict\[str, TaskContext\] = {} self.\_ttl\_minutes = ttl\_minutes self.storage = storage def create\_task( self, session\_id: str, raw\_query: str, extraction: Dict\[str, Any\], user\_id: Optional\[str\] = None, candidates: Optional\[Dict\[str, list\[Dict\[str, Any\]\]\]\] = None, partial\_query\_state: Optional\[QueryState\] = None, ) -> TaskContext: """创建一个待确认/待执行任务 Args: session\_id: 所属会话 raw\_query: 用户原始输入 extraction: LLM 提取结果 user\_id: 用户ID candidates: 候选项 {"event": \[{"value": "x", "score": 0.9}\]} partial\_query\_state: 正在构建中的查询状态 Returns: TaskContext """ task\_id = uuid.uuid4().hex\[:12\] task = TaskContext( task\_id=task\_id, session\_id=session\_id, user\_id=user\_id, raw\_query=raw\_query, extraction=extraction, partial\_query\_state=partial\_query\_state, candidates=candidates or {}, status="waiting\_confirmation", ) self.\_tasks\[task\_id\] = task self.\_persist\_task(task) logger.info(f"Task created: {task\_id} (session={session\_id}, status=waiting\_confirmation)") return task def get\_task(self, task\_id: str) -> Optional\[TaskContext\]: """读取任务 如果任务已过期，自动标记为 expired 并返回 None。 """ task = self.\_tasks.get(task\_id) if not task: restored = self.\_restore\_task(task\_id) if restored: task = restored else: return None if task.is\_expired(self.\_ttl\_minutes): task.status = "expired" self.\_persist\_task(task) logger.info(f"Task expired: {task\_id}") return None return task def confirm\_task(self, task\_id: str, field: str, value: str) -> Optional\[TaskContext\]: """用户确认某个候选 幂等：重复 confirm 同一 field 不会报错，只更新值。 Args: task\_id: 任务ID field: 确认的字段（如 "event"） value: 用户选择的值（如 "payment\_success"） Returns: 更新后的 TaskContext，如果任务不存在或已过期则返回 None """ task = self.get\_task(task\_id) if not task: logger.warning(f"Task not found or expired: {task\_id}") return None # 1. 记录用户选择 task.user\_selection\[field\] = value # 2. patch partial\_query\_state if task.partial\_query\_state: if field == "event": task.partial\_query\_state.event = value elif field == "metric": task.partial\_query\_state.metric = value elif field == "region": task.partial\_query\_state.region\_filter = \[value\] elif field == "group\_by": task.partial\_query\_state.group\_by = value if isinstance(value, list) else \[value\] elif field == "chart\_type": task.partial\_query\_state.chart\_type = value elif field == "interaction\_mode": task.partial\_query\_state.interaction\_mode = value # 3. 检查是否所有需要确认的字段都已确认 all\_confirmed = all( field in task.user\_selection for field in task.candidates.keys() ) if all\_confirmed: task.status = "confirmed" task.updated\_at = datetime.now() self.\_persist\_task(task) logger.info(f"Task confirmed: {task\_id}, field={field}, value={value}, all\_confirmed={all\_confirmed}") return task def update\_status(self, task\_id: str, status: str) -> Optional\[TaskContext\]: """更新任务状态 状态机：waiting\_confirmation → confirmed → completed → cancelled → expired（自动） """ task = self.get\_task(task\_id) if not task: return None valid\_transitions = { "waiting\_confirmation": \["confirmed", "cancelled"\], "confirmed": \["completed", "cancelled"\], } current = task.status if status in valid\_transitions.get(current, \[\]): task.status = status task.updated\_at = datetime.now() self.\_persist\_task(task) logger.info(f"Task status updated: {task\_id} {current} → {status}") else: logger.warning(f"Invalid status transition: {task\_id} {current} → {status}") return task def expire\_task(self, task\_id: str) -> None: """手动标记任务过期""" task = self.\_tasks.get(task\_id) if task: task.status = "expired" task.updated\_at = datetime.now() self.\_persist\_task(task) logger.info(f"Task manually expired: {task\_id}") def cleanup\_expired(self) -> int: """清理所有过期任务，返回清理数量""" to\_remove = \[\] for task\_id, task in self.\_tasks.items(): if task.is\_expired(self.\_ttl\_minutes) or task.status == "expired": to\_remove.append(task\_id) for task\_id in to\_remove: del self.\_tasks\[task\_id\] if self.storage: self.storage.delete(task\_id) if to\_remove: logger.info(f"Cleaned up {len(to\_remove)} expired tasks") return len(to\_remove) def list\_tasks(self, session\_id: Optional\[str\] = None) -> list\[TaskContext\]: """列出任务，可按 session\_id 过滤""" tasks = list(self.\_tasks.values()) if session\_id: tasks = \[t for t in tasks if t.session\_id == session\_id\] return tasks def \_persist\_task(self, task: TaskContext) -> None: if not self.storage: return self.storage.append( task.task\_id, { "task\_id": task.task\_id, "session\_id": task.session\_id, "user\_id": task.user\_id, "raw\_query": task.raw\_query, "extraction": task.extraction, "partial\_query\_state": task.partial\_query\_state.to\_dict() if task.partial\_query\_state else None, "candidates": task.candidates, "user\_selection": task.user\_selection, "status": task.status, "turn\_type": task.turn\_type, "resume\_count": task.resume\_count, "last\_prompt": task.last\_prompt, "created\_at": task.created\_at.isoformat(), "updated\_at": task.updated\_at.isoformat(), }, ) def \_restore\_task(self, task\_id: str) -> Optional\[TaskContext\]: if not self.storage: return None latest = self.storage.read\_latest(task\_id) if not latest: return None partial\_state = latest.get("partial\_query\_state") task = TaskContext( task\_id=latest\["task\_id"\], session\_id=latest\["session\_id"\], raw\_query=latest\["raw\_query"\], user\_id=latest.get("user\_id"), extraction=latest.get("extraction", {}), partial\_query\_state=QueryState(\*\*partial\_state) if partial\_state else None, candidates=latest.get("candidates", {}), user\_selection=latest.get("user\_selection", {}), status=latest.get("status", "waiting\_confirmation"), turn\_type=latest.get("turn\_type", "confirmation"), resume\_count=latest.get("resume\_count", 0), last\_prompt=latest.get("last\_prompt"), created\_at=datetime.fromisoformat(latest\["created\_at"\]), updated\_at=datetime.fromisoformat(latest\["updated\_at"\]), ) self.\_tasks\[task\_id\] = task return task

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

"""任务管理器：独立于 Session，管理查询任务的创建/确认/过期"""

from \_\_future\_\_ import annotations

import logging

import uuid

from datetime import datetime

from typing import Any, Dict, Optional

from memory.storage.memory\_file import TaskStorage

from service.session\_models import TaskContext, QueryState

logger \= logging.getLogger(\_\_name\_\_)

class TaskManager:

"""任务管理器

管理查询任务的完整生命周期：创建 → 确认 → 完成 → 过期。

关键设计：

\- 与 Session 解耦：通过 task\_id 索引，不挂在 session 上

\- TTL 机制：默认 30 分钟过期

\- 幂等：同一 task\_id 多次 confirm 不会重复执行

"""

DEFAULT\_TTL\_MINUTES \= 30

def \_\_init\_\_(self, ttl\_minutes: int \= DEFAULT\_TTL\_MINUTES, storage: TaskStorage | None \= None):

self.\_tasks: Dict\[str, TaskContext\] \= {}

self.\_ttl\_minutes \= ttl\_minutes

self.storage \= storage

def create\_task(

self,

session\_id: str,

raw\_query: str,

extraction: Dict\[str, Any\],

user\_id: Optional\[str\] \= None,

candidates: Optional\[Dict\[str, list\[Dict\[str, Any\]\]\]\] \= None,

partial\_query\_state: Optional\[QueryState\] \= None,

) \-> TaskContext:

"""创建一个待确认/待执行任务

Args:

session\_id: 所属会话

raw\_query: 用户原始输入

extraction: LLM 提取结果

user\_id: 用户ID

candidates: 候选项 {"event": \[{"value": "x", "score": 0.9}\]}

partial\_query\_state: 正在构建中的查询状态

Returns:

TaskContext

"""

task\_id \= uuid.uuid4().hex\[:12\]

task \= TaskContext(

task\_id\=task\_id,

session\_id\=session\_id,

user\_id\=user\_id,

raw\_query\=raw\_query,

extraction\=extraction,

partial\_query\_state\=partial\_query\_state,

candidates\=candidates or {},

status\="waiting\_confirmation",

)

self.\_tasks\[task\_id\] \= task

self.\_persist\_task(task)

logger.info(f"Task created: {task\_id} (session={session\_id}, status=waiting\_confirmation)")

return task

def get\_task(self, task\_id: str) \-> Optional\[TaskContext\]:

"""读取任务

如果任务已过期，自动标记为 expired 并返回 None。

"""

task \= self.\_tasks.get(task\_id)

if not task:

restored \= self.\_restore\_task(task\_id)

if restored:

task \= restored

else:

return None

if task.is\_expired(self.\_ttl\_minutes):

task.status \= "expired"

self.\_persist\_task(task)

logger.info(f"Task expired: {task\_id}")

return None

return task

def confirm\_task(self, task\_id: str, field: str, value: str) \-> Optional\[TaskContext\]:

"""用户确认某个候选

幂等：重复 confirm 同一 field 不会报错，只更新值。

Args:

task\_id: 任务ID

field: 确认的字段（如 "event"）

value: 用户选择的值（如 "payment\_success"）

Returns:

更新后的 TaskContext，如果任务不存在或已过期则返回 None

"""

task \= self.get\_task(task\_id)

if not task:

logger.warning(f"Task not found or expired: {task\_id}")

return None

\# 1. 记录用户选择

task.user\_selection\[field\] \= value

\# 2. patch partial\_query\_state

if task.partial\_query\_state:

if field \== "event":

task.partial\_query\_state.event \= value

elif field \== "metric":

task.partial\_query\_state.metric \= value

elif field \== "region":

task.partial\_query\_state.region\_filter \= \[value\]

elif field \== "group\_by":

task.partial\_query\_state.group\_by \= value if isinstance(value, list) else \[value\]

elif field \== "chart\_type":

task.partial\_query\_state.chart\_type \= value

elif field \== "interaction\_mode":

task.partial\_query\_state.interaction\_mode \= value

\# 3. 检查是否所有需要确认的字段都已确认

all\_confirmed \= all(

field in task.user\_selection

for field in task.candidates.keys()

)

if all\_confirmed:

task.status \= "confirmed"

task.updated\_at \= datetime.now()

self.\_persist\_task(task)

logger.info(f"Task confirmed: {task\_id}, field={field}, value={value}, all\_confirmed={all\_confirmed}")

return task

def update\_status(self, task\_id: str, status: str) \-> Optional\[TaskContext\]:

"""更新任务状态

状态机：waiting\_confirmation → confirmed → completed

→ cancelled

→ expired（自动）

"""

task \= self.get\_task(task\_id)

if not task:

return None

valid\_transitions \= {

def cleanup\_expired(self) \-> int:

"""清理所有过期任务，返回清理数量"""

to\_remove \= \[\]

for task\_id, task in self.\_tasks.items():

if task.is\_expired(self.\_ttl\_minutes) or task.status \== "expired":

to\_remove.append(task\_id)

for task\_id in to\_remove:

del self.\_tasks\[task\_id\]

if self.storage:

self.storage.delete(task\_id)

if to\_remove:

logger.info(f"Cleaned up {len(to\_remove)} expired tasks")

return len(to\_remove)

def list\_tasks(self, session\_id: Optional\[str\] \= None) \-> list\[TaskContext\]:

"""列出任务，可按 session\_id 过滤"""

tasks \= list(self.\_tasks.values())

if session\_id:

tasks \= \[t for t in tasks if t.session\_id \== session\_id\]

return tasks

def \_persist\_task(self, task: TaskContext) \-> None:

if not self.storage:

return

self.storage.append(

task.task\_id,

{

"task\_id": task.task\_id,

"session\_id": task.session\_id,

"user\_id": task.user\_id,

"raw\_query": task.raw\_query,

"extraction": task.extraction,

"partial\_query\_state": task.partial\_query\_state.to\_dict() if task.partial\_query\_state else None,

"candidates": task.candidates,

"user\_selection": task.user\_selection,

"status": task.status,

"turn\_type": task.turn\_type,

"resume\_count": task.resume\_count,

"last\_prompt": task.last\_prompt,

"created\_at": task.created\_at.isoformat(),

"updated\_at": task.updated\_at.isoformat(),

},

)

def \_restore\_task(self, task\_id: str) \-> Optional\[TaskContext\]:

if not self.storage:

return None

latest \= self.storage.read\_latest(task\_id)

if not latest:

return None

partial\_state \= latest.get("partial\_query\_state")

task \= TaskContext(

task\_id\=latest\["task\_id"\],

session\_id\=latest\["session\_id"\],

raw\_query\=latest\["raw\_query"\],

user\_id\=latest.get("user\_id"),

extraction\=latest.get("extraction", {}),

partial\_query\_state\=QueryState(\*\*partial\_state) if partial\_state else None,

candidates\=latest.get("candidates", {}),

user\_selection\=latest.get("user\_selection", {}),

status\=latest.get("status", "waiting\_confirmation"),

turn\_type\=latest.get("turn\_type", "confirmation"),

resume\_count\=latest.get("resume\_count", 0),

last\_prompt\=latest.get("last\_prompt"),

created\_at\=datetime.fromisoformat(latest\["created\_at"\]),

updated\_at\=datetime.fromisoformat(latest\["updated\_at"\]),

)

self.\_tasks\[task\_id\] \= task

return task