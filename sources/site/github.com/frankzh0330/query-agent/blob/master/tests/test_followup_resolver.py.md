# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_followup_resolver.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_followup\_resolver.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_followup_resolver.py)

Open commit details

History

58 lines (46 loc) · 2.6 KB

## FilesExpand file tree

master

/

# test\_followup\_resolver.py

Copy path

Top

## File metadata and controls

- Code

- Blame

58 lines (46 loc) · 2.6 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_followup_resolver.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""followup\_resolver.py tests.""" from service.followup\_resolver import detect\_followup from service.session\_models import QueryState class TestDetectFollowup: def test\_returns\_new\_query\_without\_previous\_state(self): decision = detect\_followup("换成昨天", None) assert decision.is\_followup is False assert decision.mode == "new\_query" assert decision.reason == "no\_previous\_state" def test\_detects\_time\_only\_followup(self, sample\_query\_state): decision = detect\_followup("昨天", sample\_query\_state) assert decision.is\_followup is True assert decision.mode == "followup\_patch" assert decision.patch\_hints\["time\_range"\]\["n"\] == 1 def test\_detects\_prefix\_followup(self, sample\_query\_state): decision = detect\_followup("换成UV", sample\_query\_state) assert decision.is\_followup is True assert decision.reason == "followup\_prefix" def test\_detects\_short\_patch\_like\_region(self, sample\_query\_state): decision = detect\_followup("那美国呢", sample\_query\_state) assert decision.is\_followup is True assert decision.mode == "followup\_patch" assert "region\_token" in decision.matched\_signals def test\_detects\_contextual\_group\_by\_patch(self, sample\_query\_state): decision = detect\_followup("还是按国家看吧", sample\_query\_state) assert decision.is\_followup is True assert decision.reason == "followup\_prefix" assert "group\_by\_token" in decision.matched\_signals assert "patch\_verb" in decision.matched\_signals def test\_detects\_comparison\_phrase\_followup(self, sample\_query\_state): decision = detect\_followup("和昨天比一下", sample\_query\_state) assert decision.is\_followup is True assert decision.reason == "comparison\_phrase" assert "comparison\_phrase" in decision.matched\_signals def test\_detects\_contextual\_metric\_patch(self, sample\_query\_state): decision = detect\_followup("也看看UV", sample\_query\_state) assert decision.is\_followup is True assert "metric\_token" in decision.matched\_signals def test\_confirmation\_reply\_wins\_when\_pending\_task(self, sample\_query\_state): decision = detect\_followup("1", sample\_query\_state, pending\_task=True) assert decision.is\_followup is False assert decision.mode == "confirmation\_reply" def test\_full\_query\_is\_not\_followup(self, sample\_query\_state): decision = detect\_followup("查询 payment\_success 的 UV", sample\_query\_state) assert decision.is\_followup is False assert decision.mode == "new\_query"

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

"""followup\_resolver.py tests."""

from service.followup\_resolver import detect\_followup

from service.session\_models import QueryState

class TestDetectFollowup:

def test\_returns\_new\_query\_without\_previous\_state(self):

decision \= detect\_followup("换成昨天", None)

assert decision.is\_followup is False

assert decision.mode \== "new\_query"

assert decision.reason \== "no\_previous\_state"

def test\_detects\_time\_only\_followup(self, sample\_query\_state):

decision \= detect\_followup("昨天", sample\_query\_state)

assert decision.is\_followup is True

assert decision.mode \== "followup\_patch"

assert decision.patch\_hints\["time\_range"\]\["n"\] \== 1

def test\_detects\_prefix\_followup(self, sample\_query\_state):

decision \= detect\_followup("换成UV", sample\_query\_state)

assert decision.is\_followup is True

assert decision.reason \== "followup\_prefix"

def test\_detects\_short\_patch\_like\_region(self, sample\_query\_state):

decision \= detect\_followup("那美国呢", sample\_query\_state)

assert decision.is\_followup is True

assert decision.mode \== "followup\_patch"

assert "region\_token" in decision.matched\_signals

def test\_detects\_contextual\_group\_by\_patch(self, sample\_query\_state):

decision \= detect\_followup("还是按国家看吧", sample\_query\_state)

assert decision.is\_followup is True

assert decision.reason \== "followup\_prefix"

assert "group\_by\_token" in decision.matched\_signals

assert "patch\_verb" in decision.matched\_signals

def test\_detects\_comparison\_phrase\_followup(self, sample\_query\_state):

decision \= detect\_followup("和昨天比一下", sample\_query\_state)

assert decision.is\_followup is True

assert decision.reason \== "comparison\_phrase"

assert "comparison\_phrase" in decision.matched\_signals

def test\_detects\_contextual\_metric\_patch(self, sample\_query\_state):

decision \= detect\_followup("也看看UV", sample\_query\_state)

assert decision.is\_followup is True

assert "metric\_token" in decision.matched\_signals

def test\_confirmation\_reply\_wins\_when\_pending\_task(self, sample\_query\_state):

decision \= detect\_followup("1", sample\_query\_state, pending\_task\=True)

assert decision.is\_followup is False

assert decision.mode \== "confirmation\_reply"

def test\_full\_query\_is\_not\_followup(self, sample\_query\_state):

decision \= detect\_followup("查询 payment\_success 的 UV", sample\_query\_state)

assert decision.is\_followup is False

assert decision.mode \== "new\_query"