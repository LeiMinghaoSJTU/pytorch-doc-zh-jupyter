# torch.hub [¶](#torch-hub "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/hub>
>
> 原始地址：<https://pytorch.org/docs/stable/hub.html>


 Pytorch Hub 是一个预先训练的模型存储库，旨在促进研究的可重复性。


## 发布模型 [¶](#publishing-models "此标题的永久链接")


 Pytorch Hub 支持通过添加简单的“hubconf.py”文件将预训练模型(模型定义和预训练权重)发布到 GitHub 存储库；


`hubconf.py` 可以有多个入口点。每个入口点都定义为一个 python 函数(例如：要发布的预训练模型)。


```
def entrypoint_name(*args, **kwargs):
    # args & kwargs are optional, for models which take positional/keyword arguments.
    ...

```


### 如何实现入口点？ [¶](#how-to-implement-an-entrypoint"此标题的永久链接")


 如果我们扩展“pytorch/vision/hubconf.py”中的实现，这里的代码片段指定了“resnet18”模型的入口点。在大多数情况下，在“hubconf.py”中导入正确的函数就足够了。这里我们只是想用扩展版本作为例子来展示它是如何工作的。你可以在[pytorch/vision repo](https://github.com/pytorch/vision/blob/master/hubconf.py)中看到完整的脚本)



```
dependencies = ['torch']
from torchvision.models.resnet import resnet18 as _resnet18

# resnet18 is the name of entrypoint
def resnet18(pretrained=False, **kwargs):
 """ # This docstring shows up in hub.help()
 Resnet18 model
 pretrained (bool): kwargs, load pretrained weights into the model
 """
    # Call the model, load pretrained weights
    model = _resnet18(pretrained=pretrained, **kwargs)
    return model

```



* `dependency` 变量是 **加载** 模型所需的包名称的 **列表**。请注意，这可能与训练模型所需的依赖项略有不同。
* `args` 和 `kwargs` 被传递给真正的可调用函数。
* 函数的文档字符串用作帮助消息。它解释了模型的作用以及允许的位置/关键字参数。强烈建议在此处添加一些示例。
* 入口点函数可以返回模型(nn.module)，也可以返回辅助工具以使用户工作流程更顺畅，例如
* 带有下划线前缀的可调用对象被视为辅助函数，不会出现在 [`torch.hub.list()`](#torch.hub.list "torch.hub.list") 中。
* 预训练权重可以可以本地存储在 GitHub 存储库中，也可以通过 [`torch.hub.load_state_dict_from_url()`](#torch.hub.load_state_dict_from_url "torch.hub.load_state_dict_from_url") 加载。如果小于 2GB，建议将其附加到[项目版本](https://help.github.com/en/articles/distributing-large-binaries) 并使用版本中的 url。在上面的示例中` torchvision.models.resnet.resnet18` 处理 `pretrained` ，或者您可以将以下逻辑放在入口点定义中。


```
if pretrained:
    # For checkpoint saved in local GitHub repo, e.g. <RELATIVE_PATH_TO_CHECKPOINT>=weights/save.pth
    dirname = os.path.dirname(__file__)
    checkpoint = os.path.join(dirname, <RELATIVE_PATH_TO_CHECKPOINT>)
    state_dict = torch.load(checkpoint)
    model.load_state_dict(state_dict)

    # For checkpoint saved elsewhere
    checkpoint = 'https://download.pytorch.org/models/resnet18-5c106cde.pth'
    model.load_state_dict(torch.hub.load_state_dict_from_url(checkpoint, progress=False))

```


### 重要通知 [¶](#important-notice"永久链接到此标题")



* 发布的模型至少应该在一个分支/标签中。它不可能是随机提交。


## 从 Hub 加载模型 [¶](#loading-models-from-hub "固定链接到此标题")


 Pytorch Hub 提供了方便的 API 来探索 hub 中的所有可用模型，通过 [`torch.hub.list()`](#torch.hub.list "torch.hub.list") ，通过 [`torch.hub.list()`](#torch.hub.list "torch.hub.list") 显示文档字符串和示例。 help()`](#torch.hub.help "torch.hub.help") 并使用 [`torch.hub.load()`](#torch.hub.load "torch.hub" 加载预训练模型。加载”) 。


 火炬中心。



 list
 


 ( *github
* , *force_reload



 =
 


 False
* , *跳过_validation



 =
 


 假*，*信任_repo



 =
 


 无
* ) [[source]](_modules/torch/hub.html#list)[¶](#torch.hub.list "此定义的永久链接")


 列出 `github` 指定的存储库中可用的所有可调用入口点。


 参数 
* **github** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 格式为“的字符串” repo_owner/repo_name[:ref]”，带有可选的ref(标签或分支)。如果未指定 ref ，则默认分支如果存在则假定为 main ，否则默认分支为 master 。示例： 'pytorch/vision:0.10'
* **force_reload** ( [*bool*] (https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 是否放弃现有缓存并强制重新下载。默认值为 `False`.
* **skip_validation** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") *,
* *可选
* ) – 如果 `False` ，torchhub 将检查 `github` 参数指定的分支或提交是否正确属于存储库所有者。这将向 GitHub API 发出请求；您可以通过设置 `GITHUB_TOKEN` 环境变量来指定非默认 GitHub 令牌。默认值为 `False`.
* **trust_repo** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*或
* *无
* ) –


`"check"` 、 `True` 、 `False` 或 `None` 。此参数在 v1.12 中引入，有助于确保用户仅运行他们信任的存储库中的代码。



+ 如果为“False”，则会出现提示询问用户是否应信任该存储库。 
+ 如果为 True ，则存储库将被添加到受信任列表中并加载，无需明确确认。 
+ 如果为 `"check"` ，则将根据缓存中受信任的存储库列表检查存储库。如果该列表中不存在，则行为将回退到“trust_repo=False”选项。 
+ 如果 `None` ：这将引发警告，邀请用户将 `trust_repo` 设置为 `False` 、 `True` 或 `"check"` 。这只是为了向后兼容而存在，并将在 v2.0 中删除。默认为“None”，最终在 v2.0 中更改为“check”。


 退货


 可用的可调用入口点


 Return type


[列表](https://docs.python.org/3/library/stdtypes.html#list“(在Python v3.12中)”)


 例子


```
>>> entrypoints = torch.hub.list('pytorch/vision', force_reload=True)

```


 火炬中心。



 help
 


 ( *github
* , *model
* , *force_reload



 =
 


 False
* , *跳过_validation



 =
 


 假*，*信任_repo



 =
 


 无
* ) [[source]](_modules/torch/hub.html#help)[¶](#torch.hub.help "此定义的永久链接")


 显示入口点 `model` 的文档字符串。


 参数 
* **github** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 格式为 < 的字符串repo_owner/repo_name[:ref]
> 带有可选的ref(标签或分支)。如果未指定 ref ，则默认分支假设为 main (如果存在)，否则为 master 。示例： 'pytorch/vision:0.10'
* **model** ( [*str*](https ://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 在存储库的 `hubconf.py` 中定义的入口点名称字符串
* **force_reload
* 
* ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 是否丢弃现有的缓存并强制重新下载。默认为 `False`.
* **skip_validation** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `False` ，torchhub 将检查 `github` 参数指定的引用是否正确属于存储库所有者。这将向 GitHub API 发出请求；您可以通过设置 `GITHUB_TOKEN` 环境变量来指定非默认 GitHub 令牌。默认值为 `False`.
* **trust_repo** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*或
* *无
* ) –


`"check"` 、 `True` 、 `False` 或 `None` 。此参数在 v1.12 中引入，有助于确保用户仅运行他们信任的存储库中的代码。



+ 如果为“False”，则会出现提示询问用户是否应信任该存储库。 
+ 如果为 True ，则存储库将被添加到受信任列表中并加载，无需明确确认。 
+ 如果为 `"check"` ，则将根据缓存中受信任的存储库列表检查存储库。如果该列表中不存在，则行为将回退到“trust_repo=False”选项。 
+ 如果 `None` ：这将引发警告，邀请用户将 `trust_repo` 设置为 `False` 、 `True` 或 `"check"` 。这只是为了向后兼容而存在，并将在 v2.0 中删除。默认为“None”，最终在 v2.0 中更改为“check”。


 例子


```
>>> print(torch.hub.help('pytorch/vision', 'resnet18', force_reload=True))

```


 火炬中心。



 load
 


 ( *repo_or_dir
* , *model
* , **


 参数*，*源



 =
 


 'github'
* , *trust_repo



 =
 


 无
* , *强制_reload



 =
 


 错误*、*冗长



 =
 


 True
* , *跳过_validation



 =
 


 错误的
* ， ***


 kwargs
* ) [[source]](_modules/torch/hub.html#load)[¶](#torch.hub.load "此定义的永久链接")


 从 github 存储库或本地目录加载模型。


 注意：加载模型是典型的用例，但这也可以用于加载其他对象，例如分词器、损失函数等。


 如果“source”是“github”，则“repo_or_dir”应采用“repo_owner/repo_name[:ref]”形式，并带有可选引用(标签或分支)。


 如果“source”是“local”，则“repo_or_dir”应该是本地目录的路径。


 参数 
* **repo_or_dir** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 如果`source` 是 'github'，这应该对应于格式为 `repo_owner/repo_name[:ref]` 的 github 存储库，并带有可选的引用(标签或分支)，例如“pytorch/vision:0.10”。如果未指定“ref”，则默认分支假定为“main”(如果存在)，否则为“master”。如果“source”为“local”，则它应该是本地目录的路径。
* ** model** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 定义的可调用(入口点)的名称在repo/dir 的 `hubconf.py`.
* ***args** ( *可选
* ) – 可调用 `model` 的相应参数.
* **source** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* *可选
* ) – 'github' 或 'local'。指定如何解释 `repo_or_dir`。默认为 'github'.
* **trust_repo** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*或
* *无
* ) –


`"check"` 、 `True` 、 `False` 或 `None` 。此参数在 v1.12 中引入，有助于确保用户仅运行他们信任的存储库中的代码。



+ 如果为“False”，则会出现提示询问用户是否应信任该存储库。 
+ 如果为 True ，则存储库将被添加到受信任列表中并加载，无需明确确认。 
+ 如果为 `"check"` ，则将根据缓存中受信任的存储库列表检查存储库。如果该列表中不存在，则行为将回退到“trust_repo=False”选项。 
+ 如果 `None` ：这将引发警告，邀请用户将 `trust_repo` 设置为 `False` 、 `True` 或 `"check"` 。这只是为了向后兼容而存在，并将在 v2.0 中删除。默认值为 `None`，最终在 v2.0 中更改为 `"check"`。
* **force_reload** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 是否强制无条件重新下载 github 存储库。如果 `source = 'local'` 则没有任何效果。默认值为 `False`.
* **详细** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*, 
* *可选
* ) – 如果为“False”，则静音有关命中本地缓存的消息。请注意，有关首次下载的消息无法静音。如果 `source = 'local'` 则没有任何效果。默认为 `True` 。
* **skip_validation** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `False` ，torchhub 将检查 `github` 参数指定的分支或提交是否正确属于存储库所有者。这将向 GitHub API 发出请求；您可以通过设置 `GITHUB_TOKEN` 环境变量来指定非默认 GitHub 令牌。默认值为 `False`.
* ****kwargs** ( *可选
* ) – 可调用 `model` 对应的 kwargs。


 退货


 使用给定的 `*args` 和 `**kwargs` 调用时可调用的 `model` 的输出。


 例子


```
>>> # from a github repo
>>> repo = 'pytorch/vision'
>>> model = torch.hub.load(repo, 'resnet50', weights='ResNet50_Weights.IMAGENET1K_V1')
>>> # from a local directory
>>> path = '/some/local/path/pytorch/vision'
>>> model = torch.hub.load(path, 'resnet50', weights='ResNet50_Weights.DEFAULT')

```


 火炬中心。


 下载_url_到_文件


 ( *url
* 、 *dst
* 、 *hash_prefix



 =
 


 无
* , *进展



 =
 


 True
* ) [[source]](_modules/torch/hub.html#download_url_to_file)[¶](#torch.hub.download_url_to_file "此定义的永久链接")


 将给定 URL 处的对象下载到本地路径。


 参数 
* **url** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 对象的 URL download
* **dst** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 对象所在的完整路径被拯救，例如`/tmp/temporary_file`
* **hash_prefix** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中) ")*,
* *可选
* ) – 如果不是“无”，则 SHA256 下载的文件应以 `hash_prefix` 开头。默认：无
* **进度** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 是否向 stderr 显示进度条默认: True


 例子


```
>>> torch.hub.download_url_to_file('https://s3.amazonaws.com/pytorch/models/resnet18-5c106cde.pth', '/tmp/temporary_file')

```


 火炬中心。


 从_url 加载_state_dict_


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
* ) [[source]](_modules/torch/hub.html#load_state_dict_from_url)[¶](#torch.hub.load_state_dict_from_url "此定义的永久链接")


 在给定 URL 加载 Torch 序列化对象。


 如果下载的文件是zip文件，它将自动解压缩。


 如果该对象已存在于 model_dir 中，则会反序列化并返回。 `model_dir` 的默认值为 `<hub_dir>/checkpoints`，其中 `hub_dir` 是 [`get_dir` 返回的目录()`](#torch.hub.get_dir "torch.hub.get_dir") 。


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


### 运行加载的模型：[¶](#running-a-loaded-model "Permalink to this header")


 请注意，[`torch.hub.load()`](#torch.hub.load "torch.hub.load") 中的 `*args` 和 `**kwargs` 用于**实例化**一个模型。加载模型后，如何了解可以使用该模型做什么？建议的工作流程是



* `dir(model)` 查看模型的所有可用方法。
* `help(model.foo)` 检查 `model.foo` 需要哪些参数来运行


 为了帮助用户探索而无需来回参考文档，我们强烈建议存储库所有者使功能帮助消息清晰简洁。包含一个最小的工作示例也很有帮助。


### 我下载的模型保存在哪里？ [¶](#where-are-my-downloaded-models-saved"此标题的永久链接")


 这些位置按以下顺序使用



* 如果设置了环境变量 `TORCH_HOME`，则调用 `hub.set_dir(<PATH_TO_HUB_DIR>)`
* `$TORCH_HOME/hub` 。
* `$XDG_CACHE_HOME/torch/hub` ，如果设置了环境变量 `XDG_CACHE_HOME`。
* `~/.cache/torch/hub`


 火炬中心。


 获取_dir


 ( ) [[source]](_modules/torch/hub.html#get_dir)[¶](#torch.hub.get_dir "此定义的永久链接")


 获取用于存储下载的模型和权重的 Torch Hub 缓存目录。


 如果未调用 [`set_dir()`](#torch.hub.set_dir "torch.hub.set_dir")，则默认路径为 `$TORCH_HOME/hub`，其中环境变量 `$TORCH_HOME` 默认为`$XDG_CACHE_HOME/torch` 。 `$XDG_CACHE_HOME` 遵循 Linux 文件系统布局的 X Design Group 规范，如果未设置环境变量，则使用默认值 `~/.cache`。


 火炬中心。


 设置_dir


 ( *d
* ) [[source]](_modules/torch/hub.html#set_dir)[¶](#torch.hub.set_dir "此定义的永久链接")


 (可选)设置用于保存下载的模型和权重的 Torch Hub 目录。


 Parameters


**d** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要保存的本地文件夹的路径下载的模型和权重。


### 缓存逻辑 [¶](#caching-logic "此标题的永久链接")


 默认情况下，我们在加载文件后不会清理文件。如果 [`get_dir()`](#torch.hub.get_dir "torch.hub.get_dir") 返回的目录中已存在该缓存，则 Hub 默认使用缓存。


 用户可以通过调用 hub.load(...,force_reload=True) 来强制重新加载。这将删除现有的 GitHub 文件夹和下载的权重，重新初始化新的下载。当更新发布到同一分支时，这很有用，用户可以跟上最新版本。


### 已知限制：[¶](#known-limitations"永久链接到此标题")


 Torch hub 的工作方式是导入软件包，就像安装它一样。在 Python 中导入会带来一些副作用。例如，您可以在 Python 缓存 `sys.modules` 和 `sys.path_importer_cache` 中看到新项目，这是正常的 Python 行为。这也意味着从不同存储库导入不同模型时，如果存储库具有相同的子包名称(通常是“mo​​del”子包)。解决此类导入错误的方法是从“sys.modules”字典中删除有问题的子包；更多详细信息可以在[此 GitHub 问题](https://github.com/pytorch/hub/issues/243#issuecomment-942403391) 中找到。


 这里值得一提的已知限制是：用户**不能**在**相同的 python 进程**中加载同一存储库的两个不同分支。这就像在Python中安装两个同名的包一样，这是不好的。如果您真正尝试的话，缓存可能会加入其中并给您带来惊喜。当然，将它们加载到单独的进程中是完全可以的。