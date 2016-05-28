

###[**Implementing Security**](http://source.android.com/security/overview/implement.html)

Android安全工作组经常性收到android设备上的防止可能漏洞的信息请求. 同时该小组也会临时性的检查各类android设备来提醒供应商和相关开发者潜在问题的出现.

该文档提供基于android安全经验的最佳实现做法, 它扩展了提供给开发者的安全指导[**`Designing for Security`**](http://developer.android.com/guide/practices/security.html) 文档, 并且包含了特有的如何在设备上创建/安装系统级别的安全软件指导.

为促进该指导的适用性, Android安全小组尽可能的在[**`Android Compatibility Test Suite (CTS)`**](http://source.android.com/compatibility/cts-intro.html)和[**`Android Lint`**](http://tools.android.com/tips/lint)中增加了安全测试项. android鼓励合作伙伴增加测试项以帮助其他的android用户(在 **`root/cts/tests/tests/security/src/android/security/cts`**查看安全相关的测试项).

> **Development process**

在开发环境和过程中使用以下的方式来实现安全性:

**Reviewing source code**

源代码评审能够检查出大部分的安全问题, 它包含在该文档描述中. android强烈建议同时进行手动和自动的代码检查机制:

 - 在使用Android SDK的应用上运行Android Lint并且修复所有的已知问题.
 - Native代码建议使用自动检查工具来检测常见的内存管理问题, 例如溢出等.

**Using automated testing**

使用自动测试机可以检测出广泛的安全问题, 包括以下讨论的大多项::

 - CTS是一个随版本更新的测试系统. 在发布版本中进行CTS测试来验证兼容性.
 - 在开发过程中持续进行CTS测试来减少后期修正的时间, android自身也使用CTS作为持续集成过程中的自动构建步骤之一, 每天运行数次.
 - 设备提供商应当进行接口的自动化测试, 包括各类异常输入(模糊测试).

**Signing system images**

系统镜像的签名是保证其完整性的关键点, 推荐:

 - 设备不能使用公开的已知密钥进行签名.
 - 签名使用的密钥必须是以业界标准进行管理的, 管理方式包括通过硬件安全模块(HSM)来提供受限和安全的访问.

**Signing applications (APKs)**

应用程序签名在设备安全中扮演重要角色, 同时被用来在软件升级时进行权限控制. 在选择一种密钥对应用进行签名时, 重要的是考虑该应用仅在单独的设备上使用, 或者是被多设备进行访问. 推荐做法:

 - 应用不应当使用已公开的密钥进行签名. 
 - 使用的密钥必须由规范的安全组织来进行管理, 同时应当由HSM来进行受限可控的访问保护.
 - 应用不应当使用平台的密钥进行签名.
 - 同样包名的应用必须使用同样的密钥进行签名.  这通常发生在多设备上发布应用的情况. 尤其是如果在多设备上使用平台密钥签名. 如果应用是设备无关的, 那么应当对应用使用同一密钥签名, 如果是设备相关的, 那么为每种设备上的应用使用不同的唯一包名和密钥.

**Publishing applications**

Google Play支持设备生产商在不进行整体系统升级的情况下来更新应用. 这帮助进行快速安全问题更新, 同时可以方便的更新功能. 为应用能够获取唯一的包名: 

 - 将应用上传至Google Play来允许自动升级功能. 已经上传但是未发布的应用是用户不可见的, 但是其内容还是被更新了. 已经使用当前版本的用户可以重新安装它, 或者在其他兼容设备上安装. 
 - 使用公司商标的名称来为应用包进行命名, 这样可以保持唯一性.
 - 平台生产者所发布的应用应当上传至Google Play Store来避免包名与第三方开发者提供的应用产生重复.  否则第三方开发者可以通过上传同样包名的应用到Google Play并修改部分元数据的方法, 来重新取代当前平台生产者提供的应用. 并造成混淆.

**Responding to incidents**

外部的开发者们必须能够联系到设备提供商, 来报告一些设备相关的安全问题, 推荐是创建一个接受公共邮件的邮箱来管理安全事务.
创建一个 `security@your-company.com`或者类似名字的邮箱, 并且公布出来. 如果你发现影响Android系统或者多个设备提供商的android设备的安全问题. 可以直接通过填写安全问题报告([`Security Bug Report`](https://code.google.com/p/android/issues/entry?template=Security%20bug%20report))来联系Android Security Team.



> **Product implementation**

使用以下的推荐方式来发布产品.

**Isolating root processes**

在所有的权限升级攻击中, root进程是最常见的目标, 减少设备中root进程的数量来降低安全风险. CTS就包括了一个列出当前系统中root进程的信息测试.推荐做法:

 - 设备应当以root权限运行最少需要的代码. 如果可能的话, 尽量使用已存在的常规Android进程. 例如Galaxy Nexus ICS版本上仅存在6个root进程: **`vold, inetd, zygote, tf_daemon, ueventd, 和 init.`** 如果设备上的某个进程需要以root权限运行, 可以提交AOSP功能请求, 这样它的行为可以被公开审核.
 - 可能的情况下, root代码应该被隔离在未知的数据之外, 通过IPC来访问. 例如: 将所有的root权限下提供的功能集中到特定的小service中, 以验证权限的方式将这些功能通过Binder方式开放给其他应用. 并且提供最小/禁止网络访问权限.
 - root进程不允许监听网络端口.
 - root进程不允许为其他应用提供通用运行时环境(Java VM).

**Isolating system apps**

通常情况下, 预安装的应用不应当使用共享系统UID来运行. 如果应用需要以系统共享UID来运行或者是需要访问某些特权功能(如电话), 这个应用应当不扩展出任何可能被用户安装的第三方应用所访问的功能, 例如服务/广播监听器/内容提供等. 推荐做法:

 - 可能情况下, 设备仅在系统(system)权限下运行最小部分的代码.
Devices should run the minimum necessary code as system. Where possible, use an Android process with its own UID rather than reusing the system UID.
Where possible, system code should be isolated from untrusted data and expose IPC only to other trusted processes.
System processes must not listen on a network socket.
Isolating processes

The Android Application Sandbox provides applications with an expectation of isolation from other processes on the system, including root processes and debuggers. Unless debugging is specifically enabled by the application and the user, no application should violate that expectation. Best practices:

Root processes must not access data within individual application data folders, unless using a documented Android debugging method.
Root processes must not access memory of applications, unless using a documented Android debugging method.
Devices must not include any application that accesses data or memory of other applications or processes.
Securing SUID files

New setuid programs should not be accessible by untrusted programs. Setuid programs have frequently been the location of vulnerabilities that can be used to gain root access, so strive to minimize the availability of the setuid program to untrusted applications. Best practices:

SUID processes must not provide a shell or backdoor that can be used to circumvent the Android security model.
SUID programs must not be writable by any user.
SUID programs should not be world readable or executable. Create a group, limit access to the SUID binary to members of that group, and place any applications that should be able to execute the SUID program into that group.
SUID programs are a common source of user rooting of devices. To reduce this risk, SUID programs should not be executable by the shell user.
CTS verifier includes an informational test listing SUID files; some setuid files are not permitted per CTS tests.

Securing listening sockets

CTS tests fail when a device is listening on any port, on any interface. In the event of a failure, Android verifies the following best practices are in use:

There should be no listening ports on the device.
Listening ports must be able to be disabled without an OTA. This can be either a server or user-device configuration change.
Root processes must not listen on any port.
Processes owned by the system UID must not listen on any port.
For local IPC using sockets, applications must use a UNIX Domain Socket with access limited to a group. Create a file descriptor for the IPC and make it +RW for a specific UNIX group. Any client applications must be within that UNIX group.
Some devices with multiple processors (e.g. a radio/modem separate from the application processor) use network sockets to communicate between processors. In such instances, the network socket used for inter-processor communication must use an isolated network interface to prevent access by unauthorized applications on the device (i.e. use iptables to prevent access by other applications on the device).
Daemons that handle listening ports must be robust against malformed data. Google may conduct fuzz-testing against the port using an unauthorized client, and, where possible, authorized client. Any crashes will be filed as bugs with an appropriate severity.
Logging data

Logging data increases the risk of exposure of that data and reduces system performance. Multiple public security incidents have occurred as the result of logging sensitive user data by applications installed by default on Android devices. Best practices:

Applications or system services should not log data provided from third-party applications that might include sensitive information.
Applications must not log any Personally Identifiable Information (PII) as part of normal operation.
CTS includes tests that check for the presence of potentially sensitive information in the system logs.

Limiting directory access

World-writable directories can introduce security weaknesses and enable an application to rename trusted files, substitute files, or conduct symlink-based attacks (attackers may use a symlink to a file to trick a trusted program into performing actions it shouldn't). Writable directories can also prevent the uninstall of an application from properly cleaning up all files associated with an application.

As a best practice, directories created by the system or root users should not be world writable. CTS tests help enforce this best practice by testing known directories.

Securing configuration files

Many drivers and services rely on configuration and data files stored in directories such as /system/etc and /data. If these files are processed by a privileged process and are world writable, it is possible for an app to exploit a vulnerability in the process by crafting malicious contents in the world-writable file. Best practices:

Configuration files used by privileged processes should not be world readable.
Configuration files used by privileged processes must not be world writable.
Storing native code libraries

Any code used by privileged device manufacturer processes must be in /vendor or /system; these filesystems are mounted read-only on boot. As a best practice, libraries used by system or other highly-privileged apps installed on the phone should also be in these filesystems. This can prevent a security vulnerability that could allow an attacker to control the code that a privileged process executes.

Limiting device driver access

Only trusted code should have direct access to drivers. Where possible, the preferred architecture is to provide a single-purpose daemon that proxies calls to the driver and restricts driver access to that daemon. As a best practice, driver device nodes should not be world readable or writable. CTS tests help enforce this best practice by checking for known instances of exposed drivers.

Disabling ADB

Android debug bridge (adb) is a valuable development and debugging tool, but is designed for use in controlled, secure environments and should not be enabled for general use. Best practices:

ADB must be disabled by default.
ADB must require the user to turn it on before accepting connections.
Unlocking bootloaders

Many Android devices support unlocking, enabling the device owner to modify the system partition and/or install a custom operating system. Common use cases include installing a third-party ROM and performing systems-level development on the device. For example, a Google Nexus device owner can run fastboot oem unlock to start the unlocking process, which presents the following message to the user:

Unlock bootloader?

If you unlock the bootloader, you will be able to install custom operating system software on this phone.

A custom OS is not subject to the same testing as the original OS, and can cause your phone and installed applications to stop working properly.

To prevent unauthorized access to your personal data, unlocking the bootloader will also delete all personal data from your phone (a "factory data reset").

Press the Volume Up/Down buttons to select Yes or No. Then press the Power button to continue.

Yes: Unlock bootloader (may void warranty)

No: Do not unlock bootloader and restart phone.


As a best practice, unlockable Android devices must securely erase all user data prior to being unlocked. Failure to properly delete all data on unlocking may allow a physically proximate attacker to gain unauthorized access to confidential Android user data. To prevent the disclosure of user data, a device that supports unlocking must implement it properly (we've seen numerous instances where device manufacturers improperly implemented unlocking). A properly implemented unlocking process has the following properties:

When the unlocking command is confirmed by the user, the device must start an immediate data wipe. The unlocked flag must not be set until after the secure deletion is complete.
If a secure deletion cannot be completed, the device must stay in a locked state.
If supported by the underlying block device, ioctl(BLKSECDISCARD) or equivalent should be used. For eMMC devices, this means using a Secure Erase or Secure Trim command. For eMMC 4.5 and later, this means using a normal Erase or Trim followed by a Sanitize operation.
If BLKSECDISCARD is not supported by the underlying block device, ioctl(BLKDISCARD) must be used instead. On eMMC devices, this is a normal Trim operation.
If BLKDISCARD is not supported, overwriting the block devices with all zeros is acceptable.
An end user must have the option to require that user data be wiped prior to flashing a partition. For example, on Nexus devices, this is done via the fastboot oem lock command.
A device may record, via efuses or similar mechanism, whether a device was unlocked and/or relocked.
These requirements ensure that all data is destroyed upon the completion of an unlock operation. Failure to implement these protections is considered a moderate level security vulnerability.

A device that is unlocked may be subsequently relocked using the fastboot oem lock command. Locking the bootloader provides the same protection of user data with the new custom OS as was available with the original device manufacturer OS (e.g. user data will be wiped if the device is unlocked again).
