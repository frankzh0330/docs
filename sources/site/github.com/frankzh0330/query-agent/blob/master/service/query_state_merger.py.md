# Source: https://github.com/frankzh0330/query-agent/blob/master/service/query_state_merger.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# query\_state\_merger.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/service/query_state_merger.py)

Open commit details

History

78 lines (64 loc) · 2.26 KB

## FilesExpand file tree

master

/

# query\_state\_merger.py

Copy path

Top

## File metadata and controls

- Code

- Blame

78 lines (64 loc) · 2.26 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/query_state_merger.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Utilities for merging follow-up patches into QueryState.""" from \_\_future\_\_ import annotations from copy import deepcopy from typing import Any from service.session\_models import QueryState \_STATE\_FIELDS = ( "event", "metric", "time\_range", "region\_filter", "group\_by", "filters", "chart\_type", "interaction\_mode", ) def merge\_query\_state( previous: QueryState | None, extracted\_patch: dict\[str, Any\], project\_id: int, ) -> QueryState: """将 follow-up patch 合并到上一轮 QueryState。 规则： - patch 中出现的字段视为 explicit - previous 中存在且 patch 未覆盖的字段视为 inherited - previous 不存在时，返回只包含 explicit 字段的新状态 """ patch = deepcopy(extracted\_patch or {}) state = \_clone\_or\_init(previous, project\_id) explicit\_fields: list\[str\] = \[\] inherited\_fields: list\[str\] = \[\] field\_sources: dict\[str, str\] = {} for field\_name in \_STATE\_FIELDS: if field\_name in patch: value = deepcopy(patch\[field\_name\]) setattr(state, field\_name, value) explicit\_fields.append(field\_name) field\_sources\[field\_name\] = "explicit" else: current\_value = getattr(state, field\_name) if \_has\_value(current\_value): inherited\_fields.append(field\_name) field\_sources\[field\_name\] = "inherited" state.project\_id = project\_id state.confidence = patch.get("confidence", state.confidence) state.explicit\_fields = explicit\_fields state.inherited\_fields = inherited\_fields state.field\_sources = field\_sources state.turn\_type = "followup\_patch" if previous is not None else "new\_query" return state def \_clone\_or\_init(previous: QueryState | None, project\_id: int) -> QueryState: if previous is None: return QueryState(project\_id=project\_id) data = previous.to\_dict() data.pop("explicit\_fields", None) data.pop("inherited\_fields", None) data.pop("field\_sources", None) data.pop("turn\_type", None) return QueryState(\*\*deepcopy(data)) def \_has\_value(value: Any) -> bool: if value is None: return False if isinstance(value, (list, dict, str)): return bool(value) return True

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

"""Utilities for merging follow-up patches into QueryState."""

from \_\_future\_\_ import annotations

from copy import deepcopy

from typing import Any

from service.session\_models import QueryState

\_STATE\_FIELDS \= (

"event",

"metric",

"time\_range",

"region\_filter",

"group\_by",

"filters",

"chart\_type",

"interaction\_mode",

)

def merge\_query\_state(

previous: QueryState | None,

extracted\_patch: dict\[str, Any\],

project\_id: int,

) \-> QueryState:

"""将 follow-up patch 合并到上一轮 QueryState。

规则：

\- patch 中出现的字段视为 explicit

\- previous 中存在且 patch 未覆盖的字段视为 inherited

\- previous 不存在时，返回只包含 explicit 字段的新状态

"""

patch \= deepcopy(extracted\_patch or {})

state \= \_clone\_or\_init(previous, project\_id)

explicit\_fields: list\[str\] \= \[\]

inherited\_fields: list\[str\] \= \[\]

field\_sources: dict\[str, str\] \= {}

for field\_name in \_STATE\_FIELDS:

if field\_name in patch:

value \= deepcopy(patch\[field\_name\])

setattr(state, field\_name, value)

explicit\_fields.append(field\_name)

field\_sources\[field\_name\] \= "explicit"

else:

current\_value \= getattr(state, field\_name)

if \_has\_value(current\_value):

inherited\_fields.append(field\_name)

field\_sources\[field\_name\] \= "inherited"

state.project\_id \= project\_id

state.confidence \= patch.get("confidence", state.confidence)

state.explicit\_fields \= explicit\_fields

state.inherited\_fields \= inherited\_fields

state.field\_sources \= field\_sources

state.turn\_type \= "followup\_patch" if previous is not None else "new\_query"

return state

def \_clone\_or\_init(previous: QueryState | None, project\_id: int) \-> QueryState:

if previous is None:

return QueryState(project\_id\=project\_id)

data \= previous.to\_dict()

data.pop("explicit\_fields", None)

data.pop("inherited\_fields", None)

data.pop("field\_sources", None)

data.pop("turn\_type", None)

return QueryState(\*\*deepcopy(data))

def \_has\_value(value: Any) \-> bool:

if value is None:

return False

if isinstance(value, (list, dict, str)):

return bool(value)

return True