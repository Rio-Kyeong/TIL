# Git-Flow
<pre>
Git-Flow는 대표적인 브랜칭(branching) 전략 중 하나로 5가지 종류의 브랜치가 존재한다.
항상 유지되는 메인 브랜치들(master, develop)과 일정 기간 동안만 유지되는 보조 브랜치들(feature, release, hotfix)이 있다.

 - master : 제품으로 출시될 수 있는 브랜치 (사용자에게 최종적으로 노출되는 버전을 가짐)
 - develop : 실질적 개발(구현)하는 브랜치 (마스터로부터 파생된 브랜치)
 - feature : 각각의 기능을 개발하는 브랜치 (develop branch 에서 파생하여 작업)
 - release : 사용자들에게 배포할 버전을 준비하는 목적으로 만들어진 브랜치
 - hotfixes : 출시 버전에서 긴급하게 발생한 버그를 수정하는 브랜치
</pre>
## Git-Flow 전략
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/gitFlow.PNG">

 - 최초 master 브랜치에서 develop 브랜치가 파생된다.
 - develop 브랜치에서는 상시로 버그를 수정한 커밋들이 추가된다.
 - 새로운 기능 추가 작업이 있는 경우 develop 브랜치에서 feature 브랜치를 생성한다.
 - feature 브랜치는 develop 브랜치의 내용을 항상 최신화하여 가지고 있어야 한다.
 - 기능 추가 작업이 완료되었다면 feature 브랜치는 develop 브랜치로 merge 된다.
 - develop에 이번 버전에 포함되는 모든 기능이 merge 되었다면 QA를 하기 위해 develop 브랜치에서부터 release 브랜치를 생성한다.
 - QA를 진행하면서 발생한 버그들과 업데이트는 release 브랜치에 수정되며, develop 브랜치에 바로바로 merge를 통해 동기화를 한다.
 - QA를 통과하였다면 release 브랜치를 master와 develop 브랜치로 merge 한다.
 - 마지막으로 출시된 master 브랜치에서 버전태그를 추가한다.
</pre>

## Git Repository 구성하기
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/Git/img/RepositoryConfiguration.png">

 - Upstream Repository : 개발자들이 공유하는 저장소(최신 소스코드가 저장되어 있는 원격 저장소)
 - Origin Repository : Upstream Repository를 Fork한 원격 개인 저장소
 - Local Repository : 내 컴퓨터에 저장되어 있는 개인 로컬 저장소
</pre>  



