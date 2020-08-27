

# AT&T ASM Syntax

  

## 介绍

  开发一个OS，尽管绝大部分代码只需要用C/C++等高级语言就可以了，但至少和硬件相关部分的代码需要使用汇编语言，另外，由于启动部分的代码有大小限制，使用精练的汇编可以缩小目标代码的Size。另外，对于某些需要被经常调用的代码，使用汇编来写可以提高性能。所以我们必须了解汇编语言，即使你有可能并不喜欢它。



  如果你是计算机专业的话，在大学里你应该学习过Intel格式的8086/80386汇编，这里就不再讨论。如果我们选择的OS开发工具是GCC以及GAS的话，就必须了解AT&T汇编语言语法，因为GCC/GAS只支持这种汇编语法。

  本书不会去讨论8086/80386的汇编编程，这类的书籍很多，你可以参考它们。这里只会讨论AT&T的汇编语法，以及GCC的内嵌汇编语法。



---------------



###  Syntax



#### 寄存器引用



  **引用寄存器**要在寄存器号前加百分号 `%` , 如:

```assembly
movl %eax, %ebx
```





  80386有如下寄存器：

  8个32-bit寄存器: 

```assembly
%eax，%ebx，%ecx，%edx，%edi，%esi，%ebp，%esp
```



  8个16-bit寄存器，事实上是上面8个32-bit寄存器的低16位：

```assembly
%ax，%bx，%cx，%dx，%di，%si，%bp，%sp
```



  8个8-bit寄存器：事实上是寄存器%ax，%bx，%cx，%dx的高8位 和 低8位；

```assembly
%ah，%al，%bh，%bl，%ch，%cl，%dh，%dl
```



  6个段寄存器：

```assembly
%cs(code)，%ds(data)，%ss(stack), %es，%fs，%gs
```



  3个控制寄存器：

```assembly
%cr0，%cr2，%cr3
```



  6个debug寄存器：

```assembly
%db0，%db1，%db2，%db3，%db6，%db7
```



  2个测试寄存器：

```assembly
%tr6，%tr7
```



  8个浮点寄存器栈：

```assembly
%st(0)，%st(1)，%st(2)，%st(3)，%st(4)，%st(5)，%st(6)，%st(7)
```



#### 操作数顺序

  操作数排列是从源（左）到目的（右）

```assembly
movl %eax(源）, %ebx(目的）
```



#### 立即数

  使用立即数，要在数前面加符号`$` , 如

```assembly
movl $0x04, %ebx
```

  或者：

```assembly
para = 0x04
movl $para, %ebx
```



  指令执行的结果是将立即数`04h`装入寄存器`ebx`。



#### 符号常数

  符号常数直接引用 如

```assembly
value: .long 0x12a3f2de
movl value, %ebx
```

  指令执行的结果是将常数`0x12a3f2de`装入寄存器`ebx`。



  引用符号地址在符号前加符号 `$` , 如

```assembly
movl $value, %ebx
```



则是将符号`value`的地址装入寄存器`ebx`。



#### 操作数的长度

  操作数的长度用加在指令后的符号表示 `b(byte, 8-bit)` ,  `w(word, 16-bits)` , `l(long, 32-bits)`，如:

```assembly
movb %al, %bl
movw %ax, %bx
movl %eax, %ebx
```



  如果没有指定操作数长度的话，编译器将按照目标操作数的长度来设置。比如指令:

```assembly
mov %ax, %bx
```



由于目标操作数bx的长度为word，那么编译器将把此指令等同于“movw %ax, %bx”。



同样道理，指令“mov $4, %ebx”等同于指令“movl $4, %ebx”，“push %al”等同于“pushb %al”。



对于没有指定操作数长度，但编译器又无法猜测的指令，编译器将会报错，比如指令“push $4”





#### 符号扩展和零扩展指令

  绝大多数面向80386的AT&T汇编指令与Intel格式的汇编指令都是相同的，符号扩展指令和零扩展指令则是仅有的不同格式指令。

  符号扩展指令和零扩展指令需要指定源操作数长度和目的操作数长度，即使在某些指令中这些操作数是隐含的。

  在AT&T语法中，符号扩展和零扩展指令的格式为，基本部分"movs"和"movz"（对应Intel语法的movsx和movzx），后面跟上源操作数长度和目的操作数长度。movsbl意味着movs （from）byte （to）long；movbw意味着movs （from）byte （to）word；movswl意味着movs （from）word （to）long。对于movz指令也一样。比如指令“movsbl %al, %edx”意味着将al寄存器的内容进行符号扩展后放置到edx寄存器中。

  其它的Intel格式的符号扩展指令还有：

  cbw -- sign-extend byte in %al to word in %ax；

  cwde -- sign-extend word in %ax to long in %eax；

  cwd -- sign-extend word in %ax to long in %dx:%ax；

  cdq -- sign-extend dword in %eax to quad in %edx:%eax；

  对应的AT&T语法的指令为cbtw，cwtl，cwtd，cltd。

#### 调用和跳转指令

  段内调用和跳转指令为"call"，"ret"和"jmp"，段间调用和跳转指令为"lcall"，"lret"和"ljmp"。

  段间调用和跳转指令的格式为“lcall/ljmp $SECTION, $OFFSET”，而段间返回指令则为“lret $STACK-ADJUST”。

#### 前缀

  操作码前缀被用在下列的情况：

  字符串重复操作指令(rep,repne)；

  指定被操作的段(cs,ds,ss,es,fs,gs)；

  进行总线加锁(lock)；

  指定地址和操作的大小(data16,addr16)；

  在AT&T汇编语法中，操作码前缀通常被单独放在一行，后面不跟任何操作数。例如，对于重复scas指令，其写法为：

  repne

  scas

  上述操作码前缀的意义和用法如下：

  指定被操作的段前缀为cs,ds,ss,es,fs,和gs。在AT&T语法中，只需要按照section:memory-operand的格式就指定了相应的段前缀。比如：lcall %cs:realmode_swtch

  操作数／地址大小前缀是“data16”和"addr16"，它们被用来在32-bit操作数／地址代码中指定16-bit的操作数／地址。

  总线加锁前缀“lock”，它是为了在多处理器环境中，保证在当前指令执行期间禁止一切中断。这个前缀仅仅对ADD, ADC, AND, BTC, BTR, BTS, CMPXCHG,DEC, INC, NEG, NOT, OR, SBB, SUB, XOR, XADD,XCHG指令有效，如果将Lock前缀用在其它指令之前，将会引起异常。

  字符串重复操作前缀"rep","repe","repne"用来让字符串操作重复“%ecx”次。

####  [内存](http://product.it168.com/list/b/0205_1.shtml)引用

  Intel语法的间接[内存](http://product.it168.com/list/b/0205_1.shtml)引用的格式为：

  section:[base+index*scale+displacement]

  而在AT&T语法中对应的形式为：

  section:displacement(base,index,scale)

  其中，base和index是任意的32-bit base和index寄存器。scale可以取值1，2，4，8。如果不指定scale值，则默认值为1。section可以指定任意的段寄存器作为段前缀，默认的段寄存器在不同的情况下不一样。如果你在指令中指定了默认的段前缀，则编译器在目标代码中不会产生此段前缀代码。

  下面是一些例子：

  -4(%ebp)：base=%ebp，displacement=-4，section没有指定，由于base＝%ebp，所以默认的section=%ss，index,scale没有指定，则index为0。

  foo(,%eax,4)：index=%eax，scale=4，displacement=foo。其它域没有指定。这里默认的section=%ds。

  foo(,1)：这个表达式引用的是指针foo指向的地址所存放的值。注意这个表达式中没有base和index，并且只有一个逗号，这是一种异常语法，但却合法。

  %gs:foo：这个表达式引用的是放置于%gs段里变量foo的值。

  如果call和jump操作在操作数前指定前缀“*”，则表示是一个绝对地址调用/跳转，也就是说jmp/call指令指定的是一个绝对地址。如果没有指定"*"，则操作数是一个相对地址。

  任何指令如果其操作数是一个[内存](http://product.it168.com/list/b/0205_1.shtml)操作，则指令必须指定它的操作尺寸(byte,word,long），也就是说必须带有指令后缀(b,w,l)。

##  GCC Inline ASM

  GCC支持在C/C++代码中嵌入汇编代码，这些汇编代码被称作GCC Inline ASM——GCC内联汇编。这是一个非常有用的功能，有利于我们将一些C/C++语法无法表达的指令直接潜入C/C++代码中，另外也允许我们直接写C/C++代码中使用汇编编写简洁高效的代码。

  1.基本内联汇编

  GCC中基本的内联汇编非常易懂，我们先来看两个简单的例子：

  __asm__("movl %esp,%eax"); // 看起来很熟悉吧！

  或者是

  __asm__("

  movl $1,%eax // SYS_exit

  xor %ebx,%ebx

  int $0x80

  ");

  或

  __asm__(

  "movl $1,%eax\r\t" \

  "xor %ebx,%ebx\r\t" \

  "int $0x80" \

  );

  基本内联汇编的格式是

  __asm__ __volatile__("Instruction List");

  1、__asm__

  __asm__是GCC关键字asm的宏定义：

  \#define __asm__ asm

  __asm__或asm用来声明一个内联汇编表达式，所以任何一个内联汇编表达式都是以它开头的，是必不可少的。

  2、Instruction List

  Instruction List是汇编指令序列。它可以是空的，比如：__asm__ __volatile__(""); 或__asm__ ("");都是完全合法的内联汇编表达式，只不过这两条语句没有什么意义。但并非所有Instruction List为空的内联汇编表达式都是没有意义的，比如：__asm__ ("":::"memory"); 就非常有意义，它向GCC声明：“我对[内存](http://product.it168.com/list/b/0205_1.shtml)作了改动”，GCC在编译的时候，会将此因素考虑进去。

  我们看一看下面这个例子：

  $ cat example1.c

  int main(int __argc, char* __argv[])

  {

  int* __p = (int*)__argc;

  (*__p) = 9999;

  //__asm__("":::"memory");

  if((*__p) == 9999)

  return 5;

  return (*__p);

  }

  在这段代码中，那条内联汇编是被注释掉的。在这条内联汇编之前，[内存](http://product.it168.com/list/b/0205_1.shtml)指针__p所指向的[内存](http://product.it168.com/list/b/0205_1.shtml)被赋值为9999，随即在内联汇编之后，一条if语句判断__p所指向的[内存](http://product.it168.com/list/b/0205_1.shtml)与9999是否相等。很明显，它们是相等的。GCC在优化编译的时候能够很聪明的发现这一点。我们使用下面的命令行对其进行编译：

  $ gcc -O -S example1.c

  选项-O表示优化编译，我们还可以指定优化等级，比如-O2表示优化等级为2；选项-S表示将C/C++源文件编译为汇编文件，文件名和C/C++文件一样，只不过扩展名由.c变为.s。

  我们来查看一下被放在example1.s中的编译结果，我们这里仅仅列出了使用gcc 2.96在redhat 7.3上编译后的相关函数部分汇编代码。为了保持清晰性，无关的其它代码未被列出。

  $ cat example1.s

  main:

  pushl %ebp

  movl %esp, %ebp

  movl 8(%ebp), %eax # int* __p = (int*)__argc

  movl $9999, (%eax) # (*__p) = 9999

  movl $5, %eax # return 5

  popl %ebp

  ret

  参照一下C源码和编译出的汇编代码，我们会发现汇编代码中，没有if语句相关的代码，而是在赋值语句(*__p)=9999后直接return 5；这是因为GCC认为在(*__p)被赋值之后，在if语句之前没有任何改变(*__p)内容的操作，所以那条if语句的判断条件(*__p) == 9999肯定是为true的，所以GCC就不再生成相关代码，而是直接根据为true的条件生成return 5的汇编代码（GCC使用eax作为保存返回值的寄存器）。

  我们现在将example1.c中内联汇编的注释去掉，重新编译，然后看一下相关的编译结果。

  $ gcc -O -S example1.c

  $ cat example1.s

  main:

  pushl %ebp

  movl %esp, %ebp

  movl 8(%ebp), %eax # int* __p = (int*)__argc

  movl $9999, (%eax) # (*__p) = 9999

  \#APP

  \# __asm__("":::"memory")

  \#NO_APP

  cmpl $9999, (%eax) # (*__p) == 9999 ?

  jne .L3 # false

  movl $5, %eax # true, return 5

  jmp .L2

  .p2align 2

  .L3:

  movl (%eax), %eax

  .L2:

  popl %ebp

  ret

  由于内联汇编语句__asm__("":::"memory")向GCC声明，在此内联汇编语句出现的位置[内存](http://product.it168.com/list/b/0205_1.shtml)内容可能了改变，所以GCC在编译时就不能像刚才那样处理。这次，GCC老老实实的将if语句生成了汇编代码。

  可能有人会质疑：为什么要使用__asm__("":::"memory")向GCC声明[内存](http://product.it168.com/list/b/0205_1.shtml)发生了变化？明明“Instruction List”是空的，没有任何对[内存](http://product.it168.com/list/b/0205_1.shtml)的操作，这样做只会增加GCC生成汇编代码的数量。

  确实，那条内联汇编语句没有对[内存](http://product.it168.com/list/b/0205_1.shtml)作任何操作，事实上它确实什么都没有做。但影响[内存](http://product.it168.com/list/b/0205_1.shtml)内容的不仅仅是你当前正在运行的程序。比如，如果你现在正在操作的[内存](http://product.it168.com/list/b/0205_1.shtml)是一块[内存](http://product.it168.com/list/b/0205_1.shtml)映射，映射的内容是外围I/O设备寄存器。那么操作这块[内存](http://product.it168.com/list/b/0205_1.shtml)的就不仅仅是当前的程序，I/O设备也会去操作这块[内存](http://product.it168.com/list/b/0205_1.shtml)。既然两者都会去操作同一块[内存](http://product.it168.com/list/b/0205_1.shtml)，那么任何一方在任何时候都不能对这块[内存](http://product.it168.com/list/b/0205_1.shtml)的内容想当然。所以当你使用高级语言C/C++写这类程序的时候，你必须让编译器也能够明白这一点，毕竟高级语言最终要被编译为汇编代码。

  你可能已经注意到了，这次输出的汇编结果中，有两个符号：#APP和#NO_APP，GCC将内联汇编语句中"Instruction List"所列出的指令放在#APP和#NO_APP之间，由于__asm__("":::"memory")中“Instruction List”为空，所以#APP和#NO_APP中间也没有任何内容。但我们以后的例子会更加清楚的表现这一点。

  关于为什么内联汇编__asm__("":::"memory")是一条声明[内存](http://product.it168.com/list/b/0205_1.shtml)改变的语句，我们后面会详细讨论。

  刚才我们花了大量的内容来讨论"Instruction List"为空是的情况，但在实际的编程中，"Instruction List"绝大多数情况下都不是空的。它可以有1条或任意多条汇编指令。

  当在"Instruction List"中有多条指令的时候，你可以在一对引号中列出全部指令，也可以将一条或几条指令放在一对引号中，所有指令放在多对引号中。如果是前者，你可以将每一条指令放在一行，如果要将多条指令放在一行，则必须用分号（；）或换行符（\n，大多数情况下\n后还要跟一个\t，其中\n是为了换行，\t是为了空出一个tab宽度的空格）将它们分开。比如：

  __asm__("movl %eax, %ebx

  sti

  popl %edi

  subl %ecx, %ebx");

  __asm__("movl %eax, %ebx; sti

  popl %edi; subl %ecx, %ebx");

  __asm__("movl %eax, %ebx; sti\n\t popl %edi

  subl %ecx, %ebx");

  都是合法的写法。如果你将指令放在多对引号中，则除了最后一对引号之外，前面的所有引号里的最后一条指令之后都要有一个分号(；)或(\n)或(\n\t)。比如：

  __asm__("movl %eax, %ebx

  sti\n"

  "popl %edi;"

  "subl %ecx, %ebx");

  __asm__("movl %eax, %ebx; sti\n\t"

  "popl %edi; subl %ecx, %ebx");

  __asm__("movl %eax, %ebx; sti\n\t popl %edi\n"

  "subl %ecx, %ebx");

  __asm__("movl %eax, %ebx; sti\n\t popl %edi;" "subl %ecx, %ebx");

  都是合法的。

  上述原则可以归结为：

  任意两个指令间要么被分号(；)分开，要么被放在两行；

  放在两行的方法既可以从通过\n的方法来实现，也可以真正的放在两行；

  可以使用1对或多对引号，每1对引号里可以放任一多条指令，所有的指令都要被放到引号中。

  在基本内联汇编中，“Instruction List”的书写的格式和你直接在汇编文件中写非内联汇编没有什么不同，你可以在其中定义Label，定义对齐(.align n )，定义段(.section name )。例如：

  __asm__(".align 2\n\t"

  "movl %eax, %ebx\n\t"

  "test %ebx, %ecx\n\t"

  "jne error\n\t"

  "sti\n\t"

  "error: popl %edi\n\t"

  "subl %ecx, %ebx");

  上面例子的格式是Linux内联代码常用的格式，非常整齐。也建议大家都使用这种格式来写内联汇编代码。

  3、__volatile__

  __volatile__是GCC关键字volatile的宏定义：

  \#define __volatile__ volatile

  __volatile__或volatile是可选的，你可以用它也可以不用它。如果你用了它，则是向GCC声明“不要动我所写的Instruction List，我需要原封不动的保留每一条指令”，否则当你使用了优化选项(-O)进行编译时，GCC将会根据自己的判断决定是否将这个内联汇编表达式中的指令优化掉。

  那么GCC判断的原则是什么？我不知道（如果有哪位朋友清楚的话，请告诉我）。我试验了一下，发现一条内联汇编语句如果是基本内联汇编的话（即只有“Instruction List”，没有Input/Output/Clobber的内联汇编，我们后面将会讨论这一点），无论你是否使用__volatile__来修饰，GCC 2.96在优化编译时，都会原封不动的保留内联汇编中的“Instruction List”。但或许我的试验的例子并不充分，所以这一点并不能够得到保证。

  为了保险起见，如果你不想让GCC的优化影响你的内联汇编代码，你最好在前面都加上__volatile__，而不要依赖于编译器的原则，因为即使你非常了解当前编译器的优化原则，你也无法保证这种原则将来不会发生变化。而__volatile__的含义却是恒定的。

  2、带有C/C++表达式的内联汇编

  GCC允许你通过C/C++表达式指定内联汇编中"Instrcuction List"中指令的输入和输出，你甚至可以不关心到底使用哪个寄存器被使用，完全靠GCC来安排和指定。这一点可以让程序员避免去考虑有限的寄存器的使用，也可以提高目标代码的效率。

  我们先来看几个例子：

  __asm__ (" " : : : "memory" ); // 前面提到的

  __asm__ ("mov %%eax, %%ebx" : "=b"(rv) : "a"(foo) : "eax", "ebx");

  __asm__ __volatile__("lidt %0": "=m" (idt_descr));

  __asm__("subl %2,%0\n\t"

  "sbbl %3,%1"

  : "=a" (endlow), "=d" (endhigh)

  : "g" (startlow), "g" (starthigh), "0" (endlow), "1" (endhigh));

  怎么样，有点印象了吧，是不是也有点晕？没关系，下面讨论完之后你就不会再晕了。（当然，也有可能更晕^_^）。讨论开始——

  带有C/C++表达式的内联汇编格式为：

  __asm__　__volatile__("Instruction List" : Output : Input : Clobber/Modify);

  从中我们可以看出它和基本内联汇编的不同之处在于：它多了3个部分(Input，Output，Clobber/Modify)。在括号中的4个部分通过冒号(:)分开。

  这4个部分都不是必须的，任何一个部分都可以为空，其规则为：

  如果Clobber/Modify为空，则其前面的冒号(:)必须省略。比如__asm__("mov %%eax, %%ebx" : "=b"(foo) : "a"(inp) : )就是非法的写法；而__asm__("mov %%eax, %%ebx" : "=b"(foo) : "a"(inp) )则是正确的。

  如果Instruction List为空，则Input，Output，Clobber/Modify可以不为空，也可以为空。比如__asm__ ( " " : : : "memory" );和__asm__(" " : : );都是合法的写法。

  如果Output，Input，Clobber/Modify都为空，Output，Input之前的冒号(:)既可以省略，也可以不省略。如果都省略，则此汇编退化为一个基本内联汇编，否则，仍然是一个带有C/C++表达式的内联汇编，此时"Instruction List"中的寄存器写法要遵守相关规定，比如寄存器前必须使用两个百分号(%%)，而不是像基本汇编格式一样在寄存器前只使用一个百分号(%)。比如__asm__( " mov %%eax, %%ebx" : : )；__asm__( " mov %%eax, %%ebx" : )和__asm__( " mov %eax, %ebx" )都是正确的写法，而__asm__( " mov %eax, %ebx" : : )；__asm__( " mov %eax, %ebx" : )和__asm__( " mov %%eax, %%ebx" )都是错误的写法。

  如果Input，Clobber/Modify为空，但Output不为空，Input前的冒号(:)既可以省略，也可以不省略。比如__asm__( " mov %%eax, %%ebx" : "=b"(foo) : )；__asm__( " mov %%eax, %%ebx" : "=b"(foo) )都是正确的。

  如果后面的部分不为空，而前面的部分为空，则前面的冒号(:)都必须保留，否则无法说明不为空的部分究竟是第几部分。比如， Clobber/Modify，Output为空，而Input不为空，则Clobber/Modify前的冒号必须省略（前面的规则），而Output前的冒号必须为保留。如果Clobber/Modify不为空，而Input和Output都为空，则Input和Output前的冒号都必须保留。比如__asm__( " mov %%eax, %%ebx" : : "a"(foo) )和__asm__( " mov %%eax, %%ebx" : : : "ebx" )。

  从上面的规则可以看到另外一个事实，区分一个内联汇编是基本格式的还是带有C/C++表达式格式的，其规则在于在"Instruction List"后是否有冒号(:)的存在，如果没有则是基本格式的，否则，则是带有C/C++表达式格式的。

  两种格式对寄存器语法的要求不同：基本格式要求寄存器前只能使用一个百分号(%)，这一点和非内联汇编相同；而带有C/C++表达式格式则要求寄存器前必须使用两个百分号(%%)，其原因我们会在后面讨论。



### Output

  Output用来指定当前内联汇编语句的输出。我们看一看这个例子：

  __asm__("movl %%cr0, %0": "=a" (cr0));

  这个内联汇编语句的输出部分为"=r"(cr0)，它是一个“操作表达式”，指定了一个输出操作。我们可以很清楚得看到这个输出操作由两部分组成：括号括住的部分(cr0)和引号引住的部分"=a"。这两部分都是每一个输出操作必不可少的。括号括住的部分是一个C/C++表达式，用来保存内联汇编的一个输出值，其操作就等于C/C++的相等赋值cr0 = output_value，因此，括号中的输出表达式只能是C/C++的左值表达式，也就是说它只能是一个可以合法的放在C/C++赋值操作中等号(=)左边的表达式。那么右值output_value从何而来呢？

  答案是引号中的内容，被称作“操作约束”（Operation Constraint），在这个例子中操作约束为"=a"，它包含两个约束：等号(=)和字母a，其中等号(=)说明括号中左值表达式cr0是一个Write-Only的，只能够被作为当前内联汇编的输入，而不能作为输入。而字母a是寄存器EAX / AX / AL的简写，说明cr0的值要从eax寄存器中获取，也就是说cr0 = eax，最终这一点被转化成汇编指令就是movl %eax, address_of_cr0。现在你应该清楚了吧，操作约束中会给出：到底从哪个寄存器传递值给cr0。

  另外，需要特别说明的是，很多文档都声明，所有输出操作的操作约束必须包含一个等号(=)，但GCC的文档中却很清楚的声明，并非如此。因为等号(=)约束说明当前的表达式是一个Write-Only的，但另外还有一个符号——加号(+)用来说明当前表达式是一个Read-Write的，如果一个操作约束中没有给出这两个符号中的任何一个，则说明当前表达式是Read-Only的。因为对于输出操作来说，肯定是必须是可写的，而等号(=)和加号(+)都表示可写，只不过加号(+)同时也表示是可读的。所以对于一个输出操作来说，其操作约束只需要有等号(=)或加号(+)中的任意一个就可以了。

  二者的区别是：等号(=)表示当前操作表达式指定了一个纯粹的输出操作，而加号(+)则表示当前操作表达式不仅仅只是一个输出操作还是一个输入操作。但无论是等号(=)约束还是加号(+)约束所约束的操作表达式都只能放在Output域中，而不能被用在Input域中。

  另外，有些文档声明：尽管GCC文档中提供了加号(+)约束，但在实际的编译中通不过；我不知道老版本会怎么样，我在GCC 2.96中对加号(+)约束的使用非常正常。

  我们通过一个例子看一下，在一个输出操作中使用等号(=)约束和加号(+)约束的不同。

  $ cat example2.c

  int main(int __argc, char* __argv[])

  {

  int cr0 = 5;

  __asm__ __volatile__("movl %%cr0, %0":"=a" (cr0));

  return 0;

  }

  $ gcc -S example2.c

  $ cat example2.s

  main:

  pushl %ebp

  movl %esp, %ebp

  subl $4, %esp

  movl $5, -4(%ebp) # cr0 = 5

  \#APP

  movl %cr0, %eax

  \#NO_APP

  movl %eax, %eax

  movl %eax, -4(%ebp) # cr0 = %eax

  movl $0, %eax

  leave

  ret

  这个例子是使用等号(=)约束的情况，变量cr0被放在[内存](http://product.it168.com/list/b/0205_1.shtml)-4(%ebp)的位置，所以指令mov %eax, -4(%ebp)即表示将%eax的内容输出到变量cr0中。

  下面是使用加号(+)约束的情况：

  $ cat example3.c

  int main(int __argc, char* __argv[])

  {

  int cr0 = 5;

  __asm__ __volatile__("movl %%cr0, %0" : "+a" (cr0));

  return 0;

  }

  $ gcc -S example3.c

  $ cat example3.s

  main:

  pushl %ebp

  movl %esp, %ebp

  subl $4, %esp

  movl $5, -4(%ebp) # cr0 = 5

  movl -4(%ebp), %eax # input ( %eax = cr0 )

  \#APP

  movl %cr0, %eax

  \#NO_APP

  movl %eax, -4(%ebp) # output (cr0 = %eax )

  movl $0, %eax

  leave

  ret

  从编译的结果可以看出，当使用加号(+)约束的时候，cr0不仅作为输出，还作为输入，所使用寄存器都是寄存器约束(字母a，表示使用eax寄存器)指定的。关于寄存器约束我们后面讨论。

  在Output域中可以有多个输出操作表达式，多个操作表达式中间必须用逗号(,)分开。例如：

  __asm__(

  "movl %%eax, %0 \n\t"

  "pushl %%ebx \n\t"

  "popl %1 \n\t"

  "movl %1, %2"

  : "+a"(cr0), "=b"(cr1), "=c"(cr2));

  2、Input

  Input域的内容用来指定当前内联汇编语句的输入。我们看一看这个例子：

  __asm__("movl %0, %%db7" : : "a" (cpu->db7));

  例中Input域的内容为一个表达式"a"[cpu->db7)，被称作“输入表达式”，用来表示一个对当前内联汇编的输入。

  像输出表达式一样，一个输入表达式也分为两部分：带括号的部分(cpu->db7)和带引号的部分"a"。这两部分对于一个内联汇编输入表达式来说也是必不可少的。

  括号中的表达式cpu->db7是一个C/C++语言的表达式，它不必是一个左值表达式，也就是说它不仅可以是放在C/C++赋值操作左边的表达式，还可以是放在C/C++赋值操作右边的表达式。所以它可以是一个变量，一个数字，还可以是一个复杂的表达式（比如a+b/c*d）。比如上例可以改为：__asm__("movl %0, %%db7" : : "a" (foo))，__asm__("movl %0, %%db7" : : "a" (0x1000))或__asm__("movl %0, %%db7" : : "a" (va*vb/vc))。

  引号号中的部分是约束部分，和输出表达式约束不同的是，它不允许指定加号(+)约束和等号(=)约束，也就是说它只能是默认的Read-Only的。约束中必须指定一个寄存器约束，例中的字母a表示当前输入变量cpu->db7要通过寄存器eax输入到当前内联汇编中。

  我们看一个例子：

  $ cat example4.c

  int main(int __argc, char* __argv[])

  {

  int cr0 = 5;

  __asm__ __volatile__("movl %0, %%cr0"::"a" (cr0));

  return 0;

  }

  $ gcc -S example4.c

  $ cat example4.s

  main:

  pushl %ebp

  movl %esp, %ebp

  subl $4, %esp

  movl $5, -4(%ebp) # cr0 = 5

  movl -4(%ebp), %eax # %eax = cr0

  \#APP

  movl %eax, %cr0

  \#NO_APP

  movl $0, %eax

  leave

  ret

  我们从编译出的汇编代码可以看到，在"Instruction List"之前，GCC按照我们的输入约束"a"，将变量cr0的内容装入了eax寄存器。

  \3. Operation Constraint

  每一个Input和Output表达式都必须指定自己的操作约束Operation Constraint，我们这里来讨论在80386平台上所可能使用的操作约束。

  1、寄存器约束

  当你当前的输入或输入需要借助一个寄存器时，你需要为其指定一个寄存器约束。你可以直接指定一个寄存器的名字，比如：

  __asm__ __volatile__("movl %0, %%cr0"::"eax" (cr0));

  也可以指定一个缩写，比如：

  __asm__ __volatile__("movl %0, %%cr0"::"a" (cr0));

  如果你指定一个缩写，比如字母a，则GCC将会根据当前操作表达式中C/C++表达式的宽度决定使用%eax，还是%ax或%al。比如：

  unsigned short __shrt;

  __asm__ ("mov %0，%%bx" : : "a"(__shrt));

  由于变量__shrt是16-bit short类型，则编译出来的汇编代码中，则会让此变量使用%ex寄存器。编译结果为：

  movw -2(%ebp), %ax # %ax = __shrt

  \#APP

  movl %ax, %bx

  \#NO_APP

  无论是Input，还是Output操作表达式约束，都可以使用寄存器约束。

  下表中列出了常用的寄存器约束的缩写。

  约束 Input/Output 意义

  r I,O 表示使用一个通用寄存器，由GCC在%eax/%ax/%al, %ebx/%bx/%bl, %ecx/%cx/%cl, %edx/%dx/%dl中选取一个GCC认为合适的。

  q I,O 表示使用一个通用寄存器，和r的意义相同。

  a I,O 表示使用%eax / %ax / %al

  b I,O 表示使用%ebx / %bx / %bl

  c I,O 表示使用%ecx / %cx / %cl

  d I,O 表示使用%edx / %dx / %dl

  D I,O 表示使用%edi / %di

  S I,O 表示使用%esi / %si

  f I,O 表示使用浮点寄存器

  t I,O 表示使用第一个浮点寄存器

  u I,O 表示使用第二个浮点寄存器

  2、[内存](http://product.it168.com/list/b/0205_1.shtml)约束

  如果一个Input/Output操作表达式的C/C++表达式表现为一个[内存](http://product.it168.com/list/b/0205_1.shtml)地址，不想借助于任何寄存器，则可以使用[内存](http://product.it168.com/list/b/0205_1.shtml)约束。比如：

  __asm__ ("lidt %0" : "=m"(__idt_addr)); 或 __asm__ ("lidt %0" : :"m"(__idt_addr));

  我们看一下它们分别被放在一个C源文件中，然后被GCC编译后的结果：

  $ cat example5.c

  // 本例中，变量sh被作为一个[内存](http://product.it168.com/list/b/0205_1.shtml)输入

  int main(int __argc, char* __argv[])

  {

  char* sh = (char*)&__argc;

  __asm__ __volatile__("lidt %0" : : "m" (sh));

  return 0;

  }

  $ gcc -S example5.c

  $ cat example5.s

  main:

  pushl %ebp

  movl %esp, %ebp

  subl $4, %esp

  leal 8(%ebp), %eax

  movl %eax, -4(%ebp) # sh = (char*) &__argc

  \#APP

  lidt -4(%ebp)

  \#NO_APP

  movl $0, %eax

  leave

  ret

  $ cat example6.c

  // 本例中，变量sh被作为一个[内存](http://product.it168.com/list/b/0205_1.shtml)输出

  int main(int __argc, char* __argv[])

  {

  char* sh = (char*)&__argc;

  __asm__ __volatile__("lidt %0" : "=m" (sh));

  return 0;

  }

  $ gcc -S example6.c

  $ cat example6.s

  main:

  pushl %ebp

  movl %esp, %ebp

  subl $4, %esp

  leal 8(%ebp), %eax

  movl %eax, -4(%ebp) # sh = (char*) &__argc

  \#APP

  lidt -4(%ebp)

  \#NO_APP

  movl $0, %eax

  leave

  ret

  首先，你会注意到，在这两个例子中，变量sh没有借助任何寄存器，而是直接参与了指令lidt的操作。

  其次，通过仔细观察，你会发现一个惊人的事实，两个例子编译出来的汇编代码是一样的！虽然，一个例子中变量sh作为输入，而另一个例子中变量sh作为输出。这是怎么回事？

  原来，使用[内存](http://product.it168.com/list/b/0205_1.shtml)方式进行输入输出时，由于不借助寄存器，所以GCC不会按照你的声明对其作任何的输入输出处理。GCC只会直接拿来用，究竟对这个C/C++表达式而言是输入还是输出，完全依赖与你写在"Instruction List"中的指令对其操作的指令。

  由于上例中，对其操作的指令为lidt，lidt指令的操作数是一个输入型的操作数，所以事实上对变量sh的操作是一个输入操作，即使你把它放在Output域也不会改变这一点。所以，对此例而言，完全符合语意的写法应该是将sh放在Input域，尽管放在Output域也会有正确的执行结果。

  所以，对于[内存](http://product.it168.com/list/b/0205_1.shtml)约束类型的操作表达式而言，放在Input域还是放在Output域，对编译结果是没有任何影响的，因为本来我们将一个操作表达式放在Input域或放在Output域是希望GCC能为我们自动通过寄存器将表达式的值输入或输出。既然对于[内存](http://product.it168.com/list/b/0205_1.shtml)约束类型的操作表达式来说，GCC不会自动为它做任何事情，那么放在哪儿也就无所谓了。但从程序员的角度而言，为了增强代码的可读性，最好能够把它放在符合实际情况的地方。

  约束 Input/Output 意义

  m I,O 表示使用系统所支持的任何一种[内存](http://product.it168.com/list/b/0205_1.shtml)方式，不需要借助寄存器

  3、立即数约束

  如果一个Input/Output操作表达式的C/C++表达式是一个数字常数，不想借助于任何寄存器，则可以使用立即数约束。

  由于立即数在C/C++中只能作为右值，所以对于使用立即数约束的表达式而言，只能放在Input域。

  比如：__asm__ __volatile__("movl %0, %%eax" : : "i" (100) );

  立即数约束很简单，也很容易理解，我们在这里就不再赘述。

  约束 Input/Output 意义

  i I 表示输入表达式是一个立即数(整数)，不需要借助任何寄存器

  F I 表示输入表达式是一个立即数(浮点数)，不需要借助任何寄存器

  4、通用约束

  约束 Input/Output 意义

  g I,O 表示可以使用通用寄存器，[内存](http://product.it168.com/list/b/0205_1.shtml)，立即数等任何一种处理方式。

  0,1,2,3,4,5,6,7,8,9 I 表示和第n个操作表达式使用相同的寄存器/[内存](http://product.it168.com/list/b/0205_1.shtml)。

  通用约束g是一个非常灵活的约束，当程序员认为一个C/C++表达式在实际的操作中，究竟使用寄存器方式，还是使用[内存](http://product.it168.com/list/b/0205_1.shtml)方式或立即数方式并无所谓时，或者程序员想实现一个灵活的模板，让GCC可以根据不同的C/C++表达式生成不同的访问方式时，就可以使用通用约束g。比如：

  \#define JUST_MOV(foo) __asm__ ("movl %0, %%eax" : : "g"(foo))

  JUST_MOV(100)和JUST_MOV(var)则会让编译器产生不同的代码。

  int main(int __argc, char* __argv[])

  {

  JUST_MOV(100);

  return 0;

  }

  编译后生成的代码为：

  main:

  pushl %ebp

  movl %esp, %ebp

  \#APP

  movl $100, %eax

  \#NO_APP

  movl $0, %eax

  popl %ebp

  ret

  很明显这是立即数方式。而下一个例子：

  int main(int __argc, char* __argv[])

  {

  JUST_MOV(__argc);

  return 0;

  }

  经编译后生成的代码为：

  main:

  pushl %ebp

  movl %esp, %ebp

  \#APP

  movl 8(%ebp), %eax

  \#NO_APP

  movl $0, %eax

  popl %ebp

  ret

  这个例子是使用[内存](http://product.it168.com/list/b/0205_1.shtml)方式。

  一个带有C/C++表达式的内联汇编，其操作表达式被按照被列出的顺序编号，第一个是0，第2个是1，依次类推，GCC最多允许有10个操作表达式。比如：

  __asm__ ("popl %0 \n\t"

  "movl %1, %%esi \n\t"

  "movl %2, %%edi \n\t"

  : "=a"(__out)

  : "r" (__in1), "r" (__in2));

  此例中，__out所在的Output操作表达式被编号为0，"r"(__in1)被编号为1，"r"(__in2)被编号为2。

  再如：

  __asm__ ("movl %%eax, %%ebx" : : "a"(__in1), "b"(__in2));

  此例中，"a"(__in1)被编号为0，"b"(__in2)被编号为1。

  如果某个Input操作表达式使用数字0到9中的一个数字（假设为1）作为它的操作约束，则等于向GCC声明：“我要使用和编号为1的Output操作表达式相同的寄存器（如果Output操作表达式1使用的是寄存器），或相同的[内存](http://product.it168.com/list/b/0205_1.shtml)地址（如果Output操作表达式1使用的是[内存](http://product.it168.com/list/b/0205_1.shtml)）”。上面的描述包含两个限定：数字0到数字9作为操作约束只能用在Input操作表达式中，被指定的操作表达式（比如某个Input操作表达式使用数字1作为约束，那么被指定的就是编号为1的操作表达式）只能是Output操作表达式。

  由于GCC规定最多只能有10个Input/Output操作表达式，所以事实上数字9作为操作约束永远也用不到，因为Output操作表达式排在Input操作表达式的前面，那么如果有一个Input操作表达式指定了数字9作为操作约束的话，那么说明Output操作表达式的数量已经至少为10个了，那么再加上这个Input操作表达式，则至少为11个了，以及超出GCC的限制。

  5、Modifier Characters（修饰符）

  等号(=)和加号(+)用于对Output操作表达式的修饰，一个Output操作表达式要么被等号(=)修饰，要么被加号(+)修饰，二者必居其一。使用等号(=)说明此Output操作表达式是Write-Only的，使用加号(+)说明此Output操作表达式是Read-Write的。它们必须被放在约束字符串的第一个字母。比如"a="(foo)是非法的，而"+g"(foo)则是合法的。

  当使用加号(+)的时候，此Output表达式等价于使用等号(=)约束加上一个Input表达式。比如

  __asm__ ("movl %0, %%eax; addl %%eax, %0" : "+b"(foo)) 等价于

  __asm__ ("movl %1, %%eax; addl %%eax, %0" : "=b"(foo) : "b"(foo))

  但如果使用后一种写法，"Instruction List"中的别名也要相应的改动。关于别名，我们后面会讨论。

  像等号(=)和加号(+)修饰符一样，符号(&)也只能用于对Output操作表达式的修饰。当使用它进行修饰时，等于向GCC声明："GCC不得为任何Input操作表达式分配与此Output操作表达式相同的寄存器"。其原因是&修饰符意味着被其修饰的Output操作表达式要在所有的Input操作表达式被输入前输出。我们看下面这个例子：

  int main(int __argc, char* __argv[])

  {

  int __in1 = 8, __in2 = 4, __out = 3;

  __asm__ ("popl %0 \n\t"

  "movl %1, %%esi \n\t"

  "movl %2, %%edi \n\t"

  : "=a"(__out)

&nb