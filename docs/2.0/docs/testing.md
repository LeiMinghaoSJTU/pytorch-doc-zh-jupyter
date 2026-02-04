# torch.testing [¶](#module-torch.testing "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/testing>
>
> 原始地址：<https://pytorch.org/docs/stable/testing.html>


 火炬测试。


 断言_close


 (*实际*、*预期*、***、*允许_子类



 =
 


 真*，*rtol



 =
 


 无*, *atol



 =
 


 无
* , *等于_nan



 =
 


 假*，*检查_设备



 =
 


 True
* , *检查_dtype



 =
 


 True
* , *检查_layout



 =
 


 True
* , *检查_stride



 =
 


 错误*，*消息



 =
 


 无
* ) [[source]](_modules/torch/testing/_comparison.html#assert_close)[¶](#torch.testing.assert_close "此定义的永久链接")


 断言“实际”和“预期”接近。


 如果“实际”和“预期”是跨步的、非量化的、实值的和有限的，则它们被认为是接近的，如果


 ∣ 实际 − 预期 ∣ ≤ atol 
+ rtol ⋅ ∣ 预期 ∣


 \lvert 	ext{实际} 
- 	ext{预期} 
vert \le 	exttt{atol} 
+ 	exttt{rtol} \cdot \lvert 	ext{预期} 
vert


 ∣
 


 实际的




 −
 


 预期的


 ∣
 



 ≤
 


 atol
 




 +
 


 rtol
 




 ⋅
 




 ∣
 


 预期的


 ∣
 


 非有限值(“-inf”和“inf”)仅当且仅当它们相等时才被视为接近。仅当“equal_nan”为“True”时，“NaN”才被视为彼此相等。


 此外，只有当它们具有相同的特征时才被认为是接近的



* [`device`](generated/torch.Tensor.device.html#torch.Tensor.device "torch.Tensor.device") (如果 `check_device` 为 `True` ),
* `dtype` (如果 ` check_dtype` 为 `True` )，
* `layout` (如果 `check_layout` 为 `True` )，以及
* stride (如果 `check_stride` 为 `True` )。


 如果“实际”或“预期”是元tensor，则仅执行属性检查。


 如果“实际”和“预期”稀疏(具有 COO、CSR、CSC、BSR 或 BSC 布局)，则分别检查其跨步成员。索引，即 COO 的“indices”，CSR 和 BSR 的“crow_indices”和“col_indices”，或者 CSC 和 BSC 布局的“ccol_indices”和“row_indices”，始终会检查是否相等而根据上面的定义检查值的接近度。


 如果“实际”和“预期”被量化，如果它们具有相同的 [`qscheme()`](generated/torch.Tensor.qscheme.html#torch.Tensor.qscheme "torch.Tensor.qscheme" ，则认为它们接近)并且 [`dequantize()`](generated/torch.Tensor.dequantize.html#torch.Tensor.dequantize "torch.Tensor.dequantize") 的结果根据上面的定义很接近。


`actual` 和 `expected` 可以是 [`Tensor`](tensors.html#torch.Tensor "torch.Tensor") 或 [`torch.Tensor`](tensors 中的任何tensor或标量类.html#torch.Tensor "torch.Tensor") 可以用 [`torch.as_tensor()`](generated/torch.as_tensor.html#torch.as_tensor "torch.as_tensor") 构造。除了 Python 标量之外，输入类型必须直接相关。另外，`actual`和`expected`可以是[`Sequence`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Sequence"(在Python v3.12中)") 'sor [`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping "(in Python v3.12)") 的其中如果它们的结构匹配，并且根据上述定义，它们的所有元素都被认为是接近的，则它们被认为是接近的。




!!! note "笔记"

    Python 标量是类型关系要求的一个例外，因为它们的 `type()` ，即 [`int`](https://docs.python.org/3/library/functions.html#int "(在 Python v3 中).12)") 、 [`float`](https://docs.python.org/3/library/functions.html#float "(在 Python v3.12)") 和 [`complex`](https ://docs.python.org/3/library/functions.html#complex "(in Python v3.12)") 相当于tensor的 `dtype`。因此，可以检查不同类型的Python标量，但需要 `check_dtype=False` 。


 参数 
* **实际** ( *Any
* ) – 实际输入。
* **预期** ( *Any
* ) – 预期输入。
* **允许_子类** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果为“True”(默认)且 Python 标量除外，则允许直接相关类型的输入。否则需要类型相等。
* **rtol** ( *可选
* *[
* [*float*](https://docs.python.org/3/library/functions.html#float "(在 Python v3.1 中) 12)")*]
* ) – 相对公差。如果指定了`atol`也必须指定。如果省略，则使用下表选择基于“dtype”的默认值。
* **atol** ( *可选
* *[
* [*float*](https://docs.python.org/3/library/functions.html#float "(在 Python v3.12 中)")*]
* ) – 绝对容差。如果指定，还必须指定“rtol”。如果省略，则使用下表选择基于“dtype”的默认值。
* **equal_nan** ( *Union
* *[
* [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中)")*,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3 中).12)")*]
* ) – 如果 `True` ，两个 `NaN` 值将被视为相等。
* **check_device** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果为“True”(默认)，则断言相应的tensor位于同一 [`device`](generated/torch.Tensor.device.html#torch.Tensor.device "torch.Tensor.device").如果禁用此检查，则不同 [`device`](generated/torch.Tensor.device.html#torch.Tensor.device "torch.Tensor.device") 上的tensor在比较之前会移动到 CPU。
* **check_dtype** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` ( default)，断言相应的tensor具有相同的 `dtype` 。如果禁用此检查，则具有不同“dtype”的tensor将提升为通用“dtype”(根据 [`torch.promote_types()`](generated/torch.promote_types.html#torch.promote_types "torch. Promotion_types") ) 在进行比较之前。
* **check_layout** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12 中) ") ) – 如果 `True` (默认)，则断言相应的tensor具有相同的 `layout` 。如果禁用此检查，则具有不同 `layout` 的tensor在比较之前将转换为跨步tensor。
* **check_stride** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果 `True` 和相应的tensor跨步，则断言它们具有相同的跨度。
* **msg** ( *可选
* *[
* *Union 
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*,
* *可调用
* *[
* *[ 
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* *]
* *]
* ) – 在比较过程中发生失败时使用的可选错误消息。也可以作为可调用传递，在这种情况下，它将使用生成的消息进行调用，并应返回新消息。


 引发 
* [**ValueError**](https://docs.python.org/3/library/exceptions.html#ValueError "(in Python v3.12)") – 如果没有 [`torch.Tensor`]( tensors.html#torch.Tensor "torch.Tensor") 可以从输入构造。
* [**ValueError**](https://docs.python.org/3/library/exceptions.html#ValueError "(在 Python v3.12 中)") – 如果仅指定了 `rtol` 或 `atol`。
* [**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError " (在 Python v3.12 中)") – 如果相应的输入不是 Python 标量并且不直接相关。
* [**AssertionError**](https://docs.python.org/3/library/exceptions.html# AssertionError "(in Python v3.12)") – 如果 `allow_subclasses` 为 `False` ，但相应的输入不是 Python 标量并且具有不同的类型。
* [**AssertionError**](https://docs. python.org/3/library/exceptions.html#AssertionError "(in Python v3.12)") – 如果输入是 [`Sequence`](https://docs.python.org/3/library/collections. abc.html#collections.abc.Sequence "(in Python v3.12)") 的，但它们的长度不匹配。
* [**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.12)") – 如果输入是 [`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections. abc.Mapping "(in Python v3.12)") 的，但它们的键集不匹配。
* [**AssertionError**](https://docs.python.org/3/library/exceptions. html#AssertionError "(in Python v3.12)") – 如果对应的tensor不具有相同的 [`shape`](generated/torch.Tensor.shape.html#torch.Tensor.shape "torch.Tensor.shape" ).
* [**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.12)") – 如果 `check_layout` 为 `True ` ，但相应的tensor不具有相同的 `layout` 。
* [**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(在 Python v3.12 中) ") – 如果只有一个对应的tensor被量化。
* [**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.12)") – 如果相应的tensor被量化，但具有不同的 [`qscheme()`](generated/torch.Tensor.qscheme.html#torch.Tensor.qscheme "torch.Tensor.qscheme") 's.
* [**AssertionError
* *](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.12)") – 如果 `check_device` 为 `True` ，但相应的tensor未打开相同的 [`device`](generated/torch.Tensor.device.html#torch.Tensor.device "torch.Tensor.device").
* [**AssertionError**](https://docs.python.org /3/library/exceptions.html#AssertionError "(in Python v3.12)") – 如果 `check_dtype` 为 `True` ，但对应的tensor不具有相同的 `dtype` 。
* [**AssertionError
* *](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.12)") – 如果 `check_stride` 为 `True` ，但相应的跨步tensor不具有相同的步幅。
* [**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.12)") – 如果相应tensor的值根据上面的定义，它们并不接近。


 下表显示了不同“dtype”的默认“rtol”和“atol”。如果“dtype”不匹配，则使用两个容差的最大值。


| 	`dtype`	 | 	`rtol`	 | 	`atol`	 |
| --- | --- | --- |
| 	`float16`	 | 	`1e-3`	 | 	`1e-5`	 |
| 	`bfloat16`	 | 	`1.6e-2`	 | 	`1e-5`	 |
| 	`float32`	 | 	`1.3e-6`	 | 	`1e-5`	 |
| 	`float64`	 | 	`1e-7`	 | 	`1e-7`	 |
| 	`complex32`	 | 	`1e-3`	 | 	`1e-5`	 |
| 	`complex64`	 | 	`1.3e-6`	 | 	`1e-5`	 |
| 	`complex128`	 | 	`1e-7`	 | 	`1e-7`	 |
| 	`quint8`	 | 	`1.3e-6`	 | 	`1e-5`	 |
| 	`quint2x4`	 | 	`1.3e-6`	 | 	`1e-5`	 |
| 	`quint4x2`	 | 	`1.3e-6`	 | 	`1e-5`	 |
| 	`qint8`	 | 	`1.3e-6`	 | 	`1e-5`	 |
| 	`qint32`	 | 	`1.3e-6`	 | 	`1e-5`	 |
| 	 other	  | 	`0.0`	 | 	`0.0`	 |




!!! note "笔记"

    [`assert_close()`](#torch.testing.assert_close "torch.testing.assert_close") 是高度可配置的，具有严格的默认设置。鼓励用户 [`partial()`](https://docs.python.org/3/library/functools.html#functools.partial "(in Python v3.12)") 它来适应他们的用例。例如，如果需要进行相等性检查，则可以定义一个“assert_equal”，默认情况下对每个“dtype”使用零容差：


```
>>> import functools
>>> assert_equal = functools.partial(torch.testing.assert_close, rtol=0, atol=0)
>>> assert_equal(1e-9, 1e-10)
Traceback (most recent call last):
...
AssertionError: Scalars are not equal!

Expected 1e-10 but got 1e-09.
Absolute difference: 9.000000000000001e-10
Relative difference: 9.0

```


 例子


```
>>> # tensor to tensor comparison
>>> expected = torch.tensor([1e0, 1e-1, 1e-2])
>>> actual = torch.acos(torch.cos(expected))
>>> torch.testing.assert_close(actual, expected)

```



```
>>> # scalar to scalar comparison
>>> import math
>>> expected = math.sqrt(2.0)
>>> actual = 2.0 / math.sqrt(2.0)
>>> torch.testing.assert_close(actual, expected)

```



```
>>> # numpy array to numpy array comparison
>>> import numpy as np
>>> expected = np.array([1e0, 1e-1, 1e-2])
>>> actual = np.arccos(np.cos(expected))
>>> torch.testing.assert_close(actual, expected)

```



```
>>> # sequence to sequence comparison
>>> import numpy as np
>>> # The types of the sequences do not have to match. They only have to have the same
>>> # length and their elements have to match.
>>> expected = [torch.tensor([1.0]), 2.0, np.array(3.0)]
>>> actual = tuple(expected)
>>> torch.testing.assert_close(actual, expected)

```



```
>>> # mapping to mapping comparison
>>> from collections import OrderedDict
>>> import numpy as np
>>> foo = torch.tensor(1.0)
>>> bar = 2.0
>>> baz = np.array(3.0)
>>> # The types and a possible ordering of mappings do not have to match. They only
>>> # have to have the same set of keys and their elements have to match.
>>> expected = OrderedDict([("foo", foo), ("bar", bar), ("baz", baz)])
>>> actual = {"baz": baz, "bar": bar, "foo": foo}
>>> torch.testing.assert_close(actual, expected)

```



```
>>> expected = torch.tensor([1.0, 2.0, 3.0])
>>> actual = expected.clone()
>>> # By default, directly related instances can be compared
>>> torch.testing.assert_close(torch.nn.Parameter(actual), expected)
>>> # This check can be made more strict with allow_subclasses=False
>>> torch.testing.assert_close(
...     torch.nn.Parameter(actual), expected, allow_subclasses=False
... )
Traceback (most recent call last):
...
TypeError: No comparison pair was able to handle inputs of type
<class 'torch.nn.parameter.Parameter'> and <class 'torch.Tensor'>.
>>> # If the inputs are not directly related, they are never considered close
>>> torch.testing.assert_close(actual.numpy(), expected)
Traceback (most recent call last):
...
TypeError: No comparison pair was able to handle inputs of type <class 'numpy.ndarray'>
and <class 'torch.Tensor'>.
>>> # Exceptions to these rules are Python scalars. They can be checked regardless of
>>> # their type if check_dtype=False.
>>> torch.testing.assert_close(1.0, 1, check_dtype=False)

```



```
>>> # NaN != NaN by default.
>>> expected = torch.tensor(float("Nan"))
>>> actual = expected.clone()
>>> torch.testing.assert_close(actual, expected)
Traceback (most recent call last):
...
AssertionError: Scalars are not close!

Expected nan but got nan.
Absolute difference: nan (up to 1e-05 allowed)
Relative difference: nan (up to 1.3e-06 allowed)
>>> torch.testing.assert_close(actual, expected, equal_nan=True)

```



```
>>> expected = torch.tensor([1.0, 2.0, 3.0])
>>> actual = torch.tensor([1.0, 4.0, 5.0])
>>> # The default error message can be overwritten.
>>> torch.testing.assert_close(actual, expected, msg="Argh, the tensors are not close!")
Traceback (most recent call last):
...
AssertionError: Argh, the tensors are not close!
>>> # If msg is a callable, it can be used to augment the generated message with
>>> # extra information
>>> torch.testing.assert_close(
...     actual, expected, msg=lambda msg: f"Header

{msg}

Footer"
... )
Traceback (most recent call last):
...
AssertionError: Header

Tensor-likes are not close!

Mismatched elements: 2 / 3 (66.7%)
Greatest absolute difference: 2.0 at index (1,) (up to 1e-05 allowed)
Greatest relative difference: 1.0 at index (1,) (up to 1.3e-06 allowed)

Footer

```


 火炬测试。


 使_tensor


 (
 
*\*
 


 形状*、*dtype*、*设备*、*低



 =
 


 无
* , *高



 =
 


 无
* , *需要_grad



 =
 


 假
* , *不连续



 =
 


 False
* , *排除_zero



 =
 


 False
* , *内存_格式



 =
 


 无
* ) [[source]](_modules/torch/testing/_creation.html#make_tensor)[¶](#torch.testing.make_tensor "此定义的永久链接")


 使用给定的 `shape` 、 `device` 和 `dtype` 创建一个tensor，并填充从 `[low, high)` 统一抽取的值。


 如果指定了“low”或“high”，并且超出了“dtype”可表示有限值的范围，则它们将分别被限制为最低或最高可表示有限值。如果为“None”，则下表描述了“low”和“high”的默认值取决于“dtype”。


| 	`dtype`	 | 	`low`	 | 	`high`	 |
| --- | --- | --- |
| 	 boolean type	  | 	`0`	 | 	`2`	 |
| 	 unsigned integral type	  | 	`0`	 | 	`10`	 |
| 	 signed integral types	  | 	`-9`	 | 	`10`	 |
| 	 floating types	  | 	`-9`	 | 	`9`	 |
| 	 complex types	  | 	`-9`	 | 	`9`	 |


 参数 
* **shape** ( *Tuple
* *[
* [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")
* ,
* *...
* *]
* ) – 定义输出tensor形状的单个整数或整数序列。
* **dtype** ( [`torch.dtype`](tensor_attributes.html#torch.dtype " torch.dtype") ) – 返回tensor的数据类型。
* **device** ( *Union
* *[
* [*str*](https://docs.python.org/3/library/stdtypes. html#str "(in Python v3.12)")*,
* [*torch.device*](tensor_attributes.html#torch.device "torch.device")*]
* ) – 返回tensor的设备。
* **low** ( *可选
* *[
* *Number
* *]
* ) – 设置给定范围的下限(含)。如果提供了一个数字，它将被限制为给定数据类型的最小可表示的有限值。当“无”(默认)时，该值根据“dtype”确定(参见上表)。默认值：`无`.
* **高** ( *可选
* *[
* *数量
* *]
* ) –


 设置给定范围的上限(不包括上限)。如果提供了一个数字，它将被限制为给定数据类型的最大可表示有限值。当“无”(默认)时，该值是根据“dtype”确定的(参见上表)。默认值：`无`。


 自 2.1 版起已弃用：自 2.1 版起已弃用将浮点或复杂类型的 `low==high` 传递给 [`make_tensor()`](#torch.testing.make_tensor "torch.testing.make_tensor")，并将在2.3.使用 [`torch.full()`](generated/torch.full.html#torch.full "torch.full") 代替。
* **需要_grad** ( *可选
* *[
* [*bool*] (https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*]
* ) – autograd 是否应记录对返回tensor的操作。默认值：`False`.
* **不连续** ( *可选
* *[
* [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.1 中) 12)")*]
* ) – 如果 True ，返回的tensor将是不连续的。如果构造的tensor少于两个元素，则忽略此参数。与 `memory_format` 互斥。
* **排除_zero** ( *可选
* *[
* [*bool*](https://docs.python.org/3/library/functions.html#bool " (在 Python v3.12 中)")*]
* ) – 如果 `True` 则根据 `dtype` 将零替换为 dtype 的小正值。对于布尔和整数类型，零被替换为一。对于浮点类型，它被替换为 dtype 的最小正正规数(`dtype` 的 `finfo()` 对象的“微小”值)，对于复杂类型，它被替换为复数，其实部和虚部为两者都是复数类型可表示的最小正正规数。默认 `False`.
* **memory_format** ( *可选
* *[
* [*torch.memory_format*](tensor_attributes.html#torch.memory_format "torch.memory_format")*]
* ) – 内存返回的tensor的格式。与“非连续”互斥。


 引发 
* [**ValueError**](https://docs.python.org/3/library/exceptions.html#ValueError "(in Python v3.12)") – 如果传递 `requires_grad=True`对于积分 dtype
* [**ValueError**](https://docs.python.org/3/library/exceptions.html#ValueError "(in Python v3.12)") – 如果 `low >= high` 。 
* [**ValueError**](https://docs.python.org/3/library/exceptions.html#ValueError "(in Python v3.12)") – 如果 `low` 或 `high` 为 ` nan`.
* [**ValueError**](https://docs.python.org/3/library/exceptions.html#ValueError "(in Python v3.12)") – 如果 `noncontigulous` 和 `memory _format` 被传递。
* [**TypeError**](https://docs.python.org/3/library/exceptions.html#TypeError "(in Python v3.12)") – 如果 `dtype` 不是不支持此功能。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> from torch.testing import make_tensor
>>> # Creates a float tensor with values in [-1, 1)
>>> make_tensor((3,), device='cpu', dtype=torch.float32, low=-1, high=1)
tensor([ 0.1205, 0.2282, -0.6380])
>>> # Creates a bool tensor on CUDA
>>> make_tensor((2, 2), device='cuda', dtype=torch.bool)
tensor([[False, False],
 [False, True]], device='cuda:0')

```


 火炬测试。


 断言_allclose


 (*实际*、*预期*、*rtol



 =
 


 无*, *atol



 =
 


 无
* , *等于_nan



 =
 


 真*，*消息



 =
 


 ''
* ) [[source]](_modules/torch/testing/_comparison.html#assert_allclose)[¶](#torch.testing.assert_allclose "此定义的永久链接")


!!! warning "警告"

    [`torch.testing.assert_allclose()`](#torch.testing.assert_allclose "torch.testing.assert_allclose") 自 `1.12` 起已弃用，并将在未来版本中删除。请使用 [`torch.testing.assert_close()`](#torch.testing.assert_close "torch.testing.assert_close") 代替。您可以在[此处](https://github.com/pytorch/pytorch/issues/61844)找到详细的升级说明。