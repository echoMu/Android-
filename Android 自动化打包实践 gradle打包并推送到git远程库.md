我们希望在打包的时候能够做到：
1. 使用 Android studio ,使用 gradle 进行构建；
2. 在实际开发中，需要配置我们的 gradle 脚本以支持参数化的方式；
3. 想获得一个可配置打包脚本的方法，允许配置人员根据需要修改服务器地址，versionCode, versionName等；
4. 隔离的源代码的配置，使用者在shell脚本里进行配置。

在阅读本文之前，一些关于gradle的配置项，可以通过这篇文章复习了解
[Android使用gradle打包的各种配置](http://www.jianshu.com/p/1a320062aedd)

#####一.自动打包和推送git的shell脚本
XXX_autoReleaseToGit.sh           
这个脚本会顺序执行打包脚本XXX_assemble.sh和git操作脚本XXX_PUSH_APK.sh，结果是可以根据使用者的选择打包出发布版本、测试版本（极光正式key）和测试版本（极光测试key）。当然，你也可以通过修改脚本，一次性打包出以上版本。

    # 自动打包和推送git的shell脚本
    # 打开自动打包工具目录
    # 执行打包脚本
    cd 你的自动打包工具目录
    echo "进入自动化打包脚本目录..."
    pwd
    #使用gradle命令进行打包
    echo "使用gradle命令开始打包..."
    `dirname ${0}`/XXX_assemble.sh
    #推送到git远程库
    echo "git操作开始..."
    `dirname ${0}`/XXX_PUSH_APK.sh
****执行shell脚本进行自动打包****
XXX_assembleRelease.sh 
这个脚本包含了代码分支更新、代码更新、选择打包环境和第三方服务（极光推送）的操作。

参数都是自定义的，这里写入了多个参数，有指定的各个服务器地址，apk输入文件路径，和环境标识后缀名、极光标识。

    project_path="你的工程根目录"
    gradlew_path="${project_path}/gradlew"
    # 切换到项目目录
    cd ${project_path}
    echo "切换到项目目录..."
    pwd

    # 进行代码分支选择 
    echo "正在更新代码分支信息..."

    # 更新分支信息
    git fetch -p

    # 获取所有远程分支信息
    remote_branchs=`git branch -r`
    # 分割成数组后让使用者选择打包分支
    echo "请选择待打包分支: "
    echo "请选择待打包分支: "
    arr=(${remote_branchs// /})
    index=1
    for i in ${arr[@]}; do
	    #statements
	    echo ${index}". "${i:7}
	    ((index++))
    done

    # 读取使用者数据
    read branch_index
    echo "你选择要打包的分支是： ${branch_index} "

    if [[ -z ${branch_index} ]]; then
	    echo "Error: 选择的分支序号不合法" 
	    echo "Error: 选择的分支序号不合法"
    fi
    ((branch_index--))
    # 切换branch并拉取最新代码
    echo "切换到该分支并拉取最新代码..."
    if [[ ${arr[${branch_index}]} =~ "HEAD" ]]; then
	    echo "Error: 不能切换到该分支(${arr[${branch_index}]})" 
	    echo "Error: 不能切换到该分支(${arr[${branch_index}]})"
	    exit 1
    fi

    # git reset --hard HEAD
    #result_code=$?
    git checkout ${arr[${branch_index}]:7} 
    result_code=$?
    git pull 
    result_code=$?


    if [[ ${result_code} != 0 ]]; then
	    echo "Error: 拉取代码失败" 
	    echo "Error: 拉取代码失败"
	    exit 1
    fi

    # gradlew文件增加可执行权限
    chmod u+x ${gradlew_path}

    echo "请选择版本的环境地址"
    echo "1:正式环境"
    echo "2:测试环境"

    read environment


    # 执行gradlew.bat 进行打包
    # -P表示后面的是自定义参数 如-POUT_PUT_DIR_PARA 表示自定义了一个OUT_PUT_DIR_PARA参数 后面是赋值
    # OUT_PUT_DIR_PARA APK输出目录
    # BASE_URL_PARA 服务器基本请求地址
    # ENVIRONMENT_PARA 服务器环境表示 1.real 正式环境 2.test 测试环境
    # JPUSH_APPKEY_PARA 只要有自定义这个参数 就代表要输出极光推送的APK
    if [[ ${environment} = 1 ]]; then
	    #statements
	    echo "你选择要打包的地址是：正式环境"
	    ${gradlew_path} assembleRelease -POUT_PUT_DIR_PARA=你的APK输出目录 -PBASE_URL_PARA=你的正式环境服务器地址 -PENVIRONMENT_PARA=real --info --stacktrace
    else
	    #statements
	    echo "你选择要打包的地址是：测试环境"
	    echo "----"
	    echo "请选择版本的极光Key"
	    echo "1:正式Key"
	    echo "2:测试Key"

	    read jpushAppKey
	    if [[ ${jpushAppKey} = 1 ]]; then
		    #statements
		    echo "你选择要打包的极光Key是：正式Key"
		    ${gradlew_path} assembleRelease -POUT_PUT_DIR_PARA=你的APK输出目录 -PBASE_URL_PARA=你的测试环境服务器地址 -PENVIRONMENT_PARA=test --info --stacktrace
	    else
		    #statements
		    echo "你选择要打包的极光Key是：测试Key"
		    ${gradlew_path} assembleRelease -POUT_PUT_DIR_PARA=你的APK输出目录 -PBASE_URL_PARA=你的测试环境服务器地址 -PENVIRONMENT_PARA=test_int -PJPUSH_APPKEY_PARA=1 --info --stacktrace
	    fi
    fi
####二.配置gradle文件
**配置 defaultConfig 节点**

    defaultConfig {  
       if (project.hasProperty('JPUSH_APPKEY_PARA')) {            
           //如果有指定极光key的自定义参数，那么就设置极光推送测试key对应的appId                         
            applicationId project.APPLICATIONID_JPUSH        
       } else {            
           //工程本来的appId            
           applicationId project.APPLICATIONID_RELEASE        
       }        
      //最低安装版本Android 4.0        
      minSdkVersion rootProject.ext.minSdkVersion        
      targetSdkVersion rootProject.ext.targetSdkVersion        
      versionCode rootProject.ext.versionCode        
      versionName rootProject.ext.versionName        
      // dex突破65535的限制        
      multiDexEnabled true        
      manifestPlaceholders = [                
            // 默认是umeng测试的渠道                
            UMENG_CHANNEL_VALUE: "TEST",                
            // 默认是正式的极光key 
            JPUSH_APPKEY: project.JPUSH_APPKEY_VALUE_RELEASE        
      ]        
      //配置 defaultConfig 下的  buildConfigField字段 ,这是为了 代码编译的方便，使得在各个环境下都有 BASE_URL 这个字段。        
      //正式服务器        
      buildConfigField("String", "BASE_URL", "\"" + project.BASE_URL_REAL + "\"")
    }

**配置debug节点各个服务器地址的值**
同配置defaultConfig节点一样

    debug {
        //测试服务器请求
        buildConfigField("String", "BASE_URL", "\"" + project.BASE_URL_TEST + "\"")
    }
**配置release节点**
读取上面XXX_assemble.sh文件传入的参数的值作为各个服务器地址的值。
在读取参数的时候，我们先检查参数是否存在，使用代码：

    project.hasProperty('参数名')
所有通过命令行传入的参数都或作为 project 内建对象的属性，我们这里判断了指定的参数名是否存在。如何使用参数呢？直接使用即可。

    versionCode Integer.parseInt(VERSION_CODE_PARA) //注意这里，进行了转型，从字符串转型为 int 类型
    versionName VERSION_NAME_PARA
和普通的变量使用方法是一样的。我们还会遇到在字符串中使用的时候，可以使用表达式 来引用，比如：

    ${参数名}
示例;

    fileName = fileName.replace(".apk", "-${android.defaultConfig.versionName}.apk")

详细如下：
      
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
            if (project.hasProperty('JPUSH_APPKEY_PARA')) {                
                //接收自定义参数的值，指定测试的极光key
                manifestPlaceholders = [
                      JPUSH_APPKEY: project.JPUSH_APPKEY_VALUE_DEBUG
                ]            
             }

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

            productFlavors.all { 
                flavor ->    flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
            }
        }
    }
####三.打包完成之后，将APK提交并推送到git远程库
XXX_PUSH_APK.sh

    cd 你的APK输出目录
    pwd
    #同步远程库
    git pull;
    #add新增加的APK文件
    git add *;
    #提交APK
    git commit -m '提交APK';
    #推送到远程库
    git push;

将以上工作做完之后，我们就可以通过执行脚本来打包了，我们可以打出一系列debug和release的不同服务器环境的版本，对应不同的第三方服务的版本（如极光推送生产环境和发布环境的版本）。当然，你可以修改脚本，写入定时执行功能，将脚本完全写成自动化定时执行的脚本。