

###[**System and kernel security**](http://source.android.com/security/overview/kernel-security.html)

在操作系统的级别上, android平台定义了linux kernel的安全性, 以及在不同进程中执行的应用之间通讯的安全性, 这通过安全的进程间通讯(IPC)功能来实现. 这些系统级别的安全机制保证了应用沙盒中的应用的native代码也是安全受限的. 无论这些native代码所实现的是正常应用返回结果或者是开发漏洞造成的失控行为, 系统都能够阻止应用对其他应用/android系统或者设备本省造成损害. 

> **Linux Security**

android平台基于linux kernel建立. kernel已经经过了多年广泛的使用, 并且被运用在百万级的安全敏感的环境中. 通过开发者持续的对kernel进行研究/渗透/破解和修补, linux已经成为被大部分公司和安全专家所信赖的系统环境.

linux kernel为android平台提供以下的关键安全功能:

 - 用户权限模型.
 - 进程隔离
 - 可扩展的安全IPC机制
 - 可移除无用/高危模块的功能

作为多用户操作系统, linux kernel的安全基本问题是隔离各个用户之间的资源. 实现例如:

 - 避免用户A读取用户B的文件
 - 保证用户A并不会访问/影响到用户B的内存
 - 保证用户A并不会访问/影响到用户B的计算资源
 - 保证用户A并不会访问/影响到用户B的设备(例如: 电话/GPS/蓝牙)


> **The Application Sandbox**

android系统充分使用kernel提供的基于用户管理的权限模型, 来定义和隔离应用资源. android系统为每一个android应用运行独立的进程, 并为其分配一个唯一的UID. 在android系统中这种方式不同于其他的系统(包括经典的linux系统配置, 这种配置中多进程共享同样的用户权限).

为了建立起kernel级别的应用沙盒, kernel通过标准的linux机制在应用和系统之间强制实行安全管理. 每个应用均被赋予uid/gid. 在缺省情况下, 应用之间不能交互, 同时对系统也只存在限制的访问权限. 举例说明: 当A应用尝试某些危险行为类似于读取B应用的数据或者在无权限的时候拨打电话(电话是一个独立应用), 系统会阻止这类操作因为A并没有足够的用户权限. 沙盒是比较简单的安全机制, 它基于历史久远的UNIX风格的以用户方式分离的处理/文件权限.

![overview](http://source.android.com/security/images/android_software_stack.png)

因为应用沙盒存在于kernel中, 这个安全模块可以同时管理native代码和系统应用. 上图中所有kernel之上的软件模块包括系统库/应用中间件/应用运行环境和所有应用都处在应用沙盒保护中. 在某些其他平台上, 开发者被限制在某些特定的中间件/api支持环境下以保证安全. 但是在Android中, 对于应用的编写没有这种限制来保证安全, 在这一点上, 所有native代码的应用和解释性代码的应用具有同样的安全性.

在某些系统中, 内存越界访问会导致完全的设备安全性问题. 但在android中不存在这种情况, 所有的应用和它们的资源均在OS级别被沙盒保护. 应用的内存越界问题仅仅允许当前应用的代码可能被非法执行, 同时它仍然在操作系统的权限控制之内.

同所有的安全特性模块一样, 沙盒机制并不是完全无法突破的, 但是在一个正确配置的设备上破解应用沙盒限制, 那么攻击者首先必须攻破linux kernel的安全机制.


> **System Partition and Safe Mode**

系统分区包含有android的核心, 它由android系统库, 应用运行环境, 应用中间件和应用组成. 当用户将系统启动至安全模式(Safe Mode), 那么只有核心android应用可以被使用. 这可以保证用户可以将设备运行在没有第三方软件的环境中.

> **Filesystem Permissions**

在UNIX类型的环境中, 文件系统权限管理保证用户不能修改或者读取其他用户的文件. Android在这种环境中将所有应用运行在它自己的用户管理下, 每个应用创建的文件都不能被其他应用读取或者修改, 除非该应用明确的向其他应用提供文件访问权限.

> **Security-Enhanced Linux**

Android使用Security-Enhanced Linux (SELinux)来完成访问控制规则, 并在此基础上建立强制访问控制的环境(mac). 细节参见[**Validating Security-Enhanced Linux in Android.**](http://source.android.com/security/selinux/index.html)

> **Cryptography**

Android为应用提供一套加密APIs来使用. 它包含有标准广泛使用的加密算法实现: AES/RSA/DSA/SHA. 同时, 也提供高级别的协议支持, 例如SSL/HTTPS. 

从Android 4.0中引入了[`KeyChain`](http://developer.android.com/reference/android/security/KeyChain.html)类允许应用来使用系统凭证的存储空间来保存私有的key和证书链.


> **Rooting of Devices**

缺省情况下, android系统中只有kernel和很小一套的核心应用运行在root权限上. android并不阻止应用运行在root权限下来修改系统/kernel或者其他应用. 通常, root权限拥有对所有应用和数据的访问权限. 用户在android设备上对应用进行root权限提升会造成安全模型被恶意软件和漏洞攻击的风险加大.

自行修改android平台对Android平台的开发者来说是很重要的. 在很多android设备上, 开发者可以通过解锁bootloader来安装修改过的android版本, 在这些修改版本上开发者/用户可以对应用自行升级root权限, 这可以帮助进行应用/系统组件的调试, 并且可以让应用访问一些未通过标准android api提供出来的访问权限.

在某些设备上, 拥有物理控制和usb线缆的连接的个人可以自行安装新的修改过的android版本来获取root权限. 为了保护在这种情况下的用户数据, 解锁bootloader的机制中需要增加对所有以存在的用户数据的清空动作. 利用kernel缺陷和安全漏洞的root权限提升可以跳过这个保护.

在设备上存储key的方式来加密并不能在root用户面前保护应用数据, 应用可以通过不存在设备上的key加密来保护数据. 这种方法可以在key不存在时临时保护数据, 但是当它被提供给应用时它对于root用户来讲就是可见的.

更加稳健的从root用户访问情况下保护数据的方式是通过使用硬件方案来完成的. OEMs将选择实现硬件安全方案, 这可以限制对特定类型内容的访问, 例如 播放视屏时的DRM, 或者Google钱包中使用的可信存储的NFC数据.

在这种情况下, 丢失或者被偷盗的设备, 它的完整的文件系统被通过设备密码保护的加密key所保护, 在没有设备密码的情况下即使修改bootloader或者系统本身也无法获取用户数据.


> **User Security Features**

**Filesystem Encryption**

Android 3.0和后期的版本提供了全局文件系统加密支持, 所有的用户数据均能在kernel中通过dmcrypt来完成, 它通过CBC和ESSIV:SHA256来实现AES128的加密. 加密kernel以从用户密码中产生的key来进行AES128保护. 在没有用户密码的情况下无法访问存储的数据. 为了避免密码破解攻击(例如: 彩虹表,暴力破解), 密码同一个随机数和预先产生的使用标准PDBDF2算法的SHA1重复哈希数进行混合, 之后才能用来解密文件系统的key. 为了避免密码字典猜测的解密方式, android提供了一套复杂的密码定义机制, 它由系统管理者设备并且由系统强制实现. 文件系统加密需要用户密码的支持, 常见的图形密码锁定并不支持这一点.

More details on implementation of filesystem encryption are available at Encryption.

Password Protection

Android can be configured to verify a user-supplied password prior to providing access to a device. In addition to preventing unauthorized use of the device, this password protects the cryptographic key for full filesystem encryption.

Use of a password and/or password complexity rules can be required by a device administrator.

Device Administration

Android 2.2 and later provide the Android Device Administration API, which provides device administration features at the system level. For example, the built-in Android Email application uses the APIs to improve Exchange support. Through the Email application, Exchange administrators can enforce password policies — including alphanumeric passwords or numeric PINs — across devices. Administrators can also remotely wipe (that is, restore factory defaults on) lost or stolen handsets.

In addition to use in applications included with the Android system, these APIs are available to third-party providers of Device Management solutions. Details on the API are provided at Device Administration.

