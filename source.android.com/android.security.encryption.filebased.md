
###[**File-Based Encryption**](http://source.android.com/security/encryption/file-based.html)

> **基于文件的加密**

-----
Android 7.0及以上版本提供基于文件的加密方式(**`FBE`**). 这种方式允许使用不同的密钥对不同的文件进行加密, 并且可以独立进行解密.

该文章描述如何在新设备上提供基于文件的加密方式, 以及系统应用如何高效使用新的**`Direct Boot APIs`**来提供给用户优质和安全的体验.

-----

> **Direct Boot**

-----
Android 7.0为基于文件的加密引入了新的功能, 命名为Direct Boot. 它允许加密设备直接启动至锁定屏幕的状态. 在以前的版本中, 使用全局磁盘加密(**`FDE`**)的设备, 用户需要在任何数据能够被访问前提供认证凭证(图形/密码/指纹), 来预防设备可能被进行的任何基本操作. 例如: 闹钟无法执行, 辅助功能服务无法使用, 电话无法接受通话请求, 除了最基本的紧急通话操作.

使用基于文件加密的设备和新的APIs, 应用可以了解当前加密的情况, 同时被允许执行限定范围内的操作. 这些操作能够在用户提供认证凭证前执行, 同时依然能够保护私有用户信息.

在使用FBE的设备上, 设备的每个用户对于设备应用均拥有两个存储空间:

 - 凭证加密存储区间(**`Credential Encrypted (CE)`**), 这是缺省的存储空间, 并且只有当用户解锁设备后才可用.

 - 设备加密存储空间(**`Device Encrypted (DE)`**), 该存储空间在Direct Boot模式和用户解锁设备后均可用.

这种区分能够使工作模型更为安全, 因为它允许在同一时间能够保护多个用户, 基于当前的加密可能并不基于一个单独的启动时密码.
(**`这段有点难以理解, 后期结合代码分析`**).

Direct Boot API允许加密相关的应用能够访问以上的区域. 应用的生命周期会有所改变, 当用户的**`CE`**存储空间由于在锁定屏幕情况下第一次输入认证凭证时, 它需要向应用发送通知并且应用需要进行响应. 另一种情况是工作模式中提供模式改变([**`work challenge`**](https://developer.android.com/preview/api-overview.html#android_for_work)). 运行Android 7.0的设备无论底层是否实现FBE, 都必须支持新的Direct Boot API和相关的应用生命周期. 在没有FBE支持的情况下, DE/CE的存储空间在解锁状态下均应该存在.

完整的基于文件加密的实现, 底层由Ext4文件系统支持时, 代码示例由AOSP提供. 仅当当前设备满足条件时才会被启用. 设备提供商可以根据选用的SOC情况, 自行决定调整和优化FBE的功能.

所有AOSP项目中和Direct Boot相关的应用包均已经被更新. 然而, 当设备提供商使用这个应用包的定制版本时, 一般基于direct boot相关的最小集合, 该集合提供以下服务:

 - 电话服务和拨号.
 - 输入机制来提供在锁屏界面下输入密码.

-----

> **例子和参考代码**

-----
Android提供基于文件加密的参考实现, 其中vold(**`system/vold`**)负责提供android中管理存储设备和卷标的功能. FBE向vold提供了额外的新命令, 以支持在多用户情况下对于CE/DE的密钥进行管理的功能. 另外在核心改动中, 使用kernel支持的基于[**`ext4 encryption`**](http://source.android.com/security/encryption/file-based.html#kernel-support)的加密属性, 很多系统应用包括锁屏界面和SystemUI都被修改以支持FBE和Direct Boot功能. 相关的应用包括:

 - AOSP拨号 (**`packages/apps/Dialer`**)
 - 桌面时钟(**`packages/apps/DeskClock`**)
 - 输入法(**`packages/inputmethods/LatinIME`**) *
 - 系统设置 (**`packages/apps/Settings`**) *
 - 系统界面 (**`frameworks/base/packages/SystemUI`**) *

* 系统应用将使用[**`defaultToDeviceProtectedStorage`**](http://source.android.com/security/encryption/file-based.html#supporting-direct-boot-in-system-applications)的申明属性.

可以通过在AOSP代码树中使用**`mangrep directBootAware`**命令来查看更多的使用FBE相关的应用和服务.

-----

> **依赖 Dependencies**

-----
设备必须满足以下的要求来安全使用AOSP的FBE实现.

 - **Kernel支持**ext4加密(相关的kernel config为: **`EXT4_FS_ENCRYPTION`**)
 - **Keymaster支持**1.0/2.0的HAL版本. 0.3版本的keymaster hal并不提供相关需要的属性支持, 而且不能保证加密密钥的安全保护机制.
 - **Keymaster/Keystore 和 GateKeeper**必须在可信任执行环境([**`TEE`**](http://source.android.com/security/trusty/index.html))中实现, 用以提供对DE密钥的保护, 该情况下, 未认证的系统OS(自行输入到设备上的OS)将不能简单的访问DE密钥.
 - **kernel中提供的加密性能**必须达到使用AES XTS时最低为50MB/s来保证良好的用户体验.
 - 基于硬件根的验证和验证启动机制必须绑定至keymaster的初始化过程中. 这将保证设备加密证书不会被未验证的操作系统所保护.

Note: 存储规则将指定至文件夹和所有的子文件夹. 设备提供商应当限制可能在OTA过程中被解开的目录和包含解密系统密钥的目录的内容. 大部分的内容应当存放在以证书加密的存储空间中, 而不是设备加密的存储空间中.

-----

> **实现 Implementation**

-----
第一步也是最关键的一步, 类似闹钟/电话/辅助工具等功能的应用必须参照[**`Direct Boot`**](https://developer.android.com/preview/features/direct-boot.html)开发文档设置**`android:directBootAware`**属性.

**Kernel支持**

基于文件加密功能在AOSP中的实现使用Linux 4.4 kernel中的ext4加密特性. 推荐方案是使用基于4.4或更新版本的kernel. 同时, ext4加密也被移植到了3.10 kernel中. 该部分已经存在android标准开发仓库和相关支持的Nexus kernel中.

AOSP 的 kernel/common git仓库的 android-3.10.y 分支中已经提供了一个较好的开始点, 可以被设备提供商在实现这些功能到私有的设备kernel中进行参考. 然而, 这仍然需要从最新的稳定linux kernel(当前**`linux-4.6`**)中将大部分的ext4/jbd2的补丁结成进入到私有kernel中. Nexus设备的kernel已经包含了其中的许多补丁.es.

    Device:  
    Kernel
    
    Android Common:	
    kernel/common android-3.10.y ([git](https://android.googlesource.com/kernel/common/+/android-3.10.y))
    
    Nexus 5X (bullhead):
    kernel/msm android-msm-bullhead-3.10-n-preview-2 ([git](https://android.googlesource.com/kernel/msm/+/android-msm-bullhead-3.10-n-preview-2))
    
    Nexus 6P (angler):
    kernel/msm android-msm-angler-3.10-n-preview-2 ([git](https://android.googlesource.com/kernel/msm/+/android-msm-angler-3.10-n-preview-2))

注意以上的每一个kernel都是用一份移植到3.10的拷贝. 基于linux 3.18 的 ext4和jbd2的驱动被移植到了3.10. 由于kernel模块中的内部依赖问题, 在Nexus设备上的移植操作放弃了一定量的功能. 放弃的功能包括:

 - ext3的驱动, 尽管ext4也可以挂载和使用ext3文件系统.
 - 全局文件系统支持**`Global File Sytem (GFS)`**.
 - ext4中的ACL支持.

对于ext4加密功能的增强支持, 设备提供商可以考虑实现加解密加速功能, 用以提高基于文件加密的速度和提高用户体验.

-----
**使用基于文件的加密**

FBE通过在fstab的userdata分区行的最后一列增加不附加参数fileencryption标志位来申明. 可以参见以下的例子:
[**`https://android.googlesource.com/device/lge/bullhead/+/nougat-release/fstab_fbe.bullhead`**](https://android.googlesource.com/device/lge/bullhead/+/nougat-release/fstab_fbe.bullhead)

同时可以通过以下方法来测试设备上的FBE实现. 可以指定一下的标志位:
**`forcefdeorfbe="<path/to/metadata/partition>"`**

该标志位将设置设备以FDE形式来启动, 但允许将其对开发者转换为FBE模式. 一般情况中, 该行为类似于**`forceencrypt`**, 将设备置于FDE模式. 同时, 在开发者预览的版本中它将提供一个调试选项来允许开发者将其转变为FBE模式. fastboot模式中也可以转变FBE模式:

    $ fastboot --wipe-and-use-fbe

该命令仅仅为开发的目的使用, 它将在真正的FBE设备发布前, 延时相关的功能. 该选项在未来可能被废弃.

-----
**同Keymaster整合 Integrating with Keymaster**

vold负责处理FBE密钥的产生和kernel相关钥匙对的管理. AOSP的实现中, 要求设备实现支持Keymaster HAL的版本为1.0或以上. 早期的Keymaster HAL版本已经不再支持.

在第一次启动是, 0号用户的密钥会产生, 并且在boot过程的前期被安装. 在init完成 on-post-fs 阶段之前, Keymaster必须已经准备好接受请求. 在Nexus设备中, 该项处理由以下的脚本块来完成:

 - 保证Keymaster在/data分区被挂载前启动.
 - 指定文件加密使用的算法组: AOSP对于FBE的实现使用了XTS模式的AES-256算法.

Note: 所有的加密均基于XTS模式的AES-256算法. 由于XTS的定义方式, 他需要两个256位的密钥; 所以造成CE/DE都是512位的密钥.

-----
**加密规则 Encryption policy**

Ext4加密可以将加密规则应用到文件夹级别. 当设备的用户数据分区初次创建时, 由init脚本来赋值基本的结构和规则. 该脚本同时会触发来创建第一个用户(0用户)的CE/DE密钥, 并且定义那些目录需要由该密钥进行加密. 

当其余的用户和工作模式被创建时, 相关所需的密钥会被创建, 并且存储到keystore中. 相关的证书和设备存储地址会被创建并且由加密规则将密钥和目录链接起来.

在当前的AOSP实现中, 加密规则被硬编码到以下的位置:
**`/system/extras/ext4_utils/ext4_crypt_init_extensions.cpp`**

可以通过在这个文件中增加例外的规则来避免指定的目录被加密, 目录可以加入到**`directories_to_exclude`**列表中. 如果进行了该类的修改, 设备提供商必须包含增加SElinux的规则, 以使得只有必须使用未加密目录的应用才能获取访问权限, 其他未授权的应用必须被排除在外.

目前已知的可以使用以上的例外情况的例子是支持早期版本的OTA情况.

-----
**在系统应用中支持Direct Boot**

**申明应用需要Direct Boot通知**

为了方便系统应用的快速移植, 应用级别为支持Direct Boot新增了两个新的属性. 其中**`defaultToDeviceProtectedStorage`**属性是仅对系统app支持的. 而**`directBootAware`**属性对所有app支持.

    <application
        android:directBootAware="true"
        android:defaultToDeviceProtectedStorage="true">

应用级别的directBootAware属性比较短, 它的作用是表明该应用中的所有组件都是已经和加密相关的.

**`defaultToDeviceProtectedStorage`**属性将缺省的应用存储空间由原来的指向CE区间重新连接指向DE区间. 而使用该属性标志的应用必须仔细调整所有存储于缺省空间的敏感数据. 将其中的敏感数据修改路径存储至CE空间. 使用标志的设备提供商应用也必须仔细考虑, 在存储相关数据时, 必须确保在DE空间内不包含用户信息相关的内容.

当应用在当前模式下运行时, 以下的系统APIs可用, 它们应当在需要CE存储空间支持来处理内容时被调用, 行为类似于它们在设备保护空间(TEE)中的副本一样.

 - Context.createCredentialProtectedStorageContext()
 - Context.isCredentialProtectedStorage()


**支持多用户Supporting multiple users**

每个在多用户环境中的用户将拥有单独的加密密钥. 每个用户有两份密钥: DE/CE密钥. 0用户必须登录进入设备, 因为它是特殊用户. 这部分是和设备管理权限使用相关的.

加密相关的应用以以下的规则同多用户环境进行互动: **`INTERACT_ACROSS_USERS / INTERACT_ACROSS_USERS_FULL`**, 其中后者将允许应用同设备上的所有应用进行交互. 但即使是这种情况下, 如果用户自己已经解锁, 这些拥有**USERS_FULL**规则的应用也仅仅被允许访问基于CE的目录(**`解锁操作导致设备加密(DE)的区间无法被保护`**).

某些应用也许可以自由的在所有的DE区域中进行交互, 但单个的用户解锁行为并不代表当前设备上的所有用户均已解锁. 应用自己应当在访问这些区域间自行进行检查.

每个工作模型下的用户ID同样拥有两份密钥: DE/CE. 当工作模型改变(work challenge)发生时, 当前的模型用户被解锁, TEE中的Keymaster将会提供当前模型的TEE密钥.

**处理更新 Handling updates**

恢复模式分区将无法访问用户数据分区中基于DE保护的内容. 所有实现FBE的设备均被强烈推荐支持即将到来的A/B系统更新模式的OTA. 因为该模式的OTA可以在正常操作下完成, 所以它不需要recovery自身访问存在于加密设备上的数据.

如果使用早期的OTA方案, 需要recovery进程访问存在于用户空间中存储的OTA文件, 那么使用以下方法:

 - 在用户空间分区重创建根目录的文件夹.(例如: “**`misc_ne`**”)
 - 将该根目录加入到加密规则的例外列表中(参见上文 加密规则).
 - 在该目录中创建保存OTA升级包的目录.
 - 为该目录和其中的内容创建SELinux规则和相关的文件规则, 用以控制对其进行的访问.仅允许获取OTA更新的应用或是进程来对该目录进行读写访问.
 - 其他所有的应用或进程对该目录无法进行访问.


-----
> **验证**

-----
为保证以上描述的功能能够正常工作, Android部署了部分的CTS加密测试.

当设备使用的kernel被编译时, 需要对源码进行额外的基于x86的kernel编译, 该kernel可以使用QEMU进行测试(android 模拟器). 这将允许该实现可以使用xfstest来进行测试. 测试方式为:

    $ kvm-xfstests -c encrypt -g auto

同时, 支持了FBE的设备提供商可以在设备上进行以下的手动测试:

 - 检查ro.crypto.state是否存在.
  - 保证ro.crypto.state是被加密的.
 - 检查ro.crypto.type是否存在.
  - 保证ro.crypto.type被设置进入文件中.

额外的测试包括, 测试者在主用户(0用户)已设置锁屏启动的设备上以userdebug进程启动, 然后使用adb shell连接进入系统, 并使用su成为root. 以root用户身份列出**`/data/data`**区间内容, 此时应该可以看到它包含的是加密过的文件名, 否则实现中存在有错误部分.

-----
> **AOSP 实现细节**

-----
该章节描述AOSP实现的细节, 并讲解基于文件的加密工作模式. 设备提供商基于该实现参考在设备上实现FBE和Direct Boot无需进行修改.

**ext4 加密**

AOSP实现使用kernel中的ext4加密功能, kernel被配置为:

 - 使用XTS 模式的 AES-256 加密文件内容.
 - 使用CBC-CTS模式的AES-256 加密文件名.

**密钥管理**

使用512位的AES-XTS密钥作为磁盘加密密钥, 它被保存在TEE中的另一份密钥(256位的AES-GCM密钥)进行加密, 然后存储. 如果需要使用TEE密钥, 必须满足以下三个条件:

 - 验证令牌. The auth token
 - 扩展证书. The stretched credential
 - 随机hash. The “secdiscardable hash”

**`auth token`**是由GateKeeper在用户成功登录时产生的加密令牌. TEE只有在正确的加密令牌被提供时才能够对AES-XTS密钥进行处理. 如果当前用户并没有证书, 那么验证的令牌将不被使用或者没有必要使用.

**`stretched credential`**是用户的证书, 同时经过了加密算法和加盐的处理. 并且在由lock setting服务传递给vold进行加密运算前, 还进行过hash处理. 该证书同所有通过**`KM_TAG_APPLICATION_ID`**进行的提权行为绑定, 并通过TEE中的密钥进行加密. 如果用户没有证书, 该扩展证书将不存在或者没有必要使用.
**`扩展证书同提权行为的绑定, 需要结合应用实例理解, 此处翻译可能不正确`**

**`secdiscardable hash`**是一个随机的16KB单独存放的文件, 用来重建密钥, 它类似于密码种子. 这份文件将在密钥删除时同样被安全删除掉, 或者被以其他方式再加密. 这个额外的保护将限制攻击者必须恢复这个删除文件的每一个比特来重建密钥. 这同样通过**`KM_TAG_APPLICATION_ID`**同TEE中的密钥进行绑定. 细节请参见[**`Keystore Implementer's Reference`**](http://source.android.com/security/keystore/implementer-ref.html).

-----