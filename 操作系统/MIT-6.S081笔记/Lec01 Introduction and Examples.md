# 课程内容简介
这门课程的目标有：
1. 理解操作系统的设计和实现。设计是指整体的结构，实现是指具体的代码长什么样。对于这两者，我们都会花费大量时间讲解。
2. 为了深入了解具体的工作原理，你们可以通过一个小的叫做XV6的操作系统，获得实际动手经验。通过研究现有的操作系统，并结合课程配套的实验，你可以获得扩展操作系统，修改并提升操作系统的相关经验，并且能够通过操作系统接口，编写系统软件。

操作系统的共性（是什么）:
1. 抽象硬件。从硬件角度而言，操作系统就是C程序
2. 协调硬件资源。保证多个应用程序能共享硬件资源
3. 隔离性。保证多个应用程序不会相互干扰
4. 内容的限定共享。有些东西应该要共享，这样多个应用程序都可以使用。有些出于安全考虑不应该进行共享
5. 发挥硬件的性能

# 操作系统结构
操作系统内部可以分为两个部分，用户空间和内核空间。
* 用户空间负责运行我们的各种应用程序，像是编辑器、编译器之类的。
* 内核空间运行Kernel。Kernel是计算机资源的守护者。当你打开计算机时，Kernel总是第一个被启动。Kernel程序只有一个，它维护数据来管理每一个用户空间进程。Kernel同时还维护了大量的数据结构来帮助它管理各种各样的硬件资源，以供用户空间的程序使用。Kernel同时还有大量内置的服务

在这们课中，我们围绕Kernel展开，主要关注Kernel与用户空间连接的接口（API）、Kernel内部软件的架构。

我们还会关注Kernel中的服务，像是文件系统、进程管理系统、网络通信。

我们也关注接口到底是什么，这决定了应用程序如何访问Kernel，通常来说，这里是通过所谓的系统调用（System Call）来完成。系统调用与程序中的函数调用看起来是一样的，但区别是系统调用会实际运行到系统内核中，**并执行内核中对于系统调用的实现**（系统调用在Kernel中实现）。

# 课程结构和资源
XV6是我们的一个小的用于教学的操作系统，我们会介绍它是如何工作，查看它的代码，并在课程中演示代码的运行。

在每一节课程之前都会有作业，作业会要求你们阅读介绍XV6的书籍，书籍的内容是XV6如何运行以及设计思想。所以你应该在课程之前完成相应的阅读，这样你才能理解课程的讨论内容。

这门课程的下一大部分是lab，几乎每周都会有一些编程实验。实验的意义在于帮助你获得一些使用和实现操作系统的实际动手经验。

# read, write, exit系统调用
XV6运行在一个RISC-V微处理器上。理论上，我们可以在一个RISC-V计算机上运行XV6。但是我们会在一个QEMU模拟器上运行XV6。

make qemu，这条指令会编译并构建xv6内核和所有的用户进程，并将它们运行在QEMU模拟器下。make clean 会清楚仓库中不相关的文件。事实上这些都可以在Makefile中找到答案，可以看到他们具体是怎么执行的，像make clean就是清除 .o、.tex一类的不相关文件。

像下图这样出现 $ 符号就是已经启动了操作系统，接着我们就可以把它当作一个XV6的Shell进行使用。
![[Pasted image 20230815114141.png]]

XV6本身很小，并且自带了一小部分的工具程序，例如ls。我这里运行ls，它会输出xv6中的所有文件，这里只有20多个。可以看到这都是一些自带的命令工具。
![Pasted image 20230815114815.png](Pasted%20image%2020230815114815.png)

此处演示了一个copy.c的程序，其执行了三个系统调用，分别是read，write和exit。事实上在我们的书中就已经列好XV6的所有系统调用，他们也在user.h中有写明。
![[Pasted image 20230815115819.png]]

# Shell
学生提问：有一个系统调用和编译器的问题。编译器如何处理系统调用？生成的汇编语言是不是会调用一些由操作系统定义的代码段？

Robert教授：有一个特殊的RISC-V指令，程序可以调用这个指令，并将控制权交给内核。所以，实际上当你运行C语言并执行例如open或者write的系统调用时，从技术上来说，open是一个C函数，但是这个函数内的指令实际上是机器指令，也就是说我们调用的open函数并不是一个C语言函数，它是由汇编语言实现，组成这个系统调用的汇编语言实际上在RISC-V中被称为ecall。这个特殊的指令将控制权转给内核。之后内核检查进程的内存和寄存器，并确定相应的参数。

也就是说，系统调用虽然是C函数，但其内核实现是汇编语言，也就是机器码。

# fork系统调用
fork会拷贝当前进程的内存，并创建一个新的进程，这里的内存包含了进程的指令和数据。之后，我们就有了两个拥有完全一样内存的进程。fork系统调用在两个进程中都会返回，在原始的进程中，fork系统调用会返回大于0的整数，这个是新创建进程的ID。而在新创建的进程中，fork系统调用会返回0。**所以即使两个进程的内存是完全一样的，我们还是可以通过fork的返回值区分旧进程和新进程。**

下图程序中，父进程先创建一个子进程，接着进程就拥有了相同的指令和数据，会一起向下执行。接着分支语句会根据fork指令返回的pid进行判断，pid == 0就是子进程，打印 child。否则就是父进程，打印parent。
![[Pasted image 20230815120349.png]]

接着会出现这样的打印情况，这是因为进程会同时进行，同时进行打印，所以这里相当于打印了两个fork，两个returned，可以猜测子进程pid = 19。
![[Pasted image 20230815120536.png]]

>学生提问：fork产生的子进程是不是总是与父进程是一样的？它们有可能不一样吗？
>
>Robert教授：在XV6中，除了fork的返回值，两个进程是一样的。两个进程的指令是一样的，数据是一样的，栈是一样的，同时，两个进程又有各自独立的地址空间，它们都认为自己的内存从0开始增长，但这里是不同的内存。 在一个更加复杂的操作系统，有一些细节我们现在并不关心，这些细节偶尔会导致父子进程不一致，但是在XV6中，父子进程除了fork的返回值，其他都是一样的。除了内存是一样的以外，文件描述符的表单也从父进程拷贝到子进程。所以如果父进程打开了一个文件，子进程可以看到同一个文件描述符，尽管子进程看到的是一个文件描述符的表单的拷贝。除了拷贝内存以外，fork还会拷贝文件描述符表单这一点还挺重要的，我们接下来会看到。

fork指令是一个很重要的指令，事实上当我们在Shell中运行东西的时候，Shell实际上会创建一个新的进程来运行你输入的每一个指令。

# exec, wait系统调用
有关exec系统调用，有一些重要的事情：
1. exec系统调用会保留当前的文件描述符表单。所以任何在exec系统调用之前的文件描述符，例如0，1，2等。它们在新的程序中表示相同的东西。
2. 通常来说exec系统调用不会返回，因为exec会完全替换当前进程的内存，相当于当前进程不复存在了，所以exec系统调用已经没有地方能返回了。

所以，exec系统调用从文件中读取指令，执行这些指令，然后就没有然后了。exec系统调用只会当出错时才会返回，因为某些错误会阻止操作系统为你运行文件中的指令，例如程序文件根本不存在，因为exec系统调用不能找到文件，exec会返回-1来表示：出错了，我找不到文件。所以通常来说exec系统调用不会返回，它只会在kernel不能运行相应的文件时返回。

这里就会出现一个问题，我们需要使用exec来执行一些命令，但这样就会替代当前进程，因而我们不希望Shell被替代掉（Shell实际上也是一个进程），因而我们在Shell中运行指令时，都会先执行fork，之后在fork出的子进程再调用exec系统调用，这是一个非常常见的Unix程序调用风格。对于那些想要运行程序，但是还希望能拿回控制权的场景，可以先执行fork系统调用，然后在子进程中调用exec。

这里需要注意的是，fork首先拷贝了整个父进程的，但是之后exec整个将这个拷贝丢弃了，并用你要运行的文件替换了内存的内容。某种程度上来说这里的拷贝操作浪费了，因为所有拷贝的内存都被丢弃并被exec替换。**之后的课程我们会对这种操作进行优化。**

>Robert教授：Unix并没有一个直接的方法让子进程等待父进程。wait系统调用只能让父进程等待当前进程的子进程。

# I/O Redirect
在这个例子中，我们将讲述为什么Shell能实现I/O重定向。
![[Pasted image 20230815143028.png]]
第13行，Shell创建了一个子进程，这是一个经典套路，然后让子进程去执行echo的操作。但在此之前，子进程close了文件描述符1，而这通常是进程用来作为输出的，也就是Stdout。

然后子进程open了一个文件，这个地方也会返回一个文件描述符，用于表示引用该文件，由于open会返回当前进程未使用的最小文件描述符序号，且因为我们刚刚关闭了文件描述符1，而文件描述符0还对应着console的输入，所以open一定可以返回1。在代码第16行之后，文件描述符1与文件output.txt关联。

这样我们就实现了I/O的重定向。

这个例子同时也演示了分离fork和exec的好处。fork和exec是分开的系统调用，意味着在子进程中有一段时间，fork返回了，但是exec还没有执行，子进程仍然在运行父进程的指令。在这个过程中，我们就可以进行一个操作，像是I/O重定向一类的操作。

# 总结
这节课主要学习了一些进程的接口和抽象。
* 这里需要记住的是，接口是相对的简单，你只需要传入表示文件描述符的整数，和进程ID作为参数给相应的系统调用。而接口内部的实现逻辑相对来说是复杂的，比如创建一个新的进程，拷贝当前进程。
* 除此之外，还展示了一些例子。通过这些例子你可以看到，尽管接口本身是简单的，但是可以将多个接口结合起来形成复杂的用例。比如说创建I/O重定向。




