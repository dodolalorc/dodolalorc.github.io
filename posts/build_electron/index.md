# Electron 构建记录


> System：Windows 11
>
> Python：Python 3.13.3
>
> node：v22.14.0
>
> npm：10.9.2
>
> yarn：1.22.22
>
> 足够的磁盘空间和流量

## 初始化环境

`Electron`  是嵌入  [Chromium](https://www.chromium.org/)  和  [Node.js](https://nodejs.org/)  到 二进制进行开发，所以在构建`Electron`时显然也要准备这两者的开发环境。

### Chromium 开发工具

参考[Chromium 此页](https://chromium.googlesource.com/chromium/tools/depot_tools.git)拉取`depot_tools`。

拉取完毕之后，将`depot_tools`的路径加入到系统变量`path`中，并将变量位置上移到至少`git`变量之前。

`depot_tools`需要本机安装了`Git for Windows`，以及`python 3.8`。

#### 识别不到安装的`git`

报错信息：

```bash
$ gclient --version
Updating depot_tools...
WARNING:root:depot_tools will stop bundling Git for Windows on 2025-01-27.
To prepare for this change, please install Git directly. See
https://chromium.googlesource.com/chromium/src/+/main/docs/windows_build_instructions.md#Install-git

Having issues and not ready for depot_tools to stop bundling
Git for Windows? File a bug at:
https://issues.chromium.org/issues/new?component=1456702&template=2045785

ERROR:root:Failed to bootstrap depot_tools.
Git was not found in PATH. Have you installed it?
```

本机使用[Scoop](https://scoop.sh/)安装大部分的软件，难过的是通过这样下载的`git`无法被`depot_tools`工具识别，只能卸载本机当前的`git`后在[git](https://git-scm.com/downloads)重新下载一遍，之后的`gclient`。

### Electron build-tools

本篇使用的脚本是[Electron build-tools](https://github.com/electron/build-tools)。

按照 readme 的提示安装：

```bash
npm i -g @electron/build-tools
```

在本机找好存放源码的位置后，进行：

```bash
e init --root=~/electron --bootstrap testing
```

`--root=~/electron`是拉取的时候想要存源码的位置，如果留空则在当前路径下，建议选择合适的磁盘，至少光是源码已经占据 20 多个 G 了，考虑到之后还有依赖包。

#### 识别不到命令

在`e init ...`命令执行到一半，发现中断在某处，重新进行`e sync -f`后，出现找不到 npm 命令的反馈。

```bash
$ e sync -f
Checking for build-tools updates
'npm' 不是内部或外部命令，也不是可运行的程序或批处理文件
'npx' 不是内部或外部命令，也不是可运行的程序或批处理文件
```

进行拉取时，脚本在执行到大约`python script/lib/npx.py yarn@1.15.2 install --frozen-lockfile`的时候发生了报错，称`FileNotFoundError: [WinError 2]`，导致`ERROR gclient sync failed`失败。

仔细阅读日志之后，先在命令行直接执行上述脚本语句，发现运行是正常的，同时想的本机在平时开发使用 python、node.js 均没有出现找不到命令的问题，且实验当时在命令行测试`node -v`、`yarn --version`和`npm -v`都有版本返回，排错查看环境变量`path`也有正确设置。

本机使用了 node.js 版本管理工具`fnm`，继而想到或许需要激活 node 环境，于是先运行了`fnm use v22`，发现继续的拉取依然有问题，想到或许是通过`fnm`设置的临时环境会不被识别，于是选择直接在 [node.js](https://nodejs.org/en) 官网下载一个合适的版本，这样操作之后能成功运行完毕。

不太确定`nvm`会不会也有这样的问题，但是插个眼，解决方法还是比较简单的。

## 构建

附一张成功拉取的截图(嘻嘻)

![image.png](https://img.dodolalorc.cn/i/2025/04/26/680c5ddac8c6d.png)

在进行`e build`之后，初次构建需要鉴权，点击命令行提示的网址操作一下就好啦~

然后就是继续泡一杯咖啡等待构建。

