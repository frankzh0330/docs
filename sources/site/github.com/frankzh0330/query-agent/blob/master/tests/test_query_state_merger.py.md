# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_query_state_merger.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_query\_state\_merger.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_query_state_merger.py)

Open commit details

History

78 lines (69 loc) · 2.82 KB

## FilesExpand file tree

master

/

# test\_query\_state\_merger.py

Copy path

Top

## File metadata and controls

- Code

- Blame

78 lines (69 loc) · 2.82 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_query_state_merger.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""query\_state\_merger.py tests.""" from service.query\_state\_merger import merge\_query\_state from service.session\_models import QueryState class TestMergeQueryState: def test\_merge\_time\_patch\_keeps\_other\_fields(self, sample\_query\_state): merged = merge\_query\_state( previous=sample\_query\_state, extracted\_patch={"time\_range": {"type": "last\_n\_days", "n": 1}}, project\_id=55, ) assert merged.time\_range == {"type": "last\_n\_days", "n": 1} assert merged.event == "app\_launch" assert merged.metric == "pv" assert "time\_range" in merged.explicit\_fields assert "event" in merged.inherited\_fields assert merged.field\_sources\["metric"\] == "inherited" def test\_merge\_metric\_patch\_overrides\_metric\_only(self, sample\_query\_state): merged = merge\_query\_state( previous=sample\_query\_state, extracted\_patch={"metric": "uv"}, project\_id=55, ) assert merged.metric == "uv" assert merged.event == "app\_launch" assert merged.field\_sources\["metric"\] == "explicit" def test\_merge\_region\_patch\_overrides\_region\_only(self, sample\_query\_state): merged = merge\_query\_state( previous=sample\_query\_state, extracted\_patch={"region\_filter": \["USTTP"\]}, project\_id=55, ) assert merged.region\_filter == \["USTTP"\] assert merged.group\_by == \["country"\] def test\_merge\_multiple\_fields(self, sample\_query\_state): merged = merge\_query\_state( previous=sample\_query\_state, extracted\_patch={ "metric": "uv", "time\_range": {"type": "last\_n\_days", "n": 30}, "group\_by": \["channel"\], }, project\_id=55, ) assert merged.metric == "uv" assert merged.time\_range\["n"\] == 30 assert merged.group\_by == \["channel"\] assert sorted(merged.explicit\_fields) == \["group\_by", "metric", "time\_range"\] def test\_merge\_without\_previous\_creates\_new\_state(self): merged = merge\_query\_state( previous=None, extracted\_patch={ "event": "payment\_success", "metric": "uv", }, project\_id=60, ) assert merged.project\_id == 60 assert merged.event == "payment\_success" assert merged.metric == "uv" assert merged.inherited\_fields == \[\] assert merged.turn\_type == "new\_query" def test\_merge\_does\_not\_mutate\_previous\_state(self, sample\_query\_state): original = sample\_query\_state.to\_dict() merge\_query\_state( previous=sample\_query\_state, extracted\_patch={"metric": "uv"}, project\_id=55, ) assert sample\_query\_state.to\_dict() == original

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

"""query\_state\_merger.py tests."""

from service.query\_state\_merger import merge\_query\_state

from service.session\_models import QueryState

class TestMergeQueryState:

def test\_merge\_time\_patch\_keeps\_other\_fields(self, sample\_query\_state):

merged \= merge\_query\_state(

previous\=sample\_query\_state,

extracted\_patch\={"time\_range": {"type": "last\_n\_days", "n": 1}},

project\_id\=55,

)

assert merged.time\_range \== {"type": "last\_n\_days", "n": 1}

assert merged.event \== "app\_launch"

assert merged.metric \== "pv"

assert "time\_range" in merged.explicit\_fields

assert "event" in merged.inherited\_fields

assert merged.field\_sources\["metric"\] \== "inherited"

def test\_merge\_metric\_patch\_overrides\_metric\_only(self, sample\_query\_state):

merged \= merge\_query\_state(

previous\=sample\_query\_state,

extracted\_patch\={"metric": "uv"},

project\_id\=55,

)

assert merged.metric \== "uv"

assert merged.event \== "app\_launch"

assert merged.field\_sources\["metric"\] \== "explicit"

def test\_merge\_region\_patch\_overrides\_region\_only(self, sample\_query\_state):

merged \= merge\_query\_state(

previous\=sample\_query\_state,

extracted\_patch\={"region\_filter": \["USTTP"\]},

project\_id\=55,

)

assert merged.region\_filter \== \["USTTP"\]

assert merged.group\_by \== \["country"\]

def test\_merge\_multiple\_fields(self, sample\_query\_state):

merged \= merge\_query\_state(

previous\=sample\_query\_state,

extracted\_patch\={

"metric": "uv",

"time\_range": {"type": "last\_n\_days", "n": 30},

"group\_by": \["channel"\],

},

project\_id\=55,

)

assert merged.metric \== "uv"

assert merged.time\_range\["n"\] \== 30

assert merged.group\_by \== \["channel"\]

assert sorted(merged.explicit\_fields) \== \["group\_by", "metric", "time\_range"\]

def test\_merge\_without\_previous\_creates\_new\_state(self):

merged \= merge\_query\_state(

previous\=None,

extracted\_patch\={

"event": "payment\_success",

"metric": "uv",

},

project\_id\=60,

)

assert merged.project\_id \== 60

assert merged.event \== "payment\_success"

assert merged.metric \== "uv"

assert merged.inherited\_fields \== \[\]

assert merged.turn\_type \== "new\_query"

def test\_merge\_does\_not\_mutate\_previous\_state(self, sample\_query\_state):

original \= sample\_query\_state.to\_dict()

merge\_query\_state(

previous\=sample\_query\_state,

extracted\_patch\={"metric": "uv"},

project\_id\=55,

)

assert sample\_query\_state.to\_dict() \== original