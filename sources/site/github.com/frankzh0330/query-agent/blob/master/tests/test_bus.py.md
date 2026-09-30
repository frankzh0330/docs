# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_bus.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_bus.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add TaskManager confirmation flow and rebuild test suite (138 t…](https://github.com/frankzh0330/query-agent/commit/68792e59e016c780e1c83c89e4205a0ab5e4fafb)

Open commit details

Apr 14, 2026

[68792e5](https://github.com/frankzh0330/query-agent/commit/68792e59e016c780e1c83c89e4205a0ab5e4fafb) · Apr 14, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_bus.py)

Open commit details

History

137 lines (110 loc) · 4.96 KB

## FilesExpand file tree

master

/

# test\_bus.py

Copy path

Top

## File metadata and controls

- Code

- Blame

137 lines (110 loc) · 4.96 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_bus.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Bus 消息总线测试""" import os from unittest import mock import pytest class TestBusMessage: """bus/message\_schema.py""" def test\_bus\_message\_defaults(self): from bus.message\_schema import BusMessage msg = BusMessage(channel="telegram", chat\_id="123", text="hello") assert msg.msg\_id # auto-generated assert msg.project\_id == 55 assert msg.created\_at > 0 def test\_bus\_message\_serialization\_roundtrip(self): from bus.message\_schema import BusMessage msg = BusMessage(channel="telegram", chat\_id="123", text="hello") json\_str = msg.model\_dump\_json() restored = BusMessage.model\_validate\_json(json\_str) assert restored.msg\_id == msg.msg\_id assert restored.channel == msg.channel assert restored.text == msg.text class TestBusResult: """bus/message\_schema.py""" def test\_bus\_result\_success(self): from bus.message\_schema import BusMessage, BusResult msg = BusMessage(channel="telegram", chat\_id="123", text="hello") result = BusResult(msg=msg, success=True) assert result.nl2dsl\_result is None assert result.query\_result is None def test\_bus\_result\_failure(self): from bus.message\_schema import BusMessage, BusResult msg = BusMessage(channel="telegram", chat\_id="123", text="hello") result = BusResult(msg=msg, success=False, error="boom") assert result.error == "boom" class TestDirectCallBus: """bus/direct\_call\_bus.py""" @pytest.mark.asyncio async def test\_enqueue\_without\_worker\_raises(self): from bus.direct\_call\_bus import DirectCallBus from bus.message\_schema import BusMessage bus = DirectCallBus() msg = BusMessage(channel="telegram", chat\_id="123", text="hello") with pytest.raises(RuntimeError, match="Worker not bound"): await bus.enqueue\_request(msg) @pytest.mark.asyncio async def test\_enqueue\_calls\_worker(self): from bus.direct\_call\_bus import DirectCallBus from bus.message\_schema import BusMessage bus = DirectCallBus() mock\_worker = mock.AsyncMock() mock\_worker.process.return\_value = {"nl2dsl": {"status": "ok"}, "query": {}} bus.bind\_worker(mock\_worker) msg = BusMessage(channel="telegram", chat\_id="123", text="hello") await bus.enqueue\_request(msg) mock\_worker.process.assert\_called\_once\_with(msg) @pytest.mark.asyncio async def test\_enqueue\_dispatches\_result\_on\_success(self): from bus.direct\_call\_bus import DirectCallBus from bus.message\_schema import BusMessage bus = DirectCallBus() mock\_worker = mock.AsyncMock() mock\_worker.process.return\_value = {"nl2dsl": {"status": "ok"}, "query": {}} mock\_dispatcher = mock.AsyncMock() bus.bind\_worker(mock\_worker) bus.bind\_dispatcher(mock\_dispatcher) msg = BusMessage(channel="telegram", chat\_id="123", text="hello") await bus.enqueue\_request(msg) mock\_dispatcher.dispatch.assert\_called\_once() bus\_result = mock\_dispatcher.dispatch.call\_args\[0\]\[0\] assert bus\_result.success is True @pytest.mark.asyncio async def test\_enqueue\_dispatches\_error\_on\_failure(self): from bus.direct\_call\_bus import DirectCallBus from bus.message\_schema import BusMessage bus = DirectCallBus() mock\_worker = mock.AsyncMock() mock\_worker.process.side\_effect = ValueError("test error") mock\_dispatcher = mock.AsyncMock() bus.bind\_worker(mock\_worker) bus.bind\_dispatcher(mock\_dispatcher) msg = BusMessage(channel="telegram", chat\_id="123", text="hello") await bus.enqueue\_request(msg) bus\_result = mock\_dispatcher.dispatch.call\_args\[0\]\[0\] assert bus\_result.success is False assert "test error" in bus\_result.error @pytest.mark.asyncio async def test\_dequeue\_raises\_not\_implemented(self): from bus.direct\_call\_bus import DirectCallBus bus = DirectCallBus() with pytest.raises(NotImplementedError): await bus.dequeue\_request() @pytest.mark.asyncio async def test\_dequeue\_result\_raises\_not\_implemented(self): from bus.direct\_call\_bus import DirectCallBus bus = DirectCallBus() with pytest.raises(NotImplementedError): await bus.dequeue\_result() class TestCreateBus: """bus/\_\_init\_\_.py""" def test\_create\_bus\_direct\_default(self): from bus import create\_bus from bus.direct\_call\_bus import DirectCallBus bus = create\_bus() assert isinstance(bus, DirectCallBus) def test\_create\_bus\_redis(self): from bus import create\_bus mock\_instance = mock.MagicMock() with mock.patch.dict(os.environ, {"MESSAGE\_BUS\_BACKEND": "redis"}): with mock.patch("bus.redis\_bus.RedisMessageBus", return\_value=mock\_instance): bus = create\_bus() assert bus is mock\_instance

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

"""Bus 消息总线测试"""

import os

from unittest import mock

import pytest

class TestBusMessage:

"""bus/message\_schema.py"""

def test\_bus\_message\_defaults(self):

from bus.message\_schema import BusMessage

msg \= BusMessage(channel\="telegram", chat\_id\="123", text\="hello")

assert msg.msg\_id \# auto-generated

assert msg.project\_id \== 55

assert msg.created\_at \> 0

def test\_bus\_message\_serialization\_roundtrip(self):

from bus.message\_schema import BusMessage

msg \= BusMessage(channel\="telegram", chat\_id\="123", text\="hello")

json\_str \= msg.model\_dump\_json()

restored \= BusMessage.model\_validate\_json(json\_str)

assert restored.msg\_id \== msg.msg\_id

assert restored.channel \== msg.channel

assert restored.text \== msg.text

class TestBusResult:

"""bus/message\_schema.py"""

def test\_bus\_result\_success(self):

from bus.message\_schema import BusMessage, BusResult

msg \= BusMessage(channel\="telegram", chat\_id\="123", text\="hello")

result \= BusResult(msg\=msg, success\=True)

assert result.nl2dsl\_result is None

assert result.query\_result is None

def test\_bus\_result\_failure(self):

from bus.message\_schema import BusMessage, BusResult

msg \= BusMessage(channel\="telegram", chat\_id\="123", text\="hello")

result \= BusResult(msg\=msg, success\=False, error\="boom")

assert result.error \== "boom"

class TestDirectCallBus:

"""bus/direct\_call\_bus.py"""

@pytest.mark.asyncio

async def test\_enqueue\_without\_worker\_raises(self):

from bus.direct\_call\_bus import DirectCallBus

from bus.message\_schema import BusMessage

bus \= DirectCallBus()

msg \= BusMessage(channel\="telegram", chat\_id\="123", text\="hello")

with pytest.raises(RuntimeError, match\="Worker not bound"):

await bus.enqueue\_request(msg)

@pytest.mark.asyncio

async def test\_enqueue\_calls\_worker(self):

from bus.direct\_call\_bus import DirectCallBus

from bus.message\_schema import BusMessage

bus \= DirectCallBus()

mock\_worker \= mock.AsyncMock()

mock\_worker.process.return\_value \= {"nl2dsl": {"status": "ok"}, "query": {}}

bus.bind\_worker(mock\_worker)

msg \= BusMessage(channel\="telegram", chat\_id\="123", text\="hello")

await bus.enqueue\_request(msg)

mock\_worker.process.assert\_called\_once\_with(msg)

@pytest.mark.asyncio

async def test\_enqueue\_dispatches\_result\_on\_success(self):

from bus.direct\_call\_bus import DirectCallBus

from bus.message\_schema import BusMessage

bus \= DirectCallBus()

mock\_worker \= mock.AsyncMock()

mock\_worker.process.return\_value \= {"nl2dsl": {"status": "ok"}, "query": {}}

mock\_dispatcher \= mock.AsyncMock()

bus.bind\_worker(mock\_worker)

bus.bind\_dispatcher(mock\_dispatcher)

msg \= BusMessage(channel\="telegram", chat\_id\="123", text\="hello")

await bus.enqueue\_request(msg)

mock\_dispatcher.dispatch.assert\_called\_once()

bus\_result \= mock\_dispatcher.dispatch.call\_args\[0\]\[0\]

assert bus\_result.success is True

@pytest.mark.asyncio

async def test\_enqueue\_dispatches\_error\_on\_failure(self):

from bus.direct\_call\_bus import DirectCallBus

from bus.message\_schema import BusMessage

bus \= DirectCallBus()

mock\_worker \= mock.AsyncMock()

mock\_worker.process.side\_effect \= ValueError("test error")

mock\_dispatcher \= mock.AsyncMock()

bus.bind\_worker(mock\_worker)

bus.bind\_dispatcher(mock\_dispatcher)

msg \= BusMessage(channel\="telegram", chat\_id\="123", text\="hello")

await bus.enqueue\_request(msg)

bus\_result \= mock\_dispatcher.dispatch.call\_args\[0\]\[0\]

assert bus\_result.success is False

assert "test error" in bus\_result.error

@pytest.mark.asyncio

async def test\_dequeue\_raises\_not\_implemented(self):

from bus.direct\_call\_bus import DirectCallBus

bus \= DirectCallBus()

with pytest.raises(NotImplementedError):

await bus.dequeue\_request()

@pytest.mark.asyncio

async def test\_dequeue\_result\_raises\_not\_implemented(self):

from bus.direct\_call\_bus import DirectCallBus

bus \= DirectCallBus()

with pytest.raises(NotImplementedError):

await bus.dequeue\_result()

class TestCreateBus:

"""bus/\_\_init\_\_.py"""

def test\_create\_bus\_direct\_default(self):

from bus import create\_bus

from bus.direct\_call\_bus import DirectCallBus

bus \= create\_bus()

assert isinstance(bus, DirectCallBus)

def test\_create\_bus\_redis(self):

from bus import create\_bus

mock\_instance \= mock.MagicMock()

with mock.patch.dict(os.environ, {"MESSAGE\_BUS\_BACKEND": "redis"}):

with mock.patch("bus.redis\_bus.RedisMessageBus", return\_value\=mock\_instance):

bus \= create\_bus()

assert bus is mock\_instance