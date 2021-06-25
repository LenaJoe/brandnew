
---
Title: original video data
Description:
Platform: Android
UpdatedAt: Tue Apr 28 2020 04:48:08 GMT+0800 (CST)
---
# Raw video data
## Functional description

In the process of audio and video transmission, we can carry out pre-processing and post-processing on the collected audio and video data to obtain the desired playing effect.

For scenarios where you need to process your own audio and video data, Agora provides raw data functionality, allowing you to pre-process the data before sending it to the encoder, modifying the captured voice signal or video frame;

By providing `ivideoframeobserver` class, the original SDK realizes the function of collecting and modifying original video data.

## Implementation method

Before using the raw data feature, make sure you have completed the basic real-time audio and video features in the project, as detailed in [one-on-one calls ](../../cn/Interactive%20Broadcast/start_call_android.md)or [interactive live](../../cn/Interactive%20Broadcast/start_live_android.md).

Refer to the following steps to implement the original video data feature in your project:

1. The `registervideoframeobserver `method is called to` register` the video observer before joining the channel, and an `ivideoframeobserver` class is implemented in the method.
2. After successful registration, the SDK sends the captured raw video data via the `oncapturevideoframe`, onpreencodeoframe``, or `onrendervideoframe` callbacks as each video frame is captured.
3. After the user gets the video data, the user processes the video data by himself according to the needs of the scene. And then the processed video data is sent to the SDK through the callback.

### API call timing

The following figure shows the API call timing using raw video data:

![](https://web-cdn.agora.io/docs-files/1577090428042)


### Sample code

You can implement the original video data feature in your project by referring to the following example code snippet against the API timing diagram:

```c++
#include <jni.h>
#include <android/log.h>
#include <cstring>

#include "agora/IAgoraRtcEngine.h"
#include "agora/IAgoraMediaEngine.h"

#include "video_preprocessing_plugin_jni.h"

class AgoraVideoFrameObserver : public agora::media::IVideoFrameObserver
{
public:
    // 获取本地摄像头采集到的视频帧
    virtual bool onCaptureVideoFrame(VideoFrame& videoFrame) override
    {
        int width = videoFrame.width;
        int height = videoFrame.height;

        memset(videoFrame.uBuffer, 128, videoFrame.uStride * height / 2);
        memset(videoFrame.vBuffer, 128, videoFrame.vStride * height / 2);

        return true;
    }

    // 获取远端用户发送的视频帧
    virtual bool onRenderVideoFrame(unsigned int uid, VideoFrame& videoFrame) override
    {
        return true;
    }
	// 获取本地视频编码前的视频帧
    virtual bool onPreEncodeVideoFrame(VideoFrame& videoFrame) override
    {
        return true;
    }
};

class IVideoFrameObserver
{
     public:
         enum VIDEO_FRAME_TYPE {
         FRAME_TYPE_YUV420 = 0,  // 视频帧格式为 YUV 420
         };
     struct VideoFrame {
         VIDEO_FRAME_TYPE type;
         int width;  // 视频帧的宽
         int height;  // 视频帧的高
         int yStride;  // YUV 数据中的 Y 缓冲区的行跨度
         int uStride;  // YUV 数据中的 U 缓冲区的行跨度
         int vStride;  // YUV 数据中的 V 缓冲区的行跨度
         void* yBuffer;  // YUV 数据中的 Y 缓冲区的指针
         void* uBuffer;  // YUV 数据中 U 缓冲区的指针
         void* vBuffer;  // YUV 数据中 V 缓冲区的指针
         int rotation; // 该帧的旋转信息，可设为 0, 90, 180, 270
         int64_t renderTimeMs; // 该帧的时间戳
         };
     public:
         virtual bool onCaptureVideoFrame(VideoFrame& videoFrame) = 0;
         virtual bool onRenderVideoFrame(unsigned int uid, VideoFrame& videoFrame) = 0;
		 virtual bool onPreEncodeVideoFrame(VideoFrame& videoFrame) { return true; }
};
```

At the same time, we provide an open source[ video encrypt](https://github.com/AgoraIO/Advanced-Video/tree/master/Android/sample-video-encrypt) sample project at GitHub. You can download it at, or refer to the code in the [`iagoramediaengine.h`](https://github.com/AgoraIO/Advanced-Video/blob/master/Android/sample-video-encrypt/src/main/cpp/include/agora/IAgoraMediaEngine.h) file.

### API reference

- [`Registervideoframeobserver`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1media_1_1_i_media_engine.html#a5eee4dfd1fd46e4a865feba163f3c5de)
- [`Oncapturevideoframe`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1media_1_1_i_video_frame_observer.html#a915c673aec879dcc2b08246bb2fcf49a)
- [`Onrendervideoframe`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1media_1_1_i_video_frame_observer.html#a966ed2459b6887c52112af638bc27c14)
- [`Onpreencodevideoframe`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1media_1_1_i_video_frame_observer.html#a2be41cdde19fcc0f365d4eb14a963e1c)

## Development considerations

The raw data interface used in this article is the C++ interface. If you're developing on Android, please refer to the following steps to register the video data Viewer using the SDK library's JNI and Plug-in Manager.

1. Before joining a channel, create a shared library Project. The project must be prefixed with `libapm-`and suffixed with `so`, such as `libapm-encryption.So`.
2. Upon successful creation, rtcengine automatically loads the `libapm-encryption.So` file as a statistics directory plug-in.
3. After the SDK destroys the boot, call the `unloadagorartcengineplugin` interface, and rtcengine will automatically uninstall the statistics directory plug-in.

```java
static AgoraVideoFrameObserver s_videoFrameObserver;
static agora::rtc::IRtcEngine* rtcEngine = NULL;


#ifdef __cplusplus
extern "C" {
#endif

int __attribute__((visibility("default"))) loadAgoraRtcEnginePlugin(agora::rtc::IRtcEngine* engine)
{
    __android_log_print(ANDROID_LOG_ERROR, "plugin", "plugin loadAgoraRtcEnginePlugin");
    rtcEngine = engine;
    return 0;
}

void __attribute__((visibility("default"))) unloadAgoraRtcEnginePlugin(agora::rtc::IRtcEngine* engine)
{
    __android_log_print(ANDROID_LOG_ERROR, "plugin", "plugin unloadAgoraRtcEnginePlugin");
    rtcEngine = NULL;
}

JNIEXPORT void JNICALL Java_io_agora_propeller_preprocessing_VideoPreProcessing_enablePreProcessing
  (JNIEnv *env, jobject obj, jboolean enable)
{
    if (!rtcEngine)
        return;
    agora::util::AutoPtr<agora::media::IMediaEngine> mediaEngine;
    mediaEngine.queryInterface(rtcEngine, agora::AGORA_IID_MEDIA_ENGINE);
    if (mediaEngine) {
        if (enable) {
            mediaEngine->registerVideoFrameObserver(&s_videoFrameObserver);
        } else {
            mediaEngine->registerVideoFrameObserver(NULL);
        }
    }
}

#ifdef __cplusplus
}
#endif
```

## Related documents

If you also want to implement the original audio data feature in your project, refer to [the original audio data](../../cn/Interactive%20Broadcast/raw_data_audio_android.md).
