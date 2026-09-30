# Source: https://github.com/frankzh0330/query-agent/blob/master/dsl/time_utils.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# time\_utils.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/dsl/time_utils.py)

Open commit details

History

15 lines (10 loc) · 486 Bytes

## FilesExpand file tree

master

/

# time\_utils.py

Copy path

Top

## File metadata and controls

- Code

- Blame

15 lines (10 loc) · 486 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/dsl/time_utils.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from datetime import datetime, timedelta, timezone def utc\_day\_start(dt: datetime) -> datetime: dt = dt.astimezone(timezone.utc) return datetime(dt.year, dt.month, dt.day, 0, 0, 0, tzinfo=timezone.utc) def last\_n\_days\_spans\_utc(n: int) -> tuple\[int, int\]: now = datetime.now(timezone.utc) end\_day = utc\_day\_start(now) start\_day = end\_day - timedelta(days=n - 1) return int(start\_day.timestamp()), int(end\_day.timestamp())

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

from \_\_future\_\_ import annotations

from datetime import datetime, timedelta, timezone

def utc\_day\_start(dt: datetime) \-> datetime:

dt \= dt.astimezone(timezone.utc)

return datetime(dt.year, dt.month, dt.day, 0, 0, 0, tzinfo\=timezone.utc)

def last\_n\_days\_spans\_utc(n: int) \-> tuple\[int, int\]:

now \= datetime.now(timezone.utc)

end\_day \= utc\_day\_start(now)

start\_day \= end\_day \- timedelta(days\=n \- 1)

return int(start\_day.timestamp()), int(end\_day.timestamp())