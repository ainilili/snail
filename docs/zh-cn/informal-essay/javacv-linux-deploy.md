## 背景
耗时一小周的小玩具``byte-animation``的Beta版本终于可以发布一个小版本了，找了一个空闲时间，我娴熟的连上我的vps，拉下代码/编译打包/运行/一气呵成~

服务端部署完毕，没有问题，之后继续将我扣脚水平的前端代码部署上去，然后其他事情（域名解析/安全组/路由）配置完毕之后，迫不及待的打开页面准备体验一下（其实我之前一直再windows平台开发测试）。

期间遇到一些小问题，例如linux下使用awt库要在``JAVA-OPS``中增加``-Djava.awt.headless=true``变量，又或者要手动配置中文字体库，这些处理完毕之后，体验了一下静态图和``gif``图的处理，一切正常，在之后，体验了下视频的处理......

于是就有了接下来的故事 orz...
## 预热
这里介绍一下，``byte-animation``是一款小玩具，可以将动态或静态图片/视频转成字符画或字符视频，对于视频块的处理，使用的是[javacv](https://github.com/bytedeco/javacv)`中的``ffmpeg``库进行处理。

地址[byte-animation](http://byte.ikuvn.com)
## 问题
使用``javacv-ffmpeg``在windows平台下处理视频完全没问题，移植到linux平台下导致不可用，异常栈：
```java
Exception in thread "main" java.lang.NoClassDefFoundError: Could not initialize class org.bytedeco.javacpp.avutil
    at java.lang.Class.forName0(Native Method)
    at java.lang.Class.forName(Unknown Source)
    at org.bytedeco.javacpp.Loader.load(Loader.java:386)
    at org.bytedeco.javacpp.Loader.load(Loader.java:354)
    at org.bytedeco.javacpp.avformat$AVFormatContext.<clinit>(avformat.java:2719)
    at org.bytedeco.javacv.FFmpegFrameGrabber.startUnsafe(FFmpegFrameGrabber.java:391)
    at org.bytedeco.javacv.FFmpegFrameGrabber.start(FFmpegFrameGrabber.java:385)
    at com.diyoron.ai.examples.VideoFrameProccessor.main(VideoFrameProccessor.java:38)
```
## 故事展开
问题已经定位到了，原因是对应的类编译时是可用的，使用时找不到对应的定义。

于是我打开[javacv](https://github.com/bytedeco/javacv)所在的github仓库的issue，寻求一些坑友的帮助。

果不其然，这种问题发生的可能性有很多，这里简单列举一下：
 - ``javacv``和``ffmpeg``版本冲突
 - maven打包时没有指定平台，``mvn -Djavacpp.platform=linux-x86_64 --update-snapshots [...]``
 - maven依赖错误
 - 缺少``.so``文件

因为之前在windows环境下运行是完全ok的，所以我将怀疑目标定向了maven编译时出了问题，于是到项目``target``目录下将``jar``包解压，一探究竟：
```powershell
java -xvf xxx.jar
```
发现对应的类时存在的，那么我之前的思路都是错误的，那么就要寻找新的思路，顿时感觉这个问题变得有些棘手。

没多想，继续尝试其他可能性：
 - 我将``javacv``大部分的版本都尝试了一遍，失败
 - mvn打包指定平台，失败
 - 执行jar包时，开启平台依赖``java -Djava.awt.headless=true -Dplatform.dependencies=true -jar xxx``，失败
 - 在windows平台打包上传到linux中执行，失败

经历完这些之后，我已经面临崩溃，再次将希望寄托于issue，再次检索了之前的问题，大部分的解决过程都不是我想要的，忽然间，浏览到一个新的希望 [How can I output the error of ffmpeg to the java log  #1182](https://github.com/bytedeco/javacv/issues/1182)

#### question
```
I want to output the error of javacv about the use of ffmpeg to the java log, so it is easier to find the running problem, what should I do?
```
#### answer
```java
If SLF4J supports your logging framework, we can do something like the following during initialization:

System.setProperty("org.bytedeco.javacpp.logger", "slf4j");
FFmpegLogCallback.set();
```
大概意思时可以通过打开日志开关来看更多的异常信息，对于当时的困境来讲非常值得一试（因为我已经开始怀疑问题的本质并不是表面那么简单）

开启日志，再次尝试，果然发现了疑点：
```java
libstdc++.so.6: cannot open shared object file: No such file or directory
```
这个错误信息提醒我们是缺失``so``文件，再回看之前的``NoClassDefFoundError``异常，顿时发现有些微妙的联系！
```java
Exception in thread "main" java.lang.NoClassDefFoundError: Could not initialize class org.bytedeco.javacpp.avutil
    at java.lang.Class.forName0(Native Method)
    at java.lang.Class.forName(Unknown Source)
```
再次分析下：``javacv``通过依赖了大量的本机库，通过``JNI``手段去调用，``so``文件是C/C++在Linux平台上的动态链接库文件，缺少``so``文件说明本机并没有相应的库，那么很可能是对应类中的``native``方法中有些定义不存在。
```java
    at java.lang.Class.forName0(Native Method)
```
注意``Native Method``这个信息，它是从Native方法中抛出的异常，我们继续深入一下对应的实现，分別以``open-jdk 8u``版本的源码为例，我们来跟踪一下``Class.forName0``的底层具体实现。

首先我们都知道，``class``被加载的过程是：``装载、验证、准备、解析、初始化``，一定是其中一个环节抛出的上述异常。

首先是``jdk-43f4f5b4becc\src\share\native\java\lang\Class.c -> Java_java_lang_Class_forName0``，在这里会做一些准备工作，例如一些参数校验，这时并未真正的交给``jvm``去加载：
```c
Java_java_lang_Class_forName0(JNIEnv *env, jclass this, jstring classname,
                              jboolean initialize, jobject loader, jclass caller)
{
    char *clname;
    jclass cls = 0;
    char buf[128];
    jsize len;
    jsize unicode_len;

    if (classname == NULL) {
        JNU_ThrowNullPointerException(env, 0);
        return 0;
    }

    len = (*env)->GetStringUTFLength(env, classname);
    unicode_len = (*env)->GetStringLength(env, classname);
    if (len >= (jsize)sizeof(buf)) {
        clname = malloc(len + 1);
        if (clname == NULL) {
            JNU_ThrowOutOfMemoryError(env, NULL);
            return NULL;
        }
    } else {
        clname = buf;
    }
    (*env)->GetStringUTFRegion(env, classname, 0, unicode_len, clname);

    if (VerifyFixClassname(clname) == JNI_TRUE) {
        /* slashes present in clname, use name b4 translation for exception */
        (*env)->GetStringUTFRegion(env, classname, 0, unicode_len, clname);
        JNU_ThrowClassNotFoundException(env, clname);
        goto done;
    }

    if (!VerifyClassname(clname, JNI_TRUE)) {  /* expects slashed name */
        JNU_ThrowClassNotFoundException(env, clname);
        goto done;
    }

    cls = JVM_FindClassFromCaller(env, clname, initialize, loader, caller);

 done:
    if (clname != buf) {
        free(clname);
    }
    return cls;
}
```
Next：``hotspot-884a9feb3bb8\src\share\vm\prims\jvm.cpp -> JVM_FindClassFromCaller``，这里已经将需要加载的信息交给了jvm，准备加载：
```c
// Find a class with this name in this loader, using the caller's protection domain.
JVM_ENTRY(jclass, JVM_FindClassFromCaller(JNIEnv* env, const char* name,
                                          jboolean init, jobject loader,
                                          jclass caller))
  JVMWrapper2("JVM_FindClassFromCaller %s throws ClassNotFoundException", name);
  // Java libraries should ensure that name is never null...
  if (name == NULL || (int)strlen(name) > Symbol::max_length()) {
    // It's impossible to create this class;  the name cannot fit
    // into the constant pool.
    THROW_MSG_0(vmSymbols::java_lang_ClassNotFoundException(), name);
  }

  TempNewSymbol h_name = SymbolTable::new_symbol(name, CHECK_NULL);

  oop loader_oop = JNIHandles::resolve(loader);
  oop from_class = JNIHandles::resolve(caller);
  oop protection_domain = NULL;
  // If loader is null, shouldn't call ClassLoader.checkPackageAccess; otherwise get
  // NPE. Put it in another way, the bootstrap class loader has all permission and
  // thus no checkPackageAccess equivalence in the VM class loader.
  // The caller is also passed as NULL by the java code if there is no security
  // manager to avoid the performance cost of getting the calling class.
  if (from_class != NULL && loader_oop != NULL) {
    protection_domain = java_lang_Class::as_Klass(from_class)->protection_domain();
  }

  Handle h_loader(THREAD, loader_oop);
  Handle h_prot(THREAD, protection_domain);
  jclass result = find_class_from_class_loader(env, h_name, init, h_loader,
                                               h_prot, false, THREAD);

  if (TraceClassResolution && result != NULL) {
    trace_class_resolution(java_lang_Class::as_Klass(JNIHandles::resolve_non_null(result)));
  }
  return result;
JVM_END
```
Next：``hotspot-884a9feb3bb8\src\share\vm\prims\jvm.cpp -> find_class_from_class_loader``，已经经过了``装载``、``签名``，开始``解析``：
```c
jclass find_class_from_class_loader(JNIEnv* env, Symbol* name, jboolean init,
                                    Handle loader, Handle protection_domain,
                                    jboolean throwError, TRAPS) {
  // Security Note:
  //   The Java level wrapper will perform the necessary security check allowing
  //   us to pass the NULL as the initiating class loader.  The VM is responsible for
  //   the checkPackageAccess relative to the initiating class loader via the
  //   protection_domain. The protection_domain is passed as NULL by the java code
  //   if there is no security manager in 3-arg Class.forName().
  Klass* klass = SystemDictionary::resolve_or_fail(name, loader, protection_domain, throwError != 0, CHECK_NULL);

  KlassHandle klass_handle(THREAD, klass);
  // Check if we should initialize the class
  if (init && klass_handle->oop_is_instance()) {
    klass_handle->initialize(CHECK_NULL);
  }
  return (jclass) JNIHandles::make_local(env, klass_handle->java_mirror());
}
```
Next：``hotspot-884a9feb3bb8\src\share\vm\classfile\systemDictionary.cpp -> resolve_or_fail``，解析并检测是否可用：
```c
Klass* SystemDictionary::resolve_or_fail(Symbol* class_name, Handle class_loader, Handle protection_domain, bool throw_error, TRAPS) {
  Klass* klass = resolve_or_null(class_name, class_loader, protection_domain, THREAD);//解析，如果解析失败，返回null
  if (HAS_PENDING_EXCEPTION || klass == NULL) {
    KlassHandle k_h(THREAD, klass);
    // can return a null klass
    klass = handle_resolution_exception(class_name, class_loader, protection_domain, throw_error, k_h, THREAD);
  }
  return klass;
}
```
``resolve_or_fail``中将会对传入的``class_name``进行解析，如果解析失败返回null，接下来会判断如果``klass``为null，进入``handle_resolution_exception``方法处理异常：
```c
Klass* SystemDictionary::handle_resolution_exception(Symbol* class_name, Handle class_loader, Handle protection_domain, bool throw_error, KlassHandle klass_h, TRAPS) {
  if (HAS_PENDING_EXCEPTION) {
    // If we have a pending exception we forward it to the caller, unless throw_error is true,
    // in which case we have to check whether the pending exception is a ClassNotFoundException,
    // and if so convert it to a NoClassDefFoundError
    // And chain the original ClassNotFoundException
    if (throw_error && PENDING_EXCEPTION->is_a(SystemDictionary::ClassNotFoundException_klass())) {
      ResourceMark rm(THREAD);
      assert(klass_h() == NULL, "Should not have result with exception pending");
      Handle e(THREAD, PENDING_EXCEPTION);
      CLEAR_PENDING_EXCEPTION;
      THROW_MSG_CAUSE_NULL(vmSymbols::java_lang_NoClassDefFoundError(), class_name->as_C_string(), e);
    } else {
      return NULL;
    }
  }
  // Class not found, throw appropriate error or exception depending on value of throw_error
  if (klass_h() == NULL) {
    ResourceMark rm(THREAD);
    if (throw_error) {
      THROW_MSG_NULL(vmSymbols::java_lang_NoClassDefFoundError(), class_name->as_C_string());
    } else {
      THROW_MSG_NULL(vmSymbols::java_lang_ClassNotFoundException(), class_name->as_C_string());
    }
  }
  return (Klass*)klass_h();
}
```
到此为止，我们已经找到了``NoClassDefFoundError``异常发生的根源（不是太肯定），至于``resolve_or_null``到底如何执行的，之后有空再说，那么目前已经知道class在解析的时候发生了问题，说明``class``为空或者某些依赖库的丢失，结合之前的``so``文件缺失日志，那么之后唯一要做的就是将丢失的``so``文件补上之后再试一下。

于是一顿``yum``操作：
```
yum provides xx
yum update xx
yum install xx -y
```
再次重试服务，日志一路狂刷，没有发生异常，试下功能，也完全ok，看来果然是``so``文件缺失的问题 orz...（心疼美好的周末中的一下午时间
## 总结
这次问题的解决浪费了很多时间，也怪自己前期长时间在无用的问题点上周旋，导致一直在做无用功，也再次感谢``javacv``的开发者和各大搜索引擎上的知识贡献者，同时也要弥补一下自己对jvm工作流程的的认知（前期一直以为是类不存在 xD)，以此篇章，记之，警之！




