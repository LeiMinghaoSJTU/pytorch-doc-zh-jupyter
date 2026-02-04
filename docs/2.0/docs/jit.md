# TorchScript [¶](#torchscript "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/jit>
>
> 原始地址：<https://pytorch.org/docs/stable/jit.html>



* [TorchScript 语言参考](jit_language_reference_v2.html)



* [创建 TorchScript 代码](#creating-torchscript-code)
* [混合跟踪和脚本](#mixing-tracing-and-scripting)
* [TorchScript 语言](#torchscript-language)
* [内置函数和模块](#内置函数和模块)



+ [PyTorch 函数和模块](#pytorch-functions-and-modules) 
+ [Python 函数和模块](#python-functions-and-modules) 
+ [Python 语言参考比较](#python-language-reference-comparison )
* [调试](#调试)



+ [禁用 JIT 调试](#disable-jit-for-debugging) 
+ [检查代码](#inspecting-code) 
+ [解释图形](#interpreting-graphs) 
+ [跟踪器](#tracer)
* [频繁提出的问题](#常见问题)
* [已知问题](#known-issues)
* [附录](#appendix)



+ [迁移到 PyTorch 1.2 递归脚本 API](#migration-to-pytorch-1-2-recursive-scripting-api) 
+ [Fusion 后端](#fusion-backends) 
+ [参考资料](#references)


 TorchScript 是一种从 PyTorch 代码创建可序列化和可优化模型的方法。任何 TorchScript 程序都可以从 Python 进程保存并加载到不存在 Python 依赖项的进程中。


 我们提供工具将模型从纯 Python 程序逐步转换为可以独立于 Python 运行的 TorchScript 程序，例如在独立的 C++ 程序中。这使得可以使用 Python 中熟悉的工具在 PyTorch 中训练模型，然后导出模型通过 TorchScript 到生产环境，其中 Python 程序可能因性能和多线程原因而处于不利地位。


 有关 TorchScript 的简单介绍，请参阅 [TorchScript 简介](https://pytorch.org/tutorials/beginner/Intro_to_TorchScript_tutorial.html) 教程。


 有关将 PyTorch 模型转换为 TorchScript 并在 C++ 中运行的端到端示例，请参阅[在 C++ 中加载 PyTorch 模型](https://pytorch.org/tutorials/advanced/cpp_export.html)教程。


## [创建 TorchScript 代码](#id4) [¶](#creating-torchscript-code "永久链接到此标题")


|  |  |
| --- | --- |
| [`脚本`](generated/torch.jit.script.html#torch.jit.script“torch.jit.script”)|编写函数或 nn.Module 脚本将检查源代码，使用 TorchScript 编译器将其编译为 TorchScript 代码，并返回 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule " torch.jit.ScriptModule") 或 [`ScriptFunction`](generated/torch.jit.ScriptFunction.html#torch.jit.ScriptFunction "torch.jit.ScriptFunction") 。 |
| [`trace`](generated/torch.jit.trace.html#torch.jit.trace "torch.jit.trace") |跟踪函数并返回将使用即时编译进行优化的可执行文件或 [`ScriptFunction`](generated/torch.jit.ScriptFunction.html#torch.jit.ScriptFunction "torch.jit.ScriptFunction")。 |
| [`script_if_tracing`](generated/torch.jit.script_if_tracing.html#torch.jit.script_if_tracing "torch.jit.script_if_tracing") |在跟踪期间首次调用“fn”时编译它。 |
| [`trace_module`](generated/torch.jit.trace_module.html#torch.jit.trace_module "torch.jit.trace_module") |跟踪模块并返回一个可执行文件 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule")，该可执行文件将使用即时编译进行优化。 |
| [`fork`](generated/torch.jit.fork.html#torch.jit.fork "torch.jit.fork") |创建一个异步任务执行 func 以及对此执行结果值的引用。 |
| [`等待`](generated/torch.jit.wait.html#torch.jit.wait“torch.jit.wait”)|强制完成 torch.jit.Future[T] 异步任务，并返回任务结果。 |
| [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule“torch.jit.ScriptModule”) | C++ `torch::jit::Module` 的包装器。 |
| [`ScriptFunction`](generated/torch.jit.ScriptFunction.html#torch.jit.ScriptFunction "torch.jit.ScriptFunction") |功能上相当于 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") ，但表示单个函数并且没有任何属性或参数。 |
| [`freeze`](generated/torch.jit.freeze.html#torch.jit.freeze "torch.jit.freeze") |冻结 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 将克隆它并尝试将克隆模块的子模块、参数和属性作为常量内联TorchScript IR 图。 |
| [`optimize_for_inference`](generated/torch.jit.optimize_for_inference.html#torch.jit.optimize_for_inference "torch.jit.optimize_for_inference") |执行一组优化遍来优化模型以进行推理。 |
| [`启用_onednn_fusion`](generated/torch.jit.enable_onednn_fusion.html#torch.jit.enable_onednn_fusion“torch.jit.enable_onednn_fusion”)|根据参数启用或禁用 onednn JIT 融合。 |
| [`onednn_fusion_enabled`](generated/torch.jit.onednn_fusion_enabled.html#torch.jit.onednn_fusion_enabled "torch.jit.onednn_fusion_enabled") |返回是否启用 onednn JIT fusion |
| [`set_fusion_strategy`](generated/torch.jit.set_fusion_strategy.html#torch.jit.set_fusion_strategy "torch.jit.set_fusion_strategy") |设置融合过程中可以发生的专业化的类型和数量。 |
| [`strict_fusion`](generated/torch.jit.strict_fusion.html#torch.jit.strict_fusion "torch.jit.strict_fusion") |如果不是所有节点都在推理中融合，或者在训练中符号区分，则此类错误。 |
| [`save`](generated/torch.jit.save.html#torch.jit.save "torch.jit.save") |保存该模块的离线版本以供在单独的进程中使用。 |
| [`load`](generated/torch.jit.load.html#torch.jit.load "torch.jit.load") |加载 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 或 [`ScriptFunction`](generated/torch.jit.ScriptFunction.html#torch. jit.ScriptFunction "torch.jit.ScriptFunction") 先前使用 [`torch.jit.save`](generated/torch.jit.save.html#torch.jit.save "torch.jit.save") |
| 保存


[`ignore`](generated/torch.jit.ignore.html#torch.jit.ignore“torch.jit.ignore”)|此装饰器向编译器指示应忽略函数或方法并将其保留为 Python 函数。 |
| [`未使用`](generated/torch.jit.unused.html#torch.jit.unused "torch.jit.unused") |此装饰器向编译器指示应忽略函数或方法并用引发异常来替换。 |
| [`isinstance`](generated/torch.jit.isinstance.html#torch.jit.isinstance“torch.jit.isinstance”) |此函数提供 TorchScript 中的容器类型细化。 |
| [`属性`](generated/torch.jit.Attribute.html#torch.jit.Attribute“torch.jit.Attribute”) |该方法是一个返回 value 的传递函数，主要用于向 TorchScript 编译器指示左侧表达式是类型为 type 的类实例属性。 |
| [`注释`](generated/torch.jit.annotate.html#torch.jit.annotate "torch.jit.annotate") |该方法是一个传递函数，返回_value，用于提示TorchScript编译器_value的类型。 |


## [混合跟踪和脚本](#id5) [¶](#mixing-tracing-and-scripting "永久链接到此标题")


 在许多情况下，跟踪或脚本编写是将模型转换为 TorchScript 的更简单方法。可以组合跟踪和脚本编写以满足模型某一部分的特定要求。


 脚本化函数可以调用跟踪函数。当您需要在简单的前馈模型周围使用控制流时，这特别有用。例如，序列到序列模型的波束搜索通常用脚本编写，但可以调用使用跟踪生成的编码器模块。


 示例(在脚本中调用跟踪函数)：


```
import torch

def foo(x, y):
    return 2 * x + y

traced_foo = torch.jit.trace(foo, (torch.rand(3), torch.rand(3)))

@torch.jit.script
def bar(x):
    return traced_foo(x, x)

```


 跟踪函数可以调用脚本函数。当模型的一小部分需要一些控制流时，即使模型的大部分只是前馈网络，这也很有用。由跟踪函数调用的脚本函数内部的控制流被正确保留。


 示例(在跟踪函数中调用脚本函数)：


```
import torch

@torch.jit.script
def foo(x, y):
    if x.max() > y.max():
        r = x
    else:
        r = y
    return r


def bar(x, y, z):
    return foo(x, y) + z

traced_bar = torch.jit.trace(bar, (torch.rand(3), torch.rand(3), torch.rand(3)))

```


 这种组合也适用于 nn.Module ，它可用于使用跟踪生成子模块，该子模块可从脚本模块的方法调用。


 示例(使用跟踪模块)：


```
import torch
import torchvision

class MyScriptModule(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.means = torch.nn.Parameter(torch.tensor([103.939, 116.779, 123.68])
                                        .resize_(1, 3, 1, 1))
        self.resnet = torch.jit.trace(torchvision.models.resnet18(),
                                      torch.rand(1, 3, 224, 224))

    def forward(self, input):
        return self.resnet(input - self.means)

my_script_module = torch.jit.script(MyScriptModule())

```


## [TorchScript 语言](#id6) [¶](#torchscript-language "此标题的永久链接")


 TorchScript 是 Python 的静态类型子集，因此许多 Python 功能直接适用于 TorchScript。有关详细信息，请参阅完整的 [TorchScript 语言参考](jit_language_reference.html#language-reference)。


## [内置函数和模块](#id7) [¶](#built-in-functions-and-modules "永久链接到此标题")


 TorchScript 支持使用大多数 PyTorch 函数和许多 Python 内置函数。有关受支持函数的完整参考，请参阅 [TorchScript 内置函数](jit_builtin_functions.html#builtin-functions)。


### [PyTorch 函数和模块](#id8) [¶](#pytorch-functions-and-modules "此标题的永久链接")


 TorchScript 支持 PyTorch 提供的tensor和神经网络函数的子集。 TorchScript 支持 Tensor 上的大多数方法以及“torch”命名空间中的函数、“torch.nn.function”中的所有函数以及“torch.nn”中的大多数模块。


 有关不支持的 PyTorch 函数和模块的列表，请参阅 [TorchScript 不支持的 PyTorch 构造](jit_unsupported.html#jit-unsupported)。


### [Python 函数和模块](#id9) [¶](#python-functions-and-modules "此标题的永久链接")


 TorchScript 支持许多 Python 的[内置函数](https://docs.python.org/3/library/functions.html)。[`math`](https://docs.python.org/3/library/math.html#module-math "(in Python v3.12)") 模块也受支持(有关详细信息，请参阅 [math 模块](jit_builtin_functions.html#math-module))，但不支持其他 Python 模块(支持内置或第三方)。


### [Python 语言参考比较](#id10) [¶](#python-language-reference-comparison "此标题的永久链接")


 有关支持的 Python 功能的完整列表，请参阅 [Python 语言参考覆盖范围](jit_python_reference.html#python-language-reference) 。


## [调试](#id11) [¶](#debugging "此标题的永久链接")


### [禁用 JIT 调试](#id12) [¶](#disable-jit-for-debugging "永久链接到此标题")


 PYTORCH_JIT [¶](#envvar-PYTORCH_JIT "此定义的永久链接")


 设置环境变量“PYTORCH_JIT=0”将禁用所有脚本和跟踪注释。如果您的 TorchScript 模型之一存在难以调试的错误，您可以使用此标志强制所有内容都使用 nativePython 运行。由于此标志禁用了 TorchScript(脚本和跟踪)，因此您可以使用“pdb”等工具来调试模型代码。例如：


```
@torch.jit.script
def scripted_fn(x : torch.Tensor):
    for i in range(12):
        x = x + x
    return x

def fn(x):
    x = torch.neg(x)
    import pdb; pdb.set_trace()
    return scripted_fn(x)

traced_fn = torch.jit.trace(fn, (torch.rand(4, 5),))
traced_fn(torch.rand(3, 4))

```


 除非我们调用 [`@torch.jit.script`](generated/torch.jit.script.html#torch.jit.script "torch.jit.script") 函数，否则使用 pdb 调试此脚本是有效的。我们可以全局禁用JIT，这样我们就可以像普通Python一样调用[`​​@torch.jit.script`](generated/torch.jit.script.html#torch.jit.script "torch.jit.script")函数函数而不编译它。如果上面的脚本名为“disable_jit_example.py”，我们可以像这样调用它：


```
$ PYTORCH_JIT=0 python disable_jit_example.py

```


 我们将能够像普通的 Python 函数一样单步执行 [`@torch.jit.script`](generated/torch.jit.script.html#torch.jit.script "torch.jit.script") 函数。要禁用特定函数的 TorchScript 编译器，请参阅 [`@torch.jit.ignore`](generated/torch.jit.ignore.html#torch.jit.ignore "torch.jit.ignore") 。


### [检查代码](#id13) [¶](#inspecting-code "永久链接到此标题")


 TorchScript 为所有 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 实例提供了一个代码漂亮的打印机。这个漂亮的打印机将脚本方法的代码解释为有效的 Python 语法。例如：


```
@torch.jit.script
def foo(len):
    # type: (int) -> torch.Tensor
    rv = torch.zeros(3, 4)
    for i in range(len):
        if i < 10:
            rv = rv - 1.0
        else:
            rv = rv + 1.0
    return rv

print(foo.code)

```


 具有单个 `forward` 方法的 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 将具有一个属性 `code` ，您可以使用它来检查 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 的代码。如果 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 有多个方法，您需要访问方法本身而不是模块上的 `.code`。我们可以通过访问 `.foo.code` 来检查 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 上名为 `foo` 的方法的代码.上面的示例产生以下输出：


```
def foo(len: int) -> Tensor:
    rv = torch.zeros([3, 4], dtype=None, layout=None, device=None, pin_memory=None)
    rv0 = rv
    for i in range(len):
        if torch.lt(i, 10):
            rv1 = torch.sub(rv0, 1., 1)
        else:
            rv1 = torch.add(rv0, 1., 1)
        rv0 = rv1
    return rv0

```


 这是 TorchScript 对“forward”方法代码的编译。您可以使用它来确保 TorchScript(跟踪或脚本)正确捕获您的模型代码。


### [解释图表](#id14) [¶](#interpreting-graphs "此标题的永久链接")


 TorchScript 还具有比代码漂亮打印机更低级别的表示形式，以 IR 图的形式。


 TorchScript 使用静态单赋值 (SSA) 中间表示 (IR) 来表示计算。这种格式的指令由ATen(PyTorch的C++后端)运算符和其他原始运算符组成，包括循环和条件的控制流运算符。举个例子：


```
@torch.jit.script
def foo(len):
    # type: (int) -> torch.Tensor
    rv = torch.zeros(3, 4)
    for i in range(len):
        if i < 10:
            rv = rv - 1.0
        else:
            rv = rv + 1.0
    return rv

print(foo.graph)

```


关于“forward”方法查找，“graph”遵循[检查代码](#inspecting-code)部分中描述的相同规则。


 上面的示例脚本生成图表：


```
graph(%len.1 : int):
  %24 : int = prim::Constant[value=1]()
  %17 : bool = prim::Constant[value=1]() # test.py:10:5
  %12 : bool? = prim::Constant()
  %10 : Device? = prim::Constant()
  %6 : int? = prim::Constant()
  %1 : int = prim::Constant[value=3]() # test.py:9:22
  %2 : int = prim::Constant[value=4]() # test.py:9:25
  %20 : int = prim::Constant[value=10]() # test.py:11:16
  %23 : float = prim::Constant[value=1]() # test.py:12:23
  %4 : int[] = prim::ListConstruct(%1, %2)
  %rv.1 : Tensor = aten::zeros(%4, %6, %6, %10, %12) # test.py:9:10
  %rv : Tensor = prim::Loop(%len.1, %17, %rv.1) # test.py:10:5
    block0(%i.1 : int, %rv.14 : Tensor):
      %21 : bool = aten::lt(%i.1, %20) # test.py:11:12
      %rv.13 : Tensor = prim::If(%21) # test.py:11:9
        block0():
          %rv.3 : Tensor = aten::sub(%rv.14, %23, %24) # test.py:12:18
          -> (%rv.3)
        block1():
          %rv.6 : Tensor = aten::add(%rv.14, %23, %24) # test.py:14:18
          -> (%rv.6)
      -> (%17, %rv.13)
  return (%rv)

```


 以指令 `%rv.1 : Tensor = aten::zeros(%4, %6, %6, %10, %12) # test.py:9:10` 为例。



* `%rv.1 : Tensor` 意味着我们将输出分配给一个名为 `rv.1` 的(唯一)值，该值是 `Tensor` 类型，并且我们不知道它的具体形状。
* `aten:: Zeros` 是运算符(相当于 `torch.zeros` )，输入列表 `(%4, %6, %6, %10, %12)` 指定范围内的哪些值应作为输入传递。像 `aten::zeros` 这样的内置函数的模式可以在 [Builtin Functions](#builtin-functions) 中找到。
* `# test.py:9:10` 是生成的原始源文件中的位置该指令。在本例中，它是一个名为 test.py 的文件，位于第 9 行第 10 字符处。


 请注意，运算符还可以具有关联的“块”，即“prim::Loop”和“prim::If”运算符。在图形打印输出中，这些运算符的格式被格式化以反映其等效的源代码形式，以方便调试。


 可以如图所示检查图形，以确认 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 描述的计算在自动和手动中都是正确的时尚，如下所述。


### [Tracer](#id15) [¶](#tracer "此标题的永久链接")


#### 跟踪边缘情况 [¶](#tracing-edge-cases "永久链接到此标题")


 在某些边缘情况下，给定 Python 函数/模块的跟踪不能代表底层代码。这些情况可以包括：



* 跟踪依赖于输入的控制流(例如tensor形状)
* 跟踪tensor视图的就地操作(例如赋值左侧的索引)


 请注意，这些案例实际上在未来可能是可追踪的。


#### 自动跟踪检查 [¶](#automatic-trace-checking "永久链接到此标题")


 自动捕获跟踪中的许多错误的一种方法是使用“torch.jit.trace()” API 上的“check_inputs”。 `check_inputs` 接受一个输入元组列表，这些输入元组将用于重新跟踪计算并验证结果。例如：


```
def loop_in_traced_fn(x):
    result = x[0]
    for i in range(x.size(0)):
        result = result * x[i]
    return result

inputs = (torch.rand(3, 4, 5),)
check_inputs = [(torch.rand(4, 5, 6),), (torch.rand(2, 3, 4),)]

traced = torch.jit.trace(loop_in_traced_fn, inputs, check_inputs=check_inputs)

```


 为我们提供以下诊断信息：


```
ERROR: Graphs differed across invocations!
Graph diff:

            graph(%x : Tensor) {
            %1 : int = prim::Constant[value=0]()
            %2 : int = prim::Constant[value=0]()
            %result.1 : Tensor = aten::select(%x, %1, %2)
            %4 : int = prim::Constant[value=0]()
            %5 : int = prim::Constant[value=0]()
            %6 : Tensor = aten::select(%x, %4, %5)
            %result.2 : Tensor = aten::mul(%result.1, %6)
            %8 : int = prim::Constant[value=0]()
            %9 : int = prim::Constant[value=1]()
            %10 : Tensor = aten::select(%x, %8, %9)
        -   %result : Tensor = aten::mul(%result.2, %10)
        +   %result.3 : Tensor = aten::mul(%result.2, %10)
        ?          ++
            %12 : int = prim::Constant[value=0]()
            %13 : int = prim::Constant[value=2]()
            %14 : Tensor = aten::select(%x, %12, %13)
        +   %result : Tensor = aten::mul(%result.3, %14)
        +   %16 : int = prim::Constant[value=0]()
        +   %17 : int = prim::Constant[value=3]()
        +   %18 : Tensor = aten::select(%x, %16, %17)
        -   %15 : Tensor = aten::mul(%result, %14)
        ?     ^                                 ^
        +   %19 : Tensor = aten::mul(%result, %18)
        ?     ^                                 ^
        -   return (%15);
        ?             ^
        +   return (%19);
        ?             ^
            }

```


 此消息向我们表明，我们第一次跟踪它时和使用“check_inputs”跟踪它时的计算有所不同。事实上，“loop_in_traced_fn”主体内的循环取决于输入“x”的形状，因此当我们尝试另一个具有不同形状的“x”时，跟踪会有所不同。


 在这种情况下，可以使用 [`torch.jit.script()`](generated/torch.jit.script.html#torch.jit.script "torch.jit.script") 捕获这样的依赖于数据的控制流反而：


```
def fn(x):
    result = x[0]
    for i in range(x.size(0)):
        result = result * x[i]
    return result

inputs = (torch.rand(3, 4, 5),)
check_inputs = [(torch.rand(4, 5, 6),), (torch.rand(2, 3, 4),)]

scripted_fn = torch.jit.script(fn)
print(scripted_fn.graph)
#print(str(scripted_fn.graph).strip())

for input_tuple in [inputs] + check_inputs:
    torch.testing.assert_close(fn(*input_tuple), scripted_fn(*input_tuple))

```


 其产生：


```
graph(%x : Tensor) {
    %5 : bool = prim::Constant[value=1]()
    %1 : int = prim::Constant[value=0]()
    %result.1 : Tensor = aten::select(%x, %1, %1)
    %4 : int = aten::size(%x, %1)
    %result : Tensor = prim::Loop(%4, %5, %result.1)
    block0(%i : int, %7 : Tensor) {
        %10 : Tensor = aten::select(%x, %1, %i)
        %result.2 : Tensor = aten::mul(%7, %10)
        -> (%5, %result.2)
    }
    return (%result);
}

```


#### 跟踪器警告 [¶](#tracer-warnings"此标题的永久链接")


 跟踪器会针对跟踪计算中的几种有问题的模式发出警告。例如，跟踪一个包含tensor切片(视图)上的就地赋值的函数：


```
def fill_row_zero(x):
    x[0] = torch.rand(*x.shape[1:2])
    return x

traced = torch.jit.trace(fill_row_zero, (torch.rand(3, 4),))
print(traced.graph)

```


 产生几个警告和一个仅返回输入的图表：


```
fill_row_zero.py:4: TracerWarning: There are 2 live references to the data region being modified when tracing in-place operator copy_ (possibly due to an assignment). This might cause the trace to be incorrect, because all other views that also reference this data will not reflect this change in the trace! On the other hand, if all other views use the same memory chunk, but are disjoint (e.g. are outputs of torch.split), this might still be safe.
    x[0] = torch.rand(*x.shape[1:2])
fill_row_zero.py:6: TracerWarning: Output nr 1. of the traced function does not match the corresponding output of the Python function. Detailed error:
Not within tolerance rtol=1e-05 atol=1e-05 at input[0, 1] (0.09115803241729736 vs. 0.6782537698745728) and 3 other locations (33.00%)
    traced = torch.jit.trace(fill_row_zero, (torch.rand(3, 4),))
graph(%0 : Float(3, 4)) {
    return (%0);
}

```


 我们可以通过修改代码以不使用就地更新来解决此问题，而是使用“torch.cat”异地构建结果tensor：


```
def fill_row_zero(x):
    x = torch.cat((torch.rand(1, *x.shape[1:2]), x[1:2]), dim=0)
    return x

traced = torch.jit.trace(fill_row_zero, (torch.rand(3, 4),))
print(traced.graph)

```


## [常见问题](#id16) [¶](#frequently-asked-questions "此标题的永久链接")


 问：我想在 GPU 上训练模型并在 CPU 上进行推理。最佳实践是什么？



> 
> 
> 
>  First convert your model from GPU to CPU and then save it, like so:
>  
> 
> 
> 
> 
> 
> ```
> cpu_model = gpu_model.cpu()
> sample_input_cpu = sample_input_gpu.cpu()
> traced_cpu = torch.jit.trace(cpu_model, sample_input_cpu)
> torch.jit.save(traced_cpu, "cpu.pt")
> 
> traced_gpu = torch.jit.trace(gpu_model, sample_input_gpu)
> torch.jit.save(traced_gpu, "gpu.pt")
> 
> # ... later, when using the model:
> 
> if use_gpu:
>   model = torch.jit.load("gpu.pt")
> else:
>   model = torch.jit.load("cpu.pt")
> 
> model(input)
> 
> ```
> 
> 
> 
> 
>  This is recommended because the tracer may witness tensor creation on a
> specific device, so casting an already-loaded model may have unexpected
> effects. Casting the model
>  *before* 
>  saving it ensures that the tracer has
> the correct device information.
>  
> 
> 
> 
> 


 问：如何在 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 上存储属性？



> 
> 
> 
>  Say we have a model like:
>  
> 
> 
> 
> 
> 
> ```
> import torch
> 
> class Model(torch.nn.Module):
>     def __init__(self):
>         super().__init__()
>         self.x = 2
> 
>     def forward(self):
>         return self.x
> 
> m = torch.jit.script(Model())
> 
> ```
> 
> 
> 
> 
>  If
>  `Model`
>  is instantiated it will result in a compilation error
> since the compiler doesn’t know about
>  `x`
>  . There are 4 ways to inform the
> compiler of attributes on
>  `ScriptModule`
>  :
>  
> 
> 
> 
>  1.
>  `nn.Parameter`
>  - Values wrapped in
>  `nn.Parameter`
>  will work as they
> do on
>  `nn.Module`
>  s
>  
> 
> 
> 
>  2.
>  `register_buffer`
>  - Values wrapped in
>  `register_buffer`
>  will work as
> they do on
>  `nn.Module`
>  s. This is equivalent to an attribute (see 4) of type
>  `Tensor`
>  .
>  
> 
> 
> 
>  3. Constants - Annotating a class member as
>  `Final`
>  (or adding it to a list called
>  `__constants__`
>  at the class definition level) will mark the contained names
> as constants. Constants are saved directly in the code of the model. See
>  
>  builtin-constants
>  
>  for details.
>  
> 
> 
> 
>  4. Attributes - Values that are a
>  
>  supported type
>  
>  can be added as mutable
> attributes. Most types can be inferred but some may need to be specified, see
>  
>  module attributes
>  
>  for details.
>  
> 
> 
> 
> 


 问：我想跟踪模块的方法，但我不断收到此错误：


`运行时错误：无法插入需要 grad 作为常量的tensor。考虑将其作为参数或输入，或分离梯度`



> 
> 
> 
> 这个错误通常意味着您正在跟踪的方法使用了模块的参数，并且您正在传递模块的方法而不是模块实例(例如
> `my_module_instance.forward`
> vs
> `my_module _instance`
> ).
> 
> 
> 
> 
> 
> 
> 
> 
> 
* 调用
> 
> `trace`
> 
> 使用模块的方法捕获模块参数(可能需要梯度)作为
> 
> **常量** 
> 
>.
> 
> 
* 另一方面，使用模块的实例调用 >> `trace`>> (例如 >> `my_module`>> )创建一个新模块并正确地将参数复制到新模块中，以便它们可以在需要时累积梯度.
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 
> 要跟踪模块上的特定方法，请参阅
> [`torch.jit.trace_module`](generated/torch.jit.trace_module.html#torch.jit. trace_module "torch.jit.trace_module")
> 
> 
> 
> >


## [已知问题](#id17) [¶](#known-issues "此标题的固定链接")


 如果您将“Sequential”与 TorchScript 一起使用，则某些“Sequential”子模块的输入可能会被错误地推断为“Tensor”，即使它们有其他注释。规范的解决方案是子类化“nn.Sequential”并使用正确键入的输入重新声明“forward”。


## [附录](#id18) [¶](#appendix "此标题的永久链接")


### [迁移到 PyTorch 1.2 递归脚本 API](#id19) [¶](#migration-to-pytorch-1-2-recursive-scripting-api "永久链接到此标题")


 本节详细介绍 PyTorch 1.2 中对 TorchScript 的更改。如果您是 TorchScript 新手，您可以跳过本节。 PyTorch 1.2 中的 TorchScript API 有两个主要变化。


 1. [`torch.jit.script`](generated/torch.jit.script.html#torch.jit.script "torch.jit.script") 现在将尝试递归编译遇到的函数、方法和类。一旦你调用了 `torch.jit.script` ，编译就是“选择退出”，而不是“选择加入”。


 2. `torch.jit.script(nn_module_instance)` 现在是创建 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule 的首选方法") s，而不是继承自 `torch.jit.ScriptModule` 。这些更改结合起来提供了一个更简单、更易于使用的 API，用于将 `nn.Module` 转换为 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") s，准备在非Python环境中优化和执行。


 新的用法如下所示：


```
import torch
import torch.nn as nn
import torch.nn.functional as F

class Model(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 20, 5)
        self.conv2 = nn.Conv2d(20, 20, 5)

    def forward(self, x):
        x = F.relu(self.conv1(x))
        return F.relu(self.conv2(x))

my_model = Model()
my_scripted_model = torch.jit.script(my_model)

```



* 模块的 `forward` 是默认编译的。从 `forward` 调用的方法按照它们在 `forward` 中使用的顺序进行延迟编译。
* 要编译不是从 `forward` 调用的除 `forward` 之外的方法，请添加 `@torch.jit.export` 。*要阻止编译器编译方法，请添加 [`@torch.jit.ignore`](generated/torch.jit.ignore.html#torch.jit.ignore "torch.jit.ignore") 或 [`@torch.ignore. jit.unused`](generated/torch.jit.unused.html#torch.jit.unused "torch.jit.unused") 。 `@ignore` 将 
* 方法保留为对 python 的调用，而 `@unused` 将其替换为异常。 `@ignored` 无法导出； `@unused` 可以。
* 大多数属性类型都可以推断，因此 `torch.jit.Attribute` 不是必需的。对于空容器类型，使用 [PEP 526-style](https://www.python.org/dev/peps/pep-0526/#class-and-instance-variable-annotations) 类注释来注释其类型。
* 常量可以用 `Final` 类注释进行标记，而不是将成员的名称添加到 `__constants__` 中。
* 可使用 Python 3 类型提示代替 `torch.jit.annotate`


 由于这些更改，以下项目被视为已弃用，不应出现在新代码中： 
* `@torch.jit.script_method` 装饰器
* 从 `torch.jit.ScriptModule` 继承的类
* `torch.jit.Attribute` 包装类
* `__constants__` 数组
* `torch.jit.annotate` 函数


#### 模块 [¶](#modules"此标题的永久链接")


!!! warning "警告"

     PyTorch 1.2 中的 [`@torch.jit.ignore`](generated/torch.jit.ignore.html#torch.jit.ignore "torch.jit.ignore") 注释的行为发生了变化。在 PyTorch 1.2 之前，@ignore 装饰器用于使函数或方法可从导出的代码中调用。要恢复此功能，请使用“@torch.jit.unused()”。 `@torch.jit.ignore` 现在相当于 `@torch.jit.ignore(drop=False)` 。请参阅 [`@torch.jit.ignore`](generated/torch.jit.ignore.html#torch.jit.ignore "torch.jit.ignore") 和 [`@torch.jit.unused`](generated/torch.jit.unused.html#torch.jit.unused "torch.jit.unused")了解详细信息。


 当传递给 [`torch.jit.script`](generated/torch.jit.script.html#torch.jit.script "torch.jit.script") 函数时， `torch.nn.Module` 的数据复制到 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 并且 TorchScript 编译器编译该模块。默认情况下编译该模块的 `forward`。从 `forward` 调用的方法按照它们在 `forward` 中使用的顺序进行延迟编译，以及任何 `@torch.jit.export` 方法。


 火炬.jit。


 出口


 ( *fn
* ) [[source]](_modules/torch/_jit_internal.html#export)[¶](#torch.jit.export "此定义的永久链接")


 此装饰器指示 `nn.Module` 上的方法用作 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 的入口点并且应该被编译。


`forward` 隐式地被假定为入口点，所以它不需要这个装饰器。从 `forward` 调用的函数和方法是按照编译器看到的那样进行编译的，所以它们也不需要这个装饰器。


 示例(在方法上使用“@torch.jit.export”)：


```
import torch
import torch.nn as nn

class MyModule(nn.Module):
    def implicitly_compiled_method(self, x):
        return x + 99

    # `forward` is implicitly decorated with `@torch.jit.export`,
    # so adding it here would have no effect
    def forward(self, x):
        return x + 10

    @torch.jit.export
    def another_forward(self, x):
        # When the compiler sees this call, it will compile
        # `implicitly_compiled_method`
        return self.implicitly_compiled_method(x)

    def unused_method(self, x):
        return x - 20

# `m` will contain compiled methods:
# `forward`
# `another_forward`
# `implicitly_compiled_method`
# `unused_method` will not be compiled since it was not called from
# any compiled methods and wasn't decorated with `@torch.jit.export`
m = torch.jit.script(MyModule())

```


#### 函数 [¶](#functions "此标题的永久链接")


 函数变化不大，可以用 [`@torch.jit.ignore`](generated/torch.jit.ignore.html#torch.jit.ignore "torch.jit.ignore") 或 [`torch.jit.unused`](generated/torch.jit.unused.html#torch.jit.unused "torch.jit.unused") 如果需要的话。


```
# Same behavior as pre-PyTorch 1.2
@torch.jit.script
def some_fn():
    return 2

# Marks a function as ignored, if nothing
# ever calls it then this has no effect
@torch.jit.ignore
def some_fn2():
    return 2

# As with ignore, if nothing calls it then it has no effect.
# If it is called in script it is replaced with an exception.
@torch.jit.unused
def some_fn3():
  import pdb; pdb.set_trace()
  return 4

# Doesn't do anything, this function is already
# the main entry point
@torch.jit.export
def some_fn4():
    return 2

```


#### TorchScript 类 [¶](#torchscript-classes "此标题的永久链接")


!!! warning "警告"

     TorchScript 类支持是实验性的。目前它最适合简单的类似记录的类型(想想带有附加方法的“NamedTuple”)。


 用户定义的 [TorchScript 类](torchscript-class) 中的所有内容默认都会导出，函数可以用 [`@torch.jit.ignore`](generated/torch.jit.ignore.html#torch.jit.ignore " 修饰torch.jit.ignore”)如果需要的话。


#### 属性 [¶](#attributes "此标题的永久链接")


 TorchScript 编译器需要知道模块属性的类型。大多数类型可以从成员的值推断出来。空列表和字典不能推断其类型，并且必须使用 [PEP 526 样式](https://www.python.org/dev/peps/pep-0526/#class-and-instance-variable-annotations ) 类注释。如果无法推断类型且未显式注释，则不会将其作为属性添加到生成的 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch. jit.ScriptModule")


 旧 API：


```
from typing import Dict
import torch

class MyModule(torch.jit.ScriptModule):
    def __init__(self):
        super().__init__()
        self.my_dict = torch.jit.Attribute({}, Dict[str, int])
        self.my_int = torch.jit.Attribute(20, int)

m = MyModule()

```


 新的API：


```
from typing import Dict

class MyModule(torch.nn.Module):
    my_dict: Dict[str, int]

    def __init__(self):
        super().__init__()
        # This type cannot be inferred and must be specified
        self.my_dict = {}

        # The attribute type here is inferred to be `int`
        self.my_int = 20

    def forward(self):
        pass

m = torch.jit.script(MyModule())

```


#### 常量 [¶](#constants "此标题的永久链接")


 `Final` 类型构造函数可用于将成员标记为常量。如果成员未标记为常量，它们将作为属性复制到生成的 [`ScriptModule`](generated/torch.jit.ScriptModule.html#torch.jit.ScriptModule "torch.jit.ScriptModule") 中。如果已知该值是固定的，则使用“Final”可以提供优化的机会，并提供额外的类型安全性。


 旧 API：


```
class MyModule(torch.jit.ScriptModule):
    __constants__ = ['my_constant']

    def __init__(self):
        super().__init__()
        self.my_constant = 2

    def forward(self):
        pass
m = MyModule()

```


 新的API：


```
from typing import Final

class MyModule(torch.nn.Module):

    my_constant: Final[int]

    def __init__(self):
        super().__init__()
        self.my_constant = 2

    def forward(self):
        pass

m = torch.jit.script(MyModule())

```


#### 变量 [¶](#variables "此标题的永久链接")


 假设容器具有“Tensor”类型并且是非可选的(有关更多信息，请参阅默认类型)。以前，“torch.jit.annotate”用于告诉 TorchScript 编译器类型应该是什么。现在支持 Python 3 样式类型提示。


```
import torch
from typing import Dict, Optional

@torch.jit.script
def make_dict(flag: bool):
    x: Dict[str, int] = {}
    x['hi'] = 2
    b: Optional[int] = None
    if flag:
        b = 2
    return x, b

```


### [Fusion Backends](#id20) [¶](#fusion-backends "此标题的永久链接")


 有几个融合后端可用于优化 TorchScript 执行。 CPU上默认的融合器是NNC，它可以对CPU和GPU进行融合。 GPU 上的默认融合器是 NVFuser，它支持更广泛的运算符，并已证明生成的内核具有更高的吞吐量。有关使用和调试的更多详细信息，请参阅 [NVFuser 文档](https://github.com/pytorch/pytorch/blob/main/torch/csrc/jit/codegen/cuda/README.md)。


### [参考文献](#id21) [¶](#references"此标题的永久链接")



* [Python 语言参考覆盖范围](jit_python_reference.html)
* [TorchScript 不支持的 PyTorch 结构](jit_unsupported.html)