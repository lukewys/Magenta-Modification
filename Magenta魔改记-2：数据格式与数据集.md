# Magenta魔改记-2：数据格式与数据集

## 数据格式：MusicXML与MIDI

上一节我们主要提到了两种数据格式：MIDI（.mid/.midi）和MusicXML（.xml/.mxl）。其实他们二者，包括提到的ABC notation，都是在计算机中用于记谱的文件格式。
在这其中，ABC notation是以文本形式储存的，更接近于简谱。并且，ABC notation也应用的不广泛，更不用说和打谱软件对接了，因此在此就不详细展开了，感兴趣的可以看<https://en.wikipedia.org/wiki/ABC_notation> 。
而对于MIDI与MusicXML二者，他们表达的内容很相似，但差别在文件储存的格式上。

## MIDI

MIDI，全称为Musical Instrument Digital Interface。而现如今对于MIDI这个名称的含义有三重意思：

1. 连接音乐设备的物理接口（Musical Instrument Digital Interface）
2. 存储和回放音乐的文件格式（Standard MIDI File）
3. 通用MIDI（General MIDI）

对于MIDI的历史，最早可以追溯到八世纪水力驱动的自动乐器，到后来的八音盒、自动钢琴等。  
而我们重点关注的是第二个含义，标准MIDI文件（SMF）。  
关于标准MIDI文件的格式，https://www.csie.ntu.edu.tw/~r92092/ref/midi/ 有着很详尽的介绍。  
MIDI文件是一种二进制文件，其中绝大部分由一个个事件（Events）组成，每个事件都有其相对应的时间戳。这其中最普遍的时间就是音符（Note）。音符又可分为按下（Note on）和抬起（Note off）。这二者的参数都是（通道，音高，力度），其中通道指的是？，而音高就是MIDI中以八位二进制数相对应的音高（中央C对应60，各音高按十二平均律依次加减）。
其他的常见事件包括time_signature（节拍标记，例如4/4拍）、key_signature（调式标记，例如B小调）、以及control_changes（控制器输入变化，如踩踏板）。  
因此，我们在上一节中，可以看到Magenta利用pretty_midi库将MIDI文件转化成了一个一个事件列表，其中音符事件又合并成了Note（start,end,key,instrument,programe），其中program又指的是这个音符应当播放的音色。
然而，我们可以发现，在MIDI文件中，音符的start和end的绝对时间非常不整，通常都是很多位小数。这是由于：  

1. 在相对节拍到绝对时间戳的转换时，很容易会有除不尽的情况。因此start的时间戳很容易就会是很多位的小数。
2. 在真实演奏中，一个音和下一个音很少是完全连接的，人们会或故意或无意的对两个音进行间隔，例如在钢琴中弹奏一个音之后若要再弹奏下一个距离远的音，需要切换手的位置，因此两个音不可避免的会有间隔。MIDI文件是可以直接根据音符的start和end进行播放的，因此很多MIDI文件在导出时，为了让每个音符的结尾听起来更自然，都不会写满此音符在谱子中的时值，而是缩短很少一段时间，以模拟真实演奏中情况。

更为复杂的是，MIDI文件中还会有单通道、多通道同步、多通道异步（0、1、2）三种格式，以及更多不规范格式（但可以播放），这更加大了提取MIDI信息的难度。这就引出了我们下面要介绍的MusicXML格式。

## MusicXML

由于MIDI文件的复杂，不规范以及面向播放的特性，其并不能完全满足音乐软件对谱子显示及排版的需求。因此，MusicXML应运而生。
MusicXML是一个开放自由，易于分发的西洋乐记谱格式，其在万维网联盟（W3C）管理下。MusicXML文件基于标准XML技术，因此本质上是一种文本文件，有别于标准MIDI文件为二进制文件。MusicXML的优点主要在于其对显示格式有着精确的定义，因此可以做到对于同一个文件在不同的环境下打开都有着同样的谱面显示内容。
MusicXML中的音乐语义主要有elements表达，也就是其中的XML标签，并以标签的嵌套关系表达音乐语义的元素包含关系。

### MusicXML文件格式：

MusicXML文件分为两种类型：score-partwise与score-timewise，其中较为常见的是score-partwise。  
score-partwise大结构如下：  

```
谱子信息，XML文件信息  
各声部信息
声部1全曲：
    小节1：
            属性
        音符1
        音符2
        ……
    小节2：
        音符1
        音符2
        ……
    ……
声部2全曲：
    小节：
            属性
        音符1
        音符2
        ……
    小节2：
        音符1
        音符2
        ……
    ……
```

而score-timewise大体结构则为：

```
谱子信息，XML文件信息
各声部信息
小节1：
    声部1：
        属性
        音符1
        音符2
        ……
    声部2：
        属性
        音符1
        音符2
        ……
小节2：
    声部1：
        属性
        音符1
        音符2
        ……
    声部2：
        属性
        音符1
        音符2
        ……
```

声部信息（part-list）通常包含这个声部的乐器的ID、名字，MIDI乐器编号（MIDI Program）、音量、MIDI通道等。
而小节中的属性元素，则指的是乐谱中这一小节中的一些特殊元素，如谱号、调号、记号、文字等。它们隶属于这一小节。  
一个属性（attributes）通常包含以下信息：

* Divisions：最小时值单位
* Key：调号
* Time：拍号
* Clef：谱号

对于一个音符（Note），通常包含以下信息：

* Step：音名
* Octave：八度位置
* Duration：相对长度
* Type：音符类型

MusicXML的小节记号为Measure标签。音符嵌套在Measure标签中，代表其与这一小节的从属关系。每个音符以及小节都有其标号。
我们可以看到，音符的时长（duration）是整数。一个音符的时值是由它的时长属性与声部中的divisions属性共同定义的。具体来说，divisions代表了音符时值的最小单位，单位则是每四分音符。因此，对于一个duration=1，divisions=2的音符来说，它是一个八分音符。基于上述特点，对于MusicXML文件的音符时值提取就显得相比MIDI容易许多。

除此之外，MusicXML文件还有许多定义布局、像素、位置、记号等显示信息，如Measure的width属性，Note的default-x, default-y属性，stem属性等等。

MusicXML文件的后缀名为.xml。但一个用文本格式存储的MusicXML文件通常会占用很多空间，因此在MusicXML标准中有对MusicXML文件进行压缩的格式与标准。压缩后的文件后缀名为.mxl。压缩规范见MusicXML用户手册。

与XML语言一样，MusicXML也有其文档类型定义，即dtd文件，见：  
https://github.com/w3c/musicxml/  
我们可以通过IDE查看MusicXML文件，并点击其中的标签跳转到它们的标准定义。  
下面提供几个MusicXML的详情介绍及教程：  
中文：  
https://wenku.baidu.com/view/49b3cab4960590c69ec376a2.html  
English：  
http://usermanuals.musicxml.com/MusicXML/MusicXML.htm#Tutorial.htm  
https://en.wikipedia.org/wiki/MusicXML

在Python中可以使用`music21`库处理MusicXML文件与MIDI文件，或使用`pretty_midi`库处理MIDI文件。

## 总结：MIDI与MusicXML

总的来说，MIDI历史悠久，以播放为目的，可以搜集到的MIDI文件也相对较多。但是各个MIDI文件数据格式不统一，数据杂乱，并不是数字记谱的好格式。

MusicXML与MIDI相比，有着无可比拟的优点。MusicXML有着明确而唯一的记谱排版格式，例如小节线的编排、临时升降号、拍号与谱号等等。但是，因为MusicXML格式相对复杂，也出现的较晚，网络上并不好搜到原生的MusicXML谱（尽管你可以将MIDI文件直接转成MuiscXML，但是这毫无意义）。

我认为，对于计算机音乐的符号化音乐分析，我们还是需要尽量使用MusicXML格式，尽管目前还有很多困难。

## 数据集：

因为MIDI文件形式历史较长，应用广泛，因此现行的大多数音乐的记谱传播格式都是MIDI文件，但有一些互联网众包整理的MIDI文件数据集虽然数据量大，但集、风格不一，格式杂乱。对于MusicXML文件格式，也有一些经过挑选与整理的，特定风格与作曲家的数据集。同时，还有一些针对表情、和弦、和谐音程、主旋律等标注的数据集。
对于具体可用的数据集，请参阅：

```
Briot, Jean-Pierre, Gaëtan Hadjeres, and François Pachet. "Deep learning techniques for music generation-a survey." arXiv preprint arXiv:1709.01620 (2017).
```

其中给出了一些不错的数据集。

**更新：推荐一个知乎@[谢圜](https://www.zhihu.com/people/huan--xie)同学整理的数据集介绍，相当全面也相当不错：[https://ldzhangyx.github.io/2019/02/25/music-toolkits/](https://ldzhangyx.github.io/2019/02/25/music-toolkits/)**

 

参考：

https://zhuanlan.zhihu.com/p/27523799

https://en.wikipedia.org/wiki/MIDI

https://www.csie.ntu.edu.tw/~r92092/ref/midi/

https://en.wikipedia.org/wiki/MusicXML

https://www.musicxml.com/tutorial/

https://en.wikipedia.org/wiki/ABC_notation

https://www.midi.org/articles-old/the-history-of-midi

https://wenku.baidu.com/view/49b3cab4960590c69ec376a2.html

Briot, Jean-Pierre, Gaëtan Hadjeres, and François Pachet. "Deep learning techniques for music generation-a survey." arXiv preprint arXiv:1709.01620 (2017).

http://web.mit.edu/music21/

https://www.musicxml.com/tutorial/compressed-mxl-files/compressed-file-format/

https://www.musicxml.com/

https://wenku.baidu.com/view/49b3cab4960590c69ec376a2.html

http://usermanuals.musicxml.com/MusicXML/MusicXML.htm#TutMusicXML2-1.htm%3FTocPath%3DMusicXML%25203.0%2520Tutorial%7C_____1

https://w3c.github.io/musicxml/

https://github.com/w3c/musicxml/

https://en.wikipedia.org/wiki/MusicXML

https://www.midi.org/articles-old/the-history-of-midi

https://steinberg.help/cubase_pro_artist/v9/en/cubase_nuendo/topics/midi_editor_score_editor/working_with_music_xml/score_editor_musicxml_advantages_of_c.html
