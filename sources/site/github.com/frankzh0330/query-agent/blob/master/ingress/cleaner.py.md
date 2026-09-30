# Source: https://github.com/frankzh0330/query-agent/blob/master/ingress/cleaner.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# cleaner.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/ingress/cleaner.py)

Open commit details

History

61 lines (46 loc) · 1.67 KB

## FilesExpand file tree

master

/

# cleaner.py

Copy path

Top

## File metadata and controls

- Code

- Blame

61 lines (46 loc) · 1.67 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/ingress/cleaner.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""消息清洗器 - 清洗脏数据和无意义的前缀""" import re from typing import List class MessageCleaner: """消息清洗器 功能： 1. 移除控制字符和脏字符 2. 移除无意义前缀（如机器人命令） 3. 标准化空白字符 """ # 脏数据模式 DIRTY\_PATTERNS = \[ r'\[\\x00-\\x08\\x0b-\\x0c\\x0e-\\x1f\\x7f-\\x9f\]', # 控制字符 r'\[^\\w\\s\\u4e00-\\u9fff\\u3000-\\u303f\\uff00-\\uffef,.!?;:()""''【】（）。，！？；：\]', # 非常用字符 \] # 无意义的前缀（如机器人命令） PREFIX\_PATTERNS = \[ r'^/\[a-zA-Z\]+\\s\*', # Telegram bot commands: /start, /help 等 \] def \_\_init\_\_(self): # 编译正则表达式 self.dirty\_regex = re.compile('|'.join(self.DIRTY\_PATTERNS)) self.prefix\_regex = re.compile('|'.join(self.PREFIX\_PATTERNS)) def clean(self, text: str) -> str: """清洗文本""" if not text: return "" # 1. 先移除无意义前缀（但保留内容） # 必须在脏字符清洗之前，否则前缀可能被破坏 text = self.prefix\_regex.sub('', text) # 2. 移除控制字符和脏字符 text = self.dirty\_regex.sub('', text) # 3. 标准化空白字符 text = re.sub(r'\\s+', ' ', text) # 4. 去除首尾空格 text = text.strip() # 5. 检查清洗后是否为空 if not text or len(text) < 2: return "" return text def is\_valid(self, text: str) -> bool: """检查文本是否有效""" if not text: return False cleaned = self.clean(text) return len(cleaned) >= 2

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

"""消息清洗器 - 清洗脏数据和无意义的前缀"""

import re

from typing import List

class MessageCleaner:

"""消息清洗器

功能：

1\. 移除控制字符和脏字符

2\. 移除无意义前缀（如机器人命令）

3\. 标准化空白字符

"""

\# 脏数据模式

DIRTY\_PATTERNS \= \[

r'\[\\x00-\\x08\\x0b-\\x0c\\x0e-\\x1f\\x7f-\\x9f\]', \# 控制字符

r'\[^\\w\\s\\u4e00-\\u9fff\\u3000-\\u303f\\uff00-\\uffef,.!?;:()""''【】（）。，！？；：\]', \# 非常用字符

\]

\# 无意义的前缀（如机器人命令）

PREFIX\_PATTERNS \= \[

r'^/\[a-zA-Z\]+\\s\*', \# Telegram bot commands: /start, /help 等

\]

def \_\_init\_\_(self):

\# 编译正则表达式

self.dirty\_regex \= re.compile('|'.join(self.DIRTY\_PATTERNS))

self.prefix\_regex \= re.compile('|'.join(self.PREFIX\_PATTERNS))

def clean(self, text: str) \-> str:

"""清洗文本"""

if not text:

return ""

\# 1. 先移除无意义前缀（但保留内容）

\# 必须在脏字符清洗之前，否则前缀可能被破坏

text \= self.prefix\_regex.sub('', text)

\# 2. 移除控制字符和脏字符

text \= self.dirty\_regex.sub('', text)

\# 3. 标准化空白字符

text \= re.sub(r'\\s+', ' ', text)

\# 4. 去除首尾空格

text \= text.strip()

\# 5. 检查清洗后是否为空

if not text or len(text) < 2:

return ""

return text

def is\_valid(self, text: str) \-> bool:

"""检查文本是否有效"""

if not text:

return False

cleaned \= self.clean(text)

return len(cleaned) \>= 2