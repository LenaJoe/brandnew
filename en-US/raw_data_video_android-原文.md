
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

Before using the raw data feature, make sure you have completed the basic real-time audio and video features in your project, see [one-on-one ](call../../CN/interactive%20voice /start_call_androidorinteractive)[](live../../CN/interactive % 20voice /start_Live_android )for details.

Refer to the following steps to implement the original video data feature in your project:

1. The `registervideoframeobserver `method is called to` register` the video observer before joining the channel, and an `ivideoframeobserver` class is implemented in the method.
2. After successful registration, the SDK sends the captured raw video data via the `oncapturevideoframe`, onpreencodeoframe``, or `onrendervideoframe` callbacks as each video frame is captured.
3. After the user gets the video data, the user processes the video data by himself according to the needs of the scene. And then the processed video data is sent to the SDK through the callback.

### API call timing

The following figure shows the API call timing using raw video data:

![](Https://web-cdn.Agora.io /docs-files /1577090428042)


### Sample code

You can implement the original video data feature in your project by referring to the following example code snippet against the API timing diagram:

```c++
#Include <JNI.h>
#Include <Android /log.h>
#Include <CString>

#Include "Agora /iagorortcengine.h"
#Include "Agora /iagoramediaengine.h"

#Include "video_preprocessing_plugin_JNI.h"

Class agoravideoframeobserver: public Agora:: media:: ivideoframeobserver
{
Public:
    //Get video frames collected by local camera
    Virtual bool oncapturevideoframe (VideoFrame &VideoFrame) override
    {
        Int width=VideoFrame.width;
        Int height=VideoFrame.height;

        Memset (VideoFrame.ubuffer, 128, VideoFrame.ustride*height /2)
        Memset (VideoFrame.vbuffer, 128, VideoFrame.vstride*height /2)

        Return true;
    Oh.

    //Get video frames sent by remote users
    Virtual bool onrendervideoframe (unsigned int uid, VideoFrame &VideoFrame) override
    {
        Return true;
    Oh.
	//Get video frame before local video encoding
    Virtual bool onpreencodevideoframe (VideoFrame &VideoFrame) override
    {
        Return true;
    Oh.
;

Class ivideoframeobserver
{
     Public:
         Enum video_frame_type {
         Frame_type_YUV420=0,//the video frame format is YUV 420
         ;
     Struct VideoFrame {
         Video_frame_type type;
         Int width;//width of video frame
         Int height;//height of video frame
         Int ystride;
         Int ustride;
         Int vstride;
         Void*ybuffer;
         Void*ubuffer;
         Void*vbuffer;
         Int rotation;
         Int64_t rendertimems;
         ;
     Public:
         Virtual bool oncapturevideoframe (VideoFrame &VideoFrame)=0;
         Virtual bool onrendervideoframe (unsigned int uid, VideoFrame &VideoFrame)=0;
		 Virtual bool onpreencodevideoframe (VideoFrame &VideoFrame){return true;}
;
```

At the same time, we offer an open source[ video encrypthttps]( at GitHub: )You can download it at, or refer to [`iagoramediaengine.hhttps`](:)

### API reference

- [`Registervideoframeobserverhttps`](://docs.Agora.io/interactive % 20broadcast /API % 20reference /cpp/classagora_1_1media_1_I_media_engine.html #A5EEE4DFD1FD46E4a865FEBA163F3c5de)
- [`Oncapturevideoframehttps`](://docs.Agora.io/interactive % 20broadcast /API % 20reference /cpp/classagora_1_1media_1_I_video_frame_observer.html #A915C673AEC879DCC2b08246BB2FCF49a)
- [`Onrendervideoframehttps`](://docs.Agora.io/CN/interactive % 20broadcast /API % 20reference /cpp/classagora_1_1media_1_I_video_frame_observer.html #A966ed2459B6887C52112af638BC27c 14)
- [`Onpreencodevideoframehttps`](://docs.Agora.io/CN/interactive % 20broadcast /API % 20reference /cpp/classagora_1_1media_1_I_video_frame_observer.html #A2BE41CDDE19FCC0f365d4EB14a963E1c)

## Development considerations

The raw data interface used in this article is the C++ interface. If you're developing on Android, please refer to the following steps to register the video data Viewer using the SDK library's JNI and Plug-in Manager.

1. Before joining a channel, create a shared library Project. The project must be prefixed with `libapm-`and suffixed with `so`, such as `libapm-encryption.So`.
2. Upon successful creation, rtcengine automatically loads the `libapm-encryption.So` file as a statistics directory plug-in.
3. After the SDK destroys the boot, call the `unloadagorartcengineplugin` interface, and rtcengine will automatically uninstall the statistics directory plug-in.

```java
Static agoravideoframeobserver s_videoframeobserver;
Static Agora:: RTC:: irtcengine*rtcengine=null;


#Ifdef __cplusplus
Extern "C"{
#Endif

Int __attribute__((profitability (" default "))) loadagorartcengineplugin (Agora:: RTC:: irtcengine*engine)
{
    __Android_log_print (Android_log_error," plugin,"" plugin loadagorartcengineplugin ");
    Rtcengine=engine;
    Return 0;
Oh.

Void __attribute__((profitability (" default "))) unloadagorartcengineplugin (Agora:: RTC:: irtcengine*engine)
{
    __Android_log_print (Android_log_error," plugin,"" plugin unloadagorartcengineplugin ");
    Rtcengine=null;
Oh.

Jniexport void jnicall Java_io_Agora_propeller_preprocessing_videopreprocessing_enablepreprocessing
  (JNIEnv*env, jobject obj, jboolean enable)
{
    If (! Rtcengine)
        Return;
    Agora:: util:: autoptf <Agora:: media:: imediaengine> mediaengine;
    Mediaengine.QueryInterface (rtcengine, Agora:: Agora_IID_media_engine);
    If (mediaengine){
        If (enable){
            Mediaengine-> registervideoframeobserver (&S_videoframeobserver);
        } Else {
            Mediaengine-> registervideoframeobserver (null);
        Oh.
    Oh.
Oh.

#Ifdef __cplusplus
Oh.
#Endif
```

## Related documents

If you also want to implement raw audio data )functionality in your project, please refer to [](raw audio data../../CN/interactive % 20audio /raw_data_Audio_android.
