#### 1. JEP 102: Process API Updates
##### 概要

改进用于控制和管理操作系统进程的API。

##### 动机

当前API的局限性通常迫使开发人员使用本机代码。

##### 描述

Java SE为本机操作系统进程提供了有限的支持。它提供了一个基本的API来设置环境并启动一个进程。自Java SE 7以来，进程流可以重定向到文件，管道或可以被继承。一旦开始，API可以用于销毁进程和/或等待进程终止。

增强了java.lang.Process类，以提供进程的操作特定进程ID，有关进程的信息，包括参数，命令，进程的开始时间，进程的累积CPU时间和用户名的过程。

java.lang.ProcessHandle类返回有关操作系统提供的每个进程的信息，包括进程id，参数，命令，开始时间等。ProcessHandle可以返回进程的父进程，直接子进程以及所有后代，一系列ProcessHandles。

ProcessHandles可用于销毁进程并监视进程活动。使用ProcessHandle.onExit，可以使用CompletableFuture的异步机制来调度在进程退出时要执行的操作。

访问关于进程和进程控制的信息受安全管理员权限的限制，并受到正常操作系统访问控制的限制。

##### 测试

引入的类或方法将需要与实施一起开发的新单元测试。更多功能测试也将是有用的。

##### 风险与假设

这个API的主要风险是操作系统，特别是Windows之间的差异。

该API的设计需要适应在具有不同操作系统型号的较小设备上的可能部署。它还应该考虑在同一操作系统进程中运行多个Java虚拟机的环境。这些考虑可能导致更抽象的API和/或增加设计工作。

#### JEP 110: HTTP/2 Client (Incubator)
##### 概要

定义一个新的实现HTTP / 2和WebSocket的HTTP客户端API，并可以替代传统的HttpURLConnection API。该API将作为JEP 11定义的孵化器模块与JDK 9一起提供。这意味着：

API和实现不会是Java SE的一部分。

API将生活在jdk.incubtor命名空间下。

该模块在编译或运行时默认情况下不会解决。

##### 动机

现有的HttpURLConnection API及其实现有许多问题：

基本URLConnection API的设计考虑到多个协议，几乎所有这些协议都已经停止了（ftp，gopher等）。

API早于HTTP / 1.1，并且过于抽象。

很难使用，有许多无证件的行为。

它仅在阻塞模式下工作（即每个请求/响应一个线程）。

很难维护。

##### 目标

一般情况下易于使用，包括简单的阻止模式。

必须提供“收到的标题”，错误和“收到的响应体”等事件的通知。此通知不一定基于回调，但可以使用异步机制，如CompletableFuture。

一个简单而简洁的API，可满足80-90％的应用需求。这可能意味着相对较小的API占用空间不一定会暴露协议的所有功能。

必须将HTTP协议请求的所有相关方面暴露给服务器，以及来自服务器的响应（标题，正文，状态代码等）。

必须支持标准和通用的认证机制。这最初将仅限于基本身份验证。

必须能够轻松设置WebSocket握手。

必须支持HTTP / 2。 （HTTP / 2的应用级语义大体上与1.1相同，尽管线路协议完全不同）

必须能够将升级协议从1.1升级到2（或不），或者从一开始就选择2。

必须支持服务器推送，即服务器将资源推送到客户端而不需要客户端明确请求的能力。

必须执行与现有网络API一致的安全检查。

应该对新的语言功能（如lambda表达式）很友好。

应该对嵌入式系统要求很友好，特别是避免永久运行定时器线程。

必须支持HTTPS / TLS。

HTTP / 1.1的性能要求：

性能必须与现有的HttpURLConnection实现一致。

性能必须与Apache HttpClient库一致，Netty和Jetty用作客户端API时。

当用作客户端API时，新API的内存消耗必须等于或低于HttpURLConnection，Apache HttpClient，Netty和Jetty。

HTTP / 2的性能要求：

尽管有任何平台限制（例如，TCP片段确认窗口），性能必须以新协议所期望的方式（即，在可扩展性和延迟）中优于HTTP / 1.1。

当用作HTTP / 2的客户端API时，性能必须与Netty和Jetty相当。

当使用HttpURLConnection，Apache HttpClient，Netty和Jetty作为客户端API时，新API的内存消耗必须等于或低于或等于。

性能比较只能在可比较的操作模式的背景下进行，因为新API将强调简单易用，覆盖所有可能的用例，

此工作适用于JDK 9. Java EE中的某些代码可能会在Servlet 4.0 API中实现HTTP / 2时重新使用，因此只有JDK 8语言功能以及可能的API将被使用。

旨在通过在JDK 9中使用API​​的经验，可以在JDK 10中的java.net命名空间下的Java SE中对API进行标准化。当发生这种情况时，作为未来JEP的一部分，API将不再作为孵化器模块存在。

##### 非目标

该API旨在最终替代HttpURLConnection API以获取新代码，但是我们并不打算立即使用新的API重新实现旧的API。这可能会在未来的工作中发生。

在JDK 8的这个JEP的早期版本中考虑了一些要求，但是为了使API尽可能简单，它们被忽略：

请求/响应过滤，
可插拔连接缓存，和
一般升级机制。
这些要求中的一些，例如连接缓存，随着逐渐采用HTTP / 2而变得不那么重要。

##### 描述

针对JDK 9进行了一些原型开发工作，其中为HTTP客户端，请求和响应定义了单独的类。使用构建器模式将可变实体与不可变产品分开。定义同步阻塞模式用于发送和接收，并且还定义了一个基于java.util.concurrent.CompletableFuture构建的异步模式。

该原型是基于NIO SocketChannels构建的，它与Selectors和外部提供的ExecutorServices一起实现异步行为。

原型实现是独立的，即

#### JEP 143: Improve Contended Locking
##### 概要

提高竞争Java对象监视器的性能。

##### 目标

通过以下基准测试来衡量竞争对手的Java对象监视器的整体性能：

CallTimerGrid（虽然压力测试比基准测试更多）
达卡波巴赫（dacapo2009）
_ avrora
_蜡染
_ fop
_ h2
_ luindex
_ lusearch
_ pmd
_ sunflow
_ tomcat
_ tradebeans
_ tradesoap
_ xalan
DerbyContentionModelCounted
HighContentionSimulator
LockLoops-JSR166-Doug-Sept2009（LockLoops）
的PointBase
SPECjbb2013-critical（was specjbb2005）
SPECjbb2013-MAX
specjvm2008
volano29（volano2509）
##### 非目标

解决内部虚拟机监视器或互斥体的任何性能改进不是该项目的目标; Java监视器和内部VM监视器/互斥体由不同的代码实现。虽然此项目中的一些概念可能适用于内部VM监视器/互斥体，但代码不是直接适用的。

该项目的目标不是在每个基准测试或测试中提高竞争对手的Java监视器性能;在某些情况下，特定基准测试或测试可能会导致性能下降。为了在另一个基准测试或测试中获得性能改进，性能下降可能被认为是可接受的。

##### 成功指标

如果按照上述基准衡量出现可证明的业绩增长，而不抵消重大的业绩回归，该项目将被视为成功。

对于不受控制的锁，绝对不能有一个不平凡的性能回归。

##### 动机

除了Volano和DaCapo等行业基准之外，改进的竞争锁定将显着有益于真实世界的应用。

##### 描述

该项目将在以下与竞争对手的Java监视器相关的领域中探讨性能改进：

字段重排序和高速缓存行对齐
加速PlatformEvent :: unpark（）
快速Java监视器进入操作
快速Java监视器退出操作
快速Java监视器通知/ notifyAll操作
原始的工作还包括“更快的哈希码”的更改;由于Java对象的哈希码支持与竞争的Java监视器没有直接关系，因此该项目不会包含该工作。

该项目还将为在工作过程中发现的各种错误产生修复;这些错误修复将独立于性能改进工作进行管理，以便更快地集成修补程序。

这个项目被管理简单的以下“伞”错误所覆盖：

JDK-6607129减少L2 $一致性错误流量在竞争的锁定旋转循环中，特别是对于ctn系列的derby

但是，当完成子任务或错误修复时，将使用单独的错误ID来集成工作。这允许通过一个错误ID（JDK-6607129）引用整个项目，同时允许比等待整个项目完成更快的增量改进。

##### 测试

##### 功能测试

Java监视器专用的功能测试似乎并不是特定的，也没有必要。即使是最简单的Java程序，Java监视器也被广泛使用，Java监视器中的几乎任何功能破坏都应该是显而易见的。

##### 压力测试

需要为Java监视器提供一组众所周知的压力测试。这些可以针对具体的Java监视器场景进行目标压力测试，或者通常已知具有特定压力诱导选项运行的Java监视器的重度用户的测试。

注意：使用'-XX：-UseBiasedLocking -XX：+ UseHeavyMonitors'来绕过基于锁定和基于堆栈的锁定;强制使用ObjectMonitor对象。

##### 场重新排序和缓存线对齐子任务压力测试

压力测试应集中于生成大量活动的ObjectMonitor对象。压力测试的目标是ObjectMonitor的使用峰值，ObjectMonitor块分配算法和ObjectMonitor免费列表管理代码。以下是目标：

为了具有相同或更好的峰值ObjectMonitor用于小到中等配置，
没有内存泄漏，和
没有数据结构管理失败。
##### 加速PlatformEvent :: unpark（）子任务压力测试

压力测试应该集中在大量的并发服务器和/或并发进出口线程上。进入等待退出和进出口线程的组合应该是可配置的。压力测试的目标是后继机制。

目标：不会因为失去操作而失去意义。

##### 快速Java监视器进入操作子任务压力测试

压力测试应侧重于具有可伸缩数量的并行线程的进出口操作的正确性。压力测试的目标是Java监控所有权。

目标：不止一个线程认为拥有Java监视器的所有权冲突。

##### 快速Java监视器退出操作子任务压力测试

应该通过“加速PlatformEvent :: unpark（）”和“快速Java监视器进入操作”子任务的压力测试来覆盖。

##### 快速Java监视器通知/ Notify所有操作子任务压力测试

压力测试应关注具有可伸缩数量的并行线程的进入等待退出操作的正确性。压力测试的目标是等待（）完成后的Java监视器所有权，并重新输入Java监视器。

目标：不止一个线程认为拥有Java监视器的所有权冲突。

#### JEP 158: Unified JVM Logging
##### 概要

为JVM的所有组件引入通用日志记录系统。

##### 目标

所有日志记录的常用命令行选项
日志消息使用标签进行分类（例如，编译器，gc，classload，metaspace，svc，jfr，...）。一个消息可以有多个标签（标签集）
记录在不同的级别执行：错误，警告，信息，调试，跟踪，开发。
可以选择基于级别记录的消息。
可能将日志记录重定向到控制台或文件。
默认配置是将所有使用警告和错误级别的消息输出到stderr。
根据要保留的文件大小和数量文件轮换日志文件（类似于今天可用的GC日志）
一行打印（在同一行内不进行交错）
日志消息是人类可读的纯文本
消息可以“装饰”。默认装饰有：正常运行时间，级别，标签。
能够配置要打印的装饰。
现有的'tty-> print ...'日志应该使用统一的日志记录作为输出
可以通过jcmd或MBean在运行时动态配置日志记录
测试和支持 - 如果/当用户/客户启用时，不应该崩溃
伸展目标：

多行日志记录：输出时可以记录几行以保持它们在一起（非交错）的方式
启用/禁用单个日志消息（例如通过使用\__FILE\__ / \__LINE\__）
实现syslog和Windows事件查看器输出
能够配置打印装饰的顺序
##### 非目标

在此JEP的范围之外，添加来自所有JVM组件的实际日志记录调用。这个JEP只会提供进行日志记录的基础设施。

除了装饰的格式和人类可读的纯文本的使用之外，它还超出了JEP的范围，以执行记录格式。

此JEP不会将日志记录添加到JDK中的Java代码。

##### 动机

JVM是复杂的系统级组件，根本原因分析通常是一项艰巨而耗时的任务。没有广泛的可用性功能，通常几乎不可能在生产环境中找到间歇性崩溃或性能怪癖的根本原因。可以使用支持和维护工程的细粒度易于配置的JVM记录是一个这样的功能。

JRockit有一个相似的功能，它有助于为客户提供支持。

##### 描述

##### 标签

日志框架定义了JVM中的一组标签。每个标签都以其名称（例如：gc，编译器，线程等）标识。可以根据需要在源代码中更改标签集。当添加日志消息时，它应该与对记录的信息进行分类的标签集相关联。标签集由一个或多个标签组成。

##### 水平

每个日志消息都具有与其关联的日志记录级别。可用的级别是错误，警告，信息，调试，跟踪和开发以增加的冗长顺序。开发级仅适用于非产品构建。

对于每个输出，可以配置记录级别以控制写入该输出的信息量。替代关闭完全禁用日志记录。

##### 饰

使用有关该消息的信息装饰记录消息。以下是可能的装饰清单：

时间 - ISO-8601格式的当前时间和日期
正常运行时间 - 从JVM开始的时间（秒和毫秒）（例如6.567s）
timemillis - 与System.currentTimeMillis（）生成的值相同
risingimemillis - 从JVM开始以来的毫秒数
timenanos - 与System.nanoTime（）生成的值相同
上升线 - JVM开始以来的纳秒
pid - 进程标识符
tid - 线程标识符
level - 与日志消息相关联的级别
标签 - 与日志消息相关联的标签集
每个输出都可以配置为使用一组自定义的装饰器。他们的顺序总是在上面。使用的装饰可以由用户在运行时配置。装饰品将被添加到日志消息中

示例：\[6.567s\] \[info\] \[gc，old\]旧集合完成
##### 输出

目前有三种类型的输出支持：

stdout - 输出到stdout。

stderr - 输出到stderr。

文本文件 - 输出到文本文件。

可以配置为根据书写大小和要旋转的文件数量来处理文件旋转。示例：旋转日志文件每10MB，保留5个文件旋转。这些文件名将在旋转中附加它们的数字。示例：hotspot.log.1，hotspot.log.2，...，hotspot.log.5当前打开的文件不会附加任何数字。示例：hotspot.log。不能保证文件的大小正好与配置的大小一致。大小最多可以溢出最后写入的日志消息的大小。

某些输出类型可能需要额外的配置。附加的输出类型可以使用一个简单和定义良好的接口来实现。

##### 命令行选项

将添加一个新的命令行选项，以控制来自JVM的所有组件的日志记录。

-Xlog
多个参数将按照命令行中显示的顺序应用。相同输出的多个'-xlog'参数将以给定的顺序相互覆盖。最后的配置规则。

将使用以下语法配置日志记录：   

```
-Xlog[:option]
    option         :=  [<what>][:[<output>][:[<decorators>][:<output-options>]]]
                       'help'
                       'disable'
    what           :=  <selector>[,...]
    selector       :=  <tag-set>[*][=<level>]
    tag-set        :=  <tag>[+...]
                       'all'
    tag            :=  name of tag
    level          :=  trace
                       debug
                       info
                       warning
                       error
    output         :=  'stderr'
                       'stdout'
                       [file=]<filename>
    decorators     :=  <decorator>[,...]
                       'none'
    decorator      :=  time
                       uptime
                       timemillis
                       uptimemillis
                       timenanos
                       uptimenanos
                       pid
                       tid
                       level
                       tags
    output-options :=  <output_option>[,...]
    output-option  :=  filecount=<file count>
                       filesize=<file size in kb>
                       parameter=value
```
                
“all”标签是由所有可用标签集组成的元标记。 'tag-set'定义中的'*'表示“通配符”标签匹配。 不使用'*'表示“所有与所指定的标签匹配的信息”。

省略“what”将默认标记所有和级别的信息。

省略“level”将默认为信息

省略'output'将默认为stdout

省略“decorators”将默认为正常运行时间，级别，标签

“none”装饰器是特殊的，用于关闭所有装饰。

提供的级别是汇总的。 例如，如果输出配置为使用'level'信息。 将输出所有与“什么”中的标签与日志级别信息，警告和错误相匹配的消息。   

```
-Xlog:disable
    this turns off all logging and clears all configuration of the
    logging framework. Even warnings and errors.
-Xlog:help
    prints -Xlog usage syntax and available tags, levels, decorators
    along with some example command lines.
```
##### 默认配置：
```
 -Xlog:all=warning:stderr:uptime,level,tags
    - default configuration if nothing is configured on command line
    - 'all' is a special tag name aliasing all existing tags
    - this configuration will log all messages with a level that
    matches ´warning´ or ´error´ regardless of what tags the
    message is associated with
```
##### 简单的例子：
```
-Xlog   
```
等价于

```
-Xlog:all
    - log messages using 'info' level to stdout
    - level 'info' and output 'stdout' are default if nothing else
    is provided
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:gc
    - log messages tagged with 'gc' tag using 'info' level to
    'stdout'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:gc=debug:file=gc.txt:none
    - log messages tagged with 'gc' tag using 'debug' level to
    a file called 'gc.txt' with no decorations
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:gc=trace:file=gctrace.txt:uptimemillis,pids:filecount=5,filesize=1024
    - log messages tagged with 'gc' tag using 'trace' level to
    a rotating fileset with 5 files with size 1MB with base name
    'gctrace.txt' and use decorations 'uptimemillis' and 'pid'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:gc::uptime,tid
    - log messages tagged with 'gc' tag using default 'info' level to
    default output 'stdout' and use decorations 'uptime' and 'tid'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:gc*=info,rt*=off
    - log messages tagged with at least 'gc' using 'info' level but turn
    off logging of messages tagged with 'rt'
    - messages tagged with both 'gc' and 'rt' will not be logged
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:disable -Xlog:rt=trace:rttrace.txt
    - turn off 'all' logging, even warnings and errors, except
    messages tagged with 'rt' using 'trace' level
    - output to a file called 'rttrace.txt'
```
##### 复杂的例子：
```
 -Xlog:gc+rt*=debug
    - log messages tagged with at least 'gc' and 'rt' tag using 'debug'
    level to 'stdout'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:gc+meta*=trace,rt*=off:file=gcmetatrace.txt
    - log messages tagged with at least 'gc' and 'meta' tag using 'trace'
    level to file 'metatrace.txt' but turn off all messages tagged
    with 'rt'
    - again, messages tagged with 'gc', 'meta' and 'rt' will not be logged
    since 'rt' is set to off
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:gc+meta=trace
    - log messages tagged with exactly 'gc' and 'meta' tag using 'trace'
    level to 'stdout'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

 -Xlog:gc+rt+compiler*=debug,meta*=warning,svc*=off
    - log messages tagged with at least 'gc' and 'rt' and 'compiler' tag
    using 'trace' level to 'stdout' but only log messages tagged
    with 'meta' with level 'warning' or 'error' and turn off all
    messages tagged with 'svc'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect
```
##### 在运行时控制
可以通过诊断命令（jcmd实用程序）在运行时控制日志记录。可以使用诊断命令动态地指定在命令行中指定的所有内容。由于诊断命令会自动显示为MBean，因此可以使用JMX在运行时更改日志记录配置。

对日志记录配置和参数进行枚举的辅助支持将被添加到运行时控制命令列表中。

##### JVM界面

在JVM中，将使用类似于以下API的API创建一组宏：

log_<level>(Tag1[,...])(fmtstr, ...)
    syntax for the log macro
##### 例：

```
log\_info(gc, rt, classloading)("Loaded %d objects.", object_count)
    the macro is checking the log level to avoid uneccessary
    calls and allocations.

log_debug(svc, debugger)("Debugger interface listening at port %d.", port_number)
```
##### 等级信息：

```
LogHandle（gc，meta，classunloading）log;
if（log.is_trace（））{
    ...
}
if（log.is_debug（））{
    ...
}
```

为避免执行生成仅用于记录的数据的代码，可以向Log类询问当前配置的日志级别。

##### 性能

不同的日志级别应具有定义级别的预期性能开销的准则。例如：“警告级别不应影响性能;信息级别对于生产应该是可以接受的;调试，跟踪和错误级别不具备性能要求。运行日志禁用应该尽可能少的性能影响。通常会记录成本。

##### 未来可能的扩展

将来，添加一个用于将日志消息写入此基础架构的Java API可能是有用的，供JDK中的类使用。

最初只有三个后端将被开发：stdout，stderr和file。未来的项目可能会增加其他后端。例如：syslog，Windows事件查看器，套接字等

##### 开放式问题

我们应该在API中提供一个替代方案来提供作为宏的参数的级别吗？
如果装饰物被[]以外的其他东西包围，以便更容易解析输出？
datestamp装饰的确切格式是什么？提出ISO 8601。
##### 测试

非常重要的是，登录本身不会引起任何不稳定，因此需要进行广泛的测试。

功能测试必须通过启用某些日志消息并检查其在stderr或文件上的存在来完成。

因为可以动态启用日志记录，所以我们需要通过在运行应用程序时持续启用和禁用日志记录来强调测试。

日志框架将使用单元测试进行测试。

##### 风险与假设

上述设计可能无法满足当今日志记录的所有用途。如果是这种情况，则必须重新设计。

##### 碰撞

兼容性：日志消息格式将发生变化，也可能会改变某些JVM选项的含义。
安全性：需要验证文件权限。
性能/可扩展性：如果启用了大量日志记录，性能将受到影响。
用户体验：命令行选项将更改。记录输出将改变。
I18n / L10n：日志消息不会被本地化或国际化。
文档：新选项及其用法必须记录在案。

####JEP 165: Compiler Control

##### 概要

该JEP提出了一种改进的方法来控制JVM编译器。它支持运行时可管理，依赖于方法的编译器标志。 （在编译期间不变）

##### 目标

JVM编译器（C1和C2）的细粒度和方法上下文相关控制

能够在运行时更改JVM编译器控制选项

没有性能下降

##### 动机

编译过程的方法上下文相关控制是编写可以运行而不重新启动整个JVM的小型JVM编译器测试的强大工具。为JVM编译器中的错误创建变通方法也非常有用。编译器选项的良好封装也是很好的卫生。

##### 描述

##### 指令

控制JVM编译器的所有选项将被收集到一组选项中。一组具有值的选项称为编译器指令，是一个如何编译的指令。一个指令与一个方法匹配器一起提供给VM，该方法匹配器决定了它适用于哪些方法。同时可以在运行时激活几个指令，但是只有一个指令应用于特定的编译。在运行时可以添加和删除指令。

##### 指令格式

指令文件具有指定的标准化和可读的文件格式。指令文件可以通过命令行和诊断命令加载。指令文件具有一个或多个定义的指令。指令包含一个方法模式和一些具有值的选项。指令的顺序是重要的。 compileBroker将将其模式匹配的第一个指令应用于编译。

指令文件格式将是JSON的一个子集，并附加一些内容。该格式以以下方式偏离JSON：

只支持与命令行选项兼容的数字 - int和doubles。
允许注释 - 以“//”开头的行
在数组和对象中允许使用额外的尾随“'”
可能不允许转义的字符（TBD）
选项名称是字符串，但可选地引用
该文件可以使用JVM规范支持的所有UTF-8字符。这些字符为文件格式保留：

{ - curly brace open
 } - curly brace end
 [ - square brace open
 ] - square brace end
 " - quoutes
 : - colon
 , - comma
##### 指令示例1

```
[//如果指令数组，则启动
    {//开始指令块
        //一次匹配一个或几个模式
        //单个模式不需要数组
        match：[“java *。*”，“oracle *。*”]，
        //仅适用于单个编译器的指令块
        c1：{
             //一个bool选项。额外的逗号不应该导致解析错误
             PrintAssembly：真实，
        }，
        //另一个编译器块
        c2：{
             //强制在线模式加上+，防止与 - 
             inline：[“+ vm *。*”，“ - *。*”]
        }，
        //编译器外的选项适用于所有编译器
        BreakAtExecute：true //在编译代码中启用中断
        BreakAtCompile：true //在编译器中启用中断
    }，
    {//开始另一个指令块
        //匹配ant方法，其类以“并发”结尾
        匹配：[“*并发*”]，
        c2：{
             //禁用编译
             排除方法：真正的，
        }
        //与c1指令未指定的选项保持默认。
    }
]
```
##### 指令示例2
```
[
   {
         //模式匹配类+方法+签名
         //允许前导和尾随通配符（*）
         比赛：“apa / Dingo。*”，
 
         //覆盖指定编译器的默认值
         //最内部的选项是最高的
         c1：{
           //覆盖c1预设
           PrintInlining：false //示例 - 此选项可能不存在
         }
         c2：{
           //控制方法的内联
           // +强制内联， - 不要内联
           inline：[“+ java / util。*”，“-com / sun。*”]，
         }
         //特定预设之外的指令适用于所有编译器
         inline：[“+ java / util。*”，“-com / sun。*”]，
         PrintAssembly：true
   }，
   {
         //匹配几个模式需要一个数组
         比赛：[“steve。*”，“alex。*”]
         c2：{
              启用：false，//忽略c2的此指令。
              BreakAtExecute：true //由于Enable为false，因此不会应用
              
         }
         //适用于所有编译器
         // +强制内联， - 不要内联
         inline：[“+ java / util。*”，“-com / sun。*”]，
         PrintInline：true
   }，
]
```
##### 指令选项列表

第一个实现包含以下选项。所有选项以前已在CompileCommand选项命令中使用。将添加更多选项。

公共标志： Enable, bool Exclude, bool BreakAtExecute, bool BreakAtCompile, bool Log, bool PrintAssembly, bool PrintInlining, bool PrintNMethods, bool ReplayInline, bool DumpReplay, bool DumpInline, bool CompilerDirectivesIgnoreCompileCommands, bool Inline, ccstr[]

C2 only：BlockLayoutByFrequency，bool PrintOptoAssembly，bool PrintIntrinsics，bool raceOptoPipelining，bool TraceOptoOutput，bool TraceSpilling，bool Vectorize，bool VectorizeDebug，bool CloneMapDebug，bool IGVPrintLevel，intx MaxNodeLimit，intx DisableIntrinsics，ccstr

```
inline：<one pattern or a array of string patterns>
```

该模式是与方法名称匹配的字符串的方式与指令匹配相同。
模式中的'+'表示方法匹配应强制内联。
应该防止内联的' - '。
使用匹配的第一个模式的命令。
示例1：inline：[“+ java / lang /*.*”， - “sun *。*”]
示例2：inline：“+ java / lang /*.*”

##### 指令模式

在“match”和“inline”-option中使用的方法模式具有以下模式：Class.method（signature）

类包括由/ Class分隔的包名称，方法可以带前导和尾随\*的通配符，或者代替\*如果签名被省略，则默认为\*

这些是有效的模式：: "java/lang/String,indexOf" "/lang/String,indexOf(I)" "java/lang/String.(I)" "java/lang/String.()" "." "java/lang/."

##### 指令解析器

指令解析器负责解析指令文件，并将信息添加到VM内部格式。

如果在命令行上指定了格式错误的伪指令文件，VM将打印错误并退出。如果通过诊断命令添加了一个格式不正确的伪指令文件，它将被忽略，并且将打印一个正确的警告。

解析器将验证所有选项都有效。依赖平台的选项将在不支持平台的平台上打印警告。理由是相同的指令文件应该可以使用，无论其部署在哪个平台上。

未指定的选项将使用默认值。如果指定了一个默认值的命令行选项。方法模式的默认值为“。” （匹配所有方法）。

##### 编译经纪人

compilerBroker具有包含所有应用指令的指令栈。 bottom指令是默认设置，永远不能被删除。当文件加载附加指令时，将以相反的顺序添加文件，文件中的第一个指令将以堆栈的顶部结尾。这是一个可用性功能。

当为编译提交一个方法时，compileBroker将选择匹配的第一个指令并将其传递给编译器。 compilerBroker和编译器将忽略将会创建坏代码的选项（强制在不支持它的平台上强制硬件指令），并且将发出适当的警告。指令选项与正常的命令行标志具有相同的限制 - 例如强制内联只会在IR不会变大的情况下被尊重。

##### 命令行界面

一个指令文件可以添加一个命令行。如果标志错误（正常命令行解析），文件丢失或文件内容格式错误，则VM将退出并显示错误消息。

```
-XX：CompilerDirectivesFile = <文件>
```
##### 诊断命令界面

这些是与编译器控件一起使用的诊断命令：

```
jcmd <pid> Compiler.add_directives <file>
```
从文件添加其他指令。 新的指令将被添加在旧的指令之上，文件中的第一个指令最终位于指令栈的顶部。

```
jcmd <pid> Compiler.list_directives
```
从上到下列出指令堆栈上的所有指令。

```
jcmd <pid> Compiler.clear_directives
```
清除指令栈

```
jcmd <pid> Compiler.remove_directives
```
从指令栈中删除顶部元素

##### 编译命令和向后兼容性

CompilerControl应该在所有用例中替换CompileCommand。 CompileCommand将保持向后兼容性，目标是尽可能保持行为。

有四层可以应用的控制。编译器控制将具有最高优先级，并覆盖任何其他标志或命令。二是CompileCommand，第三是任何命令行标志，第四是默认标志值。如果同时使用Compiler控件和CompileCommand，那么Compiler控件就会认为CompileCommand会覆盖默认值。

如果使用CompileCommand和编译器指令，JVM应该打印一个警告。

##### 方法模式

编译器控件将使用与CompileCommand相同的方法模式格式。该模式由包和类名称，方法名称和签名三部分组成。这三个中的任何一个可能是带有前导或尾随*的通配符。任何部分的默认值为*。

例：

java/example/Test.split
由三部分组成

jjava/example/Test + split + (Ljava/lang/String;)Ljava/lang/String;
##### 风险与假设

编译器选项的绝对数量将限制我们最初专注于一个子集。我们将专注于一个子集，并从那里扩展。

##### 依赖性

诊断命令 - 已经到位
使用完整的JDK - 已经到位
##### 碰撞

文档：标志和API
CCC：指令格式，JVM编译器标志更改和API将需要CCC请求。
性能：标准回归测试

#### JEP 193: Variable Handles
##### 概要

定义一种标准方法来调用对象字段和数组元素上的各种java.util.concurrent.atomic和sun.misc.Unsafe操作的等效项，对于对存储器排序进行细粒度控制的标准集合围栏操作以及标准可达性-fence操作，以确保引用的对象保持强大可访问。

##### 目标

以下是必需的目标：

安全。不可能将Java虚拟机置于损坏的内存状态。例如，对象的一个​​字段只能使用可转换为字段类型的实例进行更新，或者如果数组索引在数组边界内，数组元素只能在数组中访问。

诚信。访问对象的字段除了与对象的最终字段无法更新的约束之外，还遵循与getfield和putfield字节代码相同的访问规则。 （注意：这种安全和完整性规则也适用于给一个字段进行读取或写入访问的MethodHandles。）

性能。性能特征必须与等效的sun.misc.Unsafe操作相同或类似。（具体来说，生成的汇编代码应该几乎相同，模数一些不能被折叠的安全检查）。

可用性。 API必须比sun.misc.Unsafe API更好。

这是可取的，但不是必需的，API与java.util.concurrent.atomic API一样好。

##### 动机

由于Java中的并行和并行编程不断扩展，程序员越来越受挫，因为无法使用Java构造在单个类的字段上安排原子或有序操作;例如，原子地增加计数字段。到目前为止，实现这些效果的唯一方法是使用独立的AtomicInteger（添加空间开销和额外的并发问题来管理间接），或者在某些情况下，使用原子FieldUpdaters（通常会遇到比操作本身更多的开销） ，或者使用不安全（不可移植和不受支持的）sun.misc.Unsafe API用于JVM内在函数。 Intrinsics更快，所以它们已经被广泛使用，损害了安全性和便携性。

没有这个JEP，这些问题预计会随着原子API扩展以覆盖附加访问一致性策略（与最近的C ++ 11内存模型一致）作为Java内存模型修订的一部分而变得更糟。

##### 描述

可变句柄是对变量的类型引用，它支持在各种访问模式下对变量进行读写访问。支持的变量种类包括实例字段，静态字段和数组元素。正在考虑其他可变类型，并且可以被支持，例如数组视图，将字节或字符数组视为长数组，以及由ByteBuffers描述的非堆栈区域中的位置。

变量句柄需要库增强，JVM增强和编译器支持。此外，它需要对Java语言规范和Java虚拟机规范的微小更新。还考虑了增强编译时类型检查和补充现有语法的次要语言增强功能。

如果将这些规范添加到Java中，那么生成的规范预计将以自然的方式扩展到额外的类似原始类型的值类型或其他类似数组的类型。然而，这不是用于控制多个变量的访问和更新的通用事务机制。可以在本JEP的过程中探讨表达和实施这些结构的替代形式，并可能是进一步JEP的主题。

变量句柄由单个抽象类java.lang.invoke.VarHandle建模，其中每个变量访问模式由签名多态方法表示。

访问模式集代表最小的可行集合，并且被设计为与C / C ++ 11原子兼容，而不依赖于对Java内存模型的修订更新。如果需要，将添加其他访问模式。某些访问模式可能不适用于某些变量类型，如果是这样，在关联的VarHandle实例上调用时会抛出UnsupportedOperationException异常。

访问模式分为以下几类：

读取访问模式，例如读取具有易失性存储器排序效果的变量;

写访问模式，例如用释放存储器排序效果更新变量;

原子更新访问模式，例如对具有易读存储器顺序影响的变量的比较和设置，用于读取和写入;

数字原子更新访问模式，例如获取和添加具有写入的简单存储器顺序效果并获取用于读取的存储器顺序效应。

逐位原子更新访问模式，如获取和按位，以及释放存储器顺序效应，用于写入和简单的存储器顺序效果用于读取。

后面的三个类别通常被称为读 - 修改 - 写模式。

访问模式方法的签名 - 多态特征使可变句柄能够使用一个抽象类来支持许多变量类型和变量类型。这避免了可变类型和类型特定类的爆炸。此外，即使访问模式方法签名被声明为Object的可变参数数组，因此这种签名 - 多态特征确保不会包含原始值参数，也不会将参数打包到数组中。这在HotSpot解释器和C1 / C2编译器的运行时可实现可预测的行为和性能。

创建VarHandle实例的方法位于与生成访问等效或类似变量类型的MethodHandle实例相同的区域。

创建用于实例和静态字段变量种类的VarHandle实例的方法位于java.lang.invoke.MethodHandles.Lookup中，并由查找关联的接收类中的字段的进程创建。例如，在接收器类Foo上为类型int命名为i的字段获得VarHandle的这种查找可以如下执行：

```
class Foo {
    int i;
    ...
}
...
class Bar {
    static final VarHandle VH_FOO_FIELD_I;
    static {
        try {
            VH_FOO_FIELD_I = MethodHandles.lookup().
                in(Foo.class).
                findVarHandle(Foo.class, "i", int.class);
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```
在生成并返回VarHandle之前，对VarHandle的查询将在生成并返回VarHandle之前执行完全相同的访问控制检查（代表查找类），这些检查通过查找可以读取和写入访问权限的MethodHandle来执行相同的字段（参见MethodHandles.Lookup类中的find {，Static} {Getter，Setter}方法）。

访问模式方法将在以下条件下调用时抛出UnsupportedOperationException：

将VarHandle的访问模式方法写入最终字段。

基于数字的访问模式方法（getAndAdd和addAndGet）用于引用变量类型或非数字类型（如布尔值）。

引用变量类型或浮点型和双类型的基于位的访问模式方法（后面的限制可能会在将来的修订版中删除）

对于相关的VarHandle，一个字段不需要标记为volatile，以执行易失性访问。实际上，挥发性改性剂（如果存在）被忽略。这与java.util.concurrent.atomic.Atomic {Int，Long，Reference} FieldUpdater的行为不同，其中相应的字段必须标记为volatile。在某些情况下，这可能是太限制性的，在某些情况下，已知某些易失性访问并不总是需要的。

为基于数组的变量类型创建VarHandle实例的方法位于java.lang.invoke.MethodHandles中（请参阅MethodHandles类中的arrayElement {Getter，Setter}方法）。例如，可以创建一个VarHandle到int数组，如下所示：

```
VarHandle intArrayHandle = MethodHandles.arrayElementVarHandle(int[].class);
```

访问模式方法将在以下条件下调用时抛出UnsupportedOperationException：

基于数字的访问模式方法（getAndAdd和addAndGet）用于数组组件引用变量类型或非数字类型（如布尔值）

引用变量类型或浮点型和双类型的基于位的访问模式方法（后面的限制可能会在将来的修订版中删除）

对于实例字段，静态字段和数组元素的变量类型，支持所有原始类型和引用类型。 其他可变类型可以支持这些类型的全部或一部分。

为基于数组视图的变量类型创建VarHandle实例的方法也位于java.lang.invoke.MethodHandles中。 例如，可以创建一个VarHandle来查看字节数组作为未对齐的long数组，如下所示：
VarHandle longArrayViewHandle = MethodHandles.byteArrayViewVarHandle(
        long[].class, java.nio.ByteOrder.BIG_ENDIAN);
虽然可以使用java.nio.ByteBuffer实现类似的机制，但它要求创建一个ByteBuffer实例来包装一个字节数组。由于逃脱分析的脆弱性，这并不总是保证可靠的性能，访问必须经过ByteBuffer实例。在未对齐访问的情况下，除普通访问模式之外，所有方法将抛出IllegalStateException。对于对齐访问，某些易失性操作取决于变量类型是可能的。这样的VarHandle实例可以用于向量化数组访问。

访问模式方法的参数数量，参数类型和返回类型由变量类型，变量类型和访问模式的特性决定。 VarHandle创建方法（如前所述）将记录要求。例如，在先前查找的VH_FOO_FIELD_I句柄中的compareAndSet需要3个参数，一个是接收器Foo的实例，另外两个int用于预期值和实际值：

```
Foo f = ...
boolean r = VH_FOO_FIELD_I.compareAndSet（f，0，1）;
```
相反，getAndSet需要2个参数，一个是Foo的实例，一个int是要设置的值：

```
nt o =（int）VH_FOO_FIELD_I.getAndSet（f，2）;
```
访问数组元素将需要在接收器和值参数（如果有的话）之间的类型为int的附加参数，这些参数对应于要操作的元素的数组索引。

对于运行时的可预测行为和性能，VarHandle实例应该保留在静态final字段中（根据Atomic {Int，Long，Reference} FieldUpdater的实例的要求）。这样可以确保访问模式方法调用会发生常量折叠，例如折叠方法签名检查和/或参数转换检查。

注意：未来的HotSpot增强功能可能支持VarHandle或MethodHandle的常量折叠，这些实例保存在非静态final字段，方法参数或局部变量中。

可以通过使用MethodHandles.Lookup.findVirtual为VarHandle访问模式方法生成MethodHandle。例如，要为特定变量类型的“compareAndSet”访问模式生成一个MethodHandle，并键入：

```
Foo f = ...
MethodHandle mhToVhCompareAndSet = MethodHandles.publicLookup（）。findVirtual（
        VarHandle.class，
        “compareAndSet”
        MethodType.methodType（boolean.class，Foo.class，int.class，int.class））;
```

然后可以使用变量类型调用MethodHandle，并将兼容的VarHandle实例作为第一个参数：

```
boolean r =（boolean）mhToVhCompareAndSet.invokeExact（VH_FOO_FIELD_I，f，0,1）;
```
或者mhToVhCompareAndSet可以绑定到VarHandle实例，然后调用：

```
MethodHandle mhToBoundVhCompareAndSet = mhToVhCompareAndSet
        .bindTo（VH_FOO_FIELD_I）;
boolean r =（boolean）mhToBoundVhCompareAndSet.invokeExact（f，0，1）;
```
这样使用findVirtual的MethodHandle查找将执行一个asType转换来调整参数和返回值。该行为相当于使用MethodHandles.varHandleInvoker（MethodHandles.invoker`的类比）生成的MethodHandle：

```
MethodHandle mhToVhCompareAndSet = MethodHandles.varHandleExactInvoker（
        VarHandle.AccessMode.COMPARE_AND_SET，
        MethodType.methodType（boolean.class，Foo.class，int.class，int.class））;

boolean r =（boolean）mhToVhCompareAndSet.invokeExact（VH_FOO_FIELD_I，f，0,1）;
```
因此，VarHandle可以在包装类的擦除或反射场景中使用，例如替换java.util.concurrent.Atomic * FieldUpdater / Atomic * Array类中的Unsafe用法。 （尽管需要进一步的工作，以便更新程序被授予对声明类中查找字段的访问权限。）

访问模式方法调用的源编译将遵循与对于MethodHandle.invokeExact和MethodHandle.invoke的签名 - 多态方法调用相同的规则。 Java语言规范将需要以下补充：

参考VarHandle类中的签名 - 多态访问模式方法。
允许签名多态方法返回Object以外的类型，表示返回类型不是多态的（否则将通过调用站点上的转换声明）。这使得它更容易地调用基于写入的访问方法返回void并调用返回布尔值的compareAndSet。
但是，希望而不是要求增强签名 - 多态方法调用的源编译以执行多态返回类型的目标类型，以便不需要显式转换。

注意：使用方法引用语法（如VarHandle VH_FOO_FIELD_I = Foo :: i）查找MethodHandle或VarHandle的语法和运行时支持是可取的，但不在本JEP的范围内。
访问模式方法调用的运行时调用将遵循类似于对MethodHandle.invokeExact和MethodHandle.invoke的签名 - 多态方法调用的规则。 Java虚拟机规范将需要以下补充：

参考VarHandle类中的签名 - 多态访问模式方法。
指定invokevirtual字节代码行为的调用访问模式签名多态方法。可以通过定义从访问模式方法调用到MethodHandle的转换来指定这种行为，然后使用具有相同参数的invokeExact来调用（参见之前使用的MethodHandles.Lookup.findVirtual）。
重要的是，支持的变量种类，类型和访问模式的VarHandle实现是可靠的高效且符合性能目标。利用签名多态方法有助于避免拳击和阵列打包。实施将：

在java.lang.invoke包中，其中HotSpot将该包中的类的最终字段视为真正的final，当VarHandle本身在静态final字段中引用时，可以实现常量折叠;

利用JDK内部注释@Stable为不断折叠仅更改一次的值，而@ForceInline确保即使达到正常的内联阈值，方法也会内联;和

使用sun.misc.Unsafe进行底层增强的易失性访问。

一些HotSpot内在函数是必需的，其中一些列举如下：

已经添加了Class.cast的内在函数（见JDK-8054492）。在这个内在的添加之前，一个不断折叠的Class.cast会留下可能导致不必要的优化的冗余检查。
对于并发访问变量，获取访问模式的内在函数可以与设置释放访问模式的内在同步（请参阅sun.misc.Unsafe.putOrdered {Int，Long，Object}）。

数组边界的内在函数检查JDK-8042997。静态方法可以添加执行这种检查的java.util.Arrays，并接受调用的函数以返回要抛出的异常或字符串消息，以便在检查失败时包含在异常中。这样的内在函数可以使用无符号值进行更好的比较（因为数组长度总是为正），而且在阵列元素之外的展开循环之外更好地提升范围检查。

此外，HotSpot进行的范围检查的进一步改进已经实现（JDK-8073480）或者需要（JDK-8003585来强化减少范围检查，比如说fork / join框架，或者说HashMap或ConcurrentHashMap）。

VarHandle实现应该对java.lang.invoke包中的其他类具有最小的依赖关系，以避免增加启动时间，并避免在静态初始化期间发生循环依赖。例如，ConcurrentHashMap由这样的类使用，如果ConcurrentHashMap被修改为使用VarHandles，则需要确保不引入循环依赖。使用ThreadLocalRandom及其使用AtomicInteger的其他更微妙的周期是可能的。对于包含VarHandle方法调用的方法，C2 HotSpot编译时间也是不合适的。
##### 记忆围栏
栅栏操作被定义为VarHandle类中的静态方法，并且表示对存储器排序的细粒度控制的最小可行集合。

```
   /**
    * Ensures that loads and stores before the fence will not be
    * reordered with loads and stores after the fence.
    *
    * @apiNote Ignoring the many semantic differences from C and
    * C++, this method has memory ordering effects compatible with
    * atomic_thread_fence(memory_order_seq_cst)
    */
   public static void fullFence() {}
   /**
    * Ensures that loads before the fence will not be reordered with
    * loads and stores after the fence.
    *
    * @apiNote Ignoring the many semantic differences from C and
    * C++, this method has memory ordering effects compatible with
    * atomic_thread_fence(memory_order_acquire)
    */
   public static void acquireFence() {}
   /**
    * Ensures that loads and stores before the fence will not be
    * reordered with stores after the fence.
    *
    * @apiNote Ignoring the many semantic differences from C and
    * C++, this method has memory ordering effects compatible with
    * atomic_thread_fence(memory_order_release)
    */
   public static void releaseFence() {}
   /**
    * Ensures that loads before the fence will not be reordered with
    * loads after the fence.
    */
   public static void loadLoadFence() {}
   /**
    * Ensures that stores before the fence will not be reordered with
    * stores after the fence.
    */
   public static void storeStoreFence() {}
```

一个完整的栅栏比一个比负载栅栏更强的获得围栏更坚固（在订购保证方面）。 同样，一个完整的栅栏比一个比商店门槛更强的释放栅栏更强大。

##### 可达性围栏
可达性栅栏定义为java.lang.ref.Reference上的静态方法：

```
class java.lang.ref.Reference {
   // add:
   /**
    * Ensures that the object referenced by the given reference
    * remains <em>strongly reachable</em> (as defined in the {@link
    * java.lang.ref} package documentation), regardless of any prior
    * actions of the program that might otherwise cause the object to
    * become unreachable; thus, the referenced object is not
    * reclaimable by garbage collection at least until after the
    * invocation of this method. Invocation of this method does not
    * itself initiate garbage collection or finalization.
    *
    * @param ref the reference. If null, this method has no effect.
    */
   public static void reachabilityFence(Object ref) {}
}
```

目前，提供一个注释（@Finalized说）要在一个方法上声明的注释是超出范围的，在一个方法中，编译或运行时会导致方法体被包装如下：

```
try {
    <method body>
} finally {
    Reference.reachabilityFence(this);
}
```

预计这种功能可以被编译时注释处理器支持。
##### 备择方案
引入新形式的“价值类型”被认为支持不稳定的行动。但是，这与其他类型的属性不一致，并且还需要更多的程序员的使用。还依赖于java.util.concurrent.atomic FieldUpdaters，但是它们的动态开销和使用限制使它们不合适。

在讨论这些问题的多年以来，其他几种替代方案，包括基于实地参考的替代方案，已经被提出和被驳回，因为句法，效率和/或可用性的原因是不可行的。

在此JEP的以前版本中考虑了语法增强功能，但被认为是“神奇的”，其中使用volatile关键字范围到浮动接口，一个用于引用，一个用于每个支持的原语类型。

在该JEP的以前版本中考虑了从VarHandle扩展的通用类型，但是对于通用类型和盒式类型变量的特殊处理，增加的多态性签名被认为是未成熟的，因为未来的Java版本具有价值类型和泛型， JEP 218，以及使用Arrays 2.0改进的阵列。

此JEP的以前版本中还考虑了实现特定的调用动力学方法。这要求使用和不使用invokedynamic的编译方法调用被仔细对齐，在语义方面是相同的。此外，在核心类中使用invokedynamic（例如ConcurrentHashMap）将导致循环依赖。

##### 测试

将使用jcstress线束开发压力测试。

##### 风险与假设

VarHandle的原型实现已经通过纳基准测试和叉/连接基准测试，其中fork / join库使用sun.misc.Unsafe被VarHandle替代。目前还没有观察到主要的性能问题，并且识别出的HotSpot编译器问题似乎并不繁重（折叠检查和改进阵列边界检查）。因此，我们有信心这种方法的可行性。然而，我们期望这将需要更多的实验来确保编译技术在最常需要这些构造的性能关键上下文中是可靠的。

##### 依赖性

java.util.concurrent（以及JDK中标识的其他区域）中的类将从sun.misc.Unsafe迁移到VarHandle。

此JEP不依赖于JEP 188：Java内存模型更新。

####JEP 197: Segmented Code Cache
##### 概要

将代码缓存分为不同的段，每个段都包含特定类型的编译代码，以提高性能并启用未来的扩展。

##### 目标

分离非方法，异型和非分析代码
由于跳过非方法代码的专门的迭代器，扫描时间缩短
提高一些编译密集型基准测试的执行时间
更好地控制JVM内存占用
降低高度优化代码的碎片
改善代码位置，因为相同类型的代码可能会及时访问
更好的iTLB和iCache行为
建立未来扩展的基础
改进异构代码的管理;例如，苏门答腊（GPU代码）和AOT编译代码
每个代码堆的细粒度锁定的可能性
代码和元数据的未来分离（见JDK-7072317）
##### 非目标

分段代码缓存仅为未来的扩展提供基础，例如细粒度锁定;它还没有实现任何这些改进。

##### 成功指标

分离不同的代码类型
较短的扫描时间
执行时间缩短
高度优化代码的碎片减少
减少iTLB和iCache未命中的数量
##### 动机

编译代码的组织和维护对性能有重大影响。如果代码缓存执行错误的操作，则会报告几个因素的性能回归实例。随着分层编译的引入，代码缓存的作用变得更加重要，因为与使用非分层编译相比，编译代码的数量增加了2X-4X。分层编译还引入了一个新的编译代码类型：被编码的代码（异构代码）。分析代码与非分析代码具有不同的属性;一个重要的区别是，分析代码具有预定义的有限生命期，而非分析代码可能永远保留在代码缓存中。

当前的代码缓存被优化以处理均匀的代码，即只有一种类型的编译代码。代码缓存在连续的内存块的顶部被组织为单个堆数据结构。因此，具有预定义的有限寿命的分析代码与非分析代码混合，这些代码可能永远保留在代码缓存中。这导致不同的性能和设计问题。例如，方法清除器必须在扫描时扫描整个代码高速缓存，即使某些条目从不刷新或包含非方法代码。

##### 描述

代码缓存不是单个代码堆，而是分段为不同的代码堆，每个代码堆都包含特定类型的编译代码。这样的设计使我们能够分离具有不同属性的代码。有三种不同的顶级类型的编译代码：

JVM内部（非方法）代码
异形码
非分类代码
相应的代码堆是：

包含非方法代码的非方法代码堆，例如编译器缓冲区和字节码解释器。这种代码类型将永远保留在代码缓存中。

一个分析代码堆，包含轻微优化的简档方法，寿命短。

包含完全优化的非剖析方法的非概要代码堆，其潜在的长期使用寿命。

非方法代码堆的固定大小为3 MB，用于考虑VM内部加上编译器缓冲区的其他空间。根据C1 / C2编译器线程的数量调整此额外的空间。剩余的代码缓存空间分布在异型和非异构代码堆之间。

引入以下命令行开关来控制代码堆的大小：

-XX：NonProfiledCodeHeapSize：设置包含非概要分析方法的代码堆的大小（以字节为单位）。

-XX：ProfiledCodeHeapSize：设置包含配置文件方法的代码堆的大小（以字节为单位）。

-XX：NonMethodCodeHeapSize：设置包含非方法代码的代码堆的大小（以字节为单位）。

代码缓存的接口和实现适用于支持多个代码堆。由于代码缓存是JVM的中心组件，许多其他组件都受到这些更改的影响，其中包括：

代码缓存清理器：现在只有迭代方法代码堆
分层编译策略：根据代码堆中的可用空间设置编译阈值
Java Flight Recorder（JFR）：与代码缓存相关的事件
间接引用来自：
可服务性代理：Java接口代码缓存内部
DTrace ustack帮助脚本（jhelper.d）：解析编译的Java方法的名称
Pstack支持库（libjvm_db.c）：堆栈跟踪编译的Java方法
##### 备择方案

替代实现将定义优选地分配不同代码类型的逻辑存储器区域。如果有可用空间，我们将分配到首选的内存区域，如果没有可用空间，我们将分配给别的地方。

##### 测试

使用JPRT，Nashorn + Octane，SPECjbb2013，SPECjbb2005，SPECjvm2008的强化正确性测试。

我们需要确保没有性能降低，特别是对于具有小代码缓存大小的嵌入式使用。

测试受影响的组件，包括可维护性代理，DTrace，Pstack，Java Flight Recorder。

##### 风险与假设

如果一个代码堆已满，并且在另一个代码堆中仍然有空间，每个代码堆的固定大小会导致内存的潜在浪费。特别是对于非常小的代码高速缓存大小，即使还有空间可用，也可能会关闭编译器。为了解决这个问题，将添加一个选项来关闭小代码缓存大小的分段。

非方法代码的大小取决于Java应用程序，底层平台和JVM设置。因此，在JVM启动时，很难确定非方法代码堆中所需的空间。

该补丁的未来版本可以实现动态调整大小（由扫描器支持）或不同的分配策略来降低浪费内存的风险。
#### JEP 199: Smart Java Compilation, Phase Two
##### 概要

改进sjavac工具，使其可以在JDK构建中默认使用，并将其概括为可以用于构建JDK以外的大型项目。

##### 目标

由于与稳定性和可移植性有关的各种问题，sjavac在JDK构建脚本中默认不被使用。这个JEP的第一个目标是解决这些问题。这包括确保该工具始终在所有软件/硬件配置上产生可靠的结果。

总体目标是提高sjavac的质量，使之成为能够编译大型任意Java项目的通用javac包装。

一个后续的项目将探讨如何在JDK工具链中暴露sjavac;这可能是单独支持的独立工具，不支持的独立工具，与javac集成或其他内容。

##### 非目标

这个JEP的目标不是引进今天没有的新功能，而不是我们自己的构建脚本所要求的，并且不会为通用的javac包装器做出贡献。意图是不要重写包装器。目前功能很好的功能将保持原样。

如果需要，将sjavac纳入产品将成为单独JEP的主题。

##### 动机

目前的实现被证明是有用的，确实提高了构建速度并允许增量版本。然而，整个工具的守则质量和稳定性并不令人满意，当然没有准备公开发行。

##### 描述

这项工作建立在JEP139中描述的以前提供的实现的基础上。本JEP中描述的工作将通过编写，测试和提交增量补丁到现有实现来逐步执行。

最初的工作重点是解决与构​​建JDK有关的问题。这包括解决：

JDK-8014510在JPRT的所有平台上修复sjavac
JDK-8019239正确清理sjavac启发式
将处理的其他问题包括：

可能通过利用RMI来重构协议
使选项名称和语法与javac的选项名称和语法更一致
替换系统。{out，err}记录更合适的东西
在可能的情况下，通过在javac中共享更多的资源来增加并行性
要进一步解决的问题包括但不限于：

国际化
支持拼图
将需要对javac进行某些调整。最值得注意的是，必须重构javac以允许多个并发编译共享通用数据结构，以便在多核机器上实现更好的加速和更高的CPU利用率。

##### 测试

除了通常的强制性单元测试之外，sjavac将在所有平台上进行彻底的测试，因为这些工具的一部分涉及特定于平台的方面，例如文件路径和后台进程的产生。

此外，由于服务器 - 客户端架构本质上是并行的，所以sjavac将会考虑到并发性的压力测试。

##### 依赖性

该JEP提出的工作基于JEP 139的结果：增强javac以提高构建速度。

####JEP 200: The Modular JDK
##### 概要

使用由JSR 376指定并由JEP 261实现的Java Platform Module System来模块化JDK。

##### 目标

将JDK划分成一组模块，可以在编译时，构建时间或运行时间组合到各种配置中，包括但不限于：

对应于完整的Java SE平台，完整的JRE和完整的JDK的配置;

在Java SE 8中定义的每个紧凑型配置文件的内容大致相同的配置;和

自定义配置只包含一组指定的模块和这些模块所需的模块。

模块化结构的定义必须明确区分标准模块，其规范由Java社区流程管理，并且特定于JDK的模块。它还必须区分将被提议包含在Java SE平台规范中的模块，从而在所有其他模块的每个平台实施中都是强制性的。

##### 动机

项目拼图旨在为Java SE平台设计和实施标准模块系统，并将该系统应用于平台本身以及JDK。其主要目标是使平台的实现更容易扩展到小型设备，提高安全性和可维护性，提高应用程序性能，并为开发人员提供更好的编程工具。

##### 描述

##### 设计原则

这里提出的模块化结构实现了以下原则：

标准模块的规格由JCP管理，必须以字符串“java”开头。

所有其他模块只是JDK的一部分，必须以字符串“jdk”开头。

如果一个模块导出一个包含一个包含公共或受保护成员的类型的包，而这个包反过来又引用了一个其他模块的类型，那么第一个模块必须通过require public来为第二个模块提供隐含的可读性。 （这样可以确保方法调用链以明显的方式工作。）

标准模块可以包含标准API和非标准API包。如果标准模块导出标准API包，则导出可能是合格的;如果标准模块导出非标准API包，那么导出必须合格。在任一情况下，如果标准模块导出具有限定条件的包，则导出必须是JDK中某些模块子集。如果标准模块是Java SE模块，即要被提议包含在Java SE平台规范中，那么它不能导出任何非SE API软件包，至少没有资格。

标准模块可以依赖于一个或多个非标准模块。它不得将任何非标准模块隐含的可读性。如果它是Java SE模块，那么它不能向任何非SE模块授予隐含的可读性。

非标准模块不得导出任何标准API包。非标准模块可以向标准模块授予隐含的可读性。

原则4和5的一个重要结果是仅依赖于Java SE模块的代码将仅依赖于标准Java SE类型，因此可以移植到Java SE平台的所有实现。

#####  模块图

JDK的模块化结构可以可视化为一个图形：每个模块都是一个节点，如果第一个依赖于第二个模块，则有一个模块到另一个模块的有向边。完整的模块图具有太多的边缘可以轻松显示;这里是图形的传递减少，其中省略了冗余边（点击放大）：

![image](https://bugs.openjdk.java.net/secure/attachment/57772/jdk.png)

随后参观模块图：

标准Java SE模块为橙色;非SE模块是蓝色的。

如果一个模块依赖于另一个模块，并且给予该模块隐含的可读性，则从第一个模块到第二个模块的边缘是固定的;否则，边缘是虚线。

最底层是java.base模块，它包含必需的类，如java.lang.Object和java.lang.String。基本模块取决于没有模块，每个其他模块都取决于基本模块。基座模块的边缘比其他边缘轻。

顶部是java.se.ee模块，它将组合Java SE Platform的所有模块集合在一起，包括与Java EE Platform Specification重叠的模块。这是一个聚合器模块的示例，它收集并重新导出其他模块的内容，但不添加自己的内容。配置为包含java.se.ee模块的运行时系统将包含SE Platform的所有API包。将建议将一个模块包含在Java SE平台规范中，并且仅当它是可从java.se.ee模块访问的标准模块时。

java.se聚合器模块将Java SE Platform中与Java EE不重叠的部分集中在一起。

java.compact1，java.compact2和java.compact3聚合器模块实现Java SE Compact配置文件。还有一个非标准的jdk.compact3模块，因为JDK的compact3构建包括一些非SE API包，而另一个Compact Profile构建则不包含。

其他非标准模块包含调试和可维护性工具和API（例如，jdk.jdi，jdk.jcmd和jdk.jconsole），开发工具（例如jdk.compiler，jdk.javadoc和jdk.xml.bind）和各种服务提供商（例如jdk.charsets，jdk.scripting.nashorn和jdk.crypto.ec），它们通过现有的java.util.ServiceLoader机制提供给其他模块。

java.smartcardio模块是标准的，但不是Java SE平台规范的一部分，因此其名称以字符串“java”开头。但它是蓝色的，并且它不可从java.se模块访问。

模块图实际上是一种新的API，并将被指定和演进。将提出根据java.se.ee模块生成的所有非SE模块和相应边缘被删除的模块图的子图将被包含在Java SE平台规范中;其进化之后将由JCP管理。图表剩余部分的演变将会覆盖未来的JEP。在任一情况下，如果一个模块被指定为可用于一般使用，那么它将受到与其他API相同的进化约束。特别是删除这样的模块或改变它不一定要提前公布至少一个主要版本。

所有模块的表格摘要，包括Linux / AMD64构建的脚印指标，可在此处获得。

##### 开放式问题

目前的模块图定义至少有一个已知的缺陷，如下所列。修复此问题可能会导致图形内容和结构的微小变化。

java.management模块取决于java.rmi，它本身是大小不重要的，在嵌入式设备场景中通常是不必要的;我们正在将javax.management.remote.rmi API包移动到自己的模块中。
##### 测试

JDK和jtreg中的单元和回归测试（用于运行它们的线束）将被增强，以便根据它们所依赖的模块来选择测试，以便JDK模块的任意配置可以被测试。

此增强功能的主要功能测试将能够检查配置的模块集，以确保它是本文定义的模块的有效组合，每个模块都具有预期内容并导出预期的API包，并且模块具有预期的依赖关系。

必须增强JCK以测试成为Java SE平台规范一部分的模块图形的这些方面。这包括SE模块的名称，其导出的API包以及导致SE API包重新导出的依赖关系。还必须增强JCK以测试平台实现中存在的SE模块的任意配置。

##### 风险与假设

这里定义的模块化结构可能不会解决一些重要的用例。如果在此JEP的第一个发布实施中不支持关键用例，那么我们希望能够在以后的版本中通过重构模块图来解决这个问题。

如上所述，本JEP中定义的模块图的Java SE方面将被提交给最终的Java SE Platform JSR专家组进行审查和标准化。该审查可能要求进一步修改图表。

##### 依赖性

这个JEP是几个项目拼图之一。其他JEP是：
201: Modular Source Code
220: Modular Run-Time Images
260: Encapsulate Most Internal APIs
261: Module System; see also JSR 376
282: jlink: The Java Linker

#### JEP 201: Modular Source Code
##### 概要

将JDK源代码重组为模块，增强构建系统以编译模块，并在构建时执行模块边界。

##### 非目标

该JEP不会改变JRE和JDK二进制映像的结构，也不会引入模块系统。这项工作将由相关JEP和适当的JSR涵盖。

该JEP将为JDK定义一个新的源代码布局。这种布局可以在JDK之外使用，但是这个JEP不是设计广泛接受的通用模块化源代码布局的目标。

##### 动机

项目拼图旨在为Java SE平台设计和实施标准模块系统，并将该系统应用于平台本身以及JDK。其主要目标是使平台的实现更容易扩展到小型设备，提高安全性和可维护性，提高应用程序性能，并为开发人员提供更好的编程工具。

这个JEP是项目拼图的第一步;后来的JEP将模块化JRE和JDK映像（JEP 220），然后引入模块系统（JEP 261）。

在这个早期阶段重组源代码的动机是：

让JDK开发人员有机会熟悉系统的模块化结构;

即使在引入模块系统之前，仍然通过在构建中实施模块边界来保持结构的前进;和

实现项目拼图的进一步开发，无需将当前的非模块化源代码“混洗”成模块化形式。

##### 描述

##### 现行方案

大多数JDK源代码今天组织起来，大致上可以追溯到1997年的方案。缩写形式：

```
src/{share,$OS}/{classes,native}/$PACKAGE/*.{java,c,h,cpp,hpp}
```

哪里：

共享目录包含共享的跨平台代码;

$ OS目录包含操作系统特定的代码，其中$ OS是solaris，windows等之一;

classes目录包含Java源文件和可能的资源文件;

本机目录包含C或C ++源文件;和

$ PACKAGE是相关的Java API包名称，用斜杠替换句点。

要简单的示例，jdk存储库中java.lang.Object类的源代码驻留在两个文件中，一个在Java中，另一个在C：

```
src/share/classes/java/lang/Object.java
          native/java/lang/Object.c
```

对于一个不太简单的例子，package-private java.lang.ProcessImpl和ProcessEnvironment类的源代码是操作系统特定的;对于类Unix系统，它驻留在三个文件中：

```
src/solaris/classes/java/lang/ProcessImpl.java
                              ProcessEnvironment.java
            native/java/lang/ProcessEnvironment_md.c
```
（是的，二级目录被命名为solaris，即使该代码与所有Unix衍生工具相关;下面更多的）

在src / {share，$ OS}下有一些与当前结构不匹配的目录，包括：

| Directory | Content |
| --------- | --------- | 
| src/{share,$OS}/back | JDWP back end | 
| bin | Java launcher | 
| instrument | Instrumentation support | 
| javavm |  Exported JVM include files | 
|  lib | Files for $JAVA_HOME/lib | 
| transport  | JDWP transports | 

##### 新计划

JDK的模块化提供了一个难得的机会来完全重构源代码，以便使其更容易维护。 我们建议在除了热点之外的JDK林中的每个仓库中实施以下方案。 简写形式：

```
src/$MODULE/{share,$OS}/classes/$PACKAGE/*.java
                        native/include/*.{h,hpp}
                               $LIBRARY/*.{c,cpp}
                        conf/*
```
              
##### 哪里：

$ MODULE是一个模块名称（例如java.base）;

共享目录包含与之前一样的共享的跨平台代码;

$ OS目录包含操作系统特定的代码，如前所述，其中$ OS是unix，windows等之一;

classes目录包含Java源文件和资源文件，如前所述，组织成目录树，反映其API $ PACKAGE层次结构;

本机目录包含C或C ++源文件，如前所述，但组织方式不同：

include目录包含用于外部使用的C或C ++头文件（例如jni.h）;

C或C ++源文件放置在$ LIBRARY目录中，其名称是将编译代码链接到的共享库或DLL的名称（例如libjava或libawt）; 最后，

conf目录包含要由最终用户编辑的配置文件（例如，net.properties）。

要重写以前的示例，java.lang.Object类的源代码将按如下方式排列：

```
src/java.base/share/classes/java/lang/Object.java
                    native/libjava/Object.c
```
package-private java.lang.ProcessImpl和ProcessEnvironment类的源代码将以这种方式布置：

```
src/java.base/unix/classes/java/lang/ProcessImpl.java
                                     ProcessEnvironment.java
                   native/libjava/ProcessEnvironment_md.c
```

（我们借此机会，最后，将solaris目录重命名为unix。）

目前src/{share,$OS}目录中与目前结构不符的目录将被移动到适当的模块中：

```
Directory                     Module
--------------------------    --------------------------
src/{share,$OS}/back          jdk.jdwp.agent
                bin           java.base
                instrument    java.instrument
                javavm        java.base
                lib           $MODULE/{share,$OS}/conf
                transport     jdk.jdwp.agent
```
当前lib目录中的文件不会被最终用户编辑，将被转换为资源文件。
##### 构建系统更改

修改系统将一次编译一个模块而不是一个存储库，它将根据模块图的反向拓扑排序来编译模块。 直接或间接相互依赖的模块将尽可能并行编译。

编译模块而不是存储库的一个好处是，corba，jaxp和jaxws存储库中的代码将能够使用新的Java语言功能和API。 这是以前被禁止的，因为这些存储库是在jdk存储库之前编译的。

中间（即非图像）构建中的编译类将被分为模块。 今天我们有：

```
jdk/classes/*.class
```
修订后的制作系统将产生：

```
jdk/modules/$MODULE/*.class
```
如上所述，图像构建的结构不会改变; 他们的内容会有很小的差异。

模块边界将在构建系统的执行期间尽可能地执行。 如果模块边界被违反，则构建将失败。 边界将在JEP 200中描述的modules.xml文件中定义，它将与源代码一起保留。 此文件的更改将需要“项目拼图工程师”的审查。
##### 备择方案

还有许多其他可能的源 - 布局方案，包括：

在顶部保留{share，$ OS}，其中包含模块目录以包含模块类文件：

```
src/{share,$OS}/modules/$MODULE/$PACKAGE/*.java
                native/include/*.{h,hpp}
                       $LIBRARY/*.{c,cpp}
                conf/*
```

将所有内容放在相应的$ MODULE目录下，但保持{share，$ OS}在顶部：

```
src/{share,$OS}/$MODULE/classes/$PACKAGE/*.java
                        native/include/*.{h,hpp}
                               $LIBRARY/*.{c,cpp}
                        conf/*
```
将{share，$ OS}下拉到$ MODULE目录中，如本建议中所述，但删除中间类目录，并使用下划线对native和conf目录的名称进行前缀，以简化纯文本的常见情况Java模块：

```
src/$MODULE/{share,$OS}/$PACKAGE/*.java
                        _native/include/*.{h,hpp}
                                $LIBRARY/*.{c,cpp}
                        _conf/*
```
方案3的一个变体，但是顶部有{share，$ OS}

```
src/{share,$OS}/$MODULE/$PACKAGE/*.java
                        _native/include/*.{h,hpp}
                                $LIBRARY/*.{c,cpp}
                        _conf/*
```
方案3的另一个变体是更深入地推动{share，$ OS}，以便进一步简化纯Java模块的案例，而不需要$ OS特定的代码：

```
src/$MODULE/$PACKAGE/*.java
            _native/include/*.{h,hpp}
                    $LIBRARY/*.{c,cpp}
            _conf/*
            _$OS/$PACKAGE/*.java
                _native/include/*.{h,hpp}
                        $LIBRARY/*.{c,cpp}
                _conf/*
```
我们拒绝涉及强调（3-5）的方案太陌生，难以浏览。我们更喜欢现有的方案1和方案2，因为它需要从当前的方案进行最小的改变，同时将模块的所有源代码放在单个目录下。必须修改依赖于当前方案的工具和脚本，但至少对于Java源代码，每个$ MODULE目录下的结构与以前相同。

我们考虑的其他问题：

我们应该为资源文件定义不同的目录，以便它们与Java源文件分开？ - 不 这似乎不值得麻烦。

一些模块具有跨越存储库的内容; 这是个问题吗？ - 这是一个烦恼，但构建系统可以通过VPATH机制的魔力来处理它。 随着时间的推移，我们可能会重组存储库以减少甚至消除交叉回购模块，但这超出了此JEP的范围。

一些模块有多个本机库; 我们应该合并它们，以便每个模块最多有一个本机库？ - 不 在某些情况下，我们需要每个模块多个本机库的灵活性，例如“无头”与“头脑”的AWT。

##### 测试

如上所述，这个JEP不会改变JRE和JDK二进制图像的结构，只会对内容进行微小的改变。因此，我们可以通过比较使用它构建的图像与没有它的图像进行比较来验证此更改，并运行测试以验证实际的微小更改。

##### 风险与假设

我们假设Mercurial将能够处理大量的文件重命名操作，以执行此更改，并保留进程中的所有历史信息。早期的测试表明，Mercurial能够实现这一目标，但是文件的新旧位置之间的关系也不会被正确记录的风险仍然很小。在这种情况下，文件在其旧位置的历史仍将在存储库中;这将更难找到。

将使用旧方案将存储库创建的修补程序直接应用到使用新方案的存储库是不可能的，反之亦然。为了减轻这种情况，我们计划开发一个脚本，将补丁中的文件名从旧位置翻译成新位置。

##### 依赖性

这个JEP是项目拼图的几个JEP的第二个。它结合了来自JEP 200的JDK的模块化结构的定义，但并不明确地依赖于该JEP。

#### JEP 211: Elide Deprecation Warnings on Import Statements
##### 概要

从Java SE 8开始，通过对Java语言规范进行合理解释，通过名称导入不推荐使用的类型，或者静态导入不推荐使用的成员（方法，字段，嵌套类型）时，Java语言规范将会出现Java编译器。这些警告是不知情的，不应该被要求。应该保留对弃用成员实际使用的弃用警告。

##### 目标

这个JEP的目标是促进大规模代码清理棉绒警告。使用@SuppressWarnings注释不能使用代码中不赞成使用的成员来抑制对导入的废弃警告。在像JDK这样的大型代码库中，不推荐使用的功能通常需要一段时间才能被支持，只有导入不推荐使用的结构，如果所有使用不推荐使用的构造都是有意的和被抑制的，那么它就不能证明一个警告消息。

##### 非目标

这个JEP的目标不是在JDK代码中实际解决所有的弃用警告。但是，这可能是JDK 9中单独维护工作的一部分。

##### 描述

从规格的角度来看，需要的变化很小。在JLS 8中，@Deprecated部分指出：

当明确或隐式声明的结构中使用（覆盖，调用或引用名称）的类型，方法，字段或构造函数（其声明由@Deprecated注释）时，Java编译器必须产生废弃警告，除非：

使用在一个实体中，其本身使用注释@Deprecated注释;要么
使用注释的一个实体中，以通过注释@SuppressWarnings（“deprecation”）抑制警告;要么
使用和声明都在同一最外层。
规范更改将是添加另一个项目符号，说明额外的排除：

使用在import语句中。
在javac引用实现中，在查找deprecation警告时，会有一个简单的检查来跳过import语句。

##### 测试

正常单元测试应足以测试此功能。对于更改的规范，可能需要更新少量JCK测试。

#### JEP 212: Resolve Lint and Doclint Warnings
##### 概要

JDK代码库包含javac报告的许多lint和doclint错误。这些警告应该解决，至少对于平台的基本部分。

##### 目标

在操作上，目标是至少在javac的lint和doclint警告下，平台（在core-libs上讨论的那些基础软件包，awt-dev，swing-dev，2d-dev等）至少可以完全编译。希望其他包，如包括JAXP，JAX-WS和CORBA的软件包也可以在启用所有警告的情况下进行干净的编译。

##### 成功指标

当javac命令使用-Xlint：all选项时，成功构建了相关的源代码。可能可以接受的稍微较弱的目标是启用所有与源相关的lint选项，而不是非源属性的lint选项。例如，一些lint选项涉及javac命令行的属性，而不是正在编译的源。

##### 描述

该JEP建议完成在JDK 8和JDK 9中进行的警告修复工作，以及形式化以前提出给jdk9-dev的源代码改进子集。大多数警告通过修改方法体的内部来解决。解决一些rawtypes警告涉及更改方法签名，例如将参数类型从原始java.lang.Class更改为java.lang.Class <？>或更具体的类型。任何API更改将保留在JDK的一般演进策略中。

##### 测试

成功的编译/构建是大多数更改的主要测试，但现有的回归测试应该继续通过。如果Java SE API具有签名更改，则相应的JCK签名测试将需要相应更新。

##### 依赖性

如果导入不推荐的类型不会触发废弃警告，则解决JDK中的弃用警告将被缓解。

#### JEP 213: Milling Project Coin
##### 概要

作为JDK 7 / Java SE 7的一部分，Project Coin / JSR 334中包含的小型语言变化已经很容易使用，并且在实践中表现良好。然而，一些修正案可以解决这些变化的粗暴边缘。此外，使用下划线（“_”）作为生成Java SE 8的警告的标识符，应该在Java SE 9中变为错误。还建议允许接口具有私有方法。

##### 非目标

该JEP不建议运行“硬币2.0”的努力或一般征求新的语言建议。

##### 描述

提出了对Java编程语言的五个小修改：

允许@SafeVargs在私有实例方法。 @SafeVarargs注释只能应用于不能被覆盖的方法，包括静态方法和最终实例方法。私有实例方法是@SafeVargs可以适应的另一种用例。

允许有效的最终变量在try-with-resources语句中用作资源。 Java SE 7中的try-with-resources语句的最终版本需要为语句管理的每个资源声明一个新的变量。这是从早期的功能迭代中的变化。 JSR 334的公共审查草案讨论了从提交资源审查版本的早期审查版本的变化的原理，允许表达式管理表达式。 JSR 334专家组赞成对资源尝试进行额外的改进：如果资源由最终或有效的最终变量引用，那么try-with-resources语句可以管理资源，而不会声明新的变量。由try-with-resources语句管理的这种受限表达式避免了消除一般表达式支持的语义问题。当专家组解决这一改进时，发布时间表中没有足够的时间来适应这一变化。

如果推断类型的参数类型是可表示的，则允许具有匿名类的菱形。因为使用具有匿名类构造函数的菱形的推断类型可能不在签名属性支持的类型集合之外，所以在Java SE 7中不允许使用具有匿名类的菱形。如JSR 334提出的最终草案中所指出的那样，如果推断的类型是可表示的，可以缓解这种限制。

完成从Java SE 8开始的从法定标识符名称集合中下划线的删除。

考虑到接口中的私有方法的支持被简单地考虑在Java SE 8中，作为添加对Lambda表达式的支持的一部分，但是被撤回以便更好地关注Java SE 8的更高优先级任务。现在建议支持为了实现私有接口方法，使得接口的非抽象方法能够在它们之间共享代码。

在Java语言变化的空间中，这些改进是非常小的变化。 @SafeVarags更改可能只涉及使用javac中类似大小的更改来更改规范中的一个或两个。但是，与任何Java语言的更改一样，必须注意处理需要更新的所有平台。

##### 测试

语言变化将需要通常的javac单元和回归测试。需要更新JCK编译器套件，包括正测试和否定测试。

#### JEP 214: Remove GC Combinations Deprecated in JDK 8
##### 概要

删除先前在JDK 8中通过JEP 173弃用的GC组合。

##### 非目标

这项工作的目的不是为了移除GC组合来实施更换。在大多数情况下，剩余的收集器应该比除去的GC组合更好或更好。这项工作的目的不是增强任何垃圾收集者的表现以及正在移除的垃圾收集器。只有对垃圾收集者的调整才能被认为是履行目标的一部分。

##### 成功指标

没有为相关的JVM选项打印弃用消息。相反，JVM认为它们是未知的标志。

从ParNew + SerialOld到ParallelScavenge + SerialOld的性能影响，对于一套合理的基准测试是非常明白的。

##### 动机

如JEP 173中所述，与保持所有现有GC组合相关联的维护成本较高。删除已弃用的GC组合将允许HotSpot GC代码中的许多简化，这反过来将减少错误的数量，并允许更快速地开发剩余的GC组合。

##### 描述

用于控制在JEP 173中不推荐使用的GC组合的标志以及启用CMS前台收集器（不作为JDK-8027876的一部分使用）的标志将从代码库中移除。这意味着不再为他们打印任何警告信息;相反，如果使用这些标志，JVM将不启动。

一旦标志被删除，那么任何现在死亡的代码都将被删除为GC代码库。代码库可能会有简化，这是因为这个工作可能做，但范围很大。这样的简化可以作为单独的改变来分离。

以下是将停止工作的标志和标志组合的详细摘要：

```
DefNew + CMS       : -XX:-UseParNewGC -XX:+UseConcMarkSweepGC
ParNew + SerialOld : -XX:+UseParNewGC
ParNew + iCMS      : -Xincgc
ParNew + iCMS      : -XX:+CMSIncrementalMode -XX:+UseConcMarkSweepGC
DefNew + iCMS      : -XX:+CMSIncrementalMode -XX:+UseConcMarkSweepGC -XX:-UseParNewGC
CMS foreground     : -XX:+UseCMSCompactAtFullCollection
CMS foreground     : -XX:+CMSFullGCsBeforeCompaction
CMS foreground     : -XX:+UseCMSCollectionPassing
```
对于ParNew + SerialOld组合，此JEP的工作还将包括将ParNew + SerialOld与ParallelScavenge + SerialOld进行比较的性能测试。这将导致调整从ParNew + SerialOld迁移到ParallelScavenge + SerialOld的建议。

##### 备择方案

在代码中保留所有或部分不推荐的选项。这将阻止在GC代码中进行重大的清理工作，并减慢代码库中的新开发。

##### 测试

目前有测试验证列出的选项是否打印警告消息。这些测试需要更新，以期望这些选项是未知的消息。

##### 风险与假设

正在使用任何标志的用户将不得不更新其JVM启动命令行。如果他们从JDK 8迁移到JDK 9，那么他们已经看到了警告信息，因此不应该感到惊讶。假设大多数用户将受益于从正在删除的标志更改为更现代的GC调优。

##### 依赖性

这项工作取决于JEP 173：退出一些很少使用的GC组合。该JEP在JDK 8中实现，所以目前还没有很好的依赖关系。
#### JEP 215: Tiered Attribution for javac
##### 概要

在javac中实现一种新的方法类型检查策略，以加快参数位置中poly表达式的归属。

##### 目标

实施一种新的方法来检查多边形表达式，分层归属（TA），它提供：

通过实现一种减少归属给定表达式所需的（冗余）传递次数的方法来提高性能，以及

与当前的类型检查实现相同的结果。

##### 非目标

改变可编译程序的空间并不是一个目标，尽管在现有的类型检查方案中修复潜伏的错误可能会导致一些更改。也可以预期编译器生成的消息中的一些细微差异。

显着提高当前代码的清洁度和可维护性也是一个非目标 - 尽管可能的是，此JEP引入的某些更改也将导致更清晰的代码。

##### 动机

目前实现的Java SE 8多元表达式的类型检查方法被称为“投机归因”（SA）; SA的主要思想是对不同的目标进行多次检查相同的树;这使得可以例如检查一个lambda表达式对多个重载分辨率目标。

在重载分辨率中进行类型检查的能力是一种非常强大而灵活的技术，但在性能方面的价格非常高。更准确地说，使用N个过载候选，相同的参数表达式可以被检查到N * 3（每个过载阶段一次，严格，松散，可变量）+ 1（最终检查阶段）。如果参数表达式允许嵌套（例如，lambda返回多方法调用），那么这些因素需要相乘，导致归因调用的组合爆炸。

对投机归属机制的指数数量已经导致已经被观察和报告为bug的性能问题，例如JDK-8077247，JDK-8078093和JDK-8055984。

##### 描述

该JEP提出了一种替代和更有效的实现方案，用于支持javac中的多项表达式的类型检查。从概念上讲，在执行重载分析时，不需要键入检查表达式;实际上，一个参数表达式可以用自下而上的样式进行类型检查 - 导致无归属重载检查，或者这种表达式与适用性无关（见JLS 15.12.2.2） - 意思是这样的表达不会有贡献对过载分辨率的适用性检查。例如，lambda表达式可以是显式的 - 在这种情况下，可以在重载分辨率之前执行身体的类型检查;或者可以是隐式的，在这种情况下，在重载解析期间不需要进行类型检查。

分层归属背后的主要思想是，在重载解析之前，产生自下而上的结构类型（一个用于给定方法调用中发生的每个多参数表达式），以及执行重载分辨率适用性检查所需的所有信息 - 即不再进一步需要归因这种结构类型可能包含部分推断的类型变量，这些类型变量只能在稍后阶段被固定 - 在调用类型推理过程中（见JLS 18.5.2）。将为以下参数表达式创建新的结构类型：

Lambda表达式，
条件聚表达式，
通用方法调用，
括号多项表达式，
方法参考，和
钻石实例创建表达式。
由于上面提到的一些表达式可以允许嵌套，所以结构类型可以提及其他结构类型。例如，返回一般方法调用的lambda表达式的情况可以使用结构类型（对于lambda表达式）进行建模，指向另一种构造通用方法调用的结构类型。在这种情况下，与结构类型相关联的过载检查可以是递归的。 lambda体中的返回表达式提到的所有结构类型都必须根据重载分辨率机制提供的目标类型进行检查。

在归因方法/构造函数调用的参数时，通常会创建结构类型。在方法/构造函数调用以外的上下文中出现的多义表达式应以与当今相同的方式进行处理。

还需要对类型推断进行一些更改，以便适当地实现嵌套的泛型方法调用;更具体地说，类型推理机构必须配备更好的保存和回滚功能，以便处理涉及通用方法调用（可能需要执行一些推理任务）的过载检查，而不会永久性地污染与这种呼叫相关联的上下文信息。

##### 测试

javac已经包含了一套全面的回归测试来证明SA按预期工作;而大多数这些测试主要是黑盒测试，它们应该是b
#### JEP 216: Process Import Statements Correctly
##### 概要

修复javac正确接受和拒绝程序，而不管import语句的顺序和extends和implements子句。

##### 动机

在某些情况下，javac将接受具有特定进口顺序的源代码，并拒绝与重新排序的导入（例如，JDK-7177813）相同的源代码。这是错误和混乱。

##### 描述

javac在编译类时使用几个阶段。考虑到进口处理，两个重要阶段是：

类型分辨率，通过提供的AST查找类和接口声明，以及

会员决议，其中包括：

（1a）如果T是toplevel，定义T的源文件中的导入将被处理，导入成员将添加到T的范围内
（1b）如果T是嵌套的，则直接包围T的类的解析（如果有的话）
（2）对T的扩展/实现子句进行类型检查
（3）对类型变量进行类型检查
上述阶段是javac解析类的过程的一部分，其中包括确定类的超类型，类型变量和成员。

为了看这个在执行过程中，请考虑以下代码：

```
package P;

import static P.Outer.Nested.*;
import P.Q.*;

public class Outer {
    public static class Nested implements I {
    }
}

package P.Q;
public interface I {
}
```
在类型分解阶段期间，认识到存在类型P.Outer，P.Outer.Nested和P.Q.I.然后，如果要分析P.Outer类，则成员解析阶段的工作原理如下：

解决P.Outer开始
2.处理进口静态P.Outer.Nested。*;开始，每1a，这意味着P.Outer.Nested的成员及其传递超类型被查找。
3.解析P.Outer.Nested类启动（静态导入也可以导入继承类型）
4.触发P.Outer的解决方案，它已经在进行中被跳过
5.对I（implements子句）的类型检查运行，但Icannot将被解析，因为它还没有在范围内。
6.导入P.Q. *启动的解决方案，它将所有成员类型的P.Q（包括接口I）导入到当前文件的范围内
继续执行P.Outer和其他班级的决议
如果导入被交换，则步骤6发生在步骤5之前，因此在步骤5期间找到。

以上不是与进口处理有关的唯一问题。另一个已知的问题是类的类型参数的边界可能有效地引用它们的声明类的可能的内部类。在某些情况下，这会导致无法解决的周期，例如：

```
package P;

import static P.Outer.Nested.*;

public class Outer {
    public static class Nested<T extends I> {
        static class I { }
    }
}
```
该问题的设想解决方案是将javac成员解析的现有第一阶段拆分为三个：第一阶段将分析封装文件的导入，第二阶段将仅构建类/接口层次结构，而不需要任何类型参数，注释等， 第三个将正确分析类头，包括类型参数。

预计此更改将允许javac接受当前被拒绝但不拒绝当前接受的程序。
#### JEP 217: Annotations Pipeline 2.0
##### 概要

重新设计javac注释管道以更好地满足处理注释的注释和工具的需求。

##### 目标

编译器应该在注释和类型注释方面正常运行：发布的类文件应具有各种注释的正确格式的属性。

编译时反射（javax.lang.model和javax.annotation.processing）应正确处理签名可见位置中的所有注释。

运行时反射（Core Reflection）应该能够在类文件中使用注释。

Checkers框架必须继续工作。

Java编译器API应该继续按照设计工作。

##### 非目标

不会添加新的语言功能或API。在javadoc和javap等相关工具中更新对注解的支持并不是一个目标。

##### 动机

Java SE 8引入了两个新的注释功能：重复注释（JEP 120）和Java类型注释（JSR 308 / JEP 104）。另外，Lambda表达式（JSR 335 / JEP 126）为注释添加了新的句法位置。当javac注释管道最初被构想时，这些功能都不存在，可以组合在一起，导致以下模式：

```
Function<String, String> fss = (@Anno @Anno String s) -> s;
```

现有的注释管道不能开箱即用;因此，为了适应新的用例，原来的设计已经被拉伸，导致了一个脆弱和难以维护的实施。这项工作的目标是用一种新的使用案例替代这种老化架构，以更直接的方式支持新的用例，从而导致更正确和可维护的代码。

##### 描述

重构javac注释管道。除了我们修复错误并提高正确性之外，这不应该是外部显着的。第一步是提高测试覆盖率，以便我们测量和评估我们的退出标准。之后是一系列渐进重构。这项工作将在OpenJDK注释管道2.0项目中完成。

javadoc工具与类型注释有一些相关的问题。然而，javadoc在Javadoc.Next Project中作为一部分工作正在进行中。该工作的一部分包括将javadoc转换为使用javax.lang.model API而不是旧的com.sun.javadoc API。因此，这个项目不是使用javadoc来确保注释（包括类型注释）正确显示的目标。预计，作为JavaDoc.Next项目的一部分，将增强javadoc以利用作为此项目目标的javax.lang.model API的更新。

##### 测试

已经很好地覆盖了大量注释的端到端用例。这包括JCK和langtools回归测试。这项工作的很大一部分是开展进一步的测试，以确保成功度量的可衡量性。

如上所述，我们将创建测试，以运行Java SE 8中的新功能。

#### JEP 219: Datagram Transport Layer Security (DTLS)
##### 概要

定义数据报传输层安全性（DTLS）版本1.0（RF​​C 4347）和1.2（RFC 6347）的API。

##### 非目标

支持传输特定接口（例如DatagramSocket的DTLS）不是目标。

支持PMTU发现不是一个目标。

##### 成功指标

在客户端和服务器模式下的实现必须与至少两个其他DTLS实现成功交互。

##### 动机

支持DTLS以满足越来越多的数据报兼容应用程序的安全传输要求是非常重要的。 RFC 4347列出了TLS对于这些类型的应用程序不足够的一些原因：

“TLS是用于保护网络流量的最广泛部署的协议...但是，TLS必须运行在可靠的传输通道（通常是TCP）上，因此不能用于保护不可靠的数据报流量。

“...已经设计出越来越多的使用UDP传输的应用层协议，特别是诸如会话发起协议（SIP）和电子游戏协议之类的协议越来越受欢迎。

“在许多情况下，保护客户端/服务器应用程序最理想的方式是使用TLS;但是，对数据报语义的要求自动禁止使用TLS，因此，与数据报兼容的TLS变体是非常可取的。

支持DTLS的协议包括但不限于：

RFC 5238，数据报传输层安全（DTLS）通过数据报拥塞控制协议（DCCP）

RFC 6083，用于流控制传输协议（SCTP）的数据报传输层安全（DTLS）

RFC 5764，用于建立安全实时传输协议（SRTP）密钥的数据报传输层安全（DTLS）扩展

RFC 7252，约束应用协议（CoAP）

Google Chrome和Firefox现在支持用于Web实时通信（WebRTC）的DTLS-SRTP。主要TLS供应商和实施包括OpenSSL，GnuTLS和Microsoft SChannel都支持DTLS 1.0和1.2版本。

##### 描述

我们期望DTLS API和实现相当小。新的API应该是独立于传输的，类似于javax.net.ssl.SSLEngine。随着工作的进行，这个API的更多细节会被添加到这里。一些初步设计考虑如下：

DTLS API和实现不会管理读取超时。应用程序负责确定适当的超时值，以及何时以及如何触发超时事件。

可能会添加一个新的API来设置最大应用数据报大小（PMTU减去DTLS每记录开销）。但是，如果没有明确指定大小，则DTLS实现应该自动调整大小。如果片段丢失了两三次，则实现可能会减小最大应用数据报大小的大小，直到其足够小。

DTLS实现应为每个解包或包装操作消耗或产生最多一个TLS记录，以便可以单独地在数据报层中传送记录，或者如果传送失败，则可以更容易地重新组装。

如有必要，应用程序有责任组织无序应用程序数据。 DTLS API应提供对每个DTLS消息中应用程序数据的访问。

#### JEP 220: Modular Run-Time Images
##### 概要

重新组织JDK和JRE运行时映像以适应模块，并提高性能，安全性和可维护性。定义一个新的URI方案，用于命名存储在运行时映像中的模块，类和资源，而不会显示图像的内部结构或格式。根据需要修改现有规格以适应这些变化。

##### 目标

对存储的类和资源文件采用运行时格式：

比传统的jar格式更加时间和空间高效，而后者又是基于古老的zip格式;

可以在每个模块的基础上定位并加载类和资源文件;

可以从JDK模块和库和应用程序模块存储类和资源文件;和

可以扩展以适应未来的其他类型的数据，例如预计算的JVM数据结构和Java类的预编译本地代码。

重新构建JDK和JRE运行时映像，以便明确区分开发人员，部署者和最终用户可以依赖的文件，并在适当时对其进行修改，与实施内部的文件形成对比，并且如果没有注意。

提供支持的方法来执行通常的操作，现在只能通过检查运行时映像的内部结构来完成，例如枚举图像中存在的所有类。

启用对当前授予所有安全权限但不实际需要这些权限的JDK类的选择性脱离特权。

保留行为良好的应用程序的现有行为，即不依赖于JRE和JDK运行时映像的内部方面的应用程序。

##### 成功指标

与之前JDK 9构建的JRE，JDK和Compact Profile图像相当的模块化运行时映像不能对代表性的启动，静态占位和动态脚印基准进行回归。

##### 非目标

保存当前运行时映像结构的所有方面不是目标。

保留所有现有API的确切当前行为不是目标。

##### 动机

项目拼图旨在为Java SE平台设计和实施标准模块系统，并将该系统应用于平台本身以及JDK。其主要目标是使平台的实现更容易扩展到小型设备，提高安全性和可维护性，提高应用程序性能，并为开发人员提供更好的编程工具。

这个JEP是计划用于项目拼图的四个JEP中的三分之一。早期的JEP 200定义了模块化JDK的结构，JEP 201将JDK源代码重组为模块。后来的JEP将介绍实际的模块系统。

#####  描述

##### 当前运行时映像结构

JDK构建系统目前生成两种类型的运行时映像：Java R平台的完整实现的Java运行时环境（JRE）和嵌入JRE的Java开发工具包（JDK），其中包含开发工具和图书馆。 （三个Compact Profile构建是JRE的子集。）

JRE映像的根目录包含两个目录bin和lib，其中包含以下内容：

bin目录包含必需的可执行文件，特别是用于启动运行时系统的java命令。 （在Windows操作系统上，它还包含运行时系统的动态链接本机库。）

lib目录包含各种文件和子目录：

各种.properties和.policy文件，其中大部分可能是很少被开发人员，部署者和最终用户编辑的;

默认情况下不存在的认可目录可以放入包含认可标准和独立技术实现的jar文件中;

ext目录，可以放置包含扩展名或可选软件包的jar文件;

各种实现内部数据文件，分类二进制格式，例如字体，颜色配置文件和时区数据;

各种jar文件，包括rt.jar，其中包含运行时系统的Java类和资源文件。

运行时系统在Mac OS，Linux和Solaris操作系统上的动态链接本机库。

JDK映像包含JRE子目录中的JRE副本，并包含其他子目录：

bin目录包含命令行开发和调试工具，例如javac，javadoc和jconsole，以及为了方便起见，jre / bin目录中的二进制文件的重复;

演示和示例目录分别包含演示程序和示例代码;

man目录包含UNIX样式的手册页;

include目录包含C / C ++头文件，用于编译将直接与运行时系统接口的本地代码;和

lib目录包含各种jar文件和其他类型的文件，其中包括JDK工具的实现，其中包括javac编译器的类的tools.jar。

JDK映像的根目录或未嵌入到JDK映像中的JRE映像的根目录还包含各种COPYRIGHT，LICENSE和README文件，以及根据简单的键/值属性对描述映像的发行文件。例如：

```
JAVA_VERSION="1.9.0"
OS_NAME="Linux"
OS_VERSION="2.6"
OS_ARCH="amd64"
```
##### 新的运行时映像结构

JRE和JDK映像之间的区别是纯粹的历史性的，这是JDK 1.2发行版的后期执行决策的结果，从未被重新审视。新的图像结构将消除这种区别：JDK图像将只是一个运行时映像，恰好包含JDK中历史上所有的开发工具和其他项目。

模块化运行时映像将包含以下目录：

bin目录将包含由链接到映像中的模块定义的任何命令行启动器。 （在Windows上，它将继续包含运行时系统的动态链接本机库。）

conf目录将包含要由开发人员，部署者和最终用户编辑的.properties，.policy和其他类型的文件，这些文件以前在lib目录或其子目录中找到。

Unix上的lib目录将包含运行时系统的动态链接本机库，就像今天一样。这些可能与嵌入运行时系统的程序相关联。

lib目录中的所有其他文件和目录必须被视为运行时系统的私有实现细节。它们不适合外部使用，其名称，格式和内容将随时更改，恕不另行通知。

另外，完整的JDK映像还将包含demo，sample，man和include目录。

模块化运行时映像的根目录当然也包含必要的COPYRIGHT，许可证，自述文件和发行文件。为了便于说明运行时映像中存在哪些模块，将使用新的属性MODULES来扩充发行文件，MODULES将是这些模块名称的空格分隔列表。该列表将根据模块的依赖关系进行拓扑排列，因此java.base模块将始终是第一个。
##### 被删除：被认可的标准覆盖机制

经认可的标准覆盖机制允许在Java社区进程之外维护的较新版本的标准的实现，或作为Java SE平台一部分的独立API，而不断独立发展，以安装到运行时映像中。

已批准的标准机制目前根据类似路径的系统属性java.endorsed.dirs和该属性的缺省值$ JAVA_HOME / lib / approved来定义。包含经认可的标准或独立API的较新实现的jar文件可以通过将其放置在由系统属性命名的目录之一中，或通过将其放置在默认的lib /认可目录中，将其安装到运行时映像中系统属性未定义。这样的jar文件在运行时被添加到JVM的引导类路径中，从而覆盖了运行时系统本身存储的任何定义。

模块化图像由模块而不是jar文件组成。展望未来，我们希望通过可升级模块的概念，以模块化形式支持认可标准和独立API。因此，我们建议删除java.endorsed.dirs系统属性，lib / approvaled目录以及实现此机制的代码。为了帮助识别此机制的任何现有使用，我们将修改编译器和启动器，如果设置此系统属性或lib /认可目录存在，则会失败。

##### 删除：扩展机制

扩展机制允许将包含扩展Java SE平台的API的jar文件安装到运行时映像中，以便它们的内容对于在该映像上编译或运行的每个应用程序都可见。

该机制根据类似路径的系统属性java.ext.dirs和由$ JAVA_HOME / lib / ext和特定于平台的系统目录组成的属性的默认值进行定义（例如，/ usr / Linux / packages / lib / ext）。它的工作方式与认可标准机制的方式大致相同，只是放置在扩展目录中的jar文件由运行时环境的扩展类加载器加载，该引导类加载器是系统类的父类，加载器，实际上加载要从类路径运行的应用程序。因此，扩展类不能覆盖由引导加载程序加载的JDK类，但是它们优先于系统加载器及其后代定义的类加载。

扩展机制是在1998年发布的JDK 1.2中引入的，但在现代，我们看不到其使用的证据。这并不奇怪，因为大多数Java应用程序今天将他们需要的库直接放在类路径上，而不是要求将这些库作为运行时系统的扩展进行安装。

在技​​术上可能的是，尽管尴尬，继续支持模块化JDK中的扩展机制。为了简化Java SE平台和JDK，我们建议删除java.ext.dirs系统属性，lib / ext目录以及实现此机制的代码。为了帮助识别此机制的任何现有使用，我们将修改编译器和启动器，如果设置了此系统属性或lib / ext目录存在，则会失败。默认情况下，编译器和启动器将忽略特定于平台的系统范围的扩展目录，但如果指定了-XX：+ CheckEndorsedAndExtDirs命令行选项，那么如果该目录存在并且不为空则它们将失败。

与扩展机制相关的几个功能将被保留，因为它们自己有用：

Class-Path manifest属性，用于指定另一个jar文件所需的jar文件;

指定包和jar文件版本信息的{Specification，Implementation} - {Title，Version，Vendor}清单属性;

密封的清单属性，封装一个包或一个jar文件;和

扩展类加载器本身。

扩展类加载器将被保留以保持兼容性。对于由系统类加载器加载的类Foo，特别是表达式

```
Foo.class.getClassLoader().getParent() != null
```
将保持原样
##### 已删除：rt.jar和tools.jar

以前存储在lib / rt.jar，lib / tools.jar，lib / dt.jar和各种其他内部jar文件中的类和资源文件现在将以更高效的格式存储在lib目录中的实现特定文件中。这些文件的格式将不会被指定，如有更改，恕不另行通知。

##### 删除rt.jar和类似文件会导致三个不同的问题：

现有的标准API（如ClassLoader :: getSystemResource方法）会返回URL对象，以在运行时映像中命名类和资源文件。例如，在JDK 8上运行代码

```
ClassLoader.getSystemResource("java/lang/Class.class");
```
返回窗体的jar URL

```
ClassLoader.getSystemResource("java/lang/Class.class");
```
如可以看到的，它嵌入一个文件URL来命名运行时映像中的实际jar文件。该URL对象的getContent方法可以用于通过jar URL方案的内置协议处理程序来检索类文件的内容。

模块化图像不会包含任何jar文件，因此上述形式的URL是没有意义的。 getSystemResource和相关方法的规范，幸运的是，不需要这些方法返回的URL对象实际使用jar方案。但是，它们需要通过这些URL对象加载存储的类或资源文件的内容。

java.security.CodeSource API和安全策略文件使用URL来命名要授予指定权限的代码库的位置。需要特定权限的运行时系统的组件目前通过文件URL在lib / security / java.policy文件中标识。椭圆曲线密码学提供者例如被识别为

```
file:${java.home}/lib/ext/sunec.jar
```
这显然在模块化图像中没有意义。

IDE和其他类型的开发工具需要能够枚举存储在运行时映像中的类和资源文件，并读取其内容。今天他们经常通过打开和读取rt.jar和类似的文件来直接做到这一点。当然，这将不可能通过模块化的图像。

命名存储的模块，类和资源的新URI方案

为了解决上述三个问题，我们建议定义一个新的URL方案jrt，用于命名存储在运行时映像中的模块，类和资源，而不会显示图像的内部结构或格式。

jrt URL是根据RFC 3986的语法分层URI

```
jrt:/[$MODULE[/$PATH]]
```
其中$ MODULE是可选的模块名称，$ PATH（如果存在）是该模块中特定类或资源文件的路径。 jrt URL的含义取决于其结构：

jrt:/$MODULE/$PATH指的是给定$ MODULE中名为$ PATH的特定类或资源文件。

jrt:/$MODULE指模块$ MODULE中的所有类和资源文件。

jrt:/指的是存储在当前运行时映像中的类和资源文件的整个集合。

这三种形式的jrt URL解决了以下问题：

现在返回jar URL的API现在将返回jrt URL。以上调用ClassLoader :: getSystemResource，例如现在将返回URL

```
jrt:/java.base/java/lang/Class.class
```
将定义jrt方案的内置协议处理程序，以便这些URL对象的getContent方法检索命名类或资源文件的内容。

CodeSource API的安全策略文件和其他用途可以使用jrt URL来命名特定的模块，以授予权限。例如，椭圆曲线加密提供者现在可以由jrt URL来标识

```
jrt:/jdk.crypto.ec
```
当前授予所有权限但实际上不需要它们的其他模块可以平凡地享有de权限，即准确地给予它们所需的权限。

将为jrt URL方案定义内置的NIO FileSystem提供程序，以便开发工具可以通过加载由URL jrt：/命名的FileSystem来枚举和读取运行时映像中的类和资源文件，如下所示：

```
FileSystem fs = FileSystems.getFileSystem(URI.create("jrt:/"));
byte[] jlo = Files.readAllBytes(fs.getPath("java.base",
                                           "java/lang/Object.class"));
```
对于支持JDK 9代码开发但是在JDK 8上运行的代码的工具，适用于JDK 8的此文件系统提供者的副本将被放置在JDK 9运行时映像的根目录中，名为JRT-fs.jar。

（jrt URL协议处理程序不会返回第二和第三个表单的URL的任何内容。）

##### 构建系统更改

构建系统将被修改以产生上述新的运行时映像格式。我们也将借此机会，最后，将image / j2sdk-image，images / j2re-image和images / j2re-compact $ N-images目录重命名为images / jdk，images / jre和images / jre-紧凑$ N。

##### 次要规格变化

在JDK 8中实现的JEP 162进行了一些更改，以准备Java SE平台和JDK，用于此处和相关JEP中提出的模块化工作。在这些更改中，删除了需要在运行时映像的lib目录中查找某些配置文件的规范规范语句，因为这些文件现在将被放置在conf目录中。大多数具有此类语句的SE专用API作为Java SE 8的一部分进行了修改，但是跨越Java SE和EE平台共享的一些API仍然包含以下语句：

javax.xml.stream.XMLInputFactory指定$ {java.home} /lib/stax.properties（JSR 173）。

javax.xml.ws.spi.Provider指定$ {java.home} /lib/jaxws.properties（JSR 224）。

javax.xml.soap.MessageFactory和相关类，指定$ {java.home} /lib/jaxm.properties（JSR 67）。

这些语句将被修改，以便不像现在那样授权lib目录。

##### 开放式问题

可能还需要对字体配置进行一些更改。

lib/security目录仍然包含两个jar文件，其内容只是本地和US-export加密策略文件;我们打算用它们的内容替换这些jar文件（8061842）。

lib/$ARCH目录仅存在于Linux和Solaris版本中。这是一种残留的图像残留，可以支持多种CPU架构，这不再需要。我们将调查其内容是否可以直接放在lib目录中，就像Mac OS和Windows一样，在这种情况下，将不再需要lib/$ARCH目录（8066474）。 （不再是公开的问题;不再有lib/$ARCH目录）

演示，样本和目录的内容应理想地从适当的模块中导出;我们将调查如何最好地做到这一点（8066476）。

javax.activation.MailcapCommandMap类和相关类指定${java.home}/lib/mailmap（JSR 925）;这需要修改，以免不要使用lib目录。

##### 测试

一些现有的测试直接使用运行时映像内部（例如rt.jar）或引用不再存在的系统属性（例如java.ext.dirs）。这些测试将被修复。

我们计划发布包含这些更改的早期访问构建，然后鼓励更广泛的Java社区的成员针对这些构建来测试其工具，库和应用程序，以帮助挑选任何剩余的兼容性问题。

风险与假设

这个提案的核心风险是兼容性，总结如下：

如上所述，JDK映像将不再包含jre子目录。假定存在该目录的现有代码可能无法正常工作。

如上所述，不再定义系统属性java.endorsed.dirs和java.ext.dirs。假定这些属性具有非空值的现有代码可能无法正常工作。

如上所述，JDK和JRE映像将不再包含lib / rt.jar，lib / tools.jar，lib / dt.jar和其他内部jar文件的文件。假定存在这些文件的现有代码可能无法正常工作。

如上所述，现有的返回URL对象以在运行时映像中命名类和资源文件的标准API现在返回jrt URL。预期这些API返回jar URL的现有代码可能无法正常工作。这样的代码例如已经在Glassfish应用服务器中找到。

内部系统属性sun.boot.class.path将不再命名一系列jar文件和目录。取决于此属性的现有代码可能无法正常工作。

以前在lib / tools.jar中找到的类和资源文件，只有当该文件被添加到类路径中时才可见，在JDK映像中，通过系统类加载器或某些情况下可以看到引导类加载器。但是，包含这些文件的模块在应用程序类路径中不会被提及，即系统属性java.class.path的值。

以前在lib / dt.jar中找到的类和资源文件，只有当该文件添加到类路径时才可见，并且可以通过引导类加载器可见，并且存在于JRE和JDK中。

某些现有软件包中类型的定义类加载器将会更改。对这些类型的类加载器进行假设的现有代码可能无法正常工作。具体的变化是：

```
Package                          Old loader     New loader
-----------------------------    -----------    -----------
com.sun.crypto                   extension      boot
com.sun.jndi.dns                 boot           extension
com.sun.jndi.url.dns             boot           extension
com.sun.tools.corba.se.idl       application    boot
com.sun.tools.script             application    boot
com.sun.tracing                  boot           application
sun.security.tools.policytool    boot           application
sun.tools.jar                    boot           application
sun.tracing.dtrace               boot           application
```
这些更改是包含API和工具的组件模块化的方式的结果。这种组件的类历史上是在rt.jar和tools.jar之间分割的，但现在所有这些类都将在一个单独的模块中。

JRE映像中的bin目录将包含以前仅在JDK映像中发现的几个命令，即appletviewer，idlj，java-rmi.cgi，jrunscript和jstatd。与上一个项目一样，这些更改是将包含API和工具的组件模块化的方式的结果。

在抽象中确定这些变化的全部影响是不可能的。因此，我们必须依靠广泛的内部和外部测试。复杂的应用程序（如IDE）比直接库和更简单的应用程序更容易受到这些更改的影响。如果其中的一些变化被证明是对开发人员，部署者或最终用户来说是不可逾越的障碍，那么我们将会调查如何减轻其影响。

##### 依赖性

该JEP是项目拼图的四个JEP中的三分之一。这取决于JEP 201，它将JDK源代码重新组合成模块，并将构建系统升级到编译模块。它还取决于JEP 162中早期的准备工作，在JDK 8中实施。

#### JEP 221: New Doclet API
##### 概要

提供替代Doclet API以利用适当的Java SE和JDK API，并更新标准doclet以使用新的API。

##### 注意

在本文档中，术语“旧Doclet API”是指com.sun.javadoc中的API，“旧标准doclet”是指com.sun.tools.doclets.standard.Standard。

术语“新Doclet API”是指jdk.javadoc.doclet中的API，“新标准doclet”是指jdk.javadoc.doclet.StandardDoclet。

##### 目标

减少过时API的维护负担。

消除使用自定义语言模型API，以支持Java SE 6中引入的标准语言模型API javax.lang.model。

消除对分析文档注释的简单支持，有利于JDK 8中引入的Compiler Tree API，com.sun.source.doctree。

用合适的新界面类型替换“模板类”com.sun.javadoc.Doclet的使用。

##### 非目标

虽然提高性能不是一个目标，但是由于这项工作，预计将会改进javadoc工具和新的标准doclet的性能。

##### 动机

旧的Doclet API有以下问题需要解决。

API指定一个doclet只是一个实现一些或全部静态方法的类，如模板类com.sun.javadoc.Doclet所示。使用静态方法是特别麻烦的，因为它们需要使用静态成员在方法之间共享数据。这对并发使用和测试都有负面影响。

API提供了自己的语言模型API，它具有许多限制（例如，数组不是很好地建模），而随着Java语言以影响API签名的方式发展，这是一个更新的负担（例如泛型，类型注释和默认方法。）

API提供了对分析文档注释内容的最小，低效和不完全指定的支持。这对任何想要处理评论内容的文档来说都是一个重大的负担。 API还不支持确定注释中的构造的包含源文件中的位置。这使得无法为应报告的任何诊断提供准确的位置信息。分析评论的支持是由旧的标准doclet中的一个糟糕而且效率低下的实现所支持，旧的标准doclet在很大程度上依赖于使用子串匹配来分隔注释中的结构。

##### 描述

新的Doclet API在jdk.javadoc.doclet包中声明。它使用语言模型API和编译器树API。

更新javadoc工具以识别针对新的Doclet API编写的doclet。旧的Doclet API将被支持用于过渡目的，并且将被冻结，也就是不被更新以支持在过渡期间引入的任何新的语言特征。

现有的标准doclet支持称为Taglet API的辅助插件API。 Taglets提供了用户定义可以在文档注释中使用的自定义标签的能力，并指定了这些标签在生成的文档中的显示方式。更新的标准doclet支持更新的taglet API。

##### 风险与假设

此外，目前支持的API的旧Doclet API已被弃用，可能会在将来的版本中删除。鼓励旧Doclet API的用户迁移其代码以使用新的Doclet API。

已知有一些现有的用户编写的doclet可以直接引用旧的“标准doclet”中的代码，即使该代码不是（并且从未有）受支持的接口。由于该代码难以维护和更新，特别是对于最近的新语言功能，旧的“标准doclet”已被弃用，以在JDK 9中进行删除，并将在以后的版本中删除。

##### 测试

已经针对Doclet API和标准doclet的现有测试集进行了测试，以测试新的API和新的标准doclet。添加了额外的测试来覆盖边缘案例。

#### JEP 222: jshell: The Java Shell (Read-Eval-Print Loop)
##### 概要

提供交互式工具来评估Java编程语言的声明，语句和表达式以及API，以便其他应用程序可以利用此功能。

##### 目标

JShell API和工具将提供一种在JShell状态下交互评估Java编程语言的声明，语句和表达式的方法。 JShell状态包括不断发展的代码和执行状态。为了便于快速调查和编码，语句和表达式不需要在方法中发生，变量和方法不需要在类中发生。

jshell工具将是一个具有简化交互功能的命令行工具，包括：编辑历史记录，制表完成，自动添加所需的终端分号以及可配置的预定义导入和定义。

##### 非目标

新的互动语言不是目标：所有接受的输入必须与Java语言规范（JLS）中的语法生产相匹配。此外，在适当的周围环境中，所有接受的输入必须是有效的Java代码（JShell将自动提供周围的上下文 - “包装”）。也就是说，如果X是JShell接受的输入（与拒绝的错误相反），则存在A和B，使得AXB是Java编程语言中的有效程序。

超出范围是图形界面和调试器支持。 JShell API旨在允许IDE和其他工具中的JShell功能，但是jshell工具并不是一个IDE。

##### 动机

学习编程语言及其API时，立即反馈很重要。学校引用Java作为教学语言的第一个原因是其他语言具有“REPL”，并且具有远低于最初的“Hello，world！”的条形码。程序。读取评估打印循环（REPL）是一种交互式编程工具，循环，连续读取用户输入，评估输入，打印输入的值或对输入引起的状态变化的描述。 Scala，Ruby，JavaScript，Haskell，Clojure和Python都有REPL并且都允许小的初始程序。 JShell将REPL功能添加到Java平台。

编码选项的探索对于开发人员原型代码或调查新的API也很重要。在这方面，互动评估比编辑/编译/执行和System.out.println更有效率。

没有Foo类的公开（public static void main（String [] args）{...}}，学习和探索精简。

##### 描述

##### 功能

JShell API将提供所有JShell的评估功能。输入到API的代码片段被称为“片段”。 jshell工具还将使用JShell完成API来确定何时输入不完整（并且用户必须被提示输入更多），如果添加了分号（在这种情况下该工具将附加分号），则完成在使用选项卡请求完成时，如何完成输入。该工具将具有一组用于查询，保存和恢复工作以及配置的命令。命令将与主要的斜杠与片段区分开来。

##### 文档

JShell模块API规范可以在这里找到：

http://download.java.net/java/jdk9/docs/api/jdk.jshell-summary.html
其中包括主JShell API（jdk.jshell包）规范：

http://download.java.net/java/jdk9/docs/api/jdk/jshell/package-summary.html
jshell工具参考：

https://docs.oracle.com/javase/9/tools/jshell.htm
是Java平台标准版工具参考的一部分：

https://docs.oracle.com/javase/9/tools/tools-and-command-reference.htm

##### 条款

在本文档中，术语“类”是指Java虚拟机规范（JVMS）中使用的意义，其中包括Java语言规范（JLS）类，接口，枚举和注释类型。如果意图不同，文本将会明确。

##### 片段

代码片段必须符合以下JLS语法生成之一：

表达
声明
ClassDeclaration
InterfaceDeclaration
MethodDeclaration
FieldDeclaration
导入声明
在JShell中，“变量”是一个存储位置，并具有关联的类型。使用FieldDeclaration代码片段显式创建变量：

```
int a = 42;
```
或隐含地由表达式（见下文）。变量具有少量的字段语义/语法（例如，允许volatile修饰符）。但是，变量没有用户可见的类包围它们，并且通常会像局部变量一样被查看和使用。

所有表达式都被接受为片段。这包括没有副作用的表达式，如常量，变量访问和lambda表达式：

```
1
a
2+2
Math.PI
x -> x+1
(String s) -> s.length()
```

以及具有副作用的表达式，如作业和方法调用：

```
a = 1
System.out.println("Hello world");
new BufferedReader(new InputStreamReader(System.in))
```
某些形式的表达片段隐式地创建一个变量来存储表达式的值，以便以后可以被其他代码片段引用。 默认情况下，隐式创建的变量名称为$ X，其中X是代码段标识符。 如果表达式为void（println示例），或者表达式的值可以由简单名称引用（如上述'a'和'a = 1'），则不会隐式创建变量。 所有其他示例都有为其隐式创建的变量）。

所有的声明都被接受为片段，除了'break'，'continue'和'return'。 但是，代码段可能包含'break'，'continue'或'return'语句，在那里它们符合用于封闭上下文的Java编程语言的通常规则。 例如，此代码段中的return语句是有效的，因为它包含在lambda表达式中：

```
() -> { return 42; }
```
声明片段（ClassDeclaration，InterfaceDeclaration，MethodDeclaration或FieldDeclaration）是一个明确引入可由其他片段引用的名称的片段。声明片段遵循以下规则：

访问修饰符（public，protected和private）将被忽略（所有其他代码段都可访问所有声明片段）
修饰符final被忽略（将来的更改/继承被允许）
修饰符static被忽略（没有用户可见的包含类）
默认和同步的修饰符是不允许的
修饰符抽象只允许在类上。
除了ImportDeclaration形式的所有片段，都可能包含嵌套声明。例如，作为类实例创建表达式的片段可以使用嵌套方法声明指定匿名类体。 Java编程语言的通常规则适用于嵌套声明的修饰符，而不是上述规则。例如，下面的类代码段被接受，并且嵌套方法声明中的私有修饰符被遵从，因此片段new C().secret()”将不被接受：

```
class C {
  int answer() { return 2 * secret(); }
  private int secret() { return 21; }
}
```
代码段不能声明包或模块。所有JShell代码都放在一个未命名模块中的单个包中。该包的名称由JShell控制。

在jshell工具中，如果分号是输入的最后一个字符（不包括空格和注释），则可以省略片段的终端分号。

##### 状态

JShell状态在JShell的一个实例中。使用eval（...）方法在JShell中评估片段，产生错误，声明代码或执行语句或表达式。在具有初始化器的变量的情况下，发生声明和执行。 JShell的一个实例包含先前定义和修改的变量，方法和类，先前定义的导入声明，先前输入的语句和表达式（包括变量初始化器）的副作用和外部代码库。

##### 修改

由于所需的用途是探索，所以声明（变量，方法和类）必须能够随着时间而发展，同时保留评估数据。一个选择是在一些或所有情况下，将改变的声明变成新的附加实体，但这肯定会令人困惑，并且不能很好地探索声明之间的相互作用。在JShell中，每个唯一的声明密钥在任何给定的时间只有一个声明。对于变量和类，唯一的声明键是名称，方法的唯一声明键是名称和参数类型（允许重载）。由于这是Java，变量，方法和类都有自己的名称空间。

##### 前转参考

在Java编程语言中，在一个类的主体内，对于稍后出现的成员的引用可能会出现;这是一个向前参考。随着在JShell中依次输入和评估代码，这些引用将暂时解决。在某些情况下，例如相互递归，需要转发参考。这也可能发生在探索性编程中，同时输入代码，例如，意识到应该调用另一个（迄今未被写的）方法。 JShell支持方法体中的转发引用，返回类型和参数类型，变量类型以及类中的引用。由于语义要求它们立即执行，所以不支持在变量初始化器中转发引用。

##### 代码段依赖关系

代码状态保持最新和一致;也就是说，当评估片段时，对依赖片段的任何更改都会立即传播。

当代码段被成功声明时，声明将是三种：添加，修改或替换。如果是该密钥的第一个声明，则会添加一个代码段。如果代码片段与以前的代码片段匹配，则代码段被替换，但是它们的签名不同。如果代码段的键匹配以前的代码片段并且其签名匹配，则该代码段被修改;在这种情况下，不会影响依赖片段。在“已修改”和“替换”的情况下，以前的代码段不再是代码状态的一部分。

当片段被添加时，它可能会提供未解析的参考。替换代码片段时，可能会更新现有的代码段。例如，如果一个方法的返回类型被声明为C类，然后C类被替换，那么方法的签名已经改变，并且该方法必须被替换。注意：这可能会导致先前有效的方法或类无效。

希望用户数据尽可能持久。这是可以实现的，除了变量Replace的情况。当变量被用户直接或间接地通过依赖更新替换时，该变量被设置为其默认值（null，因为这只能与引用变量一起出现）。

当声明无效时，由于通过更新导致转发引用或变为无效，则声明为“corralled”。可以在其他声明和代码中使用被声明的声明，但是如果尝试执行它，将会发生运行时异常，这将解释未解析的引用或其他问题。

##### 包装

在Java编程语言中，变量，方法，语句和表达式必须嵌套在其他构造中，最终是嵌套在一个类中。当JShell的实现将一个变量，方法，语句和表达式代码片段编译为Java代码时，需要一个人工上下文，如下所示：

变量，方法和类
作为合成类的静态成员
表达和声明
作为合成类中合成静态方法中的表达式和语句
这种包装也可以使代码段更新，因此，请注意，代码段类也包含在合成类中。

##### 模块化环境配置

jshell工具有以下选项来控制模块化环境：

--module路径
--add模块
--add出口
也可以通过直接添加到编译器和运行时选项来配置模块化环境。可以使用-C选项添加编译器标志。可以使用-R选项添加运行时标志。

所有的jshell工具选项都记录在“工具参考”中（见上文）。

可以使用JShell.Builder上的compilerOptions和remoteVMOptions方法在API级别配置模块化环境。

JShell未命名模块读取的模块集与JEP 261“根模块”所建立的未命名模块的默认组件模块相同：

http://openjdk.java.net/jeps/261

##### 命名

模
	jdk.jshell
工具启动器
	jshell
API包
	jdk.jshell
SPI封装
	jdk.jshell.spi
执行引擎“库”包
	jdk.jshell.execution
工具启动API包
	jdk.jshell.tool
工具实现包
	jdk.internal.jshell.tool
OpenJDK项目
	库拉

##### 备择方案

一个更简单的选择只是提供一个批处理脚本包装器，无需交互/更新支持。

另一个替代方案是维护现状：使用其他语言或使用第三方REPL（如BeanShell），尽管该特定的REPL已经休眠多年，它基于JDK 1.3，并对该语言进行任意更改。

许多IDE，例如NetBeans调试器和BlueJ的CodePad，提供交互式评估表达式的机制。保留的上下文和代码保持基于类，并且不支持方法粒度。他们使用特制解析器/口译员。

##### 测试

API有助于进行详细的点测试。一个测试框架使书写测试顺利进行。

由于该工具的评估和查询功能是基于API构建的，因此大多数测试都是API。然而，该工具的命令测试和完整性测试也是必需的。该工具是用于测试线束的钩子，用于工具测试。

测试由三部分组成：

测试API。这些测试包括正面和负面情况。每个公共方法都必须包括添加变量，方法和类，重新定义等的测试。

测试jshell工具。这些测试检查jshell命令和编译以及Java代码的执行是否具有正确的行为。

压力测试。为了确保JShell可以编译所有允许的Java代码段，将使用JDK本身中的正确的Java代码。这些测试解析源，将代码块提供给API，并测试API的行为。

##### 依赖性

该实施将尽一切努力利用JDK中现有语言支持的准确性和工程设计。 JShell状态被建模为JVM实例。代码分析和可执行代码（jdk.jshell API）的生成将由Java编译器（javac）通过编译器API执行。代码替换（jdk.jshell.execution）将使用Java调试接口（JDI）。

原始片段（即，未包装的片段）的解析将使用Compiler API与解析器的小子类进行，以允许原始片段。生成的信息将用于将代码段包装到有效的编译单元中，包括用于先前评估的代码的导入的类声明。类文件的进一步分析和生成将使用Java编译器的未修改实例完成。生成的类文件将保存在内存中，而不会写入存储。存在用于配置执行引擎的jdk.jshell.spi SPI。默认执行引擎的行为如下。类文件将通过套接字发送到远程进程。远程代理将处理加载和执行。替换将通过JDI VirtualMachine.redefineClasses（）工具完成。

制表完成分析（jdk.jshell API）也将使用编译器API。完成检测将使用javac lexer，自定义和表驱动代码。

jshell工具（jdk.internal.jshell.tool）将使用“jline2”进行控制台输入，编辑和历史记录。 jline2已经私下卷入JDK。

#### JEP 223: New Version-String Scheme
##### 概要

定义一个可以轻松区分主要，次要和安全更新版本的版本字符串方案，并将其应用于JDK。

##### 目标

人类易于理解，易于程序解析。

符合当前的行业惯例，特别是语义版本控制。

可以由现有的包装系统和平台部署机制（包括RPM，dpkg，IPS和Java网络启动协议（JNLP））来采用。

消除当前在版本字符串的一个元素中编码两种类型的信息的做法，即次要版本号和安全级别，这很难解密，并导致跳过许多版本号。

为版本字符串解析，验证和比较提供简单的API。

##### 非目标

在此JEP的目标版本之前，请更改任何版本所使用的版本字符串格式。
##### 动机

哪个版本包含所有最新的安全修复程序：JDK 7 Update 55或JDK 7 Update 60？

看起来像JDK 7 Update 60是五十五个发布晚于Update 55，所以它必须包括更多的安全修复，对吧？

可悲的是，这个结论是错误的：这两个版本都包含完全相同的安全修复程序。要了解此答案，您首先需要了解JDK Update版本的当前编号方案。包含超出安全修复程序的更改的次要版本是20的倍数。基于以前的次要版本的安全性版本是奇数增加5，如果需要，增加六个，以保持更新号奇数。要了解一个次要版本是否比较早版本更安全，最终需要查看发行说明或源代码。

名为“JDK 7 Update 60”，“1.7.0_60”和“JDK 7u60”的版本有什么区别？

这些只是同一版本的不同名称。这些差异使得难以识别和验证等效的版本。解析令牌序列的简单点对比是不够的;相反，需要一个相当复杂的算法。使用小写字母“u”不是行业标准，不是语言中立的。

一个更简单，更直观的版本控制方案已经很久了。

##### 描述

##### 版本号

版本号$ VNUM是由句点字符（U + 002E）分隔的元素的非空序列。一个元素是零，或者一个无符号整数，不带前导零。版本号中的最后一个元素不能为零。格式为：

```
[1-9][0-9]*((\.0)*\.[1-9][0-9]*)*
```
序列可以是任意长度，但前三个元素被赋予具体含义，如下所示：

```
$MAJOR.$MINOR.$SECURITY
```

$MAJOR ---主要版本号，增加了主要版本，其中包含新版本的Java SE平台规范中指定的重要新功能，例如Java SE 8的JSR 337功能可能会在主要版本中删除，提前通知至少一个主要版本，并且在合理的情况下可能会发生不相容的更改。 JDK 8的$ MAJOR版本号是8; JDK 9的$ MAJOR版本号为9.当$ MAJOR递增时，所有后续元素都将被删除。

$MINOR ---次要版本号，可能包含兼容的错误修复的小型更新版本，由相关平台规范的维护版本规定的标准API的修订以及该规范范围之外的实现功能，如新的JDK特定的API，其他服务提供商，新的垃圾收集器和新的硬件体系结构的端口。

$SECURITY ---对于包含关键修复程序的安全更新版本，包括为提高安全性所需的修补程序而言，安全级别增加。 $ MINOR递增时$ SECURITY不会重置为零。因此，对于给定的$ MAJOR值，$ SECURITY的值越高，始终表示更安全的版本，无论$ MINOR的值如何。

版本号的第四个和后来的元素可供JDK代码库的下游用户使用。除了相应的安全版本中的安全修复之外，这样的消费者可以例如使用第四个元件来识别包含少量关键的非安全性修补程序的修补程序版本。

版本号不包括尾随零元素;即如果$ SECURITY值为零，则省略$ SECURITY，如果$ MINOR和$ SECURITY均为零，则省略$ MINOR。

将版本号中的数字序列与数字，逐点方式中的另一个这样的序列进行比较;例如，9.9.1小于9.10.3。如果一个序列比另一个序列短，则较短序列的丢失元素被认为小于较长序列的相应元素;例如，9.1.2小于9.1.2.1。

##### 版本字符串

版本字符串$ VSTR由版本号$ VNUM组成，如上所述，可选地遵循以下格式之一的预发布和构建信息：

$VNUM(-$PRE)?\+$BUILD(-$OPT)?
$VNUM-$PRE(-$OPT)?
$VNUM(+-$OPT)?
哪里：

$PRE，匹配（[a-zA-Z0-9]+）---预发行标识符。通常情况下，对于内部开发人员构建的早期访问版本，它们处于积极的开发阶段，并且可能不稳定或内部。

当比较两个版本字符串时，具有预发行标识符的字符串总是小于等于$VNUM但没有这样的标识符的字符串。预发行标识符仅在数字组成时才进行数字比较，而按字典顺序排列。数字标识符被认为小于非数字标识符。

```
$BUILD，matching（0|[1-9][0-9]*）---构建号，每增强构建增加。当$VNUM的任何部分增加时，$BUILD将重置为1。
```
当比较两个版本字符串相等的$VNUM和$PRE组件时，没有$BUILD组件的字符串总是小于一个具有$BUILD组件的字符串;否则，$BUILD数字被数字比较。

```
$OPT，匹配（[-a-zA-Z0-9\。]+）---附加构建信息，如果需要。在内部构建的情况下，这通常会包含构建的日期和时间。
```
当比较两个版本字符串时，根据所选择的比较方法，$OPT的值（如果存在）可能是也可能不重要。

版本号10-ea匹配$VNUM=“10”和$PRE=“ea”。版本号10+-ea匹配$VNUM=“10”和$OPT=“ea”。

下表比较了JDK9的潜在版本字符串，使用现有的和提出的格式：

```
Existing                Proposed
Release Type    long           short    long           short
------------    --------------------    --------------------
Early Access    1.9.0-ea-b19    9-ea    9-ea+19        9-ea
Major           1.9.0-b100      9       9+100          9
Security #1     1.9.0_5-b20     9u5     9.0.1+20       9.0.1
Security #2     1.9.0_11-b12    9u11    9.0.2+12       9.0.2
Minor #1        1.9.0_20-b62    9u20    9.1.2+62       9.1.2
Security #3     1.9.0_25-b15    9u25    9.1.3+15       9.1.3
Security #4     1.9.0_31-b08    9u31    9.1.4+8        9.1.4
Minor #2        1.9.0_40-b45    9u40    9.2.4+45       9.2.4
```
作为参考，此表显示了新格式的版本字符串，因为它们假定将用于某些JDK 7更新和安全性版本：

```
Actual               Hypothetical
Release Type        long           short    long          short
------------        --------------------    -------------------
Security 2013/04    1.7.0_21-b11    7u21    7.4.10+11    7.4.10
Security 2013/06    1.7.0_25-b15    7u25    7.4.11+15    7.4.11
Minor    2013/09    1.7.0_40-b43    7u40    7.5.11+43    7.5.11
Security 2013/10    1.7.0_45-b18    7u45    7.5.12+18    7.5.12
Security 2014/01    1.7.0_51-b13    7u51    7.5.13+13    7.5.13
Security 2014/04    1.7.0_55-b13    7u55    7.5.14+13    7.5.14
Minor    2014/05    1.7.0_60-b19    7u60    7.6.14+19    7.6.14
Security 2014/07    1.7.0_65-b20    7u65    7.6.15+20    7.6.15
```

##### 从版本号中删除初始的1个元素

该提案从JDK版本号中删除初始的1个元素。也就是说，它表明JDK 9的第一个版本的版本号为9.0.0而不是1.9.0.0。

近二十年后，很明显，当前版本号码方案的第二个元素是JDK的实际$ MAJOR版本号。当我们添加重要的新功能时，以及当我们进行不兼容的更改时，我们会增加该元素。

我们可以开始将当前方案的初始元素作为$ MAJOR版本号来处理，但是即使每个人都将其称为“JDK 9”，JDK 9也将具有版本号2.0.0。这不会帮助任何人。

如果我们保留初始的1，那么JDK版本号将继续违反语义版本控制的原则，而Java新手的开发人员将继续困惑于例如1.9和9之间的区别。

丢弃初始化有一些风险1.有很多方法可以比较版本号;有些人会正常工作，而有些则不会。

现有代码通过解析其元素并将其数字比较来比较版本号将继续工作，因为九个大于1;即9.0.0将被认为晚于1.8.0。

当初始元素值为1时，跳过初始元素的现有代码也将继续工作，因为在新方案中，初始元素将永远不会有该值。

然而，假设初始元素具有值1，因此在比较版本号时总是跳过第二个元素的现有代码将无法正常工作;例如，这样的代码将考虑9.0.1在1.8.0之前。

轶事证据表明，第三类中的现有代码不是很常见，但我们欢迎数据相反。

API

将定义一个用于解析，验证和比较版本字符串的简单Java API（8072379,8144062）：

```
package java.lang;

import java.util.Optional;

public class Runtime {

    public static Version version();

    public static class Version
        implements Comparable<Version>
    {

        public static Version parse(String);

        public int major();
        public int minor();
        public int security();

        public List<Integer> version();
        public Optional<String> pre();
        public Optional<Integer> build();
        public Optional<String> optional();

        public int compareTo(Version o);
        public int compareToIgnoreOpt(Version o);

        public boolean equals(Object o);
        public boolean equalsIgnoreOpt(Object o);

        public String toString();
        public int hashCode();
    }
}
```

将定义一个等效的C API，最有可能是修改后的jvm_version_info结构。

检查和比较JDK版本字符串的JDK中的所有代码将被更新以使用这些API。 图书馆或应用程序检查和比较JDK版本字符串的开发人员将被鼓励使用这些API。

##### 系统属性

由以下系统属性返回的值由此JEP修改。 一般语法如下：

```
Name                            Syntax
------------------------------  --------------
java.version                    $VNUM(\-$PRE)?  
java.runtime.version            $VSTR
java.vm.version                 $VSTR
java.specification.version      $VNUM
java.vm.specification.version   $VNUM
```

系统属性java.class.version不受影响。

下表显示了不同版本类型的现有值和建议值：

```
System Property                   Existing      Proposed
-------------------------------   ------------  --------
Early Access 
  java.version                    1.9.0-ea      9-ea
  java.runtime.version            1.9.0-ea-b73  9-ea+73
  java.vm.version                 1.9.0-ea-b73  9-ea+73
  java.specification.version      1.9           9
  java.vm.specification.version   1.9           9

Major (GA)
  java.version                    1.9.0         9
  java.runtime.version            1.9.0-b100    9+100
  java.vm.version                 1.9.0-b100    9+100
  java.specification.version      1.9           9
  java.vm.specification.version   1.9           9

Minor #1 (GA)
  java.version                    1.9.0_20      9.1.2
  java.runtime.version            1.9.0_20-b62  9.1.2+62
  java.vm.version                 1.9.0_20-b62  9.1.2+62
  java.specification.version      1.9           9
  java.vm.specification.version   1.9           9

Security #1 (GA)
  java.version                    1.9.0_5       9.0.1
  java.runtime.version            1.9.0_5-b20   9.0.1+20
  java.vm.version                 1.9.0_5-b20   9.0.1+20
  java.specification.version      1.9           9
  java.vm.specification.version   1.9           9
```

请注意，历史上检测到的所有代码。 在任何这些系统属性作为版本识别的一部分将需要检查和潜在的修改。 例如，System.getProperty（“java.version”）。indexof（'。'）将为主要版本返回-1。

##### 发射台

在OpenJDK java启动器实现中，系统属性用于报告版本信息，例如 java -version，java -fullversion和java -showversion。

发射器输出继续依赖于系统属性，如下所示：

```
$ java -version
openjdk version \"${java.version}\"
${java.runtime.name} (build ${java.runtime.version})
${java.vm.name} (build ${java.vm.version}, ${java.vm.info})

$ java -showversion < ... >
openjdk version \"${java.version}\"
${java.runtime.name} (build ${java.runtime.version})
${java.vm.name} (build ${java.vm.version}, ${java.vm.info})
[ ... ]

$ java -fullversion
openjdk full version \"${java.runtime.version}\"
```

实现细节可以在源中找到。

##### @since JavaDoc标签

@since JavaDoc标签的值将继续与系统属性java.specification.version对齐;因此，新的JDK 9 API将由@since 9指示。

##### Mercurial变更集标签

Mercurial标签用于识别促销的变更集。将增强用于验证推送到JDK发行版的所有更改集的代码工具jcheck等工具，以支持使用新版本方案的标签。

Mercurial标签的一般语法是 jdk\-$VNUM\+$BUILD。下表显示了不同版本类型的建议值：

```
Release Type      Proposed
----------------  -----------
Major (GA)        jdk-9+100
Minor #1 (GA)     jdk-9.1.2+27
Security #1 (GA)  jdk-9.0.1+3
```
一些工具可能需要支持现有和建议的标签格式。

##### 测试

更改版本字符串的语法和语义将需要对所有组件区域进行大量测试。独立于JDK版本字符串的现有测试应继续通过。

#### JEP 224: HTML5 Javadoc
##### 概要

增强javadoc工具来生成HTML5标记。

##### 目标

为标准doclet提供一个选项，以请求HTML 4或HTML5输出。 HTML5标记是语义的，即将意义与风格和内容分开。使用HTML5标记的标准doclet生成的页面部分满足辅助功能要求。

##### 非目标

替换生成的HTML页面的当前三帧/非帧结构是不是一个目标;这可能是未来JEP的主题。

授权在文档评论中使用HTML5不是一个目标，也不是提供将使用HTML 4编写的文档评论转换为HTML5的功能的目标。

##### 动机

在JDK 8及更早版本中，标准doclet生成了HTML 4.01中的页面，这是一个旧的标准，不能提供满足辅助功能要求的支持。
HTML5是HTML的最新标准。 HTML5增加了网页的语义价值，使得创建可访问的网页变得更加容易。

##### 描述

将一个命令行选项添加到标准doclet中以请求特定类型的输出标记。当前类型的HTML4将是默认值。 HTML5将成为JDK 10中的默认值。

通过使用结构化的HTML5元素（如页眉，页脚，导航等），可以改善生成的HTML的语义价值。

HTML5标记实现了可访问性的WAI-ARIA标准。使用role属性将特定角色分配给HTML文档中的元素。

-Xdoclint功能是根据请求的输出标记类型更新检查常见错误的文档注释。

##### 测试

提供测试以确保：

HTML5标记有效
HTML5标记可以访问
新的命令行选项工作正常
所有支持的浏览器都支持HTML5标记
缺乏用于测试可访问性的自动化工具妨碍了全面的可访问性测试。可以使用在线的基于Web的工具检查不同类型页面的代表性示例。

#### JEP 225: Javadoc Search
##### 概要

将一个搜索框添加到由标准doclet生成的API文档中，该文档可用于在文档中搜索程序元素和标记的单词和短语。搜索框出现在标准doclet生成的所有页面的标题中。

##### 目标

搜索功能在本地实现，不依赖于任何服务器端的计算资源。

##### 非目标

实现一般搜索引擎来搜索文档注释中的所有单词并不是一个目标。在这个JEP中更改一般的三帧/非帧布局或其内容也不是一个目标，尽管这可能在后续的工作中被考虑。

##### 动机

如果您不熟悉他们的布局，那么由标准doclet生成的API文档页面可能很难导航。
可以使用外部搜索引擎，但可能会导致过时或不相关的页面。可以使用浏览器的内置搜索功能，但是限于在当前页面中搜索而不是整个文档。

##### 描述

什么可以搜索？

模块，包，类型和成员的声明名称被索引和搜索。由于方法可以重载，方法参数类型的简单名称也被索引并可以搜索。方法参数名称不进行索引。

可以搜索使用新的内联标签@index索引的搜索字词或短语。其他内联标签不能嵌套在@index中。只能在声明的文档评论中标出@index的短语或搜索词，可以搜索。例如，整个java.lang.Math类中都使用域特定术语“ulps”，但不会出现在任何类或方法声明名称中。为了帮助Math API的用户，API设计器可以在类级别或方法级文档注释中标记各种“ulps”的出现。使用{@index ulps}实现标记。 “ulps”将被javadoc索引。

生成的索引的格式和位置可能随时间而变化，不能被其他工具所依赖。

##### 准备搜索

默认情况下，运行javadoc将生成一个索引，允许生成的HTML支持搜索框。客户端JavaScript用于产生搜索结果。 javadoc的-noindex选项可用于禁用索引和搜索功能。与所有可能生成或可能不生成的其他文件一样，javadoc不会删除输出目录中的任何过时的文件。在运行javadoc之前，用户有责任确保输出目录为空，以确保输出目录中以前的运行中没有过时的文件。

##### 如何搜索

生成的API页面上提供了一个搜索框，并提供：

可以根据用户输入调用搜索的设施。搜索输入支持骆驼案例搜索。例如，要搜索addFocusListener（）方法，用户可以在搜索框中输入“addFL”。搜索输入不支持正则表达式，但在后续工作中可能会考虑这一点。

结果，包括与输入的字符完全匹配的结果，后跟包含字符串中任何位置的输入字符的结果。多个结果显示为搜索框下方的简单滚动列表。结果被分类为“模块”，“包”，“类型”，“成员”和“搜索标签”，以便于分类和适当的用户选择;和

基于用户选择的页面重定向。

##### 图书馆

jQuery UI自动填充和JSZip作为实现的一部分，用于提供浏览器独立的自动完成实现。这是客户端功能。

##### 测试

提供测试以确保以下内容：

搜索索引的准确度
使用新的@index标签
自动完成选择和重定向的准确性
验证防止恶意注射

#### JEP 226: UTF-8 Property Resource Bundles
##### 概要

定义应用程序指定以UTF-8编码的属性文件的方法，并扩展ResourceBundle API以加载它们。

##### 动机

该平台长期以来提供了基于ISO-8859-1的属性文件格式，并为该编码中无法表示的字符提供了转义机制。该格式由标准资源束查找机制支持。如相关RFE所述，这种格式很难使用，因为它需要在直接编辑的字符编码之间的转义格式和文本之间进行连续转换。

##### 描述

更改ResourceBundle类的默认文件编码，以将属性文件从ISO-8859-1加载到UTF-8。通过这样做，应用程序不再需要使用转义机制转换属性文件。现有的属性文件很少受此更改的影响，因为ISO-8859-1的U + 0000-U + 007F与UTF-8兼容，代码点超过U + 00FF的字符应已被转义。如果在读取UTF-8中的属性文件，MalformedInputException或UnmappableCharacterException异常，则会从头重新读取属性文件，并恢复为使用ISO-8859-1编码。为了能够将ISO-8859-1属性文件识别为有效的UTF-8文件的罕见场合，该JEP提供了通过设置ISO-8859-1或UTF-8来明确指定编码的方法。系统属性“java.util.PropertyResourceBundle.encoding”。

#### JEP 227: Unicode 7.0
##### 概要

升级现有的平台API，以支持Unicode Standard版本7.0。

##### 目标

支持最新版本的Unicode，主要在以下类中：

java.lang包中的Character和String，以及
Bidi，BreakIterator和Normalizer在java.text包中。
非目标

这个JEP没有实现两个相关的Unicode规范：

UTS＃10，Unicode排序算法，和
UTS＃46，Unicode IDNA兼容性处理。
动机

Unicode是一个行业标准，因此Java支持最新版本很重要。

##### 描述

Java SE 8支持Unicode 6.2。

这种升级将包括对双向行为的改进，这使得在Unicode 6.3中引入的诸如阿拉伯语和希伯来语之类的语言能够更好地显示文本。 Unicode 7.0本身将增加约三千个字符和二十多个脚本。

##### 测试

我们需要验证相关类是否正确使用了最新的Unicode数据。

##### 风险与假设

如果在JDK 9时间段内发布了一个7.0以下的版本，那么这个JEP最有可能被更新以指定该版本。

##### 依赖性

此功能取决于Unicode Consortium的Unicode标准。

#### JEP 228: Add More Diagnostic Commands
##### 概要

定义其他诊断命令，以提高Hotspot和JDK的诊断能力。

##### 描述

这是新命令的列表（确切名称是TBD）：

##### print_class_summary

打印所有加载的类及其继承结构的列表。
负责组：运行时
print_codegenlist

使用C1或C2排队的打印方法（单独的队列）
负责组：编译器
##### print_utf8pool

打印所有UTF-8字符串常量。
负责组：运行时
##### datadump_request

指示JVM对JVMTI执行数据转储请求。
负责人：服务能力
##### dump_codelist

打印具有完整签名，地址范围和状态（活着，非入侵和僵尸）的n方法（编译）。
允许选择打印到stdout或文件。
允许XML或文本打印输出。
负责组：编译器
##### print_codeblocks

使用地址打印代码缓存的大小和代码缓存中的块列表。
负责组：编译器
##### set_vmflag

在VM或库中设置命令行标志/选项。
负责任团队：可维护性
##### 测试

需要对每个命令进行测试以验证输出。

#### JEP 229: Create PKCS12 Keystores by Default
##### 概要

将默认密钥库类型从JKS转换为PKCS12。

##### 目标

提高安全性PKCS12提供比JKS更强大的加密算法。

保持向前和向后的兼容性。访问JKS和PKCS12密钥库的应用程序必须在JDK版本中继续运行。

##### 动机

JKS是一种定制的JDK特定密钥库类型。 JDK 1.2以来，它一直是Java平台的默认密钥库类型。 JKS密钥库只能存储私有密钥和受信任的公钥证书，它们基于新的密码算法不易扩展的专用格式。

PKCS12是用于存储加密密钥的可扩展，标准和广泛支持的格式。从JDK 8开始，PKCS12密钥库可以存储私钥，可信公钥证书和密钥。切换到PKCS12可提高密钥库的完整性和机密性。它还为与其他支持PKCS12的系统的互操作性打开了机会。

##### 描述

此功能将默认密钥库类型从JKS更改为PKCS12。默认情况下，将以PKCS12密钥库格式创建新密钥库。现有密钥库不会更改，密钥库应用程序可以继续显式指定所需的密钥库类型。

现有应用程序不得中断。密钥库往往是长期存在的，所以我们需要支持多个JDK版本的访问。访问由早期JDK版本创建的密钥库的应用程序必须在JDK 9上运行不变。类似地，访问JDK 9创建的密钥库的应用程序应在早期的JDK版本上运行不变。

这个要求是通过引入一个了解JKS和PKCS12格式的密钥库检测机制来实现的。在加载密钥库的格式之前，请检查密钥库的格式以确定其类型，然后使用适当的密钥库实现来访问密钥库。该机制默认启用，但如果需要，可以禁用。

支持此密钥仓库检测机制可能会返回到较早版本的JDK版本。

##### 测试

需要在JDK版本中进行重大测试，以确保为访问密钥库的应用程序保持兼容性。

#### JEP 231: Remove Launch-Time JRE Version Selection
##### 概要

删除在JRE启动时请求JRE正在启动的JRE的版本。

##### 动机

“多重JRE”（“mJRE”）功能允许开发人员指定可以使用什么JRE版本或版本的范围来启动应用程序。版本选择条件可以在应用程序的jar文件（JRE-Version）的清单条目中指定，也可以作为java启动器的命令行选项（-version :)指定。如果启动的JRE的版本不符合条件，那么启动程序将搜索一个版本，如果它找到一个版本，那么它会启动该版本。

在实践中部署应用程序需要做的不仅仅是选择一个特定的JRE。现代应用程序是通过Java Web Start（JNLP），本机OS包装系统，或主动安装通常部署，所有这些技术都有自己的发现的方式，有时甚至安装和更新后，适当的JRE中的应用。

mJRE功能仅解决整个部署问题的一部分。再其次，当它在JDK 5引入没有完全记载：该-version：选项是Java命令，但JRE-版本清单条目的文件中所提及的任何普通的JDK文档中没有提及，也没有在Java SE平台规范。据我们所知，这个功能很少使用。它不必要地使Java发射器的实现复杂化，使得维护和增强变得繁重。

##### 描述

删除mJRE功能。修改启动器如下：

如果在命令行中给出了-version：选项，则发出错误消息并退出

如果在JAR文件中找到JRE-Version清单条目，则发出警告消息并继续。

在第二种情况下，警告而不是致命错误的理由是，清单条目可能存在于不能轻易修改的旧jar文件中，因此最好继续而不是中止。我们期望在JDK 10中将此案变为致命错误。

##### 测试

需要进行测试以确保删除该功能，并报告相应的警告和错误。

#### JEP 232: Improve Secure Application Performance
##### 概要

改进安装安全管理器运行的应用程序的性能。

##### 目标

更好地了解性能问题，并实施增强功能，以提高性能。将创建子任务以评估和跟踪每个潜在的改进。

##### 非目标

提高性能是一个理想的结果，但是这个JEP不会承诺提高性能的任何具体指标。

##### 动机

使用Java SE，Java EE和相关技术的许多开发人员已经要求提高使用安全管理器运行的应用程序的性能。虽然具体号码未被确认，并且可能因多个因素而异，但是已启用运行安全管理器的Java应用程序已经导致10-15％的性能下降。尽管某些性能损失是不可避免的，但是缩小性能差距具有性能和安全性的好处。

##### 描述

我们探索并实施了一些优化和增强功能，以提高安装安全管理器的应用程序的性能。其中一些优化提高了性能，而另一些则没有。其他几个人被证明有承诺，但由于各种原因不会被整合为本JEP的一部分。对于所考虑的每个优化和使用JMH创建的微型基准测试，都会打开新的JBS问题（如果以前没有存在）。

##### 优化

基于测试和社区反馈，我们提高绩效的主要重点领域是执行安全策略和权限评估。权限类和默认JDK策略实现被设计为线程安全。然而，多线程的性能测试表明这些类是一个热点。我们实施了几项改进，可以提高吞吐量并减少线程争用：
1. 使用ConcurrentHashMap将ProtectionDomain映射到PermissionCollection
2. SecureClassLoader应该使用ConcurrentHashMap
3. 删除在IdentityPolicyEntries列表上同步的策略提供程序代码
4. 在Permissions类中的ConcurrentHashMap而不是HashMap中存储PermissionCollection条目
5. 在PermissionCollection子类中的并发集合中存储权限

我们还改进了另外两个关键领域的表现：

我们更改了java.security.CodeSource的hashCode方法，以避免使用代码源URL的字符串形式来计算哈希码来进行潜在的昂贵的DNS查找。 有关详细信息，请参阅JDK-6826789。

我们增强了java.lang.SecurityManager的checkPackageAccess方法的包检查算法。 有关详细信息，请参阅JDK-8072692。

##### 测试

在每个潜在优化应用之前和之后，都需要进行测试来测量性能。 可能需要执行多个测试以确保适当的代码覆盖，并反映不同类型的用例。 分析工具将用于帮助确定潜在优化领域。 此外，我们将运行SPECjEnterprise基准测试来衡量优化对Java EE应用程序的性能影响。

最后，将在所有支持的JDK平台中测量每个优化，以确保结果始终如一。

#### JEP 233: Generate Run-Time Compiler Tests Automatically
##### 概要

开发一个工具，通过自动生成测试用例来测试运行时编译器。

##### 目标

生成的测试应该是jtreg兼容的

该工具应根据最终结果的语言（Java源代码，Java字节码），语言结构的使用，控制流程和表达式复杂性等进行配置。

测试应该随机生成，但可以重现

##### 动机

当我们添加新的平台，利用新的CPU指令，引入新的优化，并对运行时编译器进行其他增强时，通过直接，有针对性的测试来有效测试编译器变得越来越不可行。

##### 描述

该工具将随机生成语法和语义上正确的Java源代码或字节代码，如有必要，请在解释（-Xint）和编译（-Xcomp）模式下运行它，并验证结果。

该工具将自动工作，无需人工交互。生成的测试将在合理的时间内涵盖尽可能多的组合。

Java源代码编译器javac不使用Java的所有字节代码，因此仅生成Java源代码将会隐藏一些字节代码。生成所有类型测试的字节码将是一个复杂的任务，所以我们将采用一种混合方法来生成Java源代码和字节码。

在测试执行期间编译源代码对于嵌入式平台是有问题的，其中完整的JDK可能不可用，因此该工具将提供一种预编译源代码测试的方法。

生成的测试用例将包括复杂的表达式和控制流图，并将利用内在函数，浮点运算，try-catch-finally结构等。将有一种方法来调整工具的配置。

该工具将随机生成测试，但是对于重现性，它将报告其随机化种子并接受这样的种子，以便重播完全相同的生成测试序列。

该工具的源代码将被放置在热点/测试/ testlibrary / jit-tester目录中。可以通过工具的makefile中提供的目标生成测试。测试生成的结果是一个完整的jtreg测试套件，可以从同一个makefile或直接通过jtreg运行。工具makefile不会集成到HotSpot / JDK构建基础结构中。

鉴于测试生成过程花费大量时间，生成和运行这些测试不会被预先作为预集成测试的一部分。然而，有意义的是定期运行预生成的测试，可靠性测试和新的生成测试，以获得更好的代码覆盖。发现错误的生成测试应该作为常规回归测试集成到适当的测试套件中，并以与其他回归测试相同的方式运行。

##### 备择方案

以编译模式运行现有测试可以被认为是该工具的可行替代方案。这种方法有几个缺点：

它不保证涵盖所有语言结构和不同优化的组合

测试失败不一定意味着运行时编译器的缺陷，需要额外的工程师进行调查/复制

由于一些测试可以是运行时编译器的特定测试，它们可能会强制运行时编译本身和/或需要特定的运行时编译器状态。所以明确的编译模式可以改变测试行为并导致假阳性结果

创建测试失败的回归测试相对较难

由于这些缺点，这种方法不能完全取代所提出的工具。

#### JEP 235: Test Class-File Attributes Generated by javac
##### 概要

编写测试以验证javac生成的类文件属性的正确性。

##### 目标

创建测试，检查所有类文件属性是否正确生成，即使这些属性未在常规编译和运行场景中使用。另外，记录任何现有的这样的测试。

##### 动机

类文件属性测试覆盖率不足。未经VM检查的可选属性和属性无法通过编译输入源，运行它们以及验证编译后的程序是否符合预期的方式进行测试。缺乏覆盖会导致可能的错误，这些错误只能由外部工具（如调试器）检测到。需要一个特殊的测试套件，它将通过其他方式分析类文件和测试类文件属性的正确性。

##### 描述

测试由javac生成的文件的常见方法是运行编译的类，并验证生成的程序是否按预期运行。这种方法不适用于可选的类文件属性，也不适用于未经VM验证的属性，因此必须通过其他方法测试这两种属性。将开发将接受Java源代码作为输入，编译源代码，读取编译的类文件的类文件属性并验证其正确性的测试。

根据JVMS，类文件属性分为三组。

##### 可选属性

这些属性对javac，JVM或类库的正确操作并不重要，但它们被工具使用。测试这些属性是高度优先的，因为它们不被JDK的任何组件使用。

SourceFile
SourceDebugExtension
LineNumberTable
LocalVariableTable
LocalVariableTypeTable
Deprecated
##### JVM不使用的属性

JVM不使用这些属性，但它们由javac或类库使用。测试这些属性是一个中等优先级。

InnerClasses
EnclosingMethod
Synthetic
Signature
RuntimeVisibleAnnotations
RuntimeInvisibleAnnotations
RuntimeVisibleParameterAnnotations
RuntimeInvisibleParameterAnnotations
RuntimeVisibleTypeAnnotations
RuntimeInvisibleTypeAnnotations
AnnotationDefault
MethodParameters
##### 属性由JVM使用

这些属性由JVM的字节码验证器检查。不需要进一步测试。

ConstantValue
Code
StackMapTable
Exceptions
BootstrapMethods
##### 测试

为了测试这些新的测试，我们将检查测试是否失败，并且在针对故意损坏的类文件运行时使用适当的错误消息。

#### JEP 236: Parser API for Nashorn
##### 概要

为Nashorn的ECMAScript抽象语法树定义支持的API。

##### 目标

提供接口类来表示Nashorn语法树节点。

提供一个工厂来创建配置的解析器实例，配置完成后通过API传递Nashorn命令行选项。

提供一个访问者模式API来访问AST节点。

提供样本/测试程序以使用API​​。

##### 非目标

AST节点将尽可能在ECMAScript规范中表示概念，但它们不会完全相同。只要可能，ECMAScript将采用javac树API的接口。

不会使用外部解析器/树标准或API。

将没有脚本级的解析器API。这是一个Java API，尽管脚本可以调用Java，因此可以使用这个API。

##### 动机

诸如NetBeans的IDE使用Nashorn进行ECMAScript编辑/调试以及ECMAScript代码分析。这些工具和框架目前使用Nashorn的内部AST表示进行代码分析。 jdk.nashorn.internal.ir包中的内部类的使用及其子包阻止了Nashorn的内部实现类的自由演进。此JEP将在暴露的包jdk.nashorn.api.tree中定义Nashorn解析器API。 javac在com.sun.source包及其子包中已经支持类似的抽象语法树API。

解析器API将通过诸如IDE和服务器端框架的程序启用ECMAScript代码分析，而不需要依赖于Nashorn的内部实现类。

##### 描述

附加的javadoc文件包含新的jdk.nashorn.api.tree包的建议接口和类的文档。 API的起点是ParserFactory和ParserFactoryImpl类。 ParserFactory对象接受一个字符串数组，它们是配置解析器的选项。支持的选项与Nashorn shell工具jjs支持的选项以及Nashorn脚本引擎的nashorn.args系统属性相同。

一旦创建了一个解析器实例，那么来自一个字符串，URL或者文件的ECMAScript源可以被提交给解析器，这将返回一个CompilationUnitTree对象。任何解析错误都会通过调用者提供的DiagnosticListener对象进行报告。

##### 测试

已经有超过1000个Nashorn脚本测试。所有可解析的源将被解析，以检查解析器不会崩溃。将解析包含各种代码模式的所选文件，并将其JSON表示与预期结果进行比较。此外，将添加一组具有解析错误的文件，以检查用户提供的DiagnosticListener是否收到错误消息，并且生成的CompilationUnitTree根据需要具有ErrorTree节点。

#### JEP 237: Linux/AArch64 Port
##### 概要

端口JDK 9到Linux / AArch64。

##### 动机

AArch64是ARM控股公司的新处理器架构。这是与32位ARM处理器架构的脱节，实际上是一个完全重新设计。它需要一个新的OpenJDK端口。

##### 描述

我们（AArch64移植项目）将JDK移植到了一个新的平台：Linux / AArch64。我们已经实现了模板解释器，C1（客户端）和C2（服务器）JIT编译器。

这个JEP的重点不在于移植工作本身已经完成，而是将端口集成到OpenJDK主数据库中。

目前，我们在HotSpot存储库的共享部分中有大量的微不足道的变更集。这些大多数是＃ifdef，包括相关的平台特定文件。其他类型有一些变化，但再次由#ifdef AARCH64保护。因此，对其他平台的风险很低。

HotSpot的构建机器和JDK的其余部分也有变化，为字节顺序，字大小等添加适当的定义。再次，这些不应该影响其他平台。

要整合的大多数更改不会以任何方式影响当前的OpenJDK平台，因为它们仅在新平台上处于活动状态。

构建系统也有变化，但不应该造成很大的麻烦。

以下是HotSpot中共享代码所需更改的补丁。

所有更改集都将收集到分段存储库中：http://hg.openjdk.java.net/aarch64-port/stage/

##### 测试

Red Hat和Linaro定期构建和测试移植平台以及当前支持的JDK 8平台上的端口，以确保不会导致破坏现有平台的回归。

Red Hat致力于全面支持（即定期更新，增强和测试）此工作引入的新平台的代码。

##### 风险与假设

AArch64硬件可用性可能存在一些问题，但我们希望在本项目中可以广泛使用。

与PPC / AIX端口类似，将创建由AArch64 Port Project拥有的登台林（例如，aarch64端口/阶段），以包含经过审核和批准的更改集。

#### JEP 238: Multi-Release JAR Files
##### 概要

扩展JAR文件格式，以允许多个Java版本的类文件共存在单个存档中。

##### 目标

增强Java存档工具（jar），以便它可以创建多版本的JAR文件。

在JRE中实现多版本的JAR文件，包括在标准类加载器和JarFile API中的支持。

加强其他关键工具（例如javac，javap，jdeps等）来解释多版本的JAR文件。

支持目标1到3的多版本模块化JAR文件。

保持性能：使用多版本JAR文件的工具和组件的性能不能受到重大影响。特别地，访问普通（即不是多版本）JAR文件时的性能不能降级。

##### 动机

第三方库和框架通常支持一系列Java平台版本，通常会回到几个版本。因此，它们通常不会利用较新版本中可用的语言或API功能，因为很难表达条件平台依赖关系，通常涉及反射或为不同平台版本分发不同的库工件。

这对图书馆和框架造成了不利影响，因而使用新功能，从而为用户升级到新的JDK版本 - 阻碍采用的恶性循环，降低每个人的不利影响。

此外，一些库和框架使用JDK的内部API，当严格执行模块边界时，它将在Java 9中无法访问。当对这种内部API进行公共的支持的API替换时，这也会阻止支持新的平台版本。

##### 描述

JAR文件具有包含类和资源的内容根，以及包含有关JAR元数据的META-INF目录。通过向特定的文件组添加一些版本元数据，JAR格式可以以兼容的方式对不同目标Java平台版本的多个版本的库进行编码。

多版本JAR（“MRJAR”）将包含主要属性：

```
Multi-Release: true
```
在JAR MANIFEST.MF的主要部分宣布。 属性名称也被声明为常量java.util.jar.Attributes.MULTI_RELEASE。 像其他主要属性一样，在MANIFEST.MF中声明的名称不区分大小写。 该值也不区分大小写，但不得有前面或后面的空格（这样的限制有助于确保满足性能目标）。

多版本JAR（“MRJAR”）将包含特定于Java平台版本的类和资源的附加目录。 典型图书馆的JAR可能如下所示：

```
jar root
  - A.class
  - B.class
  - C.class
  - D.class
```

假设有备用版本的A和B可以利用Java 9功能。 我们可以将它们捆绑成一个JAR，如下所示：

```
jar root
  - A.class
  - B.class
  - C.class
  - D.class
  - META-INF
     - versions
        - 9
           - A.class
           - B.class
```
在不支持MRJAR的JDK中，只有根目录中的类和资源将可见，并且两个包装将是无法区分的。 在支持MRJAR的JDK中，与任何后来的Java平台版本相对应的目录将被忽略; 它将首先在与当前运行的主要Java平台版本相对应的Java平台特定目录中搜索类和资源，然后搜索较低版本的JAR根目录。 在Java 9 JDK上，就好像有一个特定于JAR的类路径首先包含版本9文件，然后包含JAR根; 在Java 8 JDK上，此类路径将仅包含JAR根。

假设稍后将发布Java 10，并更新A以利用Java 10功能。 MRJAR可能看起来像这样：

```
jar root
  - A.class
  - B.class
  - C.class
  - D.class
  - META-INF
     - versions
        - 9
           - A.class
           - B.class
        - 10
           - A.class
```
通过这种方案，为稍后的Java平台版本设计的类的版本可能会覆盖为早期Java平台版本设计的同一类的版本。在上面的例子中，当在支持MRJAR的Java 9 JDK上运行时，它会看到A和B的9个特定版本以及C和D的一般版本;在未来的MRJAR感知Java 10 JDK中，它会看到A的10个特定版本和B的9个特定版本;在旧的或非MRJAR感知的JDK上，它只会看到所有的根版本。

JAR元数据（例如在MANIFEST.MF文件和META-INF / services目录中找到的元数据）不需要进行版本控制。一个MRJAR本质上是一个发行版本，所以它只有一个发行版本（与普通JAR没有区别，通过Maven Central分发），尽管内部包含多个版本的库实现，用于不同的Java平台版本。图书馆的每个版本应提供相同的API;需要进行调查来确定这是否应该是严格的向后兼容性，其中API完全相同（字节码签名相等），或者这是否可以在一定程度上放宽，而不一定能够引入新的增强功能，从而模糊一个发布单位这可能意味着至少在发布专用目录中存在的公共类也应该存在于根目录中，尽管它不需要存在于较早的发行版目录中。运行时系统不会验证此属性，但是工具可以并且应该检测这种API兼容性问题，还可以提供库方法来执行这种变更（例如对于java.util.jar.JarFile）。

最终，这种机制使库和框架开发人员能够将特定Java平台版本中的API的使用与所有用户迁移到该版本的要求进行脱钩。图书馆和框架维护人员可以逐步迁移和支持新功能，同时仍然支持旧功能，打破鸡蛋和鸡蛋循环，从而使图书馆可以“Java 9就绪”而不需要Java 9。

##### 细节

将修改JDK的以下组件以支持多版本的JAR文件。

基于JAR的URLClassLoader必须按正在运行的Java平台版本来读取所选的类文件版本。由Project Jigsaw引入的基于模块的类加载器将需要类似的修改。

jar URL方案和java.util.jar.JarFile类的协议处理程序必须从多版本JAR中选择一个类的适当版本。

Java编译器（javac）通过底层的JavacFileManager和ZipFileSystem API必须读取由-target和-release命令行选项指定的类文件的选定版本。 javah，schemagen和wsgen的工具将利用对JavacFileManager和ZipFileSystem的底层更改。

Java Archive工具（jar）将被增强，以便它可以创建多版本的JAR文件。

必须更新JAR打包工具（pack200 / unpack200）（请参阅JDK-8066272）。

必须更新javap工具以启用版本化类文件的选择。

jdeps工具将需要修改才能显示版本信息，并遵循特定于版本的类文件依赖关系。

必须修改JAR规范以描述多版本JAR文件格式和任何相关更改（例如，可能添加到清单）。

##### 兼容性

默认情况下，java.util.jar.JarFile和jar方案协议处理程序的行为将保持不变。 有必要选择导入一个指向MRJAR的JarFile来进行条目的版本选择。 同样，有必要选择使用jar URL（有关详细信息，请参阅下一节）。

运行时为类加载创建的JarFile实例将选择加入并创建配置为根据正在运行的Java平台的版本选择条目的实例。 如JarFile实例被称为运行时版本控制。

##### 类装载资源

由类加载器生成的资源URL，用于标识MRJAR中的资源将直接引用版本化条目（如果存在）。 例如，对于版本化资源，foo / baz / resource.txt：

```
URL r = loader.getResource("foo/baz/resource.txt");
```
URL'r'可能是：

```
jar:file:/mrjar.jar!/META-INF/versions/9/foo/baz/resource.txt
```
而不是

```
jar:file:/mrjar.jar!/foo/baz/resource.txt
```

这种方法被认为是最不利的选择。更改资源URL的结构不是没有风险（例如新的方案或附加的片段）。传统代码可以直接处理URL字符，而不是解析URL并正确提取组件。虽然这样的URL过程不正确，但是认为最好不要打破这样的代码。

##### 模块化多版本JAR文件

模块化多版本JAR文件是一个多版本JAR文件，在顶部的根中具有一个模块描述符module-info.class，就像模块化的JAR文件（参见JEP 261的Packaging：Modular JAR部分）。此外，模版描述符可能存在于版本化区域中。这样的版本描述符必须与根模块描述符相同，有两个例外：

一个版本描述符可以具有不同的非传递性要求的java。*和jdk。*模块的条款;和

版本描述符可以具有不同的使用条款，即使是在java。*和jdk。*模块之外定义的服务类型。

这里的理由是这些是实现细节，而不是模块的API表面的一部分，并且可能希望随着JDK本身的发展而改变它们。非公开对非JDK模块的更改不允许。如果需要，则需要新版本的模块（至少增加它的版本号），这是一种不同的兼容性问题，超出了MRJAR的范围。

多版本模块化不需要在所在根的模块描述符。在这方面，模块描述符将被视为不同于任何其他类或资源文件。这可以确保，例如，只有Java 8版本类存在于根区域，而Java 9版本类（包括模块描述符）存在于9版本区域中。

##### Classpath和modulepath

可以构造模块化JAR，使其在Java 8运行时的类路径，Java 9运行时的类路径或Java 9运行时的模块路径上正常工作。模块化多版本JAR文件（除了module-info.class之外，其他类可能为Java 9平台编译）的情况是相同的。

如果模块描述符没有将某些包声明为导出，因此这些包中的公共类对该模块是私有的，则当对应的JAR文件放在模块路径上时，类将无法访问。但是，如果JAR文件放在类路径上，那么这些类将被访问。这是支持classpath和modulepath的不幸后果。

因此，与放置在模块路径上时相比，多版本JAR文件的公共API可能与放置在类路径上时不同。通常，构建多版本JAR文件时的jar工具，如果检测到公共API中存在任何观察到的差异，则尽力而为地失败。然而，当构建模块化多版本JAR文件时，建议如果公共API差异是将JAR文件放置在类路径上时可以访问的模块专用类的结果，则jar工具会输出警告。

##### 多版本的jar和引导加载程序

引导加载程序不支持多版本JAR（例如，当使用-Xbootclasspath / a选项声明多版本JAR文件时）。这种支持将使引导加载程序实现复杂化，这被认为是罕见的用例。

##### 备择方案

常见的方法是使用静态反射检查来确定API特征是否存在，并因此选择分别取决于该特征的适当类别。反射成本是在类初始化时引起的，而不是每次使用从属特征。选择Java平台版本进行编译，将源标记和目标标志设置为较低版本，以生成与该较低版本兼容的类文件。通常使用诸如Animal Sniffer之类的工具来增强这种方法，以检查API的不兼容性，除了执行API兼容性代码之外，还可以对其进行注释，以说明是否依赖于较晚的Java平台版本。这种方法有一些限制：

反思性检查需要谨慎维护。

不可能使用较新的语言功能。

如果平台发布API功能被删除（可能是内部API），则依赖代码将无法编译。

“Fat”类文件被考虑，其中类可能有一个或多个方法针对不同的Java平台版本。在支持这种方法声明和动态选择所需的语言和运行时特性方面，这被认为太复杂了。

由于需要维护二进制兼容性，因此无法使用方法句柄（invokedynamic）。

##### 风险与假设

预计MRJAR的生产主要兼容现有的流行构建工具，因此与支持这些工具的IDE兼容，但开发人员的经验可以通过增强来改进。

使用多模块项目，Maven可以支持MRJAR文件的源布局和构建。例如，参见这个可以产生一个当前基本的MRJAR文件的Maven项目。将有一个子项目用于根和具体的Java平台发布，以及将上述子项目组装成MVJAR的子项目。可以使用特定的Maven插件来增强装配过程，利用与jar工具相同的功能来实现向后兼容性。

MRJAR的运行时处理的设计和实现目前假设运行时使用URL类加载器或自定义类加载器利用JarFile来获取特定于平台的类文件。类装载器使用ZipFile加载类的运行时不会被MRJAR识别。需要检查流行的应用程序框架和工具，如Jetty，Tomcat和Maven等。

##### 依赖性

考虑到Java Platform Module System的增强JAR文件格式需要考虑多版本的JAR元数据。

支持对较旧版本的平台库进行编译的JEP 247（编译旧版本）可能有助于在多版本JAR文件的生成中构建工具。

#### JEP 240: Remove the JVM TI hprof Agent
##### 概要

从JDK中删除hprof代理。

##### 非目标

以hprof格式创建堆转储的能力将保持不变。

##### 动机

hprof代理的有用功能已被更好的替代品所取代。

##### 堆堆（堆=转储）

该功能已被JVM中相同的功能所取代。使用诊断命令GC.heap_dump（jcmd <pid> GC.heap_dump）可以要求JVM以hprof文件格式转储堆（这也可以通过jmap -dump使用）。

##### 分配分析器（heap = sites）

Java VisualVM工具提供与多个第三方剖析器相同的功能。

##### CPU分析器（cpu = samples，cpu = times）

CPU分析器有几个缺点，这些缺点在http://www.brendangregg.com/blog/2014-06-09/java-cpu-sampling-using-hprof.html和其他地方有详细描述。还有其他工具可以提供更好的功能，减少问题。其中包括与JDK捆绑的Java VisualVM和Java Flight Recorder以及一些第三方剖析器。

##### 演示代码

hprof代理程序被写为JVM工具接口的演示代码，而不是作为生产工具。代码和文档包含以下形式的许多语句：

这是JVM TI接口的演示代码和BCI的使用，它不是JDK的官方产品或正式部分。

##### 描述

作为JDK的一部分，停止构建和运送hprof代理程序库（libhprof.so）。

将代码移动到OpenJDK下的单独项目中。该代码仍然是JVM TI功能演示的有价值的参考，但不符合运输产品的要求。

更改应在适当的发行说明中记录。考虑到文档中已经存在的广泛的免责声明，用户不应该惊讶于hprof被删除，但应该更新文档以指向替代工具。

##### 测试

现有的hprof代理测试将需要删除。

#### JEP 241: Remove the jhat Tool
##### 概要

删除过时的jhat工具。

##### 动机

基于java.net HAT项目，jhat被添加到JDK 6中。 jhat是一个实验性的，不受支持的和过时的工具。 高级堆可视化和分析仪已经可用多年了。

##### 描述

停止在JDK中构建和运送jhat工具。

在适当的发行说明中记录此更改。 很少有人使用这个工具，所以对用户的影响应该是最小的。 该工具长期以来被标记为在文档中被删除：

注意：此工具是实验性的，可能在将来的JDK版本中不可用。

##### 测试

现有的jhat测试将需要删除。

一些现有的hprof文件创建测试使用jhat验证文件。 我们需要保留在这些测试中使用的jhat hprof文件解析器/验证器。 然而，这并不意味着代码必须在产品中，而应该是测试的一部分。

#### JEP 243: Java-Level JVM Compiler Interface
##### 概要

开发基于Java的JVM编译器接口（JVMCI），使Java编写的编译器能够被JVM用作动态编译器。

##### 目标

允许针对JVMCI编程的Java组件在运行时加载并由JVM的编译代理程序使用。

允许针对JVMCI编程的Java组件在运行时加载，并由受信任的Java代码使用，以便在JVM中安装机器代码，该代码可以通过对已安装代码的Java引用来调用。

##### 非目标

基于JVMCI的动态编译器（如Graal）的集成。
##### 成功指标

基于JVMCI的动态编译器在未修改的JVM上运行的能力。编译器及其生成的代码的性能不是很受关注，因为JVMCI将是JDK 9中的一个实验性功能，因此只能通过在命令行上指定某些标志来实现。
##### 动机

优化编译器是一个复杂的软件，可从Java提供的功能中大大获益，例如自动内存管理，异常处理，同步，优秀（和免费）IDE，优异的单元测试支持和运行时可扩展性，通过服务加载程序命名少数。另外，编译器不需要许多其他JVM子系统（如字节码解释器和垃圾收集器）所需的低级语言特性。这些观察结果强烈地表明，在Java中编写JVM编译器应该允许生产高质量的编译器，这将比C或C ++开发的现有编译器更容易维护和改进。 JVMCI提供必要的API来证明和实验这个潜力。

##### 描述

JVMCI API将由以下机制组成：

访问优化字节码到机器代码编译器所需的VM数据结构，如类，字段，方法，分析信息等。
安装编译代码以及JVM所需的所有元数据，用于管理编译代码，如GC地图和信息，以支持去优化。
插入JVM的编译系统来处理服务JVM请求以生成方法的机器代码。
Graal提供了使用JVMCI在JVM中编写和部署高性能编译器的一个很好的演示，Graal在各种基准测试中证明了与C2相当的峰值性能。

##### 备择方案

没有可用于Java编译器的现有替代方法，可用作JVM中的动态编译器。

##### 测试

JVMCI实现包括对接受编译器开发人员的API部分进行的广泛的单元测试，以及内部部件的白盒测试，可以调用VM来实现公共部分。

##### 风险与假设

允许VM内部访问和编译代码进行安装和执行的Java API具有明显的安全隐患。在JDK8（其中JVMCI是第一个原型并且与JDK 9端口一起开发的JVMCI）中，JVMCI API通过加载一个类加载器，而不能访问引导类路径上的代码，而将其加载到不可信代码中。在JDK 9中，目的是使用作为模块系统（JSR 261）一部分的访问控制来防止不受信任的代码能够使用JVMCI。 JVMCI需要像Unsafe类一样安全。

JVMCI将在JDK 9中进行实验，这将需要额外的命令行选项来启用它。例如：

```
-XX:+UnlockExperimentalVMOptions -XX:+EnableJVMCI -XX:+UseJVMCICompiler -Djvmci.Compiler=<name of compiler>
```
使JVMCI实验能够进行广泛的实验，同时减轻对JVM客户的任何风险。

##### 依赖

如上所述，JVMCI依赖于JSR 261中的访问控制，以将JVMCI与不受信任的代码隔离开来。

#### JEP 244: TLS Application-Layer Protocol Negotiation Extension
##### 概要

扩展javax.net.ssl包以支持TLS应用层协议协商（ALPN）扩展，它提供了协商TLS连接的应用协议的方法。

##### 动机

为了支持希望在同一传输层端口上使用多个应用层协议的TLS客户端和服务器，ALPN扩展允许客户端以优先级顺序提供其支持的应用层协议的列表。然后，服务器可以选择所通告的客户端协议之一，并告知客户端在TLS连接中将使用哪种协议。

这种TLS扩展的一个重要消费者将是实现HTTP / 2的HTTP / 2客户端（JEP 110）。

##### 描述

该功能定义了一个公共API来协商可以通过给定TLS连接传输的应用层协议。在初始TLS握手期间，协议名称在客户端和服务器之间传送。

TLS应用程序可以使用扩展SSLParameters类来获取和设置在给定连接上可以支持的应用层协议列表。 TLS实现也使用此类来检索应用程序声明的协议名称。

默认行为是选择服务器最受欢迎的应用程序协议值的交叉值。

服务器应用程序还可以外部扫描初始明文ClientHellos以为该连接选择适当的ALPN协议值。可以根据提供的TLS协议，密码伙伴，服务器名称指示值等进行此决定。服务器应用程序可以：

选择一个提供的协议，如果它将支持它，
决定远程提供和本地支持的ALPN值是互斥的，或者
完全忽略扩展。
根据在连接期间可用的应用协议，服务器可能会更改连接参数，例如其发布的服务器证书。

在SSL / TLS握手开始之后，SSLSocket / SSLEngine上有新的方法允许应用程序查询是否已经选择了ALPN值（getHandshakeApplicationProtocol（））。
一旦TLS握手完成，应用程序就可以使用getApplicationProtocol（）方法来检查已协商的协议。

所提出的设计遵循用于JDK 8中引入的服务器名称指示扩展（JEP 114）的类似的API方法，但不同之处在于ALPN值与连接而不是SSLSession相关联。

##### 测试

客户端实现应该能够使用支持HTTP / 2和支持SPDY的Web服务器（例如，Apache / mod_spdy，Jetty以及其他可能的）。 SPDY正在变得不那么重要，因为HTTP / 2是其非常公开的替代品。 SPDY实施应该很快就会变得很少。我们一定会使用JEP 110中的新的HTTP / 2客户端进行测试。
应该针对能够使用ALPN（例如GnuTLS，NSS，OpenSSL（beta）和Microsoft SChannel 8.1）的众所周知的TLS客户端实现来测试服务器端实现。我们目前没有计划推出任何支持服务器端ALPN的应用程序。这里的大多数测试将是一些简单的TLS握手，并检查协商的值。

#### JEP 245: Validate JVM Command-Line Flag Arguments
##### 概要

验证所有JVM命令行标志的参数，以避免崩溃，并确保在无效时显示相应的错误消息。

##### 非目标

我们不会验证JVM未处理的标志的参数。

我们不会尝试调整参数落在允许的范围内;我们只会检测到错误的参数，而不是正确的参数。

##### 成功指标

显示获取值并提供超出范围值的任何JVM标志不会导致JVM崩溃，而是发出信息错误消息。这将针对JVM标志进行，无论它们是否符合人体工程学设置（例如，在包含文件中），在命令行上设置，通过工具输入设置，或通过API设置（例如attachListener / jcmd或java.lang.management）。

##### 描述

任何接口，无论是程序化还是用户可见，都必须提供足够的能力来验证输入值。在命令行的情况下，对需要用户指定值的参数实现范围检查至关重要。 globals.hpp源文件包含标志值的起点和初步范围检查。扩展和完成这个可以提供正确的覆盖。

此外，我们应该定义一个框架，以便为添加新的JVM命令行标志来利用这种有效性检查提供便利。框架应该是灵活的，允许检查单个值，在最小值和最大值之间，或在一组值之内等。

我们将通过扩展现有的宏表（例如，RUNTIME_FLAGS）来实现此功能，以添加可选范围（最小，最大）和约束（function_pointer）条目。当前范围检查和其他特殊验证码将被移植并移除。

当所有标志的最终值都设置时，范围和约束检查在每次标志更改时以及JVM初始化程序（即，在stubRoutines_init2（））之后的init_globals（））中执行。只要JVM运行，我们将继续检查可管理的标志。

对于那些依赖于其他可能在设置所涉及的标志时可能未设置的标志的标志，我们将以API（CommandLineFlags :: finishedInitializing（））的形式提供一种机制，让约束函数知道所有标志是否具有最终值根据需要设置并将其行为从NOP更改为错误。

拦截标志值更改在CommandLineFlags :: xxxxAtPut设置器中以较低级别完成，以确保可管理的标志检查其范围和约束（例如，通过jcmd设置的标志）。例如，使用超出允许范围的jcmd PID VM.set_flag MinHeapFreeRatio 101将打印出

```
PID:
MinHeapFreeRatio error: must have value in range [0...100]
```
在jcmd输出。

范围检查不会对JVM初始化过程施加任何行为更改。 特别是它们不会终止JVM，而是将它们的状态传播到使用它们的代码。 约束函数可以终止JVM以根据需要匹配现有的自定义行为。

默认情况下，范围/约束检查不是冗长的，以阻止其消息被打印出来。 范围检查会在JVM初始化期间在错误流上打印错误消息，以匹配当前行为。 打印到可管理标志的错误流被禁止; 而是通过WriteableFlags代码处理任何错误状态，以便在提供的FormatBuffer中提供详细状态，以便它可以由jcmd进程本身而不是由目标进程打印。

在JVM初始化期间发生范围检查失败的情况下，默认情况下将以以下形式打印出错误消息：

```
uintx UnguardOnExecutionViolation = 3 is outside the allowed range [ 0 ... 2 ]
```
然而，我们目前并不承诺任何特定的格式。 预期某种消息格式的现有测试将必须被修改以允许新的格式。

即使我们有这种能力，现有的行为也不会相对于夹紧标志而改变，以适合规定的范围（即我们不夹紧）。 在初始化JVM（即，我们终止进程）时，我们符合错误检测期间的现有行为。

##### 备择方案

Variant宏提供了一个稍微不同的，也可能是更清洁的方式来定义范围和约束，但Solaris C ++编译器中的“带有空args的尾随逗号”问题阻止了这种方法的采用。

#### JEP 246: Leverage CPU Instructions for GHASH and RSA
##### 概要

通过利用最近推出的SPARC和Intel x64 CPU指令，提高GHASH和RSA加密操作的性能。

##### 成功指标

对JDK 8中包含的AES-CBC的支持（见JEP 164）显示了基于纯软件实现的改进。不同的算法会有所不同，但我们应该看到类似的显着性能提升。

##### 动机

我们使用本机库（例如PKCS＃11）越少，与复杂的本机API交互引起的复杂代码和内存问题就越少。 JNI对本地库的调用越少，加密越快。通过直接在JVM中实现加密操作，我们可以通过内置提供商来控制其实现和管理，从而提供开箱即用的支持。

##### 描述

没有现有的API将被修改或扩展。

##### 算法

当支持这些指令时，现有的实现会在HotSpot中调用AES指令。除了CBC模式之外，还有一些优化可帮助AES和CBC快速合作。指令和优化替代了当前的SunJCE字节码方法。该计划是为GCM和RSA实现类似的优化，这可以从硬件帮助中获益很大。 AES-GCM和RSA都是TLS密码套件的一部分。

作为GCM一部分的GHASH将使用英特尔x64上的pclmulqdq和SPARC上的xmul / xmulhi来加速。

RSA将通过使用位操作指令集2来加速。其他非对称算法很可能从这些变化中受益，但是它们将通过RSA来衡量。鉴于“montmul”和“montsqr”指令的复杂性和局限性，SPARC指令未被添加。使用本地库提供完整的RSA功能，而不会出现问题。此外，由于RSA是一个缓慢的操作，JNI和本机API层最可能在整个所有性能图片中花费很少。

##### 供应商

算法的管理是一个随着时间的推移变得越来越复杂的重要问题。极端情况是Solaris的默认提供程序配置。 SunPKCS11提供商在提供商列表中领先于SunJCE。 SunPKCS11提供程序支持Solaris的所有硬件加速和优化算法。要使用JDK 8 AES-CBC支持，SunJCE必须先于SunPKCS11。对于仅需要AES-CBC的应用程序，如性能测试，则不需要其他算法，因此可以实现。然而，对于使用多种算法的应用，其他算法将使用非加速的基于软件的实现而不是SunPKCS11的硬件加速实现来运行。当使用NSS（通过SunPKCS11提供程序配置）时，其他操作系统也可能会出现此问题。

因此，在java.security文件jdk.security.provider.preferred中添加了一个新的安全属性，以允许某些算法和算法组在检查有序提供程序列表之前被引导到特定的提供者。此属性适用于高级用户，默认情况下不设置。在目前使用的x86和SPARC CPU的许多不同版本中，设置默认值可能会导致旧系统的性能回归，并需要持续维护，因为新的CPU提供更多的支持。此外，现有的JDK配置（如FIPS 140或其他专业提供商）可能不知不觉地针对不同的提供商。因此，默认情况下，最好将jdk.security.provider.preferred属性取消设置，但让供应商和高级用户将属性设置为其支持的CPU。

##### 测试

现有的已知答案测试（KAT）应足以进行功能测试。将使用现有的基准测试和内部测试进行大量的性能测试。

#### JEP 247: Compile for Older Platform Versions
##### 概要

增强javac，以便它可以编译Java程序以在所选的较旧版本的平台上运行。

##### 动机

javac提供了两个命令行选项-source和-target，它们可用于分别选择编译器接受的Java语言版本及其生成的类文件的版本。然而，默认情况下，javac针对最新版本的平台API进行编译。因此，编译的程序可能会意外使用仅在当前版本的平台中可用的API。这些程序无法在较旧版本的平台上运行，无论传递给-source和-target选项的值如何。这是一个长期可用性的痛点，因为用户期望通过使用这些选项，他们将获得可以在由-target指定的平台版本上运行的类文件。

##### 描述

定义了一个新的命令行选项--release，它会自动配置编译器来生成将与给定平台版本的实现链接的类文件。 - 释放N大致相当于：

对于N <9：-source N -target N -bootclasspath <recorded-APIs-from-N>，
对于N> = 9：-source N -target N - 系统<documents-APIs-from-N>。
对于N <9，记录的API由JDK N的javac缺省bootclasspath上的公共API组成。

对于N> = 9，记录的API包括（i）从作为JDK N文档的一部分的JDK映像中的那些模块导出的API;和（ii）从jdk.unsupported模块导出的API（在JEP 260中记录）。也就是说，记录的API主要是通过JDK映像中的模块和为JDK N记录的模块之间导出的API。JDK映像中没有其他模块可以观察到。如果使用了--limit-modules，那么它只能进一步限制可观察模块，而不是观察其他模块。不允许访问JDK映像中的模块内部。

--release N选项与影响一组平台或系统类的其他选项不兼容。这包括：

对于N <9：-bootclasspath，-Xbootclasspath，-Xbootclasspath / a :, -Xbootclasspath / p :, -endorseddirs，-Djava.endorsed.dirs，-extdirs，-Djava.ext.dirs设置平台类的选项。
对于N> = 9：设置系统模块（即JDK映像中的模块）的--system和-upgrade-module-path选项以及--add-exports，--add-reading和 - -patch-module选项，如果它们修改系统模块。 （对于非系统模块，即不是JDK映像的一部分的模块，允许使用--add-exports，--add-reading和-patch-module。）
对于任何N，-source和-target选项，因为它们自动设置为N.
假设已记录的API将仅在主要版本中更改。对于在JDK 6的次要版本中将JAX-WS从2.0更新为2.1的遗留情况，JAX-WS 2.1被认为是已记录的API。

作为--release实现的限制，在编译期间不使用给定目标平台的Unicode版本;而是使用当前平台的Unicode版本。

##### 履行

对于JDK N和 - 释放M，M <N，需要平台版本M的已记录的API的签名数据。该数据存储在$ JDK_ROOT / lib / ct.sym文件中，与JDK 8中相同名称的文件类似，但不一样。ct.sym文件是一个ZIP文件，包含已删除的类对应于目标平台版本的类文件的文件。

对于JDK N和--release N，JDK自己的映像被用作编译的类文件的源代码。然而，可观察模块的列表对于已记录的模块和jdk.unsupported模块是有限的。

##### 风险与假设

JDK源代码存储库需要包含过去版本的平台API的描述。描述的大小可能是相当大的，并且生成的JDK构建将会更大。已经注意尽可能地减少这些空间开销。

#### JEP 248: Make G1 the Default Garbage Collector
##### 概要

使G1成为32位和64位服务器配置上的默认垃圾收集器。

##### 动机

一般来说，限制GC暂停时间比最大化吞吐量更重要。切换到诸如G1之类的低暂停收集器应为大多数用户提供比面向吞吐量的收集器（如并行GC）更好的整体体验，目前它是默认的。

在JDK 8及其更新版本中，对G1进行了许多性能改进，并为JDK 9进行了进一步的改进。在JDK 8u40中引入了并发类卸载（JEP 156），使G1成为功能齐全的垃圾收集器，默认。

##### 描述

更改默认收集器是直接的。

##### 风险与假设

这种变化是基于这样的假设：限制延迟通常比最大化吞吐量更重要。如果这个假设是不正确的，那么这个改变可能需要重新考虑。

G1被认为是一个健壮且经过良好测试的收集器。预计不会有稳定性问题，但成为默认收藏者将增加其可见性，并可能揭示以前未知的问题。如果发现在JDK 9时间框架内无法解决的关键问题，我们将恢复使用Parallel GC作为JDK 9 GA的默认值。

G1的资源使用与Parallel不同。当需要最小化资源使用开销时，应使用G1以外的收集器，并且在此更改之后，必须明确指定备用收集器。

#### JEP 249: OCSP Stapling for TLS
##### 概要

通过TLS证书状态请求扩展（RFC 6066第8节）和多证书状态请求扩展（RFC 6961）实现OCSP装订。

##### 成功指标

客户端和服务器模式的实现必须与支持OCSP装订的至少两个第三方TLS实现成功地进行互操作。

##### 动机

检查X.509证书的吊销状态是有效的基于证书的身份验证的关键部分。然而，使用OCSP的证书状态检查通常涉及要检查的每个证书的网络请求。由于额外的网络请求，在客户端启用对TLS的OCSP检查可能会对性能产生重大影响。

OCSP装订允许证书的主持人（而不是颁发证书颁发机构（CA））承担提供OCSP响应的资源成本。在TLS上下文中，TLS服务器有责任在SSL / TLS握手期间请求OCSP响应并将其发送给客户端。这也允许服务器缓存OCSP响应并将其提供给所有连接到它的客户端。这可以显着降低OCSP响应者的负担，因为响应可以由服务器缓存并定期刷新，而不是由每个客户端刷新。

目前，客户端可以启用证书吊销状态检查。然而，这种经典的方法面临着几个挑战：

##### 性能

如果客户端直接从OCSP响应方获取吊销状态，则对于与特定服务器建立连接的每个客户端，OCSP响应方必须以特定证书状态进行响应。对于高流量网站，OCSP响应者可能是性能瓶颈。此外，撤销状态检查涉及几次往返。如果在客户端启用OCSP检查，则会产生显着的性能影响。

##### 安全

Adam Langley在他的一篇博客中谈到了客户端应用程序面对传统OCSP的安全挑战。他描述了大多数浏览器实施的“软失败”行为，其中与OCSP响应者联系失败的策略不会导致撤销检查失败。这允许攻击者通过拦截或阻止来自客户端的OCSP请求，或者通过对响应者本身安装拒绝服务攻击来使客户端绕过吊销检查。

OCSP装订本身并不能完全减轻这一挑战，而是消除了客户端和响应者之间OCSP检查的需要。服务器仍然可以获得自己的OCSP响应，并将其作为TLS握手的一部分进行带内放置。

目前在IETF中的草案形式是一个“必须”证书扩展的提案，需要在TLS握手期间附加OCSP响应。这实际上将覆盖客户端可能采用的任何软故障行为。这个提议的扩展超出了本JEP的范围，但是如果这个扩展超出了一个草案，它将来可能是一个有用的补充。

最后，Java客户端可能没有与浏览器执行的软失败默认值相同的需求。在某些情况下，客户端可能会喜欢使用硬失败的方法，或者在未能收到OCSP响应的情况下，选择通过对话框获取用户反馈。作为示例，Java Plugin和Java Web Start的默认软失败方法是在加载签名的applet时显示一个警告对话框。

##### OCSP请求的潜在隐私损害

在正常的OCSP情况下，当客户端发送OCSP请求时，它会将服务器（通过服务器证书条目）和自身（至少通过IP地址）公开给OCSP响应者，从而可以公开客户端行为。 OCSP装订解决了这个问题，因为客户端不再向OCSP响应者发出请求。

##### “专属门户”技术的局限性

在使用网络正常之前，“captive portal”技术强制网络上的HTTP客户端下载特殊网页，通常用于认证。在这种环境中，客户端无法检查SSL / TLS证书的OCSP状态，因为所有网络访问都将被阻止，直到认证成功。

##### 概要

上述问题可以通过使用CRL来部分缓解，或者通过OCSP装订更好地解决。

总之，通过减少OCSP响应者的性能瓶颈，OCSP装订可以帮助提高TLS的性能。也可以防止OCSP请求的潜在隐私损害，避免限制“强制门户”技术。

##### 描述

该功能将在SunJSSE提供商实现中实现。计划进行微小的API更改，目标是尽可能减小这些更改。实现将为OCSP特定参数选择合理的默认值，并通过以下系统属性提供这些默认值的配置：

jdk.tls.client.enableStatusRequestExtension：默认情况下，此属性为true。这将启用status_request和status_request_v2扩展，并启用对服务器发送的CertificateStatus消息的处理。
jdk.tls.server.enableStatusRequestExtension：默认情况下，此属性为false。这使服务器端支持OCSP装订。
jdk.tls.stapling.responseTimeout：此属性控制服务器用于获取OCSP响应的最大时间，无论是从缓存还是通过联系OCSP响应程序。已收到的响应将根据正在完成的装订类型（如果适用）发送到CertificateStatus消息中。该属性以毫秒为单位的整数值，默认值为5000。
jdk.tls.stapling.cacheSize：此属性控制条目中的最大缓存大小。默认值为256个对象。如果缓存已满，并且需要缓存新的响应，则最新使用的缓存条目将被替换为新的缓存。该属性的值为零或更小意味着缓存对其可能包含的响应数量没有上限。
jdk.tls.stapling.cacheLifetime：此属性控制缓存响应的最长使用寿命。该值以秒为单位，默认值为3600（1小时）。如果响应具有比缓存生命周期早到的nextUpdate字段，则响应的生命周期可能比使用此属性设置的值更短。此属性的值为零或更小将禁用缓存生存期。如果对象没有nextUpdate，并且已禁用缓存生存期，则不会缓存响应。
jdk.tls.stapling.responderURI：此属性允许管理员在TLS使用的证书没有权限信息访问扩展名的情况下设置默认URI。除非设置了jdk.tls.stapling.responderOverride属性，否则不会覆盖AIA扩展值（见下文）。默认情况下不设置此属性。
jdk.tls.stapling.responderOverride：此属性允许通过jdk.tls.stapling.responderURI属性提供的URI来覆盖任何AIA扩展值。默认为false。
jdk.tls.stapling.ignoreExtensions：此属性禁用在status_request或status_request_v2 TLS扩展中指定的OCSP扩展的转发。默认为false。
客户端和服务器端Java实现将能够支持status_request和status_request_v2 TLS hello扩展。 status_request扩展在RFC 6066中描述。支持服务器将包括用于在新的TLS握手消息（CertificateStatus）中标识服务器的证书的单个OCSP响应。 status_request_v2扩展在RFC 6961中描述。该扩展允许客户端请求服务器在CertificateStatus消息中提供单个OCSP响应（类似于status_request）或请求服务器为证书列表中的每个证书提取OCSP响应证书消息中提供（以下称为ocsp_multi类型）。

##### 客户端

默认情况下将启用OCSP装订，并可以通过设置系统属性来禁用。这可以通过jdk.tls.client.enableStatusRequestExtension属性来完成。

默认情况下，客户端将在ClientHello握手消息中声明status_request和status_request_v2扩展。对于status_request_v2扩展，ocsp和ocsp_multi类型将被断言。

创建hello扩展将需要在sun.security.ssl中创建新类，类似于ServerNameIndicator，RenegotiationInfoExtension和其他扩展的实现方式。

为了使用新的扩展，ClientHello类将添加额外的方法来添加这些扩展。这些方法将从ClientHandshaker.clientHello（）中调用。

需要创建HandshakeMessage类中的一个新的握手消息类来处理CertificateStatus消息的编码和解析。

在ExtendedSSLSession中需要进行公共API更改，这允许呼叫者获取在握手过程中收到的OCSP响应。新方法是：

```
public List<byte[]> getStatusResponses();
```

##### 服务器端

默认情况下，服务器端实现将禁用OCSP装订，但可以通过jdk.tls.server.enableStatusRequestExtension系统属性启用。禁用OCSP装订支持的服务器将忽略status_request和status_request_v2扩展名。

ServerHello消息中的status_request或status_request_v2信息的服务器端群体将取决于客户端如何声明这些扩展。通常，ClientHello中的相同请求扩展将在ServerHello中返回，但有以下例外：

在ClientHello中接收status_request和status_request_v2扩展的服务器将在ServerHello中断言status_request_v2。

在ClientHello中使用ocsp和ocsp_multi类型接收status_request_v2扩展名的服务器将在CertificateStatus消息中的ServerHello消息和ocsp_multi中断言status_request_v2。

在选择了status_request_v2 / ocsp_multi的情况下，将使用不同的线程来获取每个响应。这将由一个StatusResponseManager管理，它将处理OCSP响应的提取和缓存。

应尽可能缓存OCSP响应。在status_request [_v2]扩展名中未指定过滤器的客户端可能会收到缓存的响应。

如果当前时间晚于nextUpdate字段，则不应使用缓存响应。

没有nextUpdate字段的缓存响应可能会保留在高速缓存中以达到预定的生命周期（请参见下面的可调参数）。

接收带有nonce扩展名的status_requests的服务器不能在CertificateStatus消息中返回缓存的响应。

服务器端装订支持将通过上述系统属性进行调整。

StatusResponseManager被创建为SSLContext实例化的一部分。在SSLContext构建期间对属性值进行采样。可以更改这些属性值，并且创建新的SSLContext对象时，StatusResponseManager将具有这些新值。

##### 装订和X509扩展托管管理器

开发人员在如何处理通过OCSP装订提供的响应方面具有一定的灵活性。此JEP不会更改证书路径检查和撤销检查中涉及的当前方法。这意味着客户端和服务器都可以断言status_request扩展，通过CertificateStatus消息获取OCSP响应，并允许用户灵活地对撤销信息做出反应或缺乏它。

与以前的JDK版本一样，如果调用者没有提供PKIXBuilderParameters，则禁用撤销检查。如果调用者创建一个PKIXBuilderParameters，并使用setRevocationEnabled方法来启用撤销检查，则会对OCSP响应进行装订。如果com.sun.net.ssl.checkRevocation属性设置为true，则也是如此。下表显示了几个不同的方法作为示例（假设客户端和服务器都启用了OCSP装订）：

```
PKIXBuilderParameters	checkRevocation Property	PKIXRevocationChecker	Result
default	default	default	Revocation checking is disabled
default	true	default	Revocation checking enabled*, SOFT_FAIL set
instantiated	default	default	Revocation checking enabled*, SOFT_FAIL set
instantiated	default	instantiated, added to PKIXBuilderParameters	Revocation checking enabled*, hard fail behavior.
```

*仅当ocsp.enable Security属性设置为true时，才会出现客户端OCSP回退

有关PKIXBuilderParameters和PKIXRevocationChecker对象的配置及其与JSSE的关系的更多详细信息，请参见“Java PKI API程序员指南”和“JSSE参考指南”。

##### 测试

OCSP Stapling实现不能破坏向后兼容性。

客户端实现必须能够向支持服务器发送RFC 6066样式的status_request ClientHello扩展。它必须能够正确解析ServerHello握手消息中相同的hello扩展，并正确解析后续的CertificateStatus握手消息内容。

客户端实现必须能够向支持服务器发送RFC 6961样式的status_request_v2 ClientHello扩展。它必须能够在hello扩展中声明ocsp或ocsp_multi类型（或两者）。它必须能够正确解析ServerHello握手消息中相同的hello扩展，并正确解析后续的CertificateStatus握手消息内容。

服务器实现必须能够在ClientHello握手消息中接收status_request和status_request_v2扩展，并查询相应的OCSP响应程序。它必须能够将OCSP响应放在CertificateStatus TLS握手消息中以返回给客户端。

服务器实现必须能够缓存有效的OCSP响应，以便在其status_request [_v2] hello扩展中不使用nonce扩展的请求的客户端重新使用。

客户端必须能够与能够执行OCSP装订（例如Apache 2.4+）的至少两个不同的Web服务器进行互操作。

服务器必须能够与能够断言status_request或status_request_v2的至少两个不同的客户端实现互操作。目前，大多数主流浏览器（Firefox，Chrome等）都可以生成status_request hello扩展，其他工具（如OpenSSL的s_client）也可以生成。为了实现自动测试，可以创建与NSS和OpenSSL库链接的小型应用程序，以建立与OCSP装订的TLS连接。

##### 风险与假设

默认情况下，将使用JDK中的Java客户端启用OCSP装订功能。但是，对于不能接受status_request或status_request_v2 TLS扩展的TLS服务器，存在潜在的互操作性问题。已经定义了系统或安全属性，以在必要时禁用OCSP装订。

#### JEP 250: Store Interned Strings in CDS Archives
##### 概要

将实习的字符串存储在类数据共享（CDS）档案中。

##### 目标

通过在不同JVM进程之间共享String对象和底层char数组对象来减少内存消耗。

只支持G1 GC的共享字符串。共享字符串需要固定区域，G1是唯一支持固定的HotSpot GC。

只支持具有压缩对象和类指针的64位平台。

启动时间，字符串查找时间，GC暂停时间或使用常规基准测试的运行时性能没有显着降级（<2-3％）。

##### 非目标

减少启动时间不是目标。

不支持其他类型的GC（除了G1）。

不支持32位平台。

##### 动机

目前，当CDS将类存储到存档中时，常量池中的CONSTANT_String项目由UTF-8字符串表示。加载类时，UTF-8字符串按需转换为java.lang.String对象。这可能浪费内存，因为每个内部字符串中的每个字符占用三个字节或更多（String中的两个字节，UTF-8中的1-3个字节）。

另外，因为这些字符串是动态创建的，所以它们不能在JVM进程间轻松共享。

##### 描述
在转储时间，在堆初始化期间，在Java堆中分配指定的字符串空间。对于内部String对象及其底层char数组对象的指针将被修改，就像这些对象来自指定的空间一样，写出实体的字符串表和String对象时。

字符串表被压缩，然后在转储时间存储在存档中。字符串表的压缩技术与共享符号表相同（见JDK-8059510）。常规的窄oop编码和解码用于从压缩字符串表访问共享的String对象。

在具有压缩Oop指针的64位平台上，使用偏移（带或不带缩放）从窄的基座进行编码。目前有四种不同的编码模式：32位非标度，零基，不相交的堆和基于堆的。根据堆大小和堆最小值，选择适当的编码模式。在转储时间和运行时间，窄oop编码模式（包括编码移位）必须相同，以便共享字符串空间中的oop指针在运行时保持有效。在运行时，共享字符串空间可以被视为具有限制的可重定位。它不需要映射到与转储时间相同的地址，但它应该在转储时间和运行时间处于与窄oop base相同的偏移量。只要使用相同的编码模式，堆大小不需要在转储时间和运行时间相同。字符串空间和oop编码模式（和移位）的偏移量应存储在用于运行时验证的归档中。如果编码模式发生变化，则会使每个共享字符串的char数组的oop指针的编码无效。在这种情况下，共享字符串数据将被忽略，而剩余的共享数据仍然可以由VM使用。虚拟机将报告由于GC配置不兼容而未使用共享字符串的警告。

在运行时，字符串空间作为Java堆的一部分映射到与转储时间相同的oop编码基址偏移量。映射从保存在存档中的字符串空间的最低页对齐地址开始。映射的字符串空间包含共享的字符串和字符数组对象。与该映射空间重叠的所有G1区域将被标记为固定;这些G1区域不可用于运行时分配。在映射结束时，可能会在部分重叠的区域中浪费未使用的空间，但最多只能有一个这样的区域。由于使用相同的窄Oop编码，因此无需对字符串空间中的oop指针进行修补。共享字符串空间是可写的，但GC不应写入空间中的oops，以便在不同进程之间保持可共享性。尝试锁定其中一个共享字符串并因此写入共享空间的应用程序将获取该页面的私有副本，从而失去共享该特定页面的好处。这种情况很少见。

共享字符串表在运行时与常规字符串表不同。查找内部字符串时会搜索两个表。共享字符串表是运行时的只读表;没有条目可以添加或删除。

G1字符串重复数据删除表是一个单独的哈希表，其中包含运行时重复数据删除的字符数组。当字符串被内联并添加到StringTable时，该字符串将被重复数据删除，并且底层char数组将添加到重复数据删除表（如果还没有）。重复数据删除表不存储到存档中。使用共享字符串数据在虚拟机启动期间填充重复数据删除表。作为优化，在G1StringDedupThread（在G1StringDedupThread :: run（）中，initialize_in_thread（）之后）完成工作以减少启动时间。共享字符串的哈希值在转储时间预先计算并存储在字符串中，以避免重复数据删除代码在运行时写入哈希值。

##### 测试

测试此功能将涵盖以下几个方面：

此功能的基本操作;

与此功能不兼容的模式，如非G1 GC和未压缩对象/类指针;

转储时间和运行时间之间的普通对象 - 指针编码的变化;

字符串文件格式无效;

使用此功能时选择字符串操作，如实习和字符串比较; 和

确保此功能不会导致使用GC诊断模式的堆损坏。

##### 依赖性

需要更新可用性代理以增加对共享字符串表的支持（参见JDK-8079830）。

随着JDK-8054307提出的改变，底层的char数组将被改为一个字节数组。 将内部字符串复制到字符串空间并执行重复数据删除的代码将需要反映出如果和何时集成了JDK-8054307。 影响应该很小。

#### JEP 251: Multi-Resolution Images
##### 概要

定义多分辨率图像API，以便可以轻松地操纵和显示具有分辨率变体的图像。

##### 描述

要在java.awt.image包中定义的新API将允许将一组具有不同分辨率的图像封装到单个多分辨率图像中。

多分辨率图像的基本操作是：

根据给定的DPI度量和图像转换集合，检索特定于分辨率的图像变体，以及

检索图像中的所有变体。

除了这些操作之外，多分辨率图像否则将以与普通图像相同的方式表现。 java.awt.Graphics类将根据当前显示DPI度量和任何应用的转换从多分辨率图像中检索必要的变体。

拟议的API草图：

```
package java.awt.image;

/**
 * This interface is designed to provide a set of images at various resolutions.
 *
 * The {@code MultiResolutionImage} interface should be implemented by any
 * class whose instances are intended to provide image resolution variants
 * according to the given image width and height.
 *
 * @since 1.9
 */
public interface MultiResolutionImage {

    /**
     * Gets a specific image that is the best variant to represent
     * this logical image at the indicated size.
     *
     * @param destImageWidth the width of the destination image, in pixels.
     * @param destImageHeight the height of the destination image, in pixels.
     * @return image resolution variant.
     *
     * @since 1.9
     */
    Image getResolutionVariant(float destImageWidth, float destImageHeight);

    /**
     * Gets a readable list of all resolution variants.
     * Note that many implementations might return an unmodifiable list.
     *
     * @return list of resolution variants.
     * @since 1.9
     */
    public List<Image> getResolutionVariants();
}
```
##### 备择方案

在当前的Java 2D API中，无法检测是否正在使用高分辨率显示。 至少必须提供DPI比例因子。 开发人员可以使用比例因子来绘制具有必要分辨率的图像，但这可能非常繁琐。

##### 测试

新的API将需要在具有Retina显示器的Mac OS X上进行测试，并在具有HiDPI显示器的Windows上进行测试。

可以测试以下情况：

从一组图像创建多分辨率图像，

从文件系统上的图像创建多分辨率图像，遵循通常的缩放命名约定，

基于另一个多分辨率图像创建多分辨率图像

绘制具有不同DPI度量和应用变换的多分辨率图像。

#### JEP 252: Use CLDR Locale Data by Default
##### 概要

默认情况下，使用Unicode Consortium的公共区域设置数据存储库（CLDR）的区域设置数据。

##### 动机

Unicode Consortium的Common Locale Data Repository是许多平台上的语言环境数据的事实上的标准。 虽然CLDR语言环境数据与JDK 8的JRE捆绑在一起，但默认情况下不启用。 要打开它，最终用户必须显式地设置系统属性java.locale.providers，例如：

```
java.locale.providers=JRE,CLDR
```

通过默认使用CLDR数据，将向用户提供事实上的标准区域设置数据，而无需其他任何进一步的操作。

##### 描述

在默认隐式LocaleProviderAdapter首选项列表的前面插入CLDR。

默认查找顺序将是CLDR，COMPAT，SPI，其中COMPAT在JDK9中指定JRE的区域设置数据。如果特定提供商不能提供所请求的区域设置数据，则搜索将依次进行到下一个提供者。显示字符串的格式化和翻译的本地化模式（如区域设置名称）在某些区域设置可能会有所不同。要启用与JDK 8兼容的行为，请将系统属性java.locale.providers设置为具有CLDR之前的COMPAT的值。

##### 测试

对于JDK 8中不支持的区域设置，区域设置敏感的服务（如日期，时间和数字格式）的行为可能不同。现有测试和应用程序将需要修改。

##### 风险与假设

我们不对CLDR的数据的有效性负责;我们假设这是“好的数据”。

#### JEP 253: Prepare JavaFX UI Controls & CSS APIs for Modularization
##### 概要

为JavaFX UI控件和CSS功能定义公共API，目前仅通过内部API可用，因此由于模块化而变得无法访问。

##### 目标

许多使用JavaFX的UI控件和CSS功能的开发人员历来都忽略了避免内部com.sun。* API的警告。在许多情况下，为了达到预期的效果，开发人员别无选择，只能使用这些内部API。随着即将发布的Java 9，特别是在Project Jigsaw的模块之间引入了强大的界限，开发人员将发现，由于com.sun * *将不再可用，开发人员将不再编译或运行它们的代码。该JEP的目标是为内部API提供的功能定义公共API。

##### 非目标

考虑到模块化的含义，即com.sun。* package的即将到来，无法以保持任何程度的向后兼容性的方式来实现。因此，这个JEP不是保留向后兼容性的目标。这并不意味着我们可以打破任何我们喜欢的事情;我们的目的是仅引入直接由模块边界强制执行的新API（并演变现有的私有API）。不受模块化影响的所有其他现有API将保持不变。

##### 成功指标

成功可以通过两种方式来衡量：

依赖于JavaFX内部API的项目，特别是Scene Builder，ControlsFX和JFXtras，在更新到新的API之后，继续运行，而不会丢失功能。这三个项目都是开放源码，它们将提供一个优秀的测试台，以确保提供所有必要的API。

关于新API的讨论结果，例如控制行为所需的输入映射工作，可以改善开发人员长期以来的功能。最终，如果所有的计划都可以工作，第三方控件应该是可构建的，而不需要依赖内部API。

##### 动机

如果没有完成这项工作，许多项目将需要大大降低其提供的功能，对于某些项目来说，这可能是致命的。例如，无法访问Scene Builder目前拥有的内部API，可能会难以实现 - 肯定其提供CSS样式和控制属性操纵功能的能力将受到严重破坏，而这两个核心Scene Builder的功能部件。大多数其他基于JavaFX的项目都具有相同的参数，具有任何程度的自定义控件或CSS实现。

##### 描述

这个JEP分为两个半相关的子项目，每个项目对于达成最终目标都是重要的。这些项目没有特定的顺序必须进行。

Project One：将UI控制到公共API中

目前，所有的皮肤都位于com.sun.javafx.scene.control.skin中。这意味着，在JDK 9中，没有功能的应用程序将保留延长皮肤的第三方（例如TextFieldSkin）以添加附加功能，覆盖现有方法或以其他方式修改控件外观的视觉或行为，当然，这是依赖于非公开API的用户的错误，但是我们长期以来一直讨论将此API公开化，以便更好地启用UI控件的第三方修改。

目的是将许多JavaFX控件外观移植到适当的公共包中，最可能是javafx.scene.control.skin。没有意图也移动相关的行为类。

这项工作的大部分内容是审查每个现有的皮肤类，确保以下内容：

皮肤之间存在API一致性，

存在高质量的Javadoc和单元测试

通过将公共方法更改为私有，将每个类中的API保持在最低限度。

这项研究已经在一个单独的沙盒回购中已经相当进展，而且耗时，只需要几个关注的类，例如实用程序类，重复类和真正实现的类，需要进一步分析。除此之外，还有一些类可能有资格被引入到javafx.scene.control包中，或至少需要进一步考虑，因为它们不是真正的皮肤。这些包括FXVK（虚拟键盘），ColorPalette，CustomColorDialog，DatePickerContent和InputField。最后，有几个类，理想情况下，方法是私有的，除了其他的只实现类依赖于这个API的事实。解决方案将针对所有这些问题进行调查，并不存在任何主要问题。

在9中将皮肤制作为公共API的目的是确保其持续可用性。 API将被有目的地保持在最低限度，并尽可能大大减少，意图在后续版本中跟随开发人员请求的更有用的API。非常感谢，API（主要是）永远是，所以允许公开皮肤API在几个更新版本中成熟似乎是最好的行动方案。

截至6月中旬，这个项目是几乎所有代码被移动和清理的点。其意图是在7月中旬至8月初期间将其公开发行在JDK 9中。以下是作为公共API移入javafx.scene.control.skin的所有类的列表：
> AccordionSkin
ButtonBarSkin
ButtonSkin
CellSkinBase
CheckBoxSkin
ChoiceBoxSkin
ColorPickerSkin
ComboBoxBaseSkin
ComboBoxListViewSkin
ComboBoxPopupControl
ContextMenuSkin
DateCellSkin
DatePickerSkin
HyperlinkSkin
LabelSkin
LabeledSkinBase
ListCellSkin
ListViewSkin
MenuBarSkin
MenuButtonSkin
MenuButtonSkinBase
NestedTableColumnHeader
PaginationSkin
ProgressBarSkin
ProgressIndicatorSkin
RadioButtonSkin
ScrollBarSkin
ScrollPaneSkin
SeparatorSkin
SliderSkin
SpinnerSkin
SplitMenuButtonSkin
SplitPaneSkin
TabPaneSkin
TableCellSkin
TableCellSkinBase
TableColumnHeader
TableHeaderRow
TableRowSkin
TableRowSkinBase
TableViewSkin
TableViewSkinBase
TextAreaSkin
TextFieldSkin
TextInputControlSkin
TitledPaneSkin
ToggleButtonSkin
ToolBarSkin
TooltipSkin
TreeCellSkin
TreeTableCellSkin
TreeTableRowSkin
TreeTableViewSkin
TreeViewSkin
VirtualContainerBase
VirtualFlow

截至6月中旬，这些课程几乎被剥离了几乎没有继承自SkinBase的API。展望未来，其目的是为了添加有用的API，因为基于早期访问构建的反馈。一些类，例如文本输入控件和虚拟化控件，已经有了额外的API来支持它们的功能。

##### 项目二：审查和公布相关的CSS API

与Project One一样，该项目涉及到目前居住在com.sun。*包中的公共API。同样，这将需要代码审查以最小化API，以及额外的单元测试和更多的文档。

此工作的驱动程序将继续允许Scene Builder在JDK 9中进行编译，并进行适当的修改。

截至6月中旬，这个项目是几乎所有代码被移动和清理的点。其意图是在7月中旬至8月初期间将其公开发行在JDK 9中。以下是作为公共API移入javafx.css的所有类的列表：

```
CascadingStyle.java:public class CascadingStyle implements Comparable<CascadingStyle> {
CascadingStyle.java:    public Style getStyle() {
CascadingStyle.java:    public CascadingStyle(final Style style, Set<PseudoClass> pseudoClasses,
CascadingStyle.java:    public String getProperty() {
CascadingStyle.java:    public Selector getSelector() {
CascadingStyle.java:    public Rule getRule() {
CascadingStyle.java:    public StyleOrigin getOrigin() {
CascadingStyle.java:    public ParsedValueImpl getParsedValueImpl() {

CompoundSelector.java:final public class CompoundSelector extends Selector {
CompoundSelector.java:    public List<SimpleSelector> getSelectors() {
CompoundSelector.java:    public CompoundSelector(List<SimpleSelector> selectors, List<Combinator> relationships)
CompoundSelector.java:    public Match createMatch() {

CssError.java:public class CssError {
CssError.java:    public static void setCurrentScene(Scene scene) {
CssError.java:    public final String getMessage() {
CssError.java:    public CssError(String message) {
CssError.java:    public final static class PropertySetError extends CssError {
CssError.java:        public PropertySetError(CssMetaData styleableProperty,

Declaration.java:final public class Declaration {
Declaration.java:    public ParsedValue getParsedValue() {
Declaration.java:    public String getProperty() {
Declaration.java:    public Rule getRule() {

Rule.java:final public class Rule {
Rule.java:    public final ObservableList<Declaration> getDeclarations() {
Rule.java:    public final ObservableList<Selector> getSelectors() {
Rule.java:    public Stylesheet getStylesheet() {
Rule.java:    public StyleOrigin getOrigin() {

Selector.java:abstract public class Selector {
Selector.java:    public Rule getRule() {
Selector.java:    public void setOrdinal(int ordinal) {
Selector.java:    public int getOrdinal() {
Selector.java:    public abstract Match createMatch();
Selector.java:    public abstract boolean applies(Styleable styleable);
Selector.java:    public abstract boolean applies(Styleable styleable, Set<PseudoClass>[] triggerStates, int bit);
Selector.java:    public abstract boolean stateMatches(Styleable styleable, Set<PseudoClass> state);
Selector.java:    public static Selector createSelector(final String cssSelector) {
Selector.java:    protected void writeBinary(DataOutputStream os, StringStore stringStore)

SimpleSelector.java:final public class SimpleSelector extends Selector {
SimpleSelector.java:    public String getName() {
SimpleSelector.java:    public List<String> getStyleClasses() {
SimpleSelector.java:    public Set<StyleClass> getStyleClassSet() {
SimpleSelector.java:    public String getId() {
SimpleSelector.java:    public NodeOrientation getNodeOrientation() {

Size.java:final public class Size {
Size.java:    public Size(double value, SizeUnits units) {
Size.java:    public double getValue() {
Size.java:    public SizeUnits getUnits() {
Size.java:    public boolean isAbsolute() {
Size.java:    public double pixels(double multiplier, Font font) {
Size.java:    public double pixels(Font font) {
Size.java:    public double pixels() {

Style.java:final public class Style {
Style.java:    public Selector getSelector() {
Style.java:    public Declaration getDeclaration() {
Style.java:    public Style(Selector selector, Declaration declaration) {

Stylesheet.java:public class Stylesheet {
Stylesheet.java:    public String getUrl() {
Stylesheet.java:    public StyleOrigin getOrigin() {
Stylesheet.java:    public void setOrigin(StyleOrigin origin) {
Stylesheet.java:    public List<Rule> getRules() {
Stylesheet.java:    public List<FontFace> getFontFaces() {
Stylesheet.java:    public static Stylesheet loadBinary(URL url) throws IOException {
Stylesheet.java:    public static void convertToBinary(File source, File destination) throws IOException {

CssParser.java:final public class CssParser {
CssParser.java:    public CssParser() {
CssParser.java:    public Stylesheet parse(final String stylesheetText) {
CssParser.java:    public Stylesheet parse(final URL url) throws IOException {
CssParser.java:    public Stylesheet parseInlineStyle(final Styleable node) {
CssParser.java:    public ParsedValueImpl parseExpr(String property, String expr) {
CssParser.java:    public static ObservableList<CssError> errorsProperty() {
```

##### 概要

这两个项目的最终结果是创建：

用于UI控件外观的新包javafx.scene.control.skin经过审查，记录和测试;

必要的CSS相关课程的移动和审查，文档和测试。

##### 测试

测试将仅限于没有特殊平台或硬件要求的附加单元测试。

##### 风险与假设

主要风险在于工作范围超过预期。 已经进行了一些研究，以更好地了解要求，但不能否认，生产好的API将需要合理的时间投资。

#### JEP 254: Compact Strings
概要

为字符串采用更节省空间的内部表示。

##### 目标

提高String类和相关类的空间效率，同时保持大多数情况下的性能，并保持所有相关Java和本机接口的完全兼容性。

##### 非目标

在字符串的内部表示中使用诸如UTF-8之类的替代编码不是一个目标。随后的JEP可以探索这种方法。

##### 动机

String类的当前实现将字符存储在char数组中，每个字符使用两个字节（十六位）。从许多不同的应用程序收集的数据表明字符串是堆使用的主要组成部分，此外，大多数String对象只包含Latin-1个字符。这样的字符只需要一个字节的存储空间，因此这些String对象的内部char数组中的一半空间将不被使用。

##### 描述

我们建议将String类的内部表示从UTF-16字符数组更改为字节数组加编码标志字段。新的String类将根据字符串的内容存储编码为ISO-8859-1 / Latin-1（每个字符一个字节）或UTF-16（每个字符两个字节）的字符。编码标志将指示使用哪个编码。

与AbstractStringBuilder，StringBuilder和StringBuffer相关的字符串相关类将被更新为使用相同的表示形式，HotSpot VM的内部字符串操作也将使用相同的表示形式。

这纯粹是一个实现变更，而不改变现有的公共接口。没有计划添加任何新的公共API或其他接口。

迄今为止完成的原型设计工作确认了内存占用量的预期减少，GC活动的大幅度减少以及某些角落中的较小的性能回归。

有关详细信息，请参阅：

字符串密度表现状态
字符串密度对SPARC上SPECjbb2005的影响
##### 备择方案

我们尝试使用-XX标志启用JDK 6更新版本中的“压缩字符串”功能。启用时，String.value已更改为Object引用，并将指向字节数组，仅包含7位US-ASCII字符的字符串，或者指向字符数组。这种实现不是开源的，所以很难维护并保持与主线JDK源的同步。它已被删除。

##### 测试

彻底的兼容性和回归测试对于改变平台这样一个基本部分至关重要。

我们还需要确认我们已经实现了这个项目的绩效目标。需要分析内存节省。性能测试应使用范围广泛的工作负载进行，从重点微距基准到大型服务器工作负载。

我们将鼓励整个Java社区对此进行早期测试，以确定任何剩余的问题。

##### 风险与假设

优化内存的字符存储可能会在运行时性能方面取得平衡。我们预计这将被GC活动降低所抵消，我们将能够保持典型服务器基准测试的吞吐量。如果没有，我们将调查可以在内存保存和运行时性能之间达到可接受的平衡的优化。

其他最近的项目已经减少了字符串使用的堆空间，特别是G1中的JEP 192：字符串重复数据删除。即使删除重复，如果编码更有效，剩余的字符串数据可以减少占用空间。我们假设这个项目仍将提供与所需努力相称的好处。

#### JEP 255: Merge Selected Xerces 2.11.0 Updates into JAXP
##### 概要

升级包含在JDK中的Xerces XML解析器的版本与Xerces 2.11.0的重要更改。

##### 非目标

JDK的Xerces副本包含许多JDK特定的更改和改进。将JDK代码库与Apache Xerces Project完全同步并不是一个目标。

##### 成功指标

所选类别中的课程已完全更新。

不介绍不相容性。与现有JDK代码的任何冲突都得到解决，以便优先使用现有的JDK代码。

回归不会发生。

##### 动机

JDK包含较旧的Xerces 2.7.1解析器。在开发JDK 7期间，更新了Xerces 2.10.0的所有关键和许多重大变化。此后，Xerces 2.11.0发布。升级到最新版本将有助于提高JDK实现的质量。

##### 描述

从Xerces 2.11.0更新以下类别的更改JDK：

数据类型，
DOM L3串行器，
XPointer的，
目录解析器，和
XML模式验证（包括错误修复，但不是XML Schema 1.1开发代码）。
JAXP公共API将不会有任何变化。

此更新将分批完成。不是每个修订都可以单独测试。

##### 测试

Xerces中的相关测试可能会合并到现有测试套件中。

将开发新的测试来确保充分覆盖变化。

将需要进行重大测试，以确保更新不会对现有实施产生回归。

##### 风险与假设

由于将Xerces 2.7.1集成到JDK 6中，许多JDK特定的错误修复，改进和功能添加都是独立于Apache Xerces Project实现的。这包括并入StAX解析器，该解析器共享并修改了Xerces的扫描器实现。因此，某些Xerces修补程序可能需要修改才能解决冲突。

#### JEP 256: BeanInfo Annotations
##### 概要

用适当的注释替换@beaninfo Javadoc标签，并在运行时处理这些注释以动态生成BeanInfo类。

##### 动机

简化自定义BeanInfo类的创建，并实现客户端库的模块化。

##### 描述

大多数BeanInfo类在运行时自动生成，但许多Swing类仍然在编译时从@beaninfo Javadoc标签生成BeanInfo类。 我们建议使用以下注释替换@beaninfo标签，并扩展现有的内省算法来解释它们：

```
package java.beans;
public @interface JavaBean {
    String description() default "";
    String defaultProperty() default "";
    String defaultEventSet() default "";
}

package java.beans;
public @interface BeanProperty {
    boolean bound() default true;
    boolean expert() default false;
    boolean hidden() default false;
    boolean preferred() default false;
    boolean visualUpdate() default false;
    String description() default "";
    String[] enumerationValues() default {};
}

package javax.swing;
public @interface SwingContainer {
    boolean value() default true;
    String delegate() default "";
}
```
有关详细信息，请参阅JavaBean，BeanProperty和SwingContainer的Javadoc。

这些注释将在运行时在BeanInfo生成期间设置相应的特征属性。开发人员可以轻松地在Bean类中直接指定这些属性，而不是为每个Bean类创建一个单独的BeanInfo类。它还将允许删除自动生成的类，这将使得更容易模块化客户端库。

##### 测试

我们需要验证新的内省算法是否符合预期。我们还需要验证新的内省算法不会破坏向后兼容性，否则确保兼容性破坏的情况很少见。

##### 风险与假设

新的内省算法可能会有所不同，但我们预计不会有任何严重的后向不兼容性。

我们不希望任何性能下降。反思算法的重构可能会提高性能。

##### JEP 257: Update JavaFX/Media to Newer Version of GStreamer
##### 概要

更新FX / Media中包含的GStreamer版本，以提高安全性，稳定性和性能。

##### 非目标

引入任何新功能（例如动态管道）并不是一个目标。

##### 动机

今天FX / Media中包含的GStreamer版本已经过期，自2011年6月以来，GStreamer的C代码缺少稳定性和性能修正。从目前的0.10.35版本更新为1.x版本的GStreamer需要时间并努力，我们希望主动更新，以便我们能够更轻松地回应未来的GStreamer安全更新。

##### 描述

GStreamer的最新稳定版本是1.4.4，于2014/11/10发布。集成此版本将需要更新我们现有的插件，因为它包含不兼容的API更改。构建GStreamer pipline的JFXMedia层不应该需要很多更改;它最有可能与最新的GStreamer兼容。

我们还需要更新FX / Media中包含的GLib的副本。 （这在Windows和OS X上使用，它们本身不提供GLib;在Linux上我们使用发行版提供的GLib。）我们的GLib副本目前是2.28.8版本;新的GStreamer至少需要2.32，但最好采用最新版本，所以我们将更新到GLib 2.42.1，以及LibFFI 3.2.1。

##### 测试

不需要新的测试。

#### JEP 258: HarfBuzz Font-Layout Engine
##### 概要

用HarfBuzz替换现有的ICU OpenType字体布局引擎。

##### 动机

OpenType布局引擎（如ICU或HarfBuzz）提供脚本感知代码来处理为阿拉伯语和印度语等脚本正确呈现文本所需的某些字体表。没有这种支持，这些脚本中的文本呈现不仅仅是错误的，而是难以辨认。

ICU OpenType布局引擎不再积极开发，而IBM，项目所有者打算将其替换为ICU项目中的HarfBuzz。 JDK必须遵守，否则使用不受支持和过时的库。

##### 描述

将JDF中的HarfBuzz库的拷贝与JDK中的ICU和其他开放源代码库（如libpng）相同。集成该本地库来支持复杂文本布局所需的Java API和实现，以替代ICU。

##### 备择方案

HarfBuzz没有可行的开源替代方案。像Pango这样的库现在正在以HarfBuzz周围的包装器重新实现。

##### 风险与假设

由于不同的实现和使用更新的OpenType规范，库之间可能会有一些轻微的渲染差异。

#### JEP 259: Stack-Walking API
##### 概要

定义一个有效的标准API，用于堆栈行走，允许轻松过滤和懒惰地访问堆栈跟踪中的信息。

##### 非目标

转换JDK中所有现有的堆栈步行代码来使用这个新的API并不是一个目标。
##### 动机

没有标准的API有效地遍历执行堆栈上的选定帧，并访问每个帧的Class实例。

现有的API提供对线程堆栈的访问：

Throwable :: getStackTrace和Thread :: getStackTrace返回一个StackTraceElement对象的数组，它包含每个stack-trace元素的类名和方法名。

SecurityManager :: getClassContext是一个受保护的方法，它允许SecurityManager子类访问类上下文。

这些API要求虚拟机热切捕获整个堆栈的快照，并返回表示整个堆栈的信息。如果呼叫者只对堆栈中的前几帧感兴趣，则无法避免检查所有帧的成本。 Throwable :: getStackTrace和Thread :: getStackTrace方法都返回一个StackTraceElement对象的数组，它包含类名和方法名，而不是实际的Class实例。对于对整个堆栈感兴趣的应用程序，规范允许VM实现在堆栈中省略一些帧来执行性能。换句话说，Thread :: getStackTrace可能会返回部分堆栈跟踪。

这些API不满足当前依赖于JDK内部sun.reflect.Reflection :: getCallerClass方法的用例，否则它们的性能开销是不能容忍的。这些用例包括：

走栈，直到找到直接的调用者的类。每个JDK调用者敏感的API都会查找其直接调用者的类，以确定API的行为。例如，Class :: forName和ResourceBundle :: getBundle方法使用立即调用者的类加载器分别加载一个类和一个资源束。诸如Class :: getMethod的反射API使用直接调用者的类加载器来确定要执行的安全检查。

步行堆栈，过滤出特定实现类的堆栈帧，以找到第一个未过滤的帧。 java.util.logging API，Log4j和Groovy运行时过滤器中间堆栈帧（通常是实现特定的和反射帧）来查找调用者的类。

步行堆栈查找所有保护域，直到达到第一个特权框架。为了进行权限检查，这是必需的。

走整个堆栈，可能有一个深度限制。这是生成任何Throwable对象的堆栈跟踪，并实现Thread :: dumpStack方法所必需的。

##### 描述

该JEP将定义一个可以进行懒惰和框架过滤的堆栈行走API，支持在符合给定条件的框架上停止的短步行，还支持遍历整个堆栈的长行程。

将增强JVM以提供灵活的机制来遍历和实现所需的堆栈帧信息，并在需要时允许对其他堆栈帧进行高效的延迟访问。本地JVM转换将被最小化。实现需要对线程堆栈有一个稳定的视图：返回一个持有堆栈指针的流以不受控制的方式进一步操作将不起作用，因为一旦流工厂返回，JVM将可以自由地重组控件堆栈（例如通过去优化）。这将影响API的定义。

当使用安全管理器运行时，API将指定其行为，以便对堆栈帧中的Class对象的访问不会危及安全性。

该建议是定义一个基于能力的StackWalker API来遍历堆栈。每个StackWalker对象的构造而不是每次使用时都会执行安全许可检查。它将定义以下方法：

```
public <T> T walk(Function<Stream<StackFrame>, T> function);
public Class<?> getCallerClass();
```
walk方法打开当前线程的StackFrame的顺序流，然后将该函数应用于StackFrame流。 流的分流器以有序的方式执行堆栈帧遍历。 Stream <StackFrame>对象只能遍历一次，当walk方法返回时，它将被关闭。 一旦关闭，流将无效使用。 例如，找到第一个调用者过滤一个已知的实现类列表：

```
Optional<Class<?>> frame = new StackWalker().walk((s) ->
{
    s.filter(f -> interestingClasses.contains(f.getDeclaringClass()))
     .map(StackFrame::getDeclaringClass)
     .findFirst();
});
```
为了快照当前线程的堆栈跟踪，

```
List<StackFrame> stack =
     new StackWalker().walk((s) -> s.collect(Collectors.toList()));
```
getCallerClass（）方法是为了方便查找调用者的框架，并且是sun.reflect.Reflection.getCallerClass的替代。使用walk方法获取调用者类的方法是：

```
walk((s) -> s.map(StackFrame::declaringClass).skip(2).findFirst());
```
##### 备择方案

一个替代的API选择将是walk方法返回Stream <StackFrame>。这样的替代方案将不起作用，因为返回的流对象可以以不受控制的方式用于进一步的操纵。当创建堆栈帧流时，只要流工厂返回，JVM可以自由地重组控制堆栈（例如通过去优化），并且没有强大的方法来检测堆栈是否已被突变。

相反，类似于AccessController :: doPrivileged，必须至少创建一个本地方法，它将建立自己的堆栈帧，然后为旧的帧提供对JVM堆栈行走逻辑的受控访问。当此本机方法返回时，该功能必须被禁用，或以其他方式使其无法访问。这样，我们可以在线程自己的控制堆栈的稳定视图上对堆栈帧进行高效的懒惰访问。

#### JEP 260: Encapsulate Most Internal APIs
##### 概要

使大多数JDK的内部API默认情况下无法访问，但留下一些关键的，广泛使用的内部API可访问，直到所有或大部分功能都支持替换。

##### 非目标

本JEP本身不会为任何内部API提出替代方案;这些工作将由单独的JEP和适当时的JSR覆盖。

此JEP不承诺保留任何内部API在版本之间的兼容性;他们仍然不稳定，如有变更，恕不另行通知。

##### 动机

一些流行的库使用非标准，不稳定和不受支持的API，这些API是JDK的内部实现细节，并不是外部使用的。通过利用即将推出的模块系统（JEP 200）来限制对这些API的访问将提高平台的完整性和安全性，因为许多这些内部API定义了特权，安全敏感的操作。从长远来看，这一变化将降低JDK本身的维护者以及知道或不知情的使用这些内部API的图书馆和应用程序的维护者所承担的成本。

##### 描述

基于对包括Maven Central在内的各种大量代码的分析，以及从发布JDK 8及其依赖关系分析工具（jdeps）以来收到的反馈，我们可以将JDK的内部API分为两大类：

那些似乎不被JDK以外的代码使用的代码，或者为了方便起见而被外部代码所使用，也就是说，在支持的API中可以使用或者可以很容易地由库提供的功能（例如，sun.misc.BASE64Decoder ）;和

那些提供关键功能（即使不是不可能）在JDK本身之外实现（例如sun.misc.Unsafe）也是困难的。

在JDK 9中，我们建议：

默认情况下封装所有非关键内部API：定义它们的模块不会导出其包以供外部使用。 （通过编译时和运行时的命令行标志，最终可以访问这些API，除非由于其他原因修改或删除了这些API）。

封装关键的内部API，JDK 8中以相同的方式和相同的最后手段解决方案存在受支持的替换。 （支持的替换是Java SE 8标准的一部分（即在java。*或javax。*包中）的一部分，或者是JDK特定的，并且用@ jdk.Exported注释（通常在com.sun中）。 *或jdk。* package））

不封装关键的内部API，JDK 8中不存在支持的替换，并且还禁止在JDK 10中支持替换JDK 9中的替换，甚至可能将其删除。

提出在JDK 9中保持访问的关键内部API是：

sun.misc {信号，SignalHandler}

sun.misc.Unsafe（此类中许多方法的功能现在可通过变量句柄（JEP 193）获得）。

sun.reflect.Reflection :: getCallerClass（int）（此方法的功能可以通过JEP 259以标准形式提供。）

sun.reflect.ReflectionFactory.newConstructorForSerialization

欢迎对此列表的建议添加，由真实用例和开发人员和最终用户影响的估计证明。

上述关键的内部API将被放置在JDK特定的名为jdk.unsupported的模块中，并将它们的包导出和打开。该模块将以完整的JRE和JDK映像存在。因此，默认情况下，这些API可以访问类路径上的代码，如果这些模块声明依赖于jdk.unsupported模块，则可以在模块中进行代码访问。

如上所述，其中一些内部API的替换已经全部或部分存在于JDK 9中。强烈建议开发人员在JDK 9早期访问版本中测试这些替换，并在替换不足的情况下发送反馈可以改进。

在JDK 9中引入替换的关键内部API将在JDK 9中被废弃，并在JDK 10中被封装或删除。

jdk.unsupports导出和打开sun.misc和sun.reflect的后果是：

sun.misc和sun.reflect包中的非关键内部API将被移动或适当删除，因为它们不可访问

标准和JDK模块不能依赖于jdk.unsupported

使用JDK 9中存在替换的关键内部API的库的维护者可能希望使用多版本JAR文件（JEP 238），以便在JDK 9之前的版本上发布使用旧API的单个工件，并将替换的API on以后的版本。

##### 风险与假设

如果在发布JDK 9的时候没有识别出一些广泛使用的关键内部API，则依赖于它的应用程序将失败。这种情况的短期解决方法将是最终用户通过上述命令行标志暴露API;从长远来看，在JDK 9更新版本中，API可以移动到jdk.unsupported模块并导出以供外部使用。
由于jdk.unsupported模块将导出并打开sun.misc和sun.reflect，所以这些软件包中的所有非关键内部API都将被适当移动或删除。如果删除，将不再可见或可访问。如果移动，它们将通过使用命令行标志来访问，但是它们的完全限定名称将被更改，也就是说它们将被包含在另一个名称中。

除了提出的关于sun.reflect的关键API之外，所述包包含实现java.lang（.reflect）子系统的机制。该机构将转移到基本模块中的内部，非出口的包装。因此，反射调用的堆栈跟踪将显得有些不同。也就是说，代表反射实现的堆栈框架将看到它们的类名称（StackTraceElement.getClassName（））从sun.reflect.XXX更改为jdk.internal.reflect.XXX。基于堆栈跟踪元素的类名称的任何代码分析或过滤都应该适当更新，以便处理此事。详见8137058。

#### JEP 261: Module System
##### 概要

实现JSR 376指定的Java平台模块系统，以及相关的JDK特定的更改和增强功能。

##### 描述

Java平台模块系统（JSR 376）提出了Java编程语言，Java虚拟机和标准Java API的更改和扩展。该JEP将实施该规范。因此，javac编译器，HotSpot虚拟机和运行时库将实现模块作为基本的新类型的Java程序组件，并为开发各个阶段的模块提供可靠的配置和强大的封装。

此JEP还将更改，扩展和添加与编译，链接和执行相关的JDK特定工具和API，这些工具和API不属于JSR的范围。其他工具和API（例如javadoc工具和Doclet API）的相关更改将成为单独JEP的主题。

该JEP假定读者熟悉最新的模块系统状态文档以及其他项目拼图JEP：

200: The Modular JDK
201: Modular Source Code
220: Modular Run-Time Images
260: Encapsulate Most Internal APIs
282: jlink: The Java Linker
##### 相

对于编译时间（javac命令）和运行时间（java运行时启动器）的熟悉阶段，我们添加了链接时间的概念，两者之间是可选阶段，其中可以将一组模块组装和优化为自定义运行时映像。链接工具jlink是JEP 282的主题;由javac和java实现的许多新的命令行选项也由jlink实现。

##### 模块路径

javac，jlink和java命令以及其他几个命令现在可以接受选项来指定各种模块路径。模块路径是一个序列，其每个元素都是模块定义或包含模块定义的目录。每个模块定义都是

模块工件，即包含编译模块定义的模块化JAR文件或JMOD文件，否则

分解模块目录，按照惯例，名称是模块的名称，其内容是与包层次结构对应的“分解”目录树。

在后一种情况下，目录树可以是一个编译的模块定义，填充有单独的类和资源文件以及根文件的一个module-info.class文件，或者在编译时，源模块定义，填充有单独的源文件和module-info.java文件在根目录下。

像其他类型的路径一样，模块路径由主机平台的路径分隔符（在Windows上大多数平台上为';'）分隔的路径名字串指定。

模块路径与类路径非常不同：类路径是定位各种类型和资源的方法，而模块路径是定位整个模块的一种手段。类路径的每个元素是类型和资源定义的容器，即JAR文件或分解的包分层目录树。相反，模块路径的每个元素是模块定义或目录，目录中的每个元素都是模块定义，即类型和资源定义的容器，即模块化JAR文件，JMOD文件，或分解模块目录。

在解决过程中，模块系统通过根据相位搜索沿着若干不同路径定位模块，并且还可以按照以下顺序搜索内置到环境中的已编译模块：

编译模块路径（由命令行选项--module-source-path指定）包含源代码形式的模块定义（仅限编译时间）。

升级模块路径（--upgrade-module-path）包含用于代替内置于环境中的可升级模块（编译时间和运行时间）的模块的编译定义。

系统模块是内置在环境中的编译模块（编译时间和运行时间）。这些通常包括Java SE和JDK模块，但在自定义链接图像的情况下，还可以包括库和应用程序模块。在编译时，可以通过-system选项覆盖系统模块，该选项指定要加载系统模块的JDK映像。

应用程序模块路径（--module-path或简称为-mp）包含库和应用程序模块的编译定义（所有阶段）。在链接时，此路径还可以包含Java SE和JDK模块。

这些路径上存在的模块定义与系统模块一起定义了可观察模块的范围。

在模块路径中搜索特定名称的模块时，模块系统将首先定义该名称的模块。版本字符串（如果存在）被忽略;如果模块路径的元素包含具有相同名称的多个模块的定义，则解析将失败，并且编译器，链接器或虚拟机将报告错误并退出。构建工具的责任

##### 根模块

模块系统通过解决相对于可观察模块集合的一组根模块的依赖关系的传递闭包来构建模块图。

当编译器编译未命名模块中的代码时，或者调用java启动器，并将应用程序的主类从类路径加载到应用程序类加载器的未命名模块中，则未命名模块的默认根模块计算如下：

java.se模块是一个根，如果它存在。如果不存在，则升级模块路径上的每个java。*模块或导出至少一个软件包的系统模块（不含资格）都是根。

升级模块路径上的每个非java。*模块或导出至少一个软件包的系统模块（不含资格）也是根。

否则，默认的根模块集取决于阶段：

在编译时通常是正在编译的模块集（下面更多的）;

在链接的时候它是空的和

在运行时，它是应用程序的主要模块，通过--module（或-m for）启动器选项指定。

有时需要将模块添加到默认的根集，以确保模块图中将包含特定的平台，库或服务提供者模块。在任何阶段的选项

```
--add-modules <module>(,<module>)*
```
其中<module>是模块名称，将命名的模块添加到默认的根模块中。

作为运行时的特殊情况，如果<module>是ALL-DEFAULT，那么如上定义的未命名模块的默认的根模块将被添加到根集中。当应用程序是承载其他应用程序的容器时，这是有用的，这些应用程序又可以依赖于容器本身不需要的模块。

作为运行时的另一个特殊情况，如果<module>是ALL-SYSTEM，则将所有系统模块添加到根集，无论它们是否在默认集中。测试线有时需要这样做。此选项将导致许多模块得到解决;一般来说，ALL-DEFAULT应该是首选的。

作为最后的特殊情况，如果<module>是ALL-MODULE-PATH，则将相关模块路径上找到的所有可观察模块都添加到根集中。 ALL-MODULE-PATH在编译时和运行时都有效。这被提供供构建工具（如Maven）使用，Maven已经确保需要模块路径上的所有模块。将自动模块添加到根集中也是一种方便的方法。

##### 限制可观察模块

当主模块是由类路径的应用程序类加载器定义的未命名模块时，限制可观察模块（例如调试）或减少解析模块的数量有时是有用的。可以在任何阶段使用--limit-modules选项来执行此操作。其语法是：

```
--limit-modules <module>(,<module>)*
```
其中<module>是一个模块名称。此选项的作用是将可观察模块限制在命名模块的传递性关闭以及主模块（如果有的话）以及通过--add-modules选项指定的任何其他模块中。

（对于--limit-modules选项的解释而计算的传递闭包是临时结果，仅用于计算可观察模块的有限集合，将再次调用解析器以计算实际的模块图。）

##### 提高可读性

当测试和调试时，有时需要安排一个模块来读取其他模块，即使第一个模块在其模块声明中不依赖于第二个模块的require子句。这可能是需要的，例如，为了使待测模块访问测试线束本身，或者访问与线束相关的库。可以在编译时和运行时使用--add-reads选项来执行此操作。其语法是：

```
--add-reads <source-module>=<target-module>
```
其中<source-module>和<target-module>是模块名称。

--add-reads选项可以多次使用。每个实例的作用是从源模块向目标模块添加可读性边。这实际上是一个模块声明中的require子句的命令行形式，或者是对Module :: addReads方法的无限制形式的调用。因此，源模块中的代码将能够访问目标模块的包中的类型，只要每个包都通过源模块声明中的exports子句导出，调用Module :: addExports方法或--add-exports选项的实例（定义如下）。

例如，如果测试工具将一个白盒测试类注入到java.management模块中，并且该类在（假设的）testng模块中扩展了一个导出的实用程序类，则可以通过该选项授予所需的访问权限

```
--add-reads java.management=testng
```
作为特殊情况，如果<target-module>是ALL-UNNAMED，则可读性边缘将从源模块添加到所有当前和未来的未命名模块，包括对应于类路径的模块。这允许模块中的代码由尚未自动转换为模块化形式的测试框架进行测试。

##### 打破封装

有时候有必要违反模块系统定义的访问控制边界，并由编译器和虚拟机强制执行，以便允许一个模块访问另一个模块的某些未注册的类型。这可能是期望的，以便例如实现内部类型的白盒测试，或者将不支持的内部API暴露给依赖于它们的代码。可以在编译时和运行时使用--add-exports选项来执行此操作。其语法是：

```
--add-exports <source-module>/<package>=<target-module>(,<target-module>)*
```
其中<source-module>和<target-module>是模块名称，<package>是包的名称。

--add-exports选项可以多次使用，但对于源模块和程序包名称的任何特定组合，最多可以使用一次。每个实例的作用是将命名包的合格导出从源模块添加到目标模块。这实质上是模块声明中的export子句的命令行形式，或者是对Module :: addExports方法的无限制形式的调用。因此，如果目标模块通过其模块声明中的require子句读取源模块，则可以访问源模块的命名包中的类型，调用Module :: addReads方法或--add-reads选项的实例。

例如，如果模块jmx.wbtest包含java.management模块的未导出的com.sun.jmx.remote.internal包的白盒测试，则可以通过该选项授予它所需的访问权限

```
--add-exports java.management/com.sun.jmx.remote.internal=jmx.wbtest
```
作为特殊情况，如果<target-module>是ALL-UNNAMED，则源包将被导出到所有未命名的模块，无论它们是否始终存在或稍后创建。因此，可以通过该选项将对java.management模块的sun.management包的访问权限授予类路径上的所有代码

```
--add-exports java.management/sun.management=ALL-UNNAMED
```
必须非常小心地使用--add-exports选项。您可以使用它来访问库模块或甚至JDK本身的内部API，但是您自担风险：如果内部API更改或删除，则库或应用程序将失败。

##### 修补模块内容

在测试和调试时，有时候用替代或实验版本替换特定模块的所选类文件或资源，或提供全新的类文件，资源甚至包。这可以通过--patch-module选项在编译时和运行时完成。其语法是：

```
--patch-module <module>=<file>(<pathsep><file>)*
```
其中<module>是模块名，<file>是模块定义的文件系统路径名，<pathsep>是主机平台的路径分隔符。

--patch-module选项可以多次使用，但对于任何特定的模块名称最多可以使用一次。每个实例的作用是改变模块系统如何在指定的模块中搜索类型。在检查实际模块之前，无论是部分系统还是在模块路径上定义，它首先按顺序检查为选项指定的每个模块定义。补丁路径命名模块定义序列，但它不是模块路径，因为它具有类似路径的类似语义的“泄漏”。这允许测试线束，例如，将多个测试注入同一个软件包，而不必将所有测试复制到单个目录中。

--patch-module选项不能用于替换module-info.class文件。如果在补丁路径上的模块定义中找到了一个module-info.class文件，则会发出一个警告，该文件将被忽略。

如果修补程序路径中的模块定义中找到的程序包尚未由该模块导出，那么它仍然不会被导出。它可以通过反射API或--add-exports选项显式导出。

--patch-module选项替代了已经删除的-Xbootclasspath：/ p选项（见下文）。

--patch-module选项仅用于测试和调试。
强烈不鼓励在生产环境中使用它。

##### 编译时间

javac编译器实现上述选项，适用于编译时：--module-source-path，--upgrade-module-path，-system，--module-path，--add-modules，--limit-模块，--add-reading，--add-exports和-patch-module。

编译器以三种模式之一运行，每种模式都执行其他选项。

当由-source，-target和-release选项定义的编译环境小于或等于8时，将启用传统模式。可以使用上述模块化选项。
在传统模式下，编译器的行为与JDK 8基本相同。

当编译环境为9或更高版本并且未使用--module-source-path选项时，单模块模式被启用。可以使用上述其他模块化选项;现有选项-bootclasspath，-Xbootclasspath，-extdirs，-endorseddirs和-XXuserPathsFirst可能不被使用。
单模块模式用于编译组织在传统的包分层目录树中的代码。对于形式的传统模式的简单使用是自然的替代

```
$ javac -d classes -classpath classes -sourcepath src Foo.java
```
如果在命令行中指定了module-info.java或module-info.class文件形式的模块描述符，或者在源路径或类路径上找到，则源文件将被编译为由该描述符命名的模块，该模块将是唯一的根模块。否则如果--module <module>选项存在，那么源文件将被编译为<module>的成员，它将是根模块。否则，源文件将被编译为未命名模块的成员，根模块将按照上述方式进行计算。

在这种模式下，可以在类路径上放置任意类和JAR文件，但不推荐这样做，因为它相当于将这些类和JAR文件作为正在编译的模块的一部分进行处理。

当编译环境为9或更高版本并且使用了--module-source-path选项时，多模块模式被启用。还必须使用用于命名输出目录的现有-d选项;可以使用上述其他模块化选项;现有选项-bootclasspath，-Xbootclasspath，-extdirs，-endorseddirs和-XXuserPathsFirst可能不被使用。
多模块模式用于编译一个或多个模块，其源代码在模块源路径上的分解模块目录中布局。在此模式下，类型的模块成员资格由其源文件在模块源路径中的位置确定，因此在命令行上指定的每个源文件必须存在于该路径的元素中。根模块集是指定至少一个源文件的模块集。

与其他模式相反，在此模式下，必须通过-d选项指定输出目录。输出目录将被构造为模块路径的元素，即它将包含自己包含类和资源文件的分解模块目录。如果编译器在模块源路径上找到一个模块，但是在该模块中找不到某种类型的源文件，那么它将在输出目录中搜索相应的类文件。

在大型系统中，特定模块的源代码可以分布在几个不同的目录中。在JDK本身中，例如，模块的源文件可以在任何一个目录src / <module> / share / classes，src / <module> / <os> / classes或build / gensrc / module>，其中<os>是目标操作系统的名称。为了在模块源路径中表达，同时保留模块标识，我们允许这样一个路径的每个元素使用大括号（{和}）括住逗号分隔的备选列表和一个星号（*）来代替模块名称。然后可以将JDK的模块源路径写为

```
{src/*/{share,<os>}/classes,build/gensrc/*}
```
##### 包装：模块化JAR文件

可以使用jar工具来创建模块化的JAR文件，因为模块化的JAR文件只是一个JAR文件，其根目录下有一个module-info.class文件。

该jar工具实现以下新选项，以便在模块打包时将其他信息插入到模块描述符中：

--main-class = <class-name>或-e <class-name>，将<class-name>记录在module-info.class文件中，作为包含模块的public static void main的类入口点。 （这不是一个新的选项;它已经将主类记录在JAR文件的清单中。）

--module-version = <version>将将<version>作为模块的版本字符串记录在module-info.class文件中。

--hash-modules = <pattern>使得在一组特定的可观察模块中依赖于该模块的特定模块的内容的散列将被记录在module-info.class文件中，以供以后用于验证依赖。 仅对名称与正则表达式<pattern>匹配的模块记录哈希。 如果使用此选项，则还必须使用--- module-path选项来指定可观察模块的集合，以便计算依赖于此模块的模块。
此外，新的--print-module-descriptor选项（或简称-p）将显示现有JAR文件的模块描述符（如果有）。

jar工具的--help选项可用于获取其命令行选项的完整摘要。

##### Packaging: JMOD files
新的JMOD格式超出了JAR文件，包括本机代码，配置文件以及其他类型的数据，如果有的话，这些数据根本不适合JAR文件。 JMOD文件用于封装JDK本身的模块;如果需要，开发人员也可以使用它们来打包自己的模块。 JMOD文件的最终格式是一个开放的问题，但现在它基于ZIP文件。

JMOD文件可以在编译时和链接时使用，但不能在运行时使用。为了在运行时支持它们，一般来说，我们将准备提取和链接本地代码库。这在大多数平台上是可行的，尽管它可能非常棘手，我们还没有看到需要这种功能的许多用例，所以为了简单起见，我们选择限制这个版本中的JMOD文件的实用性。

一个新的命令行工具jmod可用于创建JMOD文件并列出现有文件的内容。其一般语法是：

```
$ jmod (create|list|describe) <options> <jmod-file>
```
列表和描述子命令不接受任何选项;对于create子命令，<options>可以包括上面对jar工具描述的--main-class，--module-version，--hash-modules和--- module-path选项，还有：

--class-path <path>指定一个类路径，其内容将被复制到生成的JMOD文件中。

--cmds <path>指定一个或多个包含要复制的本地命令的目录。

--config <path>指定一个或多个包含要复制的配置文件的目录。

--libs <path>指定一个或多个包含要复制的本机库的目录。

--os-arch <arch>指定要记录在module-info.class文件中的操作系统体系结构。

--os-name <os>指定要记录在module-info.class文件中的操作系统名称。

--os-version <version>指定要记录在module-info.class文件中的操作系统版本。

jmod工具的--help选项可用于获取其命令行选项的完整摘要。

##### 链接时间

JEP 282中描述了命令行链接工具jlink的详细信息。在高级别中，其通用语法是：

```
$ jlink <options> ---module-path <modulepath> --output <path>
```
--- module-path选项指定链接器要考虑的可观察模块集，--output选项指定将包含生成的运行时映像的目录的路径。其他<options>可以包括上述的--- limit-modules和--- add-modules选项，以及其他链接器特定的选项。

jlink工具的--help选项可用于获取其命令行选项的完整摘要。

##### 运行

HotSpot虚拟机实现上述选项，适用于运行时：-upgrade-module-path，--module-path，--add-modules，--limit-modules，--add-reads， - add-exports和-patch-module。这些选项可以传递给命令行启动器，java以及JNI调用API。

这个阶段的特定的附加选项和启动器支持的是：

- 模块<module>或-m <module>简称为模块化应用程序的主模块。这将是默认的根模块，用于构建应用程序的初始模块图。如果主模块的描述符不指示主类，则可以使用语法<module> / <class>，其中<class>命名包含应用程序的public static void main入口点的类。

--list-modules以与java -version相同的方式显示可观察模块的名称和版本字符串，然后退出。

--list-modules <module>（，<module>）*显示命名模块的完整模块描述符，如果可见，然后退出。

启动器支持的其他诊断选项包括：

-Xdiag：解析器使模块系统在构建初始模块图时描述其活动。

-Dsun.reflect.debugModuleAccessChecks导致当java.lang.reflect API中的访问检查失败，并显示IllegalAccessException或InaccessibleObjectException时，将显示线程转储。当故障的根本原因被隐藏时，这是非常有用的，因为异常被捕获而不被重新抛出。

-Xlog：modules = [debug | trace]使VM在运行时模块图中定义和更改模块时记录调试或跟踪消息。这些选项在启动过程中产生大量的输出。

在运行时为异常生成的堆栈跟踪已经扩展到包括相关模块的名称和版本字符串（如果存在）。 ClassCastException，IllegalAccessException和IllegalAccessError等异常的详细信息字符串也已被更新为包含模块信息。正在开展与其他类型诊断信息类似增强工作。

一个扩展的例子

假设我们有一个应用程序模块com.foo.bar，它取决于库模块com.foo.baz。如果我们在模块路径目录src中有两个模块的源代码：

```
src/com.foo.bar/module-info.java
src/com.foo.bar/com/foo/bar/Main.java
src/com.foo.baz/module-info.java
src/com.foo.baz/com/foo/baz/BazGenerator.java
```
那么我们可以一起编译它们：

```
$ javac --module-source-path src -d mods $(find src -name '*.java')
```
输出目录mods是一个模块路径目录，包含两个模块的分解，编译定义：

```
mods/com.foo.bar/module-info.class
mods/com.foo.bar/com/foo/bar/Main.class
mods/com.foo.baz/module-info.class
mods/com.foo.baz/com/foo/baz/BazGenerator.class
```
假设com.foo.bar.Main类包含应用程序的入口点，我们可以按原样运行这些模块：

```
$ java -mp mods -m com.foo.bar/com.foo.bar.Main
```
或者，我们可以将它们打包成模块化的JAR文件：

```
$ jar --create -f mlib/com.foo.bar-1.0.jar \
      --main-class com.foo.bar.Main --module-version 1.0 \
      -C mods/com.foo.bar .
$ jar --create -f mlib/com.foo.baz-1.0.jar \
      --module-version 1.0 -C mods/com.foo.baz .
```
mlib目录是一个包含两个模块的打包，编译定义的模块路径目录：

```
$ ls -l mlib
-rw-r--r-- 1501 Sep  6 12:23 com.foo.bar-1.0.jar
-rw-r--r-- 1376 Sep  6 12:23 com.foo.baz-1.0.jar
```
我们现在可以直接运行打包的模块：

```
$ java -mp mlib -m com.foo.bar
```

###### jtreg增强功能

jtreg测试工具支持一个新的声明性标签@modules。它需要一系列参数，每个参数可以是<module>或<module> / <package>的形式。在任一情况下，只有当命名模块存在时才会运行测试。后一种情况另外表明，测试需要访问该模块的命名包;如果包不被导出，则线束将安排将其导出到包含测试的模块。

可以将默认的一组@modules参数指定为TEST.ROOT文件或任何TEST.properties文件中的modules属性的值，该参数将用于不包含此类标记的目录层次结构中的所有测试。

现有的@compile标签接受一个新的选项，/ module = <module>。这具有使用上面定义的javac的--module选项来将指定的类编译为命名模块的成员。

##### 类装载机

Java SE Platform API历史上指定了两个类加载器：引导类加载器，它从引导类路径加载类，以及系统类加载器，它是新类加载器的默认委派父，通常是用于加载并启动应用程序。该规范并不要求这些类加载器中的任何一个的具体类型，也不是其准确的委托关系。

JDK自从1.2版本以来，实现了一个三级层次结构的类加载器，每个加载器都将委托给下一个：

应用程序类加载器（java.net.URLClassLoader的实例）从类路径加载类，并作为系统类加载器安装，除非通过系统属性java.system.class.loader指定备用系统加载程序。

扩展类加载器也是URLClassLoader的实例，通过扩展机制加载可用的类，以及JDK内置的一些资源和服务提供者。 （Java SE平台API规范中没有明确提及此加载器。）

引导类加载器仅在虚拟机中实现，并在ClassLoader API中由null表示，从引导类路径加载类。

为了保持兼容性，JDK 9保留了这个三级层次结构，同时进行以下更改来实现模块系统：

应用程序类加载器不再是URLClassLoader的实例，而是内部类的实例。它是既不是Java SE也不是JDK模块的命名模块的默认加载程序。

扩展类加载器不再是URLClassLoader的实例，而是内部类的实例。它不再通过JEP 220删除的扩展机制来加载类。但是，它确定了所选的Java SE和JDK模块，其中更多的是下面。在其新角色中，此加载程序被称为平台类加载器，它可通过新的ClassLoader :: getPlatformClassLoader方法获得，并且它将被Java SE Platform API规范所要求。

引导类加载器在库代码和虚拟机中均实现，但为了兼容性，它在ClassLoader API中仍然由null表示。它定义了核心Java SE和JDK模块。

平台类加载器不仅保留兼容性，而且还提高了安全性。引导类加载器加载的类型隐式授予所有安全权限（AllPermission），但其中许多类型实际上并不需要所有权限。我们有权限的模块，通过将它们定义到平台类加载器而不是引导类加载器，并通过授予他们在默认安全策略文件中实际需要的权限，不需要所有权限。定义到平台类加载器的Java SE和JDK模块是：

```
java.activation
java.annotations.common
java.compact1
java.compact2
java.compact3
java.compiler
java.corba
java.scripting
java.se
java.se.ee
java.sql
java.sql.rowset
java.transaction
java.xml.bind
java.xml.ws
jdk.accessibility
jdk.charsets
jdk.crypto.ec
jdk.crypto.pkcs11
jdk.dynalink
jdk.jsobject
jdk.localedata
jdk.naming.dns
jdk.scripting.nashorn
jdk.xml.dom
jdk.zipfs
```

所有其他Java SE和JDK模块都定义为引导类加载器，除了提供工具或导出工具API的JDK模块，这些API被定义为应用程序类加载器。 这些是：

```
jdk.attach
jdk.compiler
jdk.hotspot.agent
jdk.internal.le
jdk.internal.opt
jdk.jartool
jdk.javadoc
jdk.jconsole
jdk.jdeps
jdk.jdi
jdk.jlink
jdk.jshell
jdk.jstatd
jdk.jvmstat
```

三个内置的类加载器一起工作，加载类如下：

应用程序类加载器首先搜索定义到所有内置加载器的命名模块。如果将合适的模块定义到这些装载机之一，那么该加载程序将加载该类。如果在定义为其中一个加载器的命名模块中找不到类，则应用程序类加载器将委托给其父进程。如果一个类没有被父进程找到，那么应用程序类加载器就会搜索类路径。在类路径中找到的类作为该加载器未命名模块的成员加载。

平台类加载器搜索定义到所有内置加载器的命名模块。如果将合适的模块定义到这些装载机之一，那么该加载程序将加载该类。 （因此，平台类加载器现在可以委托给应用程序类加载器，当升级模块路径上的模块取决于应用程序模块路径上的模块时，该加载程序可能很有用。）如果在命名模块中找不到类定义为其中一个装载程序，然后应用程序类加载器将委托给其父进程。

引导类加载器搜索定义为自身的命名模块。如果在定义为引导加载程序的命名模块中找不到类，那么引导类加载器将通过-Xbootclasspath / a选项搜索添加到引导类路径的文件和目录。引导类路径中找到的类作为此加载程序未命名模块的成员加载。

应用程序和平台类加载程序委托给其各自的父加载程序，以确保在定义到其中一个内置加载程序的模块中找不到类时仍会搜索引导类路径。

##### 已删除：Bootstrap类路径选项

在早期版本中，-Xbootclasspath选项允许覆盖默认引导类路径，-Xbootclasspath / p选项允许将一系列文件和目录添加到默认路径。此路径的计算值通过JDK特定的系统属性sun.boot.class.path报告。

在模块系统到位时，引导类路径默认为空，因为引导类是从各自的模块加载的。 javac编译器仅支持旧模式下的-Xbootclasspath选项，java启动器不再支持这些选项，系统属性sun.boot.class.path已被删除。

编译器的-system选项可用于指定系统模块的备用源，如上所述，其“-release”选项可用于指定备用平台版本，如JEP 247（旧版平台编译）中所述。在运行时，上述的-patch-module选项可用于将内容注入初始模块图中的模块。

相关选项-Xbootclasspath / a允许将文件和目录附加到默认引导类路径。此选项以及java.lang.instrument包中的相关API有时被仪器代理使用，因此兼容性在运行时仍然受支持。其值（如果指定）通过JDK特定的系统属性jdk.boot.class.path.append报告。该选项可以传递给命令行启动器，java以及JNI调用API。

##### 开放设计问题

--add-modules选项是否允许指定添加的模块的预期类加载器？

在javac的传统模式中，module-info.java源文件被拒绝还是被忽略？

在javac的多模块模式中，-sourcepath和-classpath选项令人困惑。我们应该禁止他们吗？

jlink是否支持--module选项，还是以其他方式指定入口点及其与链接时生成的命令行启动器的关系？

jlink，jar和jmod工具使用GNU风格的选项，但是javac和java没有。这些都应该合理化。

必须最终确定JMOD文件的格式。

jtreg实现的现有的@library和@build标签应该扩展到与模块一起工作。

##### 测试

许多现有测试将受到引入模块系统的影响。在JDK 9中，上述的@modules标签已经被添加到超过3000个单元和回归测试中，许多使用-Xbootclasspath / p选项或者假设系统类加载器是URLClassLoader的测试已被更新。

当然，模块系统本身还有一套广泛的单元测试。在JDK 9源林中，大多数运行时测试都位于jdk仓库的test / jdk / jigsaw目录和热点仓库的runtime / modules目录下;大多数编译时测试都在langtools仓库的tools / javac / modules目录中。

包含此处描述的更改的早期访问构建已经可用了一段时间。我们鼓励更广泛的Java社区的成员测试这些构建的工具，库和应用程序，以帮助挑选出任何剩余的兼容性问题。

##### 风险与假设

此提案的主要风险是由于现有语言结构，API和工具的更改而导致的兼容性。

主要由于引入Java平台模块系统（JSR 376）引起的更改包括：

将公共修饰符应用于API元素不再保证元素将无处不在。辅助功能现在还取决于包含该元素的包是否由其定义模块导出，以及该模块是否可读取包含尝试访问它的代码的模块。例如，以下表单的代码可能无法正常工作：

```
Class<?> c = Class.forName(...);
if (Modifier.isPublic(c.getModifiers()) {
    // Assume that c is accessible
}
```
如果在命名模块和类路径中定义了一个包，则类路径上的包将被忽略。因此，类路径不能再用于覆盖环境中内置的类型。 javax.transaction包例如由java.transaction模块定义，因此不会在javax.transaction包中搜索类路径。这种限制对于避免跨类装载器和模块间拆分包很重要。在编译时和运行时，升级模块路径可用于将内置的环境升级到环境中。 --patch-module选项可用于其他特殊修补。

ClassLoader :: getResource *和Class :: getResource *方法不能再用于读取JDK内部资源。可以通过Module :: getResourceAsStream方法或者通过jrt：URL方案和JEP 220中定义的文件系统读取模块私有资源。

java.lang.reflect.AccessibleObject :: setAccessible方法不能用于访问未由其定义模块导出的包的成员;将抛出InaccessibleObjectException异常。如果框架库（例如序列化程序）需要访问这些成员，那么相关软件包必须通过模块描述符中的导出声明或--add-exports命令行选项导出到框架模块。

JVM TI代理不再能够在运行时环境启动的早期运行Java代码。特别是ClassFileLoadHook事件在原始阶段不再发送。仅在VM初始化之前才会发出启动阶段开始的VMStart事件，以便可以在除java.base之外的模块中加载类。两个新功能，can_generate_early_class_hook_events和can_generate_early_vmstart，可以由经过精心编写的代理程序添加到VM初始化早期处理事件。更多详细信息可以在类文件load hook事件和start事件的更新描述中找到。

定义Java EE API或Java EE应用程序感兴趣的API的模块在类路径上的代码默认情况下不会解决：

未命名模块的默认的根模块集合基于java.se模块而不是java.se.ee模块。因此，默认情况下，未命名模块中的代码将无法访问以下模块中的API：

```
java.activation
java.annotations.common
java.corba
java.transaction
java.xml.bind
java.xml.ws
```

这是一个有意的，如果痛苦的选择，由两个目标驱动：

避免与某些相同软件包中定义类型的流行库不必要的冲突。广泛使用的jsr305.jar例如定义了javax.annotation包中的注释类型，它也由java.annotations.common模块定义。

使现有应用服务器更容易迁移到JDK 9.应用服务器通常会覆盖这些模块中的一个或多个的内容，而在短期内，它们最有可能通过继续放置必需的非模块化JAR类路径上的文件。如果默认情况下解决了这些模块，则应用程序服务器的维护者必须采取尴尬的行动才能排除它们以覆盖它们。

这些模块仍然是JDK 9的一部分。类路径上的代码可以通过--add-modules java.se.ee选项授予对它们的访问权限。

一些Java SE API的运行时行为已经改变，尽管以继续遵守其现有规范的方式：

应用程序和平台类加载器不再是java.net.URLClassLoader类的实例，如上所述。调用ClassLoader :: getSystemClassLoader并盲目地将结果转换为URLClassLoader或与该类加载器的父级执行相同操作的现有代码可能无法正常工作。

一些Java SE类型已经被取消了特权，现在由平台类加载器加载，而不是引导类加载器，如上所述。直接委托给引导类加载器的现有自定义类加载器可能无法正常工作;它们应该被更新以委托给平台类加载器，这可以通过新的ClassLoader :: getPlatformClassLoader方法容易地获得。

如果系统属性java.security.policy用于覆盖系统的内置安全策略而不是扩充，则替换策略必须向平台类加载器加载的特权系统模块授予必要的权限。

有一个源不兼容的Java SE API更改：

在java.lang.instrument.ClassFileTransformer接口中声明的五参数变换方法现在是一个默认方法。该接口现在还声明一种新的变换方法，使得相关的java.lang.reflect.Module对象在加载时对类进行测试时可用于变压器。现有的编译代码将继续运行，但现有的使用现有五参数变换方法作为功能界面的源代码将不再编译。
最后，由于对JDK特定的API和工具的修订而导致的更改包括：

默认情况下，大多数JDK的内部API都无法访问，如JEP 260中所详述的。依赖于这些API的现有代码可能无法正常工作。解决方法是通过上面定义的--add-exports选项来打破封装。已经将sun.misc和sun.reflect包中的所选关键内部API移动到jdk.unsupported模块，如JEP 260中所述。这些软件包中的非关键内部API，如sun.misc.BASE64 {De，En }编码器，已被删除。

-Xbootclasspath和-Xbootclasspath / p选项已被删除，如上所述。在编译时，新的-release选项可用于指定备用平台版本（参见JEP 247）。在运行时，上述新的-patch-module选项可用于将内容注入到系统模块中。

JDK特定的系统属性sun.boot.class.path已被删除，因为引导类路径默认为空。使用此属性的现有代码可能无法正常工作。

由JEP 179引入的JDK特定注解@ jdk.Exported将被删除，因为它传达的信息现在记录在模块描述符的导出声明中。我们没有看到JDK以外的工具使用该注释的证据。

以前在rt.jar和其他内部工件中找到的META-INF / services资源文件不存在于相应的系统模块中，因为现在在模块描述符中声明了服务提供程序和依赖关系。扫描此类文件的现有代码可能无法正常工作。

JDK特定的系统属性file.encoding可以像以前一样通过-D选项在命令行上设置，但只能在基本模块中定义的字符集时工作。指定其他字符集的现有启动脚本可能无法正常工作。

与引入模块化图像一样，不可能在抽象中确定这些变化的全部影响。 因此，我们必须依靠广泛的内部和外部测试。 如果其中的一些变化被证明是对开发人员，部署者或最终用户来说是不可逾越的障碍，那么我们将会调查如何减轻其影响。

##### 依赖性

JEP 200（模块化JDK）作为临时措施最初定义了JDK中存在于XML文档中的模块。 此JEP将这些定义转移到适当的模块描述符，即module-info.java和module-info.class文件，并删除根源代码库中的modules.xml文件。

JDK 9中JEP 220（模块化运行时映像）的初始实现使用自定义构建时工具来构建JRE和JDK映像。 该JEP用jlink工具替换了该工具。

根据JEP 238，模块化JAR文件也可以是多版本JAR文件。

#### JEP 262: TIFF Image I/O
##### 概要

扩展标准的Image I / O插件，以支持TIFF图像格式。

##### 动机

作为Java SE一部分的Image I / O框架（javax.imageio）提供了插入图像编解码器的标准方式。某些格式的编解码器，例如PNG和JPEG，必须由所有实现提供。这个集合中缺少广泛使用的格式TIFF。多年来，对于这种格式，来自代表小型和大型ISV的开发者都有多个请求。现在，由于OS X使用TIFF作为标准平台图像格式，因此我们至今仍无法支持这一点。

##### 描述

以前在Java Advanced Imaging API工具项目（javadoc）中开发的完全用Java编写的合适的TIFF读写器插件。我们将把它合并到JDK中，以及现有的Image I / O插件。该包将被重命名为javax.imageio.plugins.tiff，因为它将成为Java SE规范的标准部分。 XML元数据格式名称将被重新命名。

##### 测试

除了包名称更改之外的代码将按原样并入;它已经进行了多年的现场测试。我们将为暴露的API创建jtreg格式测试

#### JEP 263: HiDPI Graphics on Windows and Linux
##### 概要

在Windows和Linux上实现HiDPI图形。

##### 动机

开发人员和用户对于在HiDPI显示器上运行的应用程序有一些基本的期望：

Windows和GUI组件应根据平台建议具有适当的大小，

尽管HiDPI设置中指定的任何默认缩放比例，文本仍应保持清晰

图标和图像应该是平滑的，并且优选地具有适合于显示器的像素密度的细节。

不幸的是，Java应用程序仍然基于Windows和Linux上的像素进行大小和渲染，即使在可以具有传统显示器的2到3倍的像素密度的HiDPI显示器上。这导致GUI组件和窗口是2到3倍太小，因此太小，无法读取或有效地进行交互。

JDK已经在Mac OS X上支持HiDPI“视网膜显示”，并以适当的尺寸显示清晰的文字和图像，以显示密度。所有平台上支持HiDPI显示器的基础工作已经由Mac“视网膜”支持提供，但应用程序在各种平台上处理HiDPI的方式差异意味着额外的工作仍然是推广所做的工作对于Mac

现在应该为Windows和Linux平台提供相同的自动缩放和大小调整。

##### 描述

##### HiDPI在Windows上

Windows控制面板为用户提供了多种方式，可以在桌面上请求扩展几个版本的组件和窗口（自Windows XP以来，以各种形式）。一些操作系统版本还提供了自动缩放未声明为“DPI感知”的应用程序的设施。 “DPI不知道”应用程序的默认缩放可能会导致许多视觉工件，包括模糊的窗口和不准确的布局和剪切的文本，因此Java运行时已经声明自己是“DPI感知”一段时间，以避免模糊和布局问题。根据DPI扩展内容的支持并不是一件微不足道的工作，而且由于大多数显示密度仅需要25％的缩放比例，因此决定忽略推荐的缩放指南和策略，并继续渲染AWT和Swing组件以1：1像素缩放，让JDK应用程序略小于规范，但具有清晰的文本和图形。

最近，尽管如此，显示器的像素密度大幅度上升，并且容易找到具有屏幕的笔记本电脑，这些屏幕是做出这些决定时常见的显示器密度的2至3倍。因此，这些较新的Windows机器上的Java应用程序有时可能太小，因为无法使用 - 这不再是“稍微小于模糊的窗口”的情况，因此我们需要适当地扩展我们的应用程序窗口。

Windows API提供了编写DPI感知应用程序的指标，机制和指南，以使UI在各种DPI显示设置中看起来保持一致，并响应用户偏好，使用户界面足够大，使用户能够轻松与UI组件进行交互。

Windows Direct2D图形API自动表示系统DPI，表示DIP中的坐标（独立于设备的像素），使位图具有DPI，并通过考虑DPI正确缩放它们。但是，AWT和Swing库不是基于Direct2D图形API。

Windows 7及更高版本提供了许多方法来获取桌面的水平和垂直DPI，包括GetDesktopDpi方法，最近在Windows 8.1中，GetDpiForMonitor方法和WM_DPICHANGED消息。这些值可用于缩放AWT / Swing库中HiDPI支持的窗口大小，鼠标坐标和字体。在Java2D渲染层中已经存在对缩放的必要支持，但是我们需要向Swing和AWT通告这些值，并更新其组件以识别和符合比例因子。该支持包括确保将所需的渲染比例因子提供给Java2D Graphics对象，中间渲染图像以适当的比例因子呈现给目标，并且任何关于渲染调用中坐标如何映射到像素的假设需要为复查。

##### HiDPI在Linux上

在GTK + 3库中已经实现了HiDPI支持。当使用HiDPI显示器时，GTK库会自动缩放客户端的应用程序。

Java2D和AWT使用不支持HiDPI的XLib库。因此，目前的Java应用程序可能在Linux上的HiDPI显示屏上看起来可能要小2到3倍。

此外，使用GTK外观风格的任何Java应用程序可能会自动受益于GTK库中HiDPI的支持，而窗口和其他组件的大小可能与已扩大2-3倍的GTK组件匹配以满足HiDPI要求。这可能会导致诸如JDK-8058742中描述的问题。

##### 备择方案

开发人员可以在HiDPI显示器上运行时手动缩放应用程序中的所有GUI元素，但这需要对应用程序进行大量重构工作，并且不会将Swing和AWT组件写入，以便与外部扩展进行协调。

##### 测试

应在HiDPI显示屏上进行以下测试。各种屏幕比例将是优选的 - 192 DPI和144 DPI将是一个很好的最小支持矩阵。请注意，使用Windows控制面板可以将非常高的DPI显示设置为较低的人造DPI，因此不一定需要其他系统，但必须在测试运行之间进行重新配置。此外，在Windows上更改DPI通常需要重新启动以将新的DPI通告给应用程序。

在Windows中，可以在控制面板中设置DPI设置。控制面板的不同变体和允许的设置存在于Windows 7，Windows 8.1和Windows的旧版本上，但都倾向于导致将相同类型的信息传递给应用程序（X＆Y指定的DPI） ，所以目前在控制面板的变化下不需要额外的测试，这只是一个影响测试环境如何配置的细节。

在Linux上，可以使用GTK + 3的GDK_SCALE环境变量或GNOME 3的缩放因子设置对DPI设置进行仿真。

应检查以下项目：

文字不应该模糊
组件（标签和按钮）中的文本不应该被剪切
UI组件布局在AWT / Swing上应该保持敏感（即没有重叠或奇数差距）
当应用程序或测试用例提供时，应在AWT / Swing对话框和组件上使用高分辨率图标
应用程序应正确处理鼠标事件
应用程序应具有与运行良好的DPI感知的本机应用程序相似的大小

#### JEP 264: Platform Logging API and Service
##### 概要

定义一个最小的日志记录API，哪些平台类可以用来记录消息，以及这些消息的消费者的服务接口。库或应用程序可以提供此服务的实现，以便将平台日志消息路由到其选择的日志记录框架。如果没有提供实现，那么将使用基于java.util.logging API的默认实现。

##### 目标

在java.base模块中定义和使用，因此它不能依赖于java.util.logging API。

可以轻松应用使用外部日志框架的应用程序，如SLF4J或Log4J。

减少对java.logging模块的依赖，以简化JDK模块图。

引导引导问题，使平台类可以在日志消息初始化之前记录消息。

默认情况下，当存在java.logging模块时，通过java.util.logging API记录消息。

##### 非目标

定义通用界面进行日志记录并不是一个目标。服务接口只包含JDK自己使用所需的最小方法集。

通过API支持程序化日志记录配置（设置级别，目标文件等）并不是一个目标。

将JDK中的所有类转换为通过此新API进行日志记录并不是一个目标。

##### 动机

与java.util.logging API相比，大多数现代日志记录框架（例如Log4J 2.0，Logback）被分为立面和实现。通过这样的外部框架登录的应用程序应该创建日志记录器，并通过该框架提供或支持的立面进行日志记录。

所提出的服务使应用程序能够将JDK配置为与应用程序使用相同的日志记录框架：它只需要提供返回平台日志记录器的实现，该平台日志记录器将包含首选日志记录框架的记录器。

应用程序将继续使用它正在使用的日志框架提供的外观。 LoggerFinder服务使得可以将JDK配置为使用相同的框架。

##### 描述

使用java.util.ServiceLoader API，使用系统类加载器定位并加载系统范围的LoggerFinder实现。如果没有找到具体的实现，则使用LoggerFinder服务的JDK内部默认实现。当存在java.logging模块时，该服务的默认实现使用java.util.logging作为后端，因此默认情况下，将日志消息路由到java.util.logging.Logger。但是，LoggerFinder服务使应用程序/框架可以插入自己的外部日志记录后端，而不需要同时配置java.util.logging和后端。

LoggerFinder服务的实现应该能够区分系统日志记录器（系统类从Bootstrap类装载器（BCL）使用）和应用程序记录器（由应用程序为其自己的用途创建）。这种区别对于平台安全性很重要。记录器的创建者可以将记录器创建的类或模块传递给LoggerFinder，以便LoggerFinder可以确定要返回哪种记录器。

JDK中的类通过调用System类的工厂方法获取从LoggerFinder创建的记录器：

```
package java.lang;

...

public class System {

    System.Logger getLogger(String name) { ... }

    System.Logger getLogger(String name, ResourceBundle bundle) { ... }

}
```

将修改JDK内部的sun.util.logging.PlatformLogger API，以通过这些方法返回的系统记录器发出日志消息。

##### 测试

将添加新的测试以测试LoggerFinder服务和映射到默认的java.util.logging后端。 我们还需要验证插入现有的外部框架是否正常工作，特别是在潜在的引导问题方面。

##### 风险与假设

当java.util.logging是后端时，已转换为使用系统日志记录器的JDK类仍然可以像以前一样通过java.util.logging进行配置，如果可能的话，不应该更改日志消息和级别。

一些Java SE API目前在其公共签名中公开java.util.logging类型。 这可能是通过系统记录器将这些API转换为日志的障碍。

#### JEP 265: Marlin Graphics Renderer
##### 概要

更新Java 2D以使用Marlin渲染器作为默认图形光栅化器。

##### 成功指标

Marlin Renderer必须始终如一地执行当前的图形光栅化器以及相同或更好的质量和精度的双鱼座。在大多数基准上，它必须显着优于双鱼座。它应该表现出优于双鱼座的多线程可扩展性。开发版本已经达到了这些目标。额外的目标是在单线程性能上等于或优于封闭源Ductus光栅化器，并在质量和准确度上匹配它。 Marlin已经显示出比Ductus更好的MT可扩展性。

##### 动机

JDK目前​​使用一种称为Pisces的图形光栅化器。它用于抗锯齿渲染（字体除外），因此其性能对于许多图形密集型应用程序至关重要，但其性能比由Oracle发布的封闭源Ductus光栅化器的性能差得多。因此，Marlin已经被一些应用程序用作双鱼座的替代品，将其放置在引导类路径上。因此，高性能的光栅化器对于采用是很重要的。

##### 描述

图形光栅化器是Java 2D使用的实现库。它实现了与AWT和Java 2D子系统进行通信的内部接口，但对Java开发人员没有任何外部可见的API。 Marlin是所有的Java代码（到目前为止），是双鱼座光栅化器的一个叉子。虽然它分开托管了很长时间，但它现在正在通过继续开发OpenJDK图形光栅化器项目，该项目最初专注于双鱼座。 Marlin已经很大程度上完成了测试，并通过了Oracle的内部测试。它将逐步进一步开发，然后集成到JDK中作为双鱼座的替代品，并且在Oracle的选择上也许是Ductus。

##### 测试

这不需要任何新的测试开发，因为没有API。然而，我们预计为现有J2DBench或其他测试套件开发额外的测试场景，以便在我们识别差距的情况下检查MT性能。测试执行将是现有的功能和合规性测试以及现有的性能测试，例如J2DBench。目前版本的Marlin已经通过了JDK 9中的所有功能，回归和兼容性（JCK）测试，由Oracle SQE组。

#### JEP 266: More Concurrency Updates
##### 概要

可互操作的发布 - 订阅框架，对CompletableFuture API的增强功能以​​及各种其他改进。

##### 动机

在应用程序中并发和并行使用的不断演变需要图书馆支持的不断发展。

##### 描述

接口支持Reactive Streams发布 - 订阅框架，嵌套在新类Flow中。出版商生产由一个或多个订阅者消费的物品，每个订阅者都由订阅管理。通信依赖于一种简单的流量控制形式（方法Subscription.request，用于传达反向压力），可用于避免在“推”式系统中可能发生的资源管理问题。提供实用程序类SubmissionPublisher，开发人员可以使用它创建自定义组件。
这些（非常小的）接口对应于广泛参与（从“活动流”计划）定义的界面，并支持在JVM上运行的许多异步系统之间的互操作性。将界面嵌套在一个类中是一个保守的政策，允许他们在各种短期和长期的可能性中使用。没有计划为分布式消息传递提供基于网络或基于I / O的java.util.concurrent组件，但是未来的JDK版本可能会在其他软件包中包含这些API。

对CompletableFuture API的增强

添加了基于时间的增强功能，使未来能够在一定时间内完成一个值，或者在某个持续时间后完成此操作，请参阅方法或者Timeout和completeTimeout。另外，由一个名为delayedExecutor的静态方法返回的补全执行程序允许任务在一定时间后执行。这可以与在CompletableFuture上的执行者接收方法相结合，以支持具有时间延迟的操作。
添加子类增强功能可以使其更容易从CompletionFuture扩展，例如提供支持备用默认执行程序的子类。
自JDK 8以来积累的大量实施改进;其中许多都很小，但有些则包括Javadoc规范改写。

##### 测试

持续的JSR 166 EG成员为所有组件提供功能，TCK和性能测试。

##### 风险与假设

将注意确保保留JDK 9存储库和166存储库之间的任何小但必需的代码差异。

#### JEP 267: Unicode 8.0
##### 概要

升级现有的平台API以支持Unicode标准版本8.0。

##### 目标

支持最新版本的Unicode，并修改以下类：

200新新新新旗新新旗新新旗新新旗新新旗新新旗新新200新新旗新新旗新新旗新新旗新
NumericShaper在java.awt.font包中，和
Bidi，BreakIterator和Normalizer在java.text包中。
##### 非目标

这个JEP将不会实现两个相关的Unicode规范：

UTS＃10，Unicode排序算法，和
UTS＃46，Unicode IDNA兼容性处理。
##### 动机

Unicode是一个不断发展的行业标准，所以我们必须将Java保持最新版本。

##### 描述

这是JEP 227的后续操作，它在JDK 9中引入了Unicode 7.0。Unicode 8.0添加了大约8,000个字符，10个块和6个脚本。

##### 测试

我们需要验证相关类是否正确使用了最新的Unicode数据。

##### 风险与假设

Unicode 8于2015年6月发布。在JDK 9开发中相当晚的时候，始终实现最新的Unicode标准很重要。推迟到JDK 10将使我们超过三年。

在JDK 9发布之前，可能会发布Unicode标准（例如8.0.X）的一个小小的更新，在这种情况下，我们可能需要考虑纳入该版本。

##### 依赖性

此功能取决于Unicode Consortium的Unicode标准。

#### JEP 268: XML Catalogs
##### 概要

开发支持OASIS XML Catalogs标准v1.1的标准XML Catalog API。 API将定义可以与接受解析器的JAXP处理器一起使用的目录和目录解析器抽象。

##### 非目标

新API不打算复制现有的内部目录实现。换句话说，没有意图提供兼容的API或维护源或二进制兼容性。使用内部API的现有库或应用程序将需要迁移到新的API，以便利用新功能。

##### 动机

解析XML / XSD / XSL中的外部引用时，XML目录非常有用，无需重复检索外部资源。在某些情况下，需要XML目录以确保应用程序在导入的XML资源的源与原始源不同的本地环境中正常工作。

XML目录还可以通过将远程外部引用引导到本地目录，然后禁止检索外部资源来提高应用程序的安全性。

自JDK 6以来，内部目录解析器已被包含在JDK中。由于缺少公共API，它已被外部库/应用程序直接使用或引用。它也被捆绑并交付在Maven存储库中，并被诸如JAX-WS / JAXB之类的应用程序使用（参见例如com.sun.xml.ws.util.xml.XmlUtil）。标准API是可取的，因此实际可以支持该功能。

##### 描述

XML Catalog API将根据OASIS XML目录标准v1.1定义以下接口。它将为Catalog标准的核心功能提供直接支持，实现EntityResolver和URIResolver接口：

CatalogManager将管理XML Catalogs和CatalogResolver的创建，以及功能和属性。

目录将实现OASIS Open Catalog文件的语义。它将定义一个实体目录，将外部标识符和URI引用映射到（其他）URI引用，并将其委派给其他目录。

一个CatalogResolver将实现JAXP现有的EntityResolver和URIResolver接口。解析器将支持OASIS标准处理指令作为SAX XMLFilter。

该JEP还建议在公开API发布后删除内部目录解析器实现。

新的API将符合当前的规范版本1.1，它与OASIS技术分辨率9401：1997（TR 9401的修正2）兼容，这是内部目录解析器所基于的。

#### JEP 269: Convenience Factory Methods for Collections
##### 概要

定义库API，方便地创建具有少量元素的集合和映射的实例，以减轻Java编程语言中没有集合文字的痛苦。

##### 目标

在集合接口上提供静态工厂方法，创建紧凑的，不可修改的集合实例。 API被故意保持最小。

##### 非目标

提供一个完全通用的“集合构建器”功能并不是一个目的，例如，让用户控制集合实现或各种特性，如可变性，预期大小，加载因素，并发级别等。

支持具有任意数量元素的高性能，可扩展的集合并不是一个目标。重点是小集合。

提供不可修改的收集类型不是目标。也就是说，即使所提出的实现实际上是不可修改的，该提议并没有揭示类型系统中的不可修改性的特征。

提供“不变永久”或“功能”集合不是一个目标。

##### 动机

Java经常被批评为冗长。创建一个小的，不可修改的集合（例如一个集合）涉及构造它，将其存储在局部变量中，并在其上调用add（）几次，然后包装它。例如，

```
Set<String> set = new HashSet<>();
set.add("a");
set.add("b");
set.add("c");
set = Collections.unmodifiableSet(set);
```
这是相当冗长的，并且因为它不能在单个表达式中表达，静态集合必须在静态初始化程序块中填充，而不是通过更方便的字段初始化程序。或者，可以使用另一个集合的副本构造函数填充集合：

```
Set<String> set = Collections.unmodifiableSet(new HashSet<>(Arrays.asList("a", "b", "c")));
```
这仍然有点冗长，也不太明显，因为在创建集合之前必须创建一个列表。另一种选择是使用所谓的“双支架”技术：

```
Set<String> set = Collections.unmodifiableSet(new HashSet<String>() {{
    add("a"); add("b"); add("c");
}});
```
这将使用一个匿名内部类的实例初始化器构造，这是一个比较漂亮的。然而，这是非常模糊的，每次使用都需要额外的课程。它还包含对包围实例和任何捕获对象的隐藏引用。这可能会导致内存泄漏或串行化问题。由于这些原因，最好避免这种技术。

通过组合流工厂方法和收集器，Java 8 Stream API可用于构建小集合。例如，

set <String> set = Collections.unmodifiableSet（Stream.of（“a”，“b”，“c”）。collect（toSet（）））;
（流收集器不保证他们返回的集合的可变性。在Java 8中，返回的集合是普通的可变集合，例如ArrayList，HashSet和HashMap，但在将来的JDK版本中可能会发生变化）。

这是有点迂回，虽然不是晦涩，但也不是很明显。它还涉及一定量的不必要的对象创建和计算。通常，Map是异常值。流不能以这种方式用于构建Map，除非可以从键计算值，或者流元素包含键和值两者。

在过去，已经有一些建议用来修改Java编程语言来支持收集文字。然而，通常情况下，语言功能的情况下，没有任何功能像人们可能想象的一样简单或干净，因此收集文字不会出现在下一个Java版本中。

通过提供用于创建小集合实例的库API可以获得收集文字的许多好处，与更改语言相比，成本和风险显着降低。例如，创建一个小的Set实例的代码可能如下所示：

```
Set<String> set = Set.of("a", "b", "c");
```
Collections类中有现有的工厂来支持创建空的列表，集合和地图。还有工厂生产具有一个元素或键值对的单例列表，集合和地图。 EnumSet包含几个重载的（...）方法，它们使用固定或可变数量的参数，以方便地创建具有指定元素的EnumSet。但是，没有很好的通用方式来创建包含任意类型对象的列表，集合和地图。

Collections类中有一些用于创建不可修改的列表，集合和地图的组合方法。这些不会产生固有的不可修改的集合。相反，他们会收集另一个集合，并将其包装在拒绝修改请求的类中，从而创建原始集合的不可修改的视图。拥有对基础集合的引用仍然允许修改。每个包装器是一个额外的对象，需要另一个间接级别，并且比原始集合消耗更多的内存。最后，包装收藏仍然承担支持突变的代价，即使它从未被修改。

##### 描述

在“列表”，“设置”和“映射”界面上提供静态工厂方法，以创建这些集合的不可修改实例。 （请注意，与类中的静态方法不同，接口上的静态方法不会被继承，因此无法通过实现类调用它们，也不可能通过接口类型的实例进行调用）。

对于列表和集合，这些工厂方法将工作如下：

```
List.of(a, b, c);
Set.of(d, e, f, g);
```
这些将包括varargs重载，这样对集合大小没有固定的限制。但是，如此创建的集合实例可能会调整为较小的大小。将提供多达十个元素的特殊情况API（固定参数重载）。虽然这在API中引入了一些混乱，但它避免了由varargs调用引起的数组分配，初始化和垃圾回收开销。重要的是，无论是否调用fixed-arg或varargs重载，调用站点的源代码都是相同的。

对于Maps，将提供一组固定参数方法：

```
Map.of()
Map.of(k1, v1)
Map.of(k1, v1, k2, v2)
Map.of(k1, v1, k2, v2, k3, v3)
...
```
我们预计支持多达十个关键值对的小地图将足以覆盖大多数用例。 对于较大数量的条目，将提供一个API，以创建给定任意数量的键值对的Map实例：

```
Map.ofEntries(Map.Entry<K,V>...)
```

虽然这种方法类似于List和Set的等效的varargs API，但不幸的是要求每个键值对都被包装。 一种适用于静态导入的键和值的方法将使这更方便：

```
Map.Entry<K,V> entry(K k, V v)
```
使用这些方法，可以创建具有任意数量条目的地图：

```
Map.ofEntries(
    entry(k1, v1),
    entry(k2, v2),
    entry(k3, v3),
    // ...
    entry(kn, vn));
```

（可能通过在JDK的未来版本中使用价值类型来减轻拳击费用。entry（）便利方法实际上将返回一个新引入的实现Map.Entry的具体类型，以便于潜在的未来迁移到值类型。）

提供用于创建小型，不可修改的集合的API满足大量用例，并且有助于保持规范和实现简单。不可修改的集合避免了制作防御性副本的需要，而且更适合并行处理。

小集合占用的运行时空间也是一个很强的考虑。使用包装API，使用两个元素的一个不可修改的HashSet的简单创建将由六个对象组成：包装器，包含HashMap的HashSet，其中的桶（数组）表和每个元素的一个Node实例。与存储的数据量相比，这带来了巨大的开销，对数据的访问不可避免地需要多个方法调用和指针取消引用。为小型固定大小的集合设计的实现可以避免大部分这种开销，使用紧凑的基于现场或基于阵列的布局。不需要支持突变（并且在创建时知道收集大小）也有助于节省空间。

这些工厂返回的具体类不会被公开为公共API。不会保证返回的集合的运行时类型或身份。这将允许实现随时间而改变，而不会破坏兼容性。调用者唯一应该依赖的是返回的引用是其接口类型的实现。

生成的对象将是可序列化的。序列化代理对象将被用作实现类的通用序列化表单。这将阻止有关具体实现的信息泄漏到序列化形式，从而保留未来维护的灵活性，并允许具体实现从版本更改到发布，而不会影响序列化兼容性。

零元素，键和值将被禁止。 （最近没有引入的集合已经支持null。）此外，禁止空值为更紧凑的内部表示，更快的访问和更少的特殊情况提供了机会。

List实现预期通过索引提供快速元素访问，因此它们将实现RandomAccess标记接口。

存储在这些集合中的元素必须支持典型的集合合同，包括对hashCode（）和equals（）的适当支持。如果Set的元素或Map的一个元素以影响其hashCode（）或equals（）方法的方式进行突变，则集合的行为可能会变为未指定。

一旦构建并安全地发布，这些集合实例将对多个线程的并发访问是安全的。

将搜索JDK，以便可以使用这些新API的潜在网站。这些网站将被更新为使用新的API作为时间和时间表许可。

##### 备择方案

语言变化已被考虑过多次，被拒绝：

项目硬币计划，2009年3月29日
项目硬币计划，2009年3月30日
关于lambda-dev的JEP 186讨论会，2014年1月 - 3月
这些语言建议被放在了这个消息中总结的基于图书馆的提案。

Google Guava图书馆拥有丰富的实用工具，用于创建不可变的集合，包括构建器模式，以及创建各种可变集合。 Guava图书馆是非常有用和普遍的，但可能会被包含在Java SE平台中。该提案类似于Stephen Colebourne lambda-dev，2014年2月19日的提案，其中包含了番石榴不可变收集工厂方法的一些想法。

用于初始化任意数量条目的Map的Map.fromEntries（）方法并不理想，但它似乎是替代方案中最不好的。它的优点是它是类型安全的，它在语法中具有相邻的键和值，在编译时已经知道条目的数量，并且它适合用作字段初始化器。然而，它涉及拳击，而且是相当冗长的。考虑了几种替代方案，它们都引入了似乎使它们比目前提案更差的权衡。

具体收集类的静态工厂方法（例如ArrayList，HashSet）已从此提案中删除。他们似乎是有用的，但在实践中，他们往往会分散开发人员使用不可变集合的工厂方法。有一小组使用情况用于使用预定义的值初始化可变集合实例。通常优选的是将这些预定义值置于不可变集合中，然后通过复制构造函数初始化可变集合。

还有一个皱纹，就是类的静态方法是由子类继承的。假设要添加一个静态工厂方法HashMap.of（）。由于LinkedHashMap是HashMap的子类，因此应用程序代码可能调用LinkedHashMap.of（）。这将最终调用HashMap.of（），而不是所有的期望！减轻这种情况的一种方法是确保所有具体的收集实现具有相同的工厂方法集，以便不会发生继承。对于具体集合的用户定义的子类，继承仍然是一个问题。

##### 测试

在JDK回归测试套件中将会有通常的单元测试集，JCK会测试公共API。序列化表格也可能由JCK涵盖。

将开发一套尺寸和性能测试。与基准测量比较的典型目标相反，这些测试将将新的集合实现与现有测试进行比较。期望的是，新的集合将消耗更少的堆空间，无论是在固定开销和每个元素的基础上。在某些情况下，新集合可能会较慢，因为与现有集合相比，内部表现形式不同。任何这种减速应该是合理的。虽然没有具体的表现目标，但是10倍慢是不能接受的。此外，随着元素数量的增加，新集合应保持一致的性能。最后，测量将建立基准性能数据，以便今后的变化应与之进行比较。

#### JEP 270: Reserved Stack Areas for Critical Sections
##### 概要

在线程堆栈上保留额外的空间以供关键部分使用，这样即使堆栈溢出也可以完成。

##### 目标

提供一种机制来减轻由关键数据导致的死锁风险，例如由关键部分中抛出的StackOverflowError引起的java.util.concurrent锁（例如ReentrantLock）。

该解决方案必须大部分基于JVM，以免对java.util.concurrent算法或已发布的接口或现有库和应用程序代码进行修改。

该解决方案不能限于ReentrantLock案例，并且应适用于特权代码中的任何关键部分。

##### 非目标

该解决方案的目的不是为了提供对非特权代码的堆栈溢出的鲁棒性。

该解决方案不旨在避免StackOverflowErrors，而是减轻在关键部分中抛出此类错误从而破坏某些数据结构的风险。

提出的解决方案是在解决一些众所周知的腐败案例之间进行权衡，同时保持性能，合理的资源成本和相对较低的复杂性。

##### 动机

StackOverflowError是一个异步异常，只要线程中的计算需要比允许的更大的堆栈（JVM规范§2.5.2和§2.5.6），Java虚拟机就可以抛出异常异常。 Java语言规范允许通过方法调用（JLS§11.1.3）同步抛出一个StackOverflowError。 HotSpot VM使用此属性在方法条目上实现“堆栈轰击”机制。

堆栈冲突机制是一种干净的方式来报告堆栈溢出发生，同时保持JVM的完整性，但它不能为应用程序从这种情况中恢复提供安全的方式。在一系列修改的中间可能会发生堆栈溢出，如果不完整，可能会使数据结构处于不一致的状态。

例如，当在java.util.concurrent.locks.ReentrantLock类的关键部分中抛出一个StackOverflowError时，锁定状态可能会处于不一致状态，从而导致潜在的死锁。 ReentrantLock类使用AbstractSynchronizerQueue的实例来实现其关键部分。其lock（）方法的实现是：

```
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```
该方法尝试用原子操作来更改状态字。如果修改成功，则通过调用setter方法设置所有者，否则调用慢速路径。问题是如果在状态字被改变之后抛出了一个StackOverflowError，并且在所有者被有效地设置之前，锁就变得不可用了：它的状态字表示它被锁定，但没有所有者被设置，所以没有线程可以解锁它。因为堆栈大小检查是在方法调用时间（至少在HotSpot）中执行的，所以当调用Thread.currentThread（）或调用setExclusiveOwnerThread（）时，可以抛出一个StackOverflowError。在任一情况下，都会导致ReentrantLock实例的损坏，并且尝试获取此锁的所有线程将永久被阻止。

这个特殊的问题在JDK 7中引起了一些严重的问题，因为并行类加载是使用ConcurrentHashMap实现的，而当时的ConcurrentHashMap代码使用了ReentrantLock实例。如果由于StackOverflowError，ReentrantLock实例被破坏，则类加载机制本身可能会死锁。 （这发生在压力测试（JDK-7011862）中，但也可能在现场发生。）

ConcurrentHashMap类的实现在2013年6月完全更改。新实现使用synchronized语句而不是ReentrantLock实例，因此JDK 8和更高版本的ReentrantLocks由于损坏而不会受到类加载死锁。然而，使用ReentrantLock的任何代码仍然可能会受到影响并导致死锁。 concurrency-interest@cs.oswego.edu邮件列表中已经报告了这些问题。

问题不仅限于ReentrantLock类。

Java应用程序或库通常依靠数据结构的一致性正常工作。这些数据结构的任何修改都是关键的部分：在关键部分的执行之前，数据结构是一致的，并且在执行之后数据结构也是一致的。然而，在执行期间，数据结构可能会经历暂时不一致的状态。

如果关键部分由不包含其他方法调用的单个Java方法组成，则当前的堆栈溢出机制运行良好：可用的堆栈是足够的，并且方法执行没有麻烦，或者这是不够的，因此在之前抛出一个StackOverflowError执行方法的第一个字节码。

当关键部分由几种方法组成时，会发生此问题，例如调用方法B的方法A.可用的堆栈足以使方法A开始执行。方法A开始修改数据结构，然后调用方法B，但剩余的堆栈不足以执行B，导致抛出一个StackOverflowError。由于方法B和方法A的其余部分尚未执行，所以数据结构的一致性可能已被破坏。

##### 描述

所提出的解决方案的主要思想是在执行堆栈上为关键部分保留一些空间，以使其能够完成其执行，其中常规代码将被堆栈溢出中断。假设关键部分相对较小，并且在执行堆栈上不需要巨大的空间来成功完成。目标是不要拯救遇到其堆栈限制的错误线程，而是保留如果在关键部分中抛出StackOverflowError可能会损坏的共享数据结构。

主要机制将在JVM中实现。 Java源代码中所需的唯一修改是必须用于标识关键部分的注释。此注释（目前命名为jdk.internal.vm.annotation.ReservedStackAccess）是可由任何类型的特权代码使用的运行时方法注释（请参阅下文关于此注释的可访问性的段落）。

为了防止共享数据结构的破坏，JVM将尝试延迟抛出一个StackOverflowError，直到有问题的线程退出了所有的关键部分。每个Java线程都有一个在其执行堆栈中定义的新区域，称为保留区域。仅当Java线程在其当前调用堆栈中具有使用jdk.internal.vm.annotation.ReservedStackAccess注释的方法时，才能使用此区域。当JVM检测到堆栈溢出状况，并且线程在其调用堆栈中具有注释方法时，JVM授予临时访问保留区域，直到调用堆栈中不再有注释方法。当对保留区域的访问被撤销时，抛出一个延迟的StackOverflowError。如果在检测到堆栈溢出条件时，线程在其调用堆栈中没有注释方法，则立即抛出StackOverflow（这是当前的JVM行为）。

请注意，保留堆栈空间可以通过注释方法使用，也可以通过直接或传递方式从其调用的方法。自然地支持注释方法的嵌套，但每个线程都有一个共享的保留区;也就是说，调用注释方法不会添加新的保留区域。保留区域的大小必须根据所有注释关键部分的最坏情况进行。

默认情况下，jdk.internal.vm.annotation.ReservedStackAccess注释仅适用于特权代码（由引导程序或扩展类加载器加载的代码）。可以使用此注释对特权代码和非特权代码进行注释，但默认情况下，JVM将忽略非特权代码。此默认策略背后的理由是关键部分的保留堆栈空间是所有关键部分之间的共享资源。如果任意代码能够使用这个空间，那么它不再是保留的空间，而是会使整个解决方案失败。即使在产品构建中也可以使用JVM标志来放松此策略，并允许任何代码能够从此功能中受益。

##### 履行

在HotSpot VM中，每个Java线程在其执行堆栈的末尾都有两个区域：黄色区域和红色区域。两个内存区域均受到所有访问的保护。

如果在其执行期间线程尝试使用黄色区域中的内存，则触发保护故障，临时删除黄色区域的保护，并创建并抛出一个StackOverflowError。在解开线程执行堆栈以传播StackOverflowError之前，恢复黄色区域的保护。

如果线程尝试在其红色区域中使用内存，则JVM将立即分支到JVM错误报告代码，导致生成错误报告和JVM进程的崩溃转储。

由提出的解决方案定义的新区域位于黄色区域之前。如果线程在其调用堆栈中具有ReservedStackAccess注释方法，则该保留区域将像常规堆栈空间一样运行，否则为黄色区域。

在设置Java线程的执行堆栈期间，保留区域的保护方式与黄色区域和红色区域相同。如果在其执行期间线程与其保留区域相匹配，则产生SIGSEGV信号，并且信号处理器应用以下算法：

如果故障的地址在红色区域，则生成JVM错误报告和崩溃转储。

如果错误的地址在黄色区域，则创建并抛出一个StackOverflowError。

如果故障的地址在保留区域中，请执行堆栈步行，以检查调用堆栈中是否有注释jdk.internal.vm.annotation.ReservedStackAccess的方法。如果没有，创建并抛出一个StackOverflowError。如果找到注释方法，请删除对关键区域的保护，并将C ++ Thread对象中的最外层激活（框架）的堆栈指针与注释方法相关联。

如果保留区域的保护已被删除，以允许关键部分完成其执行，则必须恢复保护，并且一旦线程退出临界区，就会抛出延迟的StackOverflowError抛出。 HotSpot解释器已被修改，以检查是否正在退出注册的最外面的注释方法。通过将正在还原的堆栈指针的值与存储在C ++ Thread对象中的值进行比较，对每个帧激活删除执行检查。如果恢复的堆栈指针高于存储值（堆栈向下扩展），则在跳转到StackOverflowError生成代码之前，执行对运行时间的调用以更改内存保护并重置Thread对象中的堆栈指针值。这两个编译器已被修改为对方法退出执行相同的检查，但仅对于已保留的已注释方法或已注册方法的ReservedStackAccess注释方法进行编译。

当引发异常时，控制流不会通过常规方法退出代码，因此如果异常在注释方法之上传播，则保留区域的保护将无法正确恢复。为了防止这种情况，恢复保留区域的保护，并且每次异常开始传播时，都会重置存储在C ++ Thread对象中的堆栈指针值。在这种情况下，延迟的StackOverflowError不会被抛出。理由是抛出的异常比延迟的StackOverflowError更重要，因为它表示一个原因和正常执行中断的一个点。

抛出一个StackOverflowError是通知应用程序线程达到其堆栈限制的Java方式。然而，异常和错误有时被Java代码捕获，并且通知丢失或处理不正确，这可以使问题的调查真的很难。为了在存在保留堆栈区域的情况下缓解堆栈溢出错误的故障排除，JVM在授予访问保留堆栈区域时提供了另外两个通知：一个是由JVM打印的警告（与所有其他JVM消息在同一流中） ，第二个是JFR事件。请注意，即使延迟的StackOverflowError没有被抛出，因为在关键部分中抛出了另一个异常，所以生成了JVM警告和JFR事件并可用于进行故障排除。

保留堆栈功能由两个JVM标志控制，一个用于配置保留区域的大小（所有线程使用相同的大小），一个允许非特权代码使用该功能。将保留区域的大小设置为零将完全禁用该功能。禁用时，解释代码和编译代码不执行方法退出检查。

该解决方案的内存成本：对于每个线程，成本是其保留区域的虚拟内存，作为其堆栈空间的一部分。已经考虑了将不同存储区域中的保留区域实现为备用堆栈的选项。然而，这将大大增加任何堆栈行走码的复杂性，因此该选项已被拒绝。

性能成本：对ReentrantLocks进行的JSR-166测试所做的测量对x86平台的性能没有显着影响。

##### 性能

以下是该解决方案如何影响性能。

该解决方案中最昂贵的操作是在调用堆栈中查找注释方法时执行的堆栈执行。 只有当JVM检测到潜在的堆栈溢出时才执行此操作。 没有这个修复，JVM将抛出一个StackOverflowError。 所以即使操作相对昂贵，它比当前的行为更好，因为它将防止数据损坏。 该解决方案中最常执行的部分是当注释方法退出时执行的检查，以检查是否必须重新启用保留区域的保护。 此检查的性能关键版本在编译器中。 当前的实现将以下代码序列添加到注释方法的编译代码中：

```
0x00007f98fcef5809: cmp    rsp,QWORD PTR [r15+0x298]
0x00007f98fcef5810: jle    0x00007f98fcef583c
0x00007f98fcef5816: mov    rdi,r15
0x00007f98fcef5819: test   esp,0xf
0x00007f98fcef581f: je     0x00007f98fcef5837
0x00007f98fcef5825: sub    rsp,0x8
0x00007f98fcef5829: call   0x00007f9910f62670  ;   {runtime_call}
0x00007f98fcef582e: add    rsp,0x8
0x00007f98fcef5832: jmp    0x00007f98fcef583c
0x00007f98fcef5837: call   0x00007f9910f62670  ;   {runtime_call}
```
此代码适用于x86_64平台。在快速情况下（不需要重新启用保留区域的保护），它会添加两个指令，包括一个小跳转。 x86_32的版本较大，因为它没有在寄存器中始终可用的Thread对象的地址。该功能也适用于Solaris / SPARC。

##### 开放式问题

预留区域的默认大小仍然是一个开放的问题。这个大小将取决于使用ReservedStackAccess注释的JDK代码中最长的关键区域，并且还将取决于平台体系结构。我们还可以考虑不同的默认值，具体取决于JVM是在高端服务器还是在虚拟内存受限的环境中运行。

为了减轻调整问题，添加了调试/故障排除功能。此功能在调试版本上默认启用，并可在产品构建中作为诊断JVM选项使用。当激活时，它将在JVM即将抛出一个StackOverflowError时运行：它会调用调用堆栈，并且如果找到一个或多个注释为ReservedStackAccess注释的方法，则其名称将在JVM标准输出上打印一条警告消息。控制此功能的JVM标志的名称为PrintReservedStackAccessOnStackOverflow。

保留区域的默认大小是一页（4K），实验表明这足以覆盖目前已注释的java.util.concurrent锁的关键部分。

Windows平台上没有完全支持保留的堆栈区域。在Windows的功能开发过程中，发现了堆栈的特殊区域被控制的一个错误（JDK-8067946）。此错误阻止JVM授予对保留堆栈区域的访问权限。因此，当在Windows上检测到堆栈溢出条件，并且在调用堆栈上发出注释方法时，将打印JVM警告，触发JFR事件，并立即抛出一个StackOverflowError。应用程序的JVM的行为没有变化。但是，JVM警告和JFR事件可以帮助排除故障，表明发生潜在的有害情况。

##### 备择方案

已经考虑了几种替代方法，其中一些已被实施和测试。以下是这些方法的列表。

##### 基于语言的解决方案

try / catch / finally构造：它们不解决任何东西，因为不能保证finally子句也不会触发堆栈溢出。

新的结构如：

```
new CriticalSection(
       () -> {
           // do critical section code
        }).enter();
```
这种构造可能需要在javac和JVM中进行大量的工作，即使没有在堆栈溢出情况下运行，其使用可能对保留堆栈区域的性能有很大的影响。

代码转换解决方案：

避免方法调用（因为堆栈溢出检查在方法调用时执行）通过强制JIT将内联所有调用的方法：内联可能需要加载和初始化应用程序未使用的类，强制内联可能与编译器规则（代码大小，内联深度），内联不适用于所有代码模式（例如反射）。

代码重构以避免在源级别的方法调用：重构将需要修改已经复杂的代码（java.util.concurrent），并且这种重构将会破坏封装。

基于堆栈的解决方案：

扩展堆栈敲击：在进入关键部分之前进一步砰击堆栈：即使没有处于堆栈溢出状态，这个解决方案也具有性能成本，并且难以用嵌套的关键部分进行维护。

可扩展堆栈：从几个不连续的内存块构建堆栈，在检测到堆栈溢出时添加一个新块：此解决方案为JVM增加了很大的复杂性，以管理不连续的堆栈（包括当前基于堆栈中的指针比较的所有逻辑管理）;它也可能要求我们复制/移动堆栈的一些部分，并且由于分段问题而对内存分配后端施加更大的压力。

##### 测试

这种改变带有可靠的单元测试，可以重现由堆栈溢出引起的java.util.concurrent.lock.ReentrantLock损坏。

##### 依赖

保留堆栈区域依赖于“黄页”机制。这种机制目前在Windows JDK-8067946中部分被破坏，所以在这个平台上没有完全支持保留的堆栈区域。

#### JEP 271: Unified GC Logging
##### 概要

使用JEP 158中引入的统一JVM日志记录框架重新实现GC日志记录。

##### 非目标

确保当前GC日志解析器在新的GC日志中没有更改的情况下工作并不是一个目标。

并不是所有的日志条目都将以新的日志格式转载。

##### 描述

以与当前GC记录格式合理的方式重新实现GC记录。新格式和旧格式之间必然会有一些区别。

##### “gc”标签

这个想法是-Xlog：gc（仅登录在信息级别的“gc”标签）应该类似于-XX：+ PrintGC所做的，这是每个GC打印一行。这意味着应该非常仔细地使用log_info（gc）（“message”）。不要在“gc”标签的信息级别登录，除非是每个GC应该打印的一条消息。

如果与其他标签组合，使用“gc”标签登录信息级别是很好的。例如：

log_info（gc，heap，ergo）（“堆扩展”）;
这里的想法是-Xlog：gc应该有点类似于你用-XX：+ PrintGCDetails获得的东西。但是，此映射并不像从-Xlog：gc映射到-XX：+ PrintGC那样严格。 -XX：+ PrintGC的规则很清楚：每个GC一行。 -XX：+ PrintGCDetails的规则从未如此清楚。所以，一些-XX：+ PrintGCDetials日志记录可以映射到几个标签，一些可能被映射到“gc”标签的调试级别。

所有与GC相关的日志应使用“gc”标签。大多数日志记录不应仅使用“gc”标记，而应将其与其他标签相结合。

还有边界线案例，不清楚“gc”是否是适当的标签，例如分配代码。大多数情况下，大概不应该使用“gc”标签。

##### 其他标签

除了“gc”之外还有许多其他标签。其中有些地图很干净地映入旧旗。例如，PrintAdaptiveSizePolicy或多或少映射到“ergo”标签（与“gc”标签和潜在的其他标签相结合）。

##### 详细

由Verbose标志（开发标志）保护的大多数日志记录应该映射到跟踪级别。例如，如果从性能角度来看，这是非常昂贵的日志记录，在这种情况下，它将映射到开发级别。

##### 字首

统一日志记录框架中的前缀支持用于将GC标识添加到GC日志消息。 GC标识只对GC中发生的记录感兴趣。由于为特定标签集定义了前缀，即标签的组合，因此有必要确保在GC之间发生的日志记录不会使用与GC期间完成的记录相同的标记集。

##### 动态配置

一些日志记录要求在较早的状态下收集数据。统一的日志框架允许使用jcmd动态地打开和关闭所有日志记录。这意味着对于依赖于先前收集的数据的日志记录来说，检查日志是否已启用是不够的;还必须有支票来检查数据是否可用。

#### JEP 272: Platform-Specific Desktop Features
##### 概要

定义新的公共API来访问特定于平台的桌面功能，例如与任务栏或停靠点交互，或监听系统或应用程序事件。

##### 目标

在即将发布的JDK 9版本中，内部API（如Mac OS X com.apple.eawt软件包中的API）将无法再访问。该JEP的目标是为这些不可访问的API提供适当的替代，并且还引入相关的平台特定功能。我们计划在有必要支持的平台上实施新功能。在可行的情况下，API将被设计为跨平台，以便每个功能可以在尽可能广泛的平台上实现。正在删除apple.applescript类，而不提供任何替换。

##### 非目标

我们不打算直接替代JDK 8中存在的所有内部OS X API。具体来说，我们不会为com.apple.concurrent包提供替代。与我们提供替代品的内部API保持兼容性也不是目标。

##### 描述

该JEP包含两个子任务：

##### 提供公共API来替换com.apple中的功能。{eawt，eio}

此子任务的目的是避免OS X开发人员的功能丢失。我们将在JDK-internal com.apple.eawt和com.apple.eio包中提供API的替换。

##### 提供对其他平台上类似功能的访问

除了特定于OS X的功能之外，还有其他平台（包括Windows和Linux）可以支持的类似功能：

登录/注销和屏幕锁处理程序：在系统登录/注销（或屏幕锁定/解锁）上提供事件侦听器，以允许应用程序启动持久性任务或在需要时保存其状态。

任务栏/码头交互

请求用户注意：允许应用程序请求用户的注意，使用平台功能，例如闪烁任务栏中的应用程序图标或弹出Dock中的图标。

指示任务进度：在任务栏/停靠栏中显示进度条或其他指示符。

操作快捷方式：提供可由弹出菜单访问的操作快捷方式，例如Windows跳转列表。

这些功能的支持将由平台功能决定。

##### API

我们建议将这两个子任务的公共API添加到现有的java.awt.Desktop类中。目标支持的平台是Mac OS X，Windows，Linux。

拟议的API草图：

```
package java.awt;

public class Desktop {

    /* ... */

    /**
     * Adds sub-types of {@link AppEventListener} to listen for notifications
     * from the native system.
     *
     * @param listener
     * @see AppForegroundListener
     * @see AppHiddenListener
     * @see AppReOpenedListener
     * @see AppScreenSleepListener
     * @see AppSystemSleepListener
     * @see AppUserSessionListener
     */

    public void addAppEventListener(final AppEventListener listener) {}

    /**
     * Requests user attention to this application (usually through bouncing the Dock icon).
     * Critical requests will continue to bounce the Dock icon until the app is activated.
     *
     */
    public void requestUserAttention(final boolean critical) {}

    /**
     * Attaches the contents of the provided PopupMenu to the application's Dock icon.
     */
    public void setDockMenu(final PopupMenu menu) {}

    /**
     * Changes this application's Dock icon to the provided image.
     */
    public void setDockIconImage(final Image image) {}


    /**
     * Affixes a small system provided badge to this application's Dock icon. Usually a number.
     */
    public void setDockIconBadge(final String badge) {}

    /**
     * Displays or hides a progress bar or other indicator in
     * the dock.
     *
     * @see DockProgressState.NORMAL
     * @see DockProgressState.PAUSED
     * @see DockProgressState.ERROR
     *
     * @see #setDockProgressValue
     */
    public void setDockProgressState(int state) {}

    /**
     * Sets the progress bar's current value to {@code n}.
     */
    public void setDockProgressValue(int n) {}

    /**
     * Tests whether a feature is supported on the current platform.
     */

    public boolean isSupportedFeature(Feature f) {}

    /* ... */
}
```

##### 测试

测试将限于为使用新API而编写的其他手动测试。 测试将需要检查新功能是否受到预期支持的平台的支持，并且它们在不支持的平台上正常失败。

#### JEP 273: DRBG-Based SecureRandom Implementations
##### 概要

实现NIST 800-90Ar1中描述的三个确定性随机位发生器（DRBG）机制。

##### 非目标

为熵输入源（SEI）提供API，或在所有平台上实施已批准的SEI，其中“已批准”表示由NIST或FIPS批准。

##### 动机

JDK有两种SecureRandom实现。一个是平台依赖的，并且基于本机调用或OS设备，例如在Unix上阅读/ dev / {u}随机，在Windows上使用CryptoAPI，并使用各种预配置的PKCS11库。 Solaris，Linux和Windows的最新版本已经支持DRBG，但旧版本和嵌入式系统可能不支持。另一种是使用基于旧的基于SHA1的RNG实现的纯Java实现，其不如被批准的DRBG机制使用的算法那么强。

由NIST（如SP 800-90Ar1）开发和批准的DRBG机制使用与SHA-512和AES-256一样强大的现代算法。这些机制中的每一个都可以配置不同的安全优势和功能，以匹配用户需求。这些机制的支持在一些环境中变得非常重要，特别是美国政府。

##### 描述

根据NIST SP 800-90，使用熵输入源（800-90B和800-90C）和DRBG机制（800-90Ar1）构造随机位发生器（RBG，800-90C）。熵输入源提供新鲜随机（熵）作为DRBG机制的种子，然后能够连续生成“随机”比特序列。

##### APIs

SecureRandom匹配800-90C的新方法，允许配置一个SecureRandom对象，并在播种，重新种植和随机生成的过程中指定额外的输入。

SecureRandomSpi中的新方法，以实现上述新方法。

一个新的SecureRandomParameters接口，以便可以向新的SecureRandom方法提供额外的输入。

这些新API应该适用于任何SecureRandom风格（不仅仅是DRBG），并且可以添加到SecureRandom和SecureRandomSpi。

一个新的DrbgParameters类（及其内部类）实现DRBG使用的SecureRandomParameters。
##### 履行

在800-90Ar1（在所有平台上）实现三个DRBG机制（Hash_DRBG，HMAC_DRBG，CTR_DRBG）。
##### 副产品

SHA-512/224和SHA-512/256安全散列算法，如FIPS 180-4所述。

相关HmacSHA512 / 224和HmacSHA512 / 256算法。

##### 测试

DRBG实现必须通过CAVP测试向量。

SHA-512/224和SHA-512/256测试载体。

HmacSHA512 / 224和HmacSHA512 / 256的非正式测试载体。

#### JEP 274: Enhanced Method Handles
##### 概要

增强java.lang.invoke包的MethodHandle，MethodHandles和MethodHandles.Lookup类，以简化常见的用例，并通过新的MethodHandle组合器和查询细化来实现更好的编译器优化。

##### 目标

在java.lang.invoke包中的MethodHandles类中，为循环和try / finally块提供新的MethodHandle组合器。

使用新的MethodHandle组合器来增强MethodHandle和MethodHandles类的参数处理。

在MethodHandles.Lookup类中实现接口方法和可选的超级构造函数的新查找。

##### 非目标

除了可能需要的本机功能之外，VM级别扩展和增强功能（特别是编译器优化）是非目标。

Java语言级别的扩展显然超出范围。

##### 动机

在mlvm-dev邮件列表（第1部分，第2部分）的一个线程中，开发人员已经讨论了java.lang.invoke包中的MethodHandle，MethodHandles和MethodHandles.Lookup类的可能扩展，以使常见用例的实现更容易，并且还允许被认为重要但目前不被支持的用例。

下面提出的扩展不仅允许更简洁地使用MethodHandle API，而且还减少了在某些情况下创建的MethodHandle实例的数量。这反过来将有助于更好地优化VM的编译器。

##### 更多声明的组合者

循环。 MethodHandles类从MethodHandle实例中不提供循环构造的抽象。应该有一种方法来构造循环的方法来代表循环的body，以及初始化和条件或计数。

Try/finally 块。 MethodHandles也没有为try / finally块提供抽象。应该提供一种从代表try和finally部件的方法句柄构造这些块的方法。

##### 更好的论证处理

争论蔓延。使用MethodHandle.asSpreader（Class <？> arrayType，int arrayLength），存在一个创建一个方法句柄的操作，该方法句柄将数组参数的内容扩展到多个参数。应该提供一个额外的asSpreader方法，允许在方法签名中的任何位置将包含在数组中的一些参数扩展到多个不同的参数。

参数集合。 MethodHandle.asCollector（Class <？> arrayType，int arrayLength）的方法生成一个句柄，它将数组的尾随的数组长度参数收集起来。方法签名中其他地方的许多参数没有办法实现相同的方法。应该有一个额外的asCollector方法来支持这个。

参数折叠。折叠组合器foldArguments（MethodHandle target，MethodHandle combinator）不允许控制参数列表中哪个折叠应该开始的位置。应增加立场说明;折叠的参数数量隐式地表示为组合器接受的参数数。

##### 更多查找功能

接口中的非抽象方法。目前，这样的用例将在运行时在指定的位置失败：

```
interface I1 {
    default void m() { System.err.println("I1.m"); }
}

interface I2 {
    default void m() { System.err.println("I2.m"); }
}

class C implements I1, I2 {
    public void m() { I2.super.m(); System.err.println("C.m"); }
}

public class IfcSuper {
    public static void main(String[] args) throws Throwable {
        C c = new C();
        MethodHandles.Lookup l = MethodHandles.lookup();
        MethodType t = MethodType.methodType(void.class);
        // This lookup will fail with an IllegalAccessException.
        MethodHandle di1m = l.findSpecial(I1.class, "m", t, C.class);
        ci1m.invoke(c);
    }
}
```

然而，应该可以构造在接口中绑定到非抽象方法的MethodHandles。

类查找。最后，查找API应该允许从不同的上下文中查找类，这是目前不可能的。在MethodHandles区域中，所有必需的访问检查都是在查找时间完成的（与运行时相反，就像反射一样）。类根据它们的.class实例传递。为了便于在上下文（例如，跨模块边界）上进行某种控制的查找，应该有一种查找方法，它可以传递具有正确限制的Class实例，以便在MethodHandle组合器中​​进一步使用。

描述

循环组合

大多数通用循环抽象

循环的核心抽象包括循环的初始化，要检查的谓词和要求评估的主体。用于创建循环的最通用的MethodHandle组合器将被添加到MethodHandles中，如下所示：

MethodHandle循环（MethodHandle [] ...子句）
构造一个表示循环的方法句柄，该循环具有在每次迭代时更新和检查的多个循环变量。在由于其中一个谓词终止循环后，运行相应的终结器，并传递循环的结果，循环的结果是结果句柄的返回值。

直观地，每个循环由一个或多个“子句”形成，每个“子句”指定本地迭代值和/或循环出口。循环的每次迭代按顺序执行每个子句。一个子句可以选择更新其迭代变量;它还可以选择执行测试和条件循环退出。为了在方法句柄方面表达这个逻辑，每个子句将决定四个动作：

在循环执行之前，迭代变量或循环不变本地的初始化。

当一个子句执行时，迭代变量的更新步骤。

当一个子句执行时，一个谓词执行来测试循环退出。

如果一个子句导致循环退出，则执行终结器来计算循环的返回值。

这些条款部分中的一些可以根据某些规则被省略，并且在这种情况下提供有用的默认行为。详见下文。

除了子句初始化器之外，每个子句功能都能够观察整个循环状态，因为它将传递所有当前迭代变量值以及所有传入循环参数。大多数子句函数不需要所有这些信息，但它们将被正确连接，就像通过dropArguments一样。

给定一组子句，执行了多个检查和调整，以连接循环的所有部分。它们在下面的步骤中详细说明。在这些步骤中，如果循环组合器的输入不满足所需约束，则单词“必需”的每次出现都对应于可能抛出IllegalArgumentException的地方。应用于参数类型列表的术语“有效相同”意味着它们必须相同，否则一个列表必须是另一个的正确前缀。

步骤0：确定子句结构。

子句数组（类型为MethodHandle \[] []必须非空，并且至少包含一个元素。

子句数组可能不包含长于四个元素的空值或子数组。

短于四个元素的条款被视为被零元素填充到四分之一。填充通过将元素附加到数组来进行。

具有所有null的条款被忽略。

每个子句被视为一个四元组的函数，称为“init”，“step”，“pred”和“fini”。

步骤1A：确定迭代变量。

检查init和step函数返回类型，成对确定每个子句的迭代变量类型。

如果省略这两个功能，请使用void;否则如果省略，请使用其他的返回类型;否则使用通用返回类型（它们必须相同）。

形成返回类型列表（按子句顺序），省略所有出现的void。

这种类型的列表被称为“公共前缀”。

步骤1B：确定循环参数。

检查init函数参数列表。

省略的init函数被视为具有空参数列表。

所有init函数参数列表必须有效地相同。

最长的参数列表（必须是唯一的）称为“公共后缀”。

步骤1C：确定回路返回类型。

检查fini函数返回类型，忽略忽略fini函数。

如果没有fini函数，则使用void作为循环返回类型。

否则，使用通用返回类型的fini函数;它们都必须相同。

步骤1D：检查其他类型。

必须至少有一个非省略的pred函数。

每个非省略的pred函数必须具有布尔返回类型。

（实施注意：步骤1A，1B，1C，1D在逻辑上彼此独立，并且可以以任何顺序执行。）

步骤2：确定参数列表。

结果循环句柄的参数列表将是“常用后缀”。

init函数的参数列表将被调整为“常用后缀”。 （请注意，它们的参数列表已经与公共后缀有效地相同。）

non-init（step，pred和fini）函数的参数列表将被调整为公共前缀，后跟公共后缀，称为“公共参数序列”。

每个非init，非省略的功能参数列表必须与公共参数序列有效地相同。

步骤3：填写省略功能。

如果省略一个init函数，请使用相应的null / zero / false / void类型的常量函数。 （为此，一个常量void只是一个不做任何事情并返回void的函数;它可以通过MethodHandle.asType类型通过类型转换从另一个常量函数中获取。）

如果省略步骤功能，请使用子句迭代变量类型的身份函数;在前面的子句的非void迭代变量的identity函数的前面插入参数参数。 （这将使循环变量变为本地循环不变量。）

如果省略了pred功能，则也必须省略相应的fini功能。

如果省略了pred函数，则使用常量true函数。 （就本条而言，这将保持循环。）

如果省略了一个fini函数，则使用循环返回类型的常量null / zero / false / void函数。

步骤4：填写缺少的参数类型。

此时，每个init函数参数列表与公用后缀有效地相同，但有些列表可能更短。对于具有短参数列表的每个初始化函数，通过删除参数来填充列表的末尾。

此时，每个非init函数参数列表与公共参数序列有效地相同，但有些列表可能更短。对于每个具有短参数列表的非init函数，通过删除参数来排除列表的末尾。

最终观察。

在这些步骤之后，通过提供省略的函数和参数来调整所有子句。

所有init函数都有一个通用参数类型列表，最后一个循环句柄也将具有。

所有fini函数都有一个常见的返回类型，最后一个循环句柄也将具有。

所有非init函数都有一个公共参数类型列表，它是通用参数序列，（非空）迭代变量后跟循环参数。

每对init和step函数在返回类型中都是一致的。

每个非init函数将能够通过公共前缀来观察所有迭代变量的当前值。

循环执行。

当循环被调用时，循环输入值被保存在本地，将被传递给每个子句函数（作为公共后缀）。这些本地是循环不变的。

每个init函数按照子句顺序（传递公共后缀）执行，非空值值作为公共前缀保存到本地文件中。这些本地人是循环变化的（除非他们的步骤是身份功能，如上所述）。

所有函数执行（init函数除外）将被传递通用参数序列，由非void迭代值（按子句顺序）和循环输入（按参数顺序）组成。

然后按照子句顺序（pred before pred）执行step和pred函数，直到pred函数返回false。

来自步骤函数调用的非空值结果用于更新相应的循环变量。所有后续的函数调用都会立即显示已更新的值。

如果pred函数返回false，则调用相应的fini函数，并从循环返回结果值。

从循环返回的MethodHandle l的语义如下：

```
l(arg*) =>
{
    let v* = init*(arg*);
    for (;;) {
        for ((v, s, p, f) in (v*, step*, pred*, fini*)) {
            v = s(v*, arg*);
            if (!p(v*, arg*)) {
                return f(v*, arg*);
            }
        }
    }
}
```
基于这种最通用的循环抽象，应该将多个方便的组合器添加到MethodHandles中。 以下讨论它们。

##### 简单的while和do-while循环

这些组合器将被添加到MethodHandles中：

```
MethodHandle whileLoop(MethodHandle init, MethodHandle pred, MethodHandle body)

MethodHandle doWhileLoop(MethodHandle init, MethodHandle body, MethodHandle pred)
```

调用从whileLoop返回的MethodHandle对象的语义如下：

```
wl(arg*) =>
{
    let r = init(arg*);
    while (pred(r, arg*)) { r = body(r, arg*); }
    return r;
}
```
对于从doWhileLoop返回的MethodHandle dwl，语义如下：

```
dwl(arg*) =>
{
    let r = init(arg*);
    do { r = body(r, arg*); } while (pred(r, arg*));
    return r;
}
```
该方案对三个组成的MethodHandles可以具有的签名施加一些限制：

初始化器init的返回类型也是身体和整个循环的返回类型，以及谓词pred和body body的第一个参数的类型。

谓词pred的返回类型必须是布尔值。

##### 计数循环

为方便起见，还将提供以下回路组合器：

MethodHandle countingLoop（MethodHandle iterations，MethodHandle init，MethodHandle body）

从countLoop返回的MethodHandle cl具有以下语义：

```
cl(arg*) =>
{
    let end = iterations(arg*);
    let r = init(arg*);
    for (int i = 0; i < end; i++) {
        r = body(i, r, arg*);
    }
    return r;
}
```

MethodHandle countingLoop（MethodHandle start，MethodHandle end，MethodHandle init，MethodHandle body）
从CountLoop的这个变体返回的MethodHandle cl具有以下语义：

```
cl(arg*) =>
{
    let s = start(arg*);
    let e = end(arg*);
    let r = init(arg*);
    for (int i = s; i < e; i++) {
        r = body(i, r, arg*);
    }
    return r;
}
```

在这两种情况下，body的第一个参数的类型必须为int，init和body的返回类型以及body的第二个参数必须相同。

##### 数据结构迭代

此外，迭代的循环组合是有帮助的：

MethodHandle iteratedLoop（MethodHandle iterator，MethodHandle init，MethodHandle body）
从iteratedLoop返回的MethodHandle具有以下语义：

```
it(arg*) =>
{
    let it = iterator(arg*);
    let v = init(arg*);
    for (T t : it) {
        v = body(t, v, a);
    }
    return v;
}
```

##### 备注
可以想到更方便的环路组合器。

虽然继续的语义可以很容易地通过从身体返回来模拟，但是如何可以模拟休息语义是一个悬而未决的问题。 这可以通过使用专用异常（例如，LoopMethodHandle.BreakException）来实现。

##### try/finally块组合器

为了方便使用来自MethodHandles的try / finally语义来构建功能，以下新的组合器将被引入到MethodHandles中：

MethodHandle tryFinally（MethodHandle target，MethodHandle cleanup）

从tryFinally返回的调用MethodHandle tf的语义如下：

```
tf(arg*) =>
{
    Throwable t;
    Object r;
    try {
        r = target(arg*);
    } catch (Throwable x) {
        t = x;
        throw x;
    } finally {
        r = cleanup(t, r, arg*);
    }
    return r;
}
```

也就是说，所得到的MethodHandle的返回类型将是目标句柄的返回类型。目标和清理都必须具有匹配的参数列表，扩展名为cleanup，它接受一个Throwable参数和 - 可能是中间结果。如果在执行目标期间抛出异常，则此参数将保留该异常。

##### 争议处理组合者

作为MethodHandles中现有API的补充，将介绍以下方法：

添加类MethodHandle - 新的实例方法：

```
MethodHandle asSpreader(int pos, Class<?> arrayType, int arrayLength)
```
在结果的签名中，在位置pos，期望arrayType类型的arrayLength参数。在结果中，插入一个消耗此MethodHandle的arrayLength参数的数组。如果该签名在该位置没有足够的参数，或者如果该位置不在签名中，则提出适当的异常。

例如，如果这个签名是（Ljava / lang / String; IIILjava / lang / Object;）V，则调用asSpreader（int []。class，1，3））将导致生成的签名（Ljava / lang / String [ILjava /郎/对象;）V。

添加类MethodHandle - 新的实例方法：

```
MethodHandle asCollector(int pos, Class<?> arrayType, int arrayLength)
```
在签名中，在位置pos，期望一个数组参数。在结果的签名中，在位置pos中，将有该数组类型的arrayLength参数。 pos之前的所有参数都不受影响。 pos之后的所有参数都被arrayLength向右移动。期望在运行时在数组中提供要传播的参数;如果它们不是，则抛出ArrayIndexOutOfBoundsException。

例如，如果这个签名是（Ljava / lang / String; [ILjava / lang / Object;）V，则调用asCollector（int []。class，1,3））将导致生成的签名（Ljava / lang /串; IIILjava /郎/对象;）V。

添加类MethodHandles - 新的静态方法：

```
MethodHandle foldArguments(MethodHandle target, int pos, MethodHandle combiner)
```
所产生的MethodHandle在被调用时将像现有的方法foldArguments（MethodHandle target，MethodHandle combiner）一样，区别在于已经存在的方法意味着折叠位置为0，而所提出的新方法允许指定除0之外的折叠位置。

例如，如果目标签名是（ZLjava / lang / String; ZI）I，并且组合器签名是（ZI）Ljava / lang / String;则调用foldArguments（target，1，combiner）将导致结果签名ZZI）和第二和第三（boolean和int）参数将在每次调用时被折叠成String。

这些新的组合器将使用现有的抽象和API来实现。如果需要，非公开API将被修改。

##### 查找

将修改MethodHandles.Lookup.findSpecial（Class <？> refc，String name，MethodType类型，Class <？> specialCaller）的方法的实现，以便在接口上找到超级可调用的方法。虽然这不是API的变化，但其记录的行为显着变化。

此外，MethodHandles.Lookup类将通过以下两种方法进行扩展：

类<？> findClass（String targetName）

这将检索表示由targetName标识的所需目标类的Class <？>的实例。查找应用由隐式访问上下文定义的限制。如果访问不可行，则该方法会引发适当的异常。

类<？> accessClass（Class <？> targetClass）

这将尝试访问给定的类，应用由隐式访问上下文定义的限制。如果访问不可行，则该方法会引发适当的异常。

##### 风险与假设

由于这是一个纯粹的附加API扩展，所以没有使用MethodHandle API的现有客户端的代码将受到负面影响。提议的扩展也不依赖于任何其他正在进行的开发。

将提供所有上述API扩展的单元测试。

#####  依赖性

该JEP与JEP 193（可变句柄）相关，由于VarHandles取决于MethodHandle API，因此可以进行一定的重叠。这将与JEP 193的所有者合作解决。

关于维护版本的JSR 292增强功能的JBS问题可以被认为是这个JEP的起点，从这个问题中可以看出达成协议的点。







