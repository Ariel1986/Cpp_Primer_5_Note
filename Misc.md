### 类型转换

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
 
### 含有可变参数的函数
+ 为了编写能处理不同数量实参的函数，C＋＋11新标准提供了两种主要的方法
+ 如果所有的实参类型相同，可以传递一个名为initializer_list的标准库类型;
  * 如果函数的实参数量未知但是全部实参的类型都相同，可以用initializer_list类型的形參。
  * initializer_list是一种标准库类型，可以用于某种特定类型的值的数组。
  * 和vector不同的是，initializer_list对象中的元素永远都是常量值，无法改变initializer_list对象中元素的值。
  * 如果向initializer_list形參中传递一个值的序列，必须把序列放在一对花括号内：
 ```
 if(expected != actual)
  error_msg({"functionX", expected, actual是string
 else
  error_msg({"functionX", "okay"});
 ```
  * 含有initializer_list形參的函数也可以同时拥有其他形參。
 ```
  void error_msg(ErrCode e, initializer_list<string> il)
  {
   ...
  }
 ```
| 操作                               | 含义          
| --------------------------------- |--------------:| 
| initializer_list<T> lst           | 默认初始化；T类型元素的空列表 | 
| initializer_list<T> lst{a,b,c...} | lst的元素数量和初始值一样多；lst的元素是对应初始值的副本；列表中的元素是const      |  
| lst2(lst) 或者 lst2 = lst          | 拷贝或赋值一个initializer_list对象不会拷贝列表中的元素；拷贝后，原始列表和副本元素共享     | 
| lst.size()                        | 列表中的元素数量     |  
| lst.begin()                       | 返回指向lst中首元素的指针     |  
| lst.end()                         | 返回指向lst中尾元素下一个位置的指针     | 

+ 如果实参的类型不同，可以编写可变参数模板;
+ 省略符形参：仅仅用于c和c＋＋通用的类型。大多数类型的对象在传递给省略符形参时都无法正确拷贝。
  * 该方法是为了便于c＋＋程序访问某些特殊的c代码而设置的，这些代码使用了名为varargs的c标准库功能。
  * 省略符形參只能出现在形參列表的最后一个位置，有如下两种形式：
```
  void foo(parm_list, ...);  //指定了foo函数的部分形參的类型，对于这些形參的实參会执行正常的类型检查；形參声明后面的逗号是可选的
  void foo(...);             //省略符形參所对应的实参无需类型检查。
```
### 类的静态成员
+ 和其他成员一样，静态成员可以是public的或private的。静态数据成员的类型可以是常量，引用，指针，类类型等。
+ 类的静态成员存在于任何对象之外，对象中不包含任何与静态数据有关的数据。
+ 类的静态成员函数也不与任何对象绑定在一起，他们不包含this指针。作为结果，静态成员函数不能声明成const的，而且也不能在static函数体内使用this指针。这一限制既适用于this的显式使用，也对调用非静态成员的隐式使用有效。
#### 使用类的静态成员
+ 使用作用域运算符直接访问静态成员：
```
double r;
r = Account::rate();  //使用作用域运算符访问静态成员
```
+ 使用类的对象，引用或者指针访问静态成员
```
Account ac1;
Account *ac2 = &ac1;
r = ac1.rate();   // 通过Account的对象或引用
r = ac2->rate();  // 通过指向Account对象的指针
```
+ 成员函数不用通过作用域运算符就能直接使用静态成员
```
class Account {
public:
 void calculate() { amount += amount * interestRate; }
private:
 static double interestRate;
 //...
};
```
#### 定义静态成员
+ 静态成员函数：和其他成员函数一样，既可以在类的内部也可以在类的外部定义静态成员函数。当在类的外部定义静态成员时，不能重复static关键字，该关键字只出现在类内部的声明语句
```
//在类的外部定义类的静态函数成员，必须指明成员所属的类名。static关键字只出现在类内部的声明语句中
void Account::rate(double newRate)
{
 interestRate = newRate;
}
```
+ 静态数据成员：因为静态数据成员不属于类的任何一个对象，所以它们不是在创建类的对象时被定义的，也就是说它们不是由构造函数初始化的。
+ 一般，不在类的内部初始化静态成员。相反，必须在类的外部定义和初始化每个静态成员。和其他对象一样，一个静态数据成员只能定义一次。
+ 类似于全局变量，静态数据成员定义在任何函数之外。因此一旦它被定义，就将一直存在于程序的整个生命周期中
+ 定义静态数据成员的方式类似于在类的外部定义成员函数，需要指定对象的类型名，类名，作用域运算符以及成员自己的名字：
```
double Account::interestRate = initRate(); //定义并初始化一个静态成员
```
+ 想要确保对象只定义一次，最好的办法就是把静态数据成员的定义与其它非内联函数的定义放在同一个文件中。

#### 静态成员的类内初始化
+ 通常情况下，类的静态成员不应该在类的内部初始化。然而，可以为静态成员提供const整数类型的类内初始值，不过要求静态成员必须是字面值常量类型的constexpr。
+ 初始值必须是常量表达式，因为这些成员本身就是常量表达式，所以它们能用在所有适用于常量表达式的地方
```
class Account {
public:
 static double rate() {return interestRate;}
 static void rate(double);
private:
 static constexpr int period = 30;  //period是常量表达式
 double daily[period];
};

constexpr int Accout::period; //初始值在类的定义内提供
```
+ 即使一个常量静态数据成员在类内部被初始化了，通常情况下页应该在类的外部定义一下该成员。

#### 静态成员能用于某些场景，而普通成员不能
+ 静态成员可以是不完全类型：静态数据成员的类型可以就是它所属的类类型。而非静态数据成员则受到限制，只能声明成它所属类的指针或引用
+ 可以使用静态成员作为默认实参，非静态数据成员不能作为默认实参，因为它的值本身属于对象的一部分，这么做的结果是无法真正提供一个对象以便从中获取成员的值，最终将引发错误。
```
class Screen {
public:
 Screen& clear(char = bkground);
private:
 static const char bkgroud;
};
```

### 类定义的各种关键字
+ inline:可以在类的内部把inline作为声明的一部分显式地声明成员函数，也可以在类的外部用inline关键字修饰函数的定义。可以同时在声明和定义的地方说明inline，但是最好只在类外部定义的地方说明inline，这样做使类更容易理解。
+ explicit:只能在类内声明构造函数时使用explicit关键字，在类外部定义时不应重复。
+ static: 当在类的外部定义静态成员时，不能重复static关键字，该关键字只出现在类内部的声明语句
+ =default: 位于类内部默认构造函数声明语句的参数列表之后，要求编译器生成构造函数。
+ 关键字Virtual只能出现在类内部的声明语句之前而不能用于类外部的函数定义。如果基类把一个函数声明成虚函数，则该函数在派生类中隐式地也是虚函数。
+ C++11新标准允许派生类显式地注明它使用某个成员函数覆盖了它继承的虚函数。具体做法是在形参后面、或者在const成员函数的const关键字后面，或者在引用成员函数的引用限定符后面添加一个关键字override。
