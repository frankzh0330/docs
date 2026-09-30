# Source: https://github.com/frankzh0330/query-agent/blob/master/ingress/telegram_adapter.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# telegram\_adapter.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/ingress/telegram_adapter.py)

Open commit details

History

79 lines (60 loc) · 2.37 KB

## FilesExpand file tree

master

/

# telegram\_adapter.py

Copy path

Top

## File metadata and controls

- Code

- Blame

79 lines (60 loc) · 2.37 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/ingress/telegram_adapter.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Telegram 渠道适配器""" from datetime import datetime from typing import Any, Dict from ingress.base\_adapter import BaseIngressAdapter from ingress.cleaner import MessageCleaner from ingress.deduplicator import MessageDeduplicator from ingress.models import StandardMessage # 全局去重器实例（跨所有 TelegramAdapter 共享） \_global\_deduplicator = MessageDeduplicator() class TelegramAdapter(BaseIngressAdapter): """Telegram 渠道适配器 将 Telegram update 转换为 StandardMessage """ def \_\_init\_\_(self, deduplicator: MessageDeduplicator | None = None): """初始化适配器 Args: deduplicator: 可选的去重器，默认使用全局共享实例 """ self.cleaner = MessageCleaner() self.deduplicator = deduplicator or \_global\_deduplicator def adapt(self, raw\_update: Dict\[str, Any\]) -> StandardMessage: """转换 Telegram update 为标准消息 Args: raw\_update: Telegram Bot API 的 update 对象 Returns: StandardMessage: 标准化消息 """ message = raw\_update.get("message", {}) # 提取基础信息 message\_id = self.extract\_message\_id(raw\_update) user\_id = str(message.get("from", {}).get("id", "")) chat\_id = str(message.get("chat", {}).get("id", "")) raw\_text = message.get("text", "") # 清洗文本 cleaned\_text = self.cleaner.clean(raw\_text) # 去重检查 is\_duplicate = self.deduplicator.is\_duplicate(message\_id) return StandardMessage( message\_id=message\_id, user\_id=user\_id, chat\_id=chat\_id, channel="telegram", text=cleaned\_text, raw\_text=raw\_text, timestamp=datetime.now(), metadata={"raw\_update": raw\_update}, is\_cleaned=(cleaned\_text != raw\_text), is\_duplicate=is\_duplicate, ) def extract\_message\_id(self, raw\_update: Dict\[str, Any\]) -> str: """提取 Telegram 消息唯一ID Args: raw\_update: Telegram Bot API 的 update 对象 Returns: 消息唯一标识 """ update\_id = raw\_update.get("update\_id", 0) message = raw\_update.get("message", {}) message\_id = message.get("message\_id", 0) return f"telegram\_{update\_id}\_{message\_id}"

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

"""Telegram 渠道适配器"""

from datetime import datetime

from typing import Any, Dict

from ingress.base\_adapter import BaseIngressAdapter

from ingress.cleaner import MessageCleaner

from ingress.deduplicator import MessageDeduplicator

from ingress.models import StandardMessage

\# 全局去重器实例（跨所有 TelegramAdapter 共享）

\_global\_deduplicator \= MessageDeduplicator()

class TelegramAdapter(BaseIngressAdapter):

"""Telegram 渠道适配器

将 Telegram update 转换为 StandardMessage

"""

def \_\_init\_\_(self, deduplicator: MessageDeduplicator | None \= None):

"""初始化适配器

Args:

deduplicator: 可选的去重器，默认使用全局共享实例

"""

self.cleaner \= MessageCleaner()

self.deduplicator \= deduplicator or \_global\_deduplicator

def adapt(self, raw\_update: Dict\[str, Any\]) \-> StandardMessage:

"""转换 Telegram update 为标准消息

Args:

raw\_update: Telegram Bot API 的 update 对象

Returns:

StandardMessage: 标准化消息

"""

message \= raw\_update.get("message", {})

\# 提取基础信息

message\_id \= self.extract\_message\_id(raw\_update)

user\_id \= str(message.get("from", {}).get("id", ""))

chat\_id \= str(message.get("chat", {}).get("id", ""))

raw\_text \= message.get("text", "")

\# 清洗文本

cleaned\_text \= self.cleaner.clean(raw\_text)

\# 去重检查

is\_duplicate \= self.deduplicator.is\_duplicate(message\_id)

return StandardMessage(

message\_id\=message\_id,

user\_id\=user\_id,

chat\_id\=chat\_id,

channel\="telegram",

text\=cleaned\_text,

raw\_text\=raw\_text,

timestamp\=datetime.now(),

metadata\={"raw\_update": raw\_update},

is\_cleaned\=(cleaned\_text != raw\_text),

is\_duplicate\=is\_duplicate,

)

def extract\_message\_id(self, raw\_update: Dict\[str, Any\]) \-> str:

"""提取 Telegram 消息唯一ID

Args:

raw\_update: Telegram Bot API 的 update 对象

Returns:

消息唯一标识

"""

update\_id \= raw\_update.get("update\_id", 0)

message \= raw\_update.get("message", {})

message\_id \= message.get("message\_id", 0)

return f"telegram\_{update\_id}\_{message\_id}"