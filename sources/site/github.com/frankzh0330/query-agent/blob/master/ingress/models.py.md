# Source: https://github.com/frankzh0330/query-agent/blob/master/ingress/models.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# models.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/ingress/models.py)

Open commit details

History

43 lines (36 loc) · 1.31 KB

## FilesExpand file tree

master

/

# models.py

Copy path

Top

## File metadata and controls

- Code

- Blame

43 lines (36 loc) · 1.31 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/ingress/models.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Ingress 数据模型""" from dataclasses import dataclass, field from datetime import datetime from typing import Any, Dict @dataclass class StandardMessage: """标准化消息格式 所有外部消息（Telegram, WhatsApp 等）都转换为该格式 """ # 基础信息 message\_id: str # 消息唯一标识 (用于去重) user\_id: str # 用户ID chat\_id: str # 聊天ID channel: str # 渠道标识: "telegram", "whatsapp" 等 text: str # 清洗后的文本内容 # 元数据 raw\_text: str # 原始文本（用于调试） timestamp: datetime # 消息时间戳 metadata: Dict\[str, Any\] = field(default\_factory=dict) # 其他元数据 # 清洗标记 is\_cleaned: bool = False # 是否经过清洗 is\_duplicate: bool = False # 是否是重复消息 def to\_dict(self) -> Dict\[str, Any\]: """转换为字典格式""" return { "message\_id": self.message\_id, "user\_id": self.user\_id, "chat\_id": self.chat\_id, "channel": self.channel, "text": self.text, "raw\_text": self.raw\_text, "timestamp": self.timestamp.isoformat(), "metadata": self.metadata, "is\_cleaned": self.is\_cleaned, "is\_duplicate": self.is\_duplicate, }

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

"""Ingress 数据模型"""

from dataclasses import dataclass, field

from datetime import datetime

from typing import Any, Dict

@dataclass

class StandardMessage:

"""标准化消息格式

所有外部消息（Telegram, WhatsApp 等）都转换为该格式

"""

\# 基础信息

message\_id: str \# 消息唯一标识 (用于去重)

user\_id: str \# 用户ID

chat\_id: str \# 聊天ID

channel: str \# 渠道标识: "telegram", "whatsapp" 等

text: str \# 清洗后的文本内容

\# 元数据

raw\_text: str \# 原始文本（用于调试）

timestamp: datetime \# 消息时间戳

metadata: Dict\[str, Any\] \= field(default\_factory\=dict) \# 其他元数据

\# 清洗标记

is\_cleaned: bool \= False \# 是否经过清洗

is\_duplicate: bool \= False \# 是否是重复消息

def to\_dict(self) \-> Dict\[str, Any\]:

"""转换为字典格式"""

return {

"message\_id": self.message\_id,

"user\_id": self.user\_id,

"chat\_id": self.chat\_id,

"channel": self.channel,

"text": self.text,

"raw\_text": self.raw\_text,

"timestamp": self.timestamp.isoformat(),

"metadata": self.metadata,

"is\_cleaned": self.is\_cleaned,

"is\_duplicate": self.is\_duplicate,

}