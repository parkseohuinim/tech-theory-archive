# Python 기본 개념과 이론

## 목차

- [1. 기본 개념](#1-기본-개념)
  - [Object(객체)와 Data Type(자료형)](#object객체와-data-type자료형)
  - [Comment(주석)](#comment주석)
  - [Object(객체)의 상호 작용 및 이름](#object객체의-상호-작용-및-이름)
  - [Function(함수) 사용법](#function함수-사용법)
  - [Flow Control(흐름 제어)](#flow-control흐름-제어)
  - [Object(객체) 사용](#object객체-사용)
  - [User Defined Data Type(사용자 정의 자료형)](#user-defined-data-type사용자-정의-자료형)
  - [Module](#module)
- [2. 숫자와 자료형](#2-숫자와-자료형)
  - [Python에서의 숫자](#python에서의-숫자)
- [3. 변수](#3-변수)
  - [변수 네이밍](#변수-네이밍)
  - [Assignment Operators(대입 연산자)](#assignment-operators대입-연산자)
  - [Dynamic Typing(동적 자료형 결정)](#dynamic-typing동적-자료형-결정)
  - [Mutability(가변성)과 Immutability(불변성)](#mutability가변성과-immutability불변성)
- [4. Strings(문자열)](#4-strings문자열)
  - [string literal](#string-literal)
  - [string 출력](#string-출력)
  - [indexing(인덱싱)과 slicing(슬라이싱)](#indexing인덱싱과-slicing슬라이싱)

## 1. 기본 개념

### Object(객체)와 Data Type(자료형)

- python에서는 모든 것이 **object(객체)**
- 메모리에 저장된 모든 데이터는 object라는 개념
- object는 두 가지로 구성: **value(값)**와 **무엇을 할 수 있는지(기능)**
  - `10`은 정수형 object
    
     값은 10
     
     가능한 행위는 덧셈, 뺄셈 등
     
     예: `10 + 5` → 덧셈 기능 수행
- object가 어떤 데이터 값으로 이루어져 있으며, 어떤 행위를 할 수 있는지를 정해놓은 것 → **data type(자료형)**
- python에서는 다양한 data type을 기본적으로 제공

  - `int`, `float`, `str`, `list`, `dict`, `bool` 등

```python
# int
type(5)

# float
type(3.14)

# python에서 문자열을 만들 때 겹따옴표(") 또는 홑따옴표(') 사용 가능
# Black은 겹따옴표(") 사용을 기본 스타일로 강제 변환함
# str
type('bye')
```

---

### Comment(주석)

- 코드 실행에 영향을 주지 않고, 사람을 위한 설명
- 코드를 읽는 개발자 혹은 미래의 나에게 정보를 남기는 수단
- 주석은 **python 인터프리터가 무시**
- python은 C나 Java처럼 공식적인 여러 줄 주석 문법이 없음

#### 방법 1: #을 여러 줄에 반복

```python
# 이것은 주석
x = 5 # 변수 x에 5를 할당
# 여러 줄 주석처럼 보이나
# 각각의 한 줄 주석임
# 이것이 가장 안전하고 추천되는 방식
```

- 진짜 주석 처리되며, Black과 같은 모든 formatter와 분석기에서 권장하는 방식

#### 방법 2: `""" """` 또는 `''' '''` 사용

```python
"""
이건 주석처럼 보이지만
실제로는 문자열 리터럴
"""
```

- python 입장에선 이것도 하나의 문자열
- 변수에 할당되지 않으면 실행은 안되지만, 메모리에 존재
- 이름 없는 문자열이 코드 중간에 있는 것
- 코드 설명용 주석처럼 쓰이나, 진짜 주석은 아님

#### Docstring과 주석의 차이

```python
def hello():
    """
    이 함수는 인사말을 출력
    """
    print("Hello")
  
print(hello.__doc__) # 이 함수는 인사말을 출력
```

- 함수, 클래스, 모듈에 대한 설명을 위해 사용
- `__doc__` 속성으로 조회 가능
- 실제로는 문자열임

---

### Object(객체)의 상호 작용 및 이름

`5 + 7`

- `5`, `7` → 각각 정수 값을 갖는 object 생성 지시
- `+` → 두 정수 object의 값을 더해 **새로운 object** 생성 (값: `12`)
- `5`, `7`처럼 직접 입력한 값 → **literal(리터럴)**
- `+`, `-` 등 연산 명령 기호 → **operator(연산자)**

- object에 이름을 붙여주면 이름을 이용해 객체를 계속 사용할 수 있음
- 객체의 이름을 **variable(변수)**라고 함

```python
# 5 + 7
a = 5 + 7

# 등호(=)는 수학의 '같다'는 의미와 다름
# python에서는 assignment operator(대입 연산자)라고 부름
# 전산학에서는 화살표(<-)를 사용하기도 함 ex) a <- 5 + 7
# 5 + 7의 결과로 새로운 12 object를 만들고 a라는 이름을 붙임

# 여러 객체에 같은 이름을 쓸 수 없기 때문에 a는 a + 1을 통해서 새로 만들어진 객체만을 가리킬 수 있음
a = 5
print(a) # 5
print(id(a)) # 4367674464

a = a + 1

print(a) # 6
print(id(a)) # 4367674496
```

---

### Function(함수) 사용법

- 함수는 필요할 때, 사용할 수 있도록 코드를 묶어놓은 것을 의미
- 코드를 재사용하기 편하도록 '기능'별로 묶어서 분류
- 입력과 출력을 고민해야함

```python
# 기본 입출력 함수
name = input("당신의 이름은? ")
print(name)

# 함수를 직접 만들기
def your_name():
    name = input("당신의 이름은? ")
    print(name)

# 함수 호출
your_name()
```

---

### Flow Control(흐름 제어)

- 조건에 따라 실행 결과가 달라지도록 흐름을 제어할 수 있음
- 원하는 부분을 원하는 횟수만큼 반복할 수 있으며, 매번 변화를 줄 수 있음

```python
a = 5

if a > 3:
    print("3 초과") # 이 문구를 출력
else:
    print("3 이하")
  
for b in range(4):
    print(b)
# 0
# 1
# 2
# 3
```

---

### Object(객체) 사용

- object의 이름 뒤에 `.`(점)을 찍어 **attribute(속성)**와 **method(메서드)**를 사용

```python
a = int(1) # a = 1
b = a.__add__(1) # a에 1을 더하기
print(b) # 2
```

---

### User Defined Data Type(사용자 정의 자료형)

- `class` keyword를 사용해 새로운 자료형을 만들 수 있음

```python
class MyInt:
    def __init__(self, value):
        self.value = value  # instance variable

    def print(self):  # method
        print("내가 만든 정수형", self.value)

a = MyInt(135)
a.print()
```

---

### Module

- `import` keyword를 사용해 외부의 module을 가져와서 편리하게 사용 가능
- 여러 module 모음집을 package라고 함

```python
import os

type(os) # module
os.getcwd()  # os module의 getcwd()라는 메서드를 호출
```

---

## 2. 숫자와 자료형

### Python에서의 숫자

| 자료형    | 종류                         | 예시            | 특징                                               |
| --------- | ---------------------------- | --------------- | -------------------------------------------------- |
| `int`     | 정수형(소수점 없는 숫자)     | `10`, `-3`, `0` | 크기 제한 없음                                     |
| `float`   | 부동소수점(소수 포함 숫자)   | `3.14`, `-0.5`  | 부동소수점 오차 존재 가능                          |
| `complex` | 복소수형(실수부+허수부 구성) | `3+4j`, `1j`    | `j`는 허수 단위, 실수부와 허수부는 각각 float 사용 |
| `bool`    | 논리형, 숫자처럼 동작 가능   | `True`, `False` | 내부적으로 `1`, `0`처럼 처리                       |

- 부동소수점 표현으로 실수는 부호, 유효숫자, 지수 세 가지로 나누어서 저장
  - 십진수 예시) `-3.141592` = `-1` X `3141592` X `10⁻⁶` 
- 아주 큰 숫자를 더하면 정밀도 문제로 작은 숫자가 무시
  - `500000000000000000000.0` + `5.0` = `5e+20`
- 컴퓨터는 내부적으로 2진수로 계산을 하기 때문에, 소수점 아래 자리의 정밀도가 떨어짐
  - `0.4` X `0.4` = `0.16000000000000003`
- 연산자 우선순위는 간단히 괄호 > 거듭제곱 > 곱셈, 나눗셈 > 덧셈, 뺄셈
  - `(7 + 11)` X `4` =  `72`

```python
type(123) # int

# 덧셈
1 + 4 # 5

# 뺄셈
2 - 4 # -2

# 곱셈
3 * 5 # 15

# 나누기
4 / 3 # 1.3333333333333333

# 나누기
6 / 3 # 2.0

# 0으로 나누기
5 / 0 # ZeroDivisionError

# 나머지 구하기
6 % 4 # 2

# 절대값
abs(-7) # 7

# 몫과 나머지 한꺼번에 구하기
divmod(7, 2) # (3, 1)

# 거듭제곱 연산자(Power operator)
2 ** 3 # 8

# 거듭제곱 함수(Power function)
pow(2, 3) # 8

# 숫자 리터럴 사이에 _(언더스코어)를 삽입하면 무시
a = 2_000_000 # 2000000

# Negate
a = 123
print(-a) # -123

type(1.0) # float

# 과학적 표기법(지수 표기법)
# e뒤의 나오는 숫자는 10의 얼마나 제곱을 해야하는지를 뜻함
3.1415e2 # 314.15

type(3.1415e2) # float

# float과 int가 같이 계산(결과는 float)
2.0 + 1 # 3.0

# 소수점 이하 버림 나누기(결과는 float)
2.1 // 1 # 2.0

# 나머지(결과는 float)
8.0 % 3 # 2.0

# 실수를 정수로 변환
int(9.9999) # 9

# 정수를 실수로 변환
float(99) # 99.0

# 복소수(Complex) 사용 예: 실수부와 허수부(j를 붙임)
1 + 2j
```

---

## 3. 변수

### 변수 네이밍

- 숫자로 시작할 수 없음
- 중간에 빈 칸을 넣을 수 없음
- 기호들을 사용할 수 없음
- 대소문자를 구분
- 파이썬의 예약어로는 사용할 수 없음
- 예약어는 `import keyword`하여, `print(keyword.kwlist)`로 확인 가능
- 알파벳(abcde...), 숫자(1234...), 그리고 `_`(언더스코어)를 사용하며, 한글 사용은 권장하지 않음
- 변수명은 너무 짧거나 의미 없는 이름 지양
  - `a`, `b` 대신 의미 있는 이름 사용(`count`, `user_name`)
- 변수명은 대개 명사, 함수는 동사 느낌으로 작성
  - 변수: `file_path`, `user_age`
  - 함수: `get_data()`, `print_result()`
- 상수는 모두 대문자 + Snake case 사용
  - `PI = 3.14`, `MAX_RETRY = 5`
- module/파일명은 소문자 + Snake case 권장
  - `user_modle.py`, `config_loader.py`
- python의 표준 스타일 권고안(PEP8)에서는 함수와 변수의 이름에 스네이크 케이스 방식을 권장하며, class에서는 Camel case 권장

```python
# snake case 규칙
snake_case

# camel case 규칙
CamelCase
```

---

### Assignment Operators(대입 연산자)

- 수학에서 대입과 달리 python에서의 `=`(대입 연산자)는 메모리에 있는 object에 이름표를 달아주는 것에 가까움
- 운영체제가 관리하는 메모리의 어딘가에서 찾아서 사용할 수 있게끔 도와줌

```python
a = 7
print(id(a)) # 4366970016

a = a + a
print(id(a)) # 4366970240

# 정의하지 않고는 변수 사용이 불가
print(b) # NameError: name 'b' is not defined

# 여러 변수를 동시에 정의할 수 있음
a, b, c = 1, 3, 5
d, e, f = -1, 16.0, "Good"
```

---

### Dynamic Typing(동적 자료형 결정)

- python에서는 같은 변수의 이름을 여러가지 자료형의 object에게 바꿔가면서 할당할 수 있음
- 자료형을 고정해놓는 것이 아닌 동적으로 그때그때 바꾸는 것을 Dynamic Typing이라고 함
- C언어에서는 `int a;` 같이 선언하면 자료형을 바꿀 수 없는 것과 상반됨

```python
a = 5
print(type(a)) # <class 'int'>

a = 5.1
print(type(a)) # <class 'float'>

a = "Good Bye!"
print(type(a)) # <class 'str'>
```

---

### Mutability(가변성)과 Immutability(불변성)

|      | Immutable                              | Mutable               |
| ---- | -------------------------------------- | --------------------- |
| Type | `int`, `float`, `bool`, `str`, `tuple` | `List`, `dict`, `set` |

```python
# 같은 객체에 이름표 두개를 발급할 수 있음
a = 777
b = a

print(id(a), a) # 4628210224 777
print(id(b), b) # 4628210224 777

# int 자료형은 객체의 값을 바꿀 수 없는 자료형이기 때문에 새로운 값을 갖는 객체를 새로 만듬
# b의 값을 바꾸면 id가 달라짐
b = b + 1
print(id(a), a) # 4628210224 777
print(id(b), b) # 4628122032 778
```

---

## 4. Strings(문자열)

- `string`은 순서대로 나열되어있는 문자들을 다루기에 편한 자료형
- 글자를 여러 개 담을 수 있기에 container 자료형 중 하나로 분류
- container 자료형 중에서도 element들이 순서대로 나열되어 있는 점에서 sequence로 다시 분류
- sequence로 분류되는 자료형에는 `string`, `list`, `tuple`이 존재
- sequence에서는 indexing과 slicing을 사용할 수 있음

### string literal

```python
"Good Bye!" # 큰 따옴표 사용
'Hello, World!' # 작은 따옴표 사용

# 큰 따옴표로 시작했으면 큰 따옴표로 끝나고, 작은 따옴표로 시작했으면, 작은 따옴표로 끝나야함
'나는 "박서희"다!' # 작은 따옴표 안에서, 큰 따옴표 사용
"나는 '박서희'다!" # 작은 따옴표 안에서, 큰 따옴표 사용

# 작은 따옴표 안에서 작은 따옴표는 안됨
'작은 따옴표 안에서 '작은 따옴표''

# 작은 따옴표 안에서 작은 따옴표 쓰기
'작은 따옴표 안에서 쓰려면 \'이렇게 쓰지요'

# 큰 따옴표 안에서 큰 따옴표 사용
"내가 \"박서희\"다!"
```

- `\'`, `\"`와 같이 앞에 백슬래쉬를 붙여서 다른 의미를 갖게된 문자들을 escape character(확장 문자)라고 함

### string 출력

```python
# 두 줄 출력
# 첫째 줄 
# 둘째 줄
print("첫째 줄 \n둘째 줄")

# tab 사용(여러 줄의 줄을 맞출 때)
print("나는 \t박서희") # 나는 	박서희
print("호호호 \t박서희") # 호호호 	박서희

# backspace 사용
print("박서희\b") # 박서

# 의도하지 않는 결과 주의(윈도우에서 파일 경로 표시할 때, 백슬래쉬 사용)
# C:lvin
# ame
print("C:\alvin\name")

# 앞에 r을 붙여 백슬래쉬를 일반적인 문자로 처리(Raw strings)
print(R"C:\alvin\name") # C:\alvin\name

# 여러 줄로 literal을 만들기
# 백슬래쉬를 넣으면 줄바꿈을 없앨 수 있음
# 안녕하세요?박서희입니다.
# 반갑습니다.
message = """안녕하세요?\
박서희입니다.
반갑습니다.
"""
print(message)

# 문자열 2개는 자동으로 합쳐짐
"안녕" "나야" # 안녕나야

# 변수명이나 표현식으로 합쳐지는 것은 불가
# SyntaxError: invalid syntax
name = "박서희"
name "다"

# +(더하기 연산자)
"하나둘셋" + str(456) # 하나둘셋456
```

### indexing(인덱싱)과 slicing(슬라이싱)

```python
# 길이 체크
len("ABC DEF GH") # 10
```

- 여러 글자로 이루어진 `string`에서 글자 하나를 가져오기 위해 `[]`(대괄호) 안에 위치를 의미하는 숫자 index를 사용하고 이를 indexing이라고 함
- `string` 외의 `list`나 `tuple`같은 sequence에서도 사용 가능

```python
# index는 0부터 시작
name = "박서희"
name[2] # 희

# index -1은 마지막 값이며 음의 index는 역순으로 가져옴
message = "박서희입니다"
message[-1] # 다
message[-2] # 니

# 글자 수 이상의 index를 사용할 경우
message = "박서희입니다"
message[100] # IndexError: string index out of range
```

- indexing의 기능을 확장해 일부를 잘라서 가져오는 것을 slicing이라고 함

```python
# :(콜론) 앞의 숫자는 잘라내기 시작하는 위치이며, 뒤의 숫자는 마지막 위치의 하나 전까지를 의미
message = "저는박서희입니다"
message[1:4] # 는박서

# message[:i] + message[i:]는 message와 동일
message = "저는박서희입니다"
message[:6] + message[6:] # 저는박서희입니다

# slicing 위치 시작을 len(i) 이상의 index를 사용할 경우
message = "안녕호호"
message[100:] # ''

# 건너뛰면서 가져오고 싶을 때는 두번째 :(콜론) 오른쪽에 1외에 숫자를 사용, 이를 step이라고 함
message = "후덜하고만"
message[::2] # 후하만

# step에 음수를 사용하면 역순
message = "후덜하고만"
message[::-2] # 만하후
```
