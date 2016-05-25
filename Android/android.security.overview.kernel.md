

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
