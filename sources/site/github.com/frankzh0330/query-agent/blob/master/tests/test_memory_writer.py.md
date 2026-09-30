# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_memory_writer.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_memory\_writer.py

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[feat: add LLM-driven memory writer and replace print() with logger.de…](https://github.com/frankzh0330/query-agent/commit/c6d596c4549efeaba4556e94aa2afef33305403f)

Open commit details

Apr 14, 2026

[c6d596c](https://github.com/frankzh0330/query-agent/commit/c6d596c4549efeaba4556e94aa2afef33305403f) · Apr 14, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_memory_writer.py)

Open commit details

History

178 lines (143 loc) · 7.06 KB

## FilesExpand file tree

master

/

# test\_memory\_writer.py

Copy path

Top

## File metadata and controls

- Code

- Blame

178 lines (143 loc) · 7.06 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_memory_writer.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""MemoryWriter — LLM 驱动记忆写入测试""" import json from unittest import mock import pytest class TestMemoryWriter: """memory/memory\_writer.py""" def test\_ensure\_index\_creates\_memory\_md(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) writer.\_ensure\_index(project\_dir) index = project\_dir / "MEMORY.md" assert index.exists() assert "auto\_learned.md" in index.read\_text() def test\_ensure\_index\_appends\_to\_existing(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) (project\_dir / "MEMORY.md").write\_text("- \[Manual\](manual.md)\\n") writer.\_ensure\_index(project\_dir) content = (project\_dir / "MEMORY.md").read\_text() assert "manual.md" in content assert "auto\_learned.md" in content def test\_ensure\_index\_idempotent(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) writer.\_ensure\_index(project\_dir) writer.\_ensure\_index(project\_dir) content = (project\_dir / "MEMORY.md").read\_text() assert content.count("auto\_learned.md") == 1 def test\_append\_memory\_creates\_file(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) writer.\_append\_memory(project\_dir, "correction", "purchase 应匹配 payment\_submit") auto = project\_dir / "auto\_learned.md" assert auto.exists() assert "purchase 应匹配 payment\_submit" in auto.read\_text() assert "\[correction\]" in auto.read\_text() def test\_append\_memory\_appends(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) writer.\_append\_memory(project\_dir, "correction", "第一条") writer.\_append\_memory(project\_dir, "preference", "第二条") lines = (project\_dir / "auto\_learned.md").read\_text().strip().split("\\n") assert len(lines) == 2 def test\_is\_duplicate\_detects\_similar(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) writer.\_append\_memory(project\_dir, "correction", "purchase 应匹配 payment\_submit") assert writer.\_is\_duplicate(project\_dir, "purchase 应匹配 payment\_submit") assert not writer.\_is\_duplicate(project\_dir, "完全不同的内容") def test\_is\_duplicate\_no\_file(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) assert not writer.\_is\_duplicate(project\_dir, "anything") def test\_trim\_limits\_lines(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) # 写入超过限制的行数 for i in range(40): writer.\_append\_memory(project\_dir, "test", f"记忆 {i}") lines = (project\_dir / "auto\_learned.md").read\_text().strip().split("\\n") assert len(lines) <= 30 # 最新的保留 assert "记忆 39" in lines\[-1\] def test\_read\_existing(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True) assert writer.\_read\_existing(project\_dir) == "" writer.\_append\_memory(project\_dir, "test", "内容") assert "内容" in writer.\_read\_existing(project\_dir) @pytest.mark.asyncio async def test\_maybe\_save\_no\_save\_when\_llm\_says\_no(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) # mock \_judge 返回 should\_save=false writer.\_judge = mock.AsyncMock(return\_value={"should\_save": False}) await writer.maybe\_save( project\_id=55, user\_query="德国的PV", extraction={"event": "app\_launch"}, resolver\_explain={"event": {"score": 100}}, current\_state={"event": "app\_launch"}, ) auto = tmp\_path / "memory" / "project\_55" / "auto\_learned.md" assert not auto.exists() @pytest.mark.asyncio async def test\_maybe\_save\_writes\_when\_llm\_says\_yes(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) writer.\_judge = mock.AsyncMock(return\_value={ "should\_save": True, "category": "preference", "content": "用户偏好查询UV而非PV", }) await writer.maybe\_save( project\_id=55, user\_query="我要看UV", extraction={"metric": "uv"}, resolver\_explain={"metric": {"score": 100, "value": "uv"}}, current\_state={"metric": "uv"}, ) auto = tmp\_path / "memory" / "project\_55" / "auto\_learned.md" assert auto.exists() content = auto.read\_text() assert "用户偏好查询UV而非PV" in content assert "\[preference\]" in content # 索引文件也应被创建 index = tmp\_path / "memory" / "project\_55" / "MEMORY.md" assert index.exists() assert "auto\_learned.md" in index.read\_text() @pytest.mark.asyncio async def test\_maybe\_save\_skips\_duplicate(self, tmp\_path): from memory.memory\_writer import MemoryWriter writer = MemoryWriter(data\_path=str(tmp\_path / "memory")) # 先写入一条 writer.\_judge = mock.AsyncMock(return\_value={ "should\_save": True, "category": "correction", "content": "purchase 应匹配 payment\_submit", }) await writer.maybe\_save(55, "purchase", {}, {}, {}) # 再次返回相同内容 → 应被去重跳过 await writer.maybe\_save(55, "purchase", {}, {}, {}) auto = tmp\_path / "memory" / "project\_55" / "auto\_learned.md" lines = \[l for l in auto.read\_text().strip().split("\\n") if l.strip()\] assert len(lines) == 1 # 不重复写入

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

170

171

172

173

174

175

176

177

178

"""MemoryWriter — LLM 驱动记忆写入测试"""

import json

from unittest import mock

import pytest

class TestMemoryWriter:

"""memory/memory\_writer.py"""

def test\_ensure\_index\_creates\_memory\_md(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

writer.\_ensure\_index(project\_dir)

index \= project\_dir / "MEMORY.md"

assert index.exists()

assert "auto\_learned.md" in index.read\_text()

def test\_ensure\_index\_appends\_to\_existing(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

(project\_dir / "MEMORY.md").write\_text("- \[Manual\](manual.md)\\n")

writer.\_ensure\_index(project\_dir)

content \= (project\_dir / "MEMORY.md").read\_text()

assert "manual.md" in content

assert "auto\_learned.md" in content

def test\_ensure\_index\_idempotent(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

writer.\_ensure\_index(project\_dir)

writer.\_ensure\_index(project\_dir)

content \= (project\_dir / "MEMORY.md").read\_text()

assert content.count("auto\_learned.md") \== 1

def test\_append\_memory\_creates\_file(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

writer.\_append\_memory(project\_dir, "correction", "purchase 应匹配 payment\_submit")

auto \= project\_dir / "auto\_learned.md"

assert auto.exists()

assert "purchase 应匹配 payment\_submit" in auto.read\_text()

assert "\[correction\]" in auto.read\_text()

def test\_append\_memory\_appends(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

writer.\_append\_memory(project\_dir, "correction", "第一条")

writer.\_append\_memory(project\_dir, "preference", "第二条")

lines \= (project\_dir / "auto\_learned.md").read\_text().strip().split("\\n")

assert len(lines) \== 2

def test\_is\_duplicate\_detects\_similar(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

writer.\_append\_memory(project\_dir, "correction", "purchase 应匹配 payment\_submit")

assert writer.\_is\_duplicate(project\_dir, "purchase 应匹配 payment\_submit")

assert not writer.\_is\_duplicate(project\_dir, "完全不同的内容")

def test\_is\_duplicate\_no\_file(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

assert not writer.\_is\_duplicate(project\_dir, "anything")

def test\_trim\_limits\_lines(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

\# 写入超过限制的行数

for i in range(40):

writer.\_append\_memory(project\_dir, "test", f"记忆 {i}")

lines \= (project\_dir / "auto\_learned.md").read\_text().strip().split("\\n")

assert len(lines) <= 30

\# 最新的保留

assert "记忆 39" in lines\[\-1\]

def test\_read\_existing(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True)

assert writer.\_read\_existing(project\_dir) \== ""

writer.\_append\_memory(project\_dir, "test", "内容")

assert "内容" in writer.\_read\_existing(project\_dir)

@pytest.mark.asyncio

async def test\_maybe\_save\_no\_save\_when\_llm\_says\_no(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

\# mock \_judge 返回 should\_save=false

writer.\_judge \= mock.AsyncMock(return\_value\={"should\_save": False})

await writer.maybe\_save(

project\_id\=55,

user\_query\="德国的PV",

extraction\={"event": "app\_launch"},

resolver\_explain\={"event": {"score": 100}},

current\_state\={"event": "app\_launch"},

)

auto \= tmp\_path / "memory" / "project\_55" / "auto\_learned.md"

assert not auto.exists()

@pytest.mark.asyncio

async def test\_maybe\_save\_writes\_when\_llm\_says\_yes(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

writer.\_judge \= mock.AsyncMock(return\_value\={

"should\_save": True,

"category": "preference",

"content": "用户偏好查询UV而非PV",

})

await writer.maybe\_save(

project\_id\=55,

user\_query\="我要看UV",

extraction\={"metric": "uv"},

resolver\_explain\={"metric": {"score": 100, "value": "uv"}},

current\_state\={"metric": "uv"},

)

auto \= tmp\_path / "memory" / "project\_55" / "auto\_learned.md"

assert auto.exists()

content \= auto.read\_text()

assert "用户偏好查询UV而非PV" in content

assert "\[preference\]" in content

\# 索引文件也应被创建

index \= tmp\_path / "memory" / "project\_55" / "MEMORY.md"

assert index.exists()

assert "auto\_learned.md" in index.read\_text()

@pytest.mark.asyncio

async def test\_maybe\_save\_skips\_duplicate(self, tmp\_path):

from memory.memory\_writer import MemoryWriter

writer \= MemoryWriter(data\_path\=str(tmp\_path / "memory"))

\# 先写入一条

writer.\_judge \= mock.AsyncMock(return\_value\={

"should\_save": True,

"category": "correction",

"content": "purchase 应匹配 payment\_submit",

})

await writer.maybe\_save(55, "purchase", {}, {}, {})

\# 再次返回相同内容 → 应被去重跳过

await writer.maybe\_save(55, "purchase", {}, {}, {})

auto \= tmp\_path / "memory" / "project\_55" / "auto\_learned.md"

lines \= \[l for l in auto.read\_text().strip().split("\\n") if l.strip()\]

assert len(lines) \== 1 \# 不重复写入