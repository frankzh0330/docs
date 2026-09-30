# Source: https://github.com/frankzh0330/query-agent/blob/master/memory/judge_prompt.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# judge\_prompt.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add LLM-driven memory writer and replace print() with logger.de…](https://github.com/frankzh0330/query-agent/commit/c6d596c4549efeaba4556e94aa2afef33305403f)

Open commit details

Apr 14, 2026

[c6d596c](https://github.com/frankzh0330/query-agent/commit/c6d596c4549efeaba4556e94aa2afef33305403f) · Apr 14, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/memory/judge_prompt.py)

Open commit details

History

82 lines (60 loc) · 2.45 KB

## FilesExpand file tree

master

/

# judge\_prompt.py

Copy path

Top

## File metadata and controls

- Code

- Blame

82 lines (60 loc) · 2.45 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/memory/judge_prompt.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Memory Judge — LLM 判断是否需要写入记忆的 prompt""" from \_\_future\_\_ import annotations JUDGE\_SYSTEM\_PROMPT = r"""你是"记忆判断器"。分析本轮查询对话，判断是否有值得长期记住的信息。 ## 值得记住的（should\_save=true） 1. \*\*用户纠正\*\*（category=correction） - 用户明确否定了系统默认选择，通过确认流选了不同的值 - 用户说"不是X，是Y"、"我要看的是Z" - 例子：系统默认选了 app\_launch，用户纠正为 payment\_submit 2. \*\*用户偏好\*\*（category=preference） - 用户表达了稳定偏好："我一般/总是/默认/习惯 看XX" - 例子："我一般看近30天的"、"我习惯查UV" 3. \*\*约束发现\*\*（category=constraint） - 从对话中发现的项目特定规则 - 例子："purchase 在这个项目里就是 payment\_submit" ## 不值得记住的（should\_save=false） - 普通查询（无纠正、无偏好表达、无新发现） - 已有记忆中已包含的信息 - 一次性的、不确定的表述 - 系统正常工作的高置信度匹配 ## 输出格式 返回 JSON，不要输出其他内容： {"should\_save": true/false, "category": "correction|preference|constraint", "content": "一句话描述，用中文"} """ JUDGE\_USER\_TEMPLATE = r"""请分析以下查询是否值得写入长期记忆。 ## 本轮对话信息 用户查询: {user\_query} LLM 提取结果: {extraction} Matcher 解析: {resolver\_explain} 最终查询状态: {current\_state} 上一轮查询状态: {prev\_state} ## 已有记忆（避免重复） {existing\_memory} --- 返回 JSON：""" JUDGE\_TOOL\_SCHEMA = { "type": "function", "function": { "name": "judge\_memory", "description": "判断是否需要将本轮对话信息写入长期记忆", "parameters": { "type": "object", "properties": { "should\_save": { "type": "boolean", "description": "是否值得写入记忆", }, "category": { "type": "string", "enum": \["correction", "preference", "constraint"\], "description": "记忆类别", }, "content": { "type": "string", "description": "一句话描述，用中文，下次 LLM 可直接理解", }, }, "required": \["should\_save"\], }, }, }

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

"""Memory Judge — LLM 判断是否需要写入记忆的 prompt"""

from \_\_future\_\_ import annotations

JUDGE\_SYSTEM\_PROMPT \= r"""你是"记忆判断器"。分析本轮查询对话，判断是否有值得长期记住的信息。

\## 值得记住的（should\_save=true）

1\. \*\*用户纠正\*\*（category=correction）

\- 用户明确否定了系统默认选择，通过确认流选了不同的值

\- 用户说"不是X，是Y"、"我要看的是Z"

\- 例子：系统默认选了 app\_launch，用户纠正为 payment\_submit

2\. \*\*用户偏好\*\*（category=preference）

\- 用户表达了稳定偏好："我一般/总是/默认/习惯 看XX"

\- 例子："我一般看近30天的"、"我习惯查UV"

3\. \*\*约束发现\*\*（category=constraint）

\- 从对话中发现的项目特定规则

\- 例子："purchase 在这个项目里就是 payment\_submit"

\## 不值得记住的（should\_save=false）

\- 普通查询（无纠正、无偏好表达、无新发现）

\- 已有记忆中已包含的信息

\- 一次性的、不确定的表述

\- 系统正常工作的高置信度匹配

\## 输出格式

返回 JSON，不要输出其他内容：

{"should\_save": true/false, "category": "correction|preference|constraint", "content": "一句话描述，用中文"}

"""

JUDGE\_USER\_TEMPLATE \= r"""请分析以下查询是否值得写入长期记忆。

\## 本轮对话信息

用户查询: {user\_query}

LLM 提取结果: {extraction}

Matcher 解析: {resolver\_explain}

最终查询状态: {current\_state}

上一轮查询状态: {prev\_state}

\## 已有记忆（避免重复）

{existing\_memory}

\---

返回 JSON："""

JUDGE\_TOOL\_SCHEMA \= {

"type": "function",

"function": {

"name": "judge\_memory",

"description": "判断是否需要将本轮对话信息写入长期记忆",

"parameters": {

"type": "object",

"properties": {

"should\_save": {

"type": "boolean",

"description": "是否值得写入记忆",

},

"category": {

"type": "string",

"enum": \["correction", "preference", "constraint"\],

"description": "记忆类别",

},

"content": {

"type": "string",

"description": "一句话描述，用中文，下次 LLM 可直接理解",

},

},

"required": \["should\_save"\],

},

},

}