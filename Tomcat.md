# Tomcat & Jetty

[TOC]

## 基础



**Web容器**：HTTP服务器 +  Servlet容器

+ HTTP服务器：负责监听、解析、处理HTTP请求、返回HTTP响应
+ Servlet容器：负责管理Servlet的生命周期
+ Spring框架：是对Servlet的封装，本身就是一个Servlet



**HTTP工作流程**：

![](https://static001.geekbang.org/resource/image/f5/ca/f5bd0c7840160d5a121c191e7e54b4ca.jpg)



HTTP是无状态的，所以为了保存会话状态引入了`Cookie`和`Session`技术

+ Cookie: HTTP的请求头，本质上是存储在用户本地的文件，里面包含了每次请求所需传递的信息
+ Session:数据存储在服务端，每个sessionId对应一个session对象



**Servlet规范与Servlet容器**



解耦HTTP服务器与Servlet业务类：

![](https://static001.geekbang.org/resource/image/df/01/dfe304d3336f29d833b97f2cfe8d7801.jpg)

Servlet规范：Servlet容器 + Servlet接口

```java
// Servlet接口
public interface Servlet {
    void init(ServletConfig config) throws ServletException;
    // 可以拿到web.xml中的设置的参数
    ServletConfig getServletConfig();
    // ServletRequest是对HTTP请求的封装 
  	// ServletResponse是对HTTP响应的封装 本质上是对通信协议的封装
    void service(ServletRequest req, ServletResponse res）throws ServletException, IOException;
    
    String getServletInfo();
    
    void destroy();
}
```



工作流：

1. HTTP服务器将请求封装为`ServletRequest`对象，将请求对象传递给Servlet容器
2. Servlet容器拿到请求后，根据URL和Servlet的映射关系，找到对应的Servlet，如果该Servlet还没有加载，则利用反射机制创建该类，并调用该类的init方法完成初始化
3. 调用Servlet的service方法处理请求
4. 将`ServletResponse`对象返回给HTTP服务器
5. HTTP服务器将响应对象解析为响应报文，发送给客户端



**Web应用目录结构**:

```
| -  MyWebApp
      | -  WEB-INF/web.xml        -- 配置文件，用来配置Servlet等
      | -  WEB-INF/lib/           -- 存放Web应用所需各种JAR包
      | -  WEB-INF/classes/       -- 存放你的应用类，比如Servlet类
      | -  META-INF/              -- 目录存放工程的一些信息
```

Servlet规范规定：一个Web应用会对应一个`ServletContext`接口，Web应用部署好后，servlet容器在启动时会加载Web应用，并为该应用创建一个唯一的全局对象`ServletContext`，不同的Servlet可以通过ServletContext共享数据



Servlet规范提供的扩展点：

+ Filter
+ Listener: 监听Servlet容器中发生的各种事件



三个容器：

+ Servlet容器：用于管理Servlet生命周期的
+ Spring容器：用于管理Spring bean（service dao）生命周期的
+ Spring MVC容器：用于管理Spring MVC Bean(controller)生命周期的 



Web容器完整启动过程

Tomcat/Jetty启动，对每个WebApp，依次执行初始化工作

1. 每个webapp都会对应一个classLoader和ServletContext
2. ServletContext初始化时，会扫描`web.xml`中的Listener、filter、servlet配置
3. 如果**Listener**中配有Spring的ContextLoaderListener，ContextLoaderListener会获取webapp的状态信息，会在ServletContext初始化时，对Spring IOC容器进行初始化，并将容器存放到ServletContext中
4. 如果Servlet中配有SpringMVC的DispatcherServlet，DispatcherServlet初始化时（即第一次请求到达时），会初始化SpringMVC容器，SpringMVC容器通过ServletContext获取Spring容器，并将其设置为自己的父容器，子容器可以访问父容器(controller可以访问service)，初始化完毕后，DispatcherServlet处理MVC中URL和servlet的映射关系



**手动实现一个Servlet**

步骤：

1. 下载tomcat

2. 编写一个继承`HttpServlet`的类 重写`doGet` `doPost`方法

   ```java
   public class MyServlet extends HttpServlet {
   		// 模板方法设计模式
       @Override
       protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
           System.out.println("get method");
           // setContentType要在getWriter前 否则编码格式不起作用
           resp.setContentType("text/html;charset=utf-8");
           PrintWriter writer = resp.getWriter();
           writer.println("<h1>hello daxiao you send a get method</h1>");
       }
   
       @Override
       protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
           System.out.println("post method");
           // setContentType要在getWriter前 否则编码格式不起作用
           resp.setContentType("text/html;charset=utf-8");
           PrintWriter writer = resp.getWriter();
           writer.println("<h1>hello daxiao you send a post method</h1>");
       }
   }
   ```

3. 编译Java -> class

   `javac -cp servlet-api.jar MyServlet.java`

4. 建立web应用的目录结构并配置web.xml

   ```xml
   <servlet>
       <servlet-name>myServlet</servlet-name>
       <servlet-class>MyServlet</servlet-class>
     </servlet>
   
     <servlet-mapping>
           <servlet-name>myServlet</servlet-name>
           <url-pattern>/myServlet</url-pattern>
     </servlet-mapping>
   ```

   

5. 部署web应用

   tomcat负责实例化servlet

   `/webapp/MyWebApp/WEB-INF/classs/MyServlet.class`

   `/webapp/MyWebApp/WEB-INF/web.xml`

6. 启动tomcat

   `./startup.sh`

7. 浏览器访问 查看tomcat日志



Tomcat目录结构：

+ `/bin` 存放启动或者关闭的脚本文件
+ `/conf` 全局配置文件 最重要的是`server.xml`
+ `/lib` 存放Tomcat以及所有Web应用都可以用的jar包
+ `/log` Tomcat日志
+ `/webapps` 应用目录 



## 系统架构



先了解需求 -> 设计系统

**Web容器的需求**：

+ HTTP服务器：处理Socket连接，负责网络字节流和Request Response对象的转化
+ Servlet容器：加载和管理Servlet，具体处理Request请求

**系统的两大组成部分**：

+ 连接器 *Connector*
+ 容器 *Container*

![](https://static001.geekbang.org/resource/image/ee/d6/ee880033c5ae38125fa91fb3c4f8cad6.jpg)



### **连接器的设计**

连接器为容器屏蔽了IO模型和应用层协议的细节，为容器提供统一的ServletRequest对象



**连接器的功能**(按步骤来)：

1. 监听网络端口
2. 接收网络连接请求
3. 读取网络请求字节流
4. 根据具体的应用层协议（HTTP/AJP）解析字节流，生成统一的`Tomcat Request`对象
5. 将`Tomcat Request`对象转化为`ServletRequest`对象
6. 调用`Servlet`容器，获取`ServletResponse`，
7. 将ServletResponse对象转化为Tomcat Response对象
8. 将Tomcat Response对象转化为网络字节流
9. 将响应字节流写回给浏览器



连接器主要分为三个高内聚的模块：

+ 网络通信 `Endpoint`

  Tomcat支持的IO模型：

  + NIO
  + NIO.2
  + APR

  组成部分:

  + `Acceptor` 监听Socket连接请求
  + `SocketProcessor` 处理接收到的Socket请求，是`Runnable`的实现子类，在run方法里调用协议解析组件Processor，被提交给线程池进行处理

+ 应用层协议解析 `Processor`

  Tomcat支持的应用层协议：

  + HTTP/1.1
  + AJP
  + HTTP/2

  主要功能：将字节流和Tomcat Request/Response对象的转化

+ Tomcat Request/Response 和 Servlet Request/Response的转化 `Adapter`

> ProtocolHandler = EndPoint + Processor

![](https://static001.geekbang.org/resource/image/6e/ce/6eeaeb93839adcb4e76c15ee93f545ce.jpg)

连接器细化：

![](https://static001.geekbang.org/resource/image/30/cf/309cae2e132210489d327cf55b284dcf.jpg)



### 容器的设计

容器的结构：

> Engine 对应一个Service
>
> > Host 对应一个虚拟主机
> >
> > > Context 对应一个Web应用
> > >
> > > > Wrapper: Servlet 

![](https://static001.geekbang.org/resource/image/cc/ed/cc968a11925591df558da0e7393f06ed.jpg)



```xml
<Server port="8005" shutdown="SHUTDOWN"> 可以包含多个service
  <!-- A "Service" is a collection of one or more "Connectors" that share
       a single "Container" Note:  A "Service" is not itself a "Container",
       so you may not define subcomponents such as "Valves" at this level.
       Documentation at /docs/config/service.html
   -->
  <Service name="Catalina"> 可以包含一个Engine多个 连接器
    <Connector port="8080" protocol="HTTP/1.1" 
               connectionTimeout="20000"
               redirectPort="8443" /> 连接器 通信接口
    <Engine name="Catalina" defaultHost="localhost"> 处理Service下的所有请求
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Context> Web容器
        </Context>
      </Host> 处理特定下host下的请求 可以包含多个Context
    </Engine>
  </Service>
</Server>

```



Tomcat**管理容器的方法**：组合模式，一致化树的分支和叶子结点操作

```java
// Engine Host Context Wrapper都实现Container接口
public interface Container extends Lifecycle {
    public void setName(String name);
    public Container getParent();
    public void setParent(Container container);
    public void addChild(Container child);
    public void removeChild(Container child);
    public Container findChild(String name);
}
```



定位Servlet的过程：通过`Mapper`组件

1. 根据协议和端口号找到Service和Engine
2. 根据域名找到Host
3. 根据URL路径找到Context组件和Wrapper



处理请求的过程：父容器处理完之后传递给子容器——责任链模式

链表结点:  Valve

```java
public interface Valve {
  public Valve getNext();
  public void setNext(Valve valve);
  public void invoke(Request request, Response response)
}
```

链表：

```java
public interface Pipeline extends Contained {
  public void addValve(Valve valve);
  public Valve getBasic();
  public void setBasic(Valve valve);
  public Valve getFirst();
}
```

![](https://static001.geekbang.org/resource/image/b0/ca/b014ecce1f64b771bd58da62c05162ca.jpg)

> Adapter调用Engine中的第一个Valve
>
> Wrapper中的最后一个Valve会创建一个FilterChain 最终调用到Servlet的Service方法



### Tomcat实现一键式启停

[嵌入式方式启动Tomcat](https://github.com/heroku/devcenter-embedded-tomcat)



**一个HTTP请求在Tomcat中的流转过程**

![](https://static001.geekbang.org/resource/image/12/9b/12ad9ddc3ff73e0aacf2276bcfafae9b.png)



**设计系统时需要解决的问题**：

+ 如何管理组件的创建 初始化 启动 停止 销毁（生命周期管理）
+ 如何方便的添加和删除组件
+ 如何做到组件启动和停止不重复 不遗漏



Tomcat**组件之间的关系**：

- 大小关系：大组件管理小组件 server管理service service管理connector和engine
- 内外关系：外层组件调用内层组件完成功能



**组件创建顺序**（由组件依赖关系决定）：

+ 先创建子组件 再创建父组件 子组件需要被注入到父组件中
+ 先创建内层组件 再创建外层组件 内层组件需要被注入到外层组件中 



**一键式启停的关键**：Lifecycle接口（每个组件都实现该接口）

组合模式：父组件调用`init()`方法的时候，会创建子组件，并调用子组件的`init()`方法，`start()`方法也是同理，这样只要Server. init() start() 整个Tomcat就都启动起来了



**可扩展性**：LifeCycle事件（观察者模式）

当我们需要往init或者start等方法里面加逻辑时，会不可避免地违反开闭逻辑，这个时候我们可以把init看成是状态转变的一个事件，给这个事件添加监听器，低耦合，非侵入式地添加功能

![](https://static001.geekbang.org/resource/image/dd/c0/dd0ce38fdff06dcc6d40714f39fc4ec0.png)



**代码复用**：LifeCycleBase 抽象基类（模板设计模式）

存放公共的逻辑：生命状态的转变和维护、生命事件的触发、监听器的添加和删除

![](https://static001.geekbang.org/resource/image/67/d9/6704bf8a3e10e1d4cfb35ba11e6de5d9.png)



```java
// LifeCycleBase
@Override
public final synchronized void init() throws LifecycleException {
    //1. 状态检查
    if (!state.equals(LifecycleState.NEW)) {
        invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
    }

    try {
        //2.触发INITIALIZING事件的监听器
        setStateInternal(LifecycleState.INITIALIZING, null, false);
        
        //3.调用具体子类的初始化方法
        initInternal();
        
        //4. 触发INITIALIZED事件的监听器
        setStateInternal(LifecycleState.INITIALIZED, null, false);
    } catch (Throwable t) {
      ...
    }
}
```



监听器的注册时间：

+ 父组件创建子组件时注册的
+ Tomcat启动时会解析server.xml 创建里面的监听器并注册到组件中



生命周期管理图

![](https://static001.geekbang.org/resource/image/de/90/de55ad3475e714acbf883713ee077690.png)





### Tomcat的高层组件负责做的事情



**Tomcat启动过程**

1. `./startup.sh`启动JVM，并运行Tomcat的启动类`Bootstrap`
2. Bootstrap中会初始化Tomcat自己的类加载器，并创建Catalina
3. Catalina也是一个启动类，会解析`server.xml`文件，创建相应的组件，创建Server 并调用其start方法
4. Server管理service 调用service的start方法
5. Service 管理连接器和engine 调用连接器和engine的start方法



**Catalina**

主要任务：创建Server，并调用其init和start方法

具体实现：

+ 解析server.xml,将里面配置的组件创建出来，再调用server的init和start方法
+ 创建并注册关闭钩子：观察者模式(注册某个事件对应的动作)，JVM退出时 执行catalina.stop()



**Server**

主要任务：

+ 管理Service，用一个数组存储，且数组长度动态扩增，每次增大一个空间
+ await方法中监听8005端口，监测socket中传来的数据是否是`SHUTDOWN`，如果是就退出轮询，进入stop流程



**Service**

+ 管理connector和engine
+ `MapperListner`：监听器，监听容器的变化，并信息更新到Mapper中



**Engine**

+ 管理host组件
+ 启动host组件时用的是线程池（ContainerBase里面的 所以所有容器都是用线程池）
+ 处理请求，将请求转发给某个host处理





### Jetty的Connector

[Jetty源码查看](https://github.com/jetty-project/embedded-jetty-jsp)

整体结构

![](https://static001.geekbang.org/resource/image/95/b6/95b908af86695af107fd3877a02190b6.jpg)



+ Connector：实现HTTP服务器的功能
+ Handler：可插拔，需要支持servlet就配置使用servletHandler，需要支持session就配置使用sesionhandler
+ Threadpool：全局线程池(Tomcat中每个连接器都有自己的线程池)
+ Server:负责创建并初始化以上三个组件



**Connector**：用于封装IO模型和应用层协议



| 服务端IO通信主要做的三件事 | Jetty中对应的组件 |
| -------------------------- | ----------------- |
| 监听连接                   | Acceptor          |
| IO事件查询                 | SelectorManager   |
| 数据读写                   | Connection        |



**Acceptor**

+ 是一个Runnable，通过全局线程池来执行
+ 通过阻塞方式接收连接
+ 接收连接成功后 设置channel为非阻塞模式 并交给selectorManager处理



**SelectorManager**

selectorManager处理channel的流程

1. 从自身的selector数组中选择一个selector来处理这个channel

2. 将channel注册到这个selector上 拿到selectionKey

3. 创建一个endpoint和connection 并将其和selectionkey绑定在一起

   

**Connection**

- 是endpoint的一个内部类，一个runnable

- 负责具体协议的解析 得到request对象
- 调用handler容器进行处理



请求的处理：

- 向endpoint中注册回调方法，告诉endpoint等数据到了就调用这些方法
- 回调方法中，调用endpint的接口读取数据，解析HTTP信息，封装成request对象



响应的处理：

通过endpoint将数据写入到channel中





**Connector工作流程**

![](https://static001.geekbang.org/resource/image/b5/83/b526a90be6ee4c3e45c94e122c9c1e83.jpg)

1. Acceptor监听连接请求，当有连接请求就接收连接，一个连接对应一个channel，acceptor将channel交给managedSelector处理
2. managedSelector将channel注册到selector上，并创建一个endpoint和connection跟channel绑定，接着不断监测IO事件
3. IO事件到了就调用endpoint的方法拿到一个runnable 扔给线程池执行
4. runnable中解析读到的数据，生成request对象交给handler处理



设计特点：使用了回调函数



### Jetty的Handler



Handler接口

```java

public interface Handler extends LifeCycle, Destroyable
{
    //处理请求的方法
    public void handle(String target, Request baseRequest, HttpServletRequest request, HttpServletResponse response)
        throws IOException, ServletException;
    
    //每个Handler都关联一个Server组件，被Server管理
    public void setServer(Server server);
    public Server getServer();

    //销毁方法相关的资源
    public void destroy();
}
```



**Handler继承关系**

![](https://static001.geekbang.org/resource/image/3a/64/3a7b3fbf16bb79594ec23620507c5c64.png)

关键点解析：

+ `AbstractHanlderContainer`：因为handler要把请求传递给其他的hanlder（实现责任链模式的链式调用）所以要持有其他handler对象的引用，所以叫container，它的两个子类的区别就在于拥有的handler个数不同
+ `Server`：Handler模块的入口
+ `scopedHanlder`：具有上下文信息，其子类用来实现servlet规范(servlet listener filter)
+ `HandlerCollection`：维护了一个Handler数组，比如server中就有一个handlercollection，其中的每个handler都对应一个web应用的handler入口



**Handler类型**

+ 协调handler：负责转发请求，如handlerCollection
+ 过滤器handler：自己完处理请求后再转发请求，如handlerWrapper
+ 内容handler：会真正调用servlet处理请求，并生成响应的内容



Servlet规范中需要有的组件

- Context
- Servlet
- Listener
- Filter
- Session



**调用过程**

![](https://static001.geekbang.org/resource/image/5f/c1/5f1404567deec36ac68c36e44bb06cc1.jpg)



**定制化方法**

将通用组件抽象成了Handler，添加或者删除功能时，就在执行链中添加或者删除handler即可（微内核 + 插件的设计思想）



### 组件化设计规范

**Web容器如何实现组件化设计**

- 根据需求，按照高内聚低耦合的原则将不同的功能进行拆分，每个组件抽象成接口
- 组件的载体(Server) + 组件的连接方式（职责链模式）
- 用户可以通过配置来组装组件



**组件的创建**

1. 扫描配置

2. 解析配置

3. 反射创建

   > 实例化对象之前Web容器需要组件的类加载到JVM中 Tomcat实现了自己的类加载器 
   >
   > 且给每个Web应用都分配了类加载器（Spring用的类加载器就是这个）



**组件的生命周期管理**

- 父组件负责子组件的初始化 启停 销毁 一键式启停（组合模式）
- 定义了生命周期状态，将状态的转化作为观察的事件，从而触发在事件上注册的行为（观察者模式）



**组件的骨架抽象类和模板模式**

在抽象类中实现通用逻辑，并提供扩展点（由子类实现的抽象方法）

+ 代码复用
+ 扩展性



### 提高Tomcat启动速度



- 清理Tomcat
  - 清理不必要的Web应用，每次Tomcat重新启动都会重新部署所有的Web项目
  - 清理xml配置文件，减少解析所需的时间
  - 清理Jar文件
  - 及时清理日志
- 禁止Tomcat TLD扫描
- 关闭Websocket支持
- 关闭JSP支持
- 禁止Servlet扫描注解
- 配置Web-fragment扫描
- 随机数熵源优化
- 并行启动多个Web应用



### 如何学习源码



开源框架和中间件：

- 服务接入层：反向代理Nginx, API网关Node.js
- 业务逻辑层：Web容器Tomcat, Jetty 应用层框架Spring SpringMVC SpringBoot MyBatis
- 数据缓存层：内存数据库Redis 消息中间件Kafka
- 数据存储层：关系型数据库MySQL 非关系型数据库MongoDB 文件存储HDFS，搜索分析引擎Elasticeearch
- RPC框架：Spring Cloud Dubbo
- 网络通信：Netty 分布式协调：ZooKeeper



先学习简单或者熟悉的的框架或中间件，Tomcat Spring，把一个学透然后学其他的就能总结出套路



**学习源码的要点**：

- 弄清楚中间件的核心功能是什么
- 完成这些核心功能涉及到的代码流程先研究 不要深入到其他细节
- 将源码跑起来 打断点调试看变量的值和调用栈
- 带着问题去学习源码 了解某个功能如何实现的 自己设计这些功能时会怎么做 弄清楚细节后及时记录下来 画流程图或者类图
- 参考官方文档 看大概的设计思路 看网上博客 参考别人的分析





## 连接器

### NioEndpoint组件：Tomcat如何实现非阻塞I/O



UNIX下的IO模型

- 同步阻塞
- 同步非阻塞
- IO多路复用
- 信号驱动
- 异步



IO:

> 计算机内存与外部设备拷贝数据的过程

IO模型需要解决的问题：

程序通过CPU向外部设备发送一个读指令后，数据到达内存需要一定的时间，这时程序面临的选择有：

- 把CPU让出去（阻塞）
- 让CPU不断轮询数据有没有到达（非阻塞）



**JavaIO模型**

当用户线程发起IO操作后，读取网络数据会经历两个步骤

1. 用户线程等待内核将数据从网卡拷贝到内核空间
2. 内核将数据从内核空间拷贝到用户空间



| IO模型       | 实现网络数据读取的方式                                       |
| ------------ | ------------------------------------------------------------ |
| 同步阻塞IO   | 用户线程在进行read调用后阻塞了，让出CPU，之后内核等待网卡数据到来，将数据从网卡拷贝到内核空间，再将数据从内核空间拷贝到用户空间，内核唤醒用户线程，read调用返回 |
| 同步非阻塞IO | 用户线程不断发起read调用，在数据到达内核空间之前一直返回失败（此时用户线程在轮询时可以做其他的事），到达之后，此时read调用就会阻塞，之后的逻辑跟阻塞IO一样 |
| IO多路复用   | 将用户线程的读取操作分为两个步骤，线程首先发起select调用，向内核中的多个channel查询数据是否准备好了，准备好了之后，再发起read调用，阻塞等待数据从内核空间拷贝到用户空间 |
| 异步IO       | 用户线程发起read调用时，会注册一个回调函数，read立即返回，内核将数据准备好后，会调用注册的回调函数完成处理（整个过程用户线程一直没有阻塞） |



**NioEndpoint组件**

实现了IO多路复用模型

Java中多路复用器的使用步骤：

1. 创建一个Selector，在它身上注册各种感兴趣的事件，调用select，等待感兴趣的事件发生
2. 感兴趣的事件发生了，比如说可读了，那就创建一个新的线程从channel中读数据



组成：

![](https://static001.geekbang.org/resource/image/c4/65/c4bbda75005dd5e8519c2bc439359465.jpg)

- Limitlatch：限制连接数
- accpetor:监听连接请求，将channel交给一个poller(跑在一个单线程里面，在一个死循环里面用accept方法接收新请求)
- Poller:检车channel的IO事件，可读时创建socketprocessor任务类给线程池处理（每一个poller就是一个selector，都维护了一个channel数据，也是泡在单独的线程里面）
- 任务类run方法中：调用http11processor处理请求，http11processor通过niosocketwrapper读写数据





**LimitLatch**

就类似于一个Semephore，用于控制连接个数，当连接数到达Limit之后操作系统底层还是会接收客户端连接，但是用户层不再接收

```java

public class LimitLatch {
  // AQS是一个基于队列的同步器，里面维护了一个状态和线程队列
  // 这里的Sync没有用到state作为状态 而是用外部类的count作为状态
    private class Sync extends AbstractQueuedSynchronizer {
     
      /*
      	在AQS的acquireSharedInterruptibly()方法中
      	会调用此方法 返回值>0表明该线程获取到了资源 
      	<0表明需要将该线程挂起 并加入到阻塞队列中
      */
        @Override
        protected int tryAcquireShared() {
            long newCount = count.incrementAndGet();
            if (newCount > limit) {
                count.decrementAndGet();
                return -1;
            } else {
                return 1;
            }
        }

      /*
      	AQS的releaseShared()中会调用此方法
      	返回true表明有资源释放 可以唤醒阻塞队列中的线程
      */
        @Override
        protected boolean tryReleaseShared(int arg) {
            count.decrementAndGet();
            return true;
        }
    }
		
    private final Sync sync;
    private final AtomicLong count;
    private volatile long limit;
    
    //线程调用这个方法来获得接收新连接的许可，线程可能被阻塞
    public void countUpOrAwait() throws InterruptedException {
      sync.acquireSharedInterruptibly(1);
    }

    //调用这个方法来释放一个连接许可，那么前面阻塞的线程可能被唤醒
    public long countDown() {
      sync.releaseShared(0);
      long result = getCount();
      return result;
   }
}
```



**Acceptor**

实现了`Runnable`接口，一个端口号对应一个`ServerSocketChannel`，多个acceptor共享一个`serversocketchannel`

```java
serverSock = ServerSocketChannel.open();
// acceptCount表示操作系统的ACCEPT等待队列长度
serverSock.socket().bind(addr,getAcceptCount());
// 以阻塞方式接收连接(read系统调用后阻塞线程)
serverSock.configureBlocking(true);
```

主要功能：

- `run`方法中使用`serverSocketChannel`的`accept()`方法获取新的连接（socketchannel对象）
- 将socketchannel对象封装在pollereven对象中，将其放入poller的queue中（生产者-消费者模式）



源码解析：

- 启动acceptor线程

```java
// NioEndpoint
public void startInternal() throws Exception {

        if (!running) {
          // 调用基类AbstractEndpoint的方法
            startAcceptorThreads();
        }
}

// AbstractEndpoint 维护了一个acceptor数组
protected final void startAcceptorThreads() {
        int count = getAcceptorThreadCount();
        acceptors = new Acceptor[count];

        for (int i = 0; i < count; i++) {
            acceptors[i] = createAcceptor();
            String threadName = getName() + "-Acceptor-" + i;
            acceptors[i].setThreadName(threadName);
            Thread t = new Thread(acceptors[i], threadName);
            t.setPriority(getAcceptorThreadPriority());
            t.setDaemon(getDaemon());
            t.start();
        }
    }

```



- Run方法逻辑

```java
public void run() {

            int errorDelay = 0;

            // Loop until we receive a shutdown command
            while (running) {

                // Loop if endpoint is paused
                while (paused && running) {
                    state = AcceptorState.PAUSED;
                    try {
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        // Ignore
                    }
                }

                if (!running) {
                    break;
                }
                state = AcceptorState.RUNNING;

                try {
                    //if we have reached max connections, wait
                    countUpOrAwaitConnection();

                    SocketChannel socket = null;
                    try {
                      // 调用该endpoint唯一的Serversocketchannel的accept方法 获取socketchannel（连接）
                        // Accept the next incoming connection from the server
                        // socket
                        socket = serverSock.accept();
                    } catch (IOException ioe) {
                        // We didn't get a socket
                        countDownConnection();
                        if (running) {
                            // Introduce delay if necessary
                            errorDelay = handleExceptionWithDelay(errorDelay);
                            // re-throw
                            throw ioe;
                        } else {
                            break;
                        }
                    }
                    // Successful accept, reset the error delay
                    errorDelay = 0;

                    // Configure the socket
                    if (running && !paused) {
                        // setSocketOptions() will hand the socket off to
                        // an appropriate processor if successful
                      // 传递socketchannel给setSocketOptions
                        if (!setSocketOptions(socket)) {
                            closeSocket(socket);
                        }
                    } else {
                        closeSocket(socket);
                    }
                } catch (Throwable t) {
                    ExceptionUtils.handleThrowable(t);
                    log.error(sm.getString("endpoint.accept.fail"), t);
                }
            }
            state = AcceptorState.ENDED;
        }



    protected boolean setSocketOptions(SocketChannel socket) {
        // Process the connection
        try {
            //disable blocking, APR style, we are gonna be polling it
            socket.configureBlocking(false);
            Socket sock = socket.socket();
            socketProperties.setProperties(sock);

            NioChannel channel = nioChannels.pop();
            if (channel == null) {
                SocketBufferHandler bufhandler = new SocketBufferHandler(
                        socketProperties.getAppReadBufSize(),
                        socketProperties.getAppWriteBufSize(),
                        socketProperties.getDirectBuffer());
                if (isSSLEnabled()) {
                    channel = new SecureNioChannel(socket, bufhandler, selectorPool, this);
                } else {
                    channel = new NioChannel(socket, bufhandler);
                }
            } else {
                channel.setIOChannel(socket);
                channel.reset();
            }
          // 轮询从Poller数组中获取一个poller，并将niochannel注册到poller中(socketchannel放到了niochannel中)
            getPoller0().register(channel);
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            try {
                log.error("",t);
            } catch (Throwable tt) {
                ExceptionUtils.handleThrowable(tt);
            }
            // Tell to close the socket
            return false;
        }
        return true;
    }


// 将niochannel封装到niosocketwrapper中 将其封装到PollerEvent中 并放入Poller对象维护的一个同步队列中
public void register(final NioChannel socket) {
            socket.setPoller(this);
            NioSocketWrapper ka = new NioSocketWrapper(socket, NioEndpoint.this);
            socket.setSocketWrapper(ka);
            ka.setPoller(this);
            ka.setReadTimeout(getSocketProperties().getSoTimeout());
            ka.setWriteTimeout(getSocketProperties().getSoTimeout());
            ka.setKeepAliveLeft(NioEndpoint.this.getMaxKeepAliveRequests());
            ka.setReadTimeout(getConnectionTimeout());
            ka.setWriteTimeout(getConnectionTimeout());
            PollerEvent r = eventCache.pop();
            ka.interestOps(SelectionKey.OP_READ);//this is what OP_REGISTER turns into.
            if ( r==null) r = new PollerEvent(socket,ka,OP_REGISTER);
            else r.reset(socket,ka,OP_REGISTER);
            addEvent(r);
        }
```



**Poller**

主要功能：

- 监听多个channel，当事件发生时，生成一个processor对象交给线程池执行
- 检查连接是否过期

```java
public class Poller implements Runnable {
				// 实际上就是一个selector
        private Selector selector;
  			// event里面封装了socketchannel 多个acceptor线程可能同时对一个poller的queue进行读取 所以需要同步
        private final SynchronizedQueue<PollerEvent> events =
                new SynchronizedQueue<>();
}
```



```java
public void run() {
    // Loop until destroy() is called
    while (true) {

        boolean hasEvents = false;

        try {
            if (!close) {
              	// events方法是遍历queue，执行event的run方法，run方法中将socketchannel注册到了自己的selector上
                hasEvents = events();
              	// 检查是否有自己感兴趣的事件
                if (wakeupCounter.getAndSet(-1) > 0) {
                    // If we are here, means we have other stuff to do
                    // Do a non blocking select
                    keyCount = selector.selectNow();
                } else {
                    keyCount = selector.select(selectorTimeout);
                }
                wakeupCounter.set(0);
            }
            if (close) {
                events();
                timeout(0, false);
                try {
                    selector.close();
                } catch (IOException ioe) {
                    log.error(sm.getString("endpoint.nio.selectorCloseFail"), ioe);
                }
                break;
            }
        } catch (Throwable x) {
            ExceptionUtils.handleThrowable(x);
            log.error("",x);
            continue;
        }
        // Either we timed out or we woke up, process events first
        if (keyCount == 0) {
            hasEvents = (hasEvents | events());
        }
			
        Iterator<SelectionKey> iterator =
            keyCount > 0 ? selector.selectedKeys().iterator() : null;
        // Walk through the collection of ready keys and dispatch
        // any active event.
        while (iterator != null && iterator.hasNext()) {
            SelectionKey sk = iterator.next();
            iterator.remove();
            NioSocketWrapper socketWrapper = (NioSocketWrapper) sk.attachment();
            // Attachment may be null if another thread has called
            // cancelledKey()
            if (socketWrapper != null) {
              	// 处理IO事件
                processKey(sk, socketWrapper);
            }
        }

        // Process timeouts 检查channel是否过期
        timeout(keyCount,hasEvents);
    }

    getStopLatch().countDown();
}
```



```java
public boolean processSocket(SocketWrapperBase<S> socketWrapper,
        SocketEvent event, boolean dispatch) {
    try {
        if (socketWrapper == null) {
            return false;
        }
      	// 复用对象作为容器 避免GC
        SocketProcessorBase<S> sc = processorCache.pop();
        if (sc == null) {
            sc = createSocketProcessor(socketWrapper, event);
        } else {
            sc.reset(socketWrapper, event);
        }
      	// 生成socketprocessor对象 交给线程池执行
        Executor executor = getExecutor();
        if (dispatch && executor != null) {
            executor.execute(sc);
        } else {
            sc.run();
        }
    } catch (RejectedExecutionException ree) {
        getLog().warn(sm.getString("endpoint.executor.fail", socketWrapper) , ree);
        return false;
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        // This means we got an OOM or similar creating a thread, or that
        // the pool and its queue are full
        getLog().error(sm.getString("endpoint.process.fail"), t);
        return false;
    }
    return true;
}
```



**SocketProcessor**

主要功能：

调用http11processor处理请求，读取channel的数据来生成servletrequest对象



**Executor**

执行socketProcessor的run方法，解析请求并通过容器来处理请求，最终会调用到serlvet



**高并发思路**

- 设计合理的线程模型，不要让线程阻塞，尽量让CPU忙起来

- 有多少任务，就用响应规模的线程去处理

  如接受连接 检测IO事件 处理请求 用三个不同的线程组去处理，且线程的数量是可配置的



**IO模型相关问题**

- 阻塞与非阻塞：应用程序在发起IO操作后，是等待还是立即返回
- 同步与异步：数据从内核空间到应用空间的拷贝，是由内核主动发起还是由应用程序发起





### Nio2Endpoint组件：Tomcat如何实现异步IO

**异步IO的工作模式**

1. 应用程序调用read API的同时，告诉内核数据准备好了之后应该拷贝到的buffer和处理这些数据需要调用的回调函数
2. 内核接收到read调用后，等待网卡数据到达，到达后产生硬件中断，内核在中断程序中把数据从网卡拷贝到内核空间，进行TCP/IP层面的解包和重组，然后把数据拷贝到指定的buffer，最后调用回调函数



**Java nio2实现异步IO**

```java

public class Nio2Server {

   void listen(){
      //1.创建一个线程池 内核只需要把工作交给线程池就返回了
      ExecutorService es = Executors.newCachedThreadPool();

      //2.创建异步通道群组
      AsynchronousChannelGroup tg = AsynchronousChannelGroup.withCachedThreadPool(es, 1);
      
      //3.创建服务端异步通道
      AsynchronousServerSocketChannel assc = AsynchronousServerSocketChannel.open(tg);

      //4.绑定监听端口
      assc.bind(new InetSocketAddress(8080));

      //5. 监听连接，传入回调类处理连接请求
      assc.accept(this, new AcceptHandler()); 
   }
}
```



```java

//AcceptHandler类实现了CompletionHandler接口的completed方法。它还有两个模板参数，第一个是异步通道，第二个就是Nio2Server本身
public class AcceptHandler implements CompletionHandler<AsynchronousSocketChannel, Nio2Server> {

   //具体处理连接请求的就是completed方法，它有两个参数：第一个是异步通道，第二个就是上面传入的NioServer对象
   @Override
   public void completed(AsynchronousSocketChannel asc, Nio2Server attachment) {      
      //调用accept方法继续接收其他客户端的请求
      attachment.assc.accept(attachment, this);
      
      //1. 先分配好Buffer，告诉内核，数据拷贝到哪里去
      ByteBuffer buf = ByteBuffer.allocate(1024);
      
      //2. 调用read函数读取数据，除了把buf作为参数传入，还传入读回调类
      channel.read(buf, buf, new ReadHandler(asc)); 

}
```



```java
// V：IO调用的返回值 A：附件类
public interface CompletionHandler<V,A> {

    void completed(V result, A attachment);

    void failed(Throwable exc, A attachment);
}
```



```java

public class ReadHandler implements CompletionHandler<Integer, ByteBuffer> {   
    //读取到消息后的处理  
    @Override  
    public void completed(Integer result, ByteBuffer attachment) {  
        //attachment就是数据，调用flip操作，其实就是把读的位置移动最前面
        attachment.flip();  
        //读取数据
        ... 
    }  

    void failed(Throwable exc, A attachment){
        ...
    }
}
```



Nio2Endpoint工作步骤

1. limitlatch控制最大连接数
2. Nio2acceptor跑在一个单独的线程中，也是一个线程组，获取连接后得到一个asynchronoussocketchannel，acceptor把这个channel封装成一个nio2socketchannelwrapper，并创建一个socketprocessor任务类给线程池处理
3. executor在执行任务类的时候，socketprocessor的run方法会调用http11processor处理请求，解析数据，读写都是通过niosocketwrapper

> 异步IO模式下，selector的工作交给内核做了



**Nio2Acceptor**

监听连接的过程不是在一个死循环里面调用accept方法，而是通过回调函数完成的

```java
// 回调类是acceptor本身
serverSock.accept(null, this);
```



```java
// 返回结果是异步的socketchannel
protected class Nio2Acceptor extends Acceptor<AsynchronousSocketChannel>
    implements CompletionHandler<AsynchronousSocketChannel, Void> {
    
@Override
public void completed(AsynchronousSocketChannel socket,
        Void attachment) {
        // 运行到这里说内核已经把数据拷贝到用户空间了
  // 所以继续调用accept方法监听下一个请求
    if (isRunning() && !isPaused()) {
        if (getMaxConnections() == -1) {
            //如果没有连接限制，继续接收新的连接
            serverSock.accept(null, this);
        } else {
            //如果有连接限制，就在线程池里跑run方法，run方法会检查连接数
            getExecutor().execute(this);
        }
        //处理请求
        if (!setSocketOptions(socket)) {
            closeSocket(socket);
        }
    } 
}
```





**Nio2SocketWrapper**

主要作用：

- 封装channel
- 提供接口给http11processor进行读写



Http11processor对nio2socketwrapper进行了两次的read调用

1. 第一个read调用 注册回调类readCompletionHandler read方法立即返回
2. 第二次read调用 数据到达应用层中的buffer后，回调类被调用，在回调方法里面会创建一个socketprocessor处理请求，而nio2socketwrapper也可以重新拿到（作为回调方法中的附带类）

```java

this.readCompletionHandler = new CompletionHandler<Integer, SocketWrapperBase<Nio2Channel>>() {
    public void completed(Integer nBytes, SocketWrapperBase<Nio2Channel> attachment) {
        ...
        //通过附件类SocketWrapper拿到所有的上下文
        Nio2SocketWrapper.this.getEndpoint().processSocket(attachment, SocketEvent.OPEN_READ, false);
    }

    public void failed(Throwable exc, SocketWrapperBase<Nio2Channel> attachment) {
        ...
    }
}
```





### AprEndpoint：Tomcat APR提高IO性能的秘密

APR: *apache portable runtime libraries* 用C语言实现的，与操作系统和网络交互方面会比较快



JNI：*java native interface* java调用其他语言编写的程序或者库



工作过程：

![](https://static001.geekbang.org/resource/image/37/93/37117f9fd6ed5523a331ac566a906893.jpg)



与NioEndpoint的区别在Acceptor和Poller的实现



**Acceptor**

acceptor的功能：监听连接，接收并建立连接

本质就是调用了操作系统的API：

- socket
- bind
- listen
- accept



Java通过JNI调用C语言的API

- 封装一个Java类，在里面定义native方法
- 用C实现这些方法



**Poller**

通过JNI调用APR中的poll方法，其中又调用了操作系统的epoll API来实现（检查这个socket的IO事件）

AprEndpoint中的一个参数：`deferAccept`，对应TCP协议中的`TCP_DEFER_ACCEPT`

设置该参数后，TCP客户端有新的连接到达时，TCP服务端先不建立连接，而是等到客户端有请求数据到达后才建立连接，这样就省去了服务端不断用selector查询数据是否准备就绪的环节



**JVM堆 VS 本地内存**

C代码运行过程中用到的内存：本地内存

![](https://static001.geekbang.org/resource/image/83/80/839bfab2636634d47477cbd0920b5980.jpg)



```java

//分配HeapByteBuffer
ByteBuffer buf = ByteBuffer.allocate(1024);

//分配DirectByteBuffer
ByteBuffer buf = ByteBuffer.allocateDirect(1024);

//将buf作为read函数的参数
int bytesRead = socketChannel.read(buf);
```



**HeapByteBuffer 与 DirectByteBuffer的区别**：以接收网络数据为例

从内核中读取数据时，使用Headbytebuffer需要把数据先拷贝到本地内存，再从本地内存拷贝到堆

使用directbytebuffer则只需要把数据拷贝到本地内存，堆中持有一个本地内存地址即可（但是本地内存难以管理）



**sendfile**：浏览器通过Tomcat获取一个Html文件

通过sendfile特性可以避免内核与应用之间的内存拷贝以及用户态和内核态之间的切换

使用HeapByteBuffer的情况

![](https://static001.geekbang.org/resource/image/2b/0e/2b902479c36647142ccd413320b3900e.jpg)



AprEndpoint直接调用操作系统的sendfile API

![](https://static001.geekbang.org/resource/image/19/00/193df268fccb59a09195810e34080a00.jpg)



### Executor



Java线程池

**ThreadPoolExecutor**

```java

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

在提交任务到线程池的时候：

- 如果当前线程数 < 核心线程数，就创建一个新的线程执行该任务

- 如果当前线程数 = 核心线程数，将任务放到任务队列中（线程会不停poll，从任务队列中取任务来执行）

- 如果任务队列已满 `queue.offer()`返回false，如果当前线程数 < 最大线程数，则创建临时线程执行该任务（临时线程有存活时间 到了就会被销毁 临时线程取任务的方法是`poll(keepAliveTime,unit)`）

  否则就执行拒绝策略



**Tomcat线程池**

创建一个线程池需要考虑的问题：

- 是否限制线程数量个数
- 是否队列长度



```java
// AbstractEndpoint中的方法 供子类调用
public void createExecutor() {
    internalExecutor = true;
  	// 自己的任务队列
    TaskQueue taskqueue = new TaskQueue();
  	// 自己的线程工厂
    TaskThreadFactory tf = new TaskThreadFactory(getName() + "-exec-", daemon, getThreadPriority());
  	// 自己的线程池
    executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), 60, TimeUnit.SECONDS,taskqueue, tf);
    taskqueue.setParent( (ThreadPoolExecutor) executor);
}
```



Tomcat线程池做的改进

在临时线程时也满的时候，不立即执行拒绝策略，而是等待一段时间后，再次尝试把任务放到队列中（因为等待的这段时间内，可以队列中就有任务被取走了 空了位置出来）

```java

public class ThreadPoolExecutor extends java.util.concurrent.ThreadPoolExecutor {
  
  ...
  
  public void execute(Runnable command, long timeout, TimeUnit unit) {
    	// 已经提交但没有执行完的任务个数
      submittedCount.incrementAndGet();
      try {
          //调用Java原生线程池的execute去执行任务
          super.execute(command);
      } catch (RejectedExecutionException rx) {
         //如果总线程数达到maximumPoolSize，Java原生线程池执行拒绝策略
          if (super.getQueue() instanceof TaskQueue) {
              final TaskQueue queue = (TaskQueue)super.getQueue();
              try {
                  //继续尝试把任务放到任务队列中去
                  if (!queue.force(command, timeout, unit)) {
                      submittedCount.decrementAndGet();
                      //如果缓冲队列也满了，插入失败，执行拒绝策略。
                      throw new RejectedExecutionException("...");
                  }
              } 
          }
      }
}
```



Tomcat对任务队列做的改进：

offer方法返回false表示加入队列失败

改进目的：在任务队列长度没有限制的情况下，让线程池有机会可以创建新的线程

```java

public class TaskQueue extends LinkedBlockingQueue<Runnable> {

  ...
   @Override
  //线程池调用任务队列的方法时，当前线程数肯定已经大于核心线程数了
  public boolean offer(Runnable o) {

      //如果线程数已经到了最大值，不能创建新线程了，只能把任务添加到任务队列。
      if (parent.getPoolSize() == parent.getMaximumPoolSize()) 
          return super.offer(o);
          
      //执行到这里，表明当前线程数大于核心线程数，并且小于最大线程数。
      //表明是可以创建新线程的，那到底要不要创建呢？分两种情况：
      
      //1. 如果已提交的任务数小于当前线程数，表示还有空闲线程，无需创建新线程
      if (parent.getSubmittedCount()<=(parent.getPoolSize())) 
          return super.offer(o);
          
      //2. 如果已提交的任务数大于当前线程数，线程不够用了，返回false去创建新线程
      if (parent.getPoolSize()<parent.getMaximumPoolSize()) 
          return false;
          
      //默认情况下总是把任务添加到任务队列
      return super.offer(o);
  }
  
}
```



### Tomcat如何支持WebSocket



**Socket**: IP + PORT

是对TCP/IP协议抽象出来的API



**WebSocket工作原理**

Websocket是一个应用层协议，通过HTTP完成握手，之后的数据传输都是通过TCP

握手：

- HTTP请求：加上两个请求头

  `Connection:Updagrade`

  `Upgrade: websocket`

- HTTP响应也加上这两个响应头

数据传输：以frame的形式传输，会将一个消息分为多个frame



服务端实现代码：

```java

//Tomcat端的实现类加上@ServerEndpoint注解，里面的value是URL路径
@ServerEndpoint(value = "/websocket/chat")
public class ChatEndpoint {

    private static final String GUEST_PREFIX = "Guest";
    
    //记录当前有多少个用户加入到了聊天室，它是static全局变量。为了多线程安全使用原子变量AtomicInteger
    private static final AtomicInteger connectionIds = new AtomicInteger(0);
    
    //每个用户用一个ChatEndpoint实例来维护，请你注意它是一个全局的static变量，所以用到了线程安全的CopyOnWriteArraySet
    private static final Set<ChatEndpoint> connections =
            new CopyOnWriteArraySet<>();

    private final String nickname;
    private Session session;

    public ChatEndpoint() {
        nickname = GUEST_PREFIX + connectionIds.getAndIncrement();
    }

    //新连接到达时，Tomcat会创建一个Session，并回调这个函数
    @OnOpen
    public void start(Session session) {
        this.session = session;
        connections.add(this);
        String message = String.format("* %s %s", nickname, "has joined.");
        broadcast(message);
    }

   //浏览器关闭连接时，Tomcat会回调这个函数
    @OnClose
    public void end() {
        connections.remove(this);
        String message = String.format("* %s %s",
                nickname, "has disconnected.");
        broadcast(message);
    }

    //浏览器发送消息到服务器时，Tomcat会回调这个函数
    @OnMessage
    public void incoming(String message) {
        // Never trust the client
        String filteredMessage = String.format("%s: %s",
                nickname, HTMLFilter.filter(message.toString()));
        broadcast(filteredMessage);
    }

    //WebSocket连接出错时，Tomcat会回调这个函数
    @OnError
    public void onError(Throwable t) throws Throwable {
        log.error("Chat Error: " + t.toString(), t);
    }

    //向聊天室中的每个用户广播消息
    private static void broadcast(String msg) {
        for (ChatEndpoint client : connections) {
            try {
                synchronized (client) {
                    client.session.getBasicRemote().sendText(msg);
                }
            } catch (IOException e) {
              ...
            }
        }
    }
}
```



Tomcat主要做的两件事：

- Endpoint加载
- WebSocket请求处理



**Endpoint加载**

通过SCI(监听Servlet容器的启动事件，从而执行一些初始化工作)

```java
@HandlesTypes({ServerEndpoint.class, ServerApplicationConfig.class, Endpoint.class})
public class WsSci implements ServletContainerInitializer {
  
  public void onStartup(Set<Class<?>> clazzes, ServletContext ctx) throws ServletException {
  ...
  }
}
```

> Tomcat在启动阶段扫描类的时候，会将HandlersTypes注解中指定的类都扫描出来，作为`onStartup`的参数
>
> 在这个方法中构造一个WebsocketContainer实例，将类注册到这个容器中，并维护URL -> Endpoint的映射关系



**Websocket请求处理**

Tomcat先判断有没有ws的请求头，如果有就进行协议升级：

- Upgradeprotocolhandler替换当前的httpprotocolhandler
- upgradeprocessor替换processor
- 创建websocket Session实例和endpoint实例与当前websocket连接一一对应，此websocket连接不会立即关闭



