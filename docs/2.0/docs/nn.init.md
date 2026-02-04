# torch.nn.init [¶](#torch-nn-init "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/nn.init>
>
> 原始地址：<https://pytorch.org/docs/stable/nn.init.html>


!!! warning "警告"

     该模块中的所有函数都旨在用于初始化神经网络参数，因此它们都在 [`torch.no_grad()`](generated/torch.no_grad.html#torch.no_grad "torch.no_grad") 中运行模式，并且 autograd 不会考虑。


 火炬.nn.init。


 计算增益


 ( *非线性
* , *参数



 =
 


 无
* ) [[source]](_modules/torch/nn/init.html#calculate_gain)[¶](#torch.nn.init.calculate_gain "此定义的永久链接")


 返回给定非线性函数的建议增益值。值如下：


| 	 nonlinearity	  | 	 gain	  |
| --- | --- |
| 	 Linear /Identity	  |




 1
 


 1
 


 1
 


 |
|转化次数{1,2,3}D |




 1
 


 1
 


 1
 


 |
|乙状结肠 |




 1
 


 1
 


 1
 


 |
|腥味|


 5
 

 3
 


 rac{5}{3}


 3
 



 5
 


 ​
 


 |
| ReLU |


 2
 


 \sqrt{2}




 2
 


 ​
 


 |
|泄漏 Relu |



 2
 


 1
 

 +
 


 负斜率 2


 \sqrt{rac{2}{1 
+ 	ext{负\_斜率}^2}}




 1
 

 +
 


 负_斜率




 2
 




 2
 


 ​
 


 ​
 


 |
|村庄|


 3
 

 4
 


 \压裂{3}{4}


 4
 



 3
 


 ​
 




 |


!!! warning "警告"

     为了实现[自归一化神经网络](https://papers.nips.cc/paper/2017/hash/5d44ee6f2c3f71b73125876103c8f6c4-Abstract.html)，你应该使用 `nonlinearity='linear'` 而不是 `nonlinearity= 'selu'` 。这为初始权重提供了`1 /N` 的方差，这是在前向传播中引入稳定固定点所必需的。相反，`SELU` 的默认增益牺牲了归一化效果以获得更稳定的梯度呈矩形层状流动。


 参数 
* **非线性** – 非线性函数( nn.function 名称)
* **param** – 非线性函数的可选参数


 例子


```
>>> gain = nn.init.calculate_gain('leaky_relu', 0.2)  # leaky_relu with negative_slope=0.2

```


 火炬.nn.init。


 制服_


 ( *tensor
* , *a



 =
 


 0.0*,*b



 =
 


 1.0
* ) [[source]](_modules/torch/nn/init.html#uniform_)[¶](#torch.nn.init.uniform_"此定义的永久链接")


 用从均匀分布中提取的值填充输入tensor


 U ( a , b )


 \mathcal{U}(a, b)


 U(一，



 b
 

 )
 




.
 


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个 n 维 torch.Tensor
* **a** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(Python v3.12)") ) – 均匀分布的下界
* **b** ( [*float*]( https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 均匀分布的上限


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.uniform_(w)

```


 火炬.nn.init。


 普通的_


 (*tensor*，*平均值



 =
 


 0.0
* , *标准



 =
 


 1.0
* ) [[source]](_modules/torch/nn/init.html#normal_)[¶](#torch.nn.init.normal_"此定义的永久链接")


 用从正态分布中提取的值填充输入tensor


 N(均值，


 标准2


 )
 


 \mathcal{N}(	ext{mean}, 	ext{std}^2)


 N
 

 (
 


 mean
 


 ,
 


 std
 




 2
 


 )
 




.
 


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个 n 维 torch.Tensor
* **mean** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(Python v3.12)") ) – 正态分布的平均值
* **std** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(Python v3.12)") ) – 正态分布的标准差


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.normal_(w)

```


 火炬.nn.init。


 持续的_


 ( *tensor
* , *val
* ) [[source]](_modules/torch/nn/init.html#constant_)[¶](#torch.nn.init.constant_ "此定义的永久链接")


 用值填充输入 Tensor



 val
 


 	ext{选择}



 val
 


.
 


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个 n 维 torch.Tensor
* **val** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 用于填充tensor的值


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.constant_(w, 0.3)

```


 火炬.nn.init。


 那些_


 ( *tensor
* ) [[source]](_modules/torch/nn/init.html#ones_)[¶](#torch.nn.init.ones_ "此定义的永久链接")


 用标量值 1 填充输入 Tensor。


 Parameters


**tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – n 维 torch.Tensor


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.ones_(w)

```


 火炬.nn.init。


 零_


 ( *tensor
* ) [[source]](_modules/torch/nn/init.html#zeros_)[¶](#torch.nn.init.zeros_ "此定义的永久链接")


 用标量值 0 填充输入tensor。


 Parameters


**tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – n 维 torch.Tensor


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.zeros_(w)

```


 火炬.nn.init。



 eye\_
 


 ( *tensor
* ) [[source]](_modules/torch/nn/init.html#eye_)[¶](#torch.nn.init.eye_ "此定义的永久链接")


 用单位矩阵填充二维输入tensor。保留线性层中输入的身份，其中保留尽可能多的输入。


 Parameters


**tensor** – 二维火炬。Tensor


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.eye_(w)

```


 火炬.nn.init。


 狄拉克_


 ( *tensor
* , *组



 =
 


 1
* ) [[source]](_modules/torch/nn/init.html#dirac_)[¶](#torch.nn.init.dirac_"此定义的永久链接")


 使用 Diracdelta 函数填充 {3, 4, 5} 维输入 Tensor。保留卷积层中输入的身份，其中保留尽可能多的输入通道。如果组>1，每组通道保留身份


 参数 
* **tensor** – {3, 4, 5} 维 torch.Tensor
* **groups** ( [*int*](https://docs.python.org/3/library/functions. html#int "(in Python v3.12)")*,
* *可选
* ) – 转换层中的组数(默认值：1)


 例子


```
>>> w = torch.empty(3, 16, 5, 5)
>>> nn.init.dirac_(w)
>>> w = torch.empty(3, 24, 5, 5)
>>> nn.init.dirac_(w, 3)

```


 火炬.nn.init。


 泽维尔_制服_


 (*tensor*，*增益



 =
 


 1.0
* ) [[source]](_modules/torch/nn/init.html#xavier_uniform_)[¶](#torch.nn.init.xavier_uniform_"此定义的永久链接")


 根据理解训练深度前馈神经网络的难度 
- Glorot, X. & Bengio, Y. (2010) 中描述的方法，使用均匀分布，用值填充输入tensor。得到的tensor将具有从以下样本采样的值


 U ( − a , a )


 \mathcal{U}(-a, a)


 U ( − a ,



 a
 

 )
 




 where
 


 a = 增益 ×



 6
 


 来自_in 
+ 来自_out


 a = 	ext{增益} 	imes \sqrt{rac{6}{	ext{fan\_in} 
+ 	ext{fan\_out}}}


 a
 



 =
 


 gain
 




 ×
 


 来自_in




 +
 


 扇出


 6
 




 ​
 


 ​
 


 也称为 Glorot 初始化。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个 n 维 torch.Tensor
* **增益** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 可选的缩放因子


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.xavier_uniform_(w, gain=nn.init.calculate_gain('relu'))

```


 火炬.nn.init。


 泽维尔_正常_


 (*tensor*，*增益



 =
 


 1.0
* ) [[source]](_modules/torch/nn/init.html#xavier_normal_)[¶](#torch.nn.init.xavier_normal_"此定义的永久链接")


 根据理解训练深度前馈神经网络的难度 
- Glorot, X. & Bengio, Y. (2010) 中描述的方法，使用正态分布，用值填充输入tensor。得到的tensor将具有从以下样本采样的值


 N ( 0 ,


 标准2


 )
 


 \mathcal{N}(0, 	ext{std}^2)


 N ( 0 ,


 std
 




 2
 


 )
 




 where
 


 标准差 = 增益 ×



 2
 


 来自_in 
+ 来自_out


 	ext{std} = 	ext{增益} 	imes \sqrt{rac{2}{	ext{fan\_in} 
+ 	ext{fan\_out}}}



 std
 




 =
 


 gain
 




 ×
 


 来自_in




 +
 


 扇出


 2
 




 ​
 


 ​
 


 也称为 Glorot 初始化。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个 n 维 torch.Tensor
* **增益** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 可选的缩放因子


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.xavier_normal_(w)

```


 火炬.nn.init。


 kaiming_uniform_


 ( *tensor
* , *a



 =
 


 0
* , *模式



 =
 


 'fan_in'
* , *非线性



 =
 


 'leaky_relu'
* ) [[source]](_modules/torch/nn/init.html#kaiming_uniform_)[¶](#torch.nn.init.kaiming_uniform_ "此定义的永久链接")


 根据深入研究整流器中描述的方法用值填充输入tensor：在 ImageNet 分类上超越人类水平的性能 
- He, K. 等人。 (2015)，使用均匀分布。得到的tensor将具有从以下样本采样的值


 U (− 束缚 , 束缚 )


 \mathcal{U}(-	ext{绑定}, 	ext{绑定})


 U(-


 bound
 


 ,
 




 bound
 


 )
 




 where
 


 界限=增益×


 3_模式


 	ext{bound} = 	ext{增益} 	imes \sqrt{rac{3}{	ext{fan\_mode}}}



 bound
 




 =
 


 gain
 




 ×
 


 来自_模式


 3
 




 ​
 


 ​
 


 也称为 He 初始化。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个 n 维 torch.Tensor
* **a** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 该层之后使用的整流器的负斜率(仅与 `'leaky_relu'` 一起使用)
* **mode** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要么 `'fan\ _in'` (默认)或 ''fan_out'` 。选择“fan_in”可以保留前向传播中权重方差的大小。选择“fan_out”会保留向后传递中的幅度。
* **非线性** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 非线性函数( nn.function name)，建议仅与 `'relu'` 或 `'leaky_relu'` (默认)一起使用。


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.kaiming_uniform_(w, mode='fan_in', nonlinearity='relu')

```


 火炬.nn.init。


 kaiming_normal_


 ( *tensor
* , *a



 =
 


 0
* , *模式



 =
 


 'fan_in'
* , *非线性



 =
 


 'leaky_relu'
* ) [[source]](_modules/torch/nn/init.html#kaiming_normal_)[¶](#torch.nn.init.kaiming_normal_ "此定义的永久链接")


 根据深入研究整流器中描述的方法用值填充输入tensor：在 ImageNet 分类上超越人类水平的性能 
- He, K. 等人。 (2015)，使用非正态分布。得到的tensor将具有从以下样本采样的值


 N ( 0 ,


 标准2


 )
 


 \mathcal{N}(0, 	ext{std}^2)


 N ( 0 ,


 std
 




 2
 


 )
 




 where
 


 标准=


 gain
 


 来自_模式


 	ext{std} = rac{	ext{增益}}{\sqrt{	ext{fan\_mode}}}



 std
 




 =
 


 来自_模式



 ​
 


 gain
 


 ​
 


 也称为 He 初始化。


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个 n 维 torch.Tensor
* **a** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 该层之后使用的整流器的负斜率(仅与 `'leaky_relu'` 一起使用)
* **mode** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要么 `'fan\ _in'` (默认)或 ''fan_out'` 。选择“fan_in”可以保留前向传播中权重方差的大小。选择“fan_out”会保留向后传递中的幅度。
* **非线性** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 非线性函数( nn.function name)，建议仅与 `'relu'` 或 `'leaky_relu'` (默认)一起使用。


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.kaiming_normal_(w, mode='fan_out', nonlinearity='relu')

```


 火炬.nn.init。


 后备箱_正常_


 (*tensor*，*平均值



 =
 


 0.0
* , *标准



 =
 


 1.0*,*a



 =
 


 -2.0*,*b



 =
 


 2.0
* , *发电机



 =
 


 无
* ) [[source]](_modules/torch/nn/init.html#trunc_normal_)[¶](#torch.nn.init.trunc_normal_ "此定义的永久链接")


 使用从截断正态分布中提取的值填充输入tensor。这些值有效地从正态分布中得出


 N(均值，


 标准2


 )
 


 \mathcal{N}(	ext{mean}, 	ext{std}^2)


 N
 

 (
 


 mean
 


 ,
 


 std
 




 2
 


 )
 


 与外界的价值观


 [一，二]


 [一、二]


 [  A  ，



 b
 

 ]
 


 重新绘制，直到它们在范围内。用于生成随机值的方法在以下情况下效果最佳


 a ≤ 平均值 ≤ b


 a \leq 	ext{mean} \leq b


 a
 



 ≤
 


 mean
 




 ≤
 




 b
 




.
 


 参数 
* **tensor** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 一个 n 维 torch.Tensor
* **mean** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(Python v3.12)") ) – 正态分布的平均值
* **std** ( [*float*](https ://docs.python.org/3/library/functions.html#float "(Python v3.12)") ) – 正态分布的标准差
* **a** ( [*float*]( https://docs.python.org/3/library/functions.html#float "(Python v3.12)") ) – 最小截止值
* **b** ( [*float*](https://docs.python.org/3/library/functions.html#float "(in Python v3.12)") ) – 最大截止值


 Return type


[*tensor*](tensors.html#torch.Tensor "torch.Tensor")


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.trunc_normal_(w)

```


 火炬.nn.init。


 正交_


 (*tensor*，*增益



 =
 


 1
* ) [[source]](_modules/torch/nn/init.html#orthogonal_)[¶](#torch.nn.init.orthogonal_"此定义的永久链接")


 用(半)正交矩阵填充输入tensor，如深度线性神经网络中学习的非线性动态的精确解决方案中所述 
- Saxe, A. 等人。 (2013)。输入tensor必须至少有 2 个维度，对于超过 2 个维度的tensor，尾部维度将被展平。


 参数 
* **tensor** – 一个 n 维 torch.Tensor ，其中


 n≥2


 2


 n
 



 ≥
 


 2
* **增益** – 可选比例因子


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.orthogonal_(w)

```


 火炬.nn.init。


 疏_


 (*tensor*，*稀疏性*，*标准



 =
 


 0.01
* ) [[source]](_modules/torch/nn/init.html#sparse_)[¶](#torch.nn.init.sparse_"此定义的永久链接")


 将 2D 输入tensor填充为稀疏矩阵，其中将从正态分布中提取非零元素


 N ( 0 , 0.01 )


 \mathcal{N}(0, 0.01)


 N ( 0 ,


 0.01)


 ，如《通过无 Hessian 优化进行深度学习》中所述 
- Martens, J. (2010)。


 参数 
* **tensor** – 一个 n 维 torch.Tensor
* **sparsity** – 每列中要设置为零的元素分数
* **std** – 用于计算的正态分布的标准偏差生成非零值


 例子


```
>>> w = torch.empty(3, 5)
>>> nn.init.sparse_(w, sparsity=0.1)

```