# Source: https://github.com/frankzh0330/query-agent/blob/master/bus/__init__.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# \_\_init\_\_.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/bus/__init__.py)

Open commit details

History

31 lines (23 loc) · 798 Bytes

## FilesExpand file tree

master

/

# \_\_init\_\_.py

Copy path

Top

## File metadata and controls

- Code

- Blame

31 lines (23 loc) · 798 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/bus/__init__.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import os from bus.base import MessageBus from bus.message\_schema import BusMessage, BusResult def create\_bus() -> MessageBus: """ 工厂函数：根据 MESSAGE\_BUS\_BACKEND 环境变量创建消息总线 - "direct"（默认）: DirectCallBus，内存直连，适用于开发和单进程部署 - "redis": RedisMessageBus，通过 Redis LIST 队列，适用于多 Worker 部署 """ backend = os.getenv("MESSAGE\_BUS\_BACKEND", "direct").lower() if backend == "redis": from bus.redis\_bus import RedisMessageBus return RedisMessageBus() else: from bus.direct\_call\_bus import DirectCallBus return DirectCallBus() \_\_all\_\_ = \[ "MessageBus", "BusMessage", "BusResult", "create\_bus", \]

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

from \_\_future\_\_ import annotations

import os

from bus.base import MessageBus

from bus.message\_schema import BusMessage, BusResult

def create\_bus() \-> MessageBus:

"""

工厂函数：根据 MESSAGE\_BUS\_BACKEND 环境变量创建消息总线

\- "direct"（默认）: DirectCallBus，内存直连，适用于开发和单进程部署

\- "redis": RedisMessageBus，通过 Redis LIST 队列，适用于多 Worker 部署

"""

backend \= os.getenv("MESSAGE\_BUS\_BACKEND", "direct").lower()

if backend \== "redis":

from bus.redis\_bus import RedisMessageBus

return RedisMessageBus()

else:

from bus.direct\_call\_bus import DirectCallBus

return DirectCallBus()

\_\_all\_\_ \= \[

"MessageBus",

"BusMessage",

"BusResult",

"create\_bus",

\]