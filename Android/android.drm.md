###[**DRM**](http://source.android.com/devices/tv/index.html)

![DRM](http://source.android.com/devices/images/ape_fwk_hal_drm.png)
这篇文档提供**Android DRM Framework**的整体描述. 并介绍**DRM**插件所必需实现的接口. 它并不描述**DRM**方案本身的健壮性和规范性.

> **Introduction**

**Android**平台提供可扩展的**DRM Framework**. 它允许应用按相应证书来管理内容许可保护的内容. **DRM Framework**支持多种**DRM**方案. 其中由平台级别的**DRM**方案由设备供应商提供. 

从**Android 3.0**中开始引入**DRM Framework**为应用开发者提供统一的标准接口并隐藏复杂的**DRM**操作细节. **DRM Framework**为受保护/无保护的内容提供同样的操作模式. DRM方案可以按照证书许可证的区分定义非常复杂的处理模型. **DRM Framework**提供**DRM**内容和证书之间的连接, 并处理权限管理. 它可以帮助媒体播放器从**DRM**保护/**非DRM**保护的内容中抽象出来. 

查看 [**MediaDrm**](https://developer.android.com/reference/android/media/MediaDrm.html)章节来理解这个为受保护流进行解密的类. 

![Android DRM HAL](http://source.android.com/devices/images/ape_fwk_drm.png)
**Figure 1.** `DRM Hardware Abstraction Layer`

对于用户来讲, 丰富的数字内容是非常重要的, 为保证内容广泛的被使用, **Android**开发者和数字内容提供商需要**Android**系统提供统一的**DRM**实现. **Google**在所有兼容的Android设备上免授权提供统一的**DRM**管理框架来让数字内容在该设备上可用. 从**Android 3.0**起, 该**DRM**插件同**Android DRM Framework**集成在一起. 并且可以使用基于硬件的保护进行内容证书和用户证书的保护. 

由**DRM**插件提供的内容保护依赖于底层硬件平台的安全性和内容保护能力. 设备硬件平台的安全性包含硬件的安全启动, 来建立一条以安全和受保护密匙为基准的可信任链路. 平台的内容保护能力包括通过可信任的输出保护机制来进行对**加密帧**的保护. 并不是所有的硬件平台均能支持以上所有的安全/内容保护功能点. 安全保护并不在协议栈和系统的某一个位置上实现. 而是依赖于硬件/软件和服务的集成来完成. 实现完整的安全设备由**硬件安全功能**, **可信任的安全启动**,和**单独的安全**OS****来处理所有的安全相关功能联合来共同实现. 


> **Architecture**

**DRM Framework**设计的目标是为了对特定的DRM实现方案进行抽象/聚合不确定的细节, 来实现一个标准的基于方案的**DRM**插件. **DRM Framework**的工作内容包含: 处理和隐藏复杂DRM操作的简单的**API**, 向在线**DRM**服务注册用户和设备, 从证书中解析约束信息, 关联**DRM**内容和证书, 最后解密**DRM**内容.

**Android DRM Framework**实现以下两个架构性的层:
 
 - 一套**DRM framework API**, 通过基于java虚拟机的方式向应用提供标准的**android framework**支持.
 - 一套本地代码的**DRM manager**, 实现**DRM Framework**并暴露**DRM**插件(代理)的接口来处理多种方案下的权限管理和解密操作.

![Android DRM Framework](http://source.android.com/devices/images/ape_fwk_drm_2.png)
**Figure 2.** `DRM framework`

细节部分, 参考[**Android DRM package reference**](http://developer.android.com/reference/android/drm/package-summary.html).


> **Plug-ins**

从下图实例可见, **DRM Framework**使用插件架构来支持多种**DRM**方案. **DRM Manager**服务同**DRM**插件以分离的方式, 在单独的进程中运行. 每个从**DrmManagerClient**中发起的**API**调用通过**Binder IPC**机制跨进程的向**DrmManagerService**发起. **DrmManagerClient**提供给运行时应用java实现. 同时它也为本地的**module**提供**DrmManagerClient-native**的实现. **DRM Framework**的调用者只同**DrmManagerClient**交流并且不需要关心具体的**DRM**实现方案.

![Android DRM Plug-in](http://source.android.com/devices/images/ape_fwk_drm_plugins.png)
**Figure 3.** `DRM framework with plug-ins`
在**DrmManagerService**启动时, 插件将被自动载入. 如下图所示, **DRMPluginManger**将载入/卸载所有可用的插件. **DRM Framework**将自动载入它在以下目录中查找到的插件:
**`/system/lib/drm/plugins/native/`**

![Android DRM Plug-in Lifecycle](http://source.android.com/devices/images/ape_fwk_drm_plugins_life.png)
**Figure 4.** `DRM plug-in lifecycle`
插件开发者需要保证插件被放置到指定的目录中. 以下章节是实现的细节.


> **Implementation**

**IDrmEngine**
**IDrmEngine** 是一套**DRM**使用实例的**API**接口. 插件开发者必须实现**IDrmEngine**中指定的接口. 并且实现下面的监听器接口, 接口的源代码定义存在于:
**`<platform_root>/frameworks/base/drm/libdrmframework/plugins/common/include`**

**DRM Info**
**DrmInfo** 是一个封装同**DRM**服务器通讯的协议的层. 服务器注册, 解除注册, 获取许可,和其他所有的服务器相关的传输均能被打包在一个进程中对**DrmInfo**进行处理. 协议应当由插件以**XML**文件的方式来描述. 每个**DRM**插件均应当对协议进行解析以完成传输工作. **DRM Framework**定义了**API**调用**`acquireDrmInfo()`**来获取**DrmInfo**实例. 

**`DrmInfo* acquireDrmInfo(int uniqueId, const DrmInfoRequest* drmInfoRequest);`**
获取注册/解注册或者权限请求的相关信息. 细节参考[**DrmInfoRequest**](http://developer.android.com/reference/android/drm/DrmInfoRequest.html).

**`DrmInfoStatus* processDrmInfo(int uniqueId, const DrmInfo* drmInfo);`**
**processDrmInfo()** 异步处理**DrmInfo**相关的信息. 它的处理结果通过**`OnEventListener`** 或者 **`OnErrorListener`**来返回.

**DRM rights**
播放**DRM**内容时, 需要联合**DRM**内容和它的证书. 当联合完成时, **DRM Framework**将处理证书(license)内容, 这可以帮助媒体播放器从具体的证书中抽象出来. 

**`int checkRightsStatus (String path, int action)`**
检查给定的受限内容是否有对应可用的证书.

**`status_t saveRights(int uniqueId, const DrmRights& drmRights, const String8& rightsPath, const String8& contentPath);`**
将DRM权限按照对应的路径存储并将其同内容路径联合起来.

**License Metadata**
证书元数据包括有证书有效期, 可重复次数等. 它可能被打包在受保护内容的权限中. **Android DRM Framework**提供**API**来返回它同受保护内容的约束关联. 查看**[DrmManagerClient](http://developer.android.com/reference/android/drm/DrmManagerClient.html)**来获取详细信息. 

**`DrmConstraints* getConstraints(int uniqueId, const String path, int action);`**
getConstraint函数调用返回受保护流中打包的键值对. 
The getConstraint function call returns key-value pairs of constraints embedded in protected content. To retrieve the constraints, the uniqueIds (the Unique identifier for a session and path of the protected content) are required. The action, defined as Action::DEFAULT, Action::PLAY, etc., is also required.

Android DRM License Metadata
Figure 5. Retrieve license metadata

DrmMetadata* getMetadata(int uniqueId, const String path);
Get metadata information associated with input content for a given path of the protected content to return key-value pairs of metadata.

Decrypt session

To maintain the decryption session, the caller of the DRM framework must invoke openDecryptSession() at the beginning of the decryption sequence. openDecryptSession() asks each DRM plug-in if it can handle input DRM content.

status_t openDecryptSession( int uniqueId, DecryptHandle* decryptHandle, int fd, off64_t offset, off64_t length);
The above call allows you to save DRM rights to specified rights path and make association with content path. DrmRights parameter is the rights to be saved, file path where rights should be and content file path where content should be saved.

DRM plug-in Listeners

Some APIs in DRM framework behave asynchronously in a DRM transaction. An application can register three listener classes to DRM framework.

OnEventListener for results of asynchronous APIs
OnErrorListener for receiving errors of asynchronous APIs
OnInfoListener for any supplementary information during DRM transactions.
Source

The Android DRM framework includes a passthru plug-in as a sample plug-in. The implementation for passthru plug-in can be found in the Android source tree at:
<platform_root>/frameworks/base/drm/libdrmframework/plugins/passthru

Build and Integration

Add the following to the Android.mk of the plug-in implementation. The passthruplugin is used as a sample.

PRODUCT_COPY_FILES += $(TARGET_OUT_SHARED_LIBRARIES)/<plugin_library>:system/lib/drm/plugins/native/<plugin_library> e.g.,
PRODUCT_COPY_FILES += $(TARGET_OUT_SHARED_LIBRARIES)/ libdrmpassthruplugin.so:system/lib/drm/plugins/native/libdrmpassthruplugin.so 

Plug-in developers must locate their respective plug-ins under this directory like so:
/system/lib/drm/plugins/native/libdrmpassthruplugin.so