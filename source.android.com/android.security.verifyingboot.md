
###[**Verifying Boot**](http://source.android.com/security/verifiedboot/verified-boot.html)

-----
> **Objective**

-----
Verified boot 保证设备软件能够从基于从硬件底层开始到系统分区级别的正确环境下进行启动. 在boot过程中, 每一级的模块在执行下一级的模块之前先进行验证动作以保证正确性.

这个属性还可以用来提示用户,当他们使用一个'旧'的设备时, 软件部分曾经有过未预期的修改. 同时它可以提供基于远程验证的信号反馈, 连同加密/**`TEE`**环境一起的可信任绑定机制, 对用户数据进行额外的保护以避免恶意软件的修改.

必须注意如果在某阶段验证动作(**`verification`**)失败, 用户必须能够得到明显的可视化通知, 并且能够按照自己的意愿进行选择继续使用当前的设备.

-----
> **Glossary**

-----
verified boot相关的术语词汇表

    术语: 定义
    
    Boot state:
    设备的启动状态表明当设备启动时提供给最终用户的保护级别. 状态有: GREEN / YELLOW / ORANGE / RED.
    
    Device state:
    设备状态表明当前设备支持如何被刷入软件. 设备状态分为 LOCKED / UNLOCKED.
    
    dm-verity:
    运行时通过签名数据和hash树来校验分区信息的kernel功能.
    
    Keystore:
    公钥的签名组合称为keystore.
    
    OEM key:
    OEM密钥是一个固定无法修改的密钥, 它将提供给bootloader, 并被用来验证boot image(kernel+ramdisk).

-----
> **Overview**

-----
为了对设备状态(**`device state`**)进行补充 - 在已经存在于设备中并且控制bootloader是否允许其他软件刷入, android加入了启动状态(**`boot state`**)的实现, 并由此来指明当前系统的可信任级别.

-----
**Classes**

verified boot按照设备提供商实现推荐标准的完整性, 分为两类.

Class A在到分区验证的整个启动链中均实现了verified boot. 该实现必须支持**`LOCKED`**设备状态, 和**`GREEN/RED`** 启动状态.

Class B除了实现Class A的内容之外, 还提供 **`UNLOCKED`** 设备状态和 **`ORANGE`** 启动状态.

-----
**Verification keys**

Bootloader必须使用基于硬件信任机制来进行验证. 为了bootloader能够验证**`boot/recovery`**分区, bootloader必须能够访问固定的OEM密钥. 它将首先尝试使用OEM密钥对boot分区进行验证, 如果失败再尝试使用其他的验证密钥.

在Class B的实现中, 用户必须能够在设备被解锁(**`UNLOCKED`**)的情况下, 刷入使用其他密钥进行签名的软件. 如果设备处于锁定(**`LOCKED`**)状态, 并且使用OEM密钥验证失败, bootloader将会尝试使用该分区签名中自带的证书来进行验证. 而且, 在任何时候只要使用不是OEM密钥签名的分区, bootloader均应返回明显的提示, 或者是警告. 其内容由下面描述.

-----
**Boot state**

使用verified boot机制设备将在每次的启动尝试后最终启动至以下四种状态中的一种:

 - **`GREEN`**. 说明当前已经从bootloader到验证分区的全链表验证均正确完成, 包含bootloader, boot分区和其他所有的验证分区(system/vendor/data/xx).

 - **`YELLOW`**. 说明boot分区已经通过了它自带的证书校验, 该签名是有效的. 启动过程中, bootloader将会显示提示, 并且打印出公钥的指纹.

**`[Tips: 在某些情况下, 设备提供商可以通过review第三方发行image的情况. 提供密钥给第三方发行相关image. 该密钥能够被设备提供商的bootloader所验证.]`**

 - **`ORANGE`**. 说明当前设备已经被自由修改过. 设备验证工作留给用户自行确认. bootloader必须显示警告信息. 然后进行boot动作.

 - **`RED`**. 说明设备的认证动作失败, bootloader必须显示警告信息, 然后由用户确认是否继续boot操作.

recovery分区的验证必须遵循同样的机制.

-----
**Device state**

设备在所有的时间都必须处于以下两种状态之一:

 - **`LOCKED`**. 说明当前设备不能被刷写. LOCKED设备只能启动至GREEN/YELLOW/RED状态中的一种.

 - **`UNLOCKED`**. 说明当前设备可以被自由刷写, 并且不再进行验证工作. UNLOCKED设备只能boot至ORANGE状态

![Figure 1. Verified boot flow](http://source.android.com/security/images/verified_boot.png)

**`Verified boot flow`**

-----
> **Detailed design**

-----
实现全链的信任验证机制需要bootloader和boot分区上的软件的同时支持, boot分区上的软件将要负责对剩余的分区进行挂载动作.

验证使用的元数据(**`verification metadata`**)必须要被附加在系统分区和所有其他需要被验证的分区上.

-----
**Bootloader requirements**

**`bootloader`**是设备状态的监护者, 同时它负责要初始化TEE环境, 并且对其进行根(**`root`**)信任绑定.

同时也很重要的是, bootloader需要验证boot/recovery分区的正确性, 并且在需要执行其分区的kernel之前, 对相关的验证结果显示警告/提示信息, 用以提示用户当前的启动状态(**`Boot State`**).

**`[Tips: 即使设备提供商不实现UNLOCKED功能. 对于bootloader的需求仍然存在, 设备本身/运行在设备上的敏感应用均需要确认处于安全环境.]`**

-----
**Changing device state**

设备状态的转换是通过使用fastboot flashing 命令**`[unlock|lock]`**来完成的. 为了保护用户数据, 所有的状态改变过程均需要数据清除操作. 注意在数据被清除之前一定需要得到用户的操作确认.

在用户购买一个使用过的开发设备时, **`UNLOCKED/LOCKED`**的转换状态必须是被确认的.

当设备处于锁定(**`locking`**)状态, 用户可以认定当前的设备是处于OEM保护状态之下.

LOCKED到UNLOCKED状态的转换可以是因为开发者需要解除设备验证动作.

为完成设备状态转换的fastboot命令的需求按照下表来体现:

    fastboot command:    Requirements

    flashing lock:
    在用户确认后进行清除data操作. 设置写保护位来指明设备已经locked.
    
    flashing unlock:
    在用户确认后进行清除data操作. 清除写保护位来指明设备已经locked.

当修改分区内容时, bootloader必须检查以上操作所设置的比特位来进行以下操作.

    fastboot command:    Requirements

    flash <partition>:
    如果flashing unlock已经设置, 则可以flash该分区, 否则不能flash该分区.

任何可被执行的fastboot 命令用来修改分区的内容时, 都必须进行同样的检查.

**注意: Class B的实现必须支持改变设备状态.**

-----
**Binding TEE root of trust**

当设备支持TEE环境时, 在完成分区验证和 TEE初始化后, bootloader需要将以下信息传递给TEE, 用来对Keymaster进行根(**`root`**)信任绑定.

 - 用来对boot分区进行签名的公钥.

 - 当前设备的状态(**`LOCKED/UNLOCKED`**).

以上操作将改变TEE派生出来的密钥. 以分区加密来作为例子, 这可以在device状态改变时保护用户数据被解密.

**注意: 改变TEE派生的密钥意味着当系统软件/设备状态改变时, 加密的用户数据将无法被访问, 因为TEE将尝试使用收到的公钥来对加密数据进行解密.**

**`[Tips: 使用相关验证同样是保证TEE image正确性的必须要求.]`**

-----
**Booting into recovery**

对recovery分区的检查必须严格的同对boot分区进行的检查和操作一致.

-----
**Communicating boot state**

系统软件需要能够确认前一阶段的验证结果. bootloader需要将当前boot状态的结果作为参数从kernel command line中指定(或者通过设备节点树的方式**`firmware/android/verifiedbootstate`**)将其传递给系统.描述见下表:

**`Kernel command line parameters`**

    androidboot.verifiedbootstate=green:
    设备启动至GREEN状态, boot分区使用了OEM密钥验证并且可用.
    
    androidboot.verifiedbootstate=yellow:
    设备启动至YELLOW状态, boot分区使用内置的证书验证并且可用.
    
    androidboot.verifiedbootstate=orange:
    设备启动至ORANGE状态, 设备被unlocked并且不进行任何验证.
    
    androidboot.verifiedbootstate=red:
    设备启动至RED状态, 验证失败.

-----
**Boot partition**

当执行控制权转移到boot分区时, 该分区内的软件负责为后续的分区设置验证环境. 

因为一般情况下后续分区(**`/system /data `**)都比较大, 系统分区通常不会使用前面的方式来进行验证. 但它必须通过其他访问方式来进行验证, 例如**`dm-verity`**的kernel支持或者是其他类似方案. 

如果使用**`dm-verity`**来验证大的分区, 附加到每个分区上的验证元数据签名必须首先被验证, 然后才能进行dm-verity设置和挂载操作.

-----
**Managing dm-verity**

缺省情况下, dm-verity工作在enforcing模式, 它在设置时会得到一份hash树的表数据, 然后它将读取分区的每一个block同hash表进行验证. 

如果在某一个block上验证失败, 它将返回I/O错误, 并且将该block设置为未知的块区, 使得该块区不能被用户空间所访问. 

如果某些block被设置为损坏, 这将造成部分依赖于该块区的程序无法使用.

如果dm-verity一直以正确的签名元数据进行enforcing验证, 那么无需进行而外的操作. 

但是, 通过使用可选的verity table参数, dm-verity可以被配置为logging模式来工作, 它将检查并记录验证失败的block, 但是允许该block的I/O访问. 

如果在任何情况下dm-verity没有执行,或者是验证元数据无法正确完成验证, 在设备允许被启动的情况下相关的信息需要通知给用户, 类似于上面的描述设备启动进入**`RED`**状态.

![Figure 2. dm-verity management](http://source.android.com/security/images/dm-verity_mgmt.png)

**`dm-verity management`**

-----
**Recovering from dm-verity errors**

由于系统分区通常情况下都要比boot分区大, 所以验证失败的可能性也要更高. 

尤其是存在更大的可能性发生某些无意的磁盘损坏, 这将造成验证失败, 并且可能会造成其他模块中良好的设备应用因为关键block无法访问而无法使用的情况. 

如果在enforcing模式下dm-verity验证失败并且当前logging模式已经实现, 那么设备会重启至logging模式来进行dm-verity, 它可能不断后续重启来记录验证失败的块区. 直到已验证的分区被重新刷写或者是OTA更新. 

这种情况下需要有一个持续使用的标识来记录dm-verity的结果. 当已验证的分区被修改后, 该标识将被清除, dm-verity将继续进入enforcing模式. 

任何时候dm-verity没有进入到enforcing模式, 在验证的分区被挂载前都要对用户进行通知. 用户未得到通知前不能有任何未被验证的数据进入到用户空间.

-----
**Verified partition**

在一个已支持验证的设备中, 系统分区必须一直是被验证的. 同时任何其他的只读分区也最好被设置为进行验证. 

如果存在任何的包含有可执行代码的只读分区, 例如vendor/OEM分区的情况下, 支持验证的设备必须包含对该分区的验证.

在分区进行验证的过程中, 分区上必须附加有签名的验证元数据. 相关元数据包括有一份基于分区信息的hash树, 和一份包含有签名参数的验证表, 同时也加入了hash树的根信息. 

如果相关信息在dm-verity进行设置时出现信息丢失或者无效, 那么用户必须得到相关的警告信息.

-----
> **Implementation details**

-----
**Key types and sizes**

OEM密钥推荐使用RSA算法密钥, 并且使用2048位或者更高的位数. 

并且使用65537(F4)来作为盐值指数. 实际使用的OEM密钥必须具有相当或者更高级别的强度.

**Signature format**

-----
android verifitable boot image的签名使用一个ASN.1 DER-加密信息, 该信息可以被类似于以下参考[**`platform/bootable/recovery/asn1_decoder.cpp`**](https://android.googlesource.com/platform/bootable/recovery/+/master/asn1_decoder.cpp)的解码器来进行解密. 
信息本身的结构如下:

    AndroidVerifiedBootSignature DEFINITIONS ::=
         BEGIN
              FormatVersion ::= INTEGER
              Certificate ::= Certificate OPTIONAL
              AlgorithmIdentifier  ::=  SEQUENCE {
                   algorithm OBJECT IDENTIFIER,
                   parameters ANY DEFINED BY algorithm OPTIONAL
              }
              AuthenticatedAttributes ::= SEQUENCE {
                     target CHARACTER STRING,
                     length INTEGER
              }
    
              Signature ::= OCTET STRING
         END

该格式中的**`Certificate`**位是一份完整的X.509的签名, 它包含有用来签名的公钥, 相关定义参见[**`RFC5280`**](http://tools.ietf.org/html/rfc5280#section-4.1.1.2) 的4.1小节. 

当设备处于锁定状态(**`LOCKED`**)时, bootloader必须一直首先使用OEM密钥来进行验证. 并且在使用自带的签名验证情况下启动至**`YELLOW/RED`**状态.

剩余的数据结构定义类似于[**`RFC5280`**](http://tools.ietf.org/html/rfc5280#section-4.1.1.2)4.1.1.2/4.1.1.3小节所定义的内容. 

除了**`AuthenticatedAttributes`**位是例外. 该位包含有该image用来被验证的整数型的长度, 并且包含有该分区存在于image中的那个位置(boot, recovery, xx).

-----
**Signing and verifying an image**

产生签名的image:

 1. 生成未签名的image.

 2. 使用0填充来将image填充至下一个页表大小边界(如果刚好在边界大小则无需该步骤).

 3. 按照已填充的image大小和所在的目标分区信息填充上面的**`AuthenticatedAttributes`**字段.

 4. 将该**`AuthenticatedAttributes`**字段附加到以上的image中.

 5. 对image进行签名.

对image进行验证:

 1. 确定需要读取的包括padding位image的大小(例如通过相关的头信息来获得).

 2. 按照制作签名image的定义的偏移位置来读取签名.

 3. 验证**`AuthenticatedAttributes`**位的内容. 如果相关数值并不合法, 将其定义为签名验证失败.

 4. 按照签名方式校验image和**`AuthenticatedAttributes`**的其它内容.

-----
**User experience**

设备进入到GREEN启动状态的用户, 只会接收到正常设备启动过程中的交互请求, 而不会需要增加其它额外的交互. 

在其它的启动状态下, 用户必须看到最少持续5秒钟的警告信息. 用户最好在该时段同设备进行交互, 警告信息必须维持可见至少30秒, 或者直到用户主动清除该信息.

以下例子展示了其它启动状态下用户交互的界面表示:

-----
设备状态: **`YELLOW (用户交互之前和之后)`**

![yellow state sample 1](http://source.android.com/security/images/boot_yellow1.png)

![yellow state sample 2](http://source.android.com/security/images/boot_yellow2.png)

-----
设备状态: **`ORANGE`**

![orange state sample](http://source.android.com/security/images/boot_orange.png)

-----
设备状态: **`RED`**

![red state sample](http://source.android.com/security/images/boot_red.png)