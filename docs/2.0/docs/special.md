# torch.special [¶](#torch-special "此标题的固定链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/special>
>
> 原始地址：<https://pytorch.org/docs/stable/special.html>


 torch.special 模块，模仿 SciPy 的 [special](https://docs.scipy.org/doc/scipy/reference/special.html) 模块。


## 函数 [¶](#functions "此标题的永久链接")


 火炬.特别。


 通风_ai


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.airy_ai"此定义的永久链接")


 通风功能



 Ai
 


 (  输入  )


 	ext{Ai}\left(	ext{输入}
ight)



 Ai
 


 (
 


 input
 


 )
 


.
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 火炬.特别。


 贝塞尔_j0


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.bessel_j0"此定义的永久链接")


 第一类贝塞尔函数



 0
 


 0
 


 0
 




.
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 火炬.特别。


 贝塞尔_j1


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.bessel_j1"此定义的永久链接")


 第一类贝塞尔函数



 1
 


 1
 


 1
 




.
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 火炬.特别。


 迪伽玛


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.digamma"此定义的永久链接")


 计算输入 上伽玛函数的对数导数。


 ϝ ( x ) =


 d
 


 d
 

 x
 



 ln
 

 ⁡
 


 (
 

 Γ
 


 (  X  )


 )
 


 =
 




 Γ
 

 ′
 


 (  X  )


 C(x)


 \digamma(x) = rac{d}{dx} \ln\left(\Gamma\left(x
ight)
ight) = rac{\Gamma'(x)}{\Gamma(x)}


 ϝ ( x )



 =
 



 d
 

 x
 




 d
 




 ​
 



 ln
 




 (
 

 Γ
 


 (  X  )


 )
 




 =
 


 C(x)


 Γ
 




 ′
 


 (  X  )




 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 用于计算 digamma 函数的tensor


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。



!!! note "笔记"

    该函数类似于SciPy的scipy.special.digamma


!!! note "笔记"

    从 PyTorch 1.8 开始，digamma 函数返回 -Inf 表示 0 。之前它返回 NaN 表示 0 。


 例子：


```
>>> a = torch.tensor([1, 0.5])
>>> torch.special.digamma(a)
tensor([-0.5772, -1.9635])

```


 火炬.特别。



 entr
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.entr"此定义的永久链接")


 按元素计算“输入”(如下定义)的熵。


 输入(x)=


 {
 


 − x ∗ ln ⁡ ( x )


 x 
> 0


 0
 


 x = 0.0



 −
 

 ∞
 


 x < 0


 egin{align}	ext{entr(x)} = egin{cases} -x * \ln(x) & x 
> 0 \ 0 & x = 0.0 \ -\infty & x < 0\end {案例}\结束{对齐}


 输入(x)




 =
 




 ⎩
 


 ⎨
 


 ⎧
 




 ​
 



 −
 

 x
 



 ∗
 


 ln(x)




 0
 




 −
 

 ∞
 




 ​
 


 x
 



 >
 



 0
 




 x
 



 =
 



 0.0
 




 x
 



 <
 



 0
 




 ​
 



 ​
 


 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> a = torch.arange(-0.5, 1, 0.5)
>>> a
tensor([-0.5000, 0.0000, 0.5000])
>>> torch.special.entr(a)
tensor([ -inf, 0.0000, 0.3466])

```


 火炬.特别。



 erf
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.erf"此定义的永久链接")


 计算 `input` 的误差函数。误差函数定义如下：


 erf


 ( x ) =


 2
 


 π
 


 ∫ 0×



 e
 


 −
 


 t
 

 2
 




 d
 

 t
 


 \mathrm{erf}(x) = rac{2}{\sqrt{\pi}} \int_{0}^{x} e^{-t^2} dt



 erf
 


 (  X  )



 =
 


 π
 


 ​
 




 2
 




 ​
 




 ∫
 




 0
 



 x
 


 ​
 


 e
 




 −
 


 t
 



 2
 




 d
 

 t
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> torch.special.erf(torch.tensor([0, -1., 10.]))
tensor([ 0.0000, -0.8427, 1.0000])

```


 火炬.特别。



 erfc
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.erfc"此定义的永久链接")


 计算“输入”的互补误差函数。互补误差函数定义如下：


 erfc


 ( x ) = 1 −


 2
 


 π
 


 ∫ 0×



 e
 


 −
 


 t
 

 2
 




 d
 

 t
 


 \mathrm{erfc}(x) = 1 
- rac{2}{\sqrt{\pi}} \int_{0}^{x} e^{-t^2} dt



 erfc
 


 (  X  )



 =
 




 1
 



 −
 


 π
 


 ​
 




 2
 




 ​
 




 ∫
 




 0
 



 x
 


 ​
 


 e
 




 −
 


 t
 



 2
 




 d
 

 t
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> torch.special.erfc(torch.tensor([0, -1., 10.]))
tensor([ 1.0000, 1.8427, 0.0000])

```


 火炬.特别。



 erfcx
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.erfcx"此定义的永久链接")


 计算“输入”每个元素的缩放互补误差函数。缩放互补误差函数定义如下：


 er f c x


 ( x ) =


 e
 


 x
 

 2
 


 erfc


 (  X  )


 \mathrm{erfcx}(x) = e^{x^2} \mathrm{erfc}(x)



 erfcx
 


 (  X  )



 =
 


 e
 


 x
 



 2
 


 erfc
 


 (  X  )


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> torch.special.erfcx(torch.tensor([0, -1., 10.]))
tensor([ 1.0000, 5.0090, 0.0561])

```


 火炬.特别。


 埃尔芬夫


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.erfinv"此定义的永久链接")


 计算“输入”的逆误差函数。逆误差函数定义在范围内


 ( − 1 , 1 )


 (-1, 1)


 ( − 1 ,



 1
 

 )
 




 as:
 


 erfinv


 (
 


 erf


 ( x ) ) = x


 \mathrm{erfinv}(\mathrm{erf}(x)) = x


 埃尔芬夫


 (
 


 erf
 


 (  X  ))



 =
 




 x
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> torch.special.erfinv(torch.tensor([0, 0.5, -1.]))
tensor([ 0.0000, 0.4769, -inf])

```


 火炬.特别。



 exp2
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.exp2"此定义的永久链接")


 计算 `input` 的以二为底的指数函数。


 y
 

 i
 


 =
 


 2
 


 x
 

 i
 


 y_{i} = 2^{x_{i}}



 y
 




 i
 


 ​
 




 =
 


 2
 


 x
 




 i
 


 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> torch.special.exp2(torch.tensor([0, math.log2(2.), 3, 4]))
tensor([ 1., 2., 8., 16.])

```


 火炬.特别。



 expit
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.expit"此定义的永久链接")


 计算 `input` 元素的输出(也称为逻辑 sigmoid 函数)。


 出我


 =
 


 1
 


 1
 

 +
 


 e
 


 −
 


 输入我


 	ext{输出}_{i} = rac{1}{1 
+ e^{-	ext{输入}_{i}}}




 out
 


 i
 


 ​
 




 =
 



 1
 



 +
 




 e
 




 −
 



 input
 


 i
 


 ​
 




 1
 




 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> t = torch.randn(4)
>>> t
tensor([ 0.9213, 1.0887, -0.8858, -1.7683])
>>> torch.special.expit(t)
tensor([ 0.7153, 0.7481, 0.2920, 0.1458])

```


 火炬.特别。



 expm1
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.expm1"此定义的永久链接")


 计算元素的指数减去 `input` 的 1。


 y
 

 i
 


 =
 


 e
 


 x
 

 i
 



 −
 

 1
 


 y_{i} = e^{x_{i}} 
- 1



 y
 




 i
 


 ​
 




 =
 


 e
 


 x
 




 i
 


 ​
 



 −
 




 1
 




!!! note "笔记"

    对于较小的 x 值，此函数提供比 exp(x) 
- 1 更高的精度。


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> torch.special.expm1(torch.tensor([0, math.log(2.)]))
tensor([ 0., 1.])

```


 火炬.特别。


 伽玛克


 ( *输入
* , *其他
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.gammainc"此定义的永久链接")


 计算正则化下不完全伽马函数：


 出我


 =
 


 1
 


 Γ
 

 (
 


 输入我


 )
 




 ∫
 

 0
 


 其他我




 t
 


 输入我


 −
 

 1
 




 e
 


 −
 

 t
 



 d
 

 t
 


 	ext{out}_{i} = rac{1}{\Gamma(	ext{input}_i)} \int_0^{	ext{other}_i} t^{	ext{input }_i-1} e^{-t} dt




 out
 


 i
 


 ​
 




 =
 



 Γ
 

 (
 



 input
 




 i
 




 ​
 


 )
 




 1
 




 ​
 




 ∫
 



 0
 




 other
 




 i
 




 ​
 



 ​
 


 t
 



 input
 




 i
 




 ​
 


 −
 

 1
 




 e
 




 −
 

 t
 



 d
 

 t
 


 两者都在哪里


 输入我


 	ext{输入}_i




 input
 




 i
 




 ​
 


 and
 


 其他我


 	ext{其他}_i




 other
 




 i
 




 ​
 


 是弱正数，并且至少有一个是严格正数。如果两者都为零或其中一个为负数，则


 出我


 = 在


 	ext{out}_i=	ext{nan}




 out
 




 i
 




 ​
 




 =
 


 nan
 


.
 


 C (·)


 \伽玛(\cdot)


 C (·)


 上式中是伽玛函数，




 Γ
 

 (
 


 输入我


 )
 

 =
 


 ∫ 0 ∞



 t
 


 (
 


 输入我


 − 1 )




 e
 


 −
 

 t
 


 d t 。


 \Gamma(	ext{输入}_i) = \int_0^\infty t^{(	ext{输入}_i-1)} e^{-t} dt。


 Γ
 

 (
 



 input
 




 i
 




 ​
 


 )
 



 =
 


 ∫
 



 0
 




 ∞
 




 ​
 


 t
 




 (
 



 input
 




 i
 




 ​
 


 − 1 )




 e
 




 −
 

 t
 


 d t 。


 请参阅 [`torch.special.gammaincc()`](#torch.special.gammaincc "torch.special.gammaincc") 和 [`torch.special.gammaln()`](#torch.special.gammaln "torch.special.gammaln") 相关功能。


 支持[广播到通用形状](notes/broadcasting.html#broadcasting-semantics) 和浮动输入。




!!! note "笔记"

    尚不支持关于“输入”的向后传递。请在 PyTorch 的 Github 上打开一个问题来请求它。


 参数 
* **input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 第一个非负输入tensor
* **其他** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 第二个非负输入tensor


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> a1 = torch.tensor([4.0])
>>> a2 = torch.tensor([3.0, 4.0, 5.0])
>>> a = torch.special.gammaincc(a1, a2)
tensor([0.3528, 0.5665, 0.7350])
tensor([0.3528, 0.5665, 0.7350])
>>> b = torch.special.gammainc(a1, a2) + torch.special.gammaincc(a1, a2)
tensor([1., 1., 1.])

```


 火炬.特别。


 伽马公司


 ( *输入
* , *其他
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.gammaincc"此定义的永久链接")


 计算正则化上不完全伽玛函数：


 出我


 =
 


 1
 


 Γ
 

 (
 


 输入我


 )
 




 ∫
 


 其他我


 ∞
 



 t
 


 输入我


 −
 

 1
 




 e
 


 −
 

 t
 



 d
 

 t
 


 	ext{out}_{i} = rac{1}{\Gamma(	ext{input}_i)} \int_{	ext{other}_i}^{\infty} t^{ 	ext{输入}_i-1} e^{-t} dt




 out
 


 i
 


 ​
 




 =
 



 Γ
 

 (
 



 input
 




 i
 




 ​
 


 )
 




 1
 




 ​
 




 ∫
 



 other
 




 i
 




 ​
 




 ∞
 


 ​
 


 t
 



 input
 




 i
 




 ​
 


 −
 

 1
 




 e
 




 −
 

 t
 



 d
 

 t
 


 两者都在哪里


 输入我


 	ext{输入}_i




 input
 




 i
 




 ​
 


 and
 


 其他我


 	ext{其他}_i




 other
 




 i
 




 ​
 


 是弱正数，并且至少有一个是严格正数。如果两者都为零或其中一个为负数，则


 出我


 = 在


 	ext{out}_i=	ext{nan}




 out
 




 i
 




 ​
 




 =
 


 nan
 


.
 


 C (·)


 \伽玛(\cdot)


 C (·)


 上式中是伽玛函数，




 Γ
 

 (
 


 输入我


 )
 

 =
 


 ∫ 0 ∞



 t
 


 (
 


 输入我


 − 1 )




 e
 


 −
 

 t
 


 d t 。


 \Gamma(	ext{输入}_i) = \int_0^\infty t^{(	ext{输入}_i-1)} e^{-t} dt。


 Γ
 

 (
 



 input
 




 i
 




 ​
 


 )
 



 =
 


 ∫
 



 0
 




 ∞
 




 ​
 


 t
 




 (
 



 input
 




 i
 




 ​
 


 − 1 )




 e
 




 −
 

 t
 


 d t 。


 请参阅 [`torch.special.gammainc()`](#torch.special.gammainc "torch.special.gammainc") 和 [`torch.special.gammaln()`](#torch.special.gammaln "torch.special.gammaln") 相关功能。


 支持[广播到通用形状](notes/broadcasting.html#broadcasting-semantics) 和浮动输入。




!!! note "笔记"

    尚不支持关于“输入”的向后传递。请在 PyTorch 的 Github 上打开一个问题来请求它。


 参数 
* **input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 第一个非负输入tensor
* **其他** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 第二个非负输入tensor


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> a1 = torch.tensor([4.0])
>>> a2 = torch.tensor([3.0, 4.0, 5.0])
>>> a = torch.special.gammaincc(a1, a2)
tensor([0.6472, 0.4335, 0.2650])
>>> b = torch.special.gammainc(a1, a2) + torch.special.gammaincc(a1, a2)
tensor([1., 1., 1.])

```


 火炬.特别。


 旧的


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.gammaln"此定义的永久链接")


 计算 `input` 上 gamma 函数绝对值的自然对数。


 出我


 = ln ⁡ Γ ( ∣


 输入我


 ∣
 

 )
 


 	ext{输出}_{i} = \ln \Gamma(|	ext{输入}_{i}|)




 out
 


 i
 


 ​
 




 =
 




 ln
 


 C ( ∣



 input
 


 i
 


 ​
 


 ∣
 

 )
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> a = torch.arange(0.5, 2, 0.5)
>>> torch.special.gammaln(a)
tensor([ 0.5724, 0.0000, -0.1208])

```


 火炬.特别。



 i0
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.i0"此定义的永久链接")


 计算 `input` 的每个元素的第一类零阶修正贝塞尔函数。


 出我


 =
 


 I
 

 0
 


 (
 


 输入我


 )
 

 =
 


 ∑
 


 k = 0


 ∞
 




 (
 


 输入 i 2


 /
 

 4
 


 )
 

 k
 


 ( k !


 )
 

 2
 


 	ext{out}_{i} = I_0(	ext{输入}_{i}) = \sum_{k=0}^{\infty} rac{(	ext{输入} _{i}^2/4)^k}{(k!)^2}




 out
 


 i
 


 ​
 




 =
 


 I
 



 0
 




 ​
 


 (
 



 input
 


 i
 


 ​
 


 )
 



 =
 


 k = 0


 ∑
 


 ∞
 


 ​
 


 ( k !


 )
 



 2
 


 (
 



 input
 


 i
 


 2
 




 ​
 


 /4
 


 )
 



 k
 


 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> torch.i0(torch.arange(5, dtype=torch.float32))
tensor([ 1.0000, 1.2661, 2.2796, 4.8808, 11.3019])

```


 火炬.特别。



 i0e
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.i0e"此定义的永久链接")


 为 `input` 的每个元素计算第一类指数缩放的零阶修正贝塞尔函数(如下定义)。


 出我


 = exp ⁡ ( − ∣ x ∣ ) ∗ i 0 ( x ) = exp ⁡ ( − ∣ x ∣ ) ∗


 ∑
 


 k = 0


 ∞
 




 (
 


 输入 i 2


 /
 

 4
 


 )
 

 k
 


 ( k !


 )
 

 2
 


 	ext{out}_{i} = \exp(-|x|) * i0(x) = \exp(-|x|) * \sum_{k=0}^{\infty} rac{(	ext{输入}_{i}^2/4)^k}{(k!)^2}




 out
 


 i
 


 ​
 




 =
 


 exp ( − ∣ x ∣ )



 ∗
 


 我 0 ( x )



 =
 


 exp ( − ∣ x ∣ )



 ∗
 


 k = 0


 ∑
 


 ∞
 


 ​
 


 ( k !


 )
 



 2
 


 (
 



 input
 


 i
 


 2
 




 ​
 


 /4
 


 )
 



 k
 


 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> torch.special.i0e(torch.arange(5, dtype=torch.float32))
tensor([1.0000, 0.4658, 0.3085, 0.2430, 0.2070])

```


 火炬.特别。



 i1
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.i1"此定义的永久链接")


 计算 `input` 的每个元素的第一类一阶修正贝塞尔函数(如下定义)。


 出我


 =
 



 (
 


 输入我


 )
 


 2
 


 ∗
 


 ∑
 


 k = 0


 ∞
 




 (
 


 输入 i 2


 /
 

 4
 


 )
 

 k
 


 (k!)*(k+1)!


 	ext{out}_{i} = rac{(	ext{输入}_{i})}{2} * \sum_{k=0}^{\infty} rac{( 	ext{输入}_{i}^2/4)^k}{(k!) * (k+1)!}




 out
 


 i
 


 ​
 




 =
 



 2
 




 (
 



 input
 


 i
 


 ​
 


 )
 




 ​
 



 ∗
 


 k = 0


 ∑
 


 ∞
 


 ​
 


 (克！)



 ∗
 



 (
 

 k
 



 +
 



 1
 

 )!
 




 (
 



 input
 


 i
 


 2
 




 ​
 


 /4
 


 )
 



 k
 


 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> torch.special.i1(torch.arange(5, dtype=torch.float32))
tensor([0.0000, 0.5652, 1.5906, 3.9534, 9.7595])

```


 火炬.特别。



 i1e
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.i1e"此定义的永久链接")


 计算 `input` 的每个元素的第一类指数缩放一阶修正贝塞尔函数(如下定义)。


 出我


 = exp ⁡ ( − ∣ x ∣ ) ∗ i 1 ( x ) = exp ⁡ ( − ∣ x ∣ ) ∗



 (
 


 输入我


 )
 


 2
 


 ∗
 


 ∑
 


 k = 0


 ∞
 




 (
 


 输入 i 2


 /
 

 4
 


 )
 

 k
 


 (k!)*(k+1)!


 	ext{out}_{i} = \exp(-|x|) * i1(x) = \exp(-|x|) * rac{(	ext{输入}_{i} )}{2} * \sum_{k=0}^{\infty} rac{(	ext{input}_{i}^2/4)^k}{(k!) * (k+1)！}




 out
 


 i
 


 ​
 




 =
 


 exp ( − ∣ x ∣ )



 ∗
 


 我 1 ( x )



 =
 


 exp ( − ∣ x ∣ )



 ∗
 



 2
 




 (
 



 input
 


 i
 


 ​
 


 )
 




 ​
 



 ∗
 


 k = 0


 ∑
 


 ∞
 


 ​
 


 (克！)



 ∗
 



 (
 

 k
 



 +
 



 1
 

 )!
 




 (
 



 input
 


 i
 


 2
 




 ​
 


 /4
 


 )
 



 k
 


 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> torch.special.i1e(torch.arange(5, dtype=torch.float32))
tensor([0.0000, 0.2079, 0.2153, 0.1968, 0.1788])

```


 火炬.特别。



 log1p
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.log1p"此定义的永久链接")


 [`torch.log1p()`](generated/torch.log1p.html#torch.log1p "torch.log1p") 的别名。


 火炬.特别。


 日志_ndtr


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.log_ndtr "此定义的永久链接")


 计算标准高斯概率密度函数下面积的对数，按元素从负无穷大积分到“输入”。


 log_ndtr ( x ) = log ⁡


 (
 


 1
 



 2
 

 π
 


 ∫
 


 −
 

 ∞
 


 x
 



 e
 


 −
 


 1
 

 2
 



 t
 

 2
 


 d t )


 	ext{log\_ndtr}(x) = \log\left(rac{1}{\sqrt{2 \pi}}\int_{-\infty}^{x} e^{-rac {1}{2}t^2} dt \右)


 日志_ndtr


 (  X  )



 =
 




 lo
 
 g
 



 (
 



 2
 

 π
 


 ​
 




 1
 




 ​
 




 ∫
 




 −
 

 ∞
 



 x
 


 ​
 


 e
 




 −
 




 2
 



 1
 


 ​
 


 t
 



 2
 




 d
 

 t
 


 )
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> torch.special.log_ndtr(torch.tensor([-3., -2, -1, 0, 1, 2, 3]))
tensor([-6.6077 -3.7832 -1.841 -0.6931 -0.1728 -0.023 -0.0014])

```


 火炬.特别。


 log_softmax


 (*输入*，*暗淡*，***，*dtype



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.log_softmax"此定义的永久链接")


 计算 softmax，然后计算对数。


 虽然在数学上相当于 log(softmax(x))，但单独执行这两个操作速度较慢且数值不稳定。该函数计算如下：


 log_softmax (


 x
 

 i
 


 ) = 对数 ⁡


 (
 


 经验 ⁡ (


 x
 

 i
 


 )
 




 ∑
 

 j
 


 经验 ⁡ (


 x
 

 j
 


 )
 



 )
 


 	ext{log\_softmax}(x_{i}) = \log\left(rac{\exp(x_i) }{ \sum_j \exp(x_j)} 
ight)


 log_softmax


 (
 


 x
 




 i
 


 ​
 


 )
 



 =
 




 lo
 
 g
 



 (
 


 ∑
 



 j
 




 ​
 


 经验(


 x
 



 j
 




 ​
 


 )
 


 经验(


 x
 



 i
 




 ​
 


 )
 




 ​
 


 )
 


 参数 
* **输入** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入
* **dim** ( [*int*](https://docs.python.org/3/library/functions.html#int "(Python v3.12)") ) – 计算 log_softmax 的维度。
* **dtype** ( [`torch.dtype`]( tensor_attributes.html#torch.dtype "torch.dtype") ，可选) – 返回tensor所需的数据类型。如果指定，则在执行操作之前输入tensor将转换为“dtype”。这对于防止数据类型溢出很有用。默认值：无。


 例子：：




```
>>> t = torch.ones(2, 2)
>>> torch.special.log_softmax(t, 0)
tensor([[-0.6931, -0.6931],
 [-0.6931, -0.6931]])

```


 火炬.特别。



 logit
 


 (*输入*，*eps



 =
 


 无
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.logit"此定义的永久链接")


 返回一个新的tensor，其中包含 `input` 元素的 logit。当 eps 不为 None 时，`input` 被限制为 [eps, 1 
- eps]。当 eps 为 None 且 `input` < 0 或 `input` 
> 1 时，该函数将产生 NaN。




 y
 

 i
 


 = ln ⁡ (



 z
 

 i
 



 1
 

 −
 


 z
 

 i
 




 )
 



 z
 

 i
 



 =
 


 {
 



 x
 

 i
 


 如果 eps 为“无”




 eps
 



 if
 


 x
 

 i
 


 <每股收益



 x
 

 i
 


 如果 eps ≤


 x
 

 i
 


 ≤ 1 − 每股收益


 1 
- 每股收益




 if
 


 x
 

 i
 


 
> 1 − 每股收益


 egin{align}y_{i} &= \ln(rac{z_{i}}{1 
- z_{i}}) \z_{i} &= egin{情况} x_{i} & 	ext{如果 eps 为 None} \ 	ext{eps} & 	ext{如果 } x_{i} < 	ext{eps} \ x_{i} & 	ext{if } 	ext{eps} \leq x_{i} \leq 1 
- 	ext{eps} \ 1 
- 	ext{eps} & 	ext{if } x_{i} 
> 1 
- 	ext{eps}\end{案例}\end{对齐}



 y
 




 i
 


 ​
 



 z
 




 i
 


 ​
 


 ​
 




 =
 



 ln
 

 (
 



 1
 



 −
 




 z
 




 i
 


 ​
 



 z
 




 i
 


 ​
 


 ​
 




 )
 


 =
 




 ⎩
 


 ⎨
 


 ⎧
 




 ​
 




 x
 




 i
 


 ​
 



 eps
 



 x
 




 i
 


 ​
 


 1
 



 −
 




 eps
 


 ​
 


 如果 eps 为“无”



 if
 



 x
 




 i
 


 ​
 




 <
 




 eps
 



 if
 



 eps
 




 ≤
 




 x
 




 i
 


 ​
 




 ≤
 



 1
 



 −
 




 eps
 



 if
 



 x
 




 i
 


 ​
 




 >
 



 1
 



 −
 




 eps
 


 ​
 



 ​
 


 ​
 


 参数 
* **input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。
* **eps** ( [*float*](https://docs.python.org/3/library/functions.html#float "(Python v3.12)")*,
* *可选
* ) – 输入钳位边界的 epsilon。默认值：`无`


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> a = torch.rand(5)
>>> a
tensor([0.2796, 0.9331, 0.6486, 0.1523, 0.6516])
>>> torch.special.logit(a, eps=1e-6)
tensor([-0.9466, 2.6352, 0.6131, -1.7169, 0.6261])

```


 火炬.特别。


 对数和表达式


 (*输入*，*暗淡*，*保持暗淡



 =
 


 假*、***、*输出



 =
 


 无
* ) [¶](#torch.special.logsumexp"此定义的永久链接")


 [`torch.logsumexp()`](generated/torch.logsumexp.html#torch.logsumexp "torch.logsumexp") 的别名。


 火炬.特别。


 多伽玛林


 ( *输入
* , *p
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.multigammaln"此定义的永久链接")


 计算带有维度的[多元对数伽玛函数] (https://en.wikipedia.org/wiki/Multivariate_gamma_function)



 p
 


 p
 


 p
 


 按元素计算，由下式给出


 对数 ⁡ (


 Γ
 

 p
 


 ( a ) ) = C +



 ∑
 


 我 = 1


 p
 


 日志⁡


 (
 

 Γ
 


 (一-


 我-1


 2
 


 )
 


 )
 


 \log(\Gamma_{p}(a)) = C 
+ \displaystyle \sum_{i=1}^{p} \log\left(\Gamma\left(a 
- rac{i 
- 1) }{2}\右)\右)


 lo
 
 g
 


 (
 


 Γ
 




 p
 


 ​
 


 (  A  ))



 =
 




 C
 



 +
 


 我 = 1


 ∑
 


 p
 


 ​
 



 lo
 
 g
 



 (
 


 Γ
 


 (
 


 a
 



 −
 


 2
 




 i
 



 −
 



 1
 




 ​
 


 )
 




 )
 


 where
 


 C = log ⁡ ( π ) ⋅


 p ( p − 1 )


 4
 


 C = \log(\pi) \cdot rac{p (p 
- 1)}{4}


 C
 



 =
 




 lo
 
 g
 


 (Pi)



 ⋅
 




 4
 


 p ( p − 1 )


 ​
 




 and
 


 C(-)


 \伽玛(-)


 C(-)


 是伽玛函数。


 所有元素必须大于


 p−1


 2
 


 rac{p 
- 1}{2}


 2
 


 p−1


 ​
 


 ，否则该行为是不可防御的。


 参数 
* **input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 用于计算多元 log-gamma 函数的tensor
* **p** ( [*int
* ](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 维数


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> a = torch.empty(2, 3).uniform_(1, 2)
>>> a
tensor([[1.6835, 1.8474, 1.1929],
 [1.0475, 1.7162, 1.4180]])
>>> torch.special.multigammaln(a, 2)
tensor([[0.3928, 0.4007, 0.7586],
 [1.0311, 0.3901, 0.5049]])

```


 火炬.特别。



 ndtr
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.ndtr"此定义的永久链接")


 计算标准高斯概率密度函数下的面积，按元素从负无穷大积分到“输入”。


 ndtr ( x ) =


 1
 



 2
 

 π
 


 ∫
 


 −
 

 ∞
 


 x
 



 e
 


 −
 


 1
 

 2
 



 t
 

 2
 




 d
 

 t
 


 	ext{ndtr}(x) = rac{1}{\sqrt{2 \pi}}\int_{-\infty}^{x} e^{-rac{1}{2}t^ 2} dt



 ndtr
 


 (  X  )



 =
 


 2
 

 π
 


 ​
 




 1
 




 ​
 




 ∫
 




 −
 

 ∞
 



 x
 


 ​
 


 e
 




 −
 




 2
 



 1
 


 ​
 


 t
 



 2
 




 d
 

 t
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> torch.special.ndtr(torch.tensor([-3., -2, -1, 0, 1, 2, 3]))
tensor([0.0013, 0.0228, 0.1587, 0.5000, 0.8413, 0.9772, 0.9987])

```


 火炬.特别。



 ndtri
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.ndtri"此定义的永久链接")


 计算参数 x，其高斯概率密度函数(从负无穷大到 x 积分)下的面积等于 ‘input’ ，逐元素。


 ndtri ( p ) =


 2
 



 erf
 


 −
 

 1
 


 ( 2 p − 1 )


 	ext{ndtri}(p) = \sqrt{2}	ext{erf}^{-1}(2p 
- 1)



 ndtri
 


 ( p )



 =
 



 2
 


 ​
 



 erf
 


 −
 

 1
 


 ( 2 p



 −
 




 1
 

 )
 




!!! note "笔记"

    也称为正态分布的分位数函数。


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> torch.special.ndtri(torch.tensor([0, 0.25, 0.5, 0.75, 1]))
tensor([ -inf, -0.6745, 0.0000, 0.6745, inf])

```


 火炬.特别。


 多伽玛


 ( *n
* , *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.polygamma"此定义的永久链接")


 计算




 n
 


 t
 

 h
 


 n^{第}



 n
 




 t
 

 h
 


 `input` 上的 digamma 函数的导数。


 n≥0


 n = 0


 n
 



 ≥
 




 0
 


 称为多伽玛函数的阶。


 ψ
 


 (n)


 ( x ) =



 d
 


 (n)




 d
 


 x
 


 (n)


 ψ ( x )


 \psi^{(n)}(x) = rac{d^{(n)}}{dx^{(n)}} \psi(x)



 ψ
 


 (n)


 (  X  )



 =
 



 d
 


 x
 


 (n)




 d
 


 (n)



 ​
 


 ψ ( x )




!!! note "笔记"

    该函数仅针对非负整数实现


 n≥0


 n = 0


 n
 



 ≥
 




 0
 




.
 


 参数 
* **n** ( [*int*](https://docs.python.org/3/library/functions.html#int "(in Python v3.12)") ) – 多重伽玛的阶数function
* **input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> a = torch.tensor([1, 0.5])
>>> torch.special.polygamma(1, a)
tensor([1.64493, 4.9348])
>>> torch.special.polygamma(2, a)
tensor([ -2.4041, -16.8288])
>>> torch.special.polygamma(3, a)
tensor([ 6.4939, 97.4091])
>>> torch.special.polygamma(4, a)
tensor([ -24.8863, -771.4742])

```


 火炬.特别。



 psi
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.psi"此定义的永久链接")


 [`torch.special.digamma()`](#torch.special.digamma "torch.special.digamma") 的别名。


 火炬.特别。



 round
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.round"此定义的永久链接")


 [`torch.round()`](generated/torch.round.html#torch.round "torch.round") 的别名。


 火炬.特别。


 缩放_修改_贝塞尔_k0


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.scaled_modified_bessel_k0 "此定义的永久链接")


 第二类缩放修正贝塞尔函数



 0
 


 0
 


 0
 




.
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 火炬.特别。


 缩放_修改_贝塞尔_k1


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.scaled_modified_bessel_k1"此定义的永久链接")


 第二类缩放修正贝塞尔函数



 1
 


 1
 


 1
 




.
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 火炬.特别。



 sinc
 


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.sinc "此定义的永久链接")


 计算“输入”的归一化正弦。


 出我


 =
 


 {
 



 1
 

 ,
 




 if
 


 输入我


 =
 

 0
 


 罪 ⁡ ( p


 输入我


 ) /( 圆周率


 输入我


 )
 

 ,
 


 否则


 	ext{out}_{i} =egin{cases} 1, & 	ext{if}\ 	ext{输入}_{i}=0 \ \sin(\pi 	ext{输入}\ _{i}) /(\pi 	ext{输入}_{i}), & 	ext{否则}\end{案例}




 out
 


 i
 


 ​
 




 =
 



 {
 



 1
 

 ,
 


 罪 ( p



 input
 


 i
 


 ​
 


 ) /( 圆周率



 input
 


 i
 


 ​
 


 )
 

 ,
 




 ​
 



 if
 



 input
 


 i
 


 ​
 




 =
 



 0
 


 否则


 ​
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> t = torch.randn(4)
>>> t
tensor([ 0.2252, -0.2948, 1.0267, -1.1566])
>>> torch.special.sinc(t)
tensor([ 0.9186, 0.8631, -0.0259, -0.1300])

```


 火炬.特别。


 软最大


 (*输入*，*暗淡*，***，*dtype



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.softmax"此定义的永久链接")


 计算 softmax 函数。


 Softmax 定义为：


 Softmax (


 x
 

 i
 


 )
 

 =
 


 经验 ⁡ (


 x
 

 i
 


 )
 




 ∑
 

 j
 


 经验 ⁡ (


 x
 

 j
 


 )
 


 	ext{Softmax}(x_{i}) = rac{\exp(x_i)}{\sum_j \exp(x_j)}


 软最大


 (
 


 x
 




 i
 


 ​
 


 )
 



 =
 


 ∑
 



 j
 




 ​
 


 经验值


 (
 


 x
 



 j
 




 ​
 


 )
 


 经验值


 (
 


 x
 



 i
 




 ​
 


 )
 


 ​
 


 它应用于沿暗淡方向的所有切片，并将重新缩放它们，以便元素位于 [0, 1] 范围内且总和为 1。


 参数 
* **输入** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入
* **dim** ( [*int*](https://docs.python.org/3/library/functions.html#int "(Python v3.12)") ) – 计算 softmax 的维度。
* **dtype** ( [`torch.dtype`](tensor_attributes. html#torch.dtype "torch.dtype") ，可选) – 返回tensor所需的数据类型。如果指定，则在执行操作之前输入tensor将转换为“dtype”。这对于防止数据类型溢出很有用。默认值：无。


 例子：：




```
>>> t = torch.ones(2, 2)
>>> torch.special.softmax(t, 0)
tensor([[0.5000, 0.5000],
 [0.5000, 0.5000]])

```


 火炬.特别。


 球面_贝塞尔_j0


 ( *输入
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.spherical_bessel_j0"此定义的永久链接")


 第一类球贝塞尔函数



 0
 


 0
 


 0
 




.
 


 Parameters


**input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 输入tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 火炬.特别。


 xlog1py


 ( *输入
* , *其他
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.xlog1py"此定义的永久链接")


 使用以下情况计算“输入 * log1p(other)”。


 出我


 =
 


 {
 


 NaN
 



 if
 


 其他我


 = 南


 0
 



 if
 


 输入我


 = 0.0 和


 其他我


 ！ = 南


 输入我


 
* log1p (


 其他我


 )
 


 否则


 	ext{out}_{i} = egin{cases} 	ext{NaN} & 	ext{if } 	ext{other}_{i} = 	ext{NaN} \ 0 & 	ext{ if } 	ext{input}_{i} = 0.0 	ext{ 和 } 	ext{other}_{i} != 	ext{NaN} \ 	ext{input}_{i} * 	ext{log1p}(	ext{其他}_{i})& 	ext{其他}\end{案例}




 out
 


 i
 


 ​
 




 =
 


 ⎩
 


 ⎨
 


 ⎧
 




 ​
 




 NaN
 


 0
 



 input
 


 i
 


 ​
 




 ∗
 




 log1p
 


 (
 



 other
 


 i
 


 ​
 


 )
 




 ​
 



 if
 




 other
 


 i
 


 ​
 




 =
 




 NaN
 



 if
 




 input
 


 i
 


 ​
 




 =
 



 0.0
 


 and
 




 other
 


 i
 


 ​
 


 !
 



 =
 




 NaN
 


 否则


 ​
 


 类似于 SciPy 的 scipy.special.xlog1py 。


 参数 
* **输入** ( *Number
* *或
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 乘数
* **其他** ( *Number
* *或
* [ *Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – Parameters



!!! note "笔记"

    至少“input”或“other”之一必须是tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> x = torch.zeros(5,)
>>> y = torch.tensor([-1, 0, 1, float('inf'), float('nan')])
>>> torch.special.xlog1py(x, y)
tensor([0., 0., 0., 0., nan])
>>> x = torch.tensor([1, 2, 3])
>>> y = torch.tensor([3, 2, 1])
>>> torch.special.xlog1py(x, y)
tensor([1.3863, 2.1972, 2.0794])
>>> torch.special.xlog1py(x, 4)
tensor([1.6094, 3.2189, 4.8283])
>>> torch.special.xlog1py(2, y)
tensor([2.7726, 2.1972, 1.3863])

```


 火炬.特别。



 xlogy
 


 ( *输入
* , *其他
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.xlogy"此定义的永久链接")


 在以下情况下计算“输入 * log(other)”。


 出我


 =
 


 {
 


 NaN
 



 if
 


 其他我


 = 南


 0
 



 if
 


 输入我


 = 0.0


 输入我


 
* 日志⁡


 (
 


 其他我


 )
 


 否则


 	ext{out}_{i} = egin{cases} 	ext{NaN} & 	ext{if } 	ext{other}_{i} = 	ext{NaN} \ 0 & 	ext{ if } 	ext{输入}_{i} = 0.0 \ 	ext{输入}_{i} * \log{(	ext{other}_{i})} & 	ext{其他} \结束{案例}




 out
 


 i
 


 ​
 




 =
 


 ⎩
 


 ⎨
 


 ⎧
 




 ​
 




 NaN
 


 0
 



 input
 


 i
 


 ​
 




 ∗
 



 lo
 
 g
 


 (
 



 other
 


 i
 


 ​
 


 )
 


 ​
 



 if
 




 other
 


 i
 


 ​
 




 =
 




 NaN
 



 if
 




 input
 


 i
 


 ​
 




 =
 



 0.0
 


 否则


 ​
 


 类似于 SciPy 的 scipy.special.xlogy 。


 参数 
* **输入** ( *Number
* *或
* [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 乘数
* **其他** ( *Number
* *或
* [ *Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – Parameters



!!! note "笔记"

    至少“input”或“other”之一必须是tensor。


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：


```
>>> x = torch.zeros(5,)
>>> y = torch.tensor([-1, 0, 1, float('inf'), float('nan')])
>>> torch.special.xlogy(x, y)
tensor([0., 0., 0., 0., nan])
>>> x = torch.tensor([1, 2, 3])
>>> y = torch.tensor([3, 2, 1])
>>> torch.special.xlogy(x, y)
tensor([1.0986, 1.3863, 0.0000])
>>> torch.special.xlogy(x, 4)
tensor([1.3863, 2.7726, 4.1589])
>>> torch.special.xlogy(2, y)
tensor([2.1972, 1.3863, 0.0000])

```


 火炬.特别。



 zeta
 


 ( *输入
* , *其他
* , *** , *输出



 =
 


 没有任何
* )


 → [tensor](tensors.html#torch.Tensor "torch.Tensor")


[¶](#torch.special.zeta"此定义的永久链接")


 按元素计算 Hurwitz zeta 函数。


 δ ( x , q ) =


 ∑
 


 k = 0


 ∞
 



 1
 


 ( k 
+ q


 )
 

 x
 


 \zeta(x, q) = \sum_{k=0}^{\infty} rac{1}{(k 
+ q)^x}


 z ( x ,



 q
 

 )
 



 =
 


 k = 0


 ∑
 


 ∞
 


 ​
 


 (
 

 k
 



 +
 



 q
 


 )
 



 x
 


 1
 




 ​
 


 参数 
* **input** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – x 对应的输入tensor.
* **other** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor") ) – 对应于 q 的输入tensor。



!!! note "笔记"

    黎曼 zeta 函数对应于 q = 1 时的情况


 关键字参数


**out** ( [*Tensor*](tensors.html#torch.Tensor "torch.Tensor")*,
* *可选
* ) – 输出tensor。


 例子：：




```
>>> x = torch.tensor([2., 4.])
>>> torch.special.zeta(x, 1)
tensor([1.6449, 1.0823])
>>> torch.special.zeta(x, torch.tensor([1., 2.]))
tensor([1.6449, 0.0823])
>>> torch.special.zeta(2, torch.tensor([1., 2.]))
tensor([1.6449, 0.6449])

```