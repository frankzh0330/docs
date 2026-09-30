# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/conftest.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# conftest.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/conftest.py)

Open commit details

History

67 lines (50 loc) · 1.69 KB

## FilesExpand file tree

master

/

# conftest.py

Copy path

Top

## File metadata and controls

- Code

- Blame

67 lines (50 loc) · 1.69 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/conftest.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""共享 fixtures""" import pytest from memory.storage.memory\_file import SessionStorage, TaskStorage from memory.long\_term\_memory import LongTermMemory from service.session\_manager import SessionManager from service.task\_manager import TaskManager from service.session\_models import QueryState from service.llm\_extractions import Extraction, ExtractionsJson @pytest.fixture def session\_storage(tmp\_path): return SessionStorage(data\_path=str(tmp\_path / "sessions")) @pytest.fixture def task\_storage(tmp\_path): return TaskStorage(data\_path=str(tmp\_path / "tasks")) @pytest.fixture def long\_term\_memory(tmp\_path): return LongTermMemory(data\_path=str(tmp\_path / "memory")) @pytest.fixture def session\_manager(tmp\_path): return SessionManager(data\_path=str(tmp\_path / "data")) @pytest.fixture def task\_manager(task\_storage): return TaskManager(ttl\_minutes=30, storage=task\_storage) @pytest.fixture def sample\_query\_state(): return QueryState( project\_id=55, event="app\_launch", metric="pv", time\_range={"type": "last\_n\_days", "n": 7}, region\_filter=\["ROW"\], group\_by=\["country"\], filters=\[\], ) @pytest.fixture def sample\_extractions\_json(): return ExtractionsJson( metric\_extractions=\[Extraction(text="PV")\], time\_extractions=\[Extraction(text="近7天")\], event\_extractions=\[Extraction(text="app\_launch")\], region\_filter=\["ROW"\], group\_by\_extractions=\[Extraction(text="country")\], ) @pytest.fixture def sample\_extractions\_no\_event(): return ExtractionsJson( metric\_extractions=\[Extraction(text="PV")\], event\_extractions=\[\], region\_filter=\["ROW"\], )

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

"""共享 fixtures"""

import pytest

from memory.storage.memory\_file import SessionStorage, TaskStorage

from memory.long\_term\_memory import LongTermMemory

from service.session\_manager import SessionManager

from service.task\_manager import TaskManager

from service.session\_models import QueryState

from service.llm\_extractions import Extraction, ExtractionsJson

@pytest.fixture

def session\_storage(tmp\_path):

return SessionStorage(data\_path\=str(tmp\_path / "sessions"))

@pytest.fixture

def task\_storage(tmp\_path):

return TaskStorage(data\_path\=str(tmp\_path / "tasks"))

@pytest.fixture

def long\_term\_memory(tmp\_path):

return LongTermMemory(data\_path\=str(tmp\_path / "memory"))

@pytest.fixture

def session\_manager(tmp\_path):

return SessionManager(data\_path\=str(tmp\_path / "data"))

@pytest.fixture

def task\_manager(task\_storage):

return TaskManager(ttl\_minutes\=30, storage\=task\_storage)

@pytest.fixture

def sample\_query\_state():

return QueryState(

project\_id\=55,

event\="app\_launch",

metric\="pv",

time\_range\={"type": "last\_n\_days", "n": 7},

region\_filter\=\["ROW"\],

group\_by\=\["country"\],

filters\=\[\],

)

@pytest.fixture

def sample\_extractions\_json():

return ExtractionsJson(

metric\_extractions\=\[Extraction(text\="PV")\],

time\_extractions\=\[Extraction(text\="近7天")\],

event\_extractions\=\[Extraction(text\="app\_launch")\],

region\_filter\=\["ROW"\],

group\_by\_extractions\=\[Extraction(text\="country")\],

)

@pytest.fixture

def sample\_extractions\_no\_event():

return ExtractionsJson(

metric\_extractions\=\[Extraction(text\="PV")\],

event\_extractions\=\[\],

region\_filter\=\["ROW"\],

)