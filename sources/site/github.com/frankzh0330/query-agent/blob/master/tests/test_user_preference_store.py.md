# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_user_preference_store.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_user\_preference\_store.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_user_preference_store.py)

Open commit details

History

33 lines (25 loc) · 1.4 KB

## FilesExpand file tree

master

/

# test\_user\_preference\_store.py

Copy path

Top

## File metadata and controls

- Code

- Blame

33 lines (25 loc) · 1.4 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_user_preference_store.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

from memory.user\_preference\_store import UserPreferenceStore class TestUserPreferenceStore: def test\_record\_selection\_is\_scoped\_by\_project\_and\_user(self, tmp\_path): store = UserPreferenceStore(data\_path=str(tmp\_path / "prefs")) store.record\_selection(55, "user\_a", event="payment\_success", metric="uv", group\_by=\["channel"\]) store.record\_selection(55, "user\_a", event="payment\_success") store.record\_selection(60, "user\_a", event="app\_launch") bucket\_55 = store.get\_bucket(55, "user\_a", "event") bucket\_60 = store.get\_bucket(60, "user\_a", "event") assert bucket\_55\["payment\_success"\] == 2 assert "app\_launch" not in bucket\_55 assert bucket\_60\["app\_launch"\] == 1 def test\_rerank\_candidates\_only\_applies\_post\_recall\_bias(self, tmp\_path): store = UserPreferenceStore(data\_path=str(tmp\_path / "prefs")) for \_ in range(6): store.record\_selection(55, "user\_a", event="payment\_submit") candidates = \[ {"value": "purchase\_success", "score": 55.0}, {"value": "payment\_submit", "score": 54.0}, \] reranked, explain = store.rerank\_candidates(55, "user\_a", "event", candidates) assert explain\["applied"\] is True assert reranked\[0\]\["value"\] == "payment\_submit" assert reranked\[0\]\["score"\] > reranked\[1\]\["score"\] assert reranked\[0\]\["preference\_count"\] == 6

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

from memory.user\_preference\_store import UserPreferenceStore

class TestUserPreferenceStore:

def test\_record\_selection\_is\_scoped\_by\_project\_and\_user(self, tmp\_path):

store \= UserPreferenceStore(data\_path\=str(tmp\_path / "prefs"))

store.record\_selection(55, "user\_a", event\="payment\_success", metric\="uv", group\_by\=\["channel"\])

store.record\_selection(55, "user\_a", event\="payment\_success")

store.record\_selection(60, "user\_a", event\="app\_launch")

bucket\_55 \= store.get\_bucket(55, "user\_a", "event")

bucket\_60 \= store.get\_bucket(60, "user\_a", "event")

assert bucket\_55\["payment\_success"\] \== 2

assert "app\_launch" not in bucket\_55

assert bucket\_60\["app\_launch"\] \== 1

def test\_rerank\_candidates\_only\_applies\_post\_recall\_bias(self, tmp\_path):

store \= UserPreferenceStore(data\_path\=str(tmp\_path / "prefs"))

for \_ in range(6):

store.record\_selection(55, "user\_a", event\="payment\_submit")

candidates \= \[

{"value": "purchase\_success", "score": 55.0},

{"value": "payment\_submit", "score": 54.0},

\]

reranked, explain \= store.rerank\_candidates(55, "user\_a", "event", candidates)

assert explain\["applied"\] is True

assert reranked\[0\]\["value"\] \== "payment\_submit"

assert reranked\[0\]\["score"\] \> reranked\[1\]\["score"\]

assert reranked\[0\]\["preference\_count"\] \== 6