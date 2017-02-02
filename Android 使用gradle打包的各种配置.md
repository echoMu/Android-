在AS中利用gradle打包，可以高效并且自由地配置各种参数，发布不同的版本。关于配置gradle文件的一些做法，总结为如下。
####一.替换AndroidManifest中的占位符
举个例子，在AndroidManifest文件中，我们将极光推送的key值指定为一个占位符

    <!-- 极光KEY -->
    <meta-data    
      android:name="JPUSH_APPKEY"   
      android:value="${JPUSH_APPKEY}" />
在build.gradle文件中，这里介绍3种方法去替换该占位符
1.接收gradlew assemble命令输入的自定义参数的值
    
    manifestPlaceholders = [        
                // 默认是正式的极光key        
                JPUSH_APPKEY: "\"" + JPUSH_APPKEY_PARA + "\""
    ]
2.使用string文件的值

    manifestPlaceholders = [JPUSH_APPKEY:"@string/JPUSH_APPKEY"]
3.使用gradle.properties文件的值，具体参考第九
####二.独立配置签名信息
签名相关的信息,直接写在gradle不利于安全，我们可以把这些信息写到工程主module根目录下的signing.properties文件，注意这个文件不要添加进版本控制。

    KEYSTORE_FILE=你的keystore文件位置
    KEYSTORE_PASSWORD= 你的keystore文件密码
    KEY_ALIAS= 你的keystore文件用到的别名
    KEY_PASSWORD= 你的keystore文件用到的别名的密码
然后在build.gradle中加载这个文件，引用其中的参数就可以了

    //加载签名配置的文件
    Properties props = new Properties()props.load(new
    FileInputStream(file("signing.properties")))
    android {
      signingConfigs {    
        release{        
            //设置release的签名信息       
            keyAlias props['KEY_ALIAS']        
            keyPassword props['KEY_PASSWORD']        
            storeFile file(props['KEYSTORE_FILE'])        
            storePassword props['KEYSTORE_PASSWORD']    
        }
      }
    
    ...

      buildTypes {
          debug {
            ...
            signingConfig signingConfigs.release}
          }
          ...    
          release {
            ...
            signingConfig signingConfigs.release}
        }
      }
    }
####三. 多渠道打包
国内应用市场非常多，为了统计各个应用市场的app下载量和使用情况，我们需要多渠道的打包。

1.配置AndroidManifest.xml
以友盟渠道为例，渠道信息一般都是写在 AndroidManifest.xml文件中：
    
    <meta-data android:name="UMENG_CHANNEL" android:value="xiaomi" />
如果不使用多渠道打包方法，那就需要我们手动一个一个去修改value中的值，xiaomi，360，qq，wandoujia等等。使用多渠道打包的方式，就需要把上面的value配置成下面的方式：

    <meta-data android:name="UMENG_CHANNEL" android:value="${UMENG_CHANNEL_VALUE}" />
其中${UMENG_CHANNEL_VALUE}中的值就是你在gradle中自定义配置的值。

2.在build.gradle设置productFlavors
写法如下：

    productFlavors { 
      wandoujia  { 
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"] 
      } 
      xiaomi  { 
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"] 
      }
      qq  { 
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "qq"] 
      } 
      360  { 
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "360"] 
      }
    }

其中[UMENG_CHANNEL_VALUE: "wandoujia"]就是对应
${UMENG_CHANNEL_VALUE}的值。这里还有简洁的写法：

    android {
      ...

      buildTypes {
          debug {
            ...
          }
          ...    
          release {
            ...
            productFlavors { 
                wandoujia{} 
                xiaomi{} 
                qq{} 
                360 {} 
            } 
            productFlavors.all { flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name] 
          }
        }
    }
其中name的值对相对应各个productFlavors的选项值，这样就达到自动替换渠道值的目的了。这样（用AS自带工具Generate signed apk）生成apk时，选择相应的Flavors来生成指定渠道的包就可以了，而且生成的apk会自动帮你加上相应渠道的后缀。

3.一次生成所有渠道包
使用CMD命令，进入到项目所在的目录，直接输入命令：

    gradle assembleRelease
就可以打Release包，或者，在Android Studio中的下方底栏中有个命令行工具Terminal，你也可以直接打开，输入上面的命令，进行打包。

**注意：如果没有对gradle配置的话，可能输入上面的命令，会提示“不是内部或者外部命令”，不要着急，我们只需要找到gradle的目录，把它配置到电脑中的环境变量中去即可。**

配置方式如下：

先找到gralde的根目录，在系统变量里添加两个环境变量：

变量名为：GRADLE_HOME，变量值就为gradle的根目录；

比如变量值为：
D:androidandroid-studio-ide-143.2739321-windowsandroid-studiogradlegradle-2.10

还有一个在系统变量里PATH里面添加gradle的bin目录

比如：
D:androidandroid-studio-ide-143.2739321-windowsandroid-studiogradlegradle-2.10bin

这样就配置完了，执行以下这个命令：gradle assembleRelease，看看是不是可以了。
####4.修改导出包的文件目录和apk名称

    // 定义一个打包时间
    def releaseTime() {    
          return new Date().format("yyyyMMdd", TimeZone.getTimeZone("UTC"))
    }

    android {
      ...

      buildTypes {
          debug {
            ...
          }
          ...    
          release {
            ...
            applicationVariants.all { 
                variant ->    variant.outputs.each { output ->        
                    def outputFile = output.outputFile        
                    if (outputFile != null && outputFile.name.endsWith('.apk')) {           
                        // 输出apk名称为XXX20160328_v1.0.0_vc10_XXXX_test.apk        
                        if (project.hasProperty('ENVIRONMENT_PARA') 
                        def fileName=" XXX${releaseTime()}_v${defaultConfig.versionName}_vc${defaultConfig.versionCode}_${variant.productFlavors[0].name}_${ENVIRONMENT_PARA}.apk"                  
                        //控制输出的APK的存放路径                
                        if (project.hasProperty('OUT_PUT_DIR_PARA')) {                    
                            File output_dir1 = file("${OUT_PUT_DIR_PARA}");                    
                            output.outputFile = new File(output_dir1, fileName)                    
                             println "输出文件位置： " + output.outputFile                
                        } else {                    
                            output.outputFile = new File(outputFile.parent, fileName)                    
                            println "输出文件位置： " + output.outputFile                
                        }            
                    }        
                }    
            }
          }
      }
    }
####四.多工程全局配置
随着产品渠道的铺开，往往一套代码需要支持多个产品形态，这就需要抽象出主要代码到一个Library，然后基于Library扩展几个App Module。相信每个module的build.gradle都会有这个代码：

    android { 
      compileSdkVersion 22 
      buildToolsVersion "23.0.1" 
      defaultConfig { 
        minSdkVersion 10 
        targetSdkVersion 22 
        versionCode 34 
        versionName "v2.6.1" 
      }
     }
当升级sdk、build tool、target sdk等，几个module都要更改，非常的麻烦。也可能导致app module之间的差异不统一，导致不可控。强大的gradle插件在1.1.0之后支持全局变量设定，一举解决了这个问题。先在project的根目录下的build.gradle定义ext全局变量:

    ext { 
      compileSdkVersion = 22 
      buildToolsVersion = "23.0.1" 
      minSdkVersion = 10 
      targetSdkVersion = 22 
      versionCode = 34 
      versionName = "v2.6.1"
    }
然后在各module的build.gradle中引用如下：

    android { 
      compileSdkVersion rootProject.ext.compileSdkVersion 
      buildToolsVersion rootProject.ext.buildToolsVersion 

      defaultConfig { 
        applicationId "com.xxx.xxx" 
        minSdkVersion rootProject.ext.minSdkVersion 
        targetSdkVersion rootProject.ext.targetSdkVersion 
        versionCode rootProject.ext.versionCode 
        versionName rootProject.ext.versionName 
      }
    }
每次修改project级别的build.gradle即可实现全局统一配置。
####五.混淆代码
release版本开启混淆

    //混淆编译
    minifyEnabled true
    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

混淆文件可以参考这篇文章进行配置[Android代码混淆在AS的实践](http://www.jianshu.com/p/e19cc5194a31)
####六.动态设置一些额外信息
把当前的编译时间、编译的机器、最新的commit版本添加到apk

    android { 
      defaultConfig { 
          resValue "string", "build_time", buildTime() 
          resValue "string", "build_host", hostName() 
          resValue "string", "build_revision", revision() 
      }
    }

    def buildTime() { 
          return new Date().format("yyyy-MM-dd HH:mm:ss")
    }

    def hostName() { 
          return System.getProperty("user.name") + "@" + InetAddress.localHost.hostName
    }

    def revision() { 
          def code = new ByteArrayOutputStream() 
          exec { 
            commandLine 'git', 'rev-parse', '--short', 'HEAD' 
            standardOutput = code 
          } 
          return code.toString()
    }
上述代码实现了动态的添加了3个字符串资源: build_time、build_host、build_revision, 然后在其他地方可像如引用字符串一样使用如下：

    // 在Activity里调用
    getString(R.string.build_time) // 输出2016-09-07 17:01
    getString(R.string.build_host) // 输出电脑的用户名和PC
    getString(R.string.build_revision) // 输出3dd5823, 这是最后一次commit的sha值
####七.buildConfigField自定义配置
大家可能会遇到下面这种情况，就是Beta版本服务器和Release版本服务器通常不在一台服务器上，而测试希望可以同时发布两个服务器的版本用于测试，这个时候我们就需要修改代码，然后一个一个老老实实的发包。gradle提供buildConfigField配合多渠道打不同服务器版本的方法。其实用法很简单,首先在相应的节点加上定义，比如：

    buildTypes { 
      debug { 
        buildConfigField "boolean", "LOG_DEBUG", "true"//是否输出LOG信息   
        buildConfigField "String", "API_HOST", "\"http://api.test.com\""//API Host 
      } 
    }
然后在代码中通过BuildConfig.LOG_DEBUG或者BuildConfig.API_HOST调用即可。
####八.dex突破65535的限制
随着项目的一天天变大，慢慢的都会遇到单个dex最多65535个方法数的瓶颈，如果是ANT构建的项目就会比较麻烦，但是Gradle已经帮我们处理好了，而添加的方法也很简单，总共就分三步 :
1.首先是在defaultConfig节点使能多DEX功能

    android { 
      defaultConfig { 
          // dex突破65535的限制 
          multiDexEnabled true 
      } 
    }
2.然后就是引入multidex库文件
    
    dependencies { 
      compile 'com.android.support:multidex:1.0.0' 
    }
3.最后就是你的AppApplication继承一下MultiDexApplication即可。
####九.在gradle.properties文件中配置服务器生产环境和正式环境的地址、第三方服务appkey以及对于包名的配置 
项目中加入用到一些第三方的SDK的话，就避免不了各种key的写入，一般都会有生产环境和正式环境各自使用的值
**gradle.properties如下：**

    # 极光推送的测试key
    JPUSH_APPKEY_VALUE_DEBUG=111111111111111111
    # 极光推送的正式key
    JPUSH_APPKEY_VALUE_RELEASE=111111111111111111
    # 极光推送的测试key对应的包名
    APPLICATIONID_JPUSH=com.xxx.xxx
    # 极光推送的正式key对应的包名
    APPLICATIONID_RELEASE=com.xxx.xxx
    # 测试环境地址
    BASE_URL_TEST=
    # 正式环境地址
    BASE_URL_REAL=
在build.gradle文件中引用，

    defaultConfig {
      if (project.hasProperty('JPUSH_APPKEY_PARA')) {    
        //如果有指定极光key的自定义参数，那么就设置极光推送测试key对应的appId
        applicationId project.APPLICATIONID_JPUSH
      } else {    
        //工程本来的appId    
        applicationId project.APPLICATIONID_RELEASE
      }

       manifestPlaceholders = [                
          // 默认是正式的极光key
          JPUSH_APPKEY: project.JPUSH_APPKEY_VALUE_RELEASE        
      ]
    }
也可配合buildConfigField自定义配置：

    buildConfigField("String", "BASE_URL", "\"" + project.BASE_URL_TEST + "\"")