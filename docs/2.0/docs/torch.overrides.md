# torch.overrides [¶](#torch-overrides "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/torch.overrides>
>
> 原始地址：<https://pytorch.org/docs/stable/torch.overrides.html>


 该模块公开了 `__torch_function__` 协议的各种辅助函数。有关 `__torch_function__` 协议的更多详细信息，请参阅[扩展 torch Python API](notes/extending.html#extending-torch-python)。


## 函数 [¶](#functions "此标题的永久链接")


 火炬.覆盖。


 获取_忽略_函数


 ( ) [[source]](_modules/torch/overrides.html#get_ignored_functions)[¶](#torch.overrides.get_ignored_functions "此定义的永久链接")


 返回不能被 `__torch_function__` 覆盖的公共函数。


 退货


 在 torch API 中公开可用但不能用 `__torch_function__` 覆盖的函数元组。这主要是因为这些函数的参数都不是tensor或类tensor。


 Return type


 设置[可调用]


 例子


```
>>> torch.Tensor.as_subclass in torch.overrides.get_ignored_functions()
True
>>> torch.add in torch.overrides.get_ignored_functions()
False

```


 火炬.覆盖。


 获取可覆盖的函数


 ( ) [[source]](_modules/torch/overrides.html#get_overridable_functions)[¶](#torch.overrides.get_overridable_functions "此定义的永久链接")


 列出可通过 __torch_function__ 覆盖的函数


 退货


 将包含可重写函数的命名空间映射到该命名空间中可重写的函数的字典。


 Return type


 字典[任意，列表[可调用]]


 火炬.覆盖。


 解析_name


 ( *f
* ) [[source]](_modules/torch/overrides.html#resolve_name)[¶](#torch.overrides.resolve_name "此定义的永久链接")


 获取传递给__torch_function__ 的函数的人类可读字符串名称


 Parameters


**f** ( *Callable
* ) – 解析名称的函数。


 退货


 函数名称；如果被评估，它应该返回输入函数。


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 火炬.覆盖。


 获取_测试_覆盖


 ( ) [[source]](_modules/torch/overrides.html#get_testing_overrides)[¶](#torch.overrides.get_testing_overrides "此定义的永久链接")


 返回一个包含所有可重写函数的虚拟重写的字典


 退货


 一个字典，将 PyTorch API 中的可重写函数映射到与实际函数具有相同签名的 lambda 函数，并无条件返回 -1。这些 lambda 函数对于测试定义 `__torch_function__` 的类型的 API 覆盖率非常有用。


 Return type


 字典[可调用，可调用]


 例子


```
>>> import inspect
>>> my_add = torch.overrides.get_testing_overrides()[torch.add]
>>> inspect.signature(my_add)
<Signature (input, other, out=None)>

```


 火炬.覆盖。


 手柄_火炬_功能


 ( *public_api
* , *相关_args
* , **


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/overrides.html#handle_torch_function)[¶](#torch.overrides.handle_torch_function "此定义的永久链接")


 实现一个函数并检查 `__torch_function__` 覆盖。


 请参阅 torch::autograd::handle_torch_function 以了解 C++ 实现中此函数的等效项。


 参数 
* **public_api** ( *function
* ) – 由公共 torch API 公开的函数，最初称为“public_api(*args, **kwargs)”，现在正在检查其参数。
* 
* *relevant_args** ( *iterable
* ) – 用于检查 __torch_function__ 方法的可迭代参数。
* **args** ( [*tuple*](https://docs.python.org/3/library/stdtypes.html#tuple "(in Python v3.12)") ) – 最初传递到 `public_api` 的任意位置参数.
* **kwargs** ( [*tuple*](https ://docs.python.org/3/library/stdtypes.html#tuple "(in Python v3.12)") ) – 最初传递到 `public_api` 的任意关键字参数。


 退货


 根据需要调用 `implementation` 或 `__torch_function__` 方法的结果。


 Return type


[对象](https://docs.python.org/3/library/functions.html#object“(在Python v3.12中)”)


 ：引发 TypeError ：如果没有找到实现。：


 例子


```
>>> def func(a):
...     if has_torch_function_unary(a):
...         return handle_torch_function(func, (a,), a)
...     return a + 0

```


 火炬.覆盖。


 具有_手电筒_功能


 ( ) [¶](#torch.overrides.has_torch_function "此定义的永久链接")


 如果启用了 __torch_function__ 模式，请检查可迭代元素中的 __torch_function__ 实现。考虑精确的“Tensor”和“Parameter”是不可分派的。用它来保护对 [`handle_torch_function()`](#torch.overrides.handle_torch_function "torch.overrides.handle_torch_function") 的调用；不要用它来测试某些东西是否类似于 Tensor，请使用 [`is_tensor_like()`](#torch.overrides.is_tensor_like "torch.overrides.is_tensor_like") 代替。:param related_args: Iterable 或用于检查 __torch_function__ 方法的参数。:类型相关_args: 可迭代


 退货


 如果相关参数的任何元素具有 __torch_function__implementations，则为 True，否则为 False。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 也可以看看


`torch.is_tensor_like`


 检查某些东西是否是类似 Tensor 的，包括精确的 `Tensor` 。


 火炬.覆盖。


 是_tensor_like


 ( *inp
* ) [[source]](_modules/torch/overrides.html#is_tensor_like)[¶](#torch.overrides.is_tensor_like "此定义的永久链接")


 如果传入的输入是类似tensor的，则返回“True”。


 目前，只要输入类型上存在 `__torch_function__` 属性，就会发生这种情况。


 例子


 tensor的子类通常是类tensor的。


```
>>> class SubTensor(torch.Tensor): ...
>>> is_tensor_like(SubTensor([0]))
True

```


 内置类型或用户类型通常与tensor不同。


```
>>> is_tensor_like(6)
False
>>> is_tensor_like(None)
False
>>> class NotATensor: ...
>>> is_tensor_like(NotATensor())
False

```


 但是，可以通过实现 __torch_function__ 使它们类似于tensor。


```
>>> class TensorLike:
...     @classmethod
...     def __torch_function__(cls, func, types, args, kwargs):
...         return -1
>>> is_tensor_like(TensorLike())
True

```


 火炬.覆盖。


 是_tensor_方法_或_属性


 ( *func
* ) [[source]](_modules/torch/overrides.html#is_tensor_method_or_property)[¶](#torch.overrides.is_tensor_method_or_property "此定义的永久链接")


 如果传入的函数是属于 `torch.Tensor` 的方法或属性的处理程序，则返回 True，如传递到 `__torch_function__` 。




!!! note "笔记"

    对于属性，必须传入其`__get__`方法。


 特别是出于以下原因，可能需要这样做：


1. 方法/属性有时不包含 __module__ slot.2.它们要求第一个传入的参数是 `torch.Tensor` 的实例。


 例子


```
>>> is_tensor_method_or_property(torch.Tensor.add)
True
>>> is_tensor_method_or_property(torch.add)
False

```


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 火炬.覆盖。


 包装_火炬_功能


 ( *dispatcher
* ) [[source]](_modules/torch/overrides.html#wrap_torch_function)[¶](#torch.overrides.wrap_torch_function "此定义的永久链接")


 用 `__torch_function__` 相关功能包装给定的函数。


 Parameters


**dispatcher** ( *Callable
* ) – 一个可调用函数，返回传递到函数中的类似 Tensor 的可迭代对象。



!!! note "笔记"

    此装饰器可能会降低代码的性能。一般来说，将代码表达为一系列函数就足够了，这些函数本身支持 __torch_function__。如果您发现自己处于罕见的情况，但情况并非如此，例如如果您正在包装低级库并且还需要它为类似 Tensor 的工作，那么此功能可用。


 例子


```
>>> def dispatcher(a): # Must have the same signature as func
...     return (a,)
>>> @torch.overrides.wrap_torch_function(dispatcher)
>>> def func(a): # This will make func dispatchable by __torch_function__
...     return a + 0

```