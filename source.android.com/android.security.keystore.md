
###[**Hardware-backed Keystore**](http://source.android.com/security/keystore/index.html)

在芯片上运行的TEE系统的可用性, 决定了android设备是否可以向运行在其上的android系统/平台服务/甚至是第三方应用来提供基于硬件的,强安全性的服务.

在android 6.0上, Keystore有很多的[**加强和扩展**](http://source.android.com/security/keystore/features.html). 其中包括增加的对称加密原语, AES和HMAC, 基于硬件密钥的访问控制系统. 该访问控制系统可以在密钥产生的时候加以指定,并在使用时强制施行密钥的使用周期. 每个密钥均能被限制在用户认证之后使用, 并能限制为指定的目的和使用指定的加密参数. 更多的信息可以查看[**实现参考**](http://source.android.com/security/keystore/implementer-ref.html).

在android 6.0之前, android已经提供了简单的基于硬件的加解密服务API, 它由Keymaster硬件抽象层(HAL)的0.2/0.3版本来实现. Keystore提供数字签名和认证操作, 并加入了产生/导入非对称签名密钥对的功能. 在很多设备上该功能已经实现, 但是由于安全目标的多样性, 这些功能无法由一套签名的API来兼容. 所以在android 6.0上扩展了Keystore API来提供广泛属性的支持.

> **Goals**

android 6.0的Keystore API和其底层的KeyMaster 1.0 HAL的实现目标， 是提供一套基本但完备的加解密原语操作, 用以允许实现基于访问控制和硬件密钥的协议.

android 6.0的Keystore同时加入了以下加解密的支持:

 - 使用控制方案的定义来实现密钥的受限使用, 这样可以减少因为密钥的错误使用而导致的安全风险.
 - 访问控制方案的英译来允许对密钥进行指定用户/客户端/可定义时间的限制.


> **Architecture**

KeyMaster HAL是一个由OEM提供实现, 动态加载的库文件, 它由Keystore服务使用, 向上层提供基于硬件的加解密服务. HAL的实现中, 必须将所有的敏感操作都排除在用户空间/内核空间之外. 敏感操作的行为必须被限制在通过部分kernel接口访问的安全处理器中完成. 建议的实现架构如下:

![Access to Keymaster](http://source.android.com/security/images/access-to-keymaster.png)
图一: `访问KeyMaster`

在android设备中, 上图描述的使用Keymaster HAL的**`Client`**可以从多个位置发起(例如app,framework,或者KeyStore进程), 但这部分同该文档的描述无关. 当前描述的Keymaster HAL Api是最底层的实现部分, 它会由平台内部使用而不会暴露给开发者. 高级别的API将会在[**`Android Developer site`**](http://developer.android.com/reference/java/security/KeyStore.html)介绍.

KeyMaster HAL的目的并不是实现任何安全相关的算法, 而是向安全环境中的副本发起挂载/解载密钥的操作. 该描述中的线性格式化操作是基于实现定义的.

> **Compatibility with previous versions**

KeyMaster 1.0 HAL是同早期发布版本完全兼容的, 会促进兼容升级的方式, 预升级Marshmallow的设备上如果使用的是早期的KeyMaster HAL版本, Keystore提供一套实现1.0 HAL版本的代理, 通过该代理可以直接访问已实现的硬件库. 该代理不能完全实现1.0 HAL的功能, 它仅仅支持RSA/ECDSA算法, 并且密钥强制验证的工作将由代理在非安全环境中实现.

