# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_matchers.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_matchers.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_matchers.py)

Open commit details

History

258 lines (203 loc) · 7.34 KB

## FilesExpand file tree

master

/

# test\_matchers.py

Copy path

Top

## File metadata and controls

- Code

- Blame

258 lines (203 loc) · 7.34 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_matchers.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import pytest from common.text\_utils import ( contains\_chinese, is\_single\_chinese\_char, normalize, tokenize\_mixed, ) from matcher.dimension\_matcher import ( DimensionMatcher, build\_dimension\_matcher\_from\_catalog, ) from matcher.event\_matcher import EventMatcher, build\_event\_matcher\_from\_catalog from matcher.metric\_matcher import MetricMatcher, build\_metric\_matcher\_from\_catalog from matcher.time\_matcher import TimeMatcher # ========================= # 测试数据 # ========================= MOCK\_EVENTS = { "app\_launch": { "aliases": \["应用启动", "启动应用", "app启动", "打开app", "打开应用"\] }, "payment\_success": { "aliases": \["支付成功", "支付完成", "payment success"\] }, "video\_play\_start": { "aliases": \["开始播放", "视频开始播放", "video play start"\] }, } MOCK\_DIMENSIONS = { "country": { "aliases": \["country", "国家", "地区"\], "property\_name": "country", }, "city": { "aliases": \["city", "城市"\], "property\_name": "city", }, } MOCK\_METRICS = { "pv": { "aliases": \["PV", "浏览量", "访问量"\], }, "uv": { "aliases": \["UV", "独立访客", "访客数"\], }, } # ========================= # text\_utils 测试 # ========================= class TestTextUtils: """测试文本工具函数""" def test\_normalize\_lowercase(self): """测试小写转换""" assert normalize("APP\_LAUNCH") == "app launch" def test\_normalize\_camel\_case(self): """测试 camelCase 处理""" assert normalize("appLaunch") == "app launch" def test\_normalize\_underscore(self): """测试下划线处理""" assert normalize("app\_launch") == "app launch" def test\_normalize\_special\_chars(self): """测试特殊字符处理""" assert normalize("app@launch#test") == "app launch test" def test\_tokenize\_mixed\_english(self): """测试英文分词""" tokens = tokenize\_mixed("app launch test") assert "app" in tokens assert "launch" in tokens assert "test" in tokens def test\_tokenize\_mixed\_chinese(self): """测试中文分词""" tokens = tokenize\_mixed("应用启动") assert len(tokens) > 0 def test\_tokenize\_mixed\_combined(self): """测试中英混合分词""" tokens = tokenize\_mixed("app\_launch启动") assert "app" in tokens assert "launch" in tokens assert "启动" in tokens def test\_contains\_chinese\_true(self): """测试检测中文 - 包含中文""" assert contains\_chinese("应用启动") is True def test\_contains\_chinese\_false(self): """测试检测中文 - 不包含中文""" assert contains\_chinese("app launch") is False def test\_is\_single\_chinese\_char\_true(self): """测试单字中文 - 是单字""" assert is\_single\_chinese\_char("应") is True def test\_is\_single\_chinese\_char\_false(self): """测试单字中文 - 不是单字""" assert is\_single\_chinese\_char("应用") is False assert is\_single\_chinese\_char("app") is False # ========================= # EventMatcher 测试 # ========================= class TestEventMatcher: """测试事件匹配器""" @pytest.fixture def matcher(self): return build\_event\_matcher\_from\_catalog(MOCK\_EVENTS) def test\_exact\_match(self, matcher: EventMatcher): """测试精确匹配""" result = matcher.match("app\_launch") assert result.matched == "app\_launch" assert result.score == 100.0 def test\_alias\_match(self, matcher: EventMatcher): """测试别名匹配""" result = matcher.match("应用启动") assert result.matched == "app\_launch" assert result.score == 100.0 def test\_fuzzy\_match(self, matcher: EventMatcher): """测试模糊匹配""" result = matcher.match("app启动") assert result.matched is not None def test\_no\_match(self, matcher: EventMatcher): """测试无匹配""" result = matcher.match("xyz123不存在的") # 可能返回 None 或低分匹配 assert result.score < 100.0 # ========================= # DimensionMatcher 测试 # ========================= class TestDimensionMatcher: """测试维度匹配器""" @pytest.fixture def matcher(self): return build\_dimension\_matcher\_from\_catalog(MOCK\_DIMENSIONS) def test\_exact\_match(self, matcher: DimensionMatcher): """测试精确匹配""" result = matcher.match("country") assert result.matched == "country" def test\_chinese\_alias\_match(self, matcher: DimensionMatcher): """测试中文别名匹配""" result = matcher.match("国家") assert result.matched == "country" def test\_match\_property(self, matcher: DimensionMatcher): """测试匹配 property\_name""" prop\_name, score, explain = matcher.match\_property("country") assert prop\_name == "country" # ========================= # MetricMatcher 测试 # ========================= class TestMetricMatcher: """测试指标匹配器""" @pytest.fixture def matcher(self): return build\_metric\_matcher\_from\_catalog(MOCK\_METRICS) def test\_pv\_match(self, matcher: MetricMatcher): """测试 PV 匹配""" result = matcher.match("PV") assert result.matched == "pv" def test\_uv\_match(self, matcher: MetricMatcher): """测试 UV 匹配""" result = matcher.match("UV") assert result.matched == "uv" def test\_chinese\_alias\_match(self, matcher: MetricMatcher): """测试中文别名匹配""" result = matcher.match("浏览量") assert result.matched == "pv" # ========================= # TimeMatcher 测试 # ========================= class TestTimeMatcher: """测试时间匹配器""" @pytest.fixture def matcher(self): return TimeMatcher() def test\_last\_n\_days\_chinese(self, matcher: TimeMatcher): """测试中文时间表达式""" result = matcher.match("近7天") assert result.days == 7 assert result.time\_type == "last\_n\_days" def test\_last\_n\_days\_english(self, matcher: TimeMatcher): """测试英文时间表达式""" result = matcher.match("last 30 days") assert result.days == 30 assert result.time\_type == "last\_n\_days" def test\_yesterday(self, matcher: TimeMatcher): """测试昨天""" result = matcher.match("昨天") assert result.days == 1 assert result.time\_type == "yesterday" def test\_today(self, matcher: TimeMatcher): """测试今天""" result = matcher.match("今天") assert result.days == 1 assert result.time\_type == "today" def test\_this\_week(self, matcher: TimeMatcher): """测试本周""" result = matcher.match("本周") assert result.days == 7 assert result.time\_type == "this\_week" def test\_default(self, matcher: TimeMatcher): """测试默认值""" result = matcher.match("无效输入") assert result.days == 7 assert result.time\_type == "default" # ========================= # 运行测试 # ========================= if \_\_name\_\_ == "\_\_main\_\_": pytest.main(\[\_\_file\_\_, "-v"\])

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

150

151

152

153

154

155

185

186

187

188

189

190

191

192

193

194

195

196

197

198

199

200

201

202

203

204

205

206

207

208

209

210

211

212

213

214

215

216

217

218

219

220

221

222

223

224

225

226

227

228

229

230

231

232

233

234

235

236

237

238

239

240

241

242

243

244

245

246

247

248

249

250

251

252

253

254

255

256

257

258

from \_\_future\_\_ import annotations

import pytest

from common.text\_utils import (

contains\_chinese,

is\_single\_chinese\_char,

normalize,

tokenize\_mixed,

)

from matcher.dimension\_matcher import (

DimensionMatcher,

build\_dimension\_matcher\_from\_catalog,

)

from matcher.event\_matcher import EventMatcher, build\_event\_matcher\_from\_catalog

from matcher.metric\_matcher import MetricMatcher, build\_metric\_matcher\_from\_catalog

from matcher.time\_matcher import TimeMatcher

\# =========================

\# 测试数据

\# =========================

MOCK\_EVENTS \= {

"app\_launch": {

"aliases": \["应用启动", "启动应用", "app启动", "打开app", "打开应用"\]

},

"payment\_success": {

"aliases": \["支付成功", "支付完成", "payment success"\]

},

"video\_play\_start": {

"aliases": \["开始播放", "视频开始播放", "video play start"\]

},

}

MOCK\_DIMENSIONS \= {

"country": {

"aliases": \["country", "国家", "地区"\],

"property\_name": "country",

},

"city": {

"aliases": \["city", "城市"\],

"property\_name": "city",

},

}

MOCK\_METRICS \= {

"pv": {

"aliases": \["PV", "浏览量", "访问量"\],

},

"uv": {

"aliases": \["UV", "独立访客", "访客数"\],

},

}

\# =========================

\# text\_utils 测试

\# =========================

class TestTextUtils:

"""测试文本工具函数"""

def test\_normalize\_lowercase(self):

"""测试小写转换"""

assert normalize("APP\_LAUNCH") \== "app launch"

def test\_normalize\_camel\_case(self):

"""测试 camelCase 处理"""

assert normalize("appLaunch") \== "app launch"

def test\_normalize\_underscore(self):

"""测试下划线处理"""

assert normalize("app\_launch") \== "app launch"

def test\_normalize\_special\_chars(self):

"""测试特殊字符处理"""

assert normalize("app@launch#test") \== "app launch test"

def test\_tokenize\_mixed\_english(self):

"""测试英文分词"""

tokens \= tokenize\_mixed("app launch test")

assert "app" in tokens

assert "launch" in tokens

assert "test" in tokens

def test\_tokenize\_mixed\_chinese(self):

"""测试中文分词"""

tokens \= tokenize\_mixed("应用启动")

assert len(tokens) \> 0

def test\_tokenize\_mixed\_combined(self):

"""测试中英混合分词"""

tokens \= tokenize\_mixed("app\_launch启动")

assert "app" in tokens

assert "launch" in tokens

assert "启动" in tokens

def test\_contains\_chinese\_true(self):

"""测试检测中文 - 包含中文"""

assert contains\_chinese("应用启动") is True

def test\_contains\_chinese\_false(self):

"""测试检测中文 - 不包含中文"""

assert contains\_chinese("app launch") is False

def test\_is\_single\_chinese\_char\_true(self):

"""测试单字中文 - 是单字"""

assert is\_single\_chinese\_char("应") is True

def test\_is\_single\_chinese\_char\_false(self):

"""测试单字中文 - 不是单字"""

assert is\_single\_chinese\_char("应用") is False

assert is\_single\_chinese\_char("app") is False

\# =========================

\# EventMatcher 测试

\# =========================

class TestEventMatcher:

"""测试事件匹配器"""

@pytest.fixture

def matcher(self):

return build\_event\_matcher\_from\_catalog(MOCK\_EVENTS)

def test\_exact\_match(self, matcher: EventMatcher):

"""测试精确匹配"""

result \= matcher.match("app\_launch")

assert result.matched \== "app\_launch"

assert result.score \== 100.0

def test\_alias\_match(self, matcher: EventMatcher):

"""测试别名匹配"""

result \= matcher.match("应用启动")

assert result.matched \== "app\_launch"

assert result.score \== 100.0

def test\_fuzzy\_match(self, matcher: EventMatcher):

"""测试模糊匹配"""

result \= matcher.match("app启动")

assert result.matched is not None

def test\_no\_match(self, matcher: EventMatcher):

"""测试无匹配"""

result \= matcher.match("xyz123不存在的")

\# 可能返回 None 或低分匹配

assert result.score < 100.0

\# =========================

\# DimensionMatcher 测试

\# =========================

class TestDimensionMatcher:

@pytest.fixture

def matcher(self):

return build\_metric\_matcher\_from\_catalog(MOCK\_METRICS)

def test\_pv\_match(self, matcher: MetricMatcher):

"""测试 PV 匹配"""

result \= matcher.match("PV")

assert result.matched \== "pv"

def test\_uv\_match(self, matcher: MetricMatcher):

"""测试 UV 匹配"""

result \= matcher.match("UV")

assert result.matched \== "uv"

def test\_chinese\_alias\_match(self, matcher: MetricMatcher):

"""测试中文别名匹配"""

result \= matcher.match("浏览量")

assert result.matched \== "pv"

\# =========================

\# TimeMatcher 测试

\# =========================

class TestTimeMatcher:

"""测试时间匹配器"""

@pytest.fixture

def matcher(self):

return TimeMatcher()

def test\_last\_n\_days\_chinese(self, matcher: TimeMatcher):

"""测试中文时间表达式"""

result \= matcher.match("近7天")

assert result.days \== 7

assert result.time\_type \== "last\_n\_days"

def test\_last\_n\_days\_english(self, matcher: TimeMatcher):

"""测试英文时间表达式"""

result \= matcher.match("last 30 days")

assert result.days \== 30

assert result.time\_type \== "last\_n\_days"

def test\_yesterday(self, matcher: TimeMatcher):

"""测试昨天"""

result \= matcher.match("昨天")

assert result.days \== 1

assert result.time\_type \== "yesterday"

def test\_today(self, matcher: TimeMatcher):

"""测试今天"""

result \= matcher.match("今天")

assert result.days \== 1

assert result.time\_type \== "today"

def test\_this\_week(self, matcher: TimeMatcher):

"""测试本周"""

result \= matcher.match("本周")

assert result.days \== 7

assert result.time\_type \== "this\_week"

def test\_default(self, matcher: TimeMatcher):

"""测试默认值"""

result \= matcher.match("无效输入")

assert result.days \== 7

assert result.time\_type \== "default"

\# =========================

\# 运行测试

\# =========================

if \_\_name\_\_ \== "\_\_main\_\_":

pytest.main(\[\_\_file\_\_, "-v"\])