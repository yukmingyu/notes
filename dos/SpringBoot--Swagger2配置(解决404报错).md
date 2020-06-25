# SpringBoot--Swagger2配置(解决404报错) 

在 spring boot 项目中配置 Swagger2 突然出现了 404 报错, 究其原因,是因为 MVC 没有找到 swagger-ui 包中的 swagger-ui.html 文件; 以下就是 swagger2 的配置,及解决方案:

一, 引入Maven :

```
        <dependency> <!-- API -->
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.7.0</version>
        </dependency>
```

二, 编写配置文件 :

```
package com.gy.fast.common.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import io.swagger.annotations.ApiOperation;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * Swagger2 API配置
 * @author geYang
 * @date 2018-05-11
 */
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        System.out.println("======  SWAGGER CONFIG  ======");
        return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo()).select()
            .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
            .paths(PathSelectors.any())
            .build();
    }
    
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
            .title("Fast 疾速开发  RESTful APIs")
            .description("快速上手,快速开发,快速交接")
            .contact(new Contact("geYang", "https://my.oschina.net/u/3681868/home", "572119197@qq.com"))
            .version("1.0.0")
            .build();
    } 

}
```

到目前已经配置完成, 启动项目访问: http://localhost:${port}/${context-path}/swagger-ui.html

如果访问成功,则不需要配置,如果访问失败, 报错404, 则进行下面配置:

三, 解决404报错:

```
package com.gy.fast.common.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

/**
 * WebMvc配置
 * @author geYang
 * @date 2018-05-14
 */
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
   			registry.addResourceHandler("/statics/**").addResourceLocations("classpath:/statics/");
        // 解决 SWAGGER 404报错
        registry.addResourceHandler("/swagger-ui.html").addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/");
    }

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
       
    }

}
```

其原理就是帮助MVC找到 swagger-ui.html 及其 CSS,JS 对应的文件;

