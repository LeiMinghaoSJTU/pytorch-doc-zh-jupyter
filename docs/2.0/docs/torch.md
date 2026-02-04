# torch [¶](#module-torch "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/torch>
>
> 原始地址：<https://pytorch.org/docs/stable/torch.html>


 torch 包包含多维tensor的数据结构，并定义了这些tensor的数学运算。此外，它还提供了许多用于tensor和任意类型的高效序列化的实用程序，以及其他有用的实用程序。


 它有一个 CUDA 对应项，使您能够在计算能力 >= 3.0 的 NVIDIA GPU 上运行tensor计算。


## tensor [¶](#tensors "此标题的固定链接")


|  |  |
| --- | --- |
| [`is_tensor`](generated/torch.is_tensor.html#torch.is_tensor "torch.is_tensor") |如果 obj 是 PyTorch tensor，则返回 True。 |
| [`is_storage`](generated/torch.is_storage.html#torch.is_storage "torch.is_storage") |如果 obj 是 PyTorch 存储对象，则返回 True。 |
| [`is_complex`](generated/torch.is_complex.html#torch.is_complex "torch.is_complex") |如果 `input` 的数据类型是复杂数据类型，即 `torch.complex64` 和 `torch.complex128` 之一，则返回 True。 |
| [`is_conj`](generated/torch.is_conj.html#torch.is_conj "torch.is_conj") |如果“输入”是共轭tensor，即其共轭位设置为 True ，则返回 True 。 |
| [`is_floating_point`](generated/torch.is_floating_point.html#torch.is_floating_point "torch.is_floating_point") |如果 `input` 的数据类型是浮点数据类型，即 `torch.float64` 、 `torch.float32` 、 `torch.float16` 和 `torch.bfloat16` 之一，则返回 True。 |
| [`is_nonzero`](generated/torch.is_nonzero.html#torch.is_nonzero "torch.is_nonzero") |如果“输入”是类型转换后不等于零的单元素tensor，则返回 True。 |
| [`set_default_dtype`](generated/torch.set_default_dtype.html#torch.set_default_dtype "torch.set_default_dtype") |将默认浮点数据类型设置为 `d` 。 |
| [`get_default_dtype`](generated/torch.get_default_dtype.html#torch.get_default_dtype "torch.get_default_dtype") |获取当前默认浮点 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype") 。 |
| [`set_default_device`](generated/torch.set_default_device.html#torch.set_default_device "torch.set_default_device") |设置要在“device”上分配的默认“torch.Tensor”。 |
| [`set_default_tensor_type`](generated/torch.set_default_tensor_type.html#torch.set_default_tensor_type "torch.set_default_tensor_type") |将默认的“torch.Tensor”类型设置为浮点tensor类型“t”。 |
| [`numel`](generated/torch.numel.html#torch.numel "torch.numel") |返回“输入”tensor中的元素总数。 |
| [`set_printoptions`](generated/torch.set_printoptions.html#torch.set_printoptions "torch.set_printoptions") |设置打印选项。 |
| [`set_flush_denormal`](generated/torch.set_flush_denormal.html#torch.set_flush_denormal "torch.set_flush_denormal") |禁用 CPU 上的非正规浮点数。 |


### 创建操作 [¶](#creation-ops "此标题的永久链接")




!!! note "笔记"

    随机采样创建操作列在[随机采样](#random-sampling)下，包括：[`torch.rand()`](generated/torch.rand.html#torch.rand "torch.rand")[`torch.rand()`](generated/torch.rand.html#torch.rand "torch.rand") rand_like()`](generated/torch.rand_like.html#torch.rand_like "torch.rand_like")[`torch.randn()`](generated/torch.randn.html#torch.randn "torch.randn ")[`torch.randn_like()`](generated/torch.randn_like.html#torch.randn_like "torch.randn_like")[`torch.randint()`](generated/torch.randint.html#torch.randint "torch.randint")[`torch.randint_like()`](generated/torch.randint_like.html#torch.randint_like "torch.randint_like")[`torch.randperm()`](generated/torch.randperm.html#torch.randperm "torch.randperm") 您还可以将 [`torch.empty()`](generated/torch.empty.html#torch.empty "torch.empty") 与 [In-放置随机采样](#inplace-random-sampling) 方法来创建具有从更广泛的分布中采样的值的 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 。


|  |  |
| --- | --- |
| [`tensor`](generated/torch.tensor.html#torch.tensor "torch.tensor") |通过复制 `data` 构造一个没有 autograd 历史的tensor(也称为“叶tensor”，请参阅 [Autograd 力学](notes/autograd.html) )。 |
| [`sparse_coo_tensor`](generated/torch.sparse_coo_tensor.html#torch.sparse_coo_tensor "torch.sparse_coo_tensor") |在给定的 `indices` 处构造一个具有指定值的 [COO(rdinate) 格式的稀疏tensor](sparse.html#sparse-coo-docs)。 |
| [`sparse_csr_tensor`](generated/torch.sparse_csr_tensor.html#torch.sparse_csr_tensor "torch.sparse_csr_tensor") |在给定的 `crow_indices` 和 `col_indices` 处使用指定值构造 [CSR(压缩稀疏行)中的稀疏tensor](sparse.html#sparse-csr-docs)。 |
| [`sparse_csc_tensor`](generated/torch.sparse_csc_tensor.html#torch.sparse_csc_tensor "torch.sparse_csc_tensor") |在给定的 `ccol_indices` 和 `row_indices` 处使用指定值构造 [CSC(压缩稀疏列)中的稀疏tensor](sparse.html#sparse-csc-docs)。 |
| [`sparse_bsr_tensor`](generated/torch.sparse_bsr_tensor.html#torch.sparse_bsr_tensor "torch.sparse_bsr_tensor") |在给定的 `crow_indices` 和 `col_indices` 处使用指定的二维块​​构造 [BSR(块压缩稀疏行)中的稀疏tensor](sparse.html#sparse-bsr-docs)。 |
| [`sparse_bsc_tensor`](generated/torch.sparse_bsc_tensor.html#torch.sparse_bsc_tensor "torch.sparse_bsc_tensor") |在给定的 `ccol_indices` 和 `row_indices` 处使用指定的二维块​​构造 [BSC(块压缩稀疏列)中的稀疏tensor](sparse.html#sparse-bsc-docs)。 |
| [`asarray`](generated/torch.asarray.html#torch.asarray "torch.asarray") |将 `obj` 转换为tensor。 |
| [`as_tensor`](generated/torch.as_tensor.html#torch.as_tensor "torch.as_tensor") |将“数据”转换为tensor，共享数据并在可能的情况下保留自动分级历史记录。 |
| [`as_strided`](generated/torch.as_strided.html#torch.as_strided "torch.as_strided") |使用指定的 `size` 、 `stride` 和 `storage_offset` 创建现有 torch.Tensor `input` 的视图。 |
| [`from_numpy`](generated/torch.from_numpy.html#torch.from_numpy "torch.from_numpy") |从 [`numpy.ndarray`](https://numpy.org/doc/stable/reference/generated/numpy.ndarray) 创建一个 [`Tensor`](tensors.html#torch.Tensor "torch.Tensor").html#numpy.ndarray“(在 NumPy v1.26 中)”)。 |
| [`from_dlpack`](generated/torch.from_dlpack.html#torch.from_dlpack "torch.from_dlpack") |将外部库中的tensor转换为“torch.Tensor”。 |
| [`frombuffer`](generated/torch.frombuffer.html#torch.frombuffer "torch.frombuffer") |从实现 Python 缓冲区协议的对象创建一维 [`Tensor`](tensors.html#torch.Tensor "torch.Tensor")。 |
| [`zeros`](generated/torch.zeros.html#torch.zeros "torch.zeros") |返回一个用标量值 0 填充的tensor，其形状由变量参数 `size` 定义。 |
| [`zeros_like`](generated/torch.zeros_like.html#torch.zeros_like "torch.zeros_like") |返回一个用标量值 0 填充的tensor，其大小与 `input` 相同。 |
| [`ones`](generated/torch.ones.html#torch.ones "torch.ones") |返回一个用标量值 1 填充的tensor，其形状由变量参数 `size` 定义。 |
| [`ones_like`](generated/torch.ones_like.html#torch.ones_like "torch.ones_like") |返回一个用标量值 1 填充的tensor，其大小与 `input` 相同。 |
| [`arange`](generated/torch.arange.html#torch.arange "torch.arange") |返回大小为的一维tensor



 ⌈
 


 结束 
- 开始


 step
 


 ⌉
 


 \left\lceil rac{	ext{end} 
- 	ext{start}}{	ext{step}} 
ight
ceil




 ⌈
 



 step
 


 end
 


 −
 


 start
 



 ​
 


 ⌉
 


 区间“[start, end)”中的值与从 start 开始的公差“step”一起获取。 |
| [`范围`](generated/torch.range.html#torch.range "torch.range") |返回大小为的一维tensor




 ⌊
 


 结束 
- 开始


 step
 


 ⌋
 


 +
 

 1
 


 \left\lfloor rac{	ext{end} 
- 	ext{start}}{	ext{step}} 
ight
floor 
+ 1




 ⌊
 



 step
 


 end
 


 −
 


 start
 



 ​
 


 ⌋
 


 +
 




 1
 


 值从“start”到“end”，步骤为“step”。 |
| [`linspace`](generated/torch.linspace.html#torch.linspace "torch.linspace") |创建一个大小为“steps”的一维tensor，其值从“start”到“end”(含)均匀分布。 |
| [`logspace`](generated/torch.logspace.html#torch.logspace "torch.logspace") |创建一个大小为“steps”的一维tensor，其值与


 基地开始


 {{	ext{{基础}}}}^{{	ext{{开始}}}}




 base
 


 start
 



 to
 


 基端


 {{	ext{{base}}}}^{{	ext{{end}}}}




 base
 


 end
 


 ，包含在内，以对数刻度为基数 `base` 。 |
| [`eye`](generated/torch.eye.html#torch.eye "torch.eye") |返回一个二维tensor，对角线上为 1，其他位置为 0。 |
| [`空`](generated/torch.empty.html#torch.empty "torch.empty") |返回充满未初始化数据的tensor。 |
| [`empty_like`](generated/torch.empty_like.html#torch.empty_like "torch.empty_like") |返回一个与 input 大小相同的未初始化tensor。 |
| [`empty_strided`](generated/torch.empty_strided.html#torch.empty_strided "torch.empty_strided") |创建具有指定“大小”和“步幅”的tensor，并填充未定义的数据。 |
| [`完整`](generated/torch.full.html#torch.full "torch.full") |创建一个大小为“size”的tensor，并用“fill_value”填充。 |
| [`full_like`](generated/torch.full_like.html#torch.full_like "torch.full_like") |返回一个与“input”大小相同的tensor，并填充“fill_value”。 |
| [`quantize_per_tensor`](generated/torch.quantize_per_tensor.html#torch.quantize_per_tensor "torch.quantize_per_tensor") |将浮点tensor转换为具有给定尺度和零点的量化tensor。 |
| [`quantize_per_channel`](generated/torch.quantize_per_channel.html#torch.quantize_per_channel "torch.quantize_per_channel") |将浮点tensor转换为具有给定尺度和零点的每通道量化tensor。 |
| [`dequantize`](generated/torch.dequantize.html#torch.dequantize "torch.dequantize") |通过反量化量化tensor |
| 返回 fp32 tensor


[`complex`](generated/torch.complex.html#torch.complex "torch.complex") |构造一个复tensor，其实部等于 [`real`](generated/torch.real.html#torch.real "torch.real") ，其虚部等于 [`imag`](generated/torch.imag.html#torch.imag“火炬.imag”)。 |
| [`极性`](generated/torch.polar.html#torch.polar“火炬.极性”)|构造一个复数tensor，其元素是笛卡尔坐标，对应于绝对值为 [`abs`](generated/torch.abs.html#torch.abs "torch.abs") 和角度 [`angle`](generated/torch.angle.html#torch.angle "torch.angle") 。 |
| [`heaviside`](generated/torch.heaviside.html#torch.heaviside "torch.heaviside") |计算 `input` 中每个元素的 Heaviside 阶跃函数。 |


### 索引、切片、连接、变异操作 [¶](#indexing-slicing-joining-mutating-ops "永久链接到此标题")


|  |  |
| --- | --- |
| [`伴随`](generated/torch.adjoint.html#torch.adjoint "torch.adjoint") |返回共轭tensor的视图，并转置最后两个维度。 |
| [`argwhere`](generated/torch.argwhere.html#torch.argwhere "torch.argwhere") |返回一个tensor，其中包含 `input` 的所有非零元素的索引。 |
| [`cat`](generated/torch.cat.html#torch.cat "torch.cat") |连接给定维度中给定的“seq”tensor序列。 |
| [`concat`](generated/torch.concat.html#torch.concat "torch.concat") | [`torch.cat()`](generated/torch.cat.html#torch.cat "torch.cat") 的别名。 |
| [`concatenate`](generated/torch.concatenate.html#torch.concatenate "torch.concatenate") | [`torch.cat()`](generated/torch.cat.html#torch.cat "torch.cat") 的别名。 |
| [`conj`](generated/torch.conj.html#torch.conj "torch.conj") |返回具有翻转共轭位的“输入”视图。 |
| [`chunk`](generated/torch.chunk.html#torch.chunk "torch.chunk") |尝试将tensor拆分为指定数量的块。 |
| [`dsplit`](generated/torch.dsplit.html#torch.dsplit "torch.dsplit") |根据“indices_or_sections”将具有三个或更多维度的tensor“input”按深度拆分为多个tensor。 |
| [`column_stack`](generated/torch.column_stack.html#torch.column_stack "torch.column_stack") |通过水平堆叠 `tensors` 中的tensor来创建一个新的tensor。 |
| [`dstack`](generated/torch.dstack.html#torch.dstack "torch.dstack") |按深度顺序(沿第三轴)堆叠tensor。 |
| [`聚集`](generated/torch.gather.html#torch.gather "torch.gather") |沿着 dim 指定的轴收集值。 |
| [`hsplit`](generated/torch.hsplit.html#torch.hsplit "torch.hsplit") |根据 `indices_or_sections` 将具有一维或多维的tensor `input` 水平拆分为多个tensor。 |
| [`hstack`](generated/torch.hstack.html#torch.hstack "torch.hstack") |按水平顺序堆叠tensor(按列)。 |
| [`index_add`](generated/torch.index_add.html#torch.index_add "torch.index_add") |函数说明参见 [`index_add_()`](generated/torch.Tensor.index_add_.html#torch.Tensor.index_add_ "torch.Tensor.index_add_") 。 |
| [`index_copy`](generated/torch.index_copy.html#torch.index_copy "torch.index_copy") |函数说明参见 [`index_add_()`](generated/torch.Tensor.index_add_.html#torch.Tensor.index_add_ "torch.Tensor.index_add_") 。 |
| [`index_reduce`](generated/torch.index_reduce.html#torch.index_reduce "torch.index_reduce") |函数说明参见 [`index_reduce_()`](generated/torch.Tensor.index_reduce_.html#torch.Tensor.index_reduce_ "torch.Tensor.index_reduce_") 。 |
| [`index_select`](generated/torch.index_select.html#torch.index_select "torch.index_select") |返回一个新的tensor，它使用“index”中的条目沿维度“dim”索引“input”tensor，该索引是一个 LongTensor 。 |
| [`masked_select`](generated/torch.masked_select.html#torch.masked_select "torch.masked_select") |返回一个新的一维tensor，该tensor根据布尔掩码“mask”(即 BoolTensor)索引“输入”tensor。 |
| [`movedim`](generated/torch.movedim.html#torch.movedim "torch.movedim") |将 `source` 位置处的 `input` 维度移动到 `destination` 位置。 |
| [`moveaxis`](generated/torch.moveaxis.html#torch.moveaxis "torch.moveaxis") | [`torch.movedim()`](generated/torch.movedim.html#torch.movedim "torch.movedim") 的别名。 |
| [`narrow`](generated/torch.narrow.html#torch.narrow "torch.narrow") |返回一个新tensor，它是“输入”tensor的缩小版本。 |
| [`narrow_copy`](generated/torch.narrow_copy.html#torch.narrow_copy "torch.narrow_copy") |与 [`Tensor.narrow()`](generated/torch.Tensor.narrow.html#torch.Tensor.narrow "torch.Tensor.narrow") 相同，只是返回一个副本而不是共享存储。 |
| [`nonzero`](generated/torch.nonzero.html#torch.nonzero "torch.nonzero") | |
| [`permute`](generated/torch.permute.html#torch.permute "torch.permute") |返回原始tensor“输入”的视图，其维度已排列。 |
| [`reshape`](generated/torch.reshape.html#torch.reshape "torch.reshape") |返回一个tensor，其数据和元素数量与“input”相同，但具有指定的形状。 |
| [`row_stack`](generated/torch.row_stack.html#torch.row_stack "torch.row_stack") | [`torch.vstack()`](generated/torch.vstack.html#torch.vstack "torch.vstack") 的别名。 |
| [`选择`](generated/torch.select.html#torch.select "torch.select") |沿给定索引处的选定维度对“输入”tensor进行切片。 |
| [`scatter`](generated/torch.scatter.html#torch.scatter "torch.scatter") | [`torch.Tensor.scatter_()`](generated/torch.Tensor.scatter_.html#torch.Tensor.scatter_ "torch.Tensor.scatter_") |
| 的异地版本


[`对角线_scatter`](generated/torch.diagonal_scatter.html#torch.diagonal_scatter "torch.diagonal_scatter") |相对于“dim1”和“dim2”，沿着“input”的对角线元素将“src”tensor的值嵌入到“input”中。 |
| [`select_scatter`](generated/torch.select_scatter.html#torch.select_scatter "torch.select_scatter") |将“src”tensor的值嵌入到给定索引处的“input”中。 |
| [`slice_scatter`](generated/torch.slice_scatter.html#torch.slice_scatter "torch.slice_scatter") |将“src”tensor的值嵌入到给定维度的“input”中。 |
| [`scatter_add`](generated/torch.scatter_add.html#torch.scatter_add "torch.scatter_add") | [`torch.Tensor.scatter_add_()`](generated/torch.Tensor.scatter_add_.html#torch.Tensor.scatter_add_ "torch.Tensor.scatter_add_") 的异地版本 |
| [`scatter_reduce`](generated/torch.scatter_reduce.html#torch.scatter_reduce "torch.scatter_reduce") | [`torch.Tensor.scatter_reduce_()`](generated/torch.Tensor.scatter_reduce_.html#torch.Tensor.scatter_reduce_ "torch.Tensor.scatter_reduce_") 的异地版本 |
| [`split`](generated/torch.split.html#torch.split "torch.split") |将tensor分割成块。 |
| [`挤压`](generated/torch.squeeze.html#torch.squeeze "torch.squeeze") |返回一个tensor，其中删除了大小为 1 的“输入”的所有指定维度。 |
| [`stack`](generated/torch.stack.html#torch.stack "torch.stack") |沿新维度连接一系列tensor。 |
| [`swapaxes`](generated/torch.swapaxes.html#torch.swapaxes "torch.swapaxes") | [`torch.transpose()`](generated/torch.transpose.html#torch.transpose "torch.transpose") 的别名。 |
| [`swapdims`](generated/torch.swapdims.html#torch.swapdims "torch.swapdims") | [`torch.transpose()`](generated/torch.transpose.html#torch.transpose "torch.transpose") 的别名。 |
| [`t`](generated/torch.t.html#torch.t "torch.t") |期望“输入”<= 2-D tensor并转置维度 0 和 1。 |
| [`take`](generated/torch.take.html#torch.take "torch.take") |返回一个新tensor，其中包含给定索引处的“input”元素。 |
| [`take_along_dim`](generated/torch.take_along_dim.html#torch.take_along_dim "torch.take_along_dim") |从沿着给定的“dim”的“indices”的一维索引处的“input”中选择值。 |
| [`tensor_split`](generated/torch.tensor_split.html#torch.tensor_split "torch.tensor_split") |根据 `indices_or_sections` 指定的索引或节数，将tensor拆分为多个子tensor，所有子tensor都是 `input` 的视图，沿着维度 `dim` 。 |
| [`tile`](generated/torch.tile.html#torch.tile "torch.tile") |通过重复 `input` 的元素构造一个tensor。 |
| [`transpose`](generated/torch.transpose.html#torch.transpose "torch.transpose") |返回一个tensor，它是 `input` 的转置版本。 |
| [`unbind`](generated/torch.unbind.html#torch.unbind "torch.unbind") |删除tensor维度。 |
| [`unsqueeze`](generated/torch.unsqueeze.html#torch.unsqueeze "torch.unsqueeze") |返回一个新的tensor，其尺寸为 1 插入到指定位置。 |
| [`vsplit`](generated/torch.vsplit.html#torch.vsplit "torch.vsplit") |根据 `indices_or_sections` 将具有二维或更多维度的tensor `input` 垂直拆分为多个tensor。 |
| [`vstack`](generated/torch.vstack.html#torch.vstack "torch.vstack") |按顺序垂直(行)堆叠tensor。 |
| [`where`](generated/torch.where.html#torch.where "torch.where") |返回从 `input` 或 `other` 中选择的元素tensor，具体取决于 `condition` 。 |


## 生成器 [¶](#generators "此标题的永久链接")


|  |  |
| --- | --- |
| [`生成器`](generated/torch.Generator.html#torch.Generator "torch.Generator") |创建并返回一个生成器对象，该对象管理生成伪随机数的算法的状态。 |


## 随机采样 [¶](#random-sampling "此标题的永久链接")


|  |  |
| --- | --- |
| [`seed`](generated/torch.seed.html#torch.seed "torch.seed") |将生成随机数的种子设置为非确定性随机数。 |
| [`手册_seed`](generated/torch.manual_seed.html#torch.manual_seed "torch.manual_seed") |设置用于生成随机数的种子。 |
| [`初始_seed`](generated/torch.initial_seed.html#torch.initial_seed "torch.initial_seed") |返回用于生成随机数的初始种子作为 Python long 。 |
| [`get_rng_state`](generated/torch.get_rng_state.html#torch.get_rng_state "torch.get_rng_state") |以 torch.ByteTensor 形式返回随机数生成器状态。 |
| [`set_rng_state`](generated/torch.set_rng_state.html#torch.set_rng_state "torch.set_rng_state") |设置随机数生成器状态。 |


 火炬。


 default_generator *返回默认CPU torch.Generator*[¶](#torch.torch.default_generator "永久链接到此定义")


|  |  |
| --- | --- |
| [`伯努利`](generated/torch.bernoulli.html#torch.bernoulli "torch.bernoulli") |从伯努利分布中提取二进制随机数(0 或 1)。 |
| [`多项式`](generated/torch.multinomial.html#torch.multinomial "torch.multinomial") |返回一个tensor，其中每行包含从tensor“input”相应行中的多项概率分布采样的“num_samples”索引。 |
| [`正常`](generated/torch.normal.html#torch.normal "torch.normal") |返回从给定均值和标准差的单独正态分布中抽取的随机数tensor。 |
| [`泊松`](generated/torch.poisson.html#torch.poisson "torch.poisson") |返回与“input”大小相同的tensor，每个元素均从泊松分布中采样，速率参数由“input”中相应元素给出，即 |
| [`rand`](generated/torch.rand.html#torch.rand "torch.rand") |返回一个tensor，其中填充了区间上均匀分布的随机数


 [ 0 , 1 )


 [0, 1)


 [ 0 ,



 1
 

 )
 




 |
| 


[`rand_like`](generated/torch.rand_like.html#torch.rand_like "torch.rand_like") |返回一个与“输入”大小相同的tensor，该tensor由区间上均匀分布的随机数填充


 [ 0 , 1 )


 [0, 1)


 [ 0 ,



 1
 

 )
 




.
  |
| 


[`randint`](generated/torch.randint.html#torch.randint "torch.randint") |返回一个tensor，其中填充了在“low”(包含)和“high”(不包含)之间均匀生成的随机整数。 |
| [`randint_like`](generated/torch.randint_like.html#torch.randint_like "torch.randint_like") |返回一个与tensor“input”形状相同的tensor，其中填充了在“low”(包含)和“high”(不包括)之间均匀生成的随机整数。 |
| [`randn`](generated/torch.randn.html#torch.randn "torch.randn") |返回一个tensor，该tensor填充了平均值为 0、方差为 1 的正态分布(也称为标准正态分布)的随机数。 |
| [`randn_like`](generated/torch.randn_like.html#torch.randn_like "torch.randn_like") |返回一个与“输入”大小相同的tensor，该tensor填充有均值 0 和方差 1 的正态分布中的随机数。


[`randperm`](generated/torch.randperm.html#torch.randperm "torch.randperm") |返回从 `0` 到 `n 
- 1` 的整数的随机排列。 |


### 就地随机采样 [¶](#in-place-random-sampling "此标题的固定链接")


 在tensor上还定义了一些就地随机采样函数。点击查看他们的文档：



* [`torch.Tensor.bernoulli_()`](generated/torch.Tensor.bernoulli_.html#torch.Tensor.bernoulli_ "torch.Tensor.bernoulli_") 
- [`torch.bernoulli() 的就地版本)`](generated/torch.bernoulli.html#torch.bernoulli "torch.bernoulli")
* [`torch.Tensor.cauchy_()`](generated/torch.Tensor.cauchy_.html#torch.Tensor. cauchy_ "torch.Tensor.cauchy_") 
- 从柯西分布中得出的数字
* [`torch.Tensor.exponential_()`](generated/torch.Tensor.exponential_.html#torch.Tensor.exponential_ "torch.Tensor.exponential_") 
- 从指数分布中得出的数字
* [`torch.Tensor.geometric_()`](generated/torch.Tensor.geometric_.html#torch.Tensor.geometric_ "torch.Tensor.geometric_") -从几何分布中提取的元素
* [`torch.Tensor.log_normal_()`](generated/torch.Tensor.log_normal_.html#torch.Tensor.log_normal_ "torch.Tensor.log_normal_") 
- 来自对数正态分布
* [`torch.Tensor.normal_()`](generated/torch.Tensor.normal_.html#torch.Tensor.normal_ "torch.Tensor.normal_") 
- [`的就地版本torch.normal()`](generated/torch.normal.html#torch.normal "torch.normal")
* [`torch.Tensor.random_()`](generated/torch.Tensor.random_.html# torch.Tensor.random_ "torch.Tensor.random_") 
- 从离散均匀分布中采样的数字
* [`torch.Tensor.uniform_()`](generated/torch.Tensor.uniform_.html#torch.Tensor. uniform_ "torch.Tensor.uniform_") 
- 从连续均匀分布中采样的数字


### 准随机采样 [¶](#quasi-random-sampling "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`quasirandom.SobolEngine`](generated/torch.quasirandom.SobolEngine.html#torch.quasirandom.SobolEngine "torch.quasirandom.SobolEngine")	 | 	 The	 [`torch.quasirandom.SobolEngine`](generated/torch.quasirandom.SobolEngine.html#torch.quasirandom.SobolEngine "torch.quasirandom.SobolEngine")	 is an engine for generating (scrambled) Sobol sequences.	  |


## 序列化 [¶](#serialization "此标题的永久链接")


|  |  |
| --- | --- |
| [`save`](generated/torch.save.html#torch.save "torch.save") |将对象保存到磁盘文件。 |
| [`load`](generated/torch.load.html#torch.load "torch.load") |从文件加载使用 [`torch.save()`](generated/torch.save.html#torch.save "torch.save") 保存的对象。 |


## 并行性 [¶](#parallelism "此标题的永久链接")


|  |  |
| --- | --- |
| [`get_num_threads`](generated/torch.get_num_threads.html#torch.get_num_threads "torch.get_num_threads") |返回用于并行化 CPU 操作的线程数 |
| [`set_num_threads`](generated/torch.set_num_threads.html#torch.set_num_threads "torch.set_num_threads") |设置 CPU 上用于操作内并行性的线程数。 |
| [`get_num_interop_threads`](generated/torch.get_num_interop_threads.html#torch.get_num_interop_threads "torch.get_num_interop_threads") |返回 CPU 上用于操作间并行性的线程数(例如 |
| [`set_num_interop_threads`](generated/torch.set_num_interop_threads.html#torch.set_num_interop_threads "torch.set_num_interop_threads") |设置用于互操作并行性的线程数(例如 |


## 本地禁用梯度计算 [¶](#locally-disabling-gradient-computation "永久链接到此标题")


 上下文管理器 [`torch.no_grad()`](generated/torch.no_grad.html#torch.no_grad "torch.no_grad") 、 [`torch.enable_grad()`](generated/torch.enable_grad.html#torch.enable_grad "torch.enable_grad") 和 [`torch.set_grad_enabled()`](generated/torch.set_grad_enabled.html#torch.set_grad_enabled "torch.set_grad_enabled") 对于本地禁用很有帮助并启用梯度计算。有关其用法的更多详细信息，请参阅[本地禁用梯度计算](autograd.html#locally-disable-grad)。这些上下文管理器是线程本地的，因此如果您使用“threading”模块等将工作发送到另一个线程，它们将不起作用。


 例子：


```
>>> x = torch.zeros(1, requires_grad=True)
>>> with torch.no_grad():
...     y = x * 2
>>> y.requires_grad
False

>>> is_train = False
>>> with torch.set_grad_enabled(is_train):
...     y = x * 2
>>> y.requires_grad
False

>>> torch.set_grad_enabled(True)  # this can also be used as a function
>>> y = x * 2
>>> y.requires_grad
True

>>> torch.set_grad_enabled(False)
>>> y = x * 2
>>> y.requires_grad
False

```


|  |  |
| --- | --- |
| [`no_grad`](generated/torch.no_grad.html#torch.no_grad "torch.no_grad") |禁用梯度计算的上下文管理器。 |
| [`enable_grad`](generated/torch.enable_grad.html#torch.enable_grad "torch.enable_grad") |支持梯度计算的上下文管理器。 |
| [`set_grad_enabled`](generated/torch.set_grad_enabled.html#torch.set_grad_enabled "torch.set_grad_enabled") |设置梯度计算打开或关闭的上下文管理器。 |
| [`is_grad_enabled`](generated/torch.is_grad_enabled.html#torch.is_grad_enabled "torch.is_grad_enabled") |如果当前启用了渐变模式，则返回 True。 |
| [`inference_mode`](generated/torch.inference_mode.html#torch.inference_mode "torch.inference_mode") |启用或禁用推理模式的上下文管理器 |
| [`is_inference_mode_enabled`](generated/torch.is_inference_mode_enabled.html#torch.is_inference_mode_enabled "torch.is_inference_mode_enabled") |如果当前启用了推理模式，则返回 True。 |


## 数学运算 [¶](#math-operations "此标题的永久链接")


### Pointwise Ops [¶](#pointwise-ops "此标题的永久链接")


|  |  |
| --- | --- |
| [`abs`](generated/torch.abs.html#torch.abs "torch.abs") |计算 `input` 中每个元素的绝对值。 |
| [`绝对`](generated/torch.absolute.html#torch.absolute "torch.absolute") | [`torch.abs()`](generated/torch.abs.html#torch.abs "torch.abs") |
| 的别名


[`acos`](generated/torch.acos.html#torch.acos "torch.acos") |计算 `input` 中每个元素的反余弦。 |
| [`arccos`](generated/torch.arccos.html#torch.arccos "torch.arccos") | [`torch.acos()`](generated/torch.acos.html#torch.acos "torch.acos") 的别名。 |
| [`acosh`](generated/torch.acosh.html#torch.acosh“torch.acosh”)|返回具有 `input` 元素的反双曲余弦的新tensor。 |
| [`arccosh`](generated/torch.arccosh.html#torch.arccosh“torch.arccosh”)| [`torch.acosh()`](generated/torch.acosh.html#torch.acosh "torch.acosh") 的别名。 |
| [`add`](generated/torch.add.html#torch.add "torch.add") |将按 `alpha` 缩放的 `other` 添加到 `input` 中。 |
| [`addcdiv`](generated/torch.addcdiv.html#torch.addcdiv "torch.addcdiv") |将 `tensor1` 按元素除以 `tensor2`，将结果乘以标量 `value` 并将其添加到 `input` 中。 |
| [`addcmul`](generated/torch.addcmul.html#torch.addcmul "torch.addcmul") |执行“tensor1”与“tensor2”的逐元素乘法，将结果乘以标量“value”并将其添加到“input”。 |
| [`角度`](generated/torch.angle.html#torch.angle "torch.angle") |计算给定“输入”tensor的元素角度(以弧度为单位)。 |
| [`asin`](generated/torch.asin.html#torch.asin "torch.asin") |返回一个新的tensor，其值为 `input` 元素的反正弦。 |
| [`arcsin`](generated/torch.arcsin.html#torch.arcsin "torch.arcsin") | [`torch.asin()`](generated/torch.asin.html#torch.asin "torch.asin") 的别名。 |
| [`asinh`](generated/torch.asinh.html#torch.asinh "torch.asinh") |返回一个具有 `input` 元素的反双曲正弦的新tensor。 |
| [`arcsinh`](generated/torch.arcsinh.html#torch.arcsinh "torch.arcsinh") | [`torch.asinh()`](generated/torch.asinh.html#torch.asinh "torch.asinh") 的别名。 |
| [`atan`](generated/torch.atan.html#torch.atan "torch.atan") |返回一个新的tensor，其值为 `input` 元素的反正切值。 |
| [`arctan`](generated/torch.arctan.html#torch.arctan "torch.arctan") | [`torch.atan()`](generated/torch.atan.html#torch.atan "torch.atan") 的别名。 |
| [`atanh`](generated/torch.atanh.html#torch.atanh "torch.atanh") |返回一个带有 `input` 元素的反双曲正切的新tensor。 |
| [`arctanh`](generated/torch.arctanh.html#torch.arctanh "torch.arctanh") | [`torch.atanh()`](generated/torch.atanh.html#torch.atanh "torch.atanh") 的别名。 |
| [`atan2`](generated/torch.atan2.html#torch.atan2 "torch.atan2") |元素方面的反正切


 输入我


 /
 


 其他我


 	ext{输入}_{i} /	ext{其他}_{i}




 input
 


 i
 


 ​
 


 /
 



 other
 


 i
 


 ​
 


 考虑到象限。 |
| [`arctan2`](generated/torch.arctan2.html#torch.arctan2 "torch.arctan2") | [`torch.atan2()`](generated/torch.atan2.html#torch.atan2 "torch.atan2") 的别名。 |
| [`bitwise_not`](generated/torch.bitwise_not.html#torch.bitwise_not "torch.bitwise_not") |计算给定输入tensor的按位 NOT。 |
| [`bitwise_and`](generated/torch.bitwise_and.html#torch.bitwise_and "torch.bitwise_and") |计算 `input` 和 `other` 的按位与。 |
| [`按位_or`](generated/torch.bitwise_or.html#torch.bitwise_or "torch.bitwise_or") |计算 `input` 和 `other` 的按位或。 |
| [`按位_xor`](generated/torch.bitwise_xor.html#torch.bitwise_xor "torch.bitwise_xor") |计算 `input` 和 `other` 的按位异或。 |
| [`bitwise_left_shift`](generated/torch.bitwise_left_shift.html#torch.bitwise_left_shift "torch.bitwise_left_shift") |通过“其他”位计算“输入”的左算术移位。 |
| [`bitwise_right_shift`](generated/torch.bitwise_right_shift.html#torch.bitwise_right_shift "torch.bitwise_right_shift") |通过“其他”位计算“输入”的右算术移位。 |
| [`ceil`](generated/torch.ceil.html#torch.ceil "torch.ceil") |返回一个新的tensor，其中包含 `input` 元素的 ceil，即大于或等于每个元素的最小整数。 |
| [`clamp`](generated/torch.clamp.html#torch.clamp "torch.clamp") |将 `input` 中的所有元素限制在范围 [ [`min`](generated/torch.min.html#torch.min "torch.min") , [`max`](generated/torch.max.html#torch.max "火炬.max") ]. |
| [`clip`](generated/torch.clip.html#torch.clip“torch.clip”)| [`torch.clamp()`](generated/torch.clamp.html#torch.clamp "torch.clamp") 的别名。 |
| [`conj_physical`](generated/torch.conj_physical.html#torch.conj_physical "torch.conj_physical") |计算给定“输入”tensor的逐元素共轭。 |
| [`copysign`](generated/torch.copysign.html#torch.copysign“torch.copysign”) |创建一个新的浮点tensor，其大小为“input”，符号为“other”(按元素)。 |
| [`cos`](generated/torch.cos.html#torch.cos "torch.cos") |返回一个新的tensor，其值为 `input` 元素的余弦值。 |
| [`cosh`](generated/torch.cosh.html#torch.cosh "torch.cosh") |返回具有 `input` 元素的双曲余弦的新tensor。 |
| [`deg2rad`](generated/torch.deg2rad.html#torch.deg2rad "torch.deg2rad") |返回一个新的tensor，其中“input”的每个元素都从角度(以度为单位)转换为弧度。 |
| [`div`](generated/torch.div.html#torch.div "torch.div") |将输入 `input` 的每个元素除以 `other` 的相应元素。 |
| [`divide`](generated/torch.divide.html#torch.divide "torch.divide") | [`torch.div()`](generated/torch.div.html#torch.div "torch.div") 的别名。 |
| [`digamma`](generated/torch.digamma.html#torch.digamma“torch.digamma”)| [`torch.special.digamma()`](special.html#torch.special.digamma "torch.special.digamma") 的别名。 |
| [`erf`](generated/torch.erf.html#torch.erf "torch.erf") | [`torch.special.erf()`](special.html#torch.special.erf "torch.special.erf") 的别名。 |
| [`erfc`](generated/torch.erfc.html#torch.erfc "torch.erfc") | [`torch.special.erfc()`](special.html#torch.special.erfc "torch.special.erfc") 的别名。 |
| [`erfinv`](generated/torch.erfinv.html#torch.erfinv "torch.erfinv") | [`torch.special.erfinv()`](special.html#torch.special.erfinv "torch.special.erfinv") 的别名。 |
| [`exp`](generated/torch.exp.html#torch.exp "torch.exp") |返回一个新tensor，其具有输入tensor“input”元素的指数。 |
| [`exp2`](generated/torch.exp2.html#torch.exp2 "torch.exp2") | [`torch.special.exp2()`](special.html#torch.special.exp2 "torch.special.exp2") 的别名。 |
| [`expm1`](generated/torch.expm1.html#torch.expm1 "torch.expm1") | [`torch.special.expm1()`](special.html#torch.special.expm1 "torch.special.expm1") 的别名。 |
| [`fake_quantize_per_channel_affine`](generated/torch.fake_quantize_per_channel_affine.html#torch.fake_quantize_per_channel_affine "torch.fake_quantize_per_channel_affine") |返回一个新的tensor，其中“input”中的数据在“axis”指定的通道上使用“scale”、“zero_point”、“quant_min”和“quant_max”按通道进行量化。 |
| [`fake_quantize_per_tensor_affine`](generated/torch.fake_quantize_per_tensor_affine.html#torch.fake_quantize_per_tensor_affine "torch.fake_quantize_per_tensor_affine") |返回一个新的tensor，其中“input”中的数据使用“scale”、“zero_point”、“quant_min”和“quant_max”进行量化。 |
| [`修复`](generated/torch.fix.html#torch.fix "torch.fix") | [`torch.trunc()`](generated/torch.trunc.html#torch.trunc "torch.trunc") 的别名 |
| [`float_power`](generated/torch.float_power.html#torch.float_power "torch.float_power") |以双精度按元素将“输入”提高到“指数”次方。 |
| [`floor`](generated/torch.floor.html#torch.floor "torch.floor") |返回一个新的tensor，其元素为“input”的下限，即小于或等于每个元素的最大整数。 |
| [`floor_divide`](generated/torch.floor_divide.html#torch.floor_divide "torch.floor_divide") | |
| [`fmod`](generated/torch.fmod.html#torch.fmod "torch.fmod") |按条目应用 C++ 的 [std::fmod](https://en.cppreference.com/w/cpp/numeric/math/fmod)。 |
| [`frac`](generated/torch.frac.html#torch.frac "torch.frac") |计算 `input` 中每个元素的小数部分。 |
| [`frexp`](generated/torch.frexp.html#torch.frexp "torch.frexp") |将“输入”分解为尾数和指数tensor，使得


 输入 = 尾数 ×


 2 指数


 	ext{输入} = 	ext{尾数} 	imes 2^{	ext{指数}}



 input
 




 =
 


 尾数




 ×
 


 2
 


 指数




.
  |
| 


[`渐变`](generated/torch.gradient.html#torch.gradient "torch.gradient") |估计函数的梯度



 g
 

 :
 


 R
 

 n
 


 →
 

 R
 


 g : \mathbb{R}^n 
ightarrow \mathbb{R}


 g
 



 :
 


 R
 



 n
 




 →
 




 R
 


 使用[二阶精确中心差分法]在一维或多维中(https://www.ams.org/journals/mcom/1988-51-184/S0025-5718-1988-0935077-0/S0025-5718 -1988-0935077-0.pdf)以及边界处的一阶或二阶估计。 |
| [`imag`](generated/torch.imag.html#torch.imag "torch.imag") |返回一个新tensor，其中包含“self”tensor的虚值。 |
| [`ldexp`](generated/torch.ldexp.html#torch.ldexp "torch.ldexp") |将 `input` 乘以 2 ** `other` 。 |
| [`lerp`](generated/torch.lerp.html#torch.lerp“torch.lerp”)|基于标量或tensor“权重”对两个tensor“start”(由“input”给出)和“end”进行线性插值，并返回结果“out”tensor。 |
| [`lgamma`](generated/torch.lgamma.html#torch.lgamma "torch.lgamma") |计算 `input` 上 gamma 函数绝对值的自然对数。 |
| [`log`](generated/torch.log.html#torch.log "torch.log") |返回具有 `input` 元素的自然对数的新tensor。 |
| [`log10`](generated/torch.log10.html#torch.log10 "torch.log10") |返回一个新的tensor，其对数以 `input` 的元素为底 10。 |
| [`log1p`](generated/torch.log1p.html#torch.log1p "torch.log1p") |返回自然对数为 (1 
+ `input` ) 的新tensor。 |
| [`log2`](generated/torch.log2.html#torch.log2 "torch.log2") |返回一个新的tensor，其对数以 `input` 的元素为底 2。 |
| [`logaddexp`](generated/torch.logaddexp.html#torch.logaddexp "torch.logaddexp") |输入的幂总和的对数。 |
| [`logaddexp2`](generated/torch.logaddexp2.html#torch.logaddexp2 "torch.logaddexp2") |输入以 2 为底的指数总和的对数。 |
| [`逻辑_and`](generated/torch.逻辑_and.html#torch.逻辑_and "torch.逻辑_and") |计算给定输入tensor的逐元素逻辑与。 |
| [`逻辑_not`](generated/torch.逻辑_not.html#torch.逻辑_not“火炬.逻辑_not”) |计算给定输入tensor的逐元素逻辑 NOT。 |
| [`逻辑_or`](generated/torch.逻辑_or.html#torch.逻辑_or "torch.逻辑_or") |计算给定输入tensor的逐元素逻辑或。 |
| [`逻辑_xor`](generated/torch.逻辑_xor.html#torch.逻辑_xor "torch.逻辑_xor") |计算给定输入tensor的逐元素逻辑异或。 |
| [`logit`](generated/torch.logit.html#torch.logit "torch.logit") | [`torch.special.logit()`](special.html#torch.special.logit "torch.special.logit") 的别名。 |
| [`hypot`](generated/torch.hypot.html#torch.hypot "torch.hypot") |给定直角三角形的边，返回其斜边。 |
| [`i0`](generated/torch.i0.html#torch.i0 "torch.i0") | [`torch.special.i0()`](special.html#torch.special.i0 "torch.special.i0") 的别名。 |
| [`igamma`](generated/torch.igamma.html#torch.igamma "torch.igamma") | [`torch.special.gammainc()`](special.html#torch.special.gammainc "torch.special.gammainc") 的别名。 |
| [`igammac`](generated/torch.igammac.html#torch.igammac "torch.igammac") | [`torch.special.gammaincc()`](special.html#torch.special.gammaincc "torch.special.gammaincc") 的别名。 |
| [`mul`](generated/torch.mul.html#torch.mul "torch.mul") |将 `input` 乘以 `other` 。 |
| [`multiply`](generated/torch.multiply.html#torch.multiply "torch.multiply") | [`torch.mul()`](generated/torch.mul.html#torch.mul "torch.mul") 的别名。 |
| [`mvlgamma`](generated/torch.mvlgamma.html#torch.mvlgamma“torch.mvlgamma”)| [`torch.special.multigammaln()`](special.html#torch.special.multigammaln "torch.special.multigammaln") 的别名。 |
| [`nan_to_num`](generated/torch.nan_to_num.html#torch.nan_to_num "torch.nan_to_num") |将“input”中的“NaN”、正无穷大和负无穷值分别替换为“nan”、“posinf”和“neginf”指定的值。 |
| [`neg`](generated/torch.neg.html#torch.neg "torch.neg") |返回一个新tensor，其值为 `input` 元素的负数。 |
| [`负数`](generated/torch.负数.html#torch.负数“火炬.负数”) | [`torch.neg()`](generated/torch.neg.html#torch.neg "torch.neg") |
| 的别名


[`nextafter`](generated/torch.nextafter.html#torch.nextafter "torch.nextafter") |按元素将 `input` 之后的下一个浮点值返回到 `other` 。 |
| [`polygamma`](generated/torch.polygamma.html#torch.polygamma "torch.polygamma") | [`torch.special.polygamma()`](special.html#torch.special.polygamma "torch.special.polygamma") 的别名。 |
| [`积极`](generated/torch. Positive.html#torch. Positive "torch. Positive") |返回`输入`。 |
| [`pow`](generated/torch.pow.html#torch.pow "torch.pow") |使用“exponent”获取“input”中每个元素的幂，并返回带有结果的tensor。 |
| [`量化_batch_norm`](generated/torch.quantized_batch_norm.html#torch.quantized_batch_norm "torch.quantized_batch_norm") |对 4D (NCHW) 量化tensor应用批量归一化。 |
| [`quantized_max_pool1d`](generated/torch.quantized_max_pool1d.html#torch.quantized_max_pool1d "torch.quantized_max_pool1d") |在由多个输入平面组成的输入量化tensor上应用一维最大池化。 |
| [`quantized_max_pool2d`](generated/torch.quantized_max_pool2d.html#torch.quantized_max_pool2d "torch.quantized_max_pool2d") |在由多个输入平面组成的输入量化tensor上应用 2D 最大池化。 |
| [`rad2deg`](generated/torch.rad2deg.html#torch.rad2deg "torch.rad2deg") |返回一个新的tensor，其中“input”的每个元素都从弧度角度转换为度数。 |
| [`real`](generated/torch.real.html#torch.real "torch.real") |返回一个新tensor，其中包含“self”tensor的实值。 |
| [`倒数`](generated/torch.reciprocal.html#torch.reciprocal "torch.reciprocal") |返回一个新的tensor，其值为 `input` 元素的倒数 |
| [`remainder`](generated/torch.remainder.html#torch.remainder "torch.remainder") |按条目计算 [Python 的模运算](https://docs.python.org/3/reference/expressions.html#binary-arithmetic-operations)。 |
| [`round`](generated/torch.round.html#torch.round "torch.round") |将“input”的元素舍入为最接近的整数。 |
| [`rsqrt`](generated/torch.rsqrt.html#torch.rsqrt "torch.rsqrt") |返回一个新的tensor，其值为 `input` 每个元素的平方根的倒数。 |
| [`sigmoid`](generated/torch.sigmoid.html#torch.sigmoid“torch.sigmoid”) | [`torch.special.expit()`](special.html#torch.special.expit "torch.special.expit") 的别名。 |
| [`sign`](generated/torch.sign.html#torch.sign "torch.sign") |返回一个带有 `input` 元素符号的新tensor。 |
| [`sgn`](generated/torch.sgn.html#torch.sgn "torch.sgn") |该函数是 torch.sign() 对复杂tensor的扩展。 |
| [`signbit`](generated/torch.signbit.html#torch.signbit "torch.signbit") |测试“input”的每个元素是否设置了符号位。 |
| [`sin`](generated/torch.sin.html#torch.sin "torch.sin") |返回一个新的tensor，其值为 `input` 元素的正弦值。 |
| [`sinc`](generated/torch.sinc.html#torch.sinc "torch.sinc") | [`torch.special.sinc()`](special.html#torch.special.sinc "torch.special.sinc") 的别名。 |
| [`sinh`](generated/torch.sinh.html#torch.sinh "torch.sinh") |返回具有 `input` 元素的双曲正弦的新tensor。 |
| [`softmax`](generated/torch.softmax.html#torch.softmax "torch.softmax") | [`torch.nn.function.softmax()`](generated/torch.nn.function.softmax.html#torch.nn.function.softmax "torch.nn.function.softmax") 的别名。 |
| [`sqrt`](generated/torch.sqrt.html#torch.sqrt "torch.sqrt") |返回一个新的tensor，其值为 `input` 元素的平方根。 |
| [`square`](generated/torch.square.html#torch.square "torch.square") |返回一个新的tensor，其值为 `input` 元素的平方。 |
| [`sub`](generated/torch.sub.html#torch.sub "torch.sub") |从 `input` 中减去按 `alpha` 缩放的 `other` 。 |
| [`减法`](generated/torch.subtract.html#torch.subtract "torch.subtract") | [`torch.sub()`](generated/torch.sub.html#torch.sub "torch.sub") 的别名。 |
| [`tan`](generated/torch.tan.html#torch.tan "torch.tan") |返回一个新的tensor，其值为 `input` 元素的正切值。 |
| [`tanh`](generated/torch.tanh.html#torch.tanh "torch.tanh") |返回具有 `input` 元素的双曲正切的新tensor。 |
| [`true_divide`](generated/torch.true_divide.html#torch.true_divide "torch.true_divide") | [`torch.div()`](generated/torch.div.html#torch.div "torch.div") 的别名为 `rounding_mode=None` 。 |
| [`trunc`](generated/torch.trunc.html#torch.trunc "torch.trunc") |返回一个新的tensor，其中包含 `input` 元素的截断整数值。 |
| [`xlogy`](generated/torch.xlogy.html#torch.xlogy "torch.xlogy") | [`torch.special.xlogy()`](special.html#torch.special.xlogy "torch.special.xlogy") 的别名。 |


### 缩减操作 [¶](#reduction-ops "此标题的永久链接")


|  |  |
| --- | --- |
| [`argmax`](generated/torch.argmax.html#torch.argmax "torch.argmax") |返回“输入”tensor中所有元素的最大值的索引。 |
| [`argmin`](generated/torch.argmin.html#torch.argmin "torch.argmin") |返回展平tensor或沿维度 |
| 的最小值的索引


[`amax`](generated/torch.amax.html#torch.amax "torch.amax") |返回给定维度“dim”中“输入”tensor的每个切片的最大值。 |
| [`amin`](generated/torch.amin.html#torch.amin "torch.amin") |返回给定维度“dim”中“输入”tensor的每个切片的最小值。 |
| [`aminmax`](generated/torch.aminmax.html#torch.aminmax "torch.aminmax") |计算“输入”tensor的最小值和最大值。 |
| [`全部`](generated/torch.all.html#torch.all "torch.all") |测试“input”中的所有元素的计算结果是否为 True 。 |
| [`any`](generated/torch.any.html#torch.any "torch.any") |测试“input”中的任何元素的计算结果是否为 True 。 |
| [`max`](generated/torch.max.html#torch.max "torch.max") |返回“输入”tensor中所有元素的最大值。 |
| [`min`](generated/torch.min.html#torch.min "torch.min") |返回“输入”tensor中所有元素的最小值。 |
| [`dist`](generated/torch.dist.html#torch.dist "torch.dist") |返回 ( `input` 
- `other` ) |
| 的 p 范数


[`logsumexp`](generated/torch.logsumexp.html#torch.logsumexp "torch.logsumexp") |返回给定维度“dim”中“输入”tensor每行的指数总和的对数。 |
| [`mean`](generated/torch.mean.html#torch.mean "torch.mean") |返回“输入”tensor中所有元素的平均值。 |
| [`nanmean`](generated/torch.nanmean.html#torch.nanmean "torch.nanmean") |计算沿指定维度的所有非 NaN 元素的平均值。 |
| [`中值`](generated/torch.median.html#torch.median "torch.median") |返回 `input` 中值的中位数。 |
| [`nanmedian`](generated/torch.nanmedian.html#torch.nanmedian "torch.nanmedian") |返回 `input` 中值的中位数，忽略 `NaN` 值。 |
| [`模式`](generated/torch.mode.html#torch.mode "torch.mode") |返回一个命名元组“(values,indices)”，其中“values”是给定维度“dim”中“input”tensor的每一行的众数，即该行中最常出现的值，而“indices”是找到的每个模式值的索引位置。 |
| [`norm`](generated/torch.norm.html#torch.norm "torch.norm") |返回给定tensor的矩阵范数或向量范数。 |
| [`nansum`](generated/torch.nansum.html#torch.nansum "torch.nansum") |返回所有元素的总和，将非数字 (NaN) 视为零。 |
| [`prod`](generated/torch.prod.html#torch.prod "torch.prod") |返回“输入”tensor中所有元素的乘积。 |
| [`quantile`](generated/torch.quantile.html#torch.quantile "torch.quantile") |计算“input”tensor沿维度“dim”每行的第 q 个分位数。 |
| [`nanquantile`](generated/torch.nanquantile.html#torch.nanquantile "torch.nanquantile") |这是 [`torch.quantile()`](generated/torch.quantile.html#torch.quantile "torch.quantile") 的一个变体，它“忽略”`NaN`值，计算分位数`q`，就像` `input` 中的 NaN` 值不存在。 |
| [`std`](generated/torch.std.html#torch.std "torch.std") |计算“dim”指定尺寸的标准偏差。 |
| [`std_mean`](generated/torch.std_mean.html#torch.std_mean "torch.std_mean") |计算“dim”指定尺寸的标准差和平均值。 |
| [`sum`](generated/torch.sum.html#torch.sum "torch.sum") |返回“输入”tensor中所有元素的总和。 |
| [`unique`](generated/torch.unique.html#torch.unique "torch.unique") |返回输入tensor的唯一元素。 |
| [`unique_consecutive`](generated/torch.unique_consecutive.html#torch.unique_consecutive "torch.unique_consecutive") |从每个连续的等效元素组中消除除第一个元素之外的所有元素。 |
| [`var`](generated/torch.var.html#torch.var "torch.var") |计算“dim”指定维度上的方差。 |
| [`var_mean`](generated/torch.var_mean.html#torch.var_mean "torch.var_mean") |计算“dim”指定维度上的方差和平均值。 |
| [`count_nonzero`](generated/torch.count_nonzero.html#torch.count_nonzero "torch.count_nonzero") |沿着给定的“dim”计算tensor“input”中非零值的数量。 |


### 比较操作 [¶](#comparison-ops "此标题的永久链接")


|  |  |
| --- | --- |
| [`allclose`](generated/torch.allclose.html#torch.allclose "torch.allclose") |该函数检查“input”和“other”是否满足条件： |
| [`argsort`](generated/torch.argsort.html#torch.argsort“torch.argsort”) |返回沿给定维度按值升序对tensor进行排序的索引。 |
| [`eq`](generated/torch.eq.html#torch.eq "torch.eq") |计算元素级相等 |
| [`equal`](generated/torch.equal.html#torch.equal "torch.equal") |如果两个tensor具有相同的大小和元素，则为“True”，否则为“False”。 |
| [`ge`](generated/torch.ge.html#torch.ge "torch.ge") |计算


 输入≥其他


 	ext{输入} \geq 	ext{其他}



 input
 




 ≥
 


 other
 


 元素方面。 |
| [`greater_equal`](generated/torch.greater_equal.html#torch.greater_equal "torch.greater_equal") | [`torch.ge()`](generated/torch.ge.html#torch.ge "torch.ge") 的别名。 |
| [`gt`](generated/torch.gt.html#torch.gt "torch.gt") |计算


 输入 
> 其他


 	ext{输入} 
> 	ext{其他}



 input
 




 >
 


 other
 


 元素方面。 |
| [`greater`](generated/torch.greater.html#torch.greater "torch.greater") | [`torch.gt()`](generated/torch.gt.html#torch.gt "torch.gt") 的别名。 |
| [`isclose`](generated/torch.isclose.html#torch.isclose "torch.isclose") |返回一个新的tensor，其中布尔元素表示“input”的每个元素是否“接近”“other”的相应元素。 |
| [`isfinite`](generated/torch.isfinite.html#torch.isfinite "torch.isfinite") |返回一个新tensor，其中布尔元素表示每个元素是否有限。 |
| [`isin`](generated/torch.isin.html#torch.isin "torch.isin") |测试 `elements` 的每个元素是否在 `test_elements` 中。 |
| [`isinf`](generated/torch.isinf.html#torch.isinf "torch.isinf") |测试“input”的每个元素是否是无限的(正无穷大或负无穷大)。 |
| [`isposinf`](generated/torch.isposinf.html#torch.isposinf "torch.isposinf") |测试“input”的每个元素是否为正无穷大。 |
| [`isneginf`](generated/torch.isneginf.html#torch.isneginf "torch.isneginf") |测试“input”的每个元素是否为负无穷大。 |
| [`isnan`](generated/torch.isnan.html#torch.isnan "torch.isnan") |返回一个新的tensor，其中布尔元素表示“input”的每个元素是否为 NaN。 |
| [`isreal`](generated/torch.isreal.html#torch.isreal "torch.isreal") |返回一个新的tensor，其中布尔元素表示“input”的每个元素是否为实值。 |
| [`kthvalue`](generated/torch.kthvalue.html#torch.kthvalue "torch.kthvalue") |返回一个命名元组“(values,indices)”，其中“values”是给定维度“dim”中“input”tensor每行的第“k”个最小元素。 |
| [`le`](generated/torch.le.html#torch.le "torch.le") |计算


 输入≤其他


 	ext{输入} \leq 	ext{其他}



 input
 




 ≤
 


 other
 


 元素方面。 |
| [`less_equal`](generated/torch.less_equal.html#torch.less_equal "torch.less_equal") | [`torch.le()`](generated/torch.le.html#torch.le "torch.le") 的别名。 |
| [`lt`](generated/torch.lt.html#torch.lt "torch.lt") |计算


 输入<其他


 	ext{输入} < 	ext{其他}



 input
 




 <
 


 other
 


 元素方面。 |
| [`less`](generated/torch.less.html#torch.less "torch.less") | [`torch.lt()`](generated/torch.lt.html#torch.lt "torch.lt") 的别名。 |
| [`最大值`](generated/torch.maximum.html#torch.maximum "torch.maximum") |计算 `input` 和 `other` 的逐元素最大值。 |
| [`最小值`](generated/torch.minimum.html#torch.minimum "torch.minimum") |计算 `input` 和 `other` 的元素最小值。 |
| [`fmax`](generated/torch.fmax.html#torch.fmax "torch.fmax") |计算 `input` 和 `other` 的逐元素最大值。 |
| [`fmin`](generated/torch.fmin.html#torch.fmin "torch.fmin") |计算 `input` 和 `other` 的元素最小值。 |
| [`ne`](generated/torch.ne.html#torch.ne "torch.ne") |计算


 输入≠其他


 	ext{输入} 
eq 	ext{其他}



 input
 




 
 



 =
 



 other
 


 元素方面。 |
| [`不_等于`](generated/torch.not_equal.html#torch.not_equal "torch.not_equal") | [`torch.ne()`](generated/torch.ne.html#torch.ne "torch.ne") 的别名。 |
| [`排序`](generated/torch.sort.html#torch.sort "torch.sort") |按给定维度按值升序对“输入”tensor的元素进行排序。 |
| [`topk`](generated/torch.topk.html#torch.topk "torch.topk") |返回给定“输入”tensor沿给定维度的“k”个最大元素。 |
| [`msort`](generated/torch.msort.html#torch.msort "torch.msort") |按值沿其第一个维度对“输入”tensor的元素进行升序排序。 |


### Spectral Ops [¶](#spectral-ops "此标题的永久链接")


|  |  |
| --- | --- |
| [`stft`](generated/torch.stft.html#torch.stft "torch.stft") |短时傅立叶变换 (STFT)。 |
| [`istft`](generated/torch.istft.html#torch.istft "torch.istft") |短时傅里叶逆变换。 |
| [`bartlett_window`](generated/torch.bartlett_window.html#torch.bartlett_window "torch.bartlett_window") |巴特利特窗函数。 |
| [`blackman_window`](generated/torch.blackman_window.html#torch.blackman_window "torch.blackman_window") |布莱克曼窗函数。 |
| [`hamming_window`](generated/torch.hamming_window.html#torch.hamming_window "torch.hamming_window") |汉明窗函数。 |
| [`hann_window`](generated/torch.hann_window.html#torch.hann_window "torch.hann_window") |汉恩窗函数。 |
| [`kaiser_window`](generated/torch.kaiser_window.html#torch.kaiser_window "torch.kaiser_window") |使用窗口长度 `window_length` 和形状参数 `beta` 计算 Kaiser 窗口。 |


### 其他操作 [¶](#other-operations "此标题的永久链接")


|  |  |
| --- | --- |
| [`atleast_1d`](generated/torch.atleast_1d.html#torch.atleast_1d "torch.atleast_1d") |返回每个零维度输入tensor的一维视图。 |
| [`atleast_2d`](generated/torch.atleast_2d.html#torch.atleast_2d "torch.atleast_2d") |返回每个零维度输入tensor的二维视图。 |
| [`atleast_3d`](generated/torch.atleast_3d.html#torch.atleast_3d "torch.atleast_3d") |返回每个零维度输入tensor的 3 维视图。 |
| [`bincount`](generated/torch.bincount.html#torch.bincount "torch.bincount") |计算非负整数数组中每个值的频率。 |
| [`block_diag`](generated/torch.block_diag.html#torch.block_diag "torch.block_diag") |从提供的tensor创建块对角矩阵。 |
| [`broadcast_tensors`](generated/torch.broadcast_tensors.html#torch.broadcast_tensors "torch.broadcast_tensors") |根据 [广播语义](notes/broadcasting.html#broadcasting-semantics) 广播给定的tensor。 |
| [`broadcast_to`](generated/torch.broadcast_to.html#torch.broadcast_to "torch.broadcast_to") |将“input”广播到形状“shape”。 |
| [`broadcast_shapes`](generated/torch.broadcast_shapes.html#torch.broadcast_shapes "torch.broadcast_shapes") |与 [`broadcast_tensors()`](generated/torch.broadcast_tensors.html#torch.broadcast_tensors "torch.broadcast_tensors") 类似，但适用于形状。 |
| [`bucketize`](generated/torch.bucketize.html#torch.bucketize "torch.bucketize") |返回“input”中每个值所属的桶的索引，其中桶的边界由“boundaries”设置。 |
| [`cartesian_prod`](generated/torch.cartesian_prod.html#torch.cartesian_prod "torch.cartesian_prod") |计算给定tensor序列的笛卡尔积。 |
| [`cdist`](generated/torch.cdist.html#torch.cdist "torch.cdist") |批量计算两个行向量集合的每对之间的 p 范数距离。 |
| [`克隆`](generated/torch.clone.html#torch.clone "torch.clone") |返回 `input` 的副本。 |
| [`组合`](generated/torch.combinations.html#torch.combinations "torch.combinations") |计算长度组合



 r
 


 r
 


 r
 


 给定的tensor。 |
| [`corrcoef`](generated/torch.corrcoef.html#torch.corrcoef "torch.corrcoef") |估计“输入”矩阵给出的变量的皮尔逊积矩相关系数矩阵，其中行是变量，列是观测值。 |
| [`cov`](generated/torch.cov.html#torch.cov“torch.cov”)|估计“输入”矩阵给出的变量的协方差矩阵，其中行是变量，列是观测值。 |
| [`cross`](generated/torch.cross.html#torch.cross "torch.cross") |返回“input”和“other”维度“dim”中向量的叉积。 |
| [`cummax`](generated/torch.cummax.html#torch.cummax "torch.cummax") |返回一个命名元组“(values,indices)”，其中“values”是维度“dim”中“input”元素的累积最大值。 |
| [`cummin`](generated/torch.cummin.html#torch.cummin "torch.cummin") |返回一个命名元组“(values,indices)”，其中“values”是维度“dim”中“input”元素的累积最小值。 |
| [`cumprod`](generated/torch.cumprod.html#torch.cumprod“torch.cumprod”)|返回维度“dim”中“input”元素的累积乘积。 |
| [`cumsum`](generated/torch.cumsum.html#torch.cumsum "torch.cumsum") |返回维度“dim”中“input”元素的累积和。 |
| [`diag`](generated/torch.diag.html#torch.diag "torch.diag") | 
* 如果 `input` 是一个向量(一维tensor)，则返回一个二维平方tensor |
| [`diag_embed`](generated/torch.diag_embed.html#torch.diag_embed "torch.diag_embed") |创建一个tensor，其某些 2D 平面(由 `dim1` 和 `dim2` 指定)的对角线由 `input` 填充。 |
| [`diagflat`](generated/torch.diagflat.html#torch.diagflat“torch.diagflat”) | 
* 如果 `input` 是一个向量(一维tensor)，则返回一个二维平方tensor |
| [`对角线`](generated/torch.diagonal.html#torch.diagonal "torch.diagonal") |返回“input”的部分视图，其对角线元素相对于“dim1”和“dim2”作为尺寸附加在形状的末尾。 |
| [`diff`](generated/torch.diff.html#torch.diff "torch.diff") |计算沿给定维度的第 n 个前向差异。 |
| [`einsum`](generated/torch.einsum.html#torch.einsum "torch.einsum") |使用基于爱因斯坦求和约定的符号沿指定的维度对输入“操作数”的元素的乘积求和。 |
| [`展平`](generated/torch.展平.html#torch.展平“火炬.展平”) |通过将“输入”重塑为一维tensor来展平“输入”。 |
| [`flip`](generated/torch.flip.html#torch.flip "torch.flip") |沿给定轴反转 n 维tensor的顺序(以暗度为单位)。 |
| [`fliplr`](generated/torch.fliplr.html#torch.fliplr "torch.fliplr") |向左/右方向翻转tensor，返回一个新的tensor。 |
| [`flipud`](generated/torch.flipud.html#torch.flipud "torch.flipud") |向上/向下翻转tensor，返回一个新的tensor。 |
| [`kron`](generated/torch.kron.html#torch.kron "torch.kron") |计算克罗内克积，表示为



 ⊗
 


 有时


 ⊗
 


 ，“输入”和“其他”。 |
| [`rot90`](generated/torch.rot90.html#torch.rot90“torch.rot90”)|在 dims 轴指定的平面中将 n 维tensor旋转 90 度。 |
| [`gcd`](generated/torch.gcd.html#torch.gcd "torch.gcd") |计算 `input` 和 `other` 的按元素最大公约数 (GCD)。 |
| [`histc`](generated/torch.histc.html#torch.histc "torch.histc") |计算tensor的直方图。 |
| [`直方图`](generated/torch.histogram.html#torch.histogram "torch.histogram") |计算tensor中值的直方图。 |
| [`histogramdd`](generated/torch.histogramdd.html#torch.histogramdd "torch.histogramdd") |计算tensor中值的多维直方图。 |
| [`meshgrid`](generated/torch.meshgrid.html#torch.meshgrid "torch.meshgrid") |创建由 attr :tensors 中的 1D 输入指定的坐标网格。 |
| [`lcm`](generated/torch.lcm.html#torch.lcm "torch.lcm") |计算 `input` 和 `other` 的按元素最小公倍数 (LCM)。 |
| [`logcumsumexp`](generated/torch.logcumsumexp.html#torch.logcumsumexp "torch.logcumsumexp") |返回维度“dim”中“input”元素求幂的累积和的对数。 |
| [`ravel`](generated/torch.ravel.html#torch.ravel "torch.ravel") |返回连续的展平tensor。 |
| [`renorm`](generated/torch.renorm.html#torch.renorm "torch.renorm") |返回一个tensor，其中“input”沿维度“dim”的每个子tensor都经过归一化，使得子tensor的 p 范数低于值“maxnorm” |
| [`repeat_interleave`](generated/torch.repeat_interleave.html#torch.repeat_interleave "torch.repeat_interleave") |重复tensor的元素。 |
| [`roll`](generated/torch.roll.html#torch.roll "torch.roll") |沿给定维度滚动tensor“输入”。 |
| [`searchsorted`](generated/torch.searchsorted.html#torch.searchsorted "torch.searchsorted") |从 `sorted_sequence` 的 *innermost
* 维度中查找索引，这样，如果 `values` 中的相应值插入到索引之前，则排序时，在 `sorted_sequence` 中对应的 *innermost
* 维度的顺序将被保留。 |
| [`tensordot`](generated/torch.tensordot.html#torch.tensordot "torch.tensordot") |返回 a 和 b 在多个维度上的收缩。 |
| [`trace`](generated/torch.trace.html#torch.trace "torch.trace") |返回输入二维矩阵对角线元素的总和。 |
| [`tril`](generated/torch.tril.html#torch.tril "torch.tril") |返回矩阵(二维tensor)或批量矩阵“input”的下三角部分，结果tensor“out”的其他元素设置为 0。 |
| [`tril_indices`](generated/torch.tril_indices.html#torch.tril_indices "torch.tril_indices") |返回 2×N tensor中“row”×“col”矩阵的下三角部分的索引，其中第一行包含所有索引的行坐标，第二行包含列坐标。 |
| [`triu`](generated/torch.triu.html#torch.triu "torch.triu") |返回矩阵(二维tensor)或批量矩阵“input”的上三角部分，结果tensor“out”的其他元素设置为 0。 |
| [`triu_indices`](generated/torch.triu_indices.html#torch.triu_indices "torch.triu_indices") |返回 2×N tensor中“row”×“col”矩阵的上三角部分的索引，其中第一行包含所有索引的行坐标，第二行包含列坐标。 |
| [`unflatten`](generated/torch.unflatten.html#torch.unflatten "torch.unflatten") |将输入tensor的维度扩展到多个维度。 |
| [`vander`](generated/torch.vander.html#torch.vander "torch.vander") |生成范德蒙矩阵。 |
| [`view_as_real`](generated/torch.view_as_real.html#torch.view_as_real "torch.view_as_real") |返回“输入”作为实tensor的视图。 |
| [`view_as_complex`](generated/torch.view_as_complex.html#torch.view_as_complex "torch.view_as_complex") |返回“输入”作为复tensor的视图。 |
| [`resolve_conj`](generated/torch.resolve_conj.html#torch.resolve_conj "torch.resolve_conj") |如果“input”的共轭位设置为 True，则返回具有物化共轭的新tensor，否则返回“input”。 |
| [`resolve_neg`](generated/torch.resolve_neg.html#torch.resolve_neg "torch.resolve_neg") |如果“input”的负位设置为 True，则返回具有具体化否定的新tensor，否则返回“input”。 |


### BLAS 和 LAPACK 操作 [¶](#blas-and-lapack-operations "永久链接到此标题")


|  |  |
| --- | --- |
| [`addbmm`](generated/torch.addbmm.html#torch.addbmm "torch.addbmm") |对存储在“batch1”和“batch2”中的矩阵执行批量矩阵-矩阵乘积，并减少加法步骤(所有矩阵乘法都沿第一维累积)。 |
| [`addmm`](generated/torch.addmm.html#torch.addmm "torch.addmm") |执行矩阵 `mat1` 和 `mat2` 的矩阵乘法。 |
| [`addmv`](generated/torch.addmv.html#torch.addmv "torch.addmv") |执行矩阵 `mat` 和向量 `vec` 的矩阵向量乘积。 |
| [`addr`](generated/torch.addr.html#torch.addr "torch.addr") |执行向量 `vec1` 和 `vec2` 的外积并将其添加到矩阵 `input` 中。 |
| [`baddbmm`](generated/torch.baddbmm.html#torch.baddbmm "torch.baddbmm") |对 `batch1` 和 `batch2` 中的矩阵执行批量矩阵-矩阵乘积。 |
| [`bmm`](generated/torch.bmm.html#torch.bmm "torch.bmm") |对存储在 `input` 和 `mat2` 中的矩阵执行批量矩阵矩阵乘积。 |
| [`chain_matmul`](generated/torch.chain_matmul.html#torch.chain_matmul "torch.chain_matmul") |返回矩阵乘积



 N
 


 N
 


 N
 


 二维tensor。 |
| [`cholesky`](generated/torch.cholesky.html#torch.cholesky "torch.cholesky") |计算对称正定矩阵的 Cholesky 分解



 A
 


 A
 


 A
 


 或者对于批量对称正定矩阵。 |
| [`cholesky_inverse`](generated/torch.cholesky_inverse.html#torch.cholesky_inverse "torch.cholesky_inverse") |计算对称正定矩阵的逆矩阵



 A
 


 A
 


 A
 


 使用其 Cholesky 因子



 u
 


 u
 


 u
 


 ：返回矩阵“inv”。 |
| [`cholesky_solve`](generated/torch.cholesky_solve.html#torch.cholesky_solve "torch.cholesky_solve") |在给定 Cholesky 因子矩阵的情况下，求解具有要逆的正半定矩阵的线性方程组



 u
 


 u
 


 u
 




.
  |
| 


[`dot`](generated/torch.dot.html#torch.dot "torch.dot") |计算两个一维tensor的点积。 |
| [`geqrf`](generated/torch.geqrf.html#torch.geqrf“torch.geqrf”)|这是直接调用 LAPACK 的 geqrf 的低级函数。 |
| [`ger`](generated/torch.ger.html#torch.ger "torch.ger") | [`torch.outer()`](generated/torch.outer.html#torch.outer "torch.outer") 的别名。 |
| [`inner`](generated/torch.inner.html#torch.inner "torch.inner") |计算一维tensor的点积。 |
| [`inverse`](generated/torch.inverse.html#torch.inverse "torch.inverse") | [`torch.linalg.inv()`](generated/torch.linalg.inv.html#torch.linalg.inv "torch.linalg.inv") 的别名 |
| [`it`](generated/torch.det.html#torch.det "torch.det") | [`torch.linalg.det()`](generated/torch.linalg.det.html#torch.linalg.det "torch.linalg.det") |
| 的别名


[`logdet`](generated/torch.logdet.html#torch.logdet "torch.logdet") |计算方阵或方阵批次的对数行列式。 |
| [`slogdet`](generated/torch.slogdet.html#torch.slogdet "torch.slogdet") | [`torch.linalg.slogdet()`](generated/torch.linalg.slogdet.html#torch.linalg.slogdet "torch.linalg.slogdet") |
| 的别名


[`lu`](generated/torch.lu.html#torch.lu "torch.lu") |计算一个或多个矩阵“A”的 LU 分解。 |
| [`lu_solve`](generated/torch.lu_solve.html#torch.lu_solve "torch.lu_solve") |返回线性系统的 LU 解


 A x = b


 轴=b


 A
 

 x
 



 =
 




 b
 


 使用 [`lu_factor()`](generated/torch.linalg.lu_factor.html#torch.linalg.lu_factor "torch.linalg.lu_factor") 对 A 进行部分枢轴 LU 分解。 |
| [`lu_unpack`](generated/torch.lu_unpack.html#torch.lu_unpack "torch.lu_unpack") |将 [`lu_factor()`](generated/torch.linalg.lu_factor.html#torch.linalg.lu_factor "torch.linalg.lu_factor") 返回的 LU 分解解包为 P、L、U 矩阵。 |
| [`matmul`](generated/torch.matmul.html#torch.matmul "torch.matmul") |两个tensor的矩阵乘积。 |
| [`matrix_power`](generated/torch.matrix_power.html#torch.matrix_power "torch.matrix_power") | [`torch.linalg.matrix_power()`](generated/torch.linalg.matrix_power.html#torch.linalg.matrix_power "torch.linalg.matrix_power") |
| 的别名


[`matrix_exp`](generated/torch.matrix_exp.html#torch.matrix_exp "torch.matrix_exp") | [`torch.linalg.matrix_exp()`](generated/torch.linalg.matrix_exp.html#torch.linalg.matrix_exp "torch.linalg.matrix_exp") 的别名。 |
| [`mm`](generated/torch.mm.html#torch.mm "torch.mm") |执行矩阵 `input` 和 `mat2` 的矩阵乘法。 |
| [`mv`](generated/torch.mv.html#torch.mv "torch.mv") |执行矩阵“input”和向量“vec”的矩阵向量乘积。 |
| [`orgqr`](generated/torch.orgqr.html#torch.orgqr "torch.orgqr") | [`torch.linalg.householder_product()`](generated/torch.linalg.householder_product.html#torch.linalg.householder_product "torch.linalg.householder_product") 的别名。 |
| [`ormqr`](generated/torch.ormqr.html#torch.ormqr "torch.ormqr") |计算 Householder 矩阵与一般矩阵的乘积的矩阵-矩阵乘法。 |
| [`outer`](generated/torch.outer.html#torch.outer "torch.outer") | `input` 和 `vec2` 的外积。 |
| [`pinverse`](generated/torch.pinverse.html#torch.pinverse "torch.pinverse") | [`torch.linalg.pinv()`](generated/torch.linalg.pinv.html#torch.linalg.pinv "torch.linalg.pinv") |
| 的别名


[`qr`](generated/torch.qr.html#torch.qr "torch.qr") |计算一个矩阵或一批矩阵 `input` 的 QR 分解，并返回tensor的命名元组 (Q, R)，使得


 输入 = QR


 	ext{输入} = Q R



 input
 




 =
 




 QR
 




 with
 



 Q
 


 Q
 


 Q
 


 是一个正交矩阵或一批正交矩阵，并且



 R
 


 R
 


 R
 


 是一个上三角矩阵或一批上三角矩阵。 |
| [`svd`](generated/torch.svd.html#torch.svd "torch.svd") |计算一个矩阵或一批矩阵“输入”的奇异值分解。 |
| [`svd_lowrank`](generated/torch.svd_lowrank.html#torch.svd_lowrank "torch.svd_lowrank") |返回矩阵、矩阵批次或稀疏矩阵的奇异值分解“(U, S, V)”



 A
 


 A
 


 A
 


 这样


 A ≈ U d i a g ( S )


 V
 

 T
 


 A pprox U diag(S) V^T


 A
 



 ≈
 


 U d ig ( S )


 V
 



 T
 


.
  |
| 


[`pca_lowrank`](generated/torch.pca_lowrank.html#torch.pca_lowrank "torch.pca_lowrank") |对低秩矩阵、此类矩阵的批次或稀疏矩阵执行线性主成分分析 (PCA)。 |
| [`lobpcg`](generated/torch.lobpcg.html#torch.lobpcg“torch.lobpcg”)|使用无矩阵 LOBPCG 方法查找对称正定广义特征值问题的 k 个最大(或最小)特征值和相应的特征向量。 |
| [`trapz`](generated/torch.trapz.html#torch.trapz "torch.trapz") | [`torch.trapezoid()`](generated/torch.trapezoid.html#torch.trapezoid "torch.trapezoid") 的别名。 |
| [`梯形`](generated/torch.trapezoid.html#torch.trapezoid "torch.trapezoid") |沿 `dim` 计算[梯形规则](https://en.wikipedia.org/wiki/Trapezoidal_rule)。 |
| [`累积_梯形`](generated/torch.cumulative_trapezoid.html#torch.cumulative_trapezoid "torch.cumulative_trapezoid") |


 沿 `dim` 累积计算[梯形规则](https://en.wikipedia.org/wiki/Trapezoidal_rule)。 |
| [`三角_solve`](generated/torch.triangle_solve.html#torch.triangle_solve "torch.triangle_solve") |求解具有方形上三角或下三角可逆矩阵的方程组



 A
 


 A
 


 A
 


 和多个右侧



 b
 


 b
 


 b
 




.
  |
| 


[`vdot`](generated/torch.vdot.html#torch.vdot "torch.vdot") |计算两个一维向量沿某个维度的点积。 |


### Foreach 操作 [¶](#foreach-operations "此标题的永久链接")


!!! warning "警告"

     此 API 处于测试阶段，未来可能会发生变化。不支持转发模式 AD。


|  |  |
| --- | --- |
| [`_foreach_abs`](generated/torch._foreach_abs.html#torch._foreach_abs "torch._foreach_abs") |将 [`torch.abs()`](generated/torch.abs.html#torch.abs "torch.abs") 应用于输入列表的每个tensor。 |
| [`_foreach_abs_`](generated/torch._foreach_abs_.html#torch._foreach_abs_ "torch._foreach_abs_") |将 [`torch.abs()`](generated/torch.abs.html#torch.abs "torch.abs") 应用于输入列表的每个tensor。 |
| [`_foreach_acos`](generated/torch._foreach_acos.html#torch._foreach_acos "torch._foreach_acos") |将 [`torch.acos()`](generated/torch.acos.html#torch.acos "torch.acos") 应用于输入列表的每个tensor。 |
| [`_foreach_acos_`](generated/torch._foreach_acos_.html#torch._foreach_acos_ "torch._foreach_acos_") |将 [`torch.acos()`](generated/torch.acos.html#torch.acos "torch.acos") 应用于输入列表的每个tensor。 |
| [`_foreach_asin`](generated/torch._foreach_asin.html#torch._foreach_asin "torch._foreach_asin") |将 [`torch.asin()`](generated/torch.asin.html#torch.asin "torch.asin") 应用于输入列表的每个tensor。 |
| [`_foreach_asin_`](generated/torch._foreach_asin_.html#torch._foreach_asin_ "torch._foreach_asin_") |将 [`torch.asin()`](generated/torch.asin.html#torch.asin "torch.asin") 应用于输入列表的每个tensor。 |
| [`_foreach_atan`](generated/torch._foreach_atan.html#torch._foreach_atan "torch._foreach_atan") |将 [`torch.atan()`](generated/torch.atan.html#torch.atan "torch.atan") 应用于输入列表的每个tensor。 |
| [`_foreach_atan_`](generated/torch._foreach_atan_.html#torch._foreach_atan_ "torch._foreach_atan_") |将 [`torch.atan()`](generated/torch.atan.html#torch.atan "torch.atan") 应用于输入列表的每个tensor。 |
| [`_foreach_ceil`](generated/torch._foreach_ceil.html#torch._foreach_ceil "torch._foreach_ceil") |将 [`torch.ceil()`](generated/torch.ceil.html#torch.ceil "torch.ceil") 应用于输入列表的每个tensor。 |
| [`_foreach_ceil_`](generated/torch._foreach_ceil_.html#torch._foreach_ceil_ "torch._foreach_ceil_") |将 [`torch.ceil()`](generated/torch.ceil.html#torch.ceil "torch.ceil") 应用于输入列表的每个tensor。 |
| [`_foreach_cos`](generated/torch._foreach_cos.html#torch._foreach_cos "torch._foreach_cos") |将 [`torch.cos()`](generated/torch.cos.html#torch.cos "torch.cos") 应用于输入列表的每个tensor。 |
| [`_foreach_cos_`](generated/torch._foreach_cos_.html#torch._foreach_cos_ "torch._foreach_cos_") |将 [`torch.cos()`](generated/torch.cos.html#torch.cos "torch.cos") 应用于输入列表的每个tensor。 |
| [`_foreach_cosh`](generated/torch._foreach_cosh.html#torch._foreach_cosh "torch._foreach_cosh") |将 [`torch.cosh()`](generated/torch.cosh.html#torch.cosh "torch.cosh") 应用于输入列表的每个tensor。 |
| [`_foreach_cosh_`](generated/torch._foreach_cosh_.html#torch._foreach_cosh_ "torch._foreach_cosh_") |将 [`torch.cosh()`](generated/torch.cosh.html#torch.cosh "torch.cosh") 应用于输入列表的每个tensor。 |
| [`_foreach_erf`](generated/torch._foreach_erf.html#torch._foreach_erf "torch._foreach_erf") |将 [`torch.erf()`](generated/torch.erf.html#torch.erf "torch.erf") 应用于输入列表的每个tensor。 |
| [`_foreach_erf_`](generated/torch._foreach_erf_.html#torch._foreach_erf_ "torch._foreach_erf_") |将 [`torch.erf()`](generated/torch.erf.html#torch.erf "torch.erf") 应用于输入列表的每个tensor。 |
| [`_foreach_erfc`](generated/torch._foreach_erfc.html#torch._foreach_erfc "torch._foreach_erfc") |将 [`torch.erfc()`](generated/torch.erfc.html#torch.erfc "torch.erfc") 应用于输入列表的每个tensor。 |
| [`_foreach_erfc_`](generated/torch._foreach_erfc_.html#torch._foreach_erfc_ "torch._foreach_erfc_") |将 [`torch.erfc()`](generated/torch.erfc.html#torch.erfc "torch.erfc") 应用于输入列表的每个tensor。 |
| [`_foreach_exp`](generated/torch._foreach_exp.html#torch._foreach_exp "torch._foreach_exp") |将 [`torch.exp()`](generated/torch.exp.html#torch.exp "torch.exp") 应用于输入列表的每个tensor。 |
| [`_foreach_exp_`](generated/torch._foreach_exp_.html#torch._foreach_exp_ "torch._foreach_exp_") |将 [`torch.exp()`](generated/torch.exp.html#torch.exp "torch.exp") 应用于输入列表的每个tensor。 |
| [`_foreach_expm1`](generated/torch._foreach_expm1.html#torch._foreach_expm1 "torch._foreach_expm1") |将 [`torch.expm1()`](generated/torch.expm1.html#torch.expm1 "torch.expm1") 应用于输入列表的每个tensor。 |
| [`_foreach_expm1_`](generated/torch._foreach_expm1_.html#torch._foreach_expm1_ "torch._foreach_expm1_") |将 [`torch.expm1()`](generated/torch.expm1.html#torch.expm1 "torch.expm1") 应用于输入列表的每个tensor。 |
| [`_foreach_floor`](generated/torch._foreach_floor.html#torch._foreach_floor "torch._foreach_floor") |将 [`torch.floor()`](generated/torch.floor.html#torch.floor "torch.floor") 应用于输入列表的每个tensor。 |
| [`_foreach_floor_`](generated/torch._foreach_floor_.html#torch._foreach_floor_ "torch._foreach_floor_") |将 [`torch.floor()`](generated/torch.floor.html#torch.floor "torch.floor") 应用于输入列表的每个tensor。 |
| [`_foreach_log`](generated/torch._foreach_log.html#torch._foreach_log "torch._foreach_log") |将 [`torch.log()`](generated/torch.log.html#torch.log "torch.log") 应用于输入列表的每个tensor。 |
| [`_foreach_log_`](generated/torch._foreach_log_.html#torch._foreach_log_ "torch._foreach_log_") |将 [`torch.log()`](generated/torch.log.html#torch.log "torch.log") 应用于输入列表的每个tensor。 |
| [`_foreach_log10`](generated/torch._foreach_log10.html#torch._foreach_log10 "torch._foreach_log10") |将 [`torch.log10()`](generated/torch.log10.html#torch.log10 "torch.log10") 应用于输入列表的每个tensor。 |
| [`_foreach_log10_`](generated/torch._foreach_log10_.html#torch._foreach_log10_ "torch._foreach_log10_") |将 [`torch.log10()`](generated/torch.log10.html#torch.log10 "torch.log10") 应用于输入列表的每个tensor。 |
| [`_foreach_log1p`](generated/torch._foreach_log1p.html#torch._foreach_log1p "torch._foreach_log1p") |将 [`torch.log1p()`](generated/torch.log1p.html#torch.log1p "torch.log1p") 应用于输入列表的每个tensor。 |
| [`_foreach_log1p_`](generated/torch._foreach_log1p_.html#torch._foreach_log1p_ "torch._foreach_log1p_") |将 [`torch.log1p()`](generated/torch.log1p.html#torch.log1p "torch.log1p") 应用于输入列表的每个tensor。 |
| [`_foreach_log2`](generated/torch._foreach_log2.html#torch._foreach_log2 "torch._foreach_log2") |将 [`torch.log2()`](generated/torch.log2.html#torch.log2 "torch.log2") 应用于输入列表的每个tensor。 |
| [`_foreach_log2_`](generated/torch._foreach_log2_.html#torch._foreach_log2_ "torch._foreach_log2_") |将 [`torch.log2()`](generated/torch.log2.html#torch.log2 "torch.log2") 应用于输入列表的每个tensor。 |
| [`_foreach_neg`](generated/torch._foreach_neg.html#torch._foreach_neg "torch._foreach_neg") |将 [`torch.neg()`](generated/torch.neg.html#torch.neg "torch.neg") 应用于输入列表的每个tensor。 |
| [`_foreach_neg_`](generated/torch._foreach_neg_.html#torch._foreach_neg_ "torch._foreach_neg_") |将 [`torch.neg()`](generated/torch.neg.html#torch.neg "torch.neg") 应用于输入列表的每个tensor。 |
| [`_foreach_tan`](generated/torch._foreach_tan.html#torch._foreach_tan "torch._foreach_tan") |将 [`torch.tan()`](generated/torch.tan.html#torch.tan "torch.tan") 应用于输入列表的每个tensor。 |
| [`_foreach_tan_`](generated/torch._foreach_tan_.html#torch._foreach_tan_ "torch._foreach_tan_") |将 [`torch.tan()`](generated/torch.tan.html#torch.tan "torch.tan") 应用于输入列表的每个tensor。 |
| [`_foreach_sin`](generated/torch._foreach_sin.html#torch._foreach_sin "torch._foreach_sin") |将 [`torch.sin()`](generated/torch.sin.html#torch.sin "torch.sin") 应用于输入列表的每个tensor。 |
| [`_foreach_sin_`](generated/torch._foreach_sin_.html#torch._foreach_sin_ "torch._foreach_sin_") |将 [`torch.sin()`](generated/torch.sin.html#torch.sin "torch.sin") 应用于输入列表的每个tensor。 |
| [`_foreach_sinh`](generated/torch._foreach_sinh.html#torch._foreach_sinh "torch._foreach_sinh") |将 [`torch.sinh()`](generated/torch.sinh.html#torch.sinh "torch.sinh") 应用于输入列表的每个tensor。 |
| [`_foreach_sinh_`](generated/torch._foreach_sinh_.html#torch._foreach_sinh_ "torch._foreach_sinh_") |将 [`torch.sinh()`](generated/torch.sinh.html#torch.sinh "torch.sinh") 应用于输入列表的每个tensor。 |
| [`_foreach_round`](generated/torch._foreach_round.html#torch._foreach_round "torch._foreach_round") |将 [`torch.round()`](generated/torch.round.html#torch.round "torch.round") 应用于输入列表的每个tensor。 |
| [`_foreach_round_`](generated/torch._foreach_round_.html#torch._foreach_round_ "torch._foreach_round_") |将 [`torch.round()`](generated/torch.round.html#torch.round "torch.round") 应用于输入列表的每个tensor。 |
| [`_foreach_sqrt`](generated/torch._foreach_sqrt.html#torch._foreach_sqrt "torch._foreach_sqrt") |将 [`torch.sqrt()`](generated/torch.sqrt.html#torch.sqrt "torch.sqrt") 应用于输入列表的每个tensor。 |
| [`_foreach_sqrt_`](generated/torch._foreach_sqrt_.html#torch._foreach_sqrt_ "torch._foreach_sqrt_") |将 [`torch.sqrt()`](generated/torch.sqrt.html#torch.sqrt "torch.sqrt") 应用于输入列表的每个tensor。 |
| [`_foreach_lgamma`](generated/torch._foreach_lgamma.html#torch._foreach_lgamma "torch._foreach_lgamma") |将 [`torch.lgamma()`](generated/torch.lgamma.html#torch.lgamma "torch.lgamma") 应用于输入列表的每个tensor。 |
| [`_foreach_lgamma_`](generated/torch._foreach_lgamma_.html#torch._foreach_lgamma_ "torch._foreach_lgamma_") |将 [`torch.lgamma()`](generated/torch.lgamma.html#torch.lgamma "torch.lgamma") 应用于输入列表的每个tensor。 |
| [`_foreach_frac`](generated/torch._foreach_frac.html#torch._foreach_frac "torch._foreach_frac") |将 [`torch.frac()`](generated/torch.frac.html#torch.frac "torch.frac") 应用于输入列表的每个tensor。 |
| [`_foreach_frac_`](generated/torch._foreach_frac_.html#torch._foreach_frac_ "torch._foreach_frac_") |将 [`torch.frac()`](generated/torch.frac.html#torch.frac "torch.frac") 应用于输入列表的每个tensor。 |
| [`_foreach_reciprocal`](generated/torch._foreach_reciprocal.html#torch._foreach_reciprocal "torch._foreach_reciprocal") |将 [`torch.reciprocal()`](generated/torch.reciprocal.html#torch.reciprocal "torch.reciprocal") 应用于输入列表的每个tensor。 |
| [`_foreach_reciprocal_`](generated/torch._foreach_reciprocal_.html#torch._foreach_reciprocal_ "torch._foreach_reciprocal_") |将 [`torch.reciprocal()`](generated/torch.reciprocal.html#torch.reciprocal "torch.reciprocal") 应用于输入列表的每个tensor。 |
| [`_foreach_sigmoid`](generated/torch._foreach_sigmoid.html#torch._foreach_sigmoid "torch._foreach_sigmoid") |将 [`torch.sigmoid()`](generated/torch.sigmoid.html#torch.sigmoid "torch.sigmoid") 应用于输入列表的每个tensor。 |
| [`_foreach_sigmoid_`](generated/torch._foreach_sigmoid_.html#torch._foreach_sigmoid_ "torch._foreach_sigmoid_") |将 [`torch.sigmoid()`](generated/torch.sigmoid.html#torch.sigmoid "torch.sigmoid") 应用于输入列表的每个tensor。 |
| [`_foreach_trunc`](generated/torch._foreach_trunc.html#torch._foreach_trunc "torch._foreach_trunc") |将 [`torch.trunc()`](generated/torch.trunc.html#torch.trunc "torch.trunc") 应用于输入列表的每个tensor。 |
| [`_foreach_trunc_`](generated/torch._foreach_trunc_.html#torch._foreach_trunc_ "torch._foreach_trunc_") |将 [`torch.trunc()`](generated/torch.trunc.html#torch.trunc "torch.trunc") 应用于输入列表的每个tensor。 |
| [`_foreach_zero_`](generated/torch._foreach_zero_.html#torch._foreach_zero_ "torch._foreach_zero_") |将“torch.zero()”应用于输入列表的每个tensor。 |


## 实用程序 [¶](#utilities "此标题的永久链接")


|  |  |
| --- | --- |
| [`compiled_with_cxx11_abi`](generated/torch.compiled_with_cxx11_abi.html#torch.compiled_with_cxx11_abi "torch.compiled_with_cxx11_abi") |返回 PyTorch 是否使用 _GLIBCXX_USE_CXX11_ABI=1 |
| 构建


[`结果_type`](generated/torch.result_type.html#torch.result_type "torch.result_type") |返回对提供的输入tensor执行算术运算所产生的 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype")。 |
| [`can_cast`](generated/torch.can_cast.html#torch.can_cast "torch.can_cast") |确定类型提升 [文档](tensor_attributes.html#type-promotion-doc) 中描述的 PyTorch 转换规则是否允许类型转换。 |
| [`promote_types`](generated/torch.promote_types.html#torch.promote_types "torch.promote_types") |返回具有最小尺寸和标量种类的 [`torch.dtype`](tensor_attributes.html#torch.dtype "torch.dtype")，该标量种类不小于 type1 或 type2 ，也不低于 type1 或 type2 。 |
| [`使用_确定性_算法`](generated/torch.use_definistic_algorithms.html#torch.use_definistic_algorithms "torch.use_definistic_algorithms") |设置 PyTorch 操作是否必须使用“确定性”算法。 |
| [`are_definistic_algorithms_enabled`](generated/torch.are_definistic_algorithms_enabled.html#torch.are_definistic_algorithms_enabled "torch.are_definistic_algorithms_enabled") |如果全局确定性标志打开，则返回 True。 |
| [`is_definistic_algorithms_warn_only_enabled`](generated/torch.is_definistic_algorithms_warn_only_enabled.html#torch.is_definistic_algorithms_warn_only_enabled "torch.is_definistic_algorithms_warn_only_enabled") |如果全局确定性标志设置为仅警告，则返回 True。 |
| [`设置_确定性_调试_模式`](generated/torch.set_确定性_调试_模式.html#torch.set_确定性_调试_模式“火炬.set_确定性_调试_模式”) |设置确定性操作的调试模式。 |
| [`获取_确定性_调试_模式`](generated/torch.get_确定性_调试_模式.html#torch.get_确定性_调试_模式“torch.get_确定性_调试_模式”) |返回确定性操作的调试模式的当前值。 |
| [`set_float32_matmul_ precision`](generated/torch.set_float32_matmul_ precision.html#torch.set_float32_matmul_ precision "torch.set_float32_matmul_ precision") |设置 float32 矩阵乘法的内部精度。 |
| [`get_float32_matmul_ precision`](generated/torch.get_float32_matmul_ precision.html#torch.get_float32_matmul_ precision "torch.get_float32_matmul_ precision") |返回 float32 矩阵乘法精度的当前值。 |
| [`set_warn_always`](generated/torch.set_warn_always.html#torch.set_warn_always "torch.set_warn_always") |当此标志为 False(默认)时，某些 PyTorch 警告可能每个进程仅出现一次。 |
| [`is_warn_always_enabled`](generated/torch.is_warn_always_enabled.html#torch.is_warn_always_enabled "torch.is_warn_always_enabled") |如果全局 warn_always 标志打开，则返回 True。 |
| [`vmap`](generated/torch.vmap.html#torch.vmap "torch.vmap") | vmap 是矢量化映射； `vmap(func)` 返回一个新函数，将 `func` 映射到输入的某个维度。 |
| [`_assert`](generated/torch._assert.html#torch._assert "torch._assert") | Python 断言的包装器，可进行符号追踪。 |


## 符号数字 [¶](#symbolic-numbers "此标题的永久链接")


*班级*


 火炬。


 符号整数


 ( *node
* ) [[source]](_modules/torch.html#SymInt)[¶](#torch.SymInt "此定义的永久链接")


 与 int 类似(包括魔术方法)，但重定向包装节点上的所有操作。这特别用于在符号形状工作流程中以符号方式记录操作。


*班级*


 火炬。


 符号浮点数


 ( *node
* ) [[source]](_modules/torch.html#SymFloat)[¶](#torch.SymFloat "此定义的永久链接")


 类似于浮点数(包括魔术方法)，但重定向包装节点上的所有操作。这特别用于在符号形状工作流程中以符号方式记录操作。


*班级*


 火炬。


 象征


 ( *node
* ) [[source]](_modules/torch.html#SymBool)[¶](#torch.SymBool "此定义的永久链接")


 类似于 bool(包括魔术方法)，但重定向包装节点上的所有操作。这特别用于在符号形状工作流程中以符号方式记录操作。


 与常规布尔值不同，常规布尔运算符将强制进行额外的保护，而不是进行符号求值。请改用按位运算符来处理此问题。


|  |  |
| --- | --- |
| [`sym_float`](generated/torch.sym_float.html#torch.sym_float "torch.sym_float") |用于浮动铸造的 SymInt 感知实用程序。 |
| [`sym_int`](generated/torch.sym_int.html#torch.sym_int "torch.sym_int") |用于 int 转换的 SymInt 感知实用程序。 |
| [`sym_max`](generated/torch.sym_max.html#torch.sym_max "torch.sym_max") | max() 的 SymInt 感知实用程序。 |
| [`sym_min`](generated/torch.sym_min.html#torch.sym_min "torch.sym_min") | max() 的 SymInt 感知实用程序。 |
| [`sym_not`](generated/torch.sym_not.html#torch.sym_not "torch.sym_not") |用于逻辑否定的 SymInt 感知实用程序。 |


## 导出路径 [¶](#export-path "此标题的永久链接")


!!! warning "警告"

     此功能是一个原型，将来可能会出现兼容性重大变化。


 导出generated/导出数据库/索引


## 优化 [¶](#optimizations "此标题的永久链接")


|  |  |
| --- | --- |
| [`编译`](generated/torch.compile.html#torch.compile "torch.compile") |使用 TorchDynamo 和指定后端优化给定模型/函数。 | [torch.compile 文档](https://pytorch.org/docs/main/compile/index.html)


## 操作员标签 [¶](#operator-tags "此标题的永久链接")


*班级*


 火炬。


 标签 [¶](#torch.Tag"此定义的永久链接")


 成员：



 core
 


 数据依赖输出


 动态_输出_形状


 生成的


 就地_view


 按位非确定性


 不确定性_种子


 逐点


 查看_复制


*财产*


 name [¶](#torch.Tag.name "此定义的永久链接")