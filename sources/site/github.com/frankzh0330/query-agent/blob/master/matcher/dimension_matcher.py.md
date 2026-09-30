# Source: https://github.com/frankzh0330/query-agent/blob/master/matcher/dimension_matcher.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# dimension\_matcher.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/matcher/dimension_matcher.py)

Open commit details

History

149 lines (125 loc) · 4.72 KB

## FilesExpand file tree

master

/

# dimension\_matcher.py

Copy path

Top

## File metadata and controls

- Code

- Blame

149 lines (125 loc) · 4.72 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/matcher/dimension_matcher.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from typing import Any, Dict, Set from matcher.base import DEFAULT\_STOPWORDS, DEFAULT\_SYNONYM\_MAP, BaseMatcher, \_stable\_id class DimensionMatcher(BaseMatcher): """ 维度匹配器，用于匹配 property\_name（如 country） 支持： 1. exact alias 命中 2. 中英混合分词 3. 倒排索引召回 4. RapidFuzz 重排 5. query 侧同义词扩展 """ def \_build\_index(self, catalog\_dimensions: Dict\[str, Dict\[str, Any\]\], enable\_jieba\_userdict: bool = True) -> None: """ 构建维度索引 :param catalog\_dimensions: 结构示例： { "country": { "aliases": \["country", "国家"\], "property\_type": "profile", "property\_name": "country", "property\_compose\_type": "origin" } } """ tmp\_index: Dict\[str, Set\[int\]\] = {} # 先分配 dimension\_id（使用稳定哈希 ID） for dimension\_name, info in catalog\_dimensions.items(): dim\_id = \_stable\_id(dimension\_name) # 稳定的整数 ID aliases = info.get("aliases", \[\]) or \[\] all\_aliases = \[dimension\_name\] + list(aliases) all\_aliases = list(dict.fromkeys(all\_aliases)) # 去重 self.name\_to\_id\[dimension\_name\] = dim\_id self.id\_to\_doc\[dim\_id\] = { "dimension\_id": dim\_id, "dimension\_name": dimension\_name, "property\_name": info.get("property\_name", dimension\_name), "property\_type": info.get("property\_type", "profile"), "property\_compose\_type": info.get("property\_compose\_type", "origin"), "aliases": all\_aliases, } self.id\_to\_norm\_aliases\[dim\_id\] = \[\] # 加入 jieba 词典 if enable\_jieba\_userdict: self.\_add\_to\_jieba\_dict(all\_aliases) # 再建 exact map + 倒排索引 for dim\_id, doc in self.id\_to\_doc.items(): self.\_build\_inverted\_index(dim\_id, doc\["aliases"\], tmp\_index) self.token\_to\_ids = self.\_finalize\_index(tmp\_index) def \_get\_name\_by\_id(self, doc\_id: int) -> str: return self.id\_to\_doc\[doc\_id\]\["dimension\_name"\] def match\_property(self, query: str) -> tuple: """ 匹配并返回 property\_name :return: (property\_name, score, explain) """ result = self.match(query) if result.matched: doc = self.id\_to\_doc\[self.name\_to\_id\[result.matched\]\] return doc\["property\_name"\], result.score, result.explain return None, result.score, result.explain # ========================= # 帮助函数：从 catalog 创建 matcher # ========================= def build\_dimension\_matcher\_from\_catalog(catalog\_dimensions: Dict\[str, Dict\[str, Any\]\]) -> DimensionMatcher: """从 catalog 创建 DimensionMatcher""" return DimensionMatcher( catalog\_data=catalog\_dimensions, stopwords=DEFAULT\_STOPWORDS, synonym\_map=DEFAULT\_SYNONYM\_MAP, fuzzy\_threshold=70.0, max\_candidates=200, max\_posting\_per\_token=1000, enable\_jieba\_userdict=True, ) # ========================= # 本地测试 # ========================= if \_\_name\_\_ == "\_\_main\_\_": mock\_catalog\_dimensions = { "country": { "aliases": \["country", "国家", "地区"\], "property\_type": "profile", "property\_name": "country", "property\_compose\_type": "origin" }, "city": { "aliases": \["city", "城市"\], "property\_type": "profile", "property\_name": "city", "property\_compose\_type": "origin" }, "app\_version": { "aliases": \["app\_version", "应用版本", "版本"\], "property\_type": "profile", "property\_name": "app\_version", "property\_compose\_type": "origin" }, "os": { "aliases": \["os", "操作系统", "系统"\], "property\_type": "profile", "property\_name": "os", "property\_compose\_type": "origin" }, } matcher = build\_dimension\_matcher\_from\_catalog(mock\_catalog\_dimensions) test\_queries = \[ "country", "国家", "地区", "城市", "版本", "应用版本", "操作系统", "os", \] for q in test\_queries: result = matcher.match(q) prop\_name, score, explain = matcher.match\_property(q) print("=" \* 100) print("query:", q) print("matched dimension:", result.matched) print("property\_name:", prop\_name) print("score:", score)

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

119

120

121

122

123

124

125

126

127

128

129

130

131

132

133

134

135

136

137

138

139

140

141

142

143

144

145

146

147

148

149

from \_\_future\_\_ import annotations

from typing import Any, Dict, Set

from matcher.base import DEFAULT\_STOPWORDS, DEFAULT\_SYNONYM\_MAP, BaseMatcher, \_stable\_id

class DimensionMatcher(BaseMatcher):

"""

维度匹配器，用于匹配 property\_name（如 country）

支持：

1\. exact alias 命中

2\. 中英混合分词

3\. 倒排索引召回

4\. RapidFuzz 重排

5\. query 侧同义词扩展

"""

def \_build\_index(self, catalog\_dimensions: Dict\[str, Dict\[str, Any\]\], enable\_jieba\_userdict: bool \= True) \-> None:

"""

构建维度索引

:param catalog\_dimensions: 结构示例：

{

"country": {

"aliases": \["country", "国家"\],

"property\_type": "profile",

"property\_name": "country",

"property\_compose\_type": "origin"

}

}

"""

tmp\_index: Dict\[str, Set\[int\]\] \= {}

\# 先分配 dimension\_id（使用稳定哈希 ID）

for dimension\_name, info in catalog\_dimensions.items():

dim\_id \= \_stable\_id(dimension\_name) \# 稳定的整数 ID

aliases \= info.get("aliases", \[\]) or \[\]

all\_aliases \= \[dimension\_name\] + list(aliases)

all\_aliases \= list(dict.fromkeys(all\_aliases)) \# 去重

self.name\_to\_id\[dimension\_name\] \= dim\_id

self.id\_to\_doc\[dim\_id\] \= {

"dimension\_id": dim\_id,

"dimension\_name": dimension\_name,

"property\_name": info.get("property\_name", dimension\_name),

"property\_type": info.get("property\_type", "profile"),

"property\_compose\_type": info.get("property\_compose\_type", "origin"),

"aliases": all\_aliases,

}

self.id\_to\_norm\_aliases\[dim\_id\] \= \[\]

\# 加入 jieba 词典

if enable\_jieba\_userdict:

self.\_add\_to\_jieba\_dict(all\_aliases)

\# 再建 exact map + 倒排索引

for dim\_id, doc in self.id\_to\_doc.items():

self.\_build\_inverted\_index(dim\_id, doc\["aliases"\], tmp\_index)

self.token\_to\_ids \= self.\_finalize\_index(tmp\_index)

def \_get\_name\_by\_id(self, doc\_id: int) \-> str:

return self.id\_to\_doc\[doc\_id\]\["dimension\_name"\]

def match\_property(self, query: str) \-> tuple:

"""

匹配并返回 property\_name

:return: (property\_name, score, explain)

"""

result \= self.match(query)

if result.matched:

doc \= self.id\_to\_doc\[self.name\_to\_id\[result.matched\]\]

return doc\["property\_name"\], result.score, result.explain

return None, result.score, result.explain

\# =========================

\# 帮助函数：从 catalog 创建 matcher

\# =========================

def build\_dimension\_matcher\_from\_catalog(catalog\_dimensions: Dict\[str, Dict\[str, Any\]\]) \-> DimensionMatcher:

"""从 catalog 创建 DimensionMatcher"""

return DimensionMatcher(

catalog\_data\=catalog\_dimensions,

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

mock\_catalog\_dimensions \= {

"country": {

"aliases": \["country", "国家", "地区"\],

"property\_type": "profile",

"property\_name": "country",

"property\_compose\_type": "origin"

},

"city": {

"aliases": \["city", "城市"\],

"property\_type": "profile",

"property\_name": "city",

"property\_compose\_type": "origin"

},

"app\_version": {

"aliases": \["app\_version", "应用版本", "版本"\],

"property\_type": "profile",

"property\_name": "app\_version",

"property\_compose\_type": "origin"

},

"os": {

"aliases": \["os", "操作系统", "系统"\],

"property\_type": "profile",

"property\_name": "os",

"property\_compose\_type": "origin"

},

}

matcher \= build\_dimension\_matcher\_from\_catalog(mock\_catalog\_dimensions)

test\_queries \= \[

"country",

"国家",

"地区",

"城市",

"版本",

"应用版本",

"操作系统",

"os",

\]

for q in test\_queries:

result \= matcher.match(q)

prop\_name, score, explain \= matcher.match\_property(q)

print("=" \* 100)

print("query:", q)

print("matched dimension:", result.matched)

print("property\_name:", prop\_name)

print("score:", score)