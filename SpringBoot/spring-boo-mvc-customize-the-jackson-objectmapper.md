# SpringBoot MVC 定制 Jackson ObjectMapper

## 一|文档

Spring MVC 使用 `HttpMessageConverters` 来做 Http 请求的数据转换。如果 `Jackson` 在项目路径下，SpringBoot 会自动配置一个 `Jackson2ObjectMapperBuilder`，它提供一个默认的转换器（即 `ObjectMapper`）。

默认配置的 `ObjectMapper`（或 `XmlMapper`）配置了以下定制属性：

- 禁用 MapperFeature.DEFAULT_VIEW_INCLUSION
- 禁用 DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES
- 禁用 SerializationFeature.WRITE_DATES_AS_TIMESTAMPS

可以使用环境变量来配置 `ObjectMapper/XmlMapper`，`Jackson` 提供了一系列可用于配置其处理各个方面的开关特性，如下：

| Enum | Property | Values |
|:-|:-|:-|
| com.fasterxml.jackson.databind.DeserializationFeature | spring.jackson.deserialization.<feature_name> | true, false |
| com.fasterxml.jackson.core.JsonGenerator.Feature | spring.jackson.generator.<feature_name> | true, false |
| com.fasterxml.jackson.databind.MapperFeature | spring.jackson.mapper.<feature_name> | true, false |
| com.fasterxml.jackson.core.JsonParser.Feature | spring.jackson.parser.<feature_name> | true, false |
| com.fasterxml.jackson.databind.SerializationFeature | spring.jackson.serialization.<feature_name> | true, false |
| com.fasterxml.jackson.annotation.JsonInclude.Include | spring.jackson.default-property-inclusion | always, non_null, non_absent, non_default, non_empty |

更多详见 [JacksonProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jackson/JacksonProperties.java) 。

环境配置会被应用到自动配置的 `Jackson2ObjectMapperBuilder` Bean 中，进而应用到由其配置的 mappers，包括 `ObjectMapper` Bean。

被一个或多个 `Jackson2ObjectMapperBuilderCustomizer` Bean 定制，可以被按顺序（实现 Order 接口/ 加 @Order 注解）应用到 `Jackson2ObjectMapperBuilder` Bean 进行定制；SpringBoot 默认提供了一个顺序为0的 `StandardJackson2ObjectMapperBuilderCustomizer`。

任何注册进自动配置的 `Jackson2ObjectMapperBuilder` 的 `com.fasterxml.jackson.databind.Module` Bean 都会被应用到其创建的 ObjectMapper 实例上。

如果你想彻底替换默认的 `ObjectMapper`：

- 要么提供一个 `ObjectMapper` Bean，并为其加上 @Primary。
- 要么提供一个  `Jackson2ObjectMapperBuilder` Bean。
- 以上两种方式，都会禁用掉自动配置的 `ObjectMapper`。

手动提供 `MappingJackson2HttpMessageConverter` 类型的 Bean 都会替换掉自动配置的 `MappingJackson2HttpMessageConverter` Bean。

## 二|应用

我们日常开发中，当然有定制 `MappingJackson2HttpMessageConverter` / `ObjectMapper` 的需求啊，SpringBoot 自动配置的 `ObjectMapper` 还是有些不尽人意的地方：

- 序列化一个值为 null 的属性时，该属性也会参与序列化并被序列化成：`"key": null`，这样对前端很不友好，语义也不明确。
- 序列化一个空对象（如：`new Object()`）时，会抛出运行时异常。
- 反序列化时，遇到未知属性，会抛出运行时异常。
- 序列化/反序列化时，我们一般都需要对String类型的值做去掉前后无用空格的操作。

结合以上的文档可以看出，提供一个 `Jackson2ObjectMapperBuilderCustomizer` Bean 是对默认配置影响最小的定制方式，定制以上需求的配置代码如下：

```java
@Configuration
public class LocalJackson2ObjectMapperConfiguration {

    /**
     * 定制 SpringContext 中的 ObjectMapper
     */
    @Bean
    Jackson2ObjectMapperBuilderCustomizer localJackson2ObjectMapperBuilderCustomizer () {
        return new Jackson2ObjectMapperBuilderCustomizer() {
            @Override
            public void customize(Jackson2ObjectMapperBuilder jacksonObjectMapperBuilder) {
                jacksonObjectMapperBuilder
                        .serializationInclusion(JsonInclude.Include.NON_NULL)
                        .failOnEmptyBeans(false)
                        .failOnUnknownProperties(false)
                        .modules(new StringTrimModule());
            }
        };
    }

    /**
     * 序列化/反序列化时，对 String 类型参数，执行 trim() 操作
     */
    public static class StringTrimModule extends SimpleModule {
        public StringTrimModule() {
            addSerializer(String.class, new StdScalarSerializer<String>(String.class) {
                @Override
                public void serialize(String value, JsonGenerator gen, SerializerProvider provider) throws IOException {
                    gen.writeString(value.trim());
                }
            });
            addDeserializer(String.class, new StdScalarDeserializer<String>(String.class) {
                @Override
                public String deserialize(JsonParser jsonParser, DeserializationContext ctx) throws IOException, JsonProcessingException {
                    return jsonParser.getValueAsString().trim();
                }
            });
        }
    }
}
```

## 三|源码

源码没啥好讲的，主要是 `JacksonAutoConfiguration` 这个自动配置类起作用，详见：[JacksonAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration.java)。

## 四|参考

- [Customize the Jackson ObjectMapper](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-spring-mvc.html#howto-customize-the-jackson-objectmapper)
- [JacksonAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration.java)