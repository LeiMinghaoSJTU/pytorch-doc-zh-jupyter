# 自动微分包 
- torch.autograd [¶](#module-torch.autograd "永久链接到此标题")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/autograd>
>
> 原始地址：<https://pytorch.org/docs/stable/autograd.html>


`torch.autograd` 提供了实现任意标量值函数自动微分的类和函数。它需要对现有代码进行最少的更改 
- 您只需要声明“Tensor”，以便使用“requires_grad=True”关键字计算哪些梯度。到目前为止，我们仅支持浮点“Tensor”类型的 autograd(一半) 、float、double 和 bfloat16)和复杂的“Tensor”类型(cfloat、cdouble)。


|  |  |
| --- | --- |
| [`向后`](generated/torch.autograd.backward.html#torch.autograd.backward "torch.autograd.backward") | 计算给定tensor相对于图叶的梯度总和。 |
|[`grad`](generated/torch.autograd.grad.html#torch.autograd.grad“torch.autograd.grad”) | 计算并返回输出相对于输入的梯度总和。 |


## 正向模式自动区分 [¶](#forward-mode-automatic-differiation "永久链接到此标题")


!!! warning "警告"

     该 API 处于测试阶段。尽管函数签名不太可能改变，但在我们认为它稳定之前，我们已经计划了改进操作符覆盖范围。


 有关如何使用此 API 的详细步骤，请参阅[转发模式 AD 教程](https://pytorch.org/tutorials/intermediate/forward_ad_usage.html)。


|  |  |
| --- | --- |
| 	[`forward_ad.dual_level`](generated/torch.autograd.forward_ad.dual_level.html#torch.autograd.forward_ad.dual_level "torch.autograd.forward_ad.dual_level")	 | 	 Context-manager that enables forward AD.	  |
| 	[`forward_ad.make_dual`](generated/torch.autograd.forward_ad.make_dual.html#torch.autograd.forward_ad.make_dual "torch.autograd.forward_ad.make_dual")	 | 	 Associates a tensor value with a forward gradient, the tangent, to create a "dual tensor", which is used to compute forward AD gradients.	  |
| 	[`forward_ad.unpack_dual`](generated/torch.autograd.forward_ad.unpack_dual.html#torch.autograd.forward_ad.unpack_dual "torch.autograd.forward_ad.unpack_dual")	 | 	 Unpacks a "dual tensor" to get both its Tensor value and its forward AD gradient.	  |


## 功能性更高级别 API [¶](#function-higher-level-api "此标题的永久链接")


!!! warning "警告"

     该 API 处于测试阶段。尽管函数签名不太可能改变，但在我们认为稳定之前，我们已经计划对性能进行重大改进。


 本节包含 autograd 的更高级别 API，它构建在上述基本 API 的基础上，并允许您计算雅可比矩阵、粗麻布矩阵等。


 此 API 可与用户提供的函数配合使用，这些函数仅采用tensor作为输入并仅返回tensor。如果您的函数采用非tensor的其他参数或没有设置 require_grad 的tensor，则可以使用 lambda 来捕获它们。例如，对于需要三个输入的函数“f”，一个我们想要雅可比的tensor，另一个应被视为常量的tensor和一个布尔标志“f(input,constant,flag=flag)”，您可以将其用作`function.jacobian(lambda x: f(x, 常量, flag=flag), 输入)` 。


|  |  |
| --- | --- |
| 	[`functional.jacobian`](generated/torch.autograd.functional.jacobian.html#torch.autograd.functional.jacobian "torch.autograd.functional.jacobian")	 | 	 Function that computes the Jacobian of a given function.	  |
| 	[`functional.hessian`](generated/torch.autograd.functional.hessian.html#torch.autograd.functional.hessian "torch.autograd.functional.hessian")	 | 	 Function that computes the Hessian of a given scalar function.	  |
| 	[`functional.vjp`](generated/torch.autograd.functional.vjp.html#torch.autograd.functional.vjp "torch.autograd.functional.vjp")	 | 	 Function that computes the dot product between a vector	 `v`	 and the Jacobian of the given function at the point given by the inputs.	  |
| 	[`functional.jvp`](generated/torch.autograd.functional.jvp.html#torch.autograd.functional.jvp "torch.autograd.functional.jvp")	 | 	 Function that computes the dot product between the Jacobian of the given function at the point given by the inputs and a vector	 `v`	.	  |
| 	[`functional.vhp`](generated/torch.autograd.functional.vhp.html#torch.autograd.functional.vhp "torch.autograd.functional.vhp")	 | 	 Function that computes the dot product between a vector	 `v`	 and the Hessian of a given scalar function at the point given by the inputs.	  |
| 	[`functional.hvp`](generated/torch.autograd.functional.hvp.html#torch.autograd.functional.hvp "torch.autograd.functional.hvp")	 | 	 Function that computes the dot product between the Hessian of a given scalar function and a vector	 `v`	 at the point given by the inputs.	  |


## 本地禁用梯度计算 [¶](#locally-disabling-gradient-computation "永久链接到此标题")


 有关无梯度和推理模式之间的差异以及可能与两者混淆的其他相关机制的更多信息，请参阅[本地禁用梯度计算](notes/autograd.html#locally-disable-grad-doc)。另请参阅[本地禁用梯度计算](torch.html#torch-rst-local-disable-grad)，了解可用于本地禁用梯度的函数列表。


## 默认渐变布局 [¶](#default-gradient-layouts "永久链接到此标题")


 当非稀疏 `param` 在 [`torch.autograd.backward()`](generated/torch.autograd.backward.html#torch.autograd.backward "torch.autograd.backward") 期间收到非稀疏梯度时或 [`torch.Tensor.backward()`](generated/torch.Tensor.backward.html#torch.Tensor.backward "torch.Tensor.backward")`param.grad` 累积如下。


 如果 `param.grad` 最初是 `None` ：


1. 如果 `param` 的内存是非重叠且密集的，则使用与 `param` 匹配的步幅创建 `.grad`(从而匹配 `param` 的布局)。2.否则，会使用行主连续步幅创建“.grad”。


 如果 `param` 已经有一个非稀疏的 `.grad` 属性：


3. 如果 `create_graph=False` ， `backward()` 就地累积到 `.grad` 中，从而保留其步幅。4.如果 create_graph=True ，则 backward() 会用新的tensor.grad 
+ new grad 替换.grad ，它会尝试(但不保证)匹配预先存在的.grad 的步幅。


 建议使用默认行为(在第一个 `backward()` 之前让 `.grad` 为 `None`，以便根据 1 或 2 创建它们的布局，并根据 3 或 4 随着时间的推移保留它们的布局)以获得最佳性能.调用“model.zero_grad()”或“optimizer.zero_grad()”不会影响“.grad”布局。


 事实上，在每个累积阶段之前将所有“.grad”重置为“None”，例如：


```
for iterations...
    ...
    for param in model.parameters():
        param.grad = None
    loss.backward()

```


 这样每次都会根据 1 或 2 重新创建它们，是“model.zero_grad()”或“optimizer.zero_grad()”的有效替代方案，可以提高某些网络的性能。


### 手动渐变布局 [¶](#manual-gradient-layouts "永久链接到此标题")


 如果您需要手动控制 `.grad` 的步长，请在第一个 `backward()` 之前指定 `param.grad =` 一个具有所需步长的归零tensor，并且永远不要将其重置为 `None`.3 保证您的布局被保留只要 `create_graph=False`.4 表明即使 `create_graph=True` ，您的布局也“可能”被保留。


## tensor上的就地操作 [¶](#in-place-operations-on-tensors "永久链接到此标题")


 支持 autograd 中的就地操作是一件困难的事情，我们在大多数情况下不鼓励使用它们。 Autograd 积极的缓冲区释放和重用使其非常高效，并且很少有情况下就地操作实际上会显着降低内存使用量。除非您在内存压力很大的情况下运行，否则您可能永远不需要使用它们。


### 就地正确性检查 [¶](#in-place
- Correctness-checks "永久链接到此标题")


 所有“tensor”都会跟踪应用于它们的就地操作，如果实现检测到在其中一个函数中为向后保存了一个tensor，但随后对其进行了就地修改，则一旦开始向后传递，就会引发错误。这确保了如果您使用就地函数并且没有看到任何错误，则可以确定计算的梯度是正确的。


## 变量(已弃用)[¶](#variable-deprecated "此标题的永久链接")


!!! warning "警告"

     变量 API 已被弃用：使用带有tensor的 autograd 不再需要变量。 Autograd 自动支持将 `requires_grad` 设置为 `True` 的tensor。请在下面找到有关更改内容的快速指南：



* `Variable(tensor)` 和 `Variable(tensor, require_grad)` 仍然按预期工作，但它们返回tensor而不是变量。
* `var.data` 与 `tensor.data` 相同。
* 方法例如 `var.backward()、var.detach()、var.register_hook()` 现在可以处理具有相同方法名称的tensor。


 此外，现在可以使用工厂方法创建带有 `requires_grad=True` 的tensor，例如 [`torch.randn()`](generated/torch.randn.html#torch.randn "torch.randn") 、 [` torch.zeros()`](generated/torch.zeros.html#torch.zeros "torch.zeros") , [`torch.ones()`](generated/torch.ones.html#torch.ones "火炬.的”)，以及其他类似的内容：


`autograd_tensor = torch.randn((2, 3, 4), 要求_grad=True)`


## tensor自动求导函数 [¶](#tensor-autograd-functions "此标题的固定链接")


|  |  |
| --- | --- |
| 	`torch.Tensor.grad`	 | 	 This attribute is	 `None`	 by default and becomes a Tensor the first time a call to	 [`backward()`](generated/torch.autograd.backward.html#torch.autograd.backward "torch.autograd.backward")	 computes gradients for	 `self`	.	  |
| 	`torch.Tensor.requires_grad`	 | 	 Is	 `True`	 if gradients need to be computed for this Tensor,	 `False`	 otherwise.	  |
| 	`torch.Tensor.is_leaf`	 | 	 All Tensors that have	 `requires_grad`	 which is	 `False`	 will be leaf Tensors by convention.	  |
| 	`torch.Tensor.backward`	 ([gradient, ...])	  | 	 Computes the gradient of current tensor wrt graph leaves.	  |
| 	`torch.Tensor.detach`	 | 	 Returns a new Tensor, detached from the current graph.	  |
| 	`torch.Tensor.detach_`	 | 	 Detaches the Tensor from the graph that created it, making it a leaf.	  |
| 	`torch.Tensor.register_hook`	 (hook)	  | 	 Registers a backward hook.	  |
| 	`torch.Tensor.register_post_accumulate_grad_hook`	 (hook)	  | 	 Registers a backward hook that runs after grad accumulation.	  |
| 	`torch.Tensor.retain_grad`	 ()	  | 	 Enables this Tensor to have their	 [`grad`](generated/torch.autograd.grad.html#torch.autograd.grad "torch.autograd.grad")	 populated during	 [`backward()`](generated/torch.autograd.backward.html#torch.autograd.backward "torch.autograd.backward")	.	  |


## 函数 [¶](#function "此标题的永久链接")


*班级*


 火炬.autograd。


 功能


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/autograd/function.html#Function)[¶](#torch.autograd.Function "此定义的永久链接")


 创建自定义 autograd.Function 的基类


 要创建自定义 autograd.Function ，请子类化此类并实现 [`forward()`](generated/torch.autograd.Function.forward.html#torch.autograd.Function.forward "torch.autograd.Function.forward")和 [`backward()`](generated/torch.autograd.backward.html#torch.autograd.backward "torch.autograd.backward") 静态方法。然后，要在前向传播中使用您的自定义操作，请调用类方法“apply”。不要直接调用 [`forward()`](generated/torch.autograd.Function.forward.html#torch.autograd.Function.forward "torch.autograd.Function.forward")。


 为了确保正确性和最佳性能，请确保您在“ctx”上调用正确的方法，并使用 [`torch.autograd.gradcheck()`](generated/torch.autograd.gradcheck.html#torch.autograd. gradcheck“torch.autograd.gradcheck”)。


 有关如何使用此类的更多详细信息，请参阅[扩展 torch.autograd](notes/extending.html#extending-autograd)。


 例子：


```
>>> class Exp(Function):
>>>     @staticmethod
>>>     def forward(ctx, i):
>>>         result = i.exp()
>>>         ctx.save_for_backward(result)
>>>         return result
>>>
>>>     @staticmethod
>>>     def backward(ctx, grad_output):
>>>         result, = ctx.saved_tensors
>>>         return grad_output * result
>>>
>>> # Use it by calling the apply method:
>>> output = Exp.apply(input)

```


|  |  |
| --- | --- |
| 	[`Function.forward`](generated/torch.autograd.Function.forward.html#torch.autograd.Function.forward "torch.autograd.Function.forward")	 | 	 This function is to be overridden by all subclasses.	  |
| 	[`Function.backward`](generated/torch.autograd.Function.backward.html#torch.autograd.Function.backward "torch.autograd.Function.backward")	 | 	 Defines a formula for differentiating the operation with backward mode automatic differentiation (alias to the vjp function).	  |
| 	[`Function.jvp`](generated/torch.autograd.Function.jvp.html#torch.autograd.Function.jvp "torch.autograd.Function.jvp")	 | 	 Defines a formula for differentiating the operation with forward mode automatic differentiation.	  |
| 	[`Function.vmap`](generated/torch.autograd.Function.vmap.html#torch.autograd.Function.vmap "torch.autograd.Function.vmap")	 | 	 Defines a rule for the behavior of this autograd.Function underneath	 [`torch.vmap()`](generated/torch.vmap.html#torch.vmap "torch.vmap")	.	  |


## 上下文方法 mixins [¶](#context-method-mixins "永久链接到此标题")


 创建新的 [`Function`](#torch.autograd.Function "torch.autograd.Function") 时，以下方法可用于 ctx 。


|  |  |
| --- | --- |
| 	[`function.FunctionCtx.mark_dirty`](generated/torch.autograd.function.FunctionCtx.mark_dirty.html#torch.autograd.function.FunctionCtx.mark_dirty "torch.autograd.function.FunctionCtx.mark_dirty")	 | 	 Marks given tensors as modified in an in-place operation.	  |
| 	[`function.FunctionCtx.mark_non_differentiable`](generated/torch.autograd.function.FunctionCtx.mark_non_differentiable.html#torch.autograd.function.FunctionCtx.mark_non_differentiable "torch.autograd.function.FunctionCtx.mark_non_differentiable")	 | 	 Marks outputs as non-differentiable.	  |
| 	[`function.FunctionCtx.save_for_backward`](generated/torch.autograd.function.FunctionCtx.save_for_backward.html#torch.autograd.function.FunctionCtx.save_for_backward "torch.autograd.function.FunctionCtx.save_for_backward")	 | 	 Saves given tensors for a future call to	 [`backward()`](generated/torch.autograd.Function.backward.html#torch.autograd.Function.backward "torch.autograd.Function.backward")	.	  |
| 	[`function.FunctionCtx.set_materialize_grads`](generated/torch.autograd.function.FunctionCtx.set_materialize_grads.html#torch.autograd.function.FunctionCtx.set_materialize_grads "torch.autograd.function.FunctionCtx.set_materialize_grads")	 | 	 Sets whether to materialize grad tensors.	  |


## 数值梯度检查[¶](#numerical-gradient-checking "此标题的固定链接")


|  |  |
| --- | --- |
| [`gradcheck`](generated/torch.autograd.gradcheck.html#torch.autograd.gradcheck "torch.autograd.gradcheck") |检查通过小有限差分计算的梯度与浮点或复杂类型的“输入”中的解析梯度tensor以及“requires_grad=True”。 |
| [`gradgradcheck`](generated/torch.autograd.gradgradcheck.html#torch.autograd.gradgradcheck“torch.autograd.gradgradcheck”)|检查通过小有限差分计算出的梯度与“inputs”和“grad_outputs”中浮点或复数类型且具有“requires_grad=True”tensor的解析梯度的梯度。 |


## 探查器 [¶](#profiler "此标题的永久链接")


 Autograd 包含一个分析器，可让您检查模型内不同运算符的成本 
- 无论是在 CPU 上还是在 GPU 上。目前实现了三种模式 
- 仅 CPU 使用 [`profile`](#torch.autograd.profiler.profile "torch.autograd.profiler.profile").nvprof 基于(注册 CPU 和 GPU 活动)使用 [` emit_nvtx`](#torch.autograd.profiler.emit_nvtx "torch.autograd.profiler.emit_nvtx") 和基于 vtune 分析器使用 [`emit_itt`](#torch.autograd.profiler.emit_itt "torch.autograd.profiler.emit_itt"​​) 。


*班级*


 torch.autograd.profiler。


 轮廓


 (*已启用



 =
 


 真*，***，*使用_cuda



 =
 


 假*，*使用_device



 =
 


 无
* , *记录_shapes



 =
 


 False
* , *带_flops



 =
 


 假*，*配置文件_内存



 =
 


 False
* , *with_stack



 =
 


 False
* , *with_modules



 =
 


 假
* , *使用_kineto



 =
 


 假
* , *使用_cpu



 =
 


 真*，*使用_mtia



 =
 


 假*，*实验_config



 =
 


 无
* ) [[source]](_modules/torch/autograd/profiler.html#profile)[¶](#torch.autograd.profiler.profile "此定义的永久链接")


 上下文管理器，管理 autograd 分析器状态并保存结果摘要。在幕后，它仅记录在 C++ 中执行的函数的事件，并将这些事件公开给 Python。您可以将任何代码包装到其中，它只会报告 PyTorch 函数的运行时。注意：探查器是线程本地的，并且会自动传播到异步任务中


 参数 
* **启用** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 将其设置为 False 会使该上下文管理器成为无操作。
* **use_cuda** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")*,
* *可选
* ) – 启用 CUDA 事件计时以及使用 cudaEvent API。为每个tensor操作添加大约 4us 的开销。
* **record_shapes** ( [*bool *](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果设置了形状记录，则有关输入尺寸的信息将被收集。这允许人们查看在幕后使用了哪些维度，并使用 prof.key_averages(group_by_input_shape=True) 对它们进行进一步分组。请注意，形状记录可能会扭曲您的分析数据。建议使用带形状记录和不带形状记录的单独运行来验证计时。最底部事件的偏差很可能可以忽略不计(在嵌套函数调用的情况下)。但对于更高级别的函数，由于 shapecollection，totalself cpu 时间可能会人为增加。
* **with_flops** ( [*bool*](https://docs.python.org/3/library/functions.html #bool "(在 Python v3.12 中)")*,
* *可选
* ) – 如果设置了 with_flops，分析器将使用运算符的输入形状来估计 FLOP(浮点运算)值。这允许人们估计硬件性能。目前，此选项仅适用于矩阵乘法和 2D 卷积运算符。
* **profile_memory** ( [*bool*](https://docs.python.org/3/library/functions.html#bool " (在 Python v3.12 中)")*,
* *可选
* ) – 跟踪tensor内存分配/解除分配。
* **with_stack** ( [*bool*](https://docs.python.org/3 /library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 记录操作的源信息(文件和行号)。
* **with_modules** ( [
* bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 记录与操作的调用堆栈对应的模块层次结构(包括函数名称) 。例如如果模块 A 的前向调用模块 B 的前向包含 aten::add 操作，则 aten::add 的模块层次结构为 A。B请注意，目前此支持仅适用于 TorchScript 模型，不适用于 eager 模式模型。
* ** use_kineto** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 实验性，使用 Kineto 分析器启用分析。
* **use_cpu** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)" )*,
* *可选
* ) – 分析 CPU 事件；设置为“False”需要“use_kineto=True”，可用于降低仅 GPU 分析的开销。
* **experimental_config** ( *_ExperimentalConfig
* ) – 分析器库使用的一组实验选项就像基内托一样。请注意，不保证向后兼容性。


 例子


```
>>> x = torch.randn((1, 1), requires_grad=True)
>>> with torch.autograd.profiler.profile() as prof:
>>>     for _ in range(100):  # any normal python code, really!
>>>         y = x ** 2
>>>         y.backward()
>>> # NOTE: some columns were removed for brevity
>>> print(prof.key_averages().table(sort_by="self_cpu_time_total"))
----------------------------------- --------------- --------------- ---------------
Name Self CPU total CPU time avg Number of Calls
----------------------------------- --------------- --------------- ---------------
mul 32.048ms 32.048ms 200
pow 27.041ms 27.041ms 200
PowBackward0 9.727ms 55.483ms 100
torch::autograd::AccumulateGrad 9.148ms 9.148ms 100
torch::autograd::GraphRoot 691.816us 691.816us 100
----------------------------------- --------------- --------------- ---------------

```


|  |  |
| --- | --- |
| 	[`profiler.profile.export_chrome_trace`](generated/torch.autograd.profiler.profile.export_chrome_trace.html#torch.autograd.profiler.profile.export_chrome_trace "torch.autograd.profiler.profile.export_chrome_trace")	 | 	 Exports an EventList as a Chrome tracing tools file.	  |
| 	[`profiler.profile.key_averages`](generated/torch.autograd.profiler.profile.key_averages.html#torch.autograd.profiler.profile.key_averages "torch.autograd.profiler.profile.key_averages")	 | 	 Averages all function events over their keys.	  |
| 	[`profiler.profile.self_cpu_time_total`](generated/torch.autograd.profiler.profile.self_cpu_time_total.html#torch.autograd.profiler.profile.self_cpu_time_total "torch.autograd.profiler.profile.self_cpu_time_total")	 | 	 Returns total time spent on CPU obtained as a sum of all self times across all the events.	  |
| 	[`profiler.profile.total_average`](generated/torch.autograd.profiler.profile.total_average.html#torch.autograd.profiler.profile.total_average "torch.autograd.profiler.profile.total_average")	 | 	 Averages all events.	  |


*班级*


 torch.autograd.profiler。


 发出_nvtx


 (*已启用



 =
 


 True
* , *记录_shapes



 =
 


 False
* ) [[source]](_modules/torch/autograd/profiler.html#emit_nvtx)[¶](#torch.autograd.profiler.emit_nvtx "此定义的永久链接")


 上下文管理器，使每个自动分级操作发出 NVTX 范围。


 在 nvprof 下运行程序时很有用：


```
nvprof --profile-from-start off -o trace_name.prof -- <regular command here>

```


 不幸的是，没有办法强制 nvprof 将其收集的数据刷新到磁盘，因此对于 CUDA 分析，必须使用此上下文管理器来注释 envprof 跟踪并等待进程退出，然后再检查它们。然后，使用 NVIDIA Visual Profiler (nvvp)可用于可视化时间线，或 [`torch.autograd.profiler.load_nvprof()`](generated/torch.autograd.profiler.load_nvprof.html#torch.autograd.profiler.load_nvprof "torch.autograd.profiler.load_nvprof") 可以加载检查结果，例如在 Python REPL 中。


 参数 
* **启用** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 设置 `enabled=False` 使该上下文管理器成为无操作。默认值：`True`.
* **record_shapes** ( [*bool*](https://docs.python.org/3/library) /functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `record_shapes=True` ，则 nvtx rangewrappingeach autograd 操作将附加有关该操作接收到的tensor参数大小的信息op，格式如下：`[[arg0.size(0), arg0.size(1),...], [arg1.size(0), arg1.size(1),...], 。..]` 非tensor参数将由 `[]` 表示。参数将按照后端操作接收的顺序列出。请注意，此顺序可能与这些参数在 Python 上传递的顺序不匹配边。另请注意，形状记录可能会增加 nvtx 范围创建的开销。默认值：`False`


 例子


```
>>> with torch.cuda.profiler.profile():
...     model(x)  # Warmup CUDA memory allocator and profiler
...     with torch.autograd.profiler.emit_nvtx():
...         model(x)

```


**前向-后向相关性**


 在 Nvidia Visual Profiler 中查看使用 [`emit_nvtx`](#torch.autograd.profiler.emit_nvtx "torch.autograd.profiler.emit_nvtx") 创建的配置文件时，将每个后向传递操作与相应的前向传递相关联为了缓解此任务，[`emit_nvtx`](#torch.autograd.profiler.emit_nvtx "torch.autograd.profiler.emit_nvtx") 将序列号信息附加到它生成的范围。


 在前向传递过程中，每个函数范围都用 `seq=<N>` 修饰。 `seq` 是一个运行计数器，每次创建新的后向 Function 对象并为后向存储时都会递增。因此，与每个前向函数范围关联的 seq=<N>` 注释告诉您，如果由此创建了后向 Function 对象向前函数，向后对象将接收序列号 N。在向后传递期间，包装每个 C++ 向后函数的 `apply()` 调用的顶级范围都用 `stashed seq=<M>` 修饰。 “M”是创建后向对象的序列号。通过比较向后的“stashed seq”数字与向前的“seq”数字，您可以追踪哪个前向操作创建了每个向后函数。


 向后传递期间执行的任何函数也用 `seq=<N>` 修饰。在默认向后(使用 `create_graph=False` )期间，此信息是无关紧要的，事实上，对于所有此类函数，`N` 可能只是 0。只有与后向 Function 对象的“apply()”方法关联的顶级范围才有用，作为将这些 Function 对象与早期前向传递相关联的一种方式。


**双后退**


 另一方面，如果正在使用 `create_graph=True` 进行向后传递(换句话说，如果您正在设置双向后)，则向后传递期间的每个函数的执行都会被赋予一个非零的、有用的 `seq= <N>` 。这些函数本身可能会创建稍后在双向后传递过程中执行的 Function 对象，就像前向传递中的原始函数一样。向后传递和双向后传递之间的关系在概念上与向前和向后之间的关系相同：函数仍然发出电流-sequence-number-tagged 范围，它们创建的 Function 对象仍然存储这些序列号，并且在最终的双向后期间，Function 对象的 `apply()` 范围仍然用 `stashed seq` 数字标记，可以进行比较从向后传递中对数字进行排序。


*班级*


 torch.autograd.profiler。


 在这里发出_


 (*已启用



 =
 


 True
* , *记录_shapes



 =
 


 False
* ) [[source]](_modules/torch/autograd/profiler.html#emit_itt)[¶](#torch.autograd.profiler.emit_itt "此定义的永久链接")


 上下文管理器使每个 autograd 操作发出一个 ITT 范围。


 在 Intel(R) VTune Profiler 下运行程序时它很有用：


```
vtune <--vtune-flags> <regular command here>

```


 仪器和跟踪技术 (ITT) API 使您的应用程序能够在跨不同英特尔工具执行期间生成和控制跟踪数据的收集。此上下文管理器用于注释英特尔(R) VTune 分析跟踪。借助此上下文管理器，您将能够在英特尔(R) VTune Profiler GUI 中查看标记范围。


 参数 
* **启用** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 设置 `enabled=False` 使该上下文管理器成为无操作。默认值：`True`.
* **record_shapes** ( [*bool*](https://docs.python.org/3/library) /functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `record_shapes=True` ，则 itt rangewrappingeach autograd 操作将附加有关接收到的tensor参数大小的信息op，格式如下：`[[arg0.size(0), arg0.size(1),...], [arg1.size(0), arg1.size(1),...], 。..]` 非tensor参数将由 `[]` 表示。参数将按照后端操作接收的顺序列出。请注意，此顺序可能与这些参数在 Python 上传递的顺序不匹配边。另请注意，形状记录可能会增加 itt 范围创建的开销。默认值：`False`


 例子


```
>>> with torch.autograd.profiler.emit_itt():
...     model(x)

```


|  |  |
| --- | --- |
| 	[`profiler.load_nvprof`](generated/torch.autograd.profiler.load_nvprof.html#torch.autograd.profiler.load_nvprof "torch.autograd.profiler.load_nvprof")	 | 	 Opens an nvprof trace file and parses autograd annotations.	  |


## 异常检测 [¶](#anomaly-detection "永久链接到此标题")


*班级*


 火炬.autograd。


 检测_异常


 (*报到



 =
 


 True
* ) [[source]](_modules/torch/autograd/anomaly_mode.html#detect_anomaly)[¶](#torch.autograd.detect_anomaly "此定义的永久链接")


 为 autograd 引擎启用异常检测的上下文管理器。


 这有两件事：



* 在启用检测的情况下运行前向传递将允许后向传递打印创建失败后向函数的前向操作的回溯。
* 如果 `check_nan` 为 `True` ，则任何生成“nan”值的后向计算都会引发错误。默认为“True”。


!!! warning "警告"

     仅应在调试时启用此模式，因为不同的测试会减慢程序的执行速度。


 例子


```
>>> import torch
>>> from torch import autograd
>>> class MyFunc(autograd.Function):
...     @staticmethod
...     def forward(ctx, inp):
...         return inp.clone()
...     @staticmethod
...     def backward(ctx, gO):
...         # Error during the backward pass
...         raise RuntimeError("Some error in backward")
...         return gO.clone()
>>> def run_fn(a):
...     out = MyFunc.apply(a)
...     return out.sum()
>>> inp = torch.rand(10, 10, requires_grad=True)
>>> out = run_fn(inp)
>>> out.backward()
 Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
 File "/your/pytorch/install/torch/_tensor.py", line 93, in backward
 torch.autograd.backward(self, gradient, retain_graph, create_graph)
 File "/your/pytorch/install/torch/autograd/__init__.py", line 90, in backward
 allow_unreachable=True) # allow_unreachable flag
 File "/your/pytorch/install/torch/autograd/function.py", line 76, in apply
 return self._forward_cls.backward(self, *args)
 File "<stdin>", line 8, in backward
 RuntimeError: Some error in backward
>>> with autograd.detect_anomaly():
...     inp = torch.rand(10, 10, requires_grad=True)
...     out = run_fn(inp)
...     out.backward()
 Traceback of forward call that caused the error:
 File "tmp.py", line 53, in <module>
 out = run_fn(inp)
 File "tmp.py", line 44, in run_fn
 out = MyFunc.apply(a)
 Traceback (most recent call last):
 File "<stdin>", line 4, in <module>
 File "/your/pytorch/install/torch/_tensor.py", line 93, in backward
 torch.autograd.backward(self, gradient, retain_graph, create_graph)
 File "/your/pytorch/install/torch/autograd/__init__.py", line 90, in backward
 allow_unreachable=True) # allow_unreachable flag
 File "/your/pytorch/install/torch/autograd/function.py", line 76, in apply
 return self._forward_cls.backward(self, *args)
 File "<stdin>", line 8, in backward
 RuntimeError: Some error in backward

```


*班级*


 火炬.autograd。


 设置_检测_异常


 (*模式*，*检查_nan



 =
 


 True
* ) [[source]](_modules/torch/autograd/anomaly_mode.html#set_detect_anomaly)[¶](#torch.autograd.set_detect_anomaly "此定义的永久链接")


 用于设置 autograd 引擎打开或关闭异常检测的上下文管理器。


`set_detect_anomaly` 将根据其参数 `mode` 启用或禁用自动分级异常检测。它可以用作上下文管理器或函数。


 有关异常检测行为的详细信息，请参阅上面的“Detect_anomaly”。


 参数 
* **mode** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 标记是否启用异常检测 ( `True` )，或禁用 ( `False` )。
* **check_nan** ( [*bool*](https://docs.python.org/3/library/functions.html#bool " (在 Python v3.12 中)") ) – 标记向后生成“nan”时是否引发错误


## Autograd 图 [¶](#autograd-graph "此标题的固定链接")


 Autograd 公开了一些方法，允许人们在向后传递过程中检查图形并插入行为。


 如果tensor是某个操作的输出，则 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 的 `grad_fn` 属性包含一个 `torch.autograd.graph.Node`由 autograd 记录的(即启用 grad_mode 且至少其中一个输入需要梯度)，否则为“无”。


|  |  |
| --- | --- |
| 	[`graph.Node.name`](generated/torch.autograd.graph.Node.name.html#torch.autograd.graph.Node.name "torch.autograd.graph.Node.name")	 | 	 Returns the name.	  |
| 	[`graph.Node.metadata`](generated/torch.autograd.graph.Node.metadata.html#torch.autograd.graph.Node.metadata "torch.autograd.graph.Node.metadata")	 | 	 Returns the metadata.	  |
| 	[`graph.Node.next_functions`](generated/torch.autograd.graph.Node.next_functions.html#torch.autograd.graph.Node.next_functions "torch.autograd.graph.Node.next_functions")	 | 	 |
| 	[`graph.Node.register_hook`](generated/torch.autograd.graph.Node.register_hook.html#torch.autograd.graph.Node.register_hook "torch.autograd.graph.Node.register_hook")	 | 	 Registers a backward hook.	  |
| 	[`graph.Node.register_prehook`](generated/torch.autograd.graph.Node.register_prehook.html#torch.autograd.graph.Node.register_prehook "torch.autograd.graph.Node.register_prehook")	 | 	 Registers a backward pre-hook.	  |


 有些操作需要在前向传递过程中保存中间结果，以便执行后向传递。这些中间结果被保存为 `grad_fn` 上的属性，并且可以被访问。例如：


```
>>> a = torch.tensor([0., 0., 0.], requires_grad=True)
>>> b = a.exp()
>>> print(isinstance(b.grad_fn, torch.autograd.graph.Node))
True
>>> print(dir(b.grad_fn))
['__call__', '__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '_raw_saved_result', '_register_hook_dict', '_saved_result', 'metadata', 'name', 'next_functions', 'register_hook', 'register_prehook', 'requires_grad']
>>> print(torch.allclose(b.grad_fn._saved_result, b))
True

```


 您还可以定义如何使用钩子打包/解包这些保存的tensor。一个常见的应用是将这些中间结果保存到磁盘或 CPU 而不是将它们保留在 GPU 上，从而以计算换取内存。如果您发现您的模型在评估期间适合 GPU，但不适合训练，这尤其有用。另请参阅[保存tensor的挂钩](notes/autograd.html#saved-tensors-hooks-doc)。


*班级*


 torch.autograd.graph。


 已保存_tensors_hooks


 ( *pack_hook
* , *unpack_hook
* ) [[source]](_modules/torch/autograd/graph.html#saved_tensors_hooks)[¶](#torch.autograd.graph.saved_tensors_hooks "此定义的永久链接")


 上下文管理器，为保存的tensor设置一对打包/解包钩子。


 使用此上下文管理器来定义操作的中间结果在保存之前应如何打包，以及在检索时如何解包。


 在这种情况下，每次操作保存向后tensor时都会调用“pack_hook”函数(这包括使用“save_for_backward()”保存的中间结果，还包括由 PyTorch 定义的操作记录的结果)。然后，“pack_hook”的输出而不是原始tensor存储在计算图中。


 当需要访问保存的tensor时，即执行 [`torch.Tensor.backward()`](generated/torch.Tensor.backward.html#torch.Tensor.backward "torch. Tensor.backward") 或 [`torch.autograd.grad()`](generated/torch.autograd.grad.html#torch.autograd.grad "torch.autograd.grad") 。它以 `pack_hook` 返回的 *packed
* 对象作为参数，并且应该返回一个与原始tensor具有相同内容的tensor(作为输入传递给相应的 `pack_hook` )。


 挂钩应具有以下签名：



> 
> 
> 
> pack_hook(tensor: Tensor) -
> Any
> 
> 
> 
> 
> unpack_hook(Any) -
> Tensor
> 
> 
> 
> >


 其中 `pack_hook` 的返回值是 `unpack_hook` 的有效输入。


 一般来说，您希望 `unpack_hook(pack_hook(t))` 在值、大小、数据类型和设备方面等于 `t`。


 例子：


```
>>> def pack_hook(x):
...     print("Packing", x)
...     return x
>>>
>>> def unpack_hook(x):
...     print("Unpacking", x)
...     return x
>>>
>>> a = torch.ones(5, requires_grad=True)
>>> b = torch.ones(5, requires_grad=True) * 2
>>> with torch.autograd.graph.saved_tensors_hooks(pack_hook, unpack_hook):
...     y = a * b
Packing tensor([1., 1., 1., 1., 1.], requires_grad=True)
Packing tensor([2., 2., 2., 2., 2.], grad_fn=<MulBackward0>)
>>> y.sum().backward()
Unpacking tensor([1., 1., 1., 1., 1.], requires_grad=True)
Unpacking tensor([2., 2., 2., 2., 2.], grad_fn=<MulBackward0>)

```


!!! warning "警告"

     对任一钩子的输入执行就地操作可能会导致未定义的行为。


!!! warning "警告"

     一次只允许使用一对钩子。当递归嵌套此上下文管理器时，只会应用最里面的一对钩子。


*班级*


 torch.autograd.graph。


 保存_on_cpu


 (*pin_内存



 =
 


 假*，*设备_类型



 =
 


 'cuda'
* ) [[source]](_modules/torch/autograd/graph.html#save_on_cpu)[¶](#torch.autograd.graph.save_on_cpu "此定义的永久链接")


 上下文管理器，在该管理器下，前向传递保存的tensor将存储在 cpu 上，然后检索以用于向后传递。


 在此上下文管理器中执行操作时，前向传递过程中保存在图中的中间结果将被移至 CPU，然后在后向传递需要时复制回原始设备。如果图已在 CPU 上，则不执行tensor复制。


 使用此上下文管理器以计算量换取 GPU 内存使用量(例如，当您的模型在训练期间不适合 GPU 内存时)。


 Parameters


**pin_memory** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` tensor将在打包期间保存到 CPU 固定内存，并在解包期间异步复制到 GPU。默认为“False”。另请参阅[使用固定内存缓冲区](notes/cuda.html#cuda-memory-pinning)。


 例子：


```
>>> a = torch.randn(5, requires_grad=True, device="cuda")
>>> b = torch.randn(5, requires_grad=True, device="cuda")
>>> c = torch.randn(5, requires_grad=True, device="cuda")
>>>
>>> def f(a, b, c):
...     prod_1 = a * b           # a and b are saved on GPU
...     with torch.autograd.graph.save_on_cpu():
...         prod_2 = prod_1 * c  # prod_1 and c are saved on CPU
...     y = prod_2 * a           # prod_2 and a are saved on GPU
...     return y
>>>
>>> y = f(a, b, c)
>>> del a, b, c  # for illustration only
>>> # the content of a, b, and prod_2 are still alive on GPU
>>> # the content of prod_1 and c only live on CPU
>>> y.sum().backward()  # all CPU tensors are moved back to GPU, for backward
>>> # all intermediary tensors are released (deleted) after the call to backward

```


*班级*


 torch.autograd.graph。


 禁用_saved_tensors_hooks


 ( *error_message
* ) [[source]](_modules/torch/autograd/graph.html#disable_saved_tensors_hooks)[¶](#torch.autograd.graph.disable_saved_tensors_hooks "此定义的永久链接")


 禁用保存的tensor默认挂钩功能的上下文管理器。


 如果您要创建的功能无法与保存的tensor默认挂钩一起使用，则非常有用。


 Parameters


**error_message** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 保存tensor时默认钩子当它们被禁用时使用，会引发带有此错误消息的运行时错误。


 例子：


```
>>> message = "saved tensors default hooks are disabled"
>>> with torch.autograd.graph.disable_saved_tensors_hooks(message):
...     # Raises RuntimeError: saved tensors default hooks are disabled
...     with torch.autograd.graph.save_on_cpu():
...         pass

```


*班级*


 torch.autograd.graph。


 注册_multi_grad_hook


 ( *tensors
* , *fn
* ) [[source]](_modules/torch/autograd/graph.html#register_multi_grad_hook)[¶](#torch.autograd.graph.register_multi_grad_hook "此定义的永久链接")


 注册一个多级向后钩子。


 在计算出“tensors”中每个tensor的梯度后，将调用该钩子。如果tensor位于“tensors”中但不是图形的一部分，或者如果不需要tensor来计算为当前“.backward()”或“.grad()”调用指定的任何“输入”的梯度，则此tensor将被忽略，并且钩子不会等待其梯度被计算。


 计算每个不可忽略的tensor的梯度后，将使用这些梯度调用“fn”。对于没有计算梯度的tensor，将传递“None”。


 钩子不应修改其参数。


 该函数返回一个句柄，并使用“handle.remove()”方法来删除钩子。




!!! note "笔记"

    有关此钩子何时执行以及其相对于其他钩子的执行顺序的更多信息，请参阅[向后钩子执行](notes/autograd.html#backward-hooks-execution)。


 例子：


```
>>> import torch
>>>
>>> a = torch.rand(2, 3, requires_grad=True)
>>> b = torch.rand(2, 3, requires_grad=True)
>>> c = a * b
>>> d = a * b
>>>
>>> def fn(grads):
...     print([g is not None for g in grads])
...
>>> torch.autograd.graph.register_multi_grad_hook((a, b, c, d), fn)
>>>
>>> c.sum().backward(retain_graph=True)
[True, True, True, False]
>>> c.sum().backward(inputs=(a,), retain_graph=True)
[True, False, True, False]
>>>

```


*班级*


 torch.autograd.graph。


 allowed_mutation_on_saved_tensors [[source]](_modules/torch/autograd/graph.html#allow_mutation_on_saved_tensors)[¶](#torch.autograd.graph.allow_mutation_on_saved_tensors "此定义的永久链接")


 上下文管理器，允许保存向后的变异tensor


 在这个上下文管理器下，为后向保存的tensor在突变时被克隆，因此在后向过程中仍然可以使用原始版本。通常，改变为向后保存的tensor会导致在向后使用时引发错误。


 为了确保正确的行为，前进和后退都应该在同一个上下文管理器下运行。


 退货


 一个 _AllowMutationOnSavedContext 对象，存储由此上下文管理器管理的状态。该对象可用于调试目的。上下文管理器管理的状态在退出时自动清除。


 例子：


```
>>> import torch
>>> with torch.autograd.graph.allow_mutation_on_saved_tensors():
...     # forward
...     a = torch.ones(2, 3, requires_grad=True)
...     b = a.clone()
...     out = (b**2).sum()
...     b.sin_()
...     # backward
...     out.sum().backward()
...
tensor([[0.8415, 0.8415, 0.8415],
 [0.8415, 0.8415, 0.8415]], grad_fn=<SinBackward0>)

```