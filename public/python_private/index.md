# [Python]Private (_, __)


Python에서 Private속성을 다루는 방법과 name mangling에 대한 정리

<!--more-->

## _

python은 java에서의 private처럼 객체 내부에서만 접근할 수 있는 <<비공개>> 인스턴스 변수는 존재하지 않는다. 그러나 변수명 앞에 `_`를 붙이는 규약을 통해 공개적이지 않은 부분으로 취급하는 규약이 있다. 이는 규약일 뿐, 얼마든지 해당 변수에 접근과 변경이 가능하다.

```python
class Person:
    def __init__(self, name, age, weight):
        self.name = name
        self.age = age
        self._weight = weight

 
>>> harry = Person('harry', 25, 70)

>>> harry.name
'harry'

# _weight 접근 가능
>>> harry._weight
70

# _weihgt 변경 가능
>>> harry._weight = 80
>>> harry._weight
80
```



## __

그런데 변수명 앞에 `__`를 붙이고 접근을 하면 AttributeError가 발생하는 것을 확인할 수 있다.

```python
class Person:
        self.name = name
        self.age = age
        self.__weight = weight
        
>>> poter = Person('poter', 17, 60)
>>> poter.__weight
Traceback (most recent call last):
  File "<input>", line 1, in <module>
AttributeError: 'Person' object has no attribute '__weight'
```

 `__`를 붙여서 접근을 못하게끔하여 private한 변수로 만드는 것으로 생각할 수도 있다. 그러나 AttributeError는 해당 필드(`__weight`)가 존재하지 않는다는 에러이다. Python에서 이는 이름 뒤섞기(name mangling)이라는 것으로 `_<class-name>__<attribute-name>`이라는 속성을 만든다. 따라서 위의 예제에서 `__weight`에 접근하기 위해서는 아래와 같이 접근할 수 있다.

```python
>>> poter._Person__weight
60

# poter가 가진 인스턴스 변수 확인
>>> poter.__dict__
{'name': 'poter', 'age': 17, '_Person__weight': 65}

# 물론 변경도 가능하다.
>>> poter._Person__weight = 65
>>> poter._Person__weight?ㅇ
65
```

이러한 기능은 서브 클래스에서 정의된 이름들과의 충돌을 피하고자 존재한다고 한다. 따라서 클래스 내부의 메서드 호출을 방해하지 않고 서브 클래스들이 메서드를 재정의할 수 있도록 하는데 도움을 준다고 한다.



[Python Doc](https://docs.python.org/ko/3/tutorial/classes.html?highlight=mangling#private-variables)


