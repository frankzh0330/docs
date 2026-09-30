# Source: https://github.com/frankzh0330/query-agent/blob/master/bus/redis_bus.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# redis\_bus.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add message bus, async worker and ingress layer for multi-gatew…](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1)

Open commit details

Apr 14, 2026

[ec3eb43](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1) · Apr 14, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/bus/redis_bus.py)

Open commit details

History

73 lines (57 loc) · 2.54 KB

## FilesExpand file tree

master

/

# redis\_bus.py

Copy path

Top

## File metadata and controls

- Code

- Blame

73 lines (57 loc) · 2.54 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/bus/redis_bus.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import logging import os from typing import Optional import redis.asyncio as aioredis from bus.base import MessageBus from bus.message\_schema import BusMessage, BusResult logger = logging.getLogger(\_\_name\_\_) REQUEST\_QUEUE = "query\_agent:request\_queue" RESULT\_QUEUE = "query\_agent:result\_queue" class RedisMessageBus(MessageBus): """ Redis LIST 实现的消息总线。 - 请求队列：query\_agent:request\_queue（所有 Gateway 共用） - 结果队列：query\_agent:result\_queue（所有 Worker 共用） 使用 BLPOP 实现阻塞式消费，支持多 Worker 并行。 """ def \_\_init\_\_(self, redis\_url: Optional\[str\] = None): self.\_redis\_url = redis\_url or os.getenv("REDIS\_URL", "redis://localhost:6379/0") self.\_redis: Optional\[aioredis.Redis\] = None async def \_get\_redis(self) -> aioredis.Redis: """延迟创建 Redis 连接""" if self.\_redis is None: self.\_redis = aioredis.from\_url(self.\_redis\_url, decode\_responses=True) logger.info(f"Redis connected: {self.\_redis\_url}") return self.\_redis async def enqueue\_request(self, msg: BusMessage) -> None: """将消息放入请求队列""" r = await self.\_get\_redis() await r.rpush(REQUEST\_QUEUE, msg.model\_dump\_json()) logger.info(f"Enqueued request: msg\_id={msg.msg\_id} channel={msg.channel}") async def dequeue\_request(self, timeout: float = 30) -> Optional\[BusMessage\]: """从请求队列阻塞取出消息""" r = await self.\_get\_redis() result = await r.blpop(REQUEST\_QUEUE, timeout=int(timeout)) if result is None: return None \_, data = result return BusMessage.model\_validate\_json(data) async def enqueue\_result(self, result: BusResult) -> None: """将结果放入结果队列""" r = await self.\_get\_redis() await r.rpush(RESULT\_QUEUE, result.model\_dump\_json()) logger.info(f"Enqueued result: msg\_id={result.msg.msg\_id}") async def dequeue\_result(self, timeout: float = 30) -> Optional\[BusResult\]: """从结果队列阻塞取出结果""" r = await self.\_get\_redis() result = await r.blpop(RESULT\_QUEUE, timeout=int(timeout)) if result is None: return None \_, data = result return BusResult.model\_validate\_json(data) async def close(self) -> None: """关闭 Redis 连接""" if self.\_redis: await self.\_redis.close() logger.info("Redis connection closed")

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

from \_\_future\_\_ import annotations

import logging

import os

from typing import Optional

import redis.asyncio as aioredis

from bus.base import MessageBus

from bus.message\_schema import BusMessage, BusResult

logger \= logging.getLogger(\_\_name\_\_)

REQUEST\_QUEUE \= "query\_agent:request\_queue"

RESULT\_QUEUE \= "query\_agent:result\_queue"

class RedisMessageBus(MessageBus):

"""

Redis LIST 实现的消息总线。

\- 请求队列：query\_agent:request\_queue（所有 Gateway 共用）

\- 结果队列：query\_agent:result\_queue（所有 Worker 共用）

使用 BLPOP 实现阻塞式消费，支持多 Worker 并行。

"""

def \_\_init\_\_(self, redis\_url: Optional\[str\] \= None):

self.\_redis\_url \= redis\_url or os.getenv("REDIS\_URL", "redis://localhost:6379/0")

self.\_redis: Optional\[aioredis.Redis\] \= None

async def \_get\_redis(self) \-> aioredis.Redis:

"""延迟创建 Redis 连接"""

if self.\_redis is None:

self.\_redis \= aioredis.from\_url(self.\_redis\_url, decode\_responses\=True)

logger.info(f"Redis connected: {self.\_redis\_url}")

return self.\_redis

async def enqueue\_request(self, msg: BusMessage) \-> None:

"""将消息放入请求队列"""

r \= await self.\_get\_redis()

await r.rpush(REQUEST\_QUEUE, msg.model\_dump\_json())

logger.info(f"Enqueued request: msg\_id={msg.msg\_id} channel={msg.channel}")

async def dequeue\_request(self, timeout: float \= 30) \-> Optional\[BusMessage\]:

"""从请求队列阻塞取出消息"""

r \= await self.\_get\_redis()

result \= await r.blpop(REQUEST\_QUEUE, timeout\=int(timeout))

if result is None:

return None

\_, data \= result

return BusMessage.model\_validate\_json(data)

async def enqueue\_result(self, result: BusResult) \-> None:

"""将结果放入结果队列"""

r \= await self.\_get\_redis()

await r.rpush(RESULT\_QUEUE, result.model\_dump\_json())

logger.info(f"Enqueued result: msg\_id={result.msg.msg\_id}")

async def dequeue\_result(self, timeout: float \= 30) \-> Optional\[BusResult\]:

"""从结果队列阻塞取出结果"""

r \= await self.\_get\_redis()

result \= await r.blpop(RESULT\_QUEUE, timeout\=int(timeout))

if result is None:

return None

\_, data \= result

return BusResult.model\_validate\_json(data)

async def close(self) \-> None:

"""关闭 Redis 连接"""

if self.\_redis:

await self.\_redis.close()

logger.info("Redis connection closed")