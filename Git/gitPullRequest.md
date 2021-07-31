# Pull Request(PR)
<pre>
PR(Pull Request)는 <b>오픈소스 프로젝트에 참여할 때 가장 기본이 되는 동작</b>이다.
</pre>
## 1. Fork
 - 타겟 프로젝트의 저장소를 자신의 저장소로 Fork 한다.
<pre>
<b>Fork</b> : 다른 사람의 github repository를 복제하여 어떤 부분을 수정, 추가, 삭제를 용이하도록 해주는 복제기능

- fork 한 저장소는 원본(repository)와 연결되어있어 원본에 변화가 생기면 그대로 forked된 repository로 반영할 수 있다. 
  이 때 fetch나 rebase의 과정 필요하다.

- 그 후 original repository에 변경사항을 적용하고 싶으면 해당 저장소에 pull request를 해야한다.
  pull request 하기 전까지는 내 github에 있는 forked repository에만 change만 적용된다.
</pre>
## 2. clone, remote 설정
1. fork로 생성한 본인 계정의 저장소에서 Code 버튼을 누르고 표시되는 url을 복사한다(브라우저 url을 그냥 복사하면 안 된다)
2. 자신의 로컬 저장소에 clone 한다.

    ```
    $ git clone [fork한 저장소 url]
    ```
3. 클론된 로컬 저장소에 원격 저장소를 연결한다(기본 값으로 원격 저장소에는 origin 이라는 이름을 부여된다)

    ```
    # 원격 저장소 추가
    $ git remote add origin [원본 프로젝트 저장소 url]
    
    # 원격 저장소 설정 확인
    $ git remote -v
    ```
## 3. branch 생성
 - 자신의 로컬 저장소에서 코드 파일을 추가하는 작업은 branch를 만들어서 진행한다.

    ```
    # branch 생성 후 변경
    $ git checkout -b [branch name]
    
    # branch 확인
    $ git branch
    ```
## 4. 작업 후 add, commit, push
1. 코드 파일 수정, 추가, 삭제 작업을 진행한다.
2. 작업이 완료되면 add, commit, push를 통해서 자신의 원격 저장소(origin)에 수정사항을 반영한다.</br>
   (push 진행시에 branch 이름을 명시해주어야 한다)
    ```
    # 파일 이름 대신 마침표(.)를 기술하면 모든 변경 내용을 stage(커밋대기)한다.
      (CRLF 관련 경고 문구가 나타날 수 있는데 이는 무시해도 무관하다)
    $ git add [file name]

    $ git commit -m "message"

    # branch의 수정 내역을 origin 으로 푸시한다.
    $ git push origin [branch name]
    ```
## 5. Pull Request 생성
1. push 완료 후 원본 프로젝트 저장소에 들어오면 Compare & pull request 버튼이 활성화되어 있다.
2. 해당 버튼을 선택해서 메시지를 작성하고 PR을 생성한다.
## 6. code review, Merge Pull Request
 - PR을 받은 원본 저장소 관리자는 코드 변경내역을 확인하고 Merge 여부를 결정한다.
## 7. Merge 이후 동기화 및 branch 삭제
 1. 원본 저장소에 Merge가 완료되면 로컬 저장소의 코드와 원본 저장소의 코드를 동기화 한다.
 2. 작업하던 로컬의 branch를 삭제한다.
    
    ```
    # 코드 동기화
    $ git pull origin

    # 브랜치 삭제
    $ git branch -d [branch name]
    ```
- `나중에 추가 작업 시 동기화 후 3 ~ 7을 반복한다.`