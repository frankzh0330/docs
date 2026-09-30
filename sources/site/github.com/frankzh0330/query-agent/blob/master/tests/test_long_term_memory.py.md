# Source: https://github.com/frankzh0330/query-agent/blob/master/tests/test_long_term_memory.py

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# test\_long\_term\_memory.py

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

[History](https://github.com/frankzh0330/query-agent/commits/master/tests/test_long_term_memory.py)

Open commit details

History

128 lines (99 loc) · 5.54 KB

## FilesExpand file tree

master

/

# test\_long\_term\_memory.py

Copy path

Top

## File metadata and controls

- Code

- Blame

128 lines (99 loc) · 5.54 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/tests/test_long_term_memory.py)

Copy raw file

Download raw file

Open symbols panel

Edit and raw actions

"""LongTermMemory 记忆注入测试""" import time class TestLongTermMemory: """memory/long\_term\_memory.py""" def \_write\_memory\_index(self, directory, filename, content): """辅助：创建 MEMORY.md 索引和引用的记忆文件""" directory.mkdir(parents=True, exist\_ok=True) memory\_file = directory / filename memory\_file.write\_text(content, encoding="utf-8") index = directory / "MEMORY.md" index.write\_text(f"- \[Test\]({filename})\\n", encoding="utf-8") def test\_no\_memory\_dir\_returns\_empty(self, long\_term\_memory, tmp\_path): # data\_path 已被 fixture 创建（mkdir parents=True），但里面没有内容 result = long\_term\_memory.load\_memory\_context(project\_id=55) assert result == "" def test\_empty\_global\_dir\_returns\_empty(self, long\_term\_memory, tmp\_path): global\_dir = tmp\_path / "memory" / "\_global" global\_dir.mkdir(parents=True) # 没有 MEMORY.md result = long\_term\_memory.load\_memory\_context() assert result == "" def test\_global\_memory\_loaded(self, long\_term\_memory, tmp\_path): global\_dir = tmp\_path / "memory" / "\_global" self.\_write\_memory\_index(global\_dir, "corrections.md", "Always use ROW for global queries") result = long\_term\_memory.load\_memory\_context() assert "Always use ROW for global queries" in result def test\_project\_memory\_loaded(self, long\_term\_memory, tmp\_path): # 先创建 global global\_dir = tmp\_path / "memory" / "\_global" self.\_write\_memory\_index(global\_dir, "common.md", "Global rule") # 再创建 project\_55 project\_dir = tmp\_path / "memory" / "project\_55" self.\_write\_memory\_index(project\_dir, "eu.md", "EU metrics only") result = long\_term\_memory.load\_memory\_context(project\_id=55) assert "Global rule" in result assert "EU metrics only" in result def test\_project\_memory\_not\_loaded\_for\_wrong\_id(self, long\_term\_memory, tmp\_path): project\_dir = tmp\_path / "memory" / "project\_55" self.\_write\_memory\_index(project\_dir, "eu.md", "EU metrics only") result = long\_term\_memory.load\_memory\_context(project\_id=60) assert "EU metrics only" not in result def test\_no\_project\_id\_loads\_only\_global(self, long\_term\_memory, tmp\_path): global\_dir = tmp\_path / "memory" / "\_global" self.\_write\_memory\_index(global\_dir, "common.md", "Global rule") project\_dir = tmp\_path / "memory" / "project\_55" self.\_write\_memory\_index(project\_dir, "eu.md", "EU metrics only") result = long\_term\_memory.load\_memory\_context(project\_id=None) assert "Global rule" in result assert "EU metrics only" not in result def test\_cache\_returns\_same\_result(self, long\_term\_memory, tmp\_path): global\_dir = tmp\_path / "memory" / "\_global" self.\_write\_memory\_index(global\_dir, "common.md", "Global rule") r1 = long\_term\_memory.load\_memory\_context() r2 = long\_term\_memory.load\_memory\_context() assert r1 == r2 def test\_cache\_invalidated\_on\_mtime\_change(self, long\_term\_memory, tmp\_path): global\_dir = tmp\_path / "memory" / "\_global" self.\_write\_memory\_index(global\_dir, "common.md", "Version 1") r1 = long\_term\_memory.load\_memory\_context() assert "Version 1" in r1 # 更新文件内容（mtime 变化） (global\_dir / "common.md").write\_text("Version 2", encoding="utf-8") r2 = long\_term\_memory.load\_memory\_context() assert "Version 2" in r2 assert "Version 1" not in r2 def test\_truncate\_at\_max\_lines(self, long\_term\_memory, tmp\_path): global\_dir = tmp\_path / "memory" / "\_global" long\_content = "\\n".join(\[f"Line {i}" for i in range(100)\]) self.\_write\_memory\_index(global\_dir, "big.md", long\_content) result = long\_term\_memory.load\_memory\_context() # \_MAX\_MEMORY\_LINES = 50 assert len(result.split("\\n")) <= 50 def test\_frontmatter\_stripped(self, long\_term\_memory, tmp\_path): global\_dir = tmp\_path / "memory" / "\_global" content = "---\\nname: test\\ntype: feedback\\n---\\nActual content here" self.\_write\_memory\_index(global\_dir, "note.md", content) result = long\_term\_memory.load\_memory\_context() assert "name: test" not in result assert "Actual content here" in result def test\_missing\_linked\_file\_skipped(self, long\_term\_memory, tmp\_path): global\_dir = tmp\_path / "memory" / "\_global" global\_dir.mkdir(parents=True, exist\_ok=True) # MEMORY.md 引用不存在的文件 (global\_dir / "MEMORY.md").write\_text("- \[Missing\](nonexistent.md)\\n", encoding="utf-8") result = long\_term\_memory.load\_memory\_context() # 不崩溃，返回空 assert result == "" def test\_selects\_relevant\_memory\_parts\_for\_query(self, long\_term\_memory, tmp\_path): project\_dir = tmp\_path / "memory" / "project\_55" project\_dir.mkdir(parents=True, exist\_ok=True) (project\_dir / "activation.md").write\_text("激活默认映射 activation\_success", encoding="utf-8") (project\_dir / "payment.md").write\_text("支付成功默认看 payment\_success", encoding="utf-8") (project\_dir / "MEMORY.md").write\_text( "- \[Activation\](activation.md)\\n- \[Payment\](payment.md)\\n", encoding="utf-8", ) result = long\_term\_memory.load\_memory\_context(project\_id=55, query\_text="看激活数据") assert "activation\_success" in result assert "payment\_success" not in result

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

"""LongTermMemory 记忆注入测试"""

import time

class TestLongTermMemory:

"""memory/long\_term\_memory.py"""

def \_write\_memory\_index(self, directory, filename, content):

"""辅助：创建 MEMORY.md 索引和引用的记忆文件"""

directory.mkdir(parents\=True, exist\_ok\=True)

memory\_file \= directory / filename

memory\_file.write\_text(content, encoding\="utf-8")

index \= directory / "MEMORY.md"

index.write\_text(f"- \[Test\]({filename})\\n", encoding\="utf-8")

def test\_no\_memory\_dir\_returns\_empty(self, long\_term\_memory, tmp\_path):

\# data\_path 已被 fixture 创建（mkdir parents=True），但里面没有内容

result \= long\_term\_memory.load\_memory\_context(project\_id\=55)

assert result \== ""

def test\_empty\_global\_dir\_returns\_empty(self, long\_term\_memory, tmp\_path):

global\_dir \= tmp\_path / "memory" / "\_global"

global\_dir.mkdir(parents\=True)

\# 没有 MEMORY.md

result \= long\_term\_memory.load\_memory\_context()

assert result \== ""

def test\_global\_memory\_loaded(self, long\_term\_memory, tmp\_path):

global\_dir \= tmp\_path / "memory" / "\_global"

self.\_write\_memory\_index(global\_dir, "corrections.md", "Always use ROW for global queries")

result \= long\_term\_memory.load\_memory\_context()

assert "Always use ROW for global queries" in result

def test\_project\_memory\_loaded(self, long\_term\_memory, tmp\_path):

\# 先创建 global

global\_dir \= tmp\_path / "memory" / "\_global"

self.\_write\_memory\_index(global\_dir, "common.md", "Global rule")

\# 再创建 project\_55

project\_dir \= tmp\_path / "memory" / "project\_55"

self.\_write\_memory\_index(project\_dir, "eu.md", "EU metrics only")

result \= long\_term\_memory.load\_memory\_context(project\_id\=55)

assert "Global rule" in result

assert "EU metrics only" in result

def test\_project\_memory\_not\_loaded\_for\_wrong\_id(self, long\_term\_memory, tmp\_path):

project\_dir \= tmp\_path / "memory" / "project\_55"

self.\_write\_memory\_index(project\_dir, "eu.md", "EU metrics only")

result \= long\_term\_memory.load\_memory\_context(project\_id\=60)

assert "EU metrics only" not in result

def test\_no\_project\_id\_loads\_only\_global(self, long\_term\_memory, tmp\_path):

global\_dir \= tmp\_path / "memory" / "\_global"

self.\_write\_memory\_index(global\_dir, "common.md", "Global rule")

project\_dir \= tmp\_path / "memory" / "project\_55"

self.\_write\_memory\_index(project\_dir, "eu.md", "EU metrics only")

result \= long\_term\_memory.load\_memory\_context(project\_id\=None)

assert "Global rule" in result

assert "EU metrics only" not in result

def test\_cache\_returns\_same\_result(self, long\_term\_memory, tmp\_path):

global\_dir \= tmp\_path / "memory" / "\_global"

self.\_write\_memory\_index(global\_dir, "common.md", "Global rule")

r1 \= long\_term\_memory.load\_memory\_context()

r2 \= long\_term\_memory.load\_memory\_context()

assert r1 \== r2

def test\_cache\_invalidated\_on\_mtime\_change(self, long\_term\_memory, tmp\_path):

global\_dir \= tmp\_path / "memory" / "\_global"

self.\_write\_memory\_index(global\_dir, "common.md", "Version 1")

r1 \= long\_term\_memory.load\_memory\_context()

assert "Version 1" in r1

\# 更新文件内容（mtime 变化）

(global\_dir / "common.md").write\_text("Version 2", encoding\="utf-8")

r2 \= long\_term\_memory.load\_memory\_context()

assert "Version 2" in r2

assert "Version 1" not in r2

def test\_truncate\_at\_max\_lines(self, long\_term\_memory, tmp\_path):

global\_dir \= tmp\_path / "memory" / "\_global"

long\_content \= "\\n".join(\[f"Line {i}" for i in range(100)\])

self.\_write\_memory\_index(global\_dir, "big.md", long\_content)

result \= long\_term\_memory.load\_memory\_context()

\# \_MAX\_MEMORY\_LINES = 50

assert len(result.split("\\n")) <= 50

def test\_frontmatter\_stripped(self, long\_term\_memory, tmp\_path):

global\_dir \= tmp\_path / "memory" / "\_global"

content \= "---\\nname: test\\ntype: feedback\\n\---\\nActual content here"

self.\_write\_memory\_index(global\_dir, "note.md", content)

result \= long\_term\_memory.load\_memory\_context()

assert "name: test" not in result

assert "Actual content here" in result

def test\_missing\_linked\_file\_skipped(self, long\_term\_memory, tmp\_path):

global\_dir \= tmp\_path / "memory" / "\_global"

global\_dir.mkdir(parents\=True, exist\_ok\=True)

\# MEMORY.md 引用不存在的文件

(global\_dir / "MEMORY.md").write\_text("- \[Missing\](nonexistent.md)\\n", encoding\="utf-8")

result \= long\_term\_memory.load\_memory\_context()

\# 不崩溃，返回空

assert result \== ""

def test\_selects\_relevant\_memory\_parts\_for\_query(self, long\_term\_memory, tmp\_path):

project\_dir \= tmp\_path / "memory" / "project\_55"

project\_dir.mkdir(parents\=True, exist\_ok\=True)

(project\_dir / "activation.md").write\_text("激活默认映射 activation\_success", encoding\="utf-8")

(project\_dir / "payment.md").write\_text("支付成功默认看 payment\_success", encoding\="utf-8")

(project\_dir / "MEMORY.md").write\_text(

"- \[Activation\](activation.md)\\n\- \[Payment\](payment.md)\\n",

encoding\="utf-8",

)

result \= long\_term\_memory.load\_memory\_context(project\_id\=55, query\_text\="看激活数据")

assert "activation\_success" in result

assert "payment\_success" not in result