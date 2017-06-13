
### [**兼容性测试套件**](http://source.android.com/compatibility/cts/index.html)

内容

 - CTS如何工作?
 - 流程
 - 测试用例类型
 - 涉及的领域

-----
#### CTS如何工作?

-----
兼容性测试套件(CTS)是一个免费的, 商业级的测试套件, 请在该地址[下载](https://source.android.com/compatibility/cts/downloads.html). CTS代表了设备满足兼容性的机制.

CTS运行在桌面机器上并直接在连接的设备或仿真器执行相关的测试用例. CTS为一组被设计成方便测试工程师集成到设备的（例如通过连续的构建系统CI）日常工作流程中的的单元测试集, 。它的目的是尽早揭示设备驱动的不兼容性，并确保软件在整个开发过程是兼容的.

CTS是自动测试工具包括以下两个主要的软件组件：

CTS交互测试工具运行在桌面计算机上，并管理执行测试项。 单独的测试用例在被测设备（DUT）上执行。 测试用例以Java代码进行编写为JUnit测试，并打包为Android .apk文件，安装到实际的设备目标上运行。 

兼容性测试套件验证器（CTS Verifier）是可供下载的CTS的补充。 CTS Verifier提供的API和功能的测试不能在没有手动输入（例如音频质量，加速度计等）的固定设备上测试。

The CTS Verifier is a tool for manual testing and includes the following software components:

The CTS verifier app that is executed on the DUT and collects the results.
The executable(s) or script(s) that are executed on the desktop machine to provide data or additional control for some test cases in the CTS Verifier app.
Workflow

CTS flow
Figure 1. How to use CTS

This diagram summarizes CTS workflow. Please refer to the subpages of this section starting with Setup for detailed instructions.

Types of test cases

The CTS includes the following types of test cases:

Unit tests test atomic units of code within the Android platform; e.g. a single class, such as java.util.HashMap.
Functional tests test a combination of APIs together in a higher-level use-case.
Future versions of the CTS will include the following types of test cases:

Robustness tests test the durability of the system under stress.
Performance tests test the performance of the system against defined benchmarks, for example rendering frames per second.
Areas covered

The unit test cases cover the following areas to ensure compatibility:

Area	Description
Signature tests	For each Android release, there are XML files describing the signatures of all public APIs contained in the release. The CTS contains a utility to check those API signatures against the APIs available on the device. The results from signature checking are recorded in the test result XML file.
Platform API Tests	Test the platform (core libraries and Android Application Framework) APIs as documented in the SDK Class Index to ensure API correctness, including correct class, attribute and method signatures, correct method behavior, and negative tests to ensure expected behavior for incorrect parameter handling.
Dalvik Tests	The tests focus on testing the Dalvik Executable Format.
Platform Data Model	The CTS tests the core platform data model as exposed to application developers through content providers, as documented in the SDK android.provider package: contacts, browser, settings, etc.
Platform Intents	The CTS tests the core platform intents, as documented in the SDK Available Intents.
Platform Permissions	The CTS tests the core platform permissions, as documented in the SDK Available Permissions.
Platform Resources	The CTS tests for correct handling of the core platform resource types, as documented in the SDK Available Resource Types. This includes tests for: simple values, drawables, nine-patch, animations, layouts, styles and themes, and loading alternate resources.