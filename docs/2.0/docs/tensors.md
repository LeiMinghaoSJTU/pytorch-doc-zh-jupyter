# torch.Tensor [¶](#torch-tensor "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/tensors>
>
> 原始地址：<https://pytorch.org/docs/stable/tensors.html>


 [`torch.Tensor`](#torch.Tensor "torch.Tensor") 是包含单一数据类型元素的多维矩阵。


## 数据类型 [¶](#data-types "此标题的固定链接")


 Torch 定义了 10 种具有 CPU 和 GPU 变体的tensor类型，如下所示：


| 	 Data type	  | 	 dtype	  | 	 CPU tensor	  | 	 GPU tensor	  |
| --- | --- | --- | --- |
| 	 32-bit floating point	  | 	`torch.float32`	 or	 `torch.float`	 | 	`torch.FloatTensor`	 | 	`torch.cuda.FloatTensor`	 |
| 	 64-bit floating point	  | 	`torch.float64`	 or	 `torch.double`	 | 	`torch.DoubleTensor`	 | 	`torch.cuda.DoubleTensor`	 |
| 	 16-bit floating point	 [1](#id4) 	 | 	`torch.float16`	 or	 `torch.half`	 | 	`torch.HalfTensor`	 | 	`torch.cuda.HalfTensor`	 |
| 	 16-bit floating point	 [2](#id5) 	 | 	`torch.bfloat16`	 | 	`torch.BFloat16Tensor`	 | 	`torch.cuda.BFloat16Tensor`	 |
| 	 32-bit complex	  | 	`torch.complex32`	 or	 `torch.chalf`	 |  |  |
| 	 64-bit complex	  | 	`torch.complex64`	 or	 `torch.cfloat`	 |  |  |
| 	 128-bit complex	  | 	`torch.complex128`	 or	 `torch.cdouble`	 |  |  |
| 	 8-bit integer (unsigned)	  | 	`torch.uint8`	 | 	`torch.ByteTensor`	 | 	`torch.cuda.ByteTensor`	 |
| 	 8-bit integer (signed)	  | 	`torch.int8`	 | 	`torch.CharTensor`	 | 	`torch.cuda.CharTensor`	 |
| 	 16-bit integer (signed)	  | 	`torch.int16`	 or	 `torch.short`	 | 	`torch.ShortTensor`	 | 	`torch.cuda.ShortTensor`	 |
| 	 32-bit integer (signed)	  | 	`torch.int32`	 or	 `torch.int`	 | 	`torch.IntTensor`	 | 	`torch.cuda.IntTensor`	 |
| 	 64-bit integer (signed)	  | 	`torch.int64`	 or	 `torch.long`	 | 	`torch.LongTensor`	 | 	`torch.cuda.LongTensor`	 |
| 	 Boolean	  | 	`torch.bool`	 | 	`torch.BoolTensor`	 | 	`torch.cuda.BoolTensor`	 |
| 	 quantized 8-bit integer (unsigned)	  | 	`torch.quint8`	 | 	`torch.ByteTensor`	 | 	 /	  |
| 	 quantized 8-bit integer (signed)	  | 	`torch.qint8`	 | 	`torch.CharTensor`	 | 	 /	  |
| 	 quantized 32-bit integer (signed)	  | 	`torch.qint32`	 | 	`torch.IntTensor`	 | 	 /	  |
| 	 quantized 4-bit integer (unsigned)	 [3](#id6) 	 | 	`torch.quint4x2`	 | 	`torch.ByteTensor`	 | 	 /	  | [1](#id1)


 有时称为 binary16：使用 1 个符号、5 个指数和 10 个有效数字位。当精度很重要而牺牲范围时很有用。


[2](#id2)


 有时称为 Brain 浮点：使用 1 个符号、8 个指数和 7 个有效数字位。当范围很重要时很有用，因为它具有与“float32”相同的指数位数


[3](#id3)


 量化的 4 位整数存储为 8 位有符号整数。目前仅 EmbeddingBag 运算符支持它。


[`torch.Tensor`](#torch.Tensor "torch.Tensor") 是默认tensor类型 ( `torch.FloatTensor` ) 的别名。


## 初始化和基本操作 [¶](#initializing-and-basic-operations "永久链接到此标题")


 tensor可以从 Python [`list`](https://docs.python.org/3/library/stdtypes.html#list "(in Python v3.12)") 或使用 [`torch.tensor()`](generated/torch.tensor.html#torch.tensor "torch.tensor") 构造函数：


```
>>> torch.tensor([[1., -1.], [1., -1.]])
tensor([[ 1.0000, -1.0000],
 [ 1.0000, -1.0000]])
>>> torch.tensor(np.array([[1, 2, 3], [4, 5, 6]]))
tensor([[ 1, 2, 3],
 [ 4, 5, 6]])

```


!!! warning "警告"

    [`torch.tensor()`](generated/torch.tensor.html#torch.tensor "torch.tensor") 总是复制 `data` 。如果您有一个tensor“data”并且只想更改其“requires_grad”标志，请使用 [`requires_grad_()`](generated/torch.Tensor.requires_grad_.html#torch.Tensor.requires_grad_ " torch.Tensor.requires_grad_") 或 [`detach()`](generated/torch.Tensor.detach.html#torch.Tensor.detach "torch.Tensor.detach") 以避免复制。如果你有一个 numpy 数组并且想要避免复制，请使用 [`torch.as_tensor()`](generated/torch.as_tensor.html#torch.as_tensor "torch.as_tensor") 。


 可以通过传递 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") 和/或 [`torch.device`](tensor_attributes.html#torch) 来构造特定数据类型的tensor.device "torch.device") 到构造函数或tensor创建操作：


```
>>> torch.zeros([2, 4], dtype=torch.int32)
tensor([[ 0, 0, 0, 0],
 [ 0, 0, 0, 0]], dtype=torch.int32)
>>> cuda0 = torch.device('cuda:0')
>>> torch.ones([2, 4], dtype=torch.float64, device=cuda0)
tensor([[ 1.0000, 1.0000, 1.0000, 1.0000],
 [ 1.0000, 1.0000, 1.0000, 1.0000]], dtype=torch.float64, device='cuda:0')

```


 有关构建tensor的更多信息，请参阅[Creation Ops](torch.html#tensor-creation-ops)


 可以使用 Python 的索引和切片表示法来访问和修改tensor的内容：


```
>>> x = torch.tensor([[1, 2, 3], [4, 5, 6]])
>>> print(x[1][2])
tensor(6)
>>> x[0][1] = 8
>>> print(x)
tensor([[ 1, 8, 3],
 [ 4, 5, 6]])

```


 使用 [`torch.Tensor.item()`](generated/torch.Tensor.item.html#torch.Tensor.item "torch.Tensor.item") 从包含单个值的tensor中获取 Python 数字：


```
>>> x = torch.tensor([[1]])
>>> x
tensor([[ 1]])
>>> x.item()
1
>>> x = torch.tensor(2.5)
>>> x
tensor(2.5000)
>>> x.item()
2.5

```


 有关索引的更多信息，请参阅[索引、切片、连接、变异操作](torch.html#indexing-slicing-joining)


 可以使用 `requires_grad=True` 创建tensor，以便 [`torch.autograd`](autograd.html#module-torch.autograd "torch.autograd") 记录对其的操作以进行自动微分。


```
>>> x = torch.tensor([[1., -1.], [1., 1.]], requires_grad=True)
>>> out = x.pow(2).sum()
>>> out.backward()
>>> x.grad
tensor([[ 2.0000, -2.0000],
 [ 2.0000, 2.0000]])

```


 每个tensor都有一个关联的 `torch.Storage` ，用于保存其数据。tensor类还提供存储的多维跨步视图并定义数值对其进行操作。




!!! note "笔记"

    有关tensor视图的更多信息，请参阅 [tensor视图](tensor_view.html#tensor-view-doc) 。


!!! note "笔记"

    有关 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") 、 [`torch.device`](tensor_attributes.html#torch.device "torch.device") 和的更多信息[`torch.layout`](tensor_attributes.html#torch.layout "torch.layout") [`torch.Tensor`](#torch.Tensor "torch.Tensor") 的属性，请参阅 [Tensor Attributes](tensor_attributes.html#tensor属性-doc)。


!!! note "笔记"

    改变tensor的方法用下划线后缀标记。例如，“torch.FloatTensor.abs_()”就地计算绝对值并返回修改后的tensor，而“torch.FloatTensor.abs()”则计算产生一个新的tensor。


!!! note "笔记"

    更改现有tensor的 [`torch.device`](tensor_attributes.html#torch.device "torch.device") 和/或 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") ，考虑在tensor上使用 [`to()`](generated/torch.Tensor.to.html#torch.Tensor.to "torch.Tensor.to") 方法。


!!! warning "警告"

     [`torch.Tensor`](#torch.Tensor "torch.Tensor") 的当前实现引入了内存开销，因此可能会导致具有许多微小tensor的应用程序中内存使用量异常高。如果这是您的情况，请考虑使用一个大型结构。


## tensor类参考 [¶](#tensor-class-reference "此标题的永久链接")


*班级*


 火炬。


 tensor [¶](#torch.Tensor "此定义的永久链接")


 创建tensor有几种主要方法，具体取决于您的用例。



* 要使用预先存在的数据创建tensor，请使用 [`torch.tensor()`](generated/torch.tensor.html#torch.tensor "torch.tensor") 。
* 要创建具有特定大小的tensor，请使用`torch.*` tensor创建操作(请参阅 [创建操作](torch.html#tensor-creation-ops) )。
* 要创建与另一个tensor具有相同大小(和相似类型)的tensor，请使用 `torch.\ *_like` tensor创建操作(参见 [创建操作](torch.html#tensor-creation-ops) )。
* 要创建与另一个tensor类型相似但大小不同的tensor，请使用 `tensor.new_* ` 创造行动。


 tensor。


 T [¶](#torch.Tensor.T"此定义的永久链接")


 返回该tensor的尺寸反转的视图。


 如果“n”是“x”中的维度数，则“x.T”相当于“x.permute(n-1, n-2,..., 0)”。


!!! warning "警告"

     不推荐在维度为 2 以外的tensor上使用 [`Tensor.T()`](#torch.Tensor.T "torch.Tensor.T") 来反转其形状，并且在将来的版本中会引发错误。考虑 [`mT`](#torch.Tensor.mT "torch.Tensor.mT") 转置批量矩阵或 x.permute(*torch.arange(x.ndim 
- 1, -1, -1))反转tensor的维度。


 tensor。


 H [¶](#torch.Tensor.H"此定义的永久链接")


 返回共轭和转置矩阵(二维tensor)的视图。


对于复数矩阵，“x.H”相当于“x.transpose(0, 1).conj()”，对于实数矩阵，“x.H”相当于“x.transpose(0, 1)”。


 也可以看看


[`mH`](#torch.Tensor.mH "torch.Tensor.mH") ：也适用于批量矩阵的属性。


 tensor。


 mT [¶](#torch.Tensor.mT "此定义的永久链接")


 返回该tensor的最后两个维度转置后的视图。


`x.mT` 等价于 `x.transpose(-2, -1)` 。


 tensor。


 mH [¶](#torch.Tensor.mH "此定义的永久链接")


 访问此属性相当于调用 [`adjoint()`](generated/torch.adjoint.html#torch.adjoint "torch.adjoint") 。


|  |  |
| --- | --- |
| 	[`Tensor.new_tensor`](generated/torch.Tensor.new_tensor.html#torch.Tensor.new_tensor "torch.Tensor.new_tensor")	 | 	 Returns a new Tensor with	 `data`	 as the tensor data.	  |
| 	[`Tensor.new_full`](generated/torch.Tensor.new_full.html#torch.Tensor.new_full "torch.Tensor.new_full")	 | 	 Returns a Tensor of size	 `size`	 filled with	 `fill_value`	.	  |
| 	[`Tensor.new_empty`](generated/torch.Tensor.new_empty.html#torch.Tensor.new_empty "torch.Tensor.new_empty")	 | 	 Returns a Tensor of size	 `size`	 filled with uninitialized data.	  |
| 	[`Tensor.new_ones`](generated/torch.Tensor.new_ones.html#torch.Tensor.new_ones "torch.Tensor.new_ones")	 | 	 Returns a Tensor of size	 `size`	 filled with	 `1`	.	  |
| 	[`Tensor.new_zeros`](generated/torch.Tensor.new_zeros.html#torch.Tensor.new_zeros "torch.Tensor.new_zeros")	 | 	 Returns a Tensor of size	 `size`	 filled with	 `0`	.	  |
| 	[`Tensor.is_cuda`](generated/torch.Tensor.is_cuda.html#torch.Tensor.is_cuda "torch.Tensor.is_cuda")	 | 	 Is	 `True`	 if the Tensor is stored on the GPU,	 `False`	 otherwise.	  |
| 	[`Tensor.is_quantized`](generated/torch.Tensor.is_quantized.html#torch.Tensor.is_quantized "torch.Tensor.is_quantized")	 | 	 Is	 `True`	 if the Tensor is quantized,	 `False`	 otherwise.	  |
| 	[`Tensor.is_meta`](generated/torch.Tensor.is_meta.html#torch.Tensor.is_meta "torch.Tensor.is_meta")	 | 	 Is	 `True`	 if the Tensor is a meta tensor,	 `False`	 otherwise.	  |
| 	[`Tensor.device`](generated/torch.Tensor.device.html#torch.Tensor.device "torch.Tensor.device")	 | 	 Is the	 [`torch.device`](tensor_attributes.html#torch.device "torch.device")	 where this Tensor is.	  |
| 	[`Tensor.grad`](generated/torch.Tensor.grad.html#torch.Tensor.grad "torch.Tensor.grad")	 | 	 This attribute is	 `None`	 by default and becomes a Tensor the first time a call to	 `backward()`	 computes gradients for	 `self`	.	  |
| 	[`Tensor.ndim`](generated/torch.Tensor.ndim.html#torch.Tensor.ndim "torch.Tensor.ndim")	 | 	 Alias for	 [`dim()`](generated/torch.Tensor.dim.html#torch.Tensor.dim "torch.Tensor.dim")	 |
| 	[`Tensor.real`](generated/torch.Tensor.real.html#torch.Tensor.real "torch.Tensor.real")	 | 	 Returns a new tensor containing real values of the	 `self`	 tensor for a complex-valued input tensor.	  |
| 	[`Tensor.imag`](generated/torch.Tensor.imag.html#torch.Tensor.imag "torch.Tensor.imag")	 | 	 Returns a new tensor containing imaginary values of the	 `self`	 tensor.	  |
| 	[`Tensor.nbytes`](generated/torch.Tensor.nbytes.html#torch.Tensor.nbytes "torch.Tensor.nbytes")	 | 	 Returns the number of bytes consumed by the "view" of elements of the Tensor if the Tensor does not use sparse storage layout.	  |
| 	[`Tensor.itemsize`](generated/torch.Tensor.itemsize.html#torch.Tensor.itemsize "torch.Tensor.itemsize")	 | 	 Alias for	 [`element_size()`](generated/torch.Tensor.element_size.html#torch.Tensor.element_size "torch.Tensor.element_size")	 |
| 	[`Tensor.abs`](generated/torch.Tensor.abs.html#torch.Tensor.abs "torch.Tensor.abs")	 | 	 See	 [`torch.abs()`](generated/torch.abs.html#torch.abs "torch.abs")	 |
| 	[`Tensor.abs_`](generated/torch.Tensor.abs_.html#torch.Tensor.abs_ "torch.Tensor.abs_")	 | 	 In-place version of	 [`abs()`](generated/torch.Tensor.abs.html#torch.Tensor.abs "torch.Tensor.abs")	 |
| 	[`Tensor.absolute`](generated/torch.Tensor.absolute.html#torch.Tensor.absolute "torch.Tensor.absolute")	 | 	 Alias for	 [`abs()`](generated/torch.abs.html#torch.abs "torch.abs")	 |
| 	[`Tensor.absolute_`](generated/torch.Tensor.absolute_.html#torch.Tensor.absolute_ "torch.Tensor.absolute_")	 | 	 In-place version of	 [`absolute()`](generated/torch.Tensor.absolute.html#torch.Tensor.absolute "torch.Tensor.absolute")	 Alias for	 `abs_()`	 |
| 	[`Tensor.acos`](generated/torch.Tensor.acos.html#torch.Tensor.acos "torch.Tensor.acos")	 | 	 See	 [`torch.acos()`](generated/torch.acos.html#torch.acos "torch.acos")	 |
| 	[`Tensor.acos_`](generated/torch.Tensor.acos_.html#torch.Tensor.acos_ "torch.Tensor.acos_")	 | 	 In-place version of	 [`acos()`](generated/torch.Tensor.acos.html#torch.Tensor.acos "torch.Tensor.acos")	 |
| 	[`Tensor.arccos`](generated/torch.Tensor.arccos.html#torch.Tensor.arccos "torch.Tensor.arccos")	 | 	 See	 [`torch.arccos()`](generated/torch.arccos.html#torch.arccos "torch.arccos")	 |
| 	[`Tensor.arccos_`](generated/torch.Tensor.arccos_.html#torch.Tensor.arccos_ "torch.Tensor.arccos_")	 | 	 In-place version of	 [`arccos()`](generated/torch.Tensor.arccos.html#torch.Tensor.arccos "torch.Tensor.arccos")	 |
| 	[`Tensor.add`](generated/torch.Tensor.add.html#torch.Tensor.add "torch.Tensor.add")	 | 	 Add a scalar or tensor to	 `self`	 tensor.	  |
| 	[`Tensor.add_`](generated/torch.Tensor.add_.html#torch.Tensor.add_ "torch.Tensor.add_")	 | 	 In-place version of	 [`add()`](generated/torch.Tensor.add.html#torch.Tensor.add "torch.Tensor.add")	 |
| 	[`Tensor.addbmm`](generated/torch.Tensor.addbmm.html#torch.Tensor.addbmm "torch.Tensor.addbmm")	 | 	 See	 [`torch.addbmm()`](generated/torch.addbmm.html#torch.addbmm "torch.addbmm")	 |
| 	[`Tensor.addbmm_`](generated/torch.Tensor.addbmm_.html#torch.Tensor.addbmm_ "torch.Tensor.addbmm_")	 | 	 In-place version of	 [`addbmm()`](generated/torch.Tensor.addbmm.html#torch.Tensor.addbmm "torch.Tensor.addbmm")	 |
| 	[`Tensor.addcdiv`](generated/torch.Tensor.addcdiv.html#torch.Tensor.addcdiv "torch.Tensor.addcdiv")	 | 	 See	 [`torch.addcdiv()`](generated/torch.addcdiv.html#torch.addcdiv "torch.addcdiv")	 |
| 	[`Tensor.addcdiv_`](generated/torch.Tensor.addcdiv_.html#torch.Tensor.addcdiv_ "torch.Tensor.addcdiv_")	 | 	 In-place version of	 [`addcdiv()`](generated/torch.Tensor.addcdiv.html#torch.Tensor.addcdiv "torch.Tensor.addcdiv")	 |
| 	[`Tensor.addcmul`](generated/torch.Tensor.addcmul.html#torch.Tensor.addcmul "torch.Tensor.addcmul")	 | 	 See	 [`torch.addcmul()`](generated/torch.addcmul.html#torch.addcmul "torch.addcmul")	 |
| 	[`Tensor.addcmul_`](generated/torch.Tensor.addcmul_.html#torch.Tensor.addcmul_ "torch.Tensor.addcmul_")	 | 	 In-place version of	 [`addcmul()`](generated/torch.Tensor.addcmul.html#torch.Tensor.addcmul "torch.Tensor.addcmul")	 |
| 	[`Tensor.addmm`](generated/torch.Tensor.addmm.html#torch.Tensor.addmm "torch.Tensor.addmm")	 | 	 See	 [`torch.addmm()`](generated/torch.addmm.html#torch.addmm "torch.addmm")	 |
| 	[`Tensor.addmm_`](generated/torch.Tensor.addmm_.html#torch.Tensor.addmm_ "torch.Tensor.addmm_")	 | 	 In-place version of	 [`addmm()`](generated/torch.Tensor.addmm.html#torch.Tensor.addmm "torch.Tensor.addmm")	 |
| 	[`Tensor.sspaddmm`](generated/torch.Tensor.sspaddmm.html#torch.Tensor.sspaddmm "torch.Tensor.sspaddmm")	 | 	 See	 [`torch.sspaddmm()`](generated/torch.sspaddmm.html#torch.sspaddmm "torch.sspaddmm")	 |
| 	[`Tensor.addmv`](generated/torch.Tensor.addmv.html#torch.Tensor.addmv "torch.Tensor.addmv")	 | 	 See	 [`torch.addmv()`](generated/torch.addmv.html#torch.addmv "torch.addmv")	 |
| 	[`Tensor.addmv_`](generated/torch.Tensor.addmv_.html#torch.Tensor.addmv_ "torch.Tensor.addmv_")	 | 	 In-place version of	 [`addmv()`](generated/torch.Tensor.addmv.html#torch.Tensor.addmv "torch.Tensor.addmv")	 |
| 	[`Tensor.addr`](generated/torch.Tensor.addr.html#torch.Tensor.addr "torch.Tensor.addr")	 | 	 See	 [`torch.addr()`](generated/torch.addr.html#torch.addr "torch.addr")	 |
| 	[`Tensor.addr_`](generated/torch.Tensor.addr_.html#torch.Tensor.addr_ "torch.Tensor.addr_")	 | 	 In-place version of	 [`addr()`](generated/torch.Tensor.addr.html#torch.Tensor.addr "torch.Tensor.addr")	 |
| 	[`Tensor.adjoint`](generated/torch.Tensor.adjoint.html#torch.Tensor.adjoint "torch.Tensor.adjoint")	 | 	 Alias for	 [`adjoint()`](generated/torch.adjoint.html#torch.adjoint "torch.adjoint")	 |
| 	[`Tensor.allclose`](generated/torch.Tensor.allclose.html#torch.Tensor.allclose "torch.Tensor.allclose")	 | 	 See	 [`torch.allclose()`](generated/torch.allclose.html#torch.allclose "torch.allclose")	 |
| 	[`Tensor.amax`](generated/torch.Tensor.amax.html#torch.Tensor.amax "torch.Tensor.amax")	 | 	 See	 [`torch.amax()`](generated/torch.amax.html#torch.amax "torch.amax")	 |
| 	[`Tensor.amin`](generated/torch.Tensor.amin.html#torch.Tensor.amin "torch.Tensor.amin")	 | 	 See	 [`torch.amin()`](generated/torch.amin.html#torch.amin "torch.amin")	 |
| 	[`Tensor.aminmax`](generated/torch.Tensor.aminmax.html#torch.Tensor.aminmax "torch.Tensor.aminmax")	 | 	 See	 [`torch.aminmax()`](generated/torch.aminmax.html#torch.aminmax "torch.aminmax")	 |
| 	[`Tensor.angle`](generated/torch.Tensor.angle.html#torch.Tensor.angle "torch.Tensor.angle")	 | 	 See	 [`torch.angle()`](generated/torch.angle.html#torch.angle "torch.angle")	 |
| 	[`Tensor.apply_`](generated/torch.Tensor.apply_.html#torch.Tensor.apply_ "torch.Tensor.apply_")	 | 	 Applies the function	 `callable`	 to each element in the tensor, replacing each element with the value returned by	 `callable`	.	  |
| 	[`Tensor.argmax`](generated/torch.Tensor.argmax.html#torch.Tensor.argmax "torch.Tensor.argmax")	 | 	 See	 [`torch.argmax()`](generated/torch.argmax.html#torch.argmax "torch.argmax")	 |
| 	[`Tensor.argmin`](generated/torch.Tensor.argmin.html#torch.Tensor.argmin "torch.Tensor.argmin")	 | 	 See	 [`torch.argmin()`](generated/torch.argmin.html#torch.argmin "torch.argmin")	 |
| 	[`Tensor.argsort`](generated/torch.Tensor.argsort.html#torch.Tensor.argsort "torch.Tensor.argsort")	 | 	 See	 [`torch.argsort()`](generated/torch.argsort.html#torch.argsort "torch.argsort")	 |
| 	[`Tensor.argwhere`](generated/torch.Tensor.argwhere.html#torch.Tensor.argwhere "torch.Tensor.argwhere")	 | 	 See	 [`torch.argwhere()`](generated/torch.argwhere.html#torch.argwhere "torch.argwhere")	 |
| 	[`Tensor.asin`](generated/torch.Tensor.asin.html#torch.Tensor.asin "torch.Tensor.asin")	 | 	 See	 [`torch.asin()`](generated/torch.asin.html#torch.asin "torch.asin")	 |
| 	[`Tensor.asin_`](generated/torch.Tensor.asin_.html#torch.Tensor.asin_ "torch.Tensor.asin_")	 | 	 In-place version of	 [`asin()`](generated/torch.Tensor.asin.html#torch.Tensor.asin "torch.Tensor.asin")	 |
| 	[`Tensor.arcsin`](generated/torch.Tensor.arcsin.html#torch.Tensor.arcsin "torch.Tensor.arcsin")	 | 	 See	 [`torch.arcsin()`](generated/torch.arcsin.html#torch.arcsin "torch.arcsin")	 |
| 	[`Tensor.arcsin_`](generated/torch.Tensor.arcsin_.html#torch.Tensor.arcsin_ "torch.Tensor.arcsin_")	 | 	 In-place version of	 [`arcsin()`](generated/torch.Tensor.arcsin.html#torch.Tensor.arcsin "torch.Tensor.arcsin")	 |
| 	[`Tensor.as_strided`](generated/torch.Tensor.as_strided.html#torch.Tensor.as_strided "torch.Tensor.as_strided")	 | 	 See	 [`torch.as_strided()`](generated/torch.as_strided.html#torch.as_strided "torch.as_strided")	 |
| 	[`Tensor.atan`](generated/torch.Tensor.atan.html#torch.Tensor.atan "torch.Tensor.atan")	 | 	 See	 [`torch.atan()`](generated/torch.atan.html#torch.atan "torch.atan")	 |
| 	[`Tensor.atan_`](generated/torch.Tensor.atan_.html#torch.Tensor.atan_ "torch.Tensor.atan_")	 | 	 In-place version of	 [`atan()`](generated/torch.Tensor.atan.html#torch.Tensor.atan "torch.Tensor.atan")	 |
| 	[`Tensor.arctan`](generated/torch.Tensor.arctan.html#torch.Tensor.arctan "torch.Tensor.arctan")	 | 	 See	 [`torch.arctan()`](generated/torch.arctan.html#torch.arctan "torch.arctan")	 |
| 	[`Tensor.arctan_`](generated/torch.Tensor.arctan_.html#torch.Tensor.arctan_ "torch.Tensor.arctan_")	 | 	 In-place version of	 [`arctan()`](generated/torch.Tensor.arctan.html#torch.Tensor.arctan "torch.Tensor.arctan")	 |
| 	[`Tensor.atan2`](generated/torch.Tensor.atan2.html#torch.Tensor.atan2 "torch.Tensor.atan2")	 | 	 See	 [`torch.atan2()`](generated/torch.atan2.html#torch.atan2 "torch.atan2")	 |
| 	[`Tensor.atan2_`](generated/torch.Tensor.atan2_.html#torch.Tensor.atan2_ "torch.Tensor.atan2_")	 | 	 In-place version of	 [`atan2()`](generated/torch.Tensor.atan2.html#torch.Tensor.atan2 "torch.Tensor.atan2")	 |
| 	[`Tensor.arctan2`](generated/torch.Tensor.arctan2.html#torch.Tensor.arctan2 "torch.Tensor.arctan2")	 | 	 See	 [`torch.arctan2()`](generated/torch.arctan2.html#torch.arctan2 "torch.arctan2")	 |
| 	[`Tensor.arctan2_`](generated/torch.Tensor.arctan2_.html#torch.Tensor.arctan2_ "torch.Tensor.arctan2_")	 | 	 atan2_(other) -> Tensor	  |
| 	[`Tensor.all`](generated/torch.Tensor.all.html#torch.Tensor.all "torch.Tensor.all")	 | 	 See	 [`torch.all()`](generated/torch.all.html#torch.all "torch.all")	 |
| 	[`Tensor.any`](generated/torch.Tensor.any.html#torch.Tensor.any "torch.Tensor.any")	 | 	 See	 [`torch.any()`](generated/torch.any.html#torch.any "torch.any")	 |
| 	[`Tensor.backward`](generated/torch.Tensor.backward.html#torch.Tensor.backward "torch.Tensor.backward")	 | 	 Computes the gradient of current tensor wrt graph leaves.	  |
| 	[`Tensor.baddbmm`](generated/torch.Tensor.baddbmm.html#torch.Tensor.baddbmm "torch.Tensor.baddbmm")	 | 	 See	 [`torch.baddbmm()`](generated/torch.baddbmm.html#torch.baddbmm "torch.baddbmm")	 |
| 	[`Tensor.baddbmm_`](generated/torch.Tensor.baddbmm_.html#torch.Tensor.baddbmm_ "torch.Tensor.baddbmm_")	 | 	 In-place version of	 [`baddbmm()`](generated/torch.Tensor.baddbmm.html#torch.Tensor.baddbmm "torch.Tensor.baddbmm")	 |
| 	[`Tensor.bernoulli`](generated/torch.Tensor.bernoulli.html#torch.Tensor.bernoulli "torch.Tensor.bernoulli")	 | 	 Returns a result tensor where each


 结果[i]


 	exttt{结果[i]}


 结果[i]


 是独立采样的


 伯努利 ( self[i] )


 	ext{伯努利}(	exttt{self[i]})


 伯努利


 (
 


 自我[我]


 )
 


 。 |
| [`Tensor.bernoulli_`](generated/torch.Tensor.bernoulli_.html#torch.Tensor.bernoulli_ "torch.Tensor.bernoulli_") |用独立样本填充“self”的每个位置


 伯努利 (p)


 	ext{伯努利}(	exttt{p})


 伯努利


 (
 


 p
 


 )
 




.
  |
| 
[`Tensor.bfloat16`](generated/torch.Tensor.bfloat16.html#torch.Tensor.bfloat16 "torch.Tensor.bfloat16")
 | 
`self.bfloat16()`
 is equivalent to
 `self.to(torch.bfloat16)`
.
  |
| 
[`Tensor.bincount`](generated/torch.Tensor.bincount.html#torch.Tensor.bincount "torch.Tensor.bincount")
 | 
 See
 [`torch.bincount()`](generated/torch.bincount.html#torch.bincount "torch.bincount")
 |
| 
[`Tensor.bitwise\_not`](generated/torch.Tensor.bitwise_not.html#torch.Tensor.bitwise_not "torch.Tensor.bitwise_not")
 | 
 See
 [`torch.bitwise\_not()`](generated/torch.bitwise_not.html#torch.bitwise_not "torch.bitwise_not")
 |
| 
[`Tensor.bitwise\_not\_`](generated/torch.Tensor.bitwise_not_.html#torch.Tensor.bitwise_not_ "torch.Tensor.bitwise_not_")
 | 
 In-place version of
 [`bitwise\_not()`](generated/torch.Tensor.bitwise_not.html#torch.Tensor.bitwise_not "torch.Tensor.bitwise_not")
 |
| 
[`Tensor.bitwise\_and`](generated/torch.Tensor.bitwise_and.html#torch.Tensor.bitwise_and "torch.Tensor.bitwise_and")
 | 
 See
 [`torch.bitwise\_and()`](generated/torch.bitwise_and.html#torch.bitwise_and "torch.bitwise_and")
 |
| 
[`Tensor.bitwise\_and\_`](generated/torch.Tensor.bitwise_and_.html#torch.Tensor.bitwise_and_ "torch.Tensor.bitwise_and_")
 | 
 In-place version of
 [`bitwise\_and()`](generated/torch.Tensor.bitwise_and.html#torch.Tensor.bitwise_and "torch.Tensor.bitwise_and")
 |
| 
[`Tensor.bitwise\_or`](generated/torch.Tensor.bitwise_or.html#torch.Tensor.bitwise_or "torch.Tensor.bitwise_or")
 | 
 See
 [`torch.bitwise\_or()`](generated/torch.bitwise_or.html#torch.bitwise_or "torch.bitwise_or")
 |
| 
[`Tensor.bitwise\_or\_`](generated/torch.Tensor.bitwise_or_.html#torch.Tensor.bitwise_or_ "torch.Tensor.bitwise_or_")
 | 
 In-place version of
 [`bitwise\_or()`](generated/torch.Tensor.bitwise_or.html#torch.Tensor.bitwise_or "torch.Tensor.bitwise_or")
 |
| 
[`Tensor.bitwise\_xor`](generated/torch.Tensor.bitwise_xor.html#torch.Tensor.bitwise_xor "torch.Tensor.bitwise_xor")
 | 
 See
 [`torch.bitwise\_xor()`](generated/torch.bitwise_xor.html#torch.bitwise_xor "torch.bitwise_xor")
 |
| 
[`Tensor.bitwise\_xor\_`](generated/torch.Tensor.bitwise_xor_.html#torch.Tensor.bitwise_xor_ "torch.Tensor.bitwise_xor_")
 | 
 In-place version of
 [`bitwise\_xor()`](generated/torch.Tensor.bitwise_xor.html#torch.Tensor.bitwise_xor "torch.Tensor.bitwise_xor")
 |
| 
[`Tensor.bitwise\_left\_shift`](generated/torch.Tensor.bitwise_left_shift.html#torch.Tensor.bitwise_left_shift "torch.Tensor.bitwise_left_shift")
 | 
 See
 [`torch.bitwise\_left\_shift()`](generated/torch.bitwise_left_shift.html#torch.bitwise_left_shift "torch.bitwise_left_shift")
 |
| 
[`Tensor.bitwise\_left\_shift\_`](generated/torch.Tensor.bitwise_left_shift_.html#torch.Tensor.bitwise_left_shift_ "torch.Tensor.bitwise_left_shift_")
 | 
 In-place version of
 [`bitwise\_left\_shift()`](generated/torch.Tensor.bitwise_left_shift.html#torch.Tensor.bitwise_left_shift "torch.Tensor.bitwise_left_shift")
 |
| 
[`Tensor.bitwise\_right\_shift`](generated/torch.Tensor.bitwise_right_shift.html#torch.Tensor.bitwise_right_shift "torch.Tensor.bitwise_right_shift")
 | 
 See
 [`torch.bitwise\_right\_shift()`](generated/torch.bitwise_right_shift.html#torch.bitwise_right_shift "torch.bitwise_right_shift")
 |
| 
[`Tensor.bitwise\_right\_shift\_`](generated/torch.Tensor.bitwise_right_shift_.html#torch.Tensor.bitwise_right_shift_ "torch.Tensor.bitwise_right_shift_")
 | 
 In-place version of
 [`bitwise\_right\_shift()`](generated/torch.Tensor.bitwise_right_shift.html#torch.Tensor.bitwise_right_shift "torch.Tensor.bitwise_right_shift")
 |
| 
[`Tensor.bmm`](generated/torch.Tensor.bmm.html#torch.Tensor.bmm "torch.Tensor.bmm")
 | 
 See
 [`torch.bmm()`](generated/torch.bmm.html#torch.bmm "torch.bmm")
 |
| 
[`Tensor.bool`](generated/torch.Tensor.bool.html#torch.Tensor.bool "torch.Tensor.bool")
 | 
`self.bool()`
 is equivalent to
 `self.to(torch.bool)`
.
  |
| 
[`Tensor.byte`](generated/torch.Tensor.byte.html#torch.Tensor.byte "torch.Tensor.byte")
 | 
`self.byte()`
 is equivalent to
 `self.to(torch.uint8)`
.
  |
| 
[`Tensor.broadcast\_to`](generated/torch.Tensor.broadcast_to.html#torch.Tensor.broadcast_to "torch.Tensor.broadcast_to")
 | 
 See
 [`torch.broadcast\_to()`](generated/torch.broadcast_to.html#torch.broadcast_to "torch.broadcast_to")
.
  |
| 
[`Tensor.cauchy\_`](generated/torch.Tensor.cauchy_.html#torch.Tensor.cauchy_ "torch.Tensor.cauchy_")
 | 
 Fills the tensor with numbers drawn from the Cauchy distribution:
  |
| 
[`Tensor.ceil`](generated/torch.Tensor.ceil.html#torch.Tensor.ceil "torch.Tensor.ceil")
 | 
 See
 [`torch.ceil()`](generated/torch.ceil.html#torch.ceil "torch.ceil")
 |
| 
[`Tensor.ceil\_`](generated/torch.Tensor.ceil_.html#torch.Tensor.ceil_ "torch.Tensor.ceil_")
 | 
 In-place version of
 [`ceil()`](generated/torch.Tensor.ceil.html#torch.Tensor.ceil "torch.Tensor.ceil")
 |
| 
[`Tensor.char`](generated/torch.Tensor.char.html#torch.Tensor.char "torch.Tensor.char")
 | 
`self.char()`
 is equivalent to
 `self.to(torch.int8)`
.
  |
| 
[`Tensor.cholesky`](generated/torch.Tensor.cholesky.html#torch.Tensor.cholesky "torch.Tensor.cholesky")
 | 
 See
 [`torch.cholesky()`](generated/torch.cholesky.html#torch.cholesky "torch.cholesky")
 |
| 
[`Tensor.cholesky\_inverse`](generated/torch.Tensor.cholesky_inverse.html#torch.Tensor.cholesky_inverse "torch.Tensor.cholesky_inverse")
 | 
 See
 [`torch.cholesky\_inverse()`](generated/torch.cholesky_inverse.html#torch.cholesky_inverse "torch.cholesky_inverse")
 |
| 
[`Tensor.cholesky\_solve`](generated/torch.Tensor.cholesky_solve.html#torch.Tensor.cholesky_solve "torch.Tensor.cholesky_solve")
 | 
 See
 [`torch.cholesky\_solve()`](generated/torch.cholesky_solve.html#torch.cholesky_solve "torch.cholesky_solve")
 |
| 
[`Tensor.chunk`](generated/torch.Tensor.chunk.html#torch.Tensor.chunk "torch.Tensor.chunk")
 | 
 See
 [`torch.chunk()`](generated/torch.chunk.html#torch.chunk "torch.chunk")
 |
| 
[`Tensor.clamp`](generated/torch.Tensor.clamp.html#torch.Tensor.clamp "torch.Tensor.clamp")
 | 
 See
 [`torch.clamp()`](generated/torch.clamp.html#torch.clamp "torch.clamp")
 |
| 
[`Tensor.clamp\_`](generated/torch.Tensor.clamp_.html#torch.Tensor.clamp_ "torch.Tensor.clamp_")
 | 
 In-place version of
 [`clamp()`](generated/torch.Tensor.clamp.html#torch.Tensor.clamp "torch.Tensor.clamp")
 |
| 
[`Tensor.clip`](generated/torch.Tensor.clip.html#torch.Tensor.clip "torch.Tensor.clip")
 | 
 Alias for
 [`clamp()`](generated/torch.Tensor.clamp.html#torch.Tensor.clamp "torch.Tensor.clamp")
.
  |
| 
[`Tensor.clip\_`](generated/torch.Tensor.clip_.html#torch.Tensor.clip_ "torch.Tensor.clip_")
 | 
 Alias for
 [`clamp\_()`](generated/torch.Tensor.clamp_.html#torch.Tensor.clamp_ "torch.Tensor.clamp_")
.
  |
| 
[`Tensor.clone`](generated/torch.Tensor.clone.html#torch.Tensor.clone "torch.Tensor.clone")
 | 
 See
 [`torch.clone()`](generated/torch.clone.html#torch.clone "torch.clone")
 |
| 
[`Tensor.contiguous`](generated/torch.Tensor.contiguous.html#torch.Tensor.contiguous "torch.Tensor.contiguous")
 | 
 Returns a contiguous in memory tensor containing the same data as
 `self`
 tensor.
  |
| 
[`Tensor.copy\_`](generated/torch.Tensor.copy_.html#torch.Tensor.copy_ "torch.Tensor.copy_")
 | 
 Copies the elements from
 `src`
 into
 `self`
 tensor and returns
 `self`
.
  |
| 
[`Tensor.conj`](generated/torch.Tensor.conj.html#torch.Tensor.conj "torch.Tensor.conj")
 | 
 See
 [`torch.conj()`](generated/torch.conj.html#torch.conj "torch.conj")
 |
| 
[`Tensor.conj\_physical`](generated/torch.Tensor.conj_physical.html#torch.Tensor.conj_physical "torch.Tensor.conj_physical")
 | 
 See
 [`torch.conj\_physical()`](generated/torch.conj_physical.html#torch.conj_physical "torch.conj_physical")
 |
| 
[`Tensor.conj\_physical\_`](generated/torch.Tensor.conj_physical_.html#torch.Tensor.conj_physical_ "torch.Tensor.conj_physical_")
 | 
 In-place version of
 [`conj\_physical()`](generated/torch.Tensor.conj_physical.html#torch.Tensor.conj_physical "torch.Tensor.conj_physical")
 |
| 
[`Tensor.resolve\_conj`](generated/torch.Tensor.resolve_conj.html#torch.Tensor.resolve_conj "torch.Tensor.resolve_conj")
 | 
 See
 [`torch.resolve\_conj()`](generated/torch.resolve_conj.html#torch.resolve_conj "torch.resolve_conj")
 |
| 
[`Tensor.resolve\_neg`](generated/torch.Tensor.resolve_neg.html#torch.Tensor.resolve_neg "torch.Tensor.resolve_neg")
 | 
 See
 [`torch.resolve\_neg()`](generated/torch.resolve_neg.html#torch.resolve_neg "torch.resolve_neg")
 |
| 
[`Tensor.copysign`](generated/torch.Tensor.copysign.html#torch.Tensor.copysign "torch.Tensor.copysign")
 | 
 See
 [`torch.copysign()`](generated/torch.copysign.html#torch.copysign "torch.copysign")
 |
| 
[`Tensor.copysign\_`](generated/torch.Tensor.copysign_.html#torch.Tensor.copysign_ "torch.Tensor.copysign_")
 | 
 In-place version of
 [`copysign()`](generated/torch.Tensor.copysign.html#torch.Tensor.copysign "torch.Tensor.copysign")
 |
| 
[`Tensor.cos`](generated/torch.Tensor.cos.html#torch.Tensor.cos "torch.Tensor.cos")
 | 
 See
 [`torch.cos()`](generated/torch.cos.html#torch.cos "torch.cos")
 |
| 
[`Tensor.cos\_`](generated/torch.Tensor.cos_.html#torch.Tensor.cos_ "torch.Tensor.cos_")
 | 
 In-place version of
 [`cos()`](generated/torch.Tensor.cos.html#torch.Tensor.cos "torch.Tensor.cos")
 |
| 
[`Tensor.cosh`](generated/torch.Tensor.cosh.html#torch.Tensor.cosh "torch.Tensor.cosh")
 | 
 See
 [`torch.cosh()`](generated/torch.cosh.html#torch.cosh "torch.cosh")
 |
| 
[`Tensor.cosh\_`](generated/torch.Tensor.cosh_.html#torch.Tensor.cosh_ "torch.Tensor.cosh_")
 | 
 In-place version of
 [`cosh()`](generated/torch.Tensor.cosh.html#torch.Tensor.cosh "torch.Tensor.cosh")
 |
| 
[`Tensor.corrcoef`](generated/torch.Tensor.corrcoef.html#torch.Tensor.corrcoef "torch.Tensor.corrcoef")
 | 
 See
 [`torch.corrcoef()`](generated/torch.corrcoef.html#torch.corrcoef "torch.corrcoef")
 |
| 
[`Tensor.count\_nonzero`](generated/torch.Tensor.count_nonzero.html#torch.Tensor.count_nonzero "torch.Tensor.count_nonzero")
 | 
 See
 [`torch.count\_nonzero()`](generated/torch.count_nonzero.html#torch.count_nonzero "torch.count_nonzero")
 |
| 
[`Tensor.cov`](generated/torch.Tensor.cov.html#torch.Tensor.cov "torch.Tensor.cov")
 | 
 See
 [`torch.cov()`](generated/torch.cov.html#torch.cov "torch.cov")
 |
| 
[`Tensor.acosh`](generated/torch.Tensor.acosh.html#torch.Tensor.acosh "torch.Tensor.acosh")
 | 
 See
 [`torch.acosh()`](generated/torch.acosh.html#torch.acosh "torch.acosh")
 |
| 
[`Tensor.acosh\_`](generated/torch.Tensor.acosh_.html#torch.Tensor.acosh_ "torch.Tensor.acosh_")
 | 
 In-place version of
 [`acosh()`](generated/torch.Tensor.acosh.html#torch.Tensor.acosh "torch.Tensor.acosh")
 |
| 
[`Tensor.arccosh`](generated/torch.Tensor.arccosh.html#torch.Tensor.arccosh "torch.Tensor.arccosh")
 | 
 acosh() -> Tensor
  |
| 
[`Tensor.arccosh\_`](generated/torch.Tensor.arccosh_.html#torch.Tensor.arccosh_ "torch.Tensor.arccosh_")
 | 
 acosh\_() -> Tensor
  |
| 
[`Tensor.cpu`](generated/torch.Tensor.cpu.html#torch.Tensor.cpu "torch.Tensor.cpu")
 | 
 Returns a copy of this object in CPU memory.
  |
| 
[`Tensor.cross`](generated/torch.Tensor.cross.html#torch.Tensor.cross "torch.Tensor.cross")
 | 
 See
 [`torch.cross()`](generated/torch.cross.html#torch.cross "torch.cross")
 |
| 
[`Tensor.cuda`](generated/torch.Tensor.cuda.html#torch.Tensor.cuda "torch.Tensor.cuda")
 | 
 Returns a copy of this object in CUDA memory.
  |
| 
[`Tensor.logcumsumexp`](generated/torch.Tensor.logcumsumexp.html#torch.Tensor.logcumsumexp "torch.Tensor.logcumsumexp")
 | 
 See
 [`torch.logcumsumexp()`](generated/torch.logcumsumexp.html#torch.logcumsumexp "torch.logcumsumexp")
 |
| 
[`Tensor.cummax`](generated/torch.Tensor.cummax.html#torch.Tensor.cummax "torch.Tensor.cummax")
 | 
 See
 [`torch.cummax()`](generated/torch.cummax.html#torch.cummax "torch.cummax")
 |
| 
[`Tensor.cummin`](generated/torch.Tensor.cummin.html#torch.Tensor.cummin "torch.Tensor.cummin")
 | 
 See
 [`torch.cummin()`](generated/torch.cummin.html#torch.cummin "torch.cummin")
 |
| 
[`Tensor.cumprod`](generated/torch.Tensor.cumprod.html#torch.Tensor.cumprod "torch.Tensor.cumprod")
 | 
 See
 [`torch.cumprod()`](generated/torch.cumprod.html#torch.cumprod "torch.cumprod")
 |
| 
[`Tensor.cumprod\_`](generated/torch.Tensor.cumprod_.html#torch.Tensor.cumprod_ "torch.Tensor.cumprod_")
 | 
 In-place version of
 [`cumprod()`](generated/torch.Tensor.cumprod.html#torch.Tensor.cumprod "torch.Tensor.cumprod")
 |
| 
[`Tensor.cumsum`](generated/torch.Tensor.cumsum.html#torch.Tensor.cumsum "torch.Tensor.cumsum")
 | 
 See
 [`torch.cumsum()`](generated/torch.cumsum.html#torch.cumsum "torch.cumsum")
 |
| 
[`Tensor.cumsum\_`](generated/torch.Tensor.cumsum_.html#torch.Tensor.cumsum_ "torch.Tensor.cumsum_")
 | 
 In-place version of
 [`cumsum()`](generated/torch.Tensor.cumsum.html#torch.Tensor.cumsum "torch.Tensor.cumsum")
 |
| 
[`Tensor.chalf`](generated/torch.Tensor.chalf.html#torch.Tensor.chalf "torch.Tensor.chalf")
 | 
`self.chalf()`
 is equivalent to
 `self.to(torch.complex32)`
.
  |
| 
[`Tensor.cfloat`](generated/torch.Tensor.cfloat.html#torch.Tensor.cfloat "torch.Tensor.cfloat")
 | 
`self.cfloat()`
 is equivalent to
 `self.to(torch.complex64)`
.
  |
| 
[`Tensor.cdouble`](generated/torch.Tensor.cdouble.html#torch.Tensor.cdouble "torch.Tensor.cdouble")
 | 
`self.cdouble()`
 is equivalent to
 `self.to(torch.complex128)`
.
  |
| 
[`Tensor.data\_ptr`](generated/torch.Tensor.data_ptr.html#torch.Tensor.data_ptr "torch.Tensor.data_ptr")
 | 
 Returns the address of the first element of
 `self`
 tensor.
  |
| 
[`Tensor.deg2rad`](generated/torch.Tensor.deg2rad.html#torch.Tensor.deg2rad "torch.Tensor.deg2rad")
 | 
 See
 [`torch.deg2rad()`](generated/torch.deg2rad.html#torch.deg2rad "torch.deg2rad")
 |
| 
[`Tensor.dequantize`](generated/torch.Tensor.dequantize.html#torch.Tensor.dequantize "torch.Tensor.dequantize")
 | 
 Given a quantized Tensor, dequantize it and return the dequantized float Tensor.
  |
| 
[`Tensor.det`](generated/torch.Tensor.det.html#torch.Tensor.det "torch.Tensor.det")
 | 
 See
 [`torch.det()`](generated/torch.det.html#torch.det "torch.det")
 |
| 
[`Tensor.dense\_dim`](generated/torch.Tensor.dense_dim.html#torch.Tensor.dense_dim "torch.Tensor.dense_dim")
 | 
 Return the number of dense dimensions in a
 [sparse tensor](sparse.html#sparse-docs)
`self`
.
  |
| 
[`Tensor.detach`](generated/torch.Tensor.detach.html#torch.Tensor.detach "torch.Tensor.detach")
 | 
 Returns a new Tensor, detached from the current graph.
  |
| 
[`Tensor.detach\_`](generated/torch.Tensor.detach_.html#torch.Tensor.detach_ "torch.Tensor.detach_")
 | 
 Detaches the Tensor from the graph that created it, making it a leaf.
  |
| 
[`Tensor.diag`](generated/torch.Tensor.diag.html#torch.Tensor.diag "torch.Tensor.diag")
 | 
 See
 [`torch.diag()`](generated/torch.diag.html#torch.diag "torch.diag")
 |
| 
[`Tensor.diag\_embed`](generated/torch.Tensor.diag_embed.html#torch.Tensor.diag_embed "torch.Tensor.diag_embed")
 | 
 See
 [`torch.diag\_embed()`](generated/torch.diag_embed.html#torch.diag_embed "torch.diag_embed")
 |
| 
[`Tensor.diagflat`](generated/torch.Tensor.diagflat.html#torch.Tensor.diagflat "torch.Tensor.diagflat")
 | 
 See
 [`torch.diagflat()`](generated/torch.diagflat.html#torch.diagflat "torch.diagflat")
 |
| 
[`Tensor.diagonal`](generated/torch.Tensor.diagonal.html#torch.Tensor.diagonal "torch.Tensor.diagonal")
 | 
 See
 [`torch.diagonal()`](generated/torch.diagonal.html#torch.diagonal "torch.diagonal")
 |
| 
[`Tensor.diagonal\_scatter`](generated/torch.Tensor.diagonal_scatter.html#torch.Tensor.diagonal_scatter "torch.Tensor.diagonal_scatter")
 | 
 See
 [`torch.diagonal\_scatter()`](generated/torch.diagonal_scatter.html#torch.diagonal_scatter "torch.diagonal_scatter")
 |
| 
[`Tensor.fill\_diagonal\_`](generated/torch.Tensor.fill_diagonal_.html#torch.Tensor.fill_diagonal_ "torch.Tensor.fill_diagonal_")
 | 
 Fill the main diagonal of a tensor that has at least 2-dimensions.
  |
| 
[`Tensor.fmax`](generated/torch.Tensor.fmax.html#torch.Tensor.fmax "torch.Tensor.fmax")
 | 
 See
 [`torch.fmax()`](generated/torch.fmax.html#torch.fmax "torch.fmax")
 |
| 
[`Tensor.fmin`](generated/torch.Tensor.fmin.html#torch.Tensor.fmin "torch.Tensor.fmin")
 | 
 See
 [`torch.fmin()`](generated/torch.fmin.html#torch.fmin "torch.fmin")
 |
| 
[`Tensor.diff`](generated/torch.Tensor.diff.html#torch.Tensor.diff "torch.Tensor.diff")
 | 
 See
 [`torch.diff()`](generated/torch.diff.html#torch.diff "torch.diff")
 |
| 
[`Tensor.digamma`](generated/torch.Tensor.digamma.html#torch.Tensor.digamma "torch.Tensor.digamma")
 | 
 See
 [`torch.digamma()`](generated/torch.digamma.html#torch.digamma "torch.digamma")
 |
| 
[`Tensor.digamma\_`](generated/torch.Tensor.digamma_.html#torch.Tensor.digamma_ "torch.Tensor.digamma_")
 | 
 In-place version of
 [`digamma()`](generated/torch.Tensor.digamma.html#torch.Tensor.digamma "torch.Tensor.digamma")
 |
| 
[`Tensor.dim`](generated/torch.Tensor.dim.html#torch.Tensor.dim "torch.Tensor.dim")
 | 
 Returns the number of dimensions of
 `self`
 tensor.
  |
| 
[`Tensor.dim\_order`](generated/torch.Tensor.dim_order.html#torch.Tensor.dim_order "torch.Tensor.dim_order")
 | 
 Returns a tuple of int describing the dim order or physical layout of
 `self`
.
  |
| 
[`Tensor.dist`](generated/torch.Tensor.dist.html#torch.Tensor.dist "torch.Tensor.dist")
 | 
 See
 [`torch.dist()`](generated/torch.dist.html#torch.dist "torch.dist")
 |
| 
[`Tensor.div`](generated/torch.Tensor.div.html#torch.Tensor.div "torch.Tensor.div")
 | 
 See
 [`torch.div()`](generated/torch.div.html#torch.div "torch.div")
 |
| 
[`Tensor.div\_`](generated/torch.Tensor.div_.html#torch.Tensor.div_ "torch.Tensor.div_")
 | 
 In-place version of
 [`div()`](generated/torch.Tensor.div.html#torch.Tensor.div "torch.Tensor.div")
 |
| 
[`Tensor.divide`](generated/torch.Tensor.divide.html#torch.Tensor.divide "torch.Tensor.divide")
 | 
 See
 [`torch.divide()`](generated/torch.divide.html#torch.divide "torch.divide")
 |
| 
[`Tensor.divide\_`](generated/torch.Tensor.divide_.html#torch.Tensor.divide_ "torch.Tensor.divide_")
 | 
 In-place version of
 [`divide()`](generated/torch.Tensor.divide.html#torch.Tensor.divide "torch.Tensor.divide")
 |
| 
[`Tensor.dot`](generated/torch.Tensor.dot.html#torch.Tensor.dot "torch.Tensor.dot")
 | 
 See
 [`torch.dot()`](generated/torch.dot.html#torch.dot "torch.dot")
 |
| 
[`Tensor.double`](generated/torch.Tensor.double.html#torch.Tensor.double "torch.Tensor.double")
 | 
`self.double()`
 is equivalent to
 `self.to(torch.float64)`
.
  |
| 
[`Tensor.dsplit`](generated/torch.Tensor.dsplit.html#torch.Tensor.dsplit "torch.Tensor.dsplit")
 | 
 See
 [`torch.dsplit()`](generated/torch.dsplit.html#torch.dsplit "torch.dsplit")
 |
| 
[`Tensor.element\_size`](generated/torch.Tensor.element_size.html#torch.Tensor.element_size "torch.Tensor.element_size")
 | 
 Returns the size in bytes of an individual element.
  |
| 
[`Tensor.eq`](generated/torch.Tensor.eq.html#torch.Tensor.eq "torch.Tensor.eq")
 | 
 See
 [`torch.eq()`](generated/torch.eq.html#torch.eq "torch.eq")
 |
| 
[`Tensor.eq\_`](generated/torch.Tensor.eq_.html#torch.Tensor.eq_ "torch.Tensor.eq_")
 | 
 In-place version of
 [`eq()`](generated/torch.Tensor.eq.html#torch.Tensor.eq "torch.Tensor.eq")
 |
| 
[`Tensor.equal`](generated/torch.Tensor.equal.html#torch.Tensor.equal "torch.Tensor.equal")
 | 
 See
 [`torch.equal()`](generated/torch.equal.html#torch.equal "torch.equal")
 |
| 
[`Tensor.erf`](generated/torch.Tensor.erf.html#torch.Tensor.erf "torch.Tensor.erf")
 | 
 See
 [`torch.erf()`](generated/torch.erf.html#torch.erf "torch.erf")
 |
| 
[`Tensor.erf\_`](generated/torch.Tensor.erf_.html#torch.Tensor.erf_ "torch.Tensor.erf_")
 | 
 In-place version of
 [`erf()`](generated/torch.Tensor.erf.html#torch.Tensor.erf "torch.Tensor.erf")
 |
| 
[`Tensor.erfc`](generated/torch.Tensor.erfc.html#torch.Tensor.erfc "torch.Tensor.erfc")
 | 
 See
 [`torch.erfc()`](generated/torch.erfc.html#torch.erfc "torch.erfc")
 |
| 
[`Tensor.erfc\_`](generated/torch.Tensor.erfc_.html#torch.Tensor.erfc_ "torch.Tensor.erfc_")
 | 
 In-place version of
 [`erfc()`](generated/torch.Tensor.erfc.html#torch.Tensor.erfc "torch.Tensor.erfc")
 |
| 
[`Tensor.erfinv`](generated/torch.Tensor.erfinv.html#torch.Tensor.erfinv "torch.Tensor.erfinv")
 | 
 See
 [`torch.erfinv()`](generated/torch.erfinv.html#torch.erfinv "torch.erfinv")
 |
| 
[`Tensor.erfinv\_`](generated/torch.Tensor.erfinv_.html#torch.Tensor.erfinv_ "torch.Tensor.erfinv_")
 | 
 In-place version of
 [`erfinv()`](generated/torch.Tensor.erfinv.html#torch.Tensor.erfinv "torch.Tensor.erfinv")
 |
| 
[`Tensor.exp`](generated/torch.Tensor.exp.html#torch.Tensor.exp "torch.Tensor.exp")
 | 
 See
 [`torch.exp()`](generated/torch.exp.html#torch.exp "torch.exp")
 |
| 
[`Tensor.exp\_`](generated/torch.Tensor.exp_.html#torch.Tensor.exp_ "torch.Tensor.exp_")
 | 
 In-place version of
 [`exp()`](generated/torch.Tensor.exp.html#torch.Tensor.exp "torch.Tensor.exp")
 |
| 
[`Tensor.expm1`](generated/torch.Tensor.expm1.html#torch.Tensor.expm1 "torch.Tensor.expm1")
 | 
 See
 [`torch.expm1()`](generated/torch.expm1.html#torch.expm1 "torch.expm1")
 |
| 
[`Tensor.expm1\_`](generated/torch.Tensor.expm1_.html#torch.Tensor.expm1_ "torch.Tensor.expm1_")
 | 
 In-place version of
 [`expm1()`](generated/torch.Tensor.expm1.html#torch.Tensor.expm1 "torch.Tensor.expm1")
 |
| 
[`Tensor.expand`](generated/torch.Tensor.expand.html#torch.Tensor.expand "torch.Tensor.expand")
 | 
 Returns a new view of the
 `self`
 tensor with singleton dimensions expanded to a larger size.
  |
| 
[`Tensor.expand\_as`](generated/torch.Tensor.expand_as.html#torch.Tensor.expand_as "torch.Tensor.expand_as")
 | 
 Expand this tensor to the same size as
 `other`
.
  |
| 
[`Tensor.exponential\_`](generated/torch.Tensor.exponential_.html#torch.Tensor.exponential_ "torch.Tensor.exponential_")
 | 
 Fills
 `self`
 tensor with elements drawn from the exponential distribution:
  |
| 
[`Tensor.fix`](generated/torch.Tensor.fix.html#torch.Tensor.fix "torch.Tensor.fix")
 | 
 See
 [`torch.fix()`](generated/torch.fix.html#torch.fix "torch.fix")
.
  |
| 
[`Tensor.fix\_`](generated/torch.Tensor.fix_.html#torch.Tensor.fix_ "torch.Tensor.fix_")
 | 
 In-place version of
 [`fix()`](generated/torch.Tensor.fix.html#torch.Tensor.fix "torch.Tensor.fix")
 |
| 
[`Tensor.fill\_`](generated/torch.Tensor.fill_.html#torch.Tensor.fill_ "torch.Tensor.fill_")
 | 
 Fills
 `self`
 tensor with the specified value.
  |
| 
[`Tensor.flatten`](generated/torch.Tensor.flatten.html#torch.Tensor.flatten "torch.Tensor.flatten")
 | 
 See
 [`torch.flatten()`](generated/torch.flatten.html#torch.flatten "torch.flatten")
 |
| 
[`Tensor.flip`](generated/torch.Tensor.flip.html#torch.Tensor.flip "torch.Tensor.flip")
 | 
 See
 [`torch.flip()`](generated/torch.flip.html#torch.flip "torch.flip")
 |
| 
[`Tensor.fliplr`](generated/torch.Tensor.fliplr.html#torch.Tensor.fliplr "torch.Tensor.fliplr")
 | 
 See
 [`torch.fliplr()`](generated/torch.fliplr.html#torch.fliplr "torch.fliplr")
 |
| 
[`Tensor.flipud`](generated/torch.Tensor.flipud.html#torch.Tensor.flipud "torch.Tensor.flipud")
 | 
 See
 [`torch.flipud()`](generated/torch.flipud.html#torch.flipud "torch.flipud")
 |
| 
[`Tensor.float`](generated/torch.Tensor.float.html#torch.Tensor.float "torch.Tensor.float")
 | 
`self.float()`
 is equivalent to
 `self.to(torch.float32)`
.
  |
| 
[`Tensor.float\_power`](generated/torch.Tensor.float_power.html#torch.Tensor.float_power "torch.Tensor.float_power")
 | 
 See
 [`torch.float\_power()`](generated/torch.float_power.html#torch.float_power "torch.float_power")
 |
| 
[`Tensor.float\_power\_`](generated/torch.Tensor.float_power_.html#torch.Tensor.float_power_ "torch.Tensor.float_power_")
 | 
 In-place version of
 [`float\_power()`](generated/torch.Tensor.float_power.html#torch.Tensor.float_power "torch.Tensor.float_power")
 |
| 
[`Tensor.floor`](generated/torch.Tensor.floor.html#torch.Tensor.floor "torch.Tensor.floor")
 | 
 See
 [`torch.floor()`](generated/torch.floor.html#torch.floor "torch.floor")
 |
| 
[`Tensor.floor\_`](generated/torch.Tensor.floor_.html#torch.Tensor.floor_ "torch.Tensor.floor_")
 | 
 In-place version of
 [`floor()`](generated/torch.Tensor.floor.html#torch.Tensor.floor "torch.Tensor.floor")
 |
| 
[`Tensor.floor\_divide`](generated/torch.Tensor.floor_divide.html#torch.Tensor.floor_divide "torch.Tensor.floor_divide")
 | 
 See
 [`torch.floor\_divide()`](generated/torch.floor_divide.html#torch.floor_divide "torch.floor_divide")
 |
| 
[`Tensor.floor\_divide\_`](generated/torch.Tensor.floor_divide_.html#torch.Tensor.floor_divide_ "torch.Tensor.floor_divide_")
 | 
 In-place version of
 [`floor\_divide()`](generated/torch.Tensor.floor_divide.html#torch.Tensor.floor_divide "torch.Tensor.floor_divide")
 |
| 
[`Tensor.fmod`](generated/torch.Tensor.fmod.html#torch.Tensor.fmod "torch.Tensor.fmod")
 | 
 See
 [`torch.fmod()`](generated/torch.fmod.html#torch.fmod "torch.fmod")
 |
| 
[`Tensor.fmod\_`](generated/torch.Tensor.fmod_.html#torch.Tensor.fmod_ "torch.Tensor.fmod_")
 | 
 In-place version of
 [`fmod()`](generated/torch.Tensor.fmod.html#torch.Tensor.fmod "torch.Tensor.fmod")
 |
| 
[`Tensor.frac`](generated/torch.Tensor.frac.html#torch.Tensor.frac "torch.Tensor.frac")
 | 
 See
 [`torch.frac()`](generated/torch.frac.html#torch.frac "torch.frac")
 |
| 
[`Tensor.frac\_`](generated/torch.Tensor.frac_.html#torch.Tensor.frac_ "torch.Tensor.frac_")
 | 
 In-place version of
 [`frac()`](generated/torch.Tensor.frac.html#torch.Tensor.frac "torch.Tensor.frac")
 |
| 
[`Tensor.frexp`](generated/torch.Tensor.frexp.html#torch.Tensor.frexp "torch.Tensor.frexp")
 | 
 See
 [`torch.frexp()`](generated/torch.frexp.html#torch.frexp "torch.frexp")
 |
| 
[`Tensor.gather`](generated/torch.Tensor.gather.html#torch.Tensor.gather "torch.Tensor.gather")
 | 
 See
 [`torch.gather()`](generated/torch.gather.html#torch.gather "torch.gather")
 |
| 
[`Tensor.gcd`](generated/torch.Tensor.gcd.html#torch.Tensor.gcd "torch.Tensor.gcd")
 | 
 See
 [`torch.gcd()`](generated/torch.gcd.html#torch.gcd "torch.gcd")
 |
| 
[`Tensor.gcd\_`](generated/torch.Tensor.gcd_.html#torch.Tensor.gcd_ "torch.Tensor.gcd_")
 | 
 In-place version of
 [`gcd()`](generated/torch.Tensor.gcd.html#torch.Tensor.gcd "torch.Tensor.gcd")
 |
| 
[`Tensor.ge`](generated/torch.Tensor.ge.html#torch.Tensor.ge "torch.Tensor.ge")
 | 
 See
 [`torch.ge()`](generated/torch.ge.html#torch.ge "torch.ge")
.
  |
| 
[`Tensor.ge\_`](generated/torch.Tensor.ge_.html#torch.Tensor.ge_ "torch.Tensor.ge_")
 | 
 In-place version of
 [`ge()`](generated/torch.Tensor.ge.html#torch.Tensor.ge "torch.Tensor.ge")
.
  |
| 
[`Tensor.greater\_equal`](generated/torch.Tensor.greater_equal.html#torch.Tensor.greater_equal "torch.Tensor.greater_equal")
 | 
 See
 [`torch.greater\_equal()`](generated/torch.greater_equal.html#torch.greater_equal "torch.greater_equal")
.
  |
| 
[`Tensor.greater\_equal\_`](generated/torch.Tensor.greater_equal_.html#torch.Tensor.greater_equal_ "torch.Tensor.greater_equal_")
 | 
 In-place version of
 [`greater\_equal()`](generated/torch.Tensor.greater_equal.html#torch.Tensor.greater_equal "torch.Tensor.greater_equal")
.
  |
| 
[`Tensor.geometric\_`](generated/torch.Tensor.geometric_.html#torch.Tensor.geometric_ "torch.Tensor.geometric_")
 | 
 Fills
 `self`
 tensor with elements drawn from the geometric distribution:
  |
| 
[`Tensor.geqrf`](generated/torch.Tensor.geqrf.html#torch.Tensor.geqrf "torch.Tensor.geqrf")
 | 
 See
 [`torch.geqrf()`](generated/torch.geqrf.html#torch.geqrf "torch.geqrf")
 |
| 
[`Tensor.ger`](generated/torch.Tensor.ger.html#torch.Tensor.ger "torch.Tensor.ger")
 | 
 See
 [`torch.ger()`](generated/torch.ger.html#torch.ger "torch.ger")
 |
| 
[`Tensor.get\_device`](generated/torch.Tensor.get_device.html#torch.Tensor.get_device "torch.Tensor.get_device")
 | 
 For CUDA tensors, this function returns the device ordinal of the GPU on which the tensor resides.
  |
| 
[`Tensor.gt`](generated/torch.Tensor.gt.html#torch.Tensor.gt "torch.Tensor.gt")
 | 
 See
 [`torch.gt()`](generated/torch.gt.html#torch.gt "torch.gt")
.
  |
| 
[`Tensor.gt\_`](generated/torch.Tensor.gt_.html#torch.Tensor.gt_ "torch.Tensor.gt_")
 | 
 In-place version of
 [`gt()`](generated/torch.Tensor.gt.html#torch.Tensor.gt "torch.Tensor.gt")
.
  |
| 
[`Tensor.greater`](generated/torch.Tensor.greater.html#torch.Tensor.greater "torch.Tensor.greater")
 | 
 See
 [`torch.greater()`](generated/torch.greater.html#torch.greater "torch.greater")
.
  |
| 
[`Tensor.greater\_`](generated/torch.Tensor.greater_.html#torch.Tensor.greater_ "torch.Tensor.greater_")
 | 
 In-place version of
 [`greater()`](generated/torch.Tensor.greater.html#torch.Tensor.greater "torch.Tensor.greater")
.
  |
| 
[`Tensor.half`](generated/torch.Tensor.half.html#torch.Tensor.half "torch.Tensor.half")
 | 
`self.half()`
 is equivalent to
 `self.to(torch.float16)`
.
  |
| 
[`Tensor.hardshrink`](generated/torch.Tensor.hardshrink.html#torch.Tensor.hardshrink "torch.Tensor.hardshrink")
 | 
 See
 [`torch.nn.functional.hardshrink()`](generated/torch.nn.functional.hardshrink.html#torch.nn.functional.hardshrink "torch.nn.functional.hardshrink")
 |
| 
[`Tensor.heaviside`](generated/torch.Tensor.heaviside.html#torch.Tensor.heaviside "torch.Tensor.heaviside")
 | 
 See
 [`torch.heaviside()`](generated/torch.heaviside.html#torch.heaviside "torch.heaviside")
 |
| 
[`Tensor.histc`](generated/torch.Tensor.histc.html#torch.Tensor.histc "torch.Tensor.histc")
 | 
 See
 [`torch.histc()`](generated/torch.histc.html#torch.histc "torch.histc")
 |
| 
[`Tensor.histogram`](generated/torch.Tensor.histogram.html#torch.Tensor.histogram "torch.Tensor.histogram")
 | 
 See
 [`torch.histogram()`](generated/torch.histogram.html#torch.histogram "torch.histogram")
 |
| 
[`Tensor.hsplit`](generated/torch.Tensor.hsplit.html#torch.Tensor.hsplit "torch.Tensor.hsplit")
 | 
 See
 [`torch.hsplit()`](generated/torch.hsplit.html#torch.hsplit "torch.hsplit")
 |
| 
[`Tensor.hypot`](generated/torch.Tensor.hypot.html#torch.Tensor.hypot "torch.Tensor.hypot")
 | 
 See
 [`torch.hypot()`](generated/torch.hypot.html#torch.hypot "torch.hypot")
 |
| 
[`Tensor.hypot\_`](generated/torch.Tensor.hypot_.html#torch.Tensor.hypot_ "torch.Tensor.hypot_")
 | 
 In-place version of
 [`hypot()`](generated/torch.Tensor.hypot.html#torch.Tensor.hypot "torch.Tensor.hypot")
 |
| 
[`Tensor.i0`](generated/torch.Tensor.i0.html#torch.Tensor.i0 "torch.Tensor.i0")
 | 
 See
 [`torch.i0()`](generated/torch.i0.html#torch.i0 "torch.i0")
 |
| 
[`Tensor.i0\_`](generated/torch.Tensor.i0_.html#torch.Tensor.i0_ "torch.Tensor.i0_")
 | 
 In-place version of
 [`i0()`](generated/torch.Tensor.i0.html#torch.Tensor.i0 "torch.Tensor.i0")
 |
| 
[`Tensor.igamma`](generated/torch.Tensor.igamma.html#torch.Tensor.igamma "torch.Tensor.igamma")
 | 
 See
 [`torch.igamma()`](generated/torch.igamma.html#torch.igamma "torch.igamma")
 |
| 
[`Tensor.igamma\_`](generated/torch.Tensor.igamma_.html#torch.Tensor.igamma_ "torch.Tensor.igamma_")
 | 
 In-place version of
 [`igamma()`](generated/torch.Tensor.igamma.html#torch.Tensor.igamma "torch.Tensor.igamma")
 |
| 
[`Tensor.igammac`](generated/torch.Tensor.igammac.html#torch.Tensor.igammac "torch.Tensor.igammac")
 | 
 See
 [`torch.igammac()`](generated/torch.igammac.html#torch.igammac "torch.igammac")
 |
| 
[`Tensor.igammac\_`](generated/torch.Tensor.igammac_.html#torch.Tensor.igammac_ "torch.Tensor.igammac_")
 | 
 In-place version of
 [`igammac()`](generated/torch.Tensor.igammac.html#torch.Tensor.igammac "torch.Tensor.igammac")
 |
| 
[`Tensor.index\_add\_`](generated/torch.Tensor.index_add_.html#torch.Tensor.index_add_ "torch.Tensor.index_add_")
 | 
 Accumulate the elements of
 `alpha`
 times
 `source`
 into the
 `self`
 tensor by adding to the indices in the order given in
 `index`
.
  |
| 
[`Tensor.index\_add`](generated/torch.Tensor.index_add.html#torch.Tensor.index_add "torch.Tensor.index_add")
 | 
 Out-of-place version of
 [`torch.Tensor.index\_add\_()`](generated/torch.Tensor.index_add_.html#torch.Tensor.index_add_ "torch.Tensor.index_add_")
.
  |
| 
[`Tensor.index\_copy\_`](generated/torch.Tensor.index_copy_.html#torch.Tensor.index_copy_ "torch.Tensor.index_copy_")
 | 
 Copies the elements of
 [`tensor`](generated/torch.tensor.html#torch.tensor "torch.tensor")
 into the
 `self`
 tensor by selecting the indices in the order given in
 `index`
.
  |
| 
[`Tensor.index\_copy`](generated/torch.Tensor.index_copy.html#torch.Tensor.index_copy "torch.Tensor.index_copy")
 | 
 Out-of-place version of
 [`torch.Tensor.index\_copy\_()`](generated/torch.Tensor.index_copy_.html#torch.Tensor.index_copy_ "torch.Tensor.index_copy_")
.
  |
| 
[`Tensor.index\_fill\_`](generated/torch.Tensor.index_fill_.html#torch.Tensor.index_fill_ "torch.Tensor.index_fill_")
 | 
 Fills the elements of the
 `self`
 tensor with value
 `value`
 by selecting the indices in the order given in
 `index`
.
  |
| 
[`Tensor.index\_fill`](generated/torch.Tensor.index_fill.html#torch.Tensor.index_fill "torch.Tensor.index_fill")
 | 
 Out-of-place version of
 [`torch.Tensor.index\_fill\_()`](generated/torch.Tensor.index_fill_.html#torch.Tensor.index_fill_ "torch.Tensor.index_fill_")
.
  |
| 
[`Tensor.index\_put\_`](generated/torch.Tensor.index_put_.html#torch.Tensor.index_put_ "torch.Tensor.index_put_")
 | 
 Puts values from the tensor
 `values`
 into the tensor
 `self`
 using the indices specified in
 `indices`
 (which is a tuple of Tensors).
  |
| 
[`Tensor.index\_put`](generated/torch.Tensor.index_put.html#torch.Tensor.index_put "torch.Tensor.index_put")
 | 
 Out-place version of
 [`index\_put\_()`](generated/torch.Tensor.index_put_.html#torch.Tensor.index_put_ "torch.Tensor.index_put_")
.
  |
| 
[`Tensor.index\_reduce\_`](generated/torch.Tensor.index_reduce_.html#torch.Tensor.index_reduce_ "torch.Tensor.index_reduce_")
 | 
 Accumulate the elements of
 `source`
 into the
 `self`
 tensor by accumulating to the indices in the order given in
 `index`
 using the reduction given by the
 `reduce`
 argument.
  |
| 
[`Tensor.index\_reduce`](generated/torch.Tensor.index_reduce.html#torch.Tensor.index_reduce "torch.Tensor.index_reduce")
 | 
 |
| 
[`Tensor.index\_select`](generated/torch.Tensor.index_select.html#torch.Tensor.index_select "torch.Tensor.index_select")
 | 
 See
 [`torch.index\_select()`](generated/torch.index_select.html#torch.index_select "torch.index_select")
 |
| 
[`Tensor.indices`](generated/torch.Tensor.indices.html#torch.Tensor.indices "torch.Tensor.indices")
 | 
 Return the indices tensor of a
 [sparse COO tensor](sparse.html#sparse-coo-docs)
.
  |
| 
[`Tensor.inner`](generated/torch.Tensor.inner.html#torch.Tensor.inner "torch.Tensor.inner")
 | 
 See
 [`torch.inner()`](generated/torch.inner.html#torch.inner "torch.inner")
.
  |
| 
[`Tensor.int`](generated/torch.Tensor.int.html#torch.Tensor.int "torch.Tensor.int")
 | 
`self.int()`
 is equivalent to
 `self.to(torch.int32)`
.
  |
| 
[`Tensor.int\_repr`](generated/torch.Tensor.int_repr.html#torch.Tensor.int_repr "torch.Tensor.int_repr")
 | 
 Given a quantized Tensor,
 `self.int\_repr()`
 returns a CPU Tensor with uint8\_t as data type that stores the underlying uint8\_t values of the given Tensor.
  |
| 
[`Tensor.inverse`](generated/torch.Tensor.inverse.html#torch.Tensor.inverse "torch.Tensor.inverse")
 | 
 See
 [`torch.inverse()`](generated/torch.inverse.html#torch.inverse "torch.inverse")
 |
| 
[`Tensor.isclose`](generated/torch.Tensor.isclose.html#torch.Tensor.isclose "torch.Tensor.isclose")
 | 
 See
 [`torch.isclose()`](generated/torch.isclose.html#torch.isclose "torch.isclose")
 |
| 
[`Tensor.isfinite`](generated/torch.Tensor.isfinite.html#torch.Tensor.isfinite "torch.Tensor.isfinite")
 | 
 See
 [`torch.isfinite()`](generated/torch.isfinite.html#torch.isfinite "torch.isfinite")
 |
| 
[`Tensor.isinf`](generated/torch.Tensor.isinf.html#torch.Tensor.isinf "torch.Tensor.isinf")
 | 
 See
 [`torch.isinf()`](generated/torch.isinf.html#torch.isinf "torch.isinf")
 |
| 
[`Tensor.isposinf`](generated/torch.Tensor.isposinf.html#torch.Tensor.isposinf "torch.Tensor.isposinf")
 | 
 See
 [`torch.isposinf()`](generated/torch.isposinf.html#torch.isposinf "torch.isposinf")
 |
| 
[`Tensor.isneginf`](generated/torch.Tensor.isneginf.html#torch.Tensor.isneginf "torch.Tensor.isneginf")
 | 
 See
 [`torch.isneginf()`](generated/torch.isneginf.html#torch.isneginf "torch.isneginf")
 |
| 
[`Tensor.isnan`](generated/torch.Tensor.isnan.html#torch.Tensor.isnan "torch.Tensor.isnan")
 | 
 See
 [`torch.isnan()`](generated/torch.isnan.html#torch.isnan "torch.isnan")
 |
| 
[`Tensor.is\_contiguous`](generated/torch.Tensor.is_contiguous.html#torch.Tensor.is_contiguous "torch.Tensor.is_contiguous")
 | 
 Returns True if
 `self`
 tensor is contiguous in memory in the order specified by memory format.
  |
| 
[`Tensor.is\_complex`](generated/torch.Tensor.is_complex.html#torch.Tensor.is_complex "torch.Tensor.is_complex")
 | 
 Returns True if the data type of
 `self`
 is a complex data type.
  |
| 
[`Tensor.is\_conj`](generated/torch.Tensor.is_conj.html#torch.Tensor.is_conj "torch.Tensor.is_conj")
 | 
 Returns True if the conjugate bit of
 `self`
 is set to true.
  |
| 
[`Tensor.is\_floating\_point`](generated/torch.Tensor.is_floating_point.html#torch.Tensor.is_floating_point "torch.Tensor.is_floating_point")
 | 
 Returns True if the data type of
 `self`
 is a floating point data type.
  |
| 
[`Tensor.is\_inference`](generated/torch.Tensor.is_inference.html#torch.Tensor.is_inference "torch.Tensor.is_inference")
 | 
 See
 `torch.is\_inference()`
 |
| 
[`Tensor.is\_leaf`](generated/torch.Tensor.is_leaf.html#torch.Tensor.is_leaf "torch.Tensor.is_leaf")
 | 
 All Tensors that have
 `requires\_grad`
 which is
 `False`
 will be leaf Tensors by convention.
  |
| 
[`Tensor.is\_pinned`](generated/torch.Tensor.is_pinned.html#torch.Tensor.is_pinned "torch.Tensor.is_pinned")
 | 
 Returns true if this tensor resides in pinned memory.
  |
| 
[`Tensor.is\_set\_to`](generated/torch.Tensor.is_set_to.html#torch.Tensor.is_set_to "torch.Tensor.is_set_to")
 | 
 Returns True if both tensors are pointing to the exact same memory (same storage, offset, size and stride).
  |
| 
[`Tensor.is\_shared`](generated/torch.Tensor.is_shared.html#torch.Tensor.is_shared "torch.Tensor.is_shared")
 | 
 Checks if tensor is in shared memory.
  |
| 
[`Tensor.is\_signed`](generated/torch.Tensor.is_signed.html#torch.Tensor.is_signed "torch.Tensor.is_signed")
 | 
 Returns True if the data type of
 `self`
 is a signed data type.
  |
| 
[`Tensor.is\_sparse`](generated/torch.Tensor.is_sparse.html#torch.Tensor.is_sparse "torch.Tensor.is_sparse")
 | 
 Is
 `True`
 if the Tensor uses sparse COO storage layout,
 `False`
 otherwise.
  |
| 
[`Tensor.istft`](generated/torch.Tensor.istft.html#torch.Tensor.istft "torch.Tensor.istft")
 | 
 See
 [`torch.istft()`](generated/torch.istft.html#torch.istft "torch.istft")
 |
| 
[`Tensor.isreal`](generated/torch.Tensor.isreal.html#torch.Tensor.isreal "torch.Tensor.isreal")
 | 
 See
 [`torch.isreal()`](generated/torch.isreal.html#torch.isreal "torch.isreal")
 |
| 
[`Tensor.item`](generated/torch.Tensor.item.html#torch.Tensor.item "torch.Tensor.item")
 | 
 Returns the value of this tensor as a standard Python number.
  |
| 
[`Tensor.kthvalue`](generated/torch.Tensor.kthvalue.html#torch.Tensor.kthvalue "torch.Tensor.kthvalue")
 | 
 See
 [`torch.kthvalue()`](generated/torch.kthvalue.html#torch.kthvalue "torch.kthvalue")
 |
| 
[`Tensor.lcm`](generated/torch.Tensor.lcm.html#torch.Tensor.lcm "torch.Tensor.lcm")
 | 
 See
 [`torch.lcm()`](generated/torch.lcm.html#torch.lcm "torch.lcm")
 |
| 
[`Tensor.lcm\_`](generated/torch.Tensor.lcm_.html#torch.Tensor.lcm_ "torch.Tensor.lcm_")
 | 
 In-place version of
 [`lcm()`](generated/torch.Tensor.lcm.html#torch.Tensor.lcm "torch.Tensor.lcm")
 |
| 
[`Tensor.ldexp`](generated/torch.Tensor.ldexp.html#torch.Tensor.ldexp "torch.Tensor.ldexp")
 | 
 See
 [`torch.ldexp()`](generated/torch.ldexp.html#torch.ldexp "torch.ldexp")
 |
| 
[`Tensor.ldexp\_`](generated/torch.Tensor.ldexp_.html#torch.Tensor.ldexp_ "torch.Tensor.ldexp_")
 | 
 In-place version of
 [`ldexp()`](generated/torch.Tensor.ldexp.html#torch.Tensor.ldexp "torch.Tensor.ldexp")
 |
| 
[`Tensor.le`](generated/torch.Tensor.le.html#torch.Tensor.le "torch.Tensor.le")
 | 
 See
 [`torch.le()`](generated/torch.le.html#torch.le "torch.le")
.
  |
| 
[`Tensor.le\_`](generated/torch.Tensor.le_.html#torch.Tensor.le_ "torch.Tensor.le_")
 | 
 In-place version of
 [`le()`](generated/torch.Tensor.le.html#torch.Tensor.le "torch.Tensor.le")
.
  |
| 
[`Tensor.less\_equal`](generated/torch.Tensor.less_equal.html#torch.Tensor.less_equal "torch.Tensor.less_equal")
 | 
 See
 [`torch.less\_equal()`](generated/torch.less_equal.html#torch.less_equal "torch.less_equal")
.
  |
| 
[`Tensor.less\_equal\_`](generated/torch.Tensor.less_equal_.html#torch.Tensor.less_equal_ "torch.Tensor.less_equal_")
 | 
 In-place version of
 [`less\_equal()`](generated/torch.Tensor.less_equal.html#torch.Tensor.less_equal "torch.Tensor.less_equal")
.
  |
| 
[`Tensor.lerp`](generated/torch.Tensor.lerp.html#torch.Tensor.lerp "torch.Tensor.lerp")
 | 
 See
 [`torch.lerp()`](generated/torch.lerp.html#torch.lerp "torch.lerp")
 |
| 
[`Tensor.lerp\_`](generated/torch.Tensor.lerp_.html#torch.Tensor.lerp_ "torch.Tensor.lerp_")
 | 
 In-place version of
 [`lerp()`](generated/torch.Tensor.lerp.html#torch.Tensor.lerp "torch.Tensor.lerp")
 |
| 
[`Tensor.lgamma`](generated/torch.Tensor.lgamma.html#torch.Tensor.lgamma "torch.Tensor.lgamma")
 | 
 See
 [`torch.lgamma()`](generated/torch.lgamma.html#torch.lgamma "torch.lgamma")
 |
| 
[`Tensor.lgamma\_`](generated/torch.Tensor.lgamma_.html#torch.Tensor.lgamma_ "torch.Tensor.lgamma_")
 | 
 In-place version of
 [`lgamma()`](generated/torch.Tensor.lgamma.html#torch.Tensor.lgamma "torch.Tensor.lgamma")
 |
| 
[`Tensor.log`](generated/torch.Tensor.log.html#torch.Tensor.log "torch.Tensor.log")
 | 
 See
 [`torch.log()`](generated/torch.log.html#torch.log "torch.log")
 |
| 
[`Tensor.log\_`](generated/torch.Tensor.log_.html#torch.Tensor.log_ "torch.Tensor.log_")
 | 
 In-place version of
 [`log()`](generated/torch.Tensor.log.html#torch.Tensor.log "torch.Tensor.log")
 |
| 
[`Tensor.logdet`](generated/torch.Tensor.logdet.html#torch.Tensor.logdet "torch.Tensor.logdet")
 | 
 See
 [`torch.logdet()`](generated/torch.logdet.html#torch.logdet "torch.logdet")
 |
| 
[`Tensor.log10`](generated/torch.Tensor.log10.html#torch.Tensor.log10 "torch.Tensor.log10")
 | 
 See
 [`torch.log10()`](generated/torch.log10.html#torch.log10 "torch.log10")
 |
| 
[`Tensor.log10\_`](generated/torch.Tensor.log10_.html#torch.Tensor.log10_ "torch.Tensor.log10_")
 | 
 In-place version of
 [`log10()`](generated/torch.Tensor.log10.html#torch.Tensor.log10 "torch.Tensor.log10")
 |
| 
[`Tensor.log1p`](generated/torch.Tensor.log1p.html#torch.Tensor.log1p "torch.Tensor.log1p")
 | 
 See
 [`torch.log1p()`](generated/torch.log1p.html#torch.log1p "torch.log1p")
 |
| 
[`Tensor.log1p\_`](generated/torch.Tensor.log1p_.html#torch.Tensor.log1p_ "torch.Tensor.log1p_")
 | 
 In-place version of
 [`log1p()`](generated/torch.Tensor.log1p.html#torch.Tensor.log1p "torch.Tensor.log1p")
 |
| 
[`Tensor.log2`](generated/torch.Tensor.log2.html#torch.Tensor.log2 "torch.Tensor.log2")
 | 
 See
 [`torch.log2()`](generated/torch.log2.html#torch.log2 "torch.log2")
 |
| 
[`Tensor.log2\_`](generated/torch.Tensor.log2_.html#torch.Tensor.log2_ "torch.Tensor.log2_")
 | 
 In-place version of
 [`log2()`](generated/torch.Tensor.log2.html#torch.Tensor.log2 "torch.Tensor.log2")
 |
| 
[`Tensor.log\_normal\_`](generated/torch.Tensor.log_normal_.html#torch.Tensor.log_normal_ "torch.Tensor.log_normal_")
 | 
 Fills
 `self`
 tensor with numbers samples from the log-normal distribution parameterized by the given mean
 



 μ
 


 \mu
 


 μ
 


 和标准差



 σ
 


 西格玛


 σ
 




.
  |
| 
[`Tensor.logaddexp`](generated/torch.Tensor.logaddexp.html#torch.Tensor.logaddexp "torch.Tensor.logaddexp")
 | 
 See
 [`torch.logaddexp()`](generated/torch.logaddexp.html#torch.logaddexp "torch.logaddexp")
 |
| 
[`Tensor.logaddexp2`](generated/torch.Tensor.logaddexp2.html#torch.Tensor.logaddexp2 "torch.Tensor.logaddexp2")
 | 
 See
 [`torch.logaddexp2()`](generated/torch.logaddexp2.html#torch.logaddexp2 "torch.logaddexp2")
 |
| 
[`Tensor.logsumexp`](generated/torch.Tensor.logsumexp.html#torch.Tensor.logsumexp "torch.Tensor.logsumexp")
 | 
 See
 [`torch.logsumexp()`](generated/torch.logsumexp.html#torch.logsumexp "torch.logsumexp")
 |
| 
[`Tensor.logical\_and`](generated/torch.Tensor.logical_and.html#torch.Tensor.logical_and "torch.Tensor.logical_and")
 | 
 See
 [`torch.logical\_and()`](generated/torch.logical_and.html#torch.logical_and "torch.logical_and")
 |
| 
[`Tensor.logical\_and\_`](generated/torch.Tensor.logical_and_.html#torch.Tensor.logical_and_ "torch.Tensor.logical_and_")
 | 
 In-place version of
 [`logical\_and()`](generated/torch.Tensor.logical_and.html#torch.Tensor.logical_and "torch.Tensor.logical_and")
 |
| 
[`Tensor.logical\_not`](generated/torch.Tensor.logical_not.html#torch.Tensor.logical_not "torch.Tensor.logical_not")
 | 
 See
 [`torch.logical\_not()`](generated/torch.logical_not.html#torch.logical_not "torch.logical_not")
 |
| 
[`Tensor.logical\_not\_`](generated/torch.Tensor.logical_not_.html#torch.Tensor.logical_not_ "torch.Tensor.logical_not_")
 | 
 In-place version of
 [`logical\_not()`](generated/torch.Tensor.logical_not.html#torch.Tensor.logical_not "torch.Tensor.logical_not")
 |
| 
[`Tensor.logical\_or`](generated/torch.Tensor.logical_or.html#torch.Tensor.logical_or "torch.Tensor.logical_or")
 | 
 See
 [`torch.logical\_or()`](generated/torch.logical_or.html#torch.logical_or "torch.logical_or")
 |
| 
[`Tensor.logical\_or\_`](generated/torch.Tensor.logical_or_.html#torch.Tensor.logical_or_ "torch.Tensor.logical_or_")
 | 
 In-place version of
 [`logical\_or()`](generated/torch.Tensor.logical_or.html#torch.Tensor.logical_or "torch.Tensor.logical_or")
 |
| 
[`Tensor.logical\_xor`](generated/torch.Tensor.logical_xor.html#torch.Tensor.logical_xor "torch.Tensor.logical_xor")
 | 
 See
 [`torch.logical\_xor()`](generated/torch.logical_xor.html#torch.logical_xor "torch.logical_xor")
 |
| 
[`Tensor.logical\_xor\_`](generated/torch.Tensor.logical_xor_.html#torch.Tensor.logical_xor_ "torch.Tensor.logical_xor_")
 | 
 In-place version of
 [`logical\_xor()`](generated/torch.Tensor.logical_xor.html#torch.Tensor.logical_xor "torch.Tensor.logical_xor")
 |
| 
[`Tensor.logit`](generated/torch.Tensor.logit.html#torch.Tensor.logit "torch.Tensor.logit")
 | 
 See
 [`torch.logit()`](generated/torch.logit.html#torch.logit "torch.logit")
 |
| 
[`Tensor.logit\_`](generated/torch.Tensor.logit_.html#torch.Tensor.logit_ "torch.Tensor.logit_")
 | 
 In-place version of
 [`logit()`](generated/torch.Tensor.logit.html#torch.Tensor.logit "torch.Tensor.logit")
 |
| 
[`Tensor.long`](generated/torch.Tensor.long.html#torch.Tensor.long "torch.Tensor.long")
 | 
`self.long()`
 is equivalent to
 `self.to(torch.int64)`
.
  |
| 
[`Tensor.lt`](generated/torch.Tensor.lt.html#torch.Tensor.lt "torch.Tensor.lt")
 | 
 See
 [`torch.lt()`](generated/torch.lt.html#torch.lt "torch.lt")
.
  |
| 
[`Tensor.lt\_`](generated/torch.Tensor.lt_.html#torch.Tensor.lt_ "torch.Tensor.lt_")
 | 
 In-place version of
 [`lt()`](generated/torch.Tensor.lt.html#torch.Tensor.lt "torch.Tensor.lt")
.
  |
| 
[`Tensor.less`](generated/torch.Tensor.less.html#torch.Tensor.less "torch.Tensor.less")
 | 
 lt(other) -> Tensor
  |
| 
[`Tensor.less\_`](generated/torch.Tensor.less_.html#torch.Tensor.less_ "torch.Tensor.less_")
 | 
 In-place version of
 [`less()`](generated/torch.Tensor.less.html#torch.Tensor.less "torch.Tensor.less")
.
  |
| 
[`Tensor.lu`](generated/torch.Tensor.lu.html#torch.Tensor.lu "torch.Tensor.lu")
 | 
 See
 [`torch.lu()`](generated/torch.lu.html#torch.lu "torch.lu")
 |
| 
[`Tensor.lu\_solve`](generated/torch.Tensor.lu_solve.html#torch.Tensor.lu_solve "torch.Tensor.lu_solve")
 | 
 See
 [`torch.lu\_solve()`](generated/torch.lu_solve.html#torch.lu_solve "torch.lu_solve")
 |
| 
[`Tensor.as\_subclass`](generated/torch.Tensor.as_subclass.html#torch.Tensor.as_subclass "torch.Tensor.as_subclass")
 | 
 Makes a
 `cls`
 instance with the same data pointer as
 `self`
.
  |
| 
[`Tensor.map\_`](generated/torch.Tensor.map_.html#torch.Tensor.map_ "torch.Tensor.map_")
 | 
 Applies
 `callable`
 for each element in
 `self`
 tensor and the given
 [`tensor`](generated/torch.tensor.html#torch.tensor "torch.tensor")
 and stores the results in
 `self`
 tensor.
  |
| 
[`Tensor.masked\_scatter\_`](generated/torch.Tensor.masked_scatter_.html#torch.Tensor.masked_scatter_ "torch.Tensor.masked_scatter_")
 | 
 Copies elements from
 `source`
 into
 `self`
 tensor at positions where the
 `mask`
 is True.
  |
| 
[`Tensor.masked\_scatter`](generated/torch.Tensor.masked_scatter.html#torch.Tensor.masked_scatter "torch.Tensor.masked_scatter")
 | 
 Out-of-place version of
 [`torch.Tensor.masked\_scatter\_()`](generated/torch.Tensor.masked_scatter_.html#torch.Tensor.masked_scatter_ "torch.Tensor.masked_scatter_")
 |
| 
[`Tensor.masked\_fill\_`](generated/torch.Tensor.masked_fill_.html#torch.Tensor.masked_fill_ "torch.Tensor.masked_fill_")
 | 
 Fills elements of
 `self`
 tensor with
 `value`
 where
 `mask`
 is True.
  |
| 
[`Tensor.masked\_fill`](generated/torch.Tensor.masked_fill.html#torch.Tensor.masked_fill "torch.Tensor.masked_fill")
 | 
 Out-of-place version of
 [`torch.Tensor.masked\_fill\_()`](generated/torch.Tensor.masked_fill_.html#torch.Tensor.masked_fill_ "torch.Tensor.masked_fill_")
 |
| 
[`Tensor.masked\_select`](generated/torch.Tensor.masked_select.html#torch.Tensor.masked_select "torch.Tensor.masked_select")
 | 
 See
 [`torch.masked\_select()`](generated/torch.masked_select.html#torch.masked_select "torch.masked_select")
 |
| 
[`Tensor.matmul`](generated/torch.Tensor.matmul.html#torch.Tensor.matmul "torch.Tensor.matmul")
 | 
 See
 [`torch.matmul()`](generated/torch.matmul.html#torch.matmul "torch.matmul")
 |
| 
[`Tensor.matrix\_power`](generated/torch.Tensor.matrix_power.html#torch.Tensor.matrix_power "torch.Tensor.matrix_power")
 | 



!!! note "笔记"

    [`matrix\_power()`](generated/torch.Tensor.matrix_power.html#torch.Tensor.matrix_power "torch.Tensor.matrix_power")
 is deprecated, use
 [`torch.linalg.matrix\_power()`](generated/torch.linalg.matrix_power.html#torch.linalg.matrix_power "torch.linalg.matrix_power")
 instead.
 
 |
| 
[`Tensor.matrix\_exp`](generated/torch.Tensor.matrix_exp.html#torch.Tensor.matrix_exp "torch.Tensor.matrix_exp")
 | 
 See
 [`torch.matrix\_exp()`](generated/torch.matrix_exp.html#torch.matrix_exp "torch.matrix_exp")
 |
| 
[`Tensor.max`](generated/torch.Tensor.max.html#torch.Tensor.max "torch.Tensor.max")
 | 
 See
 [`torch.max()`](generated/torch.max.html#torch.max "torch.max")
 |
| 
[`Tensor.maximum`](generated/torch.Tensor.maximum.html#torch.Tensor.maximum "torch.Tensor.maximum")
 | 
 See
 [`torch.maximum()`](generated/torch.maximum.html#torch.maximum "torch.maximum")
 |
| 
[`Tensor.mean`](generated/torch.Tensor.mean.html#torch.Tensor.mean "torch.Tensor.mean")
 | 
 See
 [`torch.mean()`](generated/torch.mean.html#torch.mean "torch.mean")
 |
| 
[`Tensor.nanmean`](generated/torch.Tensor.nanmean.html#torch.Tensor.nanmean "torch.Tensor.nanmean")
 | 
 See
 [`torch.nanmean()`](generated/torch.nanmean.html#torch.nanmean "torch.nanmean")
 |
| 
[`Tensor.median`](generated/torch.Tensor.median.html#torch.Tensor.median "torch.Tensor.median")
 | 
 See
 [`torch.median()`](generated/torch.median.html#torch.median "torch.median")
 |
| 
[`Tensor.nanmedian`](generated/torch.Tensor.nanmedian.html#torch.Tensor.nanmedian "torch.Tensor.nanmedian")
 | 
 See
 [`torch.nanmedian()`](generated/torch.nanmedian.html#torch.nanmedian "torch.nanmedian")
 |
| 
[`Tensor.min`](generated/torch.Tensor.min.html#torch.Tensor.min "torch.Tensor.min")
 | 
 See
 [`torch.min()`](generated/torch.min.html#torch.min "torch.min")
 |
| 
[`Tensor.minimum`](generated/torch.Tensor.minimum.html#torch.Tensor.minimum "torch.Tensor.minimum")
 | 
 See
 [`torch.minimum()`](generated/torch.minimum.html#torch.minimum "torch.minimum")
 |
| 
[`Tensor.mm`](generated/torch.Tensor.mm.html#torch.Tensor.mm "torch.Tensor.mm")
 | 
 See
 [`torch.mm()`](generated/torch.mm.html#torch.mm "torch.mm")
 |
| 
[`Tensor.smm`](generated/torch.Tensor.smm.html#torch.Tensor.smm "torch.Tensor.smm")
 | 
 See
 [`torch.smm()`](generated/torch.smm.html#torch.smm "torch.smm")
 |
| 
[`Tensor.mode`](generated/torch.Tensor.mode.html#torch.Tensor.mode "torch.Tensor.mode")
 | 
 See
 [`torch.mode()`](generated/torch.mode.html#torch.mode "torch.mode")
 |
| 
[`Tensor.movedim`](generated/torch.Tensor.movedim.html#torch.Tensor.movedim "torch.Tensor.movedim")
 | 
 See
 [`torch.movedim()`](generated/torch.movedim.html#torch.movedim "torch.movedim")
 |
| 
[`Tensor.moveaxis`](generated/torch.Tensor.moveaxis.html#torch.Tensor.moveaxis "torch.Tensor.moveaxis")
 | 
 See
 [`torch.moveaxis()`](generated/torch.moveaxis.html#torch.moveaxis "torch.moveaxis")
 |
| 
[`Tensor.msort`](generated/torch.Tensor.msort.html#torch.Tensor.msort "torch.Tensor.msort")
 | 
 See
 [`torch.msort()`](generated/torch.msort.html#torch.msort "torch.msort")
 |
| 
[`Tensor.mul`](generated/torch.Tensor.mul.html#torch.Tensor.mul "torch.Tensor.mul")
 | 
 See
 [`torch.mul()`](generated/torch.mul.html#torch.mul "torch.mul")
.
  |
| 
[`Tensor.mul\_`](generated/torch.Tensor.mul_.html#torch.Tensor.mul_ "torch.Tensor.mul_")
 | 
 In-place version of
 [`mul()`](generated/torch.Tensor.mul.html#torch.Tensor.mul "torch.Tensor.mul")
.
  |
| 
[`Tensor.multiply`](generated/torch.Tensor.multiply.html#torch.Tensor.multiply "torch.Tensor.multiply")
 | 
 See
 [`torch.multiply()`](generated/torch.multiply.html#torch.multiply "torch.multiply")
.
  |
| 
[`Tensor.multiply\_`](generated/torch.Tensor.multiply_.html#torch.Tensor.multiply_ "torch.Tensor.multiply_")
 | 
 In-place version of
 [`multiply()`](generated/torch.Tensor.multiply.html#torch.Tensor.multiply "torch.Tensor.multiply")
.
  |
| 
[`Tensor.multinomial`](generated/torch.Tensor.multinomial.html#torch.Tensor.multinomial "torch.Tensor.multinomial")
 | 
 See
 [`torch.multinomial()`](generated/torch.multinomial.html#torch.multinomial "torch.multinomial")
 |
| 
[`Tensor.mv`](generated/torch.Tensor.mv.html#torch.Tensor.mv "torch.Tensor.mv")
 | 
 See
 [`torch.mv()`](generated/torch.mv.html#torch.mv "torch.mv")
 |
| 
[`Tensor.mvlgamma`](generated/torch.Tensor.mvlgamma.html#torch.Tensor.mvlgamma "torch.Tensor.mvlgamma")
 | 
 See
 [`torch.mvlgamma()`](generated/torch.mvlgamma.html#torch.mvlgamma "torch.mvlgamma")
 |
| 
[`Tensor.mvlgamma\_`](generated/torch.Tensor.mvlgamma_.html#torch.Tensor.mvlgamma_ "torch.Tensor.mvlgamma_")
 | 
 In-place version of
 [`mvlgamma()`](generated/torch.Tensor.mvlgamma.html#torch.Tensor.mvlgamma "torch.Tensor.mvlgamma")
 |
| 
[`Tensor.nansum`](generated/torch.Tensor.nansum.html#torch.Tensor.nansum "torch.Tensor.nansum")
 | 
 See
 [`torch.nansum()`](generated/torch.nansum.html#torch.nansum "torch.nansum")
 |
| 
[`Tensor.narrow`](generated/torch.Tensor.narrow.html#torch.Tensor.narrow "torch.Tensor.narrow")
 | 
 See
 [`torch.narrow()`](generated/torch.narrow.html#torch.narrow "torch.narrow")
.
  |
| 
[`Tensor.narrow\_copy`](generated/torch.Tensor.narrow_copy.html#torch.Tensor.narrow_copy "torch.Tensor.narrow_copy")
 | 
 See
 [`torch.narrow\_copy()`](generated/torch.narrow_copy.html#torch.narrow_copy "torch.narrow_copy")
.
  |
| 
[`Tensor.ndimension`](generated/torch.Tensor.ndimension.html#torch.Tensor.ndimension "torch.Tensor.ndimension")
 | 
 Alias for
 [`dim()`](generated/torch.Tensor.dim.html#torch.Tensor.dim "torch.Tensor.dim")
 |
| 
[`Tensor.nan\_to\_num`](generated/torch.Tensor.nan_to_num.html#torch.Tensor.nan_to_num "torch.Tensor.nan_to_num")
 | 
 See
 [`torch.nan\_to\_num()`](generated/torch.nan_to_num.html#torch.nan_to_num "torch.nan_to_num")
.
  |
| 
[`Tensor.nan\_to\_num\_`](generated/torch.Tensor.nan_to_num_.html#torch.Tensor.nan_to_num_ "torch.Tensor.nan_to_num_")
 | 
 In-place version of
 [`nan\_to\_num()`](generated/torch.Tensor.nan_to_num.html#torch.Tensor.nan_to_num "torch.Tensor.nan_to_num")
.
  |
| 
[`Tensor.ne`](generated/torch.Tensor.ne.html#torch.Tensor.ne "torch.Tensor.ne")
 | 
 See
 [`torch.ne()`](generated/torch.ne.html#torch.ne "torch.ne")
.
  |
| 
[`Tensor.ne\_`](generated/torch.Tensor.ne_.html#torch.Tensor.ne_ "torch.Tensor.ne_")
 | 
 In-place version of
 [`ne()`](generated/torch.Tensor.ne.html#torch.Tensor.ne "torch.Tensor.ne")
.
  |
| 
[`Tensor.not\_equal`](generated/torch.Tensor.not_equal.html#torch.Tensor.not_equal "torch.Tensor.not_equal")
 | 
 See
 [`torch.not\_equal()`](generated/torch.not_equal.html#torch.not_equal "torch.not_equal")
.
  |
| 
[`Tensor.not\_equal\_`](generated/torch.Tensor.not_equal_.html#torch.Tensor.not_equal_ "torch.Tensor.not_equal_")
 | 
 In-place version of
 [`not\_equal()`](generated/torch.Tensor.not_equal.html#torch.Tensor.not_equal "torch.Tensor.not_equal")
.
  |
| 
[`Tensor.neg`](generated/torch.Tensor.neg.html#torch.Tensor.neg "torch.Tensor.neg")
 | 
 See
 [`torch.neg()`](generated/torch.neg.html#torch.neg "torch.neg")
 |
| 
[`Tensor.neg\_`](generated/torch.Tensor.neg_.html#torch.Tensor.neg_ "torch.Tensor.neg_")
 | 
 In-place version of
 [`neg()`](generated/torch.Tensor.neg.html#torch.Tensor.neg "torch.Tensor.neg")
 |
| 
[`Tensor.negative`](generated/torch.Tensor.negative.html#torch.Tensor.negative "torch.Tensor.negative")
 | 
 See
 [`torch.negative()`](generated/torch.negative.html#torch.negative "torch.negative")
 |
| 
[`Tensor.negative\_`](generated/torch.Tensor.negative_.html#torch.Tensor.negative_ "torch.Tensor.negative_")
 | 
 In-place version of
 [`negative()`](generated/torch.Tensor.negative.html#torch.Tensor.negative "torch.Tensor.negative")
 |
| 
[`Tensor.nelement`](generated/torch.Tensor.nelement.html#torch.Tensor.nelement "torch.Tensor.nelement")
 | 
 Alias for
 [`numel()`](generated/torch.Tensor.numel.html#torch.Tensor.numel "torch.Tensor.numel")
 |
| 
[`Tensor.nextafter`](generated/torch.Tensor.nextafter.html#torch.Tensor.nextafter "torch.Tensor.nextafter")
 | 
 See
 [`torch.nextafter()`](generated/torch.nextafter.html#torch.nextafter "torch.nextafter")
 |
| 
[`Tensor.nextafter\_`](generated/torch.Tensor.nextafter_.html#torch.Tensor.nextafter_ "torch.Tensor.nextafter_")
 | 
 In-place version of
 [`nextafter()`](generated/torch.Tensor.nextafter.html#torch.Tensor.nextafter "torch.Tensor.nextafter")
 |
| 
[`Tensor.nonzero`](generated/torch.Tensor.nonzero.html#torch.Tensor.nonzero "torch.Tensor.nonzero")
 | 
 See
 [`torch.nonzero()`](generated/torch.nonzero.html#torch.nonzero "torch.nonzero")
 |
| 
[`Tensor.norm`](generated/torch.Tensor.norm.html#torch.Tensor.norm "torch.Tensor.norm")
 | 
 See
 [`torch.norm()`](generated/torch.norm.html#torch.norm "torch.norm")
 |
| 
[`Tensor.normal\_`](generated/torch.Tensor.normal_.html#torch.Tensor.normal_ "torch.Tensor.normal_")
 | 
 Fills
 `self`
 tensor with elements samples from the normal distribution parameterized by
 [`mean`](generated/torch.mean.html#torch.mean "torch.mean")
 and
 [`std`](generated/torch.std.html#torch.std "torch.std")
.
  |
| 
[`Tensor.numel`](generated/torch.Tensor.numel.html#torch.Tensor.numel "torch.Tensor.numel")
 | 
 See
 [`torch.numel()`](generated/torch.numel.html#torch.numel "torch.numel")
 |
| 
[`Tensor.numpy`](generated/torch.Tensor.numpy.html#torch.Tensor.numpy "torch.Tensor.numpy")
 | 
 Returns the tensor as a NumPy
 `ndarray`
.
  |
| 
[`Tensor.orgqr`](generated/torch.Tensor.orgqr.html#torch.Tensor.orgqr "torch.Tensor.orgqr")
 | 
 See
 [`torch.orgqr()`](generated/torch.orgqr.html#torch.orgqr "torch.orgqr")
 |
| 
[`Tensor.ormqr`](generated/torch.Tensor.ormqr.html#torch.Tensor.ormqr "torch.Tensor.ormqr")
 | 
 See
 [`torch.ormqr()`](generated/torch.ormqr.html#torch.ormqr "torch.ormqr")
 |
| 
[`Tensor.outer`](generated/torch.Tensor.outer.html#torch.Tensor.outer "torch.Tensor.outer")
 | 
 See
 [`torch.outer()`](generated/torch.outer.html#torch.outer "torch.outer")
.
  |
| 
[`Tensor.permute`](generated/torch.Tensor.permute.html#torch.Tensor.permute "torch.Tensor.permute")
 | 
 See
 [`torch.permute()`](generated/torch.permute.html#torch.permute "torch.permute")
 |
| 
[`Tensor.pin\_memory`](generated/torch.Tensor.pin_memory.html#torch.Tensor.pin_memory "torch.Tensor.pin_memory")
 | 
 Copies the tensor to pinned memory, if it's not already pinned.
  |
| 
[`Tensor.pinverse`](generated/torch.Tensor.pinverse.html#torch.Tensor.pinverse "torch.Tensor.pinverse")
 | 
 See
 [`torch.pinverse()`](generated/torch.pinverse.html#torch.pinverse "torch.pinverse")
 |
| 
[`Tensor.polygamma`](generated/torch.Tensor.polygamma.html#torch.Tensor.polygamma "torch.Tensor.polygamma")
 | 
 See
 [`torch.polygamma()`](generated/torch.polygamma.html#torch.polygamma "torch.polygamma")
 |
| 
[`Tensor.polygamma\_`](generated/torch.Tensor.polygamma_.html#torch.Tensor.polygamma_ "torch.Tensor.polygamma_")
 | 
 In-place version of
 [`polygamma()`](generated/torch.Tensor.polygamma.html#torch.Tensor.polygamma "torch.Tensor.polygamma")
 |
| 
[`Tensor.positive`](generated/torch.Tensor.positive.html#torch.Tensor.positive "torch.Tensor.positive")
 | 
 See
 [`torch.positive()`](generated/torch.positive.html#torch.positive "torch.positive")
 |
| 
[`Tensor.pow`](generated/torch.Tensor.pow.html#torch.Tensor.pow "torch.Tensor.pow")
 | 
 See
 [`torch.pow()`](generated/torch.pow.html#torch.pow "torch.pow")
 |
| 
[`Tensor.pow\_`](generated/torch.Tensor.pow_.html#torch.Tensor.pow_ "torch.Tensor.pow_")
 | 
 In-place version of
 [`pow()`](generated/torch.Tensor.pow.html#torch.Tensor.pow "torch.Tensor.pow")
 |
| 
[`Tensor.prod`](generated/torch.Tensor.prod.html#torch.Tensor.prod "torch.Tensor.prod")
 | 
 See
 [`torch.prod()`](generated/torch.prod.html#torch.prod "torch.prod")
 |
| 
[`Tensor.put\_`](generated/torch.Tensor.put_.html#torch.Tensor.put_ "torch.Tensor.put_")
 | 
 Copies the elements from
 `source`
 into the positions specified by
 `index`
.
  |
| 
[`Tensor.qr`](generated/torch.Tensor.qr.html#torch.Tensor.qr "torch.Tensor.qr")
 | 
 See
 [`torch.qr()`](generated/torch.qr.html#torch.qr "torch.qr")
 |
| 
[`Tensor.qscheme`](generated/torch.Tensor.qscheme.html#torch.Tensor.qscheme "torch.Tensor.qscheme")
 | 
 Returns the quantization scheme of a given QTensor.
  |
| 
[`Tensor.quantile`](generated/torch.Tensor.quantile.html#torch.Tensor.quantile "torch.Tensor.quantile")
 | 
 See
 [`torch.quantile()`](generated/torch.quantile.html#torch.quantile "torch.quantile")
 |
| 
[`Tensor.nanquantile`](generated/torch.Tensor.nanquantile.html#torch.Tensor.nanquantile "torch.Tensor.nanquantile")
 | 
 See
 [`torch.nanquantile()`](generated/torch.nanquantile.html#torch.nanquantile "torch.nanquantile")
 |
| 
[`Tensor.q\_scale`](generated/torch.Tensor.q_scale.html#torch.Tensor.q_scale "torch.Tensor.q_scale")
 | 
 Given a Tensor quantized by linear(affine) quantization, returns the scale of the underlying quantizer().
  |
| 
[`Tensor.q\_zero\_point`](generated/torch.Tensor.q_zero_point.html#torch.Tensor.q_zero_point "torch.Tensor.q_zero_point")
 | 
 Given a Tensor quantized by linear(affine) quantization, returns the zero\_point of the underlying quantizer().
  |
| 
[`Tensor.q\_per\_channel\_scales`](generated/torch.Tensor.q_per_channel_scales.html#torch.Tensor.q_per_channel_scales "torch.Tensor.q_per_channel_scales")
 | 
 Given a Tensor quantized by linear (affine) per-channel quantization, returns a Tensor of scales of the underlying quantizer.
  |
| 
[`Tensor.q\_per\_channel\_zero\_points`](generated/torch.Tensor.q_per_channel_zero_points.html#torch.Tensor.q_per_channel_zero_points "torch.Tensor.q_per_channel_zero_points")
 | 
 Given a Tensor quantized by linear (affine) per-channel quantization, returns a tensor of zero\_points of the underlying quantizer.
  |
| 
[`Tensor.q\_per\_channel\_axis`](generated/torch.Tensor.q_per_channel_axis.html#torch.Tensor.q_per_channel_axis "torch.Tensor.q_per_channel_axis")
 | 
 Given a Tensor quantized by linear (affine) per-channel quantization, returns the index of dimension on which per-channel quantization is applied.
  |
| 
[`Tensor.rad2deg`](generated/torch.Tensor.rad2deg.html#torch.Tensor.rad2deg "torch.Tensor.rad2deg")
 | 
 See
 [`torch.rad2deg()`](generated/torch.rad2deg.html#torch.rad2deg "torch.rad2deg")
 |
| 
[`Tensor.random\_`](generated/torch.Tensor.random_.html#torch.Tensor.random_ "torch.Tensor.random_")
 | 
 Fills
 `self`
 tensor with numbers sampled from the discrete uniform distribution over
 `[from,
 

 to
 

 -
 

 1]`
.
  |
| 
[`Tensor.ravel`](generated/torch.Tensor.ravel.html#torch.Tensor.ravel "torch.Tensor.ravel")
 | 
 see
 [`torch.ravel()`](generated/torch.ravel.html#torch.ravel "torch.ravel")
 |
| 
[`Tensor.reciprocal`](generated/torch.Tensor.reciprocal.html#torch.Tensor.reciprocal "torch.Tensor.reciprocal")
 | 
 See
 [`torch.reciprocal()`](generated/torch.reciprocal.html#torch.reciprocal "torch.reciprocal")
 |
| 
[`Tensor.reciprocal\_`](generated/torch.Tensor.reciprocal_.html#torch.Tensor.reciprocal_ "torch.Tensor.reciprocal_")
 | 
 In-place version of
 [`reciprocal()`](generated/torch.Tensor.reciprocal.html#torch.Tensor.reciprocal "torch.Tensor.reciprocal")
 |
| 
[`Tensor.record\_stream`](generated/torch.Tensor.record_stream.html#torch.Tensor.record_stream "torch.Tensor.record_stream")
 | 
 Ensures that the tensor memory is not reused for another tensor until all current work queued on
 `stream`
 are complete.
  |
| 
[`Tensor.register\_hook`](generated/torch.Tensor.register_hook.html#torch.Tensor.register_hook "torch.Tensor.register_hook")
 | 
 Registers a backward hook.
  |
| 
[`Tensor.register\_post\_accumulate\_grad\_hook`](generated/torch.Tensor.register_post_accumulate_grad_hook.html#torch.Tensor.register_post_accumulate_grad_hook "torch.Tensor.register_post_accumulate_grad_hook")
 | 
 Registers a backward hook that runs after grad accumulation.
  |
| 
[`Tensor.remainder`](generated/torch.Tensor.remainder.html#torch.Tensor.remainder "torch.Tensor.remainder")
 | 
 See
 [`torch.remainder()`](generated/torch.remainder.html#torch.remainder "torch.remainder")
 |
| 
[`Tensor.remainder\_`](generated/torch.Tensor.remainder_.html#torch.Tensor.remainder_ "torch.Tensor.remainder_")
 | 
 In-place version of
 [`remainder()`](generated/torch.Tensor.remainder.html#torch.Tensor.remainder "torch.Tensor.remainder")
 |
| 
[`Tensor.renorm`](generated/torch.Tensor.renorm.html#torch.Tensor.renorm "torch.Tensor.renorm")
 | 
 See
 [`torch.renorm()`](generated/torch.renorm.html#torch.renorm "torch.renorm")
 |
| 
[`Tensor.renorm\_`](generated/torch.Tensor.renorm_.html#torch.Tensor.renorm_ "torch.Tensor.renorm_")
 | 
 In-place version of
 [`renorm()`](generated/torch.Tensor.renorm.html#torch.Tensor.renorm "torch.Tensor.renorm")
 |
| 
[`Tensor.repeat`](generated/torch.Tensor.repeat.html#torch.Tensor.repeat "torch.Tensor.repeat")
 | 
 Repeats this tensor along the specified dimensions.
  |
| 
[`Tensor.repeat\_interleave`](generated/torch.Tensor.repeat_interleave.html#torch.Tensor.repeat_interleave "torch.Tensor.repeat_interleave")
 | 
 See
 [`torch.repeat\_interleave()`](generated/torch.repeat_interleave.html#torch.repeat_interleave "torch.repeat_interleave")
.
  |
| 
[`Tensor.requires\_grad`](generated/torch.Tensor.requires_grad.html#torch.Tensor.requires_grad "torch.Tensor.requires_grad")
 | 
 Is
 `True`
 if gradients need to be computed for this Tensor,
 `False`
 otherwise.
  |
| 
[`Tensor.requires\_grad\_`](generated/torch.Tensor.requires_grad_.html#torch.Tensor.requires_grad_ "torch.Tensor.requires_grad_")
 | 
 Change if autograd should record operations on this tensor: sets this tensor's
 `requires\_grad`
 attribute in-place.
  |
| 
[`Tensor.reshape`](generated/torch.Tensor.reshape.html#torch.Tensor.reshape "torch.Tensor.reshape")
 | 
 Returns a tensor with the same data and number of elements as
 `self`
 but with the specified shape.
  |
| 
[`Tensor.reshape\_as`](generated/torch.Tensor.reshape_as.html#torch.Tensor.reshape_as "torch.Tensor.reshape_as")
 | 
 Returns this tensor as the same shape as
 `other`
.
  |
| 
[`Tensor.resize\_`](generated/torch.Tensor.resize_.html#torch.Tensor.resize_ "torch.Tensor.resize_")
 | 
 Resizes
 `self`
 tensor to the specified size.
  |
| 
[`Tensor.resize\_as\_`](generated/torch.Tensor.resize_as_.html#torch.Tensor.resize_as_ "torch.Tensor.resize_as_")
 | 
 Resizes the
 `self`
 tensor to be the same size as the specified
 [`tensor`](generated/torch.tensor.html#torch.tensor "torch.tensor")
.
  |
| 
[`Tensor.retain\_grad`](generated/torch.Tensor.retain_grad.html#torch.Tensor.retain_grad "torch.Tensor.retain_grad")
 | 
 Enables this Tensor to have their
 `grad`
 populated during
 `backward()`
.
  |
| 
[`Tensor.retains\_grad`](generated/torch.Tensor.retains_grad.html#torch.Tensor.retains_grad "torch.Tensor.retains_grad")
 | 
 Is
 `True`
 if this Tensor is non-leaf and its
 `grad`
 is enabled to be populated during
 `backward()`
 ,
 `False`
 otherwise.
  |
| 
[`Tensor.roll`](generated/torch.Tensor.roll.html#torch.Tensor.roll "torch.Tensor.roll")
 | 
 See
 [`torch.roll()`](generated/torch.roll.html#torch.roll "torch.roll")
 |
| 
[`Tensor.rot90`](generated/torch.Tensor.rot90.html#torch.Tensor.rot90 "torch.Tensor.rot90")
 | 
 See
 [`torch.rot90()`](generated/torch.rot90.html#torch.rot90 "torch.rot90")
 |
| 
[`Tensor.round`](generated/torch.Tensor.round.html#torch.Tensor.round "torch.Tensor.round")
 | 
 See
 [`torch.round()`](generated/torch.round.html#torch.round "torch.round")
 |
| 
[`Tensor.round\_`](generated/torch.Tensor.round_.html#torch.Tensor.round_ "torch.Tensor.round_")
 | 
 In-place version of
 [`round()`](generated/torch.Tensor.round.html#torch.Tensor.round "torch.Tensor.round")
 |
| 
[`Tensor.rsqrt`](generated/torch.Tensor.rsqrt.html#torch.Tensor.rsqrt "torch.Tensor.rsqrt")
 | 
 See
 [`torch.rsqrt()`](generated/torch.rsqrt.html#torch.rsqrt "torch.rsqrt")
 |
| 
[`Tensor.rsqrt\_`](generated/torch.Tensor.rsqrt_.html#torch.Tensor.rsqrt_ "torch.Tensor.rsqrt_")
 | 
 In-place version of
 [`rsqrt()`](generated/torch.Tensor.rsqrt.html#torch.Tensor.rsqrt "torch.Tensor.rsqrt")
 |
| 
[`Tensor.scatter`](generated/torch.Tensor.scatter.html#torch.Tensor.scatter "torch.Tensor.scatter")
 | 
 Out-of-place version of
 [`torch.Tensor.scatter\_()`](generated/torch.Tensor.scatter_.html#torch.Tensor.scatter_ "torch.Tensor.scatter_")
 |
| 
[`Tensor.scatter\_`](generated/torch.Tensor.scatter_.html#torch.Tensor.scatter_ "torch.Tensor.scatter_")
 | 
 Writes all values from the tensor
 `src`
 into
 `self`
 at the indices specified in the
 `index`
 tensor.
  |
| 
[`Tensor.scatter\_add\_`](generated/torch.Tensor.scatter_add_.html#torch.Tensor.scatter_add_ "torch.Tensor.scatter_add_")
 | 
 Adds all values from the tensor
 `src`
 into
 `self`
 at the indices specified in the
 `index`
 tensor in a similar fashion as
 [`scatter\_()`](generated/torch.Tensor.scatter_.html#torch.Tensor.scatter_ "torch.Tensor.scatter_")
.
  |
| 
[`Tensor.scatter\_add`](generated/torch.Tensor.scatter_add.html#torch.Tensor.scatter_add "torch.Tensor.scatter_add")
 | 
 Out-of-place version of
 [`torch.Tensor.scatter\_add\_()`](generated/torch.Tensor.scatter_add_.html#torch.Tensor.scatter_add_ "torch.Tensor.scatter_add_")
 |
| 
[`Tensor.scatter\_reduce\_`](generated/torch.Tensor.scatter_reduce_.html#torch.Tensor.scatter_reduce_ "torch.Tensor.scatter_reduce_")
 | 
 Reduces all values from the
 `src`
 tensor to the indices specified in the
 `index`
 tensor in the
 `self`
 tensor using the applied reduction defined via the
 `reduce`
 argument (
 `"sum"`
 ,
 `"prod"`
 ,
 `"mean"`
 ,
 `"amax"`
 ,
 `"amin"`
 ).
  |
| 
[`Tensor.scatter\_reduce`](generated/torch.Tensor.scatter_reduce.html#torch.Tensor.scatter_reduce "torch.Tensor.scatter_reduce")
 | 
 Out-of-place version of
 [`torch.Tensor.scatter\_reduce\_()`](generated/torch.Tensor.scatter_reduce_.html#torch.Tensor.scatter_reduce_ "torch.Tensor.scatter_reduce_")
 |
| 
[`Tensor.select`](generated/torch.Tensor.select.html#torch.Tensor.select "torch.Tensor.select")
 | 
 See
 [`torch.select()`](generated/torch.select.html#torch.select "torch.select")
 |
| 
[`Tensor.select\_scatter`](generated/torch.Tensor.select_scatter.html#torch.Tensor.select_scatter "torch.Tensor.select_scatter")
 | 
 See
 [`torch.select\_scatter()`](generated/torch.select_scatter.html#torch.select_scatter "torch.select_scatter")
 |
| 
[`Tensor.set\_`](generated/torch.Tensor.set_.html#torch.Tensor.set_ "torch.Tensor.set_")
 | 
 Sets the underlying storage, size, and strides.
  |
| 
[`Tensor.share\_memory\_`](generated/torch.Tensor.share_memory_.html#torch.Tensor.share_memory_ "torch.Tensor.share_memory_")
 | 
 Moves the underlying storage to shared memory.
  |
| 
[`Tensor.short`](generated/torch.Tensor.short.html#torch.Tensor.short "torch.Tensor.short")
 | 
`self.short()`
 is equivalent to
 `self.to(torch.int16)`
.
  |
| 
[`Tensor.sigmoid`](generated/torch.Tensor.sigmoid.html#torch.Tensor.sigmoid "torch.Tensor.sigmoid")
 | 
 See
 [`torch.sigmoid()`](generated/torch.sigmoid.html#torch.sigmoid "torch.sigmoid")
 |
| 
[`Tensor.sigmoid\_`](generated/torch.Tensor.sigmoid_.html#torch.Tensor.sigmoid_ "torch.Tensor.sigmoid_")
 | 
 In-place version of
 [`sigmoid()`](generated/torch.Tensor.sigmoid.html#torch.Tensor.sigmoid "torch.Tensor.sigmoid")
 |
| 
[`Tensor.sign`](generated/torch.Tensor.sign.html#torch.Tensor.sign "torch.Tensor.sign")
 | 
 See
 [`torch.sign()`](generated/torch.sign.html#torch.sign "torch.sign")
 |
| 
[`Tensor.sign\_`](generated/torch.Tensor.sign_.html#torch.Tensor.sign_ "torch.Tensor.sign_")
 | 
 In-place version of
 [`sign()`](generated/torch.Tensor.sign.html#torch.Tensor.sign "torch.Tensor.sign")
 |
| 
[`Tensor.signbit`](generated/torch.Tensor.signbit.html#torch.Tensor.signbit "torch.Tensor.signbit")
 | 
 See
 [`torch.signbit()`](generated/torch.signbit.html#torch.signbit "torch.signbit")
 |
| 
[`Tensor.sgn`](generated/torch.Tensor.sgn.html#torch.Tensor.sgn "torch.Tensor.sgn")
 | 
 See
 [`torch.sgn()`](generated/torch.sgn.html#torch.sgn "torch.sgn")
 |
| 
[`Tensor.sgn\_`](generated/torch.Tensor.sgn_.html#torch.Tensor.sgn_ "torch.Tensor.sgn_")
 | 
 In-place version of
 [`sgn()`](generated/torch.Tensor.sgn.html#torch.Tensor.sgn "torch.Tensor.sgn")
 |
| 
[`Tensor.sin`](generated/torch.Tensor.sin.html#torch.Tensor.sin "torch.Tensor.sin")
 | 
 See
 [`torch.sin()`](generated/torch.sin.html#torch.sin "torch.sin")
 |
| 
[`Tensor.sin\_`](generated/torch.Tensor.sin_.html#torch.Tensor.sin_ "torch.Tensor.sin_")
 | 
 In-place version of
 [`sin()`](generated/torch.Tensor.sin.html#torch.Tensor.sin "torch.Tensor.sin")
 |
| 
[`Tensor.sinc`](generated/torch.Tensor.sinc.html#torch.Tensor.sinc "torch.Tensor.sinc")
 | 
 See
 [`torch.sinc()`](generated/torch.sinc.html#torch.sinc "torch.sinc")
 |
| 
[`Tensor.sinc\_`](generated/torch.Tensor.sinc_.html#torch.Tensor.sinc_ "torch.Tensor.sinc_")
 | 
 In-place version of
 [`sinc()`](generated/torch.Tensor.sinc.html#torch.Tensor.sinc "torch.Tensor.sinc")
 |
| 
[`Tensor.sinh`](generated/torch.Tensor.sinh.html#torch.Tensor.sinh "torch.Tensor.sinh")
 | 
 See
 [`torch.sinh()`](generated/torch.sinh.html#torch.sinh "torch.sinh")
 |
| 
[`Tensor.sinh\_`](generated/torch.Tensor.sinh_.html#torch.Tensor.sinh_ "torch.Tensor.sinh_")
 | 
 In-place version of
 [`sinh()`](generated/torch.Tensor.sinh.html#torch.Tensor.sinh "torch.Tensor.sinh")
 |
| 
[`Tensor.asinh`](generated/torch.Tensor.asinh.html#torch.Tensor.asinh "torch.Tensor.asinh")
 | 
 See
 [`torch.asinh()`](generated/torch.asinh.html#torch.asinh "torch.asinh")
 |
| 
[`Tensor.asinh\_`](generated/torch.Tensor.asinh_.html#torch.Tensor.asinh_ "torch.Tensor.asinh_")
 | 
 In-place version of
 [`asinh()`](generated/torch.Tensor.asinh.html#torch.Tensor.asinh "torch.Tensor.asinh")
 |
| 
[`Tensor.arcsinh`](generated/torch.Tensor.arcsinh.html#torch.Tensor.arcsinh "torch.Tensor.arcsinh")
 | 
 See
 [`torch.arcsinh()`](generated/torch.arcsinh.html#torch.arcsinh "torch.arcsinh")
 |
| 
[`Tensor.arcsinh\_`](generated/torch.Tensor.arcsinh_.html#torch.Tensor.arcsinh_ "torch.Tensor.arcsinh_")
 | 
 In-place version of
 [`arcsinh()`](generated/torch.Tensor.arcsinh.html#torch.Tensor.arcsinh "torch.Tensor.arcsinh")
 |
| 
[`Tensor.shape`](generated/torch.Tensor.shape.html#torch.Tensor.shape "torch.Tensor.shape")
 | 
 Returns the size of the
 `self`
 tensor.
  |
| 
[`Tensor.size`](generated/torch.Tensor.size.html#torch.Tensor.size "torch.Tensor.size")
 | 
 Returns the size of the
 `self`
 tensor.
  |
| 
[`Tensor.slogdet`](generated/torch.Tensor.slogdet.html#torch.Tensor.slogdet "torch.Tensor.slogdet")
 | 
 See
 [`torch.slogdet()`](generated/torch.slogdet.html#torch.slogdet "torch.slogdet")
 |
| 
[`Tensor.slice\_scatter`](generated/torch.Tensor.slice_scatter.html#torch.Tensor.slice_scatter "torch.Tensor.slice_scatter")
 | 
 See
 [`torch.slice\_scatter()`](generated/torch.slice_scatter.html#torch.slice_scatter "torch.slice_scatter")
 |
| 
[`Tensor.softmax`](generated/torch.Tensor.softmax.html#torch.Tensor.softmax "torch.Tensor.softmax")
 | 
 Alias for
 [`torch.nn.functional.softmax()`](generated/torch.nn.functional.softmax.html#torch.nn.functional.softmax "torch.nn.functional.softmax")
.
  |
| 
[`Tensor.sort`](generated/torch.Tensor.sort.html#torch.Tensor.sort "torch.Tensor.sort")
 | 
 See
 [`torch.sort()`](generated/torch.sort.html#torch.sort "torch.sort")
 |
| 
[`Tensor.split`](generated/torch.Tensor.split.html#torch.Tensor.split "torch.Tensor.split")
 | 
 See
 [`torch.split()`](generated/torch.split.html#torch.split "torch.split")
 |
| 
[`Tensor.sparse\_mask`](generated/torch.Tensor.sparse_mask.html#torch.Tensor.sparse_mask "torch.Tensor.sparse_mask")
 | 
 Returns a new
 [sparse tensor](sparse.html#sparse-docs)
 with values from a strided tensor
 `self`
 filtered by the indices of the sparse tensor
 `mask`
.
  |
| 
[`Tensor.sparse\_dim`](generated/torch.Tensor.sparse_dim.html#torch.Tensor.sparse_dim "torch.Tensor.sparse_dim")
 | 
 Return the number of sparse dimensions in a
 [sparse tensor](sparse.html#sparse-docs)
`self`
.
  |
| 
[`Tensor.sqrt`](generated/torch.Tensor.sqrt.html#torch.Tensor.sqrt "torch.Tensor.sqrt")
 | 
 See
 [`torch.sqrt()`](generated/torch.sqrt.html#torch.sqrt "torch.sqrt")
 |
| 
[`Tensor.sqrt\_`](generated/torch.Tensor.sqrt_.html#torch.Tensor.sqrt_ "torch.Tensor.sqrt_")
 | 
 In-place version of
 [`sqrt()`](generated/torch.Tensor.sqrt.html#torch.Tensor.sqrt "torch.Tensor.sqrt")
 |
| 
[`Tensor.square`](generated/torch.Tensor.square.html#torch.Tensor.square "torch.Tensor.square")
 | 
 See
 [`torch.square()`](generated/torch.square.html#torch.square "torch.square")
 |
| 
[`Tensor.square\_`](generated/torch.Tensor.square_.html#torch.Tensor.square_ "torch.Tensor.square_")
 | 
 In-place version of
 [`square()`](generated/torch.Tensor.square.html#torch.Tensor.square "torch.Tensor.square")
 |
| 
[`Tensor.squeeze`](generated/torch.Tensor.squeeze.html#torch.Tensor.squeeze "torch.Tensor.squeeze")
 | 
 See
 [`torch.squeeze()`](generated/torch.squeeze.html#torch.squeeze "torch.squeeze")
 |
| 
[`Tensor.squeeze\_`](generated/torch.Tensor.squeeze_.html#torch.Tensor.squeeze_ "torch.Tensor.squeeze_")
 | 
 In-place version of
 [`squeeze()`](generated/torch.Tensor.squeeze.html#torch.Tensor.squeeze "torch.Tensor.squeeze")
 |
| 
[`Tensor.std`](generated/torch.Tensor.std.html#torch.Tensor.std "torch.Tensor.std")
 | 
 See
 [`torch.std()`](generated/torch.std.html#torch.std "torch.std")
 |
| 
[`Tensor.stft`](generated/torch.Tensor.stft.html#torch.Tensor.stft "torch.Tensor.stft")
 | 
 See
 [`torch.stft()`](generated/torch.stft.html#torch.stft "torch.stft")
 |
| 
[`Tensor.storage`](generated/torch.Tensor.storage.html#torch.Tensor.storage "torch.Tensor.storage")
 | 
 Returns the underlying
 [`TypedStorage`](storage.html#torch.TypedStorage "torch.TypedStorage")
.
  |
| 
[`Tensor.untyped\_storage`](generated/torch.Tensor.untyped_storage.html#torch.Tensor.untyped_storage "torch.Tensor.untyped_storage")
 | 
 Returns the underlying
 [`UntypedStorage`](storage.html#torch.UntypedStorage "torch.UntypedStorage")
.
  |
| 
[`Tensor.storage\_offset`](generated/torch.Tensor.storage_offset.html#torch.Tensor.storage_offset "torch.Tensor.storage_offset")
 | 
 Returns
 `self`
 tensor's offset in the underlying storage in terms of number of storage elements (not bytes).
  |
| 
[`Tensor.storage\_type`](generated/torch.Tensor.storage_type.html#torch.Tensor.storage_type "torch.Tensor.storage_type")
 | 
 Returns the type of the underlying storage.
  |
| 
[`Tensor.stride`](generated/torch.Tensor.stride.html#torch.Tensor.stride "torch.Tensor.stride")
 | 
 Returns the stride of
 `self`
 tensor.
  |
| 
[`Tensor.sub`](generated/torch.Tensor.sub.html#torch.Tensor.sub "torch.Tensor.sub")
 | 
 See
 [`torch.sub()`](generated/torch.sub.html#torch.sub "torch.sub")
.
  |
| 
[`Tensor.sub\_`](generated/torch.Tensor.sub_.html#torch.Tensor.sub_ "torch.Tensor.sub_")
 | 
 In-place version of
 [`sub()`](generated/torch.Tensor.sub.html#torch.Tensor.sub "torch.Tensor.sub")
 |
| 
[`Tensor.subtract`](generated/torch.Tensor.subtract.html#torch.Tensor.subtract "torch.Tensor.subtract")
 | 
 See
 [`torch.subtract()`](generated/torch.subtract.html#torch.subtract "torch.subtract")
.
  |
| 
[`Tensor.subtract\_`](generated/torch.Tensor.subtract_.html#torch.Tensor.subtract_ "torch.Tensor.subtract_")
 | 
 In-place version of
 [`subtract()`](generated/torch.Tensor.subtract.html#torch.Tensor.subtract "torch.Tensor.subtract")
.
  |
| 
[`Tensor.sum`](generated/torch.Tensor.sum.html#torch.Tensor.sum "torch.Tensor.sum")
 | 
 See
 [`torch.sum()`](generated/torch.sum.html#torch.sum "torch.sum")
 |
| 
[`Tensor.sum\_to\_size`](generated/torch.Tensor.sum_to_size.html#torch.Tensor.sum_to_size "torch.Tensor.sum_to_size")
 | 
 Sum
 `this`
 tensor to
 `size`
.
  |
| 
[`Tensor.svd`](generated/torch.Tensor.svd.html#torch.Tensor.svd "torch.Tensor.svd")
 | 
 See
 [`torch.svd()`](generated/torch.svd.html#torch.svd "torch.svd")
 |
| 
[`Tensor.swapaxes`](generated/torch.Tensor.swapaxes.html#torch.Tensor.swapaxes "torch.Tensor.swapaxes")
 | 
 See
 [`torch.swapaxes()`](generated/torch.swapaxes.html#torch.swapaxes "torch.swapaxes")
 |
| 
[`Tensor.swapdims`](generated/torch.Tensor.swapdims.html#torch.Tensor.swapdims "torch.Tensor.swapdims")
 | 
 See
 [`torch.swapdims()`](generated/torch.swapdims.html#torch.swapdims "torch.swapdims")
 |
| 
[`Tensor.t`](generated/torch.Tensor.t.html#torch.Tensor.t "torch.Tensor.t")
 | 
 See
 [`torch.t()`](generated/torch.t.html#torch.t "torch.t")
 |
| 
[`Tensor.t\_`](generated/torch.Tensor.t_.html#torch.Tensor.t_ "torch.Tensor.t_")
 | 
 In-place version of
 [`t()`](generated/torch.Tensor.t.html#torch.Tensor.t "torch.Tensor.t")
 |
| 
[`Tensor.tensor\_split`](generated/torch.Tensor.tensor_split.html#torch.Tensor.tensor_split "torch.Tensor.tensor_split")
 | 
 See
 [`torch.tensor\_split()`](generated/torch.tensor_split.html#torch.tensor_split "torch.tensor_split")
 |
| 
[`Tensor.tile`](generated/torch.Tensor.tile.html#torch.Tensor.tile "torch.Tensor.tile")
 | 
 See
 [`torch.tile()`](generated/torch.tile.html#torch.tile "torch.tile")
 |
| 
[`Tensor.to`](generated/torch.Tensor.to.html#torch.Tensor.to "torch.Tensor.to")
 | 
 Performs Tensor dtype and/or device conversion.
  |
| 
[`Tensor.to\_mkldnn`](generated/torch.Tensor.to_mkldnn.html#torch.Tensor.to_mkldnn "torch.Tensor.to_mkldnn")
 | 
 Returns a copy of the tensor in
 `torch.mkldnn`
 layout.
  |
| 
[`Tensor.take`](generated/torch.Tensor.take.html#torch.Tensor.take "torch.Tensor.take")
 | 
 See
 [`torch.take()`](generated/torch.take.html#torch.take "torch.take")
 |
| 
[`Tensor.take\_along\_dim`](generated/torch.Tensor.take_along_dim.html#torch.Tensor.take_along_dim "torch.Tensor.take_along_dim")
 | 
 See
 [`torch.take\_along\_dim()`](generated/torch.take_along_dim.html#torch.take_along_dim "torch.take_along_dim")
 |
| 
[`Tensor.tan`](generated/torch.Tensor.tan.html#torch.Tensor.tan "torch.Tensor.tan")
 | 
 See
 [`torch.tan()`](generated/torch.tan.html#torch.tan "torch.tan")
 |
| 
[`Tensor.tan\_`](generated/torch.Tensor.tan_.html#torch.Tensor.tan_ "torch.Tensor.tan_")
 | 
 In-place version of
 [`tan()`](generated/torch.Tensor.tan.html#torch.Tensor.tan "torch.Tensor.tan")
 |
| 
[`Tensor.tanh`](generated/torch.Tensor.tanh.html#torch.Tensor.tanh "torch.Tensor.tanh")
 | 
 See
 [`torch.tanh()`](generated/torch.tanh.html#torch.tanh "torch.tanh")
 |
| 
[`Tensor.tanh\_`](generated/torch.Tensor.tanh_.html#torch.Tensor.tanh_ "torch.Tensor.tanh_")
 | 
 In-place version of
 [`tanh()`](generated/torch.Tensor.tanh.html#torch.Tensor.tanh "torch.Tensor.tanh")
 |
| 
[`Tensor.atanh`](generated/torch.Tensor.atanh.html#torch.Tensor.atanh "torch.Tensor.atanh")
 | 
 See
 [`torch.atanh()`](generated/torch.atanh.html#torch.atanh "torch.atanh")
 |
| 
[`Tensor.atanh\_`](generated/torch.Tensor.atanh_.html#torch.Tensor.atanh_ "torch.Tensor.atanh_")
 | 
 In-place version of
 [`atanh()`](generated/torch.Tensor.atanh.html#torch.Tensor.atanh "torch.Tensor.atanh")
 |
| 
[`Tensor.arctanh`](generated/torch.Tensor.arctanh.html#torch.Tensor.arctanh "torch.Tensor.arctanh")
 | 
 See
 [`torch.arctanh()`](generated/torch.arctanh.html#torch.arctanh "torch.arctanh")
 |
| 
[`Tensor.arctanh\_`](generated/torch.Tensor.arctanh_.html#torch.Tensor.arctanh_ "torch.Tensor.arctanh_")
 | 
 In-place version of
 [`arctanh()`](generated/torch.Tensor.arctanh.html#torch.Tensor.arctanh "torch.Tensor.arctanh")
 |
| 
[`Tensor.tolist`](generated/torch.Tensor.tolist.html#torch.Tensor.tolist "torch.Tensor.tolist")
 | 
 Returns the tensor as a (nested) list.
  |
| 
[`Tensor.topk`](generated/torch.Tensor.topk.html#torch.Tensor.topk "torch.Tensor.topk")
 | 
 See
 [`torch.topk()`](generated/torch.topk.html#torch.topk "torch.topk")
 |
| 
[`Tensor.to\_dense`](generated/torch.Tensor.to_dense.html#torch.Tensor.to_dense "torch.Tensor.to_dense")
 | 
 Creates a strided copy of
 `self`
 if
 `self`
 is not a strided tensor, otherwise returns
 `self`
.
  |
| 
[`Tensor.to\_sparse`](generated/torch.Tensor.to_sparse.html#torch.Tensor.to_sparse "torch.Tensor.to_sparse")
 | 
 Returns a sparse copy of the tensor.
  |
| 
[`Tensor.to\_sparse\_csr`](generated/torch.Tensor.to_sparse_csr.html#torch.Tensor.to_sparse_csr "torch.Tensor.to_sparse_csr")
 | 
 Convert a tensor to compressed row storage format (CSR).
  |
| 
[`Tensor.to\_sparse\_csc`](generated/torch.Tensor.to_sparse_csc.html#torch.Tensor.to_sparse_csc "torch.Tensor.to_sparse_csc")
 | 
 Convert a tensor to compressed column storage (CSC) format.
  |
| 
[`Tensor.to\_sparse\_bsr`](generated/torch.Tensor.to_sparse_bsr.html#torch.Tensor.to_sparse_bsr "torch.Tensor.to_sparse_bsr")
 | 
 Convert a tensor to a block sparse row (BSR) storage format of given blocksize.
  |
| 
[`Tensor.to\_sparse\_bsc`](generated/torch.Tensor.to_sparse_bsc.html#torch.Tensor.to_sparse_bsc "torch.Tensor.to_sparse_bsc")
 | 
 Convert a tensor to a block sparse column (BSC) storage format of given blocksize.
  |
| 
[`Tensor.trace`](generated/torch.Tensor.trace.html#torch.Tensor.trace "torch.Tensor.trace")
 | 
 See
 [`torch.trace()`](generated/torch.trace.html#torch.trace "torch.trace")
 |
| 
[`Tensor.transpose`](generated/torch.Tensor.transpose.html#torch.Tensor.transpose "torch.Tensor.transpose")
 | 
 See
 [`torch.transpose()`](generated/torch.transpose.html#torch.transpose "torch.transpose")
 |
| 
[`Tensor.transpose\_`](generated/torch.Tensor.transpose_.html#torch.Tensor.transpose_ "torch.Tensor.transpose_")
 | 
 In-place version of
 [`transpose()`](generated/torch.Tensor.transpose.html#torch.Tensor.transpose "torch.Tensor.transpose")
 |
| 
[`Tensor.triangular\_solve`](generated/torch.Tensor.triangular_solve.html#torch.Tensor.triangular_solve "torch.Tensor.triangular_solve")
 | 
 See
 [`torch.triangular\_solve()`](generated/torch.triangular_solve.html#torch.triangular_solve "torch.triangular_solve")
 |
| 
[`Tensor.tril`](generated/torch.Tensor.tril.html#torch.Tensor.tril "torch.Tensor.tril")
 | 
 See
 [`torch.tril()`](generated/torch.tril.html#torch.tril "torch.tril")
 |
| 
[`Tensor.tril\_`](generated/torch.Tensor.tril_.html#torch.Tensor.tril_ "torch.Tensor.tril_")
 | 
 In-place version of
 [`tril()`](generated/torch.Tensor.tril.html#torch.Tensor.tril "torch.Tensor.tril")
 |
| 
[`Tensor.triu`](generated/torch.Tensor.triu.html#torch.Tensor.triu "torch.Tensor.triu")
 | 
 See
 [`torch.triu()`](generated/torch.triu.html#torch.triu "torch.triu")
 |
| 
[`Tensor.triu\_`](generated/torch.Tensor.triu_.html#torch.Tensor.triu_ "torch.Tensor.triu_")
 | 
 In-place version of
 [`triu()`](generated/torch.Tensor.triu.html#torch.Tensor.triu "torch.Tensor.triu")
 |
| 
[`Tensor.true\_divide`](generated/torch.Tensor.true_divide.html#torch.Tensor.true_divide "torch.Tensor.true_divide")
 | 
 See
 [`torch.true\_divide()`](generated/torch.true_divide.html#torch.true_divide "torch.true_divide")
 |
| 
[`Tensor.true\_divide\_`](generated/torch.Tensor.true_divide_.html#torch.Tensor.true_divide_ "torch.Tensor.true_divide_")
 | 
 In-place version of
 [`true\_divide\_()`](generated/torch.Tensor.true_divide_.html#torch.Tensor.true_divide_ "torch.Tensor.true_divide_")
 |
| 
[`Tensor.trunc`](generated/torch.Tensor.trunc.html#torch.Tensor.trunc "torch.Tensor.trunc")
 | 
 See
 [`torch.trunc()`](generated/torch.trunc.html#torch.trunc "torch.trunc")
 |
| 
[`Tensor.trunc\_`](generated/torch.Tensor.trunc_.html#torch.Tensor.trunc_ "torch.Tensor.trunc_")
 | 
 In-place version of
 [`trunc()`](generated/torch.Tensor.trunc.html#torch.Tensor.trunc "torch.Tensor.trunc")
 |
| 
[`Tensor.type`](generated/torch.Tensor.type.html#torch.Tensor.type "torch.Tensor.type")
 | 
 Returns the type if
 
 dtype
 
 is not provided, else casts this object to the specified type.
  |
| 
[`Tensor.type\_as`](generated/torch.Tensor.type_as.html#torch.Tensor.type_as "torch.Tensor.type_as")
 | 
 Returns this tensor cast to the type of the given tensor.
  |
| 
[`Tensor.unbind`](generated/torch.Tensor.unbind.html#torch.Tensor.unbind "torch.Tensor.unbind")
 | 
 See
 [`torch.unbind()`](generated/torch.unbind.html#torch.unbind "torch.unbind")
 |
| 
[`Tensor.unflatten`](generated/torch.Tensor.unflatten.html#torch.Tensor.unflatten "torch.Tensor.unflatten")
 | 
 See
 [`torch.unflatten()`](generated/torch.unflatten.html#torch.unflatten "torch.unflatten")
.
  |
| 
[`Tensor.unfold`](generated/torch.Tensor.unfold.html#torch.Tensor.unfold "torch.Tensor.unfold")
 | 
 Returns a view of the original tensor which contains all slices of size
 `size`
 from
 `self`
 tensor in the dimension
 `dimension`
.
  |
| 
[`Tensor.uniform\_`](generated/torch.Tensor.uniform_.html#torch.Tensor.uniform_ "torch.Tensor.uniform_")
 | 
 Fills
 `self`
 tensor with numbers sampled from the continuous uniform distribution:
  |
| 
[`Tensor.unique`](generated/torch.Tensor.unique.html#torch.Tensor.unique "torch.Tensor.unique")
 | 
 Returns the unique elements of the input tensor.
  |
| 
[`Tensor.unique\_consecutive`](generated/torch.Tensor.unique_consecutive.html#torch.Tensor.unique_consecutive "torch.Tensor.unique_consecutive")
 | 
 Eliminates all but the first element from every consecutive group of equivalent elements.
  |
| 
[`Tensor.unsqueeze`](generated/torch.Tensor.unsqueeze.html#torch.Tensor.unsqueeze "torch.Tensor.unsqueeze")
 | 
 See
 [`torch.unsqueeze()`](generated/torch.unsqueeze.html#torch.unsqueeze "torch.unsqueeze")
 |
| 
[`Tensor.unsqueeze\_`](generated/torch.Tensor.unsqueeze_.html#torch.Tensor.unsqueeze_ "torch.Tensor.unsqueeze_")
 | 
 In-place version of
 [`unsqueeze()`](generated/torch.Tensor.unsqueeze.html#torch.Tensor.unsqueeze "torch.Tensor.unsqueeze")
 |
| 
[`Tensor.values`](generated/torch.Tensor.values.html#torch.Tensor.values "torch.Tensor.values")
 | 
 Return the values tensor of a
 [sparse COO tensor](sparse.html#sparse-coo-docs)
.
  |
| 
[`Tensor.var`](generated/torch.Tensor.var.html#torch.Tensor.var "torch.Tensor.var")
 | 
 See
 [`torch.var()`](generated/torch.var.html#torch.var "torch.var")
 |
| 
[`Tensor.vdot`](generated/torch.Tensor.vdot.html#torch.Tensor.vdot "torch.Tensor.vdot")
 | 
 See
 [`torch.vdot()`](generated/torch.vdot.html#torch.vdot "torch.vdot")
 |
| 
[`Tensor.view`](generated/torch.Tensor.view.html#torch.Tensor.view "torch.Tensor.view")
 | 
 Returns a new tensor with the same data as the
 `self`
 tensor but of a different
 `shape`
.
  |
| 
[`Tensor.view\_as`](generated/torch.Tensor.view_as.html#torch.Tensor.view_as "torch.Tensor.view_as")
 | 
 View this tensor as the same size as
 `other`
.
  |
| 
[`Tensor.vsplit`](generated/torch.Tensor.vsplit.html#torch.Tensor.vsplit "torch.Tensor.vsplit")
 | 
 See
 [`torch.vsplit()`](generated/torch.vsplit.html#torch.vsplit "torch.vsplit")
 |
| 
[`Tensor.where`](generated/torch.Tensor.where.html#torch.Tensor.where "torch.Tensor.where")
 | 
`self.where(condition,
 

 y)`
 is equivalent to
 `torch.where(condition,
 

 self,
 

 y)`
.
  |
| 
[`Tensor.xlogy`](generated/torch.Tensor.xlogy.html#torch.Tensor.xlogy "torch.Tensor.xlogy")
 | 
 See
 [`torch.xlogy()`](generated/torch.xlogy.html#torch.xlogy "torch.xlogy")
 |
| 
[`Tensor.xlogy\_`](generated/torch.Tensor.xlogy_.html#torch.Tensor.xlogy_ "torch.Tensor.xlogy_")
 | 
 In-place version of
 [`xlogy()`](generated/torch.Tensor.xlogy.html#torch.Tensor.xlogy "torch.Tensor.xlogy")
 |
| 
[`Tensor.zero\_`](generated/torch.Tensor.zero_.html#torch.Tensor.zero_ "torch.Tensor.zero_")
 | 
 Fills
 `self`
 tensor with zeros.
  |