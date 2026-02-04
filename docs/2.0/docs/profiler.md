没有10



 单击
 [此处](#sphx-glr-download-beginner-profiler-py)
 下载完整的示例代码





# 分析您的 PyTorch 模块
 [¶](#profiling-your-pytorch-module "永久链接到此标题")


> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/profiler>
>
> 原始地址：<https://pytorch.org/docs/stable/profiler.html>



**作者：** 
[Suraj Subramanian](https://github.com/suraj813)


PyTorch 包含一个探查器 API，可用于识别代码中各种 PyTorch 操作的时间和内存成本。探查器可以\轻松集成到您的代码中，并且结果可以作为表格打印\或在 JSON 跟踪文件中返回。





 注意




 Profiler 支持多线程模型。探查器在与操作相同的线程中运行，但它也会分析可能在另一个线程中运行的子运算符。并发运行的探查器将被
限定在它们自己的线程范围内，以防止结果混合。



 没有10



 PyTorch 1.8 引入了新的 API，它将在未来的版本中取代旧的探查器 API。请在
 [此页面](https://pytorch.org/docs/master/profiler.html) 检查新 API 
 。





 请前往
 [此食谱](https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html)
 以更快地了解 Profiler API 的使用情况。





---


```
import torch
import numpy as np
from torch import nn
import torch.autograd.profiler as profiler

```


## 使用 Profiler 进行性能调试
 [¶](#performance-debugging-using-profiler "永久链接到此标题")




 探查器可用于识别
模型中的性能瓶颈。在此示例中，我们构建一个执行两个
子任务的自定义模块：



* 对输入进行线性变换，并且
* 使用变换结果来获取掩码tensor的索引。



 我们使用 `profiler.record_function("label")` 将每个子任务的代码包装在单独的标记上下文管理器中。
 。在探查器输出中，
子任务中所有操作的聚合性能指标
将显示在其相应的标签下。




 请注意，使用探查器会产生一些开销，最好仅用于调查
代码。如果您正在对运行时进行基准测试，请记住将其删除。



```
class MyModule(nn.Module):
    def __init__(self, in_features: int, out_features: int, bias: bool = True):
        super(MyModule, self).__init__()
        self.linear = nn.Linear(in_features, out_features, bias)

    def forward(self, input, mask):
        with profiler.record_function("LINEAR PASS"):
            out = self.linear(input)

        with profiler.record_function("MASK INDICES"):
            threshold = out.sum(axis=1).mean().item()
            hi_idx = np.argwhere(mask.cpu().numpy() > threshold)
            hi_idx = torch.from_numpy(hi_idx).cuda()

        return out, hi_idx

```


## 分析正向传递
 [¶](#profile-the-forward-pass "永久链接到此标题")




 我们初始化随机输入和掩码tensor以及模型。




 在运行探查器之前，我们会预热 CUDA 以确保
准确的性能基准测试。我们将模块的前向传递包装在
 `profiler.profile`
 上下文管理器中。 
 `with_stack=True`
 参数附加
跟踪中操作的文件和行号。





!!! warning "警告"

    

`with_stack=True`
 会产生额外的开销，并且更适合研究代码。
如果您正在对性能进行基准测试，请记住将其删除。




```
model = MyModule(500, 10).cuda()
input = torch.rand(128, 500).cuda()
mask = torch.rand((500, 500, 500), dtype=torch.double).cuda()

# warm-up
model(input, mask)

with profiler.profile(with_stack=True, profile_memory=True) as prof:
    out, idx = model(input, mask)

```


## 打印探查器结果
 [¶](#print-profiler-results "永久链接到此标题")




 最后，我们打印探查器结果。
 `profiler.key_averages`
 按运算符名称聚合结果，也可以选择按输入
形状和/或堆栈跟踪事件聚合结果。
按输入形状分组对于确定模型
使用哪些tensor形状。




 在这里，我们使用
 `group_by_stack_n=5`
 它通过操作及其回溯(截断为最近的 5 个事件)聚合运行时，并
在他们注册的顺序。该表还可以通过传递 `sort_by`
 参数进行排序(请参阅
 [文档](https://pytorch.org/docs/stable/autograd.html#profiler) 
 n 表示
有效的排序键)。





 注意




 在笔记本中运行探查器时，您可能会在堆栈跟踪中看到类似
 `<ipython-input-18-193a910735e8>(13):
 

forward`
 的条目，而不是文件名。这些对应于
 `<notebook-cell>(行
 

 数字)：
 

 调用函数`
 。




```
print(prof.key_averages(group_by_stack_n=5).table(sort_by='self_cpu_time_total', row_limit=5))

"""
(Some columns are omitted)

------------- ------------ ------------ ------------ ---------------------------------
 Name Self CPU % Self CPU Self CPU Mem Source Location
------------- ------------ ------------ ------------ ---------------------------------
 MASK INDICES 87.88% 5.212s -953.67 Mb /mnt/xarfuse/.../torch/au
 <ipython-input-...>(10): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(9): <module>
 /mnt/xarfuse/.../IPython/

 aten::copy_ 12.07% 715.848ms 0 b <ipython-input-...>(12): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(9): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/

 LINEAR PASS 0.01% 350.151us -20 b /mnt/xarfuse/.../torch/au
 <ipython-input-...>(7): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(9): <module>
 /mnt/xarfuse/.../IPython/

 aten::addmm 0.00% 293.342us 0 b /mnt/xarfuse/.../torch/nn
 /mnt/xarfuse/.../torch/nn
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(8): forward
 /mnt/xarfuse/.../torch/nn

 aten::mean 0.00% 235.095us 0 b <ipython-input-...>(11): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(9): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/

----------------------------- ------------ ---------- ----------------------------------
Self CPU time total: 5.931s

"""

```


## 提高内存性能
 [¶](#improve-memory-performance "永久链接到此标题")




 请注意，最昂贵的操作 - 就内存和时间而言 -
 位于
 `forward
 

 (10)`
 表示 MASK INDICES 内的操作。让’s 首先尝试解决内存消耗问题。我们可以看到第 12 行的
 `.to()`
 操作消耗了 953.67 Mb。此操作将
 `mask`
 复制到CPU。
 `mask`
 使用
 `torch.double`
 数据类型进行初始化。我们可以通过将
 转换为
 `torch.float`
 来减少内存占用吗？



```
model = MyModule(500, 10).cuda()
input = torch.rand(128, 500).cuda()
mask = torch.rand((500, 500, 500), dtype=torch.float).cuda()

# warm-up
model(input, mask)

with profiler.profile(with_stack=True, profile_memory=True) as prof:
    out, idx = model(input, mask)

print(prof.key_averages(group_by_stack_n=5).table(sort_by='self_cpu_time_total', row_limit=5))

"""
(Some columns are omitted)

----------------- ------------ ------------ ------------ --------------------------------
 Name Self CPU % Self CPU Self CPU Mem Source Location
----------------- ------------ ------------ ------------ --------------------------------
 MASK INDICES 93.61% 5.006s -476.84 Mb /mnt/xarfuse/.../torch/au
 <ipython-input-...>(10): forward
 /mnt/xarfuse/ /torch/nn
 <ipython-input-...>(9): <module>
 /mnt/xarfuse/.../IPython/

 aten::copy_ 6.34% 338.759ms 0 b <ipython-input-...>(12): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(9): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/

 aten::as_strided 0.01% 281.808us 0 b <ipython-input-...>(11): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(9): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/

 aten::addmm 0.01% 275.721us 0 b /mnt/xarfuse/.../torch/nn
 /mnt/xarfuse/.../torch/nn
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(8): forward
 /mnt/xarfuse/.../torch/nn

 aten::_local 0.01% 268.650us 0 b <ipython-input-...>(11): forward
 _scalar_dense /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(9): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/

----------------- ------------ ------------ ------------ --------------------------------
Self CPU time total: 5.347s

"""

```




 此操作的 CPU 内存占用量已减半。


## 提高时间性能
 [¶](#improve-time-performance "永久链接到此标题")




 虽然消耗的时间也减少了一点，但 ’s 仍然太高。
事实证明，将矩阵从 CUDA 复制到 CPU 的成本相当昂贵！

 `aten::copy_` 
 `forward
 

 (12)`
 中的运算符将
 `mask`
 复制到 CPU
，以便它可以使用 NumPy
 `argwhere`
 函数。
 ` aten::copy_`
 at
 `forward(13)`
 将数组作为tensor复制回 CUDA。如果我们在这里使用
 `torch`
 函数
 `nonzero()`
 来代替，我们就可以消除这两个问题。



```
class MyModule(nn.Module):
    def __init__(self, in_features: int, out_features: int, bias: bool = True):
        super(MyModule, self).__init__()
        self.linear = nn.Linear(in_features, out_features, bias)

    def forward(self, input, mask):
        with profiler.record_function("LINEAR PASS"):
            out = self.linear(input)

        with profiler.record_function("MASK INDICES"):
            threshold = out.sum(axis=1).mean()
            hi_idx = (mask > threshold).nonzero(as_tuple=True)

        return out, hi_idx


model = MyModule(500, 10).cuda()
input = torch.rand(128, 500).cuda()
mask = torch.rand((500, 500, 500), dtype=torch.float).cuda()

# warm-up
model(input, mask)

with profiler.profile(with_stack=True, profile_memory=True) as prof:
    out, idx = model(input, mask)

print(prof.key_averages(group_by_stack_n=5).table(sort_by='self_cpu_time_total', row_limit=5))

"""
(Some columns are omitted)

-------------- ------------ ------------ ------------ ---------------------------------
 Name Self CPU % Self CPU Self CPU Mem Source Location
-------------- ------------ ------------ ------------ ---------------------------------
 aten::gt 57.17% 129.089ms 0 b <ipython-input-...>(12): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(25): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/

 aten::nonzero 37.38% 84.402ms 0 b <ipython-input-...>(12): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(25): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/

 INDEX SCORE 3.32% 7.491ms -119.21 Mb /mnt/xarfuse/.../torch/au
 <ipython-input-...>(10): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(25): <module>
 /mnt/xarfuse/.../IPython/

aten::as_strided 0.20% 441.587us 0 b <ipython-input-...>(12): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(25): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/

 aten::nonzero
 _numpy 0.18% 395.602us 0 b <ipython-input-...>(12): forward
 /mnt/xarfuse/.../torch/nn
 <ipython-input-...>(25): <module>
 /mnt/xarfuse/.../IPython/
 /mnt/xarfuse/.../IPython/
-------------- ------------ ------------ ------------ ---------------------------------
Self CPU time total: 225.801ms

"""

```


## 进一步阅读
 [¶](#further-reading "此标题的永久链接")




 我们已经了解了如何使用 Profiler 来研究 PyTorch 模型中的时间和内存瓶颈。
在此处了解有关 Profiler 的更多信息：



* [分析器使用方法](https://pytorch.org/tutorials/recipes/recipes/profiler.html)
* [分析基于 RPC 的工作负载](https://pytorch.org/tutorials/recipes/distributed_rpc_profiling. html)
* [Profiler API 文档](https://pytorch.org/docs/stable/autograd.html?highlight=profiler#profiler)



**脚本的总运行时间:** 
 ( 0 分 0.000 秒)



[`下载
 

 Python
 

 源
 

 代码:
 

 profiler.py`](../_downloads/1df539a85371bf035ce170fb872b4f7f/profiler.py)



[`下载
 

 Jupyter
 

 笔记本:
 

 profiler.ipynb`](../_downloads/9fc6c90b1bbbfd4201d66c498708f33f/profiler.ipynb)



[Sphinx-Gallery 生成的图库](https://sphinx-gallery.github.io)







 =
 


 'self_cpu_time_total'
* ) [[source]](_modules/torch/profiler/profiler.html#_KinetoProfile.export_stacks)[¶](#torch.profiler._KinetoProfile.export_stacks "此定义的永久链接")


 以适合可视化的格式将堆栈跟踪保存在文件中。


 参数 
* **path** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 将堆栈文件保存到此location;
* **metric** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要使用的度量： “self_cpu_time_total”或“self_cuda_time_total”



!!! note "笔记"

    使用 FlameGraph 工具的示例：



* git clone <https://github.com/brendangregg/FlameGraph>
* cd FlameGraph
*./flamegraph.pl –title “CPU time” –countname “us”。 profiler.stacks 
> perf_viz.svg


 关键_平均值


 ( *group_by_input_shape



 =
 


 假
* , *group_by_stack_n



 =
 


 0
* ) [[source]](_modules/torch/profiler/profiler.html#_KinetoProfile.key_averages)[¶](#torch.profiler._KinetoProfile.key_averages "此定义的永久链接")


 平均事件，按操作员名称和(可选)输入形状和堆栈对它们进行分组。




!!! note "笔记"

    要使用形状/堆栈功能，请确保在创建探查器上下文管理器时设置 record_shapes/with_stack。


*班级*


 火炬.探查器。


 轮廓


 ( *** ， *活动



 =
 


 无
* , *时间表



 =
 


 无
* , *on_trace_ready



 =
 


 无
* , *记录_shapes



 =
 


 假*，*配置文件_内存



 =
 


 False
* , *with_stack



 =
 


 False
* , *带_flops



 =
 


 False
* , *with_modules



 =
 


 假*，*实验_config



 =
 


 无
* , *使用_cuda



 =
 


 无
* ) [[source]](_modules/torch/profiler/profiler.html#profile)[¶](#torch.profiler.profile "此定义的永久链接")


 探查器上下文管理器。


 参数 
* **activities** ( *iterable
* ) – 用于分析的活动组(CPU、CUDA)列表，支持的值： `torch.profiler.ProfilerActivity.CPU` 、 `torch.profiler.ProfilerActivity.CUDA` 。默认值：ProfilerActivity.CPU 和(如果可用)ProfilerActivity.CUDA.
* **schedule** ( *Callable
* ) – 可调用，将步骤 (int) 作为单个参数并返回指定探查器操作的 `ProfilerAction` 值在每个步骤执行。
* **on_trace_ready** ( *Callable
* ) – 在分析过程中，当 `schedule` 返回 `ProfilerAction.RECORD_AND_SAVE` 时，在每个步骤调用的可调用函数。
* **记录_shapes** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 保存有关运算符输入形状的信息。
* **profile_memory** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 跟踪tensor内存分配/deallocation.
* **with_stack** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 记录源操作的信息(文件和行号)。
* **with_flops** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3 中.12)") ) – 使用公式估计特定运算符(矩阵乘法和 2D 卷积)的 FLOP(浮点运算)。
* **with_modules** ( [*bool*](https://docs. python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 记录与操作的调用堆栈相对应的模块层次结构(包括函数名称)。例如如果模块 A 的前向调用模块 B 的前向包含 aten::add 操作，则 aten::add 的模块层次结构为 A。B请注意，目前此支持仅适用于 TorchScript 模型，不适用于 eager 模式模型。
* ** Experimental_config** ( *_ExperimentalConfig
* ) – 用于 Kineto 库功能的一组实验选项。请注意，不保证向后兼容性。
* **use_cuda** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中) ”) ) –


 自版本 1.8.1 起已弃用：使用 `activities` 代替。



!!! note "笔记"

    使用 [`schedule()`](#torch.profiler.schedule "torch.profiler.schedule") 生成可调用的计划。非默认计划在分析长时间训练作业时非常有用，并允许用户在不同的时间获取多个跟踪训练过程的迭代。默认计划只是在上下文管理器的持续时间内连续记录所有事件。


!!! note "笔记"

    使用 [`tensorboard_trace_handler()`](#torch.profiler.tensorboard_trace_handler "torch.profiler.tensorboard_trace_handler") 生成 TensorBoard 的结果文件：


`on_trace_ready=torch.profiler.tensorboard_trace_handler(dir_name)`


 分析后，可以在指定目录中找到结果文件。使用命令：


`tensorboard --logdir dir_name`


 在 TensorBoard 中查看结果。有关更多信息，请参阅 [PyTorch Profiler TensorBoard 插件](https://github.com/pytorch/kineto/tree/master/tb_plugin)



!!! note "笔记"

    启用形状和堆栈跟踪会导致额外的开销。当指定 record_shapes=True 时，探查器将暂时保存对tensor的引用；这可能会进一步阻止依赖于引用计数的某些优化并引入额外的tensor副本。


 例子：


```
with torch.profiler.profile(
    activities=[
        torch.profiler.ProfilerActivity.CPU,
        torch.profiler.ProfilerActivity.CUDA,
    ]
) as p:
    code_to_profile()
print(p.key_averages().table(
    sort_by="self_cuda_time_total", row_limit=-1))

```


 使用分析器的 `schedule` 、 `on_trace_ready` 和 `step` 函数：


```
# Non-default profiler schedule allows user to turn profiler on and off
# on different iterations of the training loop;
# trace_handler is called every time a new trace becomes available
def trace_handler(prof):
    print(prof.key_averages().table(
        sort_by="self_cuda_time_total", row_limit=-1))
    # prof.export_chrome_trace("/tmp/test_trace_" + str(prof.step_num) + ".json")

with torch.profiler.profile(
    activities=[
        torch.profiler.ProfilerActivity.CPU,
        torch.profiler.ProfilerActivity.CUDA,
    ],

    # In this example with wait=1, warmup=1, active=2, repeat=1,
    # profiler will skip the first step/iteration,
    # start warming up on the second, record
    # the third and the forth iterations,
    # after which the trace will become available
    # and on_trace_ready (when set) is called;
    # the cycle repeats starting with the next step

    schedule=torch.profiler.schedule(
        wait=1,
        warmup=1,
        active=2,
        repeat=1),
    on_trace_ready=trace_handler
    # on_trace_ready=torch.profiler.tensorboard_trace_handler('./log')
    # used when outputting for tensorboard
    ) as p:
        for iter in range(N):
            code_iteration_to_profile(iter)
            # send a signal to the profiler that the next iteration has started
            p.step()

```




 step
 


 ( ) [[source]](_modules/torch/profiler/profiler.html#profile.step)[¶](#torch.profiler.profile.step "此定义的永久链接")


 向分析器发出信号，表明下一个分析步骤已开始。


*班级*


 火炬.探查器。


 分析器动作


 ( *value
* ) [[source]](_modules/torch/profiler/profiler.html#ProfilerAction)[¶](#torch.profiler.ProfilerAction "此定义的永久链接")


 可以按指定时间间隔执行的探查器操作


*班级*


 火炬.探查器。


 ProfilerActivity [¶](#torch.profiler.ProfilerActivity "此定义的永久链接")


 成员：



 CPU
 



 XPU
 



 MTIA
 



 CUDA
 


*财产*


 name [¶](#torch.profiler.ProfilerActivity.name "此定义的永久链接")


 火炬.探查器。


 日程


 (***、*等待*、*热身*、*活动*、*重复



 =
 


 0
* , *跳过_first



 =
 


 0
* ) [[source]](_modules/torch/profiler/profiler.html#schedule)[¶](#torch.profiler.schedule "此定义的永久链接")


 返回一个可用作探查器“schedule”参数的可调用对象。分析器将跳过第一个“skip_first”步骤，然后等待“wait”步骤，然后为下一个“warmup”步骤进行预热，然后为下一个“active”步骤进行活动记录，然后重复开始的循环与“wait”步骤。可选的周期数由“repeat”参数指定，零值意味着周期将继续，直到分析完成。


 Return type


[*Callable*](https://docs.python.org/3/library/typing.html#typing.Callable“(在 Python v3.12 中)”)


 火炬.探查器。


 tensor板_trace_handler


 ( *dir_name
* , *worker_name



 =
 


 无
* , *使用_gzip



 =
 


 False
* ) [[source]](_modules/torch/profiler/profiler.html#tensorboard_trace_handler)[¶](#torch.profiler.tensorboard_trace_handler "此定义的永久链接")


 将跟踪文件输出到 `dir_name` 目录，然后该目录可以作为 logdir 直接传递到tensorboard。在分布式场景中，`worker_name`对于每个worker应该是唯一的，默认情况下将设置为'[hostname]_[pid]'。


## 英特尔仪器和跟踪技术 API [¶](#intel-instrumentation-and-tracing-technology-apis"此标题的永久链接")


 torch.profiler.itt。


 可用


 ( ) [[source]](_modules/torch/profiler/itt.html#is_available)[¶](#torch.profiler.itt.is_available "此定义的永久链接")


 检查 ITT 功能是否可用


 torch.profiler.itt。



 mark
 


 ( *msg
* ) [[source]](_modules/torch/profiler/itt.html#mark)[¶](#torch.profiler.itt.mark "此定义的永久链接")


 描述在某个时刻发生的瞬时事件。


 Parameters


**msg** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 与事件关联的 ASCII 消息。


 torch.profiler.itt。


 范围_push


 ( *msg
* ) [[source]](_modules/torch/profiler/itt.html#range_push)[¶](#torch.profiler.itt.range_push "此定义的永久链接")


 将范围推入嵌套范围跨度的堆栈上。返回开始范围的从零开始的深度。


 Parameters


**msg** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 与范围关联的 ASCII 消息


 torch.profiler.itt。


 范围_pop


 ( ) [[source]](_modules/torch/profiler/itt.html#range_pop)[¶](#torch.profiler.itt.range_pop "此定义的永久链接")


 从一堆嵌套范围跨度中弹出一个范围。返回结束范围的从零开始的深度。