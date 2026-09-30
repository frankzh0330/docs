# Source: https://github.com/frankzh0330/query-agent/blob/master/ingress/deduplicator.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# deduplicator.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/ingress/deduplicator.py)

Open commit details

History

74 lines (57 loc) · 2.13 KB

## FilesExpand file tree

master

/

# deduplicator.py

Copy path

Top

## File metadata and controls

- Code

- Blame

74 lines (57 loc) · 2.13 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/ingress/deduplicator.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""消息去重器 - 基于滑动窗口的内存去重""" import time from collections import deque from typing import Deque, Set, Tuple class MessageDeduplicator: """消息去重器（滑动窗口，内存存储） 使用场景： - 防止重复处理相同的消息 - 基于 message\_id 判断去重 """ def \_\_init\_\_(self, window\_size: int = 10000, ttl\_seconds: int = 3600): """初始化去重器 Args: window\_size: 窗口大小（最大记录消息数） ttl\_seconds: 消息过期时间（秒） """ self.window\_size = window\_size self.ttl\_seconds = ttl\_seconds self.seen\_messages: Set\[str\] = set() self.message\_timestamps: Deque\[Tuple\[str, float\]\] = deque() def is\_duplicate(self, message\_id: str) -> bool: """检查消息是否重复 Args: message\_id: 消息唯一标识 Returns: True 如果是重复消息，False 如果是新消息 """ # 清理过期消息 self.\_cleanup\_expired() if message\_id in self.seen\_messages: return True # 记录新消息 self.seen\_messages.add(message\_id) self.message\_timestamps.append((message\_id, time.time())) # 限制窗口大小 if len(self.message\_timestamps) > self.window\_size: old\_id, \_ = self.message\_timestamps.popleft() self.seen\_messages.discard(old\_id) return False def \_cleanup\_expired(self) -> None: """清理过期的消息记录""" now = time.time() cutoff = now - self.ttl\_seconds while self.message\_timestamps: \_, timestamp = self.message\_timestamps\[0\] if timestamp < cutoff: old\_id, \_ = self.message\_timestamps.popleft() self.seen\_messages.discard(old\_id) else: break def reset(self) -> None: """重置去重器""" self.seen\_messages.clear() self.message\_timestamps.clear() def size(self) -> int: """获取当前缓存的消息数量""" return len(self.seen\_messages)

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

"""消息去重器 - 基于滑动窗口的内存去重"""

import time

from collections import deque

from typing import Deque, Set, Tuple

class MessageDeduplicator:

"""消息去重器（滑动窗口，内存存储）

使用场景：

\- 防止重复处理相同的消息

\- 基于 message\_id 判断去重

"""

def \_\_init\_\_(self, window\_size: int \= 10000, ttl\_seconds: int \= 3600):

"""初始化去重器

Args:

window\_size: 窗口大小（最大记录消息数）

ttl\_seconds: 消息过期时间（秒）

"""

self.window\_size \= window\_size

self.ttl\_seconds \= ttl\_seconds

self.seen\_messages: Set\[str\] \= set()

self.message\_timestamps: Deque\[Tuple\[str, float\]\] \= deque()

def is\_duplicate(self, message\_id: str) \-> bool:

"""检查消息是否重复

Args:

message\_id: 消息唯一标识

Returns:

True 如果是重复消息，False 如果是新消息

"""

\# 清理过期消息

self.\_cleanup\_expired()

if message\_id in self.seen\_messages:

return True

\# 记录新消息

self.seen\_messages.add(message\_id)

self.message\_timestamps.append((message\_id, time.time()))

\# 限制窗口大小

if len(self.message\_timestamps) \> self.window\_size:

old\_id, \_ \= self.message\_timestamps.popleft()

self.seen\_messages.discard(old\_id)

return False

def \_cleanup\_expired(self) \-> None:

"""清理过期的消息记录"""

now \= time.time()

cutoff \= now \- self.ttl\_seconds

while self.message\_timestamps:

\_, timestamp \= self.message\_timestamps\[0\]

if timestamp < cutoff:

old\_id, \_ \= self.message\_timestamps.popleft()

self.seen\_messages.discard(old\_id)

else:

break

def reset(self) \-> None:

"""重置去重器"""

self.seen\_messages.clear()

self.message\_timestamps.clear()

def size(self) \-> int:

"""获取当前缓存的消息数量"""

return len(self.seen\_messages)