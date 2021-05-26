<h2 >Servlet 编程基础</h2>

Servlet 是连接 Web 服务器与服务端 Java 程序的协议，是一种通信规范。这个规范是以一套接口的形式体现的。

> Servlet 规范中包含一套接口。而 Servlet 接口仅仅是其中之一。

![image-20210526101344544](https://gitee.com/Akihij/PicGo/raw/master/img/20210526101351.png)



微观地讲，Servlet 是 Servlet 接口实现类的一个实例对象，是运行在服务器上的一段 Java小程序，即 Server Applet，也就是 Servlet 这个单词的来历。**Servlet 的主要功能是根据客户端提交的请求，调用服务器端相关 Java 代码，完成对请求的处理与运算。**



#### 1.1、Servlet生命周期

Servlt 生命周期是指，Servlet 对象的创建、Servlet 对象的初始化、Servlet 对象服务的执行，及最终 Servlet 对象被销毁的整个过程。

![image-20210526101539889](https://gitee.com/Akihij/PicGo/raw/master/img/20210526101539.png)



Servlet 的整个生命周期过程的执行，均由 Web 服务器负责管理。即 **Servlet 从创建到服务到销毁的整个过程中方法的调用，都是由 Web 服务器负责调用执行，程序员无法控制其执行流程。**

但程序员可以获取到 Servlet 的这些生命周期时间点，并可以指定让 Servlet 做一些具体业务相关的事情。

##### 1.1.1、 生命周期方法执行流程

![image-20210526101707654](https://gitee.com/Akihij/PicGo/raw/master/img/20210526101707.png)



Servlet 生命周期方法的执行流程：

1. 当请求发送到 Web 容器后，Web 容器会解析请求 URL，并从中分离出 Servlet 对应的URI。 

2. 根据分离出的 URI，通过 web.xml 中配置的 URI 与 Servlet 的映射关系，找到要执行的Servlet，即找到用于处理该请求的 Servlet。 

3. 若该 Servlet 不存在，则调用该 Servlet 的无参构造器、init()方法，实例化该 Servlet。然后执行 service()方法。

4. 若该 Servlet 已经被创建，则直接调用 service()方法。

5. 当 Web 容器被关闭，或该应用被关闭，则调用执行 destroy()方法，销毁 Servlet 实例。



~~~java
public class HelloServlet implements Servlet{
    public HelloServlet(){
        System.out.println("实例化");
    }
    
    public void init(ServletConfig config){
        System.out.println("初始化");
    }
    
    
    public void service(ServletRequest request,ServletResponse response){
        System.out.println("服务");
    }
    
    
    public void destory(){
        System.out.println("销毁");
    }
    
}
~~~



##### 1.1.2、Servlet特征

1. Servlet 是单例多线程的。 

2. 一个 Servlet 实例只会执行一次无参构造器与 init()方法，并且是在第一次访问时执行。

3. 用户每提交一次对当前 Servlet 的请求，就会执行一次 service()方法。

4. 一个 Servlet 实例只会执行一次 destroy()方法，在应用停止时执行。

5. 由于 Servlet 是单例多线程的，所以为了保证其线程安全性，一般情况下是不为 Servlet类定义可修改的成员变量的。因为每个线程均可修改这个成员变量，会出现线程安全问题。

6. 默认情况下，Servlet 在 Web 容器启动时是不会被实例化的。



##### 1.1.3、Web 容器启动时创建 **Servlet** 实例

在默认的情况下，Servlet在Web容器启动时不会被实例化

修改配置文件：

~~~xml
<servlet>
	<servlet-name>hello-servlet</servlet-name>
	<servlet-class>com.jiang.HelloServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>hello-servlet</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
~~~

在<servlet/>中添加<load-on-startup/>的作用是，**标记是否在Web服务器（这里是Tomcat）启动时创建并初始化这个 Servlet 实例**，即是否在 Web 服务器启动时调用执行该 Servlet 的无参构造器方法与 init()方法，而不是在真正访问时才创建。

- 它的值必须是一个整数。

- 当值大于等于 0 时，表示容器在启动时就加载并初始化这个 Servlet，数值越小，该 Servlet的优先级就越高，其被创建的也就越早；

- 当值小于 0 或者没有指定时，则表示该 Servlet 在真正被使用时才会去创建。 

- 当值相同时，容器会自己选择创建顺序。



##### 1.1.4、getServletInfo()方法

Servlet 接口中的方法 getServletInfo()，是由程序没自己定义的有关当前 Servlet 的一些基本信息，不属于 Servlet 生命周期中的方法。



#### 1.2、SerlvetConfig

##### 1.2.1、什么是ServletConfig

在 Servlet 接口的 init()方法中具有唯一的一个参数 **ServletConfig**。

ServletConfig 是个接口，顾名思义，就是 Servlet 配置，**即在 web.xml 中对当前 Servlet 类的配置信息。**Servlet 规范将Servlet 的配置信息全部封装到了 ServletConfig 接口对象中。

在 Web 容器调用 init()方法时，Web 容器首先会将 web.xml 中当前 Servlet 类的配置信息封装为一个对象。这个对象的类型实现了 ServletConfig 接口，Web 容器会将这个对象传递给init()方法中的 ServletConfig 参数。



##### 1.2.2、获取ServletConfig

**由于 ServletConfig 对象是 Web 容器通过 init()方法传递给当前 Servlet 类的**，而 init()方法只会在 Servlet 对象初始化时调用一次。所以，需要在 init()方法中将 ServletConfig 对象传递给 Servlet 的 ServletConfig 成员变量，这样 service()方法即可使用 ServletConfig 对象了。也就是说，我们需要在 Servlet 中声明一个ServletConfig 成员变量。

声明了成员变量之后，会不会存在线程安全问题？

ServletConfig变量的赋值是在init()方法进行赋值，对于所有线程来说，SerlvetConfig对象是可读的，但不能修改。



##### 1.2.3、ServletConfig 中的方法

在Java帮助文档中，可以看到接口的四个方法：

![image-20210526103836257](https://gitee.com/Akihij/PicGo/raw/master/img/20210526103836.png)



~~~xml
<servlet>
	<servlet-name>hello-servlet</servlet-name>
	<servlet-class>com.jiang.helloServlet</servlet-class>
    <!--设置初始化参数-->
    <init-param>
        <param-name>myName</param-name>
        <param-value>JiangZW</param-value>
    </init-param>
    
    <init-param>
    	<param-name>myAge</param-name>
        <param-value>2</param-value>
    </init-param>
</servlet>
~~~



- `getInitParameter()`：获取指定名称的初始化参数值。

  ~~~java
  getInitParameter("myName")
  ~~~

- `getInitParameterNames()`：获取当前 Servlet 所有的初始化参数名称。其返回值为枚举类型 Enumeration<String>。 

  ~~~java
  getInitParameterNames()
  ~~~

- `getServletName()`：获取当前 Servlet 的<servlet-name></servlet-name>中指定的 Servlet 名称。如上图中的 ServletName 为"hello-servlet"。

  ~~~java
  getServletName()
  ~~~

- `getServletContext()`：获取到当前 Servlet 的上下文对象 ServletContext。这是个非常重要的对象。

  ~~~java
  getServletContext()
  ~~~

  



##### 1.2.4、ServletConfig的特点

对于不同的Servlet，Tomcat会为其创建不同的ServletConfig，用于封装各自的配置信息。也就是说，**一个 Servlet 就会有其对应的一个 ServletConfig 对象。**





#### 1.3、ServletContext

##### 1.3.1、什么是ServletContext

ServletContext，即 **Servlet 上下文环境**，是个接口，是 **Web 应用中所有 Servlet 在 Web 容器中的运行时环境**。

**这个运行时环境随着 Web 应用的启动而创建，随着 Web 应用的关闭而销毁。也就是说，一个 Web 应用，就一个 Servlet 运行时环境，即一个ServletContext 对象。**



ServletContext运行环境中有哪些具体的内容？

- web.xml 文件中的配置信息

- Servlet之间 可以共享的数据



可以这么说，ServeltContext 可以代表整个Web应用。所以，ServletConetxt有另外一个名称：`application`。



##### 1.3.2、ServletContext中的方法

查看 Java的帮助文档，可以看到 `javax.servlet.ServletContext `接口中包含很多方法。

下面我们介绍几个常用的重要方法。

在 web.xml 中通过<context-param/>可以进行上下文参数的配置。



> 上下文参数<context-param/>与初始化参数<init-param/>是不同的：
>
> ​		上下文参数是当前 Web 应用中所有 Servlet 共享的参数，每个 Servlet 均可获取到.
>
> ​		Servlet 初始化参数则是只有当前Servlet 可以获取到的参数。



~~~xml
<context-param>
    <param-name>myName</param-name>
    <param-value>JiangZW</param-value>
</context-param>

<context-param>
    <param-name>myAge</param-name>
    <param-value>2</param-value>
</context-param>

<servlet>
	<servlet-name>helloServlet</servlet-name>
    <serlvet-class>com.jiang.servlet</serlvet-class>
</servlet>
~~~



方法：

- `String getInitParameter ()`

  获 取 web.xml 文 件 的 <context-param/> 中 指 定 名 称 的 上 下 文 参 数 值 。 

  

- `Enumeration getInitParameterNames()`

  获取 web.xml 文件的<context-param/>中的所有的上下文参数名称。

  

- `void setAttribute(String name, Object object)`

  ~~~java
  setAttribute("User",new User());
  ~~~

  在 ServletContext 的公共数据空间中，也称为域属性空间，放入数据。

  **这些数据对于 Web应用来说，是全局性的，与整个应用的生命周期相同。**

  

- `Object getAttribute(String name)`

  ~~~java
  User user = (User)getAttribute("User");
  ~~~

  从 ServletContext 的域属性空间中获取指定名称的数据。

  

- `void removeAttribute(String name)`

  ~~~java
  removeAttribute("User")
  ~~~

  从 ServletContext 的域属性空间中删除指定名称的数据。



- `String getRealPath(String path)`

  获取当前 Web 应用中指定文件或目录在本地文件系统中的路径，是基于盘符的路径。绝对路径。

- `String getContextPath()`

  获取当前应用在 Web 容器中的名称。



#### 1.4、欢迎页面设置

##### 1.4.1、欢迎页面设置

欢迎页面，是指当在浏览器地址栏中直接通过项目名称访问时，默认显示的页面，即默认访问的路径，可以是个.html、.jsp 等页面，也可以是 Servlet 的访问路径等。

~~~xml
<welcome-file-list>
	<welcome-file>index.jsp</welcome-file>
</welcome-file-list>
~~~

在 web.xml 中有一个<welcome-file-list/>标签，用于指定当前应用的欢迎页面。



##### 1.4.2、指定多个欢迎页面

~~~xml
<welcome-file-list>
	<welcome-file>index1.jsp</welcome-file>
	<welcome-file>index2.jsp</welcome-file>
	<welcome-file>index3.jsp</welcome-file>
</welcome-file-list>
~~~

可以为应用指定多个欢迎页面，但只会有一个起作用。系统加载这些欢迎页面的顺序与其注册顺序相同，即由上到下逐个查找。一旦找到，则马上显示，不会再向下查找





#### 1.5、<url-pattern/>的设置与匹配

<url-pattern/>标签用于对请求进行筛选匹配，对当前注册的 Servlet 所要处理的请求类型进行筛选。

对于<url-pattern/>中路径的写法，有多种不同模式，表示不同的意义。

##### 1.5.1、精确路径模式

请求路径必须与<url-pattern/>的值完全相同才可被当前 Servlet 处理。

~~~xml
<servlet-mapping>
    <servlet-name>someServlet</servlet-name>
    <url-pattern>/some</url-pattern>
    <url-pattern>/s1/s2</url-pattern>
</servlet-mapping>
~~~



##### 1.5.2、通配符路径模式

该模式中的路径由两部分组成：精确路径部分与通配符部分。

请求路径中只有携带了<url-pattern/>值中指定的精确路径部分才可被当前 Servlet 处理。

~~~xml
<servlet-mapping>
    <servlet-name>someServlet</servlet-name>
    <url-pattern>/some/*</url-pattern>
</servlet-mapping>
~~~



##### 1.5.3、全路径模式

提交的所有请求全部可被当前的 Servlet 处理。其值可以指定为/*，也可指定为/。

~~~xml
<servlet-mapping>
    <servlet-name>someServlet</servlet-name>
    <url-pattern>/*</url-pattern>
</servlet-mapping>

<servlet-mapping>
    <servlet-name>someServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
~~~



> /*与/表示所有请求均会被当前 Servlet 所处理。这两个路径的不同之处是：
>
> 1. 使用/*，表示当前的 Servlet 会拦截用户对于静态资源（.css、.js、.html、.jpg、.png…..）与动态资源（.jsp）的请求。即用户不会直接获取到这些资源文件，而是将请求交给当前 Servlet
>
> 来处理了。
>
> 2. 使用/，表示当前的 Servlet 会拦截用户对于静态资源（.css、.js、.html、.jpg、.png…..），但对于动态资源的请求，是不进行拦截的。即用户请求的静态资源文件是不能直接获取到的。



##### 1.5.4、后缀名模式

请求路径最后的资源名称必须携带<url-pattern/>中指定的后辍名，其请求才可被当前Servlet 处理。

~~~xml
<servlet-mapping>
	<servlet-name>someServlet</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>
~~~

















