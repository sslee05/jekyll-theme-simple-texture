---
layout: post
title: "Spring Boot 1일차"
description: "Spring boot 로컬 환경구성"
categories: [배움]
tags: [spring-boot]
redirect_from:
  - /2017/10/09/
---

> spring-boot 1일차


* Kramdown table of contents
{:toc .toc}

# Spring-boot 에 대한 잘 못된 내 생각
spring-boot 를 할 필요가 있나? 라는 생각으로 무관심? 했다.  
사실 project를 하면서 project상황에 맞게 spring 제공하는 기본 구성을 component를 확장하거나 그럴수 없다면(변경하고 픈 곳이 private inner class를 사용하고 있는 경우 등) 아예 class를 새로 만들어 필요한 부분을 상황에 맞게 바꾸거나 조절 할 수 있는데 spring-boot는 그것이 가능한가? 라는 생각이 들어서이기도 했기 때문이다.  
지금은 boot이야기가 뜸하긴 하지만(이미 필수가 되어서 ?) 뒤처지는 느낌도 들고.. 함 보기로 하자.  

# Spring-boot mac OSX 에서 설치 
### 필요한 것들 
- jdk 1.6+ (난 1.8로 )
- gradle (gradle project로 구성시: 없으면 wrapper가 제공)
- maven (maven project로 구성시: 없으면 wrapper가 제공)
- CMI

### groovy 설치(groovy 예제를 하기위해 난 설치)
{% highlight console %}
ssleehome:~ sslee$ brew update
ssleehome:~ sslee$ brew install groovy
ssleehome:~ sslee$ groovy -version
Groovy Version: 2.4.12 JVM: 1.8.0_111 Vendor: Oracle Corporation OS: Mac OS X
{% endhighlight %}

### CLI (Command Line Interface) 설치
{% highlight console %}
ssleehome:~ sslee$ brew tap pivotal/tap
ssleehome:~ sslee$ brew install springboot
{% endhighlight %}

# spring-boot project 생성
## 생성방법 3가지
- CLI 로 생성
- spring initalizer 로 생성
- STS 나 eclipse(+ STS plugin)에서 생성

### CLI 로 생성
{% highlight console %}
ssleehome:bygradle sslee$ spring init -dweb,data-jpa,h2 --build gradle myapp
Using service at https://start.spring.io
Project extracted to '/Users/sslee/work/temp/cli/bygradle/myapp'
{% endhighlight %}

### spring initalizer 로 생성
https://start.spring.io/ 에 접속하여 생성

### STS 나 eclipse(+ STS plugin)에서 생성
project 생성 메뉴에서 Spring Starter project로 생성

## build.gradle
{% highlight groovy %}
buildscript {
	ext {
		springBootVersion = '1.5.7.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'

group = 'com.apress.spring'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter-data-jpa')
	compile('org.springframework.boot:spring-boot-starter-thymeleaf')
	compile('org.springframework.boot:spring-boot-starter-web')
	runtime('com.h2database:h2')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
{% endhighlight %}

- buildscript section  
  spring boot gradle plugin 정보  
- apply plugin section  
  gradle task 추가  
- repositories section  
  dependency repository  
- dependencies section  
  project에 필요한 depenedency 정보 

## 필수 dependency
- spring-boot-starter-parent  
  spring-core 등 spring 필수 dependency등이 선언  
- spring-boot-starter-기술명  
  spring-boot-start 에 필요한 dependency 등이 선언  
  
예)spring-boot-starter-web

## pom.xml 이나 build.gradle만 받아서 확인 해볼수 있음
{% highlight command %}
ssleehome:tmp sslee$ curl https://start.spring.io/build.gradle -o build.gradle -d dependencies=web,data-jpa,h2,thymeleaf
{% endhighlight %}

# starter-기술명 
내가 사용하고자 하는 starter를 선언하면 된다.  
그러면 start하는데 해당하는 dependency가 추가된다.  
org.springframework.boot:spring-boot-starter-data-jpa  
org.springframework.boot:spring-boot-starter-thymeleaf  
org.springframework.boot:spring-boot-starter-web  
등..  

# @SpringBootApplication
이것이 자동화 처럼 해주는 것이 었음.
SpringBootApplication = @SpringBootConfiguration(@Configuration) + @ComponentScan + @EnableAutoConfiguration

이곳에 attribute에 원하는 속성을 넣고 빼고, 원하는 기능들을 추가 하면 될 듯 해보임..


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
