---
title:  "[Git] Github commit message 수정"
excerpt: "깃허브 커밋 메세지를 수정해보자."

tags:
  - [Git]

toc: true
toc_sticky: true
toc_label: "[Git] commit message 수정"
 
date: 2022-05-13
last_modified_at: 2022-05-13
---

## Git 커밋 메세지 수정하기.

아래 내용은 이미 커밋을 push 하여 remote 에 올린 상황일때 해결 방법이다.
커밋메세지 수정을 추천하지 않지만, 그래도 해야 될 때를 위해 작성한다.

- 참고사이트
  - [블로그](https://velog.io/@mayinjanuary/git-%EC%BB%A4%EB%B0%8B-%EB%A9%94%EC%84%B8%EC%A7%80-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0-changing-commit-message){:target="_blank"}

  - [블로그](https://holika.tistory.com/entry/Git-%EC%82%BD%EC%A7%88%EA%B8%B0%EB%A1%9D-Git-push-%EC%9D%B4%ED%9B%84%EC%97%90-%EC%BB%A4%EB%B0%8B-%EB%A9%94%EC%8B%9C%EC%A7%80%EB%A5%BC-%EC%88%98%EC%A0%95%ED%95%98%EA%B3%A0-%EC%8B%B6%EC%9D%84-%EB%95%8C){:target="_blank"}



#### `1. 커맨드 창에서 rebase 를 사용해 수정모드로 진입한다.`

![GIT](/assets/image/git/Git_Commit_Message_01.PNG)

```console
git rebase HEAD~1 -i
```

위 내용은 `바로직전` 의 commit 을 수정 할 때의 예시이며,
만약 더 이전의 내용 수정을 원한다면 `HEAD~2, HEAD~3` 와 같이 
HEAD~{몇번째 전의 commit인지} 입력하면 된다.

<br>

#### `2. pick 을 reword 로 수정한다.`

![GIT](/assets/image/git/Git_Commit_Message_02.PNG)

![GIT](/assets/image/git/Git_Commit_Message_03.PNG)

![GIT](/assets/image/git/Git_Commit_Message_04.PNG)

reword 는 내용을 그대로 둔 뒤 `커밋메세지` 만을 수정하는 명령어 이다.
따라서 편집모드에서 `pick` 을 `reword` 로 수정 한 뒤, `esc` 를 눌러 편집을 종료하고 `:wq` 를 입력 후 엔터키를 눌러 수정 한 내용을 저장 및 vi를 종료 한다.

위 내용까지 진행하게 되면 아래와 같이 `vi` 를 이용한 에디터가 켜지고, commit message 를 수정 할 수 있게 된다.

![GIT](/assets/image/git/Git_Commit_Message_05.PNG)

<br>

#### `3. commit message 수정`

2번까지 완료 후 화면에서 커밋 메세지를 수정 변경 한 다음 `esc` 를 눌러 편집을 종료하고 `:wq` 를 입력 후 엔터키를 눌러 변경한 commit message 를 저장 및 vi를 종료 한다.

![GIT](/assets/image/git/Git_Commit_Message_06.PNG)

<br>

#### `4. 메세지 덮어씌우기`

![GIT](/assets/image/git/Git_Commit_Message_07.PNG)

![GIT](/assets/image/git/Git_Commit_Message_08.PNG)

```console
git push --force
```

git bash 에서 위 명령어를 입력하면 내가 원하던 커밋 메세지가 변경 된 걸 확인 가능하다.

<br>

#### `5. force pushing 의 위험성`

github 공식 문서에 따르면, `force pushing` 을 최대한 사용하지 않아야 한다고 한다. 이유는 `push 된 커밋의 로그를 가지고 있던 다른 팀원들이 로그를 수동으로 수정해줘야 하기 때문` 이다.

<br>

> `We strongly discourage force pushing, since this changes the history of your repository. If you force push, people who have already cloned your repository will have to manually fix their local history. For more information, see "Recovering from upstream rebase" in the Git manual.`

