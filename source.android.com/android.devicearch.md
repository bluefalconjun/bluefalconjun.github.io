
##[Android Interfaces and Architecture](http://source.android.com/devices/index.html) 

----------
**Android**允许开发者自由实现设备和驱动的特殊功能. **Android**平台层通过硬件抽象层(**HAL**)这一标准机制同硬件进行挂钩.同时,它是开源的,开发者的接口和增强功能可以共享给其他人.

为保证设备能够提供上层的调用质量和用户体验一致性,设备必须通过兼容性测试(**CTS**). **CTS**保证设备能够达到标准质量. 细节参见[**Android Compatibility**](http://source.android.com/compatibility/index.html).

----------
#### 系统架构:
![Android System Architecture](http://source.android.com/devices/images/ape_fwk_all.png)


----------

> **app framework**

提供App开发者调用的Api支持.

> **binder ipc proxies**

Binder进程间交互机制(**IPC**)允许App跨进程边界调用**`android system service`**功能. 使API可以和系统服务互动. 

> **android system service**

通过API给上层调用来访问底层硬件,服务或者是模块. 关键组件有**`window manager`**等. 有两组区分的服务,系统服务和媒体服务.
**media service** / **system service**

> **HAL**

硬件抽象层定义标准的接口. 硬件厂商实现这些接口以屏蔽内部细节. HAL实现被打包为.so文件,由android system在适当时装载.
product提供的硬件必须实现对应的HAL层. android不实现HAL同driver的联系. 它由厂商自行实现. 但必须遵逊HAL interface定义.

####标准HAL结构:
指定硬件的属性定义于 [hadrware/liahardware/include/hardware/hardware.h](https://android.googlesource.com/platform/hardware/libhardware/+/android-5.1.1_r13/include/hardware/hardware.h). 它保证**HAL**层使用标准的数据结构. 该实现允许**Android**系统通过兼容的方式装载**HAL**模块的正确版本. **HAL**接口由两种常见的组件组成: **module**/**device**.

**module**代表一类打包的HAL实现. 它存在于单个**.so**动态连接库中. 它的元数据包括版本,名字,模块作者等. 用以帮助**android**正确的查找并载入. **hardware.h**文件定义了**hw_module_t**结构来描述**module**和其中包含的信息.
另外,**hw_module_t**包含有一个指向**hw_module_methods_t**结构的指针.这个**method**包含一个**open**该**module**的函数指针.这个**open**函数用以初始化同**HAL**所抽象硬件通讯部分. 特定的硬件**HAL**常通过增加接口来扩展标准的**hw_module_t**结构. 

例如在camera HAL中, **camera_module_t**结构包含**hw_module_t**结构,并增加了其他camera特定的函数:

    typedef struct camera_module {
        hw_module_t common;
        int (*get_number_of_cameras)(void);
        int (*get_camera_info)(int camera_id, struct camera_info *info);
    } camera_module_t;

当通过创建**module**结构来实现HAL module时,该结构必须定义**HAL_MODULE_INFO_SYM**. 
Nexus9 audio HAL的例子为:

    struct audio_module HAL_MODULE_INFO_SYM = {
        .common = {
            .tag = HARDWARE_MODULE_TAG,
            .module_api_version = AUDIO_MODULE_API_VERSION_0_1,
            .hal_api_version = HARDWARE_HAL_API_VERSION,
            .id = AUDIO_HARDWARE_MODULE_ID,
            .name = "NVIDIA Tegra Audio HAL",
            .author = "The Android Open Source Project",
            .methods = &hal_module_methods,
        },
    };

**device** 用来抽象**product**中的实际硬件实现. 例如: 一个**audio module**可以包含一个主**audio device**,一个**usb audio device**和一个**bt a2dp audio device**. **device**由**hw_device_t**来表示. 同**module**一样,每种类型的**device**可以通过**hw_device_t**来定义更多的特定硬件功能. 例如**audio_hw_device_t**结构包含有**audio device**特定的操作:

    struct audio_hw_device {
        struct hw_device_t common;
        /**
         * used by audio flinger to enumerate what devices are supported by
         * each audio_hw_device implementation.
         *
         * Return value is a bitmask of 1 or more values of audio_devices_t
         */
        uint32_t (*get_supported_devices)(const struct audio_hw_device *dev);
      ...
    };
    typedef struct audio_hw_device audio_hw_device_t;

除了标准属性外,每个特定硬件的HAL层结构均能定义更多独有的功能和实现. 参见[HAL Ref](http://source.android.com/devices/halref/index.html)以查看实现特定接口的不同指令.

####HAL Module:
**HAL**实现被编译为.so文件,由**android**系统动态载入. 可以通过增加**Android.mk**并指定**source**文件来实现**HAL**.通常情况下,**HAL**模块按照固定格式命名,这样**android**可以正确的查找和载入. 一般为: **module_type.device**: `audio.a2dp.so`.

> **linux kernel**

**android**使用标准**kernel**并增加/修改部分内容. 例如**wake locks/binder ipc**. 在**kernel**能够提供必须功能时,厂商可以使用任意版本的**kernel**来运行**andorid**.但建议使用最新的**android kernel**. 参见[编译kernel](http://source.android.com/source/building-kernels.html).




