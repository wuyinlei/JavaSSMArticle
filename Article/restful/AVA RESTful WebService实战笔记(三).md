# JAVA RESTful WebService实战笔记(三)
时间 : 2017年9月12号

标签（空格分隔）： JAVA后台 RESTful 

---
### 前言
AOP(Aspect oriented Programming,面向切面编程),其实现原理就是代理被调用的方法,在其被执行的方法前后,增加额外的业务功能,AOP的实现机制就是通过注解或者XML配置,就这些配置,动态的生成字节码(bytecode).使被调用代码对应的字节码被环绕注入新的功能;或者使用Java的动态代理机制,完成对被调用方法的增强.



### REST请求流程
在以下的图中,请求流程中存在3种角色,分别是用户、REST客户端和REST服务器。请求始于请求的发送,止于调用Response类的readeEntity()方法,获取响应实体。

![image.png](http://upload-images.jianshu.io/upload_images/1316820-60d04235281ccf1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 1、用户提交请求数据,客户端接收请求,进入第一个扩展点:"***客户端请求过滤器ClientRequestFilter实现类***"的filter()方法。
* 2、请求过滤器处理完毕之后,流程进入第二个扩展点:"客户端写拦截器WriterInterceptor实现类"的aroundWriteTo()方法,实现对客户端序列化操作的拦截
* 3、"***客户端消息体写处理器MessageBodyWriter***"执行序列化,流程从客户端过渡到服务器端
* 4、服务器接收请求,流程进入第三个扩展点:"***服务器前置请求过滤器ContainerRequestFilter实现类***"的filter()方法。
* 5、过滤处理完毕后,服务器根据请求匹配资源方法,如果匹配到相应的资源方法,流程进入第四个扩展点:"***服务器后置请求过滤器ContainerRequestFilter实现类***"的filter()方法
* 6、后置请求过滤器处理完毕之后,流程进入第五个扩展点:"服务器读拦截器ReaderInterceptor实现类的"aroundReadFrom()方法,拦截服务器端反序列化操作
* 7、"服务器消息体读处理器MessageBodyReader"完成对客户端数据流的反序列化。服务器执行匹配的资源方法
* 8、REST请求资源处理完毕之后,流程进入第六个扩展点:"***服务器相应过滤器ContainerResponseFilter实现类***"的filter()方法
* 9、过滤器处理完毕之后,流程进入第七个扩展点:"***服务器写拦截器WriterInterceptor实现类***"的aroundWriteTo()方法,对服务器端序列化到客户端这个操作的拦截
* 10、"***服务器消息体写处理器MessageBodyWriter***"执行序列化,流程返回到客户端一侧。
* 11、客户端接收响应,流程进入第八个扩展点:"***客户端响应过滤器ClientResponseFilter实现***类"的filter()方法
* 12、过滤器处理完毕后,客户端响应实例response返回到用户一侧,用户执行response.readEntity()流程进入第九个扩展点:"***客户端读取拦截器ReaderInterceptor实现类***"的aroundReadFrom()方法,对客户端反序列化进行拦截
* 13、"客户端消息体赌徒处理器MessageBodyReader"执行反序列化,将Java类型的对象最终作为readEntity()方法的返回值。到此,一次REST请求处理器的完整流程完毕
>这期间如果出现异常或者资源不匹配情况,会从出错点开始结束流程

### REST过滤器

##### 1、ClientRequestFilter
* 客户端请求过滤器(ClientRequestFilter)定义的过滤方法filter()包含一个输入参数,是客户端请求的上下文类ClientRequestFilter。从该上下文中可以获取请求信息,典型的示例包括获取请求context.getMethod(),获取请求资源地址context.getUri()和获取请求头信息context.getHeaders()等。过滤器的实现类中可以利用这些信息,覆写改方法以实现特有的过滤功能。ClientRequestFilter接口的实现类如下:

![image.png](http://upload-images.jianshu.io/upload_images/1316820-0f6da22681522bd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 2、ContainerRequestFilter
* 针对过滤切面,服务器请求过滤接口ContainerRequestFilter的实现类可以定义为预处理和后处理,默认情况下,采用后处理方式。及先执行容器接收请求操作.当服务器接收并处理请求后.流程才进入过滤器实现类的filter()方法。而预处理是在服务器处理接收到的请求之前就执行过滤。如果希望实现一个预处理的过滤器实现类,需要在类名上定义注解@PreMatching
* 服务器请求过滤器定义的过滤方法filter()包含一个输入参数,即容器请求上下文类ContainerRequestContext。ContainerRequestFilter接口的实现类如下:
![image.png](http://upload-images.jianshu.io/upload_images/1316820-278f7297cd58b2b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 如下代码展示了ContainerRequestFilter接口的实现类,我们以CsrfProtectionFilter为例来说明,示例代码如下所示:
```
public class CsrfProtectionFilter implements ClientRequestFilter {

    /**
     * Name of the header this filter will attach to the request.
     */
    public static final String HEADER_NAME = "X-Requested-By";

    private static final Set<String> METHODS_TO_IGNORE;

    static {
        HashSet<String> mti = new HashSet<String>();
        mti.add("GET");
        mti.add("OPTIONS");
        mti.add("HEAD");
        METHODS_TO_IGNORE = Collections.unmodifiableSet(mti);
    }

    private final String requestedBy;

    /**
     * Creates a new instance of the filter with X-Requested-By header value set to empty string.
     */
    public CsrfProtectionFilter() {
        this("");
    }

    /**
     * Initialized the filter with a desired value of the X-Requested-By header.
     *
     * @param requestedBy Desired value of X-Requested-By header the filter
     *                    will be adding for all potentially state changing requests.
     */
    public CsrfProtectionFilter(final String requestedBy) {
        this.requestedBy = requestedBy;
    }

    @Override
    public void filter(ClientRequestContext rc) throws IOException {
        if (!METHODS_TO_IGNORE.contains(rc.getMethod()) && !rc.getHeaders().containsKey(HEADER_NAME)) {
            rc.getHeaders().add(HEADER_NAME, requestedBy);
        }
    }
}

```
上述代码中,CsrfProtectionFilter定义了一个特殊的头信息"X-Requested-By"和CSRF忽略监控的方法集合。在过滤器的filter()方法中,首先从上下文中获取头信息rc.getHeaders()和请求方法信息rc.getMethod(),然后判断头信息是否包含“X-Requested-By”,方法信息是否是安全的请求方法,即"GET"、"OPTIONS"或"HEAD",如果两个条件不成立,过滤器会抛出一个运行时异常BadRequestException

##### 3、ContainerResponseFilter
* 服务器响应过滤器接口ContainerResponseFilter定义的过滤方法filter()包含两个输入参数,一个是容器请求上下文类ContainerRequestContext,另一个是容器响应上下文类ContainerResponseContext。ContainerResponseFilter接口的实现类如下图所示:


![image.png](http://upload-images.jianshu.io/upload_images/1316820-4cbb66f1c3425d82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们通过EncodingFilter为例来说明。
```
 @Override
    public void filter(ContainerRequestContext request, ContainerResponseContext response) throws IOException {
        if (!response.hasEntity()) {
            return;
        }

        // add Accept-Encoding to Vary header
        List<String> varyHeader = response.getStringHeaders().get(HttpHeaders.VARY);
        if (varyHeader == null || !varyHeader.contains(HttpHeaders.ACCEPT_ENCODING)) {
            response.getHeaders().add(HttpHeaders.VARY, HttpHeaders.ACCEPT_ENCODING);
        }

        // if Content-Encoding is already set, don't do anything
        if (response.getHeaders().getFirst(HttpHeaders.CONTENT_ENCODING) != null) {
            return;
        }

        // retrieve the list of accepted encodings
        List<String> acceptEncoding = request.getHeaders().get(HttpHeaders.ACCEPT_ENCODING);

        // if empty, don't do anything
        if (acceptEncoding == null || acceptEncoding.isEmpty()) {
            return;
        }

        // convert encodings from String to Encoding objects
        List<ContentEncoding> encodings = Lists.newArrayList();
        for (String input : acceptEncoding) {
            String[] tokens = input.split(",");
            for (String token : tokens) {
                try {
                    ContentEncoding encoding = ContentEncoding.fromString(token);
                    encodings.add(encoding);
                } catch (ParseException e) {
                    // ignore the encoding that could not parse
                    // but log the exception
                    Logger.getLogger(EncodingFilter.class.getName()).log(Level.WARNING, e.getLocalizedMessage(), e);
                }
            }
        }
        // sort based on quality parameter
        Collections.sort(encodings);
        // make sure IDENTITY_ENCODING is at the end (since it accepted if not explicitly forbidden
        // in the Accept-Content header by assigning q=0
        encodings.add(new ContentEncoding(IDENTITY_ENCODING, -1));

        // get a copy of supported encoding (we'll be modifying this set, hence the copy)
        SortedSet<String> acceptedEncodings = Sets.newTreeSet(getSupportedEncodings());

        // indicates that we can pick any of the encodings that remained in the acceptedEncodings set
        boolean anyRemaining = false;
        // final resulting value of the Content-Encoding header to be set
        String contentEncoding = null;

        // iterate through the accepted encodings, starting with the highest quality one
        for (ContentEncoding encoding : encodings) {
            if (encoding.q == 0) {
                // ok, we are down at 0 quality
                if ("*".equals(encoding.name)) {
                    // no other encoding is acceptable
                    break;
                }
                // all the 0 quality encodings need to be removed from the accepted ones (these are explicitly
                // forbidden by the client)
                acceptedEncodings.remove(encoding.name);
            } else {
                if ("*".equals(encoding.name)) {
                    // any remaining encoding (after filtering out q=0) will be acceptable
                    anyRemaining = true;
                } else {
                    if (acceptedEncodings.contains(encoding.name)) {
                        // found an acceptable one -> we are done
                        contentEncoding = encoding.name;
                        break;
                    }
                }
            }
        }

        if (contentEncoding == null) {
            // haven't found any explicit acceptable encoding, let's see if we can just pick any of the remaining ones
            // (if there are any left)
            if (anyRemaining && !acceptedEncodings.isEmpty()) {
                contentEncoding = acceptedEncodings.first();
            } else {
                // no acceptable encoding can be sent -> return NOT ACCEPTABLE status code back to the client
                throw new NotAcceptableException();
            }
        }

        // finally set the header - but no need to set for identity encoding
        if (!IDENTITY_ENCODING.equals(contentEncoding)) {
            response.getHeaders().putSingle(HttpHeaders.CONTENT_ENCODING, contentEncoding);
        }
    }

```
EncodingFilter过滤器的filter()方法通过对请求头信息"Accept-Encoding"的分析,先后为响应头信息"Vary"和"Content-Encoding"赋值,以实现编码部分的内容协商。
##### 4、ClientResponseFilter
客户端响应过滤器(ClientResponseFilter)定义的过滤方法filter()包含两个参数,一个是客户端请求上下文类ClientRequestContext,另一个是客户端响应的上下文类ClientResponseContext。ClientResponseFilter接口的实现类如下图所示:

![image.png](http://upload-images.jianshu.io/upload_images/1316820-9f7f04b4b92d3521.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们以HTTP摘要认证过滤器类HttpAuthenticationFilter为例。
```
 @Override
    public void filter(ClientRequestContext request) throws IOException {
        if ("true".equals(request.getProperty(REQUEST_PROPERTY_FILTER_REUSED))) {
            return;
        }

        if (request.getHeaders().containsKey(HttpHeaders.AUTHORIZATION)) {
            return;
        }

        Type operation = null;
        if (mode == HttpAuthenticationFeature.Mode.BASIC_PREEMPTIVE) {
            basicAuth.filterRequest(request);
            operation = Type.BASIC;
        } else if (mode == HttpAuthenticationFeature.Mode.BASIC_NON_PREEMPTIVE) {
            // do nothing
        } else if (mode == HttpAuthenticationFeature.Mode.DIGEST) {
            if (digestAuth.filterRequest(request)) {
                operation = Type.DIGEST;
            }
        } else if (mode == HttpAuthenticationFeature.Mode.UNIVERSAL) {

            Type lastSuccessfulMethod = uriCache.get(getCacheKey(request));
            if (lastSuccessfulMethod != null) {
                request.setProperty(REQUEST_PROPERTY_OPERATION, lastSuccessfulMethod);
                if (lastSuccessfulMethod == Type.BASIC) {
                    basicAuth.filterRequest(request);
                    operation = Type.BASIC;
                } else if (lastSuccessfulMethod == Type.DIGEST) {
                    if (digestAuth.filterRequest(request)) {
                        operation = Type.DIGEST;
                    }
                }
            }
        }

        if (operation != null) {
            request.setProperty(REQUEST_PROPERTY_OPERATION, operation);
        }
    }
```

