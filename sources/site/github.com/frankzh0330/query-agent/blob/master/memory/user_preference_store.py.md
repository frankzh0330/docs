# Source: https://github.com/frankzh0330/query-agent/blob/master/memory/user_preference_store.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# user\_preference\_store.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/memory/user_preference_store.py)

Open commit details

History

134 lines (115 loc) · 4.44 KB

## FilesExpand file tree

master

/

# user\_preference\_store.py

Copy path

Top

## File metadata and controls

- Code

- Blame

134 lines (115 loc) · 4.44 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/memory/user_preference_store.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""User-scoped preference signals for post-recall reranking.""" from \_\_future\_\_ import annotations import json import logging import math from pathlib import Path from typing import Any, Dict, Tuple logger = logging.getLogger(\_\_name\_\_) \_FIELD\_TO\_BUCKET = { "event": "events", "metric": "metrics", "group\_by": "dimensions", "dimension": "dimensions", } \_MAX\_BIAS\_SCORE = 6.0 class UserPreferenceStore: """Append-light JSON preference storage scoped by project\_id + user\_id.""" def \_\_init\_\_(self, data\_path: str = "data/user\_preferences"): self.data\_path = Path(data\_path) self.data\_path.mkdir(parents=True, exist\_ok=True) def record\_selection( self, project\_id: int, user\_id: str | None, \*, event: str | None = None, metric: str | None = None, group\_by: list\[str\] | None = None, ) -> None: if not user\_id: return data = self.\_load(project\_id, user\_id) updated = False if event: self.\_increment(data\["events"\], event) updated = True if metric: self.\_increment(data\["metrics"\], metric) updated = True if group\_by: for dimension in group\_by: self.\_increment(data\["dimensions"\], dimension) updated = True if updated: self.\_save(project\_id, user\_id, data) def get\_bucket(self, project\_id: int, user\_id: str | None, field\_name: str) -> Dict\[str, int\]: if not user\_id: return {} bucket = \_FIELD\_TO\_BUCKET.get(field\_name) if not bucket: return {} data = self.\_load(project\_id, user\_id) return data.get(bucket, {}) def rerank\_candidates( self, project\_id: int, user\_id: str | None, field\_name: str, candidates: list\[Dict\[str, Any\]\], ) -> Tuple\[list\[Dict\[str, Any\]\], Dict\[str, Any\]\]: if not user\_id or not candidates: return candidates, {"applied": False, "reason": "no\_user\_or\_candidates"} bucket = self.get\_bucket(project\_id, user\_id, field\_name) if not bucket: return candidates, {"applied": False, "reason": "no\_preference\_data"} reranked = \[\] for candidate in candidates: raw\_score = float(candidate.get("score", 0.0)) value = candidate.get("value") count = int(bucket.get(value, 0)) bias = min(\_MAX\_BIAS\_SCORE, round(math.log2(count + 1) \* 2.0, 2)) if count > 0 else 0.0 reranked.append( { \*\*candidate, "raw\_score": raw\_score, "preference\_count": count, "preference\_bias": bias, "score": round(raw\_score + bias, 2), } ) reranked.sort( key=lambda item: (float(item.get("score", 0.0)), float(item.get("raw\_score", 0.0))), reverse=True, ) return reranked, { "applied": True, "field": field\_name, "top\_preference\_hits": \[ { "value": item\["value"\], "count": item\["preference\_count"\], "bias": item\["preference\_bias"\], } for item in reranked\[:3\] if item.get("preference\_count", 0) > 0 \], } def \_path(self, project\_id: int, user\_id: str) -> Path: safe\_user = user\_id.replace("/", "\_") return self.data\_path / f"project\_{project\_id}\_\_user\_{safe\_user}.json" def \_load(self, project\_id: int, user\_id: str) -> Dict\[str, Dict\[str, int\]\]: path = self.\_path(project\_id, user\_id) if not path.exists(): return {"events": {}, "metrics": {}, "dimensions": {}} try: return json.loads(path.read\_text(encoding="utf-8")) except (OSError, json.JSONDecodeError): logger.warning("Failed to load user preferences: project=%s user=%s", project\_id, user\_id) return {"events": {}, "metrics": {}, "dimensions": {}} def \_save(self, project\_id: int, user\_id: str, data: Dict\[str, Dict\[str, int\]\]) -> None: path = self.\_path(project\_id, user\_id) path.write\_text(json.dumps(data, ensure\_ascii=False, indent=2), encoding="utf-8") @staticmethod def \_increment(bucket: Dict\[str, int\], key: str) -> None: bucket\[key\] = int(bucket.get(key, 0)) + 1

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

"""User-scoped preference signals for post-recall reranking."""

from \_\_future\_\_ import annotations

import json

import logging

import math

from pathlib import Path

from typing import Any, Dict, Tuple

logger \= logging.getLogger(\_\_name\_\_)

\_FIELD\_TO\_BUCKET \= {

"event": "events",

"metric": "metrics",

"group\_by": "dimensions",

"dimension": "dimensions",

}

\_MAX\_BIAS\_SCORE \= 6.0

class UserPreferenceStore:

"""Append-light JSON preference storage scoped by project\_id + user\_id."""

def \_\_init\_\_(self, data\_path: str \= "data/user\_preferences"):

self.data\_path \= Path(data\_path)

self.data\_path.mkdir(parents\=True, exist\_ok\=True)

def record\_selection(

self,

project\_id: int,

user\_id: str | None,

\*,

event: str | None \= None,

metric: str | None \= None,

group\_by: list\[str\] | None \= None,

) \-> None:

if not user\_id:

return

data \= self.\_load(project\_id, user\_id)

updated \= False

if event:

self.\_increment(data\["events"\], event)

updated \= True

if metric:

self.\_increment(data\["metrics"\], metric)

updated \= True

if group\_by:

for dimension in group\_by:

self.\_increment(data\["dimensions"\], dimension)

updated \= True

if updated:

self.\_save(project\_id, user\_id, data)

def get\_bucket(self, project\_id: int, user\_id: str | None, field\_name: str) \-> Dict\[str, int\]:

if not user\_id:

return {}

bucket \= \_FIELD\_TO\_BUCKET.get(field\_name)

if not bucket:

return {}

data \= self.\_load(project\_id, user\_id)

return data.get(bucket, {})

def rerank\_candidates(

self,

project\_id: int,

user\_id: str | None,

field\_name: str,

candidates: list\[Dict\[str, Any\]\],

) \-> Tuple\[list\[Dict\[str, Any\]\], Dict\[str, Any\]\]:

if not user\_id or not candidates:

return candidates, {"applied": False, "reason": "no\_user\_or\_candidates"}

bucket \= self.get\_bucket(project\_id, user\_id, field\_name)

if not bucket:

return candidates, {"applied": False, "reason": "no\_preference\_data"}

reranked \= \[\]

for candidate in candidates:

raw\_score \= float(candidate.get("score", 0.0))

value \= candidate.get("value")

count \= int(bucket.get(value, 0))

bias \= min(\_MAX\_BIAS\_SCORE, round(math.log2(count + 1) \* 2.0, 2)) if count \> 0 else 0.0

reranked.append(

{

\*\*candidate,

"raw\_score": raw\_score,

"preference\_count": count,

"preference\_bias": bias,

"score": round(raw\_score + bias, 2),

}

)

reranked.sort(

key\=lambda item: (float(item.get("score", 0.0)), float(item.get("raw\_score", 0.0))),

reverse\=True,

)

return reranked, {

"applied": True,

"field": field\_name,

"top\_preference\_hits": \[

{

"value": item\["value"\],

"count": item\["preference\_count"\],

"bias": item\["preference\_bias"\],

}

for item in reranked\[:3\]

if item.get("preference\_count", 0) \> 0

\],

}

def \_path(self, project\_id: int, user\_id: str) \-> Path:

safe\_user \= user\_id.replace("/", "\_")

return self.data\_path / f"project\_{project\_id}\_\_user\_{safe\_user}.json"

def \_load(self, project\_id: int, user\_id: str) \-> Dict\[str, Dict\[str, int\]\]:

path \= self.\_path(project\_id, user\_id)

if not path.exists():

return {"events": {}, "metrics": {}, "dimensions": {}}

try:

return json.loads(path.read\_text(encoding\="utf-8"))

except (OSError, json.JSONDecodeError):

logger.warning("Failed to load user preferences: project=%s user=%s", project\_id, user\_id)

return {"events": {}, "metrics": {}, "dimensions": {}}

def \_save(self, project\_id: int, user\_id: str, data: Dict\[str, Dict\[str, int\]\]) \-> None:

path \= self.\_path(project\_id, user\_id)

path.write\_text(json.dumps(data, ensure\_ascii\=False, indent\=2), encoding\="utf-8")

@staticmethod

def \_increment(bucket: Dict\[str, int\], key: str) \-> None:

bucket\[key\] \= int(bucket.get(key, 0)) + 1