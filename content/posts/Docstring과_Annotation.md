---
title: "[Python]Docstring과Annotation"
date: 2021-03-06T21:50:13+09:00
---
 훌륭한 코드는 그 자체로 (주석이 없어도) 자명하지만 문서화 또한 잘 되어있어야한다. 문서화라고해서 주석이 주렁주렁 달린 코드를 말하는 것이 아닌 문서화를 통해 데이터 타입이 무엇인지 설명하고 때에따라 예제를 제공하는 것을 목표로 한다. 

<br>

## Docstring

`Docstring`은 소스코드에 포함된 문서라고 말할 수 있으며, 기본적으로 리터럴 문자열로 구성되며 로직의 일부분을 문서화하기 위해 코드 어딘가에 배치된다. 파이썬의 경우 동적 타이핑을 하기 때문에 함수의 입력과 출력을 문서화하여 사용자가 사용할 때 함수가 어떻게 동작하는지 이해하기 쉽게끔 docstring으로 문서화를 하는 것이 코드의 동작을 이해하는데 큰 도움이 될 수 있다.

따라서 Docstring을 아래와 같이 정리해 볼 수 있다.

- Docstring은 '이유'가 아닌 '설명'이다.
- 코드의 특정 컴포넌트(module, class, method, function)에 대한 문서화이다.
- 가능한 많은 docstring을 추가하는 것이 권장된다.
- [Sphinx(스핑크스)](https://tech.ssut.me/start-python-documentation-using-sphinx/)를 실행하여 autodoc 익스텐션을 사용하면 자동으로 문서를 만들 수 있다.

<br>

#### Example

```python
>>> def my_function():
...     """
...     1. 이 함수가 하는 일은 무엇인가요?
...
...     2. 이 함수는 무엇을 입력받나요?
...
...     3. 그래서 최종적으로 이 함수는 무엇을 반환하나요?
...     """
...     pass
...
>>> my_function.__doc__
'1. 이 함수가 하는 일은 무엇인가요?\n
 \n
 2. 이 함수는 무엇을 입력받나요?\n
 \n
 3. 그래서 최종적으로 이 함수는 무엇을 반환하나요?\n'
```

- `"""{docstring}"""` : 리터럴 문자열로 특정 컴포넌트에 대한 문서화를 작성
- `__doc__` : 해당 속성을 통해 docstring에 접근 가능

<br>

```python
# dict.update에 대한 docstring 예제
>>> dict.update.__doc__
'D.update([E, ]**F) -> None.  Update D from dict/iterable E and F.\n
If E is present and has a .keys() method, then does:  for k in E: D[k] = E[k]\n
If E is present and lacks a .keys() method, then does:  for k, v in E: D[k] = v\n
In either case, this is followed by: for k in F:  D[k] = F[k]'
```

<br>

## Annotation

 Python의 경우 동적으로 타입을 결정하기 때문에 함수나 메서드를 거치면서 변수나 객체의 값이 무엇인지 알기가 어려운 경우가 많으므로 어노테이션을 통해 이러한 정보를 명시해준다면 다른 개발자가 코드를 쉽게 이해하는데 도움이 된다.

 Java에도 Annotation이 존재한다. 그러나 Python에서의 Annotation은 단순히 타입에 대한 `힌트`를 주는 역할만을 하게된다. (Java의 Annotation은 Python의 Decorator가 비슷한 역할을 한다고 볼 수 있다.)

Annotation에 대해 간단히 정리해보면 다음과 같다.

- 변수의 예상 타입을 지정하여 힌트를 줄 수 있다. (타입 뿐만 아니라 어떤 형태의 메타데이터라도 지정할 수 있다.)
- annotation정보를 사용하여 문서 생성, 유효성 검증, 타입 체크를 할 수 있다.

<br>

#### Example

```python
>>> class Point:
...     def __init__(self, lat, long):
...             self.lat = lat
...             self.long = long
...
>>> def locate(latitude: float, longitude: float) -> Point:
...     """맴에서 좌표에 해당하는 객체를 검색"""
...     return Point(latitude, longitude)
...
>>> locate.__annotations__
{'latitude': <class 'float'>, 'longitude': <class 'float'>, 'return': <class '__main__.Point'>}

# 변수에도 지정 가능 (Python 3.6이상)
>>> class Point:
...     lat: float
...     long: float
...
>>> Point.__annotations__
{'lat': <class 'float'>, 'long': <class 'float'>}

# 타입이 아닌 문자열도 가능
>>> def aa() -> "리턴":
...     pass
...
>>> aa.__annotations__
{'return': '리턴'}
```

- `:` : 함수의 파라미터 및 변수 뒤에 콜론을 이용하여 annotation을 달 수 있다.
- `->` : 함수의 반환값에 대한 annotation을 달 수 있다.
- `__annotations__` : 해당 속성을 통해 annotation에 접근 가능

<br>

## Annotation은 Docstring을 대체?

Docstring에 포함된 정보의 일부는 어노테이션으로 이동시킬 수 있는 것은 사실이지만, docstring을 통해 보다 나은 문서화를 위한 여지를 남겨두어야 한다. 동적 데이터 타입과 중첩 데이터 타입의 경우 예상 데이터의 예제를 제공하여 어떤 형태의 데이터를 다루는지 제공하는 것이 좋다는 것이다. 

<br>

```python
def data_from_response(response: dict) -> dict:
    if response["status"] != 200:
        raise ValueError
    return {'data': response["payload"]}
```

예를들어 위와 같이 데이터의 유효성을 검사하고 dict값을 반환하는 함수를 보면 `response`객체의 올바른 인스턴스의 형태는 알 수가 없다. 따라서 아래와 같이 이러한 함수에 docstring을 통해 보다 나은 설명을 추가할 수가 있다.

```python
def data_from_response(response: dict) -> dict:
    """response에 문제가 없다면 response의 payload를 반환
    
    - response의 예::
    {
        "status": 200,  # <int>
        "payload": { ... }  # 반환하려는 데이터
    }
    
    - 반환 dict 값의 예::
    {"data": { ... } }
    
    - 발생 가능한 예외:
    - HTTP status가 200이 아닌 경우 ValueError 발생
    """
    
    if response["status"] != 200:
        raise ValueError
    return {'data': response["payload"]}
```

또한, docstring은 단위 테스테에서도 유용한 정보로 사용될 수 있다. 예를들어, 테스트용 입력 값을 생성할 수도 있고 테스트의 성공 실패를 판단하는 것이다.  

간단히 정리를 해보자면, Annotation은 Docstring을 부분적으로 대체를 할 수는 있겠지만, Annotation과 Docstring을 적절히 사용하여 코드의 가독성을 높이고 코드에 대한 문서화를 더욱 보기 좋게 다듬는 것이 중요하다고 할 수 있겠다.







