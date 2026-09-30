# Source: https://github.com/frankzh0330/query-agent/blob/master/dsl/renderer.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# renderer.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/dsl/renderer.py)

Open commit details

History

117 lines (109 loc) · 4.47 KB

## FilesExpand file tree

master

/

# renderer.py

Copy path

Top

## File metadata and controls

- Code

- Blame

117 lines (109 loc) · 4.47 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/dsl/renderer.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import uuid from typing import Any, Dict from dsl.semantic\_models import SemanticDSL from dsl.time\_utils import last\_n\_days\_spans\_utc from matcher.catalog\_loader import Catalog def render\_exec\_dsl(canon: SemanticDSL, catalog: Catalog) -> Dict\[str, Any\]: start\_ts, end\_ts = last\_n\_days\_spans\_utc(canon.time\_range.n) metric\_info = catalog.metrics\[canon.metric.metric\_id\] event\_info = catalog.events\[canon.event.event\_name\] dim\_id = canon.group\_by\[0\].dimension\_id if canon.group\_by else "country" dim\_info = catalog.dimensions\[dim\_id\] show\_key = str(uuid.uuid4()) query\_filters = \[f.model\_dump() for f in canon.filters\] return { "use\_app\_cloud\_id": True, "periods": \[ { "granularity": "day", "align\_unit": "day", "timezone": canon.timezone, "week\_start": 1, "type": "past\_range", "spans": \[\[ {"offset": 0, "type": "timestamp", "timestamp": str(start\_ts)}, {"offset": 86399, "type": "timestamp", "timestamp": str(end\_ts)}, \]\], }, { "granularity": "all", "align\_unit": "day", "timezone": canon.timezone, "week\_start": 1, "skip\_period": True, "type": "past\_range", "spans": \[\[ {"offset": 0, "type": "timestamp", "timestamp": str(start\_ts)}, {"offset": 86399, "type": "timestamp", "timestamp": str(end\_ts)}, \]\], }, \], "version": 3, "content": { "profile\_groups\_v2": \[\], "profile\_filters": \[ {"expression": {"logic": "and", "expressions": \[{"logic": "or", "conditions": \[\]}\]}, "show\_name": "All Users", "show\_label": "1"} \], "orders": \[\], "query\_type": "event", "queries": \[ { "indicator\_show\_name": metric\_info\["indicator\_show\_name"\], "measure\_info": {}, "event\_indicator": metric\_info\["event\_indicator"\], "show\_name": event\_info\["show\_name"\], "show\_label": "A", "event\_name": canon.event.event\_name, "event\_type": event\_info.get("event\_type", "origin"), "filters": query\_filters, "groups\_v2": \[ { "property\_compose\_type": dim\_info\["property\_compose\_type"\], "property\_name": dim\_info\["property\_name"\], "property\_type": dim\_info\["property\_type"\], } \], "extra": {}, "show\_key": show\_key, "event\_id": None, "page": {"offset": 0, "limit": 50}, "option": { "chart\_type": "line", "use\_sample\_data": False, "refresh\_cache": False, "skip\_cache": True, "fusion": False, "insight": {}, "analysis\_subject": {}, "is\_pie": False, "skip\_period\_restrict": False, "ignored\_by\_au": False, "query\_trigger\_type": None, "query\_trigger": "modify\_condition", "cnch\_config\_map": {}, "source": "finder", "max\_execute\_time": 300, }, } \], "option": { "blend": {"status": True, "base": 0, "base\_period": True}, "transpose": False, "finder": { "sort\_rule": "conf\_order", "region\_filter": canon.region\_filter, # 只复制 canonical "custom\_tag": {}, }, }, "result\_id": 0, "privacy\_extra": {}, "cross\_region\_data\_ready\_date": "20260201", "max\_execute\_time": 300, "resources": \[{"project\_ids": \[canon.project\_id\], "subject\_ids": \[\], "app\_ids": \[\]}\], "app\_ids": \[\], "finder\_option": {}, "restriction": {}, }, }

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

from \_\_future\_\_ import annotations

import uuid

from typing import Any, Dict

from dsl.semantic\_models import SemanticDSL

from dsl.time\_utils import last\_n\_days\_spans\_utc

from matcher.catalog\_loader import Catalog

def render\_exec\_dsl(canon: SemanticDSL, catalog: Catalog) \-> Dict\[str, Any\]:

start\_ts, end\_ts \= last\_n\_days\_spans\_utc(canon.time\_range.n)

metric\_info \= catalog.metrics\[canon.metric.metric\_id\]

event\_info \= catalog.events\[canon.event.event\_name\]

dim\_id \= canon.group\_by\[0\].dimension\_id if canon.group\_by else "country"

dim\_info \= catalog.dimensions\[dim\_id\]

show\_key \= str(uuid.uuid4())

query\_filters \= \[f.model\_dump() for f in canon.filters\]

return {

"use\_app\_cloud\_id": True,

"periods": \[

{

"granularity": "day",

"align\_unit": "day",

"timezone": canon.timezone,

"week\_start": 1,

"type": "past\_range",

"spans": \[\[

{"offset": 0, "type": "timestamp", "timestamp": str(start\_ts)},

{"offset": 86399, "type": "timestamp", "timestamp": str(end\_ts)},

\]\],

},

{

"granularity": "all",

"align\_unit": "day",

"timezone": canon.timezone,

"week\_start": 1,

"skip\_period": True,

"type": "past\_range",

"spans": \[\[

{"offset": 0, "type": "timestamp", "timestamp": str(start\_ts)},

{"offset": 86399, "type": "timestamp", "timestamp": str(end\_ts)},

\]\],

},

\],

"version": 3,

"content": {

"profile\_groups\_v2": \[\],

"profile\_filters": \[

{"expression": {"logic": "and", "expressions": \[{"logic": "or", "conditions": \[\]}\]},

"show\_name": "All Users", "show\_label": "1"}

\],

"orders": \[\],

"query\_type": "event",

"queries": \[

{

"indicator\_show\_name": metric\_info\["indicator\_show\_name"\],

"measure\_info": {},

"event\_indicator": metric\_info\["event\_indicator"\],

"show\_name": event\_info\["show\_name"\],

"show\_label": "A",

"event\_name": canon.event.event\_name,

"event\_type": event\_info.get("event\_type", "origin"),

"filters": query\_filters,

"groups\_v2": \[

{

"property\_compose\_type": dim\_info\["property\_compose\_type"\],

"property\_name": dim\_info\["property\_name"\],

"property\_type": dim\_info\["property\_type"\],

}

\],

"extra": {},

"show\_key": show\_key,

"event\_id": None,

"page": {"offset": 0, "limit": 50},

"option": {

"chart\_type": "line",

"use\_sample\_data": False,

"refresh\_cache": False,

"skip\_cache": True,

"fusion": False,

"insight": {},

"analysis\_subject": {},

"is\_pie": False,

"skip\_period\_restrict": False,

"ignored\_by\_au": False,

"query\_trigger\_type": None,

"query\_trigger": "modify\_condition",

"cnch\_config\_map": {},

"source": "finder",

"max\_execute\_time": 300,

},

}

\],

"option": {

"blend": {"status": True, "base": 0, "base\_period": True},

"transpose": False,

"finder": {

"sort\_rule": "conf\_order",

"region\_filter": canon.region\_filter, \# 只复制 canonical

"custom\_tag": {},

},

},

"result\_id": 0,

"privacy\_extra": {},

"cross\_region\_data\_ready\_date": "20260201",

"max\_execute\_time": 300,

"resources": \[{"project\_ids": \[canon.project\_id\], "subject\_ids": \[\], "app\_ids": \[\]}\],

"app\_ids": \[\],

"finder\_option": {},

"restriction": {},

},

}