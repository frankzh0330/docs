# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_task_manager.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_task\_manager.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_task_manager.py)

Open commit details

History

177 lines (149 loc) · 7.62 KB

## FilesExpand file tree

master

/

# test\_task\_manager.py

Copy path

Top

## File metadata and controls

- Code

- Blame

177 lines (149 loc) · 7.62 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_task_manager.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""TaskManager 确认流生命周期测试""" from datetime import datetime, timedelta class TestTaskManager: """service/task\_manager.py""" def test\_create\_task\_returns\_task\_context(self, task\_manager): task = task\_manager.create\_task( session\_id="sess\_1", raw\_query="purchase的情况", extraction={"event": "purchase"}, user\_id="user\_1", candidates={"event": \[{"value": "payment\_submit", "score": 0.88}\]}, ) assert task.task\_id assert task.status == "waiting\_confirmation" assert task.session\_id == "sess\_1" assert task.raw\_query == "purchase的情况" def test\_create\_task\_generates\_unique\_ids(self, task\_manager): t1 = task\_manager.create\_task("s1", "q1", {}) t2 = task\_manager.create\_task("s1", "q2", {}) assert t1.task\_id != t2.task\_id def test\_get\_task\_returns\_task(self, task\_manager): created = task\_manager.create\_task("s1", "q1", {}) found = task\_manager.get\_task(created.task\_id) assert found is created def test\_get\_task\_nonexistent\_returns\_none(self, task\_manager): assert task\_manager.get\_task("nonexistent") is None def test\_get\_task\_expired\_returns\_none(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}) # 手动设置 created\_at 为 31 分钟前 task\_manager.\_tasks\[task.task\_id\].created\_at = datetime.now() - timedelta(minutes=31) assert task\_manager.get\_task(task.task\_id) is None def test\_get\_task\_expired\_marks\_status(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}) task\_manager.\_tasks\[task.task\_id\].created\_at = datetime.now() - timedelta(minutes=31) task\_manager.get\_task(task.task\_id) assert task.status == "expired" def test\_confirm\_task\_records\_selection(self, task\_manager): task = task\_manager.create\_task( "s1", "q1", {}, candidates={"event": \[{"value": "payment\_success", "score": 0.55}\]}, partial\_query\_state=QueryState(project\_id=55), ) result = task\_manager.confirm\_task(task.task\_id, "event", "payment\_success") assert result.user\_selection\["event"\] == "payment\_success" def test\_confirm\_task\_patches\_query\_state(self, task\_manager): from service.session\_models import QueryState qs = QueryState(project\_id=55) task\_manager.create\_task( "s1", "q1", {}, candidates={"event": \[{"value": "x", "score": 0.5}\]}, partial\_query\_state=qs, ) task\_id = task\_manager.list\_tasks()\[0\].task\_id task\_manager.confirm\_task(task\_id, "event", "payment\_success") assert qs.event == "payment\_success" def test\_confirm\_task\_idempotent(self, task\_manager): task = task\_manager.create\_task( "s1", "q1", {}, candidates={"event": \[{"value": "x", "score": 0.5}\]}, ) task\_manager.confirm\_task(task.task\_id, "event", "a") task\_manager.confirm\_task(task.task\_id, "event", "b") assert task.user\_selection\["event"\] == "b" def test\_confirm\_task\_partial\_confirm(self, task\_manager): task = task\_manager.create\_task( "s1", "q1", {}, candidates={ "event": \[{"value": "x", "score": 0.5}\], "metric": \[{"value": "pv", "score": 0.6}\], }, ) task\_manager.confirm\_task(task.task\_id, "event", "x") assert task.status == "waiting\_confirmation" def test\_confirm\_task\_all\_confirmed(self, task\_manager): task = task\_manager.create\_task( "s1", "q1", {}, candidates={ "event": \[{"value": "x", "score": 0.5}\], "metric": \[{"value": "pv", "score": 0.6}\], }, ) task\_manager.confirm\_task(task.task\_id, "event", "x") task\_manager.confirm\_task(task.task\_id, "metric", "pv") assert task.status == "confirmed" def test\_confirm\_task\_expired\_returns\_none(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}, candidates={"event": \[\]}) task\_manager.\_tasks\[task.task\_id\].created\_at = datetime.now() - timedelta(minutes=31) result = task\_manager.confirm\_task(task.task\_id, "event", "x") assert result is None def test\_update\_status\_valid\_transition(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}) r1 = task\_manager.update\_status(task.task\_id, "confirmed") assert r1.status == "confirmed" r2 = task\_manager.update\_status(task.task\_id, "completed") assert r2.status == "completed" def test\_update\_status\_invalid\_transition(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}) result = task\_manager.update\_status(task.task\_id, "completed") # waiting\_confirmation → completed is invalid assert result.status == "waiting\_confirmation" def test\_update\_status\_cancelled\_from\_waiting(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}) result = task\_manager.update\_status(task.task\_id, "cancelled") assert result.status == "cancelled" def test\_expire\_task(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}) task\_manager.expire\_task(task.task\_id) assert task.status == "expired" def test\_cleanup\_expired(self, task\_manager): fresh = task\_manager.create\_task("s1", "q1", {}) old = task\_manager.create\_task("s1", "q2", {}) task\_manager.\_tasks\[old.task\_id\].created\_at = datetime.now() - timedelta(minutes=31) count = task\_manager.cleanup\_expired() assert count == 1 assert task\_manager.get\_task(fresh.task\_id) is fresh def test\_list\_tasks\_all(self, task\_manager): task\_manager.create\_task("s1", "q1", {}) task\_manager.create\_task("s2", "q2", {}) assert len(task\_manager.list\_tasks()) == 2 def test\_list\_tasks\_filtered\_by\_session(self, task\_manager): task\_manager.create\_task("s1", "q1", {}) task\_manager.create\_task("s2", "q2", {}) assert len(task\_manager.list\_tasks(session\_id="s1")) == 1 def test\_update\_status\_nonexistent\_returns\_none(self, task\_manager): assert task\_manager.update\_status("nonexistent", "confirmed") is None def test\_create\_task\_persists\_to\_storage(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}, candidates={"event": \[{"value": "x", "score": 0.5}\]}) latest = task\_manager.storage.read\_latest(task.task\_id) assert latest is not None assert latest\["status"\] == "waiting\_confirmation" assert latest\["session\_id"\] == "s1" def test\_get\_task\_restores\_from\_storage(self, task\_manager): task = task\_manager.create\_task("s1", "q1", {}, candidates={"event": \[{"value": "x", "score": 0.5}\]}) task\_manager.\_tasks.clear() restored = task\_manager.get\_task(task.task\_id) assert restored is not None assert restored.task\_id == task.task\_id assert restored.session\_id == "s1" def test\_confirm\_task\_persists\_updated\_state(self, task\_manager): task = task\_manager.create\_task( "s1", "q1", {}, candidates={"event": \[{"value": "payment\_success", "score": 0.55}\]}, ) task\_manager.confirm\_task(task.task\_id, "event", "payment\_success") latest = task\_manager.storage.read\_latest(task.task\_id) assert latest\["user\_selection"\]\["event"\] == "payment\_success" assert latest\["status"\] == "confirmed" # 避免顶层 import 冲突，在测试方法内引用 from service.session\_models import QueryState

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

"""TaskManager 确认流生命周期测试"""

from datetime import datetime, timedelta

class TestTaskManager:

"""service/task\_manager.py"""

def test\_create\_task\_returns\_task\_context(self, task\_manager):

task \= task\_manager.create\_task(

session\_id\="sess\_1",

raw\_query\="purchase的情况",

extraction\={"event": "purchase"},

user\_id\="user\_1",

candidates\={"event": \[{"value": "payment\_submit", "score": 0.88}\]},

)

assert task.task\_id

assert task.status \== "waiting\_confirmation"

assert task.session\_id \== "sess\_1"

assert task.raw\_query \== "purchase的情况"

def test\_create\_task\_generates\_unique\_ids(self, task\_manager):

t1 \= task\_manager.create\_task("s1", "q1", {})

t2 \= task\_manager.create\_task("s1", "q2", {})

assert t1.task\_id != t2.task\_id

def test\_get\_task\_returns\_task(self, task\_manager):

created \= task\_manager.create\_task("s1", "q1", {})

found \= task\_manager.get\_task(created.task\_id)

assert found is created

def test\_get\_task\_nonexistent\_returns\_none(self, task\_manager):

assert task\_manager.get\_task("nonexistent") is None

def test\_get\_task\_expired\_returns\_none(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {})

\# 手动设置 created\_at 为 31 分钟前

task\_manager.\_tasks\[task.task\_id\].created\_at \= datetime.now() \- timedelta(minutes\=31)

assert task\_manager.get\_task(task.task\_id) is None

def test\_get\_task\_expired\_marks\_status(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {})

task\_manager.\_tasks\[task.task\_id\].created\_at \= datetime.now() \- timedelta(minutes\=31)

task\_manager.get\_task(task.task\_id)

assert task.status \== "expired"

def test\_confirm\_task\_records\_selection(self, task\_manager):

task \= task\_manager.create\_task(

"s1", "q1", {},

candidates\={"event": \[{"value": "payment\_success", "score": 0.55}\]},

partial\_query\_state\=QueryState(project\_id\=55),

)

result \= task\_manager.confirm\_task(task.task\_id, "event", "payment\_success")

assert result.user\_selection\["event"\] \== "payment\_success"

def test\_confirm\_task\_patches\_query\_state(self, task\_manager):

from service.session\_models import QueryState

qs \= QueryState(project\_id\=55)

task\_manager.create\_task(

"s1", "q1", {},

candidates\={"event": \[{"value": "x", "score": 0.5}\]},

partial\_query\_state\=qs,

)

task\_id \= task\_manager.list\_tasks()\[0\].task\_id

task\_manager.confirm\_task(task\_id, "event", "payment\_success")

assert qs.event \== "payment\_success"

def test\_confirm\_task\_idempotent(self, task\_manager):

task \= task\_manager.create\_task(

"s1", "q1", {},

candidates\={"event": \[{"value": "x", "score": 0.5}\]},

)

task\_manager.confirm\_task(task.task\_id, "event", "a")

task\_manager.confirm\_task(task.task\_id, "event", "b")

assert task.user\_selection\["event"\] \== "b"

def test\_confirm\_task\_partial\_confirm(self, task\_manager):

task \= task\_manager.create\_task(

"s1", "q1", {},

candidates\={

"event": \[{"value": "x", "score": 0.5}\],

"metric": \[{"value": "pv", "score": 0.6}\],

},

)

task\_manager.confirm\_task(task.task\_id, "event", "x")

assert task.status \== "waiting\_confirmation"

def test\_confirm\_task\_all\_confirmed(self, task\_manager):

task \= task\_manager.create\_task(

"s1", "q1", {},

candidates\={

"event": \[{"value": "x", "score": 0.5}\],

"metric": \[{"value": "pv", "score": 0.6}\],

},

)

task\_manager.confirm\_task(task.task\_id, "event", "x")

task\_manager.confirm\_task(task.task\_id, "metric", "pv")

assert task.status \== "confirmed"

def test\_confirm\_task\_expired\_returns\_none(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {}, candidates\={"event": \[\]})

task\_manager.\_tasks\[task.task\_id\].created\_at \= datetime.now() \- timedelta(minutes\=31)

result \= task\_manager.confirm\_task(task.task\_id, "event", "x")

assert result is None

def test\_update\_status\_valid\_transition(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {})

r1 \= task\_manager.update\_status(task.task\_id, "confirmed")

assert r1.status \== "confirmed"

r2 \= task\_manager.update\_status(task.task\_id, "completed")

assert r2.status \== "completed"

def test\_update\_status\_invalid\_transition(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {})

result \= task\_manager.update\_status(task.task\_id, "completed")

\# waiting\_confirmation → completed is invalid

assert result.status \== "waiting\_confirmation"

def test\_update\_status\_cancelled\_from\_waiting(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {})

result \= task\_manager.update\_status(task.task\_id, "cancelled")

assert result.status \== "cancelled"

def test\_expire\_task(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {})

task\_manager.expire\_task(task.task\_id)

assert task.status \== "expired"

def test\_cleanup\_expired(self, task\_manager):

fresh \= task\_manager.create\_task("s1", "q1", {})

old \= task\_manager.create\_task("s1", "q2", {})

task\_manager.\_tasks\[old.task\_id\].created\_at \= datetime.now() \- timedelta(minutes\=31)

count \= task\_manager.cleanup\_expired()

assert count \== 1

assert task\_manager.get\_task(fresh.task\_id) is fresh

def test\_list\_tasks\_all(self, task\_manager):

task\_manager.create\_task("s1", "q1", {})

task\_manager.create\_task("s2", "q2", {})

assert len(task\_manager.list\_tasks()) \== 2

def test\_list\_tasks\_filtered\_by\_session(self, task\_manager):

task\_manager.create\_task("s1", "q1", {})

task\_manager.create\_task("s2", "q2", {})

assert len(task\_manager.list\_tasks(session\_id\="s1")) \== 1

def test\_update\_status\_nonexistent\_returns\_none(self, task\_manager):

assert task\_manager.update\_status("nonexistent", "confirmed") is None

def test\_create\_task\_persists\_to\_storage(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {}, candidates\={"event": \[{"value": "x", "score": 0.5}\]})

latest \= task\_manager.storage.read\_latest(task.task\_id)

assert latest is not None

assert latest\["status"\] \== "waiting\_confirmation"

assert latest\["session\_id"\] \== "s1"

def test\_get\_task\_restores\_from\_storage(self, task\_manager):

task \= task\_manager.create\_task("s1", "q1", {}, candidates\={"event": \[{"value": "x", "score": 0.5}\]})

task\_manager.\_tasks.clear()

restored \= task\_manager.get\_task(task.task\_id)

assert restored is not None

assert restored.task\_id \== task.task\_id

assert restored.session\_id \== "s1"

def test\_confirm\_task\_persists\_updated\_state(self, task\_manager):

task \= task\_manager.create\_task(

"s1", "q1", {},

candidates\={"event": \[{"value": "payment\_success", "score": 0.55}\]},

)

task\_manager.confirm\_task(task.task\_id, "event", "payment\_success")

latest \= task\_manager.storage.read\_latest(task.task\_id)

assert latest\["user\_selection"\]\["event"\] \== "payment\_success"

assert latest\["status"\] \== "confirmed"

\# 避免顶层 import 冲突，在测试方法内引用

from service.session\_models import QueryState