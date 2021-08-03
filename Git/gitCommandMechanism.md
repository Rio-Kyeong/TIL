# Git Command Mechanism
## git add
```
1. git add 명령어를 수행 
2. .git 디렉토리의 ./index 폴더에 add 한 파일의 정보가 기록
3. ./object/[디렉토리명]/[파일명] 에 해당 파일의 정보가 저장

./objects/cc/2b59e62e5a1881965729bfce78cba65f43d973 
SHA-1 알고리즘을 이용한 40글자의 해시 이용
앞의 두글자는 디렉토리명, 나머지 38글자는 파일명

git add를 하면 변경된 내용을 실제 저장소에 저장(commit)하기 전에, 대기 상태 공간(stage area)에
올려두는데, 여기서 ./index  디렉토리가 대기 상태 공간(stage area)이다.

git은 어떠한 파일을 저장할 때, 파일의 이름이 다르더라도 내용이 동일하다면, 동일한 object 파일을 가리킨다.
(중복을 회피할 수 있다)
```
## git commit
```
commit 시, add와 동일하게 우리가 생성한 버전도 마치 파일의 내용처럼 ./object에 생성된다
(commit도 git의 내부에서는 하나의 object(객체)로 취급된다)

object 파일 구성
tree, author(저자), commiter(커밋을 한 사람), commit message, parent(이 전의 commit을 보여줌)

버전에 적혀있는 tree를 통해, 버전이 만들어진 시점의 프로젝트 폴더에 대한 상태를 알 수 있다.
이를 스냅샷(Snapshot) 이라고 한다.
```
![object](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/object.PNG)
```
object의 형태
1. conmmit : 작성자, 커밋 실행자, 날짜, 로그 메세지, tree 객체 등에 대한 정보를 가지고 있다.
2. tree : 디렉토리명과 파일의 내용인 Blob(블랍)에 대한 정보를 가지고 있다.
3. Blob : 텍스트, 이미지, 이진 파일 등 다양한 형식의 파일을 저장할 수 있으며, 파일의 메타정보를 제외한 파일의 전체 내용이 저장된다.
```
## git status
<pre>
<b>git이 commit할 파일이 있음/없음을 알 수 있을까?</b>

우리가 commit을 하면, commit에 대한 object 파일이 생성이 되고, 내부에는 tree가 존재한다.
이때 이 tree의 내용과 index의 내용이 일치한다면, 현재 커밋할 내용이 없다고 알 수 있다.
즉, commit 이후에 저장소, index, 프로젝트 폴더가 정확하게 일치할때 git status는 더이상 commit할 것이 없다고 알려준다.
</pre>
<pre>
<b>파일을 수정하고, git status를 하면 git은 어떻게 수정된 파일이 있음을 알 수 있을까?</b>

예를 들어 f2.txt를 수정했다고 한다면, index에 적혀있는 f2.txt의 해시값과, f2.txt 파일의 내용이 만들어내는 해시값이 다른 경우, 수정되었음을 알수 있다.
</pre>
<pre>
<b>git add를 한 후, commit 대기 상태임을 어떻게 알 수 있을까?</b>

예를 들어, index에서 f2.txt와 최신 commit의 tree가 가리키는 f2.txt의 내용이 다르다면 f2.txt는 index에 add 되어 commit 대기 상태임을 알 수 있다.
</pre>

## Work Tree, index, repository의 관계
![관계](https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/gitCommand.png)
* Working Directory : git 작업하는 디렉토리(.git 외부에 프로젝트 폴더)
* Staging Area : index
