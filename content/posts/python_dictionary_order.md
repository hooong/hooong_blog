---
title: "Python 3.6부터는 Dict가 순서를 기억한다."
date: 2021-05-12T22:43:20+09:00
lastmod: 2021-05-12T22:43:20+09:00
description: "입력된 순서를 보장하는 딕셔너리에 대한 정리"
tags: ["Python"]
toc:
  auto: false
---

입력된 순서를 보장하는 딕셔너리에 대한 정리

<!--more-->
 딕셔너리에 구현되어 있는 함수들 중 `pop()`과 `popitem()`이 있는 것을 보고 "딕셔너리에는 순서가 없는 것으로 알고있었는데 어떻게 큐에서나 사용되는 pop()이 구현될 수 있지?"라는 의문으로 찾아보게 되었던 결과를 글로 옮겨 적어보게 되었습니다.




## Dictionary란?

 `이름 - 홍길동`, `성별 - 남` 과 같이 대응 관계를 나타내는 자료형을 연관 배열 또는 해시라고 하는데 Python에서는 이러한 자료형을 딕셔너리라고 합니다.  

 딕셔너리는 `key: value` 쌍의 형식으로 저장되는 자료형이며 특정 key는 해시가 가능한 값으로 해시된 key값을 이용해 value와 매핑을 하게 됩니다. 즉, key로 리스트, 딕셔너리와 같이 mutable한 객체는 사용할 수 없습니다.

```python
# dict의 예시
people = {'name': '홍길동', 'age': '22'}
```

 key를 해시하여 value에 매핑하는 구조이기 때문에 리스트나 튜플처럼 순차적으로 값에 접근하지 않고 key를 통해 바로 value에 접근을 할 수 있습니다. 이 때문에 Python 3.5 까지는 딕셔너리는 순서를 가지고 있지 않았습니다.




## OrderedDict란?

 OrderedDict는 일반적인 딕셔너리(Python 3.6이전)와는 다르게 입력된 순서를 저장하여 순서를 보장해주는 Python이 제공하는 자료형입니다. 

```python
# 입력된 순서가 보장되는 OrderedDict
>>> od = OrderedDict()
>>> od['a'] = 1
>>> od['b'] = 2
>>> od['c'] = 3

>>> od
OrderedDict([('a', 1), ('b', 2), ('c', 3)])

# Python 3.5 이전에서의 dict는 입력된 순서가 보장되지 않음.
>>> d = {}
>>> d['a'] = 1
>>> d['b'] = 2
>>> d['c'] = 3

>>> d
{'b': 2, 'a': 1, 'c': 3}

# Python 3.6 이후에서의 dict는 입력된 순서 보장
>>> d_ = {}
>>> d_['a'] = 1
>>> d_['b'] = 2
>>> d_['c'] = 3
>>> d_
{'a': 1, 'b': 2, 'c': 3}
```

 그러나 Python 3.6이후부터는 기존 딕셔너리도 입력된 순서를 가지게 되어 OrderedDict와 크게 차이점이 없다고 생각을 할 수 있습니다. 하지만 동등성을 확인할 때에 OrderedDict는 순서까지도 동등한지를 확인하게 되어 더 엄격하게 동등성을 검증합니다. 

```python
# 입력 순서가 다른 dict의 비교
>>> dict_a
{'a': 'apple', 'b': 'banana', 'p': 'pineapple'}
>>> dict_b
{'b': 'banana', 'a': 'apple', 'p': 'pineapple'}

>>> dict_a == dict_b
True

# 입력 순서가 다른 OrderedDict의 비교
>>> ordered_a
OrderedDict([('a', 'apple'), ('b', 'banana'), ('p', 'pineapple')])
>>> ordered_b
OrderedDict([('b', 'banana'), ('a', 'apple'), ('p', 'pineapple')])

>>> ordered_a == ordered_b
False
```

 따라서 데이터의 순서가 아주 중요한 상황이나 하위 호환성을 고려해야 할때에는 OrderedDict를 사용하여 순서를 보장하는 것도 고려해야 합니다.




## Python 3.6 이후 Dict에 순서가 생긴 이유

 Python에서 딕셔너리를 구현함에 있어 [`USABLE_FRACTION`을 계산하게 되는데 성능상의 문제로 `2/3 * dk_size`를 넘지못하는데](https://github.com/python/cpython/blob/474ef63e38726d4bcde14f6104984a742c6cb747/Objects/dictobject.c#L375) Python 3.5 이하 버전에서 딕셔너리를 구현하기 위해 PyDictKeyEntry 타입의 희소배열(빈 공간도 여전히 PyDictKeyEntry만큼의 크기를 가짐.)을 사용하고 이 배열의 크기를 dk_size로 저장하게 되는데, 이 때문에 메모리의 사용량이 컸는데, 이를 Python 3.6에서는 3.5에서와 다르게 두 개의 배열을 사용하여 메모리의 사용량을 줄일 수 있게 됩니다. 

- 참고 : Python 3.6에서부터 순서가 생기는 구조로 변경이 된 것 같은데 Python의 레퍼런스 문서에서는 Python 3.7부터 지원한다고 써있는거 보니 3.7에서부터 공식으로 지원을 하는 것 같습니다.




## 배열의 개수가 늘었는데 메모리의 사용량이 줄었다?

 위에서 두 개의 배열을 사용하여 메모리를 줄였다고 하는데 배열이 하나에서 두 개로 늘었는데 줄었다고하면 의문을 가질수도 있습니다. 

```python
# Python 3.5 
d = {'timmy': 'red', 'barry': 'green', 'guido': 'blue'}
entries = [['--', '--', '--'],
           [-8522787127447073495, 'barry', 'green'],
           ['--', '--', '--'],
           ['--', '--', '--'],
           ['--', '--', '--'],
           [-9092791511155847987, 'timmy', 'red'],
           ['--', '--', '--'],
           [-6480567542315338377, 'guido', 'blue']]

# Python 3.6
indices =  [None, 1, None, None, None, 0, None, 2]
entries =  [[-9092791511155847987, 'timmy', 'red'],
            [-8522787127447073495, 'barry', 'green'],
            [-6480567542315338377, 'guido', 'blue']]

```

 하지만 위의 두 버전에서 각기 다른 구현된 딕셔너리를 살펴보면 이해를 할 수 있습니다.

 앞서 설명하였듯이 Python 3.5에서는 `entries`라는 빈 공간도 PyDictKeyEntry라는 타입으로 채워진 하나의 배열을 사용해 key를 해시하여 얻은 값을 인덱스로 사용하여 저장하였다면, Python 3.6에서는 key와 매핑될 객체를 추가해나가는 `entries`와 key를 해시하여 얻은 값에 해당하는 위치에 담기는 순서를 저장하는  `indices`, 총 2개의 배열을 사용하면서 `indices`는 int 타입의 희소배열을 사용하게 되고 PyDictKeyEntry는 딕셔너리에 담긴 객체의 개수만큼만 크기를 차지하게 되어 메모리를 Python 3.5에서보다 절약이 될 수 있다고 합니다. 또, 이 글을 쓰게 된 이유인 순서를 저장하는 이유가 바로  `indices`에서 나오게 됩니다.




## 결론

 이렇게 Dict, OrderedDict에 대해서 간단히 알아보고 Python 3.5와 3.6에서 딕셔너리의 구현되어 있는 구조를 알아보면서 딕셔너리에 입력 순서가 저장되는 이유에 대해서 알아보았습니다. Python 3.6이후부터 딕셔너리에 순서가 생기기는 했지만, OrderedDict 부분에서 살펴본대로 동등성을 비교하는 부분이 다르고 하위호환을 고려한다면 용도에 맞게 사용하는 것이 좋을 것 같습니다.


<br>

----

- 참고 사이트
    - https://docs.python.org/ko/3/library/stdtypes.html#dict
    - https://docs.python.org/3/library/collections.html#collections.OrderedDict
    - https://stackoverflow.com/questions/39980323/are-dictionaries-ordered-in-python-3-6/39980744#39980744
