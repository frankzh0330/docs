# Source: https://github.com/frankzh0330/query-agent/blob/master/matcher/metric_matcher.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# metric\_matcher.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/matcher/metric_matcher.py)

Open commit details

History

118 lines (97 loc) · 3.62 KB

## FilesExpand file tree

master

/

# metric\_matcher.py

Copy path

Top

## File metadata and controls

- Code

- Blame

118 lines (97 loc) · 3.62 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/matcher/metric_matcher.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from typing import Any, Dict, Set from matcher.base import DEFAULT\_STOPWORDS, DEFAULT\_SYNONYM\_MAP, BaseMatcher, \_stable\_id class MetricMatcher(BaseMatcher): """ 指标匹配器 支持： 1. exact alias 命中 2. 中英混合分词 3. 倒排索引召回 4. RapidFuzz 重排 """ def \_build\_index(self, catalog\_metrics: Dict\[str, Dict\[str, Any\]\], enable\_jieba\_userdict: bool = True) -> None: """ 构建指标索引 :param catalog\_metrics: 结构示例： { "pv": { "aliases": \["PV", "浏览量", "访问量", "点击量"\], "indicator\_show\_name": "浏览量", "event\_indicator": "pv" }, "uv": { "aliases": \["UV", "独立访客", "访客数", "用户数"\], "indicator\_show\_name": "访客数", "event\_indicator": "uv" } } """ tmp\_index: Dict\[str, Set\[int\]\] = {} # 先分配 metric\_id（使用稳定哈希 ID） for metric\_name, info in catalog\_metrics.items(): metric\_id = \_stable\_id(metric\_name) # 稳定的整数 ID aliases = info.get("aliases", \[\]) or \[\] all\_aliases = \[metric\_name\] + list(aliases) all\_aliases = list(dict.fromkeys(all\_aliases)) # 去重 self.name\_to\_id\[metric\_name\] = metric\_id self.id\_to\_doc\[metric\_id\] = { "metric\_id": metric\_id, "metric\_name": metric\_name, "indicator\_show\_name": info.get("indicator\_show\_name", metric\_name), "event\_indicator": info.get("event\_indicator", metric\_name), "aliases": all\_aliases, } self.id\_to\_norm\_aliases\[metric\_id\] = \[\] # 加入 jieba 词典 if enable\_jieba\_userdict: self.\_add\_to\_jieba\_dict(all\_aliases) # 再建 exact map + 倒排索引 for metric\_id, doc in self.id\_to\_doc.items(): self.\_build\_inverted\_index(metric\_id, doc\["aliases"\], tmp\_index) self.token\_to\_ids = self.\_finalize\_index(tmp\_index) def \_get\_name\_by\_id(self, doc\_id: int) -> str: return self.id\_to\_doc\[doc\_id\]\["metric\_name"\] def build\_metric\_matcher\_from\_catalog(catalog\_metrics: Dict\[str, Dict\[str, Any\]\]) -> MetricMatcher: """从 catalog 创建 MetricMatcher""" return MetricMatcher( catalog\_data=catalog\_metrics, stopwords=DEFAULT\_STOPWORDS, synonym\_map=DEFAULT\_SYNONYM\_MAP, fuzzy\_threshold=70.0, max\_candidates=200, max\_posting\_per\_token=1000, enable\_jieba\_userdict=True, ) # ========================= # 本地测试 # ========================= if \_\_name\_\_ == "\_\_main\_\_": mock\_catalog\_metrics = { "pv": { "aliases": \["PV", "浏览量", "访问量", "点击量"\], "indicator\_show\_name": "浏览量", "event\_indicator": "pv" }, "uv": { "aliases": \["UV", "独立访客", "访客数", "用户数"\], "indicator\_show\_name": "访客数", "event\_indicator": "uv" } } matcher = build\_metric\_matcher\_from\_catalog(mock\_catalog\_metrics) test\_queries = \[ "PV", "pv", "浏览量", "访问量", "UV", "uv", "访客数", "用户数", \] for q in test\_queries: result = matcher.match(q) print("=" \* 60) print(f"query: {q}") print(f"matched: {result.matched}, score: {result.score}")

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

85

86

87

88

89

90

91

92

93

94

95

96

97

98

99

100

101

102

103

104

105

106

107

108

109

110

111

112

113

114

115

116

117

118

from \_\_future\_\_ import annotations

from typing import Any, Dict, Set

from matcher.base import DEFAULT\_STOPWORDS, DEFAULT\_SYNONYM\_MAP, BaseMatcher, \_stable\_id

class MetricMatcher(BaseMatcher):

"""

指标匹配器

支持：

1\. exact alias 命中

2\. 中英混合分词

3\. 倒排索引召回

4\. RapidFuzz 重排

"""

def \_build\_index(self, catalog\_metrics: Dict\[str, Dict\[str, Any\]\], enable\_jieba\_userdict: bool \= True) \-> None:

"""

构建指标索引

:param catalog\_metrics: 结构示例：

{

"pv": {

"aliases": \["PV", "浏览量", "访问量", "点击量"\],

"indicator\_show\_name": "浏览量",

"event\_indicator": "pv"

},

"uv": {

"aliases": \["UV", "独立访客", "访客数", "用户数"\],

"indicator\_show\_name": "访客数",

"event\_indicator": "uv"

}

}

"""

tmp\_index: Dict\[str, Set\[int\]\] \= {}

\# 先分配 metric\_id（使用稳定哈希 ID）

for metric\_name, info in catalog\_metrics.items():

metric\_id \= \_stable\_id(metric\_name) \# 稳定的整数 ID

aliases \= info.get("aliases", \[\]) or \[\]

all\_aliases \= \[metric\_name\] + list(aliases)

all\_aliases \= list(dict.fromkeys(all\_aliases)) \# 去重

self.name\_to\_id\[metric\_name\] \= metric\_id

self.id\_to\_doc\[metric\_id\] \= {

"metric\_id": metric\_id,

"metric\_name": metric\_name,

"indicator\_show\_name": info.get("indicator\_show\_name", metric\_name),

"event\_indicator": info.get("event\_indicator", metric\_name),

"aliases": all\_aliases,

}

self.id\_to\_norm\_aliases\[metric\_id\] \= \[\]

\# 加入 jieba 词典

if enable\_jieba\_userdict:

self.\_add\_to\_jieba\_dict(all\_aliases)

\# 再建 exact map + 倒排索引

for metric\_id, doc in self.id\_to\_doc.items():

self.\_build\_inverted\_index(metric\_id, doc\["aliases"\], tmp\_index)

self.token\_to\_ids \= self.\_finalize\_index(tmp\_index)

def \_get\_name\_by\_id(self, doc\_id: int) \-> str:

return self.id\_to\_doc\[doc\_id\]\["metric\_name"\]

def build\_metric\_matcher\_from\_catalog(catalog\_metrics: Dict\[str, Dict\[str, Any\]\]) \-> MetricMatcher:

"""从 catalog 创建 MetricMatcher"""

return MetricMatcher(

catalog\_data\=catalog\_metrics,

stopwords\=DEFAULT\_STOPWORDS,

synonym\_map\=DEFAULT\_SYNONYM\_MAP,

fuzzy\_threshold\=70.0,

max\_candidates\=200,

max\_posting\_per\_token\=1000,

enable\_jieba\_userdict\=True,

)

\# =========================

\# 本地测试

\# =========================

if \_\_name\_\_ \== "\_\_main\_\_":

mock\_catalog\_metrics \= {

"pv": {

"aliases": \["PV", "浏览量", "访问量", "点击量"\],

"indicator\_show\_name": "浏览量",

"event\_indicator": "pv"

},

"uv": {

"aliases": \["UV", "独立访客", "访客数", "用户数"\],

"indicator\_show\_name": "访客数",

"event\_indicator": "uv"

}

}

matcher \= build\_metric\_matcher\_from\_catalog(mock\_catalog\_metrics)

test\_queries \= \[

"PV",

"pv",

"浏览量",

"访问量",

"UV",

"uv",

"访客数",

"用户数",

\]

for q in test\_queries:

result \= matcher.match(q)

print("=" \* 60)

print(f"query: {q}")

print(f"matched: {result.matched}, score: {result.score}")