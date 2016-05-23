###[**OTA Updates**](http://source.android.com/devices/tech/ota/index.html)

当前的**Android**设备能够接收并安装 **OTA** 来进行系统和应用程序的更新. 设备上自带有特殊的恢复分区(**recovery partition**), 分区中包含有必须的软件, 来支持对下载的更新包进行解析, 并更新到系统的其他部分上.

该章节描述OTA包的结构和如何使用提供的工具来创建它. 创建新的android设备和希望在已发布的android设备上进行更新的开发者需要实现OTA包的功能. OTA更新被设计为更新底层的操作系统和系统分区(**system partition**)上的只读app. 这些更新不会影响从**GooglePlay**上安装的应用.

当前章节针对**Andriod 5.x release**的**OTA**系统. 早期release系统的更新. 参见早期release迁移章节.

> **Android device layout**

标准的android设备通常在flash空间上有以下的分区:
![标准android设备分区结构](https://docs.google.com/drawings/d/19NQ4wnl_FrfLwbyXL7ZV8hy_5GULjcwGYntJf6IyDVI/pub?w=1440&h=1080)

**bootloader:**
包含设备提供商自定义的bootlader执行代码. 通常存在两个同样的bootloader分区. 以支持recovery模式中升级bootloader. 同时也包含进入recovery模式判断的逻辑.

**misc:**
单独的分区用以在bootloader和kernel间传递相关信息. 如recovery命令行, android设备boot次数等.

**boot:**
包含linux kernel和一个较小的root文件系统(该系统将被load进ram disk设备). 它将挂载系统和其他分区, 并启动系统分区上的其他执行信息. 

**system:**
包含源代码在**Android Open Source**项目(**AOSP**)上的系统程序和库文件. 在通常的操作中. system分区被挂载为只读格式. 只有在OTA更新时被更改.

**vendor:**
包含设备提供商的应用和库文件, 该部分的程序代码并不开源. 同system分区一样.

**userdata:**
保存由用户安装的应用所管理的数据等等, OTA过程中这部分的数据内容通常不会被改变.

**cache:**
临时性的保存非常少的应用所存储的数据(使用该分区需要特殊的系统权限). 而且用以保存下载OTA更新包. 其他的应用在预期存储的文件可能随时消失的情况. 某些OTA包在安装过程中可能会格式化该分区.

**recovery:**
包含第二个完整的linux系统. 内容有kernel和部分特殊的恢复模式执行程序, 它可以读取更新包并根据包内容来更新其他的分区.

**tz:**
通常包含设备提供商自定义的trust zone相关的密钥/加解密信息, tee小系统的代码也通常放置在此处.

**factory_setting:**
通常包含设备提供商硬件相关的信息, 如device_id/mac地址/pq数据库等.

> **Life of an OTA update**

一次经典的**OTA**更新包含以下步骤:

 1. 设备向OTA服务器进行常规检查,并且在更新可用时接到通知. 通知包含更新包的URL地址和显示给用户的描述字符串内容.
 2. 更新包被下载至cache或data分区. 并且它的加密签名同**`/system/etc/security/otacerts.zip`**中的证书进行校验, 完成后, 用户被提示可以进行更新.
 3. 设备重启至恢复模式, 该情况下recovery分区的kernel和系统将代替正常的boot分区中的内容被启动.
 4. init程序启动recovery可执行程序. 它将搜寻在**`/cache/recovery/command`**中的命令行参数, 该参数指向下载下来的更新包.
 5. recovery程序使用存在于**`/res/keys`**(该文件存在于recovery分区load的ram disk内)中的公钥来再次验证更新包.
 6. 更新包中的数据被读出, 按照需求来更新boot, system, vendor分区. 同时在system分区中会留下新的文件, 来包含新的recovery分区的内容.
 7. 设备重启至正常模式:
 	a.更新过的boot分区被挂载,并开始执行更新过的程序和系统分区.
 	b.作为正常启动的一部分,系统检查前面留下的新文件(由recovery执行程序存放在系统分区中), 将它同recovery分区进行比较. 如果不同的话, recovery分区将由该文件进行更新. 在以后的启动过程中, 该文件同recovery分区一致, 则不会进行更新. 
到此时整个系统的更新完成.

> **Migrating from Previous Releases**

略