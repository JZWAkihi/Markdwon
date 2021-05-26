

## 1、Cookie

### 1.1、Cookie概述

我们打开163邮箱官方。https://mail.163.com/

![image-20210517104225217](https://gitee.com/Akihij/PicGo/raw/master/img/20210517104232.png)



输入账号和密码，并且点击**十天内免登录**。进入邮箱。

当我们关闭浏览器或关闭电脑之后，我们再次进入邮箱。https://mail.163.com/

此时我们不需要输入用户名和密码。说明：我们的账号和密码是保存在客户端，并且是在客户端的硬盘上，而不是内存中。

客户端电脑中用于保存这些会话状态的资源，称为Cookie。



> Cookie是1993由网景公司（Netscape）前雇员发明的一种进行**网络会话状态跟踪的技术**。



`会话是由一组请求与响应组成`，是围绕着一件相关的事情所进行的请求与响应。所以在请求与响应之间需要有数据传递的。即是需要进行会话状态跟踪的。

但是**HTTP是无状态的协议**，所谓的无状态，是指：请求之间无法进行数据的传递的。也就是说，HTTP是无法直到上一次的请求是什么。

因此cookie就是这种进行请求间的数据传递会话跟踪技术。



Cookie是由服务器生成，保存在客户端的一种信息载体。这个载体中存放着用户访问该站点的会话状态信息。只要cookie没有被清空或者失效，那么，保存在其中的会话状态就有效。



用户在第一次请求后，有服务器生成cookie，并将其封装到响应头中，以响应的形式发送给客户端。客户端接受到这个响应后，将Cookie保存在客户端。当客户端再次发送同类请求后，在请求中会携带保存在客户端的Cookie数据，发送到服务端，由服务端对会话进行跟踪。





### 1.2、服务端生成Cooike

#### cookie的生成和获取

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                  http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">


    <servlet>
        <servlet-name>Demo1</servlet-name>
        <servlet-class>com.jiang.SomeServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>Demo1</servlet-name>
        <url-pattern>/demo1</url-pattern>
    </servlet-mapping>

</web-app >
```



```java
public class SomeServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //创建两个cookie
        Cookie cookie1 = new Cookie("name","JiangZW");
        Cookie cookie2 = new Cookie("age","20");

        //向响应中添加cookie
        resp.addCookie(cookie1);
        resp.addCookie(cookie2);
    }
}
```



我们可以看到在响应头中，有从服务端返回的Cookie

![image-20210518182351596](https://gitee.com/Akihij/PicGo/raw/master/img/20210518182351.png)



我们说过，cookie是用来请求之间的数据传递的。

如果我们发送同类请求，Cookie是否会发送？



```java
public class OtherServlet  extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        }

    }
}
```

```xml
<servlet>
    <servlet-name>Demo2</servlet-name>
    <servlet-class>com.jiang.OtherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>Demo2</servlet-name>
    <url-pattern>/demo2</url-pattern>
</servlet-mapping>
```



我们可以看到，我们访问localhost:8080/demo2，请求头上cookie携带上一次请求的数据。并将它传递给服务端。

![image-20210518182820192](https://gitee.com/Akihij/PicGo/raw/master/img/20210518182820.png)





我们可以在服务端接受前端传来的cookie数据

```java
public class OtherServlet  extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie[] cookies = req.getCookies();

        for (Cookie cookie:cookies) {
            System.out.println(cookie.getName() + "   " + cookie.getValue() + "    " + cookie.getPath());
        }

    }
}
```



![image-20210518183201275](https://gitee.com/Akihij/PicGo/raw/master/img/20210518183201.png)



#### 设置有效期

在默认的情况下，Cookie是保存在浏览器的缓存中，浏览器关闭，缓存消失，Cookie消失。通过Cookie设置有效时长，可以将Cookie写入客户端硬盘文件中。

```java
public class SomeServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //创建两个cookie
        Cookie cookie1 = new Cookie("name","JiangZW");
        Cookie cookie2 = new Cookie("age","20");

        //设置有效期
        cookie1.setMaxAge(100);
        cookie2.setMaxAge(100);
        //向响应中添加cookie
        resp.addCookie(cookie1);
        resp.addCookie(cookie2);

    }
}
```

#### 绑定路径

```java
public class SomeServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //创建两个cookie
        Cookie cookie1 = new Cookie("name","JiangZW");
        Cookie cookie2 = new Cookie("age","20");

        //指定cookie绑定路径
        cookie1.setPath(req.getContextPath() + "/xxx/demo1");
        cookie2.setPath(req.getContextPath() + "/xxx/demo2");
        //向响应中添加cookie
        resp.addCookie(cookie1);
        resp.addCookie(cookie2);

    }
}
```



### 1.3、Cookie禁用

浏览器是可以禁用 Cookie 的。所谓禁用 Cookie 是指客户端浏览器不接收服务器发送来

的 Cookie。不过，现在的很多网站，若浏览器禁用了 Cookie，则将无法访问。例如，163邮

箱就要求浏览器不能禁用 Cookie。



![image-20210518184140887](https://gitee.com/Akihij/PicGo/raw/master/img/20210518184140.png)







## 2、Session

Session，即会话，是 **Web 开发中的一种会话状态跟踪技术**。前面所说的 Cookie也是一种会话跟踪技术。**不同的是 Cookie 是将会话状态保存在了客户端而， Session 则是将会话状态保存在了服务器端**。



什么是会话：

对于用户来说：当用户打开浏览器，从发出第一次请求开始，一直到最终关闭浏览器，就表示一次会话的完成。

### 2.1、Session的使用



若要对 Session 进行操作，则可以通过 HttpServletRequest 的 getSession()方法获取。该方法具有两个重载的方法。

- public HttpSession getSession(boolean create)

  用于创建 Session。若参数 create 为 true，则表示若当前没有 Session，则新建一

  个 Session，若当前存在 Session 则使用当前的 Session。若参数 create 为 false 表示若当前没

  有 Session，则直接返回 null。 

- public HttpSession getSession()

  该方法用于创建 Session。相当于 getSession(true)，即没有 Session 则创建新的 Session。

  

#### 对Session域属性空间操作

Session 是一个专门用于存放数据的集合，我们一般称这个用于存放数据的内存空间为域属性空间，简称域。HttpSession 中具有三个方法，是专门用于对该域属性空间中数据进行写、读操作的。



- public void setAttribute(String name, Object value)

  该方法用于向 Session 的域属性空间中放入指定名称、指定值的域属性。

- public Object getAttribute(String name)

  该方法用于从 Session 的域属性空间中读取指定名称为域属性值。

- public void removeAttribute(String name)

  该方法用于从 Session 的域属性空间中删除指定名称的域属性。



```java
public class SomeServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //创建session
        HttpSession session = req.getSession();

        session.setAttribute("name","jzw");
    }
}
```



```java
public class OtherServlet  extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        HttpSession session = req.getSession();

        String name = (String)session.getAttribute("name");

        System.out.println("name ====== " + name);

    }
}
```

![image-20210518185341233](https://gitee.com/Akihij/PicGo/raw/master/img/20210518185341.png)





可以通过removeAttribute移除

```java
session.removeAttribute("name");
```

![image-20210518185845701](https://gitee.com/Akihij/PicGo/raw/master/img/20210518185845.png)





### 2.2、Session原理



多个用户均在自己的电脑上访问当前应用，发现不同用户从 Session 中读取到的都是自己所提交的参数值，并没有读取到别人的参数，并没有发生“错乱”现象。这是为什么呢？



Web开发中的Session机制，为每个用户都分配了一个Session。即一个用户一个Session，确切地说，是一次会话一个 Session 对象。同一用户可以发出多个会话，即会产生多个 Session。



在服务器中系统会为每个会话维护一个Session。不同的会话，对应不同的Session。

不同的会话，对应不同的 Session。那么，系统是如何识别各个 Session 对象的？即是如何做到在同一会话过程中，一直使用的是同一个 Session 对象呢？

###  Session的工作流程

#### 生成Session列表

服务器对当前应用中的 Session 是以 Map 的形式进行管理的，这个 Map 称为 Session 列 表。该 Map 的 key 为一个 32 位长度的随机串，这个随机串称为 JSessionID，value 则为 Session对象的引用。



当用户第一次提交请求时，服务端 Servlet 中执行到 request.getSession()方法后，会自动生成一个 Map.Entry 对象，key 为一个根据某种算法新生成的 JSessionID，value 则为新创建的 HttpSession 对象。

![image-20210518190522624](https://gitee.com/Akihij/PicGo/raw/master/img/20210518190522.png)



#### 服务器生成并发送Cookie

在将 Session 信息写入 Session 列表后，系统还会自动将“JSESSIONID”作为 name，这 个 32 位长度的随机串作为 value，以 Cookie 的形式存放到响应报头中，并随着响应，将该Cookie 发送到客户端。

![image-20210518191001035](https://gitee.com/Akihij/PicGo/raw/master/img/20210518191001.png)



#### 客户端接收并发送Cookie

客户端接收到这个 Cookie 后会将其存放到浏览器的缓存中。即，只要客户端浏览器不关闭，浏览器缓存中的 Cookie 就不会消失。当用户提交第二次请求时，会将缓存中的这个 Cookie，伴随着请求的头部信息，一块发送到服务端。

![image-20210518191059423](https://gitee.com/Akihij/PicGo/raw/master/img/20210518191059.png)



#### 从 **Session** 列表中查找

服务端从请求中读取到客户端发送来的 Cookie，并根据 Cookie 的 JSSESSIONID 的值，从

Map 中查找相应 key 所对应的 value，即 Session 对象。然后，对该 Session 对象的域属性进

行读写操作。



### 2.3、Session失效

Web 开发中引入的 Session 超时的概念，Session 的失效就是指 Session 的超时。若某个Session 在指定的时间范围内一直未被访问，那么 Session 将超时，即将失效。



在 web.xml 中可以通过<session-config/>标签设置 Session 的超时时间，单位为分钟。默认 Session 的超时时间为 30 分钟。需要再次强调的是，这个时间并不是从 Session 被创建开始计时的生命周期时长，而是从最后一次被访问开始计时，在指定的时长内一直未被访问的时长。



```xml
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```





我们也可以通过代码失效：

```java
session.invalidate();
```

![image-20210518191411020](https://gitee.com/Akihij/PicGo/raw/master/img/20210518191411.png)



### 2.4、禁用cookie后的session

当我们在客户端禁用cookie之后，在服务端不会受到影响。依旧会创建cookie和session。并通过响应返回给客户端。

![image-20210518192708061](https://gitee.com/Akihij/PicGo/raw/master/img/20210518192708.png)

我们不关闭浏览器，再次访问localhost:8080/demo1。

发现出现的JSESSIONID不是原来的值了。

![image-20210518192835817](https://gitee.com/Akihij/PicGo/raw/master/img/20210518192835.png)



我们没有关闭浏览器，这是一次会话，但出现的session不同。



若客户端浏览器禁用了 Cookie，会发现向服务器所提交的每一次请求，服务器在给出的响应中都会包含名称为 JSESSIONID 的 Cookie，只不过这个 Cookie 的值每一次都不同。也就是说，只要客户端浏览器所提交的请求中没有包含 JSESSIONID，服务器就会认为这是一次新的会话的开始，就会为其生成一个 Map.Entry 对象，key 为新的 32 位长度的随机串，value为新创建的 Session 会话引用。这样的话，也就无法实现会话跟踪了。







