
# 八，SpringBoot Web 开发访问静态资源(附\+详细源码剖析)


@

目录* [八，SpringBoot Web 开发访问静态资源(附\+详细源码剖析)](https://github.com)
* [1\. 基本介绍](https://github.com)
* [2\. 快速入门](https://github.com)
	+ [2\.1 准备工作](https://github.com)
* [3\. 改变静态资源访问前缀，定义为我们自己想要的](https://github.com)
* [4\. 改变Spring Boot当中的默认的静态资源路径(实现自定义静态资源路径)](https://github.com)
* [5\. 静态资源访问注意事项和细节](https://github.com)
* [6\. 总结：](https://github.com):[milou加速器](https://xinminxuehui.org)
* [7\. 最后：](https://github.com)



---


# 1\. 基本介绍


SpringBoot 中对于静态资源的访问：


1. 只要将静态资源放在类路径下: /static, /public, /resources, /META\-INF/resources 就可以被直接访问\-对应文件（**这是 Spring Boot 的默认设置好的** ）。关于这一点，我们从 **WebProperties.java** 这个类的源码上可以找到，对应的配置属性。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200821-1365385155.png)



```
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
注意：classpath:/resources/ 表示服务器就会在 resources 路径下找，你在浏览器当中输入的url地址的时候，不可以输入 resources 目录，因为服务器就是会在 classpath:/resources/ 找的，而如果你写了resources在浏览器上的话，你想表达的就是：让浏览器从resources/resouces的路径下找，这是找不到的报404错误

```


> 注意：classpath:/resources/ 表示服务器就会在 resources路径下找，你在浏览器当中输入的url地址的时候，不可以输入 resources 目录，因为服务器就是会在 classpath:/resources/ 找的，而如果你写了resources 在浏览器上的话，你想表达的就是：让浏览器从resources/resouces的路径下找，这是找不到的报404错误


2. 常见静态资源: js,css,图片(.jpg,.png,.gif,.bmp,.svg) ,字体文件(Fonts)等
3. 访问方式： 默认：项目根路径/\+静态资源名 比如: [http://localhost:8080/hi.html](https://github.com) 。关于这一点，我们可以从 WebMvcProperties.java 类当中找到答案。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201548-972192951.png)


# 2\. 快速入门


## 2\.1 准备工作


在 pom .xml 文件中导入相关的 jar 依赖。如下


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200831-1288072184.png)



```
xml version="1.0" encoding="UTF-8"?
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0modelVersion>

    <groupId>com.rainbowseagroupId>
    <artifactId>springboot_static_configurationartifactId>
    <version>1.0-SNAPSHOTversion>


    
    <parent>
        <groupId>org.springframework.bootgroupId>
        <artifactId>spring-boot-starter-parentartifactId>
        <version>2.5.3version>
    parent>

    
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.bootgroupId>
            <artifactId>spring-boot-starter-webartifactId>
        dependency>
    dependencies>

project>

```

编写启动程序：


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200814-99193510.png)



```
package com.rainbowsea.springboot;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  // 标志启动场景
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}


```



---


从上面的**基本介绍** 当中，我们知道了，Spring Boot 默认静态资源的访问路径有**4** 个，我们这里就测试这四个路径是否**可以直接访问** 。



```
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};

```

注意：classpath 表示的是类路径，就是如图下面的: `resources` 目录，简单的说就是 `classpath ===(等同于)resources` 。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200836-413689750.png)


下面：我们分别在 resources 类路径下，创建对应的Spring Boot 默认的四个目录。如下图：
![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200725-1762708677.png)


同时我们在这四个目录下，放入几张图片，用于访问测试。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201593-1552300336.png)


启动程序运行测试：


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201606-1844586652.png)


打开浏览器进行直接访问静态资源文件：


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200809-131415320.png)


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201632-1428376905.png)


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201676-1700685450.png)


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201626-1395739393.png)



> **注意：**
> 
> 
> 
> ```
> private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
> 注意：classpath:/resources/ 表示服务器就会在 resources 路径下找，你在浏览器当中输入的url地址的时候，不可以输入 resources 目录，因为服务器就是会在 classpath:/resources/ 找的，而如果你写了resources在浏览器上的话，你想表达的就是：让浏览器从resources/resouces的路径下找，这是找不到的报404错误
> 
> ```
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201600-1516926622.png)



> resources/static 是Spring Bopt的默认静态路径，默认就是以“resources/static” 作为根路径访问的，所以，不需要再额外的加上 static 在浏览器上，如果你加了，那么你实际访问的是：resources/static/static这个路径，这个路径是不存在资源的，所以报错无法找到。


# 3\. 改变静态资源访问前缀，定义为我们自己想要的


改变静态资源访问前缀，比如我们希望 [http://locahost:8080/rainbowsea/\*](https://github.com) 下的请求路径，去请求静态资源，应用场景：静态资源访问前缀和控制器请求路径冲突。


我们这里需要用到 `yaml` 语法的内容，关于 yaml 语法想要了解的，大家可以移步至：✏️✏️✏️ [七，Spring Boot 当中的 yaml 语法使用\-CSDN博客](https://github.com)


首先，我们在 `resources` 类路径下创建一个名为 `application.yaml` 的文件。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201584-429295014.png)


编写如下内容：


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201619-1827510161.png)



```
spring:
  mvc:
    static-path-pattern: /rainbowsea/**



```

运行测试：


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200827-1235381209.png)



> 被我们改为了我们自己的 `/rainbowsea/**` 注意：后面的 `/**` 不可以省略 。不然无法访问的。
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200848-1468148133.png)




---


# 4\. 改变Spring Boot当中的默认的静态资源路径(实现自定义静态资源路径)


Spring Boot 也是支持我们自定义静态资源路径，提高了灵活性。


改变默认的静态资源路径，比如：我们自己在类路径下增加 test 目录，作为静态资源路径，并完成测试。


同样要想改变 Spring Boot 当中的默认静态资源，这里我们还是使用 yaml 语法进行。在 `resources` 类路径下创建一个名为 `application.yaml` 的文件。


yaml 编写如下：


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201590-1266703238.png)



```
spring:
  web:
    resources:
      # 修改/指定 静态资源的访问路径/位置
      static-locations: ["classpath:/test/"] # 仿写
# 注意：尽量在最左边开始写，才有更多的提示


```

![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200777-1055213912.png)


在 resources 类路径下，创建一个 test目录，同时在该目录下，放入一个名为 `5.jpg` 的图片，访问测试。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200848-315463741.png)


打开浏览器访问测试：


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200832-212252464.png)



> 本质上： static\-locations修改的是 WebProperties类当中staticlocations属性的值(也就是 springboot
> 
> 
> 的默认静态路径)。
> 
> 
> 所以这里我们修改了 Spring Boot 的默认静态资源路径，之前的放置在Spring Boot 默认的静态路径下的资源就无法被访问到了。
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200833-161073490.png)



> 想要：保留原来Spring Boot 的默认静态资源路径，只需要把原来的Spring Boot 默认的路径添加上就可以了。
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200828-1697837372.png)



> 运行测试：
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201644-1571793104.png)


# 5\. 静态资源访问注意事项和细节


1. 注意：直接放在 resources 类的根路径下，是访问不到的。因为我们从 WebProperties 类源码上，就知道了，Spring Boot的默认静态资源路径，就只有四个，而 resources 类路径是不属于这四个当中的。



```
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};

```

运行测试：我们在 resources 类路径下，放入 6\.jpg 图片，试试，可不可以被直接访问。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201538-1624777789.png)


打开浏览器访问测试：


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201597-2105917890.png)


2. 当默认请求的路径的资源的名字和 Controller 控制器请求处理的路径一样，冲突的时候。\*\*优先看Controller能不能处理；不能，处理的请求交给静态资源处理，如果静态资源也找不到则相应点资源，则报：404找不到的，页面。



> 静态资源被访问原理：静态映射是 /\*\* , 也就是对所有请求拦截，请求进来，先看Controller能不能处理，不能处理的请求交给静态资源处理，如果静态资源找不到则相应 404页面
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201623-760419022.png)


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200768-1849260143.png)



```
package com.rainbowsea.springboot.controller;


import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController // @Controller + @ResponseBody
public class HiController {

    @RequestMapping("1.jpg")  // Controller 控制处理的请求的路径和静态资源的名字冲突
    public String hi(){
        return "hi";
    }
}


```

![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200856-1159682169.png)




---


# 6\. 总结：


1. 理解Spring Boot 静态资源放在类路径下: /static, /public, /resources, /META\-INF/resources 就可以被直接访问\-对应文件（**这是 Spring Boot 的默认设置好的** ）。关于这一点，我们从 **WebProperties.java** 这个类的源码上可以找到，对应的配置属性。



```
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
注意：classpath:/resources/ 表示服务器就会在 resources 路径下找，你在浏览器当中输入的url地址的时候，不可以输入 resources 目录，因为服务器就是会在 classpath:/resources/ 找的，而如果你写了resources在浏览器上的话，你想表达的就是：让浏览器从resources/resouces的路径下找，这是找不到的报404错误

```

2. 访问方式： 默认：项目根路径/\+静态资源名 比如: [http://localhost:8080/hi.html](https://github.com) 。关于这一点，我们可以从 WebMvcProperties.java 类当中找到答案。
3. 改变静态资源访问前缀，比如我们希望 [http://locahost:8080/rainbowsea/\*](https://github.com) 下的请求路径，去请求静态资源，应用场景：静态资源访问前缀和控制器请求路径冲突。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201564-755904863.png)


4. 改变Spring Boot当中的默认的静态资源路径（实现自定义静态资源路径）。


![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153200837-1115340706.png)


5. 注意：直接放在 resources 类的根路径下，是访问不到的。因为我们从 WebProperties 类源码上，就知道了，Spring Boot的默认静态资源路径，就只有四个，而 resources 类路径是不属于这四个当中的。


# 7\. 最后：



> “在这个最后的篇章中，我要表达我对每一位读者的感激之情。你们的关注和回复是我创作的动力源泉，我从你们身上吸取了无尽的灵感与勇气。我会将你们的鼓励留在心底，继续在其他的领域奋斗。感谢你们，我们总会在某个时刻再次相遇。”
> 
> 
> ![在这里插入图片描述](https://img2024.cnblogs.com/blog/3084824/202409/3084824-20240908153201725-1509381219.gif)


