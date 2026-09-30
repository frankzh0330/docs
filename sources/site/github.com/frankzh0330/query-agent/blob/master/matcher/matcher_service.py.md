# Source: https://github.com/frankzh0330/query-agent/blob/master/matcher/matcher_service.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# matcher\_service.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add TaskManager confirmation flow and rebuild test suite (138 t…](https://github.com/frankzh0330/query-agent/commit/68792e59e016c780e1c83c89e4205a0ab5e4fafb)

Open commit details

Apr 14, 2026

[68792e5](https://github.com/frankzh0330/query-agent/commit/68792e59e016c780e1c83c89e4205a0ab5e4fafb) · Apr 14, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/matcher/matcher_service.py)

Open commit details

History

201 lines (168 loc) · 7.88 KB

## FilesExpand file tree

master

/

# matcher\_service.py

Copy path

Top

## File metadata and controls

- Code

- Blame

201 lines (168 loc) · 7.88 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/matcher/matcher_service.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import logging from dataclasses import dataclass, field from typing import Any, Dict, List, Optional, Tuple from common.types import MatcherType from matcher.catalog\_loader import Catalog, load\_catalog from matcher.dimension\_matcher import build\_dimension\_matcher\_from\_catalog from matcher.event\_matcher import build\_event\_matcher\_from\_catalog from matcher.metric\_matcher import build\_metric\_matcher\_from\_catalog from matcher.time\_matcher import TimeMatcher, resolve\_last\_n\_days logger = logging.getLogger(\_\_name\_\_) @dataclass class ResolvedResult: """解析结果（含候选列表和置信度）""" value: str # 最终值（matched 或 default） score: float # 置信度 (0-100) method: str # "exact" | "fuzzy" | "default" candidates: List\[Dict\[str, Any\]\] = field(default\_factory=list) needs\_confirmation: bool = False # 是否需要用户确认 # 需要确认的置信度区间 \_CONFIRM\_SCORE\_HIGH = 80.0 \_CONFIRM\_SCORE\_LOW = 40.0 class MatcherService: """ Matcher 服务类，统一管理所有 matcher 替代 router.py 的全局变量模式，使用面向对象设计 """ def \_\_init\_\_(self, catalog\_path: str = "catalog"): """初始化服务，构建所有 matcher 索引""" logger.info("Initializing MatcherService...") # 加载 catalog self.catalog: Catalog = load\_catalog(catalog\_path) # 构建 matcher 索引 logger.info("Building matcher indexes...") self.event\_matcher = build\_event\_matcher\_from\_catalog(self.catalog.events) self.metric\_matcher = build\_metric\_matcher\_from\_catalog(self.catalog.metrics) self.dimension\_matcher = build\_dimension\_matcher\_from\_catalog(self.catalog.dimensions) self.time\_matcher = TimeMatcher() logger.info("MatcherService initialized successfully!") # ==================== 匹配方法 ==================== def match\_event(self, query: str, default: str = "app\_launch") -> Tuple\[str, Dict\[str, Any\]\]: """匹配事件""" result = self.event\_matcher.match(query) if result.matched: return result.matched, result.explain return default, {"method": "fallback", "default": default} def match\_metric(self, query: str, default: str = "pv") -> Tuple\[str, Dict\[str, Any\]\]: """匹配指标""" result = self.metric\_matcher.match(query) if result.matched: return result.matched, result.explain return default, {"method": "fallback", "default": default} def match\_dimension(self, query: str, default: str = "country") -> Tuple\[str, Dict\[str, Any\]\]: """匹配维度""" result = self.dimension\_matcher.match(query) if result.matched: return result.matched, result.explain return default, {"method": "fallback", "default": default} def resolve\_time(self, extraction: Any) -> Tuple\[int, Dict\[str, Any\]\]: """解析时间范围""" return resolve\_last\_n\_days(extraction) # ==================== 批量解析方法 ==================== def resolve\_from\_extractions( self, matcher\_type: MatcherType, extractions: list, default: str ) -> Tuple\[str, Dict\[str, Any\]\]: """ 从 LLM 提取结果中解析 :param matcher\_type: MatcherType 枚举 (EVENT/METRIC/DIMENSION) :param extractions: LLM 提取结果列表 :param default: 默认值 """ if not extractions: return default, {"method": "no\_extractions", "default": default} first = extractions\[0\] query\_text = first.text if hasattr(first, "text") else first.get("text", "") if matcher\_type == MatcherType.EVENT: return self.match\_event(query\_text, default) elif matcher\_type == MatcherType.METRIC: return self.match\_metric(query\_text, default) elif matcher\_type == MatcherType.DIMENSION: return self.match\_dimension(query\_text, default) else: raise ValueError(f"Unknown matcher\_type: {matcher\_type}") # ==================== 带候选的解析方法 ==================== def resolve\_with\_candidates( self, matcher\_type: MatcherType, extractions: list, default: str ) -> ResolvedResult: """从 LLM 提取结果中解析，返回完整候选和置信度 判断规则（确定性代码，不依赖 LLM）： - score >= 80 → 直接用，不需确认 - score 在 40-80 → 需要用户确认 - score < 40 → 直接用 default，不确认（候选太模糊没价值） """ if not extractions: return ResolvedResult( value=default, score=0.0, method="no\_extractions", candidates=\[\], needs\_confirmation=False, ) first = extractions\[0\] query\_text = first.text if hasattr(first, "text") else first.get("text", "") # 获取 MatchResult（含 score 和 top5 候选） matcher = self.\_get\_matcher(matcher\_type) result = matcher.match(query\_text) # 从 explain 中提取 top5 候选 candidates = self.\_extract\_candidates(result) if result.matched: # 有匹配结果 if result.score >= \_CONFIRM\_SCORE\_HIGH: return ResolvedResult( value=result.matched, score=result.score, method="exact" if result.score == 100.0 else "fuzzy", candidates=candidates, needs\_confirmation=False, ) elif result.score >= \_CONFIRM\_SCORE\_LOW: # 低置信度但有候选 → 需要确认 return ResolvedResult( value=result.matched, score=result.score, method="fuzzy\_low\_confidence", candidates=candidates, needs\_confirmation=True, ) else: # 分数太低，用 default return ResolvedResult( value=default, score=result.score, method="score\_too\_low", candidates=candidates, needs\_confirmation=False, ) else: # 完全没匹配 if candidates and result.score >= \_CONFIRM\_SCORE\_LOW: # 有候选但没过 threshold → 需要确认 return ResolvedResult( value=default, score=result.score, method="below\_threshold", candidates=candidates, needs\_confirmation=True, ) return ResolvedResult( value=default, score=result.score, method="no\_match", candidates=candidates, needs\_confirmation=False, ) def \_get\_matcher(self, matcher\_type: MatcherType): if matcher\_type == MatcherType.EVENT: return self.event\_matcher elif matcher\_type == MatcherType.METRIC: return self.metric\_matcher elif matcher\_type == MatcherType.DIMENSION: return self.dimension\_matcher raise ValueError(f"Unknown matcher\_type: {matcher\_type}") @staticmethod def \_extract\_candidates(result) -> List\[Dict\[str, Any\]\]: """从 MatchResult.explain 中提取 top5 候选""" rerank\_explain = result.explain.get("rerank\_explain", {}) top5 = rerank\_explain.get("top5", \[\]) recall\_explain = result.explain.get("recall\_explain", {}) top\_recall = recall\_explain.get("top\_candidates", \[\]) # 优先用 rerank 的 top5（更精确），fallback 到 recall 的 top if top5: return \[ {"value": c\["name"\], "score": c\["score"\]} for c in top5 \] if top\_recall: return \[ {"value": c\["name"\], "score": float(c.get("hit\_count", 0))} for c in top\_recall\[:5\] \] return \[\]

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

156

157

158

159

160

161

162

163

164

165

166

167

168

169

170

171

172

173

174

175

176

177

178

179

180

181

182

183

184

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

from \_\_future\_\_ import annotations

import logging

from dataclasses import dataclass, field

from typing import Any, Dict, List, Optional, Tuple

from common.types import MatcherType

from matcher.catalog\_loader import Catalog, load\_catalog

from matcher.dimension\_matcher import build\_dimension\_matcher\_from\_catalog

from matcher.event\_matcher import build\_event\_matcher\_from\_catalog

from matcher.metric\_matcher import build\_metric\_matcher\_from\_catalog

from matcher.time\_matcher import TimeMatcher, resolve\_last\_n\_days

logger \= logging.getLogger(\_\_name\_\_)

@dataclass

class ResolvedResult:

"""解析结果（含候选列表和置信度）"""

value: str \# 最终值（matched 或 default）

score: float \# 置信度 (0-100)

method: str \# "exact" | "fuzzy" | "default"

candidates: List\[Dict\[str, Any\]\] \= field(default\_factory\=list)

needs\_confirmation: bool \= False \# 是否需要用户确认

\# 需要确认的置信度区间

\_CONFIRM\_SCORE\_HIGH \= 80.0

\_CONFIRM\_SCORE\_LOW \= 40.0

class MatcherService:

"""

Matcher 服务类，统一管理所有 matcher

替代 router.py 的全局变量模式，使用面向对象设计

"""

def \_\_init\_\_(self, catalog\_path: str \= "catalog"):

"""初始化服务，构建所有 matcher 索引"""

logger.info("Initializing MatcherService...")

\# 加载 catalog

self.catalog: Catalog \= load\_catalog(catalog\_path)

\# 构建 matcher 索引

logger.info("Building matcher indexes...")

self.event\_matcher \= build\_event\_matcher\_from\_catalog(self.catalog.events)

self.metric\_matcher \= build\_metric\_matcher\_from\_catalog(self.catalog.metrics)

self.dimension\_matcher \= build\_dimension\_matcher\_from\_catalog(self.catalog.dimensions)

self.time\_matcher \= TimeMatcher()

logger.info("MatcherService initialized successfully!")

\# ==================== 匹配方法 ====================

def match\_event(self, query: str, default: str \= "app\_launch") \-> Tuple\[str, Dict\[str, Any\]\]:

"""匹配事件"""

result \= self.event\_matcher.match(query)

if result.matched:

return result.matched, result.explain

return default, {"method": "fallback", "default": default}

def match\_metric(self, query: str, default: str \= "pv") \-> Tuple\[str, Dict\[str, Any\]\]:

"""匹配指标"""

result \= self.metric\_matcher.match(query)

if result.matched:

return result.matched, result.explain

return default, {"method": "fallback", "default": default}

def match\_dimension(self, query: str, default: str \= "country") \-> Tuple\[str, Dict\[str, Any\]\]:

"""匹配维度"""

result \= self.dimension\_matcher.match(query)

if result.matched:

return result.matched, result.explain

return default, {"method": "fallback", "default": default}

def resolve\_time(self, extraction: Any) \-> Tuple\[int, Dict\[str, Any\]\]:

"""解析时间范围"""

return resolve\_last\_n\_days(extraction)

\# ==================== 批量解析方法 ====================

def resolve\_from\_extractions(

self, matcher\_type: MatcherType, extractions: list, default: str

) \-> Tuple\[str, Dict\[str, Any\]\]:

"""

从 LLM 提取结果中解析

:param matcher\_type: MatcherType 枚举 (EVENT/METRIC/DIMENSION)

:param extractions: LLM 提取结果列表

:param default: 默认值

"""

if not extractions:

return default, {"method": "no\_extractions", "default": default}

first \= extractions\[0\]

query\_text \= first.text if hasattr(first, "text") else first.get("text", "")

if matcher\_type \== MatcherType.EVENT:

return self.match\_event(query\_text, default)

elif matcher\_type \== MatcherType.METRIC:

return self.match\_metric(query\_text, default)

elif matcher\_type \== MatcherType.DIMENSION:

return self.match\_dimension(query\_text, default)

else:

raise ValueError(f"Unknown matcher\_type: {matcher\_type}")

\# ==================== 带候选的解析方法 ====================

def resolve\_with\_candidates(

self, matcher\_type: MatcherType, extractions: list, default: str

) \-> ResolvedResult:

"""从 LLM 提取结果中解析，返回完整候选和置信度

判断规则（确定性代码，不依赖 LLM）：

\- score >= 80 → 直接用，不需确认

\- score 在 40-80 → 需要用户确认

\- score < 40 → 直接用 default，不确认（候选太模糊没价值）

"""

if not extractions:

return ResolvedResult(

value\=default, score\=0.0, method\="no\_extractions",

candidates\=\[\], needs\_confirmation\=False,

)

first \= extractions\[0\]

query\_text \= first.text if hasattr(first, "text") else first.get("text", "")

\# 获取 MatchResult（含 score 和 top5 候选）

matcher \= self.\_get\_matcher(matcher\_type)

result \= matcher.match(query\_text)

\# 从 explain 中提取 top5 候选

candidates \= self.\_extract\_candidates(result)

if result.matched:

\# 有匹配结果

if result.score \>= \_CONFIRM\_SCORE\_HIGH:

return ResolvedResult(

value\=result.matched, score\=result.score,

method\="exact" if result.score \== 100.0 else "fuzzy",

candidates\=candidates, needs\_confirmation\=False,

)

elif result.score \>= \_CONFIRM\_SCORE\_LOW:

\# 低置信度但有候选 → 需要确认

return ResolvedResult(

value\=result.matched, score\=result.score,

method\="fuzzy\_low\_confidence",

candidates\=candidates, needs\_confirmation\=True,

)

else:

\# 分数太低，用 default

return ResolvedResult(

value\=default, score\=result.score,

method\="score\_too\_low",

candidates\=candidates, needs\_confirmation\=False,

)

else:

\# 完全没匹配

if candidates and result.score \>= \_CONFIRM\_SCORE\_LOW:

\# 有候选但没过 threshold → 需要确认

return ResolvedResult(

value\=default, score\=result.score,

method\="below\_threshold",

candidates\=candidates, needs\_confirmation\=True,

)

return ResolvedResult(

value\=default, score\=result.score,

method\="no\_match",

candidates\=candidates, needs\_confirmation\=False,

)

def \_get\_matcher(self, matcher\_type: MatcherType):

if matcher\_type \== MatcherType.EVENT:

return self.event\_matcher

elif matcher\_type \== MatcherType.METRIC:

return self.metric\_matcher

elif matcher\_type \== MatcherType.DIMENSION:

return self.dimension\_matcher

raise ValueError(f"Unknown matcher\_type: {matcher\_type}")

@staticmethod

def \_extract\_candidates(result) \-> List\[Dict\[str, Any\]\]:

"""从 MatchResult.explain 中提取 top5 候选"""

rerank\_explain \= result.explain.get("rerank\_explain", {})

top5 \= rerank\_explain.get("top5", \[\])

recall\_explain \= result.explain.get("recall\_explain", {})

top\_recall \= recall\_explain.get("top\_candidates", \[\])

\# 优先用 rerank 的 top5（更精确），fallback 到 recall 的 top

if top5:

return \[

{"value": c\["name"\], "score": c\["score"\]}

for c in top5

\]

if top\_recall:

return \[

{"value": c\["name"\], "score": float(c.get("hit\_count", 0))}

for c in top\_recall\[:5\]

\]

return \[\]