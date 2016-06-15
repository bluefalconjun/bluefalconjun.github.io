[文章源地址](https://matt.sh/howto-c)

**How to C in 2016**

----------

使用C语言的第一条规则: 如果能够避免则不要使用C语言.
如果需要使用C语言,那么遵循现代的规则.

C语言从1970年起开始发行.
人们在它的演化升级中从各个不同的时间点上"学习C语言",但是知识在学习过程中经常会产生定式,所以每个人在从他们开始学习时间点上,逐渐形成了不同类型的他们认为C语言应该的定式.

在你的"80s/90s学习到的"C语言开发的内容中,很重要的一点就是避免继续形成定式.

这篇文章假定你在使用一个符合现代标准的现代平台,并且你没有多余的兼容性要求. 我们不能因为某些公司拒绝升级20年的久系统就必须为旧的标准保持古老的兼容性.

----------

> **预检**:

**c99标准**(`c99`意味着"从1999年起的C标准"; `c11`意味着"从2011年起的C标准", 所以11>99).

- clang,default:
 - clang默认使用C11的扩展版本`(GNU C11 mode)`,所以不需要为现代的功能增加额外的选项.
 - 如果需要使用C11标准,需要指定`-std=c11`;同理`-std=c99`.
 - clang编译源文件要快过gcc.

- gcc需要指定`-std=c11`或`-std=c99`
 - gcc在编译源文件上要比clang慢,但有时它能产生更快的代码.对比的性能和回归测试很重要.
 - gcc-5默认设置为`GNU C11mode`,但仍然需要显式指定`-std=c11`或`-std=c99`

**优化**

- -O2,-O3
 -  通常你需要`-O2`,但有时你需要`-O3`. 在不同级别下(包括交叉编译)进行测试并保留最佳性能的程序.
- -Os
 - -Os在你担心缓存性能时帮助到你(确实存在这种情况)
-Os helps if your concern is cache efficiency (which it should be)

**警告**

- `-Wall -Wextra -pedantic`
 - [更新的编译器版本](https://twitter.com/oliviergay/status/685389448142565376) 接受 `-Wpedantic`, 但是同时也因为更好扩展兼容性接受老的`-pedantic`选项.
- 在测试过程中,应当在所有平台上增加`-Werror` 和 `-Wshadow`选项.
 - 加入`-Werror` 能够更好的产生产品级别的代码, 因为在不同的平台/编译器/库上同样的代码会产生不同的warning. 我们要避免因为不同版本的Gcc产生的不同的错误信息而删除掉这个user的整体编译流程.
- 扩展的优化选项包括`-Wstrict-overflow -fno-strict-aliasing`
 - 无论是指定`-fno-strict-aliasing` 还是确认保证只按照创建时的类型访问变量. 因为已经存在非常多的C代码通过别名访问类型, 在你不能控制整个底层的代码树时使用 `-fno-strict-aliasing` 选项是更加安全的.
- 由于目前Clang会对某些正确的语法进行警告,可以通过增加`-Wno-missing-field-initializers` 来避免.
 - GCC在 `GCC 4.7.0` 版本后移除了部分无效的警告.

**编译**

- 编译组件
 - 最常见的编译C项目的方法,是将所有的源文件分解编译为目标文件,然后在最后将它们连接在一起. 在增量开发的模式下, 这种方式工作的很好. 但对于性能和优化目标来讲, 是次优的选择. 编译器在这种模式下, 不能对文件边界下的情况进行可能的优化.
- LTO - 链接时优化
 - LTO通过在目标文件上加入中间处理标签, 然后在链接阶段进行多源码模块的优化. 这解决了"跨编译组件进行源代码分析优化的问题"
 - 这个操作会明显的减慢链接过程, 但是并行编译可以帮助提高时间: (`make -j`).
 - [clang LTO](http://llvm.org/docs/LinkTimeOptimization.html) [(guide)](http://llvm.org/docs/GoldPlugin.html)
 - [gcc LTO](https://gcc.gnu.org/onlinedocs/gccint/LTO-Overview.html)
 - 2016年, clang和gcc发布了LTO支持的版本. 在编译命令行选项中增加`-flto` 来完成目标编译和最终的库/程序链接支持. 
 - `LTO`仍然需要仔细的考虑, 有时如果程序中存在没有被直接使用而是被其他库使用的代码, LTO可能将其移除掉, 因为它检测到在全局环境下某些代码是没有被使用/执行到, 所以它在最终的链接结果中去掉了这部分代码.

**架构**

- `-march=native`
 - 授予编译器使用CPU全部功能集的权限.
 - 同时, 进行完整的性能/回归测试, 这很重要(同时比对跨平台编译器/编译器版本的结果)以保证所有加入的优化选项没有带来额外的副作用.
- `-msse2` 和 `-msse4.2` 在需要目标并不包含编译机器的功能时很有帮助.

> **代码编写**

**类型:**
如果你发现在新代码中重新输入`char/int/short/long/unsigned`, 那么你一定做错了.

在现在的程序中, 你应该 `#include <stdint.h>` 并使用标准类型.
更多的细节,参见[`stdint.h specification`](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/stdint.h.html).

通常的标准类型包括:

- `int8_t, int16_t, int32_t, int64_t` — 有符号整型.
- `uint8_t, uint16_t, uint32_t, uint64_t` — 无符号整型.
- `float` — 标准32-bit浮点型.
- `double` - 标准64-bit浮点型.
注意我们再也没有`char`类型. `char`实际在C语言中被错误的命名和使用.

开发者通常混淆使用`char`来作为"`byte`",即使他们在做无符号字节操作时. 更清楚的方法是使用uint8_t来表示单独的`一个无符号字节/八位字节值`, 使用`uint8_t*`来表示`无符号字节/八位字节值的指针`.

使用 `int` 或者不使用 `int`

某些读者指出他们确实很喜欢 `int`,  你必须从他们冰冷僵硬的手指下将 `int` 撬出来. 必须指出, 如果使用的类型在你控制之外发生改变, 那从技术上讲程序是不可能正确的.

同时从包含 [`inttypes.h`](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/inttypes.h.html) 的[基本原理](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/inttypes.h.html#tag_13_19_06)来讲, 使用非固定宽度的类型是不安全的. 如果你真的能够始终将 `int` 在某些平台上类型化为16bit, 在其他平台上类型化为32bit. 并且在所有你使用 `int` 的环境中测试16bit/32bit的边沿值, 那么请自行使用 `int` 吧 :).

在剩下的不能在头脑中完整描述多级别平台指定继承关系的人, 在写这些代码的时候我们还是使用固定宽度的类型. 这样可以自动的产生更安全的代码, 可以避免更多的测试和概念上的疑问. 

或者,使用规范中的语句来描述: "ISO C的标准整型转换规则会产生非预期的隐形改变".

祝 `int` 好运.

不使用 `char` 的例外:
2016中唯一允许使用`char`的情况是, 当已存在的API需要`char`(例如: `strncat, printf'ing "%s" ...`), 或者是初始化一个只读字符串(例如: `const char* hello = "hello";`), 因为字符串文本在C语言中的类型是 `char []`.

同时: 在C11中我们有原生unicode支持, 并且UTF-8类型的文本仍然是 char*. 即使在多字节字符串中 `const char* abcgrr = u8* "abc😬"` ;

不使用 `-{int,long,etc}` 的例外:
如果你在调用一个以原生类型作为返回值的或者作为参数的函数时, 可以使用这个函数原型声明或者是API规范的参数类型.

符号类型:
在任何时候, 你都不应该将单词 `unsigned` 写到代码中去. 现在我们能够编写不需要使用丑陋的 C 转换的代码, 它包换有多单词的类型, 造成阅读/使用的障碍. 当你可以使用 `uint64_t` 时, 不要再使用 `unsigned long long int`.  `<stdint.h>` 中的类型更加的明确和表意. 并且在代码发布和阅读中更加节省空间和明确意图. 

以整型处理指针:
但是, 你可能会说: "我需要为指针运算进行转换!" .
你可以这么说, 而且你错了.

指针运算的正确类型是在  `<stdint.h>` 中定义的  `uintptr_t` , 同时可用的 `ptrdiff_t` 在 [`stddef.h`](http://pubs.opengroup.org/onlinepubs/7908799/xsh/stddef.h.html) 中定义. 

不要使用:

    long diff = (long)ptrOld - (long)ptrNew;

而是使用:

    ptrdiff_t diff = (uintptr_t)ptrOld - (uintptr_t)ptrNew;

同理:

    printf("%p is unaligned by %" PRIuPTR " bytes.\n", (void *)p, ((uintptr_t)somePtr & (sizeof(void *) - 1)));

系统相关的类型:
你可以继续抱怨: "在32 bit系统上我需要32 bit的`long`, 64 bit系统上我需要64 bit `long`".

在不考虑这句话中有意的引入在不同系统依赖中代码难度的情况下, 你仍然没有打算使用系统相关的 `long` 类型.

在这种情况下, 你应该使用 `intptr_t`  -- 定义为当前平台的字长度的整型类型.
在32 bit系统上, `intptr_t` 为 `int32_t`.
在64 bit系统上, `intptr_t` 为 `int64_t`.
同时`intptr_t` 存在有 `uintptr_t` 的形式.

为了存储指针偏移, 我们有恰当命名的 `ptrdiff_t` 这个正确类型, 它用来存储指针减法的结果.

最大值存储
你需要一个整数类型能够存储当前系统上支持的最大整数吗?
开发者一般倾向使用最大的已知类型来处理这种情况, 例如将小的无符号类型转换为`uint64_t`, 但是可以使用更有效的方法来保证某些类型能够存储其他所有的数值. 
最安全的存储任何整型的容器是`intmax_t`(`uintmax_t`). 你可以将任何有符号整型赋值或者转换到`intmax_t`而不丢失任何精度. `uintmax_t`同理.

其他类型
最常见使用的系统相关类型是有 [`stddef.h`](http://pubs.opengroup.org/onlinepubs/7908799/xsh/stddef.h.html)提供的`size_t`.
`size_t`基本上相当于"能够存储最大的数组索引的整型",这意味着它可以存储当前程序中最大的内存偏移.
在实际使用中, `size_t`是`sizeof`操作的返回类型.
在任何情况下: `size_t`在现代系统中通常定义为同`uintptr_t`一样大小. 所以在32 bit系统上`size_t == uint32_t.` 在64 bit系统上`size_t == uint64_t.`
同时也存在一个`ssize_t`,它是一个有符号的`size_t`, 用来接收某些在出错时返回-1的库函数返回值(注意: `ssize_t`是`POSIX`标准,`windows`接口并不适用). 
综合以上,是否需要在可变的系统依赖的`size`上编写的函数参数中使用`size_t`呢? 技术上讲, `size_t`是`sizeof`的返回类型, 所以任何接受字节数目长度的`size`返回值的函数都可以使用`size_t`.
其他的使用包括: `size_t`是`malloc`的参数类型, ssi`z`e_t是`read()/write()`函数的返回类型(`windows`上`ssize_t`并不存在返回值是整型).

打印类型
打印过程中不应该进行类型转换. 必须使用 [`inttypes.h`](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/inttypes.h.html)中正确的类型指定.
这些包括但不仅限于:

- `size_t - %zu`
- `ssize_t - %zd`
- `ptrdiff_t - %td`
- 原始指针值 - %p(在现代编译器中打印hex;需要先将指针转换为(void *)).
- int64_t - "%" PRId64
- uint64_t - "%" PRIu64
 - 64 bit类型需要通过`PRI[udixXo]64` 类型宏来进行打印.
 - 为什么?
  - 在某些系统上64 bit值是long, 某些是long long.
  - 实际上不使用这些格式化宏不可能正确指定跨平台的格式化字符串, 因为这些类型转换实际上在你的控制之外发生(记住, 打印前进行数值转换并不安全,也不符合逻辑).
- `intptr_t — "%" PRIdPTR`
- `uintptr_t — "%" PRIuPTR`
- `intmax_t — "%" PRIdMAX`
- `uintmax_t — "%" PRIuMAX`

使用`PRI*`格式化指定的注意事项: `PRI*` 实际上是宏定义, 它们在基于平台的基础上被扩展为正确的打印类型指定. 这意味着你不能这么使用: 

    printf("Local number: %PRIdPTR\n\n", someIntPtr);
因为它们是宏,必须这样使用:

    printf("Local number: %" PRIdPTR "\n\n", someIntPtr);

注意上例中 `%` 符号在格式化字符串文本中, 而类型指定符在文本外,因为所有的相邻字符串都会被预处理器级联成为一个最终的联合字符串文本.

C99允许在任何地方进行变量声明. 
所以,不要做下面的操作:

    void test(uint8_t input) {
        uint32_t b;
        if (input > 3) {
            return;
        }
        b = input;
    }
而是这么做:

    void test(uint8_t input) {
        if (input > 3) {
            return;
        }
        uint32_t b = input;
    }

警告:如果使用了紧密的循环, 测试初始化变量的不同位置. 有时分散的声明会操作非预期的耗时. 对于通常的非快速排序代码(基本上到处都是), 最好是越清楚越好, 紧贴着初始化进行类型定义是一个非常好的可读性操作.

C99允许在 `for` 循环中声明计数器
所以,不要这么做:

    uint32_t i;
    for (i = 0; i < 10; i++)
应当这么做:

    for (uint32_t i = 0; i < 10; i++)
以上例子的例外: 如果你需要在循环终止后保留计数器值, 那么很明显不要在循环体范围内定义计数器.

现代的编译器支持 `#pragma once`
所以不要这么写:

    #ifndef PROJECT_HEADERNAME
    #define PROJECT_HEADERNAME
    .
    .
    .
    #endif /* PROJECT_HEADERNAME */
使用以下定义:

    #pragma once
   `#pragma once`告诉编译器只包含当前头文件一次. 无需在使用三行的头文件隔离语法. 这个编译选项在大部分的平台和编译器上都是支持的, 建议在所有使用命名头文件隔离中使用它.

细节请参见支持[`pragma once`](https://en.wikipedia.org/wiki/Pragma_once)的编译器列表.

C允许自动分配的数组的静态初始化.
所以,不要这么写:

    uint32_t numbers[64];
    memset(numbers, 0, sizeof(numbers));
使用以下写法:

    uint32_t numbers[64] = {0};
C允许自动分配的数据结构的静态初始化.
所以,不要这么写:

    struct thing {
        uint64_t index;
        uint32_t counter;
    };
    struct thing localThing;
    void initThing(void) {
        memset(&localThing, 0, sizeof(localThing));
    }
使用以下写法:

    struct thing {
        uint64_t index;
        uint32_t counter;
    };
    struct thing localThing = {0};
重要提示:如果结构体中存在填充部分, 使用 `{0}` 方法并不对额外的填充位进行 `0` 赋值. 例如, `thing` 结构体在`counter`之后存在4字节的填充(64 bit系统上)因为结构体被按照字宽度进行补齐. 如果需要对包括填充位的整个结构体进行清零, 必须使用`memset(&localThing, 0, sizeof(localThing))`,因为 `sizeof(LocalThing)==16` 字节,而其中可寻址的内容只有 `8+4=12` 字节.

如果需要对已分配的结构体进行再初始化,为后期的赋值定义一个全局的0值结构:

    struct thing {
        uint64_t index;
        uint32_t counter;
    };
    static const struct thing localThingNull = {0};
    .
    .
    .
    struct thing localThing = {.counter = 3};
    .
    .
    .
    localThing = localThingNull;
如果幸运的在C99(或更新)的环境中,你可以使用联合文法来代替保留一个全局的"0值结构体"(参见：从2001起: [The New C: Compound Literals](http://www.drdobbs.com/the-new-c-compound-literals/184401404))
联合文法允许直接从匿名结构体中赋值：

    localThing = (struct thing){0};

C99增加了变长数组(C11将其定为可选)
所以,不要这么写:

    uintmax_t arrayLength = strtoumax(argv[1], NULL, 10);
    void *array[];
    array = malloc(sizeof(*array) * arrayLength);
    /* remember to free(array) when you're done using it */
使用以下写法:

    uintmax_t arrayLength = strtoumax(argv[1], NULL, 10);
    void *array[arrayLength];
    /* no need to free array */

**严重警告**: 变长数组通常同常规数组一样是栈分配的. 如果你不会静态的创建一个3M个元素的常规数组,不要使用这个句法来动态创建3M元素的数组. 这不是可扩展的Python/Ruby的自动增长链表. 如果指定了一个运行时数组长度,而对于当前栈来讲长度过大, 程序会变得异常(崩溃,安全问题). 变长数组易于在简单,单一目标的环境下使用, 而不是针对产品级的代码. 如果这个数组某些时候是3个元素,也可能是3M个元素, 千万不要使用变长数组.

当你在运行时碰到这种情况(或者进行快速的单次测试),注意到变长数组语法是很重要的. 但它基本上被认为是一种很危险的[非常规模式](https://twitter.com/grynspan/status/685509158024691712), 因为程序可能在忘记检查元素大小边界或者是在一个很小的可用栈空间的系统上很轻易的崩溃.

注意: 在这种情况下,必须使用可用的长度来定义数组.(例如:小于几KB,有时某些平台上栈溢出的最大大小为4KB). 不要在栈上分配大的数组,但是如果你清楚当前系统栈的限制, 使用[`C99 VLA`](https://en.wikipedia.org/wiki/Variable-length_array)功能要好过通过malloc在堆上分配数组.

另一个注意点: 以上操作没有用户输入检查, 所以用户很容易通过分配一个超大的VLA来让程序崩溃. 尽管有些用户会通过VLA语法来操作. 但是进行边界检查和限制还是能够轻微的避免以上情况的出现.

C99允许注释非重叠的指针参数.
查看[限制关键字](https://en.wikipedia.org/wiki/Restrict),通常是  `__restrict`

**参数类型:**
如果函数接受**任意的**输入数据和长度并进行处理, 那么不要限制参数的输入类型.
不要这么写:

    void processAddBytesOverflow(uint8_t *bytes, uint32_t len) {
        for (uint32_t i = 0; i < len; i++) {
            bytes[0] += bytes[i];
        }
    }
应当这么写:

    void processAddBytesOverflow(void *input, uint32_t len) {
        uint8_t *bytes = input;
        for (uint32_t i = 0; i < len; i++) {
            bytes[0] += bytes[i];
        }
    }

函数的输入类型描述了代码的接口, 而不是描述代码如何处理这些参数的. 以上代码的接口表示"接受字节类型的数组和长度", 所以不要限定接口的调用者只使用`uint8_t`类型的字节流. 也许调用者会传入老式的`char*` 或者其他的类型.

通过声明输入类型为`void *`,然后在内部重新分配或者转换为实际函数使用的类型, 可以减少函数调用者考虑函数体内部的抽象操作.

部分读者会指出这个例子中的对齐问题, 但是我们支持访问输入的单字节元素, 所以该操作是可行的. 如果我们将输入转换成为更宽的类型, 我们需要考虑到对齐的问题. 在对于跨平台上的对齐问题的处理上. 参见[`Unaligned Memory Access.`](https://www.kernel.org/doc/Documentation/unaligned-memory-access.txt) (提醒: 当前部分关于跨平台下的C语言内容比较复杂, 所以使用任何其中的例子需要相关的知识配合).

**返回值类型**
C99提供了强力的`<stdbool.h>` 来定义`true`为`1`. `false`为`0`.

对于正确/错误的返回值,函数应该返回true/false, 而并不是一个int32_t类型来手动指定1/0(或者更坏的情况,1/-1,0是正确返回?1是错误? -1是错误?).
如果某个函数需要在废止操作时对输入参数进行改变,那么不要返回改变的指针,整个API需要对可能废止的输入强制使用双指针. 编写"对于某些操作, 返回值废止了输入"的代码太容易引发错误的使用.

不要编写如下的代码:

    void *growthOptional(void *grow, size_t currentLen, size_t newLen) {
        if (newLen > currentLen) {
            void *newGrow = realloc(grow, newLen);
            if (newGrow) {
                /* resize success */
                grow = newGrow;
            } else {
                /* resize failed, free existing and signal failure through NULL */
                free(grow);
                grow = NULL;
            }
        }
        return grow;
    }
使用如下的代码:

    /* Return value:
     *  - 'true' if newLen > currentLen and attempted to grow
     *    - 'true' does not signify success here, the success is still in '*_grow'
     *  - 'false' if newLen <= currentLen */
    bool growthOptional(void **_grow, size_t currentLen, size_t newLen) {
        void *grow = *_grow;
        if (newLen > currentLen) {
            void *newGrow = realloc(grow, newLen);
            if (newGrow) {
                /* resize success */
                *_grow = newGrow;
                return true;
            }
            /* resize failure */
            free(grow);
            *_grow = NULL;
            /* for this function,
             * 'true' doesn't mean success, it means 'attempted grow' */
            return true;
        }
        return false;
    }
或者,更好的方式是使用:

    typedef enum growthResult {
        GROWTH_RESULT_SUCCESS = 1,
        GROWTH_RESULT_FAILURE_GROW_NOT_NECESSARY,
        GROWTH_RESULT_FAILURE_ALLOCATION_FAILED
    } growthResult;
    
    growthResult growthOptional(void **_grow, size_t currentLen, size_t newLen) {
        void *grow = *_grow;
        if (newLen > currentLen) {
            void *newGrow = realloc(grow, newLen);
            if (newGrow) {
                /* resize success */
                *_grow = newGrow;
                return GROWTH_RESULT_SUCCESS;
            }
            /* resize failure, don't remove data because we can signal error */
            return GROWTH_RESULT_FAILURE_ALLOCATION_FAILED;
        }
        return GROWTH_RESULT_FAILURE_GROW_NOT_NECESSARY;
    }

> **代码格式**
   
编码规范是非常重要同时又毫无用处的.
如果某项目有50页的编码规范参考,那么没有人能帮助你. 但是如果该项目代码不可读,那么没有人想帮助你.
解决方案是持续使用自动代码格式化工具.
2016可用的C格式化工具是clang-format. clang-format是默认所有自动格式化C代码中最好的,而且仍在开发中.
这是推荐的clang-format参数的脚本:

    #!/usr/bin/env bash
    
    clang-format -style="{BasedOnStyle: llvm, IndentWidth: 4, AllowShortFunctionsOnASingleLine: None, KeepEmptyLinesAtTheStartOfBlocks: false}" "$@"
    Then call it as (assuming you named the script cleanup-format):
    
    matt@foo:~/repos/badcode% cleanup-format -i *.{c,h,cc,cpp,hpp,cxx}
    The -i option overwrites existing files in place with formatting changes instead of writing to new files or creating backup files.
如果有很多文件需要处理,那么可以并行递归的访问整个代码树:

    #!/usr/bin/env bash
    
    # note: clang-tidy only accepts one file at a time, but we can run it
    #       parallel against disjoint collections at once.
    find . \( -name \*.c -or -name \*.cpp -or -name \*.cc \) |xargs -n1 -P4 cleanup-tidy
    
    # clang-format accepts multiple files during one run, but let's limit it to 12
    # here so we (hopefully) avoid excessive memory usage.
    find . \( -name \*.c -or -name \*.cpp -or -name \*.cc -or -name \*.h \) |xargs -n12 -P4 cleanup-format -i
    Now, there's a new cleanup-tidy script there. The contents of cleanup-tidy is:
    
    #!/usr/bin/env bash
    
    clang-tidy \
        -fix \
        -fix-errors \
        -header-filter=.* \
        --checks=readability-braces-around-statements,misc-macro-parentheses \
        $1 \
        -- -I.
clang-tidy是一个按照规范进行代码整理的工具. 以上的选项是进行两个修正:

- `readability-braces-around-statements` — 强制所有的`if/while/for` 后面的执行段被`{}`所封闭.
对于C存在在循环体或者判断中如果只包含单个执行体则可以使用"可选封闭"语法来讲, 这是一个遗留的事故!如果使用"当时编译器支持这么做"来争辩并不意义, 而且也对代码可读性/管理/理解/扩展没有帮助. 你并不是为了取悦编译器而编码,而是为了未来使用和维护代码的人而编码. 尤其是当一段时间以后每个人都会忘记为什么这段代码存在这个位置上的情况发生.
- `misc-macro-parentheses` — 自动在所有使用宏定义的参数上增加括号.

clang-tidy工作起来时非常好, 但是也会被部分复杂的代码所阻塞. 同时它不负责格式化代码段, 所以在使用它对代码进行括号扩展和对齐后要进行clang-format.

**可读性**
写作到这部分的时候,开始慢下来了...

注释
代码文件的每部分都应当是逻辑自洽的.

文件结构
限制每个文件在1000行(1500行已经够多了). 如果源代码中包含测试部分(测试静态函数),按照需要来调整它们.

> **杂项提醒:**

永远不要使用`malloc`.
永远使用`calloc`. 不需要考虑内存清零的性能损耗. 如果不喜欢calloc的函数原型`(object cout,size per object).` 可以通过定义`#define mycalloc(N) calloc(1, N).`来调整.

读者在这里进行了一些评论:
`calloc`在**巨大空间**的分配上确实存在性能损耗.
`calloc`在某些平台上确实存在性能损耗(精简的嵌入式系统,游戏主机,30年老的硬件...).
重定义`calloc`并不是一个好主意.
不使用`malloc`的一个好理由是它不进行整型溢出的检查,这是一个潜在的安全风险. 
`calloc`分配能够避免警告对分配的内存进行无意的读取或是拷贝, 因为它在分配的时候就自动初始化为0了.
这些都是很好的建议,这也是为什么我们要在不同的编译器/平台/操作系统/硬件设备上进行性能测试和回归测试.

另外一个直接使用`calloc()`而不是重定义它的优点是, `calloc()`不同于`malloc()`, 它能够检查整型溢出的问题. 因为它将参数相乘并返回最终的分配大小. 如果只是分配很小的东西. 建议使用重定义. 如果是为了大量的数据进行分配. 尽量使用`calloc()`的原型.
没有建议是能够通用的,但是尝试给出一些确定的通用建议能够终止去读一些语法标准的书...

对于calloc()如何提供给你干净的可以free的内存. 参见以下描述:

- [`Benchmarking fun with calloc() and zero pages (2007)`](https://blogs.fau.de/hager/archives/825)
- [`Copy-on-write in virtual memory management`](https://en.wikipedia.org/wiki/Copy-on-write#Copy-on-write_in_virtual_memory_management)

我仍然坚持在2016这个章节中推荐使用**calloc()** (假定是: x64目标平台, 常人大小的数据, 不包括常人基因组大小的数据). 

子提醒: `calloc()`提供的已清零的内存是一次成型的. 如果`realloc()`你的`calloc()`分配, 额外增长的`memory`并没有清零. 它可能被kernel填充为任何数据. 如果需要的话你必须手动对额外部分进行清零`memset().`

永远不要`memset`(如果可以避免的话)
永远不要`memset(ptr, 0, len)`, 如果你可以静态初始化一个结构(或数组)为0. 或者通过对它进行重新赋值为已初始化为0的结构. 或者是从原有的已清零结构中赋值.
但是,`memset()`仍然是你对结构进行清零的唯一方法. 因为{0}仅能对定义的位进行赋值, 未定义的填充位无法赋值.


> **扩展阅读:**  

略


> **跋**

在大规模上编写正确的代码实际上是不可行的. 我们需要考虑众多的操作系统,运行时,库和硬件. 甚至不需要考虑类似于RAM上的随机比特反转,或者是块设备因为未知问题返回错误值.

我们能做的是编写最简单,易懂的代码并且尽量减少迂回或者是未文档化的技巧操作.

原作者:
[-Matt](matt@matt.sh) — [@mattsta](https://twitter.com/mattsta) — [☁mattsta](https://github.com/mattsta)

> **归属: 略**




