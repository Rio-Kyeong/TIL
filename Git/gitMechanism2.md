# Git Mechanism2
## 1. HAED
```
HEAD는 현재 Checkout된 commit을 가리키는데 그 중에서도 작업트리의 가장 최근 커밋을 가리킨다.

1. git init 명령어를 통해서 로컬 저장소를 생성하면 HEAD 파일이 자동으로 생성된다.
2. HEAD 파일은 ref : refs/heads/master(master 파일)을 가리킨다.
3. master 파일은 가장 최근 commit한 Object id 값을 가리킨다.

HEAD 파일을 통해서 알 수 있는 것
- git log 명령어를 사용했을 때 최신 커밋을 알 수 있다.
- 그 최신 커밋의 parent를 통해서 이전 커밋을 탐색해나갈 수 있다.
- git에서 branch는 refs 폴더 하위에 있는 파일을 의미한다.

HEAD는 branch의 이름을 가리킨다.(HEAD -> master)
```
## 2. branch 충돌
<pre>
1. 서로 다른 파일을 가진 경우 
    - 병합을 해도 충돌이 일어나지 않는다.

2. 서로 같은 파일을 가진 경우
    - 같은 파일이더라도 <b>서로 다른 위치를 작업한 경우</b>에는 병합을 해도 충돌이 일어나지 않는다.

    - 같은 파일일 때 <b>서로 같은 위치를 작업한 경우</b>에는 병합을 하면 충돌이 일어난다.
</pre>
### 충돌이 일어났을 때(conflict)

- 충돌이 일어난 경우 아래와 같은 메시지가 뜬다.</br>
![conflict](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/text.PNG)</br>

- git status를 통해 충돌이 일어난 파일을 찾을 수 있다. 이런 경우 충돌한 파일을 수정해야 한다.</br>
![file](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/conflict.PNG)</br>
    ```
    '<<<<<<< HEAD' 부터 '======='(구분자) 사이의 구간이 현재 체크 아웃된 master 브랜치 코드의 내용이다.
    '======='(구분자) 부터 '>>>>>>> exp' 사시의 구간이 병합하려는 대상인 exp 브랜치의 코드 내용이다.

    서로 같은 위치를 작업한 두개의 코드 내용을 알맞게 병합한 후에 특수기호를 제거해주면 된다.
    작업이 끝나면 파일을 add를 시켜서 충돌 작업을 끝냈다는 것을 깃에게 알려주면 된다.
    ```
## 3. reset & checkout 되돌리기
### reset
```
ORIG_HEAD를 참조하여 reset된 기록을 다시 되돌려준다.
$ git reset --hard ORIG_HEAD 
```
```
git은 웬만하면 어떠한 정보도 지우지 않는다.
(파일이 정말 많아지면 가비지 컬렉션으로 인해 지워질 수 있다)

ORIG_HEAD(오리그 헤드) : 정보를 잃어버릴 수 있는 가능성이 있는 명령어를 사용하면 깃은 현재 최신 커밋을 오리그 헤드에 기록해놓은 후 명령어를 사용한다.

log/refs/heads/master : master branch의 모든 로그들을 기억한다.
```
### checkout
```
HAED가 해당 [commit]을 가리키며, 해당 [commit]은 최신 커밋으로 변경된다.(detached 되어있는 상태)
다시 branch를 바꿔주면 이 내용들은 사라지고 해당 branch의 내용으로 바뀐다.
# git checkout [commit]
```
## 4. working copy & index & repository
![reset](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/reset.PNG)
### reset option
```
repository의 commit 내용만 이전 내용으로 초기화한다.
$ git reset --soft

repositroy의 commit 내용과 index의 add 내용만 이전 내용으로 초기화한다.
$ git reset --mixed

repositroy와 index와 working directory의 작업 내용 모두 초기화한다.
$ git reset --hard 
```
## 5. merge & conflict
```
병합(merge)을 하던 도중 branch 충돌이 난 경우 병합작업을 해야하는데, 병합을 전문적으로 하는 도구를 사용하여 더욱 편리하게 작업을 할 수 있다.
```
### use KDiff3 tool
<pre>
1. <a href="https://sourceforge.net/projects/kdiff3/files/latest/download">KDiff3 Download</a>
2. KDiff3 환경설정
    $ git config --global --add merge.tool kdiff3
    $ git config --global --add mergetool.kdiff3.path "C:/dev/KDiff3/kdiff3.exe"
    $ git config --global --add mergetool.kdiff3.trustExitCode false
3. git에서 KDiff3 실행
    $ git mergetool
4. 병합작업을 마치고 저장 후 KDiff3를 종료하면 merge와 add가 자동으로 된다
</pre>
## 6. 3 way merge 동작과정
![3WayMerge](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/3waymerge.PNG)
```
merge 과정에서 3-way merging 기법이 동작한다.

situation 1 - Base 코드를 수정한 Other를 따라서 아무것도 표시하지 않는다.
situation 2 - 모두 코드가 같으므로 B코드를 표시한다.
situation 3 - 모두 코드가 다르므로 conflict(충돌)이 일어난다.
situation 4 - Base 코드를 수정한 Me를 따라서 아무것도 표시하지 않는다.
```
