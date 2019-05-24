
# Melody RNN总览

本文主要介绍Magenta RNN的模型架构和通过运行Python文件进行训练、生成的方法，以及修改Python文件的参数。

###### Magenta version:1.1.1

# Melody RNN是什么？

[Melody RNN](https://github.com/tensorflow/magenta/tree/master/magenta/models/melody_rnn)是Magenta的自动作曲模型中最简单也是最基本的一个模型。万丈高楼平地起，Magenta就是从这个模型开始，一点点添加内容与架构，最后写就了如今这些种类繁多的自动作曲模型。

Melody RNN，顾名思义，是使用RNN模型对旋律（melody）进行生成的模型。该模型的输入为一段旋律（MIDI格式），而输出则是这段旋律与模型根据这段旋律继续生成的旋律，格式也为MIDI。

然而，Melody RNN也并不是个简单的模型。总的来说，Melody RNN模型是一个基于[LSTM](https://en.wikipedia.org/wiki/Long_short-term_memory)（Long short-term memory）网络进行时间序列预测的模型。只不过，我们将音乐转换为时间序列，对音高或事件进行编码，以进行预测。在Magenta的GitHub中对此模型的[介绍](https://github.com/tensorflow/magenta/tree/master/magenta/models)为：Applies language modeling to melody generation using an LSTM. 事实上，此模型是参考基本的[NLP](https://en.wikipedia.org/wiki/Natural_language_processing)（
Natural language processing）模型，尤其是其中的人工智能写作模型的。

在Melody RNN中，总体的数据表示方式为Pianoroll式，即将音符序列按一定时间间隔（通常是1/4小节）采样，形成时间序列对于见前文所述。其运用监督式学习进行训练，基于最大似然进行训练，目标函数为Softmax。
训练的目标为：输入一定长度的时间序列，预测（即输出）这个序列的下一个点的值。

在Melody RNN中，又分为四种不同的配置（或模型架构），每个架构都有不同的数据表示与RNN结构。这四种配置依次由简到繁，请读者依次阅读：

1. `basic_rnn`：基准架构，音高使用[one-hot](https://en.wikipedia.org/wiki/One-hot)编码。One-hot码的表示形式为：对于N个类，只有其中的一个位置值为1，其余为0，值为1位置的索引值即为当前编码的类的值。独热码中每个类的含义为MIDI音高，即每个类为一个半音。每个时间点的输入为按最小频率（默认为1/4个四分音符）采样的旋律的独热表示，独热码中值为1位置的索引值就是当前的事件（event）编号。Event的编号有三种：`note-on`：音符的开始，即开始弹下,其值为音符的音高。`note-off`：弹奏结束，即一个音符结束。`no-event`：当前为休止符，或为弹下一个音后的保持。在basic_rnn中，输入和输出被限制在了[48, 84]，即[C3,C5]的音高范围。这样一来，输入维度较少，计算量与内存占用就相对较小。
2. `mono_rnn`：与basic_rnn相同，但音高表示范围为128个音高，即可以生成全部[MIDI音高](https://newt.phys.unsw.edu.au/jw/notes.html)。
3. `lookback_rnn`：RNN（LSTM）很难学习（相对）长期的时序关系，或时序数据长期规律。为了解决这一问题，Magenta引入了lookback_rnn，在basic_rnn的基础上添加了一些自定义的标签。顾名思义，在lookback_rnn中，每一步的输入都包含之前的序列。也就是说，在生成下一个音符时，lookback_rnn不仅输入上一个音符，还输入一个小节前和两个小节前的所有音符，因此叫“Look Back”。同时，lookback_rnn还加了用于指示当前音符在小节中位置、当前旋律是否为重复出现的特征。总体来说，相当于在lookback_rnn中，手动对旋律做了特征工程，提取了旋律反复的信息，以及手动输入了之前序列（而不是令RNN自己学习在hidden state中储存）。对于音乐的旋律，尤其是流行音乐的旋律，出现旋律重复的情况很多，因此这样提取特征可以很直接的提取出该旋律的长期关系信息。
4. `attention_rnn`：此方法在Lookback_rnn的基础上添加了[Attention](https://arxiv.org/abs/1706.03762)机制。

对lookback_rnn与attention_rnn中数据表示的更详细介绍，可以阅读Magenta的[博客](https://magenta.tensorflow.org/2016/07/15/lookback-rnn-attention-rnn/)。关于数据表示的具体表示方法见下一节。

这四种配置在训练完毕后的输入输出都是相同的，可以理解为是一个功能的四种实现方式。同时我们可以发现，这四种模型在复杂度上层层递进，其输出的效果在我主观感受上也是越复杂的模型生成效果越好。

如果你想的话，也可以先简单浏览一下Magenta的GitHub中对Melody RNN的[介绍](https://github.com/tensorflow/magenta/tree/master/magenta/models/melody_rnn#train-your-own)。

# 如何训练与生成？

如果你想先上手试试Melody RNN的效果的话，自己训练一个模型并根据生成自己的音乐是再合适不过的了。通过命令行进行训练、生成或导入Magenta项目组提供的训练好的模型参数进行生成，请查看[这里](https://github.com/tensorflow/magenta/tree/master/magenta/models/melody_rnn#pre-trained)。

但是，如果你想对Melody RNN进行修改或者更深一步的了解，我们需要查看Melody RNN的代码，以及一步步查看它运行的过程。这样的话，我们可以使用运行Python文件的方式进行训练和生成。

在Melody RNN（包括Magenta中很多其他模型）中，模型的训练、生成过程有着以下几个特点：
- 需要先进行模型参数（模型架构类型、网络大小、超参数等）的配置。
- 需要先将`convert_dir_to_note_sequences.py`中得到的NoteSequence数据先转换成模型需要的格式。

## 参数配置

对应第一步，我们需要在Python文件中修改tf.FLAGS的值。我们需要先在[`melody_rnn_config_flags.py`](https://github.com/tensorflow/magenta/blob/master/magenta/models/melody_rnn/melody_rnn_config_flags.py)中填写：

- `config`：模型的配置。共有`basic_rnn`、`mono_rnn`、`lookback_rnn`、`attention_rnn`四种不同的配置，见上文。填写时直接填写进`tf.app.flags.DEFINE_string()`的第二个参数中，数据类型使用`string`。例如：

```
tf.app.flags.DEFINE_string(
    'config',
    'basic_rnn',
    "Which config to use. Must be one of 'basic', 'lookback', or 'attention'. "
    "Mutually exclusive with `--melody_encoder_decoder`.")
```

- `hparams`：模型的超参数。同样为字符串，格式如`batch_size=64,rnn_layer_sizes=[64,64]`。`batch_size`指的是每一个`batch`的大小，`rnn_layer_sizes`指的是RNN的层数及隐层大小（第一个数指的是第一层，第二个数指的是第二层）。

注意，`melody_rnn_config_flags.py`的修改需要在Python运行环境的路径中。比如使用Anaconda的话，就在`XXX\Anaconda3\envs\你的环境名\Lib\site-packages\库名`中修改。

## 数据转换

下一步的数据转换则对应[`melody_rnn_create_dataset.py`](https://github.com/tensorflow/magenta/blob/master/magenta/models/melody_rnn/melody_rnn_create_dataset.py)。这里我们可以把这个文件放置到任意位置并修改后运行即可。在`melody_rnn_create_dataset.py`中，需要填写：

- `input`：输入文件路径。对应`convert_dir_to_note_sequences.py`生成.tfrecord文件的路径，字符串格式。
- `output_dir`：一个文件夹路径。转换后的训练集、验证集会输出到这个路径下，字符串格式。
- `eval_ratio`：验证集占总数据集的比例，float格式。

如果一切填写完毕，在这一步我们只需要运行`melody_rnn_create_dataset.py`，就可以成功转换数据了。

数据转换的详细过程以及数据表示见下一节。

## 训练与验证

训练则对应[`melody_rnn_train.py`](https://github.com/tensorflow/magenta/blob/master/magenta/models/melody_rnn/melody_rnn_train.py)。在这里，我们需要填写：

- `run_dir`：模型参数保存的路径（文件夹），字符串格式。
- `sequence_example_file`：数据转换中`output_dir`下的tfrecord路径，字符串格式。需要手动指定tfrecord，所以我们需要填写生成的训练集`training_melodies.tfrecord`。例如，`output_dir`填写`Dataset\melody_rnn\data-representation-example`，则此处填写`\Dataset\melody_rnn\data-representation-example\training_melodies.tfrecord`。
- `num_training_steps`：训练的迭代数，int格式，直接填写。如果为0，则一直训练直到手动关闭进程。
- `summary_frequency`：进行状态总结（显示当前loss，迭代次数，训练速度等信息）的频率，int格式，直接填写。每`summary_frequency`秒总结一次。
- `num_checkpoints`：模型保存的数量，int格式，直接填写。只保存最近`num_checkpoints`个模型。如果为0，则每个模型都保存。

同时，如果是运行训练程序，需要把`eval`写为`False`，bool格式。

在一切准备完毕后，运行`melody_rnn_train.py`，我们就可以开始训练了。训练的速度取决于你电脑的配置与性能。在训练过程中，我们也可以使用`tensorboard`进行监视。在命令行中输入`tensorboard --logdir=run_dir`就可以启动`tensorboard`。这里`run_dir`指的是上文中填写的模型参数保存的路径。

验证也使用`melody_rnn_train.py`。不同的是，在运行验证程序时，我们需要把`eval`写为`True`，bool格式；同时，`sequence_example_file`为验证集，即`eval_melodies.tfrecord`的路径。最后，我们可以选择修改`num_eval_examples`，这个参数为使用验证集中数据的数目。不过我一般倾向于填写默认值0，即在全部验证集上验证。

在训练程序运行的同时，运行填写验证参数的`melody_rnn_train.py`，会启动一个新的Python进程。`run_dir`下每有一个新模型文件生成，便读取该模型文件并进行验证，显示验证结果。

然而，同时运行两个进程、两套神经网络会使用两倍显存。因此这一点是Magenta项目的一个缺点。不过，也许谷歌不会在意这一点。

# 生成

Melody RNN的生成需要先输入一段前置序列，模型以输入一段序列，预测后一个点的方式迭代运行（即把输出当做下次的输入），输出音乐。Melody RNN的生成也采用了[Beam Search](https://hackernoon.com/beam-search-a-search-strategy-5d92fb7817f)（默认不开启Beam Search）。生成程序的文件为[`melody_rnn_generate.py`](https://github.com/tensorflow/magenta/blob/master/magenta/models/melody_rnn/melody_rnn_generate.py)。

需要填写的参数如下：
- `run_dir`：模型保存路径，与训练时的`run_dir`相同，字符串格式。
- `output_dir`：输出路径，字符串格式。
- `primer_midi`：前置MIDI地址，字符串格式。MIDI可以使用MuseScore、西贝柳斯等软件进行编辑。

可选参数（可以不填或使用默认值）：
- `checkpoint_file`：直接指定模型checkpoint文件路径，字符串格式。
- `bundle_file`：bundle文件路径，字符串格式。
- `num_outputs`：生成文件的数量，int格式。默认为生成10条。
- `num_steps`：生成音乐的长度，int格式。默认生成128长度（128/4个点每四分音符/4个四分音符每小节=默认8小节生成长度）。
- `temperature`：生成随机的程度。越大则越随机，越小越不随机。浮点格式。
- `beam_size`：Beam Search中的Beam大小，int格式。
- `branch_factor`：Beam Search中树枝的数量，int格式。
