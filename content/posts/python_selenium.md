---
title: "[Python] Selenium을 사용해 매크로 만들기 기초"
date: 2021-04-13T14:00:53+09:00
description: "Selenium이 무엇이고, 이를 이용해 간단하게 매크로 만들기에 대한 내용"
tags: ["Python", "selenium"]
toc:
  auto: false
---
<!--more-->

## 시작하며
 선착순으로 구매하는 이벤트가 진행될 때 매크로를 사용하여 성공하시는 분들이 있다는 것을 알고 매크로는 어떻게 만들어지는지 궁금증이 생겨 검색을 해보면서 `selenium`을 알게 되었고, 간단히 사용해보면서 알게된 점들에 대해서 이 글에 작성하게 되었습니다.

----

## selenium이란?
```
Selenium is a suite of tools for automating web browsers.
```
위의 문구는 selenium 공식 홈페이지에 적혀있는 문구로 selenium을 한 문장으로 잘 설명하는 것 같다고 느껴졌는데, 간단히 말해 '웹 브라우저를 자동화 시킬 수 있는 도구다!' 라고 할 수 있을 것 같습니다.

따라서 selenium을 사용하면 사람이 웹 브라우저에서 입력을 한다거나 클릭하는 행동들을 코드로 작성할 수 있고 코드를 실행시킴으로써 작성된 행동들을 자동으로 실행될 수 있게됩니다!

또한, 공식문서에 따르면 현재까지 selenium이 지원되는 언어로는 `Java`, `Python`, `C#`, `Ruby`, `JavaScript`, `Kotlin`이 있습니다. (해당 글에서는 Python을 사용하여 예제를 작성하였습니다.)

----

## selenium을 사용하기 전 준비물 (feat. Chrome Driver)
selenium을 사용하기 위해서는 기존 사용하던 Chrome과 같은 웹 브라우저 애플리케이션이 아닌 별도의 web driver가 필요합니다.

일반적으로는 Chrome을 많이 사용하시고 계실거라 생각하며 필자도 Chrome을 사용하고 있어 Web driver로 Chrome driver를 사용해보았습니다. (firefox, safari 같은 타 브라우저 driver도 지원한다고 합니다. [[참고문서](https://www.selenium.dev/documentation/webdriver/capabilities/)])

<br>

- Chrome Driver 설치법 (Mac Homebrew를 이용한 방법)

  ```shell
  $ brew update
  $ brew install chromedriver
  ```

----

## selenium을 사용해 Google 검색 자동화
앞서 selenium을 사용할 준비물인 Chrome Driver가 준비되었다면 드디어 selenium을 설치하고 간단하게 자동으로 Google 페이지에 접속하고 설정된 검색어를 가지고 검색까지 해보는 과정을 통해 selenium과 친해지는 시간을 가져보도록 하겠습니다.

### selenium 설치
```shell
$ pip install selenium
```

### selenium을 사용해 웹 브라우저 띄워보기
```Python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get('https://www.google.com')
```
위의 코드를 작성하고 실행을 해보면 Chrome Browser가 실행되고 아래와 같이 Google의 메인화면으로 접속이 되는 것을 확인해볼 수 있습니다.<br>
<p align="center">
  <img width="500" alt="Screen Shot 2022-04-13 at 4 31 36 PM" src="https://user-images.githubusercontent.com/78338337/163123725-e4e27521-ff77-4590-b8fc-dd44aa27e833.png">
</p>

<br>

- [참고] <br>
  - 코드 실행 시 `“chromedriver” cannot be opened because the developer cannot be verified.`과 같은 문구와 함께 알럿이 발생하는 경우 : <br>`cancel`버튼을 누르고 Mac의 `System Preferences > Security & Privacy > General > allow apps downloaded form:`부분에 나타나는 `Allow Anyway`를 눌러주시면 해결됩니다. <br><br>
  - (꿀팁) 여러번 위의 코드를 실행했다면 Dock에도 여러개의 Goggle Chrome이 실행 중인 것을 확인하실 수 있을 것입니다. 실행되었던 Chrome을 종료시켜주지 않으면 컴퓨터가 매우 느려질 수도 있으니 코드 실행이 끝난다면 Chrome도 종료시켜주시는 것을 추천드립니다.

### Google 검색해보기
```Python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get('https://www.google.com')


# 태그의 `name`값이 `q`인 것의 web element 찾는다.
search_bar = driver.find_element(By.NAME, 'q')

search_keyword = 'selenium'
# search_bar element에 검색어를 입력한다.
search_bar.send_keys(search_keyword)
# form submit을 실행시킨다 -> 검색이 된다.
search_bar.submit()
```
Google 홈페이지를 띄워보았다면 위의 코드를 작성하면 검색어를 입력하고 검색까지를 자동화해볼 수 있습니다.
![Screen-Recording-2022-04-13-at-4 56 46-PM](https://user-images.githubusercontent.com/78338337/163129241-272205bd-257f-44a4-82d9-a25394c025e8.gif)

<br>

`find_element()`라는 메서드의 첫번째 파라미터는 어떠한 요소로 찾을지를 정할 수 있으며 `By`에 정의된 값들을 사용할 수 있습니다.

```Python
# 태그에 지정된 class명으로 찾는 예시
driver.find_element(By.CLASS_NAME, 'time')

# XPath를 이용해 찾는 예시
driver.find_element(By.XPATH, '//div[@class="time"]')
```
```Python
# By가 지원하는 locator들이 정의된 class
class By(object):
    """
    Set of supported locator strategies.
    """

    ID = "id"
    XPATH = "xpath"
    LINK_TEXT = "link text"
    PARTIAL_LINK_TEXT = "partial link text"
    NAME = "name"
    TAG_NAME = "tag name"
    CLASS_NAME = "class name"
    CSS_SELECTOR = "css selector"
```

----

## 마치며...
이번 글에서 다룬 내용을 요약해보면 다음과 같습니다.
- selenium은 웹 브라우저를 자동화 시킬 수 있는 도구다.
- chrome driver 설치법
- selenium 설치 및 사용법 (google 검색 자동화)

이외에도 selenium은 다양한 함수들이 존재하는데 잘 활용하면 로그인, 제품 구매,영화 예매 등을 자동화하여 빠르게 실행할 수 있는 등 다양한 매크로를 만들 수 있는 아주 유용한 라이브러리 같습니다!
