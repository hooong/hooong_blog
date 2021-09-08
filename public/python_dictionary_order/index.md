# Python 3.6부터는 Dict가 순서를 기억한다.



<!--more-->
 딕셔너리에 구현되어 있는 함수들 중 `pop()`과 `popitem()`이 있는 것을 보고 "딕셔너리에는 순서가 없는 것으로 알고있었는데 어떻게 큐에서나 사용되는 pop()이 구현될 수 있지?"라는 의문으로 찾아보게 되었던 결과를 글로 옮겨 적어보게 되었습니다.




## 딕셔너리란?

딕셔너리는 사전이라는 의미를 가지는 단어로 Python에서는 사전형 데이터를 담는 자료형을 뜻합니다. 

사전형 데이터란 다른 말로 연관 배열이라고도 할 수 있습니다. 연관 배열은 키 하나와 값 하나가 연관되어 키를 통해 연관된 값을 얻을 수 있습니다. 실제로 국어사전을 통해 원하는 단어를 찾아 그에 연관된 단어의 뜻을 찾아보는 것을 생각해보면 쉽게 이해할 수 있습니다.

앞서 말한대로 딕셔너리는 하나의 키와 하나의 값을 가지며  `{key: value}`  형식으로 저장됩니다. 

```python
# 딕셔너리 예시
people = {'name': '홍길동', 'age': 20}

people['name']
> '홍길동'
people['age']
> 20
```

키는 해시가 가능한 값으로 해시된 키를 이용해 값과 매핑하여 연관된 값을 저장하거나 얻어올 수 있습니다. 따라서 리스트, 딕셔너리와 같이 mutable한 객체는 해시가 불가능하여 딕셔너리의 키로는 사용할 수 없습니다. 

또한, 리스트나 튜플처럼 순차적으로 값에 접근하는 것과 달리 딕셔너리는 해시된 키를 통해 바로 값에 접근이 가능하므로 입력된 순서를 저장할 필요성이 없었으며 실제로 Python3.5까지는 딕셔너리는 입력 순서를 저장하지 않습니다. 하지만 Python3.6부터는 딕셔너리를 구현하는 내부구조의 변경으로 인해 입력 순서를 저장하게 됩니다.



## Python3.6부터는 딕셔너리가 어떻게 순서를 저장하는가?

우선 아래의 예시를 통해 Python3.5와 Python3.6에서 딕셔너리가 입력된 순서를 저장하는지부터 확인해보겠습니다.

```python
# [Python3.5] 입력된 순서가 보장되지 않음.
>>> dict_3_5 = {}
>>> dict_3_5['a'] = 1
>>> ddict_3_5['b'] = 2
>>> dict_3_5['c'] = 3

>>> dict_3_5
{'b': 2, 'a': 1, 'c': 3}

# [Python3.6] 입력된 순서가 보장됨.
>>> dict_3_6 = {}
>>> dict_3_6['a'] = 1
>>> dict_3_6['b'] = 2
>>> dict_3_6['c'] = 3
>>> dict_3_6
{'a': 1, 'b': 2, 'c': 3}
```

살펴본 바와 같이 Python3.6에서 딕셔너리가 어떻게 순서를 저장하는지를 알기 위해서는 Python3.6 이전과 이후의 딕셔너리 내부 구조에 대하여 살펴볼 필요가 있습니다.

```python
people = {
  'firstname': '길동',
  'lastname': '홍',
  'job': '개발자'
}
```

위 처럼  `people`이라는 딕셔너리 객체가 있을 때, Python3.6이전과 이후의 구조를 살펴보면 다음과 같습니다.

- Python3.6 이전의 딕셔너리 구조

<img width="515" alt="스크린샷 2021-07-10 오후 10 38 37" src="https://user-images.githubusercontent.com/78338337/125164912-9806e280-e1cf-11eb-9432-bb7d6ce9c087.png">

Python3.6 이전에는 `dk_size`크기(`dk_size`는 해시테이블의 크기로 여기서는 8이다.)만큼의 `entries`라는 해시 테이블을 생성하고 해싱된 key값에 해당하는 인덱스 위치로 엔트리가 저장됩니다. 따라서 `hash('firstname') % 8`을 통해 바로 엔트리에 접근이 가능합니다. 

<img width="569" alt="스크린샷 2021-07-10 오후 10 59 29" src="https://user-images.githubusercontent.com/78338337/125165575-7f4bfc00-e1d2-11eb-8e75-3f3838b3c76e.png">

반면, Python3.6 이후부터는 `dk_size`크기의 `indices`라는 해시 테이블을 두고 해싱된 key값에 해당하는 위치에 `entries`에 삽입될 위치의 인덱스를 저장하고 엔트리가 삽입될때마다 `entries` 배열에 삽입된 순서대로 삽입이 되는 구조로 바뀌었습니다. 즉, 순서를 저장할 수 있게되었습니다. 따라서 `hash('firstname') % 8`의 결과값을 가지고 `indices`에서 `entries`에 접근할 수 있는 인덱스를 얻은 뒤 `entries[0]`으로 접근이 가능하게 됩니다. 

- 참고로 Python 3.6에서부터 순서가 생기는 구조로 변경이 된 것 같은데 Python의 레퍼런스 문서에서는 Python 3.7부터 지원한다고 써있는거 보니 3.7부터 공식으로 지원을 하는 것 같습니다.



## 배열의 개수가 늘었는데 메모리의 사용량이 줄었다?

Python3.6 버전으로 오면서 딕셔너리를 구현하기 위해 사용하는 배열이 하나에서 두 개로 늘어났는데 메모리가 줄었다고하면 의문을 가질수도 있습니다. 결론부터 말하자면, 해시 테이블로 사용하는 희소배열의 타입을 바꿔 빈 공간에 대한 메모리를 줄일 수 있게 됩니다.

 

좀 더 자세히 살펴보면 Python3.6 이전 버전에서는 사이즈가 `dk_size`인  `PyDictKeyEntry` 타입의 `entries` 해시 테이블을 사용합니다. 따라서, 배열의 빈 공간마저 `PyDictKeyEntry`만큼의 메모리를 차지하여 낭비하는 부분이 컸습니다. 그러나 변경된 Python3.6 이후 버전에서는 사이즈가 `dk_size`인 `char`타입의 `indices`라는 해시 테이블과 `PyDictKeyEntry`를 담는 `entries`는 필요할때마다 할당되어 추가되어 Python3.6 이전 버전보다는 메모리를 효율적으로 사용할 수 있다고합니다.

<img width="950" alt="스크린샷 2021-07-11 오전 1 44 38" src="https://user-images.githubusercontent.com/78338337/125170321-92b69180-e1e9-11eb-92b4-b64c03f680fb.png">

위의 `people` 딕셔너리 객체를 가지고 계산을 해보면 다음과 같습니다.

Python3.6 이전 버전의 경우 `8(dk_size) * 8 * 3`로 192 Bytes를 차지하게 되는 반면, Python3.6 이후 버전의 경우에는 `8(dk_size) * 1 + (8 * 3 * 3)`로 80 Bytes를 차지는 것을 확인해 볼 수 있습니다.



## OrderedDict란?

사실 Python에는 3.6버전 이전부터 입력된 순서를 보장하는 특수한 딕셔너리 자료형을 가지고 있습니다. 그것이 바로 OrderedDict입니다.

OrderedDict는 앞에서 말한바와 같이 순서를 보장하는 딕셔너리로 아래 예제를 통해 순서가 보장된다는 것을 확인해 볼 수 있습니다.

```python
# 입력된 순서가 보장되는 OrderedDict
>>> odered_dict = OrderedDict()
>>> odered_dict['a'] = 1
>>> odered_dict['b'] = 2
>>> odered_dict['c'] = 3

>>> odered_dict
OrderedDict([('a', 1), ('b', 2), ('c', 3)])
```

Python 3.6이후부터는 기존 딕셔너리도 입력된 순서를 가지게 되어 OrderedDict와 크게 차이점이 없다고 생각을 할 수 있습니다. 하지만 동등성을 확인할 때에 OrderedDict는 순서까지도 동등한지를 확인하게 되어 더 엄격하게 동등성을 검증합니다. 

입력 순서는 다르지만 내용이 같은 경우에 서로 비교하는 예시를 아래에서 살펴볼 수 있습니다.

```python
# 기본 딕셔너리
>>> dict_a
{'a': 'apple', 'b': 'banana', 'p': 'pineapple'}
>>> dict_b
{'b': 'banana', 'a': 'apple', 'p': 'pineapple'}

>>> dict_a == dict_b
True

# OrderedDict
>>> ordered_a
OrderedDict([('a', 'apple'), ('b', 'banana'), ('p', 'pineapple')])
>>> ordered_b
OrderedDict([('b', 'banana'), ('a', 'apple'), ('p', 'pineapple')])

>>> ordered_a == ordered_b
False
```

따라서 데이터의 입력된 순서가 아주 중요한 상황이나 하위 호환성을 고려해야하는 상황에서는 OrderedDict를 사용하여 순서를 보장하는 고려해보는 것이 좋을 것 같습니다.



## 결론

딕셔너리와 OrderedDict에 대해서 간단히 알아보고 Python 3.5와 3.6에서 딕셔너리의 구현되어 있는 구조를 알아보면서 딕셔너리에 입력 순서가 저장되는 이유에 대해서 알아보았습니다. Python 3.6이후부터 딕셔너리에 순서가 생기기는 했지만, OrderedDict 부분에서 살펴본대로 동등성을 비교하는 부분이 다르고 하위호환을 고려한다면 용도에 맞게 사용하는 것이 좋을 것 같습니다.



- 참고 사이트

  - https://docs.python.org/ko/3/library/stdtypes.html#dict
  - https://docs.python.org/3/library/collections.html#collections.OrderedDict
  - https://stackoverflow.com/questions/39980323/are-dictionaries-ordered-in-python-3-6/39980744#39980744

  - https://github.com/zpoint/CPython-Internals/blob/master/BasicObject/dict/dict.md
  - https://kadensungbincho.tistory.com/23


