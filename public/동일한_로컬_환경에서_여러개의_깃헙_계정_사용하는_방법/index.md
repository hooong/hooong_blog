# [Git] 동일한 로컬 환경에서 여러개의 깃헙 계정 사용하는 방법

하나의 맥북에서 두 개의 깃헙 계정을 사용하는 방법에 대한 정리

<!--more-->
하나의 맥북에서 다른 두 개의 깃헙 계정을 사용해 푸시를 하고 싶은 상황이 생겨 알아보니 ssh-key를 사용하면 가능하였다. 



## 1. ssh-key 생성하기

```shell
$ cd ~/.ssh

$ ssh-keygen -t rsa -C "{github 계정 이메일}"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): {key 이름 지정}

$ ls
-rw-------  1 hongseokjun  staff   411B  3  7 00:35 id_rsa_work
-rw-r--r--  1 hongseokjun  staff   102B  3  7 00:35 id_rsa_work.pub
-rw-------  1 hongseokjun  staff   411B  3  7 00:35 id_rsa_persnal
-rw-r--r--  1 hongseokjun  staff   102B  3  7 00:35 id_rsa_persnal.pub
```

- `~/.ssh` 디렉토리로 이동 후 두 번째 명령어를 실행하고 `{key 이름 지정}` 부분에 원하는 key 이름을 입력해준다. (ex. id_rsa_work)

- 위의 과정을 필요한 계정만큼 반복한다. (여기서는 두 개를 예로 듦.) 끝이나면 `id_rsa_work`, `id_rsa_work.pub`, `id_rsa_persnal`, `id_rsa_persnal.pub`라는 이름의 파일들이 생성된 것을 확인한다.



## 2. ssh-agent에 ssh-key 등록

```shell
$ ssh-add -K ~/.ssh/id_rsa_work
$ ssh-add -K ~/.ssh/id_rsa_persnal
```



## 3. GitHub에 공개키 등록

1. 로컬에서 생성한 공개키를 복사한다.

   ```shell
   $ pbcopy < ~/.ssh/id_rsa_work.pub
   ```

2. github에 ssh-key에 해당하는 계정으로 로그인

3. github 계정의 `settings - SSH and GPG keys` 메뉴를 클릭한다.
<div align="center">
<img width="200" alt="스크린샷 2021-03-07 오후 10 58 53" src="https://user-images.githubusercontent.com/78338337/110242291-c1510100-7f98-11eb-9f01-5a886cf0e5bd.png">
</div>
3. `New SSH key`를 클릭
<div align="center">
<img align="center" width="600" alt="스크린샷 2021-03-07 오후 11 01 41" src="https://user-images.githubusercontent.com/78338337/110242364-1856d600-7f99-11eb-9a87-9144de67ca65.png">
</div>
4. `title`에는 key에 대한 이름을 지정하고 `key`에 1번에서 복사한 내용을 붙여넣어 준다.
<div align="center">
<img align="center" width="500" alt="스크린샷 2021-03-07 오후 11 02 59" src="https://user-images.githubusercontent.com/78338337/110242402-450aed80-7f99-11eb-9aa6-80e717fdb840.png">
</div>
5. 필요한 계정만큼 반복해준다.



## 4. ssh config

```shell
$ cd ~/.ssh
$ vi config			# 없다면 생성

# 아래 내용을 입력
Host github.com-work
	HostName github.com
	User work
	IdentityFile ~/.ssh/id_rsa_work
	
Host github.com-persnal
	HostName github.com
	User persnal
	IdentityFile ~/.ssh/id_rsa_persanl
```

- Host : 저장소를 구분하는 일종의 key
- IdentityFile : ssh-key를 지정



- ssh 테스트 해보기

```shell
$ ssh -T git@github.com-work
Hi {GithubID}! You've successfully authenticated, but GitHub does not provide shell access.
```

- 아래 문구가 뜬다면 정상이다.



## 5. 특정 프로젝트의 git config

```shell
$ vi {프로젝트}/.git/config

...
[user]
    email = {푸시할 계정의 이메일}
```

- `git`의 global config가 설정되어있는 경우, 위와 같은 설정을 해주어야 해당 계정으로 푸시를 할 수 있음.



## 6. remote 설정

github에서 `git ssh`로 clone을 받을 수 있다. : `git@github.com-work:work/{프로젝트}.git`

또는 이미 `remote`가 지정되어 있는 상태라면 아래 명령어를 통해 ssh로 바꿀 수 있다.

```shell
$ git remote set-url origin git@{Host}:{User}/{프로젝트}.git

ex. git remote set-url origin git@github.com-work:work/{프로젝트}.git
```





- 참고

https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

https://velog.io/@sonypark/GitHubSSH%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%B4-%EC%97%AC%EB%9F%AC%EA%B0%9C%EC%9D%98-%EA%B9%83%ED%97%88%EB%B8%8C-%EA%B3%84%EC%A0%95-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-6mk3iesh0u


