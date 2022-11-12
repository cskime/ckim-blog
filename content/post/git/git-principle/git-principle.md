---
title: "The Principle of How Git Works?"
date: 2022-11-12T11:36:52+09:00
draft: false
summary: "How git knows file changes? How git create commits?"
tags: [git]
---

# git add의 원리

✅ `'git add'는 object 파일을 만들고 'Index' 파일에 object id를 기록하는 작업이다.`

Git이 수정된 파일을 추적하도록 만들기 위해 `git add` 명령어를 사용한다.

## Object(Blob)

`git add`를 실행하면 `.git/objects` 디렉터리에 object(**blob**)를 만든다.

- 파일 내용을 binary로 변환 후 SHA-1 hash 생성
- Hash 값의 앞 두자리는 디렉터리 이름, 나머지 자리는 파일 이름으로 사용

{{< figure src="/images/git/git-principle-1.png" width="60%" >}}
    
## Index file

`git add`를 실행하면 `Index` 파일에 해당 파일의 object id와 파일 이름을 기록한다. Object 이름은 hash이므로, 같은 내용의 파일은 같은 이름의 object를 공유한다.

{{< figure src="/images/git/git-principle-2.png" >}}

# git commit의 원리

✅ `'git commit'은 commit을 만드는 시점의 파일 상태 및 기타 정보를 object로 만드는 작업이다.`

`git commit` 명령어를 사용하면 현재 파일 상태에 대한 버전(commit)을 생성한다.

`git commit`을 실행하면
    
1. 현재 `Index` 파일의 snapshot을 object(**tree**)로 생성
    - `git add`로 추가한 파일만 commit할 수 있는 이유 : `git add`를 해야 snapshot에 포함된다.
    - Commit을 하면, `Index` 파일과 commit tree에 기록된 내용이 동일하다.
2. Snapshot(tree object)과 commit 정보들을 사용해서 object(**commit**) 생성
    1. 이름, email 등 author 정보
    2. Commit message
    3. 하위 commit id(**parent**)
        - `git log`가 commit tree를 그리는 방법 : parent id를 찾아간다.
        - Initial commit은 parent object id가 없음

{{< figure src="/images/git/git-principle-4.png" >}}

# git status의 원리

✅ `'git status'는 'Index' 파일과 commit tree object의 내용을 비교해서 파일 상태를 알 수 있다.`

## 파일 상태

Git은 파일을 세 가지 상태로 인식

1. **Untracked** : Index에 기록되지 않은 파일
2. **To be committed(Tracked)** : Index에 기록되었지만 최신 commit의 tree에 없는 파일
3. **Committed** : Index에 기록되어 최신 commit의 tree에도 포함되어 있는 파일

현재 파일 상태, `Index`에 기록된 상태, commit에 기록된 상태를 비교해서 파일의 수정 상태를 판단한다.

1. 수정된 파일의 내용을 SHA-1 hash(object id)로 변환한다.
2. index에 일치하는 object id가 있는지 검사한다.
    - 없으면 untracked
    - 있으면 to be committed
3. 최신 commit의 tree에 일치하는 object id가 있는지 검사한다.
    - 없으면 to be committed
    - 있으면 committed → `git status`를 실행하면 commit할 파일이 없다는 메시지 출력

## 작업 영역

Git은 파일의 상태에 따라 **개념적으로** 세 가지 영역으로 파일을 분류한다.

1. **Working Directory** : 수정된 뒤 Index 파일에 추가되지 않은 파일 영역
    - or **Working tree**, **Working copy**
2. **Index** : Index 파일에 추가된(Commit에 포함될 수 있는) 파일 영역
    - or **Staging area**, **Cache**
3. **Repository** : Commit의 tree object(snapshot)에 포함된 파일 영역
    - or **History**, **Tree**

{{< figure src="/images/git/git-principle-5.png" width="80%" >}}

# Branch의 원리

✅ `Branch는 최신 commit object를 참조하는 'refs/heads/{branch}' 파일로 관리하고, 'HEAD'가 branch 파일을 참조하여 현재 선택된 branch를 알려준다.`

## Branch file

Git은 `refs/heads/{branch_name}` 파일을 branch로 인식한다.

- `refs/heads/{branch_name}`은 branch의 최신 commit id이 쓰여진 text 파일
- Create/Delete branch manually
    - Create : `refs/heads` 디렉터리에 commit id를 내용으로 하는 파일 생성
    - Delete : `refs/heads` 디렉터리에 있는 파일을 삭제

## HEAD file

Git은 `HEAD` 파일로부터 현재 선택된 branch 확인

- `HEAD` 파일은 `refs/heads/{branch_name}`이 쓰여진 text 파일
- Checkout branch : `HEAD` 파일 내용을 특정 branch 경로로 변경

`git log` 명령어가 commit tree를 찾아내는 방법

1. `HEAD` 파일에서 현재 선택된 branch 확인
2. 해당 branch의 최신 commit 확인
3. 해당 commit부터 parent를 찾아나감
4. Parent가 없는 commit이 나오면 종료

{{< figure src="/images/git/git-principle-6.png" width="80%" >}}

# git checkout의 원리

✅ `'git checkout'은 'HEAD' 파일 내용을 변경하는 작업이다.`

- Branch를 변경하거나, 특정 commit으로 파일 상태를 변경하는 것
    - Branch 또는 commit 변경 → 최신 commit 변경 → Tree object(snapshot) 변경
    - **즉, checkout은 `Index` 파일 내용을 특정 commit의 snapshot으로 교체하는 작업이다.**
- `HEAD` 파일을 수정하면 checkout 할 수 있다.
    - Checkout branch : `refs/heads/{branch_name}` 에서 branch name 변경
    - Checkout commit : 내용을 commit id로 변경
        - **‘Detached HEAD’** 상태라고 부른다.
        - 원래 branch를 통해서 commit에 접근해야 하는데, commit에 직접 접근했다는 뜻

# git reset의 원리

✅ `'git reset'은 branch 파일이 참조하는 commit id를 변경하는 작업이다.`

## Reset

현재 branch의 HEAD를 특정 commit으로 이동하고, 이후 commit을 제거한다.
- 실제로 object를 제거하지는 않고 `Index` 또는 commit tree에서만 제거된다.
- 남아있는 object들은 `logs/refs/heads/{branch}` 파일에 기록된다.
    {{< figure src="/images/git/git-principle-7.png" >}}
    - 특정 branch에서 일어나는 모든 활동을 기록하는 파일
    - 기록된 id를 사용해서 특정 시점으로 되돌아갈 수 있다.
    - Git은 object를 함부로 삭제하지 않고, 꼭 필요할 때만 garbage collector가 동작한다.
- Reset 하기 직전 commit은 `ORIG_HEAD` 파일에 기록된다.
    - `git reset --hard ORIG_HEAD` : Reset 바로 이전으로 되돌린다.

## Reset options

{{< figure src="/images/git/git-principle-8.png" width="80%" >}}
    
1. Soft : Commit tree의 내용만 되돌린다. → Repository만 reset
2. Mixed : Commit tree 및 index 내용만 되돌린다. → Index까지 reset
3. Hard : 모든 변경 사항을 되돌린다.

# Merge와 Conflict의 원리

✅ `Git은 3-way merge 방식으로 병합하고, 자동 merge할 수 없는 곳에 conflict를 발생시킨다.`

## 병합(Merge)

특정 base commit에서 시작한 두 branch가 서로 다른 수정사항을 가질 때, 두 branch의 변경사항을 하나로 합치는 것

- Git은 3-way merge 방식으로 자동 병합한다.
    {{< figure src="/images/git/git-principle-9.png" width="80%" >}}
- 세 가지 버전이 모두 필요하므로, git은 병합할 때 object 파일을 세 개 만든다.
    - 두 branch A, B에서 각각 수정된 object
    - 기준이 되는 base commit에 포함된 object
    - MERGE_HEAD : Conflict를 해결하고 최종 생성될 commit id를 참조하는 파일

## 충돌(Conflict)

서로 다른 branch가 같은 파일의 같은 부분을 다르게 수정하여 git이 자동으로 병합할 수 없을 때, 이것을 알리는 것.

Git은 둘 중 어떤 수정사항을 적용해야 하는지 알지 못함. 직접 수정하라고 알려주는 것

# Reference

- [생활코딩 Git의 원리](https://opentutorials.org/course/2708/15212)