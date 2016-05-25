

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

android系统充分使用kernel提供的基于用户管理的权限模型, 
The Android platform takes advantage of the Linux user-based protection as a means of identifying and isolating application resources. The Android system assigns a unique user ID (UID) to each Android application and runs it as that user in a separate process. This approach is different from other operating systems (including the traditional Linux configuration), where multiple applications run with the same user permissions.

This sets up a kernel-level Application Sandbox. The kernel enforces security between applications and the system at the process level through standard Linux facilities, such as user and group IDs that are assigned to applications. By default, applications cannot interact with each other and applications have limited access to the operating system. If application A tries to do something malicious like read application B's data or dial the phone without permission (which is a separate application), then the operating system protects against this because application A does not have the appropriate user privileges. The sandbox is simple, auditable, and based on decades-old UNIX-style user separation of processes and file permissions.

Since the Application Sandbox is in the kernel, this security model extends to native code and to operating system applications. All of the software above the kernel in Figure 1, including operating system libraries, application framework, application runtime, and all applications run within the Application Sandbox. On some platforms, developers are constrained to a specific development framework, set of APIs, or language in order to enforce security. On Android, there are no restrictions on how an application can be written that are required to enforce security; in this respect, native code is just as secure as interpreted code.

In some operating systems, memory corruption errors generally lead to completely compromising the security of the device. This is not the case in Android due to all applications and their resources being sandboxed at the OS level. A memory corruption error will only allow arbitrary code execution in the context of that particular application, with the permissions established by the operating system.

Like all security features, the Application Sandbox is not unbreakable. However, to break out of the Application Sandbox in a properly configured device, one must compromise the security of the the Linux kernel.