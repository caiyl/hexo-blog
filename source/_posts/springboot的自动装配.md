---
title: springboot的自动装配
date: 2023-07-03 09:47:57
tags:
---

**Import**

Import可以导入一个普通的bean

```java
public class PersonService {
    public void sayHello(){
        System.out.println("hello PersonService");
    }
}
```
<!--more-->
Import可以导入ImportSelector

```java
public class WeatherImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{WeatherService.class.getName()};
    }

    @Override
    public Predicate<String> getExclusionFilter() {
        return ImportSelector.super.getExclusionFilter();
    }
}

```

Import可以导入Configuration

```java
package com.third.configuration;

import com.third.service.LoginService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author chase
 * @date 2023/3/6 4:56 PM
 */
@Configuration
public class LoginConfiguration {

    @Bean
    public LoginService getLoginService(){
        return new LoginService();
    }
}

```

ImportSelector

ImportSelector可以导入多个bean

Configuration

Configuration是bean的配置文件

**每个Enablexxx背后都是Import在作怪。spring boot start的几个关键点，注解EnableAutoConfiguration**

```java
@EnableCaching
@EnableRabbit
@EnableScheduling
@EnableApolloConfig
@EnableRedissonLock
```

```java
@Import(CachingConfigurationSelector.class)
public @interface EnableCaching

@Import(RabbitListenerConfigurationSelector.class)
public @interface EnableRabbit

@Import(SchedulingConfiguration.class)
@Documented
public @interface EnableScheduling

@Import({ApolloConfigRegistrar.class})
public @interface EnableApolloConfig

@Import({RedissonDistributedLockAutoConfiguration.class})
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface EnableRedissonLock
```

由此可见，spring的Enablexxx背后都是Import。

**springboot的自动装配**

听起来很高端，实际上也是Enablexxx与Import在起作用

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication 
```

@SpringBootApplication注解是一个组合注解，有EnableAutoConfiguration何SpringBootConfiguration注解。

EnableAutoConfiguration

```java
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration
```

**自动装配也不过是Enablexxx+Import**

AutoConfigurationImportSelector的部分代码如下

```
@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
	
	protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
	
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```

**AutoConfigurationImportSelector里的这三段代码用SpringFactoriesLoader类，加载了配置在类路径里的META-INF/spring.factories类。**

META-INF/spring.factories里就是官方的starter的Configuration配置类。了解了自动装配和Enablexxx，再来手动写一个

starter就信手沾来了。

1. pom依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>2.3.4.RELEASE</version>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <version>2.3.4.RELEASE</version>
            <optional>true</optional>
        </dependency>
    </dependencies>
```



2. 写好配置类

```java
@Configuration
public class HelloConfiguration {

    @Bean
    public HelloService getHelloService(){
        return new HelloService();
    }
}

```



3. 在META-INF/spring.factories 加上

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.hello.configuration.HelloConfiguration
```

3个步骤即可完成一个starter。另外如果不想写META-INF/spring.factories文件，则写一个Enable注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(HelloConfiguration.class)
public @interface EnableHello {
}
```

在spring启动入口加上该注解即可

```java
@EnableHello
@SpringBootApplication
public class BootApp 
```

[代码链接](https://github.com/caiyl/cai.git)


