# Source: https://github.com/frankzh0330/query-agent/blob/master/dispatcher/response_dispatcher.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# response\_dispatcher.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/dispatcher/response_dispatcher.py)

Open commit details

History

75 lines (60 loc) · 2.54 KB

## FilesExpand file tree

master

/

# response\_dispatcher.py

Copy path

Top

## File metadata and controls

- Code

- Blame

75 lines (60 loc) · 2.54 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/dispatcher/response_dispatcher.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import asyncio import json import logging from typing import Any, Dict, Optional from bus.message\_schema import BusResult logger = logging.getLogger(\_\_name\_\_) class ResponseDispatcher: """ 响应分派器：根据消息的 channel 字段路由结果到对应 Gateway。 职责： 1. 从结果队列取出 BusResult 2. 根据 msg.channel 查找对应 Gateway 3. 格式化响应并调用 gateway.send\_response() """ def \_\_init\_\_(self): self.\_gateways: Dict\[str, Any\] = {} # channel -> gateway instance def register(self, channel: str, gateway) -> None: """注册 Gateway 到指定 channel""" self.\_gateways\[channel\] = gateway logger.info(f"Dispatcher registered gateway for channel: {channel}") async def dispatch(self, result: BusResult) -> None: """分派结果到对应 Gateway""" msg = result.msg gateway = self.\_gateways.get(msg.channel) if not gateway: logger.error(f"No gateway registered for channel: {msg.channel}") return try: if result.success and result.nl2dsl\_result: # 检查 early\_exit if result.error and not result.query\_result: # early\_exit 场景：发送错误提示 await gateway.send\_response(msg.chat\_id, {"text": result.error}) return # 正常结果：让 Gateway 自己格式化 response\_text = gateway.format\_response( result.nl2dsl\_result, result.query\_result ) await gateway.send\_response(msg.chat\_id, {"text": response\_text}) else: error\_msg = f"处理失败: {result.error or '未知错误'}" await gateway.send\_response(msg.chat\_id, {"text": error\_msg}) except Exception as e: logger.exception(f"Dispatch error for msg={msg.msg\_id}: {e}") async def run\_loop(self, bus) -> None: """ 持续从 bus 取结果并分派（用于 Redis 模式） DirectCallBus 模式下不需要此方法。 """ logger.info("ResponseDispatcher started in queue mode") while True: try: result = await bus.dequeue\_result(timeout=30) if result is None: continue await self.dispatch(result) except Exception as e: logger.exception(f"Dispatcher loop error: {e}") await asyncio.sleep(1)

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

from \_\_future\_\_ import annotations

import asyncio

import json

import logging

from typing import Any, Dict, Optional

from bus.message\_schema import BusResult

logger \= logging.getLogger(\_\_name\_\_)

class ResponseDispatcher:

"""

响应分派器：根据消息的 channel 字段路由结果到对应 Gateway。

职责：

1\. 从结果队列取出 BusResult

2\. 根据 msg.channel 查找对应 Gateway

3\. 格式化响应并调用 gateway.send\_response()

"""

def \_\_init\_\_(self):

self.\_gateways: Dict\[str, Any\] \= {} \# channel -> gateway instance

def register(self, channel: str, gateway) \-> None:

"""注册 Gateway 到指定 channel"""

self.\_gateways\[channel\] \= gateway

logger.info(f"Dispatcher registered gateway for channel: {channel}")

async def dispatch(self, result: BusResult) \-> None:

"""分派结果到对应 Gateway"""

msg \= result.msg

gateway \= self.\_gateways.get(msg.channel)

if not gateway:

logger.error(f"No gateway registered for channel: {msg.channel}")

return

try:

if result.success and result.nl2dsl\_result:

\# 检查 early\_exit

if result.error and not result.query\_result:

\# early\_exit 场景：发送错误提示

await gateway.send\_response(msg.chat\_id, {"text": result.error})

return

\# 正常结果：让 Gateway 自己格式化

response\_text \= gateway.format\_response(

result.nl2dsl\_result, result.query\_result

)

await gateway.send\_response(msg.chat\_id, {"text": response\_text})

else:

error\_msg \= f"处理失败: {result.error or '未知错误'}"

await gateway.send\_response(msg.chat\_id, {"text": error\_msg})

except Exception as e:

logger.exception(f"Dispatch error for msg={msg.msg\_id}: {e}")

async def run\_loop(self, bus) \-> None:

"""

持续从 bus 取结果并分派（用于 Redis 模式）

DirectCallBus 模式下不需要此方法。

"""

logger.info("ResponseDispatcher started in queue mode")

while True:

try:

result \= await bus.dequeue\_result(timeout\=30)

if result is None:

continue

await self.dispatch(result)

except Exception as e:

logger.exception(f"Dispatcher loop error: {e}")

await asyncio.sleep(1)