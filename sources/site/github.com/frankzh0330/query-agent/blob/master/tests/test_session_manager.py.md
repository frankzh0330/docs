# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_session_manager.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_session\_manager.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_session_manager.py)

Open commit details

History

224 lines (191 loc) · 10 KB

## FilesExpand file tree

master

/

# test\_session\_manager.py

Copy path

Top

## File metadata and controls

- Code

- Blame

224 lines (191 loc) · 10 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_session_manager.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""SessionManager 集成测试""" import json from datetime import datetime, timedelta class TestSessionManager: """service/session\_manager.py""" def test\_create\_or\_get\_new\_session(self, session\_manager): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) assert ctx.session\_id assert ctx.user\_id == "user\_1" assert ctx.project\_id == 55 def test\_create\_or\_get\_returns\_existing(self, session\_manager): ctx1 = session\_manager.create\_or\_get(None, "user\_1", 55) ctx2 = session\_manager.create\_or\_get(ctx1.session\_id, "user\_1", 55) assert ctx1 is ctx2 def test\_create\_or\_get\_restores\_from\_jsonl(self, session\_manager): sid = "restore\_test\_123" # 通过 storage 写入 JSONL 记录 session\_manager.storage.append(sid, { "type": "message", "role": "user", "content": "hello from file", "timestamp": datetime.now().isoformat(), "metadata": {}, }) session\_manager.storage.append(sid, { "type": "query\_state", "data": { "project\_id": 55, "event": "app\_launch", "metric": "pv", "time\_range": {"type": "last\_n\_days", "n": 7}, "region\_filter": \["ROW"\], "group\_by": \["country"\], "filters": \[\], "turn\_type": "followup\_patch", }, "turn\_type": "followup\_patch", "timestamp": datetime.now().isoformat(), }) session\_manager.storage.append(sid, { "type": "session\_meta", "pending\_task\_id": "task\_123", "last\_turn\_type": "followup\_patch", "last\_user\_query": "hello from file", "turn\_index": 1, "timestamp": datetime.now().isoformat(), }) ctx = session\_manager.create\_or\_get(sid, "user\_1", 55) assert ctx is not None assert len(ctx.messages) == 1 assert ctx.messages\[0\].content == "hello from file" assert ctx.last\_query\_state is not None assert ctx.last\_query\_state.event == "app\_launch" assert ctx.last\_turn\_type == "followup\_patch" assert ctx.pending\_task\_id == "task\_123" assert ctx.last\_user\_query == "hello from file" assert ctx.turn\_index == 1 def test\_create\_or\_get\_new\_when\_jsonl\_missing(self, session\_manager): ctx = session\_manager.create\_or\_get("nonexistent\_id", "user\_1", 55) assert ctx is not None assert ctx.session\_id == "nonexistent\_id" assert len(ctx.messages) == 0 def test\_add\_message\_stores\_in\_memory(self, session\_manager): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.add\_message(ctx.session\_id, "user", "hello") assert len(ctx.messages) == 1 assert ctx.messages\[0\].content == "hello" assert ctx.messages\[0\].role == "user" assert ctx.last\_user\_query == "hello" assert ctx.turn\_index == 1 def test\_add\_message\_persists\_to\_jsonl(self, session\_manager): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.add\_message(ctx.session\_id, "user", "hello", metadata={"test": True}) records = session\_manager.storage.read\_tail(ctx.session\_id) assert len(records) == 1 assert records\[0\]\["type"\] == "message" assert records\[0\]\["content"\] == "hello" def test\_add\_message\_truncates\_memory(self, tmp\_path): from service.session\_manager import SessionManager sm = SessionManager(data\_path=str(tmp\_path / "data"), max\_messages=3) ctx = sm.create\_or\_get(None, "user\_1", 55) for i in range(5): sm.add\_message(ctx.session\_id, "user", f"msg\_{i}") # 内存截断到 3 条 assert len(ctx.messages) == 3 assert ctx.messages\[0\].content == "msg\_2" # JSONL 保留全部 5 条 records = sm.storage.read\_tail(ctx.session\_id) assert len(records) == 5 def test\_add\_message\_unknown\_session\_is\_noop(self, session\_manager): session\_manager.add\_message("nonexistent", "user", "hello") # 不崩溃即可 def test\_update\_query\_state\_stores\_in\_memory(self, session\_manager, sample\_query\_state): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) sample\_query\_state.turn\_type = "followup\_patch" session\_manager.update\_query\_state(ctx.session\_id, sample\_query\_state) assert ctx.last\_query\_state is not None assert ctx.last\_query\_state.event == "app\_launch" assert ctx.last\_turn\_type == "followup\_patch" def test\_update\_query\_state\_persists\_to\_jsonl(self, session\_manager, sample\_query\_state): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) sample\_query\_state.turn\_type = "followup\_patch" session\_manager.update\_query\_state(ctx.session\_id, sample\_query\_state) records = session\_manager.storage.read\_tail(ctx.session\_id) state\_records = \[r for r in records if r.get("type") == "query\_state"\] assert len(state\_records) == 1 assert state\_records\[0\]\["data"\]\["event"\] == "app\_launch" assert state\_records\[0\]\["turn\_type"\] == "followup\_patch" def test\_update\_pending\_task\_stores\_in\_memory(self, session\_manager): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.update\_pending\_task(ctx.session\_id, "task\_1") assert ctx.pending\_task\_id == "task\_1" def test\_update\_pending\_task\_persists\_to\_jsonl(self, session\_manager): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.update\_pending\_task(ctx.session\_id, "task\_1") records = session\_manager.storage.read\_tail(ctx.session\_id) meta\_records = \[r for r in records if r.get("type") == "session\_meta"\] assert len(meta\_records) == 1 assert meta\_records\[0\]\["pending\_task\_id"\] == "task\_1" def test\_get\_enhanced\_context\_includes\_session(self, session\_manager, sample\_query\_state): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.add\_message(ctx.session\_id, "user", "查询PV") sample\_query\_state.turn\_type = "new\_query" session\_manager.update\_query\_state(ctx.session\_id, sample\_query\_state) context = session\_manager.get\_enhanced\_context(ctx.session\_id) assert "recent\_queries" in context assert "查询PV" in context\["recent\_queries"\] assert "last\_query\_state" in context assert context\["last\_query\_state"\]\["event"\] == "app\_launch" assert context\["last\_turn\_type"\] == "new\_query" assert context\["last\_user\_query"\] == "查询PV" assert context\["turn\_index"\] == 1 def test\_get\_enhanced\_context\_includes\_memory(self, session\_manager, tmp\_path): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.add\_message(ctx.session\_id, "user", "hello") # 创建 memory 文件 global\_dir = tmp\_path / "data" / "memory" / "\_global" global\_dir.mkdir(parents=True, exist\_ok=True) (global\_dir / "corrections.md").write\_text("Use ROW for default", encoding="utf-8") (global\_dir / "MEMORY.md").write\_text("- \[Corrections\](corrections.md)\\n", encoding="utf-8") context = session\_manager.get\_enhanced\_context(ctx.session\_id) assert "memory\_corrections" in context assert "Use ROW for default" in context\["memory\_corrections"\] def test\_get\_enhanced\_context\_selects\_memory\_by\_query(self, session\_manager, tmp\_path): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.add\_message(ctx.session\_id, "user", "看激活数据") project\_dir = tmp\_path / "data" / "memory" / "project\_55" project\_dir.mkdir(parents=True, exist\_ok=True) (project\_dir / "activation.md").write\_text("激活默认映射 activation\_success", encoding="utf-8") (project\_dir / "payment.md").write\_text("支付默认看 payment\_success", encoding="utf-8") (project\_dir / "MEMORY.md").write\_text( "- \[Activation\](activation.md)\\n- \[Payment\](payment.md)\\n", encoding="utf-8", ) context = session\_manager.get\_enhanced\_context(ctx.session\_id, query\_text="看激活数据") assert "activation\_success" in context\["memory\_corrections"\] assert "payment\_success" not in context\["memory\_corrections"\] def test\_get\_enhanced\_context\_no\_memory(self, session\_manager): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.add\_message(ctx.session\_id, "user", "hello") context = session\_manager.get\_enhanced\_context(ctx.session\_id) assert "memory\_corrections" not in context def test\_cleanup\_inactive(self, session\_manager): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) session\_manager.\_sessions\[ctx.session\_id\].last\_active = datetime.now() - timedelta(minutes=61) count = session\_manager.cleanup\_inactive(max\_age\_minutes=60) assert count == 1 assert session\_manager.get\_session(ctx.session\_id) is None def test\_cleanup\_inactive\_keeps\_active(self, session\_manager): ctx = session\_manager.create\_or\_get(None, "user\_1", 55) count = session\_manager.cleanup\_inactive(max\_age\_minutes=60) assert count == 0 assert session\_manager.get\_session(ctx.session\_id) is ctx def test\_jsonl\_recovery\_with\_corrupted\_lines(self, session\_manager): sid = "corrupt\_test" # 写一条有效记录和一条损坏记录 session\_manager.storage.append(sid, { "type": "message", "role": "user", "content": "valid message", "timestamp": datetime.now().isoformat(), "metadata": {}, }) # 写入损坏行 path = session\_manager.storage.\_session\_path(sid) with open(path, "a", encoding="utf-8") as f: f.write("{{{broken\\n") ctx = session\_manager.create\_or\_get(sid, "user\_1", 55) assert ctx is not None assert len(ctx.messages) == 1 assert ctx.messages\[0\].content == "valid message"

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

"""SessionManager 集成测试"""

import json

from datetime import datetime, timedelta

class TestSessionManager:

"""service/session\_manager.py"""

def test\_create\_or\_get\_new\_session(self, session\_manager):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

assert ctx.session\_id

assert ctx.user\_id \== "user\_1"

assert ctx.project\_id \== 55

def test\_create\_or\_get\_returns\_existing(self, session\_manager):

ctx1 \= session\_manager.create\_or\_get(None, "user\_1", 55)

ctx2 \= session\_manager.create\_or\_get(ctx1.session\_id, "user\_1", 55)

assert ctx1 is ctx2

def test\_create\_or\_get\_restores\_from\_jsonl(self, session\_manager):

sid \= "restore\_test\_123"

\# 通过 storage 写入 JSONL 记录

session\_manager.storage.append(sid, {

"type": "message",

"role": "user",

"content": "hello from file",

"timestamp": datetime.now().isoformat(),

"metadata": {},

})

session\_manager.storage.append(sid, {

"type": "query\_state",

"data": {

"project\_id": 55,

"event": "app\_launch",

"metric": "pv",

"time\_range": {"type": "last\_n\_days", "n": 7},

"region\_filter": \["ROW"\],

"group\_by": \["country"\],

"filters": \[\],

"turn\_type": "followup\_patch",

},

"turn\_type": "followup\_patch",

"timestamp": datetime.now().isoformat(),

})

session\_manager.storage.append(sid, {

"type": "session\_meta",

"pending\_task\_id": "task\_123",

"last\_turn\_type": "followup\_patch",

"last\_user\_query": "hello from file",

"turn\_index": 1,

"timestamp": datetime.now().isoformat(),

})

ctx \= session\_manager.create\_or\_get(sid, "user\_1", 55)

assert ctx is not None

assert len(ctx.messages) \== 1

assert ctx.messages\[0\].content \== "hello from file"

assert ctx.last\_query\_state is not None

assert ctx.last\_query\_state.event \== "app\_launch"

assert ctx.last\_turn\_type \== "followup\_patch"

assert ctx.pending\_task\_id \== "task\_123"

assert ctx.last\_user\_query \== "hello from file"

assert ctx.turn\_index \== 1

def test\_create\_or\_get\_new\_when\_jsonl\_missing(self, session\_manager):

ctx \= session\_manager.create\_or\_get("nonexistent\_id", "user\_1", 55)

assert ctx is not None

assert ctx.session\_id \== "nonexistent\_id"

assert len(ctx.messages) \== 0

def test\_add\_message\_stores\_in\_memory(self, session\_manager):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.add\_message(ctx.session\_id, "user", "hello")

assert len(ctx.messages) \== 1

assert ctx.messages\[0\].content \== "hello"

assert ctx.messages\[0\].role \== "user"

assert ctx.last\_user\_query \== "hello"

assert ctx.turn\_index \== 1

def test\_add\_message\_persists\_to\_jsonl(self, session\_manager):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.add\_message(ctx.session\_id, "user", "hello", metadata\={"test": True})

records \= session\_manager.storage.read\_tail(ctx.session\_id)

assert len(records) \== 1

assert records\[0\]\["type"\] \== "message"

assert records\[0\]\["content"\] \== "hello"

def test\_add\_message\_truncates\_memory(self, tmp\_path):

from service.session\_manager import SessionManager

sm \= SessionManager(data\_path\=str(tmp\_path / "data"), max\_messages\=3)

ctx \= sm.create\_or\_get(None, "user\_1", 55)

for i in range(5):

sm.add\_message(ctx.session\_id, "user", f"msg\_{i}")

\# 内存截断到 3 条

assert len(ctx.messages) \== 3

assert ctx.messages\[0\].content \== "msg\_2"

\# JSONL 保留全部 5 条

records \= sm.storage.read\_tail(ctx.session\_id)

assert len(records) \== 5

def test\_add\_message\_unknown\_session\_is\_noop(self, session\_manager):

session\_manager.add\_message("nonexistent", "user", "hello")

\# 不崩溃即可

def test\_update\_query\_state\_stores\_in\_memory(self, session\_manager, sample\_query\_state):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

sample\_query\_state.turn\_type \= "followup\_patch"

session\_manager.update\_query\_state(ctx.session\_id, sample\_query\_state)

assert ctx.last\_query\_state is not None

assert ctx.last\_query\_state.event \== "app\_launch"

assert ctx.last\_turn\_type \== "followup\_patch"

def test\_update\_query\_state\_persists\_to\_jsonl(self, session\_manager, sample\_query\_state):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

sample\_query\_state.turn\_type \= "followup\_patch"

session\_manager.update\_query\_state(ctx.session\_id, sample\_query\_state)

records \= session\_manager.storage.read\_tail(ctx.session\_id)

state\_records \= \[r for r in records if r.get("type") \== "query\_state"\]

assert len(state\_records) \== 1

assert state\_records\[0\]\["data"\]\["event"\] \== "app\_launch"

assert state\_records\[0\]\["turn\_type"\] \== "followup\_patch"

def test\_update\_pending\_task\_stores\_in\_memory(self, session\_manager):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.update\_pending\_task(ctx.session\_id, "task\_1")

assert ctx.pending\_task\_id \== "task\_1"

def test\_update\_pending\_task\_persists\_to\_jsonl(self, session\_manager):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.update\_pending\_task(ctx.session\_id, "task\_1")

records \= session\_manager.storage.read\_tail(ctx.session\_id)

meta\_records \= \[r for r in records if r.get("type") \== "session\_meta"\]

assert len(meta\_records) \== 1

assert meta\_records\[0\]\["pending\_task\_id"\] \== "task\_1"

def test\_get\_enhanced\_context\_includes\_session(self, session\_manager, sample\_query\_state):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.add\_message(ctx.session\_id, "user", "查询PV")

sample\_query\_state.turn\_type \= "new\_query"

session\_manager.update\_query\_state(ctx.session\_id, sample\_query\_state)

context \= session\_manager.get\_enhanced\_context(ctx.session\_id)

assert "recent\_queries" in context

assert "查询PV" in context\["recent\_queries"\]

assert "last\_query\_state" in context

assert context\["last\_query\_state"\]\["event"\] \== "app\_launch"

assert context\["last\_turn\_type"\] \== "new\_query"

assert context\["last\_user\_query"\] \== "查询PV"

assert context\["turn\_index"\] \== 1

def test\_get\_enhanced\_context\_includes\_memory(self, session\_manager, tmp\_path):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.add\_message(ctx.session\_id, "user", "hello")

\# 创建 memory 文件

global\_dir \= tmp\_path / "data" / "memory" / "\_global"

global\_dir.mkdir(parents\=True, exist\_ok\=True)

(global\_dir / "corrections.md").write\_text("Use ROW for default", encoding\="utf-8")

(global\_dir / "MEMORY.md").write\_text("- \[Corrections\](corrections.md)\\n", encoding\="utf-8")

context \= session\_manager.get\_enhanced\_context(ctx.session\_id)

assert "memory\_corrections" in context

assert "Use ROW for default" in context\["memory\_corrections"\]

def test\_get\_enhanced\_context\_selects\_memory\_by\_query(self, session\_manager, tmp\_path):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.add\_message(ctx.session\_id, "user", "看激活数据")

project\_dir \= tmp\_path / "data" / "memory" / "project\_55"

project\_dir.mkdir(parents\=True, exist\_ok\=True)

(project\_dir / "activation.md").write\_text("激活默认映射 activation\_success", encoding\="utf-8")

(project\_dir / "payment.md").write\_text("支付默认看 payment\_success", encoding\="utf-8")

(project\_dir / "MEMORY.md").write\_text(

"- \[Activation\](activation.md)\\n\- \[Payment\](payment.md)\\n",

encoding\="utf-8",

)

context \= session\_manager.get\_enhanced\_context(ctx.session\_id, query\_text\="看激活数据")

assert "activation\_success" in context\["memory\_corrections"\]

assert "payment\_success" not in context\["memory\_corrections"\]

def test\_get\_enhanced\_context\_no\_memory(self, session\_manager):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.add\_message(ctx.session\_id, "user", "hello")

context \= session\_manager.get\_enhanced\_context(ctx.session\_id)

assert "memory\_corrections" not in context

def test\_cleanup\_inactive(self, session\_manager):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

session\_manager.\_sessions\[ctx.session\_id\].last\_active \= datetime.now() \- timedelta(minutes\=61)

count \= session\_manager.cleanup\_inactive(max\_age\_minutes\=60)

assert count \== 1

assert session\_manager.get\_session(ctx.session\_id) is None

def test\_cleanup\_inactive\_keeps\_active(self, session\_manager):

ctx \= session\_manager.create\_or\_get(None, "user\_1", 55)

count \= session\_manager.cleanup\_inactive(max\_age\_minutes\=60)

assert count \== 0

assert session\_manager.get\_session(ctx.session\_id) is ctx

def test\_jsonl\_recovery\_with\_corrupted\_lines(self, session\_manager):

sid \= "corrupt\_test"

\# 写一条有效记录和一条损坏记录

session\_manager.storage.append(sid, {

"type": "message",

"role": "user",

"content": "valid message",

"timestamp": datetime.now().isoformat(),

"metadata": {},

})

\# 写入损坏行

path \= session\_manager.storage.\_session\_path(sid)

with open(path, "a", encoding\="utf-8") as f:

f.write("{{{broken\\n")

ctx \= session\_manager.create\_or\_get(sid, "user\_1", 55)

assert ctx is not None

assert len(ctx.messages) \== 1

assert ctx.messages\[0\].content \== "valid message"