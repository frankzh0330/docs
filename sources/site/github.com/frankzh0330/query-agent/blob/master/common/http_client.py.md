# Source: https://github.com/frankzh0330/query-agent/blob/master/common/http_client.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# http\_client.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/common/http_client.py)

Open commit details

History

35 lines (26 loc) · 1.13 KB

## FilesExpand file tree

master

/

# http\_client.py

Copy path

Top

## File metadata and controls

- Code

- Blame

35 lines (26 loc) · 1.13 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/common/http_client.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations from typing import Any, Dict, Optional import httpx class HttpClient: """HTTP 客户端基类""" def \_\_init\_\_(self, base\_url: str, token: Optional\[str\] = None): self.base\_url = base\_url.rstrip("/") self.token = token async def post(self, path: str, json: Dict\[str, Any\]) -> Dict\[str, Any\]: """发送 POST 请求""" headers = {} if self.token: headers\["Authorization"\] = f"Bearer {self.token}" async with httpx.AsyncClient() as client: resp = await client.post(f"{self.base\_url}{path}", json=json, headers=headers) resp.raise\_for\_status() return resp.json() async def get(self, path: str, params: Optional\[Dict\[str, Any\]\] = None) -> Dict\[str, Any\]: """发送 GET 请求""" headers = {} if self.token: headers\["Authorization"\] = f"Bearer {self.token}" async with httpx.AsyncClient() as client: resp = await client.get(f"{self.base\_url}{path}", params=params, headers=headers) resp.raise\_for\_status() return resp.json()

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

from \_\_future\_\_ import annotations

from typing import Any, Dict, Optional

import httpx

class HttpClient:

"""HTTP 客户端基类"""

def \_\_init\_\_(self, base\_url: str, token: Optional\[str\] \= None):

self.base\_url \= base\_url.rstrip("/")

self.token \= token

async def post(self, path: str, json: Dict\[str, Any\]) \-> Dict\[str, Any\]:

"""发送 POST 请求"""

headers \= {}

if self.token:

headers\["Authorization"\] \= f"Bearer {self.token}"

async with httpx.AsyncClient() as client:

resp \= await client.post(f"{self.base\_url}{path}", json\=json, headers\=headers)

resp.raise\_for\_status()

return resp.json()

async def get(self, path: str, params: Optional\[Dict\[str, Any\]\] \= None) \-> Dict\[str, Any\]:

"""发送 GET 请求"""

headers \= {}

if self.token:

headers\["Authorization"\] \= f"Bearer {self.token}"

async with httpx.AsyncClient() as client:

resp \= await client.get(f"{self.base\_url}{path}", params\=params, headers\=headers)

resp.raise\_for\_status()

return resp.json()