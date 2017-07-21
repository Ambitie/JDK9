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
#####概要

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
#####概要

为JVM的所有组件引入通用日志记录系统。

#####目标

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
#####非目标

在此JEP的范围之外，添加来自所有JVM组件的实际日志记录调用。这个JEP只会提供进行日志记录的基础设施。

除了装饰的格式和人类可读的纯文本的使用之外，它还超出了JEP的范围，以执行记录格式。

此JEP不会将日志记录添加到JDK中的Java代码。

#####动机

JVM是复杂的系统级组件，根本原因分析通常是一项艰巨而耗时的任务。没有广泛的可用性功能，通常几乎不可能在生产环境中找到间歇性崩溃或性能怪癖的根本原因。可以使用支持和维护工程的细粒度易于配置的JVM记录是一个这样的功能。

JRockit有一个相似的功能，它有助于为客户提供支持。

#####描述

#####标签

日志框架定义了JVM中的一组标签。每个标签都以其名称（例如：gc，编译器，线程等）标识。可以根据需要在源代码中更改标签集。当添加日志消息时，它应该与对记录的信息进行分类的标签集相关联。标签集由一个或多个标签组成。

#####水平

每个日志消息都具有与其关联的日志记录级别。可用的级别是错误，警告，信息，调试，跟踪和开发以增加的冗长顺序。开发级仅适用于非产品构建。

对于每个输出，可以配置记录级别以控制写入该输出的信息量。替代关闭完全禁用日志记录。

#####饰

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
#####输出

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
> -Xlog[:option]
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
                       
“all”标签是由所有可用标签集组成的元标记。 'tag-set'定义中的'*'表示“通配符”标签匹配。 不使用'*'表示“所有与所指定的标签匹配的信息”。

省略“what”将默认标记所有和级别的信息。

省略“level”将默认为信息

省略'output'将默认为stdout

省略“decorators”将默认为正常运行时间，级别，标签

“none”装饰器是特殊的，用于关闭所有装饰。

提供的级别是汇总的。 例如，如果输出配置为使用'level'信息。 将输出所有与“什么”中的标签与日志级别信息，警告和错误相匹配的消息。

> -Xlog:disable
    this turns off all logging and clears all configuration of the
    logging framework. Even warnings and errors.
-Xlog:help
    prints -Xlog usage syntax and available tags, levels, decorators
    along with some example command lines.
    
#####默认配置：
> -Xlog:all=warning:stderr:uptime,level,tags
    - default configuration if nothing is configured on command line
    - 'all' is a special tag name aliasing all existing tags
    - this configuration will log all messages with a level that
    matches ´warning´ or ´error´ regardless of what tags the
    message is associated with

#####简单的例子：
> -Xlog   

等价于
>-Xlog:all
    - log messages using 'info' level to stdout
    - level 'info' and output 'stdout' are default if nothing else
    is provided
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:gc
    - log messages tagged with 'gc' tag using 'info' level to
    'stdout'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:gc=debug:file=gc.txt:none
    - log messages tagged with 'gc' tag using 'debug' level to
    a file called 'gc.txt' with no decorations
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:gc=trace:file=gctrace.txt:uptimemillis,pids:filecount=5,filesize=1024
    - log messages tagged with 'gc' tag using 'trace' level to
    a rotating fileset with 5 files with size 1MB with base name
    'gctrace.txt' and use decorations 'uptimemillis' and 'pid'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:gc::uptime,tid
    - log messages tagged with 'gc' tag using default 'info' level to
    default output 'stdout' and use decorations 'uptime' and 'tid'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:gc*=info,rt*=off
    - log messages tagged with at least 'gc' using 'info' level but turn
    off logging of messages tagged with 'rt'
    - messages tagged with both 'gc' and 'rt' will not be logged
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:disable -Xlog:rt=trace:rttrace.txt
    - turn off 'all' logging, even warnings and errors, except
    messages tagged with 'rt' using 'trace' level
    - output to a file called 'rttrace.txt'
    
##### 复杂的例子：
> -Xlog:gc+rt*=debug
    - log messages tagged with at least 'gc' and 'rt' tag using 'debug'
    level to 'stdout'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:gc+meta*=trace,rt*=off:file=gcmetatrace.txt
    - log messages tagged with at least 'gc' and 'meta' tag using 'trace'
    level to file 'metatrace.txt' but turn off all messages tagged
    with 'rt'
    - again, messages tagged with 'gc', 'meta' and 'rt' will not be logged
    since 'rt' is set to off
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:gc+meta=trace
    - log messages tagged with exactly 'gc' and 'meta' tag using 'trace'
    level to 'stdout'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect

> -Xlog:gc+rt+compiler*=debug,meta*=warning,svc*=off
    - log messages tagged with at least 'gc' and 'rt' and 'compiler' tag
    using 'trace' level to 'stdout' but only log messages tagged
    with 'meta' with level 'warning' or 'error' and turn off all
    messages tagged with 'svc'
    - default output of all messages at level 'warning' to 'stderr'
    will still be in effect
    
#####在运行时控制
可以通过诊断命令（jcmd实用程序）在运行时控制日志记录。可以使用诊断命令动态地指定在命令行中指定的所有内容。由于诊断命令会自动显示为MBean，因此可以使用JMX在运行时更改日志记录配置。

对日志记录配置和参数进行枚举的辅助支持将被添加到运行时控制命令列表中。

#####JVM界面

在JVM中，将使用类似于以下API的API创建一组宏：

log_<level>(Tag1[,...])(fmtstr, ...)
    syntax for the log macro
#####例：

>log\_info(gc, rt, classloading)("Loaded %d objects.", object_count)
    the macro is checking the log level to avoid uneccessary
    calls and allocations.

>log_debug(svc, debugger)("Debugger interface listening at port %d.", port_number)

#####等级信息：

>LogHandle（gc，meta，classunloading）log;
if（log.is_trace（））{
    ...
}
if（log.is_debug（））{
    ...
}

为避免执行生成仅用于记录的数据的代码，可以向Log类询问当前配置的日志级别。

#####性能

不同的日志级别应具有定义级别的预期性能开销的准则。例如：“警告级别不应影响性能;信息级别对于生产应该是可以接受的;调试，跟踪和错误级别不具备性能要求。运行日志禁用应该尽可能少的性能影响。通常会记录成本。

#####未来可能的扩展

将来，添加一个用于将日志消息写入此基础架构的Java API可能是有用的，供JDK中的类使用。

最初只有三个后端将被开发：stdout，stderr和file。未来的项目可能会增加其他后端。例如：syslog，Windows事件查看器，套接字等

#####开放式问题

我们应该在API中提供一个替代方案来提供作为宏的参数的级别吗？
如果装饰物被[]以外的其他东西包围，以便更容易解析输出？
datestamp装饰的确切格式是什么？提出ISO 8601。
#####测试

非常重要的是，登录本身不会引起任何不稳定，因此需要进行广泛的测试。

功能测试必须通过启用某些日志消息并检查其在stderr或文件上的存在来完成。

因为可以动态启用日志记录，所以我们需要通过在运行应用程序时持续启用和禁用日志记录来强调测试。

日志框架将使用单元测试进行测试。

#####风险与假设

上述设计可能无法满足当今日志记录的所有用途。如果是这种情况，则必须重新设计。

#####碰撞

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

#####指令

控制JVM编译器的所有选项将被收集到一组选项中。一组具有值的选项称为编译器指令，是一个如何编译的指令。一个指令与一个方法匹配器一起提供给VM，该方法匹配器决定了它适用于哪些方法。同时可以在运行时激活几个指令，但是只有一个指令应用于特定的编译。在运行时可以添加和删除指令。

#####指令格式

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
#####指令示例1

>[//如果指令数组，则启动
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

#####指令示例2
>[
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

#####指令选项列表

第一个实现包含以下选项。所有选项以前已在CompileCommand选项命令中使用。将添加更多选项。

公共标志： Enable, bool Exclude, bool BreakAtExecute, bool BreakAtCompile, bool Log, bool PrintAssembly, bool PrintInlining, bool PrintNMethods, bool ReplayInline, bool DumpReplay, bool DumpInline, bool CompilerDirectivesIgnoreCompileCommands, bool Inline, ccstr[]

C2 only：BlockLayoutByFrequency，bool PrintOptoAssembly，bool PrintIntrinsics，bool raceOptoPipelining，bool TraceOptoOutput，bool TraceSpilling，bool Vectorize，bool VectorizeDebug，bool CloneMapDebug，bool IGVPrintLevel，intx MaxNodeLimit，intx DisableIntrinsics，ccstr

>inline：<one pattern or a array of string patterns>
该模式是与方法名称匹配的字符串的方式与指令匹配相同。
模式中的'+'表示方法匹配应强制内联。
应该防止内联的' - '。
使用匹配的第一个模式的命令。
示例1：inline：[“+ java / lang /*.*”， - “sun *。*”]
示例2：inline：“+ java / lang /*.*”

#####指令模式

在“match”和“inline”-option中使用的方法模式具有以下模式：Class.method（signature）

类包括由/ Class分隔的包名称，方法可以带前导和尾随\*的通配符，或者代替\*如果签名被省略，则默认为\*

这些是有效的模式：: "java/lang/String,indexOf" "/lang/String,indexOf(I)" "java/lang/String.(I)" "java/lang/String.()" "." "java/lang/."

#####指令解析器

指令解析器负责解析指令文件，并将信息添加到VM内部格式。

如果在命令行上指定了格式错误的伪指令文件，VM将打印错误并退出。如果通过诊断命令添加了一个格式不正确的伪指令文件，它将被忽略，并且将打印一个正确的警告。

解析器将验证所有选项都有效。依赖平台的选项将在不支持平台的平台上打印警告。理由是相同的指令文件应该可以使用，无论其部署在哪个平台上。

未指定的选项将使用默认值。如果指定了一个默认值的命令行选项。方法模式的默认值为“。” （匹配所有方法）。

#####编译经纪人

compilerBroker具有包含所有应用指令的指令栈。 bottom指令是默认设置，永远不能被删除。当文件加载附加指令时，将以相反的顺序添加文件，文件中的第一个指令将以堆栈的顶部结尾。这是一个可用性功能。

当为编译提交一个方法时，compileBroker将选择匹配的第一个指令并将其传递给编译器。 compilerBroker和编译器将忽略将会创建坏代码的选项（强制在不支持它的平台上强制硬件指令），并且将发出适当的警告。指令选项与正常的命令行标志具有相同的限制 - 例如强制内联只会在IR不会变大的情况下被尊重。

#####命令行界面

一个指令文件可以添加一个命令行。如果标志错误（正常命令行解析），文件丢失或文件内容格式错误，则VM将退出并显示错误消息。

>-XX：CompilerDirectivesFile = <文件>

##### 诊断命令界面

这些是与编译器控件一起使用的诊断命令：

>jcmd <pid> Compiler.add_directives <file>
从文件添加其他指令。 新的指令将被添加在旧的指令之上，文件中的第一个指令最终位于指令栈的顶部。

>jcmd <pid> Compiler.list_directives
从上到下列出指令堆栈上的所有指令。

>jcmd <pid> Compiler.clear_directives
清除指令栈

>jcmd <pid> Compiler.remove_directives
从指令栈中删除顶部元素

#####编译命令和向后兼容性

CompilerControl应该在所有用例中替换CompileCommand。 CompileCommand将保持向后兼容性，目标是尽可能保持行为。

有四层可以应用的控制。编译器控制将具有最高优先级，并覆盖任何其他标志或命令。二是CompileCommand，第三是任何命令行标志，第四是默认标志值。如果同时使用Compiler控件和CompileCommand，那么Compiler控件就会认为CompileCommand会覆盖默认值。

如果使用CompileCommand和编译器指令，JVM应该打印一个警告。

#####方法模式

编译器控件将使用与CompileCommand相同的方法模式格式。该模式由包和类名称，方法名称和签名三部分组成。这三个中的任何一个可能是带有前导或尾随*的通配符。任何部分的默认值为*。

例：

java/example/Test.split
由三部分组成

jjava/example/Test + split + (Ljava/lang/String;)Ljava/lang/String;
#####风险与假设

编译器选项的绝对数量将限制我们最初专注于一个子集。我们将专注于一个子集，并从那里扩展。

#####依赖性

诊断命令 - 已经到位
使用完整的JDK - 已经到位
#####碰撞

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
>class Foo {
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

在生成并返回VarHandle之前，对VarHandle的查询将在生成并返回VarHandle之前执行完全相同的访问控制检查（代表查找类），这些检查通过查找可以读取和写入访问权限的MethodHandle来执行相同的字段（参见MethodHandles.Lookup类中的find {，Static} {Getter，Setter}方法）。

访问模式方法将在以下条件下调用时抛出UnsupportedOperationException：

将VarHandle的访问模式方法写入最终字段。

基于数字的访问模式方法（getAndAdd和addAndGet）用于引用变量类型或非数字类型（如布尔值）。

引用变量类型或浮点型和双类型的基于位的访问模式方法（后面的限制可能会在将来的修订版中删除）

对于相关的VarHandle，一个字段不需要标记为volatile，以执行易失性访问。实际上，挥发性改性剂（如果存在）被忽略。这与java.util.concurrent.atomic.Atomic {Int，Long，Reference} FieldUpdater的行为不同，其中相应的字段必须标记为volatile。在某些情况下，这可能是太限制性的，在某些情况下，已知某些易失性访问并不总是需要的。

为基于数组的变量类型创建VarHandle实例的方法位于java.lang.invoke.MethodHandles中（请参阅MethodHandles类中的arrayElement {Getter，Setter}方法）。例如，可以创建一个VarHandle到int数组，如下所示：
> VarHandle intArrayHandle = MethodHandles.arrayElementVarHandle(int[].class);

访问模式方法将在以下条件下调用时抛出UnsupportedOperationException：

基于数字的访问模式方法（getAndAdd和addAndGet）用于数组组件引用变量类型或非数字类型（如布尔值）

引用变量类型或浮点型和双类型的基于位的访问模式方法（后面的限制可能会在将来的修订版中删除）

对于实例字段，静态字段和数组元素的变量类型，支持所有原始类型和引用类型。 其他可变类型可以支持这些类型的全部或一部分。

为基于数组视图的变量类型创建VarHandle实例的方法也位于java.lang.invoke.MethodHandles中。 例如，可以创建一个VarHandle来查看字节数组作为未对齐的long数组，如下所示：
VarHandle longArrayViewHandle = MethodHandles.byteArrayViewVarHandle(
        long[].class, java.nio.ByteOrder.BIG_ENDIAN);
虽然可以使用java.nio.ByteBuffer实现类似的机制，但它要求创建一个ByteBuffer实例来包装一个字节数组。由于逃脱分析的脆弱性，这并不总是保证可靠的性能，访问必须经过ByteBuffer实例。在未对齐访问的情况下，除普通访问模式之外，所有方法将抛出IllegalStateException。对于对齐访问，某些易失性操作取决于变量类型是可能的。这样的VarHandle实例可以用于向量化数组访问。

访问模式方法的参数数量，参数类型和返回类型由变量类型，变量类型和访问模式的特性决定。 VarHandle创建方法（如前所述）将记录要求。例如，在先前查找的VH_FOO_FIELD_I句柄中的compareAndSet需要3个参数，一个是接收器Foo的实例，另外两个int用于预期值和实际值：

>Foo f = ...
boolean r = VH_FOO_FIELD_I.compareAndSet（f，0，1）;

相反，getAndSet需要2个参数，一个是Foo的实例，一个int是要设置的值：

>nt o =（int）VH_FOO_FIELD_I.getAndSet（f，2）;

访问数组元素将需要在接收器和值参数（如果有的话）之间的类型为int的附加参数，这些参数对应于要操作的元素的数组索引。

对于运行时的可预测行为和性能，VarHandle实例应该保留在静态final字段中（根据Atomic {Int，Long，Reference} FieldUpdater的实例的要求）。这样可以确保访问模式方法调用会发生常量折叠，例如折叠方法签名检查和/或参数转换检查。

注意：未来的HotSpot增强功能可能支持VarHandle或MethodHandle的常量折叠，这些实例保存在非静态final字段，方法参数或局部变量中。

可以通过使用MethodHandles.Lookup.findVirtual为VarHandle访问模式方法生成MethodHandle。例如，要为特定变量类型的“compareAndSet”访问模式生成一个MethodHandle，并键入：

>Foo f = ...
MethodHandle mhToVhCompareAndSet = MethodHandles.publicLookup（）。findVirtual（
        VarHandle.class，
        “compareAndSet”
        MethodType.methodType（boolean.class，Foo.class，int.class，int.class））;

然后可以使用变量类型调用MethodHandle，并将兼容的VarHandle实例作为第一个参数：

>boolean r =（boolean）mhToVhCompareAndSet.invokeExact（VH_FOO_FIELD_I，f，0,1）;

或者mhToVhCompareAndSet可以绑定到VarHandle实例，然后调用：

>MethodHandle mhToBoundVhCompareAndSet = mhToVhCompareAndSet
        .bindTo（VH_FOO_FIELD_I）;
boolean r =（boolean）mhToBoundVhCompareAndSet.invokeExact（f，0，1）;

这样使用findVirtual的MethodHandle查找将执行一个asType转换来调整参数和返回值。该行为相当于使用MethodHandles.varHandleInvoker（MethodHandles.invoker`的类比）生成的MethodHandle：

>MethodHandle mhToVhCompareAndSet = MethodHandles.varHandleExactInvoker（
        VarHandle.AccessMode.COMPARE_AND_SET，
        MethodType.methodType（boolean.class，Foo.class，int.class，int.class））;

>boolean r =（boolean）mhToVhCompareAndSet.invokeExact（VH_FOO_FIELD_I，f，0,1）;

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
> /**
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
   
一个完整的栅栏比一个比负载栅栏更强的获得围栏更坚固（在订购保证方面）。 同样，一个完整的栅栏比一个比商店门槛更强的释放栅栏更强大。

##### 可达性围栏
可达性栅栏定义为java.lang.ref.Reference上的静态方法：
>class java.lang.ref.Reference {
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

目前，提供一个注释（@Finalized说）要在一个方法上声明的注释是超出范围的，在一个方法中，编译或运行时会导致方法体被包装如下：
>try {
    <method body>
} finally {
    Reference.reachabilityFence(this);
}

预计这种功能可以被编译时注释处理器支持。
##### 备择方案
引入新形式的“价值类型”被认为支持不稳定的行动。但是，这与其他类型的属性不一致，并且还需要更多的程序员的使用。还依赖于java.util.concurrent.atomic FieldUpdaters，但是它们的动态开销和使用限制使它们不合适。

在讨论这些问题的多年以来，其他几种替代方案，包括基于实地参考的替代方案，已经被提出和被驳回，因为句法，效率和/或可用性的原因是不可行的。

在此JEP的以前版本中考虑了语法增强功能，但被认为是“神奇的”，其中使用volatile关键字范围到浮动接口，一个用于引用，一个用于每个支持的原语类型。

在该JEP的以前版本中考虑了从VarHandle扩展的通用类型，但是对于通用类型和盒式类型变量的特殊处理，增加的多态性签名被认为是未成熟的，因为未来的Java版本具有价值类型和泛型， JEP 218，以及使用Arrays 2.0改进的阵列。

此JEP的以前版本中还考虑了实现特定的调用动力学方法。这要求使用和不使用invokedynamic的编译方法调用被仔细对齐，在语义方面是相同的。此外，在核心类中使用invokedynamic（例如ConcurrentHashMap）将导致循环依赖。

#####测试

将使用jcstress线束开发压力测试。

#####风险与假设

VarHandle的原型实现已经通过纳基准测试和叉/连接基准测试，其中fork / join库使用sun.misc.Unsafe被VarHandle替代。目前还没有观察到主要的性能问题，并且识别出的HotSpot编译器问题似乎并不繁重（折叠检查和改进阵列边界检查）。因此，我们有信心这种方法的可行性。然而，我们期望这将需要更多的实验来确保编译技术在最常需要这些构造的性能关键上下文中是可靠的。

#####依赖性

java.util.concurrent（以及JDK中标识的其他区域）中的类将从sun.misc.Unsafe迁移到VarHandle。

此JEP不依赖于JEP 188：Java内存模型更新。

####JEP 197: Segmented Code Cache
#####概要

将代码缓存分为不同的段，每个段都包含特定类型的编译代码，以提高性能并启用未来的扩展。

#####目标

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
#####非目标

分段代码缓存仅为未来的扩展提供基础，例如细粒度锁定;它还没有实现任何这些改进。

#####成功指标

分离不同的代码类型
较短的扫描时间
执行时间缩短
高度优化代码的碎片减少
减少iTLB和iCache未命中的数量
#####动机

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
#####备择方案

替代实现将定义优选地分配不同代码类型的逻辑存储器区域。如果有可用空间，我们将分配到首选的内存区域，如果没有可用空间，我们将分配给别的地方。

#####测试

使用JPRT，Nashorn + Octane，SPECjbb2013，SPECjbb2005，SPECjvm2008的强化正确性测试。

我们需要确保没有性能降低，特别是对于具有小代码缓存大小的嵌入式使用。

测试受影响的组件，包括可维护性代理，DTrace，Pstack，Java Flight Recorder。

#####风险与假设

如果一个代码堆已满，并且在另一个代码堆中仍然有空间，每个代码堆的固定大小会导致内存的潜在浪费。特别是对于非常小的代码高速缓存大小，即使还有空间可用，也可能会关闭编译器。为了解决这个问题，将添加一个选项来关闭小代码缓存大小的分段。

非方法代码的大小取决于Java应用程序，底层平台和JVM设置。因此，在JVM启动时，很难确定非方法代码堆中所需的空间。

该补丁的未来版本可以实现动态调整大小（由扫描器支持）或不同的分配策略来降低浪费内存的风险。
#### JEP 199: Smart Java Compilation, Phase Two
#####概要

改进sjavac工具，使其可以在JDK构建中默认使用，并将其概括为可以用于构建JDK以外的大型项目。

#####目标

由于与稳定性和可移植性有关的各种问题，sjavac在JDK构建脚本中默认不被使用。这个JEP的第一个目标是解决这些问题。这包括确保该工具始终在所有软件/硬件配置上产生可靠的结果。

总体目标是提高sjavac的质量，使之成为能够编译大型任意Java项目的通用javac包装。

一个后续的项目将探讨如何在JDK工具链中暴露sjavac;这可能是单独支持的独立工具，不支持的独立工具，与javac集成或其他内容。

#####非目标

这个JEP的目标不是引进今天没有的新功能，而不是我们自己的构建脚本所要求的，并且不会为通用的javac包装器做出贡献。意图是不要重写包装器。目前功能很好的功能将保持原样。

如果需要，将sjavac纳入产品将成为单独JEP的主题。

#####动机

目前的实现被证明是有用的，确实提高了构建速度并允许增量版本。然而，整个工具的守则质量和稳定性并不令人满意，当然没有准备公开发行。

#####描述

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

#####测试

除了通常的强制性单元测试之外，sjavac将在所有平台上进行彻底的测试，因为这些工具的一部分涉及特定于平台的方面，例如文件路径和后台进程的产生。

此外，由于服务器 - 客户端架构本质上是并行的，所以sjavac将会考虑到并发性的压力测试。

#####依赖性

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
