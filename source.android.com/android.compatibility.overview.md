### [**Android Compatibility**](http://source.android.com/compatibility/index.html)

Android开发的目标,是建立一个开放性的平台提供给开发者进行各种应用的创作.

 - Android兼容性程序定义了Android平台的技术细节,并且向OEM(平台供应商)提供工具以保证开发的应用能够在多种类型的设备上工作.
 - Android SDK提供了内建的工具集给开发者,以明确该应用所需的设备功能.
 - Google Play仅向能够支持它的应用设备开放.


> **Why build compatible Android devices?**

![Compatibility ecosystem](http://source.android.com/compatibility/images/compat-ecosystem.png)

**Figure 1.** `Android系统会推进设备兼容性`

**用户需要自定义设备.**

手持电话是一个高度个人化,持续开机,持续在线的设备. 用户会持续的需要自定义该设备来满足需求. Android因此被设计为一个健壮的满足售后应用要求的系统.

**由开发者来满足用户**

没有单独的设备提供商能够编写用户可能需要的所有应用. 由第三方开发者来为用户编写应用. Android开源项目(**AOSP**)的目标是使应用开发变得尽可能的开源和简单.

**每个用户都需要通用的生态系统**

开发者为了bug而写的每一行代码都没有增加新功能. 有更多兼容的设备, 就有更多可以在该设备上运行的应用. 通过创建高度兼容的Android设备, 可以对非常多的应用提供支持, 同时鼓励开发者创建更多的应用.

> **Android compatibility is free, and it's easy**

要创建Android兼容的设备, 参考以下三步的流程:

 1. 获取Android软件开源代码([**android software source code**](http://source.android.com/source/index.html)), 将其porting到硬件上.
 2. 完成Android兼容性定义文档([**Android Compatibility Definition Document (CDD)**](http://source.android.com/compatibility/android-cdd.pdf))的需求. CDD列出了所有兼容的Android设备的软件/硬件要求.
 3. 通过兼容性测试套件([**Compatibility Test Suite (CTS)**](http://source.android.com/compatibility/cts-intro.html))的测试. 使用CTS作为开发的持续目标来完成达到兼容性要求.

完成CDD需求并通过CTS测试后, 厂商的设备即成为Android兼容设备, 这意味着在该生态系统中的app将在厂商的设备上提供一致的用户体验. 关于Android兼容性测试的细节, 参考程序描述([**program overview**](http://source.android.com/compatibility/overview.html)).

> **Licensing Google Mobile Services (GMS)**

在创建Android兼容性设备之后, 可以考虑获取Google Mobile Service(GMS)许可. Google私有的应用套件(Google Play, YouTube, Google Maps, Gmail, 等等)在Andorid设备上运行. GMS并不是AOSP开源项目的一部分. 它只能通过Google许可证获取. 获取GMS许可参见联系Google([**Contact Us**](http://source.android.com/compatibility/contact-us.html))



