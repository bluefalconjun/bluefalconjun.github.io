

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


Since the Application Sandbox is in the kernel, this security model extends to native code and to operating system applications. All of the software above the kernel in Figure 1, including operating system libraries, application framework, application runtime, and all applications run within the Application Sandbox. On some platforms, developers are constrained to a specific development framework, set of APIs, or language in order to enforce security. On Android, there are no restrictions on how an application can be written that are required to enforce security; in this respect, native code is just as secure as interpreted code.

In some operating systems, memory corruption errors generally lead to completely compromising the security of the device. This is not the case in Android due to all applications and their resources being sandboxed at the OS level. A memory corruption error will only allow arbitrary code execution in the context of that particular application, with the permissions established by the operating system.

Like all security features, the Application Sandbox is not unbreakable. However, to break out of the Application Sandbox in a properly configured device, one must compromise the security of the the Linux kernel.