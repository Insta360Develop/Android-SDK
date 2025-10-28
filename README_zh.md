# 注意：

当前为 `V1.8.2-build4` 分支版本，若需查看其他版本，请切换对应分支。

32 位库（armeabi-v7a）已不再维护，请使用 64 位库（arm64-v8a）进行构建！

* [常见问题](#常见问题)
# 如何获取？

请访问 [https://www.insta360.com/sdk/apply](https://www.insta360.com/sdk/apply) 申请最新 SDK。

# 技术支持

开发者主页：https://www.insta360.com/developer/home  
企业解决方案：https://www.insta360.com/enterprise  
问题反馈表单：https://insta.jinshuju.com/f/hZ4aMW  

# [English Document](README.md)

# 简要

安卓端SDK主要用于连接、设置和获取相机参数、控制相机进行拍照和录制、文件下载、固件升级和支持视频导出与图片导出等。

支持机型：X5, X4, X3, ONE X2, ONE X, ONE RS, ONE RS一英寸。
# 目录

* [Camera SDK功能](#camera-sdk功能)
    * [环境准备](#环境准备)
    * [连接模块](#连接模块)
    * [Wi-Fi管理模块](#wi-fi管理模块)
    * [信息获取模块](#信息获取模块)
    * [通知模块](#通知模块)
    * [镜头控制模块](#镜头控制模块)
    * [预览模块](#预览模块)
    * [拍摄控制模块](#拍摄控制模块)
    * [直播模块](#直播模块)
    * [文件管理模块](#文件管理模块)
    * [日志记录模块](#日志记录模块)
    * [GPS数据模块](#gps数据模块)
    * [其它功能模块](#其它功能模块)
    * [错误码](#错误码)
* [Media SDK功能](#media-sdk功能)
    * [环境准备](#环境准备-1)
    * [预览模块](#预览模块-1)
    * [WorkWrapper](#workwrapper)
    * [图片播放器](#图片播放器)
    * [视频播放器](#视频播放器)
    * [导出图片](#导出图片)
    * [导出视频](#导出视频)
    * [其它](#其它)
    * [算法文件](#算法文件)
    * [错误码](#错误码-1)

# Camera SDK功能

## 环境准备

- 将maven地址添加到构建文件（项目根目录中的`build.gradle`文件）中

```Groovy
allprojects {
    repositories {
        ...
        maven {
            url 'x.x.x'             // Reference SDK Demo
            credentials {
                // Reference SDK Demo
                username = '***'
                password = '***'
            } 
        }
    }
}
```

- 在Demo目录的`build.gradle`文件中导入依赖库

```Groovy
dependencies {
    implementation 'com.arashivision.sdk:sdkcamera:x.x.x' // Reference SDK Demo
}
```

- 在应用程序中初始化SDK

```java
public class MyApp extends Application {

    @Overridepublic void onCreate() {
        super.onCreate();      
        InstaCameraSDK.init(this);
    }
}
```

## 连接模块

### 判断相机是否连接

通过获取当前相机的连接类型来获知相机是否处于连接状态。

```Java
private boolean isCameraConnected() {
    int type = InstaCameraManager.getInstance().getCameraConnectedType();
    return type != InstaCameraManager.CONNECT_TYPE_NONE;
}
```

连接类型有以下四种结果：

1. 未连接：*`InstaCameraManager.CONNECT_TYPE_NONE`*
2. USB连接：*`InstaCameraManager.CONNECT_TYPE_USB`*
3. Wi-Fi连接：*`InstaCameraManager.CONNECT_TYPE_WIFI`*
4. 蓝牙连接：*`InstaCameraManager.CONNECT_TYPE_BLE`*

### 连接相机

您可以通过蓝牙、Wi-Fi或USB的方式连接相机。

> 注意：您必须在主线程上执行连接操作。

> 注意：相机与App建立连接后，如果App切换至后台，由于Android系统的原因，导致会断开连接。因此需要开启一个前台服务以防止与相机断开连接。这个前台服务器由开发者自行实现。相关代码可参考SDK Demo;

#### 蓝牙连接

> 注意：蓝牙连接需要蓝牙相关的权限授权成功。

> 注意：蓝牙连接的情况下，不支持大数据传输。因此传输数据比较大的功能是不支持的。
>
> 不支持功能：直播、预览、获取支持列表、下载文件、固件升级、播放、导出图像。

1. 发起蓝牙扫描

```Java
InstaCameraManager.getInstance().startBleScan();
```

2. 设置蓝牙扫描监听

```Java
InstaCameraManager.getInstance().setScanBleListener(new IScanBleListener() {
    @Override
    public void onScanStartSuccess() {
        //蓝牙即将开始扫描
    }

    @Override
    public void onScanStartFail() {
        //蓝牙扫描失败
    }

    @Override
    public void onScanning(BleDevice bleDevice) {
        //扫描到一个新的蓝牙设备，可以在此处更新蓝牙设备列表
    }

    @Override
    public void onScanFinish(List<BleDevice> bleDeviceList) {
        //蓝牙扫描结束
    }
});
```

3. 停止蓝牙扫描

```Java
InstaCameraManager.getInstance().stopBleScan();
```

4. 判断是否正在蓝牙扫描中

```Java
InstaCameraManager.getInstance().isBleScanning();
```

5. 与相机建立蓝牙连接

```Java
InstaCameraManager.getInstance().connectBle(bleDevice);
```

#### Wi-Fi连接

> 注意：必须保证手机已经连接上了相机的Wi-Fi。相机的Wi-Fi密码在相机中的设置菜单可以查看，或者通过蓝牙连接，调用读取相机Wi-Fi的名称和密码去自动连接。

```Java
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_WIFI);
```

#### USB连接

> 注意：必须保证手机与相机已经建立了USB连接，并且相机需要切换到安卓模式。

```Java
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_USB);
```

### 断开连接

当你想断开连接时，你可以调用下列代码。

```Java
InstaCameraManager.getInstance().closeCamera();
```

## Wi-Fi管理模块

### 开启Wi-Fi

在蓝牙连接的情况下，可以通过下面的代码控制相机WI-Fi的开启和关闭。

```Java
ICameraOperateCallback callback = new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // 指令执行成功
    }

    @Override
    public void onFailed() {
        // 指令执行失败
    }

    @Override
    public void onCameraConnectError() {
        // 相机发生断连或未连接
    }
};
// 打开相机wifi, 使用相机默认信道
InstaCameraManager.getInstance().openCameraWifi(callback);
// 指定信道打开相机wifi
InstaCameraManager.getInstance().openCameraWifi(int countryChannel ,callback);

// 关闭相机wifi
InstaCameraManager.getInstance().closeCameraWifi(callback)

// 重启相机Wi-Fi,内含关闭和开启2个操作，使用相机默认信道开启
InstaCameraManager.getInstance().resetCameraWifi(callback)
// 指定信道重启相机Wi-Fi
InstaCameraManager.getInstance().resetCameraWifi(int countryChannel callback)
```

### Wi-Fi信息获取

```Java
// 获取相机Wi-Fi信息
WifiInfo wifiInfo = InstaCameraManager.getInstance().getWifiInfo();

// Wi-Fi名称
String ssid = wifiInfo.getSsid();

// Wi-Fi密码
String ssid = wifiInfo.getPwd();

// 密码版本
int pwdVersion= wifiInfo.getPwdVersion();

// Wi-Fi信道
int channel= wifiInfo.getChannel();

// mac地址
String address = wifiInfo.getMacAddress();

// 获取Wi-Fi国家码
Strin country = InstaCameraManager.getInstance().getWifiCountry();
```

### 切换Wi-Fi频段

```Java
// 获取当前国家码和Wi-Fi信道
InstaCameraManager.getInstance().fetchWifiChannel((countryCode, cameraChannel) -> {
    // 获取国家码和当前Wi-Fi信道
});

// 设置Wi-Fi国家码
InstaCameraManager.getInstance().setWifiCountry("CN", new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
         // 设置成功
    }

    @Override
    public void onFailed() {
        // 设置失败
    }

    @Override
    public void onCameraConnectError() {
        // 相机连接错误
    }
});

// 获取当前支持的信道列表
int[] channelList = InstaCameraManager.getInstance().getWifiChannelList();
```

## 信息获取模块

### 同步相机参数

同步相机参数意味着一次性读取相机内部的所有参数，并在应用程序中进行缓存。调用下述代码即可同步所有相机参数（所有参数状态的同步）。 

```Java
InstaCameraManager.getInstance().fetchCameraOptions(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // 相机参数同步成功
    }

    @Override
    public void onFailed() {
        // 相机参数同步失败
    }

    @Override
    public void onCameraConnectError() {
        // 相机连接失败，可能发生相机断连事件
    }
});
```

### 获取相机支持列表

X4及以后相机的拍摄模式和拍摄属性都是以支持列表中所配置的属性为准。不同相机所支持的拍摄模式有所不同，且不同拍摄模式下所支持的拍摄属性亦存在差别，在拍摄前需要同步相机的支持列表（Json文件）。

#### 初始化支持列表

当进入拍摄页或切换镜头时，需对支持列表进行初始化。可通过调用下述代码实现：

```Java
InstaCameraManager.getInstance().initCameraSupportConfig(success -> {
    // success:是否初始化成功
});
```

#### 拍摄模式支持

相机的每个镜头所支持的拍摄模式均存在差异，需通过以下代码获取其支持的拍摄模式。

```Java
// 获取所有支持的拍摄模式
List<CaptureMode> support = InstaCameraManager.getInstance().getSupportCaptureMode();
```

#### 拍摄属性支持

每种拍摄模式所支持的拍摄属性亦存在差异，需通过以下代码获取其支持的拍摄属性。

```Java
// 参数传入指定的拍摄模式
List<CaptureSetting> support = InstaCameraManager.getInstance().getSupportCaptureSettingList(CaptureMode.CAPTURE_NORMAL);
```

#### 拍摄属性的取值范围

拍摄属性的取值范围同样是在支持列表中所配置的，需通过以下代码获取其支持的拍摄属性的取值范围。 

```Java
// 获取CaptureSetting.RECORD_DURATION的属性值取值范围，返回一个List<RecordDuration>列表
InstaCameraManager.getInstance().getSupportRecordDurationList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.RECORD_RESOLUTION的属性值取值范围，返回一个List<RecordResolution>列表
InstaCameraManager.getInstance().getSupportRecordResolutionList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.PHOTO_RESOLUTION的属性值取值范围，返回一个List<PhotoResolution>列表
InstaCameraManager.getInstance().getSupportPhotoResolutionList(CaptureMode.CAPTURE_NORMAL);

// 获取CaptureSetting.INTERVAL的属性值取值范围，返回一个List<Interval>列表
InstaCameraManager.getInstance().getSupportIntervalList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.HDR_STATUS的属性值取值范围，返回一个List<HdrStatus>列表
InstaCameraManager.getInstance().getSupportHdrStatusList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.PHOTO_HDR_TYPE的属性值取值范围，返回一个List<PhotoHdrType>列表
InstaCameraManager.getInstance().getSupportPhotoHdrTypeList(CaptureMode.CAPTURE_NORMAL);

// 获取CaptureSetting.GAMMA_MODE的属性值取值范围，返回一个List<GammaMode>列表
InstaCameraManager.getInstance().getSupportGammaModeList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.RAW_TYPE的属性值取值范围，返回一个List<RawType>列表
InstaCameraManager.getInstance().getSupportRawTypeList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.PANO_EXPOSURE_MODE的属性值取值范围，返回一个List<PanoExposureMode>列表
InstaCameraManager.getInstance().getSupportPanoExposureList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.EXPOSURE的属性值取值范围，返回一个List<Exposure>列表
InstaCameraManager.getInstance().getSupportExposureList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.ISO的属性值取值范围，返回一个List<ISO>列表
InstaCameraManager.getInstance().getSupportISOList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.ISO_TOP_LIMIT的属性值取值范围，返回一个List<ISOTopLimit>列表
InstaCameraManager.getInstance().getSupportISOTopLimitList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.SHUTTER_MODE的属性值取值范围，返回一个List<ShutterMode>列表
InstaCameraManager.getInstance().getSupportShutterModeList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.SHUTTER的属性值取值范围，返回一个List<Shutter>列表
InstaCameraManager.getInstance().getSupportShutterList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.EV的属性值取值范围，返回一个List<Ev>列表
InstaCameraManager.getInstance().getSupportEVList(CaptureMode.CAPTURE_NORMAL);

// 获取CaptureSetting.EV_INTERVAL的属性值取值范围，返回一个List<EVInterval>列表
InstaCameraManager.getInstance().getSupportEVIntervalList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.AEB的属性值取值范围，返回一个List<AEB>列表
InstaCameraManager.getInstance().getSupportAEBList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.WB的属性值取值范围，返回一个List<WB>列表
InstaCameraManager.getInstance().getSupportWBList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.INTERNAL_SPLICING的属性值取值范围，返回一个List<InternalSplicing>列表
InstaCameraManager.getInstance().getSupportInternalSplicingList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.DARK_EIS_ENABLE的属性值取值范围，返回一个List<DarkEisType>列表
InstaCameraManager.getInstance().getSupportDarkEisList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.BURST_CAPTURE的属性值取值范围，返回一个List<BurstCapture>列表
InstaCameraManager.getInstance().getSupportBurstCaptureList(CaptureMode.TIMELAPSE);

// 获取CaptureSetting.I_LOG的属性值取值范围，返回一个List<ILogStatus>列表
InstaCameraManager.getInstance().getSupportILogStatusList(CaptureMode.TIMELAPSE);
```

### 信息获取

#### 相机型号

```Java
// 获取相机类型  如：Insta360 X4
String cameraType = InstaCameraManager.getInstance().getCameraType();
// 获取枚举
CameraType type = CameraType.getForType(cameraType);
```

#### 相机序列号

```Java
InstaCameraManager.getInstance().getCameraSerial();
```

#### 固件版本号

```Java
// 获取相机固件版本  如：0.9.24
String fwVersion = InstaCameraManager.getInstance().getCameraVersion();
```

#### 激活时间

```Java
// 获取相机激活时间，未激活时间为0.
long activeTime = InstaCameraManager.getInstance().getActiveTime();
```

#### SD卡总容量

```Java
// 获取SD卡总容量大小
long totalSpace = InstaCameraManager.getInstance().getCameraStorageTotalSpace();
```

#### SD卡剩余容量

```Java
// 获取SD卡可用空间大小
long freeSpace = InstaCameraManager.getInstance().getCameraStorageFreeSpace();

// 获取SD卡余量（如果是拍照，则单位是张，如果是录像，则返回可持续录制的时间）
int remaining= InstaCameraManager.getInstance().getRemaining(CaptureMode captureMode);
```

#### 电池电量信息

```Java
// 获取电池类型
// THICK = 0;       //厚电池
// THIN = 1;        //薄电池（相机默认值）
// VERTICAL;        //竖拍电池
// NONE;            //未插电池只有USB供电的情况
InstaCameraManager.getInstance().getBatteryType();

// 获取当前电量 : 0-100
int level = InstaCameraManager.getInstance().getCameraCurrentBatteryLevel();
```

#### 相机系统时间

```Java
// 获取相机系统时间
long mediaTime = InstaCameraManager.getInstance().getMediaTime();
```

### 相机状态获取

#### 是否在充电状态

```Java
// 是否正在充电
boolean isCharging = InstaCameraManager.getInstance().isCameraCharging();
```

#### 是否工作中

```Java
// 相机是否工作中
boolean isWorking = InstaCameraManager.getInstance().isCameraWorking();

//指定的拍摄模式是否正在工作中
boolean isNormalCapturing = InstaCameraManager.getInstance().isCameraWorking(CaptureMode.CAPTURE_NORMAL);
```

#### SD卡是否可用

```Java
// SD卡是否可用
boolean isEnable = InstaCameraManager.getInstance().isSdCardEnabled();
```

## 通知模块

您可以在多个页面上注册/注销`ICameraChangedCallback`，以监视相机状态的变化。

```Java
public abstract class BaseObserveCameraActivity extends AppCompatActivity implements ICameraChangedCallback {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        InstaCameraManager.getInstance().registerCameraChangedCallback(this);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        InstaCameraManager.getInstance().unregisterCameraChangedCallback(this);
    }

    /**
     * 相机状态变化
     *
     * @param enabled:相机是否可用
     * @param connectType:连接类型
     */
    @Override
    public void onCameraStatusChanged(boolean enabled, int connectType) {
    }

    /**
     * 相机连接失败
     * <p>
     * 一种常见的情况是，
     * 其他手机或这款手机的其他应用程序已经与该摄像机建立连接导致该建立失败，
     * 其他手机需要先断开与此摄像头的连接。
     * @param errorCode:错误码  参考相机连接错误码
     */
    @Override
    public void onCameraConnectError(int errorCode) {
    }

    /**
     * SD卡插入通知
     *
     * @param enabled: 当前SD卡是否可用
     */
    @Override
    public void onCameraSDCardStateChanged(boolean enabled) {
    }

    /**
     * SD卡存储状态已更改
     *
     * @param freeSpace:  当前可用容量大小
     * @param totalSpace: 总容量大小
     */
    @Override
    public void onCameraStorageChanged(long freeSpace, long totalSpace) {
    }

    /**
     * 低电量通知
     */
    @Override
    public void onCameraBatteryLow() {
    }

    /**
     * 相机电源更改通知
     *
     * @param batteryLevel: 当前功率（0-100，充电时始终返回100）
     * @param isCharging:   相机是否正在充电
     */
    @Override
    public void onCameraBatteryUpdate(int batteryLevel, boolean isCharging) {
    }

    /**
     * 相机温度变化通知
     *
     * @param tempLevel: 温度等级
     */
    @Override
    public void onCameraTemperatureChanged(TemperatureLevel tempLevel) {
    }
}
```

## 镜头控制模块

相机镜头分为以下三种：

1. 全景镜头：*`SensorMode.PANORAMA`*
2. 前镜头：*`SensorMode.FRONT`*
3. 后镜头：*`SensorMode.REAR`*

### 镜头判断

```Java
// 获取当前的镜头模式
SensorMode sensorMode = InstaCameraManager.getInstance().getCurrentSensorMode();

// 判断是否是单镜头
InstaCameraManager.getInstance().isCameraSingleSensorMode();

// 判断是否是双镜头
InstaCameraManager.getInstance().isCameraDualSensorMode();
```

### 切换镜头

注意：SDK目前只支持全景镜头，非全景镜头可能会出现未知的问题，请切换至全景镜头下使用SDK。

```Java
// 切换到相机全景镜头
InstaCameraManager.getInstance().switchPanoramaSensorMode(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // 镜头切换成功
    }

    @Override
    public void onFailed() {
        // 镜头切换失败
    }

    @Override
    public void onCameraConnectError() {
        // 相机连接错误
    }
});
```

## 预览模块

> 注意：需要跟`Media SDK`一起使用

### 开启预览流

成功连接相机后，您可以像这样操纵相机预览流。

> 如果相机被动断开连接，或者在预览过程中直接调`closeCamera()`，SDK将自动关闭预览流处理相关状态，而无需调用`closePreviewStream()`。

> 注意：下面的代码只是开启了预览流，并不能显示预览内容。需要结合**MediaSDK**的InstaCapturePlayerView控件显示预览内容。

```Java
public class PreviewActivity extends BaseObserveCameraActivity implements IPreviewStatusListener {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_preview);
        // 打开预览
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(this);
        InstaCameraManager.getInstance().startPreviewStream(PreviewStreamResolution.STREAM_1440_720_30FPS, InstaCameraManager.PREVIEW_TYPE_NORMAL);
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (isFinishing()) {
            // 关闭预览
            InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
            InstaCameraManager.getInstance().closePreviewStream();
        }
    }

    @Override
    public void onOpening() {
        // 预览流正在打开      
    }

    @Override
    public void onOpened() {
        // 预览流已打开，可以播放
    }

    @Override
    public void onIdle() {
        // 预览流已停止
    }

    @Override
    public void onError() {
        // 预览失败
    }
        
        @Override
    public void onExposureData(ExposureData exposureData) {
        // 回调频率500Hz
        // exposureData.timestamp: 相机打开后的时间
        // exposureData.exposureTime: 滚动快门曝光时间
    }

    @Override
    public void onGyroData(List<GyroData> gyroList) {
        // 回调频率10Hz, 每组50个数据
        // gyroData.timestamp: 相机打开后的时间
        // gyroData.ax, gyroData.ay, gyroData.az: 三轴加速度
        // gyroData.gx, gyroData.gy, gyroData.gz: 三轴陀螺仪
    }

    @Override
    public void onVideoData(VideoData videoData) {
        // 回调频率500Hz
        // videoData.timestamp: 相机打开后的时间
        // videoData.data: 每帧预览原始流数据
        // videoData.size: 每帧数据的长度
    }

}
```

### 获取当前预览流状态

```Java
int previewStatus = InstaCameraManager.getInstance().getPreviewStatus();

// 预览状态
// InstaCameraManager.PREVIEW_STATUS_IDLE：预览流未开启
// InstaCameraManager.PREVIEW_STATUS_OPENING：预览流正在打开
// InstaCameraManager.PREVIEW_STATUS_OPENED：预览流已打开
```

### 设置预览参数

预览流有以下三种类型：

1. `PREVIEW_TYPE_NORMAL`：用于正常预览或拍摄
2. `PREVIEW_TYPE_RECORD`：仅用于记录
3. `PREVIEW_TYPE_LIVE`：直播

#### 预览分辨率设置

您需要通过下面代码，获得实时支持的相机分辨率

```java
List<PreviewStreamResolution> supportedList = InstaCameraManager.getInstance().getSupportedPreviewStreamResolution(InstaCameraManager.PREVIEW_TYPE_LIVE);
```

> 注意：对于X4、X5，预览流分辨率无法调整，X4默认设置为2880x1440，X5默认设置为2656x1328。X3可以手动设置分辨率为3840x1920/1440x720。
>
> ```java
> //设置分辨率和预览流类型
> InstaCameraManager.getInstance().startPreviewStream(
>     PreviewStreamResolution.STREAM_368_368_30FPS，
>     InstaCameraManager.PREVIEW_TYPE_LIVE
> );
> ```

#### 预览音频

您可以设置预览时是否开启音频：

```java
PreviewParamsBuilder builder = new PreviewParamsBuilder()
        .setStreamResolution(PreviewStreamResolution.STREAM_368_368_30FPS)
        .setPreviewType(InstaCameraManager.PREVIEW_TYPE_LIVE)
        // 是否开启音频
        .setAudioEnabled(true);
InstaCameraManager.getInstance().startPreviewStream(builder);
```

## 拍摄控制模块

### 获取当前拍摄模式

```Java
CaptureMode captureMode = InstaCameraManager.getInstance().getCurrentCaptureMode();
// 当前SDK支持以下几种拍摄模式，不同的相机会存在一些区别，具体以相机屏幕显示为准：
普通拍照：CaptureMode.CAPTURE_NORMAL
HDR拍照：CaptureMode.HDR_CAPTURE
超级夜景：CaptureMode.NIGHT_SCENE
连续拍照：CaptureMode.BURST
间隔拍照：CaptureMode.INTERVAL_SHOOTING
星空模式：CaptureMode.STARLAPSE_SHOOTING
普通录像：CaptureMode.RECORD_NORMAL
HDR录像：CaptureMode.HDR_RECORD
子弹时间：CaptureMode.BULLETTIME
延时摄影：CaptureMode.TIMELAPSE
移动延时：CaptureMode.TIME_SHIFT
循环录影：CaptureMode.LOOPER_RECORDING
超级录像：CaptureMode.SUPER_RECORD
慢动作：CaptureMode.SLOW_MOTION
跟拍模式：CaptureMode.SELFIE_RECORD
夜景录像：CaptureMode.PURE_RECORD
直播：CaptureMode.LIVE
```

### 模式切换

```Java
InstaCameraManager.getInstance().setCaptureMode(captureMode, errorCode -> {
    if (errorCode == 0) {
        // 切换成功
    } else {
        // 切换失败
    }
});
```

### 拍摄状态通知

> 注意：X3以及X3之前的相机录制视频超过30分钟的部分会分为2次录制，相机会自动停止录制第一段视频之后开始录制第二段视频。

```Java
InstaCameraManager.getInstance().setCaptureStatusListener(new ICaptureStatusListener() {
    @Override
    public void onCaptureCountChanged(int captureCount) {
        // 间隔拍照，星空模式等模式的拍摄张数，
    }

    @Override
    public void onCaptureStarting() {
        // 拍摄即将开始
    }

    @Override
    public void onCaptureStopping() {
        // 拍摄即将结束
    }
    
    @Override
    public void onCaptureTimeChanged(long captureTime) {
       // 录制时间回调，只有录像模式下才有
    }

    @Override
    public void onCaptureWorking() {
        // 拍摄中
    }

    @Override
    public void onCaptureFinish(String[] filePaths) {
        // 拍摄结束，返回成片路径
    }
    
    @Override
    public void onCaptureError(int errorCode) {
        // 拍摄失败 error参考 拍摄失败错误码
    }
});
```

### 参数设置

#### 拍摄参数

当前SDK支持的拍摄属性如下：

- 曝光模式：*`CaptureSetting.EXPOSURE`* 

```Plain
FULL_AUTO     = Exposure.FULL_AUTO,     // 全自动曝光
AUTO          = Exposure.AUTO,          // 自动曝光
ISO_FIRST     = Exposure.ISO_FIRST,     // ISO优先
SHUTTER_FIRST = Exposure.SHUTTER_FIRST, // 快门优先
MANUAL        = Exposure.MANUAL,        // 手动曝光
ADAPTIVE      = Exposure.ADAPTIVE       // 全景独立曝光
```

- 全⾃动曝光：*`Exposure.FULL_AUTO`*

- 自动曝光：*`Exposure.AUTO`*

- ISO优先：*`Exposure.ISO_FIRST`*

- 快门优先：*`Exposure.SHUTTER_FIRST`*

- ⼿动曝光：*`Exposure.MANUAL`*

- 全景独立曝光：*`Exposure.ADAPTIVE`*

- EV：*`CaptureSetting.EV`*

- EV间隔：*`CaptureSetting.EV_INTERVAL`*

- 快门速度：*`CaptureSetting.SHUTTER`*

- 快门模式：*`CaptureSetting.SHUTTER_MODE`*

```Plain
AUTO   = ShutterMode.AUTO,   // 自动
SPORT  = ShutterMode.SPORT,  // 室内暗光防抖
FASTER = ShutterMode.FASTER  // 极快
```

- ISO：*`CaptureSetting.ISO`*

- 最大ISO：*`CaptureSetting.ISO_TOP_LIMIT`*

- 录像分辨率&帧率：*`CaptureSetting.RECORD_RESOLUTION`*

- 拍照分辨率：*`CaptureSetting.PHOTO_RESOLUTION`*

- 白平衡：*`CaptureSetting.WB`*

- AEB：*`CaptureSetting.AEB`*

- 间隔时长：*`CaptureSetting.INTERVAL`*

- 色彩：*`CaptureSetting.GAMMA_MODE`*

```Plain
STAND  = GammaMode.STAND,  // 标准
VIVID  = GammaMode.VIVID,  // LOG
LOG    = GammaMode.LOG,    // 鲜艳
FLAT   = GammaMode.FLAT    // 灰调
```

- 照片格式：*`CaptureSetting.RAW_TYPE`*

```Plain
JPG           = RawType.OFF,          // JPG
JPG_RAW       = RawType.DNG,          // JPG & RAW
PURESHOT      = RawType.PURESHOT,     // PURESHOT
PURESHOT_RAW  = RawType.PURESHOT_RAW, // PURESHOT & RAW
INSP          = RawType.INSP,         // INSP
INSP_RAW      = RawType.INSP_RAW      // RAW & INSP
```

- 录像时长：*`CaptureSetting.RECORD_DURATION`*

- 暗光增稳：*`CaptureSetting.DARK_EIS_ENABLE`*

- 全景独立曝光：*`CaptureSetting.PANO_EXPOSURE_MODE`*

```Plain
OFF       = PanoExposureMode.OFF,       // 关闭
ON        = PanoExposureMode.ON,        // 独立曝光
LIGHT     = PanoExposureMode.LIGHT,     // 标准
LSOLATED  = PanoExposureMode.LSOLATED   // 高
```

> 注意：Insta360 X4相机之前的相机只有开（ON）和关（OFF）2种状态，X4之后的相机的开状态分成了LIGHT、LSOLATED

- 连拍张数：*`CaptureSetting.BURST_CAPTURE`*

- 机内拼接：*`CaptureSetting.INTERNAL_SPLICING`*

```Plain
ON  = InternalSplicing.ON,   // 开
OFF = InternalSplicing.OFF   // 关
```

- *录像HDR模式：**`CaptureSetting.HDR_STATUS`*

```Plain
ON  = HdrStatus.ON,   // 开
OFF = HdrStatus.OFF   // 关
```

注意：从Insta360 X5相机开始，取消DHR录像*`CaptureMode.HDR_RECORD`*，增加*`CaptureSetting.HDR_STATUS`*属性设置以替代。

- 拍照HDR模式：*`CaptureSetting.PHOTO_HDR_TYPE`*

```Plain
开：PhotoHdrType.HDR_ON
关：PhotoHdrType.HDR_OFF
AEB：PhotoHdrType.HDR_AEB
```

> 注意：从Insta360 X5相机开始，取消HDR拍照*`CaptureMode.HDR_CAPTURE`*，增加*`CaptureSetting.PHOTO_HDR_TYPE`*属性设置以替代。

- I-LOG：*`CaptureSetting.I_LOG`*

```Plain
ON  = ILogStatus.ON,   // 开
OFF = ILogStatus.OFF   // 关
```

- 拍照倒计时：*`CaptureSetting.LATENCY`* 

> 注意：仅X5支持

#### 获取当前拍摄参数

您可以通过*`InstaCameraManager.getInstance().getXxx(CaptureMode)`*方法来获取对应的拍摄属性；

| 方法                        | 参数        | 返回值           | 说明                  |
| --------------------------- | ----------- | ---------------- | --------------------- |
| getRecordDuration()         | CaptureMode | RecordDuration   | 获取录制时长          |
| getRecordResolution()       | CaptureMode | RecordResolution | 获取录像分辨率，帧率  |
| getPhotoResolution()        | CaptureMode | PhotoResolution  | 获取拍照分辨率        |
| getInterval()               | CaptureMode | Interval         | 获取拍摄间隔时长      |
| getHdrStatus()              | CaptureMode | HdrStatus        | 获取普通录像中HDR开关 |
| getPhotoHdrType()           | CaptureMode | PhotoHdrType     | 获取普通拍照中HDR开关 |
| getGammaMode()              | CaptureMode | GammaMode        | 获取色彩              |
| getRawType()                | CaptureMode | RawType          | 获取照片格式          |
| getPanoExposureMode()       | CaptureMode | PanoExposureMode | 获取全景独立曝光开关  |
| getExposure()               | CaptureMode | Exposure         | 获取曝光模式          |
| getISO()                    | CaptureMode | ISO              | 获取ISO               |
| getISOTopLimit()            | CaptureMode | ISOTopLimit      | 获取ISO上限值         |
| getShutterMode()            | CaptureMode | ShutterMode      | 获取快门类型          |
| getShutter()                | CaptureMode | Shutter          | 获取快门速度          |
| getEv()                     | CaptureMode | Ev               | 获取EV                |
| getEVInterval()             | CaptureMode | EVInterval       | 获取EV间隔            |
| getAEB()                    | CaptureMode | AEB              | 获取AEB               |
| getWB()                     | CaptureMode | WB               | 获取白平衡            |
| getInternalSplicingEnable() | CaptureMode | InternalSplicing | 获取机内拼接开关      |
| getDarkEisType()            | CaptureMode | DarkEisType      | 获取暗光增稳开关      |
| getBurstCapture()           | CaptureMode | BurstCapture     | 获取连拍参数          |
| getILogStatus()             | CaptureMode | ILogStatus       | 获取I-LOG开关         |
| getLatency()                | CaptureMode | Latency          | 获取拍照倒计时        |

#### 设置拍摄属性

您可以通过*`InstaCameraManager.getInstance().setXxx(CaptureMode)`*方法来获取对应的拍摄属性。

| 方法                        | 参数1       | 参数2            | 返回值 | 说明                  |
| --------------------------- | ----------- | ---------------- | ------ | --------------------- |
| setRecordDuration()         | CaptureMode | RecordDuration   | void   | 设置录制时长          |
| setRecordResolution()       | CaptureMode | RecordResolution | void   | 设置录像分辨率，帧率  |
| setPhotoResolution()        | CaptureMode | PhotoResolution  | void   | 设置拍照分辨率        |
| setInterval()               | CaptureMode | Interval         | void   | 设置拍摄间隔时长      |
| setHdrStatus()              | CaptureMode | HdrStatus        | void   | 设置普通录像中HDR开关 |
| setPhotoHdrType()           | CaptureMode | PhotoHdrType     | void   | 设置普通拍照中HDR开关 |
| setGammaMode()              | CaptureMode | GammaMode        | void   | 设置色彩              |
| setRawType()                | CaptureMode | RawType          | void   | 设置照片格式          |
| setPanoExposureMode()       | CaptureMode | PanoExposureMode | void   | 设置全景独立曝光开关  |
| setExposure()               | CaptureMode | Exposure         | void   | 设置曝光模式          |
| setISO()                    | CaptureMode | ISO              | void   | 设置ISO               |
| setISOTopLimit()            | CaptureMode | ISOTopLimit      | void   | 设置ISO上限值         |
| setShutterMode()            | CaptureMode | ShutterMode      | void   | 设置快门类型          |
| setShutter()                | CaptureMode | Shutter          | void   | 设置快门速度          |
| setEv()                     | CaptureMode | Ev               | void   | 设置EV                |
| setEVInterval()             | CaptureMode | EVInterval       | void   | 设置EV间隔            |
| setAEB(CaptureMode          | CaptureMode | AEB              | void   | 设置AEB               |
| setWB()                     | CaptureMode | WB               | void   | 设置白平衡            |
| setInternalSplicingEnable() | CaptureMode | InternalSplicing | void   | 设置机内拼接开关      |
| setDarkEisType()            | CaptureMode | DarkEisType      | void   | 设置暗光增稳开关      |
| setBurstCapture()           | CaptureMode | BurstCapture     | void   | 设置连拍参数          |
| setILogStatus               | CaptureMode | ILogStatus       | void   | 设置I-LOG开关         |
| setLatency()                | CaptureMode | Latency          | void   | 设置拍照倒计时        |

#### 批量设置拍摄属性

```Java
//开始批量设置拍摄属性
InstaCameraManager.getInstance().beginSettingOptions();

InstaCameraManager.getInstance().setEv(captureMode, EV.EV_0);
InstaCameraManager.getInstance().setShutter(captureMode, Shutter.SHUTTER_1_2);
...
InstaCameraManager.getInstance().setWB(captureMode, WB.WB_2200K);

//提交批量设置以生效
InstaCameraManager.getInstance().commitSettingOptions();
```

#### 拍摄属性之间的依赖关系

不同的拍摄属性之间存在相互影响的关系。当设置 A 属性时，可能会对 B 属性的**[支持列表](https://arashivision.feishu.cn/wiki/Bc4VwYQBHiCu82kCooMcNBVun4g#share-Mwxzdh7kNoIIoyxgBKvcx9VKn2e)**产生影响。例如快门速度和FPS之间的影响。因此，在设置属性之后，有必要对其影响的其他属性进行检查。

> 注意：因为回调函数可能会回调多次，每次回调中的CaptureSetting列表元素都可能不一样

示例代码如下： 

```Java
// 示例中仅展示setPhotoHdrType的案例，其他属性设置与此相同；
InstaCameraManager.getInstance().setPhotoHdrType(captureMode, photoHdrType, new InstaCameraManager.IDependChecker() {
    @Override
    public void check(List<CaptureSetting> captureSettings) {
        // captureSettings：受到影响拍摄属性列表
        // 可在此处更新UI
    }
});
```

### 拍照

#### 判断是否拍照模式

```Java
boolean isPhotoMode = CaptureMode.PURE_RECORD.isPhotoMode();
```

#### 开始拍照

```Plain
// 开始普通拍照
InstaCameraManager.getInstance().startNormalCapture();

// 开始HDR拍照
InstaCameraManager.getInstance().startHDRCapture();

// 开始超级夜景
InstaCameraManager.getInstance().startNightScene();

// 开始连续拍照
InstaCameraManager.getInstance().startBurstCapture();

// 开始间隔拍照
InstaCameraManager.getInstance().startIntervalShooting();
// 停止间隔拍照
InstaCameraManager.getInstance().stopIntervalShooting();

// 开始星空模式
InstaCameraManager.getInstance().startStarLapseShooting();
// 停止星空模式
InstaCameraManager.getInstance().stopStarLapseShooting();
```

### 录像

#### 判断是否是录像模式

```Java
boolean isVideoMode = CaptureMode.PURE_RECORD.isVideoMode();
```

#### 开始/停止录像

```Plain
// 开始普通录像
InstaCameraManager.getInstance().startNormalRecord();
// 停止普通录像
InstaCameraManager.getInstance().stopNormalRecord();

// 开始HDR录像
InstaCameraManager.getInstance().startHDRRecord();
// 停止HDR录像
InstaCameraManager.getInstance().stopHDRRecord();

// 开始子弹时间
InstaCameraManager.getInstance().startBulletTime();
// 停止子弹时间
InstaCameraManager.getInstance().stopBulletTime();

// 开始延时摄影
InstaCameraManager.getInstance().startTimeLapse();
// 停止延时摄影
InstaCameraManager.getInstance().stopTimeLapse();

// 开始移动延时
InstaCameraManager.getInstance().startTimeShift();
// 停止移动延时
InstaCameraManager.getInstance().stopTimeShift();

// 开始循环录影
InstaCameraManager.getInstance().startLooperRecord();
// 停止循环录影
InstaCameraManager.getInstance().stopLooperRecord();

// 开始超级录像
InstaCameraManager.getInstance().startSuperRecord();
// 停止超级录像
InstaCameraManager.getInstance().stopSuperRecord();

// 开始慢动作
InstaCameraManager.getInstance().startSlowMotionRecord();
// 停止慢动作
InstaCameraManager.getInstance().stopSlowMotionRecord();

// 开始跟拍模式
InstaCameraManager.getInstance().startSelfieRecord();
// 停止跟拍模式
InstaCameraManager.getInstance().stopSelfieRecord();

// 开始夜景录像
InstaCameraManager.getInstance().startPureRecord();
// 停止夜景录像
InstaCameraManager.getInstance().stopPureRecord();
```

## 直播模块

> 注意：需要跟`Media SDK`一起使用

### 判断是否是直播模式

```Java
boolean isLiveMode = CaptureMode.PURE_RECORD.isLiveMode();
```

### 开始直播

> 注意：预览流关闭之后，相机会自动把拍摄模式切换至普通录像。开支直播之前需要先检查当前拍摄模式是否是直播。

```java
LiveParamsBuilder builder = new LiveParamsBuilder()
         //（必须）将rtmp地址设置为推送流
        .setRtmp(String rtmp)
        //（必须）设置推送的宽度，例如1440
        .setWidth(int width)
        //（必须）设置推送的高度，例如720
        .setHeight(int height)
        // 必须）设置推送的帧率，例如30
        .setFps(int fps)
        //（必须）设置要推送的比特率，例如2*1024*1024
        .setBitrate(int bitrate)
        //（可选）无论直播是否为全景，默认值均为true
        .setPanorama(true);
        
InstaCameraManager.getInstance().startLive(builder, new ILiveStatusListener() {

    @Override
    public void onLiveFpsUpdate(int fps) {
        // fps：当前直播帧率
    }

    @Override
    public void onLivePushError(int error, String desc) {
        // 直播推流失败
    }

    @Override
    public void onLivePushFinished() {
        // 直播推流结束
    }

    @Override
    public void onLivePushStarted() {
        // 直播推流开始
    }
});
```

### 结束直播

```java
InstaCameraManager.getInstance().stopLive();
```

## 文件管理模块

### 获取相机套接字地址

```java
// 返回值示例：http://192.168.42.1:80
InstaCameraManager.getInstance().getCameraHttpPrefix();
```

### 获取文件URL

```java
// 返回值示例：[/DCIM/Camera01/VID_20250326_184618_00_001.insv]
List<String> usls = InstaCameraManager.getInstance().getAllUrlList();

// 包含录像中文件
List<String> usls = InstaCameraManager.getInstance().getAllUrlListIncludeRecording();
```

### 文件删除

```java
// 删除单个相机文件
InstaCameraManager.getInstance().deleteFile(String filePath，new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // 删除成功
    }

    @Override
    public void onFailed() {
       // 删除失败
    }

    @Override
    public void onCameraConnectError() {
      // 相机连接错误
    }
});

// 批量删除相机文件
InstaCameraManager.getInstance().deleteFileList(List<String> fileUrlList，new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // 删除成功
    }

    @Override
    public void onFailed() {
       // 删除失败
    }

    @Override
    public void onCameraConnectError() {
      // 相机连接错误
    }
});
```

## 日志记录模块

### 设置日志路径

SDK的日志系统支持将日志保存到本地文件中，调用如下代码给日志设置保存位置。

```Java
// 设置日志缓存路径，如不设置，则不缓存
LogManager.instance.setLogRootPath(StorageUtils.getInternalRootPath() + "/log");
```

### 日志开关

开启全局日志实时抓取功能，该功能会抓取所有日志，包括SDK日志，系统日志，宿主APP日志等，并保存在指上面指定的目录中。

```Java
// 开启日志实时抓取功能
LogManager.instance.startLogDumper();

// 停止日志实时抓取功能
LogManager.instance.stopLogDumper();
```

## GPS数据模块

### 拍照模式下设置

```java
// android.location.Location 类，开发者调用系统api自行获取
Location location;

// 拍照时设置GPS数据
GpsData gpsData = new GpsData();
gpsData.setLatitude(location.getLatitude());
gpsData.setLongitude(location.getLongitude());
gpsData.setAltitude(location.getAltitude());
InstaCameraManager.getInstance().startNormalCapture(gpsData.simpleGpsDataToByteArray());
```

#### 不同拍照模式下传入GPS数据接口

```Plain
// 开始普通拍照，并传入GPS数据
InstaCameraManager.getInstance().startNormalCapture(byte[]);

// 开始HDR拍照，并传入GPS数据
InstaCameraManager.getInstance().startHDRCapture(byte[]);

// 开始连续拍照，并传入GPS数据
InstaCameraManager.getInstance().startBurstCapture(byte[]);

// 开始间隔拍照，并传入GPS数据
InstaCameraManager.getInstance().stopIntervalShooting(byte[]);
```

### 录像模式下设置

录像设置GPS数据分为以下2种情况：

#### 停止录像时

停止录像时和拍照一样，示例代码如下：

```java
// android.location.Location 类，开发者调用系统api自行获取
Location location;

// 停止录像时设置GPS数据
GpsData gpsData = new GpsData();
gpsData.setLatitude(location.getLatitude());
gpsData.setLongitude(location.getLongitude());
gpsData.setAltitude(location.getAltitude());
InstaCameraManager.getInstance().stopNormalRecord(gpsData.simpleGpsDataToByteArray());
```

#### 录像中

录像中可以在的任意时刻设置GPS数据。录像中设置GPS数据，一般是用来制作相机的运动轨迹。因此建议每次设置GPS的时间间隔尽量相同。示例代码如下：

```java
// 100ms获取一次gps数据，累计20个gps数据，就设置给相机

private Handler mMainHandler = new Handler(Looper.getMainLooper());
// GPS数据列表
private List<GpsData> mCollectGpsDataList = new ArrayList<GpsData>();

private Runnable mCollectGpsDataRunnable = new Runnable() {
    @Override
    public void run() {
        // android.location.Location 类，开发者调用系统api自行获取
        Location location；
        if (location != null) {
            GpsData gpsData = new GpsData();
            gpsData.setLatitude(location.getLatitude());
            gpsData.setLongitude(location.getLongitude());
            gpsData.setSpeed(location.getSpeed());
            gpsData.setBearing(location.getBearing());
            gpsData.setAltitude(location.getAltitude());
            gpsData.setUTCTimeMs(location.getTime());
            gpsData.setVaild(true);
            mCollectGpsDataList.add(gpsData);
            if (mCollectGpsDataList.size() >= 20) {
                InstaCameraManager.getInstance().setGpsData(GpsData.gpsDataToByteArray(mCollectGpsDataList));
                mCollectGpsDataList.clear();
            }
        }
        mMainHandler.postDelayed(this, 100);
    }
};

// 开始收集GPS数据
mMainHandler.post(mCollectGpsDataRunnable);
```

#### 不同录像模式下传入GPS数据接口

```Plain
// 停止普通录像，并传入GPS数据
InstaCameraManager.getInstance().stopNormalRecord(byte[]);

// 停止HDR录像，并传入GPS数据
InstaCameraManager.getInstance().stopHDRRecord(byte[]);

// 停止子弹时间，并传入GPS数据
InstaCameraManager.getInstance().stopBulletTime(byte[]);

// 停止延时摄影，并传入GPS数据
InstaCameraManager.getInstance().stopTimeLapse(byte[]);

// 停止移动延时，并传入GPS数据
InstaCameraManager.getInstance().stopTimeShift(byte[]);

// 停止超级录像，并传入GPS数据
InstaCameraManager.getInstance().stopSuperRecord(byte[]);

// 停止循环录影，并传入GPS数据
InstaCameraManager.getInstance().stopLooperRecord(byte[]);

// 停止夜景录像，并传入GPS数据
InstaCameraManager.getInstance().stopPureRecord(byte[]);

// 停止跟拍模式，并传入GPS数据
InstaCameraManager.getInstance().stopSelfieRecord(byte[]);
```

## 其它功能模块

### 激活相机

激活相机需要用到`appId`和`secretKey`这两个数据。这两个数据需要向Insta360官方申请。

> 注意：激活需要用到网络权限

> 注意：Nano S 相机不支持此功能。

示例代码如下：

```Java
String appid = "xxxxxxxx";
String mSecretKey = "xxxxxxx";
InstaCameraManager.getInstance().activateCamera(new SecretInfo(appid , mSecretKey), new InstaCameraManager.IActivateCameraCallback() {
    @Override
    public void onStart() {
        // 开始激活
    }

    @Override
    public void onSuccess() {
        // 激活成功
    }

    @Override
    public void onFailed(String message) {
        // 激活失败  
    }
});
```

### 关闭提示音

> 注意：Nano S 相机不支持获取提示音开关，但支持设置。

相机提示音的开关，示例代码如下：

```Java
// 打开相机提示音
InstaCameraManager.getInstance().setCameraBeepSwitch(true);

// 判断相机是否开启提示音
boolean isBeep = InstaCameraManager.getInstance().isCameraBeep();
```

### 格式化SD卡

```java
InstaCameraManager.getInstance().formatStorage(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // SD卡格式化成功
    }

    @Override
    public void onFailed() {
        // SD卡格式化失败
    }

    @Override
    public void onCameraConnectError() {
        // 相机连接错误
    }
});
```

### 相机锁屏

> 注意：仅支持X4、X5

相机锁屏之后用户无法操作相机屏幕，示例代码如下：

```Java
// true：锁屏   false:解除锁屏
InstaCameraManager.getInstance().setCameraLockScreen(true);
```

### 关闭相机

控制相机关机的指令，示例代码如下：

> 注意：仅Insta360X5以及之后的相机支持该功能；

```Java
InstaCameraManager.getInstance().shutdownCamera();
```

### 陀螺仪校准

> 注意：Nano S 相机不支持此功能。

当相机连接到Wi-Fi时，应使用此功能。校准前，请确保相机垂直放置在稳定水平的表面上。

```java
InstaCameraManager.getInstance().calibrateGyro(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
    }

    @Override
    public void onFailed() {
    }

    @Override
    public void onCameraConnectError() {
    }
});
```

### 固件升级

通过`FwUpgradeManager`类去进行固件版本的升级，示例代码如下：

> 注意：Nano S 相机不支持此功能。

> 注意：请确保将正确的文件上传到相机，以防止任何潜在的损坏。例如，不要将ONE X固件升级包上传到ONE X2相机。

```Java
// 判断是否正在升级
FwUpgradeManager.getInstance().isUpgrading()；

// 取消升级
FwUpgradeManager.getInstance().cancelUpgrade();

String fwFilePath = "/xxx/InstaOneXFW.bin";
// 升级固件
FwUpgradeManager.getInstance().startUpgrade(fwFilePath, new FwUpgradeListener() {
    @Override
    public void onUpgradeSuccess() {
        // 升级成功回调
    }

    @Override
    public void onUpgradeFail(int errorCode, String message) {
         // 升级失败回调
         // 固件升级错误码
    }

    @Override
    public void onUpgradeCancel() {
         // 升级取消回调
    }

    @Override
    public void onUpgradeProgress(double progress) {
        // 升级进度回调：0-1
    }
});
```

### 锐度设置（目前不支持）

## 错误码

### 相机连接错误码

| 错误码 | 解释                                                 |
| ------ | ---------------------------------------------------- |
| -122   | 蓝牙设备断连                                         |
| 501    | 蓝牙发生断连                                         |
| 2201   | 系统报错 BleException.*ERROR_CODE_SYSTEM_STATUS_133* |
| 2002   | 打开相机超时                                         |
| 2206   | 系统报错 BleException.*ERROR_CODE_NOTIFY_FAILURE*    |
| 2207   | 系统报错 BleException.*ERROR_CODE_SHAKE_HAND*        |
| 3201   | socket连接失败                                       |
| 3205   | socket连接超时                                       |
| 3206   | socket重连超时                                       |
| 4001   | 系统报错 BleException.*ERROR_CODE_SYNC_PACK*         |
| 4101   | 相机类型错误                                         |
| 4103   | *相机同步参数超时*                                   |
| 4402   | *相机操作系统错误*                                   |
| 4403   | 相机被占用                                           |
| 4405   | 解析proto失败                                        |
| 4406   | *相机同步参数重试超时*                               |
| 4407   | 相机数据异常                                         |
| 4414   | *唤醒蓝牙失败*                                       |
| 4417   | 操作过程中相机断开连接                               |
| 4504   | 连接时摄像头断开连接                                 |
| 4505   | 蓝牙绑定被拒绝                                       |
| 4507   | 蓝牙写入失败                                         |
| 4508   | *设备蓝牙繁忙*                                       |
| 4602   | 设备Wi-Fi繁忙                                        |
| 4605   | *USB错误*                                            |
| 4606   | *socket因脏数据断连*                                 |
| -14169 | 同步相机参数失败，原因是蓝牙挂了导致的               |

### 固件升级错误码

| 错误码 | 解释                                                         |
| ------ | ------------------------------------------------------------ |
| -1000  | 已在升级中                                                   |
| -1001  | 相机未连接                                                   |
| -1002  | 当相机的电池电量低于12%时，升级将失败，请确保相机的电池电量充足 |
| -14004 | Http服务器错误                                               |
| 400    | Http服务器错误                                               |
| 500    | Http服务器错误                                               |
| -14005 | 套接字IO异常                                                 |

### 拍摄失败错误码

| 错误码 | 解释                 |
| ------ | -------------------- |
| -14161 | 电量过低             |
| -14162 | 超过最大录制时长限制 |
| -14163 | SD卡已满             |
| -14165 | 超过文件数量限制     |
| -14166 | SD卡速度低           |
| -14167 | 编码异常             |
| -14168 | 帧丢失               |
| -14180 | 文件碎片过多         |
| -14181 | 相机温度过高         |
| -14182 | 低功率启动           |
| -14183 | 存储偏转启动         |
| -14184 | 高温启动             |
| -14164 | 其他情况             |

### 相机激活错误码

| 错误码 | 解释                                         |
| ------ | -------------------------------------------- |
| 400    | 参数错误，可能是相机类型或者序列号没有获取到 |
| 401    | 接口签名验证失败                             |
| 402    | 数据加密验证失败                             |
| 403    | nonce凭证已经被使用过                        |
| 406    | timestamp已经过期                            |
| 429    | 接口请求频繁                                 |
| 500    | 系统内部异常                                 |
| 3001   | 设备序列号和设备类型不匹配                   |
| 3002   | 相机已被激活                                 |
| 3003   | 拒绝激活，激活ip不在合法区域内               |
| 3004   | 无法生成可用序列号                           |
| 3005   | 系统不支持激活该型号的设备                   |
| 3006   | 序列号被后台锁定，不允许激活                 |
| 429    | 激活ip被限流，限流策略：45次/ip/天           |

# Media SDK功能

## 环境准备

- 将maven地址添加到构建文件（项目根目录中的`build.gradle`文件）中

```Groovy
allprojects {
    repositories {
        ...
        maven {
            url 'XXXXXX'
            credentials {
                // see sdk demo
                username = '***'
                password = '***'
            }
        }
    }
}
```

- 在app目录的`build.gradle`文件中导入依赖库

```Groovy
dependencies {
    implementation 'com.arashivision.sdk:sdkmedia:x.x.x'
}
```

## 预览模块

如果您已经集成了`Camera SDK`，则可以打开并显示预览内容。

### 预览控件初始化

您需要将*`InstaCapturePlayerView`*放入您的XML文件中，并且绑定Activity的生命周期：

```XML
<com.arashivision.sdkmedia.player.capture.InstaCapturePlayerView
    android:id="@+id/player_capture"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/darker_gray"
/>
@Override 
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mCapturePlayerView.setLifecycle(getLifecycle());
}
```

### 显示预览画面

在`Camera SDK`的*`IPreviewStatusListener.onOpen()`*回调中显示预览画面

```Java
@Overridepublic void onOpened() {
    InstaCameraManager.getInstance().setStreamEncode();
    mCapturePlayerView.setPlayerViewListener(new PlayerViewListener() {
        @Override
        public void onLoadingFinish() {
            // 固定写法
            InstaCameraManager.getInstance().setPipeline(mCapturePlayerView.getPipeline());
        }
        
        @Override
        public void onLoadingStatusChanged(boolean isLoading) {
            // loading状态改变
        }

        @Override
        public void onFail(String desc) {
        }
        
        @Override
        public void onFirstFrameRender() {
            // 首帧有效渲染
        }
    });
    // 设置参数，后面会详细描述
    mCapturePlayerView.prepare(createParams());
    // 播放
    mCapturePlayerView.play();
    // 保持屏幕常亮
    mCapturePlayerView.setKeepScreenOn(false);
}
```

### 释放资源

当关闭预览流时，需要释放掉*`InstaCapturePlayerView`*

```Java
@Override
protected void onStop() {
    super.onStop();
    if (isFinishing()) {
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
        InstaCameraManager.getInstance().closePreviewStream();
        mCapturePlayerView.destroy();
    }
}

// yilanli
@Override
public void onIdle() {
    mCapturePlayerView.destroy();
    mCapturePlayerView.setKeepScreenOn(false);
}
```

### 设置预览参数

```Java
private CaptureParamsBuilderV2createParams() {
    CaptureParamsBuilderV2builder = new CaptureParamsBuilderV2()
            // (可选) 设置您的分辨率和帧率，默认值由SDK内部智能生成
            .setWidth(int width)
            .setHeight(int height)
            .setFps(int fps) //最大30fps
            // (可选) 消色差  默认值为true
            .setImageFusionEnabled(boolean enable)
            // (可选) 设置手势开关，默认值由SDK内部智能生成
            .setGestureEnabled(boolean enable)
            // (可选) 设置防抖缓存帧数，缓存帧数越多，防抖效果越好，但画面延迟也会越高
            .setStabCacheFrameNum(5)
            // (可选) 设置防抖类型，默认值由SDK内部智能生成
            .setStabType(@InstaStabType.StabType int mStabType)
            //（可选）设置渲染模式，默认值为"RenderModel.AUTO"
            .setRenderModel(RenderModel.PLANE_STITCH)
            //（可选）设置屏幕比例。如9：16这种，默认值由SDK内部智能生成
            // 如果渲染模式类型为“PLANE_STITCH”，建议设置比为2:1
            .setScreenRatio(int,int)
            //（可选）设置自定义Surface信息
            .setCameraRenderSurfaceInfo(CameraRenderSurfaceInfo);
    return builder;
}
```

### 切换渲染模式

```Java
// 普通
mCapturePlayerView.switchNormalMode();
// 鱼眼
mCapturePlayerView.switchFisheyeMode();
// 透视
mCapturePlayerView.switchPerspectiveMode();
```

### 视角视距

您可以根据自己的喜好自定义视角和视距。

> Fov：视角范围 0~180
>
> Distance：视距范围  0~∞

```Java
mCapturePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### 手势操作

您可以将*`PlayerGestureListener`*设置为观察手势操作：

```Java
mCapturePlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // 用户手指按下事件
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // 用户手指单击事件，类似于View.OnClickListener.OnClick(v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // 用户手指抬起事件
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // 用户长按事件
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // 用户双指缩放事件
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // 缩放达到最大或最小是的回弹动画
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // 缩放回弹动画结束
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // 用户手指滑动事件
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // 快速滑动动画
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // 快速滑动动画结束
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## WorkWrapper

`WorkWrapper`是一次拍摄所有数据的一个集合，例如：连拍会生成多个文件，这些文件就会被封装成一个`WorkWrapper`来统一处理，后续对文件的播放、导出和信息获取都是对`WorkWrapper`对象操作实现的。

### 如何获取WorkWrapper

您可以从相机或本地目录扫描媒体文件以获取`List<WorkWrapper>`。

> 注意：这是一个耗时的操作，需要在子线程中处理。

```Java
// 从相机中获取
List<WorkWrapper> list = WorkUtils.getAllCameraWorks();

// 从本地文件中获取
List<WorkWrapper> list = WorkUtils.getAllLocalWorks(String dirPath);

// 如果您有媒体文件的url，您也可以自己创建WorkWrapper
String[] urls = {img1.insv, img2.insv, img3.insv};
WorkWrapper workWrapper = new WorkWrapper(urls);
```

### 加载文件尾数据

调用*`WorkWrapper.loadExtraData()`*函数加载文件尾数据。加载文件尾数据之后才可以使用`播放`和`导出`功能。

调用*`WorkWrapper.isExtraDataLoaded()`*函数可以判断文件尾数据是否已经加载过。

> 注意：该方法是耗时操作，需要放在IO线程中执行。

### 读取WorkWrapper的内容

您可以从*`WorkWrapper`*获取到的信息如下：

| 方法                      | 参数    | 返回值类型     | 说明                                                         |
| ------------------------- | ------- | -------------- | ------------------------------------------------------------ |
| getIdenticalKey()         | -       | String         | 用于Glide或其他的DiskCacheKey                                |
| getUrls()                 | -       | String[]       | 获取媒体文件URL                                              |
| getUrls()                 | boolean | String[]       | 获取媒体文件URL。设置true/false以决定是否包含dng文件路径     |
| getAllUrls()              | -       | String[]       | 获取所有文件路径，用来下载和删除等操作                       |
| getWidth()                | -       | int            | 获取宽度                                                     |
| getHeight()               | -       | int            | 获取高度                                                     |
| getBitrate()              | -       | int            | 获取视频比特率，如果是照片，则返回0                          |
| getFps()                  | -       | double         | 获取视频帧率，如果是照片，则返回0                            |
| isPhoto()                 | -       | boolean        | 是否为照片                                                   |
| isHDRPhoto()              | -       | boolean        | 是否为HDR照片                                                |
| supportPureShot()         | -       | boolean        | 是否支持PureShot算法                                         |
| isVideo()                 | -       | boolean        | 是否为视频                                                   |
| isHDRVideo()              | -       | boolean        | 是否为HDR视频                                                |
| isCameraFile()            | -       | boolean        | 媒体文件是否来源于相机设备                                   |
| isLocalFile()             | -       | boolean        | 媒体文件是否来源于手机设备                                   |
| getCreationTime()         | -       | long           | 获取媒体文件拍摄时间戳，单位：ms                             |
| getFirstFrameTimeOffset() | -       | long           | 获取拍摄媒体文件时时间戳相对于相机启动的偏移量，用于匹配陀螺仪、曝光和其他数据，单位：ms |
| getGyroInfo()             | -       | GyroInfo[]     | 获取媒体文件的陀螺仪数据                                     |
| getExposureData()         | -       | ExposureData[] | 获取媒体文件的曝光数据                                       |
| loadThumbnail()           | -       | Bitmap         | 加载缩略图（耗时操作）                                       |
| loadExtraData()           | -       | -              | 加载文件尾数据（耗时操作）                                   |
| isExtraDataLoaded()       | -       | boolean        | 是否加载过文件尾数据                                         |

## 图片播放器

如果要播放图像或视频，必须先创建`WorkWrapper`。

### 播放器初始化

将*`InstaImagePlayerView`*放入xml文件中，并且绑定Activity的生命周期。

```XML
<com.arashivision.sdkmedia.player.image.InstaImagePlayerView
    android:id="@+id/player_image"
    android:layout_width="match_parent"
    android:layout_height="match_parent" 
/>
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mImagePlayerView.setLifecycle(getLifecycle());
}
```

### 设置播放参数

```java
ImageParamsBuilder builder = new ImageParamsBuilder()
      // (可选) 是否启用动态拼接，默认为true
      .setDynamicStitch(boolean dynamicStitch)
      // (可选) 设置防抖类型，默认为InstaStabType.STAB_TYPE_OFF
      .setStabType(@InstaStabType.StabType int mStabType)
      // (可选) 设置播放代理文件，如拼接生成的HDR.jpg，默认为空
      .setUrlForPlay(String url)
      // (可选) 设置渲染模型类型，默认值为`RenderModel.AUTO`
      .setRenderModel(RenderModel.PLANE_STITCH)
      // (可选) 设置屏幕的纵横比，默认为全屏显示
      //如果渲染模式类型为“RenderModel.PLANE_STITCH”，建议的设置比率为2:1
      .setScreenRatio(int ratioX, int ratioY)
      // (可选) 消除图像拼接的色差，默认为false
      .setImageFusion(boolean imageFusion)
      // (可选) 设置保护镜类型，默认为 OffsetType.ORIGINAL
      .setOffsetType(OffsetType offsetType)
      // (可选) 设置色彩增强开关，默认为false
      .setColorPlusEnable(boolean enabled);
      // (可选) 设置色彩增强强度，默认为1，取值范围 0-1
      .setColorPlusFilterIntensity(float colorPlusFilterIntensity);
      // (可选) 是否允许手势操作，默认为true
      .setGestureEnabled(boolean enabled);
      // (可选) 缩略图缓存文件夹, 默认为 getCacheDir() + "/work_thumbnail"
      .setCacheWorkThumbnailRootPath(String path)
      // (可选) 防抖数据缓存文件夹, 默认为 getCacheDir() + "/stabilizer"
      .setStabilizerCacheRootPath(String path)
      // (可选) 缓存文件夹, 默认为 getExternalCacheDir() + "/template_blender"
      .setCacheTemplateBlenderRootPath(String path)
      // (可选) 星空模式图片缓存文件夹, 默认为 getCacheDir() + "/cut_scene"
      .setCacheCutSceneRootPath(String path)；
```

### 开始播放

播放前需要设置*`WorkWrapper`*和播放参数；

```Java
// 设置 WorkWrapper 和 播放参数
mImagePlayerView.prepare(workWrapper, new ImageParamsBuilder());
// 开始播放
mImagePlayerView.play();
```

### 释放资源

当Activity被销毁时，释放*`InstaImagePlayerView`*。

```Java
@Override
protected void onDestroy() {
    super.onDestroy();
    mImagePlayerView.destroy();
}
```

### 切换渲染模式

> 注意：若您想在平铺模式和其他模式之间切换，必须先重启播放器；

如果您使用的渲染模式为`RENDER_MODE_AUTO`，则可以在以下模式之间切换。

```java
// 切换至普通模式
mImagePlayerView.switchNormalMode();

// 切换至鱼眼模式
mImagePlayerView.switchFisheyeMode();

// 切换至透视模式
mImagePlayerView.switchPerspectiveMode();

// 切换至平铺模式，则需要重启播放器
ImageParamsBuilder builder = new ImageParamsBuilder()；
builder.setRenderModelType(ImageParamsBuilder.RENDER_MODE_PLANE_STITCH);
builder.setScreenRatio(2, 1);
mImagePlayerView.prepare(mWorkWrapper, builder);
mImagePlayerView.play();
```

### 视角视距

您可以根据自己的喜好自定义视角和视距。

> Fov：视角范围 0~180
>
> Distance：视距范围  0~∞

```java
mImagePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### 设置监听

您可以设置*`PlayerViewListener`*来观察播放器状态的变化。

```Java
mImagePlayerView.setPlayerViewListener(new PlayerViewListener() {
    @Override
    public void onLoadingStatusChanged(boolean isLoading) {
        // 播放器加载状态改变
        //isLoading：是否加载中
    }

    @Override
    public void onLoadingFinish() {
        // 播放器加载完成
    }

    @Override
    public void onFail(int errorCode, String errorMsg) {
        // if GPU not support, errorCode is -10003 or -10005 or -13020
        // 播放器错误
    }
    
    @Override
    public void onFirstFrameRender() {
        // 首帧有效渲染
    }
});
```

### 手势操作

您可以将*`PlayerGestureListener`*设置为观察手势操作。

```typescript
mImagePlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // 用户手指按下事件
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // 用户手指单击事件，类似于View.OnClickListener.OnClick(v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // 用户手指抬起事件
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // 用户长按事件
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // 用户双指缩放事件
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // 缩放达到最大或最小是的回弹动画
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // 缩放回弹动画结束
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // 用户手指滑动事件
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // 快速滑动动画
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // 快速滑动动画结束
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## 视频播放器

### 播放器初始化

将*`InstaVideoPlayerView `*放入xml文件中，并且绑定Activity的生命周期。

```XML
<com.arashivision.sdkmedia.player.video.InstaVideoPlayerView
   android:id="@+id/player_video"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
/>
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mVideoPlayerView.setLifecycle(getLifecycle());
}
```

### 设置播放参数

```java
VideoParamsBuilder builder = new VideoParamsBuilder()
      // (可选) 加载图标，默认为空
      .setLoadingImageResId(int resId)
      // (可选) 加载时的背景颜色，默认为黑色
      .setLoadingBackgroundColor(int color)
      // (可选) 是否自动播放
      .setAutoPlayAfterPrepared(boolean autoPlayAfterPrepared)
      // (可选) 设置防抖类型，默认为InstaStabType.STAB_TYPE_OFF
      .setStabType(@InstaStabType.StabType int stabType)
      // (可选) 是否循环播放，默认为true
      .setIsLooping(boolean isLooping)
      // (可选) 设置是否副码流播放
      .setLrvEnable(boolean isLrvEnable)
      // (可选) 是否启用动态拼接，默认为true
      .setDynamicStitch(boolean dynamicStitch)
      // (可选) 消除图像拼接的色差，默认为false
      .setImageFusion(boolean imageFusion)
      // (可选) 设置保护镜类型，默认为 OffsetType.ORIGINAL
      .setOffsetType(OffsetType offsetType)
      // (可选) 去紫边，默认值为false，需要添加算法文件
      .setDePurpleFilterOn(boolean isDePurpleFilterOn)
      // (可选) 设置色彩增强开关，默认为false
      .setColorPlusEnable(boolean enabled);
      // (可选) 设置色彩增强强度，默认为1，取值范围 0-1
      .setColorPlusFilterIntensity(float colorPlusFilterIntensity);
      // (可选) 设置渲染模型类型，默认值为`RenderModel.AUTO`
      .setRenderModel(RenderModel.PLANE_STITCH)
      // (可选) 设置屏幕的宽高比，默认为全屏显示。
      // 如果渲染模式类型为“RenderModel.PLANE_STITCH”，建议设置比为2:1
      .setScreenRatio(int ratioX, int ratioY)
      // (可选) 是否允许手势操作，默认为true
      .setGestureEnabled(boolean enabled)
      // (可选) 缩略图缓存文件夹，默认为 getCacheDir() + "/work_thumbnail"
      .setCacheWorkThumbnailRootPath(String path)
      // (可选) 星空模式图片缓存文件夹，默认为 getCacheDir() + "/cut_scene"
      .setCacheCutSceneRootPath(String path);
```

### 开始播放

播放前需要设置*`WorkWrapper`*和播放参数；

```java
mVideoPlayerView.prepare(workWrapper, new VideoParamsBuilder());
mVideoPlayerView.play();
```

### 释放资源

当Activity被销毁时，释放*`InstaVideoPlayerView`* 。

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    mVideoPlayerView.destroy();
}
```

### 播放控制

您可以通过调用*`VideoPlayerView`*中的代码，来控制视频的播放进度，暂停/恢复，播放音量等。

| 方法                    | 参数            | 返回值  | 说明                 |
| ----------------------- | --------------- | ------- | -------------------- |
| isPlaying               | -               | boolean | 视频是否正在播放     |
| isLooping               | -               | boolean | 是否循环播放         |
| setLooping              | boolean         | void    | 设置是否循环播放     |
| setVolume               | float (范围0-1) | void    | 设置播放音量         |
| pause                   | -               | void    | 暂停播放             |
| resume                  | -               | void    | 恢复播放             |
| seekTo                  | long            | void    | 快进到指定进度播放   |
| isSeeking               | -               | boolean | 是否快进中           |
| getVideoCurrentPosition | -               | long    | 获取视频当前播放进度 |
| getVideoTotalDuration   | -               | long    | 获取视频总长         |

### 切换渲染模式

> 注意：若您想在平铺模式和其他模式之间切换，必须先重启播放器；

如果您使用的渲染模式为`RENDER_MODE_AUTO`，则可以在以下模式之间切换。

```java
// 切换至普通模式
mVideoPlayerView.switchNormalMode();

// 切换至鱼眼模式
mVideoPlayerView.switchFisheyeMode();

// 切换至透视模式
mVideoPlayerView.switchPerspectiveMode();

// 切换至平铺模式，则需要重启播放器
VideoParamsBuilder builder = new VideoParamsBuilder()；
builder.setRenderModelType(ImageParamsBuilder.RENDER_MODE_PLANE_STITCH);
builder.setScreenRatio(2, 1);
mImagePlayerView.prepare(mWorkWrapper, builder);
mImagePlayerView.play();
```

### 视角视距

您可以根据自己的喜好自定义视角和视距。

> Fov：视角范围 0~180
>
> Distance：视距范围  0~∞

```java
mVideoPlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### 设置监听

您可以设置*`PlayerViewListener`*来观察播放器状态的变化。

```Java
mVideoPlayerView.setPlayerViewListener(new PlayerViewListener() {
    @Override
    public void onLoadingStatusChanged(boolean isLoading) {
        // 播放器加载状态改变
        //isLoading：是否加载中
    }

    @Overrid
    epublic void onLoadingFinish() {
        // 播放器加载完成
    }

    @Override
    public void onFail(int errorCode, String errorMsg) {
        // if GPU not support, errorCode is -10003 or -10005 or -13020
        // 播放器错误
    }
    
    @Override
    public void onFirstFrameRender() {
        // 首帧有效渲染
    }
});
```

您可以设置*`VideoStatusListener`*来观察视频状态的变化。

```java
mVideoPlayerView.setVideoStatusListener(new VideoStatusListener() {
    @Override
    public void onProgressChanged(long position, long length) {
        // 播放进度回调
        // position：已播放长度
        // length：视频总长
    }

    @Override
    public void onPlayStateChanged(boolean isPlaying) {
        // 视频播放状态回调
        // isPlaying：是否正在播放
    }

    @Override
    public void onSeekComplete() {
        // 快进完成
    }

    @Override
    public void onCompletion() {
        // 视频播放完成
    }
});
```

### 手势操作

您可以将*`PlayerGestureListener`*设置为观察手势操作。

```typescript
mVideoPlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // 用户手指按下事件
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // 用户手指单击事件，类似于View.OnClickListener.OnClick(v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // 用户手指抬起事件
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // 用户长按事件
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // 用户双指缩放事件
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // 缩放达到最大或最小是的回弹动画
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // 缩放回弹动画结束
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // 用户手指滑动事件
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // 快速滑动动画
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // 快速滑动动画结束
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## 导出图片

### 导出图片参数

如果你想导出图片，你需要先了解图片导出参数*`ExportImageParamsBuilder`*。

```java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
    // (必须) 导出文件路径
    .setTargetPath(String path)
    // (可选) 设置导出图像的宽度，默认值为从WorkWapper获取。
    .setWidth(int width)
    // (可选) 设置导出图像的高度，默认值为从WorkWapper获取。
    .setHeight(int height)
    // (可选) 设置导出模式, default is `PANORAMA`
    // ExportMode.PANORAMA: 导出全景图片
    // ExportMode.SPHERE: 导出平面缩略图
    .setExportMode(ExportUtils.ExportMode mode)
    // (可选)是否启用动态拼接，默认为true。
    .setDynamicStitch(boolean dynamicStitch)
    // (可选)设置防抖类型，默认为InstaStabType.STAB_TYPE_OFF。
    .setStabType(@InstaStabType.StabType int stabType)
    // (可选) 消除图像拼接的色差，默认为false
    .setImageFusion(boolean imageFusion)
    // (可选) 去紫边，默认为false，需要添加算法文件
    .setDePurpleFilterOn(boolean isDePurpleFilterOn)
    // (可选) 设置保护镜类型，默认为 OffsetType.ORIGINAL
    .setOffsetType(OffsetType offsetType)
    // (可选) 设置如HDR.jpg通过拼接生成，默认为null
    .setUrlForExport(String url)
    // (可选) 设置使用软件解码器，默认为false
    .setUseSoftwareDecoder(boolean useSoftwareDecoder)
    // (可选) 设置相机角度。建议在导出缩略图时使用。
    //当前显示的角度参数可以从`PlayerView.getXXX（）`中获得。
    .setFov(float fov)
    .setDistance(float distance)
    .setYaw(float yaw)
    .setPitch(float pitch)
    //（可选）缓存文件夹，默认为 getCacheDir() + "/work_thumbnail"
    .setCacheWorkThumbnailRootPath(String path)
    //（可选）缓存文件夹，默认为 getCacheDir() + "/stabilizer"
    .setStabilizerCacheRootPath(String path)
    //（可选）缓存文件夹，默认为 getCacheDir() + "/cut_scene"
    .setCacheCutSceneRootPath(String path);
```

### 导出全景图片

使用图片导出参数：

```Java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024);
int exportId = ExportUtils.exportImage(WorkWrapper, builder, new IExportCallback() {
            @Override
            public void onSuccess() {       
                // 导出成功        
            }

            @Override
            public void onFail() {
                // 导出失败
            }

            @Override
            public void onCancel() {
                // 取消导出
            }

            @Override
            public void onProgress(float progress) {
                // 导出进度
            }
        });
```

### 导出图片缩略图

使用图片导出参数：

```Java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.SPHERE)
        .setTargetPath(path)
        .setWidth(512)
        .setHeight(512)
        .setFov(mImagePlayerView.getFov())
        .setDistance(mImagePlayerView.getDistance())
        .setYaw(mImagePlayerView.getYaw())
        .setPitch(mImagePlayerView.getPitch());
int exportId = ExportUtils.exportImage(WorkWrapper, builder, new IExportCallback() {
           @Override
            public void onSuccess() {       
                // 导出成功        
            }

            @Override
            public void onFail() {
                // 导出失败
            }

            @Override
            public void onCancel() {
                // 取消导出
            }

            @Override
            public void onProgress(float progress) {
                // 导出进度
            }
        });
```

## 导出视频

### 导出视频参数

导出视频，需要先了解视频导出参数*`ExportVideoParamsBuilder`* 。

```java
ExportVideoParamsBuilder builder = new ExportVideoParamsBuilder()
    // (必须) 导出文件路径
    .setTargetPath(String path)
    // (可选) 设置导出视频的宽度。它必须是2的幂，默认值是从WorkWapper获取。
    .setWidth(int width)
    // (可选)设置导出视频的高度。它必须是2的幂，默认值是从WorkWapper获取。
    .setHeight(int height)
    // (可选) 设置导出视频的比特率，默认值为从WorkWapper获取。
    .setBitrate(int bitrate)
    // (可选) 设置导出视频的fps，默认值为从WorkWapper获取。
    .setFps(int fps)
    // (可选) 设置导出模式, default is `PANORAMA`
    // ExportMode.PANORAMA: 导出全景图片
    // ExportMode.SPHERE: 导出平面缩略图
    .setExportMode(ExportUtils.ExportMode mode)
    // (可选) 去紫边，默认为false，需要添加算法文件
    .setDePurpleFilterOn(boolean isDePurpleFilterOn)
    // (可选) 是否启用降噪 (导出图片默认启用降噪，降噪开关仅导出视频支持)
    // 需要提前调用ExportUtils.supportDenoise(WorkWrapper)方法判断是否支持降噪
    // 支持降噪则设置有效，反之设置无效
    .setDenoise(true)
    // (可选) 消除图像拼接的色差，默认为false
    .setImageFusion(boolean imageFusion)
    // (可选) 设置保护镜类型，默认为 OffsetType.ORIGINAL
    .setOffsetType(OffsetType offsetType)
    //（可选）是否启用动态拼接，默认为true。
    .setDynamicStitch(boolean dynamicStitch)
    // (可选)设置防抖类型，默认为InstaStabType.STAB_TYPE_OFF。
    .setStabType(@InstaStabType.StabType int stabType)
    // (可选) 设置使用软件解码器，默认为false
    .setUseSoftwareDecoder(boolean useSoftwareDecoder)
    // (可选) 缓存文件夹，默认为 getCacheDir() + "/work_thumbnail"
    .setCacheWorkThumbnailRootPath(String path)
    // (可选) 缓存文件夹，默认为 getCacheDir() + "/cut_scene"
    .setCacheCutSceneRootPath(String path)
    // (可选) 设置视角场
    .setFov(float fov)
    // (可选) 设置视距
    .setDistance(float distance)
    //（可选） 设置欧拉角-俯仰角
    .setPitch(float pitch)
    //（可选） 设置欧拉角-偏航角
    .setYaw(float yaw)
    //（可选） 设置欧拉角-滚转角
    .setRoll(float roll)；
```

### 导出全景视频

使用视频导出参数；

```Java
ExportVideoParamsBuilder builder = new ExportVideoParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024)
        .setBitrate(20 * 1024 * 1024)
        .setFps(30);
int exportId = ExportUtils.exportVideo(WorkWrapper, builder, new IExportCallback() {
            @Override
            public void onSuccess() {       
                // 导出成功        
            }

            @Override
            public void onFail() {
                // 导出失败
            }

            @Override
            public void onCancel() {
                // 取消导出
            }

            @Override
            public void onProgress(float progress) {
                // 导出进度
            }
        });
```

### 导出视频缩略图

导出的结果是图片，因此使用图片导出参数：

```Java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.SPHERE)
        .setTargetPath(path)
        .setWidth(512)
        .setHeight(512)
        .setFov(mVideoPlayerView.getFov())
        .setDistance(mVideoPlayerView.getDistance())
        .setYaw(mVideoPlayerView.getYaw())
        .setPitch(mVideoPlayerView.getPitch());
int exportId = ExportUtils.exportVideoToImage(WorkWrapper, builder, new IExportCallback() {
           @Override
            public void onSuccess() {       
                // 导出成功        
            }

            @Override
            public void onFail() {
                // 导出失败
            }

            @Override
            public void onCancel() {
                // 取消导出
            }

            @Override
            public void onProgress(float progress) {
                // 导出进度
            }
        });
```

### 停止导出

```Java
ExportUtils.stopExport(exportId);
```

## 其它

### 生成HDR图片

如果您有HDR图像的*`WorkWrapper`*，则可以通过HDR拼接将其生成为一个图像文件。

> 注意：这是一个耗时的操作，需要在子线程中处理。

```Java
boolean isSuccessful = StitchUtils.generateHDR(WorkWrapper workWrapper, String hdrOutputPath);
```

*`generateHDR`*调用成功后，outputPath可以作为代理播放或设置为导出url。

```Java
// 设置为播放代理
ImageParamsBuilder builder = new ImageParamsBuilder()
        // 如果HDR拼接成功，则将其设置为播放代理
        .setUrlForPlay(isSuccessful ? hdrOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();

// 设置为导出url
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        // 如果HDR拼接成功，则将其设置为导出url
        .setUrlForExport(hdrOutputPath);
```

### 生成PureShot图像

如果您有PureShot图片的*`WorkWrapper`*，则可以通过PureShot拼接将其生成为一个图像文件。

> 注意：这是一个耗时的操作，需要在子线程中处理。

```Java
boolean isSuccessful = StitchUtils.generatePureShot(WorkWrapper workWrapper, String pureshotOutputPath, String algoFolderPath);
```

> 注意：生成PureShot图像需要算法模型文件，请向我们的技术支持人员索取与sdk版本对应的算法模型文件。

algoFolderPath是算法模型文件的目录，可以是LocalStoragePath或AssetsRelativePath，SDK会在需要的时候加载这些文件。

*`generatePureShot`*调用成功后，outputPath可以作为代理播放或设置为导出url。

```Java
// 设置为播放代理
ImageParamsBuilder builder = new ImageParamsBuilder()
        // 如果PureShot拼接成功，则将其设置为播放代理
        .setUrlForPlay(isSuccessful ? pureshotOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();

// 设置为导出url
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        // 如果PureShot拼接成功，则将其设置为导出url
        .setUrlForExport(pureshotOutputPath);
```

## 算法文件

部分功能需要依赖算法文件。算法文件在demo中的assets文件夹中获取，文件路径和demo中保持一致即可。 下面是功能和其所需要的算法对应表：

| 功能         | 模型文件                                                     | 路径说明                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 去紫边       | defringe_lr_dynamic_e28c1212.ins defringe_hr_dynamic_7b56e80f.ins | 固定路径 file:///android_assets/insta360/model/              |
| 色彩增强     | lut_7a79b17a_32.ins                                          | 固定路径 file:///android_assets/insta360/model/              |
| PureShot合成 | Pmdenoise_raw_plus.insraw_denoising_abeda57d.ins283fish.ilp283wide.ilp577.ilp586.ilpX3.ilp | 开发者自定义路径由StitchUtils.*generatePureShot函数传递进来* |

## 错误码

### 图片/视频导出错误码

| 错误码 | 解释                                             |
| ------ | ------------------------------------------------ |
| -31000 | 正在导出中                                       |
| -31010 | 停止导出                                         |
| -31011 | 加载防抖数据失败                                 |
| -31012 | *EXPORT_TERMINATED_BY_WORK_REFERENCE_RELEASABLE* |
| -31016 | *EXPORT_POST_EXECUTE_FAILED*                     |
| -31017 | 原始缩略图LRV找不到                              |
| -31018 | 原始缩略图LRV加载失败                            |
| -31019 | *EXPORT_VIDEO_TO_IMAGE_SEQUENCE_CANCEL*          |
| -13016 | 导出超时                                         |
| -13017 | 取消导出                                         |
| -13018 | *SHADOW_EXPORT_INIT_SHADOW_CLONE_FAIL*           |
| -13019 | *SHADOW_EXPORT_PREPARE_EXPORT_INFO_FAIL*         |
| -13021 | 打开硬件编码失败                                 |
| -13022 | *SHADOW_EXPORT_FAIL_TARGET_TIME_NULL*            |
| -13023 | *SHADOW_EXPORT_PREPRARE_FRAME_EXPORTER_FAIL*     |
| -13024 | *SHADOW_EXPORT_INPUT_TARGET_FAIL*                |
| -13025 | *STOP_MOTION_SELECT_POSE_CLIP_NULL*              |
| -13026 | *STOP_MOTION_SELECT_CLIP_NOT_MATCH*              |
| -13027 | *STOP_MOTION_SELECT_LOAD_STABILIZER_FAIL*        |
| -13028 | *STOP_MOTION_SELECT_KEY_NULL*                    |
| -13029 | *SHADOW_EXPORT_TRACK_TARGET_EMPTY*               |
| -13031 | 帧导出器准备失败                                 |
| -13033 | 反向解析视频失败                                 |
| -13034 | *双流帧格式不同*                                 |
| -13035 | 缩略图防抖错误                                   |
| -13036 | 视频帧率等于0                                    |
| -13038 | 视频宽度等于0                                    |
| -13039 | 视频高度等于0                                    |
| -13040 | 降噪模型等于null                                 |
| -13041 | 视频源帧率等于0                                  |
| -13042 | 视频源时长等于0                                  |
| -13043 | 图片降噪文件移动错误                             |
| -13034 | 双流帧格式不同                                   |
| -13044 | 加载AAA参数失败                                  |
| -13045 | 创建降噪数据失败                                 |
| -13046 | 图片宽度等于0                                    |
| -13047 | 图片高度等于0                                    |

# 常见问题：

## Q1：连接相机时，无法访问互联网怎么办？

> 一般这种情况指的是通过Wi-Fi与相机建立连接的情况。App与相机的通信方式分为两种，一种是socket，另一种是http。下面是是两种通讯方式对应的功能：
>
> - Socket：预览流，读取相机参数，设置相机参数，读取拍摄参数，设置拍摄参数等。// TODO 支持的功能待完善
> - Http  [初始化支持列表](https://arashivision.feishu.cn/wiki/Bc4VwYQBHiCu82kCooMcNBVun4g#share-JgkudEvdgoO0lhxsUGWcSqrGn2b)，下载相机文件。// TODO 支持的功能待完善
>
> 出现这个问题的根本原因是因为在与相机建立socket连接的时候，调用了`ConnectivityManager#bindProcessToNetwork(network)` 方法，把当前进程和相机WI-FI网络进行绑定。
>
> 因此在Wi-Fi连接成功之后，调用`ConnectivityManager#bindProcessToNetwork(4GNetWork)` 方法绑定可用的网络（如4G网络），即可解决无法访问互联网的问题。此时已经连接Socket连接，解绑之后，依然可以使用Socket支持的功能。
>
> 但这样做会有一个问题。因为当前进程与相机Wi-Fi之间未绑定，在使用依赖http通信的功能时，会出现如下报错：
>
> ![Uploading image.png…]()
>
> 解决这个问题有2种方案：
>
> 1. 在使用Http通信方式相关的功能时，调用`bindProcessToNetwork(camera)`方法绑定至相机Wi-Fi。在相关功能结束时，调用`bindProcessToNetwork(4g)`切换回可用的4G网络。
>    1. 优点：实现简单
>    2. 缺点：会短暂的断开与互联网的连接
> 2. 开启一个新的进程，命名为download。调用`bindProcessToNetwork(camera)`方法将download进程与相机Wi-Fi网络进行绑定。所有与Http通讯相关的功能放在download进程中进行。
>    1. 优点：完美解决无法访问互联网的问题
>    2. 缺点：实现复杂

## Q2：VideoData对象中的数据格式是什么？

> H.264格式，包含pps和sps两帧数据。

## Q3：c++_shared冲突怎么办？

> 升级版本到1.8.1_build_05及以上

## Q4：关于okGo的问题如何处理？

> *implementation*("com.github.jeasonlzy.okhttp-OkGo:okgo:v3.0.4")
>
> *implementation*("com.arashivision.sdk:sdkcamera:1.8.0_build_11")**{**
>
> ​    *exclude*("com.github.jeasonlzy","okhttp-OkGo")
>
> **}**

## Q5：连接时报错-2110203的问题？

> socket读失败，一般是Wi-Fi被系统切掉，去系统设置页手动连接Wi-Fi，可降低被系统自动切掉的概率。

## Q6：关于华为部分手机播放黑屏的问题？

> 照片宽度16386，超过了华为部分手机的的渲染尺寸8192。
