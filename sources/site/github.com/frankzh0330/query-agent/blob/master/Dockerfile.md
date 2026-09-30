# Source: https://github.com/frankzh0330/query-agent/blob/master/Dockerfile

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# Dockerfile

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

[History](https://github.com/frankzh0330/query-agent/commits/master/Dockerfile)

Open commit details

History

15 lines (9 loc) · 261 Bytes

## FilesExpand file tree

master

/

# Dockerfile

Copy path

Top

## File metadata and controls

- Code

- Blame

15 lines (9 loc) · 261 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/Dockerfile)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

FROM python:3.11-slim WORKDIR /app ENV PYTHONDONTWRITEBYTECODE=1 \\ PYTHONUNBUFFERED=1 COPY requirements.txt . RUN pip install --no-cache-dir -r requirements.txt COPY . . EXPOSE 8000 CMD \["uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8000"\]

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

FROM python:3.11-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1 \\

PYTHONUNBUFFERED=1

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD \["uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8000"\]