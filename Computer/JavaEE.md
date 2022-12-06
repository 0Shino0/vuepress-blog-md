---
title: JavaEE
tags: 
  - JavaEE
categories: 
  - Computer
summary: JavaEE期末复习
description: JavaEE期末复习
date: 2022-11-18 15:18:34
# sticky: 2
autoGroup-2: 计算机底层
subSidebar: auto
---



## JavaEE



简答题

1. 请简要解释HTML、JavaEE、JSP、JDBC的英文含义和中文含义。

   - `HTML`:文本标记语言，即HTML（`Hypertext Markup Language`），是用于描述网页文档的一种标记语言。
   - `JavaEE`: Java EE，Java 平台企业版（`Java Platform Enterprise Edition`），之前称为 `Java 2 Platform, Enterprise Edition (J2EE)`，2018年3月更名为 Jakarta EE
   - `JSP`: （全称 `Java Server Pages`）是由Sun Microsystems公司倡导和许多公司参与共同创建的一种使软件开发者可以响应客户端请求，而动态生成HTML、XML或其他格式文档的Web网页的技术标准。JSP可以嵌套在html中。 简单地说就是java服务器端页面，控制各种页面的跳转和数据的输入输出。
   - `JDBC`: JDBC英文名为：`Java Data Base Connectivity(Java数据库连接) `，官方解释它是Java编程语言和广泛的数据库之间独立于数据库的连接标准的Java [API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)，根本上说JDBC是一种规范，它提供的接口，一套完整的，允许便捷式访问底层数据库。

2. 请简要叙述Servlet和JSP在软件开发时的作用与地位。

   1. JSP文件必须在JSP服务器内运行。

   2. JSP文件必须生成Servlet才能执行。

   3. 每个JSP页面的第一个访问者速度很慢，因为必须等待JSP编译成Servlet。

   4. JSP页面的访问者无须安装任何客户端，甚至不需要可以运行Java的运行环境，因为JSP页面输送到客户端的是标准HTML页面。

      ```text
      JSP和Servlet会有如下转换：
      1.JSP页面的静态内容、JSP脚本都会转换成Servlet的xxxService()方法，类似于自行创建Servlet时service()方法。
      2.JSP声明部分，转换成Servlet的成员部分。所有JSP声明部分可以使用private,protected,public,static等修饰符，其他地方则不行。
      3.JSP的输出表达式(<%= ..%>部分)，输出表达式会转换成Servlet的xxxService()方法里的输出语句。
      4.九个内置对象要么是xxxService()方法的形参，要么是该方法的局部变量，所以九个内置对象只能在JSP脚本和输出表达式中使用。
      ```

      

3. 请简要说明JSP页面第一次运行时的过程。

   1. 客户端发出请求
   2. web容器将jsp转化为servlet代码（.java）
   3. web容器将转化为servlet代码编译（.class）
   4. web容器加载编译后的代码并执行
   5. 将执行结果响应给客户端

   

4. 请简要叙述MVC设计模式，JSP、JavaBean和Servlet在MVC设计模式中的作用。

   - **MVC模式（Model-View-Controller），是系统分为三个基本部分：模型（Model）、视图（View）和控制器(Controller)：可以理解为：JSP充当视图，Servlet充当控制器，JavaBeans充当模型。**
     - View层（JSP）,前台交互,比如我们注册时的数据等等,serlvet就是与前台数据进行交互的
     - Contrller层（servlet充当）：Model与View之间沟通的桥梁， 这个层有业务处理,用户的注册登录就可以看做是User的业务,我们就需要将相关的处理代码写到这个层中。
     - Model层：实现系统的业务逻辑，即javaBean，常见的就是封装对象的属性、数据库连接操作等。
       常规会写一个dao层，是属于mvc里面Model层抽出来。目的就是更单纯的和数据库打交道,将servlet的数据和数据库进行交互。

   - Servlet+JSP+JavaBean。模式(MVC)适合开发复杂的web应用，在这种模式下，servlet负责处理用户请求，jsp负责数据显示，javabean负责封装数据。 Servlet+JSP+JavaBean模式程序各个模块之间层次清晰，web开发推荐采用此种模式。

5. 请简要叙述include指令和include动作的功能，以及它们的区别。

   - `include指令`指的是jsp的一种指令标记

     ```jsp
     <%@include file="文件的URL">
     ```

     

   - `include动作`指的是jsp的一种动作标记

     ```jsp
     <jsp:include page="文件的URL"/>
     ```

   - 区别

     - `include指令`执行时将被导入页面的jsp代码完全融入,两个页面融合成一个Servlet;而`include动作`则在Servlet中使用include方法来引入被导入页面的内容。因此`include指令`执行时不需编译,速度快;`include动作`需要加载执行,速度慢。
     - `include指令`执行时导入页面的编译指令会起作用;而`include动作`执行时被导入页面的编译指令则失去作用,只是插入被导入页面的body内容。
     - `include动作`还可以用param动作来为被导入页面传递参数。
     - `include指令`通过file属性指定被包含的文件，放在页面的顶部，file属性不支持任何的表达式；`include动作`是通过page属性来指定被包含的文件的，page属性支持jsp表达式。

6. 请简要叙述JavaBean的四种生存范围。

   - `page`：page范围的JavaBean只在本页有效，跳转后无效。

     ```jsp
     <jsp:useBean id="属性名" scope="范围" class="类对应的可执行文件的包路径名"/>
     ```

     

   - `request`: 客户端跳转无效，因为发送了两次请求。服务器跳转有效，只相服务器发送了一次请求，只调用了一次构造函数。

   - `application`: 客户端和服务器端跳转都有效，但是只会调用一次构造函数。这个范围是所有用户共同拥有的，只要申明后就会在服务器中保存，除非关闭了服务器。

   - `session`: 都有效，只调用一次构造函数。

7. 请简要描述Servlet的基本工作流程。

   1. Web客户向Servlet容器（tomcat）发出Http请求；
   2. Servlet容器解析Web客户的Http请求；
   3. Servlet容器创建一个HttpRequest对象，在这个对象中封装Http请求信息；
   4. Servlet容器创建一个HttpResponse对象；
   5. Servlet容器调用HttpServlet的service方法，把HttpRequest和HttpResponse对象作为service方法的参数传给HttpServlet对象；
      - *HttpServlet事实上是servlet的一种子类实例也是最一般的实例。当编写一个servlet时,必须直接或间接实现servlet接口,最可能实现的方法就是扩展javax.servlet.genericservlet或javax.servlet.http.httpservlet，其中genericservlet类提供了servlet接口的基本实现,httpservlet类扩展了genericservlet并且提供了servlet接口中具体于http的实现。
   6. HttpServlet调用HttpRequest的有关方法，获取HTTP请求信息；
   7. HttpServlet调用HttpResponse的有关方法，生成响应数据；
      - 一般通过HttpServletRequest和HttpServletResponse获取HTTP请求信息和返回响应。事实上servlet理论上可以处理多种形式的请求响应形式 http只是其中之一 所以HttpServletRequest HttpServletResponse分别是ServletRequest和ServletResponse的子类。一般，HttpServlet对应HttpServletRequest和HttpServletResponse。
   8. Servlet容器把HttpServlet的响应结果传给Web客户。

   

   

8. 请简要说明JDBC访问数据库的基本步骤。

   1. 加载（注册）数据库驱动（到JVM）。
   2. 建立（获取）数据库连接。
   3. 创建（获取）数据库操作对象。
   4. 定义操作的SQL语句。
   5. 执行数据库操作。
   6. 获取并操作结果集。
   7. 关闭对象，回收数据库资源（关闭结果集-->关闭数据库操作对象-->关闭连接）。
   8. 

9. 请简要叙述利用JDBC访问数据库时常用的类和接口有哪些。

   1. `DriverManager类` : DriverManager类是JDBC的管理层，用来管理数据库中的驱动程序，在使用java操作数据库之前，必须使用class类的静态方法forName（String classname）加载能够连接数据库的驱动程序。
   2. `Connection接口`: Connection接口代表java端与指定数据库之间的连接Connection接口的
   3. `Statement接口`: Statement接口时用来执行静态SQL语句的工具接口
   4. `PreParedStatement接口`: PreParedStatement接口是Statement接口的子接口，是用来执行动态SQL语句的工具接口
   5. `ResultSet接口`: ResultSet接口类似于一个临时表，用来暂时存放对数据库中的数据执行查询操作后的结果，ResultSet对象具有指向当前数据行的指针，指针开始的位置在第一条记录上，通过next()方法可向下移动指针

10. 请简要解释下列JSP内置对象的主要用途：request、response、session、application。

    1. `request 对象`是javax.[servlet](https://so.csdn.net/so/search?q=servlet&spm=1001.2101.3001.7020).http.HttpServletRequest类型的对象，代表客户端的请求信息，主要用于获取客户端的参数和流。
    2. `response 对象`和request是一对相应的内置对象，代表对客户端的响应
    3. `session 对象`是由服务器自动创建的与请求相关的对象，服务器为每个用户都生成一个session对象，用于保存该用户的信息，跟踪用户的操作状态。session内部使用Map来保存数据，即key-value对
    4. `application 对象`是javax.servlet.ServletContext类型的对象，可将信息保存在服务器中，直到服务器关闭，否则application对象中保存的信息会整个应用中都有。





材料分析题考点（填空）

1. 表单

   ```html
   <!DOCTYPE html>
   <html>
   <head>
   <meta charset="UTF-8">
   <title>Insert title here</title>
   </head>
   <body>
   <form>
   学号:<input type="text"name="no"><br/><br/>
   姓名：<input type="text"name="name"><br/><br/>
   班级：<input type="text"name="class"><br/><br/>
   专业：<input type="text"name="major"><br/><br/>
   政治面貌：<input type="text"name="zhengzhimianmao"><br/><br/>
   爱好：<input type="text"name="no"><br/><br/>
   性别：<input type="radio"name="sex"value="1">男<input type="radio"name="sex"value="0">女
   出生日期：
   <select name="年">
   <option>1997</option>
   <option>1998</option>
   <option>1999</option>
   <option>2000</option>
   <option>2001</option>
   <option>2002</option>
   <option selected>2003</option>
   <option>2004</option>
   <option>2005</option>
   <option>2006</option>
   <option>2007</option>
   <option>2008</option>
   </select>年
   <select name="月">
   <option>1</option>
   <option>2</option>
   <option>3</option>
   <option>4</option>
   <option>5</option>
   <option>6</option>
   <option selected>7</option>
   <option>8</option>
   <option>9</option>
   <option>10</option>
   <option>11</option>
   <option>12</option>
   </select>月
   <select name="日">
   <option selected>1</option>
   <option>2</option>
   <option>3</option>
   <option>4</option>
   <option>5</option>
   <option>6</option>
   <option>7</option>
   <option>8</option>
   <option>9</option>
   <option>10</option>
   <option>11</option>
   <option>12</option>
   <option>13</option>
   <option>14</option>
   <option>15</option>
   <option>16</option>
   <option>17</option>
   <option>18</option>
   <option>19</option>
   <option>20</option>
   <option selected>21</option>
   <option>22</option>
   <option>23</option>
   <option>24</option>
   <option>25</option>
   <option>26</option>
   <option>27</option>
   <option>28</option>
   <option>29</option>
   <option>30</option>
   <option>31</option>
   </select>日<br/><br/>
   <input type="submit"value="提交"/>
   <input type="reset"value="重置"/>
   <br/><br/>
   
   
   </form>
   
   </body>
   </html>
   
   ```

   

2. JSP脚本

   ```jsp
   // jsp 脚本
   ```

   

3. request对象、session对象、application对象

4. JavaBean及在JSP中使用JavaBean

5. 基于JDBC访问数据库

6. Servlet



编程题

1. 编写JavaBean

   ```
   // javaBean
   
   ```

   

2. JSP脚本





## 参考

- [Jsp+JavaBean模式，Jsp+Servlet模式，MVC模式介绍_江湖一点雨的博客-CSDN博客](https://blog.csdn.net/ITBigGod/article/details/86515893)
- [include指令和include动作的区别_凯哥多帅哦的博客-CSDN博客](https://blog.csdn.net/qq_31957747/article/details/53750386)
- [JavaBean的四种范围 - 简书 (jianshu.com)](https://www.jianshu.com/p/30617da21516)
- [JSP——JavaBean的四种生命周期_明子～的博客-CSDN博客](https://blog.csdn.net/lmm0513/article/details/89599754)
- [(27条消息) servlet工作流程_Rosso_的博客-CSDN博客](https://blog.csdn.net/yuyibo95/article/details/88950951)
- [(27条消息) javaJDBC中常用的类和接口_程程呀的博客-CSDN博客](https://blog.csdn.net/qq_31755183/article/details/103727431)