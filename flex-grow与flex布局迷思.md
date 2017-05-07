# flex-grow与flex布局迷思
#flex

最近再移动端写react与vue的时候重新回顾了一下flex布局，忽然发现对flex布局理解的并不透彻（因为工作中本身就很少用到_(ㄒoㄒ)_~~），起因源于类似这样一段代码

[JS Bin - Collaborative JavaScript Debugging](http://jsbin.com/xayacaf/edit?html,css,js,console,output)

大概观察了一下好像符合我的预期，`flex-grow` 为2的li是 `flex-grow` 为1的li宽度的两倍，但是再将li中填上内容的时候，却发生了好像不是那么正常的情况

[JS Bin - Collaborative JavaScript Debugging](http://jsbin.com/bilutiz/edit?html,css,js,console,output)

很明显console的结果并不如预期的那样，于是我翻了一些关于flex的资料，让我们来看下css规范中对于flex的描述

[CSS Flexible Box Layout Module Level 1](https://drafts.csswg.org/css-flexbox/#flexibility)

很明显，第一段话就大概解释了上面两段代码的问题，大体意思是弹性布局的定义是让其中的flex-items具有弹性，会改变他们的宽高去占满容器内的**可用空间**。而决定一个flex-item的宽度是由三个属性决定的，分别是 `flex-grow flex-shrink flex-basis` 他们的初始值分别是0 1 auto。

首先我们需要理解规范中提到的**可用空间**，到底什么叫可用空间，拿宽度举例，翻译成人话其实就是，**flex容器的宽度减去所有flex-item的flex-basis宽度**。

比如1000px的ul中有三个li，每个li设置了flex-basis为50px，那么可用空间就还剩下了850px。此时如果flex-grow的比例设置为2 1 1，那么剩余的850px会平分为4份，再按2 1 1的比例还给3个li，所以第一个li的宽度为50 + 850 / 4 * 2为475px，其余两个li则是50 + 850 / 4 * 1为262.5px，注意这种情况指的是有剩余空间的情况，如果**剩余空间不够则flex-shrink也会参与计算**，在此不再赘述。

回过头来看上面的两个例子，两个例子中都没有写flex-basis的值，所以初始都是auto，auto是什么意思，再来看下规范，其写到**auto的情况下，flex-basis默认是采用主要尺寸的值**（也就是盒子的height与width），如果主要尺寸依然是auto，则采用 `flex-basis: content` 也就是根据内容的宽度来计算，那么上面两个例子的问题就很好解答了。

例1中由于没有内容，则flex-basis就是0，所以可用空间就是ul的宽度本身，那么三个li的宽度比例自然是2 1 1，而例2中由于需要去计算其自身内容的宽度，所以宽度比例才并不是我们预想的一样。

- - - -

了解完上面的知识后还有一个大坑我们需要迈过，在实际开发中很多情况下我们会使用 `flex` 属性缩写，那么我们看下面的代码

[JS Bin - Collaborative JavaScript Debugging](http://jsbin.com/qilepiv/edit?html,css,js,console,output)

三个li中只分别了写了 `flex: 2; flex: 1; flex: 1;` 并且li本身中都有内容，按照我们上面了解到的知识，按说三个li的宽度比例不应该是完美的2 1 1呀，这是为什么？还是那句话，我们再看眼规范

规范中的flex的初始值确实是 0 1 auto三个值，但是请注意，当我们省略 `flex-grow flex-shrink与flex-basis` 时，这三个参数的默认值则是1 1 0
也就是 `flex: 2` 这种写法其实是等价于 `flex-grow:2; flex-shrink: 1; flex-basis: 0;` 的，所以可用空间依然是整个ul的宽度，那么自然宽度的比例就是2 1 1了。

再规范中明确的建议了我们多去使用flex这个属性，因为这更符合我们多数常见的情况。比如三列等分的布局，只要可用空间足够内容的分配，我们就不用去过于在意其中内容所占用的空间。







