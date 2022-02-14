---
date: 2021-01-17
title: "Gradle로 모듈러 프로젝트 구성하기"
categories: Gradle
---

# 1. Gradle에 대하여
이전까지 Maven을 많이 사용했지만, 근래들어 점점 Maven대신 Gradle을 사용하는 추세다 보니...Gradle로 멀티 모듈기반의 프로젝트 구성방법에 대해
알아보려고 합니다.

## 1.1. 루트 프로젝트
인텔리J에서 아래와 3개의 서브 모듈을 구성하자.
1. jungogurae-api (API 서버 관련 모듈)
2. jungogurae-front (화면 영역 담당 모듈)
3. jungogurae-common (공통으로 사용하는 모듈)


![모듈구성](/assets/images/2022-01-17-gradle-module-project-01.png){: width="60%" height="15%"} 

settings.gradle을 보면 아래와 같이 root 모듈과 4개의 서브 모듈을 include로 등록한 것을 확인할 수 있다. 즉 루트에서 4개의 모듈을 하위 모듈로 인식한다는 뜻이다.

```
rootProject.name = 'junggogurae-modules'
include 'junggogurae-api'
include 'junggogurae-front'
include 'junggogurae-common'
```

## 1.2. Root build.gradle 변경
build.gradle을 아래와 같이 하자. 현재 api 모듈에만 스프링 부트를 이용해 Main 함수를 작성하였고 나머지 모듈에는 아무 소스도 없는 상태이다.
따라서 메인 클래스가 없는 모듈은 `bootJar.enabled = false` 을 project('A') {B} 영역에 추가해 주어야 한다. 만약 그렇지 않은 경우 main class를
찾을 수 없어 빌드가 되지 않는다.

![모듈구성](/assets/images/2022-01-17-gradle-module-project-02.png){: width="60%" height="15%"} 

```
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

    sourceCompatibility = '11'
    targetCompatibility = '11'
    compileJava.options.encoding = 'UTF-8'

    repositories {
        mavenCentral()
    }

    // 하위 모듈에서 공통으로 사용하는 세팅 추가
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
}


project(':trade-common') {
    bootJar.enabled = false
    jar.enabled = true

    dependencies {
    }
}

project(':trade-chat') {
    bootJar.enabled = false
    jar.enabled = true

    dependencies {
        compileOnly project(":trade-common")
        implementation 'org.springframework.boot:spring-boot-starter-web'
    }
}

project(':trade-commerce') {
    bootJar.enabled = false
    jar.enabled = true

    dependencies {
        compileOnly project(":trade-common")
        implementation 'org.springframework.boot:spring-boot-starter-web'
    }
}

project(':trade-welcome') {
    dependencies {
        compileOnly project(":trade-common")
        implementation 'org.springframework.boot:spring-boot-starter-web'
    }
}
```
* buildscript는 소스코드를 빌드하고 실행하는데 필요한 글로벌 레벨의 dependency 와 repository를 설정.
   * `buildscript{}`는 build.gradle의 설정 자체를 위한 것이다.
   *  따라서, buildscript에서 dependency의 configuration이 굳이 compile일 필요가 없고 classpath인 점을 유의하자.
    
* 소스코드 컴파일과 빌드 작업 시작전에 실행되는 제일 먼저 실행되는 영역.
* subprojects는 setting.gradle에 include된 모듈을 관리할때 사용.
  * 자바 버전은 공통으로 11을 사용
* subprojects를 통해서 dependencies를 통해서 모든 모듈에 사용할 lombok을 추가했으며, 모든 모듈에 공통으로 적용된다.
* project('A') {B} : B의 설정값을 A 모듈에 추가하기 위해서 사용합니다.
* jar.enabled = true로 하면 공통 모듈의 Jar이 빌드 배포본에 포함된다.

***** 
## References
* [gradle buildscript-1](<https://stackoverflow.com/questions/13923766/gradle-buildscript-dependencies>)
* [gradle buildscript-2](<https://stackoverflow.com/questions/17773817/purpose-of-buildscript-block-in-gradle>)
* [공통모듈 build](https://gist.github.com/ihoneymon/a2ed116069af470fec0d08110604c5db})