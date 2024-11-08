---
title: "Springboot自定义注解注入参数"
date: 2020-05-12T20:41:04+08:00
draft: false
tags: ["java","springboot"]
categories: ["java"]
typora-root-url: ./
typora-copy-images-to: ./
---

> 使用spring security时经常会使用注解`@AuthenticationPrincipal`获取用户信息。在想要注入用户信息的Controller接口上加上注解，即可方便的获取访问用户的信息。而且Controller还“非常的智能”，比如接口上写上参数`HttpServletRequest request`，就能自动注入http请求信息

## 需求

现在有一种需求（案例简化的需求），很多接口都需要查询用户的部门`Department`。每次可能都有类似如下硬编码

```java
@GetMapping("/")
public String getSomeThing(@AuthenticationPrincipal Account account) {
    Department department = departmentManager.findOne(account.getDepartmentId());
    // doSomeThing with department ...
}
```

这样也不是不可以，但是总感觉不够优雅，特别是有时业务不像案例这么简单一个查询就OK，可能有一大段逻辑要处理才查询出部门信息。

## 解决

spring自身有如此智能的参数注入机制，猜想这个优秀的框架肯定为我们留下了可扩展的方法，让我们可以实现类似的功能。经过查找资料，找到了自定义的办法

### 自定义注解

我们写一个自定义注解`@GetDepartment`，稍后为所有有该注解的Controller接口方法参数上注入部门信息

```java
@Documented
@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface GetDepartment {
}
```



### 自定义http方法参数解析器

实现自定义解析器实现`HandlerMethodArgumentResolver`。这里因为需要用到`@Autowired`注入一个部门查询的Bean，所以加了`@Component`注解。其实不需要注入时不加`@Component`注解也是可以的。推荐加注解交给Spring管理。

```java
@Component
public class GetDepartmentHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Autowired
    private DepartmentManager departmentManager;

    private static final Class<?> commandClass = Department.class;


    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        // 过滤出符合条件的参数，这里指的是加了GetDepartment注解的参数。
        // 其他不符合条件的参数不会使用该解析器处理
        return parameter.hasParameterAnnotation(GetDepartment.class);
    }


    
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) {
        // 该方法解析器处理逻辑
        Class<?> klass = parameter.getParameterType();
        // 定义部门信息必须用Department类接收，不允许用其他的class接收
        if (!klass.isAssignableFrom(commandClass)) {
            throw new RuntimeException("部门获取失败");
        }
        // 获取用户信息
        CurrentUserDetails account = (CurrentUserDetails) mavContainer.getDefaultModel().get("currentAccount");
        // 查询部门
        Department department = departmentManager.findOne(account.getDepartmentId());
        return department;
    }
}
```

需要重写两个方法:

1. `supportsParameter`方法主要过滤出需要注入部门的参数，放行不满足条件的参数
2. `resolveArgument`方法是真正做自己业务逻辑，比如这里获取部门信息，并返回给框架，框架会帮我们注入到方法上

`resolveArgument`方法自带4个参数，经过debug，发现这从这4个参数中，我们可以获取大部分想要的信息，去继续下面的业务逻辑。

1. `ModelAndViewContainer`参数中带有用户的登录信息，即和使用`@AuthenticationPrincipal`注解时获取的用户信息一样。

2. `MethodParameter`是本次http请求的接口方法信息。

3. `NativeWebRequest`里有HttpRequest一些信息，比如可以从中获取cookie，获取方法参数等。

4. `WebDataBinderFactory`暂时没用上

### 把自定义解析器添加到spring管理

仅仅把解析器生成Spring Bean是不够的，还需要把该对象添加到特定的解析器队列中。需要自定义一个类继承`WebMvcConfigurerAdapter`，重写`addArgumentResolvers`方法，把刚才的Bean添加进去队列。

```java
@Configuration
public class CustomWebMvcConfigurerAdapter extends WebMvcConfigurerAdapter {
    @Autowired
    private GetDepartmentHandlerMethodArgumentResolver getDepartmentHandlerMethodArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(getDepartmentHandlerMethodArgumentResolver);
    }
}
```

### 使用

上面自定义两个类，一个注解后，配置已经完成。在使用处使用`@GetDepartment Department department`接收即可，这样部门信息就自动注入进来了

```java
@GetMapping("/")
public String getSomeThing(@AuthenticationPrincipal Account account,@GetDepartment Department department) {
    // doSomeThing with department ...
}
```

## 踩坑

在配置添加解析器到Spring中，最开始是自定义类继承`WebMvcConfigurationSupport`而不是继承`WebMvcConfigurerAdapter`，其实同样可以做到自定义注入参数的功能。但是后来发现自定义实现`WebMvcConfigurationSupport`后，原本使用`@AuthenticationPrincipal`注解获取用户信息的地方，现在都无法获取了。

```java
@Configuration
public class CustomWebMvcConfigurerAdapter extends WebMvcConfigurationSupport {
    @Autowired
    private GetDepartmentHandlerMethodArgumentResolver getDepartmentHandlerMethodArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(getDepartmentHandlerMethodArgumentResolver);
    }
}
```



经过查看源码发现Spring提供了`DelegatingWebMvcConfiguration`去实现`WebMvcConfigurationSupport`。我们称代理类

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

    private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();


    @Autowired(required = false)
    public void setConfigurers(List<WebMvcConfigurer> configurers) {
        if (!CollectionUtils.isEmpty(configurers)) {
            this.configurers.addWebMvcConfigurers(configurers);
        }
    }

    @Override
    protected void configurePathMatch(PathMatchConfigurer configurer) {
        this.configurers.configurePathMatch(configurer);
    }
    // ...
}
```



代理类中有个`WebMvcConfigurerComposite`对象，通过set注入mvc配置类`List<WebMvcConfigurer>`，并add进去。我们查看`WebMvcConfigurerComposite`类，下面称之为聚合类。

```java
class WebMvcConfigurerComposite implements WebMvcConfigurer {

    private final List<WebMvcConfigurer> delegates = new ArrayList<WebMvcConfigurer>();


    public void addWebMvcConfigurers(List<WebMvcConfigurer> configurers) {
        if (!CollectionUtils.isEmpty(configurers)) {
            this.delegates.addAll(configurers);
        }
    }

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        for (WebMvcConfigurer delegate : this.delegates) {
            delegate.configurePathMatch(configurer);
        }
    }
	// ...
}

```

我们发现对代理类的方法调用，都会转为对聚合类中相同方法的调用，而聚合类会把配置类集合`List<WebMvcConfigurer>`循环，一一对每个`WebMvcConfigurer`调用相同的方法。比如上面的`configurePathMatch`方法

现在找寻代理类的使用地方，找到了`WebMvcAutoConfiguration`，可以看出这个类是SpringMvc缺省的自动配置类，而这个类有个很重要的一行`@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)`。意思只有不存在`WebMvcConfigurationSupport`bean实例的时候，才会初始化下面这些bean，包括spring mvc 缺省一个自动配置`WebMvcAutoConfigurationAdapter`，还有实现了刚才代理类的子类`EnableWebMvcConfiguration`和下面其他的一些配置。这些统统不生效，实现了`WebMvcConfigurationSupport`的bean只能存在一个

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,WebMvcConfigurerAdapter.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

    @Configuration
    @Import({ EnableWebMvcConfiguration.class, MvcValidatorRegistrar.class })
    @EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
    public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter {
        // ...
    }

    @Configuration
    public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration	implements InitializingBean {
        // ...
    }
    // ...
}
```

### 总结

`WebMvcConfigurationSupport`注释如下

> ```java
> /**
>  * This is the main class providing the configuration behind the MVC Java config...
>  */
> ```

`WebMvcConfigurationSupport`是spring mvc主要配置类，所以一开始我们自定义实现`WebMvcConfigurationSupport`时，原先的spring mvc一些缺省配置都不会自动配置，也不会配置代理类，自然不会有 `调用代理类->调用聚合类->聚合类调用每一个配置类`这样一个调用链条，相当于所有的`WebMvcConfigurer`实现类都不生效。相关的mvc一些缺省配置自然不生效。

## 参考

[https://www.jianshu.com/p/c5c1503f5367](https://www.jianshu.com/p/c5c1503f5367)