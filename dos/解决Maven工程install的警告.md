# 解决Maven工程install的警告

### 1.问题发生

在控制台输出警告：**Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!**

虽然并不影响项目的正常运行和install,但是对于处女座的我来说一点都不想看到这警告的发生。所以就研究了一下，找到解决办法，现在分享给大家。



### 2.解决方法

在maven项目的pom.xml中添加如下配置：

```
    <properties>
    	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
```

**说明：如果我们当前maven项目是别的项目的子项目，只需要在父项目中加入该配置即可，子项目就可以继承了。**