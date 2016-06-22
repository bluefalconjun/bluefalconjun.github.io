
### 长命名实在是太长了...

-----
原文参见[**`这里`**](http://journal.stuffwithstuff.com/2016/06/16/long-names-are-long/?utm_source=wanqu.co&utm_campaign=Wanqu%20Daily&utm_medium=website)

-----
进行严格的代码审核, 是Google执行的很明智的规定. 每个修改, 在能够加入到主干之前, 会经过至少两种方式的审核. 

首先, 当前team的人员会进行简单的review, 以确保当前的代码做了该做的事情.

然后, 第二级的审核被称之为**`可读性(readability)`**. 它主要确认当前的代码是能够被很好的去读的.

 - 它很容易被理解和维护吗?
 - 它符合语言的惯例和格式吗?
 - 它拥有很好的自注释性(**`[Tips: 自注释指代码能够替代内部注释向读者描述其意图]`**)吗?

Google内部已经开始使用[**`Dart`**](https://www.dartlang.org/)语言, 所以我已经进行过非常多次的以上类型的代码审核. 作为语言开发者, 这项任务很令人着迷. 

我能够从第一手的角度查看人们如何使用**`Dart`**的, 这对语言开发的修正工作很有帮助. 同时我能够统计出那些常见的错误, 最多被用到的功能. 感觉上像是一个在对于本地居民进行历史记录的学者.

当然, 这不是这篇文章的主要目的, 甚至, 本文其实和**`Dart`**也无关. 这篇文章的主要目的, 是因为在很多代码中看到的情况, 逐渐将我推到一个艰难的处境: **`变量命名(identifiers)`**实在太TM的长了...

的确, 命名可以很短. 回退到当C语言仅需要外部标识符在前六个字节是唯一的; 自动补全功能(**`auto-complete`**)还没有被发明; 每次键入的操作像在雪地上一档起步那么艰难的时候, 这就是它的问题. 

我很高兴我们现在能够生活在幻想乌托邦一样的世界. 键入操作变得如此如此的简单(原文: **`keyboard farts like p, idxcrpm, and x3 are rare`**).

但是钟摆朝另一个方向走得太远了. 我们不能做海明威(**`Hemingway`**)(**`猜测是不是小说人物名字很短`**), 但我们也不需要成为田纳西.威廉斯(**`Tennessee Williams`**). 代码使用了过长的命名同样会造成释义不明的情况. 

巨型的命名标识将缩短对其进行的操作代码, 视觉上很难看清, 并且会因为行数的问题分成多段, 从而造成对代码流程的打断.

长的类命名将会造成使用者难以从其类型上构造变量, 而且会造成大规模的, 粗糙的表达式 难以使用本地类型进行构建.  

长的方法命名会使得同它自己一样重要的参数列表变得晦涩难懂. 

长的变量命名会在重复使用时使代码变得更加复杂, 和方法级联使用会让整体更个更加混乱.

我见过超过60字节的变量名. 你甚至可以在里面写一首诗(这将吸引更多并不需要使用它的读者们). 不过, 现在我们可以帮助来改善这一点:

-----
> **选一个好名字**

-----
一个命名有两个明确的目标:

 - 很清楚: 你需要知道这个名字指向什么.
 - 很精确: 你需要知道这个名字**`不`**指向什么.

当命名完成以上目标时, 任何其他的字符都是超重的. 这里有一些我编写代码的方案:

-----
**1. 忽略用来定义变量或参数类型的字段**

如果使用静态类型系统的语言(C/C++), 使用者通常知道变量的类型. 由于方法实现一般比较简单, 所以当查看一个需要推断才知道类型的本地变量, 或者是代码审核和其他的静态分析器不可用的情况下, 基本上都可以在几行代码里就能知道变量所对应的类型.

所以将类型说明加入到变量名中是多余的. 我们可以理所当然的丢弃[**`匈牙利命名法`**](https://en.wikipedia.org/wiki/Hungarian_notation). 丢掉这种命名法.

    // Bad:
    String nameString;
    DockableModelessWindow dockableModelessWindow;
    
    // Better:
    String name;
    DockableModelessWindow window;


通常情况下, 对于集合来讲, 最好使用复数的名词组合来描述其内容, 使用单独的名词来描述集合会略微混淆. 如果使用者更在乎集合中的内容, 命名名词应当反映这一点.

    // Bad:
    List<DateTime> holidayDateList;
    Map<Employee, Role> employeeRoleHashMap;
    
    // Better:
    List<DateTime> holidays;
    Map<Employee, Role> employeeRoles;


这同样对方法命名有效. 方法的命名不需要描述它的参数或者是类型 - 参数列表已经为你列出来了.

    // Bad:
    mergeTableCells(List<TableCell> cells)
    sortEventsUsingComparator(List<Event> events,
        Comparator<Event> comparator)
    
    // Better:
    merge(List<TableCell> cells)
    sort(List<Event> events, Comparator<Event> comparator)

以下这种方式能够帮助调用者的代码更好的阅读.

    mergeTableCells(tableCells);
    sortEventsUsingComparator(events, comparator);

这只是我推荐的方法, 有人响响... 应应... 吗?

---
**2. 忽略命名中不能消除歧义的字段**

某些开发人员倾向在命名中使用所有他们知道的词语来进行描述. 但是要记住, 命名只是一个标识符: 它只是告诉你它定义了什么. 它不是让读者去理解这个事物的所有信息的详细目录. 定义才是做这件事情的. 命名只是将事物指向它的定义.

当我看到一个类似于**`recentlyUpdatedAnnualSalesBid`**(最近更新的全年销售指标)这样的命名时, 我会问:

 - 存在不是最近的全年销售指标更新吗?
 - 存在最近的全年销售指标没有被更新吗?
 - 存在最近更新的销售指标不是全年的吗?
 - 存在最近更新的全年指标是和销售无关的吗?
 - 存在最精更新的全年销售额不是指标的吗?

对上面问题的回答中任何一个"不", 都意味着引入了不需要的字段.

    // Bad:
    finalBattleMostDangerousBossMonster;
    weaklingFirstEncounterMonster;
    
    // Better:
    boss;
    firstMonster;

很明显的是, 这里可能稍微有点过分, 对第一个命名的缩减方式也许会造成部分的混淆. 但是, 任何时候你都可以先将这种命名方式进行下去. 在后面的时候如果很明显这种命名造成了混淆或者不精确你可以返回来继续完善它, 这将要好过后面的时候回来继续缩减命名标识.

-----
**3. 避免能够从周围上下文中获取的字段**

在该段文字中, 我可以使用**`I`**, 因为你能够看到这篇文章是由**`Bob Nystrom`**发布的. 我的大饼脸就挂在那里. 我不需要在文章里面处处提到**`Bob Nystrom`**(尽管Bob Nystrom这个名字经常会诱惑Bob Nystrom这么去做:)). 

代码以同样的方式工作. 方法/字段在类的某段内容中出现. 变量在方法的某段内容中出现. 使用这段上下文的增强释义, 不要重复.

    // Bad:
    class AnnualHolidaySale {
      int _annualSaleRebate;
      void promoteHolidaySale() { ... }
    }
    
    // Better:
    class AnnualHolidaySale {
      int _rebate;
      void promote() { ... }
    }

在实际情况下, 这意味着一个命名被深层嵌套的越多, 它就被越多的上下文所包括. 那么更多层内部的标识就拥有更短的命名. 它的影响是更小作用域的命名标识符拥有更短的名字.

-----
**4. 避免不能足够表达任何事物的字段**

这种情况经常在游戏业中出现. 很多人倾向于服从通过增加很多严重商业性的字句来华丽的修饰它们的命名标识符的诱惑. 我猜这是因为这能够让他们的代码看起来感觉很重要, 于是, 顺便让他们觉得自己更加重要.

很多时候, 这些词语并不携带任何有意义的信息. 只是一堆空话/套话. 常见内容包括: 数据, 状态, 数量, 数值, 管理者, 引擎, 物体, 实体, 和 实例(**`data, state, amount, value, manager, engine, object, entity, and instance`**).

好的名字能够在阅读者的脑海里绘出图画. 将某事物命名为"管理者"并不能向读者描绘任何关于它是什么的图像. 它是做性能评估的吗? 它提供加薪吗?

经常问自己: **`如果我移除这个字段, 命名能够指明同样的东西吗?`**. 如果是的话, 这个字段并不重要: 从岛上把它踢出去.

-----

> **华芙饼干的实用指南**

-----
为了让你对实践中这些规则如何工作的, 这里有一个违反所有这些规则的例子.
这个浮夸的例子基本上是实际的我审核过的代码, 它令人心碎...

    class DeliciousBelgianWaffleObject {
      void garnishDeliciousBelgianWaffleWithStrawberryList(
          List<Strawberry> strawberryList) { ... }
    }

我们从参数类型中得知它装饰的是草莓列表(**`#1`**), 那么, 让我们把这些字段去掉:

    class DeliciousBelgianWaffleObject {
        void garnishDeliciousBelgianWaffle(
            List<Strawberry> strawberries) { ... }
    }

除非我们的程序需要处理四种口味的比利时华芙饼, 或者是其他国家的饼干, 否则我们可以去掉以下形容词(**`#2`**):

    class WaffleObject {
      void garnishWaffle(List<Strawberry> strawberries) { ... }
    }

方法是包含在华芙饼对象中的, 所以我们知道它是用来装饰什么东西的(**`#3`**):

    class WaffleObject {
      void garnish(List<Strawberry> strawberries) { ... }
    }

很明显它是一个对象. 任何事物都是对象. 这是"面向对象"所定义的(**`#4`**):

    class Waffle {
      void garnish(List<Strawberry> strawberries) { ... }
    }

恩, 现在好了. 
我猜想应该有更加简单的指南. 你可能认为这些东西是无关紧要的, 但是我相信命名标识符是我们编程时最基础的任务之一. 
命名标识符是我们强加到数据海洋的计算中的重要的结构.

-----
> update on stackedit.on @ win7








