深入解析 ES6：Generators
===
欢迎回到深入浅出ES6专栏，望你在ES6探索之旅中收获知识与快乐！程序员们在工作之余应当补充些额外的知识，现在我们继续深入浅出生成器，我已经为你们准备好非常棒的讨论话题。

在之前的文章《深入浅出ES6（三）：生成器 Generators》中，我为大家介绍了ES6中引入的新特性——生成器（Generators），我认为它是ES6中最具魔力的特性，很可能是异步编程下一步的发展方向。后来我这样写道：

> 生成器还有更多未提及的特性，例如：.throw()和.return()方法、可选参数.next()、yield*表达式语法。由于行文过长，估计观众老爷们已然疲乏，我们应该学习一下生成器，暂时yield在这里，剩下的干货择机为大家献上。

此时此刻，我们再续前缘。

阅读本文前，你最好先阅读一下文章的第1部分，文章比较长，你可能看得云里雾里，但那儿有一只会说话的猫陪伴你，非常有趣！

快速回顾
---
在第三篇文章中，我们着重讲解了生成器的基本行为。你可能对此感到陌生，但是并不难理解。生成器函数与普通函数有很多相似之处，它们之间最大的不同是，普通函数一次执行完毕，而生成器函数体每次执行一部分，每当执行到一个yield表达式的时候就会暂停。

尽管在那篇文章中我们进行过详细解释，但我们始终未把所有特性结合起来给大家讲解示例。现在就让我们出发吧！

```
    function* somewords() {
      yield "hello";
      yield "world";
    }
    for (var word of somewords()) {
      alert(word);
    }
```

这段脚本简单易懂，但是如果你把代码中不同的比特位当做戏剧中的任务，你会发现它变得如此与众不同。穿上新衣的代码看起来是这样的：

> （译者注：下面这是原作者创作的一个剧本，他将ES6中的各种函数和语法拟人化，以讲解生成器（Generator）的实现原理）

```
场景 - 另一个世界的计算机，白天
SCENE - INTERIOR COMPUTER, DAY

for loop女士独自站在舞台上，戴着一顶安全帽，手里拿着一个笔记板，上面记载着所有的事情。
FOR LOOP stands alone onstage, wearing a hard hat and carrying a clipboard, all business.

                          FOR LOOP
                         (calling)
                        someWords()!

generator出现：这是一位高大的、有着一丝不苟绅士外表的黄铜机器人。
The GENERATOR appears: a tall, brass, clockwork gentleman.

它看起来足够友善，但给人的感觉仍然是冷冰冰的金属。
It looks friendly enough, but it's still as a statue.

                          FOR LOOP
               (clapping her hands smartly)
           All right! Let's get some stuff done.
                     (to the generator)
                          .next()!

generator动了起来，就像突然拥有了生命。
The GENERATOR springs to life.

                         GENERATOR
               {value: "hello", done: false}

然而猝不及防的，它以一个滑稽的姿势停止了动作。
It freezes in a goofy pose.

                          FOR LOOP
                           alert!

alert小子飞快冲进舞台，眼睛大睁，上气不接下气。我们感觉的到他一向如此。
Enter ALERT at a sprint, wide-eyed and breathless. We sense he's always like that.

                          FOR LOOP
                   Tell the user "hello".

alert小子转身冲下舞台。
ALERT turns around and sprints offstage.

                           ALERT
                   (offstage, screaming)
                      Stop everything!
          The web page at hacks.mozilla.org says,
                          "hello"!

停留了几秒钟后，alert小子跑回舞台，穿过所有人滑停在for loop女士身边。
A few seconds' pause, then ALERT races back on, crossing all the way over to FOR LOOP and skidding to a halt.

                           ALERT
                     The user says OK.

                          FOR LOOP
               (clapping her hands smartly)
           All right! Let's get some stuff done.
              (turning back to the GENERATOR)
                          .next()!

generator又一次焕发生机。
The GENERATOR again springs to life.

                         GENERATOR
               {value: "world", done: false}

它换了个姿势又一次冻结。
It freezes in a different goofy pose.

                          FOR LOOP
                           alert!

                           ALERT
                     (already running)
                           On it!
                   (offstage, screaming)
                      Stop everything!
          The web page at hacks.mozilla.org says,
                          "world"!

又一次暂停，然后alert突然跋涉回到舞台，垂头丧气的。
Again, a pause, then ALERT trudges back onstage, suddenly crestfallen.

                           ALERT
               The user says OK again, but...
             but please prevent this page from
               creating additional dialogues.

他噘着嘴离开了。
He exits, pouting.

                          FOR LOOP
               (clapping her hands smartly)
           All right! Let's get some stuff done.
              (turning back to the GENERATOR)
                          .next()!

generator第三次焕发生机。
The GENERATOR springs to life a third time.

                         GENERATOR
                       (with dignity)
               {value: undefined, done: true}

它的头低下了，光芒从它的眼里消失。它不再移动。
Its head comes to rest on its chest and the lights go out of its eyes. It will never move again.

                          FOR LOOP
                  Time for my lunch break.

她离开了。
She exits.

一会儿，garbage collector（垃圾收集器）老头进入，捡起了奄奄一息的generator，将它带下舞台。
(After a while, the GARBAGE COLLECTOR enters, picks up the lifeless GENERATOR, and carries it offstage.)
```

好吧，这一出戏不太像哈姆雷特，但你应该可以想象得出来。

正如你在戏剧中看到的，当生成器对象第一次出现时，它立即暂停了。每当调用它的.next()方法，它都会苏醒并向前执行一部分。

所有动作都是单线程同步的。请注意，无论何时永远只有一个真正活动的角色，角色们不会互相打断，亦不会互相讨论，他们轮流讲话，只要他们的话没有说完都可以继续说下去。（就像莎士比亚一样！）

每当for-of循环遍历生成器时，这出戏的某个版本就展开了。这些.next()方法调用序列永远不会在你的代码的任何角落出现，在剧本里我把它们都放在舞台上了，但是对于你和你的程序而言，所有这一切都应该在幕后完成，因为生成器和for-of循环就是被设计成通过迭代器接口联结工作的。

所以，总结一下到目前为止所有的一切：

- 生成器对象是可以产生值的优雅的黄铜机器人。

- 每个生成器函数体构成的单一代码块就是一个机器人。
