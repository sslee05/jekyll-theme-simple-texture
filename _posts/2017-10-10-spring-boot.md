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
<https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html/>


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
