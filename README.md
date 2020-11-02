# 1 描述
URTCAndroid 是UCloud推出的一款适用于android平台的实时音视频 SDK，支持android5.0及以上系统，提供了音视频通话基础功能，提供灵活的接口，支持高度定制以及二次开发。

# 2 功能列表

## 2.1 基本功能
* 基本的音视频通话功能	
* 支持内置音视频采集的常见功能	
* 支持静音关闭视频功能	
* 支持视频尺寸的配置(180P - 720P)	
* 支持自动重连	
* 支持丰富的消息回调	
* 支持纯音频互动	
* 支持视频的大小窗口切换	
* 支持获取视频房间统计信息（帧率、码率、丢包率等）	
* 支持编码镜像功能		
* 支持屏幕录制功能
* 支持自动手动订阅 自动手动发布
* 支持权限（上行/下行/全部）控制
* 支持音量提示
* 支持获取sdk版本
* 支持大班小班切换功能

## 2.2 增值功能
* 终端智能测试（摄像头、麦克风、网络、播放器）
* 视频录制/视频存储
* 视频水印
* 视频直播CDN分发
* 美颜
* 贴纸/滤镜/哈哈镜
* 背景分割
* 手势
* 虚拟形象
* 变声
* 自定义的外部输入和输出扩展接口

## 2.3 文档地址
* https://docs.ucloud.cn/video/urtc/index.html 

# 3 方案优势

## 3.1 方案架构
![](http://urtcwater.cn-bj.ufileos.com/%E5%9B%BE%E7%89%871.png)

## 3.2 方案优势
* 利用边缘节点就近接入
* 可用性99.99%
* 智能链路调度
* 自有骨干专线+Internet传输优化
* 数据报文AES加密传输
* 全API开放调度能力
* 端到端链路质量探测
* 多点接入线路容灾
* 抗丢包视频30% 音频70%
* 国内平均延时低于75ms 国际平均延时低于200ms

# 4 应用场景

## 4.1 主播连麦

支持主播之间连麦一起直播，带来与传统单向直播不一样的体验
48KHz 采样率、全频带编解码以及对音乐场景的特殊优化保证观众可以听到最优质的声音

## 4.2 视频会议
小范围实时音视频互动，提供多种视频通话布局模板，更提供自定义布局方式，保证会议发言者互相之间的实时性，提升普通观众的观看体验

## 4.3 泛文娱
### 4.3.1 一对一社交
客户可以利用UCloud实时音视频云实现 QQ、微信、陌陌等社交应用的一对一视频互动
### 4.3.2 狼人杀游戏
支持15人视频通话，玩家可在游戏中选择只开启语音或同时开启音视频

## 4.4 在线教育
支持自动和手动发布订阅视频流，方便实现课堂虚拟分组概念，同时支持根据角色设置流权限，保证课程秩序

## 4.5 在线客服
线上开展音视频对话，对客户的资信情况进行审核，方便金融科技企业实现用户在线签约、视频开户验证以及呼叫中心等功能
提供云端存储空间及海量数据的处理能力，提供高可用的技术和高稳定的平台

# 5 快速使用
## 5.1 初始化
### 5.1.1 引擎环境初始化
主要配置android context  sdkmode 日志等级以及AppID ，测试用的APP_KEY,可由Android客户端自行生成Token
~~~
public class UCloudRtcApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        UCloudRtcSdkEnv.initEnv(getApplicationContext(), this);
        UCloudRtcSdkEnv.setLogLevel(UCloudRtcSdkLogLevel.UCloudRtc_SDK_LogLevelInfo) ;
        UCloudRtcSdkEnv.setSdkMode(UCloudRtcSdkMode.RTC_SDK_MODE_TRIVAL);
        UCloudRtcSdkEnv.setTokenSeckey(CommonUtils.APP_KEY);
        WindowManager windowManager = (WindowManager) getSystemService(Context.WINDOW_SERVICE);
        DisplayMetrics outMetrics = new DisplayMetrics();
        windowManager.getDefaultDisplay().getMetrics(outMetrics);
        CommonUtils.mItemWidth = outMetrics.widthPixels / 3;
        CommonUtils.mItemHeight = CommonUtils.mItemWidth;
    }
}
~~~
Note：在正式环境中，setSdkMode的参数需改为UCloudRtcSdkMode.UCLOUD_RTC_SDK_MODE_NORMAL，并且在后台服务器中部署Token服务，客户端加入房间之前，向后台服务器申请Token ，以保证Token 的安全性。服务器部署Token具体步骤参考地址：https://docs.ucloud.cn/urtc/sdk/token


### 5.1.2 继承实现UCloudRtcSdkEventListener 实现事件处理
~~~
UCloudRtcSdkEventListener eventListener = new UCloudRtcSdkEventListener() {
    @Override
    public void onServerDisconnect() {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                ToastUtils.shortShow(RoomActivity.this, " 服务器已断开");
                stopTimeShow();
                onMediaServerDisconnect() ;
            }
        });
    }

    @Override
    public void onJoinRoomResult(int code, String msg, String roomid) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                if (code == 0) {
                    ToastUtils.shortShow(RoomActivity.this, " 加入房间成功");
                    startTimeShow();
                }else {
                    ToastUtils.shortShow(RoomActivity.this, " 加入房间失败 "+
                            code +" errmsg "+ msg);
                    Intent intent = new Intent(RoomActivity.this, ConnectActivity.class);
                    onMediaServerDisconnect() ;
                    startActivity(intent) ;
                    finish();
                }

            }
        });
    }
~~~
### 5.1.3 获取SDK 引擎 并进行基础配置
~~~
sdkEngine = UCloudRtcSdkEngine.createEngnine(eventListener) ;
sdkEngine.setAudioOnlyMode(true) ; // 设置纯音频模式
sdkEngine.configLocalCameraPublish(false) ; // 设置摄像头是否发布
sdkEngine.configLocalAudioPublish(true) ; // 设置音频是否发布，用于让sdk判断自动发布的媒体类型
sdkEngine.configLocalScreenPublish(false) ; // 设置桌面是否发布，作用同上
sdkEngine.setStreamRole(UCloudRtcSdkStreamRole.UCloudRtc_SDK_STREAM_ROLE_BOTH);// 流权限
sdkEngine.setAutoPublish(true) ; // 是否自动发布
sdkEngine.setAutoSubscribe(true) ;// 是否自动订阅
sdkEngine.setVideoProfile(UCloudRtcSdkVideoProfile.matchValue(mVideoProfile)) ;// 摄像头输出等级
~~~

## 5.2 加入房间
~~~
UCloudRtcSdkAuthInfo info = new UCloudRtcSdkAuthInfo();
info.setAppId(mAppid);
info.setToken(mRoomToken);
info.setRoomId(mRoomid);
info.setUId(mUserid);
Log.d(TAG, " roomtoken = " + mRoomToken);
sdkEngine.joinChannel(info);
~~~

## 5.3 自动/手动发布
如果配置了自动发布无需调用发布视频接口，SDK会在用户成功加入房间后自动发布，只需要监听事件调用渲染接口即可。
如果配置了手动发布需要调用sdkEngine引擎的publish接口 配置手动/自动发布
### 5.3.1 媒体发布类型
现在的类型包括两大类，需要传入publish接口的mtype,hasvideo,hasaudio参数各不相同，混合类型是单一类型的组合，具体代码可参阅UCloudRtcdemo的RoomActvity中的处理。
1.	混合类型 音频+屏幕，视频+屏幕

2.	单一类型 音频 （mtype:UCloudRtc_sdk_media_type_video,hasvideo:false,hasaudio:true），视频（mtype:UCloudRtc_sdk_media_type_video,hasvideo:true,hasaudio:true），屏幕 （mtype:UCloudRtc_sdk_media_type_screen,hasvideo:true,hasaudio:false）
### 5.3.2 渲染媒体流
在onLocalPublish 回调成功后，再函数中可以调用视频渲染
~~~
localrenderview.setBackgroundColor(Color.TRANSPARENT);
sdkEngine.startPreview(info.getmMediatype(), localrenderview);
//不想渲染时可以调用停止渲染接口
sdkEngine.stopPreview(UCloudRtcSdkMediaType mediatype)
~~~
### 5.3.3 取消媒体发布流
~~~
sdkEngine.unPublish(UCloudRtcSdkMediaType mtype)
//回调事件
public void onLocalUnPublish(int code, String msg, UCloudRtcSdkStreamInfo info) 
~~~

## 5.4 自动/手动订阅
### 5.4.1 订阅媒体流
~~~
sdkEngine.subscribe(UCloudRtcSdkStreamInfo info)
//回调事件
public void onSubscribeResult(int code, String msg, UCloudRtcSdkStreamInfo info) 
~~~
### 5.4.2 取消媒体发布流
在onSubscribeResult回调成功后，再函数中可以调用视频渲染
~~~
sdkEngine. startRemoteView(UCloudRtcSdkStreamInfo info, UCloudRtcSdkSurfaceVideoView renderview)
//不想渲染时可以调用定制渲染接口
sdkEngine.stopPreview(UCloudRtcSdkMediaType mediatype)
~~~
### 5.4.3 取消订阅媒体流
~~~
sdkEngine. subscribe(UCloudRtcSdkStreamInfo info) 
//回调事件
public void onUnSubscribeResult(int code, String msg, UCloudRtcSdkStreamInfo info)
~~~

## 5.5 权限控制
权限分为发布，订阅，全部权限，全部权限包括了发布和订阅
~~~
//接口
public int setStreamRole(UCloudRtcSdkStreamRole role)
~~~
## 5.6 录像
### 5.6.1 录像开始
录像目前只支持摄像头录制，不支持桌面录制，region和bucket这两个参数默认用了ucloud自己的region和bucket，如果用自己的需要上ucloud控制台申请自己的录像存储空间，服务器会通过UCloudRtcSdkEventListener 的onRecordStart()接口作为回调返回录像开始结果。

需要特别注意的是，录像可以指定主界面是哪个用户，当非均衡模式的情况下，主界面是哪个用户，哪个用户就占据大窗口。同时，这里的用户可以是当前App中推流的用户，也可以是当前App中被订阅的用户，这个参数只要靠mainviewuid去实现，如果是上述第一种情况，可以不指定，sdk自动获取，如果是第二种，就需要App SDK使用者拿到当前订阅的用户id，用这个id去设置录像的mainviewuid

更多的录像的参数说明可以参照sdk文档以及https://github.com/UCloudDocs/urtc/blob/master/cloudRecord/RecordLaylout.md

~~~
//                如果主窗口是当前用户
UcloudRtcSdkRecordProfile recordProfile = UcloudRtcSdkRecordProfile.getInstance().assembleRecordBuilder()
                        .recordType(UcloudRtcSdkRecordProfile.RECORD_TYPE_VIDEO)
                        .mainViewMediaType(UCLOUD_RTC_SDK_MEDIA_TYPE_VIDEO.ordinal())
                        .VideoProfile(UCloudRtcSdkVideoProfile.UCLOUD_RTC_SDK_VIDEO_PROFILE_640_480.ordinal())
                        .Average(UcloudRtcSdkRecordProfile.RECORD_UNEVEN)
                        .WaterType(UcloudRtcSdkRecordProfile.RECORD_WATER_TYPE_IMG)
                        .WaterPosition(UcloudRtcSdkRecordProfile.RECORD_WATER_POS_LEFTTOP)
                        .WarterUrl("http://urtc-living-test.cn-bj.ufileos.com/test.png")
                        .Template(UcloudRtcSdkRecordProfile.RECORD_TEMPLET_9)
                        .build();
                sdkEngine.startRecord(recordProfile);
                //如果主窗口不是当前推流用户，而是被订阅的用户
//                UCloudRtcSdkStreamInfo uCloudRtcSdkStreamInfo = mVideoAdapter.getStreamInfo(0);
//                if(uCloudRtcSdkStreamInfo != null){
//                    UcloudRtcSdkRecordProfile recordProfile = UcloudRtcSdkRecordProfile.getInstance().assembleRecordBuilder()
//                            .recordType(UcloudRtcSdkRecordProfile.RECORD_TYPE_VIDEO)
//                            .mainViewUserId(uCloudRtcSdkStreamInfo.getUId())
//                            .mainViewMediaType(uCloudRtcSdkStreamInfo.getMediaType().ordinal())
//                            .VideoProfile(UCloudRtcSdkVideoProfile.UCLOUD_RTC_SDK_VIDEO_PROFILE_640_480.ordinal())
//                            .Average(UcloudRtcSdkRecordProfile.RECORD_UNEVEN)
//                            .WaterType(UcloudRtcSdkRecordProfile.RECORD_WATER_TYPE_IMG)
//                            .WaterPosition(UcloudRtcSdkRecordProfile.RECORD_WATER_POS_LEFTTOP)
//                            .WarterUrl("http://urtc-living-test.cn-bj.ufileos.com/test.png")
//                            .Template(UcloudRtcSdkRecordProfile.RECORD_TEMPLET_9)
//                            .build();
//                    sdkEngine.startRecord(recordProfile);
//                }
//UCloudRtcSdkEventListener
void onRecordStart(int code,String fileName);
~~~

### 5.6.2录像结束
服务器会通过UCloudRtcSdkEventListener 的onRecordStop()接口作为回调返回录像结束结果。
~~~
sdkEngine.stopRecord();
~~~
## 5.7 外部扩展输入与输出
sdk支持rgba系列数据（rgba，abgr，rgb565，）以及yuv420p的外部自定义输入，能够产出拉流的rgba,abgr的数据供使用者自行扩展使用，具体使用方式请参考demo内部的rgb转yuv接口使用说明.md 和 yuv转rgb接口使用说明.md
## 5.8 离开房间
~~~
sdkEngine.leaveChannel() ;
~~~

## 5.9 编译运行

