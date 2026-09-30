# Source: https://github.com/frankzh0330/query-agent/blob/master/ingress/base_adapter.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# base\_adapter.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/ingress/base_adapter.py)

Open commit details

History

23 lines (16 loc) · 535 Bytes

## FilesExpand file tree

master

/

# base\_adapter.py

Copy path

Top

## File metadata and controls

- Code

- Blame

23 lines (16 loc) · 535 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/ingress/base_adapter.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Ingress Adapter 基类""" from abc import ABC, abstractmethod from typing import Any from ingress.models import StandardMessage class BaseIngressAdapter(ABC): """Ingress Adapter 基类 所有渠道适配器都应继承该基类 """ @abstractmethod def adapt(self, raw\_message: Any) -> StandardMessage: """将原始消息转换为标准格式""" pass @abstractmethod def extract\_message\_id(self, raw\_message: Any) -> str: """提取消息ID（用于去重）""" pass

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

"""Ingress Adapter 基类"""

from abc import ABC, abstractmethod

from typing import Any

from ingress.models import StandardMessage

class BaseIngressAdapter(ABC):

"""Ingress Adapter 基类

所有渠道适配器都应继承该基类

"""

@abstractmethod

def adapt(self, raw\_message: Any) \-> StandardMessage:

"""将原始消息转换为标准格式"""

pass

@abstractmethod

def extract\_message\_id(self, raw\_message: Any) \-> str:

"""提取消息ID（用于去重）"""

pass