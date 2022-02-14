---
date: 2021-01-17
title: "gitignore 추가하기"
categories: git
---

## 1. gitignore 추가
프로젝트를 만든 후 commit을 하려고 하면 빌드 관련 파일 및 여러 ide 파일 그리고 .class파일들이 같이 git의 stage에 add 되어 수정된 
소스코드와 같이 commit되는 것을 알 수 있다.   
이를 방지 하기 위해 관련 없는 파일들은 .gitignore 을 생성한 후 추가해 주면 된다.

```
### Java template
*.class
 
# Package Files #
*.jar
*.war
*.ear
 
### macOS template
*.DS_Store
.AppleDouble
.LSOverride
 
# IntelliJ project files
.idea
.idea/*.xml
*.iml
out
gen
build
rebel.xml
 
# Compliled files
/target/
**/target
 
/example/
 
# Gradle
.gradle
/build/
.gradletasknamecache
```

혹시 .gitignore에 위의 파일을 추가한 후에도 ignore가 되지 않는다면 `git rm -r -f --cached .` 명령어를 사용하여 git에 cached되어 있는 모든 파일들을 삭제해 주자
```
sclee@iseungcheol-ui-MacBookPro junggogurae-modules % git rm -r -f --cached .
rm 'build.gradle'
rm 'junggogurae-api/build.gradle'
rm 'junggogurae-api/src/main/java/me/sclee/junggogurae/api/MainApplication.java'
rm 'junggogurae-common/build.gradle'
rm 'junggogurae-front/build.gradle'
rm 'settings.gradle'
```
삭제 후 다시 .gitignore를 생성해 주면 정상적으로 동작한다.

***** 
## References
* [gitignore](<https://stackoverflow.com/questions/25436312/gitignore-not-working>)
