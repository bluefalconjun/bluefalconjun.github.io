
###[**Security**](http://source.android.com/security/index.html)


####Security

-----
**`android`**作为现代的开放移动平台, 其中运行了使用各类硬件和软件栈的应用, 使用通过平台向用户提供创新和有价值的服务. 为了保护其中的内容, 平台必须提供能够保障用户安全的 应用/数据/设备/网络环境. 

为开放平台提供安全保障, 需要健壮的安全体系架构和严格的安全检查. **`android`**设计时已经考虑为所有用户提供: 安全保护的多级性, 安全可灵活配置的平台. 

可以参见[Security Updates and Resources](http://source.android.com/security/overview/updates-resources.html)来获取安全问题的报告和更新流程.

**`android`**被设计为开发者友好的系统. 安全控制功能的设计以尽量减少开发者负担为方向.  安全相关的开发者能够依赖灵活的安全控制功能来完成工作. 低安全相关的开发者能够被默认保护在安全状态.

**`android`**同时被设计为用户友好的系统. 用户可以清楚的了解当前程序如何工作, 并且能够完全控制程序的工作. 

这个设计包含了预期攻击者使用标准的攻击方法, 例如社会工程学的方法,诱导用户安装恶意软件. 或者攻击者通过攻击**`android`**设备上的第三方软件进行攻击. 

**`android`**被设计为尽量减少这类攻击的可能性, 并且在攻击真的成功时尽量降低其危害.

该文档提供**`android`**安全程序的大纲, 描述**`android`**安全系统的架构基本点, 并处理大部分常见的系统架构和安全分析问题. 

该文档专注于讨论**`android`**核心平台的安全功能点. 不会对具体特定的应用安全问题进行讨论. 推荐使用 创建/发行 **`android`**设备的最佳实践来进行参考, 开发**`android`**应用相关内容不在这个范围之内.

-----
> **Background**

-----
**`Android`**为所有设备提供开放平台和应用环境.

以下章节描述了**`Android`**平台上的安全功能. 该图概括了软件组件并且定义了它存在于**`android`**软件栈的层次. 每个组件都能假定其底层的支持组件是完全安全的. 

除了小部分**`Android OS`**的代码以**`root`**权限运行. 其他所有运行在**`Linux Kernel`**上的代码均被应用沙盒严格的限制.

![android software stack](http://source.android.com/security/images/android_software_stack.png)

**`android software stack`**

-----
常见**`Android`**平台由以下的模块构成:

 - **`Device Hardware:`** Android可以在多种类型的硬件设备上运行, 包括智能手机, 平板, 机顶盒. 它是处理器无关的系统. 但它会引入某些高级硬件特定的安全性能例如: **`ARM v6 eXecute-Never`** 的支持.

 - **`Android Operating System:`** 基于linux kernel核心上的android操作系统. 所有的设备资源例如摄像头, GPS, 蓝牙, 通讯, 网络都通过该系统进行访问.

 - **`Android Application Runtime:`** Android应用通常使用**`java`**语言编写, 并运行在**`Dalvik(ART)`**虚拟机上. 同时, 包括android核心服务的许多应用都是**`native`**应用(**`linux process`**), 或者以**`native`**库的方式来提供支持(linux library). 虚拟机和native服务均在同样的安全环境中运行, 受限于在同样的应用沙盒中.  每个应用均使用文件系统上的独占空间, 来进行数据库或者原始文件的读写. 

-----
Android应用扩展了android系统的核心功能. 有两种主要的应用源:

 - **`Pre-Installed Applications`**: android设备预装整套的应用集和, 包含有电话/邮件/日历/浏览器/通讯录等. 这些应用同时为用户和其他第三方应用提供服务. 预安装应用包含在AOSP开发代码中, 或者由平台OEM为特定设备进行开发.

 - **`User-Installed Applications`**: android提供开放的开发环境,对第三方应用开发进行支持. **`Google Play`**提供给用户数十万的应用使用.

-----
**`Google`**对所有兼容的android设备提供基于云端的服务, 主要内容包括:

 - **[Google Play](https://play.google.com/store)**: **`Google Play`**是一套服务集合, 用户通过它在android设备或者web上来查找/安装/购买应用. 它帮助开发者达到用户需求. 同时它也提供评价预览功能, 应用授权([license verfication](https://developer.android.com/guide/publishing/licensing.html)), 应用安全检查和其他安全功能.
 
 - **`Android Updates`**: android更新服务为设备提供新功能或者安全功能更新, 包含通过web的更新或者OTA更新.

 - **`Application Services`**: 基于Frameworks的服务允许android应用使用云端服务的功能, 例如备份([back up](https://developer.android.com/guide/topics/data/backup.html))程序数据/设置, 或者云到端的([GCM](https://developers.google.com/cloud-messaging/))消息机制.

以上的google提供的服务并不包含在AOSP项目中, 但它们同绝大多数的android设备的安全性相关. 参见相关的安全文档“Google Services for Android: Security Overview”.

-----

> **Security Program Overview**

-----
在android开发过程中, android从安全模块的需求方逐渐演化成为安全程序的提供方. android系统开发team从其他的各类设备和服务方吸取各种安全问题和经验, 并加入到当前的android系统中, 来创建android安全生态环境.

Android安全程序的关键部分有以下几点:

 - **`Design Review`**: **设计评审**. 在项目开发初期, android安全设计列出可用的安全模型. 每个主要的安全功能都由相关人员进行评审, 之后将对应的安全控制机制加入到系统中.

 - **`Penetration Testing and Code Review`**: **渗透测试/代码评审**. 在平台开发过程中, Android建立并开放部分源代码, 并交付给多种安全性评审. 这些评审包括Android安全小组, Google的信息安全程序组,  和第三方的独立安全顾问. 评审的目标是在平台代码完全开源前确定弱点和可能的漏洞, 并且模拟发布后由外部的安全专家进行的类型分析.

 - **`Open Source and Community Review`**: **开源/社区评审**. AOSP项目为所有有兴趣的团体提供安全评审的公告板. 同时Android选用经过了外部严格安全评审的系统模块, 例如 linux kernel. Google Play也提供了专为开发者/厂家提供应用信息到终端客户的论坛. 

 - **`Incident Response`**: **事件响应**. 即使存在以上的预防措施, 安全问题仍然会在产品供货后出现, 因此Android项目建立了一个全面的安全响应机制. 全职的Android安全小组监视android上/和通用平台上的安全漏洞问题. 通过查看已泄漏的漏洞信息, android安全小组使用快速响应机制来保护所有android用户, 使得已知的漏洞造成最少的潜在伤害. 这些基于云端的响应机制包括进行系统安全更新(OTA), 从Google Play中下架特定应用或者从特定地区的Android设备中移除应用. 

-----

> **Platform Security Architecture**

-----
Android力求成为最安全和易用的移动设备操作系统, 它通过重新设计/实现以下的系统安全控制来完善这个目标:

 - 保护用户数据.
 - 保护系统资源(包括网络连接).
 - 提供应用隔离

为达成以上目标, android提供以下的关键安全功能:

 - 基于linux kernel的OS级别的安全功能.
 - 为所有应用实现强制性的沙盒机制.
 - 安全的进程间通讯. 
 - 应用签名
 - 应用定义/用户批准的权限管理机制.