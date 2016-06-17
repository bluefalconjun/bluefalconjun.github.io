
###[**Graphics**](http://source.android.com/devices/graphics/index.html)

![Graphics HAL](http://source.android.com/devices/graphics/images/ape_fwk_hal_graphics.png)

-----
**android framework**通过使用厂商实现的图形驱动来提供多种2D和3D的**graphics**绘图API. 这些功能通过编入图形驱动的HAL来提供.

app开发者通过两种方式向屏幕画图. **Canvas**/**OpenGL**. 参见**android graphics arch**来理解**android** **graphics**组件的功能.

**android.graphics.Canvas**是一套开发者最常使用的2D图形API. **Canvas**完成了所有的标准和自定义的[**android.view.Views**](http://developer.android.com/reference/android/view/View.html)的绘图操作. 

在Android中, 带有硬件加速支持的**Canvas API**由名为**OpenGLRenderer**的绘图库来完成. 它将**Canvas**调用转接到**OpenGL**的操作, **OpenGL**的操作可以在**GPU**上完成.

从**Android** 4.0(**ICS**)开始, 硬件加速的**Canvas**被缺省开启. 因此在此版本之后的设备均需要硬件**GPU**来支持**OpenGL ES2.0**.  

查看[**Hardware Acceleration Guide**](https://developer.android.com/guide/topics/graphics/hardware-accel.html)来获取详细信息. 并区分硬件/软件绘图的不同路径方式.

除了**Canvas**之外,开发者绘图主要通过**OpenGL ES**直接向**surface**绘图. 开发者通过**android**提供的[**android.opgl**](http://developer.android.com/reference/android/opengl/package-summary.html)包来在SDK上完成GL实现,或者在[**Android NDK**](https://developer.android.com/tools/sdk/ndk/index.html)上完成**Native API**调用实现.

Android开发人员可以通过[**drawElements Quality Program**](http://source.android.com/devices/graphics/testing.html)来测试**OpenGL ES的**功能.

----------
> **Android graphics components**

-----
无论开发者使用何种绘图API, 所有的绘制操作均指向"**surface**". 

**surface**代表**buffer queue**的**producer**端, 常见的**consumer**端是**surfaceflinger**. **android**平台上创建的每一个**window**都包含有**surface**. 所有Activity的**surfaces**绘制均由**surfaceflinger**进行合成并绘制到屏幕上.

下图描述了关键组件是如何联和工作的:

![How Surface Are Rendered](http://source.android.com/devices/graphics/images/ape_fwk_graphics.png)

以下是主要的组件:

-----
> **Image Stream Producers**

-----
图像流生产方是可以产生提供给处理方图像数据的任何模块,例如**OpenGL ES**,**Canvas 2D**或者是**mediaserver**内的视频解码器.

> **Image Stream Consumers**

最常见的图像流处理者是**Surfaceflinger**, 它在系统中通过**Window Manager**提供的信息对所有可见的**surface**进行处理, 并提供合成surface到显示上的服务.

**Surfaceflinger**是系统中唯一可以对显示内容进行处理的服务. 它使用**OpenGl**和**Hardware Composer**来完成多组**surface**的合成.

其他的**OpenGl**应用同样可以处理图像流, 例如**camera**应用可以处理**camera**预览数据, 非GL的应用同样可以是处理方, 例如**ImageReader**类.

-----
> **Window Manager**

-----
**window manager**是一个包含各类**view**的控制窗口的系统服务. 每个**window**均包含有**surface**.  

该系统服务监视每个**window**的生存期,输入/焦点转移事件,屏幕方向,颜色过渡,动画,位置等多方面. 

a**Window Manager**将所有的**window**信息发送给**surfaceflinger**, 后者可以通过这些信息来完成显示上的**surface**合成操作.

-----
> **Hardware Composer**

-----
**Hardware Composer**是显示子系统的硬件抽象.  **surfaceflinger**能够通过委托它完成特定的合成操作, 来减低**OpenGL**和**GPU**的工作量.  **surfaceflinger**也可以通过作为**OpenGL ES**的客户端来完成任务.  在这种情况下, **OpenGL ES**可以帮助完成合成的操作,这要比完全使用**GPU**来进行操作更为节省资源.

**Hardware Composer**另外负责所有**Android**显示系统的绘制工作. 它必须支持events. **VSYNC**就是其中之一. 另外一种event是即插即用的**HDMI**热插拔信号.

查看[**Hardware Composer HAL**](http://source.android.com/devices/graphics.html#hardware_composer_hal) 来获取细节实现.

> **Gralloc**

**gralloc**是用来为图像流生产方进行内存分配的图像内存分配器.查看[**Gralloc HAL**](http://source.android.com/devices/graphics.html#gralloc) 来获取细节实现.

数据流程:
下图描述了**android**图形系统的工作流程
![graphics data flow](http://source.android.com/devices/graphics/images/graphics_pipeline.png)

左边橙色的对象是产生图像数据的绘图方,例如主页/状态栏/系统UI. **surfaceflinger**是合成代理而**hardware composer** 是合成执行者.

> **BufferQueues**

**Bufferqueue**为**android**图形组件提供交互的方式,下图表示了常见的**producer/consumer**之间的固定流程.  当**producer**将**buffer**提交后,**surfaceflinger**负责将所有的内容合成显示出来.

**Bufferqueue**通讯流程:
![BufferQueue communication process](http://source.android.com/devices/graphics/images/bufferqueue.png)

**bufferqueue**包含将图像流**producer**和**consumer**连接起来的逻辑操作. 图像**producer**的例子有由camera HAL支持的camera预览,或者基于OpenGL ES的游戏. 图像**consumer**的例子有**surfaceflinger**,或者从camera预览上进行显示的**openGL ES app**.

**bufferqueue**是一个联合**buffer**池的数据结构. 它通过队列和**Binder IPC**在不同进程中传递**buffer**. **IGraphicsBufferProducer** (它是**SurfaceTexture**的一部分)是**bufferqueue**提供给**producer**的接口,  **bufferqueue**经常是被绘制到**surface**上的. 然后由另一个进程中的**OpenGL**消费者使用.  **bufferqueue**有三种运行方式:

类同步模式(**Synchronous-like mode**) - **bufferqueue**缺省的工作模式, 每个由**producer**产生的buffer都由**consumer**使用. 这种模式下所有的**buffer**都不会被丢弃. 如果**producer**产生的太快而**consumer**处理太慢. 那么它会阻塞**producer**直到有空闲**buffer**出现.

非阻塞模式(**Non-blocking mode**) - 这种模式下, 如果没有空闲**buffer**给**producer**它会产生一个错误提示, 这种方式可以避免潜在的死锁出现. 因为常见情况下图像**framework**处理非常的复杂.

丢弃模式(**Discard mode**) - 丢弃模式下,最旧的**buffer**会被废弃, 这样不会产生错误提示或者阻塞. 例如: 如果GL朝**text view**进行持续的绘制, 那么**buffer**必须被丢弃.


> **Synchronization framework**

由于**Android** **graphics**并没有严格的对应限制, 供应商可以通过自己的**driver**来隐含实现同步机制. 

同步**framework**会明确的指出系统中异步的模块之间的依赖. **framework**提供了简单的API来通知组件**buffer release**的事件.  同时, 它也可以让存在于**kernel space**的**driver**和**user space**或者**user space**中的多进程进行异步原语通讯.

例子: 一个应用将操作指派给**GPU**执行, **GPU**开始对图像进行绘制, 即使图像没有完全绘制到内存**buffer**时, **buffer**的指针仍然可以传递给窗口合成器, **buffer**同时会附有一个标签锁, 来指定**GPU**何时完成工作. 窗口合成器可以直接在绘制完成前开始工作, 将**buffer**传递给显示控制器. 在这种规则下, **CPU**进行的工作可以提前. 当**GPU**绘制完成时, 显示控制器可以直接对图像进行显示.

同步**framework**机制同样允许供应商实现内部继承的同步资源. 同时也提供对图形处理管道的可见**debug**.

