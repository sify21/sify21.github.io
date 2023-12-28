---
title: "rust程序内存泄漏问题定位"
categories: 
  - Rust
tags:
  - memory leak
  - memory profile
  - heap profile
  - malloc
---
一个rust项目出现内存占用不断增大的问题，怀疑代码中有内存泄漏。

memory leak只会发生在heap allocation过程中。stack上的local variable是会随着stack销毁的，如果stack上留存只会是stack overflow。

jvm有heapdump。c/c++最好使用一个allocator以获得stacktrace。

目前主流的memory allocator有jemalloc(facebook维护，firefox大量使用), tcmalloc(google维护)，mimalloc(微软新出的，据说提升很大)。

这里使用了tikv维护的一个crate: jemallocator，rustc之前默认用的就是这个，后来剔除了，让用户可以自己设global allocator。

用heap profile的功能需要开启feature profiling(jemalloc的opt.prof需要--enable-prof)。我配成了只为linux target使用（cross编译windows和apple报错，可能要调整构建容器）。

```toml
[target.'cfg(target_os = "linux")'.dependencies]
tikv-jemallocator = "0.5"

[features]
profiling = ["tikv-jemallocator/profiling"] # 或者可以不加这个，在构建的时候直接指定 --features tikv-jemallocator/profiling
```

rust目前还没有target specific feature，但既然dependency已经是target specific的了，所以如果没有这个dependency开这个feature也没啥影响。

注意在rust项目构建完成后，会在对应target目录的build/tikv-jemalloc-sys-??/out/build/bin目录下包含对应jeprof脚本程序。最好用这个脚本。自己从jemalloc源码构建的可能跟rust构建用的版本不一样。

### jemalloc配置
jemalloc配置文档：http://jemalloc.net/jemalloc.3.html
> ### TUNING
> 
> Once, when the first call is made to one of the memory allocation routines, the allocator initializes its internals based in part on various options that can be specified at compile- or run-time.
> 
> The string specified via --with-malloc-conf, the string pointed to by the global variable malloc_conf, the “name” of the file referenced by the symbolic link named /etc/malloc.conf, and the value of the environment variable MALLOC_CONF, will be interpreted, in that order, from left to right as options. Note that malloc_conf may be read before main() is entered, so the declaration of malloc_conf should specify an initializer that contains the final value to be read by jemalloc. --with-malloc-conf and malloc_conf are compile-time mechanisms, whereas /etc/malloc.conf and MALLOC_CONF can be safely set any time prior to program invocation.
> 
> An options string is a comma-separated list of option:value pairs. There is one key corresponding to each opt.* mallctl (see the MALLCTL NAMESPACE section for options documentation). For example, abort:true,narenas:1 sets the opt.abort and opt.narenas options. Some options have boolean values (true/false), others have integer values (base 8, 10, or 16, depending on prefix), and yet others have raw string values.

注意在`MALLCTL NAMESPACE`中的rw, 指的是运行时调用mallctl*()是否可读写。`TUNING`中的opt.*是启动时的配置，这些基本都是r-, 这些参数在程序启动时可以用来设置初始值，在运行阶段就不能通过mallctl*()修改了。

### tikv-jemallocator配置
- feature: profiling 和 stats 分别对应构建文档:https://github.com/jemalloc/jemalloc/blob/dev/INSTALL.md 中的--enable-prof和--disable-stats

- unprefixed_malloc_on_supported_platforms默认不开启，相当于--with-jemalloc-prefix=_rjem_

### jemalloc的heap dump文件格式
```
heap_v2/524288
  t*: 28106: 56637512 [0: 0]
  [...]
  t3: 352: 16777344 [0: 0]
  [...]
  t99: 17754: 29341640 [0: 0]
  [...]
@ 0x5f86da8 0x5f5a1dc [...] 0x29e4d4e 0xa200316 0xabb2988 [...]
  t*: 13: 6688 [0: 0]
  t3: 12: 6496 [0: 0]
  t99: 1: 192 [0: 0]
[...]

MAPPED_LIBRARIES:
[...]

The following matches the above heap profile, but most tokens are replaced with <description> to indicate descriptions of the corresponding fields.

<heap_profile_format_version>/<mean_sample_interval>
  <aggregate>: <curobjs>: <curbytes> [<cumobjs>: <cumbytes>]
  [...]
  <thread_3_aggregate>: <curobjs>: <curbytes> [<cumobjs>: <cumbytes>]
  [...]
  <thread_99_aggregate>: <curobjs>: <curbytes> [<cumobjs>: <cumbytes>]
  [...]
@ <top_frame> <frame> [...] <frame> <frame> <frame> [...]
  <backtrace_aggregate>: <curobjs>: <curbytes> [<cumobjs>: <cumbytes>]
  <backtrace_thread_3>: <curobjs>: <curbytes> [<cumobjs>: <cumbytes>]
  <backtrace_thread_99>: <curobjs>: <curbytes> [<cumobjs>: <cumbytes>]
[...]

MAPPED_LIBRARIES:
</proc/<pid>/maps>
```
objects可能是指calloc: Allocates memory for an array of **num** objects of **size** and initializes all bytes in the allocated storage to zero. 最终分配的bytes= **num** * **size**

对应的malloc可能就看成了object count为1。

文件最后的MAPPED_LIBRARIES包含可执行文件的绝对路径。所以jeprof工具应当在生成dump文件的机器上本地跑。如果把dump文件拿到本地来分析，需要手动需改下文件路径到本地的可执行文件路径。

Jason Evans本人对数据的解释（http://jemalloc.net/mailman/jemalloc-discuss/2015-November/001205.html）
> The two types of stats are:
> - Current bytes/objects, aka "inuse" in pprof/jeprof terminology.  These are counts of how many sampled objects currently exist.  Use these stats to understand current memory usage.
> - Cumulative bytes/objects, aka "alloc" in pprof/jeprof terminology.  These are counts of how many sampled bytes/objects have existed since the most recent stats reset (process start or "prof.reset" mallctl call).  Use these stats to understand total allocation volume.
> 
> dumps are always based on the most recent stats reset (process start or "prof.reset" mallctl call).  You can view incremental differences between two dumps by using the --base option to jeprof
>
> opt.lg_prof_interval is merely a dump triggering mechanism.  opt.prof_accum controls whether cumulative stats are collected at all.
> 
> Take the following function as an example, run with MALLOC_CONF=prof:true,prof_accum:true :
> ``` 
> void g(void *p);
> 
> void f(void) {
>     unsigned i;
>
>     for (i = 0; i < (1U << 20); i++) {
>         void *p = malloc(1U << 30);
>         if (i == (1U << 19)) {
>             mallctl("prof.dump", NULL, NULL, NULL, 0); /* A */
>             mallctl("prof.reset", NULL, NULL, NULL, 0);
>             mallctl("prof.dump", NULL, NULL, NULL, 0); /* B */
>         }
>         if (p != NULL) {
>             g(p);
>             free(p);
>         }
>     }
>     mallctl("prof.dump", NULL, NULL, NULL, 0); /* C */
> }
> ```
> What will the heap profiling stats (as interpreted by jeprof) dumped at A, B, and C say regarding the malloc() site in f()?
> 
> A:
>   - Current: ~1 object, ~2^30 bytes
>   - Cumulative: ~2^19 objects, ~2^49 bytes
> 
> B:
>   - Current: 0 objects, 0 bytes
>   - Cumulative: 0 objects, 0 bytes
>
> C:
>   - Current: 0 objects, 0 bytes
>   - Cumulative: ~2^19 objects, ~2^49 bytes
> 
> opt.prof_accum controls whether jemalloc maintains the cumulative stats.  With MALLOC_CONF=prof:true,prof_accum:false, you will get no cumulative stats at all, no matter when/whether any resets occurred.

也就是说free会将current数据减少，cumulative数据会一直增加。reset是所有数据的计数起点。

注意这里的cumulative stats，和jeprof的top表头中的flat/cum是两个概念。flat/cum意思是: 纯自己的 vs 包含内部调用的其他函数的。

### jeprof工具使用（flamegraph）
> Note that each of the .heap files are full snapshots instead of increments. Hence, simply pick the latest file (or any historical snapshot).
> 
> jeprof is a utility provided by jemalloc to analyze heap dump files. It reads both the executable binary and the heap dump to get a full heap profiling.
> 
> Note that the heap profiler dump file must be analyzed along with exactly the same binary that it generated from.

jeprof --collapsed binary_file heap_file > heap_file.collapsed

https://github.com/brendangregg/FlameGraph

可以直接丢到这个网页上看https://www.speedscope.app/, 或者

./flamegraph.pl --color=mem --countname=bytes heap_file.collapsed > flamegraph.svg

### memory profile原理
A heap profiler associates memory allocations with the callstacks on which they happen.It is prohibitively expensive to handle every allocation done by a program, so the Android Heap Profiler employs a sampling approach that handles a statistically meaningful subset of allocations.
#### cpu profile sample
#### 程序运行时如何获取stacktrace
在大多数编程语言中，可执行程序获取调用堆栈（call stacktrace）通常通过特定的语言特性或库函数来实现。不同的编程语言可能有不同的方法来获取调用堆栈，但通常都可以通过以下方式之一：

    语言特性或内置函数： 很多编程语言提供了直接获取调用堆栈的内置函数或语言特性。例如，在Java中，可以使用Thread.currentThread().getStackTrace()方法来获取堆栈跟踪信息。

    调试器和异常处理器： 调试器通常能够提供堆栈跟踪信息。当程序抛出异常时，异常处理器通常会记录堆栈跟踪以便于调试。通过捕获异常或利用调试器的功能，程序可以获取堆栈信息。

    第三方库或工具： 有些编程语言可能需要使用第三方库或工具来获取堆栈跟踪。例如，Python中的traceback模块可以用于获取堆栈信息。

调用堆栈信息通常包括当前执行的函数、调用该函数的函数，以及整个调用链的信息。这些信息对于调试和定位程序中的问题非常有用，能够帮助开发人员追踪程序的执行流程。

不同编程语言中获取调用堆栈的实现方式可能会有所不同，但通常的实现原理大致相似。以下是一般情况下的实现方式：

    保存调用信息： 当函数被调用时，程序会在内存中创建一个称为堆栈帧（stack frame）的数据结构，用于保存该函数的信息，如函数参数、局部变量以及函数执行完后应该返回的位置等。这些堆栈帧按照调用顺序依次堆叠在一起，形成调用堆栈。

    语言运行时或编译器支持： 许多编程语言的运行时环境或编译器会提供一些特殊的数据结构或机制，用于跟踪和管理调用堆栈。这些机制可能包括在堆栈帧中添加额外信息、记录函数调用和返回、管理异常处理等。

    内置函数或API： 语言提供了特定的函数或API，允许程序访问当前调用堆栈的信息。这些函数可以访问堆栈中的堆栈帧，并从中提取有关函数调用的信息，例如函数名、文件名、行号等。

    符号表和元数据： 有些语言会使用符号表或元数据来存储函数和变量的信息，包括函数名、文件名、行号等。调用堆栈信息的获取可能涉及到解析这些符号表或元数据，以获取更多有关函数调用的信息。

总的来说，调用堆栈信息的获取通常依赖于语言的运行时环境、编译器支持以及提供的特殊函数或API。这些机制的具体实现可能因编程语言而异，但都旨在提供对程序执行过程中函数调用链的跟踪和管理。

大多数通用的 CPU 指令集并没有直接提供获取当前调用堆栈的指令。调用堆栈的概念是高级编程语言和运行时环境层面的概念，而并非在 CPU 指令集级别的概念。

CPU 指令集主要负责执行基本的操作，例如算术运算、数据移动、逻辑操作等，它们与更高层次的抽象概念（比如函数调用、堆栈跟踪）通常是分离的。

然而，在特定的架构中，有些指令集可能提供了某种形式的支持，使得在硬件级别上可以获取部分堆栈信息。例如，有些 CPU 架构可能提供指令，可以读取当前栈指针（stack pointer）或帧指针（frame pointer），这些指针是用于管理堆栈的重要指示器。但是，这些信息通常并不足以直接获取完整的调用堆栈信息，例如函数名、文件名、行号等高级调试信息。

获取完整的调用堆栈信息通常需要在更高级别的抽象层（比如编程语言的运行时环境或调试器）中实现。这些抽象层会利用 CPU 提供的基本指令以及其他运行时环境的信息，来构建和管理完整的调用堆栈信息。

backtrace 模块在 Rust 中是一个提供调用堆栈信息的标准库。其内部的实现主要依赖于底层的系统功能，比如使用操作系统提供的原生功能或者平台相关的工具来获取调用堆栈信息。

在底层，backtrace 模块通常使用了平台相关的调试信息，比如在 Linux 上可能使用了 libunwind 库，而在 Windows 上可能使用了 DbgHelp。这些库提供了对程序运行时调用堆栈信息的访问。

一般来说，这些库可以让 Rust 或其他编程语言的程序获取当前线程的调用堆栈信息。具体的实现会依赖于操作系统和硬件平台，因为不同的系统和架构可能有不同的方式来处理调用堆栈的信息。

backtrace 模块提供了一个统一的接口来获取调用堆栈信息，而底层的实现则依赖于系统和平台相关的功能来实际获取这些信息。这种抽象允许 Rust 在不同的平台上获取调用堆栈信息而无需更改程序的代码。

libunwind 是一个用于访问程序运行时调用堆栈信息的库，它提供了对堆栈展开（stack unwinding）的支持，允许程序在运行时获取调用堆栈的信息。

主要功能包括：

    堆栈展开： 能够在程序运行时追踪当前线程的函数调用链，找出函数调用的顺序和位置。
    异常处理： 在 C++ 等语言中，它也用于异常处理，允许程序在异常发生时回溯到异常点，获取相关的调用堆栈信息。
    跨平台性： 尽管它的实现可能依赖于底层操作系统和硬件架构，但它为不同的操作系统（如 Linux、BSD、macOS）提供了一致的接口。

libunwind 提供了一组 C 接口，允许程序访问和操作调用堆栈信息。这对于调试器、性能分析器以及需要获取运行时调用堆栈信息的工具和库非常有用。

在 Rust 中，libunwind 也被用于实现一些与调用堆栈相关的功能，比如 backtrace 模块就可能在一些平台上依赖于 libunwind 来获取调用堆栈信息。

在 Linux 上，libunwind 通过操作系统提供的机制来获取调用堆栈信息。它使用了一系列系统调用和底层的机制来实现堆栈展开并获取函数调用链信息。

基本上，libunwind 在 Linux 上的工作原理包括以下几个方面：

    使用系统调用获取寄存器和堆栈信息： libunwind 通过访问线程的寄存器和堆栈来获取调用堆栈信息。这包括读取栈指针（Stack Pointer）、基址指针（Base Pointer）等寄存器的值，以及从堆栈中读取调用帧信息。

    解析函数调用帧： libunwind 遍历堆栈中的帧信息，解析出函数调用链的信息。它会识别帧中的返回地址，从而确定调用链中各个函数的位置和顺序。

    使用特定的异常处理机制： 在有些情况下，libunwind 还可以利用操作系统提供的异常处理机制来获取调用堆栈信息。当程序发生异常时，操作系统可能会保存相关的调用堆栈信息，libunwind 可以利用这些信息进行堆栈展开。

需要注意的是，libunwind 的实现依赖于操作系统提供的接口和机制，并且在不同的系统和架构上可能有不同的实现方式。因此，在 Linux 上，它的工作原理可能会与其他操作系统上的实现略有不同，但基本思路是相似的：通过访问寄存器和堆栈来追踪和解析函数调用链。

libunwind 识别帧中的返回地址是通过解析调用堆栈中的帧信息来实现的。在大多数架构上，函数调用时，程序会在堆栈中创建帧（stack frame）来存储有关函数调用的信息，其中包括返回地址。

返回地址是指函数执行完毕后程序应该继续执行的地址，通常是调用该函数的指令地址加上指令长度。这个返回地址通常是存储在帧中的一个特定位置，不同的架构可能有不同的存储方式。

具体来说，在一些常见的架构中：

    x86 架构： 返回地址通常存储在栈顶，即当前函数的栈帧中的一个位置。在函数调用时，返回地址被压入栈中。libunwind 可以读取当前栈帧的栈顶，获取存储的返回地址。

    x86-64（64位）架构： 与x86类似，返回地址也通常存储在栈帧中的特定位置。同样，libunwind 在64位系统上也可以读取栈顶位置，获取返回地址。

    其他架构： 不同的架构可能有不同的堆栈结构和帧信息存储方式。libunwind 针对特定架构会使用相应的机制来读取帧信息，包括返回地址。

通过解析帧信息，libunwind 能够识别帧中存储的返回地址，从而追踪函数调用链的顺序和位置。这些返回地址告诉程序在函数执行完毕后应该回到哪里继续执行。
#### memory profile sample
 exponential distribution

> // nextSample returns the next sampling point for heap profiling. The goal is
> // to sample allocations on average every MemProfileRate bytes, but with a
> // completely random distribution over the allocation timeline; this
> // corresponds to a Poisson process with parameter MemProfileRate. In Poisson
> // processes, the distance between two samples follows the exponential
> // distribution (exp(MemProfileRate)), so the best return value is a random
> // number taken from an exponential distribution whose mean is MemProfileRate.
> func nextSample() uintptr

In many cases, memory allocation is regular. If sampling is performed at a fixed granularity, the final result may have a big error. It may happen that a particular type of memory allocation exists at each sampling. This is why randomization is chosen here.

### tokio console
