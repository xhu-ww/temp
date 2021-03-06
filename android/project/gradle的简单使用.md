# Gradle
## 1. Gradle简介
### 1.1 Gradle 定义
Gradle是一个开源的构建自动化工具，专注于灵活性和性能。
### 1.2 其他
Gradle构建脚本使用Groovy或Kotlin DSL编写。<br>
Gradle支持多种IDE，包括Android Studio，Eclipse，IntelliJ IDEA，Visual Studio和XCode。<br>
Gradle可应用于Android、C++、Groovy、Java、JavaScript、Kotlin、Scala

## 2. Gradle构建
Gradle构建语言参考：https://docs.gradle.org/current/dsl/
### 2.1 下载安装
https://gradle.org/releases/
### 2.2 配置环境变量
~\gradle-4.6\bin
### 2.3 初始化项目
新建项目文件夹，在该文件夹下执行命令：
```
gradle init                 //使用 Groovy
gradle init --dsl kotlin    //使用kotlin
```
命令执行成功后，项目文件夹中会生成如下内容

![image](./gradle_image/gradle01.png)

build.gradle.kts：用于配置当前项目的Gradle构建脚本<br>
gradle-wrapper.jar：Gradle Wrapper可执行JAR<br>
gradle-wrapper.properties：Gradle Wrapper配置属性<br>
gradlew：基于Unix的系统的Gradle Wrapper脚本<br>
gradlew.bat：适用于Windows的Gradle Wrapper脚本<br>
settings.gradle.kts：用于配置Gradle构建的Gradle设置脚本

### 2.4 创建Task
在`build.gradle.kts`文件中编写代码
```
plugins {
    id("base")
}

tasks.create<Zip>("zip"){
    from("app")
    archiveName = "GradleDemo.apk"
}

tasks.create<Copy>("copy"){
    from("build/distributions/GradleDemo.apk")
    into("Release")
}
```
执行命令，然后查看项目文件，可发现已经进行了压缩与复制
```
./gradlew zip
./gradlew copy
```
### 2.5 查看可用Task
执行命令
```
gradlew task
```
相关命令

![image](./gradle_image/gradle02.png)

## 3 Android Gradle 构建
### 3.1 项目gradle文件

通过Android Studio生成的项目，比`gradle init`生成的内容多出了以下文件
```
local.properties  
gradle.properties
build.gradle(Module:app)
```
local.properties：为构建系统配置本地环境属性，例如SDK安装路径。 该文件的内容由 Android Studio 自动生成并且专用于本地开发者环境，不应手动修改该文件，或将其纳入的版本控制系统

gradle.properties：可以在其中配置项目范围 Gradle 设置，例如 Gradle 后台进程的最大堆大小。<br>
参考：https://docs.gradle.org/current/userguide/build_environment.html

### 3.2 自定义Gradle构建

#### 3.2.1 自定义属性
在多模块开发中需要共享通用的属性时，可在Project层级下.gradle文件中自定义属性。
```
buildscript
allprojects 

...

ext {
    compileSdkVersion = 28
    buildToolsVersion = "28.0.0"
    supportVersion = "28.0.0"
    applicationId = "com.xhuww.gradledemo"
    minSdkVersion = 15
    targetSdkVersion = 28
    versionCode = 1
    versionName = "1.0"
}
```
使用属性
```
android {
    compileSdkVersion  rootProject.ext.compileSdkVersion
    defaultConfig {
        applicationId rootProject.ext.applicationId
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode rootProject.ext.versionCode
        versionName rootProject.ext.versionName
    }
}
```
#### 3.2.2 添加依赖

配置插件
```
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
```

依赖类型
```
dependencies { //配置此项目的依赖项                                              。
    //依赖本地 library module
    implementation project(":mylibrary")                        
    //依赖本地jar 包
    implementation fileTree(dir: 'libs', include: ['*.jar'])    
    //远程依赖
    implementation "com.android.support:appcompat-v7:${rootProject.ext.supportVersion}" 
}
```
依赖方式
```
compile         // Gradle Plugin 3.0.0 后已弃用
api             //依赖性运行时和编译时都可用于其他模块。
implementation  // 依赖性仅在运行时可用于其他模块。
...
```
其他参考：https://developer.android.google.cn/studio/build/dependencies.html

远程仓库
```
allprojects {           //配置此项目及其每个子项目。
    repositories {      //配置此项目的存储库。
        google()        
        jcenter()
        mavenCentral()
        mavenLocal()
    }
}
```
#### 3.2.3 签名配置
共享配置
```
android {
    signingConfigs {
        debug {
            keyAlias 'gradle_jks'
            keyPassword '123456'
            storeFile file('../jks/gradle.jks')
            storePassword '123456'
        }
    }
}
```

私密配置

新建`keystore.properties`文件，并定义
```
keyAlias = gradle_jks
keyPassword = 123456
storeFile = file('../jks/gradle.jks')
storePassword = 123456
```
`build.gradle(Module)`中
```
def keystorePropertiesFile = rootProject.file("keystore.properties")
def keystoreProperties = new Properties()
keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

android {
    signingConfigs {
        debug {
            ...
        }

        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
        }
    }
}
```
#### 3.2.4 编译类型配置
```
buildTypes {
    debug {
        minifyEnabled true
        shrinkResources true
        signingConfig signingConfigs.debug
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }

    release {
        minifyEnabled true                      // 启用代码混淆
        shrinkResources true                    // 移除未使用的资源
        signingConfig signingConfigs.release    //配置签名
        // 混淆规则文件
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
}
```
封装此项目的所有构建类型配置。

#### 3.2.5 产品风格配置
```
flavorDimensions "company"  //风格纬度
productFlavors {
    customer {
        dimension = "company"
        applicationIdSuffix ".customer"
    }
    fenrir {
        dimension = "company"
        applicationIdSuffix ".fenrir"
    }
}
```
#### 3.2.6 变体筛选
```
variantFilter { variant ->
    def names = variant.flavors*.name
    if (names.contains("fenrir") && variant.buildType.name == "release") {
        setIgnore(true)
    }
}
```
此时可查看Build Variants

筛选前

![image](./gradle_image/gradle03.png)

筛选后

![image](./gradle_image/gradle04.png)

#### 3.2.7 配置源集
Project -> Module -> src -> 右键 -> New -> Folder -> Java Folder（或其它）<br>
配置完成后

![image](./gradle_image/gradle05.png)

#### 3.2.8 多APK配置
根据手机 屏幕密度 或 应用程序二进制接口（ABI）的文件的不同，打包多个APK
```
splits {
    density {           // 根据屏幕密度配置
        enable true
        exclude "ldpi", "xxhdpi", "xxxhdpi"                         //屏幕密度
        compatibleScreens 'small', 'normal', 'large', 'xlarge'      //屏幕尺寸
    }

    abi {               // 手机架构
        enable true
        reset()
        include "armeabi", "armeabi-v7a", "arm64-v8a", "x86", "x86_64"
        universalApk false
    }
}
```
此时生成的apk

![image](./gradle_image/gradle06.png)

#### 3.2.9 自定义Apk名称
遍历，批量修改输出的 APK 名称

```
applicationVariants.all { variant ->
    variant.outputs.all { output ->
        outputFileName = "${variant.flavorName}_${variant.buildType.name}_${variant.versionName}.apk"
    }
}
```
此时生成的apk

![image](./gradle_image/gradle07.png)

#### 3.2.10 打包命令

./gradlew assemble + Build Variants 名称，例如：

```
./gradlew assembleFenrirDebug
```

## 4. 参考网址
Gradle

https://gradle.org/releases/

https://docs.gradle.org/current/userguide/userguide.html

Gradle For Android

https://developer.android.google.cn/studio/build

http://google.github.io/android-gradle-dsl/current/

https://developer.android.google.cn/studio/releases/gradle-plugin
