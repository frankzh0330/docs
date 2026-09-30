# Source: https://github.com/frankzh0330/query-agent/blob/master/service/bearer_service.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# bearer\_service.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[fix: improve early\_exit handling, logging config, and tool calling fl…](https://github.com/frankzh0330/query-agent/commit/874b818935aa4d48f0c46b5a83ab4b31069dcdfa)

Open commit details

Apr 15, 2026

[874b818](https://github.com/frankzh0330/query-agent/commit/874b818935aa4d48f0c46b5a83ab4b31069dcdfa) · Apr 15, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/service/bearer_service.py)

Open commit details

History

109 lines (90 loc) · 3.34 KB

## FilesExpand file tree

master

/

# bearer\_service.py

Copy path

Top

## File metadata and controls

- Code

- Blame

109 lines (90 loc) · 3.34 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/bearer_service.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import asyncio import json import logging import os from typing import Any, Dict, Optional from common.bearer\_client import BearerClient logger = logging.getLogger(\_\_name\_\_) # 全局客户端实例 \_bearer\_client: Optional\[BearerClient\] = None def \_is\_mock\_mode() -> bool: """检查是否启用 Mock 模式""" return os.getenv("BEARER\_MOCK", "false").lower() == "true" def get\_bearer\_client() -> BearerClient: """获取 Bearer 客户端单例""" global \_bearer\_client if \_bearer\_client is None: base\_url = os.getenv("BEARER\_URL", "http://localhost:8080") token = os.getenv("BEARER\_TOKEN", "XXX") \_bearer\_client = BearerClient(base\_url=base\_url, token=token) logger.info(f"BearerClient initialized: base\_url={base\_url}") return \_bearer\_client def \_mock\_query\_result(exec\_dsl: Dict\[str, Any\]) -> Dict\[str, Any\]: """生成 Mock 查询结果""" # 从 exec\_dsl 中提取一些信息用于生成假数据 region\_filter = exec\_dsl.get("content", {}).get("option", {}).get("finder", {}).get("region\_filter", \["ROW"\]) metric\_id = exec\_dsl.get("content", {}).get("queries", \[{}\])\[0\].get("event\_indicator", "pv") event\_name = exec\_dsl.get("content", {}).get("queries", \[{}\])\[0\].get("event\_name", "app\_launch") # 生成假数据 mock\_data = { "status": "success", "data": { "result": \[ { "date": "2024-03-24", f"{metric\_id}": 12345, "region": region\_filter\[0\] if region\_filter else "ROW", }, { "date": "2024-03-23", f"{metric\_id}": 11234, "region": region\_filter\[0\] if region\_filter else "ROW", }, { "date": "2024-03-22", f"{metric\_id}": 10234, "region": region\_filter\[0\] if region\_filter else "ROW", }, \] }, "meta": { "event": event\_name, "metric": metric\_id, "regions": region\_filter, "mock": True, "message": "This is mock data" } } logger.info(f"Mock query result: region={region\_filter}, metric={metric\_id}, event={event\_name}") return mock\_data async def execute\_query(exec\_dsl: Dict\[str, Any\]) -> Dict\[str, Any\]: """ 执行查询请求 :param exec\_dsl: 执行 DSL :return: 查询结果 """ # Mock 模式 if \_is\_mock\_mode(): # logger.info("Using MOCK mode for Bearer query") # 模拟网络延迟 await asyncio.sleep(0.5) result = \_mock\_query\_result(exec\_dsl) logger.info(f"Mock query result: {json.dumps(result, ensure\_ascii=False)}") return result # 正常模式 client = get\_bearer\_client() try: result = await client.query(exec\_dsl) logger.info("Query executed successfully") return result except Exception as e: logger.error(f"Query execution failed: {e}") raise async def check\_bearer\_health() -> bool: """检查 Bearer 服务健康状态""" if \_is\_mock\_mode(): logger.info("Mock mode: Bearer health check always returns True") return True client = get\_bearer\_client() return await client.health\_check()

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

from \_\_future\_\_ import annotations

import asyncio

import json

import logging

import os

from typing import Any, Dict, Optional

from common.bearer\_client import BearerClient

logger \= logging.getLogger(\_\_name\_\_)

\# 全局客户端实例

\_bearer\_client: Optional\[BearerClient\] \= None

def \_is\_mock\_mode() \-> bool:

"""检查是否启用 Mock 模式"""

return os.getenv("BEARER\_MOCK", "false").lower() \== "true"

def get\_bearer\_client() \-> BearerClient:

"""获取 Bearer 客户端单例"""

global \_bearer\_client

if \_bearer\_client is None:

base\_url \= os.getenv("BEARER\_URL", "http://localhost:8080")

token \= os.getenv("BEARER\_TOKEN", "XXX")

\_bearer\_client \= BearerClient(base\_url\=base\_url, token\=token)

logger.info(f"BearerClient initialized: base\_url={base\_url}")

return \_bearer\_client

def \_mock\_query\_result(exec\_dsl: Dict\[str, Any\]) \-> Dict\[str, Any\]:

"""生成 Mock 查询结果"""

\# 从 exec\_dsl 中提取一些信息用于生成假数据

region\_filter \= exec\_dsl.get("content", {}).get("option", {}).get("finder", {}).get("region\_filter", \["ROW"\])

metric\_id \= exec\_dsl.get("content", {}).get("queries", \[{}\])\[0\].get("event\_indicator", "pv")

event\_name \= exec\_dsl.get("content", {}).get("queries", \[{}\])\[0\].get("event\_name", "app\_launch")

\# 生成假数据

mock\_data \= {

"status": "success",

"data": {

"result": \[

{

"date": "2024-03-24",

f"{metric\_id}": 12345,

"region": region\_filter\[0\] if region\_filter else "ROW",

},

{

"date": "2024-03-23",

f"{metric\_id}": 11234,

"region": region\_filter\[0\] if region\_filter else "ROW",

},

{

"date": "2024-03-22",

f"{metric\_id}": 10234,

"region": region\_filter\[0\] if region\_filter else "ROW",

},

\]

},

"meta": {

"event": event\_name,

"metric": metric\_id,

"regions": region\_filter,

"mock": True,

"message": "This is mock data"

}

}

logger.info(f"Mock query result: region={region\_filter}, metric={metric\_id}, event={event\_name}")

return mock\_data

async def execute\_query(exec\_dsl: Dict\[str, Any\]) \-> Dict\[str, Any\]:

"""

执行查询请求

:param exec\_dsl: 执行 DSL

:return: 查询结果

"""

\# Mock 模式

if \_is\_mock\_mode():

\# logger.info("Using MOCK mode for Bearer query")

\# 模拟网络延迟

await asyncio.sleep(0.5)

result \= \_mock\_query\_result(exec\_dsl)

logger.info(f"Mock query result: {json.dumps(result, ensure\_ascii\=False)}")

return result

\# 正常模式

client \= get\_bearer\_client()

try:

result \= await client.query(exec\_dsl)

logger.info("Query executed successfully")

return result

except Exception as e:

logger.error(f"Query execution failed: {e}")

raise

async def check\_bearer\_health() \-> bool:

"""检查 Bearer 服务健康状态"""

if \_is\_mock\_mode():

logger.info("Mock mode: Bearer health check always returns True")

return True

client \= get\_bearer\_client()

return await client.health\_check()