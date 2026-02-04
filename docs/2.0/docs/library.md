# torch.library [¶](#torch-library "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/library>
>
> 原始地址：<https://pytorch.org/docs/stable/library.html>


 Python 运算符注册 API 提供了使用用户定义的运算符扩展 PyTorch 核心运算符库的功能。目前，这可以通过两种方式完成：


1. 创建新库



* 允许您通过指定适当的调度键来注册各种后端和功能的**新运算符**和内核。例如， 
> 
> 
> 
> 
+ 考虑在新创建的命名空间 
> `foo` 
> 中注册一个新的运算符 
> `add` 
> 。您可以使用 
> `torch.ops` 
> API 访问此运算符，并通过调用 
> `torch.ops.foo.add` 
> 进行调用。您还可以通过调用 
> `torch.ops.foo.add.{overload_name}` 
> 来访问特定的注册重载。 
> 
+ 如果您为此运算符的 
> `CUDA` 
> 调度键注册了新内核，则将为 CUDA tensor输入调用您的自定义函数。 >> 
* 这可以通过创建 `"DEF"` 类型的库类对象来完成。2.扩展现有的 C++ 库(例如 aten)



* 允许您通过指定适当的调度键，为与各种后端和功能相对应的**现有运算符**注册内核。 
* 这可能会派上用场，以填补对通过调度键实现的功能的不稳定的操作员支持。例如，
> 
> 
> 
> 
+ 您可以添加对元tensor的运算符支持(通过将函数注册到 
> `Meta` 
> 调度键)。 >> 
* 这可以通过创建“IMPL”类型的库类对象来完成。


 [Google Colab](https://colab.research.google.com/drive/1RRhSfk7So3Cn02itzLWE9K4Fam-8U011?usp=sharing) 上提供了引导您完成有关如何使用此 API 的一些示例的教程。


!!! warning "警告"

     Dispatcher 是一个复杂的 PyTorch 概念，充分了解 Dispatcher 对于能够使用此 API 执行任何高级操作至关重要。 [这篇博文](http://blog.ezyang.com/2020/09/lets-talk-about-the-pytorch-dispatcher/) 是了解 Dispatcher 的一个很好的起点。


*班级*


 火炬库。


 图书馆


 ( *ns
* , *kind
* , *dispatch_key



 =
 


 ''
* ) [[source]](_modules/torch/library.html#Library)[¶](#torch.library.Library "此定义的永久链接")


 一个用于创建库的类，该库可用于注册新运算符或覆盖 Python 中现有库中的运算符。如果用户只想注册仅与一个特定调度键对应的内核，则可以选择传入调度键名。


 要创建一个库来覆盖现有库(名称为 ns)中的运算符，请将种类设置为“IMPL”。要创建新库(名称为 ns)来注册新运算符，请将种类设置为“DEF”。要创建一个可能存在的库的片段，用于注册运算符(并绕过给定名称空间只有一个库的限制)，将种类设置为“FRAGMENT”。


 参数 
* **ns** – 库名称
* **kind** – “DEF”、“IMPL”(默认值：“IMPL”)、“FRAGMENT”
* **dispatch_key** – PyTorch 调度密钥(默认值：“IMPL”) “”)


 定义


 ( *架构
* , *别名_analysis



 =
 


 ''
* ) [[source]](_modules/torch/library.html#Library.define)[¶](#torch.library.Library.define "此定义的永久链接")


 在 ns 命名空间中定义一个新的运算符及其语义。


 参数 
* **schema** – 定义新运算符的函数架构。
* **alias_analysis** ( *可选
* ) – 指示是否可以从架构推断运算符参数的别名属性(默认行为) (“保守的”)。


 退货


 从架构中推断出的运算符名称。


 例子：：




```
>>> my_lib = Library("foo", "DEF")
>>> my_lib.define("sum(Tensor self) -> Tensor")

```


 impl
 


 ( *op_name
* , *fn
* , *dispatch_key



 =
 


 ''
* ) [[source]](_modules/torch/library.html#Library.impl)[¶](#torch.library.Library.impl "此定义的永久链接")


 注册库中定义的运算符的函数实现。


 参数 
* **op_name** – 运算符名称(以及重载)或 OpOverload 对象。
* **fn** – 输入调度键或 [`fallthrough_kernel()`]( 的运算符实现的函数) #torch.library.fallthrough_kernel "torch.library.fallthrough_kernel") 注册一个fallthrough。
* **dispatch_key** – 应该注册输入函数的调度键。默认情况下，它使用创建库时使用的调度密钥。


 例子：：




```
>>> my_lib = Library("aten", "IMPL")
>>> def div_cpu(self, other):
>>>     return self * (1 / other)
>>> my_lib.impl("div.Tensor", div_cpu, "CPU")

```


 火炬库。


 失败_kernel


 ( ) [[source]](_modules/torch/library.html#fallthrough_kernel)[¶](#torch.library.fallthrough_kernel "此定义的永久链接")


 传递给“Library.impl”以注册失败的虚拟函数。


 我们还添加了一些函数装饰器，以方便为操作符注册函数：



* `torch.library.impl()`
* `torch.library.define()`