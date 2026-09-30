# Source: https://github.com/frankzh0330/query-agent/blob/master/test_1.puml

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_1.puml

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

[History](https://github.com/frankzh0330/query-agent/commits/master/test_1.puml)

Open commit details

History

38 lines (34 loc) · 619 Bytes

## FilesExpand file tree

master

/

# test\_1.puml

Copy path

Top

## File metadata and controls

- Code

- Blame

38 lines (34 loc) · 619 Bytes

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/test_1.puml)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

@startyaml doe: "a deer, a female deer" ray: "a drop of golden sun" pi: 3.14159 xmas: true french-hens: 3 calling-birds: - huey - dewey - louie - fred xmas-fifth-day: calling-birds: four french-hens: 3 golden-rings: 5 partridges: count: 1 location: "a pear tree" turtle-doves: two @endyaml const t = "5a14dfabcbb685bf7c79126d1f563297ee7597821c024b6f"; const keys = \[ "gatewayToken", "openclaw.gatewayToken", "control.gatewayToken", "authToken", "token", "OPENCLAW\_GATEWAY\_TOKEN" \]; keys.forEach(k => { localStorage.setItem(k, t); sessionStorage.setItem(k, t); }); location.reload();

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

@startyaml

doe: "a deer, a female deer"

ray: "a drop of golden sun"

pi: 3.14159

xmas: true

french\-hens: 3

calling\-birds:

\- huey

\- dewey

\- louie

\- fred

xmas\-fifth\-day:

calling\-birds: four

french\-hens: 3

golden\-rings: 5

partridges:

count: 1

location: "a pear tree"

turtle\-doves: two

@endyaml

const t \= "5a14dfabcbb685bf7c79126d1f563297ee7597821c024b6f";

const keys \= \[

"gatewayToken",

"openclaw.gatewayToken",

"control.gatewayToken",

"authToken",

"token",

"OPENCLAW\_GATEWAY\_TOKEN"

\];

keys.forEach(k \=> {

localStorage.setItem(k, t);

sessionStorage.setItem(k, t);

});

location.reload();