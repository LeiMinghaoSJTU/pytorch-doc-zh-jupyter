# torch.Storage [¶](#torch-storage "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/storage>
>
> 原始地址：<https://pytorch.org/docs/stable/storage.html>


`torch.Storage` 是与默认数据类型对应的存储类的别名 ( [`torch.get_default_dtype()`](generated/torch.get_default_dtype.html#torch.get_default_dtype "torch.get_default_dtype") )。例如，如果默认数据类型为 `torch.float` ，则 `torch.Storage` 解析为 [`torch.FloatStorage`](#torch.FloatStorage "torch.FloatStorage") 。


 `torch.<type>Storage` 和 `torch.cuda.<type>Storage` 类，如 [`torch.FloatStorage`](#torch.FloatStorage "torch.FloatStorage") 、 [`torch.IntStorage`]( #torch.IntStorage "torch.IntStorage") 等实际上并未实例化。调用它们的构造函数会创建一个具有适当的 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") 和 [`torch.dtype`](#torch.TypedStorage "torch.TypedStorage").device`](tensor_attributes.html#torch.device "torch.device"). `torch.<type>Storage` 类具有与 [`torch.TypedStorage`](#torch.TypedStorage "torch.TypedStorage") 相同的所有类方法。


 [`torch.TypedStorage`](#torch.TypedStorage "torch.TypedStorage") 是特定 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype" 的元素的连续一维数组”)。可以给它任何 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") ，并且内部数据将被适当地解释。 [`torch.TypedStorage`](#torch.TypedStorage "torch.TypedStorage") 包含一个 [`torch.UntypedStorage`](#torch.UntypedStorage "torch.UntypedStorage")，它将数据保存为无类型字节数组。


 每个跨步的 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 都包含一个 [`torch.TypedStorage`](#torch.TypedStorage "torch.TypedStorage") ，它存储所有数据[`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 视图。


!!! warning "警告"

     将来，除了 [`torch.UntypedStorage`](#torch.UntypedStorage "torch.UntypedStorage") 之外的所有存储类都将被删除，并且 [`torch.UntypedStorage`](#torch.UntypedStorage "torch.UntypedStorage") 将被删除。在所有情况下都使用。


*班级*


 火炬。


 类型化存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch/storage.html#TypedStorage)[¶](#torch.TypedStorage "此定义的永久链接")


 bfloat16


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.bfloat16)[¶](#torch.TypedStorage.bfloat16 "此定义的永久链接")


 将此存储转换为 bfloat16 类型


 bool
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.bool)[¶](#torch.TypedStorage.bool "此定义的永久链接")


 将此存储转换为 bool 类型


 byte
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.byte)[¶](#torch.TypedStorage.byte "此定义的永久链接")


 将此存储转换为字节类型


 char
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.char)[¶](#torch.TypedStorage.char "此定义的永久链接")


 将此存储转换为 char 类型


 clone
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.clone)[¶](#torch.TypedStorage.clone "此定义的永久链接")


 返回此存储的副本


 复杂_double


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.complex_double)[¶](#torch.TypedStorage.complex_double "此定义的永久链接")


 将此存储转换为复杂双精度类型


 复数_float


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.complex_float)[¶](#torch.TypedStorage.complex_float "此定义的永久链接")


 将此存储转换为复杂的浮点类型


 复制_


 (*源*，*非_阻塞



 =
 


 无
* ) [[source]](_modules/torch/storage.html#TypedStorage.copy_)[¶](#torch.TypedStorage.copy_"此定义的永久链接")




 cpu
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.cpu)[¶](#torch.TypedStorage.cpu "此定义的永久链接")


 如果该存储尚未位于 CPU 上，则返回该存储的 CPU 副本


 cuda
 


 ( *设备



 =
 


 无
* , *非_阻塞



 =
 


 错误的
* ， ***


 kwargs
* ) [[source]](_modules/torch/storage.html#TypedStorage.cuda)[¶](#torch.TypedStorage.cuda "此定义的永久链接")


 返回 CUDA 内存中该对象的副本。


 如果该对象已位于 CUDA 内存中且位于正确的设备上，则不会执行任何复制并返回原始对象。


 参数 
* **device** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 目标 GPU ID。默认为当前设备。
* **non_blocking** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果“True”且源位于固定内存中，则副本将相对于主机异步。否则，该参数无效。
* ****kwargs** – 为了兼容性，可以包含键“async”来代替“non_blocking”参数。


 Return type


*T* 


 数据_ptr


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.data_ptr)[¶](#torch.TypedStorage.data_ptr "此定义的永久链接")


*财产*


 device [¶](#torch.TypedStorage.device"此定义的永久链接")


 双倍的


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.double)[¶](#torch.TypedStorage.double "此定义的永久链接")


 将此存储转换为双精度类型


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")*[¶](#torch.TypedStorage.dtype "此定义的永久链接")


 元素_大小


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.element_size)[¶](#torch.TypedStorage.element_size "此定义的永久链接")


 充满_


 ( *value
* ) [[source]](_modules/torch/storage.html#TypedStorage.fill_)[¶](#torch.TypedStorage.fill_ "此定义的永久链接")


 float
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.float)[¶](#torch.TypedStorage.float "此定义的永久链接")


 将此存储转换为浮点类型


*类方法*


 来自_buffer


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/storage.html#TypedStorage.from_buffer)[¶](#torch.TypedStorage.from_buffer "此定义的永久链接")


*类方法*


 从文件


 ( *文件名
* , *共享



 =
 


 假*，*尺寸



 =
 



 0*

 )
 


 →
 


 贮存


[[source]](_modules/torch/storage.html#TypedStorage.from_file)[¶](#torch.TypedStorage.from_file"此定义的永久链接")


 如果共享为 True ，则内存在所有进程之间共享。所有更改都会写入文件。如果共享为 False ，则存储上的更改不会影响文件。


 size 是存储中元素的数量。如果共享为 False ，则文件必须至少包含 size * sizeof(Type) 字节( Type 是存储类型)。如果共享为 True，则将在需要时创建文件。


 参数 
* **filename** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要映射的文件名
* **shared** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否共享内存
* ** size** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 存储中的元素数量


 获取_设备


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.get_device)[¶](#torch.TypedStorage.get_device "此定义的永久链接")


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 half
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.half)[¶](#torch.TypedStorage.half "此定义的永久链接")


 将此存储转换为 half 类型


 hpu
 


 ( *设备



 =
 


 无
* , *非_阻塞



 =
 


 错误的
* ， ***


 kwargs
* ) [[source]](_modules/torch/storage.html#TypedStorage.hpu)[¶](#torch.TypedStorage.hpu "此定义的永久链接")


 返回该对象在 HPU 内存中的副本。


 如果该对象已位于 HPU 内存中且位于正确的设备上，则不会执行任何复制并返回原始对象。


 参数 
* **device** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 目标 HPU ID。默认为当前设备。
* **non_blocking** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果“True”且源位于固定内存中，则副本将相对于主机异步。否则，该参数无效。
* ****kwargs** – 为了兼容性，可以包含键“async”来代替“non_blocking”参数。


 Return type


*T* 


 int
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.int)[¶](#torch.TypedStorage.int "此定义的永久链接")


 将此存储转换为 int 类型


*财产*


 is_cuda [¶](#torch.TypedStorage.is_cuda "此定义的永久链接")


*财产*


 is_hpu [¶](#torch.TypedStorage.is_hpu "此定义的永久链接")


 已_固定


 ( *设备



 =
 


 'cuda'
* ) [[source]](_modules/torch/storage.html#TypedStorage.is_pinned)[¶](#torch.TypedStorage.is_pinned "此定义的永久链接")


 确定 CPU TypedStorage 是否已固定在设备上。


 Parameters


**device** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*或
* [*torch.device
* ](tensor_attributes.html#torch.device "torch.device") ) – 固定内存的设备。默认值：`'cuda'`


 退货


 布尔变量。


 是_共享的


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.is_shared)[¶](#torch.TypedStorage.is_shared "此定义的永久链接")


 是_稀疏 *=


 False*[¶](#torch.TypedStorage.is_sparse"此定义的永久链接")


 long
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.long)[¶](#torch.TypedStorage.long "此定义的永久链接")


 将此存储转换为长类型


 n 字节


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.nbytes)[¶](#torch.TypedStorage.nbytes "此定义的永久链接")


 pickle_storage_type


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.pickle_storage_type)[¶](#torch.TypedStorage.pickle_storage_type "此定义的永久链接")


 引脚_内存


 ( *设备



 =
 


 'cuda'
* ) [[source]](_modules/torch/storage.html#TypedStorage.pin_memory)[¶](#torch.TypedStorage.pin_memory "此定义的永久链接")


 将 CPU TypedStorage 复制到固定内存(如果尚未固定)。


 Parameters


**device** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*或
* [*torch.device
* ](tensor_attributes.html#torch.device "torch.device") ) – 固定内存的设备。默认值：“cuda”。


 退货


 固定的 CPU 存储。


 调整大小_


 ( *size
* ) [[source]](_modules/torch/storage.html#TypedStorage.resize_)[¶](#torch.TypedStorage.resize_ "此定义的永久链接")


 共享_内存_


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.share_memory_)[¶](#torch.TypedStorage.share_memory_ "此定义的永久链接")


 将存储移动到共享内存。


 对于共享内存中已经存在的存储和 CUDA 存储来说，这是一个无操作，不需要移动它们来跨进程共享。共享内存中的存储无法调整大小。


 返回：自身


 short
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.short)[¶](#torch.TypedStorage.short "此定义的永久链接")


 将此存储转换为短类型


 size
 


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.size)[¶](#torch.TypedStorage.size "此定义的永久链接")


 列出


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.tolist)[π](#torch.TypedStorage.tolist "此定义的永久链接")


 返回包含此存储的元素的列表


 type
 


 ( *d 类型



 =
 


 无
* , *非_阻塞



 =
 


 False
* ) [[source]](_modules/torch/storage.html#TypedStorage.type)[¶](#torch.TypedStorage.type "此定义的永久链接")


 如果未提供 dtype，则Return type，否则将此对象转换为指定类型。


 如果这已经是正确的类型，则不执行任何复制并返回原始对象。


 参数 
* **dtype** ( [*type*](https://docs.python.org/3/library/functions.html#type "(in Python v3.12)")*或
* *string
* ) – 所需类型
* **non_blocking** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) –如果“True”，并且源位于固定内存中且目标位于 GPU 上，反之亦然，则复制相对于主机异步执行。否则，该参数无效。
* ****kwargs** – 为了兼容性，可以包含键“async”来代替“non_blocking”参数。 `async` 参数已被弃用。


 Return type


[*Union*](https://docs.python.org/3/library/typing.html#typing.Union "(在 Python v3.12 中)") [ *T
* , [str](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)") ]


 无类型的


 ( ) [[source]](_modules/torch/storage.html#TypedStorage.untyped)[¶](#torch.TypedStorage.untyped "此定义的永久链接")


 返回内部 [`torch.UntypedStorage`](#torch.UntypedStorage "torch.UntypedStorage")


*班级*


 火炬。


 无类型存储


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/storage.html#UntypedStorage)[¶](#torch.UntypedStorage "此定义的永久链接")


 bfloat16


 ( ) [¶](#torch.UntypedStorage.bfloat16 "此定义的永久链接")


 将此存储转换为 bfloat16 类型


 bool
 


 ( ) [¶](#torch.UntypedStorage.bool "此定义的永久链接")


 将此存储转换为 bool 类型


 byte
 


 ( ) [¶](#torch.UntypedStorage.byte "此定义的永久链接")


 将此存储转换为字节类型


 字节交换


 ( *dtype
* ) [¶](#torch.UntypedStorage.byteswap "此定义的永久链接")


 交换底层数据中的字节


 char
 


 ( ) [¶](#torch.UntypedStorage.char "此定义的永久链接")


 将此存储转换为 char 类型


 clone
 


 ( ) [¶](#torch.UntypedStorage.clone "此定义的永久链接")


 返回此存储的副本


 复杂_double


 ( ) [¶](#torch.UntypedStorage.complex_double "此定义的永久链接")


 将此存储转换为复杂双精度类型


 复数_float


 ( ) [¶](#torch.UntypedStorage.complex_float "此定义的永久链接")


 将此存储转换为复杂的浮点类型


 复制_


 ( ) [¶](#torch.UntypedStorage.copy_"此定义的永久链接")


 cpu
 


 ( ) [¶](#torch.UntypedStorage.cpu "此定义的永久链接")


 如果该存储尚未位于 CPU 上，则返回该存储的 CPU 副本


 cuda
 


 ( *设备



 =
 


 无
* , *非_阻塞



 =
 


 错误的
* ， ***


 kwargs
* ) [¶](#torch.UntypedStorage.cuda "此定义的永久链接")


 返回 CUDA 内存中该对象的副本。


 如果该对象已位于 CUDA 内存中且位于正确的设备上，则不会执行任何复制并返回原始对象。


 参数 
* **device** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 目标 GPU ID。默认为当前设备。
* **non_blocking** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果“True”且源位于固定内存中，则副本将相对于主机异步。否则，该参数无效。
* ****kwargs** – 为了兼容性，可以包含键“async”来代替“non_blocking”参数。


 数据_ptr


 ( ) [¶](#torch.UntypedStorage.data_ptr "此定义的永久链接")


 设备 *：


[device](tensor_attributes.html#torch.device "torch.device")*[¶](#torch.UntypedStorage.device "此定义的永久链接")


 双倍的


 ( ) [¶](#torch.UntypedStorage.double "此定义的永久链接")


 将此存储转换为双精度类型


 元素_大小


 ( ) [¶](#torch.UntypedStorage.element_size "此定义的永久链接")


 充满_


 ( ) [¶](#torch.UntypedStorage.fill_ "此定义的永久链接")


 float
 


 ( ) [¶](#torch.UntypedStorage.float "此定义的永久链接")


 将此存储转换为浮点类型


*静止的*


 来自_buffer


 ( ) [¶](#torch.UntypedStorage.from_buffer "此定义的永久链接")


*静止的*


 从文件


 ( *文件名
* , *共享



 =
 


 假*，*尺寸



 =
 



 0*

 )
 


 →
 


 贮存


[¶](#torch.UntypedStorage.from_file "此定义的永久链接")


 如果共享为 True ，则内存在所有进程之间共享。所有更改都会写入文件。如果共享为 False ，则存储上的更改不会影响文件。


 size 是存储中元素的数量。如果共享为 False ，则文件必须至少包含 size * sizeof(Type) 字节( Type 是存储类型)。如果共享为 True，则将在需要时创建文件。


 参数 
* **filename** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要映射的文件名
* **shared** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 是否共享内存
* ** size** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 存储中的元素数量


 获取_设备


 ( ) [¶](#torch.UntypedStorage.get_device "此定义的永久链接")


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 half
 


 ( ) [¶](#torch.UntypedStorage.half "此定义的永久链接")


 将此存储转换为 half 类型


 hpu
 


 ( *设备



 =
 


 无
* , *非_阻塞



 =
 


 错误的
* ， ***


 kwargs
* ) [¶](#torch.UntypedStorage.hpu "此定义的永久链接")


 返回该对象在 HPU 内存中的副本。


 如果该对象已位于 HPU 内存中且位于正确的设备上，则不会执行任何复制并返回原始对象。


 参数 
* **device** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 目标 HPU ID。默认为当前设备。
* **non_blocking** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果“True”且源位于固定内存中，则副本将相对于主机异步。否则，该参数无效。
* ****kwargs** – 为了兼容性，可以包含键“async”来代替“non_blocking”参数。




 int
 


 ( ) [¶](#torch.UntypedStorage.int "此定义的永久链接")


 将此存储转换为 int 类型


*财产*


 is_cuda [¶](#torch.UntypedStorage.is_cuda "此定义的永久链接")


*财产*


 is_hpu [¶](#torch.UntypedStorage.is_hpu "此定义的永久链接")


 已_固定


 ( *设备



 =
 


 'cuda'
* ) [¶](#torch.UntypedStorage.is_pinned "此定义的永久链接")


 确定 CPU 存储是否已固定在设备上。


 Parameters


**device** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*或
* [*torch.device
* ](tensor_attributes.html#torch.device "torch.device") ) – 固定内存的设备。默认值：“cuda”。


 退货


 布尔变量。


 是_共享的


 ( ) [¶](#torch.UntypedStorage.is_shared "此定义的永久链接")


 是_稀疏*：


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)**=


 False*[¶](#torch.UntypedStorage.is_sparse "此定义的永久链接")


 是_sparse_csr *：


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)**=


 False*[¶](#torch.UntypedStorage.is_sparse_csr "此定义的永久链接")


 long
 


 ( ) [¶](#torch.UntypedStorage.long "此定义的永久链接")


 将此存储转换为长类型


 mps
 


 ( ) [¶](#torch.UntypedStorage.mps "此定义的永久链接")


 如果该存储尚未位于 MPS 上，则返回该存储的 MPS 副本


 n 字节


 ( ) [¶](#torch.UntypedStorage.nbytes "此定义的永久链接")


 new
 


 ( ) [¶](#torch.UntypedStorage.new "此定义的永久链接")


 引脚_内存


 ( *设备



 =
 


 'cuda'
* ) [¶](#torch.UntypedStorage.pin_memory "此定义的永久链接")


 将 CPU 存储复制到固定内存(如果尚未固定)。


 Parameters


**device** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*或
* [*torch.device
* ](tensor_attributes.html#torch.device "torch.device") ) – 固定内存的设备。默认值：“cuda”。


 退货


 固定的 CPU 存储。


 调整大小_


 ( ) [¶](#torch.UntypedStorage.resize_"此定义的永久链接")


 共享_内存_


 (
 
*\*
 


 Parameters
* , ***


 kwargs
* ) [[source]](_modules/torch/storage.html#UntypedStorage.share_memory_)[¶](#torch.UntypedStorage.share_memory_ "此定义的永久链接")


 short
 


 ( ) [¶](#torch.UntypedStorage.short "此定义的永久链接")


 将此存储转换为短类型


 size
 


 ( ) [¶](#torch.UntypedStorage.size "此定义的永久链接")


 Return type


[int](https://docs.python.org/3/library/functions.html#int“(在Python v3.12中)”)


 列出


 ( ) [π](#torch.UntypedStorage.tolist "此定义的永久链接")


 返回包含此存储的元素的列表


 type
 


 ( *d 类型



 =
 


 无
* , *非_阻塞



 =
 


 错误的
* ， ***


 kwargs
* ) [¶](#torch.UntypedStorage.type "此定义的永久链接")


 如果未提供 dtype，则Return type，否则将此对象转换为指定类型。


 如果这已经是正确的类型，则不执行任何复制并返回原始对象。


 参数 
* **dtype** ( [*type*](https://docs.python.org/3/library/functions.html#type "(in Python v3.12)")*或
* *string
* ) – 所需类型
* **non_blocking** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) –如果“True”，并且源位于固定内存中且目标位于 GPU 上，反之亦然，则复制相对于主机异步执行。否则，该参数无效。
* ****kwargs** – 为了兼容性，可以包含键“async”来代替“non_blocking”参数。 `async` 参数已被弃用。


 无类型的


 ( ) [¶](#torch.UntypedStorage.untyped "此定义的永久链接")


*班级*


 火炬。


 双存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#DoubleStorage)[¶](#torch.DoubleStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.float64*[[source]](_modules/torch.html#DoubleStorage.dtype)[¶](#torch.DoubleStorage.dtype "此定义的永久链接")


*班级*


 火炬。


 浮动存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#FloatStorage)[¶](#torch.FloatStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.float32*[[source]](_modules/torch.html#FloatStorage.dtype)[¶](#torch.FloatStorage.dtype "此定义的永久链接")


*班级*


 火炬。


 半存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#HalfStorage)[¶](#torch.HalfStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.float16*[[source]](_modules/torch.html#HalfStorage.dtype)[¶](#torch.HalfStorage.dtype "此定义的永久链接")


*班级*


 火炬。


 长存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#LongStorage)[¶](#torch.LongStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.int64*[[source]](_modules/torch.html#LongStorage.dtype)[¶](#torch.LongStorage.dtype"此定义的永久链接")


*班级*


 火炬。


 内部存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#IntStorage)[¶](#torch.IntStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.int32*[[source]](_modules/torch.html#IntStorage.dtype)[¶](#torch.IntStorage.dtype "此定义的永久链接")


*班级*


 火炬。


 短存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#ShortStorage)[¶](#torch.ShortStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.int16*[[source]](_modules/torch.html#ShortStorage.dtype)[¶](#torch.ShortStorage.dtype "此定义的永久链接")


*班级*


 火炬。


 字符存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#CharStorage)[¶](#torch.CharStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.int8*[[source]](_modules/torch.html#CharStorage.dtype)[¶](#torch.CharStorage.dtype"此定义的永久链接")


*班级*


 火炬。


 字节存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#ByteStorage)[¶](#torch.ByteStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.uint8*[[source]](_modules/torch.html#ByteStorage.dtype)[¶](#torch.ByteStorage.dtype"此定义的永久链接")


*班级*


 火炬。


 布尔存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#BoolStorage)[¶](#torch.BoolStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.bool*[[source]](_modules/torch.html#BoolStorage.dtype)[¶](#torch.BoolStorage.dtype "此定义的永久链接")


*班级*


 火炬。


 BFloat16存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#BFloat16Storage)[¶](#torch.BFloat16Storage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.bfloat16*[[source]](_modules/torch.html#BFloat16Storage.dtype)[¶](#torch.BFloat16Storage.dtype "此定义的永久链接")


*班级*


 火炬。


 复合双存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#ComplexDoubleStorage)[¶](#torch.ComplexDoubleStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.complex128*[[source]](_modules/torch.html#ComplexDoubleStorage.dtype)[¶](#torch.ComplexDoubleStorage.dtype "此定义的永久链接")


*班级*


 火炬。


 复杂浮点存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#ComplexFloatStorage)[¶](#torch.ComplexFloatStorage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.complex64*[[source]](_modules/torch.html#ComplexFloatStorage.dtype)[¶](#torch.ComplexFloatStorage.dtype "此定义的永久链接")


*班级*


 火炬。


 QUInt8存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#QUInt8Storage)[¶](#torch.QUInt8Storage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.quint8*[[source]](_modules/torch.html#QUInt8Storage.dtype)[¶](#torch.QUInt8Storage.dtype "此定义的永久链接")


*班级*


 火炬。


 QInt8存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#QInt8Storage)[¶](#torch.QInt8Storage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.qint8*[[source]](_modules/torch.html#QInt8Storage.dtype)[¶](#torch.QInt8Storage.dtype "此定义的永久链接")


*班级*


 火炬。


 QInt32存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#QInt32Storage)[¶](#torch.QInt32Storage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.qint32*[[source]](_modules/torch.html#QInt32Storage.dtype)[¶](#torch.QInt32Storage.dtype "此定义的永久链接")


*班级*


 火炬。


 QUInt4x2存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#QUInt4x2Storage)[¶](#torch.QUInt4x2Storage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.quint4x2*[[source]](_modules/torch.html#QUInt4x2Storage.dtype)[¶](#torch.QUInt4x2Storage.dtype "此定义的永久链接")


*班级*


 火炬。


 QUInt2x4存储


 (
 
*\*
 


 args
* , *wrap_storage



 =
 


 无
* , *dtype



 =
 


 无
* , *设备



 =
 


 无
* , *_内部



 =
 


 False
* ) [[source]](_modules/torch.html#QUInt2x4Storage)[¶](#torch.QUInt2x4Storage "此定义的永久链接")


 数据类型 *:


[dtype](tensor_attributes.html#torch.dtype "torch.dtype")**=


 torch.quint2x4*[[source]](_modules/torch.html#QUInt2x4Storage.dtype)[¶](#torch.QUInt2x4Storage.dtype "此定义的永久链接")