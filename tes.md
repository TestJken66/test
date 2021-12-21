# PAAS 流量审核SDK

## 一、版本变动

* `版本号`: `默认版本号`
* `版本变动`:
    1. 优化代码

## 二、集成步骤

### 2.1、拷贝jar到对应项目中

#### 2.1.1、Eclipse SDK 集成

将需要的`jar`包拷贝到本地工程`libs`子目录下；在`Eclipse`中右键工程根目录，选择 `property`—> `Java Build Path` —> `Libraries`
，然后点击 `Add External JARs...` 选择指向 `jar` 的路径，点击 `OK`，即导入成功。（`ADT17` 及以上不需要手动导入）

#### 2.1.2、AndroidStudio SDK 集成

选择`SDK`功能组件并下载，解压`*.zip` 文件得到相应`jar` 包（例如：`x.x.x.jar`等），在 `Android Studio` 的项目工程 `libs`
目录中拷入相关组件 `jar` 包。 右键 `Android Studio` 的项目工程; 选择 `Open Module Settings` → 在 `Project Structure` 弹出框中
→ 选择 `Dependencies` 选项卡 → 点击左下"＋" → 选择 `jar`包类型 → 引入相应的 `jar` 包。

### 2.2、配置Manifest

#### 2.2.1、权限配置

``` xml
<!-- 网络发送和网络是否正常权限 -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<!-- 读写用户标识权限 -->
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<!-- 读写存储卡权限 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<!-- 获取应用活跃权限 -->
<uses-permission android:name="android.permission.GET_TASKS"/>
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS"/>
<!-- android11 安装列表权限-->
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES" />
<!-- 获取应用位置权限-->
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<!-- 
安装列表获取权限. 来源于2021年7月1日起执行的行业标准YD/T 2408-2021《移动智能终端安全能力测试方法》-->
<uses-permission android:name="com.android.permission.GET_INSTALLED_APPS" />
```

* **权限说明：**

| 权限                                           |                       用途 |
| --------------------------------------------- | -------------------------: |
| android.permission.INTERNET                   |              访问互联网的权限 |
| android.permission.READ_PHONE_STATE           |              访问电话相关信息 |
| android.permission.WRITE_EXTERNAL_STORAGE     |           获取外部存储卡写权限 |
| android.permission.ACCESS_WIFI_STATE          |             获取wifi连接状态 |
| android.permission.ACCESS_NETWORK_STATE       |              访问网络连接情况 |
| android.permission.GET_TASKS                  |      允许程序获取最近运行的应用 |
| android.permission.PACKAGE_USAGE_STATS        |                 数据统计服务 |
| android.permission.QUERY_ALL_PACKAGES         |               安装列表获取权限|
| android.permission.ACCESS_BACKGROUND_LOCATION |           允许后台获取位置信息 |
| android.permission.ACCESS_COARSE_LOCATION     |           允许获取大概位置信息 |
| android.permission.ACCESS_FINE_LOCATION       |           允许获取精准定位信息 |
| com.android.permission.GET_INSTALLED_APPS     |              安装列表获取权限 |

#### 2.2.2、组件声明

``` xml
<receiver
    android:name="com.analysys.track.receiver.AnalysysReceiver"
    android:exported="false">
  <intent-filter android:priority="9999">
    <action android:name="android.intent.action.USER_PRESENT"/>
    <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
    <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED"/>
  </intent-filter>
 <intent-filter android:priority="9999">
    <action android:name="android.app.action.NEXT_ALARM_CLOCK_CHANGED"/>
    <action android:name="android.intent.action.LOCALE_CHANGED"/>
    <action android:name="android.intent.action.TIMEZONE_CHANGED"/>
    <action android:name="android.intent.action.TIME_SET"/>
  </intent-filter>
</receiver>
```

### 2.3、初始化接口

``` java
AnalysysTracker.init(Context context, String appkey,  String channel);
```

* 参数

    * **Context**: android上下文
    * **appkey**: 为添加应用后获取到的AppKey
    * **channel**: 应用的下载渠道ID

* 调用方法

``` java
AnalysysTracker.init(context,"appkey","channel");
```

* 备注

  需要在应用的自定义的Application类的onCreate函数里面调用

### 2.4、混淆保护

如果您启用了混淆，请在你的proguard-rules.pro中加入防止混淆的配置.示例如下：

``` groovy
-keep class com.analysys.track.** {
public *;
}
-dontwarn com.analysys.track.**
```

`android 10`可以获取的有效ID减少，为解决该问题可以集成了[MSA SDK](http://www.msa-alliance.cn)。 如您已经集成，需要添加以下防混淆配置:

``` groovy
-keep class com.bun.miitmdid.core.** {*;}
```

### 2.5、证书配置(高版本兼容)

android P之后版本默认不支持HTTP通讯,为APP网络请求正常使用,建议在`AndroidMainfest.xml`中增加证书配置. 如无证书可用如下两种方式配置:

* 无证书配置方式一: 在 `AndroidMainfest.xml`中增加`usesCleartextTraffic`配置就可以正常使用.示例如下:

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest >
    <application
            android:usesCleartextTraffic="true">
    </application>
</manifest>
```

* 无证书配置方式二: 在android10甚至更高版本，部分设置按照无证书配置方式一配置无效，此时修改使用该方法配置.示例如下：

    - 需要添加配置文件(`network_security_config.xml`)如下：

      ``` xml
      <?xml version="1.0" encoding="utf-8"?>
      <network-security-config>
          <base-config cleartextTrafficPermitted="true"/>
      </network-security-config>
      ```

    - 之后在application中添加配置如下，即可：

      ``` xml
      <application
             ...
              android:networkSecurityConfig="@xml/network_security_config"
              ...
      />
      ```

### 2.6、存储适配(高版本兼容)

android Q(10)官方退出分区存储机制，在R(11)开始强制推行,建议进行适配，提供两种方式适配:

* 方式一: 在 `AndroidMainfest.xml`中增加如下配置就可以正常使用.如下:

    ``` xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest >
        <!-- 
            兼容安卓10: requestLegacyExternalStorage
            兼容安卓11: preserveLegacyExternalStorage
        -->
      <application 
        android:requestLegacyExternalStorage="true" 
        android:preserveLegacyExternalStorage="true" >
      </application>
    </manifest>
    ```


* 方式二: 在android11甚至更高版本，可以使用"**所有文件访问权限**"来访问某个文件夹或者媒体资源.

    - 增加`android.permission.MANAGE_EXTERNAL_STORAGE`权限
    -
  使用[存储访问框架](https://developer.android.com/guide/topics/providers/document-provider),即document-provider，
  或者[Media Store API](https://developer.android.com/training/data-storage/shared/media)访问某个文件夹或者媒体资源

### 2.7、分包支持

如果您使用了谷歌的混淆, 请进行如下设置, 将sdk的代码都生成到主dex。 示例如下:

* `build.gradle`

``` groovy
android {
    buildTypes {
        release {
            multiDexKeepProguard 'multidex-config.pro'
        }
    }
}
```

* `multidex-config.pro`

``` groovy
-keep class com.analysys.track.** { *; }
```