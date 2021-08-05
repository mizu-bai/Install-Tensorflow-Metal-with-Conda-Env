# 【教程】 Intel Mac 在 Conda 环境中安装 tensorflow-macos 和 tensorflow-metal

看了一下 Apple 的 [apple/tensorflow_macos](https://github.com/apple/tensorflow_macos) 仓库已经归档了，并给了一个链接 [Getting Started with tensorflow-metal PluggableDevice](https://developer.apple.com/metal/tensorflow-plugin/).

之前使用前面一个版本的 tensorflow 跑过一点模型，虽然能使用 GPU 加速了（但 Radeon Pro 555X 聊剩于无吧），但出现过用 CPU 跑能正常收敛，GPU 上 Acc 就变成 1 / num_class 了。于是打算尝试一下这个新版的 tensorflow-macos 和 tensorflow-metal。

官方推荐的 x86-64 Mac 安装方法是利用系统的 python3  创建一个虚拟环境后，再安装以上两个包，而对 arm64 Mac 推荐安装 miniforge 后在 conda 环境里安装。这区别对待有点……所以打算在 Intel Mac 上也把包装在 conda 环境里，毕竟方便管理很多。

然后就开始尝试安装了，但报错如下，无法安装：

```shell
$ pip3 install tensorflow-macos
Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
ERROR: Could not find a version that satisfies the requirement tensorflow-macos (from versions: none)
ERROR: No matching distribution found for tensorflow-macos
```

似乎是没有找到一个能符合需求的版本，搜索了一下好像也只有找到安装原版 tensorflow 出问题的例子。检查过自己的 python 是 3.8 版本，在 pypi 上查找这两个包的相关信息，tensorflow-macos 2.5.0 支持的 python 版本范围为 3.6 ~ 3.8，tensorflow-metal 0.1.1 作为一个 metal 插件也没有版本要求。

浏览了相关问题后，发现可以使用 python 自带的 platform 模块来打印相关平台信息，以此来定位问题。于是尝试了一下系统自带的 python3 输出的结果。

```shell
$ /usr/bin/python3 -c "import platform; print(platform.mac_ver())"
('11.4', ('', '', ''), 'x86_64')
```

然后又试了一下，conda 环境里的 python3

```shell
$ conda activate tf-metal-env
(tf-metal-env) $ python3 -c "import platform; print(platform.mac_ver())"
('10.16', ('', '', ''), 'x86_64')
```

嗯？这系统版本？

又查了点资料，发现 pip 获取系统信息的方法在 `~/miniforge/envs/tf-metal-env/lib/python3.8/site-packages/pip/_vendor/packaging/tags.py` 文件中，打开定位到第 429 行，

```python3
version_str, _, cpu_arch = platform.mac_ver() 
```

手动打 patch，在此行后新加一行，

```python3
version_str, _, cpu_arch = platform.mac_ver() 
version_str = "11.4"
```

再次运行以下命令，

```shell
(tf-metal-env) $ pip3 install tensorflow-macos tensorflow-metal
```

就成功装上了。

此处 patch 似乎对其他依赖包的安装不造成影响，笔者同时安装了 numpy，pandas ，matplolib，pillow 等包都未出现问题。

再次吐槽一下自 Big Sur 起 macOS 版本号混乱的问题！！！

