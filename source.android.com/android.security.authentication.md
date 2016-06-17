
###[**Authentication**](http://source.android.com/security/authentication/index.html)

> **Overview**

-----
**`Android 6.0`** 引入了基于用户认证权限的密钥管理机制. 为达成该目标, 两个关键组件需要合作运行. 第一个组件是密钥的存储管理和提供服务机制, 它完成密钥的存储并且在该基础上提供标准的加解密服务操作. 第二个是多种类型的用户认证功能, 用来证明用户身份和验证认证.

Android中的密钥存储管理由 **`keystore`** 服务和 **`KeyMaster`** 来提供(参考 [**`Android Keystore system`**](https://developer.android.com/training/articles/keystore.html)的信息. 在framework级别上, 由keystore服务来完成).  在android 6.0上, 提供两种用户验证方式, Gatekeeper(使用PIN/图形/密码验证) 和 Fingerprint (指纹验证). 这两个验证服务通过已认证的通道和 keystore 服务进行认证状态的交互.

 - 基于硬件的Keystore: 密钥服务管理, 它包含有一个基于硬件的密钥系统来进行key的存储管理. 同时也会包含有可信任执行环境(TEE).
 - Gatekeeper: 进行PIN/图形/密码验证的组件.
 - Fingerprint: 进行指纹验证的组件.

-----

> **Architecture**

-----
Gatekeeper和Fingerprint同Keystore和其他组件一起, 提供使用基于硬件的验证令牌功能([**`authentication tokens`**](http://source.android.com/security/authentication/index.html#authentication_token_format)). 

**注册 `Enrollment`**

-----
设备在进行过恢复工厂模式后的第一次启动过程中, 所有的认证机制均等待完成用户进行凭证注册工作.

在初始化的情况下, 用户必须首先输入 **`PIN/图形/密码`** 可选的一个初始密码给Gatekeeper. 该密码将被用以产生一个随机的64位的User SID(用户安全认证), 该SID将被用以对用户进行标识, 并和该用户的加密数据绑定起来. 

该User SID同用户密码是加密绑定的. 参见下面的细节, 通过GateKeeper正确认证将产生包含该User SID的AuthTokens.

当用户需要更改凭证管理(**`PIN/图形/密码`**)时, 必须首先提供已注册的凭证. 

如果该凭证成功验证, 则与该User SID关联的已存在凭证将转移关联至新的凭证. 这允许在用户修改凭证后继续访问相关的key. 

如果用户无法提供老的凭证, 那么新的随机User SID将被产生, 用户仍然可以访问设备, 但所有之前User SID相关的key均会丢失. 这称之为"未认证注册".

**注意以上的"未认证注册"在通常的android framework操作中是不允许的, 大部分用户并不会遇到这种情况. 但是, 来自设备管理者或者是攻击者进行的强制密码复位操作会造成这种情况的发生.**

-----

**验证 `Authentication`**

在注册完成后用户设置好密钥并且获得了User SID. 接下来可以进行验证的操作.

在下图示意中, 验证操作由用户开始提供一个**`PIN/图形/密码`** 或者是 **`指纹`** 开始. 所有的TEE组件均通过共享同一个密钥来验证其他组件的信息.

![Authentication flow](http://source.android.com/security/images/authentication-flow.png)

**验证流程**

以下操作的流程与设备中存在不同的验证方式相关, 同时包含**`Android OS`** 和 **`TEE OS`** 的参考:

 1. 用户提供PIN/图形/密码 或是 指纹. **`LockSettingService`**或者是**`FingerprintService`**将通过Binder向Android OS中的**`Gatekeeperd`**或者**`fingerprint`**守护进程发起请求. 

 2. 基于设备提供Gatekeeper或是指纹验证的功能. 接下来操作为:
   	a. **`Gatekeeperd`** 将第1步里接收到的密码hash值发送给TEE中的GateKeeper副本. 如果验证成功, 它将发送一份基于该hash的以HMAC密钥进行签名的包含对应User SID的凭证给Android OS中的Gatekeeper.
   	b. **`fingerprintd`**将第1步中接收到的指纹信息发送给TEE中的fingerprint副本. 如果验证成功, 则发送同样的签名凭证给Gatekeeper.
   	
 3. **`Gatekeeperd`**或**`fingerprintd`**将接收到的签名过的凭证发送给keystore服务. 额外的操作, Gatekeeperd还要在设备被重新锁定或者是用户修改密码的情况下通知keystore服务.
 
 4. **`keystore`**服务将接收到的签名凭证发送给Keymaster, Keymaster使用它共享给Gatekeeper或fingerprint的公钥进行认证. Keymaster将基于该凭证的时间戳和密钥分发机制来定义该认证的有效时间.

 **注意: 任何时候设备重启将导致认证凭据失效.**

**`[Tips: 该运行模型同样是android tee使用的典型例子.]`**

-----
> **Authentication token format**

-----

在hw_auth_token.h文件中定义的凭证格式是所有开发/组件必须包含的, 它将帮助各模块进行凭证共享. 

**`hardware/libhardware/include/hardware/hw_auth_token.h`**

下表描述了该凭证的一个简单序列化的协议. 注意所有的位都是固定大小的.

![tokenformat](https://ooo.0o0.ooo/2016/05/31/574d671c1e09e.png)

表项描述:

**`AuthToken Version`**: 定义包含的表项的版本号.

**`Challenge`**: 防止通过重试进行破解的随机整数. 通常是使用密钥操作请求的ID来定义. 当前情况下被fingerprint验证流程使用. 如果存在的话, 凭证仅在加密操作中包含有该相同的challenge值时才生效.

**`User SID`**: 独一的用户安全标示, 它以加密的形式同该设备认证上的所有key相关联. 查看[**`Gatekeeper`**](http://source.android.com/security/authentication/gatekeeper.html)的介绍来参考实现细节.

**`Authenticator ID (ASID):`** 和指定认证策略绑定来使用的标示, 每个认证器均可改变自己的ASID值来满足其特定需求.

**`Authenticator Type:`** 该项定义当前验证源是Gatekeeper还是Fingerprint.

    0x00	Gatekeeper
    0x01	Fingerprint

**`Timestamp:`** 基于最近一次的系统启动的时间(毫秒).

**`AuthToken HMAC key`**: 锁定的SHA-256位认证密钥.

-----

> **Device boot flow**

-----
在每次设备启动的过程中, HMAC验证凭证的密钥都必须生成, 以共享给所有的TEE组件使用(Gatekeeper/fingerprint/keymaster). 并且该密钥每次都必须在系统启动时随机生成, 以避免重复攻击.

在多个组件间共享HMAC密钥的协议是基于平台实现来完成的功能. 该密钥必须不能被放到TEE之外的位置. 因此, 如果TEE系统缺乏内部的进程通讯机制(IPC), 且该TEE必须通过外部OS来传递数据, 那么该项传输也必须基于安全密钥交换的协议来完成. 

可信任执行系统([**`Trusty`**](http://source.android.com/security/trusty/index.html)), 它是运行在Android设备上的TEE环境实现, 当然设备提供商可以通过其他的TEE来替代它. 

Trusty使用内部的IPC系统来完成**`keymaster/fingerprint/gatekeeper`**的直接通讯. 该系统中的HMAC密钥由**`keymaster`**单独保存. fingerprint/gatekeeper每次通过向keymaster申请的方式来使用, 并从不保存或者缓存该数据的副本.

**必须注意由于某些TEE的实现缺少IPC的基础支持, 那么该TEE上的应用不应产生通讯动作. 这将允许keystore服务直接对请求进行操作禁止, 并及时回复验证失败的信息. 因为它保存有该系统上的验证表信息, 这可以帮助节省TEE环境下可能的IPC资源消耗.**

-----
