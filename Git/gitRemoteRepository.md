# Remote Repository
<pre>
<b>직접 자신의 서버(My Server)를 이용해서 원격 저장소를 운영</b>하는 방법과
<b>온라인 서비스(Github)를 제공하는 서버를 이용해서 원격 저장소를 운영</b>하는 방법이 있다.
</pre>
## 원격 저장소 등록하기(Github) - 최초 1회
### 원격 저장소를 만들고 그 후 로컬 저장소에 클론해서 작업진행
```
- clone을 사용하여서 깃허브 원격 저장소를 복제해온다(자동으로 로컬 저장소에 원격 저장소를 연결해준다)
- HTTPS를 이용한 clone 방식과 SSH(Secure Shell)을 이용한 clone 방식이 있다.

$ git clone [클론할 저장소의 주소] [저장소로 만들 폴더 명(자동생성)]

$ git cd [저장소로 생성 된 폴더]

(파일 작업)

$ git add .

$ git commit -m"[message]"

$ git push
```
### 이미 로컬 저장소가 있고, 로컬 저장소에서 해오던 작업을 원격 저장소에 올려서 작업진행
```
현재 로컬 저장소에 원격 저장소를 연결하고 원격 저장소에는 origin이라는 이름을 부여한다.
$ git remote add origin [깃허브 원격 저장소 주소]

원격 저장소 생성 확인
$ git remote -v

(파일 작업)

$ git add .

$ git commit -m"[message]"

원격 저장소에 파일 올리기(두번 째부터는 git push로 간결하게 사용가능)
$ git push -u origin master

원격 저장소 삭제
$ git remote remove [remote repository]
```
## 동기화 하기(Github)
```
여러 컴퓨터를 이용하여 작업할 경우 시나리오

computer1
1. 최초 클론으로 원격 저장소의 내용을 가져온다.
2. 작업하고 작업한 내용을 push
6. pull을 통해 로컬 저장소 업데이트
7. 작업하고 작업한 내용 push
   
   (반복)

computer2
3. 최초 클론으로 원격 저장소의 내용을 가져온다.
4. pull을 통해 로컬 저장소 업데이트
5. 작업하고 작업한 내용 push 
```
## Secure Shell(SSH)
<pre>
SSH키를 만들고 Github에 등록하는 법은 이 곳을 <a href="https://www.lainyzine.com/ko/article/creating-ssh-key-for-github/">참고</a>

HTTPS를 사용하여 Push나 Pull을 하려고 하는 경우에는 사용자의 Username과 Password를 물어본다.
하지만 SSH는 한 번의 인증을 통해 자동 로그인을 할 수 있다.

- id_id_ed25519 : private key(개인키), 자신의 로컬 컴퓨터에 저장
- id_id_ed25519.pub : public key(공개키), 원격 저장소가 있는 서버에 저장
- <a href = "https://github.com/RyuKyeongWoo.keys">나의 공개키 확인</a>
</pre>
## 원격 저장소의 원리(Github)
<pre>
1. 원격 저장소를 등록하면 <b>remote(origin)의 정보가 있는 config 파일</b>이 .git폴더에 만들어진다.
2. git push -u origin master 명령어를 통해서 <b>refs/remotes/origin/master</b>가 생성된다.

git은 <b>refs/heads/master(로컬 저장소의 master baranch)</b>와 
<b>refs/remotes/origin/master(원격 저장소의 master baranch)</b>가 가리키는 커밋을 비교하여 차이를 안다.
</pre>
## pull vs fetch의 원리(Github)
원격 저장소(의 URL)을 등록하고 저장소 기록을 주고받습니다.
```
최신 commit 내용을 가져오고 병합(merge)한다.
$ git pull 
```
```
최신 commit 내용을 가져오고 병합(merge)하지 않는다.
$ git fetch

merge를 통해서 동일하게 만들 수 있다.
$ git merge [원격 저장소]/[원격 저장소 브랜치]
$ git merge origin/master
```
<pre>
git pull 
- 로컬 저장소의 master branch를 원격 저장소(origin)의 master branch와 같은 최신 commit을 가리키게 한다.

git fetch
- 원격 저장소(origin)에서 최신 commit 내용을 가져오기만 한다.
- 로컬 저장소의 master branch는 최신 commit을 가리키지 않는다.
- 장점 : origin과 local의 master 사이에 차이점을 비교할 수 있다.

  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/fetch.PNG"/>
</pre>
