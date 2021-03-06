#'static'作用
有时希望函数中的局部变量的值在函数调用结束后不消失而继续保留原值，即其占用的存储单元不释放，在下一次再调用的时候该变量已经有值。这时就应该指定该局部变量为静态变量，用关键字 static 进行声明。

##修饰局部变量


让局部变量只初始化一次
局部变量在程序中只有一份内存
对局部变量用static声明，把它分配在静态存储区，该变量在整个程序执行期间不释放，其所分配的空间始终存在
并不会改变局部变量的作用域，仅仅是改变了局部变量的生命周期（只到程序结束，这个局部变量才会销毁）


##修饰全局变量


对全局变量用static声明，则该变量的作用域只限于本文件模块(全局变量的作用域仅限于当前文件，即被声明的文件中)

例如单例模式中使用的 static。

##内存分配（待补充）

可编程内存在基本上分为这样的几大部分：静态存储区、堆区和栈区。他们的功能不同，对他们使用方式也就不同。
静态存储区：内存在程序编译的时候就已经分配好，这块内存在程序的整个运行期间都存在。它主要存放静态数据、全局数据和常量。
栈区：在执行函数时，函数内局部变量的存储单元都可以在栈上创建，函数执行结束时这些存储单元自动被释放。栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
堆区：亦称动态内存分配。程序在运行的时候用malloc或new申请任意大小的内存，程序员自己负责在适当的时候用free或delete释放内存。动态内存的生存期可以由我们决定，如果我们不释放内存，程序将在最后才释放掉动态内存。 但是，良好的编程习惯是：如果某动态内存不再使用，需要将其释放掉，否则，我们认为发生了内存泄漏现象。
代码区：存放函数体的二进制代码
文字常量区:—常量字符串就是放在这里的。程序结束后由系统释放

简单描述：

栈区
内存管理由系统控制，存储的为非静态的局部变量，例如：函数参数，在函数中生命的对象的指针等。当系统的栈区大小不够分配时， 系统会提示栈溢出。
堆区
内存管理由程序控制，存储的为malloc , new ,alloc出来的对象。
如果程序没有控制释放，那么在程序结束时，由系统释放。但在程序运行过程中，会出现内存泄露、内存溢出问题。
分配方式 类似于链表。
全局存储区（静态存储区）
全局变量、静态变量会存储在此区域。事实上全局变量也是静态的，因此，也叫全局静态存储区。
存储方式： 初始化的全局变量跟静态变量放在一片区域，未初始化的全局变量与静态变量放在相邻的另一片区域。
程序结束后由系统释放。
文字常量区
在程序中使用的常量存储在此区域。程序结束后，由系统释放。在程序中使用的常量，都会到文字常量区获取。
程序代码区
存放函数体的二进制代码。
运行程序就是执行代码，代码要执行就要加载进内存。

