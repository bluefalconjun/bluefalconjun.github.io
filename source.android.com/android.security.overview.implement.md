
###[**Implementing Security**](http://source.android.com/security/overview/implement.html)

-----
Android安全工作组经常性收到android设备上要求防止可能漏洞的信息请求. 同时该小组也会临时性的检查各类android设备来提醒供应商和相关开发者潜在问题的出现.

该文档提供基于android安全经验的最佳实现, 它扩展了提供给开发者的安全指导[**`Designing for Security`**](http://developer.android.com/guide/practices/security.html) 文档, 并且包含了特有的如何在设备上创建/部署系统级别的安全软件指导.

**`[Tips: 安全性需要应用开发者(developer)和OEM(平台提供商)共同完成.]`**

为促进该指导的适用性, Android安全小组尽可能的在[**`Android Compatibility Test Suite (CTS)`**](http://source.android.com/compatibility/cts-intro.html)和[**`Android Lint`**](http://tools.android.com/tips/lint)中增加向光的安全测试项. android鼓励合作伙伴增加测试项以帮助其他的android用户(在 **`root/cts/tests/tests/security/src/android/security/cts`**查看安全相关的测试项).

**`[Tips: CTS中关于安全性的检测可以用来作为修改平台软件设计的重要参考.]`**

-----
> **Development process**

-----
在开发环境和过程中使用以下的方式来实现安全性:

**Reviewing source code**

源代码评审能够检查出大部分的安全问题, 在该文档描述中. android强烈建议同时进行手动和自动的代码检查机制:

 - 在使用Android SDK的应用源码上运行Android Lint检查并且修复所有的已知问题.
 - 在Native类型的代码上建议使用相关的自动检查工具来检测常见的内存管理问题, 例如溢出等.

-----
**Using automated testing**

使用自动测试机可以检测出广泛的安全问题, 包括以下讨论的大多项:

 - CTS是一个跟随版本更新的测试系统. 在发布版本中进行CTS测试来验证设备兼容性.
 - 在开发过程中持续进行CTS测试来减少后期的修正, android自身也使用CTS作为(**`CI`**)持续集成过程中的自动构建步骤之一, 每天运行数次.
 - 设备提供商应当进行接口的自动化测试, 包括各类异常输入(模糊测试).

-----
**Signing system images**

系统镜像的签名是保证其完整性的关键点, 推荐:

 - 设备不能使用公开的已知密钥进行签名.
 - 签名使用的密钥必须是以业界标准进行管理的, 管理方式应该通过硬件安全模块(HSM)来提供受限和安全访问.

**`[Tips: 对image进行签名是平台提供商需要完成的最重要功能, 它是所有后续安全部管理的基础: TEE/DRM]`**

-----
**Signing applications (APKs)**

应用程序签名在设备安全中扮演重要角色, 同时被用来在软件升级时进行权限控制. 在选择一种密钥对应用进行签名时, 重要的是考虑该应用仅在单独的设备上使用, 或者是被多设备进行访问. 推荐做法:

 - 应用不应当使用已公开的密钥进行签名. 
 - 使用的密钥必须由规范的安全组织来进行管理, 同时应当由HSM来进行受限可控的访问保护.
 - 应用不应当使用平台的密钥进行签名.
 - 同样包名的应用必须使用同样的密钥进行签名.  这通常发生在多设备上发布应用的情况. 尤其是如果在多设备上使用平台密钥签名. 如果应用是设备无关的, 那么应当对应用使用同一密钥签名, 如果是设备相关的, 那么为每种设备上的应用使用不同的唯一包名和密钥.

**`[Tips: 设备提供商在最终出厂时, 会需要同零售商配合共同完成 OTA/签名APK密钥/image签名机制等操作.]`**

-----
**Publishing applications**

Google Play支持设备生产商在不进行整体系统升级的情况下来更新应用. 这可以帮助进行快速安全问题更新, 同时可以方便的更新功能. 

为了使应用能够获取唯一的包名: 

 - 将应用上传至Google Play来允许自动升级功能. 已经上传但是未发布的应用是用户不可见的, 但是其内容还是被更新. 已经使用当前版本的用户可以重新安装它, 或者在其他兼容设备上安装. 

 - 使用公司的商标名称来为应用包进行命名, 这样可以保持唯一性.

 - 平台生产者所发布的应用应当上传至Google Play Store来避免包名与第三方开发者提供的应用产生重复.  否则第三方开发者可以通过上传同样包名的应用到Google Play并修改部分元数据的方法, 来取代当前平台生产者提供的应用. 并造成混淆.

-----
**Responding to incidents**

第三方开发者必须能够联系到设备提供商, 来报告设备相关的安全问题, 推荐是创建一个接受公共邮件的邮箱来管理安全事务.

创建一个 `security@your-company.com`或者类似名字的邮箱, 并且公布出来. 如果你发现影响Android系统或者多个设备提供商的android设备的安全问题. 可以直接通过填写安全问题报告([`Security Bug Report`](https://code.google.com/p/android/issues/entry?template=Security%20bug%20report))来联系Android Security Team.

-----
> **Product implementation**

使用以下的推荐方式来发布产品.

**Isolating root processes**

在所有的权限升级攻击中, root进程是最常见的目标, 减少设备中的root进程数量来减低安全风险. CTS就包括了一个列出当前系统中root进程相关信息的测试.

推荐做法:

 - 设备应当以root权限运行最少的代码. 如果可能的话, 尽量使用已存在的常规Android进程. 
例如Galaxy Nexus ICS版本上仅存在6个root进程: **`vold, inetd, zygote, tf_daemon, ueventd, 和 init.`** 如果设备上的某个进程需要以root权限运行, 可以提交AOSP审核请求, 这样它的行为可以被公开评审.

 - 可能的情况下, root代码应该被隔离在未知数据之外, 通过IPC来访问. 
例如: 将所有以root权限来提供的功能集中到特定的子service中, 以验证访问权限的方式将这些功能通过Binder方式开放给其他应用. 并且提供最小/禁止网络访问权限.

 - root进程不允许监听网络端口.

 - root进程不允许为其他应用提供通用运行时环境(Java VM).

**`[Tips: 类似于平台的native media_player就是root权限程序的最好例子. ]`**

-----
**Isolating system apps**

通常情况下, 预安装的应用不应当使用(**`shared system uid`**)共享系统UID. 如果应用需要以系统共享UID来运行或者是需要访问某些特权功能(如电话), 这个应用应当不扩展出任何可能被用户安装的第三方应用所访问的功能, 例如服务/广播监听器/内容提供等. 

推荐做法:

 - 可能情况下, 设备仅在系统(**`system`**)权限下运行最小部分的代码. 在可能的情况下使用Android进程自己的UID而避免重用系统UID.

 - 可能情况下, 系统级别的代码应该被隔离在未知数据之外, 并通过IPC仅向可信任的进程提供服务.

 - 系统进程不允许监听网络端口.

-----
**Isolating processes**

Android应用沙盒(**`SandBox`**)机制保护程序不被系统上其他进程所干扰, 包括root进程和调试器也在内. 只有在应用和用户明确的允许调试行为时, 这种情况才能发生. 

推荐做法:

 - root进程不能访问应用私有的数据文件, 除非使用标准化的Android调试机制.

 - root进程不能访问应用私有的内存, 除非使用调试机制.

 - 发布的设备中不能包含任何可以访问其他应用/进程的数据/内存的应用.

-----
**Securing SUID files**

新的setuid程序不能被未授权的程序所调用. setuid程序是在获取权限升级的入侵过程中最常见被攻击的. 这种情况下将其保护在未授权程序范围之外是推荐的:

 - SUID程序必须避免提供shell或者是后门, 它们会被用于跳过android安全模型.

 - SUID程序必须对任何用户都是不可写的.

 - SUID程序不能是公开可读/可执行的, 创建一个管理组, 将任何需要访问SUID进程的应用加入到该组中.

 - SUID程序是用户root设备的标准资源, 所以不允许shell用户执行该程序以减少风险.

CTS verifier包含有一组测试来列出当前SUID文件, 在测试中某些setuid文件是被禁止的.

-----
**Securing listening sockets**

CTS测试在设备上进行时, 如果设备在监听任何端口/接口, 那么该测试将会失败. android将通过以下的推荐来验证安全性:

 - 初始运行设备时, 设备上不包含任何监听端口.

 - 必须能够以OTA以外的方式来关闭监听端口, 可以通过server连接或者用户配置的方式来完成.

 - root进程不允许监听任何端口.

 - 使用系统UID的进程不允许监听任何端口.

 - 为本地IPC而使用的socket, server应用必须使用UNIX域的socket类型(**`AF_UNIX`**), 并且以组的形式限制访问. 为这个IPC而创建的文件描述符并仅对指定的UNIX组赋予+RW权限. 任何client应用必须存在于这个UNIX组中.

**`[Tips: 设备内部的私有服务在进行通讯时, 如果不能使用binder机制, 则应该基于linux的AF_UNIX类型socket进行通讯, 并严格审核实现代码.]`**

 - 在某些包含有多个处理器的设备上(例如通讯模块和应用CPU分离的情况), 多处理器之间需要使用网络端口来进行通讯. 在这种情况下, 多处理器间的通讯必须使用隔离的网络接口来实现, 这可以避免设备上未授权的程序访问它(使用iptables来限制).

**`[Tips: 尽量通过使用local/其他的iptables定义的地址进行多处理器通讯. TEE不在此列.]`**

 - 监听端口的守护进程必须足够安全, 尤其是在处理恶意数据的情况下. Google将对端口进行未授权/授权的应用测试来检查其稳定性. 任何因此而导致的程序崩溃均会被定义为高优先级的bug来处理.

-----
**Logging data**

数据记录行为将 加大 数据被暴露的风险, 同时也会降低系统性能. android设备上已经发生过多起因为应用自行记录数据而导致的安全问题. 

推荐做法:

 - 应用和系统服务都不要记录由第三方应用产生的数据, 这里面可能包含有敏感信息.

 - 应用在正常操作过程中不要记录任何个人信息相关的数据.

CTS中包含有对系统log进行的检查, 以保证其中不包含任何可能敏感的信息内容.

**`[Tips: 所有开发过程中的log尽量使用debug宏隔开, 并最好能够以属性设置setprop/getprop的方式来打开或者关闭.]`**

-----
**Limiting directory access**

任意进程都可读写的目录将引入安全漏洞, 它将允许应用重命名可信任文件, 或者实施基于符号链接的攻击(攻击者可以通过创建符号链接来跳过程序对访问位置的限制). 可写的目录同样会造成通过删除所有应用相关的文件来达到删除应用自身的问题.

因此, 所有由系统或者root用户创建的目录不能是任意进程都可写的(**chmod 655/644 均是不允许的**). CTS测试将检查所有已知目录来帮助实现该限制.

**`[Tips: 模块设计的初期就应该避免所有对目录/文件有权限依赖的情况出现.]`**

------
**Securing configuration files**

许多驱动和服务依赖于 配置/数据 文件来进行工作. 通常存放的目录为**`/system/etc  或  /data`**. 如果这些文件是所有进程可读写并且由某些特权进程来进行处理的话, 这会导致攻击者通过伪造恶意数据内容来进行攻击. 建议方式:

 - 特权进程使用的配置文件**最好**不能是所有进程可读的.

 - 特权进程使用的配置文件**必须**不是所有进程可写的.

-----
**Storing native code libraries**

所有设备提供商提供的特权进程必须存放在**`/vendor 或 /system`**分区中; 这两个分区在启动时被挂载为只读. 

推荐的方式是, 所有系统或者其他高权限的应用所使用的相关库也必须存放在相关的分区中. 这可以避免攻击者通过替换特权进程使用的库文件来进行攻击. 

**`[Tips: 使用单独的/vendor分区可以更好的保证安全性, 并且更方便进行设备OTA.]`**

------
**Limiting device driver access**

只有可信任的代码才能够直接访问驱动. 在可能的情况下, 最好建立单一目标的守护进程来提供代理驱动访问. 可以在该守护进程上进行严格的访问限制检查. 同时, 驱动设备节点必须**不是**所有进程可读/可写的. CTS测试将检查所有公开的驱动设备节点来验证这一点. 

------
**Disabling ADB**

**`Android debug bridge (adb)`**是一个高效的开发调试工具, 但是它被设计为在可控的安全环境中使用, 而不是通用工具. 

推荐方式:

 - ADB在默认情况下是关闭的.

 - ADB在允许连接之前必须通过用户确认来打开.

------
**Unlocking bootloaders**

许多Android设备支持解锁功能. 它可以支持设备拥有者自行修改系统分区, 安装自定义的系统. 

常见的使用方式是安装第三方自制的ROM, 并且在设备上进行系统级别的开发. 例如: Google Nexus设备的拥有者可以运行oem fastboot 解锁工具来进行解锁, 解锁过程中会有以下信息显示:

    Unlock bootloader?
    
    If you unlock the bootloader, you will be able to install custom operating system software on this phone.
    
    A custom OS is not subject to the same testing as the original OS, and can cause your phone and installed applications to stop working properly.
    
    To prevent unauthorized access to your personal data, unlocking the bootloader will also delete all personal data from your phone (a "factory data reset").
    
    Press the Volume Up/Down buttons to select Yes or No. Then press the Power button to continue.
    
    **Yes**: Unlock bootloader (may void warranty)
    
    **No**: Do not unlock bootloader and restart phone.

-----

作为以上功能的推荐实现, 解锁android设备的动作必须包含安全删除所有的用户数据. 

如果删除动作失败并且设备被解锁, 那么这将造成物理磁盘上的系统被修改并且android用户数据将暴露给攻击者. 

为保护用户数据, 所有提供解锁功能的设备提供商必须正确的实现删除用户数据的操作(已经有很多设备提供商未能正确实现的例子). 

正确的实现解锁操作必须包含以下属性:

 - 当用户确认进行解锁操作, 立即开始进行数据清除工作. **`unlocked`**标识在安全的数据清除工作完成前不能被设置.

 - 如果安全的数据清除工作不能完成, 那么设备必须继续存在于锁定状态.

 - 在底层的块设备支持的情况下, **`ioctl(BLKSECDISCARD)`**或者等效的命令必须被使用来清除数据. 
对于eMMC设备, 这意味着使用安全擦除或者是安全修改(**`Secure Erase or Secure Trim`**)的命令. 
对于eMMC4.5或之后的产品, 这意味这使用一个正常的擦除/修改(**`Erase or Trim`**)命令后再完成一个消毒(**`Sanitize`**)操作.

 - 如果底层设备不支持**`BLKSECDISCARD`**操作, 使用**`ioctl(BLKDISCARD)`**来代替, 这是eMMC设备上的常见修改命令.

 - 如果**`BLKDISCARD`**也不支持, 使用全零覆写来擦除这个块设备也是可以接受的.

 - 用户在进行分区刷写之前, 必须能够对用户数据进行清空. 例如在Nexus设备上这将通过**`oem fastboot lock`**命令来完成.

 - 设备必须通过efuses/或其他方式来存储设备解锁/未解锁的信息.

以上的要求将保证一次完整的解锁过程中所有的用户数据都被安全删除掉. 不能完成该项保证的设备将被视为中等程度的安全漏洞.

设备解锁操作之后将可以继续进行上锁操作. 锁定bootloader将提供和原设备提供商OS相同方式的用户数据安全保护. 

-----