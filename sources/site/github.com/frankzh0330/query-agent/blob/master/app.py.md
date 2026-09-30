# Source: https://github.com/frankzh0330/query-agent/blob/master/app.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# app.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/app.py)

Open commit details

History

148 lines (114 loc) · 4.49 KB

## FilesExpand file tree

master

/

# app.py

Copy path

Top

## File metadata and controls

- Code

- Blame

148 lines (114 loc) · 4.49 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/app.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Query Agent API — FastAPI 端点定义 app.py 只负责： 1. FastAPI 实例 + 请求/响应模型 2. 全局服务实例化 3. HTTP 端点（委托给 QueryOrchestrator） 4. Telegram 通知（依赖环境变量） """ from \_\_future\_\_ import annotations import logging import os from typing import Any, Dict, Optional import httpx from fastapi import FastAPI, HTTPException from pydantic import BaseModel, Field from service.bearer\_service import execute\_query from service.session\_manager import SessionManager from service.session\_models import QueryState from service.task\_manager import TaskManager from memory.memory\_writer import MemoryWriter from memory.storage.memory\_file import TaskStorage from memory.user\_preference\_store import UserPreferenceStore from service.query\_orchestrator import QueryOrchestrator app = FastAPI(title="query-agent: NL to Semantic DSL") logger = logging.getLogger(\_\_name\_\_) # ==================== # 全局服务实例 # ==================== \_matcher\_service: Optional\["MatcherService"\] = None session\_manager = SessionManager(data\_path="data") task\_manager = TaskManager(storage=TaskStorage(data\_path="data/tasks")) memory\_writer = MemoryWriter(data\_path="data/memory") user\_preference\_store = UserPreferenceStore(data\_path="data/user\_preferences") orchestrator = QueryOrchestrator(session\_manager, task\_manager, memory\_writer, user\_preference\_store) def set\_matcher\_service(service: "MatcherService") -> None: global \_matcher\_service \_matcher\_service = service logger.info("MatcherService registered") def get\_matcher\_service() -> "MatcherService": if \_matcher\_service is None: raise RuntimeError("MatcherService not initialized. Start WebSocket server first.") return \_matcher\_service # ==================== # 请求/响应模型 # ==================== class NL2DSLRequest(BaseModel): text: str project\_id: int = Field(default=55) session\_id: str | None = None user\_id: str | None = None chat\_id: str | None = None class NL2DSLResponse(BaseModel): extraction\_json: Dict\[str, Any\] semantic: Dict\[str, Any\] exec\_dsl: Dict\[str, Any\] explain: Dict\[str, Any\] session\_id: str | None = None status: str = "success" message: str | None = None task\_id: str | None = None candidates: Dict\[str, Any\] | None = None class BearerQueryRequest(BaseModel): exec\_dsl: Dict\[str, Any\] class BearerQueryResponse(BaseModel): result: Dict\[str, Any\] success: bool error: Optional\[str\] = None # ==================== # Telegram 通知 # ==================== async def \_send\_telegram\_notification(chat\_id: str, message: str) -> None: bot\_token = os.getenv("TELEGRAM\_BOT\_TOKEN", "") if not bot\_token: return url = f"https://api.telegram.org/bot{bot\_token}/sendMessage" payload = {"chat\_id": chat\_id, "text": message, "parse\_mode": "Markdown"} try: async with httpx.AsyncClient() as client: await client.post(url, json=payload, timeout=5) except Exception as e: logger.warning("Failed to send Telegram notification: %s", e) # ==================== # HTTP 端点 # ==================== @app.post("/nl2dsl", response\_model=NL2DSLResponse) async def nl2dsl(req: NL2DSLRequest) -> NL2DSLResponse: """NL 转 DSL 接口""" service = get\_matcher\_service() catalog = service.catalog result = await orchestrator.process(req, service, catalog, notify\_fn=\_send\_telegram\_notification) return NL2DSLResponse(\*\*result) @app.post("/query/bearer", response\_model=BearerQueryResponse) async def query\_bearer(req: BearerQueryRequest): """执行 Bearer 查询接口""" try: result = await execute\_query(req.exec\_dsl) return BearerQueryResponse(result=result, success=True) except Exception as e: logger.error("Bearer query failed: %s", e) return BearerQueryResponse(result={}, success=False, error=str(e)) @app.get("/sessions") def list\_sessions(): sessions = session\_manager.list\_sessions() return {"count": len(sessions), "sessions": \[s.to\_dict() for s in sessions\]} @app.get("/sessions/{session\_id}") def get\_session(session\_id: str): ctx = session\_manager.get\_session(session\_id) if not ctx: raise HTTPException(status\_code=404, detail="Session not found") return ctx.to\_dict() @app.delete("/sessions/{session\_id}") def delete\_session(session\_id: str): if not session\_manager.delete\_session(session\_id): raise HTTPException(status\_code=404, detail="Session not found") return {"deleted": session\_id}

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

"""Query Agent API — FastAPI 端点定义

app.py 只负责：

1\. FastAPI 实例 + 请求/响应模型

2\. 全局服务实例化

3\. HTTP 端点（委托给 QueryOrchestrator）

4\. Telegram 通知（依赖环境变量）

"""

from \_\_future\_\_ import annotations

import logging

import os

from typing import Any, Dict, Optional

import httpx

from fastapi import FastAPI, HTTPException

from pydantic import BaseModel, Field

from service.bearer\_service import execute\_query

from service.session\_manager import SessionManager

from service.session\_models import QueryState

from service.task\_manager import TaskManager

from memory.memory\_writer import MemoryWriter

from memory.storage.memory\_file import TaskStorage

from memory.user\_preference\_store import UserPreferenceStore

from service.query\_orchestrator import QueryOrchestrator

app \= FastAPI(title\="query-agent: NL to Semantic DSL")

logger \= logging.getLogger(\_\_name\_\_)

\# ====================

\# 全局服务实例

\# ====================

\_matcher\_service: Optional\["MatcherService"\] \= None

session\_manager \= SessionManager(data\_path\="data")

task\_manager \= TaskManager(storage\=TaskStorage(data\_path\="data/tasks"))

memory\_writer \= MemoryWriter(data\_path\="data/memory")

user\_preference\_store \= UserPreferenceStore(data\_path\="data/user\_preferences")

orchestrator \= QueryOrchestrator(session\_manager, task\_manager, memory\_writer, user\_preference\_store)

def set\_matcher\_service(service: "MatcherService") \-> None:

global \_matcher\_service

\_matcher\_service \= service

logger.info("MatcherService registered")

def get\_matcher\_service() \-> "MatcherService":

if \_matcher\_service is None:

raise RuntimeError("MatcherService not initialized. Start WebSocket server first.")

return \_matcher\_service

\# ====================

\# 请求/响应模型

\# ====================

class NL2DSLRequest(BaseModel):

text: str

project\_id: int \= Field(default\=55)

session\_id: str | None \= None

user\_id: str | None \= None

chat\_id: str | None \= None

class NL2DSLResponse(BaseModel):

extraction\_json: Dict\[str, Any\]

semantic: Dict\[str, Any\]

exec\_dsl: Dict\[str, Any\]

explain: Dict\[str, Any\]

session\_id: str | None \= None

status: str \= "success"

message: str | None \= None

task\_id: str | None \= None

candidates: Dict\[str, Any\] | None \= None

class BearerQueryRequest(BaseModel):

exec\_dsl: Dict\[str, Any\]

class BearerQueryResponse(BaseModel):

result: Dict\[str, Any\]

success: bool

error: Optional\[str\] \= None

\# ====================

\# Telegram 通知

\# ====================

async def \_send\_telegram\_notification(chat\_id: str, message: str) \-> None:

bot\_token \= os.getenv("TELEGRAM\_BOT\_TOKEN", "")

if not bot\_token:

return

url \= f"https://api.telegram.org/bot{bot\_token}/sendMessage"

payload \= {"chat\_id": chat\_id, "text": message, "parse\_mode": "Markdown"}

try:

async with httpx.AsyncClient() as client:

await client.post(url, json\=payload, timeout\=5)

except Exception as e:

logger.warning("Failed to send Telegram notification: %s", e)

\# ====================

\# HTTP 端点

\# ====================

@app.post("/nl2dsl", response\_model\=NL2DSLResponse)

async def nl2dsl(req: NL2DSLRequest) \-> NL2DSLResponse:

"""NL 转 DSL 接口"""

service \= get\_matcher\_service()

catalog \= service.catalog

result \= await orchestrator.process(req, service, catalog, notify\_fn\=\_send\_telegram\_notification)

return NL2DSLResponse(\*\*result)

@app.post("/query/bearer", response\_model\=BearerQueryResponse)

async def query\_bearer(req: BearerQueryRequest):

"""执行 Bearer 查询接口"""

try:

result \= await execute\_query(req.exec\_dsl)

return BearerQueryResponse(result\=result, success\=True)

except Exception as e:

logger.error("Bearer query failed: %s", e)

return BearerQueryResponse(result\={}, success\=False, error\=str(e))

@app.get("/sessions")

def list\_sessions():

sessions \= session\_manager.list\_sessions()

return {"count": len(sessions), "sessions": \[s.to\_dict() for s in sessions\]}

@app.get("/sessions/{session\_id}")

def get\_session(session\_id: str):

ctx \= session\_manager.get\_session(session\_id)

if not ctx:

raise HTTPException(status\_code\=404, detail\="Session not found")

return ctx.to\_dict()

@app.delete("/sessions/{session\_id}")

def delete\_session(session\_id: str):

if not session\_manager.delete\_session(session\_id):

raise HTTPException(status\_code\=404, detail\="Session not found")

return {"deleted": session\_id}