# Source: https://github.com/frankzh0330/query-agent/blob/master/memory/long_term_memory.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# long\_term\_memory.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/memory/long_term_memory.py)

Open commit details

History

227 lines (192 loc) · 8.1 KB

## FilesExpand file tree

master

/

# long\_term\_memory.py

Copy path

Top

## File metadata and controls

- Code

- Blame

227 lines (192 loc) · 8.1 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/memory/long_term_memory.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""长期记忆管理器 读取 MEMORY.md 索引文件中引用的记忆文件，拼接后用于注入 system prompt。 按 project\_id 分桶，避免跨项目污染。 存储结构: data/memory/ ├── \_global/ # 所有项目共享 │ ├── MEMORY.md │ └── common\_corrections.md ├── project\_55/ # 只给 project\_id=55 注入 │ ├── MEMORY.md │ └── eu\_metrics.md └── project\_60/ # 只给 project\_id=60 注入 ├── MEMORY.md └── us\_events.md """ import logging import re from pathlib import Path from typing import Dict, Optional, Tuple logger = logging.getLogger(\_\_name\_\_) \_MAX\_MEMORY\_LINES = 50 \_MAX\_MEMORY\_BYTES = 5000 class LongTermMemory: """长期记忆 — 按 project\_id 分桶读取记忆文件并构建注入内容""" def \_\_init\_\_(self, data\_path: str = "data/memory"): self.data\_path = Path(data\_path) self.data\_path.mkdir(parents=True, exist\_ok=True) # 缓存: key=(project\_id,), value=拼接后的内容 self.\_cache: Dict\[Tuple\[int, ...\], str\] = {} self.\_cache\_mtime: Dict\[Tuple\[int, ...\], float\] = {} def load\_memory\_context( self, project\_id: Optional\[int\] = None, \*, query\_text: str | None = None, last\_query\_state: Optional\[Dict\[str, object\]\] = None, ) -> str: """加载记忆内容（用于注入 system prompt） 按 project\_id 分桶加载： 1. 始终加载 \_global/ 下的全局记忆 2. 如果指定了 project\_id，额外加载 project\_{id}/ 下的项目记忆 Args: project\_id: 项目ID，为 None 时只加载全局记忆 """ cache\_key = (project\_id,) if project\_id is not None else (None,) # 检查缓存 if cache\_key in self.\_cache and not self.\_is\_cache\_stale(cache\_key): logger.debug(f"LongTermMemory: cache hit for project={project\_id}") return self.\_cache\[cache\_key\] logger.debug(f"LongTermMemory: loading from disk for project={project\_id}") parts = \[\] # 1. 加载全局记忆 global\_dir = self.data\_path / "\_global" global\_parts = self.\_load\_dir(global\_dir) if global\_parts: parts.extend(global\_parts) # 2. 加载项目记忆 if project\_id is not None: project\_dir = self.data\_path / f"project\_{project\_id}" project\_parts = self.\_load\_dir(project\_dir) if project\_parts: parts.extend(project\_parts) if not parts: logger.debug(f"LongTermMemory: no memory found for project={project\_id}") return "" selected\_parts = self.\_select\_relevant\_parts(parts, query\_text=query\_text, last\_query\_state=last\_query\_state) result = "\\n\\n".join(selected\_parts) result = self.\_truncate(result) self.\_cache\[cache\_key\] = result self.\_cache\_mtime\[cache\_key\] = self.\_latest\_mtime(project\_id) logger.debug(f"LongTermMemory: loaded {len(result)} chars for project={project\_id}") return result def \_load\_dir(self, directory: Path) -> list\[str\]: """读取某个目录下的 MEMORY.md 索引及其引用的文件""" memory\_index = directory / "MEMORY.md" if not memory\_index.exists(): return \[\] return self.\_load\_all\_files(memory\_index) def \_load\_all\_files(self, memory\_index: Path) -> list\[str\]: """读取 MEMORY.md 索引中引用的所有文件内容""" try: index\_content = memory\_index.read\_text(encoding="utf-8").strip() except OSError: return \[\] base\_dir = memory\_index.parent parts = \[\] for line in index\_content.split("\\n"): match = re.match(r"- \\\[.+?\\\]\\((.+?)\\)", line) if match: file\_path = base\_dir / match.group(1) if file\_path.exists(): try: content = file\_path.read\_text(encoding="utf-8") # 去掉 frontmatter（--- ... ---） content = re.sub(r"^---\\n.\*?\\n---\\n", "", content, flags=re.DOTALL) stripped = content.strip() if stripped: parts.append(stripped) except OSError as e: logger.warning(f"Failed to read memory file {file\_path}: {e}") return parts def \_select\_relevant\_parts( self, parts: list\[str\], \*, query\_text: str | None = None, last\_query\_state: Optional\[Dict\[str, object\]\] = None, ) -> list\[str\]: if not query\_text and not last\_query\_state: return parts keywords = self.\_build\_keywords(query\_text=query\_text, last\_query\_state=last\_query\_state) if not keywords: return parts scored\_parts = \[\] for part in parts: lowered = part.lower() score = sum(1 for kw in keywords if kw and kw in lowered) scored\_parts.append((score, len(part), part)) matched = \[part for score, \_, part in sorted(scored\_parts, key=lambda item: (item\[0\], item\[1\]), reverse=True) if score > 0\] if matched: return matched\[:3\] # 没有命中时保守回退，避免完全丢失上下文 return parts\[:2\] @staticmethod def \_build\_keywords( \*, query\_text: str | None = None, last\_query\_state: Optional\[Dict\[str, object\]\] = None, ) -> list\[str\]: raw\_tokens: list\[str\] = \[\] if query\_text: normalized\_query = query\_text.lower() raw\_tokens.extend(re.findall(r"\[a-z0-9\_\]+", normalized\_query)) cjk\_chunks = re.findall(r"\[\\u4e00-\\u9fff\]+", normalized\_query) for chunk in cjk\_chunks: raw\_tokens.append(chunk) if len(chunk) <= 2: continue for idx in range(len(chunk) - 1): raw\_tokens.append(chunk\[idx : idx + 2\]) if last\_query\_state: for key in ("event", "metric"): value = last\_query\_state.get(key) if isinstance(value, str) and value: raw\_tokens.append(value.lower()) for key in ("group\_by", "region\_filter"): values = last\_query\_state.get(key) if isinstance(values, list): raw\_tokens.extend(str(value).lower() for value in values if value) seen = set() keywords = \[\] for token in raw\_tokens: if len(token) < 2: continue if token in seen: continue seen.add(token) keywords.append(token) return keywords def \_is\_cache\_stale(self, cache\_key: Tuple\[int, ...\]) -> bool: """检查相关目录的文件是否有更新""" project\_id = cache\_key\[0\] current\_mtime = self.\_latest\_mtime(project\_id) cached\_mtime = self.\_cache\_mtime.get(cache\_key, 0.0) return current\_mtime > cached\_mtime def \_latest\_mtime(self, project\_id: Optional\[int\] = None) -> float: """获取相关目录下所有 .md 文件的最新修改时间""" latest = 0.0 dirs = \[self.data\_path / "\_global"\] if project\_id is not None: dirs.append(self.data\_path / f"project\_{project\_id}") for d in dirs: try: for f in d.glob("\*.md"): latest = max(latest, f.stat().st\_mtime) except OSError: pass return latest @staticmethod def \_truncate( text: str, max\_lines: int = \_MAX\_MEMORY\_LINES, max\_bytes: int = \_MAX\_MEMORY\_BYTES, ) -> str: """截断保护，避免注入内容占用过多 token""" lines = text.split("\\n") if len(lines) > max\_lines: text = "\\n".join(lines\[:max\_lines\]) if len(text.encode("utf-8")) > max\_bytes: cut = text.rfind("\\n", 0, max\_bytes) text = text\[:cut\] if cut > 0 else text\[:max\_bytes\] return text

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

"""长期记忆管理器

读取 MEMORY.md 索引文件中引用的记忆文件，拼接后用于注入 system prompt。

按 project\_id 分桶，避免跨项目污染。

存储结构:

data/memory/

├── \_global/ # 所有项目共享

│ ├── MEMORY.md

│ └── common\_corrections.md

├── project\_55/ # 只给 project\_id=55 注入

│ ├── MEMORY.md

│ └── eu\_metrics.md

└── project\_60/ # 只给 project\_id=60 注入

├── MEMORY.md

└── us\_events.md

"""

import logging

import re

from pathlib import Path

from typing import Dict, Optional, Tuple

logger \= logging.getLogger(\_\_name\_\_)

\_MAX\_MEMORY\_LINES \= 50

\_MAX\_MEMORY\_BYTES \= 5000

class LongTermMemory:

"""长期记忆 — 按 project\_id 分桶读取记忆文件并构建注入内容"""

def \_\_init\_\_(self, data\_path: str \= "data/memory"):

self.data\_path \= Path(data\_path)

self.data\_path.mkdir(parents\=True, exist\_ok\=True)

\# 缓存: key=(project\_id,), value=拼接后的内容

self.\_cache: Dict\[Tuple\[int, ...\], str\] \= {}

self.\_cache\_mtime: Dict\[Tuple\[int, ...\], float\] \= {}

def load\_memory\_context(

self,

project\_id: Optional\[int\] \= None,

\*,

query\_text: str | None \= None,

last\_query\_state: Optional\[Dict\[str, object\]\] \= None,

) \-> str:

"""加载记忆内容（用于注入 system prompt）

按 project\_id 分桶加载：

1\. 始终加载 \_global/ 下的全局记忆

2\. 如果指定了 project\_id，额外加载 project\_{id}/ 下的项目记忆

Args:

project\_id: 项目ID，为 None 时只加载全局记忆

"""

cache\_key \= (project\_id,) if project\_id is not None else (None,)

\# 检查缓存

if cache\_key in self.\_cache and not self.\_is\_cache\_stale(cache\_key):

logger.debug(f"LongTermMemory: cache hit for project={project\_id}")

return self.\_cache\[cache\_key\]

logger.debug(f"LongTermMemory: loading from disk for project={project\_id}")

parts \= \[\]

\# 1. 加载全局记忆

global\_dir \= self.data\_path / "\_global"

global\_parts \= self.\_load\_dir(global\_dir)

if global\_parts:

parts.extend(global\_parts)

\# 2. 加载项目记忆

if project\_id is not None:

project\_dir \= self.data\_path / f"project\_{project\_id}"

project\_parts \= self.\_load\_dir(project\_dir)

if project\_parts:

parts.extend(project\_parts)

if not parts:

logger.debug(f"LongTermMemory: no memory found for project={project\_id}")

return ""

selected\_parts \= self.\_select\_relevant\_parts(parts, query\_text\=query\_text, last\_query\_state\=last\_query\_state)

result \= "\\n\\n".join(selected\_parts)

result \= self.\_truncate(result)

self.\_cache\[cache\_key\] \= result

self.\_cache\_mtime\[cache\_key\] \= self.\_latest\_mtime(project\_id)

logger.debug(f"LongTermMemory: loaded {len(result)} chars for project={project\_id}")

return result

def \_load\_dir(self, directory: Path) \-> list\[str\]:

"""读取某个目录下的 MEMORY.md 索引及其引用的文件"""

memory\_index \= directory / "MEMORY.md"

if not memory\_index.exists():

return \[\]

return self.\_load\_all\_files(memory\_index)

def \_load\_all\_files(self, memory\_index: Path) \-> list\[str\]:

"""读取 MEMORY.md 索引中引用的所有文件内容"""

try:

index\_content \= memory\_index.read\_text(encoding\="utf-8").strip()

except OSError:

return \[\]

base\_dir \= memory\_index.parent

parts \= \[\]

for line in index\_content.split("\\n"):

match \= re.match(r"- \\\[.+?\\\]\\((.+?)\\)", line)

if match:

file\_path \= base\_dir / match.group(1)

if file\_path.exists():

try:

content \= file\_path.read\_text(encoding\="utf-8")

\# 去掉 frontmatter（--- ... ---）

content \= re.sub(r"^---\\n.\*?\\n---\\n", "", content, flags\=re.DOTALL)

stripped \= content.strip()

if stripped:

parts.append(stripped)

except OSError as e:

logger.warning(f"Failed to read memory file {file\_path}: {e}")

return parts

def \_select\_relevant\_parts(

self,

parts: list\[str\],

\*,

query\_text: str | None \= None,

last\_query\_state: Optional\[Dict\[str, object\]\] \= None,

) \-> list\[str\]:

if not query\_text and not last\_query\_state:

return parts

keywords \= self.\_build\_keywords(query\_text\=query\_text, last\_query\_state\=last\_query\_state)

if not keywords:

return parts

scored\_parts \= \[\]

for part in parts:

lowered \= part.lower()

score \= sum(1 for kw in keywords if kw and kw in lowered)

scored\_parts.append((score, len(part), part))

matched \= \[part for score, \_, part in sorted(scored\_parts, key\=lambda item: (item\[0\], item\[1\]), reverse\=True) if score \> 0\]

if matched:

return matched\[:3\]

\# 没有命中时保守回退，避免完全丢失上下文

return parts\[:2\]

@staticmethod

def \_build\_keywords(

\*,

query\_text: str | None \= None,

last\_query\_state: Optional\[Dict\[str, object\]\] \= None,

) \-> list\[str\]:

raw\_tokens: list\[str\] \= \[\]

if query\_text:

normalized\_query \= query\_text.lower()

raw\_tokens.extend(re.findall(r"\[a-z0-9\_\]+", normalized\_query))

cjk\_chunks \= re.findall(r"\[\\u4e00-\\u9fff\]+", normalized\_query)

for chunk in cjk\_chunks:

raw\_tokens.append(chunk)

if len(chunk) <= 2:

continue

for idx in range(len(chunk) \- 1):

raw\_tokens.append(chunk\[idx : idx + 2\])

if last\_query\_state:

for key in ("event", "metric"):

value \= last\_query\_state.get(key)

if isinstance(value, str) and value:

raw\_tokens.append(value.lower())

for key in ("group\_by", "region\_filter"):

values \= last\_query\_state.get(key)

if isinstance(values, list):

raw\_tokens.extend(str(value).lower() for value in values if value)

seen \= set()

keywords \= \[\]

for token in raw\_tokens:

if len(token) < 2:

continue

if token in seen:

continue

seen.add(token)

keywords.append(token)

return keywords

def \_is\_cache\_stale(self, cache\_key: Tuple\[int, ...\]) \-> bool:

"""检查相关目录的文件是否有更新"""

project\_id \= cache\_key\[0\]

current\_mtime \= self.\_latest\_mtime(project\_id)

cached\_mtime \= self.\_cache\_mtime.get(cache\_key, 0.0)

return current\_mtime \> cached\_mtime

def \_latest\_mtime(self, project\_id: Optional\[int\] \= None) \-> float:

"""获取相关目录下所有 .md 文件的最新修改时间"""

latest \= 0.0

dirs \= \[self.data\_path / "\_global"\]

if project\_id is not None:

dirs.append(self.data\_path / f"project\_{project\_id}")

for d in dirs:

try:

for f in d.glob("\*.md"):

latest \= max(latest, f.stat().st\_mtime)

except OSError:

pass

return latest

@staticmethod

def \_truncate(

text: str,

max\_lines: int \= \_MAX\_MEMORY\_LINES,

max\_bytes: int \= \_MAX\_MEMORY\_BYTES,

) \-> str:

"""截断保护，避免注入内容占用过多 token"""

lines \= text.split("\\n")

if len(lines) \> max\_lines:

text \= "\\n".join(lines\[:max\_lines\])

if len(text.encode("utf-8")) \> max\_bytes:

cut \= text.rfind("\\n", 0, max\_bytes)

text \= text\[:cut\] if cut \> 0 else text\[:max\_bytes\]

return text