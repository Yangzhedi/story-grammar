# story-grammar in Chinese
以下只是原README的简单翻译：

一个可以用来生成故事的语法, 灵感来自 [NaNoGenMo 2016](https://github.com/NaNoGenMo/2016/).

## 开始
你可以很快的就开始使用生成故事 脚本。 可以 clone the repository, 再 run `python parser.py <input_filename>`.

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

这个故事生成器用了一个简单的 基于规则的语法 来描述一个故事。故事是由 一些由其他规则组成的细小的规则 一起构成的, 然后选择一个规则统一别的规则来建立一个故事.

在每一个新一行的开始，每个规则都是一个单词（或者称为令牌）. 下一行或下几行， 都是各种形式的规则。 举个例子， 如果你在某人打招呼， 你可能会说 `Hi!`, or `Hello!`, or even `How are you?`。 在我们的语法中，我们可能用以下形式来代表：

    Greeting
      "Hi!"
      "Hello!"
      "How are you?"

如果我们想表达一句问候，就可以用以上三个静态的响应之一，我们总结了双引号内的文本。如果我们生成一句问候，我们的规则给与我们这三个可能的响应。

当我们把规则变成由其他规则一起组成的时候，故事就开始变得有趣了起来了。 举个例子，看看这个：

    Story
      Beginning End

    Beginning
      "阿猫 "
      "阿狗 "

    End
      "很好。"
      "很开心。"

如果我们用`Story`作为“starting”的规则， 就像 `Greeting` 在前面的例子中一样。 我们看到一个 `Story` 室由一个 `Beginning` 和一个 `End` 组成的。有两个可能的beginnings，和两个可能的endings。如果这就是我们的故事生成器，我们就可以看到这样的故事：

    阿猫 很好。
    阿狗 很好。
    阿猫 很开心。
    阿狗 很开心。

我相信你会同意的，这些都是可爱的故事。

### 生成命令

如果在我们的故事生成器中使用这个，我们就需要更多的片段。 我们讨论过使用 `Story` 作为我们的主规则, 所以我们只需要确保 生成器 知道我们在想什么。 If we start a new line with the word (or token) `Generate`, we'll be able to inform the program how to write stories using our friendly grammar.

    Generate Story

The complete example is in `stories/animals.txt`, and you can generate stories using the following command:

    python parser.py stories/animals.txt

### Variables, to make things interesting

If we just use rules, our stories might get boring after a while. Our animals example is illustrative, but there are a maximum of four different stories. One way to make things more interesting is to introduce variables.

By using the `Use` command, along with a variable type, and a text file with a collection of possible replacements on each line, we can define a variable for use in our grammar.

What does this mean? Let's consider the file `text/animals.txt`, included here in full:

    dog
    cat
    otter
    lion
    mouse
    aardvark
    elephant
    bird

As you can see, this is a comprehensive list of every animal. (Okay, I might have missed one or two!) Let's modify our animal story generator above to `Use` the `Animal` list found in `text/animals.txt`.

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

There are two main differences from our earlier example.

First is the line beginning with `Use`, which says that we'd like to define a new variable of type `Animal`, which must take on one of the values found on one of the lines in `text/animals.txt`. This means that when we get an `Animal` in one of our rules, we'd like to insert one of the possibilities found in `text/animals.txt`.

Second, we've wrapped up our text in double-quotes again inside the `Beginning` rule, but we've inserted `Animal.MyAnimal` (we'll explain this in the next paragraph!) and glued three pieces together. The first piece is the text `"The "` (note the space). The third piece is the text `" "` (just a single space!). The middle piece is a reference to the Animal variable, indicated by the fact that it's not inside double-quotes, and the presence of the period. The left side of the period indicates the variable we want to substitute (`Animal`), and the right side is an optional variable name, if we want to use this same variable again later in our story. (We could have written `Animal.` instead of `Animal.MyAnimal`)

Let's see what a few runs of this program looks like:

    python parser.py stories/animals-variable.txt
    Starting rule: ['Story']
    Setting MyAnimal (Animal) to value dog
    The dog was happy.

and..

    python parser.py stories/animals-variable.txt
    Starting rule: ['Story']
    Setting MyAnimal (Animal) to value aardvark
    The aardvark was nice.

By default, the parser will let you know when a value has been assigned to a variable, as in our use of `MyAnimal` with the `Animal` variable. If we'd used `Animal.MyAnimal` in the `End` rule, it would have been the same value found earlier in the story. For example, if we wrote this:

    End
      "was a nice " Animal.MyAnimal "."
      "was a happy " Animal.MyAnimal "."

We might see output like this:

    The dog was a happy dog.
    The aardvark was a nice aardvark.
