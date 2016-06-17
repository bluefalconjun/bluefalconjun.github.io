
###[**Media**](http://source.android.com/devices/media.html)

![Media](http://source.android.com/devices/images/ape_fwk_hal_media.png)

-----
**Android**从**native**层提供了一套命名为**stagefright**的媒体处理引擎. 它包含有部分原生编入的支持常见格式的软件解码支持.  

它的功能包含同**OpenMax**编解码器整合的**audio/video**播放, 会话管理, 时间同步, 传输管理协议, **DRM**等.

在扩展功能上, **stagefright**支持同硬件编解码器的整合.  在实际产品中, 没有一个明确的**HAL**来提供**codec**的实现, 但是供应商必须实现基于硬件编解码器的**OpenMax IL(Integartion Layer)**组件来提供媒体编码/解码的硬件通路.

-----
> **Architecture**

-----
下图描述媒体应用同**native**多媒体**framework**交互的过程:

![Android media architecture](http://source.android.com/devices/images/ape_fwk_media.png)

-----

 - **Application Framework**:
**app**的代码存在于**app framework**层, 它通过**android.media**的API来和多媒体硬件互动.

 - **Binder IPC**:
**Binder IPC**代理帮助多进程进行通讯, 它的代码位于[**frameworks/av/media/libmedia**](https://android.googlesource.com/platform/frameworks/av/+/android-5.1.1_r18/media/libmedia/) 目录, 并且以字符"I"开头. 

 - **Native Multimedia Framework**:
在实现层, **stagefright**引擎缺省提供了软件编解码器, 供应商通过**OpenMax IL**层加入硬件编解码器支持. 
参见[**frameworks/av/media**](https://android.googlesource.com/platform/frameworks/av/+/android-5.1.1_r18/media/).

 - **OpenMAX Integration Layer (IL)**:
**OpenMax IL**层给**stagefright**提供了标准的方式来组织和使用特殊的基于硬件的多媒体编解码器. 这里称之为组件(**Component**). 
实现方式是提供一个名为**libstagefrighthw.so**的插件. 它将硬件编解码器同**stagefright**连接起来. 它的实现必须符合**OpenMax IL**标准.

-----
> **Implementing Custom Codecs**

-----
要自定义编解码器, 参考[**hardware/ti/omap4xxx/domx/**](https://android.googlesource.com/platform/hardware/ti/omap4xxx/+/android-5.1.1_r18/domx/) 来创建组件.

 参考[**hardware/ti/omap4xx/libstagefrighthw**](https://android.googlesource.com/platform/hardware/ti/omap4xxx/+/android-5.1.1_r18/libstagefrighthw/) 来查看为Galaxy Nexus使用的omx插件..

-----
增加自己的编解码器具体方式:

 1. 按照**OpenMax IL**组件标准创建自己的组件.
    接口定义在[**frameworks/native/include/media/OpenMAX/OMX_Component.h**](https://android.googlesource.com/platform/frameworks/native/+/android-5.1.1_r18/include/media/openmax/OMX_Component.h).
    同时可以查看[**OpenMAX**](http://www.khronos.org/openmax/)网站来参考.

 2. 创建**OpenMax**插件来将组件同**stagefright**服务链接在一起. 查看[**frameworks/native/include/media/hardware/OMXPluginBase.h**](https://android.googlesource.com/platform/frameworks/native/+/android-5.1.1_r18/include/media/hardware/OMXPluginBase.h) 和 [**HardwareAPI.h**](https://android.googlesource.com/platform/frameworks/native/+/android-5.1.1_r18/include/media/hardware/HardwareAPI.h) 来参考插件的接口定义.

 3. 将插件编译命名为**libstagefrighthw.so**的动态链接库并将该模块加入到系统的模块列表中:

----------
    LOCAL_MODULE := libstagefrighthw
    PRODUCT_PACKAGES += \
      libstagefrighthw \
      ...

-----
> **Exposing Codecs to the Framework**

-----
**Stagefright**服务通过解析**system/etc/media_codecs.xml**和**system/etc/media_profiles.xml** 来将当前平台中支持的编解码器和支持级别提供给**app**开发者. 

这一过程是通过**android.media.MediaCodecList** 和**android.media.MediaCodecProfile** 类来完成的. 

这需要在**[device/company_name/device_name/]** 目录下创建这两个文件, 并在编译安装时拷贝到平台的对应目录下:

    PRODUCT_COPY_FILES += \
      device/samsung/tuna/media_profiles.xml:system/etc/media_profiles.xml \
      device/samsung/tuna/media_codecs.xml:system/etc/media_codecs.xml \


查看[**device/samsung/tuna/media_codecs.xml**](https://android.googlesource.com/device/samsung/tuna/+/android-4.3.1_r1/media_codecs.xml) 和 **media_profiles.xml** 文件来查看完成参考.

-----