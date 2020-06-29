# PageHelper警告

最近使用PageHelper分页工具的时候遇到如下问题：

```
警告: Hessian/Burlap: 'com.github.pagehelper.Page' is an unknown class in WebappClassLoader
  context: 
  delegate: false
  repositories:
----------> Parent Classloader:
ClassRealm[plugin>org.apache.tomcat.maven:tomcat7-maven-plugin:2.2, parent: sun.misc.Launcher$AppClassLoader@18b4aac2]
:
java.lang.ClassNotFoundException: com.github.pagehelper.Page
```

经过跟踪代码发现，调用服务的时候使用PageHelper插件，使用到其中一个Page类



但是警告不影响使用，导致这个原因是因为序列化和反序列化过程没有找到Page这个类.



**解决方案：**

**只要在对应的业务层加上	”pagehelper“	依赖即可**

```
    <!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>5.1.11</version>
    </dependency>
```



**SpringBoot依赖**

```
    <!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper-spring-	boot-starter -->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-starter</artifactId>
        <version>1.2.13</version>
    </dependency>
```

