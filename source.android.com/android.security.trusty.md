
###[**Trusty TEE**](http://source.android.com/security/trusty/index.html)

-----
Trusty是一套软件组件集合, 它支持实现移动设备上的可信任执行环境(TEE).

Trusty包含有:

 - 基于处理器架构的操作系统(Trusty OS).

 - 基于TEE实现的驱动, 该驱动方便android kernel(linux)来同运行在该系统上的应用进行通讯.

 - 基于软件的库组合, 它将提供给android系统软件通过kernel驱动同该系统上的应用进行通讯.

注意: Trusty/Trusty API会由管理者进行修改.

Trusty API的相关信息, 请参考[**`API Reference`**](http://source.android.com/security/trusty/trusty-ref.html)

**`[Tips: trusty是android推荐的TEE实现方案.]`**

-----
> **Uses and examples**

-----
任何TEE系统(不仅限于Trusty)都能用于TEE环境的实现.

TEE环境的处理器通常是一个系统中单独分离的微处理器, 或者是主处理器的虚拟实例. 

TEE处理器同系统中的其他部分隔离开来, 并且通过硬件的支持来实现对内存和I/O接口的保护.

TEE处理在现在的移动设备中非常重要, 设备上的主处理器被认为是"不可信任的", 并且被限制对 某些RAM的指定空间, 硬件寄存器 和 用以存储设备提供商保存在保险区间的加密数据(例如设备相关的密钥对) 进行访问. 

主处理器上的软件在需要使用加密数据时, 均会通过TEE处理器代理完成.

android系统中, 对于该实现的最广泛的例子就是为保护内容的[**`DRM framework`**](http://source.android.com/devices/drm.html)提供服务. 

TEE上的软件可以访问设备相关的密钥, 并使用该密钥进行保护内容的解码. 主处理器仅仅能够看到加密的内容, 这将对基于软件的攻击起到高级别的保护.

基于TEE的使用案例还有其他例子. 

移动支付, 安全银行. 全局磁盘加密. 多步验证. 设备重置保护. 重播保护的存储. 无线显示("cast")受保护的内容. 安全密码和指纹验证过程都需要使用TEE. 同时它还可以进行间谍软件侦测.

**`[Tips: 一般来讲, 所有和密码/验证/敏感数据相关的操作, 都应该交由secure world进行处理.]`**

Trusty提供API来支持两种类型应用的开发:

 - 运行在TEE处理器上的可信任应用/服务.

 - 使用TEE处理器上提供服务的 正常/不可信的应用, 它将运行在主处理器上.

主处理器上软件可以使用Trusty API来同可信任应用进行通讯, 它们可以传递任意定义的消息. 

该机制类似于基于IP的网络服务.  应用可以自己定义消息数据的格式/语意, 它可以是应用级别的协议. 

底层的Trusty基本通讯功能将保证消息的可信任传输(以主处理器的驱动方式进行), 并且通讯是完全异步的.

-----
> **Trusted applications and services**

-----
可信任应用是以独立进程的方式在Trusty OS kernel上运行的. TEE处理器也会提供MMU功能, 这可以保证每个进程均运行在它自己的虚拟地址空间上. 

TEE OS kernel将基于安全计时器对全部进程按照优先级顺序进行循环运行. 在当前Trusty实现中, 所有进程共享相同的优先级.

**`[Tips: 由于进程优先级相同, 所以运行在TEE中的进程必须仔细考虑处理时间/锁死等情况, 并尽量简化操作.]`**

-----
**Language and threading support**

基于Trusty的应用可以用C/C++语言开发(C++支持有限), 并且可以访问一个较小的C库. 当前系统的main()函数并不接受参数. 系统调用过程由C库中的汇编代码实现, 所以可以通过名称来进行系统调用.

所有Trusty应用都是单线程的, Trusty用户空间不支持多线程功能.

-----
**Application structure**

Trusty应用在载入时初始化单次, 并一直保留在系统中, 直到TEE处理器被重置. Trusty不支持动态的加载/卸载应用.

Trusted应用被编写以事件驱动的服务器形式, 它等待其他应用或者从主处理器上的其他应停用发送过来的请求命令. 同时它也可以作为其他可信任应用服务的客户端. 

驱动事件的类型由API部分来描述, 并且有Trusty kernel来发送给对应的应用.

**`[Tips: TEE image将有bootloader在启动时一次初始化并运行. 这可以保证在image有效的情况下, TEE本身是安全的.]`**

-----
> **Third-party Trusty applications**

在当前情况下, 所有Trusty的应用均由设备提供商单独进行开发, 并将其和Trusty kernel镜像整合在一起. 该镜像将被签名, 并且由bootloader在启动过程中进行验证. 

第三方的应用开发目前并不支持.

即使在Trusty系统允许新的开发者进行应用开发时, 开发者也必须非常注意安全问题, 并需要较多的开发经验. 

每个新的应用将增加系统的可信任计算基础(TCB). 可信任应用可以访问设备的机密数据, 并且能够对其进行计算和数据传递操作.

当然基于TEE的应用开发可以带来广泛的机遇和改变, 由于TEE的定义限制, 这些应用在经过严格的攻击测试前是不能进行分发的. 通常情况下这类应用的运行应该由用户完全信任的数据证书来保证.

-----
> **Downloading and building Trusty**

-----
可以在以下地址访问Android开源项目(AOSP)的Trusty实现:

    https://android-review.googlesource.com/#/admin/projects/?filter=trusty

AOSP上的Trusty kernel 分支地址:

    https://android.googlesource.com/kernel/common/+/android-trusty-3.10
    https://android.googlesource.com/kernel/common/+/android-trusty-3.18

在已存在的android编译环境中使用以下方式编译Trusty:

    $ repo init -u https://android.googlesource.com/trusty/manifest
    $ repo sync
    $ make -j24 generic-arm64

可以用过选择不同的编译目标来进行以上的编译: 

**`device/*/*/project/*`**

-----
> **其他TEE介绍**

**[open tee参考](http://www.linaro.org/blog/core-dump/op-tee-open-source-security-mass-market/):**

![opentee](http://www.linaro.org/wp-content/uploads/2014/09/op-tee_diagram.png)

OpenTee协助设备提供商在arm系列CPU上实现完整的TEE-OS环境支持. 上图的绿色线框部分由OpenTee实现. 设备提供商基于tee_client/tee_os进行TEE相关应用的开发.

-----
[基于tee的android secure video path参考](http://embedded-computing.com/articles/distributed-using-trusted-execution-environment/):

![secure video path](http://d2lupdnmi5p5au.cloudfront.net/i__src9fe7ccf370bd91e1526c8fb595468df4_paraf0d99c20bd457d46a92c72841873c47.jpeg)

完成基于硬件的secure video path需要以下模块的共同支持:

 - 按照android drm plugin标准实现对应的plugin模型(playready/widevine)-android side.

 - 实现安全的TEE环境支持.

 - video process unit可以支持clear/secure video decode/display. 基于android omx plugin标准实现(5.0/6.0?).

 - 基于硬件的证书处理机制(某些运营商的证书会随设备出厂预装).

 - HDCP支持.

-----