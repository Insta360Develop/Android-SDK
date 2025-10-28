
* [Camera SDK Function](#camera-sdk-function)
    * [Environmental Preparation](#environmental-preparation)
    * [Connection Module](#connection-module)
    * [Wi-Fi Management Module](#wi-fi-management-module)
    * [Information Acquisition Module](#information-acquisition-module)
    * [Notification Module](#notification-module)
    * [Camera Lens Control Module](#camera-lens-control-module)
    * [Preview Module](#preview-module)
    * [Capture Control Module](#capture-control-module)
    * [LIVE](#live)
    * [File Management Module](#file-management-module)
    * [Logging Module](#logging-module)
    * [GPS Data Module](#gps-data-module)
    * [Other Features Module](#other-features-module)
    * [Error Code](#error-code)
* [Media SDK Function](#media-sdk-function)
    * [Environmental Preparation](#environmental-preparation-1)
    * [Preview Module](#preview-module-1)
    * [WorkWrapper](#workwrapper)
    * [Image Player](#image-player)
    * [Video Player](#video-player)
    * [Export](#export)
    * [Export Video](#export-video)
    * [Others](#others)
    * [Algorithm File](#algorithm-file)
    * [Error Code](#error-code-1)

# Camera SDK Function

## Environmental Preparation

Add the Maven repository to the build file (the `build.gradle` file in the project root):

```Groovy
allprojects {
    repositories {
        ...
        maven {
            url 'x.x.x'             // see sdk demo
            credentials {
                // see sdk demo
                username = '***'
                password = '***'
            } 
        }
    }
}
```

- Import the dependency library in the `build.gradle` file located in the Demo directory:

```Groovy
dependencies {
    implementation 'com.arashivision.sdk:sdkcamera:x.x.x' // see sdk demo
}
```

- Initialize the SDK in the application

```java
public class MyApp extends Application {

    @Overridepublic void onCreate() {
        super.onCreate();      
        InstaCameraSDK.init(this);
    }
}
```

## Connection Module

### Determine if the Camera is Connected

You can determine if the camera is connected by obtaining the current camera connection type.

```Java
private boolean isCameraConnected() {
    int type = InstaCameraManager.getInstance().getCameraConnectedType();
    return type != InstaCameraManager.CONNECT_TYPE_NONE;
}
```

The connection types have the following four outcomes:

1. Not connected：*`InstaCameraManager.CONNECT_TYPE_NONE`*
2. USB connection：*`InstaCameraManager.CONNECT_TYPE_USB`*
3. Wi-Fi connection：*`InstaCameraManager.CONNECT_TYPE_WIFI`*
4. Bluetooth connection：*`InstaCameraManager.CONNECT_TYPE_BLE`*

### Connecting the Camera

You can connect to the camera via Bluetooth, Wi-Fi, or USB

> Note: You must perform the connection operation on the main thread.

> Note: When the camera is connected, if the app switches to the background, it will disconnect due to the characteristics of the Android system. Therefore, it is necessary to enable a foreground service to prevent disconnection from the camera. This foreground server is implemented by the developer themselves. The relevant code can refer to the SDK Demo.

#### Bluetooth Connection

> Note: Bluetooth connection requires successful authorization of Bluetooth-related permissions.

> Note: In the case of a Bluetooth connection, large data transmission is not supported. Therefore, functions that transmit large amounts of data are not supported.

For example: live streaming, preview, support list, firmware upgrade, playback, export, etc.

1. Initialize Bluetooth Scanning

```Java
InstaCameraManager.getInstance().startBleScan();
```

2. Set the Bluetooth Scanning Listener

```Java
InstaCameraManager.getInstance().setScanBleListener(new IScanBleListener() {
    @Override
    public void onScanStartSuccess() {
        //Bluetooth is about to start scanning
    }

    @Override
    public void onScanStartFail() {
        //Bluetooth scanning failed
    }

    @Override
    public void onScanning(BleDevice bleDevice) {
        //A new Bluetooth device has been scanned; you can update the Bluetooth device list here
    }

    @Override
    public void onScanFinish(List<BleDevice> bleDeviceList) {
        //Bluetooth scanning finished
    }
});
```

3. Stopping Bluetooth Scanning

```Java
InstaCameraManager.getInstance().stopBleScan();
```

4. Checking if Bluetooth is Scanning

```Java
InstaCameraManager.getInstance().isBleScanning();
```

5. Establishing a Bluetooth Connection with the Camera

```Java
InstaCameraManager.getInstance().connectBle(bleDevice);
```

#### Wi-Fi Connection

> Note: Ensure that the phone is already connected to the camera’s Wi-Fi.The Wi-Fi password of the camera can be viewed in the settings menu of the camera, or by connecting via Bluetooth and calling to read the Wi-Fi information (SSID and password) of the camera.

```Java
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_WIFI);
```

#### USB Connection

> Note: Ensure that the phone and camera have established a USB connection, and the camera must be switched to Android mode.

```Java
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_USB);
```

### Disconnecting the Camera

When you want to disconnect the camera, you can call the following code.

```Java
InstaCameraManager.getInstance().closeCamera();
```

## Wi-Fi Management Module

### Camera Wi-Fi On and Off

When connected to Bluetooth, the camera's Wi-Fi can be turned on and off using the following code.

```Java
ICameraOperateCallback callback = new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // The command was executed successfully
    }

    @Override
    public void onFailed() {
        // Command execution failed
    }

    @Override
    public void onCameraConnectError() {
        // The camera is disconnected or not connected
    }
};
// Turn on the camera's wifi and use the camera's default channel
InstaCameraManager.getInstance().openCameraWifi(callback);
// Open the camera wifi on the specified channel
InstaCameraManager.getInstance().openCameraWifi(int countryChannel ,callback);

// Turn off camera wifi
InstaCameraManager.getInstance().closeCameraWifi(callback)

// Restart the camera Wi-Fi, including two operations: turn off and turn on, use the camera's default channel to turn on
InstaCameraManager.getInstance().resetCameraWifi(callback)
// Restart the camera Wi-Fi by specifying a channel
InstaCameraManager.getInstance().resetCameraWifi(int countryChannel callback)
```

### Get Wi-Fi Information

```Java
// Get camera Wi-Fi information
WifiInfo wifiInfo = InstaCameraManager.getInstance().getWifiInfo();

// Wi-Fi name
String ssid = wifiInfo.getSsid();

// Wi-Fi password
String ssid = wifiInfo.getPwd();

// Password version
int pwdVersion= wifiInfo.getPwdVersion();

// Wi-Fi channel
int channel= wifiInfo.getChannel();

// mac address
String address = wifiInfo.getMacAddress();

// Get the Wi-Fi country code
Strin country = InstaCameraManager.getInstance().getWifiCountry();
```

### Switch Wi-Fi Channel

```Java
// Get current country code and Wi-Fi channel
InstaCameraManager.getInstance().fetchWifiChannel((countryCode, cameraChannel) -> {
    // Get country code and current Wi-Fi channel
});

//  Set Wi-Fi country code
InstaCameraManager.getInstance().setWifiCountry("CN", new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
         // Setup successful
    }

    @Override
    public void onFailed() {
        // Setup failed
    }

    @Override
    public void onCameraConnectError() {
        // Camera connection error
    }
});

// Get Wi-Fi channel list
int[] channelList = InstaCameraManager.getInstance().getWifiChannelList();
```

## Information Acquisition Module

### Synchronizing Camera Parameters

Synchronizing camera parameters means reading all the internal camera parameters at once and caching them in the application. Call the following code to synchronize all camera parameters (Synchronization of all parameters and  status).

```Java
InstaCameraManager.getInstance().fetchCameraOptions(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // Camera parameters synchronized successfully
    }

    @Override
    public void onFailed() {
        // Camera parameter synchronization failed
    }

    @Override
    public void onCameraConnectError() {
        // Camera connection failed, which may indicate a camera disconnection event
    }
});
```

### Get the Camera Support List

For X4 and later cameras, the capturing mode and capturing properties of the camera are based on the properties configured in the support list. Different cameras support different capturing modes, and there are also differences in the capturing properties supported under different capturing modes. Before capturing, synchronize the camera's supported list (Json file).

#### Initialize Support List

When entering the prep page or switching lenses, the support list needs to be initialized. This can be achieved by calling the following code:

```Java
InstaCameraManager.getInstance().initCameraSupportConfig(success -> {
    // success: Whether initialization was successful
});
```

#### Capturing Mode Support

Each lens of the camera supports different capturing modes. You need to use the following code to obtain its supported capturing modes.

```Java
// Get all supported capturing modes
List<CaptureMode> support = InstaCameraManager.getInstance().getSupportCaptureMode();
```

#### Capturing Attribute Support

Each capturing mode supports different capturing attributes. You need to use the following code to obtain the supported capturing attributes.

```Java
// Pass in the specified capturing mode as parameters
List<CaptureSetting> support = InstaCameraManager.getInstance().getSupportCaptureSettingList(CaptureMode.CAPTURE_NORMAL);
```

#### The Range of Values for Capturing Attributes

The value range of the capturing attribute is also configured in the support list. You need to use the following code to obtain the value range of the supported capturing attribute.

```Java
// Gets the property value range of CaptureSetting.RECORD_DURATION, returns a List < RecordDuration >
InstaCameraManager.getInstance().getSupportRecordDurationList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.RECORD_RESOLUTION and return a List < RecordResolution >
InstaCameraManager.getInstance().getSupportRecordResolutionList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.PHOTO_RESOLUTION and return a List < PhotoResolution >
InstaCameraManager.getInstance().getSupportPhotoResolutionList(CaptureMode.CAPTURE_NORMAL);

// Get the range of property values of CaptureSetting. INTERVAL and return a List < Interval > list
InstaCameraManager.getInstance().getSupportIntervalList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.HDR_STATUS and return a List < HdrStatus >
InstaCameraManager.getInstance().getSupportHdrStatusList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.PHOTO_HDR_TYPE and return a List < PhotoHdrType >
InstaCameraManager.getInstance().getSupportPhotoHdrTypeList(CaptureMode.CAPTURE_NORMAL);

// Get the value range of the property value of CaptureSetting.GAMMA_MODE and return a List < GammaMode >
InstaCameraManager.getInstance().getSupportGammaModeList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.RAW_TYPE and return a List < RawType >
InstaCameraManager.getInstance().getSupportRawTypeList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.PANO_EXPOSURE_MODE and return a List < PanoExposureMode >
InstaCameraManager.getInstance().getSupportPanoExposureList(CaptureMode.TIMELAPSE);

// Get the property value range of CaptureSetting. EXPOSURE and return a List < Exposure >
InstaCameraManager.getInstance().getSupportExposureList(CaptureMode.TIMELAPSE);

// Get the value range of the CaptureSetting. ISO property and return a List < ISO >
InstaCameraManager.getInstance().getSupportISOList(CaptureMode.TIMELAPSE);

// Gets the value range of the CaptureSetting.ISO_TOP_LIMIT property and returns a List < ISOTopLimit >
InstaCameraManager.getInstance().getSupportISOTopLimitList(CaptureMode.TIMELAPSE);

// Gets the value range of the CaptureSetting.SHUTTER_MODE property and returns a List < ShutterMode >
InstaCameraManager.getInstance().getSupportShutterModeList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting. SHUTTER and return a List < Shutter >
InstaCameraManager.getInstance().getSupportShutterList(CaptureMode.TIMELAPSE);

// Get the value range of the attribute value of CaptureSetting. EV and return a List < Ev >
InstaCameraManager.getInstance().getSupportEVList(CaptureMode.CAPTURE_NORMAL);

// Get the value range of the property value of CaptureSetting.EV_INTERVAL and return a List < EVInterval >
InstaCameraManager.getInstance().getSupportEVIntervalList(CaptureMode.TIMELAPSE);

// Get the value range of the attribute value of CaptureSetting. AEB and return a List < AEB >
InstaCameraManager.getInstance().getSupportAEBList(CaptureMode.TIMELAPSE);

// Get the value range of the CaptureSetting. WB attribute and return a List < WB >
InstaCameraManager.getInstance().getSupportWBList(CaptureMode.TIMELAPSE);

// Gets the property value range of CaptureSetting.INTERNAL_SPLICING and returns a List < InternalSplicing >
InstaCameraManager.getInstance().getSupportInternalSplicingList(CaptureMode.TIMELAPSE);

// Gets the value range of the CaptureSetting.DARK_EIS_ENABLE property and returns a List < DarkEisType >
InstaCameraManager.getInstance().getSupportDarkEisList(CaptureMode.TIMELAPSE);

// Gets the property value range of CaptureSetting.BURST_CAPTURE, returns a List < BurstCapture >
InstaCameraManager.getInstance().getSupportBurstCaptureList(CaptureMode.TIMELAPSE);

// Get the property value range of CaptureSetting.I_LOG and return a List<ILogStatus> list
InstaCameraManager.getInstance().getSupportILogStatusList(CaptureMode.TIMELAPSE);
```

### Camera Information Acquisition

#### Camera Type

```Java
// Get the camera type, such as: Insta360 X4
String cameraType = InstaCameraManager.getInstance().getCameraType();
// Get enumeration
CameraType type = CameraType.getForType(cameraType);
```

#### Camera Serial Number

```Java
InstaCameraManager.getInstance().getCameraSerial();
```

#### Firmware Version

```Java
// Get the camera firmware version, e.g. 0.9.24
String fwVersion = InstaCameraManager.getInstance().getCameraVersion();
```

#### Camera Activation Time

```Java
// Get the camera activation time; the inactive time is 0.
long activeTime = InstaCameraManager.getInstance().getActiveTime();
```

#### SD Card Total Storage Capacity

```Java
// Get the total space size of the SD card
long totalSpace = InstaCameraManager.getInstance().getCameraStorageTotalSpace();
```

#### SD Card Available Storage Capacity

```Java
// Get the free space size of the SD card
long freeSpace = InstaCameraManager.getInstance().getCameraStorageFreeSpace();

// Get the remaining space size of the SD card (for photos, the unit is shots; for videos, the return value is the remaining recording time)
int remaining= InstaCameraManager.getInstance().getRemaining(CaptureMode captureMode);
```

#### Battery

```Java
// Get battery type
// THICK = 0;       //Thick battery
// THIN = 1;        //Thin battery (camera default)
// VERTICAL;        //Vertical shot battery
// NONE;            //Only USB power supply without battery
InstaCameraManager.getInstance().getBatteryType();

// Get current battery: 0-100
int level = InstaCameraManager.getInstance().getCameraCurrentBatteryLevel();
```

#### Camera System Time

```Java
// Get camera system time
long mediaTime = InstaCameraManager.getInstance().getMediaTime();
```

### Get Camera Status

#### Determine if the Camera is Charging

```Java
// Check if the camera is charging
boolean isCharging = InstaCameraManager.getInstance().isCameraCharging();
```

#### Determine if the Camera is Working

```Java
// Check if the camera is working
boolean isWorking = InstaCameraManager.getInstance().isCameraWorking();

//Check if the specified capturing mode is currently working
boolean isNormalCapturing = InstaCameraManager.getInstance().isCameraWorking(CaptureMode.CAPTURE_NORMAL);
```

#### Determine if the SD Card is Usable

```Java
// Check if the SD card is usable
boolean isEnable = InstaCameraManager.getInstance().isSdCardEnabled();
```

## Notification Module

You can register/unregister the`ICameraChangedCallback` on multiple pages to monitor changes in the camera status.

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
     * Camera status change
     *
     * @param enabled: whether the camera is available
     * @param connectType: connection type
     */
    @Override
    public void onCameraStatusChanged(boolean enabled, int connectType) {
    }

    /**
     * Camera coneection failure
     * <p>
     * A common scenario is that another phone or another application on this phone has already established a connection with the camera, leading to a failure in establishing the connection, in which case, other phones must first disconnect from this camera.
     * @param errorCode: Error code, refer to the camera connection error code
     */
    @Override
    public void onCameraConnectError(int errorCode) {
    }

    /**
     * SD card insertion notification
     *
     * @param enabled: whether the current SD card is available
     */
    @Override
    public void onCameraSDCardStateChanged(boolean enabled) {
    }

    /**
     * SD card storage status change
     *
     * @param freeSpace:  the current available capacity
     * @param totalSpace: the total capacity
     */
    @Override
    public void onCameraStorageChanged(long freeSpace, long totalSpace) {
    }

    /**
     * Low battery notification
     */
    @Override
    public void onCameraBatteryLow() {
    }

    /**
     * Camera power change notification
     *
     * @param batteryLevel: current power level（0-100，always returns 100 when charging）
     * @param isCharging:   whether the camera is charging
     */
    @Override
    public void onCameraBatteryUpdate(int batteryLevel, boolean isCharging) {
    }

    /**
     * Camera temperature change notification
     *
     * @param tempLevel: temperature level
     */
    @Override
    public void onCameraTemperatureChanged(TemperatureLevel tempLevel) {
    }
}
```

## Camera Lens Control Module

Camera lenses are divided into the following three types:

1. Panoramic lens: *`SensorMode.PANORAMA`*
2. Front camera: *`SensorMode.FRONT`*
3. Rear lens: *`SensorMode.REAR`*

### Determine the Sensor Mode

```Java
// Get the current sensor mode
SensorMode sensorMode = InstaCameraManager.getInstance().getCurrentSensorMode();

// Determine whether it is single sensor mode
InstaCameraManager.getInstance().isCameraSingleSensorMode();

// Determine whether it is dual sensor mode
InstaCameraManager.getInstance().isCameraDualSensorMode();
```

### Switch Camera Lenses

> Note: The SDK currently only supports panoramic lenses. Unknown issues may occur with non-panoramic lenses. Please switch to using the SDK under panoramic lenses.

```Java
// Switch to camera panorama lens
InstaCameraManager.getInstance().switchPanoramaSensorMode(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // Lens switch successful
    }

    @Override
    public void onFailed() {
        // Lens switch failed
    }

    @Override
    public void onCameraConnectError() {
        // Camera connection error
    }
});
```

## Preview Module

> Note: Must be used with `Media SDK`.

### Open Preview Stream

After successfully connecting the camera, you can manipulate the camera preview stream like this.

> If the camera is passively disconnected, or if `closeCamera()` is called directly during the preview, the SDK will automatically turn off the preview stream processing related state without calling `closePreviewStream()`。

> Note: The following code only enables the preview stream and cannot display the preview content. You need to use the InstaCapturePlayerView control of MediaSDK to display the preview content.

```Java
public class PreviewActivity extends BaseObserveCameraActivity implements IPreviewStatusListener {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_preview);
        // Open preview
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(this);
        InstaCameraManager.getInstance().startPreviewStream(PreviewStreamResolution.STREAM_1440_720_30FPS, InstaCameraManager.PREVIEW_TYPE_NORMAL);
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (isFinishing()) {
            // Close preview
            InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
            InstaCameraManager.getInstance().closePreviewStream();
        }
    }

    @Override
    public void onOpening() {
        // Preview stream is opening      
    }

    @Override
    public void onOpened() {
        // The preview stream has been opened and can be played
    }

    @Override
    public void onIdle() {
        // Preview stream has stopped
    }

    @Override
    public void onError() {
        // Preview failed
    }
        
        @Override
    public void onExposureData(ExposureData exposureData) {
        // Callback frequency 500Hz
        // exposureData.timestamp: Time after the camera is turned on
        // exposureData.exposureTime: Rolling shutter exposure time
    }

    @Override
    public void onGyroData(List<GyroData> gyroList) {
        // Callback frequency 10Hz, 50 data per group
        // gyroData.timestamp: Time after the camera is turned on
        // gyroData.ax, gyroData.ay, gyroData.az: Triaxial acceleration
        // gyroData.gx, gyroData.gy, gyroData.gz: Three-axis gyroscope
    }

    @Override
    public void onVideoData(VideoData videoData) {
        // Callback frequency 500Hz
        // videoData.timestamp: Time after the camera is turned on
        // videoData.data: Preview raw stream data per frame
        // videoData.size: Length of each frame of data
    }

}
```

### Get Preview Status

```Java
int previewStatus = InstaCameraManager.getInstance().getPreviewStatus();

// Preview status
// InstaCameraManager.PREVIEW_STATUS_IDLE：Preview stream is not opening
// InstaCameraManager.PREVIEW_STATUS_OPENING：Preview stream is opening
// InstaCameraManager.PREVIEW_STATUS_OPENED：The preview stream has been opened
```

### Set Preview Parameters

There are three types of preview streams:

1. `PREVIEW_TYPE_NORMAL`：For normal preview or capturing
2. `PREVIEW_TYPE_RECORD`：Only for recording
3. `PREVIEW_TYPE_LIVE`：LIVE

#### Preview Resolution

You need to use the following code to get real-time support for camera resolution

```java
List<PreviewStreamResolution> supportedList = InstaCameraManager.getInstance().getSupportedPreviewStreamResolution(InstaCameraManager.PREVIEW_TYPE_LIVE);
```

> Note: For X4 and X5, the preview stream resolution cannot be adjusted. The default setting for X4 is 2880x1440, and the default setting for X5 is 2656x1328. The X3 can be manually set to 3840x1920/1440x720.
>
> ```java
> //Set resolution and preview stream type
> InstaCameraManager.getInstance().startPreviewStream(
>     PreviewStreamResolution.STREAM_368_368_30FPS，
>     InstaCameraManager.PREVIEW_TYPE_LIVE
> );
> ```

#### Preview Audio

You can set whether to turn on audio during preview.

```java
PreviewParamsBuilder builder = new PreviewParamsBuilder()
        .setStreamResolution(PreviewStreamResolution.STREAM_368_368_30FPS)
        .setPreviewType(InstaCameraManager.PREVIEW_TYPE_LIVE)
        // Whether to turn on audio
        .setAudioEnabled(true);
InstaCameraManager.getInstance().startPreviewStream(builder);
```

## Capture Control Module

### Get the Current Capturing Mode

```Java
CaptureMode captureMode = InstaCameraManager.getInstance().getCurrentCaptureMode();
// The current SDK supports the following capturing modes. Specific camera models may exhibit slight variations; refer to your camera's display for exact details:
Normal Capture：CaptureMode.CAPTURE_NORMAL
HDR Capture：CaptureMode.HDR_CAPTURE
Night Scene：CaptureMode.NIGHT_SCENE
Burst Mode：CaptureMode.BURST
Interval Shooting：CaptureMode.INTERVAL_SHOOTING
Starlapse Shooting：CaptureMode.STARLAPSE_SHOOTING
Normal Video Recording：CaptureMode.RECORD_NORMAL
HDR Recording：CaptureMode.HDR_RECORD
Bullet Time：CaptureMode.BULLETTIME
Timelapse Mode：CaptureMode.TIMELAPSE
Time Shift Mode：CaptureMode.TIME_SHIFT
Loop Recording Mode：CaptureMode.LOOPER_RECORDING
Super Recording：CaptureMode.SUPER_RECORD
Slow Motion：CaptureMode.SLOW_MOTION
Selfie Tracking Mode：CaptureMode.SELFIE_RECORD
Pure Video Mode (Night Scene)：CaptureMode.PURE_RECORD
LIVE：CaptureMode.LIVE
```

### Switching Capturing Mode

```Java
InstaCameraManager.getInstance().setCaptureMode(captureMode, errorCode -> {
    if (errorCode == 0) {
        // Switching successful
    } else {
        // Switching failed
    }
});
```

### Capturing Status Notification

> Note: For the X3 and earlier camera models, videos exceeding 30 minutes will be split into two recordings. The camera will automatically stop recording the first segment and begin recording the second segment.

```Java
InstaCameraManager.getInstance().setCaptureStatusListener(new ICaptureStatusListener() {
    @Override
    public void onCaptureCountChanged(int captureCount) {
        // Number of shots captured for modes such as interval shooting and starry sky mode,
    }

    @Override
    public void onCaptureStarting() {
        // Capture is about to start
    }

    @Override
    public void onCaptureStopping() {
        // Capture is about to stop
    }
    
    @Override
    public void onCaptureTimeChanged(long captureTime) {
       // Recording time callback, applicable only in video recording mode
    }

    @Override
    public void onCaptureWorking() {
        // Capturing in progress
    }

    @Override
    public void onCaptureFinish(String[] filePaths) {
        // Capture finished, returns the file paths of the final products
    }
    
    @Override
    public void onCaptureError(int errorCode) {
        // capturing failed error reference capturing failed error code
    }
});
```

### Parameter Settings

#### Capturing Properties

The current SDK supports the following capturing properties:

- Exposure Mode: *`CaptureSetting.EXPOSURE`* 

```Plain
FULL_AUTO     = Exposure.FULL_AUTO,     // Full Auto
AUTO          = Exposure.AUTO,          // Auto
ISO_FIRST     = Exposure.ISO_FIRST,     // ISO Priority
SHUTTER_FIRST = Exposure.SHUTTER_FIRST, // Shutter Priority
MANUAL        = Exposure.MANUAL,        // Manual
ADAPTIVE      = Exposure.ADAPTIVE       // Panorama Adaptive Exposure
```

- Full Auto: *`Exposure.FULL_AUTO`*

- Auto: *`Exposure.AUTO`*

- ISO Priority: *`Exposure.ISO_FIRST`*

- Shutter Priority: *`Exposure.SHUTTER_FIRST`*

- Manual: *`Exposure.MANUAL`*

- Panorama Adaptive Exposure: *`Exposure.ADAPTIVE`*

- EV：*`CaptureSetting.EV`*

- EV Interval: *`CaptureSetting.EV_INTERVAL`*

- Shutter Speed: *`CaptureSetting.SHUTTER`*

- Shutter Mode: *`CaptureSetting.SHUTTER_MODE`*

```Plain
AUTO   = ShutterMode.AUTO,   // Auto
SPORT  = ShutterMode.SPORT,  // Indoor Low-light Stabilization
FASTER = ShutterMode.FASTER  // Extra Fast
```

- ISO: *`CaptureSetting.ISO`*

- Max ISO: *`CaptureSetting.ISO_TOP_LIMIT`*

- Video Resolution & Frame Rate: *`CaptureSetting.RECORD_RESOLUTION`*

- Photo Resolution: *`CaptureSetting.PHOTO_RESOLUTION`*

- White Balance: *`CaptureSetting.WB`*

- AEB: *`CaptureSetting.AEB`*

- Interval Duration: *`CaptureSetting.INTERVAL`*

- Color Mode: *`CaptureSetting.GAMMA_MODE`*

```Plain
STAND  = GammaMode.STAND,  // Standard
VIVID  = GammaMode.VIVID,  // LOG
LOG    = GammaMode.LOG,    // Vivid
FLAT   = GammaMode.FLAT    // Flat
```

- Photo Format: *`CaptureSetting.RAW_TYPE`*

```Plain
JPG           = RawType.OFF,          // JPG
JPG_RAW       = RawType.DNG,          // JPG & RAW
PURESHOT      = RawType.PURESHOT,     // PURESHOT
PURESHOT_RAW  = RawType.PURESHOT_RAW, // PURESHOT & RAW
INSP          = RawType.INSP,         // INSP
INSP_RAW      = RawType.INSP_RAW      // RAW & INSP
```

- Recording Duration: *`CaptureSetting.RECORD_DURATION`*

- Low Light Stabilization: *`CaptureSetting.DARK_EIS_ENABLE`*

- Panorama Adaptive Exposure: *`CaptureSetting.PANO_EXPOSURE_MODE`*

```Plain
OFF       = PanoExposureMode.OFF,       // Off
ON        = PanoExposureMode.ON,        // Independent Exposure
LIGHT     = PanoExposureMode.LIGHT,     // Standard
LSOLATED  = PanoExposureMode.LSOLATED   // High
```

Note: For cameras prior to Insta360 X4, only `ON` and `OFF` were supported. From X4 onwards, `ON` is split into `LIGHT` and `LSOLATED`.

- Burst Capture Count: *`CaptureSetting.BURST_CAPTURE`*

- In-camera Splicing: *`CaptureSetting.INTERNAL_SPLICING`*

```Plain
ON  = InternalSplicing.ON,   // On
OFF = InternalSplicing.OFF   // Off
```

- Video HDR Mode: *`CaptureSetting.HDR_STATUS`*

```Plain
ON  = HdrStatus.ON,   // On
OFF = HdrStatus.OFF   // Off
```

> Note: From Insta360 X5 onwards, `CaptureMode.HDR_RECORD` is removed. Use `CaptureSetting.HDR_STATUS` instead.

- Photo HDR Mode: *`CaptureSetting.PHOTO_HDR_TYPE`*

```Plain
On: PhotoHdrType.HDR_ON
Off: PhotoHdrType.HDR_OFF
AEB: PhotoHdrType.HDR_AEB
```

> Note: From Insta X5 onwards, *`CaptureMode.HDR_CAPTURE`* is removed. Use *`CaptureSetting.PHOTO_HDR_TYPE`* instead.

- I-LOG: *`CaptureSetting.I_LOG`*

```Plain
ON  = ILogStatus.ON,   // On
OFF = ILogStatus.OFF   // Off
```

- Photo Countdown：*`CaptureSetting.LATENCY`* 

> Note: Only supported on X5

#### Get Capturing Parameters

You can use the *`InstaCameraManager.getInstance().getXxx(CaptureMode)`* method to retrieve the corresponding capturing properties.

| Method                      | Parameter   | Return Type      | Description                               |
| --------------------------- | ----------- | ---------------- | ----------------------------------------- |
| getRecordDuration()         | CaptureMode | RecordDuration   | Get recording duration                    |
| getRecordResolution()       | CaptureMode | RecordResolution | Get video resolution and frame rate       |
| getPhotoResolution()        | CaptureMode | PhotoResolution  | Get photo resolution                      |
| getInterval()               | CaptureMode | Interval         | Get capturing interval duration           |
| getHdrStatus()              | CaptureMode | HdrStatus        | Get HDR status for normal video recording |
| getPhotoHdrType()           | CaptureMode | PhotoHdrType     | Get HDR status for normal photo capture   |
| getGammaMode()              | CaptureMode | GammaMode        | Get color mode                            |
| getRawType()                | CaptureMode | RawType          | Get photo format                          |
| getPanoExposureMode()       | CaptureMode | PanoExposureMode | Get panorama independent exposure status  |
| getExposure()               | CaptureMode | Exposure         | Get exposure mode                         |
| getISO()                    | CaptureMode | ISO              | Get ISO value                             |
| getISOTopLimit()            | CaptureMode | ISOTopLimit      | Get ISO upper limit                       |
| getShutterMode()            | CaptureMode | ShutterMode      | Get shutter mode                          |
| getShutter()                | CaptureMode | Shutter          | Get shutter speed                         |
| getEv()                     | CaptureMode | Ev               | Get EV value                              |
| getEVInterval()             | CaptureMode | EVInterval       | Get EV interval                           |
| getAEB()                    | CaptureMode | AEB              | Get AEB                                   |
| getWB()                     | CaptureMode | WB               | Get white balance                         |
| getInternalSplicingEnable() | CaptureMode | InternalSplicing | Get in-camera stitching status            |
| getDarkEisType()            | CaptureMode | DarkEisType      | Get low-light stabilization status        |
| getBurstCapture()           | CaptureMode | BurstCapture     | Get burst mode parameters                 |
| getILogStatus()             | CaptureMode | ILogStatus       | Get the I-LOG Switch                      |
| getLatency()                | CaptureMode | Latency          | Get the photo countdown                   |

#### Set Capturing Properties

You can use the*`InstaCameraManager.getInstance().setXxx(CaptureMode)`* method to set the corresponding capturing properties.

| Method                      | Parameter 1 | Parameter 2      | Return Type | Description                              |
| --------------------------- | ----------- | ---------------- | ----------- | ---------------------------------------- |
| setRecordDuration()         | CaptureMode | RecordDuration   | void        | Set recording duration                   |
| setRecordResolution()       | CaptureMode | RecordResolution | void        | Set video resolution and frame rate      |
| setPhotoResolution()        | CaptureMode | PhotoResolution  | void        | Set photo resolution                     |
| setInterval()               | CaptureMode | Interval         | void        | Set capturing interval duration          |
| setHdrStatus()              | CaptureMode | HdrStatus        | void        | Set HDR status for normal video          |
| setPhotoHdrType()           | CaptureMode | PhotoHdrType     | void        | Set HDR status for normal photo          |
| setGammaMode()              | CaptureMode | GammaMode        | void        | Set color mode                           |
| setRawType()                | CaptureMode | RawType          | void        | Set photo format                         |
| setPanoExposureMode()       | CaptureMode | PanoExposureMode | void        | Set panorama independent exposure status |
| setExposure()               | CaptureMode | Exposure         | void        | Set exposure mode                        |
| setISO()                    | CaptureMode | ISO              | void        | Set ISO value                            |
| setISOTopLimit()            | CaptureMode | ISOTopLimit      | void        | Set ISO upper limit                      |
| setShutterMode()            | CaptureMode | ShutterMode      | void        | Set shutter mode                         |
| setShutter()                | CaptureMode | Shutter          | void        | Set shutter speed                        |
| setEv()                     | CaptureMode | Ev               | void        | Set EV value                             |
| setEVInterval()             | CaptureMode | EVInterval       | void        | Set EV interval                          |
| setAEB(CaptureMode          | CaptureMode | AEB              | void        | Set AEB                                  |
| setWB()                     | CaptureMode | WB               | void        | Set white balance                        |
| setInternalSplicingEnable() | CaptureMode | InternalSplicing | void        | Set in-camera stitching                  |
| setDarkEisType()            | CaptureMode | DarkEisType      | void        | Set low-light stabilization              |
| setBurstCapture()           | CaptureMode | BurstCapture     | void        | Set burst mode parameters                |
| setILogStatus               | CaptureMode | ILogStatus       | void        | Setting the I-LOG switch                 |
| setLatency()                | CaptureMode | Latency          | void        | Set a photo countdown                    |

#### Batch Set Capturing Properties

```Java
//Start batch setting of capturing properties
InstaCameraManager.getInstance().beginSettingOptions();

InstaCameraManager.getInstance().setEv(captureMode, EV.EV_0);
InstaCameraManager.getInstance().setShutter(captureMode, Shutter.SHUTTER_1_2);
...
InstaCameraManager.getInstance().setWB(captureMode, WB.WB_2200K);

//Submit batch settings to take effect
InstaCameraManager.getInstance().commitSettingOptions();
```

#### Dependency between Capturing Properties

There are interdependencies among different capturing properties. Setting property A may affect the support list of property B. Therefore, after setting a property, it is necessary to check the other affected properties.

> Note: Since the callback function may be invoked multiple times, the elements in the CaptureSetting list may differ each time it is called.

Example： 

```Java
// The example only demonstrates setting PhotoHdrType; other property settings follow the same pattern；
InstaCameraManager.getInstance().setPhotoHdrType(captureMode, photoHdrType, new InstaCameraManager.IDependChecker() {
    @Override
    public void check(List<CaptureSetting> captureSettings) {
        // captureSettings：list of affected capturing properties
        // UI updates can be handled here
    }
});
```

### Photo Capture

#### Determine whether the Camera is in Photo Mode

```Java
boolean isPhotoMode = CaptureMode.PURE_RECORD.isPhotoMode();
```

#### Start Capturing

```Plain
// Start Normal capture
InstaCameraManager.getInstance().startNormalCapture();

// Start HDR capture
InstaCameraManager.getInstance().startHDRCapture();

// Start super night scene
InstaCameraManager.getInstance().startNightScene();

// Start burst capture
InstaCameraManager.getInstance().startBurstCapture();

// Start Interval Shooting
InstaCameraManager.getInstance().startIntervalShooting();
// Stop interval shooting
InstaCameraManager.getInstance().stopIntervalShooting();

// Start starlapse shooting
InstaCameraManager.getInstance().startStarLapseShooting();
// Stop starlapse shooting
InstaCameraManager.getInstance().stopStarLapseShooting();
```

### Video Recording

#### Determine if it is Video Mode

```Java
boolean isVideoMode = CaptureMode.PURE_RECORD.isVideoMode();
```

#### Start/Stop Recording

```Plain
// Start normal recording
InstaCameraManager.getInstance().startNormalRecord();
// Stop normal recording
InstaCameraManager.getInstance().stopNormalRecord();

// Start HDR recording
InstaCameraManager.getInstance().startHDRRecord();
// Stop HDR recording
InstaCameraManager.getInstance().stopHDRRecord();

// Start Bullet Time
InstaCameraManager.getInstance().startBulletTime();
// Stop Bullet Time
InstaCameraManager.getInstance().stopBulletTime();

// Start timelapse recording
InstaCameraManager.getInstance().startTimeLapse();
// Stop timelapse recording
InstaCameraManager.getInstance().stopTimeLapse();

// Start time shift recording
InstaCameraManager.getInstance().startTimeShift();
// Stop time shift recording
InstaCameraManager.getInstance().stopTimeShift();

// Start loop recording
InstaCameraManager.getInstance().startLooperRecord();
// Stop loop recording
InstaCameraManager.getInstance().stopLooperRecord();

// Start super recording
InstaCameraManager.getInstance().startSuperRecord();
// Stop super recording
InstaCameraManager.getInstance().stopSuperRecord();

// Start slow motion recording
InstaCameraManager.getInstance().startSlowMotionRecord();
// Stop slow motion recording
InstaCameraManager.getInstance().stopSlowMotionRecord();

// Start selfie tracking mode
InstaCameraManager.getInstance().startSelfieRecord();
// Stop selfie tracking mode
InstaCameraManager.getInstance().stopSelfieRecord();

// Start night scene recording
InstaCameraManager.getInstance().startPureRecord();
// Stop night scene recording
InstaCameraManager.getInstance().stopPureRecord();
```

## LIVE

> Note: Required to use with the `Media SDK`.

### Determine if it is Live Streaming Mode

```Java
boolean isLiveMode = CaptureMode.PURE_RECORD.isLiveMode();
```

### Start Live Streaming

> Note: After the preview stream is turned off, the camera will automatically switch the capturing mode to normal recording. Before starting LIVE, you need to check whether the current capturing mode is LIVE.

```java
LiveParamsBuilder builder = new LiveParamsBuilder()
         //(Required) Set the RTMP address for stream pushing
        .setRtmp(String rtmp)
        //(Required) Set the stream width, e.g., 1440
        .setWidth(int width)
        //(Required) Set the stream height, e.g., 720
        .setHeight(int height)
        // (Required) Set the frame rate, e.g., 30
        .setFps(int fps)
        //(Required) Set the bitrate, e.g., 2*1024*1024
        .setBitrate(int bitrate)
        //(Optional) Whether it is panoramic streaming or not, default is true
        .setPanorama(true);
        
InstaCameraManager.getInstance().startLive(builder, new ILiveStatusListener() {

    @Override
    public void onLiveFpsUpdate(int fps) {
        // fps: current live streaming frame rate
    }

    @Override
    public void onLivePushError(int error, String desc) {
        // Live stream push failed
    }

    @Override
    public void onLivePushFinished() {
        // Live stream push finished
    }

    @Override
    public void onLivePushStarted() {
        // Live stream push started
    }
});
```

### End LIVE

```java
InstaCameraManager.getInstance().stopLive();
```

## File Management Module

### Get the Camera Socket Address

```java
// Return value example：http://192.168.42.1:80
InstaCameraManager.getInstance().getCameraHttpPrefix();
```

### Get Camera File URL

```java
// Return value example：[/DCIM/Camera01/VID_20250326_184618_00_001.insv]
List<String> usls = InstaCameraManager.getInstance().getAllUrlList();

// Includes files in the video
List<String> usls = InstaCameraManager.getInstance().getAllUrlListIncludeRecording();
```

### Delete Camera Files

```java
// Delete a single camera file
InstaCameraManager.getInstance().deleteFile(String filePath，new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // Delete successfully
    }

    @Override
    public void onFailed() {
       // Delete failed
    }

    @Override
    public void onCameraConnectError() {
      // Camera connection error
    }
});

// Batch delete camera files
InstaCameraManager.getInstance().deleteFileList(List<String> fileUrlList，new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // Delete successfully
    }

    @Override
    public void onFailed() {
       // Delete failed
    }

    @Override
    public void onCameraConnectError() {
      // Camera connection error
    }
});
```

## Logging Module

### Set Log Root Path

The SDK's log system supports saving logs to local files. Call the following code to set the log save location.

```Java
// Set the log cache path. If not set, no cache will be performed.
LogManager.instance.setLogRootPath(StorageUtils.getInternalRootPath() + "/log");
```

### Start/Stop Logging

Enable the global log real-time capture function, which will capture all logs, including SDK logs, system logs, host APP logs, etc., and save them in the directory specified above.

```Java
// Enable real-time log capture
LogManager.instance.startLogDumper();

// Stop real-time log capture
LogManager.instance.stopLogDumper();
```

## GPS Data Module

### Settings in Photo Mode

```java
// android.location.Location class，developer obtains it via system API
Location location;

// Set GPS data when taking a photo
GpsData gpsData = new GpsData();
gpsData.setLatitude(location.getLatitude());
gpsData.setLongitude(location.getLongitude());
gpsData.setAltitude(location.getAltitude());
InstaCameraManager.getInstance().startNormalCapture(gpsData.simpleGpsDataToByteArray());
```

#### GPS Data Interface for Input in Different Photo Modes

```Plain
// Start normal capturing and input GPS data.
InstaCameraManager.getInstance().startNormalCapture(byte[]);

// Start HDR capturing and input GPS data.
InstaCameraManager.getInstance().startHDRCapture(byte[]);

// Start burst capturing and input GPS data.
InstaCameraManager.getInstance().startBurstCapture(byte[]);

// Start interval shooting and input GPS data.
InstaCameraManager.getInstance().stopIntervalShooting(byte[]);
```

### Settings in Record Mode

GPS data for recording settings can be divided into the following two situations:

#### When the Recording is Stopped

Stopping recording is the same as taking photos. The example code is as follows:

```java
// android.location.Location class, developer obtains it via system API
Location location;

// Set GPS data when stopping the recording
GpsData gpsData = new GpsData();
gpsData.setLatitude(location.getLatitude());
gpsData.setLongitude(location.getLongitude());
gpsData.setAltitude(location.getAltitude());
InstaCameraManager.getInstance().stopNormalRecord(gpsData.simpleGpsDataToByteArray());
```

#### In Recording

GPS data can be set at any time in the video recording. GPS data is usually used to create the camera's motion trajectory. Therefore, it is recommended to set the GPS interval as much as possible each time. The example code is as follows:

```java
// Collect GPS data every 100ms; once 20 GPS data points are accumulated, send them to the camera

private Handler mMainHandler = new Handler(Looper.getMainLooper());
// List of GPS data
private List<GpsData> mCollectGpsDataList = new ArrayList<GpsData>();

private Runnable mCollectGpsDataRunnable = new Runnable() {
    @Override
    public void run() {
        // android.location.Location class, developer obtains it via system API
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

// Start collecting GPS data
mMainHandler.post(mCollectGpsDataRunnable);
```

#### GPS Data Interface for Input in Different Recording Modes

```Plain
// Stop normal recording and input GPS data.
InstaCameraManager.getInstance().stopNormalRecord(byte[]);

// Stop HDR recording and input GPS data.
InstaCameraManager.getInstance().stopHDRRecord(byte[]);

// Stop Bullet Time and input GPS data.
InstaCameraManager.getInstance().stopBulletTime(byte[]);

// Stop Timelapse and input GPS data.
InstaCameraManager.getInstance().stopTimeLapse(byte[]);

// Stop Time Shift and input GPS data.
InstaCameraManager.getInstance().stopTimeShift(byte[]);

// Stop super Recording and input GPS data.
InstaCameraManager.getInstance().stopSuperRecord(byte[]);

// Stop loop Recording and input GPS data.
InstaCameraManager.getInstance().stopLooperRecord(byte[]);

// Stop night Scene recording and input GPS data.
InstaCameraManager.getInstance().stopPureRecord(byte[]);

// Stop selfie tracking recording and input GPS data.
InstaCameraManager.getInstance().stopSelfieRecord(byte[]);
```

## Other Features Module

### Camera Activation

Activating the camera requires the use of two pieces of data, `appId`and `secretKey`. These two data need to be applied for by the official Insta360.

> Note: Activation requires network permissions

> Note: Nano S cameras do not support this feature.

Example code is as follows：

```Java
String appid = "xxxxxxxx";
String mSecretKey = "xxxxxxx";
InstaCameraManager.getInstance().activateCamera(new SecretInfo(appid , mSecretKey), new InstaCameraManager.IActivateCameraCallback() {
    @Override
    public void onStart() {
        // Began to activate
    }

    @Override
    public void onSuccess() {
        // Activation successful
    }

    @Override
    public void onFailed(String message) {
        // Activation failed
    }
});
```

### Camera Prompt Sound

> Note: The Nano S camera does not support the prompt sound switch, but supports the setting.

The switch for camera prompt sound. Example code is as follows:

```Java
// Turn on the camera prompt tone
InstaCameraManager.getInstance().setCameraBeepSwitch(true);

// Determine whether the camera has a prompt tone turned on
boolean isBeep = InstaCameraManager.getInstance().isCameraBeep();
```

### Format the SD Card

```java
InstaCameraManager.getInstance().formatStorage(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // SD card formatted successfully
    }

    @Override
    public void onFailed() {
        // SD card formatting failed
    }

    @Override
    public void onCameraConnectError() {
        // Camera connection error
    }
});
```

### Camera Lock Screen

> Note: Nano S cameras do not support this feature.

After the camera is locked, the user cannot operate the camera screen. Example code is as follows:

```Java
// true: lock screen   false: unlock screen
InstaCameraManager.getInstance().setCameraLockScreen(true);
```

### Turn off the Camera

Instructions to control camera shutdown, sample code is as follows:

> Note: Only Insta360X5 and later cameras support this feature.

```Java
InstaCameraManager.getInstance().shutdownCamera();
```

### Calibration of Gyroscope

> Note: Nano S cameras do not support this feature.

When the camera is connected to Wi-Fi, this function should be used. Before calibration, make sure the camera is placed vertically on a stable and horizontal surface.

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

### Firmware Upgrade

Use the `FwUpgradeManager` class to upgrade the firmware version. The example code is as follows:

> Note: Nano S cameras do not support this feature.

> Note: Please make sure to upload the correct files to the camera to prevent any potential damage. For example, do not upload the ONE X firmware upgrade package to the ONE X2 camera.

```Java
// Determine if an upgrade is in progress
FwUpgradeManager.getInstance().isUpgrading()；

// Cancel the upgrade
FwUpgradeManager.getInstance().cancelUpgrade();

String fwFilePath = "/xxx/InstaOneXFW.bin";
// Upgrade firmware
FwUpgradeManager.getInstance().startUpgrade(fwFilePath, new FwUpgradeListener() {
    @Override
    public void onUpgradeSuccess() {
        // Callback after successful upgrade
    }

    @Override
    public void onUpgradeFail(int errorCode, String message) {
         // Upgrade failure callback
         // Firmware error code
    }

    @Override
    public void onUpgradeCancel() {
         // Upgrade cancel callback
    }

    @Override
    public void onUpgradeProgress(double progress) {
        // Upgrade progress callback: 0-1
    }
});
```

### Sharpness Settings (Currently Unsupported)

## Error Code

### Camera Connection Error Code

| Error Code | Explanation                                                  |
| ---------- | ------------------------------------------------------------ |
| -122       | Bluetooth device disconnected                                |
| 501        | Bluetooth disconnection occurred                             |
| 2201       | System error BleException.*ERROR_CODE_SYSTEM_STATUS_133*     |
| 2002       | Open camera timed out                                        |
| 2206       | System error BleException.*ERROR_CODE_NOTIFY_FAILURE*        |
| 2207       | System error BleException.*ERROR_CODE_SHAKE_HAND*            |
| 3201       | Socket connection failed                                     |
| 3205       | Socket connection timed out                                  |
| 3206       | Socket reconnect timed out                                   |
| 4001       | System error BleException.*ERROR_CODE_SYNC_PACK*             |
| 4101       | Camera type error                                            |
| 4103       | *Camera sync parameter timed out*                            |
| 4402       | *Camera operating system error*                              |
| 4403       | Camera is occupied                                           |
| 4405       | Parsing proto failed                                         |
| 4406       | *Camera sync parameter retry timeout*                        |
| 4407       | Camera data exception                                        |
| 4414       | *Failed to wake up Bluetooth*                                |
| 4417       | Camera disconnected during operation                         |
| 4504       | Camera disconnected when connected                           |
| 4505       | Bluetooth binding rejected                                   |
| 4507       | Bluetooth write failed                                       |
| 4508       | *Device Bluetooth busy*                                      |
| 4602       | Device Wi-Fi busy                                            |
| 4605       | *USB* *error*                                                |
| 4606       | *Socket Disconnected Due to Dirty Data*                      |
| -14169     | Failed to synchronize camera parameters due to Bluetooth failure |

### Firmware Upgrade Error Code

| Error Code | Explanation                                                  |
| ---------- | ------------------------------------------------------------ |
| -1000      | Already upgrading                                            |
| -1001      | Camera not connected                                         |
| -1002      | When the camera battery is below 12%, the upgrade will fail, please make sure the camera battery is fully charged |
| -14004     | Http server error                                            |
| 400        | Http server error                                            |
| 500        | Http server error                                            |
| -14005     | Socket IO exception                                          |

1. ### Capturing Failed Error Code

| Error Code | Explanation                               |
| ---------- | ----------------------------------------- |
| -14161     | Low battery                               |
| -14162     | Exceeded the maximum recording time limit |
| -14163     | SD card is full                           |
| -14165     | Exceeded the file limit                   |
| -14166     | SD card speed is low                      |
| -14167     | Coding anomaly                            |
| -14168     | Frame loss                                |
| -14180     | Excessive file fragmentation              |
| -14181     | Camera temperature is too high            |
| -14182     | Low power start                           |
| -14183     | Storage Deflection Start                  |
| -14184     | High temperature start                    |
| -14164     | Other situations                          |

1. ### Camera Activation Error Code

| Error Code | Explanation                                                  |
| ---------- | ------------------------------------------------------------ |
| 400        | Parameter error, possibly due to the camera type or serial number not being obtained |
| 401        | Interface signature verification failed                      |
| 402        | Data encryption verification failed                          |
| 403        | The nonce credential has already been used                   |
| 406        | The timestamp has expired                                    |
| 429        | Frequent interface requests                                  |
| 500        | Internal system exception                                    |
| 3001       | Device serial number does not match device type              |
| 3002       | Camera has been activated                                    |
| 3003       | Activation denied; IP address is not within a valid region   |
| 3004       | Unable to generate a valid serial number                     |
| 3005       | The system does not support activating devices of this model |
| 3006       | The serial number has been locked by the backend and cannot be activated |
| 429        | The activation IP is subject to rate limiting. Rate limiting policy: 45 attempts per IP per day |

# Media SDK Function

## Environmental Preparation

- Add the maven address to the build file (`build.gradle` file in the project root directory)

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

- Import the dependency library in the`build.gradle` file of the app directory

```Groovy
dependencies {
    implementation 'com.arashivision.sdk:sdkmedia:x.x.x'
}
```

## Preview Module

If you have integrated the `Camera SDK`, you can open and display the preview content.

### Preview Control Initalization

You need to put *`InstaCapturePlayerView`*into your XML file and bind the lifecycle of the Activity:

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

### Display Preview Screen

Show preview screen in *`IPreviewStatusListener.onOpen()`* callback of `Camera SDK`

```Java
@Overridepublic void onOpened() {
    InstaCameraManager.getInstance().setStreamEncode();
    mCapturePlayerView.setPlayerViewListener(new PlayerViewListener() {
        @Override
        public void onLoadingFinish() {
            // Fixed writing
            InstaCameraManager.getInstance().setPipeline(mCapturePlayerView.getPipeline());
        }
        
        @Override
        public void onLoadingStatusChanged(boolean isLoading) {
            // loading status change
        }

        @Override
        public void onFail(String desc) {
        }
        
        @Override
        public void onFirstFrameRender() {
            // First frame rendered successfully
        }
    });
    // Set parameters, which will be described in detail later
    mCapturePlayerView.prepare(createParams());
    // Play
    mCapturePlayerView.play();
    // Keep the screen always on
    mCapturePlayerView.setKeepScreenOn(false);
}
```

### Release Resources

When closing the preview stream, you need to release the *`InstaCapturePlayerView`*

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

### Set Preview Parameters

```Java
private CaptureParamsBuilderV2createParams() {
    CaptureParamsBuilderV2builder = new CaptureParamsBuilderV2()
            // (Optional) Set your resolution and frame rate. The default values are generated intelligently by the SDK.
            .setWidth(int width)
            .setHeight(int height)
            .setFps(int fps) //Max 30fps
            // (optional) achromatic Default value is true
            .setImageFusionEnabled(boolean enable)
            // (Optional) Set the gesture switch. The default value is generated intelligently by the SDK
            .setGestureEnabled(boolean enable)
            // (Optional) Set the number of frames cached for anti-shake. The more frames cached, the better the anti-shake effect, but the higher the image delay.
            .setStabCacheFrameNum(5)
            // (Optional) Set the stabilization type. The default value is generated intelligently by the SDK.
            .setStabType(@InstaStabType.StabType int mStabType)
            // (Optional) Set the rendering mode; the default value is “RenderModel.AUTO”.
            .setRenderModel(RenderModel.PLANE_STITCH)
            // (Optional) Set the screen ratio. For example, 9:16. The default value is generated by the SDK.
            // If the rendering mode type is “PLANE_STITCH”, it is recommended to set the aspect ratio to 2:1.
            .setScreenRatio(int,int)
            // (Optional) Set custom Surface information
            .setCameraRenderSurfaceInfo(CameraRenderSurfaceInfo);
    return builder;
}
```

### Switch Rendering Mode

```Java
// Ordinary
mCapturePlayerView.switchNormalMode();
// Fish eye
mCapturePlayerView.switchFisheyeMode();
// Perspective
mCapturePlayerView.switchPerspectiveMode();
```

### Field of View Distance

You can customize the viewing angle and distance according to your preferences.

> Fov: Angle range 0~180
>
> Distance: line of sight range 0~∞

```Java
mCapturePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### Gesture Operation

You can set *`PlayerGestureListener`* to observe gesture actions:

```Java
mCapturePlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // Use finger press event
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // User finger click event, similar to View. OnClickListener. OnClick (v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // User finger lift event
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // User long press event
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // User two-finger zoom event
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // Rebound animation when scaling to maximum or minimum
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // Zoom rebound animation ends
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // User finger swipe event
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // Fling animation
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // Quick swipe animation ends
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## WorkWrapper

`WorkWrapper` is a collection of all data in a slice file. For example: Continuous capturing generates multiple files, which are then encapsulated into a single `WorkWrapper` for unified processing. Subsequent operations such as playback, export, and information retrieval are all performed by manipulating the `WorkWrapper` object.

### How to Get WorkWrapper

You can scan media files from your camera or local directory to obtain`List<WorkWrapper>`。

> Note: This is a time-consuming operation that needs to be handled in a child thread.

```Java
// Get from the camera
List<WorkWrapper> list = WorkUtils.getAllCameraWorks();

// Get from local file
List<WorkWrapper> list = WorkUtils.getAllLocalWorks(String dirPath);

// If you have the URL of the media file, you can also create WorkWrapper yourself
String[] urls = {img1.insv, img2.insv, img3.insv};
WorkWrapper workWrapper = new WorkWrapper(urls);
```

### Load File Trailer Data

Call the *`WorkWrapper.loadExtraData()`* function to load the file trailer data. The `play` and `export` functions can be used only after the file trailer data is loaded.

Call the *`WorkWrapper.isExtraDataLoaded()`* function to determine whether the file trailer data has been loaded.

> Note: This method is a time-consuming operation and needs to be executed in the IO thread.

### Read the Content of WorkWrapper

The information you can obtain from *`WorkWrapper`* is as follows:

| Method                    | Parameter | Return Value Type | Explanantion                                                 |
| ------------------------- | --------- | ----------------- | ------------------------------------------------------------ |
| getIdenticalKey()         | -         | String            | For Glide or other DiskCacheKeys                             |
| getUrls()                 | -         | String[]          | Get media file URL                                           |
| getUrls()                 | boolean   | String[]          | Get the URL of the media file. Set true/false to determine whether to include the dng file path |
| getAllUrls()              | -         | String[]          | Get all file paths for operations such as downloading and deletion. |
| getWidth()                | -         | int               | Get width                                                    |
| getHeight()               | -         | int               | Get height                                                   |
| getBitrate()              | -         | int               | Get the video bitrate. If it is a photo, return 0            |
| getFps()                  | -         | double            | Get the video frame rate. If it is a photo, return 0         |
| isPhoto()                 | -         | boolean           | Whether it is a photo                                        |
| isHDRPhoto()              | -         | boolean           | Is it an HDR photo                                           |
| supportPureShot()         | -         | boolean           | Whether to support PureShot algorithm                        |
| isVideo()                 | -         | boolean           | Whether it is a video                                        |
| isHDRVideo()              | -         | boolean           | Is it HDR video                                              |
| isCameraFile()            | -         | boolean           | Is the media file sourced from the camera device             |
| isLocalFile()             | -         | boolean           | Is the media file sourced from a mobile device               |
| getCreationTime()         | -         | long              | Obtain media file shooting timestamp, unit: ms               |
| getFirstFrameTimeOffset() | -         | long              | Obtain the offset of timestamp relative to camera startup when shooting media files, used to match gyroscope, exposure, and other data, unit: ms |
| getGyroInfo()             | -         | GyroInfo[]        | Obtain gyroscope data for media files                        |
| getExposureData()         | -         | ExposureData[]    | Obtain exposure data for media files                         |
| loadThumbnail()           | -         | Bitmap            | Loading thumbnails (time-consuming operation)                |
| loadExtraData()           | -         | -                 | Load file trailer data (time-consuming operation)            |
| isExtraDataLoaded()       | -         | boolean           | Whether the file trailer data has been loaded                |

## Image Player

If you want to play images or videos, you must first create a `WorkWrapper`。

### Player Initialization

Put the *`InstaImagePlayerView`* into an XML file and bind the lifecycle of the Activity.

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

### Set Playback Parameters

```java
ImageParamsBuilder builder = new ImageParamsBuilder()
      //  (Optional) Whether to enable dynamic stitching, default is true
      .setDynamicStitch(boolean dynamicStitch)
      // (Optional) Set the stabilization type, default is InstaStabType STAB_TYPE_OFF
      .setStabType(@InstaStabType.StabType int mStabType)
      // (Optional) Set the playback proxy file, such as HDR.jpg generated by splicing, which is empty by default
      .setUrlForPlay(String url)
      // (Optional) Sets the render model type, default is Render_MODE_AUTO
      .setRenderModel(RenderModel.PLANE_STITCH)
      // (Optional) Set the aspect ratio of the screen, default is full screen display
      //If the render mode type is "RENDER_mode_PLANE_STITCH", the recommended setting ratio is 2:1
      .setScreenRatio(int ratioX, int ratioY)
      // (Optional) Eliminate color difference in image stitching, default is false
      .setImageFusion(boolean imageFusion)
      // (Optional) Set the lens guards type, default is OffsetType. ORIGINAL
      .setOffsetType(OffsetType offsetType)
      // (Optional) Set the color enhancement switch; default is false.
      .setColorPlusEnable(boolean enabled);
      // (Optional) Set the color enhancement intensity, default value is 1, valid range 0-1
      .setColorPlusFilterIntensity(float colorPlusFilterIntensity);
      // (Optional) Whether to allow gesture operations, default is true
      .setGestureEnabled(boolean enabled);
      // (Optional) thumbnail cache folder, default is getCacheDir () + "/ work_thumbnail "
      .setCacheWorkThumbnailRootPath(String path)
      // (Optional) Stabilizer data cache folder, default is get CacheD ir () + "/ stabilizer "
      .setStabilizerCacheRootPath(String path)
      // (Optional) Cache folder, default is getExternalCacheDir () + "/ template_blender "
      .setCacheTemplateBlenderRootPath(String path)
      // (Optional) Star mode image cache folder, default is getCacheDir () + "/ cut_scene "
      .setCacheCutSceneRootPath(String path)；
```

### Start Playing

`WorkWrapper` and playback parameters need to be set before playback.

```Java
// Set WorkWrapper and playback parameters
mImagePlayerView.prepare(workWrapper, new ImageParamsBuilder());
// Start playing
mImagePlayerView.play();
```

### Release Resources

When the Activity is destroyed, release *`InstaImagePlayerView`*。

```Java
@Override
protected void onDestroy() {
    super.onDestroy();
    mImagePlayerView.destroy();
}
```

### Switch Rendering Mode

> Note: If you want to switch between tiling mode and other modes, you must first restart the player.

If the rendering mode you are using is `RENDER_MODE_AUTO`, you can switch between the following modes.

```java
// Switch to normal mode
mImagePlayerView.switchNormalMode();

// Switch to fisheye mode
mImagePlayerView.switchFisheyeMode();

// Switch to perspective mode
mImagePlayerView.switchPerspectiveMode();

// To switch to tiling mode, you need to restart the player
ImageParamsBuilder builder = new ImageParamsBuilder()；
builder.setRenderModelType(ImageParamsBuilder.RENDER_MODE_PLANE_STITCH);
builder.setScreenRatio(2, 1);
mImagePlayerView.prepare(mWorkWrapper, builder);
mImagePlayerView.play();
```

### Field of View Distance

You can customize the viewing angle and distance according to your preferences.

> Fov: Angle range 0~180
>
> Distance: line of sight range 0~∞

```java
mImagePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### Set Listener

You can set the *`PlayerViewListener`* to observe changes in the player state.

```Java
mImagePlayerView.setPlayerViewListener(new PlayerViewListener() {
    @Override
    public void onLoadingStatusChanged(boolean isLoading) {
        // Player loading status changed
        //isLoading：whether loading
    }

    @Override
    public void onLoadingFinish() {
        // Player loading complete
    }

    @Override
    public void onFail(int errorCode, String errorMsg) {
        // if GPU not support, errorCode is -10003 or -10005 or -13020
        // Player error
    }
    
    @Override
    public void onFirstFrameRender() {
        // First frame rendering successfully
    }
});
```

### Gesture Operation

You can set *`PlayerGestureListener`* to observe gesture actions:

```typescript
mImagePlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // User finger press event
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // User finger click event, similar to View.OnClickListener.OnClick(v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // User finger lift event
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // User long press event
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // User two-finger zoom event
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // Rebound animation when scaling to maximum or minimum
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // Zoom rebound animation ends
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // User finger swipe event
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // Quick slide animation
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // Quick swipe animation ends
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## Video Player

### Player Initialization

Put the *`InstaVideoPlayerView `*into an XML file and bind the lifecycle of the Activity.

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

1. ### Set Playback Parameters

```java
VideoParamsBuilder builder = new VideoParamsBuilder()
      // (Optional) Load icon, default is empty
      .setLoadingImageResId(int resId)
      // (Optional) Background color during loading, default is black
      .setLoadingBackgroundColor(int color)
      // (Optional) Whether to play automatically
      .setAutoPlayAfterPrepared(boolean autoPlayAfterPrepared)
      // (Optional) Set the stabilization type, default is InstaStabType STAB_TYPE_OFF
      .setStabType(@InstaStabType.StabType int stabType)
      // (Optional) Whether to play in loop; default is true
      .setIsLooping(boolean isLooping)
      // (Optional) Set whether to play the LRV
      .setLrvEnable(boolean isLrvEnable)
      // (Optional) Whether to enable dynamic stitching, default is true
      .setDynamicStitch(boolean dynamicStitch)
      // (Optional) Eliminate color difference in image stitching, default is false
      .setImageFusion(boolean imageFusion)
      // (Optional) Set the lens guards type, default is OffsetType. ORIGINAL
      .setOffsetType(OffsetType offsetType)
      // (Optional) Remove purple fringing. Default value is false. Requires adding an algorithm file.
      .setDePurpleFilterOn(boolean isDePurpleFilterOn)
      // (Optional) Set the color enhancement switch, default is false
      .setColorPlusEnable(boolean enabled);
      // (Optional) Set the color enhancement intensity, default value is 1, valid range 0-1
      .setColorPlusFilterIntensity(float colorPlusFilterIntensity);
      // (Optional) Sets the render model type, default is Render_MODE_AUTO
      .setRenderModel(RenderModel.PLANE_STITCH)
      // (Optional) Set the aspect ratio of the screen, default is full screen display.
      // If the render mode type is "RENDER_MODE_PLANE_STITCH", it is recommended to set the ratio to 2:1
      .setScreenRatio(int ratioX, int ratioY)
      // (Optional) Whether to allow gesture operations, default is true
      .setGestureEnabled(boolean enabled)
      // (Optional) thumbnail cache folder, default is getCacheDir () + "/ work_thumbnail "
      .setCacheWorkThumbnailRootPath(String path)
      // (Optional) Star mode image cache folder, default is getCacheDir () + "/ cut_scene "
      .setCacheCutSceneRootPath(String path);
```

### Start Playing

*`WorkWrapper`* and playback parameters need to be set before playback.

```java
mVideoPlayerView.prepare(workWrapper, new VideoParamsBuilder());
mVideoPlayerView.play();
```

### Release Resources

When the Activity is destroyed, release *`InstaVideoPlayerView`* 。

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    mVideoPlayerView.destroy();
}
```

### Playback Control

You can control the playback progress, pause/resume, playback volume, etc. Of the video by calling the code in *`VideoPlayerView`*.

| 方法                    | 参数              | 返回值  | 说明                                            |
| ----------------------- | ----------------- | ------- | ----------------------------------------------- |
| isPlaying               | -                 | boolean | Is the video playing                            |
| isLooping               | -                 | boolean | Whether play it on a loop                       |
| setLooping              | boolean           | void    | Set whether play it on a loop                   |
| setVolume               | float (Range 0-1) | void    | Set playback volume                             |
| pause                   | -                 | void    | Pause playback                                  |
| resume                  | -                 | void    | Resume playback                                 |
| seekTo                  | long              | void    | Fast forward to the specified progress playback |
| isSeeking               | -                 | boolean | Is it fast forwarding?                          |
| getVideoCurrentPosition | -                 | long    | Get the current playback progress of the video  |
| getVideoTotalDuration   | -                 | long    | Get the total length of the video.              |

### Switch Rendering Mode

> Note: If you want to switch between tiling mode and other modes, you must first restart the player.

If the rendering mode you are using is `RENDER_MODE_AUTO`, you can switch between the following modes.

```java
// Switch to normal mode
mVideoPlayerView.switchNormalMode();

// Switch to fisheye mode
mVideoPlayerView.switchFisheyeMode();

// Switch to perspective mode
mVideoPlayerView.switchPerspectiveMode();

// To switch to tiling mode, you need to restart the player
VideoParamsBuilder builder = new VideoParamsBuilder()；
builder.setRenderModelType(ImageParamsBuilder.RENDER_MODE_PLANE_STITCH);
builder.setScreenRatio(2, 1);
mImagePlayerView.prepare(mWorkWrapper, builder);
mImagePlayerView.play();
```

### Field of View Distance

You can customize the viewing angle and distance according to your preferences.

> Fov：Angle range 0~180
>
> Distance：line of sight range  0~∞

```java
mVideoPlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### Set Listener

You can set the *`PlayerViewListener`* to observe changes in the player state.

```Java
mVideoPlayerView.setPlayerViewListener(new PlayerViewListener() {
    @Override
    public void onLoadingStatusChanged(boolean isLoading) {
        // Player loading status change
        //isLoading：whether loading
    }

    @Overrid
    epublic void onLoadingFinish() {
        // Player loading complete
    }

    @Override
    public void onFail(int errorCode, String errorMsg) {
        // if GPU not support, errorCode is -10003 or -10005 or -13020
        // Player error
    }
    
    @Override
    public void onFirstFrameRender() {
        // First frame rendering successfully
    }
});
```

You can set *`VideoStatusListener`* to observe changes in video status.

```java
mVideoPlayerView.setVideoStatusListener(new VideoStatusListener() {
    @Override
    public void onProgressChanged(long position, long length) {
        // Play progress callback
        // position：played length
        // length：Total length of the video
    }

    @Override
    public void onPlayStateChanged(boolean isPlaying) {
        // Video playback status callback
        // isPlaying：whether it is playing
    }

    @Override
    public void onSeekComplete() {
        // Fast forward completed
    }

    @Override
    public void onCompletion() {
        // Video playback finished
    }
});
```

### Gesture Operation

You can set *`PlayerGestureListener`* to observe gesture actions.

```typescript
mVideoPlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // User finger press event
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // User finger click event, similar to View.OnClickListener.OnClick(v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // User finger lift event
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // User long press event
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // User two-finger zoom event
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // Rebound animation when scaling to maximum or minimum
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // Zoom rebound animation ends
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // User finger swipe event
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // Quick slide animation
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // Quick swipe animation ends
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## Export

### Export Parameters

If you want to export images, you need to first understand the *`ExportImageParamsBuilder`* image export parameters.

```java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
    // (Must) Export file path
    .setTargetPath(String path)
    // (Optional) Set the width of the exported image, the default value is obtained from WorkWapper.
    .setWidth(int width)
    // (Optional) Set the height of the exported image, the default value is obtained from WorkWapper.
    .setHeight(int height)
    // (Optional) Set export mode, default is'PANORAMA'
    // ExportMode.PANORAMA: Export panoramic images
    // ExportMode.SPHERE: Export flat thumbnail
    .setExportMode(ExportUtils.ExportMode mode)
    // (Optional) Whether to enable dynamic stitching, default is true.
    .setDynamicStitch(boolean dynamicStitch)
    // (Optional) Set the stabilization type, default is InstaStabType.STAB_TYPE_OFF.
    .setStabType(@InstaStabType.StabType int stabType)
    // (Optional) Eliminate color difference in image stitching, default is false
    .setImageFusion(boolean imageFusion)
    // (Optional) Remove purple fringing. Default value is false. Requires adding an algorithm file.
    .setDePurpleFilterOn(boolean isDePurpleFilterOn)
    // (Optional) Set the lens guards type, default is OffsetType. ORIGINAL.
    .setOffsetType(OffsetType offsetType)
    // (Optional) Set as HDR.jpg generated by splicing, default is null
    .setUrlForExport(String url)
    // (Optional) Set to use software decoder, default is false
    .setUseSoftwareDecoder(boolean useSoftwareDecoder)
    // (Optional) Set the camera angle. It is recommended to use it when exporting thumbnails.
    // The currently displayed angle parameter can be obtained from'PlayerView.getXXX () '.
    .setFov(float fov)
    .setDistance(float distance)
    .setYaw(float yaw)
    .setPitch(float pitch)
    // (Optional) Cache folder, default is getCacheDir () + "/ work_thumbnail "
    .setCacheWorkThumbnailRootPath(String path)
    // (Optional) Cache folder, default is get CacheD ir () + "/ stabilizer "
    .setStabilizerCacheRootPath(String path)
    // (Optional) Cache folder, default is getCacheDir () + "/ cut_scene "
    .setCacheCutSceneRootPath(String path);
```

### Export Panoramic Images

Use images to export parameters:

```Java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024);
int exportId = ExportUtils.exportImage(WorkWrapper, builder, new IExportCallback() {
            @Override
            public void onSuccess() {       
                // Export successful        
            }

            @Override
            public void onFail() {
                // Export failed
            }

            @Override
            public void onCancel() {
                // Cancel export
            }

            @Override
            public void onProgress(float progress) {
                // Export progress
            }
        });
```

### Export Image thumbnail

Use images to export parameters:

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
                // Export successful     
            }

            @Override
            public void onFail() {
                // Export faild
            }

            @Override
            public void onCancel() {
                // Cancel export
            }

            @Override
            public void onProgress(float progress) {
                // Export progress
            }
        });
```

## Export Video

### Export Video Parameters

To export a video, you first need to understand the video export parameters *`ExportVideoParamsBuilder`* 。

```java
ExportVideoParamsBuilder builder = new ExportVideoParamsBuilder()
    // (Must) Export file path
    .setTargetPath(String path)
    // (Optional)Set the width of the exported video. It must be a power of 2, and the default value is obtained from WorkWapper.
    .setWidth(int width)
    // (Optional)Set the height of the exported video. It must be a power of 2, and the default value is obtained from WorkWapper.
    .setHeight(int height)
    // (Optional)Set the bitrate for exporting videos, with the default value being obtained from WorkWapper.
    .setBitrate(int bitrate)
    // (Optional)Set the fps for exporting videos, with the default value being obtained from WorkWapper.
    .setFps(int fps)
    // (Optional) Set export mode, default is'PANORAMA'
    // ExportMode.PANORAMA: Export panoramic images
    // ExportMode.SPHERE: Export flat thumbnail
    .setExportMode(ExportUtils.ExportMode mode)
    // (Optional) Remove purple fringing. Default value is false. Requires adding an algorithm file.
    .setDePurpleFilterOn(boolean isDePurpleFilterOn)
    // (Optional) Enable noise reduction (Noise reduction is enabled by default for exported images; the noise reduction toggle is only supported for exported videos)
    // The ExportUtils.supportDenoise(WorkWrapper) method must be called in advance to determine whether denoising is supported.
    // If noise reduction is supported, the setting is effective; otherwise, the setting is ineffective.
    .setDenoise(true)
    // (Optional) Eliminate color difference in image stitching, default is false
    .setImageFusion(boolean imageFusion)
    // (Optional) Set the lens guards type, default is OffsetType. ORIGINAL.
    .setOffsetType(OffsetType offsetType)
    // (Optional) Whether to enable dynamic stitching, default is true.
    .setDynamicStitch(boolean dynamicStitch)
    // (Optional) Set the stabilization type, default is InstaStabType.STAB_TYPE_OFF.
    .setStabType(@InstaStabType.StabType int stabType)
    // (Optional) Set to use software decoder, default is false
    .setUseSoftwareDecoder(boolean useSoftwareDecoder)
    // (Optional) Cache folder, default is getCacheDir () + "/ work_thumbnail"
    .setCacheWorkThumbnailRootPath(String path)
    // (Optional) Cache folder, default is getCacheDir () + "/ cut_scene"
    .setCacheCutSceneRootPath(String path)
    // (Optional) Set the field of view
    .setFov(float fov)
    // (Optional) Set the view distance
    .setDistance(float distance)
    // (Optional) Set the Euler angle - pitch angle
    .setPitch(float pitch)
    // (Optional) Set the Euler angle-yaw angle
    .setYaw(float yaw)
    // (Optional) Set the Euler angle-roll angle
    .setRoll(float roll)；
```

### Export Panoramic Video

Use video export parameters.

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
                // Export successful        
            }

            @Override
            public void onFail() {
                // Export failed
            }

            @Override
            public void onCancel() {
                // Cancel export
            }

            @Override
            public void onProgress(float progress) {
                // Export progress
            }
        });
```

### Export Video Thumbnail

Since the exported result is an image, use the image export parameters.

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
                // Export successful        
            }

            @Override
            public void onFail() {
                // Export failed
            }

            @Override
            public void onCancel() {
                // Cancel export
            }

            @Override
            public void onProgress(float progress) {
                // export progress
            }
        });
```

### Stop Exporting

```Java
ExportUtils.stopExport(exportId);
```

## Others

### Generate HDR Images

If you have a `WorkWrapper` with HDR images, you can generate them into an image file using HDR stitching.

> Note: This is a time-consuming operation that needs to be handled in a child thread.

```Java
boolean isSuccessful = StitchUtils.generateHDR(WorkWrapper workWrapper, String hdrOutputPath);
```

After the successful call of `generateHDR`, outputPath can be played as a proxy or set as an export URL.

```Java
// Set as playback agent
ImageParamsBuilder builder = new ImageParamsBuilder()
        // If HDR splicing is successful, set it as a playback proxy
        .setUrlForPlay(isSuccessful ? hdrOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();

// Set as export url
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        // If HDR splicing is successful, set it as export url
        .setUrlForExport(hdrOutputPath);
```

### Generate PureShot images

If you have a `WorkWrapper` of a PureShot image, you can generate it into an image file by stitching it together with PureShot.

> Note: This is a time-consuming operation that needs to be handled in a child thread.

```Java
boolean isSuccessful = StitchUtils.generatePureShot(WorkWrapper workWrapper, String pureshotOutputPath, String algoFolderPath);
```

> Note: Generating PureShot images requires algorithm model files. Please request the algorithm model files corresponding to your SDK version from our technical support team.

algoFolderPath is the directory for algorithm model files, which can be either LocalStoragePath or AssetsRelativePath. The SDK will load these files as needed.

After the successful call to `generatePureShot`, outputPath can be played as a proxy or set as an export URL.

```Java
// Set as playback agent
ImageParamsBuilder builder = new ImageParamsBuilder()
        // If PureShot splicing is successful, set it as a playback proxy
        .setUrlForPlay(isSuccessful ? pureshotOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();

// Set as export url
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        // If PureShot splicing is successful, set it as the export url
        .setUrlForExport(pureshotOutputPath);
```

## Algorithm File

Certain functions require algorithm files. These files can be found in the assets folder within the demo directory. Ensure the file paths match those in the demo.

Below is a table mapping function of their corresponding required algorithms:

| Function            | Model File                                                   | Path Description                                             |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Defringe            | defringe_lr_dynamic_e28c1212.ins defringe_hr_dynamic_7b56e80f.ins | DFixed path: file:///android_assets/insta360/model/          |
| Color Plus          | lut_7a79b17a_32.ins                                          | Fixed path: file:///android_assets/insta360/model/           |
| PureShot Generation | Pmdenoise_raw_plus.insraw_denoising_abeda57d.ins283fish.ilp283wide.ilp577.ilp586.ilpX3.ilp | Developer custom path:Passed in by the StitchUtils.generatePureShot function |

## Error Code

### Picture/Video Export Error Code

| Error code | Explanation                                      |
| ---------- | ------------------------------------------------ |
| -31000     | Exporting in progress                            |
| -31010     | Stop exporting                                   |
| -31011     | Failed to load stabilization data                |
| -31012     | *EXPORT_TERMINATED_BY_WORK_REFERENCE_RELEASABLE* |
| -31016     | *EXPORT_POST_EXECUTE_FAILED*                     |
| -31017     | Original thumbnail LRV not found                 |
| -31018     | Original thumbnail LRV loading failed            |
| -31019     | *EXPORT_VIDEO_TO_IMAGE_SEQUENCE_CANCEL*          |
| -13016     | Export timeout                                   |
| -13017     | Cancel export                                    |
| -13018     | *SHADOW_EXPORT_INIT_SHADOW_CLONE_FAIL*           |
| -13019     | *SHADOW_EXPORT_PREPARE_EXPORT_INFO_FAIL*         |
| -13021     | Failed to enable hardware encoding               |
| -13022     | *SHADOW_EXPORT_FAIL_TARGET_TIME_NULL*            |
| -13023     | *SHADOW_EXPORT_PREPRARE_FRAME_EXPORTER_FAIL*     |
| -13024     | *SHADOW_EXPORT_INPUT_TARGET_FAIL*                |
| -13025     | *STOP_MOTION_SELECT_POSE_CLIP_NULL*              |
| -13026     | *STOP_MOTION_SELECT_CLIP_NOT_MATCH*              |
| -13027     | *STOP_MOTION_SELECT_LOAD_STABILIZER_FAIL*        |
| -13028     | *STOP_MOTION_SELECT_KEY_NULL*                    |
| -13029     | *SHADOW_EXPORT_TRACK_TARGET_EMPTY*               |
| -13031     | Frame exporter preparation failed                |
| -13033     | Reverse video parsing failed                     |
| -13034     | *Dual stream frame format is different*          |
| -13035     | Thumbnail Stabilization Error                    |
| -13036     | Video frame rate equals 0                        |
| -13038     | Video width equals zero                          |
| -13039     | Video height equals zero                         |
| -13040     | The denoise model equals null.                   |
| -13041     | Video source frame rate equals 0                 |
| -13042     | Video source duration equals 0                   |
| -13043     | Image Denoise File Transfer Error                |
| -13034     | *Dual stream frame format is different*          |
| -13044     | Failed to load AAA parameters                    |
| -13045     | Failed to create denoise data                    |
| -13046     | Image width equals 0                             |
| -13047     | Image height equals 0                            |

# Frequently Asked Questions:

## Q1: What should I do if I cannot access the Internet when connecting to the camera?

> This typically refers to establishing a connection with the camera via Wi-Fi. The app communicates with the camera through two methods: socket and HTTP. Below are the corresponding functions for each communication method:
>
> - Socket：Preview stream, read camera parameters, set camera parameters, read capture parameters, set capture parameters, etc. // TODO Supported features to be completed
> - Http  Initialize the supported list, download camera files. // TODO Supported features to be completed
>
> The root cause of this issue stems from calling `ConnectivityManager#bindProcessToNetwork(network)` when establishing a socket connection with the camera, which binds the current process to the camera's Wi-Fi network.
>
> Therefore, after establishing a Wi-Fi connection, calling `ConnectivityManager#bindProcessToNetwork(4GNetwork)` to bind to an available network (such as 4G) resolves the inability to access the internet. Since the socket connection is already established, unbinding still allows continued use of socket-enabled features.
>
> However, this approach introduces a problem. Because the current process is no longer bound to the camera's Wi-Fi, using features dependent on HTTP communication will result in the following error:
>
> ![img](https://arashivision.feishu.cn/space/api/box/stream/download/asynccode/?code=NTVkNWQxMmYzZTQyZjkzOGY4OGEzMzZkYjhkMzExNTlfa05pMlBLWHc2UnBTajV0R2cwanZsbERDeXkwdVV5b2ZfVG9rZW46Vm9Yb2JqSGdFb2IzV1Z4Qkt6QWNReHV0bktoXzE3NjA1ODI4NDg6MTc2MDU4NjQ0OF9WNA)
>
> There are two solutions to this problem:
>
> 1. When using features related to HTTP communication, call the `bindProcessToNetwork(camera)` method to bind to the camera's Wi-Fi. Upon completing the relevant functionality, call `bindProcessToNetwork(4g)` to switch back to the available 4G network.
>    1. Pros: Simple implementation
>    2. Cons: Will temporarily disconnect from the Internet.
> 2. Launch a new process named download. Call the bindProcessToNetwork(camera) method to bind the download process to the camera's Wi-Fi network. All HTTP communication-related functions are executed within the download process.
>    1. Pros: Perfectly solves the problem of being unable to access the Internet.
>    2. Cons: Complex implementation

## Q2: What is the data format in the VideoData object?

> H.264 format, containing both PPS and SPS frame data.

## Q3：How to resolve C++_shared conflicts?

> Upgrade to version 1.8.1_build_05 or later

## Q4：How to handle the issue regarding OK Go?

> *implementation*("com.github.jeasonlzy.okhttp-OkGo:okgo:v3.0.4")
>
> *implementation*("com.arashivision.sdk:sdkcamera:1.8.0_build_11")**{**
>
> ​    *exclude*("com.github.jeasonlzy","okhttp-OkGo")
>
> **}**

## Q5：Connection error -2110203 issue?

> Socket read failure typically occurs when Wi-Fi is disabled by the system. Manually reconnecting to Wi-Fi via the system settings page can reduce the likelihood of automatic disconnection by the system.

## Q6：Regarding the issue of some Huawei phones displaying a black screen during playback?

> The photo width is 16386 pixels, exceeding the 8192-pixel rendering limit on some Huawei phones.
