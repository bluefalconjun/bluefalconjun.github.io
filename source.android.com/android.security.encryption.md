
###[**Full Disk Encryption**](http://source.android.com/security/encryption/index.html)

> **What is full disk encryption?**

全局磁盘加密, 是使用密钥对android设备上的所有用户数据进行加密的操作. 当设备被加密后, 所有用户产生的数据在写入到磁盘前均会被自动加密, 并在读出到使用的进程前被自动解密.

> **What we’ve added for Android 5.0**

 - 加入了快速加密方式, 它将仅对/data分区的用户数据进行加密, 这将加快第一次启动时的时间. 目前只有ext4 和 f2fs 文件系统格式支持快速加密.
 - 增加了**`forceencrypt`**标识位来提供第一次启动时的加密支持.
 - 增加了基于硬件的存储, 它可以使用TEE环境中的签名属性(类似[**`TrustZone`**](http://www.openvirtualization.org/open-source-arm-trustzone.html)). 细节请查看[**`存储密钥`**](http://source.android.com/security/encryption/index.html#storing_the_encrypted_key)章节. 

注意: 升级到android 5.0的设备在进行出厂初始化重置后, 加密的数据可能会变成未加密. 而新的android 5.0设备在第一次启动时如果进行了加密设置, 那么将无法返回至未加密状态.

> **How Android full disk encryption works**

android全局磁盘加密基于[**`dm-crypt`**](https://www.kernel.org/doc/Documentation/device-mapper/dm-crypt.txt)来实现. dm-crypt是linux kernel基于块设备的一项功能. 基于此, 对eMMC和类似模型的闪存芯片的加密, 是因为它们对kernel来讲是块设备. 基于NAND的闪存芯片上的YAFFS文件系统不能进行全局加密操作.

全局磁盘加密的算法是基于([**`CBC`**](http://searchsecurity.techtarget.com/definition/cipher-block-chaining))和ESSIV:SHA256的128位[**`AES`**](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). 主密钥通过调用openssl库来进行128位AES加密. 必须至少使用128位(256可选).

注意: 设备提供商(OEM)可以使用128位或更高的长度对主密钥进行加密.

在Android 5.0发布版本中, 有以下4种加密方式(android 6.0增加了指纹验证):

 - 缺省
 - PIN
 - 密码
 - 图形

第一次启动过程中, 设备将创建一个随机的128位主密钥, 并将其同缺省密码进行hash, 然后单独存储. 缺省密码为"**`default_password`**". 同时,该hash值也通过TEE(类似TrustZone)中的签名hash来对主密钥进行加密.

缺省密码的定义在AOSP项目的[**`cryptfs.c`**](https://android.googlesource.com/platform/system/vold/+/master/cryptfs.c#84)文件中.

当用户在设备上设置PIN/密码时, 只有128的主密钥被重新加密和存储(用户PIN/密码的更改并**不**触发用户数据的重新加密).  [**`设备管理`**](http://developer.android.com/guide/topics/admin/device-admin.html)的权限必须由PIN/图形/密码的限制策略来决定.
例子: 无密码的android设备不允许设置VPN.

加密操作由i`n`it/`vold`来管理. init调用`vold`, `vold`设置相关属性来触发`init`中的事件. 系统中的其他组件同样监听这些属性. 并会根据属性值来进行类似 状态报告/密码提示输入/致命错误时的设备重置 等操作任务. `vold`中为了使用加密功能, 系统将使用命令行工具`vdc`的`crypt`命令:`checkpw, restart, enablecrypto, changepw, cryptocomplete, verifypw, setfield, getfield, mountdefaultencrypted, getpwtype, getpw, 和 clearpw.`

在加密/解密/格式化 /data分区的时候, /data分区不能被挂载. 然和为了显示用户界面(UI), framework需要启动, 而framework需要 /data 分区来运行. 为了解决这个冲突, 在此时使用一个临时文件系统来挂载在/data上. 这允许android来提示密码输入/显示进度/建议格式化data分区. 这将在后续操作中增加部分限制, 由于系统从临时文件系统切换到真实的/data分区时, 所有基于临时文件系统打开文件的进程都必须被关闭. 并在新的/data分区挂载时重新启动. 为达到这一操作, 所有android中的服务(`service`)必须被归组在以下三种列表中: `core`, `main`, `late_start`.

 - core: 启动后从不停止运行.
 - main: 基于磁盘密码的输入, 可以停止运行并重启.
 - late_start: 在停止运行后, 仅当/data分区被加密和挂载后才开始运行.

**注意: 该项分组机制决定某些供应商私有的service应该被加入组的策略.**

为触发以上的操作, `void.decrypt`属性被设置为不同的[类型](https://android.googlesource.com/platform/system/vold/+/master/cryptfs.c). 用以停止/重启相关的服务. 支持的init命令有:

 - class_reset: 停止相关服务, 并允许该服务在class_start中启动.
 - class_start: 重启服务.
 - class_stop: 停止服务, 并对其设置**`SVC_DISABLED`**标识, 该服务将不会被class_start重启.

参考: [**`init.rc`**](https://android.googlesource.com/platform/system/core/+/master/rootdir/init.rc#511)

> **Flows**

设备磁盘加密有四种不同流程, 设备仅被加密一次,然后进行正常的启动流程.

 - 对未加密的设备进行加密:
  - 对新设备进行强制加密: 第一次启动时强制进行加密(基于adnroid L(5.0)开始).
  - 对已使用的设备进行加密: 由用户发起加密过程(Android K(4.4)和之前版本).

 - 启动加密的设备:
	 - 无密码启动加密设备: 启动一个未设置密码的加密设备(android 5.0或之后的版本).
	 - 使用密码启动加密设备: 基于设置的密码来启动加密的设备.

 在以上的情况中, 设备可能挂载 /data失败.

**Encrypt a new device with `forceencrypt`**

这是android 5.0设备首次启动的标准做法.

 1. 在存在`forceencrypt`标识的情况下来侦测未加密的文件系统.
	 /data分区未加密, 由于`forceencrypt`标识的要求, 必须对其加密. 卸载/data分区.
	 
 2. 开始加密 /data分区.
	`vold.decrypt = "trigger_encryption"`触发init.rc进行加密, 然后回调至void进行/data加密动作. 该项操作中没有密码设置(因为是新设备, 此时还没有密码设置).

 3. 挂载tmpfs
	vold挂载一个临时文件系统的/data(使用`ro.crypto.tmpfs_options`属性的配置), 并设置`vold.encrypt_progress`属性为0. vold完成了临时的/data分区后将设置属性为 `vold.decrypt : trigger_restart_min_framework`.

 4. 启动framework来显示进度
	 由于初次启动的设备/data分区下是没有数据来加密的, 进度条会因为加密的速度过快而无法显示. 查看关于[**`加密已使用的设备`**](http://source.android.com/security/encryption/index.html#encrypt_an_existing_device)处理UI的详细信息.

 5. 当/data加密完成后, 关闭framework.
	vold设置vold.decrypt为trigger_default_encryption来启动缺省的加密服务(这将启动上面描述中的缺省加密用户数据操作). trigger_default_encryption将检查加密类型, 来查看/data是应该有/无密码进行加密. 因为自android 5.0起, 设备将在第一次启动时被加密, 此时应当没有密码设置, 此时我们可以对/data进行解密, 并且挂载/data分区.

 6. 挂载/data
	init进程将卸载挂载在RAMDisk上临时文件的/data分区, 并重新挂载真实的/data分区.

 7. 重新启动framework
	 设置vold来trigger_restart_framework, 这将继续进行常规启动流程.

**Encrypt an existing device**

该流程发生在用户选择对当前未加密的设备进行加密, 它对于android K和升级至Android L版本的设备是通用的.

该流程是由用户操作发起的, 它由代码的"内部加密"来描述. 当用户操作选择了进行加密时, UI将保证电池电量水平并且已经连接至充电器. 这样才能保证有充足的电量来完成加密操作.

警告: 如果设备在加密完成之前因为电量问题关机, 那么/data中的数据将成为部分加密的状态, 此时设备必须被恢复出厂设置, 所有的用户数据都将丢失.

为了完成内部加密的操作, vold将启动一个循环来读取实际块设备每个扇区, 然后将其写入至加密块设备. vold在读取和写入扇区前会对其检测是否已经使用. 这将帮助新设备或者是数据量少的设备能够加快加密的过程. 

设备此时的状态: 设置ro.crypto.state = "unencrypted"并且执行on nonencrypted init trigger来继续启动过程.

 [**剩余内容, 略**](http://source.android.com/security/encryption/index.html#encrypt_an_existing_device)
 
[**`Starting an encrypted device with default encryption`**](http://source.android.com/security/encryption/index.html#starting_an_encrypted_device_with_default_encryption)


[**`Starting an encrypted device without default encryption`**](http://source.android.com/security/encryption/index.html#starting_an_encrypted_device_without_default_encryption)

**Failure**

设备启动时对加密分区的解密失败可能有多种原因引起. 标准的启动步骤如下:

 1. 在有密码的情况下检测到已加密的设备分区.
 2. 挂载tmpfs.
 3. 启动framework来显示UI提示输入密码.

但在framework启动后, 可能发生以下错误:

 - 密码正确但是无法解密分区.
 - 用户输入错误密码超过指定次数(30).

如果以上的错误无法解决, 建议用户进行恢复出厂设置.

**`剩余内容, 略`**

> **Storing the encrypted key**

对分区进行加密使用的密钥存储在加密区间中. 这通过基于TEE签名机制的硬件支持的环境来实现. 在早前版本中, android使用基于用户密码而产生的密钥对主密钥进行加密, 并且单独存放. 为了对该方式进行加强(密码被破解/离线攻击), android扩展了这个算法, 使用基于存储的TEE密钥对用户密码相关的密钥进行Scrypt加密. 这样产生的相关签名会成为另一个基于应用的不同长度的密钥. 用这个密钥对主密钥进行加密/解密. 为了产生这个密钥:

1. 随机生成16字节的磁盘加密密钥(DEK,即主密钥)和16字节的盐值.
2. 将用户输入的密码和盐值作为Scrypt的输入,产生一个32字节的临时密钥IK1.
3. 将临时密钥IK1及"0"填充到硬件私钥中. 例如,如果硬件使用RSA加密算法, 其私钥大小为2048位, IK1表示为: 00 || IK1 || 00 ⋯ 00, 即1字节的"0", 32字节的IK1, 233字节的"0", 一共256字节, 2048位.
4. 可信执行环境对填充的IK1进行签名, 得到256字节的临时密钥IK2.
5. 将IK2和盐值作为Scrypt的输入, 产生32字节的临时密钥IK3.
6. 将IK3的前16字节作为AES-CBC算法的加密密钥, IK3的后16字节作为初始向量(Initialization Vector, IV).
7. 使用AES-CBC算法对主密钥进行加密. 

参考实现图:

> **Changing the password**

当用户选择在设置中修改或移除密码的时候, UI发送命令cryptfs changepw到vold中, vold重新加密带有新密码的磁盘主密钥.

> **Encryption properties

vold和init通过设置属性来进行互相通讯. 查看可用的加密相关的属性列表:

[加解密相关属性](http://source.android.com/security/encryption/index.html#encryption_properties)

[init操作](http://source.android.com/security/encryption/index.html#init_actions)