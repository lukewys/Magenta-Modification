## 前言：

最近在魔改Magenta，所以会涉及到阅读、修改Magenta的源代码。我个人认为Magenta是一个很好的项目，虽然代码可能写的有点乱，但模型的架构写的都很好，从主观上来看（听），做出的自动作曲模型效果也很优异。Magenta可以为做深度学习自动作曲的研究者提供一个参考，同时我在这里也希望给有兴趣的朋友提供一些经验。
 
## Magenta：

Magenta是由google组织的一个项目组，专门进行基于机器学习的人工智能艺术方面的研究，包括自动作曲、音频生成、图画生成等方面。

主页：https://magenta.tensorflow.org/

Github： https://github.com/tensorflow/magenta

相关介绍：https://mp.weixin.qq.com/s/yCxcV2hF_o_r40QQ3ooctA
 
项目中包含的模型，在自动作曲方面，有：

Melody_rnn:一个单声部、单音作曲模型

Polyphony_rnn：二声部，一个声部为单音旋律一个声部为和弦组合的生成模型

以及imporv_rnn、drum_rnn、performance_rnn等模型

还有NSynth（Wavenet），sketch rnn（画简笔画），AIduet（跟弹）等成熟的项目。
 
## 运行环境：

Magenta主要基于Tensorflow（Python）编写，最近的项目有使用Tehsorflowjs。
 
## 安装：

https://github.com/tensorflow/magenta/blob/master/README.md#installation

总的来说就是先安装Tensorflow环境，然后直接pip install magenta/magenta-gpu就妥了。

Note：在早先没有Gpu版本时，我在Tensorflow-gpu下运行也可以跑，应该是magenta-gpu版本新加了cuDNN的支持，把RNN方法都改写成了cuDNN的。
 
## 训练/生成：

Magenta提供了命令行（Linux\Mac），在windows系统上，也可以使用bazel环境进行命令行操作。我们可以通过命令行来进行训练、用生成的权重或官方提供的权重进行自动作曲。

命令行操作方式见Github内各个Model的文档，如：

https://github.com/tensorflow/magenta/tree/master/magenta/models/polyphony_rnn

墙内：

https://www.cnblogs.com/charlotte77/p/5664523.html

https://blog.csdn.net/bestbuild/article/details/51927007

