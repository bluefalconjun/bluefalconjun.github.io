
###[**Verified Boot**](http://source.android.com/security/verifiedboot/index.html)

-----
> **Introduction**

-----
android 4.4及以后的版本支持可选的验证启动功能(**`verified boot`**). 它通过**`device-mapper-verity (dm-verity)`**的kernel功能来实现, 该功能提供基于块设备的透明完整性检查. 

**`dm-verity`**能够帮助在某些需要使用root权限的设备上的可能的权限失控问题. 该功能能够帮助android用户确认在启动一个设备时, 它的状态和上次使用完成后的状态是一致的.

拥有root权限的间谍软件通常能够从安全检查程序的检测中躲藏起来, 因为一般情况下它拥有比检查程序更高的权限, 它可以'欺骗'安全检查程序.

**`dm-verity`**功能允许你查看块设备, 和文件系统的底层的存储部分, 并确定它是否符合预期的配置. 它通过使用一份加密的hash树来完成这个目标. 通常情况下, 针对每个block(4k), 都有一个SHA256的hash值.

由于hash值是存在页面构成的树当中, 只有最上层的根'root' hash才用来被信任以检查树上的其他部分. 任何时候对blocks的修改实际上相当于破坏了整个加密hash. 下图描述了整个的结构.

![Figure 1. dm-verity hash table](http://source.android.com/security/images/dm-verity-hash-table.png)

**`Figure 1. dm-verity hash table`**

boot partition中包含有一个公钥, 该公钥由OEM在系统之外进行验证. 该公钥被用于验证root hash的签名, 并以此确认整个系统分区是受保护且未被修改的.

-----
> **Prerequisites**

-----
**如何建立verified boot flow**

为了大幅的减少处理复杂情况的风险, 一般在device中烧入一个密钥来验证kernel. 细节参考
[**`Verifying boot.`**](http://source.android.com/security/verifiedboot/verified-boot.html)

-----
**切换至基于块更新的OTA操作**

为保证**`dm-verity`**的工作, 设备必须使用基于块设备的OTA功能. 来更新整个分区. 细节参考[**`Block-Based OTAs.`**](http://source.android.com/devices/tech/ota/block.html)

**`[Tips: 如果设备已经实现dm-verity, 那么该设备的OTA必须是基于block进行的, 否则会造成OTA/增量OTA之后的dm-verity失败.]`**

-----
**配置dm-verity**

当切换至基于块设备的更新后, 集成最新的android kernel或者使用已指向相关源的kernel并在相关的配置选项中**`CONFIG_DM_VERITY`**打开dm-verity的支持. 

当使用android kernel时, dm-verity将在kernel编译的过程中打开. 实现细节请参考[**`Implementing dm-verity.`**](http://source.android.com/security/verifiedboot/dm-verity.html)

-----
> **Supporting documentation**

-----
[**`Verifying Boot`**](http://source.android.com/security/verifiedboot/verified-boot.html)
[**`Block-Based OTA`**](http://source.android.com/devices/tech/ota/block.html)
[**`Implementing dm-verity`**](http://source.android.com/security/verifiedboot/dm-verity.html)
[**`cryptsetup - dm-verity: device-mapper block integrity checking target`**](https://gitlab.com/cryptsetup/cryptsetup/wikis/DMVerity)
[**`The Chromium Projects - Verified Boot`**](http://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot)
[**`Linux Kernel Documentation: verity.txt`**](http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=blob;f=Documentation/device-mapper/verity.txt)

