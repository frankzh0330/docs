# Source: https://github.com/frankzh0330/query-agent/blob/master/server.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# server.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/server.py)

Open commit details

History

166 lines (129 loc) · 4.57 KB

## FilesExpand file tree

master

/

# server.py

Copy path

Top

## File metadata and controls

- Code

- Blame

166 lines (129 loc) · 4.57 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/server.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from \_\_future\_\_ import annotations import asyncio import logging import os from contextlib import asynccontextmanager import uvicorn from fastapi import FastAPI # 全局日志配置 LOG\_LEVEL = os.getenv("LOG\_LEVEL", "DEBUG").upper() logging.basicConfig( level=LOG\_LEVEL, format="%(asctime)s %(levelname)-5s \[%(name)s\] %(message)s", datefmt="%H:%M:%S", ) from app import app, set\_matcher\_service from bus import create\_bus from bus.direct\_call\_bus import DirectCallBus from dispatcher.response\_dispatcher import ResponseDispatcher from gateway.telegram\_gateway import TelegramGateway from matcher.matcher\_service import MatcherService from service.catalog\_scheduler import CatalogScheduler from service.catalog\_sync import CatalogSync from worker.agent\_worker import AgentWorker logger = logging.getLogger(\_\_name\_\_) # ==================== # 组件初始化 # ==================== # 全局消息总线 bus = create\_bus() # Agent Worker worker = AgentWorker() # 响应分派器 dispatcher = ResponseDispatcher() # Gateway 管理器 \_gateways: dict\[str, TelegramGateway\] = {} # ==================== # 服务生命周期 # ==================== @asynccontextmanager async def websocket\_lifespan(app: FastAPI): """ 服务生命周期管理 启动时: 1. 初始化 MatcherService（构建索引） 2. 创建 MessageBus + AgentWorker + ResponseDispatcher 3. 启动 Telegram Gateway 4. 启动 Catalog 定时同步调度器 """ logger.info("=== Starting Server ===") # 1. 初始化 MatcherService（构建索引） matcher\_service = MatcherService(catalog\_path="catalog") set\_matcher\_service(matcher\_service) logger.info("MatcherService initialized and registered") # 2. 组装 Bus + Worker + Dispatcher if isinstance(bus, DirectCallBus): bus.bind\_worker(worker) bus.bind\_dispatcher(dispatcher) logger.info("DirectCallBus: worker and dispatcher bound") else: # Redis 模式：启动 worker 和 dispatcher 循环 asyncio.create\_task(worker.run\_loop(bus)) asyncio.create\_task(dispatcher.run\_loop(bus)) logger.info("Redis bus: worker and dispatcher loops started") # 3. 启动 Telegram Gateway telegram\_token = os.getenv("TELEGRAM\_BOT\_TOKEN", "") if telegram\_token: telegram\_gw = TelegramGateway(bot\_token=telegram\_token, bus=bus) \_gateways\["telegram"\] = telegram\_gw # 注册到 Dispatcher dispatcher.register("telegram", telegram\_gw) # 在后台运行 Gateway async def run\_gateway(): await telegram\_gw.start() asyncio.create\_task(run\_gateway()) logger.info("TelegramGateway started in background") else: logger.warning("TELEGRAM\_BOT\_TOKEN not set, skipping TelegramGateway") # 4. 启动 Catalog 定时同步调度器 catalog\_api\_base = os.getenv("CATALOG\_API\_BASE") if catalog\_api\_base: catalog\_sync = CatalogSync(api\_base=catalog\_api\_base, catalog\_dir="catalog") catalog\_scheduler = CatalogScheduler(catalog\_sync, matcher\_service) catalog\_scheduler.start() logger.info(f"CatalogScheduler started with API: {catalog\_api\_base}") else: catalog\_scheduler = None logger.info("CATALOG\_API\_BASE not set, skipping CatalogScheduler") # 5. 启动 Session 定时清理 from app import session\_manager async def \_session\_cleanup\_loop(): while True: await asyncio.sleep(300) # 每 5 分钟 try: session\_manager.cleanup\_inactive(max\_age\_minutes=60) except Exception as e: logger.warning("Session cleanup failed: %s", e) asyncio.create\_task(\_session\_cleanup\_loop()) logger.info("Session cleanup loop started (interval=5min, max\_age=60min)") yield # 清理 logger.info("=== Stopping Server ===") for gw in \_gateways.values(): await gw.stop() if catalog\_scheduler: catalog\_scheduler.shutdown() # 创建带生命周期的 FastAPI 应用 app\_with\_ws = FastAPI( title="query-agent: WebSocket + HTTP", lifespan=websocket\_lifespan, ) # 直接挂载原有路由（不使用 /api 前缀） for route in app.routes: app\_with\_ws.routes.append(route) # ==================== # 启动入口 # ==================== def main(): """启动服务器""" port = int(os.getenv("PORT", "8000")) host = os.getenv("HOST", "0.0.0.0") logger.info(f"Starting server on {host}:{port}") uvicorn.run( app\_with\_ws, host=host, port=port, log\_level=LOG\_LEVEL.lower(), ) if \_\_name\_\_ == "\_\_main\_\_": main()

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

118

119

120

121

122

123

124

125

126

127

128

129

130

131

132

133

134

135

136

137

138

139

140

141

142

143

144

145

146

147

148

149

150

151

152

153

154

155

156

157

158

159

160

161

162

163

164

165

166

from \_\_future\_\_ import annotations

import asyncio

import logging

import os

from contextlib import asynccontextmanager

import uvicorn

from fastapi import FastAPI

\# 全局日志配置

LOG\_LEVEL \= os.getenv("LOG\_LEVEL", "DEBUG").upper()

logging.basicConfig(

level\=LOG\_LEVEL,

format\="%(asctime)s %(levelname)-5s \[%(name)s\] %(message)s",

datefmt\="%H:%M:%S",

)

from app import app, set\_matcher\_service

from bus import create\_bus

from bus.direct\_call\_bus import DirectCallBus

from dispatcher.response\_dispatcher import ResponseDispatcher

from gateway.telegram\_gateway import TelegramGateway

from matcher.matcher\_service import MatcherService

from service.catalog\_scheduler import CatalogScheduler

from service.catalog\_sync import CatalogSync

from worker.agent\_worker import AgentWorker

logger \= logging.getLogger(\_\_name\_\_)

\# ====================

\# 组件初始化

\# ====================

\# 全局消息总线

bus \= create\_bus()

\# Agent Worker

worker \= AgentWorker()

\# 响应分派器

dispatcher \= ResponseDispatcher()

\# Gateway 管理器

\_gateways: dict\[str, TelegramGateway\] \= {}

\# ====================

\# 服务生命周期

\# ====================

@asynccontextmanager

async def websocket\_lifespan(app: FastAPI):

"""

服务生命周期管理

启动时:

1\. 初始化 MatcherService（构建索引）

2\. 创建 MessageBus + AgentWorker + ResponseDispatcher

3\. 启动 Telegram Gateway

4\. 启动 Catalog 定时同步调度器

"""

logger.info("=== Starting Server ===")

\# 1. 初始化 MatcherService（构建索引）

matcher\_service \= MatcherService(catalog\_path\="catalog")

set\_matcher\_service(matcher\_service)

logger.info("MatcherService initialized and registered")

\# 2. 组装 Bus + Worker + Dispatcher

if isinstance(bus, DirectCallBus):

bus.bind\_worker(worker)

bus.bind\_dispatcher(dispatcher)

logger.info("DirectCallBus: worker and dispatcher bound")

else:

\# Redis 模式：启动 worker 和 dispatcher 循环

asyncio.create\_task(worker.run\_loop(bus))

asyncio.create\_task(dispatcher.run\_loop(bus))

logger.info("Redis bus: worker and dispatcher loops started")

\# 3. 启动 Telegram Gateway

telegram\_token \= os.getenv("TELEGRAM\_BOT\_TOKEN", "")

if telegram\_token:

telegram\_gw \= TelegramGateway(bot\_token\=telegram\_token, bus\=bus)

\_gateways\["telegram"\] \= telegram\_gw

\# 注册到 Dispatcher

dispatcher.register("telegram", telegram\_gw)

\# 在后台运行 Gateway

async def run\_gateway():

await telegram\_gw.start()

asyncio.create\_task(run\_gateway())

logger.info("TelegramGateway started in background")

else:

logger.warning("TELEGRAM\_BOT\_TOKEN not set, skipping TelegramGateway")

\# 4. 启动 Catalog 定时同步调度器

catalog\_api\_base \= os.getenv("CATALOG\_API\_BASE")

if catalog\_api\_base:

catalog\_sync \= CatalogSync(api\_base\=catalog\_api\_base, catalog\_dir\="catalog")

catalog\_scheduler \= CatalogScheduler(catalog\_sync, matcher\_service)

catalog\_scheduler.start()

logger.info(f"CatalogScheduler started with API: {catalog\_api\_base}")

else:

catalog\_scheduler \= None

logger.info("CATALOG\_API\_BASE not set, skipping CatalogScheduler")

\# 5. 启动 Session 定时清理

from app import session\_manager

async def \_session\_cleanup\_loop():

while True:

await asyncio.sleep(300) \# 每 5 分钟

try:

session\_manager.cleanup\_inactive(max\_age\_minutes\=60)

except Exception as e:

logger.warning("Session cleanup failed: %s", e)

asyncio.create\_task(\_session\_cleanup\_loop())

logger.info("Session cleanup loop started (interval=5min, max\_age=60min)")

yield

\# 清理

logger.info("=== Stopping Server ===")

for gw in \_gateways.values():

await gw.stop()

if catalog\_scheduler:

catalog\_scheduler.shutdown()

\# 创建带生命周期的 FastAPI 应用

app\_with\_ws \= FastAPI(

title\="query-agent: WebSocket + HTTP",

lifespan\=websocket\_lifespan,

)

\# 直接挂载原有路由（不使用 /api 前缀）

for route in app.routes:

app\_with\_ws.routes.append(route)

\# ====================

\# 启动入口

\# ====================

def main():

"""启动服务器"""

port \= int(os.getenv("PORT", "8000"))

host \= os.getenv("HOST", "0.0.0.0")

logger.info(f"Starting server on {host}:{port}")

uvicorn.run(

app\_with\_ws,

host\=host,

port\=port,

log\_level\=LOG\_LEVEL.lower(),

)

if \_\_name\_\_ \== "\_\_main\_\_":

main()