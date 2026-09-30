# Source: https://github.com/frankzh0330/query-agent/blob/master/dsl/semantic_models.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# semantic\_models.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/dsl/semantic_models.py)

Open commit details

History

53 lines (35 loc) · 1.14 KB

## FilesExpand file tree

master

/

# semantic\_models.py

Copy path

Top

## File metadata and controls

- Code

- Blame

53 lines (35 loc) · 1.14 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/dsl/semantic_models.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from typing import Any, List from pydantic import BaseModel, Field, field\_validator ALLOWED\_REGION\_CODES = {"EUTTP", "USTTP", "ROW"} class TimeRange(BaseModel): type: str = "last\_n\_days" n: int = 7 class Metric(BaseModel): metric\_id: str = "pv" class Event(BaseModel): event\_name: str = "app\_launch" class GroupBy(BaseModel): dimension\_id: str = "country" class Filter(BaseModel): property\_name: str op: str value: Any class SemanticDSL(BaseModel): dsl\_version: str = "sem\_v1" registry\_version: str = "2026-02-18" timezone: str = "UTC" project\_id: int region\_filter: List\[str\] = Field(default\_factory=list) metric: Metric event: Event time\_range: TimeRange group\_by: List\[GroupBy\] = Field(default\_factory=list) filters: List\[Filter\] = Field(default\_factory=list) @field\_validator("region\_filter") @classmethod def \_check\_region(cls, v): bad = \[x for x in v if x not in ALLOWED\_REGION\_CODES\] if bad: raise ValueError(f"Invalid region\_filter: {bad}, allowed={sorted(ALLOWED\_REGION\_CODES)}") return v

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

from \_\_future\_\_ import annotations

from typing import Any, List

from pydantic import BaseModel, Field, field\_validator

ALLOWED\_REGION\_CODES \= {"EUTTP", "USTTP", "ROW"}

class TimeRange(BaseModel):

type: str \= "last\_n\_days"

n: int \= 7

class Metric(BaseModel):

metric\_id: str \= "pv"

class Event(BaseModel):

event\_name: str \= "app\_launch"

class GroupBy(BaseModel):

dimension\_id: str \= "country"

class Filter(BaseModel):

property\_name: str

op: str

value: Any

class SemanticDSL(BaseModel):

dsl\_version: str \= "sem\_v1"

registry\_version: str \= "2026-02-18"

timezone: str \= "UTC"

project\_id: int

region\_filter: List\[str\] \= Field(default\_factory\=list)

metric: Metric

event: Event

time\_range: TimeRange

group\_by: List\[GroupBy\] \= Field(default\_factory\=list)

filters: List\[Filter\] \= Field(default\_factory\=list)

@field\_validator("region\_filter")

@classmethod

def \_check\_region(cls, v):

bad \= \[x for x in v if x not in ALLOWED\_REGION\_CODES\]

if bad:

raise ValueError(f"Invalid region\_filter: {bad}, allowed={sorted(ALLOWED\_REGION\_CODES)}")

return v