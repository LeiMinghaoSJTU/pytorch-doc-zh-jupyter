# torch.utils.model_zoo [¶](#torch-utils-model-zoo "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/model_zoo>
>
> 原始地址：<https://pytorch.org/docs/stable/model_zoo.html>


 移至 torch.hub 。


 torch.utils.model_zoo。


 加载_url


 ( *url
* , *model_dir



 =
 


 无
* , *地图_位置



 =
 


 无
* , *进展



 =
 


 True
* , *检查_hash



 =
 


 假
* , *文件_name



 =
 


 无
* , *权重_only



 =
 


 False
* ) [¶](#torch.utils.model_zoo.load_url "此定义的永久链接")


 在给定 URL 加载 Torch 序列化对象。


 如果下载的文件是zip文件，它将自动解压缩。


 如果该对象已存在于 model_dir 中，则反序列化并返回。 `model_dir` 的默认值为 `<hub_dir>/checkpoints`，其中 `hub_dir` 是 [`get_dir 返回的目录()`](hub.html#torch.hub.get_dir "torch.hub.get_dir") 。


 参数 
* **url** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 对象的 URL下载
* **model_dir** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*,
* *可选
* ) – 保存对象的目录
* **map_location** ( *可选
* ) – 指定如何重新映射存储位置的函数或字典(参见 torch.load)
* **progress** ( [
* bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 是否显示进度条stderr.Default: True
* **check_hash** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")
* ,
* *可选
* ) – 如果为 True，则 URL 的文件名部分应遵循命名约定 `filename-<sha256>.ext`，其中 `<sha256>` 是内容的 SHA256 哈希值的前八位或更多位文件。哈希用于确保唯一名称并验证文件的内容。默认值：False
* **file_name** ( [*str*](https://docs.python.org/3/library/stdtypes. html#str "(Python v3.12)")*,
* *可选
* ) – 下载文件的名称。如果未设置，将使用“url”中的文件名。
* **weights_only** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(Python 中) v3.12)")*,
* *可选
* ) – 如果为 True，则仅加载权重，不会加载复杂的 pickled 对象。建议用于不受信任的来源。有关更多详细信息，请参阅 [`load()`](generated/torch.load.html#torch.load "torch.load")。


 Return type


[*Dict*](https://docs.python.org/3/library/typing.html#typing.Dict "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(Python v3.12)") , [*Any*](https://docs.python.org/3/library/typing.html#typing.Any " (在 Python v3.12 中)") ]


 例子


```
>>> state_dict = torch.hub.load_state_dict_from_url('https://s3.amazonaws.com/pytorch/models/resnet18-5c106cde.pth')

```