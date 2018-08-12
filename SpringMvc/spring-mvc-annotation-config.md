# Spring MVC 全注解配置

在古老的项目中，一个 SpringMvc 项目，都是基于 xml 文件来配置。下面记录一下全注解方式配置 SpringMvc`的方式。

其实关键类是 `@EnableWebMvc` ，这个类上的注释/示例，也讲解了大部分细节了。主要两种用法：

## 方式一：@EnableWebMvc + WebMvcConfigurerAdapter/WebMvcConfigurer

`@EnableWebMvc` 注解加在一个被 @Configuration 注解的类上时，会 @Import 一个 `DelegatingWebMvcConfiguration` 类，`DelegatingWebMvcConfiguration` 又继承自 `WebMvcConfigurationSupport`，`WebMvcConfigurationSupport` 就是框架提供的核心的 MVC 配置类了。

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackageClasses = { MyConfiguration.class })
public class MyWebConfiguration {

}
```

WebMvcConfigurationSupport 做了很多事情，详见其注释吧！此处不表。

自定义配置时，一般是实现 WebMvcConfigurer 接口，或继承 WebMvcConfigurerAdapter：

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackageClasses = { MyConfiguration.class })
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

以上自定义配置，都会被 `DelegatingWebMvcConfiguration` 收集，合并进 `WebMvcConfigurationSupport`，源代码逻辑如下：

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
    // more ...
}
```

`DelegatingWebMvcConfiguration` 使用 `WebMvcConfigurerComposite` 通过注入收集所有的 `WebMvcConfigurer` 配置 Bean，最终会应用到 `WebMvcConfigurationSupport`。

## 方式二：extend WebMvcConfigurationSupport/DelegatingWebMvcConfiguration

如果方式一中的 `WebMvcConfigurer` 配置接口没有暴露一些需要配置的项目，则可以**移除** `@EnableWebMvc` 后**直接继承** `WebMvcConfigurationSupport` 或 `DelegatingWebMvcConfiguration`，重写其配置方法：

```java
@Configuration
@ComponentScan(basePackageClasses = { MyConfiguration.class })
public class MyConfiguration extends WebMvcConfigurationSupport {

    @Override
    public void addFormatters(FormatterRegistry formatterRegistry) {
        formatterRegistry.addConverter(new MyConverter());
    }

    @Bean
    public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
        // Create or delegate to "super" to create and
        // customize properties of RequestMappingHandlerAdapter
    }
}
```

一般 `WebMvcConfigurationSupport` 已提供的配置，已经满足了我们大部分的需求，优先使用方式一。