# Source: https://github.com/frankzh0330/query-agent/blob/master/service/catalog_sync.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# catalog\_sync.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/service/catalog_sync.py)

Open commit details

History

229 lines (181 loc) · 7.73 KB

## FilesExpand file tree

master

/

# catalog\_sync.py

Copy path

Top

## File metadata and controls

- Code

- Blame

229 lines (181 loc) · 7.73 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/catalog_sync.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

""" 目录同步服务 - 从 HTTP API 获取 events/dimensions 数据并写入 YAML """ from \_\_future\_\_ import annotations import json import os from datetime import datetime from typing import Dict, List import httpx import yaml class CatalogSync: """目录同步服务""" def \_\_init\_\_(self, api\_base: str, catalog\_dir: str = "catalog"): self.api\_base = api\_base self.catalog\_dir = catalog\_dir self.state\_file = os.path.join(catalog\_dir, ".sync\_state.json") async def fetch\_events(self, since: str | None = None) -> List\[Dict\]: """获取 events 数据 Args: since: 时间戳，只返回 update\_time >= since 的数据 """ params = {} if since: params\["since"\] = since async with httpx.AsyncClient() as client: resp = await client.get(f"{self.api\_base}/events", params=params) resp.raise\_for\_status() return resp.json() async def fetch\_dimensions(self, since: str | None = None) -> List\[Dict\]: """获取 dimensions 数据 Args: since: 时间戳，只返回 update\_time >= since 的数据 """ params = {} if since: params\["since"\] = since async with httpx.AsyncClient() as client: resp = await client.get(f"{self.api\_base}/dimensions", params=params) resp.raise\_for\_status() return resp.json() def \_parse\_aliases(self, alias\_str: str) -> List\[str\]: """解析逗号分隔的别名""" if not alias\_str: return \[\] return \[a.strip() for a in alias\_str.split(",") if a.strip()\] def \_load\_state(self) -> Dict: """加载同步状态""" if os.path.exists(self.state\_file): with open(self.state\_file) as f: return json.load(f) return {"last\_full\_sync\_events": None, "last\_incremental\_sync\_events": None, "last\_full\_sync\_dimensions": None, "last\_incremental\_sync\_dimensions": None} def \_save\_state(self, state: Dict): """保存同步状态""" os.makedirs(self.catalog\_dir, exist\_ok=True) with open(self.state\_file, "w") as f: json.dump(state, f, indent=2, ensure\_ascii=False) def \_load\_existing\_events(self) -> Dict: """加载现有 events.yaml""" path = os.path.join(self.catalog\_dir, "events.yaml") if os.path.exists(path): with open(path, encoding="utf-8") as f: data = yaml.safe\_load(f) or {} return data.get("events", {}) return {} def \_load\_existing\_dimensions(self) -> Dict: """加载现有 dimensions.yaml""" path = os.path.join(self.catalog\_dir, "dimensions.yaml") if os.path.exists(path): with open(path, encoding="utf-8") as f: data = yaml.safe\_load(f) or {} return data.get("dimensions", {}) return {} def \_write\_events(self, events: Dict): """写入 events.yaml""" os.makedirs(self.catalog\_dir, exist\_ok=True) path = os.path.join(self.catalog\_dir, "events.yaml") with open(path, "w", encoding="utf-8") as f: yaml.dump({"events": events}, f, allow\_unicode=True, default\_flow\_style=False, sort\_keys=False) def \_write\_dimensions(self, dimensions: Dict): """写入 dimensions.yaml""" os.makedirs(self.catalog\_dir, exist\_ok=True) path = os.path.join(self.catalog\_dir, "dimensions.yaml") with open(path, "w", encoding="utf-8") as f: yaml.dump({"dimensions": dimensions}, f, allow\_unicode=True, default\_flow\_style=False, sort\_keys=False) async def incremental\_sync\_events(self) -> int: """增量同步 events""" state = self.\_load\_state() since = state.get("last\_incremental\_sync\_events") items = await self.fetch\_events(since=since) existing = self.\_load\_existing\_events() # 更新/新增 for item in items: event\_name = item.get("event\_name") or item.get("name") if not event\_name: continue if item.get("status") == 0: # 删除 status=0 的 if event\_name in existing: del existing\[event\_name\] continue existing\[event\_name\] = { "aliases": self.\_parse\_aliases(item.get("alias", "")), "show\_name": item.get("show\_name", event\_name), "event\_type": item.get("event\_type", "origin"), "event\_name": event\_name, } # 写入 YAML self.\_write\_events(existing) # 更新状态 state\["last\_incremental\_sync\_events"\] = datetime.now().isoformat() self.\_save\_state(state) return len(items) async def full\_sync\_events(self) -> int: """全量同步 events""" items = await self.fetch\_events() events = {} for item in items: event\_name = item.get("event\_name") or item.get("name") if not event\_name: continue if item.get("status") == 0: continue # 跳过禁用的 events\[event\_name\] = { "aliases": self.\_parse\_aliases(item.get("alias", "")), "show\_name": item.get("show\_name", event\_name), "event\_type": item.get("event\_type", "origin"), "event\_name": event\_name, } self.\_write\_events(events) state = self.\_load\_state() state\["last\_full\_sync\_events"\] = datetime.now().isoformat() state\["last\_incremental\_sync\_events"\] = datetime.now().isoformat() self.\_save\_state(state) return len(events) async def incremental\_sync\_dimensions(self) -> int: """增量同步 dimensions""" state = self.\_load\_state() since = state.get("last\_incremental\_sync\_dimensions") items = await self.fetch\_dimensions(since=since) existing = self.\_load\_existing\_dimensions() for item in items: dim\_name = item.get("property\_name") or item.get("name") if not dim\_name: continue if item.get("status") == 0: if dim\_name in existing: del existing\[dim\_name\] continue existing\[dim\_name\] = { "aliases": self.\_parse\_aliases(item.get("alias", "")), "property\_type": item.get("property\_type", "profile"), "property\_name": dim\_name, "property\_compose\_type": item.get("property\_compose\_type", "origin"), } self.\_write\_dimensions(existing) state\["last\_incremental\_sync\_dimensions"\] = datetime.now().isoformat() self.\_save\_state(state) return len(items) async def full\_sync\_dimensions(self) -> int: """全量同步 dimensions""" items = await self.fetch\_dimensions() dimensions = {} for item in items: dim\_name = item.get("property\_name") or item.get("name") if not dim\_name: continue if item.get("status") == 0: continue dimensions\[dim\_name\] = { "aliases": self.\_parse\_aliases(item.get("alias", "")), "property\_type": item.get("property\_type", "profile"), "property\_name": dim\_name, "property\_compose\_type": item.get("property\_compose\_type", "origin"), } self.\_write\_dimensions(dimensions) state = self.\_load\_state() state\["last\_full\_sync\_dimensions"\] = datetime.now().isoformat() state\["last\_incremental\_sync\_dimensions"\] = datetime.now().isoformat() self.\_save\_state(state) return len(dimensions)

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

225

226

227

228

229

"""

目录同步服务 - 从 HTTP API 获取 events/dimensions 数据并写入 YAML

"""

from \_\_future\_\_ import annotations

import json

import os

from datetime import datetime

from typing import Dict, List

import httpx

import yaml

class CatalogSync:

"""目录同步服务"""

def \_\_init\_\_(self, api\_base: str, catalog\_dir: str \= "catalog"):

self.api\_base \= api\_base

self.catalog\_dir \= catalog\_dir

self.state\_file \= os.path.join(catalog\_dir, ".sync\_state.json")

async def fetch\_events(self, since: str | None \= None) \-> List\[Dict\]:

"""获取 events 数据

Args:

since: 时间戳，只返回 update\_time >= since 的数据

"""

params \= {}

if since:

params\["since"\] \= since

async with httpx.AsyncClient() as client:

resp \= await client.get(f"{self.api\_base}/events", params\=params)

resp.raise\_for\_status()

return resp.json()

async def fetch\_dimensions(self, since: str | None \= None) \-> List\[Dict\]:

"""获取 dimensions 数据

Args:

since: 时间戳，只返回 update\_time >= since 的数据

"""

params \= {}

if since:

params\["since"\] \= since

async with httpx.AsyncClient() as client:

resp \= await client.get(f"{self.api\_base}/dimensions", params\=params)

resp.raise\_for\_status()

return resp.json()

def \_parse\_aliases(self, alias\_str: str) \-> List\[str\]:

"""解析逗号分隔的别名"""

if not alias\_str:

return \[\]

return \[a.strip() for a in alias\_str.split(",") if a.strip()\]

def \_load\_state(self) \-> Dict:

"""加载同步状态"""

if os.path.exists(self.state\_file):

with open(self.state\_file) as f:

return json.load(f)

return {"last\_full\_sync\_events": None, "last\_incremental\_sync\_events": None,

"last\_full\_sync\_dimensions": None, "last\_incremental\_sync\_dimensions": None}

def \_save\_state(self, state: Dict):

"""保存同步状态"""

os.makedirs(self.catalog\_dir, exist\_ok\=True)

with open(self.state\_file, "w") as f:

json.dump(state, f, indent\=2, ensure\_ascii\=False)

def \_load\_existing\_events(self) \-> Dict:

"""加载现有 events.yaml"""

path \= os.path.join(self.catalog\_dir, "events.yaml")

if os.path.exists(path):

with open(path, encoding\="utf-8") as f:

data \= yaml.safe\_load(f) or {}

return data.get("events", {})

return {}

def \_load\_existing\_dimensions(self) \-> Dict:

"""加载现有 dimensions.yaml"""

path \= os.path.join(self.catalog\_dir, "dimensions.yaml")

if os.path.exists(path):

with open(path, encoding\="utf-8") as f:

data \= yaml.safe\_load(f) or {}

return data.get("dimensions", {})

return {}

def \_write\_events(self, events: Dict):

"""写入 events.yaml"""

os.makedirs(self.catalog\_dir, exist\_ok\=True)

path \= os.path.join(self.catalog\_dir, "events.yaml")

with open(path, "w", encoding\="utf-8") as f:

yaml.dump({"events": events}, f, allow\_unicode\=True, default\_flow\_style\=False, sort\_keys\=False)

def \_write\_dimensions(self, dimensions: Dict):

"""写入 dimensions.yaml"""

os.makedirs(self.catalog\_dir, exist\_ok\=True)

path \= os.path.join(self.catalog\_dir, "dimensions.yaml")

with open(path, "w", encoding\="utf-8") as f:

yaml.dump({"dimensions": dimensions}, f, allow\_unicode\=True, default\_flow\_style\=False, sort\_keys\=False)

async def incremental\_sync\_events(self) \-> int:

"""增量同步 events"""

state \= self.\_load\_state()

since \= state.get("last\_incremental\_sync\_events")

items \= await self.fetch\_events(since\=since)

existing \= self.\_load\_existing\_events()

\# 更新/新增

for item in items:

event\_name \= item.get("event\_name") or item.get("name")

if not event\_name:

continue

if item.get("status") \== 0:

\# 删除 status=0 的

if event\_name in existing:

del existing\[event\_name\]

continue

existing\[event\_name\] \= {

"aliases": self.\_parse\_aliases(item.get("alias", "")),

"show\_name": item.get("show\_name", event\_name),

"event\_type": item.get("event\_type", "origin"),

"event\_name": event\_name,

}

\# 写入 YAML

self.\_write\_events(existing)

\# 更新状态

state\["last\_incremental\_sync\_events"\] \= datetime.now().isoformat()

self.\_save\_state(state)

return len(items)

async def full\_sync\_events(self) \-> int:

"""全量同步 events"""

items \= await self.fetch\_events()

events \= {}

for item in items:

event\_name \= item.get("event\_name") or item.get("name")

if not event\_name:

continue

if item.get("status") \== 0:

continue \# 跳过禁用的

events\[event\_name\] \= {

"aliases": self.\_parse\_aliases(item.get("alias", "")),

"show\_name": item.get("show\_name", event\_name),

"event\_type": item.get("event\_type", "origin"),

"event\_name": event\_name,

}

self.\_write\_events(events)

state \= self.\_load\_state()

state\["last\_full\_sync\_events"\] \= datetime.now().isoformat()

state\["last\_incremental\_sync\_events"\] \= datetime.now().isoformat()

self.\_save\_state(state)

return len(events)

async def incremental\_sync\_dimensions(self) \-> int:

"""增量同步 dimensions"""

state \= self.\_load\_state()

since \= state.get("last\_incremental\_sync\_dimensions")

items \= await self.fetch\_dimensions(since\=since)

existing \= self.\_load\_existing\_dimensions()

for item in items:

dim\_name \= item.get("property\_name") or item.get("name")

if not dim\_name:

continue

if item.get("status") \== 0:

if dim\_name in existing:

del existing\[dim\_name\]

continue

existing\[dim\_name\] \= {

"aliases": self.\_parse\_aliases(item.get("alias", "")),

"property\_type": item.get("property\_type", "profile"),

"property\_name": dim\_name,

"property\_compose\_type": item.get("property\_compose\_type", "origin"),

}

self.\_write\_dimensions(existing)

state\["last\_incremental\_sync\_dimensions"\] \= datetime.now().isoformat()

self.\_save\_state(state)

return len(items)

async def full\_sync\_dimensions(self) \-> int:

"""全量同步 dimensions"""

items \= await self.fetch\_dimensions()

dimensions \= {}

for item in items:

dim\_name \= item.get("property\_name") or item.get("name")

if not dim\_name:

continue

if item.get("status") \== 0:

continue

dimensions\[dim\_name\] \= {

"aliases": self.\_parse\_aliases(item.get("alias", "")),

"property\_type": item.get("property\_type", "profile"),

"property\_name": dim\_name,

"property\_compose\_type": item.get("property\_compose\_type", "origin"),

}

self.\_write\_dimensions(dimensions)

state \= self.\_load\_state()

state\["last\_full\_sync\_dimensions"\] \= datetime.now().isoformat()

state\["last\_incremental\_sync\_dimensions"\] \= datetime.now().isoformat()

self.\_save\_state(state)

return len(dimensions)