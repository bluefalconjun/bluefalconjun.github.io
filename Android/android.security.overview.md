

###[**Security**](http://source.android.com/security/index.html)

**Security**

android作为现代的开放移动平台,在其中运行的应用使用各类硬件和软件栈,通过平台向用户提供创新和有价值的服务. 为了保护其内容, 平台必须提供一套保障用户安全的应用/数据/设备/网络环境. 

为一个开放平台进行安全保障, 需要健壮的安全体系架构和严格的安全检查. Android被设计为为所有平台用户提供安全保护的多级安全性的灵活平台. 获取安全问题的报告和更新流程相关. 参见[Security Updates and Resources](http://source.android.com/security/overview/updates-resources.html);

Android被设计为方便开发者的系统. 安全控制功能被设计为尽量减少开发者的负担为方向.  安全相关的开发者能够依赖灵活的安全控制功能来完成工作. 低安全相关的开发者能够被默认保护在安全状态.

Android同样被设计为考虑用户的系统. 用户对程序如何工作是可见的, 并且完全控制程序的工作. 这个设计包含了预期攻击者使用标准的攻击方法, 例如社会工程学的方法,诱导用户安装恶意软件. 或者是通过攻击Android设备上的第三方软件进行攻击. Android被设计为尽量减少这类攻击的可能性, 在攻击成功后尽量降低其危害.

该文档提供Android安全程序的大纲, 描述Android安全系统的架构基本点, 并回答大部分常见的系统架构和安全分析的问题. 文档专注于Android核心平台的安全功能点. 不会对具体特定的应用安全问题进行讨论. 推荐建立/发行Android设备的最佳实践, 开发Android应用相关内容不在这个范围之内.



> **Background**


Android为所有设备提供一个开放平台和应用环境.


下面的章节描述了Android平台上的安全功能. 图1概括了安全组件并且定义了它存在于andorid软件栈的层次. 每个组件都能假定其下层的支持组件是完整安全的. 除了一小部分Android OS的代码以root权限运行. 其他所有Linux Kernel上的代码均被应用沙盒严格的限制.

![android software stack](http://source.android.com/security/images/android_software_stack.png)

最主要的Androd平台构成模块为以下部分:

 - **Device Hardware**: Android在各种类型的硬件设备上运行, 包括智能手机, 平板, 机顶盒. Android是处理器无关的系统. 但它会引入某些高级硬件指定的安全性能例如: ARM v6 eXecute-Never的支持.

 - **Android Operating System**: 核心的android操作系统被构建于linux kernel之上. 所有的设备资源例如摄像头, GPS数据, 蓝牙功能, 通讯功能, 网络功能都通过该系统进行访问.

 - **Android Application Runtime**: Android应用通常通过java语言编写, 并运行在Dalvik(ART)虚拟机上. 同时, 许多应用包括android核心服务都是native的应用(linux process), 或者包含native库支持(linux library). 虚拟机和native服务均在同样的安全环境中运行, 包含在同样的应用沙盒中.  每个应用均使用文件系统上的独占空间, 来进行数据库或者原始文件的读写. 


Android应用扩展了android系统的核心功能. 有两种主要的应用源:

 - **Pre-Installed Applications**: android设备包括一套预装的应用集, 包含有电话/邮件/日历/浏览器/通讯录等. 这些应用同时为用户和其他第三方应用提供服务. 预安装应用包含在AOSP开发代码中, 或者由平台OEM为特定设备进行开发.

 - **User-Installed Applications**: android提供一个开放的开发环境,对第三方应用进行支持. Google Play提供给用户数十万的应用以使用.

Google对所有兼容的andorid设备提供基于云端的服务, 主要服务为:

 - **[Google Play](https://play.google.com/store)**: Google Play是一套服务集合, 它提供用户在android设备或者web上来查找/安装/购买应用. 它帮助开发者达到用户的预期目标. 同时它也提供评价预览, 应用授权([license verfication](https://developer.android.com/guide/publishing/licensing.html)), 应用安全检查和其他安全功能.
 
 - **Android Updates**: android更新服务为设备提供新功能或者安全功能更新, 包含通过web的更新或者OTA.

 - **Application Services**: Frameworks服务允许android应用使用云端服务的功能, 例如备份([back up](https://developer.android.com/guide/topics/data/backup.html))程序数据/设置, 或者云到端的([GCM](https://developers.google.com/cloud-messaging/))消息机制.

以上的google提供的服务并不包含在AOSP项目中, 但它们同绝大多数的android设备的安全性相关. 参见相关的安全文档“Google Services for Android: Security Overview”.


> **Security Program Overview**

在开发android过程中, android由健壮的安全模块的需求方逐渐演化成为安全程序的提供方. android系统开发team从其他的各类设备和服务方吸取各种安全问题和经验, 并加入到当前的android系统中, 来创建android安全程序.

Android安全程序的关键部分有以下几点:

 - **Design Review**: 设计评审. The Android security process begins early in the development lifecycle with the creation of a rich and configurable security model and design. Each major feature of the platform is reviewed by engineering and security resources, with appropriate security controls integrated into the architecture of the system.

 - **Penetration Testing and Code Review**: During the development of the platform, Android-created and open source components are subject to vigorous security reviews. These reviews are performed by the Android Security Team, Google’s Information Security Engineering team, and independent security consultants. The goal of these reviews is to identify weaknesses and possible vulnerabilities well before the platform is open sourced, and to simulate the types of analysis that will be performed by external security experts upon release.

 - **Open Source and Community Review**: The Android Open Source Project enables broad security review by any interested party. Android also uses open source technologies that have undergone significant external security review, such as the Linux kernel. Google Play provides a forum for users and companies to provide information about specific applications directly to users.

 - **Incident Response**: Even with all of these precautions, security issues may occur after shipping, which is why the Android project has created a comprehensive security response process. A full-time Android security team constantly monitors Android-specific and the general security community for discussion of potential vulnerabilities. Upon the discovery of legitimate issues, the Android team has a response process that enables the rapid mitigation of vulnerabilities to ensure that potential risk to all Android users is minimized. These cloud-supported responses can include updating the Android platform (over-the-air updates), removing applications from Google Play, and removing applications from devices in the field.