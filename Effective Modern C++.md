# Effective Modern C++

## 条款一：理解模板类型推导

```c++
template<typename T>
void f(ParamType param);

f(expr);                        //从expr中推导T和ParamType
```

**情景一：ParamType是一个指针或引用，但不是通用引用：**

```c++
template<typename T>
void f(T& param);               //param是一个引用

int x=27;                       //x是int
const int cx=x;                 //cx是const int
const int& rx=x;                //rx是指向作为const int的x的引用

f(x);                           //T是int，param的类型是int&
f(cx);                          //T是const int，param的类型是const int&
f(rx);                          //T是const int，param的类型是const int&
```

```c++
template<typename T>
void f(const T& param);         //param现在是reference-to-const

int x = 27;                     //如之前一样
const int cx = x;               //如之前一样
const int& rx = x;              //如之前一样

f(x);                           //T是int，param的类型是const int&
f(cx);                          //T是int，param的类型是const int&
f(rx);                          //T是int，param的类型是const int&
```

如果`param`是一个指针（或者指向`const`的指针）而不是引用，情况本质上也一样。

**情景二：ParamType是一个通用引用：**

- 如果`expr`是左值，`T`和`ParamType`都会被推导为左值引用。这非常不寻常，第一，这是模板类型推导中唯一一种`T`被推导为引用的情况。第二，虽然`ParamType`被声明为右值引用类型，但是最后推导的结果是左值引用。
- 如果`expr`是右值，就使用正常的（也就是**情景一**）推导规则

```c++
template<typename T>
void f(T&& param);              //param现在是一个通用引用类型

int x=27;                       //如之前一样
const int cx=x;                 //如之前一样
const int & rx=cx;              //如之前一样

f(x);                           //x是左值，所以T是int&，
                                //param类型也是int&

f(cx);                          //cx是左值，所以T是const int&，
                                //param类型也是const int&

f(rx);                          //rx是左值，所以T是const int&，
                                //param类型也是const int&

f(27);                          //27是右值，所以T是int，
                                //param类型就是int&&
```

**情景三：ParamType既不是指针也不是引用：**

无论传递什么`param`都会成为它的一份拷贝——一个完整的`新对象`，即完全独立于传入实参的新对象。

- 和之前一样，如果`expr`的类型是一个引用，忽略这个引用部分
- 如果忽略`expr`的引用性（reference-ness）之后，`expr`是一个`const`，那就再忽略`const`。如果它是`volatile`，也忽略`volatile`

```c++
int x=27;                       //如之前一样
const int cx=x;                 //如之前一样
const int & rx=cx;              //如之前一样

f(x);                           //T和param的类型都是int
f(cx);                          //T和param的类型都是int
f(rx);                          //T和param的类型都是int
```

考虑这样的情况，`expr`是一个`const`指针，指向`const`对象，`expr`通过传值传递给`param`：

```c++
template<typename T>
void f(T param);                //仍然以传值的方式处理param

const char* const ptr =         //ptr是一个常量指针，指向常量对象 
    "Fun with pointers";

f(ptr);                         //传递const char * const类型的实参
```

在这里，解引用符号（*）的右边的`const`表示`ptr`本身是一个`const`：`ptr`不能被修改为指向其它地址，也不能被设置为null（解引用符号左边的`const`表示`ptr`指向一个字符串，这个字符串是`const`，因此字符串不能被修改）。当`ptr`作为实参传给`f`，组成这个指针的每一比特都被拷贝进`param`。像这种情况，`ptr`自身的值会被传给形参，根据类型推导的第三条规则，`ptr`自身的常量性`const`ness将会被省略，所以`param`是`const char*`，也就是一个可变指针指向`const`字符串。在类型推导中，这个指针指向的数据的常量性`const`ness将会被保留，但是当拷贝`ptr`来创造一个新指针`param`时，`ptr`自身的常量性`const`ness将会被忽略。

**数组实参**：

数组会退化为指针：

```c++
const char name[] = "J. P. Briggs";     //name的类型是const char[13]

const char * ptrToName = name;          //数组退化为指针
```

在这里`const char*`指针`ptrToName`会由`name`初始化，而`name`的类型为`const char[13]`，这两种类型（`const char*`和`const char[13]`）是不一样的，但是由于数组退化为指针的规则，编译器允许这样的代码。

```c++
template<typename T>
void f(T param);                        //传值形参的模板

f(name);                                //T和param会推导成什么类型?
```

我们从一个简单的例子开始，这里有一个函数的形参是数组:

```cpp
void myFunc(int param[]);
```

但是数组声明会被视作指针声明，这意味着`myFunc`的声明和下面声明是等价的：

```cpp
void myFunc(int* param);                //与上面相同的函数
```

数组与指针形参这样的等价是C语言的产物，C++又是建立在C语言的基础上，它让人产生了一种数组和指针是等价的的错觉。

因为数组形参会视作指针形参，所以传值给模板的一个数组类型会被推导为一个指针类型。这意味着在模板函数`f`的调用中，它的类型形参`T`会被推导为`const char*`：

```cpp
f(name);                        //name是一个数组，但是T被推导为const char*
```

但是现在难题来了，虽然函数不能声明形参为真正的数组，但是**可以**接受指向数组的**引用**！所以我们修改`f`为传引用：

```cpp
template<typename T>
void f(T& param);                       //传引用形参的模板
```

我们这样进行调用，

```cpp
f(name);                                //传数组给f
```

`T`被推导为了真正的数组！这个类型包括了数组的大小，在这个例子中`T`被推导为`const char[13]`，`f`的形参（该数组的引用）的类型则为`const char (&)[13]`。

可声明指向数组的引用的能力，使得我们可以创建一个模板函数来推导出数组的大小：

```cpp
//在编译期间返回一个数组大小的常量值（//数组形参没有名字，
//因为我们只关心数组的大小）
template<typename T, std::size_t N>                     //关于
constexpr std::size_t arraySize(T (&)[N]) noexcept      //constexpr
{                                                       //和noexcept
    return N;                                           //的信息
}                                                       //请看下面
```

在[Item15](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item15.html)提到将一个函数声明为`constexpr`使得结果在编译期间可用。这使得我们可以用一个花括号声明一个数组，然后第二个数组可以使用第一个数组的大小作为它的大小，就像这样：

```cpp
int keyVals[] = { 1, 3, 7, 9, 11, 22, 35 };             //keyVals有七个元素

int mappedVals[arraySize(keyVals)];                     //mappedVals也有七个
```

当然作为一个现代C++程序员，你自然应该想到使用`std::array`而不是内置的数组：

```cpp
std::array<int, arraySize(keyVals)> mappedVals;         //mappedVals的大小为7
```

至于`arraySize`被声明为`noexcept`，会使得编译器生成更好的代码，具体的细节请参见Item14。

**函数实参**：

在C++中不只是数组会退化为指针，函数类型也会退化为一个函数指针。

```c++
void someFunc(int, double);         //someFunc是一个函数，
                                    //类型是void(int, double)

template<typename T>
void f1(T param);                   //传值给f1

template<typename T>
void f2(T & param);                 //传引用给f2

f1(someFunc);                       //param被推导为指向函数的指针，
                                    //类型是void(*)(int, double)
f2(someFunc);                       //param被推导为指向函数的引用，
                                    //类型是void(&)(int, double)
```

## 条款二：理解auto类型推导

Item1基于`ParamType`——在函数模板中`param`的类型说明符——的不同特征，把模板类型推导分成三个部分来讨论。在使用`auto`作为类型说明符的变量声明中，类型说明符代替了`ParamType`，因此Item1描述的三个情景稍作修改就能适用于auto：

- 情景一：类型说明符是一个指针或引用但不是通用引用
- 情景二：类型说明符一个通用引用
- 情景三：类型说明符既不是指针也不是引用

```c++
auto x = 27;                    //情景三（x既不是指针也不是引用）
const auto cx = x;              //情景三（cx也一样）
const auto & rx=cx;             //情景一（rx是非通用引用）

auto&& uref1 = x;               //x是int左值，
                                //所以uref1类型为int&
auto&& uref2 = cx;              //cx是const int左值，
                                //所以uref2类型为const int&
auto&& uref3 = 27;              //27是int右值，
                                //所以uref3类型为int&&

```

Item1讨论并总结了对于non-reference类型说明符，数组和函数名如何退化为指针。那些内容也同样适用于`auto`类型推导：

```c++
const char name[] =             //name的类型是const char[13]
 "R. N. Briggs";

auto arr1 = name;               //arr1的类型是const char*
auto& arr2 = name;              //arr2的类型是const char (&)[13]

void someFunc(int, double);     //someFunc是一个函数，
                                //类型为void(int, double)

auto func1 = someFunc;          //func1的类型是void (*)(int, double)
auto& func2 = someFunc;         //func2的类型是void (&)(int, double)
```

声明一个带有初始值27的`int`，C++98提供两种语法选择：

```c++
int x1 = 24;
int x2(27);
```

C++11添加了用于支持统一初始化的语法：

```c++
int x3 = { 27 };
int x4{ 27 };
```

使用`auto`说明符代替指定类型说明符:

```c++
auto x1 = 27;                   //类型是int，值是27
auto x2(27);                    //同上
auto x3 = { 27 };               //类型是std::initializer_list<int>，
                                //值是{ 27 }
auto x4{ 27 };                  //同上

```

前面两个语句确实声明了一个类型为`int`值为27的变量，但是后面两个声明了一个存储一个元素27的 `std::initializer_list<int>`类型的变量。这就造成了`auto`类型推导不同于模板类型推导的特殊情况。当用`auto`声明的变量使用花括号进行初始化，`auto`类型推导推出的类型则为`std::initializer_list`。如果这样的一个类型不能被成功推导（比如花括号里面包含的是不同类型的变量），编译器会拒绝这样的代码：

```c++
auto x5 = { 1, 2, 3.0 };        //错误！无法推导std::initializer_list<T>中的T
```

当使用`auto`声明的变量使用花括号的语法进行初始化的时候，会推导出`std::initializer_list<T>`的实例化，但是对于模板类型推导这样就行不通：

```c++
auto x = { 11, 23, 9 };         //x的类型是std::initializer_list<int>

template<typename T>            //带有与x的声明等价的
void f(T param);                //形参声明的模板

f({ 11, 23, 9 });               //错误！不能推导出T
```

然而如果在模板中指定`T`是`std::initializer_list<T>`而留下未知`T`,模板类型推导就能正常工作：

```c++
template<typename T>
void f(std::initializer_list<T> initList);

f({ 11, 23, 9 });               //T被推导为int，initList的类型为
                                //std::initializer_list<int>
```

C++14允许`auto`用于函数返回值并会被推导，而且C++14的*lambda*函数也允许在形参声明中使用`auto`。但是在这些情况下`auto`实际上使用模板类型推导的那一套规则在工作，而不是`auto`类型推导，所以说下面这样的代码不会通过编译：

```c++
auto createInitList()
{
    return { 1, 2, 3 };         //错误！不能推导{ 1, 2, 3 }的类型
}

std::vector<int> v;
…
auto resetV = 
    [&v](const auto& newValue){ v = newValue; };        //C++14
…
resetV({ 1, 2, 3 });            //错误！不能推导{ 1, 2, 3 }的类型
```

## 条款三：理解decltype

给一个名字或者表达式`decltype`就会告诉你这个名字或者表达式的类型。通常，它会精确的告诉你你想要的结果。

在C++11中，`decltype`最主要的用途就是用于声明函数模板，而这个函数返回类型依赖于形参类型。

```c++
template<typename Container, typename Index>    //可以工作，
auto authAndAccess(Container& c, Index i)       //但是需要改良
    ->decltype(c[i])
{
    authenticateUser();
    return c[i];
}
```

函数名称前面的`auto`不会做任何的类型推导工作。相反的，他只是暗示使用了C++11的**尾置返回类型**语法，即在函数形参列表后面使用一个”`->`“符号指出函数的返回类型，尾置返回类型的好处是我们可以在函数返回类型中使用函数形参相关的信息。

C++11允许自动推导单一语句的*lambda*表达式的返回类型， C++14扩展到允许自动推导所有的*lambda*表达式和函数，甚至它们内含多条语句。对于`authAndAccess`来说这意味着在C++14标准下我们可以忽略尾置返回类型，只留下一个`auto`。使用这种声明形式，auto表示这里会发生类型推导。更准确的说，编译器将会从函数实现中推导出函数的返回类型。

```c++
template<typename Container, typename Index>    //C++14版本，
auto authAndAccess(Container& c, Index i)       //不那么正确
{
    authenticateUser();
    return c[i];                                //从c[i]中推导返回类型
}
```

`operator[]`对于大多数`T`类型的容器会返回一个`T&`，但Item1解释了在模板类型推导期间，表达式的引用性（reference-ness）会被忽略，即函数返回的是值的拷贝。基于这样的规则，考虑它会对下面用户的代码有哪些影响：

```c++
std::deque<int> d;
…
authAndAccess(d, 5) = 10;               //认证用户，返回d[5]，
                                        //然后把10赋值给它
                                        //无法通过编译器！
```

C++期望在某些情况下当类型被暗示时需要使用`decltype`类型推导的规则，C++14通过使用`decltype(auto)`说明符使得这成为可能。我们可以这样解释它的意义：`auto`说明符表示这个类型将会被推导，`decltype`说明`decltype`的规则将会被用到这个推导过程中。`decltype(auto)`的使用不仅仅局限于函数返回类型，当你想对初始化表达式使用`decltype`推导的规则，你也可以使用。因此我们可以这样写`authAndAccess`：

```c++
template<typename Container, typename Index>    //C++14版本，
decltype(auto)                                  //可以工作，
authAndAccess(Container& c, Index i)            //但是还需要
{                                               //改良
    authenticateUser();
    return c[i];
}
```

容器通过传引用的方式传递非常量左值引用（lvalue-reference-to-non-**const**），因为返回一个引用允许用户可以修改容器。但是这意味着在不能给这个函数传递右值容器，右值不能被绑定到左值引用上（除非这个左值引用是一个const（lvalue-references-to-**const**），但是这里明显不是）。`authAndAccess`传递一个临时变量也并不是没有意义，有时候用户可能只是想简单的获得临时容器中的一个元素的拷贝，比如这样：

```c++
std::deque<std::string> makeStringDeque();      //工厂函数

//从makeStringDeque中获得第五个元素的拷贝并返回
auto s = authAndAccess(makeStringDeque(), 5);
```

要想支持这样使用`authAndAccess`载是一个不错的选择（一个函数重载声明为左值引用，另一个声明为右值引用），但是我们就不得不维护两个重载函数。另一个方法是使`authAndAccess`的引用可以绑定左值和右值，Item24解释了那正是通用引用能做的，所以我们这里可以使用通用引用进行声明：

```c++
template<typename Containter, typename Index>   //现在c是通用引用
decltype(auto) authAndAccess(Container&& c, Index i);
```

我们还需要更新一下模板的实现（c++14版本），让它能听从Item25的告诫应用`std::forward`实现通用引用：

```c++
template<typename Container, typename Index>    //最终的C++14版本
decltype(auto)
authAndAccess(Container&& c, Index i)
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

C++11版本的模板：

```c++
template<typename Container, typename Index>    //最终的C++11版本
auto
authAndAccess(Container&& c, Index i)
->decltype(std::forward<Container>(c)[i])
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

decltype的例外行为，将`decltype`应用于变量名会产生该变量名的声明类型。虽然变量名都是左值表达式，但这不会影响`decltype`的行为。（译者注：这里是说对于单纯的变量名，`decltype`只会返回变量的声明类型）然而，对于比单纯的变量名更复杂的左值表达式，`decltype`可以确保报告的类型始终是左值引用：

```c++
decltype(auto) f1()
{
    int x = 0;
    …
    return x;                            //decltype(x）是int，所以f1返回int
}

decltype(auto) f2()
{
    int x = 0;
    return (x);                          //decltype((x))是int&，所以f2返回int&
}
```

**请记住：**

- `decltype`总是不加修改的产生变量或者表达式的类型。
- 对于`T`类型的不是单纯的变量名的左值表达式，`decltype`总是产出`T`的引用即`T&`。
- C++14支持`decltype(auto)`，就像`auto`一样，推导出类型，但是它使用`decltype`的规则进行推导。

## 条款四：学会查看类型推导结果

**std::type_info::name：**

```c++
template<typename T>                    //要调用的模板函数
void f(const T& param);

std::vector<Widget> createVec();        //工厂函数

const auto vw = createVec();            //使用工厂函数返回值初始化vw

if (!vw.empty()){
    f(&vw[0]);                          //调用f
    …
}

template<typename T>
void f(const T& param)
{
    using std::cout;
    cout << "T =     " << typeid(T).name() << '\n';             //显示T

    cout << "param = " << typeid(param).name() << '\n';         //显示
    …                                                           //param
}                                                               //的类型
```

输出结果：

```c++
T =     class Widget const *
param = class Widget const *
```

`std::type_info::name`的结果并不总是可信的。因为它们本质上可以不正确，因为`std::type_info::name`规范批准**像传值形参一样来对待这些类型**。正如Item1提到的，如果传递的是一个引用，那么引用部分（reference-ness）将被忽略，如果忽略后还具有`const`或者`volatile`，那么常量性`const`ness或者易变性`volatile`ness也会被忽略。那就是为什么`param`的类型`const Widget * const &`会输出为`const Widget *`，首先引用被忽略，然后这个指针自身的常量性`const`ness被忽略，剩下的就是指针指向一个常量对象。

**Boost.TypeIndex：**

```c++
#include <boost/type_index.hpp>

template<typename T>
void f(const T& param)
{
    using std::cout;
    using boost::typeindex::type_id_with_cvr;

    //显示T
    cout << "T =     "
         << type_id_with_cvr<T>().pretty_name()
         << '\n';
    
    //显示param类型
    cout << "param = "
         << type_id_with_cvr<decltype(param)>().pretty_name()
         << '\n';
}

std::vetor<Widget> createVec();         //工厂函数
const auto vw = createVec();            //使用工厂函数返回值初始化vw
if (!vw.empty()){
    f(&vw[0]);                          //调用f
    …
}
```

输出结果：

```c++
T =     class Widget const *
param = class Widget const * const &
```

`boost::typeindex::type_id_with_cvr`获取一个类型实参（我们想获得相应信息的那个类型），它不消除实参的`const`，`volatile`和引用修饰符（因此模板名中有“`with_cvr`”）。结果是一个`boost::typeindex::type_index`对象，它的`pretty_name`成员函数输出一个`std::string`，包含我们能看懂的类型表示。

## 条款五：优先考虑auto而非显示类型声明

`auto`变量从初始化表达式中推导出类型，所以我们必须初始化。

**声明一个局部变量，类型是一个闭包，闭包的类型只有编译器知道，因此使用auto类型推导技术：**

```c++
template<typename It>           //如之前一样
void dwim(It b,It e)
{
    while (b != e) {
        auto currValue = *b;
        …
    }
}
```

**使用auto代替std::function模板实例化：**

*lambda*表达式能产生一个可调用对象，可以把闭包存放到`std::function`对象中：

```c++
std::function<bool(const std::unique_ptr<Widget> &,
                   const std::unique_ptr<Widget> &)>
derefUPLess = [](const std::unique_ptr<Widget> &p1,
                 const std::unique_ptr<Widget> &p2)
                { return *p1 < *p2; };
```

语法冗长不说，还需要重复写很多形参类型，使用`std::function`还不如使用`auto`。用`auto`声明的变量保存一个和闭包一样类型的（新）闭包，因此使用了与闭包相同大小存储空间。实例化`std::function`并声明一个对象这个对象将会有固定的大小。这个大小可能不足以存储一个闭包，这个时候`std::function`的构造函数将会在堆上面分配内存来存储，这就造成了使用`std::function`比`auto`声明变量会消耗更多的内存。并且通过具体实现我们得知通过`std::function`调用一个闭包几乎无疑比`auto`声明的对象调用要慢。换句话说，`std::function`方法比`auto`方法要更耗空间且更慢，还可能有*out-of-memory*异常。并且正如上面的例子，比起写`std::function`实例化的类型来，使用`auto`要方便得多：

c++11版本：

```c++
auto derefUPLess = 
    [](const std::unique_ptr<Widget> &p1,       //用于std::unique_ptr
       const std::unique_ptr<Widget> &p2)       //指向的Widget类型的
    { return *p1 < *p2; };                      //比较函数
```

c++14版本：

```c++
auto derefLess =                                //C++14版本
    [](const auto& p1,                          //被任何像指针一样的东西
       const auto& p2)                          //指向的值的比较函数
    { return *p1 < *p2; };
```

**避免与类型快捷方式（type shortcuts）有关的问题：**

```c++
std::vector<int> v;
…
unsigned sz = v.size();
```

`v.size()`的标准返回类型是`std::vector<int>::size_type`，但是只有少数开发者意识到这点。`std::vector<int>::size_type`实际上被指定为无符号整型，所以很多人都认为用`unsigned`就足够了，写下了上述的代码。这会造成一些有趣的结果。举个例子，在Windows 32-bit上`std::vector<int>::size_type`和`unsigned`是一样的大小，但是在Windows 64-bit上`std::vector<int>::size_type`是64位，`unsigned`是32位。这意味着这段代码在Windows 32-bit上正常工作，但是当把应用程序移植到Windows 64-bit上时就可能会出现一些问题。所以使用`auto`可以确保你不需要浪费时间：

```c++
auto sz =v.size();                      //sz的类型是std::vector<int>::size_type
```

再看下面的代码：

```c++
std::unordered_map<std::string, int> m;
…

for(const std::pair<std::string, int>& p : m)
{
    …                                   //用p做一些事
}
```

要想看到错误你就得知道`std::unordered_map`的*key*是`const`的，所以*hash table*（`std::unordered_map`本质上的东西）中的`std::pair`的类型不是`std::pair<std::string, int>`，而是`std::pair<const std::string, int>`。但那不是在循环中的变量`p`声明的类型。编译器会努力的找到一种方法把`std::pair<const std::string, int>`（即*hash table*中的东西）转换为`std::pair<std::string, int>`（`p`的声明类型）。它会成功的，因为它会通过拷贝`m`中的对象创建一个临时对象，这个临时对象的类型是`p`想绑定到的对象的类型，即`m`中元素的类型，然后把`p`的引用绑定到这个临时对象上。在每个循环迭代结束时，临时对象将会销毁，如果你写了这样的一个循环，你可能会对它的一些行为感到非常惊讶，因为你确信你只是让成为`p`指向`m`中各个元素的引用而已。使用`auto`可以避免这些很难被意识到的类型不匹配的错误：

```c++
for(const auto& p : m)
{
    …                                   //如之前一样
}
```

## 条款六：auto推导若非己愿，使用显示类型初始化惯用法

假如我有一个函数，参数为`Widget`，返回一个`std::vector<bool>`，这里的`bool`表示`Widget`是否提供一个独有的特性。更进一步假设第5个*bit*表示`Widget`是否具有高优先级，我们可以写这样的代码：

```c++
std::vector<bool> features(const Widget& w);

Widget w;
…
bool highPriority = features(w)[5];
//这里，features返回一个std::vector<bool>对象后再调用operator[]，operator[]将会返回一个std::vector<bool>::reference对象，然后再通过隐式转换赋值给bool变量highPriority。highPriority因此表示的是features返回的std::vector<bool>中的第五个bit，这也正如我们所期待的那样。
…
processWidget(w, highPriority);         //根据它的优先级处理w

auto highPriority = features(w)[5];     //w高优先级吗？

processWidget(w,highPriority);          //未定义行为！
//同样的，features返回一个std::vector<bool>对象，再调用operator[]，operator[]将会返回一个std::vector<bool>::reference对象，但是现在这里有一点变化了，auto推导highPriority的类型为std::vector<bool>::reference，但是highPriority对象没有第五bit的值。
```

使用`auto`后`highPriority`不再是`bool`类型。虽然从概念上来说`std::vector<bool>`意味着存放`bool`，但是`std::vector<bool>`的`operator[]`不会返回容器中元素的引用（这就是`std::vector::operator[]`可返回**除了`bool`以外**的任何类型），取而代之它返回一个`std::vector<bool>::reference`的对象（一个嵌套于`std::vector<bool>`中的类）。`std::vector<bool>::reference`之所以存在是因为`std::vector<bool>`规定了使用一个打包形式（packed form）表示它的`bool`，每个`bool`占一个*bit*。那给`std::vector`的`operator[]`带来了问题，因为`std::vector<T>`的`operator[]`应当返回一个`T&`，但是C++禁止对`bit`s的引用。无法返回一个`bool&`，`std::vector<bool>`的`operator[]`返回一个**行为类似于**`bool&`的对象。要想成功扮演这个角色，`bool&`适用的上下文`std::vector<bool>::reference`也必须一样能适用。在`std::vector<bool>::reference`的特性中，使这个原则可行的特性是一个可以向`bool`的隐式转化。这个值取决于`std::vector<bool>::reference`的具体实现。其中的一种实现是这样的（`std::vector<bool>::reference`）对象包含一个指向机器字（*word*）的指针，然后加上方括号中的偏移实现被引用*bit*这样的行为。然后再来考虑`highPriority`初始化表达的意思，注意这里假设`std::vector<bool>::reference`就是刚提到的实现方式。调用`features`将返回一个`std::vector<bool>`临时对象，这个对象没有名字，为了方便我们的讨论，我这里叫他`temp`。`operator[]`在`temp`上调用，它返回的`std::vector<bool>::reference`包含一个指向存着这些*bit*s的一个数据结构中的一个*word*的指针（`temp`管理这些*bit*s），还有相应于第5个*bit*的偏移。`highPriority`是这个`std::vector<bool>::reference`的拷贝，所以`highPriority`也包含一个指针，指向`temp`中的这个*word*，加上相应于第5个*bit*的偏移。在这个语句结束的时候`temp`将会被销毁，因为它是一个临时变量。因此`highPriority`包含一个悬置的（*dangling*）指针，如果用于`processWidget`调用中将会造成未定义行为。

显式类型初始器惯用法使用`auto`声明一个变量，然后对表达式强制类型转换（*cast*）得出你期望的推导结果：

```c++
auto highPriority = static_cast<bool>(features(w)[5]);
```

这里，`features(w)[5]`还是返回一个`std::vector<bool>::reference`对象，就像之前那样，但是这个转型使得表达式类型为`bool`，然后`auto`才被用于推导`highPriority`。在运行时，对`std::vector<bool>::operator[]`返回的`std::vector<bool>::reference`执行它支持的向`bool`的转型，在这个过程中指向`std::vector<bool>`的指针已经被解引用。这就避开了我们之前的未定义行为。然后5将被用于指向*bit*的指针，`bool`值被用于初始化`highPriority`。

显式类型初始器惯用法也可以用于**强调**你声明了一个变量类型，它的类型不同于初始化表达式的类型。举个例子，假设你有这样一个表达式计算公差值：

```c++
double calcEpsilon();                           //返回公差值
float ep = calcEpsilon();                       //double到float隐式转换
```

但是这几乎没有表明“我确实要减少函数返回值的精度”。使用显式类型初始器惯用法我们可以这样：

```c++
auto ep = static_cast<float>(calcEpsilon());
```

**请记住：**

- 不可见的代理类可能会使`auto`从表达式中推导出“错误的”类型
- 显式类型初始器惯用法强制`auto`推导出你想要的结果

## 条款七：区别使用()和{}创建对象

**内置类型间隐式的变窄转换：**

花括号表达式还有一个少见的特性，即它不允许内置类型间隐式的变窄转换（*narrowing conversion*）。如果一个使用了花括号初始化的表达式的值，不能保证由被初始化的对象的类型来表示，代码就不会通过编译：

```c++
double x, y, z;

int sum1{ x + y + z };          //错误！double的和可能不能表示为int
```

但括号表达式可以：

```c++
int sum2(x + y +z);             //可以（表达式的值被截为int）

int sum3 = x + y + z;           //同上
```

**花括号初始化最令人头疼的解析问题：**

问题的根源是如果你调用带参构造函数，你可以这样做：

```c++
Widget w1(10);                  //使用实参10调用Widget的一个构造函数
```

但是如果你尝试使用相似的语法调用`Widget`无参构造函数，它就会变成函数声明：

```c++
Widget w2();                    //最令人头疼的解析！声明一个函数w2，返回Widget
```

由于函数声明中形参列表不能带花括号，所以使用花括号初始化表明你想调用默认构造函数构造对象就没有问题：

```cpp
Widget w3{};                    //调用没有参数的构造函数构造对象
```

**括号初始化的缺点：**

在构造函数调用中，只要不包含`std::initializer_list`形参，那么花括号初始化和圆括号初始化都会产生一样的结果：

```cpp
class Widget { 
public:  
    Widget(int i, bool b);      //构造函数未声明
    Widget(int i, double d);    //std::initializer_list这个形参 
    …
};
Widget w1(10, true);            //调用第一个构造函数
Widget w2{10, true};            //也调用第一个构造函数
Widget w3(10, 5.0);             //调用第二个构造函数
Widget w4{10, 5.0};             //也调用第二个构造函数
```

然而，如果有一个或者多个构造函数的声明包含一个`std::initializer_list`形参，那么使用括号初始化语法的调用更倾向于选择带`std::initializer_list`的那个构造函数。如果编译器遇到一个括号初始化并且有一个带std::initializer_list的构造函数，那么它一定会选择该构造函数。如果上面的`Widget`类有一个`std::initializer_list<long double>`作为参数的构造函数，就像这样：

```cpp
class Widget { 
public:  
    Widget(int i, bool b);      //同上
    Widget(int i, double d);    //同上
    Widget(std::initializer_list<long double> il);      //新添加的
    …
}; 
```

`w2`和`w4`将会使用新添加的构造函数，即使另一个非`std::initializer_list`构造函数和实参更匹配：

```cpp
Widget w1(10, true);    //使用圆括号初始化，同之前一样
                        //调用第一个构造函数

Widget w2{10, true};    //使用花括号初始化，但是现在
                        //调用带std::initializer_list的构造函数
                        //(10 和 true 转化为long double)

Widget w3(10, 5.0);     //使用圆括号初始化，同之前一样
                        //调用第二个构造函数 

Widget w4{10, 5.0};     //使用花括号初始化，但是现在
                        //调用带std::initializer_list的构造函数
                        //(10 和 5.0 转化为long double)
```

甚至普通构造函数和移动构造函数都会被带`std::initializer_list`的构造函数劫持：

```cpp
class Widget { 
public:  
    Widget(int i, bool b);                              //同之前一样
    Widget(int i, double d);                            //同之前一样
    Widget(std::initializer_list<long double> il);      //同之前一样
    operator float() const;                             //转换为float
    …
};

Widget w5(w4);                  //使用圆括号，调用拷贝构造函数

Widget w6{w4};                  //使用花括号，调用std::initializer_list构造
                                //函数（w4转换为float，float转换为double）

Widget w7(std::move(w4));       //使用圆括号，调用移动构造函数

Widget w8{std::move(w4)};       //使用花括号，调用std::initializer_list构造
                                //函数（与w6相同原因）
```

编译器一遇到括号初始化就选择带`std::initializer_list`的构造函数的决心是如此强烈，以至于就算带`std::initializer_list`的构造函数不能被调用，它也会硬选。

```cpp
class Widget { 
public: 
    Widget(int i, bool b);                      //同之前一样
    Widget(int i, double d);                    //同之前一样
    Widget(std::initializer_list<bool> il);     //现在元素类型为bool
    …                                           //没有隐式转换函数
};

Widget w{10, 5.0};              //错误！要求变窄转换
```

这里，编译器会直接忽略前面两个构造函数（其中第二个构造函数是所有实参类型的最佳匹配），然后尝试调用`std::initializer_list<bool>`构造函数。调用这个函数将会把`int(10)`和`double(5.0)`转换为`bool`，由于会产生变窄转换（`bool`不能准确表示其中任何一个值），括号初始化拒绝变窄转换，所以这个调用无效，代码无法通过编译。

只有当没办法把括号初始化中实参的类型转化为`std::initializer_list`时，编译器才会回到正常的函数决议流程中。比如我们在构造函数中用`std::initializer_list<std::string>`代替`std::initializer_list<bool>`，这时非`std::initializer_list`构造函数将再次成为函数决议的候选者，因为没有办法把`int`和`bool`转换为`std::string`:

```cpp
class Widget { 
public:  
    Widget(int i, bool b);                              //同之前一样
    Widget(int i, double d);                            //同之前一样
    //现在std::initializer_list元素类型为std::string
    Widget(std::initializer_list<std::string> il);
    …                                                   //没有隐式转换函数
};

Widget w1(10, true);     // 使用圆括号初始化，调用第一个构造函数
Widget w2{10, true};     // 使用花括号初始化，现在调用第一个构造函数
Widget w3(10, 5.0);      // 使用圆括号初始化，调用第二个构造函数
Widget w4{10, 5.0};      // 使用花括号初始化，现在调用第二个构造函数
```

代码的行为和我们刚刚的论述如出一辙。这里还有一个有趣的[边缘情况](https://en.wikipedia.org/wiki/Edge_case)。假如你使用的花括号初始化是空集，并且你欲构建的对象有默认构造函数，也有`std::initializer_list`构造函数。你的空的花括号意味着什么？如果它们意味着没有实参，就该使用默认构造函数，但如果它意味着一个空的`std::initializer_list`，就该调用`std::initializer_list`构造函数。

最终会调用默认构造函数。空的花括号意味着没有实参，不是一个空的`std::initializer_list`：

```cpp
class Widget { 
public:  
    Widget();                                   //默认构造函数
    Widget(std::initializer_list<int> il);      //std::initializer_list构造函数

    …                                           //没有隐式转换函数
};

Widget w1;                      //调用默认构造函数
Widget w2{};                    //也调用默认构造函数
Widget w3();                    //最令人头疼的解析！声明一个函数
```

如果你**想**用空`std::initializer`来调用`std::initializer_list`构造函数，你就得创建一个空花括号作为函数实参——把空花括号放在圆括号或者另一个花括号内来界定你想传递的东西。

```cpp
Widget w4({});                  //使用空花括号列表调用std::initializer_list构造函数
Widget w5{{}};                  //同上
```

你可能会想研究半天这些东西在你的日常编程中到底占多大比例。可能比你想象的要有用。`std::vector`有一个非`std::initializer_list`构造函数允许你去指定容器的初始大小，以及使用一个值填满你的容器。但它也有一个`std::initializer_list`构造函数允许你使用花括号里面的值初始化容器。如果你创建一个数值类型的`std::vector`（比如`std::vector<int>`），然后你传递两个实参，把这两个实参放到圆括号和放到花括号中有天壤之别：

```cpp
std::vector<int> v1(10, 20);    //使用非std::initializer_list构造函数
                                //创建一个包含10个元素的std::vector，
                                //所有的元素的值都是20
std::vector<int> v2{10, 20};    //使用std::initializer_list构造函数
                                //创建包含两个元素的std::vector，
                                //元素的值为10和20
```

第一，作为一个类库作者，你需要意识到如果一堆重载的构造函数中有一个或者多个含有`std::initializer_list`形参，用户代码如果使用了括号初始化，可能只会看到你`std::initializer_list`版本的重载的构造函数。因此，你最好把你的构造函数设计为不管用户是使用圆括号还是使用花括号进行初始化都不会有什么影响。换句话说，了解了`std::vector`设计缺点后，你以后设计类的时候应该避免诸如此类的问题。

第二，作为一个类库使用者，你必须认真的在花括号和圆括号之间选择一个来创建对象。大多数开发者都使用其中一种作为默认情况，只有当他们不能使用这种的时候才会考虑另一种。默认使用花括号初始化的开发者主要被适用面广、禁止变窄转换、免疫C++最令人头疼的解析这些优点所吸引。这些开发者知道在一些情况下（比如给定一个容器大小和一个初始值创建`std::vector`）要使用圆括号。默认使用圆括号初始化的开发者主要被C++98语法一致性、避免`std::initializer_list`自动类型推导、避免不会不经意间调用`std::initializer_list`构造函数这些优点所吸引。这些开发者也承认有时候只能使用花括号（比如创建一个包含着特定值的容器）。

如果你是一个模板的作者，花括号和圆括号创建对象就更麻烦了。通常不能知晓哪个会被使用。举个例子，假如你想创建一个接受任意数量的参数来创建的对象。使用可变参数模板（*variadic template*）可以非常简单的解决：

```cpp
template<typename T,            //要创建的对象类型
         typename... Ts>        //要使用的实参的类型
void doSomeWork(Ts&&... params)
{
    create local T object from params...
    …
} 
```

在现实中我们有两种方式实现这个伪代码（关于`std::forward`请参见Item25）：

```cpp
T localObject(std::forward<Ts>(params)...);             //使用圆括号
T localObject{std::forward<Ts>(params)...};             //使用花括号
```

考虑这样的调用代码：

```cpp
std::vector<int> v; 
…
doSomeWork<std::vector<int>>(10, 20);
```

如果`doSomeWork`创建`localObject`时使用的是圆括号，`std::vector`就会包含10个元素。如果`doSomeWork`创建`localObject`时使用的是花括号，`std::vector`就会包含2个元素。哪个是正确的？`doSomeWork`的作者不知道，只有调用者知道。

## 条款八：优先考虑nullptr而非0和NULL

在C++98中，如果给下面的重载函数传递`0`或`NULL`，它们绝不会调用指针版本的重载函数：

```cpp
void f(int);        //三个f的重载函数
void f(bool);
void f(void*);

f(0);               //调用f(int)而不是f(void*)

f(NULL);            //可能不会被编译，一般来说调用f(int)，
                    //绝对不会调用f(void*)
```

而`f(NULL)`的不确定行为是由`NULL`的实现不同造成的。如果`NULL`被定义为`0L`（指的是`0`为`long`类型），这个调用就具有二义性，因为从`long`到`int`的转换或从`long`到`bool`的转换或`0L`到`void*`的转换都同样好。有趣的是源代码**表现出**的意思（“我使用空指针`NULL`调用`f`”）和**实际表达出**的意思（“我是用整型数据而不是空指针调用`f`”）是相矛盾的。

`nullptr`的优点是它不是整型。老实说它也不是一个指针类型，但是你可以把它认为是**所有**类型的指针。`nullptr`的真正类型是`std::nullptr_t`，在一个完美的循环定义以后，`std::nullptr_t`又被定义为`nullptr`。`std::nullptr_t`可以隐式转换为指向任何内置类型的指针，这也是为什么`nullptr`表现得像所有类型的指针。

使用`nullptr`调用`f`将会调用`void*`版本的重载函数，因为`nullptr`不能被视作任何整型：

```cpp
f(nullptr);         //调用重载函数f的f(void*)版本
```

使用`nullptr`代替`0`和`NULL`可以避开了那些令人奇怪的函数重载决议，这不是它的唯一优势。它也可以使代码表意明确，尤其是当涉及到与`auto`声明的变量一起使用时：

```c++
auto result = findRecord( /* arguments */ );
if (result == 0) {
    …
} 

auto result = findRecord( /* arguments */ );

if (result == nullptr) {  
    …
}
```

第一个`if`中，如果你不知道`findRecord`返回了什么（或者不能轻易的找出），那么你就不太清楚到底`result`是一个指针类型还是一个整型。毕竟，`0`（用来测试`result`的值的那个）也可以像我们之前讨论的那样被解析；第二个`if`就没有任何歧义：`result`的结果一定是指针类型。

当模板出现时`nullptr`就更有用了。假如你有一些函数只能被合适的已锁互斥量调用。每个函数都有一个不同类型的指针：

```c++
int    f1(std::shared_ptr<Widget> spw);     //只能被合适的
double f2(std::unique_ptr<Widget> upw);     //已锁互斥量
bool   f3(Widget* pw);                      //调用

template<typename FuncType,
         typename MuxType,
         typename PtrType>
decltype(auto) lockAndCall(FuncType func,       //C++14
                           MuxType& mutex,
                           PtrType ptr)
{ 
    MuxGuard g(mutex);  
    return func(ptr); 
}

auto result1 = lockAndCall(f1, f1m, 0);         //错误！
...
auto result2 = lockAndCall(f2, f2m, NULL);      //错误！
...
auto result3 = lockAndCall(f3, f3m, nullptr);   //没问题
```

在第一个调用中存在的问题是当`0`被传递给`lockAndCall`模板，模板类型推导会尝试去推导实参类型，`0`的类型总是`int`，所以这就是这次调用`lockAndCall`实例化出的`ptr`的类型。不幸的是，这意味着`lockAndCall`中`func`会被`int`类型的实参调用，这与`f1`期待的`std::shared_ptr<Widget>`形参不符。当`NULL`被传递给`lockAndCall`，形参`ptr`被推导为整型（译注：由于依赖于具体实现所以不一定是整数类型，所以用整型泛指`int`，`long`等类型），然后当`ptr`——一个`int`或者类似`int`的类型——传递给`f2`的时候就会出现类型错误，`f2`期待的是`std::unique_ptr<Widget>`。当`nullptr`传给`lockAndCall`时，`ptr`被推导为`std::nullptr_t`。当`ptr`被传递给`f3`的时候，隐式转换使`std::nullptr_t`转换为`Widget*`，因为`std::nullptr_t`可以隐式转换为任何指针类型。

**记住**

- 优先考虑`nullptr`而非`0`和`NULL`
- 避免重载指针和整型

## 条款九：优先考虑别名声明而非typedef

别名声明可以被模板化（这种情况下称为别名模板*alias template*s）但是`typedef`不能。这使得C++11程序员可以很直接的表达一些C++98中只能把`typedef`嵌套进模板化的`struct`才能表达的东西。

考虑一个链表的别名，链表使用自定义的内存分配器，`MyAlloc`：

```cpp
template<typename T>                            //MyAllocList<T>是
using MyAllocList = std::list<T, MyAlloc<T>>;   //std::list<T, MyAlloc<T>>的同义词

MyAllocList<Widget> lw;                         //用户代码
```

使用`typedef`，你就只能从头开始：

```cpp
template<typename T>                            //MyAllocList<T>是
struct MyAllocList {                            //std::list<T, MyAlloc<T>>
    typedef std::list<T, MyAlloc<T>> type;      //的同义词  
};

MyAllocList<Widget>::type lw;                   //用户代码
```

如果你想使用在一个模板内使用`typedef`声明一个链表对象，而这个对象又使用了模板形参，要使用这样的嵌套从属类型必须在前面加上`typename`，因为当编译器在`Widget`的模板中看到`MyAllocList<T>::type`（使用`typedef`的版本），它不能确定那是一个类型的名称。因为可能存在一个`MyAllocList`的它们没见到的特化版本，那个版本的`MyAllocList<T>::type`指代了一种不是类型的东西：

```c++
template<typename T>
class Widget {                              //Widget<T>含有一个
private:                                    //MyAllocLIst<T>对象
    typename MyAllocList<T>::type list;     //作为数据成员
    …
}; 
```

如果使用别名声明定义一个`MyAllocList`，就不需要使用`typename`（同时省略麻烦的“`::type`”后缀），因为当编译器处理`Widget`模板时遇到`MyAllocList<T>`（使用模板别名声明的版本），它们知道`MyAllocList<T>`是一个类型名，因为`MyAllocList`是一个别名模板：它**一定**是一个类型名。因此`MyAllocList<T>`就是一个**非依赖类型**（*non-dependent type*），就不需要也不允许使用`typename`修饰符：

```cpp
template<typename T> 
using MyAllocList = std::list<T, MyAlloc<T>>;   //同之前一样

template<typename T> 
class Widget {
private:
    MyAllocList<T> list;                        //没有“typename”
    …                                           //没有“::type”
};
```

C++11的类型转换工具:

```c++
std::remove_const<T>::type          //从const T中产出T
std::remove_reference<T>::type      //从T&和T&&中产出T
std::add_lvalue_reference<T>::type  //从T中产出T&
```

如果你在一个模板内部将他们施加到类型形参上（实际代码中你也总是这么用），你也需要在它们前面加上`typename`。因为这些C++11的*type traits*是通过在`struct`内嵌套`typedef`来实现的。

对于C++11的类型转换`std::`transformation`<T>::type`在C++14中变成了`std::`transformation`_t`：：

```c++
std::remove_const<T>::type          //C++11: const T → T 
std::remove_const_t<T>              //C++14 等价形式

std::remove_reference<T>::type      //C++11: T&/T&& → T 
std::remove_reference_t<T>          //C++14 等价形式

std::add_lvalue_reference<T>::type  //C++11: T → T& 
std::add_lvalue_reference_t<T>      //C++14 等价形式
```

如果没办法使用C++14，可以使用别名模板：

```c++
template <class T> 
using remove_const_t = typename remove_const<T>::type;

template <class T> 
using remove_reference_t = typename remove_reference<T>::type;

template <class T> 
using add_lvalue_reference_t =
  typename add_lvalue_reference<T>::type; 
```

**请记住：**

- `typedef`不支持模板化，但是别名声明支持。
- 别名模板避免了使用“`::type`”后缀，而且在模板中使用`typedef`还需要在前面加上`typename`
- C++14提供了C++11所有*type traits*转换的别名声明版本

## 条款十：优先考虑限域enum而非未限域enum

**限域`enum`枚举名泄露：**

```cpp
enum Color {black, white, red};			//black, white, red在
                                   		//Color所在的作用域
auto white = false;                 	//错误! white早已在这个作用
                                    	//域中声明
```

```cpp
enum class Color { black, white, red }; //black, white, red
                                        //限制在Color域内
auto white = false;                     //没问题，域内没有其他“white”

Color c = white;                        //错误，域中没有枚举名叫white

Color c = Color::white;                 //没问题
auto c = Color::white;                  //也没问题（也符合Item5的建议）
```

**在限域`enum`的作用域中，枚举名是强类型：**

未限域`enum`中的枚举名会隐式转换为整型（现在，也可以转换为浮点类型）：

```c++
enum Color { black, white, red };       //未限域enum

std::vector<std::size_t>                //func返回x的质因子
  primeFactors(std::size_t x);

Color c = red;
…

if (c < 14.5) {                         // Color与double比较 (!)
    auto factors =                      // 计算一个Color的质因子(!)
      primeFactors(c);
    …
}
```

不存在任何隐式转换可以将限域`enum`中的枚举名转化为任何其他类型：

```c++
enum class Color { black, white, red }; //Color现在是限域enum

Color c = Color::red;                   //和之前一样，只是
...                                     //多了一个域修饰符

if (c < 14.5) {                         //错误！不能比较
                                        //Color和double
    auto factors =                      //错误！不能向参数为std::size_t
      primeFactors(c);                  //的函数传递Color参数
    …
}
```

可以使用static_cast<>强制类型转换：

```c++
if (static_cast<double>(c) < 14.5) {    //奇怪的代码，
                                        //但是有效
    auto factors =                                  //有问题，但是
      primeFactors(static_cast<std::size_t>(c));    //能通过编译
    …
}
```

**限域`enum`可以被前置声明：**

```cpp
enum Color;         //错误！
enum class Color;   //没问题
```

非限域`enum`不能被前置声明的原因是编译器在前置声明时**无法确定它的底层类型**，看下面的两份代码：

```cpp
enum Color { black, white, red };
```

编译器可能选择`char`作为底层类型，因为这里只需要表示三个值。然而，有些`enum`中的枚举值范围可能会大些，比如：

```cpp
enum Status { good = 0,
              failed = 1,
              incomplete = 100,
              corrupt = 200,
              indeterminate = 0xFFFFFFFF
            };
```

这里值的范围从`0`到`0xFFFFFFFF`。除了在不寻常的机器上（比如一个`char`至少有32bits的那种），编译器都会选择一个比`char`大的整型类型来表示`Status`。

为了高效使用内存，编译器通常在确保能包含所有枚举值的前提下为`enum`选择一个最小的底层类型。在一些情况下，编译器将会优化速度，舍弃大小，这种情况下它可能不会选择最小的底层类型，但它们当然希望能够针对大小进行优化。为此，C++98只支持`enum`定义（所有枚举名全部列出来）；`enum`声明是不被允许的。这使得编译器能在使用之前为每一个`enum`选择一个底层类型。

但是不能前置声明最大的缺点莫过于它可能增加编译依赖，比如上面的`Status`，这种`enum`很有可能用于整个系统，因此系统中每个包含这个头文件的组件都会依赖它。如果引入一个新状态值那么可能整个系统都得重新编译，即使只有一个子系统——或者只有一个函数——使用了新添加的枚举名。C++11中的前置声明`enum`s可以解决这个问题。

在C++11中，限域`enum`的底层类型总是已知的，而对于非限域`enum`，你可以指定它：

默认情况下，限域枚举的底层类型是`int`：

```cpp
enum class Status;                  //底层类型是int
```

如果默认的`int`不适用，你可以重写它：

```cpp
enum class Status: std::uint32_t;   //Status的底层类型
                                    //是std::uint32_t
                                    //（需要包含 <cstdint>）
```

不管怎样，编译器都知道限域`enum`中的枚举名占用多少字节。

要为非限域`enum`指定底层类型，你可以同上，结果就可以前向声明：

```cpp
enum Color: std::uint8_t;   //非限域enum前向声明
                            //底层类型为
                            //std::uint8_t
```

底层类型说明也可以放到`enum`定义处：

```cpp
enum class Status: std::uint32_t { good = 0,
                                   failed = 1,
                                   incomplete = 100,
                                   corrupt = 200,
                                   audited = 500,
                                   indeterminate = 0xFFFFFFFF
                                 };
```

**非限域`enum`的好处：**

假设我们有一个*tuple*保存了用户的名字，email地址，声望值：

```
using UserInfo =                //类型别名，参见Item9
    std::tuple<std::string,     //名字
               std::string,     //email地址
               std::size_t> ;   //声望
UserInfo uInfo;                 //tuple对象
…
auto val = std::get<1>(uInfo);	//获取第一个字段
```

你应该记住第一个字段代表用户的email地址吗？我认为不。可以使用非限域`enum`将名字和字段编号关联起来以避免上述需求：

```cpp
enum UserInfoFields { uiName, uiEmail, uiReputation };

UserInfo uInfo;                         //同之前一样
…
auto val = std::get<uiEmail>(uInfo);    //啊，获取用户email字段的值
```

之所以它能正常工作是因为`UserInfoFields`中的枚举名隐式转换成`std::size_t`了，其中`std::size_t`是`std::get`模板实参所需的。

但对应的限域`enum`版本很啰嗦：

```cpp
enum class UserInfoFields { uiName, uiEmail, uiReputation };

UserInfo uInfo;                         //同之前一样
…
auto val =
    std::get<static_cast<std::size_t>(UserInfoFields::uiEmail)>
        (uInfo);
```

`std::get`是一个模板（函数），需要你给出一个`std::size_t`值的模板实参（注意使用`<>`而不是`()`），因此将枚举名变换为`std::size_t`值的函数必须**在编译期**产生这个结果。如Item15提到的，那必须是一个`constexpr`函数。

```cpp
template<typename E>
constexpr typename std::underlying_type<E>::type
    toUType(E enumerator) noexcept
{
    return
        static_cast<typename
                    std::underlying_type<E>::type>(enumerator);
}
```

在C++14中，`toUType`还可以进一步用`std::underlying_type_t`（参见[Item9](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item9.html)）代替`typename std::underlying_type<E>::type`打磨：

```cpp
template<typename E>                //C++14
constexpr std::underlying_type_t<E>
    toUType(E enumerator) noexcept
{
    return static_cast<std::underlying_type_t<E>>(enumerator);
}
```

还可以再用C++14 `auto`（参见[Item3](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item3.html)）打磨一下代码：

```cpp
template<typename E>                //C++14
constexpr auto
    toUType(E enumerator) noexcept
{
    return static_cast<std::underlying_type_t<E>>(enumerator);
}
```

不管它怎么写，`toUType`现在允许这样访问tuple的字段了：

```cpp
auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);
```

这仍然比使用非限域`enum`要写更多的代码，但同时它也避免命名空间污染，防止不经意间使用隐式转换。



**记住**

- C++98的`enum`即非限域`enum`。
- 限域`enum`的枚举名仅在`enum`内可见。要转换为其它类型只能使用*cast*。
- 非限域/限域`enum`都支持底层类型说明语法，限域`enum`底层类型默认是`int`。非限域`enum`没有默认底层类型。
- 限域`enum`总是可以前置声明。非限域`enum`仅当指定它们的底层类型时才能前置。

## 条款十一：优先考虑使用deleted函数而非使用未定义的私有声明

c++98中防止调用拷贝构造函数或者赋值运算符的方法是将它们声明为私有函数：

```c++
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
    …

private:
    basic_ios(const basic_ios& );           // not defined
    basic_ios& operator=(const basic_ios&); // not defined
};
```

将它们声明为私有成员可以防止客户端调用这些函数。故意不定义它们意味着假如还是有代码用它们（比如成员函数或者类的友元`friend`），就会在链接时引发缺少函数定义（*missing function definitions*）错误。

c++11中的` = delete`可以实现相同的效果：

```c++
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
    …

    basic_ios(const basic_ios& ) = delete;
    basic_ios& operator=(const basic_ios&) = delete;
    …
};
```

删除这些函数（译注：添加"`= delete`"）和声明为私有成员可能看起来只是方式不同，别无其他区别。其实还有一些实质性意义。*deleted*函数不能以任何方式被调用，即使你在成员函数或者友元函数里面调用*deleted*函数也不能通过编译。**这是较之C++98行为的一个改进，C++98中不正确的使用这些函数在链接时才被诊断出来**。

***deleted*函数有一个重要的优势是任何函数都可以标记为*deleted*，而只有成员函数可被标记为`private`:**

假如我们有一个非成员函数，它接受一个整型参数，检查它是否为幸运数：

```cpp
bool isLucky(int number);

if (isLucky('a')) …         //字符'a'是幸运数？
if (isLucky(true)) …        //"true"是?
if (isLucky(3.5)) …         //难道判断它的幸运之前还要先截尾成3？
```

创建*deleted*重载函数，其参数就是我们想要过滤的类型：

```c++
bool isLucky(int number);       //原始版本
bool isLucky(char) = delete;    //拒绝char
bool isLucky(bool) = delete;    //拒绝bool
bool isLucky(double) = delete;  //拒绝float和double
```

***deleted*还可以禁止一些模板的实例化：**

加入下面的模板仅支持原生指针：

```c++
template<typename T>
void processPointer(T* ptr);
```

一是`void*`指针，因为没办法对它们进行解引用，或者加加减减等。另一种指针是`char*`，因为它们通常代表C风格的字符串，而不是正常意义下指向单个字符的指针。这两种情况要特殊处理，在`processPointer`模板里面，我们假设正确的函数应该拒绝这些类型。也即是说，`processPointer`不能被`void*`和`char*`调用：

```c++
template<>
void processPointer<void>(void*) = delete;

template<>
void processPointer<char>(char*) = delete;

template<>
void processPointer<const void>(const void*) = delete;

template<>
void processPointer<const char>(const char*) = delete;
```

模板特例化必须位于一个命名空间作用域（全局命名空间或者某个命名空间中的代码块。这是大多数函数、变量和类型定义所在的层次），而不是类作用域，因此下面的代码出错：

```c++
class Widget {
public:
    …
    template<typename T>
    void processPointer(T* ptr)
    { … }

private:
    template<>                          //错误！
    void processPointer<void>(void*);
    
};
```

*deleted*函数不会出现这个问题，因为它不需要一个不同的访问级别，且他们可以在类外被删除（因此位于命名空间作用域）：

```cpp
class Widget {
public:
    …
    template<typename T>
    void processPointer(T* ptr)
    { … }
    …

};

template<>                                          //还是public，
void Widget::processPointer<void>(void*) = delete;  //但是已经被删除了
```



**请记住：**

- 比起声明函数为`private`但不定义，使用*deleted*函数更好
- 任何函数都能被删除（be deleted），包括非成员函数和模板实例（译注：实例化的函数）

## 条款十二：使用override声明重载函数

要想重写一个函数，必须满足下列要求：

- 基类函数必须是`virtual`
- 基类和派生类函数名必须完全一样（除非是析构函数)
- 基类和派生类函数形参类型必须完全一样
- 基类和派生类函数常量性`const`ness必须完全一样
- 基类和派生类函数的返回值和异常说明（*exception specifications*）必须兼容

除了这些C++98就存在的约束外，C++11又添加了一个：

- 函数的引用限定符（*reference qualifiers*）必须完全一样。它可以限定成员函数只能用于左值或者右值。成员函数不需要`virtual`也能使用它们：

```cpp
class Base {
public:
    virtual void mf1() const;
    virtual void mf2(int x);
    virtual void mf3() &;
    void mf4() const;
};

class Derived: public Base {
public:
    virtual void mf1();
    virtual void mf2(unsigned int x);
    virtual void mf3() &&;
    void mf4() const;
};
```

基类`Derived`中的函数并没有重写同名基类函数，并且绝大部分编译器未必能检测出全部的`warning`。

**C++11可以显式地指定一个派生类函数是基类版本的重写：将它声明为`override`：**

```cpp
class Derived: public Base {
public:
    virtual void mf1() override;
    virtual void mf2(unsigned int x) override;
    virtual void mf3() && override;
    virtual void mf4() const override;
};
```

代码不能编译，编译器会抱怨所有与重写有关的问题。除此之外，如果你考虑修改修改基类虚函数的函数签名，`override`还可以帮你评估后果。如果派生类全都用上`override`，你可以只改变基类函数签名，重编译系统，再看看你造成了多大的问题（即，多少派生类不能通过编译），然后决定是否值得如此麻烦更改函数签名。

对于`override`，它只在成员函数声明结尾处才被视为关键字。这意味着如果你以前写的代码里面已经用过**override**这个名字，那么换到C++11标准你也无需修改代码（final也一样）：

```cpp
class Warning {         //C++98潜在的传统类代码
public:
    …
    void override();    //C++98和C++11都合法（且含义相同）
    …
};
```

**成员函数引用限定：**

假设我们的`Widget`类有一个`std::vector`数据成员，提供一个访问函数让客户端可以直接访问它：

```c++
class Widget {
public:
    using DataType = std::vector<double>;   //“using”的信息参见Item9
    …
    DataType& data() { return values; }
    …
private:
    DataType values;
};
```

```cpp
Widget w;
…
auto vals1 = w.data();  //拷贝w.values到vals1
```

`Widget::data`函数的返回值是一个左值引用（准确的说是`std::vector<double>&`）, 因为左值引用是左值，所以`vals1`是从左值初始化的。因此`vals1`由`w.values`拷贝构造而得。

现在假设有一个创建`Widget`s的工厂函数，

```cpp
Widget makeWidget();
```

我们想用`makeWidget`返回的`Widget`里的`std::vector`初始化一个变量：

```cpp
auto vals2 = makeWidget().data();   //拷贝Widget里面的值到vals2
```

`Widgets::data`返回的是左值引用，还有，左值引用是左值。所以，我们的对象（`vals2`）得从`Widget`里的`values`拷贝构造。这一次，`Widget`是`makeWidget`返回的临时对象（即右值），所以将其中的`std::vector`进行拷贝纯属浪费。最好是移动，但是因为`data`返回左值引用，C++的规则要求编译器不得不生成一个拷贝。

我们需要的是指明当`data`被右值`Widget`对象调用的时候结果也应该是一个右值。现在就可以使用引用限定，为左值`Widget`和右值`Widget`写一个`data`的重载函数来达成这一目的：

```cpp
class Widget {
public:
    using DataType = std::vector<double>;
    …
    DataType& data() &              //对于左值Widgets,
    { return values; }              //返回左值
    
    DataType data() &&              //对于右值Widgets,
    { return std::move(values); }   //返回右值
    …

private:
    DataType values;
};
```

注意`data`重载的返回类型是不同的，左值引用重载版本返回一个左值引用（即一个左值），右值引用重载返回一个临时对象（即一个右值）。这意味着现在客户端的行为和我们的期望相符了：

```cpp
auto vals1 = w.data();              //调用左值重载版本的Widget::data，
                                    //拷贝构造vals1
auto vals2 = makeWidget().data();   //调用右值重载版本的Widget::data, 
                                    //移动构造vals2
```



**请记住：**

- 为重写函数加上`override`
- 成员函数引用限定让我们可以区别对待左值对象和右值对象（即`*this`)

## 条款十三：优先考虑const_iterator而非iterator

在C++98中，标准库对`const_iterator`的支持不是很完整：

```c++
std::vector<int> values;
…
std::vector<int>::iterator it =
    std::find(values.begin(), values.end(), 1983);
values.insert(it, 1998);
```

因为这段代码不修改`iterator`指向的内容，所以应该用`const_iterator`重写这段代码，下面是一种在c++98中概念上可行但是不正确的方法：

```c++
typedef std::vector<int>::iterator IterT;               //typedef
typedef std::vector<int>::const_iterator ConstIterT;

std::vector<int> values;
…
ConstIterT ci =
    std::find(static_cast<ConstIterT>(values.begin()),  //cast
              static_cast<ConstIterT>(values.end()),    //cast
              1983);

values.insert(static_cast<IterT>(ci), 1998);    //可能无法通过编译，
                                                //原因见下
```

使用`typedef`而不是Item9提到的别名声明，因为这段代码在演示C++98做法，别名声明是C++11加入的特性。之所以`std::find`的调用会出现类型转换是因为在C++98中`values`是non-`const`容器，没办法简简单单的从non-`const`容器中获取`const_iterator`。严格来说类型转换不是必须的，因为用其他方法获取`const_iterator`也是可以的（比如你可以把`values`绑定到reference-to-`const`变量上，然后再用这个变量代替`values`），但不管怎么说，从non-`const`容器中获取`const_iterator`的做法都有点别扭。当你费劲地获得了`const_iterator`，事情可能会变得更糟，因为C++98中，插入操作（以及删除操作）的位置只能由`iterator`指定，`const_iterator`是不被接受的。这也是我在上面的代码中，将`const_iterator`（我那么小心地从`std::find`搞出来的东西）转换为`iterator`的原因，因为向`insert`传入`const_iterator`不能通过编译。上面的代码也可能无法编译，因为没有一个可移植的从`const_iterator`到`iterator`的方法，即使使用`static_cast`也不行。甚至传说中的牛刀`reinterpret_cast`也不行。（它不是C++98的限制，也不是C++11的限制，只是`const_iterator`就是不能转换为`iterator`）。因此，C++98的`const_iterator`不是那么实用。

C++11中`const_iterator`既容易获取又容易使用。容器的成员函数`cbegin`和`cend`产出`const_iterator`，甚至对于non-`const`容器也可用，那些之前使用*iterator*指示位置（如`insert`和`erase`）的STL成员函数也可以使用`const_iterator`了。使用C++11 `const_iterator`重写C++98使用`iterator`的代码也稀松平常：

```c++
std::vector<int> values;                                //和之前一样
…
auto it =                                               //使用cbegin
    std::find(values.cbegin(), values.cend(), 1983);//和cend
values.insert(it, 1998);
```

唯一一个C++11对于`const_iterator`支持不足（译注：C++14支持但是C++11的时候还没）的情况是：当你想写最大程度通用的库，并且这些库代码为一些容器和类似容器的数据结构提供`begin`、`end`（以及`cbegin`，`cend`，`rbegin`，`rend`等）作为**非成员函数**而不是成员函数时。其中一种情况就是原生数组，还有一种情况是一些只由自由函数组成接口的第三方库。最大程度通用的库会考虑使用非成员函数而不是假设成员函数版本存在。

例如下面的例子：

```c++
template<typename C, typename V>
void findAndInsert(C& container,            //在容器中查找第一次
                   const V& targetVal,      //出现targetVal的位置，
                   const V& insertVal)      //然后在那插入insertVal
{
    using std::cbegin;
    using std::cend;

    auto it = std::find(cbegin(container),  //非成员函数cbegin
                        cend(container),    //非成员函数cend
                        targetVal);
    container.insert(it, insertVal);
}
```

它可以在C++14工作良好，但是C++11不在良好之列。由于标准化的疏漏，C++11只添加了非成员函数`begin`和`end`，但是没有添加`cbegin`，`cend`，`rbegin`，`rend`，`crbegin`，`crend`。

可以简单的写下自己的实现。比如，下面就是非成员函数`cbegin`的实现：

```cpp
template <class C>
auto cbegin(const C& container)->decltype(std::begin(container))
{
    return std::begin(container);   //解释见下
}
```

这个`cbegin`模板接受任何代表类似容器的数据结构的实参类型`C`，并且通过reference-to-`const`形参`container`访问这个实参。如果`C`是一个普通的容器类型（如`std::vector<int>`），`container`将会引用一个`const`版本的容器（如`const std::vector<int>&`）。对`const`容器调用非成员函数`begin`（由C++11提供）将产出`const_iterator`，这个迭代器也是模板要返回的。用这种方法实现的好处是就算容器只提供`begin`成员函数（对于容器来说，C++11的非成员函数`begin`调用这些成员函数）不提供`cbegin`成员函数也没问题。那么现在你可以将这个非成员函数`cbegin`施于只直接支持`begin`的容器。

如果`C`是原生数组，这个模板也能工作。这时，`container`成为一个`const`数组的引用。C++11为数组提供特化版本的非成员函数`begin`，它返回指向数组第一个元素的指针。一个`const`数组的元素也是`const`，所以对于`const`数组，非成员函数`begin`返回指向`const`的指针（pointer-to-`const`）。在数组的上下文中，所谓指向`const`的指针（pointer-to-`const`），也就是`const_iterator`了。



**请记住：**

- 优先考虑`const_iterator`而非`iterator`
- 在最大程度通用的代码中，优先考虑非成员函数版本的`begin`，`end`，`rbegin`等，而非同名成员函数

## 条款十四：如果函数不抛出异常请使用noexcept

**`noexcept`是函数接口的一部分，这意味着调用者可能会依赖它**

**`noexcept`函数较之于non-`noexcept`函数更容易优化:**

```cpp
int f(int x) throw();   //C++98风格，没有来自f的异常
int f(int x) noexcept;  //C++11风格，没有来自f的异常
```

如果在运行时，`f`出现一个异常，那么就和`f`的异常说明冲突了。在C++98的异常说明中，调用栈（the *call stack*）会展开至`f`的调用者，在一些与这地方不相关的动作后，程序被终止。C++11异常说明的运行时行为有些不同：调用栈只是**可能**在程序终止前展开。

展开调用栈和**可能**展开调用栈两者对于代码生成（code generation）有非常大的影响。在一个`noexcept`函数中，当异常可能传播到函数外时，优化器不需要保证运行时栈（the runtime stack）处于可展开状态；也不需要保证当异常离开`noexcept`函数时，`noexcept`函数中的对象按照构造的反序析构。而标注“`throw()`”异常声明的函数缺少这样的优化灵活性，没加异常声明的函数也一样。可以总结一下：

```cpp
RetType function(params) noexcept;  //极尽所能优化
RetType function(params) throw();   //较少优化
RetType function(params);           //较少优化
```

**`noexcept`对于移动语义，`swap`，内存释放函数和析构函数非常有用:**

`Widget`通过`push_back`一次又一次的添加进`std::vector`：

```cpp
std::vector<Widget> vw;
…
Widget w;
…                   //用w做点事
vw.push_back(w);    //把w添加进vw
…
```

在 C++98 中，当 `std::vector` 的容量已满时，需要为新元素分配一块更大的内存空间，并把原来的元素从旧内存区域**复制**到新内存区域。这是通过调用每个元素的复制构造函数来完成的。为了保证强异常安全性，如果在这个过程中抛出异常，`std::vector` 会保证其状态保持不变，元素还会留在旧的内存中，只有当所有的元素都复制成功后，才会析构旧内存中的元素。

C++11 引入了移动语义，它允许对象通过"移动"的方式转移所有权，而不是复制。这在需要频繁分配和释放资源的场景下能带来显著的性能提升。当 `std::vector` 需要扩展存储时，如果元素类型支持移动构造，编译器会优先选择移动操作而不是复制操作，这能减少不必要的复制开销。**问题是**: 如果在移动过程中抛出异常，可能会导致 `std::vector` 的状态不一致。假设你已经成功将前 `n` 个元素从旧内存移动到了新内存区域，但在移动第 `n+1` 个元素时抛出了异常，此时旧内存中的元素已被部分移动到新内存，`std::vector` 的状态就可能变得不可恢复了。由于移动操作不像复制那样能提供强异常安全性（除非移动操作被明确标记为 `noexcept`），因此如果移动操作可能抛出异常，直接使用移动语义可能导致程序的异常安全性被破坏。C++98 的 `std::vector::push_back` 函数能提供强异常保证，但 C++11 的移动语义引入了新的风险。C++11 引入了 `noexcept` 关键字，用来标识一个函数是否能抛出异常。通过标记移动构造函数为 `noexcept`，编译器可以确定这个操作在任何情况下都不会抛出异常。因此，当 `std::vector` 的实现遇到一个支持 `noexcept` 移动操作的类型时，它可以安全自动使用移动语义操作替代复制操作。

C++11 的 `std::vector::push_back` 会遵循一个策略：**如果移动操作是 `noexcept` 的，就使用移动操作；否则，回退到使用复制操作**。这样做可以在提供最佳性能的同时，保持与 C++98 的强异常保证一致。

`swap`函数是`noexcept`的另一个绝佳用地。`swap`是STL算法实现的一个关键组件，它也常用于拷贝运算符重载中。标准库的`swap`是否`noexcept`有时依赖于用户定义的`swap`是否`noexcept`。比如，数组和`std::pair`的`swap`声明如下：

```cpp
template <class T, size_t N>
void swap(T (&a)[N],
          T (&b)[N]) noexcept(noexcept(swap(*a, *b)));  //见下文

template <class T1, class T2>
struct pair {
    …
    void swap(pair& p) noexcept(noexcept(swap(first, p.first)) &&
                                noexcept(swap(second, p.second)));
    …
};
```

这些函数**视情况**`noexcept`：它们是否`noexcept`依赖于`noexcept`声明中的表达式是否`noexcept`。假设有两个`Widget`数组，交换数组操作为`noexcept`的前提是数组中的元素交换是`noexcept`的，即`Widget`的`swap`是`noexcept`。因此`Widget`的`swap`的作者决定了交换`widget`的数组是否`noexcept`。对于`Widget`的交换是否`noexcept`决定了对于`Widget`数组的交换是否`noexcept`，以及其他交换，比如`Widget`的数组的数组的交换是否`noexcept`。类似地，交换两个含有`Widget`的`std::pair`是否`noexcept`依赖于`Widget`的`swap`是否`noexcept`。事实上交换高层次数据结构是否`noexcept`取决于它的构成部分的那些低层次数据结构是否`noexcept`，这激励你只要可以就提供`noexcept` `swap`函数（译注：因为如果你的函数不提供`noexcept`保证，其它依赖你的高层次`swap`就不能保证`noexcept`）。

在C++98，允许内存释放（memory deallocation）函数（即`operator delete`和`operator delete[]`）和析构函数抛出异常是糟糕的代码设计，C++11将这种作风升级为语言规则。默认情况下，内存释放函数和析构函数——不管是用户定义的还是编译器生成的——都是隐式`noexcept`。因此它们不需要声明`noexcept`。（这么做也不会有问题，只是不合常规）。析构函数非隐式`noexcept`的情况仅当类的数据成员（包括继承的成员还有继承成员内的数据成员）明确声明它的析构函数可能抛出异常（如声明“`noexcept(false)`”）。这种析构函数不常见，标准库里面没有。如果一个对象的析构函数可能被标准库使用（比如在容器内或者被传给一个算法），析构函数又可能抛异常，那么程序的行为是未定义的。

**大多数函数是异常中立的（译注：可能抛也可能不抛异常）而不是`noexcept`：**

异常中立函数允许那些抛出异常的函数在调用链上更进一步直到遇到异常处理程序，而不是就地终止。异常中立函数决不应该声明为`noexcept`，因为它们可能抛出那种“让它们过吧”的异常（译注：也就是说在当前这个函数内不处理异常，但是又不立即终止程序，而是让调用这个函数的函数处理异常。即在`catch`块中添加`throw`语句，将异常继续传播至调用该函数的函数）因此大多数函数缺少`noexcept`设计。

**一般情况下需要用到`noexcept`的函数：**

- 移动构造函数和移动赋值运算符
- 析构函数：析构函数不应抛出异常。如果析构函数抛出异常，且该异常在另一个异常处理中抛出，会导致 `std::terminate()` 被调用。为了避免这种情况，析构函数应该声明为 `noexcept`。
- 交换函数（swap）：在实现自定义的交换函数时，通常希望它不会抛出异常，以便能够在 `std::swap` 或标准库容器的实现中高效使用。
- 简单的访问器函数：对于不修改类状态的简单访问器函数（如获取某个成员变量的值），如果这些函数不会抛出异常，可以声明为 `noexcept`。
- 处理基本数据类型或指针的函数：对于只操作基本数据类型（如 `int`, `double`）或指针的简单函数，这些操作通常不会抛出异常。声明为 `noexcept` 可以提高性能。
- 异常安全保证函数：如果一个函数的设计保证不会抛出异常，则应该使用 `noexcept` 来明确其意图，这有助于优化。
- 模板和泛型代码：在模板或泛型代码中，特别是当编写库时，使用 `noexcept` 可以提高代码的灵活性和性能，尤其是在需要确保异常安全时。

## 条款十五：尽可能的使用constexpr

**`constexpr`对象是`const`，它被在编译期可知的值初始化，但又不同于`const`：**

“其值编译期可知”的常量整数会出现在需要“整型常量表达式（**integral constant expression**）的上下文中，这类上下文包括数组大小，整数模板参数（包括`std::array`对象的长度），枚举名的值，对齐修饰符（译注：[`alignas(val)`](https://en.cppreference.com/w/cpp/language/alignas)），等等。如果你想在这些上下文中使用变量，你一定会希望将它们声明为`constexpr`，因为编译器会确保它们是编译期可知的：

```c++
constexpr auto arraySize2 = 10;     //没问题，10是
                                    //编译期可知常量
std::array<int, arraySize2> data2;  //没问题, arraySize2是constexpr
```

`constexpr`函数用于需求编译期常量的上下文时，如果你传给`constexpr`函数的实参在编译期可知，那么结果将在编译期计算。如果实参的值在编译期不知道，你的代码就会被拒绝:

```cpp
int sz;                             //non-constexpr变量
…
constexpr auto arraySize1 = sz;     //错误！sz的值在
                                    //编译期不可知
std::array<int, sz> data1;          //错误！一样的问题
```

注意`const`不提供`constexpr`所能保证之事，因为`const`对象不需要在编译期初始化它的值。

```cpp
int sz;                            //和之前一样
…
const auto arraySize = sz;         //没问题，arraySize是sz的const复制
std::array<int, arraySize> data;   //错误，arraySize值在编译期不可知
```

简而言之，**所有`constexpr`对象都是`const`，但不是所有`const`对象都是`constexpr`。**如果你想编译器保证一个变量有一个值，这个值可以放到那些需要编译期常量（compile-time constants）的上下文的地方，你需要的工具是`constexpr`而不是`const`。

**当传递编译期可知的值时，`constexpr`函数可以产出编译期可知的结果：**

举个例子：

```cpp
constexpr                                   //pow是绝不抛异常的
int pow(int base, int exp) noexcept         //constexpr函数
{
 …                                          //实现在下面
}
constexpr auto numConds = 5;                //（上面例子中）条件的个数
std::array<int, pow(3, numConds)> results;  //结果有3^numConds个元素
```

因为`constexpr`函数必须能在编译期值调用的时候返回编译期结果，就必须对它的实现施加一些限制。这些限制在C++11和C++14标准间有所出入：

c++11中，`constexpr`函数的代码不超过一行语句：一个`return`：

```c++
constexpr int pow(int base, int exp) noexcept
{
    return (exp == 0 ? 1 : base * pow(base, exp - 1));
}
```

c++14中，`constexpr`函数的限制变得非常宽松：

```cpp
constexpr int pow(int base, int exp) noexcept   //C++14
{
    auto result = 1;
    for (int i = 0; i < exp; ++i) result *= base;
    
    return result;
}
```

`constexpr`函数限制为只能获取和返回**字面值类型**，这基本上意味着那些有了值的类型能在编译期决定。在C++11中，除了`void`外的所有内置类型，以及一些用户定义类型都可以是字面值类型，因为构造函数和其他成员函数可能是`constexpr`：

```cpp
class Point {
public:
    constexpr Point(double xVal = 0, double yVal = 0) noexcept
    : x(xVal), y(yVal)
    {}

    constexpr double xValue() const noexcept { return x; } 
    constexpr double yValue() const noexcept { return y; }

    void setX(double newX) noexcept { x = newX; }
    void setY(double newY) noexcept { y = newY; }

private:
    double x, y;
};
```

`Point`的构造函数可被声明为`constexpr`，因为如果传入的参数在编译期可知，`Point`的数据成员也能在编译器可知。因此这样初始化的`Point`就能为`constexpr`：

```cpp
constexpr Point p1(9.4, 27.7);  //没问题，constexpr构造函数
                                //会在编译期“运行”
constexpr Point p2(28.8, 5.3);  //也没问题
```

类似的，`xValue`和`yValue`的*getter*（取值器）函数也能是`constexpr`，因为如果对一个编译期已知的`Point`对象（如一个`constexpr` `Point`对象）调用*getter*，数据成员`x`和`y`的值也能在编译期知道。这使得我们可以写一个`constexpr`函数，里面调用`Point`的*getter*并初始化`constexpr`的对象：

```cpp
constexpr
Point midpoint(const Point& p1, const Point& p2) noexcept
{
    return { (p1.xValue() + p2.xValue()) / 2,   //调用constexpr
             (p1.yValue() + p2.yValue()) / 2 }; //成员函数
}
constexpr auto mid = midpoint(p1, p2);      //使用constexpr函数的结果
                                            //初始化constexpr对象
```

它意味着`mid`对象通过调用构造函数，*getter*和非成员函数来进行初始化过程就能在只读内存中被创建出来！它也意味着你可以在模板实参或者需要枚举名的值的表达式里面使用像`mid.xValue() * 10`的表达式！（因为`Point::xValue`返回`double`，`mid.xValue() * 10`也是个`double`。浮点数类型不可被用于实例化模板或者说明枚举名的值，但是它们可以被用来作为产生整数值的大表达式的一部分。比如，`static_cast<int>(mid.xValue() * 10)`可以被用来实例化模板或者说明枚举名的值。）它也意味着以前相对严格的编译期完成的工作和运行时完成的工作的界限变得模糊，一些传统上在运行时的计算过程能并入编译时。越多这样的代码并入，你的程序就越快。（然而，编译会花费更长时间）

在C++11中，有两个限制使得`Point`的成员函数`setX`和`setY`不能声明为`constexpr`。第一，它们修改它们操作的对象的状态， 并且在C++11中，`constexpr`成员函数是隐式的`const`。第二，它们有`void`返回类型，`void`类型不是C++11中的字面值类型。这两个限制在C++14中放开了，所以C++14中`Point`的*setter*（赋值器）也能声明为`constexpr`：

```cpp
class Point {
public:
    …
    constexpr void setX(double newX) noexcept { x = newX; } //C++14
    constexpr void setY(double newY) noexcept { y = newY; } //C++14
    …
};
```

现在也能写这样的函数：

```cpp
//返回p相对于原点的镜像
constexpr Point reflection(const Point& p) noexcept
{
    Point result;                   //创建non-const Point
    result.setX(-p.xValue());       //设定它的x和y值
    result.setY(-p.yValue());
    return result;                  //返回它的副本
}
```

客户端代码可以这样写：

```cpp
constexpr Point p1(9.4, 27.7);          //和之前一样
constexpr Point p2(28.8, 5.3);
constexpr auto mid = midpoint(p1, p2);

constexpr auto reflectedMid =         //reflectedMid的值
    reflection(mid);                  //(-19.1, -16.5)在编译期可知
```

**`constexpr`对象和函数可以使用的范围比non-`constexpr`对象和函数要大：**

当一个`constexpr`函数被一个或者多个编译期不可知值调用时，它就像普通函数一样，运行时计算它的结果。这意味着你不需要两个函数，一个用于编译期计算，一个用于运行时计算。`constexpr`全做了。

前面的例子中，`pow`前面的`constexpr`不表明`pow`返回一个`const`值，它只说了如果`base`和`exp`是编译期常量，`pow`的值可以被当成编译期常量使用。如果`base`和/或`exp`不是编译期常量，`pow`结果将会在运行时计算。这意味着`pow`不止可以用于像`std::array`的大小这种需要编译期常量的地方，它也可以用于运行时环境：

```cpp
auto base = readFromDB("base");     //运行时获取这些值
auto exp = readFromDB("exponent"); 
auto baseToExp = pow(base, exp);    //运行时调用pow函数
```

**`constexpr`是对象和函数接口的一部分**

## 条款十六：让const成员函数线程安全

**保证线程安全：**

看下面一个多项式的类，roots用来计算多项式的根，他不会更改多项式，因此被声明为`const`函数。计算多项式的根是很复杂的，因此如果不需要的话，我们就不做。如果必须做，我们肯定不想再做第二次。所以，如果必须计算它们，就缓存多项式的根，然后实现`roots`来返回缓存的值：

```c++
class Polynomial {
public:
    using RootsType = std::vector<double>;
    
    RootsType roots() const
    {
        if (!rootsAreValid) {               //如果缓存不可用
            …                               //计算根
                                            //用rootVals存储它们
            rootsAreValid = true;
        }
        
        return rootVals;
    }
    
private:
    mutable bool rootsAreValid{ false };    //初始化器（initializer）的
    mutable RootsType rootVals{};           //更多信息请查看条款7
};
```

从概念上讲，`roots`并不改变它所操作的`Polynomial`对象。但是作为缓存的一部分，它也许会改变`rootVals`和`rootsAreValid`的值。因为即使成员函数被声明为`const`，被声明为`mutable`的变量也可更改。

假设现在有两个线程同时调用`Polynomial`对象的`roots`方法，`roots`是`const`成员函数，那就表示着它是一个读操作，在没有同步的情况下，让多个线程执行读操作是安全的。它最起码应该做到这点。在本例中却没有做到线程安全。因为在`roots`中，这些线程中的一个或两个可能尝试修改成员变量`rootsAreValid`和`rootVals`。这就意味着在没有同步的情况下，这些代码会有不同的线程读写相同的内存，这就是数据竞争（*data race*）的定义。这段代码的行为是未定义的。

```c++
class Polynomial {
public:
    using RootsType = std::vector<double>;
    
    RootsType roots() const
    {
        std::lock_guard<std::mutex> g(m);       //锁定互斥量
        
        if (!rootsAreValid) {                   //如果缓存无效
            …                                   //计算/存储根值
            rootsAreValid = true;
        }
        
        return rootsVals;
    }                                           //解锁互斥量
    
private:
    mutable std::mutex m;
    mutable bool rootsAreValid { false };
    mutable RootsType rootsVals {};
};
```

`std::mutex m`被声明为`mutable`，因为锁定和解锁它的都是non-`const`成员函数。在`roots`（`const`成员函数）中，`m`却被视为`const`对象。因此，为了能够在 `const` 成员函数中修改 `std::mutex m`，必须将它声明为 `mutable`。`mutable` 关键字允许 `const` 成员函数在不破坏 `const` 承诺的情况下修改某些特定成员变量。（（译者注：实际上 `std::mutex` 既不可移动，也不可复制。因而包含他们的类也同时是不可移动和不可复制的。）

**使用`std::atomic`变量可能比互斥量提供更好的性能：**

在某些情况下，互斥量的副作用显会得过大。例如，如果你所做的只是计算成员函数被调用了多少次，使用`std::atomic` 修饰的计数器（保证其他线程视它的操作为不可分割的整体，参见[item40](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item40.html)）通常会是一个开销更小的方法。以下是如何使用`std::atomic`来统计调用次数。（译者注：与 `std::mutex` 类似的，实际上 `std::atomic` 既不可移动，也不可复制。因而包含他们的类也同时是不可移动和不可复制的。）

```c++
class Point {                                   //2D点
public:
    …
    double distanceFromOrigin() const noexcept  //noexcept的使用
    {                                           //参考条款14
        ++callCount;                            //atomic的递增
        
        return std::sqrt((x * x) + (y * y));
    }

private:
    mutable std::atomic<unsigned> callCount{ 0 };
    double x, y;
};
```

因为对`std::atomic`变量的操作通常比互斥量的获取和释放的消耗更小，所以你可能会过度倾向与依赖`std::atomic`。例如，在一个类中，缓存一个开销昂贵的`int`，你就会尝试使用一对`std::atomic`变量而不是互斥量。

```c++
class Widget {
public:
    …
    int magicValue() const
    {
        if (cacheValid) return cachedValue;
        else {
            auto val1 = expensiveComputation1();
            auto val2 = expensiveComputation2();
            cachedValue = val1 + val2;              //第一步
            cacheValid = true;                      //第二步
            return cachedValid;
        }
    }
    
private:
    mutable std::atomic<bool> cacheValid{ false };
    mutable std::atomic<int> cachedValue;
};
```

这是可行的，但难以避免有时出现重复计算的情况。考虑：

- 一个线程调用`Widget::magicValue`，将`cacheValid`视为`false`，执行这两个昂贵的计算，并将它们的和分配给`cachedValue`。
- 此时，第二个线程调用`Widget::magicValue`，也将`cacheValid`视为`false`，因此执行刚才完成的第一个线程相同的计算。（这里的“第二个线程”实际上可能是其他**几个**线程。）

这种行为与使用缓存的目的背道而驰。将`cachedValue`和`CacheValid`的赋值顺序交换可以解决这个问题，但结果会更糟：

```c++
class Widget {
public:
    …
    int magicValue() const
    {
        if (cacheValid) return cachedValue;
        else {
            auto val1 = expensiveComputation1();
            auto val2 = expensiveComputation2();
            cacheValid = true;                      //第一步
            return cachedValue = val1 + val2;       //第二步
        }
    }
    …
}
```

假设`cacheValid`是false，那么：

- 一个线程调用`Widget::magicValue`，刚执行完将`cacheValid`设置`true`的语句。
- 在这时，第二个线程调用`Widget::magicValue`，检查`cacheValid`。看到它是`true`，就返回`cacheValue`，即使第一个线程还没有给它赋值。因此返回的值是不正确的。

**对于需要同步的是单个的变量或者内存位置，使用`std::atomic`就足够了。不过，一旦你需要对两个以上的变量或内存位置作为一个单元来操作的话，就应该使用互斥量：**

对于`Widget::magicValue`是这样的：

```c++
class Widget {
public:
    …
    int magicValue() const
    {
        std::lock_guard<std::mutex> guard(m);   //锁定m
        
        if (cacheValid) return cachedValue;
        else {
            auto val1 = expensiveComputation1();
            auto val2 = expensiveComputation2();
            cachedValue = val1 + val2;
            cacheValid = true;
            return cachedValue;
        }
    }                                           //解锁m
    …

private:
    mutable std::mutex m;
    mutable int cachedValue;                    //不再用atomic
    mutable bool cacheValid{ false };           //不再用atomic
};
```

现在，这个条款是基于，多个线程可以同时在一个对象上执行一个`const`成员函数这个假设的。如果你不是在这种情况下编写一个`const`成员函数——你可以**保证**在一个对象上永远不会有多个线程执行该成员函数——该函数的线程安全是无关紧要的。比如，为独占单线程使用而设计的类的成员函数是否线程安全并不重要。然而，这种线程无关的情况越来越少见，而且很可能会越来越少。可以肯定的是，`const`成员函数应支持并发执行，这就是为什么你应该确保`const`成员函数是线程安全的。



**请记住：**

- 确保`const`成员函数线程安全，除非你**确定**它们永远不会在并发上下文（*concurrent context*）中使用。
- 使用`std::atomic`变量可能比互斥量提供更好的性能，但是它只适合操作单个变量或内存位置。

## 条款十七：理解特殊成员函数的生成

**C++11对于特殊成员函数处理的规则如下：**

- **默认构造函数**：和C++98规则相同。仅当类不存在用户声明的构造函数时才自动生成。
- **析构函数**：基本上和C++98相同；稍微不同的是现在析构默认`noexcept`（参见[Item14](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item14.html)）。和C++98一样，仅当基类析构为虚函数时该类析构才为虚函数。
- **拷贝构造函数**：和C++98运行时行为一样：逐成员拷贝non-static数据。仅当类没有用户定义的拷贝构造时才生成。如果类声明了移动操作它就是*delete*的。当用户声明了拷贝赋值或者析构，该函数自动生成已被废弃。
- **拷贝赋值运算符**：和C++98运行时行为一样：逐成员拷贝赋值non-static数据。仅当类没有用户定义的拷贝赋值时才生成。如果类声明了移动操作它就是*delete*的。当用户声明了拷贝构造或者析构，该函数自动生成已被废弃。
- **移动构造函数**和**移动赋值运算符**：都对非static数据执行逐成员移动。仅当类没有用户定义的拷贝操作，移动操作或析构时才自动生成。

这个规则告诉我们如果你声明了拷贝构造函数，拷贝赋值运算符，或者析构函数三者之一，你应该也声明其余两个。它来源于长期的观察，即用户接管拷贝操作的需求几乎都是因为该类会做其他资源的管理，这也几乎意味着（1）无论哪种资源管理如果在一个拷贝操作内完成，也应该在另一个拷贝操作内完成（2）类的析构函数也需要参与资源的管理（通常是释放）。通常要管理的资源是内存，这也是为什么标准库里面那些管理内存的类（如会动态内存管理的STL容器）都声明了“*the big three*”：拷贝构造，拷贝赋值和析构。

*Rule of Three*带来的后果就是只要出现用户定义的析构函数就意味着简单的逐成员拷贝操作不适用于该类。那意味着如果一个类声明了析构，拷贝操作可能不应该自动生成，因为它们做的事情可能是错误的。在C++98提出的时候，上述推理没有得倒足够的重视，所以C++98用户声明析构函数不会左右编译器生成拷贝操作的意愿。**C++11中情况仍然如此，但仅仅是因为限制拷贝操作生成的条件会破坏老代码。**

使用C++的`= default`可以让用户使用编译器生成的函数行为正确的默认函数：

```c++
class Widget {
    public:
    … 
    ~Widget();                              //用户声明的析构函数
    …                                       //默认拷贝构造函数
    Widget(const Widget&) = default;        //的行为还可以

    Widget&                                 //默认拷贝赋值运算符
        operator=(const Widget&) = default; //的行为还可以
    … 
};
```

在多态基类中，通常有一个虚析构函数。除非类继承了一个已经是`virtual`的析构函数，否则要想析构函数为虚函数的唯一方法就是加上`virtual`关键字。在这种情况下通常默认实现是对的，`= default`是一个不错的方式表达默认实现。然而用户声明的析构函数会抑制编译器生成移动操作，所以如果该类需要具有移动性，就为移动操作加上`= default`。声明移动会抑制拷贝生成，所以如果拷贝性也需要支持，再为拷贝操作加上`= default`（前提是编译器生成的默认函数行为是正确的）：

```c++
class Base {
public:
    virtual ~Base() = default;              //使析构函数virtual
    
    Base(Base&&) = default;                 //支持移动
    Base& operator=(Base&&) = default;
    
    Base(const Base&) = default;            //支持拷贝
    Base& operator=(const Base&) = default;
    … 
};
```

实际上，就算编译器乐于为你的类生成拷贝和移动操作，生成的函数也如你所愿，你也应该手动声明它们然后加上`= default`。这看起来比较多余，但是它让你的意图更明确，也能帮助你避免一些微妙的bug。比如，你有一个类来表示字符串表，即一种支持使用整数ID快速查找字符串值的数据结构：

```cpp
class StringTable {
public:
    StringTable() {}
    …                   //插入、删除、查找等函数，但是没有拷贝/移动/析构功能
private:
    std::map<int, std::string> values;
};
```

假设这个类没有声明拷贝操作，没有移动操作，也没有析构，如果它们被用到编译器会自动生成。没错，很方便。

后来需要在对象构造和析构中打日志，增加这种功能很简单：

```cpp
class StringTable {
public:
    StringTable()
    { makeLogEntry("Creating StringTable object"); }    //增加的

    ~StringTable()                                      //也是增加的
    { makeLogEntry("Destroying StringTable object"); }
    …                                               //其他函数同之前一样
private:
    std::map<int, std::string> values;              //同之前一样
};
```

看起来合情合理，但是声明析构有潜在的副作用：它阻止了移动操作的生成。然而，拷贝操作的生成是不受影响的。因此代码能通过编译，运行，也能通过功能（译注：即打日志的功能）测试。功能测试也包括移动功能，因为即使该类不支持移动操作，对该类的移动请求也能通过编译和运行。这个请求正如之前提到的，会转而由拷贝操作完成。它意味着对`StringTable`对象的移动实际上是对对象的拷贝，即拷贝里面的`std::map<int, std::string>`对象。拷贝`std::map<int, std::string>`对象很可能比移动慢几个数量级。简单的加个析构就引入了极大的性能问题！对拷贝和移动操作显式加个`= default`，问题将不再出现。

**注意没有“成员函数模版阻止编译器生成特殊成员函数”的规则：**

```cpp
class Widget {
    …
    template<typename T>                //从任何东西构造Widget
    Widget(const T& rhs);

    template<typename T>                //从任何东西赋值给Widget
    Widget& operator=(const T& rhs);
    …
};
```

编译器仍会生成移动和拷贝操作（假设正常生成它们的条件满足），即使可以模板实例化产出拷贝构造和拷贝赋值运算符的函数签名。



## 条款十八：对于独占资源使用std::unique_ptr

**工厂函数中智能指针的应用：**

智能指针默认情况下，销毁将通过`delete`进行，但在构造过程中，`std::unqiue_ptr`对象可以被设置为使用（对资源的）**自定义删除器**：当资源需要销毁时可调用的任意函数（或者函数对象，包括*lambda*表达式）。如果通过`makeInvestment`创建的对象不应仅仅被`delete`，而应该先写一条日志，`makeInvestment`可以以如下方式实现:

```c++
class Investment {
public:
    …
    virtual ~Investment();          //关键设计部分！
    …
};
class Stock: public Investment { … };
class Bond: public Investment { … };
class RealEstate: public Investment { … };

auto delInvmt = [](Investment* pInvestment)         //自定义删除器
                {                                   //（lambda表达式）
                    makeLogEntry(pInvestment);
                    delete pInvestment; 
                };

template<typename... Ts>
std::unique_ptr<Investment, decltype(delInvmt)>     //更改后的返回类型
makeInvestment(Ts&&... params)
{
    std::unique_ptr<Investment, decltype(delInvmt)> //应返回的指针
        pInv(nullptr, delInvmt);
    if (/*一个Stock对象应被创建*/)
    {
        pInv.reset(new Stock(std::forward<Ts>(params)...));
    }
    else if ( /*一个Bond对象应被创建*/ )   
    {     
        pInv.reset(new Bond(std::forward<Ts>(params)...));   
    }   
    else if ( /*一个RealEstate对象应被创建*/ )   
    {     
        pInv.reset(new RealEstate(std::forward<Ts>(params)...));   
    }   
    return pInv;
}
```

- `delInvmt`是从`makeInvestment`返回的对象的自定义的删除器。所有的自定义的删除行为接受要销毁对象的原始指针，然后执行所有必要行为实现销毁操作。在上面情况中，操作包括调用`makeLogEntry`然后应用`delete`。使用*lambda*创建`delInvmt`是方便的原因如下：当使用默认删除器时（如`delete`），你可以合理假设`std::unique_ptr`对象和原始指针大小相同；当自定义删除器时，情况可能不再如此。函数指针形式的删除器，通常会使`std::unique_ptr`的大小从一个字（*word*）增加到两个。对于函数对象形式的删除器来说，变化的大小取决于函数对象中存储的状态多少，无状态函数（stateless function）对象（比如不捕获变量的*lambda*表达式）对大小没有影响，这意味当自定义删除器可以实现为函数或者*lambda*时，尽量使用*lambda*。
- 当使用自定义删除器时，删除器类型必须作为第二个类型实参传给`std::unique_ptr`。在上面情况中，就是`delInvmt`的类型，这就是为什么`makeInvestment`返回类型是`std::unique_ptr<Investment, decltype(delInvmt)>`。（对于`decltype`，更多信息查看[Item3](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item3.html)）
- `makeInvestment`的基本策略是创建一个空的`std::unique_ptr`，然后指向一个合适类型的对象，然后返回。为了将自定义删除器`delInvmt`与`pInv`关联，我们把`delInvmt`作为`pInv`构造函数的第二个实参。
- 尝试将原始指针（比如`new`创建）赋值给`std::unique_ptr`通不过编译，因为是一种从原始指针到智能指针的隐式转换。这种隐式转换会出问题，所以C++11的智能指针禁止这个行为。这就是通过`reset`来让`pInv`接管通过`new`创建的对象的所有权的原因。
- 使用`new`时，我们使用`std::forward`把传给`makeInvestment`的实参完美转发出去（查看[Item25](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item25.html)）。这使调用者提供的所有信息可用于正在创建的对象的构造函数。
- 自定义删除器的一个形参，类型是`Investment*`，不管在`makeInvestment`内部创建的对象的真实类型（如`Stock`，`Bond`，或`RealEstate`）是什么，它最终在*lambda*表达式中，作为`Investment*`对象被删除。这意味着我们通过基类指针删除派生类实例，为此，基类`Investment`必须有虚析构函数：

**在C++14中，函数返回类型推导的存在（参阅Item3）意味着`makeInvestment`可以以更简单，更封装的方式实现：**

```cpp
template<typename... Ts>
auto makeInvestment(Ts&&... params)                 //C++14
{
    auto delInvmt = [](Investment* pInvestment)     //现在在
                    {                               //makeInvestment里
                        makeLogEntry(pInvestment);
                        delete pInvestment; 
                    };
    ... //同之前一样
}
```

**`std::unique_ptr`有两种形式：**

一种用于单个对象（`std::unique_ptr<T>`），一种用于数组（`std::unique_ptr<T[]>`）。`std::unique_ptr<T[]>`有用的唯一情况是你使用类似C的API返回一个指向堆数组的原始指针，而你想接管这个数组的所有权。

**`std::unique_ptr`是C++11中表示专有所有权的方法，但是其最吸引人的功能之一是它可以轻松高效的转换为`std::shared_ptr`：**

```cpp
std::shared_ptr<Investment> sp =            //将std::unique_ptr
    makeInvestment(arguments);              //转为std::shared_ptr
```

这就是`std::unique_ptr`非常适合用作工厂函数返回类型的原因的关键部分。 工厂函数无法知道调用者是否要对它们返回的对象使用专有所有权语义，或者共享所有权（即`std::shared_ptr`）是否更合适。 通过返回`std::unique_ptr`，工厂为调用者提供了最有效的智能指针，但它们并不妨碍调用者用其更灵活的兄弟替换它。



**请记住：**

- `std::unique_ptr`是轻量级、快速的、只可移动（*move-only*）的管理专有所有权语义资源的智能指针
- `std::unique_ptr`只能通过其构造函数初始化，或者`reset`成员函数重新指定原始指针
- 默认情况，资源销毁通过`delete`实现，但是支持自定义删除器。有状态的删除器和函数指针会增加`std::unique_ptr`对象的大小
- 将`std::unique_ptr`转化为`std::shared_ptr`非常简单

## 条款十九：对于共享资源使用std::shared_ptr

**`std::shared_ptr`为有共享所有权的任意资源提供一种自动垃圾回收的便捷方式：**

一个通过`std::shared_ptr`访问的对象其生命周期由指向它的有共享所有权（*shared ownership*）的指针们来管理。没有特定的`std::shared_ptr`拥有该对象。相反，所有指向它的`std::shared_ptr`都能相互合作确保在它不再使用的那个点进行析构。当最后一个指向某对象的`std::shared_ptr`不再指向那（比如因为`std::shared_ptr`被销毁或者指向另一个不同的对象），`std::shared_ptr`会销毁它所指向的对象。就垃圾回收来说，客户端不需要关心指向对象的生命周期，而对象的析构是确定性的。

**默认资源销毁是通过`delete`，但是也支持自定义删除器。删除器的类型是什么对于`std::shared_ptr`的类型没有影响：**

`std::shared_ptr`使用`delete`作为资源的默认销毁机制，但是它也支持自定义的删除器。这种支持有别于`std::unique_ptr`。对于`std::unique_ptr`来说，删除器类型是智能指针类型的一部分。对于`std::shared_ptr`则不是：

```CPP
auto loggingDel = [](Widget *pw)        //自定义删除器
                  {                     //（和条款18一样）
                      makeLogEntry(pw);
                      delete pw;
                  };

std::unique_ptr<                        //删除器类型是
    Widget, decltype(loggingDel)        //指针类型的一部分
    > upw(new Widget, loggingDel);
std::shared_ptr<Widget>                 //删除器类型不是
    spw(new Widget, loggingDel);        //指针类型的一部分
```

`std::shared_ptr`的设计更为灵活。考虑有两个`std::shared_ptr<Widget>`，每个自带不同的删除器（比如通过*lambda*表达式自定义删除器）：

```CPP
auto customDeleter1 = [](Widget *pw) { … };     //自定义删除器，
auto customDeleter2 = [](Widget *pw) { … };     //每种类型不同
std::shared_ptr<Widget> pw1(new Widget, customDeleter1);
std::shared_ptr<Widget> pw2(new Widget, customDeleter2);
```

因为`pw1`和`pw2`有相同的类型，所以它们都可以放到存放那个类型的对象的容器中：

```CPP
std::vector<std::shared_ptr<Widget>> vpw{ pw1, pw2 };
```

它们也能相互赋值，也可以传入一个形参为`std::shared_ptr<Widget>`的函数。但是自定义删除器类型不同的`std::unique_ptr`就不行，因为`std::unique_ptr`把删除器视作类型的一部分。

另一个不同于`std::unique_ptr`的地方是，指定自定义删除器不会改变`std::shared_ptr`对象的大小。不管删除器是什么，一个`std::shared_ptr`对象都是两个指针大小。自定义删除器可以是函数对象，函数对象可以包含任意多的数据。它意味着函数对象是任意大的。管理删除器的那部分内存不是`std::shared_ptr`对象的一部分。那部分在堆上面，或者`std::shared_ptr`创建者利用`std::shared_ptr`对自定义分配器的支持能力，那部分内存随便在哪都行。

**较之于`std::unique_ptr`，`std::shared_ptr`对象通常大两倍，控制块会产生开销，需要原子性的引用计数修改操作：**

`std::shared_ptr`构造函数只是**“通常”**递增指向对象的引用计数，原因是移动构造函数的存在。从另一个`std::shared_ptr`移动构造新`std::shared_ptr`会将原来的`std::shared_ptr`设置为null，那意味着老的`std::shared_ptr`不再指向资源，同时新的`std::shared_ptr`指向资源。这样的结果就是不需要修改引用计数值。因此移动`std::shared_ptr`会比拷贝它要快：拷贝要求递增引用计数值，移动不需要。移动赋值运算符同理，所以移动构造比拷贝构造快，移动赋值运算符也比拷贝赋值运算符快。

引用计数暗示着性能问题：

- **`std::shared_ptr`大小是原始指针的两倍**，因为它内部包含一个指向资源的原始指针，还包含一个指向资源的引用计数值的原始指针。（这种实现法并不是标准要求的，但是我（指原书作者Scott Meyers）熟悉的所有标准库都这样实现。）
- **引用计数的内存必须动态分配**。 概念上，引用计数与所指对象关联起来，但是实际上被指向的对象不知道这件事情（译注：不知道有一个关联到自己的计数值）。因此它们没有办法存放一个引用计数值。（一个好消息是任何对象——甚至是内置类型的——都可以由`std::shared_ptr`管理。）[Item21](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item21.html)会解释使用`std::make_shared`创建`std::shared_ptr`可以避免引用计数的动态分配，但是还存在一些`std::make_shared`不能使用的场景，这时候引用计数就会动态分配。
- **递增递减引用计数必须是原子性的**，因为多个reader、writer可能在不同的线程。比如，指向某种资源的`std::shared_ptr`可能在一个线程执行析构（于是递减指向的对象的引用计数），在另一个不同的线程，`std::shared_ptr`指向相同的对象，但是执行的却是拷贝操作（因此递增了同一个引用计数）。原子操作通常比非原子操作要慢，所以即使引用计数通常只有一个*word*大小，你也应该假定读写它们是存在开销的。

前面提到了`std::shared_ptr`对象包含了所指对象的引用计数的指针。没错，但是有点误导人。因为引用计数是另一个更大的数据结构的一部分，那个数据结构通常叫做**控制块**（*control block*）。每个`std::shared_ptr`管理的对象都有个相应的控制块。控制块除了包含引用计数值外还有一个自定义删除器的拷贝，当然前提是存在自定义删除器。如果用户还指定了自定义分配器，控制块也会包含一个分配器的拷贝。控制块可能还包含一些额外的数据，正如Item21提到的，一个次级引用计数*weak count*，但是目前我们先忽略它。**可以想象`std::shared_ptr`对象在内存中是这样：**

std::shared_ptr< T >

Ptr to T -> T Object

Ptr to Control Block -> (Reference Count, Weak Count, Other Data(自定义删除器，自定义分配器等))

当指向对象的`std::shared_ptr`一创建，对象的控制块就建立了。

通常，对于一个创建指向对象的`std::shared_ptr`的函数来说不可能知道是否有其他`std::shared_ptr`早已指向那个对象，所以控制块的创建会遵循下面几条规则：

- **`std::make_shared`（参见[Item21](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item21.html)）总是创建一个控制块**。它创建一个要指向的新对象，所以可以肯定`std::make_shared`调用时对象不存在其他控制块。
- **当从独占指针（即`std::unique_ptr`或者`std::auto_ptr`）上构造出`std::shared_ptr`时会创建控制块**。独占指针没有使用控制块，所以指针指向的对象没有关联控制块。（作为构造的一部分，`std::shared_ptr`侵占独占指针所指向的对象的独占权，所以独占指针被设置为null）
- **当从原始指针上构造出`std::shared_ptr`时会创建控制块**。如果你想从一个早已存在控制块的对象上创建`std::shared_ptr`，你将假定传递一个`std::shared_ptr`或者`std::weak_ptr`（参见[Item20](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item20.html)）作为构造函数实参，而不是原始指针。用`std::shared_ptr`或者`std::weak_ptr`作为构造函数实参创建`std::shared_ptr`不会创建新控制块，因为它可以依赖传递来的智能指针指向控制块。

**避免从原始指针变量上创建`std::shared_ptr`：**

上述规则造成的后果就是从原始指针上构造超过一个`std::shared_ptr`就会产生未定义行为，因为指向的对象有多个控制块关联。多个控制块意味着多个引用计数值，多个引用计数值意味着对象将会被销毁多次（每个引用计数一次）。那意味着像下面的代码是有问题的：

```cpp
auto pw = new Widget;                           //pw是原始指针
…
std::shared_ptr<Widget> spw1(pw, loggingDel);   //为*pw创建控制块
…
std::shared_ptr<Widget> spw2(pw, loggingDel);   //为*pw创建第二个控制块
```

第一，避免传给`std::shared_ptr`构造函数原始指针。通常替代方案是使用`std::make_shared`（参见[Item21](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item21.html)），不过上面例子中，我们使用了自定义删除器，用`std::make_shared`就没办法做到。第二，如果你必须传给`std::shared_ptr`构造函数原始指针，直接传`new`出来的结果，不要传指针变量。如果上面代码第一部分这样重写：

```cpp
std::shared_ptr<Widget> spw1(new Widget,    //直接使用new的结果
                             loggingDel);
```

会少了很多从原始指针上构造第二个`std::shared_ptr`的诱惑。相应的，创建`spw2`也会很自然的用`spw1`作为初始化参数（即用`std::shared_ptr`拷贝构造函数），那就没什么问题了：

```CPP
std::shared_ptr<Widget> spw2(spw1);         //spw2使用spw1一样的控制块
```

使用`this`指针作为`std::shared_ptr`构造函数实参的时候可能导致创建多个控制块。假设我们的程序使用`std::shared_ptr`管理`Widget`对象，我们有一个数据结构用于跟踪已经处理过的`Widget`对象：

```cpp
std::vector<std::shared_ptr<Widget>> processedWidgets;
```

继续，假设`Widget`有一个用于处理的成员函数：

```cpp
class Widget {
public:
    …
    void process();
    …
};
```

对于`Widget::process`看起来合理的代码如下：

```cpp
void Widget::process()
{
    …                                       //处理Widget
    processedWidgets.emplace_back(this);    //然后将它加到已处理过的Widget
}                                           //的列表中，这是错的！
```

错误的部分是传递`this`，而不是使用了`emplace_back`。如果你不熟悉`emplace_back`，参见Item42。上面的代码可以通过编译，但是向`std::shared_ptr`的容器传递一个原始指针（`this`），`std::shared_ptr`会由此为指向的`Widget`（`*this`）创建一个控制块，但在成员函数外面可能早已存在指向那个`Widget`对象的指针，对应的控制块可能早就建立了。

如果你想创建一个用`std::shared_ptr`管理的类，这个类能够用`this`指针安全地创建一个`std::shared_ptr`，C++标准库中的`std::enable_shared_from_this`可作为基类的模板类：

```cpp
class Widget: public std::enable_shared_from_this<Widget> {
public:
    …
    void process();
    …
};
```

`std::enable_shared_from_this`是一个基类模板。它的模板参数总是某个继承自它的类，这种设计模式就是奇异递归模板模式（*The Curiously Recurring Template Pattern*（*CRTP*））`std::enable_shared_from_this`定义了一个成员函数，成员函数会创建指向当前对象的`std::shared_ptr`却不创建多余控制块。这个成员函数就是`shared_from_this`，无论在哪当你想在成员函数中使用`std::shared_ptr`指向`this`所指对象时都请使用它：

```cpp
void Widget::process()
{
    //和之前一样，处理Widget
    …
    //把指向当前对象的std::shared_ptr加入processedWidgets
    processedWidgets.emplace_back(shared_from_this());
}
```

从内部来说，`shared_from_this`查找当前对象控制块，然后创建一个新的`std::shared_ptr`关联这个控制块。设计的依据是当前对象已经存在一个关联的控制块。要想符合设计依据的情况，必须已经存在一个指向当前对象的`std::shared_ptr`（比如调用`shared_from_this`的成员函数外面已经存在一个`std::shared_ptr`）。如果没有`std::shared_ptr`指向当前对象（即当前对象没有关联控制块），行为是未定义的，`shared_from_this`通常抛出一个异常。

要想防止客户端在存在一个指向对象的`std::shared_ptr`前先调用含有`shared_from_this`的成员函数，继承自`std::enable_shared_from_this`的类通常将它们的构造函数声明为`private`，并且让客户端通过返回`std::shared_ptr`的工厂函数创建对象。以`Widget`为例，代码可以是这样：

```cpp
class Widget: public std::enable_shared_from_this<Widget> {
public:
    //完美转发参数给private构造函数的工厂函数
    template<typename... Ts>
    static std::shared_ptr<Widget> create(Ts&&... params);
    …
    void process();     //和前面一样
    …
private:
    …                   //构造函数
};
```

**`std::shared_ptr`的成本：**

控制块通常只占几个*word*大小，自定义删除器和分配器可能会让它变大一点。通常控制块的实现比你想的更复杂一些。它使用继承，甚至里面还有一个虚函数（用来确保指向的对象被正确销毁）。这意味着使用`std::shared_ptr`还会招致控制块使用虚函数带来的成本。在通常情况下，使用默认删除器和默认分配器，使用`std::make_shared`创建`std::shared_ptr`，产生的控制块只需三个word大小。它的分配基本上是无开销的。（开销被并入了指向的对象的分配成本里。细节参见[Item21](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item21.html)）。对`std::shared_ptr`解引用的开销不会比原始指针高。执行需要原子引用计数修改的操作需要承担一两个原子操作开销，这些操作通常都会一一映射到机器指令上，所以即使对比非原子指令来说，原子指令开销较大，但是它们仍然只是单个指令上的。对于每个被`std::shared_ptr`指向的对象来说，控制块中的虚函数机制产生的开销通常只需要承受一次，即对象销毁的时候。

**`std::shared_ptr`不能处理的另一个东西是数组：**

和`std::unique_ptr`不同的是，`std::shared_ptr`的API设计之初就是针对单个对象的，没有办法`std::shared_ptr<T[]>`。（译者注: 自 C++17 起 std::shared_ptr 可以用于管理动态分配的数组，使用 std::shared_ptr<T[]>）“聪明”的程序员踌躇于是否该使用`std::shared_ptr<T>`指向数组，然后传入自定义删除器来删除数组（即`delete []`）。这可以通过编译，但一方面，`std::shared_ptr`没有提供`operator[]`，所以数组索引操作需要借助怪异的指针算术。另一方面，`std::shared_ptr`支持转换为指向基类的指针，这对于单个对象来说有效，但是当用于数组类型时相当于在类型系统上开洞。（出于这个原因，`std::unique_ptr<T[]>` API禁止这种转换。）更重要的是，C++11已经提供了很多内置数组的候选方案（比如`std::array`，`std::vector`，`std::string`）。

## 条款二十：当std::shared_ptr可能悬空时使用std::weak_ptr

**`std::weak_ptr`不是一个独立的智能指针，而是`std::shared_ptr`的增强。**`std::weak_ptr`通常从`std::shared_ptr`上创建。当从`std::shared_ptr`上创建`std::weak_ptr`时两者指向相同的对象，但是`std::weak_ptr`参与对象的共享所有权，不会影响所指向对象的引用计数：

```c++
auto spw =                      //spw创建之后，指向的Widget的
    std::make_shared<Widget>(); //引用计数（ref count，RC）为1。
                                //std::make_shared的信息参见条款21
…
std::weak_ptr<Widget> wpw(spw); //wpw指向与spw所指相同的Widget。RC仍为1
…
spw = nullptr;                  //RC变为0，Widget被销毁。
                                //wpw现在悬空
```

悬空的`std::weak_ptr`被称作已经**expired**（过期）：

```CPP
if (wpw.expired()) …            //如果wpw没有指向对象…
```

**通常期望的是检查`std::weak_ptr`是否已经过期，如果没有过期则访问其指向的对象。**`std::weak_ptr`因为缺少解引用操作，没有办法写这样的代码。即使有，将检查和解引用分开会引入竞态条件：在调用`expired`和解引用操作之间，另一个线程可能对指向这对象的`std::shared_ptr`重新赋值或者析构，并由此造成对象已析构。这种情况下，你的解引用将会产生未定义行为。通过一个原子操作检查`std::weak_ptr`是否已经过期，如果没有过期就访问所指对象。这可以通过从`std::weak_ptr`创建`std::shared_ptr`来实现，具体有两种形式可以从`std::weak_ptr`上创建`std::shared_ptr`，具体用哪种取决于`std::weak_ptr`过期时你希望`std::shared_ptr`表现出什么行为。一种形式是`std::weak_ptr::lock`，它返回一个`std::shared_ptr`，如果`std::weak_ptr`过期这个`std::shared_ptr`为空：

```cpp
std::shared_ptr<Widget> spw1 = wpw.lock();  //如果wpw过期，spw1就为空
 											
auto spw2 = wpw.lock();                     //同上，但是使用auto
```

另一种形式是以`std::weak_ptr`为实参构造`std::shared_ptr`。这种情况中，如果`std::weak_ptr`过期，会抛出一个异常：

```cpp
std::shared_ptr<Widget> spw3(wpw);          //如果wpw过期，抛出std::bad_weak_ptr异常
```

**实用例子1：**

```cpp
std::unique_ptr<const Widget> loadWidget(WidgetID id);
```

如果调用`loadWidget`是一个昂贵的操作（比如它操作文件或者数据库I/O）并且重复使用ID很常见，一个合理的优化是再写一个函数除了完成`loadWidget`做的事情之外再缓存它的结果。当每个请求获取的`Widget`阻塞了缓存也会导致本身性能问题，所以另一个合理的优化可以是当`Widget`不再使用的时候销毁它的缓存。

对于可缓存的工厂函数，返回`std::unique_ptr`不是好的选择。调用者应该接收缓存对象的智能指针，调用者也应该确定这些对象的生命周期，但是缓存本身也需要一个指针指向它所缓存的对象。缓存对象的指针需要知道它是否已经悬空，因为当工厂客户端使用完工厂产生的对象后，对象将被销毁，关联的缓存条目会悬空。所以缓存应该使用`std::weak_ptr`，这可以知道是否已经悬空。这意味着工厂函数返回值类型应该是`std::shared_ptr`，因为只有当对象的生命周期由`std::shared_ptr`管理时，`std::weak_ptr`才能检测到悬空。

```c++
std::shared_ptr<const Widget> fastLoadWidget(WidgetID id)
{
    static std::unordered_map<WidgetID,
                              std::weak_ptr<const Widget>> cache;
                                        //译者注：这里std::weak_ptr<const Widget>是高亮
    auto objPtr = cache[id].lock();     //objPtr是去缓存对象的
                                        //std::shared_ptr（或
                                        //当对象不在缓存中时为null）

    if (!objPtr) {                      //如果不在缓存中
        objPtr = loadWidget(id);        //加载它
        cache[id] = objPtr;             //缓存它
    }
    return objPtr;
}
```

**实用例子2：**

观察者设计模式（Observer design pattern）。此模式的主要组件是subjects（状态可能会更改的对象）和observers（状态发生更改时要通知的对象）。在大多数实现中，每个subject都包含一个数据成员，该成员持有指向其observers的指针。这使subjects很容易发布状态更改通知。subjects对控制observers的生命周期（即它们什么时候被销毁）没有兴趣，但是subjects对确保另一件事具有极大的兴趣，那事就是一个observer被销毁时，不再尝试访问它。一个合理的设计是每个subject持有一个`std::weak_ptr`容器指向observers，因此可以在使用前检查是否已经悬空。

**实用例子2：**

考虑一个持有三个对象`A`、`B`、`C`的数据结构，`A`和`C`共享`B`的所有权，因此持有`std::shared_ptr`：

A——std::shared_ptr——> B <——std::shared_ptr——C

假定从B指向A的指针也很有用，应该使用哪种指针：

A<——？—— B 

- **原始指针**。使用这种方法，如果`A`被销毁，但是`C`继续指向`B`，`B`就会有一个指向`A`的悬空指针。而且`B`不知道指针已经悬空，所以`B`可能会继续访问，就会导致未定义行为。
- **`std::shared_ptr`**。这种设计，`A`和`B`都互相持有对方的`std::shared_ptr`，导致的`std::shared_ptr`环状结构（`A`指向`B`，`B`指向`A`）阻止`A`和`B`的销毁。甚至`A`和`B`无法从其他数据结构访问了（比如，`C`不再指向`B`），每个的引用计数都还是1。如果发生了这种情况，`A`和`B`都被泄漏：程序无法访问它们，但是资源并没有被回收。
- **`std::weak_ptr`**。这避免了上述两个问题。如果`A`被销毁，`B`指向它的指针悬空，但是`B`可以检测到这件事。尤其是，尽管`A`和`B`互相指向对方，`B`的指针不会影响`A`的引用计数，因此在没有`std::shared_ptr`指向`A`时不会导致`A`无法被销毁。



**请记住：**

- 用`std::weak_ptr`替代可能会悬空的`std::shared_ptr`。
- `std::weak_ptr`的潜在使用场景包括：缓存、观察者列表、打破`std::shared_ptr`环状结构。

## 条款二十一：优先考虑使用std::make_unique和std::make_shared，而非直接使用new

`std::make_unique`和`std::make_shared`是三个**make函数** 中的两个：接收任意的多参数集合，完美转发到构造函数去动态分配一个对象，然后返回这个指向这个对象的指针。第三个`make`函数是`std::allocate_shared`。它行为和`std::make_shared`一样，只不过第一个参数是用来动态分配内存的*allocator*对象。

**避免重复代码导致编译时间增加：**

```c++
auto upw1(std::make_unique<Widget>());      //使用make函数
std::unique_ptr<Widget> upw2(new Widget);   //不使用make函数
auto spw1(std::make_shared<Widget>());      //使用make函数
std::shared_ptr<Widget> spw2(new Widget);   //不使用make函数
```

**避免异常安全：**

假设有一个函数来计算相关的优先级，

```c++
int computePriority();
```

并且我们在调用`processWidget`时使用了`new`而不是`std::make_shared`：

```c++
processWidget(std::shared_ptr<Widget>(new Widget),  //潜在的资源泄漏！
              computePriority());
```

潜在的资源泄露和编译器将源码转换为目标代码有关。在运行时，一个函数的实参必须先被计算，这个函数再被调用，所以在调用`processWidget`之前，必须执行以下操作，`processWidget`才开始执行：

- 表达式“`new Widget`”必须计算，例如，一个`Widget`对象必须在堆上被创建
- 负责管理`new`出来指针的`std::shared_ptr<Widget>`构造函数必须被执行
- `computePriority`必须运行

编译器不需要按照执行顺序生成代码，可能按照这个执行顺序生成代码：

1. 执行“`new Widget`”
2. 执行`computePriority`
3. 运行`std::shared_ptr`构造函数

如果运行时`computePriority`产生了异常，那么第一步动态分配的`Widget`就会泄漏。因为它永远都不会被第三步的`std::shared_ptr`所管理；

使用`std::make_shared`可以防止这种问题：

```c++
processWidget(std::make_shared<Widget>(),   //没有潜在的资源泄漏
              computePriority());
```

在运行时，`std::make_shared`和`computePriority`其中一个会先被调用。如果是`std::make_shared`先被调用，在`computePriority`调用前，动态分配`Widget`的原始指针会安全的保存在作为返回值的`std::shared_ptr`中。如果`computePriority`产生一个异常，那么`std::shared_ptr`析构函数将确保管理的`Widget`被销毁。如果首先调用`computePriority`并产生一个异常，那么`std::make_shared`将不会被调用，因此也就不需要担心动态分配`Widget`（会泄漏）。

在编写异常安全代码时，使用`std::make_unique`而不是`new`与使用`std::make_shared`（而不是`new`）同样重要。

**效率提升：**

以下对new的直接使用：

```c++
std::shared_ptr<Widget> spw(new Widget);
```

显然，这段代码需要进行内存分配，但它实际上执行了两次。Item19解释了每个`std::shared_ptr`指向一个控制块，其中包含被指向对象的引用计数，还有其他东西。这个控制块的内存在`std::shared_ptr`构造函数中分配。因此，直接使用`new`需要为`Widget`进行一次内存分配，为控制块再进行一次内存分配。

如果使用`std::make_shared`代替：

```c++
auto spw = std::make_shared<Widget>();
```

一次分配足矣。这是因为`std::make_shared`分配一块内存，同时容纳了`Widget`对象和控制块。这种优化减少了程序的静态大小，因为代码只包含一个内存分配调用，并且它提高了可执行代码的速度，因为内存只分配一次。此外，使用`std::make_shared`避免了对控制块中的某些簿记信息的需要，潜在地减少了程序的总内存占用。

对于`std::make_shared`的效率分析同样适用于`std::allocate_shared`，因此`std::make_shared`的性能优势也扩展到了该函数。

**不允许自定义删除器：**

`std::unique_ptr`和`std::shared_ptr`的构造函数可以接收一个删除器参数。有个`Widget`的自定义删除器：

```cpp
auto widgetDeleter = [](Widget* pw) { … };
```

创建一个使用它的智能指针只能直接使用`new`：

```cpp
std::unique_ptr<Widget, decltype(widgetDeleter)>
    upw(new Widget, widgetDeleter);

std::shared_ptr<Widget> spw(new Widget, widgetDeleter);
```

对于`make`函数，没有办法做同样的事情。

**花括号不能完美转发：**

```c++
auto upv = std::make_unique<std::vector<int>>(10, 20);
auto spv = std::make_shared<std::vector<int>>(10, 20);
```

两种调用都创建了10个元素，每个值为20的`std::vector`。这意味着在`make`函数中，完美转发使用小括号，而不是花括号。坏消息是如果你想用花括号初始化指向的对象，你必须直接使用`new`。使用`make`函数会需要能够完美转发花括号初始化的能力，但是，正如[Item30](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item30.html)所说，花括号初始化无法完美转发。但是，Item30介绍了一个变通的方法：使用`auto`类型推导从花括号初始化创建`std::initializer_list`对象（见[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)），然后将`auto`创建的对象传递给`make`函数。

```cpp
//创建std::initializer_list
auto initList = { 10, 20 };
//使用std::initializer_list为形参的构造函数创建std::vector
auto spv = std::make_shared<std::vector<int>>(initList);
```

对于`std::unique_ptr`，只有这两种情景（自定义删除器和花括号初始化）使用`make`函数有点问题。对于`std::shared_ptr`和它的`make`函数，还有2个问题。

**不用`make`函数创建重载了`operator new`和`operator delete`类的对象指针：**

一些类重载了`operator new`和`operator delete`。这些函数的存在意味着对这些类型的对象的全局内存分配和释放是不合常规的。设计这种定制操作往往只会精确的分配、释放对象大小的内存。例如，`Widget`类的`operator new`和`operator delete`只会处理`sizeof(Widget)`大小的内存块的分配和释放。这种系列行为不太适用于`std::shared_ptr`对自定义分配（通过`std::allocate_shared`）和释放（通过自定义删除器）的支持，因为`std::allocate_shared`需要的内存总大小不等于动态分配的对象大小，还需要**再加上**控制块大小。

**对象的内存释放可能被延迟：**

与直接使用`new`相比，`std::make_shared`在大小和速度上的优势源于`std::shared_ptr`的控制块与指向的对象放在同一块内存中。当对象的引用计数降为0，对象被销毁（即析构函数被调用）。但是，因为控制块和对象被放在同一块分配的内存块中，直到控制块的内存也被销毁，对象占用的内存才被释放。

控制块除了引用计数，还包含簿记信息。引用计数追踪有多少`std::shared_ptr`指向控制块，但控制块还有第二个计数，记录多少个`std::weak_ptr`s指向控制块。第二个引用计数就是*weak count*。（实际上，*weak count*的值不总是等于指向控制块的`std::weak_ptr`的数目，因为库的实现者找到一些方法在*weak count*中添加附加信息，促进更好的代码产生。为了本条款的目的，我们会忽略这一点，假定*weak count*的值等于指向控制块的`std::weak_ptr`的数目。）当一个`std::weak_ptr`检测它是否过期时（见Item19），它会检测指向的控制块中的引用计数（而不是*weak count*）。如果引用计数是0（即对象没有`std::shared_ptr`再指向它，已经被销毁了），`std::weak_ptr`就已经过期。否则就没过期。

只要`std::weak_ptr`引用一个控制块（即*weak count*大于零），该控制块必须继续存在。只要控制块存在，包含它的内存就必须保持分配。通过`std::shared_ptr`的`make`函数分配的内存，直到最后一个`std::shared_ptr`和最后一个指向它的`std::weak_ptr`已被销毁，才会释放。

如果对象类型非常大，而且销毁最后一个`std::shared_ptr`和销毁最后一个`std::weak_ptr`之间的时间很长，那么在销毁对象和释放它所占用的内存之间可能会出现延迟。

但如果直接只用`new`，一旦最后一个`std::shared_ptr`被销毁，`ReallyBigType`对象的内存就会被释放。

**避免异常安全的其他代替方法：**

如果发现自己处于不可能或不合适使用`std::make_shared`的情况下，想要保证自己不受之前看到的异常安全问题的影响。最好的方法是确保在直接使用`new`时，在一个不做其他事情的语句中，立即将结果传递到智能指针构造函数。这可以防止编译器生成的代码在使用`new`和调用管理`new`出来对象的智能指针的构造函数之间发生异常。

考虑前面讨论过的`processWidget`函数，对其指定一个自定义删除器:

```c++
void processWidget(std::shared_ptr<Widget> spw,     //和之前一样
                   int priority);
void cusDel(Widget *ptr);                           //自定义删除器
```

非异常安全调用:

```c++
processWidget( 									    //和之前一样，
    std::shared_ptr<Widget>(new Widget, cusDel),    //潜在的内存泄漏！
    computePriority() 
```

这里使用自定义删除排除了对`std::make_shared`的使用，因此避免出现问题的方法是将`Widget`的分配和`std::shared_ptr`的构造放入它们自己的语句中，然后使用得到的`std::shared_ptr`调用`processWidget`：

```c++
std::shared_ptr<Widget> spw(new Widget, cusDel);
processWidget(spw, computePriority());  // 正确，但是没优化，见下
```

如果`spw`的构造函数抛出异常（比如无法为控制块动态分配内存），仍然能够保证`cusDel`会在“`new Widget`”产生的指针上调用。

在非异常安全调用中，我们将一个右值传递给`processWidget`：

```c++
processWidget(
    std::shared_ptr<Widget>(new Widget, cusDel),    //实参是一个右值
    computePriority()
);
```

但是在异常安全调用中，我们传递了左值：

```c++
processWidget(spw, computePriority());              //实参是左值
```

因为`processWidget`的`std::shared_ptr`形参是传值，从右值构造只需要移动，而传递左值构造需要拷贝。对`std::shared_ptr`而言，拷贝`std::shared_ptr`需要对引用计数原子递增，移动则不需要对引用计数有操作。为了使异常安全代码达到非异常安全代码的性能水平，我们需要用`std::move`将`spw`转换为右值（见Item23）：

```c++
processWidget(std::move(spw), computePriority());   //高效且异常安全
```



## 条款二十二：当使用Pimpl惯用法，请在实现文件中定义特殊成员函数

```c++
class Widget() {                    //定义在头文件“widget.h”
public:
    Widget();
    …
private:
    std::string name;
    std::vector<double> data;
    Gadget g1, g2, g3;              //Gadget是用户自定义的类型
};
```

**Pimpl（*pointer to implementation*）惯用法：**

c++98：

```c++
class Widget                        //仍然在“widget.h”中
{
public:
    Widget();
    ~Widget();                      //析构函数在后面会分析
    …

private:
    struct Impl;                    //声明一个 实现结构体
    Impl *pImpl;                    //以及指向它的指针
};
```

```c++
#include "widget.h"             //以下代码均在实现文件“widget.cpp”里
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl {           //含有之前在Widget中的数据成员的
    std::string name;           //Widget::Impl类型的定义
    std::vector<double> data;
    Gadget g1,g2,g3;
};

Widget::Widget()                //为此Widget对象分配数据成员
: pImpl(new Impl)
{}

Widget::~Widget()               //销毁数据成员
{ delete pImpl; }

```

类`Widget`不再提到类型`std::string`，`std::vector`以及`Gadget`，`Widget`的使用者不再需要为了这些类型而引入头文件。 这可以加速编译，并且意味着，如果这些头文件中有所变动，`Widget`的使用者不会受到影响。一个已经被声明，却还未被实现的类型，被称为**不完整类型**（*incomplete type*）。 `Widget::Impl`就是这种类型。 能对一个不完整类型做的事很少，但是声明一个指向它的指针是可以的。

C++98的代码使用了原始指针，原始的`new`和原始的`delete`。

**C++11用`std::unqiue_ptr`代替原始指针：**

```cpp
class Widget {                      //在“widget.h”中
public:
    Widget();
    …

private:
    struct Impl;
    std::unique_ptr<Impl> pImpl;    //使用智能指针而不是原始指针
};

```

```cpp
#include "widget.h"                 //在“widget.cpp”中
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl {               //跟之前一样
    std::string name;
    std::vector<double> data;
    Gadget g1,g2,g3;
};

Widget::Widget()                    //根据条款21，通过std::make_unique
: pImpl(std::make_unique<Impl>())   //来创建std::unique_ptr
{}

```

智能指针的众多优点之一：它使我们从手动资源释放中解放出来。

**Pimpl惯用法中不完整类型与智能指针的冲突：**

以上的代码能编译，但是，最普通的`Widget`用法却会导致编译出错：

```cpp
#include "widget.h"

Widget w;                           //错误！
```

你所看到的错误信息根据编译器不同会有所不同，但是其文本一般会提到一些有关于“把`sizeof`或`delete`应用到不完整类型上”的信息。对于不完整类型，使用以上操作是禁止的。在Pimpl惯用法中使用`std::unique_ptr`会抛出错误，有点惊悚，因为第一`std::unique_ptr`宣称它支持不完整类型，第二Pimpl惯用法是`std::unique_ptr`的最常见的使用情况之一。对该问题的清楚认识：在对象`w`被析构时（例如离开了作用域），在这个时候，它的析构函数被调用。我们在类的定义里使用了`std::unique_ptr`，所以我们没有声明一个析构函数，因为我们并没有任何代码需要写在里面。根据编译器自动生成的特殊成员函数的规则（见 [Item17](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item17.html)），编译器会自动为我们生成一个析构函数。 在这个析构函数里，编译器会插入一些代码来调用类`Widget`的数据成员`pImpl`的析构函数。 `pImpl`是一个`std::unique_ptr<Widget::Impl>`，也就是说，一个使用默认删除器的`std::unique_ptr`。 默认删除器是一个函数，它使用`delete`来销毁内置于`std::unique_ptr`的原始指针。然而，在使用`delete`之前，通常会使默认删除器使用C++11的特性`static_assert`来确保原始指针指向的类型不是一个不完整类型。 当编译器为`Widget w`的析构生成代码时，它会遇到`static_assert`检查并且失败，这通常是错误信息的来源。 这些错误信息只在对象`w`销毁的地方出现，因为类`Widget`的析构函数，正如其他的编译器生成的特殊成员函数一样，是暗含`inline`属性的。 错误信息自身往往指向对象`w`被创建的那行，因为这行代码明确地构造了这个对象，导致了后面潜在的析构。为了解决这个问题，你只需要确保在编译器生成销毁`std::unique_ptr<Widget::Impl>`的代码之前， `Widget::Impl`已经是一个完整类型（*complete type*）。 当编译器“看到”它的定义的时候，该类型就成为完整类型了。 但是 `Widget::Impl`的定义在`widget.cpp`里。成功编译的关键，就是在`widget.cpp`文件内，让编译器在“看到” `Widget`的析构函数实现之前（也即编译器插入的，用来销毁`std::unique_ptr`这个数据成员的代码的，那个位置），先定义`Widget::Impl`。

只需要先在`widget.h`里，只声明类`Widget`的析构函数，但不要在这里定义它：

```cpp
class Widget {                  //跟之前一样，在“widget.h”中
public:
    Widget();
    ~Widget();                  //只有声明语句
    …

private:                        //跟之前一样
    struct Impl;
    std::unique_ptr<Impl> pImpl;
};
```

在`widget.cpp`文件中，在结构体`Widget::Impl`被定义之后，再定义析构函数：

```cpp
#include "widget.h"                 //跟之前一样，在“widget.cpp”中
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl {               //跟之前一样，定义Widget::Impl
    std::string name;
    std::vector<double> data;
    Gadget g1,g2,g3;
}

Widget::Widget()                    //跟之前一样
: pImpl(std::make_unique<Impl>())
{}

Widget::~Widget() = default;        //析构函数的定义
```

移动操作也一样，编译器生成的移动赋值操作符，在重新赋值之前，需要先销毁指针`pImpl`指向的对象。然而在`Widget`的头文件里，`pImpl`指针指向的是一个不完整类型。移动构造函数的情况有所不同。 移动构造函数的问题是编译器自动生成的代码里，包含有抛出异常的事件，在这个事件里会生成销毁`pImpl`的代码。然而，销毁`pImpl`需要`Impl`是一个完整类型。

解决方案也一样——把移动操作的定义移动到实现文件里：

```cpp
class Widget {                          //仍然在“widget.h”中
public:
    Widget();
    ~Widget();

    Widget(Widget&& rhs);               //只有声明
    Widget& operator=(Widget&& rhs);
    …

private:                                //跟之前一样
    struct Impl;
    std::unique_ptr<Impl> pImpl;
};
#include <string>                   //跟之前一样，仍然在“widget.cpp”中
…
    
struct Widget::Impl { … };          //跟之前一样

Widget::Widget()                    //跟之前一样
: pImpl(std::make_unique<Impl>())
{}

Widget::~Widget() = default;        //跟之前一样

Widget::Widget(Widget&& rhs) = default;             //这里定义
Widget& Widget::operator=(Widget&& rhs) = default;
```

Pimpl惯用法是用来减少类的实现和类使用者之间的编译依赖的一种方法，但是，从概念而言，使用这种惯用法并不改变这个类的表现。 原来的类`Widget`包含有`std::string`，`std::vector`和`Gadget`数据成员，并且，假设类型`Gadget`，如同`std::string`和`std::vector`一样，允许复制操作，所以类`Widget`支持复制操作也很合理。 我们必须要自己来写这些函数，因为第一，对包含有只可移动（*move-only*）类型，如`std::unique_ptr`的类，编译器不会生成复制操作；第二，即使编译器帮我们生成了，生成的复制操作也只会复制`std::unique_ptr`（也即浅拷贝（*shallow copy*）），而实际上我们需要复制指针所指向的对象（也即深拷贝（*deep copy*））：

```cpp
class Widget {                          //仍然在“widget.h”中
public:
    …

    Widget(const Widget& rhs);          //只有声明
    Widget& operator=(const Widget& rhs);

private:                                //跟之前一样
    struct Impl;
    std::unique_ptr<Impl> pImpl;
};
#include <string>                   //跟之前一样，仍然在“widget.cpp”中
…
    
struct Widget::Impl { … };          //跟之前一样

Widget::~Widget() = default;		//其他函数，跟之前一样

Widget::Widget(const Widget& rhs)   //拷贝构造函数
: pImpl(std::make_unique<Impl>(*rhs.pImpl))
{}

Widget& Widget::operator=(const Widget& rhs)    //拷贝operator=
{
    *pImpl = *rhs.pImpl;
    return *this;
}
```

在每个情况中，我们都只从源对象（`rhs`）中，复制了结构体`Impl`的内容到目标对象中（`*this`）。我们利用了编译器会为我们自动生成结构体`Impl`的复制操作函数的机制，而不是逐一复制结构体`Impl`的成员，自动生成的复制操作能自动复制每一个成员。 因此我们通过调用编译器生成的`Widget::Impl`的复制操作函数来实现了类`Widget`的复制操作。 在复制构造函数中，注意，我们仍然遵从了Item21的建议，使用`std::make_unique`而非直接使用`new`。

**若用`std::shared_ptr`代替原始指针：**

如果使用`std::shared_ptr`而不是`std::unique_ptr`来做`pImpl`指针， 会发现本条款的建议不再适用。 因为不需要在类`Widget`里声明析构函数，没有了用户定义析构函数，编译器将会愉快地生成移动操作，并且将会如我们所期望般工作。`widget.h`里的代码如下，

```cpp
class Widget {                      //在“widget.h”中
public:
    Widget();
    …                               //没有析构函数和移动操作的声明

private:
    struct Impl;
    std::shared_ptr<Impl> pImpl;    //用std::shared_ptr
};                                  //而不是std::unique_ptr
```

这是`#include`了`widget.h`的客户代码，

```cpp
Widget w1;
auto w2(std::move(w1));     //移动构造w2
w1 = std::move(w2);         //移动赋值w1
```

这些都能编译，并且工作地如我们所望：`w1`将会被默认构造，它的值会被移动进`w2`，随后值将会被移动回`w1`，然后两者都会被销毁（因此导致指向的`Widget::Impl`对象一并也被销毁）。

`std::unique_ptr`和`std::shared_ptr`在`pImpl`指针上的表现上的区别的深层原因在于，他们支持自定义删除器的方式不同。 对`std::unique_ptr`而言，删除器的类型是这个智能指针的一部分，这让编译器有可能生成更小的运行时数据结构和更快的运行代码。 这种更高效率的后果之一就是`std::unique_ptr`指向的类型，在编译器的生成特殊成员函数（如析构函数，移动操作）被调用时，必须已经是一个完整类型。 而对`std::shared_ptr`而言，删除器的类型不是该智能指针的一部分，这让它会生成更大的运行时数据结构和稍微慢点的代码，但是当编译器生成的特殊成员函数被使用的时候，指向的对象不必是一个完整类型。（译者注：知道`std::unique_ptr`和`std::shared_ptr`的实现，这一段才比较容易理解。）



**请记住：**

- Pimpl惯用法通过减少在类实现和类使用者之间的编译依赖来减少编译时间。
- 对于`std::unique_ptr`类型的`pImpl`指针，需要在头文件的类里声明特殊的成员函数，但是在实现文件里面来实现他们。即使是编译器自动生成的代码可以工作，也要这么做。
- 以上的建议只适用于`std::unique_ptr`，不适用于`std::shared_ptr`。





## 条款二十三：理解std::move和std::forward

**`std::move`无条件的将它的实参转换为右值:**

**C++11的`std::move`的示例实现：**

```c++
template<typename T>
typename remove_reference<T>::type&&
move(T&& param)
{
	using ReturnType = typename remove_reference<T>::type&&;
	return static_cast<ReturnType>(param);
}
```

`std::move`接受一个对象的引用（准确的说，一个通用引用，见Item24)，返回一个指向同对象的引用。该函数返回类型的`&&`部分表明`std::move`函数返回的是一个右值引用，但是，如果类型`T`恰好是一个左值引用，那么`T&&`将会成为一个左值引用。为了避免如此，*type trait*（见[Item9](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item9.html)）`std::remove_reference`应用到了类型`T`上，因此确保了`&&`被正确的应用到了一个不是引用的类型上。这保证了`std::move`返回的是右值引用。

C++14中的函数返回值类型推导（见Item3）和标准库的模板别名`std::remove_reference_t`(见Item9)，可以使`std::move`这样实现：

```c++
template<typename T>
decltype(auto) move(T&& param)          //C++14，仍然在std命名空间
{
    using ReturnType = remove_referece_t<T>&&;
    return static_cast<ReturnType>(param);
}
```

**移动语义和`const`的矛盾：**

```c++
class Annotation {
public:
    explicit Annotation(const std::string text)
    ：value(std::move(text))    //“移动”text到value里；这段代码执行起来
    { … }                       //并不是看起来那样
    
    …

private:
    std::string value;
};
```

这段代码可以编译，可以链接，可以运行。这段代码将数据成员`value`设置为`text`的值。但 `text`并不是被移动到`value`，而是被**拷贝**。诚然，`text`通过`std::move`被转换到右值，但是`text`被声明为`const std::string`，所以在转换之前，`text`是一个左值的`const std::string`，而转换的结果是一个右值的`const std::string`，当编译器决定哪一个`std::string`的构造函数被调用时，考虑它的作用，将会有两种可能性：

```cpp
class string {                  //std::string事实上是
public:                         //std::basic_string<char>的类型别名
    …
    string(const string& rhs);  //拷贝构造函数
    string(string&& rhs);       //移动构造函数
    …
};
```

在类`Annotation`的构造函数的成员初始化列表中，`std::move(text)`的结果是一个`const std::string`的右值。这个右值不能被传递给`std::string`的移动构造函数，因为移动构造函数只接受一个指向*non-`const`的`std::string`的右值引用。然而，该右值却可以被传递给`std::string`的拷贝构造函数，因为lvalue-reference-to-`const`允许被绑定到一个`const`右值上。因此，`std::string`在成员初始化的过程中调用了**拷贝**构造函数，即使`text`已经被转换成了右值。这样是为了确保维持`const`属性的正确性。从一个对象中移动出某个值通常代表着修改该对象，所以语言不允许`const`对象被传递给可以修改他们的函数。

**`std::forward`是有条件的转换：**

最常见的情景是一个模板函数，接收一个通用引用形参，并将它传递给另外的函数：

```cpp
void process(const Widget& lvalArg);        //处理左值
void process(Widget&& rvalArg);             //处理右值

template<typename T>                        //用以转发param到process的模板
void logAndProcess(T&& param)
{
    auto now =                              //获取现在时间
        std::chrono::system_clock::now();
    
    makeLogEntry("Calling 'process'", now);
    process(std::forward<T>(param));
}
```

考虑两次对`logAndProcess`的调用，一次左值为实参，一次右值为实参：

```cpp
Widget w;

logAndProcess(w);               //用左值调用
logAndProcess(std::move(w));    //用右值调用
```

但是`param`，正如所有的其他函数形参一样，是一个左值。每次在函数`logAndProcess`内部对函数`process`的调用，都会因此调用函数`process`的左值重载版本。为防如此，我们需要一种机制：当且仅当传递给函数`logAndProcess`的用以初始化`param`的实参是一个右值时，`param`会被转换为一个右值。这就是`std::forward`做的事情。这就是为什么`std::forward`是一个**有条件**的转换：它的实参用右值初始化时，转换为一个右值。`std::forward`是怎么分辨`param`是被一个左值还是右值初始化的？ 简短的说，该信息藏在函数`logAndProcess`的模板参数`T`中。该参数被传递给了函数`std::forward`，它解开了含在其中的信息。该机制工作的细节可以查询Item28.

可以免于使用`std::move`而在任何地方只使用`std::forward`。 从纯技术的角度，`std::forward`是可以完全胜任，`std::move`并非必须。假设在下面这个类中，唯一一个非静态的数据成员是`std::string`，一种经典的移动构造函数（即，使用`std::move`）可以被实现如下：

```cpp
class Widget {
public:
    Widget(Widget&& rhs)
    : s(std::move(rhs.s))
    { ++moveCtorCalls; }

    …

private:
    static std::size_t moveCtorCalls;
    std::string s;
};
```

如果要用`std::forward`来达成同样的效果：

```cpp
class Widget{
public:
    Widget(Widget&& rhs)                    //不自然，不合理的实现
    : s(std::forward<std::string>(rhs.s))
    { ++moveCtorCalls; }

    …

}
```

第一，`std::move`只需要一个函数实参（`rhs.s`），而`std::forward`不但需要一个函数实参（`rhs.s`），还需要一个模板类型实参`std::string`。其次，我们传递给`std::forward`的类型应当是一个non-reference，因为惯例是传递的实参应该是一个右值（见[Item28](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item28.html)）。同样，这意味着`std::move`比起`std::forward`来说需要打更少的字，并且免去了传递一个表示我们正在传递一个右值的类型实参。同样，它根绝了我们传递错误类型的可能性（例如，`std::string&`可能导致数据成员`s`被复制而不是被移动构造）。

**请记住：**

- `std::move`执行到右值的无条件的转换，但就自身而言，它不移动任何东西。

- `std::forward`只有当它的参数被绑定到一个右值时，才将参数转换为右值。

- `std::move`和`std::forward`在运行期什么也不做。它们不产生任何可执行代码，一字节也没有。

  

## 条款二十四：区分通用引用和右值引用

**“`T&&`”有两种不同的意思：**

第一种，是右值引用：它们只绑定到右值上，并且它们主要的存在原因就是为了识别可以移动操作的对象。

另一种意思是，它既可以是右值引用，也可以是左值引用。它们既可以绑定到右值上（就像右值引用），也可以绑定到左值上（就像左值引用）。 此外，它们还可以绑定到`const`或者non-`const`的对象上，也可以绑定到`volatile`或者non-`volatile`的对象上，甚至可以绑定到既`const`又`volatile`的对象上。

**在两种情况下会出现通用引用：**

函数模板形参：

```cpp
template<typename T>
void f(T&& param);                  //param是一个通用引用
```

第二种情况是`auto`声明符：

```cpp
auto&& var2 = var1;                 //var2是一个通用引用
```

这两种情况的共同之处就是都存在**类型推导**（*type deduction*）。如果初始值是一个右值，那么通用引用就会是对应的右值引用，如果初始值是一个左值，那么通用引用就会是一个左值引用。

对一个通用引用而言，类型推导是必要的，并且引用声明的**形式**必须正确，它必须恰好为“`T&&`”，例如：

```cpp
template <typename T>
void f(std::vector<T>&& param);     //param是一个右值引用
```

当函数`f`被调用的时候，类型`T`会被推导（除非调用者显式地指定它，这种边缘情况我们不考虑）。但是`param`的类型声明并不是`T&&`，而是一个`std::vector<T>&&`。这排除了`param`是一个通用引用的可能性。`param`因此是一个右值引用。

`const`修饰符的出现，也不能成为通用引用：

```cpp
template <typename T>
void f(const T&& param);        //param是一个右值引用
```

在模板内部并不保证一定会发生类型推导：

```cpp
template<class T, class Allocator = allocator<T>>   //来自C++标准
class vector
{
public:
    void push_back(T&& x);
    …
}
```

`push_back`在有一个特定的`vector`实例之前不可能存在，而实例化`vector`时的类型已经决定了`push_back`的声明。也就是说，

```cpp
std::vector<Widget> v;
```

将会导致`std::vector`模板被实例化为以下代码：

```cpp
class vector<Widget, allocator<Widget>> {
public:
    void push_back(Widget&& x);             //右值引用
    …
};
```

`std::vector`内的概念上相似的成员函数`emplace_back`，却确实包含类型推导:

```cpp
template<class T, class Allocator = allocator<T>>   //依旧来自C++标准
class vector {
public:
    template <class... Args>
    void emplace_back(Args&&... args);
    …
};
```

这儿，类型参数（*type parameter*）`Args`是独立于`vector`的类型参数`T`的，所以`Args`会在每次`emplace_back`被调用的时候被推导。

**右值引用的情况：**

```cpp
void f(Widget&& param);         //没有类型推导，
                                //param是一个右值引用
Widget&& var1 = Widget();       //没有类型推导，
                                //var1是一个右值引用
```



**请记住：**

- 如果一个函数模板形参的类型为`T&&`，并且`T`需要被推导得知，或者如果一个对象被声明为`auto&&`，这个形参或者对象就是一个通用引用。
- 如果类型声明的形式不是标准的`type&&`，或者如果类型推导没有发生，那么`type&&`代表一个右值引用。
- 通用引用，如果它被右值初始化，就会对应地成为右值引用；如果它被左值初始化，就会成为左值引用。



## 条款二十五：对右值引用使用std::move，对通用引用使用std::forward

**对右值引用使用std::move：**

```cpp
class Widget {
public:
    Widget(Widget&& rhs)        //rhs是右值引用
    : name(std::move(rhs.name)),
      p(std::move(rhs.p))
      { … }
    …
private:
    std::string name;
    std::shared_ptr<SomeDataStructure> p;
};
```

**对通用引用使用std::forward：**

通用引用使用右值初始化时，才将其强制转换为右值

```cpp
class Widget {
public:
    template<typename T>
    void setName(T&& newName)           //newName是通用引用
    { name = std::forward<T>(newName); }

    …
};
```

**避免在右值引用上使用`std::forward`：**

Item23解释说，可以在右值引用上使用`std::forward`表现出适当的行为，但是代码较长，容易出错，所以应该避免在右值引用上使用`std::forward`。

**避免在通用引用上使用`std::move`：**

```cpp
class Widget {
public:
    template<typename T>
    void setName(T&& newName)       //通用引用可以编译，
    { name = std::move(newName); }  //但是代码太太太差了！
    …

private:
    std::string name;
    std::shared_ptr<SomeDataStructure> p;
};

std::string getWidgetName();        //工厂函数

Widget w;

auto n = getWidgetName();           //n是局部变量

w.setName(n);                       //把n移动进w！

…                                   //现在n的值未知
```

上面的例子，局部变量`n`被传递给`w.setName`，调用方可能认为这是对`n`的只读操作——这一点倒是可以被原谅。但是因为`setName`内部使用`std::move`无条件将传递的引用形参转换为右值，`n`的值被移动进`w.name`，调用`setName`返回时`n`最终变为未定义的值。

可能会有这个想法：`setName`不应该将其形参声明为通用引用，此类引用不能使用`const`（见[Item24](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item24.html)），但是`setName`肯定不应该修改其形参。因此如果为`const`左值和为右值分别重载`setName`可以避免整个问题，比如这样：

```cpp
class Widget {
public:
    void setName(const std::string& newName)    //用const左值设置
    { name = newName; }
    
    void setName(std::string&& newName)         //用右值设置
    { name = std::move(newName); }
    
    …
};
```

这样的话，当然可以工作，但是有缺点。首先编写和维护的代码更多（两个函数而不是单个模板）；其次，效率下降。比如，考虑如下场景：

```cpp
w.setName("Adela Novak");
```

使用通用引用的版本的`setName`，字面字符串“`Adela Novak`”可以被传递给`setName`，再传给`w`内部`std::string`的赋值运算符。`w`的`name`的数据成员通过字面字符串直接赋值，没有临时`std::string`对象被创建。但是，`setName`重载版本，会有一个临时`std::string`对象被创建，`setName`形参绑定到这个对象，然后这个临时`std::string`移动到`w`的数据成员中。一次`setName`的调用会包括`std::string`构造函数调用（创建中间对象），`std::string`赋值运算符调用（移动`newName`到`w.name`），`std::string`析构函数调用（析构中间对象）。这比调用接受`const char*`指针的`std::string`赋值运算符开销昂贵许多。并且设计的可扩展性差。`Widget::setName`有一个形参，因此需要两种重载实现，但是对于有更多形参的函数，每个都可能是左值或右值，重载函数的数量几何式增长：n个参数的话，就要实现2的n次种重载。对于这种函数，对于左值和右值分别重载就不能考虑了：通用引用是仅有的实现方案。

**在最后一次使用时，使用`std::move`（对右值引用）或者`std::forward`（对通用引用）：**

```cpp
template<typename T>
void setSignText(T&& text)                  //text是通用引用
{
  sign.setText(text);                       //使用text但是不改变它
  
  auto now = 
      std::chrono::system_clock::now();     //获取现在的时间
  
  signHistory.add(now, 
                  std::forward<T>(text));   //有条件的转换为右值
}
```

这里，我们想要确保`text`的值不会被`sign.setText`改变，因为我们想要在`signHistory.add`中继续使用。因此`std::forward`只在最后使用。对于`std::move`，同样的思路（即最后一次用右值引用的时候再调用`std::move`），但是需要注意，在有些稀少的情况下，你需要调用`std::move_if_noexcept`代替`std::move`，参考Item14。

**在按值返回的函数中，返回值绑定到右值引用或者通用引用上，需要对返回的引用使用`std::move`或者`std::forward`：**

```cpp
Matrix                              //按值返回
operator+(Matrix&& lhs, const Matrix& rhs)
{
    lhs += rhs;
    return std::move(lhs);	        //移动lhs到返回值中
}
```

通过在`return`语句中将`lhs`转换为右值（通过`std::move`），`lhs`可以移动到返回值的内存位置。如果省略了`std::move`调用，

```cpp
Matrix                              //同之前一样
operator+(Matrix&& lhs, const Matrix& rhs)
{
    lhs += rhs;
    return lhs;                     //拷贝lhs到返回值中
}
```

`lhs`是个左值的事实，会强制编译器拷贝它到返回值的内存空间。假定`Matrix`支持移动操作，并且比拷贝操作效率更高，在`return`语句中使用`std::move`的代码效率更高。

使用通用引用和`std::forward`的情况类似。

**误用：对要返回的局部对象应用上一点同样的优化：**

```cpp
Widget makeWidget()                 //makeWidget的“拷贝”版本
{
    Widget w;                       //局部对象
    …                               //配置w
    return w;                       //“拷贝”w到返回值中
}
```

他们想要“优化”代码，把“拷贝”变为移动：

```cpp
Widget makeWidget()                 //makeWidget的移动版本
{
    Widget w;
    …
    return std::move(w);            //移动w到返回值中（不要这样做！）
}
```

C++标准中**返回值优化**已经实现，，`makeWidget`的“拷贝”版本可以避免复制局部变量`w`的需要，通过在分配给函数返回值的内存中来构造`w`（拷贝消除）。就是说，编译器可能会在按值返回的函数中消除对局部对象的拷贝（或者移动），如果满足（1）局部对象与函数返回值的类型相同；（2）局部对象就是要返回的东西。（适合的局部对象包括大多数局部变量（比如`makeWidget`里的`w`），还有作为`return`语句的一部分而创建的临时对象。函数形参不满足要求。

再看`makeWidget`的“拷贝”版本：

```cpp
Widget makeWidget()                 //makeWidget的“拷贝”版本
{
    Widget w;
    …
    return w;                       //“拷贝”w到返回值中
}
```

这里两个条件都满足，C++编译器都会应用RVO来避免拷贝`w`。那意味着`makeWidget`的“拷贝”版本实际上不拷贝任何东西。

移动版本的`makeWidget`行为与其名称一样（假设`Widget`有移动构造函数），将`w`的内容移动到`makeWidget`的返回值位置。但是为编译器不使用RVO消除这种移动，而是在分配给函数返回值的内存中再次构造`w`，条件（2）中规定，仅当返回值为局部对象时，才进行RVO，但是`makeWidget`的移动版本不满足这条件，再次看一下返回语句：

```cpp
return std::move(w);
```

返回的已经不是局部对象`w`，而是**`w`的引用**——`std::move(w)`的结果。返回局部对象的引用不满足RVO的第二个条件，所以编译器必须移动`w`到函数返回值的位置。开发者试图对要返回的局部变量用`std::move`帮助编译器优化，反而限制了编译器的优化选项。

有些情况很难让编译器实现RVO，比如当函数不同控制路径返回不同局部变量时。编译器必须产生一些代码在分配的函数返回值的内存中构造适当的局部变量，但是编译器不能确定哪个变量是合适的。如果这样，你可能会愿意以移动的代价来保证不会产生拷贝。那就是，极可能仍然认为应用`std::move`到一个要返回的局部对象上是合理的，只因为可以不再担心拷贝的代价。但C++标准关于RVO的部分表明，如果满足RVO的条件，但是编译器选择不执行拷贝消除，则返回的对象**必须被视为右值**。实际上，标准要求当RVO被允许时，或者实行拷贝消除，或者将`std::move`隐式应用于返回的局部对象。因此，在`makeWidget`的“拷贝”版本中，

```cpp
Widget makeWidget()                 //同之前一样
{
    Widget w;
    …
    return w;
}
```

编译器要不消除`w`的拷贝，要不把函数看成像下面写的一样：

```cpp
Widget makeWidget()
{
    Widget w;
    …
    return std::move(w);            //把w看成右值，因为不执行拷贝消除
}
```

这种情况与返回函数传值形参的情况很像。传值形参们没资格参与函数返回值的拷贝消除，但是如果作为返回值的话编译器会将其视作右值。结果就是，如果代码如下：

```cpp
Widget makeWidget(Widget w)         //传值形参，与函数返回的类型相同
{
    …
    return w;
}
```

编译器必须看成像下面这样写的代码：

```cpp
Widget makeWidget(Widget w)
{
    …
    return std::move(w);
}
```

这意味着，如果对从按值返回的函数返回来的局部对象使用`std::move`，你并不能帮助编译器（如果不能实行拷贝消除的话，他们必须把局部对象看做右值），而是阻碍其执行优化选项（通过阻止RVO）。在某些情况下，将`std::move`应用于局部变量可能是一件合理的事（即，你把一个变量传给函数，并且知道不会再用这个变量），但是满足RVO的`return`语句或者返回一个传值形参并不在此列。



**请记住：**

- 最后一次使用时，在右值引用上使用`std::move`，在通用引用上使用`std::forward`。
- 对按值返回的函数要返回的右值引用和通用引用，执行相同的操作。
- 如果局部对象可以被返回值优化消除，就绝不使用`std::move`或者`std::forward`。



## 条款二十六：避免在通用引用上重载

考虑下面的代码，它使用名字作为形参，打印当前日期和时间到日志中，然后将名字加入到一个全局数据结构中：

```c++
std::multiset<std::string> names;           //全局数据结构
void logAndAdd(const std::string& name)
{
    auto now =                              //获取当前时间
        std::chrono::system_clock::now();
    log(now, "logAndAdd");                  //志记信息
    names.emplace(name);                    //把name加到全局数据结构中；
}                 

std::string petName("Darla");
logAndAdd(petName);                     //传递左值std::string
logAndAdd(std::string("Persephone"));	//传递右值std::string
logAndAdd("Patty Dog");                 //传递字符串字面值
```

在第一个调用中，`logAndAdd`的形参`name`绑定到变量`petName`。在`logAndAdd`中`name`最终传给`names.emplace`。因为`name`是左值，会拷贝到`names`中。没有方法避免拷贝，因为是左值（`petName`）传递给`logAndAdd`的。

在第二个调用中，形参`name`绑定到右值（显式从“`Persephone`”创建的临时`std::string`）。`name`本身是个左值，所以它被拷贝到`names`中，但是我们意识到，原则上，它的值可以被移动到`names`中。本次调用中，我们有个拷贝代价，但是我们应该能用移动勉强应付。

在第三个调用中，形参`name`也绑定一个右值，但是这次是通过“`Patty Dog`”隐式创建的临时`std::string`变量。就像第二个调用中，`name`被拷贝到`names`，但是这里，传递给`logAndAdd`的实参是一个字符串字面量。如果直接将字符串字面量传递给`emplace`，就不会创建`std::string`的临时变量，而是直接在`std::multiset`中通过字面量构建`std::string`。在第三个调用中，我们有个`std::string`拷贝开销，但是我们连移动开销都不想要，更别说拷贝的。

使用通用引用：

```cpp
template<typename T>
void logAndAdd(T&& name)
{
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(std::forward<T>(name));
}

std::string petName("Darla");           //跟之前一样
logAndAdd(petName);                     //跟之前一样，拷贝左值到multiset
logAndAdd(std::string("Persephone"));	//移动右值而不是拷贝它
logAndAdd("Patty Dog");                 //在multiset直接创建std::string
                                        //而不是拷贝一个临时std::string
```

**在通用引用上重载：**

**例一：**

```cpp
std::string nameFromIdx(int idx);   //返回idx对应的名字

void logAndAdd(int idx)             //新的重载
{
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(nameFromIdx(idx));
}
```

之后的两个调用按照预期工作：

```cpp
std::string petName("Darla");           //跟之前一样

logAndAdd(petName);                     //跟之前一样，
logAndAdd(std::string("Persephone")); 	//这些调用都去调用
logAndAdd("Patty Dog");                 //T&&重载版本

logAndAdd(22);                          //调用int重载版本
```

事实上，这只能基本按照预期工作，假定一个客户将`short`类型索引传递给`logAndAdd`：

```cpp
short nameIdx;
…                                       //给nameIdx一个值
logAndAdd(nameIdx);                     //错误！
```

有两个重载的`logAndAdd`。使用通用引用的那个推导出`T`的类型是`short`，因此可以精确匹配。对于`int`类型参数的重载也可以在`short`类型提升后匹配成功。根据正常的重载解决规则，精确匹配优先于类型提升的匹配，所以被调用的是通用引用的重载。在通用引用那个重载中，`name`形参绑定到要传入的`short`上，然后`name`被`std::forward`给`names`（一个`std::multiset<std::string>`）的`emplace`成员函数，然后又被转发给`std::string`构造函数。`std::string`没有接受`short`的构造函数，所以`logAndAdd`调用里的`multiset::emplace`调用里的`std::string`构造函数调用失败。

**例二：**

在适当的条件下，C++会生成拷贝和移动构造函数，即使类包含了模板化的构造函数，模板函数能实例化产生与拷贝和移动构造函数一样的签名，也在合适的条件范围内。如果拷贝和移动构造被生成：

```cpp
class Person {
public:
    template<typename T>            //完美转发的构造函数
    explicit Person(T&& n)
    : name(std::forward<T>(n)) {}

    explicit Person(int idx);       //int的构造函数

    Person(const Person& rhs);      //拷贝构造函数（编译器生成）
    Person(Person&& rhs);           //移动构造函数（编译器生成）
    …
};
```

```cpp
Person p("Nancy"); 
auto cloneOfP(p);                   //从p创建新Person；这通不过编译！
```

这里我们试图通过一个`Person`实例创建另一个`Person`，显然应该调用拷贝构造即可。但是这份代码不是调用拷贝构造函数，而是调用完美转发构造函数。然后，完美转发的函数将尝试使用`Person`对象`p`初始化`Person`的`std::string`数据成员，编译器就会报错。

```cpp
class Person {
public:
    explicit Person(Person& n)          //由完美转发模板初始化
    : name(std::forward<Person&>(n)) {}

    explicit Person(int idx);           //同之前一样

    Person(const Person& rhs);          //拷贝构造函数（编译器生成的）
    …
};
```

其中`p`被传递给拷贝构造函数或者完美转发构造函数。调用拷贝构造函数要求在`p`前加上`const`的约束来满足函数形参的类型，而调用完美转发构造不需要加这些东西。从模板产生的重载函数是更好的匹配，所以编译器按照规则：调用最佳匹配的函数。“拷贝”non-`const`左值类型的`Person`交由完美转发构造函数处理，而不是拷贝构造函数。

如果将本例中的传递的对象改为`const`的，会得到完全不同的结果：

```cpp
const Person cp("Nancy");   //现在对象是const的
auto cloneOfP(cp);          //调用拷贝构造函数！
```

因为被拷贝的对象是`const`，是拷贝构造函数的精确匹配。虽然模板化的构造函数可以被实例化为有完全一样的函数签名，但是没啥影响，因为重载规则规定当模板实例化函数和非模板函数（或者称为“正常”函数）匹配优先级相当时，优先使用“正常”函数。

**例三：**

当继承纳入考虑范围时：

```cpp
class SpecialPerson: public Person {
public:
    SpecialPerson(const SpecialPerson& rhs) //拷贝构造函数，调用基类的
    : Person(rhs)                           //完美转发构造函数！
    { … }

    SpecialPerson(SpecialPerson&& rhs)      //移动构造函数，调用基类的
    : Person(std::move(rhs))                //完美转发构造函数！
    { … }
};
```

如同注释表示的，派生类的拷贝和移动构造函数没有调用基类的拷贝和移动构造函数，而是调用了基类的完美转发构造函数！因为派生类将`SpecialPerson`类型的实参传递给其基类，然后通过模板实例化和重载解析规则作用于基类`Person`。最终，代码无法编译，因为`std::string`没有接受一个`SpecialPerson`的构造函数。



**请记住：**

- 对通用引用形参的函数进行重载，通用引用函数的调用机会几乎总会比你期望的多得多。
- 完美转发构造函数是糟糕的实现，因为对于non-`const`左值，它们比拷贝构造函数而更匹配，而且会劫持派生类对于基类的拷贝和移动构造函数的调用。



## 条款二十七：熟悉通用引用重载的替代方法

**放弃重载：**

在Item26的`logAndAdd`例子中，可以使用不同的名字来避免在通用引用上的重载的弊端。例如两个重载的`logAndAdd`函数，可以分别改名为`logAndAddName`和`logAndAddNameIdx`。但是，这种方式不能用在`Person`构造函数中，因为构造函数的名字被语言固定了。

**传递`const T&`:**

一种替代方案是退回到C++98，然后将传递通用引用替换为传递lvalue-refrence-to-`const`。事实上，这是[Item26](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item26.html)中首先考虑的方法。缺点是效率不高。现在知道了通用引用和重载的相互关系，所以放弃一些效率来确保行为正确简单可能也是一种不错的折中。

**传值：**

通常在不增加复杂性的情况下提高性能的一种方法是，将按传引用形参替换为按值传递，这是违反直觉的。该设计遵循Item41中给出的建议，即在你知道要拷贝时就按值传递，这里，在`Person`的例子中展示：

```cpp
class Person {
public:
    explicit Person(std::string n)  //代替T&&构造函数，
    : name(std::move(n)) {}         //std::move的使用见条款41
  
    explicit Person(int idx)        //同之前一样
    : name(nameFromIdx(idx)) {}
    …

private:
    std::string name;
};
```

因为没有`std::string`构造函数可以接受整型参数，所有`int`或者其他整型变量（比如`std::size_t`、`short`、`long`等）都会使用`int`类型重载的构造函数。相似的，所有`std::string`类似的实参（还有可以用来创建`std::string`的东西，比如字面量“`Ruth`”等）都会使用`std::string`类型的重载构造函数。没有意外情况。

**使用tag dispatch:**

通过查看所有重载的所有形参以及调用点的所有传入实参，然后选择最优匹配的函数——考虑所有形参/实参的组合。通用引用通常提供了最优匹配，但是如果通用引用是包含其他**非**通用引用的形参列表的一部分，则非通用引用形参的较差匹配会使有一个通用引用的重载版本不被运行。这就是*tag dispatch*方法的基础。

Item26已经指出下面这段代码的弊端：

```c++
std::multiset<std::string> names;       //全局数据结构

template<typename T>                    //志记信息，将name添加到数据结构
void logAndAdd(T&& name)
{
    auto now = std::chrono::system_clokc::now();
    log(now, "logAndAdd");
    names.emplace(std::forward<T>(name));
}

std::string nameFromIdx(int idx);   //返回idx对应的名字

void logAndAdd(int idx)             //新的重载
{
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(nameFromIdx(idx));
}
```

解决方法：

两个真正执行逻辑的函数命名为`logAndAddImpl`，即我们使用重载。其中一个函数接受通用引用。但是每个函数接受第二个形参，表征传入的实参是否为整型。

```cpp
template<typename T>                            //非整型实参：添加到全局数据结构中
void logAndAddImpl(T&& name, std::false_type)	//译者注：高亮std::false_type
{
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(std::forward<T>(name));
}

std::string nameFromIdx(int idx);   //返回idx对应的名字

void logAndAdd(int idx, std::true_type)             //新的重载
{
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(nameFromIdx(idx));
}
```

```cpp
template<typename T>
void logAndAdd(T&& name) 
{
    logAndAddImpl(std::forward<T>(name),
                  std::is_integral<T>());   //不那么正确
}
```

类型`std::true_type`和`std::false_type`是“标签”（tag），其唯一目的就是强制重载解析按照我们的想法来执行。注意到我们甚至没有对这些参数进行命名。他们在运行时毫无用处，事实上我们希望编译器可以意识到这些标签形参没被使用，然后在程序执行时优化掉它们。（至少某些时候有些编译器会这样做。）通过创建标签对象，在`logAndAdd`内部将重载实现函数的调用“分发”（*dispatch*）给正确的重载。因此这个设计名称为：*tag dispatch*

但如果左值实参传递给通用引用`name`，对`T`类型推断会得到左值引用。所以如果左值`int`被传入`logAndAdd`，`T`将被推断为`int&`。这不是一个整型类型，因为引用不是整型类型。这意味着`std::is_integral<T>`对于任何左值实参返回false，即使确实传入了整型值。

C++标准库有一个*type trait*（参见Item9），`std::remove_reference`：

```cpp
template<typename T>
void logAndAdd(T&& name)
{
    logAndAddImpl(
        std::forward<T>(name),
        std::is_integral<typename std::remove_reference<T>::type>()
    );
}
```

**约束使用通用引用模板：**

Item26中所述第二个问题案例是`Person`类的完美转发构造函数，编译器可能会自行生成拷贝和移动构造函数，所以即使你只写了一个构造函数并在其中使用*tag dispatch*，有一些对构造函数的调用也被编译器生成的函数处理，绕过了分发机制。

`std::enable_if`可以提供一种强制编译器执行行为的方法，像是特定模板不存在一样。这种模板被称为被**禁止**（disabled）。默认情况下，所有模板是**启用**的（enabled），但是使用`std::enable_if`可以使得仅在`std::enable_if`**指定的条件**满足时模板才启用。

```cpp
class Person {
public:
    template<typename T,
             typename = typename std::enable_if<condition>::type>   //译者注：本行高亮，condition为某其他特定条件
    explicit Person(T&& n);
    …
};
```

这里我们想表示的条件是确认`T`不是`Person`类型，即模板构造函数应该在`T`不是`Person`类型的时候启用，*type trait*可以确定两个对象类型是否相同（`std::is_same`），这个例子需要的是`!std::is_same<Person, T>::value`。但是使用左值来初始化通用引用的话会推导成左值引用，`Person`和`Person&`类型是不同的。如果更精细考虑仅当`T`不是`Person`类型才启用模板构造函数，因此应该忽略：**是否是个引用**；**是不是`const`或者`volatile`**。这意味着需要一种方法消除对于`T`的引用，`const`，`volatile`修饰。再次，标准库提供了这样功能的*type trait*，就是`std::decay`。`std::decay<T>::type`与`T`是相同的，只不过会移除引用和cv限定符的修饰。因此加入条件后的代码如下：

`Person`的完美转发构造函数的声明如下：

```cpp
class Person {
public:
    template<
        typename T,
        typename = typename std::enable_if<
                       !std::is_same<Person, 
                                     typename std::decay<T>::type
                                    >::value
                   >::type
    >
    explicit Person(T&& n);
    …
};
```

假定从`Person`派生的类以常规方式实现拷贝和移动操作：

```cpp
class SpecialPerson: public Person {
public:
    SpecialPerson(const SpecialPerson& rhs) //拷贝构造函数，调用基类的
    : Person(rhs)                           //完美转发构造函数！
    { … }
    
    SpecialPerson(SpecialPerson&& rhs)      //移动构造函数，调用基类的
    : Person(std::move(rhs))                //完美转发构造函数！
    { … }
    
    …
};
```

标准库中也有*type trait*判断一个类型是否继承自另一个类型，就是`std::is_base_of`。如果`std::is_base_of<T1, T2>`是true就表示`T2`派生自`T1`。类型也可被认为是从他们自己派生，所以`std::is_base_of<T, T>::value`总是true。想要修正控制`Person`完美转发构造函数的启用条件，只有当`T`在消除引用和cv限定符之后，并且既不是`Person`又不是`Person`的派生类时，才满足条件。所以使用`std::is_base_of`代替`std::is_same`就可以了：

```cpp
class Person {
public:
    template<
        typename T,
        typename = typename std::enable_if<
                       !std::is_base_of<Person, 
                                        typename std::decay<T>::type
                                       >::value
                   >::type
    >
    explicit Person(T&& n);
    …
};
```

c++14版本：

```cpp
class Person  {                                         //C++14
public:
    template<
        typename T,
        typename = std::enable_if_t<                    //这儿更少的代码
                       !std::is_base_of<Person,
                                        std::decay_t<T> //还有这儿
                                       >::value
                   >                                    //还有这儿
    >
    explicit Person(T&& n);
    …
};
```

如果有整数参数的函数重载，只需要再加上形参T（除去引用后）是否是整数类型即可：

```cpp
class Person {
public:
    template<
        typename T,
        typename = std::enable_if_t<
            !std::is_base_of<Person, std::decay_t<T>>::value
            &&
            !std::is_integral<std::remove_reference_t<T>>::value
        >
    >
    explicit Person(T&& n)          //对于std::strings和可转化为
    : name(std::forward<T>(n))      //std::strings的实参的构造函数
    { … }

    explicit Person(int idx)        //对于整型实参的构造函数
    : name(nameFromIdx(idx))
    { … }

    …                               //拷贝、移动构造函数等

private:
    std::string name;
};
```

**折中：**

通常，完美转发更有效率，因为它避免了仅仅去为了符合形参声明的类型而创建临时对象。在`Person`构造函数的例子中，完美转发允许将“`Nancy`”这种字符串字面量转发到`Person`内部的`std::string`的构造函数，不使用完美转发的技术则会从字符串字面值创建一个临时`std::string`对象，来满足`Person`构造函数指定的形参要求。

完美转发也有缺点。即使某些类型的实参可以传递给接受特定类型的函数，也无法完美转发。Item30中探索了完美转发失败的例子。

第二个问题是当客户传递无效参数时错误消息的可理解性，例如假如客户传递了一个由`char16_t`（一种C++11引入的类型表示16位字符）而不是`char`（`std::string`包含的）组成的字符串字面值来创建一个`Person`对象：

```cpp
Person p(u"Konrad Zuse");   //“Konrad Zuse”由const char16_t类型字符组成
```

使用本条款中讨论的前三种方法，编译器将看到可用的采用`int`或者`std::string`的构造函数，它们或多或少会产生错误消息，表示没有可以从`const char16_t[12]`转换为`int`或者`std::string`的方法。但是，基于完美转发的方法，`const char16_t`不受约束地绑定到构造函数的形参。从那里将转发到`Person`的`std::string`数据成员的构造函数，在这里，调用者传入的内容（`const char16_t`数组）与所需内容（`std::string`构造函数可接受的类型）发生的不匹配会被发现。由此产生的错误消息会让人更印象深刻，在作者使用的编译器上，会产生超过160行错误信息。

在这个例子中，通用引用仅被转发一次（从`Person`构造函数到`std::string`构造函数），但是更复杂的系统中，在最终到达判断实参类型是否可接受的地方之前，通用引用会被多层函数调用转发。通用引用被转发的次数越多，产生的错误消息偏差就越大。许多开发者发现，这种特殊问题是发生在留有通用引用形参的接口上，这些接口以性能作为首要考虑点。

在`Person`这个例子中，我们知道完美转发函数的通用引用形参要作为`std::string`的初始化器，所以我们可以用`static_assert`来确认它可以起这个作用。`std::is_constructible`这个*type trait*执行编译时测试，确定一个类型的对象是否可以用另一个不同类型（或多个类型）的对象（或多个对象）来构造：

```cpp
class Person {
public:
    template<                       //同之前一样
        typename T,
        typename = std::enable_if_t<
            !std::is_base_of<Person, std::decay_t<T>>::value
            &&
            !std::is_integral<std::remove_reference_t<T>>::value
        >
    >
    explicit Person(T&& n)
    : name(std::forward<T>(n))
    {
        //断言可以用T对象创建std::string
        static_assert(
        std::is_constructible<std::string, T>::value,
        "Parameter n can't be used to construct a std::string"
        );

        …               //通常的构造函数的工作写在这

    }
    
    …                   //Person类的其他东西（同之前一样）
};
```

如果客户代码尝试使用无法构造`std::string`的类型创建`Person`，会导致指定的错误消息。在这个例子中，`static_assert`在构造函数体中，但是转发的代码作为成员初始化列表的部分在检查之前。所以作者使用的编译器，结果是由`static_assert`产生的清晰的错误消息在常规错误消息（多达160行以上那个）后出现。



**请记住：**

- 通用引用和重载的组合替代方案包括使用不同的函数名，通过lvalue-reference-to-`const`传递形参，按值传递形参，使用*tag dispatch*。
- 通过`std::enable_if`约束模板，允许组合通用引用和重载使用，但它也控制了编译器在哪种条件下才使用通用引用重载。
- 通用引用参数通常具有高效率的优势，但是可用性就值得斟酌。



## 条款二十八：理解引用折叠

当实参被用来实例化通用引用形参时，被推导的模板形参`T`根据实参是左值还是右值来编码。当左值实参被传入时，`T`被推导为左值引用。当右值被传入时，`T`被推导为非引用。

```cpp
template<typename T>
void func(T&& param);

Widget widgetFactory();     //返回右值的函数
Widget w;                   //一个变量（左值）
func(w);                    //用左值调用func；T被推导为Widget&
func(widgetFactory());      //用右值调用func；T被推导为Widget
```

**在C++中编写引用的引用是非法：**

```cpp
int x;
…
auto& & rx = x;             //错误！不能声明引用的引用
```

**引用折叠：**

规则：如果任一引用为左值引用，则结果为左值引用。否则（即，如果引用都是右值引用），结果为右值引用。引用折叠发生在四种情况下：

**第一，也是最常见的就是模板实例化：**

引用折叠是`std::forward`工作的一种关键机制。`std::forward`应用在通用引用参数上，所以经常能看到这样使用：

```cpp
template<typename T>
void f(T&& fParam)
{
    …                                   //做些工作
    someFunc(std::forward<T>(fParam));  //转发fParam到someFunc
}
```

因为`fParam`是通用引用，我们知道类型参数`T`的类型根据`f`被传入实参（即用来实例化`fParam`的表达式）是左值还是右值来编码。`std::forward`的作用是当且仅当传给`f`的实参为右值时，即`T`为非引用类型，才将`fParam`（左值）转化为一个右值。

`std::forward`可以这样实现：

```cpp
template<typename T>                                //在std命名空间
T&& forward(typename
                remove_reference<T>::type& param)
{
    return static_cast<T&&>(param);
}
```

假设传入到`f`的实参是`Widget`的左值类型。`T`被推导为`Widget&`，然后调用`std::forward`将实例化为`std::forward<Widget&>`。`Widget&`带入到上面的`std::forward`的实现中：

```cpp
Widget& && forward(typename 
                       remove_reference<Widget&>::type& param)
{ return static_cast<Widget& &&>(param); }
```

所以`std::forward`成为：

```cpp
Widget& && forward(Widget& param)
{ return static_cast<Widget& &&>(param); }
```

根据引用折叠规则，返回值和强制转换可以化简，最终版本的`std::forward`调用就是：

```cpp
Widget& forward(Widget& param)
{ return static_cast<Widget&>(param); }
```

当左值实参被传入到函数模板`f`时，`std::forward`被实例化为接受和返回左值引用。内部的转换不做任何事，因为`param`的类型已经是`Widget&`，所以转换没有影响。左值实参传入`std::forward`会返回左值引用。

假设传递给`f`的实参是一个`Widget`的右值：

`f`的类型参数`T`的推导类型就是`Widget`。`f`内部的`std::forward`调用因此为`std::forward<Widget>`，`std::forward`实现中把`T`换为`Widget`得到：

```cpp
Widget&& forward(typename
                     remove_reference<Widget>::type& param)
{ return static_cast<Widget&&>(param); }
```

将`std::remove_reference`引用到非引用类型`Widget`上还是相同的类型（`Widget`），所以`std::forward`变成：

```cpp
Widget&& forward(Widget& param)
{ return static_cast<Widget&&>(param); }
```

这里没有引用的引用，所以不需要引用折叠。

在C++14中，`std::remove_reference_t`的存在使得实现变得更简洁：

```cpp
template<typename T>                        //C++14；仍然在std命名空间
T&& forward(remove_reference_t<T>& param)
{
  return static_cast<T&&>(param);
}
```

**第二，是`auto`变量的类型生成：**

具体细节类似于模板，因为`auto`变量的类型推导基本与模板类型推导雷同：

```cpp
Widget widgetFactory();     //返回右值的函数
Widget w;                   //一个变量（左值）
func(w);                    //用左值调用func；T被推导为Widget&
func(widgetFactory());      //用右值调用func；T被推导为Widget
```

在auto的写法中，规则是类似的。声明

```cpp
auto&& w1 = w;
```

用一个左值初始化`w1`，因此为`auto`推导出类型`Widget&`。把`Widget&`代回`w1`声明中的`auto`里，产生了引用的引用，

```cpp
Widget& && w1 = w;
```

应用引用折叠规则，就是

```cpp
Widget& w1 = w
```

结果就是`w1`是一个左值引用。

另一方面，这个声明，

```cpp
auto&& w2 = widgetFactory();
```

使用右值初始化`w2`，为`auto`推导出非引用类型`Widget`。把`Widget`代入`auto`得到：

```cpp
Widget&& w2 = widgetFactory()
```

没有引用的引用，这就是最终结果，`w2`是个右值引用。

**第三，`typedef`和别名声明的产生和使用中：**

假设有一个`Widget`的类模板，该模板具有右值引用类型的嵌入式`typedef`：

```cpp
template<typename T>
class Widget {
public:
    typedef T&& RvalueRefToT;
    …
};
```

假设我们使用左值引用实例化`Widget`：

```cpp
Widget<int&> w;
```

`Widget`模板中把`T`替换为`int&`得到：

```cpp
typedef int& && RvalueRefToT;
```

引用折叠就会发挥作用：

```cpp
typedef int& RvalueRefToT;
```

这清楚表明我们为`typedef`选择的名字可能不是我们希望的那样：当使用左值引用类型实例化`Widget`时，`RvalueRefToT`是**左值引用**的`typedef`。

**第四，`decltype`使用的情况：**

如果在分析`decltype`期间，出现了引用的引用，引用折叠规则就会起作用。



**请记住：**

- 引用折叠发生在四种情况下：模板实例化，`auto`类型推导，`typedef`与别名声明的创建和使用，`decltype`。
- 当编译器在引用折叠环境中生成了引用的引用时，结果就是单个引用。有左值引用折叠结果就是左值引用，否则就是右值引用。
- 通用引用就是在特定上下文的右值引用，上下文是通过类型推导区分左值还是右值，并且发生引用折叠的那些地方。



## 条款二十九：假定移动操作不存在，成本高，未被使用

除了Item17和Item11的情况外，即使显式支持了移动操作，结果可能也没有那么好。比如，所有C++11的标准库容器都支持了移动操作，但是认为移动所有容器的开销都非常小是个错误。对于某些容器来说，压根就不存在开销小的方式来移动它所包含的内容。对另一些容器来说，容器的开销真正小的移动操作会有些容器元素不能满足的注意条件。

C++11的新容器`std::array`本质上是具有STL接口的内置数组。这与其他标准容器将内容存储在堆内存不同。存储具体数据在堆内存的容器，本身只保存了指向堆内存中容器内容的指针（真正实现当然更复杂一些，但是基本逻辑就是这样）。这个指针的存在使得在常数时间移动整个容器成为可能，只需要从源容器拷贝保存指向容器内容的指针到目标容器，然后将源指针置为空指针就可以了：

```cpp
std::vector<Widget> vw1;

//把数据存进vw1
…

//把vw1移动到vw2。以常数时间运行。只有vw1和vw2中的指针被改变
auto vw2 = std::move(vw1);
```

`std::array`没有这种指针实现，数据就保存在`std::array`对象中：

```cpp
std::array<Widget, 10000> aw1;

//把数据存进aw1
…

//把aw1移动到aw2。以线性时间运行。aw1中所有元素被移动到aw2
auto aw2 = std::move(aw1);
```

`aw1`中的元素被**移动**到了`aw2`中。假定`Widget`类的移动操作比复制操作快，移动`Widget`的`std::array`就比复制要快。所以`std::array`确实支持移动操作。但是使用`std::array`的移动操作还是复制操作都将花费线性时间的开销，因为每个容器中的元素终归需要拷贝或移动一次，这与“移动一个容器就像操作几个指针一样方便”的含义相去甚远。

另一方面，`std::string`提供了常数时间的移动操作和线性时间的复制操作。这听起来移动比复制快多了，但是可能不一定。许多字符串的实现采用了小字符串优化（*small string optimization*，SSO）。“小”字符串（比如长度小于15个字符的）存储在了`std::string`的缓冲区中，并没有存储在堆内存，移动这种存储的字符串并不必复制操作更快。

因此，存在几种情况，C++11的移动语义并无优势：

- **没有移动操作**：Item11的例子中，要移动的对象没有提供移动操作，所以移动的写法也会变成复制操作。
- **移动不会更快**：要移动的对象提供的移动操作并不比复制速度更快。
- **移动不可用**：Item17中的例子中，进行移动的上下文要求移动操作不会抛出异常，但是该操作没有被声明为`noexcept`。

值得一提的是，还有另一个场景，会使得移动并没有那么有效率：

- **源对象是左值**：除了极少数的情况外（例如[Item25](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item25.html)），只有右值可以作为移动操作的来源。

但是该条款的标题是假定移动操作不存在，成本高，未被使用。这就是通用代码中的典型情况，比如编写模板代码，因为你不清楚你处理的具体类型是什么。在这种情况下，你必须像出现移动语义之前那样，像在C++98里一样保守地去复制对象。但是，通常，你了解你代码里使用的类型，依赖他们的特性不变性（比如是否支持快速移动操作）。这种情况，你无需这个条款的假设，只需要查找所用类型的移动操作详细信息。如果类型提供了快速移动操作，并且在调用移动操作的上下文中使用对象，可以安全的使用快速移动操作替换复制操作。



**请记住：**

- 假定移动操作不存在，成本高，未被使用。
- 在已知的类型或者支持移动语义的代码中，就不需要上面的假设。



## 条款三十：熟悉完美转发失败的情况

假定有函数`f`，然后编写一个转发给它的函数（事实上是一个函数模板）：

```cpp
template<typename T>
void fwd(T&& param)             //接受任何实参
{
    f(std::forward<T>(param));  //转发给f
}
```

从本质上说，转发函数是通用的，接受任何类型的实参，并转发得到的任何东西。这种通用性的逻辑扩展是，转发函数不仅是模板，而且是可变模板，因此可以接受任何数量的实参。`fwd`的可变形式如下：

```cpp
template<typename... Ts>
void fwd(Ts&&... params)            //接受任何实参
{
    f(std::forward<Ts>(params)...); //转发给f
}
```

给定我们的目标函数`f`和转发函数`fwd`，如果`f`使用某特定实参会执行某个操作，但是`fwd`使用相同的实参会执行不同的操作，完美转发就会失败

```cpp
f( expression );        //调用f执行某个操作
fwd( expression );		//但调用fwd执行另一个操作，则fwd不能完美转发expression给f
```

导致这种失败的实参种类如下：

**花括号初始化器**：

假定`f`这样声明：

```cpp
void f(const std::vector<int>& v);
```

在这个例子中，用花括号初始化调用`f`通过编译，

```cpp
f({ 1, 2, 3 });         //可以，“{1, 2, 3}”隐式转换为std::vector<int>
```

但是传递相同的列表初始化给fwd不能编译

```cpp
fwd({ 1, 2, 3 });       //错误！不能编译
```

在对`f`的直接调用（例如`f({ 1, 2, 3 })`），编译器把调用地的实参和声明的实参进行比较，看看是否匹配，并且必要时执行隐式转换操作使得调用成功。在上面的例子中，从`{ 1, 2, 3 }`生成了临时`std::vector<int>`对象，因此`f`的形参`v`会绑定到`std::vector<int>`对象上。当通过调用函数模板`fwd`间接调用`f`时，编译器不再把调用地传入给`fwd`的实参和`f`的声明中形参类型进行比较。而是**推导**传入给`fwd`的实参类型，然后比较推导后的实参类型和`f`的形参声明类型。当下面情况任何一个发生时，完美转发就会失败：

- **编译器不能推导出`fwd`的一个或者多个形参类型。** 这种情况下代码无法编译。
- **编译器推导“错”了`fwd`的一个或者多个形参类型。** 在这里，“错误”可能意味着`fwd`的实例将无法使用推导出的类型进行编译，但是也可能意味着使用`fwd`的推导类型调用`f`，与用传给`fwd`的实参直接调用`f`表现出不一致的行为。这种不同行为的原因可能是因为`f`是个重载函数的名字，并且由于是“不正确的”类型推导，在`fwd`内部调用的`f`重载和直接调用的`f`重载不一样。

在上面的`fwd({ 1, 2, 3 })`例子中，问题在于，将花括号初始化传递给未声明为`std::initializer_list`的函数模板形参，被判定为——就像标准说的——“非推导上下文”。简单来讲，这意味着编译器不准在对`fwd`的调用中推导表达式`{ 1, 2, 3 }`的类型，因为`fwd`的形参没有声明为`std::initializer_list`。对于`fwd`形参的推导类型被阻止，编译器只能拒绝该调用。

[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)说明了使用花括号初始化的`auto`的变量的类型推导是成功的。这种变量被视为`std::initializer_list`对象，在转发函数应推导出类型为`std::initializer_list`：

```cpp
auto il = { 1, 2, 3 };  //il的类型被推导为std::initializer_list<int>
fwd(il);                //可以，完美转发il给f
```

**0或者NULL作为空指针：**

Item8说明当你试图传递`0`或者`NULL`作为空指针给模板时，类型推导会出错，会把传来的实参推导为一个整型类型（典型情况为`int`）而不是指针类型。结果就是不管是`0`还是`NULL`都不能作为空指针被完美转发。因此，传一个`nullptr`而不是`0`或者`NULL`。

**仅有声明的整型static const数据成员：**

通常，无需在类中定义整型`static const`数据成员；声明就可以了。这是因为编译器会对此类成员实行**常量传播**（*const propagation*），因此消除了保留内存的需要。比如，考虑下面的代码：

```cpp
class Widget {
public:
    static const std::size_t MinVals = 28;  //MinVal的声明
    …
};
…                                           //没有MinVals定义

std::vector<int> widgetData;
widgetData.reserve(Widget::MinVals);        //使用MinVals
```

这里，我们使用`Widget::MinVals`（或者简单点`MinVals`）来确定`widgetData`的初始容量，即使`MinVals`缺少定义。编译器通过将值28放入所有提到`MinVals`的位置来补充缺少的定义（就像它们被要求的那样）。没有为`MinVals`的值留存储空间是没有问题的。如果要使用`MinVals`的地址（例如，有人创建了指向`MinVals`的指针），则`MinVals`需要存储（这样指针才有可指向的东西），尽管上面的代码仍然可以编译，但是链接时就会报错，直到为`MinVals`提供定义。

按照这个思路，让`f`（`fwd`要转发实参给它的那个函数）这样声明：

```cpp
void f(std::size_t val);
```

使用`MinVals`调用`f`是可以的，因为编译器直接将值28代替`MinVals`：

```cpp
f(Widget::MinVals);         //可以，视为“f(28)”
```

不过如果我们尝试通过`fwd`调用`f`，事情不会进展那么顺利：

```cpp
fwd(Widget::MinVals);       //错误！不应该链接
```

代码可以编译，但是不应该链接。因为尽管代码中没有使用`MinVals`的地址，但是`fwd`的形参是通用引用，而引用，在编译器生成的代码中，通常被视作指针。在程序的二进制底层代码中（以及硬件中）指针和引用是一样的。在这个水平上，引用只是可以自动解引用的指针。只要给整型`static const`提供一个定义就可以链接成功，比如这样：

```cpp
const std::size_t Widget::MinVals;  //在Widget的.cpp文件
```

注意定义中不要重复初始化（这个例子中就是赋值28）。但是不要忽略这个细节。如果你忘了，并且在两个地方都提供了初始化，编译器就会报错，提醒你只能初始化一次。

**重载函数的名称和模板名称：**

假定我们的函数`f`（我们想通过`fwd`完美转发实参给的那个函数）可以通过向其传递执行某些功能的函数来自定义其行为。假设这个函数接受和返回值都是`int`，`f`声明为：

```cpp
void f(int (*pf)(int));             //pf = “process function”
```

值得注意的是，也可以使用更简单的非指针语法声明。含义与上面是一样的：

```cpp
void f(int pf(int));                //与上面定义相同的f
```

现在假设有了一个重载函数，`processVal`：

```cpp
int processVal(int value);
int processVal(int value, int priority);
```

我们可以传递`processVal`给`f`，

```cpp
f(processVal);                      //可以
```

`f`要求一个函数指针作为实参，但是`processVal`不是一个函数指针或者一个函数，它是同名的两个不同函数。但是，编译器可以知道它需要哪个：匹配上`f`的形参类型的那个。因此选择了仅带有一个`int`的`processVal`地址传递给`f`。但是，`fwd`是一个函数模板，没有它可接受的类型的信息，使得编译器不可能决定出哪个函数应被传递：

```cpp
fwd(processVal);                    //错误！那个processVal？
```

单用`processVal`是没有类型信息的，所以就不能类型推导，完美转发失败。

如果试图使用函数模板而不是（或者也加上）重载函数的名字，同样的问题也会发生。一个函数模板不代表单独一个函数，它表示一个函数族：

```cpp
template<typename T>
T workOnVal(T param)                //处理值的模板
{ … }

fwd(workOnVal);                     //错误！哪个workOnVal实例？
```

要让像`fwd`的完美转发函数接受一个重载函数名或者模板名，方法是指定要转发的那个重载或者实例。比如，你可以创造与`f`相同形参类型的函数指针，通过`processVal`或者`workOnVal`实例化这个函数指针（这可以引导选择正确版本的`processVal`或者产生正确的`workOnVal`实例），然后传递指针给`fwd`：

```cpp
using ProcessFuncType =                         //写个类型定义；见条款9
    int (*)(int);

ProcessFuncType processValPtr = processVal;     //指定所需的processVal签名

fwd(processValPtr);                             //可以
fwd(static_cast<ProcessFuncType>(workOnVal));   //也可以
```

当然，这要求你知道`fwd`转发的函数指针的类型。没有理由去假定完美转发函数会记录着这些东西。毕竟，完美转发被设计为接受任何内容。

**位域：**

假定IPv4的头部有如下模型：

```cpp
struct IPv4Header {
    std::uint32_t version:4,
                  IHL:4,
                  DSCP:6,
                  ECN:2,
                  totalLength:16;
    …
};
```

如果声明我们的函数`f`（转发函数`fwd`的目标）为接收一个`std::size_t`的形参，则使用`IPv4Header`对象的`totalLength`字段进行调用没有问题：

```cpp
void f(std::size_t sz);         //要调用的函数

IPv4Header h;
…
f(h.totalLength);               //可以
```

如果通过`fwd`转发`h.totalLength`给`f`呢，那就是一个不同的情况了：

```cpp
fwd(h.totalLength);             //错误！
```

问题在于`fwd`的形参是引用，而`h.totalLength`是non-`const`位域。non-`const`引用不应该绑定到位域。禁止的理由很充分。位域可能包含了机器字的任意部分（比如32位`int`的3-5位），但是这些东西无法直接寻址。所以没有办法创建一个指向任意*bit*的指针（C++规定你可以指向的最小单位是`char`），同样没有办法绑定引用到任意*bit*上。没有函数可以绑定引用到位域，也没有函数可以接受指向位域的指针，因为不存在这种指针。位域可以传给的形参种类只有按值传递的形参，有趣的是，还有reference-to-`const`。在传值形参的情况中，被调用的函数接受了一个位域的副本；在传reference-to-`const`形参的情况中，标准要求这个引用实际上绑定到存放位域值的副本对象，这个对象是某种整型（比如`int`）。reference-to-`const`不直接绑定到位域，而是绑定位域值拷贝到的一个普通对象。

传递位域给完美转发的关键就是利用传给的函数接受的是一个副本的事实。可以自己创建副本然后利用副本调用完美转发。在`IPv4Header`的例子中，可以如下写法：

```cpp
//拷贝位域值；参看条款6了解关于初始化形式的信息
auto length = static_cast<std::uint16_t>(h.totalLength);

fwd(length);                    //转发这个副本
```



**请记住：**

- 当模板类型推导失败或者推导出错误类型，完美转发会失败。
- 导致完美转发失败的实参种类有花括号初始化，作为空指针的`0`或者`NULL`，仅有声明的整型`static const`数据成员，模板和重载函数的名字，位域。



## 条款三十一：避免使用默认的捕获模式

`lambda`在C++11中有两种默认的捕获模式：按引用捕获和按值捕获。

**按引用捕获：**

按引用捕获会导致闭包中包含了对某个局部变量或者形参的引用，变量或形参只在定义*lambda*的作用域中可用。如果该*lambda*创建的闭包生命周期超过了局部变量或者形参的生命周期，那么闭包中的引用将会变成悬空引用。假如有元素是过滤函数（filtering function）的一个容器，该函数接受一个`int`，并返回一个`bool`，该`bool`的结果表示传入的值是否满足过滤条件：

```c++
using FilterContainer =                     //“using”参见条款9，
    std::vector<std::function<bool(int)>>;  //std::function参见条款2

FilterContainer filters;                    //过滤函数
```

可以添加一个过滤器，用来过滤掉5的倍数：

```c++
filters.emplace_back(                       //emplace_back的信息见条款42
    [](int value) { return value % 5 == 0; }
);
```

然而可能需要的是能够在运行期计算除数（divisor），即不能将5硬编码到*lambda*中。因此添加的过滤器逻辑将会是如下这样：

```c++
void addDivisorFilter()
{
    auto calc1 = computeSomeValue1();
    auto calc2 = computeSomeValue2();

    auto divisor = computeDivisor(calc1, calc2);

    filters.emplace_back(                               //危险！对divisor的引用
        [&](int value) { return value % divisor == 0; } //将会悬空！
    );
}
```

*lambda*对局部变量`divisor`进行了引用，但该变量的生命周期会在`addDivisorFilter`返回时结束，刚好就是在语句`filters.emplace_back`返回之后。因此添加到`filters`的函数添加完，该函数就死亡了。使用这个过滤器（译者注：就是那个添加进`filters`的函数）会导致未定义行为，这是由它被创建那一刻起就决定了的。

同样的问题也会出现在`divisor`的显式按引用捕获。

```c++
filters.emplace_back(
    [&divisor](int value) 			    //危险！对divisor的引用将会悬空！
    { return value % divisor == 0; }
);
```

但通过显式的捕获，**能更容易看到*lambda*的可行性依赖于变量`divisor`的生命周期。另外，写下“divisor”这个名字能够提醒我们要注意确保`divisor`的生命周期至少跟*lambda*闭包一样长。比起“`[&]`”传达的意思，显式捕获能让人更容易想起“确保没有悬空变量”。**

因此，显式列出*lambda*依赖的局部变量和形参，是更加符合软件工程规范的做法。

C++14支持了在*lambda*中使用`auto`来声明变量，上面的代码在C++14中可以进一步简化，`ContElemT`的别名可以去掉，`if`条件可以修改为：

```c++
if (std::all_of(begin(container), end(container),
               [&](const auto& value)               // C++14
               { return value % divisor == 0; }))			
```

**按值捕获：**

在通常情况下，按值捕获并不能完全解决悬空引用的问题。看下面的解决办法：

```c++
filters.emplace_back( 							    //现在divisor不会悬空了
    [=](int value) { return value % divisor == 0; }
);
```

虽然足以满足本实例的要求，但在通常情况下，按值捕获并不能完全解决悬空引用的问题。这里的问题是如果你按值捕获的是一个指针，你将该指针拷贝到*lambda*对应的闭包里，但这样并不能避免*lambda*外`delete`这个指针的行为，从而导致你的副本指针变成悬空指针。

例如，假设在一个`Widget`类，可以实现向过滤器的容器添加条目：

```c++
class Widget {
public:
    …                       //构造函数等
    void addFilter() const; //向filters添加条目
private:
    int divisor;            //在Widget的过滤器使用
};
```

这是`Widget::addFilter`的定义：

```c++
void Widget::addFilter() const
{
    filters.emplace_back(
        [=](int value) { return value % divisor == 0; }
    );
}	
```

错误，完全错误。

捕获只能应用于*lambda*被创建时所在作用域里的non-`static`局部变量（包括形参）。在`Widget::addFilter`的视线里，`divisor`并不是一个局部变量，而是`Widget`类的一个成员变量。它不能被捕获。而如果默认捕获模式被删除，代码就不能编译了，因为这表示该lambda不捕获任何外部变量，它只能使用其内部定义的变量，无法访问外部作用域中的任何变量：

```c++
void Widget::addFilter() const
{
    filters.emplace_back(                               //错误！
        [](int value) { return value % divisor == 0; }  //divisor不可用
    ); 
} 
```

另外，如果尝试去显式地捕获`divisor`变量（或者按引用或者按值——这不重要），也一样会编译失败，因为`divisor`不是一个局部变量或者形参。

```c++
void Widget::addFilter() const
{
    filters.emplace_back(
        [divisor](int value)                //错误！没有名为divisor局部变量可捕获
        { return value % divisor == 0; }
    );
}
```

所以如果默认按值捕获不能捕获`divisor`，而不用默认按值捕获代码就不能编译，解释就是这里隐式使用了一个原始指针：`this`。每一个non-`static`成员函数都有一个`this`指针，每次你使用一个类内的数据成员时都会使用到这个指针。例如，在任何`Widget`成员函数中，编译器会在内部将`divisor`替换成`this->divisor`。在默认按值捕获的`Widget::addFilter`版本中，

```c++
void Widget::addFilter() const
{
    filters.emplace_back(
        [=](int value) { return value % divisor == 0; }
    );
}
```

真正被捕获的是`Widget`的`this`指针，而不是`divisor`。编译器会将上面的代码看成以下的写法：

```c++
void Widget::addFilter() const
{
    auto currentObjectPtr = this;

    filters.emplace_back(
        [currentObjectPtr](int value)
        { return value % currentObjectPtr->divisor == 0; }
    );
}
```

因此*lambda*闭包的生命周期与`Widget`对象的关系是，闭包内含有`Widget`的`this`指针的拷贝。

使用智能指针，调用`Widget`的成员函数：

```c++
using FilterContainer = 					//跟之前一样
    std::vector<std::function<bool(int)>>;

FilterContainer filters;                    //跟之前一样

void doSomeWork()
{
    auto pw =                               //创建Widget；std::make_unique
        std::make_unique<Widget>();         //见条款21

    pw->addFilter();                        //添加使用Widget::divisor的过滤器

    …
}                                           //销毁Widget；filters现在持有悬空指针！
```

当调用`doSomeWork`时，就会创建一个过滤器，其生命周期依赖于由`std::make_unique`产生的`Widget`对象，即一个含有指向`Widget`的指针——`Widget`的`this`指针——的过滤器。这个过滤器被添加到`filters`中，但当`doSomeWork`结束时，`Widget`会由管理它的`std::unique_ptr`来销毁（见[Item18](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item18.html)）。从这时起，`filter`会含有一个存着悬空指针的条目。

这个特定的问题可以通过给你想捕获的数据成员做一个局部副本，然后捕获这个副本去解决：

```c++
void Widget::addFilter() const
{
    auto divisorCopy = divisor;                 //拷贝数据成员

    filters.emplace_back(
        [divisorCopy](int value)                //捕获副本
        { return value % divisorCopy == 0; }	//使用副本
    );
}
```

事实上如果采用这种方法，默认的按值捕获也是可行的。

```c++
void Widget::addFilter() const
{
    auto divisorCopy = divisor;                 //拷贝数据成员

    filters.emplace_back(
        [=](int value)                          //捕获副本
        { return value % divisorCopy == 0; }	//使用副本
    );
}
```

但为什么要冒险呢？当一开始你认为你捕获的是`divisor`的时候，默认捕获模式就是造成可能意外地捕获`this`的元凶。

在C++14中，一个更好的捕获成员变量的方式是使用通用的*lambda*捕获：

```c++
void Widget::addFilter() const
{
    filters.emplace_back(                   //C++14：
        [divisor = divisor](int value)      //拷贝divisor到闭包
        { return value % divisor == 0; }	//使用这个副本
    );
}
```

这种通用的*lambda*捕获并没有默认的捕获模式，因此在C++14中，本条款的建议——避免使用默认捕获模式——仍然是成立的。

使用默认的按值捕获还有另外的一个缺点，它们预示了相关的闭包是独立的并且不受外部数据变化的影响。一般来说，这是不对的。*lambda*可能会依赖局部变量和形参（它们可能被捕获），还有静态存储生命周期（static storage duration）的对象。这些对象定义在全局空间或者命名空间，或者在类、函数、文件中声明为`static`。这些对象也能在*lambda*里使用，但它们不能被捕获。但默认按值捕获可能会因此误导你，让你以为捕获了这些变量。参考下面版本的`addDivisorFilter`函数：

```c++
void addDivisorFilter()
{
    static auto calc1 = computeSomeValue1();    //现在是static
    static auto calc2 = computeSomeValue2();    //现在是static
    static auto divisor =                       //现在是static
    computeDivisor(calc1, calc2);

    filters.emplace_back(
        [=](int value)                          //什么也没捕获到！
        { return value % divisor == 0; }        //引用上面的static
    );

    ++divisor;                                  //调整divisor
}
```

可能看到“`[=]`”，就会认为“好的，*lambda*拷贝了所有使用的对象，因此这是独立的”。但其实不独立。这个*lambda*没有使用任何的non-`static`局部变量，所以它没有捕获任何东西。然而*lambda*的代码引用了`static`变量`divisor`，在每次调用`addDivisorFilter`的结尾，`divisor`都会递增，通过这个函数添加到`filters`的所有*lambda*都展示新的行为（分别对应新的`divisor`值）。这个*lambda*是通过引用捕获`divisor`，这和默认的按值捕获表示的含义有着直接的矛盾。如果你一开始就避免使用默认的按值捕获模式，你就能解除代码的风险。



**请记住：**

- 默认的按引用捕获可能会导致悬空引用。
- 默认的按值捕获对于悬空指针很敏感（尤其是`this`指针），并且它会误导人产生*lambda*是独立的想法。



## 条款三十二：使用初始化捕获来移动对象到闭包中

**C++14的初始化捕获：**

- 移动捕获
- 使用初始化捕获可以让你指定：
  - 从lambda生成的闭包类中的**数据成员名称**；
  - 初始化该成员的**表达式**；

使用初始化捕获将`std::unique_ptr`移动到闭包中的方法：

```c++
class Widget {                          //一些有用的类型
public:
    …
    bool isValidated() const;
    bool isProcessed() const;
    bool isArchived() const;
private:
    …
};

auto pw = std::make_unique<Widget>();   //创建Widget；使用std::make_unique
                                        //的有关信息参见条款21

…                                       //设置*pw

auto func = [pw = std::move(pw)]        //使用std::move(pw)初始化闭包数据成员
            { return pw->isValidated()
                     && pw->isArchived(); };
```

在上面的示例中，“`=`”左侧的名称`pw`表示闭包类中的数据成员，而右侧的名称`pw`表示在*lambda*上方声明的对象，即由调用`std::make_unique`去初始化的变量。因此，“`pw = std::move(pw)`”的意思是“在闭包中创建一个数据成员`pw`，并使用将`std::move`应用于局部变量`pw`的结果来初始化该数据成员”。“`=`”左侧的作用域不同于右侧的作用域。左侧的作用域是闭包类，右侧的作用域和*lambda*定义所在的作用域相同。

“设置`*pw`”表示在由`std::make_unique`创建`Widget`之后，*lambda*捕获到指向`Widget`的`std::unique_ptr`之前，该`Widget`以某种方式进行了修改。如果不需要这样的设置，即如果`std::make_unique`创建的`Widget`处于适合被*lambda*捕获的状态，则不需要局部变量`pw`，因为闭包类的数据成员可以通过`std::make_unique`直接初始化：

```c++
auto func = [pw = std::make_unique<Widget>()]   //使用调用make_unique得到的结果
            { return pw->isValidated()          //初始化闭包数据成员
                     && pw->isArchived(); };
```

**C++11手动模拟C++14`lambda`的移动语义效果：**

```c++
class IsValAndArch {                            //“is validated and archived”
public:
    using DataType = std::unique_ptr<Widget>;
    
    explicit IsValAndArch(DataType&& ptr)       //条款25解释了std::move的使用
    : pw(std::move(ptr)) {}
    
    bool operator()() const
    { return pw->isValidated() && pw->isArchived(); }
    
private:
    DataType pw;
};

auto func = IsValAndArch(std::make_unique<Widget>())();
```

**C++11使用`std::bind()`模拟`lambda`的移动语义：**

1. 将要捕获的对象移动到由`std::bind`产生的函数对象中；
2. 将“被捕获的”对象的引用赋予给\*lambda\*。

假设创建一个本地的`std::vector`，在其中放入一组适当的值，然后将其移动到闭包中。在C++14中：

```c++
std::vector<double> data;               //要移动进闭包的对象

…                                       //填充data

auto func = [data = std::move(data)]    //C++14初始化捕获
            { /*使用data*/ };
```

C++11的等效代码如下：

```c++
std::vector<double> data;               //同上

…                                       //同上

auto func =
    std::bind(                              //C++11模拟初始化捕获
        [](const std::vector<double>& data) //译者注：本行高亮
        { /*使用data*/ },
        std::move(data)                     //译者注：本行高亮
    );
```

如*lambda*表达式一样，`std::bind`产生函数对象。由`std::bind`返回的函数对象称为**bind对象**（*bind objects*）。`std::bind`的第一个实参是可调用对象，后续实参表示要传递给该对象的值。一个bind对象包含了传递给`std::bind`的所有实参的副本。对于每个左值实参，bind对象中的对应对象都是复制构造的。对于每个右值，它都是移动构造的。在此示例中，第二个实参是一个右值（`std::move`的结果，请参见[Item23](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item23.html)），因此将`data`移动构造到绑定对象中。这种移动构造是模仿移动捕获的关键，因为将右值移动到bind对象是我们解决无法将右值移动到C++11闭包中的方法。当调用`func`（bind对象）时，`func`中所移动构造的`data`副本将作为实参传递给`std::bind`中的*lambda*。该*lambda*与我们在C++14中使用的*lambda*相同，只是添加了一个形参`data`来对应我们的伪移动捕获对象。此形参是对bind对象中`data`副本的左值引用。（这不是右值引用，因为尽管用于初始化`data`副本的表达式（`std::move(data)`）为右值，但`data`副本本身为左值。）因此，*lambda*将对绑定在对象内部的移动构造的`data`副本进行操作。

默认情况下，从*lambda*生成的闭包类中的`operator()`成员函数为`const`的。这具有在*lambda*主体内把闭包中的所有数据成员渲染为`const`的效果。但是，bind对象内部的移动构造的`data`副本不是`const`的，因此，为了防止在*lambda*内修改该`data`副本，*lambda*的形参应声明为reference-to-`const`。 如果将*lambda*声明为`mutable`，则闭包类中的`operator()`将不会声明为`const`，并且在*lambda*的形参声明中省略`const`也是合适的：

```c++
auto func =
    std::bind(                                  //C++11对mutable lambda
        [](std::vector<double>& data) mutable	//初始化捕获的模拟
        { /*使用data*/ },
        std::move(data)
    );
```

因为bind对象存储着传递给`std::bind`的所有实参的副本，所以在我们的示例中，bind对象包含由*lambda*生成的闭包副本，这是它的第一个实参。 因此闭包的生命周期与bind对象的生命周期相同。 这很重要，因为这意味着只要存在闭包，包含伪移动捕获对象的bind对象也将存在。

基本要点应该清楚：

- 无法移动构造一个对象到C++11闭包，但是可以将对象移动构造进C++11的bind对象。
- 在C++11中模拟移动捕获包括将对象移动构造进bind对象，然后通过传引用将移动构造的对象传递给*lambda*。
- 由于bind对象的生命周期与闭包对象的生命周期相同，因此可以将bind对象中的对象视为闭包中的对象。

第二个示例，闭包中创建`std::unique_ptr`的C++14代码：

```c++
auto func = [pw = std::make_unique<Widget>()]   //同之前一样
            { return pw->isValidated()          //在闭包中创建pw
                     && pw->isArchived(); };
```

这是C++11的模拟实现：

```c++
auto func = std::bind(
                [](const std::unique_ptr<Widget>& pw)
                { return pw->isValidated()
                         && pw->isArchived(); },
                std::make_unique<Widget>()
            );
```



**请记住：**

- 使用C++14的初始化捕获将对象移动到闭包中。
- 在C++11中，通过手写类或`std::bind`的方式来模拟初始化捕获。



## 条款三十三：对auto&&形参使用decltype以std::forward他们

**C++14泛型lambda：在lambda的形参中使用auto关键字**

这个特性的实现是非常直截了当的：即在闭包类中的`operator()`函数是一个函数模版。例如存在这么一个*lambda*，

```c++
auto f = [](auto x){ return func(normalize(x)); };
```

对应的闭包类中的函数调用操作符看来就变成这样：

```c++
class SomeCompilerGeneratedClassName {
public:
    template<typename T>                //auto返回类型见条款3
    auto operator()(T x) const
    { return func(normalize(x)); }
    …                                   //其他闭包类功能
};
```

如果函数`normalize`对待左值右值的方式不一样，这个*lambda*的实现方式就不大合适了，因为即使传递到*lambda*的实参是一个右值，*lambda*传递进`normalize`的总是一个左值（形参`x`）。实现这个*lambda*的正确方式是把`x`完美转发给函数`normalize`。这样做需要对代码做两处修改。首先，`x`需要改成通用引用（见Item24），其次，需要使用`std::forward`将`x`转发到函数`normalize`（见Item25）：

```c++
auto f = [](auto&& x)
         { return func(normalize(std::forward<???>(x))); };
```

一般来说，当你在使用完美转发时，你是在一个接受类型参数为`T`的模版函数里，所以你可以写`std::forward<T>`。但在泛型*lambda*中，没有可用的类型参数`T`。在*lambda*生成的闭包里，模版化的`operator()`函数中的确有一个`T`，但在*lambda*里却无法直接使用它，所以也没什么用。

Item28解释过如果一个左值实参被传给通用引用的形参，那么形参类型会变成左值引用。传递的是右值，形参就会变成右值引用。这意味着在这个*lambda*中，可以通过检查形参`x`的类型来确定传递进来的实参是一个左值还是右值，`decltype`就可以实现这样的效果（见Item3）。传递给*lambda*的是一个左值，`decltype(x)`就能产生一个左值引用；如果传递的是一个右值，`decltype(x)`就会产生右值引用。

Item28也解释过在调用`std::forward`时，惯例决定了类型实参是左值引用时来表明要传进左值，类型实参是非引用就表明要传进右值。在前面的*lambda*中，如果`x`绑定的是一个左值，`decltype(x)`就能产生一个左值引用。这符合惯例。然而如果`x`绑定的是一个右值，`decltype(x)`就会产生右值引用，而不是常规的非引用。

Item28中关于`std::forward`的C++14实现：

```c++
template<typename T>                        //在std命名空间
T&& forward(remove_reference_t<T>& param)
{
    return static_cast<T&&>(param);
}
```

如果用户想要完美转发一个`Widget`类型的右值时，它会使用`Widget`类型（即非引用类型）来实例化`std::forward`，然后产生以下的函数：

```c++
Widget&& forward(Widget& param)             //当T是Widget时的std::forward实例
{
    return static_cast<Widget&&>(param);
}
```

如果用户代码想要完美转发一个`Widget`类型的右值，但没有遵守规则将`T`指定为非引用类型，而是将`T`指定为右值引用。在`std::forward`实例化、应用了`std::remove_reference_t`后，引用折叠之前，`std::forward`看起来像这样：

```c++
Widget&& && forward(Widget& param)          //当T是Widget&&时的std::forward实例
{                                           //（引用折叠之前）
    return static_cast<Widget&& &&>(param);
}
```

应用了引用折叠之后（右值引用的右值引用变成单个右值引用），代码会变成：

```c++
Widget&& forward(Widget& param)             //当T是Widget&&时的std::forward实例
{                                           //（引用折叠之后）
    return static_cast<Widget&&>(param);
}
```

对比这个实例和用`Widget`设置`T`去实例化产生的结果，它们完全相同。表明用右值引用类型和用非引用类型去初始化`std::forward`产生的相同的结果。

因此*lambda*的完美转发可以写成：

```c++
auto f =
    [](auto&& param)
    {
        return
            func(normalize(std::forward<decltype(param)>(param)));
    };
```

再加上6个点，就可以让*lambda*完美转发接受多个形参了，因为C++14中的*lambda*也可以是可变形参的：

```c++
auto f =
    [](auto&&... params)
    {
        return
            func(normalize(std::forward<decltype(params)>(params)...));
    };
```



**请记住：**

- 对`auto&&`形参使用`decltype`以`std::forward`它们。



## 条款三十四：优先考虑lambda表达式而非std::bind

***lambda*更易读：**

假设有一个设置警报器的函数：

```c++
//一个时间点的类型定义（语法见条款9）
using Time = std::chrono::steady_clock::time_point;

//“enum class”见条款10
enum class Sound { Beep, Siren, Whistle };

//时间段的类型定义
using Duration = std::chrono::steady_clock::duration;

//在时间t，使用s声音响铃时长d
void setAlarm(Time t, Sound s, Duration d);
```

假设在程序的某个时刻，我们已经确定需要设置一个小时后响30秒的警报器。 但是，具体声音仍未确定。可以编写一个*lambda*来修改`setAlarm`的界面，以便仅需要指定声音：

```c++
//setSoundL（“L”指代“lambda”）是个函数对象，允许指定一小时后响30秒的警报器的声音
auto setSoundL =
    [](Sound s) 
    {
        //使std::chrono部件在不指定限定的情况下可用
        using namespace std::chrono;

        setAlarm(steady_clock::now() + hours(1),    //一小时后响30秒的闹钟
                 s,                                 //译注：setAlarm三行高亮
                 seconds(30));
    };
```

通过使用标准后缀如秒（`s`），毫秒（`ms`）和小时（`h`）等简化在C++14中的代码，其中标准后缀基于C++11对用户自定义常量的支持。这些后缀在`std::literals`命名空间中实现，因此上述代码可以按照以下方式重写：

```c++
auto setSoundL =
    [](Sound s)
    {
        using namespace std::chrono;
        using namespace std::literals;      //对于C++14后缀

        setAlarm(steady_clock::now() + 1h,	//C++14写法，但是含义同上
                 s,
                 30s);
    };
```

第一次编写对应的`std::bind`调用：

```c++
using namespace std::chrono;                //同上
using namespace std::literals;
using namespace std::placeholders;          //“_1”使用需要

auto setSoundB =                            //“B”代表“bind”
    std::bind(setAlarm,
              steady_clock::now() + 1h,     //不正确！见下
              _1,
              30s);
```

在对`std::bind`的调用中未标识此实参的类型，因此读者必须查阅`setAlarm`声明以确定将哪种实参传递给`setSoundB`。代码并不完全正确。在*lambda*中，表达式`steady_clock::now() + 1h`显然是`setAlarm`的实参。调用`setAlarm`时将对其进行计算。但是，在`std::bind`调用中，将`steady_clock::now() + 1h`作为实参传递给了`std::bind`，而不是`setAlarm`。这意味着将在调用`std::bind`时对表达式进行求值，并且该表达式产生的时间将存储在产生的bind对象中。结果，警报器将被设置为在**调用`std::bind`后一小时**发出声音，而不是在调用`setAlarm`一小时后发出。

第二次编写对应的`std::bind`调用：

将对`std::bind`的第二个调用嵌套在第一个调用中：

```c++
auto setSoundB =
    std::bind(setAlarm,
              std::bind(std::plus<>(), std::bind(steady_clock::now), 1h),
              _1,
              30s);
```

尖括号之间未指定任何类型，即该代码包含“`std::plus<>`”，而不是“`std::plus<type>`”。 在C++14中，通常可以省略标准运算符模板的模板类型实参，因此无需在此处提供。 C++11没有提供此类功能，因此等效于*lambda*的C++11 `std::bind`为：

```c++
using namespace std::chrono;                //同上
using namespace std::placeholders;
auto setSoundB =
    std::bind(setAlarm,
              std::bind(std::plus<steady_clock::time_point>(),
                        std::bind(steady_clock::now),
                        hours(1)),
              _1,
              seconds(30));
```

当`setAlarm`重载时，会出现一个新问题。 假设有一个重载函数，其中第四个形参指定了音量：

```c++
enum class Volume { Normal, Loud, LoudPlusPlus };

void setAlarm(Time t, Sound s, Duration d, Volume v);
```

*lambda*能继续像以前一样使用，因为根据重载规则选择了`setAlarm`的三实参版本：

```c++
auto setSoundL =                            //和之前一样
    [](Sound s)
    {
        using namespace std::chrono;
        setAlarm(steady_clock::now() + 1h,  //可以，调用三实参版本的setAlarm
                 s,
                 30s);
    };
```

然而，`std::bind`的调用将会编译失败：

```c++
auto setSoundB =                            //错误！哪个setAlarm？
    std::bind(setAlarm,
              std::bind(std::plus<>(),
                        steady_clock::now(),
                        1h),
              _1,
              30s);
```

这里的问题是，编译器无法确定应将两个`setAlarm`函数中的哪一个传递给`std::bind`。 它们仅有的是一个函数名称，而这个单一个函数名称是有歧义的。

要使对`std::bind`的调用能编译，必须将`setAlarm`强制转换为适当的函数指针类型：

```c++
using SetAlarm3ParamType = void(*)(Time t, Sound s, Duration d);

auto setSoundB =                                            //现在可以了
    std::bind(static_cast<SetAlarm3ParamType>(setAlarm),
              std::bind(std::plus<>(),
                        steady_clock::now(),
                        1h), 
              _1,
              30s);
```

***lambda*可能会比使用`std::bind`能生成更快的代码：**

 在`setSoundL`的函数调用操作符（即*lambda*的闭包类对应的函数调用操作符）内部，对`setAlarm`的调用是正常的函数调用，编译器可以按常规方式进行内联：

```c++
setSoundL(Sound::Siren);    //setAlarm函数体在这可以很好地内联
```

但是，对`std::bind`的调用是将函数指针传递给`setAlarm`，这意味着在`setSoundB`的函数调用操作符（即绑定对象的函数调用操作符）内部，对`setAlarm`的调用是通过一个函数指针。 编译器不太可能通过函数指针内联函数，这意味着与通过`setSoundL`进行调用相比，通过`setSoundB`对`setAlarm的`调用，其函数不大可能被内联：

```c++
setSoundB(Sound::Siren); 	//setAlarm函数体在这不太可能内联
```

因此，使用*lambda*可能会比使用`std::bind`能生成更快的代码。

**lambda比`std::bind`更具有表达力：**

假设有一个函数可以创建`Widget`的压缩副本，

```c++
enum class CompLevel { Low, Normal, High }; //压缩等级

Widget compress(const Widget& w,            //制作w的压缩副本
                CompLevel lev);
```

并且创建一个函数对象，该函数对象允许指定`Widget w`的压缩级别。这种使用`std::bind`的话将创建一个这样的对象：

```c++
Widget w;
using namespace std::placeholders;
auto compressRateB = std::bind(compress, w, _1);
```

将`w`传递给`std::bind`时，必须将其存储起来，以便以后进行压缩。它存储在对象`compressRateB`中，是按值捕获的（`std::bind`总是拷贝它的实参，但是调用者可以使用引用来存储实参，这要通过应用`std::ref`到实参上实现。`auto compressRateB = std::bind(compress, std::ref(w), _1);`的结果就是`compressRateB`行为像是持有`w`的引用而非副本。），但唯一知道的方法是记住`std::bind`的工作方式；在对`std::bind`的调用中没有任何迹象。然而在*lambda*方法中，其中`w`是通过值还是通过引用捕获是显式的：

```c++
auto compressRateL =                //w是按值捕获，lev是按值传递
    [w](CompLevel lev)
    { return compress(w, lev); };
```

同样明确的是形参是如何传递给*lambda*的。 在这里，很明显形参`lev`是通过值传递的。 因此：

```c++
compressRateL(CompLevel::High);     //实参按值传递
```

但是在对由`std::bind`生成的对象调用中，实参如何传递？

```c++
compressRateB(CompLevel::High);     //实参如何传递？
```

同样，唯一的方法是记住`std::bind`的工作方式。（答案是传递给bind对象的所有实参都是通过引用传递的，因为此类对象的函数调用运算符使用完美转发。）

**在C++11中，在两个受约束的情况下证明使用`std::bind`是合理的：**

- **移动捕获**。C++11的*lambda*不提供移动捕获，但是可以通过结合*lambda*和`std::bind`来模拟。 有关详细信息，请参阅Item32，该条款还解释了在C++14中，*lambda*对初始化捕获的支持消除了这个模拟的需求。
- **多态函数对象**。因为bind对象上的函数调用运算符使用完美转发，所以它可以接受任何类型的实参（以Item30中描述的完美转发的限制为界限）。当你要绑定带有模板化函数调用运算符的对象时，此功能很有用。 例如这个类，

```c++
class PolyWidget {
public:
    template<typename T>
    void operator()(const T& param);
    …
};
```

`std::bind`可以如下绑定一个`PolyWidget`对象：

```c++
PolyWidget pw;
auto boundPW = std::bind(pw, _1);
```

`boundPW`可以接受任意类型的对象了：

```c++
boundPW(1930);              //传int给PolyWidget::operator()
boundPW(nullptr);           //传nullptr给PolyWidget::operator()
boundPW("Rosebud"); 		//传字面值给PolyWidget::operator()
```

这一点无法使用C++11的*lambda*做到。 但是，在C++14中，可以通过带有`auto`形参的*lambda*轻松实现：

```c++
auto boundPW = [pw](const auto& param)  //C++14 
               { pw(param); };
```

当然，这些是特殊情况，并且是暂时的特殊情况，因为支持C++14 *lambda*的编译器越来越普遍。



**请记住：**

- 与使用`std::bind`相比，*lambda*更易读，更具表达力并且可能更高效。
- 只有在C++11中，`std::bind`可能对实现移动捕获或绑定带有模板化函数调用运算符的对象时会很有用。



## 条款三十五：优先考虑基于任务的编程而非基于线程的编程

如果开发者想要异步执行`doAsyncWork`函数，通常有两种方式。其一是通过创建`std::thread`执行`doAsyncWork`，这是应用了**基于线程**（*thread-based*）的方式：

```cpp
int doAsyncWork();
std::thread t(doAsyncWork);
```

其二是将`doAsyncWork`传递给`std::async`，一种**基于任务**（*task-based*）的策略：

```cpp
auto fut = std::async(doAsyncWork); //“fut”表示“future”
```

这种方式中，传递给`std::async`的函数对象被称为一个**任务**（*task*）。

**`std::thread` API不能直接访问异步执行的结果，如果执行函数有异常抛出，代码会终止执行：**

假设调用`doAsyncWork`的代码对于其提供的返回值是有需求的。基于线程的方法对此无能为力，而基于任务的方法就简单了，因为`std::async`返回的*future*提供了`get`函数（从而可以获取返回值）。如果`doAsycnWork`发生了异常，`get`函数就显得更为重要，因为`get`函数可以提供抛出异常的访问，而基于线程的方法，如果`doAsyncWork`抛出了异常，程序会直接终止（通过调用`std::terminate`）。

**避免手动的线程耗尽、资源超额、负责均衡、平台适配性管理：**

在C++并发软件中总结了“*thread*”的三种含义：

- 硬件线程（hardware threads）是真实执行计算的线程。现代计算机体系结构为每个CPU核心提供一个或者多个硬件线程。
- 软件线程（software threads）（也被称为系统线程（OS threads、system threads））是操作系统（假设有一个操作系统。有些嵌入式系统没有。）管理的在硬件线程上执行的线程。通常可以存在比硬件线程更多数量的软件线程，因为当软件线程被阻塞的时候（比如 I/O、同步锁或者条件变量），操作系统可以调度其他未阻塞的软件线程执行提供吞吐量。
- **`std::thread`** 是C++执行过程的对象，并作为软件线程的句柄（*handle*）。有些`std::thread`对象代表“空”句柄，即没有对应软件线程，因为它们处在默认构造状态（即没有函数要执行）；有些被移动走（移动到的`std::thread`就作为这个软件线程的句柄）；有些被`join`（它们要运行的函数已经运行完）；有些被`detach`（它们和对应的软件线程之间的连接关系被打断）。

软件线程是有限的资源。如果开发者试图创建大于系统支持的线程数量，会抛出`std::system_error`异常。即使你编写了不抛出异常的代码，这仍然会发生，比如下面的代码，即使 `doAsyncWork`是 `noexcept`，

```cpp
int doAsyncWork() noexcept;         //noexcept见条款14
```

这段代码仍然会抛出异常：

```cpp
std::thread t(doAsyncWork);         //如果没有更多线程可用，则抛出异常
```

一种方法是在当前线程执行`doAsyncWork`，但是这可能会导致负载不均，而且如果当前线程是GUI线程，可能会导致响应时间过长的问题。另一种方法是等待某些当前运行的软件线程结束之后再创建新的`std::thread`，但是仍然有可能当前运行的线程在等待`doAsyncWork`的动作（例如产生一个结果或者报告一个条件变量）。

即使没有超出软件线程的限额，仍然可能会遇到**资源超额**（*oversubscription*）的麻烦。这是一种当前准备运行的（即未阻塞的）软件线程大于硬件线程的数量的情况。情况发生时，线程调度器（操作系统的典型部分）会将软件线程时间切片，分配到硬件上。当一个软件线程的时间片执行结束，会让给另一个软件线程，此时发生上下文切换。软件线程的上下文切换会增加系统的软件线程管理开销，当软件线程安排到与上次时间片运行时不同的硬件线程上，这个开销会更高。这种情况下，（1）CPU缓存对这个软件线程很冷淡（即几乎没有什么数据，也没有有用的操作指南）；（2）“新”软件线程的缓存数据会“污染”“旧”线程的数据，旧线程之前运行在这个核心上，而且还有可能再次在这里运行。

避免资源超额很困难，因为软件线程之于硬件线程的最佳比例取决于软件线程的执行频率，那是动态改变的，比如一个程序从IO密集型变成计算密集型，执行频率是会改变的。而且比例还依赖上下文切换的开销以及软件线程对于CPU缓存的使用效率。此外，硬件线程的数量和CPU缓存的细节（比如缓存多大，相应速度多少）取决于机器的体系结构，即使经过调校，在某一种机器平台避免了资源超额（而仍然保持硬件的繁忙状态），换一个其他类型的机器这个调校并不能提供较好效果的保证。

使用`std::async`：

```cpp
auto fut = std::async(doAsyncWork); //线程管理责任交给了标准库的开发者
```

这种调用方式将线程管理的职责转交给C++标准库的开发者，这种调用方式会减少抛出资源超额异常的可能性，因为这个调用可能不会开启一个新的线程。以这种形式调用（即使用默认启动策略见Item36），`std::async`不保证会创建新的软件线程。然而，他们允许通过调度器来将特定函数（本例中为`doAsyncWork`）运行在等待此函数结果的线程上（即在对`fut`调用`get`或者`wait`的线程上），合理的调度器在系统资源超额或者线程耗尽时就会利用这个自由度。如果考虑自己实现“在等待结果的线程上运行输出结果的函数”，之前提到了可能引出负载不均衡的问题，这问题不那么容易解决，因为应该是`std::async`和运行时的调度程序来解决这个问题而不是你。遇到负载不均衡问题时，对机器内发生的事情，运行时调度程序比你有更全面的了解，因为它管理的是所有执行过程，而不仅仅个别开发者运行的代码。有了`std::async`，GUI线程中响应变慢仍然是个问题，因为调度器并不知道你的哪个线程有高响应要求。这种情况下，你会想通过向`std::async`传递`std::launch::async`启动策略来保证想运行函数在不同的线程上执行（见[Item36）。

对比基于线程的编程方式，基于任务的设计为开发者避免了手动线程管理的痛苦，并且自然提供了一种获取异步执行程序的结果（即返回值或者异常）的方式。当然，仍然存在一些场景直接使用`std::thread`会更有优势：

- **你需要访问非常基础的线程API**。C++并发API通常是通过操作系统提供的系统级API（pthreads或者Windows threads）来实现的，系统级API通常会提供更加灵活的操作方式（举个例子，C++没有线程优先级和亲和性的概念）。为了提供对底层系统级线程API的访问，`std::thread`对象提供了`native_handle`的成员函数，而`std::future`（即`std::async`返回的东西）没有这种能力。
- **你需要且能够优化应用的线程使用**。举个例子，你要开发一款已知执行概况的服务器软件，部署在有固定硬件特性的机器上，作为唯一的关键进程。
- **你需要实现C++并发API之外的线程技术**，比如，C++实现中未支持的平台的线程池



**请记住：**

- `std::thread` API不能直接访问异步执行的结果，如果执行函数有异常抛出，代码会终止执行。
- 基于线程的编程方式需要手动的线程耗尽、资源超额、负责均衡、平台适配性管理。
- 通过带有默认启动策略的`std::async`进行基于任务的编程方式会解决大部分问题。

## 条款三十六：如果有异步的必要请指定std::launch::async

`std::async`有两种标准策略，每种都通过`std::launch`这个限域`enum`的一个枚举名表示（关于枚举的更多细节参见[Item10](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item10.html)）。假定一个函数`f`传给`std::async`来执行：

- **`std::launch::async`启动策略**意味着`f`必须异步执行，即在不同的线程。
- **`std::launch::deferred`启动策略**意味着`f`仅当在`std::async`返回的*future*上调用`get`或者`wait`时才执行。这表示`f`**推迟**到存在这样的调用时才执行（译者注：异步与并发是两个不同概念，这里侧重于惰性求值）。当`get`或`wait`被调用，`f`会同步执行，即调用方被阻塞，直到`f`运行结束。如果`get`和`wait`都没有被调用，`f`将不会被执行。（这是个简化说法。关键点不是要在其上调用`get`或`wait`的那个*future*，而是*future*引用的那个共享状态。（[Item38](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item38.html)讨论了*future*与共享状态的关系。）因为`std::future`支持移动，也可以用来构造`std::shared_future`，并且因为`std::shared_future`可以被拷贝，对共享状态——对`f`传到的那个`std::async`进行调用产生的——进行引用的*future*对象，有可能与`std::async`返回的那个*future*对象不同。这非常绕口，所以经常回避这个事实，简称为在`std::async`返回的*future*上调用`get`或`wait`。）

**`std::async`的默认启动策略：**

下面的两种调用含义相同：

```cpp
auto fut1 = std::async(f);                      //使用默认启动策略运行f
auto fut2 = std::async(std::launch::async |     //使用async或者deferred运行f
                       std::launch::deferred,
                       f);
```

因此默认策略允许`f`异步或者同步执行。如同[Item35](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/Item35.html)中指出，这种灵活性允许`std::async`和标准库的线程管理组件承担线程创建和销毁的责任，避免资源超额，以及平衡负载。这就是使用`std::async`并发编程如此方便的原因。

给定一个线程`t`执行此语句：

```cpp
auto fut = std::async(f);   //使用默认启动策略运行f
```

- **无法预测`f`是否会与`t`并发运行**，因为`f`可能被安排延迟运行。
- **无法预测`f`是否会在与某线程相异的另一线程上执行，这个某线程在`fut`上调用`get`或`wait`**。如果对`fut`调用函数的线程是`t`，含义就是无法预测`f`是否在异于`t`的另一线程上执行。
- **无法预测`f`是否执行**，因为不能确保在程序每条路径上，都会不会在`fut`上调用`get`或者`wait`。

默认启动策略的调度灵活性导致使用`thread_local`变量比较麻烦，因为这意味着如果`f`读写了**线程本地存储**（*thread-local storage*，TLS），不可能预测到哪个线程的变量被访问：

```cpp
auto fut = std::async(f);   //f的TLS可能是为单独的线程建的，
                            //也可能是为在fut上调用get或者wait的线程建的
```

这还会影响到基于`wait`的循环使用超时机制，因为在一个延时的任务（参见Item35）上调用`wait_for`或者`wait_until`会产生`std::launch::deferred`值。意味着，以下循环看似应该最终会终止，但可能实际上永远运行：

```cpp
using namespace std::literals;      //为了使用C++14中的时间段后缀；参见条款34

void f()                            //f休眠1秒，然后返回
{
    std::this_thread::sleep_for(1s);
}

auto fut = std::async(f);           //异步运行f（理论上）

while (fut.wait_for(100ms) !=       //循环，直到f完成运行时停止...
       std::future_status::ready)   //但是有可能永远不会发生！
{
    …
}
```

如果`f`与调用`std::async`的线程并发运行（即，如果为`f`选择的启动策略是`std::launch::async`），这里没有问题（假定`f`最终会执行完毕），但是如果`f`是延迟执行，`fut.wait_for`将总是返回`std::future_status::deferred`。这永远不等于`std::future_status::ready`，循环会永远执行下去。这种错误很容易在开发和单元测试中忽略，因为它可能在负载过高时才能显现出来。那些是使机器资源超额或者线程耗尽的条件，此时任务推迟执行才最有可能发生。毕竟，如果硬件没有资源耗尽，没有理由不安排任务并发执行。

只需要检查与`std::async`对应的`future`是否被延迟执行即可，那样就会避免进入无限循环。不幸的是，没有直接的方法来查看`future`是否被延迟执行。相反，你必须调用一个超时函数——比如`wait_for`这种函数。在这个情况中，你不想等待任何事，只想查看返回值是否是`std::future_status::deferred`，所以无须怀疑，使用0调用`wait_for`：

```cpp
auto fut = std::async(f);               //同上

if (fut.wait_for(0s) ==                 //如果task是deferred（被延迟）状态
    std::future_status::deferred)
{
    …                                   //在fut上调用wait或get来异步调用f
} else {                                //task没有deferred（被延迟）
    while (fut.wait_for(100ms) !=       //不可能无限循环（假设f完成）
           std::future_status::ready) {
        …                               //task没deferred（被延迟），也没ready（已准备）
                                        //做并行工作直到已准备
    }
    …                                   //fut是ready（已准备）状态
}
```

这些各种考虑的结果就是，只要满足以下条件，`std::async`的默认启动策略就可以使用：

- 任务不需要和执行`get`或`wait`的线程并行执行。
- 读写哪个线程的`thread_local`变量没什么问题。
- 可以保证会在`std::async`返回的*future*上调用`get`或`wait`，或者该任务可能永远不会执行也可以接受。
- 使用`wait_for`或`wait_until`编码时考虑到了延迟状态。

如果上述条件任何一个都满足不了，你可能想要保证`std::async`会安排任务进行真正的异步执行。

**将`std::launch::async`作为第一个实参传递：**

事实上，对于一个类似`std::async`行为的函数，但是会自动使用`std::launch::async`作为启动策略的工具，拥有它会非常方便，而且编写起来很容易也使它看起来很棒。C++11版本如下：

```cpp
template<typename F, typename... Ts>
inline
std::future<typename std::result_of<F(Ts...)>::type>
reallyAsync(F&& f, Ts&&... params)          //返回异步调用f(params...)得来的future
{
    return std::async(std::launch::async,
                      std::forward<F>(f),
                      std::forward<Ts>(params)...);
}
//F：是一个模板参数，代表一个函数、函数指针、lambda 表达式或任何其他可调用对象的类型。
//F(Ts...)：表示使用类型 F 和参数 Ts... 进行调用，允许 std::result_of 推导出 F 调用时的返回类型。
```

这个函数接受一个可调用对象`f`和0或多个形参`params`，然后完美转发（参见[Item25](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item25.html)）给`std::async`，使用`std::launch::async`作为启动策略。就像`std::async`一样，返回`std::future`作为用`params`调用`f`得到的结果。确定结果的类型很容易，因为*type trait* `std::result_of`可以提供给你。（参见Item9关于*type trait*的详细表述。）

`reallyAsync`就像`std::async`一样使用：

```cpp
auto fut = reallyAsync(f); //异步运行f，如果std::async抛出异常它也会抛出
```

在C++14中，`reallyAsync`返回类型的推导能力可以简化函数的声明：

```cpp
template<typename F, typename... Ts>
inline
auto                                        // C++14
reallyAsync(F&& f, Ts&&... params)
{
    return std::async(std::launch::async,
                      std::forward<F>(f),
                      std::forward<Ts>(params)...);
}
```

这个版本清楚表明，`reallyAsync`除了使用`std::launch::async`启动策略之外什么也没有做。



**请记住：**

- `std::async`的默认启动策略是异步和同步执行兼有的。
- 这个灵活性导致访问`thread_local`s的不确定性，隐含了任务可能不会被执行的意思，会影响调用基于超时的`wait`的程序逻辑。
- 如果异步执行任务非常关键，则指定`std::launch::async`。



## 条款三十七：使std::thread在所有路径最后都不可结合

**每个`std::thread`对象处于两个状态之一：可结合的或者不可结合的：**

可结合状态的`std::thread`对应于正在运行或者可能要运行的异步执行线程。比如，对应于一个阻塞的（*blocked*）或者等待调度的线程的`std::thread`是可结合的，对应于运行结束的线程的`std::thread`也可以认为是可结合的。`std::thread`的可结合性如此重要的原因之一就是当可结合的线程的析构函数被调用，程序执行会终止。

不可结合的`std::thread`对象包括：

- **默认构造的`std::thread`s**。这种`std::thread`没有函数执行，因此没有对应到底层执行线程上。
- **已经被移动走的`std::thread`对象**。移动的结果就是一个`std::thread`原来对应的执行线程现在对应于另一个`std::thread`。
- **已经被`join`的`std::thread`** 。在`join`之后，`std::thread`不再对应于已经运行完了的执行线程。
- **已经被`detach`的`std::thread`** 。`detach`断开了`std::thread`对象与执行线程之间的连接。

假定有一个函数`doWork`，我们希望设置做过滤的线程的优先级。Item35阐释了那需要线程的原生句柄，只能通过`std::thread`的API来完成；基于任务的API（比如*future*）做不到。所以最终采用基于线程而不是基于任务：

```cpp
constexpr auto tenMillion = 10'000'000;           //constexpr见条款15
												  //C++14使用单引号作为数字分隔符

bool doWork(std::function<bool(int)> filter,    //返回计算是否执行；
            int maxVal = tenMillion)            //std::function见条款2
{
    std::vector<int> goodVals;                  //满足filter的值

    std::thread t([&filter, maxVal, &goodVals]  //填充goodVals
                  {
                      for (auto i = 0; i <= maxVal; ++i)
                          { if (filter(i)) goodVals.push_back(i); }
                  });

    auto nh = t.native_handle();                //使用t的原生句柄
    …                                           //来设置t的优先级

    if (conditionsAreSatisfied()) {
        t.join();                               //等t完成
        performComputation(goodVals);
        return true;                            //执行了计算
    }
    return false;                               //未执行计算
}
```

开始运行之后设置`t`的优先级太晚，更好的设计是在挂起状态时开始`t`（这样可以在执行任何计算前调整优先级），可以转到Item39，此处忽略。返回`doWork`。如果`conditionsAreSatisfied()`返回`true`，没什么问题，但是如果返回`false`或者抛出异常，在`doWork`结束调用`t`的析构函数时，`std::thread`对象`t`会是可结合的。这造成程序执行中止。那是因为另外两种显而易见的方式更糟：

- **隐式`join`** 。这种情况下，`std::thread`的析构函数将等待其底层的异步执行线程完成。这听起来是合理的，但是可能会导致难以追踪的异常表现。比如，如果`conditonAreStatisfied()`已经返回了`false`，`doWork`继续等待过滤器应用于所有值就很违反直觉。

- **隐式`detach`** 。这种情况下，`std::thread`析构函数会分离`std::thread`与其底层的线程。底层线程继续运行。听起来比`join`的方式好，但是可能导致更严重的调试问题。比如，在`doWork`中，`goodVals`是通过引用捕获的局部变量。它也被*lambda*修改（通过调用`push_back`）。假定，*lambda*异步执行时，`conditionsAreSatisfied()`返回`false`。这时，`doWork`返回，同时局部变量（包括`goodVals`）被销毁。栈被弹出，并在`doWork`的调用点继续执行线程。

  调用点之后的语句有时会进行其他函数调用，并且至少一个这样的调用可能会占用曾经被`doWork`使用的栈位置。我们调用那么一个函数`f`。当`f`运行时，`doWork`启动的*lambda*仍在继续异步运行。该*lambda*可能在栈内存上调用`push_back`，该内存曾属于`goodVals`，但是现在是`f`的栈内存的某个位置。这意味着对`f`来说，内存被自动修改了！

**确保使用`std::thread`对象时，在所有的路径上超出定义所在的作用域时都是不可结合的：**

覆盖每条路径可能很复杂，可能包括自然执行通过作用域，或者通过`return`，`continue`，`break`，`goto`或异常跳出作用域，有太多可能的路径。每当你想在执行跳至块之外的每条路径执行某种操作，最通用的方式就是将该操作放入局部对象的析构函数中。这些对象称为**RAII对象**（*RAII objects*），从**RAII类**中实例化。RAII类在标准库中很常见。比如STL容器（每个容器析构函数都销毁容器中的内容物并释放内存），标准智能指针（Item18-20解释了，`std::uniqu_ptr`的析构函数调用他指向的对象的删除器，`std::shared_ptr`和`std::weak_ptr`的析构函数递减引用计数），`std::fstream`对象（它们的析构函数关闭对应的文件）等。但是标准库没有`std::thread`的RAII类，可能是因为标准委员会拒绝将`join`和`detach`作为默认选项，不知道应该怎么样完成RAII。

自行实现`ThreadRAII`：

```cpp
class ThreadRAII {
public:
    enum class DtorAction { join, detach };     //enum class的信息见条款10
    
    ThreadRAII(std::thread&& t, DtorAction a)   //析构函数中对t实行a动作
    : action(a), t(std::move(t)) {}

    ~ThreadRAII()
    {                                           //可结合性测试见下
        if (t.joinable()) {
            if (action == DtorAction::join) {
                t.join();
            } else {
                t.detach();
            }
        }
    }

    std::thread& get() { return t; }            //见下

private:
    DtorAction action;
    std::thread t;
};
```

- 构造器只接受`std::thread`右值，因为我们想要把传来的`std::thread`对象移动进`ThreadRAII`。（`std::thread`不可以复制。）
- 构造器的形参顺序设计的符合调用者直觉（首先传递`std::thread`，然后选择析构执行的动作，这比反过来更合理），但是成员初始化列表设计的匹配成员声明的顺序。将`std::thread`对象放在声明最后。在这个类中，这个顺序没什么特别之处，但是通常，可能一个数据成员的初始化依赖于另一个，因为`std::thread`对象可能会在初始化结束后就立即执行函数了，所以在最后声明是一个好习惯。这样就能保证一旦构造结束，在前面的所有数据成员都初始化完毕，可以供`std::thread`数据成员绑定的异步运行的线程安全使用。
- `ThreadRAII`提供了`get`函数访问内部的`std::thread`对象。这类似于标准智能指针提供的`get`函数，可以提供访问原始指针的入口。提供`get`函数避免了`ThreadRAII`复制完整`std::thread`接口的需要，也意味着`ThreadRAII`可以在需要`std::thread`对象的上下文环境中使用。
- 在`ThreadRAII`析构函数调用`std::thread`对象`t`的成员函数之前，检查`t`是否可结合。这是必须的，因为在不可结合的`std::thread`上调用`join`或`detach`会导致未定义行为。客户端可能会构造一个`std::thread`，然后用它构造一个`ThreadRAII`，使用`get`获取`t`，然后移动`t`，或者调用`join`或`detach`，每一个操作都使得`t`变为不可结合的。
- 在`t.joinable()`的执行和调用`join`或`detach`的中间，可能有其他线程改变了`t`为不可结合，这个这个担心不必要。只有调用成员函数才能使`std::thread`对象从可结合变为不可结合状态，比如`join`，`detach`或者移动操作。在`ThreadRAII`对象析构函数调用时，应当没有其他线程在那个对象上调用成员函数。如果同时进行调用，那肯定是有竞争的，但是不在析构函数中，是在客户端代码中试图同时在一个对象上调用两个成员函数（析构函数和其他函数）。通常，仅当所有都为`const`成员函数时，在一个对象同时调用多个成员函数才是安全的。

在`doWork`的例子上使用`ThreadRAII`的代码如下：

```cpp
bool doWork(std::function<bool(int)> filter,        //同之前一样
            int maxVal = tenMillion)
{
    std::vector<int> goodVals;                      //同之前一样

    ThreadRAII t(                                   //使用RAII对象
        std::thread([&filter, maxVal, &goodVals]
                    {
                        for (auto i = 0; i <= maxVal; ++i)
                            { if (filter(i)) goodVals.push_back(i); }
                    }),
                    ThreadRAII::DtorAction::join    //RAII动作
    );

    auto nh = t.get().native_handle();
    …

    if (conditionsAreSatisfied()) {
        t.get().join();
        performComputation(goodVals);
        return true;
    }

    return false;
}
```

选择在`ThreadRAII`的析构函数对异步执行的线程进行`join`，因为在先前分析中，`detach`可能导致噩梦般的调试过程。我们之前也分析了`join`可能会导致表现异常（坦率说，也可能调试困难），但是在未定义行为（`detach`导致），程序终止（使用原生`std::thread`导致），或者表现异常之间选择一个后果，可能表现异常是最好的那个。Item39表明了使用`ThreadRAII`来保证在`std::thread`的析构时执行`join`有时不仅可能导致程序表现异常，还可能导致程序挂起。“适当”的解决方案是此类程序应该和异步执行的*lambda*通信，告诉它不需要执行了，可以直接返回，但是C++11中不支持**可中断线程**（*interruptible threads*）。可以查阅相关书籍自行实现。

Item17说明因为`ThreadRAII`声明了一个析构函数，因此不会有编译器生成移动操作，但是没有理由`ThreadRAII`对象不能移动。如果要求编译器生成这些函数，函数的功能也正确，所以显式声明来告诉编译器自动生成也是合适的：

```cpp
class ThreadRAII {
public:
    enum class DtorAction { join, detach };         //跟之前一样

    ThreadRAII(std::thread&& t, DtorAction a)       //跟之前一样
    : action(a), t(std::move(t)) {}

    ~ThreadRAII()
    {
        …                                           //跟之前一样
    }

    ThreadRAII(ThreadRAII&&) = default;             //支持移动
    ThreadRAII& operator=(ThreadRAII&&) = default;

    std::thread& get() { return t; }                //跟之前一样

private: // as before
    DtorAction action;
    std::thread t;
};
```



**请记住：**

- 在所有路径上保证`thread`最终是不可结合的。
- 析构时`join`会导致难以调试的表现异常问题。
- 析构时`detach`会导致难以调试的未定义行为。
- 声明类数据成员时，最后声明`std::thread`对象。



## 条款三十八：关注不同线程句柄的析构行为

**共享状态讲解：**

Item37中说明了可结合的`std::thread`对应于执行的系统线程。未延迟（non-deferred）任务的*future*与系统线程有相似的关系。因此，可以将`std::thread`对象和*future*对象都视作系统线程的**句柄**（*handles*）。而可结合的`std::thread`析构会终止你的程序，因为两个其他的替代选择——隐式`join`或者隐式`detach`都是更加糟糕的。但是，*future*的析构表现有时就像执行了隐式`join`，有时又像是隐式执行了`detach`，有时又没有执行这两个选择。它永远不会造成程序终止。实际上*future*是通信信道的一端，被调用者（通常是异步执行）将计算结果写入通信信道中（通常通过`std::promise`对象），调用者使用*future*读取结果。这样的通信信道可以用在任何你需要从程序一个地方传递信息到另一个地方的场景。

被调用者的结果存储：

被调用者会在调用者`get`相关的*future*之前执行完成，所以结果不能存储在被调用者的`std::promise`。`std::promise`对象是局部的，当被调用者执行结束后，会被销毁。结果同样不能存储在调用者的*future*，因为（当然还有其他原因）`std::future`可能会被用来创建`std::shared_future`（这会将被调用者的结果所有权从`std::future`转移给`std::shared_future`），而`std::shared_future`在`std::future`被销毁之后可能被复制很多次。鉴于不是所有的结果都可以被拷贝（即只可移动类型），并且结果的生命周期至少与最后一个引用它的*future*一样长。因为与被调用者关联的对象和与调用者关联的对象都不适合存储这个结果，所以必须存储在两者之外的位置。此位置称为**共享状态**（*shared state*）。共享状态通常是基于堆的对象，但是标准并未指定其类型、接口和实现。标准库的作者可以通过任何他们喜欢的方式来实现共享状态。

共享状态的存在非常重要，因为*future*的析构函数——这个条款的话题——取决于与*future*关联的共享状态。特别地，

- **引用了共享状态——使用`std::async`启动的未延迟任务建立的那个——的最后一个\*future\*的析构函数会阻塞住**，直到任务完成。本质上，这种*future*的析构函数对执行异步任务的线程执行了隐式的`join`。
- **其他所有\*future\*的析构函数简单地销毁\*future\*对象**。对于异步执行的任务，就像对底层的线程执行`detach`。对于延迟任务来说如果这是最后一个*future*，意味着这个延迟任务永远不会执行了。

这些规则听起来好复杂。

**我们真正要处理的是一个简单的“正常”行为以及一个单独的例外：**

正常行为是*future*析构函数销毁*future*。就是这样。那意味着不`join`也不`detach`，也不运行什么，只销毁*future*的数据成员（当然，还做了另一件事，就是递减了共享状态中的引用计数，这个共享状态是由引用它的*future*和被调用者的`std::promise`共同控制的。这个引用计数让库知道共享状态什么时候可以被销毁。）

正常行为的例外情况仅在某个`future`同时满足下列所有情况下才会出现：

- **它关联到由于调用`std::async`而创建出的共享状态**。
- **任务的启动策略是`std::launch::async`**（参见[Item36](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item36.html)），原因是运行时系统选择了该策略，或者在对`std::async`的调用中指定了该策略。
- **这个\*future\*是关联共享状态的最后一个\*future\***。对于`std::future`，情况总是如此，对于`std::shared_future`，如果还有其他的`std::shared_future`，与要被销毁的*future*引用相同的共享状态，则要被销毁的*future*遵循正常行为（即简单地销毁它的数据成员）。

只有当上面的三个条件都满足时，*future*的析构函数才会表现“异常”行为，就是在异步任务执行完之前阻塞住。实际上，这相当于对由于运行`std::async`创建出任务的线程隐式`join`。

*future的API没办法确定是否future引用了一个`std::async`调用产生的共享状态，因此给定一个任意的future对象，无法判断会不会阻塞析构函数从而等待异步任务的完成。当然，如果你有办法知道给定的future**不*满足上面条件的任意一条（比如由于程序逻辑造成的不满足），你就可以确定析构函数不会执行“异常”行为。比如，只有通过`std::async`创建的共享状态才有资格执行“异常”行为，但是有其他创建共享状态的方式。一种是使用`std::packaged_task`，一个`std::packaged_task`对象通过包覆（wrapping）方式准备一个函数（或者其他可调用对象）来异步执行，然后将其结果放入共享状态中。然后通过`std::packaged_task`的`get_future`函数可以获取有关该共享状态的future*：*

```cpp
int calcValue();                //要运行的函数
std::packaged_task<int()>       //包覆calcValue以异步运行
    pt(calcValue);
auto fut = pt.get_future();     //从pt获取future
```

此时，我们知道*future*没有关联`std::async`创建的共享状态，所以析构函数肯定正常方式执行。

一旦被创建，`std::packaged_task`类型的`pt`就可以在一个线程上执行。（也可以通过调用`std::async`运行，但是如果你想使用`std::async`运行任务，没有理由使用`std::packaged_task`，因为在`std::packaged_task`安排任务并执行之前，`std::async`会做`std::packaged_task`做的所有事。）

`std::packaged_task`不可拷贝，所以当`pt`被传递给`std::thread`构造函数时，必须先转为右值：

```cpp
std::thread t(std::move(pt));   //在t上运行pt
```

将这些语句放在一个作用域的语句块里：

```cpp
{                                   //开始代码块
    std::packaged_task<int()>
        pt(calcValue); 
    
    auto fut = pt.get_future(); 
    
    std::thread t(std::move(pt));   //见下
    …
}                                   //结束代码块
```

此处最有趣的代码是在创建`std::thread`对象`t`之后，代码块结束前的“`…`”。使代码有趣的事是，在“`…`”中`t`上会发生什么。有三种可能性：

- **对`t`什么也不做**。这种情况，`t`会在语句块结束时是可结合的，这会使得程序终止（参见[Item37](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item37.html)）。
- **对`t`调用`join`**。这种情况，不需要`fut`在它的析构函数处阻塞，因为`join`被显式调用了。
- **对`t`调用`detach`**。这种情况，不需要在`fut`的析构函数执行`detach`，因为显式调用了。

换句话说，当你有一个关联了`std::packaged_task`创建的共享状态的*future*时，不需要采取特殊的销毁策略，因为通常你会代码中做终止、结合或分离这些决定之一，来操作`std::packaged_task`的运行所在的那个`std::thread`。



**请记住：**

- *future*的正常析构行为就是销毁*future*本身的数据成员。
- 引用了共享状态——使用`std::async`启动的未延迟任务建立的那个——的最后一个*future*的析构函数会阻塞住，直到任务完成。



## 条款三十九：对于一次性事件通信考虑使用void的futures

有时，一个任务通知另一个异步执行的任务发生了特定的事件很有用，因为第二个任务要等到这个事件发生之后才能继续执行。事件也许是一个数据结构已经初始化，也许是计算阶段已经完成，或者检测到重要的传感器值。一个明显的方案就是使用条件变量。如果我们将检测条件的任务称为**检测任务**（*detecting task*），对条件作出反应的任务称为**反应任务**（*reacting task*），策略很简单：反应任务等待一个条件变量，检测任务在事件发生时改变条件变量。代码如下：

```cpp
std::condition_variable cv;         //事件的条件变量
std::mutex m;                       //配合cv使用的mutex
```

检测任务中的代码：

```cpp
…                                   //检测事件
cv.notify_one();                    //通知反应任务
```

如果有多个反应任务需要被通知，使用`notify_all`代替`notify_one`。

反应任务的代码稍微复杂一点，因为在对条件变量调用`wait`之前，必须通过`std::unique_lock`对象锁住互斥锁。（在等待条件变量前锁住互斥锁对线程库来说是经典操作。通过`std::unique_lock`锁住互斥锁的需要仅仅是C++11 API要求的一部分。）概念上的代码如下：

```cpp
…                                       //反应的准备工作
{                                       //开启关键部分

    std::unique_lock<std::mutex> lk(m); //锁住互斥锁

    cv.wait(lk);                        //等待通知，但是这是错的！
    
    …                                   //对事件进行反应（m已经上锁）
}                                       //关闭关键部分；通过lk的析构函数解锁m
…                                       //继续反应动作（m现在未上锁）
```

这份代码的第一个问题就是有时被称为**代码异味**（*code smell*）的一种情况：即使代码正常工作，但是有些事情也不是很正确。在这个情况中，这种问题源自于使用互斥锁。互斥锁被用于保护共享数据的访问，但是可能检测任务和反应任务可能不会同时访问共享数据，比如说，检测任务会初始化一个全局数据结构，然后给反应任务用，如果检测任务在初始化之后不会再访问这个数据结构，而在检测任务表明数据结构准备完了之前反应任务不会访问这个数据结构，这两个任务在程序逻辑下互不干扰，也就没有必要使用互斥锁。但是条件变量的方法必须使用互斥锁，这就留下了令人不适的设计。

- **如果在反应任务`wait`之前检测任务通知了条件变量，反应任务会挂起**。为了能使条件变量唤醒另一个任务，任务必须等待在条件变量上。如果在反应任务`wait`之前检测任务就通知了条件变量，反应任务就会丢失这次通知，永远不被唤醒。

- **`wait`语句虚假唤醒**。线程API的存在一个事实（在许多语言中——不只是C++），等待一个条件变量的代码即使在条件变量没有被通知时，也可能被唤醒，这种唤醒被称为**虚假唤醒**（*spurious wakeups*）。正确的代码通过确认要等待的条件确实已经发生来处理这种情况，并将这个操作作为唤醒后的第一个操作。C++条件变量的API使得这种问题很容易解决，因为允许把一个测试要等待的条件的*lambda*（或者其他函数对象）传给`wait`。因此，可以将反应任务`wait`调用这样写：

  ```cpp
  cv.wait(lk, 
          []{ return whether the evet has occurred; });
  ```

  要利用这个能力需要反应任务能够确定其等待的条件是否为真。但是我们考虑的场景中，它正在等待的条件是检测线程负责识别的事件的发生情况。反应线程可能无法确定等待的事件是否已发生。这就是为什么它在等待一个条件变量！

**共享的布尔型flag:**

flag被初始化为`false`。当检测线程识别到发生的事件，将flag置位：

```cpp
std::atomic<bool> flag(false);          //共享的flag；std::atomic见条款40
…                                       //检测某个事件
flag = true;                            //告诉反应线程
```

就其本身而言，反应线程轮询该flag。当发现flag被置位，它就知道等待的事件已经发生了：

```cpp
…                                       //准备作出反应
while (!flag);                          //等待事件
…                                       //对事件作出反应
```

这种方法不存在基于条件变量的设计的缺点。不需要互斥锁，在反应任务开始轮询之前检测任务就对flag置位也不会出现问题，并且不会出现虚假唤醒。

缺点：

flag被初始化为`false`。当检测线程识别到发生的事件，将flag置位：

```cpp
std::atomic<bool> flag(false);          //共享的flag；std::atomic见条款40
…                                       //检测某个事件
flag = true;                            //告诉反应线程
```

就其本身而言，反应线程轮询该flag。当发现flag被置位，它就知道等待的事件已经发生了：

```cpp
…                                       //准备作出反应
while (!flag);                          //等待事件
…                                       //对事件作出反应
```

这种方法不存在基于条件变量的设计的缺点。不需要互斥锁，在反应任务开始轮询之前检测任务就对flag置位也不会出现问题，并且不会出现虚假唤醒。

**将条件变量和flag的设计组合：**

一个flag表示是否发生了感兴趣的事件，但是通过互斥锁同步了对该flag的访问。因为互斥锁阻止并发访问该flag，所以如Item40所述，不需要将flag设置为`std::atomic`。一个简单的`bool`类型就可以，检测任务代码如下：

```cpp
std::condition_variable cv;             //跟之前一样
std::mutex m;
bool flag(false);                       //不是std::atomic
…                                       //检测某个事件
{
    std::lock_guard<std::mutex> g(m);   //通过g的构造函数锁住m
    flag = true;                        //通知反应任务（第1部分）
}                                       //通过g的析构函数解锁m
cv.notify_one();                        //通知反应任务（第2部分）
```

反应任务代码如下:

```cpp
…                                       //准备作出反应
{                                       //跟之前一样
    std::unique_lock<std::mutex> lk(m); //跟之前一样
    cv.wait(lk, [] { return flag; });   //使用lambda来避免虚假唤醒
    …                                   //对事件作出反应（m被锁定）
}
…                                       //继续反应动作（m现在解锁）
```

无论在检测线程对条件变量发出通知之前反应线程是否调用了`wait`都可以工作，即使出现了虚假唤醒也可以工作，而且不需要轮询。但是仍然有些古怪，因为检测任务通过奇怪的方式与反应线程通信。检测任务通过通知条件变量告诉反应线程，等待的事件可能发生了，但是反应线程必须通过检查flag来确保事件发生了。检测线程置位flag来告诉反应线程事件确实发生了，但是检测线程仍然还要先需要通知条件变量，以唤醒反应线程来检查flag。这种方案是可以工作的，但是不太优雅。

**让反应任务通过在检测任务设置的*future*上`wait`来避免使用条件变量，互斥锁和flag：**

利用发送端是个`std::promise`，接收端是个*future*的通信信道。检测任务有一个`std::promise`对象（即通信信道的写入端），反应任务有对应的*future*。当检测任务看到事件已经发生，设置`std::promise`对象（即写入到通信信道）。同时，`wait`会阻塞住反应任务直到`std::promise`被设置。现在，`std::promise`和*futures*（即`std::future`和`std::shared_future`）都是需要类型参数的模板。形参表明通过通信信道被传递的信息的类型。在这里，没有数据被传递，只需要让反应任务知道它的*future*已经被设置了。我们在`std::promise`和*future*模板中需要的东西是表明通信信道中没有数据被传递的一个类型。这个类型就是`void`。检测任务使用`std::promise<void>`，反应任务使用`std::future<void>`或者`std::shared_future<void>`。当感兴趣的事件发生时，检测任务设置`std::promise<void>`，反应任务在*future*上`wait`。尽管反应任务不从检测任务那里接收任何数据，通信信道也可以让反应任务知道，检测任务什么时候已经通过对`std::promise<void>`调用`set_value`“写入”了`void`数据。

所以，有

```cpp
std::promise<void> p;                   //通信信道的promise
```

检测任务代码很简洁：

```cpp
…                                       //检测某个事件
p.set_value();                          //通知反应任务
```

反应任务代码也同样简单：

```cpp
…                                       //准备作出反应
p.get_future().wait();                  //等待对应于p的那个future
…                                       //对事件作出反应
```

像使用flag的方法一样，此设计不需要互斥锁，无论在反应线程调用`wait`之前检测线程是否设置了`std::promise`都可以工作，并且不受虚假唤醒的影响（只有条件变量才容易受到此影响）。与基于条件变量的方法一样，反应任务在调用`wait`之后是真被阻塞住的，不会一直占用系统资源。

缺点：Item38中说明，`std::promise`和*future*之间有个共享状态，并且共享状态是动态分配的。因此你应该假定此设计会产生基于堆的分配和释放开销。也许更重要的是，`std::promise`只能设置一次。`std::promise`和*future*之间的通信是**一次性**的：不能重复使用。这是与基于条件变量或者基于flag的设计的明显差异，条件变量和flag都可以通信多次。（条件变量可以被重复通知，flag也可以重复清除和设置。）

一次通信可能没有你想象中那么大的限制。假定你想创建一个挂起的系统线程。就是，你想避免多次线程创建的那种经常开销，以便想要使用这个线程执行程序时，避免潜在的线程创建工作。或者你想创建一个挂起的线程，以便在线程运行前对其进行设置这样的设置包括优先级或者核心亲和性（*core affinity*）。C++并发API没有提供这种设置能力，但是`std::thread`提供了`native_handle`成员函数，它的结果就是提供给你对平台原始线程API的访问（通常是POSIX或者Windows的线程）。这些低层次的API使你可以对线程设置优先级和亲和性。

假设你仅仅想要对某线程挂起一次（在创建后，运行线程函数前），使用`void`的*future*就是一个可行方案。这是这个技术的关键点：

```cpp
std::promise<void> p;
void react();                           //反应任务的函数
void detect()                           //检测任务的函数
{
    std::thread t([]                    //创建线程
                  {
                      p.get_future().wait();    //挂起t直到future被置位
                      react();
                  });
    …                                   //这里，t在调用react前挂起
    p.set_value();                      //解除挂起t（因此调用react）
    …                                   //做其他工作
    t.join();                           //使t不可结合（见条款37）
}
```

因为所有离开`detect`的路径中`t`都要是不可结合的，所以使用类似于[Item37](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item37.html)中`ThreadRAII`的RAII类很明智。代码如下：

```cpp
void detect()
{
    ThreadRAII tr(                      //使用RAII对象
        std::thread([]
                    {
                        p.get_future().wait();
                        react();
                    }),
        ThreadRAII::DtorAction::join    //有危险！（见下）
    );
    …                                   //tr中的线程在这里被挂起
    p.set_value();                      //解除挂起tr中的线程
    …
}
```

这样看起来安全多了。问题在于第一个“…”区域中（注释了“`tr`中的线程在这里被挂起”的那句），如果异常发生，`p`上的`set_value`永远不会调用，这意味着*lambda*中的`wait`永远不会返回。那意味着在*lambda*中运行的线程不会结束，这是个问题，因为RAII对象`tr`在析构函数中被设置为在（`tr`中创建的）那个线程上实行`join`。换句话说，如果在第一个“…”区域中发生了异常，函数挂起，因为`tr`的析构函数永远无法完成。（有很多方案解决这个问题，自行解决）这里，只想展示如何扩展原始代码（即不使用RAII类）使其挂起然后取消挂起不仅一个反应任务，而是多个任务。简单概括，关键就是在`react`的代码中使用`std::shared_future`代替`std::future`。一旦你知道`std::future`的`share`成员函数将共享状态所有权转移到`share`产生的`std::shared_future`中，代码自然就写出来了。唯一需要注意的是，每个反应线程都需要自己的`std::shared_future`副本，该副本引用共享状态，因此通过`share`获得的`shared_future`要被在反应线程中运行的*lambda*按值捕获：

```cpp
std::promise<void> p;                   //跟之前一样
void detect()                           //现在针对多个反映线程
{
    auto sf = p.get_future().share();   //sf的类型是std::shared_future<void>
    std::vector<std::thread> vt;        //反应线程容器
    for (int i = 0; i < threadsToRun; ++i) {
        vt.emplace_back([sf]{ sf.wait();    //在sf的局部副本上wait；
                              react(); });  //emplace_back见条款42
    }
    …                                   //如果这个“…”抛出异常，detect挂起！
    p.set_value();                      //所有线程解除挂起
    …
    for (auto& t : vt) {                //使所有线程不可结合；
        t.join();                       //“auto&”见条款2
    }
}
```



**请记住：**

- 对于简单的事件通信，基于条件变量的设计需要一个多余的互斥锁，对检测和反应任务的相对进度有约束，并且需要反应任务来验证事件是否已发生。
- 基于flag的设计避免的上一条的问题，但是是基于轮询，而不是阻塞。
- 条件变量和flag可以组合使用，但是产生的通信机制很不自然。
- 使用`std::promise`和*future*的方案避开了这些问题，但是这个方法使用了堆内存存储共享状态，同时有只能使用一次通信的限制。

## 条款四十：对于并发使用std::atomic,对于特殊内存使用volatile

**并发使用std::atomic:**

**原子操作：**

如下使用`std::atmoic`的代码：

```cpp
std::atomic<int> ai(0);         //初始化ai为0
ai = 10;                        //原子性地设置ai为10
std::cout << ai;                //原子性地读取ai的值
++ai;                           //原子性地递增ai到11
--ai;                           //原子性地递减ai到10
```

在这些语句执行过程中，其他线程读取`ai`，只能读取到0，10，11三个值其中一个。没有其他可能（当然，假设只有这个线程会修改`ai`）。这个例子中有两点值得注意：

首先，在“`std::cout << ai;`”中，`ai`是一个`std::atomic`的事实只保证了对`ai`的读取是原子的。没有保证整个语句的执行是原子的。在读取`ai`的时刻与调用`operator<<`将值写入到标准输出之间，另一个线程可能会修改`ai`的值。这对于这个语句没有影响，因为`int`的`operator<<`是使用`int`型的传值形参来输出（所以输出的值就是读取到的`ai`的值），但是重要的是要理解原子性的范围只保证了读取`ai`是原子性的。

第二点值得注意的是最后两条语句——关于`ai`的递增递减。他们都是读-改-写（read-modify-write，RMW）操作，它们整体作为原子执行。这是`std::atomic`类型的最优的特性之一：一旦`std::atomic`对象被构建，所有成员函数，包括RMW操作，从其他线程来看都是原子性的。

相反，使用`volatile`在多线程中实际上不保证任何事情：

```cpp
volatile int vi(0);             //初始化vi为0
vi = 10;                        //设置vi为10 
std::cout << vi;                //读vi的值
++vi;                           //递增vi到11
--vi;                           //递减vi到10
```

代码的执行过程中，如果其他线程读取`vi`，可能读到任何值，比如-12，68，4090727——任何值！这份代码有未定义行为，因为这里的语句修改`vi`，所以如果同时其他线程读取`vi`，同时存在多个readers和writers读取没有`std::atomic`或者互斥锁保护的内存，这就是数据竞争的定义。

**禁止编译器重排序：**

代码如下：

```cpp
std::atomic<bool> valVailable(false); 
auto imptValue = computeImportantValue();   //计算值
valAvailable = true;                        //告诉另一个任务，值可用了
```

人类读这份代码，能看到在`valAvailable`赋值之前对`imptValue`赋值很关键，但是所有编译器看到的是给相互独立的变量的一对赋值操作。通常来说，编译器会被允许重排这对没有关联的操作。这意味着，给定如下顺序的赋值操作（其中`a`，`b`，`x`，`y`都是互相独立的变量），

```cpp
a = b;
x = y;
```

编译器可能重排为如下顺序：

```cpp
x = y;
a = b;
```

即使编译器没有重排顺序，底层硬件也可能重排（或者可能使它看起来运行在其他核心上），因为有时这样代码执行更快。然而，`std::atomic`会限制这种重排序，并且这样的限制之一是，在对 `std::atomic` 变量进行写操作之前，所有前面的操作都会被执行完，编译器和处理器不能将这些操作重排序到原子写操作之后。

这意味对我们的代码，

```cpp
auto imptValue = computeImportantValue();   //计算值
valAvailable = true;                        //告诉另一个任务，值可用了
```

编译器不仅要保证`imptValue`和`valAvailable`的赋值顺序，还要保证生成的硬件代码不会改变这个顺序。结果就是，将`valAvailable`声明为`std::atomic`确保了必要的顺序——其他线程看到的是`imptValue`值的改变不会晚于`valAvailable`。

将`valAvailable`声明为`volatile`不能保证上述顺序：

```cpp
volatile bool valVailable(false); 
auto imptValue = computeImportantValue();
valAvailable = true;                        //其他线程可能看到这个赋值操作早于imptValue的赋值操作
```

这份代码编译器可能将`imptValue`和`valAvailable`赋值顺序对调，如果它们没这么做，可能不能生成机器代码，来阻止底部硬件在其他核心上的代码看到`valAvailable`更改在`imptValue`之前。

**特殊内存使用volatile:**

“正常”内存应该有这个特性，在写入值之后，这个值会一直保持直到被覆盖。假设有这样一个正常的`int`

```cpp
int x;
```

编译器看到下列的操作序列：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x
```

编译器可通过忽略对`y`的一次赋值来优化代码，因为有了`y`初始化，赋值是冗余的。

正常内存还有一个特征，就是如果你写入内存没有读取，再次写入，第一次写就可以被忽略，因为值没被用过。给出下面的代码：

```cpp
x = 10;                                 //写x
x = 20;                                 //再次写x
```

编译器可以忽略第一次写入。这意味着如果写在一起：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x
x = 10;                                 //写x
x = 20;                                 //再次写x
```

编译器生成的代码是这样的：

```cpp
auto y = x;                             //读x
x = 20;                                 //写x
```

可能你会想谁会写这种重复读写的代码（技术上称为**冗余访问**（*redundant loads*）和**无用存储**（*dead stores*）），答案是开发者不会直接写——至少我们不希望开发者这样写。但是在编译器拿到看起来合理的代码，执行了模板实例化，内联和一系列重排序优化之后，结果会出现冗余访问和无用存储，所以编译器需要摆脱这样的情况并不少见。

这种优化仅仅在内存表现正常时有效。“特殊”的内存不行。最常见的“特殊”内存是用来做内存映射I/O的内存。这种内存实际上是与外围设备（比如外部传感器或者显示器，打印机，网络端口）通信，而不是读写通常的内存（比如RAM）。这种情况下，再次考虑这看起来冗余的代码：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x
```

如果`x`的值是一个温度传感器上报的，第二次对于`x`的读取就不是多余的，因为温度可能在第一次和第二次读取之间变化。

看起来冗余的写操作也类似。比如在这段代码中：

```cpp
x = 10;                                 //写x
x = 20;                                 //再次写x
```

如果`x`与无线电发射器的控制端口关联，则代码是给无线电发指令，10和20意味着不同的指令。优化掉第一条赋值会改变发送到无线电的指令流。

`volatile`是告诉编译器我们正在处理特殊内存。意味着告诉编译器“不要对这块内存执行任何优化”。所以如果`x`对应于特殊内存，应该声明为`volatile`：

```cpp
volatile int x;
```

考虑对我们的原始代码序列有何影响：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x（不会被优化掉）

x = 10;                                 //写x（不会被优化掉）
x = 20;                                 //再次写x
```

如果`x`是内存映射的（或者已经映射到跨进程共享的内存位置等），这正是想要的。(`y`的类型使用`auto`类型推导，所以使用Item2中的规则。规则上说非引用非指针类型的声明（就是`y`的情况），`const`和`volatile`限定符被拿掉。`y`的类型因此仅仅是`int`。这意味着对`y`的冗余读取和写入可以被消除。在例子中，编译器必须执行对`y`的初始化和赋值两个语句，因为`x`是`volatile`的，所以第二次对`x`的读取可能会产生一个与第一次不同的值。)

编译器被允许消除对`std::atomic`的冗余操作。代码的编写方式与`volatile`那些不那么相同，但是如果我们暂时忽略它，只关注编译器执行的操作，则概念上可以说，编译器看到这个，

```cpp
std::atomic<int> x;
auto y = x;                             //概念上会读x（见下）
y = x;                                  //概念上会再次读x（见下）

x = 10;                                 //写x
x = 20;                                 //再次写x
```

会优化为：

```cpp
auto y = x;                             //概念上会读x（见下）
x = 20;                                 //写x
```

对于特殊内存，显然这是不可接受的。

当`x`是`std::atomic`时，这两条语句都无法编译通过：

```cpp
auto y = x;                             //错误
y = x;                                  //错误
```

这是因为`std::atomic`类型的拷贝操作是被删除的（参见Item11）。因为有个很好的理由删除。想象一下如果`y`使用`x`来初始化会发生什么。因为`x`是`std::atomic`类型，`y`的类型被推导为`std::atomic`（参见Item2。我之前说了`std::atomic`最好的特性之一就是所有成员函数都是原子性的，但是为了使从`x`拷贝初始化`y`的过程是原子性的，编译器不得不生成代码，把读取`x`和写入`y`放在一个单独的原子性操作中。硬件通常无法做到这一点，因此`std::atomic`不支持拷贝构造。出于同样的原因，拷贝赋值也被删除了，这也是为什么从`x`赋值给`y`也编译失败。（移动操作在`std::atomic`没有显式声明，因此根据Item17中描述的规则来看，`std::atomic`不支持移动构造和移动赋值）。

可以将`x`的值传递给`y`，但是需要使用`std::atomic`的`load`和`store`成员函数。`load`函数原子性地读取，`store`原子性地写入。要使用`x`初始化`y`，然后将`x`的值放入`y`，代码应该这样写：

```cpp
std::atomic<int> y(x.load());           //读x
y.store(x.load());                      //再次读x
```

这可以编译，读取`x`（通过`x.load()`）是与初始化或者存储到`y`相独立的函数，这个事实清楚地表明没理由期待上面的任何一个语句会在单独的原子性的操作中整体执行。

给出上面的代码，编译器可以通过存储x的值到寄存器代替读取两次来“优化”：

```cpp
register = x.load();                    //把x读到寄存器
std::atomic<int> y(register);           //使用寄存器值初始化y
y.store(register);                      //把寄存器值存储到y
```

结果如你所见，仅读取`x`一次，这是对于特殊内存必须避免的优化。（这种优化是不允许对`volatile`类型变量执行的。）

**结合使用：**

```cpp
volatile std::atomic<int> vai;          //对vai的操作是原子性的，且不能被优化掉
```

如果`vai`变量关联了内存映射I/O的位置，被多个线程并发访问，这会很有用。

最后一点，一些开发者在即使不必要时也尤其喜欢使用`std::atomic`的`load`和`store`函数，因为这在代码中显式表明了这个变量不“正常”。强调这一事实并非没有道理。因为访问`std::atomic`确实会比non-`std::atomic`更慢一些，我们也看到了`std::atomic`会阻止编译器对代码执行一些特定的，本应被允许的顺序重排。调用`load`和`store`可以帮助识别潜在的可扩展性瓶颈。从正确性的角度来看，**没有**看到在一个变量上调用`store`来与其他线程进行通信（比如用个flag表示数据的可用性）可能意味着该变量在声明时本应使用而没有使用`std::atomic`。



**请记住：**

- `std::atomic`用于在不使用互斥锁情况下，来使变量被多个线程访问的情况。是用来编写并发程序的一个工具。
- `volatile`用在读取和写入不应被优化掉的内存上。是用来处理特殊内存的一个工具。



## **条款四十一：对于移动成本低且总是被拷贝的可拷贝形参，考虑按值传递**

有些函数的形参是可拷贝的。（在本条款中，“拷贝”一个形参通常意思是，将形参作为拷贝或移动操作的源对象。）比如说，`addName`成员函数可以拷贝自己的形参到一个私有容器。为了提高效率，应该拷贝左值，移动右值。

**函数重载：**

```cpp
class Widget {
public:
    void addName(const std::string& newName)    //接受左值；拷贝它
    { names.push_back(newName); }

    void addName(std::string&& newName)         //接受右值；移动它；std::move
    { names.push_back(std::move(newName)); }    //的使用见条款25
    …
private:
    std::vector<std::string> names;
};
```

这是可行的，但是需要编写两个函数来做本质相同的事。但是存在两个函数声明，两个函数实现，两个函数写说明，两个函数的维护。此外，目标代码中会有两个函数——你可能会担心程序的空间占用。这种情况下，两个函数都可能内联，可能会避免同时两个函数同时存在导致的代码膨胀问题，但是一旦没有被内联，目标代码就会出现两个函数。

**具有通用引用的函数模板：**

```cpp
class Widget {
public:
    template<typename T>                            //接受左值和右值；
    void addName(T&& newName) {                     //拷贝左值，移动右值；
        names.push_back(std::forward<T>(newName));  //std::forward的使用见条款25
    }
    …
};
```

这减少了源代码的维护工作，但是通用引用会导致其他复杂性。作为模板，`addName`的实现必须放置在头文件中。在编译器展开的时候，可能会产生多个函数，因为不止为左值和右值实例化，也可能为`std::string`和可转换为`std::string`的类型分别实例化为多个函数（参考Item25）。同时有些实参类型不能通过通用引用传递（参考Item30），而且如果传递了不合法的实参类型，编译器错误会令人生畏（参考Item27）。

**按值传递形参：**

```cpp
class Widget {
public:
    void addName(std::string newName) {         //接受左值或右值；移动它
        names.push_back(std::move(newName));
    }
    …
}
```

使用`std::move`后原对象可能变得不再可用，但是在这里，（1）`newName`是，与调用者传进来的对象相完全独立的，一个对象，所以改变`newName`不会影响到调用者；（2）这就是`newName`的最后一次使用，所以移动它不会影响函数内此后的其他代码。

按值传递在C++98中，可以肯定确实开销会大。无论调用者传入什么，形参`newName`都是由拷贝构造出来。但是在C++11中，只有在左值实参情况下，`addName`被拷贝构造出来；对于右值，它会被移动构造。来看如下例子：

```cpp
Widget w;
…
std::string name("Bart");
w.addName(name);            //使用左值调用addName
…
w.addName(name + "Jenne");  //使用右值调用addName（见下）
```

第一处调用`addName`（当`name`被传递时），形参`newName`是使用左值被初始化。`newName`因此是被拷贝构造，就像在C++98中一样。第二处调用，`newName`使用`std::string`对象被初始化，这个`std::string`对象是调用`std::string`的`operator+`（即*append*操作）得到的结果。这个对象是一个右值，因此`newName`是被移动构造的。

**分析上面三种方法：**

将前两个版本称为“按引用方法”，因为都是通过引用传递形参。

仍然考虑这两种调用方式：

```cpp
Widget w;
…
std::string name("Bart");
w.addName(name);                                //传左值
…
w.addName(name + "Jenne");                      //传右值
```

现在分别考虑三种实现中，给`Widget`添加一个名字的两种调用方式，拷贝和移动操作的开销。会忽略编译器对于移动和拷贝操作的优化，因为这些优化是与上下文和编译器有关的，实际上不会改变我们分析的本质。

- **重载**：无论传递左值还是传递右值，调用都会绑定到一个叫`newName`的引用上。从拷贝和移动操作方面看，这个过程零开销。左值重载中，`newName`拷贝到`Widget::names`中，右值重载中，移动进去。开销总结：左值一次拷贝，右值一次移动。

- **使用通用引用**：同重载一样，调用也绑定到`addName`这个引用上，没有开销。由于使用了`std::forward`，左值`std::string`实参会拷贝到`Widget::names`，右值`std::string`实参移动进去。对`std::string`实参来说，开销同重载方式一样：左值一次拷贝，右值一次移动。

  Item25 解释了如果调用者传递的实参不是`std::string`类型，将会转发到`std::string`的构造函数，几乎也没有`std::string`拷贝或者移动操作。因此通用引用的方式有同样效率，所以这不影响本次分析，简单假定调用者总是传入`std::string`类型实参即可。

- **按值传递**：无论传递左值还是右值，都必须构造`newName`形参。如果传递的是左值，需要拷贝的开销，如果传递的是右值，需要移动的开销。在函数的实现中，`newName`总是采用移动的方式到`Widget::names`。开销总结：左值实参，一次拷贝一次移动，右值实参两次移动。对比按引用传递的方法，对于左值或者右值，均多出一次移动操作。

**对于移动成本低且总是被拷贝的可拷贝形参，考虑按值传递：**

四个原因：

1. 应该仅**考虑**使用传值方式。确实，只需要编写一个函数。确实，只会在目标代码中生成一个函数。确实，避免了通用引用的种种问题。但是毕竟开销会比那些替代方式更高（译者注：指接受引用的两种实现方式），而且下面还会讨论，还会存在一些目前我们并未讨论到的开销。

2. 仅考虑对于**可拷贝形参**使用按值传递。不符合此条件的的形参必须有只可移动的类型（*move-only types*）（的数据成员），因为函数总是会做副本（译注：指的是传值时形参总是实参的一个副本），而如果它们不可拷贝，副本就必须通过移动构造函数创建。（这样的句子就说明有一个术语来区分拷贝操作制作的副本，和移动操作制作的副本，是非常好的。）回忆一下传值方案比“重载”方案的优势在于，仅有一个函数要写。但是对于只可移动类型，没必要为左值实参提供重载，因为拷贝左值需要拷贝构造函数，只可移动类型的拷贝构造函数是禁用的。那意味着只需要支持右值实参，“重载”方案只需要一个重载函数：接受右值引用的函数。

   考虑一个类：有个`std::unique_ptr<std::string>`的数据成员，对它有个赋值器（*setter*）。`std::unique_ptr`是只可移动的类型，所以赋值器的“重载”方式只有一个函数：

   ```cpp
   class Widget {
   public:
       …
       void setPtr(std::unique_ptr<std::string>&& ptr)
       { p = std::move(ptr); }
   
   private:
       std::unique_ptr<std::string> p;
   };
   ```

   调用者可能会这样写：

   ```cpp
   Widget w;
   …
   w.setPtr(std::make_unique<std::string>("Modern C++"));
   ```

   这样，从`std::make_unique`返回的右值`std::unique_ptr<std::string>`通过右值引用被传给`setPtr`，然后移动到数据成员`p`中。整体开销就是一次移动。

   如果`setPtr`使用传值方式接受形参：

   ```cpp
   class Widget {
   public:
       …
       void setPtr(std::unique_ptr<std::string> ptr)
       { p = std::move(ptr); }
       …
   };
   ```

   同样的调用就会先移动构造`ptr`形参，然后`ptr`再移动赋值到数据成员`p`，整体开销就是两次移动——是“重载”方法开销的两倍。

3. 按值传递应该仅考虑那些**移动开销小**的形参。当移动的开销较低，额外的一次移动才能被开发者接受，但是当移动的开销很大，执行不必要的移动就类似执行一个不必要的拷贝，而避免不必要的拷贝的重要性就是最开始C++98规则中避免传值的原因！

4. 你应该只对**总是被拷贝**的形参考虑按值传递。为了看清楚为什么这很重要，假定在拷贝形参到`names`容器前，`addName`需要检查新的名字的长度是否过长或者过短，如果是，就忽略增加名字的操作：

   ```cpp
   class Widget {
   public:
       void addName(std::string newName)
       {
           if ((newName.length() >= minLen) &&
               (newName.length() <= maxLen))
           {
               names.push_back(std::move(newName));
           }
       }
       …
   private:
    std::vector<std::string> names;
   };
   ```

   即使这个函数没有在`names`添加任何内容，也增加了构造和销毁`newName`的开销，而按引用传递会避免这笔开销。

**通过构造拷贝形参可能比通过赋值拷贝形参开销大的多：**

即使你编写的函数对可拷贝类型执行无条件的复制，且这个类型移动开销小，有时也可能不适合按值传递。这是因为函数拷贝一个形参存在两种方式：一种是通过构造函数（拷贝构造或者移动构造），还有一种是赋值（拷贝赋值或者移动赋值）。`addName`使用构造函数，它的形参`newName`传递给`vector::push_back`，在这个函数内部，`newName`是通过拷贝构造在`std::vector`末尾创建一个新元素。对于使用构造函数拷贝形参的函数，之前的分析已经可以给出最终结论：按值传递对于左值和右值均增加了一次移动操作的开销。当形参通过赋值操作进行拷贝时，分析起来更加复杂。比如，有一个表征密码的类，因为密码可能会被修改，提供了赋值器函数`changeTo`。用按值传递的策略，实现一个`Password`类如下：

```cpp
class Password {
public:
    explicit Password(std::string pwd)      //传值
    : text(std::move(pwd)) {}               //构造text
    void changeTo(std::string newPwd)       //传值
    { text = std::move(newPwd); }           //赋值text
    …
private:
    std::string text;                       //密码的text
};
```

考虑这段代码：

```cpp
std::string initPwd("Supercalifragilisticexpialidocious");
Password p(initPwd);
```

毫无疑问：`p.text`被给定的密码构造，在构造函数用按值传递的方式增加了一次移动构造的开销，该程序的用户可能对这个密码不太满意，因为“Supercalifragilisticexpialidocious”可以在许多字典中找到。因此采取等价于如下代码的一些动作：

```cpp
std::string newPassword = "Beware the Jabberwock";
p.changeTo(newPassword);
```

传递给`changeTo`的实参是一个左值（`newPassword`），所以`newPwd`形参被构造时，`std::string`的拷贝构造函数会被调用，这个函数会分配新的存储空间给新密码。`newPwd`会移动赋值到`text`，这会导致`text`本来持有的内存被释放。所以`changeTo`存在两次动态内存管理的操作：一次是为新密码创建内存，一次是销毁旧密码的内存。但是在这个例子中，旧密码比新密码长度更长，所以不需要分配新内存，销毁旧内存的操作。如果使用重载的方式，有可能两次动态内存管理操作得以避免：

```cpp
class Password {
public:
    …
    void changeTo(std::string& newPwd)      //对左值的重载
    {
        text = newPwd;                      //如果text.capacity() >= newPwd.size()，
                                            //可能重用text的内存
    }
    …
private:
    std::string text;                       //同上
};
```

这种情况下，按值传递的开销包括了内存分配和内存销毁——可能会比`std::string`的移动操作高出几个数量级。如果旧密码短于新密码，在赋值过程中就不可避免要销毁、分配内存，这种情况，按值传递跟按引用传递的效率是一样的。因此，基于赋值的形参拷贝操作开销取决于具体的实参的值，这种分析适用于在动态分配内存中存值的形参类型。不是所有类型都满足，但是很多——包括`std::string`和`std::vector`——是这样。这种潜在的开销增加仅在传递左值实参时才适用，因为执行内存分配和释放通常发生在真正的拷贝操作（即，不是移动）中。对右值实参，移动几乎就足够了。

**结论：**

结论是，使用通过赋值拷贝一个形参进行按值传递的函数的额外开销，取决于传递的类型，左值和右值的比例，这个类型是否需要动态分配内存，以及，如果需要分配内存的话，赋值操作符的具体实现，还有赋值目标占的内存至少要跟赋值源占的内存一样大。对于`std::string`来说，开销还取决于实现是否使用了小字符串优化（SSO——参考[Item29](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item29.html)），如果是，那么要赋值的值是否匹配SSO缓冲区。

所以，当形参通过赋值进行拷贝时，分析按值传递的开销是复杂的。通常，最有效的经验就是“在证明没问题之前假设有问题”，就是除非已证明按值传递会为你需要的形参类型产生可接受的执行效率，否则使用重载或者通用引用的实现方式。对于需要运行尽可能快的软件来说，按值传递可能不是一个好策略，因为避免即使开销很小的移动操作也非常重要。此外，有时并不能清楚知道会发生多少次移动操作。在`Widget::addName`例子中，按值传递仅多了一次移动操作，但是如果`Widget::addName`调用了`Widget::validateName`，这个函数也是按值传递。（假定它有理由总是拷贝它的形参，比如把验证过的所有值存在一个数据结构中。）并假设`validateName`调用了第三个函数，也是按值传递……对于需要运行尽可能快的软件来说，按值传递可能不是一个好策略，因为避免即使开销很小的移动操作也非常重要。此外，有时并不能清楚知道会发生多少次移动操作。在`Widget::addName`例子中，按值传递仅多了一次移动操作，但是如果`Widget::addName`调用了`Widget::validateName`，这个函数也是按值传递。（假定它有理由总是拷贝它的形参，比如把验证过的所有值存在一个数据结构中。）并假设`validateName`调用了第三个函数，也是按值传递……在调用链中，每个函数都使用传值，因为“只多了一次移动的开销”，但是整个调用链总体就会产生无法忍受的开销，通过引用传递，调用链不会增加这种开销。

**按值传递存在切片问题：**

跟性能无关，但总是需要考虑的是，不像按引用传递，按值传递会受到切片问题的影响。这是C++98的事，在此不在详述，但是如果要设计一个函数，来处理这样的形参：基类**或者任何其派生类**，你肯定不想声明一个那个类型的传值形参，因为你会“切掉”传入的任意派生类对象的派生类特征：

```cpp
class Widget { … };                         //基类
class SpecialWidget: public Widget { … };   //派生类
void processWidget(Widget w);               //对任意类型的Widget的函数，包括派生类型
…                                           //遇到对象切片问题
SpecialWidget sw;
…
processWidget(sw);                          //processWidget看到的是Widget，
                                            //不是SpecialWidget！
```

切片问题是C++98中默认按值传递名声不好的另一个原因（要在效率问题的原因之上）。有充分的理由来说明为什么你学习C++编程的第一件事就是避免用户自定义类型进行按值传递。C++11没有从根本上改变C++98对于按值传递的智慧。通常，按值传递仍然会带来你希望避免的性能下降，而且按值传递会导致切片问题。C++11中新的功能是区分了左值和右值实参。利用对于可拷贝类型的右值的移动语义，需要重载或者通用引用，尽管两者都有其缺陷。对于特殊的场景，可拷贝且移动开销小的类型，传递给总是会拷贝他们的一个函数，并且切片也不需要考虑，这时，按值传递就提供了一种简单的实现方式，效率接近传递引用的函数，但是避免了传引用方案的缺点。



**请记住：**

- 对于可拷贝，移动开销低，而且无条件被拷贝的形参，按值传递效率基本与按引用传递效率一致，而且易于实现，还生成更少的目标代码。
- 通过构造拷贝形参可能比通过赋值拷贝形参开销大的多。
- 按值传递会引起切片问题，所说不适合基类形参类型。



## 条款四十二：考虑使用置入代替插入

`std::vector`的`push_back`被按左值和右值分别重载：

```cpp
template <class T,                  //来自C++11标准
          class Allocator = allocator<T>>
class vector {
public:
    …
    void push_back(const T& x);     //插入左值
    void push_back(T&& x);          //插入右值
    …
};
```

在

```cpp
vs.push_back("xyzzy");
```

这个调用中，编译器看到实参类型（`const char[6]`）和`push_back`采用的形参类型（`std::string`的引用）之间不匹配。它们通过从字符串字面量创建一个`std::string`类型的临时对象来消除不匹配，然后传递临时变量给`push_back`。换句话说，编译器处理的这个调用应该像这样：

```cpp
vs.push_back(std::string("xyzzy")); //创建临时std::string，把它传给push_back
```

代码可以编译并运行。下面是在`push_back`运行时发生了什么：

1. 一个`std::string`的临时对象从字面量“`xyzzy`”被创建。这个对象没有名字，我们可以称为`temp`。`temp`的构造是第一次`std::string`构造。因为是临时变量，所以`temp`是右值。
2. `temp`被传递给`push_back`的右值重载函数，绑定到右值引用形参`x`。在`std::vector`的内存中一个`x`的副本被创建。这次构造——也是第二次构造——在`std::vector`内部真正创建一个对象。（将`x`副本拷贝到`std::vector`内部的构造函数是移动构造函数，因为`x`在它被拷贝前被转换为一个右值，成为右值引用。有关将右值引用形参强制转换为右值的信息，请参见Item25）。
3. 在`push_back`返回之后，`temp`立刻被销毁，调用了一次`std::string`的析构函数。

**`emplace_back`：**

可以获取字符串字面量并将其直接传入到步骤2里在`std::vector`内构造`std::string`的代码中，可以避免临时对象`temp`的创建与销毁。没有临时变量会生成：

```cpp
vs.emplace_back("xyzzy");           //直接用“xyzzy”在vs内构造std::string
```

`emplace_back`使用完美转发，因此只要你没有遇到完美转发的限制（参见[Item30](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item30.html)），就可以传递任何实参以及组合到`emplace_back`。比如，如果你想通过接受一个字符和一个数量的`std::string`构造函数，在`vs`中创建一个`std::string`，代码如下：

```cpp
vs.emplace_back(50, 'x');           //插入由50个“x”组成的一个std::string
```

`emplace_back`可以用于每个支持`push_back`的标准容器。类似的，每个支持`push_front`的标准容器都支持`emplace_front`。每个支持`insert`（除了`std::forward_list`和`std::array`）的标准容器支持`emplace`。关联容器提供`emplace_hint`来补充接受“hint”迭代器的`insert`函数，`std::forward_list`有`emplace_after`来匹配`insert_after`。

使得置入（emplacement）函数功能优于插入函数的原因是它们有灵活的接口。插入函数接受**对象**去插入，而置入函数接受**对象的构造函数接受的实参**去插入。这种差异允许置入函数避免插入函数所必需的临时对象的创建和销毁。因为可以传递容器内元素类型的实参给置入函数（因此该实参使函数执行复制或者移动构造函数），所以在插入函数不会构造临时对象的情况，也可以使用置入函数。在这种情况下，插入和置入函数做的是同一件事，比如：

```cpp
std::string queenOfDisco("Donna Summer");
```

下面的调用都是可行的，对容器的实际效果也一样：

```cpp
vs.push_back(queenOfDisco);         //拷贝构造queenOfDisco
vs.emplace_back(queenOfDisco);      //同上
```

因此，置入函数可以完成插入函数的所有功能。并且有时效率更高，至少在理论上，不会更低效。

**为什么不在所有场合使用它们：**

因为，就像说的那样，只是“理论上”，在理论和实际上没有什么区别，但是实际上区别还是有的。在当前标准库的实现下，有些场景，就像预期的那样，置入执行性能优于插入，但是，有些场景反而插入更快。这种场景不容易描述，因为依赖于传递的实参的类型、使用的容器、置入或插入到容器中的位置、容器中类型的构造函数的异常安全性，和对于禁止重复值的容器（即`std::set`，`std::map`，`std::unordered_set`，`set::unordered_map`）要添加的值是否已经在容器中。因此，大致的调用建议是：通过benchmark测试来确定置入和插入哪种更快。还有一种启发式的方法来帮助你确定是否应该使用置入。如果下列条件都能满足，置入会优于插入：

- **值是通过构造函数添加到容器，而不是直接赋值。** 例子就像本条款刚开始的那样（用“`xyzzy`”添加`std::string`到`std::vector`容器`vs`中），值添加到`vs`末尾——一个先前没有对象存在的地方。新值必须通过构造函数添加到`std::vector`。如果我们回看这个例子，新值放到已经存在了对象的一个地方，那情况就完全不一样了。考虑下：

  ```cpp
  std::vector<std::string> vs;        //跟之前一样
  …                                   //添加元素到vs
  vs.emplace(vs.begin(), "xyzzy");    //添加“xyzzy”到vs头部
  ```

  对于这份代码，没有实现会在已经存在对象的位置`vs[0]`构造这个添加的`std::string`。而是，通过移动赋值的方式添加到需要的位置。但是移动赋值需要一个源对象，所以这意味着一个临时对象要被创建，而置入优于插入的原因就是没有临时对象的创建和销毁，所以当通过赋值操作添加元素时，置入的优势消失殆尽。

  而且，向容器添加元素是通过构造还是赋值通常取决于实现者。但是，启发式仍然是有帮助的。基于节点的容器实际上总是使用构造添加新元素，大多数标准库容器都是基于节点的。例外的容器只有`std::vector`，`std::deque`，`std::string`。（`std::array`也不是基于节点的，但是它不支持置入和插入，所以它与这儿无关。）在不是基于节点的容器中，你可以依靠`emplace_back`来使用构造向容器添加元素，对于`std::deque`，`emplace_front`也是一样的。

- **传递的实参类型与容器的初始化类型不同。** 再次强调，置入优于插入通常基于以下事实：当传递的实参不是容器保存的类型时，接口不需要创建和销毁临时对象。当将类型为`T`的对象添加到`container<T>`时，没有理由期望置入比插入运行的更快，因为不需要创建临时对象来满足插入的接口。

- **容器不拒绝重复项作为新值。** 这意味着容器要么允许添加重复值，要么你添加的元素大部分都是不重复的。这样要求的原因是为了判断一个元素是否已经存在于容器中，置入实现通常会创建一个具有新值的节点，以便可以将该节点的值与现有容器中节点的值进行比较。如果要添加的值不在容器中，则链接该节点。然后，如果值已经存在，置入操作取消，创建的节点被销毁，意味着构造和析构时的开销被浪费了。这样的节点更多的是为置入函数而创建，相比起为插入函数来说。

**注意资源管理：**

假定你有一个盛放`std::shared_ptr<Widget>`s的容器，

```cpp
std::list<std::shared_ptr<Widget>> ptrs;
```

然后你想添加一个通过自定义删除器释放的`std::shared_ptr`（参见[Item19](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item19.html)）。[Item21](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item21.html)说明你应该使用`std::make_shared`来创建`std::shared_ptr`，但是它也承认有时你无法做到这一点。比如当你要指定一个自定义删除器时。这时，你必须直接`new`一个原始指针，然后通过`std::shared_ptr`来管理。

如果自定义删除器是这个函数，

```cpp
void killWidget(Widget* pWidget);
```

使用插入函数的代码如下：

```cpp
ptrs.push_back(std::shared_ptr<Widget>(new Widget, killWidget));
```

也可以像这样：

```cpp
ptrs.push_back({new Widget, killWidget});
```

不管哪种写法，在调用`push_back`前会生成一个临时`std::shared_ptr`对象。`push_back`的形参是`std::shared_ptr`的引用，因此必须有一个`std::shared_ptr`。

用`emplace_back`应该可以避免`std::shared_ptr`临时对象的创建，但是在这个场景下，临时对象值得被创建。考虑如下可能的时间序列：

1. 在上述的调用中，一个`std::shared_ptr<Widget>`的临时对象被创建来持有“`new Widget`”返回的原始指针。称这个对象为`temp`。
2. `push_back`通过引用接受`temp`。在存储`temp`的副本的*list*节点的内存分配过程中，内存溢出异常被抛出。
3. 随着异常从`push_back`的传播，`temp`被销毁。作为唯一管理这个`Widget`的`std::shared_ptr`，它自动销毁`Widget`，在这里就是调用`killWidget`。

这样的话，即使发生了异常，没有资源泄漏：在调用`push_back`中通过“`new Widget`”创建的`Widget`在`std::shared_ptr`管理下自动销毁。生命周期良好。

考虑使用`emplace_back`代替`push_back`：

```cpp
ptrs.emplace_back(new Widget, killWidget);
```

1. 通过`new Widget`创建的原始指针完美转发给`emplace_back`中，*list*节点被分配的位置。如果分配失败，还是抛出内存溢出异常。
2. 当异常从`emplace_back`传播，原始指针是仅有的访问堆上`Widget`的途径，但是因为异常而丢失了，那个`Widget`的资源（以及任何它所拥有的资源）发生了泄漏。

在这个场景中，生命周期不良好，这个失误不能赖`std::shared_ptr`。使用带自定义删除器的`std::unique_ptr`也会有同样的问题。根本上讲，像`std::shared_ptr`和`std::unique_ptr`这样的资源管理类的高效性是以资源（比如从`new`来的原始指针）被**立即**传递给资源管理对象的构造函数为条件的。实际上，`std::make_shared`和`std::make_unique`这样的函数自动做了这些事，是使它们如此重要的原因。在对存储资源管理类对象的容器（比如`std::list<std::shared_ptr<Widget>>`）调用插入函数时，函数的形参类型通常确保在资源的获取（比如使用`new`）和资源管理对象的创建之间没有其他操作。在置入函数中，完美转发推迟了资源管理对象的创建，直到可以在容器的内存中构造它们为止，这给“异常导致资源泄漏”提供了可能。所有的标准库容器都容易受到这个问题的影响。在使用资源管理对象的容器时，必须注意确保在使用置入函数而不是插入函数时，不会为提高效率带来的降低异常安全性付出代价。

无论如何，你不应该将“`new Widget`”之类的表达式传递给`emplace_back`或者`push_back`或者大多数这种函数，因为，就像Item21中解释的那样，这可能导致我们刚刚讨论的异常安全性问题。消除资源泄漏可能性的方法是，使用独立语句把从“`new Widget`”获取的指针传递给资源管理类对象，然后这个对象作为右值传递给你本来想传递“`new Widget`”的函数（Item21有这个观点的详细讨论）。使用`push_back`的代码应该如下：

```cpp
std::shared_ptr<Widget> spw(new Widget,      //创建Widget，让spw管理它
                            killWidget);
ptrs.push_back(std::move(spw));              //添加spw右值
```

`emplace_back`的版本如下：

```cpp
std::shared_ptr<Widget> spw(new Widget, killWidget);
ptrs.emplace_back(std::move(spw));
```

无论哪种方式，都会产生`spw`的创建和销毁成本。选择置入而非插入的动机是避免容器元素类型的临时对象的开销。但是对于`spw`的概念来讲，当添加资源管理类型对象到容器中，并根据正确的方式确保在获取资源和连接到资源管理对象上之间无其他操作时，置入函数不太可能胜过插入函数。

**注意与`explicit`的构造函数的交互：**

C++11对正则表达式的支持，假设你创建了一个正则表达式对象的容器：

```cpp
std::vector<std::regex> regexes;
```

写出了如下看似毫无意义的代码：

```cpp
regexes.emplace_back(nullptr);           //添加nullptr到正则表达式的容器中？
```

你没有注意到错误，编译器也没有提示你，所以你浪费了大量时间来调试。突然，你发现你插入了空指针到正则表达式的容器中。但是这怎么可能？指针不是正则表达式，如果你试图下面这样写，

```cpp
std::regex r = nullptr;                  //错误！不能编译
```

编译器就会报错。有趣的是，如果你调用`push_back`而不是`emplace_back`，编译器也会报错：

```cpp
regexes.push_back(nullptr);              //错误！不能编译
```

当前你遇到的奇怪行为来源于“可能用字符串构造`std::regex`对象”的事实，这就意味着下面代码合法：

```cpp
std::regex upperCaseWorld("[A-Z]+");
```

通过字符串创建`std::regex`要求相对较长的运行时开销，所以为了最小程度减少无意中产生此类开销的可能性，采用`const char*`指针的`std::regex`构造函数是`explicit`的。这就是下面代码无法编译的原因：

```cpp
std::regex r = nullptr;                  //错误！不能编译
regexes.push_back(nullptr);              //错误
```

在上面的代码中，我们要求从指针到`std::regex`的隐式转换，但是构造函数的`explicit`ness拒绝了此类转换。

但是在`emplace_back`的调用中，我们没有说明要传递一个`std::regex`对象。然而，我们传递了一个`std::regex`**构造函数实参**。那不被认为是个隐式转换要求。相反，编译器看你像是写了如下代码：

```cpp
std::regex r(nullptr);                   //可编译
```

如果简洁的注释“可编译”缺乏直观理解，好的，因为这个代码可以编译，但是行为不确定。使用`const char*`指针的`std::regex`构造函数要求字符串是一个有效的正则表达式，空指针并不满足要求。如果你写出并编译了这样的代码，最好的希望就是运行时程序崩溃掉。如果你不幸运，就会花费大量的时间调试。

相似的初始化语句导致了不一样的结果：

```cpp
std::regex r1 = nullptr;                 //错误！不能编译
std::regex r2(nullptr);                  //可以编译
```

在标准的官方术语中，用于初始化`r1`的语法（使用等号）是所谓的**拷贝初始化**。相反，用于初始化`r2`的语法是（使用小括号，有时也用花括号）被称为**直接初始化**。

```cpp
using regex   = basic_regex<char>;

explicit basic_regex(const char* ptr,flag_type flags); //定义 (1)explicit构造函数

basic_regex(const basic_regex& right); //定义 (2)拷贝构造函数
```

拷贝初始化不被允许使用`explicit`构造函数（译者注：即没法调用相应类的`explicit`拷贝构造函数）：对于`r1`,使用赋值运算符定义变量时将调用拷贝构造函数`定义 (2)`，其形参类型为`basic_regex&`。因此`nullptr`首先需要隐式装换为`basic_regex`。而根据`定义 (1)`中的`explicit`，这样的隐式转换不被允许，从而产生编译时期的报错。对于直接初始化，编译器会自动选择与提供的参数最匹配的构造函数，即`定义 (1)`。就是初始化`r1`不能编译，而初始化`r2`可以编译的原因。回到`push_back`和`emplace_back`，更一般来说是，插入函数和置入函数的对比。置入函数使用直接初始化，这意味着可能使用`explicit`的构造函数。插入函数使用拷贝初始化，所以不能用`explicit`的构造函数。因此：

```cpp
regexes.emplace_back(nullptr);           //可编译。直接初始化允许使用接受指针的
                                         //std::regex的explicit构造函数
regexes.push_back(nullptr);              //错误！拷贝初始化不允许用那个构造函数
```

获得的经验是，使用置入函数时，请特别小心确保传递了正确的实参，因为即使是`explicit`的构造函数也会被编译器考虑，编译器会试图以有效方式解释代码。



**请记住：**

- 原则上，置入函数有时会比插入函数高效，并且不会更差。
- 实际上，当以下条件满足时，置入函数更快：（1）值被构造到容器中，而不是直接赋值；（2）传入的类型与容器的元素类型不一致；（3）容器不拒绝已经存在的重复值。
- 置入函数可能执行插入函数拒绝的类型转换。



























































































