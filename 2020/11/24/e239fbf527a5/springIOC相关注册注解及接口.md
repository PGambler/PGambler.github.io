---
title: springIOC相关注册注解及接口
comments: true
tags:
  - SPRING
  - IOC注解
categories:
  - [后台, JAVA]
  - [学习, JAVA, SPRING]
date: 2020-11-24 20:12:11
updated: 2020-11-24 20:12:11
keywords: [SPRING, IOC注解]
---
IOC作为spring的核心，有哪些相关的注解以及接口？
<!-- more -->
借助网上的一篇文章特此做一份自己记录

> [深入学习Spring组件注册](https://mrbird.cc/Spring-Bean-Regist.html)

1. @Bean
这是一个方法级别上的注解，主要用在@Configuration注解的类里，也可以在@component注解的类里
```
@Configuration
public class AppConfig {
    @Bean
    public Demo demo() {
        return new Demo();
    }
}
```
2. @ComponentScan
这是一个类级别上的注解，配合@Configuration注解一起使用。从定义扫描的路径（未指定即为当前注解类所在包）中找出标注@Controller、@Service、@Repository、@Component的类装配到spring容器中
```
@Configuration
@ComponentScan("com.demo")
public class AppConfig {
}
```
excludeFilters来排除一些组件的扫描：
```
@Configuration
@ComponentScan(value="com.demo", excludeFilters = {
  @Filter(type = FilterType.ANNOTATION,classes = {Controller.class, Repository.class}),
   @Filter(type = FilterType.ASSIGNABLE_TYPE,classes = {Demo1.class, Demo2.class})
})
public class AppConfig {
}
```
includeFilters的作用和excludeFilters相反，其指定的是哪些组件需要被扫描：
```
@Configuration
@ComponentScan(value="com.demo", includeFilters = {
  @Filter(type = FilterType.ANNOTATION, classes = Service.class)},
  useDefaultFilters = false)
})
public class AppConfig {
}
```
3. @Scope
组件的作用域，类与方法级别都可使用该注解。默认情况下IOC容器里面的组件均是单例（容器创建过程注入），可通过@Scope注解进行组件作用域配置：
1） singleton：单实例（默认）,在Spring IOC容器启动的时候会调用方法创建对象然后纳入到IOC容器中，以后每次获取都是直接从IOC容器中获取（map.get()）；
2）prototype：多实例，IOC容器启动的时候并不会去创建对象，而是在每次获取的时候才会去调用方法创建对象；
3）request：一个请求对应一个实例；
4）session：同一个会话对应一个实例；
```
@Configuration
public class WebConfig {
    @Bean
    @Scope("prototype")
    public User user() {
        return new User("mrbird", 18);
    }
}
```
4. @Lazy
懒加载是针对单例模式而言的，正如前面所说，IOC容器中的u组件默认是单例的，容器启动的时候会调用方法创建对象然后注入到IOC容器中。而当使用懒加载模式时，对象将不在容器启动过程中马上创建（除非使用），而是等到对象首次被调用时进行实例化注入容器并返回给调用方。
```
@Configuration
public class WebConfig {
    @Bean
    @Lazy
    public User user() {
        return new User("mrbird", 18);
    }
}
```
5. Conditional
条件注册注解，类与方法级别的注解，该注解可以指定组件注册的条件，当组件满足特定的条件后才注入IOC容器。该注解接受一个实现Condition接口的类型：
```
class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String osName = context.getEnvironment().getProperty("os.name");
        return osName != null && osName.contains("Windows");
    }
}

@Configuration
class Demo {
    @Bean
    @Conditional(MyCondition.class)
    public User user() {
        return new User("mrbird", 18);
    }
}
```
6. Profile
@Profile可以根据不同的环境变量来注册不同的组件
> [Spring 中的@Profile 注解的作用是什么？](https://cloud.tencent.com/developer/article/1362768)
```
@Configuration
class AppConfig {
    @Bean("demo")
    @Profile("java7")
    public Demo demo1() {
        return new Demo("mrbird", "java7");
    }
    @Bean("demo")
    @Profile("java8")
    public Demo demo2() {
        return new Demo("mrbird", "java8");
    }
}
```
7. @Import
到目前为止，我们可以使用包扫描和@Bean来实现组件注册。除此之外，我们还可以使用@Import来快速地往IOC容器中添加组件。
```
class Hello {
}
@Configuration
@Import({Hello.class})
public class AppConfig {
}
```
8. ImportSelector
通过@Import我们已经实现了组件的导入，如果需要一次性导入较多组件，我们可以使用ImportSelector来实现。
```
class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{
                "cc.mrbird.demo.domain.Apple",
                "cc.mrbird.demo.domain.Banana",
                "cc.mrbird.demo.domain.Watermelon"
        };
    }
}

@Import({MyImportSelector.class})
public class WebConfig {
    ...
}
```
9. ImportBeanDefinitionRegistrar
除了上面两种往IOC容器导入组件的方法外，我们还可以使用ImportBeanDefinitionRegistrar来手动往IOC容器导入组件。
```
class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        final String beanName = "strawberry";
        boolean contain = registry.containsBeanDefinition(beanName);
        if (!contain) {
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Strawberry.class);
            registry.registerBeanDefinition(beanName, rootBeanDefinition);
        }
    }
}

@Configuration
@Import({MyImportBeanDefinitionRegistrar.class})
public class WebConfig {
    ...
}
```
10. FactoryBean
Spring还提供了一个FactoryBean接口，我们可以通过实现该接口来注册组件，该接口包含了两个抽象方法和一个默认方法：
```
class CherryFactoryBean implements FactoryBean<Cherry> {
    @Override
    public Cherry getObject() {
        return new Cherry();
    }

    @Override
    public Class<?> getObjectType() {
        return Cherry.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}

@Bean
public CherryFactoryBean cherryFactoryBean() {
    return new CherryFactoryBean();
}
```
以上有些是个人观点，如发现有误还望指出

（完）