# torch.sparse [¶](#torch-sparse "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/sparse>
>
> 原始地址：<https://pytorch.org/docs/stable/sparse.html>


!!! warning "警告"

     稀疏tensor的 PyTorch API 处于测试阶段，可能会在不久的将来发生变化。我们非常欢迎将功能请求、错误报告和一般建议作为 GitHub 问题。


## 为什么以及何时使用稀疏性 [¶](#why-and-when-to-use-sparsity "永久链接到此标题")


 默认情况下，PyTorch 存储 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 将元素连续存储在物理内存中。这导致需要快速访问元素的各种数组处理算法的有效实现。


 现在，一些用户可能决定用*元素大多为零值*的tensor来表示图邻接矩阵、修剪权重或点云等数据。我们认识到这些是重要的应用程序，并旨在通过稀疏存储格式为这些用例提供性能优化。


 多年来已经开发出各种稀疏存储格式，例如COO、CSR/CSC、半结构化、LIL等。虽然它们在精确布局上有所不同，但它们都通过零值元素的有效表示来压缩数据。我们将未压缩值称为“指定”，与“未指定”、压缩元素相对照。


 通过压缩重复零，稀疏存储格式旨在节省各种 CPU 和 GPU 上的内存和计算资源。特别是对于高度稀疏性或高度结构化稀疏性，这可能会产生显着的性能影响。因此，稀疏存储格式可以被视为一种性能优化。


 与许多其他性能优化一样，稀疏存储格式并不总是有利的。当为您的用例尝试稀疏格式时，您可能会发现执行时间增加而不是减少。


 如果您在分析上期望看到性能显着提高，但实际测量到性能下降，请放心地打开 GitHub 问题。这有助于我们优先考虑高效内核的实现和更广泛的性能优化。


 我们可以轻松尝试不同的稀疏布局，并在它们之间进行转换，而无需固执己见地选择最适合您的特定应用程序。


## 功能概述[¶](#功能性概述"此标题的永久链接")


 我们希望通过为每个布局提供转换例程，可以直接从给定的稠密tensor构造稀疏tensor。


 在下一个示例中，我们将具有默认密集(跨步)布局的 2D tensor转换为由 COO 内存布局支持的 2D tensor。在这种情况下，仅存储非零元素的值和索引。


```
>>> a = torch.tensor([[0, 2.], [3, 0]])
>>> a.to_sparse()
tensor(indices=tensor([[0, 1],
 [1, 0]]),
 values=tensor([2., 3.]),
 size=(2, 2), nnz=2, layout=torch.sparse_coo)

```


 PyTorch 目前支持 [COO](#sparse-coo-docs) 、[CSR](#sparse-csr-docs) 、[CSC](#sparse-csc-docs) 、[BSR](#sparse-bsr-docs)和 [BSC](#sparse-bsc-docs) 。


 我们还有一个原型实现来支持：ref：半结构化稀疏<sparse-semi-structed-docs>。请参阅参考资料以获取更多详细信息。


 请注意，我们对这些格式进行了一些概括。


 批处理：GPU 等设备需要批处理才能获得最佳性能，因此我们支持批处理维度。


 我们目前提供一个非常简单的批处理版本，其中稀疏格式本身的每个组件都是批处理的。这还要求每个批次条目具有相同数量的指定元素。在本示例中，我们从 3D 密集tensor构造 3D(批次)CSR tensor。


```
>>> t = torch.tensor([[[1., 0], [2., 3.]], [[4., 0], [5., 6.]]])
>>> t.dim()
3
>>> t.to_sparse_csr()
tensor(crow_indices=tensor([[0, 1, 3],
 [0, 1, 3]]),
 col_indices=tensor([[0, 0, 1],
 [0, 0, 1]]),
 values=tensor([[1., 2., 3.],
 [4., 5., 6.]]), size=(2, 2, 2), nnz=3,
 layout=torch.sparse_csr)

```


 密集维度：另一方面，某些数据(例如图嵌入)可能更好地被视为向量的稀疏集合而不是标量。


 在此示例中，我们从 3D 跨步tensor创建一个具有 2 个稀疏维度和 1 个稠密维度的 3D 混合 COO tensor。如果 3D 跨步tensor中的整行为零，则不会存储它。然而，如果该行中的任何值非零，则它们将被完全存储。这减少了索引的数量，因为我们需要每行一个索引，而不是每个元素一个索引。但它也增加了值的存储量。因为只有“完全”为零的行才能被发出，并且任何非零值元素的存在都会导致整个行被存储。


```
>>> t = torch.tensor([[[0., 0], [1., 2.]], [[0., 0], [3., 4.]]])
>>> t.to_sparse(sparse_dim=2)
tensor(indices=tensor([[0, 1],
 [1, 1]]),
 values=tensor([[1., 2.],
 [3., 4.]]),
 size=(2, 2, 2), nnz=2, layout=torch.sparse_coo)

```


## 操作员概述 [¶](#operator-overview "此标题的永久链接")


 从根本上讲，对具有稀疏存储格式的tensor的操作与对具有跨步(或其他)存储格式的tensor的操作的行为相同。存储的特殊性(即数据的物理布局)会影响操作的性能，但不应影响语义。


 我们正在积极增加稀疏tensor的算子覆盖范围。用户不应期望支持与密集tensor相同级别的支持。有关列表，请参阅我们的 [operator](#sparse-ops-docs) 文档。


```
>>> b = torch.tensor([[0, 0, 1, 2, 3, 0], [4, 5, 0, 6, 0, 0]])
>>> b_s = b.to_sparse_csr()
>>> b_s.cos()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
RuntimeError: unsupported tensor layout: SparseCsr
>>> b_s.sin()
tensor(crow_indices=tensor([0, 3, 6]),
 col_indices=tensor([2, 3, 4, 0, 1, 3]),
 values=tensor([ 0.8415, 0.9093, 0.1411, -0.7568, -0.9589, -0.2794]),
 size=(2, 6), nnz=6, layout=torch.sparse_csr)

```


 如上面的示例所示，我们不支持非零保留一元运算符，例如 cos。非零保留一元运算的输出将无法在与输入相同的程度上利用稀疏存储格式，并可能导致内存灾难性增加。相反，我们依赖用户首先显式转换为密集tensor然后运行该操作。


```
>>> b_s.to_dense().cos()
tensor([[ 1.0000, -0.4161],
 [-0.9900, 1.0000]])

```


 我们知道一些用户希望忽略 cos 等运算的压缩零，而不是保留运算的确切语义。为此，我们可以参考 torch.masked 及其 MaskedTensor，后者也由稀疏存储格式和内核提供支持和支持。


 另请注意，目前用户无法选择输出布局。例如，将稀疏tensor添加到常规跨步tensor会产生跨步tensor。有些用户可能更喜欢保持稀疏布局，因为他们知道结果仍然足够稀疏。


```
>>> a + b.to_sparse()
tensor([[0., 3.],
 [3., 0.]])

```


 我们承认，访问能够有效生成不同输出布局的内核非常有用。后续操作可能会从接收特定布局中受益匪浅。我们正在开发一个 API 来控制结果布局，并认识到为任何给定模型规划更优化的执行路径是一个重要功能。


## 稀疏半结构化tensor [¶](#sparse-semi-structed-tensors "此标题的固定链接")


!!! warning "警告"

     稀疏半结构化tensor目前是一个原型功能，可能会发生变化。如果您有反馈要分享，请随时打开问题来报告错误。


 半结构化稀疏性是一种稀疏数据布局，首次在 NVIDIA 的 Ampere 架构中引入。它也称为 **细粒度结构化稀疏性** 或 **2:4 结构化稀疏性** 。


 这种稀疏布局每 2n 个元素存储 n 个元素，其中 n 由tensor数据类型 (dtype) 的宽度决定。最常用的数据类型是 float16，其中 n=2，因此术语“2:4 结构化稀疏性”。


 [这篇 NVIDIA 博客文章](https://developer.nvidia.com/blog/exploiting-ampere-structed-sparsity-with-cusparselt) 更详细地解释了半结构化稀疏性。


 在 PyTorch 中，半结构化稀疏性是通过 Tensor 子类实现的。通过子类化，我们可以重写 `__torch_dispatch__` ，允许我们在执行矩阵乘法时使用更快的稀疏内核。我们还可以存储tensor在子类中以压缩形式存在，以减少内存开销。


 在这种压缩形式中，稀疏tensor通过仅保留“指定”元素和一些对掩码进行编码的元数据来存储。




!!! note "笔记"

    半结构化稀疏tensor的指定元素和元数据掩码一起存储在单个平面压缩tensor中。它们相互附加以形成连续的内存块。


 压缩tensor = [ 原始tensor的指定元素 |元数据_mask ]


 对于大小为 (r, c) 的原始tensor，我们期望前 m * k //2 个元素是保留的元素，tensor的其余部分是元数据。


 为了让用户更容易查看指定的元素和掩码，可以使用.indices() 和.values() 分别访问掩码和指定的元素。



* `.values()` 返回大小为 (r, c//2) 的tensor中的指定元素，并且具有与稠密矩阵相同的数据类型。
* `.indices()` 返回tensor中的元数据_mask size (r, c//2 ) ，如果 dtype 为 torch.float16 或 torch.bfloat16，则元素类型为 `torch.int16`；如果 dtype 为 torch.int8，则元素类型为 `torch.int32`。


 对于 2:4 稀疏tensor，元数据开销很小 
- 每个指定元素只需 2 位。




!!! note "笔记"

    需要注意的是，“torch.float32”仅支持 1:2 稀疏度。因此，它不遵循与上面相同的公式。


 在这里，我们详细介绍如何计算 2:4 稀疏tensor的压缩比(尺寸密集/尺寸稀疏)。


 令 (r, c) = tensor.shape 且 e = bitwidth(tensor.dtype) ，因此对于 `torch.float16` 和 `torch.bfloat16` e = 16，对于 `torch.int8` e = 8。


 M
 


 密集


 = r × c × e




 M
 


 稀疏



 =
 


 M
 


 指定的



 +
 


 M
 


 元数据


 = r ×


 c
 

 2
 


 × e 
+ r ×


 c
 

 2
 


 × 2 =


 RCE


 2
 


 
+ r c = r c e (


 1
 

 2
 


 +
 


 1
 

 e
 


 )
 


 M_{密集} = r 	imes c 	imes e \M_{稀疏} = M_{指定} 
+ M_{元数据} = r 	imes rac{c}{2} 	imes e 
+ r 	imes rac{c}{2} 	imes 2 = rac{rce}{2} 
+ rc =rce(rac{1}{2} +rac{1}{e})



 M
 


 致密


 ​
 




 =
 




 r
 



 ×
 




 c
 



 ×
 




 e
 


 M
 


 稀疏


 ​
 




 =
 


 M
 


 指定的


 ​
 




 +
 


 M
 


 元数据


 ​
 




 =
 




 r
 



 ×
 



 2
 




 c
 




 ​
 



 ×
 




 e
 



 +
 




 r
 



 ×
 



 2
 




 c
 




 ​
 



 ×
 




 2
 



 =
 



 2
 




 rce
 




 ​
 



 +
 




 rc
 



 =
 


 有效值(



 2
 




 1
 




 ​
 



 +
 



 e
 




 1
 




 ​
 




 )
 


 使用这些计算，我们可以确定原始密集表示和新稀疏表示的总内存占用量。


 这为我们提供了一个简单的压缩比公式，该公式仅取决于tensor数据类型的位宽。




 C
 

 =
 



 M
 


 稀疏




 M
 


 密集




 =
 


 1
 

 2
 


 +
 


 1
 

 e
 


 C = rac{M_{稀疏}}{M_{密集}} = rac{1}{2} 
+ rac{1}{e}


 C
 



 =
 




 M
 


 致密


 ​
 



 M
 


 稀疏


 ​
 


 ​
 



 =
 



 2
 




 1
 




 ​
 



 +
 



 e
 




 1
 




 ​
 


 通过这个公式，我们发现 `torch.float16` 或 `torch.bfloat16` 的压缩率为 56.25%，而 `torch.int8` 的压缩率为 62.5%。


### 构造稀疏半结构化tensor [¶](#constructing-sparse-semi-structed-tensors "永久链接到此标题")


 只需使用“torch.to_sparse_semi_structed”函数即可将稠密tensor转换为稀疏半结构化tensor。


 另请注意，我们仅支持 CUDA tensor，因为半结构化稀疏性的硬件兼容性仅限于 NVIDIA GPU。


 半结构化稀疏性支持以下数据类型。请注意，每种数据类型都有自己的形状约束和压缩因子。


| 	 PyTorch dtype	  | 	 Shape Constraints	  | 	 Compression Factor	  | 	 Sparsity Pattern	  |
| --- | --- | --- | --- |
| 	`torch.float16`	 | 	 Tensor must be 2D and (r, c) must both be a positive multiple of 64	  | 	 9/16	  | 	 2:4	  |
| 	`torch.bfloat16`	 | 	 Tensor must be 2D and (r, c) must both be a positive multiple of 64	  | 	 9/16	  | 	 2:4	  |
| 	`torch.int8`	 | 	 Tensor must be 2D and (r, c) must both be a positive multiple of 128	  | 	 10/16	  | 	 2:4	  |


 要构造半结构化稀疏tensor，首先创建一个遵循 2:4(或半结构化)稀疏格式的常规稠密tensor。为此，我们平铺一个小的 1x4 条带以创建 16x16 稠密 float16 tensor。然后，我们可以调用`to_sparse_semi_structed`函数对其进行压缩以加速推理。


```
>>> from torch.sparse import to_sparse_semi_structured
>>> A = torch.Tensor([0, 0, 1, 1]).tile((128, 32)).half().cuda()
tensor([[0., 0., 1., ..., 0., 1., 1.],
 [0., 0., 1., ..., 0., 1., 1.],
 [0., 0., 1., ..., 0., 1., 1.],
 ...,
 [0., 0., 1., ..., 0., 1., 1.],
 [0., 0., 1., ..., 0., 1., 1.],
 [0., 0., 1., ..., 0., 1., 1.]], device='cuda:0', dtype=torch.float16)
>>> A_sparse = to_sparse_semi_structured(A)
SparseSemiStructuredTensor(shape=torch.Size([128, 128]), transposed=False, values=tensor([[1., 1., 1., ..., 1., 1., 1.],
 [1., 1., 1., ..., 1., 1., 1.],
 [1., 1., 1., ..., 1., 1., 1.],
 ...,
 [1., 1., 1., ..., 1., 1., 1.],
 [1., 1., 1., ..., 1., 1., 1.],
 [1., 1., 1., ..., 1., 1., 1.]], device='cuda:0', dtype=torch.float16), metadata=tensor([[-4370, -4370, -4370, ..., -4370, -4370, -4370],
 [-4370, -4370, -4370, ..., -4370, -4370, -4370],
 [-4370, -4370, -4370, ..., -4370, -4370, -4370],
 ...,
 [-4370, -4370, -4370, ..., -4370, -4370, -4370],
 [-4370, -4370, -4370, ..., -4370, -4370, -4370],
 [-4370, -4370, -4370, ..., -4370, -4370, -4370]], device='cuda:0',
dtype=torch.int16))

```


### 稀疏半结构化tensor运算 [¶](#sparse-semi-structed-tensor-operations "永久链接到此标题")


 目前，半结构化稀疏tensor支持以下操作：



* torch.addmm(偏置、密集、稀疏.t())
* torch.mm(密集、稀疏)
* torch.mm(稀疏、密集)
* aten.linear.default(密集、稀疏、偏置)
* aten.t.default(稀疏)
* aten.t.detach(稀疏)


 要使用这些操作，只需在tensor在半结构化稀疏格式中包含 0 时，将“to_sparse_semi_structured(tensor)”的输出传递给“to_sparse_semi_structed(tensor)”，而不是使用“tensor”，如下所示：


```
>>> a = torch.Tensor([0, 0, 1, 1]).tile((64, 16)).half().cuda()
>>> b = torch.rand(64, 64).half().cuda()
>>> c = torch.mm(a, b)
>>> a_sparse = to_sparse_semi_structured(a)
>>> torch.allclose(c, torch.mm(a_sparse, b))
True

```


### 使用半结构化稀疏性加速 nn.Linear [¶](#acceleating-nn-线性-with-semi-structed-sparsity "永久链接到此标题")


 如果权重已经是半结构化稀疏的，只需几行代码，您就可以加速模型中的线性层：


```
>>> input = torch.rand(64, 64).half().cuda()
>>> mask = torch.Tensor([0, 0, 1, 1]).tile((64, 16)).cuda().bool()
>>> linear = nn.Linear(64, 64).half().cuda()
>>> linear.weight = nn.Parameter(to_sparse_semi_structured(linear.weight.masked_fill(~mask, 0)))

```


## 稀疏 COO tensor [¶](#sparse-coo-tensors "此标题的固定链接")


 PyTorch 实现了所谓的坐标格式(COOformat)，作为实现稀疏tensor的存储格式之一。在 COO 格式中，指定的元素存储为元素索引和相应值的元组。尤其，



> 
> 
> 
* 指定元素的索引被收集在 
> `indices`
> tensor 
> `(ndim,
> 
> 
> nse)`
> 中，并且元素类型 
> `torch.int64`
> ,
> 
* 相应的值为收集在
> `values`
> tensor
> size
> `(nse,)`
> 且具有任意整数或浮点
> number 元素类型，
> 
> 
> >


 其中“ndim”是tensor的维数，“nse”是指定元素的数量。




!!! note "笔记"

    稀疏 COO tensor的内存消耗至少为 `(ndim * 8 
+ <元素类型大小(以字节为单位)>) * nse` 字节(加上存储其他tensor数据的常量开销)。


 跨步tensor的内存消耗至少为 `product(<tensor shape>) * <size of element type in bytes>` 。


 例如，使用 COO 时，具有 100 000 个非零 32 位浮点数的 10 000 x 10 000 tensor的内存消耗至少为 `(2 * 8 
+ 4) * 100 000 = 2 000 000` 字节使用默认的跨步tensor布局时，tensor布局和 `10 000 * 10 000 * 4 = 400 000 000` 字节。请注意，使用 COO 存储格式可节省 200 倍的内存。


### 构造 [¶](#construction "此标题的永久链接")


 稀疏 COO tensor可以通过向函数 [`torch.sparse_coo_tensor( )`](generated/torch.sparse_coo_tensor.html#torch.sparse_coo_tensor "torch.sparse_coo_tensor").


 假设我们要定义一个稀疏tensor，其中条目 3 位于位置 (0, 2)，条目 4 位于位置 (1, 0)，条目 5 位于位置 (1, 2)。假设未指定的元素具有相同的值，填充值，默认为零。然后我们会写：


```
>>> i = [[0, 1, 1],
 [2, 0, 2]]
>>> v =  [3, 4, 5]
>>> s = torch.sparse_coo_tensor(i, v, (2, 3))
>>> s
tensor(indices=tensor([[0, 1, 1],
 [2, 0, 2]]),
 values=tensor([3, 4, 5]),
 size=(2, 3), nnz=3, layout=torch.sparse_coo)
>>> s.to_dense()
tensor([[0, 0, 3],
 [4, 0, 5]])

```


 请注意，输入“i”不是索引元组列表。如果您想以这种方式编写索引，则应该在将它们传递给稀疏构造函数之前进行转置：


```
>>> i = [[0, 2], [1, 0], [1, 2]]
>>> v =  [3,      4,      5    ]
>>> s = torch.sparse_coo_tensor(list(zip(*i)), v, (2, 3))
>>> # Or another equivalent formulation to get s
>>> s = torch.sparse_coo_tensor(torch.tensor(i).t(), v, (2, 3))
>>> torch.sparse_coo_tensor(i.t(), v, torch.Size([2,3])).to_dense()
tensor([[0, 0, 3],
 [4, 0, 5]])

```


 可以通过仅指定其大小来构造空的稀疏 COO tensor：


```
>>> torch.sparse_coo_tensor(size=(2, 3))
tensor(indices=tensor([], size=(2, 0)),
 values=tensor([], size=(0,)),
 size=(2, 3), nnz=0, layout=torch.sparse_coo)

```


### 稀疏混合 COO tensor [¶](#sparse-hybrid-coo-tensors "永久链接到此标题")


 PyTorch 将具有标量值的稀疏tensor扩展为具有(连续)tensor值的稀疏tensor。这样的tensor称为混合tensor。


 PyTorch 混合 COO tensor通过允许“values”tensor成为多维tensor来扩展稀疏 COO tensor，以便我们有：



> 
> 
> 
* 指定元素的索引被收集在
> `indices`
> tensor
> `(sparse_dims,
> 
> 
> nse)`
> 和元素类型
> `torch.int64`
> ,
> 
* 对应的(tensor)值收集在 
> `values`
> tensor大小 
> `(nse,
> 
> >ense_dims)`
> 中，并且具有任意整数 
> 或浮点数元素类型。
> 
> 
> >




!!! note "笔记"

    我们使用 (M 
+ K) 维tensor来表示 N 维稀疏混合tensor，其中 M 和 K 分别是稀疏维和稠密维的数量，使得 M 
+ K == N 成立。


 假设我们要创建一个 (2 
+ 1) 维tensor，其中条目 [3, 4] 位于位置 (0, 2)，条目 [5, 6] 位于位置 (1, 0)，条目[7, 8 ] 在位置 (1, 2)。我们会写


```
>>> i = [[0, 1, 1],
 [2, 0, 2]]
>>> v =  [[3, 4], [5, 6], [7, 8]]
>>> s = torch.sparse_coo_tensor(i, v, (2, 3, 2))
>>> s
tensor(indices=tensor([[0, 1, 1],
 [2, 0, 2]]),
 values=tensor([[3, 4],
 [5, 6],
 [7, 8]]),
 size=(2, 3, 2), nnz=3, layout=torch.sparse_coo)

```



```
>>> s.to_dense()
tensor([[[0, 0],
 [0, 0],
 [3, 4]],
 [[5, 6],
 [0, 0],
 [7, 8]]])

```


 一般来说，如果 `s` 是一个稀疏 COO tensor并且 `M = s.sparse_dim()` 、 `K = s.dense_dim()` ，那么我们有以下不变量：



> 
> 
> 
* `M
> 
> 
> +
> 
> 
> K
> 
> 
> ==
> 
> 
> len(s.shape)
> 
> 
> ==
> 
> 
> s.ndim`
> 
- tensor的维数
> 是总和稀疏和稠密维度的数量，
> 
* `s.indices().shape
> 
> 
> ==
> 
> 
> (M,
> 
> 
> nse)`
> 
- 显式存储稀疏索引，
> 
* `s. value().shape
> 
> 
> ==
> 
> 
> (nse,)
> 
> 
> +
> 
> 
> s.shape[M
> 
> 
> :
> 
> 
> M
> 
> 
> +
> 
> 
> K]`
> -混合tensor的值>是K维tensor，
> 
* `s.values().layout
> 
> 
> ==
> 
> 
> torch.strided`
> 
- 值存储为>跨步tensor。
> 
> 
> >




!!! note "笔记"

    稠密维度始终遵循稀疏维度，即不支持稠密维度和稀疏维度的混合。


!!! note "笔记"

    为了确保构造的稀疏tensor具有一致的索引、值和大小，可以通过 `check_invariants=True` 关键字参数在每次tensor创建时启用不变检查，或者全局使用 [`torch.sparse.check_sparse_tensor\ _invariants`](generated/torch.sparse.check_sparse_tensor_invariants.html#torch.sparse.check_sparse_tensor_invariants“torch.sparse.check_sparse_tensor_invariants”)上下文管理器实例。默认情况下，稀疏tensor不变量检查被禁用。


### 未合并的稀疏 COO tensor [¶](#uncoalesced-sparse-coo-tensors "永久链接到此标题")


 PyTorch 稀疏 COO tensor格式允许稀疏*未合并*tensor，其中索引中可能存在重复坐标；在这种情况下，解释是该索引处的值是所有重复值条目的总和。例如，可以为同一索引“1”指定多个值“3”和“4”，这会产生一个 1-Duncoalesced tensor：


```
>>> i = [[1, 1]]
>>> v =  [3, 4]
>>> s=torch.sparse_coo_tensor(i, v, (3,))
>>> s
tensor(indices=tensor([[1, 1]]),
 values=tensor( [3, 4]),
 size=(3,), nnz=2, layout=torch.sparse_coo)

```


 而合并过程将使用求和将多值元素累积为单个值：


```
>>> s.coalesce()
tensor(indices=tensor([[1]]),
 values=tensor([7]),
 size=(3,), nnz=1, layout=torch.sparse_coo)

```


 一般来说， [`torch.Tensor.coalesce()`](generated/torch.Tensor.coalesce.html#torch.Tensor.coalesce "torch.Tensor.coalesce") 方法的输出是具有以下属性的稀疏tensor：



* 指定tensor元素的索引是唯一的，
* 索引按字典顺序排序，
* [`torch.Tensor.is_coalesced()`](generated/torch.Tensor.is_coalesced.html#torch.Tensor.is_coalesced " torch.Tensor.is_coalesced") 返回 `True` 。




!!! note "笔记"

    在大多数情况下，您不必关心稀疏tensor是否合并，因为在给定稀疏合并或未合并tensor的情况下，大多数操作的工作方式相同。


 然而，一些操作可以在未合并tensor上更有效地实现，而一些操作则可以在合并tensor上更有效地实现。


 例如，稀疏 COO tensor的加法是通过简单地连接索引和值tensor来实现的：


```
>>> a = torch.sparse_coo_tensor([[1, 1]], [5, 6], (2,))
>>> b = torch.sparse_coo_tensor([[0, 0]], [7, 8], (2,))
>>> a + b
tensor(indices=tensor([[0, 0, 1, 1]]),
 values=tensor([7, 8, 5, 6]),
 size=(2,), nnz=4, layout=torch.sparse_coo)

```


 如果重复执行可能产生重复条目的操作(例如， [`torch.Tensor.add()`](generated/torch.Tensor.add.html#torch.Tensor.add "torch.Tensor.add") )，您应该偶尔合并稀疏tensor以防止它们变得太大。


 另一方面，索引的字典顺序对于实现涉及许多元素选择操作的算法(例如切片或矩阵乘积)可能是有利的。


### 使用稀疏 COO tensor [¶](#working-with-sparse-coo-tensors "永久链接到此标题")


 让我们考虑以下示例：


```
>>> i = [[0, 1, 1],
 [2, 0, 2]]
>>> v =  [[3, 4], [5, 6], [7, 8]]
>>> s = torch.sparse_coo_tensor(i, v, (2, 3, 2))

```


 如上所述，稀疏 COO tensor是一个 [`torch.Tensor`](tensors.html#torch.Tensor "torch.Tensor") 实例，为了将其与使用其他布局的 Tensor 实例区分开来，可以使用 [` torch.Tensor.is_sparse`](generated/torch.Tensor.is_sparse.html#torch.Tensor.is_sparse "torch.Tensor.is_sparse") 或 `torch.Tensor.layout` 属性：


```
>>> isinstance(s, torch.Tensor)
True
>>> s.is_sparse
True
>>> s.layout == torch.sparse_coo
True

```


 稀疏和稠密维度的数量可以使用方法 [`torch.Tensor.sparse_dim()`](generated/torch.Tensor.sparse_dim.html#torch.Tensor.sparse_dim "torch.Tensor.sparse_dim") 和 [分别是 `torch.Tensor.dense_dim()`](generated/torch.Tensor.dense_dim.html#torch.Tensor.dense_dim "torch.Tensor.dense_dim") 。例如：


```
>>> s.sparse_dim(), s.dense_dim()
(2, 1)

```


 如果 s 是稀疏 COO tensor，则可以使用方法 [`torch.Tensor.indices()`](generated/torch.Tensor.indices.html#torch.Tensor.indices "torch.Tensor.indices" 获取其 COO 格式数据。索引") 和 [`torch.Tensor.values()`](generated/torch.Tensor.values.html#torch.Tensor.values "torch.Tensor.values") 。




!!! note "笔记"

    目前，只有在tensor实例合并时才能获取COO格式的数据：


```
>>> s.indices()
RuntimeError: Cannot get indices on an uncoalesced tensor, please call .coalesce() first

```


 要获取未合并tensor的 COO 格式数据，请使用 `torch.Tensor._values()` 和 `torch.Tensor._indices()` ：


```
>>> s._indices()
tensor([[0, 1, 1],
 [2, 0, 2]])

```


!!! warning "警告"

     调用 `torch.Tensor._values()` 将返回一个*分离的*tensor。要跟踪梯度，必须使用 `torch.Tensor.coalesce().values()`。


 构造一个新的稀疏 COO tensor会产生一个未合并的tensor：


```
>>> s.is_coalesced()
False

```


 但可以使用 [`torch.Tensor.coalesce()`](generated/torch.Tensor.coalesce.html#torch.Tensor.coalesce "torch.Tensor.coalesce") 方法构造稀疏 COO tensor的合并副本：


```
>>> s2 = s.coalesce()
>>> s2.indices()
tensor([[0, 1, 1],
 [2, 0, 2]])

```


 当使用未合并的稀疏 COO tensor时，必须考虑未合并数据的相加性质：相同索引的值是求和给出相应tensor元素的值的项。例如，稀疏未合并tensor上的标量乘法可以通过将所有未合并值与标量相乘来实现，因为 `c * (a 
+ b) == c * a 
+ c * b` 成立。然而，任何非线性运算(例如平方根)都不能通过将运算应用于未合并的数据来实现，因为“sqrt(a 
+ b) == sqrt(a) 
+ sqrt(b)”通常不成立。


 仅支持稠密维度对稀疏 COO tensor进行切片(具有正步长)。稀疏维度和密集维度都支持索引：


```
>>> s[1]
tensor(indices=tensor([[0, 2]]),
 values=tensor([[5, 6],
 [7, 8]]),
 size=(3, 2), nnz=2, layout=torch.sparse_coo)
>>> s[1, 0, 1]
tensor(6)
>>> s[1, 0, 1:]
tensor([6])

```


 在 PyTorch 中，稀疏tensor的填充值无法明确指定，通常假设为零。然而，存在可能以不同方式解释填充值的操作。例如，[`torch.sparse.softmax()`](generated/torch.sparse.softmax.html#torch.sparse.softmax "torch.sparse.softmax") 假设填充值为负无穷大来计算 softmax。


## 稀疏压缩tensor [¶](#sparse-compressed-tensors "此标题的固定链接")


 稀疏压缩tensor表示一类稀疏tensor，其共同特征是使用编码来压缩特定维度的索引，该编码能够对稀疏压缩tensor的线性代数核进行某些优化。此编码基于 PyTorch 稀疏压缩tensor在稀疏tensor批次的支持下扩展的压缩稀疏行(CSR)格式，允许多维tensor值，并将稀疏tensor值存储在密集块中。




!!! note "笔记"

    我们使用 (B 
+ M 
+ K) 维tensor来表示 N 维稀疏压缩混合tensor，其中 B、M 和 K 分别是批量、稀疏和密集维度的数量，使得 `B 
+ M 
+ K == N`成立。稀疏压缩tensor的稀疏维度数始终为 2，“M == 2”。


!!! note "笔记"

    如果满足以下不变量，我们就说索引tensor“compressed_indices”使用 CSRencoding：



* `compressed_indices` 是一个连续的跨步 32 或 64 位整数tensor
* `compressed_indices` 形状是 `(*batchsize,compressed_dim_size 
+ 1)`，其中 `compressed_dim_size` 是压缩的数量维度(例如行或列)
* `compressed_indices[..., 0] == 0` 其中 `...` 表示批索引
* `compressed_indices[...,compressed_dim_size] == nse ` 其中 `nse` 是指定元素的数量
* 对于 `i，0 <=compressed_indices[..., i] -compressed_indices[..., i 
- 1] <= plain_dim_size` =1,...,compressed_dim_size` ，其中 `plain_dim_size` 是普通维度的数量(与压缩维度正交，例如列或行)。


 为了确保构造的稀疏tensor具有一致的索引、值和大小，可以通过 `check_invariants=True` 关键字参数在每次tensor创建时启用不变检查，或者全局使用 [`torch.sparse.check_sparse_tensor\ _invariants`](generated/torch.sparse.check_sparse_tensor_invariants.html#torch.sparse.check_sparse_tensor_invariants“torch.sparse.check_sparse_tensor_invariants”)上下文管理器实例。默认情况下，稀疏tensor不变量检查被禁用。


!!! note "笔记"

    将稀疏压缩布局推广到 N 维tensor可能会导致有关指定元素计数的一些混乱。当稀疏压缩tensor包含批次维度时，指定元素的数量将对应于每批次此类元素的数量。当稀疏压缩tensor具有密集维度时，所考虑的元素现在是 K 维数组。此外，对于块稀疏压缩布局，二维块被视为指定的元素。以 3 维块稀疏tensor为例，其中一个批次维度长度为“b”，块形状为“p, q”。如果这个tensor有“n”个指定元素，那么事实上我们每个批次指定了“n”个块。该tensor将具有形状为“(b, n, p, q)”的“值”。对指定元素数量的这种解释来自所有稀疏压缩布局，这些布局源自二维矩阵的压缩。批量维度被视为稀疏矩阵的堆叠，密集维度将元素的含义从简单的标量值更改为具有自己维度的数组。


### 稀疏 CSR tensor [¶](#sparse-csr-tensor "此标题的永久链接")


 与 COO 格式相比，CSR 格式的主要优点是更好地利用存储和更快的计算操作，例如使用 MKL 和 MAGMA 后端的稀疏矩阵向量乘法。


 在最简单的情况下，(0 
+ 2 
+ 0) 维稀疏 CSR tensor由三个一维tensor组成： `crow_indices` 、 `col_indices` 和 `values` ：



> 
> 
> 
* `crow_indices`
> tensor由压缩的 row
> 索引组成。这是一个大小为 
> `nrows
> 
> 
> +
> 
> 
> 1`
> 的一维tensor(
> 行数加 1)。 
> `crow_indices`
> 的最后一个元素是指定元素的数量，
> `nse`
> 。该tensor根据给定行
> 的开始位置对
> `values`
> 和
> `col_indices`
> 中的索引进行编码。tensor中的每个连续数字减去>表示给定行中元素数量的数字。
> 
* `col_indices`
> tensor包含每个>元素的列索引。这是一个大小为 
> `nse`
> 的一维tensor。
> 
* 
> `values`
> tensor包含 CSR tensor
> 元素的值。这是一个大小为 
> `nse`
> 的一维tensor。
> 
> 
> >




!!! note "笔记"

    索引tensor `crow_indices` 和 `col_indices` 的元素类型应为 `torch.int64` (默认)或 `torch.int32` 。如果您想使用支持 MKL 的矩阵运算，请使用 `torch.int32` 。这是由于 pytorch 与 MKL LP64 的默认链接所致，MKL LP64 使用 32 位整数索引。


 一般情况下，(B 
+ 2 
+ K) 维稀疏 CSR tensor由两个 (B 
+ 1) 维索引tensor `crow_indices` 和 `col_indices` 组成，并且为 (1 
+ K) 维`values` tensor使得



> 
> 
> 
* `crow_indices.shape
> 
> 
> ==
> 
> 
> (*batchsize,
> 
> 
> nrows
> 
> 
> +
> 
> 
> 1)`
> 
* `col_indices.shape
> 
> 
> == 
> 
> 
> (*batchsize,
> 
> 
> nse)`
> 
* `values.shape
> 
> 
> ==
> 
> 
> (nse,
> 
> 
> *densesize)`
> 
> 
> >


 而稀疏 CSR tensor的形状为 `(*batchsize, nrows, ncols, *densesize)` ，其中 `len(batchsize) == B` 和 `len(densesize) == K` 。




!!! note "笔记"

    稀疏 CSR tensor的批次是相关的：所有批次中指定元素的数量必须相同。这种有点人为的约束允许有效存储不同 CSR 批次的索引。


!!! note "笔记"

    稀疏和稠密维度的数量可以使用 [`torch.Tensor.sparse_dim()`](generated/torch.Tensor.sparse_dim.html#torch.Tensor.sparse_dim "torch.Tensor.sparse_dim") 和 [ `torch.Tensor.dense_dim()`](generated/torch.Tensor.dense_dim.html#torch.Tensor.dense_dim "torch.Tensor.dense_dim") 方法。批量尺寸可以根据tensorshape计算：`batchsize =tensor.shape[:-tensor.sparse_dim() 
- tensor.dense_dim()]`。


!!! note "笔记"

    稀疏 CSR tensor的内存消耗至少为 `(nrows * 8 
+ (8 
+ <元素类型大小(以字节为单位)
> * prod(densesize)) * nse) * prod(batchsize)` 字节(加上存储其他tensor数据的恒定开销)。


 使用[稀疏 COO 格式说明](#sparse-coo-docs) 的相同示例数据，具有 100 000 个非零 32 位浮点数的 10 000x 10 000 tensor的内存消耗至少为 `(10000 * 8 
+ (8 
+ 4 * 1) * 100 000) * 1 = 1 280 000` 使用 CSR tensor布局时的字节。请注意，与使用 COO 和 strided 格式相比，使用 CSR 存储格式分别节省了 1.6 倍和 310 倍。


#### CSR tensor的构造 [¶](#construction-of-csr-tensors "永久链接到此标题")


 稀疏 CSR tensor可以使用 [`torch.sparse_csr_tensor()`](generated/torch.sparse_csr_tensor.html#torch.sparse_csr_tensor "torch.sparse_csr_tensor") 函数直接构造。用户必须分别提供行索引和列索引以及值tensor，其中必须使用 CSR 压缩编码指定行索引。 “size”参数是可选的，如果不存在，将从“crow_indices”和“col_indices”中推导出来。


```
>>> crow_indices = torch.tensor([0, 2, 4])
>>> col_indices = torch.tensor([0, 1, 0, 1])
>>> values = torch.tensor([1, 2, 3, 4])
>>> csr = torch.sparse_csr_tensor(crow_indices, col_indices, values, dtype=torch.float64)
>>> csr
tensor(crow_indices=tensor([0, 2, 4]),
 col_indices=tensor([0, 1, 0, 1]),
 values=tensor([1., 2., 3., 4.]), size=(2, 2), nnz=4,
 dtype=torch.float64)
>>> csr.to_dense()
tensor([[1., 2.],
 [3., 4.]], dtype=torch.float64)

```


!!! note "笔记"

    推导的“size”中稀疏维度的值是根据“crow_indices”的大小和“col_indices”中的最大索引值计算的。如果列数需要大于推导的“大小”，则必须显式指定“大小”参数。


 从 astrided 或稀疏 COO tensor构造二维稀疏 CSR tensor的最简单方法是使用 [`torch.Tensor.to_sparse_csr()`](generated/torch.Tensor.to_sparse_csr.html#torch.Tensor.to_sparse_csr“torch.Tensor.to_sparse_csr”)方法。 (跨步)tensor中的任何零都将被解释为稀疏tensor中的缺失值：


```
>>> a = torch.tensor([[0, 0, 1, 0], [1, 2, 0, 0], [0, 0, 0, 0]], dtype=torch.float64)
>>> sp = a.to_sparse_csr()
>>> sp
tensor(crow_indices=tensor([0, 1, 3, 3]),
 col_indices=tensor([2, 0, 1]),
 values=tensor([1., 1., 2.]), size=(3, 4), nnz=3, dtype=torch.float64)

```


#### CSR tensor运算 [¶](#csr-tensor-operations "此标题的永久链接")


 稀疏矩阵向量乘法可以使用“tensor.matmul()”方法执行。这是目前 CSR tensor支持的唯一数学运算。


```
>>> vec = torch.randn(4, 1, dtype=torch.float64)
>>> sp.matmul(vec)
tensor([[0.9078],
 [1.3180],
 [0.0000]], dtype=torch.float64)

```


### 稀疏 CSC tensor [¶](#sparse-csc-tensor "此标题的永久链接")


 稀疏 CSC(压缩稀疏列)tensor格式实现了用于存储二维tensor的 CSC 格式，并具有支持批量稀疏 CSC tensor和多维tensor值的扩展。




!!! note "笔记"

    当转置涉及交换稀疏维度时，稀疏 CSC tensor本质上是稀疏 CSR tensor的转置。


 与[稀疏 CSR tensor](#sparse-csr-docs) 类似，稀疏 CSC tensor由三个tensor组成： `ccol_indices` 、 `row_indices` 和 `values` ：



> 
> 
> 
* `ccol_indices`
> tensor由压缩的列索引组成。这是形状为 (B 
+ 1)-D tensor
> `(*batchsize,
> 
> 
> ncols
> 
> 
> +
> 
> 
> 1)`
>.
> 最后一个元素是指定的>元素数量，
> `恩瑟`>。该tensor根据给定列的起始位置对“values”和“row_indices”中的索引进行编码。 
> tensor中的每个连续数字减去它之前的数字
> 表示给定列中的元素数量。
> 
* `row_indices`
> tensor包含每个
> 元素的行索引。这是形状为 (B 
+ 1)-D tensor
> `(*batchsize,
> 
> 
> nse)`
>.
> 
* 
> `values`
> tensor包含 CSC tensor
> 元素的值。这是形状为 (1 
+ K)-D tensor
> `(nse,
> 
> 
> *densesize)`
>.
> 
> 
> >


#### CSC tensor的构造 [¶](#construction-of-csc-tensors "永久链接到此标题")


 稀疏CSCtensor可以使用 [`torch.sparse_csc_tensor()`](generated/torch.sparse_csc_tensor.html#torch.sparse_csc_tensor "torch.sparse_csc_tensor") 函数直接构造。用户必须分别提供行和列索引以及值tensor，其中必须使用 CSR 压缩编码指定列索引。 “size”参数是可选的，如果不存在，将从“row_indices”和“ccol_indices”tensor中推导出来。


```
>>> ccol_indices = torch.tensor([0, 2, 4])
>>> row_indices = torch.tensor([0, 1, 0, 1])
>>> values = torch.tensor([1, 2, 3, 4])
>>> csc = torch.sparse_csc_tensor(ccol_indices, row_indices, values, dtype=torch.float64)
>>> csc
tensor(ccol_indices=tensor([0, 2, 4]),
 row_indices=tensor([0, 1, 0, 1]),
 values=tensor([1., 2., 3., 4.]), size=(2, 2), nnz=4,
 dtype=torch.float64, layout=torch.sparse_csc)
>>> csc.to_dense()
tensor([[1., 3.],
 [2., 4.]], dtype=torch.float64)

```


!!! note "笔记"

    稀疏 CSC tensor构造函数在行索引参数之前具有压缩列索引参数。


 (0 
+ 2 
+ 0) 维稀疏 CSC tensor可以使用 [`torch.Tensor.to_sparse_csc()`](generated/torch.Tensor.to_sparse_csc.html#torch. Tensor.to_sparse_csc“torch.Tensor.to_sparse_csc”)方法。 (跨步)tensor中的任何零都将被解释为稀疏tensor中的缺失值：


```
>>> a = torch.tensor([[0, 0, 1, 0], [1, 2, 0, 0], [0, 0, 0, 0]], dtype=torch.float64)
>>> sp = a.to_sparse_csc()
>>> sp
tensor(ccol_indices=tensor([0, 1, 2, 3, 3]),
 row_indices=tensor([1, 1, 0]),
 values=tensor([1., 2., 1.]), size=(3, 4), nnz=3, dtype=torch.float64,
 layout=torch.sparse_csc)

```


### 稀疏 BSR tensor [¶](#sparse-bsr-tensor "此标题的固定链接")


 稀疏 BSR(块压缩稀疏行)tensor格式实现了用于存储二维tensor的 BSR 格式，并进行了扩展以支持批量稀疏 BSR tensor和作为多维tensor块的值。


 稀疏 BSR tensor由三个tensor组成： `crow_indices` 、 `col_indices` 和 `values` ：



> 
> 
> 
* `crow_indices`
> tensor由压缩的 row
> 索引组成。这是形状为 
> `(*batchsize,
> 
> 
> nrowblocks
> 
> 
> +
> 
> 
> 1)`
> 的 (B 
+ 1)-D tensor。最后一个元素是指定块的数量，
> `nse`
> 。该tensor根据给定列块>的开始位置对>`values`>和>`col_indices`>中的索引进行编码。tensor中的每个连续数字减去>数字，然后它表示给定行中的块数。
> 
* `col_indices`
> tensor包含每个>元素的列块索引。这是形状为 (B 
+ 1)-D tensor
> `(*batchsize,
> 
> 
> nse)`
>.
> 
* 
> `values`
> tensor包含收集为两个的稀疏 BSR tensor
> 元素的值维块。这是形状为 (1 
+ 2 +
> K)-D tensor
> `(nse,
> 
> 
> nrowblocks,
> 
> 
> ncolblocks,
> 
> 
> *densesize)`
>.
> 
> 
> >


#### BSR tensor的构造 [¶](#construction-of-bsr-tensors "永久链接到此标题")


 稀疏 BSR tensor可以使用 [`torch.sparse_bsr_tensor()`](generated/torch.sparse_bsr_tensor.html#torch.sparse_bsr_tensor "torch.sparse_bsr_tensor") 函数直接构造。用户必须分别提供行块索引和值tensor，其中必须使用 CSR 压缩编码指定行块索引。“size”参数是可选的，将从“crow_indices”和“col_indices”推导出来tensor(如果不存在)。


```
>>> crow_indices = torch.tensor([0, 2, 4])
>>> col_indices = torch.tensor([0, 1, 0, 1])
>>> values = torch.tensor([[[0, 1, 2], [6, 7, 8]],
...                        [[3, 4, 5], [9, 10, 11]],
...                        [[12, 13, 14], [18, 19, 20]],
...                        [[15, 16, 17], [21, 22, 23]]])
>>> bsr = torch.sparse_bsr_tensor(crow_indices, col_indices, values, dtype=torch.float64)
>>> bsr
tensor(crow_indices=tensor([0, 2, 4]),
 col_indices=tensor([0, 1, 0, 1]),
 values=tensor([[[ 0., 1., 2.],
 [ 6., 7., 8.]],
 [[ 3., 4., 5.],
 [ 9., 10., 11.]],
 [[12., 13., 14.],
 [18., 19., 20.]],
 [[15., 16., 17.],
 [21., 22., 23.]]]),
 size=(4, 6), nnz=4, dtype=torch.float64, layout=torch.sparse_bsr)
>>> bsr.to_dense()
tensor([[ 0., 1., 2., 3., 4., 5.],
 [ 6., 7., 8., 9., 10., 11.],
 [12., 13., 14., 15., 16., 17.],
 [18., 19., 20., 21., 22., 23.]], dtype=torch.float64)

```


 (0 
+ 2 
+ 0) 维稀疏 BSR tensor可以使用 [`torch.Tensor.to_sparse_bsr()`](generated/torch.Tensor.to_sparse_bsr.html#torch. Tensor.to_sparse_bsr "torch.Tensor.to_sparse_bsr") 方法还需要指定值块大小：


```
>>> dense = torch.tensor([[0, 1, 2, 3, 4, 5],
...                       [6, 7, 8, 9, 10, 11],
...                       [12, 13, 14, 15, 16, 17],
...                       [18, 19, 20, 21, 22, 23]])
>>> bsr = dense.to_sparse_bsr(blocksize=(2, 3))
>>> bsr
tensor(crow_indices=tensor([0, 2, 4]),
 col_indices=tensor([0, 1, 0, 1]),
 values=tensor([[[ 0, 1, 2],
 [ 6, 7, 8]],
 [[ 3, 4, 5],
 [ 9, 10, 11]],
 [[12, 13, 14],
 [18, 19, 20]],
 [[15, 16, 17],
 [21, 22, 23]]]), size=(4, 6), nnz=4,
 layout=torch.sparse_bsr)

```


### 稀疏 BSC tensor [¶](#sparse-bsc-tensor "此标题的永久链接")


 稀疏 BSC(块压缩稀疏列)tensor格式实现了用于存储二维tensor的 BSC 格式，并进行了扩展以支持批量稀疏 BSC tensor和作为多维tensor块的值。


 稀疏 BSC tensor由三个tensor组成： `ccol_indices` 、 `row_indices` 和 `values` ：



> 
> 
> 
* `ccol_indices`
> tensor由压缩的列索引组成。这是形状为 
> `(*batchsize,
> 
> 
> ncolblocks
> 
> 
> +
> 
> 
> 1)`
> 的 (B 
+ 1)-D tensor。最后一个元素是指定块的数量，
> `nse`
> 。该tensor根据给定行块的开始位置对“values”和“row_indices”中的索引进行编码。tensor中的每个连续数字减去>数字，然后它表示给定列中的块数。
> 
* `row_indices`
> tensor包含每个>元素的行块索引。这是形状为 (B 
+ 1)-D tensor
> `(*batchsize,
> 
> 
> nse)`
>.
> 
* 
> `values`
> tensor包含收集成两个的稀疏 BSC tensor
> 元素的值维块。这是形状为 (1 
+ 2 +
> K)-D tensor
> `(nse,
> 
> 
> nrowblocks,
> 
> 
> ncolblocks,
> 
> 
> *densesize)`
>.
> 
> 
> >


#### BSC tensor的构造 [¶](#construction-of-bsc-tensors "永久链接到此标题")


 稀疏 BSC tensor可以使用 [`torch.sparse_bsc_tensor()`](generated/torch.sparse_bsc_tensor.html#torch.sparse_bsc_tensor "torch.sparse_bsc_tensor") 函数直接构造。用户必须分别提供行块索引和值tensor，其中必须使用 CSR 压缩编码指定列块索引。“size”参数是可选的，将从“ccol_indices”和“row_indices”推导出来tensor(如果不存在)。


```
>>> ccol_indices = torch.tensor([0, 2, 4])
>>> row_indices = torch.tensor([0, 1, 0, 1])
>>> values = torch.tensor([[[0, 1, 2], [6, 7, 8]],
...                        [[3, 4, 5], [9, 10, 11]],
...                        [[12, 13, 14], [18, 19, 20]],
...                        [[15, 16, 17], [21, 22, 23]]])
>>> bsc = torch.sparse_bsc_tensor(ccol_indices, row_indices, values, dtype=torch.float64)
>>> bsc
tensor(ccol_indices=tensor([0, 2, 4]),
 row_indices=tensor([0, 1, 0, 1]),
 values=tensor([[[ 0., 1., 2.],
 [ 6., 7., 8.]],
 [[ 3., 4., 5.],
 [ 9., 10., 11.]],
 [[12., 13., 14.],
 [18., 19., 20.]],
 [[15., 16., 17.],
 [21., 22., 23.]]]), size=(4, 6), nnz=4,
 dtype=torch.float64, layout=torch.sparse_bsc)

```


### 用于处理稀疏压缩tensor的工具 [¶](#tools-for-working-with-sparse-compressed-tensors "永久链接到此标题")


 所有稀疏压缩tensor(CSR、CSC、BSR 和 BSC tensor)在概念上非常相似，因为它们的索引数据分为两部分：使用 CSRencoding 的所谓压缩索引，以及与压缩正交的所谓普通索引。指数。这允许这些tensor上的各种工具共享由tensor布局参数化的相同实现。


#### 稀疏压缩tensor的构造 [¶](#construction-of-sparse-compressed-tensors "永久链接到此标题")


 稀疏 CSR、CSC、BSR 和 CSC tensor可以使用 [`torch.sparse_compressed_tensor()`](generated/torch.sparse_compressed_tensor.html#torch.sparse_compressed_tensor "torch.sparse_compressed_tensor") 函数构造，该函数具有与上面讨论的构造函数相同的接口 [`torch.sparse_csr_tensor()`](generated/torch.sparse_csr_tensor.html#torch.sparse_csr_tensor "torch.sparse_csr_tensor") ， [`torch.sparse_csc_tensor() `](generated/torch.sparse_csc_tensor.html#torch.sparse_csc_tensor "torch.sparse_csc_tensor") , [`torch.sparse_bsr_tensor()`](generated/torch.sparse_bsr_tensor.html#torch.sparse_bsr_tensor "torch.sparse_bsr_tensor ") 和 [`torch.sparse_bsc_tensor()`](generated/torch.sparse_bsc_tensor.html#torch.sparse_bsc_tensor "torch.sparse_bsc_tensor") ，但需要额外的 `layout` 参数。以下示例说明了通过为 [`torch.sparse_compressed_tensor()`](generated/torch.sparse_compressed_tensor.html#torch.sparse_compressed_tensor "torch.稀疏_压缩_tensor”)函数：


```
>>> compressed_indices = torch.tensor([0, 2, 4])
>>> plain_indices = torch.tensor([0, 1, 0, 1])
>>> values = torch.tensor([1, 2, 3, 4])
>>> csr = torch.sparse_compressed_tensor(compressed_indices, plain_indices, values, layout=torch.sparse_csr)
>>> csr
tensor(crow_indices=tensor([0, 2, 4]),
 col_indices=tensor([0, 1, 0, 1]),
 values=tensor([1, 2, 3, 4]), size=(2, 2), nnz=4,
 layout=torch.sparse_csr)
>>> csc = torch.sparse_compressed_tensor(compressed_indices, plain_indices, values, layout=torch.sparse_csc)
>>> csc
tensor(ccol_indices=tensor([0, 2, 4]),
 row_indices=tensor([0, 1, 0, 1]),
 values=tensor([1, 2, 3, 4]), size=(2, 2), nnz=4,
 layout=torch.sparse_csc)
>>> (csr.transpose(0, 1).to_dense() == csc.to_dense()).all()
tensor(True)

```


## 支持的操作 [¶](#supported-operations "固定链接到此标题")


### 线性代数运算 [¶](#线性代数操作"此标题的永久链接")


 下表总结了稀疏矩阵上支持的线性代数运算，其中操作数布局可能有所不同。这里“T[layout]”表示具有给定布局的tensor。类似地，“M[layout]”表示矩阵(2-D PyTorch tensor)，“V[layout]”表示向量(1-D PyTorch tensor)。此外，“f”表示标量(浮点或 0-D PyTorch tensor)，“*”是逐元素乘法，“@”是矩阵乘法。


| 	 PyTorch operation	  | 	 Sparse grad?	  | 	 Layout signature	  |
| --- | --- | --- |
| 	[`torch.mv()`](generated/torch.mv.html#torch.mv "torch.mv")	 | 	 no	  | 	`M[sparse_coo]	 		 @	 		 V[strided]	 		 ->	 		 V[strided]`	 |
| 	[`torch.mv()`](generated/torch.mv.html#torch.mv "torch.mv")	 | 	 no	  | 	`M[sparse_csr]	 		 @	 		 V[strided]	 		 ->	 		 V[strided]`	 |
| 	[`torch.matmul()`](generated/torch.matmul.html#torch.matmul "torch.matmul")	 | 	 no	  | 	`M[sparse_coo]	 		 @	 		 M[strided]	 		 ->	 		 M[strided]`	 |
| 	[`torch.matmul()`](generated/torch.matmul.html#torch.matmul "torch.matmul")	 | 	 no	  | 	`M[sparse_csr]	 		 @	 		 M[strided]	 		 ->	 		 M[strided]`	 |
| 	[`torch.matmul()`](generated/torch.matmul.html#torch.matmul "torch.matmul")	 | 	 no	  | 	`M[SparseSemiStructured]	 		 @	 		 M[strided]	 		 ->	 		 M[strided]`	 |
| 	[`torch.matmul()`](generated/torch.matmul.html#torch.matmul "torch.matmul")	 | 	 no	  | 	`M[strided]	 		 @	 		 M[SparseSemiStructured]	 		 ->	 		 M[strided]`	 |
| 	[`torch.mm()`](generated/torch.mm.html#torch.mm "torch.mm")	 | 	 no	  | 	`M[sparse_coo]	 		 @	 		 M[strided]	 		 ->	 		 M[strided]`	 |
| 	[`torch.mm()`](generated/torch.mm.html#torch.mm "torch.mm")	 | 	 no	  | 	`M[SparseSemiStructured]	 		 @	 		 M[strided]	 		 ->	 		 M[strided]`	 |
| 	[`torch.mm()`](generated/torch.mm.html#torch.mm "torch.mm")	 | 	 no	  | 	`M[strided]	 		 @	 		 M[SparseSemiStructured]	 		 ->	 		 M[strided]`	 |
| 	[`torch.sparse.mm()`](generated/torch.sparse.mm.html#torch.sparse.mm "torch.sparse.mm")	 | 	 yes	  | 	`M[sparse_coo]	 		 @	 		 M[strided]	 		 ->	 		 M[strided]`	 |
| 	[`torch.smm()`](generated/torch.smm.html#torch.smm "torch.smm")	 | 	 no	  | 	`M[sparse_coo]	 		 @	 		 M[strided]	 		 ->	 		 M[sparse_coo]`	 |
| 	[`torch.hspmm()`](generated/torch.hspmm.html#torch.hspmm "torch.hspmm")	 | 	 no	  | 	`M[sparse_coo]	 		 @	 		 M[strided]	 		 ->	 		 M[hybrid	 		 sparse_coo]`	 |
| 	[`torch.bmm()`](generated/torch.bmm.html#torch.bmm "torch.bmm")	 | 	 no	  | 	`T[sparse_coo]	 		 @	 		 T[strided]	 		 ->	 		 T[strided]`	 |
| 	[`torch.addmm()`](generated/torch.addmm.html#torch.addmm "torch.addmm")	 | 	 no	  | 	`f	 		 *	 		 M[strided]	 		 +	 		 f	 		 *	 		 (M[sparse_coo]	 		 @	 		 M[strided])	 		 ->	 		 M[strided]`	 |
| 	[`torch.addmm()`](generated/torch.addmm.html#torch.addmm "torch.addmm")	 | 	 no	  | 	`f	 		 *	 		 M[strided]	 		 +	 		 f	 		 *	 		 (M[SparseSemiStructured]	 		 @	 		 M[strided])	 		 ->	 		 M[strided]`	 |
| 	[`torch.addmm()`](generated/torch.addmm.html#torch.addmm "torch.addmm")	 | 	 no	  | 	`f	 		 *	 		 M[strided]	 		 +	 		 f	 		 *	 		 (M[strided]	 		 @	 		 M[SparseSemiStructured])	 		 ->	 		 M[strided]`	 |
| 	[`torch.sparse.addmm()`](generated/torch.sparse.addmm.html#torch.sparse.addmm "torch.sparse.addmm")	 | 	 yes	  | 	`f	 		 *	 		 M[strided]	 		 +	 		 f	 		 *	 		 (M[sparse_coo]	 		 @	 		 M[strided])	 		 ->	 		 M[strided]`	 |
| 	[`torch.sspaddmm()`](generated/torch.sspaddmm.html#torch.sspaddmm "torch.sspaddmm")	 | 	 no	  | 	`f	 		 *	 		 M[sparse_coo]	 		 +	 		 f	 		 *	 		 (M[sparse_coo]	 		 @	 		 M[strided])	 		 ->	 		 M[sparse_coo]`	 |
| 	[`torch.lobpcg()`](generated/torch.lobpcg.html#torch.lobpcg "torch.lobpcg")	 | 	 no	  | 	`GENEIG(M[sparse_coo])	 		 ->	 		 M[strided],	 		 M[strided]`	 |
| 	[`torch.pca_lowrank()`](generated/torch.pca_lowrank.html#torch.pca_lowrank "torch.pca_lowrank")	 | 	 yes	  | 	`PCA(M[sparse_coo])	 		 ->	 		 M[strided],	 		 M[strided],	 		 M[strided]`	 |
| 	[`torch.svd_lowrank()`](generated/torch.svd_lowrank.html#torch.svd_lowrank "torch.svd_lowrank")	 | 	 yes	  | 	`SVD(M[sparse_coo])	 		 ->	 		 M[strided],	 		 M[strided],	 		 M[strided]`	 |


 “稀疏毕业”在哪里？列指示 PyTorch 操作是否支持稀疏矩阵参数的向后。所有 PyTorch 操作，除了 [`torch.smm()`](generated/torch.smm.html#torch.smm "torch.smm") ，都支持相对于 stridedmatrix 参数的向后操作。




!!! note "笔记"

    目前，PyTorch 不支持带有布局签名 `M[strided] @ M[sparse_coo]` 的矩阵乘法。然而，应用程序仍然可以使用矩阵关系“D @ S == (S.t() @ D.t()).t()”来计算它。


### tensor方法和稀疏 [¶](#tensor-methods-and-sparse "永久链接到此标题")


 以下 Tensor 方法与稀疏tensor相关：


|  |  |
| --- | --- |
| 	[`Tensor.is_sparse`](generated/torch.Tensor.is_sparse.html#torch.Tensor.is_sparse "torch.Tensor.is_sparse")	 | 	 Is	 `True`	 if the Tensor uses sparse COO storage layout,	 `False`	 otherwise.	  |
| 	[`Tensor.is_sparse_csr`](generated/torch.Tensor.is_sparse_csr.html#torch.Tensor.is_sparse_csr "torch.Tensor.is_sparse_csr")	 | 	 Is	 `True`	 if the Tensor uses sparse CSR storage layout,	 `False`	 otherwise.	  |
| 	[`Tensor.dense_dim`](generated/torch.Tensor.dense_dim.html#torch.Tensor.dense_dim "torch.Tensor.dense_dim")	 | 	 Return the number of dense dimensions in a	 [sparse tensor](#sparse-docs)	`self`	.	  |
| 	[`Tensor.sparse_dim`](generated/torch.Tensor.sparse_dim.html#torch.Tensor.sparse_dim "torch.Tensor.sparse_dim")	 | 	 Return the number of sparse dimensions in a	 [sparse tensor](#sparse-docs)	`self`	.	  |
| 	[`Tensor.sparse_mask`](generated/torch.Tensor.sparse_mask.html#torch.Tensor.sparse_mask "torch.Tensor.sparse_mask")	 | 	 Returns a new	 [sparse tensor](#sparse-docs)	 with values from a strided tensor	 `self`	 filtered by the indices of the sparse tensor	 `mask`	.	  |
| 	[`Tensor.to_sparse`](generated/torch.Tensor.to_sparse.html#torch.Tensor.to_sparse "torch.Tensor.to_sparse")	 | 	 Returns a sparse copy of the tensor.	  |
| 	[`Tensor.to_sparse_coo`](generated/torch.Tensor.to_sparse_coo.html#torch.Tensor.to_sparse_coo "torch.Tensor.to_sparse_coo")	 | 	 Convert a tensor to	 [coordinate format](#sparse-coo-docs)	.	  |
| 	[`Tensor.to_sparse_csr`](generated/torch.Tensor.to_sparse_csr.html#torch.Tensor.to_sparse_csr "torch.Tensor.to_sparse_csr")	 | 	 Convert a tensor to compressed row storage format (CSR).	  |
| 	[`Tensor.to_sparse_csc`](generated/torch.Tensor.to_sparse_csc.html#torch.Tensor.to_sparse_csc "torch.Tensor.to_sparse_csc")	 | 	 Convert a tensor to compressed column storage (CSC) format.	  |
| 	[`Tensor.to_sparse_bsr`](generated/torch.Tensor.to_sparse_bsr.html#torch.Tensor.to_sparse_bsr "torch.Tensor.to_sparse_bsr")	 | 	 Convert a tensor to a block sparse row (BSR) storage format of given blocksize.	  |
| 	[`Tensor.to_sparse_bsc`](generated/torch.Tensor.to_sparse_bsc.html#torch.Tensor.to_sparse_bsc "torch.Tensor.to_sparse_bsc")	 | 	 Convert a tensor to a block sparse column (BSC) storage format of given blocksize.	  |
| 	[`Tensor.to_dense`](generated/torch.Tensor.to_dense.html#torch.Tensor.to_dense "torch.Tensor.to_dense")	 | 	 Creates a strided copy of	 `self`	 if	 `self`	 is not a strided tensor, otherwise returns	 `self`	.	  |
| 	[`Tensor.values`](generated/torch.Tensor.values.html#torch.Tensor.values "torch.Tensor.values")	 | 	 Return the values tensor of a	 [sparse COO tensor](#sparse-coo-docs)	.	  |


 以下tensor方法特定于稀疏 COO tensor：


|  |  |
| --- | --- |
| 	[`Tensor.coalesce`](generated/torch.Tensor.coalesce.html#torch.Tensor.coalesce "torch.Tensor.coalesce")	 | 	 Returns a coalesced copy of	 `self`	 if	 `self`	 is an	 [uncoalesced tensor](#sparse-uncoalesced-coo-docs)	.	  |
| 	[`Tensor.sparse_resize_`](generated/torch.Tensor.sparse_resize_.html#torch.Tensor.sparse_resize_ "torch.Tensor.sparse_resize_")	 | 	 Resizes	 `self`	[sparse tensor](#sparse-docs)	 to the desired size and the number of sparse and dense dimensions.	  |
| 	[`Tensor.sparse_resize_and_clear_`](generated/torch.Tensor.sparse_resize_and_clear_.html#torch.Tensor.sparse_resize_and_clear_ "torch.Tensor.sparse_resize_and_clear_")	 | 	 Removes all specified elements from a	 [sparse tensor](#sparse-docs)	`self`	 and resizes	 `self`	 to the desired size and the number of sparse and dense dimensions.	  |
| 	[`Tensor.is_coalesced`](generated/torch.Tensor.is_coalesced.html#torch.Tensor.is_coalesced "torch.Tensor.is_coalesced")	 | 	 Returns	 `True`	 if	 `self`	 is a	 [sparse COO tensor](#sparse-coo-docs)	 that is coalesced,	 `False`	 otherwise.	  |
| 	[`Tensor.indices`](generated/torch.Tensor.indices.html#torch.Tensor.indices "torch.Tensor.indices")	 | 	 Return the indices tensor of a	 [sparse COO tensor](#sparse-coo-docs)	.	  |


 以下方法特定于[稀疏 CSR tensor](#sparse-csr-docs) 和 [稀疏 BSR tensor](#sparse-bsr-docs)：


|  |  |
| --- | --- |
| 	[`Tensor.crow_indices`](generated/torch.Tensor.crow_indices.html#torch.Tensor.crow_indices "torch.Tensor.crow_indices")	 | 	 Returns the tensor containing the compressed row indices of the	 `self`	 tensor when	 `self`	 is a sparse CSR tensor of layout	 `sparse_csr`	.	  |
| 	[`Tensor.col_indices`](generated/torch.Tensor.col_indices.html#torch.Tensor.col_indices "torch.Tensor.col_indices")	 | 	 Returns the tensor containing the column indices of the	 `self`	 tensor when	 `self`	 is a sparse CSR tensor of layout	 `sparse_csr`	.	  |


 以下方法特定于[稀疏 CSC tensor](#sparse-csc-docs) 和 [稀疏 BSC tensor](#sparse-bsc-docs) ：


|  |  |
| --- | --- |
| 	[`Tensor.row_indices`](generated/torch.Tensor.row_indices.html#torch.Tensor.row_indices "torch.Tensor.row_indices")	 | 	 |
| 	[`Tensor.ccol_indices`](generated/torch.Tensor.ccol_indices.html#torch.Tensor.ccol_indices "torch.Tensor.ccol_indices")	 | 	 |


 以下tensor方法支持稀疏 COO tensor：


[`add()`](generated/torch.Tensor.add.html#torch.Tensor.add "torch.Tensor.add")[`add_()`](generated/torch.Tensor.add_.html #torch.Tensor.add_ "torch.Tensor.add_")[`addmm()`](generated/torch.Tensor.addmm.html#torch.Tensor.addmm "torch.Tensor.addmm")[`addmm_ ()`](generated/torch.Tensor.addmm_.html#torch.Tensor.addmm_ "torch.Tensor.addmm_")[`any()`](generated/torch.Tensor.any.html#torch.Tensor.任何“torch.Tensor.any”)[`asin()`](generated/torch.Tensor.asin.html#torch.Tensor.asin“torch.Tensor.asin”)[`asin_()`](generated/torch.Tensor.asin_.html#torch.Tensor.asin_ "torch.Tensor.asin_")[`arcsin()`](generated/torch.Tensor.arcsin.html#torch.Tensor.arcsin "torch.Tensor.arcsin")[`arcsin_()`](generated/torch.Tensor.arcsin_.html#torch.Tensor.arcsin_ "torch.Tensor.arcsin_")[`bmm()`](generated/torch.Tensor.bmm.html#torch.Tensor.bmm "torch.Tensor.bmm")[`clone()`](generated/torch.Tensor.clone.html#torch.Tensor.clone "torch.Tensor.clone")[ `deg2rad()`](generated/torch.Tensor.deg2rad.html#torch.Tensor.deg2rad "torch.Tensor.deg2rad")`deg2rad_()`[`detach()`](generated/torch.Tensor.detach.html#torch.Tensor.detach "torch.Tensor.detach")[`detach_()`](generated/torch.Tensor.detach_.html#torch.Tensor.detach_ "torch.Tensor.detach_" )[`dim()`](generated/torch.Tensor.dim.html#torch.Tensor.dim "torch.Tensor.dim")[`div()`](generated/torch.Tensor.div.html# torch.Tensor.div "torch.Tensor.div")[`div_()`](generated/torch.Tensor.div_.html#torch.Tensor.div_ "torch.Tensor.div_")[`floor\ _divide()`](generated/torch.Tensor.floor_divide.html#torch.Tensor.floor_divide "torch.Tensor.floor_divide")[`floor_divide_()`](generated/torch.Tensor.floor_divide_.html #torch.Tensor.floor_divide_ "torch.Tensor.floor_divide_")[`get_device()`](generated/torch.Tensor.get_device.html#torch.Tensor.get_device "torch.Tensor.get_device")[`index _select()`](generated/torch.Tensor.index_select.html#torch.Tensor.index_select "torch.Tensor.index_select")[`isnan()`](generated/torch.Tensor.isnan.html#torch. Tensor.isnan "torch.Tensor.isnan")[`log1p()`](generated/torch.Tensor.log1p.html#torch.Tensor.log1p "torch.Tensor.log1p")[`log1p_()` ](generated/torch.Tensor.log1p_.html#torch.Tensor.log1p_ "torch.Tensor.log1p_")[`mm()`](generated/torch.Tensor.mm.html#torch.Tensor.mm "火炬.Tensor.mm")[`mul()`](generated/torch.Tensor.mul.html#torch.Tensor.mul "torch.Tensor.mul")[`mul_()`](generated/torch.Tensor.mul_.html#torch.Tensor.mul_ "torch.Tensor.mul_")[`mv()`](generated/torch.Tensor.mv.html#torch.Tensor.mv "torch.Tensor.mv" )[`narrow_copy()`](generated/torch.Tensor.narrow_copy.html#torch.Tensor.narrow_copy "torch.Tensor.narrow_copy")[`neg()`](generated/torch.Tensor.neg. html#torch.Tensor.neg "torch.Tensor.neg")[`neg_()`](generated/torch.Tensor.neg_.html#torch.Tensor.neg_ "torch.Tensor.neg_")[`负()`](generated/torch.Tensor. Negative.html#torch.Tensor. Negative "torch.Tensor.负")[`负_()`](generated/torch.Tensor. Negative_.html#torch.Tensor.negative_ "torch.Tensor.negative_")[`numel()`](generated/torch.Tensor.numel.html#torch.Tensor.numel "torch.Tensor.numel")[`rad2deg()`](generated/torch.Tensor.rad2deg.html#torch.Tensor.rad2deg "torch.Tensor.rad2deg")`rad2deg_()`[`resize_as_()`](generated/torch.Tensor.resize_as_.html#torch.Tensor.resize_as_ "torch.Tensor.resize_as_")[`size()`](generated/torch.Tensor.size.html#torch.Tensor.size "torch.Tensor.size")[`pow ()`](generated/torch.Tensor.pow.html#torch.Tensor.pow "torch.Tensor.pow")[`sqrt()`](generated/torch.Tensor.sqrt.html#torch.Tensor. sqrt "torch.Tensor.sqrt")[`square()`](generated/torch.Tensor.square.html#torch.Tensor.square "torch.Tensor.square")[`smm()`](generated/torch.Tensor.smm.html#torch.Tensor.smm "torch.Tensor.smm")[`sspaddmm()`](generated/torch.Tensor.sspaddmm.html#torch.Tensor.sspaddmm "torch.Tensor.sspaddmm ")[`sub()`](generated/torch.Tensor.sub.html#torch.Tensor.sub "torch.Tensor.sub")[`sub_()`](generated/torch.Tensor.sub_.html#torch.Tensor.sub_ "torch.Tensor.sub_")[`t()`](generated/torch.Tensor.t.html#torch.Tensor.t "torch.Tensor.t")[`t _()`](generated/torch.Tensor.t_.html#torch.Tensor.t_ "torch.Tensor.t_")[`transpose()`](generated/torch.Tensor.transpose.html#torch. Tensor.transpose "torch.Tensor.transpose")[`transpose_()`](generated/torch.Tensor.transpose_.html#torch.Tensor.transpose_ "torch.Tensor.transpose_")[`零_( )`](generated/torch.Tensor.zero_.html#torch.Tensor.zero_ "torch.Tensor.zero_")


### 特定于稀疏tensor的 Torch 函数 [¶](#torch-functions-specific-to-sparse-tensors "永久链接到此标题")


|  |  |
| --- | --- |
| [`sparse_coo_tensor`](generated/torch.sparse_coo_tensor.html#torch.sparse_coo_tensor "torch.sparse_coo_tensor") |在给定的 `indices` 处构造一个具有指定值的 [COO(rdinate) 格式的稀疏tensor](#sparse-coo-docs)。 |
| [`sparse_csr_tensor`](generated/torch.sparse_csr_tensor.html#torch.sparse_csr_tensor "torch.sparse_csr_tensor") |在给定的 `crow_indices` 和 `col_indices` 处使用指定值构造一个 [CSR(压缩稀疏行)中的稀疏tensor](#sparse-csr-docs)。 |
| [`sparse_csc_tensor`](generated/torch.sparse_csc_tensor.html#torch.sparse_csc_tensor "torch.sparse_csc_tensor") |在给定的 `ccol_indices` 和 `row_indices` 处使用指定值构造 [CSC(压缩稀疏列)中的稀疏tensor](#sparse-csc-docs)。 |
| [`sparse_bsr_tensor`](generated/torch.sparse_bsr_tensor.html#torch.sparse_bsr_tensor "torch.sparse_bsr_tensor") |在给定的 `crow_indices` 和 `col_indices` 处使用指定的二维块​​构造 [BSR(块压缩稀疏行)中的稀疏tensor](#sparse-bsr-docs)。 |
| [`sparse_bsc_tensor`](generated/torch.sparse_bsc_tensor.html#torch.sparse_bsc_tensor "torch.sparse_bsc_tensor") |在给定的 `ccol_indices` 和 `row_indices` 处使用指定的二维块​​构造 [BSC(块压缩稀疏列)中的稀疏tensor](#sparse-bsc-docs)。 |
| [`sparse_compressed_tensor`](generated/torch.sparse_compressed_tensor.html#torch.sparse_compressed_tensor "torch.sparse_compressed_tensor") |在给定的 `compressed_indices` 和 `plain_indices` 处使用指定值构造 [压缩稀疏格式的稀疏tensor 
- CSR、CSC、BSR 或 BSC -](#sparse-compressed-docs)。 |
| [`sparse.sum`](generated/torch.sparse.sum.html#torch.sparse.sum "torch.sparse.sum") |返回给定维度“dim”中稀疏tensor“input”每行的总和。 |
| [`sparse.addmm`](generated/torch.sparse.addmm.html#torch.sparse.addmm "torch.sparse.addmm") |该函数与前向中的 [`torch.addmm()`](generated/torch.addmm.html#torch.addmm "torch.addmm") 执行完全相同的操作，除了它支持稀疏 COO 矩阵 `mat1` 的后向功能。 |
| [`sparse.sampled_addmm`](generated/torch.sparse.sampled_addmm.html#torch.sparse.sampled_addmm "torch.sparse.sampled_addmm") |在“input”的稀疏模式指定的位置处执行密集矩阵“mat1”和“mat2”的矩阵乘法。 |
| [`sparse.mm`](generated/torch.sparse.mm.html#torch.sparse.mm "torch.sparse.mm") |执行稀疏矩阵 `mat1` |
| 的矩阵乘法


[`sspaddmm`](generated/torch.sspaddmm.html#torch.sspaddmm "torch.sspaddmm") | Matrix 将稀疏tensor“mat1”与稠密tensor“mat2”相乘，然后将稀疏tensor“input”添加到结果中。 |
| [`hspmm`](generated/torch.hspmm.html#torch.hspmm "torch.hspmm") |执行[稀疏 COO 矩阵](#sparse-coo-docs)`mat1` 和跨步矩阵 `mat2` 的矩阵乘法。 |
| [`smm`](generated/torch.smm.html#torch.smm "torch.smm") |执行稀疏矩阵“input”与稠密矩阵“mat”的矩阵乘法。 |
| [`sparse.softmax`](generated/torch.sparse.softmax.html#torch.sparse.softmax "torch.sparse.softmax") |应用 softmax 函数。 |
| [`sparse.log_softmax`](generated/torch.sparse.log_softmax.html#torch.sparse.log_softmax "torch.sparse.log_softmax") |应用 softmax 函数，然后应用对数。 |
| [`sparse.spdiags`](generated/torch.sparse.spdiags.html#torch.sparse.spdiags“torch.sparse.spdiags”) |通过将“对角线”行中的值沿输出的指定对角线放置来创建稀疏二维tensor |


### 其他函数 [¶](#other-functions "永久链接到此标题")


 以下 [`torch`](torch.html#module-torch "torch") 函数支持稀疏tensor：


[`cat()`](generated/torch.cat.html#torch.cat "torch.cat")[`dstack()`](generated/torch.dstack.html#torch.dstack "torch.dstack") [`empty()`](generated/torch.empty.html#torch.empty "torch.empty")[`empty_like()`](generated/torch.empty_like.html#torch.empty_like "torch.empty_like ")[`hstack()`](generated/torch.hstack.html#torch.hstack "torch.hstack")[`index_select()`](generated/torch.index_select.html#torch.index_select "火炬.index_select")[`is_complex()`](generated/torch.is_complex.html#torch.is_complex "torch.is_complex")[`is_floating_point()`](generated/torch.is_floating_point.html #torch.is_floating_point "torch.is_floating_point")[`is_nonzero()`](generated/torch.is_nonzero.html#torch.is_nonzero "torch.is_nonzero")`is_same_size()``is_signed ()`[`is_tensor()`](generated/torch.is_tensor.html#torch.is_tensor "torch.is_tensor")[`lobpcg()`](generated/torch.lobpcg.html#torch.lobpcg " torch.lobpcg")[`mm()`](generated/torch.mm.html#torch.mm "torch.mm")`native_norm()`[`pca_lowrank()`](generated/torch.pca_lowrank.html#torch.pca_lowrank "torch.pca_lowrank")[`select()`](generated/torch.select.html#torch.select "torch.select")[`stack()`](generated/torch.stack.html#torch.stack "torch.stack")[`svd_lowrank()`](generated/torch.svd_lowrank.html#torch.svd_lowrank "torch.svd_lowrank")[`unsqueeze()`](generated/torch.unsqueeze.html#torch.unsqueeze "torch.unsqueeze")[`vstack()`](generated/torch.vstack.html#torch.vstack "torch.vstack")[`zeros()`](generated/torch.zeros.html#torch.zeros "torch.zeros")[`zeros_like()`](generated/torch.zeros_like.html#torch.zeros_like "torch.zeros_like")


 要管理检查稀疏tensor不变量，请参阅：


|  |  |
| --- | --- |
| 	[`sparse.check_sparse_tensor_invariants`](generated/torch.sparse.check_sparse_tensor_invariants.html#torch.sparse.check_sparse_tensor_invariants "torch.sparse.check_sparse_tensor_invariants")	 | 	 A tool to control checking sparse tensor invariants.	  |


 要将稀疏tensor与 [`gradcheck()`](generated/torch.autograd.gradcheck.html#torch.autograd.gradcheck "torch.autograd.gradcheck") 函数一起使用，请参阅：


|  |  |
| --- | --- |
| 	[`sparse.as_sparse_gradcheck`](generated/torch.sparse.as_sparse_gradcheck.html#torch.sparse.as_sparse_gradcheck "torch.sparse.as_sparse_gradcheck")	 | 	 Decorator for torch.autograd.gradcheck or its functools.partial variants that extends the gradcheck function with support to input functions that operate on or/and return sparse tensors.	  |


### 一元函数 [¶](#unary-functions "此标题的永久链接")


 我们的目标是支持所有零保留一元函数。


 如果您发现我们缺少您需要的零保留一元函数，请放心为功能请求打开问题。一如既往，请在打开问题之前先尝试搜索功能。


 以下运算符当前支持稀疏 COO/CSR/CSC/BSR/CSR tensor输入。


[`abs()`](generated/torch.abs.html#torch.abs "torch.abs")[`asin()`](generated/torch.asin.html#torch.asin "torch.asin") [`asinh()`](generated/torch.asinh.html#torch.asinh "torch.asinh")[`atan()`](generated/torch.atan.html#torch.atan "torch.atan") [`atanh()`](generated/torch.atanh.html#torch.atanh "torch.atanh")[`ceil()`](generated/torch.ceil.html#torch.ceil "torch.ceil") [`conj_physical()`](generated/torch.conj_physical.html#torch.conj_physical "torch.conj_physical")[`floor()`](generated/torch.floor.html#torch.floor "torch.floor ")[`log1p()`](generated/torch.log1p.html#torch.log1p "torch.log1p")[`neg()`](generated/torch.neg.html#torch.neg "torch.neg ")[`round()`](generated/torch.round.html#torch.round "torch.round")[`sin()`](generated/torch.sin.html#torch.sin "torch.sin ")[`sinh()`](generated/torch.sinh.html#torch.sinh "torch.sinh")[`sign()`](generated/torch.sign.html#torch.sign "torch.sign ")[`sgn()`](generated/torch.sgn.html#torch.sgn "torch.sgn")[`signbit()`](generated/torch.signbit.html#torch.signbit "torch.signbit ")[`tan()`](generated/torch.tan.html#torch.tan "torch.tan")[`tanh()`](generated/torch.tanh.html#torch.tanh "torch.tanh ")[`trunc()`](generated/torch.trunc.html#torch.trunc "torch.trunc")[`expm1()`](generated/torch.expm1.html#torch.expm1 "torch.expm1 ")[`sqrt()`](generated/torch.sqrt.html#torch.sqrt "torch.sqrt")[`angle()`](generated/torch.angle.html#torch.angle "火炬.angle ")[`isinf()`](generated/torch.isinf.html#torch.isinf "torch.isinf")[`isposinf()`](generated/torch.isposinf.html#torch.isposinf "torch.isposinf ")[`isneginf()`](generated/torch.isneginf.html#torch.isneginf "torch.isneginf")[`isnan()`](generated/torch.isnan.html#torch.isnan "torch.isnan ")[`erf()`](generated/torch.erf.html#torch.erf "torch.erf")[`erfinv()`](generated/torch.erfinv.html#torch.erfinv "torch.erfinv ”)