---
title: tinker-注解部分
date: 2017-04-08 19:39:13
tags: tinker

---
<Excerpt in index | 首页摘要>
### 前言

tinker-android-anno是用来生成application类的，我们就来看下是如何生成的

<!-- more -->
<The rest of contents | 余下全文>


### DefaultLifeCycle

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
@Inherited
public @interface DefaultLifeCycle {

    String application();

    String loaderClass() default "com.tencent.tinker.loader.TinkerLoader";

    int flags();

    boolean loadVerifyFlag() default false;


}
```

### Processor

```
 private void processDefaultLifeCycle(Set<? extends Element> elements) {
        // DefaultLifeCycle
        for (Element e : elements) {
            DefaultLifeCycle ca = e.getAnnotation(DefaultLifeCycle.class);

            String lifeCycleClassName = ((TypeElement) e).getQualifiedName().toString();
            String lifeCyclePackageName = lifeCycleClassName.substring(0, lifeCycleClassName.lastIndexOf('.'));
            lifeCycleClassName = lifeCycleClassName.substring(lifeCycleClassName.lastIndexOf('.') + 1);

            String applicationClassName = ca.application();
            if (applicationClassName.startsWith(".")) {
                applicationClassName = lifeCyclePackageName + applicationClassName;
            }
            String applicationPackageName = applicationClassName.substring(0, applicationClassName.lastIndexOf('.'));
            applicationClassName = applicationClassName.substring(applicationClassName.lastIndexOf('.') + 1);

            String loaderClassName = ca.loaderClass();
            if (loaderClassName.startsWith(".")) {
                loaderClassName = lifeCyclePackageName + loaderClassName;
            }

            System.out.println("*");

            final InputStream is = AnnotationProcessor.class.getResourceAsStream(APPLICATION_TEMPLATE_PATH);
            final Scanner scanner = new Scanner(is);
            final String template = scanner.useDelimiter("\\A").next();
            final String fileContent = template
                .replaceAll("%PACKAGE%", applicationPackageName)
                .replaceAll("%APPLICATION%", applicationClassName)
                .replaceAll("%APPLICATION_LIFE_CYCLE%", lifeCyclePackageName + "." + lifeCycleClassName)
                .replaceAll("%TINKER_FLAGS%", "" + ca.flags())
                .replaceAll("%TINKER_LOADER_CLASS%", "" + loaderClassName)
                .replaceAll("%TINKER_LOAD_VERIFY_FLAG%", "" + ca.loadVerifyFlag());

            try {
                JavaFileObject fileObject = processingEnv.getFiler().createSourceFile(applicationPackageName + "." + applicationClassName);
                processingEnv.getMessager().printMessage(Diagnostic.Kind.NOTE, "Creating " + fileObject.toUri());
                Writer writer = fileObject.openWriter();
                try {
                    PrintWriter pw = new PrintWriter(writer);
                    pw.print(fileContent);
                    pw.flush();

                } finally {
                    writer.close();
                }
            } catch (IOException x) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, x.toString());
            }
        }
    }
```

* 首先获取包名，applicaitonClassName
* 然后获取resource目录下的TinkerAnnoApplication.tmpl文件，根据注解的信息，替换其中的内容，生成一个新的Application类

```
public class %APPLICATION% extends TinkerApplication {

    public %APPLICATION%() {
        super(%TINKER_FLAGS%, "%APPLICATION_LIFE_CYCLE%", "%TINKER_LOADER_CLASS%", %TINKER_LOAD_VERIFY_FLAG%);
    }

}
```

可以看到，注解生成的Application类继承自TinkerApplication，接下来，我们要看的就是这个啦。

### 生成的Application以及传入的参数

从模板文件中，我们看到继承自TinkerApplication。而从生成的过程来看，传入的参数如下：

* applicationClassName，注解中application部分，在demo中就是tinker.sample.android.app.SampleApplication
* lifeCyclePackageName，对应demo中的包名+SampleApplicationLike
* falg 就是注解中的flag
* loaderClass就是loaderClassName，对应demo中就是默认值，om.tencent.tinker.loader.TinkerLoader
* verify_flag就是 flag

生成的文件中，会调用父类的构造方法，对应为

```
   protected TinkerApplication(int tinkerFlags, String delegateClassName,
                                String loaderClassName, boolean tinkerLoadVerifyFlag) {
        this.tinkerFlags = tinkerFlags;
        this.delegateClassName = delegateClassName;
        this.loaderClassName = loaderClassName;
        this.tinkerLoadVerifyFlag = tinkerLoadVerifyFlag;

    }
```



### 最近访客
<ul class="ds-recent-visitors" data-num-items="46" data-avatar-size="40"></ul>