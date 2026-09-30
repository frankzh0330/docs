# Source: https://github.com/frankzh0330/query-agent/blob/master/service/query_orchestrator.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# query\_orchestrator.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/service/query_orchestrator.py)

Open commit details

History

645 lines (542 loc) · 28 KB

## FilesExpand file tree

master

/

# query\_orchestrator.py

Copy path

Top

## File metadata and controls

- Code

- Blame

645 lines (542 loc) · 28 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/service/query_orchestrator.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""查询编排器 — 从 app.py 提取的核心业务逻辑 三条处理路径： 1. new\_query: 完整的新查询（LLM 提取 → Matcher → DSL） 2. followup\_patch: 基于上轮状态的修改（检测 → 补丁 → 合并 → DSL） 3. confirmation: 用户确认低置信度候选 消除重复： - \_resolve\_field: 统一 resolve + user preference bias - \_build\_semantic: 统一 SemanticDSL 构建 - \_format\_candidates\_message: 统一候选消息格式化 """ from \_\_future\_\_ import annotations import asyncio import json import logging import time from dataclasses import dataclass from typing import Any, Dict, Optional from common.types import MatcherType from dsl.renderer import render\_exec\_dsl from dsl.semantic\_models import Event, GroupBy, Metric, SemanticDSL, TimeRange from dsl.validators import validate\_region, validate\_region\_consistency from memory.memory\_writer import MemoryWriter from memory.user\_preference\_store import UserPreferenceStore from service.followup\_resolver import detect\_followup from service.llm\_extractions import extract\_llm\_async from service.query\_state\_merger import merge\_query\_state from service.session\_manager import SessionManager from service.session\_models import QueryState from service.task\_manager import TaskManager logger = logging.getLogger(\_\_name\_\_) # ==================== 常量 ==================== DEFAULT\_METRIC = "pv" DEFAULT\_EVENT = "app\_launch" DEFAULT\_DIMENSION = "country" DEFAULT\_N\_DAYS = 7 \_REGION\_TOKENS = ("美国", "德国", "欧洲", "新加坡", "加州", "纽约", "ROW") # ==================== 异常 ==================== class FollowupConfirmationNeeded(Exception): def \_\_init\_\_(self, field\_name: str, resolved\_result): self.field\_name = field\_name self.resolved\_result = resolved\_result super().\_\_init\_\_(field\_name) # ==================== 数据类 ==================== @dataclass class FollowupPatchResult: patch: dict\[str, Any\] resolver\_explain: Dict\[str, Any\] patch\_hints: Dict\[str, Any\] confirmation\_needed: Optional\[tuple\[str, Any\]\] = None # ==================== 编排器 ==================== class QueryOrchestrator: """查询编排器：封装三条处理路径的业务逻辑""" def \_\_init\_\_( self, session\_manager: SessionManager, task\_manager: TaskManager, memory\_writer: MemoryWriter, user\_preference\_store: UserPreferenceStore, ): self.session = session\_manager self.task = task\_manager self.memory = memory\_writer self.preferences = user\_preference\_store # ==================== 主入口 ==================== async def process(self, req, service, catalog, notify\_fn=None) -> dict: """主入口：路由到三条处理路径 Args: req: NL2DSLRequest service: MatcherService catalog: catalog 数据 notify\_fn: async callable(chat\_id, message) for Telegram notifications Returns: NL2DSLResponse dict """ ctx = self.session.create\_or\_get(req.session\_id, req.user\_id, req.project\_id) prev\_qs = ctx.last\_query\_state # 1. 确认流拦截 if ctx.pending\_task\_id: task = self.task.get\_task(ctx.pending\_task\_id) if task and task.status == "waiting\_confirmation": return await self.\_process\_confirmation(req, ctx, task, service, catalog) # 2. LLM 提取 session\_context = self.session.get\_enhanced\_context(ctx.session\_id, query\_text=req.text) timing = {} total\_start = time.time() layer1\_start = time.time() extraction\_json = await extract\_llm\_async(req.text, session\_context=session\_context) timing\["layer1\_llm\_extraction\_s"\] = round(time.time() - layer1\_start, 3) logger.debug("Layer1: user=%s, extraction=%s", req.text, json.dumps(extraction\_json.model\_dump(), ensure\_ascii=False)) # 3. Follow-up 检测 followup\_decision = detect\_followup(req.text, prev\_qs, pending\_task=False) logger.debug("Turn decision: mode=%s, confidence=%.2f, reason=%s", followup\_decision.mode, followup\_decision.confidence, followup\_decision.reason) # 4. Early exit（无 event 且非 follow-up） if not extraction\_json.event\_extractions and not followup\_decision.is\_followup: self.session.add\_message(ctx.session\_id, "user", req.text, metadata={"error": "missing\_event"}) return self.\_make\_response( extraction\_json=extraction\_json.model\_dump(), status="early\_exit", session\_id=ctx.session\_id, message="您目前没有输入任何event\_name,无法查询哦", ) # 5. Follow-up 路径 if followup\_decision.is\_followup and prev\_qs is not None: return await self.\_process\_followup( req, ctx, extraction\_json, followup\_decision, prev\_qs, service, catalog, ) # 6. 新查询路径 return await self.\_process\_new\_query( req, ctx, extraction\_json, followup\_decision, service, catalog, timing, total\_start, notify\_fn=notify\_fn, ) # ==================== 新查询路径 ==================== async def \_process\_new\_query( self, req, ctx, extraction\_json, followup\_decision, service, catalog, timing, total\_start, notify\_fn=None, ) -> dict: """处理完整的新查询""" prev\_qs = ctx.last\_query\_state # Layer2: Matcher 解析 layer2\_start = time.time() region\_filter = extraction\_json.region\_filter or \["ROW"\] region\_explain = {"stage": "llm\_direct", "region\_filter": region\_filter} metric\_result, metric\_bias = self.\_resolve\_field( service, MatcherType.METRIC, extraction\_json.metric\_extractions, DEFAULT\_METRIC, "metric", req.project\_id, ctx.user\_id, ) event\_result, event\_bias = self.\_resolve\_field( service, MatcherType.EVENT, extraction\_json.event\_extractions, DEFAULT\_EVENT, "event", req.project\_id, ctx.user\_id, ) group\_by\_result, group\_by\_bias = self.\_resolve\_field( service, MatcherType.DIMENSION, extraction\_json.group\_by\_extractions, DEFAULT\_DIMENSION, "group\_by", req.project\_id, ctx.user\_id, ) n\_days, time\_explain = service.resolve\_time(extraction\_json) logger.debug("Layer2 resolved: event=%s(%.0f/%s), metric=%s(%.0f/%s), n\_days=%d", event\_result.value, event\_result.score, event\_result.method, metric\_result.value, metric\_result.score, metric\_result.method, n\_days) timing\["layer2\_resolution\_s"\] = round(time.time() - layer2\_start, 3) resolver\_explain = self.\_build\_resolver\_explain( region\_explain, metric\_result, metric\_bias, event\_result, event\_bias, group\_by\_result, group\_by\_bias, time\_explain, ) # 确认流判断 pending\_fields = {} if event\_result.needs\_confirmation: pending\_fields\["event"\] = event\_result.candidates if metric\_result.needs\_confirmation: pending\_fields\["metric"\] = metric\_result.candidates if pending\_fields: return await self.\_create\_confirmation\_task( req, ctx, extraction\_json, pending\_fields, resolver\_explain, event\_result.value, metric\_result.value, n\_days, region\_filter, group\_by\_result.value, notify\_fn=notify\_fn, ) # 高置信度 → 构建 DSL metric\_id = metric\_result.value event\_name = event\_result.value group\_by\_dim = group\_by\_result.value if req.chat\_id and notify\_fn: region\_display = region\_filter\[0\] if region\_filter else "ROW" await notify\_fn(req.chat\_id, f"解析结果为: {region\_display}|{event\_name}|{metric\_id}|近{n\_days}天\\n正在查询中，请稍后...") semantic, exec\_dsl, timing = self.\_build\_and\_render\_dsl( req.project\_id, region\_filter, metric\_id, event\_name, n\_days, \[group\_by\_dim\], \[\], catalog, timing, ) timing\["total\_s"\] = round(time.time() - total\_start, 3) logger.debug("Timing: %s", json.dumps(timing, ensure\_ascii=False)) # 会话记录 self.session.add\_message(ctx.session\_id, "user", req.text, metadata={ "region\_filter": region\_filter, "metric\_id": metric\_id, "event\_name": event\_name, }) query\_state = self.\_build\_new\_query\_state( req.project\_id, event\_name, metric\_id, n\_days, region\_filter, \[group\_by\_dim\], ) self.session.update\_query\_state(ctx.session\_id, query\_state) self.\_record\_preferences(ctx, query\_state) # 异步记忆学习 prev\_state\_dict = prev\_qs.to\_dict() if prev\_qs else None asyncio.create\_task(self.memory.maybe\_save( project\_id=req.project\_id, user\_query=req.text, extraction=extraction\_json.model\_dump(), resolver\_explain=resolver\_explain, current\_state=query\_state.to\_dict(), prev\_state=prev\_state\_dict, )) return self.\_make\_response( extraction\_json=extraction\_json.model\_dump(), semantic=semantic.model\_dump(), exec\_dsl=exec\_dsl, explain={ "turn\_explain": self.\_build\_turn\_explain("new\_query", query\_state, decision=followup\_decision), "resolver\_explain": resolver\_explain, "timing": timing, }, session\_id=ctx.session\_id, ) # ==================== Follow-up 路径 ==================== async def \_process\_followup( self, req, ctx, extraction\_json, decision, prev\_qs, service, catalog, ) -> dict: """处理 follow-up 补丁查询""" patch\_result = self.\_build\_followup\_patch( req.text, extraction\_json, decision, service, project\_id=req.project\_id, user\_id=ctx.user\_id, ) if patch\_result.confirmation\_needed: field\_name, resolved = patch\_result.confirmation\_needed return await self.\_create\_followup\_confirmation( req, ctx, extraction\_json, decision, prev\_qs, field\_name, resolved, ) logger.debug("Follow-up patch built: patch=%s, session\_id=%s", json.dumps(patch\_result.patch, ensure\_ascii=False), ctx.session\_id) merged\_state = merge\_query\_state(prev\_qs, patch\_result.patch, req.project\_id) semantic, exec\_dsl, \_ = self.\_build\_and\_render\_dsl\_from\_state(merged\_state, catalog) self.session.add\_message(ctx.session\_id, "user", req.text, metadata={"turn\_mode": "followup\_patch", "patch": patch\_result.patch}) self.session.update\_query\_state(ctx.session\_id, merged\_state) self.\_record\_preferences(ctx, merged\_state) return self.\_make\_response( extraction\_json=extraction\_json.model\_dump(), semantic=semantic.model\_dump(), exec\_dsl=exec\_dsl, explain={ "turn\_explain": self.\_build\_turn\_explain("followup\_patch", merged\_state, decision=decision, patch=patch\_result.patch), "resolver\_explain": patch\_result.resolver\_explain, }, session\_id=ctx.session\_id, ) # ==================== 确认路径 ==================== async def \_process\_confirmation(self, req, ctx, task, service, catalog) -> dict: """处理用户对确认任务的回复""" user\_input = req.text.strip() confirmed\_values = {} for field\_name, candidates in task.candidates.items(): matched = \_match\_user\_input\_to\_candidates(user\_input, candidates) if matched: self.task.confirm\_task(task.task\_id, field\_name, matched) confirmed\_values\[field\_name\] = matched logger.info("User confirmed %s=%s for task %s", field\_name, matched, task.task\_id) if not confirmed\_values: hint\_msg = self.\_format\_unmatched\_message(task.candidates) return self.\_make\_response( extraction\_json=task.extraction, status="needs\_confirmation", session\_id=ctx.session\_id, message=hint\_msg, task\_id=task.task\_id, candidates=task.candidates, ) all\_confirmed = all(f in task.user\_selection for f in task.candidates) if not all\_confirmed: remaining = {f: c for f, c in task.candidates.items() if f not in task.user\_selection} hint\_msg = f"已确认: {confirmed\_values}\\n" + self.\_format\_candidates\_message(remaining) return self.\_make\_response( extraction\_json=task.extraction, status="needs\_confirmation", session\_id=ctx.session\_id, message=hint\_msg, task\_id=task.task\_id, candidates=remaining, ) # 全部确认 self.session.update\_pending\_task(ctx.session\_id, None) self.task.update\_status(task.task\_id, "confirmed") qs = task.partial\_query\_state for field, value in task.user\_selection.items(): if field in ("event", "metric"): setattr(qs, field, value) \_mark\_confirmation\_state(qs, task.user\_selection) semantic, exec\_dsl, \_ = self.\_build\_and\_render\_dsl\_from\_state(qs, catalog) self.session.add\_message(ctx.session\_id, "user", task.raw\_query) self.session.add\_message(ctx.session\_id, "user", user\_input, metadata={"confirmed": task.user\_selection}) self.session.update\_query\_state(ctx.session\_id, qs) self.\_record\_preferences(ctx, qs) self.task.update\_status(task.task\_id, "completed") asyncio.create\_task(self.memory.maybe\_save( project\_id=qs.project\_id, user\_query=task.raw\_query, extraction=task.extraction, resolver\_explain={"user\_confirmed": task.user\_selection}, current\_state=qs.to\_dict(), confirmed\_selection=task.user\_selection, )) return self.\_make\_response( extraction\_json=task.extraction, semantic=semantic.model\_dump(), exec\_dsl=exec\_dsl, explain={ "confirmed": task.user\_selection, "method": "user\_confirmation", "turn\_explain": self.\_build\_turn\_explain("confirmation", qs, confirmed\_fields=task.user\_selection), }, session\_id=ctx.session\_id, status="success", message=f"已按您的选择生成查询: {task.user\_selection}", ) # ==================== 确认任务创建 ==================== async def \_create\_confirmation\_task( self, req, ctx, extraction\_json, pending\_fields, resolver\_explain, event\_val, metric\_val, n\_days, region\_filter, group\_by\_val, notify\_fn=None, ) -> dict: """创建确认任务并返回 needs\_confirmation 响应""" partial\_state = QueryState( project\_id=req.project\_id, event=event\_val, metric=metric\_val, time\_range={"type": "last\_n\_days", "n": n\_days}, region\_filter=region\_filter, group\_by=\[group\_by\_val\], filters=\[\], ) task = self.task.create\_task( session\_id=ctx.session\_id, raw\_query=req.text, extraction=extraction\_json.model\_dump(), user\_id=ctx.user\_id, candidates=pending\_fields, partial\_query\_state=partial\_state, ) self.session.update\_pending\_task(ctx.session\_id, task.task\_id) confirm\_msg = self.\_format\_candidates\_message(pending\_fields) + "\\n回复编号或名称即可" if req.chat\_id and notify\_fn: await notify\_fn(req.chat\_id, confirm\_msg) return self.\_make\_response( extraction\_json=extraction\_json.model\_dump(), status="needs\_confirmation", session\_id=ctx.session\_id, message=confirm\_msg, task\_id=task.task\_id, candidates=pending\_fields, explain={"resolver\_explain": resolver\_explain}, ) async def \_create\_followup\_confirmation( self, req, ctx, extraction\_json, decision, prev\_qs, field\_name, resolved\_result, ) -> dict: """Follow-up 中触发确认流""" partial\_state = merge\_query\_state(prev\_qs, {}, req.project\_id) task = self.task.create\_task( session\_id=ctx.session\_id, raw\_query=req.text, extraction=extraction\_json.model\_dump(), user\_id=ctx.user\_id, candidates={field\_name: resolved\_result.candidates}, partial\_query\_state=partial\_state, ) self.session.update\_pending\_task(ctx.session\_id, task.task\_id) field\_display = "事件" if field\_name == "event" else "指标" lines = \[f"请选择{field\_display}:"\] lines += \[f" {i}. {c\['value'\]} (匹配度 {c\['score'\]:.0f}%)" for i, c in enumerate(resolved\_result.candidates\[:5\], 1)\] lines.append("回复编号或名称即可") return self.\_make\_response( extraction\_json=extraction\_json.model\_dump(), status="needs\_confirmation", session\_id=ctx.session\_id, message="\\n".join(lines), task\_id=task.task\_id, candidates={field\_name: resolved\_result.candidates}, explain={"turn\_explain": { "mode": "followup\_patch", "reason": decision.reason, "decision": \_serialize\_followup\_decision(decision), }}, ) # ==================== 去重：统一 resolve + bias ==================== def \_resolve\_field(self, service, matcher\_type, extractions, default, field\_name, project\_id, user\_id): """统一 resolve + user preference bias""" result = service.resolve\_with\_candidates(matcher\_type, extractions, default=default) bias = self.\_apply\_bias(result, project\_id=project\_id, user\_id=user\_id, field\_name=field\_name) return result, bias def \_apply\_bias(self, result, \*, project\_id, user\_id, field\_name): """应用用户偏好加权""" reranked, bias\_explain = self.preferences.rerank\_candidates( project\_id, user\_id, field\_name, result.candidates, ) if not bias\_explain.get("applied"): return bias\_explain top\_candidate = reranked\[0\] if reranked else None result.candidates = reranked if top\_candidate and top\_candidate\["value"\] != result.value: result.value = top\_candidate\["value"\] result.score = float(top\_candidate\["score"\]) result.method = f"{result.method}+user\_bias" elif top\_candidate: result.score = max(float(result.score), float(top\_candidate\["score"\])) return bias\_explain # ==================== 去重：统一 Follow-up Patch ==================== def \_build\_followup\_patch( self, req\_text, extraction\_json, decision, service, \*, project\_id, user\_id, ) -> FollowupPatchResult: """构建 follow-up 补丁，返回结果对象（不再用异常控制流）""" patch: dict\[str, Any\] = {} resolver\_explain: Dict\[str, Any\] = {} confirmation = None # Metric if extraction\_json.metric\_extractions: r, \_ = self.\_resolve\_field(service, MatcherType.METRIC, extraction\_json.metric\_extractions, DEFAULT\_METRIC, "metric", project\_id, user\_id) resolver\_explain\["metric"\] = {"method": r.method, "score": r.score} if r.needs\_confirmation and confirmation is None: confirmation = ("metric", r) else: patch\["metric"\] = r.value # Event if extraction\_json.event\_extractions: r, \_ = self.\_resolve\_field(service, MatcherType.EVENT, extraction\_json.event\_extractions, DEFAULT\_EVENT, "event", project\_id, user\_id) resolver\_explain\["event"\] = {"method": r.method, "score": r.score} if r.needs\_confirmation and confirmation is None: confirmation = ("event", r) else: patch\["event"\] = r.value # GroupBy if extraction\_json.group\_by\_extractions: r, \_ = self.\_resolve\_field(service, MatcherType.DIMENSION, extraction\_json.group\_by\_extractions, DEFAULT\_DIMENSION, "group\_by", project\_id, user\_id) resolver\_explain\["group\_by"\] = {"method": r.method, "score": r.score} patch\["group\_by"\] = \[r.value\] # Time if extraction\_json.time\_extractions: n\_days, time\_explain = service.resolve\_time(extraction\_json) resolver\_explain\["time"\] = time\_explain patch\["time\_range"\] = {"type": "last\_n\_days", "n": n\_days} elif decision.patch\_hints.get("time\_range"): patch\["time\_range"\] = decision.patch\_hints\["time\_range"\] resolver\_explain\["time"\] = {"method": "followup\_hint", "value": decision.patch\_hints\["time\_range"\]} # Region if \_query\_mentions\_region(req\_text): patch\["region\_filter"\] = extraction\_json.region\_filter or \["ROW"\] resolver\_explain\["region"\] = {"method": "llm\_direct", "region\_filter": patch\["region\_filter"\]} return FollowupPatchResult( patch=patch, resolver\_explain=resolver\_explain, patch\_hints=decision.patch\_hints, confirmation\_needed=confirmation, ) # ==================== 去重：统一 DSL 构建 ==================== def \_build\_semantic(self, qs: QueryState) -> SemanticDSL: """从 QueryState 构建 SemanticDSL""" return SemanticDSL( project\_id=qs.project\_id, region\_filter=qs.region\_filter or \[\], metric=Metric(metric\_id=qs.metric or DEFAULT\_METRIC), event=Event(event\_name=qs.event or DEFAULT\_EVENT), time\_range=TimeRange(type="last\_n\_days", n=(qs.time\_range or {}).get("n", DEFAULT\_N\_DAYS)), group\_by=\[GroupBy(dimension\_id=d) for d in (qs.group\_by or \[DEFAULT\_DIMENSION\])\], filters=qs.filters or \[\], ) def \_build\_and\_render\_dsl(self, project\_id, region\_filter, metric\_id, event\_name, n\_days, group\_by, filters, catalog, timing=None) -> tuple: """构建 SemanticDSL + 渲染 exec\_dsl""" semantic = SemanticDSL( project\_id=project\_id, region\_filter=region\_filter, metric=Metric(metric\_id=metric\_id), event=Event(event\_name=event\_name), time\_range=TimeRange(type="last\_n\_days", n=n\_days), group\_by=\[GroupBy(dimension\_id=d) for d in group\_by\], filters=filters or \[\], ) exec\_dsl = render\_exec\_dsl(semantic, catalog) validate\_region(exec\_dsl) validate\_region\_consistency(semantic, exec\_dsl) return semantic, exec\_dsl, timing def \_build\_and\_render\_dsl\_from\_state(self, qs: QueryState, catalog) -> tuple: """从 QueryState 构建 + 渲染""" semantic = self.\_build\_semantic(qs) exec\_dsl = render\_exec\_dsl(semantic, catalog) validate\_region(exec\_dsl) validate\_region\_consistency(semantic, exec\_dsl) return semantic, exec\_dsl, None # ==================== 去重：统一候选消息格式化 ==================== def \_format\_candidates\_message(self, pending\_fields: dict) -> str: """格式化候选提示消息""" field\_display\_map = {"event": "事件", "metric": "指标"} lines = \[\] for field\_name, cands in pending\_fields.items(): lines.append(f"请选择{field\_display\_map.get(field\_name, field\_name)}:") for i, c in enumerate(cands\[:5\], 1): lines.append(f" {i}. {c\['value'\]} (匹配度 {c\['score'\]:.0f}%)") return "\\n".join(lines) def \_format\_unmatched\_message(self, candidates: dict) -> str: """格式化未匹配候选的消息""" lines = \["未能识别您的选择，请回复编号或名称:"\] lines.append(self.\_format\_candidates\_message(candidates)) return "\\n".join(lines) # ==================== 辅助方法 ==================== def \_record\_preferences(self, ctx, qs: QueryState) -> None: self.preferences.record\_selection( qs.project\_id, ctx.user\_id, event=qs.event, metric=qs.metric, group\_by=qs.group\_by, ) def \_build\_resolver\_explain(self, region\_explain, mr, mb, er, eb, gr, gb, time\_explain) -> dict: def \_field\_explain(result, bias): e = {"method": result.method, "score": result.score} if bias.get("applied"): e\["user\_preference\_bias"\] = bias return e return { "region": region\_explain, "metric": \_field\_explain(mr, mb), "event": \_field\_explain(er, eb), "group\_by": \_field\_explain(gr, gb), "time": time\_explain, } def \_build\_new\_query\_state(self, project\_id, event, metric, n\_days, region\_filter, group\_by) -> QueryState: return QueryState( project\_id=project\_id, event=event, metric=metric, time\_range={"type": "last\_n\_days", "n": n\_days}, region\_filter=region\_filter, group\_by=group\_by, filters=\[\], explicit\_fields=\["event", "metric", "time\_range", "region\_filter", "group\_by"\], field\_sources={f: "explicit" for f in \["event", "metric", "time\_range", "region\_filter", "group\_by"\]}, turn\_type="new\_query", ) def \_build\_turn\_explain(self, mode, qs, \*, decision=None, patch=None, confirmed\_fields=None) -> dict: explain = { "mode": mode, "explicit\_fields": qs.explicit\_fields, "inherited\_fields": qs.inherited\_fields, "field\_sources": qs.field\_sources, "state\_snapshot": \_build\_state\_snapshot(qs), } if decision is not None: explain\["decision"\] = \_serialize\_followup\_decision(decision) if patch is not None: explain\["applied\_patch"\] = patch explain\["applied\_patch\_fields"\] = list(patch.keys()) if confirmed\_fields: explain\["confirmed\_fields"\] = confirmed\_fields return explain @staticmethod def \_make\_response(\*\*kwargs) -> dict: """构建响应 dict（保持字段顺序）""" defaults = {"semantic": {}, "exec\_dsl": {}, "explain": {}, "status": "success"} defaults.update(kwargs) return defaults # ==================== 模块级工具函数 ==================== def \_build\_state\_snapshot(qs: QueryState) -> Dict\[str, Any\]: return { "event": qs.event, "metric": qs.metric, "time\_range": qs.time\_range, "region\_filter": qs.region\_filter, "group\_by": qs.group\_by, } def \_serialize\_followup\_decision(decision) -> Dict\[str, Any\]: return { "mode": decision.mode, "confidence": decision.confidence, "reason": decision.reason, "matched\_signals": decision.matched\_signals, "patch\_hints": decision.patch\_hints, "normalized\_text": decision.normalized\_text, } def \_match\_user\_input\_to\_candidates(user\_input: str, candidates: list\[dict\]) -> str | None: text = user\_input.strip() if text.isdigit(): idx = int(text) - 1 if 0 <= idx < len(candidates): return candidates\[idx\]\["value"\] text\_lower = text.lower() for c in candidates: if c\["value"\].lower() == text\_lower: return c\["value"\] for c in candidates: if text\_lower in c\["value"\].lower() or c\["value"\].lower() in text\_lower: return c\["value"\] return None def \_query\_mentions\_region(text: str) -> bool: return any(token in text for token in \_REGION\_TOKENS) def \_mark\_confirmation\_state(qs: QueryState, confirmed\_fields: Dict\[str, str\]) -> None: qs.turn\_type = "confirmation" merged\_explicit = list(dict.fromkeys(\[\*(qs.explicit\_fields or \[\]), \*confirmed\_fields.keys()\])) qs.explicit\_fields = merged\_explicit active\_fields = \[f for f in ("event", "metric", "time\_range", "region\_filter", "group\_by") if getattr(qs, f, None)\] qs.inherited\_fields = \[f for f in active\_fields if f not in merged\_explicit\] field\_sources = dict(qs.field\_sources or {}) for f in active\_fields: if f in confirmed\_fields: field\_sources\[f\] = "confirmed" elif f not in field\_sources: field\_sources\[f\] = "explicit" if f in merged\_explicit else "inherited" qs.field\_sources = field\_sources

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

572

573

574

575

576

577

578

579

580

581

582

583

584

585

586

587

588

589

590

591

592

593

594

595

596

597

598

599

600

601

602

603

604

605

606

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

"""查询编排器 — 从 app.py 提取的核心业务逻辑

三条处理路径：

1\. new\_query: 完整的新查询（LLM 提取 → Matcher → DSL）

2\. followup\_patch: 基于上轮状态的修改（检测 → 补丁 → 合并 → DSL）

3\. confirmation: 用户确认低置信度候选

消除重复：

\- \_resolve\_field: 统一 resolve + user preference bias

\- \_build\_semantic: 统一 SemanticDSL 构建

\- \_format\_candidates\_message: 统一候选消息格式化

"""

from \_\_future\_\_ import annotations

import asyncio

import json

import logging

import time

from dataclasses import dataclass

from typing import Any, Dict, Optional

from common.types import MatcherType

from dsl.renderer import render\_exec\_dsl

from dsl.semantic\_models import Event, GroupBy, Metric, SemanticDSL, TimeRange

from dsl.validators import validate\_region, validate\_region\_consistency

from memory.memory\_writer import MemoryWriter

from memory.user\_preference\_store import UserPreferenceStore

from service.followup\_resolver import detect\_followup

from service.llm\_extractions import extract\_llm\_async

from service.query\_state\_merger import merge\_query\_state

from service.session\_manager import SessionManager

from service.session\_models import QueryState

from service.task\_manager import TaskManager

logger \= logging.getLogger(\_\_name\_\_)

\# ==================== 常量 ====================

DEFAULT\_METRIC \= "pv"

DEFAULT\_EVENT \= "app\_launch"

DEFAULT\_DIMENSION \= "country"

DEFAULT\_N\_DAYS \= 7

\_REGION\_TOKENS \= ("美国", "德国", "欧洲", "新加坡", "加州", "纽约", "ROW")

\# ==================== 异常 ====================

class FollowupConfirmationNeeded(Exception):

def \_\_init\_\_(self, field\_name: str, resolved\_result):

self.field\_name \= field\_name

self.resolved\_result \= resolved\_result

super().\_\_init\_\_(field\_name)

\# ==================== 数据类 ====================

@dataclass

class FollowupPatchResult:

patch: dict\[str, Any\]

resolver\_explain: Dict\[str, Any\]

patch\_hints: Dict\[str, Any\]

confirmation\_needed: Optional\[tuple\[str, Any\]\] \= None

\# ==================== 编排器 ====================

class QueryOrchestrator:

"""查询编排器：封装三条处理路径的业务逻辑"""

def \_\_init\_\_(

self,

session\_manager: SessionManager,

task\_manager: TaskManager,

memory\_writer: MemoryWriter,

user\_preference\_store: UserPreferenceStore,

):

self.session \= session\_manager

self.task \= task\_manager

self.memory \= memory\_writer

self.preferences \= user\_preference\_store

\# ==================== 主入口 ====================

async def process(self, req, service, catalog, notify\_fn\=None) \-> dict:

"""主入口：路由到三条处理路径

Args:

req: NL2DSLRequest

service: MatcherService

catalog: catalog 数据

notify\_fn: async callable(chat\_id, message) for Telegram notifications

Returns:

NL2DSLResponse dict

"""

ctx \= self.session.create\_or\_get(req.session\_id, req.user\_id, req.project\_id)

prev\_qs \= ctx.last\_query\_state

\# 1. 确认流拦截

if ctx.pending\_task\_id:

task \= self.task.get\_task(ctx.pending\_task\_id)

if task and task.status \== "waiting\_confirmation":

return await self.\_process\_confirmation(req, ctx, task, service, catalog)

\# 2. LLM 提取

session\_context \= self.session.get\_enhanced\_context(ctx.session\_id, query\_text\=req.text)

timing \= {}

total\_start \= time.time()

layer1\_start \= time.time()

extraction\_json \= await extract\_llm\_async(req.text, session\_context\=session\_context)

timing\["layer1\_llm\_extraction\_s"\] \= round(time.time() \- layer1\_start, 3)

logger.debug("Layer1: user=%s, extraction=%s", req.text,

json.dumps(extraction\_json.model\_dump(), ensure\_ascii\=False))

\# 3. Follow-up 检测

followup\_decision \= detect\_followup(req.text, prev\_qs, pending\_task\=False)

logger.debug("Turn decision: mode=%s, confidence=%.2f, reason=%s",

followup\_decision.mode, followup\_decision.confidence, followup\_decision.reason)

\# 4. Early exit（无 event 且非 follow-up）

if not extraction\_json.event\_extractions and not followup\_decision.is\_followup:

self.session.add\_message(ctx.session\_id, "user", req.text, metadata\={"error": "missing\_event"})

return self.\_make\_response(

extraction\_json\=extraction\_json.model\_dump(), status\="early\_exit",

session\_id\=ctx.session\_id, message\="您目前没有输入任何event\_name,无法查询哦",

)

\# 5. Follow-up 路径

if followup\_decision.is\_followup and prev\_qs is not None:

return await self.\_process\_followup(

req, ctx, extraction\_json, followup\_decision, prev\_qs, service, catalog,

)

\# 6. 新查询路径

return await self.\_process\_new\_query(

req, ctx, extraction\_json, followup\_decision, service, catalog,

timing, total\_start, notify\_fn\=notify\_fn,

)

\# ==================== 新查询路径 ====================

async def \_process\_new\_query(

self, req, ctx, extraction\_json, followup\_decision,

service, catalog, timing, total\_start, notify\_fn\=None,

) \-> dict:

"""处理完整的新查询"""

prev\_qs \= ctx.last\_query\_state

\# Layer2: Matcher 解析

layer2\_start \= time.time()

region\_filter \= extraction\_json.region\_filter or \["ROW"\]

region\_explain \= {"stage": "llm\_direct", "region\_filter": region\_filter}

"mode": mode,

"explicit\_fields": qs.explicit\_fields,

"inherited\_fields": qs.inherited\_fields,

"field\_sources": qs.field\_sources,

"state\_snapshot": \_build\_state\_snapshot(qs),

}

if decision is not None:

explain\["decision"\] \= \_serialize\_followup\_decision(decision)

if patch is not None:

explain\["applied\_patch"\] \= patch

explain\["applied\_patch\_fields"\] \= list(patch.keys())

if confirmed\_fields:

explain\["confirmed\_fields"\] \= confirmed\_fields

return explain

@staticmethod

def \_make\_response(\*\*kwargs) \-> dict:

"""构建响应 dict（保持字段顺序）"""

defaults \= {"semantic": {}, "exec\_dsl": {}, "explain": {}, "status": "success"}

defaults.update(kwargs)

return defaults

\# ==================== 模块级工具函数 ====================

def \_build\_state\_snapshot(qs: QueryState) \-> Dict\[str, Any\]:

return {

"event": qs.event, "metric": qs.metric,

"time\_range": qs.time\_range, "region\_filter": qs.region\_filter,

"group\_by": qs.group\_by,

}

def \_serialize\_followup\_decision(decision) \-> Dict\[str, Any\]:

return {

"mode": decision.mode, "confidence": decision.confidence,

"reason": decision.reason, "matched\_signals": decision.matched\_signals,

"patch\_hints": decision.patch\_hints, "normalized\_text": decision.normalized\_text,

}

def \_match\_user\_input\_to\_candidates(user\_input: str, candidates: list\[dict\]) \-> str | None:

text \= user\_input.strip()

if text.isdigit():

idx \= int(text) \- 1

if 0 <= idx < len(candidates):

return candidates\[idx\]\["value"\]

text\_lower \= text.lower()

for c in candidates:

if c\["value"\].lower() \== text\_lower:

return c\["value"\]

for c in candidates:

if text\_lower in c\["value"\].lower() or c\["value"\].lower() in text\_lower:

return c\["value"\]

return None

def \_query\_mentions\_region(text: str) \-> bool:

return any(token in text for token in \_REGION\_TOKENS)

def \_mark\_confirmation\_state(qs: QueryState, confirmed\_fields: Dict\[str, str\]) \-> None:

qs.turn\_type \= "confirmation"

merged\_explicit \= list(dict.fromkeys(\[\*(qs.explicit\_fields or \[\]), \*confirmed\_fields.keys()\]))

qs.explicit\_fields \= merged\_explicit

active\_fields \= \[f for f in ("event", "metric", "time\_range", "region\_filter", "group\_by") if getattr(qs, f, None)\]

qs.inherited\_fields \= \[f for f in active\_fields if f not in merged\_explicit\]

field\_sources \= dict(qs.field\_sources or {})

for f in active\_fields:

if f in confirmed\_fields:

field\_sources\[f\] \= "confirmed"

elif f not in field\_sources:

field\_sources\[f\] \= "explicit" if f in merged\_explicit else "inherited"

qs.field\_sources \= field\_sources