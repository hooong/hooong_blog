---
title: "[Python] a, b = b, a는 어떻게 동작하는걸까?"
date: 2021-11-10T20:54:21+09:00
description: "python multiple assignment의 동작에 관한 정리"
tags: ["Python"]
toc:
  auto: false
---

<!--more-->
# [Python] a, b = b, a는 어떻게 동작하는걸까?



## 시작하며

일반적으로 Java나 C언어에서 변수 a와 b의 값을 서로 바꾸기 위해서는 tmp변수를 두어 아래와 같이 구현을 해야한다.

```c
void swap(int a, int b) {
  int tmp = a;
  a = b;
  b = tmp;
}
```

그러나 신기하게도 Python에서는 아래와 같이 a와 b의 값을 바꿀 수 있다.

```python
def swap(a, b):
  a, b = b, a
```

도대체 Python 어떻게 동작하길래 a와 b를 바로 바꿔서 저장을 할 수 있는걸까?라는 의문이 생겼고, 이를 조사해본 내용을 정리해보고자 합니다.



## Python의 swap 동작 방식

Python은 공식적으로 버추얼 머신을 구현하여 제공하지 않아 CPython, PyPy, IronPython, Jython등 다양한 구현체가 존재합니다. 각기 구현체마다 swap을 하는 방식은 다를 수 있겠지만 이번 글에서는 대표적으로 사용되는 CPython과 PyPy에 대해서 살펴보고자 합니다.



### CPython에서의 swap

swap의 동작 방식을 알기위해서는 작성된 코드의 바이트코드를 살펴보아야 했습니다. ([dis 라이브러리](https://docs.python.org/ko/3.8/library/dis.html)는 컴파일 되는 바이트 코드를 분석하기 쉽게 역어셈블을 해준다.)

그럼 위에서 작성되었던 python의 sw는p 코드인 `a, b = b, a`를 바이트 코드의 동작 과정을 살펴보면 아래와 같습니다.

```
>> dis.dis(swap)
0 LOAD_FAST         1 (b)
2 LOAD_FAST         0 (a)
4 ROT_TWO
6 STORE_FAST        0 (a)
8 STORE_FAST        1 (b)
```

각 줄에는 코드의 offset(각 명령어는 2바이트를 차지함), opname(명령어), arg 순으로 보여지게 됩니다. 위의 동작 과정을 알기 위해서는 여기서 나오는 각 명령어들에 대해서 살펴보아야할 필요성이 있었습니다. 간단하게 정리해보면 아래와 같습니다.

> [LOAD_FAST](https://docs.python.org/ko/3.8/library/dis.html#opcode-LOAD_FAST) : 로컬 메모리에 저장된 값을 스택으로 푸시
>
> [STORE_FAST](https://docs.python.org/ko/3.8/library/dis.html#opcode-STORE_FAST) : 스택의 최상단의 값을 로컬 메모리에 저장
>
> [ROT_TWO](https://docs.python.org/ko/3.8/library/dis.html#opcode-ROT_TWO) : 스택 최상단의 두 값의 자리를 바꿈

위의 명령어를 보면 CPython의 버추얼 머신이 스택으로 동작한다는 것을 눈치채볼 수 있습니다. CPython의 버추얼 머신은 글로벌, 로컬 메모리와 연산을 위한 스택을 가진다고합니다. 따라서 어떠한 연산을 하기 위해서는 글로벌 또는 로컬에 저장되어 있는 값을 스택에 복사를 해온 뒤 사용하게 됩니다.

그럼 이제 명령어에 대해서도 얼핏 알게되었으니 swap의 동작 과정을 살펴봅시다. 차례대로 동작 과정을 한글로 정리해보면 다음과 같습니다.

```
1. 변수 b의 값을 스택에 푸시
2. 변수 a의 값을 스택에 푸시
3. 스택 상단의 두 값 위치를 변경
4. 스택 최상단의 값(b)를 변수 a에 저장
5. 스택 최상단의 값(a)를 변수 b에 저장
```

그럼 여기서 의문을 가질수도 있습니다. 3번 과정(스택의 상단 두 개의 값의 위치를 변경)은 어떻게 동작하는가. 구현체를 살펴보면 아래와 같습니다.

```c++
TARGET(ROT_TWO) {
  PyObject *top = TOP();
  PyObject *second = SECOND();
  SET_TOP(second);
  SET_SECOND(top);
  FAST_DISPATCH();
}
```

위의 코드를 살펴보면 `top`, `second`라는 두 개의 임시변수를 사용하고 있다는 것을 눈치챌 수 있습니다. 즉, 전통적인 java, c에서 구현하는 swap보다 비효율적이라는 것을 알 수 있었습니다. *(필자는 코드가 단축된 만큼 swap과정에서 더 적은 메모리를 사용하여 효율적이라고 문득 생각했지만 이 생각이 잘못되었다는 생각을 가지게 된 부분이다.)* 또한, 이미 스택이라는 임시 공간을 사용하는 것으로 비효율적이라고 할 수 있습니다. 

쉽게 비유를 해보면, 전통적인 java 혹은 c에서의 swap 동작에서는 의자가 3개만 있으면 되지만, CPython에서는 2(로컬 메모리) + 2(스택) + 2(ROT_TWO에서의 임시변수)로 총 6개의 의자를 가져야하는 것입니다.



### PyPy에서의 swap

CPython보다 높은 성능을 내는것을 주된 목표를 가진 PyPy에서는 CPython에서와 동작 과정은 살짝 다릅니다. 바로 PyPy 바이트 코드의 동작과정을 살펴보면 아래와 같습니다.(PyPy를 사용한 가상환경에서 dis라이브러리를 사용)

```
0 LOAD_FAST                1 (b)
2 LOAD_FAST                0 (a)
4 STORE_FAST               1 (b)
6 STORE_FAST               0 (a)
```

위의 동작 과정을 보면 CPython과 가장 큰 차이점으로는 `ROT_TWO`이 없다는 것이다. PyPy의 경우에는 가장 상단의 값을 바로 b에 저장하도록 구현이 되어있기 때문이라고 합니다. 즉 아래와 같이 동작합니다.

```
1. 변수 b의 값을 스택에 푸시
2. 변수 a의 값을 스택에 푸시
3. 스택 최상단의 값(a)를 변수 b에 저장
4. 스택 최상단의 값(b)를 변수 a에 저장
```

따라서 CPython에 비해 2개의 의자를 덜 사용하기 때문에 더 빠르고 효율적이라고 볼 수 있습니다.



### 그럼... 그냥 Python에서도 전통적인 swap을 구현하면 효율적이지 않나요?

아니요! 그렇지 않습니다. 

```python
def swap(a, b):
     tmp = a
     a = b
     b = tmp
```

위의 Python으로 짠 전통적인 swap함수의 바이트 코드를 분석해보면 다음과 같습니다.

```
0 LOAD_FAST                0 (a)
2 STORE_FAST               2 (tmp)

4 LOAD_FAST                1 (b)
6 STORE_FAST               0 (a)

8 LOAD_FAST                2 (tmp)
10 STORE_FAST              1 (b)
```

얼핏 보면 `ROT_TWO`가 없으니까 `a, b = b, a`보다 효율적이지 않은가?라고 생각할 수도 있습니다. 그러나 위에서도 알 수 있었듯이 python은 스택 머신으로 로컬 메모리로 바로 값이 할당되지 않고 연산을 위해서는 스택을 거친다는 것입니다. 따라서 a를 tmp로 할당하는 과정, b를 a에 할당하는 과정, tmp를 b로 할당하는 과정에서 스택을 사용하게 됩니다. 즉, 여기서의 swap함수를 실행하는데 4개의 의자가 필요한 것입니다.(이는 의자 3개를 사용하는 java나 c보다 비효율적이다.)

추가적으로 참조한 블로그 글에 따르면 `LOAD_FAST`와 `STORE_FAST`를 하는 것보다 `ROT_TWO`를 하는게 더 빠르다고 한다. 그렇기때문에 Python을 사용한다면 전통적인 swap 구현 방식 보다는 `a, b = b, a`를 사용하는 것이 더 빠르다고 한다.



## 마무리하며

막연한 궁금증으로 시작하여 깊게는 아니지만 Python의 특징들에 대해 알아볼 수 있는 계기가 되었던 것 같고, 앞으로도 더 깊게 찾아볼 Python에 대한 주제들도 얻어갈 수 있는 시간이어서 정말 좋았던 것 같다. 앞으로도 시간이 될때마다 CPython, PyPy등 다양한 구현체의 특징에 대해서도 알아보아야겠다.



## 레퍼런스

- https://blog.seulgi.kim/2017/01/python-swap.html?m=1

- https://github.com/python/cpython/blob/752c979775c75f9b95d1227bfd60caa2beae6590/Python/ceval.c#L1210-L1216

