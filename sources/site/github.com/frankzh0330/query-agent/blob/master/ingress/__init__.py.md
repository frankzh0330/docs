# Source: https://github.com/frankzh0330/query-agent/blob/master/ingress/__init__.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# \_\_init\_\_.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add message bus, async worker and ingress layer for multi-gatew…](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1)

Open commit details

Apr 14, 2026

[ec3eb43](https://github.com/frankzh0330/query-agent/commit/ec3eb43feaf927dececb50be87bf722d47ab7cb1) · Apr 14, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/ingress/__init__.py)

Open commit details

History

15 lines (13 loc) · 440 Bytes

## FilesExpand file tree

master

/

# \_\_init\_\_.py

Copy path

Top

## File metadata and controls

- Code

- Blame

15 lines (13 loc) · 440 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/ingress/__init__.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Ingress Adapter 层 - 统一处理外部消息输入""" from ingress.models import StandardMessage from ingress.base\_adapter import BaseIngressAdapter from ingress.cleaner import MessageCleaner from ingress.deduplicator import MessageDeduplicator from ingress.telegram\_adapter import TelegramAdapter \_\_all\_\_ = \[ "StandardMessage", "BaseIngressAdapter", "MessageCleaner", "MessageDeduplicator", "TelegramAdapter", \]

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

"""Ingress Adapter 层 - 统一处理外部消息输入"""

from ingress.models import StandardMessage

from ingress.base\_adapter import BaseIngressAdapter

from ingress.cleaner import MessageCleaner

from ingress.deduplicator import MessageDeduplicator

from ingress.telegram\_adapter import TelegramAdapter

\_\_all\_\_ \= \[

"StandardMessage",

"BaseIngressAdapter",

"MessageCleaner",

"MessageDeduplicator",

"TelegramAdapter",

\]