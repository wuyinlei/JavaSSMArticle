# JAVA RESTful WebService实战笔记(二)

时间 : 2017年9月11号

标签（空格分隔）： JAVA后台 RESTful 

---


### 资源定位之注解相关简介

#### @QueryParam注解
JAX-RS2定义了@QueryParam注解来定义查询参数,如下表所示
|接口描述|资源地址|
|:---:|:---:|
|分页查询列表数据|/query-resource/test?start=24&size=10|
|排序并分页查询列表收|/query-resource/test?limit=5&sort=program|
|查询单项数据|/query-resource/test?id=9|

##### 1、分页查询
```
   /**
     *
     * @param start  起始条目  参数都使用了final进行限制
     *               符合Checkstyle风格 即输入参数只作为逻辑算法的依据使用
     *               其本身并不会再找个过程中被修改
     * @param size 查询的条目
     * @return
     */
    @Path("/page")  //page?start=0&size=10  访问
    @GET
    @Consumes(MediaType.APPLICATION_JSON)  //指定请求返回的响应体为JSON
    @Produces(MediaType.APPLICATION_JSON)
    public User getByPaging(@QueryParam("start") final int start,@QueryParam("size") final int size){

        return null;
    }
```
##### 2、排序查询
```

    /**
     * @param limit  分页查询条目
     * @param sortName 排序规则
     * @return
     */
    @Path("/order")    //order?limit=10&sort=web 访问
    @GET
    @Consumes(MediaType.APPLICATION_JSON)  //指定请求返回的响应体为JSON
    @Produces(MediaType.APPLICATION_JSON)
    public User getByOrder(@QueryParam("limit") final int limit,@QueryParam("sort") final int sortName){

        return null;
    }
```
##### 3、单项查询
```
    /**
     * @param segId  
     * @return
     */
    @Path("/query")    //order?id=10 访问
    @GET
    @Consumes(MediaType.APPLICATION_JSON)  //指定请求返回的响应体为JSON
    @Produces(MediaType.APPLICATION_JSON)
    public User getByQuery(@QueryParam("id") final int segId){

        return null;
    }
```

#### @PathParam注解
JAX-RS2定义@PathParam注解来定义路径参数----每个参数对应一个子资源,示例列表如下：
|接口描述|资源地址|
|:--:|:--:|
|基本路径参数|/path-resource/Eric|
|J结合查询参数|/path-resource/Eric?hometown=Luoma|
|带有标点符号的资源路径|/path-resource/199-1999<br>/path-resource/01,2012-12,2014|
|子资源变长的资源路径|/path-resource/Asia/china/northeast/liaoning/shenyang/huanggu<br>//path-resource/q/restful;program=java;type=web<br>/path-resource/q2/restful;program=java;type=web|

##### 1、@Path注解
JAX-RS2定义@Path注解来定义资源路径,@Path接收一个value参数来解析资源路径地址,可以使用静态定义的方式外,也可以使用动态变量的方式,格式:{参数名称:正则表达式},例如资源地址:/path-resource、199-1999.参考示例如下
```
 @Path("{from:\\d+}-{to:\\d+}")    //order?id=10 访问
    @GET
    @Consumes(MediaType.APPLICATION_JSON)  //指定请求返回的响应体为JSON
    @Produces(MediaType.APPLICATION_JSON)
    public User getByCondition(@QueryParam("from") final Integer from, @PathParam("to") final Integer to) {

        return null;
    }
```
在来一个复杂的例子：/path-resource、01，2012-12，2014(引入了逗号(,))
```
@Path("{beginMonth:\\d+},{beginYear:\\d+}-{endMonth:\\d+},{endYear:\\d+}")
```
##### 2、正则表达式
##### 3、路径配查询
查询参数和路径参数在一个接口中配合使用,可以更便捷的完成资源定位。
```
    /**
     * /path-resource/Eric?hometown=Luoma 
     *
     * @param user   Eric
     * @param hometown  Luoma
     * @return
     */
    @Path("{user:[a-zA-Z][a-zA-Z_0-9]*}")    //order?id=10 访问
    @GET
    @Consumes(MediaType.APPLICATION_JSON)  //指定请求返回的响应体为JSON
    @Produces(MediaType.APPLICATION_JSON)
    public User getUserInfo(@QueryParam("user") final String user,
                            @DefaultValue("Shen Yang") @QueryParam("hometown") final String hometown) {

        return null;
    }
```
##### 4、路径区间
路径区间(PathSegment)是对资源地址更加灵活的支持,使资源类的一个方法可以支持更加广泛的资源地址的请求,例如下面的例子
```
/path-resource/Asia/china/northeast/liaoning/shenyang/huanggu
/path-resource/Asia/china/northeast/liaoning/shenyang/tiexi
/path-resource/china/liaoning/shenyang
```
如上所示的资源地址中含有固定子资源(shenyang),和动态子资源两部分,对于动态匹配变长的子资源地址,PathSegment类型的参数结合正则表达式将大显身手,如下:
```
    @Path("{region:.+}/shenyang/{district:\\w+}")              //order?id=10 访问
    @GET
    @Consumes(MediaType.APPLICATION_JSON)  //指定请求返回的响应体为JSON
    @Produces(MediaType.APPLICATION_JSON)
    public User getByAddress(@PathParam("region")final List<PathSegment> region,
                             @PathParam("district") final String district) {
        
        final StringBuilder result = new StringBuilder();
        for (PathSegment pathSegment : region) {
            result.append(pathSegment.getPath()).append("-");
        }
        result.append("shenyang-" + district);

        return null;
    }
```
#### @MatrixParam注解
通过@MatrixParam注解来逐一的定义参数,即通过声明方式来获取,示例代码如下:
```
///path-resource/q2/program=java;type=web
    @Path("q2/{condition}")    //order?id=10 访问
    @GET
    @Consumes(MediaType.APPLICATION_JSON)  //指定请求返回的响应体为JSON
    @Produces(MediaType.APPLICATION_JSON)
    public User getByCondition(@PathParam("condition") final PathSegment condition,
                               @MatrixParam("program") final String program,
                               @MatrixParam("type") final String type) {


        return null;
    }
```
#### @FormParam注解
JAX-RS2定义了@FormParam注解来定义表单参数,相应的REST方法用以处理请求实体媒体类型为Content-Type:application/x-www-form-urlencoded的请求,示例代码如下:
```
@Path("form-resource)
public class FormResource{
    @Post
    public String newPassword(
        @DefaultValue("ruolan") @FormParam(FromResource.USER) final String user,
        @Encoded @FormParam(FormParam.PW) final String password,
        @Encoded @FormParam(FormParam.NPW) final String newPassword,
        @FormParam(FormParam.VNPW) final String verification{
        }
    )
}
```
上述代码中,newPassword()方法是@FormParam注解定义了user等4个参数,这些参数是容器请求中获取并且匹配的,测试代码示例如下:
```
    @Test
    public void testPost(){
        final Form form = new Form();

        form.param(FormResource.USER,"ruolan");
        form.param(FormResource.PW,"北京");
        form.param(FormResource.NPW,"上海");
        form.param(FormResource.VNPW,"上海");

        final String result = target.("form-resource").request()
                .post(Entity.entity(form,MediaType.APPLICATION_FORM_URLENCODED_TYPE),String.class);
        FormTest.LOGGER.debug(result);
        Assert.assertEquals("encoded should let id to disable decoding",
                "ruolan:%E5%8C%97%E4%BA%AC:%E4%B8%8A%E6%B5%B7:上海",result);

    }

```
注意:


* @Encoded注解用以标识禁用自动解码,示例中的测试结果中%E4%B8%8A%E6%B5%B7是newPassword()方法中的参数值"上海"的编码值,当对newPassword使用@Encoded注解,REST方法得到的参数值就不会被编码
* @DefaultValue注解,用以为客户端没有为其提供值的参数 提供默认的参数
#### @BeanParam注解
JAX-RS2定义了@BeanParam注解用于自定义参数组合,使REST方法可以使用简洁的参数形式完成复杂的接口设计
```
public String getByAddress(@BeanParam Jaxrs2GuideParam param) {
//关注点2：参数组合 
public class Jaxrs2GuideParam {
    @HeaderParam("accept")
    private String acceptParam;
    @PathParam("region")
    private String regionParam;
    @PathParam("district")
    private String districtParam;
    @QueryParam("station")
    private String stationParam;
    @QueryParam("vehicle")
    private String vehicleParam;
 
public void testBeanParam() {
...
    final WebTarget queryTarget = target(path).path("China").path("northeast")
    .path("shenyang").path("tiexi")
.queryParam("station", "Workers Village").queryParam("vehicle", "bus");
    result = queryTarget.request().get().readEntity(String.class);
    //关注点3：查询结果断言 
    Assert.assertEquals("China/northeast:tiexi:Workers Village:bus", result);
}
 
```
//关注点4：复杂的查询请求 
```
http://localhost:9998/ctx-resource/China/shenyang/tiexi?station=Workers+Village&vehicle=bus 
```
在这段代码中，getByAddress()方法只用了一个使用@BeanParam注解定义的Jaxrs2GuideParam类型的参数，见关注点1；Jaxrs2GuideParam类定义了一系列REST方法会用到的参数类型，包括示例中使用的查询参数"station"和路径参数"region"等，从而使得getByAddress()方法可以匹配更为复杂的资源路径，见关注点2；在变长子资源的例子基础上，增加了查询条件，但测试方法testBeanParam()发起的请求的资源地址见关注点4；可以看出这是一个较为复杂的查询请求。其中路径部分包括`China/shenyang/tiexi`，查询条件包括`station=Workers+Village和vehicle=bus。`这些条件均在Jaxrs2GuideParam类中可以匹配，因此从关注点3的测试断言中可以看出，该请求响应的预期结果是
```
"China/northeast:tiexi:Workers Village:bus"。
```
#### @CookieParam注解
JAX-RS2定义了@CookieParam注解用以匹配Cookie中的键值对信息，示例如下。
```
@GET
public String getHeaderParams(@CookieParam("longitude") final String longitude,
    @CookieParam("latitude") final String latitude,
    @CookieParam("population") final double population,
    @CookieParam("area") final int area) {//关注点1：资源方法入参 
    return longitude + "," + latitude + " population=" + population + ",area=" + area;
@Test
public void testContexts() {
    final Builder request = target(path).request();
    request.cookie("longitude", "123.38");
    request.cookie("latitude", "41.8");
    request.cookie("population", "822.8");
    request.cookie("area", "12948");
    result = request.get().readEntity(String.class);
    //关注点2：测试结果断言 
    Assert.assertEquals("123.38,41.8 population=822.8,area=12948", result);
}
```
在这段代码中，getHeaderParams()方法包含4个使用@CookieParam注解定义的参数，用于匹配Cookie的字段，见关注点1；在测试方法testContexts中，客户端Builder实例填充了相应的cookie键值对信息，其断言是对cookie字段值的验证，见关注点2。
#### @Context注解

JAX-RS2定义了@Context注解来解析上下文参数，JAX-RS2中有多种元素可以通过@Context注解作为上下文参数使用，示例代码如下。
```
public String getByAddress(
    @Context final Application application,
    @Context final Request request,
    @Context final javax.ws.rs.ext.Providers provider,
    @Context final UriInfo uriInfo,
    @Context final HttpHeaders headers){
在这段代码中，分别定义了Application、Request、Providers、UriInfo和HttpHeaders等5种类型的上下文实例。从这些实例中可以获取请求过程中的重要参数信息，示例代码如下。
final MultivaluedMap<String, String> pathMap = uriInfo.getPathParameters();
final MultivaluedMap<String, String> queryMap = uriInfo.getQueryParameters();
final List<PathSegment> segmentList = uriInfo.getPathSegments();
final MultivaluedMap<String, String> headerMap = headers.getRequestHeaders();
```
在这段代码中，UriInfo类是路径信息的上下文，从中可以获取路径参数集合getPath-Parameters()和查询参数集合getQueryParameters()。类似地，我们可以从HttpHeaders类中获取头信息集合getRequestHeaders()。这些业务逻辑处理中常用的辅助信息的获取，要通过@Context注解定义方法的参数或者类的字段来实现。



----
### 处理响应
REST的响应处理结果应包括响应头中HTTP状态码,响应实体中媒体参数类型和返回值类型,以及异常情况处理。JAX-RS2支持4中返回值类型的响应,分别是无返回值、返回Response类实例、返回GenericEntity类实例和返回自定义类的实例.
##### VOID(无返回值类型)
在返回值类型中是void的响应,其响应实体为空,HTTP状态码是204。例子代码如下:
```
    @DELETE
    @Path("{s}")
    public void deleteTest(@PathParam("s") final String s){
        
        
    }
```
因为delete操作不需要返回值关于资源标书的信息,因此该方法没有返回值.（疑问点,如果需要通知客户端是否删除成功了没有呢??????）

##### RESPONSE(Response返回值类型)
在Response的响应中,响应实体为Response类的entity()方法定义的实体实例。如果该内容为空,则HTTP状态码是204,否则HTTP状态码为200 OK,示例代码如下:
```
    @POST
    @Path("c")
    public Response getResponse(final String s){

        return Response.ok().entity("char[] : " + s ).build();
    }
```
在上述的代码中,Response首先定义了HTTP的状态码为OK,然后填充实体信息,最后调用build()方法构建Response实例

##### GenericEntity(GenericEntity返回值类型)
通用实体类型作为返回值的情况并不是很常用。其形式是构造一个统一的实体实例并将其返回,实体实例作为第一个参数,该实体类型作为第二个参数,示例代码如下:
```
    @POST
    @Path("b")
    public String getGenericEntity(final  byte[] bytes) {
       
        return "byte[] : "  + new String(bytes);
    }
    
    public GenericEntity<String> getGenericTest(final  byte[] bytes){
        
        //构建GenericEntity实例
        return new GenericEntity<>("byte[] : " + new String(bytes),String.class );
    }
```

##### 自定义类型
JDK中的类(例如File,String等)都可以作为返回值类型,更常用的是返回自定义的POJO类型。示例代码如下:
```
    @POST
    @Path("f")
    public File getFile(final File f) throws FileNotFoundException, IOException {
        BufferedReader br = new BufferedReader(new FileReader(f));
        String s;
        do {
            s = br.readLine();
        } while (s != null);
        return f;
    }
    
    @POST
    @Consumes({MediaType.APPLICATION_XML,MediaType.APPLICATION_JSON})
    @Produces(MediaType.APPLICATION_XML)
    public User getEntity(User user){
        return user;
    }

    @POST
    @Consumes({MediaType.APPLICATION_XML,MediaType.APPLICATION_JSON})
    @Produces(MediaType.APPLICATION_XML)
    public User getEntity(JAXBElement<User> user){
        User user1 = user.getValue();
        return user1;
    }
    
```

### 处理异常
##### 处理状态码
|状态码|含义|
|:--:|:--:|
|200 OK| 服务器正常响应|
|201 Created|创建新实体,响应头Location指定访问该实体的URL|
|202 Accepted|服务器接受请求,处理尚未完成。可用于异步处理机制|
|204 No Content|服务器正常响应,但是响应实体为空|
|301 Moved Permanently|请求资源的地址发生永久变动,响应头Location指定新的URL|
|302 Found|请求资源的额地址发生临时变动|
|304 Not Modified|客户端缓存资源依然有效|
|400 Bad Request|请求信息出现语法错误|
|401 Unauthorized|请求资源无法授权给未验证错误|
|403 Frobidden|请求资源未授权当前用户|
|404 Not Found|请求资源不存在|
|405 Method Not Allowed|请求方法不匹配|
|406 Not Acceptable|请求资源的媒体类型不匹配|
|500 Internale Server Error|服务器内部错误,意外终止响应|
|501 Not Implemented|服务器不支持当前请求|


### 内容协商
##### @Produces注解
@Produces注解用于定义方法的响应实体的数据类型,可以定义一个或者多个,同事可以为每种类型定义质量因素(qualityfactor)。质量因素是取值范围从0到1的小数值。如果不定义质量因素,那么该类型的质量因素默认为1
```
    @GET
    @Path("{id}")
    @Produces(MediaType.APPLICATION_XML)
    public User getJaxUser(@PathParam("id") final int userId) {
        return new User(userId);
    }

   
    @GET
    @Path("{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public User getJsonUser(@PathParam("id") final int userId) {
        return new User(userId);
    }
    
    /**
    * 以下的代码中 定义了xml和json两种表述数据类型,xml的质量因素是0.5 json的质量因素是0.9
    **/
    @GET
    @Path("book/{id}")
    @Produces({"application/json;qs=0.9","application/xml;qs=0.6"})
    public User getUser(@PathParam("id") final int userId) {
        return new User(userId);
    }
```

如果客户端的请求中,明确接收的数据类型是两者之一,响应实体使用指定类型.如果没有定义或者两者都定义且JSON的质量因素大雨或者等于XML,则返回JSON,还有一种用例就是,两者都定义但是json的质量因素小于XML,**内容协商的结果按照客户端的喜好选择相应实体的数据类型(xml格式)**

##### @Consumes注解
@Consumes注解用于定义方法的请求实体的数据类型,和@Produces不同的是,@Consumes注解的数据类型的定义只用于JAX-RS2匹配请求处理的方法,不做内容协商使用,如果匹配不到,服务器会返回HTTP状态码415(Unsupported Meia Type),示例代码如下:
```
    @POST
    @Consumes({MediaType.APPLICATION_XML,MediaType.APPLICATION_JSON})
    @Produces(MediaType.APPLICATION_XML)
    public User getEntity(User user){
        final Builder request = Target(path).request();

        final User result = request.post(Entity.entity(user,MediaType.APPLICATION_XML), User.class);
        
        return user;
    }
    
```
>@Consumes媒体类型为XML格式和JSON格式,那么在客户端的请求中,如果请求实体的数据类型定义是两者之一,该方法会被选择为处理请求的方法,否则查找是否有定义为相应数据类型的方法,如果没有抛出javax.ws.rs.NotSupportedException异常,则使用该方法处理请求。
