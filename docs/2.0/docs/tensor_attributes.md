# tensor属性 [¶](#tensor-attributes "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/tensor_attributes>
>
> 原始地址：<https://pytorch.org/docs/stable/tensor_attributes.html>


 每个 `torch.Tensor` 都有一个 [`torch.dtype`](#torch.dtype "torch.dtype") 、 [`torch.device`](#torch.device "torch.device") 和 [`torch.layout`](#torch.layout "torch.layout").


## torch.dtype [¶](#torch-dtype "此标题的永久链接")


*班级*


 火炬。


 dtype [¶](#torch.dtype "此定义的永久链接")


 [`torch.dtype`](#torch.dtype "torch.dtype") 是一个表示 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 数据类型的对象。 PyTorch 有十二种不同的数据类型：


| 	 Data type	  | 	 dtype	  | 	 Legacy Constructors	  |
| --- | --- | --- |
| 	 32-bit floating point	  | 	`torch.float32`	 or	 `torch.float`	 | 	`torch.*.FloatTensor`	 |
| 	 64-bit floating point	  | 	`torch.float64`	 or	 `torch.double`	 | 	`torch.*.DoubleTensor`	 |
| 	 64-bit complex	  | 	`torch.complex64`	 or	 `torch.cfloat`	 |  |
| 	 128-bit complex	  | 	`torch.complex128`	 or	 `torch.cdouble`	 |  |
| 	 16-bit floating point	 [1](#id3) 	 | 	`torch.float16`	 or	 `torch.half`	 | 	`torch.*.HalfTensor`	 |
| 	 16-bit floating point	 [2](#id4) 	 | 	`torch.bfloat16`	 | 	`torch.*.BFloat16Tensor`	 |
| 	 8-bit integer (unsigned)	  | 	`torch.uint8`	 | 	`torch.*.ByteTensor`	 |
| 	 8-bit integer (signed)	  | 	`torch.int8`	 | 	`torch.*.CharTensor`	 |
| 	 16-bit integer (signed)	  | 	`torch.int16`	 or	 `torch.short`	 | 	`torch.*.ShortTensor`	 |
| 	 32-bit integer (signed)	  | 	`torch.int32`	 or	 `torch.int`	 | 	`torch.*.IntTensor`	 |
| 	 64-bit integer (signed)	  | 	`torch.int64`	 or	 `torch.long`	 | 	`torch.*.LongTensor`	 |
| 	 Boolean	  | 	`torch.bool`	 | 	`torch.*.BoolTensor`	 | [1](#id1)


 有时称为 binary16：使用 1 个符号、5 个指数和 10 个有效数字位。当精度很重要时很有用。


[2](#id2)


 有时称为 Brain 浮点：使用 1 个符号、8 个指数和 7 个有效数字位。当范围很重要时很有用，因为它具有与“float32”相同的指数位数


 要查明 [`torch.dtype`](#torch.dtype "torch.dtype") 是否为浮点数据类型，可使用属性 [`is_floating_point`](generated/torch.is_floating_point.html#可以使用 torch.is_floating_point "torch.is_floating_point")，如果数据类型是浮点数据类型，则返回 `True`。


 要查明 [`torch.dtype`](#torch.dtype "torch.dtype") 是否是复杂数据类型，属性 [`is_complex`](generated/torch.is_complex.html#torch.is_complex可以使用“torch.is_complex”)，如果数据类型是复杂数据类型，则返回“True”。


 当算术运算( add 、 sub 、 div 、 mul )的输入数据类型不同时，我们通过查找满足以下规则的最小数据类型来进行提升：



* 如果标量操作数的类型属于比tensor操作数更高的类别(其中复数>浮点>积分>布尔)，我们将提升为具有足够大小的类型以容纳该类别的所有标量操作数。*如果零维tensor操作数具有比维度操作数更高的类别，我们将提升为具有足够大小和类别的类型来容纳该类别的所有零维tensor操作数。*如果没有更高类别的零维操作数，我们将提升为具有足够大小和类别的类型。 size和category来保存所有有尺寸的操作数。


 浮点标量操作数的数据类型为 torch.get_default_dtype() ，整数非布尔标量操作数的数据类型为 torch.int64 。与 numpy 不同，我们在确定操作数的最小数据类型时不检查值。尚不支持量化和复杂类型。


 促销示例：


```
>>> float_tensor = torch.ones(1, dtype=torch.float)
>>> double_tensor = torch.ones(1, dtype=torch.double)
>>> complex_float_tensor = torch.ones(1, dtype=torch.complex64)
>>> complex_double_tensor = torch.ones(1, dtype=torch.complex128)
>>> int_tensor = torch.ones(1, dtype=torch.int)
>>> long_tensor = torch.ones(1, dtype=torch.long)
>>> uint_tensor = torch.ones(1, dtype=torch.uint8)
>>> double_tensor = torch.ones(1, dtype=torch.double)
>>> bool_tensor = torch.ones(1, dtype=torch.bool)
# zero-dim tensors
>>> long_zerodim = torch.tensor(1, dtype=torch.long)
>>> int_zerodim = torch.tensor(1, dtype=torch.int)

>>> torch.add(5, 5).dtype
torch.int64
# 5 is an int64, but does not have higher category than int_tensor so is not considered.
>>> (int_tensor + 5).dtype
torch.int32
>>> (int_tensor + long_zerodim).dtype
torch.int32
>>> (long_tensor + int_tensor).dtype
torch.int64
>>> (bool_tensor + long_tensor).dtype
torch.int64
>>> (bool_tensor + uint_tensor).dtype
torch.uint8
>>> (float_tensor + double_tensor).dtype
torch.float64
>>> (complex_float_tensor + complex_double_tensor).dtype
torch.complex128
>>> (bool_tensor + int_tensor).dtype
torch.int32
# Since long is a different kind than float, result dtype only needs to be large enough
# to hold the float.
>>> torch.add(long_tensor, float_tensor).dtype
torch.float32

```


 当指定算术运算的输出tensor时，我们允许转换为其数据类型，但以下情况除外： 
* 整数输出tensor不能接受浮点tensor。 
* 布尔输出tensor不能接受非布尔tensor。 
* 非复数输出tensor不能接受复tensor


 铸造实例：


```
# allowed:
>>> float_tensor *= float_tensor
>>> float_tensor *= int_tensor
>>> float_tensor *= uint_tensor
>>> float_tensor *= bool_tensor
>>> float_tensor *= double_tensor
>>> int_tensor *= long_tensor
>>> int_tensor *= uint_tensor
>>> uint_tensor *= int_tensor

# disallowed (RuntimeError: result type can't be cast to the desired output type):
>>> int_tensor *= float_tensor
>>> bool_tensor *= int_tensor
>>> bool_tensor *= uint_tensor
>>> float_tensor *= complex_float_tensor

```


## torch.device [¶](#torch-device "此标题的永久链接")


*班级*


 火炬。


 device [¶](#torch.device"此定义的永久链接")


 [`torch.device`](#torch.device "torch.device") 是一个表示 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") isor 的设备的对象将被分配。


 [`torch.device`](#torch.device "torch.device") 包含设备类型(`'cpu'`、`'cuda'` 或 `'mps'` )和设备类型的可选 deviceordinal。如果设备序号不存在，则该对象将始终代表该设备类型的当前设备，即使在 [`torch.cuda.set_device()`](generated/torch.cuda.set_device.html#torch.cuda 之后也是如此。 set_device“torch.cuda.set_device”)被调用；例如，使用设备 `'cuda'` 构造的 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 相当于 `'cuda:X'`，其中 X 是 [`torch' 的结果.cuda.current_device()`](generated/torch.cuda.current_device.html#torch.cuda.current_device "torch.cuda.current_device").


 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 的设备可以通过 [`Tensor.device`](generated/torch.Tensor.device.html#torch. Tensor.device“torch.Tensor.device”)属性。


 [`torch.device`](#torch.device "torch.device") 可以通过字符串或通过字符串和设备序号构造


 通过字符串：


```
>>> torch.device('cuda:0')
device(type='cuda', index=0)

>>> torch.device('cpu')
device(type='cpu')

>>> torch.device('mps')
device(type='mps')

>>> torch.device('cuda')  # current cuda device
device(type='cuda')

```


 通过字符串和设备序数：


```
>>> torch.device('cuda', 0)
device(type='cuda', index=0)

>>> torch.device('mps', 0)
device(type='mps', index=0)

>>> torch.device('cpu', 0)
device(type='cpu', index=0)

```


 设备对象还可以用作上下文管理器来更改分配的默认设备tensor：


```
>>> with torch.device('cuda:1'):
...     r = torch.randn(2, 3)
>>> r.device
device(type='cuda', index=1)

```


 如果向工厂函数传递显式的非 None 设备参数，则此上下文管理器不起作用。要全局更改默认设备，另请参阅 [`torch.set_default_device()`](generated/torch.set_default_device.html#torch.set_default_device "torch.set_default_device") 。


!!! warning "警告"

     此函数对 torch API 的每个 Python 调用(不仅仅是工厂函数)都会产生轻微的性能成本。如果这给您带来问题，请评论 <https://github.com/pytorch/pytorch/issues/92701>



!!! note "笔记"

    函数中的 [`torch.device`](#torch.device "torch.device") 参数通常可以用字符串替换。这样可以快速构建代码原型。


```
>>> # Example of a function that takes in a torch.device
>>> cuda1 = torch.device('cuda:1')
>>> torch.randn((2,3), device=cuda1)

```



```
>>> # You can substitute the torch.device with a string
>>> torch.randn((2,3), device='cuda:1')

```



!!! note "笔记"

    由于遗留原因，可以通过单个设备序号构建设备，该设备被视为 cuda 设备。这与 [`Tensor.get_device()`](generated/torch.Tensor.get_device.html#torch.Tensor.get_device "torch.Tensor.get_device") 匹配，它返回 cudatensors 的序数，并且 cpu 不支持tensor。


```
>>> torch.device(1)
device(type='cuda', index=1)

```



!!! note "笔记"

    采用设备的方法通常会接受(正确格式的)字符串或(传统)整数设备序数，即以下内容都是等效的：


```
>>> torch.randn((2,3), device=torch.device('cuda:1'))
>>> torch.randn((2,3), device='cuda:1')
>>> torch.randn((2,3), device=1)  # legacy

```


## torch.layout [¶](#torch-layout "此标题的永久链接")


*班级*


 火炬。


 布局 [¶](#torch.layout "此定义的永久链接")


!!! warning "警告"

     `torch.layout` 类处于测试阶段，可能会发生变化。


 [`torch.layout`](#torch.layout "torch.layout") 是一个表示 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 内存布局的对象。目前，我们支持“torch.strided”(密集tensor)，并对“torch.sparse_coo”(稀疏 COO tensor)提供测试版支持。


“torch.strided”表示密集tensor，是最常用的内存布局。每个跨步tensor都有一个关联的“torch.Storage”，用于保存其数据。这些tensor提供了存储的多维跨步视图。步长是一个整数列表：第 k 个步长表示在tensor的第 k 维中从一个元素到下一个元素所需的内存跳转。这个概念使得高效地执行许多tensor运算成为可能。


 例子：


```
>>> x = torch.tensor([[1, 2, 3, 4, 5], [6, 7, 8, 9, 10]])
>>> x.stride()
(5, 1)

>>> x.t().stride()
(1, 5)

```


 有关“torch.sparse_coo”tensor的更多信息，请参阅 [torch.sparse](sparse.html#sparse-docs) 。


## torch.memory_format [¶](#torch-memory-format "此标题的永久链接")


*班级*


 火炬。


 memory_format [¶](#torch.memory_format "此定义的永久链接")


 [`torch.memory_format`](#torch.memory_format "torch.memory_format") 是一个对象，表示 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor" 所在的内存格式") isor 将被分配。


 可能的值为：



* `torch.contigious_format` ：tensor正在或将被分配在密集的非重叠内存中。步幅由按降序排列的值表示。
* `torch.channels_last` ：tensor正在或将被分配在密集的非重叠内存中。由 `strides[0] 
> strides[2] 
> strides[3] 
> strides[1] == 1` 又名 NHWC order.
* `torch.channels_last_3d` 中的值表示的步幅：tensor已经或将被分配在密集的非重叠内存中。步幅由 `strides[0] 
> strides[2] 
> strides[3] 
> strides[4] 
> strides[1] == 1` 又名 NDHWC order 中的值表示。
* `torch.preserve_format` ：在函数中使用像克隆一样保留输入tensor的内存格式。如果输入tensor分配在密集的非重叠内存中，则输出tensor步幅将从输入复制。否则输出步幅将遵循“torch.contigious_format”