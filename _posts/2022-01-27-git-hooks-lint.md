---
date: 2021-01-27
title: "Git hooks 및 린트를 통한 코드스타일 통일"
categories: git,lint
---

## 1. 개요
* 팀 단위의 프로젝트를 진행할 경우 각자의 코딩 스타일이 다르다. 
* 코딩 스타일을 통일해 놓지 않으면 프로젝트가 진행될 수록 결국 표준화된 규격을 맞추느라 수고가 들게 되고 결국 기술 부채로 남게 된다.
* 이를 방지하기 위해 로컬에서 commit을 할 때 린트를 적용하여 특정 코딩 스타일에 맞지 않으면 자동으로 커밋이 되지 않도록 하는 방법에 대해 알아본다.

## 2.1 코딩 스타일 적용
* 먼저 적용할 코딩 스타일을 정해보자
* Java에서 가장 많이 사용하는 코딩 스타일은 checkstyle이다. [checkstyle 홈페이지](<https://checkstyle.sourceforge.io/>)
* 위의 주소에 들어가면 버전 별 checkstyle을 확인해 볼 수 있다.
* 9.0 버전을 선택하여 적용한다.
* [](<>) 에서 다운로드 후 압축을 풀고 resources 폴더에 가면 아래의 2개의 checkstyle을 확인할 수 있으며 이 중 google_checks.xml을 사용하기로 하자.
  * sun_checks.xml
  * google_checks.xml
* google_checks의 경우 indentation이 2로 설정되어 있어 코드 보기가 비좁아서 아래 처럼 바꾸도록 한다.
    ```  
    <module name="Indentation">
            <property name="basicOffset" value="4"/>
            <property name="braceAdjustment" value="0"/>
            <property name="caseIndent" value="4"/>
            <property name="throwsIndent" value="8"/>
            <property name="lineWrappingIndentation" value="8"/>
            <property name="arrayInitIndent" value="4"/>
    </module>
    ```
* 수정한 xml파일을 현재 작업할 rootDir에 config/checkstyle 디렉토리안에 위치 시켜 놓는다. 
  * 디렉토리를 config/checkstyle을 활용해야 Gradle을 통한 checkstyle을 확인할 수 있다.


## 2.2. Gradle 수정
* 2.1에서 코딩 스타일을 적용할 checkstyle을 저장하였다.
* Gradle 명령어를 통해 위에서 저장한 checkstyle을 적용해보자 
* 먼저 `apply plugin: `checkstyle` 입력한다.
* 아래와 같이 checkstyle 설정 정보를 입력해 준다.
  * configFile 통해 checkstyle.xml 적용
  * 버전명은 꼭 9.0 (다운받았던 버전)으로 맞춰줘야 에러 없이 동작한다.
```gradle
    checkstyle {
        toolVersion = '9.0'
    }

    task checkstyle(type: Checkstyle) {
        configFile = file("${rootDir}/config/checkstyle/checkstyle.xml")
        source 'src/main/java/'
        include '**/*.java'
        showViolations true
        classpath = files()
        ignoreFailures = false
        maxWarnings = 0
    }
```

* 이제 Gradle 빌드를 하게 되면 자동으로 checkstyle을 확인하게 되며 스타일이 맞지 않은 경우 경고를 내보내게 된다.


## 2.3. pre-commit 적용
* 로컬에서 커밋을 할 때 자동으로 해당 gradle checkstyle task를 수행하게 되면 커밋 전 코딩 스타일의 검사를 실시하게 된다.
* .git 디렉토리 안에는 hooks이라는 폴더가 있는데 이 안에는 git 명령어를 수행하기 전에 자동으로 사전에 호출될 수 있게 끔 하는 파일들이 sample들로 있다.
* 우리는 이 중에서 commit하기 전에 checkstyle을 적용해야 함으로 pre-commit에 미리 사전에 작업할 내용을 채워넣으면 된다.
* [lint 적용](<https://programmersought.com/article/57431602993/>) 을 참조하여 등록한 모듈에 대해 checkstyle task를 수행하도록 작성하자.
  
```bash
#!/bin/bash

red='\033[0;31;1m'
green='\033[0;32;1m'
noColor='\033[0m'

# for each module
projectDir=$(git rev-parse --show-toplevel)
lintProjects="trade-app trade-chat trade-commerce trade-common trade-user"

for project in ${lintProjects[@]}
do
    # Checkstyle
    printf "-----------------Starting run java checkstyle for ${project}-----------------"
    checkstylePath="${project}/build/reports/checkstyle/checkstyle.html"

    ./gradlew ${project}:checkstyle >/dev/null
    checkstyleStatus=$?
    if [ $checkstyleStatus -ne 0 ]
    then
        printf "${red}Failed, ${project} project has checkstyle issues!${noColor}"
        open ${projectDir}/${checkstylePath}
        exit $checkstyleStatus
    fi
    printf "${green}Succeeded, no android checkstyle issues found for ${project}${noColor}\n\n"
done
``` 

* pre-commit 에 위의 내용을 채워 넣은 후 chmod +x pre-commit 을 통해 실행 가능한 파일을 만들어 놓도록 한다.
* git config core.hooksPath .githooks 명령어를 통해 .githooks에 해당 pre-commit을 넣어놓고 사용하는 git에 자동으로 현재 hooks들이 위치하고 있는 곳을 설정해두자
  * 참고로 git version이 2.9 이상인 경우에만 동작하니 유의하자

## 2.4. 전체 결과
위에서 부터 하나씩 등록하면 현재 프로젝트 폴더 구조는 아래와 같다.

<p align="center">
  <img src="/assets/images/2022-01-27-git-hooks-lint-01.png" height ="30%" width="60%" alt="git hooks"/>
</p>
<p align="center">
    프로젝트 구조
</p>

전체 gradle 의 정보는 아래와 같다. 모듈로 구성을 했기 때문에 checkstyle plugin이 subtask안에 있다.
```gradle
buildscript {
    ext {
        springBootVersion = '2.6.2'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}"
        classpath "io.spring.gradle:dependency-management-plugin:1.0.11.RELEASE"
    }
}

allprojects {
    group 'team.comito'
    version '1.0.0'
}

subprojects {
    apply plugin: 'java'
    apply plugin: 'idea'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'
    apply plugin: 'checkstyle'

    sourceCompatibility = '11'
    targetCompatibility = '11'
    compileJava.options.encoding = 'UTF-8'

    repositories {
        mavenCentral()
    }

    dependencies {
        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
        annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"
        testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'
        testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'
    }

    test {
        useJUnitPlatform()
    }

    checkstyle {
        toolVersion = '9.0'
    }

    task checkstyle(type: Checkstyle) {
        configFile = file("${rootDir}/config/checkstyle/checkstyle.xml")
        source 'src/main/java/'
        include '**/*.java'
        showViolations true
        classpath = files()
        ignoreFailures = false
        maxWarnings = 0
    }
}
```

## 2.5 결과
* 제대로 동작하는 지 확인해 보자
* 코드의 간격을 아래와 같이 엉망으로 만들자
<p align="center">
  <img src="/assets/images/2022-01-27-git-hooks-lint-02.png" height ="100%" width="100%" alt="test code"/>
</p>
<p align="center">
    [테스트 코드]
</p>

* 그리고 난 후 intelliJ에서 commit을 실행하자. 그림과 같이 git hooks을 적용여부의 체크박스 화면이 나오는것을 확인할 수 있다.
<p align="center">
  <img src="/assets/images/2022-01-27-git-hooks-lint-04.png" height ="100%" width="100%" alt="test code"/>
</p>
<p align="center">
   [Run git hooks]
</p>

* 커밋을 누르면 제대로 커밋되지 못하고 실패가 된 후 html 페이지를 띄워주게 되는데 html 내역을 보면 indentation이 제대로 되지 않았다는 것을 확인할 수 있다.
<p align="center">
  <img src="/assets/images/2022-01-27-git-hooks-lint-03.png" height ="100%" width="100%" alt="pre commit html"/>
</p>
<p align="center">
  [출력 결과]
</p>

* 제대로 간격을 맞춰준 후 다시 커밋을 하면 정상 동작한다.



***** 
## References
* [린트 적용 참고](<https://velog.io/@9geunu/GitHooks-CheckStyle-AndroidLint>)
* [그래들 문서](<https://docs.gradle.org/current/userguide/checkstyle_plugin.html>)