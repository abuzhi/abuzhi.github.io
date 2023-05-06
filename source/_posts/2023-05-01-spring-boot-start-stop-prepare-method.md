---
title: Spring启动初始化及关闭前执行方法探究
author: abuzhi
date: 2023-05-01 18:32:00
categories: [Java]
tags: [Java,SpringBoot]
---

Spring启动初始化及关闭前执行方法探究


# 前言

最近项目中有用到spring boot及web，在初始化执行方法时，遇到了重复执行的问题。

为查找原因，现对spring启动及关闭相关流程进行简单探究，主要针对spring 容器启动完成后，想预期执行某些初始化方法的执行，及关闭前执行方法。

另外需要深入理清Listener模式下的各Event事件触发的顺序及情景。

新建spring boot web项目，加入常用的一些初始化方法及关闭前方法。

项目结构

![](/images/2023-05-01/project.png)


主方法

```java
public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(WebStarter.class, args);
        System.out.println("context 启动成功");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        SpringApplication.exit(context, (ExitCodeGenerator) () -> 1);
        /**
         *  hook要加在代码最后才能执行
         */
        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
            @Override
            public void run() {
                ClosePreHook.close();
            }
        }));
    }
```

# Spring启动初始化方法

spring启动后加载方式一般有如下几种

另外，spring如果用的xml方式声明bean，也可以在xml中进行init ，pre destroy等配置，这里暂不讨论。

## CommandLineRunner 接口

```java
@Component
public class AppInitA implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println(System.currentTimeMillis() + " init CommandLineRunner : args=" + args);
    }
}
```

## ApplicationRunner 接口

```java
@Component
public class AppInitB implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(System.currentTimeMillis() + " init ApplicationRunner : args=" + args.getSourceArgs());
    }
}
```

## ApplicationListener 接口

此接口为事件驱动的，对应的event有很多种，不同的event对应事件不一样，如果不清楚对应触发时刻就很可能踩坑
一般用法如下：

```java
@Component
public class AppInitC implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println(System.currentTimeMillis() + " init ApplicationListener  ContextRefreshedEvent : event=" );
    }
}
```

## InitializingBean接口

```java
@Component
@Slf4j
public class InitInitializingBean implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        Thread.sleep(3000);
        log.info(" init " + this.getClass().getSimpleName());
    }
}
```

## PostConstruct注解

注解也可以，如下：

```java
@Component
@Slf4j
public class InitPostConstruct {

    @PostConstruct
    public void init(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info(" init " + this.getClass().getSimpleName());

    }
}
```

# Spring关闭前执行方法

目前发现有三种方式：


## @PreDestroy注解

```java
@Component
@Slf4j
public class ClosePreDestroy {
    @PreDestroy
    public void init(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info(" close PreDestroy" + this.getClass().getSimpleName());
    }

}
```

## jvm的addShutdownHook

```java
@Slf4j
public class ClosePreHook {
    public static void close(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info(" close ClosePreHook");
    }

}

// 主main中加入：

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
            @Override
            public void run() {
                ClosePreHook.close();
            }
        }));
```

## ApplicationListener 接口

目前发现只有两个接口是纯关闭前执行的：ExitCodeEvent（必须exit code非0才可以调用到），ContextClosedEvent

```java
@Component
@Slf4j
public class CloseListenerExitCodeEvent implements ApplicationListener<ExitCodeEvent> {

    @Override
    public void onApplicationEvent(ExitCodeEvent event) {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info(" close " + this.getClass().getSimpleName());
    }
}
```

# 启动加载及关闭的执行顺序

写一个web demo，加入各不同的方式，每个方法执行3s，观察输出

输出日志简略如下：

```shell
2023-05-05 19:41:07.840 default [main] INFO  com.xiao.demo.boot.WebStarter[640] - The following 1 profile is active: "dev"
2023-05-05 19:41:08.570 default [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer[108] - Tomcat initialized with port(s): 18080 (http)
2023-05-05 19:41:08.575 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Initializing ProtocolHandler ["http-nio-18080"]
2023-05-05 19:41:08.576 default [main] INFO  o.a.catalina.core.StandardService[173] - Starting service [Tomcat]
2023-05-05 19:41:08.576 default [main] INFO  o.a.catalina.core.StandardEngine[173] - Starting Servlet engine: [Apache Tomcat/9.0.65]
2023-05-05 19:41:08.645 default [main] INFO  o.a.c.c.C.[Tomcat].[localhost].[/][173] - Initializing Spring embedded WebApplicationContext
2023-05-05 19:41:08.645 default [main] INFO  o.s.b.w.s.c.ServletWebServerApplicationContext[292] - Root WebApplicationContext: initialization completed in 779 ms
2023-05-05 19:41:11.884 default [main] INFO  c.x.d.boot.init.InitInitializingBean[13] -  init InitInitializingBean
2023-05-05 19:41:14.893 default [main] INFO  c.x.demo.boot.init.InitPostConstruct[20] -  init PostConstructInitPostConstruct
2023-05-05 19:41:17.908 default [main] INFO  c.x.demo.boot.init.InitPostConstruct[30] -  init afterPropertiesSetInitPostConstruct
2023-05-05 19:41:18.202 default [main] INFO  o.s.b.a.e.web.EndpointLinksResolver[58] - Exposing 5 endpoint(s) beneath base path ''
2023-05-05 19:41:18.221 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Starting ProtocolHandler ["http-nio-18080"]
2023-05-05 19:41:18.232 default [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer[220] - Tomcat started on port(s): 18080 (http) with context path ''
2023-05-05 19:41:21.236 default [main] INFO  c.x.d.b.i.InitListenerServletWebServerInitializedEvent[20] -  init InitListenerServletWebServerInitializedEvent
2023-05-05 19:41:24.246 default [main] INFO  c.x.d.b.i.InitListenerWebServerInitializedEvent[19] -  init InitListenerWebServerInitializedEvent
2023-05-05 19:41:27.278 default [main] INFO  c.x.d.b.i.InitListenerApplicationContextEvent[19] -  init InitListenerApplicationContextEvent
2023-05-05 19:41:30.287 default [main] INFO  c.x.d.b.i.InitListenerContextRefreshedEvent[19] -  init InitListenerContextRefreshedEvent
2023-05-05 19:41:30.295 default [main] INFO  com.xiao.demo.boot.WebStarter[61] - Started WebStarter in 22.662 seconds (JVM running for 23.489)
2023-05-05 19:41:33.299 default [main] INFO  c.x.d.b.i.InitListenerApplicationStartedEvent[19] -  init InitListenerApplicationStartedEvent
2023-05-05 19:41:36.300 default [main] INFO  c.x.d.b.i.InitListenerSpringApplicationEvent[20] -  init InitListenerSpringApplicationEvent
2023-05-05 19:41:39.324 default [main] INFO  c.x.d.b.i.InitListenerAvailabilityChangeEvent[20] -  init InitListenerAvailabilityChangeEvent
2023-05-05 19:41:42.328 default [main] INFO  c.x.d.b.i.InitListenerPayloadApplicationEvent[20] -  init InitListenerPayloadApplicationEvent
2023-05-05 19:41:45.339 default [main] INFO  c.x.d.b.init.InitApplicationRunner[14] -  init InitApplicationRunner
2023-05-05 19:41:48.346 default [main] INFO  c.x.d.b.init.InitCommandLineRunner[13] -  init InitCommandLineRunner
2023-05-05 19:41:49.026 default [RMI TCP Connection(5)-192.168.137.1] INFO  o.a.c.c.C.[Tomcat].[localhost].[/][173] - Initializing Spring DispatcherServlet 'dispatcherServlet'
2023-05-05 19:41:49.026 default [RMI TCP Connection(5)-192.168.137.1] INFO  o.s.web.servlet.DispatcherServlet[525] - Initializing Servlet 'dispatcherServlet'
2023-05-05 19:41:49.027 default [RMI TCP Connection(5)-192.168.137.1] INFO  o.s.web.servlet.DispatcherServlet[547] - Completed initialization in 1 ms
2023-05-05 19:41:49.028 default [RMI TCP Connection(4)-192.168.137.1] INFO  com.zaxxer.hikari.HikariDataSource[110] - HikariPool-1 - Starting...
2023-05-05 19:41:49.178 default [RMI TCP Connection(4)-192.168.137.1] INFO  com.zaxxer.hikari.HikariDataSource[123] - HikariPool-1 - Start completed.
2023-05-05 19:41:51.347 default [main] INFO  c.x.d.b.i.InitListenerApplicationReadyEvent[19] -  init InitListenerApplicationReadyEvent
2023-05-05 19:41:54.361 default [main] INFO  c.x.d.b.i.InitListenerSpringApplicationEvent[20] -  init InitListenerSpringApplicationEvent
2023-05-05 19:41:57.366 default [main] INFO  c.x.d.b.i.InitListenerAvailabilityChangeEvent[20] -  init InitListenerAvailabilityChangeEvent
2023-05-05 19:42:00.380 default [main] INFO  c.x.d.b.i.InitListenerPayloadApplicationEvent[20] -  init InitListenerPayloadApplicationEvent
context 启动成功
2023-05-05 19:42:17.964 default [main] INFO  c.x.d.b.i.CloseListenerExitCodeEvent[20] -  close CloseListenerExitCodeEvent
2023-05-05 19:42:27.394 default [main] INFO  c.x.d.b.i.InitListenerAvailabilityChangeEvent[20] -  init InitListenerAvailabilityChangeEvent
2023-05-05 19:42:30.397 default [main] INFO  c.x.d.b.i.InitListenerPayloadApplicationEvent[20] -  init InitListenerPayloadApplicationEvent
2023-05-05 19:42:33.403 default [main] INFO  c.x.d.b.i.CloseListenerContextClosedEvent[20] -  close CloseListenerContextClosedEvent
2023-05-05 19:42:36.419 default [main] INFO  c.x.d.b.i.InitListenerApplicationContextEvent[19] -  init InitListenerApplicationContextEvent
2023-05-05 19:42:36.685 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Pausing ProtocolHandler ["http-nio-18080"]
2023-05-05 19:42:36.686 default [main] INFO  o.a.catalina.core.StandardService[173] - Stopping service [Tomcat]
2023-05-05 19:42:36.689 default [main] INFO  o.a.c.c.C.[Tomcat].[localhost].[/][173] - Destroying Spring FrameworkServlet 'dispatcherServlet'
2023-05-05 19:42:36.698 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Stopping ProtocolHandler ["http-nio-18080"]
2023-05-05 19:42:36.701 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Destroying ProtocolHandler ["http-nio-18080"]
2023-05-05 19:42:39.725 default [main] INFO  c.x.demo.boot.init.ClosePreDestroy[21] -  close PreDestroyClosePreDestroy
2023-05-05 19:42:39.750 default [main] INFO  com.zaxxer.hikari.HikariDataSource[350] - HikariPool-1 - Shutdown initiated...
2023-05-05 19:42:39.767 default [main] INFO  com.zaxxer.hikari.HikariDataSource[352] - HikariPool-1 - Shutdown completed.
Disconnected from the target VM, address: '127.0.0.1:7070', transport: 'socket'

```

通过日志观察各初始化方式的顺序，有这么几个阶段：
- **1.tomcat启动，初始化bean及注解优先，可见InitializingBean和PostConstruct是最先开始初始化的**
- **2.ApplicationRunner和CommandLineRunner接口基本同时初始化**
- **3.Listener接口的实现中，根据不同的类型，会有对应的容器启动中，启动后，关闭前等各种情况的触发，所以Listener接口的实现中，加载时间会比较复杂，需要明白各event的明确含义再进行使用。**
- **4.event类中，有一些事件是会在整个容器周期内有多次触发的，如：PayloadApplicationEvent**

# 探究源码

要搞清楚各触发阶段，需要跟进源码中观察，后再根据源码流程，梳理出event事件的顺序

## 启动初始化及关闭流程源码执行

源码太多，跟进后，画了一个时序图，可以直观明了的看到各加载时间顺序。

这里只重点标明各初始化及关闭前的各方法触发节点，不对spring加载过程做细说，如想了解可参考其他博客文章

**主要代码在SpringApplication.run和AbstractApplicationContext.refresh两个方法中**

```java
// SpringApplication.run
public ConfigurableApplicationContext run(String... args) {
		long startTime = System.nanoTime();
		DefaultBootstrapContext bootstrapContext = createBootstrapContext();
		ConfigurableApplicationContext context = null;
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting(bootstrapContext, this.mainApplicationClass);
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			context.setApplicationStartup(this.applicationStartup);
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
            /** 步骤1，2，3，4 **/
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), timeTakenToStartup);
			}
            /** 步骤5 **/
			listeners.started(context, timeTakenToStartup);
            /** 步骤5 **/
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, listeners);
			throw new IllegalStateException(ex);
		}
		try {
			Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
			listeners.ready(context, timeTakenToReady);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}

    // AbstractApplicationContext.refresh
    @Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
			prepareRefresh();
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			prepareBeanFactory(beanFactory);
			try {
				postProcessBeanFactory(beanFactory);
				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				invokeBeanFactoryPostProcessors(beanFactory);
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();
				initMessageSource();
				initApplicationEventMulticaster();
            
				onRefresh();
				registerListeners();
                /** 步骤1，2 **/
				finishBeanFactoryInitialization(beanFactory);
                /** 步骤3，4 **/
				finishRefresh();
			}
			....
		}
	}

```

**过程如下：**

![](/images/2023-05-01/apprun.jpg)


## 步骤1和2，InitializingBean和PostConstruct

在AbstractApplicationContext 中的finishBeanFactoryInitialization方法中完成的，是在各bean完成初始化后，执行init method时执行的。

这两个属于同级别的执行，执行顺序上以源码顺序为主，

所以，如果在同一个类中即实现了InitializingBean接口，同时也加入了PostConstruct注解，如：

![](/images/2023-05-01/InitializingBean.png)

实际执行中，此类下PostConstruct优先于afterPropertiesSet执行，这里需要注意

```java
@Component
@Slf4j
public class InitPostConstruct implements InitializingBean {

    @PostConstruct
    public void init(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info(" init PostConstruct" + this.getClass().getSimpleName());
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info(" init afterPropertiesSet" + this.getClass().getSimpleName());
    }
}
```

## 步骤3，ServletWebServerInitializedEvent

spring getLifecycleProcessor().onRefresh() 加载各bean完成，web容器初始化成功后，

WebServerStartStopLifecycle触发ServletWebServerInitializedEvent事件，

由于ServletWebServerInitializedEvent是继承自WebServerInitializedEvent，所以此事件也会触发。

这里实际会触发两个event：ServletWebServerInitializedEvent , WebServerInitializedEvent


步骤4，ContextRefreshedEvent
执行完refresh最后，直接触发ContextRefreshedEvent事件，
ContextRefreshedEvent的父类ApplicationContextEvent也同时会触发一次
本调用也是触发了两个事件：ContextRefreshedEvent，ApplicationContextEvent
执行完后，SpringApplication run中的afterRefresh执行结束


步骤5，listeners.started 相关事件
这里触发的是spring start启动完成后的event事件：
ApplicationStartedEvent及其父类SpringApplicationEvent;
AvailabilityChangeEvent及其父类PayloadApplicationEvent;

步骤6，callRunners
这里触发两个runner：ApplicationRunner，CommandLineRunner
同级执行


步骤7，listeners.ready 相关事件
spring web启动完成后触发的事件，
ApplicationReadyEvent及其父类SpringApplicationEvent；
AvailabilityChangeEvent及其父类PayloadApplicationEvent；

步骤8，exit
ExitCodeEvent 触发：当SpringApplication.exit(context, (ExitCodeGenerator) () -> 1); exit code为非0时触发

步骤9，ContextClosedEvent
触发spring close event事件：
AvailabilityChangeEvent及其父类PayloadApplicationEvent；
ContextClosedEvent及其父类ApplicationContextEvent；


步骤10和11，最后处理
步骤10先调用bean的@PreDestroy 注解的方法
步骤11，spring关闭后，jvm触发shutdown hook：
Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
    @Override
    public void run() {
        ClosePreHook.close();
    }
}));

梳理Listener模式下的各Event
在springboot及web开发中，常用主要的几个event类在下面几个包中，这里不讨论spring data，spring cloud相关的event，大体上是一样的
org.springframework.context.event 
org.springframework.boot.context.event
org.springframework.boot.web.context
基础类都继承自 org.springframework.context.ApplicationEvent
关系如下图，这里有spring boot下的7个，context下4个，web下3个，另有几个单独的，如果再加入spring data，cloud，jpa等相关的就很多了，这里只讨论基础常用的下图中这些

先搞清楚各event事件的类关系，各event类顶层是ApplicationEvent抽象类，子类有五个常用
ApplicationEvent
- ApplicationContextEvent
- ExitCodeEvent
- PayloadApplicationEvent
- WebServerInitializedEvent
- SpringApplicationEvent

其中需要注意的是几个容易重复触发的事件：
SpringApplicationEvent，PayloadApplicationEvent，ApplicationContextEvent
这三个类调用基本是由于子类触发的，如果同一个项目中，如果有同时声明父类和子类的触发，则很容易触发父类的方法两次，所以谨慎使用上面三个事件，子类也在不同情况下会有可能触发两次
所以，按需求，在不同的阶段进行调用。推荐最好都不用event进行初始化或者关闭前执行方法

加入apm后event加载两次问题
项目中出现问题是主程序引入了apm agent后，有两次触发的情况，对上面demo代码加入apm 配置：
application.metrics.name=boot_web_@spring.profiles.active@
management.endpoints.web.base-path=/
management.endpoints.web.exposure.include=prometheus,health,info,metrics,env
management.server.port=12345
启动后，执行相同的代码，日志输出如下：
2023-05-06 16:28:21.138 default [main] INFO  com.xiao.demo.boot.WebStarter[640] - The following 1 profile is active: "dev"
2023-05-06 16:28:21.836 default [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer[108] - Tomcat initialized with port(s): 18080 (http)
2023-05-06 16:28:21.842 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Initializing ProtocolHandler ["http-nio-18080"]
2023-05-06 16:28:21.842 default [main] INFO  o.a.catalina.core.StandardService[173] - Starting service [Tomcat]
2023-05-06 16:28:21.842 default [main] INFO  o.a.catalina.core.StandardEngine[173] - Starting Servlet engine: [Apache Tomcat/9.0.65]
2023-05-06 16:28:21.912 default [main] INFO  o.a.c.c.C.[Tomcat].[localhost].[/][173] - Initializing Spring embedded WebApplicationContext
2023-05-06 16:28:21.913 default [main] INFO  o.s.b.w.s.c.ServletWebServerApplicationContext[292] - Root WebApplicationContext: initialization completed in 747 ms
2023-05-06 16:28:25.128 default [main] INFO  c.x.d.boot.init.InitInitializingBean[13] -  init InitInitializingBean
2023-05-06 16:28:28.148 default [main] INFO  c.x.demo.boot.init.InitPostConstruct[20] -  init PostConstructInitPostConstruct
2023-05-06 16:28:31.157 default [main] INFO  c.x.demo.boot.init.InitPostConstruct[30] -  init afterPropertiesSetInitPostConstruct
2023-05-06 16:28:44.773 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Starting ProtocolHandler ["http-nio-18080"]
2023-05-06 16:28:44.787 default [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer[220] - Tomcat started on port(s): 18080 (http) with context path ''
2023-05-06 16:28:52.083 default [main] INFO  c.x.d.b.i.InitListenerServletWebServerInitializedEvent[20] -  init InitListenerServletWebServerInitializedEvent
2023-05-06 16:28:55.091 default [main] INFO  c.x.d.b.i.InitListenerWebServerInitializedEvent[19] -  init InitListenerWebServerInitializedEvent
2023-05-06 16:28:55.143 default [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer[108] - Tomcat initialized with port(s): 12345 (http)
2023-05-06 16:28:55.144 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Initializing ProtocolHandler ["http-nio-12345"]
2023-05-06 16:28:55.144 default [main] INFO  o.a.catalina.core.StandardService[173] - Starting service [Tomcat]
2023-05-06 16:28:55.144 default [main] INFO  o.a.catalina.core.StandardEngine[173] - Starting Servlet engine: [Apache Tomcat/9.0.65]
2023-05-06 16:28:55.152 default [main] INFO  o.a.c.c.C.[Tomcat-1].[localhost].[/][173] - Initializing Spring embedded WebApplicationContext
2023-05-06 16:28:55.152 default [main] INFO  o.s.b.w.s.c.ServletWebServerApplicationContext[292] - Root WebApplicationContext: initialization completed in 56 ms
2023-05-06 16:28:55.159 default [main] INFO  o.s.b.a.e.web.EndpointLinksResolver[58] - Exposing 5 endpoint(s) beneath base path ''
2023-05-06 16:29:03.805 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Starting ProtocolHandler ["http-nio-12345"]
2023-05-06 16:29:03.809 default [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer[220] - Tomcat started on port(s): 12345 (http) with context path ''
2023-05-06 16:29:06.819 default [main] INFO  c.x.d.b.i.InitListenerServletWebServerInitializedEvent[20] -  init InitListenerServletWebServerInitializedEvent
2023-05-06 16:29:09.830 default [main] INFO  c.x.d.b.i.InitListenerWebServerInitializedEvent[19] -  init InitListenerWebServerInitializedEvent
2023-05-06 16:29:16.630 default [main] INFO  c.x.d.b.i.InitListenerApplicationContextEvent[19] -  init InitListenerApplicationContextEvent
2023-05-06 16:29:19.642 default [main] INFO  c.x.d.b.i.InitListenerContextRefreshedEvent[19] -  init InitListenerContextRefreshedEvent
2023-05-06 16:29:53.836 default [main] INFO  c.x.d.b.i.InitListenerApplicationContextEvent[19] -  init InitListenerApplicationContextEvent
2023-05-06 16:29:56.843 default [main] INFO  c.x.d.b.i.InitListenerContextRefreshedEvent[19] -  init InitListenerContextRefreshedEvent
2023-05-06 16:29:56.849 default [main] INFO  com.xiao.demo.boot.WebStarter[61] - Started WebStarter in 95.942 seconds (JVM running for 96.735)
2023-05-06 16:29:59.862 default [main] INFO  c.x.d.b.i.InitListenerApplicationStartedEvent[19] -  init InitListenerApplicationStartedEvent
2023-05-06 16:30:02.877 default [main] INFO  c.x.d.b.i.InitListenerSpringApplicationEvent[20] -  init InitListenerSpringApplicationEvent
2023-05-06 16:30:05.887 default [main] INFO  c.x.d.b.i.InitListenerAvailabilityChangeEvent[20] -  init InitListenerAvailabilityChangeEvent
2023-05-06 16:30:08.895 default [main] INFO  c.x.d.b.i.InitListenerPayloadApplicationEvent[20] -  init InitListenerPayloadApplicationEvent
2023-05-06 16:30:11.907 default [main] INFO  c.x.d.b.init.InitApplicationRunner[14] -  init InitApplicationRunner
2023-05-06 16:30:14.920 default [main] INFO  c.x.d.b.init.InitCommandLineRunner[13] -  init InitCommandLineRunner
2023-05-06 16:30:15.412 default [RMI TCP Connection(5)-192.168.137.1] INFO  o.a.c.c.C.[Tomcat].[localhost].[/][173] - Initializing Spring DispatcherServlet 'dispatcherServlet'
2023-05-06 16:30:15.413 default [RMI TCP Connection(5)-192.168.137.1] INFO  o.s.web.servlet.DispatcherServlet[525] - Initializing Servlet 'dispatcherServlet'
2023-05-06 16:30:15.414 default [RMI TCP Connection(5)-192.168.137.1] INFO  o.s.web.servlet.DispatcherServlet[547] - Completed initialization in 1 ms
2023-05-06 16:30:15.415 default [RMI TCP Connection(4)-192.168.137.1] INFO  com.zaxxer.hikari.HikariDataSource[110] - HikariPool-1 - Starting...
2023-05-06 16:30:15.570 default [RMI TCP Connection(4)-192.168.137.1] INFO  com.zaxxer.hikari.HikariDataSource[123] - HikariPool-1 - Start completed.
2023-05-06 16:30:17.938 default [main] INFO  c.x.d.b.i.InitListenerApplicationReadyEvent[19] -  init InitListenerApplicationReadyEvent
2023-05-06 16:30:20.950 default [main] INFO  c.x.d.b.i.InitListenerSpringApplicationEvent[20] -  init InitListenerSpringApplicationEvent
2023-05-06 16:30:23.964 default [main] INFO  c.x.d.b.i.InitListenerAvailabilityChangeEvent[20] -  init InitListenerAvailabilityChangeEvent
2023-05-06 16:30:26.969 default [main] INFO  c.x.d.b.i.InitListenerPayloadApplicationEvent[20] -  init InitListenerPayloadApplicationEvent
context 启动成功
2023-05-06 16:31:07.872 default [main] INFO  c.x.d.b.i.CloseListenerExitCodeEvent[20] -  close CloseListenerExitCodeEvent
2023-05-06 16:31:13.741 default [main] INFO  c.x.d.b.i.InitListenerAvailabilityChangeEvent[20] -  init InitListenerAvailabilityChangeEvent
2023-05-06 16:31:16.752 default [main] INFO  c.x.d.b.i.InitListenerPayloadApplicationEvent[20] -  init InitListenerPayloadApplicationEvent
2023-05-06 16:31:19.766 default [main] INFO  c.x.d.b.i.CloseListenerContextClosedEvent[20] -  close CloseListenerContextClosedEvent
2023-05-06 16:31:22.777 default [main] INFO  c.x.d.b.i.InitListenerApplicationContextEvent[19] -  init InitListenerApplicationContextEvent
2023-05-06 16:31:25.788 default [main] INFO  c.x.d.b.i.InitListenerAvailabilityChangeEvent[20] -  init InitListenerAvailabilityChangeEvent
2023-05-06 16:31:28.798 default [main] INFO  c.x.d.b.i.InitListenerPayloadApplicationEvent[20] -  init InitListenerPayloadApplicationEvent
2023-05-06 16:31:31.810 default [main] INFO  c.x.d.b.i.CloseListenerContextClosedEvent[20] -  close CloseListenerContextClosedEvent
2023-05-06 16:31:34.818 default [main] INFO  c.x.d.b.i.InitListenerApplicationContextEvent[19] -  init InitListenerApplicationContextEvent
2023-05-06 16:31:35.084 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Pausing ProtocolHandler ["http-nio-12345"]
2023-05-06 16:31:35.085 default [main] INFO  o.a.catalina.core.StandardService[173] - Stopping service [Tomcat]
2023-05-06 16:31:35.097 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Stopping ProtocolHandler ["http-nio-12345"]
2023-05-06 16:31:35.101 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Destroying ProtocolHandler ["http-nio-12345"]
2023-05-06 16:31:41.645 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Pausing ProtocolHandler ["http-nio-18080"]
2023-05-06 16:31:41.646 default [main] INFO  o.a.catalina.core.StandardService[173] - Stopping service [Tomcat]
2023-05-06 16:31:41.646 default [main] INFO  o.a.c.c.C.[Tomcat].[localhost].[/][173] - Destroying Spring FrameworkServlet 'dispatcherServlet'
2023-05-06 16:31:41.650 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Stopping ProtocolHandler ["http-nio-18080"]
2023-05-06 16:31:41.653 default [main] INFO  o.a.coyote.http11.Http11NioProtocol[173] - Destroying ProtocolHandler ["http-nio-18080"]
2023-05-06 16:31:46.609 default [main] INFO  c.x.demo.boot.init.ClosePreDestroy[21] -  close PreDestroyClosePreDestroy
2023-05-06 16:31:46.634 default [main] INFO  com.zaxxer.hikari.HikariDataSource[350] - HikariPool-1 - Shutdown initiated...
2023-05-06 16:31:46.651 default [main] INFO  com.zaxxer.hikari.HikariDataSource[352] - HikariPool-1 - Shutdown completed.
Disconnected from the target VM, address: '127.0.0.1:11460', transport: 'socket'

Process finished with exit code 0


加入apm agent后，由于启动时实际是启动了两个web server对应两个不同的web端口，一个是本应用的端口18080，另外一个为apm 收集metrics的端口12345
所以实际启动中，会触发两次web 初始化event，content refresh event，change event等，如果预期初始化的方法写在了用listener 实现，那么有极大可能导致方法会执行两次以上
本来在无apm时加载一次的event方法，在此情景下也可能加载两次。
所以不推荐使用event方式去进行初始化和关闭前执行
总结用法及注意事项
1.初始化加载时
参考上面启动及关闭的流程，可知整个过程中，有可能触发两次的是event事件中的三个SpringApplicationEvent，PayloadApplicationEvent，ApplicationContextEvent，最好不要使用
InitializingBean 和@PostConstruct 最早加载，推荐无动态参数启动需要的情况下，优先使用。
ApplicationRunner和CommandLineRunner在初始化bean及web启动后执行，可以在需要动态传入启动参数的情况下，优先使用。
如果确实非常有需要监听容器的启动关闭等过程，推荐使用明确的子类事件，且要确实明白每个event的加载时机。
最好是尽量不用event方式进行初始化,上面两个方式足够使用了。

2.关闭应用前执行
推荐主要还是以PreDestroy为主
hook方式也可以，但是写代码就多一些。
event close方式不推荐，有重复执行的可能



