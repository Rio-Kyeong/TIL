# Branch

```text
프로젝트 하나는 여러 단위의 하위 모듈들이 존재할 것이고 개발자들은 각각 자신들이 맡은 모듈을 개발하고 최종적으로 합치는 방식으로 
수행한다. 이 경우 개발자들은 각각 전체 모듈을 아우르는 소스 코드를 복사하여 각자 개발하고 마지막에 합치는 방식으로 개발을 수행한다. 
이를 간편하게 도와주는 Git의 기능이 Branch이다.

Branch로부터 개발자들은 각각 자신들의 모듈에 해당하는 이름의 Branch를 분기해오게 되며 이렇게 Branch를 따면 Branch의 소스 코드들이 
복사가 될 것이다. 그리고 개발자들은 각각의 Branch에서 자신들의 모듈을 개발하고 최종적으로 Merge 요청을 내리게 되면 Branch의 관리자가 
확인하고 Branch로 Merge를 시켜주는 방식이 된다.

Branch의 기본 값은 master로 설정되어 있다. 
```

## branch 명령어
### branch 확인
```text
현재 등록된 Branch를 확인
$ git branch
```
### branch 비교
```
$ git log --branches --graph --oneline

--branches : 모든 브랜치들의 커밋정보를 보여준다.
--graph  : 브랜치 정보를 그래프로 보여준다.
--oneline : 그래프를 간단하게 한 줄로 보여준다.
```
![branch](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/branch.PNG)
- `(HEAD -> master)` : 현재 master branch에 checkout 되었으며, master의 최신 커밋은 3753e3f 이다.
- `(exp)` : exp의 최신 커밋은 e9a99c6 이다.
- 아래의 화면 기준으로 `commit 2로부터 분기`되어 commit 3(exp)과 commit 5(master)가 만들어졌고, commit 3으로 부터 commit 4(exp)가 만들어 졌다.
- `master branch`는 commit 1,2,5 존재
- `exp branch`는 commit 1,2,3,4 존재

```
브랜치1과 브랜치2의 차이를 보여준다(브랜치1에는 없고 브랜치2에는 있는 commit을 보여준다)
$ git log [branch 1]..[branch 2]
```
### branch 생성
```
Branch를 생성(현재 branch를 분기해온다)
$ git branch [branch-name]

$ git branch [새로운 브랜치] [분기해 올 브랜치]
```
### branch 이동
```
Branch를 이동
$ git checkout [branch-name]

어떠한 commit의 파일들이 궁금할 때(해당 커밋으로 HEAD가 변경된다)
$ git checkout [commit] 
```
### branch 병합
```
특정 브랜치로 전환하고(checkout), 워킹 디렉토리를 업데이트
$ git merge [branch-name]

ex) exp의 내용을 master로 이동 시키기

1. master branch로 이동
$ git checkout master

2. 병합(반대의 경우도 가능)
$ git merge exp

```
* 병합에는 `Fast-forward방식`과 `Not Fast-forward방식`이 있다.
* `Fast-forward방식`은 별도의 commit을 생성하지 않고, `Not Fast-forward방식`은 merge commit을 만든다. 
[[참고]](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
### branch 삭제
```
Branch를 삭제
$ git branch -d [branch-name]
```

# Stash
``` 
어떤 작업을 하던 중에 다른 요청이 들어와서 하던 작업을 멈추고 다른 브랜치로 변경을 해야할 때,
완료하지 않은 일을 commit하는 것을 껄끄럽기 때문에 stash를 사용한다.
(그냥 checkout을 하게되면 이동한 branch에 작업하던 파일의 영향을 준다)

git stash란 아직 마무리하지 않은 작업을 스택에 잠시 저장할 수 있도록 하는 명령어이다.
```
```
git stash는 버전관리가 되는 파일만 적용이 가능하다.

1. Modified이면서 Tracked 상태인 파일
    - Tracked : 과거에 이미 commit하여 스냅샷에 넣어진 관리 대상 상태의 파일

2. Staging Area에 있는 파일(Staged 상태의 파일)
    - git add 명령을 실행하여 Staged 상태인 파일
```
## stash 명령어
### stash 임시 저장
```
버전 관리 중인 모든 파일의 변경점을 임시로 저장한다.
$ git stash
```
### stash 내용 복원
```
가장 최근 임시 저장한 내용을 복원한다(stash list에 내용은 그대로 존재)
$ git stash apply

가장 최근 임시 저장한 내용을 복원하고 그 내용을 stash list에서 삭제한다.
$ git stash pop
```
### stash 목록 검색
```
임시로 저장된 모든 변경점의 목록을 보여준다.
$ git stash list
```
### stash 내용 삭제
```
가장 최근 임시 저장한 내용을 지웁니다.
$ git stash drop
```
