# Source: https://github.com/frankzh0330/query-agent/blob/master/common/__init__.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/common/__init__.py)

Open commit details

History

21 lines (19 loc) · 445 Bytes

## FilesExpand file tree

master

/

# \_\_init\_\_.py

Copy path

Top

## File metadata and controls

- Code

- Blame

21 lines (19 loc) · 445 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/common/__init__.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from common.bearer\_client import BearerClient from common.http\_client import HttpClient from common.text\_utils import ( contains\_chinese, is\_single\_chinese\_char, normalize, tokenize\_mixed, ) from common.types import MatcherType \_\_all\_\_ = \[ "HttpClient", "BearerClient", "normalize", "tokenize\_mixed", "contains\_chinese", "is\_single\_chinese\_char", "MatcherType", \]

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

from \_\_future\_\_ import annotations

from common.bearer\_client import BearerClient

from common.http\_client import HttpClient

from common.text\_utils import (

contains\_chinese,

is\_single\_chinese\_char,

normalize,

tokenize\_mixed,

)

from common.types import MatcherType

\_\_all\_\_ \= \[

"HttpClient",

"BearerClient",

"normalize",

"tokenize\_mixed",

"contains\_chinese",

"is\_single\_chinese\_char",

"MatcherType",

\]