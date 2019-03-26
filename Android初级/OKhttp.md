# OKhttp

<a name="e05dce83"></a>
# 简介

<br />OKhttp是square公司出品的，它是一个高效的HTTP客户端，（Retrofit中的http通信实现也是基于OKhttp的）它有以下默认特性：<br />

1. HTTP/2 support allows all requests to the same host to share a socket.
1. Connection pooling reduces request latency (if HTTP/2 isn’t available).
1. Transparent GZIP shrinks download sizes.
1. Response caching avoids the network completely for repeat requests.


---

<a name="d19f2c10"></a>
# 基本使用
<a name="ab5b6106"></a>
## 1.添加依赖
目前最新的okhttp的version是3.13.1

```java
dependencies {

   implementation 'com.squareup.okhttp3:okhttp:3.13.1'

}
```


<a name="2dc3a8bd"></a>
## 2.GET方式

<a name="f9e2e636"></a>
### （1）同步请求，关键调用okhttpClient.execute()

```java
    private void GetOKhttp() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                //创建OkhttpClient
                OkHttpClient okHttpClient = new OkHttpClient();
                //创建request
                Request request = new Request.Builder()
                        .url("http://www.baidu.com")
                        .build();
                //同步网络请求
                try {
                    okhttp3.Response response = okHttpClient.newCall(request).execute();
                    //若成功返回
                    if(response.isSuccessful()){
                        ok_string = response.body().toString();
                        handler.obtainMessage(1).sendToTarget();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
```

<a name="51381d85"></a>
### （2）异步请求，关键调用okhttpClient.enqueue

```java
    private void GetOKhttp() {
        //创建OkHttpClient实例
        OkHttpClient okHttpClient = new OkHttpClient();
        //创建Request对象
        Request request = new Request.Builder()
                .url("http://www.baidu.com")
                .build();
        //获取Call接口
        okhttp3.Call call = okHttpClient.newCall(request);
        //异步网络请求
        call.enqueue(new okhttp3.Callback() {
            @Override
            public void onFailure(okhttp3.Call call, IOException e) {

            }

            @Override
            public void onResponse(okhttp3.Call call, okhttp3.Response response) throws IOException {
                Log.i("xw", "11111:" + response.isSuccessful());
            }
        });
    }
```

同步请求是在主线程中进行的，因此若网络请求耗时我们需要将其放在线程中去处理；而异步请求，因为回调方法中就是在子线程中实现的，因此无需另起线程。

<a name="bf223461"></a>
## 3.POST请求
post请求也是有同步和异步的分别。我们这里就以异步情况为例：

```java
    private void PostOKhttp() {
        //创建OkhttpClient对象
        OkHttpClient okHttpClient = new OkHttpClient();
        //创建表单对象
        FormBody.Builder builder = new FormBody.Builder();
        //键值对输入
        builder.add("key","123123123");
        builder.add("cityname","上海");
        //创建request参数，其中需要添加pos方式
        Request request = new Request.Builder()
                .post(builder.build())
                .url("http://api.avatardata.cn/?")
                .build();
        //创建Call接口
        okhttp3.Call call = okHttpClient.newCall(request);
        //异步请求
        call.enqueue(new okhttp3.Callback() {
            @Override
            public void onFailure(okhttp3.Call call, IOException e) {
                Log.i("xw","fail!");
            }

            @Override
            public void onResponse(okhttp3.Call call, okhttp3.Response response) throws IOException {
                Log.i("xw","sucess!" + response.code());
                if (response.isSuccessful()){
                }
            }
        });
    }
```

post方法接收的参数是RequestBody 对象。可以传递如下几种：
<a name="3258f3e2"></a>
#### FormBody对象
FormBody是RequestBody的子类。多用于传递键值对。

```java
public final class FormBody extends RequestBody {
  private static final MediaType CONTENT_TYPE = MediaType.get("application/x-www-form-urlencoded");

  private final List<String> encodedNames;
  private final List<String> encodedValues;

  FormBody(List<String> encodedNames, List<String> encodedValues) {
```

<a name="1e90cc25"></a>
#### JSON对象/File对象
这里关键就是在于MediaType的设置。查看源码MediaType类可以看到：

```java
public final class MediaType {
  private static final String TOKEN = "([a-zA-Z0-9-!#$%&'*+.^_`{|}~]+)";
  private static final String QUOTED = "\"([^\"]*)\"";
  private static final Pattern TYPE_SUBTYPE = Pattern.compile(TOKEN + "/" + TOKEN);
  private static final Pattern PARAMETER = Pattern.compile(
      ";\\s*(?:" + TOKEN + "=(?:" + TOKEN + "|" + QUOTED + "))?");
```

JSON对象：

```java
        //数据类型为JSON类型
        MediaType json = MediaType.get("application/json;charset=utf-8");
        //需要上传的json数据
        String jsonString = "{\"username\":\"lisi\",\"nickname\":\"李四\"}";
        //创建requestBody对象
        RequestBody requestBody = RequestBody.create(json,jsonString);
```

File对象：

```java
        //数据类型为File类型
        MediaType fileType = MediaType.get("File/*");
        //需要上传的File对象
        File file = new File(getExternalCacheDir().toString());
        //创建requestBody对象
        RequestBody requestBody = RequestBody.create(fileType,file);
```

<a name="807f5a36"></a>
#### MultipartBody传递多类型参数
查看MultipartBody也是RequestBody的子类，他的关键方法为：

```java
    /** Add a part to the body. */
    public Builder addPart(RequestBody body) {
      return addPart(Part.create(body));
    }

    /** Add a part to the body. */
    public Builder addPart(@Nullable Headers headers, RequestBody body) {
      return addPart(Part.create(headers, body));
    }

    /** Add a form data part to the body. */
    public Builder addFormDataPart(String name, String value) {
      return addPart(Part.createFormData(name, value));
    }

    /** Add a form data part to the body. */
    public Builder addFormDataPart(String name, @Nullable String filename, RequestBody body) {
      return addPart(Part.createFormData(name, filename, body));
    }

    /** Add a part to the body. */
    public Builder addPart(Part part) {
      if (part == null) throw new NullPointerException("part == null");
      parts.add(part);
      return this;
    }
```

我们可以传递键值对或者键值对和另一个RequestBody一起：

```java
    MultipartBody multipartBody =new MultipartBody.Builder()
    .setType(MultipartBody.FORM)
    .addFormDataPart("groupId",""+groupId)//添加键值对参数
    .addFormDataPart("title","title")
    .addFormDataPart("file",file.getName(),RequestBody.create(MediaType.parse("file/*"), file))//添加文件
    .build();

```

<a name="6ee29e7b"></a>
#### 自定义的RequestBody
这里就需要我们自定义一个子类继承RequestBody，需要重写contentType()、contentLength()、writeTo()。

```java
        //自定义RequestBody，重写writeTo
        RequestBody body = new RequestBody() {
            @Override
            public MediaType contentType() {
                return null;
            }

            @Override
            public void writeTo(BufferedSink sink) throws IOException {
                FileInputStream fio= new FileInputStream(new File(getExternalCacheDir().toString()));
                byte[] buffer = new byte[1024];
                if(fio.read(buffer) != -1){
                    sink.write(buffer);
                }
            }
        };
```

可以看到在writeTo()方法中传递的参数是BufferedSkin，这是Okio包里输入流，有write方法可写入。

<a name="865f3f6c"></a>
## 4.拦截器（Interceptor）
这是Okhttp库中最核心的地方。首先Okhttp的拦截器加载是有序的，他是按照顺序从上到下来加载的。为什么？且看源码：

```java
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    //用户自定义的interceptor
    interceptors.addAll(client.interceptors());
    //RetryAndFollowUpInterceptor
    interceptors.add(new RetryAndFollowUpInterceptor(client));
    //BridgeInterceptor
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    //CacheInterceptor
    interceptors.add(new CacheInterceptor(client.internalCache()));
    //ConnectInterceptor
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
			//用户自定义的networkInterceptors
      interceptors.addAll(client.networkInterceptors());
    }
    //CallServerInterceptor
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, transmitter, null, 0,
        originalRequest, this, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    boolean calledNoMoreExchanges = false;
    try {
      //开始
      Response response = chain.proceed(originalRequest);
```



<a name="RetryAndFollowUpInterceptor"></a>
### RetryAndFollowUpInterceptor
这个拦截器主要是用于重定向:

```java
  @Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Transmitter transmitter = realChain.transmitter();

    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      transmitter.prepareToConnect(request);

      if (transmitter.isCanceled()) {
        throw new IOException("Canceled");
      }

      Response response;
      boolean success = false;
      try {
        response = realChain.proceed(request, transmitter, null);
        success = true;
      } catch (RouteException e) {
        // The attempt to connect via a route failed. The request will not have been sent.
        if (!recover(e.getLastConnectException(), transmitter, false, request)) {
          throw e.getFirstConnectException();
        }
```

<a name="BridgeInterceptor"></a>
### BridgeInterceptor
这个拦截器主要是用于添加请求头header的，其中如果有Accept-Encoding：gzip 字段的时候还会去做压缩处理。

```java
  @Override public Response intercept(Chain chain) throws IOException {
    Request userRequest = chain.request();
    Request.Builder requestBuilder = userRequest.newBuilder();

    RequestBody body = userRequest.body();
    if (body != null) {
      MediaType contentType = body.contentType();
      if (contentType != null) {
        requestBuilder.header("Content-Type", contentType.toString());
      }

      long contentLength = body.contentLength();
      if (contentLength != -1) {
        requestBuilder.header("Content-Length", Long.toString(contentLength));
        requestBuilder.removeHeader("Transfer-Encoding");
      } else {
        requestBuilder.header("Transfer-Encoding", "chunked");
        requestBuilder.removeHeader("Content-Length");
      }
    }

    if (userRequest.header("Host") == null) {
      requestBuilder.header("Host", hostHeader(userRequest.url(), false));
    }

    if (userRequest.header("Connection") == null) {
      requestBuilder.header("Connection", "Keep-Alive");
    }

    // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
    // the transfer stream.
    boolean transparentGzip = false;
    if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
      transparentGzip = true;
      requestBuilder.header("Accept-Encoding", "gzip");
    }
    
    ....
    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);

    if (transparentGzip
        && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        && HttpHeaders.hasBody(networkResponse)) {
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      String contentType = networkResponse.header("Content-Type");
      responseBuilder.body(new RealResponseBody(contentType, -1L, Okio.buffer(responseBody)));
    }

    return responseBuilder.build();

```

<a name="CacheInterceptor"></a>
### CacheInterceptor
这个拦截器主要是用于缓存请求并且可以将响应写入缓存。

```java
  @Override public Response intercept(Chain chain) throws IOException {
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();

    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    if (cache != null) {
      cache.trackResponse(strategy);
    }

    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.
    if (networkRequest == null && cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    Response networkResponse = null;
    try {
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.
    if (cacheResponse != null) {
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache.trackConditionalCacheHit();
        cache.update(cacheResponse, response);
        return response;
      } else {
        closeQuietly(cacheResponse.body());
      }
    }
```


上面完成之后会去获取响应response，然后去判断缓存是否为空，不为空的话会将response存入到缓存中；否则执行下一个拦截器

```java
    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (cache != null) {
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
        // Offer this request to the cache.
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }
```

<a name="ConnectInterceptor"></a>
### ConnectInterceptor
这个拦截器的作用是打开与目标服务器的连接，然后执行下一个拦截器：

```java
  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    Transmitter transmitter = realChain.transmitter();

    // We need the network to satisfy this request. Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    Exchange exchange = transmitter.newExchange(chain, doExtensiveHealthChecks);

    return realChain.proceed(request, transmitter, exchange);
  }
```

<a name="CallServerInterceptor"></a>
### CallServerInterceptor
这个拦截器是链路中的最后一个拦截器，他的作用是向服务器进行网络请求：

```java
  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Exchange exchange = realChain.exchange();
    Request request = realChain.request();

    long sentRequestMillis = System.currentTimeMillis();

    exchange.writeRequestHeaders(request);

    boolean responseHeadersStarted = false;
    Response.Builder responseBuilder = null;
    if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
      // If there's a "Expect: 100-continue" header on the request, wait for a "HTTP/1.1 100
      // Continue" response before transmitting the request body. If we don't get that, return
      // what we did get (such as a 4xx response) without ever transmitting the request body.
      if ("100-continue".equalsIgnoreCase(request.header("Expect"))) {
        exchange.flushRequest();
        responseHeadersStarted = true;
        exchange.responseHeadersStart();
        responseBuilder = exchange.readResponseHeaders(true);
      }

      if (responseBuilder == null) {
        if (request.body() instanceof DuplexRequestBody) {
          // Prepare a duplex body so that the application can send a request body later.
          exchange.flushRequest();
          BufferedSink bufferedRequestBody = Okio.buffer(
              exchange.createRequestBody(request, true));
          request.body().writeTo(bufferedRequestBody);
        } else {
          // Write the request body if the "Expect: 100-continue" expectation was met.
          BufferedSink bufferedRequestBody = Okio.buffer(
              exchange.createRequestBody(request, false));
          request.body().writeTo(bufferedRequestBody);
          bufferedRequestBody.close();
        }
      } else {
        exchange.noRequestBody();
        if (!exchange.connection().isMultiplexed()) {
          // If the "Expect: 100-continue" expectation wasn't met, prevent the HTTP/1 connection
          // from being reused. Otherwise we're still obligated to transmit the request body to
          // leave the connection in a consistent state.
          exchange.noNewExchangesOnConnection();
        }
      }
    } else {
      exchange.noRequestBody();
    }

    if (!(request.body() instanceof DuplexRequestBody)) {
      exchange.finishRequest();
    }

    if (!responseHeadersStarted) {
      exchange.responseHeadersStart();
    }

    if (responseBuilder == null) {
      responseBuilder = exchange.readResponseHeaders(false);
    }

    Response response = responseBuilder
        .request(request)
        .handshake(exchange.connection().handshake())
        .sentRequestAtMillis(sentRequestMillis)
        .receivedResponseAtMillis(System.currentTimeMillis())
        .build();

    int code = response.code();
    if (code == 100) {
      // server sent a 100-continue even though we did not request one.
      // try again to read the actual response
      response = exchange.readResponseHeaders(false)
          .request(request)
          .handshake(exchange.connection().handshake())
          .sentRequestAtMillis(sentRequestMillis)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();

      code = response.code();
    }

    exchange.responseHeadersEnd(response);
```


     其实有点类似网络分层的情况，每层处理自身的实现即可。每个Interceptor处理自己的实现并不会去影响，同时这也形成了链路复用。<br />

<a name="09b4cc07"></a>
# 流程图
Okhttp 的方法调用流程大致如下（以异步请求为例）：

![](https://cdn.nlark.com/yuque/__puml/5863462dd768f6395d74d9e4fa3b3cbf.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aactor%20%22%E7%BD%91%E7%BB%9C%22%20as%20Start%0Aparticipant%20%22OkttpClient%22%20as%20Client%0Aparticipant%20%22RealCall%22%20as%20RealCall%20%0Aparticipant%20%22Dispatcher%22%20as%20Dispatcher%0Aparticipant%20%22Response%22%20as%20Response%0Aparticipant%20%22Interceptor%22%20as%20Interceptor%0Aparticipant%20%22RealInterceptorChain%22%20as%20RealInterceptorChain%0A%0Aactivate%20Start%0A%0AStart%20-%3E%20Client%3A%20%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%0Aactivate%20Browser%0A%0AClient%20-%3E%20Client%3A%20new%20OkhttpClient%20%0Anote%20right%20of%20Client%3A%20%E6%9E%84%E9%80%A0%E8%80%85%E6%A8%A1%E5%BC%8F%20builder%0A%0AClient%20-%3E%20Client%3A%20newCall%0Aactivate%20Client%0A%0AClient%20-%3E%20RealCall%3A%20newRealCall%0Aactivate%20RealCall%0A%0ARealCall%20-%3E%20RealCall%3A%20enqueue%0Anote%20right%20of%20RealCall%3A%20%E5%BC%82%E6%AD%A5%E8%AF%B7%E6%B1%82%0A%0ARealCall%20-%3E%20Dispatcher%3A%20enqueue%0Aactivate%20Dispatcher%0Anote%20right%20of%20Dispatcher%3A%E5%B0%86%E7%94%A8%E6%88%B7%E6%8E%A5%E5%8F%A3responseCallback%E5%AF%B9%E8%B1%A1%E5%B0%81%E8%A3%85%E6%88%90%E4%B8%80%E4%B8%AAAsyncCall%E5%AF%B9%E8%B1%A1%0A%0ADispatcher%20-%3E%20Dispatcher%3A%20promoteAndExecute%0Aactivate%20Dispatcher%0Anote%20right%20of%20Dispatcher%3A%E8%8B%A5%E7%BA%BF%E7%A8%8B%E6%B1%A0%E4%B8%AD%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%9C%AA%E8%B6%85%E8%BF%87%E6%9C%80%E5%A4%A7%E5%AE%B9%E9%87%8F%EF%BC%8C%E5%88%99%E5%B0%86AsyncCall%E5%AF%B9%E8%B1%A1%E5%8A%A0%E5%85%A5%0A%0ADispatcher%20-%3E%20RealCall%3A%20executeOn%0Aactivate%20RealCall%0Anote%20right%20of%20RealCall%3A%E9%81%8D%E5%8E%86%E8%BF%90%E8%A1%8CexecutableCalls%E4%B8%AD%E7%9A%84%E7%BA%BF%E7%A8%8B%0A%0ARealCall%20-%3E%20RealCall%3A%20execute%0Aactivate%20RealCall%0A%0ARealCall%20-%3E%20RealCall%3A%20getResponseWithInterceptorChain%0Aactivate%20RealCall%0Anote%20right%20of%20RealCall%3A%E8%8B%A5%E6%98%AF%E5%90%8C%E6%AD%A5%E8%AF%B7%E6%B1%82execute%E5%88%99%E4%BC%9A%E7%9B%B4%E6%8E%A5%E8%B0%83%E7%94%A8%E3%80%82%E8%BF%99%E9%87%8C%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E6%8B%A6%E6%88%AA%E5%99%A8%E7%9A%84%E5%8A%A0%E8%BD%BD%E4%BC%9A%E4%BE%9D%E6%AC%A1%E6%8C%89%E5%BA%8F%E5%8A%A0%E8%BD%BD%E5%88%B0interceptors%E5%BA%8F%E5%88%97%E4%B8%AD%0A%0ARealCall%20-%3E%20Interceptor%3Aproceed%0Aactivate%20Interceptor%0A%0AInterceptor%20-%3E%20RealInterceptorChain%3Aproceed%0Aactivate%20RealInterceptorChain%0A%0ARealInterceptorChain%20-%3E%20RealInterceptorChain%3Aproceed%0Aactivate%20RealInterceptorChain%0Anote%20right%20of%20RealInterceptorChain%3A%E9%93%BE%E5%BC%8F%E8%B0%83%E7%94%A8%0A%0ARealInterceptorChain%20-%3E%20Interceptor%3Aintercept%0Aactivate%20Interceptor%0Anote%20right%20of%20Interceptor%3A%E4%BE%9D%E6%AC%A1%E8%B0%83%E7%94%A8interceptors%E5%BA%8F%E5%88%97%E4%B8%AD%E7%9A%84%E6%8B%A6%E6%88%AA%E5%99%A8intercept%E6%96%B9%E6%B3%95%0A%0A%40enduml)

<a name="9e048bb1"></a>
# 补充
<a name="78d6c791"></a>
### 1.有报出BootstrapMethodError

```java
    03-12 10:06:10.596 21989 21989 E AndroidRuntime: java.lang.BootstrapMethodError: Exception from call site #7 bootstrap method

    03-12 10:06:10.596 21989 21989 E AndroidRuntime:        at okhttp3.internal.Util.<clinit>(Util.java:87)

    03-12 10:06:10.596 21989 21989 E AndroidRuntime:        at okhttp3.internal.Util.immutableList(Util.java:234)

    03-12 10:06:10.596 21989 21989 E AndroidRuntime:        at okhttp3.OkHttpClient.<clinit>(OkHttpClient.java:127)

    03-12 10:06:10.596 21989 21989 E AndroidRuntime:        at okhttp3.OkHttpClient$Builder.<init>(OkHttpClient.java:482)
```

原因：jdk中的方法不存在导致error

解决方案：<br />1. 将依赖的OKhttp的版本号降低；<br />或<br />2. 在build.gradle中添加：

```java
   compileOptions {

       sourceCompatibility JavaVersion.VERSION_1_8

       targetCompatibility JavaVersion.VERSION_1_8

   }
```

<a name="55875d99"></a>
### 2.Okhttp资料
[https://square.github.io/okhttp/](https://square.github.io/okhttp/)  :官方地址<br />[https://github.com/square/okhttp](https://github.com/square/okhttp) ： 官方github地址

最后，有不足的地方还希望大佬们多指教指正。感谢。
