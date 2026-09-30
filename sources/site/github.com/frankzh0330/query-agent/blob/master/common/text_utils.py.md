# Source: https://github.com/frankzh0330/query-agent/blob/master/common/text_utils.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# text\_utils.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add alias/token limits and dynamic catalog sync](https://github.com/frankzh0330/query-agent/commit/973279b515c7a09e1ccf1bda4ca282547f0145de)

Open commit details

Mar 22, 2026

[973279b](https://github.com/frankzh0330/query-agent/commit/973279b515c7a09e1ccf1bda4ca282547f0145de) · Mar 22, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/common/text_utils.py)

Open commit details

History

84 lines (61 loc) · 2.02 KB

## FilesExpand file tree

master

/

# text\_utils.py

Copy path

Top

## File metadata and controls

- Code

- Blame

84 lines (61 loc) · 2.02 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/common/text_utils.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import re from typing import List import jieba def contains\_chinese(text: str) -> bool: """检测文本是否包含中文""" return bool(re.search(r"\[\\u4e00-\\u9fff\]", text or "")) def is\_single\_chinese\_char(token: str) -> bool: """检测是否为单个中文字符""" return bool(re.fullmatch(r"\[\\u4e00-\\u9fff\]", token)) def normalize(text: str) -> str: """ 全局 normalize： - lower - camelCase -> camel Case - \_ - -> 空格 - 去掉大部分特殊符号 - 压缩空格 """ if text is None: return "" text = text.strip() # camelCase -> camel Case text = re.sub(r"(\[a-z\])(\[A-Z\])", r"\\1 \\2", text) text = text.lower() text = text.replace("\_", " ").replace("-", " ") # 保留中英文、数字、空格 text = re.sub(r"\[^\\w\\u4e00-\\u9fff\\s\]", " ", text) text = re.sub(r"\\s+", " ", text).strip() return text def tokenize\_mixed(text: str, max\_tokens: int = 5) -> List\[str\]: """ 中英混合分词： - 英文 / 数字 / 下划线块：按 \_ 和空格拆 - 中文块：用 jieba.cut Args: text: 待分词文本 max\_tokens: 最多返回多少个 token，默认 5 """ norm\_text = text.strip().lower() if not norm\_text: return \[\] # 按中英文块切开 parts = re.findall(r"\[a-z0-9\_\]+|\[\\u4e00-\\u9fff\]+", norm\_text) tokens: List\[str\] = \[\] for part in parts: # 英文块 if re.fullmatch(r"\[a-z0-9\_\]+", part): english\_tokens = re.split(r"\[\_\\s\]+", part) tokens.extend(\[t for t in english\_tokens if t\]) # 中文块 else: chinese\_tokens = \[t.strip() for t in jieba.cut(part) if t.strip()\] tokens.extend(chinese\_tokens) # normalize once more tokens = \[normalize(t) for t in tokens if normalize(t)\] tokens = list(dict.fromkeys(tokens)) # 去重保序 # 限制 token 数量 if len(tokens) > max\_tokens: tokens = tokens\[:max\_tokens\] return tokens

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

80

81

82

83

84

from \_\_future\_\_ import annotations

import re

from typing import List

import jieba

def contains\_chinese(text: str) \-> bool:

"""检测文本是否包含中文"""

return bool(re.search(r"\[\\u4e00-\\u9fff\]", text or ""))

def is\_single\_chinese\_char(token: str) \-> bool:

"""检测是否为单个中文字符"""

return bool(re.fullmatch(r"\[\\u4e00-\\u9fff\]", token))

def normalize(text: str) \-> str:

"""

全局 normalize：

\- lower

\- camelCase -> camel Case

\- \_ - -> 空格

\- 去掉大部分特殊符号

\- 压缩空格

"""

if text is None:

return ""

text \= text.strip()

\# camelCase -> camel Case

text \= re.sub(r"(\[a-z\])(\[A-Z\])", r"\\1 \\2", text)

text \= text.lower()

text \= text.replace("\_", " ").replace("-", " ")

\# 保留中英文、数字、空格

text \= re.sub(r"\[^\\w\\u4e00-\\u9fff\\s\]", " ", text)

text \= re.sub(r"\\s+", " ", text).strip()

return text

def tokenize\_mixed(text: str, max\_tokens: int \= 5) \-> List\[str\]:

"""

中英混合分词：

\- 英文 / 数字 / 下划线块：按 \_ 和空格拆

\- 中文块：用 jieba.cut

Args:

text: 待分词文本

max\_tokens: 最多返回多少个 token，默认 5

"""

norm\_text \= text.strip().lower()

if not norm\_text:

return \[\]

\# 按中英文块切开

parts \= re.findall(r"\[a-z0-9\_\]+|\[\\u4e00-\\u9fff\]+", norm\_text)

tokens: List\[str\] \= \[\]

for part in parts:

\# 英文块

if re.fullmatch(r"\[a-z0-9\_\]+", part):

english\_tokens \= re.split(r"\[\_\\s\]+", part)

tokens.extend(\[t for t in english\_tokens if t\])

\# 中文块

else:

chinese\_tokens \= \[t.strip() for t in jieba.cut(part) if t.strip()\]

tokens.extend(chinese\_tokens)

\# normalize once more

tokens \= \[normalize(t) for t in tokens if normalize(t)\]

tokens \= list(dict.fromkeys(tokens)) \# 去重保序

\# 限制 token 数量

if len(tokens) \> max\_tokens:

tokens \= tokens\[:max\_tokens\]

return tokens