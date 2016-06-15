
###[**Authentication**](http://source.android.com/security/authentication/index.html)

> **Overview**

Android 6.0引入了基于用户认证权限的密钥管理机制. 为达成这个目标, 需要两个关键组件的合作运行. 第一个组件是密钥的存储管理和提供服务, 它完成密钥的存储并且在这个基础上提供标准的加解密操作. 第二个是多种类型的用户认证功能, 用来证明用户身份和成功认证.

Android中的密钥存储管理由 keystore 服务和 KeyMaster来提供.(参考 [**`Android Keystore system`**](https://developer.android.com/training/articles/keystore.html)的信息, 在framework级别上, 由keystore服务来完成).  在android 6.0上, 提供两种验证方式, Gatekeeper(使用PIN/图形/密码验证) 和 Fingerprint (指纹验证). 这两个验证服务通过已认证的通道和 keystore 服务进行认证状态的交互.

 - 基于硬件的Keystore: 密钥服务管理, 它包含有一个基于硬件的密钥来进行key的存储管理. 同时也会包含有可信任执行环境(TEE).
 - Gatekeeper: 进行PIN/图形/密码验证的组件.
 - Fingerprint: 进行指纹验证的组件.


> **Architecture**

Gatekeeper和Fingerprint同Keystore和其他组件一起, 提供使用基于硬件的验证令牌功能([**`authentication tokens`**](http://source.android.com/security/authentication/index.html#authentication_token_format)). 

**Enrollment**

在完成恢复工厂模式后的第一次启动过程中, 所有的认证机制均等待完成用户进行的凭证注册工作.

初始化的情况下, 用户必须首先输入 PIN/图形/密码 可选的一个初始密码给Gatekeeper. 该注册密码将被用以产生一个随机的64位的User SID(用户安全认证), 该SID将被用以对用户进行标识, 并和该用户的加密数据绑定起来. 该User SID同用户密码是加密绑定的. 参见下面的细节, 正确的认证GateKeeper将产生包含该User SID的AuthTokens.

当用户需要更改凭证管理(PIN/图形/密码)时, 必须首先提供已存在的凭证. 如果该凭证成功验证, 与该User SID关联的已存在凭证将转移关联至新的凭证. 这允许在用户修改凭证后继续访问相关的key. 如果用户无法提供老的凭证, 那么新的随机User SID将被产生, 用户仍然可以访问设备, 但所有之前User SID相关的key均会丢失. 这称之为"未认证的重新注册".

注意以上的"未认证的重新注册"在通常的android framework操作中是不允许的, 所有大部分用户并不会遇到这种情况. 但是, 由设备管理者或者攻击者来进行的强制密码复位操作会造成这种情况的发生.


**Authentication**

在注册完成后用户设置好了密钥并且获得了User SID. 接下来可以进行验证的操作.

在下图示意中, 验证操作由用户开始提供一个PIN/图形/密码 或者是 指纹 开始. 所有的TEE组件均通过共享同一个密钥来验证其他组件的信息.

![Authentication flow](http://source.android.com/security/images/authentication-flow.png)
图一: 验证流程

以下操作的流程与设备中存在不同的验证方式相关, 并且同时包含Android OS 和 TEE OS的参考:

 1. 用户首先提供PIN/图形/密码 或是 指纹. LockSettingService或者是FingerprintService将通过Binder向Android OS中的Gatekeeperd或者fingerprint守护进程发起请求. 

 2. 基于当前设备提供基于Gatekeeper或是指纹验证的功能. 接下来的操作有:
   	a. Gatekeeperd 将第1步里接受到的密码hash值发送给TEE中的GateKeeper副本. 如果验证成功, 它将发送一份基于hash的HMAC密钥进行签名的包含对应User SID的凭证给Android OS中的Gatekeeper.
   	b. fingerprintd将第1步中接收到的指纹信息发送给TEE中的fingerprint副本. 如果验证成功, 则发送同样的签名凭证给Gatekeeper.
   	
 3. Gatekeeperd或fingerprintd将接受到的签名过的凭证发送给keystore服务. 额外的情况下, Gatekeeperd还要在设备被重新锁定或者是用户修改密码的情况下通知keystore服务.
 
 4. keystore服务将接收到的签名凭证发送给Keymaster, 通过它共享给Gatekeeper或fingerprint的公钥进行认证. Keymaster将基于该凭证的时间戳和密钥分发机制来定义该认证的有效时间.
 **注意: 任何时候设备重启将导致认证凭据失效.**


> **Authentication token format**

在hw_auth_token.h文件中定义的凭证格式是所有开发/组件必须包含的, 它将帮助各模块进行凭证共享. 
**`hardware/libhardware/include/hardware/hw_auth_token.h`**

下表描述了该凭证的一个简单序列化的协议. 注意所有的位都是固定大小的.

![tokenformat](https://ooo.0o0.ooo/2016/05/31/574d671c1e09e.png)

表项描述:

AuthToken Version: 所有包含的表项的版本说明.

Challenge: 随机整数来防止重试破解. 通常是使用密钥操作请求的ID来定义. 当前情况下作为fingerprint验证流程使用. 如果存在的话, 凭证仅在加密操作包含该相同的challenge值时生效.

User SID: 不重复的用户标示符, 它以加密的形式同该设备认证上的所有key相关联. 查看[**`Gatekeeper`**](http://source.android.com/security/authentication/gatekeeper.html)的介绍来参考细节.

Authenticator ID (ASID): 为同指定的认证策略绑定而使用的标示符, 每个认证器均可改变自己的ASID值来满足其特定需求.

Authenticator Type: 该项定义了当前验证的源是Gatekeeper还是Fingerprint.

    0x00	Gatekeeper
    0x01	Fingerprint

Timestamp: 基于最近的系统启动的时间(毫秒).

AuthToken HMAC key: 除了HMAC位之外的所有锁定SHA-256位.

> **Device boot flow**

在每次设备启动的过程中, HMAC验证凭证的密钥都必须生成, 以共享给所有的TEE组件使用(Gatekeeper/fingerprint/keymaster). 并且该密钥每次都必须在系统启动时随机生成, 以对重复攻击进行保护.

在多个组件间共享HMAC密钥的协议是基于平台实现来完成的功能. 该密钥必须不能被放到TEE之外的地方. 因此, 如果一个TEE系统缺乏内部的进程通讯机制(IPC), 而且TEE必须通过外部OS来传递数据, 那么该项传输必须基于安全密钥交换的协议来完成. 

可信任执行系统([**`Trusty`**](http://source.android.com/security/trusty/index.html)), 运行在Android设备上, 是一份TEE环境的例子实现, 当然设备提供商可以通过其他的TEE来替代它. Trusty使用内部的IPC系统来完成keymaster/fingerprint/gatekeeper的直接通讯. 该系统中的HMAC密钥由keymaster单独保存. fingerprint/gatekeeper通过向keymaster每次申请的方式来使用, 并且从不保存或者缓存该数据的副本.

必须注意的事由于某些TEE的实现缺少IPC的基础支持, 那么该TEE上的应用不会产生通讯动作. 这将允许keystore服务直接对请求进行禁止操作, 并及时回复验证失败的信息. 因为它保存有该系统上的验证表信息, 这可以帮助节省TEE环境下可能的IPC资源消耗.


