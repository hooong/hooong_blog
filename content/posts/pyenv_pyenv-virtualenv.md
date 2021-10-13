---
title: "[Python] Pyenv와 Pyenv-virtualenv를 이용한 Python 개발환경 구성"
date: 2021-10-13T21:48:11+09:00
description: "pyenv와 pyenv-virtualenv 치트시트"
tags: ["Python"]
toc:
  auto: false
---

<!--more-->

## Pyenv란?

Pyenv는 여러 버전의 Python을 쉽게 다운로드하고 필요에 따라 전환할 수 있는 도구라고 할 수 있습니다. 예를들어 하나의 머신에서 각기 다른 Python 버전으로 구성된 프로젝트가 있을 경우 pyenv를 이용한다면 프로젝트에 따라 쉽고 빠르게 버전을 전환할 수 있게 도와준다. 



## Pyenv의 동작 방식

pyenv는 shim 실행 파일을 사용해 Python 명령어를 가로채고 PATH에 지정된 Python 버전을 확인하고 해당 Python 버전으로 명령을 우회시키는 방식으로 동작하게 된다고 합니다. 따라서 pyenv를 실행시키기 위해서는 아래와 같이 PATH에 `$(pyenv root)/shims`를추가해주어야 합니다. 

```sh
$(pyenv root)/shims:/usr/local/bin:/usr/bin:/bin
```

예를들어, `python` 이라는 명령어를 실행한다면 `$(pyenv root)/shims` 에서 `python` 이라는 이름을 가진 shim파일을 찾아 실행시키므로써 pyenv에서 지정한 특정 python 버전이 실행되게 된답니다.



## Pyenv 설치

### MacOS

- homebrew를 통해 pyenv를 설치해줍니다.

  ```sh
  $ brew update
  $ brew install pyenv
  ```

- 환경변수를 설정합니다.

  ```sh
  $ echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
  $ echo 'eval "$(pyenv init -)"' >> ~/.zshrc
  $ source ~/.zshrc
  ```

  - `eval "$(pyenv init --path)"` : `$PATH` 앞에 `$(pyenv root)/shims` 를 붙여줍니다.
  - `eval "$(pyenv init -)"` :  자동완성을 지원해줍니다.

### Ubuntu

- pyenv를 사용하기 위한 패키지를 설치한다.

  ```sh
  $ sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
  	libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
  	xz-utils tk-dev
  ```

- pyenv git에서 clone 해온다.

  ```sh
  $ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
  ```

- 환경변수를 설정합니다.

  ```sh
  $ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zprofile
  $ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zprofile
  $ echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
  $ echo 'eval "$(pyenv init -)"' >> ~/.zprofile
  $ source ~/.zprofile
  ```



## 원하는 python 버전 설치 및 사용하기

### 설치 가능한 Python 버전 확인

```sh
$ pyenv install --list

Available versions:
  2.1.3
  2.2.3
	...
	3.9.2
  3.9.4
  3.9.5
  3.9.6
  ...
  stackless-3.4.2
  stackless-3.4.7
  stackless-3.5.4
  stackless-3.7.5
```

### 특정 버전의 Python 설치하기

```sh
$ pyenv install 3.9.6
```

### 설치된 Python 버전 확인하기

```sh
$ pyenv versions
	3.6.2
* 3.9.6 (set by /home/ubuntu/.pyenv/version)
	...
```

### 원하는 버전을 전역으로 설정하기

 ```sh
$ pyenv global 3.9.6
 ```

### 디렉터리별 Python 버전 설정하기

```sh
$ pyenv local 3.6.2
```



## Pyenv-virtualenv

Pyenv-virtualenv는 pyenv를 통해 설치된 python 버전으로 가상환경을 구성해줍니다.

### 설치하기

```sh
$ git clone https://github.com/yyuu/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv
```

### 환경변수 추가

```sh
$ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
$ source ~/.zshrc
```

### virtualenv 생성하기

```sh
$ pyenv virtualenv {python 버전} {virtualenv이름}
# ex) pyenv virtualenv 3.9.6 test_venv-3.9.6
```

### 생성된 virtualenv 확인하기

```sh
$ pyenv virtualenvs
```

### virtualenv 활성화 및 비활성화

```sh
# 활성화
$ pyenv active {virtualenv이름}

# 비활성화 (활성화되어 있는 상태에서 실행)
(test_venv-3.9.6) $ pyenv deactive
```

가상환경을 필요할때마다 활성화를 할수도 있지만 python을 디렉터리별로 버전을 설정한 것처럼 활성화를 자동으로 시킬 수 있습니다.

```sh
$ pyenv local test_venv-3.9.6
```



### 참고자료

- https://github.com/pyenv/pyenv
- https://en.wikipedia.org/wiki/Shim_(computing)

