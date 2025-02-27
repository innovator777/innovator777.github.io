---
layout: post
title: Spring Boot 개발을 위한 JDK, IDE 설치 및 설정
date: 2025-02-26
categories: [Spring Boot]
tags: [spring boot, java, jdk, ide, 개발환경]
---

Spring Boot 애플리케이션을 개발하기 위해서는 적절한 개발 환경이 필요합니다. 이번 포스팅에서는 Spring Boot 개발을 위한 JDK(Java Development Kit)와 IDE(통합 개발 환경) 설치 및 설정 방법을 단계별로 알아보겠습니다.

## JDK 설치하기

Spring Boot 3.0 이상 버전은 Java 17 이상을 요구합니다. 따라서 최신 버전의 JDK를 설치하는 것이 좋습니다.

### JDK 버전 선택하기

| Spring Boot 버전 | 권장 Java 버전 |
|-----------------|--------------|
| Spring Boot 3.2.x | Java 17 이상 |
| Spring Boot 3.0.x ~ 3.1.x | Java 17 이상 |
| Spring Boot 2.7.x | Java 8, 11 (Java 17 지원) |
| Spring Boot 2.5.x ~ 2.6.x | Java 8, 11 |
| Spring Boot 2.0.x ~ 2.4.x | Java 8 |

### Windows에서 JDK 설치

1. **JDK 다운로드**
   - [Oracle JDK](https://www.oracle.com/java/technologies/downloads/) 또는
   - [Eclipse Temurin(OpenJDK)](https://adoptium.net/) 사이트 방문
   - 운영체제에 맞는 Java 17 이상 버전 다운로드

2. **설치 파일 실행**
   - 다운로드한 설치 파일을 실행하고 안내에 따라 설치 진행
   - 기본 설치 경로: `C:\Program Files\Java\jdk-17`

3. **환경 변수 설정**
   - 시작 메뉴에서 '시스템 환경 변수 편집' 검색
   - '환경 변수' 버튼 클릭
   - '시스템 변수' 섹션에서 '새로 만들기' 클릭
   - 변수 이름: `JAVA_HOME`, 변수 값: JDK 설치 경로 (예: `C:\Program Files\Java\jdk-17`)
   - '시스템 변수' 섹션에서 'Path' 변수 선택 후 '편집' 클릭
   - '새로 만들기' 클릭 후 `%JAVA_HOME%\bin` 입력
   - 모든 창에서 '확인' 클릭

### macOS에서 JDK 설치

1. **Homebrew를 사용한 설치 (권장)**
   ```bash
   brew install openjdk@17
   ```
   설치 후 Homebrew가 안내하는 대로 심볼릭 링크 생성:
   ```bash
   sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
   ```

2. **수동 설치**
   - [Oracle JDK](https://www.oracle.com/java/technologies/downloads/) 또는 [Eclipse Temurin](https://adoptium.net/)에서 macOS용 패키지 다운로드
   - 다운로드한 패키지 파일(.pkg) 실행
   - 설치 마법사의 안내에 따라 설치 진행

3. **환경 변수 설정**
   - 터미널에서 다음 명령어 실행:
   ```bash
   echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 17)' >> ~/.zshrc
   echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.zshrc
   source ~/.zshrc
   ```

### Linux에서 JDK 설치

1. **Ubuntu/Debian**
   ```bash
   sudo apt update
   sudo apt install openjdk-17-jdk
   ```

2. **CentOS/Fedora/RHEL**
   ```bash
   sudo dnf install java-17-openjdk-devel
   ```

3. **환경 변수 설정**
   ```bash
   echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
   echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
   source ~/.bashrc
   ```

### JDK 설치 확인

터미널 또는 명령 프롬프트에서 다음 명령어를 실행하여 JDK가 올바르게 설치되었는지 확인합니다:

```bash
java -version
```

다음과 같은 출력이 나타나야 합니다 (버전은 다를 수 있음):
```
openjdk version "17.0.6" 2023-01-17
OpenJDK Runtime Environment (build 17.0.6+10)
OpenJDK 64-Bit Server VM (build 17.0.6+10, mixed mode, sharing)
```

## IDE 설치하기

Spring Boot 개발을 위한 주요 IDE로는 IntelliJ IDEA, Spring Tool Suite(STS), Visual Studio Code가 있습니다. 각각의 설치 방법을 알아보겠습니다.

### IntelliJ IDEA 설치 (권장)

IntelliJ IDEA는 JetBrains에서 개발한 강력한 Java IDE로, Spring Boot 개발에 최적화된 기능을 제공합니다.

1. **다운로드**
   - [IntelliJ IDEA 다운로드 페이지](https://www.jetbrains.com/idea/download/) 방문
   - Community Edition(무료) 또는 Ultimate Edition(유료, 30일 평가판 제공) 선택
   - 운영체제에 맞는 버전 다운로드

2. **설치**
   - Windows: 다운로드한 .exe 파일 실행 후 설치 마법사 따라 진행
   - macOS: 다운로드한 .dmg 파일 실행 후 Applications 폴더로 드래그
   - Linux: 다운로드한 압축 파일 해제 후 bin 디렉토리의 idea.sh 실행

3. **Spring Boot 플러그인 확인**
   - IntelliJ IDEA Ultimate Edition에는 Spring Boot 관련 플러그인이 기본 포함
   - Community Edition 사용 시 일부 Spring 기능 제한될 수 있음
   - File → Settings → Plugins에서 'Spring Boot' 검색하여 필요한 플러그인 설치

4. **JDK 설정**
   - File → Project Structure → Project 메뉴
   - Project SDK 설정에서 설치한 JDK 선택
   - 없을 경우 'Add SDK' → 'JDK'를 선택하고 JDK 설치 경로 지정

### Spring Tool Suite (STS) 설치

Spring Tool Suite는 Eclipse 기반의 Spring 전용 IDE입니다.

1. **다운로드**
   - [Spring Tool Suite 다운로드 페이지](https://spring.io/tools) 방문
   - 운영체제에 맞는 버전 다운로드

2. **설치**
   - Windows/Linux: 다운로드한 압축 파일 해제 후 SpringToolSuite4.exe(Windows) 또는 SpringToolSuite4(Linux) 실행
   - macOS: 다운로드한 .dmg 파일 실행 후 Applications 폴더로 드래그

3. **JDK 설정**
   - Window → Preferences → Java → Installed JREs
   - 'Add' 버튼 클릭 → 'Standard VM' 선택 → 'Next'
   - JRE home 또는 JDK home에 JDK 설치 경로 입력
   - 'Finish' 클릭 후 새로 추가한 JDK 체크

### Visual Studio Code 설치

Visual Studio Code는 가볍고 확장성이 뛰어난 코드 에디터로, 적절한 확장 프로그램을 설치하면 Spring Boot 개발에 활용할 수 있습니다.

1. **다운로드 및 설치**
   - [Visual Studio Code 다운로드 페이지](https://code.visualstudio.com/) 방문
   - 운영체제에 맞는 버전 다운로드 및 설치

2. **Java 확장 프로그램 설치**
   - VS Code 실행 후 왼쪽 사이드바의 Extensions 아이콘 클릭
   - 다음 확장 프로그램 검색 및 설치:
     - Extension Pack for Java
     - Spring Boot Extension Pack
     - Spring Boot Dashboard
     - Spring Initializr Java Support
     - Maven for Java

3. **JDK 설정**
   - VS Code 설정(File → Preferences → Settings)에서 'java.home' 검색
   - 'Edit in settings.json' 클릭
   - 다음과 같이 JDK 경로 설정:
   ```json
   {
     "java.home": "C:\\Program Files\\Java\\jdk-17",
     "java.configuration.updateBuildConfiguration": "automatic"
   }
   ```

## IDE 비교 및 선택 가이드

| 기능 | IntelliJ IDEA | Spring Tool Suite | Visual Studio Code |
|-----|--------------|-------------------|-------------------|
| Spring Boot 지원 | 매우 우수 (Ultimate) | 우수 | 양호 (확장 프로그램 필요) |
| 자동 완성 | 매우 우수 | 우수 | 양호 |
| 디버깅 | 매우 우수 | 우수 | 양호 |
| 리팩토링 | 매우 우수 | 우수 | 기본적 |
| 성능 | 높은 메모리 사용 | 중간 메모리 사용 | 낮은 메모리 사용 |
| 학습 곡선 | 중간~높음 | 중간 | 낮음 |
| 라이센스 | 유료(Ultimate)/무료(Community) | 무료 | 무료 |

### 초보자를 위한 추천
- **Spring Tool Suite**: Spring 특화 기능과 직관적인 인터페이스
- **IntelliJ IDEA Community**: 강력한 기능과 무료 버전 제공

### 전문 개발자를 위한 추천
- **IntelliJ IDEA Ultimate**: 최고의 Spring Boot 개발 경험
- **Visual Studio Code**: 다양한 언어 지원과 가벼운 성능 필요 시

## Spring Boot 프로젝트 생성 테스트

개발 환경 설정이 완료되었다면, 간단한 Spring Boot 프로젝트를 생성하여 테스트해 봅시다.

### IntelliJ IDEA에서 프로젝트 생성

1. File → New → Project 선택
2. 왼쪽 패널에서 'Spring Initializr' 선택
3. 프로젝트 설정:
   - Name: 프로젝트 이름
   - Language: Java
   - Type: Maven 또는 Gradle
   - Group: com.example
   - Artifact: demo
   - Package name: com.example.demo
   - JDK: 설치한 JDK 버전
   - Java: 17 이상
   - Packaging: Jar
4. 'Next' 클릭
5. 의존성 추가:
   - 'Spring Web' 선택
6. 'Create' 클릭

### Spring Tool Suite에서 프로젝트 생성

1. File → New → Spring Starter Project 선택
2. 프로젝트 설정:
   - Name: 프로젝트 이름
   - Type: Maven 또는 Gradle
   - Packaging: Jar
   - Java Version: 17
   - Group: com.example
   - Artifact: demo
3. 'Next' 클릭
4. 의존성 추가:
   - 'Web' 카테고리에서 'Spring Web' 선택
5. 'Finish' 클릭

### Visual Studio Code에서 프로젝트 생성

1. View → Command Palette (Ctrl+Shift+P 또는 Cmd+Shift+P)
2. 'Spring Initializr: Create a Maven Project' 입력 및 선택
3. Spring Boot 버전 선택
4. 프로젝트 언어로 'Java' 선택
5. Group ID 입력 (예: com.example)
6. Artifact ID 입력 (예: demo)
7. 패키징 타입 선택 (Jar 권장)
8. Java 버전 선택 (17 이상)
9. 의존성 추가:
   - 'Spring Web' 선택
10. 프로젝트 저장 위치 선택

### 프로젝트 실행 테스트

1. 생성된 프로젝트의 메인 클래스 실행:
   - IntelliJ IDEA: 메인 클래스에서 우클릭 → Run
   - STS: 프로젝트 우클릭 → Run As → Spring Boot App
   - VS Code: 메인 클래스 열고 Run 버튼 클릭

2. 콘솔에서 다음과 같은 메시지 확인:
   ```
   .   ____          _            __ _ _
   /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
   ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
   \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
   '  |____| .__|_| |_|_| |_\__, | / / / /
   =========|_|==============|___/=/_/_/_/
   :: Spring Boot ::                (v3.x.x)
   
   ... 애플리케이션 시작 로그 ...
   ```

3. 웹 브라우저에서 `http://localhost:8080` 접속
   - 기본 페이지가 없으므로 Whitelabel Error Page가 표시되는 것이 정상

## 간단한 REST API 만들기

개발 환경이 제대로 설정되었는지 확인하기 위해 간단한 REST API를 만들어 봅시다.

1. 프로젝트에 새 Java 클래스 생성:
   - 패키지: com.example.demo.controller
   - 클래스명: HelloController

2. 다음 코드 작성:

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
```

3. 애플리케이션 실행 (또는 재실행)

4. 웹 브라우저에서 `http://localhost:8080/hello` 접속
   - "Hello, Spring Boot!" 메시지가 표시되면 성공

## 결론

이번 포스팅에서는 Spring Boot 개발을 위한 JDK와 IDE 설치 및 설정 방법을 알아보았습니다. 개발 환경 설정은 프로젝트의 성공적인 시작을 위한 중요한 단계입니다. 자신의 개발 스타일과 프로젝트 요구사항에 맞는 IDE를 선택하고, 최신 버전의 JDK를 설치하여 Spring Boot의 모든 기능을 활용할 수 있도록 하세요.

다음 포스팅에서는 Spring Boot 프로젝트의 기본 구조와 주요 설정 파일에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [JDK 17 다운로드](https://www.oracle.com/java/technologies/downloads/#java17)
- [IntelliJ IDEA 문서](https://www.jetbrains.com/idea/documentation/)
- [Spring Tool Suite 문서](https://spring.io/tools)
- [Visual Studio Code Java 튜토리얼](https://code.visualstudio.com/docs/java/java-tutorial) 