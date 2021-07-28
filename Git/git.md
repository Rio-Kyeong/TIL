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
* 명령모드에서 Shift + :(콜론)을 누르면 `exmode`가 된다.
* `exmode`에서는 파일열기, 저장, 닫기, 종료등을 사용할 수 있다.
* 종료(**:q**), 저장(**:w**), 저장 후 종료(**:wq**), 열기(**:e [file-name]**)
## Git 명령어
### 저장소 생성하기
```text
새로운 로컬 저장소를 생성하고 이름을 정합니다(이름을 정하지 않으면 .git만 생성)
$ git init [project-name] 
```
```text
기존 프로젝트의 모든 커밋 내역을 가져와 저장소를 만든다(로컬 저장소로 복제)
$ git clone [클론할 저장소의 주소]
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
(commit 하기전 마지막으로 어떻게 변경 되었는가를 확인할 수 있다)
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
```text
모든 변경점과 기록을 버리고 특정 커밋으로 되돌아간다.(HEAD를 적으면 가장 최신 커밋상태로 돌아간다.)
$ git reset --hard [commit]

현재 파일의 변경 사항은 그대로 두고 '[커밋]' 이후의 모든 커밋 내용을 되돌린다.
$ git reset [commit]
```
```text
커밋을 취소하면서 새로운 버전을 생성한다.
$ git revert [commit]
```
