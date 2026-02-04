# torch.backends [¶](#module-torch.backends "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/backends>
>
> 原始地址：<https://pytorch.org/docs/stable/backends.html>


 torch.backends 控制 PyTorch 支持的各种后端的行为。


 这些后端包括：



* `torch.backends.cpu`
* `torch.backends.cuda`
* `torch.backends.cudnn`
* `torch.backends.mps`
* `torch.backends.mkl`
* `torch.backends.mkldnn`*` torch.backends.openmp`
* `torch.backends.opt_einsum`
* `torch.backends.xeon`


## torch.backends.cpu [¶](#module-torch.backends.cpu "此标题的永久链接") 


 torch.backends.cpu。


 获取_cpu_能力


 ( ) [[source]](_modules/torch/backends/cpu.html#get_cpu_capability)[¶](#torch.backends.cpu.get_cpu_capability "此定义的永久链接")


 以字符串值形式返回 cpu 能力。


 可能的值：-“默认”-“VSX”-“Z 向量”-“无 AVX”-“AVX2”-“AVX512”


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


## torch.backends.cuda [¶](#module-torch.backends.cuda "此标题的永久链接")


 火炬.后端.cuda。


 建成


 ( ) [[source]](_modules/torch/backends/cuda.html#is_built)[¶](#torch.backends.cuda.is_built "此定义的永久链接")


 返回 PyTorch 是否是使用 CUDA 支持构建的。请注意，这并不一定意味着 CUDA 可用；只是如果这个 PyTorch 二进制文件运行在具有可用 CUDA 驱动程序和设备的机器上，我们就能够使用它。


 torch.backends.cuda.matmul。


 allow_tf32 [¶](#torch.backends.cuda.matmul.allow_tf32 "此定义的永久链接")


 一个 [`bool`](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") 控制 TensorFloat-32 tensor核是否可以用于矩阵乘法Ampere 或更新的 GPU。请参阅[Ampere 设备上的 TensorFloat-32(TF32)](notes/cuda.html#tf32-on-ampere) 。


 torch.backends.cuda.matmul。


 allow_fp16_reduced_ precision_reduction [¶](#torch.backends.cuda.matmul.allow_fp16_reduced_ precision_reduction "此定义的永久链接")


 一个 [`bool`](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") 控制是否降低精度(例如，使用 fp16 累积类型) fp16 GEMM 允许。


 torch.backends.cuda.matmul。


 allow_bf16_reduced_ precision_reduction [¶](#torch.backends.cuda.matmul.allow_bf16_reduced_ precision_reduction "此定义的永久链接")


 一个 [`bool`](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") 控制 bf16 GEMM 是否允许降低精度。


 火炬.后端.cuda。


 cufft_plan_cache [¶](#torch.backends.cuda.cufft_plan_cache "此定义的永久链接")


`cufft_plan_cache` 包含每个 CUDA 设备的 cuFFT 计划缓存。通过 torch.backends.cuda.cufft_plan_cache[i] 查询特定设备 i 的缓存。


 torch.backends.cuda.cufft_plan_cache。


 size [¶](#torch.backends.cuda.cufft_plan_cache.size "此定义的永久链接")


 只读 [`int`](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")，显示当前 cuFFT 计划缓存中的计划数量。


 torch.backends.cuda.cufft_plan_cache。


 max_size [¶](#torch.backends.cuda.cufft_plan_cache.max_size "此定义的永久链接")


 控制 cuFFT 计划缓存容量的 [`int`](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")。


 torch.backends.cuda.cufft_plan_cache。



 clear
 


 ( ) [¶](#torch.backends.cuda.cufft_plan_cache.clear "此定义的永久链接")


 清除 cuFFT 计划缓存。


 火炬.后端.cuda。


 首选_linalg_library


 (*后端



 =
 


 无
* ) [[source]](_modules/torch/backends/cuda.html#preferred_linalg_library)[¶](#torch.backends.cuda.preferred_linalg_library "此定义的永久链接")


!!! warning "警告"

     该标志是实验性的，可能会发生变化。


 当 PyTorch 运行 CUDA 线性代数运算时，它通常使用 cuSOLVER 或 MAGMA 库，如果两者都可用，它会决定使用启发式方法。此标志(a [`str`](https://docs.python.org /3/library/stdtypes.html#str "(in Python v3.12)") ) 允许覆盖这些启发式方法。



* 如果设置“cusolver”，则将尽可能使用 cuSOLVER。
* 如果设置“magma”，则将尽可能使用 MAGMA。
* 如果设置“default”(默认)，则将使用启发式在 cuSOLVER 和MAGMA 如果两者都可用。
* 当没有给出输入时，此函数返回当前首选库。
* 用户可以使用环境变量 TORCH_LINALG_PREFER_CUSOLVER=1 将首选库全局设置为 cuSOLVER。此标志仅设置首选库的初始值和首选库可能仍会被脚本中的此函数调用覆盖。


 注意：当首选库时，如果首选库未实现调用的操作，则仍可能使用其他库。如果 PyTorch 的启发式库选择对于应用程序的输入不正确，则此标志可能会实现更好的性能。


 目前支持的 linalg 运算符：



* [`torch.linalg.inv()`](generated/torch.linalg.inv.html#torch.linalg.inv "torch.linalg.inv")
* [`torch.linalg.inv_ex()`](generated/torch.linalg.inv_ex.html#torch.linalg.inv_ex "torch.linalg.inv_ex")
* [`torch.linalg.cholesky()`](generated/torch.linalg.cholesky.html#torch.linalg.cholesky "torch.linalg.cholesky")
* [`torch.linalg.cholesky_ex()`](generated/torch.linalg.cholesky_ex.html#torch.linalg.cholesky_ex "torch.linalg.cholesky_ex")
* [ `torch.cholesky_solve()`](generated/torch.cholesky_solve.html#torch.cholesky_solve "torch.cholesky_solve")
* [`torch.cholesky_inverse()`](generated/torch.cholesky_inverse.html#torch.cholesky_inverse "torch.cholesky_inverse")
* [`torch.linalg.lu_factor()`](generated/torch.linalg.lu_factor.html#torch.linalg.lu_factor "torch.linalg.lu_factor")
* [`torch.linalg.lu()`](generated/torch.linalg.lu.html#torch.linalg.lu "torch.linalg.lu")
* [`torch.linalg.lu_solve()`](generated/torch.linalg.lu_solve.html#torch.linalg.lu_solve "torch.linalg.lu_solve")
* [`torch.linalg.qr()`](generated/torch.linalg.qr.html#torch.linalg.qr "火炬.linalg.qr")
* [`torch.linalg.eigh()`](generated/torch.linalg.eigh.html#torch.linalg.eigh "torch.linalg.eigh")
* `torch.linalg.eighvals( )`
* [`torch.linalg.svd()`](generated/torch.linalg.svd.html#torch.linalg.svd "torch.linalg.svd")
* [`torch.linalg.svdvals()`](generated/torch.linalg.svdvals.html#torch.linalg.svdvals“torch.linalg.svdvals”)


 Return type


*_Linalg后端*


*班级*


 火炬.后端.cuda。


 SDP后端


 ( *value
* ) [[source]](_modules/torch/backends/cuda.html#SDPBackend)[¶](#torch.backends.cuda.SDPBackend "此定义的永久链接")


 用于缩放点积注意力后端的枚举类。


!!! warning "警告"

     该课程处于测试阶段，可能会发生变化。


 该类需要与以下定义的枚举保持一致：pytorch/aten/src/ATen/native/transformers/sdp_utils_cpp.h


 火炬.后端.cuda。


 启用闪存_sdp_


 ( ) [[source]](_modules/torch/backends/cuda.html#flash_sdp_enabled)[¶](#torch.backends.cuda.flash_sdp_enabled "此定义的永久链接")


!!! warning "警告"

     该标志是测试版，可能会发生变化。


 返回是否启用 Flash 缩放点积注意力。


 火炬.后端.cuda。


 启用_mem_高效_sdp


 ( *enabled
* ) [[source]](_modules/torch/backends/cuda.html#enable_mem_efficient_sdp)[¶](#torch.backends.cuda.enable_mem_efficient_sdp "此定义的永久链接")


!!! warning "警告"

     该标志是测试版，可能会发生变化。


 启用或禁用内存高效的缩放点积注意力。


 火炬.后端.cuda。


 mem_高效_sdp_启用


 ( ) [[source]](_modules/torch/backends/cuda.html#mem_efficient_sdp_enabled)[¶](#torch.backends.cuda.mem_efficient_sdp_enabled "此定义的永久链接")


!!! warning "警告"

     该标志是测试版，可能会发生变化。


 返回是否启用内存高效缩放点积注意力。


 火炬.后端.cuda。


 启用_flash_sdp


 ( *enabled
* ) [[source]](_modules/torch/backends/cuda.html#enable_flash_sdp)[¶](#torch.backends.cuda.enable_flash_sdp "此定义的永久链接")


!!! warning "警告"

     该标志是测试版，可能会发生变化。


 启用或禁用闪存缩放点积注意力。


 火炬.后端.cuda。


 数学_sdp_启用


 ( ) [[source]](_modules/torch/backends/cuda.html#math_sdp_enabled)[¶](#torch.backends.cuda.math_sdp_enabled "此定义的永久链接")


!!! warning "警告"

     该标志是测试版，可能会发生变化。


 返回是否启用数学缩放点积注意力。


 火炬.后端.cuda。


 启用_math_sdp


 ( *enabled
* ) [[source]](_modules/torch/backends/cuda.html#enable_math_sdp)[¶](#torch.backends.cuda.enable_math_sdp "此定义的永久链接")


!!! warning "警告"

     该标志是测试版，可能会发生变化。


 启用或禁用数学缩放点积注意力。


 火炬.后端.cuda。


 sdp_kernel


 ( *启用_flash



 =
 


 True
* , *启用_math



 =
 


 True
* , *启用_mem_efficient



 =
 


 True
* ) [[source]](_modules/torch/backends/cuda.html#sdp_kernel)[¶](#torch.backends.cuda.sdp_kernel "此定义的永久链接")


!!! warning "警告"

     该标志是测试版，可能会发生变化。


 此上下文管理器可用于临时启用或禁用缩放点积注意力的三个后端中的任何一个。退出上下文管理器后，将恢复标志的先前状态。


## torch.backends.cudnn [¶](#module-torch.backends.cudnn "此标题的永久链接") 


 torch.backends.cudnn。


 版本


 ( ) [[source]](_modules/torch/backends/cudnn.html#version)[¶](#torch.backends.cudnn.version "此定义的永久链接")


 返回 cuDNN 的版本


 torch.backends.cudnn。


 可用


 ( ) [[source]](_modules/torch/backends/cudnn.html#is_available)[¶](#torch.backends.cudnn.is_available "此定义的永久链接")


 返回一个布尔值，指示 CUDNN 当前是否可用。


 torch.backends.cudnn。


 启用[¶](#torch.backends.cudnn.enabled"此定义的永久链接")


 控制是否启用 cuDNN 的 [`bool`](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")。


 torch.backends.cudnn。


 allow_tf32 [¶](#torch.backends.cudnn.allow_tf32 "此定义的永久链接")


 一个 [`bool`](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") 控制 TensorFloat-32 tensor核心可以在 cuDNN 卷积中使用的位置Ampere 或更新的 GPU。请参阅[Ampere 设备上的 TensorFloat-32(TF32)](notes/cuda.html#tf32-on-ampere) 。


 torch.backends.cudnn。


 确定性 [¶](#torch.backends.cudnn.definistic "此定义的永久链接")


 一个 [`bool`](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")，如果为 True，则导致 cuDNN 仅使用确定性卷积算法。另请参阅 [`torch.are_definistic_algorithms_enabled()`](generated/torch.are_definistic_algorithms_enabled.html#torch.are_definistic_algorithms_enabled "torch.are_definistic_algorithms_enabled") 和 [`torch.use_definistic_algorithms()`](generated/torch.use_definistic_algorithms.html#torch.use_definistic_algorithms“torch.use_definistic_algorithms”)。


 torch.backends.cudnn。


 基准 [¶](#torch.backends.cudnn.benchmark"此定义的永久链接")


 一个 [`bool`](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")，如果为 True，则使 cuDNN 对多个卷积算法进行基准测试并选择最快的。


 torch.backends.cudnn。


 benchmark_limit [¶](#torch.backends.cudnn.benchmark_limit "此定义的永久链接")


 一个 [`int`](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)")，指定 torch.txt 时要尝试的 cuDNN 卷积算法的最大数量。 backends.cudnn.benchmark 为 True。将 benchmark_limit 设置为零以尝试每种可用的算法。请注意，此设置仅影响通过 cuDNN v8 API 调度的卷积。


## torch.backends.mps [¶](#module-torch.backends.mps "此标题的永久链接") 


 torch.backends.mps。


 可用


 ( ) [[source]](_modules/torch/backends/mps.html#is_available)[¶](#torch.backends.mps.is_available "此定义的永久链接")


 返回一个布尔值，指示 MPS 当前是否可用。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 torch.backends.mps。


 建成


 ( ) [[source]](_modules/torch/backends/mps.html#is_built)[¶](#torch.backends.mps.is_built "此定义的永久链接")


 返回 PyTorch 是否构建有 MPS 支持。请注意，这并不一定意味着 MPS 可用；只是如果这个 PyTorch 二进制文件运行在具有可用 MPS 驱动程序和设备的机器上，我们就可以使用它。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


## torch.backends.mkl [¶](#module-torch.backends.mkl "此标题的永久链接")


 torch.backends.mkl。


 可用


 ( ) [[source]](_modules/torch/backends/mkl.html#is_available)[¶](#torch.backends.mkl.is_available "此定义的永久链接")


 返回 PyTorch 是否是使用 MKL 支持构建的。


*班级*


 torch.backends.mkl。


 冗长的


 ( *enable
* ) [[source]](_modules/torch/backends/mkl.html#verbose)[¶](#torch.backends.mkl.verbose "此定义的永久链接")


 按需 oneMKL 详细信息功能为了更轻松地调试性能问题，oneMKL 可以在执行内核时转储包含执行信息(例如持续时间)的详细消息。详细功能可以通过名为 MKL_VERBOSE 的环境变量调用。然而，这种方法会在所有步骤中转储消息。这些是大量冗长的消息。此外，为了调查性能问题，通常在一次迭代中获取详细消息就足够了。这种按需详细信息功能使得控制详细消息转储的范围成为可能。在以下示例中，将转储详细消息，仅用于第二次推理。


```
import torch
model(data)
with torch.backends.mkl.verbose(torch.backends.mkl.VERBOSE_ON):
    model(data)

```


 Parameters


**level** – 详细级别 
- `VERBOSE_OFF` ：禁用详细信息 
- `VERBOSE_ON` ：启用详细信息


## torch.backends.mkldnn [¶](#module-torch.backends.mkldnn "此标题的永久链接") 


 torch.backends.mkldnn。


 可用


 ( ) [[source]](_modules/torch/backends/mkldnn.html#is_available)[¶](#torch.backends.mkldnn.is_available "此定义的永久链接")


 返回 PyTorch 是否是使用 MKL-DNN 支持构建的。


*班级*


 torch.backends.mkldnn。


 冗长的


 ( *level
* ) [[source]](_modules/torch/backends/mkldnn.html#verbose)[¶](#torch.backends.mkldnn.verbose "此定义的永久链接")


 按需 oneDNN(以前的 MKL-DNN)详细功能为了更轻松地调试性能问题，oneDNN 可以在执行内核时转储包含内核大小、输入数据大小和执行持续时间等信息的详细消息。详细功能可以通过名为 DNNL_VERBOSE 的环境变量调用。然而，这种方法会在所有步骤中转储消息。这些是大量冗长的消息。此外，为了调查性能问题，通常在一次迭代中获取详细消息就足够了。这种按需详细功能使得控制详细消息转储的范围成为可能。在以下示例中，仅在第二次推理时才会转储详细消息。


```
import torch
model(data)
with torch.backends.mkldnn.verbose(torch.backends.mkldnn.VERBOSE_ON):
    model(data)

```


 Parameters


**level** – 详细级别 
- `VERBOSE_OFF` ：禁用详细信息 
- `VERBOSE_ON` ：启用详细信息 
- `VERBOSE_ON_CREATION` ：启用详细信息，包括 oneDNN 内核创建


## torch.backends.openmp [¶](#module-torch.backends.openmp "此标题的永久链接") 


 torch.backends.openmp。


 可用


 ( ) [[source]](_modules/torch/backends/openmp.html#is_available)[¶](#torch.backends.openmp.is_available "此定义的永久链接")


 返回 PyTorch 是否是使用 OpenMP 支持构建的。


## torch.backends.opt_einsum [¶](#module-torch.backends.opt_einsum "此标题的永久链接")


 torch.backends.opt_einsum。


 可用


 ( ) [[source]](_modules/torch/backends/opt_einsum.html#is_available)[¶](#torch.backends.opt_einsum.is_available "此定义的永久链接")


 返回一个布尔值，指示 opt_einsum 当前是否可用。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)


 torch.backends.opt_einsum。


 获取_opt_einsum


 ( ) [[source]](_modules/torch/backends/opt_einsum.html#get_opt_einsum)[¶](#torch.backends.opt_einsum.get_opt_einsum "此定义的永久链接")


 如果 opt_einsum 当前可用，则返回 opt_einsum 包，否则返回 None。


 Return type


[*任何*](https://docs.python.org/3/library/typing.html#typing.Any“(在Python v3.12中)”)


 torch.backends.opt_einsum。


 启用[¶](#torch.backends.opt_einsum.enabled"此定义的永久链接")


 一个 :class: `bool` 控制是否启用 opt_einsum (默认为 `True`)。如果是这样，torch.einsum 将使用 opt_einsum ( <https://optimized-einsum.readthedocs.io/en/stable/path_finding.html
> )(如果可用)来计算最佳收缩路径以获得更快的性能。


 如果 opt_einsum 不可用，torch.einsum 将回退到默认的从左到右的收缩路径。


 torch.backends.opt_einsum。


 策略 [¶](#torch.backends.opt_einsum.strategy "此定义的永久链接")


 一个 :class: `str` 指定当 `torch.backends.opt_einsum.enabled` 为 `True` 时尝试哪些策略。默认情况下，torch.einsum 将尝试“自动”策略，但也支持“贪婪”和“最优”策略。请注意，“最佳”策略是输入数量的阶乘，因为它会尝试所有可能的路径。请参阅 opt_einsum 的文档( <https://optimized-einsum.readthedocs.io/en/stable/path_finding.html
> )了解更多详细信息。


## torch.backends.xeon [¶](#module-torch.backends.xeon "此标题的永久链接")