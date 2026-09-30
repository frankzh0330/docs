# Source: https://github.com/frankzh0330/query-agent/blob/master/matcher/time_matcher.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# time\_matcher.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/matcher/time_matcher.py)

Open commit details

History

208 lines (180 loc) · 5.79 KB

## FilesExpand file tree

master

/

# time\_matcher.py

Copy path

Top

## File metadata and controls

- Code

- Blame

208 lines (180 loc) · 5.79 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/matcher/time_matcher.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import re from dataclasses import dataclass from typing import Any, Dict, Optional from service.llm\_extractions import ExtractionsJson @dataclass class TimeMatchResult: """时间匹配结果""" days: int time\_type: str # "last\_n\_days", "yesterday", "today", etc. explain: Dict\[str, Any\] class TimeMatcher: """ 时间匹配器，支持多种时间表达式 """ DEFAULT\_LAST\_N\_DAYS = 7 # 时间模式定义 PATTERNS = { "last\_n\_days": r"(最近|近|过去|last|past)\\s\*(\\d+)\\s\*(天|days?|d)", "yesterday": r"昨天|yesterday", "today": r"今天|today", "this\_week": r"本周|this week", "last\_week": r"上周|last week", "this\_month": r"本月|this month", "last\_month": r"上月|last month", } def match(self, query: str) -> TimeMatchResult: """ 匹配时间表达式 :param query: 时间表达式文本 :return: TimeMatchResult """ query = query.strip().lower() # 1. last\_n\_days 模式 match = re.search(self.PATTERNS\["last\_n\_days"\], query, re.IGNORECASE) if match: try: days = int(match.group(2)) return TimeMatchResult( days=days, time\_type="last\_n\_days", explain={ "method": "regex\_match", "input": query, "pattern": "last\_n\_days", "days": days, }, ) except ValueError: pass # 2. yesterday if re.search(self.PATTERNS\["yesterday"\], query, re.IGNORECASE): return TimeMatchResult( days=1, time\_type="yesterday", explain={ "method": "regex\_match", "input": query, "pattern": "yesterday", "days": 1, }, ) # 3. today if re.search(self.PATTERNS\["today"\], query, re.IGNORECASE): return TimeMatchResult( days=1, time\_type="today", explain={ "method": "regex\_match", "input": query, "pattern": "today", "days": 1, }, ) # 4. this\_week (7 days) if re.search(self.PATTERNS\["this\_week"\], query, re.IGNORECASE): return TimeMatchResult( days=7, time\_type="this\_week", explain={ "method": "regex\_match", "input": query, "pattern": "this\_week", "days": 7, }, ) # 5. last\_week (7 days) if re.search(self.PATTERNS\["last\_week"\], query, re.IGNORECASE): return TimeMatchResult( days=7, time\_type="last\_week", explain={ "method": "regex\_match", "input": query, "pattern": "last\_week", "days": 7, }, ) # 6. this\_month (30 days) if re.search(self.PATTERNS\["this\_month"\], query, re.IGNORECASE): return TimeMatchResult( days=30, time\_type="this\_month", explain={ "method": "regex\_match", "input": query, "pattern": "this\_month", "days": 30, }, ) # 7. last\_month (30 days) if re.search(self.PATTERNS\["last\_month"\], query, re.IGNORECASE): return TimeMatchResult( days=30, time\_type="last\_month", explain={ "method": "regex\_match", "input": query, "pattern": "last\_month", "days": 30, }, ) # 默认返回 return TimeMatchResult( days=self.DEFAULT\_LAST\_N\_DAYS, time\_type="default", explain={ "method": "fallback", "input": query, "default\_days": self.DEFAULT\_LAST\_N\_DAYS, }, ) def resolve\_last\_n\_days(extraction\_json: ExtractionsJson) -> tuple\[int, Dict\[str, Any\]\]: """ 从 LLM 提取结果中解析时间范围 :param extraction\_json: LLM 提取结果 :return: (days, explain) """ matcher = TimeMatcher() for extraction in extraction\_json.time\_extractions: text = extraction.text if hasattr(extraction, "text") else extraction.get("text", "") result = matcher.match(text) if result.time\_type != "default": return result.days, result.explain return matcher.DEFAULT\_LAST\_N\_DAYS, { "method": "fallback", "reason": "no valid time pattern found", "default": matcher.DEFAULT\_LAST\_N\_DAYS, } # ========================= # 本地测试 # ========================= if \_\_name\_\_ == "\_\_main\_\_": matcher = TimeMatcher() test\_queries = \[ "近7天", "最近30天", "过去3天", "last 7 days", "past 14 days", "昨天", "yesterday", "今天", "today", "本周", "this week", "上周", "last week", "本月", "this month", "上月", "last month", "无效的输入", \] for q in test\_queries: result = matcher.match(q) print("=" \* 60) print(f"query: {q}") print(f"days: {result.days}, type: {result.time\_type}") print(f"explain: {result.explain}")

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

202

203

204

205

206

207

208

from \_\_future\_\_ import annotations

import re

from dataclasses import dataclass

from typing import Any, Dict, Optional

from service.llm\_extractions import ExtractionsJson

@dataclass

class TimeMatchResult:

"""时间匹配结果"""

days: int

time\_type: str \# "last\_n\_days", "yesterday", "today", etc.

explain: Dict\[str, Any\]

class TimeMatcher:

"""

时间匹配器，支持多种时间表达式

"""

DEFAULT\_LAST\_N\_DAYS \= 7

\# 时间模式定义

PATTERNS \= {

"last\_n\_days": r"(最近|近|过去|last|past)\\s\*(\\d+)\\s\*(天|days?|d)",

"yesterday": r"昨天|yesterday",

"today": r"今天|today",

"this\_week": r"本周|this week",

"last\_week": r"上周|last week",

"this\_month": r"本月|this month",

"last\_month": r"上月|last month",

}

def match(self, query: str) \-> TimeMatchResult:

"""

匹配时间表达式

:param query: 时间表达式文本

:return: TimeMatchResult

"""

query \= query.strip().lower()

\# 1. last\_n\_days 模式

match \= re.search(self.PATTERNS\["last\_n\_days"\], query, re.IGNORECASE)

if match:

try:

days \= int(match.group(2))

return TimeMatchResult(

days\=days,

time\_type\="last\_n\_days",

explain\={

"method": "regex\_match",

"input": query,

"pattern": "last\_n\_days",

"days": days,

},

)

except ValueError:

pass

\# 2. yesterday

if re.search(self.PATTERNS\["yesterday"\], query, re.IGNORECASE):

return TimeMatchResult(

days\=1,

time\_type\="yesterday",

explain\={

"method": "regex\_match",

"input": query,

"pattern": "yesterday",

"days": 1,

},

)

\# 3. today

if re.search(self.PATTERNS\["today"\], query, re.IGNORECASE):

return TimeMatchResult(

days\=1,

time\_type\="today",

explain\={

"method": "regex\_match",

"input": query,

"pattern": "today",

"days": 1,

},

)

\# 4. this\_week (7 days)

if re.search(self.PATTERNS\["this\_week"\], query, re.IGNORECASE):

return TimeMatchResult(

days\=7,

time\_type\="this\_week",

explain\={

"method": "regex\_match",

"input": query,

"pattern": "this\_week",

"days": 7,

},

)

\# 5. last\_week (7 days)

if re.search(self.PATTERNS\["last\_week"\], query, re.IGNORECASE):

return TimeMatchResult(

days\=7,

time\_type\="last\_week",

explain\={

"method": "regex\_match",

"input": query,

"pattern": "last\_week",

"days": 7,

},

)

\# 6. this\_month (30 days)

if re.search(self.PATTERNS\["this\_month"\], query, re.IGNORECASE):

return TimeMatchResult(

days\=30,

time\_type\="this\_month",

explain\={

"method": "regex\_match",

"input": query,

"pattern": "this\_month",

"days": 30,

},

)

\# 7. last\_month (30 days)

if re.search(self.PATTERNS\["last\_month"\], query, re.IGNORECASE):

return TimeMatchResult(

days\=30,

time\_type\="last\_month",

explain\={

"method": "regex\_match",

"input": query,

"pattern": "last\_month",

"days": 30,

},

)

\# 默认返回

return TimeMatchResult(

days\=self.DEFAULT\_LAST\_N\_DAYS,

time\_type\="default",

explain\={

"method": "fallback",

"input": query,

"default\_days": self.DEFAULT\_LAST\_N\_DAYS,

},

)

def resolve\_last\_n\_days(extraction\_json: ExtractionsJson) \-> tuple\[int, Dict\[str, Any\]\]:

"""

从 LLM 提取结果中解析时间范围

:param extraction\_json: LLM 提取结果

:return: (days, explain)

"""

matcher \= TimeMatcher()

for extraction in extraction\_json.time\_extractions:

text \= extraction.text if hasattr(extraction, "text") else extraction.get("text", "")

result \= matcher.match(text)

if result.time\_type != "default":

return result.days, result.explain

return matcher.DEFAULT\_LAST\_N\_DAYS, {

"method": "fallback",

"reason": "no valid time pattern found",

"default": matcher.DEFAULT\_LAST\_N\_DAYS,

}

\# =========================

\# 本地测试

\# =========================

if \_\_name\_\_ \== "\_\_main\_\_":

matcher \= TimeMatcher()

test\_queries \= \[

"近7天",

"最近30天",

"过去3天",

"last 7 days",

"past 14 days",

"昨天",

"yesterday",

"今天",

"today",

"本周",

"this week",

"上周",

"last week",

"本月",

"this month",

"上月",

"last month",

"无效的输入",

\]

for q in test\_queries:

result \= matcher.match(q)

print("=" \* 60)

print(f"query: {q}")

print(f"days: {result.days}, type: {result.time\_type}")

print(f"explain: {result.explain}")