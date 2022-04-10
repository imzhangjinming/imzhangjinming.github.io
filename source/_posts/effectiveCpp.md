---
title: Effective Cpp -- 读书摘抄
categories:
  - notes
date: 2021-07-24 15:42:48
tags:
---

{% asset_img 1.jpg %}

<!-- more -->
# 1 让自己习惯C++  
---

## 条款01：视C++为一个语言联邦  
## 条款02：尽量以 `const` `enum` `inline` 替换 `#define`  
* 这个条款的意思其实是用编译器代替预处理器  
* class专属常量：为了将常量的作用域限制于class内部，我们必须让它成为class的一个成员；而为了确保此常量至多只有一个实体，必须定义为 `static` 成员  
* `the enum hack` 欺骗编译器的把戏，将枚举类型的数值当作 `int` 使用，可以用来实现类内初始化；这个技术也是 template metaprogramming（模板元编程）  
* 以 template inline 函数代替 macro（宏）  
## 条款03：尽可能使用 `const`  
* 如果关键字 `const` 出现在星号的左边，表示被指物是常量，出现在星号右边，说明指针本身是常量  
* 迭代器与 `const` 的组合  
	`const std::vector<int>::iterator iter = vec.begin();// iter 的作用像一个 T* const`  
	`std::vector<int>::const_iterator cIter = vec.begin();// cIter 的作用像是一个 T const *`  
* `const` 实施于成员函数的目的是为了确认该成员函数可用于 `const` 对象身上  
* 一个事实：两个成员函数如果只是常量性不同可以被重载  
* `mutable` 能释放掉 `non-static` 成员变量的 `bitwise constness` 约束  

## 条款04：确定对象被使用前已被初始化  
* 永远在使用对象之前将它初始化  
* 函数内的 `static` 对象称为 `local static` 对象，其他 `static` 对象称为 `non-local static` 对象  
* C++对“定义于不同编译单元内的 `non-local static` 对象”的初始化次序并无明确定义  
* `Singleton`设计模式  
	* 这个手法的基础在于:C++保证，函数内的 `local static`对象该函数被调用期间或者首次遇上该对象之定义式时被初始化。  
	* 运用 `reference-returning` 函数防止“初始化次序问题”  

# 2 构造/析构/赋值运算  
---
## 条款05：了解C++默默编写并调用哪些函数  
## 条款06：若不想使用编译器自动生成的函数，就该明确拒绝  
* 为了阻止对对象的拷贝，需要自己声明拷贝构造函数和拷贝赋值运算符，不定义，并把它们声明为 `private`  
* 为了将调用不能拷贝类对象的报错移至编译期，可以定义一个基类，这个基类没有数据成员，拷贝构造函数和拷贝赋值运算符被声明为 `private` ,让不能拷贝的类继承这个基类，因为编译器在生成派生类的构造函数和拷贝赋值运算符时会尝试调用基类的对应”兄弟“，而由于我们已经把基类的对应函数声明为 `private` ，所以编译器会拒绝访问  

## 条款07：为多态基类声明 `virtual` 析构函数  
* 任何 `class` 只要带有 `virtual` 函数都几乎确定应该也有一个 `virtual` 析构函数，因为这表明这个类几乎肯定会被其他类继承  
* 如何决定调用哪个虚函数？这取决于对象携带的信息，这个所谓的信息通常是一个 `vptr(virtual table pointer)` ，这个指针指向一个由**函数指针**构成的数组（`vtbl` virtual table）；每一个带有 `virtual` 函数的 `class` 都有一个虚函数表。当对象调用某一 `virtual` 函数，实际被调用的函数取决于该对象的 `vptr` 所指的那个 `vtbl` ，编译器在其中寻找适当的函数指针  
* 标准 `string` 不含任何 `virtual` 函数，不含 `virtual` 析构函数，所以继承 `string` 类可能会出现局部释放的问题  

## 条款08：别让异常逃离析构函数  
* C++ 不喜欢析构函数抛出异常  

## 条款09：绝不在构造和析构过程中调用 `virtual` 函数  
* base class 构造期间 `virtual` 函数绝不会下降到 derived classes 阶层。
* 在 base class 构造期间，`virtual` 函数不是 `virtual` 函数 :laughing:  

## 条款10：令 `operator=` 返回一个 reference to `*this`  
* 为了实现“连锁赋值”，赋值操作符必须返回一个 reference 指向操作符的左侧实参  

## 条款11：在 `operator=` 中处理“自我赋值”  
* 一个“异常安全”并且能够处理“自我赋值”的 `operator=` 的版本  
	```C++
	class Wiget{
		···
		void swap(Wiget& rhs);	//交换 *this 和 rhs 的数据
		...
	};
	Wiget& Wiget::operator=(const Wiget& rhs)
	{
		Wiget temp(rhs);
		swap(temp);
		return *this;
	}
	```
	异常安全方面的知识我了解不多  

## 条款12：复制对象时勿忘其每一个成分  
* *copy* 构造函数和 *copy assignment* 操作符  
* 当编写一个 *copying* 函数，确保（1）复制所有 local 成员变量；（2）调用所有 base class 内的适当的 *copying* 函数。  
* 如果发现 *copy* 构造函数和 *copy assignment* 操作符有相近的代码，消除重复代码的做法是，建立一个新的成员函数给两者调用。这样的函数往往是 `private` 而且常常被命名为 `init`  

# 3 资源管理  
---
## 条款13：以对象管理资源  
* 主要想法：把资源放进对象内，我们便可以依赖C++的“析构函数自动调用机制”确保资源被释放  
* 以对象管理资源的两个关键想法：
	* 获得资源后**立刻**放进管理对象  
	* 管理对象运用析构函数确保资源被释放  
* **RCSP** reference-counting smart pointer  

## 条款14：在资源管理类中小心 *copying* 行为  
## 条款15：在资源管理类中提供对原始资源的访问  
## 条款16： 成对使用 `new` 和 `delete` 时要采取相同形式  
* 使用 `new` 时，先分配内存，然后在这块内存上调用一次或多次构造函数  
* 使用 `delete` 时，先调用一次或多次析构函数，然后再释放内存  
* 使用 `new` 分配内存则使用 `delete` 释放内存；使用 `new[]` 分配内存则使用 `delete[]` 释放内存  
## 条款17：以独立语句将 new*ed* 对象置入智能指针  

# 4 设计与声明  
---
## 条款18：让接口容易被正确使用，不易被误用  
* 理想上，如果客户企图使用某个接口而却没有获得他所期望的行为，这个代码**不该通过编译**；如果代码通过了编译，它的行为就是客户所想要的  
* （接口）不一致性对开发人员造成的心理和精神上的摩擦与争执，没有任何一个IDE可以完全抹除:satisfied:  
* 任何接口如果要求客户必须记得做某些事情，就是有着“不正确使用”的倾向  
## 条款19：设计 `class` 犹如设计 `type`  
* 这个条款通过提出一系列问题来表达作者的观点：
	* 新 `type` 的对象应该如何被创建和销毁？  
	* 对象的初始化和对象的赋值该有什么样的差别？
	* 新 `type` 的对象如果被 *passed by value* ，意味着什么？意味着需要考虑*copy*构造函数的相关事宜  
	* 什么是新 `type` 的合法值？
	* 你的新 `type` 需要配合某个继承图系吗？  
	* 你的新 `type` 需要什么样的转换？  
	* 什么样的操作符和函数对此新 `type` 而言是合理的？  
	* 什么样的标准函数应该驳回？（拷贝构造函数，拷贝赋值运算符等等）  
	* 谁该取用新 `type` 的成员？（`public` `pretected` `private` `friend` 等）
	* 什么是新 `type` 的“未声明接口”？
	* 你的新 `type` 有多么一般化？  
	* 你真的需要一个新 `type` 吗？
## 条款20：宁以 pass-by-reference-to-const 替换 pass-by-value  
* 以 by reference 方式传递参数可以提高程序的**效率**  
* 以 by reference 方式传递参数也可以避免 *slicing* 问题  
* **内置类型**的 padd by value 往往比 pass by reference 的效率高些:open_mouth:  
## 条款21：必须返回对象时，别妄想返回其 reference  
* 别返回 local 对象的 引用 或 指针  
## 条款22：将成员变量声明为 `private`  
* 将成员变量隐藏在函数接口的背后，可以为“所有可能的实现”提供弹性  
* `protected` 成员变量封装性并不比 `public` 强多少  
## 条款23：宁以 non-member non-friend 替换 member 函数  
## 条款24：若所有参数皆需类型转换，请为此采用 non-member 函数  
* 如果需要为某个函数的所有参数（包括被 `this` 指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个 non-member , 这个条款可能在重载运算符的时候需要考虑  
## 条款25：考虑写出一个不抛出异常的 `swap` 函数  
* `swap` 的典型实现  
  ```C++
  namespace std{
	  template<typename T>
	  void swap(T& a,T& b)
	  {
		  T temp(a);
		  a=b;
		  b=temp;
	  }
  }
  ```
* 然而上面这种实现方式可能会降低效率，一个典型的例子就是 swap “以指针指向一个对象，内含真正数据”的类型的对象。  
* pimpl "*pointer to impletation*"  
* `swap` 的另一种实现  
  ```C++
  class WidgetImpl{
	  public:
	  ...
	  private:
	  int a,b,c;
	  std::vector<double> v;
	  ...
  };

  class Widget{
	  public:
	  ...
	  void swap(Widget& other)
	  {
		  using std::swap;
		  swap(pImpl,other.pImpl);
	  }
	  ...
	  private:
	  WidgetImpl* pImpl;
  };

  namespace std{
	  template<>
	  void swap<Widget>(Widget& a,Widget& b)
	  {
		  a.swap(b);
	  }
  }
  ```
  这里为 `Widget` 类定义了一个 `swap` 成员函数，然后将标准命名空间里的 `swap` 函数全特化，并在其中调用类内版本的 `swap`，这样用户对 `Widget` 对象调用 `swap` 时，使用的就是我们在类内定义的效率较高的版本

* 自定义的 `swap` 版本应该在提高效率的同时不抛出异常  
* 调用 `swap` 时应针对 `std::swap` 使用 `using` 声明式，然后调用 `swap` 并且不带任何命名空间资格修饰，让编译器自己查找应该调用哪个版本的 `swap`  

# 5 实现
## 条款26：尽可能延后变量定义式的出现时间  
* 这个条款和 C 语言的书写风格完全相反，C 语言建议将所有变量统一声明于代码块开始，而C++不是这样，为什么呢？因为C++有异常，这会导致程序提前结束而定义的变量并没有被使用，那样的话，构造变量需要的那些时间不就白白浪费了嘛  
## 条款27：尽量少做转型动作  
* 旧式转型 （old-style casts）
  ```C++
  (T)expression;
  T(expression);
  ```
* 接下来我要摘抄一大段  
	>C++ 还提供四种新式转型
	```C++
	const_cast<T>(expression);
	dynamic_cast<T>(expression);
	reinterpret_cast<T>(expression);
	static_cast<T>(expression);
	```
	>* const_cast 通常被用来将对象的常量性转除。它也是**唯一**有此能力的C++-style转型操作
	>* dynamic_cast 主要用来执行“安全向下转型”，也就是用来决定某对象是否归属继承体系中的某个类型
	>* reinterpret_cast 意图执行低级转型,实际动作取决于编译器，这也就说明它不可移植  
	>* static_cast 用来强迫隐式转换  
* 之所以需要 dynamic_cast ,通常是因为你想在一个你认定为 derived class 对象身上执行 derived class 操作函数，但你的手上却只有一个指向 base 的指针或引用  
## 条款28：避免返回 handles 指向对象内部成分  
## 条款29：为“异常安全”而努力是值得的  
* copy and swap 策略：为打算修改的对象做出一个副本，在副本上做改变，原对象仍然保持不变。所有改变都成功后，再将修改过的那个副本和原对象在一个不抛出异常的操作中置换（swap）  
* 如果系统内有一个函数不具备异常安全性，整个系统就不具备异常安全性  
* 写异常安全的代码时，首先是“以对象管理资源”防止资源泄漏。然后挑选异常安全保证的标准，并应用到每个函数中去  
## 条款30：透彻了解 inlining 的里里外外  
* `inline` 只是对编译器的一个申请，**不是强制指令**  
* 一开始先不要将任何函数声明为 `inline` ，或至少将 `inlining` 施行范围局限在那些**一定成为** `inline` 或“十分平淡无奇”的函数身上  
* 将大多数 `inlining` 限制在小型、被频繁调用的函数身上  
## 条款31：将文件间的编译依存关系降至最低  
* 降低文件间的编译依存关系的关键在于以“声明的依存性”替换“定义的依存性”：现实中让头文件尽可能自我满足，万一做不到，则让它与其他文件内的声明式相依。具体而言是以下的策略：  
	* 如果使用 **对象的引用或指针** 可以完成任务，就不要使用 **对象**，因为声明对象的引用或指针只需要类的声明（不需要定义，如果使用对象则需要定义），这就让此处的代码不再依赖对象的定义，也就是以“声明依存性”替换了“定义的依存性”  
	* 如果能够，尽量以 `class` 声明式替换 `class` 定义式  
	```C++
	class Date;							//只提供类的声明
	Date today();						//没问题，这里暂时不需要类的定义
	void clearAppointments(Date d);	//Date 的定义式
	```
	* 为声明式和定义式提供不同的头文件  
* C++提供关键字 `export` ，允许将 `template` 声明式和 `template` 定义式分割于不同的头文件内（支持这个关键字的编译器还很少）  
* 两种方式：handle classes 和 interface classes  
	* handle classes 是在一个类中实现，在另一个类中保存指向实现类对象的指针的做法  
	* interface classes 是定义一个接口类，并把所有接口声明为虚函数，用实现类继承接口类并提供接口的定义
	以上两种方法都能降低文件间的编译依存性，也即以“声明依存性”替换“定义依存性”  

# 6 继承与面向对象设计  
## 条款32：确定你的 `public` 继承塑模出 **is-a** 关系  
* public inheritance 意味 "is-a" 的关系  
## 条款33：避免遮掩继承而来的名称  
* 派生类的作用域嵌套在基类作用域中  
* 如果继承 base class 并加上重载函数，而又希望重新定义或覆写其中一部分，那么必须为那些原本会被遮掩的每个名称引入一个 using 声明，否则某些希望继承的名称会被遮掩  
## 条款34：区分接口继承和实现继承  
* public 继承概念由两部分组成：函数接口继承和函数实现继承  
* 声明一个 pure virtual 函数的目的是为了让 derived classes 只继承函数接口  
* impure virtual 函数要求派生类继承其接口，但是它也提供一份定义，派生类可以选择继承这个实现，也可以选择重写它  
* 声明一个 impure virtual 函数的目的是让 derived classes 继承该函数的接口和缺省实现  
* 纯虚函数可以提供缺省实现，但是在派生类中必须显式调用，如 `BASECLASS::func();`  
* 声明 non-virtual 函数的目的是为了令 derived classes 继承函数的接口及一份强制性实现  
## 条款35：考虑 `virtual` 函数以外的其他选择  
* 借由 Non-Virtual Interface 手法实现 Template Method 模式
	这一设计的基本想法是，将 `virtual` 函数声明为 `private` ，通过 public non-virtual 成员函数间接调用 `private virtual` 函数，称为 non-virtual interface(NVI) 手法。它是所谓 **Template Method**设计模式的一个独特表现形式  
	其实就是用 public non-virtual 成员函数包装了 `virtual` 函数  
* 借由 Function Pointers 实现 **Strategy**模式  
* 借由 `tr1::function` 完成 **Strategy**模式  
* 因为不懂设计模式，这个条款的内容没有看得很明白  
## 条款36：绝不重新定义继承而来的 non-virtual 函数  
* 重新定义一个继承而来的 non-virtual 函数永远是错误的
## 条款37：绝不重新定义继承而来的缺省参数值  
* `virtual` 函数动态绑定，缺省参数值静态绑定  
* 对象的所谓静态类型，就是它在程序中**被声明时**所采用的类型  
* 对象的所谓动态类型则是指“目前所指对象”的类型  
* 如果重新定义了继承而来的缺省参数值，那么当我们使用基类指针调用派生类成员函数时，会发生怪异的事情，那个调用将使用派生类的函数，基类的缺省参数值！因为缺省参数值是静态绑定，它在声明指针的时候就已经绑定了，当使用此指针调用派生类成员函数时，就使用基类的缺省参数值，怪事！  
## 条款38：通过复合(composition)塑模出 has-a 或“根据某物实现出”  
## 条款39：明智而审慎地使用 `private` 继承  
* 如果 classes 之间的关系是 `private` 继承，那么编译器不会自动将一个 derived class 对象转换为一个 base class 对象  
* `private` 继承纯粹只是一种实现技术，并不表示类之间有什么观念上的关系  
* 尽可能使用复合，必要时才使用 `private` 继承  
## 条款40：明智而审慎地使用多重继承  
* C++在解析重载函数时，首先确认这个函数对此调用之言是最佳匹配，然后再看取用性（`private` `public` 等限制）  

# 7 模板与泛型编程  
## 条款41：了解隐式接口和编译期多态  
* 在面向对象编程中，重要的是显式接口和运行期多态；而在模板和泛型编程中重要的是隐式接口和**编译期多态**  
* 函数的*签名式*：函数名称、参数类型、返回类型  
## 条款42：了解 `typename` 的双重定义  
* 想要在template 中指涉一个嵌套从属类型名称，就必须在紧邻它的前一个位置放上关键字 `typename`，`calss` 却没有这个功能  
* `std::iterator_traits<IterT>::value_type` 表示 `IterT` 之对象所指之物的类型  
## 条款43：学习处理模板化基类内的名称  
## 条款44：将与参数无关的代码抽离 `templates` 
## 条款45：运用成员函数模板接受所有兼容类型  
* 如果声明了 member templates 用于 泛化copy构造 或 泛化assignment操作，还是需要声明正常的copy构造函数和 copy assignment操作符  
## 条款46：需要类型转换时请为模板定义非成员函数  
* 在 template 实参推导过程中从不将隐式类型转换函数纳入考虑  
## 条款47： 请使用 traits classes 表现类型信息  
## 条款48： 认识 template 元编程  
* template metaprograms 执行于C++编译期  

# 8 定制 `new` 和 `delete`  
## 条款49：了解 new-handler 的行为  
* 当 `operator new` 抛出异常以反映一个未获满足的内存需求之前，它会先调用一个客户指定的错误处理函数，即所谓的 new-handler  
* 设计一个良好的 new-handler 函数必须做以下事情  
	* 让更多内存可被使用  
	* 安装另一个 new-handler  
	* 卸载 new-handler  
	* 抛出 `bad_alloc` 的异常  
	* 不返回，通常调用 `abort` 或 `exit`  
## 条款50：了解 `new` 和 `delete` 的合理替换时机  
## 条款51：编写 `new` 和 `delete` 时需固守成规  
* operator new 应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就应该调用 new-handler 。它也应该有能力处理 0 bytes 申请。
* operator delete 应该在收到 null 指针时不做任何事。 
## 条款52：写了 placement new 也要写 placement delete  
* 一般性术语 “placement new” 意味除了 `size_t` 带任意额外参数的 `new`  
* `Widget* pw = new Widget;` 共有两个函数被调用：一个是用于分配内存的 opertor `new` ,一个是 Widget 的*default*构造函数  

# 9 杂项讨论  
## 条款53：不要轻易忽略编译器的警告（warnings）  
## 条款54： 让自己熟悉包括 TR1 在内的标准程序库  
* C++98 列入的 C++ 标准程序库：
	* STL  
	* iostreams  
	* 国际化支持  
	* 数值处理  
	* 异常继承体系  
	* C89 标准程序库  
* TR1 的一些组件  
	* 智能指针  
	* tr1::function  
	* tr1::bind  
	* Hash tables  
	* Regular expressions  
	* tuples  
	* tr1::array  
	* tr1::mem_fn  
	* tr1::reference_wrapper  
	* 随机数生成工具  
	* 数学特殊函数  
	* C99 兼容扩充  
	* Type traits  
	* tr1::result_of  
## 条款55：让自己熟悉Boost  
