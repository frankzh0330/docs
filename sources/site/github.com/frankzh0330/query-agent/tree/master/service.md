# Source: https://github.com/frankzh0330/query-agent/tree/master/service

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# service

/

Copy path

## Directory actions

## More options

More options

## Directory actions

## More options

More options

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[update docs and removed unused codes](https://github.com/frankzh0330/query-agent/commit/28cd5d550776847f844dead59a860a776081b9b4)

May 1, 2026

[28cd5d5](https://github.com/frankzh0330/query-agent/commit/28cd5d550776847f844dead59a860a776081b9b4) · May 1, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/service)

Open commit details

History

## FilesExpand file tree

master

/

# service

/

Copy path

Top

## Folders and files

| Name | Name | 
Last commit message

 | 

Last commit date

 |
| --- | --- | --- | --- |
| 

### parent directory

[..](https://github.com/frankzh0330/query-agent/tree/master) |
| 

[\_\_init\_\_.py](https://github.com/frankzh0330/query-agent/blob/master/service/__init__.py '__init__.py')

 | 

[\_\_init\_\_.py](https://github.com/frankzh0330/query-agent/blob/master/service/__init__.py '__init__.py')

 | 

[feat: add message bus, async worker and ingress layer for multi-gatew…](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1 'feat: add message bus, async worker and ingress layer for multi-gateway architecture
Refactor gateway, enhance bearer service and session models, replace PlantUML with Mermaid diagrams.')

 | 

Apr 14, 2026

 |
| 

[bearer\_service.py](https://github.com/frankzh0330/query-agent/blob/master/service/bearer_service.py 'bearer_service.py')

 | 

[bearer\_service.py](https://github.com/frankzh0330/query-agent/blob/master/service/bearer_service.py 'bearer_service.py')

 | 

[fix: improve early\_exit handling, logging config, and tool calling fl…](https://github.com/frankzh0330/query-agent/commit/874b818935aa4d48f0c46b5a83ab4b31069dcdfa 'fix: improve early_exit handling, logging config, and tool calling flexibility

- Add event inheritance from session context (last_query_state) so follow-up
 queries without explicit event can reuse the previous one instead of early_exit
- Fix DirectCallBus to pass early_exit error message to BusResult for proper
 Telegram display
- Add logging.basicConfig() with LOG_LEVEL env var (default DEBUG) so all
 logger.info/debug calls actually print
- Remove hardcoded return True in bearer_service mock mode, restore env var
- Replace remaining print() with logger calls in bearer_service
- Add import json to bearer_service
- Make tool calling configurable via TOOL_CALLING_ENABLED env var (default true)
- Update extraction prompt: only extract event when explicitly mentioned, no
 guessing or default filling
- Update few-shot examples to show empty event_extractions for queries without
 events
- Unify uvicorn log_level with LOG_LEVEL env var')

 | 

Apr 15, 2026

 |
| 

[catalog\_scheduler.py](https://github.com/frankzh0330/query-agent/blob/master/service/catalog_scheduler.py 'catalog_scheduler.py')

 | 

[catalog\_scheduler.py](https://github.com/frankzh0330/query-agent/blob/master/service/catalog_scheduler.py 'catalog_scheduler.py')

 | 

[feat: add alias/token limits and dynamic catalog sync](https://github.com/frankzh0330/query-agent/commit/973279b515c7a09e1ccf1bda4ca282547f0145de 'feat: add alias/token limits and dynamic catalog sync

- Add max_tokens=5 limit to tokenize_mixed()
- Add max 20 chars filter for aliases in _build_inverted_index()
- Add CatalogSync service for HTTP API based events/dimensions sync
- Add CatalogScheduler with APScheduler for periodic sync (1h incremental, 6h full)
- Integrate hot-reload mechanism for MatcherService')

 | 

Mar 22, 2026

 |
| 

[catalog\_sync.py](https://github.com/frankzh0330/query-agent/blob/master/service/catalog_sync.py 'catalog_sync.py')

 | 

[catalog\_sync.py](https://github.com/frankzh0330/query-agent/blob/master/service/catalog_sync.py 'catalog_sync.py')

 | 

[feat: add alias/token limits and dynamic catalog sync](https://github.com/frankzh0330/query-agent/commit/973279b515c7a09e1ccf1bda4ca282547f0145de 'feat: add alias/token limits and dynamic catalog sync

- Add max_tokens=5 limit to tokenize_mixed()
- Add max 20 chars filter for aliases in _build_inverted_index()
- Add CatalogSync service for HTTP API based events/dimensions sync
- Add CatalogScheduler with APScheduler for periodic sync (1h incremental, 6h full)
- Integrate hot-reload mechanism for MatcherService')

 | 

Mar 22, 2026

 |
| 

[followup\_resolver.py](https://github.com/frankzh0330/query-agent/blob/master/service/followup_resolver.py 'followup_resolver.py')

 | 

[followup\_resolver.py](https://github.com/frankzh0330/query-agent/blob/master/service/followup_resolver.py 'followup_resolver.py')

 | 

[feat: add multi-turn follow-up query pipeline and extract QueryOrches…](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4 'feat: add multi-turn follow-up query pipeline and extract QueryOrchestrator')

 | 

May 1, 2026

 |
| 

[llm\_extractions.py](https://github.com/frankzh0330/query-agent/blob/master/service/llm_extractions.py 'llm_extractions.py')

 | 

[llm\_extractions.py](https://github.com/frankzh0330/query-agent/blob/master/service/llm_extractions.py 'llm_extractions.py')

 | 

[update docs and removed unused codes](https://github.com/frankzh0330/query-agent/commit/28cd5d550776847f844dead59a860a776081b9b4 'update docs and removed unused codes')

 | 

May 1, 2026

 |
| 

[query\_orchestrator.py](https://github.com/frankzh0330/query-agent/blob/master/service/query_orchestrator.py 'query_orchestrator.py')

 | 

[query\_orchestrator.py](https://github.com/frankzh0330/query-agent/blob/master/service/query_orchestrator.py 'query_orchestrator.py')

 | 

[feat: add multi-turn follow-up query pipeline and extract QueryOrches…](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4 'feat: add multi-turn follow-up query pipeline and extract QueryOrchestrator')

 | 

May 1, 2026

 |
| 

[query\_state\_merger.py](https://github.com/frankzh0330/query-agent/blob/master/service/query_state_merger.py 'query_state_merger.py')

 | 

[query\_state\_merger.py](https://github.com/frankzh0330/query-agent/blob/master/service/query_state_merger.py 'query_state_merger.py')

 | 

[feat: add multi-turn follow-up query pipeline and extract QueryOrches…](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4 'feat: add multi-turn follow-up query pipeline and extract QueryOrchestrator')

 | 

May 1, 2026

 |
| 

[session\_manager.py](https://github.com/frankzh0330/query-agent/blob/master/service/session_manager.py 'session_manager.py')

 | 

[session\_manager.py](https://github.com/frankzh0330/query-agent/blob/master/service/session_manager.py 'session_manager.py')

 | 

[update docs and removed unused codes](https://github.com/frankzh0330/query-agent/commit/28cd5d550776847f844dead59a860a776081b9b4 'update docs and removed unused codes')

 | 

May 1, 2026

 |
| 

[session\_models.py](https://github.com/frankzh0330/query-agent/blob/master/service/session_models.py 'session_models.py')

 | 

[session\_models.py](https://github.com/frankzh0330/query-agent/blob/master/service/session_models.py 'session_models.py')

 | 

[feat: add multi-turn follow-up query pipeline and extract QueryOrches…](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4 'feat: add multi-turn follow-up query pipeline and extract QueryOrchestrator')

 | 

May 1, 2026

 |
| 

[task\_manager.py](https://github.com/frankzh0330/query-agent/blob/master/service/task_manager.py 'task_manager.py')

 | 

[task\_manager.py](https://github.com/frankzh0330/query-agent/blob/master/service/task_manager.py 'task_manager.py')

 | 

[feat: add multi-turn follow-up query pipeline and extract QueryOrches…](https://github.com/frankzh0330/query-agent/commit/3560e9590f95c224a8ef2350a454f1c473237ac4 'feat: add multi-turn follow-up query pipeline and extract QueryOrchestrator')

 | 

May 1, 2026

 |
| 

View all files

 |