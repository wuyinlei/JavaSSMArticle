# Struts 2详细工作流程

标签（空格分隔）： JAVA后台

---

转载

##### 读者如果曾经学习过Struts1.x或者有过Struts1.x的开发经验，那么千万不要想当然地以为这一章可以跳过。实际上Struts1.x与Struts2并无我们想象的血缘关系。虽然Struts2的开发小组极力保留Struts1.x的习惯，但因为Struts2的核心设计完全改变，从思想到设计到工作流程，都有了很大的不同。
##### Struts2是Struts社区和WebWork社区的共同成果，我们甚至可以说，Struts2是WebWork的升级版，他采用的正是WebWork的核心，所以，Struts2并不是一个不成熟的产品，相反，构建在WebWork基础之上的Struts2是一个运行稳定、性能优异、设计成熟的WEB框架。
##### 本章主要对Struts的源代码进行分析，因为Struts2与WebWork的关系如此密不可分，因此，读者需要下载xwork的源代码，访问http://www.opensymphony.com/xwork/download.action即可自行下载。
##### 下载的Struts2源代码文件是一个名叫struts-2.1.0-src.zip的压缩包，里面的目录和文件非常多，读者可以定位到struts-2.1.0-src"struts-2.0.10"src"core"src"main"java目录下查看Struts2的源文件，如图14所示。
![http://www.blogjava.net/images/blogjava_net/lzhidj/14.PNG][1]

### 主要的包和类
* Struts2框架的正常运行，除了占核心地位的xwork的支持以外，Struts2本身也提供了许多类，这些类被分门别类组织到不同的包中。从源代码中发现，基本上每一个Struts2类都访问了WebWork提供的功能，从而也可以看出Struts2与WebWork千丝万缕的联系。但无论如何，Struts2的核心功能比如将请求委托给哪个Action处理都是由xwork完成的，Struts2只是在WebWork的基础上做了适当的简化、加强和封装，并少量保留Struts1.x中的习惯。
* 以下是对各包的简要说明：

|包名| 说明|
|:---:|:----:|
|org.apache.struts2. components | 该包封装视图组件，Struts2在视图组件上有了很大加强，不仅增加了组件的属性个数，更新增了几个非常有用的组件，如updownselect、doubleselect、datetimepicker、token、tree等另外，Struts2可视化视图组件开始支持主题(theme)，缺省情况下，使用自带的缺省主题，如果要自定义页面效果，需要将组件的theme属性设置为simple。|
|org.apache.struts2. config |  该包定义与配置相关的接口和类。实际上，工程中的xml和properties文件的读取和解析都是由WebWork完成的，Struts只做了少量的工作。|
|org.apache.struts2.dispatcher |  Struts2的核心包，最重要的类都放在该包中。|
|org.apache.struts2.impl |  该包只定义了3个类，他们是StrutsActionProxy、StrutsActionProxyFactory、StrutsObjectFactory，这三个类都是对xwork的扩展。|
|org.apache.struts2.interceptor |  定义内置的截拦器|
|org.apache.struts2.util |  实用包。|
|org.apache.struts2.util |  只定义了一个类：DWRValidator|
|org.apache.struts2.views |  提供freemarker、jsp、velocity等不同类型的页面呈现。|

#### 下表是对一些重要类的说明：
| 类名 | 说明 | 
| :---: | :----: |
| org.apache.struts2.dispatcher. Dispatcher | 该类有两个作用：1、初始化  2、调用指定的Action的execute()方法。 | 
| org.apache.struts2.dispatcher. FilterDispatcher  |这是一个过滤器。文档中已明确说明，如果没有经验，配置时请将url-pattern的值设成/*。该类有四个作用：1、执行Actio2、清理ActionContext，避免内存泄漏3、处理静态内容（Serving static content）4、为请求启动xwork’s的截拦器链。      | 
| com.opensymphony.xwork2. ActionProxy  | Action的代理接口。     | 
| com.opensymphony.xwork2. ctionProxyFactory   | 生产ActionProxy的工厂。      | 
| com.opensymphony.xwork2.ActionInvocation    | 负责调用Action和截拦器。      | 
| com.opensymphony.xwork2.config.providers. XmlConfigurationProvider   | 负责Struts2的配置文件的解析。      | 

## 三、Struts2的工作机制
#### 3.1Struts2体系结构图
 Strut2的体系结构如图15所示：
 ![Strut2的体系结构][2]
 #### 3.2Struts2的工作机制
 从图15可以看出，一个请求在Struts2框架中的处理大概分为以下几个步骤：
 
* 1、客户端初始化一个指向Servlet容器（例如Tomcat）的请求；
* 2、这个请求经过一系列的过滤器（Filter）（这些过滤器中有一个叫做ActionContextCleanUp的可选过滤器，这个过滤器对于Struts2和其他框架的集成很有帮助，例如：SiteMesh Plugin）；
* 3、接着FilterDispatcher被调用，FilterDispatcher询问ActionMapper来决定这个请求是否需要调用某个Action；
* 4、如果ActionMapper决定需要调用某个Action，FilterDispatcher把请求的处理交给ActionProxy；
* 5、ActionProxy通过Configuration Manager询问框架的配置文件，找到需要调用的Action类；
* 6、ActionProxy创建一个ActionInvocation的实例。
* 7、ActionInvocation实例使用命名模式来调用，在调用Action的过程前后，涉及到相关拦截器（Intercepter）的调用。
* 8、一旦Action执行完毕，ActionInvocation负责根据struts.xml中的配置找到对应的返回结果。返回结果通常是（但不总是，也可能是另外的一个Action链）一个需要被表示的JSP或者FreeMarker的模版。在表示的过程中可以使用Struts2框架中继承的标签。在这个过程中需要涉及到ActionMapper。
***注：以上步骤参考至网上，具体网址已忘记。在此表示感***谢！

#### 3.3Struts2源代码分析
>和Struts1.x不同，Struts2的启动是通过FilterDispatcher过滤器实现的。下面是该过滤器在web.xml文件中的配置：

代码清单6：web.xml（截取）
```
 <filter>
      <filter-name>struts2</filter-name>
      <filter-class>
           org.apache.struts2.dispatcher.FilterDispatcher
      </filter-class>
   </filter>
   <filter-mapping>
      <filter-name>struts2</filter-name>
      <url-pattern>/*</url-pattern>
   </filter-mapping>
```
 Struts2建议，在对Struts2的配置尚不熟悉的情况下，将url-pattern配置为/*，这样该过滤器将截拦所有请求。
 > 实际上，FilterDispatcher除了实现Filter接口以外，还实现了StrutsStatics接口，继承代码如下：
 
 代码清单7：FilterDispatcher结构
 
```
publicclassFilterDispatcherimplementsStrutsStatics,Filter {
}
```
>StrutsStatics并没有定义业务方法，只定义了若干个常量。Struts2对常用的接口进行了重新封装，比如HttpServletRequest、HttpServletResponse、HttpServletContext等。　以下是StrutsStatics的定义：

代码清单8：StrutsStatics.Java
```
publicinterfaceStrutsStatics {
   /**
    *ConstantfortheHTTPrequestobject.
    */
   publicstaticfinalStringHTTP_REQUEST="com.opensymphony.xwork2.dispatcher.HttpServletRequest";
   /**
    *ConstantfortheHTTPresponseobject.
    */
   publicstaticfinalStringHTTP_RESPONSE="com.opensymphony.xwork2.dispatcher.HttpServletResponse";
   /**
    *ConstantforanHTTPrequest dispatcher}.
    */
   publicstaticfinalStringSERVLET_DISPATCHER="com.opensymphony.xwork2.dispatcher.ServletDispatcher";
   /**
    *Constantfortheservlet context}object.
    */
   publicstaticfinalStringSERVLET_CONTEXT="com.opensymphony.xwork2.dispatcher.ServletContext";
   /**
    *ConstantfortheJSPpage context}.
    */
publicstaticfinalStringPAGE_CONTEXT="com.opensymphony.xwork2.dispatcher.PageContext";
   /**ConstantforthePortletContextobject*/
   publicstaticfinalStringSTRUTS_PORTLET_CONTEXT="struts.portlet.context";
}
```
>容器启动后，FilterDispatcher被实例化，调用init(FilterConfig filterConfig)方法。该方法创建Dispatcher类的对象，并且将FilterDispatcher配置的初始化参数传到对象中（详情请参考代码清单10），并负责Action的执行。然后得到参数packages，值得注意的是，还有另外三个固定的包和该参数进行拼接，分别是org.apache.struts2.static、template、和org.apache.struts2.interceptor.debugging，中间用空格隔开，经过解析将包名变成路径后存储到一个名叫pathPrefixes的数组中，这些目录中的文件会被自动搜寻。

代码清单9：FilterDispatcher.init()方法
```
publicvoidinit(FilterConfig filterConfig)throwsServletException {
       this.filterConfig = filterConfig;      
       dispatcher = createDispatcher(filterConfig);
       dispatcher.init();      
       String param = filterConfig.getInitParameter("packages");
       String packages ="org.apache.struts2.static template org.apache.struts2.interceptor.debugging";
       if(param !=null) {
            packages = param +" "+ packages;
       }
       this.pathPrefixes= parse(packages);
}
```
代码清单10：FilterDispatcher.createDispatcher()方法
```
 protectedDispatcher createDispatcher(FilterConfig filterConfig) {
       Map<String,String> params =newHashMap<String,String>();
       for(Enumeration e = filterConfig.getInitParameterNames(); e.hasMoreElements(); ) {
           String name = (String) e.nextElement();
           String value = filterConfig.getInitParameter(name);
           params.put(name, value);
       }
       returnnewDispatcher(filterConfig.getServletContext(), params);
   }
```
>当用户向Struts2发送请求时，FilterDispatcher的doFilter()方法自动调用，这个方法非常关键。首先，Struts2对请求对象进行重新包装，此次包装根据请求内容的类型不同，返回不同的对象，如果为multipart/form-data类型，则返回MultiPartRequestWrapper类型的对象，该对象服务于文件上传，否则返回StrutsRequestWrapper类型的对象，MultiPartRequestWrapper是StrutsRequestWrapper的子类，而这两个类都是HttpServletRequest接口的实现。包装请求对象如代码清单11所示

 Struts 2框架本身大致可以分为3个部分：核心控制器FilterDispatcher、业务控制器Action和用户实现的企业业务逻辑组件。
#### 3.1.1  核心控制器FilterDispatcher
核心控制器FilterDispatcher是Struts 2框架的基础，包含了框架内部的控制流程和处理机制。业务控制器Action和业务逻辑组件是需要用户来自己实现的。用户在开发Action和业务逻辑组件的同时，还需要编写相关的配置文件，供核心控制器FilterDispatcher来使用。
Struts 2的工作流程相对于Struts 1要简单，与WebWork框架基本相同，所以说Struts 2是WebWork的升级版本。Struts 2框架按照模块来划分，可以分为Servlet Filters、Struts核心模块、拦截器和用户实现部分。Struts 2框架结构图如图3.1所示。
![Struts 2框架结构图][3]

一个请求在Struts 2框架中的处理大概分为以下几个步骤。

* 客户端提交一个（HttpServletRequest）请求，如上文在浏览器中输入http://localhost: 8080/bookcode/ch2/Reg.action就是提交一个（HttpServletRequest）请求。
* 请求被提交到一系列（主要是3层）的过滤器（Filter），如（ActionContextCleanUp、其他过滤器（SiteMesh等）、FilterDispatcher）。注意：这里是有顺序的，先ActionContext CleanUp，再其他过滤器（Othter Filters、SiteMesh等），最后到FilterDispatcher。
* FilterDispatcher是控制器的核心，就是MVC的Struts 2实现中控制层（Controller）的核心。
    * FilterDispatcher询问ActionMapper是否需要调用某个Action来处理这个（HttpServlet Request）请求，如果ActionMapper决定需要调用某个Action，FilterDispatcher则把请求的处理交给ActionProxy。
    * ActionProxy通过Configuration Manager（struts.xml）询问框架的配置文件，找到需要调用的Action类。例如，用户注册示例将找到UserReg类。
    * ActionProxy创建一个ActionInvocation实例，同时ActionInvocation通过代理模式调用Action。但在调用之前，ActionInvocation会根据配置加载Action相关的所有Interceptor（拦截器）。
    * 一旦Action执行完毕，ActionInvocation负责根据struts.xml中的配置找到对应的返回结果result。
Struts 2的核心控制器是FilterDispatcher，有3个重要的方法：destroy()、doFilter()和Init()，可以在Struts 2的下载文件夹中找到源代码，如代码3.1所示。

代码3.1  核心控制器FilterDispatcher
 ```
 
     public class FilterDispatcher implements StrutsStatics, Filter {

    /**
     * 定义一个Log实例
     */

    private static final Log LOG = LogFactory.getLog(FilterDispatcher.class);

    /**

     * 存放属性文件中的.STRUTS_I18N_ENCODING值

     */

    private static String encoding;

    /**

     * 定义ActionMapper实例

     */

    private static ActionMapper actionMapper;

    /**

     * 定义FilterConfig实例

     */

    private FilterConfig filterConfig;

    protected Dispatcher dispatcher;

    /**

     * 创建一个默认的dispatcher，初始化filter

     * 设置默认的packages     *

     */

    public void init(FilterConfig filterConfig) throws ServletException {

     this.filterConfig = filterConfig;

       dispatcher = createDispatcher(filterConfig);

        dispatcher.init();

        String param = filterConfig.getInitParameter("packages");

        String packages = "org.apache.struts2.static template org.apache.struts2.interceptor.debugging";

        if (param != null) {

            packages = param + " " + packages;

        }

        this.pathPrefixes = parse(packages);
    }
        //销毁filter方法
       public void destroy() {

        if (dispatcher == null) {

            LOG.warn("something is seriously wrong, Dispatcher is not initialized (null) ");

        } else {

            dispatcher.cleanup();

        }

    }

    /**

     * 处理一个Action或者资源请求

     * <p/>

     * filter尝试将请求同action mapping相匹配

     * 如果找到，将执行dispatcher的serviceAction方法

     * 如果Action处理失败, doFilter将建立一个异常

     * <p/>

     * 如果请求静态资源

     * 资源将被直接复制给 response

     * <p/>

     * 如果找不到匹配Action 或者静态资源，则直接跳出

     public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) req;

        HttpServletResponse response = (HttpServletResponse) res;

        ServletContext servletContext = getServletContext();

        String timerKey = "FilterDispatcher_doFilter: ";

        try {

            UtilTimerStack.push(timerKey);

            request = prepareDispatcherAndWrapRequest(request, response);

            ActionMapping mapping;

            try {

                mapping=actionMapper.getMapping(request, dispatcher.getConfigurationManager());

            } catch (Exception ex) {

                LOG.error("error getting ActionMapping", ex);

                dispatcher.sendError(request, response, servletContext, HttpServletResponse.SC_INTERNAL_SERVER_ERROR, ex);

                return;

            }

            if (mapping == null) {

                String resourcePath = RequestUtils.getServletPath(request);

                if ("".equals(resourcePath) && null != request.getPathInfo()) {

                    resourcePath = request.getPathInfo();

                }

                if (serveStatic && resourcePath.startsWith("/struts")) {

                    String name = resourcePath.substring("/struts".length());

                    findStaticResource(name, request, response);

                } else {

                    //为一个普通的request, 则通过

                    chain.doFilter(request, response);

                }

                return;

            }

        /**
        *这个方法询问ActionMapper是否需要调用某个Action来处理这个（request）请求，
        *如果ActionMapper决定需要调用某个Action，
        *FilterDispatcher则把请求的处理交给ActionProxy
        */
            dispatcher.serviceAction(request, response, servletContext, mapping);

        } finally {

            try {

                ActionContextCleanUp.cleanUp(req);

            } finally {

                UtilTimerStack.pop(timerKey);

            }

        }
    }
    }
```
>在doFilter()方法中，将调用dispatcher.serviceAction，该方法如果找到相应的Action，将把用户请求交给ActionProxy。serviceAction()代码在Dispatcher.java中，如代码3.2所示。
代码3.2  Dispatcher类
```
    public class Dispatcher {
    /**
     * 为mapping加载类，并调用相应的方法或者直接返回result
     * <p/>

     * 根据用户请求的参数，建立Action上下文

     * 根据指定的Action’名称和包空间名称，加载一个Action代理 <tt>ActionProxy</tt> 

     * 然后Action的相应方法将被执行，

     */

    public void serviceAction(HttpServletRequest request, HttpServletResponse response, ServletContext context, ActionMapping mapping) throws ServletException {

        Map<String, Object> extraContext = createContextMap(request, response, mapping, context);

        //如果存在一个值栈，则建立一个新的并复制以备Action使用

        ValueStack stack = (ValueStack) request.getAttribute(ServletActionContext.STRUTS_VALUESTACK_KEY);

        if (stack!= null) {

            extraContext.put(ActionContext.VALUE_STACK, ValueStackFactory.getFactory().createValueStack(stack));

        }

        String timerKey = "Handling request from Dispatcher";

        try {

            UtilTimerStack.push(timerKey);

            String namespace = mapping.getNamespace();

            String name = mapping.getName();

            String method = mapping.getMethod();

            Configuration config = configurationManager.getConfiguration();

            //FilterDispatcher把请求的处理交给ActionProxy

            ActionProxy proxy = config.getContainer().getInstance(ActionProxyFactory.class).createActionProxy(namespace, name, extraContext, true, false);

            proxy.setMethod(method);

            request.setAttribute(ServletActionContext.STRUTS_VALUESTACK_KEY, proxy.getInvocation().getStack());

            //ActionMapping 直接返回一个result

            if (mapping.getResult() != null) {

                Result result = mapping.getResult();

                result.execute(proxy.getInvocation());

            } else {

                proxy.execute();

            }

            if (stack != null) {

                request.setAttribute(ServletActionContext.STRUTS_VALUESTACK_KEY, stack);

            }

        } catch (ConfigurationException e) {

            LOG.error("Could not find action or result", e);

            sendError(request, response, context, HttpServletResponse.SC_NOT_FOUND, e);

        } catch (Exception e) {

            throw new ServletException(e);

        } finally {

            UtilTimerStack.pop(timerKey);

        }

    }
        
    ｝
```
从上面代码中可以看出来，Struts 2用于处理用户请求的Action实例，并不是用户实现的业务控制器，而是Action代理。关于Action代理相关内容，读者可以参考拦截器章节的介绍。

★ 提示 ★
--------
前面一直在说Action可以是一个普通的Java类，与Servlet API完全分离，但是为了实现业务逻辑，Action需要使用HttpServletRequest内容。

Struts 2设计的精巧之处就是使用了Action代理，Action代理可以根据系统的配置，加载一系列的拦截器，由拦截器将HttpServletRequest参数解析出来，传入Action。同样，Action处理的结果也是通过拦截器传入HttpServletResponse，然后由HttpServletRequest传给用户。其实，该处理过程是典型的AOP（面向切面编程）的方式，读者可以在后面详细了解到。Struts 2处理过程模型如图3.2所示。
![图3.2  Struts 2处理过程模型][4]

★ 说明 ★
------

拦截器是Struts 2框架的核心，通过拦截器，实现了AOP（面向切面编程）。使用拦截器，可以简化Web开发中的某些应用，例如，权限拦截器可以简化Web应用中的权限检查。

#### 3.1.2  业务控制器Action
业务控制器Action是由开发者自己编写实现的，Action类可以是一个简单的Java类，与Servlet API完全分离。Action一般都有一个execute()方法，也可以定义其他业务控制方法，详细内容将在后面介绍。
Action的execute()返回一个String类型值，这与Struts 1返回的ActionForward相比，简单易懂。Struts 2提供了一个ActionSupport工具类，该类实现了Action接口和validate()方法，一般开发者编写Action可以直接继承ActionSupport类。编写Action类后，开发者还必须在配置文件中配置Action。一个Action的配置应该包含下面几个元素：

* 该Action的name，即用户请求所指向的URL。
* Action所对应的class元素，对应Action类的位置。
* 指定result逻辑名称和实际资源的定位。
*  Action是业务控制器，笔者建议在编写Action的时候，尽量避免将业务逻辑放到其中，尽量减少Action与业务逻辑模块或者组件的耦合程度。

#### 3.1.3  业务模型组件
业务模型组件可以是实现业务逻辑的模块，可以是EJB、POJO或者JavaBean，在实际开发中，对业务模型组件的区分和定义也是比较模糊的，实际上也超出了Struts 2框架的范围。不同的开发者或者团队，都有自己的方式来实现业务逻辑模块，Struts 2框架的目的就是使用Action来调用业务逻辑模块。例如一个银行存款的业务逻辑模块，如代码3.3所示。
代码3.3  模拟一个银行业务的实现模块


    public class Bank {
    
        //定义银行账户
    
        private String accounts;
    
        //定义操作金额
    
        private double money;
    
        //属性的getter和setter方法
    
        public String getAccounts() {
    
            return accounts;
    
        }
    
        public void setAccounts(String accounts) {
    
            this.accounts = accounts;
    
        }
    
        public double getMoney() {
    
            return money;
    
        }
    
        public void setMoney(double money) {
    
            this.money = money;
    
        }
    
        //模拟银行存款方法
    
        public boolean saving(String accounts, double money) {
    
            //调用DAO等模块读写数据库
    
            return dosomeing();
    
        }
    }

>上面实例在实际开发中没有任何意义，这里只是作为业务逻辑模块来说明，在执行saving(String accounts,double money)方法时，可以调用相应的数据库访问其他组件，来实现存款操作。使用Action调用该业务逻辑组件可以在execute()方法中实现，如代码3.4所示。

代码3.4  业务控制器Bank_Saving_Action

    public class Bank_Saving_Action extends ActionSupport {

    //定义银行账户

    private String accounts;

    //定义操作金额

    private double money;

    

    public String execute() throws Exception {

        //创建Bank实例

        Bank bk=new Bank();

        //调用存款方法

        if (bk.saving(accounts, money)){

        return SUCCESS;

        }else{

        return ERROR;

        }

    }

    //属性的getter和setter方法

    public String getAccounts() {

        return accounts;

    }

    public void setAccounts(String accounts) {

        this.accounts = accounts;

    }

    public double getMoney() {

        return money;

    }

    public void setMoney(double money) {

        this.money = money;

    }



Bank_Saving_Action演示了对银行存款业务逻辑组件的调用，这里是通过在Action中创建业务逻辑组件实例的方式实现的。在实际开发中，可以使用静态工厂获得业务逻辑组件的实例或者使用IoC容器来管理。Action中不实现任何业务逻辑，只是负责组织调度业务逻辑组件。调用关系如图3.3所示
![图3.3  调用业务逻辑组件][5]

★ 说明 ★
------
业务控制器Action一般情况下不是直接创建业务逻辑组件实例，而是使用工厂模式或者是从spring容器中获得业务逻辑组件实例，这样可以提高系统的性能。

  [1]: http://www.blogjava.net/images/blogjava_net/lzhidj/14.PNG
  [2]: http://www.blogjava.net/images/blogjava_net/lzhidj/15.PNG
  [3]: http://p.blog.csdn.net/images/p_blog_csdn_net/Struts2BBS/3-1.jpg
  [4]: http://p.blog.csdn.net/images/p_blog_csdn_net/Struts2BBS/3-2.jpg
  [5]: http://p.blog.csdn.net/images/p_blog_csdn_net/Struts2BBS/3-3.jpg