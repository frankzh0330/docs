# Source: https://github.com/frankzh0330/query-agent/blob/master/bus/base.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# base.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/bus/base.py)

Open commit details

History

30 lines (22 loc) · 861 Bytes

## FilesExpand file tree

master

/

# base.py

Copy path

Top

## File metadata and controls

- Code

- Blame

30 lines (22 loc) · 861 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/bus/base.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from abc import ABC, abstractmethod from typing import Any, Dict, Optional, Tuple from bus.message\_schema import BusMessage, BusResult class MessageBus(ABC): """消息总线抽象基类""" @abstractmethod async def enqueue\_request(self, msg: BusMessage) -> None: """将消息放入请求队列""" ... @abstractmethod async def dequeue\_request(self, timeout: float = 30) -> Optional\[BusMessage\]: """从请求队列取出消息（阻塞等待）""" ... @abstractmethod async def enqueue\_result(self, result: BusResult) -> None: """将处理结果放入结果队列""" ... @abstractmethod async def dequeue\_result(self, timeout: float = 30) -> Optional\[BusResult\]: """从结果队列取出结果（阻塞等待）""" ...

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

from \_\_future\_\_ import annotations

from abc import ABC, abstractmethod

from typing import Any, Dict, Optional, Tuple

from bus.message\_schema import BusMessage, BusResult

class MessageBus(ABC):

"""消息总线抽象基类"""

@abstractmethod

async def enqueue\_request(self, msg: BusMessage) \-> None:

"""将消息放入请求队列"""

...

@abstractmethod

async def dequeue\_request(self, timeout: float \= 30) \-> Optional\[BusMessage\]:

"""从请求队列取出消息（阻塞等待）"""

...

@abstractmethod

async def enqueue\_result(self, result: BusResult) \-> None:

"""将处理结果放入结果队列"""

...

@abstractmethod

async def dequeue\_result(self, timeout: float \= 30) \-> Optional\[BusResult\]:

"""从结果队列取出结果（阻塞等待）"""

...