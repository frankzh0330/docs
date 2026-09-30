# Source: https://github.com/frankzh0330/query-agent/blob/master/mcp_server.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# mcp\_server.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/mcp_server.py)

Open commit details

History

222 lines (180 loc) · 5.53 KB

## FilesExpand file tree

master

/

# mcp\_server.py

Copy path

Top

## File metadata and controls

- Code

- Blame

222 lines (180 loc) · 5.53 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/mcp_server.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

#!/usr/bin/env python3 """ Query Agent MCP 服务器 提供三个调试命令： - /token <query> - 测试分词 - /recall <query> - 测试召回 - /rerank <query> - 测试重排 """ from \_\_future\_\_ import annotations import asyncio import json from typing import Any from mcp.server.fastmcp import FastMCP from common.text\_utils import normalize, tokenize\_mixed from matcher.event\_matcher import build\_event\_matcher\_from\_catalog from matcher.metric\_matcher import build\_metric\_matcher\_from\_catalog from matcher.dimension\_matcher import build\_dimension\_matcher\_from\_catalog from matcher.catalog\_loader import load\_catalog # 全局 matcher 实例 \_event\_matcher = None \_metric\_matcher = None \_dimension\_matcher = None def \_init\_matchers(): """初始化所有 matcher""" global \_event\_matcher, \_metric\_matcher, \_dimension\_matcher if \_event\_matcher is None: catalog = load\_catalog("catalog") \_event\_matcher = build\_event\_matcher\_from\_catalog(catalog.events) \_metric\_matcher = build\_metric\_matcher\_from\_catalog(catalog.metrics) \_dimension\_matcher = build\_dimension\_matcher\_from\_catalog(catalog.dimensions) def \_get\_matcher(matcher\_type: str): """根据类型获取 matcher""" \_init\_matchers() if matcher\_type == "event": return \_event\_matcher elif matcher\_type == "metric": return \_metric\_matcher elif matcher\_type == "dimension": return \_dimension\_matcher else: raise ValueError(f"Unknown matcher\_type: {matcher\_type}") # 创建 MCP 服务器 mcp = FastMCP("Query Agent") @mcp.tool() async def token( query: str, matcher\_type: str = "event", ) -> dict\[str, Any\]: """ 测试分词功能 Args: query: 要分词的查询文本 matcher\_type: matcher 类型 (event/metric/dimension) Returns: { "original": str, # 原始查询 "normalized": str, # 规范化后的查询 "tokens": list\[str\], # 分词结果 "expanded": list\[str\], # 同义词扩展后的 tokens } """ matcher = \_get\_matcher(matcher\_type) # 1. 分词 tokens = tokenize\_mixed(query) # 2. 同义词扩展 expanded\_tokens = matcher.\_expand\_query\_tokens(tokens) return { "original": query, "normalized": normalize(query), "tokens": tokens, "expanded": expanded\_tokens, } @mcp.tool() async def recall( query: str, matcher\_type: str = "event", ) -> dict\[str, Any\]: """ 测试召回功能 Args: query: 查询文本 matcher\_type: matcher 类型 (event/metric/dimension) Returns: { "query": str, "tokens": list\[str\], "expanded": list\[str\], "candidates": list, # 候选 ID 列表 "explain": dict, # 召回解释 } """ matcher = \_get\_matcher(matcher\_type) # 分词 + 扩展 tokens = tokenize\_mixed(query) expanded\_tokens = matcher.\_expand\_query\_tokens(tokens) # 召回 candidates, explain = matcher.\_recall\_candidates(expanded\_tokens) # 转换候选结果（包含名称） candidate\_details = \[\] for item in explain.get("top\_candidates", \[\]): doc\_id = item\["id"\] if doc\_id in candidates: candidate\_details.append({ "id": doc\_id, "name": item\["name"\], "hit\_count": item\["hit\_count"\], }) return { "query": query, "tokens": tokens, "expanded": expanded\_tokens, "candidates": candidates, "candidate\_details": candidate\_details, "explain": explain, } @mcp.tool() async def rerank( query: str, matcher\_type: str = "event", ) -> dict\[str, Any\]: """ 测试重排功能 Args: query: 查询文本 matcher\_type: matcher 类型 (event/metric/dimension) Returns: { "query": str, "tokens": list\[str\], "expanded": list\[str\], "candidates": list, # 候选 ID 列表 "result": dict | None, # 重排结果 } """ matcher = \_get\_matcher(matcher\_type) # 分词 + 扩展 tokens = tokenize\_mixed(query) expanded\_tokens = matcher.\_expand\_query\_tokens(tokens) # 召回 candidates, recall\_explain = matcher.\_recall\_candidates(expanded\_tokens) if not candidates: return { "query": query, "tokens": tokens, "expanded": expanded\_tokens, "candidates": \[\], "result": None, "recall\_explain": recall\_explain, } # 重排 rerank\_result = matcher.\_rerank(query, candidates) if rerank\_result is None: return { "query": query, "tokens": tokens, "expanded": expanded\_tokens, "candidates": candidates, "result": None, "recall\_explain": recall\_explain, } best\_id, best\_score, rerank\_explain = rerank\_result return { "query": query, "tokens": tokens, "expanded": expanded\_tokens, "candidates": candidates, "result": { "id": best\_id, "name": matcher.\_get\_name\_by\_id(best\_id), "score": best\_score, "explain": rerank\_explain, }, "recall\_explain": recall\_explain, } if \_\_name\_\_ == "\_\_main\_\_": # 初始化 matchers \_init\_matchers() print("Query Agent MCP Server initialized", file=sys.stderr) # 运行服务器 mcp.run()

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

#!/usr/bin/env python3

"""

Query Agent MCP 服务器

提供三个调试命令：

\- /token <query> - 测试分词

\- /recall <query> - 测试召回

\- /rerank <query> - 测试重排

"""

from \_\_future\_\_ import annotations

import asyncio

import json

from typing import Any

from mcp.server.fastmcp import FastMCP

from common.text\_utils import normalize, tokenize\_mixed

from matcher.event\_matcher import build\_event\_matcher\_from\_catalog

from matcher.metric\_matcher import build\_metric\_matcher\_from\_catalog

from matcher.dimension\_matcher import build\_dimension\_matcher\_from\_catalog

from matcher.catalog\_loader import load\_catalog

\# 全局 matcher 实例

\_event\_matcher \= None

\_metric\_matcher \= None

\_dimension\_matcher \= None

def \_init\_matchers():

"""初始化所有 matcher"""

global \_event\_matcher, \_metric\_matcher, \_dimension\_matcher

if \_event\_matcher is None:

catalog \= load\_catalog("catalog")

\_event\_matcher \= build\_event\_matcher\_from\_catalog(catalog.events)

\_metric\_matcher \= build\_metric\_matcher\_from\_catalog(catalog.metrics)

\_dimension\_matcher \= build\_dimension\_matcher\_from\_catalog(catalog.dimensions)

def \_get\_matcher(matcher\_type: str):

"""根据类型获取 matcher"""

\_init\_matchers()

if matcher\_type \== "event":

return \_event\_matcher

elif matcher\_type \== "metric":

return \_metric\_matcher

elif matcher\_type \== "dimension":

return \_dimension\_matcher

else:

raise ValueError(f"Unknown matcher\_type: {matcher\_type}")

\# 创建 MCP 服务器

mcp \= FastMCP("Query Agent")

@mcp.tool()

async def token(

query: str,

matcher\_type: str \= "event",

) \-> dict\[str, Any\]:

"""

测试分词功能

Args:

query: 要分词的查询文本

matcher\_type: matcher 类型 (event/metric/dimension)

Returns:

{

"original": str, # 原始查询

"normalized": str, # 规范化后的查询

"tokens": list\[str\], # 分词结果

"expanded": list\[str\], # 同义词扩展后的 tokens

}

"""

matcher \= \_get\_matcher(matcher\_type)

\# 1. 分词

tokens \= tokenize\_mixed(query)

\# 2. 同义词扩展

expanded\_tokens \= matcher.\_expand\_query\_tokens(tokens)

return {

"original": query,

"normalized": normalize(query),

"tokens": tokens,

"expanded": expanded\_tokens,

}

@mcp.tool()

async def recall(

query: str,

matcher\_type: str \= "event",

) \-> dict\[str, Any\]:

"""

测试召回功能

Args:

query: 查询文本

matcher\_type: matcher 类型 (event/metric/dimension)

Returns:

{

"query": str,

"tokens": list\[str\],

"expanded": list\[str\],

"candidates": list, # 候选 ID 列表

"explain": dict, # 召回解释

}

"""

matcher \= \_get\_matcher(matcher\_type)

\# 分词 + 扩展

tokens \= tokenize\_mixed(query)

expanded\_tokens \= matcher.\_expand\_query\_tokens(tokens)

\# 召回

candidates, explain \= matcher.\_recall\_candidates(expanded\_tokens)

\# 转换候选结果（包含名称）

candidate\_details \= \[\]

for item in explain.get("top\_candidates", \[\]):

doc\_id \= item\["id"\]

if doc\_id in candidates:

candidate\_details.append({

"id": doc\_id,

"name": item\["name"\],

"hit\_count": item\["hit\_count"\],

})

return {

"query": query,

"tokens": tokens,

"expanded": expanded\_tokens,

"candidates": candidates,

"candidate\_details": candidate\_details,

"explain": explain,

}

@mcp.tool()

async def rerank(

query: str,

matcher\_type: str \= "event",

) \-> dict\[str, Any\]:

"""

测试重排功能

Args:

query: 查询文本

matcher\_type: matcher 类型 (event/metric/dimension)

Returns:

{

"query": str,

"tokens": list\[str\],

"expanded": list\[str\],

"candidates": list, # 候选 ID 列表

"result": dict | None, # 重排结果

}

"""

matcher \= \_get\_matcher(matcher\_type)

\# 分词 + 扩展

tokens \= tokenize\_mixed(query)

expanded\_tokens \= matcher.\_expand\_query\_tokens(tokens)

\# 召回

candidates, recall\_explain \= matcher.\_recall\_candidates(expanded\_tokens)

if not candidates:

return {

"query": query,

"tokens": tokens,

"expanded": expanded\_tokens,

"candidates": \[\],

"result": None,

"recall\_explain": recall\_explain,

}

\# 重排

rerank\_result \= matcher.\_rerank(query, candidates)

if rerank\_result is None:

return {

"query": query,

"tokens": tokens,

"expanded": expanded\_tokens,

"candidates": candidates,

"result": None,

"recall\_explain": recall\_explain,

}

best\_id, best\_score, rerank\_explain \= rerank\_result

return {

"query": query,

"tokens": tokens,

"expanded": expanded\_tokens,

"candidates": candidates,

"result": {

"id": best\_id,

"name": matcher.\_get\_name\_by\_id(best\_id),

"score": best\_score,

"explain": rerank\_explain,

},

"recall\_explain": recall\_explain,

}

if \_\_name\_\_ \== "\_\_main\_\_":

\# 初始化 matchers

\_init\_matchers()

print("Query Agent MCP Server initialized", file\=sys.stderr)

\# 运行服务器

mcp.run()