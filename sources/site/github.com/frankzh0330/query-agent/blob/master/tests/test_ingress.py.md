# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_ingress.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_ingress.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_ingress.py)

Open commit details

History

169 lines (137 loc) · 5.94 KB

## FilesExpand file tree

master

/

# test\_ingress.py

Copy path

Top

## File metadata and controls

- Code

- Blame

169 lines (137 loc) · 5.94 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_ingress.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""Ingress 模块测试：清洗、去重、Telegram 适配""" import time class TestMessageDeduplicator: """ingress/deduplicator.py""" def test\_new\_message\_not\_duplicate(self): from ingress.deduplicator import MessageDeduplicator dedup = MessageDeduplicator() assert dedup.is\_duplicate("msg\_1") is False def test\_same\_message\_is\_duplicate(self): from ingress.deduplicator import MessageDeduplicator dedup = MessageDeduplicator() dedup.is\_duplicate("msg\_1") assert dedup.is\_duplicate("msg\_1") is True def test\_different\_messages\_not\_duplicate(self): from ingress.deduplicator import MessageDeduplicator dedup = MessageDeduplicator() dedup.is\_duplicate("msg\_1") assert dedup.is\_duplicate("msg\_2") is False def test\_ttl\_expiry(self): from ingress.deduplicator import MessageDeduplicator dedup = MessageDeduplicator(ttl\_seconds=1) dedup.is\_duplicate("msg\_1") time.sleep(1.1) assert dedup.is\_duplicate("msg\_1") is False def test\_window\_size\_eviction(self): from ingress.deduplicator import MessageDeduplicator dedup = MessageDeduplicator(window\_size=2) dedup.is\_duplicate("msg\_1") dedup.is\_duplicate("msg\_2") dedup.is\_duplicate("msg\_3") # msg\_1 evicted assert dedup.is\_duplicate("msg\_1") is False def test\_reset\_clears\_all(self): from ingress.deduplicator import MessageDeduplicator dedup = MessageDeduplicator() dedup.is\_duplicate("msg\_1") dedup.is\_duplicate("msg\_2") dedup.reset() assert dedup.size() == 0 assert dedup.is\_duplicate("msg\_1") is False def test\_size\_tracking(self): from ingress.deduplicator import MessageDeduplicator dedup = MessageDeduplicator() for i in range(5): dedup.is\_duplicate(f"msg\_{i}") assert dedup.size() == 5 class TestMessageCleaner: """ingress/cleaner.py""" def test\_clean\_normal\_text(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() assert cleaner.clean("hello world") == "hello world" def test\_clean\_control\_chars(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() result = cleaner.clean("hello\\x00world\\x01") assert "hello" in result assert "world" in result assert "\\x00" not in result assert "\\x01" not in result def test\_clean\_bot\_command\_prefix(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() result = cleaner.clean("/start hello") assert result == "hello" def test\_clean\_empty\_input(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() assert cleaner.clean("") == "" def test\_clean\_too\_short\_returns\_empty(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() assert cleaner.clean("a") == "" def test\_clean\_chinese\_preserved(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() assert cleaner.clean("应用启动") == "应用启动" def test\_is\_valid\_true(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() assert cleaner.is\_valid("hello") is True def test\_is\_valid\_false\_empty(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() assert cleaner.is\_valid("") is False def test\_is\_valid\_false\_too\_short(self): from ingress.cleaner import MessageCleaner cleaner = MessageCleaner() assert cleaner.is\_valid("a") is False class TestTelegramAdapter: """ingress/telegram\_adapter.py""" def \_make\_update(self, text="hello", update\_id=1, message\_id=100, user\_id=42, chat\_id=999): return { "update\_id": update\_id, "message": { "message\_id": message\_id, "from": {"id": user\_id}, "chat": {"id": chat\_id}, "text": text, }, } def test\_adapt\_basic\_message(self): from ingress.telegram\_adapter import TelegramAdapter from ingress.deduplicator import MessageDeduplicator adapter = TelegramAdapter(deduplicator=MessageDeduplicator()) msg = adapter.adapt(self.\_make\_update()) assert msg.channel == "telegram" assert msg.user\_id == "42" assert msg.chat\_id == "999" assert msg.text == "hello" assert msg.is\_duplicate is False def test\_adapt\_cleans\_text(self): from ingress.telegram\_adapter import TelegramAdapter from ingress.deduplicator import MessageDeduplicator adapter = TelegramAdapter(deduplicator=MessageDeduplicator()) msg = adapter.adapt(self.\_make\_update(text="/start 查询PV")) assert msg.text == "查询PV" assert msg.is\_cleaned is True def test\_adapt\_detects\_duplicate(self): from ingress.telegram\_adapter import TelegramAdapter from ingress.deduplicator import MessageDeduplicator dedup = MessageDeduplicator() adapter = TelegramAdapter(deduplicator=dedup) update = self.\_make\_update(update\_id=42, message\_id=100) adapter.adapt(update) msg2 = adapter.adapt(update) assert msg2.is\_duplicate is True def test\_extract\_message\_id\_format(self): from ingress.telegram\_adapter import TelegramAdapter adapter = TelegramAdapter() msg\_id = adapter.extract\_message\_id(self.\_make\_update(update\_id=42, message\_id=100)) assert msg\_id == "telegram\_42\_100" def test\_adapt\_empty\_text(self): from ingress.telegram\_adapter import TelegramAdapter from ingress.deduplicator import MessageDeduplicator adapter = TelegramAdapter(deduplicator=MessageDeduplicator()) msg = adapter.adapt(self.\_make\_update(text="")) assert msg.text == ""

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

156

157

158

159

160

161

162

163

164

165

166

167

168

169

"""Ingress 模块测试：清洗、去重、Telegram 适配"""

import time

class TestMessageDeduplicator:

"""ingress/deduplicator.py"""

def test\_new\_message\_not\_duplicate(self):

from ingress.deduplicator import MessageDeduplicator

dedup \= MessageDeduplicator()

assert dedup.is\_duplicate("msg\_1") is False

def test\_same\_message\_is\_duplicate(self):

from ingress.deduplicator import MessageDeduplicator

dedup \= MessageDeduplicator()

dedup.is\_duplicate("msg\_1")

assert dedup.is\_duplicate("msg\_1") is True

def test\_different\_messages\_not\_duplicate(self):

from ingress.deduplicator import MessageDeduplicator

dedup \= MessageDeduplicator()

dedup.is\_duplicate("msg\_1")

assert dedup.is\_duplicate("msg\_2") is False

def test\_ttl\_expiry(self):

from ingress.deduplicator import MessageDeduplicator

dedup \= MessageDeduplicator(ttl\_seconds\=1)

dedup.is\_duplicate("msg\_1")

time.sleep(1.1)

assert dedup.is\_duplicate("msg\_1") is False

def test\_window\_size\_eviction(self):

from ingress.deduplicator import MessageDeduplicator

dedup \= MessageDeduplicator(window\_size\=2)

dedup.is\_duplicate("msg\_1")

dedup.is\_duplicate("msg\_2")

dedup.is\_duplicate("msg\_3") \# msg\_1 evicted

assert dedup.is\_duplicate("msg\_1") is False

def test\_reset\_clears\_all(self):

from ingress.deduplicator import MessageDeduplicator

dedup \= MessageDeduplicator()

dedup.is\_duplicate("msg\_1")

dedup.is\_duplicate("msg\_2")

dedup.reset()

assert dedup.size() \== 0

assert dedup.is\_duplicate("msg\_1") is False

def test\_size\_tracking(self):

from ingress.deduplicator import MessageDeduplicator

dedup \= MessageDeduplicator()

for i in range(5):

dedup.is\_duplicate(f"msg\_{i}")

assert dedup.size() \== 5

class TestMessageCleaner:

"""ingress/cleaner.py"""

def test\_clean\_normal\_text(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

assert cleaner.clean("hello world") \== "hello world"

def test\_clean\_control\_chars(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

result \= cleaner.clean("hello\\x00world\\x01")

assert "hello" in result

assert "world" in result

assert "\\x00" not in result

assert "\\x01" not in result

def test\_clean\_bot\_command\_prefix(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

result \= cleaner.clean("/start hello")

assert result \== "hello"

def test\_clean\_empty\_input(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

assert cleaner.clean("") \== ""

def test\_clean\_too\_short\_returns\_empty(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

assert cleaner.clean("a") \== ""

def test\_clean\_chinese\_preserved(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

assert cleaner.clean("应用启动") \== "应用启动"

def test\_is\_valid\_true(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

assert cleaner.is\_valid("hello") is True

def test\_is\_valid\_false\_empty(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

assert cleaner.is\_valid("") is False

def test\_is\_valid\_false\_too\_short(self):

from ingress.cleaner import MessageCleaner

cleaner \= MessageCleaner()

assert cleaner.is\_valid("a") is False

class TestTelegramAdapter:

"""ingress/telegram\_adapter.py"""

def \_make\_update(self, text\="hello", update\_id\=1, message\_id\=100, user\_id\=42, chat\_id\=999):

return {

"update\_id": update\_id,

"message": {

"message\_id": message\_id,

"from": {"id": user\_id},

"chat": {"id": chat\_id},

"text": text,

},

}

def test\_adapt\_basic\_message(self):

from ingress.telegram\_adapter import TelegramAdapter

from ingress.deduplicator import MessageDeduplicator

adapter \= TelegramAdapter(deduplicator\=MessageDeduplicator())

msg \= adapter.adapt(self.\_make\_update())

assert msg.channel \== "telegram"

assert msg.user\_id \== "42"

assert msg.chat\_id \== "999"

assert msg.text \== "hello"

assert msg.is\_duplicate is False

def test\_adapt\_cleans\_text(self):

from ingress.telegram\_adapter import TelegramAdapter

from ingress.deduplicator import MessageDeduplicator

adapter \= TelegramAdapter(deduplicator\=MessageDeduplicator())

msg \= adapter.adapt(self.\_make\_update(text\="/start 查询PV"))

assert msg.text \== "查询PV"

assert msg.is\_cleaned is True

def test\_adapt\_detects\_duplicate(self):

from ingress.telegram\_adapter import TelegramAdapter

from ingress.deduplicator import MessageDeduplicator

dedup \= MessageDeduplicator()

adapter \= TelegramAdapter(deduplicator\=dedup)

update \= self.\_make\_update(update\_id\=42, message\_id\=100)

adapter.adapt(update)

msg2 \= adapter.adapt(update)

assert msg2.is\_duplicate is True

def test\_extract\_message\_id\_format(self):

from ingress.telegram\_adapter import TelegramAdapter

adapter \= TelegramAdapter()

msg\_id \= adapter.extract\_message\_id(self.\_make\_update(update\_id\=42, message\_id\=100))

assert msg\_id \== "telegram\_42\_100"

def test\_adapt\_empty\_text(self):

from ingress.telegram\_adapter import TelegramAdapter

from ingress.deduplicator import MessageDeduplicator

adapter \= TelegramAdapter(deduplicator\=MessageDeduplicator())

msg \= adapter.adapt(self.\_make\_update(text\=""))

assert msg.text \== ""