# Source: https://github.com/frankzh0330/query-agent/blob/master/matcher/base.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# base.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add message bus, async worker and ingress layer for multi-gatew…](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1)

Open commit details

Apr 14, 2026

[ec3eb43](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1) · Apr 14, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/matcher/base.py)

Open commit details

History

412 lines (339 loc) · 12.9 KB

## FilesExpand file tree

master

/

# base.py

Copy path

Top

## File metadata and controls

- Code

- Blame

412 lines (339 loc) · 12.9 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/matcher/base.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import zlib from collections import defaultdict from dataclasses import dataclass from typing import Any, Dict, List, Optional, Set, Tuple import jieba from rapidfuzz import fuzz, process from common.text\_utils import ( contains\_chinese, is\_single\_chinese\_char, normalize, tokenize\_mixed, ) # ========================= # 配置项 # ========================= DEFAULT\_STOPWORDS = {"", " ", "\_", "-", "event", "埋点", "数据"} DEFAULT\_SYNONYM\_MAP = { # 中文 -> 英文 / 同义词扩展 "应用": \["app"\], "启动": \["launch", "打开", "拉起"\], "打开": \["launch", "启动"\], "支付": \["payment"\], "成功": \["success"\], "失败": \["failed"\], "播放": \["play"\], "视频": \["video"\], "注册": \["signup", "register"\], } TIME\_LIKE\_WORDS = {"今天", "昨天", "近7天", "最近7天", "过去7天"} # ========================= # 工具函数 # ========================= def \_stable\_id(name: str) -> int: """ 生成稳定的整数 ID - 相同名称永远产生相同的正整数 ID - ID 范围：0 ~ 2^31-1（避免负数） - 使用 CRC32 算法，速度快且分布均匀 示例： \_stable\_id("app\_launch") = 382517219 \_stable\_id("payment\_success") = 129485012 # 任何顺序、任何次数启动，永远相同 """ return zlib.crc32(name.encode()) & 0x7fffffff # ========================= # 数据结构 # ========================= @dataclass(frozen=True) class MatchResult: """匹配结果""" matched: Optional\[str\] score: float explain: Dict\[str, Any\] # ========================= # BaseMatcher 基类 # ========================= class BaseMatcher: """ 匹配器基类，提供： 1. 文本规范化 2. 中英混合分词 3. 同义词扩展 4. 倒排索引召回 5. RapidFuzz 重排 """ def \_\_init\_\_( self, catalog\_data: Dict\[str, Dict\[str, Any\]\], stopwords: Optional\[Set\[str\]\] = None, synonym\_map: Optional\[Dict\[str, List\[str\]\]\] = None, fuzzy\_threshold: float = 70.0, max\_candidates: int = 200, max\_posting\_per\_token: int = 1000, enable\_jieba\_userdict: bool = True, time\_like\_words: Optional\[Set\[str\]\] = None, ): """ :param catalog\_data: 配置数据，如 {"app\_launch": {"aliases": \["应用启动"\]}} :param stopwords: 停用词 :param synonym\_map: query 侧同义词扩展 :param fuzzy\_threshold: RapidFuzz 阈值 :param max\_candidates: 召回后最多保留多少候选 :param max\_posting\_per\_token: 每个 token 最多保留多少 posting :param enable\_jieba\_userdict: 是否把 alias 加入 jieba 词典 :param time\_like\_words: 时间相关词汇（不参与匹配） """ self.stopwords = stopwords or DEFAULT\_STOPWORDS self.synonym\_map = synonym\_map or DEFAULT\_SYNONYM\_MAP self.fuzzy\_threshold = fuzzy\_threshold self.max\_candidates = max\_candidates self.max\_posting\_per\_token = max\_posting\_per\_token self.time\_like\_words = time\_like\_words or TIME\_LIKE\_WORDS # 主存储 self.id\_to\_doc: Dict\[int, Dict\[str, Any\]\] = {} self.name\_to\_id: Dict\[str, int\] = {} # exact alias: normalized\_alias -> id self.exact\_alias\_map: Dict\[str, int\] = {} # 倒排索引: token -> list\[id\] self.token\_to\_ids: Dict\[str, List\[int\]\] = {} # 每个 doc 对应的 normalized aliases（供 rerank 用） self.id\_to\_norm\_aliases: Dict\[int, List\[str\]\] = {} # 子类实现具体索引构建 self.\_build\_index(catalog\_data, enable\_jieba\_userdict=enable\_jieba\_userdict) def \_build\_index(self, catalog\_data: Dict\[str, Dict\[str, Any\]\], enable\_jieba\_userdict: bool = True) -> None: """构建索引，子类需要实现""" raise NotImplementedError def \_add\_to\_jieba\_dict(self, aliases: List\[str\]) -> None: """将别名加入 jieba 词典""" for alias in aliases: if contains\_chinese(alias): jieba.add\_word(alias) def \_build\_inverted\_index( self, doc\_id: int, aliases: List\[str\], tmp\_index: Dict\[str, Set\[int\]\], ) -> None: """为文档构建倒排索引""" MAX\_ALIAS\_LENGTH = 20 # 别名最大长度 for alias in aliases: # 跳过过长的别名 if len(alias) > MAX\_ALIAS\_LENGTH: continue norm\_alias = normalize(alias) if not norm\_alias: continue # exact alias map self.exact\_alias\_map.setdefault(norm\_alias, doc\_id) # 保存 normalized aliases 用于 rerank self.id\_to\_norm\_aliases\[doc\_id\].append(norm\_alias) # tokenize for inverted index，限制最多 5 个 token tokens = tokenize\_mixed(alias, max\_tokens=5) for token in tokens: token = normalize(token) if not token or token in self.stopwords or token in self.time\_like\_words: continue # 不索引中文单字，避免爆炸 if is\_single\_chinese\_char(token): continue # 确保 token 在 tmp\_index 中存在 if token not in tmp\_index: tmp\_index\[token\] = set() tmp\_index\[token\].add(doc\_id) def \_finalize\_index(self, tmp\_index: Dict\[str, Set\[int\]\]) -> Dict\[str, List\[int\]\]: """完成索引构建，限制 posting 长度""" final\_index: Dict\[str, List\[int\]\] = {} for token, ids in tmp\_index.items(): id\_list = sorted(ids) if len(id\_list) > self.max\_posting\_per\_token: id\_list = id\_list\[:self.max\_posting\_per\_token\] final\_index\[token\] = id\_list return final\_index # ========================= # 匹配主入口 # ========================= def match(self, query: str) -> MatchResult: """匹配入口""" norm\_query = normalize(query) if not norm\_query: return MatchResult( matched=None, score=0.0, explain={"method": "empty\_query", "query": query}, ) # 1. exact alias 命中 exact\_id = self.exact\_alias\_map.get(norm\_query) if exact\_id is not None: return MatchResult( matched=self.\_get\_name\_by\_id(exact\_id), score=100.0, explain={ "method": "exact\_alias\_match", "query": query, "normalized\_query": norm\_query, "matched\_id": exact\_id, "matched\_name": self.\_get\_name\_by\_id(exact\_id), }, ) # 2. query tokenize + synonym expand query\_tokens = tokenize\_mixed(query) expanded\_tokens = self.\_expand\_query\_tokens(query\_tokens) # 3. 倒排索引召回 candidates, recall\_explain = self.\_recall\_candidates(expanded\_tokens) if not candidates: return MatchResult( matched=None, score=0.0, explain={ "method": "no\_candidates", "query": query, "normalized\_query": norm\_query, "query\_tokens": query\_tokens, "expanded\_tokens": expanded\_tokens, "recall\_explain": recall\_explain, }, ) # 4. RapidFuzz 重排 rerank\_result = self.\_rerank(query, candidates) if rerank\_result is None: return MatchResult( matched=None, score=0.0, explain={ "method": "rerank\_failed", "query": query, "normalized\_query": norm\_query, "query\_tokens": query\_tokens, "expanded\_tokens": expanded\_tokens, "recall\_explain": recall\_explain, }, ) best\_id, best\_score, rerank\_explain = rerank\_result best\_name = self.\_get\_name\_by\_id(best\_id) if best\_score < self.fuzzy\_threshold: return MatchResult( matched=None, score=best\_score, explain={ "method": "score\_below\_threshold", "query": query, "normalized\_query": norm\_query, "query\_tokens": query\_tokens, "expanded\_tokens": expanded\_tokens, "threshold": self.fuzzy\_threshold, "recall\_explain": recall\_explain, "rerank\_explain": rerank\_explain, }, ) return MatchResult( matched=best\_name, score=best\_score, explain={ "method": "inverted\_index\_plus\_rerank", "query": query, "normalized\_query": norm\_query, "query\_tokens": query\_tokens, "expanded\_tokens": expanded\_tokens, "matched\_id": best\_id, "matched\_name": best\_name, "score": best\_score, "recall\_explain": recall\_explain, "rerank\_explain": rerank\_explain, }, ) def \_get\_name\_by\_id(self, doc\_id: int) -> str: """根据 ID 获取名称，子类实现""" raise NotImplementedError # ========================= # 召回与重排 # ========================= def \_recall\_candidates(self, expanded\_tokens: List\[str\]) -> Tuple\[List\[int\], Dict\[str, Any\]\]: """从倒排索引召回候选""" hit\_counts: Dict\[int, int\] = defaultdict(int) token\_hits = \[\] for token in expanded\_tokens: token = normalize(token) if not token or token in self.stopwords or token in self.time\_like\_words: continue ids = self.token\_to\_ids.get(token, \[\]) token\_hits.append((token, len(ids))) for doc\_id in ids: hit\_counts\[doc\_id\] += 1 if not hit\_counts: return \[\], { "token\_hits": token\_hits, "candidate\_count": 0, "top\_candidates": \[\], } ranked = sorted(hit\_counts.items(), key=lambda x: (-x\[1\], x\[0\])) candidate\_ids = \[doc\_id for doc\_id, \_ in ranked\[:self.max\_candidates\]\] explain = { "token\_hits": token\_hits, "candidate\_count": len(candidate\_ids), "top\_candidates": \[ { "id": doc\_id, "name": self.\_get\_name\_by\_id(doc\_id), "hit\_count": cnt, } for doc\_id, cnt in ranked\[:10\] \], } return candidate\_ids, explain def \_rerank(self, query: str, candidate\_ids: List\[int\]) -> Optional\[Tuple\[int, float, Dict\[str, Any\]\]\]: """RapidFuzz 重排""" norm\_query = normalize(query) best\_id = None best\_score = 0.0 top\_details = \[\] for doc\_id in candidate\_ids: norm\_aliases = self.id\_to\_norm\_aliases.get(doc\_id, \[\]) if not norm\_aliases: continue # 在该 doc 的 aliases 中找最匹配的一个 best\_alias\_match = process.extractOne( norm\_query, norm\_aliases, scorer=fuzz.WRatio, ) if best\_alias\_match is None: continue matched\_alias, score, \_ = best\_alias\_match score = float(score) top\_details.append({ "id": doc\_id, "name": self.\_get\_name\_by\_id(doc\_id), "matched\_alias": matched\_alias, "score": score, }) if score > best\_score: best\_score = score best\_id = doc\_id if best\_id is None: return None top\_details.sort(key=lambda x: (-x\["score"\], x\["name"\])) explain = { "best\_id": best\_id, "best\_name": self.\_get\_name\_by\_id(best\_id), "best\_score": best\_score, "top5": top\_details\[:5\], } return best\_id, best\_score, explain # ========================= # 同义词扩展 # ========================= def \_expand\_query\_tokens(self, tokens: List\[str\]) -> List\[str\]: """只在 query 侧做同义词扩展，避免索引爆炸""" expanded: List\[str\] = \[\] for token in tokens: if not token or token in self.stopwords: continue expanded.append(token) # 同义词扩展 for syn in self.synonym\_map.get(token, \[\]): syn = normalize(syn) if syn and syn not in expanded: expanded.append(syn) return expanded

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

339

340

341

342

343

344

345

346

347

348

349

350

351

352

353

354

355

356

357

358

359

360

361

362

363

364

365

366

367

368

369

370

371

372

373

374

375

376

377

378

379

380

381

382

383

384

385

386

387

388

389

390

391

392

393

394

395

396

397

398

399

400

401

402

403

404

405

406

407

408

409

410

411

412

from \_\_future\_\_ import annotations

import zlib

from collections import defaultdict

from dataclasses import dataclass

from typing import Any, Dict, List, Optional, Set, Tuple

import jieba

from rapidfuzz import fuzz, process

from common.text\_utils import (

contains\_chinese,

is\_single\_chinese\_char,

normalize,

tokenize\_mixed,

)

\# =========================

\# 配置项

\# =========================

DEFAULT\_STOPWORDS \= {"", " ", "\_", "-", "event", "埋点", "数据"}

DEFAULT\_SYNONYM\_MAP \= {

\# 中文 -> 英文 / 同义词扩展

"应用": \["app"\],

"启动": \["launch", "打开", "拉起"\],

"打开": \["launch", "启动"\],

"支付": \["payment"\],

"成功": \["success"\],

"失败": \["failed"\],

"播放": \["play"\],

"视频": \["video"\],

"注册": \["signup", "register"\],

}

TIME\_LIKE\_WORDS \= {"今天", "昨天", "近7天", "最近7天", "过去7天"}

\# =========================

\# 工具函数

\# =========================

def \_stable\_id(name: str) \-> int:

"""

生成稳定的整数 ID

\- 相同名称永远产生相同的正整数 ID

\- ID 范围：0 ~ 2^31-1（避免负数）

\- 使用 CRC32 算法，速度快且分布均匀

示例：

\_stable\_id("app\_launch") = 382517219

\_stable\_id("payment\_success") = 129485012

\# 任何顺序、任何次数启动，永远相同

"""

return zlib.crc32(name.encode()) & 0x7fffffff

\# =========================

\# 数据结构

\# =========================

@dataclass(frozen\=True)

class MatchResult:

"""匹配结果"""

matched: Optional\[str\]

score: float

explain: Dict\[str, Any\]

\# =========================

\# BaseMatcher 基类

\# =========================

class BaseMatcher:

"""

匹配器基类，提供：

1\. 文本规范化

2\. 中英混合分词

3\. 同义词扩展

4\. 倒排索引召回

5\. RapidFuzz 重排

"""

def \_\_init\_\_(

self,

catalog\_data: Dict\[str, Dict\[str, Any\]\],

stopwords: Optional\[Set\[str\]\] \= None,

synonym\_map: Optional\[Dict\[str, List\[str\]\]\] \= None,

fuzzy\_threshold: float \= 70.0,

max\_candidates: int \= 200,

max\_posting\_per\_token: int \= 1000,

enable\_jieba\_userdict: bool \= True,

time\_like\_words: Optional\[Set\[str\]\] \= None,

):

"""

:param catalog\_data: 配置数据，如 {"app\_launch": {"aliases": \["应用启动"\]}}

:param stopwords: 停用词

:param synonym\_map: query 侧同义词扩展

:param fuzzy\_threshold: RapidFuzz 阈值

:param max\_candidates: 召回后最多保留多少候选

:param max\_posting\_per\_token: 每个 token 最多保留多少 posting

:param enable\_jieba\_userdict: 是否把 alias 加入 jieba 词典

:param time\_like\_words: 时间相关词汇（不参与匹配）

"""

self.stopwords \= stopwords or DEFAULT\_STOPWORDS

self.synonym\_map \= synonym\_map or DEFAULT\_SYNONYM\_MAP

self.fuzzy\_threshold \= fuzzy\_threshold

self.max\_candidates \= max\_candidates

self.max\_posting\_per\_token \= max\_posting\_per\_token

self.time\_like\_words \= time\_like\_words or TIME\_LIKE\_WORDS

\# 主存储

self.id\_to\_doc: Dict\[int, Dict\[str, Any\]\] \= {}

self.name\_to\_id: Dict\[str, int\] \= {}

\# exact alias: normalized\_alias -> id

self.exact\_alias\_map: Dict\[str, int\] \= {}

\# 倒排索引: token -> list\[id\]

self.token\_to\_ids: Dict\[str, List\[int\]\] \= {}

\# 每个 doc 对应的 normalized aliases（供 rerank 用）

self.id\_to\_norm\_aliases: Dict\[int, List\[str\]\] \= {}

\# 子类实现具体索引构建

self.\_build\_index(catalog\_data, enable\_jieba\_userdict\=enable\_jieba\_userdict)

def \_build\_index(self, catalog\_data: Dict\[str, Dict\[str, Any\]\], enable\_jieba\_userdict: bool \= True) \-> None:

"""构建索引，子类需要实现"""

raise NotImplementedError

def \_add\_to\_jieba\_dict(self, aliases: List\[str\]) \-> None:

"""将别名加入 jieba 词典"""

for alias in aliases:

if contains\_chinese(alias):

jieba.add\_word(alias)

def \_build\_inverted\_index(

self,

doc\_id: int,

aliases: List\[str\],

tmp\_index: Dict\[str, Set\[int\]\],

) \-> None:

"""为文档构建倒排索引"""

MAX\_ALIAS\_LENGTH \= 20 \# 别名最大长度

for alias in aliases:

\# 跳过过长的别名

if len(alias) \> MAX\_ALIAS\_LENGTH:

continue

norm\_alias \= normalize(alias)

if not norm\_alias:

}

return candidate\_ids, explain

def \_rerank(self, query: str, candidate\_ids: List\[int\]) \-> Optional\[Tuple\[int, float, Dict\[str, Any\]\]\]:

"""RapidFuzz 重排"""

norm\_query \= normalize(query)

best\_id \= None

best\_score \= 0.0

top\_details \= \[\]

for doc\_id in candidate\_ids:

norm\_aliases \= self.id\_to\_norm\_aliases.get(doc\_id, \[\])

if not norm\_aliases:

continue

\# 在该 doc 的 aliases 中找最匹配的一个

best\_alias\_match \= process.extractOne(

norm\_query,

norm\_aliases,

scorer\=fuzz.WRatio,

)

if best\_alias\_match is None:

continue

matched\_alias, score, \_ \= best\_alias\_match

score \= float(score)

top\_details.append({

"id": doc\_id,

"name": self.\_get\_name\_by\_id(doc\_id),

"matched\_alias": matched\_alias,

"score": score,

})

if score \> best\_score:

best\_score \= score

best\_id \= doc\_id

if best\_id is None:

return None

top\_details.sort(key\=lambda x: (\-x\["score"\], x\["name"\]))

explain \= {

"best\_id": best\_id,

"best\_name": self.\_get\_name\_by\_id(best\_id),

"best\_score": best\_score,

"top5": top\_details\[:5\],

}

return best\_id, best\_score, explain

\# =========================

\# 同义词扩展

\# =========================

def \_expand\_query\_tokens(self, tokens: List\[str\]) \-> List\[str\]:

"""只在 query 侧做同义词扩展，避免索引爆炸"""

expanded: List\[str\] \= \[\]

for token in tokens:

if not token or token in self.stopwords:

continue

expanded.append(token)

\# 同义词扩展

for syn in self.synonym\_map.get(token, \[\]):

syn \= normalize(syn)

if syn and syn not in expanded:

expanded.append(syn)

return expanded