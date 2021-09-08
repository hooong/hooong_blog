# [Python]property


Python에서 prooperty에 대한 정리

<!--more-->



## Property

이 글은 `파이썬 클린코드` 책의 내용을 정리한 글입니다.

프로퍼티는 객체의 어떤 속성에 대한 접근을 제어하려는 경우 사용한다. 이렇게 하는게 또한 파이썬스러운 코드라고 한다. 프로퍼티는 자바에서의 접근메서드인 getter와 setter를 만드는 것과 같은 용도라고 할 수 있다.

```python
def is_valid_email(potentially_valid_email: str):
    return re.match(EMAIL_FORMAT, potentially_valid_email) is not None


class User:
    def __init__(self, username):
        self.username = username
        self._email = None
        
    @property
    def email(self):
        return self._email
    
    @email.setter
    def email(self, new_email):
        if not is_valid_email(new_email):
            raise ValueError(f'유요한 이메일이 아니므로 {new_email} 값을 사용할 수 없음')
        self._email = new_email
        
    @email.deleter
    def email(self):
        self._email = None

# 사용 예시
> hong = User('hong')

> hong.email = 'hong@test.kr'

> hong.email
'hong@test.kr'

> del hong.email
  
> hong.__dict__
{'username': 'hong', '_email': None}
```

- `@property`가 붙은 메서드는 private 속성인 `email` 값을 반환한다.
- `@email.setter`가 붙은 메서드는 `email`값을 검증한 뒤 업데이트한다.
- `@email.deleter`가 붙은 메서드는 `email` 속성을 `None`로 초기화한다.



프로퍼티를 사용하면 명령-쿼리 분리 원칙(command and query separation)을 따르기 위한 좋은 방법이다.

명령-쿼리 분리 원칙이란?
- 객체의 메서드가 무언가의 상태를 변경하는 커맨드이거나 무언가의 값을 반환하는 쿼리이거나 둘 중에 하나만 수행해야지 둘 다 동시에 수행하면 안된다는 것



즉, 프로퍼티를 명령-쿼리로 나누어보면 `@property` 데코레이터는 무언가에 응답하기 위한 쿼리이고, `@<property_name>.setter`, `@<property_name>.deleter`데코레이터는 무언가를 하기 위한 커맨드이다. 



또한, 책에서 덧붙이는 팁은 `메서드는 한 가지만 수행해야 한다. 작업을 처리한 다음 상태를 확인하려면 메서드를 분리해야 한다.` 이다.  예를들어  `if self.set_email("a@j.com")`처럼 코드를 사용하게 되면 이메일을 설정하려는 건지, 이미 이메일이 해당 값으로 설정되어 있는지 확인하려는지, 아니면 동시에 이메일 값을 설정하고 상태가 유효한지 체크하는 것인지를 구분하기 어렵기때문이다.

----

## 추가적인 프로퍼티 사용법

위의 프로퍼티는 데코레이터를 사용한 방법이고 아래는 `property` class를 직접적으로 사용한 예제이다.

```python
class property(fget=None, fset=None, fdel=None, doc=None)
	# Return a property attribute
```



```python
class C:
    def __init__(self):
        self._x = None
        
    def getx(self):
        return self._x
      
    def setx(self7777777777, value):
        self._x = value
        
    def delx(self):
        del self._x
        
    x = property(getx, setx, delx, "I'm the 'x' property.")
    

# property 객체 반환
> C.x
<property object at 0x109119cc0>

# property doc
> C.x.__doc__
"I'm the 'x' property."
 
> a = C()

# setx
> a.x = 'haha'

# getx
> a.x
'haha'

# delx
> del a.x
```


