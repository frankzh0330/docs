# Source: https://github.com/frankzh0330/query-agent/blob/master/common/bearer_client.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# bearer\_client.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/common/bearer_client.py)

Open commit details

History

24 lines (17 loc) · 696 Bytes

## FilesExpand file tree

master

/

# bearer\_client.py

Copy path

Top

## File metadata and controls

- Code

- Blame

24 lines (17 loc) · 696 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/common/bearer_client.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from typing import Any, Dict, Optional from common.http\_client import HttpClient class BearerClient(HttpClient): """Bearer API 客户端""" def \_\_init\_\_(self, base\_url: str = "http://localhost:8080", token: str = "XXX"): super().\_\_init\_\_(base\_url, token) async def query(self, exec\_dsl: Dict\[str, Any\]) -> Dict\[str, Any\]: """发送查询请求""" return await self.post("/query", exec\_dsl) async def health\_check(self) -> bool: """健康检查""" try: result = await self.get("/health") return result.get("status") == "ok" except Exception: return False

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

from \_\_future\_\_ import annotations

from typing import Any, Dict, Optional

from common.http\_client import HttpClient

class BearerClient(HttpClient):

"""Bearer API 客户端"""

def \_\_init\_\_(self, base\_url: str \= "http://localhost:8080", token: str \= "XXX"):

super().\_\_init\_\_(base\_url, token)

async def query(self, exec\_dsl: Dict\[str, Any\]) \-> Dict\[str, Any\]:

"""发送查询请求"""

return await self.post("/query", exec\_dsl)

async def health\_check(self) \-> bool:

"""健康检查"""

try:

result \= await self.get("/health")

return result.get("status") \== "ok"

except Exception:

return False