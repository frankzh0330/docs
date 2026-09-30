# Source: https://github.com/frankzh0330/query-agent/blob/master/bus/direct_call_bus.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# direct\_call\_bus.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[fix: improve early\_exit handling, logging config, and tool calling fl…](https://github.com/frankzh0330/query-agent/commit/874b818935aa4d48f0c46b5a83ab4b31069dcdfa)

Open commit details

Apr 15, 2026

[874b818](https://github.com/frankzh0330/query-agent/commit/874b818935aa4d48f0c46b5a83ab4b31069dcdfa) · Apr 15, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/bus/direct_call_bus.py)

Open commit details

History

72 lines (56 loc) · 2.43 KB

## FilesExpand file tree

master

/

# direct\_call\_bus.py

Copy path

Top

## File metadata and controls

- Code

- Blame

72 lines (56 loc) · 2.43 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/bus/direct_call_bus.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import asyncio import logging from typing import Optional from bus.base import MessageBus from bus.message\_schema import BusMessage, BusResult logger = logging.getLogger(\_\_name\_\_) class DirectCallBus(MessageBus): """ 内存直连实现：enqueue\_request 直接调用 Worker 处理。 不经过队列，用于开发和单进程部署。 行为等价于重构前的直接函数调用。 """ def \_\_init\_\_(self): self.\_worker = None # 延迟绑定 self.\_dispatcher = None # 延迟绑定 def bind\_worker(self, worker) -> None: """绑定 AgentWorker""" self.\_worker = worker def bind\_dispatcher(self, dispatcher) -> None: """绑定 ResponseDispatcher""" self.\_dispatcher = dispatcher async def enqueue\_request(self, msg: BusMessage) -> None: """直接调用 worker 处理，结果交给 dispatcher""" if not self.\_worker: raise RuntimeError("Worker not bound to DirectCallBus") logger.info(f"DirectCallBus: processing msg={msg.msg\_id} from {msg.channel}") try: result = await self.\_worker.process(msg) bus\_result = BusResult( msg=msg, success=True, nl2dsl\_result=result.get("nl2dsl"), query\_result=result.get("query"), error=result.get("message") if result.get("early\_exit") else None, ) except Exception as e: logger.exception(f"Worker error for msg={msg.msg\_id}: {e}") bus\_result = BusResult( msg=msg, success=False, error=str(e), ) # 直接分派结果 if self.\_dispatcher: await self.\_dispatcher.dispatch(bus\_result) async def dequeue\_request(self, timeout: float = 30) -> Optional\[BusMessage\]: """DirectCallBus 不支持 dequeue，消息直接处理""" raise NotImplementedError("DirectCallBus processes messages inline, no dequeue") async def enqueue\_result(self, result: BusResult) -> None: """DirectCallBus 结果直接分派，不走队列""" if self.\_dispatcher: await self.\_dispatcher.dispatch(result) async def dequeue\_result(self, timeout: float = 30) -> Optional\[BusResult\]: """DirectCallBus 不支持 dequeue""" raise NotImplementedError("DirectCallBus dispatches results inline, no dequeue")

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

from \_\_future\_\_ import annotations

import asyncio

import logging

from typing import Optional

from bus.base import MessageBus

from bus.message\_schema import BusMessage, BusResult

logger \= logging.getLogger(\_\_name\_\_)

class DirectCallBus(MessageBus):

"""

内存直连实现：enqueue\_request 直接调用 Worker 处理。

不经过队列，用于开发和单进程部署。

行为等价于重构前的直接函数调用。

"""

def \_\_init\_\_(self):

self.\_worker \= None \# 延迟绑定

self.\_dispatcher \= None \# 延迟绑定

def bind\_worker(self, worker) \-> None:

"""绑定 AgentWorker"""

self.\_worker \= worker

def bind\_dispatcher(self, dispatcher) \-> None:

"""绑定 ResponseDispatcher"""

self.\_dispatcher \= dispatcher

async def enqueue\_request(self, msg: BusMessage) \-> None:

"""直接调用 worker 处理，结果交给 dispatcher"""

if not self.\_worker:

raise RuntimeError("Worker not bound to DirectCallBus")

logger.info(f"DirectCallBus: processing msg={msg.msg\_id} from {msg.channel}")

try:

result \= await self.\_worker.process(msg)

bus\_result \= BusResult(

msg\=msg,

success\=True,

nl2dsl\_result\=result.get("nl2dsl"),

query\_result\=result.get("query"),

error\=result.get("message") if result.get("early\_exit") else None,

)

except Exception as e:

logger.exception(f"Worker error for msg={msg.msg\_id}: {e}")

bus\_result \= BusResult(

msg\=msg,

success\=False,

error\=str(e),

)

\# 直接分派结果

if self.\_dispatcher:

await self.\_dispatcher.dispatch(bus\_result)

async def dequeue\_request(self, timeout: float \= 30) \-> Optional\[BusMessage\]:

"""DirectCallBus 不支持 dequeue，消息直接处理"""

raise NotImplementedError("DirectCallBus processes messages inline, no dequeue")

async def enqueue\_result(self, result: BusResult) \-> None:

"""DirectCallBus 结果直接分派，不走队列"""

if self.\_dispatcher:

await self.\_dispatcher.dispatch(result)

async def dequeue\_result(self, timeout: float \= 30) \-> Optional\[BusResult\]:

"""DirectCallBus 不支持 dequeue"""

raise NotImplementedError("DirectCallBus dispatches results inline, no dequeue")