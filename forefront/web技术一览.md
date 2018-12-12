# Web技术一览

## Contents
* 客户端与服务器交互方式
* Web Application
* 模板引擎
* MVC
* Ajax及SPA的出现（后端技术思想在前端的应用）
* 前后端分离探索
* 前端技术梳理


## 客户端与服务器交互方式

### 示意图
![客户端与服务器交互方式](https://github.com/chinaoarq/blog/blob/dev/assets/bc.png?raw=true)  

### 推荐资料：
[《大型网站技术架构》](https://book.douban.com/subject/25723064/) ———— 了解服务器架构演变的科普图书。  

[《图解HTTP》](https://book.douban.com/subject/25863515/) ———— 简单易懂，语言流畅，适合入门阅读。  

[《HTTP权威指南》](https://book.douban.com/subject/10746113/) ———— 进阶版，讲解深入原理，高手之路必读。  
  
  
  
## Web Application

### Web后端常见技术
1. JavaEE家族（Java、servlet&jsp、Spring、MyBatis/Hibernate、Apache、tomcat/JBoss/WebLogic)
2. .Net家族（C#、asp.net、.NET Framework、IIS）
3. LAMP（Linux、Apache、Mysql、PHP）

### Servlet示例
#### HelloWorldServlet
``` Java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;

public class HelloWorld extends HttpServlet {
	private String message;

    public void init() throws ServletException {
        message = "Hello World";
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");

        PrintWriter out = response.getWriter();
        out.println("<h1>" + message + "</h1>");
    }
    
    public void destroy() {
    }
}
```

#### A Real Servlet
``` Java
public class HelloForm extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();
        String title = "使用 GET 方法读取表单数据";
        // 处理中文
        String name =new String(request.getParameter("name").getBytes("ISO8859-1"),"UTF-8");
        String docType = "<!DOCTYPE html> \n";
        out.println(docType +
            "<html>\n" +
            "<head><title>" + title + "</title></head>\n" +
            "<body bgcolor=\"#f0f0f0\">\n" +
            "<h1 align=\"center\">" + title + "</h1>\n" +
            "<ul>\n" +
            "  <li><b>站点名</b>："
            + name + "\n" +
            "  <li><b>网址</b>："
            + request.getParameter("url") + "\n" +
            "</ul>\n" +
            "</body></html>");
    }
}
```

### Servlet缺点
HTML硬编码在java代码中，导致
1. 开发效率低下
2. 容易出bug 
3. 难以维护
4. 要求程序猿必须具备全栈能力



## 模板引擎

### JSP
``` Java
// ProductServlet.java
public class ProductServlet extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response){

        String name = request.getParam("name");
        List<Product> products = ProductService.search(name);

        request.setAttribute("products", products);

        RequestDispatcher rd = getServletContext().getRequestDispatcher("/products.jsp");
        rd.forward(request,response); 
    }
}
```
``` html
<!-- products.jsp --> 
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
    <body>
        <c:forEach items="products" var="prod">
            <div class="product">
                <p>${prod.name}</p>
                <p>${prod.price}</p>
            </div>
        </c:forEach>
    </body>
</html>
```

### Thymeleaf
``` html
<!DOCTYPE html>
<html lang="en">
    <body>
        <div th:each="prod: ${products}" class="product">
            <p th:text="${prod.name}">苹果</p>
            <p th:text="${prod.price}">5.5元/斤</p>
        </div>
    </body>
</html>
```

### Spring template engines
* JMustache
* Thymeleaf
* Groovy
* Pebble
* Apache Velocity
* Apache FreeMarker

参考：[https://www.baeldung.com/spring-template-engines](https://www.baeldung.com/spring-template-engines)


### 推荐学习：
[《Head First Servlets & JSP》](https://book.douban.com/subject/1942934/) ———— java web程序猿必读。  
[《How Tomcat Works》](https://book.douban.com/subject/1943128/) ———— tomcat是servlet容器，了解了servlet是什么之后应该会比较好奇容器的工作原理。  



## MVC
MVC要实现的目标是**将软件用户界面(View)和业务逻辑(Model)分离**以使代码可扩展性、可复用性、可维护性、灵活性加强。  
  
那Controller扮演什么角色呢？在软件开发领域有一句俗话：**“没有什么解耦问题是不能通过加一个抽象层解决的，如果有，那就加两层。”**  
  
哈哈，这不是一句玩笑话，是真的这么回事，MVC再一次证明了这句话的正确性，Controller就是这样一个负责在Model和View中间斡旋的抽象层。  
  
  

### 框架关系图
![MVC](https://github.com/chinaoarq/blog/blob/dev/assets/mvc.png?raw=true)  

从上面的Google结果图可以看出，如今已经找不到一个定式的MVC关系图，从[MVC被提出](http://wiki.jikexueyuan.com/project/javascript-design-patterns/mvc.html)在现在，在不同技术领域，都发展出了[适合各自场景的MVC](https://zh.wikipedia.org/wiki/MVC#%E5%AE%9E%E7%8E%B0)。  

其实MVC更应该称作一种设计思想，而不是设计模式，实现MVC往往需要几种设计模式的组合使用，比如【观察者模式】，当Model被改变时，它会通知观察者(View)一些东西已经被更新了；【策略模式】，当用户操作view时，controller做出不同响应（操作model）。  


### Why MVC?
> 假设需求1：小明想做一个卖衣服的程序，可以在web和手机app上向全世界卖衣服。  
> 假设需求2：web网站主要作为营销渠道，需要根据用户的喜好频繁调整页面样式，达到吸引用户购买的目的。

``` Java
// com.demo.controller.web.WebProductServlet.java
public class WebProductServlet extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response){
 
        String name = request.getParam("name");
        List<Product> products = ProductService.search(name);

        request.setAttribute("products", products);

        RequestDispatcher rd = getServletContext().getRequestDispatcher("/products.jsp");
        rd.forward(request,response); 
    }
}

// com.demo.controller.app.ProductServlet.java
public class AppProductServlet extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response){
 
        String name = request.getParam("name");
        List<Product> products = ProductService.search(name);

        response.setAttribute('products', products.toJsonString());
    }
}

// Model 部分可以复用
// com.demo.model.ProductService.java
public class ProductService {

    public static List<Product> search(name){
        List<Product> products = new ArrayList<Product>();

        /* 
            code for search products by name...
        */

        return products;
    }
}
```

``` html
<!-- products.jsp --> 
<!-- 页面样式可随意改动，不影响业务逻辑代码 --> 
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
	<body>
		<c:forEach items="products" var="prod">
			<div class="product">
				<p>${prod.name}</p>
		   		<p>${prod.price}</p>
			</div>
		</c:forEach>
	</body>
</html>
```


## Ajax及SPA的出现（后端技术思想在前端的应用）

## 前后端分离探索

## 前端技术梳理
