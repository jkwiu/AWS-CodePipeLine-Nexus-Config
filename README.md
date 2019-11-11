# AWS-CodePipeLine-Nexus-Config

![CodePipeLine-Nexus-Deploy-Scenario](./images/CodePipeLine-Nexus-Deploy-Scenario.pdf)

1.빌드 시나리오

 1)개발자가 CodeCommit에 push

 2) CodePipeLine에서 코드 변경 감지

 3) 자동으로 build 후, Nexus서버에 deploy

2. 가이드

 pom.xml, settings.xml에 Nexus접속 정보를 설정해 준다.

buildspec.yml 파일 작성

 1) `pom.xml`
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.ibiz</groupId>
	<artifactId>nexus</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web-services</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
	<distributionManagement>
		<snapshotRepository>
		<id>nexus-snapshots</id>
		<url>http://<nexus url>:<port>/repository/maven-snapshots/</url>
		</snapshotRepository>
		<repository>
		<id>nexus-releases</id>
		<url>http://<nexus url>:<port>/repository/maven-releases/</url>
		</repository>
	</distributionManagement>

</project>
```

 2) `settings.xml`
```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.1.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd">

  <servers>
    <server>
      <id>nexus-snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <id>nexus-releases</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
  </servers>

  <mirrors>
    <mirror>
      <id>central</id>
      <name>central</name>
      <url>http://<nexus url>:<port>/repository/maven-central/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
</settings>
```
3) `buildspec.yml`
```
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto8
    commands:
      - echo Entered the install phase...
      - echo copy setting.xml to Local
      - cp ./config/settings.xml /root/.m2/settings.xml
    finally:
      - echo Copy finished
  build:
    commands:
      - echo MVN Build is Starting....
      - mvn deploy
    finally:
      - echo MVN Build Finished
artifacts:
  files:
    - config/settings.xml
cache:
  paths:
    - '/root/.m2/**/*'
```

4) AWS Config

 a. CodePipeLine 생성 클릭

  a-1) 파이프라인 설정(이름 설정, 서비스 역할 새 서비스 or 기존 서비스 선택)

  a-2) 소스 스테이지 추가(CodeCommit, Repository, Branch Name 선택, 변경감지는 둘중 택)

  a-3) 빌드 스테이지 추가

    - CodeBuild, 프로젝트 생성

    - 관리형 이미지(Amazon Linux2 Standard 1.0)

    - BuildSpec 사용(경로 지정)

    - 배포는 건너뛰기

    - 생성

끝.

  
