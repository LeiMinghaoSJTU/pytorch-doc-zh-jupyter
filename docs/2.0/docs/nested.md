# torch.nested [¶](#module-torch.nested "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/nested>
>
> 原始地址：<https://pytorch.org/docs/stable/nested.html>


## 简介 [¶](#introduction "此标题的永久链接")


!!! warning "警告"

     嵌套tensor的 PyTorch API 处于原型阶段，并将在不久的将来发生变化。


 NestedTensor 允许用户将tensor列表打包到单个高效的数据结构中。


 输入tensor的唯一约束是它们的维度必须匹配。


 这可以实现更高效的元数据表示和对专用内核的访问。


 NestedTensors 的一种应用是表达各个域中的顺序数据。传统方法是填充可变长度序列，而 NestedTensor 使用户能够绕过填充。用于在嵌套tensor上调用操作的 API 与常规 `torch.Tensor` 没有什么不同，它应该允许与现有模型无缝集成，主要区别在于 [输入的构造](#construction) 。


 由于这是一个原型功能，[支持的操作](#supported-operations) 仍然有限。但是，我们欢迎提出问题、功能请求和贡献。有关贡献的更多信息可以在[本自述文件](https://github.com/pytorch/pytorch/blob/main/aten/src/ATen/native/nested/README.md)中找到。


## 构造 [¶](#construction "此标题的永久链接")


 构造很简单，涉及将tensor列表传递给“torch.nested.nested_tensor”构造函数。


```
>>> a, b = torch.arange(3), torch.arange(5) + 3
>>> a
tensor([0, 1, 2])
>>> b
tensor([3, 4, 5, 6, 7])
>>> nt = torch.nested.nested_tensor([a, b])
>>> nt
nested_tensor([
 tensor([0, 1, 2]),
 tensor([3, 4, 5, 6, 7])
 ])

```


 数据类型、设备以及是否需要渐变可以通过常用的关键字参数进行选择。


```
>>> nt = torch.nested.nested_tensor([a, b], dtype=torch.float32, device="cuda", requires_grad=True)
>>> nt
nested_tensor([
 tensor([0., 1., 2.], device='cuda:0', requires_grad=True),
 tensor([3., 4., 5., 6., 7.], device='cuda:0', requires_grad=True)
], device='cuda:0', requires_grad=True)

```


 按照“torch.as_tensor”的脉络，“torch.nested.as_nested_tensor”可用于保留传递给构造函数的tensor的自动梯度历史记录。有关更多信息，请参阅[嵌套tensor构造函数和转换函数](#constructor-functions) 部分。


 为了形成有效的 NestedTensor，所有传递的tensor都需要在维度上匹配，但其他属性都不需要匹配。


```
>>> a = torch.randn(3, 50, 70) # image 1
>>> b = torch.randn(3, 128, 64) # image 2
>>> nt = torch.nested.nested_tensor([a, b], dtype=torch.float32)
>>> nt.dim()
4

```


 如果其中一个维度不匹配，构造函数将抛出错误。


```
>>> a = torch.randn(50, 128) # text 1
>>> b = torch.randn(3, 128, 64) # image 2
>>> nt = torch.nested.nested_tensor([a, b], dtype=torch.float32)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
RuntimeError: All Tensors given to nested_tensor must have the same dimension. Found dimension 3 for Tensor at index 1 and dimension 2 for Tensor at index 0.

```


 请注意，传递的tensor将被复制到连续的内存中。结果NestedTensor分配新的内存来存储它们并且不保留引用。


 目前我们只支持一层嵌套，即简单、扁平的tensor列表。将来我们可以添加对多层嵌套的支持，例如完全由tensor列表组成的列表。请注意，对于此扩展，重要的是保持跨条目的均匀嵌套级别，以便生成的 NestedTensor 具有明确定义的维度。如果您需要此功能，请提出功能请求，以便我们可以跟踪它并做出相应的计划。


## size [¶](#size "此标题的固定链接")


 即使 NestedTensor 不支持 `.size()` (或 `.shape` )，但如果维度 i 是规则的，它也支持 `.size(i)` 。


```
>>> a = torch.randn(50, 128) # text 1
>>> b = torch.randn(32, 128) # text 2
>>> nt = torch.nested.nested_tensor([a, b], dtype=torch.float32)
>>> nt.size(0)
2
>>> nt.size(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
RuntimeError: Given dimension 1 is irregular and does not have a size.
>>> nt.size(2)
128

```


 如果所有维度都是规则的，则 NestedTensor 在语义上与规则的 `torch.Tensor` 没有区别。


```
>>> a = torch.randn(20, 128) # text 1
>>> nt = torch.nested.nested_tensor([a, a], dtype=torch.float32)
>>> nt.size(0)
2
>>> nt.size(1)
20
>>> nt.size(2)
128
>>> torch.stack(nt.unbind()).size()
torch.Size([2, 20, 128])
>>> torch.stack([a, a]).size()
torch.Size([2, 20, 128])
>>> torch.equal(torch.stack(nt.unbind()), torch.stack([a, a]))
True

```


 将来我们可能会更轻松地检测这种情况并进行无缝转换。


 如果您需要此功能(或任何其他相关功能)，请打开功能请求。


## 取消绑定 [¶](#unbind "此标题的永久链接")


“unbind”允许您检索成分的视图。


```
>>> import torch
>>> a = torch.randn(2, 3)
>>> b = torch.randn(3, 4)
>>> nt = torch.nested.nested_tensor([a, b], dtype=torch.float32)
>>> nt
nested_tensor([
 tensor([[ 1.2286, -1.2343, -1.4842],
 [-0.7827, 0.6745, 0.0658]]),
 tensor([[-1.1247, -0.4078, -1.0633, 0.8083],
 [-0.2871, -0.2980, 0.5559, 1.9885],
 [ 0.4074, 2.4855, 0.0733, 0.8285]])
])
>>> nt.unbind()
(tensor([[ 1.2286, -1.2343, -1.4842],
 [-0.7827, 0.6745, 0.0658]]), tensor([[-1.1247, -0.4078, -1.0633, 0.8083],
 [-0.2871, -0.2980, 0.5559, 1.9885],
 [ 0.4074, 2.4855, 0.0733, 0.8285]]))
>>> nt.unbind()[0] is not a
True
>>> nt.unbind()[0].mul_(3)
tensor([[ 3.6858, -3.7030, -4.4525],
 [-2.3481, 2.0236, 0.1975]])
>>> nt
nested_tensor([
 tensor([[ 3.6858, -3.7030, -4.4525],
 [-2.3481, 2.0236, 0.1975]]),
 tensor([[-1.1247, -0.4078, -1.0633, 0.8083],
 [-0.2871, -0.2980, 0.5559, 1.9885],
 [ 0.4074, 2.4855, 0.0733, 0.8285]])
])

```


 请注意，“nt.unbind()[0]”不是副本，而是底层内存的切片，它表示 NestedTensor 的第一个条目或组成部分。


## 嵌套tensor构造函数和转换函数 [¶](#nested-tensor-constructor-and-conversion-functions "永久链接到此标题")


 以下函数与嵌套tensor相关：


 火炬.嵌套。


 嵌套_tensor


 ( *tensor_list
* , *** , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *需要_grad



 =
 


 假
* , *pin_内存



 =
 


 错误的
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.nested.nested_tensor"此定义的永久链接")


 从tensor列表“tensor_list”构造一个没有 autograd 历史记录的嵌套tensor(也称为“叶tensor”，请参阅 [Autograd 力学](notes/autograd.html#autograd-mechanics) )。


 参数 
* **tensor_list** ( *List
* *[
* *array_like
* *]
* ) – tensor列表，或可以传递给 torch.tensor 的任何内容，
* **维度。** ( *其中每个元素
* *的
* *列表具有相同的
* ) –


 关键字参数 
* **dtype** ( [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") , 可选) – 返回的嵌套tensor的所需类型。默认值：如果没有，则相同 [` torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") 作为列表中最左边的tensor。
* **device** ( [`torch.device`](tensor_attributes.html#torch.device "torch.device") , 可选) – 返回的嵌套tensor所需的设备。默认: 如果 None, 与列表中最左边的tensor相同的 [`torch.device`](tensor_attributes.html#torch.device "torch.device") **需要_grad** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – autograd 是否应该记录返回的嵌套tensor的操作。默认值：`False`.
* **pin_memory** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") *,
* *可选
* ) – 如果设置，返回的嵌套tensor将分配在固定内存中。仅适用于 CPU tensor。默认值：`False`。


 例子：


```
>>> a = torch.arange(3, dtype=torch.float, requires_grad=True)
>>> b = torch.arange(5, dtype=torch.float, requires_grad=True)
>>> nt = torch.nested.nested_tensor([a, b], requires_grad=True)
>>> nt.is_leaf
True

```


 火炬.嵌套。


 作为_嵌套_tensor


 ( *tensor_list
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* ) [[source]](_modules/torch/nested.html#as_nested_tensor)[¶](#torch.nested.as_nested_tensor "此定义的永久链接")


 从tensor列表“tensor_list”构造一个嵌套tensor，保留 autograd 历史记录。




!!! note "笔记"

    由于当前的嵌套tensor语义，列表中的tensor始终由该函数复制。


 Parameters


**tensor_list** ( *List
* *[
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*]
* ) – 具有相同 ndim 的tensor列表


 关键字参数 
* **dtype** ( [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") , 可选) – 返回的嵌套tensor的所需类型。默认值：如果没有，则相同 [` torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") 作为列表中最左边的tensor。
* **device** ( [`torch.device`](tensor_attributes.html#torch.device "torch.device") , 可选) – 返回的嵌套tensor所需的设备。默认: 如果 None, 与列表中最左边的tensor相同的 [`torch.device`](tensor_attributes.html#torch.device "torch.device")


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子：


```
>>> a = torch.arange(3, dtype=torch.float, requires_grad=True)
>>> b = torch.arange(5, dtype=torch.float, requires_grad=True)
>>> nt = torch.nested.as_nested_tensor([a, b])
>>> nt.is_leaf
False
>>> fake_grad = torch.nested.nested_tensor([torch.ones_like(a), torch.zeros_like(b)])
>>> nt.backward(fake_grad)
>>> a.grad
tensor([1., 1., 1.])
>>> b.grad
tensor([0., 0., 0., 0., 0.])

```


 火炬.嵌套。


 到_填充_tensor


 (*输入*，*填充*，*输出_大小



 =
 


 无
* , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.nested.to_padded_tensor"此定义的永久链接")


 通过填充“input”嵌套tensor返回一个新的(非嵌套)tensor。前导条目将填充嵌套数据，而尾随条目将被填充。


!!! warning "警告"

    [`to_pated_tensor()`](#torch.nested.to_padded_tensor "torch.nested.to_padded_tensor") 始终复制底层数据，因为嵌套和非嵌套tensor的内存布局不同。


 Parameters


**padding** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 尾随条目的填充值。


 关键字参数 
* **output_size** ( *Tuple
* *[
* [*int*](https://docs.python.org/3/library/functions.html#int "(在 Python v3.12 中) ")*]
* ) – 输出tensor的大小。如果给定，它必须足够大以包含所有嵌套数据；否则，将通过沿每个维度获取每个嵌套子tensor的最大大小来推断。
* ** out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> nt = torch.nested.nested_tensor([torch.randn((2, 5)), torch.randn((3, 4))])
nested_tensor([
 tensor([[ 1.6862, -1.1282, 1.1031, 0.0464, -1.3276],
 [-1.9967, -1.0054, 1.8972, 0.9174, -1.4995]]),
 tensor([[-1.8546, -0.7194, -0.2918, -0.1846],
 [ 0.2773, 0.8793, -0.5183, -0.6447],
 [ 1.8009, 1.8468, -0.9832, -1.5272]])
])
>>> pt_infer = torch.nested.to_padded_tensor(nt, 0.0)
tensor([[[ 1.6862, -1.1282, 1.1031, 0.0464, -1.3276],
 [-1.9967, -1.0054, 1.8972, 0.9174, -1.4995],
 [ 0.0000, 0.0000, 0.0000, 0.0000, 0.0000]],
 [[-1.8546, -0.7194, -0.2918, -0.1846, 0.0000],
 [ 0.2773, 0.8793, -0.5183, -0.6447, 0.0000],
 [ 1.8009, 1.8468, -0.9832, -1.5272, 0.0000]]])
>>> pt_large = torch.nested.to_padded_tensor(nt, 1.0, (2, 4, 6))
tensor([[[ 1.6862, -1.1282, 1.1031, 0.0464, -1.3276, 1.0000],
 [-1.9967, -1.0054, 1.8972, 0.9174, -1.4995, 1.0000],
 [ 1.0000, 1.0000, 1.0000, 1.0000, 1.0000, 1.0000],
 [ 1.0000, 1.0000, 1.0000, 1.0000, 1.0000, 1.0000]],
 [[-1.8546, -0.7194, -0.2918, -0.1846, 1.0000, 1.0000],
 [ 0.2773, 0.8793, -0.5183, -0.6447, 1.0000, 1.0000],
 [ 1.8009, 1.8468, -0.9832, -1.5272, 1.0000, 1.0000],
 [ 1.0000, 1.0000, 1.0000, 1.0000, 1.0000, 1.0000]]])
>>> pt_small = torch.nested.to_padded_tensor(nt, 2.0, (2, 2, 2))
RuntimeError: Value in output_size is less than NestedTensor padded size. Truncation is not supported.

```


## 支持的操作 [¶](#supported-operations "固定链接到此标题")


 在本节中，我们总结了 NestedTensor 当前支持的操作及其具有的任何约束。


| 	 PyTorch operation	  | 	 Constraints	  |
| --- | --- |
| 	[`torch.matmul()`](generated/torch.matmul.html#torch.matmul "torch.matmul")	 | 	 Supports matrix multiplication between two (>= 3d) nested tensors where	the last two dimensions are matrix dimensions and the leading (batch) dimensions have the same size	(i.e. no broadcasting support for batch dimensions yet).	  |
| 	[`torch.bmm()`](generated/torch.bmm.html#torch.bmm "torch.bmm")	 | 	 Supports batch matrix multiplication of two 3-d nested tensors.	  |
| 	[`torch.nn.Linear()`](generated/torch.nn.Linear.html#torch.nn.Linear "torch.nn.Linear")	 | 	 Supports 3-d nested input and a dense 2-d weight matrix.	  |
| 	[`torch.nn.functional.softmax()`](generated/torch.nn.functional.softmax.html#torch.nn.functional.softmax "torch.nn.functional.softmax")	 | 	 Supports softmax along all dims except dim=0.	  |
| 	[`torch.nn.Dropout()`](generated/torch.nn.Dropout.html#torch.nn.Dropout "torch.nn.Dropout")	 | 	 Behavior is the same as on regular tensors.	  |
| 	[`torch.Tensor.masked_fill()`](generated/torch.Tensor.masked_fill.html#torch.Tensor.masked_fill "torch.Tensor.masked_fill")	 | 	 Behavior is the same as on regular tensors.	  |
| 	`torch.relu()`	 | 	 Behavior is the same as on regular tensors.	  |
| 	`torch.gelu()`	 | 	 Behavior is the same as on regular tensors.	  |
| 	`torch.silu()`	 | 	 Behavior is the same as on regular tensors.	  |
| 	[`torch.abs()`](generated/torch.abs.html#torch.abs "torch.abs")	 | 	 Behavior is the same as on regular tensors.	  |
| 	[`torch.sgn()`](generated/torch.sgn.html#torch.sgn "torch.sgn")	 | 	 Behavior is the same as on regular tensors.	  |
| 	[`torch.logical_not()`](generated/torch.logical_not.html#torch.logical_not "torch.logical_not")	 | 	 Behavior is the same as on regular tensors.	  |
| 	[`torch.neg()`](generated/torch.neg.html#torch.neg "torch.neg")	 | 	 Behavior is the same as on regular tensors.	  |
| 	[`torch.sub()`](generated/torch.sub.html#torch.sub "torch.sub")	 | 	 Supports elementwise subtraction of two nested tensors.	  |
| 	[`torch.add()`](generated/torch.add.html#torch.add "torch.add")	 | 	 Supports elementwise addition of two nested tensors. Supports addition of a scalar to a nested tensor.	  |
| 	[`torch.mul()`](generated/torch.mul.html#torch.mul "torch.mul")	 | 	 Supports elementwise multiplication of two nested tensors. Supports multiplication of a nested tensor by a scalar.	  |
| 	[`torch.select()`](generated/torch.select.html#torch.select "torch.select")	 | 	 Supports selecting along all dimensions.	  |
| 	[`torch.clone()`](generated/torch.clone.html#torch.clone "torch.clone")	 | 	 Behavior is the same as on regular tensors.	  |
| 	`torch.detach()`	 | 	 Behavior is the same as on regular tensors.	  |
| 	[`torch.unbind()`](generated/torch.unbind.html#torch.unbind "torch.unbind")	 | 	 Supports unbinding along	 `dim=0`	 only.	  |
| 	[`torch.reshape()`](generated/torch.reshape.html#torch.reshape "torch.reshape")	 | 	 Supports reshaping with size of	 `dim=0`	 preserved (i.e. number of tensors nested cannot be changed).	Unlike regular tensors, a size of	 `-1`	 here means that the existing size is inherited.	In particular, the only valid size for a irregular dimension is	 `-1`	.	Size inference is not implemented yet and hence for new dimensions the size cannot be	 `-1`	.	  |
| 	[`torch.Tensor.reshape_as()`](generated/torch.Tensor.reshape_as.html#torch.Tensor.reshape_as "torch.Tensor.reshape_as")	 | 	 Similar constraint as for	 `reshape`	.	  |
| 	[`torch.transpose()`](generated/torch.transpose.html#torch.transpose "torch.transpose")	 | 	 Supports transposing of all dims except	 `dim=0`	.	  |
| 	[`torch.Tensor.view()`](generated/torch.Tensor.view.html#torch.Tensor.view "torch.Tensor.view")	 | 	 Rules for the new shape are similar to that of	 `reshape`	.	  |
| 	[`torch.empty_like()`](generated/torch.empty_like.html#torch.empty_like "torch.empty_like")	 | 	 Behavior is analogous to that of regular tensors; returns a new empty nested tensor (i.e. with uninitialized values) matching the nested structure of the input.	  |
| 	[`torch.randn_like()`](generated/torch.randn_like.html#torch.randn_like "torch.randn_like")	 | 	 Behavior is analogous to that of regular tensors; returns a new nested tensor with values randomly initialized according to a standard normal distribution matching the nested structure of the input.	  |
| 	[`torch.zeros_like()`](generated/torch.zeros_like.html#torch.zeros_like "torch.zeros_like")	 | 	 Behavior is analogous to that of regular tensors; returns a new nested tensor with all zero values matching the nested structure of the input.	  |
| 	[`torch.nn.LayerNorm()`](generated/torch.nn.LayerNorm.html#torch.nn.LayerNorm "torch.nn.LayerNorm")	 | 	 The	 `normalized_shape`	 argument is restricted to not extend into the irregular dimensions of the NestedTensor.	  |