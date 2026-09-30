# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_app_endpoints.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_app\_endpoints.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add multi-turn follow-up query pipeline and extract QueryOrches…](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4)

Open commit details

May 1, 2026

[3560e95](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4) · May 1, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_app_endpoints.py)

Open commit details

History

680 lines (559 loc) · 28.4 KB

## FilesExpand file tree

master

/

# test\_app\_endpoints.py

Copy path

Top

## File metadata and controls

- Code

- Blame

680 lines (559 loc) · 28.4 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_app_endpoints.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""app.py HTTP 端点测试（替代 test\_nl2dsl\_api.py）""" from unittest import mock import pytest from fastapi.testclient import TestClient from matcher.matcher\_service import ResolvedResult from service.llm\_extractions import Extraction, ExtractionsJson from service.session\_models import QueryState # ==================== Fixtures ==================== @pytest.fixture def client(tmp\_path): """创建隔离的 TestClient，使用 tmp\_path 避免 state 泄漏""" import app as app\_module from memory.storage.memory\_file import TaskStorage from memory.user\_preference\_store import UserPreferenceStore from service.session\_manager import SessionManager from service.task\_manager import TaskManager # 替换全局实例为隔离版本 app\_module.session\_manager = SessionManager(data\_path=str(tmp\_path / "data")) app\_module.task\_manager = TaskManager( storage=TaskStorage(data\_path=str(tmp\_path / "data" / "tasks")) ) app\_module.user\_preference\_store = UserPreferenceStore( data\_path=str(tmp\_path / "data" / "user\_preferences") ) # 同步更新 orchestrator 的引用 app\_module.orchestrator.session = app\_module.session\_manager app\_module.orchestrator.task = app\_module.task\_manager app\_module.orchestrator.preferences = app\_module.user\_preference\_store # 初始化 matcher\_service（用 mock 避免加载真实 catalog） mock\_service = mock.MagicMock() mock\_service.catalog = mock.MagicMock() app\_module.\_matcher\_service = mock\_service with TestClient(app\_module.app) as c: yield c # 清理 app\_module.\_matcher\_service = None def \_mock\_high\_confidence\_resolver(): """高置信度 resolver（不需要确认）""" svc = mock.MagicMock() svc.resolve\_with\_candidates.side\_effect = lambda mtype, extractions, default: ResolvedResult( value="app\_launch" if mtype.value == "event" else ("pv" if mtype.value == "metric" else "country"), score=95.0, method="exact", candidates=\[\], needs\_confirmation=False, ) svc.resolve\_time.return\_value = (7, {"method": "default", "n": 7}) return svc def \_mock\_low\_confidence\_resolver(): """低置信度 resolver（需要确认）""" svc = mock.MagicMock() call\_count = {"n": 0} def \_resolve(mtype, extractions, default): call\_count\["n"\] += 1 if mtype.value == "event": return ResolvedResult( value="purchase\_success", score=55.0, method="fuzzy\_low\_confidence", candidates=\[ {"value": "purchase\_success", "score": 55.0}, {"value": "payment\_submit", "score": 48.0}, \], needs\_confirmation=True, ) return ResolvedResult( value="pv" if mtype.value == "metric" else "country", score=95.0, method="exact", candidates=\[\], needs\_confirmation=False, ) svc.resolve\_with\_candidates.side\_effect = \_resolve svc.resolve\_time.return\_value = (7, {"method": "default", "n": 7}) return svc def \_mock\_low\_score\_resolver(): """极低分 resolver（fallback 到 default）""" svc = mock.MagicMock() svc.resolve\_with\_candidates.side\_effect = lambda mtype, extractions, default: ResolvedResult( value=default, score=20.0, method="score\_too\_low", candidates=\[\], needs\_confirmation=False, ) svc.resolve\_time.return\_value = (7, {"method": "default", "n": 7}) return svc # ==================== Tests: Happy Path ==================== class TestNL2DSLHappyPath: @staticmethod def \_seed\_previous\_state(): import app as app\_module ctx = app\_module.session\_manager.create\_or\_get(None, "user\_1", 55) prev\_qs = QueryState( project\_id=55, event="app\_launch", metric="pv", time\_range={"type": "last\_n\_days", "n": 7}, region\_filter=\["ROW"\], group\_by=\["country"\], filters=\[\], turn\_type="new\_query", ) app\_module.session\_manager.update\_query\_state(ctx.session\_id, prev\_qs) return ctx, prev\_qs def test\_nl2dsl\_success(self, client): extraction = ExtractionsJson( metric\_extractions=\[Extraction(text="PV")\], event\_extractions=\[Extraction(text="app\_launch")\], region\_filter=\["ROW"\], group\_by\_extractions=\[Extraction(text="country")\], ) mock\_service = \_mock\_high\_confidence\_resolver() with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp = client.post("/nl2dsl", json={ "text": "查询近7天 app\_launch 的PV", "project\_id": 55, }) assert resp.status\_code == 200 body = resp.json() assert body\["status"\] == "success" assert "semantic" in body assert "exec\_dsl" in body assert body\["session\_id"\] def test\_nl2dsl\_returns\_session\_id(self, client): extraction = ExtractionsJson( event\_extractions=\[Extraction(text="app\_launch")\], region\_filter=\["ROW"\], ) mock\_service = \_mock\_high\_confidence\_resolver() with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp = client.post("/nl2dsl", json={"text": "test", "project\_id": 55}) assert resp.json()\["session\_id"\] is not None def test\_nl2dsl\_uses\_existing\_session(self, client): import app as app\_module ctx = app\_module.session\_manager.create\_or\_get(None, "user\_1", 55) sid = ctx.session\_id extraction = ExtractionsJson( event\_extractions=\[Extraction(text="app\_launch")\], region\_filter=\["ROW"\], ) mock\_service = \_mock\_high\_confidence\_resolver() with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp = client.post("/nl2dsl", json={ "text": "test", "project\_id": 55, "session\_id": sid, }) assert resp.json()\["session\_id"\] == sid def test\_nl2dsl\_followup\_inherits\_previous\_state(self, client): ctx, \_ = self.\_seed\_previous\_state() extraction = ExtractionsJson( time\_extractions=\[Extraction(text="昨天")\], event\_extractions=\[\], region\_filter=\["ROW"\], ) mock\_service = mock.MagicMock() mock\_service.resolve\_time.return\_value = (1, {"method": "matched", "n": 1}) with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp = client.post("/nl2dsl", json={ "text": "昨天", "project\_id": 55, "session\_id": ctx.session\_id, }) body = resp.json() assert body\["status"\] == "success" assert body\["semantic"\]\["event"\]\["event\_name"\] == "app\_launch" assert body\["semantic"\]\["metric"\]\["metric\_id"\] == "pv" assert body\["semantic"\]\["time\_range"\]\["n"\] == 1 assert body\["explain"\]\["turn\_explain"\]\["mode"\] == "followup\_patch" assert body\["explain"\]\["turn\_explain"\]\["decision"\]\["reason"\] == "time\_only\_term" assert body\["explain"\]\["turn\_explain"\]\["applied\_patch\_fields"\] == \["time\_range"\] def test\_nl2dsl\_followup\_changes\_metric\_only(self, client): ctx, \_ = self.\_seed\_previous\_state() extraction = ExtractionsJson( metric\_extractions=\[Extraction(text="UV")\], event\_extractions=\[\], region\_filter=\["ROW"\], ) mock\_service = mock.MagicMock() def \_resolve(mtype, extractions, default): if mtype.value == "metric": return ResolvedResult( value="uv", score=96.0, method="exact", candidates=\[\], needs\_confirmation=False, ) raise AssertionError(f"unexpected matcher type: {mtype}") mock\_service.resolve\_with\_candidates.side\_effect = \_resolve with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp = client.post("/nl2dsl", json={ "text": "改成UV", "project\_id": 55, "session\_id": ctx.session\_id, }) body = resp.json() assert body\["status"\] == "success" assert body\["semantic"\]\["metric"\]\["metric\_id"\] == "uv" assert body\["semantic"\]\["event"\]\["event\_name"\] == "app\_launch" assert body\["explain"\]\["turn\_explain"\]\["explicit\_fields"\] == \["metric"\] assert body\["explain"\]\["turn\_explain"\]\["decision"\]\["matched\_signals"\] assert body\["explain"\]\["turn\_explain"\]\["state\_snapshot"\]\["metric"\] == "uv" def test\_nl2dsl\_followup\_changes\_region\_only(self, client): ctx, \_ = self.\_seed\_previous\_state() extraction = ExtractionsJson( event\_extractions=\[\], region\_filter=\["USTTP"\], ) mock\_service = mock.MagicMock() with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp = client.post("/nl2dsl", json={ "text": "那美国呢", "project\_id": 55, "session\_id": ctx.session\_id, }) body = resp.json() assert body\["status"\] == "success" assert body\["semantic"\]\["region\_filter"\] == \["USTTP"\] assert body\["semantic"\]\["metric"\]\["metric\_id"\] == "pv" assert body\["semantic"\]\["event"\]\["event\_name"\] == "app\_launch" assert body\["explain"\]\["turn\_explain"\]\["explicit\_fields"\] == \["region\_filter"\] assert "region\_token" in body\["explain"\]\["turn\_explain"\]\["decision"\]\["matched\_signals"\] def test\_nl2dsl\_followup\_changes\_group\_by\_only(self, client): ctx, \_ = self.\_seed\_previous\_state() extraction = ExtractionsJson( event\_extractions=\[\], region\_filter=\["ROW"\], group\_by\_extractions=\[Extraction(text="渠道")\], ) mock\_service = mock.MagicMock() def \_resolve(mtype, extractions, default): if mtype.value == "dimension": return ResolvedResult( value="channel", score=94.0, method="exact", candidates=\[\], needs\_confirmation=False, ) raise AssertionError(f"unexpected matcher type: {mtype}") mock\_service.resolve\_with\_candidates.side\_effect = \_resolve with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp = client.post("/nl2dsl", json={ "text": "再按渠道拆一下", "project\_id": 55, "session\_id": ctx.session\_id, }) body = resp.json() assert body\["status"\] == "success" assert body\["semantic"\]\["group\_by"\] == \[{"dimension\_id": "channel"}\] assert body\["semantic"\]\["metric"\]\["metric\_id"\] == "pv" assert body\["semantic"\]\["event"\]\["event\_name"\] == "app\_launch" assert body\["explain"\]\["turn\_explain"\]\["explicit\_fields"\] == \["group\_by"\] assert body\["explain"\]\["turn\_explain"\]\["applied\_patch"\]\["group\_by"\] == \["channel"\] def test\_nl2dsl\_applies\_user\_preference\_rerank\_after\_recall(self, client): import app as app\_module for \_ in range(6): app\_module.user\_preference\_store.record\_selection( 55, "user\_pref", event="payment\_submit" ) extraction = ExtractionsJson( event\_extractions=\[Extraction(text="支付")\], region\_filter=\["ROW"\], ) mock\_service = mock.MagicMock() def \_resolve(mtype, extractions, default): if mtype.value == "event": return ResolvedResult( value="purchase\_success", score=55.0, method="fuzzy\_low\_confidence", candidates=\[ {"value": "purchase\_success", "score": 55.0}, {"value": "payment\_submit", "score": 54.0}, \], needs\_confirmation=True, ) return ResolvedResult( value="pv" if mtype.value == "metric" else "country", score=95.0, method="exact", candidates=\[\], needs\_confirmation=False, ) mock\_service.resolve\_with\_candidates.side\_effect = \_resolve mock\_service.resolve\_time.return\_value = (7, {"method": "default", "n": 7}) with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): resp = client.post("/nl2dsl", json={ "text": "支付情况", "project\_id": 55, "user\_id": "user\_pref", }) body = resp.json() assert body\["status"\] == "needs\_confirmation" assert body\["candidates"\]\["event"\]\[0\]\["value"\] == "payment\_submit" assert body\["explain"\]\["resolver\_explain"\]\["event"\]\["user\_preference\_bias"\]\["applied"\] is True # ==================== Tests: Early Exit ==================== class TestNL2DSLEarlyExit: def test\_nl2dsl\_early\_exit\_no\_event(self, client): extraction = ExtractionsJson( metric\_extractions=\[Extraction(text="PV")\], event\_extractions=\[\], region\_filter=\["ROW"\], ) with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): resp = client.post("/nl2dsl", json={ "text": "PV数据", "project\_id": 55, }) body = resp.json() assert body\["status"\] == "early\_exit" assert body\["message"\] assert "event" in body\["message"\] def test\_nl2dsl\_early\_exit\_records\_message(self, client): import app as app\_module extraction = ExtractionsJson( event\_extractions=\[\], region\_filter=\["ROW"\], ) with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): resp = client.post("/nl2dsl", json={ "text": "some query", "project\_id": 55, }) sid = resp.json()\["session\_id"\] ctx = app\_module.session\_manager.get\_session(sid) assert ctx is not None assert any(m.content == "some query" for m in ctx.messages) # ==================== Tests: Confirmation Flow ==================== class TestNL2DSLConfirmation: @staticmethod def \_seed\_previous\_state(): import app as app\_module ctx = app\_module.session\_manager.create\_or\_get(None, "user\_1", 55) prev\_qs = QueryState( project\_id=55, event="app\_launch", metric="pv", time\_range={"type": "last\_n\_days", "n": 7}, region\_filter=\["ROW"\], group\_by=\["country"\], filters=\[\], turn\_type="new\_query", ) app\_module.session\_manager.update\_query\_state(ctx.session\_id, prev\_qs) return ctx, prev\_qs def test\_nl2dsl\_needs\_confirmation(self, client): extraction = ExtractionsJson( event\_extractions=\[Extraction(text="purchase")\], region\_filter=\["ROW"\], ) mock\_service = \_mock\_low\_confidence\_resolver() with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): resp = client.post("/nl2dsl", json={ "text": "purchase的情况", "project\_id": 55, }) body = resp.json() assert body\["status"\] == "needs\_confirmation" assert body\["task\_id"\] is not None assert body\["candidates"\] is not None assert "event" in body\["candidates"\] def test\_nl2dsl\_confirmation\_then\_reply(self, client): import app as app\_module extraction = ExtractionsJson( event\_extractions=\[Extraction(text="purchase")\], region\_filter=\["ROW"\], ) mock\_service = \_mock\_low\_confidence\_resolver() # 第一次请求：触发确认 with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): resp1 = client.post("/nl2dsl", json={ "text": "purchase的情况", "project\_id": 55, }) body1 = resp1.json() sid = body1\["session\_id"\] task\_id = body1\["task\_id"\] assert body1\["status"\] == "needs\_confirmation" # 第二次请求：用户回复确认 with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp2 = client.post("/nl2dsl", json={ "text": "1", "project\_id": 55, "session\_id": sid, }) body2 = resp2.json() assert body2\["status"\] == "success" assert "confirmed" in body2.get("message", "") or body2\["status"\] == "success" assert body2\["explain"\]\["turn\_explain"\]\["mode"\] == "confirmation" assert body2\["explain"\]\["turn\_explain"\]\["field\_sources"\]\["event"\] == "confirmed" assert body2\["explain"\]\["turn\_explain"\]\["confirmed\_fields"\]\["event"\] == "purchase\_success" def test\_nl2dsl\_followup\_confirmation\_then\_reply(self, client): ctx, \_ = self.\_seed\_previous\_state() extraction = ExtractionsJson( event\_extractions=\[Extraction(text="purchase")\], region\_filter=\["ROW"\], ) mock\_service = \_mock\_low\_confidence\_resolver() with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): resp1 = client.post("/nl2dsl", json={ "text": "对比purchase", "project\_id": 55, "session\_id": ctx.session\_id, }) body1 = resp1.json() assert body1\["status"\] == "needs\_confirmation" assert body1\["explain"\]\["turn\_explain"\]\["mode"\] == "followup\_patch" with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp2 = client.post("/nl2dsl", json={ "text": "1", "project\_id": 55, "session\_id": ctx.session\_id, }) body2 = resp2.json() assert body2\["status"\] == "success" assert body2\["semantic"\]\["event"\]\["event\_name"\] == "purchase\_success" assert body2\["semantic"\]\["metric"\]\["metric\_id"\] == "pv" assert body2\["semantic"\]\["group\_by"\] == \[{"dimension\_id": "country"}\] assert body2\["explain"\]\["turn\_explain"\]\["mode"\] == "confirmation" assert body2\["explain"\]\["turn\_explain"\]\["field\_sources"\]\["event"\] == "confirmed" assert "metric" in body2\["explain"\]\["turn\_explain"\]\["inherited\_fields"\] assert body2\["explain"\]\["turn\_explain"\]\["confirmed\_fields"\]\["event"\] == "purchase\_success" def test\_nl2dsl\_confirmation\_after\_session\_restore(self, client): import app as app\_module from service.session\_manager import SessionManager extraction = ExtractionsJson( event\_extractions=\[Extraction(text="purchase")\], region\_filter=\["ROW"\], ) mock\_service = \_mock\_low\_confidence\_resolver() # 第一次请求：触发确认并落盘 with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): resp1 = client.post("/nl2dsl", json={ "text": "purchase的情况", "project\_id": 55, }) body1 = resp1.json() sid = body1\["session\_id"\] assert body1\["status"\] == "needs\_confirmation" # 模拟服务重启：清空内存 session 和 task，再从同一路径恢复 original\_session\_manager = app\_module.session\_manager original\_task\_manager = app\_module.task\_manager app\_module.session\_manager = SessionManager( data\_path=str(original\_session\_manager.storage.data\_path.parent) ) app\_module.task\_manager = type(original\_task\_manager)( ttl\_minutes=30, storage=original\_task\_manager.storage, ) try: with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp2 = client.post("/nl2dsl", json={ "text": "1", "project\_id": 55, "session\_id": sid, }) finally: # 恢复 fixture 中的隔离实例，避免影响后续测试 app\_module.session\_manager = original\_session\_manager app\_module.task\_manager = original\_task\_manager body2 = resp2.json() assert body2\["status"\] == "success" # ==================== Tests: Fallback ==================== class TestNL2DSLFallback: def test\_nl2dsl\_fallback\_on\_low\_score(self, client): extraction = ExtractionsJson( event\_extractions=\[Extraction(text="unknown\_thing")\], region\_filter=\["ROW"\], ) mock\_service = \_mock\_low\_score\_resolver() with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable=mock.AsyncMock, return\_value=extraction): with mock.patch("app.get\_matcher\_service", return\_value=mock\_service): with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value={"content": {"queries": \[\]}}): with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"): resp = client.post("/nl2dsl", json={ "text": "unknown query", "project\_id": 55, }) body = resp.json() assert body\["status"\] == "success" # 使用了 default 值（app\_launch / pv / country） assert body\["semantic"\]\["event"\]\["event\_name"\] == "app\_launch" assert body\["semantic"\]\["metric"\]\["metric\_id"\] == "pv" # ==================== Tests: Bearer Query ==================== class TestBearerQuery: def test\_query\_bearer\_success(self, client): with mock.patch("app.execute\_query", new\_callable=mock.AsyncMock, return\_value={"data": "mock"}): resp = client.post("/query/bearer", json={"exec\_dsl": {"content": {}}}) body = resp.json() assert body\["success"\] is True assert body\["result"\] == {"data": "mock"} def test\_query\_bearer\_failure(self, client): with mock.patch("app.execute\_query", new\_callable=mock.AsyncMock, side\_effect=ValueError("API error")): resp = client.post("/query/bearer", json={"exec\_dsl": {"content": {}}}) body = resp.json() assert body\["success"\] is False assert "API error" in body\["error"\] # ==================== Tests: Session Endpoints ==================== class TestSessionEndpoints: def test\_list\_sessions\_empty(self, client): resp = client.get("/sessions") assert resp.status\_code == 200 assert resp.json()\["count"\] == 0 def test\_list\_sessions\_with\_data(self, client): import app as app\_module app\_module.session\_manager.create\_or\_get(None, "user\_1", 55) resp = client.get("/sessions") assert resp.json()\["count"\] == 1 def test\_get\_session\_found(self, client): import app as app\_module ctx = app\_module.session\_manager.create\_or\_get(None, "user\_1", 55) resp = client.get(f"/sessions/{ctx.session\_id}") assert resp.status\_code == 200 assert resp.json()\["session\_id"\] == ctx.session\_id def test\_get\_session\_not\_found(self, client): resp = client.get("/sessions/nonexistent") assert resp.status\_code == 404 def test\_delete\_session\_found(self, client): import app as app\_module ctx = app\_module.session\_manager.create\_or\_get(None, "user\_1", 55) resp = client.delete(f"/sessions/{ctx.session\_id}") assert resp.status\_code == 200 resp2 = client.get(f"/sessions/{ctx.session\_id}") assert resp2.status\_code == 404 def test\_delete\_session\_not\_found(self, client): resp = client.delete("/sessions/nonexistent") assert resp.status\_code == 404

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

607

608

609

610

611

612

613

614

615

616

617

618

619

620

621

622

623

624

625

626

627

628

629

630

631

632

633

634

635

636

637

638

639

640

641

642

643

644

645

646

647

648

649

650

651

652

653

654

655

656

657

658

659

660

661

662

663

664

665

666

667

668

669

670

671

672

673

674

675

676

677

678

679

680

"""app.py HTTP 端点测试（替代 test\_nl2dsl\_api.py）"""

from unittest import mock

import pytest

from fastapi.testclient import TestClient

from matcher.matcher\_service import ResolvedResult

from service.llm\_extractions import Extraction, ExtractionsJson

from service.session\_models import QueryState

\# ==================== Fixtures ====================

@pytest.fixture

def client(tmp\_path):

"""创建隔离的 TestClient，使用 tmp\_path 避免 state 泄漏"""

import app as app\_module

from memory.storage.memory\_file import TaskStorage

from memory.user\_preference\_store import UserPreferenceStore

from service.session\_manager import SessionManager

from service.task\_manager import TaskManager

\# 替换全局实例为隔离版本

app\_module.session\_manager \= SessionManager(data\_path\=str(tmp\_path / "data"))

app\_module.task\_manager \= TaskManager(

storage\=TaskStorage(data\_path\=str(tmp\_path / "data" / "tasks"))

)

app\_module.user\_preference\_store \= UserPreferenceStore(

data\_path\=str(tmp\_path / "data" / "user\_preferences")

)

\# 同步更新 orchestrator 的引用

app\_module.orchestrator.session \= app\_module.session\_manager

app\_module.orchestrator.task \= app\_module.task\_manager

app\_module.orchestrator.preferences \= app\_module.user\_preference\_store

\# 初始化 matcher\_service（用 mock 避免加载真实 catalog）

mock\_service \= mock.MagicMock()

mock\_service.catalog \= mock.MagicMock()

app\_module.\_matcher\_service \= mock\_service

with TestClient(app\_module.app) as c:

yield c

\# 清理

app\_module.\_matcher\_service \= None

def \_mock\_high\_confidence\_resolver():

"""高置信度 resolver（不需要确认）"""

svc \= mock.MagicMock()

svc.resolve\_with\_candidates.side\_effect \= lambda mtype, extractions, default: ResolvedResult(

value\="app\_launch" if mtype.value \== "event" else ("pv" if mtype.value \== "metric" else "country"),

score\=95.0,

method\="exact",

candidates\=\[\],

needs\_confirmation\=False,

)

svc.resolve\_time.return\_value \= (7, {"method": "default", "n": 7})

return svc

def \_mock\_low\_confidence\_resolver():

"""低置信度 resolver（需要确认）"""

svc \= mock.MagicMock()

call\_count \= {"n": 0}

def \_resolve(mtype, extractions, default):

call\_count\["n"\] += 1

if mtype.value \== "event":

return ResolvedResult(

value\="purchase\_success",

score\=55.0,

method\="fuzzy\_low\_confidence",

candidates\=\[

{"value": "purchase\_success", "score": 55.0},

{"value": "payment\_submit", "score": 48.0},

\],

needs\_confirmation\=True,

)

return ResolvedResult(

value\="pv" if mtype.value \== "metric" else "country",

score\=95.0,

method\="exact",

candidates\=\[\],

needs\_confirmation\=False,

)

svc.resolve\_with\_candidates.side\_effect \= \_resolve

svc.resolve\_time.return\_value \= (7, {"method": "default", "n": 7})

return svc

def \_mock\_low\_score\_resolver():

"""极低分 resolver（fallback 到 default）"""

svc \= mock.MagicMock()

svc.resolve\_with\_candidates.side\_effect \= lambda mtype, extractions, default: ResolvedResult(

value\=default,

score\=20.0,

method\="score\_too\_low",

candidates\=\[\],

needs\_confirmation\=False,

)

svc.resolve\_time.return\_value \= (7, {"method": "default", "n": 7})

return svc

\# ==================== Tests: Happy Path ====================

class TestNL2DSLHappyPath:

@staticmethod

def \_seed\_previous\_state():

import app as app\_module

ctx \= app\_module.session\_manager.create\_or\_get(None, "user\_1", 55)

prev\_qs \= QueryState(

project\_id\=55,

event\="app\_launch",

metric\="pv",

time\_range\={"type": "last\_n\_days", "n": 7},

region\_filter\=\["ROW"\],

group\_by\=\["country"\],

filters\=\[\],

turn\_type\="new\_query",

)

app\_module.session\_manager.update\_query\_state(ctx.session\_id, prev\_qs)

return ctx, prev\_qs

def test\_nl2dsl\_success(self, client):

extraction \= ExtractionsJson(

metric\_extractions\=\[Extraction(text\="PV")\],

event\_extractions\=\[Extraction(text\="app\_launch")\],

region\_filter\=\["ROW"\],

group\_by\_extractions\=\[Extraction(text\="country")\],

)

mock\_service \= \_mock\_high\_confidence\_resolver()

with mock.patch("service.query\_orchestrator.extract\_llm\_async", new\_callable\=mock.AsyncMock, return\_value\=extraction):

with mock.patch("app.get\_matcher\_service", return\_value\=mock\_service):

with mock.patch("service.query\_orchestrator.render\_exec\_dsl", return\_value\={"content": {"queries": \[\]}}):

with mock.patch("service.query\_orchestrator.validate\_region"), mock.patch("service.query\_orchestrator.validate\_region\_consistency"):

resp \= client.post("/nl2dsl", json\={

"text": "查询近7天 app\_launch 的PV",

"project\_id": 55,

})

assert resp.status\_code \== 200

body \= resp.json()

assert body\["status"\] \== "success"

assert "semantic" in body

assert "exec\_dsl" in body

assert body\["session\_id"\]

def test\_nl2dsl\_returns\_session\_id(self, client):

extraction \= ExtractionsJson(

resp \= client.post("/nl2dsl", json\={

"text": "unknown query",

"project\_id": 55,

})

body \= resp.json()

assert body\["status"\] \== "success"

\# 使用了 default 值（app\_launch / pv / country）

assert body\["semantic"\]\["event"\]\["event\_name"\] \== "app\_launch"

assert body\["semantic"\]\["metric"\]\["metric\_id"\] \== "pv"

\# ==================== Tests: Bearer Query ====================

class TestBearerQuery:

def test\_query\_bearer\_success(self, client):

with mock.patch("app.execute\_query", new\_callable\=mock.AsyncMock, return\_value\={"data": "mock"}):

resp \= client.post("/query/bearer", json\={"exec\_dsl": {"content": {}}})

body \= resp.json()

assert body\["success"\] is True

assert body\["result"\] \== {"data": "mock"}

def test\_query\_bearer\_failure(self, client):

with mock.patch("app.execute\_query", new\_callable\=mock.AsyncMock, side\_effect\=ValueError("API error")):

resp \= client.post("/query/bearer", json\={"exec\_dsl": {"content": {}}})

body \= resp.json()

assert body\["success"\] is False

assert "API error" in body\["error"\]

\# ==================== Tests: Session Endpoints ====================

class TestSessionEndpoints:

def test\_list\_sessions\_empty(self, client):

resp \= client.get("/sessions")

assert resp.status\_code \== 200

assert resp.json()\["count"\] \== 0

def test\_list\_sessions\_with\_data(self, client):

import app as app\_module

app\_module.session\_manager.create\_or\_get(None, "user\_1", 55)

resp \= client.get("/sessions")

assert resp.json()\["count"\] \== 1

def test\_get\_session\_found(self, client):

import app as app\_module

ctx \= app\_module.session\_manager.create\_or\_get(None, "user\_1", 55)

resp \= client.get(f"/sessions/{ctx.session\_id}")

assert resp.status\_code \== 200

assert resp.json()\["session\_id"\] \== ctx.session\_id

def test\_get\_session\_not\_found(self, client):

resp \= client.get("/sessions/nonexistent")

assert resp.status\_code \== 404

def test\_delete\_session\_found(self, client):

import app as app\_module

ctx \= app\_module.session\_manager.create\_or\_get(None, "user\_1", 55)

resp \= client.delete(f"/sessions/{ctx.session\_id}")

assert resp.status\_code \== 200

resp2 \= client.get(f"/sessions/{ctx.session\_id}")

assert resp2.status\_code \== 404

def test\_delete\_session\_not\_found(self, client):

resp \= client.delete("/sessions/nonexistent")

assert resp.status\_code \== 404