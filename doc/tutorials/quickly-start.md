# Makefile 快速入门

参考：{cite:p}`SimpleMakefileTutorial`。

{guilabel}`目标`：快速、轻松地为中小型项目创建自己的 Makefile。

Makefile 是组织代码编译的一种简单方法。直接从例子开始：以下三个文件：{download}`cpp/src/hellomake.c`、{download}`cpp/src/hellofunc.c` 和 {download}`cpp/include/hellomake.h`，分别代表典型的主程序、单独文件中的一些函数代码和 include 文件。

::::{grid} 3
:gutter: 1

:::{grid-item-card}
```{include} cpp/src/hellomake.c
:code: c
```
:::
:::{grid-item-card}
```{include} cpp/src/hellofunc.c
:code: c
```
:::
:::{grid-item-card}
```{include} cpp/include/hellomake.h
:code: c
```
:::
::::

通常，可以通过执行下面的命令来编译这些代码：

```shell
cd src
gcc -o hellomake hellomake.c hellofunc.c -I .
```

这将编译两个 `.c` 文件，并将可执行文件命名为 `hellomake`。`-I .` 被包含，以便 GCC 在当前目录（`.`）中查找 include 文件 `hellomake.h`。在没有 Makefile 的情况下，测试/修改/调试循环的典型方法是在终端中使用向上箭头返回到上一个编译命令，这样就不必每次都输入它，特别是在添加了几个 `.c` 文件之后。

不幸的是，这种编译方法有两个缺点。首先，如果您丢失了编译命令或切换了计算机，您必须从头开始重新输入它，这是效率最低的。其次，如果您只对一个 `.c` 文件进行更改，那么每次重新编译所有这些文件也是非常耗时和低效的。Makefile 提供了一些便利。

最简单的 Makefile 如下所示：

```Makefile
hellomake: hellomake.c hellofunc.c
     gcc -o hellomake hellomake.c hellofunc.c -I .
```

将该规则放入名为 `Makefile` 或 `makefile` 的文件中，然后在命令行上键入 {program}`make`，它将执行 `Makefile` 中编写的编译命令。注意，不带参数的 {program}`make` 执行文件中的第一条规则。此外，通过将命令所依赖的文件列表放在 `:` 之后的第一行，{program}`make` 知道如果其中任何文件发生更改，就需要执行 `hellomake` 规则。立即解决了第 1 个问题，可以避免重复使用向上箭头，查找最后一个编译命令。然而，就只编译最新的更改而言，该系统仍然没有效率。

需要注意的一件非常重要的事情是，在 `Makefile` 中的 {program}`gcc` 命令之前有制表符。在任何命令的开头都必须有制表符，如果没有这个制表符，{program}`make` 会困扰的。

为了更有效率，试试下面的方法：

```Makefile
CC = gcc
CFLAGS = -I .

hellomake: hellomake.o hellofunc.o
	$(CC) -o hellomake hellomake.o hellofunc.o
```

定义一些常量 {makevar}`CC` 和 {makevar}`CFLAGS`。事实证明，它们都是一些特殊的常量，它们通过 {program}`make` 通信来确定想要编译的 `hellomake.c` 和 `hellofunc.c` 文件方式。具体来说，宏 {makevar}`CC` 是要使用的 C 编译器，而 {makevar}`CFLAGS` 是要传递给编译命令的 flags 列表。通过放置目标文件—— `hellomake.o` 和 `hellofunc.o` —— 在依赖项列表和规则中，{program}`make` 知道它必须首先单独编译 `.c` 版本，然后构建可执行的 `hellomake`。

使用这种形式的 Makefile 对于大多数小型项目来说已经足够了。然而，有件事被遗漏了：对 include 文件的依赖。例如，如果要对 `hellomake.h` 进行更改，{program}`make` 将不会重新编译 `.c` 文件，即使它们需要重新编译。为了解决这个问题，需要告知 {program}`make` 所有的 `.c` 文件依赖于特定的 `.h` 文件。可以通过编写简单的规则并将其添加到 Makefile 来实现这一点。

```Makefile
CC = gcc
CFLAGS = -I .
DEPS = hellomake.h

%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

hellomake: hellomake.o hellofunc.o
	$(CC) -o hellomake hellomake.o hellofunc.o
```

这一添加首先创建宏 {makevar}`DEPS`，它是 `.c` 文件所依赖的一组 `.h` 文件。然后定义适用于所有后缀为 `.o` 的文件的规则。该规则指出，`.o` 文件依赖于文件的 `.c` 版本和 {makevar}`DEPS` 宏中包含的 `.h` 文件。根据规则，要生成 `.o` 文件， make 需要使用 {makevar}`CC` 宏中定义的编译器编译 `.c` 文件。`-c` 标志表示生成目标文件，`-o $@` 表示将编译的输出放在 `:` 左边、{makevar}`$<` 是依赖项列表中的第一项，{makevar}`CFLAGS` 宏的定义如上所述。

作为最后的简化，使用特殊的宏 {makevar}`$@` 和 {makevar}`$^`，它们分别位于 {makevar}`:` 的左右两侧，以使整个编译规则更加通用。在下面的例子中，所有的 include 文件都应该作为宏 {makevar}`DEPS` 的一部分列出，所有的目标文件都应该作为宏 {makevar}`OBJ` 的一部分列出。

```Makefile
CC = gcc
CFLAGS = -I .
DEPS = hellomake.h
OBJ = hellomake.o hellofunc.o

%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

hellomake: $(OBJ)
	$(CC) -o $@ $^ $(CFLAGS)
```

那么，如果想把 `.h` 文件放在 `include/` 目录中，源代码放在 `src/` 目录中，而一些本地库放在 `lib/` 目录中，该怎么办呢？此外，能否隐藏那些到处挂着的烦人的 `.o`文件？答案当然是肯定的。下面的 Makefile 定义了 `include/` 和 `lib/` 目录的路径，并将对象文件放在 `obj/` 目录中。它还为您想要包含的任何库定义了宏，例如数学库 `-lm`。请注意，它还包括一个规则，用于在输入 `make clean` 时清理源目录和对象目录。`.PHONY` 规则防止 {program}`make` 对名为 `clean` 的文件进行操作。

```Makefile
IDIR = include
CC = gcc
CFLAGS = -I $(IDIR)
ODIR = obj
LDIR = src
LIBS = -lm
_DEPS = hellomake.h
DEPS = $(patsubst %,$(IDIR)/%,$(_DEPS))
_OBJ = hellomake.o hellofunc.o 
OBJ = $(patsubst %,$(ODIR)/%,$(_OBJ))


$(ODIR)/%.o: $(LDIR)/%.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

run: $(OBJ)
	$(CC) -o $@ $^ $(CFLAGS) $(LIBS)

.PHONY: clean

clean:
	rm -f $(ODIR)/*.o *~ core $(INCDIR)/*~ run
```

````{note}
目录结构如下：

```{figure} images/makefile.png
:width: 600
:height: 300
```
````

现在有了非常好的 Makefile，可以修改它来管理中小型软件项目。可以在 Makefile 中添加多条规则；甚至可以创建调用其他规则的规则。有关 Makefile 和 `make` 函数的更多信息，请参阅 [GNU make Manual](http://www.gnu.org/software/make/manual/make.html)，它将告诉您比您想要知道的更多（实际）。
