# 命名tensor运算符覆盖率 [¶](#named-tensors-operator-coverage "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/name_inference>
>
> 原始地址：<https://pytorch.org/docs/stable/name_inference.html>


 请首先阅读[命名tensor](named_tensor.html#named-tensors-doc)，了解命名tensor的介绍。


 本文档是 *name inference
* 的参考，这是一个定义tensor命名的过程：


1. 使用名称提供额外的自动运行时正确性检查2。将名称从输入tensor传播到输出tensor


 下面是命名tensor支持的所有操作及其关联的名称推理规则的列表。


 如果您没有看到此处列出的操作，但它会对您的用例有所帮助，请[搜索是否已提交问题](https://github.com/pytorch/pytorch/issues?q=is%3Aopen +is%3Aissue+label%3A%22module%3A+named+tensor%22) 如果不是，则[文件一](https://github.com/pytorch/pytorch/issues/new/choose) 。


!!! warning "警告"

     指定的tensor API 是实验性的，可能会发生变化。


 支持的操作 [¶](#id1"此表的永久链接")


| 	 API	  | 	 Name inference rule	  |
| --- | --- |
| 	[`Tensor.abs()`](generated/torch.Tensor.abs.html#torch.Tensor.abs "torch.Tensor.abs")	 ,	 [`torch.abs()`](generated/torch.abs.html#torch.abs "torch.abs")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.abs_()`](generated/torch.Tensor.abs_.html#torch.Tensor.abs_ "torch.Tensor.abs_")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.acos()`](generated/torch.Tensor.acos.html#torch.Tensor.acos "torch.Tensor.acos")	 ,	 [`torch.acos()`](generated/torch.acos.html#torch.acos "torch.acos")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.acos_()`](generated/torch.Tensor.acos_.html#torch.Tensor.acos_ "torch.Tensor.acos_")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.add()`](generated/torch.Tensor.add.html#torch.Tensor.add "torch.Tensor.add")	 ,	 [`torch.add()`](generated/torch.add.html#torch.add "torch.add")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.add_()`](generated/torch.Tensor.add_.html#torch.Tensor.add_ "torch.Tensor.add_")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.addmm()`](generated/torch.Tensor.addmm.html#torch.Tensor.addmm "torch.Tensor.addmm")	 ,	 [`torch.addmm()`](generated/torch.addmm.html#torch.addmm "torch.addmm")	 | 	[Contracts away dims](#contracts-away-dims-doc)	 |
| 	[`Tensor.addmm_()`](generated/torch.Tensor.addmm_.html#torch.Tensor.addmm_ "torch.Tensor.addmm_")	 | 	[Contracts away dims](#contracts-away-dims-doc)	 |
| 	[`Tensor.addmv()`](generated/torch.Tensor.addmv.html#torch.Tensor.addmv "torch.Tensor.addmv")	 ,	 [`torch.addmv()`](generated/torch.addmv.html#torch.addmv "torch.addmv")	 | 	[Contracts away dims](#contracts-away-dims-doc)	 |
| 	[`Tensor.addmv_()`](generated/torch.Tensor.addmv_.html#torch.Tensor.addmv_ "torch.Tensor.addmv_")	 | 	[Contracts away dims](#contracts-away-dims-doc)	 |
| 	[`Tensor.align_as()`](named_tensor.html#torch.Tensor.align_as "torch.Tensor.align_as")	 | 	 See documentation	  |
| 	[`Tensor.align_to()`](named_tensor.html#torch.Tensor.align_to "torch.Tensor.align_to")	 | 	 See documentation	  |
| 	[`Tensor.all()`](generated/torch.Tensor.all.html#torch.Tensor.all "torch.Tensor.all")	 ,	 [`torch.all()`](generated/torch.all.html#torch.all "torch.all")	 | 	 None	  |
| 	[`Tensor.any()`](generated/torch.Tensor.any.html#torch.Tensor.any "torch.Tensor.any")	 ,	 [`torch.any()`](generated/torch.any.html#torch.any "torch.any")	 | 	 None	  |
| 	[`Tensor.asin()`](generated/torch.Tensor.asin.html#torch.Tensor.asin "torch.Tensor.asin")	 ,	 [`torch.asin()`](generated/torch.asin.html#torch.asin "torch.asin")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.asin_()`](generated/torch.Tensor.asin_.html#torch.Tensor.asin_ "torch.Tensor.asin_")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.atan()`](generated/torch.Tensor.atan.html#torch.Tensor.atan "torch.Tensor.atan")	 ,	 [`torch.atan()`](generated/torch.atan.html#torch.atan "torch.atan")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.atan2()`](generated/torch.Tensor.atan2.html#torch.Tensor.atan2 "torch.Tensor.atan2")	 ,	 [`torch.atan2()`](generated/torch.atan2.html#torch.atan2 "torch.atan2")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.atan2_()`](generated/torch.Tensor.atan2_.html#torch.Tensor.atan2_ "torch.Tensor.atan2_")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.atan_()`](generated/torch.Tensor.atan_.html#torch.Tensor.atan_ "torch.Tensor.atan_")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.bernoulli()`](generated/torch.Tensor.bernoulli.html#torch.Tensor.bernoulli "torch.Tensor.bernoulli")	 ,	 [`torch.bernoulli()`](generated/torch.bernoulli.html#torch.bernoulli "torch.bernoulli")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.bernoulli_()`](generated/torch.Tensor.bernoulli_.html#torch.Tensor.bernoulli_ "torch.Tensor.bernoulli_")	 | 	 None	  |
| 	[`Tensor.bfloat16()`](generated/torch.Tensor.bfloat16.html#torch.Tensor.bfloat16 "torch.Tensor.bfloat16")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.bitwise_not()`](generated/torch.Tensor.bitwise_not.html#torch.Tensor.bitwise_not "torch.Tensor.bitwise_not")	 ,	 [`torch.bitwise_not()`](generated/torch.bitwise_not.html#torch.bitwise_not "torch.bitwise_not")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.bitwise_not_()`](generated/torch.Tensor.bitwise_not_.html#torch.Tensor.bitwise_not_ "torch.Tensor.bitwise_not_")	 | 	 None	  |
| 	[`Tensor.bmm()`](generated/torch.Tensor.bmm.html#torch.Tensor.bmm "torch.Tensor.bmm")	 ,	 [`torch.bmm()`](generated/torch.bmm.html#torch.bmm "torch.bmm")	 | 	[Contracts away dims](#contracts-away-dims-doc)	 |
| 	[`Tensor.bool()`](generated/torch.Tensor.bool.html#torch.Tensor.bool "torch.Tensor.bool")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.byte()`](generated/torch.Tensor.byte.html#torch.Tensor.byte "torch.Tensor.byte")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`torch.cat()`](generated/torch.cat.html#torch.cat "torch.cat")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.cauchy_()`](generated/torch.Tensor.cauchy_.html#torch.Tensor.cauchy_ "torch.Tensor.cauchy_")	 | 	 None	  |
| 	[`Tensor.ceil()`](generated/torch.Tensor.ceil.html#torch.Tensor.ceil "torch.Tensor.ceil")	 ,	 [`torch.ceil()`](generated/torch.ceil.html#torch.ceil "torch.ceil")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.ceil_()`](generated/torch.Tensor.ceil_.html#torch.Tensor.ceil_ "torch.Tensor.ceil_")	 | 	 None	  |
| 	[`Tensor.char()`](generated/torch.Tensor.char.html#torch.Tensor.char "torch.Tensor.char")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.chunk()`](generated/torch.Tensor.chunk.html#torch.Tensor.chunk "torch.Tensor.chunk")	 ,	 [`torch.chunk()`](generated/torch.chunk.html#torch.chunk "torch.chunk")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.clamp()`](generated/torch.Tensor.clamp.html#torch.Tensor.clamp "torch.Tensor.clamp")	 ,	 [`torch.clamp()`](generated/torch.clamp.html#torch.clamp "torch.clamp")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.clamp_()`](generated/torch.Tensor.clamp_.html#torch.Tensor.clamp_ "torch.Tensor.clamp_")	 | 	 None	  |
| 	[`Tensor.copy_()`](generated/torch.Tensor.copy_.html#torch.Tensor.copy_ "torch.Tensor.copy_")	 | 	[out function and in-place variants](#out-function-semantics-doc)	 |
| 	[`Tensor.cos()`](generated/torch.Tensor.cos.html#torch.Tensor.cos "torch.Tensor.cos")	 ,	 [`torch.cos()`](generated/torch.cos.html#torch.cos "torch.cos")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.cos_()`](generated/torch.Tensor.cos_.html#torch.Tensor.cos_ "torch.Tensor.cos_")	 | 	 None	  |
| 	[`Tensor.cosh()`](generated/torch.Tensor.cosh.html#torch.Tensor.cosh "torch.Tensor.cosh")	 ,	 [`torch.cosh()`](generated/torch.cosh.html#torch.cosh "torch.cosh")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.cosh_()`](generated/torch.Tensor.cosh_.html#torch.Tensor.cosh_ "torch.Tensor.cosh_")	 | 	 None	  |
| 	[`Tensor.acosh()`](generated/torch.Tensor.acosh.html#torch.Tensor.acosh "torch.Tensor.acosh")	 ,	 [`torch.acosh()`](generated/torch.acosh.html#torch.acosh "torch.acosh")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.acosh_()`](generated/torch.Tensor.acosh_.html#torch.Tensor.acosh_ "torch.Tensor.acosh_")	 | 	 None	  |
| 	[`Tensor.cpu()`](generated/torch.Tensor.cpu.html#torch.Tensor.cpu "torch.Tensor.cpu")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.cuda()`](generated/torch.Tensor.cuda.html#torch.Tensor.cuda "torch.Tensor.cuda")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.cumprod()`](generated/torch.Tensor.cumprod.html#torch.Tensor.cumprod "torch.Tensor.cumprod")	 ,	 [`torch.cumprod()`](generated/torch.cumprod.html#torch.cumprod "torch.cumprod")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.cumsum()`](generated/torch.Tensor.cumsum.html#torch.Tensor.cumsum "torch.Tensor.cumsum")	 ,	 [`torch.cumsum()`](generated/torch.cumsum.html#torch.cumsum "torch.cumsum")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.data_ptr()`](generated/torch.Tensor.data_ptr.html#torch.Tensor.data_ptr "torch.Tensor.data_ptr")	 | 	 None	  |
| 	[`Tensor.deg2rad()`](generated/torch.Tensor.deg2rad.html#torch.Tensor.deg2rad "torch.Tensor.deg2rad")	 ,	 [`torch.deg2rad()`](generated/torch.deg2rad.html#torch.deg2rad "torch.deg2rad")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	`Tensor.deg2rad_()`	 | 	 None	  |
| 	[`Tensor.detach()`](generated/torch.Tensor.detach.html#torch.Tensor.detach "torch.Tensor.detach")	 ,	 `torch.detach()`	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.detach_()`](generated/torch.Tensor.detach_.html#torch.Tensor.detach_ "torch.Tensor.detach_")	 | 	 None	  |
| 	[`Tensor.device`](generated/torch.Tensor.device.html#torch.Tensor.device "torch.Tensor.device")	 ,	 [`torch.device()`](tensor_attributes.html#torch.device "torch.device")	 | 	 None	  |
| 	[`Tensor.digamma()`](generated/torch.Tensor.digamma.html#torch.Tensor.digamma "torch.Tensor.digamma")	 ,	 [`torch.digamma()`](generated/torch.digamma.html#torch.digamma "torch.digamma")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.digamma_()`](generated/torch.Tensor.digamma_.html#torch.Tensor.digamma_ "torch.Tensor.digamma_")	 | 	 None	  |
| 	[`Tensor.dim()`](generated/torch.Tensor.dim.html#torch.Tensor.dim "torch.Tensor.dim")	 | 	 None	  |
| 	[`Tensor.div()`](generated/torch.Tensor.div.html#torch.Tensor.div "torch.Tensor.div")	 ,	 [`torch.div()`](generated/torch.div.html#torch.div "torch.div")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.div_()`](generated/torch.Tensor.div_.html#torch.Tensor.div_ "torch.Tensor.div_")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.dot()`](generated/torch.Tensor.dot.html#torch.Tensor.dot "torch.Tensor.dot")	 ,	 [`torch.dot()`](generated/torch.dot.html#torch.dot "torch.dot")	 | 	 None	  |
| 	[`Tensor.double()`](generated/torch.Tensor.double.html#torch.Tensor.double "torch.Tensor.double")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.element_size()`](generated/torch.Tensor.element_size.html#torch.Tensor.element_size "torch.Tensor.element_size")	 | 	 None	  |
| 	[`torch.empty()`](generated/torch.empty.html#torch.empty "torch.empty")	 | 	[Factory functions](#factory-doc)	 |
| 	[`torch.empty_like()`](generated/torch.empty_like.html#torch.empty_like "torch.empty_like")	 | 	[Factory functions](#factory-doc)	 |
| 	[`Tensor.eq()`](generated/torch.Tensor.eq.html#torch.Tensor.eq "torch.Tensor.eq")	 ,	 [`torch.eq()`](generated/torch.eq.html#torch.eq "torch.eq")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.erf()`](generated/torch.Tensor.erf.html#torch.Tensor.erf "torch.Tensor.erf")	 ,	 [`torch.erf()`](generated/torch.erf.html#torch.erf "torch.erf")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.erf_()`](generated/torch.Tensor.erf_.html#torch.Tensor.erf_ "torch.Tensor.erf_")	 | 	 None	  |
| 	[`Tensor.erfc()`](generated/torch.Tensor.erfc.html#torch.Tensor.erfc "torch.Tensor.erfc")	 ,	 [`torch.erfc()`](generated/torch.erfc.html#torch.erfc "torch.erfc")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.erfc_()`](generated/torch.Tensor.erfc_.html#torch.Tensor.erfc_ "torch.Tensor.erfc_")	 | 	 None	  |
| 	[`Tensor.erfinv()`](generated/torch.Tensor.erfinv.html#torch.Tensor.erfinv "torch.Tensor.erfinv")	 ,	 [`torch.erfinv()`](generated/torch.erfinv.html#torch.erfinv "torch.erfinv")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.erfinv_()`](generated/torch.Tensor.erfinv_.html#torch.Tensor.erfinv_ "torch.Tensor.erfinv_")	 | 	 None	  |
| 	[`Tensor.exp()`](generated/torch.Tensor.exp.html#torch.Tensor.exp "torch.Tensor.exp")	 ,	 [`torch.exp()`](generated/torch.exp.html#torch.exp "torch.exp")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.exp_()`](generated/torch.Tensor.exp_.html#torch.Tensor.exp_ "torch.Tensor.exp_")	 | 	 None	  |
| 	[`Tensor.expand()`](generated/torch.Tensor.expand.html#torch.Tensor.expand "torch.Tensor.expand")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.expm1()`](generated/torch.Tensor.expm1.html#torch.Tensor.expm1 "torch.Tensor.expm1")	 ,	 [`torch.expm1()`](generated/torch.expm1.html#torch.expm1 "torch.expm1")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.expm1_()`](generated/torch.Tensor.expm1_.html#torch.Tensor.expm1_ "torch.Tensor.expm1_")	 | 	 None	  |
| 	[`Tensor.exponential_()`](generated/torch.Tensor.exponential_.html#torch.Tensor.exponential_ "torch.Tensor.exponential_")	 | 	 None	  |
| 	[`Tensor.fill_()`](generated/torch.Tensor.fill_.html#torch.Tensor.fill_ "torch.Tensor.fill_")	 | 	 None	  |
| 	[`Tensor.flatten()`](generated/torch.Tensor.flatten.html#torch.Tensor.flatten "torch.Tensor.flatten")	 ,	 [`torch.flatten()`](generated/torch.flatten.html#torch.flatten "torch.flatten")	 | 	 See documentation	  |
| 	[`Tensor.float()`](generated/torch.Tensor.float.html#torch.Tensor.float "torch.Tensor.float")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.floor()`](generated/torch.Tensor.floor.html#torch.Tensor.floor "torch.Tensor.floor")	 ,	 [`torch.floor()`](generated/torch.floor.html#torch.floor "torch.floor")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.floor_()`](generated/torch.Tensor.floor_.html#torch.Tensor.floor_ "torch.Tensor.floor_")	 | 	 None	  |
| 	[`Tensor.frac()`](generated/torch.Tensor.frac.html#torch.Tensor.frac "torch.Tensor.frac")	 ,	 [`torch.frac()`](generated/torch.frac.html#torch.frac "torch.frac")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.frac_()`](generated/torch.Tensor.frac_.html#torch.Tensor.frac_ "torch.Tensor.frac_")	 | 	 None	  |
| 	[`Tensor.ge()`](generated/torch.Tensor.ge.html#torch.Tensor.ge "torch.Tensor.ge")	 ,	 [`torch.ge()`](generated/torch.ge.html#torch.ge "torch.ge")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.get_device()`](generated/torch.Tensor.get_device.html#torch.Tensor.get_device "torch.Tensor.get_device")	 ,	 `torch.get_device()`	 | 	 None	  |
| 	[`Tensor.grad`](generated/torch.Tensor.grad.html#torch.Tensor.grad "torch.Tensor.grad")	 | 	 None	  |
| 	[`Tensor.gt()`](generated/torch.Tensor.gt.html#torch.Tensor.gt "torch.Tensor.gt")	 ,	 [`torch.gt()`](generated/torch.gt.html#torch.gt "torch.gt")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.half()`](generated/torch.Tensor.half.html#torch.Tensor.half "torch.Tensor.half")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	`Tensor.has_names()`	 | 	 See documentation	  |
| 	[`Tensor.index_fill()`](generated/torch.Tensor.index_fill.html#torch.Tensor.index_fill "torch.Tensor.index_fill")	 ,	 `torch.index_fill()`	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.index_fill_()`](generated/torch.Tensor.index_fill_.html#torch.Tensor.index_fill_ "torch.Tensor.index_fill_")	 | 	 None	  |
| 	[`Tensor.int()`](generated/torch.Tensor.int.html#torch.Tensor.int "torch.Tensor.int")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.is_contiguous()`](generated/torch.Tensor.is_contiguous.html#torch.Tensor.is_contiguous "torch.Tensor.is_contiguous")	 | 	 None	  |
| 	[`Tensor.is_cuda`](generated/torch.Tensor.is_cuda.html#torch.Tensor.is_cuda "torch.Tensor.is_cuda")	 | 	 None	  |
| 	[`Tensor.is_floating_point()`](generated/torch.Tensor.is_floating_point.html#torch.Tensor.is_floating_point "torch.Tensor.is_floating_point")	 ,	 [`torch.is_floating_point()`](generated/torch.is_floating_point.html#torch.is_floating_point "torch.is_floating_point")	 | 	 None	  |
| 	[`Tensor.is_leaf`](generated/torch.Tensor.is_leaf.html#torch.Tensor.is_leaf "torch.Tensor.is_leaf")	 | 	 None	  |
| 	[`Tensor.is_pinned()`](generated/torch.Tensor.is_pinned.html#torch.Tensor.is_pinned "torch.Tensor.is_pinned")	 | 	 None	  |
| 	[`Tensor.is_shared()`](generated/torch.Tensor.is_shared.html#torch.Tensor.is_shared "torch.Tensor.is_shared")	 | 	 None	  |
| 	[`Tensor.is_signed()`](generated/torch.Tensor.is_signed.html#torch.Tensor.is_signed "torch.Tensor.is_signed")	 ,	 `torch.is_signed()`	 | 	 None	  |
| 	[`Tensor.is_sparse`](generated/torch.Tensor.is_sparse.html#torch.Tensor.is_sparse "torch.Tensor.is_sparse")	 | 	 None	  |
| 	[`Tensor.is_sparse_csr`](generated/torch.Tensor.is_sparse_csr.html#torch.Tensor.is_sparse_csr "torch.Tensor.is_sparse_csr")	 | 	 None	  |
| 	[`torch.is_tensor()`](generated/torch.is_tensor.html#torch.is_tensor "torch.is_tensor")	 | 	 None	  |
| 	[`Tensor.item()`](generated/torch.Tensor.item.html#torch.Tensor.item "torch.Tensor.item")	 | 	 None	  |
| 	[`Tensor.itemsize`](generated/torch.Tensor.itemsize.html#torch.Tensor.itemsize "torch.Tensor.itemsize")	 | 	 None	  |
| 	[`Tensor.kthvalue()`](generated/torch.Tensor.kthvalue.html#torch.Tensor.kthvalue "torch.Tensor.kthvalue")	 ,	 [`torch.kthvalue()`](generated/torch.kthvalue.html#torch.kthvalue "torch.kthvalue")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.le()`](generated/torch.Tensor.le.html#torch.Tensor.le "torch.Tensor.le")	 ,	 [`torch.le()`](generated/torch.le.html#torch.le "torch.le")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.log()`](generated/torch.Tensor.log.html#torch.Tensor.log "torch.Tensor.log")	 ,	 [`torch.log()`](generated/torch.log.html#torch.log "torch.log")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.log10()`](generated/torch.Tensor.log10.html#torch.Tensor.log10 "torch.Tensor.log10")	 ,	 [`torch.log10()`](generated/torch.log10.html#torch.log10 "torch.log10")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.log10_()`](generated/torch.Tensor.log10_.html#torch.Tensor.log10_ "torch.Tensor.log10_")	 | 	 None	  |
| 	[`Tensor.log1p()`](generated/torch.Tensor.log1p.html#torch.Tensor.log1p "torch.Tensor.log1p")	 ,	 [`torch.log1p()`](generated/torch.log1p.html#torch.log1p "torch.log1p")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.log1p_()`](generated/torch.Tensor.log1p_.html#torch.Tensor.log1p_ "torch.Tensor.log1p_")	 | 	 None	  |
| 	[`Tensor.log2()`](generated/torch.Tensor.log2.html#torch.Tensor.log2 "torch.Tensor.log2")	 ,	 [`torch.log2()`](generated/torch.log2.html#torch.log2 "torch.log2")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.log2_()`](generated/torch.Tensor.log2_.html#torch.Tensor.log2_ "torch.Tensor.log2_")	 | 	 None	  |
| 	[`Tensor.log_()`](generated/torch.Tensor.log_.html#torch.Tensor.log_ "torch.Tensor.log_")	 | 	 None	  |
| 	[`Tensor.log_normal_()`](generated/torch.Tensor.log_normal_.html#torch.Tensor.log_normal_ "torch.Tensor.log_normal_")	 | 	 None	  |
| 	[`Tensor.logical_not()`](generated/torch.Tensor.logical_not.html#torch.Tensor.logical_not "torch.Tensor.logical_not")	 ,	 [`torch.logical_not()`](generated/torch.logical_not.html#torch.logical_not "torch.logical_not")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.logical_not_()`](generated/torch.Tensor.logical_not_.html#torch.Tensor.logical_not_ "torch.Tensor.logical_not_")	 | 	 None	  |
| 	[`Tensor.logsumexp()`](generated/torch.Tensor.logsumexp.html#torch.Tensor.logsumexp "torch.Tensor.logsumexp")	 ,	 [`torch.logsumexp()`](generated/torch.logsumexp.html#torch.logsumexp "torch.logsumexp")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.long()`](generated/torch.Tensor.long.html#torch.Tensor.long "torch.Tensor.long")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.lt()`](generated/torch.Tensor.lt.html#torch.Tensor.lt "torch.Tensor.lt")	 ,	 [`torch.lt()`](generated/torch.lt.html#torch.lt "torch.lt")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`torch.manual_seed()`](generated/torch.manual_seed.html#torch.manual_seed "torch.manual_seed")	 | 	 None	  |
| 	[`Tensor.masked_fill()`](generated/torch.Tensor.masked_fill.html#torch.Tensor.masked_fill "torch.Tensor.masked_fill")	 ,	 `torch.masked_fill()`	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.masked_fill_()`](generated/torch.Tensor.masked_fill_.html#torch.Tensor.masked_fill_ "torch.Tensor.masked_fill_")	 | 	 None	  |
| 	[`Tensor.masked_select()`](generated/torch.Tensor.masked_select.html#torch.Tensor.masked_select "torch.Tensor.masked_select")	 ,	 [`torch.masked_select()`](generated/torch.masked_select.html#torch.masked_select "torch.masked_select")	 | 	 Aligns mask up to input and then unifies_names_from_input_tensors	  |
| 	[`Tensor.matmul()`](generated/torch.Tensor.matmul.html#torch.Tensor.matmul "torch.Tensor.matmul")	 ,	 [`torch.matmul()`](generated/torch.matmul.html#torch.matmul "torch.matmul")	 | 	[Contracts away dims](#contracts-away-dims-doc)	 |
| 	[`Tensor.mean()`](generated/torch.Tensor.mean.html#torch.Tensor.mean "torch.Tensor.mean")	 ,	 [`torch.mean()`](generated/torch.mean.html#torch.mean "torch.mean")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.median()`](generated/torch.Tensor.median.html#torch.Tensor.median "torch.Tensor.median")	 ,	 [`torch.median()`](generated/torch.median.html#torch.median "torch.median")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.nanmedian()`](generated/torch.Tensor.nanmedian.html#torch.Tensor.nanmedian "torch.Tensor.nanmedian")	 ,	 [`torch.nanmedian()`](generated/torch.nanmedian.html#torch.nanmedian "torch.nanmedian")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.mm()`](generated/torch.Tensor.mm.html#torch.Tensor.mm "torch.Tensor.mm")	 ,	 [`torch.mm()`](generated/torch.mm.html#torch.mm "torch.mm")	 | 	[Contracts away dims](#contracts-away-dims-doc)	 |
| 	[`Tensor.mode()`](generated/torch.Tensor.mode.html#torch.Tensor.mode "torch.Tensor.mode")	 ,	 [`torch.mode()`](generated/torch.mode.html#torch.mode "torch.mode")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.mul()`](generated/torch.Tensor.mul.html#torch.Tensor.mul "torch.Tensor.mul")	 ,	 [`torch.mul()`](generated/torch.mul.html#torch.mul "torch.mul")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.mul_()`](generated/torch.Tensor.mul_.html#torch.Tensor.mul_ "torch.Tensor.mul_")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.mv()`](generated/torch.Tensor.mv.html#torch.Tensor.mv "torch.Tensor.mv")	 ,	 [`torch.mv()`](generated/torch.mv.html#torch.mv "torch.mv")	 | 	[Contracts away dims](#contracts-away-dims-doc)	 |
| 	[`Tensor.names`](named_tensor.html#torch.Tensor.names "torch.Tensor.names")	 | 	 See documentation	  |
| 	[`Tensor.narrow()`](generated/torch.Tensor.narrow.html#torch.Tensor.narrow "torch.Tensor.narrow")	 ,	 [`torch.narrow()`](generated/torch.narrow.html#torch.narrow "torch.narrow")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.nbytes`](generated/torch.Tensor.nbytes.html#torch.Tensor.nbytes "torch.Tensor.nbytes")	 | 	 None	  |
| 	[`Tensor.ndim`](generated/torch.Tensor.ndim.html#torch.Tensor.ndim "torch.Tensor.ndim")	 | 	 None	  |
| 	[`Tensor.ndimension()`](generated/torch.Tensor.ndimension.html#torch.Tensor.ndimension "torch.Tensor.ndimension")	 | 	 None	  |
| 	[`Tensor.ne()`](generated/torch.Tensor.ne.html#torch.Tensor.ne "torch.Tensor.ne")	 ,	 [`torch.ne()`](generated/torch.ne.html#torch.ne "torch.ne")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.neg()`](generated/torch.Tensor.neg.html#torch.Tensor.neg "torch.Tensor.neg")	 ,	 [`torch.neg()`](generated/torch.neg.html#torch.neg "torch.neg")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.neg_()`](generated/torch.Tensor.neg_.html#torch.Tensor.neg_ "torch.Tensor.neg_")	 | 	 None	  |
| 	[`torch.normal()`](generated/torch.normal.html#torch.normal "torch.normal")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.normal_()`](generated/torch.Tensor.normal_.html#torch.Tensor.normal_ "torch.Tensor.normal_")	 | 	 None	  |
| 	[`Tensor.numel()`](generated/torch.Tensor.numel.html#torch.Tensor.numel "torch.Tensor.numel")	 ,	 [`torch.numel()`](generated/torch.numel.html#torch.numel "torch.numel")	 | 	 None	  |
| 	[`torch.ones()`](generated/torch.ones.html#torch.ones "torch.ones")	 | 	[Factory functions](#factory-doc)	 |
| 	[`Tensor.pow()`](generated/torch.Tensor.pow.html#torch.Tensor.pow "torch.Tensor.pow")	 ,	 [`torch.pow()`](generated/torch.pow.html#torch.pow "torch.pow")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.pow_()`](generated/torch.Tensor.pow_.html#torch.Tensor.pow_ "torch.Tensor.pow_")	 | 	 None	  |
| 	[`Tensor.prod()`](generated/torch.Tensor.prod.html#torch.Tensor.prod "torch.Tensor.prod")	 ,	 [`torch.prod()`](generated/torch.prod.html#torch.prod "torch.prod")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.rad2deg()`](generated/torch.Tensor.rad2deg.html#torch.Tensor.rad2deg "torch.Tensor.rad2deg")	 ,	 [`torch.rad2deg()`](generated/torch.rad2deg.html#torch.rad2deg "torch.rad2deg")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	`Tensor.rad2deg_()`	 | 	 None	  |
| 	[`torch.rand()`](generated/torch.rand.html#torch.rand "torch.rand")	 | 	[Factory functions](#factory-doc)	 |
| 	[`torch.rand()`](generated/torch.rand.html#torch.rand "torch.rand")	 | 	[Factory functions](#factory-doc)	 |
| 	[`torch.randn()`](generated/torch.randn.html#torch.randn "torch.randn")	 | 	[Factory functions](#factory-doc)	 |
| 	[`torch.randn()`](generated/torch.randn.html#torch.randn "torch.randn")	 | 	[Factory functions](#factory-doc)	 |
| 	[`Tensor.random_()`](generated/torch.Tensor.random_.html#torch.Tensor.random_ "torch.Tensor.random_")	 | 	 None	  |
| 	[`Tensor.reciprocal()`](generated/torch.Tensor.reciprocal.html#torch.Tensor.reciprocal "torch.Tensor.reciprocal")	 ,	 [`torch.reciprocal()`](generated/torch.reciprocal.html#torch.reciprocal "torch.reciprocal")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.reciprocal_()`](generated/torch.Tensor.reciprocal_.html#torch.Tensor.reciprocal_ "torch.Tensor.reciprocal_")	 | 	 None	  |
| 	[`Tensor.refine_names()`](named_tensor.html#torch.Tensor.refine_names "torch.Tensor.refine_names")	 | 	 See documentation	  |
| 	[`Tensor.register_hook()`](generated/torch.Tensor.register_hook.html#torch.Tensor.register_hook "torch.Tensor.register_hook")	 | 	 None	  |
| 	[`Tensor.register_post_accumulate_grad_hook()`](generated/torch.Tensor.register_post_accumulate_grad_hook.html#torch.Tensor.register_post_accumulate_grad_hook "torch.Tensor.register_post_accumulate_grad_hook")	 | 	 None	  |
| 	[`Tensor.rename()`](named_tensor.html#torch.Tensor.rename "torch.Tensor.rename")	 | 	 See documentation	  |
| 	[`Tensor.rename_()`](named_tensor.html#torch.Tensor.rename_ "torch.Tensor.rename_")	 | 	 See documentation	  |
| 	[`Tensor.requires_grad`](generated/torch.Tensor.requires_grad.html#torch.Tensor.requires_grad "torch.Tensor.requires_grad")	 | 	 None	  |
| 	[`Tensor.requires_grad_()`](generated/torch.Tensor.requires_grad_.html#torch.Tensor.requires_grad_ "torch.Tensor.requires_grad_")	 | 	 None	  |
| 	[`Tensor.resize_()`](generated/torch.Tensor.resize_.html#torch.Tensor.resize_ "torch.Tensor.resize_")	 | 	 Only allow resizes that do not change shape	  |
| 	[`Tensor.resize_as_()`](generated/torch.Tensor.resize_as_.html#torch.Tensor.resize_as_ "torch.Tensor.resize_as_")	 | 	 Only allow resizes that do not change shape	  |
| 	[`Tensor.round()`](generated/torch.Tensor.round.html#torch.Tensor.round "torch.Tensor.round")	 ,	 [`torch.round()`](generated/torch.round.html#torch.round "torch.round")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.round_()`](generated/torch.Tensor.round_.html#torch.Tensor.round_ "torch.Tensor.round_")	 | 	 None	  |
| 	[`Tensor.rsqrt()`](generated/torch.Tensor.rsqrt.html#torch.Tensor.rsqrt "torch.Tensor.rsqrt")	 ,	 [`torch.rsqrt()`](generated/torch.rsqrt.html#torch.rsqrt "torch.rsqrt")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.rsqrt_()`](generated/torch.Tensor.rsqrt_.html#torch.Tensor.rsqrt_ "torch.Tensor.rsqrt_")	 | 	 None	  |
| 	[`Tensor.select()`](generated/torch.Tensor.select.html#torch.Tensor.select "torch.Tensor.select")	 ,	 [`torch.select()`](generated/torch.select.html#torch.select "torch.select")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.short()`](generated/torch.Tensor.short.html#torch.Tensor.short "torch.Tensor.short")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.sigmoid()`](generated/torch.Tensor.sigmoid.html#torch.Tensor.sigmoid "torch.Tensor.sigmoid")	 ,	 [`torch.sigmoid()`](generated/torch.sigmoid.html#torch.sigmoid "torch.sigmoid")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.sigmoid_()`](generated/torch.Tensor.sigmoid_.html#torch.Tensor.sigmoid_ "torch.Tensor.sigmoid_")	 | 	 None	  |
| 	[`Tensor.sign()`](generated/torch.Tensor.sign.html#torch.Tensor.sign "torch.Tensor.sign")	 ,	 [`torch.sign()`](generated/torch.sign.html#torch.sign "torch.sign")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.sign_()`](generated/torch.Tensor.sign_.html#torch.Tensor.sign_ "torch.Tensor.sign_")	 | 	 None	  |
| 	[`Tensor.sgn()`](generated/torch.Tensor.sgn.html#torch.Tensor.sgn "torch.Tensor.sgn")	 ,	 [`torch.sgn()`](generated/torch.sgn.html#torch.sgn "torch.sgn")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.sgn_()`](generated/torch.Tensor.sgn_.html#torch.Tensor.sgn_ "torch.Tensor.sgn_")	 | 	 None	  |
| 	[`Tensor.sin()`](generated/torch.Tensor.sin.html#torch.Tensor.sin "torch.Tensor.sin")	 ,	 [`torch.sin()`](generated/torch.sin.html#torch.sin "torch.sin")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.sin_()`](generated/torch.Tensor.sin_.html#torch.Tensor.sin_ "torch.Tensor.sin_")	 | 	 None	  |
| 	[`Tensor.sinh()`](generated/torch.Tensor.sinh.html#torch.Tensor.sinh "torch.Tensor.sinh")	 ,	 [`torch.sinh()`](generated/torch.sinh.html#torch.sinh "torch.sinh")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.sinh_()`](generated/torch.Tensor.sinh_.html#torch.Tensor.sinh_ "torch.Tensor.sinh_")	 | 	 None	  |
| 	[`Tensor.asinh()`](generated/torch.Tensor.asinh.html#torch.Tensor.asinh "torch.Tensor.asinh")	 ,	 [`torch.asinh()`](generated/torch.asinh.html#torch.asinh "torch.asinh")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.asinh_()`](generated/torch.Tensor.asinh_.html#torch.Tensor.asinh_ "torch.Tensor.asinh_")	 | 	 None	  |
| 	[`Tensor.size()`](generated/torch.Tensor.size.html#torch.Tensor.size "torch.Tensor.size")	 | 	 None	  |
| 	[`Tensor.softmax()`](generated/torch.Tensor.softmax.html#torch.Tensor.softmax "torch.Tensor.softmax")	 ,	 [`torch.softmax()`](generated/torch.softmax.html#torch.softmax "torch.softmax")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.split()`](generated/torch.Tensor.split.html#torch.Tensor.split "torch.Tensor.split")	 ,	 [`torch.split()`](generated/torch.split.html#torch.split "torch.split")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.sqrt()`](generated/torch.Tensor.sqrt.html#torch.Tensor.sqrt "torch.Tensor.sqrt")	 ,	 [`torch.sqrt()`](generated/torch.sqrt.html#torch.sqrt "torch.sqrt")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.sqrt_()`](generated/torch.Tensor.sqrt_.html#torch.Tensor.sqrt_ "torch.Tensor.sqrt_")	 | 	 None	  |
| 	[`Tensor.squeeze()`](generated/torch.Tensor.squeeze.html#torch.Tensor.squeeze "torch.Tensor.squeeze")	 ,	 [`torch.squeeze()`](generated/torch.squeeze.html#torch.squeeze "torch.squeeze")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.std()`](generated/torch.Tensor.std.html#torch.Tensor.std "torch.Tensor.std")	 ,	 [`torch.std()`](generated/torch.std.html#torch.std "torch.std")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`torch.std_mean()`](generated/torch.std_mean.html#torch.std_mean "torch.std_mean")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.stride()`](generated/torch.Tensor.stride.html#torch.Tensor.stride "torch.Tensor.stride")	 | 	 None	  |
| 	[`Tensor.sub()`](generated/torch.Tensor.sub.html#torch.Tensor.sub "torch.Tensor.sub")	 ,	 [`torch.sub()`](generated/torch.sub.html#torch.sub "torch.sub")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.sub_()`](generated/torch.Tensor.sub_.html#torch.Tensor.sub_ "torch.Tensor.sub_")	 | 	[Unifies names from inputs](#unifies-names-from-inputs-doc)	 |
| 	[`Tensor.sum()`](generated/torch.Tensor.sum.html#torch.Tensor.sum "torch.Tensor.sum")	 ,	 [`torch.sum()`](generated/torch.sum.html#torch.sum "torch.sum")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.tan()`](generated/torch.Tensor.tan.html#torch.Tensor.tan "torch.Tensor.tan")	 ,	 [`torch.tan()`](generated/torch.tan.html#torch.tan "torch.tan")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.tan_()`](generated/torch.Tensor.tan_.html#torch.Tensor.tan_ "torch.Tensor.tan_")	 | 	 None	  |
| 	[`Tensor.tanh()`](generated/torch.Tensor.tanh.html#torch.Tensor.tanh "torch.Tensor.tanh")	 ,	 [`torch.tanh()`](generated/torch.tanh.html#torch.tanh "torch.tanh")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.tanh_()`](generated/torch.Tensor.tanh_.html#torch.Tensor.tanh_ "torch.Tensor.tanh_")	 | 	 None	  |
| 	[`Tensor.atanh()`](generated/torch.Tensor.atanh.html#torch.Tensor.atanh "torch.Tensor.atanh")	 ,	 [`torch.atanh()`](generated/torch.atanh.html#torch.atanh "torch.atanh")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.atanh_()`](generated/torch.Tensor.atanh_.html#torch.Tensor.atanh_ "torch.Tensor.atanh_")	 | 	 None	  |
| 	[`torch.tensor()`](generated/torch.tensor.html#torch.tensor "torch.tensor")	 | 	[Factory functions](#factory-doc)	 |
| 	[`Tensor.to()`](generated/torch.Tensor.to.html#torch.Tensor.to "torch.Tensor.to")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.topk()`](generated/torch.Tensor.topk.html#torch.Tensor.topk "torch.Tensor.topk")	 ,	 [`torch.topk()`](generated/torch.topk.html#torch.topk "torch.topk")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.transpose()`](generated/torch.Tensor.transpose.html#torch.Tensor.transpose "torch.Tensor.transpose")	 ,	 [`torch.transpose()`](generated/torch.transpose.html#torch.transpose "torch.transpose")	 | 	[Permutes dimensions](#permutes-dimensions-doc)	 |
| 	[`Tensor.trunc()`](generated/torch.Tensor.trunc.html#torch.Tensor.trunc "torch.Tensor.trunc")	 ,	 [`torch.trunc()`](generated/torch.trunc.html#torch.trunc "torch.trunc")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.trunc_()`](generated/torch.Tensor.trunc_.html#torch.Tensor.trunc_ "torch.Tensor.trunc_")	 | 	 None	  |
| 	[`Tensor.type()`](generated/torch.Tensor.type.html#torch.Tensor.type "torch.Tensor.type")	 | 	 None	  |
| 	[`Tensor.type_as()`](generated/torch.Tensor.type_as.html#torch.Tensor.type_as "torch.Tensor.type_as")	 | 	[Keeps input names](#keeps-input-names-doc)	 |
| 	[`Tensor.unbind()`](generated/torch.Tensor.unbind.html#torch.Tensor.unbind "torch.Tensor.unbind")	 ,	 [`torch.unbind()`](generated/torch.unbind.html#torch.unbind "torch.unbind")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.unflatten()`](generated/torch.Tensor.unflatten.html#torch.Tensor.unflatten "torch.Tensor.unflatten")	 | 	 See documentation	  |
| 	[`Tensor.uniform_()`](generated/torch.Tensor.uniform_.html#torch.Tensor.uniform_ "torch.Tensor.uniform_")	 | 	 None	  |
| 	[`Tensor.var()`](generated/torch.Tensor.var.html#torch.Tensor.var "torch.Tensor.var")	 ,	 [`torch.var()`](generated/torch.var.html#torch.var "torch.var")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`torch.var_mean()`](generated/torch.var_mean.html#torch.var_mean "torch.var_mean")	 | 	[Removes dimensions](#removes-dimensions-doc)	 |
| 	[`Tensor.zero_()`](generated/torch.Tensor.zero_.html#torch.Tensor.zero_ "torch.Tensor.zero_")	 | 	 None	  |
| 	[`torch.zeros()`](generated/torch.zeros.html#torch.zeros "torch.zeros")	 | 	[Factory functions](#factory-doc)	 |


## 保留输入名称 [¶](#keeps-input-names "永久链接到此标题")


 所有逐点一元函数以及其他一些一元函数都遵循此规则。



* 检查名称：无
* 传播名称：输入tensor的名称传播到输出。


```
>>> x = torch.randn(3, 3, names=('N', 'C'))
>>> x.abs().names
('N', 'C')

```


## 删除尺寸 [¶](#removes-dimensions "此标题的永久链接")


 所有归约操作如 [`sum()`](generated/torch.Tensor.sum.html#torch.Tensor.sum "torch.Tensor.sum") 通过减少所需的维度来删除维度。其他操作如 [`select()`](generated/torch.Tensor.select.html#torch.Tensor.select "torch.Tensor.select") 和 [`squeeze()`](generated/torch.Tensor.squeeze.html#torch.Tensor.squeeze "torch.Tensor.squeeze") 删除尺寸。


 只要可以将整数维度索引传递给运算符，就可以传递维度名称。采用维度索引列表的函数也可以采用维度名称列表。



* 检查名称：如果“dim”或“dims”作为名称列表传入，请检查这些名称是否存在于“self”中。
* 传播名称：如果输入tensor的维度由“dim”或“dims”指定` 不存在于输出tensor中，那么这些维度的相应名称不会出现在 `output.names` 中。


```
>>> x = torch.randn(1, 3, 3, 3, names=('N', 'C', 'H', 'W'))
>>> x.squeeze('N').names
('C', 'H', 'W')

>>> x = torch.randn(3, 3, 3, 3, names=('N', 'C', 'H', 'W'))
>>> x.sum(['N', 'C']).names
('H', 'W')

# Reduction ops with keepdim=True don't actually remove dimensions.
>>> x = torch.randn(3, 3, 3, 3, names=('N', 'C', 'H', 'W'))
>>> x.sum(['N', 'C'], keepdim=True).names
('N', 'C', 'H', 'W')

```


## 统一输入中的名称 [¶](#unizes-names-from-inputs "固定链接到此标题")


 所有二进制算术运算都遵循此规则。广播的操作仍然从右侧按位置广播，以保持与未命名tensor的兼容性。要按名称执行显式广播，请使用 [`Tensor.align_as()`](named_tensor.html#torch.Tensor.align_as "torch.Tensor.align_as") 。



* 检查名称：所有名称必须从右侧开始位置匹配。即，在 `tensor 
+ other` 中， `match(tensor.names[i], other.names[i])` 对于 `(-min(tensor.dim(), other.dim) 中的所有 `i` 必须为 true ()) 
+ 1, -1]`.
* 检查名称：此外，所有命名维度必须从右对齐。在匹配过程中，如果我们将命名维度 `A` 与未命名维度 `None` 匹配，则为 `A ` 不得出现在具有未命名维度的tensor中。
* 传播名称：统一两个tensor右侧的名称对以生成输出名称。


 例如，


```
# tensor: Tensor[ N, None]
# other: Tensor[None, C]
>>> tensor = torch.randn(3, 3, names=('N', None))
>>> other = torch.randn(3, 3, names=(None, 'C'))
>>> (tensor + other).names
('N', 'C')

```


 检查姓名：



* `match(tensor.names[-1], other.names[-1])` 为 `True`
* `match(tensor.names[-2], tensor.names[-2])` 为 `True` 
* 因为我们将 [`tensor`](generated/torch.tensor.html#torch.tensor "torch.tensor") 中的 `None` 与 `'C'` 匹配，请检查以确保 `'C'` 不匹配存在于 [`tensor`](generated/torch.tensor.html#torch.tensor "torch.tensor") 中(不存在)。
* 检查以确保 `'N'` 不存在于 `other` 中 (它不是)。


 最后，输出名称通过 `[unify('N', None), unify(None, 'C')] = ['N', 'C']` 计算


 更多示例：


```
# Dimensions don't match from the right:
# tensor: Tensor[N, C]
# other: Tensor[ N]
>>> tensor = torch.randn(3, 3, names=('N', 'C'))
>>> other = torch.randn(3, names=('N',))
>>> (tensor + other).names
RuntimeError: Error when attempting to broadcast dims ['N', 'C'] and dims
['N']: dim 'C' and dim 'N' are at the same position from the right but do
not match.

# Dimensions aren't aligned when matching tensor.names[-1] and other.names[-1]:
# tensor: Tensor[N, None]
# other: Tensor[ N]
>>> tensor = torch.randn(3, 3, names=('N', None))
>>> other = torch.randn(3, names=('N',))
>>> (tensor + other).names
RuntimeError: Misaligned dims when attempting to broadcast dims ['N'] and
dims ['N', None]: dim 'N' appears in a different position from the right
across both lists.

```


!!! note "笔记"

    在最后两个示例中，可以按名称对齐tensor，然后执行加法。使用 [`Tensor.align_as()`](named_tensor.html#torch.Tensor.align_as "torch.Tensor.align_as") 按名称对齐tensor或 [`Tensor.align_to()`](named_tensor.html #torch.Tensor.align_to "torch.Tensor.align_to") 将tensor对齐到自定义维度顺序。


## 排列维度 [¶](#permutes-dimensions "永久链接到此标题")


 某些操作，例如 [`Tensor.t()`](generated/torch.Tensor.t.html#torch.Tensor.t "torch.Tensor.t") ，会排列维度的顺序。维度名称附加到各个维度，因此它们也会被排列。


 如果运算符采用位置索引 `dim` ，它也可以采用维度名称作为 `dim` 。



* 检查名称：如果“dim”作为名称传递，请检查它是否存在于tensor中。 
* 传播名称：以与正在排列的维度相同的方式排列维度名称。


```
>>> x = torch.randn(3, 3, names=('N', 'C'))
>>> x.transpose('N', 'C').names
('C', 'N')

```


## Contracts away dims [¶](#contracts-away-dims "此标题的固定链接")


 矩阵乘法函数遵循此函数的一些变体。我们先看一下 [`torch.mm()`](generated/torch.mm.html#torch.mm "torch.mm")，然后概括批量矩阵乘法的规则。


 对于 `torch.mm(tensor，其他)` ：



* 检查名称：无
* 传播名称：结果名称为 `(tensor.names[-2], other.names[-1])` 。


```
>>> x = torch.randn(3, 3, names=('N', 'D'))
>>> y = torch.randn(3, 3, names=('in', 'out'))
>>> x.mm(y).names
('N', 'out')

```


 本质上，矩阵乘法在二维上执行点积，从而折叠它们。当两个tensor进行矩阵相乘时，收缩的维度会消失并且不会出现在输出tensor中。


[`torch.mv()`](generated/torch.mv.html#torch.mv "torch.mv") , [`torch.dot()`](generated/torch.dot.html#torch.dot " torch.dot") 以类似的方式工作：名称推断不会检查输入名称并删除点积中涉及的维度：


```
>>> x = torch.randn(3, 3, names=('N', 'D'))
>>> y = torch.randn(3, names=('something',))
>>> x.mv(y).names
('N',)

```


 现在，让我们看一下“torch.matmul(tensor, other)”。假设 `tensor.dim() >= 2` 和 `other.dim() >= 2` 。



* 检查名称：检查输入的批量维度是否对齐且可广播。请参阅[统一输入中的名称](#unizes-names-from-inputs-doc)，了解对齐输入的含义。
* 传播名称：结果名称是通过统一批量维度并删除收缩维度获得的： `unify(tensor.names[:-2], other.names[:-2]) 
+ (tensor.names[-2], other.names[ -1])` 。


 例子：


```
# Batch matrix multiply of matrices Tensor['C', 'D'] and Tensor['E', 'F'].
# 'A', 'B' are batch dimensions.
>>> x = torch.randn(3, 3, 3, 3, names=('A', 'B', 'C', 'D'))
>>> y = torch.randn(3, 3, 3, names=('B', 'E', 'F'))
>>> torch.matmul(x, y).names
('A', 'B', 'C', 'F')

```


 最后，还有许多 matmul 函数的融合“add”版本。即， [`addmm()`](generated/torch.addmm.html#torch.addmm "torch.addmm") 和 [`addmv()`](generated/torch.addmv.html#torch.addmv "torch.添加MV”)。这些被视为构成名称推断，即 [`mm()`](generated/torch.mm.html#torch.mm "torch.mm") 和 [`add()`](generated/torch.add.html#torch.add "torch.add").


## 工厂函数 [¶](#factory-functions "固定链接到此标题")


 工厂函数现在采用一个新的“names”参数，该参数将名称与每个维度相关联。


```
>>> torch.zeros(2, 3, names=('N', 'C'))
tensor([[0., 0., 0.],
 [0., 0., 0.]], names=('N', 'C'))

```


## 输出函数和就地变体 [¶](#out-function-and-in-place-variants "永久链接到此标题")


 指定为“out=”tensor的tensor具有以下行为：



* 如果它没有命名维度，则从操作计算的名称将传播到它。 
* 如果它有任何命名维度，则从操作计算的名称必须与现有名称完全相同。否则，操作出错。


 所有就地方法都会修改输入，使其名称等于通过名称推断计算出的名称。例如：


```
>>> x = torch.randn(3, 3)
>>> y = torch.randn(3, 3, names=('N', 'C'))
>>> x.names
(None, None)

>>> x += y
>>> x.names
('N', 'C')

```