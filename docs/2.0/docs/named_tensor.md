# 命名tensor [¶](#named-tensors "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/named_tensor>
>
> 原始地址：<https://pytorch.org/docs/stable/named_tensor.html>


 命名tensor允许用户为tensor维度给出明确的名称。在大多数情况下，采用维度参数的操作将接受维度名称，从而避免需要按位置跟踪维度。此外，命名tensor使用名称来自动检查 API 是否在以下位置正确使用：运行时，提供额外的安全性。名称还可以用于重新排列维度，例如支持“按名称广播”而不是“按位置广播”。


!!! warning "警告"

     指定的tensor API 是一个原型功能，可能会发生变化。


## 创建命名tensor [¶](#creating-named-tensors "此标题的永久链接")


 工厂函数现在采用一个新的“names”参数，该参数将名称与每个维度相关联。


```
>>> torch.zeros(2, 3, names=('N', 'C'))
tensor([[0., 0., 0.],
 [0., 0., 0.]], names=('N', 'C'))

```


 命名维度与常规tensor维度一样，是有序的。 `tensor.names[i]` 是 `tensor` 维度 `i` 的名称。


 以下工厂函数支持命名tensor：



* [`torch.empty()`](generated/torch.empty.html#torch.empty "torch.empty")
* [`torch.rand()`](generated/torch.rand.html#torch.rand "torch.rand")
* [`torch.randn()`](generated/torch.randn.html#torch.randn "torch.randn")
* [`torch.ones()`](generated/torch.ones.html#torch.ones "torch.ones")
* [`torch.tensor()`](generated/torch.tensor.html#torch.tensor "torch.tensor")
* [`torch.zeros()`](generated/torch.zeros.html#torch.zeros“torch.zeros”)


## 命名维度 [¶](#named-dimensions "此标题的永久链接")


 有关tensor名称的限制，请参阅 [`names`](#torch.Tensor.names "torch.Tensor.names")。


 使用 [`names`](#torch.Tensor.names "torch.Tensor.names") 访问tensor的维度名称，并 [`rename()`](#torch.Tensor.rename "torch.Tensor.rename ") 重命名命名尺寸。


```
>>> imgs = torch.randn(1, 2, 2, 3 , names=('N', 'C', 'H', 'W'))
>>> imgs.names
('N', 'C', 'H', 'W')

>>> renamed_imgs = imgs.rename(H='height', W='width')
>>> renamed_imgs.names
('N', 'C', 'height', 'width)

```


 命名tensor可以与未命名tensor共存；命名tensor是 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 的实例。未命名的tensor具有“None”命名的维度。命名tensor不需要命名所有维度。


```
>>> imgs = torch.randn(1, 2, 2, 3 , names=(None, 'C', 'H', 'W'))
>>> imgs.names
(None, 'C', 'H', 'W')

```


## 名称传播语义 [¶](#name-propagation-semantics "永久链接到此标题")


 命名tensor使用名称来自动检查 API 在运行时是否被正确调用。这发生在名为“名称推断”的过程中。更正式地说，名称推断由以下两个步骤组成：



* **检查名称**：运算符可以在运行时执行自动检查，检查某些维度名称是否必须匹配。
* **传播名称**：名称推断将名称传播到输出tensor。


 所有支持命名tensor的操作都会传播名称。


```
>>> x = torch.randn(3, 3, names=('N', 'C'))
>>> x.abs().names
('N', 'C')

```


### 匹配语义 [¶](#match-semantics "此标题的永久链接")


 如果两个名称相等(字符串相等)或者至少有一个名称为“None”，则它们*匹配*。None 本质上是一种特殊的“通配符”名称。


“unify(A, B)”确定将哪个名称“A”和“B”传播到输出。如果两个名称匹配，它将返回两个名称中更具体的一个。如果名称不匹配，则会出错。




!!! note "笔记"

    在实践中，当使用命名tensor时，应该避免使用未命名的维度，因为它们的处理可能很复杂。建议使用 [`refine_names()`](#torch.Tensor.refine_names "torch.Tensor.refine_names") 将所有未命名维度提升为命名维度。


### 基本名称推断规则 [¶](#basic-name-inference-rules "永久链接到此标题")


 让我们看看在没有广播的情况下添加两个一维tensor的情况下如何在名称推断中使用“match”和“unify”。


```
x = torch.randn(3, names=('X',))
y = torch.randn(3)
z = torch.randn(3, names=('Z',))

```


**检查名称**：检查两个tensor的名称*匹配*。


 对于以下示例：


```
>>> # x + y # match('X', None) is True
>>> # x + z # match('X', 'Z') is False
>>> # x + x # match('X', 'X') is True

>>> x + z
Error when attempting to broadcast dims ['X'] and dims ['Z']: dim 'X' and dim 'Z' are at the same position from the right but do not match.

```


**传播名称**：*统一*名称以选择要传播的名称。在 `x 
+ y` 的情况下，`unify('X', None) = 'X'` 因为 `'X'` 是比 `None` 更具体。


```
>>> (x + y).names
('X',)
>>> (x + x).names
('X',)

```


 有关名称推断规则的完整列表，请参阅[命名tensor运算符覆盖率](name_inference.html#name-inference-reference-doc)。以下是两个可能有用的常见操作：



* 二进制算术运算：[统一输入中的名称](name_inference.html#unizes-names-from-inputs-doc)
* 矩阵乘法运算：[Contractsaway dims](name_inference.html#contracts-away-dims-doc)


## 按名称显式对齐 [¶](#explicit-alignment-by-names "永久链接到此标题")


 使用 [`align_as()`](#torch.Tensor.align_as "torch.Tensor.align_as") 或 [`align_to()`](#torch.Tensor.align_to "torch.Tensor.align_to")将tensor尺寸按名称对齐到指定的顺序。这对于执行“按名称广播”很有用。


```
# This function is agnostic to the dimension ordering of `input`,
# as long as it has a `C` dimension somewhere.
def scale_channels(input, scale):
    scale = scale.refine_names('C')
    return input * scale.align_as(input)

>>> num_channels = 3
>>> scale = torch.randn(num_channels, names=('C',))
>>> imgs = torch.rand(3, 3, 3, num_channels, names=('N', 'H', 'W', 'C'))
>>> more_imgs = torch.rand(3, num_channels, 3, 3, names=('N', 'C', 'H', 'W'))
>>> videos = torch.randn(3, num_channels, 3, 3, 3, names=('N', 'C', 'H', 'W', 'D')

>>> scale_channels(imgs, scale)
>>> scale_channels(more_imgs, scale)
>>> scale_channels(videos, scale)

```


## 操作尺寸 [¶](#manipulated-dimensions "此标题的固定链接")
- -


 使用 [`align_to()`](#torch.Tensor.align_to "torch.Tensor.align_to") 排列大量维度，而不按照 [`permute()`](generated/火炬) 的要求提及所有维度.Tensor.permute.html#torch.Tensor.permute "torch.Tensor.permute").


```
>>> tensor = torch.randn(2, 2, 2, 2, 2, 2)
>>> named_tensor = tensor.refine_names('A', 'B', 'C', 'D', 'E', 'F')

# Move the F (dim 5) and E dimension (dim 4) to the front while keeping
# the rest in the same order
>>> tensor.permute(5, 4, 0, 1, 2, 3)
>>> named_tensor.align_to('F', 'E', ...)

```


 使用 [`flatten()`](generated/torch.Tensor.flatten.html#torch.Tensor.flatten "torch.Tensor.flatten") 和 [`unflatten()`](generated/torch.Tensor.unflatten.html #torch.Tensor.unflatten "torch.Tensor.unflatten") 分别用于展平和展开尺寸。这些方法比 [`view()`](generated/torch.Tensor.view.html#torch.Tensor.view "torch.Tensor.view") 和 [`reshape()`](generated/torch. Tensor.reshape.html#torch.Tensor.reshape "torch.Tensor.reshape") ，但对于阅读代码的人来说具有更多语义含义。


```
>>> imgs = torch.randn(32, 3, 128, 128)
>>> named_imgs = imgs.refine_names('N', 'C', 'H', 'W')

>>> flat_imgs = imgs.view(32, -1)
>>> named_flat_imgs = named_imgs.flatten(['C', 'H', 'W'], 'features')
>>> named_flat_imgs.names
('N', 'features')

>>> unflattened_named_imgs = named_flat_imgs.unflatten('features', [('C', 3), ('H', 128), ('W', 128)])
>>> unflattened_named_imgs.names
('N', 'C', 'H', 'W')

```


## Autograd 支持 [¶](#autograd-support "此标题的固定链接")


 Autograd 目前以有限的方式支持命名tensor：autograd 忽略所有tensor上的名称。梯度计算仍然是正确的，但我们失去了名称给我们带来的安全性。


```
>>> x = torch.randn(3, names=('D',))
>>> weight = torch.randn(3, names=('D',), requires_grad=True)
>>> loss = (x - weight).abs()
>>> grad_loss = torch.randn(3)
>>> loss.backward(grad_loss)
>>> weight.grad  # Unnamed for now. Will be named in the future
tensor([-1.8107, -0.6357, 0.0783])

>>> weight.grad.zero_()
>>> grad_loss = grad_loss.refine_names('C')
>>> loss = (x - weight).abs()
# Ideally we'd check that the names of loss and grad_loss match but we don't yet.
>>> loss.backward(grad_loss)
>>> weight.grad
tensor([-1.8107, -0.6357, 0.0783])

```


## 当前支持的操作和子系统 [¶](#currently-supported-operations-and-subsystems "永久链接到此标题")


### 运算符 [¶](#operators "此标题的永久链接")


 有关支持的 torch 和tensor操作的完整列表，请参阅[命名tensor运算符覆盖率](name_inference.html#name-inference-reference-doc)。我们尚不支持链接未涵盖的以下内容：



* 索引，高级索引。


 对于“torch.nn.function”运算符，我们支持以下内容：



* [`torch.nn.function.relu()`](generated/torch.nn.function.relu.html#torch.nn.function.relu "torch.nn.function.relu")
* [`torch.nn.function.softmax()`](generated/torch.nn.function.softmax.html#torch.nn.function.softmax "torch.nn.function.softmax")
* [`torch.nn.function.log_softmax ()`](generated/torch.nn.function.log_softmax.html#torch.nn.function.log_softmax "torch.nn.function.log_softmax")
* [`torch.nn.function.tanh()`](generated/torch.nn.function.tanh.html#torch.nn.function.tanh "torch.nn.function.tanh")
* [`torch.nn.function.sigmoid()`](generated/torch.nn.function.sigmoid.html#torch.nn.function.sigmoid "torch.nn.function.sigmoid")
* [`torch.nn.function.dropout()`](generated/torch.nn.function.dropout.html#torch.nn.功能性.dropout“火炬.nn.功能性.dropout”)


### 子系统 [¶](#subsystems "此标题的永久链接")


 支持 Autograd，请参阅 [Autograd 支持](#named-tensors-autograd-doc)。由于梯度当前未命名，优化器可能可以工作，但未经测试。


 目前不支持 NN 模块。当调用具有命名tensor输入的模块时，这可能会导致以下结果：



* NN 模块参数未命名，因此输出可能被部分命名。
* NN 模块前向传递的代码不支持命名tensor，并且会相应地出错。


 我们也不支持以下子系统，尽管有些子系统可以开箱即用：



* 发行版
* 序列化 ( [`torch.load()`](generated/torch.load.html#torch.load "torch.load") , [`torch.save()`](generated/torch.save.html #torch.save "torch.save") )
* 多处理
* JIT
* 分布式
* ONNX


 如果其中任何一个对您的用例有帮助，请[搜索是否已提交问题](https://github.com/pytorch/pytorch/issues?q=is%3Aopen+is%3Aissue+label%3A% 22module%3A+named+tensor%22) 如果没有，[文件一](https://github.com/pytorch/pytorch/issues/new/choose) 。


## 命名tensor API 参考 [¶](#named-tensor-api-reference "此标题的永久链接")


 在本节中，请找到有关命名tensor特定 API 的文档。有关如何通过其他 PyTorchoperators 传播名称的综合参考，请参阅 [命名tensor运算符覆盖率](name_inference.html#name-inference-reference-doc) 。


*班级*


 火炬。


 tensor


 名称 [¶](#torch.Tensor.names"此定义的永久链接")


 存储该tensor每个维度的名称。


“names[idx]”对应于tensor维度“idx”的名称。如果维度已命名，则名称为字符串；如果维度未命名，则名称为“None”。


 维度名称可以包含字符或下划线。此外，维度名称必须是有效的 Python 变量名称(即不以下划线开头)。


 tensor不能有两个同名的命名维度。


!!! warning "警告"

     指定的tensor API 是实验性的，可能会发生变化。


 改名


 (
 
*\*
 


 名称
* , ***


 rename_map
* ) [[source]](_modules/torch/_tensor.html#Tensor.rename)[¶](#torch.Tensor.rename "此定义的永久链接")


 重命名 `self` 的维度名称。


 主要有两种用途：


`self.rename(**rename_map)` 返回tensor的视图，该tensor已按映射 `rename_map` 中指定的方式进行暗重命名。


`self.rename(*names)` 返回tensor的视图，使用 [`names`](#torch.Tensor.names "torch.Tensor.names") 对所有维度进行位置重命名。使用 `self.rename(None)`在tensor上删除名字。


 不能同时指定位置参数 [`names`](#torch.Tensor.names "torch.Tensor.names") 和关键字参数 `rename_map` 。


 例子：


```
>>> imgs = torch.rand(2, 3, 5, 7, names=('N', 'C', 'H', 'W'))
>>> renamed_imgs = imgs.rename(N='batch', C='channels')
>>> renamed_imgs.names
('batch', 'channels', 'H', 'W')

>>> renamed_imgs = imgs.rename(None)
>>> renamed_imgs.names
(None, None, None, None)

>>> renamed_imgs = imgs.rename('batch', 'channel', 'height', 'width')
>>> renamed_imgs.names
('batch', 'channel', 'height', 'width')

```


!!! warning "警告"

     指定的tensor API 是实验性的，可能会发生变化。


 改名_


 (
 
*\*
 


 名称
* , ***


 rename_map
* ) [[source]](_modules/torch/_tensor.html#Tensor.rename_)[¶](#torch.Tensor.rename_ "此定义的永久链接")


 [`rename()`](#torch.Tensor.rename "torch.Tensor.rename") 的就地版本。


 细化_名称


 (
 
*\*
 


 名称
* ) [[source]](_modules/torch/_tensor.html#Tensor.refine_names)[¶](#torch.Tensor.refine_names "此定义的永久链接")


 根据 [`names`](#torch.Tensor.names "torch.Tensor.names") 细化 `self` 的维度名称。


 精炼是重命名的一种特殊情况，它“提升”未命名的维度。“无”暗淡可以精炼为具有任何名称；命名的 dim 只能被定义为具有相同的名称。


 由于命名tensor可以与未命名tensor共存，因此细化名称提供了一种编写命名tensor感知代码的好方法，该代码可以同时使用命名和未命名tensor。


[`names`](#torch.Tensor.names "torch.Tensor.names") 最多可以包含一个省略号 ( `...` )。省略号会贪婪地扩展；它使用来自“self.names”相应索引的名称就地扩展以填充 [`names`](#torch.Tensor.names "torch.Tensor.names") 到与 `self.dim()` 相同的长度`。


 Python 2 不支持省略号，但可以使用字符串文字(“...”)。


 Parameters


**names** ( *iterable
* *of
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 的输出tensor的所需名称。最多可包含一个省略号。


 例子：


```
>>> imgs = torch.randn(32, 3, 128, 128)
>>> named_imgs = imgs.refine_names('N', 'C', 'H', 'W')
>>> named_imgs.names
('N', 'C', 'H', 'W')

>>> tensor = torch.randn(2, 3, 5, 7, 11)
>>> tensor = tensor.refine_names('A', ..., 'B', 'C')
>>> tensor.names
('A', None, None, 'B', 'C')

```


!!! warning "警告"

     指定的tensor API 是实验性的，可能会发生变化。


 对齐_as


 ( *其他
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.Tensor.align_as"此定义的永久链接")


 排列“self”tensor的维度以匹配“other”tensor中的维度顺序，为任何新名称添加大小为 1 的暗淡。


 此操作对于按名称进行显式广播非常有用(请参阅示例)。


 为了使用此方法，必须命名“self”的所有暗淡。生成的tensor是原始tensor的视图。


 `self` 的所有维度名称必须出现在 `other.names` 中。 “other”可能包含“self.names”中没有的命名维度；输出tensor对于每个新名称都有一个大小为一的维度。


 要将tensor对齐到特定顺序，请使用 [`align_to()`](#torch.Tensor.align_to "torch.Tensor.align_to") 。


 例子：


```
# Example 1: Applying a mask
>>> mask = torch.randint(2, [127, 128], dtype=torch.bool).refine_names('W', 'H')
>>> imgs = torch.randn(32, 128, 127, 3, names=('N', 'H', 'W', 'C'))
>>> imgs.masked_fill_(mask.align_as(imgs), 0)


# Example 2: Applying a per-channel-scale
>>> def scale_channels(input, scale):
>>>    scale = scale.refine_names('C')
>>>    return input * scale.align_as(input)

>>> num_channels = 3
>>> scale = torch.randn(num_channels, names=('C',))
>>> imgs = torch.rand(32, 128, 128, num_channels, names=('N', 'H', 'W', 'C'))
>>> more_imgs = torch.rand(32, num_channels, 128, 128, names=('N', 'C', 'H', 'W'))
>>> videos = torch.randn(3, num_channels, 128, 128, 128, names=('N', 'C', 'H', 'W', 'D'))

# scale_channels is agnostic to the dimension order of the input
>>> scale_channels(imgs, scale)
>>> scale_channels(more_imgs, scale)
>>> scale_channels(videos, scale)

```


!!! warning "警告"

     指定的tensor API 是实验性的，可能会发生变化。


 对齐_到


 (
 
*\*
 


 名称
* ) [[source]](_modules/torch/_tensor.html#Tensor.align_to)[¶](#torch.Tensor.align_to "此定义的永久链接")


 排列 `self` tensor的维度以匹配 [`names`](#torch.Tensor.names "torch.Tensor.names") 中指定的顺序，为任何新名称添加大小为 1 的暗淡。


 为了使用此方法，必须命名“self”的所有暗淡。生成的tensor是原始tensor的视图。


 `self` 的所有维度名称必须出现在 [`names`](#torch.Tensor.names "torch.Tensor.names") 中。 [`names`](#torch.Tensor.names "torch.Tensor.names") 可能包含 `self.names` 中没有的其他名称；输出tensor对于每个新名称都有一个大小为一的维度。


[`names`](#torch.Tensor.names "torch.Tensor.names") 最多可以包含一个省略号 ( `...` )。省略号会扩展为等于 `self` 的所有维度名称在 [`names`](#torch.Tensor.names "torch.Tensor.names") 中没有按照它们出现在 `self` 中的顺序提及。


 Python 2 不支持省略号，但可以使用字符串文字(“...”)。


 Parameters


**names** ( *iterable
* *of
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 的输出tensor的所需维度排序。最多可包含一个省略号，该省略号可扩展到所有未提及的“self”暗淡名称。


 例子：


```
>>> tensor = torch.randn(2, 2, 2, 2, 2, 2)
>>> named_tensor = tensor.refine_names('A', 'B', 'C', 'D', 'E', 'F')

# Move the F and E dims to the front while keeping the rest in order
>>> named_tensor.align_to('F', 'E', ...)

```


!!! warning "警告"

     指定的tensor API 是实验性的，可能会发生变化。


 压扁


 ( *dims
* , *out_dim
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


 将“dims”展平为名为“out_dim”的单一维度。


 所有暗淡在“self”tensor中必须按顺序连续，但在内存中不必连续。


 例子：


```
>>> imgs = torch.randn(32, 3, 128, 128, names=('N', 'C', 'H', 'W'))
>>> flat_imgs = imgs.flatten(['C', 'H', 'W'], 'features')
>>> flat_imgs.names, flat_imgs.shape
(('N', 'features'), torch.Size([32, 49152]))

```


!!! warning "警告"

     指定的tensor API 是实验性的，可能会发生变化。