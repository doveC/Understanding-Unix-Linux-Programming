### 1. stdin, stdout, stderr

stdin, stdout, stderr都是FILE*类型的变量，代表3个默认打开的文件

![1557908870480](C:\Users\a1599\AppData\Roaming\Typora\typora-user-images\1557908870480.png)

（stdin、stdout、stderr在头文件stdio.h中的定义）

FILE* 是IO相关库函数的操作句柄，其中的缓冲区等功能属于库函数提供的功能。fd文件描述符才是IO相关系统调用的句柄，fd的值表示某一个进程打开文件列表数组的下标。

### 2. 自己实现的more程序 version 0.1

程序分为三个函数，首先main函数中根据命令行参数决定more从什么地方得到输入，没有参数就从stdin获取输入，有参数就依次从参数所代表的文件中读取输入。然后do_more函数负责依次打印输入里的每一行，每打够PAGELEN行就调用see_more函数获取键盘输入决定是向下滚动还是向下翻页还是退出。

这个函数已经可以大致完成more的功能，不足之处是屏幕上的文字上滚的时候，最下面的提示符也会上滚。以及获取键盘输入的时候'q' 或 ' '必须要按回车才能提交给程序，而不是实时变化的。而且还不能显示正版more下面的阅读的百分比。还有就是如果配合上管道符使用，会有bug。

### 3. more程序 version 0.2

这个版本解决了管道符的bug。

首先区分输入输出重定向与管道符的区别。输入输出重定向一般是将程序的输出重定向到文件中或者另外的基本流中，在单个进程内就可以完成，例如将当前进程的stdout通过系统调用dup2重定向到需要的文件中，这样输入就不送往屏幕而会写入文件。但管道符一般用来将一个进程的输出传输给另一个进程作为另一个进程的输入，这是发生在不同的进程中的。

上面所说的管道符的bug在于，如果将一个进程例如ls /bin 的输出重定向到了管道，而把我们自己写的more程序的标准输入也重定向到管道中，这样more函数确实可以从管道读取输入，但more里最关键的一步通过键盘输入决定是滚动还是翻页就做不到了，因为标准输入已经被重定向了。解决的方法就在于系统中的一个文件"/dev/tty"，这是键盘和显示器的设备描述文件，向这个文件写相当于写在用户的屏幕上，读相当于从键盘上读取用户输入。即使程序的输入/输出被重定向，程序还是可以通过这个文件与终端交换数据。

另外三个不足还是没解决。