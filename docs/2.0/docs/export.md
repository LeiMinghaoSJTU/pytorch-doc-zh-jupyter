# torch.export [¶](#torch-export "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/export>
>
> 原始地址：<https://pytorch.org/docs/stable/export.html>


!!! warning "警告"

     此功能是正在积极开发的原型，将来将会发生重大变化。


## 概述 [¶](#overview "此标题的永久链接")


[`torch.export.export()`](#torch.export.export "torch.export.export") 接受任意 Python 可调用函数([`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ，一个函数或方法)并生成一个跟踪图，以提前(AOT)方式仅表示该函数的tensor计算，随后可以使用不同的输出或序列化来执行。


```
import torch
from torch.export import export

def f(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
    a = torch.sin(x)
    b = torch.cos(y)
    return a + b

example_args = (torch.randn(10, 10), torch.randn(10, 10))

exported_program: torch.export.ExportedProgram = export(
    f, args=example_args
)
print(exported_program)

```



```
ExportedProgram:
    class GraphModule(torch.nn.Module):
        def forward(self, arg0_1: f32[10, 10], arg1_1: f32[10, 10]):
            # code: a = torch.sin(x)
            sin: f32[10, 10] = torch.ops.aten.sin.default(arg0_1);

            # code: b = torch.cos(y)
            cos: f32[10, 10] = torch.ops.aten.cos.default(arg1_1);

            # code: return a + b
            add: f32[10, 10] = torch.ops.aten.add.Tensor(sin, cos);
            return (add,)

    Graph signature: ExportGraphSignature(
        parameters=[],
        buffers=[],
        user_inputs=['arg0_1', 'arg1_1'],
        user_outputs=['add'],
        inputs_to_parameters={},
        inputs_to_buffers={},
        buffers_to_mutate={},
        backward_signature=None,
        assertion_dep_token=None,
    )
    Range constraints: {}
    Equality constraints: []

```


`torch.export` 生成具有以下不变量的干净中间表示 (IR)。有关 IR 的更多规格可在此处找到(即将推出！)。



* **健全性** ：保证是原始程序的健全表示，并保持与原始程序相同的调用约定。 
* **标准化** ：图中没有 Python 语义。原始程序中的子模块被内联以形成一个完全扁平化的计算图。
* **定义的运算符集**：生成的图仅包含一个小的定义的 [Core ATen IR](torch.compiler_ir.html#torch-compiler-ir) opset 和注册的自定义操作符。
* **图属性** ：该图是纯函数式的，这意味着它不包含具有突变或别名等副作用的操作。它不会改变任何中间值、参数或缓冲区。
* **元数据**：图表包含跟踪期间捕获的元数据，例如来自用户代码的堆栈跟踪。


 在底层，“torch.export”利用了以下最新技术：



* **TorchDynamo (torch._dynamo)** 是一个内部 API，它使用称为框架评估 API 的 CPython 功能来安全地跟踪 PyTorch 图。这提供了极大改进的图形捕获体验，完全跟踪 PyTorch 代码所需的重写次数要少得多。
* **AOT Autograd** 提供功能化的 PyTorch 图形，并确保图形分解/降低为小型定义的 Core ATen 运算符集。
* **Torch FX (torch.fx)** 是图形的底层表示，允许灵活的基于 Python 的转换。


### 现有框架 [¶](#existing-frameworks "此标题的永久链接")


[`torch.compile()`](generated/torch.compile.html#torch.compile "torch.compile") 也使用与 `torch.export` 相同的 PT2 堆栈，但略有不同：



* **JIT 与 AOT** ：[`torch.compile()`](generated/torch.compile.html#torch.compile "torch.compile") 是一个 JIT 编译器，但不打算用于生成部署之外的编译工件。
* **部分与完整图形捕获**：当 [`torch.compile()`](generated/torch.compile.html#torch.compile "torch.compile") 遇到无法追踪的部分时一个模型，它将“图形中断”并回退到在急切的 Python 运行时中运行程序。相比之下，“torch.export”旨在获得 PyTorch 模型的完整图形表示，因此当遇到无法追踪的情况时，它会出错。由于 `torch.export` 生成与任何 Python 功能或运行时不相交的全图，因此可以在不同的环境和语言中保存、加载和运行该图。
* **可用性权衡** : 由于 [`torch.compile()` ](generated/torch.compile.html#torch.compile "torch.compile") 每当遇到无法追踪的东西时就能够回退到Python运行时，它更加灵活。相反，“torch.export”将要求用户提供更多信息或重写其代码以使其可追踪。


 与 [`torch.fx.symbolic_trace()`](fx.html#torch.fx.symbolic_trace "torch.fx.symbolic_trace") 相比，`torch.export` 使用在 Python 字节码级别运行的 TorchDynamo 进行跟踪，给出它能够跟踪任意 Python 构造，不受 Python 运算符重载支持的限制。此外，“torch.export”经常保持细粒度的跟踪或元数据，因此tensor形状等条件不会导致跟踪失败。一般来说，“torch.export”预计可以在更多用户程序上工作，并生成较低级别的图表(在“torch.ops.aten”操作员级别)。请注意，用户仍然可以使用 [`torch.fx.symbolic_trace()`](fx.html#torch.fx.symbolic_trace "torch.fx.symbolic_trace") 作为 `torch.export` 之前的预处理步骤。


 与 [`torch.jit.script()`](generated/torch.jit.script.html#torch.jit.script "torch.jit.script") 相比，`torch.export` 不捕获 Python 控制流或数据结构，但它比 TorchScript 支持更多的 Python 语言功能(因为它更容易全面覆盖 Python 字节码)。生成的图更简单，并且仅具有直线控制流(显式控制流运算符除外)。


 与 [`torch.jit.trace()`](generated/torch.jit.trace.html#torch.jit.trace "torch.jit.trace") 相比，`torch.export` 是健全的：它能够跟踪对大小执行整数计算并记录显示特定跟踪对其他输入有效所需的所有辅助条件的代码。


## 导出 PyTorch 模型 [¶](#exporting-a-pytorch-model "永久链接到此标题")


### 示例 [¶](#an-example "此标题的永久链接")


 主要入口点是通过 [`torch.export.export()`](#torch.export.export "torch.export.export") ，它接受一个可调用的 ( [`torch.nn.Module`](generated/torch. nn.Module.html#torch.nn.Module "torch.nn.Module") 、函数或方法)和示例输入，并将计算图捕获到 [`torch.export.ExportedProgram`](#torch.export.导出程序“torch.export.ExportedProgram”)。一个例子：


```
import torch
from torch.export import export

# Simple module for demonstration
class M(torch.nn.Module):
    def __init__(self) -> None:
        super().__init__()
        self.conv = torch.nn.Conv2d(
            in_channels=3, out_channels=16, kernel_size=3, padding=1
        )
        self.relu = torch.nn.ReLU()
        self.maxpool = torch.nn.MaxPool2d(kernel_size=3)

    def forward(self, x: torch.Tensor, *, constant=None) -> torch.Tensor:
        a = self.conv(x)
        a.add_(constant)
        return self.maxpool(self.relu(a))

example_args = (torch.randn(1, 3, 256, 256),)
example_kwargs = {"constant": torch.ones(1, 16, 256, 256)}

exported_program: torch.export.ExportedProgram = export(
    M(), args=example_args, kwargs=example_kwargs
)
print(exported_program)

```



```
ExportedProgram:
    class GraphModule(torch.nn.Module):
        def forward(self, arg0_1: f32[16, 3, 3, 3], arg1_1: f32[16], arg2_1: f32[1, 3, 256, 256], arg3_1: f32[1, 16, 256, 256]):

            # code: a = self.conv(x)
            convolution: f32[1, 16, 256, 256] = torch.ops.aten.convolution.default(
                arg2_1, arg0_1, arg1_1, [1, 1], [1, 1], [1, 1], False, [0, 0], 1
            );

            # code: a.add_(constant)
            add: f32[1, 16, 256, 256] = torch.ops.aten.add.Tensor(convolution, arg3_1);

            # code: return self.maxpool(self.relu(a))
            relu: f32[1, 16, 256, 256] = torch.ops.aten.relu.default(add);
            max_pool2d_with_indices = torch.ops.aten.max_pool2d_with_indices.default(
                relu, [3, 3], [3, 3]
            );
            getitem: f32[1, 16, 85, 85] = max_pool2d_with_indices[0];
            return (getitem,)

    Graph signature: ExportGraphSignature(
        parameters=['L__self___conv.weight', 'L__self___conv.bias'],
        buffers=[],
        user_inputs=['arg2_1', 'arg3_1'],
        user_outputs=['getitem'],
        inputs_to_parameters={
            'arg0_1': 'L__self___conv.weight',
            'arg1_1': 'L__self___conv.bias',
        },
        inputs_to_buffers={},
        buffers_to_mutate={},
        backward_signature=None,
        assertion_dep_token=None,
    )
    Range constraints: {}
    Equality constraints: []

```


 检查 `ExportedProgram` ，我们可以注意到以下内容：



* [`torch.fx.Graph`](fx.html#torch.fx.Graph "torch.fx.Graph") 包含原始程序的计算图，以及原始代码的记录，以便于调试。
* graph 仅包含在 [Core ATen IR](torch.compiler_ir.html#torch-compiler-ir) opset 和自定义运算符中找到的 `torch.ops.aten` 运算符，并且功能齐全，没有任何就地运算符，例如 `torch. add_`.
* 参数(转换的权重和偏差)被提升为图形的输入，导致图形中没有 `get_attr` 节点，该节点之前存在于 [`torch.fx.symbolic\ _trace()`](fx.html#torch.fx.symbolic_trace "torch.fx.symbolic_trace").
* [`torch.export.ExportGraphSignature`](#torch.export.ExportGraphSignature "torch.export.ExportGraphSignature")对输入和输出签名进行建模，并指定哪些输入是参数。
* 图中每个节点生成的tensor的最终形状和数据类型均已注明。例如，“卷积”节点将产生 dtype“torch.float32”和形状(1,16,256,256)的tensor。


### 表达活力 [¶](#expressing-dynamism "永久链接到此标题")


 默认情况下，`torch.export` 将跟踪程序，假设所有输入形状都是 **static** ，并将导出的程序专门针对这些维度。然而，某些维度(例如批次维度)可以是动态的，并且在不同的运行中会有所不同。此类尺寸必须使用 [`torch.export.dynamic_dim()`](#torch.export.dynamic_dim "torch.export.dynamic_dim") API 标记为动态，并传递到 [`torch.export.export() `](#torch.export.export "torch.export.export") 通过 `constraints` 参数。一个例子：


```
import torch
from torch.export import export, dynamic_dim

class M(torch.nn.Module):
    def __init__(self):
        super().__init__()

        self.branch1 = torch.nn.Sequential(
            torch.nn.Linear(64, 32), torch.nn.ReLU()
        )
        self.branch2 = torch.nn.Sequential(
            torch.nn.Linear(128, 64), torch.nn.ReLU()
        )
        self.buffer = torch.ones(32)

    def forward(self, x1, x2):
        out1 = self.branch1(x1)
        out2 = self.branch2(x2)
        return (out1 + self.buffer, out2)

example_args = (torch.randn(32, 64), torch.randn(32, 128))
constraints = [
    # First dimension of each input is a dynamic batch size
    dynamic_dim(example_args[0], 0),
    dynamic_dim(example_args[1], 0),
    # The dynamic batch size between the inputs are equal
    dynamic_dim(example_args[0], 0) == dynamic_dim(example_args[1], 0),
]

exported_program: torch.export.ExportedProgram = export(
  M(), args=example_args, constraints=constraints
)
print(exported_program)

```



```
ExportedProgram:
    class GraphModule(torch.nn.Module):
        def forward(self, arg0_1: f32[32, 64], arg1_1: f32[32], arg2_1: f32[64, 128], arg3_1: f32[64], arg4_1: f32[32], arg5_1: f32[s0, 64], arg6_1: f32[s0, 128]):

            # code: out1 = self.branch1(x1)
            permute: f32[64, 32] = torch.ops.aten.permute.default(arg0_1, [1, 0]);
            addmm: f32[s0, 32] = torch.ops.aten.addmm.default(arg1_1, arg5_1, permute);
            relu: f32[s0, 32] = torch.ops.aten.relu.default(addmm);

            # code: out2 = self.branch2(x2)
            permute_1: f32[128, 64] = torch.ops.aten.permute.default(arg2_1, [1, 0]);
            addmm_1: f32[s0, 64] = torch.ops.aten.addmm.default(arg3_1, arg6_1, permute_1);
            relu_1: f32[s0, 64] = torch.ops.aten.relu.default(addmm_1);  addmm_1 = None

            # code: return (out1 + self.buffer, out2)
            add: f32[s0, 32] = torch.ops.aten.add.Tensor(relu, arg4_1);
            return (add, relu_1)

    Graph signature: ExportGraphSignature(
        parameters=[
            'branch1.0.weight',
            'branch1.0.bias',
            'branch2.0.weight',
            'branch2.0.bias',
        ],
        buffers=['L__self___buffer'],
        user_inputs=['arg5_1', 'arg6_1'],
        user_outputs=['add', 'relu_1'],
        inputs_to_parameters={
            'arg0_1': 'branch1.0.weight',
            'arg1_1': 'branch1.0.bias',
            'arg2_1': 'branch2.0.weight',
            'arg3_1': 'branch2.0.bias',
        },
        inputs_to_buffers={'arg4_1': 'L__self___buffer'},
        buffers_to_mutate={},
        backward_signature=None,
        assertion_dep_token=None,
    )
    Range constraints: {s0: RangeConstraint(min_val=2, max_val=9223372036854775806)}
    Equality constraints: [(InputDim(input_name='arg5_1', dim=0), InputDim(input_name='arg6_1', dim=0))]

```


 一些额外的注意事项：



* 通过 [`torch.export.dynamic_dim()`](#torch.export.dynamic_dim "torch.export.dynamic_dim") API，我们将每个输入的第一个维度指定为动态。查看输入 `arg5_1` 和 `arg6_1` ，它们的符号形状为 (s0, 64) 和 (s0, 128)，而不是 (32, 64) 和 (32, 128) 形状的tensor我们作为示例输入传入。 `s0` 是一个符号，表示该维度可以是一个值范围。
* `exported_program.range_constraints` 描述了图中出现的每个符号的范围。在本例中，我们看到 `s0` 的范围为 [2, inf]。由于这里难以解释的技术原因，它们被假定为不是 0 或 1。这不是一个错误，并不一定意味着导出的程序不适用于维度 0 或 1。请参阅 [0/1 专业化问题](https://docs.google.com/document/d/16VPOa3d-Liikf48teAOmxLc92rgvJdfosIy-yoT38Io/edit?fbclid=IwAR3HNwmmexcitV0pbZm_x1a4ykdXZ9th_eJWK-3hBtVgKnrkmemz6Pm5jRQ#heading=h.ez923tom jvyk) 对此主题进行深入讨论。
* `exported\ _program.equality_constraints` 描述了哪些维度需要相等。由于我们在约束中指定每个参数的第一维是等效的( `dynamic_dim(example_args[0], 0) ==dynamic_dim(example_args[1], 0)` )，我们看到在等式约束中，元组指定“arg5_1”维度 0 和“arg6_1”维度 0 相等。


### 序列化 [¶](#serialization "此标题的永久链接")


 要保存 `ExportedProgram` ，用户可以使用 [`torch.export.save()`](#torch.export.save "torch.export.save") 和 [`torch.export.load()`]( #torch.export.load "torch.export.load") API。约定是使用“.pt2”文件扩展名保存“ExportedProgram”。


 一个例子：


```
import torch
import io

class MyModule(torch.nn.Module):
    def forward(self, x):
        return x + 10

exported_program = torch.export.export(MyModule(), torch.randn(5))

torch.export.save(exported_program, 'exported_program.pt2')
saved_exported_program = torch.export.load('exported_program.pt2')

```


### 专业化 [¶](#specialization "此标题的永久链接")


#### 输入形状 [¶](#input-shapes "此标题的固定链接")


 如前所述，默认情况下，`torch.export` 将跟踪专门针对输入tensor形状的程序，除非通过 [`torch.export.dynamic_dim()`](#torch.export. dynamic_dim“torch.export.dynamic_dim”)API。这意味着，如果存在与形状相关的控制流，“torch.export”将专门针对给定样本输入所采用的分支。例如：


```
import torch
from torch.export import export

def fn(x):
    if x.shape[0] > 5:
        return x + 1
    else:
        return x - 1

example_inputs = (torch.rand(10, 2),)
exported_program = export(fn, example_inputs)
print(exported_program)

```



```
ExportedProgram:
    class GraphModule(torch.nn.Module):
        def forward(self, arg0_1: f32[10, 2]):
            add: f32[10, 2] = torch.ops.aten.add.Tensor(arg0_1, 1);
            return (add,)

```


 ( `x.shape[0] 
> 5` ) 的条件不会出现在 `ExportedProgram` 中，因为示例输入的静态形状为 (10, 2)。由于“torch.export”专门研究输入的静态形状，因此永远不会到达 else 分支(“x 
- 1”)。要根据跟踪图中tensor的形状保留动态分支行为，需要使用 [`torch.export.dynamic_dim()`](#torch.export.dynamic_dim "torch.export.dynamic_dim")指定输入tensor(`x.shape[0]`)的维度是动态的，源代码需要[重写](#data-shape-dependent-control-flow)。


#### 非tensor输入 [¶](#non-tensor-inputs "此标题的永久链接")


`torch.export` 还根据非 `torch.Tensor` 的输入值专门化跟踪图，例如 `int` 、 `float` 、 `bool` 和 `str` 。但是，我们可能会更改这一点在不久的将来，不要专门研究原始类型的输入。


 例如：


```
import torch
from torch.export import export

def fn(x: torch.Tensor, const: int, times: int):
    for i in range(times):
        x = x + const
    return x

example_inputs = (torch.rand(2, 2), 1, 3)
exported_program = export(fn, example_inputs)
print(exported_program)

```



```
ExportedProgram:
    class GraphModule(torch.nn.Module):
        def forward(self, arg0_1: f32[2, 2], arg1_1, arg2_1):
            add: f32[2, 2] = torch.ops.aten.add.Tensor(arg0_1, 1);
            add_1: f32[2, 2] = torch.ops.aten.add.Tensor(add, 1);
            add_2: f32[2, 2] = torch.ops.aten.add.Tensor(add_1, 1);
            return (add_2,)

```


 因为整数是专门化的，所以 `torch.ops.aten.add.Tensor` 操作都是使用内联常量 `1` 计算的，而不是 `arg1_1` 。此外，`for` 循环中使用的 `times` 迭代器也通过 3 次重复的 `torch.ops.aten.add.Tensor` 调用在图中“内联”，并且从未使用输入 `arg2_1`。


## torch.export 的限制 [¶](#limitations-of-torch-export "永久链接到此标题")


### Graph Breaks [¶](#graph-breaks "此标题的永久链接")


 由于“torch.export”是从 PyTorch 程序捕获计算图的一次性过程，因此它最终可能会遇到程序的不可跟踪部分，因为它几乎不可能支持跟踪所有 PyTorch 和 Python 功能。在 `torch.compile` 的情况下，不受支持的操作将导致“graphbreak”，并且不受支持的操作将使用默认的 Python 评估运行。相反，“torch.export”将要求用户提供附加信息或重写部分代码使其可追溯。由于跟踪基于 TorchDynamo，它在 Python 字节码级别进行评估，因此与以前的跟踪框架相比，所需的重写次数将大大减少。


 当遇到图表中断时，[ExportDB](generated/exportdb/index.html#torch-export-db) 是一个很好的资源，可用于了解支持和不支持的程序类型，以及重写程序以使其可追踪的方法。


### 数据/形状相关控制流 [¶](#data-shape-dependent-control-flow "永久链接到此标题")


 当形状没有被专门化时，在数据相关的控制流(`if x.shape[0] 
> 2`)上也可能会遇到图形中断，因为跟踪编译器不可能在不为组合爆炸数量的路径生成代码的情况下进行处理。在这种情况下，用户将需要使用特殊的控制流运算符重写代码(即将推出！)。


### 数据相关访问 [¶](#data-dependent-accesses "此标题的永久链接")


 数据依赖行为，例如使用tensor内部的值构造另一个tensor，或者使用tensor的值切片到另一个tensor，也是跟踪器无法完全确定的。用户需要使用内联约束 API [`torch.export.constrain_as_size()`](#torch.export.constrain_as_size "torch.export.constrain_as_size") 和 [`torch.export.constrain\ _as_value()`](#torch.export.constrain_as_value "torch.export.constrain_as_value") 。


### 缺少运算符的元内核 [¶](#missing-meta-kernels-for-operators "永久链接到此标题")


 跟踪时，所有操作符都需要 META 实现(或“元内核”)。这用于推断该运算符的输入/输出形状。


 请注意，用于为自定义操作注册自定义元内核的官方 API 目前正在开发中。最终 API 正在完善中，您可以参考[此处](https://docs.google.com/document/d/1GgvOe7C8_NVOMLOCwDaYV1mXXyHMXY7ExoewHqooxrs/edit#heading=h.64r4npvq0w0) 的文档。


 如果不幸的是，您的模型使用的 ATen 运算符尚未实现元内核，请提出问题。


## 阅读更多内容 [¶](#read-more "此标题的永久链接")


 导出用户的附加链接



* [在 ATen IR 上编写图形转换](torch.compiler_transformations.html)
* [IR](torch.compiler_ir.html)
* [ExportDB](generated/exportdb/index.html)


 PyTorch 开发人员深入探讨



* [TorchDynamo 深度探索](torch.compiler_deepdive.html)
* [动态形状](torch.compiler_dynamic_shapes.html)
* [假tensor](torch.compiler_fake_tensor.html)


## API 参考 [¶](#module-torch.export "此标题的永久链接")


 火炬.导出。


 出口


 (*f*、*args*、*kwargs



 =
 


 无
* , *** , *约束



 =
 


 无
* ) [[source]](_modules/torch/export.html#export)[¶](#torch.export.export "此定义的永久链接")


[`export()`](#torch.export.export "torch.export.export") 接受任意 Python 可调用对象(nn.Module、函数或方法)并生成仅表示函数的 Tensorcomputation 的跟踪图以提前 (AOT) 方式执行，随后可以使用不同的输出执行或串行化。 Tracedgraph (1) 生成一个标准化运算符集，仅由功能性 [Core ATen Operator Set](https://pytorch.org/docs/stable/ir.html) 和用户指定的自定义运算符组成，(2) 消除了所有 Python控制流和数据结构(某些条件除外)，并且(3)具有一组形状约束，以表明这种标准化和控制流消除对于未来的输入来说是合理的。


**稳健性保证**


 在跟踪时，[`export()`](#torch.export.export "torch.export.export") 会记录用户程序和底层 PyTorch 运算符内核所做的与形状相关的假设。输出 [`ExportedProgram`] (#torch.export.ExportedProgram "torch.export.ExportedProgram") 仅当这些假设成立时才被视为有效。


 追踪过程中有两种类型的假设



* 输入tensor的形状(不是值)。
* 通过“.item()”或直接索引从中间tensor提取的值的范围(下限和上限)。


 所有假设都必须在图形捕获时进行验证，[`export()`](#torch.export.export "torch.export.export") 才能成功。具体来说：



* 输入tensor静态形状的假设会自动验证，无需额外工作。 
* 输入tensor动态形状的假设需要使用 [`dynamic_dim()`](#torch.export.dynamic_dim "torch.export. dynamic_dim") API
* 对中间值范围的假设需要显式的内联约束，构造时使用 [`constrain_as_size()`](#torch.export.constrain_as_size "torch.export.constrain_as_size") 和 `constraint_as\ _value()` API。


 如果任何假设无法得到验证，就会引发致命错误。发生这种情况时，错误消息将包括构建必要的约束以验证假设所需的建议代码，例如 [`export()`](#torch.export.export "torch.export.export") 将建议以下输入约束代码:


```
def specify_constraints(x):
    return [
        # x:
        dynamic_dim(x, 0) <= 5,
    ]

```


 此示例意味着程序要求输入“x”的 dim 0 小于或等于 5 才有效。您可以检查所需的约束，然后将此确切的函数复制到代码中以生成所需的约束并传递到“constraints”参数中。


 参数 
* **f** ( [*Callable*](https://docs.python.org/3/library/typing.html#typing.Callable "(in Python v3.12)") ) – 可调用跟踪。
* **args** ( [*Tuple*](https://docs.python.org/3/library/typing.html#typing.Tuple "(在 Python v3.12 中)")*[
* [ *Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)")*,
* *...
* *]
* ) – 示例位置输入。
* **kwargs** ( [*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(in Python v3.12)")*[
* [*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12)")*[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* [*Any*](https://docs.python.org/3/library/typing. html#typing.Any "(in Python v3.12)")*]
* *]
* ) – 可选示例关键字输入。
* **约束** ( [*可选*](https://docs.python.org /3/library/typing.html#typing.Optional "(Python v3.12)")*[
* [*List*](https://docs.python.org/3/library/typing.html#typing.List "(in Python v3.12)")*[
* [*Constraint*](#torch.export.Constraint "torch.export.Constraint")*]
* *]
* ) – 可选的约束列表动态参数指定其可能的形状范围。默认情况下，输入 torch.Tensors 的形状被假定为静态的。如果输入 torch.Tensoris 期望具有动态形状，请使用 [`dynamic_dim()`](#torch.export.dynamic_dim "torch.export.dynamic_dim") 定义 [`Constraint`](#torch.export.Constraint“torch.export.Constraint”)指定动态和可能的形状范围的对象。有关如何使用它的示例，请参阅 [`dynamic_dim()`](#torch.export.dynamic_dim "torch.export.dynamic_dim") 文档字符串。


 退货


 包含跟踪的可调用对象的 [`ExportedProgram`](#torch.export.ExportedProgram "torch.export.ExportedProgram")。


 Return type


[*ExportedProgram*](#torch.export.ExportedProgram "torch.export.ExportedProgram")


**可接受的输入/输出类型**


 可接受的输入类型(对于 `args` 和 `kwargs` )和输出包括：



* 原始类型，即 `torch.Tensor` 、 `int` 、 `float` 、 `bool` 和 `str` 。
* (嵌套)由 `dict` 、 `list` 、 `tuple` 、 `namedtuple` 组成的数据结构和包含所有上述类型的“OrderedDict”。


 火炬.导出。


 动态_dim


 ( *t
* , *index
* ) [[source]](_modules/torch/export.html#dynamic_dim)[¶](#torch.export.dynamic_dim "此定义的永久链接")


[`dynamic_dim()`](#torch.export.dynamic_dim "torch.export.dynamic_dim") 构造一个 [`Constraint`](#torch.export.Constraint "torch.export.Constraint") 对象，该对象描述tensor“t”的维度“index”的动态性。 [`Constraint`](#torch.export.Constraint "torch.export.Constraint") 对象应传递给 [`export()`](#torch.export.export "torch.export.export" 的 `constraints` 参数”)。


 参数 
* **t** ( [*torch.Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 具有动态维度大小的示例输入tensor
* **index** ( [ *int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 动态维度索引


 退货


 描述形状动态的 [`Constraint`](#torch.export.Constraint "torch.export.Constraint") 对象。它可以传递给 [`export()`](#torch.export.export "torch.export.export") 以便 [`export()`](#torch.export.export "torch.export.export")不假设指定tensor的静态大小，即保持其动态作为符号大小，而不是根据示例跟踪输入的大小进行专门化。


 具体来说， [`dynamic_dim()`](#torch.export.dynamic_dim "torch.export.dynamic_dim") 可用于表达以下类型的动态。



* 维度的大小是动态且无限制的：


```
t0 = torch.rand(2, 3)
t1 = torch.rand(3, 4)

# First dimension of t0 can be dynamic size rather than always being static size 2
constraints = [dynamic_dim(t0, 0)]
ep = export(fn, (t0, t1), constraints=constraints)

```
* Size of a dimension is dynamic with a lower bound:
 


```
t0 = torch.rand(10, 3)
t1 = torch.rand(3, 4)

# First dimension of t0 can be dynamic size with a lower bound of 5 (inclusive)
# Second dimension of t1 can be dynamic size with a lower bound of 2 (exclusive)
constraints = [
    dynamic_dim(t0, 0) >= 5,
    dynamic_dim(t1, 1) > 2,
]
ep = export(fn, (t0, t1), constraints=constraints)

```
* Size of a dimension is dynamic with an upper bound:
 


```
t0 = torch.rand(10, 3)
t1 = torch.rand(3, 4)

# First dimension of t0 can be dynamic size with a upper bound of 16 (inclusive)
# Second dimension of t1 can be dynamic size with a upper bound of 8 (exclusive)
constraints = [
    dynamic_dim(t0, 0) <= 16,
    dynamic_dim(t1, 1) < 8,
]
ep = export(fn, (t0, t1), constraints=constraints)

```
* Size of a dimension is dynamic and it is always equal to size of another dynamic dimension:
 


```
t0 = torch.rand(10, 3)
t1 = torch.rand(3, 4)

# Sizes of second dimension of t0 and first dimension are always equal
constraints = [
    dynamic_dim(t0, 1) == dynamic_dim(t1, 0),
]
ep = export(fn, (t0, t1), constraints=constraints)

```
* Mix and match all types above as long as they do not express conflicting requirements


 火炬.导出。


 限制_为_大小


 ( *符号
* , *最小



 =
 


 无
* , *最大



 =
 


 无
* ) [[source]](_modules/torch/export.html#constrain_as_size)[¶](#torch.export.constrain_as_size "此定义的永久链接")


 提示 [`export()`](#torch.export.export "torch.export.export") 关于表示tensor形状的中间标量值的约束，以便可以正确跟踪后续tensor构造函数，因为许多运算符需要关于尺寸范围的假设。


 参数 
* **symbol** – 应用范围约束的中间标量值(现在仅限 int)。
* **min** ( *可选
* *[
* [*int*](https://docs.python. org/3/library/functions.html#int "(Python v3.12)")*]
* ) – 给定符号的最小可能值(包括)
* **max** ( *可选
* *[
* [
* int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*]
* ) – 给定符号的最大可能值(包含)


 退货


 None
 


 例如，如果不使用 [`constrain_as_size()`](#torch.export.constrain_as_size "torch.export.constrain_as_size") 给出 [`export()`](#torch.export.export "torch.export.export") 关于形状范围的提示：


```
def fn(x):
    d = x.max().item()
    return torch.ones(v)

```


[`export()`](#torch.export.export "torch.export.export") 会给出以下错误：


```
torch._dynamo.exc.Unsupported: guard on data-dependent symbolic int/float

```


 假设 `d` 的实际范围可以在 [3, 10] 之间，您可以添加对 [`constrain_as_size()`](#torch.export.constrain_as_size "torch.export.constrain_as_size") 的调用源代码是这样的：


```
def fn(x):
    d = x.max().item()
    torch.export.constrain_as_size(d, min=3, max=10)
    return torch.ones(d)

```


 通过附加提示，[`export()`](#torch.export.export "torch.export.export") 将能够通过采取 `else` 分支来正确跟踪程序，从而产生下图：


```
graph():
    %arg0_1 := placeholder[target=arg0_1]

    # d = x.max().item()
    %max_1 := call_functiontarget=torch.ops.aten.max.default)
    %_local_scalar_dense := call_functiontarget=torch.ops.aten._local_scalar_dense.default)

    # Asserting 3 <= d <= 10
    %ge := call_functiontarget=operator.ge)
    %scalar_tensor := call_functiontarget=torch.ops.aten.scalar_tensor.default)
    %_assert_async := call_functiontarget=torch.ops.aten._assert_async.msg)
    %le := call_functiontarget=operator.le)
    %scalar_tensor_1 := call_functiontarget=torch.ops.aten.scalar_tensor.default)
    %_assert_async_1 := call_functiontarget=torch.ops.aten._assert_async.msg)
    %sym_constrain_range_for_size := call_functiontarget=torch.ops.aten.sym_constrain_range_for_size.default, kwargs = {min: 3, max: 10})

    # Constructing new tensor with d
    %full := call_functiontarget=torch.ops.aten.full.default,
        kwargs = {dtype: torch.float32, layout: torch.strided, device: cpu, pin_memory: False})

    ......

```


!!! warning "警告"

     如果您的尺寸是动态的，请勿测试尺寸是否等于 0 或 1，这些将默默地报告 false 并被绕过


 火炬.导出。


 约束为值


 ( *符号
* , *最小



 =
 


 无
* , *最大



 =
 


 无
* ) [[source]](_modules/torch/export.html#constrain_as_value)[¶](#torch.export.constrain_as_value "此定义的永久链接")


 提示 [`export()`](#torch.export.export "torch.export.export") 关于中间标量值的约束，以便可以准确地跟踪检查上述标量值范围的后续分支行为。


!!! warning "警告"

     (请注意，如果中间标量值将像大小一样使用，包括作为大小参数传递到tensor工厂或视图，请调用 [`constrain_as_size()`](#torch.export.constrain_as_size "torch.export.constrain_as_size") 代替。)


 参数 
* **symbol** – 应用范围约束的中间标量值(现在仅限 int)。
* **min** ( *可选
* *[
* [*int*](https://docs.python. org/3/library/functions.html#int "(Python v3.12)")*]
* ) – 给定符号的最小可能值(包括)
* **max** ( *可选
* *[
* [
* int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*]
* ) – 给定符号的最大可能值(包含)


 退货


 None
 


 例如，以下程序无法被正确追踪：


```
def fn(x):
    v = x.max().item()
    if v > 1024:
        return x
    else:
        return x * 2

```


`v` 是一个与数据相关的值，假设其范围为 (-inf, inf)。 [`export()`](#torch.export.export "torch.export.export") 关于采用哪个分支的提示将无法确定跟踪的分支决策是否正确。因此 [`export()`](#torch.export.export "torch.export.export") 会给出以下错误：


```
torch._dynamo.exc.UserError: Consider annotating your code using
torch.export.constrain_as_size() or torch.export().constrain_as_value() APIs.
It appears that you're trying to get a value out of symbolic int/float whose value
is data-dependent (and thus we do not know the true value.)  The expression we were
trying to evaluate is f0 > 1024 (unhinted: f0 > 1024).

```


 假设 `v` 的实际范围可以在 [10, 200] 之间，您可以添加对 [`constrain_as_value()`](#torch.export.constrain_as_value "torch.export.constrain_as_value") 的调用源代码是这样的：


```
def fn(x):
    v = x.max().item()

    # Give export() a hint
    torch.export.constrain_as_value(v, min=10, max=200)

    if v > 1024:
        return x
    else:
        return x * 2

```


 通过附加提示，[`export()`](#torch.export.export "torch.export.export") 将能够通过采取 `else` 分支来正确跟踪程序，从而产生下图：


```
graph():
    %arg0_1 := placeholder[target=arg0_1]

    # v = x.max().item()
    %max_1 := call_functiontarget=torch.ops.aten.max.default)
    %_local_scalar_dense := call_functiontarget=torch.ops.aten._local_scalar_dense.default)

    # Asserting 10 <= v <= 200
    %ge := call_functiontarget=operator.ge)
    %scalar_tensor := call_functiontarget=torch.ops.aten.scalar_tensor.default)
    %_assert_async := call_functiontarget=torch.ops.aten._assert_async.msg)
    %le := call_functiontarget=operator.le)
    %scalar_tensor_1 := call_functiontarget=torch.ops.aten.scalar_tensor.default)
    %_assert_async_1 := call_functiontarget=torch.ops.aten._assert_async.msg)
    %sym_constrain_range := call_functiontarget=torch.ops.aten.sym_constrain_range.default, kwargs = {min: 10, max: 200})

    # Always taking `else` branch to multiply elements `x` by 2 due to hints above
    %mul := call_functiontarget=torch.ops.aten.mul.Tensor, kwargs = {})
    return (mul,)

```


 火炬.导出。



 save
 


 ( *ep
* 、 *f
* 、 *** 、 *extra_files



 =
 


 无
* , *opset_version



 =
 


 无
* ) [[source]](_modules/torch/export.html#save)[¶](#torch.export.save "此定义的永久链接")


!!! warning "警告"

     在积极开发中，保存的文件可能无法在较新版本的 PyTorch 中使用。


 将 [`ExportedProgram`](#torch.export.ExportedProgram "torch.export.ExportedProgram") 保存到类似文件的对象。然后可以使用 Python API [`torch.export.load`](#torch.export.load "torch.export.load") 加载它。


 参数 
* **ep** ( [*ExportedProgram*](#torch.export.ExportedProgram "torch.export.ExportedProgram") ) – 要保存的导出程序。
* **f** ( *Union
* *[
* [ *str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* [*pathlib.Path*](https://docs.python.org/3/library/pathlib.html#pathlib.Path "(Python v3.12)")*,
* [*io.BytesIO*](https://docs.python.org/3/library /io.html#io.BytesIO "(in Python v3.12)") ) – 类文件对象(必须实现写入和刷新)或包含文件名的字符串。
* **extra_files** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* 
* Any
* *]
* *]
* ) – 从文件名映射到将作为 f.
* **opset_version** 的一部分存储的内容 ( *可选
* *[
* *Dict
* *[
* [*str*]( https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*]
* *]
* ) – opset 名称到此 opset 版本的映射


 例子：


```
import torch
import io

class MyModule(torch.nn.Module):
    def forward(self, x):
        return x + 10

ep = torch.export.export(MyModule(), torch.randn(5))

# Save to file
torch.export.save(ep, 'exported_program.pt2')

# Save to io.BytesIO buffer
buffer = io.BytesIO()
torch.export.save(ep, buffer)

# Save with extra files
extra_files = {'foo.txt': b'bar'}
torch.export.save(ep, 'exported_program.pt2', extra_files=extra_files)

```


 火炬.导出。



 load
 


 ( *f
* , *** , *额外_文件



 =
 


 无
* , *预期_opset_version



 =
 


 无
* ) [[source]](_modules/torch/export.html#load)[¶](#torch.export.load "此定义的永久链接")


!!! warning "警告"

     在积极开发中，保存的文件可能无法在较新版本的 PyTorch 中使用。


 加载之前使用 [`torch.export.save`](#torch.export.save "torch.export.save") 保存的 [`ExportedProgram`](#torch.export.ExportedProgram "torch.export.ExportedProgram") 。


 参数 
* **ep** ( [*ExportedProgram*](#torch.export.ExportedProgram "torch.export.ExportedProgram") ) – 要保存的导出程序。
* **f** ( *Union
* *[
* [ *str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* [*pathlib.Path*](https://docs.python.org/3/library/pathlib.html#pathlib.Path "(Python v3.12)")*,
* [*io.BytesIO*](https://docs.python.org/3/library /io.html#io.BytesIO "(in Python v3.12)") ) – 类文件对象(必须实现写入和刷新)或包含文件名的字符串。
* **extra_files** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* 
* Any
* *]
* *]
* ) – 将加载此映射中给出的额外文件名，并将其内容存储在提供的映射中。
* **expected_opset_version** ( *可选
* *[
* *Dict
* 
* [
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*,
* [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")*]
* *]
* ) – opset 名称到预期 opset 版本的映射


 退货


 一个 [`ExportedProgram`](#torch.export.ExportedProgram "torch.export.ExportedProgram") 对象


 Return type


[*ExportedProgram*](#torch.export.ExportedProgram "torch.export.ExportedProgram")


 例子：


```
import torch
import io

# Load ExportedProgram from file
ep = torch.export.load('exported_program.pt2')

# Load ExportedProgram from io.BytesIO object
with open('exported_program.pt2', 'rb') as f:
    buffer = io.BytesIO(f.read())
buffer.seek(0)
ep = torch.export.load(buffer)

# Load with extra files.
extra_files = {'foo.txt': ''}  # values will be replaced with data
ep = torch.export.load('exported_program.pt2', extra_files=extra_files)
print(extra_files['foo.txt'])

```


*班级*


 火炬.导出。


 约束


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/export.html#Constraint)[¶](#torch.export.Constraint "此定义的永久链接")


!!! warning "警告"

     不要直接构造 [`Constraint`](#torch.export.Constraint "torch.export.Constraint")，使用 [`dynamic_dim()`](#torch.export.dynamic_dim "torch.export.dynamic_dim")反而。


 这表示对输入tensor维度的约束，例如，要求它们是完全多态的或在某个范围内。


*班级*


 火炬.导出。


 导出程序


 ( *root
* 、 *graph
* 、 *graph_signature
* 、 *call_spec
* 、 *state_dict
* 、 *range_constraints
* 、 *equality_constraints
* 、 *module_call_graph
* 、 *example\ _输入



 =
 


 无
* ) [[source]](_modules/torch/export.html#ExportedProgram)[¶](#torch.export.ExportedProgram "此定义的永久链接")


 来自 [`export()`](#torch.export.export "torch.export.export") 的程序包。它包含表示tensor计算的 [`torch.fx.Graph`](fx.html#torch.fx.Graph "torch.fx.Graph")、包含所有提升参数和缓冲区的tensor值的 state_dict 以及各种元数据。


 您可以使用相同的调用约定来调用 ExportedProgram，就像 [`export()`](#torch.export.export "torch.export.export") 跟踪的原始可调用程序一样。


 要对图形执行转换，请使用.module 属性访问 [`torch.fx.GraphModule`](fx.html#torch.fx.GraphModule "torch.fx.GraphModule") 。然后，您可以使用 [FX 转换](https://pytorch.org/docs/stable/fx.html#writing-transformations) 重写图表。之后，您只需再次使用 [`export()`](#torch.export.export "torch.export.export") 即可构造正确的 ExportedProgram。


 模块


 ( ) [[source]](_modules/torch/export.html#ExportedProgram.module)[¶](#torch.export.ExportedProgram.module "此定义的永久链接")


 返回一个自包含的 GraphModule，其中内联了所有参数/缓冲区。


 Return type


[*模块*](generated/torch.nn.Module.html#torch.nn.Module“torch.nn.modules.module.Module”)


*班级*


 火炬.导出。


 导出向后签名


 ( *梯度_到_参数



 :
 


 Dict
 


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ]
* , *梯度_到_用户_输入



 :
 


 Dict
 


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ]
* , *损失_输出



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str"(Python v3.12)")*) [[source]](_modules/torch/export.html#ExportBackwardSignature )[¶](#torch.export.ExportBackwardSignature"此定义的永久链接")


*班级*


 火炬.导出。


 导出图签名


 (*参数*、*缓冲区*、*用户输入*、*用户输出*、*输入_到_参数*、*输入_到_缓冲区*、*缓冲区_到_变异*、*向后_签名
* , *断言_dep_token



 =
 


 无
* ) [[source]](_modules/torch/export.html#ExportGraphSignature)[¶](#torch.export.ExportGraphSignature "此定义的永久链接")


[`ExportGraphSignature`](#torch.export.ExportGraphSignature "torch.export.ExportGraphSignature") 对 Export Graph 的输入/输出签名进行建模，这是一个具有更强不变性保证的 fx.Graph。


 导出图功能正常，不会通过“getattr”节点访问图中的参数或缓冲区等“状态”。相反，[`export()`](#torch.export.export "torch.export.export") 保证参数和缓冲区作为输入从图表中取出。同样，缓冲区的任何突变都不包含在图表中或者，突变缓冲区的更新值被建模为导出图的附加输出。


 所有输入和输出的顺序为：


```
Inputs = [*parameters_buffers, *flattened_user_inputs]
Outputs = [*mutated_inputs, *flattened_user_outputs]

```


 例如如果导出以下模块：


```
class CustomModule(nn.Module):
    def __init__(self):
        super(CustomModule, self).__init__()

        # Define a parameter
        self.my_parameter = nn.Parameter(torch.tensor(2.0))

        # Define two buffers
        self.register_buffer('my_buffer1', torch.tensor(3.0))
        self.register_buffer('my_buffer2', torch.tensor(4.0))

    def forward(self, x1, x2):
        # Use the parameter, buffers, and both inputs in the forward method
        output = (x1 + self.my_parameter) * self.my_buffer1 + x2 * self.my_buffer2

        # Mutate one of the buffers (e.g., increment it by 1)
        self.my_buffer2.add_(1.0) # In-place addition

        return output

```


 结果图将是：


```
graph():
    %arg0_1 := placeholder[target=arg0_1]
    %arg1_1 := placeholder[target=arg1_1]
    %arg2_1 := placeholder[target=arg2_1]
    %arg3_1 := placeholder[target=arg3_1]
    %arg4_1 := placeholder[target=arg4_1]
    %add_tensor := call_functiontarget=torch.ops.aten.add.Tensor, kwargs = {})
    %mul_tensor := call_functiontarget=torch.ops.aten.mul.Tensor, kwargs = {})
    %mul_tensor_1 := call_functiontarget=torch.ops.aten.mul.Tensor, kwargs = {})
    %add_tensor_1 := call_functiontarget=torch.ops.aten.add.Tensor, kwargs = {})
    %add_tensor_2 := call_functiontarget=torch.ops.aten.add.Tensor, kwargs = {})
    return (add_tensor_2, add_tensor_1)

```


 结果 ExportGraphSignature 将是：


```
ExportGraphSignature(
    # Indicates that there is one parameter named `my_parameter`
    parameters=['L__self___my_parameter'],

    # Indicates that there are two buffers, `my_buffer1` and `my_buffer2`
    buffers=['L__self___my_buffer1', 'L__self___my_buffer2'],

    # Indicates that the nodes `arg3_1` and `arg4_1` in produced graph map to
    # original user inputs, ie. x1 and x2
    user_inputs=['arg3_1', 'arg4_1'],

    # Indicates that the node `add_tensor_1` maps to output of original program
    user_outputs=['add_tensor_1'],

    # Indicates that there is one parameter (self.my_parameter) captured,
    # its name is now mangled to be `L__self___my_parameter`, which is now
    # represented by node `arg0_1` in the graph.
    inputs_to_parameters={'arg0_1': 'L__self___my_parameter'},

    # Indicates that there are two buffers (self.my_buffer1, self.my_buffer2) captured,
    # their name are now mangled to be `L__self___my_my_buffer1` and `L__self___my_buffer2`.
    # They are now represented by nodes `arg1_1` and `arg2_1` in the graph.
    inputs_to_buffers={'arg1_1': 'L__self___my_buffer1', 'arg2_1': 'L__self___my_buffer2'},

    # Indicates that one buffer named `L__self___my_buffer2` is mutated during execution,
    # its new value is output from the graph represented by the node named `add_tensor_2`
    buffers_to_mutate={'add_tensor_2': 'L__self___my_buffer2'},

    # Backward graph not captured
    backward_signature=None,

    # Work in progress feature, please ignore now.
    assertion_dep_token=None
)

```


*班级*


 火炬.导出。


 参数类型


 ( *value
* ) [[source]](_modules/torch/export.html#ArgumentKind)[¶](#torch.export.ArgumentKind "此定义的永久链接")


 一个枚举。


*班级*


 火炬.导出。


 参数规范


 ( *种类



 :
 


[torch.export.ArgumentKind](#torch.export.ArgumentKind "torch.export.ArgumentKind")
* , *值



 :
 


 Any
* ) [[source]](_modules/torch/export.html#ArgumentSpec)[¶](#torch.export.ArgumentSpec "此定义的永久链接")


*班级*


 火炬.导出。


 模块调用签名


 (*输入



 :
 


 List
 


 [ [torch.export.ArgumentSpec](#torch.export.ArgumentSpec "torch.export.ArgumentSpec")


 ]
* , *输出



 :
 


 List
 


 [ [torch.export.ArgumentSpec](#torch.export.ArgumentSpec "torch.export.ArgumentSpec")


 ]
* , *在_spec中



 :
 


 torch.utils._pytree.TreeSpec
* , *out_spec



 :
 


 torch.utils._pytree.TreeSpec
* ) [[source]](_modules/torch/export.html#ModuleCallSignature)[¶](#torch.export.ModuleCallSignature "此定义的永久链接")


*班级*


 火炬.导出。


 模块调用入口


 ( *fqn



 :
 


[str](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")
* , *签名



 :
 


 Union
 


 [ [torch.export.ModuleCallSignature](#torch.export.ModuleCallSignature "torch.export.ModuleCallSignature")


 ,
 


 无类型


 ]
 



 =
 


 无
* ) [[source]](_modules/torch/export.html#ModuleCallEntry)[¶](#torch.export.ModuleCallEntry "此定义的永久链接")