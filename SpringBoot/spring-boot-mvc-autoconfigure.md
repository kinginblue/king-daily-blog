# SpringBoot Mvc 自动配置

## SpringBoot 1.5.7 源码解析

主要起作用的是 `spring-boot-autoconfigure` 包下的 `WebMvcAutoConfiguration` 自动配置类。自动配置逻辑源码摘要如下：

```java
/**
 * {@link EnableAutoConfiguration Auto-configuration} for {@link EnableWebMvc Web MVC}.
 *
 * @author Phillip Webb
 * @author Dave Syer
 * @author Andy Wilkinson
 * @author Sébastien Deleuze
 * @author Eddú Meléndez
 * @author Stephane Nicoll
 */
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurerAdapter.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

    // Defined as a nested config to ensure WebMvcConfigurerAdapter is not read when not
    // on the classpath
    @Configuration
    @Import(EnableWebMvcConfiguration.class)
    @EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
    public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter {
        // more ...
    }

    /**
     * Configuration equivalent to {@code @EnableWebMvc}.
    */
    @Configuration
    public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
        // more ...
    }

}
```

和 [Spring MVC 全注解配置](../SpringMvc/spring-mvc-annotation-config.md) 类似，主要就是引入 `WebMvcConfigurationSupport/DelegatingWebMvcConfiguration` 并继承 `WebMvcConfigurerAdapter` 进行一些自定义配置：

- `WebMvcAutoConfigurationAdapter` 继承自 `WebMvcConfigurerAdapter` 做了一些自定义配置。
- `EnableWebMvcConfiguration` 继承自 `DelegatingWebMvcConfiguration`，作用与 `@EnableWebMvc` 相当。

所以在 SpringBoot 项目中，引入 `spring-boot-starter-web` 就可以获得一个自动配置好的 Web 环境，既不需要显式加 `@EnableWebMvc` 注解开启，也不需要继承 `WebMvcConfigurationSupport/DelegatingWebMvcConfiguration` 配置类。

如果要做一些自定义配置，继承 `WebMvcConfigurerAdapter` 并重写相关方法即可：

```java
@Configuration
public class MyConfiguration extends WebMvcConfigurerAdapter {

    @Override
    public void addFormatters(FormatterRegistry formatterRegistry) {
        formatterRegistry.addConverter(new MyConverter());
    }

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new MyHttpMessageConverter());
    }

    // More overridden methods ...
}
```

## MORE

- [Spring MVC 全注解配置](../SpringMvc/spring-mvc-annotation-config.md)