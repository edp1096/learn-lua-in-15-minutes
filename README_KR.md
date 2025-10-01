# 몇 분만에 배우는 Lua

## 주석

```lua
-- 대시 두 개는 한 줄 주석을 시작합니다.

--[[
     대괄호 두 개를 추가하면
     여러 줄 주석이 됩니다.
--]]
```

---

## 1. 변수와 흐름 제어

### 숫자와 변수

```lua
num = 42  -- 모든 숫자는 double입니다.
-- 걱정하지 마세요. 64비트 double은 정확한 정수 값을 저장하기 위해 52비트를 가지고 있습니다.
-- 52비트 미만의 정수에는 기계 정밀도가 문제가 되지 않습니다.
```

### 문자열

```lua
s = 'walternate'  -- Python처럼 불변 문자열입니다.
t = "double-quotes are also fine"
u = [[ Double brackets
       start and end
       multi-line strings.]]
t = nil  -- t를 정의 해제합니다. Lua는 가비지 컬렉션이 있습니다.
```

### While 반복문

```lua
-- 블록은 do/end 같은 키워드로 표시됩니다:
while num < 50 do
  num = num + 1  -- ++나 += 같은 연산자는 없습니다.
end
```

### If 문

```lua
-- If 절:
if num > 40 then
  print('over 40')
elseif s ~= 'walternate' then  -- ~=는 같지 않음을 의미합니다.
  -- 동등 확인은 Python처럼 ==입니다. 문자열에도 사용 가능합니다.
  io.write('not over 40\n')  -- 기본적으로 stdout으로 출력됩니다.
else
  -- 변수는 기본적으로 전역입니다.
  thisIsGlobal = 5  -- 카멜 케이스가 일반적입니다.

  -- 변수를 지역으로 만드는 방법:
  local line = io.read()  -- 다음 stdin 줄을 읽습니다.

  -- 문자열 연결은 .. 연산자를 사용합니다:
  print('Winter is coming, ' .. line)
end
```

### Nil과 불리언 값

```lua
-- 정의되지 않은 변수는 nil을 반환합니다.
-- 이것은 오류가 아닙니다:
foo = anUnknownVariable  -- 이제 foo = nil입니다.

aBoolValue = false

-- nil과 false만 거짓입니다. 0과 ''는 참입니다!
if not aBoolValue then print('twas false') end

-- 'or'와 'and'는 단락 평가됩니다.
-- 이것은 C/js의 a?b:c 연산자와 유사합니다:
ans = aBoolValue and 'yes' or 'no'  --> 'no'
```

### For 반복문

```lua
karlSum = 0
for i = 1, 100 do  -- 범위는 양쪽 끝을 포함합니다.
  karlSum = karlSum + i
end

-- 카운트다운하려면 "100, 1, -1"을 범위로 사용하세요:
fredSum = 0
for j = 100, 1, -1 do fredSum = fredSum + j end

-- 일반적으로 범위는 begin, end[, step]입니다.
```

### Repeat-Until 반복문

```lua
-- 또 다른 반복문 구조:
repeat
  print('the way of the future')
  num = num - 1
until num == 0
```

---

## 2. 함수

### 기본 함수

```lua
function fib(n)
  if n < 2 then return 1 end
  return fib(n - 2) + fib(n - 1)
end
```

### 클로저와 익명 함수

```lua
-- 클로저와 익명 함수를 사용할 수 있습니다:
function adder(x)
  -- 반환된 함수는 adder가 호출될 때 생성되고,
  -- x의 값을 기억합니다:
  return function (y) return x + y end
end
a1 = adder(9)
a2 = adder(36)
print(a1(16))  --> 25
print(a2(64))  --> 100
```

### 다중 반환 값

```lua
-- 반환, 함수 호출, 할당은 모두
-- 길이가 일치하지 않는 리스트와 함께 작동합니다.
-- 일치하지 않는 수신자는 nil입니다.
-- 일치하지 않는 송신자는 버려집니다.

x, y, z = 1, 2, 3, 4
-- 이제 x = 1, y = 2, z = 3이고, 4는 버려집니다.

function bar(a, b, c)
  print(a, b, c)
  return 4, 8, 15, 16, 23, 42
end

x, y = bar('zaphod')  --> "zaphod  nil nil"을 출력합니다
-- 이제 x = 4, y = 8이고, 15..42 값들은 버려집니다.
```

### 함수 선언 스타일

```lua
-- 함수는 일급 객체이며, 지역/전역이 될 수 있습니다.
-- 다음은 같습니다:
function f(x) return x * x end
f = function (x) return x * x end

-- 그리고 다음도 같습니다:
local function g(x) return math.sin(x) end
local g; g  = function (x) return math.sin(x) end
-- 'local g' 선언은 g의 자기 참조를 가능하게 합니다.

-- 참고로 삼각 함수는 라디안으로 작동합니다.

-- 문자열 매개변수 하나를 가진 호출은 괄호가 필요 없습니다:
print 'hello'  -- 잘 작동합니다.
```

---

## 3. 테이블

### 개요

```lua
-- 테이블 = Lua의 유일한 복합 데이터 구조입니다.
--          연관 배열입니다.
-- PHP 배열이나 JS 객체와 유사하며,
-- 리스트로도 사용할 수 있는 해시 조회 딕셔너리입니다.
```

### 딕셔너리/맵으로서의 테이블

```lua
-- 딕셔너리/맵으로 테이블 사용하기:

-- 딕셔너리 리터럴은 기본적으로 문자열 키를 가집니다:
t = {key1 = 'value1', key2 = false}

-- 문자열 키는 JS처럼 점 표기법을 사용할 수 있습니다:
print(t.key1)  -- 'value1'을 출력합니다.
t.newKey = {}  -- 새로운 키/값 쌍을 추가합니다.
t.key2 = nil   -- 테이블에서 key2를 제거합니다.

-- nil이 아닌 모든 값을 키로 사용하는 리터럴 표기법:
u = {['@!#'] = 'qbert', [{}] = 1729, [6.28] = 'tau'}
print(u[6.28])  -- "tau"를 출력합니다

-- 키 매칭은 기본적으로 숫자와 문자열에는 값으로,
-- 테이블에는 동일성으로 이루어집니다.
a = u['@!#']  -- 이제 a = 'qbert'입니다.
b = u[{}]     -- 1729를 예상할 수 있지만, nil입니다:
-- b = nil입니다. 조회가 실패하기 때문입니다.
-- 사용한 키가 원래 값을 저장할 때 사용한 객체와
-- 동일한 객체가 아니기 때문에 실패합니다.
-- 따라서 문자열과 숫자가 더 이식 가능한 키입니다.

-- 테이블 매개변수 하나를 가진 함수 호출은 괄호가 필요 없습니다:
function h(x) print(x.key1) end
h{key1 = 'Sonmi~451'}  -- 'Sonmi~451'을 출력합니다.

for key, val in pairs(u) do  -- 테이블 반복.
  print(key, val)
end

-- _G는 모든 전역 변수의 특별한 테이블입니다.
print(_G['_G'] == _G)  -- 'true'를 출력합니다.
```

### 리스트/배열로서의 테이블

```lua
-- 리스트/배열로 테이블 사용하기:

-- 리스트 리터럴은 암시적으로 정수 키를 설정합니다:
v = {'value1', 'value2', 1.21, 'gigawatts'}
for i = 1, #v do  -- #v는 리스트 v의 크기입니다.
  print(v[i])  -- 인덱스는 1에서 시작합니다!! 정말 놀랍죠!
end
-- '리스트'는 실제 타입이 아닙니다. v는 단지
-- 연속된 정수 키를 가진 테이블이며, 리스트로 취급됩니다.
```

---

## 3.1 메타테이블과 메타메서드

### 기본 메타테이블

```lua
-- 테이블은 메타테이블을 가질 수 있으며,
-- 연산자 오버로딩과 유사한 동작을 제공합니다.
-- 나중에 메타테이블이 JS 프로토타입과 유사한 동작을
-- 지원하는 방법을 볼 것입니다.

f1 = {a = 1, b = 2}  -- 분수 a/b를 나타냅니다.
f2 = {a = 2, b = 3}

-- 이것은 실패합니다:
-- s = f1 + f2

metafraction = {}
function metafraction.__add(f1, f2)
  sum = {}
  sum.b = f1.b * f2.b
  sum.a = f1.a * f2.b + f2.a * f1.b
  return sum
end

setmetatable(f1, metafraction)
setmetatable(f2, metafraction)

s = f1 + f2  -- f1의 메타테이블에서 __add(f1, f2)를 호출합니다

-- f1, f2는 메타테이블에 대한 키를 가지고 있지 않습니다.
-- JS의 프로토타입과 달리, getmetatable(f1)처럼
-- 검색해야 합니다. 메타테이블은 일반 테이블이며,
-- Lua가 알고 있는 키(__add 같은)를 가집니다.

-- 그러나 다음 줄은 s가 메타테이블을 가지지 않으므로 실패합니다:
-- t = s + s
-- 아래 주어진 클래스와 유사한 패턴이 이것을 고칠 것입니다.
```

### __index 메타메서드

```lua
-- 메타테이블의 __index는 점 조회를 오버로드합니다:
defaultFavs = {animal = 'gru', food = 'donuts'}
myFavs = {food = 'pizza'}
setmetatable(myFavs, {__index = defaultFavs})
eatenBy = myFavs.animal  -- 작동합니다! 메타테이블 덕분입니다

-- 실패한 직접 테이블 조회는 메타테이블의 __index 값을 사용하여
-- 재시도하며, 이것은 재귀적입니다.

-- __index 값은 더 사용자 정의된 조회를 위해
-- function(tbl, key)가 될 수도 있습니다.
```

### 메타메서드 전체 목록

```lua
-- __index, add 등의 값을 메타메서드라고 합니다.
-- 전체 목록. 여기서 a는 메타메서드를 가진 테이블입니다.

-- __add(a, b)                     a + b용
-- __sub(a, b)                     a - b용
-- __mul(a, b)                     a * b용
-- __div(a, b)                     a / b용
-- __mod(a, b)                     a % b용
-- __pow(a, b)                     a ^ b용
-- __unm(a)                        -a용
-- __concat(a, b)                  a .. b용
-- __len(a)                        #a용
-- __eq(a, b)                      a == b용
-- __lt(a, b)                      a < b용
-- __le(a, b)                      a <= b용
-- __index(a, b)  <fn 또는 테이블>  a.b용
-- __newindex(a, b, c)             a.b = c용
-- __call(a, ...)                  a(...)용
```

---

## 3.2 클래스형 테이블과 상속

### 클래스 만들기

```lua
-- 클래스는 내장되어 있지 않습니다. 테이블과 메타테이블을
-- 사용하여 만드는 다양한 방법이 있습니다.

-- 이 예제에 대한 설명은 아래에 있습니다.

Dog = {}                                   -- 1.

function Dog:new()                         -- 2.
  newObj = {sound = 'woof'}                -- 3.
  self.__index = self                      -- 4.
  return setmetatable(newObj, self)        -- 5.
end

function Dog:makeSound()                   -- 6.
  print('I say ' .. self.sound)
end

mrDog = Dog:new()                          -- 7.
mrDog:makeSound()  -- 'I say woof'         -- 8.

-- 1. Dog는 클래스처럼 작동합니다. 실제로는 테이블입니다.
-- 2. function tablename:fn(...)은
--    function tablename.fn(self, ...)와 같습니다.
--    :는 단지 self라는 첫 번째 인자를 추가합니다.
--    self가 값을 얻는 방법은 아래 7과 8을 참조하세요.
-- 3. newObj는 Dog 클래스의 인스턴스가 될 것입니다.
-- 4. self = 인스턴스화되는 클래스입니다. 종종
--    self = Dog이지만, 상속은 이것을 변경할 수 있습니다.
--    newObj의 메타테이블과 self의 __index를 모두
--    self로 설정하면 newObj가 self의 함수를 얻습니다.
-- 5. 알림: setmetatable은 첫 번째 인자를 반환합니다.
-- 6. :는 2와 같이 작동하지만, 이번에는
--    self가 클래스 대신 인스턴스일 것으로 예상합니다.
-- 7. Dog.new(Dog)와 같으므로, new()에서 self = Dog입니다.
-- 8. mrDog.makeSound(mrDog)와 같습니다. self = mrDog입니다.
```

### 상속 예제

```lua
-- 상속 예제:

LoudDog = Dog:new()                           -- 1.

function LoudDog:makeSound()
  s = self.sound .. ' '                       -- 2.
  print(s .. s .. s)
end

seymour = LoudDog:new()                       -- 3.
seymour:makeSound()  -- 'woof woof woof'      -- 4.

-- 1. LoudDog는 Dog의 메서드와 변수를 가져옵니다.
-- 2. self는 new()에서 'sound' 키를 가집니다. 3을 참조하세요.
-- 3. LoudDog.new(LoudDog)와 같으며,
--    Dog.new(LoudDog)로 변환됩니다.
--    LoudDog에는 'new' 키가 없지만,
--    메타테이블에 __index = Dog가 있기 때문입니다.
--    결과: seymour의 메타테이블은 LoudDog이고,
--    LoudDog.__index = LoudDog입니다. 따라서 seymour.key는
--    주어진 키를 가진 첫 번째 테이블인
--    seymour.key, LoudDog.key, Dog.key가 됩니다.
-- 4. 'makeSound' 키는 LoudDog에서 발견됩니다.
--    이것은 LoudDog.makeSound(seymour)와 같습니다.

-- 필요한 경우, 서브클래스의 new()는 베이스와 비슷합니다:
function LoudDog:new()
  newObj = {}
  -- newObj 설정
  self.__index = self
  return setmetatable(newObj, self)
end
```

---

## 4. 모듈

### 모듈 패턴

```lua
--[[ 이 섹션을 주석 처리하여
--   이 스크립트의 나머지 부분이 실행 가능하도록 합니다.
-- mod.lua 파일이 다음과 같다고 가정합니다:
local M = {}

local function sayMyName()
  print('Hrunkner')
end

function M.sayHello()
  print('Why hello there')
  sayMyName()
end

return M

-- 다른 파일에서 mod.lua의 기능을 사용할 수 있습니다:
local mod = require('mod')  -- mod.lua 파일을 실행합니다.

-- require는 모듈을 포함하는 표준 방법입니다.
-- require는 다음처럼 작동합니다: (캐시되지 않은 경우; 아래 참조)
local mod = (function ()
  <mod.lua의 내용>
end)()
-- mod.lua가 함수 본문과 같아서,
-- mod.lua 내부의 지역 변수가 외부에서 보이지 않습니다.

-- 여기서 mod = mod.lua의 M이므로 작동합니다:
mod.sayHello()  -- Hrunkner에게 인사합니다.

-- 이것은 잘못되었습니다. sayMyName은 mod.lua에만 존재합니다:
mod.sayMyName()  -- 오류

-- require의 반환 값은 캐시되므로 파일은
-- 여러 번 require되어도 최대 한 번만 실행됩니다.

-- mod2.lua가 "print('Hi!')"를 포함한다고 가정합니다.
local a = require('mod2')  -- Hi!를 출력합니다
local b = require('mod2')  -- 출력하지 않습니다. a=b입니다.

-- dofile은 캐싱 없이 require와 같습니다:
dofile('mod2.lua')  --> Hi!
dofile('mod2.lua')  --> Hi! (다시 실행합니다)

-- loadfile은 lua 파일을 로드하지만 아직 실행하지 않습니다.
f = loadfile('mod2.lua')  -- f()를 호출하여 실행합니다.

-- loadstring은 문자열용 loadfile입니다.
g = loadstring('print(343)')  -- 함수를 반환합니다.
g()  -- 343을 출력합니다. 이전에는 아무것도 출력되지 않았습니다.

--]]
```

---

## 5. 참고자료

```lua
--[[

저는 Löve 2D 게임 엔진으로 게임을 만들기 위해
Lua를 배우는 것에 흥분했습니다. 그것이 이유입니다.

저는 BlackBulletIV의 Lua for programmers로 시작했습니다.
다음으로 공식 Programming in Lua 책을 읽었습니다.
그것이 방법입니다.

lua-users.org의 Lua 짧은 참고자료를
확인하는 것이 도움이 될 수 있습니다.

다루지 않은 주요 주제는 표준 라이브러리입니다:
 * string 라이브러리
 * table 라이브러리
 * math 라이브러리
 * io 라이브러리
 * os 라이브러리

참고로, 이 전체 파일은 유효한 Lua입니다.
learn.lua로 저장하고 "lua learn.lua"로 실행하세요!

이것은 처음에 tylerneylon.com을 위해 작성되었습니다.
github gist로도 이용 가능합니다. 이것과 같은 스타일로
다른 언어에 대한 튜토리얼은 여기에 있습니다:

https://learnxinyminutes.com/

Lua를 즐기세요!

--]]
```

출처: https://tylerneylon.com/a/learn-lua
