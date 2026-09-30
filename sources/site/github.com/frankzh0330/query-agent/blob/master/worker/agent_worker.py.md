# Source: https://github.com/frankzh0330/query-agent/blob/master/worker/agent_worker.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# agent\_worker.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/worker/agent_worker.py)

Open commit details

History

107 lines (88 loc) · 3.82 KB

## FilesExpand file tree

master

/

# agent\_worker.py

Copy path

Top

## File metadata and controls

- Code

- Blame

107 lines (88 loc) · 3.82 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/worker/agent_worker.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import asyncio import logging from typing import Any, Dict, Optional from bus.message\_schema import BusMessage logger = logging.getLogger(\_\_name\_\_) class AgentWorker: """ Agent Worker：从请求队列取出消息 → 执行 nl2dsl + execute\_query → 返回结果。 设计要点： - 同步的 extract\_llm() 通过 asyncio.to\_thread() 包装，避免阻塞事件循环 - 无状态处理，可横向扩展为多个 Worker 实例 """ async def process(self, msg: BusMessage) -> Dict\[str, Any\]: """ 处理单条消息 Args: msg: 统一消息协议 Returns: {"nl2dsl": result.model\_dump(), "query": query\_result} """ from app import NL2DSLRequest, get\_matcher\_service, orchestrator, \_send\_telegram\_notification from service.bearer\_service import execute\_query logger.info(f"Worker processing: msg={msg.msg\_id}, channel={msg.channel}, text={msg.text\[:60\]}") # 用 chat\_id 作为 session\_id，同一聊天的消息共享会话上下文 req = NL2DSLRequest( text=msg.text, user\_id=msg.user\_id, project\_id=msg.project\_id, chat\_id=msg.chat\_id, session\_id=f"tg\_{msg.chat\_id}", ) service = get\_matcher\_service() result = await orchestrator.process(req, service, service.catalog, notify\_fn=\_send\_telegram\_notification) logger.debug("Worker nl2dsl done: status=%s, session=%s", result.get("status"), result.get("session\_id")) # early\_exit 场景（缺少 event\_name 等） if result.get("status") == "early\_exit": logger.info("Worker early\_exit: msg=%s, reason=%s", msg.msg\_id, result.get("message")) return { "nl2dsl": result, "query": None, "early\_exit": True, "message": result.get("message"), } # 执行 Bearer 查询 try: query\_result = await execute\_query(result\["exec\_dsl"\]) logger.info("Worker query done: msg=%s, success=True", msg.msg\_id) except Exception as e: logger.exception("Bearer query failed for msg=%s: %s", msg.msg\_id, e) query\_result = {"success": False, "error": str(e)} return { "nl2dsl": result, "query": query\_result, "early\_exit": False, } async def run\_loop(self, bus) -> None: """ 持续从 bus 取消息并处理（用于 Redis 模式） DirectCallBus 模式下不需要此方法。 """ logger.info("AgentWorker started in queue mode") while True: try: msg = await bus.dequeue\_request(timeout=30) if msg is None: continue logger.info( "\[Worker\] ← Dequeued msg\_id=%s user=%s chat=%s text=%.80s", msg.msg\_id, msg.user\_id, msg.chat\_id, msg.text, ) result = await self.process(msg) logger.info( "\[Worker\] → Enqueuing result msg\_id=%s early\_exit=%s has\_query=%s", msg.msg\_id, result.get("early\_exit"), result.get("query") is not None, ) from bus.message\_schema import BusResult bus\_result = BusResult( msg=msg, success=True, nl2dsl\_result=result.get("nl2dsl"), query\_result=result.get("query"), error=result.get("message") if result.get("early\_exit") else None, ) await bus.enqueue\_result(bus\_result) except Exception as e: logger.exception(f"Worker loop error: {e}") await asyncio.sleep(1)

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

from \_\_future\_\_ import annotations

import asyncio

import logging

from typing import Any, Dict, Optional

from bus.message\_schema import BusMessage

logger \= logging.getLogger(\_\_name\_\_)

class AgentWorker:

"""

Agent Worker：从请求队列取出消息 → 执行 nl2dsl + execute\_query → 返回结果。

设计要点：

\- 同步的 extract\_llm() 通过 asyncio.to\_thread() 包装，避免阻塞事件循环

\- 无状态处理，可横向扩展为多个 Worker 实例

"""

async def process(self, msg: BusMessage) \-> Dict\[str, Any\]:

"""

处理单条消息

Args:

msg: 统一消息协议

Returns:

{"nl2dsl": result.model\_dump(), "query": query\_result}

"""

from app import NL2DSLRequest, get\_matcher\_service, orchestrator, \_send\_telegram\_notification

from service.bearer\_service import execute\_query

logger.info(f"Worker processing: msg={msg.msg\_id}, channel={msg.channel}, text={msg.text\[:60\]}")

\# 用 chat\_id 作为 session\_id，同一聊天的消息共享会话上下文

req \= NL2DSLRequest(

text\=msg.text,

user\_id\=msg.user\_id,

project\_id\=msg.project\_id,

chat\_id\=msg.chat\_id,

session\_id\=f"tg\_{msg.chat\_id}",

)

service \= get\_matcher\_service()

result \= await orchestrator.process(req, service, service.catalog, notify\_fn\=\_send\_telegram\_notification)

logger.debug("Worker nl2dsl done: status=%s, session=%s", result.get("status"), result.get("session\_id"))

\# early\_exit 场景（缺少 event\_name 等）

if result.get("status") \== "early\_exit":

logger.info("Worker early\_exit: msg=%s, reason=%s", msg.msg\_id, result.get("message"))

return {

"nl2dsl": result,

"query": None,

"early\_exit": True,

"message": result.get("message"),

}

\# 执行 Bearer 查询

try:

query\_result \= await execute\_query(result\["exec\_dsl"\])

logger.info("Worker query done: msg=%s, success=True", msg.msg\_id)

except Exception as e:

logger.exception("Bearer query failed for msg=%s: %s", msg.msg\_id, e)

query\_result \= {"success": False, "error": str(e)}

return {

"nl2dsl": result,

"query": query\_result,

"early\_exit": False,

}

async def run\_loop(self, bus) \-> None:

"""

持续从 bus 取消息并处理（用于 Redis 模式）

DirectCallBus 模式下不需要此方法。

"""

logger.info("AgentWorker started in queue mode")

while True:

try:

msg \= await bus.dequeue\_request(timeout\=30)

if msg is None:

continue

logger.info(

"\[Worker\] ← Dequeued msg\_id=%s user=%s chat=%s text=%.80s",

msg.msg\_id, msg.user\_id, msg.chat\_id, msg.text,

)

result \= await self.process(msg)

logger.info(

"\[Worker\] → Enqueuing result msg\_id=%s early\_exit=%s has\_query=%s",

msg.msg\_id, result.get("early\_exit"), result.get("query") is not None,

)

from bus.message\_schema import BusResult

bus\_result \= BusResult(

msg\=msg,

success\=True,

nl2dsl\_result\=result.get("nl2dsl"),

query\_result\=result.get("query"),

error\=result.get("message") if result.get("early\_exit") else None,

)

await bus.enqueue\_result(bus\_result)

except Exception as e:

logger.exception(f"Worker loop error: {e}")

await asyncio.sleep(1)