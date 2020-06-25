# SpringBoot支持AJAX跨域请求

> 利用注解的方式解决AJAX请求跨域问题



1.编写一个支持跨域请求的 Configuration

## - 第一种方式

### - CorsConfig.java

```
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

/**
 * 处理AJAX请求跨域的问题
 * @author Levin
 * @time 2017-07-13
 */
@Configuration
public class CorsConfig extends WebMvcConfigurerAdapter {
	static final String ORIGINS[] = new String[] { "GET", "POST", "PUT", "DELETE" };
	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/**").allowedOrigins("*").allowCredentials(true).allowedMethods(ORIGINS)
				.maxAge(3600);
	}
}
```

2.HTTP请求接口

### - HelloController.java

```
@RestController
public class HelloController {
	
	@Autowired
	HelloService helloService;
	
	@GetMapping(value = "/test", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
	public String query() {
		return "hello";
	}
}
```

## - 第二种方式（推荐）

PS：第一种存在一个问题，当服务器抛出 500 的时候依旧存在跨域问题

```
@SpringBootApplication
@ComponentScan
@EnableDiscoveryClient
public class ManagementApplication {

    public static void main(String[] args) {
        SpringApplication.run(ManagementApplication.class, args);
    }

    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addExposedHeader(HttpHeaderConStant.X_TOTAL_COUNT);
        return corsConfiguration;
    }

    /**
     * 跨域过滤器
     *
     * @return
     */
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig()); // 4
        return new CorsFilter(source);
    }
}
```

## - index.html

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>跨域请求</title>
<script src="https://cdn.bootcss.com/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
	$("button").click(function(){
		$.ajax({url:"http://localhost:8080/test",success:function(result){
			$("#p1").html(result);
		}});
	});
});
</script>
</head>
<body>

<p width="500px" height="100px" id="p1"></p>
<button>获取其他内容</button>
</body>
</html>
```

