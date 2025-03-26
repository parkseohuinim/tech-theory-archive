# Python 기본 개념과 이론

## 1. 기본

### Object(객체)와 Data type(자료형)

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

### Object의 상호 작용 및 이름

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

### Flow control

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

### Object 사용

- object의 이름 뒤에 `.`(점)을 찍어 **attribute(속성)**와 **method(메서드)**를 사용

```python
a = int(1) # a = 1
b = a.__add__(1) # a에 1을 더하기
print(b) # 2
```

---

### User defined data type(사용자 정의 자료형)

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
