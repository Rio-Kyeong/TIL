# tags(release)
```
커밋을 참조하기 쉽도록 알기 쉬운 이름을 붙이는 것을 말한다.
한 번 붙인 태그는 브랜치처럼 위치가 이동하지 않고 고정되며 나중에 시간이 지남 다음에도 그 버전이 어떤 커밋에 해당하는 알 수 있다.
```
## tag 원리
```
refs/tags/[tag name] : 태그를 생성하면 만들어지며, 특정 Object id(tag한 commit)를 가리킨다.
branch의 원리와 tag의 원리는 거의 똑같다. 차이로는 branch는 최신 commit id가 바뀌지만 tag는 바뀌지 않는다.
```
## tag 명령어
### tag 사용
```
일반 태그(light weight tag) 생성 - commit을 쓰지 않으면 현재 HEAD가 가리키는 branch의 최신 커밋을 tag로 생성한다.
$ git tag [tag namne] [commit]
```
```
주석 태그(annotated tag) 생성
$ git tag -a [tag name] -m "messege"
# git tag -a v1.0.3 -m"Release version 1.0.3"
```
### tag 조회
```
tag를 확인한다.
$ git tag
$ git tag -v [tag name]
```
### tag 원격 저장소에 올리기
```
로컬 저장소에서 만든 tag를 원격 저장소로 보내기
$ git push [remote name][tag name]

모든 tag 올리기
$ git push --tags
```
### tag 삭제
```
tag 삭제한다. 
git tag -d [tag name]

원격 저장소에 올라간 tag 삭제하기
git push [remote name] :[tag name]
```
### 해당 tag로 이동
```
tag에 해당하는 commit으로 이동한다(해당 tag가 가리키는 커밋으로 HEAD가 변경된다)
$ git checkout [tag name]
```
