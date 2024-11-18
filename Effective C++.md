# 条款01：视 C++为一个语言联邦

# 条款02：尽量以const, enum, inline 替换 #define

**对于单纯常量，最好以const对象或enums替换#defines：**

1. ```c++
   #define ASPECT_RATIO 1.653：
   ```

   记号名称 ASPECT_RATIO也许从未被编译器看见；也许在编译器开始处理源码之前它就被预处理器移走了。于是记号名称ASPECTRATIO有可能没进入记号表(symboltable)内。于是当你运用此常量但获得一个编译错误信息时，可能会带来困惑，因为这个错误信息也许会提到1.653而不是ASPECTRATIO。如果ASPECT RATIO 被定义在一个非你所写的头文件内，你肯定对1.653 以及它来自何处毫无概念，于是你将因为追踪它而浪费时间。这个问题也可能出现在记号式调试器(symbolic debugger)中,原因相同:你所使用的名称可能并未进入记号表(symboltable)

   解决之道是以一个常量替换上述的宏(#define):

   ```c++
   const double AspectRatio=1.653;//大写名称通常用于宏，因此这里改变名称写法
   ```

   作为一个语言常量，AspectRatio肯定会被编译器看到，当然就会进入记号表内。此外对浮点常量(floatingpointconstant，就像本例)而言，使用常量可能比使用#define 导致较小量的码，因为预处理器“自目地将宏名称ASPECTRATIO替换为 1.653”可能导致目标码(objectcode)出现多份1.653,若改用常量 AspectRatio绝不会出现相同情况。

   当我们以常量替换#defines，有两种特殊情况：

   1. 定义常量指针：

      由于常量定义式通常被放在头文件内(以便被不同的源码含入)，因此有必要将指针(而不只是指针所指之物)声明为const。例如若要在头文件内定义一个常量的(不变的)char*-based字符串，你必须写const两次:

      ```c++
      const char*const authorName="Scott Meyers";
      ```

      string对象通常比其前辈char*-based合宜，所以上述的authorName往往定义成这样更好些:

      ```c++
      const std::string authorName("Scott Meyers");
      ```

   2. class专属常量：

      为了将常量的作用域(scope)限制于class内，你必须让它成为class的一个成员(member);而为确保此常量至多只有一份实体，你必须让它成为一个 static 成员:

      ```c++
      class GamePlayer{
      	private:
          	static const int NumTurns=5;//常量声明式
      		int scores[NumTurns];//使用该常量
      };
      ```

      然而你所看到的是NumTurns的声明式而非定义式。通常C++要求你对你所使用的任何东西提供一个定义式,但如果它是个class 专属常量又是 static 且为整数类型(integraltype，例如 ints,chars,bools)，则需特殊处理。只要不取它们的地址你可以声明并使用它们而无须提供定义式。但如果你取某个class专属常量的地址或纵使你不取其地址而你的编译器却(不正确地)坚持要看到一个定义式，你就必须另外提供定义式如下:

      const int GamePlayer::NumTurns;//NumTurns的定义

      请把这个式子放进一-个实现文件而非头文件。由于cass常量已在声明时获得初值(例如先前声明Numrurns时为它设初值S)，因此定义时不可以再设初值。请注意，我们无法利用#define创建一个class专属常量，因为#defines并不重视作用域(scope)。一旦宏被定义，它就在其后的编译过程中有效(除非在某处被#undef)。这意味#defines不仅不能够用来定义 class 专属常量也不能够提供任何封装性，也就是说没有所谓private #define这样的东西。而当然const成员变量是可以被封装的，Numrurns就是。

2. 当你在class 编译期间需要一个 class 常量值，例如在上述的GamePlayer::scores的数组声明式中(是的，编译器坚持必须在编译期间知道数组的大小)。这时候万一你的编译器(错误地)不允许“static 整数型 class 常量”完成“in class 初值设定”,可改用所谓的"the enumhack”补偿做法。其理论基础是:“一个属于枚举类型(enumeratedtype)的数值可权充 ints 被使用”，于是 Gameplayer可定义如下:

   ```c++
   class GamePlayer{
   private:
   	enum {NumTurns=5}; //"the enum hack"_令NumTurns成为5的一个记号名称
   	int scores[NumTurns];
   }
   ```

   第一，enum hack的行为某方面说比较像 #define而不像const,有时候这正是你想要的。例如取一个 const的地址是合法的，但取一个 enum的地址就不合法，而取一个#define的地址通常也不合法。如果你不想让别人获得一个 pointer或reference 指向你的某个整数常量,enum 可以帮助你实现这个约束。此外虽然优秀的编译器不会为“整数型const对象”设定另外的存储空间(除非你创建一个pointer或reference指向该对象)，不够优秀的编译器却可能如此，而这可能是你不想要的。Enums和#defines一样绝不会导致非必要的内存分配。

**对于形似函数的宏(macros)，最好改用inline函数替换#defines**

1. #define实现宏(macros)：

   ```c++
   #define CALL_WITH_MAX(a,b) f((a)>(b)?(a):(b)) //无论何时当你写出这种宏，你必须记住为宏中的所有实参加上小括号
   ```

   看下面这段代码：

   ```c++
   int a=5，b=0;
   CALL WITH MAX(++a,b);	//a被累加二次
   CALL WITH MAX(++a，b+10);	//a 被累加一次
   ```

   在这里，调用f之前，a的递增次数竟然取决于“它被拿来和谁比较”!

   你可以获得宏带来的效率以及一般函数的所有可预料行为和类型安全性(typesafety)--只要你写出template inline函数(见条款30):

   ```c++
   template<typename T>
   inline void callWithMax(const T& a, const T& b)
   {
   	f(a > b ? a : b);
   }
   ```

   此外由于 cal1withMax是个真正的函数，它遵守作用域(scope)和访问规则。例如你绝对可以写出一个“class内的privateinline 函数”。一般而言宏无法完成此事。

# 条款03：尽可能使用const

```c++
char greeting="Hello";
char* p= greeting;			//non-const pointer, non-const data
const char*p=greeting;		//non-const pointer, const data
char* const p=greeting;		//const pointer, non-const data
const char* const p=greeting;	//const pointer, const data
```

**有些程序员会将关键字const写在类型之前，有些人会把它写在类型之后、星号之前。两种写法的意义相同，所以下列两个函数接受的参数类型是一样的:**

```c++
void fl(const widget*pw);	//f1获得一个指针，指向一个常量的(不变的)widget对象
void f2(Widget const *pw);	//f2也是
```

**令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而又不至于放弃安全性和高效性，例如：**

```c++
class Rational {...}；
operator*const Rational (const Rational& lhs,constRational&rhs);
```

为什么返回一个 const：

许多程序员会在无意识中那么做，只因为单纯的打字错误(以及一个可被隐式转换为 bool的类型):

```c++
Rational a,b,c
if(a * b = c) ...	//在a*b的成果上调用operator=
```

如果a和b都是内置类型，这样的代码直截了当就是不合法。而一个“良好的用户自定义类型”的特征是它们避免无端地与内置类型不兼容(见条款18)，因此允许对两值乘积做赋值动作也就没什么意思了。将operator*的回传值声明为const可以预防那个“没意思的赋值动作”，这就是该那么做的原因。

**const 成员函数：**

这有两个流行概念:bitwiseconstness(又称physical constness)和 logical constness:

bitwise const阵营的人相信,成员函数只有在不更改对象之任何成员变量(static除外)时才可以说是const。也就是说它不更改对象内的任何一个bit。这种论点的好处是很容易侦测违反点:编译器只需寻找成员变量的赋值动作即可。bitwiseconstness 正是C++ 对常量性(constness)的定义，因此 const 成员函数不可以更改对象内任何 non-static 成员变量。不幸的是许多成员函数虽然不足具备const性质却能通过 bitwise 测试。更具体地说，一个更改了“指针所指物”的成员函数虽然不能算是const，但如果只有指针(而非其所指物)隶属于对象，那么称此函数为bitwiseconst不会引发编译器异议。这导致反直观结果。假设我们有一个TextBlock-like class，它将数据存储为 char*而不是 string，因为它需要和一个不认识 string对象的CAPI沟通:

```c++
class CTextBlock
{
    public:
        ...
        char& operator[](std::size_t position) const // bitwise const 声明, 但其实不适当
        {return pText[position];}
    private:
        char* pText;
}
```

这个class不适当地将其operator[]声明为const成员函数，而该函数却返回一个 reference 指向对象内部值(条款28对此有深刻讨论)。假设暂时不管这个事实，请注意，operator[]实现代码并不更改pText。于是编译器很开心地为operator[]产出目标码。它是 bitwise const，所有编译器都这么认定。但是它允许发生什么事:

```c++
CTextBlock cctb("Hello");	//声明一一个常量对象。
constchar* pc=&cctb[0];		//调用 const operator[]取得--个指针// 指向 cctb的数据。
*pc ='J';					//cctb现在有了"Jello”这样的内容。
```

这其中当然不该有任何错误:你创建一个常量对象并设以某值，而且只对它调用const成员函数。但你终究还是改变了它的值。

这种情况导出所谓的logical constness。这一派拥护者主张，一个const 成员函数可以修改它所处理的对象内的某些 bits，但只有在客户端侦测不出的情况下才得如此。例如你的 cTextBlockclass有可能高速缓存(cache)文本区块的长度以便应付询问:

```c++
class CTextBlock
{
	public :
		...
		std::sizetlength()const;
	private:
		char* pText;
		std::size ttextLength;	//最近一次计算的文本区块长度。
		bool lengthIsValid;		//目前的长度是否有效。
};
std::size tCTextBlock::length() const
{
	if(!lengthIsValid){
		textLength = std::strlen(pText);	//错误!在const 成员函数内不能赋值给 textLength
		lengthIsValid =true;				//和lengthIsvalid.
	return textlength;
}
```

length的实现当然不是bitwise const，因为textLength和 lengthIsValid都可能被修改。这两笔数据被修改对const cTextBlock对象而言虽然可接受，但编译器不同意。它们坚持 bitwise constness。解决办法很简单:利用C++的一个与 const相关的摆动场:mutable(可变的)。mutable释放掉 non-static成员变量的 bitwise constness 约束:

```c++
class CTextBlock
{
	public:
		...
		std::sizet length()const;
    private:
		char* pText;
		mutable std::size ttextLength;		//这些成员变量可能总是
		mutable bool lengthIsValid;			//会被更改，即使在const 成员函数内。
};
std::sizetCTextBlock::length() const
{
	if(!lengthIsValid){
		textLength=std::strlen(pText);	//现在，可以这样，
		lengthIsValid =true;			//也可以这样。
	return textLength;
}
```

**在const 和 non-const 成员函数中避免重复:**

对于“bitwise-constness 非我所欲”的问题，mutable 是个解决办法，但它不能解决所有的 const相关难题。举个例子，假设TextBlock(和cTextBlock)内的 operator[]不单只是返回一个reference 指向某字符，也执行边界检验(boundschecking)、志记访问信息(oggedaccessinfo.)、甚至可能进行数据完善性检验。把所有这些同时放进const和non-const operator[]中，导致这样的怪物(暂且不管那将会成为一个“长度颇为可议”的隐喻式 inline 函数--见条款 30):

```c++
class TextBlock
{
	public:
		const char& operatorl](std::size_t position) const
        {
         	...	//边界检验(bounds checking)
            ...	//志记数据访问(1ogaccess data)
            ...	//检验数据完整性(verify dataintegrity)
			return text[position];
        }
		char& operator[](std::size_t position) 
        {
         	...	//边界检验(bounds checking)
            ...	//志记数据访问(1ogaccess data)
            ...	//检验数据完整性(verify dataintegrity)
			return text[position];
        }
	private:
		std::string text;
};
```

其中发生的代码重复以及伴随的编译时间、维护、代码膨胀等令人头痛的问题吗?当然将边界检验……等所有代码移到另一个成员函数(往往是个 private)并令两个版本的operator[]调用它，是可能的，但你还是重复了些代码，例如函数调用、两次 return语句等等。你真正该做的是实现operatorl]的机能一次并使用它两次。也就是说，你必须令其中一个调用另一个。这促使我们将常量性转除(castimgawayconstmess)。就一般守则而言，转型(casting)是一个糟糕的想法，我将贡献一整个条款来谈这码事(条款27)，告诉你不要那么做。然而代码重复也不是什么令人愉快的经验。本例中 const operator[]完全做掉了non-const版本该做的一切，唯一的不同是其返回类型多了一个const 资格修饰。这种情况下如果将返回值的const 转除是安全的，因为不论谁调用non-const operator[]都一定首先有个 non-const对象,否则就不能够调用 non-const函数。所以令 non-const operator[]调用其 const兄弟是一个避免代码重复的安全做法--即使过程中需要一个转型动作。下面是代码:

```c++
class TextBlock
{
	public:
    	...
		const char& operatorl](std::size_t position)	const //一如既往
        {
            ...
            ...
            ...
            return textlposition];
        }

    	char& operatorl](std::size_t position)	//现在只调用 const op[]
        {
            return	const_cast<char&>(static_cast<const TextBlock&>(*this)	////将 op[]返回值的 const 转除，为*this加上const[position]，调用 const op[]
        }
);
```

更值得了解的是，反向做法--令const版本调用non-const 版本以避免重复--并不是你该做的事。记住，const成员函数承诺绝不改变其对象的逻辑状态(1ogical state)，non-const成员函数却没有这般承诺。如果在const函数内调用non-const函数，就是冒了这样的风险:你曾经承诺不改动的那个对象被改动了。这就是为什么“const成员函数调用non-const成员函数”是一种错误行为:因为对象有可能因此被改动。实际上若要令这样的代码通过编译，你必须使用一个const cast将this身上的 const性质解放掉。反向调用(也就是我们先前使用的那个)才是安全的:non-const成员函数本来就可以对其对象做任何动作，所以在其中调用一个const成员函数并不会带来风险。这就是为什么本例以 static cast作用于this的原因:这里并不存在 const相关危险。

# 条款04：确定对象被使用前已先被初始化

1. 为内置型对象进行手工初始化，因为C不保证初始化它们；

2. 构造函数最好使用成员初值列(member initializationlist)，而不要在构造函数本体内使用赋值操作(assignment)。初值列列出的成员变量，其排列次序应该和它们在 class 中的声明次序相同；

3. 跨编译单元之初始化次序问题：

   ```c++
   class FileSystem 
   {
       ...
       public：
           std::size_t numDisks() const;
       ...
   };
   extern FileSystem tfs;
   
   class Directory { ... };
   Directory::Directory(parms)
   {
       ...
       std::size_t disks = tfs.numDisks();
       ...
   };
   Diretory tempDir(params);
   ```
   
   除非tfs在tempDir之前先被初始化否则 tempDir 的构造函数会用到尚未初始化的 tfs。但 tfs和 tempDir 是不同的人在不同的时间于不同的源码文件建立起来的，它们是定义于不同编译单元内的non-local static 对象。而C++对“定义于不同的编译单元内的 non-local static对象”的初始化相对次序并无明确定义。
   
   为免除“跨编译单元之初始化次序”问题，请以local static对象替换 non-local static 对象。意思是将每个non-local static 对象搬到自己的专属函数内(该对象在此函数内被声明为static)这些函数返回一个reference 指向它所含的对象。然后用户调用这些函数，而不直接指涉这些对象。换句话说，non-localstatic对象被localstatic对象替换了。示例如下：
   
   ```c++
   class FileSystem { ... };
   FileSystem& tfs()
   {
       static FileSystem fs;
       return fs;
   };
   class Directory { ... };
   Directory::Directory(parms)
   {
       ...
       std::size_t disks = tfs().numDisks();
       ...
   }
   Directory& tempDir()
   {
       static Diretory td;
       return td;
   }
   ```

# 条款05：了解 C++ 默默编写并调用哪些函数

1. 如果没有声明任何构造函数,编译器也会为你声明一个 defaut构造函数。

2. 编译器产出的析构函数是个non-virtual，除非这个class的 base class 自身声明有 virtual析构函数(这种情况下这个函数的虚属性 virtualness 主要来自base class)。

3. copy构造函数，编译器创建的版本只是单纯地将来源对象的每一个 non-static 成员变量拷贝到目标对象。

4. copy assignment操作符，其行为基本上与copy 构造函数如出一辙，但一般而言只有当生出的代码合法且有适当机会证明它有意义。看下面的例子：

   ```c++
   
   template<class T>
   class NamedObject
   {
   	public:
   		NamedObject(string& name) : nameValue(name) {}
   	private:
   		std::string& nameValue;
   };
   
   int main()
   {
       string newDog("Persephone");
       string oldDog("Satch");
       NamedObject<int> p(newDog);
       NamedObject<int> s(oldDog);
       p = s; //invalid
   }
   ```

   ```c++
   //自定义赋值运算符后允许
   template<class T>
   class NamedObject
   {
       public:
           NamedObject(string& name) : nameValue(name) {}
           NamedObject& operator=(NamedObject& other)
           {
               if (this != &other)
               {
                   nameValue = other.nameValue;
               }
               return *this;
           }
       private:
       	std::string& nameValue;
   };
   ```

   C++并不允许“让reference 改指向不同对象”，面对如此情况，C++的响应是拒绝编译那一行赋值动作。

   如果你打算在一个“内含reference 成员”的 class 内支持赋值操作(assignment)，你必须自己定义 copyassignment操作符。面对“内含const成员”编译器的反应也一样。更改 const 成员是不合法的，所以编译器不知道如何在它自己生成的赋值函数内面对它们。最后还有一种情况:如果某个basecasses将copy assignment操作符声明为private，编译器将拒绝为其derived classes生成一个 copy assignment 操作符。毕竟编译器为derived ciasses所生的copy assignment操作符想象中可以处理 base class成分，但它们当然无法调用 derived class 无权调用的成员函数。

5. 所有这些默认函数都是public且inline；

# 条款06：若不想使用编译器自动生成的函数，就该明确拒绝

1. 为驳回编译器自动提供的机能，可将相应的成员函数声明为private 并且不予实现。当客户企图拷贝对象，编译器会阻挠他。如果你不慎在 member 函数或 friend 函数之内那么做，轮到连接器发出抱怨。将连接期错误移至编译期是可能的(愈早侦测出错误愈好)，只要将 copy构造函数和 copy assignment操作符声明为 private就可以办到，但不是在该类自身，而是在一个专门为了阻止 copying 动作而设计的 base class 内。

   ```c++
   class Uncopyable
   {
       protected:
       	Uncopyable() {}
       	~Uncopyable() {}
       private:
       	Uncopyable(const Uncopyable&);
       	Uncopyable& operator=(const Uncopyable&);
   };
   
   class Derived : private Uncopyable { ... }
   ```

   只要任何人—甚至是member 函数或 friend 函数—尝试拷贝 Derived 对象，编译器便试着生成一个 copy 构造函数和一个 copy assignment操作符，这些函数的“编译器生成版”会尝试调用其   baseclass 的对应兄弟，那些调用会被编译器拒绝，因为其baseclass的拷贝函数是 private。

# 条款07：为多态基类声明 virtual析构函数

1. polymorphic(带多态性质的)base classes应该声明一个 virtual 析构函数。如果class 带有任何 virtual函数，它就应该拥有一个 virtual析构函数，否则可能造成“局部销毁；

   ```c++
   class TimeKeeper{
       public:
   		TimeKeeper();
       	~TimeKeeper();
   };
   AtomicClock:public TimeKeeper{...}; //原子钟class 
   WaterClock:public TimeKeeper{...};	//水钟class 
   WristWatch:public TimeKeeper{..};	//腕表class 
   
   TimeKeeper* getTimeKeeper();	//返回一个指针，指向一个
   								//TimeKeeper派生类的动态分配对象
   								
   TimeKeeper* ptk=getTimeKeeper();	//从TimeKeeper继承体系//获得一个动态分配对象。
   ...									//运用它
   delete ptk;							//释放它，避免资源泄漏。
   ```

   问题出在 getrimeKeeper返回的指针指向一个derived class 对象(例如AtomicClock)，而那个对象却经由一个base class指针(例如一个 TimeKeeper*指针)被删除，而目前的baseclass(TimeKeeper)有个non-virtual析构函数。这是一个引来灾难的秘诀，因为C++明白指出，当derived class对象经由一个 baseclass指针被删除,而该base class 带着一个non-virtual析构函数,其结果未有定义-实际执行时通常发生的是对象的 derived 成分没被销毁。如果getTimeKeeper 返回指针指向一个Atomicclock对象，其内的tomicclock成分(也就是声明于AtomicClock class内的成员变量)很可能没被销毁，而tomicclock的析构函数也未能执行起来。然而其base class成分(也就是TimeKeeper这一部分)通常会被销毁，于是造成一个诡异的“局部销毁”对象。这可是形成资源泄漏、败坏之数据结构、在调试器上浪费许多时间的绝佳途径。消除这个问题的做法很简单:给base class一个virtual析构函数。此后删除 derivedclass 对象就会如你想要的那般。它会销毁整个对象，包括所有derived class成分:

   ```c++
   class TimeKeeper{
       public:
       	TimeKeeper();
   		virtual ~TimeKeeper();
   };
   TimeKeeper* getTimeKeeper();
   ...
   delete
   ```

2. Classes的设计目的如果不是作为 base classes 使用，或不是为了具备多态性，就不该声明 virtual 析构函数；

3. 若希望拥有抽象基类，由于抽象class总是企图被当作一个base class来用，而又由于 base class 应该有个 virtual 析构函数，并且由于 pure virtual 函数会导致抽象 class，因此为你希望它成为抽象的那个 class 声明一个 pure virtual 析构函数即可。下面是个例子:

   ```c++
   class AWOV
   {
   	public:
   		virtual ~AWOV( ) = 0;
   };
   ```

   必须为这个pure virtual析构函数提供一份定义，可以为空定义（除此之外的其他纯虚函数可以没有定义）:

   AWOV::~AWOV( ) { ... }

   析构函数的运作方式是，最深层派生(mostderived)的那个class其析构函数最先被调用,然后是其每一个base class的析构函数被调用。编译器会在AWOV的 derive classes 的析构函数中创建一个对~AWOV的调用动作，所以你必须为这个函数提供一份定义。如果不这样做，连接器会发出抱怨。

# 条款08：别让异常逃离析构函数

考虑下面的代码：

```
class widget{
	public:
		...
		~widget(){...}//假设这个可能吐出一个异常
};

void doSomething()
{
	std::vector<widget> v;
}

//v在这里被自动销毁
```

当vectorv被销毁，它有责任销毁其内含的所有widgets。假设v内含十个widgets，而在析构第一个元素期间，有个异常被抛出。其他九个 widgets 还是应该被销毁(否则它们保存的任何资源都会发生泄漏)，因此v应该调用它们各个析构函数。但假设在那些调用期间，第二个 widget析构函数又抛出异常。现在有两个同时作用的异常，这对C++而言太多了。在两个异常同时存在的情况下，程序若不是结束执行就是导致不明确行为。本例中它会导致不明确的行为。使用标准程序库的任何其他容器(如list,set)或TR1的任何容器(见条款54)或甚至array，也会出现相同情况。容器或 array 并非遇上麻烦的必要条件，只要析构函数吐出异常，即使并非使用容器或 arrays，程序也可能过早结束或出现不明确行为。是的，C++不喜欢析构函数吐出异常!

1. 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们(不传播)或结束程序；

   假设你使用一个 class 负责数据库连接:

   ```c++
   class DBConnection
   {
       public:
       	...
       	static DBConnection create();	//这个函数返回
   										//DBConnection 对象:
       									//为求简化暂略参数。
       	void close();					//关闭联机;失败则抛出异常。		
   }
   ```

   为确保客户不忘记在 DBConnection对象身上调用close()，一个合理的想法是创建一个用来管理 DBConnection资源的 class，并在其析构函数中调用 close（这一类用于资源管理的classes在第3章有详细探讨）:

   ```c++
   class DBConn	//这个 class 用来管理 DBConnection 对象
   {
   	public:
   		~DBConn()	//确保数据库连接总是会被关闭
           {
           	db.close();
           }		
   	private:
   		DBConnection db;
   };
   ```

   客户代码：

   ```c++
   {
   	DBConn dbc(DBConnection::create());		//开启一个区块(block)
   											//建立 DBConnection对象并
   											//交给 DBConn 对象以便管理。
   											//通过 DBConn 的接口
   											//使用 DBConnection 对象。
   											//在区块结束点，DBConn对象
   											//被销毁，因而自动
   											//为 DBConnection对象调用close
   	...
   }
   
   ```

   1. 如果close抛出异常就结束程序。通常通过调用abort完成:

      ```c++
      DBConn::~DBConn()
      {
      	try { db.close(); }
      	catch(...) {
      		//制作运转记录，记下对close的调用失败;
      		std::abort();
      	}
      }
      ```

      如果程序遭遇一个“于析构期间发生的错误”后无法继续执行，“强迫结束程序”是个合理选项。毕竟它可以阻止异常从析构函数传播出去(那会导致不明确的行为)。也就是说调用abort可以抢先制“不明确行为”于死地。

   2. 吞下因调用 close 而发生的异常:

      ```c++
      DBConn::~DBConn()
      {
      	try { db.close(); }
      	catch(...) {
      		//制作运转记录，记下对close的调用失败;
      	}
      }
      ```

      一般而言，将异常吞掉是个坏主意，因为它压制了“某些动作失败”的重要信息!然而有时候吞下异常也比负担“草率结束程序”或“不明确行为带来的风险”好。为了让这成为一个可行方案，程序必须能够继续可靠地执行，即使在遭遇并忽略一个错误之后。

2. 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数(而非在析构函数中)执行该操作。

   ```c++
   class DBConn
   {
   	public:
   		...
   		void close()
   		{
   			db.close();
   			closed = true;
   		}
   		~DBConn()
           {
           	if(!closed)
           	{
                   try { db.close(); }
                   catch(...) {
                       //制作运转记录，记下对close的调用失败;
                       ...
                   }
           	}
           }		
   	private:
   		DBConnection db;
   		bool closed;
   };
   ```

# 条款09：绝不在构造和析构过程中调用 virtual函数

```c++
class Transaction
{
    public:
    	Transaction();
    	virtual void logTransaction() const = 0;
    	...
};

Transaction::Transaction()
{
    ...
    logTransaction();
};

class BuyTransaction : public Transaction()
{
    public:
    	virtual void logTransaction() const;
    	...
};

class SellTransaction : public Transaction
{
    public:
    	virtual void logTransaction() const;
    	...
};

BuyTransaction b;
//无疑地会有一个 BuyTransaction 构造函数被调用，但首先 Transaction 构造函数一定会更早被调用;是的,derived class对象内的base class成分会在 derived class自身成分被构造之前先构造妥当。Transaction构造函数的最后一行调用virtual函数 logTransaction，这正是引发惊奇的起点。这时候被调用的 logtransaction 是Transaction内的版本，不是 BuyTransaction内的版本---即使目前即将建立的对象类型是 BuyTransaction。是的，baseclass构造期间virtual 函数绝不会下降到derived classes阶层。取而代之的是，对象的作为就像隶属 base 类型一样。非正式的说法或许比较传神:在 baseclass构造期间，virual函数不是virtual函数。
```

在构造和析构期间不要调用 virtual函数，因为这类调用从不下降至 derived class(比起当前执行构造函数和析构函数的那层)。

其他方案可以解决这个问题。一种做法是在classTransaction内将logTransaction函数改为 non-virtual，然后要求 derived class 构造函数传递必要信息给Transaction构造函数，而后那个构造函数便可安全地调用non-virtuallogTransaction。像这样:

```c++
class Transaction
{
	public:
		explicit Transaction(const std::string& logInfo);
    	void logTransaction(const std::string& logInfo) const;
};

Transaction::Transaction(const std::string& logInfo)
{
    ...
    logTransaction (logInfo);
}

class BuyTransaction : public Transactionpublic
{
	public:
		BuyTransaction(parameters) : Transaction(createLogString(parameters)){ ... }
		...
	private:
		static std::string createLogstring(parameters);

};
```

请注意本例之 BuyTransaction内的private static 函数 createlogString 的运用。是的，比起在成员初值列(member initializationlist)内给予base class 所需数据，利用辅助函数创建一个值传给base class构造函数往往比较方便(也比较可读)。令此函数为 static，也就不可能意外指向“初期未成熟之 BuyTransaction 对象内尚未初始化的成员变量”。这很重要，正是因为“那些成员变量处于未定义状态”，所以“在 base class 构造和析构期间调用的 virtual 函数不可下降至 derived classes。

# 条款10：令operator=返回一个reference to *this(左侧对象)

“左侧对象” 通常是指在 C++ 中的赋值操作符 (`=`) 或者其他二元运算符（如加法 `+`、减法 `-` 等）中，位于左边的操作数对象。

# 条款11：在operator=中处理“自我赋值“

1. 如果 px和 py恰巧指向同一个东西，这也是自我赋值。这些并不明显的自我赋值，是“别名”(aliasing)带来的结果:所谓“别名”就是“有一个以上的方法指称(指涉)某对象”。一般而言如果某段代码操作pointers或references 而它们被用来“指向多个相同类型的对象”，就需考虑这些对象是否为同一个。实际上两个对象只要来自同一个继承体系，它们甚至不需声明为相同类型就可能造成“别名”因为一个base class的reference 或 pointer 可以指向一个 derived class 对象；

   a[i] = a[j];

   *px = *py；

2. 确保当对象自我赋值时 operator=有良好行为。

   意外释放资源：

   ```c++
   class Bitmap { ... }
   class Widget
   {
       ...
       private:
       	Bitmap* pb;
   }
   Widget& Widget::operator=(const Widget& rhs)
   {
       delete pb;
       pb = new Bitmap(*rhs.pb);
       return *this;
   }
   ```

   1. 比较“来源对象”和“目标对象”的地址：

      ```c++
      class Bitmap { ... }
      class Widget
      {
          ...
          private:
          	Bitmap* pb;
      }
      Widget& Widget::operator=(const Widget& rhs)
      {
          if(this == &rhs) return *this;
          delete pb;
          pb = new Bitmap(*rhs.pb);
          return *this;
      }
      ```

   2. 精心周到的语句顺序：

      ```c++
      Widget& Widget::operator=(const Widget& rhs)
      {
          Bitmap* pOrig = pb;
          pb = new Bitmap(*rhs.pb);
          delete pOrig;
          return *this;
      }
      ```

   3. copy-and-swap：

      ```c++
      class Widget
      {
          ...
          void swap(Widget& rhs);
          ...
      };
      Widget& Widget::operator=(const Widget& rhs)
      {
          Widget temp(rhs);
          swap(temp);
          return *this;
      }
      ```

      ```c++
      Widget& Widget::operator=(Widget rhs)
      {
          swap(rhs);
          return *this;
      }
      ```

3. 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。

# 条款12：复制对象时勿忘其每一个成分

1. Copying函数应该确保复制“对象内的所有成员变量”及“所有base class 成分，即当你编写一个copying函数，请确保(1)复制所有local成员变量，(2)调用所有baseclasses内的适当的copying 函数。

   ```c++
   void logCall(const std::string& funcName); //志记
   class Date { ... }
   class Customer
   {
       public:
       	...
       private:
       	std::string name;
       	Date lastTransaction;
   };
   
   class PriorityCustomer : public Customer
   {
   	public:
   		...
   		PriorityCustomer(const PriorityCustomer& rhs);
   		PriorityCustomer& operator(const PriorityCustomer& rhs);
   	private:
   		int priority;
   };
   
   PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs) : Customer(rhs), priority(rhs.priority)
   {
       logCall("PriorityCustomer copy constructor");
   }
   
   PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
   {
       logCall("PriorityCustomer copy assignment i4operator");
       Customer::operator=(rhs);
       priority = rhs.priority;
       return *this;
   }
   ```

2. 这两个 copying函数往往有近似相同的实现本体,这可能会诱使你让某个函数调用另一个函数以避免代码重复。但是令某个copying函数调用另一个 copying函数却无法让你达到你想要的目标。令copyassignment操作符调用copy构造函数是不合理的，因为这就像试图构造一个已经存在的对象。反方向--令 copy构造函数调用 copy assignment操作符--同样无意义。构造函数用来初始化新对象，而assignment操作符只施行于已初始化对象身上。对一个尚未构造好的对象赋值，就像在一个尚未初始化的对象身上做“只对已初始化对象才有意义”的事一样。**不要尝试以某个 copying函数实现另一个copying函数。应该将共同机能放进第三个函数中（往往是private 而且常被命名为 init），并由两个 coping函数共同调用。**

# 条款13：以对象管理资源

```c++
class Investment { ... };
Investment* pInv = createInvestment( );
...
delete pInv;
```

这看起来妥当，但若干情况下可能无法删除它得自createInvestment的投资对象，或许因为”...“区域内的一个过早的 return语句。如果这样一个 return被执行起来，控制流就绝不会触及delete语。类似情况发生在对createInvestment的使用及delete 动作位于某循环内，而该循环由于某个continue或goto语句过早退出。最后一种可能是"...”区域内的语句抛出异常果真如此控制流将再次不会幸临 delete。无论 delete如何被略过去，我们泄漏的不只是内含投资对象的那块内存，还包括那些投资对象所保存的任何资源。

1. 为防止资源泄漏，在获得资源后立刻放入管理对象，它们在构造函数中获得资源并在析构函数中释放资源。以对象管理资源，也被称为资源取得时机便是初始化（RALL）；

2. 两个常被使用的RAIl classes分别是tr1::shared_ptr和auto_ptr。前者通常是较佳选择,因为其copy行为比较直观。若选择auto_ptr，复制动作会使它(被复制物)指向 null，复制所得的指针将取得资源的唯一拥有权。（c++98之后auto_ptr被废弃，移步modern c++）

   ```c++
   std::auto_ptr<Investment> pInv(createInvestment());
   ...
   ```

   ```c++
   std::tr1::shared_ptr<Investment> pInv(createInvestment());
   ...
   ```

3. auto ptr和tr1::shared ptr 两者都在其析构函数内做 delete 而不是delete[]动作(条款16对两者的不同有些描述)。那意味在动态分配而得的array身上使用 auto ptr或tr1::shared ptr是个馊主意。尽管如此，那么做仍能通过编译:

   ```c++
   std::auto ptr<std::string>						//会用上错误的
   aps(new std::string[10]);						//delete形式。
   std::trl::shared ptr<int> spi(new int[1024]);	//相同问题
   ```

   你或许会惊讶地发现，并没有特别针对“C++动态分配数组”而设计的类似auto_ptr或tr1::shared ptr那样的东西，甚至TR1中也没有。那是因为 vector和 string 几乎总是可以取代动态分配而得的数组。如果你还是认为拥有针对数组而设计、类似 auto ptr和 tr1::shared ptr那样的 classes 较好，参考 Boost 吧(见条款55)。在那儿会发现boost::scoped_array和boost::shared_array classes，它们都提供你要的行为。

# 条款14：在资源管理类中小心 coping 行为

**并非所有资源都是 heap-based，对那种资源而言，像auto_ptr和tr1::shared_ptr这样的智能指针往往不适合作为资源掌管者(resourcehanders)。既然如此，你需要建立自己的资源管理类:**

假设使用C API函数处理类型为Mutex的互斥器对象(mutexobjects)，共有lock和 unlock两函数可用:

```c++
void lock(Mutex*pm);	 //锁定 pm 所指的互斥器
void unlock(Mutex*pm);	 //将互斥器解除锁定.
```

为确保绝不会忘记将一个被锁住的Mutex解锁，你可能会希望建立-class用来管理机锁。这样的 class的基本结构由RAII守则支配，也就是“资源在构造期间获得，在析构期间释放”：

```c++
class Lock
{
    public:
    	explicit Lock(Mutex* pm) : mutexPtr(pm)
        {
            lock(mutexPtr); //加锁
        }
    	~Lock() { unlock(mutexPtr); } //解锁
    private:
    	Mutex *mutePtr;
};
Mutex m; //定义所需要的互斥器
...
{
    Lock ml(&m);
    ...
}
```

**如果底部资源Lock对象被复制，可能面临一些问题，此情况下一般会选择以下两种可能：**

```c++
Lock ml1(&m);
Lock ml2(ml1);
```

1. **禁止复制：**

   许多时候允许 RAIl 对象被复制并不合理。对一个像lock这样的 class这是有可能的，因为很少能够合理拥有“同步化基础器物”(synchronizationprimitives)的复件(副本)。如果复制动作对RAIclass 并不合理，你便应该禁止之。条款6告诉你怎么做：将copying操作声明为private。对ock而言看起来是这样:

   ```c++
   class Lock: private Uncopyable
   {
       public:
       	...
   }
   ```

2. **施行引用计数法(reference counting):**

   如果前述的 Lock打算使用 reference counting,它可以改变 mutexPtr的类型，将它从Mutex*改为tr1::shared ptr< Mutex >。然而很不幸 tr1::shared ptr的缺省行为是“当引用次数为0时删除其所指物”那不是我们所要的行为。当我们用上一个Mutex，我们想要做的释放动作是解除锁定而非删除，幸运的是tr1::shared_ptr允许指定所谓的“删除器”(deleter)，那是一个函数或函数对象(function object)，当引用次数为0时便被调用(此机能并不存在于 auto ptr—它总是将其指针删除)。删除器对tr1::shared ptr构造函数而言是可有可无的第二参数，代码如下：

   ```c++
   class Lock
   {
       public:
       	explicit Lock(Mutex* pm) : mutexPtr(pm, unlock)
           {
               lock(mutexPtr.get()); //加锁，.get()获取原始资源
           }
       private:
       	std::tr1::shared_ptr<Mutex> mutexPtr;
   };
   ```

   请注意，本例的 Lock class不再声明析构函数。因为没有必要。条款5说过,class析构函数(无论是编译器生成的,或用户自定的)会自动调用其 non-static成员变量(本例为mutexptr)的析构函数。而mutexPtr的析构函数会在互斤器的引用次数为0时自动调用tr1::sharedptr的删除器(本例为unlock);

**以上两种做法都不会复制底部资源，下面两种则是其他情况下采用的复制策略**

1. 复制底部资源。有时候，只要你喜欢，可以针对一份资源拥有其任意数量的复件(副本)。而你需要“资源管理类”的唯一理由是，当你不再需要某个复件时确保它被释放。在此情况下复制资源管理对象，应该同时也复制其所包覆的资源。也就是说，复制资源管理对象时，进行的是“深度拷贝”。某些标准字符串类型是由“指向heap内存”之指针构成(那内存被用来存放字符串的组成字符)。这种字符串对象内含一个指针指向一块heap内存。当这样一个字符串对象被复制，不论指针或其所指内存都会被制作出一个复件。这样的字符串展现深度复制(deepcopying)行为。
2. 转移底部资源的拥有权。某些罕见场合下你可能希望确保永远只有一个RAI对象指向一个未加工资源(rawresource)，即使RAI对象被复制依然如此。此时资源的拥有权会从被复制物转移到目标物。一如条款13所述，这是auto_ptr奉行的复制意义。

**请记住：**

- 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的 copying行为。
- 普遍而常见的RAIl class copying行为是:抑制copying、施行引用计数法。不过其他行为也都可能被实现。(reference counting)

# 条款15：在资源管理类中提供对原始资源的访问

1. APIS往往要求访问原始资源(rawresources)，所以每一个RAII class 应该提供一个“取得其所管理之资源”的办法。对原始资源的访问可能经由显式转换或隐式转换；
   1. 显示转换：
      1. 其他 RAII class 可以定义成员函数：get()，它返回所管理的原始资源对象；

         ```c++
         //用于字体的RALL calss
         FontHandle getFont();
         void releaseFont(FontHandle fh);
         class Font
         {
             public:
             	explicit Font(FontHandle fh) : f(fh) { }
             	FontHandle get() const { return f; }
             	...
             	~Font() { releaseFont(f); }
             private:
             	FontHandle f;
         };
         
         //客户每当想要使用 API时就必须调用get
         void changeFontSize(FontHandle f, int newSize);
         Font f(getFont());
         int newFontSize;
         ...
         changeFontSize(f.get(), newFontSize);
         ```

      2. 对于智能指针保存 factory 函数的调用结果：使用.get()成员函数执行显示转换，返回智能指针内部的原始指针；

         ```c++
         std::tr1::shared_ptr<Investment> pInv(createInvestment());
         int daysHeld(const Incestment* pi);
         int days = daysHeld(pInv.get());
         ```
   2. 隐式转换：
      1. 其他 RAII class 可以定义成员函数：operator 原始资源类型() const，它返回所管理的原始资源对象;

         ```c++
         class Font
         {
             public:
             	...
             	operator FontHandle() const { return f; }
             	...
         };
         
         void changeFontSize(FontHandle f, int newSize);
         Font f(getFont());
         int newFontSize;
         ...
         changeFontSize(f, newFontSize);
         
         //增加错误机会
         Font f1(getFont());
         ...
         FontHandle f2 = f1; //原意是要拷贝一个 Font 对象，却反而将 f1 隐式转换为其底部的 FontHandle 然后才复制它，当1被销毁，字体被释放，而f2因此成为“虚吊的”(dangle)。
         ```

      2. 对于智能指针保存 factory 函数的调用结果：智能指针重载了指针取值操作符 -> 和 * ，操作符 -> 返回被封装的那个指针（通过操作符 -> 获取内部的指定成员）， 操作符 * 返回被封装的那个指针指向的原始资源对象的引用；

         ```c++
         class Investment
         {
             public:
             	bool isTaxFree() const;
             	...
         };
         Investment* createInvestment();
         std::tr1::shared_ptr<Investment> pi1(createInvestment());
         bool taxablel = !(pi1->isTaxFree());
         ...
         std::auto_ptr<Investment> pi2(createInvestment());
         bool taxavle2 = !((*pi2).isTaxFree());
         ...
         ```
2. 一般而言显式转换比较安全但隐式转换对客户比较方便。相较于显式转换，采用隐式转换可能在某些场合发生错误。

# 条款16：成对使用 new和 delete时要采取相同形式

1. 如果你在 new表达式中使用[]，必须在相应的 delete表达式中也使用[]；
2. 如果你在 new表达式中不使用[]，一定不要在相应的 de1ete 表达式中使用[]。

# 条款17: 以独立语句将 newed 对象置入智能指针

```c++
int priority();
void processWidget(std::tr1::shared_ptr< widget > pw, int priority);
processWidget(new Widget, priority()); 
```

不能通过编译，tr1::shared_ptr构造函数需要一个原始指针(raw pointer)，但该构造函数是个explicit 构造函数，无法进行隐式转换，将得自"new widget"的原始指针转换为process widget所要求的tr1::shared_ptr。

```c++
processWidget(std::tr1::shared_ptr< widget >(new Widget), priority());
```

在调用processwidget之前，编译器必须创建代码，做以下三件事：调用 priority；执行“new widget"；调用tr1::shared ptr构造函数。可以确定的是：new widget一定执行于 tr1::shared_ptr 构造函数被调用之前,因为这个表达式的结果还要被传递作为 tr1::shared_ptr 构造函数的一个实参，但对priority的调用则可以排在第一或第二或第三执行。如果编译器选择以第二顺位执行它，最终获得这样的操作序列:
1.执行"new widget"
2.调用 priority
3.调用 tr1::shared ptr构造函数

万一对priority的调用导致异常，new widget 返回的指针将会遗失，因为它尚未被置入tr1::shared ptr内，可能会造成资源泄露，以独立语句将 newed 对象存储于(置入)智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄露：

```c++
std::tr1::shared_ptr< widget > pw(new Widget);
processWidget(pw, priority());
```

# 条款18：让接口容易被正确使用，不易被误用

1. “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容；

   例如条款3说明为什么“以const修饰 operator的返回类型”可阻止客户因“用户自定义类型”而犯错：if(a * b = c) ... //原意其实是要做一次比较动作

   如果a和b都是ints，那么对 a*b 赋值并不合法，所以除非你有好的理由与此行为分道扬镳，否则应该让你的types也有相同的表现。是的，一旦怀疑，就请拿 ints 做范本。

2. “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值；

   ```c++
   class Date
   {
       public:
       	Date(int month, int day, int year);
       	...
   };
   
   //容易犯下错误
   Date d(30, 3, 1995);
   Date d(2, 30, 1995);
   ```

   使用struct:

   ```c++
   struct day
   {
   	explicit Day(int d) :val(d) {}
   	int val;
   };
   
   struct Month { ... }
   struct Year { ... }
   
   class Date
   {
       public:
       	Date(const Month& month, const Day& day, const Year& year);
       	...
   };
   
   Data d(3, 30, 1995); //invalid
   Data d(Month(3), Day(30), Year(1995)); //valid
   ```

   使用class(可以封装数据，比简单实用struct要更好更安全):

   ```c++
   class Date
   {
       public:
       	Date(const Month& m, const Day& d, const Year& y);
       	...
   };
   
   class Month
   {
       public:
       	//如果“以函数替换对象，表现某个特定月份”让你觉得诡异，或许是因为你忘记了non-local static对象的初始化次序有可能出问题。建议阅读条款4恢复记忆。
       	static Month Jan() { return Month(1); }
       	static Month Feb() { return Month(2); }
       	...
       	static Month Dec() { return Month(12); }
       	...
       private:
       	explicit Month(int m);
       	...
   };
   
   Date d(Month::Mar(), Day(30), Year(1995));
   ```

3. 消除客户的资源管理责任；

   例如条款13导入了一个factory 函数，它返回一个指针指向 Investment 继承体系内的一个动态分配对象：Investment* createInvestment();

   为避免资源泄漏，createInvestment返回的指针最终必须被删除，但那至少开启了两个客户错误机会:没有删除指针，或删除同一个指针超过一次。

   此时应该使用智能指针：std::tr1::shared ptr< Investment > createInvestment();

   假设 cass 设计者期许那些“从createInvestment取得 Investment*指针”的客户将该指针传递给一个名为getRidofInvestment的函数，而不是直接在它身上动刀(使用delete)。这样一个接口又开启通往另一个客户错误的大门，该错误是“企图使用错误的资源析构机制”(也就是拿delete替换getRidofInvestment)。

   返回一个“将getRidofInvestment绑定为删除器(deleter)”的tr1::shared_ptr：

   ```c++
   std::tr1::shared_ptr<Investment> createInvestment()
   {
       std::tr1::shared_ptr<Investment> retVal(static_cast<Investment*>(0), getRidOfInvestment);
       retVal = ...;
       return retVal;
   }
   ```

4. tr1::shared ptr支持定制型删除器(custom deleter)。这可防范 DLL问题，可被用来自动解除互斥锁(mutexes;见条款14)等等。tr1::sharedptr有一个特别好的性质是:它会自动使用它的“每个指针专属的删除器”，因而消除另一个潜在的客户错误:所谓的"cross-DLLproblem"。这个问题发生于“对象在动态连接程序库(DLL)中被 new创建，却在另一个DLL 内被 delete销毁”。在许多平台上，这一类“跨 DLL之 newdelete 成对运用”会导致运行期错误。tr1::shared ptr没有这个问题，因为它缺省的删除器是来自“tr1::shared ptr诞生所在的那个 DLL”的delete。这意思是……唔……让我举个例子，如果 stock派生自Investment而createInvestment实现如下:

   ```c++
   std::tr1::shared_ptr<Investment> createInvestment()
   {
       return std::tr1::shared_ptr<Investment>(new Stock);
   }
   ```

   返回的那个 tr1::shared ptr可被传递给任何其他DLLS，无需在意"cross-DLLproblem"。这个指向 stock的 tr1::shared ptrs 会追踪记录“当 stock的引用次数变成0时该调用的那个 DLL's delete”

# 条款19：设计 class 犹如设计 type

# 条款20：宁以pass-by-reference-to-const替换 pass-by-value

1. 尽量以 pass-by-reference-to-const替换 pass-by-value。前者通常比较高效，并可避免切割问题(slicing probiem);         

   切割问题：

   ```c++
   class Window
   {
       public:
       	...
       	std::string name() const;
       	virtual void display() const;
   };
   
   class WindowWithScrollBars : public Window
   {
       public:
       	...
       	virtual void display() const;
   };
   
   void printNameAndDisplay(Window w)
   {
       std::cout << w.name();
       w.display();
   }
   
   WindowWithScrollBars wwsb;
   printNameAndDisplay(wwsb);
   ```

   参数w会被构造成为一个 window对象;它是 passed by value，造成 wwsb“之所以是个 windowwithScrollBars 对象”的所有特化信息都会被切除。在 printNameAndDisplay函数内不论传递过来的对象原本是什么类型，参数w就像一个 window对象(因为其类型是 window).因此在 printNameAndDisplay内调用 display调用的总是 window::display，绝不会是windowwithScrollBars::display。

2. 如果窥视 C++编译器的底层，你会发现，references 往往以指针实现出来，因此pass by reference通常意味真正传递的是指针。因此如果你有个对象属于内置类型(例如 int)，pass by value往往比 pass byreference的效率高些。对内置类型而言，当你有机会选择采用 pass-by-value或pass-by-reference-to-const时，选择 pass-by-.value并非没有道理。这个忠告也适用于STL的迭代器和函数对象，因为习惯上它们都被设计为passed by value。

# 条款21：必须返回对象时，别妄想返回其reference

考虑下面的代码：

```c++
class Rational
{
    public:
    	Rational(int numerator = 0, int denominator = 1);
    	...
    private:
    	int n, d;
    	friend const Rational operator* (const Rational& lhs, const Rational& rhs);
};
```

这个版本的operator*系以byvalue方式返回其计算结果(一个对象)。如果你完全不担心该对象的构造和析构成本，你其实是明显逃避了你的专业责任。若非必要没有人会想要为这样的对象付出太多代价，问题是需要付出任何代价吗？考虑下面的代码带来的后果：

1. 绝不要返回 pointer 或reference 指向一个 local stack对象；

   ```c++
   const Rational& operator* (const Rational& lhs, const Rational& rhs)
   {
       Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
       return result;
   }
   ```

   你可以拒绝这种做法，因为你的目标是要避免调用构造函数，而resu1t却必须像任何对象一样地由构造函数构造起来。更严重的是:这个函数返回一个reference 指向result，但result是个local对象，而local对象在函数退出前被销毁了。

2. 绝不要返回 reference 指向一个 heap-allocated 对象；

   ```c++
   const Rational& operator* (const Rational& lhs, const Rational& rhs)
   {
       Rational* resukt = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
       return *result;
   }
   ```

   还是必须付出一个“构造函数调用”代价，并且即使调用者诚实谨慎，并且出于良好意识，他们还是不太能够在这样合情合理的用法下阻止内存泄漏:

   Rational w, x, y, z;

   w = x * y * z;

   *这里，同一个语句内调用了两次operator，因而两次使用new，也就需要两次delete。但却没有合理的办法让 operator使用者进行那些delete 调用，因为没有合理的办法让他们取得operator返回的references 背后隐藏的那个指针。这绝对导致资源泄漏。**

3. 绝不要返回pointer 或reference 指向一个 local static 对象而有可能同时需要多个这样的对象；

   ```c++
   const Rational& operator* (const Rational& lhs, const Rational& rhs)
   {
       static Rational result;
       result = ...;
       return result;
   }
   
   bool operator==(const Rational& lhs, const Rational& rhs);
   Rational a, b, c, d;
   ...
   if((a * b) == (c * d)) //结果总为true
   {
       ...
   }
   else
   {
       ...
   }
   ```

   写成等价的函数形式：if(operator==(operator * (a, b), operator * (c, d)));

   注意，在operator==被调用前，已有两个operator*调用式起作用，每一个都返回 reference 指向 operator * 内部定义的 static Rational对象。因此 operator== 被要求将“operator *内的 static Rational对象值”拿来和“operator *内的 staticRationa对象值”比较，结果必然相等。(译注:这里我补充说明:两次 operator *调用的确各自改变了 static Rational对象值，但由于它们返回的都是reference，因此调用端看到的永远是 static Rational对象的“现值”。)

   如果一个 static 不够，或许一个 static array可以，但首先你必须选择array大小n。如果n太小，你可能会耗尽“用以存储函数返回值”的空间，那么情况就回到了我们刚才讨论过的单static 设计。但如果n太大，会因此降低程序效率，因为array内的每一个对象都会在函数第一次被调用时构造完成。那么将消耗n个构造函数和n个析构函数---即使我们所讨论的函数只被调用一次。条款4已经为“在单线程环境中合理返回reference指向一个 local static 对象”提供了一份设计实例。

# 条款22：将成员变量声明为 private

1. 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。
2. protected 并不比 public 更具封装性，因为派生类的成员可以直接访问基类的保护成员，但不能直接访问基类的私有成员。对于外部世界来说， 保护成员的行为与私有成员相似；但对于派生类来说，保护成员的行为与公有成员相似。

# 条款23：宁以non-member、non-friend 替换 member 函数

宁可拿 non-member non-friend 函数替换 member 函数。这样做可以增加封装性、裹弹性和机能扩充性：

1. 封装性：

   如果要你在一个 member 函数(它不只可以访问 class内的 private 数据，也可以取用 private 函数、enums、typedefs 等等)和一个 non-member non-fiend函数(它无法访问上述任何东西)之间做扶择，而且两者提供相同机能，那么，导致较大封装性的是 non-member non-friend 函数，因为它并不增加“能够访问class 内之private 成分”的函数数量；

2. 裹弹性和机能扩充性：

   一个像 webBrowser 这样的 class 可能拥有大量便利函数,某些与书签(bookmarks)有关，某些与打印有关，还有一些与cookie的管理有关…… 通常大多数客户只对其中某些感兴趣。没道理一个只对书签相关便利函数感兴趣的客户却与cookie 相关便利函数发生编译相依关系。分离它们的最直接做法就是将书签相关便利函数声明于一个头文件，将cookie 相关便利函数声明于另一个头文件，再将打印相关便利函数声明于第三个头文件，依此类推。将所有便利函数放在多个头文件内但隶属同一个命名空间，意味客户可以轻松扩展这一组便利函数。因为namespace和classes不同，前跨越多个源码文件而后者不能。他们需要做的就是添加更多non-member non-friend 函数到此命名空间内。举个例子，如果某个webBrowser客户决定写些与影像下载相关的便利函数他只需要在 webBrowserstuff命名空间内建立一个头文件,内含那些函数的声明即可。新函数就像其他旧有的便利函数那样可用且整合为一体。这是class 无法提供的另一个性质，因为class定义式对客户而言是不能扩展的。当然啦，客户可以派生出新的classes，但 derived classes无法访问 base class中被封装的(即 private)成员，于是如此的"扩展机能"拥有的只是次级身份。此外一如条款7所说，并非所有classes 都被设计用来作为 base classes。

   ```c++
   //webbrowser.h
   namespace WebBrowserStuff
   {
       class WebBrowser { ... };
       //核心机能，例如几乎所有客户都需要的non-member函数
       ...
   }
   ```

   ```c++
   //webbrowserbookmarks.h
   namespace WebBrowserStuff
   {
       ...
   }
   ```

   ```c++
   //webbrowsercookies.h
   namespace WebBrowserStuff
   {
       ...
   }
   ```

   ```c++
   //main.cpp
   #include "webbrowser.h"
   #include "webbrowserbookmarks.h"
   #include "webbrowsercookies.h"
   
   ...
   ```

# 条款24：若所有参数皆需类型转换，请为此采用non-member函数

```c++
class Rational
{
    public:
    	Rational(int numerator = 0, int denominator = 1); //允许隐式转换
    	int numerator() const;
    	int denominator() const;
    	const Rational operator*(const Rational& lhs, const Rational& rhs)
    private:
    	...
};

Rational oneFourth(1, 4);
Rational result;
result = oneFourth * 2; //valid
result = 2 * oneFourth; //invalid
```

只有当参数被列于参数列(parameterlist)内，这个参数才是隐式类型转换的合格参与者。地位相当于“被调用之成员函数所隶属的那个对象”---即this 对象--的那个隐喻参数，绝不是隐式转换的合格参与者。这就是为什么上述第一次调用可通过编译，第二次调用则否，因为第一次调用伴随一个放在参数列内的参数，第二次调用则否。

```c++
class Rational
{
    ...
};

const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
```

operator* 是否应该成为Rational class 的一个友元函数呢？

就本例而言答案是否定的，因为operator*可以完全藉由Rational的public 接口完成任务，上面代码已表明此种做法。这导出一个重要的观察:member函数的反面是 non-member函数，不是friend函数。

1. 如果你需要为某个函数的所有参数(包括被 this 指针所指的那个隐喻参数)进行类型转换，那么这个函数必须是个non-member。
2. 如果函数不需要访问类的私有成员，只需要通过公共接口来操作对象，选择非成员函数。如果你希望保持类的封装性，避免直接暴露私有成员，使用非成员函数。如果一个函数必须访问类的私有成员，并且这种访问是必要的，且没有提供返回私有成员的接口，使用友元函数。如果该函数在逻辑上属于类的一部分，但因为需要对两个类进行对称操作而不能成为成员函数，且没有提供返回私有成员的接口，使用友元函数。

# 条款25：考虑写出一个不抛异常的 swap 函数

1. 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常；
2. 如果你提供一个 member swap，也该提供一个non-member swap用来调用前者。对于classes(而非templates)，也请特化 std::swap；
3. 调用 swap时应针对 std::swap使用 using声明式，然后调用 swap并且不带任何“命名空间资格修饰”；
4. 为“用户定义类型”进行 std templates 全特化是好的，但千万不要尝试在 std 内加入某些对 std而言全新的东西。

# 条款26：尽可能延后变量定义式的出现时间

1. 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。“尽可能延后”的真正意义：你不只应该延后变量的定义，直到非得使用该变量的前一刻为止，甚至应该尝试延后这份定义直到能够给它初值实参为止。如果这样，不仅能够避免构造(和析构)非必要对象，还可以避免无意义的 default 构造行为。更深一层说，以“具明显意义之初值”将变量初始化，还可以附带说明变量的目的。

2. 如果变量只在循环内使用，那么把它定义于循环外并在每次循环迭代时赋值给它比较好，还是该把它定义于循环内?

   循环内：1个构造函数+1个析构函数+n个赋值操作;

   循环外：n个构造函数+n个析构函数；

   如果 classes 的一个赋值成本低于一组构造+析构成本，做法 A大体而言比较高效。尤其当n值很大的时候。否则做法B或许较好。此外做法A造成名称w的作用域(覆盖整个循环)比做法B更大，有时那对程序的可理解性和易维护性造成冲突。因此除非(1)你知道赋值成本比“构造+析构”成本低，(2)你正在处理代码中效率高度敏感(performance-sensitive)的部分，否则你应该使用做法B。

# 条款27：尽置少做转型动作

1. 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_casts。如果有个设计需要转型动作，试着发展无需转型的替代设计；

   ```c++
   class Window { ... };
   class SpecialWindow : public Window
   {
   	public:
   		void blink();
   		...
   };
   
   //不希望如下的代码
   typedef std::vector<std::tr1::share_ptr<Window>> VPW;
   VPW winPtrs;
   ...
   for(VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) 
   {
       if(SpecialWindow * psw = dynamic_cast<SpecialWindow *>(iter -> get()))
           psw -> blink();
   }
   
   //使用类型安全容器
   typedef std::vector<std::tr1::share_ptr<SpecialWindow>> VPW;
   VPW winPtrs;
   for(VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) 
       (*iter) -> blink();
   
   //或者使用虚函数
   class Window 
   {
   	public:
       	virtual void blink() { } //缺省实现代码，“什么也不做”
       	...
   };
   class SpecialWindow : public Window
   {
   	public:
   		virtual void blink();
   		...
   };
   typedef std::vector<std::tr1::share_ptr<Window>> VPW;
   VPW winPtrs;
   ...
   for(VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) 
       (*iter) -> blink();
   
   //但必须避免连串的“dynamic_cast”，这样产生出来的代码又大又慢，而且基础不稳，因为每次window class 继承体系一有改变，所有这一类代码都必须再次检阅看看是否需要修改。
   
   class Window { ... }；
   ...
   typedef std::vector<std::tr1::share_ptr<Window>> VPW;
   VPW winPtrs;
   ...
   for(VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) 
   {
       if(SpecialWindow1 * psw = dynamic_cast<SpecialWindow *>(iter -> get())) { ...}
       else if(SpecialWindow2 * psw = dynamic_cast<SpecialWindow *>(iter -> get())) { ...}
       else if(SpecialWindow3 * psw = dynamic_cast<SpecialWindow *>(iter -> get())) { ...}
   }
   ```

2. 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数而不需将转型放进他们自己的代码内；

3. 宁可使用C++-styie(新式)转型，不要使用旧式转型。前者很容易辨识出来而且也比较有着分门别类的职掌。

# 条款28：避免返回 handles 指向对象内部成分

1. 避免返回handles（包括引用、指针、迭代器）指向对象内部，可防止外部世界修改对象内部，即使返回 const handles（包括引用、指针、迭代器）也是放松封装，因为对象内部对于外部世界仍然具有可读性。遵守这个条款可增加封装性，帮助 const 成员函数的行为像个 const ；

2. 避免按值返回对象内部成分，可以避免发生“虚吊号码牌”(dangling handles)；

   什么是虚吊号码牌，看以下代码：

   ```c++
   class Point
   {
       public:
       	point(int x, int y);
       	...
           void setX(int newVal);
       	void setY(int newVal);
      		...
   };
   struct RectData
   {
       Point ulhc;
       Point lrhc;
   };
   class Rectangle
   {
       ...
       private:
       	std::tr1::shared_ptr<RectData> pData;
   };
   class GUIObject { ... };
   const Rectangle boundingBox(const GUIObject& ogj);
   GUIObject* pgo;
   ...
   const Point* pUpperLeft = & (boundingBox(*pgo).upperLeft());
   ```

   对 boundingBox的调用获得一个新的、暂时的 Rectangle对象。这个对象没有名称，所以我们权且称它为temp。随后upperLeft作用于 temp 身上，返回一个reference 指向temp的一个内部成分，更具体地说是指向一个用以标示temp的Points。于是pUpperLeft指向那个Point对象。目前为止一切还好，但故事尚未结束，因为在那个语句结束之后，boundingBox的返回值，也就是我们所说的temp将被销毁，而那间接导致temp内的 Points析构。最终导致 pUpperLeft 指向一个不再存在的对象;也就是说一旦产出pUpperLeft的那个语句结束，pUpperLeft也就变成空悬、虚吊(dangling)!

3. 这并不意味你绝对不可以让成员函数返回handle，有时候你必须那么做，例如operator[]就允许你“摘采” strings 和 vectors的个别元素，而这些 operator[] 就是返回 references 指向“容器内的数据”(见条款 3)，那些数据会随着容器被销毁而销毁。尽管如此，这样的函数毕竟是例外，不是常态。

# 条款29：为“异常安全”而努力是值得的

1. 异常安全函数(Exception-safefunctions)即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证:基本型、强烈型、不抛异常型；

   1. 基本承诺:如果异常被抛出，程序内的任何事物仍然保持在有效状态下。没有任何对象或数据结构会因此而败坏，所有对象都处于一种内部前后一致的状态(例如所有的 class约束条件都继续获得满足)。然而程序的现实状态(exactstate)恐怕不可预料。举个例子，我们可以撰写 changeBackground 使得一旦有异常被抛出时，Prettyenu对象可以继续拥有原背景图像，或是令它拥有某个缺省背景图像，但客户无法预期哪一种情况。如果想知道，他们恐怕必须调用某个成员函数以得知当时的背景图像是什么。

   2. 强烈保证:如果异常被抛出，程序状态不改变。调用这样的函数需有这样的认知:如果函数成功，就是完全成功，如果函数失败，程序会回复到“调用函数之前”的状态。和这种提供强烈保证的函数共事，比和刚才说的那种只提供基本承诺的函数共事，容易多了，因为在调用一个提供强烈保证的函数后，程序状态只有两种可能:如预期般地到达函数成功执行后的状态，或回到函数被调用前的状态与此成对比的是，如果调用一个只提供基本承诺的函数，而真的出现异常，程序有可能处于任何状态--只要那是个合法状态。

   3. 不抛掷(nothrow)保证，承诺绝不抛出异常，因为它们总是能够完成它们原先承诺的功能。作用于内置类型(例如ints，指针等等)身上的所有操作都提供nothrow 保证。这是异常安全码中一个必不可少的关键基础材料。
      如果我们假设，函数带着“空白的异常明细”(emptyexception specification)者必为nothrow函数，似乎合情合理，其实不尽然。举个例子，考虑以下函数：

      int doSomething() throw();//注意“空白的异常明细“

      这并不是说 dosomething绝不会抛出异常，而是说如果dosomething抛出异常，将是严重错误，会有你意想不到的函数被调用1。实际上doSomething也许完全没有提供任何异常保证。函数的声明式(包括其异常明细--如果有的话)并不能够告诉你是否它是正确的、可移植的或高效的，也不能够告诉你它是否提供任何异常安全性保证。所有那些性质都由函数的实现决定，无关乎声明。

2. “强烈保证”往往能够以 copy-and-swap 实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义。

   假设有个 class用来表现夹带背景图案的GU菜单。这个 class 希望用于多线程环境，所以它有个互斥器(mutex)作为并发控制(concurrency control)之用：

   ```c++
   class PrettyMenu
   {
       public:
       	...
       	void changeBackground(std::istream& imgSrc)
       	...
       private:
       	Mutex mutex;
       	Image* bgImage;
       	int imageChanges;
   }
   ```

   下面是 PrettyMenu的 changeBackground 函数的一个可能实现：

   ```c++
   void PrettyMenu::changeBackground(std::istream& imgSrc)
   {
       lock(&mutex);
       delete bgImage;
       ++imagechanges;
       bgImage = new Image(imgSrc);
       unlick(&mutex);
   }
   ```

   提供强烈保证的版本：

   ```c++
   struct PMImpl
   {
       std::tr1::shared_ptr<Image> bgImage;
       int imagechanges;
   };
   
   //让PMImpl成为一个struct而不是一个class，这是因为PrettyMenu的数据封装性已经由于“pImpl是private”而获得了保证。如果令PMImpl为一个 class，虽然一样好，有时候却不太方便(但也保持了面向对象纯度)。如果你要，也可以将 PMImpl嵌套于PrettyMenu内，但打包问题(packaging，例如“独立撰写异常安全码”)是我们这里所挂虑的事。
   
   class PrettyMenu
   {
       ...
       private:
       	Mutex mutex;
       	std::tr1::shared_ptr<PMImpl> pImpl; //pimpl idiom手法
   };
   void PrettyMenu::changeBackground(std::istream& imgSrc)
   {
       using std::swap;
       Lock ml(&mutex); //获得互斥器并确保释放
       std::tr1::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl)); //智能指针，自动释放，避免泄露资源
       pNew -> bgImage.reset(new Image(imgSrc));
       ++pNew -> imageChanges; //设置新背景成功后累加
       swap(pImpl, pNew); //必须提供不抛出异常的置换操作
   }
   ```

   "copy-and-swap"策略是对对象状态做出“全有或全无”改变的一个很好办法但一般而言它并不保证整个函数有强烈的异常安全性。为了解原因，让我们考虑changeBackground的一个抽象概念:someFunc。它使用copy-and-swap策略，但函数内还包括对另外两个函数f1和 f2的调用:

   ```c++
   void someFunc()
   {
   	...	//对 local状态做一份副本
   	f1();
   	f2();
   	...	//将修改后的状态置换过来
   }
   ```

   很显然，如果f1或f2的异常安全性比“强烈保证”低，就很难让somerunc成为“强烈异常安全”。举个例子，假设 f1只提供基本保证，那么为了让 somerunc提供强烈保证，我们必须写出代码获得调用1之前的整个程序状态、捕捉1的所有可能异常、然后恢复原状态。
   如果 f1和 2都是“强烈异常安全”，情况并不就此好转。毕竟如果 1圆满结束，程序状态在任何方面都可能有所改变，因此如果2随后抛出异常，程序状态和 somerunc被调用前并不相同，甚至当f2没有改变任何东西时也是如此。
   问题出在“连带影响”(side efects)。如果函数只操作局部性状态(local state例如 somerunc 只影响其“调用者对象”的状态)，便相对容易地提供强烈保证。但是当函数对“非局部性数据”(non-local data)有连带影响时，提供强烈保证就困难得多。举个例子，如果调用f1带来的影响是某个数据库被改动了，那就很难让somerunc具备强烈安全性。一般而言在“数据库修改动作”送出之后，没有什么做法可以取消并恢复数据库旧观，因为数据库的其他客户可能已经看到了这一笔新数据。
   这些议题想必会阻止你为函数提供强烈保证--即使你想那么做。另一个主题是效率。copy-and-swap的关键在于“修改对象数据的副本，然后在一个不抛异常的函数中将修改后的数据和原件置换”，因此必须为每一个即将被改动的对象做出一个副本，那得耗用你可能无法(或无意愿)供应的时间和空间。当“强烈保证”不切实际时，你就必须提供“基本保证”；

3. 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。

# 条款30：透彻了解 inlining 的里里外外

有时候虽然编译器有意愿inlining某个函数，还是可能为该函数生成一个函数本体。举个例子，如果程序要取某个inline 函数的地址，编译器通常必须为此函数生成一个outlined函数本体。毕竟编译器哪有能力提出一个指针指向并不存在的函数呢?与此并提的是,编译器通常不对“通过函数指针而进行的调用"实施inlining这意味对 inline 函数的调用有可能被 inlined，也可能不被 inlined，取决于该调用的实施方式:

```c++
inline void f() { ... }
void (*pf)() = f;
...
f(); //inlined
pf(); //获取不被inlined
```

考虑以下 Derived class 构造函数：

```c++
class Base
{
    public:
    	...
    private:
    	std::string bm1, bm2;
};
class Derived : public Base
{
    public:
    	Derived() { }
    	...
    private:
    	std::string dm1, dm2, dm3;
};
```

这个构造函数看起来是inlining 的绝佳候选人，因为它根本不含任何代码。但表面上看起来为空的Derived构造函数所产生的代码，相当于以下所列（这段代码并不能代表编译器真正制造出来的代码，因为真正的编译器会以更精致复杂的做法来处理异常。尽管如此，这已能准确反映 Derived的空白构造函数必须提供的行为）:

```c++
Derived::Derived()
{
    Base::Base();
    try { dm1.std::string::string(); }
    catch (...)
    {
        Base::~Base();
        throw;
    }
    try { dm2.std::string::string(); }
    catch (...)
    {
    	dm1.std::string::~string();
        Base::~Base();
        throw;
    }
    try { dm3.std::string::string(); }
    catch (...)
    {
    	dm2.std::string::~string();
    	dm1.std::string::~string();
        Base::~Base();
        throw;
    }
}
```

相同理由也适用于 Base构造函数，所以如果它被inlined，所有替换“Base构造函数调用”而插入的代码也都会被插入到“Derived 构造函数调用”内(因为Derived构造函数调用了 Base构造函数)。如果string构造函数恰巧也被 inlined,Derived 构造函数将获得五份“string 构造函数代码”副本，每一份副本对应于Derived对象内的五个字符串(两个来自继承，三个来自自己的声明)之一。现在或许很清楚了，“是否将 Derived 构造函数 inline 化”并非是个轻松的决定。类似思考也适用于 Derived析构函数，在那儿我们必须看到“被 Derived构造函数初始化的所有对象”被一一销毁，无论以哪种方式进行。

1. 将大多数 inlining 限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级(binary upgradability)更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化；
2. Inline 函数通常一定被置于头文件内,因为大多数建置环境(buildenvironments)在编译过程中进行 inlining，而为了将一个“函数调用”替换为“被调用函数的本体”，编译器必须知道那个函数长什么样子。不要只因为function templates 出现在头文件，就将它们声明为inline，需要考虑成本。
3. 程序库设计者必须评估“将函数声明为imnline”的冲击:inline函数无法随着程序库的升级而升级。换句话说如果f是程序库内的一个inline函数，客户将“f函数本体”编进其程序中，一旦程序库设计者决定改变，所有用到的客户端程序都必须重新编译。这往往是大家不愿意见到的。然而如果是non-inline函数，一旦它有任何修改，客户端只需重新连接就好，远比重新编译的负担少很多。如果程序库采取动态连接，升级版函数甚至可以不知不觉地被应用程序吸纳。

# 条款31：将文件间的编译依存关系降至最低

```cpp
#include <string>
#include "date.h"
#include "address.h"

class Person
{
    public:
    	Person(const std::string& name, const Data& birthday, const Address& addr);
    	std::string name() const;
    	std::string birthDate() const;
    	std::string address() const;
    	...
    private:
    	std::string theName; //实现细目
    	Date theBirthDate; //实现细目
    	Address theAddress; //实现细目
};
```

如果这些头文件中有任何一个被改变，或泽依存关系(compilation dependency这些头文件所倚赖的其他头文件有任何改变，那么每一个含入Person class 的文件就得重新编译，任何使用 Person class的文件也必须重新编译。

1. 支持“编译依存性最小化”的一般构想是:相依于声明式，不要相依于定义式。基于此构想的两个手段是 Handle classes 和Interface classes，它们解除了接口和实现之间的耦合关系。

   1. Handle classes：

      ```cpp
      namesapce std{
      	class string;
      }
      
      class PersonImpl;
      class Date;
      class Address;
      
      class Person
      {
          public:
          	Person(const std::string& name, const Data& birthday, const Address& addr);
          	std::string name() const;
          	std::string birthDate() const;
          	std::string address() const;
          	...
          private:
          	std::tr1::shared_ptr<PersonImpl> pImpl;
      };
      ```

      这个想法存在两个问题：

      第一，string不是个class，它是个typedef(定义为basic string< char >)。因此上述针对 string而做的前置声明并不正确;正确的前置声明比较复杂，因为涉及额外的templates。然而那并不要紧，因为你本来就不该尝试手工声明一部分标准程序库。你应该仅仅使用适当的#includes 完成目的。标准头文件不太可能成为编译瓶颈，特别是如果你的建置环境允许你使用预编译头文件(precompiled headers)。如果解析(parsing)标准头文件真的是个问题，你可能需要改变你的接口设计，避免使用标准程序库中“引发不受欢迎之#includes那一部分。

      第二，编译器必须在编译期间知道对象的大小：

      ```cpp
      int main()
      {
      	int x;
      	Person p(params);
      	...
      }
      ```

      当编译器看到p的定义,它也知道必须分配足够空间以放置一个person，编译器获得对象大小信息的唯一办法就是询问class定义式。然而如果class定义式可以合法地不列出实现细目，编译器如何知道该分配多少空间?

      针对Person我们可以这样做:把Person分割为两个classes，一个只提供接口，另一个负责实现该接口。如果负责实现的那个所谓classes ,implementation class 取名为 PersonImpl，Person 将定义如下:

      ```c++
      #include <string>
      #include <memory>
      
      class PersonImpl;
      class Date;
      class Address;
      
      class Person
      {
          public:
          	Person(const std::string& name, const Data& birthday, const Address& addr);
          	std::string name() const;
          	std::string birthDate() const;
          	std::string address() const;
          	...
          private:
          	std::tr1::shared_ptr<PersonImpl> pImpl;
      };
      ```

      ```c++
      #include "Person.h"
      #include "PersonImpl.h"
      
      Person::Person(const std::string& name. const Date& borthday. const Address& addr) : pImpl(new PersonImpl(name, birthday, addr)) {}
      
      std::string Person::name() const
      {
      	return pImpl->name();
      }
      ...
      ```

      这个分离的关键在于以“声明的依存性”替换“定义的依存性”，那正是编译依存性最小化的本质:现实中让头文件尽可能自我满足，万一做不到，则让它与其他文件内的声明式(而非定义式)相依。其他每一件事都源自于这个简单的设计策路:

      1. 如果使用objectreferences 或object pointers 可以完成任务，就不要使用objects。你可以只靠一个类型声明式就定义出指向该类型的references和pointers;但如果定义某类型的 objects，就需要用到该类型的定义式。

      2. 如果能够，尽量以 class 声明式替换 class 定义式。注意，当你声明一个函数而它用到某个class时，你并不需要该class 的定义;纵使函数以 byvalue方式传递该类型的参数(或返回值)亦然:

         ```
         class Date;	//class 声明式。
         Date today();						//没问题 - 这里并不需要
         void clearAppointments(Date dy;		// Date的定义式。
         ```

      3. 为声明式和定义式提供不同的头文件。为了促进严守上述准则，需要两个头文件，一个用于声明式，一个用于定义式。当然，这些文件必须保持一致性，如果有个声明式被改变了,两个文件都得改变。因此程序库客户应该总是#include一个声明文件而非前置声明若干函数，程序库作者也应该提供这两个头文件。举个例子，Date的客户如果希望声明today和clearAppointments，他们不该像先前那样以手工方式前置声明Date，而是应该#include 适当的、内含声明式的头文件:

         ```cpp
         #include "datefwd.h"	//这个头文件内声明(但未定义)class Date。
         Date today();			//同前。
         void clearAppointments(Date d);
         ```

      成员函数必须通过 implementation pointer 取得对象数据。那会为每一次访问增加一层间接性。而每一个对象消耗的内存数量必须增加implementation pinter 的大小。最后,implementation pointer 必须初始化(在 Handleclass构造函数内)，指向一个动态分配得来的immplementation object，所以你将蒙受因动态内存分配(及其后的释放动作)而来的额外开销，以及遭遇bad alloc异常(内存不足)的可能性。

   2. Interface classes：

      令Person 成为一种特殊的 abstract baseclass(抽象基类)，称为Interface class。这种 class的目的是详细--描述 derivedclasses的接口(见条款34)，因此它通常不带成员变量，也没有构造函数，只有个 virtual 析构函数(见条款 7)以及一组 pure virtual函数，用来叙述整个接口。这个 class 的客户必须以Person的 pointers和references 来撰写应用程序，因为它不可能针对“内含 pure virtual函数”的Person classes 具现出实体。(然而却有可能对派生自Person的classes具现出实体，详下。)就像Handle classes 的客户一样，除非Interface class 的接口被修改否则其客户不需重新编译。Interface class 的客户必须有办法为这种 class 创建新对象。他们通常调用一个特殊函数，此函数扮演“真正将被具现化”的那个derivedclasses的构造函数角色。这样的函数通常称为factory(工厂)函数(见条款13)或virual 构造函数。它们返回指针(或更为可取的智能指针，见条款18)，指向动态分配所得对象，而该对象支持Interface class的接口。这样的函数又往往在Interface class 内被声明为

      ```c++
      //Person.h
      class Date;
      class Address;
      
      class Person
      {
          public:
          	virtual ~Person();
          	virtual std::string name() const = 0;
          	virtual std::string birthDate() const = 0;
          	virtual std::string addresss() const = 0;
          	static std::tr1::shared_ptr<Person> Person::create(const std::string& name, const Date& birthday, const Address& addr);
          	...
      };
      
      //Person.cpp
      #include "Person.h"
      
      class Date;
      class Address;
      
      std::tr1::shared_ptr<Person> Person::create(const std::string& name, const Date& birthday, const Address& addr)
      {
          return std::tr1::shared_ptr<Person> (new RealPerson(name, birthday, addr));
      }
      
      //RealPerson.h
      #include "Person.h"
      #include <string>
      
      class Date;
      class Address;
      
      class RealPerson : public Person
      {
          public:
          	RealPerson(const std::string& name, const Date& birthday, const Address& addr) : theName(name), theBirthDate(birthday), theAddress(addr) {}
          	virtual ~RealPerson() {}
          	std::string name() const;
          	std::string birthDate() const;
          	std::string address() const;
          private:
          	std::string theName;
          	Date theBirthDate;
          	Address theAddress;
      };
      
      //RealPerson.cpp
      ...
      //main.cpp
      #include "Person.h"
      #include <string>
      #include <iostream>
      ...
      std::string name;
      date dateOfBirth;
      Address address;
      ...
      std::tr1::shared_ptr<Person> pp(Person::create(namee, dateOfBiorth, address))
      ...
      ```

      由于每个函数都是 virtual，所以你必须为每次函数调用付出一个间接跳跃(indirectjump)成本(见条款7)。此外 Interface class 派生的对象必须内含一个 vptr(virtual table pointer，再次见条款7)，这个指针可能会增加存放对象所需的内存数量--实际取决于这个对象除了Interface class 之外是否还有其他 virtual 函数来源。

2. 程序库头文件应该以“完全且仅有声明式”(full and declaration-only forms)的形式存在。这种做法不论是否涉及templates 都适用。

3. 在程序发展过程中使用 Handle classes和 Interface classes 以求实现码有所变化时对其客户带来最小冲击。而当它们导致速度和/或大小差异过于重大以至于 classes 之间的耦合相形之下不成为关键时，就以具象类(concreteclasses)替换Handle classes和Interfaceclasses .

# 条款32：确定你的 public 继承塑模出 is-a 关系

public 继承意味is-a。适用于base classes身上的每一件事情一定也适用于derived classes 身上，因为每一个 derived class 对象也都是一个 base class 对象。

# 条款33：避免遮掩继承而来的名称

1. Derived classes内的名称会遮掩baseclasses内的名称。在public 继承下从来没有人希望如此；
2. 为了让被遮掩的名称再见天日，可使用using声明式或转交函数(forwardingfunctions )

考虑下面一个例子：

```c++
//Base的作用域：x, mf1(2个函数)，mf2(1个函数)，mf3(2个函数)，包含Derived的作用域
class Base
{
    private:
    	int x;
    public:
    	virtual void mf1() = 0;
    	virtual void mf1(int)
    	virtual void mf2();
    	void mf3();
    	void mf3(double);
    	...
};

//Deried的作用域：mf1(1个函数)，mf3(1个函数)，mf4(1个函数)
class Deried : public Base
{
    public:
    	virtual void mf1();
    	void mf3();
    	void mf4();
    	...
};
```

以作用域为基础的“名称遮掩规则”（逐渐向作用域更大的范围寻找，直到找到所需名称声明式）并没有改变，因此base class内所有名为mf1和 mf3的函数都被 derivedclass内的mf1和mf3 函数遮掩掉了。从名称査找观点来看Base::mf1和 Base::mf3不再被 Derived 继承！

```c++
Dreived d;
int x;
...
d.mf1(); //调用Derived::mf1
d.mf1(x); //Error
d.mf2(); //调用Base::mf2
d.mf3(); //调用Derived::mf3
d.mf3(x); //Error
```

这些行为背后的基本理由是为了防止你在程序库或应用框架(applicationftamework)内建立新的 derived class 时附带地从疏远的base classes 继承重载函数不幸的是你通常会想继承重载函数。实际上如果你正在使用public继承而又不继承那些重载函数，就是违反 base和 derived classes 之间的is-a关系，而条款 32 说过is-a是 public 继承的基石。因此你几乎总会想要推翻(override)C++对“继承而来的名称”的缺省遮掩行为。

使用 using声明式达成目标:

```c++
//Base的作用域：x, mf1(2个函数)，mf2(1个函数)，mf3(2个函数)，包含Derived的作用域
class Base
{
    private:
    	int x;
    public:
    	virtual void mf1() = 0;
    	virtual void mf1(int)
    	virtual void mf2();
    	void mf3();
    	void mf3(double);
    	...
};

//Deried的作用域：mf1(2个函数)，mf3(2个函数)，mf4(1个函数)
class Deried : public Base
{
    public:
    	using Base::mf1;
    	using Base::mf3;
    	virtual void mf1();
    	void mf3();
    	void mf4();
    	...
};
```

# 条款34：区分接口继承和实现继承

1. 接口继承和实现继承不同。在public 继承之下,derived classes总是继承 base class的接口；

2. 简朴的(非纯)impure virtual函数具体指定接口继承及缺省实现继承；

   ```c++
   class Airport { ... };
   class Airplane
   {
       public:
       	virtual void fly(const Airport& destination);
       	...
   };
   void Airplane::fly(const Airport& destination)
   {
       缺省代码，将飞机飞往指定地方
   }
   //A和B都继承Airplane的飞行方式，但原则上不同飞机有不同的飞行方式，因此声明fly()为虚函数
   class ModelA : public Airplane { ... };
   class ModelB : public Airplane { ... };
   //C有自己的飞行方式，因此需要重新定义fly()函数，但因为某种原因遗忘，此时C错误的使用了Airplane的飞行方式
   class ModelC : public Airplane { ... };
   
   ```

   问题不在 Airplane::fly有缺省行为，而在于Modelc在未明白说出“我要的情况下就继承了该缺省行为。因此可以切断“virtual函数接口”和其“缺省实现”之间的连接。

   下面是一种做法:

   ```c++
   class Airplane
   {
       public:
       	virtual void fly(const Airport& destination) = 0;
       	...
       protected:
       	void defaultFly(const Airport& destination);
   };
   void Airplane::defaultFly(const Airport& destination)
   {
       缺省代码，将飞机飞往指定地方
   }
   class ModelA : public Airplane
   {
   	public:
   		virtual void fly(const Airport& destination)
   		{
               defaultFly(destination);
   		}
   		...
   };
   class ModelB : public Airplane
   {
       public:
   		virtual void fly(const Airport& destination)
   		{
               defaultFly(destination);
   		}
   		...
   };
   class ModelC : public Airplane
   {
       public:
   		virtual void fly(const Airport& destination)
   		...
   };
   void ModelC::fly(const Airport& destination)
   {
       C型飞机飞至指定目的地
   }
   ```

3. pure virtual函数只具体指定接口继承，但是也可在基类中给他定义；

   ```c++
   class Airplane
   {
       public:
       	virtual void fly(const Airport& destination) = 0;
       	...
       protected:
       	void defaultFly(const Airport& destination);
   };
   void Airplane::fly(const Airport& destination)
   {
       缺省代码，将飞机飞往指定地方
   }
   class ModelA : public Airplane
   {
   	public:
   		virtual void fly(const Airport& destination)
   		{
               Airplane::fly(destination);
   		}
   		...
   };
   class ModelB : public Airplane
   {
       public:
   		virtual void fly(const Airport& destination)
   		{
               Airplane::fly(destination);
   		}
   		...
   };
   class ModelC : public Airplane
   {
       public:
   		virtual void fly(const Airport& destination)
   		...
   };
   void ModelC::fly(const Airport& destination)
   {
       C型飞机飞至指定目的地
   }
   ```

4. non-virtual函数具体指定接口继承以及强制性实现继承，它的意义是不变性(invariant)凌驾特异性(specialization),。

# 条款35：考虑 virtual函数以外的其他选择

假设你正在写一个视频游戏软件，你打算为游戏内的人物设计一个继承体系。你决定提供一个成员函数 healthValue，它会返回一个整数，表示人物的健康程度。由于不同的人物可能以不同的方式计算他们的健康指数，将healthValue声明为 virtual 似乎是再明白不过的做法，但认真考虑以下替代做法：

1. 使用 non-virtual interface(NVI)手法，那是 Template Method 设计模式的一种特殊形式。它以 public non-virtual 成员函数包裹较低访问性(private或 protected)的 virtual 函数；

   ```c++
   class GameCharacter
   {
       public:
       	int healthValue() const
       	{
               ...	//事前工作
               int retVal = doHealthValue();
               ...	//事后工作
               return retVal;
       	}
       	...
       private:
       	virtual int doHeathValue() const
       	{
               ...
       	}
   };
   ```

   NVI手法的一个优点隐身在上述代码注释“做一些事前工作”和“做一些事后工作”之中。那些注释用来告诉你当时的代码保证在“virtual函数进行真正工作之前和之后”被调用。这意味外覆器(wrapper)确保得以在一个virtual 函数被调用之前设定好适当场景，并在调用结束之后清理场景。“事前工作”可以包括锁定互斥器(1ockingamutex)、制造运转日志记录项(10gentry)、验证 class 约束条件、验证函数先决条件等等。“事后工作”可以包括互斥器解除锁定(unlockingamutex)、验证函数的事后条件、再次验证 class约束条件等等。如果你让客户直接调用 virtual函数，就没有任何好办法可以做这些事。

2. 将 virtual函数替换为“函数指针成员变量”，这是Sirafegy设计模式的一种分解表现形式；

   ```c++
   class GameCharacter;
   int defaultHealthCalc(const GameCharacter&);
   class GameCharacter
   {
       public:
       	typedef int (*HealthCalcFunc)(const GameCharacter&);
       	explict GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf) {}
       	int healthValue() const
       	{
               return healthFunc(*this);
       	}
       	...
       private:
       	HealthCalcFunc healthFunc;
   }
   
   class EvilBadGuy : public GameCharacter
   {
   	public:
   		explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc) : GameCharacter(hcf) { ... }
   		...
   };
   
   int loseHealthQuickly(const GameCharacter&);
   int loseHealthSlowly(const GameCharacter&);
   
   //同一人物类型之不同实体可以有不同的健康计算函数
   EvilBadGuy ebg1(loseHealthQuickly);
   EvilBadGuy ebg2(loseHealthSlowly);
   
   //某已知人物之健康指数计算函数可在运行期变更。例如Gamecharacter可提供一个成员函数 setHealthcalculator，用来替换当前的健康指数计算函数。
   ```

3. 以tr1::function成员变量替换virtual函数，因而允许使用任何可调用物(callable entity)搭配一个兼容于需求的签名式，trl::function是个 template，以其目标函数的签名(targetfumction signature)为参数:。这也是 Strategy 设计模式的某种形式；

   ```c++
   class GameCharacter;
   int defaultHealthCalc(const GameCharacter&);
   class GameCharacter
   {
       public:
       	typedef std::tr1:function<int (const GameCharacter&)> HealthCalcFunc;
       	explict GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf) {}
       	int healthValue() const
       	{
               return healthFunc(*this);
       	}
       	...
       private:
       	HealthCalcFunc healthFunc;
   }
   ```

   和前一个设计(其 Gamecharacter持有的是函数指针)比较，这个设计几乎相同。唯一不同的是如今 Gamecharacter 持有一个 tr1::function 对象，相当于一个指向函数的泛化指针，更富弹性地接受任何可调用物(callable entity)，只要这个可调用物接受一个 GameCharacter& 或任何可被转换为 GameCharacter& 的东西，并返回一个 int或任何可被转换为int的东西。这个改变如此细小，我总说它没有什么外显影响，除非客户在“指定健康计算函数”这件事上需要更惊人的弹性；

   ```c++
   short calcHealth(const GameCharacter&);
   
   struct HealthCalculator
   {
       int operator()(const GameCharacter&) const
       {
           ...
       }
   };
   
   class GameLevel
   {
       public:
       	//GameLevel::health 宣称它自己接受一个参数(那是个 reference指向 Gamecharacter)，但它实际上接受两个参数，因为它也获得一个隐式参数GameLevel，也就是 this所指的那个。
       	float health(const GameCharacter&) const;
       ...
   };
   
   class EvilBadGuy : public GameCharacter { ... };
   
   class EyeCandyCharacter ecc1() : public GameCharacter { ... };
   
   EviBadGuy ebg1(calcHealth);
   EyeCandyCharacter ecc1(HealthCalculator());
   ...
   // Gamecharacters 的健康计算函数只接受单一参数:GameCharacter(这个对象将被计算出健康指数)。如果我们使用GameLevel::health作为 ebg2 的健康计算函数，我们必须以某种方式转换它，使它不再接受两个参数(一个 GameCharacter和一个GameLevel)，转而接受单一参数(一个 Gamecharacter)。在这个例子中我们必然会想要使用currentLevel作为“ebg2的健康计算函数所需的那个 GameLevel对象”,于是我们将 currentLevel绑定为 GameLevel对象，让它在“每次 GameLevel::health被调用以计算 ebg2 的健康”时被使用。那正是tr1::bind的作为:它指出ebg2的健康计算函数应该总是以 currentLevel作为 GameLevel对象。当ebg2.healthValue()被调用时，它将会调用currentLevel.health(ebg2)来计算健康值.
   GameLevel currentLevel;
   EviBadGuy ebg2(std::tr1::bind(&GameLevel::health, currentLevel, _1));
   ```

4. 将继承体系内的 virtual函数替换为另一个继承体系内的virtual 函数。这是Strategy 设计模式的传统实现手法。这个解法的吸引力在于，熟悉标准 Strategy模式的人很容易辨认它，而且它还提供“将一个既有的健康算法纳入使用”的可能性--只要为 HealthcalcFunc 继承体系添加一个 derived class 即可。

# 条款36：绝不重新定义继承而来的 non-virtual函数

绝不重新定义继承而来的 non-virtual函数

```c++
class B
{
   	public:
   		void mf();
   		...
};
class D : public B
{
   	public:
   		void mf();
   		...
}
D x;
B* pB = &x;
pB->mf(); //调用B::mf
D* pD = &x;
pD->mf(); //调用D::mf
```

造成此一两面行为的原因是，non-virtual函数如B::mf和D::mf都是静态绑定statically bound，见条款 37)。这意思是，由于pB被声明为一个 pointer-to-B，通过pB 调用的 non-virtual函数永远是B所定义的版本，即使pB指向一个类型为“B派生之 class”的对象。但另一方面，virtual函数却是动态绑定(dynamicallybound，见条款37)，所以它们不受这个问题之苦。如果 mf是个 virtual 函数,不论是通过 pB或pD调用 mf,都会导致调用 D::mf，因为pB和pD真正指的都是一个类型为D的对象。

# 条款37：绝不重新定义继承而来的缺省参数值

绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而 virtual 函数应该重写的定义却是动态绑定，看下面这个例子：

```c++
class Shape
{
    public:
    	enum ShapeColor { Red, Green, Blue};
    	virtual void draw(ShapeColor color = Red) const = 0;
    	...
};
class Rectangle : public Shape
{
    public:
    	virtual void draw(ShapeColor color = Green) const;
    	...
};
class Circle : public Shape
{
    public:
    	virtual void draw(ShapeColor color) const;
}

// ps, pc和 pr 都被声明为 pointer-to-shape 类型，所以它们都以它为静态类型。
Shape* ps;
Shape* pc = new Circle;
Shape* pr = new Rectangle;

//Virtual函数系动态绑定而来，意思是调用一个 virtual 函数时，究竟调用哪一份函数实现代码，取决于发出调用的那个对象的动态类型。
pc->draw(Shape::Red);
pr->draw(Shape::Red);

//pr的动态类型是Rectangle*，所以调用的是Rectangle的 virtual函数，一如你所预期。Rectangle::draw函数的缺省参数值应该是GREEN，但由于pr的静态类型是 shape*，所以此一调用的缺省参数值来自Shape class 而非Rectangle class!
pr->draw; //调用Rectangle::draw(Shape::Red)
```

为什么这么做，答案在于运行期效率！如果缺省参数值是动态绑定,编译器就必须有某种办法在运行期为virtual 函数决定适当的参数缺省值。这比目前实行的“在编译期决定”的机制更慢而且更复杂。为了程序的执行速度和编译器实现上的简易度，C++做了这样的取舍，其结果就是你如今所享受的执行效率。但如果你没有注意本条款所揭示的忠告，很容易发生混淆。

解决方案：NVI手法，

```c++
class Shape
{
    public:
    	enum ShapeColor { Red, Green, Blue};
    	void draw(ShapeColor color = Red) const
        {
            doDraw(color)
        }
    private:
    	virtual void doDraw(ShapeColor color) const = 0;
};
class Rectangle : public Shape
{
    public:
    	...
    private:
    	virtual void doDraw(ShapeColor color) const; //不指定缺省参数值，指定也没用
    	...
};
```

# 条款38：通过复合塑模出 has-a或"根据某物实现出"

1. 复合的意义和public 继承完全不同；
2. 在应用域（程序中的对象其实相当于你所塑造的世界中的某些事物，例如人、domains )汽车、一张张视频画面等等，复合意味has-a(有一个)。在实现域（实现细节上的人工制品,像是缓冲区(buffers)、互斥器(mutexes)、查找树(search trees)等等），复合意味 is-implemented-in-terms-of(根据某物实现出)。

# 条款39：明智而审慎地使用 private 继承

1. Private 继承意味is-implemented-in-termsof(根据某物实现出)。当derived class 需要访问 protected base class 的成员，或需要重新定义继承而来的 virtual函数时，这么设计是合理的，并且private继承可以造成empty base最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。

   empty base最优化：C++裁定凡是独立(非附属)对象都必须有非零大小，所以如果你这样做：

   ```c++
   class Empty {  }; //没有数据
   class HoldsAnInt
   {
       private:
       	int x;
       	Empty e;
   };
   ```

   你会发现 sizeof(HoldsAnInt) > sizeof(int)；

   ```c++
   class HoldsAnInt : private Empty
   {
       private:
       	int x;
   };
   ```

   几乎可以确定 sizeof(HoldsAnInt) == sizeof(int)，这就是所谓的EBO（空白基类最优化），但现实中的"empty" classes并不真的是empty。虽然它们从未拥有 non-static 成员变量，却往往内含typedefs,enums,static 成员变量，或non-virtual函数。STL 就有许多技术用途的emptyclasses，其中内含有用的成员(通常是typedefs)，包括 baseclasses unary function和 binary function，这些是“用户自定义之函数对象”通常会继承的 classes。感谢 EBO 的广泛实践,使这样的继承很少增加 derived ciasses的大小。

2. 若无需考虑上述特殊情况，复合是更好的选择;

   ```c++
   class Widget
   {
       private:
       	class WidgetTimer : public Timer
       	{
           	public:
           		virtual void onTick() const;
                   ...
       	};
       	WidgetTimer timer;
       	...
   }
   ```

   首先，你或许会想设计 widget使它得以拥有 derived classes，但同时你可能会想阻止 derived classes 重新定义 onTick。如果 Widget继承自 Timer，上面的想法就不可能实现，即使是 private 继承也不可能。其次，你或许会想要将widget的编译依存性降至最低。如果widget继承Timer，当 widget 被编译时 Timer的定义必须可见，所以定义 widget 的那个文件恐怕必须#include rimer.h。但如果 Widgetrimer 移出 widget 之外而 widget 内含指针指向一个 widgetrimer，widget 可以只带着一个简单的 widgetTimer 声明式不再需要#include任何与Timer有关的东西。对大型系统而言，如此的解耦(decouplings)可能是重要的措施。关于编译依存性的最小化，详见条款31。

# 条款40：明智而审慎地使用多重继承

1. 多重继承比单一继承复杂。它可能导致新的歧义性，以及对virtual继承的需要;

   ```c++
   class BorrowableItem
   {
       public:
       	void checkOut();
       	...
   };
   
   class ElectronicGadget
   {
       private:
       	bool checkOut() const;
   };
   
   class MP3Player : public BorrowableItem, public ElectronicGadget { ... };
   MP3Player mp;
   mp.checkOut(); // checkout 的调用是歧义(模棱两可)的，即使两个函数之中只有一个可取用(BorrowableItem内的check0ut是public，ElectronicGadget内的却是 private)。这与C++用来解析(resolving)重载函数调用的规则相符:在看到是否有个函数可取用之前，C++首先确认这个函数对此调用之言是最佳匹配找出最佳匹配函数后才检验其可取用性。本例的两个checkouts有相同的匹配程度(译注:因此才造成歧义),没有所谓最佳匹配。因此ElectronicGadget::checkout的可取用性也就从未被编译器审查。
   
   mp.BorrowableItem::checkOut(); //valid
   ```

   钻石型多重继承：

   ```c++
   class File { ... };
   class InputFile : public File { ... };
   class OuptFile : public File { ... };
   class IOFile : public InputFile, public OutputFile { ... };
   ```

   某个角度说，IOFile 从其每一个base class 继承一份，所以其对象内应该有两份 fileName 成员变量。但从另一个角度说，简单的逻辑告诉我们，IOFile对象只该有一个文件名称，所以它继承自两个base classes 而来的 fileName 不该重复。为了这样做，必须令所有继承自它的classes采用“virtual继承”：

   ```c++
   class File { ... };
   class InputFile : virtual public File { ... };
   class OuptFile : virtual public File { ... };
   class IOFile : public InputFile, public OutputFile { ... };
   ```

   virtual base 的初始化责任是由继承体系中的最低层(most derived)class负责，这暗示(1)classes若派生自 virtual bases而需要初始化，必须认知其 virtual bases，不论那些 bases 距离多远，(2)当一个新的 derived class加入继承体系中，它必须承担其 virtualbases(不论直接或间接)的初始化责任。

   非必要不使用 virtualbases。平常请使用 non-virtual 继承。如果你必须使用 virtualbase classes，尽可能避免在其中放置数据。

2. virtual 继承会增加大小、速度、初始化(及赋值)复杂度等等成本。如果 virtualbase classes不带任何数据，将是最具实用价值的情况；

3. 多重继承的确有正当用途。其中一个情节涉及“public继承某个Interface class和“private 继承某个协助实现的class”的两相组合。

   ```c++
   class IPerson
   {
   	public:
       	static std::tr1::shared_ptr<IPerson> makePerson(DatabaseID personIdentifier)
   		virtual ~IPerson();
   		virtual std::string name() const = 0;
   		virtual std::string birthDate() const = 0;
   };
   
   class DatabaseID { ... };
   
   class PersonInfo
   {
   	public:
   		explicit PersonInfo(DatabaseID pid);
   		virtual ~PersonInfo();
   		virtual const char* theName() const;
   		virtual const char* theBirthDate() const;
       private:
   		virtual const char* valueDelimOpen() const;
   		virtual const char* valueDelimClose() const;
   		...
   };
   
   class CPerson : public IPerson, private PersonInfo
   {
   	public:
   		explicit CPerson(DatabaseID pid) : PersonInfo(pid) {}
   		virtual std::string name() const
   		{
   			return PersonInfo::theName();
   		}
   		virtual std::string birthDate() const
   		{
   			return PersonInfo::theBirthDate();
   		}
   	private:
   		const char* valueDelimOpen() const { return ""; }
   		const char* valueDelimClose() const { return ""; }
   }
   ```

   CPerson和 PersonInfo的关系是：PersonInfo刚好有若干函数可帮助 CPerson比较容易实现出来。就这样。它们的关系因此是is-implemented-in-terms-of(根据某物实现出),而我们知道这种关系可以两种技术实现:复合(见条款38)和private继承(见条款 39)。条款39指出复合通常是较受欢迎的做法，但如果需要重新定义virtual函数，那么继承是必要的。本例之中CPerson需要重新定义valueDelimopen和 valueDelimclose，所以单纯的复合无法应付。最直接的解法就是令 CPerson以 private 形式继承 PersonInfo,虽然条款 39 也说过，只要多加点工作，CPerson也可以结合“复合+继承”技术以求有效重新定义PersonInfo的 virtual函数。此处使用 private 继承。

# 条款41：了解隐式接口和编译期多态

1. classes和templates都支持接口(interfaces)和多态(polymorphism）；

2. 对 classes 而言接口是显式的(explicit),以函数签名为中心。多态则是通过 virtual函数发生于运行期；

   ```c++
   class Widget
   {
       public:
       	Widget();
       	virtual ~Widget();
       	virtual std::size_t size() const;
       	virtual void normalize();
       	void swap(Widget& other);
       	...
   };
   ```

   由于w的类型被声明为 widget，所以w必须支持 widget接口。我们可以在源码中找出这个接口(例如在 widget的.h文件中)，看看它是什么样子，所以我称此为一个显式接口(explicitinterface)，也就是它在源码中明确可见。

   由于 widget 的某些成员函数是 virtual，w对那些函数的调用将表现出运行期多态(runrime polymorphism)，也就是说将于运行期根据w的动态类型(见条款37)决定究竟调用哪一个函数。

3. 对 template 参数而言，接口是隐式的(implicit)，奠基于有效表达式。多态则是通过 template 具现化和函数重载解析(function overloading resolution)发生于编译期。

   ```c++
   template<typename T>
   void doProcessing(T &w)
   {
       if(w.size() > 10 && w != someNastyWidget)
       {
           T temp(w);
           temp.normallize();
           temp.swap(w);
       }
   }
   ```

   w必须支持哪一种接口，系由template中执行于w身上的操作来决定。本例看来w的类型T好像必须支持size,normalize和swap成员函数、copy构造函数(用以建立 temp)、不等比较(inequalitycomparison,用来比较someNasty-widget)我们很快会看到这并非完全正确，但对目前而言足够真实。重要的是，这一组表达式(对此template而言必须有效编译)便是必须支持的一组隐式接口;

   凡涉及w的任何函数调用,例如operator>和operator!=,有可能造成template具现化(instantiated)，使这些调用得以成功。在编译时，编译器会通过模板具现化来生成具体类型的代码，这些代码在编译期就已经确定。编译器根据实际传入的类型选择合适的模板实例，这就形成了一种在编译期的多态性，不同于传统的运行时多态（如虚函数）。

# 条款42：了解 typename 的双重意义

1. 声明 template 参数时，前缀关键字 class 和 typename 可互换；

2. 请使用关键字 typename标识嵌套从属类型名称，但不得在base class lists(基类列)或member initialization list(成员初值列)内以它作为 base class 修饰符。

   ```c++
   template<typename T>
   class Derived : public Base<T>::Nested
   {
       public:
       	explicit Derived(int x) : Base<T>::Nested(x)
           {
               typename Base<T>::Nested temp;
               ...
           }
       	...
   };
   ```

# 条款43：学习处理模板化基类内的名称

```c++
class CompanyA
{
    public:
    	...
        void sendCleartext(const std::string& msg);
    	void sendEncrypted(const std::string& msg);
    	...
};

class CompanyB
{
    public:
    	...
        void sendCleartext(const std::string& msg);
    	void sendEncrypted(const std::string& msg);
    	...
};

class MsgInfo { ... } //保存信息

template<typename Company>
class MsgSender
{
    public:
    	...
        void sendClear(const MsgInfo& info)
        {
            std::string msg;
            ...
            根据info产生信息；
            ...
            Company c;
            c.sendCleartext(msg);
        }
    	void sendSecret(const MsgInfo& info)
        { ... }
};
```

```c++
template<typename Company>
class LoggingMsgSender : public MsgSender<company>
{
    public:
    	...
        void sendClearMsg(const MsgInfo& info)
        {
            将传送前的信息写到log；
            sendClear(info);
            将传送后的信息写到log；
        }
    	...
};
```

这段代码无法通过编译问题在于，当编译器遭遇 class template LoggingMsgSender定义式时，并不知道它继承什么样的 class。当然它继承的是 MsgSender< Company >,但其中的 Company是个 template 参数，不到后来(当LoggingMsgSender被具现化)无法确切知道它是什么。而如果不知道 Company是什么，就无法知道 class MsgSender< company >看起来像什么--更明确地说是没办法知道它是否有个sendclear 函数。

```c++
class CompanyZ
{
    public:
    	...
    	void sendEncrypted(const std::string& msg);
    	...
};

//全特化的MsgSender：和一般的template相同，差别在于删掉了sendClear
template<>
class MsgSender<CompanyZ>
{
    public:
    	...
    	void sendSecret(const MsgInfo& info)
        { ... }
};
```

再次考虑派生类模板LoggingMsgSender的代码，当 base class 被指定为 MsgSender< CompanyZ >时这段代码不合法，因为那个 class 并未提供 sendclear函数!那就是为什么C++ 拒绝这个调用的原因:它知道 base class templates有可能被特化，而那个特化版本可能不提供和一般性 template 相同的接口。因此它往往拒绝在 templatized base classes(模板化基类本例的 MsgSender< Company >)内寻找继承而来的名称(本例的Sendclear)。就某种意义而言，当我们从 Object Oriented C++ 跨进Template C++(见条款 1)，继承就不像以前那般畅行无阻了。

有三种方法解决上述编译问题：

1. 在 base class 函数调用动作之前加上"this ->"：

   this -> sendClear(info);

2. using声明式声明后，即可直接调用：

   using MsgSender< Company >::sendClear;

3. 明白指出被调用的函数位于base class内:

   MsgSender< Company >::sendClear(info);

   但这往往是最不让人满意的一个解法，因为如果被调用的是 virtual函数，上述的明确资格修饰(explicit qualification)会关闭“virtual 绑定行为”

从名称可视点(visibilitypoint)的角度出发，上述每一个解法做的事情都相同对编译器承诺“base casstemplate的任何特化版本都将支持其一般(泛化)版本所提供的接口”。这样一个承诺是编译器在解析(parse)像 LoggingMsgSender这样的 derived class template 时需要的。但如果这个承诺最终未被实践出来，往后的编译最终还是会还给事实一个公道。举个例子，如果稍后的源码内含这个:

```c++
LoggingMsgSender<CompanyZ> zMsgSender;
MsgInfo msgData;
...
zMsgSender.sendClearMsg(msgData);
```

其中对 sendclearMsg的调用动作将无法通过编译，因为在那个点上，编译器知道 base class 是个 template 特化版本 MsgSender< companyz >，而且它们知道那个class 不提供 sendclear函数，而后者却是 sendclearMsg 尝试调用的函数。

# 条款44：将与参数无关的代码抽离templates

1. Templates生成多个 classes 和多个函数,所以任何template 代码都不该与某个造成膨胀的 template 参数产生相依关系；

   ```c++
   template<typename T, std::size_t n>
   class SquareMatrix
   {
       public:
       	...
       	void invert();
   };
   
   SquareMatrix<double, 5> sm1;
   ...
   sm1.invert();
   SquareMatrix<double, 5> sm2;
   ...
   sm2.invert();
   ```

   这会具现化两份invert。这些函数并非完完全全相同，因为其中一个操作的是5*5矩阵而另一个操作的是10 * 10矩阵，但除了常量5和10，两个函数的其他部分完全相同。这是template引出代码膨胀的一个典型例子。

2. 因非类型模板参数(non-typetemplate parameters)而造成的代码膨胀，往往可消除，做法是以函数参数或class成员变量替换template 参数；

   ```c++
   template<typename T>
   class SquareMatrixBase
   {
       public:
       	SquareMatrixBase(std::size_t n, T* pMem) : size(n), pData(pMem) { }
       	void setDaraPtr(T* ptr) { pData = ptr; }
       	...
       private:
       	std::size_t size;
       	T* pData;
       protected:
       	void invert(std::size_t matrixSize);
   };
   
   template<typename T, std::size_t n>
   class SquareMatrix : private SquareMatrixBase<T>
   {
       public:
       	SquareMatrix() : SquareMatrixBase<T>(n, 0), pData(new T[n*n])
       	{
               this->setDataPtr(pData.get());
       	}
       	void invert() { this->invert(n); }
       private:
       	boost::scoped_array<T> pData;
   }
   ```

3. 因类型参数(typeparameters)而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述(binary representations)的具现类型(instantiation types)共享实现码。

# 条款45：运用成员函数模板接受所有兼容类型

1. 请使用member function templates(成员函数模板)生成“可接受所有兼容类型的函数；
2. 如果你声明 member templates 用于“泛化 copy构造”或“泛化 assignment操作”你还是需要声明正常的copy构造函数和copyassignment操作符。

TR1规范中关于tr1:shared_ptr的一份摘录：

```c++
template<class T>
class shared_ptr
{
	public:
		template<class Y>
			explicit shared_ptr(Y* p);
		share_ptr(share_ptr const& r);
		template<class Y>
			shared_ptr(shared_ptr<Y> const& r);
		template<class Y>
			shared_ptr(weak_ptr<Y> const& r);
		template<class Y>
			shared_ptr(auto_ptr<Y>& r);
			
		shared_ptr& operator=(shared_ptr const& r);
		template<class Y>
			share_ptr& operator=(shared_ptr<Y> const& r);
		template<class Y>
			share_ptr& operator=(auto_ptr<Y> const& r);
		...
}
```

```c++
class Top { ... };
class Middle : public Top { ... };
class Bottom : public Middle { ... };

template<typename T>
class SmartPtr
{
    public:
    	template<typename U>
    	SmartPtr(const SmartPtr<U>& other) : heldPtr(other.get()) { ... }
    	T* get() const { return heldPtr; }
    	...
    private:
    	T* HeldPtr;
};

SmartPtr<Top> pt1 = SmartPtr<Middle>(new Middle);
SmartPtr<Top> pt2 = SmartPtr<Bottom>(new Bottom);
SmartPtr<const Top> pct2 = pt1;
```

# 条款46：需要类型转换时请为模板定义非成员函数

当我们编写一个 class template，而它所提供之“与此template 相关的”函数支持“所有参数之隐式类型转换”时，请将那些函数定义为“classtemplate内部的 friend 函数”。

参考条款24，将其模板化后如下：

```c++
template<typename T>
class Rational
{
    public:
    	Rational(const T& numerator = 0, const T& denominator = 1);
    	const T numerator() const;
    	const T denominator() const;
    	...
};

template<typename T>
const Rational<T> operator* (const Rational<T>& lhs, const Rational<T>& rhs) { ... }

Rational<int> onehalf(1, 2);
Rational<int> result = onehalf * 2; //不通过编译
```

以oneHalf进行推导，过程并不困难。operator*的第一参数被声明为Rational< T >，而传递给operator * 的第一实参(oneHalf)的类型是Rational< int >，所以!一定是 int。其他参数的推导则没有这么顺利。operator * 的第二参数被声明为 Rational< T >，但传递给 operator *的第二实参(2)类型是int。编译器如何根据这个推算出T?你或许会期盼编译器使用 Rational< int >的non-explicit构造函数将2转换为Rational< int >，进而将推导为 int，但它们不那么做，因为在 template 实参推导过程中从不将隐式类型转换函数纳入考虑。绝不!这样的转换在函数调用过程中的确被使用，但在能够调用一个函数之前，首先必须知道那个函数存在。而为了知道它，必须先为相关的functiontemplate 推导出参数类型(然后才可将适当的函数具现化出来)。然而template 实参推导过程中并不考虑采纳“通过构造函数而发生的”隐式类型转换。

只要利用一个事实，我们就可以缓和编译器在template 实参推导方面受到的挑战:templateclass内的friend声明式可以指涉某个特定函数。那意味classRational< T >可以声明 operator * 是它的一个 fiend 函数。Class templates 并不倚赖 template 实参推导(后者只施行于functiontemplates身上)，所以编译器总是能够在 class Rational< T >具现化时得知T。因此，令 Rational< T > class 声明适当的operator *为其 fiend 函数，可简化整个问题:

```c++
template<typename T> class Rational;

template<typename T>
const Rational<T> doMultiply(const Rational<T>& lhs, const Rational<T>& rhs)
{
    return Rational<T>(lhs.numerator * rhs.numerator, lhs.denominator * rhs.denominator)
}

template<typename T>
class Rational
{
    public:
    	friend const Rational<T> operator* (const Rational<T>& lhs, const Rational<T>& rhs)
        {
            return doMultiply(lhs, rhs);
        }
}
```

现在对 operator * 的混合式调用可以通过编译了，因为当对象oneHalf 被声明为一个 Rational< int >，class Rational< int >于是被具现化出来，而作为过程的一部分，friend 函数operator *(接受Rational< int >参数)也就被自动声明出来。后者身为一个函数而非函数模板(functiontemplate)，因此编译器可在调用它时使用隐式转换函数(例如 Rationa1的 non-explicit构造函数)，而这便是混合式调用之所以成功的原因。

# 条款47：请使用 traits classes 表现类型信息

1. Traits classes 使得“类型相关信息”在编译期可用。它们以templates 和“templates特化”完成实现；
2. 整合重载技术(overloading)后，traitscasses有可能在编译期对类型执行if...else 测试，在编译期完成的事延到运行期才做的话，不仅浪费时间，也造成可执行文件膨胀。

traits classes：

iterator_traits的运作方式是，针对每一个类型IterT，在struct iterator_traits< IterT >内一定声明某个 typedef名为 iterator category。这个 typedef用来确认 IterT 的迭代器分类。iterator traits 以两个部分实现上述所言。首先它要求每一个“用户自定义的迭代器类型”必须嵌套一个typedef,名为iterator category，用来确认适当的卷标结构(tagstruct)。例如 degue的迭代器可随机访问，所以一个针对 degue迭代器而设计的class 看起来会是这样子:

```c++
template< ... >
class deque
{
    public:
    	class iterator
    	{
            public:
            	typedef random_access_iterator_tag iterator_category;
            	...
    	};
    	...
};
```

 iterator_traits响应 iterator class 的嵌套式 typedef:

```c++
template<typename IterT>
struct iterator_traits
{
    typedef typename IterT::iterator_categpry iterator_category;
    ...
}
```

为了支持指针迭代器，iterator traits特别针对指针类型提供一个偏特化版本(partial template specialization)。由于指针的行径与 random access迭代器类似，所以 iterator traits为指针指定的迭代器类型是：

```c++
template<typename IterT>
struct iterator_traits<IterT*>
{
    typedef typename IterT::iterator_categpry iterator_category;
    ...
}
```

typeid-based 解法如下：

```c++
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    if(typeid(typename std::iterator_traits<IterT>::iterator_category) == typeid(std::random_access_iterator_tag))
    {
        iter += d; //会出现编译问题
    }
    else
    {
        id(d > 0) { while(d--) ++iter; }
        else { while(d++) --iter; }
    }
}
```

这个 typeid-based 解法的效率比 traits 解法低,因为在此方案中,1)类型测试发生于运行期而非编译期，(2)“运行期类型测试”代码会出现在(或说被连接于)可执行文件中。

编译问题出在所强调的那一行代码使用了+=操作符，那便是尝试在一个1ist< int >::iterator身上使用+=,但list< int >::iterator是 bidirectional选代器(见条款47)，并不支持+=。只有random access迭代器才支持+=。此刻我们知道绝不会执行起 +-那一行，因为测试typeid 的那一行总是会因为1ist< int >::iterators 而失败，但编译器必须确保所有源码都有效，纵使是不会执行起来的代码!而当iter不是 random access迭代器时"iter += d”无效。与此对比的是 traits-based TMP 解法，其针对不同类型而进行的代码，被拆分为不同的函数，每个函数所使用的操作(操作符)都可施行于该函数所对付的类型。

如何使用一个traits class：

1. 彼此间的建立一组重载函数(身份像劳工)或函数模板(例如doAdvance)差异只在于各自的 traits 参数。令每个函数实现码与其接受之 traits 信息相应和；
2. 建立一个控制函数(身份像工头)或函数模板(例如advance)，它调用上述那些“劳工函数”并传递traits class 所提供的信息。

```c++
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::random_access_iterator_tag)
{
    
    iter += d;
}

template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag)
{
    if(d >= 0) { while(d--) ++iter; }
    else{ while(d++) --iter; }
}

template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::input_iterator_tag)
{
    if(d < 0)
    {
        throw std::out_of_tange("Negative distance);
    }
    while(d--) ++iter;
}

template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    doAdvance(iter, d, typename std::iterator_traits<IterT>::iterator_category());
}
```

# 条款48：认识 template 元编程

1. Template metaprogramming(TMP，模板元编程)可将工作由运行期移往编译期因而得以实现早期错误侦测和更高的执行效率；
2. TMP 可被用来生成“基于政策选择组合”(basedoncombinationsofpolicychoices)的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。

# 条款49：了解 new-handler 的行为

1. set new handler允许客户指定一个函数，在内存分配无法获得满足时被调用；

2. 一个设计良好的new-handler函数必须做以下事情：

   1. 让更多内存可被使用。这便造成operatornew内的下一次内存分配动作可能成功。实现此策略的一个做法是，程序一开始执行就分配一大块内存，而后new-handler第一次被调用，将它们释还给程序使用；
   2. 安装另一个 new-handler。如果目前这个 new-handler 无法取得更多可用内存或许它知道另外哪个new-handler有此能力。果真如此，目前这个new-handler就可以安装另外那个 new-handler 以替换自己(只要调用set new handler)下次当operator new调用new-handler，调用的将是最新安装的那个。(这个旋律的变奏之一是让 new-handler 修改自己的行为，于是当它下次被调用，就会做某些不同的事。为达此目的，做法之一是令new-handler修改“会影响new-handler 行为”的 static 数据、namespace 数据或 global 数据。)；
   3. 卸除 new-handler，也就是将null指针传给set new handler。一旦没有安装任何 new-handler，operator new会在内存分配不成功时抛出异常；
   4. 抛出 bad_alloc(或派生自 bad alloc)的异常。这样的异常不会被 operatornew捕捉，因此会被传播到内存索求处；
   5. 不返回，通常调用 abort或exit。

3. 实现 class 专属之 new-handlers：

   ```c++
   class NewHandlerHolder
   {
       public:
       	explicit NewHandlerHolder(std::new_handler nh) : handler(nh) {}
       	~NewHandlerHolder()
           {
               std::set_new_handler(handler);
   		}
       private:
       	std::new_handler handler;
       	NewHandlerHolder(const NewHandlerHolder&);
       	NewHandlerHolder& operator=(const NewHandlerHolder&);
   };
   
   template<typename T>
   class NewHandlerSupport
   {
   	public:
   		static std::new_handler set_new_handler(std::new_handler p) throw();
   		static void* operator new(std::size_t size) throw(std::bad_alloc);
   		...
   	private:
   		static::std::new_handler currentHandler;
   };
   
   template<typename T>
   std::new_handler NewHandlerSupport<T>::set_new_handler(std::new_handler p) throw()
   {
   	std::new_handler oldHandler = currentHandler;
   	currentHandler = p;
   	return oldHandler;
   }
   
   template<typename T>
   void* NewHandlerSupport<T>::operator new(std::size_t size) throw(std::bad_alloc)
   {
   	NewHandlerHolder h(std::set_new_handler(currentHandler));
   	return ::operator new(size);
   }
   
   template<typename T>
   std::new_handler NewHandlerSupport<T>::currentHandler = 0;
   
   class Widget : public NewHandlerSupport<Widget> { ... };
   
   //widget的客户应该类似这样使用其new-handling
   void outOfMem(); //Widget对象分配失败时调用的函数
   Widget::set_new_handler(outOfMem); //设定Widget的new-handling的函数
   Widget* pw1 = new Widget; //分配失败，则调用outfMem
   
   //调用标准 set_new_handler，告知 widget的错误处理函数。这会将 widget 的new-handler 安装为 global new-handler。，
   //调用 global operator new，执行实际之内存分配。如果分配失败，operator new会调用widge的new-handler，因为那个函数才刚被安装为 global new-handler。如果 global operator new 最终无法分配足够内存，会抛出一个bad_alloc 异常。在此情况下 widget 的 operator new 必须恢复原本的 global new-handler，然后再传播该异常。为确保原本的new-handler 总是能够被重新安装回去，widget将globalnew-handler 视为资源并遵守条款13的忠告，运用资源管理对象(resource-managing objects)防止资源泄漏。
   //如果 global operator new能够分配足够一个 widget 对象所用的内存， widget的 operator new 会返回一个指针，指向分配所得。widget 析构函数会管理global new-handler，它会自动将 widget's operator new被调用前的那个 global new-handler 恢复回来。
   ```

   这个设计的 base class 部分让 derived classes 继承它们所需的 set new handler和 operator new，而 template 部分则确保每一个 derived class 获得一个实体互异的currentHandler 成员变量；

4. Nothrow new是一个颇为局限的工具，因为它只适用于内存分配；后继的构造函数调用还是可能抛出异常：

   直至1993年，C++都还要求operator new必须在无法分配足够内存时返回2null。新一代的 operator new则应该抛出 bad alloc异常，但很多 C++ 程序是编译器开始支持新修规范前写出来的。C++标准委员会不想抛弃那些“侦测null”的族群，于是提供另一形式的operator new，负责供应传统的“分配失败便返回null”行为。这个形式被称为"nothrow”形式--某种程度上是因为他们在 new的使用场合用了 nothrow对象(定义于头文件< new >)；

   ```c++
   class Widget { ... }
   Widget* pw1 = new Widget; //分配失败的话，抛出bad_alloc.
   if(pw1 == 0) ... //这个测试一定失败
   Widget* pw2 = new (std::nothrow) Widget; //分配失败的话，返回0.
   if(pw2 == 0) ... //可能成功
   ```

   如果分配成功，接下来 widget构造函数会被调用，而在那一点上所有的筹码便都耗尽，因为widget 构造函数可以做它想做的任何事。它有可能又 new 一些内存，而没人可以强迫它再次使用 nothrow new。因此虽然"new(std::nothrow)widget”调用的operator new并不抛掷异常，但 widget构造函数却可能会。如果它真那么做，该异常会一如往常地传播。需要结论吗?结论就是:使用nothrownew只能保证operator new不抛掷异常，不保证像"new(std::nothrow)widget"这样的表达式绝不导致异常。因此你其实没有运用 nothrow new的需要。

# 条款50：了解 new和 delete 的合理替换时机

1. 为了检测运用错误，如果将“new所得内存”delete 掉却不幸失败，会导致内存泄漏(memory leaks)。如果在“new所得内存”身上多次 delete则会导致不确定行为。如果operator new持有一串动态分配所得地址，而operator delete将地址从中移走，倒是很容易检测出上述错误用法。此外各式各样的编程错误可能导致数据“overruns"(写入点在分配区块尾端之后)或"underruns(写入点在分配区块起点之前)。如果我们自行定义一个operator news，便可超额分配内存，以额外空间(位于客户所得区块之前或后)放置特定的 bytepatterms(即签名，signatures)。operator deletes便得以检查上述签名是否原封不动，若否就表示在分配区的某个生命时间点发生了overrun或underrun，这时候 operator delete可以志记(1og)那个事实以及那个惹是生非的指针；
2. 为了收集动态分配内存之使用统计信息，在一头栽进定制型 news 和定制型 deletes 之前，理当先收集你的软件如何使用其动态内存。分配区块的大小分布如何?寿命分布如何?它们倾向于以FIFO(先进先出)次序或LIFO(后进先出)次序或随机次序来分配和归还?它们的运用型态是否随时间改变，也就是说你的软件在不同的执行阶段有不同的分配/归还形态吗?任何时刻所使用的最大动态分配量(高水位)是多少?自行定义operator new和operator delete 使我们得以轻松收集到这些信息；
3. 为了增加分配和归还的速度。泛用型分配器往往(虽然并不总是)比定制型分配器慢，特别是当定制型分配器专门针对某特定类型之对象而设计时。Class专属分配器是“区块尺寸固定”之分配器实例，例如 Boost提供的Pool程序库便是。如果你的程序是个单线程程序，但你的编译器所带的内存管理器具备线程安全，你或许可以写个不具线程安全的分配器而大幅改善速度。当然，在获得operator new和 operator delete有加快程序速度的价值”这个结论之前首先请分析你的程序，确认程序瓶颈的确发生在那些内存函数身上；
4. 为了降低缺省内存管理器带来的空间额外开销。泛用型内存管理器往往(虽然并非总是)不只比定制型慢，它们往往还使用更多内存，那是因为它们常常在每一个分配区块身上招引某些额外开销。针对小型对象而开发的分配器(例如Boost 的Pool程序库)本质上消除了这样的额外开销；
5. 为了弥补缺省分配器中的非最佳齐位(suboptimalalignment)一如先前所说，在x86体系结构上doubles的访问最是快速--如果它们都是8-byte 齐位。但是编译器自带的 operator news并不保证对动态分配而得的 doubles 采取8-byte 齐位。这种情况下，将缺省的operator new替换为一个 8-byte 齐位保证版，可导致程序效率大幅提升；
6. 为了将相关对象成簇集中，如果你知道特定之某个数据结构往往被一起使用，而你又希望在处理这些数据时将“内存页错误”(page faults)的频率降至最低，那么为此数据结构创建另一个heap 就有意义，这么一来它们就可以被成簇集中在尽可能少的内存页(pages)上。new和delete的“placement版本”(见条款 52)有可能完成这样的集簇行为；
7. 为了获得非传统的行为。有时候你会希望operators new和 delete做编译器附带版没做的某些事情。例如你可能会希望分配和归还共享内存(sharedmemory)内的区块，但唯一能够管理该内存的只有C API函数，那么写下一个定制版 new和 delete(很可能是 placement版本，见条款52)，你便得以为C API穿上一件 C++ 外套。你也可以写一个自定的operator delete，在其中将所有归还内存内容覆盖为0，藉此增加应用程序的数据安全；

# 条款51：编写 new和 delete 时需固守常规

1. operator new应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用new-handler。operator new也应该有能力处理0bytes申请：

   ```c++
   void* operator new(std::size_t size) throw(std::bad_alloc)
   {
   	using namespace std;
   	if(size == 0)
   	{
   		size = 1;
   	}
   	while(true)
   	{
   		尝试分配size bytes;
   		if(分配成功)
   			return (一个指针，指向分配而来的内存);
   		
   		//分配失败：找出目前的new-handling函数
   		new_handler globalHandler = set_new_handler(0);
   		set_new_handler(globalHandler);
   		
   		if(globalHandler) (*globalHandler)();
   		else throw std::bad_alloc();
   	}
   }
   ```

   Class专属版本则还应该处理“比正确大小更大的(错误)申请”：

   许多人没有意识到 operator new成员函数会被 derived classes 继承。这会导致某些有趣的复杂度。注意上述 operatornew伪码中，函数尝试分配size bytes(除非size是0)。那非常合理,因为 size是函数接受的实参。然而就像条款 50所言写出定制型内存管理器的一个最常见理由是为针对某特定class的对象分配行为提供最优化，却不是为了该class的任何derived classes。也就是说，针对class X而设计的 operator new，其行为很典型地只为大小刚好为 sizeof(X)的对象而设计。然而一旦被继承下去，有可能base class的operator new被调用用以分配 derivedclass 对象：

   ```c++
   class Base
   {
   	public:
   		static void* operator new(std::size_T size) throw(std::bad_alloc);
   		...
   };
   
   class Derived : public Base //假设Derived未声明operator new
   { ... };
   Derived* p = new Derived; //这里调用的是Base::operator new
   
   void* Base::operator new(std::size_t size) throw(std::bad_alloc)
   {
   	if(size != sizeof(Base))
   		return ::operator new(size);
   	...
   }
   ```

   你可能考虑忘了检验 size等于0这种病态但是可能出现的情况。需要注意的是，它和上述的“size与 sizeof(Base)的检测”融合一起了。是的，C++ 在某种秘境中运行，而其中一个秘境就是它裁定所有非附属(独立式)对象必须有非零大小(见条款 39)。因此 sizeof(Base)无论如何不能为零，所以如果size是0，这份申请会被转交到::operatornew手上，后者有责任以某种合理方式对待这份申请。

2. operator delete应该在收到 null 指针时不做任何事，Class 专属版本则还应该处理“比正确大小更大的(错误)申请”：

   ```c++
   class Base
   {
   	public:
   		static void* operator new(std::size_T size) throw(std::bad_alloc);
   		static void operator delete(void* rawMemory, std::size_t size) throw()
   		...
   };
   
   void Base::operator delete(void* rawMemory, std::size_t size) throw()
   {
   	if(rawMemory == 0) return;
   	if(size != sizeof(Base))
   	{
   		::operator delete(rawMemory);
   		return;
   	}
   	...
   	return;
   }
   ```

# 采款52：写了placement new也要写placement delete

1. 当你写一个 placement operator new，请确定也写出了对应的 placement operator delete。如果没有这样做，你的程序可能会发生隐微而时断时续的内存泄漏。即  new 正常调用，内存分配成功，但类的构造函数抛出异常，此时运行期系统有责任取消 operator new的分配并恢复旧观，然而运行期系统无法知道真正被调用的那个 operator new 如何运作，因此它无法取消分配并恢复旧观，所以上述做法行不通。取而代之的是，运行期系统寻找“参数个数和类型都与operator new相同”的某个 operator delete，如果找到,那就是它的调用对象，如果找不到，发生内存泄漏；

2. 如果你在 class 内声明任何operator news，它会遮掩标准形式。除非你的意思就是要阻止class的客户使用这些形式，否则请确保它们在你所生成的任何定制型 operator new之外还可用。对于每一个可用的 operator new也请确定提供对应的 operator delete。如果你希望这些函数有着平常的行为，只要令你的class 专属版本调用 global版本即可。完成以上所言的一个简单做法是，建立一个base class，内含所有正常形式的new和delete:

   ```c++
   class StandardNewDeleteForms
   {
       public:
       	//normal new/delete
       	static void* operator new(std::size_t size) throw(std::bad_alloc) { return ::operator new(size); }
       	static void operator delete(void* pMemory) throw() { ::operator delete(pMemory); }
       	//placement new/delete
       	static void* operator new(std::size_t size, void* ptr) throw() { return ::operator new(size, ptr); }
       	static void operator delete(void* pMemory, void* ptr) throw() { return ::operator delete(pMemory, ptr); }
       	//nothrow new/delete
       	static void* operator new(std::size_t size, const std::nothrow_t& nt) throw() { return ::operator new(size, nt); }
       	static void operator delete(void* pMemory, const std::nothrow_t&) throw() { return ::operator delete(pMemory); }
   }
   
   //凡是想以自定形式扩充标准形式的客户，可利用继承机制及using声明式
   
   class Widget : public StandardNewDeleteForms
   {
   	public:
   		using StandardNewDeleteForms::operator new;
   		using StandardNewDeleteForms::operator delete;
   		static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);
   		static void operator new(void* pMemory, std::ostream& logStream) throw();
   		...
   }
   ```

# 条款53：不要轻忽编译器的警告

1. 严肃对待编译器发出的警告信息。努力在你的编译器的最高(最严苛)警告级别下争取“无任何警告”的荣誉；
2. 不要过度倚赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦移植到另一个编译器上，你原本倚赖的警告信息有可能消失。

# 条款54：让自己熟悉包括 TR1在内的标准程序库













