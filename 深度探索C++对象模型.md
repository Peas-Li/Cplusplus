# 关于对象

## C++对象模式

在C++中，有两种class data members: static和nonstatic，以及三种class member functions: static、nonsiatic和 virtual。已知下面这个 class Point 声明：

```c++
class Point{
	public:
		point(float xval);
		virtoal ~Point();
		float x()const;
		static int PointCount();
	protected:
		virtual ostream& print(ostream &os) const;
		float x;
		static int point count;
};
```

1. **简单对象模型：**

   第一个模型十分简单。它可能是为了尽量减低C++编译器的设计复杂度而开发出来的，赔上的则是空间和执行期的效率。在这个简单模型中，一个object是一系列的 slots，每一个 slot指向一个 members。Members 按其声明次序，各被指定一个 slot。每一个data member或function member 都有自己的一个 slot。

   ![简单对象模型](F:\Cplusplus\typoraImg\简单对象模型.jpg)

   **优缺点：**

   - 优点：设计简单，指针的大小是固定的，这种模型可以避免“不同类型members需要不同存储空间”的问题
   - 缺点：多了一层间接性，空间和执行期效率稍低。 应用： 未实际应用。

2. **表格驱动对象模型:**
   为了对所有classes的所有objects 都有一致的表达方式,另一种对象模型是把所有与 members相关的信息抽出来，放在一个datamember table 和一个member functiontable之中，class object本身则内含指向这两个表格的指针。Member function table 是一系列的 slots,每一个 slot 指出一个 member function;Datamembertable 则直接含有 data本身。

   ![表格驱动对象模型](F:\Cplusplus\typoraImg\表格驱动对象模型.jpg)

   **优缺点：**

   - 优点：弹性较大，可以修改成员变量
   - 缺点：多了一层间接性，付出了空间和执行效率的代价

3. C++对象模型：

   Stroustrup当初设计(当前亦仍占有优势)的C++对象模型是从简单对象模型派生而来的，并对内存空间和存取时间做了优化。在此模型中，非静态数据成员被配置于每一个类对象之内，静态数据成员则被存放在所有的类对象之外。静态和非静态函数成员也被放在所有的类对象之外。虚函数则以两个步骤支持之:

   1. 每一个类产生出一堆指向虚拟函数的指针，放在表格之中。这个表格被称为虚拟函数!表(vtbl)中的一个实例。
   2. 每一个类对象被添加了一个指针，指向相关的虚拟表。通常这个指针被称为虚拟表。Vptr的设定(set)和重置(reset)都由每一个类的构造函数，析构函数和复制赋值运算符自动完成(我将在第5章讨论)。每一个类所关联的type_infoobject(用以支持非时间类型标识，RTTI)也经由虚拟表被指出来，通常是放在表格的第一个slot处。

   ![c++对象模型](F:\Cplusplus\typoraImg\c++对象模型.jpg)

   **特性：**

- 每个非内联(non-inline)成员函数只会诞生一个函数实例. 而内联函数会在每个使用者身上产生一个函数实例.

- C++在布局以及存取时间上的额外负担主要由虚(virtual)引起的:

  1. 虚函数机制(virtual function)用于支持一个有效率的运行期绑定(runtime binding).
  2. 虚基类(virtual base class) 用以实现多次出现在继承体系中的基类, 有一个单一而被共享的实例.
  3. 额外负担, 派生类转换。

- 在C++ 中，有两种静态的数据成员（static）、非静态的数据成员（nonstatic）；

- 有三种成员函数：静态的函数（static）、非静态的函数（nonstatic）、虚函数（virtual）

  **优缺点：**

- 优点：在于它的空间和存取时间的效率;
- 缺点：如果应用程序代码本身未曾改变，但所用到的classobjects的nonstatic datamembers 有所修改(可能是增加、移除或更改)，那么那些应用程序代码同样得重新编译，关于这点，前述的双表格模型就提供了较大的弹性，因为它多提供了一层间接性，不过它也因此付出空间和执行效率两方面的代价。

## class 和 struct

**关键词所带来的差异(A Keyword Distinction )**

struct的默认访问修饰符是public；而class的默认访问修饰符是private。 struct的默认继承方式是public，而class的默认继承方式是private。除此之外使用时没有区别。

- 某种意义上, 在C++中struct和class这两个关键字是可以互换的。

**关键字的困扰：struct 和 class**

- class不仅是一个关键字, 它还会引入它所支持的封装和继承的哲学。
- 为了维护c++与C语言之间的兼容性，重载函数的解决方式变得很复杂。

**策略性正确的struct（The Politically Correct Struct）**

```
struct mumble
{
	/* stuff */
	char pc[1];
}

//从档案或标准输入装置中取得一个字符串，然后为struct本身和该字符串配置足够的内存

struct mumble* pmumbl = (struct mumble*)malloc(sizeof(struct mumble) + strlen(string) + 1);
strcpy(*pmumbl.pc, string);
```

如果我们改用class来声明，而该class是:

- 指定多个 access sections，内含数据:
- 从另一个 class 派生而来;
- 定义有一个或多个 virtual functions;

这里讲述的是在C中，struct没有访问修饰符，因此struct中的成员变量是按声明顺序出现的内存中的，而在C++中，中凡处于同一个accesssection的数据，必定保证以其声明次序出现在内存布局当中。然而被放置在多个access sections中的各笔数据，排列次序就不一定了。下面的声明中，前述的C伎俩或许可以有效运行，或许不能，需视protected datamembers 被放在 private datamembers 的前面或后面而定(译注:放在前面才可以):

```c++
class stumble
{
	public:
		// operations...
	protected:
		// protected stuff
	private:
		/* private stuff */
		char pc[1];
};
```

同样的道理,baseclasses和derived classes的 data members 的布局也没有谁先谁后的强制规定，因而也就不保证前述的伎俩一定有效。虚函数也会其失效。

如果一个程序员迫切需要一个相当复杂的C++class 的某部分数据，使他拥有C声明的那种样子：

- 继承（不在推荐）：那一部分最好抽取出来成为一个独立的struct声明。将C与C++组合在一起的作法就是，从C struct中派生C++的部分:

  ```c++
  struct C_point{...};
  class Point :public C_point{...};
  ```

  C与C++都将支持：

  ```c++
  extern void draw_line(Point,Point);
  extern "C" void draw_rect(C_point, C_point);
  draw_line(Point(0,0),Point(100，100));
  draw_rect(Point(0,0)，Point(100，100));
  ```

- 组合而非继承, 才是把C和C++结合在一起的唯一可行方法：

  上述习惯用法现已不再被推荐，因为某些编译器(如Microsoft C++)在支持virtual function的机制中对于class的继承布局做了一些改变(请看3.4 节的讨论)。组合(composition)，而非继承，才是把C和C++结合在一起的唯一可行方法(conversion运算符提供了一个十分便利的萃取方法):

  ```c++
  struct C_point{...};
  class Point{
  	public:
  		operator C_point() { returnc_point; }
          // ...
      private :
  		C_point _c_point;
  }
  ```

## 对象的差异

**三种程序设计范式**

- 程序模型：
  - 像C一样，普通的程序
- 抽象数据类型模型（ADT）：
  - 如string类，所谓的抽象由public接口提供。
- 面向对象模型（OO）：
  - C++通过class的pointers和references来支持多态，这种程序设计风格被称为“面向对象”。   - 在此模型中有一些彼此相关的类型，通过抽象的base class封装起来。

**OO（Object-Oriented，面向对象）和ADT（Abstract Data Type，抽象数据类型）区别：**

ADT（抽象数据类型）

1. 定义：ADT是对数据结构的一种抽象描述，它关注的是数据类型的逻辑定义，包括数据的存储结构和在此结构上可以执行的操作，而不关心这些操作的具体实现细节。ADT提供了一种封装机制，隐藏了数据的内部表示，只暴露必要的接口给用户。
2. 特点：
   - 抽象性：ADT强调的是数据类型的逻辑特性，而不是物理实现。
   - 封装性：隐藏数据结构和实现细节，只暴露接口。
   - 通用性：ADT的定义独立于任何编程语言，可以用任何支持相应数据结构的语言实现。
3. 示例：string等都是典型的ADT，它们定义了数据的添加、删除、查看等操作，而不关心这些操作是如何具体实现的。

OO（面向对象）

1. 定义：面向对象编程是一种编程范式，它使用“对象”来设计软件，这些对象包含了数据（属性）和操作这些数据的方法（行为）。OO不仅仅是关于数据类型，更是一种软件设计思想，强调通过对象的组合、继承和多态来构建复杂系统。
2. 特点：
   - 封装：与ADT相似，但更全面，不仅限于数据，还包括方法的封装。
   - 继承：允许创建分层的类结构，子类可以继承父类的属性和方法，并可进行扩展或覆盖。
   - 多态：同一种消息可以被不同的对象以不同的方式响应，增加了代码的灵活性和可复用性。
3. 示例：在面向对象的语言如Java或C++中，定义一个“动物”类，可以有“名字”和“年龄”属性，以及“叫”这样的方法。然后，可以创建“狗”和“猫”这样的子类，它们继承自“动物”类，并可以重写或添加新的方法。

OB提供一个 public 接口和一个 private 实作品，包括数据和算法，但是不支持类型的扩充。一个OB设计可能比一个对等的OO设计速度更快而且空间更紧凑。速度快是因为所有的函数引发操作都在编译时期解析完成，对象建构起来时不需要设置 virtual机制;空间紧凑则是因为每一个class object 不需要负担传统上为了支持 virtual 机制而需要的额外负荷，不过，OB 设计比较没有弹性

**对象的大小**

1. 非静态的数据成员
2. 字节对齐额外需要的空间。
3. 为了支持多态(virtual机制)，内部产生的额外空间。

**指针的类型**

```c++
class ZooAnimal
{
	public:
		ZooAnimal();
		virtual ~ZooAnimal();
		virtual void rotate();
	protected:
		int loc;
		String name;
};
```

```c++
ZooAnimal* px;
int* pi;
Array<String>* pta;
```

以内存需求的观点来说，没有什么不同!它们三个都需要有足够的内存来放置一个机器地址(通常是个 word,译注).“指向不同类型之各指针”间的差异既不在其指针表示法不同，也不在其内容(代表一个地址)不同，而是在其所寻址出来的 object 类型不同。也就是说，“指针类型”会教导编译器如何解释某个特定地址中的内存内容及其大小:

1. 一个指向地址1000的整数指针，在32位机器上，将盖地址空间1000~1003(译注:因为32位机器上的整数是4-bytes)
2. 如果 String 是传统的8-bytes(包括一个 4-bytes 的字符指针和一个用来表示字符串长度的整数)，那么一个ZooAnimal指针将横跨地址空间1000~1015(译注:4+8+4)

![独立非派生的对象的内存布局](F:\Cplusplus\typoraImg\独立非派生的对象的内存布局.png)

加上多态之后：

```c++
class Bear : public ZooAnimal
{
	public:
		Bear();
		~Bear();
		// ...
		void rotate();
		virtual void dance();
		// ...
	protected:
		enum Dances { ... };
		Dances dances_known;
		int cell_block;
};
Bear b( "Yogi” );
Bear *pb = &b;
Bear &rb = *pb;
```

不管是pointer或reference 都只需要一个word的空间(在32位机器上是 4-bytes)。Bear object 需要 24 bytes，也就是ZooAnimal的16bytes加上Bear所带来的8bytes：

![派生类的对象和指针布局](F:\Cplusplus\typoraImg\派生类的对象和指针布局.png)

假设：

```c++
Bear b;
ZooAnimal *pz = &b;
Bear *pb = &b;
```

它们每个都指向Bearobject的第一个 byte。其间的差别是，pb 所涵盖的地址包含整个 Bearobject，而pz所涵盖的地址只包含 Bearobject中的 Zoodnimalsubobject。除了ZooAnimalsubobject中出现的members，你不能够使用pz来直接处理Bear的任何members。唯一例外是通过 virtual 机制：

```c++
//不合法:cellblock不是ooAnimal的一个member,
//虽然我们知道 pz当前指向一个 Bear object.
pz->cell block;
// ok:经过一个明白的 downcast 操作就没有问题!
((Bear*)pz)->cell block;
//下面这样更好，但它是一个run-time operation(译注:成本较高)
if(Bear*pb2=dynamic cast<Bear*>(pz ))
    pb2->cell block;
//ok:因为cellblock是Bear的一个 member.
pb->cell block;
```

再假设：

```c++
Bear b;
2ooAnimal za = b;//译注:这会引起切割(siiced)
//调用 2ooAnimal::rotate()
za.rotate();
```

为什么 rotate0所调用的是 ZooAnimal实体而不是Bear 实体?

如果初始化函数(译注:应用于上述assignment操作发生时)将一个object内容完整拷贝到另一个 object中去，为什么za的vptr 不指向 Bear 的 virtual table?

答案是，编译器在(1)初始化及(2)指定(assignment)操作(将一个 class object 指定给另一个 class object)之间做出了仲裁。编译器必须确保如果某个obiect含有一个或一个以上的vptrs，那些vptrs 的内容不会被base class object 初始化或改变。

为什么 rotate()所调用的是 ZooAnimal实体而不是Bear 实体?

答案是:za并不是(而且也绝不会是)一个Bear，它是(并且只能是)一个ZooAnimal.多态所造成的“一个以上的类型”的潜在力量并不能够实际发挥在“直接存取objects”这件事情上。有一个似是而非的观念:OO 程序设计并不支持对object的直接处理，举个例子，下面这一组定义:

```c++
ZooAnimal za;
ZooAnimal *pza;
Bear b;
Panda *pp = new panda; //Panda继承Bear
pza = &b;
```

其内存布局可能如下：

![内存布局](F:\Cplusplus\typoraImg\内存布局.png)

将za或b的地址，或Pp所含的内容(也是个地址)指定给pza，显然不是问题。一个pointer或一个reference 之所以支持多态，是因为它们并不引发内存中任何“与类型有关的内存委托操作(type-dependentcommitment)”;会受到改变的只是它们所指向的内存的“大小和内容解释方式”而已。然而，任何人如果企图改变object za的大小，便会违反其定义中受契约保护的“资源需求量”。如果把整个Bearobject指定给za，则会溢出它所配置得到的内存。执行结果当然也就不对了，当一个 base class object 被直接初始化为(或是被指定为)一个 derived classobject 时，derived object 就会被切割(sliced)，以塞人较小的 base type 内存中derived type 将没有留下任何蛛丝马迹。多态于是不再呈现，而一个严格的编译器可以在编译时期解析一个“通过该 obiect 而触发的 virtual function 调用操作”，因而回避 virtual 机制。如果 virtual function 被定义为 inline，则更有效率上的大收获。

# 构造函数语义学

## Default Constructor的建构操作

对于 class X，如果没有任何 user-declared constructor，那么会有一个 default constructor 被暗中(implicitly)声明出来……一个被暗中声明出来的 defaul constructor 将是一个 trivial (浅薄而无能，没啥用的)constructor…，没有存在以下四种情况而又没有声明任何 constructor 的classes，我们说它们拥有的是 implicit trivial default constructors，它们实际上并不会被合成出来。在合成的 default constructor 中，只有 base class subobjects  和 member class objects 会被初始化。所有其它的 nonstatic data member，如整数、整数指针、整数数组等等都不会被初始化。这些初始化操作对程序而言或许有要，但对编译器则并非必要。如果程序需要一个“把某指针设为0”的 default constructor，那么提供它的人应该是程序员.

**“带有 Default Constructor ”的Member Class Object**

如果一个 class 没有任何 constructor，但它内含一个 member object，而后者有default constructor，那么这个class的implicit default constructor 就是non trivial”，编译器需要为此 class合成出一个 default constructor。不过这个合成操作只有在constructor真正需要被调用时才会发生。在C++各个不同的编译模块中（不同的编译模块意指不同的档案），编译器如何避免合成出多个default constructor（譬如说一个是为 A.C 档合成,另一个是为 B.C档合成）呢?

解决方法是把合成的default constructor、copy constructor、destructor、assignment copy operator 都以 inline 方式完成。一个 inline函数有静态链接（staticlinkage），不会被档案以外者看到，如果函数太复杂，不适合做成 inline，就会合成出一个explicitnon-inlinestatic 实体。

举个例子：

```c++
class Foo{	public: Foo(); Foo(int)...	};
class Bar{	public: Foo foo; char*str;	};		//内涵非继承

void foo_bar()
{
    Bar bar;	//Bar::foo 必须在此处初始化
				//译注: Bar::foo是一个member object,而其class Foo
    			//		拥有 default constructor，符合本小节主题  
    if(bar.str){ }..
}
```

被合成的 Bar default constructor内含必要的代码，能够调用class Foo 的default constructor 来处理 member object Bar::foo，但它并不产生任何码来初始化Bar::str.是的，将Bar::str初始化是编译器的责任，将Bar::str初始化则是程序员的责任。被合成的defaultconstructor看起来可能像这样：

```c++
inline
Bar::Bar()
{
	//c++ 伪码
	foo.Foo:Foo();
}
```

被合成的 default constructor 只满足编译器的需要，而不是程序的需要。为了让这个程序片段能够正确执行，字符指针 str 也需要被初始化.让我们假设程序员经由下面的 default constructor 提供了 str 的初始化操作:

```c++
//程序员定义的 default constructor
Bar::Bar(){ str = 0; }
```

现在程序的需求获得满足了，但是编译器还需要初始化 member object foo.由于 default constructor 已经被明确地定义出来,编译器没办法合成第二个。编译器的行动：”如果classA内含一个或一个以上的 member classobjects，那么class d的每一个 constructor 必须调用每一个 member classes 的 default constructor”。编译器会扩张已存在的 constructors，在其中安插一些码使得 user code 在被执行之前，先调用必要的 default constructors。沿续前一个例子，扩张后的constructors 可能像这样:

```c++
//扩张后的 default constructor
//c++伪码
Bar::Bar()
{ 
	foo.Foo::Foo();
	str = 0;
}
```

如果有多个 class member objects 都要求 constructor 初始化操作，C++ 语言要求以“ member objects 在 class 中的声明次序”来调用各个 constructors。这一点由编译器完成，它为每一个 constructor 安插程序代码，以 member 声明次序”调用每一个 member 所关联的 default constructors.这些码将被安插在 explicit user code 之前。

**“带有 Default Constructor”的 Base Class**

如果一个没有任何 constructors 的 class 派生自一个“带有 default constructor”的 base class,那么这个 derived class 的 default constructor 会被视为 nontrivial，并因此需要被合成出来。它将调用上一层 base classes 的 default constructor(根据它们的声明次序)。对一个后继派生的 class 而言，这个合成的 constructor和一个“被明确提供的 default constructor”没有什么差异。如果设计者提供多个 constructors ，但其中都没有 default constructor ，编译器会扩张现有的每一个 constructors，将“用以调用所有必要之 default constructors ”的程序代码加进去，它不会合成一个新的 default constructor，这是因为其它“由user所提供的 constructors 存在的缘故。如果同时亦存在着“带有 default constructors ”的 member class objects，那些 default constructor 也会被调用——在所有 base class constructor 都被调用之后。

**“带有一个 Virtual Function”的 Class**

另有两种情况，也要合成出 default constructor：

- class 声明(或继承)一个 virtual function；
- 派生自一个继承串链，其中有一个或更多的 virtual base classes；

不管哪一种情况，由于缺乏由 user 声明的 constructors，编译器会详细记录合成一个 default constructor 的必要信息。以下面这个程序片段为例：

```c++
class Widget
{
	public:
		virtual void flip() = 0;
		...
};

void flip(const Widget& widget) { widget.flip(); }

//假设 Bell 和 Whistle 都派生自 Widget
void foo()
{
	Bell b;
	Whistle w;
	
	flip(b);
	flip(w);
}
```

下面两个扩张操作会在编译期间发生:

1. 一个 virtual function table(在 cfront 中被称为 vtbl)会被编译器产生出来，内放 class 的 virtual functions 地址；
2. 在每一个 class object 中，一个额外的 pointer member(也就是 vptr)会被编译器合成出来，内含相关的 class vtbl 的地址；

此外，widget.flip() 的虚拟引发操作(virtual invocation)会被重新改写，以使用 widget 的 vptr 和 vtbl 中的 flip() 条目:

```c++
//widget.flip()的虚拟引发操作（virtual invocation）的转变
(*widget.vptr[1](&widget))
```

其中:

- 1 表示 flip() 在 virtual table 中的固定索引；
- &widget 代表要交给“被调用的某个 flip() 函数实体”的this 指针；

为了让这个机制发挥功效，编译器必须为每一个 Widget(或其派生类之) object  的 vptr 设定初值，放置适当的 virtual table 地址。对于 class 所定义的每一个 constructor 编译器会安插一些码来做这样的事情(请看5.2节)。对于那未声明任何 constructor 的 classes，编译器会为它们合成一个 default constructor，以便正确地初始化每一个 class object 的 vptr。

**“带有一个 Virtual Base Class”的 Class**

Virtual base class 的实现法在不同的编译器之间有极大的差异。然而，每一种实现法的共通点在于必须使 virtual base class 在其每一个 derived cass object 中的位置，能够于执行期准备妥当。例如下面这段程序代码中：

```c++
class X { public: int i; };
class A : public virtual X { public: int j; };
class B : public virtual X { public : double d; };
class C : public A, public B { public : int k; };

//无法在编译时期决定(resolve)出pa->x::i的位置
void foo(const A* pa) { pa->i = 1024; }

main()
{
	foo( new A );
    foo( new C )
}
```

对于 foo(const A* pa)，当传入 A 或 C 的实例时，pa->i 的实际偏移位置在编译期并不固定，因为 pa 的具体类型在运行时才知道。具体而言：

- 当 pa 指向 A 的实例时，X 的位置可能直接在 A 内存布局的一部分；
- 当 pa 指向 C 的实例时，X 的位置可能更复杂，因为 C 是 A 和 B 的子类，且 X 通过虚继承共享在 A 和 B 中；

编译器必须改变“执行存取操作”的那些码，使 X::i 可以延迟至执行期才决定下来。原先 cfront 的做法是靠“在derived class obiect 的每一个 virtual base classes 中安插一个指针”完成。所有“经由 reference 或 pointer 来存取一个 virtual base class”的操作都可以通过相关指针完成。在我的例子中，foo() 可以被改写如下，以符合这样的实现策略：

```c++
//可能的编译器转变操作
void foo(const A* pa) { pa->_vbcX->i = 1024; }
```

其中 vbcX 表示编译器所产生的指针，指向 virtual base class X。正如你所臆测的那样，_vbcX(或编译器所做出的某个什么东西)是在 class object 建构期间被完成的。对于class 所定义的每一个 constructor，编译器会安插那些“允许每一个 virtual base class 的执行期存取操作”的码，如果 class 没有声明任何 constructors，编译器必须为它合成一个 default constructor.

## Copy Constructor的建构操作

如果 class 没有提供一个 explicit copy constructor 又当如何?当 class object 以“相同 class 的另一个 object ”作为初值时，其内部是以所谓的 default memberwise initialization 手法完成的，也就是把每一个内建的或派生的 data member(例如一个指针或一数目组)的值，从某个object拷贝一份到另一个 object身上。不过它并不会拷贝其中的member class object，而是以递归的方式施行 memberwise inirializaton。

如果一个 Class 没有声明一个 copy constructor，编译器就会隐式声明一个 copy constructor，只有编译器需要的时候，编译器才会定义一个 copy constructor 实例，并合成于程序之中，而编译器需要的时候是指 Class 不展现出bitwise copy semantics（位逐次拷贝）。即“如果一个 Class 未定义出 copy constructor，编译器就会自动为它产生出一个”这句话是不对的，只有当 Class 不展现出 bitwise copy semantics 时编译器才会产生一个。

**Bitwise Copy Semantics(位逐次拷贝)**

```c++
#include "Word.h"

Word noun("book");

void foo()
{
	Word verb = noun;
}
```

在使用复制构造时，如果该class没有定义 explicit copy constructor，那么是否会有一个编译器合成的实体被调用呢？这就得视该 class 是否展现"bitwise copy semantics”而定。

声明一：

```c++
class Word
{
	public:
		Word(const char*);
		~Word() { delete [] str; }
		// ...
	private:
		int cnt;
		char *str;
}
```

这种情况下并不需要合成出一个 default copy constructor，因为上述声明展现了“default copy semantics”，而 verb 的初始化操作也就不需要以一个函数调用收场。当然，程序的执行将因为 class Word 如此宣告而灾情惨重。如今local object verb 和 global object noun 都指向相同的字符串。在退出 foo() 之前，local object verb 会执行 destnuctor，于是字符串被删除，global objectnoun 从此指向一堆无意义之物。memberstr 的问题只能够靠“由 class 设计者实现出一个 explicit copy constructor 以改写 default memberwise initiaization”或是靠“不允许完全拷贝”解决之。不过这和“是否有一个 copy constructor 被编译器合成出来”没有关系。

声明二：

```c++
class Word
{
	public:
		Word(const String&);
		~Word() { delete [] str; }
		// ...
	private:
		int cnt;
		String str;
}
```

其中 String 声明了一个 explicit copy constructor :

```c++
class String
{
	public:
		String(const char*);
		String(const String&);
		~String();
		// ...
}
```

在这个情况下，编译器必须合成出一个 copy constructor 以便调用 member class String object 的 copy constructor:

```c++
//一个被合成出来的 copy constructor
//C++伪码
inline Word::Word(const Word& wd)
{
	str.String::String(wd.str);
	cnt = wd.cnt;
}
```

**不要 Bitwise Copy Semantics !**

1. 当 class 内含一个 member object 而后者的 class 声明有一个 copy constructor 时(不论是被class设计者明确地声明，就像前面的 String那样；或是被编译器合成，像class Word那样)；
2. 当 class 继承自一个 base class 而后者存在有一个 copy constructor 时(再次强调，不论是被明确声明或是被合成而得)；
3. 当 class 声明了一个或多个 virtual functions 时；
4. 当 class 派生自一个继承串链，其中有一个或多个 virtual base classes 时；

在这些情况下 class 不再保持 bitwise copy semantics”，而且 default copy constructor 如果未被声明的话，会被视为是 nontrivial。在这四种情况下，如果缺乏一个已声明的 copy constructor，编译器为了正确处理“以一个 class object 作为另一个 class object 的初值”，必须合成出一个 copy constructor。

前两种情况中，编译器必须将 member 或 base class 的“copy constructors 调用操作”安插到被合成的 copy constructor 中。后两种情况解释分析如下：

**重新设定 Virtual Table 的指针**

只要有一个 class 声明了一个或多个 virtual functions 就会如此：

- 增加一个 virtual function table(vtbl)，内含每一个有作用的 virtual function 的地址。
- 将一个指向 virtual function table 的指针(vptr)，安插在每一个classobject 内。

很显然，如果编译器对于每一个新产生的 class object 的 vptr 必须成功而正确地设好其初值。因此，当编译器导人一个 vptr 到 class 之中时，该class就不再展现 bitwise semantics 了。现在，编译器需要合成出一个 copy constructor，以求将 vptr 适当地初始化：

```c++
class ZooAnimal
{
	public:
		ZooAnimal();
		virtual ~ZooAnimal();
		
		virtual void animate();
		virtual void draw();
		// ...
	private:
		//ZooAnimal 的animate() 和 draw()
		//所需的数据
}

class Bear : public ZooAnimal
{
	public:
		Bear();
		virtual void dance();
		void animate();
		void draw();
		// ...
	private:
		//Bear 的animate() 和 draw() 和 dance()
		//所需的数据
}
```

ZooAnimal class object 以另一个 ZooAnimal class object 作为初值，或 Bearclass object 以另一个 Bear class object 作为初值，都可以直接靠“bitwise copysemantics”完成（除了可能会有的pointermember 之外）。

例如：

```c++
Bear yogi;
Bear winnie = yogi;
```

yogi会被 default Bear constructor初始化。而在 constructor中,yogi的 vptr被设定指向 Bearclass的 virtual table(靠编译器安插的码完成)。因此，把 yogi的vptr值拷贝给winnie的vptr 是安全的。

![yogi和winnie的关系](F:\Cplusplus\typoraImg\yogi和winnie的关系.png)

当一个base class object以其 derived class的object 内容做初始化操作时其vptr复制操作也必须保证安全，例如:

```c++
ZooAnimal franny = yoai; //会发生切割行为
```

franny 的 vptr 不可以被设定指向 Bear class 的 virtual table(但如果 yogi 的 franny vptr 被直接“bitwise copy”的话，就会导致此结果)，否则当下面程序片段中的 draw() 被调用而 fanny 被传进去时，就会“炸毁”(blowup)（通过 franny 调用 virtual function draw()，调用的是 Zoonimal 实体而非 Bear 实体(甚至虽然 fanny 是以 Bear object yogi 作为初值)，因为 fanny 是一个 ZooAnimal object。事实上，yogi 中的 Bear 部分已经在 fanny 初始化时被切割(sliced)掉了。如果 fanny 被声明为一个 reference (或如果它是一个指针，而其值为yogi的地址)，那么经由 fanny 所调用的 draw()才会是 Bear 的函数实体）:

```c++
void draw(const ZooAnimal& zoey) { zoey.draw(); }
void foo()
{
	//franny 的 vptr 指向 ZooAnimal 的 virtual table
	ZooAnimal franny = yogi;
	
	draw(yogi);		//Bear::draw()
	drwa(franny);	//ZooAnimal::draw(s)
}
```

![yogi和franny的关系2](F:\Cplusplus\typoraImg\yogi和franny的关系2.png)

**处理 Virtual Base Class Subobject**

一个 class object 如果以另一个 object 作为初值，而后者有一个 virtual base class subobject，那么也会使“bitwisecopy semantics”失效。因为每一个编译器对于虚拟继承的支持承诺，都表示必须让“derived class object中的 virtual base class subobject 位置”在执行期就准备妥当。维护“位置的完整性”是编译器的责任。“Bitwise copy semantics”可能会破坏这个位置，所以编译器必须在它自己合成出来的 copy constructor 中做出仲裁。

例如：

```c++
class Raccoon : public virtual ZooAnimal
{
	public:
    	//编译器所产生的代码(用以调用ZooAnimal的default constructor、将Raccoon的vptr初始化，
    	//并定位出 Raccoon中的 ZooAnimal subobject)被安插在两个 Raccoon constructors 之内。
		Raccoon() { /*设定 private data 初值*/ }
		Raccoon(int val) { /*设定 private data 初值*/ }
		// ...
	private:
		// 所有必要的数据
}

class RedPanda : public Raccoon
{
	public:
		RedPanda() { /*设定 private data 初值*/ }
		RedPanda(int val) { /*设定 private data 初值*/ }
		// ...
	private:
		// 所有必要的数据
}
```

一个 virtual base class 的存在会使 bitwise copy semantics 无效，其次，问题并不发生于“一个 class object 以另一个同类的 object 作为初值”之时，而是发生于“一个 class object 以其 derived classes 的某个 object 作为初值”之时。

```c++
RedPanda little_red;
Racoon little_critter = little_red
```

在这种情况下，为了完成正确的 little_critter 初值设定，编译器必须合成个 copy constructor，安插一些码以设定 virtual base class pointer/offset 的初值(或只是简单地确定它没有被抹消)，对每一个 members 执行必要的memberwise 初始化操作，以及执行其它的内存相关工作。

![little_red和little_critter的关系](F:\Cplusplus\typoraImg\little_red和little_critter的关系.png)

在下面的情况中，编译器无法知道是否"bitwise copy semantics”还保持着,因为它无法知道(没有流程分析) Raccoon 指针是否指向一个真正的 Raccoon object，或是指向一个 derived class object:

```c++
RedPanda *ptr;
Racoon little_critter = *ptr;
```































