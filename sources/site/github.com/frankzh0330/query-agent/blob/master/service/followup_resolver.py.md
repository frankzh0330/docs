# Source: https://github.com/frankzh0330/query-agent/blob/master/service/followup_resolver.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# followup\_resolver.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add multi-turn follow-up query pipeline and extract QueryOrches…](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4)

Open commit details

May 1, 2026

[3560e95](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4) · May 1, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/service/followup_resolver.py)

Open commit details

History

224 lines (190 loc) · 6.71 KB

## FilesExpand file tree

master

/

# followup\_resolver.py

Copy path

Top

## File metadata and controls

- Code

- Blame

224 lines (190 loc) · 6.71 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/followup_resolver.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Turn-based follow-up query detection.""" from \_\_future\_\_ import annotations from dataclasses import dataclass, field from typing import Any from service.session\_models import QueryState # 明确的 follow-up 信号，通常表示“在上一轮基础上修改” \_FOLLOWUP\_PREFIXES = ( "换成", "改成", "改为", "再按", "再看", "那", "还是", "也", "继续", "顺便", "然后", "对比", "比较", ) \_REGION\_TOKENS = ("美国", "德国", "欧洲", "新加坡", "row", "usttp", "euttp") \_GROUP\_BY\_TOKENS = ("渠道", "国家", "地区", "城市", "平台") \_METRIC\_TOKENS = ("uv", "pv", "arpu", "gmv", "留存", "转化") \_FOLLOWUP\_SUFFIXES = ("呢", "吧", "一下", "看看", "再来") \_COMPARISON\_PATTERNS = ("和昨天比", "和今天比", "和上周比", "同比", "环比") \_TIME\_ONLY\_TERMS = { "今天": {"time\_range": {"type": "last\_n\_days", "n": 1}}, "昨天": {"time\_range": {"type": "last\_n\_days", "n": 1}}, "本周": {"time\_range": {"type": "last\_n\_days", "n": 7}}, "这周": {"time\_range": {"type": "last\_n\_days", "n": 7}}, "本月": {"time\_range": {"type": "last\_n\_days", "n": 30}}, "这个月": {"time\_range": {"type": "last\_n\_days", "n": 30}}, } @dataclass class FollowupDecision: """当前轮次是否应视为上一轮的 follow-up。""" mode: str # "new\_query" | "followup\_patch" | "confirmation\_reply" confidence: float is\_followup: bool patch\_hints: dict\[str, Any\] = field(default\_factory=dict) reason: str = "" matched\_signals: list\[str\] = field(default\_factory=list) normalized\_text: str = "" def detect\_followup( text: str, last\_query\_state: QueryState | None, pending\_task: bool = False, ) -> FollowupDecision: """判断本轮是否为 follow-up。 该函数只做确定性规则判断，不依赖 LLM。 """ normalized = (text or "").strip() if not normalized: return FollowupDecision( mode="new\_query", confidence=0.0, is\_followup=False, reason="empty\_input", normalized\_text=normalized, ) if pending\_task and \_looks\_like\_confirmation\_reply(normalized): return FollowupDecision( mode="confirmation\_reply", confidence=0.98, is\_followup=False, reason="pending\_task\_confirmation\_reply", matched\_signals=\["confirmation\_reply"\], normalized\_text=normalized, ) if last\_query\_state is None: return FollowupDecision( mode="new\_query", confidence=0.05, is\_followup=False, reason="no\_previous\_state", normalized\_text=normalized, ) if normalized in \_TIME\_ONLY\_TERMS: return FollowupDecision( mode="followup\_patch", confidence=0.95, is\_followup=True, patch\_hints=\_TIME\_ONLY\_TERMS\[normalized\], reason="time\_only\_term", matched\_signals=\["time\_only\_term"\], normalized\_text=normalized, ) if any(normalized.startswith(prefix) for prefix in \_FOLLOWUP\_PREFIXES): return FollowupDecision( mode="followup\_patch", confidence=0.9, is\_followup=True, patch\_hints={}, reason="followup\_prefix", matched\_signals=\_collect\_patch\_signals(normalized), normalized\_text=normalized, ) if any(pattern in normalized for pattern in \_COMPARISON\_PATTERNS): return FollowupDecision( mode="followup\_patch", confidence=0.88, is\_followup=True, patch\_hints={}, reason="comparison\_phrase", matched\_signals=\_collect\_patch\_signals(normalized), normalized\_text=normalized, ) contextual\_signals = \_collect\_patch\_signals(normalized) if \_looks\_like\_contextual\_patch(normalized, contextual\_signals): return FollowupDecision( mode="followup\_patch", confidence=0.82, is\_followup=True, patch\_hints={}, reason="contextual\_patch\_like\_input", matched\_signals=contextual\_signals, normalized\_text=normalized, ) if len(normalized) <= 12 and \_looks\_like\_short\_patch(normalized): return FollowupDecision( mode="followup\_patch", confidence=0.75, is\_followup=True, patch\_hints={}, reason="short\_patch\_like\_input", matched\_signals=\_collect\_patch\_signals(normalized), normalized\_text=normalized, ) return FollowupDecision( mode="new\_query", confidence=0.2, is\_followup=False, reason="looks\_like\_new\_query", normalized\_text=normalized, ) def \_looks\_like\_confirmation\_reply(text: str) -> bool: if text.isdigit(): return True lowered = text.lower() confirmation\_terms = {"第一个", "第二个", "第1个", "第2个", "就是这个", "选这个"} if lowered in confirmation\_terms: return True return False def \_looks\_like\_short\_patch(text: str) -> bool: patch\_keywords = ( "昨天", "今天", "本周", "本月", "uv", "pv", "美国", "德国", "欧洲", "渠道", "国家", "地区", ) lowered = text.lower() return any(keyword.lower() in lowered for keyword in patch\_keywords) def \_collect\_patch\_signals(text: str) -> list\[str\]: lowered = text.lower() signals: list\[str\] = \[\] if any(token in lowered for token in \_REGION\_TOKENS): signals.append("region\_token") if any(token in lowered for token in \_GROUP\_BY\_TOKENS): signals.append("group\_by\_token") if any(token in lowered for token in \_METRIC\_TOKENS): signals.append("metric\_token") if any(token in text for token in \_FOLLOWUP\_SUFFIXES): signals.append("followup\_suffix") if any(token in text for token in ("按", "拆", "看", "对比", "比较")): signals.append("patch\_verb") if any(token in text for token in \_COMPARISON\_PATTERNS): signals.append("comparison\_phrase") return signals def \_looks\_like\_contextual\_patch(text: str, signals: list\[str\]) -> bool: if not signals: return False short\_enough = len(text) <= 18 if short\_enough and any(signal in signals for signal in ("region\_token", "metric\_token", "group\_by\_token")): return True if "comparison\_phrase" in signals: return True if "followup\_suffix" in signals and any( signal in signals for signal in ("region\_token", "metric\_token", "group\_by\_token", "patch\_verb") ): return True return False

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

"""Turn-based follow-up query detection."""

from \_\_future\_\_ import annotations

from dataclasses import dataclass, field

from typing import Any

from service.session\_models import QueryState

\# 明确的 follow-up 信号，通常表示“在上一轮基础上修改”

\_FOLLOWUP\_PREFIXES \= (

"换成",

"改成",

"改为",

"再按",

"再看",

"那",

"还是",

"也",

"继续",

"顺便",

"然后",

"对比",

"比较",

)

\_REGION\_TOKENS \= ("美国", "德国", "欧洲", "新加坡", "row", "usttp", "euttp")

\_GROUP\_BY\_TOKENS \= ("渠道", "国家", "地区", "城市", "平台")

\_METRIC\_TOKENS \= ("uv", "pv", "arpu", "gmv", "留存", "转化")

\_FOLLOWUP\_SUFFIXES \= ("呢", "吧", "一下", "看看", "再来")

\_COMPARISON\_PATTERNS \= ("和昨天比", "和今天比", "和上周比", "同比", "环比")

\_TIME\_ONLY\_TERMS \= {

"今天": {"time\_range": {"type": "last\_n\_days", "n": 1}},

"昨天": {"time\_range": {"type": "last\_n\_days", "n": 1}},

"本周": {"time\_range": {"type": "last\_n\_days", "n": 7}},

"这周": {"time\_range": {"type": "last\_n\_days", "n": 7}},

"本月": {"time\_range": {"type": "last\_n\_days", "n": 30}},

"这个月": {"time\_range": {"type": "last\_n\_days", "n": 30}},

}

@dataclass

class FollowupDecision:

"""当前轮次是否应视为上一轮的 follow-up。"""

mode: str \# "new\_query" | "followup\_patch" | "confirmation\_reply"

confidence: float

is\_followup: bool

patch\_hints: dict\[str, Any\] \= field(default\_factory\=dict)

reason: str \= ""

matched\_signals: list\[str\] \= field(default\_factory\=list)

normalized\_text: str \= ""

def detect\_followup(

text: str,

last\_query\_state: QueryState | None,

pending\_task: bool \= False,

) \-> FollowupDecision:

"""判断本轮是否为 follow-up。

该函数只做确定性规则判断，不依赖 LLM。

"""

normalized \= (text or "").strip()

if not normalized:

return FollowupDecision(

mode\="new\_query",

confidence\=0.0,

is\_followup\=False,

reason\="empty\_input",

normalized\_text\=normalized,

)

if pending\_task and \_looks\_like\_confirmation\_reply(normalized):

return FollowupDecision(

mode\="confirmation\_reply",

confidence\=0.98,

is\_followup\=False,

reason\="pending\_task\_confirmation\_reply",

matched\_signals\=\["confirmation\_reply"\],

normalized\_text\=normalized,

)

if last\_query\_state is None:

return FollowupDecision(

mode\="new\_query",

confidence\=0.05,

is\_followup\=False,

reason\="no\_previous\_state",

normalized\_text\=normalized,

)

if normalized in \_TIME\_ONLY\_TERMS:

return FollowupDecision(

mode\="followup\_patch",

confidence\=0.95,

is\_followup\=True,

patch\_hints\=\_TIME\_ONLY\_TERMS\[normalized\],

reason\="time\_only\_term",

matched\_signals\=\["time\_only\_term"\],

normalized\_text\=normalized,

)

if any(normalized.startswith(prefix) for prefix in \_FOLLOWUP\_PREFIXES):

return FollowupDecision(

mode\="followup\_patch",

confidence\=0.9,

is\_followup\=True,

patch\_hints\={},

reason\="followup\_prefix",

matched\_signals\=\_collect\_patch\_signals(normalized),

normalized\_text\=normalized,

)

if any(pattern in normalized for pattern in \_COMPARISON\_PATTERNS):

return FollowupDecision(

mode\="followup\_patch",

confidence\=0.88,

is\_followup\=True,

patch\_hints\={},

reason\="comparison\_phrase",

matched\_signals\=\_collect\_patch\_signals(normalized),

normalized\_text\=normalized,

)

contextual\_signals \= \_collect\_patch\_signals(normalized)

if \_looks\_like\_contextual\_patch(normalized, contextual\_signals):

return FollowupDecision(

mode\="followup\_patch",

confidence\=0.82,

is\_followup\=True,

patch\_hints\={},

reason\="contextual\_patch\_like\_input",

matched\_signals\=contextual\_signals,

normalized\_text\=normalized,

)

if len(normalized) <= 12 and \_looks\_like\_short\_patch(normalized):

return FollowupDecision(

mode\="followup\_patch",

confidence\=0.75,

is\_followup\=True,

patch\_hints\={},

reason\="short\_patch\_like\_input",

matched\_signals\=\_collect\_patch\_signals(normalized),

normalized\_text\=normalized,

)

return FollowupDecision(

mode\="new\_query",

confidence\=0.2,

is\_followup\=False,

reason\="looks\_like\_new\_query",

normalized\_text\=normalized,

)

def \_looks\_like\_confirmation\_reply(text: str) \-> bool:

if text.isdigit():

return True

lowered \= text.lower()

confirmation\_terms \= {"第一个", "第二个", "第1个", "第2个", "就是这个", "选这个"}

if lowered in confirmation\_terms:

return True

return False

def \_looks\_like\_short\_patch(text: str) \-> bool:

patch\_keywords \= (

"昨天",

"今天",

"本周",

"本月",

"uv",

"pv",

"美国",

"德国",

"欧洲",

"渠道",

"国家",

"地区",

)

lowered \= text.lower()

return any(keyword.lower() in lowered for keyword in patch\_keywords)

def \_collect\_patch\_signals(text: str) \-> list\[str\]:

lowered \= text.lower()

signals: list\[str\] \= \[\]

if any(token in lowered for token in \_REGION\_TOKENS):

signals.append("region\_token")

if any(token in lowered for token in \_GROUP\_BY\_TOKENS):

signals.append("group\_by\_token")

if any(token in lowered for token in \_METRIC\_TOKENS):

signals.append("metric\_token")

if any(token in text for token in \_FOLLOWUP\_SUFFIXES):

signals.append("followup\_suffix")

if any(token in text for token in ("按", "拆", "看", "对比", "比较")):

signals.append("patch\_verb")

if any(token in text for token in \_COMPARISON\_PATTERNS):

signals.append("comparison\_phrase")

return signals

def \_looks\_like\_contextual\_patch(text: str, signals: list\[str\]) \-> bool:

if not signals:

return False

short\_enough \= len(text) <= 18

if short\_enough and any(signal in signals for signal in ("region\_token", "metric\_token", "group\_by\_token")):

return True

if "comparison\_phrase" in signals:

return True

if "followup\_suffix" in signals and any(

signal in signals for signal in ("region\_token", "metric\_token", "group\_by\_token", "patch\_verb")

):

return True

return False