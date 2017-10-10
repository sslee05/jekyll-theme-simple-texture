---
layout: post
title: "Spring Boot 2일차"
description: "Spring boot 자동구성 및 CLI"
categories: [spring-boot]
tags: [spring-boot]
redirect_from:
  - /2017/10/10/
---

> spring-boot 2일차


* Kramdown table of contents
{:toc .toc}

# spring-boot 자동구성 
@SpringBootApplication = @SpringBootConfiguration(@Configuration) + @ComponentScan + @EnableAutoConfiguration  


@EnableAutoConfiguration => AutoConfigurationImportSelector => SpringFactoriesLoader 가 spring-factories file를 read and decide instance  

spring-factories file은  
spring-boot-autoconfiguration.jar / META-INF/ spring-factories 에 각종 구성에 필요한 class들이 열거되어 있다  

이 목록들을 읽어 들여 SpringFactoriesLoader 가 필요한 component들을 조건에 맞는다면 자동 생성한다.  

spring-factories file에 열거되있는 java file 열어 자동 instance화 되는 조건의 annotation을 볼 필요가 있다.  

예)
@ConditionalOnClass(javax.transaction.Transaction.class)
@ConditionalOnProperty(prefix = "spring.jta", value = "enabled", matchIfMissing = true)


# SpringApplicationBuilder 
@SpringBootApplication 으로 선언한 file에 public static main method를 보면 SpringApplication 가 있다. 이를 build pattern을 이용하여 설정들을 좀더 편하게 해준다.  

{% highlight java %}
public static void main(String[] args) {
  //SpringApplication.run(SpringBootJournalApplication.class, args);
  new SpringApplicationBuilder(SpringBootJournalApplication.class)
      .bannerMode(Banner.Mode.CONSOLE)
      .logStartupInfo(true)
      .listeners(new ApplicationListener<ApplicationEvent>() {
				
        @Override
        public void onApplicationEvent(ApplicationEvent event) {
          log.info("==>"+event.getClass().getCanonicalName());
          //ApplicationStartedEvent: app를 시동할 때
          //ApplicationEnvironmentPreparedEvent: 처음 환경 준비를 마쳤을때 
          //ApplicationReadyEvent: app 준비가 끝났을 때 
          //ApplicationFailedEvent: 시동중 예외가 발생시 
        }
      })
      .run(args);
}
{% endhighlight %}

# SpringApplication arguement
SpringAppliction.run(args) 에서 실행시 arguements 받을 수 있다.  
args는 String[] 이것을 ApplicationArguements 의 형태로 좀더 편하게 받을 수 있는데  
이를 @Autowired 로 받으면 이를 이용하여 arguements를 boot 전에 이용할 수 있다.  

또는 ApplicationRunner 나 CommandLineRunner(spring batch 의 CommandLineRunner가 아님) interface의 run method 구현시 이를 받을 수 도 있다.  

특히 ApplicationRunner의 run method는 ApplicationArguements 인자를 받는다.
{% highlight java %}
public class SpringBootTestApplicaton implements ApplicationRunner {
 //... 중략  

 @Override
 public void run(ApplicationArguements args) throws Exception {
   //... 중략 
 }
}
{% endhighlight %}

# Properties
우선순위는 크게 보아서 다음과 같다.
1. 실행명령어에서 준 인자 (가장 우선순위가 높다.)
2. 환경변수
3. properties file 

# Properties file 위치
properies file 위치의 우선순의는 다음과 같다.  
1. classpath
2. cplasspath:/config
3. file:
4. file:config/

보통 개발,test,운영 마다 다른 환경에 따른 설정은  
application-dev.properties, application-test.properties, application-prod.properties  
처럼 application-이름.properties 로 하여 실행시 다음과 같이 하는 것이 좋겠다.  
실행시 -Dspring.profiles.acttive=prod 처럼  

default properties 를 확인 하려면 다음을 참조  

<https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html >

# CLI
## init 생성
아래 option이외에도 여러가지가 있음 어차피 다 왜우지 못하니 많이 쓰는 부분만 나머지는 필요할때 찾아 보기  
아래 option 명만 봐도 짐작 가능
{% highlight java %}
spring init --build gradle myapp -g=com.apres.spring -a=spring-boot-test --package=com.apres.spring
{% endhighlight %}

## test


{% highlight java %}
@RunWith(SpringRunner.class)
@SpringBootTest
public class MyTest {
  //....
}
{% endhighlight %}
위처럼 @SpringBootTest 인데 이때 아무런 attribute가 없을경우 해당 test java file과 같은 package에 @SpringBootApplication으로 선언된 java file이 있어야 됨.  
위치가 다를 경우 @SpringBootTest(classes = “Application.class”) 이런식으로 지정해야 함.  

예)  
src > main > com > example > Application.java  
src > test > com > example > MyTest.java

{% highlight console %}
spring test Test.java
{% endhighlight %}

## spring help
1. spring CLI 도움말
{% highlight java %}
spring --help
{% endhighlight %}

2. 명령에 대한 option 
{% highlight console %}
//예 jar 에 대해서
spring help jar
{% endhighlight %}

## 기타 여러 명령과 option
<https://docs.spring.io/spring-boot/docs/current/reference/html/cli-using-the-cli.html >

# CommandLineRunner
{% highlight java %}
package com.apres.spring;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class StandaloneBoot implements CommandLineRunner {

	@Override
	public void run(String... args) throws Exception {
		// TODO Auto-generated method stub
		//SpringApplication.run 호출한 이후 실행할 것들 정의 
	}
	
	@Bean
	InitializingBean beforeCommandRun() {
		return () -> {
			// doSomething
			//SpringApplication.run 호출한 이후 CommandLineRunner의 run 이전에 something do 
		};
	}
	
	public static void main(String[] args) throws Exception {
		SpringApplication.run(StandaloneBoot.class, args)
	}

}
{% endhighlight %}

# @ImportResource
기존 xml 기반의 spring 통합시 @ImportResource({ "classpath:batch/batch-context.xml" }) 이렇게 기존처럼 import 끝
{% highlight java %}
package com.apres.spring;

import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.ImportResource;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import com.sslee.batch.helper.JobParameterHolder;

@ImportResource({ "classpath:batch/batch-context.xml" })
@SpringBootApplication
public class StandaloneBoot  {

	@Autowired
	private JobParameterHolder paramHolder;

	public static void main(String[] args) throws Exception {
		SpringApplication.run(StandaloneBoot.class, args)
	}

}

{% endhighlight %}

# spring-boot enable
사용하고 싶은 것이 있다면 @Enable기술명을 사용하면 된다.
예)
@EnableJms
@EnableCaching
@EnableKafka
@EnableRabbit
....


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
