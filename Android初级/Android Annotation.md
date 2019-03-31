# Android Annotation

<a name="d04164f9"></a>
# 1.注解的定义
什么是注解？字面意思注释解释。用来给我们注释的，同时解释给编译器。<br />Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。<br />举一个最最常见的：

```java
	@Override
	protected Bitmap transform(
      @NonNull BitmapPool pool, @NonNull Bitmap toTransform, int outWidth, int outHeight) {
    return TransformationUtils.centerCrop(pool, toTransform, outWidth, outHeight);
	}
```



<a name="3948f988"></a>
# 2.注解的用法
上面这个@Override就是一个注解，意思是重写。我们看下@Override具体的是如何实现：

```java
package java.lang;

import java.lang.annotation.*;

/**
 * Indicates that a method declaration is intended to override a
 * method declaration in a supertype. If a method is annotated with
 * this annotation type compilers are required to generate an error
 * message unless at least one of the following conditions hold:
 *
 * <ul><li>
 * The method does override or implement a method declared in a
 * supertype.
 * </li><li>
 * The method has a signature that is override-equivalent to that of
 * any public method declared in {@linkplain Object}.
 * </li></ul>
 *
 * @author  Peter von der Ah&eacute;
 * @author  Joshua Bloch
 * @jls 9.6.1.4 @Override
 * @since 1.5
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
<a name="d41d8cd9"></a>
## 
<a name="c159bc46"></a>
## 1.标准注解
如上面的@Override，有三种标准注解：
1. @Override：对覆盖超类中的方法进行标记，有这个标记就是需要标记的方法去重写超类中的方法；
1. @Deprecared：对过时/废弃的方法进行注解
1. @SuppressWarning：选择性的取消代码中的警告

<a name="8917717c"></a>
## 2.元注解
元注解是由java提供的基础注解，负责注解其它注解。JAVA提供如下几个：<br /><br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/253153/1553329467621-4d4ec9a8-8592-4704-ab9f-f90509e54096.png#align=left&display=inline&height=242&name=image.png&originHeight=266&originWidth=879&size=49970&status=done&width=799)<br /><br /><br />具体的java类定义在\sdk\sources\android-28\java\lang\annotation 路径下。<br />    
<a name="d57456c8"></a>
### @Documented
表示这个注解应该被Javadoc 记录<br />

<a name="9c59a283"></a>
### @Target
定义了注解所修饰对象的范围。其中的取值是一个ElementType类型。我们可以从源码中看到它的取值范围：

```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    //修饰类、接口、枚举类型
    TYPE,

    /** Field declaration (includes enum constants) */
    //修饰成员变量
    FIELD,

  	//修饰方法
    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    //修饰参数
    PARAMETER,

    /** Constructor declaration */
    //修饰构造方法
    CONSTRUCTOR,

    /** Local variable declaration */
    //修饰局部变量
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    //修饰注解
    ANNOTATION_TYPE,

    /** Package declaration */
    //修饰包
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    //修饰类型参数
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    //修饰使用类型
    TYPE_USE
}

```


<a name="4a3bded1"></a>
### @Retention
是声明注解的保留策略。共有三种保存策略：1，源码级注解Source、2，编译时注解Class、3，运行时注解Runtime：

```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

<a name="fcd247d0"></a>
### @Inherited
表示注解时可以被继承


<a name="f552646b"></a>
## 3.自定义注解

<a name="2770f7ec"></a>
### 1.运行时注解
所谓运行时注解，就是注解中的修饰注解策略为运行时:@Retention(RetentionPolicy.RUNTIME)

1. 自定义一个注解ParentAnnotation，首先要使用@**interface **关键字，与接口是类似但有区别的。这个注解仅仅是定义了一个，但是没有方法。注解的成员变量是可以以没有参数的方法来声明的，而若果需要指定默认值，则使用**default**关键字：：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ParentAnnotation {
      String value() default "";
}
```

1. 上面的ParentAnnotation的注解对象是在方法上的。因此我们在某个方法上加上我们自定义的注解ParentAnnotation:

```java
public class Parent {

    @ParentAnnotation(value = "zhangsan")
    public String getName(){
        return "";
    }

    @ParentAnnotation(value = "China")
    public String getCountry(){
        return "";
    }

}
```

1. 通过反射方法来获取对应的方法/属性等：

```java
        //通过反射方法获取Parent类中公有、私有方法等
         Method[] methods = Parent.class.getDeclaredMethods();
        //
        for (Method m : methods){
            //获取方法中的ParentAnnotation对象
            ParentAnnotation parentAnnotation = m.getAnnotation(ParentAnnotation.class);
            if(parentAnnotation != null) {
                Log.i("xw", "value = " + parentAnnotation.value());
            }
        }
```

打印log如下：<br />

```java
03-25 22:02:39.391 8150-8150/rose.android.com.annotationtest I/xw: value = China
03-25 22:02:39.391 8150-8150/rose.android.com.annotationtest I/xw: value = zhangsan
```

这就能看到这里的注解就是生效的，且是在运行时注解的。但是由于考虑到这里是通过反射来拿到注解的相关信息，而反射本身效率就不高，且过多的反射耗时严重。

<a name="588cdcc1"></a>
### 2.编译时注解(Annotation Processor)

<a name="8358a32b"></a>
#### A.定义注解
新建一个Java Library来存放注解：<br />                                    ![QQ截图20190326013834.png](https://cdn.nlark.com/yuque/0/2019/png/253153/1553536143229-cc1fc3db-e9ca-4041-b020-886759fa5dd4.png#align=left&display=inline&height=240&name=QQ%E6%88%AA%E5%9B%BE20190326013834.png&originHeight=240&originWidth=435&size=8418&status=done&width=435)

同时自定义一个注解，作用是去做findViewById()的操作：
```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.CLASS)
public @interface TestAnnotation {
    int value() default 1;
}

```

<a name="521c1fc8"></a>
#### B.定义编译注解器
我们也去用一个Java Library来存放编译注解器<br />                        ![QQ截图20190326015409.png](https://cdn.nlark.com/yuque/0/2019/png/253153/1553536535604-a4783174-c3ec-46eb-aa30-326349e9c841.png#align=left&display=inline&height=256&name=QQ%E6%88%AA%E5%9B%BE20190326015409.png&originHeight=256&originWidth=445&size=8818&status=done&width=445)

这是我们定义MyProcessor类，它需要继承AbstractProcessor，重写方法。写法如下：

```
public class MyProcessor extends AbstractProcessor {

    //如果 processor 类是使用 SupportedOptions 注释的，则返回一个不可修改的集合
    @Override
    public Set<String> getSupportedOptions() {
        return super.getSupportedOptions();
    }

    //指定这个注解器是注册给哪个的注解
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        return super.getSupportedAnnotationTypes();
    }

    //指定Java版本
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return super.getSupportedSourceVersion();
    }

    //初始化工具类调用
    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
    }

    //注解器实际处理逻辑的地方
    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        return false;
    }
}
```

我们来具体实现MyProcessor类，相关说明注释在代码中：

```java
public class MyProcessor extends AbstractProcessor {
  
    //提供和类有关的操作，比如获取父类等
    Types mTypeUtils;
    //提供和元素有关的操作，比如获取包名等
    Elements mElementUtils;
    //用于文件操作，比如创建生成的代码文件
    Filer mFiler;
    //用于打印信息
    Messager mMessager;

    //如果 processor 类是使用 SupportedOptions 注释的，则返回一个不可修改的集合
    @Override
    public Set<String> getSupportedOptions() {
        return super.getSupportedOptions();
    }

    //指定这个注解器是注册给哪个的注解
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> annotation = new HashSet<String>();
        annotation.add(TestAnnotation.class.getCanonicalName());
        return annotation;
    }

    //指定Java版本
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported(;
    }

    //初始化工具类调用
    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
        mTypeUtils = processingEnv.getTypeUtils();
        mElementUtils = processingEnv.getElementUtils();
        mFiler = processingEnv.getFiler();
        mMessager = processingEnv.getMessager();
    }

    //注解器实际处理逻辑的地方
    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        for (Element element : roundEnvironment.getElementsAnnotatedWith(TestAnnotation.class)) {
            if (element.getKind() == ElementKind.FIELD) {
                //输出类名全称
                mMessager.printMessage(Diagnostic.Kind.NOTE,"getEnclosingElement : "
                        + element.getEnclosingElement().toString());
                //输出元素的简单名称
                mMessager.printMessage(Diagnostic.Kind.NOTE,"getSimpleName : "
                        + element.getSimpleName().toString());
                //输出元素的超类类型
                mMessager.printMessage(Diagnostic.Kind.NOTE,"directSupertypes : "
                        + mTypeUtils.directSupertypes(element.asType()));
                //输出元素的种类
                mMessager.printMessage(Diagnostic.Kind.NOTE,"getKind :"
                        + element.getKind());
                //输出元素定义的类型
                mMessager.printMessage(Diagnostic.Kind.NOTE,"asType :"
                        + element.asType().toString());
            }
        }
        return false;
    }
}
```

而我们这里需要在process()中添加我们的实现，比如扫描、查询处理注解，或者生成java文件。

上面的debug log打印如下：

```
注: getEnclosingElement : rose.android.com.annotationtest.MainActivity
注: getSimpleName : button
注: directSupertypes : android.widget.TextView
注: getKind :FIELD
注: asType :android.widget.Button
```

<a name="ab15416c"></a>
#### C.注册编译注解器
创建一个 java META_INF 文件来告诉系统具有注解处理功能。在注解处理库路径下src/main/resources/META-INF/services ，新建一个test文本，命名为：javax.annotation.processing.Processor。如下：<br />    ![QQ截图20190326023107.png](https://cdn.nlark.com/yuque/0/2019/png/253153/1553538674518-73f8c643-acf1-49ec-b131-896b2613b992.png#align=left&display=inline&height=251&name=QQ%E6%88%AA%E5%9B%BE20190326023107.png&originHeight=251&originWidth=460&size=9521&status=done&width=460)

之后在这个文件中添加你所定义的所有的注解编译器。以上面的实现为例：

```
rose.android.com.myprocessor.MyProcessor
```


<a name="e0cdf305"></a>
#### D.添加注解
在我们主程序中添加注解。<br />首先，需要先配置添加前面建的两个模块为依赖项：

```java
dependencies {
    ...
    compile project(':MyAnnotation')
    compile project(':MyProcessor')
}
```

接着在我们代码中应用注解：

```java
public class MainActivity extends AppCompatActivity {

    @TestAnnotation(value = R.id.button_test)
    Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void Test(View view) {
    }
}
```


这时我们make project后会在Gradle console中看到mMessager打印出的信息：<br />

```
:MyProcessor:classes
:MyProcessor:jar
:app:javaPreCompileDebug
注: getEnclosingElement : rose.android.com.annotationtest.MainActivity
注: getSimpleName : button
注: directSupertypes : android.widget.TextView
注: getKind :FIELD
注: asType :android.widget.Button
:app:compileDebugJavaWithJavac
```

<a name="e0fe62d4"></a>
#### E.总结
a.如果一个已存在的Processor实例未被使用,编译工具会调用注解处理器的无参构造函数实例化出一个Processor对象.<br />b.然后编译工具会调用init函数,并传入一个合适的ProcessingEnvironment.<br />C.接着编译工具会调用getSupportedAnnotationTypes,getSupportedOptions和getSupportedSourceVersion.这些方法只会在每一次运行时被调用一次,但不会在每个注解回合都被调用.<br />D.正常情况下,编译工具会调用注解处理器实例的process函数;每个注解回合并不会产生新的注解实例.

AnnotationProcessor生成额外文件的规则是在依赖库里定义的，只在编译的时候执行，但是库最终不打包到apk中。因此他是不会增加APK大小的。这是他相较于运行时注解的优势所在。

<a name="1a3b5c18"></a>
# 3.注解第三方框架

<a name="1.ButterKnife"></a>
## 1.ButterKnife
Field and method binding for Android views 。这是官网上的简介。可以看到这个框架其实是Android View的字段和方法的绑定。实际上ButterKnife更多是对于View注入，比如减少大量的findViewById、setOnClickListener代码。<br />

<a name="f90ff28a"></a>
### 1.常用方法

<a name="033d395a"></a>
#### A.绑定控件

使用@BindView来替换findViewById的调用，即用来绑定控件的id。写法如下：

```java
public class MainActivity extends AppCompatActivity {

    @BindView(R.id.button)
    Button button;

    @BindView(R.id.imageView)
    ImageView imageView;

    String URI = "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1553183270503&di=7d9b6764e19eeaa7678def5671a81336&imgtype=0&src=http%3A%2F%2Fimg.027cgb.com%2F609794%2Fwallpaper%2F1%2F1%2F1%2F1%2F1%2F981030.jpg";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		    //这个bind方法是必须的，否则会报错
        ButterKnife.bind(this);
    }

    public void btnClick(View view){
        Glide.with(this)
                .load(URI)
                .transform(new MultiTransformation<Bitmap>(new FitCenter(),new CircleCrop()))
                .into(imageView);
    }
}
```

注意：这里的控件定义修饰符不能是private或者是static。否则会报如下错误：

```
C:\Users\peng.zhang\AndroidStudioProjects\AnnotationTest\app\src\main\java\rose\android\com\annotationtest\MainActivity.java:18: 错误: @BindView fields must not be private or static. (rose.android.com.annotationtest.MainActivity.button)
    private Button button;
                   ^
1 个错误
:app:compileDebugJavaWithJavac FAILED
```

<a name="9e50aaa2"></a>
#### B.绑定资源
对应的有@BindBool, @BindColor, @BindDimen, @BindDrawable, @BindInt, @BindString。只需要传入对应的id资源即可。

```java
public class MainActivity extends AppCompatActivity {

    @BindView(R.id.button)
    Button button;

    @BindView(R.id.imageView)
    ImageView imageView;

    @BindString(R.string.app_name) String value;

    String URI = "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1553183270503&di=7d9b6764e19eeaa7678def5671a81336&imgtype=0&src=http%3A%2F%2Fimg.027cgb.com%2F609794%2Fwallpaper%2F1%2F1%2F1%2F1%2F1%2F981030.jpg";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Log.i("xw","value = " + value);

        //这个bind方法是必须的，否则会vlaue为null
        ButterKnife.bind(this);

        Log.i("xw","value 1= " + value);

    }
```

这里要注意需要在bind之后调用注解的对象才行。如上述代码的log打印出来：<br />

```java
	03-27 13:21:53.077 12494 12494 I xw      : value = null
	03-27 13:21:53.081 12494 12494 I xw      : value 1= My Application
```

<a name="dc74d6a7"></a>
#### C.View List
多个重复的view控件，我们可以用@BindViews 来绑定多个控件id,如下：

```java
public class MainActivity extends AppCompatActivity {

    @BindViews({R.id.button1,R.id.button2})
    List<Button> buttons;

    @BindView(R.id.imageView)
    ImageView imageView;

    String URI = "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1553183270503&di=7d9b6764e19eeaa7678def5671a81336&imgtype=0&src=http%3A%2F%2Fimg.027cgb.com%2F609794%2Fwallpaper%2F1%2F1%2F1%2F1%2F1%2F981030.jpg";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
    }
}
```

<a name="e15c5111"></a>
#### D.LISTENER BINDING
监听绑定，比如button的onclick、onLongClickl；checkbox的oncheck；EditText的onTextChanged等，写法如下：

```java
    @OnClick(R.id.button1)
    public void btnClick1(View view){
        Log.i("xw","click button1!");
        Glide.with(this)
                .load(URI)
                .transform(new MultiTransformation<Bitmap>(new FitCenter(),new CircleCrop()))
                .into(imageView);
    }

```

<a name="428e2ada"></a>
#### E.BINDING RESET
绑定重置。由于fragment和activity的声明周期不一样。在onCreateView中绑定片段时，请在onDestroyView中将视图设置为null。 当你调用bind为你做这个时，Butter Knife返回一个Unbinder实例。 在适当的生命周期回调中调用其unbind方法。这里以官方给定的示例：

```java
public class FancyFragment extends Fragment {
	@BindView(R.id.button1) Button button1;
	@BindView(R.id.button2) Button button2;
	private Unbinder unbinder;

	@Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    unbinder = ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
	}

	@Override public void onDestroyView() {
    super.onDestroyView();
    unbinder.unbind();
	}
}
```

<a name="e0534ecc"></a>
#### F.OPTIONAL BINDINGS
 可选绑定。考虑到我们即使加上@Bind并且监听绑定，当对应的view无法找到的时候还是会有异常抛出。这里我们就新增了两个注解：对于对象使用@Nullable；对于方法使用@Optional。

```java

	@Nullable @BindView(R.id.might_not_be_there) TextView mightNotBeThere;

	@Optional @OnClick(R.id.maybe_missing) void onMaybeMissingClicked() {
	// TODO ...
	}
```

<a name="048802b2"></a>
#### G.MULTI-METHOD LISTENERS
多重方法监听。每个注解都有一个绑定的默认回调。 使用callback参数指定备用项。

```java
  @OnItemSelected(R.id.list_view)
	void onItemSelected(int position) {
	// TODO ...
	}

	@OnItemSelected(value = R.id.maybe_missing, callback = NOTHING_SELECTED)
	void onNothingSelected() {
	// TODO ...
	}
```

<a name="f4e37ec7"></a>
#### H.添加依赖

Download butterknife，在app的build.gradle中添加如下：

```java
dependencies {
  implementation 'com.jakewharton:butterknife:10.1.0'
  annotationProcessor 'com.jakewharton:butterknife-compiler:10.1.0'
}
```



<a name="9a4e69a3"></a>
### 2.ButterKnife原理

<a name="bf796871"></a>
#### 1.@BindView

我们这里以最常用的@BindView 注解来阐述。首先我们看@BindView的实现：

```java
/**
 * Bind a field to the view for the specified ID. The view will automatically be cast to the field
 * type.
 * <pre><code>
 * {@literal @}BindView(R.id.title) TextView title;
 * </code></pre>
 */
@Retention(RUNTIME) @Target(FIELD)
public @interface BindView {
  /** View ID to which the field will be bound. */
  @IdRes int value();
}
```

可以看到注解策略的是Runtime，所以说最新的ButterKnife是运行时注解，在网上很多都是说是编译时注解的，都非在最新版本上讨论的。**这里有个疑惑为什么改成了运行时注解？**<br />FIELD表示注解应用于成员变量。<br />
<br />然后我们来看ButterKnife 的注解器ButterKnifeProcessor，这个类是ButterKnife的核心实现，基本所有的注解都是在这里来实现的。

```java
@AutoService(Processor.class)
@SuppressWarnings("NullAway") // TODO fix all these...
public final class ButterKnifeProcessor extends AbstractProcessor {
  //来看process()方法的实现
  @Override public boolean process(Set<? extends TypeElement> elements, RoundEnvironment env) {
    //这里获取一个map对象
    Map<TypeElement, BindingSet> bindingMap = findAndParseTargets(env);

    for (Map.Entry<TypeElement, BindingSet> entry : bindingMap.entrySet()) {
      TypeElement typeElement = entry.getKey();
      //获取BindingSet对象
      BindingSet binding = entry.getValue();
      //调用brewJava方法生成一个JavaFile对象
      JavaFile javaFile = binding.brewJava(sdk, debuggable);
      try {
        //将JavaFile对象输出成JAVA文件
        //可以在build-generared-source-apt目录下找到生成的java文件
        javaFile.writeTo(filer);
      } catch (IOException e) {
        error(typeElement, "Unable to write binding for type %s: %s", typeElement, e.getMessage());
      }
    }

    return false;
  }
  
  //findAndParseTargets()方法
  private Map<TypeElement, BindingSet> findAndParseTargets(RoundEnvironment env) {
  	Map<TypeElement, BindingSet.Builder> builderMap = new LinkedHashMap<>();
  	Set<TypeElement> erasedTargetNames = new LinkedHashSet<>();

    // Process each @BindView element.
    //注解每一个@BindView
    for (Element element : env.getElementsAnnotatedWith(BindView.class)) {
      // we don't SuperficialValidation.validateElement(element)
      // so that an unresolved View type can be generated by later processing rounds
      try {
        parseBindView(element, builderMap, erasedTargetNames);
      } catch (Exception e) {
        logParsingError(element, BindView.class, e);
      }
    }
    ...
   }
  
  //parseBindView()方法
  private void parseBindView(Element element, Map<TypeElement, BindingSet.Builder> builderMap,
      Set<TypeElement> erasedTargetNames) {
    TypeElement enclosingElement = (TypeElement) element.getEnclosingElement();

    // Start by verifying common generated code restrictions.
    //isInaccessibleViaGeneratedCode()的用途：
    //验证字段/方法修饰符是否包含private/static、验证包含类型是否是Class、验证包含类的修饰符是否包含private
    //若包含，则返回true；不包含则返回false
    //isBindingInWrongPackage()的用途：
    //验证这个类的包名是否是以android.和java.开头。
    //若不是，则返回false；若是则返回true
    boolean hasError = isInaccessibleViaGeneratedCode(BindView.class, "fields", element)
        || isBindingInWrongPackage(BindView.class, element);

    // Verify that the target type extends from View.
    //验证目标类型是否从View扩展
    TypeMirror elementType = element.asType();
    if (elementType.getKind() == TypeKind.TYPEVAR) {
      TypeVariable typeVariable = (TypeVariable) elementType;
      elementType = typeVariable.getUpperBound();
    }
    Name qualifiedName = enclosingElement.getQualifiedName();
    Name simpleName = element.getSimpleName();
    if (!isSubtypeOfType(elementType, VIEW_TYPE) && !isInterface(elementType)) {
      if (elementType.getKind() == TypeKind.ERROR) {
        note(element, "@%s field with unresolved type (%s) "
                + "must elsewhere be generated as a View or interface. (%s.%s)",
            BindView.class.getSimpleName(), elementType, qualifiedName, simpleName);
      } else {
        error(element, "@%s fields must extend from View or be an interface. (%s.%s)",
            BindView.class.getSimpleName(), qualifiedName, simpleName);
        hasError = true;
      }
    }

    if (hasError) {
      return;
    }

    // Assemble information on the field.
    int id = element.getAnnotation(BindView.class).value();
    BindingSet.Builder builder = builderMap.get(enclosingElement);
    Id resourceId = elementToId(element, BindView.class, id);
    if (builder != null) {
      String existingBindingName = builder.findExistingBindingName(resourceId);
      if (existingBindingName != null) {
        error(element, "Attempt to use @%s for an already bound ID %d on '%s'. (%s.%s)",
            BindView.class.getSimpleName(), id, existingBindingName,
            enclosingElement.getQualifiedName(), element.getSimpleName());
        return;
      }
    } else {
      builder = getOrCreateBindingBuilder(builderMap, enclosingElement);
    }

    String name = simpleName.toString();
    TypeName type = TypeName.get(elementType);
    boolean required = isFieldRequired(element);

    builder.addField(resourceId, new FieldViewBinding(name, type, required));

    // Add the type-erased version to the valid binding targets set.
    erasedTargetNames.add(enclosingElement);
  }
}
```


因此当我们运行项目之后，可以看到生成一个MainActivity$$ViewBinder.java：<br />
![QQ截图20190329001724.png](https://cdn.nlark.com/yuque/0/2019/png/253153/1553789850679-a7efdda0-9021-4a66-80e7-9a8f83c439cc.png#align=left&display=inline&height=215&name=QQ%E6%88%AA%E5%9B%BE20190329001724.png&originHeight=496&originWidth=1723&size=76188&status=done&width=746)


<a name="2.ButterKnife.bind"></a>
#### 2.ButterKnife.bind
上面有说过若不是不添加butterKnife.bind，则会报错的。先来看bind()方法的实现：<br />

```java
  /**
   * BindView annotated fields and methods in the specified {@link Activity}. The current content
   * view is used as the view root.
   *
   * @param target Target activity for view binding.
   */
  @NonNull @UiThread
  public static Unbinder bind(@NonNull Activity target) {
    View sourceView = target.getWindow().getDecorView();
    return bind(target, sourceView);
  }

```

可以看到bind()方法有很多重载方法，上面传入的参数是Activity，也可以是View、Dialog等。那我们接着看bind(target, sourceView)的实现：

```
  /**
   * BindView annotated fields and methods in the specified {@code target} using the {@code source}
   * {@link View} as the view root.
   *
   * @param target Target class for view binding.
   * @param source View root on which IDs will be looked up.
   */
  @NonNull @UiThread
  public static Unbinder bind(@NonNull Object target, @NonNull View source) {
    Class<?> targetClass = target.getClass();
    if (debug) Log.d(TAG, "Looking up binding for " + targetClass.getName());
    Constructor<? extends Unbinder> constructor = findBindingConstructorForClass(targetClass);

    if (constructor == null) {
      return Unbinder.EMPTY;
    }

    //noinspection TryWithIdenticalCatches Resolves to API 19+ only type.
    try {
      //调用newInstance方法生成一个constructor类型的实例
      return constructor.newInstance(target, source);
    } catch (IllegalAccessException e) {
      throw new RuntimeException("Unable to invoke " + constructor, e);
    } catch (InstantiationException e) {
      throw new RuntimeException("Unable to invoke " + constructor, e);
    } catch (InvocationTargetException e) {
      Throwable cause = e.getCause();
      if (cause instanceof RuntimeException) {
        throw (RuntimeException) cause;
      }
      if (cause instanceof Error) {
        throw (Error) cause;
      }
      throw new RuntimeException("Unable to create binding instance.", cause);
    }
    
  //findBindingConstructorForClass()方法
  @Nullable @CheckResult @UiThread
    private static Constructor<? extends Unbinder> findBindingConstructorForClass(Class<?> cls) {
    Constructor<? extends Unbinder> bindingCtor = BINDINGS.get(cls);
    if (bindingCtor != null || BINDINGS.containsKey(cls)) {
      if (debug) Log.d(TAG, "HIT: Cached in binding map.");
      return bindingCtor;
    }
    String clsName = cls.getName();
    if (clsName.startsWith("android.") || clsName.startsWith("java.")
        || clsName.startsWith("androidx.")) {
      if (debug) Log.d(TAG, "MISS: Reached framework class. Abandoning search.");
      return null;
    }
    //通过反射生成class类，有看到这里是通过classname_ViewBinding来拼接的吗？
    try {
      Class<?> bindingClass = cls.getClassLoader().loadClass(clsName + "_ViewBinding");
      //noinspection unchecked
      bindingCtor = (Constructor<? extends Unbinder>) bindingClass.getConstructor(cls, View.class);
      if (debug) Log.d(TAG, "HIT: Loaded binding class and constructor.");
    } catch (ClassNotFoundException e) {
      if (debug) Log.d(TAG, "Not found. Trying superclass " + cls.getSuperclass().getName());
      bindingCtor = findBindingConstructorForClass(cls.getSuperclass());
    } catch (NoSuchMethodException e) {
      throw new RuntimeException("Unable to find binding constructor for " + clsName, e);
    }
    //将Constructor作为value、class作为key存储在BINDINGS中
    BINDINGS.put(cls, bindingCtor);
    //返回Constructor
    return bindingCtor;
  }
}
```

<br />接着我们看运行之后生成的java文件：

```java
public class MainActivity$$ViewBinder<T extends MainActivity> implements ViewBinder<T> {
  @Override
  public Unbinder bind(Finder finder, T target, Object source) {
    return new InnerUnbinder<>(target, finder, source);
  }

  protected static class InnerUnbinder<T extends MainActivity> implements Unbinder {
    protected T target;

    protected InnerUnbinder(T target, Finder finder, Object source) {
      this.target = target;
      //这里实现
      target.button = finder.findRequiredViewAsType(source, 2131165219, "field 'button'", Button.class);
    }

    @Override
    public void unbind() {
      T target = this.target;
      if (target == null) throw new IllegalStateException("Bindings already cleared.");

      target.button = null;

      this.target = null;
    }
  }
}
```

可以接着看下finder.findRequiredViewAsType，最终是调用到findOptionalViewAsType()中的findOptionalView()，而这个方法是抽象方法，具体的实现类又是根据bind()传入的对象类型有关系，我们上面传入的是this，实际上是Activity。这就会去调用到findViewById()。然后将生成的view强制转换Button类型并返回。返回值会赋值给target.button ，也就是MainActivity。这样就可以在MainActivity中使用这个Button了。<br />所以上面所说用@bindView来替代findViewById还是不准确的，因为findViewById最终实现是通过bind()方法来的

```java
  public abstract View findOptionalView(Object source, int id);

  public final <T> T findOptionalViewAsType(Object source, int id, String who, Class<T> cls) {
    View view = findOptionalView(source, id);
    try {
      return cls.cast(view);
    } catch (ClassCastException e) {
      String name = getResourceEntryName(view, id);
      throw new IllegalStateException("View '"
          + name
          + "' with ID "
          + id
          + " for "
          + who
          + " was of the wrong type. See cause for more info.", e);
    }
  }
  
  ACTIVITY {
    @Override public View findOptionalView(Object source, int id) {
      return ((Activity) source).findViewById(id);
    }

    @Override public Context getContext(Object source) {
      return (Activity) source;
    }
  },
```


<a name="5ece4894"></a>
# 4.补充
<a name="5334d2a6"></a>
## 1.异常报错
<a name="34d1d263"></a>
### 1.set android.defaultConfig.javaCompileOptions.annotationProcessorOptions.includeCompileClasspath = true to continue with previous behavior.

解决方法：

```java
android {
		...
    defaultConfig {
				...
        javaCompileOptions{
            annotationProcessorOptions{
                includeCompileClasspath = true
            }
        }
    }
}
```


![0475e45a-2d27-11e7-8d9a-0ed9e0e753d8.png](https://cdn.nlark.com/yuque/0/2019/png/253153/1553531231145-55794e28-6bb6-4fad-88b9-8121639ef5ba.png#align=left&display=inline&height=483&name=0475e45a-2d27-11e7-8d9a-0ed9e0e753d8.png&originHeight=483&originWidth=473&size=39671&status=done&width=473)

<a name="3ff7896d"></a>
### 2.Invoke-customs are only supported starting with android 0 --min-api 26

```java
android {
		...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

<a name="02c9275d"></a>
## 2.小细节
<a name="e141f51d"></a>
### 1.如何打开Gradle console窗口：

![QQ截图20190326005644.png](https://cdn.nlark.com/yuque/0/2019/png/253153/1553533014656-bc8a6ace-d771-4c65-b9be-e190b0632a4c.png#align=left&display=inline&height=342&name=QQ%E6%88%AA%E5%9B%BE20190326005644.png&originHeight=342&originWidth=698&size=36804&status=done&width=698)


<a name="31d2d59a"></a>
## 3.资料
BufferKnife github：[https://github.com/JakeWharton/butterknife](https://github.com/JakeWharton/butterknife)
