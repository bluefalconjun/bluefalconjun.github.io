
###[**SELinux concepts**](http://source.android.com/security/selinux/concepts.html)

> **Mandatory access control**

Security Enhanced Linux (SELinux), 是linux操作系统上的强制访问控制模型**`mandatory access control (MAC)`**的实现. 作为MAC系统, 它的实现不同于常见的linux自主访问控制**`discretionary access control (DAC)`**模型. 在DAC系统中, 存在有拥有者的概念, 它基于特定资源所有权的用户来定义相关的访问权限. 这是比较粗放的管理模式, 并且存在因为漏洞造成的权限升级问题. 而MAC系统, 建立的是以集中控制方式来定义和管理所有的访问尝试.

selinux实现了部分的Linux Security Module(LSM)安全模型框架, 它组织了不同的kernel objects并识别管理所有对于objects的操作. 当这些操作将被执行时, LSM钩子程序将被调用来决定这些操作是否将被执行, 它将基于一个单独存储的安全object来查询相关信息. selinux提供LSM钩子程序的实现机制, 并且管理这些安全objects, object将同它自己的规则绑定, 并决定其他操作的可行性.

在同其他android安全限制的联动过程中, android访问控制规则能够极大的限制在开放设备/帐号上的操作造成的损失. 使用android相关的自主/强制访问控制规则能够提供一套保证软件只在它能被允许的最小权限下执行的结构. 这能够减轻攻击造成的损失并且减少进程覆盖/数据流失的可能性.

从android 4.3开始, selinux在传统的DAC环境上提供了MAC控制. 例如, 如果需要对raw块设备进行写操作的软件, 必须以root用户帐号运行. 在传统DAC环境中, 该获取root权限的应用急拥有了对每个raw快设备读写的权限. 但是在selinux控制下, 这些设备可以以标识的方式加以区分, 即使拥有root权限的进程也只能对特定标识的设备进行指定规则的访问. 这样该应用就能被避免对其他块设备之外的数据和系统设置进行修改.

查看[**`Use Cases`**](http://source.android.com/security/selinux/implement.html#use_cases)来参考更多相关的例子和实现.

> **Enforcement levels**

熟悉以下的语法来理解selinux如何被定义实现为不同级别的控制强度.

 - Permissive(宽容) - selinux安全规则并不强制, 只是记录.
 - Enforcing(强制) - selinux规则强制执行, 并且记录. 错误将返回EPERM代码.

以上的选择是二选一的, 它将帮助确定对应的规则是否真实生效, 或者是仅允许获取潜在的失败信息. Permissive在开发selinux规则时是很有帮助的.

 - Unconfined(开放) - 禁止指定任务最低的规则, 在开发时提供临时的禁止隔离. 在AOSP项目内容外不应该被使用.
 - Confined(封闭) - 为服务制定的可配置的写规则. 该规则应当明确定义那些行为是允许的.

开放规则能够帮助在android开发中快速实现selinux. 它们适合大部分的root权限的应用. 但是完成后必须被转换成为封闭规则, 按照指定应用在任何时候都需要的对应限制资源进行修改.

理想情况下, 所有的规则必须同时是enforcing/confined模式. 在强制模式下开放规则将屏蔽一些潜在的违规操作, 该操作在permissive模式使用confined规则会被记录在案. 因此, andorid强烈建议所有的设备提供商都实现真正的封闭规则.

> **Labels, rules and domains**

selinux基于标签(标识)来匹配操作和规则. 标签定义了那些操作是允许的. Socket/files/process 等在selinux中均被定义有标签. selinux的判定基于定义在这些object上的标签和该object上设定的进行交互的规则来完成. 
在selinux中, 标签的格式是: **`user:role:type:mls_level`**. 其中type是最为主要的控制访问规则的字段. 它和其他字段一起构成整个规则. 每个objects都会被映射成为classes和不同的访问type, 每个class代表不同的权限.

规则的语法如下:
**`allow domains types:classes permissions;`**

 - Domain - 进程或者进程组的标签. 也称之为域名类型, 因为它代表了进程的类型.
 - Type - object或者object组的标签(例如. 文件/socket).
 - Class - 对object(文件/socket)的访问类型
 - Permission - 访问方式的执行方法(例如. 读/写)

按照以上的语法, 标准的实现例子结构:

**`allow appdomain app_data_file:file rw_file_perms;`**

这句selinux规则表明, 所有application域名下的进程(组)被允许对以app_data_file标签标识的文件具有读/写权限.  需要注意的是, 这句规则的实现, 依赖于定义在 [**`global_macros`**](https://android.googlesource.com/platform/system/sepolicy/+/master/global_macros)文件中的宏定义. 同时可以在[**`te_macros`**](https://android.googlesource.com/platform/system/sepolicy/+/master/te_macros)文件中查看其他定义. 相关定义文件存在于AOSP项目的[**`external/sepolicy`**](https://android.googlesource.com/platform/system/sepolicy/+/master)源代码中. 这些宏定义用来规范classes/permissions/rules的分组, 并且用来尽可能的避免模糊的定义, 造成访问禁止或者是权限不当.

为了方便于在避免在规则中分别列出domains/types, 可以通过属性(**attribute**)的方式来定义/使用一组domains/types. attribute是一个简单的domains/types(组)的命名. 每个domain/type可以和任意数量的attributes关联. 当写下的规则指定了attribute名时, 它将自动扩展到具有相同(**attribute**)属性的domains/types的列表中. 例如: domain属性同所有process domain是关联的, file_type属性同所有file类型属性是关联的.

以上定义的语法可以用来创建最基础的avc规则. 语法规则如下:

**`<rule variant> <source_types> <target_types> : <classes> <permissions>`**

该规则指出当 任何指定为source_types的主体 试图对某个拥有target_types标签并且符合classes类的object进行操作时, 它仅具有permissions允许(rule来定义)的权限. 
最常见的实现是allow(允许)的例子:

**`allow domain null_device:chr_file { open };`**

这条规则 **允许** 任何拥有**`domain`**属性关联的**process**, 可以对一个 **是** **`chr_file(character device file)`** **并且** 拥有 **`target_type`** 标签为**`null_device`** 的 **object** 进行**`open`** 的权限. 

该规则可以扩展为以下包含更多的权限:

**`allow domain null_device:chr_file { getattr open read ioctl lock append write};`**

在和基本定义的结合之后, 如**`domain`**属性将同所有的进程相关联, **`null_device`**是字符设备 **`/dev/null`** 的标签, 该规则可以基本被理解为: 允许所有进程读/写 **`/dev/null`** 设备.

domain通常和一个进程(process)相关, 同时该进程具有自己的标签(label).

例如一个正常的android app将有它自己的process来运行(UID), 同时默认的标签为**`untrusted_app`**, 该标签将限制它有指定的权限.
Platform app(平台应用)是被编译进入系统分区的, 它将拥有不同的标签, 并且可以扩展为不同的权限集合. System UID apps(系统应用)是android系统的核心部分之一, 它将拥有**`system_app`**标签, 是另一种权限集合.

对以下的标准标签的访问是绝对不允许直接对**`domain`**允许的, 相反, 必须要为以下object创建更加特定的类型来控制访问方式.

 - socket_device 
 - device 
 - block_device 
 - default_service 
 - system_data_file
 - tmpfs

以下是部分adbd在selinux规则中的[**`例子`**]:

![adbd.selinux](https://raw.githubusercontent.com/bluefalconjun/bluefalconjun.github.io/master/Android/pic/adbd.selinux.png)