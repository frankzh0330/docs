# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_end_to_end_evals.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_end\_to\_end\_evals.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_end_to_end_evals.py)

Open commit details

History

305 lines (244 loc) · 11.1 KB

## FilesExpand file tree

master

/

# test\_end\_to\_end\_evals.py

Copy path

Top

## File metadata and controls

- Code

- Blame

305 lines (244 loc) · 11.1 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_end_to_end_evals.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Data-driven end-to-end evals for the NL2DSL pipeline.""" from \_\_future\_\_ import annotations from pathlib import Path from unittest import mock import pytest import yaml from fastapi.testclient import TestClient from matcher.matcher\_service import ResolvedResult from service.llm\_extractions import Extraction, ExtractionsJson from service.session\_models import QueryState def \_load\_cases() -> list\[dict\]: path = Path(\_\_file\_\_).parent / "evals" / "nl2dsl\_cases.yaml" data = yaml.safe\_load(path.read\_text(encoding="utf-8")) or {} return data.get("cases", \[\]) def \_make\_extraction(payload: dict | None) -> ExtractionsJson: payload = payload or {} def \_items(key: str) -> list\[Extraction\]: return \[Extraction(text=text) for text in payload.get(key, \[\])\] return ExtractionsJson( metric\_extractions=\_items("metric\_extractions"), time\_extractions=\_items("time\_extractions"), event\_extractions=\_items("event\_extractions"), group\_by\_extractions=\_items("group\_by\_extractions"), chart\_type\_extractions=\_items("chart\_type\_extractions"), interaction\_mode\_extractions=\_items("interaction\_mode\_extractions"), region\_filter=payload.get("region\_filter", \[\]), ) def \_build\_resolver(name: str): svc = mock.MagicMock() if name == "high\_confidence": svc.resolve\_with\_candidates.side\_effect = lambda mtype, extractions, default: ResolvedResult( value="app\_launch" if mtype.value == "event" else ("pv" if mtype.value == "metric" else "country"), score=95.0, method="exact", candidates=\[\], needs\_confirmation=False, ) svc.resolve\_time.return\_value = (7, {"method": "default", "n": 7}) return svc if name == "time\_yesterday": svc.resolve\_time.return\_value = (1, {"method": "matched", "n": 1}) return svc if name == "metric\_uv": def \_resolve\_metric(mtype, extractions, default): if mtype.value == "metric": return ResolvedResult( value="uv", score=96.0, method="exact", candidates=\[\], needs\_confirmation=False, ) raise AssertionError(f"unexpected matcher type: {mtype}") svc.resolve\_with\_candidates.side\_effect = \_resolve\_metric return svc if name == "low\_confidence\_event": def \_resolve\_event(mtype, extractions, default): if mtype.value == "event": return ResolvedResult( value="purchase\_success", score=55.0, method="fuzzy\_low\_confidence", candidates=\[ {"value": "purchase\_success", "score": 55.0}, {"value": "payment\_submit", "score": 48.0}, \], needs\_confirmation=True, ) return ResolvedResult( value="pv" if mtype.value == "metric" else "country", score=95.0, method="exact", candidates=\[\], needs\_confirmation=False, ) svc.resolve\_with\_candidates.side\_effect = \_resolve\_event svc.resolve\_time.return\_value = (7, {"method": "default", "n": 7}) return svc if name == "project\_memory\_alias": def \_resolve\_alias(mtype, extractions, default): if mtype.value == "event": return ResolvedResult( value="activation\_success", score=97.0, method="memory\_alias", candidates=\[\], needs\_confirmation=False, ) return ResolvedResult( value="pv" if mtype.value == "metric" else "country", score=95.0, method="exact", candidates=\[\], needs\_confirmation=False, ) svc.resolve\_with\_candidates.side\_effect = \_resolve\_alias svc.resolve\_time.return\_value = (7, {"method": "default", "n": 7}) return svc raise ValueError(f"unknown resolver scenario: {name}") @pytest.fixture def eval\_client(tmp\_path): import app as app\_module from memory.storage.memory\_file import TaskStorage from memory.user\_preference\_store import UserPreferenceStore from service.session\_manager import SessionManager from service.task\_manager import TaskManager app\_module.session\_manager = SessionManager(data\_path=str(tmp\_path / "data")) app\_module.task\_manager = TaskManager( storage=TaskStorage(data\_path=str(tmp\_path / "data" / "tasks")) ) app\_module.user\_preference\_store = UserPreferenceStore( data\_path=str(tmp\_path / "data" / "user\_preferences") ) # 同步更新 orchestrator 的引用 app\_module.orchestrator.session = app\_module.session\_manager app\_module.orchestrator.task = app\_module.task\_manager app\_module.orchestrator.preferences = app\_module.user\_preference\_store mock\_service = mock.MagicMock() mock\_service.catalog = mock.MagicMock() app\_module.\_matcher\_service = mock\_service with TestClient(app\_module.app) as c: yield c app\_module.\_matcher\_service = None def \_build\_runtime(data\_root: Path) -> None: import app as app\_module from memory.storage.memory\_file import TaskStorage from memory.user\_preference\_store import UserPreferenceStore from service.session\_manager import SessionManager from service.task\_manager import TaskManager app\_module.session\_manager = SessionManager(data\_path=str(data\_root)) app\_module.task\_manager = TaskManager( storage=TaskStorage(data\_path=str(data\_root / "tasks")) ) app\_module.user\_preference\_store = UserPreferenceStore( data\_path=str(data\_root / "user\_preferences") ) # 同步更新 orchestrator 的引用 app\_module.orchestrator.session = app\_module.session\_manager app\_module.orchestrator.task = app\_module.task\_manager app\_module.orchestrator.preferences = app\_module.user\_preference\_store mock\_service = mock.MagicMock() mock\_service.catalog = mock.MagicMock() app\_module.\_matcher\_service = mock\_service def \_write\_project\_memory(case: dict, data\_root: Path) -> None: setup = case.get("setup") or {} project\_memory = setup.get("project\_memory") if not project\_memory: return project\_id = project\_memory\["project\_id"\] project\_dir = data\_root / "memory" / f"project\_{project\_id}" project\_dir.mkdir(parents=True, exist\_ok=True) links = \[\] for entry in project\_memory.get("entries", \[\]): filename = entry\["filename"\] (project\_dir / filename).write\_text(entry\["content"\], encoding="utf-8") links.append(f"- \[{filename}\]({filename})") (project\_dir / "MEMORY.md").write\_text("\\n".join(links) + "\\n", encoding="utf-8") def \_seed\_setup(case: dict) -> str | None: import app as app\_module setup = case.get("setup") or {} data\_root = Path(app\_module.session\_manager.storage.data\_path).parent \_write\_project\_memory(case, data\_root) last\_query\_state = setup.get("last\_query\_state") if not last\_query\_state: return None ctx = app\_module.session\_manager.create\_or\_get(None, "eval\_user", last\_query\_state\["project\_id"\]) app\_module.session\_manager.update\_query\_state(ctx.session\_id, QueryState(\*\*last\_query\_state)) return ctx.session\_id def \_assert\_expectations(body: dict, expect: dict) -> None: assert body\["status"\] == expect\["status"\] if "turn\_mode" in expect: assert body\["explain"\]\["turn\_explain"\]\["mode"\] == expect\["turn\_mode"\] if "candidates\_contains" in expect: assert expect\["candidates\_contains"\] in (body.get("candidates") or {}) semantic\_expect = expect.get("semantic") if not semantic\_expect: return semantic = body\["semantic"\] if "event" in semantic\_expect: assert semantic\["event"\]\["event\_name"\] == semantic\_expect\["event"\] if "metric" in semantic\_expect: assert semantic\["metric"\]\["metric\_id"\] == semantic\_expect\["metric"\] if "region\_filter" in semantic\_expect: assert semantic\["region\_filter"\] == semantic\_expect\["region\_filter"\] if "group\_by" in semantic\_expect: assert \[item\["dimension\_id"\] for item in semantic\["group\_by"\]\] == semantic\_expect\["group\_by"\] if "time\_n" in semantic\_expect: assert semantic\["time\_range"\]\["n"\] == semantic\_expect\["time\_n"\] @pytest.mark.parametrize("case", \_load\_cases(), ids=lambda case: case\["name"\]) def test\_nl2dsl\_end\_to\_end\_eval\_cases(eval\_client, case): import app as app\_module data\_root = Path(app\_module.session\_manager.storage.data\_path).parent session\_id = \_seed\_setup(case) for step in case\["steps"\]: if step.get("restart\_before"): \_build\_runtime(data\_root) request\_body = { "text": step\["text"\], "project\_id": step.get("project\_id", 55), } if session\_id: request\_body\["session\_id"\] = session\_id patches = \[ mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}), mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"), \] extraction\_payload = step.get("extraction") if extraction\_payload is not None: expected\_memory = step.get("assert\_memory\_contains") async def \_mock\_extract(query, session\_context=None, \*, \_payload=extraction\_payload, \_expected\_memory=expected\_memory): if \_expected\_memory: memory\_context = (session\_context or {}).get("memory\_corrections", "") assert \_expected\_memory in memory\_context return \_make\_extraction(\_payload) patches.append( mock.patch( "service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, side\_effect=\_mock\_extract, ) ) resolver\_name = step.get("resolver") if resolver\_name: patches.append( mock.patch("app.get\_matcher\_service", return\_value=\_build\_resolver(resolver\_name)) ) with patches\[0\], patches\[1\], patches\[2\]: extra\_contexts = patches\[3:\] if extra\_contexts: with extra\_contexts\[0\]: if len(extra\_contexts) > 1: with extra\_contexts\[1\]: resp = eval\_client.post("/nl2dsl", json=request\_body) else: resp = eval\_client.post("/nl2dsl", json=request\_body) else: resp = eval\_client.post("/nl2dsl", json=request\_body) body = resp.json() \_assert\_expectations(body, step\["expect"\]) session\_id = body.get("session\_id", session\_id) # eval 期间确保 session 确实持续复用，避免 accidentally stateless if session\_id: assert app\_module.session\_manager.get\_session(session\_id) is not None

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

259

260

261

262

263

264

265

266

267

268

269

270

271

272

273

274

275

276

277

278

279

280

281

282

283

284

285

286

287

288

289

290

291

292

293

294

295

296

297

298

299

300

301

302

303

304

305

"""Data-driven end-to-end evals for the NL2DSL pipeline."""

from \_\_future\_\_ import annotations

from pathlib import Path

from unittest import mock

import pytest

import yaml

from fastapi.testclient import TestClient

from matcher.matcher\_service import ResolvedResult

from service.llm\_extractions import Extraction, ExtractionsJson

from service.session\_models import QueryState

def \_load\_cases() \-> list\[dict\]:

path \= Path(\_\_file\_\_).parent / "evals" / "nl2dsl\_cases.yaml"

data \= yaml.safe\_load(path.read\_text(encoding\="utf-8")) or {}

return data.get("cases", \[\])

def \_make\_extraction(payload: dict | None) \-> ExtractionsJson:

payload \= payload or {}

def \_items(key: str) \-> list\[Extraction\]:

return \[Extraction(text\=text) for text in payload.get(key, \[\])\]

return ExtractionsJson(

metric\_extractions\=\_items("metric\_extractions"),

time\_extractions\=\_items("time\_extractions"),

event\_extractions\=\_items("event\_extractions"),

group\_by\_extractions\=\_items("group\_by\_extractions"),

chart\_type\_extractions\=\_items("chart\_type\_extractions"),

interaction\_mode\_extractions\=\_items("interaction\_mode\_extractions"),

region\_filter\=payload.get("region\_filter", \[\]),

)

def \_build\_resolver(name: str):

svc \= mock.MagicMock()

if name \== "high\_confidence":

svc.resolve\_with\_candidates.side\_effect \= lambda mtype, extractions, default: ResolvedResult(

value\="app\_launch" if mtype.value \== "event" else ("pv" if mtype.value \== "metric" else "country"),

score\=95.0,

method\="exact",

candidates\=\[\],

needs\_confirmation\=False,

)

svc.resolve\_time.return\_value \= (7, {"method": "default", "n": 7})

return svc

if name \== "time\_yesterday":

svc.resolve\_time.return\_value \= (1, {"method": "matched", "n": 1})

return svc

if name \== "metric\_uv":

def \_resolve\_metric(mtype, extractions, default):

if mtype.value \== "metric":

return ResolvedResult(

value\="uv",

score\=96.0,

method\="exact",

candidates\=\[\],

needs\_confirmation\=False,

)

raise AssertionError(f"unexpected matcher type: {mtype}")

svc.resolve\_with\_candidates.side\_effect \= \_resolve\_metric

return svc

if name \== "low\_confidence\_event":

def \_resolve\_event(mtype, extractions, default):

if mtype.value \== "event":

return ResolvedResult(

value\="purchase\_success",

score\=55.0,

method\="fuzzy\_low\_confidence",

candidates\=\[

{"value": "purchase\_success", "score": 55.0},

{"value": "payment\_submit", "score": 48.0},

\],

needs\_confirmation\=True,

)

return ResolvedResult(

value\="pv" if mtype.value \== "metric" else "country",

score\=95.0,

method\="exact",

candidates\=\[\],

needs\_confirmation\=False,

)

svc.resolve\_with\_candidates.side\_effect \= \_resolve\_event

svc.resolve\_time.return\_value \= (7, {"method": "default", "n": 7})

return svc

if name \== "project\_memory\_alias":

def \_resolve\_alias(mtype, extractions, default):

if mtype.value \== "event":

return ResolvedResult(

value\="activation\_success",

score\=97.0,

method\="memory\_alias",

candidates\=\[\],

needs\_confirmation\=False,

)

return ResolvedResult(

value\="pv" if mtype.value \== "metric" else "country",

score\=95.0,

method\="exact",

candidates\=\[\],

needs\_confirmation\=False,

)

svc.resolve\_with\_candidates.side\_effect \= \_resolve\_alias

svc.resolve\_time.return\_value \= (7, {"method": "default", "n": 7})

return svc

raise ValueError(f"unknown resolver scenario: {name}")

@pytest.fixture

def eval\_client(tmp\_path):

import app as app\_module

from memory.storage.memory\_file import TaskStorage

from memory.user\_preference\_store import UserPreferenceStore

from service.session\_manager import SessionManager

from service.task\_manager import TaskManager

app\_module.session\_manager \= SessionManager(data\_path\=str(tmp\_path / "data"))

app\_module.task\_manager \= TaskManager(

storage\=TaskStorage(data\_path\=str(tmp\_path / "data" / "tasks"))

)

app\_module.user\_preference\_store \= UserPreferenceStore(

data\_path\=str(tmp\_path / "data" / "user\_preferences")

)

\# 同步更新 orchestrator 的引用

app\_module.orchestrator.session \= app\_module.session\_manager

app\_module.orchestrator.task \= app\_module.task\_manager

app\_module.orchestrator.preferences \= app\_module.user\_preference\_store

mock\_service \= mock.MagicMock()

mock\_service.catalog \= mock.MagicMock()

app\_module.\_matcher\_service \= mock\_service

with TestClient(app\_module.app) as c:

yield c

app\_module.\_matcher\_service \= None

def \_build\_runtime(data\_root: Path) \-> None:

import app as app\_module

assert semantic\["region\_filter"\] \== semantic\_expect\["region\_filter"\]

if "group\_by" in semantic\_expect:

assert \[item\["dimension\_id"\] for item in semantic\["group\_by"\]\] \== semantic\_expect\["group\_by"\]

if "time\_n" in semantic\_expect:

assert semantic\["time\_range"\]\["n"\] \== semantic\_expect\["time\_n"\]

@pytest.mark.parametrize("case", \_load\_cases(), ids\=lambda case: case\["name"\])

def test\_nl2dsl\_end\_to\_end\_eval\_cases(eval\_client, case):

import app as app\_module

data\_root \= Path(app\_module.session\_manager.storage.data\_path).parent

session\_id \= \_seed\_setup(case)

for step in case\["steps"\]:

if step.get("restart\_before"):

\_build\_runtime(data\_root)

request\_body \= {

"text": step\["text"\],

"project\_id": step.get("project\_id", 55),

}

if session\_id:

request\_body\["session\_id"\] \= session\_id

patches \= \[

mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value\={"content": {"queries": \[\]}}),

mock.patch("service.query\_orchestrator.validate\_region"),

mock.patch("service.query\_orchestrator.validate\_region\_consistency"),

\]

extraction\_payload \= step.get("extraction")

if extraction\_payload is not None:

expected\_memory \= step.get("assert\_memory\_contains")

async def \_mock\_extract(query, session\_context\=None, \*, \_payload\=extraction\_payload, \_expected\_memory\=expected\_memory):

if \_expected\_memory:

memory\_context \= (session\_context or {}).get("memory\_corrections", "")

assert \_expected\_memory in memory\_context

return \_make\_extraction(\_payload)

patches.append(

mock.patch(

"service.query\_orchestrator.extract\_llm\_async",

new\_callable\=mock.AsyncMock,

side\_effect\=\_mock\_extract,

)

)

resolver\_name \= step.get("resolver")

if resolver\_name:

patches.append(

mock.patch("app.get\_matcher\_service", return\_value\=\_build\_resolver(resolver\_name))

)

with patches\[0\], patches\[1\], patches\[2\]:

extra\_contexts \= patches\[3:\]

if extra\_contexts:

with extra\_contexts\[0\]:

if len(extra\_contexts) \> 1:

with extra\_contexts\[1\]:

resp \= eval\_client.post("/nl2dsl", json\=request\_body)

else:

resp \= eval\_client.post("/nl2dsl", json\=request\_body)

else:

resp \= eval\_client.post("/nl2dsl", json\=request\_body)

body \= resp.json()

\_assert\_expectations(body, step\["expect"\])

session\_id \= body.get("session\_id", session\_id)

\# eval 期间确保 session 确实持续复用，避免 accidentally stateless

if session\_id:

assert app\_module.session\_manager.get\_session(session\_id) is not None