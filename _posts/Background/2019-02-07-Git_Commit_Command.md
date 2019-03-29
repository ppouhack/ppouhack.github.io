---
layout: post
title: "Git Commit Command"
featured-img: Git_Commit_Command
category: Background
---

# Git

- [Config](#Config)
  + git config
  + git init
- [Local Storage Commit](#Local_Storage_Commit)
  + git status
  + git add
  + git commit
- [Branch](#Branch)
  + git branch
  + git checkout
  + git merge
  + Branch Conflict
- [Remote Storage Commit](#Remote_Storage_Commit)
  + git remote
  + git push
  + push reject
    * git fetch
    * git diff
- [ETC](#ETC)
  + .gitignore
  + log

<a name="Config"/>
## Config
---

### git config
---

Git에서 커밋할 때 마다 기록하는 사용자 이름과 메일 주소를 설정

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git
$ git config --global user.name "ppou"
$ git config --global user.email "ppou@ppou.com"
```

### git init
---

Git 저장소로 사용할 디렉터리 초기화

```apache
$ git init
Initialized empty Git repository in C:/Users/shin/ppou_git/.git/
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
```

<a name="Local_Storage_Commit"/>
## Local Storage Commit
---

### git status
---

저장소의 상태 확인

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ echo print \"hello\" > ./test.py
$ git status
```

```no-highlight
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        test.py

nothing added to commit but untracked files present (use "git add" to track)
```

### git add
---

저장소에 파일 추가

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git add test.py
$ git status
```

```no-highlight
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   test.py
```

### git commit
---

| option       | Description |example|
|:------------:|:-------------:|:-------:|
|-m|commit message 작성|**git commit -m**|
|-a|저장소에 변경된 파일을 모두 커밋|**git commit -a -m**|

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git commit -m "first commit"
```

```no-highlight
[master (root-commit) 12f5135] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 test.py
```

<a name="Branch"/>
## Branch
---

### git branch
---

| Command|Description |
|:-------------:|:-------:|
|**git branch**| 존재하는 branch 확인|
|**git branch [name]**|branch 생성|

현재 존재하는 branch 확인
```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git branch
* master
```

- master 앞에 붙은 `*`은 현재 작업 중인 브랜치를 의미


### git checkout
---

| Option       | Command|Description |
|:------------:|:-------------:|:-------:|
||**git checkout**|branch checkout|
|-b|**git checkout -b**|branch 생성 후 checkout|


```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git branch work_branch
$ git checkout work_branch
Switched to branch 'work_branch'
```

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (work_branch)
$ echo print \"test\" >> ./test.py
$ git commit -a -m "second commit"
[work_branch 9cd3318] second commit
 1 file changed, 1 insertion(+)

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (work_branch)
$ git checkout master
Switched to branch 'master'

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ cat test.py
print "hello"
```

### git merge
---

| Command|Description |
|:-------------:|:-------:|
|**git merge [work_branch]**| 현재 branch에 work_branch를 병합|


```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git merge work_branch
```

```no-highlight
Updating 12f5135..9cd3318
Fast-forward
 test.py | 1 +
 1 file changed, 1 insertion(+)
```

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ cat test.py
print "hello"
print "test"

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git status
On branch master
nothing to commit, working tree clean
```

### branch conflict
---

master branch와 conflict_branch에서 동시에 같은 파일의 같은 곳을 수정하고, 병합하면서 충돌이 발생한다. 

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git branch conflict_branch

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (conflict_branch)
$ cat test.py
print "hello"
print "test"

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (conflict_branch)
$ echo print \"conflict_branch\" >> ./test.py

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (conflict_branch)
$ git commit -a -m "conflict_commit"
[conflict_branch 75ce3e2] conflict_commit
 1 file changed, 1 insertion(+)

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (conflict_branch)
$ git checkout master
Switched to branch 'master'

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ cat test.py
print "hello"
print "test"

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ echo print \"master\" >> ./test.py

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ cat test.py
print "hello"
print "test"
print "master"

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git commit -a -m "master_commit"
[master 29e1707] master_commit
 1 file changed, 1 insertion(+)
```

충돌이 발생하면 `(master) -> (master|MERGING)`으로 변경되며 어떤 파일에서 충돌이 발생했는지 알려준다. 

해당 파일에 접근하면 충돌이 발생한 부분의 시작은 `<<<<<<< HEAD` 충돌의 끝 부분은 `>>>>>>> BRANCH` 경계는 `=========`으로 표시된다. 

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git merge conflict_branch
Auto-merging test.py
CONFLICT (content): Merge conflict in test.py
Automatic merge failed; fix conflicts and then commit the result.


shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master|MERGING)
$ cat test.py
print "hello"
print "test"
<<<<<<< HEAD
print "master"
=======
print "conflict_branch"
>>>>>>> conflict_branch
```

충돌이 발생한 부분은 수정하고 commit 하면 `(master|MERGING) -> (master)`로 변경된다. 

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ cat test.py
print "hello"
print "test"
print "master"

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master|MERGING)
$ git commit -a -m "conflict resolved"
[master cd4f423] conflict resolved

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
```

<a name="Remote_Storage_Commit"/>
## Remote Storage Commit
---

### git remote 
---

| Option       | Command|Description |
|:------------:|:-------------:|:-------:|
||**git remote add [origin] [address]**|로컬 저장소와 원격 저장소를 연결|
|-v|**git remote -v**|로컬 저장소와 연결된 원격 저장소 list|

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git remote add origin https://github.com/ppouhack/test.git

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git remote -v
origin  https://github.com/ppouhack/test.git (fetch)
origin  https://github.com/ppouhack/test.git (push)
```

### git push
---

| Command|Description |
|:-------------:|:-------:|
|**git push [origin] [branch]**|로컬 저장소의 브랜치를 원격 저장소에 업로드|
|**git push [origin] \-\-all**|로컬 저장소의 모든 브랜치를 원격 저장소에 업로드|

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git push origin --all
```

```no-highlight
Enumerating objects: 13, done.
Counting objects: 100% (13/13), done.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (13/13), 1.03 KiB | 211.00 KiB/s, done.
Total 13 (delta 0), reused 0 (delta 0)
To https://github.com/ppouhack/test.git
 * [new branch]      conflict_branch -> conflict_branch
 * [new branch]      master -> master
 * [new branch]      work_branch -> work_branch
```

파일 수정 후 업로드

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ echo remote repository of ppou_git >> ./README.md

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git add README.md

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git commit -m "remote repository add a README.md"
[master 2af97b1] remote repository add a README.md
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git push origin master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 312 bytes | 312.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/ppouhack/test.git
   cd4f423..2af97b1  master -> master
```


### Push Reject
---
여러 사용자들과 함께 사용할 경우 로컬 저장소와 원격 저장소의 간격이 생긴다. 이때 PUSH를 하면 reject가 발생한다. 따라서 다음과 같은 방법을 사용해 저장소 간 동일한 상태를 유지하여 PUSH 해야한다. 


- github에서 일부 파일 변경 후 push
- 로컬 저장소 파일 수정 후 commmit

로컬 저장소와 원격 저장소의 간격이 발생한다. 
```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ echo local_edit >> ./README.md

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git add README.md

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git commit -m "local edit a README.md"
[master 0fec70e] local edit a README.md
 1 file changed, 1 insertion(+)
```

동일한 파일의 동일한 부분을 수정하였기 때문에 push에 실패한다. 

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git push origin master
To https://github.com/ppouhack/test.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/ppouhack/test.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

| Command|Description |
|:-------------:|:-------:|
|**git fetch**|원격 저장소의 commit 정보를 로컬 저장소로 가져옴|


```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git fetch
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), done.
From https://github.com/ppouhack/test
   2af97b1..d45a154  master     -> origin/master
```

branch list 확인 후 현재 branch(master)와 일치하는 branch(origin/master)를 병합

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git branch -a
  conflict_branch
* master
  work_branch
  remotes/origin/conflict_branch
  remotes/origin/master
  remotes/origin/work_branch

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git merge origin/master
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

| Command|Description |
|:-------------:|:-------:|
|**git diff**|원격 저장소와 로컬 저장소 사이의 차이점 확인|

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master|MERGING)
$ git diff
```

```no-highlight
diff --cc README.md
index 2aa86c9,8f2e124..0000000
--- a/README.md
+++ b/README.md
@@@ -1,2 -1,2 +1,6 @@@
  remote repository of ppou_git
++<<<<<<< HEAD
 +local_edit
++=======
+ remote edit
++>>>>>>> origin/master
```

conflict 수정 후 commit

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master|MERGING)
$ cat README.md
remote repository of ppou_git
local_edit

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master|MERGING)
$ git commit -a -m "local/remote conflict resolved"
[master 676d073] local/remote conflict resolved

shin@WIN-O8FOJ7C5O4C MINGW64 ~/ppou_git (master)
$ git push origin master
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 494 bytes | 494.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To https://github.com/ppouhack/test.git
   d45a154..676d073  master -> master
```

<a name="ETC"/>
## ETC
---

### .gitignore
---

[https://www.gitignore.io/](https://www.gitignore.io/) 에서 운영체제, 프로그래밍 언어를 입력하여 .gitignore 파일을 생성한다. 

다음 명령을 실행하면 해당 프로그래밍 언어 프로젝트를 작업할 때 불필요한 파일이 Git 저장소에 추가되지 않는다. 

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/git_tutorial (master)
$ git add .gitignore

shin@WIN-O8FOJ7C5O4C MINGW64 ~/git_tutorial (master)
$ git commit -m "added '.gitignore' file"
[master fffba00] added '.gitignore' file
 1 file changed, 150 insertions(+)
 create mode 100644 .gitignore
```

### Log
---

commit 내역을 확인할 수 있는 명령어

|목표|명령어|
|:---|:-----|
|**git log -p**| 각 커밋에 적용된 실제 변경 내용을 보여줌|
|**git log \-\-word-diff** | diff명령의 실행 결과를 단어 단위로 보여줌|
|**git log \-\-stat** | 각 커밋에서 수정된 파일의 통계 정보를 보여줌|
|**git log \-\-relative-date**| 상대적인 시간의 형식으로 정보를 보여줌|
|**git log \-\-graph**|브랜치 분기와 병합 내역을 아스키 그래프로 보여줌|

```apache
shin@WIN-O8FOJ7C5O4C MINGW64 ~/git_tutorial (master)
$ git log --graph
```

```no-highlight
*   commit e18beec126c3cca5a56bf54a0cef58f1c1870f56 (HEAD -> master)
|\  Merge: 02cb1c0 d533adf
| | Author: ppou <ppou@ppou.com>
| | Date:   Fri Feb 8 16:22:32 2019 +0900
| |
| |     conflict resolved
| |
| * commit d533adf7cce2aea08f9cdaf70d572694bb6e9876 (hotfix)
| | Author: ppou <ppou@ppou.com>
| | Date:   Fri Feb 8 15:57:58 2019 +0900
| |
| |     hotfix
| |
* | commit 02cb1c00e6d8382205ce9d2cb86323539085d05c
|/  Author: ppou <ppou@ppou.com>
|   Date:   Fri Feb 8 15:58:58 2019 +0900
|
|       coolfix
|
* commit fffba0020390d678c55858de0302414f1dade999 (coolfix)
| Author: ppou <ppou@ppou.com>
| Date:   Fri Feb 8 15:38:38 2019 +0900
|
```