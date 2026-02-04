# torch.nn [¶](#module-torch.nn "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/nn>
>
> 原始地址：<https://pytorch.org/docs/stable/nn.html>


 这些是图表的基本构建块：


 火炬网



* [容器](#containers)
* [卷积层](#卷积层)
* [池化层](#pooling-layers)
* [填充层](#padding-layers)
* [非线性激活(加权和) ，非线性)](#非线性激活加权和非线性)
* [非线性激活(其他)](#非线性激活其他)
* [归一化层](#归一化层)
* [循环层](#recurrent-layers)
* [变换器层](#transformer-layers)
* [线性层](#线性层)
* [丢弃层](#dropout-layers)
* [稀疏层](#稀疏层)
* [距离函数](#distance-functions)
* [损失函数](#loss-functions)
* [视觉层](#vision-layers)
* [随机层](#shuffle-layers)
* [ DataParallel 层(多 GPU，分布式)](#module-torch.nn.parallel)
* [实用程序](#module-torch.nn.utils)
* [量化函数](#quantized-functions)
* [延迟模块初始化](#lazy-modules-初始化)


|  |  |
| --- | --- |
| [`参数`](generated/torch.nn.parameter.Parameter.html#torch.nn.parameter.Parameter "torch.nn.parameter.Parameter") |一种被视为模块参数的tensor。 |
| [`UninitializedParameter`](generated/torch.nn.parameter.UninitializedParameter.html#torch.nn.parameter.UninitializedParameter "torch.nn.parameter.UninitializedParameter") |未初始化的参数。 |
| [`UninitializedBuffer`](generated/torch.nn.parameter.UninitializedBuffer.html#torch.nn.parameter.UninitializedBuffer "torch.nn.parameter.UninitializedBuffer") |未初始化的缓冲区。 |


## [Containers](#id1) [¶](#containers "此标题的永久链接")


|  |  |
| --- | --- |
| [`模块`](generated/torch.nn.Module.html#torch.nn.Module“torch.nn.Module”)|所有神经网络模块的基类。 |
| [`顺序`](generated/torch.nn.Sequential.html#torch.nn.Sequential "torch.nn.Sequential") |一个顺序容器。 |
| [`ModuleList`](generated/torch.nn.ModuleList.html#torch.nn.ModuleList“torch.nn.ModuleList”) |将子模块保存在列表中。 |
| [`ModuleDict`](generated/torch.nn.ModuleDict.html#torch.nn.ModuleDict "torch.nn.ModuleDict") |将子模块保存在字典中。 |
| [`ParameterList`](generated/torch.nn.ParameterList.html#torch.nn.ParameterList "torch.nn.ParameterList") |将参数保存在列表中。 |
| [`ParameterDict`](generated/torch.nn.ParameterDict.html#torch.nn.ParameterDict "torch.nn.ParameterDict") |将参数保存在字典中。 |


 模块的全局挂钩


|  |  |
| --- | --- |
| [`register_module_forward_pre_hook`](generated/torch.nn.modules.module.register_module_forward_pre_hook.html#torch.nn.modules.module.register_module_forward_pre_hook "torch.nn.modules.module.register_module_forward_pre_hook") |注册所有模块共用的前向预挂钩。 |
| [`register_module_forward_hook`](generated/torch.nn.modules.module.register_module_forward_hook.html#torch.nn.modules.module.register_module_forward_hook "torch.nn.modules.module.register_module_forward_hook") |为所有模块注册一个全局前向钩子 |
| [`register_module_backward_hook`](generated/torch.nn.modules.module.register_module_backward_hook.html#torch.nn.modules.module.register_module_backward_hook“torch.nn.modules.module.register_module_backward_hook”) |注册所有模块共用的向后挂钩。 |
| [`register_module_full_backward_pre_hook`](generated/torch.nn.modules.module.register_module_full_backward_pre_hook.html#torch.nn.modules.module.register_module_full_backward_pre_hook "torch.nn.modules.module.register_module_full_backward_pre_hook" ) |注册所有模块共用的向后预挂钩。 |
| [`register_module_full_backward_hook`](generated/torch.nn.modules.module.register_module_full_backward_hook.html#torch.nn.modules.module.register_module_full_backward_hook“torch.nn.modules.module.register_module_full_backward_hook”) |注册所有模块共用的向后挂钩。 |
| [`register_module_buffer_registration_hook`](generated/torch.nn.modules.module.register_module_buffer_registration_hook.html#torch.nn.modules.module.register_module_buffer_registration_hook“torch.nn.modules.module.register_module_buffer_registration_hook”) |注册所有模块共用的缓冲区注册挂钩。 |
| [`register_module_module_registration_hook`](generated/torch.nn.modules.module.register_module_module_registration_hook.html#torch.nn.modules.module.register_module_module_registration_hook“torch.nn.modules.module.register_module_module_registration_hook”) |注册所有模块通用的模块注册挂钩。 |
| [`register_module_parameter_registration_hook`](generated/torch.nn.modules.module.register_module_parameter_registration_hook.html#torch.nn.modules.module.register_module_parameter_registration_hook“torch.nn.modules.module.register_module_parameter_registration_hook”) |注册所有模块通用的参数注册挂钩。 |


## [卷积层](#id1) [¶](#卷积层"此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.Conv1d`](generated/torch.nn.Conv1d.html#torch.nn.Conv1d "torch.nn.Conv1d")	 | 	 Applies a 1D convolution over an input signal composed of several input planes.	  |
| 	[`nn.Conv2d`](generated/torch.nn.Conv2d.html#torch.nn.Conv2d "torch.nn.Conv2d")	 | 	 Applies a 2D convolution over an input signal composed of several input planes.	  |
| 	[`nn.Conv3d`](generated/torch.nn.Conv3d.html#torch.nn.Conv3d "torch.nn.Conv3d")	 | 	 Applies a 3D convolution over an input signal composed of several input planes.	  |
| 	[`nn.ConvTranspose1d`](generated/torch.nn.ConvTranspose1d.html#torch.nn.ConvTranspose1d "torch.nn.ConvTranspose1d")	 | 	 Applies a 1D transposed convolution operator over an input image composed of several input planes.	  |
| 	[`nn.ConvTranspose2d`](generated/torch.nn.ConvTranspose2d.html#torch.nn.ConvTranspose2d "torch.nn.ConvTranspose2d")	 | 	 Applies a 2D transposed convolution operator over an input image composed of several input planes.	  |
| 	[`nn.ConvTranspose3d`](generated/torch.nn.ConvTranspose3d.html#torch.nn.ConvTranspose3d "torch.nn.ConvTranspose3d")	 | 	 Applies a 3D transposed convolution operator over an input image composed of several input planes.	  |
| 	[`nn.LazyConv1d`](generated/torch.nn.LazyConv1d.html#torch.nn.LazyConv1d "torch.nn.LazyConv1d")	 | 	 A	 [`torch.nn.Conv1d`](generated/torch.nn.Conv1d.html#torch.nn.Conv1d "torch.nn.Conv1d")	 module with lazy initialization of the	 `in_channels`	 argument of the	 `Conv1d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyConv2d`](generated/torch.nn.LazyConv2d.html#torch.nn.LazyConv2d "torch.nn.LazyConv2d")	 | 	 A	 [`torch.nn.Conv2d`](generated/torch.nn.Conv2d.html#torch.nn.Conv2d "torch.nn.Conv2d")	 module with lazy initialization of the	 `in_channels`	 argument of the	 `Conv2d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyConv3d`](generated/torch.nn.LazyConv3d.html#torch.nn.LazyConv3d "torch.nn.LazyConv3d")	 | 	 A	 [`torch.nn.Conv3d`](generated/torch.nn.Conv3d.html#torch.nn.Conv3d "torch.nn.Conv3d")	 module with lazy initialization of the	 `in_channels`	 argument of the	 `Conv3d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyConvTranspose1d`](generated/torch.nn.LazyConvTranspose1d.html#torch.nn.LazyConvTranspose1d "torch.nn.LazyConvTranspose1d")	 | 	 A	 [`torch.nn.ConvTranspose1d`](generated/torch.nn.ConvTranspose1d.html#torch.nn.ConvTranspose1d "torch.nn.ConvTranspose1d")	 module with lazy initialization of the	 `in_channels`	 argument of the	 `ConvTranspose1d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyConvTranspose2d`](generated/torch.nn.LazyConvTranspose2d.html#torch.nn.LazyConvTranspose2d "torch.nn.LazyConvTranspose2d")	 | 	 A	 [`torch.nn.ConvTranspose2d`](generated/torch.nn.ConvTranspose2d.html#torch.nn.ConvTranspose2d "torch.nn.ConvTranspose2d")	 module with lazy initialization of the	 `in_channels`	 argument of the	 `ConvTranspose2d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyConvTranspose3d`](generated/torch.nn.LazyConvTranspose3d.html#torch.nn.LazyConvTranspose3d "torch.nn.LazyConvTranspose3d")	 | 	 A	 [`torch.nn.ConvTranspose3d`](generated/torch.nn.ConvTranspose3d.html#torch.nn.ConvTranspose3d "torch.nn.ConvTranspose3d")	 module with lazy initialization of the	 `in_channels`	 argument of the	 `ConvTranspose3d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.Unfold`](generated/torch.nn.Unfold.html#torch.nn.Unfold "torch.nn.Unfold")	 | 	 Extracts sliding local blocks from a batched input tensor.	  |
| 	[`nn.Fold`](generated/torch.nn.Fold.html#torch.nn.Fold "torch.nn.Fold")	 | 	 Combines an array of sliding local blocks into a large containing tensor.	  |


## [池化层](#id1) [¶](#pooling-layers "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.MaxPool1d`](generated/torch.nn.MaxPool1d.html#torch.nn.MaxPool1d "torch.nn.MaxPool1d")	 | 	 Applies a 1D max pooling over an input signal composed of several input planes.	  |
| 	[`nn.MaxPool2d`](generated/torch.nn.MaxPool2d.html#torch.nn.MaxPool2d "torch.nn.MaxPool2d")	 | 	 Applies a 2D max pooling over an input signal composed of several input planes.	  |
| 	[`nn.MaxPool3d`](generated/torch.nn.MaxPool3d.html#torch.nn.MaxPool3d "torch.nn.MaxPool3d")	 | 	 Applies a 3D max pooling over an input signal composed of several input planes.	  |
| 	[`nn.MaxUnpool1d`](generated/torch.nn.MaxUnpool1d.html#torch.nn.MaxUnpool1d "torch.nn.MaxUnpool1d")	 | 	 Computes a partial inverse of	 `MaxPool1d`	.	  |
| 	[`nn.MaxUnpool2d`](generated/torch.nn.MaxUnpool2d.html#torch.nn.MaxUnpool2d "torch.nn.MaxUnpool2d")	 | 	 Computes a partial inverse of	 `MaxPool2d`	.	  |
| 	[`nn.MaxUnpool3d`](generated/torch.nn.MaxUnpool3d.html#torch.nn.MaxUnpool3d "torch.nn.MaxUnpool3d")	 | 	 Computes a partial inverse of	 `MaxPool3d`	.	  |
| 	[`nn.AvgPool1d`](generated/torch.nn.AvgPool1d.html#torch.nn.AvgPool1d "torch.nn.AvgPool1d")	 | 	 Applies a 1D average pooling over an input signal composed of several input planes.	  |
| 	[`nn.AvgPool2d`](generated/torch.nn.AvgPool2d.html#torch.nn.AvgPool2d "torch.nn.AvgPool2d")	 | 	 Applies a 2D average pooling over an input signal composed of several input planes.	  |
| 	[`nn.AvgPool3d`](generated/torch.nn.AvgPool3d.html#torch.nn.AvgPool3d "torch.nn.AvgPool3d")	 | 	 Applies a 3D average pooling over an input signal composed of several input planes.	  |
| 	[`nn.FractionalMaxPool2d`](generated/torch.nn.FractionalMaxPool2d.html#torch.nn.FractionalMaxPool2d "torch.nn.FractionalMaxPool2d")	 | 	 Applies a 2D fractional max pooling over an input signal composed of several input planes.	  |
| 	[`nn.FractionalMaxPool3d`](generated/torch.nn.FractionalMaxPool3d.html#torch.nn.FractionalMaxPool3d "torch.nn.FractionalMaxPool3d")	 | 	 Applies a 3D fractional max pooling over an input signal composed of several input planes.	  |
| 	[`nn.LPPool1d`](generated/torch.nn.LPPool1d.html#torch.nn.LPPool1d "torch.nn.LPPool1d")	 | 	 Applies a 1D power-average pooling over an input signal composed of several input planes.	  |
| 	[`nn.LPPool2d`](generated/torch.nn.LPPool2d.html#torch.nn.LPPool2d "torch.nn.LPPool2d")	 | 	 Applies a 2D power-average pooling over an input signal composed of several input planes.	  |
| 	[`nn.AdaptiveMaxPool1d`](generated/torch.nn.AdaptiveMaxPool1d.html#torch.nn.AdaptiveMaxPool1d "torch.nn.AdaptiveMaxPool1d")	 | 	 Applies a 1D adaptive max pooling over an input signal composed of several input planes.	  |
| 	[`nn.AdaptiveMaxPool2d`](generated/torch.nn.AdaptiveMaxPool2d.html#torch.nn.AdaptiveMaxPool2d "torch.nn.AdaptiveMaxPool2d")	 | 	 Applies a 2D adaptive max pooling over an input signal composed of several input planes.	  |
| 	[`nn.AdaptiveMaxPool3d`](generated/torch.nn.AdaptiveMaxPool3d.html#torch.nn.AdaptiveMaxPool3d "torch.nn.AdaptiveMaxPool3d")	 | 	 Applies a 3D adaptive max pooling over an input signal composed of several input planes.	  |
| 	[`nn.AdaptiveAvgPool1d`](generated/torch.nn.AdaptiveAvgPool1d.html#torch.nn.AdaptiveAvgPool1d "torch.nn.AdaptiveAvgPool1d")	 | 	 Applies a 1D adaptive average pooling over an input signal composed of several input planes.	  |
| 	[`nn.AdaptiveAvgPool2d`](generated/torch.nn.AdaptiveAvgPool2d.html#torch.nn.AdaptiveAvgPool2d "torch.nn.AdaptiveAvgPool2d")	 | 	 Applies a 2D adaptive average pooling over an input signal composed of several input planes.	  |
| 	[`nn.AdaptiveAvgPool3d`](generated/torch.nn.AdaptiveAvgPool3d.html#torch.nn.AdaptiveAvgPool3d "torch.nn.AdaptiveAvgPool3d")	 | 	 Applies a 3D adaptive average pooling over an input signal composed of several input planes.	  |


## [填充层](#id1) [¶](#padding-layers "此标题的固定链接")


|  |  |
| --- | --- |
| 	[`nn.ReflectionPad1d`](generated/torch.nn.ReflectionPad1d.html#torch.nn.ReflectionPad1d "torch.nn.ReflectionPad1d")	 | 	 Pads the input tensor using the reflection of the input boundary.	  |
| 	[`nn.ReflectionPad2d`](generated/torch.nn.ReflectionPad2d.html#torch.nn.ReflectionPad2d "torch.nn.ReflectionPad2d")	 | 	 Pads the input tensor using the reflection of the input boundary.	  |
| 	[`nn.ReflectionPad3d`](generated/torch.nn.ReflectionPad3d.html#torch.nn.ReflectionPad3d "torch.nn.ReflectionPad3d")	 | 	 Pads the input tensor using the reflection of the input boundary.	  |
| 	[`nn.ReplicationPad1d`](generated/torch.nn.ReplicationPad1d.html#torch.nn.ReplicationPad1d "torch.nn.ReplicationPad1d")	 | 	 Pads the input tensor using replication of the input boundary.	  |
| 	[`nn.ReplicationPad2d`](generated/torch.nn.ReplicationPad2d.html#torch.nn.ReplicationPad2d "torch.nn.ReplicationPad2d")	 | 	 Pads the input tensor using replication of the input boundary.	  |
| 	[`nn.ReplicationPad3d`](generated/torch.nn.ReplicationPad3d.html#torch.nn.ReplicationPad3d "torch.nn.ReplicationPad3d")	 | 	 Pads the input tensor using replication of the input boundary.	  |
| 	[`nn.ZeroPad1d`](generated/torch.nn.ZeroPad1d.html#torch.nn.ZeroPad1d "torch.nn.ZeroPad1d")	 | 	 Pads the input tensor boundaries with zero.	  |
| 	[`nn.ZeroPad2d`](generated/torch.nn.ZeroPad2d.html#torch.nn.ZeroPad2d "torch.nn.ZeroPad2d")	 | 	 Pads the input tensor boundaries with zero.	  |
| 	[`nn.ZeroPad3d`](generated/torch.nn.ZeroPad3d.html#torch.nn.ZeroPad3d "torch.nn.ZeroPad3d")	 | 	 Pads the input tensor boundaries with zero.	  |
| 	[`nn.ConstantPad1d`](generated/torch.nn.ConstantPad1d.html#torch.nn.ConstantPad1d "torch.nn.ConstantPad1d")	 | 	 Pads the input tensor boundaries with a constant value.	  |
| 	[`nn.ConstantPad2d`](generated/torch.nn.ConstantPad2d.html#torch.nn.ConstantPad2d "torch.nn.ConstantPad2d")	 | 	 Pads the input tensor boundaries with a constant value.	  |
| 	[`nn.ConstantPad3d`](generated/torch.nn.ConstantPad3d.html#torch.nn.ConstantPad3d "torch.nn.ConstantPad3d")	 | 	 Pads the input tensor boundaries with a constant value.	  |


## [非线性激活(加权和，非线性)](#id1) [¶](#non-linear-activations-weighted-sum-nonlinearity "永久链接到此标题")


|  |  |
| --- | --- |
| 	[`nn.ELU`](generated/torch.nn.ELU.html#torch.nn.ELU "torch.nn.ELU")	 | 	 Applies the Exponential Linear Unit (ELU) function, element-wise, as described in the paper:	 [Fast and Accurate Deep Network Learning by Exponential Linear Units (ELUs)](https://arxiv.org/abs/1511.07289) 	.	  |
| 	[`nn.Hardshrink`](generated/torch.nn.Hardshrink.html#torch.nn.Hardshrink "torch.nn.Hardshrink")	 | 	 Applies the Hard Shrinkage (Hardshrink) function element-wise.	  |
| 	[`nn.Hardsigmoid`](generated/torch.nn.Hardsigmoid.html#torch.nn.Hardsigmoid "torch.nn.Hardsigmoid")	 | 	 Applies the Hardsigmoid function element-wise.	  |
| 	[`nn.Hardtanh`](generated/torch.nn.Hardtanh.html#torch.nn.Hardtanh "torch.nn.Hardtanh")	 | 	 Applies the HardTanh function element-wise.	  |
| 	[`nn.Hardswish`](generated/torch.nn.Hardswish.html#torch.nn.Hardswish "torch.nn.Hardswish")	 | 	 Applies the Hardswish function, element-wise, as described in the paper:	 [Searching for MobileNetV3](https://arxiv.org/abs/1905.02244) 	.	  |
| 	[`nn.LeakyReLU`](generated/torch.nn.LeakyReLU.html#torch.nn.LeakyReLU "torch.nn.LeakyReLU")	 | 	 Applies the element-wise function:	  |
| 	[`nn.LogSigmoid`](generated/torch.nn.LogSigmoid.html#torch.nn.LogSigmoid "torch.nn.LogSigmoid")	 | 	 Applies the element-wise function:	  |
| 	[`nn.MultiheadAttention`](generated/torch.nn.MultiheadAttention.html#torch.nn.MultiheadAttention "torch.nn.MultiheadAttention")	 | 	 Allows the model to jointly attend to information from different representation subspaces as described in the paper:	 [Attention Is All You Need](https://arxiv.org/abs/1706.03762) 	.	  |
| 	[`nn.PReLU`](generated/torch.nn.PReLU.html#torch.nn.PReLU "torch.nn.PReLU")	 | 	 Applies the element-wise function:	  |
| 	[`nn.ReLU`](generated/torch.nn.ReLU.html#torch.nn.ReLU "torch.nn.ReLU")	 | 	 Applies the rectified linear unit function element-wise:	  |
| 	[`nn.ReLU6`](generated/torch.nn.ReLU6.html#torch.nn.ReLU6 "torch.nn.ReLU6")	 | 	 Applies the element-wise function:	  |
| 	[`nn.RReLU`](generated/torch.nn.RReLU.html#torch.nn.RReLU "torch.nn.RReLU")	 | 	 Applies the randomized leaky rectified liner unit function, element-wise, as described in the paper:	  |
| 	[`nn.SELU`](generated/torch.nn.SELU.html#torch.nn.SELU "torch.nn.SELU")	 | 	 Applied element-wise, as:	  |
| 	[`nn.CELU`](generated/torch.nn.CELU.html#torch.nn.CELU "torch.nn.CELU")	 | 	 Applies the element-wise function:	  |
| 	[`nn.GELU`](generated/torch.nn.GELU.html#torch.nn.GELU "torch.nn.GELU")	 | 	 Applies the Gaussian Error Linear Units function:	  |
| 	[`nn.Sigmoid`](generated/torch.nn.Sigmoid.html#torch.nn.Sigmoid "torch.nn.Sigmoid")	 | 	 Applies the element-wise function:	  |
| 	[`nn.SiLU`](generated/torch.nn.SiLU.html#torch.nn.SiLU "torch.nn.SiLU")	 | 	 Applies the Sigmoid Linear Unit (SiLU) function, element-wise.	  |
| 	[`nn.Mish`](generated/torch.nn.Mish.html#torch.nn.Mish "torch.nn.Mish")	 | 	 Applies the Mish function, element-wise.	  |
| 	[`nn.Softplus`](generated/torch.nn.Softplus.html#torch.nn.Softplus "torch.nn.Softplus")	 | 	 Applies the Softplus function


 软加 ( x ) =


 1
 

 β
 


 
* log ⁡ ( 1 
+ exp ⁡ ( β 
* x ) )


 	ext{Softplus}(x) = rac{1}{eta} * \log(1 
+ \exp(eta * x))


 软加


 (  X  )



 =
 




 β
 



 1
 


 ​
 



 ∗
 




 lo
 
 g
 


 (
 

 1
 



 +
 


 经验 (b



 ∗
 




 x
 

 ))
 


 元素方面。 |
| [`nn.Softshrink`](generated/torch.nn.Softshrink.html#torch.nn.Softshrink "torch.nn.Softshrink") |按元素应用软收缩函数： |
| [`nn.Softsign`](generated/torch.nn.Softsign.html#torch.nn.Softsign“torch.nn.Softsign”) |应用逐元素函数：|
| [`nn.Tanh`](generated/torch.nn.Tanh.html#torch.nn.Tanh "torch.nn.Tanh") |按元素应用双曲正切 (Tanh) 函数。 |
| [`nn.Tanhshrink`](generated/torch.nn.Tanhshrink.html#torch.nn.Tanhshrink "torch.nn.Tanhshrink") |应用逐元素函数：|
| [`nn.Threshold`](generated/torch.nn.Threshold.html#torch.nn.Threshold "torch.nn.Threshold") |对输入tensor的每个元素设置阈值。 |
| [`nn.GLU`](generated/torch.nn.GLU.html#torch.nn.GLU "torch.nn.GLU") |应用门控线性单位函数


 谷氨酸


 ( a , b ) = a ⊗ σ ( b )


 {GLU}(a, b)= a \otimes \sigma(b)



 G
 

 LU
 


 (  A  ，



 b
 

 )
 



 =
 




 a
 



 ⊗
 


 s ( b )




 where
 



 a
 


 a
 


 a
 


 是输入矩阵的前半部分，



 b
 


 b
 


 b
 


 是下半场。 |


## [非线性激活(其他)](#id1) [¶](#non-linear-activations-other "永久链接到此标题")


|  |  |
| --- | --- |
| 	[`nn.Softmin`](generated/torch.nn.Softmin.html#torch.nn.Softmin "torch.nn.Softmin")	 | 	 Applies the Softmin function to an n-dimensional input Tensor rescaling them so that the elements of the n-dimensional output Tensor lie in the range	 	 [0, 1]	 	 and sum to 1.	  |
| 	[`nn.Softmax`](generated/torch.nn.Softmax.html#torch.nn.Softmax "torch.nn.Softmax")	 | 	 Applies the Softmax function to an n-dimensional input Tensor rescaling them so that the elements of the n-dimensional output Tensor lie in the range [0,1] and sum to 1.	  |
| 	[`nn.Softmax2d`](generated/torch.nn.Softmax2d.html#torch.nn.Softmax2d "torch.nn.Softmax2d")	 | 	 Applies SoftMax over features to each spatial location.	  |
| 	[`nn.LogSoftmax`](generated/torch.nn.LogSoftmax.html#torch.nn.LogSoftmax "torch.nn.LogSoftmax")	 | 	 Applies the


 log ⁡ ( Softmax ( x ) )


 \log(	ext{Softmax}(x))


 lo
 
 g
 


 (
 


 软最大


 (  X  ))


 函数到 n 维输入tensor。 |
| [`nn.AdaptiveLogSoftmaxWithLoss`](generated/torch.nn.AdaptiveLogSoftmaxWithLoss.html#torch.nn.AdaptiveLogSoftmaxWithLoss“torch.nn.AdaptiveLogSoftmaxWithLoss”)|高效的 softmax 近似，如 [Edouard Grave、Armand Joulin、Moustapha Cissé、David Grangier 和 Hervé Jégou 对 GPU 的高效 softmax 近似](https://arxiv.org/abs/1609.04309) 中所述。 |


## [标准化层](#id1) [¶](#normalization-layers "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.BatchNorm1d`](generated/torch.nn.BatchNorm1d.html#torch.nn.BatchNorm1d "torch.nn.BatchNorm1d")	 | 	 Applies Batch Normalization over a 2D or 3D input as described in the paper	 [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167) 	.	  |
| 	[`nn.BatchNorm2d`](generated/torch.nn.BatchNorm2d.html#torch.nn.BatchNorm2d "torch.nn.BatchNorm2d")	 | 	 Applies Batch Normalization over a 4D input (a mini-batch of 2D inputs with additional channel dimension) as described in the paper	 [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167) 	.	  |
| 	[`nn.BatchNorm3d`](generated/torch.nn.BatchNorm3d.html#torch.nn.BatchNorm3d "torch.nn.BatchNorm3d")	 | 	 Applies Batch Normalization over a 5D input (a mini-batch of 3D inputs with additional channel dimension) as described in the paper	 [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167) 	.	  |
| 	[`nn.LazyBatchNorm1d`](generated/torch.nn.LazyBatchNorm1d.html#torch.nn.LazyBatchNorm1d "torch.nn.LazyBatchNorm1d")	 | 	 A	 [`torch.nn.BatchNorm1d`](generated/torch.nn.BatchNorm1d.html#torch.nn.BatchNorm1d "torch.nn.BatchNorm1d")	 module with lazy initialization of the	 `num_features`	 argument of the	 `BatchNorm1d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyBatchNorm2d`](generated/torch.nn.LazyBatchNorm2d.html#torch.nn.LazyBatchNorm2d "torch.nn.LazyBatchNorm2d")	 | 	 A	 [`torch.nn.BatchNorm2d`](generated/torch.nn.BatchNorm2d.html#torch.nn.BatchNorm2d "torch.nn.BatchNorm2d")	 module with lazy initialization of the	 `num_features`	 argument of the	 `BatchNorm2d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyBatchNorm3d`](generated/torch.nn.LazyBatchNorm3d.html#torch.nn.LazyBatchNorm3d "torch.nn.LazyBatchNorm3d")	 | 	 A	 [`torch.nn.BatchNorm3d`](generated/torch.nn.BatchNorm3d.html#torch.nn.BatchNorm3d "torch.nn.BatchNorm3d")	 module with lazy initialization of the	 `num_features`	 argument of the	 `BatchNorm3d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.GroupNorm`](generated/torch.nn.GroupNorm.html#torch.nn.GroupNorm "torch.nn.GroupNorm")	 | 	 Applies Group Normalization over a mini-batch of inputs as described in the paper	 [Group Normalization](https://arxiv.org/abs/1803.08494) 	 |
| 	[`nn.SyncBatchNorm`](generated/torch.nn.SyncBatchNorm.html#torch.nn.SyncBatchNorm "torch.nn.SyncBatchNorm")	 | 	 Applies Batch Normalization over a N-Dimensional input (a mini-batch of [N-2]D inputs with additional channel dimension) as described in the paper	 [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167) 	.	  |
| 	[`nn.InstanceNorm1d`](generated/torch.nn.InstanceNorm1d.html#torch.nn.InstanceNorm1d "torch.nn.InstanceNorm1d")	 | 	 Applies Instance Normalization over a 2D (unbatched) or 3D (batched) input as described in the paper	 [Instance Normalization: The Missing Ingredient for Fast Stylization](https://arxiv.org/abs/1607.08022) 	.	  |
| 	[`nn.InstanceNorm2d`](generated/torch.nn.InstanceNorm2d.html#torch.nn.InstanceNorm2d "torch.nn.InstanceNorm2d")	 | 	 Applies Instance Normalization over a 4D input (a mini-batch of 2D inputs with additional channel dimension) as described in the paper	 [Instance Normalization: The Missing Ingredient for Fast Stylization](https://arxiv.org/abs/1607.08022) 	.	  |
| 	[`nn.InstanceNorm3d`](generated/torch.nn.InstanceNorm3d.html#torch.nn.InstanceNorm3d "torch.nn.InstanceNorm3d")	 | 	 Applies Instance Normalization over a 5D input (a mini-batch of 3D inputs with additional channel dimension) as described in the paper	 [Instance Normalization: The Missing Ingredient for Fast Stylization](https://arxiv.org/abs/1607.08022) 	.	  |
| 	[`nn.LazyInstanceNorm1d`](generated/torch.nn.LazyInstanceNorm1d.html#torch.nn.LazyInstanceNorm1d "torch.nn.LazyInstanceNorm1d")	 | 	 A	 [`torch.nn.InstanceNorm1d`](generated/torch.nn.InstanceNorm1d.html#torch.nn.InstanceNorm1d "torch.nn.InstanceNorm1d")	 module with lazy initialization of the	 `num_features`	 argument of the	 `InstanceNorm1d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyInstanceNorm2d`](generated/torch.nn.LazyInstanceNorm2d.html#torch.nn.LazyInstanceNorm2d "torch.nn.LazyInstanceNorm2d")	 | 	 A	 [`torch.nn.InstanceNorm2d`](generated/torch.nn.InstanceNorm2d.html#torch.nn.InstanceNorm2d "torch.nn.InstanceNorm2d")	 module with lazy initialization of the	 `num_features`	 argument of the	 `InstanceNorm2d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LazyInstanceNorm3d`](generated/torch.nn.LazyInstanceNorm3d.html#torch.nn.LazyInstanceNorm3d "torch.nn.LazyInstanceNorm3d")	 | 	 A	 [`torch.nn.InstanceNorm3d`](generated/torch.nn.InstanceNorm3d.html#torch.nn.InstanceNorm3d "torch.nn.InstanceNorm3d")	 module with lazy initialization of the	 `num_features`	 argument of the	 `InstanceNorm3d`	 that is inferred from the	 `input.size(1)`	.	  |
| 	[`nn.LayerNorm`](generated/torch.nn.LayerNorm.html#torch.nn.LayerNorm "torch.nn.LayerNorm")	 | 	 Applies Layer Normalization over a mini-batch of inputs as described in the paper	 [Layer Normalization](https://arxiv.org/abs/1607.06450) 	 |
| 	[`nn.LocalResponseNorm`](generated/torch.nn.LocalResponseNorm.html#torch.nn.LocalResponseNorm "torch.nn.LocalResponseNorm")	 | 	 Applies local response normalization over an input signal composed of several input planes, where channels occupy the second dimension.	  |


## [循环层](#id1) [¶](#recurrent-layers "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.RNNBase`](generated/torch.nn.RNNBase.html#torch.nn.RNNBase "torch.nn.RNNBase")	 | 	 Base class for RNN modules (RNN, LSTM, GRU).	  |
| 	[`nn.RNN`](generated/torch.nn.RNN.html#torch.nn.RNN "torch.nn.RNN")	 | 	 Applies a multi-layer Elman RNN with


 腥⁡


 	anh
 


 tanh
 




 or
 



 ReLU
 


 	ext{ReLU}



 ReLU
 


 输入序列的非线性。 |
| [`nn.LSTM`](generated/torch.nn.LSTM.html#torch.nn.LSTM "torch.nn.LSTM") |将多层长短期记忆 (LSTM) RNN 应用于输入序列。 |
| [`nn.GRU`](generated/torch.nn.GRU.html#torch.nn.GRU“torch.nn.GRU”)|将多层门控循环单元 (GRU) RNN 应用于输入序列。 |
| [`nn.RNNCell`](generated/torch.nn.RNNCell.html#torch.nn.RNNCell "torch.nn.RNNCell") |具有 tanh 或 ReLU 非线性的 Elman RNN 单元。 |
| [`nn.LSTMCell`](generated/torch.nn.LSTMCell.html#torch.nn.LSTMCell "torch.nn.LSTMCell") |长短期记忆 (LSTM) 细胞。 |
| [`nn.GRUCell`](generated/torch.nn.GRUCell.html#torch.nn.GRUCell "torch.nn.GRUCell") |门控循环单元 (GRU) 单元 |


## [Transformer Layers](#id1) [¶](#transformer-layers "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.Transformer`](generated/torch.nn.Transformer.html#torch.nn.Transformer "torch.nn.Transformer")	 | 	 A transformer model.	  |
| 	[`nn.TransformerEncoder`](generated/torch.nn.TransformerEncoder.html#torch.nn.TransformerEncoder "torch.nn.TransformerEncoder")	 | 	 TransformerEncoder is a stack of N encoder layers.	  |
| 	[`nn.TransformerDecoder`](generated/torch.nn.TransformerDecoder.html#torch.nn.TransformerDecoder "torch.nn.TransformerDecoder")	 | 	 TransformerDecoder is a stack of N decoder layers	  |
| 	[`nn.TransformerEncoderLayer`](generated/torch.nn.TransformerEncoderLayer.html#torch.nn.TransformerEncoderLayer "torch.nn.TransformerEncoderLayer")	 | 	 TransformerEncoderLayer is made up of self-attn and feedforward network.	  |
| 	[`nn.TransformerDecoderLayer`](generated/torch.nn.TransformerDecoderLayer.html#torch.nn.TransformerDecoderLayer "torch.nn.TransformerDecoderLayer")	 | 	 TransformerDecoderLayer is made up of self-attn, multi-head-attn and feedforward network.	  |


## [线性图层](#id1) [¶](#linear-layers "此标题的固定链接")


|  |  |
| --- | --- |
| 	[`nn.Identity`](generated/torch.nn.Identity.html#torch.nn.Identity "torch.nn.Identity")	 | 	 A placeholder identity operator that is argument-insensitive.	  |
| 	[`nn.Linear`](generated/torch.nn.Linear.html#torch.nn.Linear "torch.nn.Linear")	 | 	 Applies a linear transformation to the incoming data:


 y = x


 A
 

 T
 


 +
 

 b
 


 y = xA^T 
+ b


 y
 



 =
 




 x
 


 A
 



 T
 




 +
 




 b
 


 |
| [`nn.Bi线性`](generated/torch.nn.Bilinear.html#torch.nn.Bi线性“torch.nn.Bi线性”) |对传入数据应用双线性变换：



 y
 

 =
 


 x 1 吨


 A
 


 x
 

 2
 


 +
 

 b
 


 y = x_1^T A x_2 
+ b


 y
 



 =
 


 x
 



 1
 




 T
 




 ​
 


 A
 


 x
 



 2
 




 ​
 




 +
 




 b
 


 |
| [`nn.LazyLinear`](generated/torch.nn.LazyLinear.html#torch.nn.LazyLinear "torch.nn.LazyLinear") |一个 [`torch.nn.Linear`](generated/torch.nn.Linear.html#torch.nn.Linear "torch.nn.Linear") 模块，其中 in_features 被推断出来。 |


## [Dropout Layers](#id1) [¶](#dropout-layers "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.Dropout`](generated/torch.nn.Dropout.html#torch.nn.Dropout "torch.nn.Dropout")	 | 	 During training, randomly zeroes some of the elements of the input tensor with probability	 `p`	 using samples from a Bernoulli distribution.	  |
| 	[`nn.Dropout1d`](generated/torch.nn.Dropout1d.html#torch.nn.Dropout1d "torch.nn.Dropout1d")	 | 	 Randomly zero out entire channels (a channel is a 1D feature map, e.g., the



 j
 


 j
 


 j
 


 第 通道



 i
 


 i
 


 i
 


 
- 批处理输入中的第一个样本是一维tensor


 输入 [ i , j ]


 	ext{输入}[i, j]



 input
 


 [  我  ，



 j
 

 ]
 


 )。 |
| [`nn.Dropout2d`](generated/torch.nn.Dropout2d.html#torch.nn.Dropout2d“torch.nn.Dropout2d”)|随机将整个通道归零(通道是一个 2D 特征图，例如



 j
 


 j
 


 j
 


 第 通道



 i
 


 i
 


 i
 


 
- 批处理输入中的第一个样本是一个 2D tensor


 输入 [ i , j ]


 	ext{输入}[i, j]



 input
 


 [  我  ，



 j
 

 ]
 


 )。 |
| [`nn.Dropout3d`](generated/torch.nn.Dropout3d.html#torch.nn.Dropout3d“torch.nn.Dropout3d”)|随机将整个通道归零(通道是 3D 特征图，例如



 j
 


 j
 


 j
 


 第 通道



 i
 


 i
 


 i
 


 
- 批处理输入中的第一个样本是 3D tensor


 输入 [ i , j ]


 	ext{输入}[i, j]



 input
 


 [  我  ，



 j
 

 ]
 


 )。 |
| [`nn.AlphaDropout`](generated/torch.nn.AlphaDropout.html#torch.nn.AlphaDropout "torch.nn.AlphaDropout") |对输入应用 Alpha Dropout。 |
| [`nn.FeatureAlphaDropout`](generated/torch.nn.FeatureAlphaDropout.html#torch.nn.FeatureAlphaDropout“torch.nn.FeatureAlphaDropout”)|随机屏蔽整个通道(通道是一个特征图，例如 |


## [稀疏层](#id1) [¶](#sparse-layers "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.Embedding`](generated/torch.nn.Embedding.html#torch.nn.Embedding "torch.nn.Embedding")	 | 	 A simple lookup table that stores embeddings of a fixed dictionary and size.	  |
| 	[`nn.EmbeddingBag`](generated/torch.nn.EmbeddingBag.html#torch.nn.EmbeddingBag "torch.nn.EmbeddingBag")	 | 	 Computes sums or means of 'bags' of embeddings, without instantiating the intermediate embeddings.	  |


## [距离函数](#id1) [¶](#distance-functions "此标题的固定链接")


|  |  |
| --- | --- |
| 	[`nn.CosineSimilarity`](generated/torch.nn.CosineSimilarity.html#torch.nn.CosineSimilarity "torch.nn.CosineSimilarity")	 | 	 Returns cosine similarity between




 x
 

 1
 



 x\_1
 



 x
 



 1
 




 ​
 


 and
 




 x
 

 2
 



 x\_2
 



 x
 



 2
 




 ​
 


 ，沿着 dim 计算。 |
| [`nn.PairwiseDistance`](generated/torch.nn.PairwiseDistance.html#torch.nn.PairwiseDistance "torch.nn.PairwiseDistance") |计算输入向量之间或输入矩阵列之间的成对距离。 |


## [损失函数](#id1) [¶](#loss-functions "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.L1Loss`](generated/torch.nn.L1Loss.html#torch.nn.L1Loss "torch.nn.L1Loss")	 | 	 Creates a criterion that measures the mean absolute error (MAE) between each element in the input



 x
 


 x
 


 x
 


 和目标



 y
 


 y
 


 y
 


 。 |
| [`nn.MSELoss`](generated/torch.nn.MSELoss.html#torch.nn.MSELoss "torch.nn.MSELoss") |创建一个标准来测量输入中每个元素之间的均方误差(平方 L2 范数)



 x
 


 x
 


 x
 


 和目标



 y
 


 y
 


 y
 


 。 |
| [`nn.CrossEntropyLoss`](generated/torch.nn.CrossEntropyLoss.html#torch.nn.CrossEntropyLoss "torch.nn.CrossEntropyLoss") |该标准计算输入 logits 和目标之间的交叉熵损失。 |
| [`nn.CTCLoss`](generated/torch.nn.CTCLoss.html#torch.nn.CTCLoss "torch.nn.CTCLoss") |联结主义时间分类损失。 |
| [`nn.NLLLoss`](generated/torch.nn.NLLLoss.html#torch.nn.NLLLoss "torch.nn.NLLLoss") |负对数似然损失。 |
| [`nn.PoissonNLLLoss`](generated/torch.nn.PoissonNLLLoss.html#torch.nn.PoissonNLLLoss "torch.nn.PoissonNLLLoss") |目标泊松分布的负对数似然损失。 |
| [`nn.GaussianNLLLoss`](generated/torch.nn.GaussianNLLLoss.html#torch.nn.GaussianNLLLoss "torch.nn.GaussianNLLLoss") |高斯负对数似然损失。 |
| [`nn.KLDivLoss`](generated/torch.nn.KLDivLoss.html#torch.nn.KLDivLoss "torch.nn.KLDivLoss") | Kullback-Leibler 散度损失。 |
| [`nn.BCELoss`](generated/torch.nn.BCELoss.html#torch.nn.BCELoss "torch.nn.BCELoss") |创建一个衡量目标概率和输入概率之间的二元交叉熵的标准： |
| [`nn.BCEWithLogitsLoss`](generated/torch.nn.BCEWithLogitsLoss.html#torch.nn.BCEWithLogitsLoss "torch.nn.BCEWithLogitsLoss") |该损失将 Sigmoid 层和 BCELoss 结合在一个类别中。 |
| [`nn.MarginRankingLoss`](generated/torch.nn.MarginRankingLoss.html#torch.nn.MarginRankingLoss“torch.nn.MarginRankingLoss”)|创建衡量给定输入损失的标准



 x
 

 1
 


 x1
 


 x
 

 1
 




 ,
 



 x
 

 2
 


 x2
 


 x
 

 2
 


 ，两个 1D mini-batch 或 0D Tensor ，以及一个标签 1D mini-batch 或 0D Tensor




 y
 


 y
 


 y
 


 (包含1或-1)。 |
| [`nn.HingeEmbeddingLoss`](generated/torch.nn.HingeEmbeddingLoss.html#torch.nn.HingeEmbeddingLoss“torch.nn.HingeEmbeddingLoss”)|测量给定输入tensor的损失



 x
 


 x
 


 x
 


 和标签tensor



 y
 


 y
 


 y
 


 (包含1或-1)。 |
| [`nn.MultiLabelMarginLoss`](generated/torch.nn.MultiLabelMarginLoss.html#torch.nn.MultiLabelMarginLoss“torch.nn.MultiLabelMarginLoss”)|创建一个标准，优化输入之间的多类多分类铰链损失(基于边际的损失)



 x
 


 x
 


 x
 


 (2D 小批量tensor)和输出



 y
 


 y
 


 y
 


 (这是目标类别索引的二维tensor)。 |
| [`nn.HuberLoss`](generated/torch.nn.HuberLoss.html#torch.nn.HuberLoss "torch.nn.HuberLoss") |创建一个标准，如果绝对元素误差低于 delta，则使用平方项，否则使用 delta 缩放的 L1 项。 |
| [`nn.SmoothL1Loss`](generated/torch.nn.SmoothL1Loss.html#torch.nn.SmoothL1Loss "torch.nn.SmoothL1Loss") |如果绝对元素误差低于 beta，则创建一个使用平方项的标准，否则使用 L1 项。 |
| [`nn.SoftMarginLoss`](generated/torch.nn.SoftMarginLoss.html#torch.nn.SoftMarginLoss“torch.nn.SoftMarginLoss”)|创建一个标准来优化输入tensor之间的二类分类逻辑损失



 x
 


 x
 


 x
 


 和目标tensor



 y
 


 y
 


 y
 


 (包含1或-1)。 |
| [`nn.MultiLabelSoftMarginLoss`](generated/torch.nn.MultiLabelSoftMarginLoss.html#torch.nn.MultiLabelSoftMarginLoss“torch.nn.MultiLabelSoftMarginLoss”)|创建一个标准，在输入之间基于最大熵优化多标签一对一损失



 x
 


 x
 


 x
 


 和目标



 y
 


 y
 


 y
 


 尺寸的


 ( 中 , 中 )


 (北、中)


 (N,



 C
 

 )
 


 。 |
| [`nn.CosineEmbeddingLoss`](generated/torch.nn.CosineEmbeddingLoss.html#torch.nn.CosineEmbeddingLoss“torch.nn.CosineEmbeddingLoss”)|创建一个标准来测量给定输入tensor的损失




 x
 

 1
 



 x\_1
 



 x
 



 1
 




 ​
 


 ,
 




 x
 

 2
 



 x\_2
 



 x
 



 2
 




 ​
 


 和tensor标签



 y
 


 y
 


 y
 


 值为 1 或 -1。 |
| [`nn.MultiMarginLoss`](generated/torch.nn.MultiMarginLoss.html#torch.nn.MultiMarginLoss "torch.nn.MultiMarginLoss") |创建一个标准来优化输入之间的多类分类铰链损失(基于边际的损失)



 x
 


 x
 


 x
 


 (2D 小批量tensor)和输出



 y
 


 y
 


 y
 


 (这是目标类别索引的一维tensor，


 0 ≤ y ≤ x.size ( 1 ) − 1


 0 \leq y \leq 	ext{x.size}(1)-1


 0
 



 ≤
 




 y
 



 ≤
 


 x 尺寸


 ( 1 )



 −
 




 1
 


 ): |
| [`nn.TripletMarginLoss`](generated/torch.nn.TripletMarginLoss.html#torch.nn.TripletMarginLoss "torch.nn.TripletMarginLoss") |创建一个标准来测量给定输入tensor的三元组损失



 x
 

 1
 


 x1
 


 x
 

 1
 




 ,
 



 x
 

 2
 


 x2
 


 x
 

 2
 




 ,
 



 x
 

 3
 


 x3
 


 x
 

 3
 


 和一个值大于的边距



 0
 


 0
 


 0
 


 。 |
| [`nn.TripletMarginWithDistanceLoss`](generated/torch.nn.TripletMarginWithDistanceLoss.html#torch.nn.TripletMarginWithDistanceLoss“torch.nn.TripletMarginWithDistanceLoss”)|创建一个标准来测量给定输入tensor的三元组损失



 a
 


 a
 


 a
 




 ,
 



 p
 


 p
 


 p
 




 , and
 



 n
 


 n
 


 n
 


 (分别代表锚点、正例和负例)，以及用于计算锚点和正例(“正距离”)以及锚点和负例之间关系的非负实值函数(“距离函数”) (“负距离”)。 |


## [Vision Layers](#id1) [¶](#vision-layers "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.PixelShuffle`](generated/torch.nn.PixelShuffle.html#torch.nn.PixelShuffle "torch.nn.PixelShuffle")	 | 	 Rearranges elements in a tensor of shape


 ( 
* , C ×


 r
 

 2
 


 , H , W )


 (*, C 	imes r^2, H, W)


 (*,



 C
 



 ×
 


 r
 



 2
 


 ,
 



 H
 

 ,
 



 W
 

 )
 


 到形状tensor


 ( 
* , C , H × r , W × r )


 (*, C, H 	imes r, W 	imes r)


 (*,



 C
 

 ,
 



 H
 



 ×
 




 r
 

 ,
 



 W
 



 ×
 




 r
 

 )
 


 ，其中 r 是高档因子。 |
| [`nn.PixelUnshuffle`](generated/torch.nn.PixelUnshuffle.html#torch.nn.PixelUnshuffle "torch.nn.PixelUnshuffle") |通过重新排列形状tensor中的元素来反转 [`PixelShuffle`](generated/torch.nn.PixelShuffle.html#torch.nn.PixelShuffle "torch.nn.PixelShuffle") 操作


 ( 
* , C , H × r , W × r )


 (*, C, H 	imes r, W 	imes r)


 (*,



 C
 

 ,
 



 H
 



 ×
 




 r
 

 ,
 



 W
 



 ×
 




 r
 

 )
 


 到形状tensor


 ( 
* , C ×


 r
 

 2
 


 , H , W )


 (*, C 	imes r^2, H, W)


 (*,



 C
 



 ×
 


 r
 



 2
 


 ,
 



 H
 

 ,
 



 W
 

 )
 


 ，其中 r 是缩减因子。 |
| [`nn.Upsample`](generated/torch.nn.Upsample.html#torch.nn.Upsample "torch.nn.Upsample") |对给定的多通道 1D(时间)、2D(空间)或 3D(体积)数据进行上采样。 |
| [`nn.UpsamplingNearest2d`](generated/torch.nn.UpsamplingNearest2d.html#torch.nn.UpsamplingNearest2d“torch.nn.UpsamplingNearest2d”)|对由多个输入通道组成的输入信号应用 2D 最近邻上采样。 |
| [`nn.UpsamplingBilinear2d`](generated/torch.nn.UpsamplingBilinear2d.html#torch.nn.UpsamplingBilinear2d“torch.nn.UpsamplingBilinear2d”)|对由多个输入通道组成的输入信号应用 2D 双线性上采样。 |


## [随机播放图层](#id1) [¶](#shuffle-layers "此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.ChannelShuffle`](generated/torch.nn.ChannelShuffle.html#torch.nn.ChannelShuffle "torch.nn.ChannelShuffle")	 | 	 Divide the channels in a tensor of shape


 (*,C,H,W)


 (*、C、H、W)


 (*,



 C
 

 ,
 



 H
 

 ,
 



 W
 

 )
 


 分成 g 组并将它们重新排列为


 (*，C


 g
 

 ,
 


 克、高、宽)


 (*, C rac g, g, H, W)


 (*,



 C
 




 ,
 



 g
 


 ​
 




 g
 

 ,
 



 H
 

 ,
 



 W
 

 )
 


 ，同时保持原始tensor形状。 |


## [DataParallel 层(多 GPU，分布式)](#id1) [¶](#module-torch.nn.parallel"此标题的永久链接")


|  |  |
| --- | --- |
| 	[`nn.DataParallel`](generated/torch.nn.DataParallel.html#torch.nn.DataParallel "torch.nn.DataParallel")	 | 	 Implements data parallelism at the module level.	  |
| 	[`nn.parallel.DistributedDataParallel`](generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel "torch.nn.parallel.DistributedDataParallel")	 | 	 Implements distributed data parallelism that is based on	 `torch.distributed`	 package at the module level.	  |


## [实用程序](#id1) [¶](#module-torch.nn.utils "此标题的永久链接")
- -


 来自“torch.nn.utils”模块


|  |  |
| --- | --- |
| [`clip_grad_norm_`](generated/torch.nn.utils.clip_grad_norm_.html#torch.nn.utils.clip_grad_norm_“torch.nn.utils.clip_grad_norm_”) |剪辑可迭代参数的梯度范数。 |
| [`clip_grad_value_`](generated/torch.nn.utils.clip_grad_value_.html#torch.nn.utils.clip_grad_value_“torch.nn.utils.clip_grad_value_”) |将参数迭代的梯度剪裁为指定值。 |
| [`parameters_to_vector`](generated/torch.nn.utils.parameters_to_vector.html#torch.nn.utils.parameters_to_vector "torch.nn.utils.parameters_to_vector") |将参数转换为一个向量 |
| [`vector_to_parameters`](generated/torch.nn.utils.vector_to_parameters.html#torch.nn.utils.vector_to_parameters "torch.nn.utils.vector_to_parameters") |将一个向量转换为参数 |
| [`prune.BasePruningMethod`](generated/torch.nn.utils.prune.BasePruningMethod.html#torch.nn.utils.prune.BasePruningMethod“torch.nn.utils.prune.BasePruningMethod”) |用于创建新修剪技术的抽象基类。 |


|  |  |
| --- | --- |
| 	[`prune.PruningContainer`](generated/torch.nn.utils.prune.PruningContainer.html#torch.nn.utils.prune.PruningContainer "torch.nn.utils.prune.PruningContainer")	 | 	 Container holding a sequence of pruning methods for iterative pruning.	  |
| 	[`prune.Identity`](generated/torch.nn.utils.prune.Identity.html#torch.nn.utils.prune.Identity "torch.nn.utils.prune.Identity")	 | 	 Utility pruning method that does not prune any units but generates the pruning parametrization with a mask of ones.	  |
| 	[`prune.RandomUnstructured`](generated/torch.nn.utils.prune.RandomUnstructured.html#torch.nn.utils.prune.RandomUnstructured "torch.nn.utils.prune.RandomUnstructured")	 | 	 Prune (currently unpruned) units in a tensor at random.	  |
| 	[`prune.L1Unstructured`](generated/torch.nn.utils.prune.L1Unstructured.html#torch.nn.utils.prune.L1Unstructured "torch.nn.utils.prune.L1Unstructured")	 | 	 Prune (currently unpruned) units in a tensor by zeroing out the ones with the lowest L1-norm.	  |
| 	[`prune.RandomStructured`](generated/torch.nn.utils.prune.RandomStructured.html#torch.nn.utils.prune.RandomStructured "torch.nn.utils.prune.RandomStructured")	 | 	 Prune entire (currently unpruned) channels in a tensor at random.	  |
| 	[`prune.LnStructured`](generated/torch.nn.utils.prune.LnStructured.html#torch.nn.utils.prune.LnStructured "torch.nn.utils.prune.LnStructured")	 | 	 Prune entire (currently unpruned) channels in a tensor based on their L	 `n`	 -norm.	  |
| 	[`prune.CustomFromMask`](generated/torch.nn.utils.prune.CustomFromMask.html#torch.nn.utils.prune.CustomFromMask "torch.nn.utils.prune.CustomFromMask")	 | 	 |
| 	[`prune.identity`](generated/torch.nn.utils.prune.identity.html#torch.nn.utils.prune.identity "torch.nn.utils.prune.identity")	 | 	 Applies pruning reparametrization to the tensor corresponding to the parameter called	 `name`	 in	 `module`	 without actually pruning any units.	  |
| 	[`prune.random_unstructured`](generated/torch.nn.utils.prune.random_unstructured.html#torch.nn.utils.prune.random_unstructured "torch.nn.utils.prune.random_unstructured")	 | 	 Prunes tensor corresponding to parameter called	 `name`	 in	 `module`	 by removing the specified	 `amount`	 of (currently unpruned) units selected at random.	  |
| 	[`prune.l1_unstructured`](generated/torch.nn.utils.prune.l1_unstructured.html#torch.nn.utils.prune.l1_unstructured "torch.nn.utils.prune.l1_unstructured")	 | 	 Prunes tensor corresponding to parameter called	 `name`	 in	 `module`	 by removing the specified	 	 amount	 	 of (currently unpruned) units with the lowest L1-norm.	  |
| 	[`prune.random_structured`](generated/torch.nn.utils.prune.random_structured.html#torch.nn.utils.prune.random_structured "torch.nn.utils.prune.random_structured")	 | 	 Prunes tensor corresponding to parameter called	 `name`	 in	 `module`	 by removing the specified	 `amount`	 of (currently unpruned) channels along the specified	 `dim`	 selected at random.	  |
| 	[`prune.ln_structured`](generated/torch.nn.utils.prune.ln_structured.html#torch.nn.utils.prune.ln_structured "torch.nn.utils.prune.ln_structured")	 | 	 Prunes tensor corresponding to parameter called	 `name`	 in	 `module`	 by removing the specified	 `amount`	 of (currently unpruned) channels along the specified	 `dim`	 with the lowest L	 `n`	 -norm.	  |
| 	[`prune.global_unstructured`](generated/torch.nn.utils.prune.global_unstructured.html#torch.nn.utils.prune.global_unstructured "torch.nn.utils.prune.global_unstructured")	 | 	 Globally prunes tensors corresponding to all parameters in	 `parameters`	 by applying the specified	 `pruning_method`	.	  |
| 	[`prune.custom_from_mask`](generated/torch.nn.utils.prune.custom_from_mask.html#torch.nn.utils.prune.custom_from_mask "torch.nn.utils.prune.custom_from_mask")	 | 	 Prunes tensor corresponding to parameter called	 `name`	 in	 `module`	 by applying the pre-computed mask in	 `mask`	.	  |
| 	[`prune.remove`](generated/torch.nn.utils.prune.remove.html#torch.nn.utils.prune.remove "torch.nn.utils.prune.remove")	 | 	 Removes the pruning reparameterization from a module and the pruning method from the forward hook.	  |
| 	[`prune.is_pruned`](generated/torch.nn.utils.prune.is_pruned.html#torch.nn.utils.prune.is_pruned "torch.nn.utils.prune.is_pruned")	 | 	 Check whether	 `module`	 is pruned by looking for	 `forward_pre_hooks`	 in its modules that inherit from the	 `BasePruningMethod`	.	  |
| [`weight_norm`](generated/torch.nn.utils.weight_norm.html#torch.nn.utils.weight_norm "torch.nn.utils.weight_norm") |将权重归一化应用于给定模块中的参数。 |
| [`remove_weight_norm`](generated/torch.nn.utils.remove_weight_norm.html#torch.nn.utils.remove_weight_norm "torch.nn.utils.remove_weight_norm") |从模块中删除权重归一化重新参数化。 |
| [`spectral_norm`](generated/torch.nn.utils.spectral_norm.html#torch.nn.utils.spectral_norm“torch.nn.utils.spectral_norm”) |将谱归一化应用于给定模块中的参数。 |
| [`remove_spectral_norm`](generated/torch.nn.utils.remove_spectral_norm.html#torch.nn.utils.remove_spectral_norm“torch.nn.utils.remove_spectral_norm”) |从模块中删除光谱归一化重新参数化。 |
| [`skip_init`](generated/torch.nn.utils.skip_init.html#torch.nn.utils.skip_init "torch.nn.utils.skip_init") |给定模块类对象和 args /kwargs，实例化模块而不初始化参数 /缓冲区。 |


 使用 `torch.nn.utils.parameterize.register_parametrization()` 中的新参数化功能实现参数化。


|  |  |
| --- | --- |
| 	[`parametrizations.orthogonal`](generated/torch.nn.utils.parametrizations.orthogonal.html#torch.nn.utils.parametrizations.orthogonal "torch.nn.utils.parametrizations.orthogonal")	 | 	 Applies an orthogonal or unitary parametrization to a matrix or a batch of matrices.	  |
| 	[`parametrizations.spectral_norm`](generated/torch.nn.utils.parametrizations.spectral_norm.html#torch.nn.utils.parametrizations.spectral_norm "torch.nn.utils.parametrizations.spectral_norm")	 | 	 Applies spectral normalization to a parameter in the given module.	  |


 用于参数化现有模块上的tensor的实用函数。请注意，给定从输入空间映射到参数化空间的特定函数，这些函数可用于参数化给定的参数或缓冲区。它们不是将对象转换为参数的参数化。有关如何实现您自己的参数化的更多信息，请参阅[参数化教程](https://pytorch.org/tutorials/intermediate/parametrizations.html)。


|  |  |
| --- | --- |
| 	[`parametrize.register_parametrization`](generated/torch.nn.utils.parametrize.register_parametrization.html#torch.nn.utils.parametrize.register_parametrization "torch.nn.utils.parametrize.register_parametrization")	 | 	 Adds a parametrization to a tensor in a module.	  |
| 	[`parametrize.remove_parametrizations`](generated/torch.nn.utils.parametrize.remove_parametrizations.html#torch.nn.utils.parametrize.remove_parametrizations "torch.nn.utils.parametrize.remove_parametrizations")	 | 	 Removes the parametrizations on a tensor in a module.	  |
| 	[`parametrize.cached`](generated/torch.nn.utils.parametrize.cached.html#torch.nn.utils.parametrize.cached "torch.nn.utils.parametrize.cached")	 | 	 Context manager that enables the caching system within parametrizations registered with	 `register_parametrization()`	.	  |
| 	[`parametrize.is_parametrized`](generated/torch.nn.utils.parametrize.is_parametrized.html#torch.nn.utils.parametrize.is_parametrized "torch.nn.utils.parametrize.is_parametrized")	 | 	 Returns	 `True`	 if module has an active parametrization.	  |


|  |  |
| --- | --- |
| 	[`parametrize.ParametrizationList`](generated/torch.nn.utils.parametrize.ParametrizationList.html#torch.nn.utils.parametrize.ParametrizationList "torch.nn.utils.parametrize.ParametrizationList")	 | 	 A sequential container that holds and manages the	 `original`	 or	 `original0`	 ,	 `original1`	 ,.	  |


 以无状态方式调用给定模块的实用程序函数。


|  |  |
| --- | --- |
| 	[`stateless.functional_call`](generated/torch.nn.utils.stateless.functional_call.html#torch.nn.utils.stateless.functional_call "torch.nn.utils.stateless.functional_call")	 | 	 Performs a functional call on the module by replacing the module parameters and buffers with the provided ones.	  |


 其他模块中的实用函数


|  |  |
| --- | --- |
| 	[`nn.utils.rnn.PackedSequence`](generated/torch.nn.utils.rnn.PackedSequence.html#torch.nn.utils.rnn.PackedSequence "torch.nn.utils.rnn.PackedSequence")	 | 	 Holds the data and list of	 `batch_sizes`	 of a packed sequence.	  |
| 	[`nn.utils.rnn.pack_padded_sequence`](generated/torch.nn.utils.rnn.pack_padded_sequence.html#torch.nn.utils.rnn.pack_padded_sequence "torch.nn.utils.rnn.pack_padded_sequence")	 | 	 Packs a Tensor containing padded sequences of variable length.	  |
| 	[`nn.utils.rnn.pad_packed_sequence`](generated/torch.nn.utils.rnn.pad_packed_sequence.html#torch.nn.utils.rnn.pad_packed_sequence "torch.nn.utils.rnn.pad_packed_sequence")	 | 	 Pads a packed batch of variable length sequences.	  |
| 	[`nn.utils.rnn.pad_sequence`](generated/torch.nn.utils.rnn.pad_sequence.html#torch.nn.utils.rnn.pad_sequence "torch.nn.utils.rnn.pad_sequence")	 | 	 Pad a list of variable length Tensors with	 `padding_value`	 |
| 	[`nn.utils.rnn.pack_sequence`](generated/torch.nn.utils.rnn.pack_sequence.html#torch.nn.utils.rnn.pack_sequence "torch.nn.utils.rnn.pack_sequence")	 | 	 Packs a list of variable length Tensors	  |
| 	[`nn.utils.rnn.unpack_sequence`](generated/torch.nn.utils.rnn.unpack_sequence.html#torch.nn.utils.rnn.unpack_sequence "torch.nn.utils.rnn.unpack_sequence")	 | 	 Unpacks PackedSequence into a list of variable length Tensors	  |
| 	[`nn.utils.rnn.unpad_sequence`](generated/torch.nn.utils.rnn.unpad_sequence.html#torch.nn.utils.rnn.unpad_sequence "torch.nn.utils.rnn.unpad_sequence")	 | 	 Unpad padded Tensor into a list of variable length Tensors	  |


|  |  |
| --- | --- |
| 	[`nn.Flatten`](generated/torch.nn.Flatten.html#torch.nn.Flatten "torch.nn.Flatten")	 | 	 Flattens a contiguous range of dims into a tensor.	  |
| 	[`nn.Unflatten`](generated/torch.nn.Unflatten.html#torch.nn.Unflatten "torch.nn.Unflatten")	 | 	 Unflattens a tensor dim expanding it to a desired shape.	  |


## [量化函数](#id1) [¶](#quantized-functions "此标题的永久链接")


 量化是指以低于浮点精度的位宽执行计算和存储tensor的技术。 PyTorch 支持每tensor和每通道非对称线性量化。要了解如何在 PyTorch 中使用量化函数的更多信息，请参阅 [Quantization](quantization.html#quantization-doc) 文档。


## [延迟模块初始化](#id1) [¶](#lazy-modules-initialization "永久链接到此标题")


|  |  |
| --- | --- |
| 	[`nn.modules.lazy.LazyModuleMixin`](generated/torch.nn.modules.lazy.LazyModuleMixin.html#torch.nn.modules.lazy.LazyModuleMixin "torch.nn.modules.lazy.LazyModuleMixin")	 | 	 A mixin for modules that lazily initialize parameters, also known as "lazy modules."	  |