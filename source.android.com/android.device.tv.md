
###[**TV Input Framework**](http://source.android.com/devices/tv/index.html)

![TIF](http://source.android.com/devices/tv/images/ape_fwk_hal_tv.png)

-----
**Android TV Framework(TIF)**定义如何处理**android tv**上提供的直播内容. **TIF**提供了一套标准的**API**框架.  提供给设备提供商来实现输入模块, 包含有**TV**的控制, 内容搜索, 和基于**TV**输入提供的元数据进行处理的建议.

**TIF**并不包含**TV**内容的实现, 或者其他区域化特殊需求. 但它提供帮助设备供应商实现数字**TV**广播的标准而提供的协议支持, 避免重复编码. 同时,它支持第三方**app**开发者自由创建定制的**TV**输入服务.

-----
> **Components**

-----
**TIF**框架通过包含**TV input manager**来完成其功能. **TIF**被上层的**TV app**调用. 

**TV app**是一个不能被替换的系统**app**, 它访问内部的集成/或者**IP tuner**上的频道. **Tv app**通过**tv input manager**来同设备供应商提供的**tv**输入模块或者第三方模块进行通讯.

-----
**TV Input Framework**包含的内容有:

**TV Provider** (com.android.providers.tv.TvProvider): 
关于频道,节目和相关权限的数据库.

**TV App** (com.android.tv.TvActivity): 
**app**界面用来处理用户操作.

**TV Input Manager** (android.media.tv.TvInputManager): 
**TV**的输入通过它来和**TV app**进行通讯交互.

**TV Input**: 
代表物理/虚拟的解调器或者是输入端口.

**TV Input HAL** (tv_input module): 
**TV**输入的硬件抽象层. 它支持系统的**TV**输入端口来访问特定设备实现上的**TV**硬件.

**Parental Control**: 
分级访问控制可以按需对特定的频道和节目进行锁定.

**HDMI-CEC**: 
通过**HDMI-CEC**协议对相关设备进行远程控制.

-----
以下是**Android TV Input Framework**的架构:
![TIF Arch](http://source.android.com/devices/tv/images/TIF_Overview.png)

`Android TV Input Framework (TIF) architecture`

-----
> **Flow**

-----
**TIF**执行序列:

1. 用户通过**tv app**来查看和访问**tv**内容.

2. **tv app**显示通过**tv input**提供的媒体内容.

3. **tv app**并不能直接同**tv input**进行通讯. **tv input manager**

为**tv app**提供和规范所有**tv input**的状态. 

-----
> **Permissions**

-----
- 只有签名认证的或者是系统的**tv input/tv app**才能够对**tv provider**提供的数据库进行完全访问. 同时也只有该两者能够接收按键事件.

- 第三方的**TV input**对**TV provider**数据库具有包锁定的访问权限, 且只对对应包的数据库内容行有读/写权限.

- 第三方的**TV input**能够显示它自己的内容,或者是设备提供商的**passthrough tv input** 的内容. 
例如: **HDMI1**. 但是它不能显示其他非**passthrough**的内容, 例如内建的**tv input**或者是**IPTV**的解调器内容. 

- 硬件**TV input app**会使用**TV_INPUT_HARDWARE** 权限,  来通知**Tv Input Manager**服务在启动时将自己的**Tv input**加入到**TV input service** 当中. 
这个权限允许硬件**Tv input**应用通过**Tv Input service**来支持多路的**Tv input**. 而且能够动态的加载/删减所使用的**Tv input.**

-----
> **TV Provider**

-----
**TV Provider** 数据库存储**tv input**中提供的频道/节目数据. 同时, 它负责申明和管理相关的权限, 这样单独的**Tv input**只能查看和访问自己的记录.  

在实际运行过程中, 特定的**tv input**只能访问它所提供的频道节目数据, 而不能访问其他**tv input**的内容.

**tv provider**在内部将频道信息的"广播类型"映射为"典型类型".  **Tv input**将负责按照底层广播标准来填充"广播类型". 而"典型类型"信息将按照对应的**"[android.provider.TvContract.Genres](https://developer.android.com/reference/android/media/tv/TvContract.Programs.Genres.html)"**的值来填充. 

举例说明: 按照广播标准**ATSC A/65**,当频道信息的类型为**0x25**(代表"**Sports**"), **Tv input**将在"广播类型"字段填充字符串"**Sports**", **Tv provider**将在"典型类型"字段填充所对应的映射值**[android.provider.TvContract.Genres.SPORTS](https://developer.android.com/reference/android/media/tv/TvContract.Programs.Genres.html#SPORTS)** .

-----
下图是**Tv provider**的详细介绍:

![TV Provider](https://source.android.com/devices/tv/images/TIF_TV_Provider.png)

`Android TV Provider`

*只有在特权系统分区的app才能读取整个Tv Provider的数据库.*

**Passthrough**类型的**Tv Input**不存储频道和节目信息.

同时为了对标准的频道/节目信息字段的扩展, **Tv Provider**数据库提供一个**BLOB**(二进制随机存储块)字段,  **[COLUMN_INTERNAL_PROVIDER_DATA](https://developer.android.com/reference/android/media/tv/TvContract.Programs.html#COLUMN_INTERNAL_PROVIDER_DATA)**. 

这部分的数据可以包含自定义的信息,  例如对应解调器的频率, 或者其他私有协议格式的内容. 同时数据库的数据项中包含有"**Searchable**"位来指定特定的频道在某些情况下是不可搜索的(不符合相关地区法规的内容保护).

-----
> **Database field examples**

-----
**Tv Provider**数据库支持的格式如下:
频道用使用数据结构**[(android.provider.TvContract.Channels)](https://developer.android.com/reference/android/media/tv/TvContract.Channels.html)**.  节目使用表 **[(android.provider.TvContract.Programs)](https://developer.android.com/reference/android/media/tv/TvContract.Programs.html)**.  

**Tv Input**和系统**app**共同访问和填充这些表项. 表项可以分为四种类型的区域:

 - **Display**:  该类型区域包含有app希望显示给用户的信息. 例如: 频道名称(**`COLUMN_DISPLAY_NAME`**), 频道编号(**`COLUMN_DISPLAY_NUMBER`**), 或者当前节目的名称.  

 - **Metadata**:  元数据类型区域按照相应的标准有三个部分. 频道的TS ID(**`COLUMN_TRANSPORT_STREAM_ID`**), 原始网络ID(**`COLUMN_ORIGINAL_NETWORK_ID`**), 和service id(**`COLUMN_SERVICE_ID`**). 

 - **Internal data**:  该类型区域被Tv Input内部使用.  其中某些位(**`COLUMN_INTERNAL_PROVIDER_DATA`**), 可以被**Tv input**以**BLOB**形式随机存储私有的频道/节目信息.

 - **Flag**:  标记位代表该频道是否可以被搜索/浏览/播放. 这个位只能在频道级别进行设置, 该频道上的所有节目均遵从设置. 
**`COLUMN_SEARCHABLE`**:  按照特定区域进行分级搜索. **`COLUMN_SEARCHABLE = 0`** 意味着无法在搜索结果中显示(**`403 :)`**).
**`COLUMN_BROWSABLE`**:  只对系统应用显示. 它可以限制频道不被其他应用浏览. **`COLUMN_BROWSABLE = 0`** 意味这它不在频道列表中显示.
**`COLUMN_LOCKED`**: 只能被系统应用访问. 它可以限制频道只能在输入正确的PIN码后进行播放. **`COLUMN_LOCKED = 1`** 意味着 该频道受锁定访问控制. 

详尽的使用规则可以查看 **[android/frameworks/base/media/java/android/media/tv/TvContract.java](https://developer.android.com/reference/android/media/tv/TvContract.html)**.

-----
> **Permissions and access control**

-----
数据库中的所有区域,均能被对该区域所在的行具有访问权限的对象访问. 用户不能直接访问任何区域, 数据库表项仅能被**TV App/System App**或者**Tv Input**接口所访问. 

数据中每一行均包含有**`PACKAGE_NAME`**.  对应的**package**(应用)可以通过**`TvProvider.java`** 来 `Query/Insert/Update`自己的行. **Tv Input**只能访问它自己写入的行, 而不能访问其他**Tv Input**所控制的行.

`READ, WRITE` 权限通过安装时的**AndroidManifest.xml** (需要用户确认) 来决定可访问的频道. 

只有签名认证或者系统的app才能获取**`ACCESS_ALL_EPG_DATA`**权限来访问完整的数据库. 

-----
> **TV Input Manager**

-----
**TV Input Manager**提供系统**API**给**Android TV Input Framework**. 它负责管理**app**和**tv input**的交互. 

每个**tv input**必须一对一的创建**tv input manager** 会话. **tv input manager**允许访问**tv input**, 所以**app**可以通过它来完成:

 - **列出当前的Tv input,并查看它们的状态.**

 - **创建会话并绑定监听器(listener)**
 
对于每个**tv input**会话. 单个的**tv input**被**app**配置为只连接到它加入到**tv provider**数据库的**URI**上. 除此之外它可以连接到**passthrough**的**tv input**上. 

**tv input**有它自己的音量设置. **tv input**由设备提供商来提供和部署. 有权限的**app**可以通过遍历数据库来访问/搜索所有的**tv**频道和节目.

**app**可以通用**android.media.tv.TvInputManager**来注册**TvInputCallback**, 这样当tv input的状态发生改变时, 它能够进行响应. 例如在tv input断开连接时, app可以停止显示并从可选列表中去掉它.

Tv Input Manager在TV App和TV Input之间进行抽象通讯. 它的标准访问接口可以让多个设备提供商来创建自己的tv app, 这样可以帮助所有的第三方tv inpunt在所有的tv app上工作.

-----
> **TV Inputs**

-----
Tv Input是标准的android app. 它们是包含有AndroidManifest.xml文件. 可以通过play/预安装/运行时载入安装等方法来进入到系统. Android TV系统支持以上所有类型的tv input.

某些input, 例如HDMI Input/板载解调器input. 只能被供应商提供, 因为它们需要同特定的底层硬件直接对话. 

另外的一些input, 例如 IPTV, 移动TV和外部的STB, 能够从google play上以apk的形式被第三方支持. 当安装完apk后, 这些tv input能有在TV app中被选用.

-----
**Passthrough input example**

![Android TV System Input](http://source.android.com/devices/tv/images/TIF_HDMI_TV_Input.png)

`Android TV System Input`

该实例中, HDMI TV Input由设备供应商提供, 它拥有完全访问TV Provider的权限. 作为passthrough TV Input, 它并不在TV Provider中注册任何的频道或节目. 

Tv App通过使用 android.media.tv.TvContract 组件的buildChannelUriForPassthroughInput(String inputId) 方法来获得引用至passthrough input的URI. TV App通过Tv Input Manager来访问HDMI TV Input.

-----
**Built-in tuner example**

![Android TV Built-in Tuner Input](http://source.android.com/devices/tv/images/Built-in_Tuner_TV_Input.png)

`Android TV Built-in Tuner Input`

该实例中, 板载解调器的TV Input由设备提供商提供,拥有全部访问数据库的权限.

-----
**Third-party input example**

![Android TV third-party input](http://source.android.com/devices/tv/images/Third-party_Input_HDMI.png)

`Android TV third-party input`

该实例中, 外部的STB TV Input由第三方提供. 由于它不能直接访问HDMI Video来提供数据, 它必须通过TV Input Manager来使用由设备商提供的HDMI TV Input.

通过TV Input Manager, 外部的STB TV Input能够同HDMI TV Input进行通讯. 并请求它在HDMI1上显示Video. 这时STB TV Input控制video内容而供应商的HDMI TV Input负责对video进行渲染.

-----
**Picture in picture (PIP) example**

![Android TV KeyEvents](http://source.android.com/devices/tv/images/TIF_PIP-PAP.png)

`Android TV KeyEvents`

该图描述从遥控器上的按键传递到指定的TV Input来实现PIP显示的流程. 

按键将由硬件提供商的驱动进行响应. 将硬件的扫描码转为Android按键代码, 并作为(按键事件)KeyEvents传递给标准的android输入管道. 由InputReader/InputDispatcher处理. 

当TV app当前在前台(focus)时, 控制TV app的行为.

只有系统的TV Input有权限获取InputEvents. 并且仅限于它拥有**`RECEIVE_INPUT_EVENT`**权限时. 

Tv Input负责决定哪些输入事件是需要由它来响应的, 并且允许TV app处理它不响应的事件.

Tv app管理查看当前那个系统的TV input是激活的. 当它被用户选定时, tv app负责分析输入的按键事件, 通过向相应TV input的TV input Manager会话调用`**dispatchInputEvent()**`来传递按键事件. 

-----
> **MHEG-5 input example**

-----
下图详细描述KeyEvents事件如何传递至Android TIF.

![Android TV Red button example](http://source.android.com/devices/tv/images/TIF_MHEG5_app.png)

`Android TV Red button example`

该例描述Red按键app的流程. 通常欧洲地区定义Red按键允许用户同电视上的app进行互动. 该app是ts流可支持的app. 当按键被按下的时候, 用户可以访问广播支持的app. 例如查看相关网页或者是比分信息.

查看Broadcast app章节来学习它如何同TV app互动.
流程:

 1. TV app当前处于前台并接收所有按键.

 2. 按键事件(KeyEvents)作为输入事件(InputEvent)传递给当前激活的的TV input.

 3. 系统的TV Input集成了MHEG-5协议栈并拥有`**RECEIVE_INPUT_EVENT**` 系统权限.

 4. 当接收到激活的键值(Red button), TV Input启动广播app.

 5. TV Input将按键事件作为输入事件进行处理. 然后广播app将获取焦点到前台并处理所有的输入事件. 直到该app被关闭.

注意: 第三方的TV Inputs不会接受按键.

-----
> **TV Input HAL**

-----
TV Input HAL辅助TV Input对特定的TV硬件访问的开发. 同其他Android标准HAL一样. tv_input源码存在于AOSP中. 供应商自行进行开发.

-----
> **TV App**

-----
TV App通过(com.android.tv.search.TvProviderSearch)提供频道和节目搜索结果,并通过TV Input Manager调用TV Input来处理按键,调频,音量控制等操作. 

设备供应商必须实现TV App来提供给用户各种调用工作. 第三方开发者不能开发TV app. 因为Tv App需要系统/签名权限.

通过同TIF的结合,TV App一般并不会实现设备供应商或者地区化的功能. 它默认完成任务是:

**Setup and configuration**

 - 自动检测TV Input.
 - 通过TV Input初始化频道配置.
 - 配置分级内容.
 - 更改TV设置.
 - 修改频道.

**Viewing**

 - 访问和查找所有的TV频道.
 - 访问TV节目信息栏.
 - 多语言和字幕支持.
 - 分级内容PIN码处理.
 - 允许TV Input UI覆盖:
     按照TV标准(HbbTV等)

-----
> **Parental Control**

-----
分级内容(家长模式)控制可以允许用户屏蔽不需要的频道或节目. 同时可以通过PIN码的方式打开这些内容.

TV App/TV Input Manager服务/TV Provider和TV Input共同完成分级内容控制的功能.

-----
**TV Provider**
每个频道的表项中有**`COLUMN_LOCKED`** 字段来锁定对应的频道访问, 直到输入PIN码. **`COLUMN_CONTENT_RATING`** 字段只进行显示, 并不参与分级控制模式.

-----
**TV Input Manager**
TV Input Manager存储每个分级的TVContentRating(内容分级), 并在isRatingBlocked()接口中响应, 以建议当前内容是否应在分级模式下屏蔽.

-----
**TV Input**
当显示的内容发生改变(节目/频道更改),或者分级内容控制设定改变(**`ACTION_BLOCKED_RATINGS_CHANGED`** 和 **`ACTION_PARENTAL_CONTROLS_ENABLED_CHANGED`**)时, **Tv Input**将向**Tv Input Manager**调用`**isRatingBlocked()**`. 

如果当前内容应当被屏蔽, **Tv Input**将关闭音视频, 并向通过调用**`notifyContentBlocked(TvContentRating)`** 向**TV app**通知当前内容被屏蔽. 反之亦然. 接口函数为**`notifyContentAllowed()`**.

-----
**TV App**
**TV App**负责向用户显示分级控制界面, 同时在内容被屏蔽或用户试图访问屏蔽内容时提供PIN码输入界面.

**TV App**并不直接存储分级内容控制的设定. 当用户更改设置时, 每种需要被屏蔽的**TvContentRating**由**TV Input Manager**来存储. 而被屏蔽的频道由**TV Provider**来存储.

-----
> **HDMI-CEC**

-----
**HDMI-CEC**是允许当前设备来控制其他的设备协议,通过这种方式可以用单个遥控器来控制家庭中的多设备. 

该协议被**Android TV**用来通过**TV App**来加速配置和远程控制各个**TV Inputs**. 例如,可以切换输入源,开关设备等操作.

**Android TIF**将**HDMI-CEC**实现为**HDMI Control Service**, 设备提供商只需要实现底层的驱动,和轻量级的**Android TV**硬件抽象层,不需要关注复杂业务逻辑. 

为了提供标准实现,**Android**通过标准化流程和支持可选功能来减少兼容性问题. **HDMI Control Service**基于已有的**Android Services**,包括输入和开关机功能.

这意味着现有的**HDMI-CEC**实现需要重新设计来适配**Android TIF**.推荐在硬件中包含微处理器来接收处理**CEC**开机和其他命令.

![CEC integration on Android TV](http://img.blog.csdn.net/20141231171546462?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWxwaGFfaGFycmlzb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

`CEC integration on Android TV`

 1. **CEC**总线从当前活动源接收命令切换到一个不同的源.

 2. 驱动将这个命令传给**HDMI-CEC**硬件抽象层.

 3. 硬件抽象层通知所有的**`ActiveSourceChangeListeners`**.

 4. **HDMI Control Service**通过**`ActiveSourceChangeListener`**来被通知进行切换.

 5. **TV Input Manager service**生成广播来通知**TV App**切换输入源.

 6. **TV App**为新的**TV Input**创建**TV Input Manager**会话并且在这个会话上调用`**setMain**`.

 7. **TV Input Manager**会话向这个**HDMI TV Input**发送信息.

 8. **HDMI TV input**请求设置显示界面.

 9. 界面设置完成后,**TV Input Manager Service**生成相应的切换控制命令回传给**HDMI Control Service**。

-----
>**TV integration guidelines**

-----
**Broadcast app**
由于每个国家均存在私有的广播电视标准(如**MHEG**,**Teletext**,**HbbTV**等),电视供应商希望将企业解决方案提供给广播应用。

**MHEG**:地区协议栈(MHEG-5:信息技术-多媒体和超媒体信息编码第5部分,是一个欧洲标准).
**Teletext**:地区协议栈.
**HbbTV**:基于Opera browser的webkit内核改动而来.

在**Android L**的发布中**,Android TV**期望电视供应商为区域性的电视标准协议栈集成或者使用**Android**的解决方案, 将显示**Buffer**传递给电视软件协议栈, 或者将一些按键值传递给更早的协议栈。

广播应用如何和**TV App**交互的:

 1. **TV App**获取焦点,接收所有的按键事件.

 2. **TV App**传递按键(红键)到**TV Input**.

 3. **TV Input**集成了早期电视协议栈.

 4. 当收到激活按键(红键)后,**TV Input**启动广播应用.

 5. 广播应用在**TV App**中接收焦点,然后处理用户的输入.
对于声音搜索或者推荐,广播应用支持应用内搜索或者声音搜索。

-----
**DVR**
如果设备支持的话,**Android TV**支持数字视频录制. 该功能工作如下:

 1. 所有的**TV Input**都可以实现数字视频录制/实时缓存.

 2. **TV App**把按键事件分发给**TV Input**(包括录制/暂停/快放/倒放按键)

 3. 当播放录制的内容时,**TV Input**把封装为正在"假装"播放的流.

 4. **DVR**应用可以让用户浏览和管理录制的节目.

对于声音搜索和推荐:

**DVR**应用支持应用内搜索和声音搜索.

**DVR**通过通知来提供推荐内容.

下图提供了在**Android TV**中**DVR**的一个可能实现:

![Digital video recording in Android TV](http://source.android.com/devices/tv/images/TV_Input_DVR.png)

`Digital video recording in Android TV`

-----
> **Reference Code**

-----
[**tv_input.cpp**](https://android.googlesource.com/platform/hardware/libhardware/+/android-cts-6.0_r1/modules/tv_input/tv_input.cpp)的代码.

    /*****************************************************************************/
    typedef struct tv_input_private {
        tv_input_device_t device;
        // Callback related data
        const tv_input_callback_ops_t* callback;
        void* callback_data;
    } tv_input_private_t;
    
    static int tv_input_device_open(const struct hw_module_t* module, const char* name, struct hw_device_t** device);
    static struct hw_module_methods_t tv_input_module_methods = {
        open: tv_input_device_open
    };
    
    tv_input_module_t HAL_MODULE_INFO_SYM = {
        common: {
            tag: HARDWARE_MODULE_TAG,
            version_major: 0,
            version_minor: 1,
            id: TV_INPUT_HARDWARE_MODULE_ID,
            name: "Sample TV input module",
            author: "The Android Open Source Project",
            methods: &tv_input_module_methods,
        }
    };

----------

    static int tv_input_get_stream_configurations(const struct tv_input_device*, int, int*, const tv_stream_config_t**)
    {
        return -EINVAL;
    }
    
    static int tv_input_open_stream(struct tv_input_device*, int, tv_stream_t*)
    {
        return -EINVAL;
    }
    
    static int tv_input_close_stream(struct tv_input_device*, int, int)
    {
        return -EINVAL;
    }
    
    static int tv_input_request_capture(struct tv_input_device*, int, int, buffer_handle_t, uint32_t)
    {
        return -EINVAL;
    }
    
    static int tv_input_cancel_capture(struct tv_input_device*, int, int, uint32_t)
    {
        return -EINVAL;
    }

-----
> **Implementation Sample**

-----
    /***********************************************************************************************/
    static struct hw_module_methods_t tv_input_module_methods = {
        open: tv_input_device_open
    };
    
    tv_input_module_t HAL_MODULE_INFO_SYM = {
        common: {
            tag: HARDWARE_MODULE_TAG,
            version_major: 0,
            version_minor: 1,
            id: TV_INPUT_HARDWARE_MODULE_ID,
            name: "Company TV input module",
            author: "Company Technology Group Ltd.",
            methods: &tv_input_module_methods,
        }
    };

----------

    static int tv_input_get_stream_configurations(const struct tv_input_device* dev __unused,
                                                 int device_id, int* num_configurations,
                                                 const tv_stream_config_t** configs)
    {
        ALOGV("Get device %d's configuration", device_id);
        Mutex::Autolock l(device_mutex);
        ssize_t idx = devices.indexOfKey(device_id);
        if (idx >= 0) {
            mrvl_tv_device* tv_device = devices.valueFor(device_id);
            *num_configurations = 1;
            // TODO: Should we malloc it
            *configs = const_cast<tv_stream_config_t*>(&(tv_device->config));
            return 0;
        }
        ALOGE("Device %d isn't registered", device_id);
        return -EINVAL;
    }
    
    static int tv_input_open_stream(struct tv_input_device*, int device_id, tv_stream_t* tv_stream)
    {
        ALOGV("Open device %d", device_id);
        Mutex::Autolock l(device_mutex);
        ssize_t idx = devices.indexOfKey(device_id);
        if (idx >= 0) {
            mrvl_tv_device* tv_device = devices.valueFor(device_id);
            if (tv_device->device != NULL) {
                ALOGE("Device %d is already opened", device_id);
                return -EEXIST;
            }
            if (tv_device->config.stream_id != tv_stream->stream_id) {
                ALOGE("Stream %d is wrong. It should be %d", tv_stream->stream_id,
                     tv_device->config.stream_id);
                return -EINVAL;
            }
            uint32_t type;
            sp<NativeHandle> stream;
            tv_device->device = new TvDevice();
            sp<HALListener> listener = new HALListener();
            if (tv_device->device->setDataSource(String8(tv_device->uri)) != android::OK) {
                ALOGE("Can't open device %s", tv_device->uri);
                return -EINVAL;
            }
            if (tv_device->device->setListener(listener) != android::OK) {
                ALOGE("Can't set listener to device %s", tv_device->uri);
                tv_device->device->release();
                return -EINVAL;
            }
            if (tv_device->device->open() != android::OK) {
                ALOGE("Can't open device %s", tv_device->uri);
                tv_device->device->release();
                return -EINVAL;
            }
            if (tv_device->config.type == STREAM_TYPE_INDEPENDENT_VIDEO_SOURCE) {
                if (tv_device->device->getSidebandStream(stream)) {
                    ALOGE("Can't get baseband stream from device %s", tv_device->uri);
                    tv_device->device->close();
                    tv_device->device->release();
                    return -EINVAL;
                }
                tv_stream->type = TV_STREAM_TYPE_INDEPENDENT_VIDEO_SOURCE;
                tv_stream->sideband_stream_source_handle =
                                                const_cast<native_handle_t*>(stream->handle());
            } else {
                // TODO: Implement buffer producer stream later.
                ALOGE("Do not implement buffer producer now");
                return -EINVAL;
            }
            return 0;
        }
        return -EINVAL;
    }

----------
