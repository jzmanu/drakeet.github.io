---
title: 使用 Python 开发 LeanCloud 云引擎入门教程
---

作为独立应用开发者，可能很多时候没有能力或者精力再去开发一个为 App 或者网站服务的后端程序，对于这个问题，我的一个解决方案就是使用 LeanCloud 的云引擎功能，通过云引擎，可以在短时间内快速开发出一个稳定、可靠、可升级拓展的服务。正好最近学习了 Python，于是使用 Python 并基于 LeanCloud 的云引擎功能进行了一次实践，而且由于官方文档并没有写得很好，我碰了几次壁，而且内容分散在好几个页面，所以觉得有必要在此再详细记录下并分享给大家。

注：以下部分内容并非完全原创，有的摘自官方文档，官方文档链接在文末有给出。
<h3>零，你得先学会 Python</h3>
JS 也是可以的，但由于我这篇文章主讲的就是使用 Python 来开发，所以要求至少得懂得 Python。不过不懂也不要紧，只要有兴趣肯去学，一个 Python 最多也就一两周的事情。我也是上一周利用一点点业余时间学了几下就懂了，毕竟 Python 的名言是“人生苦短，我用 Python”...
<h3>一，注册 LeanCloud 账号并创建一个应用</h3>
关于这点，我就不细说了，这是一个非常简单直接的过程，没听说过 LeanCloud 的同学可以自己上官网查看捣鼓几下就知道了：<a href="https://leancloud.cn" target="_blank">https://leancloud.cn</a>
<h3>二，进入应用的云引擎页面</h3>
云引擎的入口如下：

<h3><img class="" src="http://ww2.sinaimg.cn/large/86e2ff85gw1f0d54nfg4ej20n70eldjq.jpg" alt="" width="443" height="279" /></h3>

进入之后，点击左侧的设置，我们先来自定义一个子域名。进入“设置”页面之后，可以看到右边页面有一个”Web 主机域名“选项，可以填入你想要的子域名名字，比如我在我的这个测试应用的这边填的是：lunei-dev，那么等到一切都部署完之后，我们要访问的基本链接就是：lunei-dev.leanapp.cn
<h3>三，一些准备工作</h3>
需要先安装 LeanCloud 的命令行工具，也就是在终端中使用的。安装方式是在终端中输入：

```bash
npm install -g avoscloud-code
```

但由于 npm 已经被显而易见的不可抗力变为无法访问的存在，所以我们需要在调用它之前加一点黑魔法，详情见这篇文章：<a href="http://blog.fazero.cc/2015/08/31/%E5%88%A9%E7%94%A8proxychains%E5%9C%A8%E7%BB%88%E7%AB%AF%E4%BD%BF%E7%94%A8socks5%E4%BB%A3%E7%90%86/" target="_blank">利用proxychains在终端使用socks5代理</a>

这篇文章中，作者说的 cp ./src/proxychains.conf /etc/proxychians.conf 其实应该加上 sudo，不然可能会导致没有权限而复制失败，检查方式就是使用 vi 打开 /etc/proxychians.conf 这个文件，如果第一次打开，内容是空的，说明没有复制成功，因为正常情况下，它有许多默认内容。

搞定 proxychains4 以后，你就可以使用如下命令进行安装 LeanCloud 的命令行工具了：

```bash
sudo proxychains4 npm install -g avoscloud-code
```

<h3>四，创建要部署到云引擎上的应用</h3>
我们可以基于 LeanCloud 官方提供的一个 Python 项目模板，其源代码是在 GitHub 上开源的，使用如下命令：
<!--more-->

```bash
$ git clone https://github.com/leancloud/python-getting-started.git
$ cd python-getting-started
```

然后添加应用 appId 等信息到该项目：

```bash
$ avoscloud add <APP-NAME> <APP-ID>
```

上面中，&lt;APP-NAME&gt; 应该替换成你的应用名称（可以随意，但最好是和你在第一步中创建的应用名字一样），&lt;APP-ID&gt; 替换为你的应用 ID，这个 App ID 可以从你的 LeanCloud 应用的这个页面中获得：
<h3><img class="" src="http://ww2.sinaimg.cn/large/86e2ff85gw1f0d5oyjsqnj20uc0bl771.jpg" alt="" width="552" height="211" /></h3>
<h3>五，本地运行</h3>
首先需要安装 Python 依赖：

```bash
$ sudo pip install -Ur requirements.txt
```

如果你的电脑中既有 Python 2.7 又有 Python 3，那建议你使用如下命令进行安装：

```bash
$ sudo pip2.7 install -Ur requirements.txt
```

因为 LeanCloud 的云引擎目前只支持 2.7，而且这么做之后可以避免后续一些可能的问题（俺就遇到）。

另外，这句命令中 requirements.txt 这个文件就是书写你的项目需要的第三方库的文件，它里面已经有指定要引入一些第三方库了。如果你要引入更多第三方库，最好是把它也写入这个文件，然后重新运行上面这个命令更新一下。

另外，在 Mac OSX 10.11 上运行上面这句代码可能会不能通过，原因是 six 这个库有点问题，可以使用如下命令忽略它的问题进行安装即可(pip or pip2.7)：

```bash
$ sudo pip2.7 install -Ur requirements.txt --ignore-installed six
```

然后就可以启动应用了，通过在当前文件夹目录运行这个命令即可：

```bash
avoscloud
```

窗口会提示输入 Master Key，该信息可以在 <strong>控制台 / 设置 / 应用 Key</strong> 中找到，也就是同第四步那个截图页面，在 APP ID 下面。

<strong>复制粘贴 Master Key 后，窗口不会有任何显示，直接按回车键确认即可。</strong>

应用即可启动运行：<a href="http://localhost:3000" target="_blank">http://localhost:3000</a>
<h3>六，部署到云引擎</h3>
部署到预备环境：

```bash
$ avoscloud deploy
```

如果你设置了 二级域名，即可通过 http://stg-${your_app_domain}.leanapp.cn 访问你应用的预备环境（测试环境）。比如我的 http://stg-lunei-dev.leanapp.cn

部署到生产环境：

```bash
$ avoscloud publish
```

如果你设置了 二级域名，即可通过 http://${your_app_domain}.leanapp.cn 访问你应用的生产环境，比如我的 http://lunei-dev.leanapp.cn

截止到目前为止，LeanCloud 官方的 python-getting-started 这个 demo 或称模板项目就运行和发布完了，你可以通过你的链接访问到如下页面：
<h3><img class="" src="http://ww1.sinaimg.cn/large/86e2ff85gw1f0d62u0cxvj20bg06974v.jpg" alt="" width="335" height="183" /></h3>
<h3>最后</h3>
最后，如果你是 Python 新手，由于这个模板项目是基于 Python flask 框架，我可以给你一个修改这个 python-getting-started  项目的示例，并访问 http://${your_app_domain}.leanapp.cn/xxxxxx 获得你指定的返回内容，这样可以做为 App 的接口服务或称 API：

同样是在 python-getting-started 目录之下，打开 app.py，在顶上新增一行引入：

```python
from flask import request
```

然后在底下新增一个方法：

```python
@app.route('/get_json', methods=['GET'])
def get_json():
    user_name = request.args.get('user_name')
    return r'{"hello": "' + user_name + r'"}'
```

然后保存这个文件，重新在终端中输入 avoscloud 即可重新运行这个新的服务程序了，你可以在本地访问 <a href="http://localhost:3000/get_json?user_name=drakeet" target="_blank">http://localhost:3000/get_json?user_name=drakeet</a> 进行测试，将会返回：

```json
{"hello": "drakeet"}
```

&nbsp;
<h3>参考链接：</h3>
<ul>
	<li>官方模板 <a href="https://github.com/leancloud/python-getting-started.git" target="_blank">https://github.com/leancloud/python-getting-started.git</a></li>
	<li>官方文档 <a href="https://leancloud.cn/docs/leanengine_guide-python.html" target="_blank">https://leancloud.cn/docs/leanengine_guide-python.html</a></li>
<li>官方命令行工具的更多用法 <a href="https://leancloud.cn/docs/cloud_code_commandline.html" target="_blank">https://leancloud.cn/docs/cloud_code_commandline.html</a></li>
</ul>
