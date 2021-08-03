# Rebase

<pre>
rebase는 말 그대로 <b>commit의 base를 다시(re) 정하는 작업</b>이다.

<b>git rebase</b>를 사용해 merge를 한다면 <b>git merge</b>로 merge하는 것보다 커밋 히스토리가 훨씬 깔끔하게 남기 때문에 다른 사람들의 작업을 보기가 편해진다.
</pre>
## Rebase 사용법
<pre>
# 내가 옮기고 싶은 커밋들이 있는 브랜치(여기서는 dev 브랜치)로 이동한다.
$ git checkout dev

# commit의 base를 다시(re) 정한다.
$ git rebase master
- <b>master와 dev 브랜치의 공통 조상 커밋부터 dev 브랜치까지</b>의 모든 커밋의 base를 master 브랜치의 위치로 바꾼다.
</pre>
## Rebase와 Merge의 차이점
![mainBranch](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/mainBranch.PNG)
```
C1은 dev와 master branch의 조상 커밋이다.
```
### Merge 커밋 히스토리
![merge](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/merge.PNG)
```
$ git checkout master
$ git merge dev

2개의 커밋을 포인팅하는 새로운 커밋이 생기고 이를 브랜치가 가리키게 된다.
여러명이 작업을 하고 있는 경우에 계속 git merge를 통해서 할 경우에 지저분한 커밋 히스토리가 생긴다(가독성이 떨어진다)
```
### Rebase 커밋 히스토리
![rebase](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/rebase.PNG)
```
$ git checkout dev
$ git rebase master

이 다음에 fast-forward를 시키기 위해 merge 명령어를 사용
```
![fast-forward](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/fast-forward.PNG)
```
$ git checkout master
$ git merge dev

깔끔한 커밋 히스토리가 남는다.

1. 내가 옮기고 싶은 커밋들이 있는 브랜치(여기서는 dev 브랜치)로 이동한다. (git checkout dev)
2. 커밋을 옮겨갈 목적지 브랜치(여기서는 master 브랜치)에는 없는, dev 브랜치에서만 있었던 변경사항 커밋들(여기서는 C4, C5)을 로컬에 저장한다.
3. 현재 브랜치를 옮겨갈 브랜치와 동일한 상태로 만든다. (git reset --hard master)
4. 로컬에 저장했던 각 커밋을 하나하나 dev에 적용시킨다.

4번을 진행하는 동안 master branch와 CONFLICT(충돌)가 뜬다면 rebase가 중지되고 직접 CONFLICT를 해결해야한다.

# CONFLICT(충돌) 파일확인 
$ git status

# 파일 수정 후 add 
$ git add [conflict file]

# 다시 rebase
$ git rebase --continue
```

