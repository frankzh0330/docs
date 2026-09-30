# Source: https://github.com/frankzh0330/query-agent/tree/master/bus

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# bus

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

[fix: improve early\_exit handling, logging config, and tool calling fl…](https://github.com/frankzh0330/query-agent/commit/874b818935aa4d48f0c46b5a83ab4b31069dcdfa)

Open commit details

Apr 15, 2026

[874b818](https://github.com/frankzh0330/query-agent/commit/874b818935aa4d48f0c46b5a83ab4b31069dcdfa) · Apr 15, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/bus)

Open commit details

History

## FilesExpand file tree

master

/

# bus

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

[\_\_init\_\_.py](https://github.com/frankzh0330/query-agent/blob/master/bus/__init__.py '__init__.py')

 | 

[\_\_init\_\_.py](https://github.com/frankzh0330/query-agent/blob/master/bus/__init__.py '__init__.py')

 | 

[feat: add message bus, async worker and ingress layer for multi-gatew…](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1 'feat: add message bus, async worker and ingress layer for multi-gateway architecture
Refactor gateway, enhance bearer service and session models, replace PlantUML with Mermaid diagrams.')

 | 

Apr 14, 2026

 |
| 

[base.py](https://github.com/frankzh0330/query-agent/blob/master/bus/base.py 'base.py')

 | 

[base.py](https://github.com/frankzh0330/query-agent/blob/master/bus/base.py 'base.py')

 | 

[feat: add message bus, async worker and ingress layer for multi-gatew…](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1 'feat: add message bus, async worker and ingress layer for multi-gateway architecture
Refactor gateway, enhance bearer service and session models, replace PlantUML with Mermaid diagrams.')

 | 

Apr 14, 2026

 |
| 

[direct\_call\_bus.py](https://github.com/frankzh0330/query-agent/blob/master/bus/direct_call_bus.py 'direct_call_bus.py')

 | 

[direct\_call\_bus.py](https://github.com/frankzh0330/query-agent/blob/master/bus/direct_call_bus.py 'direct_call_bus.py')

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

[message\_schema.py](https://github.com/frankzh0330/query-agent/blob/master/bus/message_schema.py 'message_schema.py')

 | 

[message\_schema.py](https://github.com/frankzh0330/query-agent/blob/master/bus/message_schema.py 'message_schema.py')

 | 

[feat: add message bus, async worker and ingress layer for multi-gatew…](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1 'feat: add message bus, async worker and ingress layer for multi-gateway architecture
Refactor gateway, enhance bearer service and session models, replace PlantUML with Mermaid diagrams.')

 | 

Apr 14, 2026

 |
| 

[redis\_bus.py](https://github.com/frankzh0330/query-agent/blob/master/bus/redis_bus.py 'redis_bus.py')

 | 

[redis\_bus.py](https://github.com/frankzh0330/query-agent/blob/master/bus/redis_bus.py 'redis_bus.py')

 | 

[feat: add message bus, async worker and ingress layer for multi-gatew…](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1 'feat: add message bus, async worker and ingress layer for multi-gateway architecture
Refactor gateway, enhance bearer service and session models, replace PlantUML with Mermaid diagrams.')

 | 

Apr 14, 2026

 |
| 

View all files

 |