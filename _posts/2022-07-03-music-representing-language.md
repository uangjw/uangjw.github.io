---
layout: post
title: 课程设计||单音轨midi音乐表示语言
categories: [Curriculum Design]
description: 编译原理课程设计“一种单音轨midi音乐的表示语言”的成果展示讲稿（整理版，删去了介绍基本乐理的内容）
keywords: principles of compiler design

---

# Music Representing Language

下面请让我向大家介绍一下我的编译原理实验课程设计：Music representing language,一种单音轨midi音乐表示语言。

## 简介

首先这个语言是基于midi来表示音乐的，midi是一种基于事件的音乐格式，经过编译后，本语言的源文件能够生成含有单个音轨的midi文件。下面这幅图展示了本语言表示的经典儿歌《小兔子乖乖》，在这个music上面的部分都是乐曲的全局配置，实际上表示小兔子乖乖的内容就是下面这8行，还是比较简单的。

<img src="/images/post-mrl/fig1.png" style="zoom:67%;" />

本语言一种可能的应用就是快速地起草一段旋律，并生成midi音乐来试听；另一方面，全局的配置一改动，生成的音乐虽然还是那首歌，但风格可能变化了，因此我们也可利用本语言来根据情境生成不同风格的同一首歌。

## 基本设计

以小兔子乖乖为例，我们可以在其五线谱看到，这首歌BPM是120，节奏是四四拍，一个小节四个四拍，整首歌的音高在中央C附近，可以确定所有音的音阶都是3左右，然后具体组成，曲子里面有单音，也有和弦，单音就是这一时刻只有一个音发声，和弦就是很多个音在同一时刻发声，最后我们看整首歌被这种小节线划分成了8个小节。

然后我们把这个五线谱和我们的语言放在一起，我们看到在这个全局设置一块，BPM设置成120，节奏，tempo，设置成4，也就是我刚刚说的，只看这里上面那个数字，一个小节4拍，然后这个程序里面，用一个等号表示一个小节，就是这一行，和这一小节，是对应的，程序里面的小竖杠是划分一个拍的时值的，这里就是划分出了一二三四四拍，然后一个字母带上它的后缀，就是一个单音，如果几个单音被括号阔在了一起，那它们就是一个和弦。

<img src="/images/post-mrl/fig2.png" style="zoom: 67%;" />

## 语言定义

本语言完整的EBNF如下图：

<img src="/images/post-mrl/fig3.png" style="zoom: 80%;" />

<img src="/images/post-mrl/fig4.png" style="zoom:80%;" />

下面具体介绍。

首先行尾注释不必多说。标识符，本语言的标识符可以表示整数对象，segment段落还有music音乐，这三者的具体定义我们后面介绍。音符，我们核心的一个东西，由音名和后缀两个部分组成，音名前面说很多了，后缀，+就是升一个八度，-降一个八度，#升一个半音，这边列出了一些例子：

<img src="/images/post-mrl/fig5.png" style="zoom:67%;" />

为了形成更丰富的节奏，我还设计了两种特殊音符，一个点，用来表示延续上一个单音或者和弦，一个百分号，用来表示休止符：

<img src="/images/post-mrl/fig6.png" style="zoom:67%;" />

和弦，用括号括起来一系列音符，就是一个和弦：

<img src="/images/post-mrl/fig7.png" style="zoom:67%;" />

整数，本语言实现了简单的整数声明、赋值，还有四则运算：

<img src="/images/post-mrl/fig8.png" style="zoom:67%;" />

为了方便地表达乐曲，本语言有两种段结构，类似我们编程语言的函数或者过程，一个音乐段，和main函数很类似，一个程序有且只有一个，是生成音乐的开始之处。在music段里面，可以有section，小节，真正的乐曲内容，用等号标识；也可以有别的语句，statement，就是整数声明赋值if-else语句之类所有和音乐内容不直接相关的语句（整数声明赋值，if-else，引用segment，还有一个set_octave，这个东西相当于一个预定义函数，调用它是用来修改全局的音阶设置的）。Music有一个标识符，这个标识符相当于是你这首歌的名字，所以生成的midi文件也是以这个标识符来命名。segment段落段，本质上就是把乐曲的一部分单独拎出来了，一个程序里可以有0到多个segment。Segment设计出来就是为了单独表示一小段音乐，它内容只能是section，不能有其他语句，然后music段可以通过segment的标识符来调用segment。

<img src="/images/post-mrl/fig9.png" style="zoom:67%;" />

下面介绍全局设置方面的内容。BPM，一分钟拍子数，正整数。INSTRUMENT，乐器序号，要对照GM的标准来设置，GM是一个音色库，这个表就是GM标准音色表，一种音色对应一个整数序号。CUR_OCTAVE，全曲的音阶，前面介绍音符的时候我们已经看到了，C3这样子一个音名‘C’一个音阶‘3’才能指定一个特定的音符；那如果这里单纯只写个C，在生成的时候看到cur_octave是3，那么就知道这个C是C3的意思。然后前面说了，可能某一段音乐整段都很高音，要是保持cur_octave的设置，那就要写好多个+号了，那这时候调用一下set_octave，把当前音阶调高点，那就省好多个加号了；当然也可以第一次演奏某一段的时候，低音一点，然后set一下octave，下次演奏就变高了。VELOCITY，强度，其实就是音量大小，这里我做的实现比较简单，就给了0和1两级选，1是标准强度，0比较小声。TEMPO，前面也说很多了，节奏，指定了一个小节必须有几拍，如果tempo写个4结果小节里小竖线只划分了3个拍子，是要报错的。

<img src="/images/post-mrl/fig10.png" style="zoom:67%;" />

## 语言实现

本语言编译生成的结果就是midi文件。词法分析是用jflex做的，javacup做语法分析；语义动作方面，我用java的sound.midi库写了个midi writer来完成写midi文件的工作，我们来讲讲这个。

### 语义动作实现

这里我直接截了javadoc生成网页的图，我写了个write_midi，里面一个midiwriter，写midi文件的，一个note，把前面提到的几类音符和它们的音名、音阶等信息封装到一起方便操作：

<img src="/images/post-mrl/fig11.png" style="zoom: 67%;" />

这里展示了midiwriter的方法，其实就是向音轨里写音符，写和弦，写一整段音乐也就是segment之类的，最后再用writeMidiFile把音轨写道文件里，结束生成：

<img src="/images/post-mrl/fig12.png" style="zoom: 67%;" />

### 翻译模式

下面就是最最核心的问题，翻译模式。这里先给出全局配置的翻译模式，可以看到每一项配置规约了之后都把配置值记录到parser的一个成员变量里，然后所有配置归约成config statement，这时就初始化MidiWriter，这部分翻译模式还是挺直接的：

<img src="/images/post-mrl/fig13.png" style="zoom: 67%;" />

整数表达式与布尔表达式，这个就更直接了，整数声明与赋值，也就是利用一个hashmap的结构来存一个整数对象的标识符和它的值（布尔表达式的翻译模式略去）：

<img src="/images/post-mrl/fig14.png" style="zoom:67%;" />

接下来要介绍的翻译模式就稍微复杂一点，为了更好地说明，这里我把cup文件里面的parser code展示一下，可以看到这些parser 的成员变量包括一些全局配置，两个用于状态标识的布尔变量condition和buffering，存segment段和整数对象的标识符映射的hashmap，用来保存segment内容的结构，用来缓存小节section内容的结构，还有一些和延音有关的东西。

<img src="/images/post-mrl/fig15.png" style="zoom:67%;" />

其中和section内容缓存有关的几个东西比较重要，buf_note是用来暂存一个音符的，bead_notes用来暂存某一拍上所有的音符，chord_notes用来暂存一个和弦上所有的音符，beat chords用来暂存某一拍上所有的和弦。为什么要搞得这么复杂，本质是因为一拍会由很多个单音、和弦还有延音的点和休止符共同构成。我的基本思路是遇到音符，归约成音符，遇到括号，归约成和弦，遇到小竖线，归约成一个拍的同时，把这个拍里所有音符遍历一遍，把延音和休止符的时间都算进去，求出每一个音或者和弦持续的时间，然后写到音轨里。

接下来我们继续看翻译模式，这里是音符相关的翻译模式，可以看到就是音名和后缀组成，把后缀和全局设置的音阶结合在一起把音符的音阶算出来，是否升音判断出来，就把这个音符存到buf_note里面：

<img src="/images/post-mrl/fig16.png" style="zoom:67%;" />

和弦是这样做的，每次把一个note归约到chord里面的时候，都是把buf_note的音符拿出来放到chord_note里面，也就是说和弦是音符的序列，这个序列存在chord_note：

<img src="/images/post-mrl/fig17.png" style="zoom:67%;" />

Sound这个非终结符是用来统一标识所有发声内容的，同时各种东西，note，chord，延音的点还有休止符在归约到sound的时候，我们都在beat_note里面填对应的信息，这个beat_note就是这一拍上面所有发声的内容的序列：

<img src="/images/post-mrl/fig18.png" style="zoom:67%;" />

Section，小节，我们是一个beat一个beat地归约到section的，就是一拍一拍地归约，然后给这个sec_part一个整数的属性，用来数规约了多少个beat，sec part归约到section的时候看一下beat数和全局的节奏tempo设置是不是对的上，对不上就报错。然后在一拍一拍的归约过程里，我们也一拍一拍地对这些音乐内容处理，这时候要看一下两个状态变量，condition是假的话就什么都不做，condition是真，buffering是假的话就直接把内容写道mdii里，buffering是真，表示现在是在归约一个segment的内容，这些内容要存储到segment相关的结构里面：

<img src="/images/post-mrl/fig19.png" style="zoom:67%;" />

If-else语句就是来控制这个condition的，在归约这个if语句的头的时候，看一下括号里面的布尔表达式，如果为假，那把condition置为假，这样接下来归约if下面的所有小节时就什么都不会做了；然后else的部分，因为有else就必定有if，else只要把condition反转一下，就能实现if部分演奏，else部分不演奏，或者if部分不演奏，else部分演奏了。最后出了if-else语句，condition必须保持true，那就在完整归约出if-else语句的时候把conditon设置为true就好了。大家可能发现了，这样子用一个布尔变量来实现选择其实是偷懒了，这样的if-else结构是不能嵌套的：

<img src="/images/post-mrl/fig20.png" style="zoom:67%;" />

接下来是statement的翻译模式，说白了就是除了小节section以外所有的东西，都直接归约到statement这个非终结符里，然后比较特殊的两个一个segment call调用segment，看一下这个segment的标识符是不是有效，有效就去吧对应的那一整段演奏写到midi里面。然后一个set octave，规约的时候把全局的音阶改成set里面的参数就行了：

<img src="/images/post-mrl/fig21.png" style="zoom:67%;" />

最后，music段，segment段还有整个文件的归约，这个整个文件mrl file就是开始符号，归约到它的时候就调用writemidifile，把音轨的内容写道midi文件里面，前面我们各种写mdii都是在写一个音轨，到了这一步才是真正写文件，所以说我们生成的是一个单音轨的音乐。Segment，前面我们说除了condition以外还有一个buffering，buffering就是用来表示“现在正在归约segment里面的内容”的意思，把segment的开头规约出来的时候，就把buffering设成true，segment归约完了，就把它设成false。用布尔变量来控制，所以也是嵌套不了的。可以看到这里segment part只能是一个section一个section归约。Music，可以有statement和section两种内容，然后在归约出它的开头的时候把这个标识符存起来，之后归约到开始符号的时候就拿来作为文件的名字。到这里语言的翻译模式设计就基本介绍完毕了。

<img src="/images/post-mrl/fig22.png" style="zoom:67%;" />

## 总结与体会

总结一下，本语言是一种比较简单的单音轨midi音乐表示语言，这个语言设计相对比较简单，然后所涉及的乐理也比较简单。里面一些比较核心的要素，一个全局设置，乐曲速度，演奏的音色，节奏，音高之类的，一个music段和segment段，这个主要是结构组织的问题，然后是小节、音符还有和弦，这些事乐曲真正的组成成分，最后就是一些简单的控制结构，if-else，整数等等。

最后是我的一些体会，遇到的困难，最开始当然是对语言设计很没概念，比较盲目地参考了一些别人做的东西，后来就格局打开，自己想到什么写什么，然后在老师的帮助下把语言的设计精简了很多，才有了现在这个看起来还行的版本；然后在实现的时候，网上对midi标准的解析比较少，我一开始其实是一边实验一边去猜midi的各个字节是什么意思，也挺费劲的；最后是感觉自己的能力还是比较有限，前面也提到很多嵌套的结构没有做，然后midi的延音事件我也不是特别懂应该怎么弄，目前这个版本其实速度慢下来延音就和休止没什么区别，所以还是有一点遗憾的。

整个设计和实现的过程还让我对“语言是一种解决问题的方法”这个点有了更深的感受，我们理论课的第一节课就提到了这个事情，当时还是觉得很模糊不太理解，现在我可以说，本语言可以成为你手边没有乐器但又想写写旋律的一个不错的帮手。

<img src="/images/post-mrl/fig23.png" style="zoom:67%;" />

最后，我还得承认，最开始真的是不太情愿做这个实验，但除了自己摸到方向之后慢慢有了兴趣之外，来自一些同学还有老师的鼓励和肯定也让我越来越有动力，非常感激，然后在语言设计的阶段，老师跟我说了一句话，叫做“要去做有趣的事情！”让我感到非常触动，在这里也想把这句话分享给大家。

我的分享就到这里，非常感谢大家的耐心聆听！谢谢！