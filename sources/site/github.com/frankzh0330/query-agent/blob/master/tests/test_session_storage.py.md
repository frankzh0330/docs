# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_session_storage.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_session\_storage.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_session_storage.py)

Open commit details

History

130 lines (103 loc) · 5.13 KB

## FilesExpand file tree

master

/

# test\_session\_storage.py

Copy path

Top

## File metadata and controls

- Code

- Blame

130 lines (103 loc) · 5.13 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_session_storage.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""SessionStorage JSONL 持久化测试""" import json class TestSessionStorage: """memory/storage/memory\_file.py""" def test\_append\_creates\_file(self, session\_storage): session\_storage.append("sess\_1", {"type": "message", "text": "hello"}) path = session\_storage.\_session\_path("sess\_1") assert path.exists() def test\_append\_writes\_valid\_jsonl(self, session\_storage): records = \[ {"type": "message", "text": "first"}, {"type": "message", "text": "second"}, {"type": "message", "text": "third"}, \] for r in records: session\_storage.append("sess\_1", r) path = session\_storage.\_session\_path("sess\_1") lines = path.read\_text(encoding="utf-8").strip().split("\\n") assert len(lines) == 3 for i, line in enumerate(lines): parsed = json.loads(line) assert parsed\["text"\] == records\[i\]\["text"\] def test\_read\_tail\_returns\_last\_n(self, session\_storage): for i in range(10): session\_storage.append("sess\_1", {"type": "message", "idx": i}) result = session\_storage.read\_tail("sess\_1", n=3) assert len(result) == 3 assert \[r\["idx"\] for r in result\] == \[7, 8, 9\] def test\_read\_tail\_nonexistent\_session(self, session\_storage): result = session\_storage.read\_tail("nonexistent") assert result == \[\] def test\_read\_tail\_empty\_file(self, session\_storage): path = session\_storage.\_session\_path("empty\_sess") path.parent.mkdir(parents=True, exist\_ok=True) path.write\_text("", encoding="utf-8") result = session\_storage.read\_tail("empty\_sess") assert result == \[\] def test\_read\_last\_state\_finds\_query\_state(self, session\_storage): session\_storage.append("sess\_1", {"type": "message", "text": "hello"}) session\_storage.append("sess\_1", { "type": "query\_state", "data": {"event": "app\_launch", "metric": "pv"}, }) session\_storage.append("sess\_1", {"type": "message", "text": "world"}) session\_storage.append("sess\_1", { "type": "query\_state", "data": {"event": "payment", "metric": "uv"}, }) result = session\_storage.read\_last\_state("sess\_1") assert result is not None assert result\["event"\] == "payment" assert result\["metric"\] == "uv" def test\_read\_last\_state\_no\_state\_records(self, session\_storage): session\_storage.append("sess\_1", {"type": "message", "text": "hello"}) session\_storage.append("sess\_1", {"type": "message", "text": "world"}) result = session\_storage.read\_last\_state("sess\_1") assert result is None def test\_read\_last\_state\_nonexistent(self, session\_storage): result = session\_storage.read\_last\_state("nonexistent") assert result is None def test\_corrupted\_line\_skipped(self, session\_storage): path = session\_storage.\_session\_path("sess\_1") path.parent.mkdir(parents=True, exist\_ok=True) with open(path, "w", encoding="utf-8") as f: f.write(json.dumps({"type": "message", "text": "valid1"}) + "\\n") f.write("{{{broken json\\n") f.write(json.dumps({"type": "message", "text": "valid2"}) + "\\n") result = session\_storage.read\_tail("sess\_1") assert len(result) == 2 assert result\[0\]\["text"\] == "valid1" assert result\[1\]\["text"\] == "valid2" def test\_all\_corrupted\_lines(self, session\_storage): path = session\_storage.\_session\_path("sess\_1") path.parent.mkdir(parents=True, exist\_ok=True) with open(path, "w", encoding="utf-8") as f: f.write("broken1\\n") f.write("broken2\\n") result = session\_storage.read\_tail("sess\_1") assert result == \[\] def test\_delete\_removes\_file(self, session\_storage): session\_storage.append("sess\_1", {"type": "message"}) assert session\_storage.\_session\_path("sess\_1").exists() session\_storage.delete("sess\_1") assert not session\_storage.\_session\_path("sess\_1").exists() def test\_delete\_nonexistent\_is\_noop(self, session\_storage): session\_storage.delete("nonexistent") # should not raise def test\_list\_sessions(self, session\_storage): for sid in \["sess\_a", "sess\_b", "sess\_c"\]: session\_storage.append(sid, {"type": "message"}) sessions = session\_storage.list\_sessions() assert set(sessions) == {"sess\_a", "sess\_b", "sess\_c"} def test\_list\_sessions\_empty(self, session\_storage): assert session\_storage.list\_sessions() == \[\] def test\_append\_multiple\_sessions\_independent(self, session\_storage): session\_storage.append("sess\_a", {"type": "message", "text": "a1"}) session\_storage.append("sess\_b", {"type": "message", "text": "b1"}) session\_storage.append("sess\_a", {"type": "message", "text": "a2"}) result\_a = session\_storage.read\_tail("sess\_a") result\_b = session\_storage.read\_tail("sess\_b") assert len(result\_a) == 2 assert len(result\_b) == 1 assert result\_a\[0\]\["text"\] == "a1" assert result\_b\[0\]\["text"\] == "b1"

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

"""SessionStorage JSONL 持久化测试"""

import json

class TestSessionStorage:

"""memory/storage/memory\_file.py"""

def test\_append\_creates\_file(self, session\_storage):

session\_storage.append("sess\_1", {"type": "message", "text": "hello"})

path \= session\_storage.\_session\_path("sess\_1")

assert path.exists()

def test\_append\_writes\_valid\_jsonl(self, session\_storage):

records \= \[

{"type": "message", "text": "first"},

{"type": "message", "text": "second"},

{"type": "message", "text": "third"},

\]

for r in records:

session\_storage.append("sess\_1", r)

path \= session\_storage.\_session\_path("sess\_1")

lines \= path.read\_text(encoding\="utf-8").strip().split("\\n")

assert len(lines) \== 3

for i, line in enumerate(lines):

parsed \= json.loads(line)

assert parsed\["text"\] \== records\[i\]\["text"\]

def test\_read\_tail\_returns\_last\_n(self, session\_storage):

for i in range(10):

session\_storage.append("sess\_1", {"type": "message", "idx": i})

result \= session\_storage.read\_tail("sess\_1", n\=3)

assert len(result) \== 3

assert \[r\["idx"\] for r in result\] \== \[7, 8, 9\]

def test\_read\_tail\_nonexistent\_session(self, session\_storage):

result \= session\_storage.read\_tail("nonexistent")

assert result \== \[\]

def test\_read\_tail\_empty\_file(self, session\_storage):

path \= session\_storage.\_session\_path("empty\_sess")

path.parent.mkdir(parents\=True, exist\_ok\=True)

path.write\_text("", encoding\="utf-8")

result \= session\_storage.read\_tail("empty\_sess")

assert result \== \[\]

def test\_read\_last\_state\_finds\_query\_state(self, session\_storage):

session\_storage.append("sess\_1", {"type": "message", "text": "hello"})

session\_storage.append("sess\_1", {

"type": "query\_state",

"data": {"event": "app\_launch", "metric": "pv"},

})

session\_storage.append("sess\_1", {"type": "message", "text": "world"})

session\_storage.append("sess\_1", {

"type": "query\_state",

"data": {"event": "payment", "metric": "uv"},

})

result \= session\_storage.read\_last\_state("sess\_1")

assert result is not None

assert result\["event"\] \== "payment"

assert result\["metric"\] \== "uv"

def test\_read\_last\_state\_no\_state\_records(self, session\_storage):

session\_storage.append("sess\_1", {"type": "message", "text": "hello"})

session\_storage.append("sess\_1", {"type": "message", "text": "world"})

result \= session\_storage.read\_last\_state("sess\_1")

assert result is None

def test\_read\_last\_state\_nonexistent(self, session\_storage):

result \= session\_storage.read\_last\_state("nonexistent")

assert result is None

def test\_corrupted\_line\_skipped(self, session\_storage):

path \= session\_storage.\_session\_path("sess\_1")

path.parent.mkdir(parents\=True, exist\_ok\=True)

with open(path, "w", encoding\="utf-8") as f:

f.write(json.dumps({"type": "message", "text": "valid1"}) + "\\n")

f.write("{{{broken json\\n")

f.write(json.dumps({"type": "message", "text": "valid2"}) + "\\n")

result \= session\_storage.read\_tail("sess\_1")

assert len(result) \== 2

assert result\[0\]\["text"\] \== "valid1"

assert result\[1\]\["text"\] \== "valid2"

def test\_all\_corrupted\_lines(self, session\_storage):

path \= session\_storage.\_session\_path("sess\_1")

path.parent.mkdir(parents\=True, exist\_ok\=True)

with open(path, "w", encoding\="utf-8") as f:

f.write("broken1\\n")

f.write("broken2\\n")

result \= session\_storage.read\_tail("sess\_1")

assert result \== \[\]

def test\_delete\_removes\_file(self, session\_storage):

session\_storage.append("sess\_1", {"type": "message"})

assert session\_storage.\_session\_path("sess\_1").exists()

session\_storage.delete("sess\_1")

assert not session\_storage.\_session\_path("sess\_1").exists()

def test\_delete\_nonexistent\_is\_noop(self, session\_storage):

session\_storage.delete("nonexistent") \# should not raise

def test\_list\_sessions(self, session\_storage):

for sid in \["sess\_a", "sess\_b", "sess\_c"\]:

session\_storage.append(sid, {"type": "message"})

sessions \= session\_storage.list\_sessions()

assert set(sessions) \== {"sess\_a", "sess\_b", "sess\_c"}

def test\_list\_sessions\_empty(self, session\_storage):

assert session\_storage.list\_sessions() \== \[\]

def test\_append\_multiple\_sessions\_independent(self, session\_storage):

session\_storage.append("sess\_a", {"type": "message", "text": "a1"})

session\_storage.append("sess\_b", {"type": "message", "text": "b1"})

session\_storage.append("sess\_a", {"type": "message", "text": "a2"})

result\_a \= session\_storage.read\_tail("sess\_a")

result\_b \= session\_storage.read\_tail("sess\_b")

assert len(result\_a) \== 2

assert len(result\_b) \== 1

assert result\_a\[0\]\["text"\] \== "a1"

assert result\_b\[0\]\["text"\] \== "b1"