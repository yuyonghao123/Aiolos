宝宝巴士-Aiolos SDK——接入说明文档V1.5

前言
宝宝巴士-Aiolos是公司内部自研的数据统计平台，主要可用于app的数据采集和分析。
目前采集的数据有：
* 设备id：标识用户唯一性
* 设备主要信息：操作系统、系统版本、产商、设备型号、网络状况、分辨率
* App使用信息：使用时长、启动次数、新增、日活、版本、渠道、APP包名

目前提供的功能有：
* 基础数据展示：启动、新增、活跃
* 事件数据展示：计时事件、计数事件、事件系列转换率
* 开机报告：为app提供数据接口定制服务
* 获取app单次使用时长接口

1、SDK嵌入
1.1.步骤1：添加SDK到工程中
请在工程文件根目录下创建一个名为 libs 的子目录，并将Aiolos SDK 的 JAR 包拷贝到 libs 目录下。

![添加jar到libs](./imgs/添加jar到libs.png)

在dependencies引入JAR包

```
dependencies {
    compile files('libs/babybus_aiolos_v1.5.jar')
}
```

1.2.步骤2：修改AndroidManifest.xml文件
添加权限声明：

```
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
```
1.3.步骤3：修改minSdkVersion为14
本sdk忽略android 14以下的版本。

2、基础数据代码接入
2.1步骤1：在后台注册app，并获取app的唯一id
登录后台：[http://dataadmin.babybus.org/](http://dataadmin.babybus.org/)

注册app：

![注册app](./imgs/注册app.png)

点击保存，系统会自动帮你创建该app的guid：

![生成app唯一id](./imgs/生成app唯一id.png)

2.2.步骤2：自定义Application，并实现ActivityLifecycleCallbacks的方法

如下在各实现的方法中加入代码：

```
public class App extends Application implements Application.ActivityLifecycleCallbacks {

    @Override
    public void onCreate() {
        super.onCreate();
        // 注册生命周期监听者
        registerActivityLifecycleCallbacks(this);
        // 设置调试模式，数据实时上传，发布模式数据是间隔1小时上传，发布时要关闭调试模式
        Aiolos.getInstance().setDebug(true);
        // 开启数据统计
        Aiolos.getInstance().startup(this, "后台生成的guid", "渠道");
    }

    @Override
    public void onActivityCreated(Activity activity, Bundle savedInstanceState) {

    }

    @Override
    public void onActivityStarted(Activity activity) {

        // 生命周期注册
        Aiolos.getInstance().onStart();
    }

    @Override
    public void onActivityResumed(Activity activity) {

        // 生命周期注册
        Aiolos.getInstance().onResume();
    }

    @Override
    public void onActivityPaused(Activity activity) {

        // 生命周期注册
        Aiolos.getInstance().onPause();
    }

    @Override
    public void onActivityStopped(Activity activity) {

        // 生命周期注册
        Aiolos.getInstance().onStop();
    }

    @Override
    public void onActivitySaveInstanceState(Activity activity, Bundle outState) {

    }

    @Override
    public void onActivityDestroyed(Activity activity) {

    }
}
```
2.3.步骤3：在使用System.exit(0);进行应用完全退出时，执行Aiolos.getInstance().onExit();保证本次使用数据在退出前被记录。
2.4.步骤4：在AndroidManifest.xml中修改<application>的android:name

```
    <application
        android:name=".App"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
```
至此，实现以上步骤，就可以采集基础数据。目前基础数据支持有：

* 设备id：标识用户唯一性
* 设备主要信息：操作系统、系统版本、产商、设备型号、网络状况、分辨率
* App使用信息：使用时长、启动次数、新增、日活、版本、渠道、APP包名


3、事件数据代码接入
Aiolos还可以采集事件数据。你可以自定义事件系列来进行数据分析。目前支持的事件数据类型分为：

* 计数事件
* 计时事件

3.1.事件注册
你需要在后台注册该事件，获取事件id然后进行代码接入。

![注册事件](./imgs/注册事件.png)

事件有两种类型，计数、计时。

注册后，系统会自动帮你生成事件的guid，在【应用管理】-【事件列表】中，你可以查看这个应用下的所有事件和各个事件的guid。

![事件列表](./imgs/事件列表.png)

3.1.1计数事件代码接入

计数：

```
Aiolos.getInstance().recordEvent("对应的事件id");
```

3.1.2计时事件代码接入

开始计时：

```
Aiolos.getInstance().startEvent("对应事件id");
```

结束计时：

```
Aiolos.getInstance().endEvent("对应的事件id");
```

当你这样嵌入代码后，可以在后台看到采集的数据：

![事件数据结果](./imgs/事件数据结果.png)

3、事件数据携带参数
Aiolos在采集事件数据的同时，可以携带参数，目前携带层级支持到三级。


3.1调用方法
计数事件提供以下方法用来携带参数：

```
Aiolos.getInstance().recordEvent(eventid, "参数1");

Aiolos.getInstance().recordEvent(eventid, "参数1", "参数2");
```

计时事件提供以下方法携带参数：

开始计时：

```
Aiolos.getInstance().startEvent(eventid, "参数1");

Aiolos.getInstance().startEvent(eventid, "参数1", "参数2");
```

结束计时：

```
Aiolos.getInstance().endEvent("对应的事件id");
```

当你这样嵌入代码后，可以在后台看到采集的数据和参数占比。

![事件数据结果](./imgs/事件参数.png)

4、提供开机报告服务
提供接口定制功能，为app量身定制数据接口，例如：通过某台设备的设备id，获知在这个设备上安装过的app的历史数据等

```
1.获得该设备上使用过产品的历史记录，按使用时长降序排列
测试url：暂无
正式url：http://dataadmin.babybus.org/index.php/Api/Summary/getData
返回数据格式：json
http请求方式：post
请求参数：deviceid
返回结果：appjson集合
    1.app key
    2.app 名称
    3.app 总使用时长
    4.app 总启动次数
    5.app 第一次启动时刻
    6.app 最后一次启动时刻
    7.app 在一天中最经常玩的时间段（以小时为单位，列出1个时间段就好）
    8.app 平均每天启动次数
    
```
5、获取app单次使用时长接口
sdk也提供了可以获取上次app使用时长的监听方法，onGetAppLastUseDurationListener，使用示例代码如下：

```
Aiolos.getInstance().setDebug(debug);
Aiolos.getInstance().startup(this, "aiolos appkey", "channel");
// 获取app上次使用时长
Aiolos.getInstance().onGetAppLastUseDurationListener(new IGetAppLastUseDurationListener() {
    @Override
    public void receive(long duration) {
				// do something...
        }
    }
});
```
注意：使用这个方法时需注册在Application的onCreate中。
