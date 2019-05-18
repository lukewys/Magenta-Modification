# Magenta魔改记-0：Magetna初见

## 前言：

最近在魔改Magenta，所以会涉及到阅读、修改Magenta的源代码。我个人认为Magenta是一个很好的项目，虽然代码可能写的有点乱，但模型的架构写的都很好，从主观上来看（听），做出的自动作曲模型效果也很优异。Magenta可以为做深度学习自动作曲的研究者提供一个参考，同时我在这里也希望给有兴趣的朋友提供一些经验。

**Note:我的解决方法不一定是最优的，除了我的解决方案之外，读者可以自行参考Magenta官方给出的一些文档（尽管相当散乱且不连续）。本文的目标是尽量使读者快速了解Magenta项目以及代码结构，并着手按自己的想法修改。**

**Update：2019.5：修改了一些内容并更新了Magenta的版本。**

## Magenta是什么：

Magenta是由google组织的一个项目组，专门进行基于机器学习的人工智能艺术方面的研究，包括自动作曲、音频生成、图画生成等方面。

主页：https://magenta.tensorflow.org/

Github： https://github.com/tensorflow/magenta

相关介绍：https://mp.weixin.qq.com/s/yCxcV2hF_o_r40QQ3ooctA

项目中包含的模型，在自动作曲方面，有：

Melody_rnn:一个单声部、单音作曲模型

Polyphony_rnn：二声部，一个声部为单音旋律一个声部为和弦组合的生成模型

以及imporv_rnn、drum_rnn、performance_rnn等模型

在其他方面，还有NSynth（Wavenet），sketch rnn（画简笔画），AIduet（人弹一段，AI跟弹一段）等成熟的项目。

## 运行环境：

Magenta主要基于Tensorflow（Python）编写，有部分项目使用Tensorflowjs。

**本文基于Magenta 1.1.1版本（2019.5）

## 安装：

官网给出的安装指南如下：
https://github.com/tensorflow/magenta/blob/master/README.md#installation

Magenta支持Python2.7与Python3.5。对于linux用户，Magenta官方给出了安装脚本，使用方法如下：

```
curl https://raw.githubusercontent.com/tensorflow/magenta/master/magenta/tools/magenta-install.sh > /tmp/magenta-install.sh
bash /tmp/magenta-install.sh
```

上述脚本安装完成后，就可以```source activate magenta```打开Magenta运行环境了。

对于Windows用户（Linux用户也适用），我们也可以轻松实用anaconda等包管理工具安装Magenta。

这里，我给出一个我安装的方法如下：

对于只使用CPU的用户：

```
conda create -n magenta python==3.5
activate magenta （Linux用户为 source activate magenta）
pip install magenta
```

Linux用户在安装Magenta之前需要先安装一些运行库，用以安装rimidi：

```
sudo apt-get install build-essential libasound2-dev libjack-dev
```

如果需要用到GPU，可以直接将上文中```pip install magenta```替换成```pip install magenta-gpu```。
```magenta```和```magenta-gpu```的区别仅在后者多了```tensorflow-gpu```库。

当然，如果需要运行```tensorflow-gpu```，需要安装NVIDIA显卡驱动和CUDA 以及Cudnn，请读者自行搜索方法，这里就不再赘述了。

## 训练/生成：

Magenta提供了命令行（Linux\Mac），在windows系统上，也可以使用bazel环境进行命令行操作。我们可以通过命令行来进行训练、用生成的权重或官方提供的权重进行自动作曲。

我在之后的文章中会分析训练和生成的具体代码。如果你想先试着运行一下Magenta，可以参考如下内容：

Magenta的GitHub内各个Model的文档，以polyphony_rnn为例：

https://github.com/tensorflow/magenta/tree/master/magenta/models/polyphony_rnn

一些国内的博客：

https://www.cnblogs.com/charlotte77/p/5664523.html

https://blog.csdn.net/bestbuild/article/details/51927007
