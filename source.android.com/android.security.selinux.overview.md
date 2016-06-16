
###[**Security-Enhanced Linux in Android**](http://source.android.com/security/selinux/index.html)

-----
> **Introduction**

-----
android安全模块基于应用沙盒来实现. 每个应用运行在它自己的沙盒中. 

在android 4.3之前, 沙盒由应用在安装时分配的唯一的UID来实现. 

从android 4.3开始, Security-Enhanced Linux (SELinux)被用作来定义android应用的使用界限.

作为android安全模型的一部分, Selinux被用来在所有进程上来执行强制访问控制(MAC), 即使该进程运行在root/superuser权限上.

Selinux从两个方面来帮助增强android安全性, 一是限制特权进程的访问范围, 二是自动生成安全规则.

有许多公司和组织对android selinux进行贡献和提交. 所有的参与者和提交内容都可以在[**`android.googlesource.com`**](https://android.googlesource.com) 上查看和评审. 

-----
基于selinux, android能够更好的保护和限制系统服务, 控制对应用数据和系统记录的访问, 防止间谍软件的破坏, 并且可以在移动设备上对因为应用本身代码缺陷而产生漏洞的情况下保护用户信息.

android包含了默认强制开启的selinux, 并在AOSP中默认集成了相关的安全规则. 

在强制模式中, 所有的非法操作均被拒绝, 并且所有的尝试操作均会被kernel记录到dmesg和logcat中. 

在设备提供商的开发过程中, 一般是通过收集和分析所有的error log, 然后修改软件/增加selinux规则来实现强制开启.

-----
背景:
Selinux是以默认禁止的方式来操作的. 任何不被明确允许的操作都会被禁止. 

同时, 它可以运行在以下两种模式之一: 

宽容(**`permissive`**)模式, 这种模式下所有不被允许权限的操作不会真的被禁止, 但是会被记录. 

另一种是强制(**`enforce`**)模式, 以上操作会被禁止同时记录. 

同时它也支持按照子域(per-domain)来区分权限的模式, 比如指定的域(进程)使用**`permissive`**模式, 而系统的其他部分执行**`enforce`**模式.
 
域(domain)是在安全规则中对于进程或者一组进程的简单标签. 所以归属于相同标签的进程将共享同样的安全规则. 

子域**`permissive`**的模式允许在已存在安全规则的系统中增加应用权限. 同时也允许对于某些服务进行权限规则的开发同时保证其余部分继续**`enforce`**模式.

-----
在android 5.0(L)版本中, 所有的android组件均处于**`enforce selinux`**模式. 4.3版本是permissive模式, 而4.4是部分enforce模式. 

同时, 在版本升级的过程中, android将 部分对于关键域(**`installd, netd, vold and zygote`**)的selinux规则扩展到所有的服务(超过60个域). 这意味着设备提供商将要理解和调整Selinux的实现来提供适配的设备.

以下为参考点:

 - 5.0版本中所有的操作都是**`enforcing`**模式.

 - init域中只有init执行程序, 不允许加入其他.

 - 任何常见的禁止操作(对于**`block_device, socket_device, default_service`**等.)表明该设备需要一个指定的域.

-----
支持文档:

查看以下文档来创建合适的selinux规则:

http://seandroid.bitbucket.org/PapersandPresentations.html

https://www.codeproject.com/Articles/806904/Android-Security-Customization-with-SEAndroid

https://events.linuxfoundation.org/sites/events/files/slides/abs2014_seforandroid_smalley.pdf

https://www.internetsociety.org/sites/default/files/02_4.pdf

http://freecomputerbooks.com/books/The_SELinux_Notebook-4th_Edition.pdf

http://selinuxproject.org/page/ObjectClassesPerms

https://www.nsa.gov/research/_files/publications/implementing_selinux.pdf

https://www.nsa.gov/research/_files/publications/selinux_configuring_policy.pdf

https://www.gnu.org/software/m4/manual/index.html

-----
