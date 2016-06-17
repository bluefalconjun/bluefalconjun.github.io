
###[**SELinux concepts**](http://source.android.com/security/selinux/concepts.html)

-----
> **Mandatory access control**

-----
Security Enhanced Linux (SELinux), 是linux操作系统上的强制访问控制模型**`mandatory access control (MAC)`**的实现. 

作为MAC系统, 它的实现不同于常见的linux自主访问控制**`discretionary access control (DAC)`**模型. 

在DAC系统中, 使用(**`owner`**)拥有者的概念, 它基于指定资源所有权的用户/组来定义相关的访问权限. 这是比较粗放的管理模式, 并且存在因为安全漏洞而造成的权限升级问题. 

而MAC系统, 建立的是使用集中控制处理的方式来定义和管理所有的访问尝试.

Selinux实现了部分Linux Security Module(LSM)安全模型框架, 它组织不同的kernel objects并识别和管理所有对于这些objects的操作. 

当这些操作将被执行时, LSM钩子程序将被调用, 并且决定这些操作是否能被执行, 它将基于一个单独存储的安全object来查询相关信息. 

selinux提供LSM钩子程序的实现机制, 并且管理这些安全objects, object将同它自己的规则绑定, 并决定其他操作的可行性.

在同其他android安全限制的联动中, android访问控制规则能够极大的限制在开放设备/帐号上的操作造成的损失. 

使用android相关的自主/强制访问控制规则(**`DAC/MAC`**)能够提供一套保证软件只在它能被允许的最小权限下执行的结构. 这能够减轻因攻击造成的损失并且减少进程覆盖/数据流失的可能性.

从android 4.3开始, selinux在传统的DAC环境上提供了MAC控制. 

-----
例如, 假设某进程需要对**`raw`**块设备进行写操作, 则进程必须以root权限运行. 

在传统DAC环境中, 该获取root权限的应用随即拥有了对每个**`raw`**快设备读写的权限. 

但是在selinux控制下, 这些设备可以用标识的方式加以区分, 即使拥有root权限的进程也只能对特定标识的设备进行指定规则的访问. 

这样该应用就能被避免对该块设备之外的数据和系统设置进行修改.

查看[**`Use Cases`**](http://source.android.com/security/selinux/implement.html#use_cases)来参考更多相关的例子和实现.

**`[Tips: 设备中的相关资源区分颗粒度越细, selinux的安全性就越高.]`**
**`[Tips: 但同时对于selinux规则的编写和服务的设计/编码工作就会越复杂.]`**

-----
> **Enforcement levels**

-----
熟悉以下的语法来理解selinux如何被定义并实现为不同级别的控制强度.

 - Permissive(宽容) - selinux安全规则并不强制, 只是记录.

 - Enforcing(强制) - selinux规则强制执行, 并且记录. 错误将返回EPERM代码.

以上的选择是二选一的, 它将帮助确定对应的规则是否真实生效, 或者是仅允许捕获潜在的失败信息. 

Permissive在开发selinux规则时是很有帮助的.

 - Unconfined(开放) - 禁止指定任务最低的规则, 在开发时提供临时的禁止隔离. 在AOSP项目内容外不应该被使用.

 - Confined(封闭) - 为服务制定的可配置的写规则. 该规则应当明确定义那些行为是允许的.

开放规则能够帮助在android开发中快速实现selinux. 它们适合大部分的root权限的进程. 

但是完成后必须被转换成为封闭规则, 按照指定进程在整个生命周期中需要的对应限制资源需求进行修改.

理想情况下, 所有的规则都必须同时是enforcing/confined模式. 

在强制模式下开放规则将屏蔽一些潜在的违规操作, 该操作在permissive模式使用confined规则会被记录在案. 

因此, android强烈建议所有的设备提供商都实现真正的封闭规则.

-----
> **Labels, rules and domains**

-----
selinux基于标签(**`Labels`** 标识)来匹配操作(**`domain`**)和规则(**`rules`**). 

标签决定了何种操作是被允许的. **`socket/files/process`** 等在selinux中均被定义有标签. selinux的判定基于定义在object上的标签和该object上设定的交互规则来完成. 

在selinux中, (**`Lables`**)标签的格式是: **`user:role:type:mls_level`**. 

其中type是最为主要的控制访问规则的字段. 它和其他字段一起构成整个规则. 每个objects都会被映射成为classes和不同的访问type, 每个class和type共同构成不同的权限.

(**`Rules`**)规则的语法如下:

**`allow domains types:classes permissions;`**

 - Domain - 进程或者进程组的标签. 也称之为域名类型, 因为它代表了进程的类型.

 - Type - object或者object组的标签(例如. 文件/socket).

 - Class - 对object(文件/socket)的访问类型

 - Permission - 访问方式的执行方法(例如. 读/写)

按照以上的语法, 标准的实现例子:

**`allow appdomain app_data_file:file rw_file_perms;`**

该规则表明, 所有application域名下的进程(组)被允许对以app_data_file标签标识的文件具有读/写权限.  

需要注意的是, 这句规则的实现, 依赖于定义在 [**`global_macros`**](https://android.googlesource.com/platform/system/sepolicy/+/master/global_macros)文件中的宏定义. 

同时可以在[**`te_macros`**](https://android.googlesource.com/platform/system/sepolicy/+/master/te_macros)文件中查看其他定义. 相关定义文件存在于AOSP项目的[**`external/sepolicy`**](https://android.googlesource.com/platform/system/sepolicy/+/master)源代码中. 

这些宏定义用来规范classes/permissions/rules的分组, 并且用来尽可能的避免模糊的定义, 造成访问禁止或者是权限不当.

-----
为了方便于在避免在规则中重复列出domains/types, 可以通过属性(**attribute**)的方式来定义/使用一组domains/types. 

attribute是一个简单的domains/types(组)的重命名. 每个domain/type可以和任意数量的attributes关联. 

当定义规则指定了attribute名时, 它将自动扩展到具有相同(**attribute**)属性的domains/types的列表中. 

例如: domain属性同所有process domain是关联的, file_type属性同所有file类型属性是关联的.

-----
以上定义的语法可以用来创建最基础的avc规则. 语法规则如下:

**`<rule variant> <source_types> <target_types> : <classes> <permissions>`**

该规则指出当 任何指定为source_types的主体 试图对某个拥有target_types标签并且符合classes类的object进行操作时, 它仅具有permissions允许(rule来定义)的权限. 

最常见的实现是allow(允许)的例子:

**`allow domain null_device:chr_file { open };`**

这条规则 **允许** 任何拥有**`domain`**属性关联的**process**, 可以对一个 **是** **`chr_file(character device file)`** **并且** 拥有 **`target_type`** 标签为**`null_device`** 的 **object** 进行**`open`** 的权限. 

该规则可以扩展为以下包含更多的权限:

**`allow domain null_device:chr_file { getattr open read ioctl lock append write};`**

在和基本定义的结合之后, 如**`domain`**属性将同所有的进程相关联, **`null_device`**是字符设备 **`/dev/null`** 的标签.

该规则可以简单理解为: 允许所有进程读/写 **`/dev/null`** 设备.

-----
**`domain`**通常和一个进程(**`process`**)相关, 同时该进程具有自己的标签(**`label`**).

例如一个正常的**`android app`**将使用它自己的process来运行(UID), 同时它的默认标签为**`untrusted_app`**, 该标签将限制它仅具有指定的权限.

**`Platform app`**(平台应用)是被编译进入系统分区的, 它将拥有不同的标签, 并且可以扩展为不同的权限集合. 

**`System UID apps`**(系统应用)是android系统的核心部分之一, 它将拥有**`system_app`**标签, 是另一种权限集合.

以下的标准标签objects的访问是绝对不允许直接对**`domain`**允许的, 相反, 必须要为以下objects创建更加特定的类型来控制访问方式.

 - socket_device 
 - device 
 - block_device 
 - default_service 
 - system_data_file
 - tmpfs

-----
[**`Tips`**]:

以下是部分系统中object domain的标签(**`labels`**)定义的[**`例子`**]:

**`user:role:type:mls_level`**

        1 /storage/emulated/legacy                    u:object_r:fuse:s0
		//legacy目录是有selinux user创建, 使用object_r role, fuse类型的object, 并具有s0的mls级别.
        2
        3 /dev/videocore                                u:object_r:video_device:s0
        4 /dev/graphics/gal3d                         u:object_r:gfx_device:s0
        5 /dev/tz                                     u:object_r:tee_device:s0
        6 /dev/tzlogger                               u:object_r:tee_log_device:s0
        7 /dev/bsm                                    u:object_r:bsm_device:s0
        8 /dev/fts                                    u:object_r:fts_device:s0
        9 /dev/mbtchar0                               u:object_r:bt_device:s0
       10 /dev/cpm                                    u:object_r:cpm_chr_device:s0
       11 /dev/snd_bt                                 u:object_r:snd_bt_device:s0
       //以上定义了kernel资源设备, 同样为selinux user创建, 使用object_r role, 类型为对应类型设备, s0级别.
       12
       13 /dev/block/mmcblk0p1                        u:object_r:fts_block_device:s0
       //emmc p1分区为rts类型的block设备.
       14 /dev/block/mmcblk0rpmb                      u:object_r:rpmb_block_device:s0
       15
       16 /dev/i2c-[0-9]                              u:object_r:i2c_chr_device:s0
       //可以使用regexp扩展缩写设备. 
       17 /dev/hidraw[0-9]                            u:object_r:hidraw_device:s0
       18
       19 /dev/rfkill                                 u:object_r:rfkill_device:s0
       20
       21 /system/bin/av_settings                     u:object_r:av_settings_exec:s0
       22 /system/bin/ethconfig                       u:object_r:ethconfig_exec:s0
       23 /system/bin/wireless_device_detect.sh       u:object_r:wireless_init_exec:s0
       24 /system/bin/mediaserver                 u:object_r:mediaserver_exec:s0
       25 /system/bin/resourcemanager                 u:object_r:resourcemanager_exec:s0
       26 /system/bin/kmsglogd                        u:object_r:kmsglogd_exec:s0
       27 /system/bin/fixdate                         u:object_r:fixdate_exec:s0
       //以上是系统进程的标签定义.
       28
       29 /system/vendor/bin/video_service              u:object_r:tee_exec:s0
       31 /system/vendor/bin/clear_crashcounter       u:object_r:clear_crashcounter_exec:s0
       32 /system/vendor/bin/install-vendor-recovery.sh u:object_r:install_recovery_exec:s0
       33 /system/vendor/bin/voicecapture             u:object_r:voicecapture_exec:s0
       34 /system/vendor/bin/cpusvc                   u:object_r:cpusvc_exec:s0
       //以上是vendor进程的标签定义.
       35
       36 /factory_setting(/.*)?                      u:object_r:factory_setting_file:s0
       37 /firmware(/.*)?                             u:object_r:firmware_file:s0
       38 /user_setting(/.*)?                         u:object_r:user_setting_file:s0
       39 /factory_data(/.*)?                         u:object_r:factory_data_file:s0
       //以上是目录的标签定义.

-----
以下是部分video process / tee 编写实现selinux规则的[**`例子`**]:

**`allow appdomain app_data_file:file rw_file_perms;`**

        1
        2 # tee domain macros
        3 file_type_auto_trans(tee, device, video_device)
        4 file_type_auto_trans(tee, socket_device, snoop_socket)
        //使用标准selinux宏扩展相关的设备文件类型.

        5 use_video_device(tee)
        6 use_tee_device(tee)
		//设置tee domain进程可以访问video/tee相关device, 此为自行编写的宏扩展.

        7 set_prop(tee, system_prop)
		//设置tee具有系统属性(system_app).
		
        8 r_dir_file(tee, factory_setting_file)
		//允许tee domain能够读取factory_setting_file标签的目录和文件.

        9 unix_socket_connect(tee, property, init)
		//允许tee domain使用unix类型socket同property/init进行通讯.

       10
       11 # tee domain rules
       12 allow  tee  self:process  { execmem };
	   //允许tee domain自身以进程的类型进行执行内存操作.

       13 allow  tee  self:capability  { sys_nice chown fowner fsetid sys_module dac_override sys_rawio};
		//允许tee domain自身使用以下的系统调用.

       14 allow  tee  tee_exec:file { execute_no_trans };
		//允许tee domain组中使用tee_exec标签的进程能够以execute_no_trans方式访问文件.

       15
       16 allow  tee  system_file:file  x_file_perms;
		//允许tee domain对具有system_file标签的文件拥有执行权限.

       17 allow  tee  sysfs:file  { write };
		//允许tee domain对使用sysfs标签的文件拥有写权限.

       18 allow  tee  shell_exec:file  rx_file_perms;
       19 allow  tee  firmware_file:dir  r_dir_perms;
       20 allow  tee  firmware_file:file  r_file_perms;
       21 allow  tee  i2c_chr_device:chr_file  rw_file_perms;
       22 allow  tee  factory_data_file:dir  r_dir_perms;
       23 allow  tee  factory_data_file:file  r_file_perms;
       24 allow  tee  tee_data_file:dir  create_dir_perms;
       25 allow  tee  tee_data_file:file  create_file_perms;
       26
       27 allow  tee  rpmb_block_device:blk_file rw_file_perms;
       28 allow  tee  block_device:dir { search };
	   //允许tee domain对块设备标签的设备的目录拥有search权限.

       29
       30 allow  tee  domain:unix_dgram_socket { sendto };
	   //允许tee domain对所有其他domain的unix_dgram_socket拥有发送(sendto)的权限.

       31
       32 recovery_only(`
       33         allow tee rootfs:file { entrypoint x_file_perms };
       34 ')
	   //该扩展定义为仅在recovery模式下生效.

       35
       36 allow  tee  device:lnk_file { create };
       37 allow  tee  cpm_chr_device:chr_file  rw_file_perms;

------