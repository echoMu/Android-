####一.关于混淆
ProGuard是一个混淆代码的开源项目，它的主要作用是混淆代码，还包括以下4个功能：
1.压缩(Shrink)：检测并移除代码中无用的类、字段、方法和特性（Attribute）；
2.优化(Optimize)：对字节码进行优化，移除无用的指令；
3.混淆(Obfuscate)：使用a，b，c，d这样简短而无意义的名称，对类、字段和方法进行重命名；
4.预检(Preveirfy)：在Java平台上对处理后的代码进行预检，确保加载的class文件是可执行的。

但是我们还应该看到以下几点问题的存在：
- 代码混淆和编译成.so的安全性都是相对的，都是增加了破解的难度。
- 有的人破解是需要了解其代码的 代码混淆可能会让别人花费更多时间，但是有的破解是不需要看其源码的，比如在一个apk里面嵌入广告，只需要找到启动广告和放入广告代码和xml注册广告权限就可以了。
- android的4大组件不允许被混淆 这就是一个很大的问题，对于反编译的人来说这就是入口。

####二.在AS中开启混淆
在proguard-rules.pro中配置混淆，将minifyEnabled设置为true

    //混淆编译
    minifyEnabled true
    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

####三.如何编写混淆文件
1.加入基本混淆项
2.针对该工程的混淆项
3.针对第三方库的解决

在proguard-rules.pro中加入以下混淆模板，基本可以涵盖大部分情况

    #下面是常见的proguard.cfg配置项
    #指定代码的压缩级别
    -optimizationpasses 5
    #包名不混合大小写
    -dontusemixedcaseclassnames
    #不去忽略非公共的库类
    -dontskipnonpubliclibraryclasses
    # 指定不去忽略非公共的库的类的成员
    -dontskipnonpubliclibraryclassmembers
    #优化  不优化输入的类文件
    -dontoptimize
    #预校验
    -dontpreverify
    #混淆时是否记录日志
    -verbose
    # 混淆时所采用的算法
    -optimizations !code/simplification/arithmetic,!field/*,!class/merging/*
    #保护注解
    -keepattributes *Annotation*

    #忽略警告
    -ignorewarning

    ##记录生成的日志数据,gradle build时在本项目根目录输出##
    #apk 包内所有 class 的内部结构
    -dump class_files.txt
    #未混淆的类和成员-printseeds seeds.txt
    #列出从 apk 中删除的代码
    -printusage unused.txt
    #混淆前后的映射-printmapping mapping.txt
    ########记录生成的日志数据，gradle build时 在本项目根目录输出-end#####

    #需要保留的东西
    # 保持哪些类不被混淆
    -keep public class * extends android.app.Fragment
    -keep public class * extends android.app.Activity
    -keep public class * extends android.app.Application
    -keep public class * extends android.app.Service
    -keep public class * extends android.content.BroadcastReceiver
    -keep public class * extends android.content.ContentProvider
    -keep public class * extends android.app.backup.BackupAgentHelper
    -keep public class * extends android.preference.Preference
    -keep public class * extends android.support.v4.**
    -keep public class com.android.vending.licensing.ILicensingService

    #如果有引用v4包可以添加下面这行
    -keep public class * extends android.support.v4.app.Fragment

    ##########JS接口类不混淆，否则执行不了
    -dontwarn com.android.JsInterface.**
    -keep class com.android.JsInterface.** {*; }

    #极光推送和百度lbs android sdk一起使用proguard 混淆的问题#http的类被混淆后，导致apk定位失败，保持apache 的http类不被混淆就好了
    -dontwarn org.apache.**
    -keep class org.apache.**{ *; }

    -keep public class * extends android.view.View {        
      public <init>(android.content.Context);        
      public <init>(android.content.Context, android.util.AttributeSet);        
      public <init>(android.content.Context, android.util.AttributeSet, int);
      public void set*(...);    
     }

    #保持 native 方法不被混淆
    -keepclasseswithmembernames class * {        
      native <methods>;    
    }

    #保持自定义控件类不被混淆
    -keepclasseswithmembers class * {        
      public <init>(android.content.Context, android.util.AttributeSet);    
    }

    #保持自定义控件类不被混淆
    -keepclassmembers class * extends android.app.Activity {       
      public void *(android.view.View);    
    }

    #保持 Parcelable 不被混淆
    -keep class * implements android.os.Parcelable {      
      public static final android.os.Parcelable$Creator *;    
    }

    #保持 Serializable 不被混淆
    -keepnames class * implements java.io.Serializable

    #保持 Serializable 不被混淆并且enum 类也不被混淆
    -keepclassmembers class * implements java.io.Serializable {        
        static final long serialVersionUID;        
        private static final java.io.ObjectStreamField[] serialPersistentFields;
        !static !transient <fields>;        
        !private <fields>;        
        !private <methods>;        
        private void writeObject(java.io.ObjectOutputStream);        
        private void readObject(java.io.ObjectInputStream);        
        java.lang.Object writeReplace();        
        java.lang.Object readResolve();    
    }

    #保持枚举 enum 类不被混淆 如果混淆报错，建议直接使用上面的 -keepclassmembers class * implements java.io.Serializable即可
    -keepclassmembers enum * {        
          public static **[] values();        
          public static ** valueOf(java.lang.String);    
    }

    -keepclassmembers class * {        
          public void *ButtonClicked(android.view.View);    
    }

    #不混淆资源类
    -keepclassmembers class **.R$* {        
          public static <fields>;    
    }

    #避免混淆泛型 如果混淆报错建议关掉    
    #–keepattributes Signature

    ######混淆保护自己项目的部分代码以及引用的第三方jar包library########
    #如果引用了v4或者v7包
    -dontwarn android.support.**
    
    #如果用到Gson解析包的，直接添加下面这几行就能成功混淆，不然会报错    
    #gson    
    #-libraryjars libs/gson-2.2.2.jar    
    -keepattributes Signature    
    # Gson specific classes    
    -keep class sun.misc.Unsafe { *; }    
    # Application classes that will be serialized/deserialized over Gson    
    -keep class com.google.gson.examples.android.model.** { *; }  
  
    #客户端代码中的JavaBean(实体类)的类名与其字段名称全部变成了a、b、c、d等等字符串，这与服务端返回的json字符串中的不一致，导致解析失败。所以，解决的办法是：在进行混淆编译进行打包apk的时候，过滤掉存放所有JavaBean（实体类)的包不进行混淆编译      
    -keep class com.android.model.** {*;}

    ####混淆保护自己项目的部分代码以及引用的第三方jar包library-end####