---
date: 2021-01-17
title: "Git remote"
categories: git
---

# 1. Git의 기본 명령어
## 1.1. Git remote
**git remote**는 원격 저장소를 관리할 수 있는 명령어이다. 예를 들어 사용자가 **git remote add origin [https://github.com/[사용자이름]/gitrepo** 를 입력할 경우 **origin**이라는 이름으로 원격 저장소를 등록한다. 그리고 **origin**을 사용하여 이제부터 원격 저장소를 접속할 수 있다.

## 1.2. 모든 Git remote 보기 
git remote -v를 이용하면 git 저장소의 url을 확인할 수 있다.
```
sclee@iseungcheol-ui-MacBookPro trade-platform % git remote -v
origin  git@github.com:f-lab-edu/trade-platform.git (fetch)
origin  git@github.com:f-lab-edu/trade-platform.git (push)
```

## 1.3. 모든 branch 확인하기
git branch -a를 입력하면 리모트와 로컬의 모든 브랜치 내역을 확인할 수 있다. 아래 브랜치 목록에서 확인 결과 현재 local branch는 main 브랜치가 한개 있으며,
remote server에는 origin이라는 이름으로 add-readme, main 브랜치가 있으며 현재 리모트에서는 현재상태는 origin/main의 브랜치 상태와 같다.
```
sclee@iseungcheol-ui-MacBookPro trade-platform % git branch -a
* main
  remotes/origin/HEAD -> origin/main
  remotes/origin/add-readme
  remotes/origin/main
```


1. 싱글톤
```java
public class Singleton{
    private static final Singleton INSTANCE = new Singleton();
    private Singleton(){}
    public static Singleton getSingleton(){
        return INSTANCE;
    }
}
```

***** 

## References
* (<https://stackoverflow.com/questions/23721115/singleton-pattern-using-enum-version>)