# 基准实用程序 
- torch.utils.benchmark [¶](#module-torch.utils.benchmark "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/benchmark_utils>
>
> 原始地址：<https://pytorch.org/docs/stable/benchmark_utils.html>


*班级*


 torch.utils.benchmark。



 Timer
 


 ( *stmt='pass'
* 、 *setup='pass'
* 、 *global_setup=''
* 、 *timer=<内置函数 perf_counter>
* 、 *globals=None
* 、 *label=None 
* , *sub_label=None
* , *description=None
* , *env=None
* , *num_threads=1
* , *language=Language.PYTHON
* ) [[source]](_modules/torch/utils/benchmark/utils/timer.html#Timer)[¶](#torch.utils.benchmark.Timer "此定义的永久链接")


 用于测量 PyTorch 语句执行时间的帮助程序类。


 有关如何使用此类的完整教程，请参阅：<https://pytorch.org/tutorials/recipes/recipes/benchmark.html>


 PyTorch 计时器基于 timeit.Timer (实际上在内部使用 timeit.Timer)，但有几个关键区别：


1.运行时感知：


 计时器将执行预热(很重要，因为 PyTorch 的某些元素是延迟初始化的)，设置线程池大小以便进行同类比较，并在必要时同步异步 CUDA 函数。2。专注于重复：


 当测量代码，特别是复杂的内核/模型时，运行之间的变化是一个重要的混杂因素。预计所有测量都应包括重复项以量化噪声并允许中值计算，这比平均值更稳健。为此，此类通过概念上合并 timeit.Timer.repeat 和 timeit.Timer.autorange 来偏离 timeit API。(精确算法在方法文档字符串中进行了讨论。)在不需要自适应策略的情况下，将复制 timeit 方法。3。可选元数据：


 定义 Timer 时，可以选择指定 label 、 sub_label 、 description 和 env 。 (稍后定义)这些字段包含在结果对象的表示中，并由Compare 类对结果进行分组并显示比较。4．指令计数


 除了挂墙时间外，Timer 还可以运行 Callgrind 下的语句并报告所执行的指令。


 直接类似于 timeit.Timer 构造函数参数：



> 
> 
> 
> 
> stmt
> 
> ,
> 
> 设置
> 
> ,
> 
> 定时器
> 
> ,
> 
> 全局变量
> 
> 
> 
> 
> >


 PyTorch 定时器特定的构造函数参数：



> 
> 
> 
> 
> 标签
> 
> ,
> 
> sub_label
> 
> ,
> 
> 描述
> 
> ,
> 
> env
> 
> ,
> 
> num_threads
> 
> 
> 
> 
> >


 参数 
* **stmt** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要运行的代码片段循环并定时。
* **setup** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) –可选设置代码。用于定义 stmt
* **global_setup** 中使用的变量 ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)" ) ) – (仅限 C++)放置在文件顶层的代码，用于 #include 语句等内容。
* **timer** ( [*Callable*](https://docs.python.org/3/library/typing.html#typing.Callable "(在 Python v3.12 中)")*[
* *[
* *]
* *,
* [*float*](https://docs.python.org/3/library /functions.html#float "(Python v3.12)")*]
* ) – 可调用，返回当前时间。如果 PyTorch 是在没有 CUDA 的情况下构建的或者没有 GPU 存在，则默认为 timeit.default_timer ；否则它将在测量时间之前同步 CUDA。
* **全局** ( [*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中) )")*[
* [*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(在 Python v3.12 中)")*[
* [*str
* ](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(in Python v3.12)")*]
* *]
* ) – 一个字典，定义执行 stmt 时的全局变量。这是提供 stmt 需要的变量的另一种方法。
* **label** ( [*Optional*](https://docs.python.org/3/library/typing.html#typing.Optional "(in Python v3.12)")*[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*]
* ) – 字符串其中总结了 stmt 。例如，如果 stmt 是“torch.nn.function.relu(torch.add(x, 1, out=out))”，则可以将标签设置为“ReLU(x 
+ 1)”以提高可读性。
* **sub _label** ( [*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中)")*[
* [*str*] (https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*]
* ) –


 提供补充信息，以消除具有相同 stmt 或标签的测量结果的歧义。例如，在我们上面的例子中，sub_label可能是“float”或“int”，这样很容易区分：“ReLU(x 
+ 1): (float)”


 “ReLU(x 
+ 1): (int)”打印测量值或使用比较进行汇总时。
* **描述** ( [*可选*](https://docs.python.org/3/library/typing.html #typing.Optional "(Python v3.12 中)")*[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12 中) “)*]
* ) –


 用于区分具有相同标签和子标签的测量的字符串。描述的主要用途是发出信号以比较数据列。例如，可以根据输入大小设置它以创建以下形式的表格：


```
                        | n=1 | n=4 | ...
                        ------------- ...
ReLU(x + 1): (float)    | ... | ... | ...
ReLU(x + 1): (int)      | ... | ... | ...

```


 使用比较。打印测量时也会包含它。
* **env** ( [*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中)")*[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* ) – 此标签表示否则，相同的任务在不同的环境中运行，因此不等效，例如在对内核更改进行 A/B 测试时。合并复制运行时，比较将把具有不同环境规范的测量视为不同的。
* **num_threads** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 执行 stmt 时 PyTorch 线程池的大小。单线程性能非常重要，因为它既是关键的推理工作负载，又是内在算法效率的良好指标，因此默认设置为 1。这与尝试利用所有核心的默认 PyTorch 线程池大小形成对比。


 被阻止_autorange


 ( *打回来



 =
 


 无
* , *min_run_time



 =
 


 0.2
* ) [[source]](_modules/torch/utils/benchmark/utils/timer.html#Timer.blocked_autorange)[¶](#torch.utils.benchmark.Timer.blocked_autorange "此定义的永久链接")


 测量多次重复，同时将计时器开销保持在最低限度。


 在较高级别上，blocked_autorange 执行以下伪代码：


```
`setup`

total_time = 0
while total_time < min_run_time
    start = timer()
    for _ in range(block_size):
        `stmt`
    total_time += (timer() - start)

```


 注意内部循环中的变量 block_size 。块大小的选择对于测量质量很重要，并且必须平衡两个相互竞争的目标：



> 
> 
> 1. 小块大小会导致更多重复，并且通常
> 更好的统计数据。
> 2. 大块大小可以更好地分摊>> 定时器>> 调用的成本，并导致偏差较小的测量。这一点很重要，因为 CUDA 同步时间非常重要(从一位数微秒到低两位数微秒)，否则会使测量产生偏差。
> 
> 
> >


 block_autorange 通过运行预热期来设置块大小，增加块大小，直到计时器开销小于总体计算的 0.1%。然后将该值用于主测量循环。


 退货


 包含测量的运行时间和重复计数的测量对象，可用于计算统计数据。(平均值、中位数等)


 Return type


[*测量*](#torch.utils.benchmark.Measurement "torch.utils.benchmark.utils.common.Measurement")


 收集_callgrind


 ( *数字



 :
 


[int](https://docs.python.org/3/library/functions.html#int "(Python v3.12)")
* , *** , *重复



 :
 


[无](https://docs.python.org/3/library/constants.html#None "(在 Python v3.12 中)")
* , *collect_baseline



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")
* , *retain_out_file



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)*)


 → [CallgrindStats](#torch.utils.benchmark.CallgrindStats "torch.utils.benchmark.utils.valgrind_wrapper.timer_interface.CallgrindStats")


[[source]](_modules/torch/utils/benchmark/utils/timer.html#Timer.collect_callgrind)[¶](#torch.utils.benchmark.Timer.collect_callgrind"此定义的永久链接")


 收集_callgrind


 ( *数字



 :
 


[int](https://docs.python.org/3/library/functions.html#int "(Python v3.12)")
* , *** , *重复



 :
 


[int](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)")
* , *collect_baseline



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")
* , *retain_out_file



 :
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)*)


 → [元组](https://docs.python.org/3/library/typing.html#typing.Tuple“(在Python v3.12中)”)


 [ [CallgrindStats](#torch.utils.benchmark.CallgrindStats "torch.utils.benchmark.utils.valgrind_wrapper.timer_interface.CallgrindStats")


 ,
 


...
 



 ]
 


 使用 Callgrind 收集指令计数。


 与挂墙时间不同，指令计数是确定性的(程序本身的模非确定性和来自 Python 解释器的少量抖动)。这使它们成为详细性能分析的理想选择。此方法在单独的进程中运行 stmt，以便 Valgrind 可以检测程序。由于仪器的使用，性能严重下降，但是，少量迭代通常足以获得良好的测量结果，这一点得到了改善。


 为了使用此方法，必须安装 valgrind 、 callgrind_control 和 callgrind_annotate 。


 由于调用者(此进程)和 stmt 执行之间存在进程边界，因此全局变量不能包含任意内存中数据结构。 (与计时方法不同)相反，全局变量仅限于内置函数、nn.Modules 和 TorchScripted 函数/模块，以减少序列化和后续反序列化带来的意外因素。 GlobalsBridge 类提供了有关此主题的更多详细信息。请特别注意 nn.Modules：它们依赖于 pickle，您可能需要在设置中添加导入才能正确传输。


 默认情况下，将收集并缓存空语句的配置文件，以指示有多少指令来自驱动 stmt 的 Python 循环。


 退货


 CallgrindStats 对象提供指令计数和一些用于分析和操作结果的基本工具。


 泰美


 ( *数字



 =
 


 1000000
* ) [[source]](_modules/torch/utils/benchmark/utils/timer.html#Timer.timeit)[¶](#torch.utils.benchmark.Timer.timeit "此定义的永久链接")


 镜像 timeit.Timer.timeit() 的语义。


 执行主语句 ( stmt ) 多次。 <https://docs.python.org/3/library/timeit.html#timeit.Timer.timeit>


 Return type


[*测量*](#torch.utils.benchmark.Measurement "torch.utils.benchmark.utils.common.Measurement")


*班级*


 torch.utils.benchmark。


 测量


 ( *number_per_run
* 、*raw_times
* 、*task_spec
* 、*元数据



 =
 


 无
* ) [[source]](_modules/torch/utils/benchmark/utils/common.html#Measurement)[¶](#torch.utils.benchmark.Measurement "此定义的永久链接")


 定时器测量的结果。


 此类存储给定语句的一个或多个测量值。它是可序列化的，并为下游消费者提供了几种方便的方法(包括详细的__repr__)。


*静止的*


 merge
 


 ( *measurements
* ) [[source]](_modules/torch/utils/benchmark/utils/common.html#Measurement.merge)[¶](#torch.utils.benchmark.Measurement.merge "此定义的永久链接")


 合并重复的便捷方法。


 合并将推断次数为 number_per_run=1 并且不会传输任何元数据。 (因为重复之间可能有所不同)


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*Measurement*](#torch.utils.benchmark.测量“torch.utils.benchmark.utils.common.Measurement”)]


*财产*


 重要数字*：


[int](https://docs.python.org/3/library/functions.html#int"(在Python v3.12中)")*[¶](#torch.utils.benchmark.Measurement.significant_figures"永久链接到这个定义")


 近似有效数字估计。


 此属性旨在提供一种估计测量精度的便捷方法。它仅使用四分位间区域来估计统计数据，以尝试减轻尾部的倾斜，并使用静态 z 值 1.645，因为预计不会将其用于较小的 n 值，因此 z 可以近似 t 。


 有效数字估计与trim_sigfig方法结合使用，以提供更人性化的可解释数据摘要。 __repr__ 不使用此方法；它只是显示原始值。有效数字估计用于 Compare 。


*班级*


 torch.utils.benchmark。


 呼叫研磨统计


 ( *task_spec
* , *number_per_run
* , *built_with_debug_symbols
* , *baseline_inclusive_stats
* , *baseline_exclusive_stats
* , *stmt_inclusive_stats
* , 
* stmt_exclusive_stats
* , *stmt_callgrind_out
* ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#CallgrindStats)[¶](#torch.utils.benchmark. CallgrindStats"此定义的永久链接")


 Timer 收集的 Callgrind 结果的顶级容器。


 操作通常使用 FunctionCounts 类完成，该类是通过调用 CallgrindStats.stats(...) 获得的。还提供了几种方便的方法；最重要的是 CallgrindStats.as_standardized() 。


 作为_标准化


 ( ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#CallgrindStats.as_standardized)[¶](#torch.utils.benchmark.CallgrindStats.as_standardized "此定义的永久链接")


 从函数字符串中去除库名称和一些前缀。


 当比较两组不同的指令计数时，绊脚石可能是路径前缀。 Callgrind 在报告函数时包含完整的文件路径(因为它应该)。但是，这可能会在比较配置文件时导致问题。如果像 Python 或 PyTorch 这样的关键组件是在两个配置文件中的不同位置构建的，则可能会导致类似于以下内容的结果：


```
23234231 /tmp/first_build_dir/thing.c:foo(...)
 9823794 /tmp/first_build_dir/thing.c:bar(...)
  ...
   53453 .../aten/src/Aten/...:function_that_actually_changed(...)
  ...
 -9823794 /tmp/second_build_dir/thing.c:bar(...)
-23234231 /tmp/second_build_dir/thing.c:foo(...)

```


 剥离前缀可以通过规范字符串并在比较时更好地取消等效调用站点来改善此问题。


 Return type


[*CallgrindStats*](#torch.utils.benchmark.CallgrindStats "torch.utils.benchmark.utils.valgrind_wrapper.timer_interface.CallgrindStats")


 计数


 ( *** , *去噪



 =
 


 False
* ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#CallgrindStats.counts)[¶](#torch.utils.benchmark.CallgrindStats.counts "此定义的永久链接")


 返回执行的指令总数。


 有关降噪参数的说明，请参阅 FunctionCounts.denoise()。


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 delta
 


 ( *其他
* , *包括



 =
 


 False
* ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#CallgrindStats.delta)[¶](#torch.utils.benchmark.CallgrindStats.delta "此定义的永久链接")


 比较两组计数。


 收集指令计数的一个常见原因是确定特定更改对执行某些工作单元所需的指令数量的影响。如果变化增加了这个数字，下一个逻辑问题就是“为什么”。这通常涉及查看代码的指令数增加的部分。该函数使该过程自动化，以便人们可以轻松地在包含和排除的基础上比较计数。


 Return type


[*FunctionCounts*](#torch.utils.benchmark.FunctionCounts "torch.utils.benchmark.utils.valgrind_wrapper.timer_interface.FunctionCounts")


 stats
 


 ( *包括的



 =
 


 False
* ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#CallgrindStats.stats)[¶](#torch.utils.benchmark.CallgrindStats.stats "此定义的永久链接")


 返回详细的函数计数。


 从概念上讲，返回的 FunctionCounts 可以被认为是一个 tupleof (count, path_and_function_name) 元组。


 Include 与 callgrind 的语义相匹配。如果为 True，则计数包括子级执行的指令。 Include=True 对于识别代码中的热点很有用； Include=False 对于在比较两次不同运行的计数时减少噪音很有用。 (有关更多详细信息，请参阅CallgrindStats.delta(...))


 Return type


[*FunctionCounts*](#torch.utils.benchmark.FunctionCounts "torch.utils.benchmark.utils.valgrind_wrapper.timer_interface.FunctionCounts")


*班级*


 torch.utils.benchmark。


 函数计数


 ( *_data
* , *包含
* , *截断_行



 =
 


 真
* , *_linewidth



 =
 


 无
* ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#FunctionCounts)[¶](#torch.utils.benchmark.FunctionCounts "此定义的永久链接")


 用于操作 Callgrind 结果的容器。


 它支持： 1. 加法和减法来组合或比较结果。2.类似元组的索引3。一个降噪函数，可以去除已知的非确定性且噪音很大的 CPython 调用。4。用于自定义操作的两种高阶方法(过滤器和变换)。


 去噪


 ( ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#FunctionCounts.denoise)[¶](#torch.utils.benchmark.FunctionCounts.denoise "此定义的永久链接")


 删除已知的嘈杂指令。


 CPython 解释器中的一些指令相当嘈杂。这些指令涉及 unicode 到字典的查找，Python 使用它来映射变量名称。 FunctionCounts 通常是一个与内容无关的容器，但这对于获得可靠的结果非常重要，足以保证例外。


 Return type


[*FunctionCounts*](#torch.utils.benchmark.FunctionCounts "torch.utils.benchmark.utils.valgrind_wrapper.timer_interface.FunctionCounts")


 筛选


 ( *filter_fn
* ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#FunctionCounts.filter)[¶](#torch.utils.benchmark.FunctionCounts.filter "永久链接这个定义")


 仅保留将 filter_fn 应用于函数名称返回 True 的元素。


 Return type


[*FunctionCounts*](#torch.utils.benchmark.FunctionCounts "torch.utils.benchmark.utils.valgrind_wrapper.timer_interface.FunctionCounts")


 转换


 ( *map_fn
* ) [[source]](_modules/torch/utils/benchmark/utils/valgrind_wrapper/timer_interface.html#FunctionCounts.transform)[¶](#torch.utils.benchmark.FunctionCounts.transform "永久链接这个定义")


 将 map_fn 应用于所有函数名称。


 这可用于规范函数名称(例如，剥离文件路径的不相关部分)、通过将多个函数映射到相同名称来合并条目(在这种情况下，计数会加在一起)等。


 Return type


[*FunctionCounts*](#torch.utils.benchmark.FunctionCounts "torch.utils.benchmark.utils.valgrind_wrapper.timer_interface.FunctionCounts")