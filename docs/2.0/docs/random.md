# torch.random [¶](#module-torch.random "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/random>
>
> 原始地址：<https://pytorch.org/docs/stable/random.html>


 火炬.随机。


 叉_rng


 ( *设备



 =
 


 无
* , *启用



 =
 


 真*，*_caller



 =
 


 'fork_rng'
* , *_devices_kw



 =
 


 '设备'
* , *设备_类型



 =
 


 'cuda'
* ) [[source]](_modules/torch/random.html#fork_rng)[¶](#torch.random.fork_rng "此定义的永久链接")


 分叉 RNG，以便当您返回时，RNG 会重置为之前的状态。


 参数 
* **devices** ( *iterable
* *of
* *Device IDs
* ) – 要分叉 RNG 的设备。 CPU RNG 状态始终是分叉的。默认情况下，[`fork_rng()`](#torch.random.fork_rng "torch.random.fork_rng") 在所有设备上运行，但如果您的机器有很多设备，则会发出警告，因为此函数运行速度非常慢在这种情况下慢慢地。如果您显式指定设备，此警告将被抑制
* **启用** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(在 Python v3.12)") ) – 如果 `False` ，则 RNG 不会分叉。这是一个方便的参数，可以轻松禁用上下文管理器，而无需删除它并取消缩进其下的 Python 代码。
* **deivce_type** ( [*str*](https://docs.python.org/3/library) /stdtypes.html#str "(in Python v3.12)") ) – 设备类型 str，默认为 cuda 。自定义设备详见【注：支持privateuse1自定义设备】


 Return type


[*生成器*](https://docs.python.org/3/library/typing.html#typing.Generator“(在Python v3.12中)”)


 火炬.随机。


 获取_rng_状态


 ( ) [[source]](_modules/torch/random.html#get_rng_state)[¶](#torch.random.get_rng_state "此定义的永久链接")


 以 torch.ByteTensor 形式返回随机数生成器状态。


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 火炬.随机。


 初始_种子


 ( ) [[source]](_modules/torch/random.html#initial_seed)[¶](#torch.random.initial_seed "此定义的永久链接")


 返回用于生成随机数的初始种子作为 aPython long 。


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 火炬.随机。


 手册_种子


 ( *seed
* ) [[source]](_modules/torch/random.html#manual_seed)[¶](#torch.random.manual_seed "此定义的永久链接")


 设置用于生成随机数的种子。返回一个 torch.Generator 对象。


 Parameters


**seed** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 所需的种子。值必须在 [-0x8000_0000_0000_0000, 0xffff_ffff_ffff_ffff] 范围内。否则，将引发运行时错误。使用公式 0xffff_ffff_ffff_ffff 
+ seed 将负输入重新映射为正值。


 Return type


[*生成器*](generated/torch.Generator.html#torch.Generator "torch._C.Generator")


 火炬.随机。



 seed
 


 ( ) [[source]](_modules/torch/random.html#seed)[¶](#torch.random.seed "此定义的永久链接")


 将生成随机数的种子设置为非确定性随机数。返回用于作为 RNG 种子的 64 位数字。


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 火炬.随机。


 设置_rng_状态


 ( *new_state
* ) [[source]](_modules/torch/random.html#set_rng_state)[¶](#torch.random.set_rng_state "此定义的永久链接")


 设置随机数生成器状态。


 Parameters


**new_state** ( *torch.ByteTensor
* ) – 所需的状态