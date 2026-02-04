# 复数 [¶](#complex-numbers "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/complex_numbers>
>
> 原始地址：<https://pytorch.org/docs/stable/complex_numbers.html>


 复数是可以用以下形式表示的数字


 a+bj


 a+bj


 a
 



 +
 




 bj
 


 ，其中a和b是实数，*j*称为虚数单位，满足方程




 j
 

 2
 


 = − 1


 j^2 = -1



 j
 



 2
 




 =
 




 −
 

 1
 


 。复数经常出现在数学和工程中，尤其是在信号处理等主题中。传统上，许多用户和库(例如 TorchAudio)通过用形状表示浮点tensor中的数据来处理复数


 (..., 2)


 (..., 2)


 (... ,



 2
 

 )
 


 其中最后一个维度包含实部和虚部值。


 复杂数据类型的tensor在处理复数时提供更自然的用户体验。对复tensor的运算(例如， [`torch.mv()`](generated/torch.mv.html#torch.mv "torch.mv") 、 [`torch.matmul()`](generated/torch.matmul. html#torch.matmul "torch.matmul") ) 可能比模仿它们的浮点tensor上的操作更快、更高效。 PyTorch 中涉及复数的运算经过优化，可使用矢量化汇编指令和专用内核(例如 LAPACK、cuBlas)。




!!! note "笔记"

    [torch.fft 模块](https://pytorch.org/docs/stable/fft.html#torch-fft) 中的光谱运算支持原生复tensor。


!!! warning "警告"

     复杂tensor是测试版功能，可能会发生变化。


## 创建复杂tensor [¶](#creating-complex-tensors "此标题的永久链接")


 我们支持两种复杂的数据类型：torch.cfloat 和 torch.cdouble



```
>>> x = torch.randn(2,2, dtype=torch.cfloat)
>>> x
tensor([[-0.4621-0.0303j, -0.2438-0.5874j],
 [ 0.7706+0.1421j, 1.2110+0.1918j]])

```


!!! note "笔记"

    复数tensor的默认数据类型由默认浮点数据类型确定。如果默认浮点数据类型为 torch.float64 ，则推断复数的数据类型为 torch.complex128 ，否则假定它们的数据类型为 torch.complex64 。


 除 [`torch.linspace()`](generated/torch.linspace.html#torch.linspace "torch.linspace") 、 [`torch.logspace()`](generated/torch.logspace.html 之外的所有工厂函数复杂tensor支持 #torch.logspace "torch.logspace") 和 [`torch.arange()`](generated/torch.arange.html#torch.arange "torch.arange") 。


## 从旧表示的过渡 [¶](#transition-from-the-old-representation "永久链接到此标题")


 目前解决缺乏具有真实形状tensor的复杂tensor问题的用户


 (..., 2)


 (..., 2)


 (... ,



 2
 

 )
 


 可以使用 [`torch.view_as_complex()`](generated/torch.view_as_complex.html#torch.view_as_complex "torch.view_as_complex") 和 [`torch.view\ _as_real()`](generated/torch.view_as_real.html#torch.view_as_real "torch.view_as_real") 。请注意，这些函数不执行任何复制并返回输入tensor的视图。


```
>>> x = torch.randn(3, 2)
>>> x
tensor([[ 0.6125, -0.1681],
 [-0.3773, 1.3487],
 [-0.0861, -0.7981]])
>>> y = torch.view_as_complex(x)
>>> y
tensor([ 0.6125-0.1681j, -0.3773+1.3487j, -0.0861-0.7981j])
>>> torch.view_as_real(y)
tensor([[ 0.6125, -0.1681],
 [-0.3773, 1.3487],
 [-0.0861, -0.7981]])

```


## 访问 real 和 imag [¶](#accessing-real-and-imag "此标题的固定链接")


 可以使用 `real` 和 `imag` 来访问复tensor的实部和虚部值。




!!! note "笔记"

    访问 real 和 imag 属性不会分配任何内存，对 real 和 imag tensor的就地更新将更新原始的复tensor。此外，返回的实tensor和图像tensor不连续。



```
>>> y.real
tensor([ 0.6125, -0.3773, -0.0861])
>>> y.imag
tensor([-0.1681, 1.3487, -0.7981])

>>> y.real.mul_(2)
tensor([ 1.2250, -0.7546, -0.1722])
>>> y
tensor([ 1.2250-0.1681j, -0.7546+1.3487j, -0.1722-0.7981j])
>>> y.real.stride()
(2,)

```


## 角度和绝对值 [¶](#angle-and-abs "此标题的永久链接")


 复tensor的角度和绝对值可以使用 [`torch.angle()`](generated/torch.angle.html#torch.angle "torch.angle") 和 [`torch.abs()`] 计算(generated/torch.abs.html#torch.abs“torch.abs”)。


```
>>> x1=torch.tensor([3j, 4+4j])
>>> x1.abs()
tensor([3.0000, 5.6569])
>>> x1.angle()
tensor([1.5708, 0.7854])

```


## 线性代数 [¶](#linear-algebra "此标题的永久链接")


 许多线性代数运算，例如 [`torch.matmul()`](generated/torch.matmul.html#torch.matmul "torch.matmul") 、 [`torch.svd()`](generated/torch.svd. html#torch.svd "torch.svd") 、 `torch.solve()` 等，支持复数。如果您想请求我们目前不支持的操作，请[搜索](https:///github.com/pytorch/pytorch/issues?q=is%3Aissue+is%3Aopen+complex) 如果问题已提交，如果没有，[提交一个](https://github.com/pytorch/pytorch /问题/新/选择)。


## 序列化 [¶](#serialization "此标题的永久链接")


 复杂的tensor可以被序列化，从而允许数据保存为复杂的值。


```
>>> torch.save(y, 'complex_tensor.pt')
>>> torch.load('complex_tensor.pt')
tensor([ 0.6125-0.1681j, -0.3773+1.3487j, -0.0861-0.7981j])

```


## Autograd [¶](#autograd "此标题的永久链接")


 PyTorch 支持复杂tensor的 autograd。计算的梯度是共轭维廷格导数，其负数正是梯度下降算法中使用的最速下降方向。因此，所有现有的优化器都可以开箱即用，具有复杂的参数。有关更多详细信息，请查看注释 [复数 Autograd](notes/autograd.html#complex-autograd-doc) 。


 我们不完全支持以下子系统：



* 量化
* JIT
* 稀疏tensor
* 分布式


 如果其中任何一个对您的用例有帮助，请[搜索](https://github.com/pytorch/pytorch/issues?q=is%3Aissue+is%3Aopen+complex)(如果问题已提交)并且如果不，[文件一](https://github.com/pytorch/pytorch/issues/new/choose)。