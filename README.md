# story-grammar in Chinese
以下只是原README的简单翻译（第一次翻译英文文档，如有不准，还请见谅。欢迎提出issue）：

一个可以用来生成故事的语法, 灵感来自 [NaNoGenMo 2016](https://github.com/NaNoGenMo/2016/).

## 开始
你可以很快的就开始使用生成故事 脚本。 可以先 clone the repository, 再 run `python parser.py <input_filename>`.

举个例子,

    git clone https://github.com/Yangzhedi/story-grammar.git
    cd story-grammar
    python parser.py stories/simple.txt

你应该会在cmd中看到这样的提示:

    Starting rule: ['Story']
    Setting Protagonist (chineseNames) to value 巫师
    Setting ProtagonistAdjective (chineseAdjectives) to value 尖叫的
    很久很久以前，有一个人叫做巫师. 巫师是一个尖叫的的战士. 接着巫师 了一个尖叫的旅行. 然后巫师就很快死掉了

Pretty scary stuff!

## 工作原理

### 规则

这个故事生成器用了一个简单的 基于规则的 语法 来描述一个故事。故事是由 一些由其他规则组成的细小的规则 一起构成的, 然后选择一个规则统一别的规则来建立一个故事.

在每一个新一行的开始，每个规则都是一个单词（或者称为令牌）. 下一行或下几行， 都是各种形式的规则。 举个例子， 如果你在某人打招呼， 你可能会说 `Hi!`, or `Hello!`, or even `How are you?`。 在我们的语法中，我们可能用以下形式来代表问候语：

    Greeting
      "Hi!"
      "Hello!"
      "How are you?"

如果我们想表达一句问候，就可以用以上三个静态的响应之一，我们总结了双引号内的文本。如果我们生成一句问候，我们的规则给与我们这三个可能的响应。

当我们把规则变成由其他规则一起组成的时候，故事就开始变得有趣了起来了。 举个例子，看看这个：

    Story
      Beginning End

    Beginning
      "The cat "
      "The dog "

    End
      "was nice."
      "was happy."

如果我们用`Story`作为“starting”规则， 就像 `Greeting` 在前面的例子中一样。 我们看到一个 `Story` 是由一个 `Beginning` 和一个 `End` 组成的。有两个可能的beginnings，和两个可能的endings。如果这就是我们的故事生成器，我们就可以看到这样的故事：

    The dog was happy.
    The cat was happy.
    The cat was nice.

我相信你会同意的，这些都是可爱的故事。

### 生成命令

如果在我们的故事生成器中使用这个，我们就需要更多的片段。 我们讨论过使用 `Story` 作为我们的主规则, 所以我们只需要确保 生成器 知道我们在想什么。 如果我们用一个单词（或者令牌）开始生成新的一行`Generate`, 我们将能够用友好的语法来告知程序如何生成我们的故事。

    Generate Story

完整的例子在`stories/animals.txt`, 你可以用下面的命令生成故事：

    python parser.py stories/animals.txt

### 变量， 让故事更加有趣

如果我们只是使用规则，那么我们的故事就会变得枯燥无味。 我们的动物的例子就说明了，但是在四个不同的故事有一个最大的限度？。 可以让事情变得更有趣的方法就是引入变量。

用 `Use` 命令， 随着一个变量类型， 和每一行上有可能替换的集合的文本文件， 我们可以定义一个在我们的语法中使用的变量。

这是什么意思？ 让我们看一下这个文件： `text/animals.txt`, 所有的包含都在这里：

    dog
    cat
    otter
    lion
    mouse
    aardvark
    elephant
    bird

正如你所见，这是一个包括很多动物的全面列表。 (好吧，我有可能遗漏了一两个)。 让我们 通过 `Use` 来使用 `text/animals.txt` 里面的 `Animal` list来修改我们的动物故事生成器

    Use Animal text/animals.txt

    Story
      Beginning End

    Beginning
      "The " Animal.MyAnimal " "
      "The " Animal.MyAnimal " "

    End
      "was nice."
      "was happy."

    Generate Story

和我们之前的例子有两个主要的区别。

第一个就是 第一行的开头使用了 `Use`, 意味着我们想要定义一个叫`Animal`的新的变量类型， 一定会采用一个在 `text/animals.txt` 某一行中的值. 这意味着 当我们在我们的规则之一得到了一个 `Animal`。 我们就可以 `text/animals.txt`中插入其中的一个可能。

第二, 我们已经在 `Beginning` 规则的双引号中结束了我们的文本,但是我们插入了`Animal.MyAnimal` (在下一段再解释) 并且把这三块合并到了一起。 第一块就是文本 `"The "` (注意空格). 第三块是文本`" "` (只是一个空格)。 中间的部分就是提到的Animal variable, 事实表明，它不在双引号内， 和期间的存在。 期间的左边表示我们要替换的变量 (`Animal`)，右边是一个可选的变量名， 如果我们想要在后边的文章再次使用相同的变量。 (我们应该写成 `Animal.` 而不是 `Animal.MyAnimal`)

让我们看看这个程序的运行起来的几种情况：

    python parser.py stories/animals-variable.txt
    Starting rule: ['Story']
    Setting MyAnimal (Animal) to value dog
    The dog was happy.

and..

    python parser.py stories/animals-variable.txt
    Starting rule: ['Story']
    Setting MyAnimal (Animal) to value aardvark
    The aardvark was nice.

默认情况下， 该解析器会告诉你一个值被分配给一个变量了， 就像我们使用 `MyAnimal` 和 `Animal` 变量. 如果我们使用 `Animal.MyAnimal` 在 `End` 规则, 这就会在故事中出现的相同的值。 举个例子, 如果我们写成这样：

    End
      "was a nice " Animal.MyAnimal "."
      "was a happy " Animal.MyAnimal "."

我们可能会看到这样的输出:

    The dog was a happy dog.
    The aardvark was a nice aardvark.
