# Source: https://github.com/frankzh0330/query-agent/blob/master/dsl/validators.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# validators.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/dsl/validators.py)

Open commit details

History

18 lines (12 loc) · 801 Bytes

## FilesExpand file tree

master

/

# validators.py

Copy path

Top

## File metadata and controls

- Code

- Blame

18 lines (12 loc) · 801 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/dsl/validators.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from typing import Any, Dict from dsl.semantic\_models import ALLOWED\_REGION\_CODES, SemanticDSL def validate\_region(exec\_dsl: Dict\[str, Any\]) -> None: actual = exec\_dsl.get("content", {}).get("option", {}).get("finder", {}).get("region\_filter", \[\]) bad = \[r for r in actual if r not in ALLOWED\_REGION\_CODES\] if bad: raise ValueError(f"Invalid region\_filter values: {bad}. Allowed: {sorted(ALLOWED\_REGION\_CODES)}") def validate\_region\_consistency(canon: SemanticDSL, exec\_dsl: Dict\[str, Any\]) -> None: actual = exec\_dsl.get("content", {}).get("option", {}).get("finder", {}).get("region\_filter", \[\]) if actual != canon.region\_filter: raise ValueError(f"region\_filter mismatch expected={canon.region\_filter}, actual={actual}")

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

from \_\_future\_\_ import annotations

from typing import Any, Dict

from dsl.semantic\_models import ALLOWED\_REGION\_CODES, SemanticDSL

def validate\_region(exec\_dsl: Dict\[str, Any\]) \-> None:

actual \= exec\_dsl.get("content", {}).get("option", {}).get("finder", {}).get("region\_filter", \[\])

bad \= \[r for r in actual if r not in ALLOWED\_REGION\_CODES\]

if bad:

raise ValueError(f"Invalid region\_filter values: {bad}. Allowed: {sorted(ALLOWED\_REGION\_CODES)}")

def validate\_region\_consistency(canon: SemanticDSL, exec\_dsl: Dict\[str, Any\]) \-> None:

actual \= exec\_dsl.get("content", {}).get("option", {}).get("finder", {}).get("region\_filter", \[\])

if actual != canon.region\_filter:

raise ValueError(f"region\_filter mismatch expected={canon.region\_filter}, actual={actual}")