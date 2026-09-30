# Source: https://github.com/frankzh0330/query-agent/blob/master/gateway/base.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/gateway/base.py)

Open commit details

History

41 lines (32 loc) · 1.08 KB

## FilesExpand file tree

master

/

# base.py

Copy path

Top

## File metadata and controls

- Code

- Blame

41 lines (32 loc) · 1.08 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/gateway/base.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from abc import ABC, abstractmethod from typing import Any, Dict, Optional class BaseGateway(ABC): """Gateway 基类""" def \_\_init\_\_(self, channel: str, bus=None): """ Args: channel: 渠道标识，如 "telegram", "whatsapp" bus: MessageBus 实例，用于消息入队 """ self.channel = channel self.bus = bus @abstractmethod async def start(self) -> None: """启动 Gateway""" pass @abstractmethod async def stop(self) -> None: """停止 Gateway""" pass @abstractmethod async def handle\_message(self, message: Any) -> None: """处理消息（Ingress 清洗 + 入队）""" pass @abstractmethod async def send\_response(self, recipient: str, response: Dict\[str, Any\]) -> None: """发送响应""" pass def format\_response(self, nl2dsl\_result: Dict, query\_result: Any = None) -> str: """格式化响应文本（子类可覆盖以适配不同渠道）""" return str(nl2dsl\_result)

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

from \_\_future\_\_ import annotations

from abc import ABC, abstractmethod

from typing import Any, Dict, Optional

class BaseGateway(ABC):

"""Gateway 基类"""

def \_\_init\_\_(self, channel: str, bus\=None):

"""

Args:

channel: 渠道标识，如 "telegram", "whatsapp"

bus: MessageBus 实例，用于消息入队

"""

self.channel \= channel

self.bus \= bus

@abstractmethod

async def start(self) \-> None:

"""启动 Gateway"""

pass

@abstractmethod

async def stop(self) \-> None:

"""停止 Gateway"""

pass

@abstractmethod

async def handle\_message(self, message: Any) \-> None:

"""处理消息（Ingress 清洗 + 入队）"""

pass

@abstractmethod

async def send\_response(self, recipient: str, response: Dict\[str, Any\]) \-> None:

"""发送响应"""

pass

def format\_response(self, nl2dsl\_result: Dict, query\_result: Any \= None) \-> str:

"""格式化响应文本（子类可覆盖以适配不同渠道）"""

return str(nl2dsl\_result)