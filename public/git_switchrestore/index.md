# [Git]switch + restore (= checkout?)


Git의 switch와 restore에 대하여...

<!--more-->
<br>

## 개요

Git 2.23.0 버전 이하에서 `checkout` 명령어 아래 두 가지의 기능을 동시에 가진다.

- branch를 전환
- working tree의 파일을 복원

이처럼 하나의 명령어가 다른 두 가지 이상의 기능을 동시에 가지게 되면 명시적이지 않을 수 있다. 따라서 Git은 2.23.0버전에서 이 두 가지의 기능을 별도의 명령어로 나누어서 좀 더 명령어를 명시적으로 사용할 수 있게끔 만들었다.

```
 * Two new commands "git switch" and "git restore" are introduced to
   split "checking out a branch to work on advancing its history" and
   "checking out paths out of the index and/or a tree-ish to work on
   advancing the current history" out of the single "git checkout"
   command.
```

[Git v2.23.0 Release Notes](https://public-inbox.org/git/xmqqy2zszuz7.fsf@gitster-ct.c.googlers.com/)

<br>

참고로 현재 *Git(2.24.3)*버전 기준으로 `$ git --help` 로 지원하는 명령어를 살펴보면 `checkout` 을 찾아볼 수 없다.

```shell
start a working area (see also: git help tutorial)
   clone     Clone a repository into a new directory
   init      Create an empty Git repository or reinitialize an existing one

work on the current change (see also: git help everyday)
   add       Add file contents to the index
   mv        Move or rename a file, a directory, or a symlink
   restore   Restore working tree files
   rm        Remove files from the working tree and from the index

examine the history and state (see also: git help revisions)
   bisect    Use binary search to find the commit that introduced a bug
   diff      Show changes between commits, commit and working tree, etc
   grep      Print lines matching a pattern
   log       Show commit logs
   show      Show various types of objects
   status    Show the working tree status

grow, mark and tweak your common history
   branch    List, create, or delete branches
   commit    Record changes to the repository
   merge     Join two or more development histories together
   rebase    Reapply commits on top of another base tip
   reset     Reset current HEAD to the specified state
   switch    Switch branches
   tag       Create, list, delete or verify a tag object signed with GPG

collaborate (see also: git help workflows)
   fetch     Download objects and refs from another repository
   pull      Fetch from and integrate with another repository or a local branch
   push      Update remote refs along with associated objects
```

<br>

## switch

`switch` 명령어는 기본적으로 기존에 branch를 이동할 때 사용하던 `git checkout <branch>` 와 동일한 기능을 수행한다.

### SYNOPSIS

```shell
$ git switch [<option>] [--no-guess] <branch>
$ git switch [<option>] --detach [<start-point>]
$ git switch [<option>] (-c|-C) <new-brach> [<start-point>]
$ git switch [<option>] --orphan <new-branch>
```

### 기본 사용 예시

```shell
# issue1이라는 branch를 생성하고 전환
$ git switch -c issue1

# master branch로 전환
$ git switch master

# HEAD 이전 커밋의 상태로 이동
$ git switch --detach HEAD~

# 현재 master branch라면 master branch상태에서 issue1 branch가 새로 생성되고 전환
$ git switch switch -C issue1
```

### 옵션 설명

- `<start-point>`
  - 특정 위치에서 branch를 따낸다.
  - default로는 `HEAD` 이다.
- `-c | --create <new-branch>`
  - `<start-point>` 의 위치에서 새로운 branch를 생성하고 전환한다.
- `-C | --force-create <new-branch>`
  - `<new-branch>` 가 이미 존재한다면 reset된 상태의 branch를 생성하고 전환한다.
- `--detach`
  - 특정 커밋으로 상태를 전환한다. (기존 커밋은 남아있는 상태)
- `-m | --merge`
  - Three-way merge를 진행한 뒤 branch를 만들고 전환한다.
- `--orphan`
  - 새로운 orphan branch를 생성한 뒤 전환한다.
- `-t | --track`
  - `-c` 옵션과 함께 사용할 때, upstream으로 지정할 원격 저장소의 브랜치를 지정할 수 있다.
  - `-c`가 없다면 원격 branch에서 새로운 branch를 따서 생성하고 전환한다.
  - `--no-track`로 지정할 경우에는 upstream을 구성하지 않는다.
- `--guess, --no-guess`
  - branch를 생성할 때 원격 저장소에 동일한 이름이 있다면 자동으로 upstream으로 지정하여 매칭한다.
  - `--guess`가 default이고 `--no-guess`는 자동으로 매칭이 되지 않게끔 설정한다.

이외의 더 많은 옵션은 [git-scm-switch](https://git-scm.com/docs/git-switch)에서 살펴볼 수 있다.

<br>

## restore

`restore` 명령어는 특정 working tree의 파일로 돌리는 기능을 수행한다.

### SYNOPSIS

```shell
$ git restore [<option>] [--source=<tree>] [--staged] [--worktree] [--] <pathspec>...
$ git restore [<option>] [--source=<tree>] [--staged] [--worktree] --pathspec-from-file=<file> [--pathspec-file-nul]
$ git restore (-p|--patch) [<option>] [--source=<tree>] [--staged] [--worktree] [--] [<pathspec>...]
```

### 기본 사용 예시

```shell
# 현재 디렉터리('.')를 HEAD의 working tree로 복원
$ git restore .

# 현재 디렉터리('.')를 특정 커밋('943453b')의 working tree로 복원
$ git restore -s 943453b .

# 현재 디렉터리('.')를 특정 branch(issue1)의 working tree로 복원
$ git restore -s issue1 .

# 현재 디렉터리('.')를 HEAD~의 index를 복원
$ git restore -s HEAD~ -S .

# 현재 디렉터리('.')를 특정 위치의 index와 working tree 모두 복원
$ git restore -s HEAD~ -SW . 

# 'test.txt'을 unstageing 상태로 복원
$ git restore --staged test.txt
```

### 옵션 설명

- `-s <tree> | --source=<tree>`
  - 특정 `<tree>`(commit, branch) 위치를 지정한다. 
- `-S | --staged`
  - 특정 위치의 `index`만을 복원한다.
  - `-s` 옵션이 지정되지 않으면 기본으로 `HEAD`로 지정된다.

- `-W | --worktree`
  - 특정 위치의 `working tree`를 복원한다.
  - `-S` 와 `-W`를 모두 지정하면 `index`와 `working tree` 모두 복원한다.
  - `-S`와 동일하게 `-s`옵션이 지정되지 않으면 기본으로 `HEAD`로 지정된다.
- `-p | --patch`
  - 대화 모드로 명령어를 실행한다.

이외의 더 많은 옵션은 [git-scm-restore](https://git-scm.com/docs/git-restore)에서 살펴볼 수 있다.

<br>

# 마무리하며...

Git의 `--help`에서도 `checkout` 명령어를 삭제한 것을 보니 `switch`와 `restore`의 사용을 권장하고 있다는 것을 알 수 있었고, 본인도 기존에 `checkout` 명령어를 사용하였기에 `switch`와 `restore`를 최대한 의식적으로 사용해보고 있는데 확실히 기능이 분리되어 명시적으로 사용할 수 있어 좋다고 느껴졌다. 이 글을 읽으시는 여러분들도 익숙한 `checkout`을 대신하여  `switch`와 `restore`를 사용해보는 것을 추천합니다!

<br>



