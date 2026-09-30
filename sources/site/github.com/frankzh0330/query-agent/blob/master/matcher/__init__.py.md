# Source: https://github.com/frankzh0330/query-agent/blob/master/matcher/__init__.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# \_\_init\_\_.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/matcher/__init__.py)

Open commit details

History

32 lines (30 loc) · 898 Bytes

## FilesExpand file tree

master

/

# \_\_init\_\_.py

Copy path

Top

## File metadata and controls

- Code

- Blame

32 lines (30 loc) · 898 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/matcher/__init__.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from matcher.base import ( DEFAULT\_STOPWORDS, DEFAULT\_SYNONYM\_MAP, BaseMatcher, MatchResult, ) from matcher.dimension\_matcher import ( DimensionMatcher, build\_dimension\_matcher\_from\_catalog, ) from matcher.event\_matcher import EventMatcher, build\_event\_matcher\_from\_catalog from matcher.matcher\_service import MatcherService from matcher.metric\_matcher import MetricMatcher, build\_metric\_matcher\_from\_catalog from matcher.time\_matcher import TimeMatcher, resolve\_last\_n\_days \_\_all\_\_ = \[ "BaseMatcher", "MatchResult", "DEFAULT\_STOPWORDS", "DEFAULT\_SYNONYM\_MAP", "EventMatcher", "build\_event\_matcher\_from\_catalog", "DimensionMatcher", "build\_dimension\_matcher\_from\_catalog", "MetricMatcher", "build\_metric\_matcher\_from\_catalog", "TimeMatcher", "resolve\_last\_n\_days", "MatcherService", \]

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

from \_\_future\_\_ import annotations

from matcher.base import (

DEFAULT\_STOPWORDS,

DEFAULT\_SYNONYM\_MAP,

BaseMatcher,

MatchResult,

)

from matcher.dimension\_matcher import (

DimensionMatcher,

build\_dimension\_matcher\_from\_catalog,

)

from matcher.event\_matcher import EventMatcher, build\_event\_matcher\_from\_catalog

from matcher.matcher\_service import MatcherService

from matcher.metric\_matcher import MetricMatcher, build\_metric\_matcher\_from\_catalog

from matcher.time\_matcher import TimeMatcher, resolve\_last\_n\_days

\_\_all\_\_ \= \[

"BaseMatcher",

"MatchResult",

"DEFAULT\_STOPWORDS",

"DEFAULT\_SYNONYM\_MAP",

"EventMatcher",

"build\_event\_matcher\_from\_catalog",

"DimensionMatcher",

"build\_dimension\_matcher\_from\_catalog",

"MetricMatcher",

"build\_metric\_matcher\_from\_catalog",

"TimeMatcher",

"resolve\_last\_n\_days",

"MatcherService",

\]