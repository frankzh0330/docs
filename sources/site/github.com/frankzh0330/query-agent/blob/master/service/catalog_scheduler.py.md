# Source: https://github.com/frankzh0330/query-agent/blob/master/service/catalog_scheduler.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# catalog\_scheduler.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/service/catalog_scheduler.py)

Open commit details

History

81 lines (64 loc) · 3.17 KB

## FilesExpand file tree

master

/

# catalog\_scheduler.py

Copy path

Top

## File metadata and controls

- Code

- Blame

81 lines (64 loc) · 3.17 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/catalog_scheduler.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

""" 目录定时同步调度器 - 使用 APScheduler 实现定时同步和热更新 """ from \_\_future\_\_ import annotations import logging from apscheduler.schedulers.asyncio import AsyncIOScheduler from matcher.catalog\_loader import load\_catalog from matcher.dimension\_matcher import build\_dimension\_matcher\_from\_catalog from matcher.event\_matcher import build\_event\_matcher\_from\_catalog from matcher.metric\_matcher import build\_metric\_matcher\_from\_catalog from service.catalog\_sync import CatalogSync logger = logging.getLogger(\_\_name\_\_) class CatalogScheduler: """目录定时同步调度器""" def \_\_init\_\_(self, sync: CatalogSync, matcher\_service): self.sync = sync self.matcher\_service = matcher\_service self.scheduler = AsyncIOScheduler() async def incremental\_sync(self): """增量同步并热更新索引""" logger.info("Starting incremental catalog sync...") try: event\_count = await self.sync.incremental\_sync\_events() dim\_count = await self.sync.incremental\_sync\_dimensions() logger.info(f"Incremental sync completed: {event\_count} events, {dim\_count} dimensions") # 热更新 matcher await self.\_reload\_matchers() logger.info("Matcher indexes hot-reloaded successfully") except Exception as e: logger.error(f"Incremental sync failed: {e}", exc\_info=True) async def full\_sync(self): """全量同步并热更新索引""" logger.info("Starting full catalog sync...") try: event\_count = await self.sync.full\_sync\_events() dim\_count = await self.sync.full\_sync\_dimensions() logger.info(f"Full sync completed: {event\_count} events, {dim\_count} dimensions") # 热更新 matcher await self.\_reload\_matchers() logger.info("Matcher indexes hot-reloaded successfully") except Exception as e: logger.error(f"Full sync failed: {e}", exc\_info=True) async def \_reload\_matchers(self): """热更新：重新加载 catalog 并重建 matcher 索引""" new\_catalog = load\_catalog(self.sync.catalog\_dir) # 重建 matcher self.matcher\_service.catalog = new\_catalog self.matcher\_service.event\_matcher = build\_event\_matcher\_from\_catalog(new\_catalog.events) self.matcher\_service.metric\_matcher = build\_metric\_matcher\_from\_catalog(new\_catalog.metrics) self.matcher\_service.dimension\_matcher = build\_dimension\_matcher\_from\_catalog(new\_catalog.dimensions) def start(self): """启动定时任务""" # 每 1 小时增量同步 self.scheduler.add\_job(self.incremental\_sync, "interval", hours=1, id="incremental\_sync") logger.info("Scheduled incremental sync: every 1 hour") # 每 6 小时全量同步 self.scheduler.add\_job(self.full\_sync, "interval", hours=6, id="full\_sync") logger.info("Scheduled full sync: every 6 hours") self.scheduler.start() logger.info("CatalogScheduler started") def shutdown(self): """关闭调度器""" self.scheduler.shutdown(wait=False) logger.info("CatalogScheduler shutdown")

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

"""

目录定时同步调度器 - 使用 APScheduler 实现定时同步和热更新

"""

from \_\_future\_\_ import annotations

import logging

from apscheduler.schedulers.asyncio import AsyncIOScheduler

from matcher.catalog\_loader import load\_catalog

from matcher.dimension\_matcher import build\_dimension\_matcher\_from\_catalog

from matcher.event\_matcher import build\_event\_matcher\_from\_catalog

from matcher.metric\_matcher import build\_metric\_matcher\_from\_catalog

from service.catalog\_sync import CatalogSync

logger \= logging.getLogger(\_\_name\_\_)

class CatalogScheduler:

"""目录定时同步调度器"""

def \_\_init\_\_(self, sync: CatalogSync, matcher\_service):

self.sync \= sync

self.matcher\_service \= matcher\_service

self.scheduler \= AsyncIOScheduler()

async def incremental\_sync(self):

"""增量同步并热更新索引"""

logger.info("Starting incremental catalog sync...")

try:

event\_count \= await self.sync.incremental\_sync\_events()

dim\_count \= await self.sync.incremental\_sync\_dimensions()

logger.info(f"Incremental sync completed: {event\_count} events, {dim\_count} dimensions")

\# 热更新 matcher

await self.\_reload\_matchers()

logger.info("Matcher indexes hot-reloaded successfully")

except Exception as e:

logger.error(f"Incremental sync failed: {e}", exc\_info\=True)

async def full\_sync(self):

"""全量同步并热更新索引"""

logger.info("Starting full catalog sync...")

try:

event\_count \= await self.sync.full\_sync\_events()

dim\_count \= await self.sync.full\_sync\_dimensions()

logger.info(f"Full sync completed: {event\_count} events, {dim\_count} dimensions")

\# 热更新 matcher

await self.\_reload\_matchers()

logger.info("Matcher indexes hot-reloaded successfully")

except Exception as e:

logger.error(f"Full sync failed: {e}", exc\_info\=True)

async def \_reload\_matchers(self):

"""热更新：重新加载 catalog 并重建 matcher 索引"""

new\_catalog \= load\_catalog(self.sync.catalog\_dir)

\# 重建 matcher

self.matcher\_service.catalog \= new\_catalog

self.matcher\_service.event\_matcher \= build\_event\_matcher\_from\_catalog(new\_catalog.events)

self.matcher\_service.metric\_matcher \= build\_metric\_matcher\_from\_catalog(new\_catalog.metrics)

self.matcher\_service.dimension\_matcher \= build\_dimension\_matcher\_from\_catalog(new\_catalog.dimensions)

def start(self):

"""启动定时任务"""

\# 每 1 小时增量同步

self.scheduler.add\_job(self.incremental\_sync, "interval", hours\=1, id\="incremental\_sync")

logger.info("Scheduled incremental sync: every 1 hour")

\# 每 6 小时全量同步

self.scheduler.add\_job(self.full\_sync, "interval", hours\=6, id\="full\_sync")

logger.info("Scheduled full sync: every 6 hours")

self.scheduler.start()

logger.info("CatalogScheduler started")

def shutdown(self):

"""关闭调度器"""

self.scheduler.shutdown(wait\=False)

logger.info("CatalogScheduler shutdown")