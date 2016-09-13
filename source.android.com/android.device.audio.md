
###[**Audio**](http://source.android.com/devices/audio/index.html)
![Audio Hal](http://source.android.com/devices/audio/images/ape_fwk_hal_audio.png)

-----
Android的audio硬件抽象层(**`HAL`**)负责将[**`android.media`**](http://developer.android.com/reference/android/media/package-summary.html)中的上层,音频相关的framework APIs同底层的audio驱动和硬件连接起来. 该章节包含有实现的指导和提高性能的参考提示.

-----
> **Audio Architecture**

-----
Android audio架构定义了audio功能的实现, 并将其同相关的源代码结合起来.

![Audio architecture](http://source.android.com/devices/audio/images/ape_fwk_audio.png)

**`Figure 1. Android audio architecture`**

-----
**应用中间层**
应用中间层包含有app代码, 它使用[android.media](http://developer.android.com/reference/android/media/package-summary.html) APIs来与音频硬件进行交互, 内部实现中, 它通过相关的JNI连接类来访问真正的底层代码, 来访问硬件.

**JNI**
Java Native Interface代码同android.media调用相关联, 来调用底层代码访问音频硬件. JNI的代码实现参考在**`frameworks/base/core/jni/ 和 frameworks/base/media/jni.`**

**本地 Native framework**
本地framework提供等同于**`android.media`**包的本地接口调用, 通过使用Binder IPC代理机制对media server中的音频相关的服务进行访问. 相关代码位于:**`frameworks/av/media/libmedia.`**

**Binder IPC**
Binder IPC代理完成进程间的通讯工作. 代理实现部分位于**`frameworks/av/media/libmedia`** 并且以字符**`"I"`**打头.

**Media server**
Media Server包含有音频服务, 该服务是对设备上的HAL实现进行真实调用的代码. media server中音频相关的部分存在于:**`frameworks/av/services/audioflinger.`**

**HAL**
硬件抽象层定义了音频服务所需要调用的标准的接口, 设备商需要基于当前的音频硬件正确的实现接口功能. 音频HAL接口位于:**`hardware/libhardware/include/hardware.`**, 细节参考**`hardware/audio.h.`**

**Kernel 驱动**
kernel中的音频驱动负责连接硬件和音频的HAL实现, 相关的实现架构可以为**`Advanced Linux Sound Architecture (ALSA)`**, **`Open Sound System (OSS)`**, 或者是自定义的驱动方式(HAL实现是与驱动无关的结构).

Note:如果使用ALSA的驱动模型, 建议使用external/tinyals作为驱动的部分, 因为它使用兼容的软件发布许可协议(标准的用户模式库是使用GPL许可的).

基于**`Open SL ES`**的Android 本地音频
这部分的API在Android NDK中提供, 它同android.media是同样级别的实现.

-----
