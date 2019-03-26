# Glide-原理篇

<a name="e05dce83"></a>
# 简介
Glide是一个快速高效的Android图片加载库，注重于平滑的滚动。Glide提供了易用的API，高性能、可扩展的图片解码管道（`decode pipeline`），以及自动的资源池技术。<br />虽然Glide 的主要目标是让任何形式的图片列表的滚动尽可能地变得更快、更平滑，但实际上，Glide几乎能满足你对远程图片的拉取/缩放/显示的一切需求。这里是基于目前最新的Glide4.9.0来介绍。<br /><br />
<a name="704f29e0"></a>
# 基本用法
<a name="ab5b6106"></a>
## 1.添加依赖
目前最新的是4.9.0

```java
dependencies {
  implementation 'com.github.bumptech.glide:glide:4.9.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.9.0'
}
```


<a name="104beade"></a>
## 2.加载图片

Glide的图片加载方法很简单，通过context然后加载url，将其放到imageView中显示出来。当然由于Glide是有缓存机制的，因此这个加载过程在第一次是比较明显的，之后的加载会很快的。

```java
    Glide.with(mContext).load(URL).into(imageView);
```


<a name="51f7c454"></a>
### with()
先是调用Glide.with()。这里with()方法中可传参有context、activity、fragment、FragmentActivity、View。with()方法主要是先去初始化Glide，并且创建RequestMannager对象，并绑定了SupportRequestManagerFragment生命周期函数。

来看源码如下：

```java
  /**
   * Begin a load with Glide by passing in a context.
   *
   * <p> Any requests started using a context will only have the application level options applied
   * and will not be started or stopped based on lifecycle events. In general, loads should be
   * started at the level the result will be used in. If the resource will be used in a view in a
   * child fragment, the load should be started with {@link #with(android.app.Fragment)}} using that
   * child fragment. Similarly, if the resource will be used in a view in the parent fragment, the
   * load should be started with {@link #with(android.app.Fragment)} using the parent fragment. In
   * the same vein, if the resource will be used in a view in an activity, the load should be
   * started with {@link #with(android.app.Activity)}}. </p>
   *
   * <p> This method is appropriate for resources that will be used outside of the normal fragment
   * or activity lifecycle (For example in services, or for notification thumbnails). </p>
   *
   * @param context Any context, will not be retained.
   * @return A RequestManager for the top level application that can be used to start a load.
   * @see #with(android.app.Activity)
   * @see #with(android.app.Fragment)
   * @see #with(android.support.v4.app.Fragment)
   * @see #with(android.support.v4.app.FragmentActivity)
   */
  @NonNull
  public static RequestManager with(@NonNull Context context) {
    return getRetriever(context).get(context);
  }
```


<br />详细的流程实现如下：

![](https://cdn.nlark.com/yuque/__puml/2059f833cc7fee4b5d7ff401798b4965.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aparticipant%20%22Glide%22%20as%20Glide%0Aparticipant%20%22RequestManagerRetriever%22%20as%20RequestManagerRetriever%0Aparticipant%20%22RequestManager%22%20as%20RequestManager%0Aparticipant%20%22Lifecycle%22%20as%20Lifecycle%20%23orange%0Aparticipant%20%22ActivityFragmentLifecycle%22%20as%20ActivityFragmentLifecycle%0Aparticipant%20%22LifecycleListener%22%20as%20LifecycleListener%20%23orange%0Aparticipant%20%22SupportRequestManagerFragment%22%20as%20SupportRequestManagerFragment%0A%0AStart%20-%3E%20Glide%3A%20with%0Aactivate%20Glide%0A%0AGlide%20-%3E%20Glide%3A%20getRetriever%0Aactivate%20Glide%0A%0AGlide%20-%3E%20Glide%3A%20get%28Context%29%0Aactivate%20Glide%0A%0AGlide%20-%3E%20Glide%3A%20checkAndInitializeGlide%0Aactivate%20Glide%0A%0AGlide%20-%3E%20Glide%3A%20initializeGlide%0Aactivate%20Glide%0Anote%20right%20of%20Glide%3A%20%E8%8E%B7%E5%8F%96%E8%87%AA%E5%AE%9A%E4%B9%89GlideModule%3B%E5%88%9D%E5%A7%8B%E5%8C%96Glide%EF%BC%8C%E4%B8%BAmodule%E6%B3%A8%E5%86%8C%E7%BB%84%E4%BB%B6%E5%92%8C%E5%BA%94%E7%94%A8%E5%8F%82%E6%95%B0%E8%AE%BE%E7%BD%AE%0A%0AGlide%20-%3E%20RequestManagerRetriever%3A%20get%0Aactivate%20RequestManagerRetriever%0Anote%20right%20of%20RequestManagerRetriever%3A%20%E6%A0%B9%E6%8D%AEcontext%E7%B1%BB%E5%9E%8B%E8%B0%83%E7%94%A8%E4%B8%8D%E5%90%8C%E7%9A%84get%28%29%E6%96%B9%E6%B3%95%0A%0ARequestManagerRetriever%20-%3E%20RequestManagerRetriever%3A%20get%28activity%29%0Anote%20right%20of%20RequestManagerRetriever%3A%E4%BB%A5activity%E4%B8%BA%E4%BE%8B%EF%BC%8C%E8%8B%A5%E6%98%AF%E5%90%8E%E5%8F%B0%E7%BA%BF%E7%A8%8B%EF%BC%8C%E5%88%99%E4%BC%9A%E8%B0%83%E7%94%A8%E5%88%B0application%E7%9A%84context%EF%BC%9B%E8%8B%A5%E4%B8%8D%E6%98%AF%E5%90%8E%E5%8F%B0%E7%BA%BF%E7%A8%8B%E5%88%99%E8%B0%83%E7%94%A8fragmentGet%28%29%0Aactivate%20RequestManagerRetriever%0A%0ARequestManagerRetriever%20-%3E%20RequestManagerRetriever%3A%20fragmentGet%0Aactivate%20RequestManagerRetriever%0Anote%20right%20of%20RequestManagerRetriever%3A%E8%8B%A5%E4%B8%BAFragmentActivity%2Ffragment%EF%BC%8C%E5%88%99%E4%BC%9A%E8%B0%83%E7%94%A8supportFragmentGet%28%29%0A%0ARequestManagerRetriever%20-%3E%20RequestManagerRetriever%3A%20getRequestManagerFragment%0Aactivate%20RequestManagerRetriever%0Anote%20right%20of%20RequestManagerRetriever%3A%E8%8E%B7%E5%BE%97SupportRequestManagerFragment%0A%0ARequestManagerRetriever%20-%3E%20SupportRequestManagerFragment%3A%20SupportRequestManagerFragment%0Aactivate%20SupportRequestManagerFragment%0A%0ARequestManagerRetriever%20-%3E%20RequestManagerRetriever%3A%20build%0Adeactivate%20RequestManagerRetriever%0Anote%20right%20of%20RequestManagerRetriever%3A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AARequestManager%E5%AF%B9%E8%B1%A1%0A%0ARequestManager%20-%3E%20RequestManager%3A%20RequestManager%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0%0Adeactivate%20RequestManager%0Anote%20right%20of%20RequestManager%3AaddListener%0A%0ARequestManager%20-%3E%20Lifecycle%3A%20addListener%0Adeactivate%20Lifecycle%0Anote%20right%20of%20Lifecycle%3A%E6%B7%BB%E5%8A%A0%E7%9B%91%E5%90%AC%E5%99%A8%0A%0ALifecycle%20-%3E%20ActivityFragmentLifecycle%3A%20addListener%0Adeactivate%20ActivityFragmentLifecycle%0Anote%20right%20of%20Lifecycle%3A%E6%B7%BB%E5%8A%A0%E7%9B%91%E5%90%AC%E5%99%A8%E7%9A%84%E5%AE%9E%E7%8E%B0%0A%0ASupportRequestManagerFragment%20-%3E%20ActivityFragmentLifecycle%3A%20onStart%28%29%2FonStop%28%29%0Adeactivate%20ActivityFragmentLifecycle%0A%0AActivityFragmentLifecycle%20-%3E%20LifecycleListener%3A%20onStart%28%29%2FonStop%28%29%0Adeactivate%20LifecycleListener%0A%0ALifecycleListener%20-%3E%20RequestManager%3A%20onStart%28%29%2FonStop%28%29%0Adeactivate%20RequestManager%0Anote%20right%20of%20RequestManager%3A%20%E5%90%AF%E5%8A%A8%E6%88%96%E6%9A%82%E5%81%9C%E8%AF%B7%E6%B1%82%0A%0A%0A%40enduml)<br /><br />getRequestManagerFragment()方法的调用流程如下：<br />
<br />首先先根据tag调用findFragmentByTag()查找fragment；若找不到，则从SupportRequestManagerFragments队列中找；若还找不到，则会新建一个SupportRequestManagerFragment，并将这个新建的fragment放入到SupportRequestManagerFragments队列中。<br />调用lifecycleListener的onStart()，将Fragment和Activity绑定。之后使用handler发送ID_REMOVE_SUPPORT_FRAGMENT_MANAGER消息将从队列中移除Fragment<br />
<br />
<br />build()方法的实现流程如下：

通过getRequestManagerFragment()获取到SupportRequestManagerFragment对象，然后获取SupportRequestManagerFragment对象的Lifecycle对象。然后调用到build()方法去创建RequestManager对象的时候会将这个Lifecycle对象作为参数传入。<br />而RequestManager类实现了LifecycleListener接口，通过lifecycle.addListener(**this**)方法添加了监听器 ，并且实现接口方法onStart()/onStop()。<br />因此当SupportRequestManagerFragment的生命周期发生变化的时候，就会去调用Lifecycle对象中的生命周期方法，而该生命周期方法的实现是在RequestManager。这就使得Fragment生命周期和Glide请求之间形成互相监听绑定。原因：由于Glide无法监听Activity状态，而fragment和activity生命周期是同步的，因此通过一个Fragment 子类SupportRequestManagerFragment来实现监听，这样如果activity销毁了，那么就能通知调用stop方法停止加载了。

<a name="9bb4b71e"></a>
### load()
接着来看RequestManager.load()，源码如下：

```java
  /**
   * Equivalent to calling {@link #asDrawable()} and then {@link RequestBuilder#load(String)}.
   *
   * @return A new request builder for loading a {@link Drawable} using the given model.
   */
  @NonNull
  @CheckResult
  @Override
  public RequestBuilder<Drawable> load(@Nullable String string) {
    return asDrawable().load(string);
  }
```

也可以看到load()方法中传入的参数有多个类型：bitmap、Drawable、String、Uri、File、Integer（如resourceid）。最后调用到RequestBuilder类中的loadGeneric()方法，将传入的参数赋值给RequestBuiler.model，同时isModelSet置为true，最终返回的是一个RequestBuilder对象。

<a name="ee0e3c5e"></a>
### into()
最后看RequestBuilder.into()，源码如下：

```java
  /**
   * Sets the {@link ImageView} the resource will be loaded into, cancels any existing loads into
   * the view, and frees any resources Glide may have previously loaded into the view so they may be
   * reused.
   *
   * @see RequestManager#clear(Target)
   *
   * @param view The view to cancel previous loads for and load the new resource into.
   * @return The
   * {@link com.bumptech.glide.request.target.Target} used to wrap the given {@link ImageView}.
   */
  @NonNull
  public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {
    Util.assertMainThread();
    Preconditions.checkNotNull(view);

    BaseRequestOptions<?> requestOptions = this;
    if (!requestOptions.isTransformationSet()
        && requestOptions.isTransformationAllowed()
        && view.getScaleType() != null) {
      // Clone in this method so that if we use this RequestBuilder to load into a View and then
      // into a different target, we don't retain the transformation applied based on the previous
      // View's scale type.
      switch (view.getScaleType()) {
        case CENTER_CROP:
          requestOptions = requestOptions.clone().optionalCenterCrop();
          break;
        case CENTER_INSIDE:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case FIT_CENTER:
        case FIT_START:
        case FIT_END:
          requestOptions = requestOptions.clone().optionalFitCenter();
          break;
        case FIT_XY:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case CENTER:
        case MATRIX:
        default:
          // Do nothing.
      }
    }
```

详细的流程实现如下：

![](https://cdn.nlark.com/yuque/__puml/5aad84b24b14a644d1e0cc2618d7f27c.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aparticipant%20%22RequestBuilder%22%20as%20RequestBuilder%0Aparticipant%20%22GlideContext%22%20as%20GlideContext%0Aparticipant%20%22ImageViewTargetFactory%22%20as%20ImageViewTargetFactory%0Aparticipant%20%22SingleRequest%22%20as%20SingleRequest%0Aparticipant%20%22RequestManager%22%20as%20RequestManager%0Aparticipant%20%22RequestTracker%22%20as%20RequestTracker%0Aparticipant%20%22ImageViewTarget%22%20as%20ImageViewTarget%0Aparticipant%20%22ImageView%22%20as%20ImageView%0A%0A%0A%0ARequestBuilder%20-%3E%20RequestBuilder%3A%20into%0Anote%20right%20of%20RequestBuilder%3AgetScaleType%28%29%E8%8E%B7%E5%8F%96ImageView%E7%9A%84ScaleType%EF%BC%8C%E5%B9%B6%E6%A0%B9%E6%8D%AEScaleType%E6%9D%A5%E7%A1%AE%E5%AE%9A%E8%B5%84%E6%BA%90%E7%9A%84%E8%BE%B9%E7%95%8C%0Aactivate%20RequestBuilder%0A%0ARequestBuilder%20--%3E%20GlideContext%3A%20buildImageViewTarget%0Aactivate%20GlideContext%0A%0AGlideContext%20--%3E%20ImageViewTargetFactory%3A%20buildTarget%0Aactivate%20ImageViewTargetFactory%0Anote%20right%20of%20ImageViewTargetFactory%3A%E9%80%9A%E8%BF%87clazz%E6%9D%A5%E5%88%A4%E6%96%AD%E4%BC%A0%E5%85%A5%E7%B1%BB%E5%9E%8BBitmap%E3%80%81Drawable%0A%0ARequestBuilder%20-%3E%20RequestBuilder%3A%20into%0Aactivate%20RequestBuilder%0A%0ARequestBuilder%20--%3E%20RequestBuilder%3A%20buildRequest%0Aactivate%20RequestBuilder%0A%0ARequestBuilder%20--%3E%20RequestBuilder%3A%20buildRequestRecursive%0A%0ARequestBuilder%20--%3E%20RequestBuilder%3A%20buildThumbnailRequestRecursive%0Anote%20right%20of%20RequestBuilder%3A%E7%BC%A9%E7%95%A5%E5%9B%BEBuilder%0A%0ARequestBuilder%20--%3E%20RequestBuilder%3A%20obtainRequest%0A%0ARequestBuilder%20--%3E%20SingleRequest%3Aobtain%0Aactivate%20SingleRequest%0A%0ASingleRequest%20--%3E%20SingleRequest%3Ainit%0Anote%20right%20of%20SingleRequest%3A%E5%88%9D%E5%A7%8B%E5%8C%96%E7%94%9F%E6%88%90%E4%B8%80%E4%B8%AARequest%E5%AF%B9%E8%B1%A1%E5%B9%B6%E8%BF%94%E5%9B%9E%0A%0ARequestBuilder%20-%3E%20RequestManager%3Atrack%0A%0ARequestManager%20-%3E%20RequestTracker%3ArunRequest%0Anote%20right%20of%20RequestTracker%3A%E8%AF%B7%E6%B1%82%E4%B8%8D%E6%98%AF%E6%9A%82%E5%81%9C%E7%8A%B6%E6%80%81%0A%0ARequestTracker%20-%3E%20SingleRequest%3Abegin%0Aactivate%20SingleRequest%0Anote%20right%20of%20SingleRequest%3Amodel%E4%B8%BA%E7%A9%BA%0A%0ASingleRequest%20-%3E%20SingleRequest%3AonLoadFailed%0Aactivate%20SingleRequest%0A%0ASingleRequest%20-%3E%20SingleRequest%3AsetErrorPlaceholder%0A%0ASingleRequest%20-%3E%20ImageViewTarget%3AonLoadFailed%0Aactivate%20ImageViewTarget%0A%0AImageViewTarget%20-%3E%20ImageViewTarget%3AsetDrawable%0Aactivate%20ImageViewTarget%0A%0AImageViewTarget%20-%3E%20ImageView%3AsetImageDrawable%0Anote%20right%20of%20ImageView%3A%E6%98%BE%E7%A4%BA%E7%A9%BA%E5%9B%BE%E7%89%87%0A%0A%40enduml)
上面的流程图先到SingleRequest@begin()中的这段代码：

```java
  public synchronized void begin() {
    assertNotCallingCallbacks();
    stateVerifier.throwIfRecycled();
    startTime = LogTime.getLogTime();
    if (model == null) {
      if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
        width = overrideWidth;
        height = overrideHeight;
      }
      // Only log at more verbose log levels if the user has set a fallback drawable, because
      // fallback Drawables indicate the user expects null models occasionally.
      int logLevel = getFallbackDrawable() == null ? Log.WARN : Log.DEBUG;
      onLoadFailed(new GlideException("Received null model"), logLevel);
      return;
    }

   ...
```

我们接着继续：

![](https://cdn.nlark.com/yuque/__puml/3678d63646a3b18a662b73936df5d904.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aparticipant%20%22SingleRequest%22%20as%20SingleRequest%0Aparticipant%20%22Engine%22%20as%20Engine%0Aparticipant%20%22EngineJob%22%20as%20EngineJob%0Aparticipant%20%22DecodeJob%22%20as%20DecodeJob%0Aparticipant%20%22DataFetcherGenerator%22%20as%20DataFetcherGenerator%20%23orange%0Aparticipant%20%22SourceGenerator%22%20as%20SourceGenerator%0Aparticipant%20%22DataFetcher%22%20as%20DataFetcher%20%23orange%0Aparticipant%20%22HttpUrlFetcher%22%20as%20HttpUrlFetcher%0Aparticipant%20%22DataFetcherGenerator%22%20as%20DataFetcherGenerator%20%23orange%0Aparticipant%20%22LoadPath%22%20as%20LoadPath%0Aparticipant%20%22DecodePath%22%20as%20DecodePath%0Aparticipant%20%22ResourceDecoder%22%20as%20ResourceDecoder%20%23orange%0Aparticipant%20%22StreamBitmapDecoder%22%20as%20StreamBitmapDecoder%0Aparticipant%20%22Downsampler%22%20as%20Downsampler%0A%0A%0ASingleRequest%20-%3E%20SingleRequest%20%3A%20begin%0Aactivate%20SingleRequest%0A%0ASingleRequest%20-%3E%20SingleRequest%20%3A%20onSizeReady%0Aactivate%20SingleRequest%0A%0ASingleRequest%20-%3E%20Engine%20%3A%20load%0Aactivate%20Engine%0Anote%20right%20of%20Engine%3A%20engineJob%E7%94%A8%E6%9D%A5%E7%AE%A1%E7%90%86%E8%B4%9F%E8%BD%BD%EF%BC%8CDecodeJob%E8%B4%9F%E8%B4%A3%E8%A7%A3%E7%A0%81%0A%0AEngine%20-%3E%20EngineJob%20%3A%20start%0Anote%20right%20of%20EngineJob%20%3A%20decodeJob%20Runnable%E6%8E%A5%E5%8F%A3%E5%AE%9E%E7%8E%B0%0Aactivate%20EngineJob%0A%0AEngineJob%20-%3E%20DecodeJob%20%3Arun%0A%0ADecodeJob%20-%3E%20DecodeJob%20%3A%20runWrapped%0Aactivate%20DecodeJob%0A%0ADecodeJob%20-%3E%20DecodeJob%20%3A%20runGenerators%0Aactivate%20DecodeJob%0Anote%20right%20of%20DecodeJob%20%3A%20INITIALIZE%2FSWITCH_TO_SOURCE_SERVICE%0A%0ADecodeJob%20-%3E%20DataFetcherGenerator%3AstartNext%0Anote%20right%20of%20DataFetcherGenerator%20%3A%20%E6%8E%A5%E5%8F%A3%E5%87%BD%E6%95%B0%EF%BC%8C%E5%AE%9E%E7%8E%B0%E7%B1%BB%E6%9C%893%E4%B8%AA%E7%94%9F%E6%88%90%E5%99%A8%0Aactivate%20DataFetcherGenerator%0A%0ADataFetcherGenerator%20-%3E%20SourceGenerator%20%3A%20startNext%0Aactivate%20SourceGenerator%0A%0ASourceGenerator%20-%3E%20DataFetcher%20%3A%20loadData%0Anote%20right%20of%20DataFetcher%20%3A%20%E6%8E%A5%E5%8F%A3%E5%87%BD%E6%95%B0%0Aactivate%20DataFetcher%0A%0ADataFetcher%20-%3E%20HttpUrlFetcher%20%3A%20loadData%0Aactivate%20HttpUrlFetcher%0A%0AHttpUrlFetcher%20--%3E%20HttpUrlFetcher%20%3A%20loadDataWithRedirects%0Anote%20right%20of%20HttpUrlFetcher%20%3A%20%E9%80%9A%E8%BF%87HttpURLConnection%E6%9D%A5%E8%8E%B7%E5%8F%96%E7%BD%91%E7%BB%9C%E5%9B%BE%E7%89%87%E8%B5%84%E6%BA%90%20%E5%B9%B6%E8%BF%94%E5%9B%9E%E8%BE%93%E5%85%A5%E6%B5%81%0A%0AHttpUrlFetcher%20-%3E%20DataFetcher%20%3A%20onDataReady%0A%0ADataFetcher%20-%3E%20SourceGenerator%20%3A%20onDataReady%0A%0ASourceGenerator%20-%3E%20DataFetcherGenerator%20%3A%20onDataFetcherReady%0Anote%20right%20of%20DataFetcherGenerator%20%3A%20%E6%8E%A5%E5%8F%A3%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0%0A%0ADataFetcherGenerator%20-%3E%20DecodeJob%20%3A%20onDataFetcherReady%0Aactivate%20DecodeJob%0A%0ADecodeJob%20-%3E%20DecodeJob%20%3A%20decodeFromRetrievedData%0Aactivate%20DecodeJob%0A%0ADecodeJob%20--%3E%20DecodeJob%20%3A%20decodeFromData%0Aactivate%20DecodeJob%0A%0ADecodeJob%20--%3E%20DecodeJob%20%3A%20decodeFromFetcher%0Aactivate%20DecodeJob%0A%0ADecodeJob%20--%3E%20DecodeJob%20%3A%20runLoadPath%0A%0ADecodeJob%20--%3E%20LoadPath%20%3A%20load%0Aactivate%20LoadPath%0A%0ALoadPath%20--%3E%20LoadPath%20%3A%20loadWithExceptionList%0Aactivate%20LoadPath%0A%0ALoadPath%20--%3E%20DecodePath%20%3A%20decode%0Aactivate%20DecodePath%0A%0ADecodePath%20--%3E%20DecodePath%20%3A%20decodeResource%0Aactivate%20DecodePath%0A%0ADecodePath%20--%3E%20DecodePath%20%3A%20decodeResourceWithList%0Aactivate%20DecodePath%0A%0ADecodePath%20--%3E%20ResourceDecoder%20%3A%20decode%0Anote%20right%20of%20ResourceDecoder%20%3A%20%E6%8E%A5%E5%8F%A3%E5%87%BD%E6%95%B0%0A%0AResourceDecoder%20-%3E%20StreamBitmapDecoder%3Adecode%0Anote%20right%20of%20StreamBitmapDecoder%20%3A%20Bitmap%E8%B5%84%E6%BA%90%0Aactivate%20StreamBitmapDecoder%0A%0AStreamBitmapDecoder%20-%3E%20Downsampler%20%3A%20decode%0Aactivate%20Downsampler%0A%0ADownsampler%20--%3E%20Downsampler%20%3A%20decodeFromWrappedStreams%0Anote%20right%20of%20Downsampler%3A%20%E5%B0%86%E8%BE%93%E5%85%A5%E6%B5%81%E8%A7%A3%E7%A0%81%E6%88%90Bitamap%E5%AF%B9%E8%B1%A1%0A%0ADecodeJob%20-%3E%20DecodeJob%20%3A%20notifyEncodeAndRelease%0A%0A%40enduml)
  部分方法说明：

1. ImageViewTargetFactory@buildTarget() ,传参clazz的传递是其实就是Bitmap.class/Drawable.class/File.class

![](https://cdn.nlark.com/yuque/__puml/d5206b96a09152c68a022f615b16a57b.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aparticipant%20%22RequestManager%22%20as%20RequestManager%0Aparticipant%20%22RequestBuilder%22%20as%20RequestBuilder%0A%0ARequestManager%20-%3E%20RequestManager%20%3A%20asBitmap%2FasDrawable%2FasFile%0Aactivate%20RequestManager%0Anote%20right%20of%20RequestManager%3A%20%E8%BF%99%E9%87%8C%E4%BC%A0%E5%8F%82%E7%9A%84%E5%88%86%E5%88%AB%E6%98%AFBitmap.class%2FDrawable.class%2FFile.class%0A%0ARequestManager%20-%3E%20RequestManager%20%3A%20as%0Aactivate%20RequestManager%0Anote%20right%20of%20RequestManager%3A%20%E8%BF%99%E9%87%8C%E5%B0%B1%E4%BC%9A%E5%B0%86%E4%B8%8A%E4%B8%80%E4%B8%AA%E4%BC%A0%E5%8F%82%E5%AE%9A%E4%B9%89%E4%B8%BAresourceClass%0A%0ARequestManager%20-%3E%20RequestBuilder%3A%20new%0Adeactivate%20RequestBuilder%0Anote%20right%20of%20RequestManager%3A%E5%B0%86resourceClass%E8%B5%8B%E5%80%BC%E7%BB%99transcodeClass%0A%0A%40enduml)
1. SingleRequest@setErrorPlaceholder()

这里就能看到，先是判断model是否为空，为空则调用getFallbackDrawable(),获取fallbackDrawable；如果没有获取到fallbackDrawable，则会调用getErrorDrawable()，获取errorDrawable；如果没有获取到errorDrawable，则会调用getPlaceholderDrawable(),获取placeholderDrawable。<br />      发现了没有:这里就是占位符、错误符、后备回调符的加载实现的地方。

```java
  private synchronized void setErrorPlaceholder() {
    if (!canNotifyStatusChanged()) {
      return;
    }

    Drawable error = null;
    if (model == null) {
      error = getFallbackDrawable();
    }
    // Either the model isn't null, or there was no fallback drawable set.
    if (error == null) {
      error = getErrorDrawable();
    }
    // The model isn't null, no fallback drawable was set or no error drawable was set.
    if (error == null) {
      error = getPlaceholderDrawable();
    }
    target.onLoadFailed(error);
  }
```


1. SourceGenerator@startNext()

```java
  @Override
  public boolean startNext() {
    ......
    while (!started && hasNextModelLoader()) {
      loadData = helper.getLoadData().get(loadDataListIndex++);
      ....
    }
    return started;
  }
```
这里主要看一下loadData的获取流程：<br />通过getLoadData()获取LoadData，这里仅以输入为string类型的URL为例（当然输入类型不一样，所调用的fetcher也不同，但是实现逻辑基本是一样的）。

```
--> DecodeHelper.getLoadData()

 --> Registry.getModelLoaders()

 --> ModelLoaderRegistry.getModelLoaders()

 --> ModelLoaderRegistry.getModelLoadersForClass()
	
	这里是通过遍历multiModelLoaderFactory

 --> Glide.Glide().append()

	Glide的构造函数中会调用Registry.append()

 --> ModelLoaderRegistry.append()

	这时我们来看，如果我们传入的URL，那么这是一个String，那有可能是：

    .append(String.class, InputStream.class, new DataUrlLoader.StreamFactory<String>())
    .append(String.class, InputStream.class, new StringLoader.StreamFactory())
    .append(String.class, ParcelFileDescriptor.class, new StringLoader.FileDescriptorFactory())

	先来看DataUrlLoader的handlers()方法，需要String以"data:image"开头，很明显我们传入的URL不是这个样子的，第三个是序列化的String也不是的。那只能是第二个StringLoader

 --> StringLoader.StreamFactory()

	工厂类创建一个<Uri.class, InputStream.class>的modelLoader，这时再去Glide的构造函数中查找可以看到.append(Uri.class, InputStream.class, new UrlUriLoader.StreamFactory())；同样去看他的工厂类实现是创建一个<GlideUrl.class, InputStream.class>的modelLoader,再去查找：
	.append(GlideUrl.class, InputStream.class, new HttpGlideUrlLoader.Factory())

	因此我们找到了modelLoader为HttpGlideUrlLoader

--> ModelLoader.buildLoadData()

--> HttpGlideUrlLoader.buildLoadData()

	这里获取传递过来的String，并且将DataFetcher设置为HttpUrlFetcher。这就使得后面在接口DataFetcher.loadData()方法的实现类是HttpUrlFetcher.loadData()
```


1. Downsampler@decodeFromWrappedStreams()

这个方法就是真正的解码实现。

<a name="17b56eb5"></a>
## 3.占位符
Glide允许用户指定三种不同类型的占位符，分别在三种不同场景使用：占位符、错误符、后备回调符。<br /><br />
<a name="760b202d"></a>
### 占位符placeload：  
**占位符是当请求正在执行时被展示的 Drawable 。当请求成功完成时，占位符会被请求到的资源替换。**如果被请求的资源是从内存中加载出来的，那么占位符可能根本不会被显示。如果请求失败并且没有设置error Drawable ，则占位符将被持续展示。类似地，如果请求的url/model为null，并且 error Drawable和 fallback drawable 都没有设置，那么占位符也会继续显示。<br /><br /><br />实现如下：

```java
RequestOptions requestOptions = new RequestOptions().placeholder(R.mipmap.loading);
Glide.with(context)
     .load(IMAGE_URL)
     .apply(requestOptions)
     .into(imageView);
```
<br />
<a name="e8c9801c"></a>
### 错误符error：
**error Drawable在请求永久性失败时展示。**error Drawable 同样也在请求的url/model为null，且并没有设置 fallback drawable时展示。<br /><br /><br />实现如下：

```java
    RequestOptions requestOptions = new RequestOptions().placeholder(R.mipmap.loading)
            .error(R.mipmap.error);

		Glide.with(context)
      		 .load(IMAGE_URL)
       		 .apply(requestOptions)
       		 .into(imageView);
```

<a name="774583fd"></a>
### 后备回调符 fallback：
**fallback drawable在请求的url/model为null时展示**。设计fallback drawable的主要目的是允许用户指示null是否为可接受的正常情况。例如，一个null的个人资料 url 可能暗示这个用户没有设置头像，因此应该使用默认头像。然而，null也可能表明这个元数据根本就是不合法的，或者取不到。 默认情况下Glide将null作为错误处理，所以可以接受null的应用应当显式地设置一个fallback drawable 。<br />
<br />实现如下：

```java
    RequestOptions requestOptions = new RequestOptions().placeholder(R.mipmap.loading)
            .error(R.mipmap.error)
            .fallback(R.mipmap.fallback);

		Glide.with(context)
      		 .load(IMAGE_URL)
       		 .apply(requestOptions)
       		 .into(imageView);
```

Glide中的大部分设置项都可以通过RequestOptions类和apply()方法来应用到程序中。<br /><br />
<a name="c1bf07ae"></a>
## 4.缓存
<br />默认情况下，Glide 会在开始一个新的图片请求之前检查以下多级的缓存：
1. 活动资源 (Active Resources) - 现在是否有另一个 View 正在展示这张图片？
1. 内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中？
1. 资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存？
1. 数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存？

前两步检查图片是否在内存中，如果是则直接返回图片。后两步则检查图片是否在磁盘上，以便快速但异步地返回图片。<br />如果四个步骤都未能找到图片，则Glide会返回到原始资源以取回数据（原始文件，Uri, Url等）。

<a name="280621f5"></a>
### (1).缓存KEY
这里Kye的生成实现是在Engine.load()中

```java
	 EngineKey key = keyFactory.buildKey(model, signature, width, height, transformations,
        resourceClass, transcodeClass, options);
```

往下流程看到EngineKey的构造函数，可以看到传参有：<br />model（请求加载的URL、File、）、<br />singature（签名）、<br />width（宽度）、<br />height（高度）、<br />transformations(变换)、<br />resourceClass(要转换的资源类型)、<br />transcodeClass(要解码的资源，对应的参数传入是RequestManager.asBitmap())、<br />options（额外添加的选项，用以适应内存）

key至少要包含model和singature这两个元素。在EngineKey类中有2个重写方法equals和hasCode，其实就是用来判断key的每一个元素是否相同，只要有一个不同就不是同一个Key。<br />

```java
  @Override
  public boolean equals(Object o) {
    if (o instanceof EngineKey) {
      EngineKey other = (EngineKey) o;
      return model.equals(other.model)
          && signature.equals(other.signature)
          && height == other.height
          && width == other.width
          && transformations.equals(other.transformations)
          && resourceClass.equals(other.resourceClass)
          && transcodeClass.equals(other.transcodeClass)
          && options.equals(other.options);
    }
    return false;
  }

  @Override
  public int hashCode() {
    if (hashCode == 0) {
      hashCode = model.hashCode();
      hashCode = 31 * hashCode + signature.hashCode();
      hashCode = 31 * hashCode + width;
      hashCode = 31 * hashCode + height;
      hashCode = 31 * hashCode + transformations.hashCode();
      hashCode = 31 * hashCode + resourceClass.hashCode();
      hashCode = 31 * hashCode + transcodeClass.hashCode();
      hashCode = 31 * hashCode + options.hashCode();
    }
    return hashCode;
  }
```

<a name="05e42d09"></a>
### (2).缓存类型
<a name="9c57e2aa"></a>
#### A.内存缓存
Glide使用LruResourceCache，这是MemoryCache接口的一个缺省实现，使用固定大小的内存和 LRU 算法。LruResourceCache的大小由 Glide 的MemorySizeCalculator类来决定，这个类主要关注设备的内存类型，设备 RAM 大小，以及屏幕分辨率。<br />默认情况下Glide是开启内存缓存的，因为BaseRequestOption.isCacheable是为true。若我们不需要内存缓存，则可以调用skipMemoryCache(true)。这样在Engine@load()方法的时候传入的isMemoryCacheable为false。这样在调用loadFromActiveResources()/loadFromCache()的时候会直接返回空，就不会从缓存中加载资源了。<br />

```java
    RequestOptions requestOptions = new RequestOptions().placeholder(R.mipmap.loading)
            .error(R.mipmap.error)
            .fallback(R.mipmap.fallback)
            .skipMemoryCache(true);
```

a.内存缓存读取逻辑<br />如上面所说，在图片加载的时候调用到loadFromActiveResources()/loadFromCache()来实现/跳过内存缓存。

先来看loadFromActiveResources()的源码，这里有看到一个弱引用Hashmap ResourceWeakReference来缓存EngineResource对象。

```java
  @Nullable
  private EngineResource<?> loadFromActiveResources(Key key, boolean isMemoryCacheable) {
    if (!isMemoryCacheable) {
      return null;
    }
    EngineResource<?> active = activeResources.get(key);
    if (active != null) {
      active.acquire();
    }

    return active;
  }
  ....
  final Map<Key, ResourceWeakReference> activeEngineResources = new HashMap<>();
  ....
  @Nullable
  synchronized EngineResource<?> get(Key key) {
    ResourceWeakReference activeRef = activeEngineResources.get(key);
    if (activeRef == null) {
      return null;
    }

    EngineResource<?> active = activeRef.get();
    if (active == null) {
      cleanupActiveReference(activeRef);
    }
    return active;
  }
```


接着看loadFromCache()方法。这里getEngineResourceFromCache()函数获取的cache即是memoryCache，对应的LruResourceCache。获取到图片缓存后，调用activate()生成一个弱引用HashMap：activeEngineResources，并将图片缓存放到activeEngineResources中;而上面的loadFromActiveResources()方法就是从弱引用activeEngineResources中获取图片缓存。这就使得在activeEngineResources中的对象不会被LRU 算法回收。


```java
	private EngineResource<?> loadFromCache(Key key, boolean isMemoryCacheable) {
    if (!isMemoryCacheable) {
      return null;
    }

    EngineResource<?> cached = getEngineResourceFromCache(key);
    if (cached != null) {
      cached.acquire();
      activeResources.activate(key, cached);
    }
    return cached;
	}

	private EngineResource<?> getEngineResourceFromCache(Key key) {
    Resource<?> cached = cache.remove(key);

    final EngineResource<?> result;
    if (cached == null) {
      result = null;
    } else if (cached instanceof EngineResource) {
      // Save an object allocation if we've cached an EngineResource (the typical case).
      result = (EngineResource<?>) cached;
    } else {
      result = new EngineResource<>(
          cached, /*isMemoryCacheable=*/ true, /*isRecyclable=*/ true, key, /*listener=*/ this);
    }
    return result;
	}
```

上面的cache是怎么来的？

![](https://cdn.nlark.com/yuque/__puml/47d613d69066df06aff340d113d8fa36.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aparticipant%20%22Glide%22%20as%20Glide%0Aparticipant%20%22GlideBuilder%22%20as%20GlideBuilder%0A%0AGlide%20-%3E%20Glide%3A%20get%0Aactivate%20Glide%0A%0AGlide%20-%3E%20Glide%3A%20checkAndInitializeGlide%0Aactivate%20Glide%0A%0AGlide%20-%3E%20Glide%3A%20initializeGlide%0Aactivate%20Glide%0A%0AGlide%20-%3E%20GlideBuilder%3A%20build%0Aactivate%20GlideBuilder%0Anote%20right%20of%20GlideBuilder%3AmemoryCache%20%3D%20new%20LruResourceCache%28memorySizeCalculator.getMemoryCacheSize%28%29%29%3B%0A%0A%40enduml)
这就印证了最开始说的，内存缓存是使用LruResourceCache<br />
<br />b.内存缓存写入逻辑<br />之前有讲到过RequestBuilder.into()的逻辑，最后的方法是在 DecodeJob@notifyEncodeAndRelease()。跟一下后续的流程。可以看到会将engineResource对象put到弱引用hashMap：activeEngineResources中。这样再下次调用的时候就会先从activeEngineResources中engineResource获取对象

![](https://cdn.nlark.com/yuque/__puml/c99291d0ded446a2c85adfa0688725c1.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aparticipant%20%22EngineJob%22%20as%20EngineJob%0Aparticipant%20%22DecodeJob%22%20as%20DecodeJob%0Aparticipant%20%22EngineJobListener%22%20as%20EngineJobListener%20%23orange%0Aparticipant%20%22Engine%22%20as%20Engine%0A%0A%0AEngineJob%20-%3E%20EngineJob%3A%20notifyEncodeAndRelease%0Aactivate%20EngineJob%0A%0AEngineJob%20-%3E%20EngineJob%3A%20notifyComplete%0A%0AEngineJob%20-%3E%20EngineJob%3A%20onResourceReady%0A%0AEngineJob%20-%3E%20DecodeJob%3A%20onResourceReady%0Anote%20right%20of%20DecodeJob%3A%20%E5%9B%9E%E8%B0%83%E6%8E%A5%E5%8F%A3%E6%96%B9%E6%B3%95%E5%AE%9E%E7%8E%B0%0A%0ADecodeJob%20-%3E%20DecodeJob%3AnotifyCallbacksOfResult%0A%0ADecodeJob%20-%3E%20EngineJob%3Abuild%0Anote%20right%20of%20EngineJob%3A%E5%AE%9E%E4%BE%8B%E5%8C%96%E4%B8%80%E4%B8%AAengineResource%E5%AF%B9%E8%B1%A1%0A%0ADecodeJob%20-%3E%20EngineJobListener%3AonEngineJobComplete%0A%0AEngineJobListener%20-%3E%20Engine%3A%20onEngineJobComplete%0Anote%20right%20of%20Engine%3A%E8%BF%99%E6%97%B6%E5%B0%B1%E4%BC%9A%E5%B0%86%E8%BF%99%E4%B8%AAengineResource%E5%AF%B9%E8%B1%A1%E6%94%BE%E5%88%B0activeEngineResources%E4%B8%AD%0A%0A%40enduml)

c.关键类EngineResource

acquire()：当调用此方法后会将计数器acquired自增1；<br />ResourceListener：监听器接口；<br />relese()：释放资源，同时将计数器acquired自减1；

因此当acquired大于0的时候，说明资源正在使用；若acquired为0的时候会触发监听器接口函数onResourceReleased()会先将缓存图片从缓存中移除，然后将这个对象put到。这样正在使用中的图片放到弱引用对象中；不使用的会先缓存起来。

```java
public class Engine implements EngineJobListener,
    MemoryCache.ResourceRemovedListener,
    EngineResource.ResourceListener {
  ....
	@Override
	public synchronized void onResourceReleased(Key cacheKey, EngineResource<?> resource) {
    activeResources.deactivate(cacheKey);
    if (resource.isMemoryCacheable()) {
      cache.put(cacheKey, resource);
    } else {
      resourceRecycler.recycle(resource);
    }
  }
  ...
}
```

<a name="d56599e3"></a>
#### B.磁盘缓存
Glide 使用DiskLruCacheWrapper作为默认的磁盘缓存。DiskLruCacheWrapper是一个使用 LRU 算法的固定大小的磁盘缓存。默认磁盘大小为250M，位置是在应用的缓存文件夹中的一个特定目录。路径为：/data/data/{包名}/cache/image_manager_disk_cache 。<br /><br />
```java
public interface DiskCache {

  /**
   * An interface for lazily creating a disk cache.
   */
  interface Factory {
    /** 250 MB of cache. */
    int DEFAULT_DISK_CACHE_SIZE = 250 * 1024 * 1024;
    String DEFAULT_DISK_CACHE_DIR = "image_manager_disk_cache";

    /** Returns a new disk cache, or {@code null} if no disk cache could be created. */
    @Nullable
    DiskCache build();
  }
  ....
}

public final class InternalCacheDiskCacheFactory extends DiskLruCacheFactory {
  ....
  public InternalCacheDiskCacheFactory(final Context context, final String diskCacheName,
                                       long diskCacheSize) {
    super(new CacheDirectoryGetter() {
      @Override
      public File getCacheDirectory() {
        File cacheDirectory = context.getCacheDir();
        if (cacheDirectory == null) {
          return null;
        }
        if (diskCacheName != null) {
          return new File(cacheDirectory, diskCacheName);
        }
        return cacheDirectory;
      }
    }, diskCacheSize);
  }
  ....
}  

```

这些在Glide初始化的时候已经创建完成的。<br /><br /><br />a.缓存策略

```java
	private void runWrapped() {
    switch (runReason) {
      case INITIALIZE:
        stage = getNextStage(Stage.INITIALIZE);
        currentGenerator = getNextGenerator();
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized run reason: " + runReason);
    }
	}

	.......

	private void runGenerators() {
    currentThread = Thread.currentThread();
    startFetchTime = LogTime.getLogTime();
    boolean isStarted = false;
    while (!isCancelled && currentGenerator != null
        && !(isStarted = currentGenerator.startNext())) {
      stage = getNextStage(stage);
      currentGenerator = getNextGenerator();

      if (stage == Stage.SOURCE) {
        reschedule();
        return;
      }
    }

	.......

	private Stage getNextStage(Stage current) {
    switch (current) {
      case INITIALIZE:
        return diskCacheStrategy.decodeCachedResource()
            ? Stage.RESOURCE_CACHE : getNextStage(Stage.RESOURCE_CACHE);
      case RESOURCE_CACHE:
        return diskCacheStrategy.decodeCachedData()
            ? Stage.DATA_CACHE : getNextStage(Stage.DATA_CACHE);
      case DATA_CACHE:
        // Skip loading from source if the user opted to only retrieve the resource from cache.
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
      case SOURCE:
      case FINISHED:
        return Stage.FINISHED;
      default:
        throw new IllegalArgumentException("Unrecognized stage: " + current);
    }
	}

```

这里都会调用到DiskCacheStrategy的decodeCachedResource()/decodeCachedData(),这两个方法是抽象方法，具体实现是：

```
public abstract class DiskCacheStrategy {
 public static final DiskCacheStrategy AUTOMATIC = new DiskCacheStrategy() {
    @Override
    public boolean isDataCacheable(DataSource dataSource) {
      return dataSource == DataSource.REMOTE;
    }

    @Override
    public boolean isResourceCacheable(boolean isFromAlternateCacheKey, DataSource dataSource,
        EncodeStrategy encodeStrategy) {
      return ((isFromAlternateCacheKey && dataSource == DataSource.DATA_DISK_CACHE)
          || dataSource == DataSource.LOCAL)
          && encodeStrategy == EncodeStrategy.TRANSFORMED;
    }

    @Override
    public boolean decodeCachedResource() {
      return true;
    }

    @Override
    public boolean decodeCachedData() {
      return true;
    }
	};
  ....
}
```

这里一共有五种缓存策略：

ALL、结合DATA和RESORCE的策略

NONE：不缓存<br />

DATA：解码前将资源写入磁盘，缓存原始图片

RESOURCE：解码后将资源写入磁盘，缓存转换后的图片

AUTOMATIC：这是默认缓存策略。它会尝试对本地和远程图片使用最佳的策略。当你加载远程数据（比如，从URL下载）时，AUTOMATIC 策略仅会存储未被你的加载过程修改过(比如，变换，裁剪)的原始数据，因为下载远程数据相比调整磁盘上已经存在的数据要昂贵得多。对于本地数据，AUTOMATIC 策略则会仅存储变换过的缩略图，因为即使你需要再次生成另一个尺寸或类型的图片，取回原始数据也很容易。(这段来自官网描述)

因此实现磁盘缓存的方式就是加载不同的缓存策略：

```java
        requestOptions = new RequestOptions().placeholder(R.mipmap.ic_launcher)
        .diskCacheStrategy(DiskCacheStrategy.RESOURCE);
```


b.磁盘缓存写入逻辑

还记得在into()流程中，实现网络请求的方法吗？没错，是在HttpUrlFetcher@loadData()中：

```
  @Override
  public void loadData(@NonNull Priority priority,
      @NonNull DataCallback<? super InputStream> callback) {
    long startTime = LogTime.getLogTime();
    try {
      InputStream result = loadDataWithRedirects(glideUrl.toURL(), 0, null, glideUrl.getHeaders());
      callback.onDataReady(result);
    } catch (IOException e) {
      if (Log.isLoggable(TAG, Log.DEBUG)) {
        Log.d(TAG, "Failed to load data for url", e);
      }
      callback.onLoadFailed(e);
    } finally {
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        Log.v(TAG, "Finished http url fetcher fetch in " + LogTime.getElapsedMillis(startTime));
      }
    }
  }
```

上面可以看到会通过调用loadDataWithRedirects来获取输入流。之后会通过回调方法onDataReady传入这个输入流。我们需要看下onDataReady做了什么！

```
class SourceGenerator implements DataFetcherGenerator,
    DataFetcher.DataCallback<Object>,
    DataFetcherGenerator.FetcherReadyCallback {
	@Override
  public void onDataReady(Object data) {
    DiskCacheStrategy diskCacheStrategy = helper.getDiskCacheStrategy();
    if (data != null && diskCacheStrategy.isDataCacheable(loadData.fetcher.getDataSource())) {
      dataToCache = data;
      // We might be being called back on someone else's thread. Before doing anything, we should
      // reschedule to get back onto Glide's thread.
      cb.reschedule();
    } else {
      cb.onDataFetcherReady(loadData.sourceKey, data, loadData.fetcher,
          loadData.fetcher.getDataSource(), originalKey);
    }
  }
  ....
}
```

上面第6行是否又看到会获取DiskCacheStrategy对象。而在第七行的会调用isDataCacheable()作为判断条件。

```
public abstract class DiskCacheStrategy {
		public static final DiskCacheStrategy AUTOMATIC = new DiskCacheStrategy() {
    @Override
    public boolean isDataCacheable(DataSource dataSource) {
      return dataSource == DataSource.REMOTE;
    }
    ....
}
```

这里我们以默认的AUTOMATIC来看，他实现的isDataCacheable()是通过传参和DataSource.REMOTE来做对比。这时我们需要看下DataSource中各属性的含义。

```
public enum DataSource {
  /**
   * Indicates data was probably retrieved locally from the device, although it may have been
   * obtained through a content provider that may have obtained the data from a remote source.
   */
  LOCAL,
  /**
   * Indicates data was retrieved from a remote source other than the device.
   */
  REMOTE,
  /**
   * Indicates data was retrieved unmodified from the on device cache.
   */
  DATA_DISK_CACHE,
  /**
   * Indicates data was retrieved from modified content in the on device cache.
   */
  RESOURCE_DISK_CACHE,
  /**
   * Indicates data was retrieved from the in memory cache.
   */
  MEMORY_CACHE,
}
```

可以看到DataSource就是分类了数据来源类型。RMOTE是代表来源于外部，那这里isDataCacheable()的返回值就是true。回溯到上面的onDataReady(）方法中的if判断条件成立，那么就将这个资源对象赋值给dataToCache对象。之后按照之前的流程看到又会走到SourceGenerator@startNext()中：

```
class SourceGenerator implements DataFetcherGenerator,
    DataFetcher.DataCallback<Object>,
    DataFetcherGenerator.FetcherReadyCallback {
  @Override
  public boolean startNext() {
    if (dataToCache != null) {
      Object data = dataToCache;
      dataToCache = null;
      cacheData(data);
    }
    ....
   }
   ....
}
```

这里看到了吗？会先去判断dataToCache中是否不为空。当然这里是有对象的，所以会调用cacheData()：

```java
  private void cacheData(Object dataToCache) {
    long startTime = LogTime.getLogTime();
    try {
      Encoder<Object> encoder = helper.getSourceEncoder(dataToCache);
      DataCacheWriter<Object> writer =
          new DataCacheWriter<>(encoder, dataToCache, helper.getOptions());
      originalKey = new DataCacheKey(loadData.sourceKey, helper.getSignature());
      helper.getDiskCache().put(originalKey, writer);
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        Log.v(TAG, "Finished encoding source to cache"
            + ", key: " + originalKey
            + ", data: " + dataToCache
            + ", encoder: " + encoder
            + ", duration: " + LogTime.getElapsedMillis(startTime));
      }
    } finally {
      loadData.fetcher.cleanup();
    }

    sourceCacheGenerator =
        new DataCacheGenerator(Collections.singletonList(loadData.sourceKey), helper, this);
  }
```

这里是不是很明显看到了：会获取当前的DiskCache，然后将DataCacherWriter和Key 以键值对方式put到DiskCache中。

c.磁盘缓存读取逻辑<br />在DecodeJob启动子线程操作的时候我们记得还是有一个Stage枚举吗？

```java
class DecodeJob<R> implements DataFetcherGenerator.FetcherReadyCallback,
    Runnable,
    Comparable<DecodeJob<?>>,
    Poolable {
	private void runWrapped() {
    switch (runReason) {
      case INITIALIZE:
        stage = getNextStage(Stage.INITIALIZE);
        currentGenerator = getNextGenerator();
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized run reason: " + runReason);
    }
  }
  ....
  private Stage getNextStage(Stage current) {
    switch (current) {
      case INITIALIZE:
        return diskCacheStrategy.decodeCachedResource()
            ? Stage.RESOURCE_CACHE : getNextStage(Stage.RESOURCE_CACHE);
      case RESOURCE_CACHE:
        return diskCacheStrategy.decodeCachedData()
            ? Stage.DATA_CACHE : getNextStage(Stage.DATA_CACHE);
      case DATA_CACHE:
        // Skip loading from source if the user opted to only retrieve the resource from cache.
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
      case SOURCE:
      case FINISHED:
        return Stage.FINISHED;
      default:
        throw new IllegalArgumentException("Unrecognized stage: " + current);
    }
  }
  ....
  private DataFetcherGenerator getNextGenerator() {
    switch (stage) {
      case RESOURCE_CACHE:
        return new ResourceCacheGenerator(decodeHelper, this);
      case DATA_CACHE:
        return new DataCacheGenerator(decodeHelper, this);
      case SOURCE:
        return new SourceGenerator(decodeHelper, this);
      case FINISHED:
        return null;
      default:
        throw new IllegalStateException("Unrecognized stage: " + stage);
    }
  }
  ....
  private void runGenerators() {
    currentThread = Thread.currentThread();
    startFetchTime = LogTime.getLogTime();
    boolean isStarted = false;
    while (!isCancelled && currentGenerator != null
        && !(isStarted = currentGenerator.startNext())) {
      stage = getNextStage(stage);
      currentGenerator = getNextGenerator();

      if (stage == Stage.SOURCE) {
        reschedule();
        return;
      }
    }
    // We've run out of stages and generators, give up.
    if ((stage == Stage.FINISHED || isCancelled) && !isStarted) {
      notifyFailed();
    }

    // Otherwise a generator started a new load and we expect to be called back in
    // onDataFetcherReady.
  }
}
```

首次加载图片：<br />RunReason为INITIALIZE，首先调用getNextStage()来获取stage，对应的默认缓存策略是AUTOMATIC，所以返回的是RESOURCE_CACHE；再调用getNextGenerator()来获取currentGenerator，返回ResourceCacheGenerator。这时调用SourceCacheGenerator.startNext() ，由于首次加载磁盘缓存为空，所以返回false，进入while循环；<br />继续RunReason为RESOURCE_CACHE，首先调用getNextStage()来获取stage，对应的默认缓存策略是AUTOMATIC，所以返回的是DATA_CACHE；再调用getNextGenerator()来获取currentGenerator，返回DataCacheGenerator。这时调用DataCacheGenerator.startNext() ，由于首次加载内存缓存为空，所以返回false，继续while循环；<br />继续RunReason为DATA_CACHE，首先调用getNextStage()来获取stage，对应的默认缓存策略是AUTOMATIC，所以返回的是SOURCE；再调用getNextGenerator()来获取currentGenerator，返回SourceGenerator。这时调用SourceGenerator.startNext()，该方法会去通过http请求获取到资源，并且在缓存策略下会将资源对象缓存到缓存中。同时Runreason赋值为SWITCH_TO_SOURCE_SERVICE，且跳出while循环，并且再次reschedule()会再次调动触发DecodeJob.run()；<br />这时调用到runWrapped()方法，Runreason为SWITCH_TO_SOURCE_SERVICE，会直接调用runGenerators()，此时的currentGenerator为SourceGenerator，调用其startNext()。这里在上一小节中讲述磁盘缓存写入逻辑中有说明，这时的缓存中是有对象的，所以该方法返回true，就不会进入到while循环中了。

大致流程如下：<br />

![](https://cdn.nlark.com/yuque/__puml/00e5ff7d482d7f44f46d9c6b90bc7080.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aparticipant%20%22DecodeJob%22%20as%20DecodeJob%0Aparticipant%20%22ResourceCacheGenerator%22%20as%20ResourceCacheGenerator%0Aparticipant%20%22DataCacheGenerator%22%20as%20DataCacheGenerator%0Aparticipant%20%22SourceGenerator%22%20as%20SourceGenerator%0A%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20getNextStage%0Aactivate%20DecodeJob%0Anote%20right%20of%20DecodeJob%3AINITIALIZE%E5%B9%B6%E4%B8%94%E7%BC%93%E5%AD%98%E7%AD%96%E7%95%A5%E9%BB%98%E8%AE%A4%E4%B8%BAAUTOMATIC%EF%BC%8C%E8%BF%94%E5%9B%9ERESOURCE_CACHE%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20getNextGenerator%0Aactivate%20DecodeJob%0Anote%20right%20of%20DecodeJob%3A%E6%96%B0%E5%BB%BAResourceCacheGenerator%0A%0ADecodeJob%20-%3E%20DecodeJob%3ArunGenerators%0Aactivate%20DecodeJob%0A%0ADecodeJob%20-%3E%20ResourceCacheGenerator%3AstartNext%0Aactivate%20ResourceCacheGenerator%0Anote%20right%20of%20ResourceCacheGenerator%3A%E9%A6%96%E6%AC%A1%E5%8A%A0%E8%BD%BD%E7%A3%81%E7%9B%98%E7%BC%93%E5%AD%98%E4%B8%BA%E7%A9%BA%EF%BC%8C%E8%BF%94%E5%9B%9Efalse%EF%BC%8C%E8%BF%9B%E5%85%A5while%E5%BE%AA%E7%8E%AF%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20getNextStage%0Aactivate%20DecodeJob%0Anote%20right%20of%20DecodeJob%3A%E4%BC%A0%E5%85%A5RESOURCE_CACHE%EF%BC%8C%E8%BF%94%E5%9B%9EDATA_CACHE%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20getNextGenerator%0Aactivate%20DecodeJob%0Anote%20right%20of%20DecodeJob%3A%E6%96%B0%E5%BB%BADataCacheGenerator%0A%0ADecodeJob%20-%3E%20DataCacheGenerator%3AstartNext%0Aactivate%20DataCacheGenerator%0Anote%20right%20of%20DataCacheGenerator%3A%E9%A6%96%E6%AC%A1%E5%8A%A0%E8%BD%BD%E5%86%85%E5%AD%98%E7%BC%93%E5%AD%98%E4%B8%BA%E7%A9%BA%EF%BC%8C%E8%BF%94%E5%9B%9Efalse%EF%BC%8C%E7%BB%A7%E7%BB%ADwhile%E5%BE%AA%E7%8E%AF%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20getNextStage%0Aactivate%20DecodeJob%0Anote%20right%20of%20DecodeJob%3A%E4%BC%A0%E5%85%A5DATA_CACHE%EF%BC%8C%E8%BF%94%E5%9B%9ESOURCE%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20getNextGenerator%0Aactivate%20DecodeJob%0Anote%20right%20of%20DecodeJob%3A%E6%96%B0%E5%BB%BASourceGenerator%0A%0ADecodeJob%20-%3E%20SourceGenerator%3AstartNext%0Aactivate%20SourceGenerator%0Anote%20right%20of%20SourceGenerator%3ARunreason%E8%B5%8B%E5%80%BC%E4%B8%BASWITCH_TO_SOURCE_SERVICE%EF%BC%8C%E4%B8%94%E8%B7%B3%E5%87%BAwhile%E5%BE%AA%E7%8E%AF%EF%BC%8C%E5%B9%B6%E4%B8%94%E5%86%8D%E6%AC%A1%E8%A7%A6%E5%8F%91run%28%29%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20run%0Aactivate%20DecodeJob%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20runWrapped%0Aactivate%20DecodeJob%0Anote%20right%20of%20DecodeJob%3ARunreason%E8%B5%8B%E5%80%BC%E4%B8%BASWITCH_TO_SOURCE_SERVICE%0A%0ADecodeJob%20-%3E%20DecodeJob%3A%20runGenerators%0Aactivate%20DecodeJob%0A%0ADecodeJob%20-%3E%20SourceGenerator%3AstartNext%0Aactivate%20SourceGenerator%0Anote%20right%20of%20SourceGenerator%3A%E7%BC%93%E5%AD%98%E4%B8%AD%E6%9C%89%E5%AF%B9%E8%B1%A1%E4%BA%86%EF%BC%8C%E6%89%80%E4%BB%A5%E4%B8%8D%E4%BC%9A%E5%86%8D%E8%BF%9B%E5%85%A5%E5%88%B0while%E5%BE%AA%E7%8E%AF%E4%B8%AD%E3%80%82%0A%0A%40enduml)
非首次加载图片：<br />这个时候缓存中有对象，所以在最开始的SourceCacheGenerator/DataCacheGenerator的startNext()会返回true，且从其中读取到缓存中的对象来加载显示了。

<a name="efa6cf62"></a>
### (3).资源管理
Glide 的磁盘和内存缓存都是 LRU ，这意味着在达到使用限制或持续接近限制值之前，它们将占用持续增加的内存或磁盘空间。为了增加额外的灵活性，Glide 提供了一些额外的方式来让你可以管理你的应用使用的资源。(这部分来自于官方文档中描述)

1. 内存缓存

默认情况下，Glide使用 LruResourceCache ，这是 MemoryCache 接口的一个缺省实现，使用固定大小的内存和 LRU 算法。LruResourceCache 的大小由 Glide 的 MemorySizeCalculator 类来决定，这个类主要关注设备的内存类型，设备 RAM 大小，以及屏幕分辨率。<br />
<br />在AppGlideModule 中使用 applyOptions(Context, GlideBuilder) 方法配置 MemorySizeCalculator

```java
	@GlideModule
	public class YourAppGlideModule extends AppGlideModule {
	@Override
	public void applyOptions(Context context, GlideBuilder builder) {
    MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context)
        .setMemoryCacheScreens(2)
        .build();
    builder.setMemoryCache(new LruResourceCache(calculator.getMemoryCacheSize()));
	}
	}
```
 <br />也可以直接覆写缓存大小：

```java
	@GlideModule
	public class YourAppGlideModule extends AppGlideModule {
	  @Override
	  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setMemoryCache(new YourAppMemoryCacheImpl());
	}
	}

```

若只是临时的改变内存大小，可以调用setMemoryCategory

1. 磁盘缓存

从上面的讲述中可知磁盘缓存大小为250M，且会存放在一个指定的目录中。因此我们可以去自定义这个size和路径：

```java
	@GlideModule
	public class YourAppGlideModule extends AppGlideModule {
	@Override
	public void applyOptions(Context context, GlideBuilder builder) {
    int diskCacheSizeBytes = 1024  1024  100;  100 MB
    builder.setDiskCache(new InternalCacheDiskCacheFactory(context, diskCacheSizeBytes));
	}
	}
```

若是要清理磁盘，可以调用clearDiskCache()即可：

```java
	new AsyncTask<Void, Void, Void> {
	@Override
	protected Void doInBackground(Void... params) {
    // This method must be called on a background thread.
    Glide.get(applicationContext).clearDiskCache();
    return null;
	}
	}
```


<a name="1e7d5435"></a>
## 5.变换
在Glide中，Transformations 可以获取资源并修改它，然后返回被修改后的资源。通常变换操作是用来完成剪裁或对位图应用过滤器，但它也可以用于转换GIF动画，甚至自定义的资源类型。<br /><br />
<a name="da3b3558"></a>
### (1).实现流程
还记得在into()方法的时候，会先有一段根据ImageView的getScaleType()方法来判断图片宽高并给requestOptions赋值吗？<br />

```java
	@NonNull
	public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {
    Util.assertMainThread();
    Preconditions.checkNotNull(view);

    BaseRequestOptions<?> requestOptions = this;
    if (!requestOptions.isTransformationSet()
        && requestOptions.isTransformationAllowed()
        && view.getScaleType() != null) {
      // Clone in this method so that if we use this RequestBuilder to load into a View and then
      // into a different target, we don't retain the transformation applied based on the previous
      // View's scale type.
      switch (view.getScaleType()) {
        case CENTER_CROP:
          requestOptions = requestOptions.clone().optionalCenterCrop();
          break;
        case CENTER_INSIDE:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case FIT_CENTER:
        case FIT_START:
        case FIT_END:
          requestOptions = requestOptions.clone().optionalFitCenter();
          break;
        case FIT_XY:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case CENTER:
        case MATRIX:
        default:
          // Do nothing.
      }
    }

    return into(
        glideContext.buildImageViewTarget(view, transcodeClass),
        /*targetListener=*/ null,
        requestOptions,
        Executors.mainThreadExecutor());
	}

```

我们以ImageView的默认ScaleType：FIT_CENTER 为例。此时会走到第23行。而23行代码实现流程如下：

![](https://cdn.nlark.com/yuque/__puml/11e844a0e3fcd18867a1bd617c706e9d.svg#card=puml&code=%40startuml%0A%0Aautonumber%0A%0Aparticipant%20%22BaseRequestOptions%22%20as%20BaseRequestOptions%0A%0A%0ABaseRequestOptions%20-%3E%20BaseRequestOptions%3A%20optionalFitCenter%0Aactivate%20BaseRequestOptions%0Anote%20right%20BaseRequestOptions%20%3A%E8%BF%99%E9%87%8C%E8%A6%81%E6%B3%A8%E6%84%8F%E4%BC%A0%E5%85%A5%E4%B8%80%E4%B8%AADownsampleStrategy%E5%AF%B9%E8%B1%A1%EF%BC%8C%E5%B9%B6%E4%B8%94%E6%96%B0%E5%BB%BA%E4%B8%80%E4%B8%AAFitCenter%E7%B1%BB%0A%0ABaseRequestOptions%20-%3E%20BaseRequestOptions%3A%20optionalScaleOnlyTransform%0Aactivate%20BaseRequestOptions%0A%0ABaseRequestOptions%20-%3E%20BaseRequestOptions%3A%20scaleOnlyTransform%0Aactivate%20BaseRequestOptions%0Anote%20right%20BaseRequestOptions%20%3AisTransformationRequired%E4%B8%BAfalse%0A%0ABaseRequestOptions%20-%3E%20BaseRequestOptions%3A%20optionalTransform%0Aactivate%20BaseRequestOptions%0A%0ABaseRequestOptions%20-%3E%20BaseRequestOptions%3A%20transform%0Aactivate%20BaseRequestOptions%0Anote%20right%20BaseRequestOptions%20%3A%E6%9C%80%E7%BB%88%E4%BC%9A%E5%B0%86DownsampleStrategy%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%85%A5%E5%88%B0DownsampleStrategy.OPTION%E4%B8%AD%EF%BC%9B%E5%86%8D%E5%B0%86FitCenter%EF%BC%88%E5%8F%98%E6%8D%A2%E5%AF%B9%E8%B1%A1%EF%BC%89%E3%80%81resourceClass%20%E5%AD%98%E5%85%A5%E5%88%B0MAP%E4%B8%AD%0A%40enduml)
接着在解码资源的时候DecodePath.decode()中会通过回调调用DecodeJob.onResourceDecoded()

```java
	public Resource<Transcode> decode(DataRewinder<DataType> rewinder, int width, int height,
      @NonNull Options options, DecodeCallback<ResourceType> callback) throws GlideException {
    Resource<ResourceType> decoded = decodeResource(rewinder, width, height, options);
    Resource<ResourceType> transformed = callback.onResourceDecoded(decoded);
    return transcoder.transcode(transformed, options);
	}
	....

	<Z> Resource<Z> onResourceDecoded(DataSource dataSource,
      @NonNull Resource<Z> decoded) {
    @SuppressWarnings("unchecked")
    Class<Z> resourceSubClass = (Class<Z>) decoded.get().getClass();
    Transformation<Z> appliedTransformation = null;
    Resource<Z> transformed = decoded;
    if (dataSource != DataSource.RESOURCE_DISK_CACHE) {
      appliedTransformation = decodeHelper.getTransformation(resourceSubClass);
      transformed = appliedTransformation.transform(glideContext, decoded, width, height);
    }
    // TODO: Make this the responsibility of the Transformation.
    if (!decoded.equals(transformed)) {
      decoded.recycle();
    }

    final EncodeStrategy encodeStrategy;
    final ResourceEncoder<Z> encoder;
    if (decodeHelper.isResourceEncoderAvailable(transformed)) {
      encoder = decodeHelper.getResultEncoder(transformed);
      encodeStrategy = encoder.getEncodeStrategy(options);
    } else {
      encoder = null;
      encodeStrategy = EncodeStrategy.NONE;
    }

    Resource<Z> result = transformed;
```

可以看到第15-18行的实现：当dataSource不是来自于缓存中的话，就会先去获取appliedTransformation对象（它是根据resourceSubClass来判断的，这个在into()方法最开始的时候就会去设置了：BaseRequestOptions.optionalTransform()！！)，若我们传入的是Bitmap，则就是调用的BitmapTransformation；接着调用重写的transform()方法。<br />

```java
	public final Resource<Bitmap> transform(
      @NonNull Context context, @NonNull Resource<Bitmap> resource, int outWidth, int outHeight) {
    if (!Util.isValidDimensions(outWidth, outHeight)) {
      throw new IllegalArgumentException(
          "Cannot apply transformation on width: " + outWidth + " or height: " + outHeight
              + " less than or equal to zero and not Target.SIZE_ORIGINAL");
    }
    BitmapPool bitmapPool = Glide.get(context).getBitmapPool();
    Bitmap toTransform = resource.get();
    int targetWidth = outWidth == Target.SIZE_ORIGINAL ? toTransform.getWidth() : outWidth;
    int targetHeight = outHeight == Target.SIZE_ORIGINAL ? toTransform.getHeight() : outHeight;
    Bitmap transformed = transform(bitmapPool, toTransform, targetWidth, targetHeight);

    final Resource<Bitmap> result;
    if (toTransform.equals(transformed)) {
      result = resource;
    } else {
      result = BitmapResource.obtain(transformed, bitmapPool);
    }
    return result;
	}
```

<br />可以看到这里会去获取期望的width和height，并作为参数传入到transform()方法中。这个方法是一个接口函数。来看他的实现类。这里我们以CenterCrop为例：

```java
	@Override
	protected Bitmap transform(
      @NonNull BitmapPool pool, @NonNull Bitmap toTransform, int outWidth, int outHeight) {
    return TransformationUtils.centerCrop(pool, toTransform, outWidth, outHeight);
	}
```

<br />TransformationUtils类里就是对于图片变化操作的具体实现方法，具体实现我们在后面看。

<a name="73b0c630"></a>
### (2).Transformation API
其实还是对transform()进行了封装，最终实现都是在TransformationUtils类中。

1. CenterCrop

```java
	public static Bitmap centerCrop(@NonNull BitmapPool pool, @NonNull Bitmap inBitmap, int width,
      int height) {
    if (inBitmap.getWidth() == width && inBitmap.getHeight() == height) {
      return inBitmap;
    }
    // From ImageView/Bitmap.createScaledBitmap.
    final float scale;
    final float dx;
    final float dy;
    Matrix m = new Matrix();
    if (inBitmap.getWidth() * height > width * inBitmap.getHeight()) {
      scale = (float) height / (float) inBitmap.getHeight();
      dx = (width - inBitmap.getWidth() * scale) * 0.5f;
      dy = 0;
    } else {
      scale = (float) width / (float) inBitmap.getWidth();
      dx = 0;
      dy = (height - inBitmap.getHeight() * scale) * 0.5f;
    }

    m.setScale(scale, scale);
    m.postTranslate((int) (dx + 0.5f), (int) (dy + 0.5f));

    Bitmap result = pool.get(width, height, getNonNullConfig(inBitmap));
    // We don't add or remove alpha, so keep the alpha setting of the Bitmap we were given.
    TransformationUtils.setAlpha(inBitmap, result);

    applyMatrix(inBitmap, result, m);
    return result;
	}
```

这里的width/height是如果获取默认的话，则是-1。当然我们如果想要自定义的话，只需要调用override()来实现即可。

```java
	public abstract class BaseRequestOptions<T extends BaseRequestOptions<T>> implements Cloneable {
	private static final int UNSET = -1;

	private int overrideWidth = UNSET;

	public final int getOverrideWidth() {
    return overrideWidth;
	}

	}
```

因此上面的if语句来看就是如果图片高度大于宽度，就会一是计算出缩放比例scale，另一个就是缩短x坐标距离；若图片宽度大于高度，则一是计算是缩放比例scale，另一个就是缩短y坐标距离。<br />这是再对Matrix对象进行缩放和平移操作。（这里可以深入看下实现）因此上面的if语句来看就是如果图片高度大于宽度，就会一是计算出缩放比例scale，另一个就是缩短x坐标距离；若图片宽度大于高度，则一是计算是缩放比例scale，另一个就是缩短y坐标距离。这时再对Matrix对象进行缩放和平移操作。（这里可以深入看下实现）



1. circleCrop

圆形裁剪，即最后生成的图片是圆形的。<br />

1. MultiTransformation

多重变换。即可以按序执行多个变换。注意的是：向 MultiTransformation 的构造器传入变换参数的顺序，决定了这些变换的应用顺序.

```java

	Glide.with(this)
	.load(url)
	.transform(new MultiTransformation(new FitCenter(), new CircleCrop())
	.into(imageView);
```


<a name="190c6036"></a>
### (3).自定义Transformation
尽管 Glide 提供了各种各样的内置 Transformation 实现，如果你需要额外的功能，你也可以实现你自己的 Transformation。<br />如果你只需要变换 Bitmap，最好是从继承 BitmapTransformation 开始。BitmapTransformation 为我们处理了一些基础的东西，例如，如果你的变换返回了一个新修改的 Bitmap ，BitmapTransformation将负责提取和回收原始的 Bitmap。<br />类似如下：

```
	public class FillSpace extends BitmapTransformation {
    private static final String ID = "com.bumptech.glide.transformations.FillSpace";
    private static final String ID_BYTES = ID.getBytes(STRING_CHARSET_NAME);

    @Override
    public Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        if (toTransform.getWidth() == outWidth && toTransform.getHeight() == outHeight) {
            return toTransform;
        }

        return Bitmap.createScaledBitmap(toTransform, outWidth, outHeight, /*filter=*/ true);
    }

    @Override
    public void equals(Object o) {
      return o instanceof FillSpace;
    }

    @Override
    public int hashCode() {
      return ID.hashCode();
    }

    @Override
    public void updateDiskCacheKey(MessageDigest messageDigest)
        throws UnsupportedEncodingException {
      messageDigest.update(ID_BYTES);
    }
	}
```

注意：<br />对于任何 Transformation 子类，包括 BitmapTransformation，你都有三个方法你 必须 实现它们，以使得磁盘和内存缓存正确地工作：<br />equals()<br />hashCode()<br />updateDiskCacheKey<br />如果你的 Transformation 没有参数，通常使用一个包含完整包限定名的 static final String 来作为一个 ID，它可以构成 hashCode() 的基础，并可用于更新 updateDiskCacheKey() 传入的 MessageDigest。如果你的 Transformation 需要参数而且它会影响到 Bitmap 被变换的方式，它们也必须被包含到这三个方法中。


<a name="9e048bb1"></a>
## 补充
<a name="e48c033f"></a>
### 1.Android Studio抛错：Error: Program type already present: android.support.v4.os.ResultReceiver

解决方法：<br />在gradle.properties中添加：

```
android.useAndroidX=true
android.enableJetifier=true
```


<a name="2941b7bc"></a>
### 2.Glide加载不出图片
碰到这个问题定位方式如下可以添加一个方法 

```java
.listener(mRequestListener)
```

接着重写RequestListener，可以调用GlideException.logRootCauses(Stirng) 将异常信息打印出来
```java
    RequestListener mRequestListener = new RequestListener() {
        @Override
        public boolean onLoadFailed(@Nullable GlideException e, Object model, Target target, boolean isFirstResource) {
            e.logRootCauses("xw");
            return false;
        }

        @Override
        public boolean onResourceReady(Object resource, Object model, Target target, DataSource dataSource, boolean isFirstResource) {
            return false;
        }
    };

```

抛错信息如下：

```java
2019-03-17 17:46:44.568 8310-8310/rose.android.com.mytestsdk I/xw: Root cause (1 of 1)
    java.io.IOException: Cleartext HTTP traffic to b-ssl.duitang.com not permitted
        at com.android.okhttp.HttpHandler$CleartextURLFilter.checkURLPermitted(HttpHandler.java:115)
        at com.android.okhttp.internal.huc.HttpURLConnectionImpl.execute(HttpURLConnectionImpl.java:458)
        at com.android.okhttp.internal.huc.HttpURLConnectionImpl.connect(HttpURLConnectionImpl.java:127)
        at com.bumptech.glide.load.data.HttpUrlFetcher.loadDataWithRedirects(HttpUrlFetcher.java:104)
        at com.bumptech.glide.load.data.HttpUrlFetcher.loadData(HttpUrlFetcher.java:59)
        at com.bumptech.glide.load.model.MultiModelLoader$MultiFetcher.loadData(MultiModelLoader.java:100)
        at com.bumptech.glide.load.model.MultiModelLoader$MultiFetcher.startNextOrFail(MultiModelLoader.java:164)
        at com.bumptech.glide.load.model.MultiModelLoader$MultiFetcher.onLoadFailed(MultiModelLoader.java:154)
        at com.bumptech.glide.load.data.HttpUrlFetcher.loadData(HttpUrlFetcher.java:65)
        at com.bumptech.glide.load.model.MultiModelLoader$MultiFetcher.loadData(MultiModelLoader.java:100)
        at com.bumptech.glide.load.engine.SourceGenerator.startNext(SourceGenerator.java:62)
        at com.bumptech.glide.load.engine.DecodeJob.runGenerators(DecodeJob.java:309)
        at com.bumptech.glide.load.engine.DecodeJob.runWrapped(DecodeJob.java:279)
        at com.bumptech.glide.load.engine.DecodeJob.run(DecodeJob.java:235)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1167)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:641)
        at java.lang.Thread.run(Thread.java:764)
        at com.bumptech.glide.load.engine.executor.GlideExecutor$DefaultThreadFactory$1.run(GlideExecutor.java:446)
```

Android P 禁止所有http。所以需要修改url的写法即可。

<a name="bac4403b"></a>
### 3.Glide参考资料

 Glide官方文档：[https://muyangmin.github.io/glide-docs-cn/](https://muyangmin.github.io/glide-docs-cn/)

 Glide github：[https://github.com/bumptech/glide](https://github.com/bumptech/glide)

郭霖大神的博客文章（基于Glide3）：[https://blog.csdn.net/sinyu890807/column/info/15318](https://blog.csdn.net/sinyu890807/column/info/15318)

最后，有不足的地方还希望大佬们多指教指正。感谢。

