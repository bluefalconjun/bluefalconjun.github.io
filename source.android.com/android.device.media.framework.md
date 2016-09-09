
###[**Media Framework Hardening**](http://source.android.com/devices/media/framework-hardening.html)

**`Media Framework Hardening 媒体处理硬件化.`**

-----
为提高设备的安全性, 从**`android 7.0`**起, 单独的**`mediaserver`**进程被分割成为多个进程, 每个进程将被按照使用需要进行设置单独的权限和运行属性限制. 这个改动将大大的降低**`media framework`**的安全性风险:

 - 将AV pipeline的组件拆分进入应用特定的沙盒进程中.
 - 允许可扩展升级的媒体组件(**`extractors,codec`**,等).

以上的变化通过显著的减少大部分media相关的安全漏洞, 为终端用户提高安全性, 保证用户设备和数据的安全.

**`OEMs`**和**`SOC`**的设备提供商需要升级**`HAL`**和**`framework`**的实现, 来适配设备到新的media framework架构. 尤其是由于以前版本中设备商提供的android代码经常假定所有的功能均运行在同一个进程中, 新架构下设备商需要修改相关的代码, 来绕过原生句柄(**`native_handle`**)的使用, 这意味着当前的运行可能是跨进程的. 和**`media hardening`**媒体处理硬件化相关的修改可以参考[**`frameworks/av`**](https://android.googlesource.com/platform/frameworks/av/+/master) 和 [**`frameworks/native`**](https://android.googlesource.com/platform/frameworks/native/+/master) .

**`关键硬件资源的多进程访问一直是OEM Driver中需要仔细处理的问题. 即使使用android原有的c/s服务架构, 在底层使用video driver/display driver 和多种硬件设备处理音视频时(例如 使用BT耳机来看电视 同时启用BT麦克风来进行语音搜索),OEM自己开发的daemon进程一向是bug最多的地方 :)`**

-----
> **Architectural changes 架构更新**

-----
前期版本中的Android使用一个独立单片的**`mediaserver`**进程, 并向其赋予了非常多的访问权限(相机/音频/视频驱动/文件/网络等等). Android 7.0将**`mediaserver`**进程分割成为多个新进程, 每个进程将拥有小得多的权限集:

如下图所示:

![Figure 1. Architecture changes for mediaserver hardening](http://source.android.com/devices/media/images/ape_media_split.png)

**`Figure 1. mediaserver 硬件化的结构更新`**

新的架构能够在保证, 即使在某个以上的进程被修改的情况下, 恶意软件也不能获取以前由mediaserver拥有的所有权限集, 每个进程的权限仍然是由Selinux和Seccomp规则所控制的.

注意: 由于某些设备商的依赖限制, 某些编解码器仍然需要运行在**`mediaserver`**进程中, 同时赋予**`mediaserver`**所需的更多的权限. 特别说明, Widevine的典型使用在Android 7.0中仍然在mediaserver中运行.

**`widevine/playready等需要porting的drm插件, 由于需要和TEE中的加解密副本进行通讯, 同时可能需要深度集成至av播放中, 所以它需要基本到root的权限, 而且非常依赖于具体的设备商安全方案相关的实现.`**

-----
**MediaServer改变部分**
 
 在Android 7.0中, mediaserver进程为了驱动播放和录制工作而存在, 例如: 在组件和进程中传递和同步各种数据缓存. 各个进程通过标准的Binder机制进行通讯.

在标准的本地文件播放实例中, 应用进程将文件描述符(**`FD`**)传递给mediaserver(通常通过**`MediaPlayer Java API`**完成), 而mediaserver进行的操作如下:

 - **`mediaserver`**将FD包装为Binder DataSource对象, 将其传递给**`extractor`**(该提取器通常为demux/pre-decode)进程, 提取器进程通过使用Binder IPC来读取文件(媒体提取器通常不会直接得到FD, 而是通过Binder回调到**`mediaserver`**来读取数据).
 - **`extractor`**进程检查文件, 创建文件类型对应的解析器(例如 MP3解析器/MPEG4解析器), 并为对应**`extractor`**向**`mediaserver`**返回Binder接口.
 - **`mediaserver`**通过Binder IPC调用**`extractor`**来判定当前文件中的数据类型(例如MP3或H.264数据).
 - **`mediaserver`**调用**`mediacodec`**进程来为所需类型创建编解码器, 并接收这些编解码器的Binder接口.
 - 重复进行Binder IPC调用, 从**`extractor`**中读取已编码的数据, 通过Binder将数据传送给mediacodec来进行解码, 并获取解码后数据.

在某些使用情况下, 可能没有codec参与到该过程中(例如后处理播放将编码数据直接送至输出设备), 或者当前的codec可能直接将解码后的数据进行渲染, 而不是返回数据至mediaserver(**`video playback`**).

-----
**MediaCodecService 改变部分**

codec服务负责实现解码/编码的实例. 由于设备商的依赖限制, 并不是所有的codecs都存在于codec进程中, 在Android 7.0中:

 - Non-secure(清流)解码器和软件编码器存在于codec进程中.
 - Secure(加密)解码器和硬件编码器存在于mediaserver进程中(与前期版本相同).

应用进程(或者是mediaserver)通过调用codec进程来创建所需类型的codec, 然后调用该codec来传入已编码数据/接收解码完成数据(解码实例), 或者是传入原始数据/接受已编码数据(编码实例). 同codecs之间的数据传输过程将使用已存在共享内存机制, 所以该部分的进程未发生变化.

-----
**MediaDrmServer  改变部分**

DRM服务在播放DRM保护的内容被使用, 例如在Google Play Movies中播放影片. 它将安全的处理对加密数据进行解密, 并且它将要访问各种证书和密钥存储设备, 和敏感组件进行通讯. 由于设备商的依赖限制, 在以上的例子中DRM 进程均未被使用.

-----
**AudioServer 改变部分**

**`AudioServer`**进程承载音频相关组件的工作, 例如音频输入和输出,**`policymanager`**用于确定音频路由(多种输入/输出时的配对和优先级问题)和FM广播服务.有关**`audioserver`**的变化和实现指导的详细信息，请参见[**`Implementing Audio`**](http://source.android.com/devices/audio/implement.html).

-----
**CameraServer 改变部分**

CameraServer负责控制摄像头, 并在录制视频时产生视频帧并将其提供给mediaserver进行后期处理. 关于CameraServer的变化和实现指导的详细信息, 参见[**`Camera Framework Hardening`**](http://source.android.com/devices/camera/versioning.html#hardening).

-----
**ExtractorService 改变部分**

解析器服务负责管理用来分析媒体播放框架支持的多种文件格式的解析器和相关组件. 解析器服务是所有服务中权限最低的服务 - 它无法读取文件句柄(**`FD`**), 仅通过Binder接口(由mediaserver为每一个播放实例创建)来访问文件.

应用实例(或者mediaserver)通过调用解析器进程来获取抽象接口(**`IMediaExtractor`**), 通过调用**`IMediaExtractor`**来获取**`IMediaSources`**来代理文件中的媒体流(track). 然后通过调用**`IMediaSources`**来从文件中读取媒体流.

为了在进程中传递数据, 应用实例(或者mediaserver)将文件数据以回复包(**`reply-Parcel`**)的形式包含在Binder传输中, 或者使用共享内存的方式来完成:

 - 使用共享内存的方式需要一次额外的Binder调用来释放它, 但该方式比较快,并且在大数据传输时比较节省耗电.
 - 使用包内形式(**`in-Parcel`**)需要额外的拷贝动作, 但是在数据量小于64KB时比较经济.

-----
> **实现参考**

-----
为了支持将**`MediaDrm`** 和 **`MediaCrypto`** 组件集成至新的mediadrmserver进程中, 设备商需要改变原有的加密内存的分配方式, 来允许这些内存能够在进程间共享.

在原有的Android实现中, 加密内存由**`mediaserver`**通过**`OMX::allocateBuffer`**进行分配, 并且用来在同一进程中进行解密操作. 如下图:

![low buffer in media server @ android 6.0](http://source.android.com/devices/media/images/ape_media_buffer_alloc_pren.png)

**`Figure 2. Android 6.0 中mediaserver对加密内存的使用.`**

在Android 7.0中, 内存分配进程以灵活的方式修改为新的机制, 并尽量减少对已有实现的影响. 通过对新的mediadrmserver进程中**`MediaDram`** 和 **`MediaCrypto`** 使用, 内存分配同以前发生了改变, 设备商需要对加密内存的句柄进行修改, 以便能够在Binder间进行传输, 例如当**`MeidaCodec`**需要向**`MediaCrypto`**发起解密操作时.

![low buffer in media server @ android 7.0](http://source.android.com/devices/media/images/ape_media_buffer_alloc_n.png)

Figure 3. Android 7.0 中mediaserver对加密内存的使用r.

-----
**使用本地句柄(native handles)**

**`OMX::allocateBuffer`**将返回一个指向native_handle结构体的指针, 它包含有文件描述符(**`FDs`**)和额外的整型数据, native_handle拥有所有使用文件描述符的优点. 包括现有的binder支持的序列化/去序列化, 这将有利于当前没有使用文件描述符的设备商更加灵活的使用它.

使用**`native_handle_create()`**来创建本地句柄. Framework代码将负责管理分配出来的native_handle结构, 并且负责在所有使用它的进程中释放它, 包括分配它的进程和其他已经去序列化的进程. framework通过以下方式来操作native handle: 在序列化/去序列化时使用**`Parcel::writeNativeHandle()/readNativeHandle().`**. 在释放时使用 **`native_handle_close() 然后紧接着使用 native_handle_delete()`**.

SoC设备商如果已经使用文件描述符来管理加密内存, 可以使用原有的FD来填充native_handle中的FD字段. 没有使用文件描述符的设备商可以使用native_buffer中的额外字段来表示/管理加密内存.

----------
**设置解密位置**

设备商需要更新当前的在native_handle中操作OEMCrypto机制, 来实施任何设备相关的加解密操作, 这是native_handle在新的进程空间中可用所必需的(通常这些改动由OEMCrypto更新来完成).

由于**`allocateBuffer`**是标准的OMX操作, Android 7.0为了native_handle支持增加了新的OMX扩展(**`OMX.google.android.index.allocateNativeHandle`**), 用来查询该扩展的支持. 并且加入了**`OMX_SetParameter`**调用来通知OMX实现, 当前必须使用native handle支持.

**`该部分类似vendor私有的handle, 不过加入了标准规范和omx插件集成. 细节需要参考aosp代码实现.`**

-----