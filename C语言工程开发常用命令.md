### C语言工程开发常用命令

#### 终端常用命令

1. 查看自己在操作系统中当前位置：`pwd`
2. 切换我们所在的位置：`cd`
3. 查看我们所在的位置有哪些文件和目录：`ls -l`
4. 将文件a移动到文件b：`mv a b`
5. 创建一个新的文件：`touch`
6. 删除一个文件：`rm`
7. 显示文件内容：`cat`

#### 多模块程序

1. 首先，对每一个模块编译出每一个模块对应的`*.o`目标代码文件，例如：

   ```c
   gcc -c -o set.o set.c
   ```

   会将我们的一个`set.c`文件编译成一个`set.o`的目标代码文件。

2. 完成每一个独立模块的编译之后，将主程序的目标代码文件与他们链接在一起，例如：

   ```c
   gcc -o program main.o set.o others.o
   ```

   将目标代码文件`set.o`和`others.o`与`main.o`链接在一起，并输出可执行文件`program`。

3. 注：为防止多次引用同一头文件，可利用宏定义，具体使用方式百度或查看计蒜客相关内容。

#### Makefile

1. 我们写一条规则（注：每一条规则的命令之前，必须要有一个制表符\t）：

   ```c
   array.o: array.c array.h
   	gcc -c -o array.o array.c
   ```

   表示生成的文件是目标代码文件`array.o`，它依赖于`array.c`和`array.h`。当我们在命令中执行`make array.o`时，根据这一规则，如果`array.o`不存在或者`array.c`与`array.h`至少之一有更新，就会执行`gcc -c -o array.o array.h`。

   我们把上述代码保存为`Makefile`，与`array.c`和`array.h`放在同一目录，在那个目录里执行`make array.o`就能看到效果。

2. 在`Makefile`有多条规则时，如果我们希望只生成其中一个，我们可以在`make`命令后加上需要生成的目标的名称。例如：

   ```c
   main: array.o main.o
       gcc -o main array.o main.o
   
   main.o: main.c array.h
       gcc -c -o main.o main.c
   
   array.o: array.c array.h
       gcc -c -o array.o array.c
   ```

   在这里我们可以执行`make main.o`、`make array.o`或`make main`。当我们执行`make main`时，`make`命令发现`array.o`和`main.o`不存在，就会根据它们为目标的规则先生成它们。

3. 将`.o`为后缀的目标代码文件和可执行的程序文件删除：

   ```c
   .PHONY: clean
   
   clean:
   	rm -f array.o main.o main
   ```

   `.PHONY`用于声明一些伪目标，`rm`命令表示删除文件，`-f`表示强制。

   当我们执行`make clean`就可以删除`array.o`、`main.o`和`main`了。

4. 在Makefile中我们还可以使用它的变量和注释。

   ```c
   # 井号开头的行是一个注释
   # 设置 C 语言的编译器
   CC = gcc
   
   # -g 增加调试信息
   # -Wall 打开大部分警告信息
   CFLAGS = -g -Wall
   
   # 整理一下 main 依赖哪些目标文件
   MAINOBJS = main.o array.o
   
   .PHONY: clean
   
   main: $(MAINOBJS)
   	$(CC) $(CFLAGS) -o main $(MAINOBJS)
   
   array.o: array.c array.h
   	$(CC) $(CFLAGS) -c -o array.o array.c
   
   main.o: main.c array.h
   	$(CC) $(CFLAGS) -c -o main.o main.c
   
   clean:
   	rm -f $(MAINOBJS) main
   ```

   以`#`开头的是注释，`CC`定义了编译器，`CFLAGS`变量标记了编译参数，`MAINOBJS`变量记录了`main`依赖的目标文件。定义的变量可以直接通过`$(变量名)`进行使用。

#### 文件操作

```c
FILE *fp;  //声明一个文件指针
fp = fopen(文件路径, 访问模式); //访问模式有r、w、a三种
fgetc(fp); //获得一个字符，指针后移一位
fputc('c', fp); //将字符'c'写入到fp关联的文件内
fscanf(in_fp, "%c", &a); //用fscanf从文件指针in_fp进行读取数据
fprintf(out_fp, "%c", a); //用fprintf向文件指针out_fp进行写出
fclose(fp); //将文件指针fp与文件的关联断开
```

#### 利用gdb调试代码

1. 想要利用gdb帮助我们调试代码，在编译时需使用`-g`作为一个编译参数，例如：

   ```c
   gcc -o program -g main.c
   ```

2. 之后再通过下方命令运行程序

   ```
   gdb ./program
   ```

3. 这时候会出现一系列关于`gdb`的说明和下方这样一个等待你通过命令进行调试的命令行

   ```c
   (gdb)
   ```

   下面介绍一些最基本的`gdb`的使用：

   - 输入`l`（list的首字母），gdb会列出带着行号的10行程序．`l`之后可以加行号作为参数，列出这一行开始的10行．
   - 输入`b`（breakpoint的首字母），表示设置程序运行的断点，程序运行到断点时就会暂停运行．`b`后面既可以加函数名作为参数也可以加行号作为参数．
   - 输入`r`（run的首字母），程序就会开始运行，并且停在我们设置的断点处：
     - 运行暂停时，输入`p 表达式`（print的首字母），则会在当前断点运行这个表达式并查看它的值；
     - 运行暂停时，输入`n`（next的首字母），程序会执行暂停位置后的下一条语句并再次暂停；
     - 运行暂停时，输入`c`（continue的首字母），程序会继续执行到下一个断点再暂停（如果没有断点就会执行直到结束）．

