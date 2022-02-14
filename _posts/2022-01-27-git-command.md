---
date: 2021-01-27
title: "Git 명령어 정리"
categories: git
---


## 1.1. git reset: 커밋 취소하기
git log를 확인해 보자. 현재 HEAD는 `9610f80` 커밋을 가리키고 있다. 이때 현재 `9610f80`과 `5821a2b` 항목을 로컬에서 삭제하고 싶다.
이런 경우 `git reset` 명령어를 사용한다.

* git reset은 HEAD의 상태를 변경시키는 명령어.
* 보통 강제로 커밋 내역을 되돌릴 때 사용한다.

이를 위해 `git reset --hard a267cd9`로 HEAD가 `a267cd9`를 가리키도록 하면, 현재 HEAD 기준으로 2개의 커밋 내역을 취소할 수 있다.

```bash
sclee@iseungcheol-ui-MacBookPro trade-platform % git log --oneline
9610f80 (HEAD -> feature/apply-github-action-lint)  린트 테스트 커밋 3
5821a2b  린트 테스트 커밋
a267cd9 (origin/feature/apply-github-action-lint) Merge pull request #2 from f-lab-edu/add-readme
c18a11a Merge pull request #4 from f-lab-edu/feature/build-error-fix
68db4bd 빌드 및 IDE 관련 untracked 파일 삭제
2d60ce9 gitignore 추가
ef17031 빌드 완료를 위해 현재 사용하지 않는 모듈에 대해 bootJar=false 설정
3de4773 관계없는 문장 삭제
48aecd2 add the README.md
8d814f1 add README.md
63de127 project building
```
reset을 통해 2개의 commit을 되돌린 것을 확인할 수 있다.
```bash
sclee@iseungcheol-ui-MacBookPro trade-platform % git reset --hard a267cd9
HEAD의 현재 위치는 a267cd9입니다 Merge pull request #2 from f-lab-edu/add-readme
```
```bash
sclee@iseungcheol-ui-MacBookPro trade-platform % git log --oneline -r
a267cd9 (HEAD -> feature/apply-github-action-lint, origin/feature/apply-github-action-lint) Merge pull request #2 from f-lab-edu/add-readme
c18a11a Merge pull request #4 from f-lab-edu/feature/build-error-fix
68db4bd 빌드 및 IDE 관련 untracked 파일 삭제
2d60ce9 gitignore 추가
ef17031 빌드 완료를 위해 현재 사용하지 않는 모듈에 대해 bootJar=false 설정
3de4773 관계없는 문장 삭제
48aecd2 add the README.md
8d814f1 add README.md
63de127 project building
```

만약에 reset한 것을 다시 이전으로 되돌리고 싶다면 어떻게 할까?   
그런 경우 `git reflog` 명령어를 사용해 전체 커밋을 확인할 수 있다. 

```bash
sclee@iseungcheol-ui-MacBookPro trade-platform % git reflog
a267cd9 (HEAD -> feature/apply-github-action-lint, origin/feature/apply-github-action-lint) HEAD@{0}: reset: moving to a267cd9
9610f80 HEAD@{1}: commit: 린트 테스트 커밋 3
5821a2b HEAD@{2}: commit: 린트 테스트 커밋
a267cd9 (HEAD -> feature/apply-github-action-lint, origin/feature/apply-github-action-lint) HEAD@{3}: reset: moving to a267cd95448109139b51c5f9418771f81c063c2e
e8e206a HEAD@{4}: commit: 테스트 37
5785f0a HEAD@{5}: commit: 테스트 36
a614124 HEAD@{6}: commit: 테스트 35
20289d7 HEAD@{7}: commit: 테스트 34
1824473 HEAD@{8}: commit: 테스트 26
741be2b HEAD@{9}: commit: 테스트 25
94000d8 HEAD@{10}: commit: 테스트 24
e4fcace HEAD@{11}: commit: test 17
2b241a3 HEAD@{12}: commit: 테스트 17
```
다시 reset 명령어를 통해 복구를 확인할 수 있다.
```bash
sclee@iseungcheol-ui-MacBookPro trade-platform % git reset --hard 9610f80
HEAD의 현재 위치는 9610f80입니다  린트 테스트 커밋 3
```