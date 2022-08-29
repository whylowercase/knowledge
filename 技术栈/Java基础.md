## hashcode和equals

hashcode()的作用是获取哈希码，实际上是一个int整数，作用是确定该对象在哈希表中的索引位置。hascode是Object类中，意味着java中的任何类都有此函数

equals默认使用`==`比较调用对象和参数对象在内存中的地址值，只要两个对象的引用指向的不是同一个对象，那么返回值一定是false。

一般来说这两个方法并没有任何的联系。但在散列表中，我们基础hashcode 算出元素要插入的数组下标，如果下标下已经有值，那么就说明发生了哈希冲突，此时我们就需要再使用equals进行下一步的比对

- 当hashcode不同时，两个对象肯定不同
- 当hashcode相同时，需要进一步调用equals对比
- 当equals返回true时，两个对象的hashcode肯定相同

## final关键字

final修饰的方法不能被重写

final修饰的类不能被继承

final修饰的变量不能被再赋值

- 基本数据类型：值不可变
- 引用数据类型：应用地址不可变

## Java中的异常体系

所有异常都来自顶级父类Throwable

- Exception：不会导致程序停止。分为RunTimeException和CheckedException，运行时异常发生在程序运行过程中，会导致程序当前线程执行失败。检查异常发生在程序编译过程中，ide会报错
- Error：程序无法处理的错误，一旦出现，程序就会被迫停止

