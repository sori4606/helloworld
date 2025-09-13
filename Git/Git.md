# Git

##목차

- [Git vs GitHub](./Git.md#git-vs-github)
- [Git에서 파일들의 상태](./Git.md#git에서-파일들의-상태)
- [Git의 다양한 명령어들](./Git.md#git의-다양한-명령어들)

### Git VS Github

git : 코드 변경사항을 추적하고 여러 개발자가 협업할 수 있게 해주는 소프트웨어 <br />
(소프트웨어 ? 컴퓨터나 전자 장치에서 특정 작업을 수행하도록 설계된 프로그램, 데이터, 명령어의 집합)

github : Git 저장소를 온라인에 저장하고 관리할 수 있는 플랫폼 <br />
(플랫폼 ? 다른 소프트웨어나 프로세스가 실행될 수 있는 기본 환경이나 시스템)

**_소프트웨어, 플랫폼 파헤치기_** <br />
소프트웨어는 특정 기능을 수행하는 프로그램.
플랫폼은 다른 소프트웨어가 작동할 수 있는 기반 환경을 제공. 예를들면,

1. 스마트폰 OS (안드로이드/iOS)

   안드로이드, iOS는 플랫폼 <br />
   이 기본 시스템 위에서 카카오톡, 유튜브 등의 앱들이 실행됨.

2. 쇼핑몰 <br />
   쇼핑몰 건물은 플랫폼. <br />
   다양한 상점들이 이 기본 구조 안에서 영업을 함. <br />
   쇼핑몰은 전기, 수도, 보안 등의 기본 서비스를 제공하고, 상점들은 그 환경 안에서 자신들의 사업을 운영

### Git에서 파일들의 상태

1. Untracked 상태
   : 파일을 새로 생성하고, 한번도 git add 해주지 않음
2. Tracked 상태
   : 여기 안에서 Staged, Unmodified, Modified상태로 나뉘는데, <br />

   1. Staged 상태 : 스테이징된 상태 (git add 해준 상태) <br />

   2. Unmodified 상태 : 현재 파일의 내용이 최신 커밋과 비교했을 때 전혀 바뀐게 없는 상태 => 커밋을 하고 난 직후에는 모든 파일들이 이 상태가 된다. <br />

   3. Modified 상태 : 최신 커밋과 비교했을 때 바뀐 내용이 있는 상태

### Git의 다양한 명령어들

1.  git init
    : 현재 디렉토리를 Git이 관리하는 디렉토리로(working directory) 설정하고, 그 안에 레포지토리(.git 디렉토리) 생성

2.  git reset
    : git add를 취소한다. (staging area에 올렸던 파일 다시 내리기) 3개의 옵션이 있다.

    1. --soft : 커밋만 되돌리고 변경 내용은 스테이징 영역에 유지 (git add한 상태로 되돌리기) (HEAD가 특정 커밋을 가리키도록 이동시킴)
    2. --mixed(기본값) : 커밋과 스테이징을 되돌리고 변경 내용은 작업 디렉토리에 유지 (git add 하기 전 상태로 되돌리기) (staging area도 특정 커밋처럼 리셋)
    3. --hard : 커밋, 스테이징, 작업 디렉토리의 변경 내용까지 모두 되돌림 (working directory도 특정 커밋처럼 리셋)

    +) 여러 커밋을 하나의 커밋으로 만들고 싶으면 ? -> mixed 또는 soft 사용
    -> HEAD는 이전 커밋을 가르키지만, working directory는 그대로!
    -> git reset --soft 하고, git add, commit 해주면 된다.
    (A--B--C 이렇게 있으면, soft 옵션으로 A로 되돌아가서 add, commit 진행)

3.  git revert
    : 지정한 커밋의 변경사항을 취소하는 새로운 커밋을 생성 <br />
    사용시점 : 협업 환경에서 이미 원격 저장소에 푸시된 커밋을 되돌릴 때. 여러개 revert도 가능. <br />
    ex) git revert facd..eea5 (만약, git revert A..C면, A는 포함되지 않고, B,C만 revert 커밋이 생성된다.)

    ### git reset VS git revert

    1. 이력 관리 : reset은 이력을 삭제하고, revert는 이력을 추가함
    2. 안전성 : revert가 더 안전하고, 협업에 적합
    3. 용도 : reset은 주로 로컬 작업에, revert는 공유 작업에 사용 (만약, 이미 레포에 push 되어진 상태에서 reset을 하려면, force push를 통해 푸시 해줘야한다. 이전 상태로 되돌리고 push를 하면 뭐 뒤쳐져있다 .. pull이 필요하다 .. 이런 오류 발생함)

4.  git commit --amend
    : 가장 최신 커밋 수정하기

    1.  메시지만 바꾸고 싶을 때

        ```bash
        git commit --amend -m "새로운 메시지"
        ```

    2.  파일을 추가하고 싶을 때 (index.js 파일을 추가안해준 상황)

        ```bash
        git add index.js
        git commit --amend
        ```

        -> 이러면 index.js 파일이 기존의 마지막 커밋에 포함됨.

⚠️ 주의사항 : 로컬에서 혼자 작업 중일 때는 아무 문제 없지만, 이미 push된 커밋을 amend 하면 force push 해야 해서 협업중엔 조심해야함. (=> 커밋 해시가 바뀜)

커밋해시 ? = 커밋의 모든 정보 (커밋 메시지, 커밋 시간, 작성자 등)를 기반으로 만들어져서 하나의 커밋 해시를 만들어냄. 마치 "DNA"같은 거임

원래는 abc123이라는 커밋이여서 다른 개발자들이 abc123이라는 커밋을 가지고 있는데, 내가 force push 해버리면 def456으로 바뀌고 push를 하면 다른 사람 히스토리랑 충돌난다!

5.  git clone 할 때 하위폴더로 다시 생성을 하기 때문에, git clone remote 주소 .을 하면(.을 붙히면) 현재 폴더에 clone이 된다

6.  커밋에 태그도 달 수 있다. (version1, version2 ...)
    : git tag [태그 이름] [커밋 아이디] <br />
    ex) git tag Version1 84ab

7.  git push -u origin master (-u는 --set-upstream 옵션의 약자)
    : 이 옵션을 주면 로컬 레포에 있는 master 브랜치가 origin에 있는 master 브랜치를 tracking(추적)하는 걸로 설정됨 -> 그래서 git push, git pull만 해줘도 동작이 됨

8.  git fetch : 리모트 레포지토리에 있는 브랜치의 내용을 일단 가져와서 살펴본 후에 머지하고 싶을 때 사용 (-> 리모트 레포지토리에 있는 내용을 일단 로컬로 가져올 때. git pull과는 다름. git pull은 가져오고 자동으로 병합)

9.  어떤 커밋으로 작성됐는지를 볼 때 ? -> git blame

10. 커밋을 누가 한건가 ? -> git show

11. git reflog
    : 헤드가 이때까지 가리켜왔던 커밋들을 기록한 정보 (git reset을 한다고 이후에 커밋들이 사라지는게 아님. 근데, 최신 커밋으로 되돌아가고 싶은데 커밋 id를 모를 때 git reflog로 알 수 있다)

12. 새로운 커밋을 만들고 싶지 않을 때 ? -> git rebase [브랜치 이름]
    : 커밋 히스토리가 더 깔끔하다는 장점이 있다. (git rebase --continue로 진행)

    🔍설명 : A, B 브랜치가 있는 상태에서 지금 HEAD가 A 브랜치를 가리킬 때, git rebase B를 실행하면 A, B 브랜치가 분기하는 시작점이 된 공통 커밋 이후로부터 존재하는 A 브랜치 상의 커밋들이 그대로 B 브랜치의 최신 커밋 이후로 이어붙여짐(git merge와 같은 효과를 가지지만 커밋 히스토리가 한 줄로 깔끔하게 된다는 차이점이 있음)

    `merge`

    ```bash
       C---D  (A)
       /    \
    A---B---E---F---G  (merge 커밋 G 생성)
    ```

    `rebase`

    ```bash
    A---B---E---F---C'---D'  (한 줄로 깔끔!)
    ```

13. 작업내용을 임시로 저장하고, 적용하고, 지우고 싶을 때 ? -> git stash, git stash apply, git stash drop, git stash pop
    : 작업 내용을 나중에 또 쓸 필요가 있다면 git stash apply를, 나중에 또 쓸 필요가 없다면 git stash pop을 쓰면 됨 (git stash apply [커밋 아이디] 처럼 특정 작업 내용을 가져오는 것도 가능. 다른것들도(drop, pop) 마찬가지)

14. 특정 커밋 하나 또는 여러개를 현재 브랜치에 가져오고 싶다 ? -> git cherry-pick (꼬였을 때 꽤 자주 쓰였다)

🔧 기본 문법

```bash
git cherry-pick <커밋 해시>
```

여러 개 가져올 땐 :

```bash
git cherry-pick <해시1> <해시2> ...
```

혹은 범위 :

```bash
git cherry-pick A^..C // A는 포함된다. reset이랑 다름
```

상황

```mathematica
main:     A -- B -- C -- D -- E -- F
feature:                 \
                          G -- H -- I

```

G, H, I는 feature 브랜치에서 작업한 커밋
근데, G ~ I 사이 어딘가에서 무슨 일이 꼬임
그래서 난 G는 괜찮고, H는 뭔가 문제고, I만 가져오고 싶음

🛠️ 해결 절차

1. 임시 브랜치 생성

   ```bash
   git checkout -b temp-fix
   ```

2. 정상적인 커밋만 cherry-pick (커밋 해시는 `git log`)

   ```bash
   git cherry-pick <G의 해시> <I의 해시>
   ```

3. 코드 확인 후 병합

   ```bash
   git checkout main
   git merge temp-fix
   ```

4. 정리

   ```bash
   git branch -d temp-fix
   ```

그 외)

1. git config user.name 'kkk' : 현재 사용자의 아이디를 'kkk'로 설정 (커밋할 때 필요한 정보)

2. git config user.email 'teacher@aaa.com' : 현재 사용자의 이메일 주소를 'teacher@aaa.com'로 설정 (커밋할 때 필요한 정보)

3. git status : Git이 현재 인식하고 있는 프로젝트 관련 내용들 출력 (문제 상황이 발생했을 때 현재 상태를 파악하기 위해 활용하면 좋음)

4. git commit -m "커밋 메시지" : 현재 staging area에 있는 것들 커밋으로 남기기

5. git config alias.[별명] [커맨드] : 길이가 긴 커맨드에 별명을 붙여서 이후로는 별명으로도 해당 커맨드를 실행할 수 있게 설정

   ex) git config --global alias.co checkout 을 하면, git checkout [브랜치 이름] 을 git co [브랜치 이름] 이렇게 바꿀 수 있음.

6. git diff [커밋 A의 해시] [커밋 B의 해시] : 두 커밋 간의 차이 비교

7. git log --oneline : 커밋 해시값 한줄로 보기

8. git branch [새 브랜치 이름] : 새로운 브랜치 생성

   1. git branch -b [새 브랜치 이름] : 새 브랜치 생성하고 바로 그 브랜치로 이동

   2. git branch -d [기존 브랜치 이름] : 해당 브랜치 삭제

   3. git checkout [기존 브랜치 이름] : 그 브랜치로 이동

9. git merge [기존 브랜치 이름] : 현재 브랜치에 다른 브랜치를 머지

   1. git merge --abort : 머지를 하다가 conflict가 발생했을 때, 일단 머지 작업을 취소하고 싶을때 이전 상태로 돌아가기

10. 머지할 때 옵션 3가지

    1. Accpet current - 현재 브랜치의 내용을 선택하는 것

    2. Accept Incoming - 가져오는 브랜치의 내용을 선택하는 것

    3. Accept both - 두 가지 모두를 선택하는 것
