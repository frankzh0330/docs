# Source: https://github.com/frankzh0330/query-agent/blob/master/matcher/catalog_loader.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# catalog\_loader.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[update docs and removed unused codes](https://github.com/frankzh0330/query-agent/commit/28cd5d550776847f844dead59a860a776081b9b4)

May 1, 2026

[28cd5d5](https://github.com/frankzh0330/query-agent/commit/28cd5d550776847f844dead59a860a776081b9b4) · May 1, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/matcher/catalog_loader.py)

Open commit details

History

74 lines (56 loc) · 2.11 KB

## FilesExpand file tree

master

/

# catalog\_loader.py

Copy path

Top

## File metadata and controls

- Code

- Blame

74 lines (56 loc) · 2.11 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/matcher/catalog_loader.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import logging import os from dataclasses import dataclass from typing import Dict import yaml logger = logging.getLogger(\_\_name\_\_) @dataclass(frozen=True) class Catalog: metrics: Dict events: Dict dimensions: Dict metric\_alias\_lookup: Dict\[str, str\] event\_alias\_lookup: Dict\[str, str\] dimension\_alias\_lookup: Dict\[str, str\] def \_load\_yaml(path: str) -> dict: with open(path, "r", encoding="utf-8") as f: return yaml.safe\_load(f) or {} def \_norm(s: str) -> str: return (s or "").strip().lower().replace(" ", "") def load\_catalog(base\_dir: str) -> Catalog: metrics = \_load\_yaml(os.path.join(base\_dir, "metrics.yaml")).get("metrics", {}) events = \_load\_yaml(os.path.join(base\_dir, "events.yaml")).get("events", {}) dimensions = \_load\_yaml(os.path.join(base\_dir, "dimensions.yaml")).get("dimensions", {}) # 构建别名查找表 metric\_alias\_lookup: Dict\[str, str\] = {} for metric\_id, info in metrics.items(): for alias in info.get("aliases", \[\]): nk = \_norm(alias) if nk: metric\_alias\_lookup\[nk\] = metric\_id logger.debug("metric\_alias\_lookup: %s", metric\_alias\_lookup) # 构建事件别名查找表 event\_alias\_lookup: Dict\[str, str\] = {} for event\_id, info in events.items(): for alias in info.get("aliases", \[\]): nk = \_norm(alias) if nk: event\_alias\_lookup\[nk\] = event\_id logger.debug("event\_alias\_lookup: %s", event\_alias\_lookup) # 构建维度别名查找表 dimension\_alias\_lookup: Dict\[str, str\] = {} for dim\_id, info in dimensions.items(): for alias in info.get("aliases", \[\]): nk = \_norm(alias) if nk: dimension\_alias\_lookup\[nk\] = dim\_id logger.debug("dimension\_alias\_lookup: %s", dimension\_alias\_lookup) return Catalog( metrics=metrics, events=events, dimensions=dimensions, metric\_alias\_lookup=metric\_alias\_lookup, event\_alias\_lookup=event\_alias\_lookup, dimension\_alias\_lookup=dimension\_alias\_lookup, )

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

from \_\_future\_\_ import annotations

import logging

import os

from dataclasses import dataclass

from typing import Dict

import yaml

logger \= logging.getLogger(\_\_name\_\_)

@dataclass(frozen\=True)

class Catalog:

metrics: Dict

events: Dict

dimensions: Dict

metric\_alias\_lookup: Dict\[str, str\]

event\_alias\_lookup: Dict\[str, str\]

dimension\_alias\_lookup: Dict\[str, str\]

def \_load\_yaml(path: str) \-> dict:

with open(path, "r", encoding\="utf-8") as f:

return yaml.safe\_load(f) or {}

def \_norm(s: str) \-> str:

return (s or "").strip().lower().replace(" ", "")

def load\_catalog(base\_dir: str) \-> Catalog:

metrics \= \_load\_yaml(os.path.join(base\_dir, "metrics.yaml")).get("metrics", {})

events \= \_load\_yaml(os.path.join(base\_dir, "events.yaml")).get("events", {})

dimensions \= \_load\_yaml(os.path.join(base\_dir, "dimensions.yaml")).get("dimensions", {})

\# 构建别名查找表

metric\_alias\_lookup: Dict\[str, str\] \= {}

for metric\_id, info in metrics.items():

for alias in info.get("aliases", \[\]):

nk \= \_norm(alias)

if nk:

metric\_alias\_lookup\[nk\] \= metric\_id

logger.debug("metric\_alias\_lookup: %s", metric\_alias\_lookup)

\# 构建事件别名查找表

event\_alias\_lookup: Dict\[str, str\] \= {}

for event\_id, info in events.items():

for alias in info.get("aliases", \[\]):

nk \= \_norm(alias)

if nk:

event\_alias\_lookup\[nk\] \= event\_id

logger.debug("event\_alias\_lookup: %s", event\_alias\_lookup)

\# 构建维度别名查找表

dimension\_alias\_lookup: Dict\[str, str\] \= {}

for dim\_id, info in dimensions.items():

for alias in info.get("aliases", \[\]):

nk \= \_norm(alias)

if nk:

dimension\_alias\_lookup\[nk\] \= dim\_id

logger.debug("dimension\_alias\_lookup: %s", dimension\_alias\_lookup)

return Catalog(

metrics\=metrics,

events\=events,

dimensions\=dimensions,

metric\_alias\_lookup\=metric\_alias\_lookup,

event\_alias\_lookup\=event\_alias\_lookup,

dimension\_alias\_lookup\=dimension\_alias\_lookup,

)