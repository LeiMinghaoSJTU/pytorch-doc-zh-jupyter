# torch.nn.function [¶](#torch-nn-function "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/nn.functional>
>
> 原始地址：<https://pytorch.org/docs/stable/nn.functional.html>


## 卷积函数 [¶](#卷积函数"此标题的永久链接")


|  |  |
| --- | --- |
| [`conv1d`](generated/torch.nn.function.conv1d.html#torch.nn.function.conv1d“torch.nn.function.conv1d”)|对由多个输入平面组成的输入信号应用一维卷积。 |
| [`conv2d`](generated/torch.nn.function.conv2d.html#torch.nn.function.conv2d“torch.nn.function.conv2d”)|对由多个输入平面组成的输入图像应用 2D 卷积。 |
| [`conv3d`](generated/torch.nn.function.conv3d.html#torch.nn.function.conv3d“torch.nn.function.conv3d”)|对由多个输入平面组成的输入图像应用 3D 卷积。 |
| [`conv_transpose1d`](generated/torch.nn.function.conv_transpose1d.html#torch.nn.function.conv_transpose1d“torch.nn.function.conv_transpose1d”) |对由多个输入平面组成的输入信号应用一维转置卷积算子，有时也称为“反卷积”。 |
| [`conv_transpose2d`](generated/torch.nn.function.conv_transpose2d.html#torch.nn.function.conv_transpose2d“torch.nn.function.conv_transpose2d”) |在由多个输入平面组成的输入图像上应用 2D 转置卷积算子，有时也称为“反卷积”。 |
| [`conv_transpose3d`](generated/torch.nn.function.conv_transpose3d.html#torch.nn.function.conv_transpose3d“torch.nn.function.conv_transpose3d”) |在由多个输入平面组成的输入图像上应用 3D 转置卷积算子，有时也称为“反卷积”|
| [`unfold`](generated/torch.nn.function.unfold.html#torch.nn.function.unfold "torch.nn.function.unfold") |从批量输入tensor中提取滑动局部块。 |
| [`fold`](generated/torch.nn.function.fold.html#torch.nn.function.fold "torch.nn.function.fold") |将一系列滑动局部块组合成一个大的包含tensor。 |


## 池化函数 [¶](#pooling-functions "此标题的永久链接")


|  |  |
| --- | --- |
| [`avg_pool1d`](generated/torch.nn.function.avg_pool1d.html#torch.nn.function.avg_pool1d“torch.nn.function.avg_pool1d”)|对由多个输入平面组成的输入信号应用一维平均池。 |
| [`avg_pool2d`](generated/torch.nn.function.avg_pool2d.html#torch.nn.function.avg_pool2d“torch.nn.function.avg_pool2d”) |应用 2D 平均池化操作


 k 高 × k 宽


 kHz × kW


 k
 

 H
 



 ×
 




 kW
 


 按步长划分的区域


 高 × 宽


 sh\乘以sW


 sH
 



 ×
 




 s
 

 W
 


 脚步。 |
| [`avg_pool3d`](generated/torch.nn.function.avg_pool3d.html#torch.nn.function.avg_pool3d“torch.nn.function.avg_pool3d”) |应用 3D 平均池化操作


 kT×kH×kW


 kT × kHz × kW


 k
 

 T
 



 ×
 




 k
 

 H
 



 ×
 




 kW
 


 按步长划分的区域


 s T × s H × s W


 sT \乘以sH \乘以SW


 s
 

 T
 



 ×
 




 sH
 



 ×
 




 s
 

 W
 


 脚步。 |
| [`max_pool1d`](generated/torch.nn.function.max_pool1d.html#torch.nn.function.max_pool1d“torch.nn.function.max_pool1d”)|对由多个输入平面组成的输入信号应用 1D 最大池化。 |
| [`max_pool2d`](generated/torch.nn.function.max_pool2d.html#torch.nn.function.max_pool2d“torch.nn.function.max_pool2d”) |对由多个输入平面组成的输入信号应用 2D 最大池化。 |
| [`max_pool3d`](generated/torch.nn.function.max_pool3d.html#torch.nn.function.max_pool3d“torch.nn.function.max_pool3d”) |对由多个输入平面组成的输入信号应用 3D 最大池化。 |
| [`max_unpool1d`](generated/torch.nn.function.max_unpool1d.html#torch.nn.function.max_unpool1d“torch.nn.function.max_unpool1d”) |计算 `MaxPool1d` 的部分逆。 |
| [`max_unpool2d`](generated/torch.nn.function.max_unpool2d.html#torch.nn.function.max_unpool2d“torch.nn.function.max_unpool2d”) |计算 `MaxPool2d` 的部分逆。 |
| [`max_unpool3d`](generated/torch.nn.function.max_unpool3d.html#torch.nn.function.max_unpool3d“torch.nn.function.max_unpool3d”) |计算 `MaxPool3d` 的部分逆。 |
| [`lp_pool1d`](generated/torch.nn.function.lp_pool1d.html#torch.nn.function.lp_pool1d“torch.nn.function.lp_pool1d”) |对由多个输入平面组成的输入信号应用一维功率平均池。 |
| [`lp_pool2d`](generated/torch.nn.function.lp_pool2d.html#torch.nn.function.lp_pool2d“torch.nn.function.lp_pool2d”) |对由多个输入平面组成的输入信号应用 2D 功率平均池化。 |
| [`adaptive_max_pool1d`](generated/torch.nn.function.adaptive_max_pool1d.html#torch.nn.function.adaptive_max_pool1d“torch.nn.function.adaptive_max_pool1d”)|对由多个输入平面组成的输入信号应用一维自适应最大池。 |
| [`adaptive_max_pool2d`](generated/torch.nn.function.adaptive_max_pool2d.html#torch.nn.function.adaptive_max_pool2d“torch.nn.function.adaptive_max_pool2d”)|对由多个输入平面组成的输入信号应用 2D 自适应最大池。 |
| [`adaptive_max_pool3d`](generated/torch.nn.function.adaptive_max_pool3d.html#torch.nn.function.adaptive_max_pool3d“torch.nn.function.adaptive_max_pool3d”)|对由多个输入平面组成的输入信号应用 3D 自适应最大池。 |
| [`adaptive_avg_pool1d`](generated/torch.nn.function.adaptive_avg_pool1d.html#torch.nn.function.adaptive_avg_pool1d“torch.nn.function.adaptive_avg_pool1d”)|对由多个输入平面组成的输入信号应用一维自适应平均池。 |
| [`adaptive_avg_pool2d`](generated/torch.nn.function.adaptive_avg_pool2d.html#torch.nn.function.adaptive_avg_pool2d“torch.nn.function.adaptive_avg_pool2d”)|对由多个输入平面组成的输入信号应用 2D 自适应平均池。 |
| [`adaptive_avg_pool3d`](generated/torch.nn.function.adaptive_avg_pool3d.html#torch.nn.function.adaptive_avg_pool3d“torch.nn.function.adaptive_avg_pool3d”)|对由多个输入平面组成的输入信号应用 3D 自适应平均池。 |
| [`fractional_max_pool2d`](generated/torch.nn.function.fractional_max_pool2d.html#torch.nn.function.fractional_max_pool2d“torch.nn.function.fractional_max_pool2d”)|对由多个输入平面组成的输入信号应用 2D 分数最大池化。 |
| [`fractional_max_pool3d`](generated/torch.nn.function.fractional_max_pool3d.html#torch.nn.function.fractional_max_pool3d“torch.nn.function.fractional_max_pool3d”)|对由多个输入平面组成的输入信号应用 3D 分数最大池化。 |


## 注意力机制 [¶](#attention-mechanisms "此标题的永久链接")


|  |  |
| --- | --- |
| [`scaled_dot_product_attention`](generated/torch.nn.function.scaled_dot_product_attention.html#torch.nn.function.scaled_dot_product_attention "torch.nn.function.scaled_dot_product_attention") |计算查询、键和值tensor上的缩放点积注意力，如果通过，则使用可选的注意力掩码，如果指定的概率大于 0.0，则应用 dropout。 |


## 非线性激活函数 [¶](#non-linear-activation-functions "永久链接到此标题")


|  |  |
| --- | --- |
| [`阈值`](generated/torch.nn.function.threshold.html#torch.nn.function.threshold“torch.nn.function.threshold”)|对输入tensor的每个元素设置阈值。 |
| [`阈值_`](generated/torch.nn.function.threshold_.html#torch.nn.function.threshold_ "torch.nn.function.threshold_") | [`threshold()`](generated/torch.nn.function.threshold.html#torch.nn.function.threshold "torch.nn.function.threshold") 的就地版本。 |
| [`relu`](generated/torch.nn.function.relu.html#torch.nn.function.relu“torch.nn.function.relu”)|按元素应用修正的线性单位函数。 |
| [`relu_`](generated/torch.nn.function.relu_.html#torch.nn.function.relu_ "torch.nn.function.relu_") | [`relu()`](generated/torch.nn.function.relu.html#torch.nn.function.relu "torch.nn.function.relu") 的就地版本。 |
| [`hardtanh`](generated/torch.nn.function.hardtanh.html#torch.nn.function.hardtanh“torch.nn.function.hardtanh”)|按元素应用 HardTanh 函数。 |
| [`hardtanh_`](generated/torch.nn.function.hardtanh_.html#torch.nn.function.hardtanh_ "torch.nn.function.hardtanh_") | [`hardtanh()`](generated/torch.nn.function.hardtanh.html#torch.nn.function.hardtanh "torch.nn.function.hardtanh") 的就地版本。 |
| [`hardswish`](generated/torch.nn.function.hardswish.html#torch.nn.function.hardswish“torch.nn.function.hardswish”)|按元素应用hardswish 函数，如论文中所述： |
| [`relu6`](generated/torch.nn.function.relu6.html#torch.nn.function.relu6“torch.nn.function.relu6”) |应用逐元素函数


 ReLU6 ( x ) = min ⁡ ( max ⁡ ( 0 , x ) , 6 )


 	ext{ReLU6}(x) = \min(\max(0,x), 6)



 ReLU6
 


 (  X  )



 =
 


 最小值(最大值(0，


 X  )  ，



 6
 

 )
 




.
  |
| 


[`elu`](generated/torch.nn.function.elu.html#torch.nn.function.elu“torch.nn.function.elu”) |按元素应用指数线性单元 (ELU) 函数。 |
| [`elu_`](generated/torch.nn.function.elu_.html#torch.nn.function.elu_ "torch.nn.function.elu_") | [`elu()`](generated/torch.nn.function.elu.html#torch.nn.function.elu "torch.nn.function.elu") 的就地版本。 |
| [`selu`](generated/torch.nn.function.selu.html#torch.nn.function.selu“torch.nn.function.selu”)|按元素应用，


 SELU ( x ) = 尺度
* ( max ⁡ ( 0 , x ) 
+ min ⁡ ( 0 , α 
* ( exp ⁡ ( x ) − 1 ) ) )


 	ext{SELU}(x) = 尺度 * (\max(0,x) 
+ \min(0, lpha * (\exp(x) 
- 1)))



 SELU
 


 (  X  )



 =
 


 规模



 ∗
 


 (最大(0，



 x
 

 )
 



 +
 


 分钟(0，



 α
 



 ∗
 


 ( 指数 ( x )



 −
 


 1)))


 ， 和


 α = 1.6732632423543772848170429916717


 lpha=1.6732632423543772848170429916717


 α
 



 =
 


 1.6732632423543772848170429916717




 and
 


 s  c  a  l  e  =  1.0507009873554804934193349852946


 规模=1.0507009873554804934193349852946


 规模



 =
 


 1.0507009873554804934193349852946




.
  |
| 


[`celu`](generated/torch.nn.function.celu.html#torch.nn.function.celu“torch.nn.function.celu”) |按元素应用，


 CELU ( x ) = 最大 ⁡ ( 0 , x ) 
+ 最小 ⁡ ( 0 , α 
* ( exp ⁡ ( x /α ) − 1 ) )


 	ext{CELU}(x) = \max(0,x) 
+ \min(0, lpha * (\exp(x/lpha) 
- 1))



 CELU
 


 (  X  )



 =
 


 最大(0，



 x
 

 )
 



 +
 


 分钟(0，



 α
 



 ∗
 


 (exp(x/α)



 −
 




 1
 

 ))
 




.
  |
| 


[`leaky_relu`](generated/torch.nn.function.leaky_relu.html#torch.nn.function.leaky_relu“torch.nn.function.leaky_relu”) |按元素应用，


 LeakyReLU ( x ) = max ⁡ ( 0 , x ) 
+ 负斜率 
* min ⁡ ( 0 , x )


 	ext{LeakyReLU}(x) = \max(0, x) 
+ 	ext{负\_斜率} * \min(0, x)


 LeakyReLU


 (  X  )



 =
 


 最大(0，



 x
 

 )
 



 +
 


 负_斜率




 ∗
 


 分钟(0，



 x
 

 )
 




 |
| 


[`leaky_relu_`](generated/torch.nn.function.leaky_relu_.html#torch.nn.function.leaky_relu_ "torch.nn.function.leaky_relu_") | [`leaky_relu()`](generated/torch.nn.function.leaky_relu.html#torch.nn.function.leaky_relu "torch.nn.function.leaky_relu") 的就地版本。 |
| [`prelu`](generated/torch.nn.function.prelu.html#torch.nn.function.prelu "torch.nn.function.prelu") |按元素应用函数


 PReLU ( x ) = max ⁡ ( 0 , x ) 
+ 权重 
* min ⁡ ( 0 , x )


 	ext{PReLU}(x) = \max(0,x) 
+ 	ext{权重} * \min(0,x)



 PReLU
 


 (  X  )



 =
 


 最大(0，



 x
 

 )
 



 +
 


 重量




 ∗
 


 分钟(0，



 x
 

 )
 


 其中权重是一个可学习的参数。 |
| [`rrelu`](generated/torch.nn.function.rrelu.html#torch.nn.function.rrelu“torch.nn.function.rrelu”)|随机泄漏 ReLU。 |
| [`rrelu_`](generated/torch.nn.function.rrelu_.html#torch.nn.function.rrelu_ "torch.nn.function.rrelu_") | [`rrelu()`](generated/torch.nn.function.rrelu.html#torch.nn.function.rrelu "torch.nn.function.rrelu") 的就地版本。 |
| [`glu`](generated/torch.nn.function.glu.html#torch.nn.function.glu“torch.nn.function.glu”)|门控线性单元。 |
| [`gelu`](generated/torch.nn.function.gelu.html#torch.nn.function.gelu "torch.nn.function.gelu") |当近似参数为“无”时，它按元素应用函数


 GELU(x)=x*Φ(x)


 	ext{GELU}(x) = x * \Phi(x)



 GELU
 


 (  X  )



 =
 




 x
 



 ∗
 


 Φ(x)




 |
| 


[`logsigmoid`](generated/torch.nn.function.logsigmoid.html#torch.nn.function.logsigmoid“torch.nn.function.logsigmoid”)|按元素应用


 对数Sigmoid (


 x
 

 i
 


 ) = 对数 ⁡


 (
 


 1
 


 1 
+ exp ⁡ ( −


 x
 

 i
 


 )
 



 )
 


 	ext{LogSigmoid}(x_i) = \log \left(rac{1}{1 
+ \exp(-x_i)}
ight)


 对数S形


 (
 


 x
 



 i
 




 ​
 


 )
 



 =
 




 lo
 
 g
 



 (
 


 1
 

 +
 


 经验值


 (
 

 −
 


 x
 



 i
 




 ​
 


 )
 



 1
 


 ​
 


 )
 



 |
| 


[`hardshrink`](generated/torch.nn.function.hardshrink.html#torch.nn.function.hardshrink“torch.nn.function.hardshrink”)|按元素应用硬收缩函数 |
| [`tanhshrink`](generated/torch.nn.function.tanhshrink.html#torch.nn.function.tanhshrink“torch.nn.function.tanhshrink”)|按元素应用，


 Tanhshrink ( x ) = x − Tanh ( x )


 	ext{Tanhshrink}(x) = x 
- 	ext{Tanhshrink}(x)


 坦赫收缩


 (  X  )



 =
 




 x
 



 −
 


 Tanh
 


 (  X  )




 |
| 


[`softsign`](generated/torch.nn.function.softsign.html#torch.nn.function.softsign“torch.nn.function.softsign”) |按元素应用，函数


 软签名 ( x ) =


 x
 


 1 
+ ∣ x ∣


 	ext{SoftSign}(x) = rac{x}{1 
+ |x|}


 软签名


 (  X  )



 =
 


 1 
+ ∣ x ∣



 x
 


 ​
 




 |
| 


[`softplus`](generated/torch.nn.function.softplus.html#torch.nn.function.softplus“torch.nn.function.softplus”)|按元素应用，函数


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
 




.
  |
| 


[`softmin`](generated/torch.nn.function.softmin.html#torch.nn.function.softmin“torch.nn.function.softmin”)|应用 softmin 函数。 |
| [`softmax`](generated/torch.nn.function.softmax.html#torch.nn.function.softmax“torch.nn.function.softmax”)|应用 softmax 函数。 |
| [`softshrink`](generated/torch.nn.function.softshrink.html#torch.nn.function.softshrink“torch.nn.function.softshrink”)|按元素应用软收缩函数 |
| [`gumbel_softmax`](generated/torch.nn.function.gumbel_softmax.html#torch.nn.function.gumbel_softmax "torch.nn.function.gumbel_softmax") |来自 Gumbel-Softmax 分布的样本([链接 1](https://arxiv.org/abs/1611.00712) [链接 2](https://arxiv.org/abs/1611.01144) )并可选择离散化。 |
| [`log_softmax`](generated/torch.nn.function.log_softmax.html#torch.nn.function.log_softmax "torch.nn.function.log_softmax") |应用 softmax，然后应用对数。 |
| [`tanh`](generated/torch.nn.function.tanh.html#torch.nn.function.tanh“torch.nn.function.tanh”)|按元素应用，


 tanh ( x ) = tanh ⁡ ( x ) =


 exp ⁡ ( x ) − exp ⁡ ( − x )


 exp ⁡ ( x ) 
+ exp ⁡ ( − x )


 	ext{Tanh}(x) = 	anh(x) = rac{\exp(x) 
- \exp(-x)}{\exp(x) 
+ \exp(-x)}



 Tanh
 


 (  X  )



 =
 


 腥味 ( x )



 =
 


 经验值


 (x) +


 经验值


 (−x)


 经验值


 ( x ) −


 经验值


 (−x)


 ​
 




 |
| 


[`sigmoid`](generated/torch.nn.function.sigmoid.html#torch.nn.function.sigmoid“torch.nn.function.sigmoid”) |应用逐元素函数


 乙状结肠 ( x ) =


 1
 


 1 
+ exp ⁡ ( − x )


 	ext{Sigmoid}(x) = rac{1}{1 
+ \exp(-x)}


 乙状结肠


 (  X  )



 =
 




 1
 

 +
 


 经验值


 (−x)



 1
 


 ​
 




 |
| 


[`hardsigmoid`](generated/torch.nn.function.hardsigmoid.html#torch.nn.function.hardsigmoid“torch.nn.function.hardsigmoid”)|应用逐元素函数 |
| [`silu`](generated/torch.nn.function.silu.html#torch.nn.function.silu“torch.nn.function.silu”)|按元素应用 Sigmoid 线性单元 (SiLU) 函数。 |
| [`mish`](generated/torch.nn.function.mish.html#torch.nn.function.mish“torch.nn.function.mish”)|按元素应用 Mish 函数。 |
| [`batch_norm`](generated/torch.nn.function.batch_norm.html#torch.nn.function.batch_norm“torch.nn.function.batch_norm”) |对一批数据中的每个通道应用批量归一化。 |
| [`group_norm`](generated/torch.nn.function.group_norm.html#torch.nn.function.group_norm“torch.nn.function.group_norm”) |对最后一定数量的维度应用组标准化。 |
| [`instance_norm`](generated/torch.nn.function.instance_norm.html#torch.nn.function.instance_norm“torch.nn.function.instance_norm”) |对批次中每个数据样本中的每个通道应用实例标准化。 |
| [`layer_norm`](generated/torch.nn.function.layer_norm.html#torch.nn.function.layer_norm "torch.nn.function.layer_norm") |对最后一定数量的维度应用层归一化。 |
| [`local_response_norm`](generated/torch.nn.function.local_response_norm.html#torch.nn.function.local_response_norm“torch.nn.function.local_response_norm”) |对由多个输入平面组成的输入信号应用局部响应归一化，其中通道占据第二维。 |
| [`normalize`](generated/torch.nn.function.normalize.html#torch.nn.function.normalize "torch.nn.function.normalize") |施行




 L
 

 p
 



 L\_p
 



 L
 



 p
 




 ​
 


 指定维度上输入的标准化。 |


## 线性函数 [¶](#线性函数"此标题的永久链接")


|  |  |
| --- | --- |
| [`线性`](generated/torch.nn.function.linear.html#torch.nn.function.linear“torch.nn.function.linear”) |对传入数据应用线性变换：


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
 




.
  |
| 


[`双线性`](generated/torch.nn.function.bilinear.html#torch.nn.function.bilinear“torch.nn.function.bilinear”)|对传入数据应用双线性变换：



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


## Dropout 函数 [¶](#dropout-functions "固定链接到此标题")


|  |  |
| --- | --- |
| [`dropout`](generated/torch.nn.function.dropout.html#torch.nn.function.dropout“torch.nn.function.dropout”) |在训练期间，使用伯努利分布中的样本以概率“p”将输入tensor的一些元素随机归零。 |
| [`alpha_dropout`](generated/torch.nn.function.alpha_dropout.html#torch.nn.function.alpha_dropout "torch.nn.function.alpha_dropout") |对输入应用 alpha dropout。 |
| [`feature_alpha_dropout`](generated/torch.nn.function.feature_alpha_dropout.html#torch.nn.function.feature_alpha_dropout "torch.nn.function.feature_alpha_dropout") |随机屏蔽整个通道(通道是一个特征图，例如 |
| [`dropout1d`](generated/torch.nn.function.dropout1d.html#torch.nn.function.dropout1d“torch.nn.function.dropout1d”) |随机将整个通道归零(通道是一维特征图，例如



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
 


 ) 输入tensor)。 |
| [`dropout2d`](generated/torch.nn.function.dropout2d.html#torch.nn.function.dropout2d“torch.nn.function.dropout2d”) |随机将整个通道归零(通道是一个 2D 特征图，例如



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
 


 ) 输入tensor)。 |
| [`dropout3d`](generated/torch.nn.function.dropout3d.html#torch.nn.function.dropout3d“torch.nn.function.dropout3d”) |随机将整个通道归零(通道是 3D 特征图，例如



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
 


 ) 输入tensor)。 |


## 稀疏函数 [¶](#sparse-functions "此标题的固定链接")


|  |  |
| --- | --- |
| [`嵌入`](generated/torch.nn.function.embedding.html#torch.nn.function.embedding“torch.nn.function.embedding”)|一个简单的查找表，用于在固定字典和大小中查找嵌入。 |
| [`embedding_bag`](generated/torch.nn.function.embedding_bag.html#torch.nn.function.embedding_bag“torch.nn.function.embedding_bag”) |计算嵌入包的总和、平均值或最大值，而不实例化中间嵌入。 |
| [`one_hot`](generated/torch.nn.function.one_hot.html#torch.nn.function.one_hot "torch.nn.function.one_hot") |获取形状为“(*)”索引值的 LongTensor 并返回形状为“(*, num_classes)”的tensor，除了最后一个维度的索引与输入tensor的相应值匹配之外，该tensor到处都是零，在在哪种情况下它将是 1。


## 距离函数 [¶](#distance-functions "此标题的固定链接")


|  |  |
| --- | --- |
| [`pairwise_distance`](generated/torch.nn.function.pairwise_distance.html#torch.nn.function.pairwise_distance "torch.nn.function.pairwise_distance") |有关详细信息，请参阅 [`torch.nn.PairwiseDistance`](generated/torch.nn.PairwiseDistance.html#torch.nn.PairwiseDistance "torch.nn.PairwiseDistance") |
| [`cosine_similarity`](generated/torch.nn.function.cosine_similarity.html#torch.nn.function.cosine_similarity "torch.nn.function.cosine_similarity") |返回 `x1` 和 `x2` 之间的余弦相似度，沿着 dim 计算。 |
| [`pdist`](generated/torch.nn.function.pdist.html#torch.nn.function.pdist“torch.nn.function.pdist”)|计算输入中每​​对行向量之间的 p 范数距离。 |


## 损失函数 [¶](#loss-functions "此标题的永久链接")


|  |  |
| --- | --- |
| [`binary_cross_entropy`](generated/torch.nn.function.binary_cross_entropy.html#torch.nn.function.binary_cross_entropy "torch.nn.function.binary_cross_entropy") |测量目标概率和输入概率之间的二元交叉熵的函数。 |
| [`binary_cross_entropy_with_logits`](generated/torch.nn.function.binary_cross_entropy_with_logits.html#torch.nn.function.binary_cross_entropy_with_logits“torch.nn.function.binary_cross_entropy_with_logits”) |测量目标和输入 logits 之间的二元交叉熵的函数。 |
| [`poisson_nll_loss`](generated/torch.nn.function.poisson_nll_loss.html#torch.nn.function.poisson_nll_loss "torch.nn.function.poisson_nll_loss") |泊松负对数似然损失。 |
| [`cosine_embedding_loss`](generated/torch.nn.function.cosine_embedding_loss.html#torch.nn.function.cosine_embedding_loss "torch.nn.function.cosine_embedding_loss") |有关详细信息，请参阅 [`CosineEmbeddingLoss`](generated/torch.nn.CosineEmbeddingLoss.html#torch.nn.CosineEmbeddingLoss "torch.nn.CosineEmbeddingLoss")。 |
| [`cross_entropy`](generated/torch.nn.function.cross_entropy.html#torch.nn.function.cross_entropy "torch.nn.function.cross_entropy") |该标准计算输入 logits 和目标之间的交叉熵损失。 |
| [`ctc_loss`](generated/torch.nn.function.ctc_loss.html#torch.nn.function.ctc_loss "torch.nn.function.ctc_loss") |联结主义时间分类损失。 |
| [`gaussian_nll_loss`](generated/torch.nn.function.gaussian_nll_loss.html#torch.nn.function.gaussian_nll_loss "torch.nn.function.gaussian_nll_loss") |高斯负对数似然损失。 |
| [`hinge_embedding_loss`](generated/torch.nn.function.hinge_embedding_loss.html#torch.nn.function.hinge_embedding_loss "torch.nn.function.hinge_embedding_loss") |有关详细信息，请参阅 [`HingeEmbeddingLoss`](generated/torch.nn.HingeEmbeddingLoss.html#torch.nn.HingeEmbeddingLoss "torch.nn.HingeEmbeddingLoss")。 |
| [`cl_div`](generated/torch.nn.function.cl_div.html#torch.nn.function.cl_div“torch.nn.function.cl_div”) | [Kullback-Leibler 散度损失](https://en.wikipedia.org/wiki/Kullback-Leibler_Divergence) |
| [`l1_loss`](generated/torch.nn.function.l1_loss.html#torch.nn.function.l1_loss "torch.nn.function.l1_loss") |取平均元素绝对值差的函数。 |
| [`mse_loss`](generated/torch.nn.function.mse_loss.html#torch.nn.function.mse_loss "torch.nn.function.mse_loss") |测量元素均方误差。 |
| [`margin_ranking_loss`](generated/torch.nn.function.margin_ranking_loss.html#torch.nn.function.margin_ranking_loss "torch.nn.function.margin_ranking_loss") |有关详细信息，请参阅 [`MarginRankingLoss`](generated/torch.nn.MarginRankingLoss.html#torch.nn.MarginRankingLoss "torch.nn.MarginRankingLoss")。 |
| [`multilabel_margin_loss`](generated/torch.nn.function.multilabel_margin_loss.html#torch.nn.function.multilabel_margin_loss "torch.nn.function.multilabel_margin_loss") |有关详细信息，请参阅 [`MultiLabelMarginLoss`](generated/torch.nn.MultiLabelMarginLoss.html#torch.nn.MultiLabelMarginLoss "torch.nn.MultiLabelMarginLoss")。 |
| [`multilabel_soft_margin_loss`](generated/torch.nn.function.multilabel_soft_margin_loss.html#torch.nn.function.multilabel_soft_margin_loss "torch.nn.function.multilabel_soft_margin_loss") |有关详细信息，请参阅 [`MultiLabelSoftMarginLoss`](generated/torch.nn.MultiLabelSoftMarginLoss.html#torch.nn.MultiLabelSoftMarginLoss "torch.nn.MultiLabelSoftMarginLoss")。 |
| [`multi_margin_loss`](generated/torch.nn.function.multi_margin_loss.html#torch.nn.function.multi_margin_loss "torch.nn.function.multi_margin_loss") |有关详细信息，请参阅 [`MultiMarginLoss`](generated/torch.nn.MultiMarginLoss.html#torch.nn.MultiMarginLoss "torch.nn.MultiMarginLoss")。 |
| [`nll_loss`](generated/torch.nn.function.nll_loss.html#torch.nn.function.nll_loss "torch.nn.function.nll_loss") |负对数似然损失。 |
| [`huber_loss`](generated/torch.nn.function.huber_loss.html#torch.nn.function.huber_loss "torch.nn.function.huber_loss") |如果绝对元素误差低于 delta，则使用平方项，否则使用 delta 缩放的 L1 项。 |
| [`smooth_l1_loss`](generated/torch.nn.function.smooth_l1_loss.html#torch.nn.function.smooth_l1_loss "torch.nn.function.smooth_l1_loss") |如果绝对元素误差低于 beta，则使用平方项，否则使用 L1 项。 |
| [`soft_margin_loss`](generated/torch.nn.function.soft_margin_loss.html#torch.nn.function.soft_margin_loss "torch.nn.function.soft_margin_loss") |有关详细信息，请参阅 [`SoftMarginLoss`](generated/torch.nn.SoftMarginLoss.html#torch.nn.SoftMarginLoss "torch.nn.SoftMarginLoss")。 |
| [`triplet_margin_loss`](generated/torch.nn.function.triplet_margin_loss.html#torch.nn.function.triplet_margin_loss "torch.nn.function.triplet_margin_loss") |有关详细信息，请参阅 [`TripletMarginLoss`](generated/torch.nn.TripletMarginLoss.html#torch.nn.TripletMarginLoss "torch.nn.TripletMarginLoss") |
| [`triplet_margin_with_distance_loss`](generated/torch.nn.function.triplet_margin_with_distance_loss.html#torch.nn.function.triplet_margin_with_distance_loss "torch.nn.function.triplet_margin_with_distance_loss") |有关详细信息，请参阅 [`TripletMarginWithDistanceLoss`](generated/torch.nn.TripletMarginWithDistanceLoss.html#torch.nn.TripletMarginWithDistanceLoss "torch.nn.TripletMarginWithDistanceLoss")。 |


## 视觉功能 [¶](#vision-functions "此标题的固定链接")


|  |  |
| --- | --- |
| [`pixel_shuffle`](generated/torch.nn.function.pixel_shuffle.html#torch.nn.function.pixel_shuffle "torch.nn.function.pixel_shuffle") |重新排列形状tensor中的元素


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
 


 ，其中 r 是 `upscale_factor` 。 |
| [`pixel_unshuffle`](generated/torch.nn.function.pixel_unshuffle.html#torch.nn.function.pixel_unshuffle "torch.nn.function.pixel_unshuffle") |通过重新排列形状tensor中的元素来反转 [`PixelShuffle`](generated/torch.nn.PixelShuffle.html#torch.nn.PixelShuffle "torch.nn.PixelShuffle") 操作


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
 


 ，其中 r 是 `downscale_factor` 。 |
| [`pad`](generated/torch.nn.function.pad.html#torch.nn.function.pad“torch.nn.function.pad”) |垫tensor。 |
| [`插值`](generated/torch.nn.function.interpolate.html#torch.nn.function.interpolate“torch.nn.function.interpolate”)|向下/向上采样输入到给定的“size”或给定的“scale_factor”|
| [`upsample`](generated/torch.nn.function.upsample.html#torch.nn.function.upsample "torch.nn.function.upsample") |将输入上采样到给定的“size”或给定的“scale_factor”|
| [`upsample_nearest`](generated/torch.nn.function.upsample_nearest.html#torch.nn.function.upsample_nearest“torch.nn.function.upsample_nearest”) |使用最近邻居的像素值对输入进行上采样。 |
| [`upsample_bilinear`](generated/torch.nn.function.upsample_bilinear.html#torch.nn.function.upsample_bilinear "torch.nn.function.upsample_bilinear") |使用双线性上采样对输入进行上采样。 |
| [`grid_sample`](generated/torch.nn.function.grid_sample.html#torch.nn.function.grid_sample "torch.nn.function.grid_sample") |给定“输入”和流场“网格”，使用“输入”值和“网格”中的像素位置计算“输出”。 |
| [`affine_grid`](generated/torch.nn.function.affine_grid.html#torch.nn.function.affine_grid "torch.nn.function.affine_grid") |给定一批仿射矩阵 `theta` ，生成 2D 或 3D 流场(采样网格)。 |


## DataParallel 函数(多 GPU，分布式)[¶](#dataparallel-functions-multi-gpu-distributed "此标题的永久链接")


### data_parallel [¶](#data-parallel "此标题的永久链接")


|  |  |
| --- | --- |
| 	`torch.nn.parallel.data_parallel`	 | 	 Evaluates module(input) in parallel across the GPUs given in device_ids.	  |