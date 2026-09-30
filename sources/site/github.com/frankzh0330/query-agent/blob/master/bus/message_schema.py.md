# Source: https://github.com/frankzh0330/query-agent/blob/master/bus/message_schema.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# message\_schema.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/bus/message_schema.py)

Open commit details

History

29 lines (21 loc) · 789 Bytes

## FilesExpand file tree

master

/

# message\_schema.py

Copy path

Top

## File metadata and controls

- Code

- Blame

29 lines (21 loc) · 789 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/bus/message_schema.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import time import uuid from typing import Any, Dict, Optional from pydantic import BaseModel, Field class BusMessage(BaseModel): """统一消息协议，用于 Gateway ↔ Worker 之间的通信""" msg\_id: str = Field(default\_factory=lambda: uuid.uuid4().hex) channel: str # "telegram" | "whatsapp" | "lark" | ... chat\_id: str # 回复目标（Telegram chat\_id 等） user\_id: Optional\[str\] = None text: str project\_id: int = 55 created\_at: float = Field(default\_factory=time.time) class BusResult(BaseModel): """Worker 处理结果""" msg: BusMessage success: bool nl2dsl\_result: Optional\[Dict\[str, Any\]\] = None query\_result: Optional\[Dict\[str, Any\]\] = None error: Optional\[str\] = None

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

from \_\_future\_\_ import annotations

import time

import uuid

from typing import Any, Dict, Optional

from pydantic import BaseModel, Field

class BusMessage(BaseModel):

"""统一消息协议，用于 Gateway ↔ Worker 之间的通信"""

msg\_id: str \= Field(default\_factory\=lambda: uuid.uuid4().hex)

channel: str \# "telegram" | "whatsapp" | "lark" | ...

chat\_id: str \# 回复目标（Telegram chat\_id 等）

user\_id: Optional\[str\] \= None

text: str

project\_id: int \= 55

created\_at: float \= Field(default\_factory\=time.time)

class BusResult(BaseModel):

"""Worker 处理结果"""

msg: BusMessage

success: bool

nl2dsl\_result: Optional\[Dict\[str, Any\]\] \= None

query\_result: Optional\[Dict\[str, Any\]\] \= None

error: Optional\[str\] \= None