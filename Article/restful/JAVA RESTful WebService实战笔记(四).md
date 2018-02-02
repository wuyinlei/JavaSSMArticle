# JAVA RESTful WebService实战笔记(四)
时间 : 2017年9月13号

标签（空格分隔）： JAVA后台 RESTful 

---

###前言
在JAVA RESTful WebService实战笔记(三)中已经完成了对JAX-RS2定义的4中过滤器的讲述学习,以下就来看看如何综合运用过滤器,完成一个记录REST请求的访问日志

### 访问日志（最新版没有AirLogFilter,应该是LoggingFilter）
* LoggingFilter实现了上述的4个过滤器,记录服务器端和客户端的请求和响应运行时候的信息,LoggingFilter类的定义如下所示:
```
public final class LoggingFilter implements ContainerRequestFilter, ClientRequestFilter, ContainerResponseFilter,
                                            ClientResponseFilter, WriterInterceptor {
```
* LoggingFilter为每一种过滤器接口定义的filter()方法提供了实现,并且也实现了写入的拦截器。在客户端请求过滤中,输出请求资源地址信息和请求投信息;在容器请求过滤中,输出请求方法,请求资源地址信息和请求头信息;在容器响应过滤中,输出HTTP状态码和请求头信息;在客户端响应过滤中,输出HTTP状态码和请求头信息,4个阶段的filter()示例代码如下:
```
 @Override
    public void filter(final ClientRequestContext context) throws IOException {
        final long id = _id.incrementAndGet();
        context.setProperty(LOGGING_ID_PROPERTY, id);
    
        final StringBuilder b = new StringBuilder();
        //获取请求方法和地址
        printRequestLine(b, "Sending client request", id, context.getMethod(), context.getUri());
        //获取请求头信息
        printPrefixedHeaders(b, id, REQUEST_PREFIX, context.getStringHeaders());

        if (printEntity && context.hasEntity()) {
            final OutputStream stream = new LoggingStream(b, context.getEntityStream());
            context.setEntityStream(stream);
            context.setProperty(ENTITY_LOGGER_PROPERTY, stream);
            // not calling log(b) here - it will be called by the interceptor
        } else {
            log(b);
        }
    }

```
```
 @Override
    public void filter(final ClientRequestContext requestContext, final ClientResponseContext responseContext)
            throws IOException {
        final Object requestId = requestContext.getProperty(LOGGING_ID_PROPERTY);
        final long id = requestId != null ? (Long) requestId : _id.incrementAndGet();

        final StringBuilder b = new StringBuilder();
        //获取容器响应状态
        printResponseLine(b, "Client response received", id, responseContext.getStatus());
        //获取容器响应头信息
        printPrefixedHeaders(b, id, RESPONSE_PREFIX, responseContext.getHeaders());
```
* 单元测试类
```
public class TIResourceJtfTest extends JerseyTest {
    @Override
    protected Application configure() {
        ResourceConfig config = new ResourceConfig(BookResource.class);
        return config.register(com.example.filter.log.AirLogFilter.class);
    }
    @Override
    protected void configureClient(ClientConfig config) {
        config.register(new AirLogFilter());
    }
```
上述代码中,为了访问日志生效,需要测试类TIResourceJtfTest在Jersey测试框架的服务器端和客户端,分别注册服务日志类AirLogFilter,单元测试结果如下
```
2017-09-13 10:01:58,135 DEBUG [com.example.resource.TIResourceJtfTest] main - >>Test Post
2017-09-13 10:01:58,232 INFO  [com.example.filter.log.AirLogFilter] main - 1 * AirLog - Request received on thread main
1 / POST http://localhost:9998/books/
1 / Accept: application/json
1 / Content-Type: application/json

2017-09-13 10:01:59,038 DEBUG [com.example.resource.interceptor.AirReaderWriterInterceptor] grizzly-http-server-0 - null:Java Restful Web Service实战-602865027284019:null
2017-09-13 10:01:59,043 DEBUG [com.example.resource.interceptor.AirReaderWriterInterceptor] grizzly-http-server-0 - 602865932131718:Java Restful Web Service实战-602865027284019:null
2017-09-13 10:01:59,084 INFO  [com.example.filter.log.AirLogFilter] main - 2 * AirLog - Response received on thread main
2 \ 200
2 \ Content-Length: 86
2 \ Date: Wed, 13 Sep 2017 02:01:59 GMT
2 \ Content-Type: application/json

2017-09-13 10:01:59,096 DEBUG [com.example.resource.TIResourceJtfTest] main - <<Test Post
九月 13, 2017 10:01:59 上午 org.glassfish.grizzly.http.server.NetworkListener shutdownNow
信息: Stopped listener bound to [localhost:9998]

```


### REST拦截器
拦截器和过滤器的相同点就是都是一种在请求--响应模型中,用作切面处理的Provider。两者的不同除了功能性上的差异(一个用于过滤消息,一个用于拦截处理)之外,形式上也不同,拦截器通常都是"读写"成对,而且没有服务器端和客户端的区分。Jersey提供的拦截器如下:
![image.png](http://upload-images.jianshu.io/upload_images/1316820-9d3ec3d40613fcfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 1、ReaderInterceptor
* 读拦截器ReaderInterceptor定义的拦截方法是aroundReadFrom(),该方法包含一个输入参数,即读拦截器的上下文接口ReaderInterceptorContext,从中可以获取到投信息,输入流以及父接口InterceptorContext提供的媒体类型等上下文信息。接口方法示例如下:
```
 /**
     * Interceptor method wrapping calls to {@link MessageBodyReader#readFrom} method.
     *
     * The parameters of the wrapped method called are available from {@code context}.
     * Implementations of this method SHOULD explicitly call {@link ReaderInterceptorContext#proceed}
     * to invoke the next interceptor in the chain, and ultimately the wrapped
     * {@link MessageBodyReader#readFrom} method.
     *
     * @param context invocation context.
     * @return result of next interceptor invoked or the wrapped method if last interceptor in chain.
     * @throws java.io.IOException if an IO error arises or is thrown by the wrapped
     *                             {@code MessageBodyReader.readFrom} method.
     * @throws javax.ws.rs.WebApplicationException
     *                             thrown by the wrapped {@code MessageBodyReader.readFrom} method.
     */
    public Object aroundReadFrom(ReaderInterceptorContext context)
            throws java.io.IOException, javax.ws.rs.WebApplicationException;
```
##### 2、WriterInterceptor
写拦截器WriterInterceptor定义的拦截方法是aroundWriteTo(),该方法包含一个输入参数,写拦截器上下文接口WriterInterceptorContext,从中可以获取投信息,输出流以及父接口InterceptorContext提供的媒体类型等上下文信息,接口方法示例如下:
```
 /**
     * Interceptor method wrapping calls to {@link MessageBodyWriter#writeTo} method.
     * The parameters of the wrapped method called are available from {@code context}.
     * Implementations of this method SHOULD explicitly call
     * {@link WriterInterceptorContext#proceed} to invoke the next interceptor in the chain,
     * and ultimately the wrapped {@code MessageBodyWriter.writeTo} method.
     *
     * @param context invocation context.
     * @throws java.io.IOException if an IO error arises or is thrown by the wrapped
     *                             {@code MessageBodyWriter.writeTo} method.
     * @throws javax.ws.rs.WebApplicationException
     *                             thrown by the wrapped {@code MessageBodyWriter.writeTo} method.
     */
    void aroundWriteTo(WriterInterceptorContext context)
            throws java.io.IOException, javax.ws.rs.WebApplicationException;
```

-----

## REST服务与异步




### 服务端实现
可以利用JAX-RS2提供额AsyncResponse,通过一个异步线程来执行查询,在查询完后,由这个异步线程完成对请求的响应。
##### 1、异步资源类
```
@Component
@Path("books")
@Produces({"application/json;charset=UTF-8", "application/javascript;charset=UTF-8", "text/javascript;charset=UTF-8"})
public class AsyncResource {
    private static final Logger log = LogManager.getLogger(AsyncResource.class);
    public static final long TIMEOUT = 120;
    final ExecutorService threadPool = Executors.newFixedThreadPool(10);

    @GET
    public void getAll(@Suspended final AsyncResponse asyncResponse) {
        //该方法用于定义回调
        configResponse(asyncResponse);
        final BatchRunner batchTask = new BatchRunner(asyncResponse);
        threadPool.submit(batchTask);
    }

    //回调方法  当请你去处理完成之后,CompletionCallback实例的onComplete()方法将会被回调,实现onComplete()方法,可以监听请求处理完成事件并实现相关业务流程。
    private void configResponse(final AsyncResponse asyncResponse) {
        asyncResponse.register((CompletionCallback) throwable -> {
            if (throwable == null) {
                log.info("CompletionCallback-onComplete: OK");
            } else {
                log.info("CompletionCallback-onComplete: ERROR: " + throwable.getMessage());
            }
        });

        //
        asyncResponse.register((ConnectionCallback) disconnected -> {
            //Status.GONE=410
            log.info("ConnectionCallback-onDisconnect");
            //当请求--响应模型的连接断开的时候,CompletionCallback实例的onDisconnect()方法会被回调。实现这个方法可以监听连接断开事件并实现相关业务,比如主动唤醒AsyncRespurce实例并设置状态码HTTP为410、客户端请求资源不可用(Response.status(Response.Status.GONE)来完成响应工作。
            disconnected.resume(Response.status(Response.Status.GONE).entity("disconnect!").build());
        });

        asyncResponse.setTimeoutHandler(new TimeoutHandler() {
        
        //TimeoutHandler是JAX-RS2定义的超时处理接口,用于处理异步响应类超时事件,当预期的超时时间到达之后,TimeoutHandler实例的handleTimeout()方法就会被调用。实现这个方法可以监听超时时间并处理相关业务。  并设置状态码为503、服务器端服务不可用(Response.Status.SERVICE_UNAVAILABLE)  TimeoutHandler的实现可以作为AsyncResource的setTimeoutHandler()方法的参数来配置。AsyncResource的setTimeout()方法用于设置超时时间,默认永不超时。
            @Override
            public void handleTimeout(AsyncResponse asyncResponse) {
                //Status.SERVICE_UNAVAILABLE=503
                log.info("TIMEOUT");
                asyncResponse.resume(Response.status(Response.Status.SERVICE_UNAVAILABLE).entity("Operation time out.").build());
            }
        });
        asyncResponse.setTimeout(TIMEOUT, TimeUnit.SECONDS);
    }

    class BatchRunner implements Runnable {
        private final AsyncResponse response;

        public BatchRunner(AsyncResponse asyncResponse) {
            this.response = asyncResponse;
        }

        @Override
        public void run() {
            try {
                Books books = doBatch();
                response.resume(books);
            } catch (InterruptedException e) {
                log.error(e);
            }
        }

        private Books doBatch() throws InterruptedException {
            Books books = new Books();
            for (int i = 0; i < 10; i++) {
                Thread.sleep(500);
                Book book = new Book(i + 10000l, "Java RESTful Web Services", "华章");
                log.debug(book);
                books.getBookList().add(book);
            }
            return books;
        }
    }
}
```

* 1、测试方法:
```
    @Test
    public void testAsync() throws InterruptedException, ExecutionException {
        final Invocation.Builder request = target("http://localhost:" + this.port + "/books").request();
        final AsyncInvoker async = request.async();
        //客户端试用AsyncInvoker的get()方法提交异步请求.该方法返回Future接口的实例,客户端线程可以以非阻塞的方法处理其他业务流程,然后调用Future的get()方法来获取服务器处理结果。
        final Future<Books> responseFuture =
    async.get(Books.class);
        long beginTime = System.currentTimeMillis();
        try {
            Books result = responseFuture.get(AsyncResource.TIMEOUT + 1, TimeUnit.SECONDS);
            log.debug("Testing result size = {}", result.getBookList().size());
            //如果在指定时间内服务器没有响应,将会报TimeoutException异常,我们可以捕获这个异常并且实现超时处理。
        } catch (TimeoutException e) {
            log.debug("Fail to request asynchronously", e);
        } finally {
            log.debug("Elapsed time = {}", System.currentTimeMillis() - beginTime);
        }
    }

```

* 2、回调方法
```
  @Test
    public void testAsyncCallBack() throws InterruptedException, ExecutionException {
        final AsyncInvoker async = target("http://localhost:" + this.port + "/books").request().async();
        final Future<Books> responseFuture = async.get(new InvocationCallback<Books>() {
            //处理REST回调成功的方法
            @Override
            public void completed(Books result) {
                log.debug("On Completed: " + result.getBookList().size());
            }
            //处理REST回调失败的方法
            @Override
            public void failed(Throwable throwable) {
                log.debug("On Failed: " + throwable.getMessage());
                throwable.printStackTrace();
            }
        });
        log.debug("First response time::" + System.currentTimeMillis());
        try {
            responseFuture.get(AsyncResource.TIMEOUT + 1, TimeUnit.SECONDS);
        } catch (TimeoutException e) {
            log.debug("", e);
        } finally {
            log.debug("Second response time::" + System.currentTimeMillis());
        }
    }
```
* 3、测试结果
```
    {
        "book":{
            {
                "bookId": 10000,
                "bookName": "JAVA RESTful Web Service",
                "publisher": "华章"
            },
             {
                "bookId": 10001,
                "bookName": "JAVA RESTful Web Service",
                "publisher": "华章"
            },
             {
                "bookId": 100002,
                "bookName": "JAVA RESTful Web Service",
                "publisher": "华章"
            },
             {
                "bookId": 10003,
                "bookName": "JAVA RESTful Web Service",
                "publisher": "华章"
            },
             {
                "bookId": 10004,
                "bookName": "JAVA RESTful Web Service",
                "publisher": "华章"
            },
             {
                "bookId": 10005,
                "bookName": "JAVA RESTful Web Service",
                "publisher": "华章"
            }
    
        }
    }
```
