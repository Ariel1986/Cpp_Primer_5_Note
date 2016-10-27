#### 显式转换
+ 命名的强制类型转换形式：
 * cast_name<type>(expression); //如果type是引用类型，则结果是左值
 * cast_name是static_cast, dynamic_cast, const_caset, reinterpret_cast中的一种。
 
+ static_cast
 * 任何具有明确定义的类型转换，只要不包含底层const，都可以使用static_cast.
```
double slope = static_cast<double>(j); 
```
 * 当需要把一个较大的算术类型赋值给较小的类型时，static_cast非常有用。
 * static_cast对于编译器无法自动执行的类型转换也非常有用。例如可以用static_cast找回存在于void*指针中的值。此时必须保证转换后所得的类型就是指针所指的类型。类型一旦不符，将产生为定义的后果。
```
void *p = &d;   //正确：任何非常量对象的地址都能存入void*
double *dp = static_cast<double*>(p);  //正确：将void*转换回初始的指针类型，如果将void*转换成其他的指针类型，结果是为定义的
```
+ const_cast
 * const_cast只能改变运算对象的底层const。
 ```
 const char* pc;
 char *p = const_cast<char*>(pc);  //正确，但是通过p写值是未定义的行为
 ```
 * 对于将常量对象转换成非常量对象的行为，称为“去掉const性质（cast away the const）”。一旦去掉了某个对象的const性质，编译器将不再阻止对该对象进行写操作。如果对象本事不是一个常量，使用强制类型转换获得写权限是合法的行为。如果对象是一个常量，再使用const_cast执行写操作会产生为定义的后果。
 * 只有const_cast能改变表达式的常量属性，使用其他形式的命名强制转换改变表达式的常量属性都将引发编译错误。同样，也不能用const_cast改变表达式的类型。
```
const char *cp;
char *q = static_cast<char*>(cp);   //错误：static_cast不能转换掉const性质
static_cast<string>(cp);            //正确：字符串字面值转换成string类型
const_cast<string>(cp);             //错误：const_cast只改变常量属性
```
 * const_cast常常用于有函数重载的上下文中。

+ reinterpret_cast
 * reinterpret_cast通常为运算对象的位模式提供较低层次上的重新解释。如下：
```
 int *ip;
 char *pc = reinterpret_cast<char*>(ip);
 
 //必须时刻牢记pc所指的真实对象是一个int而非字符，如果把pc当成普通的字符指针使用就可能在运行时发生错误。
 string str(pc);  //可能导致异常的运行时行为
```

 * 使用reniterpret_cast非常危险，上面的例子就证明了这一点。其中的关键问题是类型改变了，但是编译器没有给出任何警告或者错误的提示信息。当用int的地址初始化pc时，由于显式地声称这种转换合法，所以编译器不会发出任何警告或错误信息。但是在使用pc时会认定它的值时是char*类型，编译器没法知道它实际存放的是指向int的指针。最终的结果是，在上面的例子中虽然用pc初始化str没什么实际意义，甚至还可能引发更糟糕的后果，但仅从语法上而言这种操作无可指摘。查找这类问题的原因非常困难，如果将ip强制转换成pc的语句和用pc初始化string对象的语句分属不同文件更是如此。
 * !! reniterpret_cast本质傻姑娘依赖于机器。要想安全地使用reniterpret_cast必须对涉及的类型和编译器实现转换的过程都非常了解。
+ 建议：避免强制类型转换
  * 强制类型转换干扰了正常的类型检查，因此强烈建议避免使用强制类型转换。
  * 该建议对reniterpret_cast尤其适用，因为此类类型转换总是充满了风险。
  * 在有重载函数的上下文中使用const_cast无可厚非；但在其他情况下使用const_cast也就意味着程序存在某种设计缺陷。
  * 其他强制类型转换，比如static_cast和dynamic_cast，都不应该频繁使用。每次书写了一条强制类型转换语句，都应该反复斟酌能否以其他方式实现相同的目标。就算实在无法避免，也应该尽量限制类型转换值的作用域，并且记录对相关类型的所有假定，这样就可以减少错误发生的机会。
  
#### 旧式的强制类型转换
* 在早期版本的C＋＋语言中，显式地进行强制类型转换包含两种形式：
```
type (expr);  //函数形式的强制类型转换
(type) expr;  // C语言风格的强制类型转换 
```
* 根据所涉及的类型不同，旧式的强制类型转换分别具有与const_cast,static_cast或reinterpret_cast相似的行为。当在某处执行旧式的强制类型转换时，如果换成const_cast和static_cast也合法，则其行为与对应的命名转换一致。如果替换后不合法，则旧式强制类型转换执行与reniterpret_cast类似的功能：
```
char *pc = (char*) ip; //ip是指向整数的指针，效果与使用reniterpret_cast一样。
```
 