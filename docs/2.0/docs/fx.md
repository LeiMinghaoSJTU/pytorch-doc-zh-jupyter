# torch.fx [¶](#torch-fx "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/fx>
>
> 原始地址：<https://pytorch.org/docs/stable/fx.html>


## 概述 [¶](#module-torch.fx "此标题的永久链接")


 FX 是开发人员用来转换“nn.Module”实例的工具包。 FX 由三个主要组件组成：**符号跟踪器、**中间表示**和**Python 代码生成**。演示这些组件的实际操作：


```
import torch
# Simple module for demonstration
class MyModule(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.param = torch.nn.Parameter(torch.rand(3, 4))
        self.linear = torch.nn.Linear(4, 5)

    def forward(self, x):
        return self.linear(x + self.param).clamp(min=0.0, max=1.0)

module = MyModule()

from torch.fx import symbolic_trace
# Symbolic tracing frontend - captures the semantics of the module
symbolic_traced : torch.fx.GraphModule = symbolic_trace(module)

# High-level intermediate representation (IR) - Graph representation
print(symbolic_traced.graph)
"""
graph():
 %x : [num_users=1] = placeholder[target=x]
 %param : [num_users=1] = get_attr[target=param]
 %add : [num_users=1] = call_functiontarget=operator.add, kwargs = {})
 %linear : [num_users=1] = call_moduletarget=linear, kwargs = {})
 %clamp : [num_users=1] = call_methodtarget=clamp, kwargs = {min: 0.0, max: 1.0})
 return clamp
"""

# Code generation - valid Python code
print(symbolic_traced.code)
"""
def forward(self, x):
 param = self.param
 add = x + param; x = param = None
 linear = self.linear(add); add = None
 clamp = linear.clamp(min = 0.0, max = 1.0); linear = None
 return clamp
"""

```


 **符号跟踪器**执行Python代码的“符号执行”。它通过代码提供称为代理的虚假值。记录对这些代理的操作。有关符号跟踪的更多信息可以在 [`symbolic_trace()`](#torch.fx.symbolic_trace "torch.fx.symbolic_trace") 和 [`Tracer`](#torch.fx.Tracer "torch.fx" 中找到.Tracer”)文档。


 **中间表示**是符号跟踪期间记录的操作的容器。它由代表函数输入、调用点(函数、方法或 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module ") 实例)，并返回值。有关 IR 的更多信息可以在 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 文档中找到。 TheIR 是应用转换的格式。


**Python 代码生成** 使 FX 成为 Python 到 Python(或模块到模块)转换工具包。对于每个图 IR，我们可以创建与图的语义匹配的有效 Python 代码。此功能包含在 [`GraphModule`](#torch.fx.GraphModule "torch.fx.GraphModule") 中，它是一个 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") 实例，包含一个 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 以及从 Graph 生成的 `forward` 方法。


 总而言之，这个组件管道(符号跟踪 -
> 中间表示 -
> 转换 -
> Python 代码生成)构成了 FX 的 Python 到 Python 转换管道。此外，这些组件可以单独使用。例如，可以单独使用符号跟踪来捕获某种形式的代码以用于分析(而不是转换)目的。代码生成可用于以编程方式生成模型，例如从配置文件生成模型。 FX 有很多用途！


 可以在 [examples](https://github.com/pytorch/examples/tree/master/fx) 存储库中找到几个示例转换。


## 书写转换 [¶](#writing-transformations "此标题的固定链接")
- -


 什么是 FX 变换？本质上，它是一个看起来像这样的函数。


```
import torch
import torch.fx

def transform(m: nn.Module,
              tracer_class : type = torch.fx.Tracer) -> torch.nn.Module:
    # Step 1: Acquire a Graph representing the code in `m`

    # NOTE: torch.fx.symbolic_trace is a wrapper around a call to
    # fx.Tracer.trace and constructing a GraphModule. We'll
    # split that out in our transform to allow the caller to
    # customize tracing behavior.
    graph : torch.fx.Graph = tracer_class().trace(m)

    # Step 2: Modify this Graph or create a new one
    graph = ...

    # Step 3: Construct a Module to return
    return torch.fx.GraphModule(m, graph)

```


 您的转换将接受 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ，获取 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 从中进行一些修改，并返回一个新的 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.模块”).您应该认为 FXtransform 返回的 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") 与常规 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") – 你可以将它传递给 anotherFX 转换，你可以将它传递给 TorchScript，或者你可以运行它。确保 FX 转换的输入和输出是 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") 将允许可组合性。




!!! note "笔记"

    也可以修改现有的 [`GraphModule`](#torch.fx.GraphModule "torch.fx.GraphModule") 而不是创建一个新的，如下所示：


```
import torch
import torch.fx

def transform(m : nn.Module) -> nn.Module:
    gm : torch.fx.GraphModule = torch.fx.symbolic_trace(m)

    # Modify gm.graph
    # <...>

    # Recompile the forward() method of `gm` from its Graph
    gm.recompile()

    return gm

```


 请注意，您必须调用 [`GraphModule.recompile()`](#torch.fx.GraphModule.recompile "torch.fx.GraphModule.recompile") 以使 `GraphModule` 上生成的 `forward()` 方法同步修改后的 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 。


 假设您传入了一个已被追踪到 [`Graph `](#torch.fx.Graph "torch.fx.Graph") ，现在有两种主要方法可以用来构建新的 [`Graph`](#torch.fx.Graph "torch.fx.Graph" )。


### 图快速入门 [¶](#a-quick-primer-on-graphs "此标题的固定链接")


 图语义的完整处理可以在 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 文档中找到，但我们将在这里介绍基础知识。 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 是一种数据结构，表示 [`GraphModule`](#torch.fx.GraphModule "torch.fx.GraphModule") 上的方法。这需要的信息是：



* 方法的输入是什么？
* 方法内部运行的操作是什么？
* 方法的输出(即返回)值是什么？


 所有这三个概念都用 [`Node`](#torch.fx.Node "torch.fx.Node") 实例来表示。让我们通过一个简短的示例来看看我们的意思：


```
import torch
import torch.fx

class MyModule(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.param = torch.nn.Parameter(torch.rand(3, 4))
        self.linear = torch.nn.Linear(4, 5)

    def forward(self, x):
        return torch.topk(torch.sum(
            self.linear(x + self.linear.weight).relu(), dim=-1), 3)

m = MyModule()
gm = torch.fx.symbolic_trace(m)

gm.graph.print_tabular()

```


 这里我们定义一个模块`MyModule`用于演示目的，实例化它，象征性地跟踪它，然后调用[`Graph.print_tabular()`](#torch.fx.Graph.print_tabular "torch.fx.Graph.print_tabular ") 方法打印出一个表格，显示此 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 的节点：


> 	> 	> 	> 	> | 	>  opcode	>   | 	>  name	>   | 	>  target	>   | 	>  args	>   | 	>  kwargs	>   |	> | --- | --- | --- | --- | --- |	> | 	>  placeholder	>   | 	>  x	>   | 	>  x	>   | 	>  ()	>   | 	>  {}	>   |	> | 	>  get_attr	>   | 	>  linear_weight	>   | 	>  linear.weight	>   | 	>  ()	>   | 	>  {}	>   |	> | 	>  call_function	>   | 	>  add_1	>   | 	>  <built-in function add>	>   | 	>  (x, linear_weight)	>   | 	>  {}	>   |	> | 	>  call_module	>   | 	>  linear_1	>   | 	>  linear	>   | 	>  (add_1,)	>   | 	>  {}	>   |	> | 	>  call_method	>   | 	>  relu_1	>   | 	>  relu	>   | 	>  (linear_1,)	>   | 	>  {}	>   |	> | 	>  call_function	>   | 	>  sum_1	>   | 	>  <built-in method sum …>	>   | 	>  (relu_1,)	>   | 	>  {‘dim’: -1}	>   |	> | 	>  call_function	>   | 	>  topk_1	>   | 	>  <built-in method topk …>	>   | 	>  (sum_1, 3)	>   | 	>  {}	>   |	> | 	>  output	>   | 	>  output	>   | 	>  output	>   | 	>  (topk_1,)	>   | 	>  {}	>   |	> 	> 	> 	>


 我们可以利用这些信息来回答我们上面提出的问题。



* 该方法的输入是什么？在 FX 中，方法输入是通过特殊的“占位符”节点指定的。在本例中，我们有一个“占位符”节点，其“目标”为“x”，这意味着我们有一个名为 x 的(非自身)参数。
* 该方法内有哪些操作？ `get_attr` 、 `call_function` 、 `call_module` 和 `call_method` 节点代表方法中的操作。所有这些语义的完整处理可以在 [`Node`](#torch.fx.Node "torch.fx.Node") 文档中找到。
* 该方法的返回值是什么？ [`Graph`](#torch.fx.Graph "torch.fx.Graph") 中的返回值由特殊的 `output` 节点指定。


 鉴于我们现在了解了如何在 FX 中表示代码的基础知识，我们现在可以探索如何编辑 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 。


### 图形操作 [¶](#graph-manipulation "此标题的永久链接")


#### 直接图操作 [¶](#direct-graph-manipulation "此标题的永久链接")


 构建这个新的 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 的一种方法是直接操作旧的。为了帮助实现这一点，我们可以简单地获取从符号跟踪获得的 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 并对其进行修改。例如，假设我们希望用 [`torch.mul()`](generated/torch.mul.html#torch.mul "torch.mul") 调用。


```
import torch
import torch.fx

# Sample module
class M(torch.nn.Module):
    def forward(self, x, y):
        return torch.add(x, y)

def transform(m: torch.nn.Module,
              tracer_class : type = fx.Tracer) -> torch.nn.Module:
    graph : fx.Graph = tracer_class().trace(m)
    # FX represents its Graph as an ordered list of
    # nodes, so we can iterate through them.
    for node in graph.nodes:
        # Checks if we're calling a function (i.e:
        # torch.add)
        if node.op == 'call_function':
            # The target attribute is the function
            # that call_function calls.
            if node.target == torch.add:
                node.target = torch.mul

    graph.lint() # Does some checks to make sure the
                 # Graph is well-formed.

    return fx.GraphModule(m, graph)

```


 我们还可以进行更多涉及的[`Graph`](#torch.fx.Graph "torch.fx.Graph")重写，例如删除或追加节点。为了帮助进行这些转换，FX 具有用于转换图形的实用函数，可以在 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 文档中找到。下面是使用这些 API 附加 `torch.relu()` 调用的示例。


```
# Specifies the insertion point. Any nodes added to the
# Graph within this scope will be inserted after `node`
with traced.graph.inserting_after(node):
    # Insert a new `call_function` node calling `torch.relu`
    new_node = traced.graph.call_function(
        torch.relu, args=(node,))

    # We want all places that used the value of `node` to
    # now use that value after the `relu` call we've added.
    # We use the `replace_all_uses_with` API to do this.
    node.replace_all_uses_with(new_node)

```


 对于仅包含替换的简单转换，您还可以使用[子图重写器。](https://github.com/pytorch/pytorch/blob/main/torch/fx/subgraph_rewriter.py)


#### 使用replace_pattern() 重写子图 [¶](#subgraph-rewriting-with-replace-pattern "永久链接到此标题")


 FX 还在直接图形操作之上提供了另一个级别的自动化。 [`replace_pattern()`](#torch.fx.replace_pattern "torch.fx.replace_pattern") API 本质上是一个“查找/替换”工具编辑 [`Graph`](#torch.fx.Graph "torch.fx.Graph") s。它允许您指定“模式”和“替换”函数，它将跟踪这些函数，在“模式”图中查找操作组的实例，并将这些实例替换为“替换”图的副本。这可以帮助极大地自动化繁琐的图形操作代码，随着转换变得更加复杂，这些代码可能会变得笨拙。


#### 图形操作示例 [¶](#graph-manipulation-examples "此标题的永久链接")



* [替换 oneop](https://github.com/pytorch/examples/blob/master/fx/replace_op.py)
* [Conv/Batch Normfusion](https://github.com/pytorch/pytorch/blob/40cbf342d3c000712da92cfafeaca651b3e0bd3e/torch/fx/experimental/optimization.py#L50)
* [替换_pattern：基本用法](https://github.com/pytorch/examples/blob/master/fx/subgraph_rewriter_basic_use.py)
* [量化] (https://pytorch.org/docs/main/quantization.html#prototype-fx-graph-mode-quantization)
* [逆变换](https://github.com/pytorch/examples/blob/master/fx /invert.py)


### 代理/重新跟踪 [¶](#proxy-retracing "此标题的永久链接")


 操作 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 的另一种方法是重用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 机制用于符号追踪。例如，假设我们想要编写一个转换，将 PyTorch 函数分解为更小的操作。它将把每个 `F.relu(x)` 调用转换为 `(x 
> 0) * x` 。一种可能是执行必要的图形重写，在“F.relu”之后插入比较和乘法，然后清理原始的“F.relu”。但是，我们可以通过使用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 对象自动将操作记录到 [`Graph`](#torch.fx.Graph "torch" 中，从而自动化此过程.fx.Graph”)。


 要使用此方法，我们将要插入的操作编写为常规PyTorch代码，并使用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 对象作为参数调用该代码。这些 [`Proxy` ](#torch.fx.Proxy "torch.fx.Proxy") 对象将捕获对其执行的操作并将它们附加到 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 。


```
# Note that this decomposition rule can be read as regular Python
def relu_decomposition(x):
    return (x > 0) * x

decomposition_rules = {}
decomposition_rules[F.relu] = relu_decomposition

def decompose(model: torch.nn.Module,
              tracer_class : type = fx.Tracer) -> torch.nn.Module:
 """
 Decompose `model` into smaller constituent operations.
 Currently,this only supports decomposing ReLU into its
 mathematical definition: (x > 0) * x
 """
    graph : fx.Graph = tracer_class().trace(model)
    new_graph = fx.Graph()
    env = {}
    tracer = torch.fx.proxy.GraphAppendingTracer(new_graph)
    for node in graph.nodes:
        if node.op == 'call_function' and node.target in decomposition_rules:
            # By wrapping the arguments with proxies,
            # we can dispatch to the appropriate
            # decomposition rule and implicitly add it
            # to the Graph by symbolically tracing it.
            proxy_args = [
                fx.Proxy(env[x.name], tracer) if isinstance(x, fx.Node) else x for x in node.args]
            output_proxy = decomposition_rulesnode.target

            # Operations on `Proxy` always yield new `Proxy`s, and the
            # return value of our decomposition rule is no exception.
            # We need to extract the underlying `Node` from the `Proxy`
            # to use it in subsequent iterations of this transform.
            new_node = output_proxy.node
            env[node.name] = new_node
        else:
            # Default case: we don't have a decomposition rule for this
            # node, so just copy the node over into the new graph.
            new_node = new_graph.node_copy(node, lambda x: env[x.name])
            env[node.name] = new_node
    return fx.GraphModule(model, new_graph)

```


 除了避免显式的图形操作之外，使用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 还允许您将重写规则指定为原生 Python 代码。对于需要大量操作的转换重写规则(例如vmap或grad)，这往往可以提高规则的可读性和可维护性。请注意，在调用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 时，我们还传递了一个指向底层变量 graph 的跟踪器。如果图中的操作是 n 元的(例如 add 是二元运算符)，则这样做是为了调用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 不会创建多个实例图形跟踪器的错误可能会导致意外的运行时错误。我们建议使用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 这种方法，特别是当底层运算符不能安全地假设为一元时。


 可以找到使用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 进行 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 操作的有效示例[此处](https://github.com/pytorch/examples/blob/master/fx/proxy_based_graph_creation.py)。


### 解释器模式 [¶](#the-interpreter-pattern "此标题的永久链接")


 FX 中一个有用的代码组织模式是循环遍历 [`Graph`](#torch.fx.Graph "torch.fx.Node") 中的所有 [`Node`](#torch.fx.Node "torch.fx.Node")。 fx.Graph")并执行它们。这可用于多种用途，包括对流经图形的值进行运行时分析或通过使用 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 进行回溯来转换代码。例如，假设我们要运行一个 [`GraphModule`](#torch.fx.GraphModule "torch.fx.GraphModule") 并记录 [`torch.Tensor`](tensors.html#torch.Tensor "torch.tensor”)节点上的形状和 dtypeproperties 正如我们在运行时看到的那样。这可能看起来像：


```
import torch
import torch.fx
from torch.fx.node import Node

from typing import Dict

class ShapeProp:
 """
 Shape propagation. This class takes a `GraphModule`.
 Then, its `propagate` method executes the `GraphModule`
 node-by-node with the given arguments. As each operation
 executes, the ShapeProp class stores away the shape and
 element type for the output values of each operation on
 the `shape` and `dtype` attributes of the operation's
 `Node`.
 """
    def __init__(self, mod):
        self.mod = mod
        self.graph = mod.graph
        self.modules = dict(self.mod.named_modules())

    def propagate(self, *args):
        args_iter = iter(args)
        env : Dict[str, Node] = {}

        def load_arg(a):
            return torch.fx.graph.map_arg(a, lambda n: env[n.name])

        def fetch_attr(target : str):
            target_atoms = target.split('.')
            attr_itr = self.mod
            for i, atom in enumerate(target_atoms):
                if not hasattr(attr_itr, atom):
                    raise RuntimeError(f"Node referenced nonexistant target {'.'.join(target_atoms[:i])}")
                attr_itr = getattr(attr_itr, atom)
            return attr_itr

        for node in self.graph.nodes:
            if node.op == 'placeholder':
                result = next(args_iter)
            elif node.op == 'get_attr':
                result = fetch_attr(node.target)
            elif node.op == 'call_function':
                result = node.target(*load_arg(node.args), **load_arg(node.kwargs))
            elif node.op == 'call_method':
                self_obj, *args = load_arg(node.args)
                kwargs = load_arg(node.kwargs)
                result = getattr(self_obj, node.target)(*args, **kwargs)
            elif node.op == 'call_module':
                result = self.modulesnode.target, **load_arg(node.kwargs))

            # This is the only code specific to shape propagation.
            # you can delete this `if` branch and this becomes
            # a generic GraphModule interpreter.
            if isinstance(result, torch.Tensor):
                node.shape = result.shape
                node.dtype = result.dtype

            env[node.name] = result

        return load_arg(self.graph.result)

```


 正如您所看到的，FX 的完整解释器并不复杂，但非常有用。为了简化此模式的使用，我们提供了 [`Interpreter`](#torch.fx.Interpreter "torch.fx.Interpreter") 类，该类包含上述逻辑，可以通过方法覆盖来覆盖解释器执行的某些方面。


 除了执行操作之外，我们还可以通过解释器提供 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy") 值来生成新的 Graph。同样，我们提供 [`Transformer`] (#torch.fx.Transformer "torch.fx.Transformer") 类来包含此模式。 [`Transformer`](#torch.fx.Transformer "torch.fx.Transformer") 的行为与 [`Interpreter`](#torch.fx.Interpreter "torch.fx.Interpreter") 类似，但不是调用 ` run` 方法要从模块获取具体的输出值，您可以调用 [`Transformer.transform()`](#torch.fx.Transformer.transform "torch.fx.Transformer.transform") 方法来返回一个新的 [ `GraphModule`](#torch.fx.GraphModule "torch.fx.GraphModule") 受您作为覆盖方法安装的任何转换规则的约束。


#### 解释器模式示例 [¶](#examples-of-the-interpreter-pattern"永久链接到此标题")



* [ShapePropagation](https://github.com/pytorch/pytorch/blob/master/torch/fx/passes/shape_prop.py)
* [性能分析器](https://github.com/pytorch/tutorials/pull /1319)


## 调试 [¶](#debugging "此标题的永久链接")


### 简介 [¶](#introduction "此标题的永久链接")


 在编写转换的过程中，我们的代码常常会不太正确。在这种情况下，我们可能需要进行一些调试。关键是逆向工作：首先，检查调用生成模块的结果以证明或反证正确性。然后，检查并调试生成的代码。然后，调试生成代码的转换过程。


 如果您不熟悉调试器，请参阅辅助部分 [可用调试器](#available-debuggers) 。


### 转换创作中的常见陷阱 [¶](#common-pitfalls-in-transform-authoring "永久链接到此标题")



* 不确定的“set”迭代顺序。在 Python 中，“set”数据类型是无序的。例如，使用“set”包含“Node”等对象的集合可能会导致意外的不确定性。一个例子是迭代一组“Node”以将它们插入到“Graph”中。由于“set”数据类型是无序的，因此输出程序中的操作顺序将是不确定的，并且可能会在程序调用之间发生变化。建议的替代方案是使用“dict”数据类型，它是[插入排序](https： //mail.python.org/pipermail/python-dev/2017-December/151283.html)自 Python 3.7 起(以及自 cPython 3.6 起)。通过将要删除重复的值存储在“dict”的键中，“dict”可以等效于集合使用。


### 检查模块的正确性 [¶](#checking
- Correctness-of-modules "永久链接到此标题")


 由于大多数深度学习模块的输出由浮点 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 实例组成，因此检查两个 [`torch.nn.Module`] 结果之间的等效性( generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") 并不像进行简单的相等检查那么简单。为了激发这一点，让我们使用一个例子：


```
import torch
import torch.fx
import torchvision.models as models

def transform(m : torch.nn.Module) -> torch.nn.Module:
    gm = torch.fx.symbolic_trace(m)

    # Imagine we're doing some transforms here
    # <...>

    gm.recompile()

    return gm

resnet18 = models.resnet18()
transformed_resnet18 = transform(resnet18)

input_image = torch.randn(5, 3, 224, 224)

assert resnet18(input_image) == transformed_resnet18(input_image)
"""
RuntimeError: Boolean value of Tensor with more than one value is ambiguous
"""

```


 在这里，我们尝试使用“==”相等运算符检查两个深度学习模型的值是否相等。然而，这并没有明确定义，因为该运算符返回tensor而不是布尔值的问题，而且还因为浮点值的比较应使用误差范围(或 epsilon)来解释浮点运算的非交换性(有关更多详细信息，请参阅[此处](https://floating-point-gui.de/errors/comparison/))。我们可以使用 [`torch.allclose()`](generated/torch.allclose.html#torch.allclose "torch.allclose") 来代替，这将给我们一个考虑相对和绝对容差阈值的近似比较：


```
assert torch.allclose(resnet18(input_image), transformed_resnet18(input_image))

```


 这是我们工具箱中的第一个工具，用于检查转换后的模块与参考实现相比是否按照我们的预期运行。


### 调试生成的代码 [¶](#debugging-the
- generated-code "Permalink to this header")


 由于 FX 在 [`GraphModule`](#torch.fx.GraphModule "torch.fx.GraphModule") 上生成 `forward()` 函数，因此使用 `print` 语句或 `pdb` 等传统调试技术并不那么简单。幸运的是，我们有几种可以用来调试生成的代码的技术。


#### 使用 `pdb`[¶](#use-pdb "此标题的永久链接")


 调用“pdb”进入正在运行的程序。尽管表示 [`Graph`](#torch.fx.Graph "torch.fx.Graph") 的代码不在任何源文件中，但我们仍然可以在调用前向传递时使用 `pdb` 手动单步执行它。


```
import torch
import torch.fx
import torchvision.models as models

def my_pass(inp: torch.nn.Module, tracer_class : type = fx.Tracer) -> torch.nn.Module:
    graph = tracer_class().trace(inp)
    # Transformation logic here
    # <...>

    # Return new Module
    return fx.GraphModule(inp, graph)

my_module = models.resnet18()
my_module_transformed = my_pass(my_module)

input_value = torch.randn(5, 3, 224, 224)

# When this line is executed at runtime, we will be dropped into an
# interactive `pdb` prompt. We can use the `step` or `s` command to
# step into the execution of the next line
import pdb; pdb.set_trace()

my_module_transformed(input_value)

```


#### 打印生成的代码 [¶](#print-the
- generated-code "Permalink to this header")


 如果您想多次运行相同的代码，那么使用“pdb”单步执行正确的代码可能会有点乏味。在这种情况下，一种方法是简单地将生成的“前向”传递复制粘贴到代码中并从那里检查它。


```
# Assume that `traced` is a GraphModule that has undergone some
# number of transforms

# Copy this code for later
print(traced)
# Print the code generated from symbolic tracing. This outputs:
"""
def forward(self, y):
 x = self.x
 add_1 = x + y; x = y = None
 return add_1
"""

# Subclass the original Module
class SubclassM(M):
    def __init__(self):
        super().__init__()

    # Paste the generated `forward` function (the one we printed and
    # copied above) here
    def forward(self, y):
        x = self.x
        add_1 = x + y;  x = y = None
        return add_1

# Create an instance of the original, untraced Module. Then, create an
# instance of the Module with the copied `forward` function. We can
# now compare the output of both the original and the traced version.
pre_trace = M()
post_trace = SubclassM()

```


#### 使用 `GraphModule` 中的 `to_folder` 函数[¶](#use-the-to-folder-function-from-graphmodule "永久链接到此标题")


[`GraphModule.to_folder()`](#torch.fx.GraphModule.to_folder "torch.fx.GraphModule.to_folder") 是 `GraphModule` 中的一个方法，允许您将生成的 FX 代码转储到文件夹中。尽管将前向传递复制到代码中通常就足够了，如[打印生成的代码](#print-the-generate-code)，但使用“to_folder”检查模块和参数可能更容易。


```
m = symbolic_trace(M())
m.to_folder("foo", "Bar")
from foo import Bar
y = Bar()

```


 运行上面的示例后，我们可以查看 foo/module.py 中的代码并根据需要进行修改(例如添加 print 语句或使用 pdb )来调试生成的代码。


### 调试转换 [¶](#debugging-the-transformation "永久链接到此标题")


 现在我们已经确定转换正在创建不正确的代码，是时候调试转换本身了。首先，我们将检查文档中的[符号跟踪的局限性](#limitations-of-symbolic-tracing)部分。一旦我们验证跟踪是否按预期工作，目标就变成找出“GraphModule”转换期间出了什么问题。 [Writing Transformations](#writing-transformations) 中可能有一个快速答案，但是，如果没有，有几种方法可以检查我们跟踪的模块：


```
# Sample Module
class M(torch.nn.Module):
    def forward(self, x, y):
        return x + y

# Create an instance of `M`
m = M()

# Symbolically trace an instance of `M` (returns a GraphModule). In
# this example, we'll only be discussing how to inspect a
# GraphModule, so we aren't showing any sample transforms for the
# sake of brevity.
traced = symbolic_trace(m)

# Print the code produced by tracing the module.
print(traced)
# The generated `forward` function is:
"""
def forward(self, x, y):
 add = x + y; x = y = None
 return add
"""

# Print the internal Graph.
print(traced.graph)
# This print-out returns:
"""
graph():
 %x : [num_users=1] = placeholder[target=x]
 %y : [num_users=1] = placeholder[target=y]
 %add : [num_users=1] = call_functiontarget=operator.add, kwargs = {})
 return add
"""

# Print a tabular representation of the internal Graph.
traced.graph.print_tabular()
# This gives us:
"""
opcode name target args kwargs
------------- ------ ----------------------- ------ --------
placeholder x x () {}
placeholder y y () {}
call_function add <built-in function add> (x, y) {}
output output output (add,) {}
"""

```


 使用上面的实用函数，我们可以比较应用转换之前和之后跟踪的模块。有时，简单的视觉比较就足以追踪错误。如果仍然不清楚出了什么问题，下一步可以使用像“pdb”这样的调试器。


 离开上面的示例，考虑以下代码：


```
# Sample user-defined function
def transform_graph(module: torch.nn.Module, tracer_class : type = fx.Tracer) -> torch.nn.Module:
    # Get the Graph from our traced Module
    g = tracer_class().trace(module)

 """
 Transformations on `g` go here
 """

    return fx.GraphModule(module, g)

# Transform the Graph
transformed = transform_graph(traced)

# Print the new code after our transforms. Check to see if it was
# what we expected
print(transformed)

```


 使用上面的示例，假设对“print(traced)”的调用表明我们的转换中存在错误。我们想使用调试器找出问题所在。我们启动一个“pdb”会话。我们可以通过中断“transform_graph(traced)”，然后按“s”“进入”对“transform_graph(traced)”的调用来查看转换期间发生的情况。


 我们还可以通过编辑 `print_tabular` 方法来打印图中节点的不同属性，从而获得好运。 (例如，我们可能想查看节点的“input_nodes”和“users”。)


### 可用调试器 [¶](#available-debuggers "此标题的固定链接")


 最常见的 Python 调试器是 [pdb](https://docs.python.org/3/library/pdb.html) 。您可以通过在命令行中键入“python -m pdb FILENAME.py”，使用“pdb”以“调试模式”启动程序，其中“FILENAME”是要调试的文件的名称。之后，您可以使用“pdb”[调试器命令](https://docs.python.org/3/library/pdb.html#debugger-commands)逐步浏览正在运行的程序。通常在启动 `pdb` 时设置断点(`b LINE-NUMBER`)，然后调用 `c` 运行程序直到该点。这使您不必单步执行每一行(使用 `s` 或 `n` )来到达您想要检查的代码部分。或者，您可以编写“import pdb;” pdb.set_trace()` 在要中断的行之前。如果添加 `pdb.set_trace()` ，程序在运行时将自动以调试模式启动。 (换句话说，您只需在命令行中输入“python FILENAME.py”，而不是“python -m pdb FILENAME.py”。)在调试模式下运行文件后，您可以单步执行代码并检查使用某些命令的程序的内部状态。网上有很多关于 pdb 的优秀教程，包括 RealPython 的 [“Python Debugging With Pdb”](https://realpython.com/python-debugging-pdb/) 。


 PyCharm 或 VSCode 等 IDE 通常内置有调试器。在 IDE 中，您可以选择 a) 通过在 IDE 中拉出终端窗口(例如 VSCode 中的视图 → 终端)来使用“pdb”，或 b) 使用内置调试器调试器(通常是 `pdb` 的图形包装器)。


## 符号跟踪的限制 [¶](#limitations-of-symbolic-tracing "永久链接到此标题")


 FX 使用**符号跟踪**(也称为 [symbolicexecution](https://en.wikipedia.org/wiki/Symbolic_execution) )以可转换/可分析的形式捕获程序的语义。该系统是 **跟踪**因为它执行程序(实际上是一个 [`torch.nn.Module`](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") 或函数)来记录运营。它是**符号**，因为在此执行期间流经程序的数据不是真实数据，而是符号(FX 术语中的 [`Proxy`](#torch.fx.Proxy "torch.fx.Proxy")) 。


 尽管符号追踪适用于大多数神经网络代码，但它有一些局限性。


### 动态控制流 [¶](#dynamic-control-flow "永久链接到此标题")


 符号跟踪的主要限制是它当前不支持*动态控制流*。也就是说，循环或“if”语句，其中条件可能取决于程序的输入值。


 例如，让我们检查以下程序：


```
def func_to_trace(x):
    if x.sum() > 0:
        return torch.relu(x)
    else:
        return torch.neg(x)

traced = torch.fx.symbolic_trace(func_to_trace)
"""
 <...>
 File "dyn.py", line 6, in func_to_trace
 if x.sum() > 0:
 File "pytorch/torch/fx/proxy.py", line 155, in __bool__
 return self.tracer.to_bool(self)
 File "pytorch/torch/fx/proxy.py", line 85, in to_bool
 raise TraceError('symbolically traced variables cannot be used as inputs to control flow')
torch.fx.proxy.TraceError: symbolically traced variables cannot be used as inputs to control flow
"""

```


 `if` 语句的条件依赖于 `x.sum()` 的值，而 `x.sum()` 又依赖于函数输入 `x` 的值。由于“x”可以改变(即，如果您将新的输入tensor传递给跟踪函数)，这就是“动态控制流”。回溯会向上遍历您的代码，以向您显示这种情况发生的位置。


#### 静态控制流 [¶](#static-control-flow "永久链接到此标题")


 另一方面，支持所谓的“静态控制流”。静态控制流是循环或“if”语句，其值不能在调用之间更改。通常，在 PyTorch 程序中，这种控制流程是针对基于超参数对模型架构做出决策的代码而出现的。作为一个具体的例子：


```
import torch
import torch.fx

class MyModule(torch.nn.Module):
    def __init__(self, do_activation : bool = False):
        super().__init__()
        self.do_activation = do_activation
        self.linear = torch.nn.Linear(512, 512)

    def forward(self, x):
        x = self.linear(x)
        # This if-statement is so-called static control flow.
        # Its condition does not depend on any input values
        if self.do_activation:
            x = torch.relu(x)
        return x

without_activation = MyModule(do_activation=False)
with_activation = MyModule(do_activation=True)

traced_without_activation = torch.fx.symbolic_trace(without_activation)
print(traced_without_activation.code)
"""
def forward(self, x):
 linear_1 = self.linear(x); x = None
 return linear_1
"""

traced_with_activation = torch.fx.symbolic_trace(with_activation)
print(traced_with_activation.code)
"""
import torch
def forward(self, x):
 linear_1 = self.linear(x); x = None
 relu_1 = torch.relu(linear_1); linear_1 = None
 return relu_1
"""

```


 if 语句 `if self.do_activation` 不依赖于任何函数输入，因此它是静态的。 `do_activation` 可以被认为是一个超参数，并且具有不同参数值的 `MyModule` 的不同实例的跟踪具有不同的代码。这是符号跟踪支持的有效模式。


 动态控制流的许多实例在语义上是静态控制流。可以通过删除对输入值的数据依赖性来使这些实例支持符号跟踪，例如通过将值移动到“Module”属性或在符号跟踪期间将具体值绑定到参数：


```
def f(x, flag):
    if flag: return x
    else: return x*2

fx.symbolic_trace(f) # Fails!

fx.symbolic_trace(f, concrete_args={'flag': True})

```


 在真正动态控制流的情况下，包含此代码的程序部分可以作为对方法(请参阅[使用 Tracer 类自定义跟踪](#customizing-tracing))或函数(请参阅[`wrap( )`](#torch.fx.wrap "torch.fx.wrap") ) 而不是追踪它们。


### 非 `torch` 函数 [¶](#non-torch-functions "永久链接到此标题")


 FX 使用 `__torch_function__` 作为拦截调用的机制(请参阅[技术概述](https://github.com/pytorch/pytorch/blob/master/torch/fx/OVERVIEW. md#technical-details)了解更多信息)。某些函数，例如内置 Python 函数或 `math` 模块中的函数，未包含在 `__torch_function__` 中，但我们仍然希望以符号跟踪方式捕获它们。例如：


```
import torch
import torch.fx
from math import sqrt

def normalize(x):
 """
 Normalize `x` by the size of the batch dimension
 """
    return x / sqrt(len(x))

# It's valid Python code
normalize(torch.rand(3, 4))

traced = torch.fx.symbolic_trace(normalize)
"""
 <...>
 File "sqrt.py", line 9, in normalize
 return x / sqrt(len(x))
 File "pytorch/torch/fx/proxy.py", line 161, in __len__
 raise RuntimeError("'len' is not supported in symbolic tracing by default. If you want "
RuntimeError: 'len' is not supported in symbolic tracing by default. If you want this call to be recorded, please call torch.fx.wrap('len') at module scope
"""

```


 该错误告诉我们不支持内置函数 `len`。我们可以使用 [`wrap()`](#torch.fx.wrap " 将此类函数记录在跟踪中作为直接调用torch.fx.wrap”)API：


```
torch.fx.wrap('len')
torch.fx.wrap('sqrt')

traced = torch.fx.symbolic_trace(normalize)

print(traced.code)
"""
import math
def forward(self, x):
 len_1 = len(x)
 sqrt_1 = math.sqrt(len_1); len_1 = None
 truediv = x / sqrt_1; x = sqrt_1 = None
 return truediv
"""

```


### 使用 `Tracer` 类自定义跟踪 [¶](#customizing-tracing-with-the-tracer-class "Permalink to this header")


 [`Tracer`](#torch.fx.Tracer "torch.fx.Tracer") 类是 `symbolic_trace` 实现的基础类。跟踪的行为可以通过子类化 Tracer 来自定义，如下所示：


```
class MyCustomTracer(torch.fx.Tracer):
    # Inside here you can override various methods
    # to customize tracing. See the `Tracer` API
    # reference
    pass


# Let's use this custom tracer to trace through this module
class MyModule(torch.nn.Module):
    def forward(self, x):
        return torch.relu(x) + torch.ones(3, 4)

mod = MyModule()

traced_graph = MyCustomTracer().trace(mod)
# trace() returns a Graph. Let's wrap it up in a
# GraphModule to make it runnable
traced = torch.fx.GraphModule(mod, traced_graph)

```


#### 叶模块 [¶](#leaf-modules"此标题的永久链接")


 叶模块是在符号跟踪中显示为调用而不是被跟踪的模块。默认的叶模块集是标准“torch.nn”模块实例集。例如：


```
class MySpecialSubmodule(torch.nn.Module):
    def forward(self, x):
        return torch.neg(x)

class MyModule(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = torch.nn.Linear(3, 4)
        self.submod = MySpecialSubmodule()

    def forward(self, x):
        return self.submod(self.linear(x))

traced = torch.fx.symbolic_trace(MyModule())
print(traced.code)
# `linear` is preserved as a call, yet `submod` is traced though.
# This is because the default set of "Leaf Modules" includes all
# standard `torch.nn` modules.
"""
import torch
def forward(self, x):
 linear_1 = self.linear(x); x = None
 neg_1 = torch.neg(linear_1); linear_1 = None
 return neg_1
"""

```


 叶模块集可以通过重写 [`Tracer.is_leaf_module()`](#torch.fx.Tracer.is_leaf_module "torch.fx.Tracer.is_leaf_module") 来自定义。


### Miscellanea [¶](#miscellanea "此标题的永久链接")



* tensor构造函数(例如 `torch.zeros` 、 `torch.ones` 、 `torch.rand` 、 `torch.randn` 、 `torch.sparse_coo_tensor` )当前不可追踪。


	+ The deterministic constructors (
	 `zeros`
	 ,
	 `ones`
	 ) can be used
	and the value they produce will be embedded in the trace as a
	constant. This is only problematic if the arguments to these
	constructors refers to dynamic input sizes. In this case,
	 `ones_like`
	 or
	 `zeros_like`
	 may be a viable substitute.
	+ Nondeterministic constructors (
	 `rand`
	 ,
	 `randn`
	 ) will have a
	single random value embedded in the trace. This is likely not the
	intended behavior. One workaround is to wrap
	 `torch.randn`
	 in a
	 `torch.fx.wrap`
	 function and call that instead.
> 
> 
> 
> 
> 
> ```
> @torch.fx.wrap
> def torch_randn(x, shape):
>     return torch.randn(shape)
> 
> def f(x):
>     return x + torch_randn(x, 5)
> fx.symbolic_trace(f)
> 
> ```
> 
> 
> 
> 
> 



+ 此行为可能会在未来版本中修复。
* 类型注释



+ 支持 Python 3 样式类型注释(例如 `func(x : torch.Tensor, y : int) -
> torch.Tensor` )并将通过符号跟踪保留。 
+ 目前不支持 Python 2 风格注释类型注释 `# type: (torch.Tensor, int) -
> torch.Tensor`。 
+ 当前不支持函数内本地名称的注释。
* 关于“training”标志和子模块的问题


	+ When using functionals like
	 `torch.nn.functional.dropout`
	 , it will be common for the training argument to be passed in as
	 `self.training`
	 . During FX tracing, this will likely be baked in as a constant value.
> 
> 
> 
> 
> 
> ```
> import torch
> import torch.fx
> 
> class DropoutRepro(torch.nn.Module):
>   def forward(self, x):
>     return torch.nn.functional.dropout(x, training=self.training)
> 
> 
> traced = torch.fx.symbolic_trace(DropoutRepro())
> print(traced.code)
> """
> def forward(self, x):
>  dropout = torch.nn.functional.dropout(x, p = 0.5, training = True, inplace = False); x = None
>  return dropout
> """
> 
> traced.eval()
> 
> x = torch.randn(5, 3)
> torch.testing.assert_close(traced(x), x)
> """
> AssertionError: Tensor-likes are not close!
> 
> Mismatched elements: 15 / 15 (100.0%)
> Greatest absolute difference: 1.6207983493804932 at index (0, 2) (up to 1e-05 allowed)
> Greatest relative difference: 1.0 at index (0, 0) (up to 0.0001 allowed)
> """
> 
> ```
> 
> 
> 
> 
> 


	+ However, when the standard
	 `nn.Dropout()`
	 submodule is used, the training flag is encapsulated and–because of the preservation of the
	 `nn.Module`
	 object model–can be changed.
> 
> 
> 
> 
> 
> ```
> class DropoutRepro2(torch.nn.Module):
>   def __init__(self):
>     super().__init__()
>     self.drop = torch.nn.Dropout()
> 
>   def forward(self, x):
>     return self.drop(x)
> 
> traced = torch.fx.symbolic_trace(DropoutRepro2())
> print(traced.code)
> """
> def forward(self, x):
>  drop = self.drop(x); x = None
>  return drop
> """
> 
> traced.eval()
> 
> x = torch.randn(5, 3)
> torch.testing.assert_close(traced(x), x)
> 
> ```
> 
> 
> 
> 
>



> 
> 
> 
* 由于这种差异，请考虑将与 
> `training`
> 标志动态交互的模块标记为叶模块。
> 
> 
> >


## API 参考 [¶](#api-reference "此标题的永久链接")


 火炬.fx。


 符号_trace


 ( *root
* , *concrete_args



 =
 


 无
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#symbolic_trace)[¶](#torch.fx.symbolic_trace "此定义的永久链接")


 符号跟踪 API


 给定一个 nn.Module 或函数实例 root ，该函数将返回一个通过记录在跟踪 root 时看到的操作而构造的 GraphModule 。


`concrete_args` 允许您部分专门化您的函数，无论是删除控制流还是数据结构。


 例如：


```
def f(a, b):
    if b == True:
        return a
    else:
        return a*2

```


 由于控制流的存在，FX 通常无法对此进行跟踪。然而，我们可以使用具体_args来专门研究 b 的值来跟踪：


```
f = fx.symbolic_trace(f, concrete_args={'b': False})
assert f(3, False)  == 6

```


 请注意，尽管您仍然可以传入不同的 b 值，但它们将被忽略。


 我们还可以使用具体_args 来消除函数中的数据结构处理。这将使用 pytree 来展平您的输入。为了避免过度专业化，请传入 fx.PH 来获取不应专业化的值。例如：


```
def f(x):
    out = 0
    for v in x.values():
        out += v
    return out
f = fx.symbolic_trace(f, concrete_args={'x': {'a': fx.PH, 'b': fx.PH, 'c': fx.PH}})
assert f({'a': 1, 'b': 2, 'c': 4}) == 7

```


 参数 
* **root** ( *Union
* *[
* [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module")*,
* *Callable
* *]
* ) – 要跟踪并转换为图形表示的模块或函数。
* **concrete_args** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *any
* *]
* *]
* ) – 要部分专业化的输入


 退货


 根据“root”记录的操作创建的模块。


 Return type


[GraphModule](#torch.fx.GraphModule "torch.fx.GraphModule")




!!! note "笔记"

    保证此 API 的向后兼容性。


 火炬.fx。



 wrap
 


 ( *fn_or_name
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#wrap)[¶](#torch.fx.wrap "此定义的永久链接")


 可以在模块级范围调用此函数，以将 fn_or_name 注册为“叶函数”。“叶函数”将在 FX 跟踪中保留为 CallFunction 节点，而不是通过以下方式进行跟踪：


```
# foo/bar/baz.py
def my_custom_function(x, y):
    return x * x + y * y

torch.fx.wrap('my_custom_function')

def fn_to_be_traced(x, y):
    # When symbolic tracing, the below call to my_custom_function will be inserted into
    # the graph rather than tracing it.
    return my_custom_function(x, y)

```


 该函数也可以等效地用作装饰器：


```
# foo/bar/baz.py
@torch.fx.wrap
def my_custom_function(x, y):
    return x * x + y * y

```


 包装函数可以被认为是“叶函数”，类似于“叶模块”的概念，也就是说，它们是作为调用留在 FX 跟踪中而不是进行跟踪的函数。


 Parameters


**fn_or_name** ( *Union
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)" )*,
* *Callable
* *]
* ) – 调用时插入到图中的函数或全局函数的名称



!!! note "笔记"

    保证此 API 的向后兼容性。


*班级*


 火炬.fx。


 图模块


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/fx/graph_module.html#GraphModule)[¶](#torch.fx.GraphModule "此定义的永久链接")


 GraphModule 是从 fx.Graph 生成的 nn.Module。 Graphmodule 有一个“graph”属性，以及从该“graph”生成的“code”和“forward”属性。


!!! warning "警告"

     当重新分配“graph”时，“code”和“forward”将自动重新生成。但是，如果您编辑“graph”的内容而不重新分配“graph”属性本身，则必须调用“recompile()”来更新生成的代码。


!!! note "笔记"

    保证此 API 的向后兼容性。


 __在里面__


 ( *root
* , *graph
* , *class_name



 =
 


 'GraphModule'
* ) [[source]](_modules/torch/fx/graph_module.html#GraphModule.__init__)[¶](#torch.fx.GraphModule.__init__ "此定义的永久链接")


 构造一个 GraphModule。


 参数 
* **root** ( *Union
* *[
* [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module")*,
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *任何
* *]
* ) – `root` 可以是一个 nn.Module 实例，也可以是一个将字符串映射到任何属性类型的 Dict。在 `root` 是一个模块的情况下，对图形节点中基于模块的对象(通过限定名称)的任何引用`target` 字段将从 `root` 模块层次结构中的相应位置复制到 GraphModule 的模块层次结构中。如果 `root` 是一个字典，则将直接查找在 Node 的 `target` 中找到的限定名称在字典的键中。 Dict 映射到的对象将被复制到 GraphModule 模块层次结构中的适当位置。
* **graph** ( [*Graph*](#torch.fx.Graph "torch.fx.Graph") ) – ` graph` 包含此 GraphModule 用于代码生成的节点
* **class_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python 中) v3.12)") ) – `name` 表示此 GraphModule 的名称，用于调试目的。如果未设置，所有错误消息将报告为源自“GraphModule”。将其设置为“root”的原始名称或在转换上下文中有意义的名称可能会有所帮助。



!!! note "笔记"

    保证此 API 的向后兼容性。


 添加_子模块


 ( *target
* , *m
* ) [[source]](_modules/torch/fx/graph_module.html#GraphModule.add_submodule)[¶](#torch.fx.GraphModule.add_submodule "此定义的永久链接")


 将给定的子模块添加到 `self` 中。


 如果它们是 `target` 的子路径，这将安装尚不存在的空模块。


 参数 
* **target** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 完全限定字符串新子模块的名称(有关如何指定完全限定字符串的信息，请参阅 `nn.Module.get_submodule` 中的示例。)
* **m** ( [*Module*](generated/torch.nn.Module.html #torch.nn.Module "torch.nn.modules.module.Module") ) – 子模块本身；我们要在当前模块中安装的实际对象


 退货


 是否可以插入子模块。为了


 此方法返回 True，链中由“target”表示的每个对象必须 a) 尚不存在，或 b) 引用“nn.Module”(不是参数或其他属性)


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)




!!! note "笔记"

    保证此 API 的向后兼容性。


*财产*


 代码 *：


[str](https://docs.python.org/3/library/stdtypes.html#str"(在Python v3.12中)")*[¶](#torch.fx.GraphModule.code"永久链接到此定义")


 返回从该“GraphModule”底层的“Graph”生成的Python代码。


 删除_所有_未使用_子模块


 ( ) [[source]](_modules/torch/fx/graph_module.html#GraphModule.delete_all_unused_submodules)[¶](#torch.fx.GraphModule.delete_all_unused_submodules "此定义的永久链接")


 从 `self` 中删除所有未使用的子模块。


 如果满足以下任一条件，则模块被视为“已使用”：1.它有已使用的子项2。它的转发是通过“call_module”节点直接调用的。它有一个从 `get_attr` 节点使用的非模块属性


 可以调用此方法来清理“nn.Module”，而无需在每个未使用的子模块上手动调用“delete_submodule”。




!!! note "笔记"

    保证此 API 的向后兼容性。


 删除_子模块


 ( *target
* ) [[source]](_modules/torch/fx/graph_module.html#GraphModule.delete_submodule)[¶](#torch.fx.GraphModule.delete_submodule "此定义的永久链接")


 从 `self` 中删除给定的子模块。


 如果“target”不是有效目标，则不会删除该模块。


 Parameters


**target** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 的完全限定字符串名称新的子模块(有关如何指定完全限定字符串的信息，请参阅“nn.Module.get_submodule”中的示例。)


 退货


 目标字符串是否引用了


 我们要删除的子模块。返回值“False”意味着“target”不是对子模块的有效引用。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)




!!! note "笔记"

    保证此 API 的向后兼容性。


*财产*


 图表*：


[Graph](#torch.fx.Graph "torch.fx.graph.Graph")*[¶](#torch.fx.GraphModule.graph "此定义的永久链接")


 返回此“GraphModule”底层的“Graph”


 打印_可读


 ( *打印_输出



 =
 


 True
* ) [[source]](_modules/torch/fx/graph_module.html#GraphModule.print_ 可读取)[¶](#torch.fx.GraphModule.print_可读取"此定义的永久链接")


 返回为当前 GraphModule 及其子 GraphModule 生成的 Python 代码


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


 重新编译


 ( ) [[source]](_modules/torch/fx/graph_module.html#GraphModule.recompile)[¶](#torch.fx.GraphModule.recompile "此定义的永久链接")


 从其“graph”属性重新编译此 GraphModule。这应该在编辑包含的“graph”之后调用，否则这个“GraphModule”的生成代码将过时。




!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


*Python代码*


 到_文件夹


 ( *文件夹
* , *模块_名称



 =
 


 'FxModule'
* ) [[source]](_modules/torch/fx/graph_module.html#GraphModule.to_folder)[¶](#torch.fx.GraphModule.to_folder "此定义的永久链接")


 将模块转储到带有“module_name”的“folder”，以便可以


 使用“from <folder
> import <module_name>”导入




 Args:
 



> 
> 
> 
> 文件夹 (Union[str, os.PathLike])：将代码写入的文件夹
> 
> 
> 
> 
> 
> module_name (str)：用于 
> `Module`
> 的顶级名称while
> 
> 
> 
> 写出代码
> 
> 
> 
> 
> 
> >


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


*班级*


 火炬.fx。



 Graph
 


 ( *拥有_模块



 =
 


 无
* , *跟踪器_cls



 =
 


 无
* , *跟踪器_extras



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph)[¶](#torch.fx.Graph "此定义的永久链接")


“Graph”是 FX 中间表示中使用的主要数据结构。它由一系列“Node”组成，每个节点代表调用点(或其他语法结构)。 Node 的列表组合在一起就构成了有效的 Python 函数。


 例如下面的代码


```
import torch
import torch.fx

class MyModule(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.param = torch.nn.Parameter(torch.rand(3, 4))
        self.linear = torch.nn.Linear(4, 5)

    def forward(self, x):
        return torch.topk(torch.sum(self.linear(x + self.linear.weight).relu(), dim=-1), 3)

m = MyModule()
gm = torch.fx.symbolic_trace(m)

```


 将产生以下图表：


```
print(gm.graph)

```



```
graph(x):
    %linear_weight : [num_users=1] = self.linear.weight
    %add_1 : [num_users=1] = call_functiontarget=operator.add, kwargs = {})
    %linear_1 : [num_users=1] = call_moduletarget=linear, kwargs = {})
    %relu_1 : [num_users=1] = call_methodtarget=relu, kwargs = {})
    %sum_1 : [num_users=1] = call_functiontarget=torch.sum, kwargs = {dim: -1})
    %topk_1 : [num_users=1] = call_functiontarget=torch.topk, kwargs = {})
    return topk_1

```


 有关 `Graph` 中表示的操作的语义，请参阅 [`Node`](#torch.fx.Node "torch.fx.Node") 。




!!! note "笔记"

    保证此 API 的向后兼容性。


 __在里面__


 ( *拥有_模块



 =
 


 无
* , *跟踪器_cls



 =
 


 无
* , *跟踪器_extras



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.__init__)[¶](#torch.fx.Graph.__init__ "此定义的永久链接")


 构造一个空图。




!!! note "笔记"

    保证此 API 的向后兼容性。


 调用_函数


 ( *函数
* , *args



 =
 


 无*，*kwargs



 =
 


 无
* , *类型_expr



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.call_function)[¶](#torch.fx.Graph.call_function "此定义的永久链接")


 将 `call_function``Node` 插入到 `Graph` 中。 `call_function` 节点表示对由 `the_function` 指定的 Python 可调用对象的调用。


 参数 
* **the_function** ( *Callable
* *[
* *...
* *,
* *Any
* *]
* ) – 要调用的函数。可以是任何 PyTorchoperator、Python 函数或 `builtins` 或 `operator` 命名空间的成员。
* **args** ( *可选
* *[
* *元组
* *[
* *参数
* *,
* *... 
* *]
* *]
* ) – 要传递给被调用函数的位置参数。
* **kwargs** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs. python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *Argument
* *]
* *]
* ) – 要传递给被调用函数的关键字参数
* **类型_expr** ( *Optional
* *[
* *Any
* *]
* ) – 可选类型注释，表示此节点的输出将具有的 Python 类型。


 退货


 新创建并插入的“call_function”节点。


 Return type


[*节点*](#torch.fx.Node "torch.fx.node.Node")




!!! note "笔记"

    相同的插入点和类型表达式规则适用于此方法，如 [`Graph.create_node()`](#torch.fx.Graph.create_node "torch.fx.Graph.create_node") 。


!!! note "笔记"

    保证此 API 的向后兼容性。


 调用_方法


 ( *方法_名称
* , *参数



 =
 


 无*，*kwargs



 =
 


 无
* , *类型_expr



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.call_method)[¶](#torch.fx.Graph.call_method "此定义的永久链接")


 将 `call_method``Node` 插入到 `Graph` 中。 `call_method` 节点表示对 `args` 第 0 个元素上给定方法的调用。


 参数 
* **method_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 的名称应用于 self 参数的方法。例如，如果 args[0] 是代表“Tensor”的“Node”，则要在该“Tensor”上调用“relu()”，请将“relu”传递给“method\” _name`.
* **args** ( *Optional
* *[
* *Tuple
* *[
* *Argument
* *,
* *...
* *]
* *]
* ) – 要传递给被调用方法的位置参数。请注意，这个 *应该
* 包含一个 `self` 参数。
* **kwargs** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *Argument
* *]
* *]
* ) – 要传递给被调用方法的关键字参数
* **type_expr** ( *可选
* *[
* *Any
* *]
* ) – 可选类型注释，表示此节点的输出将具有的 Python 类型。


 退货


 新创建并插入的“call_method”节点。


 Return type


[*节点*](#torch.fx.Node "torch.fx.node.Node")




!!! note "笔记"

    相同的插入点和类型表达式规则适用于此方法，如 [`Graph.create_node()`](#torch.fx.Graph.create_node "torch.fx.Graph.create_node") 。


!!! note "笔记"

    保证此 API 的向后兼容性。


 调用_模块


 ( *模块_name
* , *args



 =
 


 无*，*kwargs



 =
 


 无
* , *类型_expr



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.call_module)[¶](#torch.fx.Graph.call_module "此定义的永久链接")


 将 `call_module``Node` 插入到 `Graph` 中。 “call_module”节点表示对“Module”层次结构中“Module”的forward()函数的调用。


 参数 
* **module_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 限定名称要调用的“Module”层次结构中的“Module”的名称。例如，如果跟踪的“Module”有一个名为“foo”的子模块，该子模块有一个名为“bar”的子模块，则限定名称“foo.bar”应作为“module_name”传递以调用该模块。
* **args
* 
* ( *可选
* *[
* *元组
* *[
* *参数
* *,
* *...
* *]
* *]
* ) – 要传递给被调用方法的位置参数。请注意，这不应*包含 `self` 参数。
* **kwargs** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3 /library/stdtypes.html#str "(in Python v3.12)")*,
* *Argument
* *]
* *]
* ) – 要传递给被调用方法的关键字参数
* **type_expr** ( *Optional
* *[
* *Any
* *]
* ) – 可选类型注释，表示此节点的输出将具有的 Python 类型。


 退货


 新创建并插入的“call_module”节点。


 Return type


[*节点*](#torch.fx.Node "torch.fx.node.Node")




!!! note "笔记"

    相同的插入点和类型表达式规则适用于此方法，如 [`Graph.create_node()`](#torch.fx.Graph.create_node "torch.fx.Graph.create_node") 。


!!! note "笔记"

    保证此 API 的向后兼容性。


 创建_node


 (*操作*，*目标*，*参数



 =
 


 无*，*kwargs



 =
 


 无*，*名称



 =
 


 无
* , *类型_expr



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.create_node)[¶](#torch.fx.Graph.create_node "此定义的永久链接")


 创建一个“Node”并将其添加到当前插入点处的“Graph”。请注意，当前插入点可以通过 [`Graph.inserting_before()`](#torch.fx.Graph. inserting_before "torch.fx.Graph.inserting_before") 和 [`Graph.inserting_after()`](#torch.fx.Graph.inserting_after "torch.fx.Graph.inserting_after") 。


 参数 
* **op** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 该节点的操作码。 ‘call_function’、‘call_method’、‘get_attr’、‘call_module’、‘placeholder’ 或 ‘output’ 之一。这些操作码的语义在 `Graph` 文档字符串中描述。
* **args** ( *Optional
* *[
* *Tuple
* *[
* *Argument
* *,
* *...
* *]
* *]
* ) – 是此节点的参数元组。
* **kwargs** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* *参数
* *]
* *]
* ) – 此节点的 kwargs
* **名称** ( *可选
* *[
* [*str
* ](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* ) – `Node` 的可选字符串名称。这将影响Python 生成的代码中分配给的值的名称。
* **type_expr** ( *Optional
* *[
* *Any
* *]
* ) – 可选类型注释，表示此节点的输出将具有的 Python 类型。


 退货


 新创建并插入的节点。


 Return type


[*节点*](#torch.fx.Node "torch.fx.node.Node")




!!! note "笔记"

    保证此 API 的向后兼容性。


 消除_dead_code


 ( ) [[source]](_modules/torch/fx/graph.html#Graph.eliminate_dead_code)[¶](#torch.fx.Graph.eliminate_dead_code "此定义的永久链接")


 根据每个节点的用户数量以及节点是否有任何副作用，从图中删除所有死代码。在调用之前必须对图进行拓扑排序。


 退货


 图形是否因传递而改变。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 例子：


 在消除死代码之前，下面的 a = x 
+ 1 中的 a 没有用户，因此可以从图中消除而不会产生任何影响。


```
def forward(self, x):
    a = x + 1
    return x + self.attr_1

```


 消除死代码后，a = x 
+ 1 已被删除，其余部分仍保留。


```
def forward(self, x):
    return x + self.attr_1

```


!!! warning "警告"

     死代码消除具有一些启发式方法，可以避免删除副作用节点(请参阅 Node.is_impure)，但一般来说覆盖率非常差，因此您应该假设此方法不适合调用，除非您知道 FX 图完全由函数操作组成。


!!! note "笔记"

    保证此 API 的向后兼容性。


 擦除_node


 ( *to_erase
* ) [[source]](_modules/torch/fx/graph.html#Graph.erase_node)[¶](#torch.fx.Graph.erase_node "此定义的永久链接")


 从“Graph”中删除“Node”。如果“Graph”中仍然存在该节点的用户，则抛出异常。


 Parameters


**to_erase** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – 要从 `Graph` 中删除的 `Node`。



!!! note "笔记"

    保证此 API 的向后兼容性。


 获取_attr


 ( *限定_name
* , *类型_expr



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.get_attr)[¶](#torch.fx.Graph.get_attr "此定义的永久链接")


 将 `get_attr` 节点插入到图表中。 `get_attr``Node` 表示从 `Module` 层次结构中获取属性。


 参数 
* **qualified_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 完全-要检索的属性的限定名称。例如，如果跟踪的 Module 有一个名为 `foo` 的子模块，该子模块有一个名为 `bar` 的子模块，该子模块有一个名为 `baz` 的属性，则限定名称为 `foo.bar.baz`应作为 `qualified_name` 传递。
* **type_expr** ( *Optional
* *[
* *Any
* *]
* ) – 表示此节点输出将具有的 Python 类型的可选类型注释。


 退货


 新创建并插入的“get_attr”节点。


 Return type


[*节点*](#torch.fx.Node "torch.fx.node.Node")




!!! note "笔记"

    相同的插入点和类型表达式规则适用于此方法，如“Graph.create_node”。


!!! note "笔记"

    保证此 API 的向后兼容性。


 图_copy


 ( *g
* , *val_map
* , *return_output_node



 =
 


 False
* ) [[source]](_modules/torch/fx/graph.html#Graph.graph_copy)[¶](#torch.fx.Graph.graph_copy "此定义的永久链接")


 将给定图中的所有节点复制到 self 中。


 参数 
* **g** ( [*Graph*](#torch.fx.Graph "torch.fx.Graph") ) – 从中复制节点的源图。
* **val_map** ( *Dict 
* *[
* [*Node*](#torch.fx.Node "torch.fx.Node")*,
* [*Node*](#torch.fx.Node "torch.fx.Node")*]
* ) – 一个字典，将填充从 `g` 中的节点到 `self` 中的节点的映射。请注意，“val_map”可以传入其中已包含的值，以覆盖某些值的复制。


 退货


 如果“g”有一个“output”节点，“self”中的值现在相当于“g”中的输出值。否则“无”。


 Return type


[*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中)") [ [*Union*](https://docs. python.org/3/library/typing.html#typing.Union "(Python v3.12)") [ [*Tuple*](https://docs.python.org/3/library/typing.html# Typing.Tuple "(Python v3.12)") [ [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)" ) , …], [*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*Any*](https ://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)") ], [*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12 中)") [ [str](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12 中) )") , [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)") ], [slice](https:///docs.python.org/3/library/functions.html#slice "(Python v3.12)") , [范围](https://docs.python.org/3/library/stdtypes.html#range "(Python v3.12)") , [Node](#torch.fx.Node "torch.fx.Node") , [str](https://docs.python.org/3/library/stdtypes. html#str "(Python v3.12)") , [int](https://docs.python.org/3/library/functions.html#int "(Python v3.12)") , [float ](https://docs.python.org/3/library/functions.html#float "(Python v3.12)") , [bool](https://docs.python.org/3/library/function.html#bool "(Python v3.12)") , [complex](https://docs.python.org/3/library/functions.html#complex "(Python v3.12)") , [*dtype*](tensor_attributes.html#torch.dtype "torch.dtype") , [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") , [*device*](tensor_attributes.html# torch.device "torch.device") , [*memory_format*](tensor_attributes.html#torch.memory_format "torch.memory_format") , [*layout*](tensor_attributes.html#torch.layout "torch.layout" ) , *OpOverload
* ]]



!!! note "笔记"

    保证此 API 的向后兼容性。


 插入_之后


 (
 
*n
 



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.inserting_after)[¶](#torch.fx.Graph.inserting_after "此定义的永久链接")


 设置 create_node 和伴随方法将插入到图中的点。


 当在“with”语句中使用时，这将临时设置插入点，然后在 with 语句退出时恢复它：


```
with g.inserting_after(n):
    ... # inserting after node n
... # insert point restored to what it was previously
g.inserting_after(n) # set the insert point permanently

```




 Args:
 



> 
> 
> 
> 
> n(可选[节点])：要在其之前插入的节点。如果没有，这将插入到
> 
> 
> >整个图表的开头之后。
> 
> 
> 
> 
> 
> >


 返回：


 一个资源管理器，它将恢复 `__exit__` 上的插入点。


!!! note "笔记"

    保证此 API 的向后兼容性。


 在之前插入_


 (
 
*n
 



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.inserting_before)[¶](#torch.fx.Graph.inserting_before "此定义的永久链接")


 设置 create_node 和伴随方法将插入到图中的点。


 当在“with”语句中使用时，这将临时设置插入点，然后在 with 语句退出时恢复它：


```
with g.inserting_before(n):
    ... # inserting before node n
... # insert point restored to what it was previously
g.inserting_before(n) # set the insert point permanently

```




 Args:
 



> 
> 
> 
> 
> n(可选[节点])：要在其之前插入的节点。如果没有，这将插入在
> 
> 
> >整个图形的开头之前。
> 
> 
> 
> 
> 
> >


 返回：


 一个资源管理器，它将恢复 `__exit__` 上的插入点。


!!! note "笔记"

    保证此 API 的向后兼容性。


 lint
 


 ( ) [[source]](_modules/torch/fx/graph.html#Graph.lint)[¶](#torch.fx.Graph.lint "此定义的永久链接")


 对此图运行各种检查以确保其格式良好。特别是： 
- 检查节点是否具有正确的所有权(由该图拥有) 
- 检查节点是否按拓扑顺序出现 
- 如果该图有一个所属的 GraphModule，则检查该 GraphModule 中是否存在目标




!!! note "笔记"

    保证此 API 的向后兼容性。


 节点_copy


 ( *node
* , *arg_transform=<function Graph.<lambda>>
* ) [[source]](_modules/torch/fx/graph.html#Graph.node_copy)[¶](#torch.fx.Graph.node_copy"此定义的永久链接")


 将一个节点从一个图中复制到另一个图中。 `arg_transform` 需要将参数从节点图转换为自身图。例子：


```
# Copying all the nodes in `g` into `new_graph`
g : torch.fx.Graph = ...
new_graph = torch.fx.graph()
value_remap = {}
for node in g.nodes:
    value_remap[node] = new_graph.node_copy(node, lambda n : value_remap[n])

```


 参数 
* **node** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – 要复制到 `self` 的节点。
* **arg_transform** ( *Callable 
* *[
* *[
* [*Node*](#torch.fx.Node "torch.fx.Node")*]
* *,
* *Argument
* *]
* ) – 将 `Node` 参数转换为将节点的 `args` 和 `kwargs` 转换为 `self` 中的等效参数。在最简单的情况下，这应该从将原始图中的节点映射到“self”的表中检索一个值。


 Return type


[*节点*](#torch.fx.Node "torch.fx.node.Node")




!!! note "笔记"

    保证此 API 的向后兼容性。


*财产*


 节点*：


 _node_list*[¶](#torch.fx.Graph.nodes "此定义的永久链接")


 获取构成该图的节点列表。


 请注意，这个“Node”列表表示是一个双向链表。迭代期间的突变(例如删除节点、添加节点)是安全的。


 退货


 节点的双向链表。请注意，可以在此列表上调用“reversed”来切换迭代顺序。


 在_generate_code上


 ( *make_transformer
* ) [[source]](_modules/torch/fx/graph.html#Graph.on_generate_code)[¶](#torch.fx.Graph.on_generate_code "此定义的永久链接")


 生成python代码时注册一个变压器函数



> 
> 
> 
> 
>  Args:
>  
> 
> 
> 
>  make_transformer (Callable[[Optional[TransformCodeFunc]], TransformCodeFunc]):
>  
> 
> 
>  a function that returns a code transformer to be registered.
> This function is called by
>  
>  on_generate_code
>  
>  to obtain the
> code transformer.
>  
> 
> 
> 
>  This function is also given as its input the currently
> registered code transformer (or None if nothing is registered),
> in case it is not desirable to overwrite it. This is useful to
> chain code transformers together.
>  
> 
> 
> 
> 
> 
> 
>  Returns:
>  
> 
> 
>  a context manager that when used in a
>  
>  with
>  
>  statement, to automatically
> restore the previously registered code transformer.
>  
> 
> 
> 
> 
> 
>  Example:
>  
> 
> 
> 
> 
> 
> ```
> gm: fx.GraphModule = ...
> 
> # This is a code transformer we want to register. This code
> # transformer prepends a pdb import and trace statement at the very
> # beginning of the generated torch.fx code to allow for manual
> # debugging with the PDB library.
> def insert_pdb(body):
>     return ["import pdb; pdb.set_trace()
", *body]
> 
> # Registers `insert_pdb`, and overwrites the current registered
> # code transformer (given by `_` to the lambda):
> gm.graph.on_generate_code(
>     lambda _: insert_pdb
> )
> 
> # Or alternatively, registers a code transformer which first
> # runs `body` through existing registered transformer, then
> # through `insert_pdb`:
> gm.graph.on_generate_code(
>     lambda current_trans: (
>         lambda body: insert_pdb(
>             current_trans(body) if current_trans
>             else body
>         )
>     )
> )
> 
> gm.recompile()
> gm(*inputs)  # drops into pdb
> 
> ```
> 
> 
> 
> 
>  This function can also be used as a context manager, with the benefit to
> automatically restores the previously registered code transformer:
>  
> 
> 
> 
> 
> 
> ```
> # ... continue from previous example
> 
> with gm.graph.on_generate_code(lambda _: insert_pdb):
>     # do more stuff with `gm`...
>     gm.recompile()
>     gm(*inputs)  # drops into pdb
> 
> # now previous code transformer is restored (but `gm`'s code with pdb
> # remains - that means you can run `gm` with pdb here too, until you
> # run next `recompile()`).
> 
> ```
> 
> 
> 
> 
> 


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


 输出


 ( *结果
* , *类型_expr



 =
 


 无
* ) [[source]](_modules/torch/fx/graph.html#Graph.output)[¶](#torch.fx.Graph.output "此定义的永久链接")


 将 `output``Node` 插入到 `Graph` 中。 “output”节点代表 Python 代码中的“return”语句。 `result` 是应该返回的值。


 参数 
* **result** ( *Argument
* ) – 要返回的值。
* **type_expr** ( *Optional
* *[
* *Any
* *]
* ) – 表示 Python 类型的可选类型注释该节点的输出将有。



!!! note "笔记"

    相同的插入点和类型表达式规则适用于此方法，如“Graph.create_node”。


!!! note "笔记"

    保证此 API 的向后兼容性。


 占位符


 ( *name
* , *type_expr=None
* , *default_value
* ) [[source]](_modules/torch/fx/graph.html#Graph.placeholder)[¶](#torch.fx.Graph.占位符"此定义的永久链接")


 将“placeholder”节点插入到图表中。 “占位符”代表函数输入。


 参数 
* **name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 输入的名称价值。这对应于这个 `Graph` 所代表的函数的位置参数的名称。
* **type_expr** ( *Optional
* *[
* *Any
* *]
* ) – 一个可选类型注释，表示 Python 类型的输出这个节点会有。在某些情况下，需要正确生成代码(例如，当随后在 TorchScript 编译中使用该函数时)。
* **default_value** ( *Any
* ) – 该函数参数应采用的默认值。注意：要允许 None 作为默认值，应将 inform.Signature.empty 作为此参数传递，以指定该参数没有默认值。


 Return type


[*节点*](#torch.fx.Node "torch.fx.node.Node")




!!! note "笔记"

    相同的插入点和类型表达式规则适用于此方法，如“Graph.create_node”。


!!! note "笔记"

    保证此 API 的向后兼容性。


 打印_表格


 ( ) [[source]](_modules/torch/fx/graph.html#Graph.print_tabular)[¶](#torch.fx.Graph.print_tabular "此定义的永久链接")


 以表格格式打印图形的中间表示。请注意，此 API 需要安装“tabulate”模块。




!!! note "笔记"

    保证此 API 的向后兼容性。


 过程_输入


 (
 
*\*
 


 args
* ) [[source]](_modules/torch/fx/graph.html#Graph.process_inputs)[¶](#torch.fx.Graph.process_inputs "此定义的永久链接")


 处理参数，以便将它们传递到 FX 图表。


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


 过程_输出


 ( *out
* ) [[source]](_modules/torch/fx/graph.html#Graph.process_outputs)[¶](#torch.fx.Graph.process_outputs "此定义的永久链接")


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


 python_code


 ( *root_module
* , *** , *详细



 =
 


 False
* ) [[source]](_modules/torch/fx/graph.html#Graph.python_code)[¶](#torch.fx.Graph.python_code "此定义的永久链接")


 将此“Graph”转换为有效的 Python 代码。


 Parameters


**root_module** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 根的名称用于查找限定名称目标的模块。这通常是“自我”。


 退货


 src：代表 objectglobals 的 Python 源代码：src 中的全局名称字典 -
> 它们引用的对象。


 Return type


 一个 PythonCode 对象，由两个字段组成



!!! note "笔记"

    保证此 API 的向后兼容性。


 设置_codegen


 ( *codegen
* ) [[source]](_modules/torch/fx/graph.html#Graph.set_codegen)[¶](#torch.fx.Graph.set_codegen "此定义的永久链接")


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


*班级*


 火炬.fx。


 ( *graph
* 、 *name
* 、 *op
* 、 *target
* 、 *args
* 、 *kwargs
* 、 *return_type



 =
 


 无
* ) [[source]](_modules/torch/fx/node.html#Node)[¶](#torch.fx.Node "此定义的永久链接")


“Node”是表示“Graph”中各个操作的数据结构。在大多数情况下，节点表示各种实体的调用点，例如运算符、方法和模块(一些例外包括指定函数输入和输出的节点)。每个“Node”都有一个由其“op”属性指定的函数。每个“op”值的“Node”语义如下：



* `placeholder` 代表函数输入。 “name”属性指定该值将采用的名称。 `target` 类似地是参数的名称。 `args` 包含：1) 没有任何内容，或者 2) 表示函数输入的默认参数的单个参数。 `kwargs` 是不在乎。占位符对应于图形打印输出中的函数参数(例如 `x` )。
* `get_attr` 从模块层次结构中检索参数。类似地，“name”是分配给 thefetch 结果的名称。 `target` 是模块层次结构中参数位置的完全限定名称。 `args` 和 `kwargs` 是无关的
* `call_function` 将自由函数应用于某些值。 `name` 类似地是要分配的值的名称。 `target` 是要应用的函数。 `args` 和 `kwargs` 表示函数的参数，遵循 Python 调用约定
* `call_module` 将模块层次结构中的 `forward()` 方法中的模块应用于给定的参数。 `name` 与之前相同。 `target` 是模块层次结构中要调用的模块的完全限定名称。 `args` 和 `kwargs` 表示调用模块的参数，*不包括 self 参数*。
* `call_method` 调用值的方法。 `name` 也很相似。 “target”是应用于“self”参数的方法的字符串名称。 `args` 和 `kwargs` 表示调用模块的参数，*包括 self 参数** `output` 在其 `args[0]` 属性中包含跟踪函数的输出。这对应于图表打印输出中的“return”语句。




!!! note "笔记"

    保证此 API 的向后兼容性。


*财产*


 所有输入节点 *:


[列表](https://docs.python.org/3/library/typing.html#typing.List“(在 Python v3.12 中)”)


 [ [节点](#torch.fx.Node "torch.fx.node.Node")


 ]*[¶](#torch.fx.Node.all_input_nodes "此定义的永久链接")


 返回作为该节点输入的所有节点。这相当于迭代“args”和“kwargs”并仅收集节点值。


 退货


 按此顺序出现在该 Node 的 args 和 kwargs 中的 Node 列表。


 附加


 ( *x
* ) [[source]](_modules/torch/fx/node.html#Node.append)[¶](#torch.fx.Node.append "此定义的永久链接")


 在图中节点列表中的该节点后面插入“x”。相当于“self.next.prepend(x)”


 Parameters


**x** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – 放置在此节点之后的节点。必须是同一图的成员。



!!! note "笔记"

    保证此 API 的向后兼容性。


*财产*


 参数*：


[元组](https://docs.python.org/3/library/typing.html#typing.Tuple“(在Python v3.12中)”)


 [ [可选](https://docs.python.org/3/library/typing.html#typing.Optional“(在 Python v3.12 中)”)


 [ [Union](https://docs.python.org/3/library/typing.html#typing.Union“(在 Python v3.12 中)”)


 [[元组](https://docs.python.org/3/library/typing.html#typing.Tuple“(在Python v3.12中)”)


 [ [任何](https://docs.python.org/3/library/typing.html#typing.Any“(Python v3.12)”)


 ,
 


...
 



 ]
 



 ,
 


[列表](https://docs.python.org/3/library/typing.html#typing.List“(在 Python v3.12 中)”)


 [ [任何](https://docs.python.org/3/library/typing.html#typing.Any“(Python v3.12)”)


 ]
 



 ,
 


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[任何](https://docs.python.org/3/library/typing.html#typing.Any“(Python v3.12)”)


 ]
 



 ,
 


[slice](https://docs.python.org/3/library/functions.html#slice“(Python v3.12)”)


 ,
 


[范围](https://docs.python.org/3/library/stdtypes.html#range“(在Python v3.12中)”)


 ,
 


[节点](#torch.fx.Node“torch.fx.node.Node”)


 ,
 


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 ,
 


[float](https://docs.python.org/3/library/functions.html#float“(在Python v3.12中)”)


 ,
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 ,
 


[复杂](https://docs.python.org/3/library/functions.html#complex“(在Python v3.12中)”)


 ,
 


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")


 ,
 


[tensor](tensors.html#torch.Tensor "torch.Tensor")


 ,
 


[设备](tensor_attributes.html#torch.device "torch.device")


 ,
 


[内存_格式](tensor_attributes.html#torch.memory_format "torch.memory_format")


 ,
 


[布局](tensor_attributes.html#torch.layout“torch.layout”)


 ,
 


 操作过载


 ]
 



 ]
 



 ,
 


...
 


 ]*[¶](#torch.fx.Node.args"此定义的永久链接")


 这个 `Node` 的参数元组。参数的解释取决于节点的操作码。有关更多信息，请参阅 [`Node`](#torch.fx.Node "torch.fx.Node") 文档字符串。


 允许分配给该属性。所有使用和用户的统计都会在分配时自动更新。


 格式_node


 ( *占位符_名称



 =
 


 无
* , *也许_return_typename



 =
 


 无
* ) [[source]](_modules/torch/fx/node.html#Node.format_node)[¶](#torch.fx.Node.format_node "此定义的永久链接")


 返回 `self` 的描述性字符串表示形式。


 此方法可以不带任何参数用作调试实用程序。


 该函数也在 `Graph` 的 `__str__` 方法内部使用。 `placeholder_names` 和 `maybe_return_typename` 中的字符串一起构成了该图周围 GraphModule 中自动生成的 `forward` 函数的签名。否则不应使用 `placeholder_names` 和 `maybe_return_typename`。


 参数 
* **占位符_names** ( [*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(in Python v3.12)")*[
* [*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)")*[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* *]
* ) – 一个列表，用于存储表示生成的 `forward` 函数中的占位符的格式化字符串。仅供内部使用。
* **也许_return_typename** ( [*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中) ")*[
* [*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)")*[
* [*str*] (https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* *]
* ) – 一个单元素列表，将存储表示的格式化字符串生成的“forward”函数的输出。仅供内部使用。


 退货


 如果 1) 我们使用 `format_node` 作为内部助手


 在 `Graph` 的 `__str__` 方法中，2) `self` 是占位符 Node，返回 `None` 。否则，返回当前节点的描述性字符串表示形式。


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)




!!! note "笔记"

    保证此 API 的向后兼容性。


 是_不纯的


 ( ) [[source]](_modules/torch/fx/node.html#Node.is_impure)[¶](#torch.fx.Node.is_impure "此定义的永久链接")


 返回此操作是否是不纯的，即它的操作是否是占位符或输出，或者 call_function 或 call_module 是否是不纯的。


 退货


 操作是否不纯。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


*财产*


 夸格斯*：


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[可选](https://docs.python.org/3/library/typing.html#typing.Optional“(在 Python v3.12 中)”)


 [ [Union](https://docs.python.org/3/library/typing.html#typing.Union“(在 Python v3.12 中)”)


 [[元组](https://docs.python.org/3/library/typing.html#typing.Tuple“(在Python v3.12中)”)


 [ [任何](https://docs.python.org/3/library/typing.html#typing.Any“(Python v3.12)”)


 ,
 


...
 



 ]
 



 ,
 


[列表](https://docs.python.org/3/library/typing.html#typing.List“(在 Python v3.12 中)”)


 [ [任何](https://docs.python.org/3/library/typing.html#typing.Any“(Python v3.12)”)


 ]
 



 ,
 


[Dict](https://docs.python.org/3/library/typing.html#typing.Dict“(Python v3.12)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[任何](https://docs.python.org/3/library/typing.html#typing.Any“(Python v3.12)”)


 ]
 



 ,
 


[slice](https://docs.python.org/3/library/functions.html#slice“(Python v3.12)”)


 ,
 


[范围](https://docs.python.org/3/library/stdtypes.html#range“(在Python v3.12中)”)


 ,
 


[节点](#torch.fx.Node“torch.fx.node.Node”)


 ,
 


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ,
 


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 ,
 


[float](https://docs.python.org/3/library/functions.html#float“(在Python v3.12中)”)


 ,
 


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 ,
 


[复杂](https://docs.python.org/3/library/functions.html#complex“(在Python v3.12中)”)


 ,
 


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")


 ,
 


[tensor](tensors.html#torch.Tensor "torch.Tensor")


 ,
 


[设备](tensor_attributes.html#torch.device "torch.device")


 ,
 


[内存_格式](tensor_attributes.html#torch.memory_format "torch.memory_format")


 ,
 


[布局](tensor_attributes.html#torch.layout“torch.layout”)


 ,
 


 操作过载


 ]
 



 ]
 


 ]*[¶](#torch.fx.Node.kwargs"此定义的永久链接")


 这个 `Node` 的关键字参数的字典。参数的解释取决于节点的操作码。有关更多信息，请参阅 [`Node`](#torch.fx.Node "torch.fx.Node") 文档字符串。


 允许分配给该属性。所有使用和用户的统计都会在分配时自动更新。


*财产*


 下一个 *：


[Node](#torch.fx.Node "torch.fx.node.Node")*[¶](#torch.fx.Node.next "此定义的永久链接")


 返回节点链接列表中的下一个“节点”。


 退货


 节点链表中的下一个“节点”。


 规范化_参数


 ( *root
* , *arg_types



 =
 


 无
* , *kwarg_types



 =
 


 无
* , *标准化_为_仅_使用_kwargs



 =
 


 False
* ) [[source]](_modules/torch/fx/node.html#Node.normalized_arguments)[¶](#torch.fx.Node.normalized_arguments "此定义的永久链接")


 返回 Python 目标的规范化参数。这意味着 args/kwargs 将与模块/功能的签名相匹配，并按位置顺序专门返回 kwargs(如果 normalize_to_only_use_kwargs 为 true)。还填充默认值。不支持纯位置参数或可变参数。


 支持模块调用。


 可能需要 arg_types 和 kwarg_types 来消除重载的歧义。


 参数 
* **root** ( [*torch.nn.Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 用于解析模块目标的模块.
* **arg_types** ( *可选
* *[
* *Tuple
* *[
* *Any
* *]
* *]
* ) – args
* 的 arg 类型元组 **kwarg_types** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* 
* Any
* *]
* *]
* ) – kwargs
* **normalize_to_only_use_kwargs** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否标准化为仅使用 kwargs。


 退货


 返回 NamedTuple ArgsKwargsPair，如果不成功则返回 None。


 Return type


[*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中)") [ *ArgsKwargsPair
* ]


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


 前置


 ( *x
* ) [[source]](_modules/torch/fx/node.html#Node.prepend)[¶](#torch.fx.Node.prepend "此定义的永久链接")


 在图中的节点列表中的该节点之前插入 x。例子：


```
Before: p -> self
        bx -> x -> ax
After:  p -> x -> self
        bx -> ax

```


 Parameters


**x** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – 要放置在此节点之前的节点。必须是同一图的成员。



!!! note "笔记"

    保证此 API 的向后兼容性。


*财产*


 上一页 *:


[Node](#torch.fx.Node "torch.fx.node.Node")*[¶](#torch.fx.Node.prev "此定义的永久链接")


 返回节点链接列表中的前一个“Node”。


 退货


 节点链表中的前一个“节点”。


 将_所有_使用_替换为


 ( *replace_with
* , *delete_user_cb=<function Node.<lambda>>
* , *** , *propagate_meta=False
* ) [[source]](_modules/torch/fx/node.html#Node.replace_all_uses_with)[¶](#torch.fx.Node.replace_all_uses_with "此定义的永久链接")


 将图中所有使用的 `self` 替换为节点 `replace_with` 。


 参数 
* **replace_with** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – 用于替换 `self` 的所有使用的节点。
* **删除_user _cb** ( *Callable
* ) – 调用回调以确定是否应删除自身节点的给定用户。
* **propagate_meta** ( [*bool*](https://docs.python. org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否将原始节点的.meta 字段上的所有属性复制到替换节点上。出于安全考虑，仅此有效如果替换节点还没有现有的.meta 字段，则执行此操作。


 退货


 进行此更改的节点列表。


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*Node*](#torch.fx.Node “火炬.fx.node.Node”)]



!!! note "笔记"

    保证此 API 的向后兼容性。


 将_输入_替换为


 ( *old_i​​nput
* , *new_input
* ) [[source]](_modules/torch/fx/node.html#Node.replace_input_with)[¶](#torch.fx.Node.replace_input_with "此定义的永久链接")


 循环遍历 `self` 的输入节点，并将 `old_i​​nput` 的所有实例替换为 `new_input` 。


 参数 
* **old_i​​nput** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – 要替换的旧输入节点。
* **new_input** ( [ *Node*](#torch.fx.Node "torch.fx.Node") ) – 替换 `old_i​​nput` 的新输入节点。



!!! note "笔记"

    保证此 API 的向后兼容性。


*财产*


 堆栈跟踪 *：


[可选](https://docs.python.org/3/library/typing.html#typing.Optional“(在 Python v3.12 中)”)


 [ [str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 ]*[¶](#torch.fx.Node.stack_trace "此定义的永久链接")


 返回跟踪期间记录的 Python 堆栈跟踪(如果有)。当使用 fx.Tracer 跟踪时，此属性通常由 Tracer.create_proxy 填充。要在跟踪期间记录堆栈跟踪以用于调试目的，请在 Tracer 实例上设置 record_stack_traces = True。当使用 dynamo 进行跟踪时，默认情况下将由 OutputGraph.create_proxy 填充此属性。


 stack_trace 将在字符串末尾有最里面的帧。


 更新_arg


 ( *idx
* , *arg
* ) [[source]](_modules/torch/fx/node.html#Node.update_arg)[¶](#torch.fx.Node.update_arg "此定义的永久链接")


 更新现有位置参数以包含新值 `arg` 。调用后，`self.args[idx] == arg`。


 参数 
* **idx** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – `self 的索引要更新的元素的.args` 
* **arg** ( *Argument
* ) – 要写入 `args` 的新参数值



!!! note "笔记"

    保证此 API 的向后兼容性。


 更新_kwarg


 ( *key
* , *arg
* ) [[source]](_modules/torch/fx/node.html#Node.update_kwarg)[¶](#torch.fx.Node.update_kwarg "此定义的永久链接")


 更新现有关键字参数以包含新值 `arg` 。调用后， `self.kwargs[key] == arg` 。


 参数 
* **key** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – `self 中的键要更新的元素的.kwargs` 
* **arg** ( *Argument
* ) – 要写入 `kwargs` 的新参数值



!!! note "笔记"

    保证此 API 的向后兼容性。


*班级*


 火炬.fx。


 示踪剂


 ( *autowrap_modules



 =
 


 (数学,)
* , *autowrap_functions



 =
 


 ()
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#Tracer)[¶](#torch.fx.Tracer "此定义的永久链接")



> 
> 
> 
> `Tracer`
> 是实现 `torch.fx.symbolic_trace`
> 符号跟踪功能的类。对
> `symbolic_trace(m)`
> 的调用相当于
> `Tracer().trace(m)`
> 。
> 
> 
> 
> 
> Tracer 可以被子类化以覆盖跟踪进程的各种行为。可以重写的不同行为在此类方法的文档字符串中进行了描述。
> 
> 
> 
> >




!!! note "笔记"

    保证此 API 的向后兼容性。


 调用_模块


 ( *m
* , *forward
* , *args
* , *kwargs
* ) [[来源]](_modules/torch/fx/_symbolic_trace.html#Tracer.call_module)[¶](#torch.fx.Tracer.call_module "此定义的永久链接")


 指定此“Tracer”在遇到对“nn.Module”实例的调用时的行为的方法。


 默认情况下，行为是通过 `is_leaf_module` 检查被调用的模块是否是叶模块。如果是，则发出一个引用“Graph”中的“m”的“call_module”节点。否则，正常调用“Module”，跟踪其“forward”函数中的操作。


 例如，可以重写此方法来创建嵌套的跟踪图形模块，或者在跨“模块”边界进行跟踪时您想要的任何其他行为。


 参数 
* **m** ( [*Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 正在发出调用的模块
* 
* *forward** ( *Callable
* ) – 要调用的`Module`的forward()方法
* **args** ( *Tuple
* ) – 模块调用点的参数
* **kwargs** ( *Dict
* ) – 模块调用点的 kwargs


 退货


 模块调用的返回值。在发出“call_module”节点的情况下，这是一个“Proxy”值。否则，它是从“Module”调用返回的任何值。


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)




!!! note "笔记"

    保证此 API 的向后兼容性。


 创建_arg


 ( *a
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#Tracer.create_arg)[¶](#torch.fx.Tracer.create_arg "此定义的永久链接")


 一种在准备用作“Graph”中节点参数的值时指定跟踪行为的方法。


 默认情况下，该行为包括：


1. 迭代集合类型(例如元组、列表、字典)并在元素上递归调用 `create_args`。2.给定一个 Proxy 对象，返回对底层 IR `Node`3 的引用。给定一个非代理tensor对象，针对各种情况发出 IR：



> 
> 
> 
> 
* 对于参数，发出一个引用该参数的 
> `get_attr`
> 节点
> 
* 对于非参数tensor，将tensor存储在引用该属性的特殊 
> 属性中。>>


 可以重写此方法以支持更多类型。


 Parameters


**a** ( *Any
* ) – 作为 `Graph` 中的 `Argument` 发出的值。


 退货


 值“a”转换为适当的“Argument”


 Return type


[*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中)") [ [*Union*](https://docs. python.org/3/library/typing.html#typing.Union "(Python v3.12)") [ [*Tuple*](https://docs.python.org/3/library/typing.html# Typing.Tuple "(Python v3.12)") [ [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)" ) , …], [*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*Any*](https ://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)") ], [*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12 中)") [ [str](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12 中) )") , [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)") ], [slice](https:///docs.python.org/3/library/functions.html#slice "(Python v3.12)") , [范围](https://docs.python.org/3/library/stdtypes.html#range "(Python v3.12)") , [Node](#torch.fx.Node "torch.fx.Node") , [str](https://docs.python.org/3/library/stdtypes. html#str "(Python v3.12)") , [int](https://docs.python.org/3/library/functions.html#int "(Python v3.12)") , [float ](https://docs.python.org/3/library/functions.html#float "(Python v3.12)") , [bool](https://docs.python.org/3/library/function.html#bool "(Python v3.12)") , [complex](https://docs.python.org/3/library/functions.html#complex "(Python v3.12)") , [*dtype*](tensor_attributes.html#torch.dtype "torch.dtype") , [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") , [*device*](tensor_attributes.html# torch.device "torch.device") , [*memory_format*](tensor_attributes.html#torch.memory_format "torch.memory_format") , [*layout*](tensor_attributes.html#torch.layout "torch.layout" ) , *OpOverload
* ]]



!!! note "笔记"

    保证此 API 的向后兼容性。


 创建_args_for_root


 ( *root_fn
* , *is_module
* , *concrete_args



 =
 


 无
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#Tracer.create_args_for_root)[¶](#torch.fx.Tracer.create_args_for_root "此定义的永久链接")


 创建与“root”模块的签名相对应的“placeholder”节点。此方法内省 root 的签名并相应地发出这些节点，还支持 `*args` 和 `**kwargs` 。


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


 创建_node


 (*种类*，*目标*，*参数*，*kwargs*，*名称



 =
 


 无
* , *类型_expr



 =
 


 无
* ) [¶](#torch.fx.Tracer.create_node "此定义的永久链接")


 插入给定目标、参数、kwargs 和名称的图形节点。


 可以重写此方法以对节点创建中使用的值进行额外的检查、验证或修改。例如，人们可能希望禁止记录就地操作。




!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


[*节点*](#torch.fx.Node "torch.fx.node.Node")


 创建_代理


 (*种类*，*目标*，*参数*，*kwargs*，*名称



 =
 


 无
* , *类型_expr



 =
 


 无
* , *proxy_factory_fn



 =
 


 无
* ) [¶](#torch.fx.Tracer.create_proxy "此定义的永久链接")


 从给定的参数创建一个 Node，然后返回包装在 Proxy 对象中的 Node。


 如果 kind = ‘placeholder’，那么我们将创建一个代表函数参数的节点。如果我们需要对默认参数进行编码，我们可以使用“args”元组。对于“占位符”节点，“args”否则为空。




!!! note "笔记"

    保证此 API 的向后兼容性。


 获取属性


 ( *attr
* , *attr_val
* , *parameter_proxy_cache
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#Tracer.getattr)[¶](#torch.fx.Tracer. getattr"此定义的永久链接")


 当我们调用 getattron 来调用“nn.Module”实例时，指定此“Tracer”行为的方法。


 默认情况下，行为是返回属性的代理值。它还将代理值存储在“parameter_proxy_cache”中，以便将来的调用将重用该代理，而不是创建一个新代理。


 可以重写此方法，例如在查询参数时不返回代理。


 参数 
* **attr** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 属性的名称被查询
* **attr_val** ( *Any
* ) – 属性的值
* **参数_proxy_cache** ( *Dict
* *[
* [*str*](https://docs. python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *Any
* *]
* ) – 代理的属性名称缓存


 退货


 getattr 调用的返回值。


!!! warning "警告"

     该 API 是实验性的，*不*向后兼容。


 是_leaf_module


 ( *m
* , *module_qualified_name
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#Tracer.is_leaf_module)[¶](#torch.fx.Tracer.is_leaf_module "此定义的永久链接")


 指定给定“nn.Module”是否为“叶”模块的方法。


 叶模块是出现在 IR 中的原子单元，由“call_module”调用引用。默认情况下，PyTorch 标准库命名空间 (torch.nn) 中的模块是叶模块。除非通过此参数另有指定，否则将跟踪所有其他模块并记录其组成操作。


 参数 
* **m** ( [*Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module") ) – 正在查询的模块
* **module_qualified _name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 该模块根目录的路径。例如，如果您有一个模块层次结构，其中子模块“foo”包含子模块“bar”，子模块“bar”又包含子模块“baz”，则该模块将在此处以限定名称“foo.bar.baz”显示。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)




!!! note "笔记"

    保证此 API 的向后兼容性。



 iter
 


 ( *obj
* ) [¶](#torch.fx.Tracer.iter "此定义的永久链接")


 当代理对象被迭代时调用，例如


 当用于控制流时。通常我们不知道该怎么做，因为我们不知道代理的值，但是自定义跟踪器可以使用 create_node 将更多信息附加到图节点，并且可以选择返回迭代器。



!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


[*迭代器*](https://docs.python.org/3/library/typing.html#typing.Iterator“(在Python v3.12中)”)


 keys
 


 ( *obj
* ) [¶](#torch.fx.Tracer.keys "此定义的永久链接")


 当代理对象调用了 keys() 方法时调用。


 这就是在代理上调用 ** 时发生的情况。这应该返回迭代器，它** 应该在您的自定义跟踪器中工作。



!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


 模块的路径


 ( *mod
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#Tracer.path_of_module)[¶](#torch.fx.Tracer.path_of_module "此定义的永久链接")


 在 `root` 的模块层次结构中查找 `mod` 的限定名称的帮助程序方法。例如，如果 `root` 有一个名为 `foo` 的子模块，该子模块有一个名为 `bar` 的子模块，则将 `bar` 传递给该函数将返回字符串“foo.bar”。


 Parameters


**mod** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 用于检索的 `Module`的限定名称。


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)




!!! note "笔记"

    保证此 API 的向后兼容性。



 proxy
 


 ( *node
* ) [¶](#torch.fx.Tracer.proxy "此定义的永久链接")




!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


[*代理*](#torch.fx.Proxy "torch.fx.proxy.Proxy")


 到_bool


 ( *obj
* ) [¶](#torch.fx.Tracer.to_bool "此定义的永久链接")


 当代理对象转换为布尔值时调用，例如


 当用于控制流时。通常我们不知道该怎么做，因为我们不知道代理的值，但是自定义跟踪器可以使用 create_node 将更多信息附加到图节点，并可以选择返回一个值。



!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 trace
 


 ( *root
* , *concrete_args



 =
 


 无
* ) [[source]](_modules/torch/fx/_symbolic_trace.html#Tracer.trace)[¶](#torch.fx.Tracer.trace "此定义的永久链接")


 跟踪“root”并返回相应的 FX“Graph”表示。 `root` 可以是一个 `nn.Module` 实例，也可以是一个 Python 可调用对象。


 请注意，在此调用之后，“self.root”可能与此处传入的“root”不同。例如，当一个自由函数传递给 `trace()` 时，我们将创建一个 `nn.Module` 实例作为根并添加嵌入常量。


 参数 
* **root** ( *Union
* *[
* [*Module*](generated/torch.nn.Module.html#torch.nn.Module "torch.nn.Module")*,
* *Callable
* 
* ]
* ) – 要跟踪的“模块”或函数。保证此参数的向后兼容性。
* **concrete_args** ( *可选
* *[
* *Dict
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *any
* *]
* *]
* ) – 不应被视为代理的具体参数。此参数是实验性的，*不*保证其向后兼容性。


 退货


 表示传入的“root”语义的“Graph”。


 Return type


[*图*](#torch.fx.Graph "torch.fx.graph.Graph")




!!! note "笔记"

    保证此 API 的向后兼容性。


*班级*


 火炬.fx。



 Proxy
 


 (*节点*，*跟踪器



 =
 


 无
* ) [[source]](_modules/torch/fx/proxy.html#Proxy)[¶](#torch.fx.Proxy "此定义的永久链接")


“Proxy”对象是“Node”包装器，在符号跟踪期间流经程序并记录它们接触不断增长的 FX 图表的所有操作(“torch”函数调用、方法调用、运算符)。


 如果您正在进行图形转换，您可以将自己的“Proxy”方法包装在原始“Node”周围，以便您可以使用重载运算符向“Graph”添加其他内容。


`Proxy` 对象不能被迭代。换句话说，如果在循环中使用“Proxy”或作为“*args”/“**kwargs”函数参数，符号跟踪器将抛出错误。


 解决这个问题有两种主要方法：1。将无法追踪的逻辑分解为顶级函数并在其上使用“fx.wrap”。2。如果控制流是静态的(即循环行程计数基于某些超参数)，则代码可以保留在其原始位置并重构为如下所示：


```
for i in range(self.some_hyperparameter):
    indexed_item = proxied_value[i]

```


 有关代理内部结构的更详细描述，请查看 torch/fx/OVERVIEW.md 中的“代理”部分


!!! note "笔记"

    保证此 API 的向后兼容性。


*班级*


 火炬.fx。


 口译员


 (*模块*，*垃圾_收集_值



 =
 


 True
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter)[¶](#torch.fx.Interpreter "此定义的永久链接")


 解释器逐节点执行 FX 图。这种模式对于很多事情都很有用，包括编写代码转换以及分析过程。


 可以重写 Interpreter 类中的方法来自定义执行行为。根据调用层次结构可重写方法的映射：


```
run()
    +-- run_node
        +-- placeholder()
        +-- get_attr()
        +-- call_function()
        +-- call_method()
        +-- call_module()
        +-- output()

```


 例子


 假设我们想要将“torch.neg”的所有实例与“torch.sigmoid”交换，反之亦然(包括它们的“Tensor”方法等效项)。我们可以像这样子类化 Interpreter：


```
class NegSigmSwapInterpreter(Interpreter):
    def call_function(self, target : Target,
                      args : Tuple, kwargs : Dict) -> Any:
        if target == torch.sigmoid:
            return torch.neg(*args, **kwargs)
        return super().call_function(n)

    def call_method(self, target : Target,
                    args : Tuple, kwargs : Dict) -> Any:
        if target == 'neg':
            call_self, *args_tail = args
            return call_self.sigmoid(*args_tail, **kwargs)
        return super().call_method(n)

def fn(x):
    return torch.sigmoid(x).neg()

gm = torch.fx.symbolic_trace(fn)
input = torch.randn(3, 4)
result = NegSigmSwapInterpreter(gm).run(input)
torch.testing.assert_close(result, torch.neg(input).sigmoid())

```


 参数 
* **module** ( [*GraphModule*](#torch.fx.GraphModule "torch.fx.GraphModule") ) – 要执行的模块
* **garbage_collect_values** ( [*bool
* ](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否在模块执行中最后一次使用后删除值。这确保了执行期间的最佳内存使用。例如，可以禁用此功能，通过查看“Interpreter.env”属性来检查执行中的所有中间值。



!!! note "笔记"

    保证此 API 的向后兼容性。


 盒装_run


 ( *args_list
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.boxed_run)[¶](#torch.fx.Interpreter.boxed_run "此定义的永久链接")


 通过解释运行模块并返回结果。这使用“装箱”调用约定，您可以在其中传递参数列表，该列表将由解释器清除。这确保了输入tensor被及时释放。




!!! note "笔记"

    保证此 API 的向后兼容性。


 调用_函数


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.call_function)[¶](#torch.fx.Interpreter.call_function "此定义的永久链接")


 执行`call_function`节点并返回结果。


 参数 
* **target** ( *Target
* ) – 此节点的调用目标。有关语义的详细信息，请参阅 [Node](https://pytorch.org/docs/master/fx.html#torch.fx.Node)
* **args** ( *Tuple
* ) – 此调用的位置参数元组
* **kwargs** ( *Dict
* ) – 此调用的关键字参数的字典


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


 返回


 Any：函数调用返回的值



!!! note "笔记"

    保证此 API 的向后兼容性。


 调用_方法


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.call_method)[¶](#torch.fx.Interpreter.call_method "此定义的永久链接")


 执行`call_method`节点并返回结果。


 参数 
* **target** ( *Target
* ) – 此节点的调用目标。有关语义的详细信息，请参阅 [Node](https://pytorch.org/docs/master/fx.html#torch.fx.Node)
* **args** ( *Tuple
* ) – 此调用的位置参数元组
* **kwargs** ( *Dict
* ) – 此调用的关键字参数的字典


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


 返回


 Any：方法调用返回的值



!!! note "笔记"

    保证此 API 的向后兼容性。


 调用_模块


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.call_module)[¶](#torch.fx.Interpreter.call_module "此定义的永久链接")


 执行`call_module`节点并返回结果。


 参数 
* **target** ( *Target
* ) – 此节点的调用目标。有关语义的详细信息，请参阅 [Node](https://pytorch.org/docs/master/fx.html#torch.fx.Node)
* **args** ( *Tuple
* ) – 此调用的位置参数元组
* **kwargs** ( *Dict
* ) – 此调用的关键字参数的字典


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


 返回


 Any：模块调用返回的值



!!! note "笔记"

    保证此 API 的向后兼容性。


 从 _env 获取_args_kwargs_


 ( *n
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.fetch_args_kwargs_from_env)[¶](#torch.fx.Interpreter.fetch_args_kwargs_from_env "此定义的永久链接")


 从当前执行环境中获取节点“n”的“args”和“kwargs”的具体值。


 Parameters


**n** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – 应获取 `args` 和 `kwargs` 的节点。


 退货


`args` 和 `kwargs` 以及 `n` 的具体值。


 Return type


 元组[元组，字典]



!!! note "笔记"

    保证此 API 的向后兼容性。


 获取_attr


 ( *target
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.fetch_attr)[¶](#torch.fx.Interpreter.fetch_attr "此定义的永久链接")


 从 `self.module` 的 `Module` 层次结构中获取属性。


 Parameters


**target** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 的完全限定名称要获取的属性


 退货


 属性的值。


 Return type


 Any
 



!!! note "笔记"

    保证此 API 的向后兼容性。


 获取_attr


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.get_attr)[¶](#torch.fx.Interpreter.get_attr "此定义的永久链接")


 执行 `get_attr` 节点。将从 `self.module` 的 `Module` 层次结构中检索属性值。


 参数 
* **target** ( *Target
* ) – 此节点的调用目标。有关语义的详细信息，请参阅 [Node](https://pytorch.org/docs/master/fx.html#torch.fx.Node)
* **args** ( *Tuple
* ) – 此调用的位置参数元组
* **kwargs** ( *Dict
* ) – 此调用的关键字参数的字典


 退货


 检索到的属性的值


 Return type


 Any
 



!!! note "笔记"

    保证此 API 的向后兼容性。


 将节点映射到值


 ( *args
* , *n
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.map_nodes_to_values)[¶](#torch.fx.Interpreter.map_nodes_to_values "此定义的永久链接")


 递归地遍历“args”并查找当前执行环境中每个“Node”的具体值。


 参数 
* **args** ( *Argument
* ) – 在其中查找具体值的数据结构
* **n** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – `args` 所属的节点。这仅用于错误报告。


 Return type


[*可选*](https://docs.python.org/3/library/typing.html#typing.Optional "(在 Python v3.12 中)") [ [*Union*](https://docs. python.org/3/library/typing.html#typing.Union "(Python v3.12)") [ [*Tuple*](https://docs.python.org/3/library/typing.html# Typing.Tuple "(Python v3.12)") [ [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)" ) , …], [*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [*Any*](https ://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)") ], [*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12 中)") [ [str](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12 中) )") , [*Any*](https://docs.python.org/3/library/typing.html#typing.Any "(Python v3.12)") ], [slice](https:///docs.python.org/3/library/functions.html#slice "(Python v3.12)") , [范围](https://docs.python.org/3/library/stdtypes.html#range "(Python v3.12)") , [*Node*](#torch.fx.Node "torch.fx.node.Node") , [str](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)") , [int](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中)" ) , [float](https://docs.python.org/3/library/functions.html#float "(Python v3.12)") , [bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)，[复杂](https://docs.python.org/3/library/functions.html#complex“(在Python v3.12中) )") , [*dtype*](tensor_attributes.html#torch.dtype "torch.dtype") , [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") , [*device*]( tensor_attributes.html#torch.device "torch.device") , [*memory_format*](tensor_attributes.html#torch.memory_format "torch.memory_format") , [*layout*](tensor_attributes.html#torch.layout " torch.layout") , *OpOverload
* ]]



!!! note "笔记"

    保证此 API 的向后兼容性。


 输出


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.output)[¶](#torch.fx.Interpreter.output "此定义的永久链接")


 执行“输出”节点。这实际上只是检索“输出”节点引用的值并返回它。


 参数 
* **target** ( *Target
* ) – 此节点的调用目标。有关语义的详细信息，请参阅 [Node](https://pytorch.org/docs/master/fx.html#torch.fx.Node)
* **args** ( *Tuple
* ) – 此调用的位置参数元组
* **kwargs** ( *Dict
* ) – 此调用的关键字参数的字典


 退货


 输出节点引用的返回值


 Return type


 Any
 



!!! note "笔记"

    保证此 API 的向后兼容性。


 占位符


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.placeholder)[¶](#torch.fx.Interpreter.placeholder "此定义的永久链接")


 执行“占位符”节点。请注意，这是有状态的：“Interpreter”维护一个传递给“run”的内部迭代器参数，并且该方法在该迭代器上返回next()。


 参数 
* **target** ( *Target
* ) – 此节点的调用目标。有关语义的详细信息，请参阅 [Node](https://pytorch.org/docs/master/fx.html#torch.fx.Node)
* **args** ( *Tuple
* ) – 此调用的位置参数元组
* **kwargs** ( *Dict
* ) – 此调用的关键字参数的字典


 退货


 检索到的参数值。


 Return type


 Any
 



!!! note "笔记"

    保证此 API 的向后兼容性。



 run
 


 (
 
*\*
 


 args
* , *初始_env



 =
 


 无
* , *启用_io_processing



 =
 


 True
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.run)[¶](#torch.fx.Interpreter.run "此定义的永久链接")


 通过解释运行模块并返回结果。


 参数 
* ***args** – 要运行的模块的参数，按位置顺序
* **initial_env** ( *可选
* *[
* *Dict
* *[
* [*Node*](#torch.fx.Node "torch.fx.Node")*,
* *Any
* *]
* *]
* ) – 可选的执行起始环境。这是一个将 Node 映射到任何值的字典。例如，这可以用于预填充某些节点的结果，以便在解释器中仅进行部分评估。
* **enable_io_processing** ( [*bool*](https://docs.python. org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果为 true，我们首先使用图的 process_inputs 和 process_outputs 函数处理输入和输出，然后再使用它们。


 退货


 执行模块返回的值


 Return type


 Any
 



!!! note "笔记"

    保证此 API 的向后兼容性。


 运行_node


 ( *n
* ) [[source]](_modules/torch/fx/interpreter.html#Interpreter.run_node)[¶](#torch.fx.Interpreter.run_node "此定义的永久链接")


 运行特定节点“n”并返回结果。根据“node.op”调用占位符、get_attr、call_function、call_method、call_module或输出


 Parameters


**n** ( [*Node*](#torch.fx.Node "torch.fx.Node") ) – 要执行的节点


 退货


 执行“n”的结果


 Return type


 Any
 



!!! note "笔记"

    保证此 API 的向后兼容性。


*班级*


 火炬.fx。


 变压器


 ( *module
* ) [[source]](_modules/torch/fx/interpreter.html#Transformer)[¶](#torch.fx.Transformer "此定义的永久链接")


`Transformer` 是一种特殊类型的解释器，它生成新的 `Module` 。它公开了一个“transform()”方法，该方法返回转换后的“Module”。 “Transformer”不需要像“Interpreter”那样运行参数。 “变形金刚”完全是象征性的。


 例子


 假设我们想要将“torch.neg”的所有实例与“torch.sigmoid”交换，反之亦然(包括它们的“Tensor”方法等效项)。我们可以像这样子类化“Transformer”：


```
class NegSigmSwapXformer(Transformer):
    def call_function(self, target : 'Target', args : Tuple[Argument, ...], kwargs : Dict[str, Any]) -> Any:
        if target == torch.sigmoid:
            return torch.neg(*args, **kwargs)
        return super().call_function(n)

    def call_method(self, target : 'Target', args : Tuple[Argument, ...], kwargs : Dict[str, Any]) -> Any:
        if target == 'neg':
            call_self, *args_tail = args
            return call_self.sigmoid(*args_tail, **kwargs)
        return super().call_method(n)

def fn(x):
    return torch.sigmoid(x).neg()

gm = torch.fx.symbolic_trace(fn)

transformed : torch.nn.Module = NegSigmSwapXformer(gm).transform()
input = torch.randn(3, 4)
torch.testing.assert_close(transformed(input), torch.neg(input).sigmoid())

```


 Parameters


**module** ( [*GraphModule*](#torch.fx.GraphModule "torch.fx.GraphModule") ) – 要转换的 `Module`。



!!! note "笔记"

    保证此 API 的向后兼容性。


 调用_函数


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Transformer.call_function)[¶](#torch.fx.Transformer.call_function "此定义的永久链接")




!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


 调用_模块


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Transformer.call_module)[¶](#torch.fx.Transformer.call_module "此定义的永久链接")




!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


 获取_attr


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Transformer.get_attr)[¶](#torch.fx.Transformer.get_attr "此定义的永久链接")


 执行 `get_attr` 节点。在 `Transformer` 中，它被重写以将新的 `get_attr` 节点插入到输出图中。


 参数 
* **target** ( *Target
* ) – 此节点的调用目标。有关语义的详细信息，请参阅 [Node](https://pytorch.org/docs/master/fx.html#torch.fx.Node)
* **args** ( *Tuple
* ) – 此调用的位置参数元组
* **kwargs** ( *Dict
* ) – 此调用的关键字参数的字典


 Return type


[*代理*](#torch.fx.Proxy "torch.fx.proxy.Proxy")




!!! note "笔记"

    保证此 API 的向后兼容性。


 占位符


 ( *target
* , *args
* , *kwargs
* ) [[source]](_modules/torch/fx/interpreter.html#Transformer.placeholder)[¶](#torch.fx.Transformer.placeholder "此定义的永久链接")


 执行“占位符”节点。在“Transformer”中，它被重写以将新的“占位符”插入到输出图中。


 参数 
* **target** ( *Target
* ) – 此节点的调用目标。有关语义的详细信息，请参阅 [Node](https://pytorch.org/docs/master/fx.html#torch.fx.Node)
* **args** ( *Tuple
* ) – 此调用的位置参数元组
* **kwargs** ( *Dict
* ) – 此调用的关键字参数的字典


 Return type


[*代理*](#torch.fx.Proxy "torch.fx.proxy.Proxy")




!!! note "笔记"

    保证此 API 的向后兼容性。


 转换


 ( ) [[source]](_modules/torch/fx/interpreter.html#Transformer.transform)[¶](#torch.fx.Transformer.transform "此定义的永久链接")


 转换 `self.module` 并返回转换后的 `GraphModule` 。




!!! note "笔记"

    保证此 API 的向后兼容性。


 Return type


[*GraphModule*](#torch.fx.GraphModule "torch.fx.graph_module.GraphModule")


 火炬.fx。


 替换_pattern


 ( *gm
* , *pattern
* , *replacement
* ) [[source]](_modules/torch/fx/subgraph_rewriter.html#replace_pattern)[¶](#torch.fx.replace_pattern "此定义的永久链接")


 匹配 GraphModule(`gm`) 图中所有可能的非重叠运算符集及其数据依赖关系 (`pattern`)，然后用另一个子图替换每个匹配的子图 (`replacement`)。


 参数 
* **gm** ( [*GraphModule*](#torch.fx.GraphModule "torch.fx.graph_module.GraphModule") ) – 包装要操作的 Graph 的 GraphModule
* **pattern** ( [
* Union*](https://docs.python.org/3/library/typing.html#typing.Union "(在 Python v3.12 中)")*[
* [*Callable*](https://docs. python.org/3/library/typing.html#typing.Callable "(Python v3.12)")*,
* [*GraphModule*](#torch.fx.GraphModule "torch.fx.graph_module.GraphModule") *]
* ) – 在 `gm` 中匹配用于替换的子图
* **替换** ( [*Union*](https://docs.python.org/3/library/typing.html#typing.Union " (在 Python v3.12 中)")*[
* [*Callable*](https://docs.python.org/3/library/typing.html#typing.Callable "(在 Python v3.12 中)")
* ,
* [*GraphModule*](#torch.fx.GraphModule "torch.fx.graph_module.GraphModule")*]
* ) – 用于替换 `pattern` 的子图


 退货


 “Match”对象的列表，表示原始图中与“pattern”匹配的位置。如果没有匹配项，则列表为空。 “匹配”定义为：


```
class Match(NamedTuple):
    # Node from which the match was found
    anchor: Node
    # Maps nodes in the pattern subgraph to nodes in the larger graph
    nodes_map: Dict[Node, Node]

```


 Return type


 列表[匹配]


 例子：


```
import torch
from torch.fx import symbolic_trace, subgraph_rewriter

class M(torch.nn.Module):
    def __init__(self):
        super().__init__()

    def forward(self, x, w1, w2):
        m1 = torch.cat([w1, w2]).sum()
        m2 = torch.cat([w1, w2]).sum()
        return x + torch.max(m1) + torch.max(m2)

def pattern(w1, w2):
    return torch.cat([w1, w2]).sum()

def replacement(w1, w2):
    return torch.stack([w1, w2])

traced_module = symbolic_trace(M())

subgraph_rewriter.replace_pattern(traced_module, pattern, replacement)

```


 上面的代码首先会匹配“traced_module”的“forward”方法中的“pattern”。模式匹配是基于 use-def 关系而不是节点名称来完成的。例如，如果“pattern”中有“p = torch.cat([a, b])”，则可以在原始“forward”函数中匹配“m = torch.cat([a, b])”，尽管变量名称不同(“p”与“m”)。


 “pattern”中的“return”语句仅根据其值进行匹配；它可能与大图中的“return”语句匹配，也可能不匹配。换句话说，该模式不必延伸到较大图表的末尾。


 当模式匹配时，它将从较大的函数中删除并用 `replacement` 替换。如果较大的函数中存在多个“pattern”匹配，则每个不重叠的匹配都将被替换。在匹配重叠的情况下，重叠匹配集中第一个找到的匹配将被替换。(这里的“第一个”被定义为节点的 use-def 关系的拓扑排序中的第一个。在大多数情况下，第一个节点是直接出现在 `self` 之后的参数，而最后一个 Node 是函数返回的任何内容。)


 需要注意的一件重要事情是，“pattern”Callable 的参数必须在 Callable 本身中使用，并且“replacement”Callable 的参数必须与模式匹配。第一条规则是为什么在上面的代码块中，“forward”函数有参数“x, w1, w2”，但“pattern”函数只有参数“w1, w2”。 `pattern` 不使用 `x` ，所以它不应该指定 `x` 作为参数。作为第二条规则的示例，考虑替换


```
def pattern(x, y):
    return torch.neg(x) + torch.relu(y)

```




 with
 


```
def replacement(x, y):
    return torch.relu(x)

```


 在这种情况下，“replacement”需要与“pattern”相同数量的参数(“x”和“y”)，即使“replacement”中未使用参数“y”。


 调用 `subgraph_rewriter.replace_pattern` 后，生成的Python代码如下所示：


```
def forward(self, x, w1, w2):
    stack_1 = torch.stack([w1, w2])
    sum_1 = stack_1.sum()
    stack_2 = torch.stack([w1, w2])
    sum_2 = stack_2.sum()
    max_1 = torch.max(sum_1)
    add_1 = x + max_1
    max_2 = torch.max(sum_2)
    add_2 = add_1 + max_2
    return add_2

```


!!! note "笔记"

    保证此 API 的向后兼容性。