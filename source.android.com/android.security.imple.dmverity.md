
###[**Implementing dm-verity**](http://source.android.com/security/verifiedboot/dm-verity.html)

-----
> **Operation**

-----
dm-verity保护存在于kernel中. 所以如果任何间谍程序在kernel启动之前进入到系统中,例如修改了kernel的代码, 它将对dm-verity免疫.

为了避免这种风险, 一般设备提供商使用烧入到设备中的密钥来验证kernel. 这个密钥在设备离开工厂后就不能被修改(**`HW OEM Key`**).

设备提供商将使用这个密钥来验证最顶级的bootloader的签名, 然后booloader将对后面的分区按照级别进行验证, 可能存在的第二级的bootloader, 直到kernel. 

一般设备提供商都会利用**`verified boot`**功能来验证kernel的正确性. 当kernel被正确验证后, kernel就能够在某个块设备(**`block device`**)被挂载时来进行验证了.

一种验证块设备的方式, 是将整个分区的内容进行hash, 然后将其同预先存储的值进行比较. 但是, 这种方式将会增加额外的耗时, 并消耗设备的电量.

而且这种情况下也会对启动增加时间, 并且不能预先提供一些服务.

在这种情况下, **`dm-verity`**在对每个block进行访问时单独验证. 当block内容被读进内存时, 每个block可以并行进行hash操作. 每个block的hash将继续以树的形式进行hash/验证. 

由于在以上操作中, 读取block是最耗时的操作, 所以由这种block级别hash/验证所带来的延时就会相对来讲更能容忍.

如果验证失败(**`block`**), 设备将产生I/O错误, 来指明某个block不能读取. 这在预期文件系统可能损坏的情况下是会产生的.

如果无法读取的数据对应用主功能影响不大, 应用在这种情况下可以选择继续进行处理. 但是如果数据影响应用. 那么应用可能无法启动.

-----

> **Implementation**

-----
**Summary**

 1. 生成ext4系统镜像(**system image**).
 2. 生成该镜像的hash树.
 3. 为该hash树生成dm-verity使用的表.
 4. 对该表进行加密来产生签名的表.
 5. 将该签名过的表和原始表数据组合加入到验证元数据(**`verity metadata`**)中.
 6. 将验证元数据, hash树同系统镜像组合起来.

查看[**`Chromium Projects - Verified Boot`**](http://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot)来了解hash树和dm-verity表的细节.

-----
**Generating the hash tree**

由上面的描述, hash树是dm-verity需要的资源. 

可以使用**`cryptsetup`**工具来产生hash树. 这是是一份通用的定义:

    <your block device name> <your block device name> <block size> <block size> <image size in blocks> <image size in blocks + 8> <root hash> <salt>

为了产生该hash树, 系统镜像在**`LEVEL 0`**被以分成4k大小的blocks. 每个block产生SHA256的hash值. 然后将**`L0`**的hash值组合成4K的blocks, 这样会生成更小的image, 然后继续以4k进行hash, 产生L1的hash值. 

该hash值按照L0/1/2...的组合, 最后会组合进入一个单独的block. 对该单独block进行hash. 就能得到hash树的根(**`root hash`**).

![Figure 1. dm-verity hash table](http://source.android.com/security/images/dm-verity-hash-table.png)

**`Figure 1. dm-verity hash table`**

hash树的大小(和对应的磁盘大小相关)不同于产生它的验证分区的大小. 一般情况下, hash树的大小都比较小, 会小于30MB(system分区一般定义为512MB).

如果在某一级的block中, 由上一级产生的hash值填充之后不能完整的产生block(4k)对齐, 需要使用0值对其填充以产生block. 

这可以让所有的hash值均被加入, 能够代表整个的分区信息摘要.

在产生hash树时, 将后一级的hash数据加入到前一级的hash数据上, 然后将所有的数据写入到磁盘中. 这种整合的操作通常不会包含最底层的hash(**`LEVEL 0`**).

概括来讲, 创建hash的算法为以下操作:

 1. 选择一个随即盐值(16进值).

 2. 将系统镜像分为4k的blocks.

 3. 对每个block,进行盐值处理的SHA256 hash.

 4. 将hash集合以产生下一级的block.

 5. 0值填充第4步的结果来符合4k block边界.

 6. 将当前填充完成的hash block加入到hash 树中.

 7. 对上一级的hash结果重复执行2~6步操作. 直到产生唯一的hash值.

该唯一的hash值就是root hash. 它和随即盐值被一起用来生成这次的dm-verity hash表.

**Building the dm-verity mapping table**

-----
使用dm-verity hash表生成dm-verity映射表, 它将向kernel提供当前block设备的证明. 该映射表被用来产生fstab(文件分配表)和启动使用. 同时该表中包含有blocks的大小和hash_start, 也可以推断出hash size blocks的偏移(layer 0的长度).

查看[**`cryptsetup`**](https://gitlab.com/cryptsetup/cryptsetup)来理解dm-verity映射表的详细组成.

-----
**Signing the dm-verity table**

对dm-verity表进行加密, 以生成签名的表. 当对分区进行验证时, 首先需要验证该签名表. 

该验证通过固定在boot image某个位置上的密钥来完成. 这个密钥通常包含在设备提供商的编译系统中, 这样可以配合bootloader/kernel来自动产生设备的镜像.

通过签名和内置的密钥来验证分区:

 1. 以**`libmincrypt`**适配的格式, 在/boot 分区的 /verity_key文件中增加一个RSA-2048密钥. 从该固定位置获取密钥来验证hash树.
 
 2. 在相关的fstab处理文件中, 对相关挂载分区的命令增加"**`verify`**"标识.

-----
**Bundling the table signature into metadata**

在将签名表和dm-verity表集成至
Bundle the table signature and dm-verity table into verity metadata. The entire block of metadata is versioned so it may be extended, such as to add a second kind of signature or change some ordering.

As a sanity check, a magic number is associated with each set of table metadata that helps identify the table. Since the length is included in the ext4 system image header, this provides a way to search for the metadata without knowing the contents of the data itself.

This makes sure you haven't elected to verify an unverified partition. If so, the absence of this magic number will halt the verification process. This number resembles:
0xb001b001

The byte values in hex are:

first byte = b0
second byte = 01
third byte = b0
fourth byte = 01
The following diagram depicts the breakdown of the verity metadata: