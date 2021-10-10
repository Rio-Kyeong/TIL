# Git
``` 
Version Control System
```
## Linux 기본 명령어
```text
현재 어떤 디렉토리경로에 있는가를 절대경로로 표시
$ pwd
/c/Users/Administrator
```
```text
이동하려는 디렉토리로 이동(C드라이브 dev폴더로 이동)
'~' 표시는 홈 디렉토리를 뜻한다.
$ cd C:/dev
```
```text
한 단계 상위 디렉토리로 이동
$ cd ..
```
```text
디렉토리 생성(gitfth 폴더 생성)
$ mkdir gitfth
```
```text
파일 내용 출력
$ cat f1.txt
souce : 1
```
```text
파일, 디렉토리를 복사
$ cp file1 file2
```
```text
파일, 디렉토리 삭제
$ rm [file name]
하위 디렉토리까지 모두 삭제
$ rm -r [file name]
```
```text
현재 폴더의 하위 디렉토리의 리스트를 보여준다(-al)
$ ls [옵션]
eclipse/  workspace/
```

## vim(text editor) 사용하기
```text
vim 명령어를 통하여 실행할 수 있다.(파일이름을 생략해도 상관없음)
$ vim [file-name]
```
* vim에서는 `편집모드`와 `명령모드`가 있다.
* **i**키를 입력해서 `편집모드`로 들어갈 수 있다.
* **esc**키를 입력해서 `편집모드`를 빠져나올 수 있다.
* 명령모드에서 :(콜론)을 누르면 `exmode`가 된다.
* `exmode`에서는 파일열기, 저장, 닫기, 종료등을 사용할 수 있다.
* 종료(**:q**), 저장(**:w**), 저장 후 종료(**:wq**), 열기(**:e [file-name]**)
## Git 명령어
### 저장소 생성
```text
새로운 로컬 저장소를 생성하고 이름을 정합니다(이름을 정하지 않으면 현재 디렉토리를 저장소로 설정)
$ git init [project-name]
```
### 로컬디스크에 원격 저장소(remote repository) 만들기
```text
일반적인 깃 저장소(원격 저장소가 따로 존재하는 로컬 저장소)는 'git init'으로 생성하지만 
원격(서버) 저장소라면 bare-repository로 생성해야 한다. 

bare-repository는 워킹 트리가 없고 변경사항만 추적하는 저장소를 말한다.
bare-repository는 어떠한 작업도 하지않는다(작업이 불가능하다)
```
```text
새로운 원격 저장소를 생성하고 이름을 정합니다(이름을 정하지 않으면 현재 디렉토리를 remote directory로 설정)
$ git init --bare [project-name] 

생성한 원격 저장소 경로를 검색
$ pwd
/c/dev/git/remote

로컬 저장소에 가서 원격 저장소를 지정(추가)한다.
$ git remote add origin /c/dev/git/remote

원격 저장소가 추가되었는지 확인
$ git remote -v
origin  C:/dev/git/remote (fetch)
origin  C:/dev/git/remote (push)

로컬 저장소 master baranch에서 push 명령을 내리면 자동으로 원격 저장소 master branch로 push를 한다는 의미를 가진다.
(두 번째부터는 git push로 간결하게 사용가능하다)
$ git push --set-upstream origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 201 bytes | 201.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To C:/dev/git/remote
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.

원격 저장소 삭제
$ git remote remove [remote repository]
```
### 저장소 복제해서 만들기 
```text
git clone은 git pull과 비슷하지만 클라이언트 상에 아무것도 없을 때 서버의 프로젝트를 내려받는 명령어이다.
저장소의 내용을 다운로드받고 자동으로 init도 된다.

기존 프로젝트의 모든 커밋 내역을 가져와 저장소를 만든다(로컬 저장소로 복제)
$ git clone [클론할 저장소의 주소] [저장소로 만들 폴더 명(자동생성)]

현재 디렉토리를 저장소로 만들고 클론한다.
$ git clone [클론할 저장소의 주소] . 
```
### 환경 설정
* 모든 로컬 저장소에 적용할 사용자 정보를 설정한다(최초 1회)
```text
$ git config --global user.name "[name]"
```
```text
$ git config --global user.email "[email address]"
```
### 변경점 저장하기
* `파일이 수정된 후에도 꼭 add를 하고 commit을 해야한다.`
```text
커밋할 수 있는 새로운 파일과 수정된 파일의 목록을 보여준다(상태확인)
아직 git add나 git commit을 하지 않았으면 파일은 Untracked 상태
$ git status
```
<pre>
수정하였으나 아직 stage하지 않은 파일의 변경점을 보여줍니다.
working copy의 내용과 index의 내용을 비교할 수 있다.
$ git diff [옵션]
diff --git a/f2.txt b/f2.txt
index e77ea5a..ed48ea0 100644
--- a/f2.txt ◀ 바뀌기 전 파일
+++ b/f2.txt ◀ 바뀐 후 파일
<span style="color:blue">@@ -1 +1 @@</span>
<span style="color:red">-souce : 2</span>  ◀ --- a/f2.txt의 파일내용
<span style="color:green">+f2.txt : 2</span> ◀ +++ b/f2.txt의 파일내용
</pre>
```text
커밋을 준비하기 위해 파일을 stage(커밋대기)한다.
(CRLF 관련 경고 문구가 나타날 수 있는데 이는 무시해도 무관하다)
$ git add [file]
```
```text
stage한 내용을 커밋으로 영구히 저장한다.
$ git commit -m"[message]"

이전에 썼던 커밋 메시지를 바꿀 수 있다(push 전에만 사용) 
$ git commit --amend
```
### 변경 기록 검토
```text
프로젝트 내 파일의 변경 기록을 살펴보고 검토한다.
(commit 직후에 사용했기 때문에 commit 내역이 보여진다)
$ git log [옵션]
commit e8891319ef96d3caba3627c398bfd7ef36fa91cb (HEAD -> master)
Author: kyeongwoo <drd9811@naver.com>
Date:   Sat Jul 24 13:38:08 2021 +0900

    "message"
```
### 커밋 되돌리기
* `실수한 내용을 지우고 기록을 바꾼다`<br/>
(공유 이후 리셋x 공유하기 전 내 컴퓨터에 있는 버전에 대해서만 리셋을 한다)
* reset에 대한 자세한 정보는 이 곳을 <a href="https://github.com/RyuKyeongWoo/TIL/blob/main/Git/gitMechanism2.md">참고</a>
```text
모든 변경점과 기록을 버리고 특정 커밋으로 되돌아간다.(HEAD를 적으면 특정 커밋을 가장 최신 커밋상태로 돌려준다)
$ git reset --hard [commit]
$ git reset --hard c3017ff6b4037d632b885d3445dc17db7dccd8b4

현재 파일의 변경 사항은 그대로 두고 '[commit]' 이후의 모든 커밋 내용을 되돌린다.
$ git reset [commit]
```
<pre>
ORIG_HEAD를 참조하여 reset된 기록을 다시 되돌려준다. 
$ git reset --hard ORIG_HEAD 
</pre>
```text
커밋을 취소하면서 새로운 버전을 생성한다.
$ git revert [commit]
```


