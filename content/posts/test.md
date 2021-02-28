---
title: "Test"
date: 2021-03-01T00:27:52+09:00
---
# [백준10266번] 시계 사진들 / Python3

## 문제

상근이는 보통의 시계와는 다른 독특한 시계 사진 두장이 있습니다. 시계는 n개의 동일한 길이와 목적을 가진 시계 바늘들을 가지고 있습니다. 애석하게도 시계의 숫자들은 희미해져 각 시계 바늘들의 위치만 구분 할 수 있습니다.

우리의 상근이는 두 사진의 시계가 같은 시각을 나타낼 수 있는지 궁금해져 각 사진을 서로 다른 각도로 돌려보려고 합니다.

두 사진에 대한 묘사가 주어질 때, 두 사진의 시계가 같은 시각을 나타내는지 결정하세요.

## 입력

첫 줄에는 바늘의 수를 나타내는 정수 n(2 ≤ n ≤ 200 000)이 주어진다.

다음 두줄에는 각각 n개의 정수가 주어지며, 주어지는 정수 ai(0 ≤ ai < 360,000)는 각 사진에서 바늘의 시계 방향 각도를 나타낸다. 이때 바늘의 각도는 특정 순서대로 주어지지는 않는다. 한 줄에는 같은 각도값이 두 번 이상 주어지지 않는다. 즉, 한 시계 안의 모든 각도값은 서로 구분된다.

## 출력

두 시계 사진이 같은 시각을 나타내고 있다면 "possible"을 아니면 "impossible"을 출력하시오.

## 예제 입력 1

```
6
1 2 3 4 5 6
7 6 5 4 3 1
```

## 예제 출력 1

```
impossible
```

## 예제 입력 2

```
2
0 270000
180000 270000
```

## 예제 출력 2

```
possible
```

## 예제 입력 3

```
7
140 130 110 120 125 100 105
235 205 215 220 225 200 240
```

## 예제 출력 3

```
impossible
```

<br>

## 나의 풀이

> 해당 문제는 주어지는 각 각도를 배열로 만들고 KMP로 패턴 매칭을 통해 해결할 수 있었다.
>
> 우선 크기가 `360,000`이고 `0`으로 채워진 배열을 만들고 입력으로 주어지는 각도값에 해당하는 인덱스에 `1`값을 지정해준다. 이렇게 입력받은 두 배열을 각각 `clock1`, `clock2`라고 한다면 시계가 원형인 것을 고려해 `clock1`을 두 개 이어붙여주고 `clock2`를 패턴으로 삼아 KMP 알고리즘을 실행하고 `clock2` 패턴이 존재한다면 `possible`, 존재하지 않는다면 `impossible`을 출려하면 된다.
>
> 글로만 이해가 힘들 수 있기때문에 문제에서 주어진 예제2번을 간단하게 만들어 살펴보았다.
>
> ![IMG_0CD6784FCCF4-1](https://user-images.githubusercontent.com/37801041/102909920-0a76d600-44bd-11eb-9a62-6e8f431104b4.jpeg)
>
> `clock1`은 0도, 270도로 0번 인덱스와 3번 인덱스를 1로 만들고나서 `clock1`을 두개 이어붙여 만들어두고 `clock2`는 180도, 270도로 2번 인덱스, 3번 인덱스를 1로 만들어둔다. 그 후 `clock1`은 문자열, `clock2`는 패턴으로 생각하고 KMP를 돌리면 `0011`부분이 일치하므로 `possible`을 출력하게 된다.

<br>

## 코드

```python
# 10266번 시계 사진들


def failure(p):
    tmp = [0] * len(p)

    j = 0
    for i in range(1, len(p)):
        while j > 0 and p[i] != p[j]:
            j = tmp[j-1]

        if p[i] == p[j]:
            j += 1
            tmp[i] = j
    return tmp


def kmp(s, p):
    table = failure(p)

    j = 0
    for i in range(len(s)):
        while j > 0 and s[i] != p[j]:
            j = table[j-1]

        if s[i] == p[j]:
            if j == len(p)-1:
                return True
            else:
                j += 1
    return False


n = int(input())
clock1 = [0] * 360000
clock2 = [0] * 360000

for a in map(int, input().split()):
    clock1[a] = 1

for b in map(int, input().split()):
    clock2[b] = 1

clock1 += clock1

if kmp(clock1, clock2):
    print("possible")
else:
    print("impossible")

```


