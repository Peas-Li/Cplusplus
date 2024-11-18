# C++ Primer Plus 学习笔记



## 绪论

## **编译原理**

1. c++预编译，编译，汇编，链接；

   **预编译**：预编译把一些#define的宏定义完成文本替换，然后将#include的文件里的内容复制到.cpp文件里，如果.h文件里还有.h文件，就递归展开，最终生成.i文件；

   **编译**：编译只是把.i文件转为汇编代码.s，它的工作是检查词法和语法规则；

   **汇编**：汇编过程将上一步的汇编代码转换成机器码(machine code)，这一步产生.o文件(可重定位文件)，是二进制格式；

   **链接**：编译只是将我们自己写的代码变成了二进制形式，它还需要和系统组件（比如标准库、动态链接库等）结合起来，这些组件都是程序运行所必须的。链接（Link）其实就是一个“打包”的过程，它将所有二进制形式的目标文件(.o)和系统组件组合成一个可执行文件，此外需要注意的是：C++程序编译的时候其实只识别.cpp文件，每个cpp文件都会分别编译一次，生成一个.o文件。这个时候，链接器除了将目标文件和系统组件组合起来，还需要将编译器生成的多个.o或者.obj文件组合起来，生成最终的可执行文件(Executable file)；

2. 若用的是c语言的库，头文件用#include <c库名>；

6. 库文件中包含了函数的编译代码，而头文件中则包含了原型；

## **数据类型**

### 整型

1. short 至少16位；

2. int 至少与short 一样长；

3. long至少32位，且至少与int 一样长；

4. long long 至少 64 位，且至少与 long 一样长；

5. sizeof是运算符，查看数据类型的大小必须加括号；查看变量可以不加括号；

   ```c++
      #include <iostream>
      #include <climits>
      
      using namespace std;
      
      int main()
      {
          int n_int = INT_MAX;
          short n_short = SHRT_MAX;
          long n_long = LONG_MAX;
          long long n_llong = LLONG_MAX;
      
          cout << "short is " << sizeof(short) << " bytes." << endl;
          cout << "short 最大值 is " << n_short << endl;
          cout << "int is " << sizeof(int) << " bytes." << endl;
          cout << "int 最大值 is " << n_int << endl;
          cout << "long is " << sizeof(long) << " bytes." << endl;
          cout << "long 最大值 is " << n_long << endl;
          cout << "long long is " << sizeof(long long) << " bytes." << endl;
          cout << "long long 最大值 is " << n_llong << endl;
      
          return 0;
      }
   ```

6. int定义整数，用cin输入时，若输入的是字符串，会自动转换为整数；

   ```c++
   #include <iostream>
   using namespace std;
   
   int main(void){ 
       int a;
       
       cin>>a;
       
       cout<< a << endl;
       
       return 0;
   }
   //输入2m
   //输出2
   //输入m2
   //输出0
   ```

   

### 字符型

1. char一般8位；

2. char可以理解为比short更小的整型，因为在内存地址中，存储在变量中的值是整型，但打印的值是整型对应的ASCLL码中的字母，这种转换是由cin、cout实现的；

   ```c++
   #include <iostream>
   
   using namespace std;
   
   int main(){
       char ch = 'M';
       int k = ch;
   
       cout << "The ascll code for " << ch << " is " << k << endl;
   
       return 0;
   }
   ```

3. 单字符读取：

   1. cin.get(char &):

      1. 会读取空白字符；

      2. get(char &)成员函数返回一个指向用于调用它的istream对象的引用，这意味着可以拼接get(char &)后面的其他抽取；

      3. 如果程序到达文件尾，就没有值可供赋给参数了。 另外，该方法还调用setstate（failbit），导致cin的返回结果转换为false；

   2. cin.get(void):

      1. get(void)成员函数的返回类型为int；

      2. 会读取空白字符；

      3. 到达文件尾后（不管是真正的文件尾还是模拟的文件尾）， cin.get(void)都将返回值EOF：

         int ch; //将ch的类型声明为int，而不是char，因为值EOF可能无法使 用char类型来表示

         while((ch = cin.get()) != EOF) {...}

4. 字符运算：

   1.  ch = ++ch; //数据类型不会提升，ch还是字符型
   2. ch = ch + 1; //数据类型提升，ch变成整形


### 常量

1. const type Name = value；//需要对常量初始化

2. int n = 20;

   const int *pt = &n; //与int const *pt = &n; 等价

   *pt = 20; //错误，pt所指向的值被设置为常量 ，不能通过 pt 指针改变所指向的 n 的值

   n = 30; //正确

3. int n = 20;

   int m = 30;

   int * const p = &n;

   *p = 30; //正确

   p = &m; //错误，指针是常量，不能修改所指向的地址

4. c++ 禁止将const数据的地址赋给非const指针，否则造成数据不安全：

   1. const float g_earth = 9.80;

      const float * pe = &g_earth; // 允许

   2. const float g_moon=1.63;

      float*pm = &g_moon; // 不允许

 

### 浮点数

1. E表示法：

   1. 3.45**E**3表示3450，E是科学计数法的表示方法；
   2. 7**E**3相当于7.0**E**3，也是浮点型；
   3. 4.1**E**-3表示0.0041；

2. float至少32位，通常为32位；

3. double至少48位，且不少于float，通常为64位；

4. long double至少和double一样多，通常为80、96或128位；

5. 浮点类型的数据在计算机中用三个部分来表示：

   1. 符号位：只占一位（0或1）；
   2. 指数位：float占8位，double占11位；
   3. 尾数部分：float占23位，double占52位；

6. 浮点数的运算比整数慢，且精度会降低，float只能保证前6-7位的精度；


   ```C++
   #include <iostream>
   
   using namespace std;
   
   int main(){
       //cout会自动删除小数结尾的0
       cout.setf(ios_base::fixed, ios_base::floatfield); //保证小数点后结尾的0显示出来
   
       float tub = 10.0 / 3.0;
       const float millon = 1.0E6;
       double mint = 10.0 / 3.0;
   
       cout << "tub = " << tub << endl;
       cout << "A millon tubs = " << millon * tub << endl;
       cout << "Ten millon tubs = " << 10 * millon * tub << endl;
   
       cout << "mint = " << tub << endl;
       cout << "A millon mint = " << millon * mint << endl;
       cout << "Ten millon mint = " << 10 * millon * mint << endl;
       return 0;
   }
   ```

### 初始化与赋值

1. 初始化：

   ​	auto关键字，会自动根据初始值转为其对应的数据类型； 

   ​	auto n = 100; // n is int

   ​	auto n = 1.5; //n is double

2. 初始化赋值：

   1. int a(10);

   2. int a=10;

   3. int a{10};

   4. int a={10};

   5. int a={}; //赋值为0

      不允许将浮点型转化为整型，在不同的整形之间转换或者整形转换为浮点型可能被允许；

      花括号内不允许是未知变量，必须是已知的量，因为未知变量不确定其值的大小；

   6. int a{}; //赋值为0

   ```c++
   #include <iostream>
   #include <climits>
   
   using namespace std;
   
   int main()
   {
       int a = 10;
       int b(10);
       int c{10};
       int d = {10};
       int e{};
       int f = {};
       
       cout << a << endl;
       cout << b << endl;
       cout << c << endl;
       cout << d << endl;
       cout << e << endl;
       cout << f << endl;
   
       return 0;
   }
   ```

3. 赋值：

   1. 较大的浮点类型转换为较小的浮点类型：精度降低或者值超过目标类型的取值范围导致结果不确定；
   2. 将浮点类型转换为整型：小数丢失，值超过目标类型的取值范围导致结果不确定；
   3. 将较大的整型转换为较小的整型，如将 long 转换为 short ：原来的值可能超出目标类型的取值范围，通常只复制右边的字节；

4. 数值转换：

   1. 如果整数可以用于表示所赋值变量的类型，那么整数直接赋值给变量，不进行额外的类型转换；
   2. 如果整数无法完全表示所赋值变量的类型，那么整数会被存储为 int 类型，然后再将 int 类型转换为变量对应的类型；
   3. 运算过程中，较小的类型将被转换为较大的类型，例如：如果有一个操作数的类型是long double，则将另一个操作数转换为 long double；

5. 强制类型转换：

   ​	typeName (value)  = (typeName) value  //不会改变原有的数值类型，创建一个新的指定类型的值，这种方式不进行类型检查，编译器不会检查转换是否安全，而是直接进行转换；

   ​	static_cast < typeName > (value)  //它在编译时进行类型检查，如果转换是不安全的或者无效的，则会产生编译错误。


## 复合类型

### 数组：

1. 初始化数组

   1. typeName variable = {value1, value2, ....}; //等号可以省略,只能在定义数组时初始化，另外数组不能赋给另一个数组；

   2. int/long/float/double variable[4] = {1, 2}; //剩下的默认为0；

      int/long/float/double variable[4] = {0}; //全为0；

   3. 禁止缩窄转换：

      long variable[] = {25, 92, 3.2}; //不允许，浮点数转换为整型是缩窄操作，即使浮点数的小数点后面为零；

      char variable[] = {'a', 'b', 1112220004}; //不允许，1112220007超出了char的取值范围；
   
   4. 二维数组初始化：
   
      1. typeName variable[n] [m]= {
   
         ​	{value1, value2, ....}，
   
         ​	{value1, value2, ....}，
   
         ​	......
   
         }
   
      2. 二维字符串数组也可以用指针
   
         char * variable[n] = {
   
         ​	"string1"，
   
         ​	"string2"，
   
         ​	......
   
         }

### 字符串数组：

1. 字符串和字符数组的区别：

   char dog[4] = {'b', 'i', 'g', '!'}; // not a string!

   char dog[4] = {'b', 'i', 'g', '\0'}; // a string!

2. 字符串初始化：

   char str[] = "hello,world!";

   char str[8] = "fish"; // 空值字符'\0'自动加到末尾，即剩下的元素全为'\0'；

   ```c++
   #include <iostream>
   #include <cstring>
   using namespace std;
   
   int main(void){ 
   
       char str1[] = "hello,"; //未声明数组大小，自动计算字符串数组大小；
       char str2[10] = "world"; //'/0'自动补齐剩下元素
   
       cout << "hello," "world"<< endl; //输出拼接
       cout << str2[6] << endl; //空值' '
       cout << sizeof str1 << endl; //7，包含末尾的空值字符
       cout << strlen(str1) << endl; //6, 不包含末尾空值字符
       cout << sizeof str2 << endl; //10，数组大小
       
       return 0;
       
   }
   ```

3. 输入字符串：

   1. cin输入的陷阱：

      ```c++
      #include <iostream>
      #include <cstring>
      using namespace std;
      
      int main(void){ 
      
          const int size = 20;
      
          char str1[size];
          char str2[size];
      
          cout << "输入前半句" << endl;
          cin >> str1; //"hello, world"
          cout << "输入后半句" << endl;
          cin >> str2;
      
          /*没有输入str2，但程序在输入str1后结束，因为cin输入默认空格符结束，
            空格后的world被加入缓存区，前半段被str1捕获，
            因此str2会捕获缓存区的数据*/
            
          cout << str1 << str2 << endl; 
          
          return 0;
      }
      ```

   2. getline(char *, int)面向行的输入：

      1. 若是字符串类型，则用 getline(cin, str); //str为字符串类型

      2. 若是字符数组，则用 cin.getline(char *, int); 

      3. 两个版本的getline( )都有第三个可选参数，用于指定使用哪个字符来确定输入的边界：getline(cin, str, ch);  cin.getline(char *, int, ch);

      4. getline(char *, int)返回一个 cin 对象，可以接着调用getline(char *, int)；
   
      5. getline(char *, int)方法将从输入队列中读取字符，将它们放到str数组的元 素中，直到（按测试顺序）到达文件尾、将要读取的字符是换行符或存储了size个字符为止。如果遇到文件尾，则设置eofbit；如果将要读取的 字符是换行符，则该字符将被读取并丢弃；如果读取了size个字符，并且 下一个字符不是换行符，则设置failbit。因此，包含size个或更多字符的输入行将终止输入。
   
      6. 若用 get 或 cin 输入后，接着用 getline(char *, int) 输入会出现问题，因为 cin 会把换行符留在缓冲区内，在 cin 输入后，加上 cin.get( ) 读取换行符即可；
   
         ```c++
         #include <iostream>
         using namespace std;
         
         int main(void){ 
             
             int a;
             string b;
         
             cin >> a; //输入11
             getline(cin, b); //没有阻塞进程，程序继续运行
         
             cout << a << endl;
             cout << b << endl;
             
             return 0;
             
         }
         //11
         //  
         ```
   
         ```c++
         #include <iostream>
         using namespace std;
         
         int main(void){ 
             
             int a;
             string b;
         
             cin >> a; //输入11
             cin.get(); //cin.get( ) 读取换行符
             getline(cin, b); //程序阻塞，要求输入，输入12
         
             cout << a << endl;
             cout << b << endl;
             
             return 0;
             
         }
         //11
         //12
         ```

   3. get(char *, int)面向行的输入：
   
      1. 只能用于字符数组，对字符串声明的变量不能用；
   
      2. cin.get(char *, int); //两个参数，读取指定数目（size-1）的字符
   
      3. 若用 get 或 cin 输入后，接着用 get(char *, int) 读取字符串会出现问题，因为 cin 会把换行符留在缓冲区内，在 cin 输入后，加上 cin.get( ) 读取换行符即可（同getline遇到此情况的操作）；
   
         ```c++
         #include <iostream>
         using namespace std;
         
         int main(void){ 
         
             const int size = 20;
         
             char str1[size];
             char str2[size];
         
         		//输入hello,world
             cin.get(str1,size); //输入 hello，world换行符
             cin.get(str2,size); //换行符还在缓冲区，此时get()认为已经到达行尾，没有可读取的内容，没有阻塞，程序继续运行结束
         
             cout << str1 << str2 << endl; 
             
             return 0;
         }
         
   
      //hello,world
         ```
      
         ```c++
         #include <iostream>
         using namespace std;
         
         int main(void){ 
         
             const int size = 20;
         
             char str1[size];
             char str2[size];
         		
         		//输入hello,world
             cin.get(str1,size); //输入 hello，world换行符
             cin.get(); //读取换行符
             //输入!
             cin.get(str2,size); //程序阻塞，要求输入；
         
             cout << str1 << str2 << endl; 
             
             return 0;
         }
         
      //hello,world！
         ```
   
      4. cin.get(char *, int, char); //第三个是可选参数，用于指定使用哪个字符来确定输入的边界；
   
      5. cin.get(char *, int)返回一个 cin 对象，可以接着调用get(char *, int);
   
         ```c++
         #include <iostream>
         using namespace std;
         
         int main(void){ 
         
             const int size = 20;
         
             char str1[size];
             char str2[size];
             
             cin.get(str1,size).get(); //输入hello，world换行符,并清除缓冲区中的换行符
             cin.get(str2,size); //输入!程序阻塞，要求输入；
         
             cout << str1 << str2 << endl; 
             
             return 0;
         }
         
   
      //hello,world！
         ```
      
      6. get(char *, int)方法。它首先测试字符数，然后测试是否为 文件尾以及下一个字符是否是换行符。如果它读取了最大数目的字符， 则不设置failbit标记。
   
   4. cin.ignore()：
   
      1. 用于从输入流中忽略一个或多个字符；
   
      2. 原型：istream & ignore(int  n = 1, char delim = EOF); //默认参数值EOF导致ignore( )读取指定数目的字符或读取到文件尾。
   
      3. 该函数返回调用对象，这使得能够拼接函数调用:
   
         cin.ignore(255, '\n').ignore(255, '\n'); //第一个ignore( )方法读取并丢弃一行，第二个调用读取并丢弃另一行，因此一共读取了两行;
   
      4. 其中指定的分界符也会读取并丢弃；
   
      

### string对象：

1. 使用string对象的方式与使用字符数组相同：

   1. 可以使用 C-风格字符串来初始化 string 对象；
   2. 可以使用 cin 来将键盘输入存储到 string 对象中；
   3. 可以使用 cout 来显示 string 对象；
   4. 可以使用数组表示法来访问存储在 string 对象中的字符；

2. 列表初始化：string variable = {"hello, world"}；

3. string 数组初始化： string variable[n] = {

   ​							"str1",

   ​							"str1",

   ​							......

   ​			   }；

4. 赋值、拼接和附加：

   1. str3 = str2；
   2. str3 = str1 + str2；
   3. str3 += str；

5. string 类字符串可以进行比较

   ```c++
   #include <iostream>
   using namespace std;
   
   int main(void){ 
   
       string str1 = "123";
       string str2 = "123";
   
       cout << (str1 == str2) << endl;
   
       return 0;
   }
   
   //1
   ```

### 结构

1. 定义与声明结构：

   ```c++
   #include <iostream>
   using namespace std;
   
   struct student{
       int sno;
       string name;
       int age;
   }; //注意分号
   
   int main(void){ 
   
       student s; //不需要额外的在 student 前加上 struct；
   
       cin >> s.sno; //2023
       cin.get(); //读取缓冲区的换行符
       getline(cin, s.name); //li xiao hui
       cin >> s.age; //21
   
       cout << s.sno << endl;
       cout << s.name << endl;
       cout << s.age << endl;
       
       return 0；
   }
   
   //2023
   //li xiao hui
   //23
   ```

2. 结构初始化：

   1. 列表初始化 structName variableName = {value1, value2, ...} //等号可以省略；
   2. struct job {
          char name[10];
          double salary;
          int floor;
      } m = {"John", 1000.0, 1}, n = {"Doe", 2000.0, 2};
   3. 若大括号内为空，则各成员都将被设置为零；

3. 结构体数组：

   1. 每个成员都是结构体对象；

   2. 列表初始化结构体数组：

      structName structVariable[n] = {

      ​	{value1, value2, ...}, //逗号分隔

      ​	{value1, value2, ...}

      };

4. 访问成员：

   运算符句点：variableName.成员

5. 结构体的对齐和补齐的规则：

   对齐：假定从零地址开始，每成员的起始地址编号，必须是它本身字节数的整数倍。
   补齐：结构的总字节数必须是它最大成员的整数倍。

### 共用体

1. 存储不同的数据类型，但只能同时存储其中的一种类型，因此共用体的长度是最大成员的长度；

   ```c++
   #include <iostream>
   using namespace std;
   
   union one3all{
       int int_val;
       long long_val;
       double double_val;
   };
   
   int main(void){ 
   
       one3all s;
   
       cin >> s.int_val; //1
       cout << s.int_val << endl;
     
       cin >> s.double_val; //1.1
       cout << s.double_val << endl;
     
       return 0;
       
   }
   
   //1
   //1.1
   ```

2. 共用体中的所有数据类型共用一个值；

   ```c++
   #include <iostream>
   using namespace std;
   
   union one2all{
       int int_val;
       char char_val;
   };
   
   int main(void){ 
   
       one2all s;
   
       cin >> s.int_val; //65
   
       cout << s.int_val << endl;
       cout << s.char_val <<endl;
   
       return 0;
       
   }
   
   //65
   //A
   ```

3. 共用体的用途：

   1. 当数据项使用两种或更多种格式(但不会同时使用)时，可节省空间；

      ```c++
      #include <iostream>
      using namespace std;
      
      struct student{
          int type;
          union id{
              int id_num;
              char id_char[20];
          }id_val;
      };
      
      int main(void){ 
      
          student s;
      
          cin >> s.type;
          if(s.type==1){
              cin >> s.id_val.id_num;
              cout << s.id_val.id_num << endl;
          }  
          else{
              cin >> s.id_val.id_char;
              cout << s.id_val.id_char << endl;
          }
          
          return 0;
      }
      ```

   2. 匿名联合体：没有名称，其成员将成为位于相同地址处的变量；

      ```c++
      #include <iostream>
      using namespace std;
      
      struct student{
          int type;
          union{
              int id_num;
              char id_char[20];
          };
      };
      
      int main(void){ 
      
          student s;
      
          cin >> s.type;
        
          if(s.type==1){
              cin >> s.id_num;
              cout << s.id_num << endl;
          }  
          else{
              cin >> s.id_char;
              cout << s.id_char << endl;
          }
          
          return 0;
      }
      ```

### 枚举

1. 枚举作用：

   将每个成员作为符号常量，这些常量叫做枚举量，默认情况下他们对应整数值0~n（n是枚举量的个数）；

2. 设置枚举的值：

   ```c++
   #include <iostream>
   using namespace std;
   
   int main(void){ 
   
       enum bits{one = 1, two = 2, four = 4, eight = 8};
       enum bigstep{first, second = 100, third}; //first = 1, third = 101
       enum s{zero, null = 0,one ,numero_uno = 1}; //zero = 0, one = 1
       
       return 0;
   }
   ```

3. 枚举的取值范围：

   如上述 bits， 上界是比8还大的2的幂减一，即15；

   若枚举的最小值不小于0，则下界为0；否则为比它小的、最大的2的幂（有负号）+1；

   因此下面代码是合法的：

   bits myflag;

   myflag = bits(6); //因为6在 bits 的取值范围内，若不在则不合法；

4. 枚举的运算：

   枚举量是整型，可以被提升为 int 类型，但是 int 类型不能自动转换为枚举类型；

   1. int num = one; //合法

      num = 5 + two; //合法

   2. bits k; 

      k = one; //合法 

      k++; //不合法

      k = 2000; //不合法

      k = one + two; //不合法，枚举没有算术运算

### 指针

1. 指针的注意问题：

   1. 指针占用的内存空间是一样的（只与操作系统有关），与指针指向的数据类型无关；

   2. 警告:一定要在对指针应用解除引用运算符(*)之前，将指针初始化为一个确定的、适当的地址：

      int * local;

      *local = 100; //不可取，local所在的地址很有可能已经存放了某些数据

   3. 不能简单地将整数赋给指针：

      int * local;

      local = 0xB8000000; //不可取，右侧数字只是16进制数字，不能表示16进制地址；

2. 动态申请内存空间：

   1. `new` ：分配内存； `delete` ：释放内存；
   2. new 与 delete 搭配使用；
   3. 对空指针应用 delete 是安全的；

3. 指针与单个实体：

   1. 使用 new 关键字：
      1. 分配：typeName * pointerName = new typeName; 
      2. 释放：delete pointerName;
      3. 创建一个空指针：
         1. pointerName = nullptr;(c++ 11)
         2. pointerName = 0;(c++98)
         3. pointerName = NULL;

4. 指针与数组：

   1. 使用 new 关键字：

      1. 分配：typeName * pointerName = new typeName [n];
      2. 释放：delete [] pointerName;
      3. 创建一个空指针：
         1. pointerName = nullptr;(c++ 11)
         2. pointerName = 0;(c++98)
         3. pointerName = NULL;

   2. 指针访问元素：

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          int * psome = new int [10];
          psome[0] = 2;
          psome[1] = 5;
          psome[2] = 9;
          
          cout << psome[0] << endl; //下标访问
          cout << *psome << endl; //取地址访问
          cout << *(psome + 1) << endl; 
      
          psome = psome + 1; //首地址指向下一位元素
      
          cout << psome[0] << endl;
      
          delete [] psome;
      
          return 0;
      }
      
      //2
      //2
      //5
      //5
      ```

   3. 数组的地址：

      1. 数组名被解释为其第一个元素的地址（例外：sizeof 运算符用于数组名时，返回整个数组的长度），而对数组名应用地址运算符时，得到的是整个数组的地址。

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          short tell[10];
          
          cout << tell << endl; //数组元素的地址
          cout << tell + 1 << endl; //数组第二个元素的地址
          cout << &tell << endl; //数组的地址
          cout << &tell + 1 << endl; ////数组的地址加上数组占用的空间后的地址
      
          return 0;
      }
      ```

   4. short * Name[n]; //指针数组

      short (* Name)[n]; //数组指针

5. 指针与字符串:

   1. 对于其他类型的指针，C++将其对应于void *，并打印地址的数值 表示。如果要获得字符串的地址，则必须将其强制转换为其他类型，即(void *) pt;

   2. 将字符串放到数组中的方法：

      1. 初始化数组时，使用 = 运算符；

      2. 其他情况用 `strcpy( )` 或 `strncpy( )`；

         1. strcpy( )：将指定的字符串复制到字符数组当中，前提是字符串的长度小于等于字符数组的长度，否则会警告；

         2. strncpy( ) 函数还接受第3个参数：要复制的最大字符数，如果该函数在到达字符串结尾之前，目标内存已经用完，则它将不会添加空字符，应该要手动设置空字符。

            ```c++
            #include <iostream>
            #include <cstring>
            using namespace std;
            
            int main(void){ 
            
                char str[20] = "123456";
                char str1[20];
            
                strcpy(str1, "hello,world");
                cout << str1 << endl;
                strncpy(str1, "a picnic basket filled with many goodies", 19);
                str1[19] = '\0';
                cout << str1 << endl;
            
                return 0;
            }
            
            //hello,world
            //a picnic basket fil
            ```

   3. 如果给 cout 提供一个字符的地址，则它将从该字符开始打印，直到遇到空字符为止。这里的关键不在于 str 是数组名，而在于 str 是一个char 的地址。用引号括起的字符串 "world\n" 对于 c++ 也是第一个元素的地址。

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          char str[20] = "hello,";
      
          cout << str << "world\n";
      
          return 0;
      }
      
      //hello,world
      ```

   4. 区别于其他类型的数组，字符数组在打印首字符地址时，需要强制类型转换；

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          char str[20] = "hello,";
      
          cout << sizeof( char ) << endl; //char类型的大小
          cout << sizeof str << endl; //str所占用的空间
          cout << &str << endl; //整个字符数组的地址
          cout << &str + 1 << endl; //字符数组地址加上数组占用的空间后的地址
          cout << (int *)str << endl; //首字符的地址
          cout << (int *)(str + 1) << endl; //第二个字符的地址
      
          return 0;
      }
      
      ```

6. 指针与结构：

   1. 创建与访问；

      1. `new` 创建结构：structName * pointerName = new structName;

      2. 指针指向结构：structName variableName = {value1, value2 ...};

         ​		  	structName *  pointerName  = &variableName;

      3. 箭头运算符 `->`：pointerName -> variableName成员;

         句点运算符 `.` ：(*pointerName).variableName成员;

      ```c++
      #include <iostream>
      using namespace std;
      
      struct student{
          char name[20];
          int age;
      };
      
      int main(void){ 
      
          student *ps = new student;
          cin >> (*ps).age; //23
          cin.get();
          cin.getline(ps -> name, 20); //li xiaohui
      
          cout << ps -> name << endl;
          cout << ps -> age << endl;
      
          delete ps;
          return 0;
      }
      
      //li xiaohui
      //23
      ```

   2. 理解下面代码：

      ```c++
      #include <iostream>
      using namespace std;
      
      struct student{
          int age;
      };
      
      int main(void){ 
      
          student s01, s02, s03;
          s01.age = 20;
          student * pa = &s02;
          pa -> age = 21;
          student s[3];
          s[0].age = 22;
          cout << s -> age << endl;
          const student * arp[3] = {&s01, &s02, &s03};
          cout << arp[1] -> age << endl;
          const student ** ppa = arp;
          auto ppb = arp;
          cout << (*ppa) -> age << endl;
          cout << (*(ppb + 1)) -> age << endl;
      
          return 0;
      }
      
      //22
      //21
      //20
      //21
      ```


### 模板类

1. `vector` 类：

   1. vector<typename> vt(n); //n是整型常量或者整型变量
   2.  vector 类数组是存储在栈上，但内部的动态分配的数组元素是存储在堆上；

   ```c++
   #include <iostream>
   #include <vector>
   #include <array>
   using namespace std;
   
   int main(void){ 
   
       double s1[4] = {1.1, 1.2, 1.3, 1.4};
   
       vector<double> s2(4);
       s2[0] = 1.0/3.0;
       s2[1] = 1.0/5.0;
       s2[2] = 1.0/7.0;
       s2[3] = 1.0/9.0;
   
       cout << "s1[0] = " << s1[0] << " at " << &s1[0] << endl;
       cout << "s2[0] = " << s2[0] << " at " << &s2[0] << endl;
       cout << "s2的地址时" << &s2 << endl;
   
       return 0;
   }
   ```

2. `array` 类：

   1. array<typename,n> arr; //n必须是整型常量

   ```c++
   #include <iostream>
   #include <vector>
   #include <array>
   using namespace std;
   
   int main(void){ 
   
       double s1[4] = {1.1, 1.2, 1.3, 1.4};
   
       array<double, 4> s3 = {3.14, 2.72, 1.62, 1.41};
       array<double, 4> s4;
   
       s4 = s3;
       
       cout << "s1[0] = " << s1[0] << " at " << &s1[0] << endl;
       cout << "s3[0] = " << s3[0] << " at " << &s3[0] << endl;
       cout << "s4[0] = " << s4[0] << " at " << &s4[0] << endl;
   
       return 0;
   }
   ```

3. 越界行为：

   1. 数组越界访问，不报错，后果需要程序员自己承担；

      ```c++
      #include <iostream>
      #include <vector>
      #include <array>
      using namespace std;
      
      int main(void){ 
      
          double s1[4] = {1.1, 1.2, 1.3, 1.4};
          
          s1[-2] = 20.2; 
          cout << "s1[-2] = " << s1[-2] << " at " << &s1[-2] << endl;
      
          return 0;
      }
      ```

   2. `vector` 和 `array` 对象能使用成员函数 `at( )` 禁止此行为，并打印错误；

      ```c++
      #include <iostream>
      #include <vector>
      #include <array>
      using namespace std;
      
      int main(void){ 
      
          double s1[4] = {1.1, 1.2, 1.3, 1.4};
      
          vector<double> s2(4);
          s2[0] = 1.0/3.0;
          s2[1] = 1.0/5.0;
          s2[2] = 1.0/7.0;
          s2[3] = 1.0/9.0;
      
          array<double, 4> s3 = {3.14, 2.72, 1.62, 1.41};
          
          s2.at(-2) = 1.5; //s3同理
          cout << "s2[-2] = " << s2[-2] << " at " << &s2[-2] << endl;
      
          return 0;
      }
      
      //terminate called after throwing an instance of 'std::out_of_range'
      ```


### 类型别名

1. #define typeName aliasName;

   1. 不适用于声明一系列变量，如下：

      #define FLOAT_POINTER float *

      FLOAT_POINTER pa, pb;

      预处理器置换将声明转换成这样：

      float * pa, pb; //pa是浮点类型指针，pb不是

   1. 一般来说，该方法用作于代替某个变量，如下：

      #define PI 3.1415926; //程序中出现 PI 即代表该浮点数常量

      s = PI * r * r; //s，r均为浮点类型变量，r 表示半径，s 代表圆的面积；

1. typedef typeName aliasName；

   1. 相较于 #define 该方法不会有上述问题；

   1. 另一种写法是将别名当做标识符进行声明，并在开头使用关键字typedef，如下：

      typedef double * pt; //pt即为 double * 类型的别名

   

## 循环和关系表达式

### for循环

用法同c语言；

### `++` `-- ` 运算符

1. 用法格式同c语言，分为前缀和后缀格式：

2. 虽然选择使用前缀格式还是后缀格式对程序的行为没有影响，然而，对于自定义类型，特别是类类型，执行速度可能有细微的差别。

   1. 前缀递增运算符 `++x`： 首先将值加 1，然后返回结果。
   2. 后缀递增运算符 `x--`： 首先复制一个原始值，然后将原始值加 1，最后返回复制的原始值。

3. 与指针搭配使用时注意，前缀运算符和 `*` 运算符的优先级一样，同时出现时从右往左计算，此外后缀运算符的优先级高于 `*` 运算符；

   ```c++
   #include <iostream>
   #include <vector>
   #include <array>
   using namespace std;
   
   int main(void){ 
   
       double s1[4] = {1.1, 1.2, 1.3, 1.4};
       double * pt = s1;
   
       cout << "*pt = " << *pt << endl;
       cout << "*++pt = " << *++pt << endl;
       cout << "++*pt = " << ++*pt << endl;
       cout << "(*pt)++ = " << (*pt)++ << endl;
       cout << "*pt = " << *pt << endl;
       cout << "*pt++ = " << *pt++ << endl;
       cout << "*pt = " << *pt << endl;
   
       return 0;
   }
   
   //*pt = 1.1    
   //*++pt = 1.2  
   //++*pt = 2.2  
   //(*pt)++ = 2.2
   //*pt = 3.2    
   //*pt++ = 3.2  
   //*pt = 1.3  
   
   ```

### 逗号运算符

1. `,` 运算符最常见的用途是把两个或更多的表达式放到一个for 循环表达式中。
2. C++ 还为这个运算符提供了另外两个特性。首先，它确保先计算第一个表达式，然后计算第二个表达式，以此类推。需要注意的是：逗号运算符的优先级是最低的；
   1. cata =17, 240; 被解释为:(cats =17), 240;
   2. cats = (17, 240); //cats 的值被设置为240；.

### while循环

1. 用法同 c语言；

2. while 常用来用作延时设置；

   ```c++
   #include <iostream>
   #include <ctime> //时间头文件
   using namespace std;
   
   int main(void){ 
   
       float time; //延时时间；
       cin >> time;
   
       clock_t delay = time * CLOCKS_PER_SEC; //CLOCKS_PER_SEC 是计算机每秒运行的次数
       clock_t start = clock(); //记录截至当前，程序运行的系统时间（并不是以秒为单位），即程序运行了多少次
   
       while(clock() - start < delay); 
   
       return 0;
   }
   ```




### do while循环

用法同 c语言；

do{

}while();

### 基于范围的 for 循环

1. 对数组的每个元素执行相同的操作；

   1. 循环显示该数组的值；

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          double price[5] = {1.1, 1.2, 1.3, 1.4, 1.5};
      
          for(double x : price)
              cout << x << endl; 
      
          return 0;
      }
      
      //1.1
      //1.2
      //1.3
      //1.4
      //1.5
      ```

   2. 修改数组的元素；

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          double price[5] = {1.1, 1.2, 1.3, 1.4, 1.5};
      
          for(double &x : price)
              x = x * 0.8;
      
          for(double x : price)
              cout << x << endl; 
      
          return 0;
      }
      
      //0.88
      //0.96
      //1.04
      //1.12
      //1.2
      ```

### 循环文本输入 

1. 哨兵字符：

   1. 使用原始的 cin 输入，直到遇到 ’#‘ 结束，每换行一次输出读取的字符，不会打印空格；

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          char ch;
          int count = 0;
          
          //输入l x h #
          cin >> ch;
      
          while(ch != '#'){
              cout << ch;
              count++;
              cin >> ch;
          }
      
          return 0;
      }
      
      //lxh
      ```

   2. 使用 cin.get(ch) 输入，直到遇到 ’#‘ 结束，每换行一次输出读取的字符，空格也会打印；

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          char ch;
          int count = 0;
      		////输入l x h #
          cin.get(ch);
      
          while(ch != '#'){
              cout << ch;
              count++;
              cin.get(ch);
          }
      
          return 0;
      }
      
      //l x h
      ```

2. 文件尾（EOF）；

   1. 循环不会因为某个字符终止；

   2. 下面的程序会从标准输入中逐字符读取字符，每换行一次输出读取的字符，循环结束的条件是 ctrl + z（Windows，其他操作系统可能有所不同）；

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void){ 
      
          char ch;
          int count = 0;
      
          cin.get(ch);
      
          while(cin.fail() == false){
              cout << ch;
              count++;
              cin.get(ch);
          }
      
          cout << count;
      
          return 0;
      }
      ```

      修改后的简洁代码；

      ```c++
      #include <iostream>
      using namespace std;
      
      int main(void) {
      
          char ch;
          int count = 0;
      
          while (cin.get(ch)) {
              cout << ch;
              count++;
          }
      
          cout << count;
      
          return 0;
      }
      
      ```


## 分支语句和逻辑运算符

### if语句

用法同c语言；

### 逻辑表达式

1. 逻辑或 || ：express1 || express2 || ...//从左往右判断，只要遇到为真的表达式，则不会继续往右判断，优先级小于所有算术运算符；
2. 逻辑与 && ：express1 && express2 && ...//从左往右判断，只要遇到为假的表达式，则不会继续往右判断，优先级高于逻辑或运算符，小于所有算术运算符；
3. 逻辑非 ！：优先级大于所有关系运算符和算术运算符；

### switch语句

1. switch  (integer-expression)

   {

   ​	case lable1 : statement(s);break; 

   ​	case lable1 : statement(s);break;

   ​	...

   ​	default : statement(s);break;

   }

   1. 若没有break，则会继续后续的case，直到遇到break或者语句块结束；
   2. integer-expression必须是整数值的表达式，包括char类型；
   3. lable必须是整数值的表达式，包括char类型；

2. 枚举类型和switch语句的应用；

   ```c++
   #include <iostream>
   using namespace std;
   
   enum {red, orange, yellow, green};
   
   int main(void){ 
       
       int code;
       cout << "输入0-3" << endl;
       cin >> code;
   
       while(code >= red && code <= green){
           switch(code){
               case red: cout << "red" << endl; break;
               case orange: cout << "orange" << endl; break;
               case yellow: cout << "yellow" << endl; break;
               default: cout << "green" << endl; break;
           }
           cin >> code;
       }
       return 0;
   }
   ```

### 读取数字的循环

1. 假设要编写一个将一系列数字读入到数组中的程序，并允许用户在数组填满之前结束输入，如果不是输入一个数字，则会发生以下情况：

   1. n的值保持不变；
   2. 不匹配的输入将被留在输入队列中；
   3. cin对象中的一个错误标记被设置；
   4. 对cin方法的调用将返回false(如果被转换为bool类型)；

   ```c++
   #include <iostream>
   using namespace std;
   
   enum {red, orange, yellow, green};
   
   int main(void){ 
       
       int n, m;
       cin >> n; //输入xyz，程序阻塞，无法输入，也无法结束
       cin >> m;
   
       cout << "n = " << n << " " << "m = " << m << endl;
       
       return 0;
   }
   ```

   此时 cin 功能会禁用，使用 cin.clear() 方法即可清除错误输入标记，同时也重置文件尾（EOF条件）程序继续运行；

   ```c++
   #include <iostream>
   using namespace std;
   
   enum {red, orange, yellow, green};
   
   int main(void){ 
       
       int n, m;
       cin >> n; //xyz
       cin.clear();
       while(cin.get() != '\n'); //清除 cin 队列里缓冲的数据
       cin >> m; //1
   
       cout << "n = " << n << " " << "m = " << m << endl;
       
       return 0;
   }
   //n = 0 m = 1
   ```

### 简单文件输入输出

1. 创建文本并写入内容：

   1. outFile 用法同 cout：

      ```c++
      #include <iostream>
      #include <fstream>
      using namespace std;
      
      int main(void){ 
          
          ofstream outFile; //创建输出文件流对象
          outFile.open("F:/C++Study/temp.txt"); //将该对象关联一个文件
          
          outFile << fixed; //以小数的方式打印
          outFile.precision(2); //保持小数点后两位精度
          outFile.setf(ios_base::showpoint); //显示浮点数的小数点和尾随零
          outFile << 1111.00 << endl; //写入1111.00
      
        	outFile.close(); //关闭文件
          return 0;
      }
      ```

2. 读取文本文件：

   1. inFile 用法同 cin：

      ```c++
      #include <iostream>
      #include <fstream>
      #include <cstdlib>
      using namespace std;
      
      
      int main(void){ 
          
          int value;
          double sum = 0.0;
          int count = 0;
          ifstream inFile;
      
          inFile.open("F:/C++Study/temp.txt");
          if(!inFile.is_open()){ //判断是否成功打开
              cout << "文件打开失败" << endl;
              cout << "程序终止" << endl;
              exit(EXIT_FAILURE); //终止程序
          }
          cout << "成功打开“temp.txt”文件" << endl;
      
          while(inFile >> value){ //判断是否读取成功
              ++count;
              sum += value;
          }
      
          if(inFile.eof()) //是否读到文件末尾
              cout << "读到文件末尾" << endl;
          else if(inFile.fail()) //读取失败，数据不匹配
              cout << "读入数据不匹配" << endl;
          else
              cout << "未知原因,没有读取到末尾" << endl;
      
          if(count == 0)
              cout << "没有数据被读取" << endl;
          else{
              cout << "总共有 "  << count << " 个数据" << endl;
              cout << "数据总和 " << sum << endl;
              cout << "平均值 " << sum / count << endl;
          }
          inFile.close();
          return 0;
      }
      
      //成功打开“temp.txt”文件
      //读到文件末尾
      //总共有 10 个数据
      //数据总和 55
      //平均值 5.5
      ```


## 函数

1. 函数原型：描述了函数到编译器的接口；

   1. 通常，原型自动将被传递的参数强制转换为期望的类型。自动类型转换并不能避免所有可能的错误。例如，如果将8.33E27传递给期望一个int 值的函数，则这样大的值将不能被正确转换为int值。当较大的类型被自动转换为较小的类型时，有些编译器将发出警告指出这可能会丢失数据。

2. 函数参数和按值传递：

   1. 在参数中修改形参的值不会影响调用程序中的数据，因为形参只是原值的拷贝；

3. 函数和数组：

   1. 在 C++中，当(且仅当)用于函数头或函数原型中，int *arr和int arr []的含义才是相同的，即下面两个函数声明等价；

      typeName func(typeName *arr,typeName n);

      typeName funName(typeName arr[],typeName n);

   2. 使用数组名（指针）作为参数，则在函数中改变数组中的值，其主函数中数组对应的值也会改变，换言之使用指针作为形参，函数直接操作的是指针对应的值，并不会对其拷贝；

   3. 若不想改变原数组的值，可以使用下面的声明语句；

      typeName func(const typeName arr[],typeName n); //使用const关键字，可以接受非const和const类型的数据

4. 函数和二维数组：

   1. typeName func(typeName (*ar2)[4], typeName size);

      typeName func(typeName *ar2[ ] [4], typeName size);

5. 函数和结构体：

   1. 一般是倾向于传递结构的地址，然后使用指针来访问结构的内容；

6. 函数和string对象：

7. 函数和array对象：

   1. 原型：

      1. 若要修改主程序中array数组中的数据：void func(array<typeName, size> *pt);
      2. 若只是读取数据：void func(const array<typeName, size> *pt);

   2. array类访问元素的方式和数组一样，但是指针的使用方式不一样；

      ```c++
      #include <iostream>
      #include <array>
      
      using namespace std;
      
      void inputArray(array<int, 4> *s, int size);
      
      int main(){
          array<int, 4> s;
      
          inputArray(&s, 4);
      
          for(int i = 0; i < 4; i++){
              cout << s[i] << " ";
          }
      
          return 0;
      }
      
      void inputArray(array<int, 4> *s, int size){
          for(int i = 0; i < size; i++){
              cin >> (*s)[i];
          }
      }
      ```

8. 递归：

9. 函数指针：

   1.  函数地址：函数名就是函数的地址；

   2. 声明函数指针：

      原型：double pam(int); 

      声明该函数类型的函数指针：double  (* pt)(int); //(* pt)也是函数，pt是指针，形参和返回值必须和目标函数的签名一致；

   3. ```c++
      #include <iostream>
      
      using namespace std;
      
      double sum(int n);
      double mul(int n);
      void fun(int n, double (*pf)(int));
      
      int main(){
          int n;
          cout << "输入 n:" << endl; //5
          cin >> n;
          fun(n, mul);
          fun(n, sum);
      }
      
      double sum(int n)
      {
          double s = 0;
          for(int i = 1; i <= n; i++)
              s += i;
          return s;
      }
      
      double mul(int n)
      {
          double s = 1;
          for(int i = 1; i <= n; i++)
              s *= i;
          return s;
      }
      
      void fun(int n, double (*pt)(int))
      {
          cout << pt(n) << endl;
      }
      
      //120
      //15
      ```

## 函数探幽

### 内联函数

1. 应有选择地使用内联函数。如果执行函数代码的时间比处理函数调用机制的时间长，则节省的时间将只占整个过程的很小一部分。如果代码执行时间很短，则内联调用就可以节省非内联调用使用的大部分时间；

2. 如果你在将内联函数的定义放在实现文件中时不删除 `inline` 关键字，编译器可能会忽略该关键字，并将函数视为普通的外部函数。这意味着函数的链接性将是外部的，而不是内联的。

   当你在头文件中声明内联函数时，编译器通常会将函数视为内联函数，并尝试在每个调用点处进行内联展开。但是，如果你在实现文件中的函数定义中保留了 `inline` 关键字，则编译器可能会选择忽略它，并将函数视为普通的外部函数。这意味着函数的定义仍然位于头文件中，但不会进行内联展开，而是在链接阶段进行链接。

   这种情况下，如果多个源文件包含了同一个头文件，并且头文件中包含了内联函数的定义，那么在链接阶段可能会出现重复定义的错误。因为每个源文件都会包含一份相同的函数定义，导致重复定义的问题。

   因此，为了避免重复定义的错误，如果将内联函数的定义放在实现文件中，则应删除 `inline` 关键字，使函数具有外部链接性(此时不是内联函数)。这样，在每个源文件中包含头文件时，只会看到函数的声明，而不会看到函数的定义，从而避免了重复定义的问题。

3. 一般情况下，通常的做法是省略原型，将整个定义（即函数头和所有函数代码） 放在本应提供原型的地方（一般是头文件），并加上inline关键字。但内联函数的定义不一定要跟声明放在一个头文件里面：定义可以放在一个单独的头文件中，里面需要给函数定义前加上inline 关键字，然后声明放在另一个头文件中，此文件include上一个头文件。

4. 在 main 函数前面定义函数，并在开头加上关键字 inline，省略了函数原型：

   ```c++
   #include <iostream>
   
   using namespace std;
   
   inline double sqrt(double n){
       return n*n;
   }
   
   int main(){
       double temp = 1.11;
       cout << sqrt(temp) << endl;
   }
   
   ```

5. 不同于宏定义 #define，如下：

   #define square(x) x*x

   a = square(5.0); //虽然也没有函数调用的时间，但在本质上不同，前者是函数，后者是文本替换

### 引用变量

1. typeName variable1 = value; 

   typeName **&** variable2 = variable1; //声明引用变量时必须对其进行初始化，variable1 和 variable2 的地址和值完全一样

2. 引用变量一旦与某个变量关联起来，将一直效忠于它，引用变量的值改变，其关联的值也会改变，反之亦然，也就是说：

   typeName **&** variable2 = variable1;

   实际上是下述代码的伪装表示：

   typeName ***** const  variable2 = **&**variable1;

   ```c++
   #include <iostream>
   
   using namespace std;
   
   int main(){
       int rats = 101;
       int &rodents = rats;
       int bunnies = 50;
   
       cout << "rats = " << rats << endl;
       cout << "rodents = " << rodents << endl;
   
       rodents = bunnies;
   
       cout << "rodents = " << rodents << endl;
       cout << "rats = " << rats << endl;
   
       return 0;
   }
   
   //rats = 101   
   //rodents = 101
   //rodents = 50 
   //rats = 50
   ```

   ```c++
   #include <iostream>
   
   using namespace std;
   
   void swap(int &a, int &b){
       int temp;
       temp = a;
       a = b;
       b = temp;
   }
   
   int main(){
       int a = 10, b = 20;
   
       cout << "a = " << a << endl;
       cout << "b = " << b << endl;
   
       swap(a, b);
   
       cout << "a = " << a << endl;
       cout << "b = " << b << endl;
   
       return 0;
   }
   
   //a = 10
   //b = 20
   //a = 20
   //b = 10
   ```

3. 将引用参数声明为常量数据的引用的理由有三个：

   1. 使用 const 可以避免无意中修改数据的编程错误；

   2. 使用 const使函数能够处理const和非 const 实参，否则将只能接受非 const 数据；

   3. 使用 const引用使函数能够正确生成并使用临时变量，假设实参的类型与引用参数类型不匹配，但可被转换为引用类型，程序将创建一个正确类型的临时变量，使用转换后的实参值来初始化它，然后传递一个指向该临时变量的引用:

      int a = 10;

      double &b = a; //错误

      const double &b = a; //正确

      ```c++
      #include <iostream>
      
      using namespace std;
      
      double sqrt1(double &n){
          return n*n;
      }
      
      double sqrt2(const double &n){
          return n*n;
      }
      int main(){
          double x = 5.0;
      
          //cout << sqrt1(5.0) << endl; 出错，只能引用变量
          cout << sqrt2(5.0) << endl; //5.0传递给临时变量，形参引用临时变量
      
          return 0;
      }
      
      //25
      ```

4. 函数返回引用的好处：

   1. 传统返回机制与按值传递函数参数类似:计算关键字 return 后面的表达式，并将结果返回给调用函数。从概念上说，这个值被复制到一个临时位置,而调用程序将使用这个值，例如下面的代码：

      double m = sqrt(16.0);

      cout << sqrt(25.0);

      在第一条语句值 4.0被复制到一个临时位置，然后被复制给 m。在第二条语句中，值 5.0 被复制到一个临时位置，然后被传递给cout(这里理论上的描述，实际上，编译器可能合并某些步骤)。

      来看下面的代码：

      ```c++
      #include <iostream>
      
      using namespace std;
      
      struct acc{
          int num1;
          int num2;
      };
      
      acc &accumulate(acc &target, const acc &stu);
      
      int main(){
          acc stu1 = {13, 14};
          acc stu2 = {10, 16};
          acc dup;
          dup = accumulate(stu2, stu1);
      
          cout << "dup.num1 = " << dup.num1 << endl;
          cout << "dup.num2 = " << dup.num2 << endl;
      
          return 0;
      }
      
      acc &accumulate(acc &target, const acc &stu){
          target.num1 += stu.num1;
          target.num2 += stu.num2;
          return target;
      }
      
      //dup.num1 = 23
      //dup.num2 = 30
      ```

      如果 accumulate()返回一个结构，而不是指向结构的引用，将把整个结构复制到一个临时位置，再将过个拷贝复制给 dup。但在返回值为引用时，将直接把 stu2 复制到 dup，其效率更高。

5. 使用引用变量时需要注意的问题：

   ```c++
   #include <iostream>
   #include <string>
   using namespace std;
   
   const string &version1(string &s1, const string &s2);
   const string &version2(string &s1, const string &s2);
   
   int main(){
       string str1 = "lixiaohui";
       string input;
       string result;
   
       getline(cin, input); //***
   
       result = version1(str1, input);
       cout << result << endl;
   
       result = version2(str1, input);
       cout << result << endl;
   
       return 0;
   }
   
   const string &version1(string &s1, const string &s2){
       s1 = s2 + s1 + s2;
       return s1;
   }
   
   const string &version2(string &s1, const string &s2){
       string temp; 
       temp = s2 + s1 + s2;
       return temp; //temp变量在函数结束后就被清除，而函数返回的是引用变量，不能对清除后的变量引用，所以报错
   }
   
   //***lixiaohui***
   //warning: reference to local variable 'temp' returned [-Wreturn-local-addr]
   ```

6. 对象、继承和**引用**：

   1. 派生类继承基类，可以使用基类的方法和特性；
   2. 基类引用可以指向派生类对象；

   ```c++
   #include <iostream>
   #include <fstream>
   using namespace std;
   
   void file_it(ostream &out, double objective, const double eps[], int n);
   
   int main(){
       
       fstream fout;
       const char *fn = "F:/C++Study/ep-data.txt";
   
       fout.open(fn);
       if(!fout.is_open()){
           cout << "无法打开文件" << fn << endl;
           exit(EXIT_FAILURE);
       }
   
       double objective;
       cout << "输入物镜的焦距:";
       cin >> objective;
       
       double eps[5];
       for(int i = 0; i < 5; i ++){
           cout << "输入第" << i + 1 << "个目镜的焦距:";
           cin >> eps[i];
       }
   
       file_it(cout, objective, eps, 5);
       file_it(fout, objective, eps, 5);
   
       return 0;
   }
   
   void file_it(ostream &out, double objective, const double eps[], int n){
       out << "物镜的焦距" << objective << endl;
       out << "目镜的焦距" << "放大倍数" << endl;
   
       for(int i = 0; i < n;i ++){
           out << eps[i] << "      " << int(objective / eps[i] + 0.5) << endl;
       }
   }
   
   ```

### 默认参数

1. 默认参数只能在函数声明中提供，而不是在函数定义中。要想使用默认参数，在函数原型中提供默认参数即可，但注意必须从右向左添加默认值。也就是说，要为某个参数设置默认值，则必须为它右边的所有参数提供默认值；

### 函数重载

1. 左值引用和右值引用：

   1. 左值引用 &：

      int a = 10;

      int b = 11;

      int &c = a; //valid

      int c = 10; //invalid

      const int &d = 10; //valid，引用的是系统分配的临时变量，此时只能读取数据，不能通过 d 修改数据

      **引用一旦绑定后不能改变**：在引用初始化时，它绑定到某个对象，并且不能再改变为引用其他对象。

      **引用总是指向它初始化时所绑定的对象**：即使该对象的值发生改变，引用始终指向相同的对象。

      ```c++
      int x = 10;
      int y = 20;
      int& ref = x;  // 引用 ref 绑定到 x
      
      ref = y;  // 这是赋值操作，而不是重新绑定引用
      // 现在 ref 指向 x, 但是 x 的值被修改为 y 的值，x 变成了 20
      
      cout << ref << endl;  // 输出 20
      cout << x << endl;    // 输出 20
      cout << y << endl;    // 输出 20
      
      ```

   2. 右值引用 &&：

      int &&c = 10; //valid
      
      1. 允许实现移动语义，优化资源管理，避免不必要的复制；
      
      2. 移动语义：移动构造函数和移动赋值运算符；
      
      3. 完美转发：利用右值引用来保持参数的值类别，从而实现高效的函数转发；
      
         ```c++
         #include <utility> // std::forward
         
         template<typename T>
         void process(T&& arg) {
             // 完美转发
             someFunction(std::forward<T>(arg));
         }
         
         void someFunction(int& arg) {
             std::cout << "Lvalue\n";
         }
         
         void someFunction(int&& arg) {
             std::cout << "Rvalue\n";
         }
         
         int main() {
             int x = 10;
             process(x);         // 调用 someFunction(int&)
             process(20);        // 调用 someFunction(int&&)
         
             return 0;
         }
         
         ```
      
         在这个例子中，process 函数使用 std::forward 完美转发其参数。std::forward 保留了传入参数的值类别（左值或右值），从而确保正确调用对应的重载函数。

2. 仅当函数基本上执行相同的任务，但使用不同形式的数据时,才应采用函数重载：

   ```c++
   #include <iostream>
   #include <fstream>
   using namespace std;
   
   char *left(const char *str, int n = 1);
   unsigned long *left(const long num, unsigned int ct);
   
   int main(){
       
       char s1[100];
       int n;
       cout << "输入一个字符串:" << endl;
       cin.get(s1, 100);
       cout << "输入一个数字n,表示你要返回字符串的前n个字符:" << endl;
       cin >> n;
       cout << left(s1, n) << endl;
   
       long num;
       int m;
       cout << "输入一个数字:" << endl;
       cin >> num;
       cout << "输入一个数字n,表示你要返回num的前n个字符:" << endl;
       cin >> m;
       cout << *left(num, m);
   
       return 0;
   }
   
   char *left(const char *str, int n){
       char *p = new char[n + 1];
       for(int i = 0; i < n; i++){
           p[i] = str[i];
       }
       p[n] = '\0';
     
       return p;
   }
   
   unsigned long *left(const long num, unsigned int ct){
       int k = 10;
       unsigned long *result = new unsigned long;
       int count = 1;
       while(num/k){
           count++;
           k = k * 10;
       }
       int k1 = count - ct;
       *result = num;
       while(k1--){
           *result = *result / 10;
       }
     
       return result;
   }
   ```

3. 通过使用默认参数来实现同样的目的；

   例如，可以用两个重载函数来代替面向字符串的left函数:

   char * left(const char*str,unsigned n); //two arguments

   char * left(const char*str); //one argument

   这种情况下使用一个带默认参数的函数要简单些。只需编写一个函数(而不是两个函数)，程序也只需为一个函数(而不是两个)请求内存；需要修改函数时，只需修改一个。然而，如果需要使用不同类型的参数，则默认参数便不管用了，在这种情况下，应该使用函数重载。

### 函数模板

1. 函数模板定义：

   template < typename AnyType >
   void func(AnyType a / AnyType *b / AnyType &c , ...);

   例如下面这个程序：

   ```c++
   #include <iostream>
   
   using namespace std;
   
   template <typename AnyType>
   void Swap(AnyType &a,AnyType &b);
   
   int main(){
       int a = 10, b = 20;
       double m = 11.11, n = 22.22;
   
       cout << "a = " << a << " b = " << b << endl;
       Swap(a, b);
       cout << "a = " << a << " b = " << b << endl;
   
       cout << "a = " << a << " b = " << b << endl;
       Swap(a, b);
       cout << "a = " << a << " b = " << b << endl;
   
       return 0;
   }
   
   template <typename AnyType>
   void Swap(AnyType &a, AnyType &b)
   {
       AnyType temp;
       temp = a;
       a = b;
       b = temp;
   }
   
   //a = 10 b = 20
   //a = 20 b = 10
   //m = 11.11 n = 22.22
   //m = 22.22 n = 11.11
   ```

2. 函数重载模板，定义多个具有相同名称但参数类型不同的函数模板：

   ```c++
   #include <iostream>
   
   using namespace std;
   
   template <typename AnyType>
   void Swap(AnyType &a,AnyType &b);
   
   template <typename AnyType>
   void Swap(AnyType *a, AnyType *b, int n);
   
   void show(const int s[], int n);
   
   int main(){
       int a = 10, b = 20;
       double m = 11.11, n = 22.22;
   
       cout << "a = " << a << " b = " << b << endl;
       Swap(a, b);
       cout << "a, b 交换之后:"  << endl;
       cout << "a = " << a << " b = " << b << endl;
   
       int d1[4] = {2, 4, 6, 8};
       int d2[4] = {1, 3, 5, 7};
   
       cout << "输出数组 d1 的元素:" << endl;
       show(d1, 4);
       cout << "输出数组 d2 的元素:" << endl;
       show(d2, 4);
   
       Swap(d1, d2, 4);
       cout << "d1, d2 交换之后:" << endl;
       cout << "输出数组 d1 的元素:" << endl;
       show(d1, 4);
       cout << "输出数组 d2 的元素:" << endl;
       show(d2, 4);
   
       return 0;
   }
   
   void show(const int s[], int n){
       for(int i = 0; i < n; i++){
           cout << "s[" << i << "] = " << s[i] << endl;
       }
   }
   
   template <typename AnyType>
   void Swap(AnyType &a, AnyType &b)
   {
       AnyType temp;
       temp = a;
       a = b;
       b = temp;
   }
   
   template <typename AnyType>
   void Swap(AnyType *a, AnyType *b, int n){
       int *temp = new int[n];
       for(int i = 0; i < n; i++){
           temp[i] = a[i];
       }
       for(int i = 0; i < n; i++){
           a[i] = b[i];
       }
       for(int i = 0; i < n; i++){
           b[i] = temp[i];
       }
   }
   ```

3. 模板的局限性：

   假设有如下模板函数:
   template < class T > //or template < typename T >

   void f(T a,T b){...}

   通常，代码假定可执行哪些操作。例如，下面的代码假定定义了赋值，但如果T为数组，这种假设将不成立:

   a = b;

   同样，下面的语句假设定义了<，但如果T为结构，该假设便不成立:
   if(a >b)

   另外，为数组名定义了运算符>，但由于数组名为地址，因此它比较的是数组的地址，而这可能不是您希望的。下面的语句假定为类型T定义了乘法运算符，但如果T为数组、指针或结构，这种假设便不成立:
   T c= a * b；

   总之，编写的模板函数很可能无法处理某些类型。另一方面，有时候通用化是有意义的，但C++语法不允许这样做。例如，将两个包含位置坐标的结构相加是有意义的，虽然没有为结构定义运算符+。一种解决方案是，C++允许您重载运算符+,以便能够将其用于特定的结构或类(运算符重载将在第 11 章讨论)，这样使用运算符 + 的模板便可处理重载了运算符 + 的结构。另一种解决方案是，为特定类型提供具体化的模板定义，下面就来介绍这种解决方案（显式具体化）：

   ```c++
   #include <iostream>
   
   using namespace std;
   
   struct job{
       char name[10];
       double salary;
       int floor;
   };
   
   template <typename AnyType>
   void Swap(AnyType &a, AnyType &b);
   
   template <> 
   void Swap<job>(job &a, job &b);
   
   int main(){
       int a = 10, b = 20;
   
       cout << "a = " << a << " b = " << b << endl;
       Swap(a, b);
       cout << "a, b 交换之后:"  << endl;
       cout << "a = " << a << " b = " << b << endl;
   
       job m = {"li", 22.22, 22};
       job n = {"zhang", 33.33, 33};
   
       cout << " m.name = " << m.name << " m.salary = " << m.salary << " m.floor = " << m.floor << endl;
       cout << " n.name = " << n.name << " n.salary = " << n.salary << " n.floor = " << n.floor << endl;
       Swap(m, n);
       cout << "m, n 交换之后:"  << endl;
       cout << " m.name = " << m.name << " m.salary = " << m.salary << " m.floor = " << m.floor << endl;
       cout << " n.name = " << n.name << " n.salary = " << n.salary << " n.floor = " << n.floor << endl;
   
       return 0;
   }
   
   template <typename AnyType>
   void Swap(AnyType &a, AnyType &b)
   {
       AnyType temp;
       temp = a;
       a = b;
       b = temp;
   }
   
   template <> 
   void Swap<job>(job &a, job &b){
       double t1;
       int t2;
   
       t1 = a.salary;
       a.salary = b.salary;
       b.salary = t1;
   
       t2 = a.floor;
       a.floor = b.floor;
       b.floor = t2;
   }
   
   ```

4. 对于给定的函数名，可以有非模板函数、模板函数和显式具体化模板函数以及它们的重载版本，显式具体化的原型和定义应以 < template >打头，并通过名称来指出类型。

   1. 具体化优先于常规模板；
   2. 非模板函数优先于具体化和常规模板；

5. 显示实例化：

   最初，编译器只能通过隐式实例化,来使用模板生成函数定义,但现在 C++还允许显式实例化(explicitinstantiation)。这意味着可以直接命令编译器创建特定的实例，如Swap< int >()。其语法是，声明所需的种类——用<>符号指示类型，并在声明前加上关键字template:

   template void Swap< int >(int,int);

   实现了这种特性的编译器看到上述声明后，将使用Swap()模板生成一个使用int 类型的实例。也就是该声明的意思是使用 Swap( )模板生成 int 类型的函数定义。

6. 显式具体化:

   与显式实例化不同的是，显式具体化使用下面两个等价的声明之一:

   template <> void Swap< int >(int &,int &);

   template <>void Swap(int &,int &);

   区别在于显示实例化一开始就定义了适用于某种特定类型的函数（占内存）；而显式具体化相当于是通用模板的一个副本，意在告诉编译器，如果需要为某种特定类型生成函数定义，不要使用通用模板，而应该使用具体化的模板，反之使用通用模板来生成定义函数，后者也叫隐式实例化（不使用就不占）；

   试图在同一个文件(或转换单元)中使用同一种类型的显式实例和显式具体化将出错；

7. 函数匹配调用：

   1. 如果函数模板的模板参数可以从函数参数自动推导出来，则调用时可以省略尖括号 <>;如果编译器无法推导出模板参数，或者你想显式指定模板参数，则调用时需要在函数名后面加上尖括号 <> 并指定模板参数;

   2. 完全匹配和最佳匹配：

      假设有下面的函数代码:

      struct blot {int a;char b[10];}；

      blot ink={25，"spots"};

      ...

      recycle(ink);

      在这种情况下，下面的原型都是完全匹配的:

      void recycle(blot); //#1 blot-to-blot

      void recycle(const blot);//#2 blot-to-(const blot)

      void recycle(blot &); //#3 blot-to-(blot &)

      void recycle(const blot&);//#4 blot-to-(const blot &)

      正如您预期的，如果有多个匹配的原型，则编译器将无法完成重载解析过程;如果没有最佳的可行函数，则编译器将生成一条错误消息，该消息可能会使用诸如“ambiguous(二义性)”这样的词语。然而，有时候，即使两个函数都完全匹配，仍可完成重载解析。首先，指向非const 数据的指针和引用优先与非 const指针和引用参数匹配。也就是说，在recycle()示例中，如果只定义了函数#3 和#4是完全匹配的，则将选择#3，因为ink没有被声明为const。然而，const和非const之间的区别只适用于指针和引用指向的数据。也就是说，如果只定义了#1和#2，则将出现二义性错误。

      一个完全匹配优于另一个的另一种情况是，其中一个是非模板函数，而另一个不是。在这种情况下，非模板函数将优先于模板函数(包括显式具体化)。

   3. 函数最优模板匹配：

      如果两个完全匹配的函数都是模板函数，则较具体的模板函数优先。例如，这意味着显式具体化将优于使用模板隐式生成的具体化:

      ```c++
      #include <iostream>
      
      using namespace std;
      
      typedef struct{
          char name[10];
          double amount;
      }debts;
      
      template <typename AnyType>
      void showarry(AnyType arr[], int n);
      
      template <typename AnyType>
      void showarry(AnyType *arr[], int n);
      
      int main(){
          
          int things[6] = {1, 2, 3, 4, 5, 6};
          debts mr_E[3] = {
              {"li", 2},
              {"xiao", 3},
              {"hui", 4}
          };
          double *pd[3];
      
          for(int i = 0; i < 3; i++){
              pd[i] = &mr_E[i].amount;
          }
      
          showarry(things, 3);
          showarry(pd, 3);
      
          return 0;
      }
      
      template <typename AnyType>
      void showarry(AnyType arr[], int n){
          cout << "templete A:" << endl;
      
          for(int i = 0; i < n; i++){
              cout << arr[i] << " ";
          }
      
          cout << endl;
      }
      
      template <typename AnyType>
      void showarry(AnyType *arr[], int n){
          cout << "templete B:" << endl;
      
          for(int i = 0; i < n; i++){
              cout << arr[i] << " ";
          }
      
          cout << endl;
      }
      
      //templete A:
      //1 2 3
      //templete B:
      //2 3 4
      ```

   4. 自己选择：

      ```c++
      #include <iostream>
      #include <cmath>
      
      using namespace std;
      
      template <typename T>
      T lesser(T a, T b){
          return a < b ? a : b;
      }
      
      int lesser(int a, int b){
          return a < b ? abs(a) : abs(b);
      }
      
      int main(){
      
          int a = 10, b = 20;
          double m = 20.1, n = 30;
      
          cout << lesser(a, b) << endl;
          cout << lesser(m, n) << endl;
      
          cout << lesser<>(m, n) << endl; //指定模板函数
          cout << lesser<int>(m, n) << endl; //指定模板函数,并强制类型转换
      
          return 0;
      }
      
      //10  
      //20.1
      //20.1
      //20  
      ```

8. template < typename T1, typename T2 >

   void gt(T1 x, T2 y){

   ​	?type? xpy = x + y;

   }

   此时不知道 xpy 该定义为什么类型；

   decltype(expression) var：

   1. 如果expression 是一个没有用括号括起的标识符，则 var 的类型与该标识符的类型相同，包括 const 等限定符:
      
      const double *pd;
      
      decltype(pd) v; //v is type const double *
      
   2. 如果 expression 是一个函数调用，则 var 的类型与函数的返回类型相同:
   
      long indeed(int)；
   
      decltype(indeed(3)) m;//m is type long
   
   3. expression是用括号括起来的标识符：
   
      double xx = 4.4;
   
      decltype (xx) w = xx; //w is double(stage 1 match)
   
      decltype ((xx)) r2 = xx;//r2 is double &
   
      顺便说一句，括号并不会改变表达式的值和左值性。例如，下面两条语句等效:
   
      xx= 98.6;
   
      (xx)=98.6; // () don't affect use of xx
   
   4. 如果前面的条件都不满足，则var的类型与expression的类型相同；
   
      int &k = 3, &n = 4;
   
      decltype(k + n) m; // m is int
   
9. template < typename T1, typename T2 >

   ?type? gt(T1 x, T2 y){

   ​	return x + y;

   }

   此时不知道返回类型是什么；

   template < typename T1, typename T2 >

   auto gt(T1 x, T2 y) -> decltype (x + y){

   ​	return x + y;

   }

   现在，decltype 在参数声明后面，因此x和y位于作用域内，可以使用它们。



## 内存模型和名称空间

1. 单独编译：
   1. 头文件中常包含的内容：
      1. 函数原型：
      2. 使用 #define 或 const 定义的符号常量：
      3. 结构声明：
      4. 类声明：
      5. 模板声明：
      6. 内联函数：除内联函数外，请不要将函数定义或变量声明放到头文件中。这样做对于简单的情况可能是可行的，但通常会引来麻烦。例如，如果在头文件包含一个函数定义，然后在其他两个文件(属于同一个程序)中包含该头文件，则同一个程序中将包含同一个函数的两个定义，除非函数是内联的，否则这将出错。

   2. 源代码文件:包含与结构有关的函数的代码、包含调用与结构相关的函数的代码。 

   3. 简单示例：

      1. 创建coordin.h文件：

         ```c++
         #ifndef __COORDIN_H__ //检查某个标识符是否已经被 #define 定义过
         #define __COORDIN_H__
         
         struct polar{
             double distance;
             double angle;
         };
         
         struct rect{
             double x;
             double y;
         };
         
         polar rect_to_polar(rect xypos);
         void show_polar(polar dapos);
         
         #endif
         ```

      2. 创建file2.cpp文件：

         ```c++
         #include <iostream>
         #include <cmath>
         #include "coordin.h"
         
         using namespace std;
         
         polar rect_to_polar(rect xypos){
         
             polar answer;
             answer.distance=sqrt(xypos.x * xypos.x + xypos.y * xypos.y);
             answer.angle= atan2(xypos.y, xypos.x);
             
             return answer;
         }
         void show_polar(polar dapos){
         
             const double Rad_to_deg =57.29577951;
             cout << "Distance" << dapos.distance << endl;
             cout << "Angle =" << dapos.angle * Rad_to_deg << " degree " << endl;
         
         }
         
         ```

      3. 创建file1.cpp文件：

         ```c++
         #include <iostream>
         #include "coordin.h"
         
         using namespace std;
         
         int main(){
             rect rplace;
             polar pplace;
         
             cout << "Enter the x and y values: ";
         
             while(cin >> rplace.x >> rplace.y){
                 pplace = rect_to_polar(rplace);
                 show_polar(pplace);
                 cout << "Next two number (q to quit):";
             }
         
             return 0;
         }
         ```

### 存储持续性、作用域和链接性

1. 存储持续性：

   1. 自动存储持续性:在函数定义中声明的变量(包括函数参数)存储持续性为自动的。它们在程序开始执行其所属的函数或代码块时被创建，在执行完函数或代码块时，它们使用的内存被释放。C++有两种存储持续性为自动的变量；
   2. 静态存储持续性:在函数定义外定义的变量和使用关键字static 定义的变量的存储持续性都为静态。它们在程序整个运行过程中都存在。C++有3种存储持续性为静态的变量；
   3. 线程存储持续性(C++11):当前，多核处理器很常见，这些CPU可同时处理多个执行任务。这让程序能够将计算放在可并行处理的不同线程中。如果变量是使用关键字thread_local 声明的，则其生命周期与所属的线程一样长。本书不探讨并行编程；
   4. 动态存储持续性:用new运算符分配的内存将一直存在，直到使用delete运算符将其释放或程序结束为止。这种内存的存储持续性为动态，有时被称为自由存储(freestore)或堆(heap)；

2. 静态持续性

   1. extern 关键字：用于声明一个全局变量或函数，但不定义它。它告诉编译器该变量或函数是在其他文件中定义的，需要在链接时进行解析；

      ```c++
      // file1.cpp
      #include <iostream>
      
      int globalVar = 42;  // 定义全局变量
      
      void printGlobalVar() {
          std::cout << "globalVar in file1.cpp: " << globalVar << std::endl;
      }
      ```
   
      ```c++
      // file2.cpp
      #include <iostream>
      
      extern int globalVar;  // 声明全局变量，但不定义
      
      void modifyGlobalVar() {
          globalVar += 10;
      }
      ```
   
      ```c++
      // main.cpp
      #include <iostream>
      
      extern void printGlobalVar();
      extern void modifyGlobalVar();
      
      int main() {
          printGlobalVar();    // 输出 42
          modifyGlobalVar();   // 修改全局变量
          printGlobalVar();    // 输出 52
          return 0;
      }
      ```
   
   2. 外部变量：可使用外部变量在多文件程序的不同部分之间共享数据，注意的是函数体内若声明了和全局变量名字相同的变量，在函数体内要访问全局变量，在变量名之前加 :: 即可；
   
   3. static 关键字：可使用链接性为内部的静态变量在同一个文件中的多个函数之间共享数据(名称空间提供了另外一种共享数据的方法)。另外，如果将作用域为整个文件的变量变为静态的，就不必担心其名称与其他文件中的作用域为整个文件的变量发生冲突，在代码块中使用 static 时，将导致局部变量的存储持续性为静态的。这意味着虽然该变量只在该代码块中可用，但它在该代码块不处于活动状态时仍然存在。因此在两次函数调用之间,静态局部变量的值将保持不变。(静态变量适用于再生-可以用它们将瑞士银行的秘密账号传递到下一个要去的地方)。另外，如果初始化了静态局部变量，则程序只在启动时进行一次初始化。以后再调用函数时，将不会像自动变量那样再次被初始化；
   
      ```c++
      #include <iostream>
      
      using namespace std;
      
      const int ArSize = 10;
      void strcount(const char *str);
      
      int main(void)
      {
          char input[ArSize];
          char next;
      
          cout << "Enter a line:" << endl;
          cin.get(input, ArSize);
          while(cin)
          {
              cin.get(next);
              while(next != '\n')
              {
                  cin.get(next);
                  cout << "Enter next line (empty line to quit): " << endl;
              }
              strcount(input);
              cin.get(input, ArSize);
      
          }
          cout << "ByeBye" << endl;
      
          return 0;
      }
      
      void strcount(const char *str)
      {
          static int total = 0;
          int count = 0;
          while(*str)
          {
              count++;
              str++;
          }
          total += count;
          cout << count << " characters" << endl;
          cout << total << " characters total" << endl;
      }
      ```
   
   4. const 关键字：在默认情况下全局变量的链接性为外部的，但 const全局变量的链接性为内部的。也就是说，在C++看来，全局const定义就像使用了 static 说明符一样。例如，假设将一组常量放在头文件中，并在同一个程序的多个文件中使用该头文件。那么，预处理器将头文件的内容包含到每个源文件中后，所有的源文件都将包含类似下面这样的定义:
   
      const int fingers = 10;
   
      如果全局const声明的链接性像常规变量那样是外部的，则根据单定义规则，这将出错。也就是说，只能有一个文件可以包含前面的声明，而其他文件必须使用 extem 关键字来提供引用声明。另外，只有未使用 extem 关键字的声明才能进行初始化；
   
   5. new 运算符初始化：
   
      1. int * pi = new int(6); //* pi set to 6
   
      2. double * pd = new double(99.99);//* pd set to 99.99
   
      3. struct where{double x;double y; double z;}; 
   
         where * one = new where {2.5，5.3，7.2};//C++11
   
      4. int * ar = new int [4] {2,4,6,7);// C++11
   
      5. 在C++11中，还可将列表初始化用于单值变量：double * pdo = new double {99.99}; // * pd set to 99.99
   
   6. new 运算符开辟内存：
   
      ```c++
      #include <iostream>
      #include <new>
      
      using namespace std;
      
      const int BUF = 512;
      const int N = 5;
      char buffer[BUF];
      
      int main(void)
      {
          double *pd1, *pd2;
      
          cout << "Calling new and placement new: " << endl;
          pd1 = new double [N];
          pd2 = new(buffer) double [N];
          for(int i = 0; i < N; i++)
          {
              pd2[i] = pd1[i] = 1000 + 20.0 * i;
          }
          cout << "pd1 = " << pd1 << ", buffer = " << (void *)buffer << endl;
          for(int i = 0; i < N; i++)
          {
              cout << pd1[i] << " at " << &pd1[i] << "; ";
              cout << pd2[i] << " at " << &pd2[i] << endl;
          }
      
          cout << "\nCalling new and placement new a second time: " << endl;
          double *pd3, *pd4;
          pd3 = new double [N];
          pd4 = new(buffer) double [N];
          for(int i = 0; i < N; i++)
          {
              pd4[i] = pd3[i] = 1000 + 40.0 * i;
          }
          for(int i = 0; i < N; i++)
          {
              cout << pd3[i] << " at " << &pd3[i] << "; ";
              cout << pd4[i] << " at " << &pd4[i] << endl;
          }
      
          cout << "\nCalling new and placement new a third time: " << endl;
          delete[] pd1;
          pd1 = new double [N];
          pd2 = new(buffer + N*sizeof(double)) double [N];
          for(int i = 0; i < N; i++)
          {
              pd2[i] = pd1[i] = 1000 + 20.0 * i;
          }
          for(int i = 0; i < N; i++)
          {
              cout << pd1[i] << " at " << &pd1[i] << "; ";
              cout << pd2[i] << " at " << &pd2[i] << endl;
          }
      
          delete[] pd1;
          delete[] pd3;
        //bufer 指定的内存是静态内存，而 delete 只能用于删除动态分配的内存，因此对于 buffer 指定的内存，不需要 delete；
          return 0;
      }
      ```


### 名称空间

1. 声明区域是可以在其中进行声明的区域。例如，可以在函数外面声明全局变 量，对于这种变量，其声明区域为其声明所在的文件。对于在函数中声明的变量，其声明区域为其声明所在的代码块潜在作用域：变量的潜在作用域从声明点开始，到其声明区域的结尾。因此潜在作用域比声明 区域小，这是由于变量必须定义后才能使用。然而，变量并非在其潜在作用域内的任何位置都是可见的。例如， 它可能被另一个在嵌套声明区域中声明的同名变量隐藏。例如，在函数中声明的局部变量（对于这种变量，声明区域为整个函数）将隐藏在同 一个文件中声明的全局变量（对于这种变量，声明区域为整个文件）。变量对程序而言可见的范围被称为作用域（scope），前面正是以这种 方式使用该术语的。

2. using声明和using编译指令：

   1. using声明：using声明由被限定的名称和它前面的关键字using组成，using声明将特定的名称添加到它所属的声明区域中。

      1. namespace Jill{

         ​	double fetch;

         }

         char fetch;

         int main(){

         ​	using Jill::fetch; //local namespace

         ​	double fetch; //Error!

         ​	cin >> fetch; //Jill::fetch

         ​	cin >> ::fetch; //global fetch;

         }

      2. namespace Jack{

         ​	double fetch;

         }

         using Jack::fetch; // global fetch

         int main(){...}

   2. using编译:using编译指令由名称空间名和它前面的关键字using namespace组 成，它使名称空间中的所有名称都可用，而不需要使用作用域解析运算符;

   3. 有关using编译指令和using声明，需要记住的一点是，它们增加了 名称冲突的可能性。也就是说，如果有名称空间jack和jill，并在代码中 使用作用域解析运算符，则不会存在二义性：

      Jack::fetch = 3;

      Jill::fetch = 3;

      变量jack::pal和jill::pal是不同的标识符，表示不同的内存单元。然而，如果使用using声明，情况将发生变化：

      using Jack::fetch;

      using Jill::fetch;

      fetch = 3;

      事实上，编译器不允许您同时使用上述两个using声明，因为这将导 致二义性。

   4. using编译指令和using声明之比较:使用using编译指令导入一个名称空间中所有的名称与使用多个 using声明是不一样的，而更像是大量使用作用域解析运算符。使用 using声明时，就好像声明了相应的名称一样。如果某个名称已经在函数 中声明了，则不能用using声明导入相同的名称。然而，使用using编译 指令时，将进行名称解析，就像在包含using声明和名称空间本身的最小声明区域中声明了名称一样。

      namespace Jill{

      ​	double fetch;

      ​	struct Hill{...};

      }

      char fetch;

      int main(){

      ​	using namespace Jill;

      ​	double fetch; //vaild

      ​	cin >> fetch; //local fetch

      ​	cin >> ::fetch; //global fetch

      ​	cin >> Jill::fetch; //Jill::fetch

      }

      虽然函数中的using编译指令将名称空间的名称视为在函数之外声明的，但它不会使得该文件中的其他函数能够使用这些名称。

      int foom(){

      ​	Hill top; //ERROR!

      }

      一般说来，使用using声明比使用using编译指令更安全，这是由于 它只导入指定的名称。如果该名称与局部名称发生冲突，编译器将发出 指示。using编译指令导入所有名称，包括可能并不需要的名称。如果与 局部名称发生冲突，则局部名称将覆盖名称空间版本，而编译器并不会 发出警告。另外，名称空间的开放性意味着名称空间的名称可能分散在 多个地方，这使得难以准确知道添加了哪些名称。

3. 名称空间的其他特性:嵌套...


```c++
//namesp.h
#include <string>

namespace pers
{
    struct Person
    {
        std::string fname;
        std::string lname;
    };

    void getPerson(Person &rp);
    void showPerson(const Person  &rp);
}

namespace debts
{
    using namespace pers;
    struct Debt
    {
        Person name;
        double amount;
    };

    void getDebt(Debt &rd);
    void showDebt(const Debt &rd);
    double sumDebts(const Debt arr[], int n);
}


```

```c++
//namesp.cpp

#include <iostream>
#include "namesp.h"

namespace pers
{
    using std::cout;
    using std::cin;

    void getPerson(Person &rp)
    {
        cout << "Enter first name: ";
        cin >> rp.fname;
        cout << "Enter last name: ";
        cin >> rp.lname;
    }
    void showPerson(const Person &rp)
    {
        cout << rp.fname << ", " << rp.lname;
    }
}

namespace debts
{
    using std::cout;
    using std::cin;

    void getDebt(Debt &rd)
    {
        getPerson(rd.name);
        cout << "Enter debt: ";
        cin >> rd.amount; 
    }
    void showDebt(const Debt &rd)
    {
        showPerson(rd.name);
        cout << ": $" << rd.amount << std::endl;
    }
    double sumDebts(const Debt arr[], int n)
    {
        double total = 0;
        for(int i = 0; i < n; i++)
            total += arr[i].amount;

        return total;
    }
}
```

```c++
//namessp.cpp

#include <iostream>
#include "namesp.h"

int main(void)
{
    using debts::Debt;
    using debts::showDebt;

    Debt golf = {{"li", "xiaohui"}, 120.0};
    showDebt(golf);

    return 0;
}
```



## 对象和类

### 抽象和类

1. 类：

   1. 访问控制：关键字 private 和 public 它们描述了对类成员的访问控制，类的默认访问权限是私 有的。使用类对象的程序都可以直接访问公有部分，但只能通过公有成员函数(或友元函数，参见第11章)来访问对象的私有成员。
   2. 控制对成员的访问：无论类成员是数据成员还是成员函数，都可以在类的公有部分或私有部分中声明它。但由于隐藏数据是 OOP主要的目标之一，因此数据项通常放在私有部分，组成类接口的成员函数放在公有部分；否则，就无法从程序中调用这些函数。

   ```c++
   //stock00.h
   
   #ifndef __STOCK00_H__
   #define __STOCK00_H__
   
   #include <string>
   
   class __STOCK00_H__
   {
       private:
               std::string company;
               long shares;
               double share_val;
               double total_val;
               void set_tot{total_val = shares * share_val};
       public:
               void acquire(const std::string &co, long n, double price);
               void buy(long num, double price);
               void sell(long num, double price);
               void update(double price);
               void show();
   };
   
   #endif
   ```

2. 类成员函数：

   1. 在类声明和类方法之外定义成员函数时，使用作用域解析运算符(::)来标识函数所属的类;

   2. 类方法可以访问类的 private 组件；

   3. 其定义位于类声明中的函数都将自动成为内联函数，类声明常将短小的成员函数作为内联函数(如下述代码的set_tot())，内联函数的特殊规则要求在每个使用它们的文件中都对其进行定义。确保内联定义对多文件程序中的所有文件都可用的、最简便的方法是:将内联定义放在定义类的头文件中(有些开发系统包含智能链接程序，允许将内联定义放在一个独立的实现文件)；

      ```c++
      #include <iostream>
      #include "stock00.h"
      
      
      void Stock::acquire(const std::string &co, long n, double price)
      {
          company = co;
          if(n < 0)
          {
              std::cout << "Number of shares can't be negative: " << company << " shares set to be 0." << std::endl;
              shares = 0;
          }
          else
              shares = n;
      
          share_val = price;
      
          set_tot();
      }
      void Stock::buy(long num, double price)
      {
          if(num < 0)
          {
              std::cout << "Number of shares can't be negative: Transaction is aborted" << std::endl;
          }
          else
          {
              shares += num;
              share_val = price;
              set_tot();
          }
      }
      void Stock::sell(long num, double price)
      {
          if(num < 0)
          {
              std::cout << "Number of shares can't be negative: Transaction is aborted" << std::endl;
          }
          else if(num > shares)
              {
                  std::cout << "You can't sell more than you have! Transaction is aborted" << std::endl;
          }
          else
          {
              shares -= num;
              share_val = price;
              set_tot();
          }
      }
      void Stock::update(double price)
      {
          share_val = price;
          set_tot();
      }
      void Stock::show()
      {
          std::cout << "Company: " << company << std::endl;
          std::cout << "Shares: " << shares << std::endl;
          std::cout << "Share price: " << share_val << std::endl;
          std::cout << "Total worth: " << total_val << std::endl;
          std::cout << "End of show function" << std::endl; // 添加这行作为调试信息
      
      }
      
      ```

3. 类调用：

   ```c++
   #include <iostream>
   #include "stock00.h"
   
   using namespace std;
   
   int main()
   {
       Stock fluffy_the_cat;
   
       fluffy_the_cat.acquire("NanoSmart", 20, 12.5);
   
       fluffy_the_cat.show();
   
       return 0;
   }
   
   ```

### 类的构造函数

1. 函数名和类的名字相同；

2. 声明：Stock(const std::string &co, long n, double price);

3. 定义：

   ```C++
   Stock::Stock(const std::string &co, long n, double pr)
   {
       company = co;
       if(n < 0)
       {
           std::cout << "Number of shares can't be negative: " << company << " shares set to be 0." << std::endl;
           shares = 0;
       }
       else
           shares = n;
   
       share_val = pr;
   
       set_tot();
   }
   
   ```

4. 隐式和显式初始化对象

   Stock stock1("Nanosmart", 12, 20.0);

   Stock stock2 = Stock("Boffo objects", 2, 2.0);

   如果构造函数只有一个参数，则将对象初始化为一个与参数的类型相同的值时，该构造函数将被调用：

   Bozo(int k);

   Bozo tubby = 32; //也可以进行初始化对象，不推荐使用；

5. 默认构造函数：

   Stock stock3; 

   Stock stock3 = Stock();

   Stock *perlief = new Stock;

   Stock first("sxu"); //不会调用默认构造函数，初始化第一个形参，其他形参使用默认值；

   Stock second(); //这是函数，返回值是类，不会调用默认构造函数

   程序员如果没有提供任何构造函数，c++将提供默认的构造函数，默认构造函数没有参数，不进行任何操作；当且仅当没有定义任何构造函数时，编译器才会提供默认构造函数。

   如果程序员提供了非默认构造函数，但程序员没有显示的定义默认构造函数，则上面的声明将出错；

   提供默认构造函数的方法：

   1. 给已有的构造函数形参加默认变量，此操作之后不能再定义 b 中的默认构造函数；

   2. 声明：Stock();

      定义：Stock::Stock()
      	{
      		company = "no name";
      		shares = 0;
      		share_val = 0.0;
      		total_val = 0.0;
      	}

### 类的析构函数

1. 由于在类对象过期时析构函数将自动被调用，因此必须有一个析构函数。如果程序员没有提供析构函数，编译器将隐式地声明一个默认析构函数，并在发现导致对象被删除的代码后（编译器在需要时才会定义默认析构函数。这通常发生在对象即将被销毁的地方，例如当对象超出作用域、被删除或程序结束时），提供默认析构函数的定义。

   声明：~Stock();

   定义：Stock::~Stock()
   	{
   		std::cout << "Bye " << company << std::endl; //函数体内可以为空
   	}

   如果构造函数使用了new，则必须提供使用delete 的析构函数。

2. const 关键字：

   const Stock land = Stock("proper", 3, 3.0);
   land.show(); //error: passing ‘const Stock’ as ‘this’ argument discards qualifiers [-fpermissive]， show() 不能保护 const 定义过的 land 的私有成员；

   因此，对 show 函数声明和定义成 const 成员函数：

   声明：void Stock::show() const；

   定义：void Stock::show() const{...}；

### this 指针

一般来说，所有的类方法都将this 指针设置为调用它的对象的地址，* this返回对象的全部内容。确实，topval()中的total_val 只不过是 this->total val 的简写；

```c++
const Stock & Stock::topval(const Stock &s) const
{
	if(s.total_val > total_val)
		return s;
	else
		return *this;
}
```

### 类的数组：

1. Stock mystuff[4]; //当程序创建未被显式初始化的类对象时，总是调用默认构造函数；

2. 可以用构造函数来初始化数组元素。在这种情况下，必须为每个元素调用构造函数:

   Stock stocks[STKS] = {
   		Stock("NanoSmart", 12, 20.0),
   		Stock("Boffo Objects", 200, 2.0),
   		Stock("Mono", 130, 3.25),
   		Stock("Fleep", 60, 6.5),
   };

3. 下述代码使用 Stock(const string& co, long n, double pr)初始化 stock[0]和 stock[2]，使用构造函数 Stock()初始化 stock[1]。由于该声明只初始化了数组的部分元素，因此余下的7个元素将使用默认构造函数进行初始化：

   const int STKS=10;

   Stock stocks[STKS]={

   ​	Stock("Nanosmart"，12.5,20),

   ​	Stock()，

   ​	Stock("Monolithic Obelisks"，130，3.25)

   }

### 类作用域

类声明可能使用字面值 30 来指定数组的长度，由于该常量对于所有对象来说都是相同的，因此创建一个由所有对象共享的常量是个不错的主意。您可能以为这样做可行:

class Bakery

{

private:
	const int Months=12; //declare a constant? FAILS

​	double costs [Months];

​	...

但这是行不通的，因为声明类只是描述了对象的形式，并没有创建对象。因此，在创建对象前，将没有用于存储值的空间;

1. 枚举：

   class Bakery

   {

   private:
   	enum {Months = 12};

   ​	double costs [Months];

   ​	...

   第一种方式是在类中声明一个枚举。在类声明中声明的枚举的作用域为整个类，因此可以用枚举为整型常量提供作用域为整个类的符号名称。注意，用这种方式声明枚举并不会创建类数据成员。也就是说，所有对象中都不包含枚举。另外，Months只是一个符号名称，在作用域为整个类的代码中遇到它时，编译器将用来替换它。由于这里使用枚举只是为了创建符号常量，并不打算创建枚举类型的变量，因此不需要提供枚举名。

2. 关键字 static：

   class Bakery

   {

   private:
   	static const int Months = 12;

   ​	double costs [Months];

   ​	...

   这将创建一个名为 Months的常量，该常量将与其他静态变量存储在一起，而不是存储在对象中。因此，只有一个 Months 常量，被所有 Bakery 对象共享。

### 抽象数据类型

类非常具体。然而，程序员常常通过定义类来表示更通用的概念。例如，使用类来表示栈：

```c++
//stack.h

#ifndef __STACK_H_
#define __STACK_H_

typedef unsigned long Item;

class Stack
{
	private:
		enum {MAX = 10};
		Item items[MAX];
		int top = 0;
	public:
		Stack();
		bool isempty() const;
		bool isfull() const;
		bool push(Item &item);
		bool pop(Item &item);
};

#endif
```

```c++
//stack.cpp

#include "stack.h"

Stack::Stack()
{

}

bool Stack::isempty() const
{
	return top == 0;
}

bool Stack::isfull() const
{
	return top == MAX;
}

bool Stack::push(Item &item)
{
	if(top < MAX)
	{
		items[top++] = item;
		return true;
	}
	else
		return false;
}

bool Stack::pop(Item &item) 
{
	if(top > 0)
	{
		item = items[--top];
		return true;
	}
	else
	{
		return false;
	}
}
```

```c++
//usestack.cpp

#include <iostream>
#include <cctype>
#include "stack.h"

using namespace std;

int main()
{
	char ch;
	unsigned long po;
	Stack st;

	cout << "Please enter A to push to stack,\n"
		 << "P to pop from stack,\n"
		 << "Q to quit.\n";

	while (cin >> ch && toupper(ch) != 'Q')
	{
		while(cin.get() != '\n')
			continue;
		switch(ch)
		{
			case 'A':
			case 'a': 
					  cout << "Enter a number you want to push to stack:\n";
					  cin >> po;
					  if(st.isfull())
						  cout << "Stack already full" << endl;
					  else
						  st.push(po);
					  break;
			case 'P':
			case 'p':
					  if(st.isempty())
						  cout << "Stack is empty" << endl;
					  else
					  {
						  st.pop(po);
						  cout << po << " is poped" << endl;
					  }
					  break;
		}

		cout << "Please enter A to push to stack,\n"
			 << "P to pop from stack, \n";
	}

	return 0;
}
```

## 使用类

### 运算重载符

operator**op**(argument-list）：

1. 假设有一个 Salesperson类，并为它定义了一个operator + ( )成员函数，以重载+运算 符，以便能够将两个Saleperson对象的销售额相加，则如果district2、sid 和sara都是Salesperson类对象，便可以编写这样的等式：

   1. district2 = sid + sara; //方便，也可以连续加好几个同类对象
   2. district2 = sid.operator+(sara);

2. op必须是有效的C++运算符，不能虚构一个新的符号；

3. 重载后的运算符必须至少有一个操作数是用户定义的类型，这将防止用户为标准类型重载运算符。

4. 使用运算符时不能违反运算符原来的句法规则。例如，不能将求模运算符（%）重载成使用一个操作数

   Time s;

    % s; //invalid

5. 同样，不能修改运算符的优先级。因此，如果将加号运算符重载成 将两个类相加，则新的运算符与原来的加号具有相同的优先级。

6. 不能重载下面的运算符：

   1. sizeof：sizeof运算符；
   2. .：成员运算符；
   3. . *：成员指针运算符；
   4. ::：作用域解析运算符；
   5. ?:：条件运算符；
   6. typeid：一个RTTI运算符；
   7. const_cast：强制类型转换运算符；
   8. dynamic_cast：强制类型转换运算符；
   9. reinterpret_cast：强制类型转换运算符；
   10. static_cast：强制类型转换运算符；

7. 下面的运算符只能通过成员函数进行重载:

   1. =：赋值运算符;
   2. ( )：函数调用运算符;
   3. [ ]：下标运算符;
   4. ->：通过指针访问类成员的运算符;

```c++
//mytime.h

#ifndef __MYTIME0_H_
#define __MYTIME0_H_

class Time
{
	private:
		 int hours;
		 int minutes;
	public:
		 Time();
		 Time(int h, int m = 0);
		 void AddMin(int m);
		 void AddHr(int h);
		 void Reset(int h = 0, int m = 0);
		 Time operator+(const Time & t) const;
		 void Show() const;
};

#endif
```

```c++
//mytime0.cpp

#include <iostream>
#include "mytime0.h"

using namespace std;

Time::Time()
{
	hours = minutes = 0;
}

Time::Time(int h, int m)
{
	hours = h;
	minutes = m;
}

void Time::AddMin(int m)
{
	minutes += m;
	hours += minutes / 60;
	minutes %= 60;
}

void Time::AddHr(int h)
{
	hours += h;
}

void Time::Reset(int h, int m)
{
	hours = h;
	minutes = m;
}

Time Time::operator+(const Time & t) const
{
	Time sum;
	sum.minutes = minutes + t.minutes;
	sum.hours = hours + t.hours + sum.minutes / 60;
	sum.minutes %= 60;
	return sum;
}

void Time::Show() const
{
	cout << "hours = " << hours << ", " << " minutes = " << minutes << "." <<endl;
}
```

```c++
//usemytime0.cpp

#include <iostream>
#include "mytime0.h"

using namespace std;

int main()
{
	Time planning;
	Time coding(2, 40);
	Time fixing(5, 55);
	Time total;

	cout << "planning time: " << endl;
	planning.Show();
	
	cout << "coding time: " << endl;
	coding.Show();
	
	cout << "fixing time: " << endl;
	fixing.Show();
 
	total = coding + fixing;
	cout << "coding + fixing: " << endl;
	total.Show();

	Time morefixing(3, 28);
	cout << "morefixing time: " << endl;
	morefixing.Show();

	total = morefixing.operator+(total);
	cout << "morefixing.operator+(total): " << endl;
	total.Show();
	
	return 0;
}

```

### 友元函数

A = 2.75 * B;

从概念上说，2.75 * B应与B *2.75相同，但第一个表达式不对应于成员函数，因为2.75不是Time类型的对象。记住，左侧的操作数应是调 用对象，但2.75不是对象。因此，编译器不能使用成员函数调用来替换该表达式。然而，有一种解决方式——非成员函数（记住，大多数运算符 都可以通过成员或非成员函数来重载）。非成员函数不是由对象调用的，它使用的所有值（包括对象）都是显式参数。这样，编译器能够将表达式与下面的非成员函数调用匹配：

A = opeartor*(2.75， B);

该函数的原型如下：

Time operator*(double m, const Time & t);

但引发了一个新问题：非成员函数不能直接访问类的私有数据，至少常规非成员函数不能访问。然而，有一类特殊的非成员函数可以访问类的私有成员，它们被称为友元函数,另外请记住，只有类声明可以决定哪一个函数是友元，因此类声明仍然控制了哪些函数可以访问私有数据。另外，友元函数不在类作用域内，访问类的静态成员或枚举成员必须加类名和 :: 运算符。总之，类方法和友元只是表达类接口的两种不同机制；

1. 创建友元函数的第一步是将其原型放在类声明中，并在原型声明前 加上关键字friend：

   friend ;

   1. 虽然operator *( )函数是在类声明中声明的，但它不是成员函数，因此不能使用成员运算符 . 来调用；	
   2. 虽然operator *( )函数不是成员函数，但它与成员函数的访问权限相同。

2. 第二步是编写函数定义。因为它不是成员函数，所以不要使用 Time::限定符。另外，不要在定义中使用关键字friend:

   Time operator*(double m, const Time & t){...}

```c++
//mytime0.h

#ifndef __MYTIME0_H_
#define __MYTIME0_H_

class Time
{
	private:
		 int hours;
		 int minutes;
	public:
		 Time();
		 Time(int h, int m = 0);
		 void AddMin(int m);
		 void AddHr(int h);
		 void Reset(int h = 0, int m = 0);
		 Time operator+(const Time & t) const;
		 Time operator*(double n) const;
		 friend Time operator*(double n, const Time & t)
			 {return t * n;}
		 friend std::ostream & operator<<(std::ostream & os, const Time & t);
  	/*
  	将一系列输出存储到文件中很容易，因为：
  	友元函数中的形参ostream，可以用fstream声明的对象。
  	ostream 是 C++ 标准库中输出流类的抽象基类，而 fstream 是 C++ 标准库中文件输		 入输出流类的一个具体实现，继承自 ostream。

		因此，你可以在友元函数中使用 fstream 声明的对象作为 ostream 类型的形参，这是		 完全可以的。因为 fstream 是 ostream 的派生类，所以它继承了 ostream 类的所有		成员和方法，可以用于输出流操作。
  	*/
};

#endif
```

```c++
//mytime0.cpp

#include <iostream>
#include "mytime0.h"

using namespace std;

Time::Time()
{
	hours = minutes = 0;
}

Time::Time(int h, int m)
{
	hours = h;
	minutes = m;
}

void Time::AddMin(int m)
{
	minutes += m;
	hours += minutes / 60;
	minutes %= 60;
}

void Time::AddHr(int h)
{
	hours += h;
}

void Time::Reset(int h, int m)
{
	hours = h;
	minutes = m;
}

Time Time::operator+(const Time & t) const
{
	Time sum;
	sum.minutes = minutes + t.minutes;
	sum.hours = hours + t.hours + sum.minutes / 60;
	sum.minutes %= 60;
	return sum;
}

Time Time::operator*(double n) const
{
	Time result;
	long totalminutes = minutes * n + hours * 60 *n;
	result.hours = totalminutes / 60;
	result.minutes = totalminutes % 60;

	return result;
}

std::ostream & operator<<(std::ostream & os, const Time & t)
{
	os << "hours = " << t.hours << ", minutes = " << t.minutes << ".";
	return os;
}
```

```c++
//usemytime0.cpp

#include <iostream>
#include "mytime0.h"

using namespace std;

int main()
{
	Time aida(3, 35);
	Time tosca(2, 48);
	Time temp;

	cout << "aida and Tosca: \n";
	cout << aida << " " << tosca << endl;
	temp = aida + tosca;
	cout << "aida + tosca: " << temp << endl;
	temp = 1.17 * aida;
	cout << "1.17 * aida: " << temp << endl;

	return 0;
}
```

### 重载运算符：作为成员函数还是非成员函数

这两种格式没有太大的区别。有时，根据类设计，使用非成员函数版本可能更好（尤其是为类定义类型转换时）。

###  再谈重载：一个矢量类

下面代码描述了另一种使用了运算符重载和友元的类设计——一个表示矢量的类。这个类还说明了类设计的其他方面，例如，在同一个对象中包 含两种描述同一样东西的不同方式等。

```c++
//vector.h

#ifndef __VECTOR_H_
#define __VECTOR_H_
#include <iostream>

namespace VECTOR
{
	class Vector
	{
		public:
			enum Mode {RECT, POL};
		private:
			double x;
			double y;
			double mag;
			double ang;
			Mode mode;
			void set_mag();
			void set_ang();
			void set_x();
			void set_y();
		public:
			Vector();
			Vector(double n1, double n2, Mode from = RECT);
			void reset(double n1, double n2, Mode form = RECT);
			~Vector();
			double xval() const {return x;}
			double yval() const {return y;}
			double magval() const {return mag;}
			double angval() const {return ang;}
			void polar_mode();
			void rect_mode();
			Vector operator+(const Vector & b) const;
			Vector operator-(const Vector & b) const;
			Vector operator-() const;
			Vector operator*(double b) const;
			friend Vector operator*(double n, const Vector & a)
				{return a * n;}
			friend std::ostream & operator<<(std::ostream &os, const Vector & v);
	};
}

#endif
```

```c++
//vector.cpp

#include <cmath>
#include "vector.h"

using namespace std;
using std::sqrt;
using std::sin;
using std::cos;
using std::atan;
using std::atan2;
using std::cout;

namespace VECTOR
{
	const double Rad_to_deg = 45.0;

	void Vector::set_mag()
	{
		mag = sqrt(x * x + y * y);
	}
	void Vector::set_ang()
	{
		if(x == 0.0 && y == 0.0)
		{
			ang = 0.0;
		}
		else
			ang = atan2(y, x);
	}
	void Vector::set_x()
	{
		x = mag * cos(ang);
	}
	void Vector::set_y()
	{
		y = mag * sin(ang);
	}

	Vector::Vector()
	{
		x = y = mag = ang = 0.0;
	}
	Vector::Vector(double n1, double n2, Mode from)
	{
		mode = from;
		if(from == RECT)
		{
			x = n1;
			y = n2;
			set_mag();
			set_ang();
		}
		else if(from == POL)
		{
			mag = n1;
			ang = n2 / Rad_to_deg;
			set_x();
			set_y();
		}
		else
		{
			cout << "incorrect 3rd argument to Vector()--";
			cout << "Vector set to:\n";
			x = y = mag = ang = 0.0;
			mode = RECT;
		}
	}
	void Vector::reset(double n1, double n2, Mode from)
	{
		mode = from;
		if(from == RECT)
		{
			x = n1;
			y = n2;
			set_mag();
			set_ang();
		}
		else if(from == POL)
		{
			mag = n1;
			ang = n2 / Rad_to_deg;
			set_x();
			set_y();
		}
		else
		{
			cout << "incorrect 3rd argument to Vector()--";
			cout << "Vector set to:\n";
			x = y = mag = ang = 0.0;
			mode = RECT;
		}
	}
	Vector::~Vector()
	{
	}
	void Vector::polar_mode()
	{
		mode = POL;
	}
	void Vector::rect_mode()
	{
		mode = RECT;
	}
	Vector Vector::operator+(const Vector & b) const
	{
		return Vector(x + b.x, y + b.y);
	}
	Vector Vector::operator-(const Vector & b) const
	{
		return Vector(x - b.x, y - b.y);
	}
	Vector Vector::operator-() const
	{
		return Vector(-x, -y);
	}
	Vector Vector::operator*(double n) const
	{
		return Vector(n * x, n * y);
	}
	std::ostream & operator<<(std::ostream & os, const Vector & v)
	{
		if(v.mode == Vector::RECT)
			os << "(x,y) = (" << v.x << ", " << v.y << ").";
		else if(v.mode == Vector::POL)
			os << "(m,a) = (" << v.mag << ", " << v.ang << ").";
		else 
			os << "vector object mode is invalid";
		return os;
	}
}
```

### 类的自动转换和强制类型转换

1. 在C++中，接受一个 参数的构造函数为将类型与该参数相同的值转换为类提供了蓝图。因 此，下面的构造函数用于将double类型的值转换为Stonewt类型：

   Stonewt(double lbs);

   Stonewt myCat; 

   myCat = 19.6; //隐式转换, 将double类型转换为Stonewt类型

2. explicit，用于关闭这种自动特性，可以这样声明构造函数：

   explicit Stonewt(double lbs);

   这将关闭上述示例中介绍的隐式转换，但仍然允许显式转换，即显式强制类型转换：

   Stonewt myCat; 

   myCat = 19.6; //invaild

   myCat = Stonewt(19.6); //valid

   如果声明中没有使用关键字 explicit ，Stonewt(double)可以用于下面的隐式转换：

   1. 将Stonewt对象初始化为double值时。 
   2. 将double值赋给Stonewt对象时。 
   3. 将double值传递给接受Stonewt参数的函数时。 
   4. 返回值被声明为Stonewt的函数试图返回double值时。 
   5. 在上述任意一种情况下，使用可转换为double类型的内置类型(如int，long等)时。

3. 转换函数：做与上述相反的转换：

   1. operator typeName();

   2. 转换函数必须是类方法； 转换函数不能指定返回类型； 转换函数不能有参数;

   3. 如下述 stone 代码中，自动应用类型转换：

      operator int() const；

      Stonewt poppins(11.1);

      int p_wt = poppins; //隐式转换；

      double p_wt2 = (double ) poppins; //显示转换；

      long gone = poppins; //错误，语句有二义性，int 和 double 都可以被赋值给 long 变量，经验表明，最好不要依赖于这种隐式转换;

   4. 和转换构造函数一样，转换函数也有其优缺点。提供执行自动、隐式转换的函数所存在的问题是：在用户不希望进行转换时，转换函数可能进行转换。在C++98中，关键字explicit不能用于转换函数， 但C++11消除了这种限制。因此，在C++11中，可将转换运算符声明为显式的：

      explicit operator int() const;

      explicit operator double() const;

```c++
//stone.h

#ifndef __STONE_H_
#define __STONE_H_

class Stonewt
{
	private:
		enum {Lbs_per_stn = 14};
		int stone;
		double pds_left;
		double pounds;
	public:
		Stonewt(double lbs);
		Stonewt(int stn, double lbs);
		Stonewt();
		~Stonewt();
		void show_lbs() const;
		void show_stn() const;
		operator int() const;
		operator double() const;
};

#endif
```

```c++
//stone.cpp

#include <iostream>
#include "stone.h"

using std::cout;

Stonewt::Stonewt(double lbs)
{
	stone = int(lbs) / Lbs_per_stn;
	pds_left = int (lbs) % Lbs_per_stn + lbs - int(lbs);
	pounds = lbs;
}

Stonewt::Stonewt(int stn, double lbs)
{
	stone = stn;
	pds_left = lbs;
	pounds = stn * Lbs_per_stn + lbs;
}

Stonewt::Stonewt()
{
	stone = pds_left = pounds = 0;
}

Stonewt::~Stonewt()
{

}

void Stonewt::show_stn() const
{
	cout << stone << " stone, " << pds_left << " pounds.\n";
}

void Stonewt::show_lbs() const
{
	cout << pounds << " pounds.\n";
}

Stonewt::operator int() const
{
	return int (pounds + 0.5);
}

Stonewt::operator double() const
{
	return pounds;
}
```

```c++
//usestone.cpp

#include <iostream>
#include "stone.h"

int main()
{
	using std::cout;
	Stonewt poppins(9, 2.8);
	double p_wt = poppins;
	cout << "convert to double => ";
	cout << "poppons: " << p_wt << " pounds.\n";
	cout << "convert to int => ";
	cout << "poppons: " << int (poppins) << " pounds.\n";
	
	return 0;
}
```

##  类和动态内存分配

###  动态内存和类

1. C++ 使用new和delete运算符来动态控制内存。在这种情况下， 析构函数将是必不可少的，而不再是可有可无的。下述代码中str成员指向警告： new分配的内存。当StringBad对象过期时，str指针也将过期。但str指向 的内存仍被分配，除非使用delete将其释放。删除对象可以释放对象本 身占用的内存，但并不能自动释放属于对象成员的指针指向的内存。因 此，必须使用析构函数。在析构函数中使用delete语句可确保对象过期时，由构造函数使用new分配的内存被释放。

2.  特殊成员函数：

   1. 默认构造函数，如果没有定义； 

   2. 默认析构函数，如果没有定义；

   3. 默认复制构造函数，如果没有定义：

      1. 原型：className::className(const className & )
      2. 每当程序生成了对象副本时，编译器都将使用复制构造函数。具体地说，当函数按值传递对象或函数返回对象时，都将使用复制构造函数。记住，按值传递意味着创建原始变量的一个副本。编译器生成临时对象时，也将使用复制构造函数。例如，将3个Vector对象相加时，编译器可能生成临时的Vector对象来保存 中间结果。何时生成临时对象随编译器而异，但无论是哪种编译器，当 按值传递和返回对象时，都将调用复制构造函数；
      3. 默认的复制构造函数逐个复制非静态成员（成员复制也称为浅复制），复制的是成员的值；如果成员本身就是类对象，则将使用这个类的复制构造函数来复制成员对象。静态成员不受影响，因为它们属于整个类，而不是各个对象；
      4. 必须定义复制构造函数的原因在于，一些类成员是使用new初始化的、指向数据的指针，而不是数据本身，这种情况下，析构函数的调用会释放连续释放来自同一片地址的值，将导致错误；
      5. 解决类设计中这种问题的方法是进行深度复制（deep copy）。

   4. 默认赋值运算符，如果没有定义； 

      1. 原型：className &className::operator=(const className &);

      2. 将已有的对象赋给另一个对象时，将使用重载的赋值运算符；

      3. 初始化对象时，并不一定会使用赋值运算符：

         className value1 = value2；

         这里，value1是一个新创建的对象，被初始化为value2的值，因此使用复制构造函数。然而，正如前面指出的，实现时也可能分两步来处理这条语句：使用复制构造函数创建一个临时对象，然后通过赋值将临时 对象的值复制到新对象中（编译器可能会对这些操作进行优化，减少性能开销）。这就是说，初始化总是会调用复制构造函 数，而使用=运算符时也可能调用赋值运算符；

      4. 复制方式与默认复制构造函数一样（浅复制）；

         1. 解决办法是提供赋 值运算符（进行深度复制）定义。其实现与复制构造函数相似，但也有 一些差别：
      2. 由于目标对象可能引用了以前分配的数据，所以函数应使用delete[ ]来释放这些数据；
         3. 函数应当避免将对象赋给自身；否则，给对象重新赋值前，释放内 存操作可能删除对象的内容；
         4. 函数返回一个指向调用对象的引用；
   
5. 默认地址运算符，如果没有定义。

###  在构造函数中使用new时应注意的事项

1. 如果在构造函数中使用new来初始化指针成员，则应在析构函数中使用delete；
2. new和delete必须相互兼容。new对应于delete，new[ ]对应于delete[ ]；
3. 如果有多个构造函数，则必须以相同的方式使用new，要么都带中括号，要么都不带。因为只有一个析构函数，所有的构造函数都必须与它兼容；
4. 应定义一个复制构造函数，通过深度复制将一个对象初始化为另一个对象；
5. 应当定义一个赋值运算符，通过深度复制将一个对象复制给另一个对象；

包含类成员的类的逐成员复制：

假设类成员的类型为String类或标准string类：

class Magazine

{

​	private:

​		String title;

​		string publisher;

...

}

String和string都使用动态内存分配，这是否意味着需要为Magazine 类编写复制构造函数和赋值运算符？不，至少对这个类本身来说不需要。默认的逐成员复制和赋值行为有一定的智能。如果您将一个 Magazine对象复制或赋值给另一个Magazine对象，逐成员复制将使用成 员类型定义的复制构造函数和赋值运算符。也就是说，复制成员title 时，将使用String的复制构造函数，而将成员title赋给另一个Magazine对 象时，将使用String的赋值运算符，依此类推。然而，如果Magazine类 因其他成员需要定义复制构造函数和赋值运算符，情况将更复杂；在这 种情况下，这些函数必须显式地调用String和string的复制构造函数和赋值运算符。

根据前两节提到的细节编写 String 类，程序如下：

```c++
//string1.h

一种非侵入式应用程序运行状态监测方法、装置及设备#ifndef __STRING1_H_
#define __STRING1_H_
#include <iostream>

class String
{
	private:
		char * str;
		int len;
		static int num_strings;
		static const int CINLIM = 80;
	public:
		String();
		String(const char * s);
		String(const string &);
		~String();
		int length () const{return len;}
		String & operator=(const String &);
		String & operator=(const char *);
		char & operator[](int i);
		const char & operator[](int i) const;
		friend bool operator<(const String &st, const String &st2);
		friend bool operator>(const String &st, const String &st2);	
		friend bool operator=(const String &st, const String &st2);
		friend ostream & operator<<(ostream & os, const String & st);
		friend istream & operator>>(istream & is, String & st);
		static int HowMany(); //静态类成员函数，首先，不能通过对象调用静态成员函数；实际上，静态成员函数甚至不能使用this指针。如果静态成员函数是在公有部分声明的，则可以使用类名和作用域解析运算符来调用它，而不实例化对象（枚举类型和嵌套类也同理，但要访问嵌套类的非静态公共成员，则必须实例化嵌套类）。其次，由于静态成员函数不与特定的对象相关联，因此只能使用静态数据成员。例如，静态方法HowMany()可以访问静态成员num_string，但不能访问str和len(非静态成员)。同样，也可以使用静态成员函数设置类级（classwide）标记，以控制某些类接口的行为。例如，类级标记可以控制显示类内容的方法所使用的格式。
		
}；

#endif
```

```c++
//string1.cpp

#include <ctring>
#include "string1.h"

using std::cin;
using std::cout;

int String::num_strings = 0;

int String::HowMany() //独立实现文件，不能包含关键字static
{
	return num_strings;
}

String::String()
{
	len = 3;
	str = new char[4];
	str = "c++";
	num_strings++;
}

String::String(const char * s)
{
	len = std::strlen(s);
	str = new char[len + 1];
	std::strcpy(str, s);
	num_strings++;
}

String::String(const String & st)
{
	len = st.len;
	str = new char[len + 1];
	std::strcpy[str, st.str];
	num_strings++;
}

String::~String()
{
	delete [] str;
	num_strings--;
}

String::String & operator=(const String & st)
{
	if(st == this)
		return *this;
	delete [] str;
	len = st.len;
	str = new char[len + 1];
	std::strcpy(str, st.str);
	return *this;
}

String & operator=(const char * s)
{
	delete [] str;
	len = std::strlen(s);
	str = new char[len + 1];
	std::strcpy(str, s);
	return *this;	
}

char & String::operator[](int i)
{
	return str[i];
}

const char & String::operator[](int i) const
{
	return str[i];
}

friend bool operator<(const String &st, const String &st2)
{
	return (std::strcmp(st, st2) < 0);
}

friend bool operator>(const String &st, const String &st2)
{
	return !(st < st2);
}

friend bool operator==(const String &st, const String &st2)
{
	return !(std::strcmp(st, st2));
}

friend ostream & operator<<(ostream & os, const String & st)
{
	os << st.str;
	return os;
}
friend istream & operator>>(istream & is, String & st)
{
	char temp[String::CINLIM];
	is.get(temp, String::CINLIM);
	if(is)
		st = temp;
	while(is && is.get() != '\n')
		continue;
	return is;
}
```



### 有关返回对象的说明

如果方法或函数要返回局部对象，则应返回对象，而不是指 向对象的引用。在这种情况下，将使用复制构造函数来生成返回的对 象。如果方法或函数要返回一个没有公有复制构造函数的类（如ostream 类）的对象，它必须返回一个指向这种对象的引用。最后，有些方法和 函数（如重载的赋值运算符）可以返回对象，也可以返回指向对象的引用，在这种情况下，应首选引用，因为其效率更高。



###  使用指向对象的指针

1.  再谈new和delete:

   String * favorite = new String("123");

   这不是为要存储的字符串分配内存，而是为对象分配内存；也就是说，为保存字符串地址的str指针和len成员分配内存（程序并没有给num_string成员分配内存，这是因为num_string成员是静态成员，它独立于对象被保存）。创建对象将调用构造函数，后者分配用于保存字符串 的内存，并将字符串的地址赋给str。然后，当程序不再需要该对象时， 使用delete删除它。对象是单个的，因此，程序使用不带中括号的 delete。与前面介绍的相同，这将只释放用于保存str指针和len成员的空间，并不释放str指向的内存，而该任务将由析构函数来完成；

   1. 如果对象是动态变量，则当执行完定义该对象的程序块时，将调用该对象的析构函数。
   2. 如果对象是静态变量（外部、静态、静态外部或来自名称空间）， 则在程序结束时将调用对象的析构函数。
   3. 如果对象是用new创建的，则仅当您显式使用delete删除对象时，其 析构函数才会被调用。

2.  再谈定位new运算符：

   指针pc1指向的地址与 buffer相同，但buffer是使用new []初始化的，因此必须使用delete []而不 是delete来释放。即使buffer是使用new而不是new []初始化的，delete pc1 也将释放buffer，而不是pc1。这是因为new/delete系统知道已分配的512 字节块buffer，但对定位new运算符对该内存块做了何种处理一无所知。

   delete [] buffer;

   释放使用常规new运算符分配的整个内存块，但它没有为定位new运算符在该内存块中创建的对象调用析构函数，解决方案是显式地为使用定位new运算符创建的对象调用析构函数。正常情况下将自动调用析构函数，这是需要显式调用析 构函数的少数几种情形之一。显式地调用析构函数时，必须指定要销毁的对象。

   需要注意的一点是正确的删除顺序。对于使用定位new运算符创建的对象，应以与创建顺序相反的顺序进行删除。原因在于，晚创建的对象可能依赖于早创建的对象。另外，仅当所有对象都被销毁后，才能释放用 于存储这些对象的缓冲区。

   ```c++
   #include <iostream>
   #include <string>
   #include <new>
   
   using namespace std;
   
   const int BUF = 512;
   
   class JustTesting
   {
   	private:
   		string words;
   		int number;
   	public:
   		JustTesting(const string & s = "Just Testing", int n = 0)
   		{ words = s; number = n; cout << words << " constructed\n"; }
   		~JustTesting() {cout << words << " destroyed\n";}
   		void Show() const { cout << words << ", " << number << endl;}
   };
   
   int main()
   {
   	char * buffer = new char[BUF];
   
   	JustTesting *pc1, *pc2;
   
   	pc1 = new (buffer) JustTesting;
   	pc2 = new JustTesting("Heap1", 20);
   
   	cout << "memory block addresses:\n" << "buffer: " << (void *) buffer << endl;
   	cout << "sizeof(JustTesting): " << sizeof(JustTesting) << endl;
   	cout << "memory contents:\n";
   	cout << pc1 << ": ";
   	pc1->Show();
   	cout << pc2 << ": ";
   	pc2->Show();
   
   	JustTesting *pc3, *pc4;
   	pc3 = new (buffer + sizeof(JustTesting)) JustTesting("Better idea", 6); //如果不加sizeof(JustTesting)，pc3将覆盖之前为pc1申请的空间；
   	pc4 = new JustTesting("Heap2", 10);
   
   	cout << "memory contents:\n";
   	cout << pc3 << ": ";
   	pc3->Show();
   	cout << pc4 << ": ";
   	pc4->Show();
   
   	delete pc2;
   	delete pc4;
   	pc3->~JustTesting();
   	pc1->~JustTesting();
   	delete [] buffer;
   	cout << "Done!\n";
    
     return 0;
   }
   ```

   

### 补充

1. 成员初始化列表的语法：

   从概念上说，调用构造函数时，对象将在括号中的代码执行之前被创建。

   如果Classy是一个类，而mem1、mem2都是这个类的数据成员，则类构造函数可 以使用如下的语法来初始化数据成员：

   Classy::Classy(int n, int m) :mem1(n), mem2(0){...}

   1. 这种格式只能用于构造函数；
   2. 必须用这种格式来初始化非静态const数据成员（至少在C++11之前是这样的），因为在 C++ 的设计中，构造函数体内的代码是对象创建完成后的“处理”部分。此时，所有成员变量（包括 const 数据成员）都应该已经处于一个有效的状态。通过在初始化列表中初始化 const 数据成员，C++ 确保了在进入构造函数体之前，对象的所有部分都已经被初始化，整个对象处于一个有效且一致的状态。
   3. 必须用这种格式来初始化引用数据成员。

   c++11的类内初始化:

   class Classy{int mem1 = 10; const int mem2 = 20;...}；

   这与使用成员初始化列表等价。然而，使用成员初始化列表的构造函数将覆盖相应的类内初始化；

2. 伪私有方法：

   class Queue

   {

   ​	private:

   ​		Queue(const Queue & q) :qsieze(0) {}

   ​		Queue & operator=(const Queue & q){return *this;}

   ...

   }

   这样做有两个作用：第一，它避免了本来将自动生成的默认方法定义。第二，因为这些方法是私有的，所以不能被广泛使用。

   Queue snick(nip); //not allowed

   tuck = nip ; //not allowed

##  类继承

### 派生类

1. class DerivedClass ：public BaseClass{ ... };

2. 特性：

   1. 派生类对象存储了基类的数据成员（派生类继承了基类的实现）；
   2. 派生类对象可以使用基类的方法（派生类继承了基类的接口）；
   3. 派生类不能直接访问基类的私有成员，而必须通过基类方法进行访问；
   4. 创建派生类对象时，程序首先创建基类对象；

3. 构造函数：

   1. 必须给新成员（如果有的话）和继承的成员提供数据；

   2. 派生类构造函数应通过成员初始化列表将基类信息传递给基类构造函数：

   3. 被初始化的顺序为它们被声明的顺序，而不是它们在初始化列表中的顺序;

      DerivedClass::DerivedClass(para1 value1,  para2 value2, 

      ​	para3 value3) : BaseClass(value2, value3) //假设基类的参数是para2， para3

      {

      ​	...；//派生类的成员初始化；

      }

      DerivedClass::DerivedClass(para1 value1,  para2 value2, 

      ​	para3 value3) : BaseClass() //默认的构造函数，此时 :BaseClass() 可省略

      {

      ​	...；//派生类的成员初始化；

      }

      DerivedClass::DerivedClass(para1 value1,  

      ​	const BaseClass & bc) : BaseClass(bc) //将调用基类的复制构造函数

      {

      ​	...；//派生类的成员初始化；

      }

4. 派生类和基类之间的特殊关系:

   1. 基类指针可以在不进行显式类型转换的情况下指向派生类对象；

   2. 基类引用可以在不进行显式类型转换的情况下引用派生类对象；

   3. 基类指针或引用只能用于调用基类方法；

      1. 引用兼容性属性也让您能够将基类对象初始化为派生类对象；

         DerivedClass ob1(value1, value2, value3);

         BaseClass ob2(ob1);

         不存在匹配的构造函数原型：BaseClass(const DerivedClass &); //doesn't exist

         但存在隐式复制构造函数：BaseClass(const BaseClass &);

      2. 将派生对象赋给基类对象；

         DerivedClass ob1(value1, value2, value3);

         BaseClass ob2;

         ob2 = ob1;

         在这种情况下，程序将使用隐式重载赋值运算符：

         BaseClass & operator=(const BaseClass &) const;

### 多态公有继承

虚方法（**virtual** method）:

1. 如果没有使用关键字 virtual，程序将根据引用类型或指针类型选择方法，即编译器对非虚方法使用静态联编；
2. 如果使用了 virtual，程序将根据引用或指针指向的对象的类型来选择方法，即编译器对虚方法使用动态联编；
3. 经常在基类中将派生类会重新定义的方法声明为虚方法。方法在基类中被声明为虚的后，它在派生类中将自动成为虚方法；
4. 虚析构函数:调用派生类的析构函数时，将自动调用基类的虚构函数。因此，使用虚析构函数可以确保正确的析构函数序列被调用，对于下面的程序清单，这种正确的行为并不是很重要，因为析构函数没有执行任何操作。但如果派生类包含一个执行某些操作的析构函数，则基类必须有一个虚析构函数，即使该析构函数不执行任何操作；

使用一个数组来表示多种类型的对象。这就是多态性:

```c++
//brass.h

#ifndef __BRASS_H_
#define __BRASS_H_
#include <string>

class Brass
{
	private:
		std::string fullName;
		long acctNum;
		double balance;
	public:
		Brass(const std::string & s = "Nullbody", long an = -1,
				double bal = 0.0);
		void Deposit(double amt);
		virtual void Withdraw(double amt);
		double Balance() const;
		virtual void ViewAcct() const;
		virtual ~Brass() {}
};

class BrassPlus : public Brass
{
	private:
		double maxloan;
		double rate;
		double owesBank;
	public:
		BrassPlus(const std::string & s = "Nullbody", long an = -1,
				double bal = 0.0, double ml = 500, double r = 0.11125);
		BrassPlus(const Brass & ba, double ml = 500, double r = 0.11125);
		virtual void ViewAcct() const;
		virtual void Withdraw(double amt);
		void ResetMax(double m) { maxloan = m; }
		void ResetRate(double r) { rate = r; }
		void ResetOwes() { owesBank = 0; }
};

#endif
```

```c++
//brass.cpp

#include <iostream>
#include "brass.h"

using std::cout;
using std::endl;
using std::string;

typedef std::ios_base::fmtflags format;
typedef std::streamsize precis;
format setFormat();
void restore(format t, precis p);

Brass::Brass(const string & s, long an, double bal)
{
	fullName = s;
	acctNum = an;
	balance = bal;
}

void Brass::Deposit(double amt)
{
	if(amt < 0)
		cout << "negative deposit not allowedl; " 
			 << "deposit is cancelled.\n";
	else
		balance += amt;
}

void Brass::Withdraw(double amt)
{
	format initialState = setFormat();
	precis prec = cout.precision(2);
	if(amt < 0)
		cout << "negative deposit not allowedl; " 
			 << "withdraw is cancelled.\n";
	else if(amt <= balance)
		balance -= amt;
	else
		cout << "withdraw amount of $" << amt
			 << " exceeds your balance.\n"
			 << "withdrawal canceled.\n";
	restore(initialState, prec);
}

double Brass::Balance() const
{
	return balance;
}

void Brass::ViewAcct() const
{
	format initialState = setFormat();
	precis prec = cout.precision(2);
	cout << "client: " << fullName << endl;
	cout << "account number: " << acctNum << endl;
	cout << "blance: $" << balance << endl;
	restore(initialState, prec);
}

BrassPlus::BrassPlus(const string & s, long an, double bal, 
		double ml, double r) : Brass(s, an, bal)
{
	maxloan = ml;
	owesBank = 0.0;
	rate = r;
}

BrassPlus::BrassPlus(const Brass & ba, double ml ,double r)
	: Brass(ba)
{
	maxloan = ml;
	owesBank = 0.0;
	rate = r;
}

void BrassPlus::ViewAcct() const
{
	format initialState = setFormat();
	precis prec = cout.precision(2);
	Brass::ViewAcct();
	cout << "maximum loan: $" << maxloan << endl;
	cout << "owed to bank: $" << owesBank << endl;
	cout.precision(3);
	cout << "loan rate: " << 100 * rate << "%\n";
	restore(initialState, prec);
}

void BrassPlus::Withdraw(double amt)
{
	format initialState = setFormat();
	precis prec = cout.precision(2);
	double bal = Balance();
	if(amt <= bal)
		Brass::Withdraw(amt);
	else if(amt <= bal + maxloan - owesBank)
	{
		double advance = amt - bal;
		owesBank += advance * (1.0 + rate);
		cout << "bank advance: $" << advance << endl;
		cout << "fiance charge: $" << advance * rate << endl;
		Deposit(advance);
		Brass::Withdraw(amt);
	}
	else
		cout << "credit limt exceeded. transaction cancelled.\n";
	restore(initialState, prec);
}

format setFormat()
{
	return cout.setf(std::ios_base::fixed, std::ios_base::floatfield);
}

void restore(format f, precis p)
{
	cout.setf(f, std::ios_base::floatfield);
	cout.precision(p);
}
```

```c++
//usebrass.cpp

#include <iostream>
#include <string>
#include "brass.h"

const int CLIENTS = 4;

int main()
{
	using std::cin;
	using std::cout;
	using std::endl;

	Brass * p_clients[CLIENTS];
	std::string temp;
	long tempnum;
	double tempbal;
	char kind;

	for(int i = 0; i < CLIENTS; i++)
	{
		cout << "enter client's name: ";
		getline(cin, temp);
		cout << "enter client's account number: ";
		cin >> tempnum;
		cout << "enter opening balance: $";
		cin >> tempbal;
		cout << "enter 1 for brass account or"
			 << "2 for brassplus account: ";
		while(cin >> kind && (kind != '1' && kind != '2'))
			cout << "enter either 1 or 2: ";
		if(kind == '1')
			p_clients[i] = new Brass(temp, tempnum, tempbal);
		else
		{
			double tmax, trate;
			cout << "enter the overdraft limit: $";
			cin >> tmax;
			cout << "enter the interest rate "
				 << "as a decimal fraction: ";
			cin >> trate;
			p_clients[i] = new BrassPlus(temp, tempnum, tempbal, 
					tmax, trate); //向上隐式类型转换
		}
		while(cin.get() != '\n')
			continue;
	}
	cout << endl;
	for(int i = 0; i < CLIENTS; i++)
	{
		p_clients[i]->ViewAcct();
		cout << endl;
	}
	
	for(int i = 0; i < CLIENTS; i++)
	{
		delete p_clients[i];
	}
	cout << "Done!\n";

    return 0;
}
```

###  静态联编和动态联编

将源代码中的函数调用解释为执行特定的函数代码块被称为函数名联编（binding）。在C++中，由于函数重载的缘故，编译器必须查看函数参数以及函数名才能确定使用哪个函数。然而，C/C++编译器可以在编译过程完成这种联编。在编译过程中进行联编被称为静态联编（static binding），又称为早期联编（early binding）。然而，使用了虚函数之后，使用哪一个函数是不能在编译时确定的，因为编译器不知道用户将选择哪种类型的对象。所以，编译器必须生成能够在程序运行时选择正确的虚方法的代码，这被称为动态联编（dynamic binding），又称为晚期联编（late binding）。

1. 两种联编对比：

   1. 效率方面：为使程序能够在运行阶段进行决策，必须采取一些方法来跟踪基类指针或引用指向的对象类型，这增加了额外的处理开销 。例如，如果类不会用作基类，则不需要动态联编。同样，如果派生类（如RatedPlayer）不重新定义基类的任何方法，也不需要使用动态联编。在这些情况下，使用静态联编更合理，效率也更高。由于静态联编的效率更高，因此被设置为C++的默认选择。
   2. 概念模型：在设计类时，可能包含一些不在派生类重新定义的成员函数。不将该函数设置为虚函数，有两方面的好处：首先效率更高；其次，指出不要重新定义该函数。这表明，仅将那些预期将被重新定义的方法声明为虚的。

2. 虚函数的工作原理：

   1. 给每个对象添加一个隐藏成员。隐藏成员中保存了一个指向函数地址数组的指针。这种数组称为虚函数表（virtual function table，vtbl）。虚函数表中存储了为类对象进行声明的虚函数的地址。例如，基类对象包含一个指针，该指针指向基类 中所有虚函数的地址表。派生类对象将包含一个指向独立地址表的指针。如果派生类提供了虚函数的新定义，该虚函数表将保存新函数的地址；如果派生类没有重新定义虚函数，该vtbl将保存函数原始版本的地 址。如果派生类定义了新的虚函数，则该函数的地址也将被添加到vtbl 中；
   2. 调用虚函数时，程序将查看存储在对象中的vtbl地址，然后转向相 应的函数地址表。如果使用类声明中定义的第一个虚函数，则程序将使 用数组中的第一个函数地址，并执行具有该地址的函数。如果使用类声明中的第三个虚函数，程序将使用地址为数组中第三个元素的函数。
   3. 虚函数的使用成本：
      1. 每个对象都将增大，增大量为存储地址的空间；
      2. 对于每个类，编译器都创建一个虚函数地址表（数组）；
      3. 对于每个函数调用，都需要执行一项额外的操作，即到表中查找地址。

3.  有关虚函数注意事项：

   1. 构造函数不能是虚函数；

   2. 析构函数应当是虚函数，除非类不用做基类；

      BaseClass * pr = new DerivedClass; ... delete pr;

      如果使用默认的静态联编，delete语句将调用~BaseClass()析构函数，这将释放由DerivedClass对象中的BaseClass部分指向的内存，但不会释放 新的类成员指向的内存。但如果析构函数是虚的，则上述代码将先调用 ~DerivedClass析构函数释放由DerivedClass组件指向的内存，然后，调用～BaseClass析构函数来释放由BaseClass组件指向的内存。

   3. 友元不能是虚函数；

   4. 如果派生类没有重新定义函数，将使用该函数的基类版本。如果派生类位于派生链中，则将使用最新的虚函数版本；

   5. 重新定义将隐藏方法，重新定义继承的方法并不是重载。如果在派生类中重新定义 函数，将不是使用相同的函数特征标覆盖基类声明，而是隐藏同名的基 类方法，不管参数特征标如何；这引出了两条经验规则：第一，如果重新定义继承的方法，应确保 与原来的原型完全相同，但如果返回类型是基类引用或指针，则可以修改为指向派生类的引用或指针（这种例外是新出现的）。这种特性被称 为返回类型协变（covariance of return type），因为允许返回类型随类类型的变化而变化。第二，如果基类虚函数被重载了，则应在派生类中重新定义所有的基类版本或者是使用using声明引入基类中的该函数，如果只重新定义一个版本而且没有使用using声明，则另外两个版本将被隐藏，派生类对象将无法使用它们。注意，如果不需要修改，则新定义可只调用基类版本。

###  访问控制：protected

派生类的成员可以直接访问基类的保护成员，但不能直接访问基类的私有成员。对于外部世界来说， 保护成员的行为与私有成员相似；但对于派生类来说，保护成员的行为 与公有成员相似。

以1.13.2中的代码为例

class Brass

{

​	protected:

​		doublebalance;

...

}

在这种情况下，BrassPlus类可以直接访问balance，而不需要使用 Brass方法。

继续以BrassPlus为例，如果balance是受保护的，则可以按下面的方式编写代码：

void BrassPlus::Reset(double amt)

{

​	balance = amt;

}

Brass类被设计成只能通过Deposit( )和Withdraw( )才能修改 balance。但对于BrassPlus对象，Reset( )方法将忽略Withdraw( )中的保护 措施，实际上使balance成为公有变量。

### 抽象基类

1. C++通过使用纯虚函数（pure virtual function） 提供未实现的函数。当类声明中包含纯虚函数时，则不能创建该类的对象。这里的理念 是，包含纯虚函数的类只用作基类。要成为真正的ABC，必须至少包含一个纯虚函数。原型中的=0使虚函数成为纯虚函数。但C++甚至允许纯虚函数有定义。总之，ABC描述的是至少使用一个纯虚函数的接口，从ABC派生出的类将根据派生类的具体特征，使用常规虚函数来实现这种接口。

   ```c++
   //acctabc.h
   
   #ifndef __ACCTABC_H_
   #define __ACCTABC_H_
   #include <string>
   
   class AcctABC
   {
   	private:
   		std::string fullName;
   		long acctNum;
   		double balance;
   	protected:
   		struct Formatting
   		{
   			std::ios_base::fmtflags flag;
   			std::streamsize pr;
   		};
   		const std::string & FullName() const { return fullName; }
   		const long AcctNum() const { return acctNum; }
   		Formatting SetFormat() const;
   		void Restore(Formatting & f) const;
   	public:
   		AcctABC(const std::string & s = "Nullbody", long an = -1,
   				double bal = 0.0);
   		void Deposit(double amt);
   		virtual void Withdraw(double amt) = 0;
   		double Balance() const { return balance; }
   		virtual void ViewAcct() const = 0;
   		virtual ~AcctABC() {}
   };
   
   class Brass : public AcctABC
   {
   	public:
   		Brass(const std::string & s = "Nullbody", long an = -1,
   				double bal = 0.0) : AcctABC(s, an, bal) {}
   		virtual void Withdraw(double amt);
   		virtual void ViewAcct() const;
   		virtual ~Brass() {}
   };
   
   class BrassPlus : public AcctABC
   {
   	private:
   		double maxloan;
   		double rate;
   		double owesBank;
   	public:
   		BrassPlus(const std::string & s = "Nullbody", long an = -1,
   				double bal = 0.0, double ml = 500, double r = 0.11125);
   		BrassPlus(const Brass & ba, double ml = 500, double r = 0.11125);
   		virtual void ViewAcct() const;
   		virtual void Withdraw(double amt);
   		void ResetMax(double m) { maxloan = m; }
   		void ResetRate(double r) { rate = r; }
   		void ResetOwes() { owesBank = 0; }
   };
   
   #endif
   ```

   ```c++
   //acctabc.cpp
   
   #include <iostream>
   #include "acctabc.h"
   
   using std::cout;
   using std::ios_base;
   using std::endl;
   using std::string;
   
   AcctABC::AcctABC(const string & s, long an, double bal)
   {
   	fullName = s;
   	acctNum = an;
   	balance = bal;
   }
   
   
   void AcctABC::Deposit(double amt)
   {
   	if(amt < 0)
   		cout << "negative deposit not allowedl; " 
   			 << "deposit is cancelled.\n";
   	else
   		balance += amt;
   }
   
   void AcctABC::Withdraw(double amt)
   {
   	balance -= amt;
   }
   
   AcctABC::Formatting AcctABC::SetFormat() const
   {
   	Formatting f;
   	f.flag = cout.setf(ios_base::fixed, ios_base::floatfield);
   	f.pr = cout.precision(2);
   	return f;
   }
   
   void AcctABC::Restore(Formatting & f) const
   {
   	cout.setf(f.flag, ios_base::floatfield);
   	cout.precision(f.pr);
   }
   
   void Brass::Withdraw(double amt)
   {
   	format initialState = setFormat();
   	precis prec = cout.precision(2);
   	if(amt < 0)
   		cout << "negative deposit not allowedl; " 
   			 << "withdraw is cancelled.\n";
   	else if(amt <= balance)
   		balance -= amt;
   	else
   		cout << "withdraw amount of $" << amt
   			 << " exceeds your balance.\n"
   			 << "withdrawal canceled.\n";
   	restore(initialState, prec);
   }
   
   double Brass::Balance() const
   {
   	return balance;
   }
   
   void Brass::ViewAcct() const
   {
   	format initialState = setFormat();
   	precis prec = cout.precision(2);
   	cout << "client: " << fullName << endl;
   	cout << "account number: " << acctNum << endl;
   	cout << "blance: $" << balance << endl;
   	restore(initialState, prec);
   }
   
   BrassPlus::BrassPlus(const string & s, long an, double bal, 
   		double ml, double r) : Brass(s, an, bal)
   {
   	maxloan = ml;
   	owesBank = 0.0;
   	rate = r;
   }
   
   BrassPlus::BrassPlus(const Brass & ba, double ml ,double r)
   	: Brass(ba)
   {
   	maxloan = ml;
   	owesBank = 0.0;
   	rate = r;
   }
   
   void BrassPlus::ViewAcct() const
   {
   	format initialState = setFormat();
   	precis prec = cout.precision(2);
   	Brass::ViewAcct();
   	cout << "maximum loan: $" << maxloan << endl;
   	cout << "owed to bank: $" << owesBank << endl;
   	cout.precision(3);
   	cout << "loan rate: " << 100 * rate << "%\n";
   	restore(initialState, prec);
   }
   
   void BrassPlus::Withdraw(double amt)
   {
   	format initialState = setFormat();
   	precis prec = cout.precision(2);
   	double bal = Balance();
   	if(amt <= bal)
   		Brass::Withdraw(amt);
   	else if(amt <= bal + maxloan - owesBank)
   	{
   		double advance = amt - bal;
   		owesBank += advance * (1.0 + rate);
   		cout << "bank advance: $" << advance << endl;
   		cout << "fiance charge: $" << advance * rate << endl;
   		Deposit(advance);
   		Brass::Withdraw(amt);
   	}
   	else
   		cout << "credit limt exceeded. transaction cancelled.\n";
   	restore(initialState, prec);
   }
   
   format setFormat()
   {
   	return cout.setf(std::ios_base::fixed, std::ios_base::floatfield);
   }
   
   void restore(format f, precis p)
   {
   	cout.setf(f, std::ios_base::floatfield);
   	cout.precision(p);
   }
   ```

   上述代码中，Brass 和 BrassPluse 对象的基类相同，因此可以用 AcctABC 指针数组同时管理这两种对象(多态性)。通过将1.13.2的use brass.cpp代码中的 Brass 指针数组替换为AcctABC 指针数组， 即可实现相同的结果；

2.  ABC理念：

   可以将ABC看作是一种必须实施的接口。ABC要求具体派生类覆盖其纯虚函数（进一步派生的类不需要）——迫使派生类遵循ABC设置的接口规则。这种模型在基于 组件的编程模式中很常见，在这种情况下，使用ABC使得组件设计人员 能够制定“接口约定”，这样确保了从ABC派生的所有组件都至少支持 ABC指定的功能。



###  继承和动态内存分配

1.  第一种情况：派生类不使用new：

   基类声明中包含了构造函数使用new时需要的特殊方法：析构函数、复制构造函数和重载赋值运算符。因此在派生类中关于基类的一些操作，都是默认调用基类定义的相关函数。

2. 第二种情况：派生类使用new：

   当基类和派生类都采用动态内存分配时，派生类的析构函数、复制构造函数、赋值运算符都必须使用相应的基类方法来处理基类 元素。这种要求是通过三种不同的方式来满足的。对于析构函数，这是 自动完成的；对于构造函数，这是通过在初始化成员列表中调用基类的 复制构造函数来完成的；如果不这样做，将自动调用基类的默认构造函数。对于赋值运算符，派生类的显式赋值运算符必须负责所有继承的基类对象的赋值，这是通过使用作用域解析运算符显式地调用基类 的赋值运算符来完成的。

   derivedClass & derivedClass::operator=(const derivedClass & dc)

   {

   ​	if(this == &dc)

   ​		return *this;

   ​	baseClass::operator=(dc);

   ​	...//为派生类的成员赋值

   }

3. 使用动态内存分配和友元的继承示例：

   作为派生类的友元，该函数能够访问派生类成员。但该函数如不是基类的友元，他使用基类的友元函数来访问派生类继承基类的成员。因为友元不是成员函数，所以不能使用作用域解析运算符来指出要使用哪个友元函数。因此使用强制类型转换，将派生类转换为基类，再使用友元函数，以便匹配原型时能够选择正确的函数。

   基类的友元：

   friend std::ostream & operator<<(std::ostream & os, const baseClass & bs)

   {

   ​	os << ...

   }

   派生类的友元：

   friend std::ostream & operator<<(std::ostream & os, const derivedClass & ds)

   {

   ​	os << (const baseClass &) ds;

   ​	os << ...

   }

##  C++中的代码重用

### valarray类

其构造函数：

1. double gpa[5] = {3.1, 3.5, 3.8, 2.9, 3.3};
2. valarray< double > v1;
3. valarray< int > v2(8);
4. valarray< int > v3(10, 8); //10个元素，每个元素都是8
5. valarray< double > v4(gpa, 4);

```c++
//studentc.h

#ifndef __STUDENTC_H_
#define __STUDENTC_H_
#include <iostream>
#include <string>
#include <valarray>

using std::string;

class Student
{
	private:
		typedef std::valarray<double> ArrayDb;
		string name;
		ArrayDb scores;
		std::ostream & arr_out(std::ostream & os) const;
	public:
		Student() : name("Null stduent"), scores() {}
		explicit Student(const string & s)
			: name(s), scores() {}
		explicit Student(int n) : name("Null sudent"), scores(n) {}
		Student(const string & s, int n)
			: name(s), scores(n) {}
		Student(const string & s, const ArrayDb & a)
			: name(s), scores(a) {}
		Student(const char * str, const double * pd, int n)
			: name(str), scores(pd, n) {}
		~Student() {}
		double Average() const;
		const std::string & Name() const;
		double & operator[](int i);
		double operator[](int i) const;
		friend std::istream & operator>>(std::istream & is, Student & stu);
		friend std::istream & getline(std::istream & is, Student & stu);
		friend std::ostream & operator<<(std::ostream & os, const Student & stu);
};

#endif
```

```c++
//studentc.cpp

#include "studentc.h"

using std::ostream;
using std::endl;
using std::istream;
using std::string;

double Student::Average() const
{
	if(scores.size() > 0)
		return scores.sum() / scores.size();
	else
		return 0;
}

const string & Student::Name() const
{
	return name;
}

double & Student::operator[](int i)
{
	return scores[i];
}

double Student::operator[](int i) const
{
	return scores[i];
}

ostream & Student::arr_out(ostream & os) const
{
	int i;
	int lim = scores.size();
	if(lim > 0)
	{
		for(i = 0; i < lim; i++)
		{
			os << scores[i] << " ";
			if(i % 5 == 4)
				os << endl;
		}
		if(i % 5 != 0)
			os << endl;
	}
	else
		os << " empty array ";
	return os;
}

istream & operator>>(istream & is, Student & stu)
{
	is >> stu.name;
	return is;
}

istream & getline(istream & is, Student & stu)
{
	getline(is, stu.name);
	return is;
}

ostream & operator<<(ostream & os, const Student & stu)
{
	os << "Score for " << stu.name << ":\n";
	stu.arr_out(os);
	return os;
}
```

```c++
//usestudent.cpp

#include <iostream>
#include "studentc.h"

using namespace std;

void set(Student & sa, int n);

const int pupils = 3;
const int quizzes = 5;

int main()
{
	Student ada[pupils] = 
	{Student(quizzes), Student(quizzes), Student(quizzes)};

	int i = 0;
	for(i = 0; i < pupils; i++)
		set(ada[i], quizzes);
	cout << "\nstudent list:\n";
	for(i = 0; i < pupils; i++)
		cout << ada[i].Name() << endl;
	cout << "\nresults:";
	for(i = 0; i < pupils; i++)
	{
		cout << endl << ada[i];
		cout << "average: " << ada[i].Average() << endl;
	}

	cout << "Done!.\n";

    return 0;
}

void set(Student & sa, int n)
{
	cout << "please enter the student's name: ";
	getline(cin, sa);
	cout << "please enter " << n << " quiz scores:\n";
	for(int i = 0; i < n; i++)
		cin >> sa[i];
	while(cin.get() != '\n')
		continue;
}
```



###  私有继承

1. 私有继承：

   class Student : private std::string, private std::valarray< double > {...}; //private关键字可省略，编译器默认是私有继承

   1. 基类的公有方法将成为派生类的私有方法；

   2. 私有继承将对象作为一个未被命名的继承对象添加到类中，包含版本提供了被显式命名的对象成员，而私有继承提供了无名称的子对象成员；

   3. 私有继承提供的特性与包含相同：获得实现，但不获得接口；

   4. 初始化基类组件：

      对于继承类，新版本的构造函数将使用成员初始化列表语法，它使用类名而不是成员名来标识构造函数：

      Student(const char * str, const double * pd, int n)

      ​	: std::string(str), ArrayDb(pd, n) {}

   5. 访问基类的方法：

      使用包含时将使用对象名来调用方法，使用私有继承时，只能在派生类的方法中使用基类的方法，私有继承使得能够使用类名和作用域解析运算符来调用基类的方法：

      double Student::Average() const

      {

      ​	if(ArrayDb::size() > 0)

      ​		return ArrayDb::sum() / ArrayDb::size();

      ​	else

      ​		return 0;

      }

   6. 访问基类对象：

      Student类的包含版本实现了Name( )方法，它返回string对象成员name；但使用私有继承时，该string对象没有名称，由于Student类是从string类派生而来的，Student类的代码使用强制类型转换访问内部的string对象：

      const string & Student::Name() const

      {

      ​	return (const string &) *this;

      }

   7. 访问基类的友元函数:

      在私有继承中，在不进行显式类型转换的情况下，不能将指向派生类的引用或指针赋给基类引用或指针，通过显式地转换为基类来调用正确的函数：

      ostream & operator<<(ostream & os, const Student & stu)

      {

      ​	os << "scores for" << (const String &) stu << ":\n";

      ​	...

      }

      即使这个例子使用的是公有继承，也必须使用显式类型转 换。原因之一是，如果不使用类型转换，代码将与友元函数原型匹 配，从而导致递归调用，另一个原因是，由于这个类使用的是多重继承，编译器将无法确定 应转换成哪个基类

   8. 通过继承得到的将是派生类，因此它 能够访问保护成员；使用组合将这样 的类包含在另一个类中，则后者将不是派生类，而是位于继承层次结构 之外，因此不能访问保护成员。

2.  保护继承：

   class Student : protected std::string, protected std::valarray< double > {...};

   1. 使用保护继承时，基类的公有成员和保护成员都将成为派生类的保 护成员。和私有继承一样，基类的接口在派生类中也是可用的，但 在继承层次结构之外是不可用的。使用私有继承时，第三 代类将不能使用基类的接口，这是因为基类的公有方法在派生类中将变 成私有方法；使用保护继承时，基类的公有方法在第二代中将变成受保 护的，因此第三代派生类可以使用它们；

3. 使用using重新定义访问权限：

   使用保护派生或私有派生时，基类的公有成员将成为保护成员或私 有成员。假设要让基类的方法在派生类外面可用，方法之一是定义一个使用该基类方法的派生类方法，如1.e。

   另一种方法是，using声明（只适用于继承，而不适用于包含）来指出派生类可以使用特定的基类成员，即使采用的是私有派生：

   class Student : private std::string, private std::valarray< double >

   {

   ​	...

   ​	public:

   ​		using std::valarray< double >::min; //using声明只使用成员名—没有圆括号、函数特征标和返回类型

   ​		using std::valarray< double >::operator[]; //两个版本（const和非const）都可用

   };

###  多重继承

1. 有多少base：

   假设从一个基类base，公有派生出两个类derived1、derived2，再从这两个类公有派生出一个类derived，通常可以将这个类对象的地址赋给基类指针，但现在将出现二义性，因为有derived1、derived2都继承了一个base组件，因此derived包含两个base组件，即derived类对象继承了两个base对象，所以应使用类型转换来指定对象：

   derived ed;

   base * p = &ed; //二义性

   base * p1 = (derived1 *) &ed; 

   base * p2 = (derived2 *) &ed;

   1. 虚基类:

      虚基类使得从多个类（它们的基类相同）派生出的对象只继承一个基类对象。

   2. 新的构造函数规则:

      derived(const base & bs, int p = 0, int v = derived2::other)

      ​	: derived1(bs, p), derived2(bs, v) {} 

      存在的问题是，自动传递信息时，将通过2条不同的途径（ derived1 和 derived2 ）将 bs 传递给 base 对象。为避免这种冲突，C++在基类是虚的时候，禁止信息通过中间类自动传递给基类。因此，上述构造函数将初始化成员p和v，但bs参数中的信息将不会传递给子对象 derived1和derived2。然而，编译器必须在构造派生对象之前构造基类对象组件；在上述情况下，编译器将使用base的默认构造函数。如果不希望默认构造函数来构造虚基类对象，则需要显式地调用所需的基类构造函数：

      derived(const base & bs, int p = 0, int v = derived2::other)

      ​	: base(bs), derived1(bs, p), derived2(bs, v) {} //对于非虚基类非法

2. 哪个方法：

   对于单继承，如果没有重新定义某个方法，则将使用最近祖先中的定义。而在多重继承中，如果每个直接祖先都有一个相同函数，如show()函数，这使得上述调用是二义性的，即不知道该调用哪个祖先的函数。

   使用模块化方式：

   void base::Data() const {...}

   void derived1::Data() const {...}

   void derived2::Data() const {...}

   void derived::Data() const

   {

   ​	derived1::data();

   ​	derived2::data();

   }

   void derived::Show() const

   {

   ​	cout << ...;

   ​	base::data();

   ​	data();

   }

   如果Data( )方法是保护的，则只能在继承层次结构中的类中使用它，在其他地方则不能使用，对象仍可使用Show( )方法。而Data( )方法只在类内 部可用，作为协助公有接口的辅助方法。然而，使Data( )方法成为私有 的将阻止derived1和derived2中的代码使用base::Data( )，这正是保护访问类的用武之地。

3. 混合使用虚基类和非虚基类:

   当类通过多条虚途径和非虚途径继承某个特定的基类时，该类将包含一个表示所有的虚途径的基类子对象和分别表示各条非虚途径 的多个基类子对象;

4. 虚基类和支配:

   使用非虚基类时，如果类从不同的类那里继承了两个或更多的同名成员（数据或 方法），则使用该成员名时，如果没有用类名进行限定，将导致二义性。但如果使用的是虚基类，如果某个名称优先于（dominates）其他所有名称，则使用它时， 即便不使用限定符，也不会导致二义性。

   class B

   {

   ​	public:

   ​		short q();

   };

   class C : virtual public B

   {

   ​	public:

   ​		long q();

   ​		int omg();

   };

   class D : public C {...};

   class E : virtual public B

   {

   ​	private:

   ​		int omg();

   };

   class F : public D, public E {...};

   类C中的q( )定义优先于类B中的q( )定义，因为类C是从类B派生而 来的。因此，F中的方法可以使用q( )来表示C::q( )。另一方面，任何一 个omg( )定义都不优先于其他omg( )定义，因为C和E都不是对方的基 类。所以，在F中使用非限定的omg( )将导致二义性。虚二义性规则与访问规则无关，也就是说，即使E::omg( )是私有 的，不能在F类中直接访问，但使用omg( )仍将导致二义性。同样，即 使C::q( )是私有的，它也将优先于D::q( )。在这种情况下，可以在类F中 调用B::q( )，但如果不限定q( )，则将意味着要调用不可访问的C::q( )。

###  类模板



1. 定义类模板：

   1. 类模板：

      template < class Type > 

      class className{ ... };

   2. 成员函数模板：

      1. 内联定义：

         void/classname func(Type item / Type & item / const Type & item ... ) { ... }

      2. 模板原型：

         void/classname func(Type item / Type & item / const Type & item ... )；

      3. 类模板外定义：

         template < class Type >

         void/classname< Type > className< Type >::func(Type item ... ) { ... };

         若原型将成员函数的返回类型声明为类，而实际的模板函数定义将类型定义为classname< Type >。前者是后者的缩写，但只能在类中使用。即可以在模板声明或模板函数定义内使用classname，但在类外部定义成员函数，即指定返回类型或使用作用域解析运算符时，必须使用完整的classname< Type >。[注入类名 - cppreference.com](https://zh.cppreference.com/w/cpp/language/injected-class-name)

      4. 所有模板信息放在一个头文件中，并在要使用这些模板的文件中包含该头文件。

   ```c++
   //stacktp.h
   
   #ifndef __STACKTP_H_
   #define __STACKTP_H_
   
   template <class Type>
   class Stack
   {
   	private:
   		enum {MAX = 10};
   		Type items[MAX];
   		int top;
   	public:
   		Stack();
   		~Stack();
   		bool isempty();
   		bool isfull();
   		bool push(const Type & item);
   		bool pop(Type & item);
   };
   
   template <class Type>
   Stack<Type>::Stack()
   {
   	top = 0;
   }
   
   template <class Type>
   Stack<Type>::~Stack()
   {
   
   }
   
   template <class Type>
   bool Stack<Type>::isempty()
   {
   	return top == 0;
   }
   
   template <class Type>
   bool Stack<Type>::isfull()
   {
   	return top == MAX;
   }
   
   template <class Type>
   bool Stack<Type>::push(const Type & item)
   {
   	if(top == MAX)
   		return false;
   	items[top++] = item;
   	return true;
   }
   
   template <class Type>
   bool Stack<Type>::pop(Type & item)
   {
   	if(top == 0)
   		return false;
   	item = items[--top];
   	return true;
   }
   
   #endif
   ```

2. 模板类实例化：

   className< Type > class1; //类型参数Type为内置类型或类对象；

   className< Type > class2; //若Type与上次实例化的Type相同，则不再生成相同的类声明；

   ```c++
   //usestacktp.cpp
   
   #include<iostream>
   #include<cctype>
   #include "stacktp.h"
   
   using namespace std;
   
   int main()
   {
   	char ch;
   	string po;
   	Stack<string> st;
   
   	cout << "Please enter A to push to stack,\n"
   		 << "P to pop from stack,\n"
   		 << "Q to quit.\n";
   
   	while (cin >> ch && toupper(ch) != 'Q')
   	{
   		while(cin.get() != '\n')
   			continue;
   		switch(ch)
   		{
   			case 'A':
   			case 'a': 
   					cout << "Enter a number you want to push to stack:\n";
   					cin >> po;
   					if(st.isfull())
   						cout << "Stack already full" << endl;
   					else
   						st.push(po);
   					break;
   			case 'P':
   			case 'p':
   					if(st.isempty())
   						cout << "Stack is empty" << endl;
   					else
   					{
   						st.pop(po);
   						cout << "PO #" << po << " is poped" << endl;
   					}
   					break;
   			default:
   					break;
   		}
   
   		cout << "Please enter A to push to stack,\n"
   			<< "P to pop from stack, \n"
   			<< "Q to quit.\n";
   	}
   
   	return 0;
   }
   ```

3. 深入探讨模板类：

   1. 不正确地使用指针栈（对上面的程序进行修改）：

      模板实例化： 

      Stack < char * > st;

      1. string po; 替换为 char * po; 

         这旨在用char指针而不是string对象来接收键盘输入。这种方法很快就失败了，因为仅仅创建指针，没有创建用于保存输入字符串的空间;

      2. string po; 替换为 char po[40];

         这为输入的字符串分配了空间。另外，po的类型为char*，因此可以被放在栈中。但数组完全与pop()方法的假设相冲突：首先，引用变量item必须引用某种类型的左值，而不是数组名。其次，代码假设可以给item赋值，即使item能够引用数组，也不能为数组名赋值。因此这种方法失败了;

      3. string po; 替换为 char * po = new char[40];

         这为输入的字符串分配了空间。另外，po是变量，因此与pop()的代码兼容。然而，这里将会遇到最基本的问题：只有一个pop变量，该变量总是指向相同的内存单元。确实，在每当读取新字符串时，内存的内容都将发生改变，但每次执行压入操作时，加入到栈中的的地址都相同。因此，对栈执行弹出操作时，得到的地址总是相同的，它总是指向读入的最后一个字符串。具体地说，栈并没有保存每一个新字符串;

   2. 正确使用指针栈:

      使用指针栈的方法之一是，让调用程序提供一个指针数组，其中每个指针都指向不同的字符串。把这些指针放在栈中是有意义的，因为每个指针都将指向不同的字符串。注意，创建不同指针是调用程序的职责，而不是栈的职责。栈的任务是管理指针，而不是创建指针。

4. 数组模板示例和非类型参数:

   template < class T, int n> class ArrayTP { ... }；

   指定特殊的类型而不是用作泛型名，称为非类型（non-type）或表达式（expression）参数。表达式参数有一些限制。表达式参数可以是整型、枚举、引用或指针。另外，模板代码不能修改参数的值，也不能使用参数的地址。所 以，在ArrayTP模板中不能使用诸如n++和&n等表达式。另外，实例化模板时，用作表达式参数的值必须是常量表达式。

   Stack中使用的构造函数方法相比：

   1. 优点：

      构造函数方法使用的是通过new和delete管理的堆内存，而表达式参数方法使用的是为自动变量维护的内存栈。这样，执行速度将更快，尤其是在使用了很多小型数组时。

   2. 缺点：

      每种数组大小都将生成自己的模板，下面的声明将生成两个独立的类声明：。

      ArrayTP<double, 12> s1;

      ArrayTP<double, 13> s2; 

      但下面的声明只生成一个类声明，并将数组大小信息传递给类的构造函数：

      Stack< int > s1(12);

      Stack< int > s2(13);

   3. 构造函数方法更通用，这是因为数组大小是作为类成员（而不是硬编码）存储在定义中的。这样可以将一种尺寸的数组赋给另一种尺寸的数组，也可以创建允许数组大小可变的类。

5. 模板多功能性：

   template < typename T >

   class Array

   {

   ​	private:

   ​		T entry;

   }

   1. 用作基类:

      template < typename Type >

      class GrowArray : public Array< Type > { ... };

   2. 用作组件类，例如使用数组模板实现栈模板：

      template < Typename Tp>

      class Stack

      {

      ​	Array< Tp > ar;

      }

   3. 用作其他模板的类型参数，例如数组模板来构造数组——数组元素是基于栈模板的栈：

      Array < Stack< int> > asi; //C++98要求使用至少一个空白字符将两个>符号分开，以免与运算符>>混淆。C++11不要求这样做

6. 模板多功能性

   1. 递归使用模板

      ArrayTp< ArrayTp< int, 5>, 10 > twodee; //等价于int twodee[10] [5];

   2. 使用多个类型参数

      模板可以包含多个类型参数。例如，假设希望类可以保存两种值， 则可以创建并使用Pair模板来保存两个不同的值（标准模板库提供了类 似的模板，名为pair）

   3. 默认类型模板参数

      template < class T1, class T2 = int > class Topo { ... };

      Topo< double, double > m1; //T1 is double, T2 is double

      Topo< double > m2; //T1 is double, T2 is int

      虽然可以为类模板类型参数提供默认值，但不能为函数模板参数提 供默认值。然而，可以为非类型参数提供默认值，这对于类模板和函数 模板都是适用的。

7. 模板的具体化

   1. 隐式实例化

      声明一个或多个对象，指出所需的类型，而编译器使用通用模板提供的处方生成具体的类定义;

   2. 显式实例化

      template class className< type1, type2, ...>;

      在这种情况下，虽然没有创建或提及类对象，编译器也将生成类声明（包括方法定义）。和隐式实例化一样，也将根据通用模板来生成具体化。

   3. 显式具体化

      template <> class className< specialized-type-name > { ... };

   4. 部分具体化

      即部分限制模板的通用性

      template < class T1, class T2 > class className { ... };

      template < class T1 > class className<T1, int> { ... };

      template <> class className<int, int> { ... };

      如果有多个模板可供选择，编译器将使用具体化程度最高的模板。

      可以通过为指针提供特殊版本来部分具体化现有的模板(函数模板不可以)。

      template < class T > class className { ... };

      template <> class className<T*> { ... };

      如果提供的类型不是指针，则编译器将使用通用版本；如果提供的 是指针，则编译器将使用指针具体化版本。

8. 成员模板

   模板可用作结构、类或模板类的成员;

   ```c++
   #include <iostream>
   
   using std::cout;
   using std::endl;
   
   template <typename T >
   class beta
   {
   	private:
   		template <typename V>
   		class hold
   		{
   			private:
   				V val;
   			public:
   				hold(V v = 0) : val(v) {}
   				void show() const { cout << val << endl; }
   				V value() const { return val; }
   		};
   		hold<T> q;
   		hold<int> n;
   	public:
   		beta(T t, int i) : q(t), n(i) {}
   		template<typename U>
   		U blab(U u, T t) { return (n.value() + q.value()) * u / t; }
   		void show() const { q.show(); n.show();}
   };
   
   int main()
   {
   	beta<double> guy(3.5, 3);
   	cout << "T was set to double.\n";
   	guy.show();
   	cout << "V was set to T, which is double, then V was set to int.\n";
   	cout << guy.blab(10, 2.3) << endl;
   	cout << "U was set to int.\n";
   	cout << guy.blab(10.0, 2.3) << endl;
   	cout << "U was set to double.\n";
   	cout << "Done\n";
   
   	return 0;
   }
   ```

   如果所用的编译器接受类外面的定义，则在beta模板之外定义模板类和模板方法的代码框架如下：

   template < typename T >

   ​	template < typename V>

   ​	class beta< T >::hold

   ​	{ ... };

   template < typename T >

   ​	template < typename U>

   ​	class beta< T >::blab(U u, T t)

   ​	{ ... };

9. 将模板用作参数

   模板可以包含类型参数（如typename T）和非类型参数（如int n）。模板还可以包含本身就是模板的参数，这种参数是模板新增的特性，用于实现STL。

   template < template < typename T > class Thing >

   class Crab { ... }; //template < typename T > class是类型，Thing是参数

   ```c++
   //tempparm.cpp
   
   #include <iostream>
   #include "stacktp.h"
   
   template <template <typename T> class Thing>
   class Crab
   {
   	private:
   		Thing<int> s1;
   		Thing<double> s2;
   	public:
   		Crab() {};
   		bool push(int a, double x) { return s1.push(a) && s2.push(x); }
   		bool pop(int & a, double & x) { return s1.pop(a) && s2.pop(x); }
   };
   
   int main()
   {
   	using std::cout;
   	using std::cin;
   	using std::endl;
   	Crab<Stack> nebula;
   	int ni;
   	double nb;
   	cout << "enter int double pairs, such as 4 3.5 (0 0 to end):\n";
   	while(cin >> ni >> nb && ni > 0 && nb > 0)
   	{
   		if(!nebula.push(ni, nb))
   			break;
   	}
   
   	while(nebula.pop(ni, nb))
   		cout << ni << ", " << nb << endl;
   	cout << "bye\n";
   
   	return 0;
   }
   ```

10. 模板类和友元

    1. 模板类的非模板友元函数

       有些编译器将对使用非模板友元发出警告（vscode c++17）：warning: friend declaration 'void reports(HasFriend<T>&)' declares a non-template function [-Wnon-template-friend]

       template < class T >

       class HasFriend

       {

       ​	friend void report(HasFriend &); //注入类名，可以省略< T >

       ​	friend void report(HasFriend< T > &);

       ​	friend void counts(); //所有HasFriend类的友元

       };

       如果想要不同的模板参数实例化表现出相同的report（即所有的实例共享一个reports），在类内声明时直接定义函数体，此时不能给某个具体类型提供额外的具体化版本，那将会发生二义性；

       friend void reports(*HasFriend* &){

       ​     cout << "HasFriend<>: " << hf.item << endl;

       }

       如果想要不同的模板参数实例化表现出不同的report，例如，带HasFriend< int >参数的report( )将成为HasFriend类的友元，同样带HasFriend< double >参数的report( )将是report( )的一个 重载版本——它也是Hasfriend类的友元，这意味着必须为要使用的友元定义显式具体化：

       void report(HasFriend< int> &) { ... };

       void report(HasFriend< double> &) { ... };

       ```c++
       #include <iostream>
       
       using std::cout;
       using std::endl;
       
       template <typename T>
       class HasFriend
       {
       	private:
       		T item;
       		static int ct;
       	public:
       		HasFriend(const T & i) : item(i) { ct++; }
       		~HasFriend() { ct--; }
       		friend void counts();
       		friend void reports(HasFriend<T> &);
       };
       
       template <typename T>
       int HasFriend<T>::ct = 0;
       
       void counts()
       {
       	cout << "int count: " << HasFriend<int>::ct << ";";
       	cout << "double count: " << HasFriend<double>::ct << ";";
       }
       
       void reports(HasFriend<int> & hf)
       {
       	cout << "HasFriend<int>: " << hf.item << endl;
       }
       
       void reports(HasFriend<double> & hf)
       {
       	cout << "HasFriend<double>: " << hf.item << endl;
       }
       
       int main()
       {
       	cout << "no objects declared: ";
       	counts();
       	HasFriend<int> hfi1(10);
       	cout << "After hfil declared: ";
       	counts();
       	HasFriend<int> hfi2(20);
       	cout << "after hfi2 declared: ";
       	counts();
       	HasFriend<double> hfdb(10.5);
       	cout << "after hfdb declared: ";
       	counts();
       	reports(hfi1);
       	reports(hfi2);
       	reports(hfdb);
       
       	return 0;
       }
       ```

    2. 模板类的约束模板友元函数

       template < class T > void counts();

       template < class T > void report(T &);

       template < class T >

       class HasFriend

       {

       ​	friend void counts< T >();  //counts( )函数没有参数，因此必须使用模板参数语法（< T >）来指明其具体化 

       ​	friend void report<  >(HasFriend< T > &); //<>可以为空，因为可以从函数参数推断出模板类型参数

       };

       ```c++
       #include <iostream>
       
       using std::cout;
       using std::endl;
       
       template <typename T> void counts();
       template <typename T> void reports(T &);
       
       template <typename T>
       class HasFriend
       {
       	private:
       		T item;
       		static int ct;
       	public:
       		HasFriend(const T & i) : item(i) { ct++; }
       		~HasFriend() { ct--; }
       		friend void counts<T>();
       		friend void reports<>(HasFriend<T> &);
       };
       
       template <typename T>
       int HasFriend<T>::ct = 0;
       
       template <typename T>
       void counts()
       {
       	cout << "template count: " << HasFriend<T>::ct << ";";
       	cout << "template size: " << sizeof(HasFriend<T>) << endl;
       }
       
       template <typename T>
       void reports(T & hf)
       {
       	cout << hf.item << endl;
       }
       
       int main()
       {
       	cout << "no objects declared: ";
       	HasFriend<int> hfi1(10);
       	cout << "After hfil declared: ";
       	counts<int>();
       	HasFriend<int> hfi2(20);
       	cout << "after hfi2 declared: ";
       	counts<int>();
       	HasFriend<double> hfdb(10.5);
       	cout << "after hfdb declared: ";
       	counts<double>();
       	reports(hfi1);
       	reports(hfi2);
       	reports(hfdb);
       
       	return 0;
       }
       ```

       请注意，前一个程序清单包含1个count( )函数， 它是所有HasFriend类的友元；而此程序清单包含两个count( )函数， 它们分别是某个被实例化的类类型的友元。因为count( )函数调用没有可 被编译器用来推断出所需具体化的函数参数，所以这些调用使用 count和coount( )指明具体化。但对于report( )调用，编译器 可以从参数类型推断出要使用的具体化

    3. 模板类的非约束模板友元函数

       template < typename T >

       class ManyFriend

       {

       ​	template < typename C, typename D> friend void show(C &, D &);//所有ManyFriend具体化的友元函数

       };

       ```c++
       #include <iostream>
       
       using std::cout;
       using std::endl;
       
       template <typename T>
       class HasFriend
       {
       	private:
       		T item;
       	public:
       		HasFriend(const T & i) : item(i) {}
       		template <typename C, typename D> friend void show(C &, D &);
       };
       
       template <typename C, typename D> void show(C & c, D & d)
       {
       	cout << c.item << ", " << d.item << endl;
       }
       
       int main()
       {
       	HasFriend<int> hfi1(10);
       	HasFriend<int> hfi2(20);
       	HasFriend<double> hfdb(10.5);
       	show(hfi1, hfi2);
       	show(hfi2, hfdb);
       
       	return 0;
       }
       ```

11. 模板别名

    1. typedef为模板具体化指定别名：

       typedef std::array<double, 12> arrd;

       typedef std::array<int, 12> arri;

       arrd gallons;

       arri days;

    2. 使用模板提供一系列别名:

       template< typename T >

       ​	using arrtype = std::array<T, 12>; //arrtype< T >表示类型std::array< T, 12 >

       arrtype< double > gallons;

       arrtype< int > days;

12. 注入类名：

    注入类名是在类的作用域内该类自身的无限定的名字。

    类模板中，注入类名能用作指代当前模板的模板名，或指代当前实例化的类名。

    [注入类名 - cppreference.com](https://zh.cppreference.com/w/cpp/language/injected-class-name)

##  友元、异常和其他

###  友元

1. 友元类

   类友元是一种自然用语，用于表示一些关系。如果不使用某些形式的友元关系，则必须将Tv类的私有部分 设置为公有的。或者创建一个笨拙的、大型类来包含电视机和遥控器，这种解决方法无法反应这样的事实，即同一个遥控器可用于多台电视机

   ```c++
   //tv.h
   
   #ifndef __TV_H_
   #define __TV_H_
   
   class Tv
   {
   	private:
   		int state;
   		int volume;
   		int maxchannel;
   		int channel;
   		int mode;
   		int input;
   	public:
   		friend class Remote;
   		enum {Off, On};
   		enum {MinVal, MaxVal = 20};
   		enum {Antenna, Cable};
   		enum {TV, DVD};
   
   		Tv(int s = Off, int mc = 125) : state(s), volume(5), 
   			maxchannel(mc), channel(2), mode(Cable), input(TV) {}
   		void onoff() { state = (state = On) ? Off : On; }
   		bool ison() const { return state == On; }
   		bool volup();
   		bool voldown();
   		void chanup();
   		void chandown();
   		void set_mode() { mode = (mode == Antenna) ? Cable : Antenna; }
   		void set_input() { input = (input == TV) ? DVD : TV; }
   		void settings() const;
   };
   
   class Remote
   {
   	private:
   		int mode;
   	public:
   		Remote(int m = Tv::TV) : mode(m) {}		
   		void onoff(Tv & t) { t.onoff(); }
   		bool volup(Tv & t) { return t.volup(); }
   		bool voldown(Tv & t) { return t.voldown(); }
   		void chanup(Tv & t) { t.chanup(); }
   		void chandown(Tv & t) { t.chandown(); }
   		void set_mode(Tv & t) { t.set_mode(); }
   		void set_chan(Tv & t, int c) { t.channel = c; }
   		void set_input(Tv & t) { t.set_input(); }
   };
   
   #endif
   ```

   ```c++
   //tv.cpp
   
   #include <iostream>
   #include "tv.h"
   
   bool Tv::volup()
   {
   	if(volume < MaxVal)
   	{
   		volume++;
   		return true;
   	}
   	else
   		return false;
   }
   
   bool Tv::voldown()
   {
   	if(volume > MinVal)
   	{
   		volume--;
   		return true;
   	}
   	else
   		return false;
   }
   
   void Tv::chanup()
   {
   	if(channel < maxchannel)
   		channel++;
   	else
   		channel = 1;
   }
   
   void Tv::chandown()
   {
   	if(channel > 1)
   		channel--;
   	else
   		channel = maxchannel;
   }
   
   void Tv::settings() const
   {
   	using std::cout;
   	using std::endl;
   	cout << "TV is " << (state == Off ? "Off" : "On") << endl;
   	if(state == On)
   	{
   		cout << "volume setting = " << volume << endl;
   		cout << "channel setting = " << channel << endl;
   		cout << "Mode = "
   			<< (mode == Antenna ? "antenna" : "cable") << endl;
   		cout << "Input = " << (input == TV ? "TV" : "DVD") << endl;
   	}
   }
   ```

2. 友元成员函数

   上述代码大多数Remote方法都是用Tv类的公有接口实现的。这意味着这些方法不是真正需要作为友元。事实上，唯 一直接访问Tv成员的Remote方法是Remote::set_chan( )，因此它是唯一 需要作为友元的方法。确实可以选择仅让特定的类成员成为另一个类的 友元，而不必让整个类成为友元，但这样做稍微有点麻烦，必须小心排 列各种声明和定义的顺序。

   在Tv类声明中将其声明为友元：

   class Tv

   {

   ​	friend void Remote::set_chan(TV & t, int c);

   ​	...

   };

   要使编译器能够处理这条语句，它必须知道Remote的定义。 否则，它无法知道Remote是一个类，而set_chan是这个类的方法。这意 味着应将Remote的定义放到Tv的定义前面。Remote的方法提到了Tv对 象，而这意味着Tv定义应当位于Remote定义之前。避开这种循环依赖 的方法是，使用前向声明（forward declaration）。为此，需要在Remote 定义的前面插入下面的语句：

   class Tv; //forward declaration

   排列次序应如下:

   class Tv;

   class Remote { ... };

   class Tv { ... };

   但是Remote声明包含了内联代码，例如：

   void onoff(Tv & t) { t.onoff(); }

   由于这将调用Tv的一个方法，所以编译器此时必须已经看到了Tv 类的声明，这样才能知道Tv有哪些方法，但正如看到的，该声明位于 Remote声明的后面。这种问题的解决方法是，使Remote声明中只包含 方法声明，并将实际的定义放在Tv类之后。这样，排列顺序将如下：

   class Tv;

   class Remote { ... }; //只包含方法声明

   class Tv { ... };

   //方法定义，在方法定义中使用 inline关键字，仍然可以使其成为内联方法

3. 其他友元关系

   一些Remote方法能够 像前面那样影响Tv对象，而一些Tv方法也能影响Remote对象。这可以 通过让类彼此成为对方的友元来实现，即除了Remote是Tv的友元外， Tv还是Remote的友元。需要记住的一点是，对于使用Remote对象的Tv 方法，其原型可在Remote类声明之前声明，但必须在Remote类声明之 后定义，以便编译器有足够的信息来编译该方法。

   class Tv

   {

   ​	public:

   ​		friend class remote;

   ​		void buzz(Remote & r);

   ​		...

   };

   class Remote

   {

   ​	public:

   ​		friend classTv;

   ​		void Bool volup(Tv & t) { t.volup(); }

   ​		...

   };

   inline void Tv::buzz(Remote & r) { ... }

4. 共同的友元：

   函数需要访问两个类的私有数据。 从逻辑上看，这样的函数应是每个类的成员函数，但这是不可能的。它可以是一个类的成员，同时是另一个类的友元，但有时将函数作为两个 类的友元更合理。例如，假定有一个Probe类和一个Analyzer类，前者表示某种可编程的测量设备，后者表示某种可编程的分析设备。这两个类都有内部时钟，且希望它们能够同步，则应该包含下述代码行：

   class Analyzer;

   class Probe

   {

   ​	friend void sync(Analyzer & a, const Probe & p);

   ​	friend void sync(Probe & p, const Analyzer & a);

   ​	...

   };

   class Analyzer

   {

   ​	friend void sync(Analyzer & a, const Probe & p);

   ​	friend void sync(Probe & p, const Analyzer & a);

   ​	...

   };

   inline friend void sync(Analyzer & a, const Probe & p) { ... }

   inline friend void sync(Probe & p, const Analyzer & a) { ... }



###  嵌套类

1. 嵌套类和访问权限
   1. 将类声明放在另一个类中。在另一个类中声明的类 被称为嵌套类（nested class），它通过提供新的类型类作用域来避免名称混乱。包含类的成员函数可以创建和使用被嵌套类的对象。对类进行嵌套与包含并不同。包含意味着将类对象作为另一个类的成员，而对类进行嵌套不创建类成员，而是定义了一种类型，该类型仅 在包含嵌套类声明的类中有效。 对类进行嵌套通常是为了帮助实现另一个类，并避免名称冲突。
   2. 如果嵌套类是在另一个类的保护部分声明的，对于外部世界则是不可见的，但派生类将知道嵌套类，并可以直接创建这种类型的对象。
   3. 如果嵌套类是在另一个类的私有部分声明的，对于外部世界和派生类都是不可见的。
   4. 如果嵌套类是在另一个类的公有部分声明的，则允许后者、后者的派生类以及外部世界使用它，因为它是公有的。然而，由于嵌套类的作用域为包含它的类，因此在外部世界使用它时，必须使用类限定符。
2.  模板也可以包含嵌套类

### 异常

1. 调用abort( )

   Abort( )函数的原型位于头文件cstdlib（或 stdlib.h）中，其典型实现是向标准错误流（即cerr使用的错误流）发送 消息abnormal program termination（程序异常终止），然后终止程序。 它还返回一个随实现而异的值，告诉操作系统（如果程序是由另一个程 序调用的，则告诉父进程），处理失败。abort( )是否刷新文件缓冲区 （用于存储读写到文件中的数据的内存区域）取决于实现。如果愿意， 也可以使用exit( )，该函数刷新文件缓冲区，但不显示消息。

   ```c++
   #include <iostream>
   #include <cstdlib>
   
   using std::cin;
   using std::cout;
   using std::endl;
   
   double hmean(double a, double b);
   
   int main()
   {
   	double x, y, z;
   
   	cout << "enter two numbers: ";
   	while(cin >> x >> y)
   	{
   		z = hmean(x, y);
   		cout << "hmean of" << x << " and" << y
   			<< " is " << z << endl;
   		cout << "enter next set of numbers  <q to quit>: ";
   	}
   
   	cout << "Bye";
   
       return 0;
   }
   
   double hmean(double a, double b)
   {
   	if(a == -b)
   	{
   		cout << "untenable arguments to hmean()\n";
   		std::abort();
   	}
   	return 2.0 * a * b / (a + b);
   }
   ```

2. 返回错误码

   比异常终止更灵活的方法是，可使用指针参数或引用参数来将值返回给调用程 序，并使用函数的返回值来指出成功还是失败。

   ```c++
   #include <iostream>
   #include <cfloat>
   
   using std::cin;
   using std::cout;
   using std::endl;
   
   bool hmean(double a, double b, double * ans);
   
   int main()
   {
   	double x, y, z;
   
   	cout << "enter two numbers: ";
   	while(cin >> x >> y)
   	{
   		if(hmean(x, y, &z))
   			cout << "hmean of" << x << " and" << y
   				<< " is " << z << endl;
   		else
   			cout << "one value should not  be the negative "
   				<< "of the other - try again.\n";
   		cout << "enter next set of numbers  <q to quit>: ";
   	}
   
   	cout << "Bye";
   
       return 0;
   }
   
   bool hmean(double a, double b, double * ans)
   {
   	if(a == -b)
   	{
   		*ans = DBL_MAX;
   		return false;
   	}
   	else
   	{
   		*ans = 2.0 * a * b / (a + b);
   		return true;
   	}
   }
   ```

3. 异常机制

   1. throw关键字引发异常，紧随其后的值（例如字符串或对象）指出了异常的特征，执行throw语句类似于执行返回语句，因为它也将终止函数的执 行；但throw不是将控制权返回给调用程序，而是导致程序沿函数调用 序列后退，直到找到包含try块的函数，如果函数引发了异常，而没有try块或没有匹配的处理 程序时，将会发生什么情况。在默认情况下，程序最终将调用abort( ) 函数，但可以修改这种行为。
   2. 异常处理程序（exception handler）来捕获异常，异常处理程序位于要处理问题的程序中。catch关键字表示捕获异常。处理程序以关键字catch开头，随后是位于括号中的类型声明，它指出了异常处理程 序要响应的异常类型；然后是一个用花括号括起的代码块，指出要采取 的措施。
   3. try块标识其中特定的异常可能被激活的代码块，它后面跟一个或多个catch块。try块是由关键字try指示的，关键字try的后面是一个由花括 号括起的代码块，表明需要注意这些代码引发的异常。

   ```c++
   #include <iostream>
   #include <cfloat>
   
   using std::cin;
   using std::cout;
   using std::endl;
   
   bool hmean(double a, double b);
   
   int main()
   {
   	double x, y, z;
   
   	cout << "enter two numbers: ";
   	while(cin >> x >> y)
   	{
   		try
   		{
   			z = hmean(x, y);
   		}
   		catch (const char * s)
   		{
   			cout << s << endl;
   			cout << "enter a new pair of numbers: ";
   			continue;
   		}
   		cout << "hmean of" << x << " and" << y
   				<< " is " << z << endl;
   		cout << "enter next set of numbers  <q to quit>: ";
   	}
   
   	cout << "Bye";
   
       return 0;
   }
   
   bool hmean(double a, double b)
   {
   	if(a == -b)
   		throw "bad hmean() arguments: a = -b note allowed";
   	return 2.0 * a * b / (a + b);
   }
   ```

4. 将对象用作异常类型

   引发异常的函数将传递一个对象。这样做的重要优点之一 是，可以使用不同的异常类型来区分不同的函数在不同情况下引发的异常。另外，对象可以携带信息，程序员可以根据这些信息来确定引发异 常的原因。同时，catch块可以根据这些信息来决定采取什么样的措施。

   ```c++
   //exc_mean.h
   
   #ifndef _EXC_MEAN_H_
   #define _EXC_MEAN_H_
   #include <iostream>
   
   using std::cout;
   
   class bad_hmean
   {
   	private:
   		double v1;
   		double v2;
   	public:
   		bad_hmean(double a = 0, double b = 0) : v1(a), v2(b) {};
   		void mesg();
   };
   
   inline void bad_hmean::mesg()
   {
   	cout << "hmean(" << v1 << ", " << v2 << "): "
   		<< "invalid argument: a = -b\n";
   }
   
   class bad_gmean
   {
       private:
       	double v1;
   		double v2;
   	public:
   		bad_gmean(double a = 0, double b = 0) : v1(a), v2(b) {};
   		const char * mesg();
   };
   
   inline const char * bad_gmean::mesg()
   {
   	return "gmean() arguments should be >= 0\n";
   }
   
   #endif
   ```

   ```c++
   //error3.cpp
   
   #include <iostream>
   #include <cmath>
   #include "exc_mean.h"
   
   using std::cin;
   using std::cout;
   using std::endl;
   
   double hmean(double a, double b);
   double gmean(double a, double b);
   
   int main()
   {
   	double x, y;
   
   	cout << "enter two numbers: ";
   	while(cin >> x >> y)
   	{
   		try
   		{
   			cout << "hmean of" << x << " and" << y
   				<< " is " << hmean(x, y) << endl;
   			cout << "geometric mean of " << x << " and " << y
   				<< " is " << gmean(x, y) << endl;
   			cout << "enter next set of numbers < q to quit>: ";
   		}
   		catch (bad_hmean & bg)
   		{
   			bg.mesg();
   			cout << "try again.\n";
   			continue;
   		}
   		catch (bad_gmean & hg)
   		{
   			cout << hg.mesg();
   			cout << "value used: " << hg.v1 << ", "
   				<< hg.v2 << endl;
   			cout << "sorry, you don't get to play any more\n";
   			break;
   		}
   	}
   
   	cout << "Bye";
   
       return 0;
   }
   
   double hmean(double a, double b)
   {
   	if(a == -b)
   		throw bad_hmean(a, b);
   	return 2.0 * a * b / (a + b);
   }
   
   double gmean(double a, double b)
   {
   	if(a < 0 || b < 0)
   		throw bad_gmean(a, b);
   	return std::sqrt(a * b);
   }
   ```

5. 栈解退

   C++通常通 过将信息放在栈中来处理函数调用。具体地说，程序将 调用函数的指令的地址（返回地址）放到栈中。当被调用的函数执行完 毕后，程序将使用该地址来确定从哪里开始继续执行。另外，函数调用 将函数参数放到栈中。在栈中，这些函数参数被视为自动变量。如果被 调用的函数创建了新的自动变量，则这些变量也将被添加到栈中。如果 被调用的函数调用了另一个函数，则后者的信息将被添加到栈中，依此 类推。当函数结束时，程序流程将跳到该函数被调用时存储的地址处， 同时栈顶的元素被释放。因此，函数通常都返回到调用它的函数，依此 类推，同时每个函数都在结束时释放其自动变量。如果自动变量是类对 象，则类的析构函数（如果有的话）将被调用。现在假设函数由于出现异常（而不是由于返回）而终止，则程序也 将释放栈中的内存，但不会在释放栈的第一个返回地址后停止，而是继续释放栈，直到找到一个位于try块中的返回地址。随后，控制权将转到块尾的异常处理程序，而不是函数调用后面的第一条 语句。这个过程被称为栈解退。引发机制的一个非常重要的特性是，和 函数返回一样，对于栈中的自动类对象，类的析构函数将被调用。然 而，函数返回仅仅处理该函数放在栈中的对象，而throw语句则处理try 块和throw之间整个函数调用序列放在栈中的对象。如果没有栈解退这 种特性，则引发异常后，对于中间函数调用放在栈中的自动类对象，其析构函数将不会被调用。

   ```c++
   #include <iostream>
   #include <cmath>
   #include <string>
   #include "exc_mean.h"
   
   using namespace std;
   
   class demo
   {
   	private:
   		string word;
   	public:
   		demo (const string & str)
   		{
   			word = str;
   			cout << "demo " << word << " created\n";
   		}
   		~demo()
   		{
   			cout << "demo " << word << " destroyed\n";
   		}
   		void show() const
   		{
   			cout << "demo " << word << " lives\n";
   		}
   };
   
   double hmean(double a, double b);
   double gmean(double a, double b);
   double means(double a, double b);
   
   int main()
   {
   	double x, y, z;
   	{
   		demo d1("found in block in main()");
   		cout << "enter two number: ";
   		while(cin >> x >> y)
   		{
   			try
   			{
   				z = means(x, y);
   				cout << "the mean mean of " << x << " and " << y
   					<< " is " << z << endl;
   				cout << "enter next pair: ";
   			}
   			catch(bad_hmean & bg)
   			{
   				bg.mesg();
   				cout << "try again.\n";
   				continue;
   			}
   			catch(bad_gmean & hg)
   			{
   				cout << hg.mesg();
   				cout << "value used: " << hg.v1 << ", "
   					<< hg.v2 << endl;
   				cout << "sorry, you don't get to to play any more.\n";
   				break;
   			}
   		}
   		d1.show();
   	}
   	cout << "bye\n";
   	cin.get();
   	cin.get();
   
   	return 0;
   }
   
   double hmean(double a, double b)
   {
   	if(a == -b)
   		throw bad_hmean(a, b);
   	return 2.0 * a * b / (a + b);
   }
   
   double gmean(double a, double b)
   {
   	if(a < 0 || b < 0)
   		throw bad_gmean(a, b);
   	return sqrt(a * b);
   }
   
   double means(double a, double b)
   {
   	double am, hm, gm;
   	demo d2("found in means()");
   	am = (a + b) / 2.0;
   	try{
   		hm = hmean(a, b);
   		gm = gmean(a, b);
   	}
   	catch(bad_hmean &bg)
   	{
   		bg.mesg();
   		cout << "caught in means()\n";
   		throw;
   	}
   	d2.show();
   	return (am + hm + gm) / 3.0;
   }
   ```

6. 其他异常特性

   引发异常时编译器总是创建一个异常对象的临时拷贝，即使异常规范和catch块中指定的是引用。因为引用还有另一个重要特征：基类引用可以执行派生类对象。假 设有一组通过继承关联起来的异常类型，则在异常规范中只需列出一个 基类引用，它将与任何派生类对象匹配。

   class bad_1 { ... };

   class bad_2 : public bad_1 { ... };

   class bad_3 : public bad_2 { ... };

   void duper()

   {

   ​	...

   ​	if(oh_no)

   ​		throw bad_1();

   ​	if(rats)

   ​		throw bad_2();

   ​	if(drat)

   ​		throw bad_3();

   } 

   try{

   ​	duper();

   }

   catch(bad_3 & be) { //statements }

   catch(bad_2 & be) { //statements }

   catch(bad_1 & be) { //statements }

   如果将bad_1 &处理程序放在最前面，它将捕获异常bad_1、bad_2 和bad_3；通过按相反的顺序排列，bad_3异常将被bad_3 &处理程序所捕获。

   假设您编 写了一个调用另一个函数的函数，而您并不知道被调用的函数可能引发 哪些异常。在这种情况下，仍能够捕获异常，即使不知道异常的类型。 方法是使用省略号来表示异常类型，从而捕获任何异常：

   catch (...) { //statements }

   在catch语句中使用基类对象时，将捕获所有的派生类对象，但派生特性将被剥去，因此将使用 虚方法的基类版本。

7. exception类

   exception头文件（以前为exception.h或except.h）定义了exception类， C++可以把它用作其他异常类的基类。代码可以引发exception异常，也可以将exception类用作基类。

   noexcept是 C++11 引入的关键字，用于指示函数是否抛出异常。它可以在函数声明和定义中使用，以确保特定函数不会抛出任何异常。如果一个函数标记为 noexcept，那么它承诺在运行时不会抛出任何异常。

   1. stdexcept异常类

      头文件stdexcept定义了logic_error和runtime_error类，它们都是以公有方式从exception派生而来的。logic_error系列异常表明存在可以通过编程修复的问题，而runtime_error系列异常表明存在无法避免的问题。所有这些错误 类有相同的常规特征，它们之间的主要区别在于：不同的类名让您能够 分别处理每种异常。另一方面，继承关系让您能够一起处理它们

   2. bad_alloc异常和new

      对于使用new导致的内存分配问题，C++的最新处理方式是让new引 发bad_alloc异常。头文件new包含bad_alloc类的声明，它是从exception 类公有派生而来的。

   3. 空指针和new

      C++标准提供了一种在失败时返回空指针的new，其用法如下：

      typename * pt = new (std::nothrow) typename;

      typename * pt = new (std::nowthrow) typename;

      在分配内存失败时返回一个空指针。这样，程序可以根据返回的指针是否为空来判断内存分配是否成功，而不需要捕获和处理异常。

8. 异常、类和继承

   从一个异常类派生出另一个；其次，可以在类定义中嵌套异 常类声明来组合异常；第三，这种嵌套声明本身可被继承，还可用作基类。

   ```c++
   //use_sales.cpp
   
   #include <iostream>
   #include "sales.h"
   
   int main()
   {
   	using std::cout;
   	using std::cin;
   	using std::endl;
   	
   	double vals1[12] = 
   	{
   		1220, 1100, 1122, 2212, 1232, 2334,
   		2884, 2393, 3302, 2922, 3002, 3544
   	};
   
   	double vals2[12] = 
   	{
   		12, 11, 22, 21, 32, 34,
   		28, 29, 33, 29, 32, 35
   	};
   
   	Sales sales1(2011, vals1, 12);
   	LabeledSales sales2("blogstar", 2012, vals2, 12);
   
   	cout << "First try block:\n";
   	try
   	{
   		int i;
   		cout << "year = " << sales1.Year() << endl;
   		for(i = 0; i < 12; i++)
   		{
   			cout << sales1[i] << " ";
   			if(i % 6 == 5)
   				cout << endl;
   		}
   		cout << "Year = " << sales2.Year() << endl;
   		cout << "Label = " << sales2.Label() << endl;
   		for(i = 0; i <= 12; ++i)
   		{
   			cout << sales2[i] << " ";
   			if(i % 6 == 5)
   				cout << endl;
   		}
   		cout << "end of try block 1.\n";
   	}
   	catch(LabeledSales::nbad_index & bad)
   	{
   		cout << bad.what();
   		cout << "company: " << bad.label_val() << endl;
   		cout << "bad index: " << bad.bi_val() << endl;
   	}
   	catch(Sales::bad_index & bad)
   	{
   		cout << bad.what();
   		cout << "bad index: " << bad.bi_val() << endl;
   	}
   	cout << "\nnext try block:\n";
   	try
   	{
   		sales2[2] = 37.5;
   		sales1[20] = 23345;
   		cout << "end of try block 2.\n";
   	}
   	catch(LabeledSales::nbad_index & bad)
   	{
   		cout << bad.what();
   		cout << "company: " << bad.label_val() << endl;
   		cout << "bad index: " << bad.bi_val() << endl;
   	}
   	catch(Sales::bad_index & bad)
   	{
   		cout << bad.what();
   		cout << "bad index: " << bad.bi_val() << endl;
   	}
   	cout << "done\n";
   
   	return 0;
   }
   ```

   ```c++
   //sales.cpp
   
   #include "sales.h"
   using std::string;
   
   Sales::bad_index::bad_index(int ix, const string & s) : std::logic_error(s), bi(ix)
   {
   
   }
   
   Sales::Sales(int yy) : year(yy)
   {
   	for(int i = 0; i < MONTHS; i++)
   	{
   		gross[i] = 0;
   	}
   }
   
   Sales::Sales(int yy, const double * gr, int n) : year(yy)
   {
   	int lim = (n < MONTHS) ? n : MONTHS;
   	int i;
   	for(i = 0; i < lim; i++)
   		gross[i] = gr[i];
   	for( ; i < MONTHS; i++)
   		gross[i] = 0;
   }
   
   double Sales::operator[](int i) const
   {
   	if(i < 0 || i >= MONTHS)
   		throw bad_index(i);
   	else 
   		return gross[i];
   }
   
   double & Sales::operator[](int i)
   {
   	if(i < 0 || i >= MONTHS)
   		throw bad_index(i);
   	else 
   		return gross[i];
   }
   
   LabeledSales::nbad_index::nbad_index(const string & lb, int ix, const string & s)
   	: Sales::bad_index(ix, s), lbl(lb)
   {
   
   }
   
   LabeledSales::LabeledSales(const string & lb, int yy) : Sales(yy), label(lb)
   {
   
   }
   
   LabeledSales::LabeledSales(const string & lb, int yy, const double * gr, int n)
   	: Sales(yy, gr, n), label(lb)
   {
   
   }
   
   double LabeledSales::operator[](int i) const
   {
   	if(i < 0 || i >= MONTHS)
   		throw nbad_index(Label(), i);
   	else
   		return Sales::operator[](i);
   }
   
   double & LabeledSales::operator[](int i)
   {
   	if(i < 0 || i >= MONTHS)
   		throw nbad_index(Label(), i);
   	else
   		return Sales::operator[](i);
   }
   ```

   ```c++
   //sales.h
   
   #ifndef __SALES_H_
   #define __SALES_H_
   #include <stdexcept>
   #include <string>
   
   class Sales
   {
   	public:
   		enum {MONTHS = 12};
   		class bad_index : public std::logic_error
   		{
   			private:
   				int bi;
   			public:
   				explicit bad_index(int ix, const std::string & s = "Index error in sales object\n");
   				int bi_val() const { return bi; }
   				virtual ~bad_index() throw() {}
   		};
   		explicit Sales(int yy = 0);
   		Sales(int yy, const double * gr, int n);
   		virtual ~Sales() {}
   		int Year() const { return year; }
   		virtual double operator[](int i) const;
   		virtual double & operator[](int i);
   	private:
   		double gross[MONTHS];
   		int year;
   };
   
   class LabeledSales : public Sales
   {
   	public:
   		class nbad_index : public Sales::bad_index
   		{
   			private:
   				std::string lbl;
   			public:
   				nbad_index(const std::string & lb, int ix, 
   						const std::string & s = "index error in labeledSales objext\n");
   				const std::string & label_val() const { return lbl; }
   				virtual ~nbad_index() throw() {}
   		};
   		explicit LabeledSales(const std::string & lb = "none", int yy = 0);
   		LabeledSales(const std::string & lb, int yy, const double * gr, int n);
   		virtual ~LabeledSales() {}
   		const std::string & Label() const { return label; }
   		virtual double operator[](int i) const;
   		virtual double & operator[](int i);
   	private:
   		std::string label;
   };
   ```

9. 异常何时会迷失方向

   c++ primer plus p1346

10. 有关异常的注意事项

    使用异常会增 加程序代码，降低程序的运行速度。异常规范不适用于模板，因为模板 函数引发的异常可能随特定的具体化而异。异常和动态内存分配并非总 能协同工作。

    void test1(int n)

    {

    ​	string mesg("I'm trapped in an endless loop");

    ​	...

    ​	if(oh_no)

    ​		throw exception();

    ​	...

    ​	return;

    }

    string类采用动态内存分配。通常，当函数结束时，将为mesg调用 string的析构函数。虽然throw语句过早地终止了函数，但它仍然使得析 构函数被调用，这要归功于栈解退。

    void test2(int n)

    {

    ​	string * ar = new double[n];

    ​	...

    ​	if(oh_no)

    ​		throw exception();

    ​	...

    ​	delete [] ar;

    ​	return;

    }

    解退栈时，将删除栈中的变量ar。但函数过早的终 止意味着函数末尾的delete[ ]语句被忽略。指针消失了，但它指向的内 存块未被释放，并且不可访问。

    这种泄漏是可以避免的。例如，可以在引发异常的函数中捕获该异 常，在catch块中包含一些清理代码，然后重新引发异常。

    void test3(int n)

    {

    ​	string * ar = new double[n];

    ​	...

    ​	try

    ​	{

    ​		if(oh_no)

    ​			throw exception();

    ​	}

    ​	catch(exception & ex)

    ​	{

    ​		delete [] ar;

    ​		throw;

    ​	}

    ​	...

    ​	delete [] ar;

    ​	return;

    }

### RTTI的工作原理

1.  RTTI的用途

   假设有一个类层次结构，其中的类都是从同一个基类派生而来的， 则可以让基类指针指向其中任何一个类的对象。这样便可以调用这样的 函数：在处理一些信息后，选择一个类，并创建这种类型的对象，然后 返回它的地址，而该地址可以被赋给基类指针。如何知道指针指向的是 哪种对象呢？

   在回答这个问题之前，先考虑为何要知道类型。可能希望调用类方 法的正确版本，在这种情况下，只要该函数是类层次结构中所有成员都 拥有的虚函数，则并不真正需要知道对象的类型。但派生对象可能包含 不是继承而来的方法，在这种情况下，只有某些类型的对象可以使用该 方法。也可能是出于调试目的，想跟踪生成的对象的类型。对于后两种 情况，RTTI提供解决方案。

   只能将RTTI用于包含虚函数的类层次结构，原因在于只有对于这种类层次结构，才应该将派生对象的地址赋给基类指针。

2.  RTTI的工作原理

   1. dynamic_cast运算符

      如果指向的对象（*pt）的类型为Type或者是从Type直接或间接派生而来的类型，则下面的表达式将指针pt转换为Type类型的指针,否则，结果为0，即空指针。然而，当转换的是引用类型时，类型不匹配会抛出 std::bad_cast 异常，而不是返回空值:

      dynamic_cast< Type * >(pt)

      ```c++
      #include <iostream>
      #include <cstdlib>
      #include <ctime>
      
      using namespace std;
      
      class Grand
      {
      	private:
      		int hold;
      	public:
      		Grand(int h = 0) : hold(h) {}
      		virtual void Speak() const { cout << "I am a grand class\n"; }
      		virtual int Value() const { return hold; }
      };
      
      class Superb : public Grand
      {
      	public:
      		Superb(int h = 0) : Grand(h) {}
      		void Speak() const { cout << "I am a superb class\n"; }
      		virtual void say() const { cout << "I hold the superb value of " << Value() << "\n"; }
      };
      
      class Magnificent : public Superb
      {
      	private:
      		char ch;
      	public:
      		Magnificent(int h = 0, char c = 'A') : Superb(h), ch(c) {}
      		void Speak() const { cout << "I  an a magnificent class\n"; }
      		void Say() const { cout << "I hold the character " << ch << " and the integer " << Value() << "\n"; } 
      };
      
      Grand * GetOne();
      
      int main()
      {
      	std::srand(std::time(0));
      	Grand * pg;
      	Superb * ps;
      	for(int i = 0; i < 5; i++)
      	{
      		pg = GetOne();
      		pg->Speak();
      		if(ps = dynamic_cast<Superb *>(pg))
      			ps->say();
      	}
      
          return 0;
      }
      
      Grand * GetOne()
      {
      	Grand * p;
      	switch(std::rand() % 3)
      	{
      		case 0: p = new Grand(std::rand() % 100);
      				break;
      		case 1: p = new Superb(std::rand() % 100);
      				break;
      		case 2: p = new Magnificent(std::rand() % 100, 'A' + std::rand() % 26);
      				break;
      	}
      	return p;
      }
      ```

   2. `dynamic_cast` 在 C++ 中只能用于指针或引用类型，同时也只能转换为指针或引用类型；

   3. typeid运算符和type_info类

      typeid运算符使得能够确定两个对象是否为同种类型，可以接受两种参数：类名和结果为对象的表达式，typeid运算符返回一个对type_info对象的引用，其中，type_info是在 头文件typeinfo（以前为typeinfo.h）中定义的一个类。type_info类重载 了= =和!=运算符，以便可以使用这些运算符来对类型进行比较。

      如果pg指向的是一个Magnificent对象，则下述表达式的结果为bool 值true，否则为false：

      typeid(Magnificent) = typeid(*pg);

      如果pg是一个空指针，程序将引发bad_typeid异常。该异常类型是 从exception类派生而来的，是在头文件typeinfo中声明的。
   
      type_info类包含一个name( )成员，该函数返回一个随实现而异的字符串：通常（但并非一定）是类的名称。

###  类型转换运算符

1. dynamic_cast

2. const_cast

   const_cast运算符用于执行只有一种用途的类型转换，即改变值为 const或volatile

   const_cast < type-name > (expression) 

   除了const或volatile特征（有或无）可以不同外，type_name和 expression的类型必须相同

   提供该运算符的原因是，有时候可能需要这样一个值，它在大多数 时候是常量，而有时又是可以修改的。在这种情况下，可以将这个值声 明为const，并在需要修改它的时候，使用const_cast。

   High bar;

   const High * pbar = &bar;

   ...

   High * pb = const_cast< High * > (pbar); //vaild

   const Low * pb = const_cast< const Low * > (pbar); //invaild;

   ```c++
   #include <iostream>
   
   using namespace std;
   
   void change(const int * pt, int n);
   
   int main()
   {
   	int pop1 = 38383;
   	const int pop2 = 2000;
   
   	cout << "pop1, pop2: " << pop1 << ", " << pop2 << endl;
   	change(&pop1, -103);
   	change(&pop2, -103);
   	cout << "pop1, pop2: " << pop1 << ", " << pop2 << endl;
   
   	return 0;
   }
   
   void change(const int * pt, int n)
   {
   	int * pc;
   
   	pc = const_cast<int *>(pt);
   	*pc += n;
   }
   ```

   const_cast运算符可以删除const int* pt中的const，使得编译器能够 接受change( )中的语句：*pc += n; 但由于pop2被声明为const，虽然编译器会接受使用 const_cast 进行类型转换，但如果尝试修改实际声明为 const 的对象，结果将是未定义行为。

3. static_cast

   static_cast < type-name > (expression)

   仅当type_name可被隐式转换为expression所属的类型或expression可被隐式转换为type_name所属的类型时，上述转换才是合法的，否则将出错。

   1. 从派生类指针到基类指针的转换：这是隐式转换，可以直接使用。
   2. 从基类指针到派生类指针的转换：这需要 static_cast 或者更安全的 dynamic_cast，但编译器必须能够确定转换的安全性。
   3. 从基本类型到枚举类型的转换：也必须是合法的，即基本类型的值必须在枚举类型的范围内。
   4. 类型之间的隐式转换：如果一种类型可以隐式转换为另一种类型，那么 static_cast 可以执行该转换。

4. reinterpret_cast

   reinterpret_cast < type-name > (expression)

   reinterprete_cast运算符并不支持所有的类型转换，不允许删除 const，可以将指针类型转换为足以存储指针表示的整型，但不能将指针转换为更 小的整型或浮点型。另一个限制是，不能将函数指针转换为数据指针， 反之亦然。

   ```c++
   #include <iostream>
   
   using namespace std;
   
   struct dat { 
       short a; 
       short b; 
   };
   
   int main() {
       long value = 0xA224B118;
       dat *pd = reinterpret_cast<dat *>(&value);
       
       cout << hex << pd->a;
       
       return 0;
   }
   ```

## string类和标准模板库

### string 类

1. 构造字符串

   1. string(const char * s) ： 将string对象初始化为s指向的字符数组；
   2. string(size_type n, char c) ： 创建一个包含n个元素的string对象，其中每个元素都被 初始化为字符c；
   3. string(const string & str) ：将一个string对象初始化为string对象str（复制构造函 数）；
   4. string( )  ： 创建一个默认的sting对象，长度为0（默认构造函数）；
   5. string(const char * s, size_type n) ： 将string对象初始化为s指向的字符数组的前n个字符，即使 超过了字符数组结尾；
   6. template < class Iter > string(Iter begin, Iter end) : 将string对象初始化为区间[begin, end)内的字符，其中 begin和end的行为就像指针，用于指定位置，范围包括 begin在内，但不包括end；
   7. string(const string & str, string size_type pos = 0, size_type n = npos) : 将一个string对象初始化为对象str中从位置pos开始到结尾的字符，或从位置pos开始的n个字符；
   8. string(string && str) noexcept : 构造函数string（string && str）类似于复制构造函数，导致新创建 的string为str的副本。但与复制构造函数不同的是，它不保证将str视为 const。这种构造函数被称为移动构造函数（move constructor）；
   9. string(initializer_list il) : 它将一个string对象初始化为初始化 列表il中的字符；

2. string类输入

   string版本的getline( )将自动调整目标string对象的大小，使之刚好能够存储输入的字符，不需要指定读取多少个 字符的数值参数

   string版本的getline( )函数从输入中读取字符，并将其存储到目标 string中，直到发生下列三种情况之一：

   1. 到达文件尾，在这种情况下，输入流的eofbit将被设置，这意味着 方法fail( )和eof( )都将返回true；
   2. 遇到分界字符（默认为\n），在这种情况下，将把分界字符从输入 流中删除，但不存储它；
   3. 读取的字符数达到最大允许值（string::npos和可供分配的内存字节 数中较小的一个），在这种情况下，将设置输入流的failbit，这意 味着方法fail( )将返回true;

3. 使用字符串

4. string还提供了哪些功能

   1. 自动调整大小的功能

      将其他字符附加到字符串末尾时，不能仅仅将已有的字符串加 大，因为相邻的内存可能被占用了。因此，可能需要分配一个新的内存 块，并将原来的内容复制到新的内存单元中。如果执行大量这样的操作，效率将非常低，因此很多C++实现分配一个比实际字符串大的内存 块，为字符串提供了增大空间。然而，如果字符串不断增大，超过了内 存块的大小，程序将分配一个大小为原来两倍的新内存块，以提供足够 的增大空间，避免不断地分配新的内存块。方法capacity( )返回当前分配给字符串的内存块的大小，而reserve( )方法让您能够请求内存块的最小长度。

      ```c++
      #include <iostream>
      #include <string>
      
      using namespace std;
      
      int main()
      {
      	string empty;
      	string small = "bit";
      	string larger = "Elephants are a girl's best friend";
      	cout << "size:\n";
      	cout << "\tempty: " << empty.size() << endl;
      	cout << "\tsmall: " << small.size() << endl;
      	cout << "\tlarger: " << larger.size() << endl;
      	cout << "capacities:\n";
      	cout << "\tempty: " << empty.capacity() << endl;
      	cout << "\tsmall: " << small.capacity() << endl;
      	cout << "\tlarger: " << larger.capacity() << endl;
      	empty.reserve(50);
      	cout << "capacity after empty.reserve(50): "
      		<< empty.capacity() << endl;
      
          return 0;
      }
      ```

   2. 如果您有string对象，但需要C-风格字符串，该如何办呢？例如， 您可能想打开一个其名称存储在string对象中的文件：

      string filename;

      cout << "Enter file name: ";

      cin >> filename;

      ofstream fout;

      open( )方法要求使用一个C-风格字符串作为参数，而c_str( )方法返回一个指向C-风格字符串的指针，该C-风格字符串 的内容与用于调用c_str( )方法的string对象相同。因此可以这样做：

      fout.open(filename.c_str());

5.  字符串种类

   string库实际上是基于一个模板类的：

   template < class charT, class traits = char_traits< charT >, 

   ​	class ALLocator = allocator< charT > >

   basic_string { ... };

   模板basic_string有4个具体化，每个具体化都有一个typedef名称：

   typedef basic_string < char > string;

   typedef basic_string < wchar_t > wstring;

   typedef basic_string < char16_t > u16string;

   typedef basic_string < char32_t > u32string;

   traits类描述关于选定字符类型的特定情况， 如如何对值进行比较。对于wchar_t、char16_t、char32_t和char类型，有预定义的char_traits模板具体化，它们都是traits的默认值。Allocator是一 个管理内存分配的类。对于各种字符类型，都有预定义的allocator模板 具体化，它们都是默认的。它们使用new和delete。

###  智能指针模板类

1.  使用智能指针

   delete 和 delete[] 只负责释放内存，不会清空内存内容。也就是说，内存中的数据在释放后依然存在，只是这块内存不再属于程序，可以被分配给其他用途。

   智能指针模板（auto_ptr、unique_ptr和shared_ptr）都定义了 类似指针的对象，可以将new获得（直接或间接）的地址赋给这种对象。当智能指针过期时，其析构函数将使用delete来释放内存并清空内存内容。

   要创建智能指针对象，必须包含头文件memory，该文件模板定义。然后使用通常的模板语法来实例化所需类型的指针。

   ```c++
   #include <iostream>
   #include <string>
   #include <memory>
   
   using namespace std;
   
   class Report
   {
   	private:
   		string str;
   	public:
   		Report(const string s) : str(s) { cout << "object created\n"; }
   		~Report() { cout << "object deleted\n"; }
   		void comment() const { std::cout << str << "\n"; }
   };
   
   int main()
   {
   	{
   		auto_ptr<Report> ps (new Report("using auto_ptr")); //c++11弃用
   		ps->comment();
   	}
   	{
   		shared_ptr<Report> ps (new Report("using shared_ptr")); //提供共享所有权，可以有多个指针指向同一个对象。当最后一个指针超出作用域时，对象才会被销毁。
   		ps->comment();
   	}
   	{
   		unique_ptr<Report> ps (new Report("using unique_ptr")); //提供独占所有权，因此只能有一个指针指向特定的对象。
   		ps->comment();
   	}
   
       return 0;
   }
   ```

   所有智能指针类都一个explicit构造函数，该构造函数将指针作为参数，只能通过构造函数显示转换

   对全部三种智能指针都应避免的一点：

   不要将栈上或全局变量的地址传递给智能指针。这会导致智能指针尝试删除栈上或全局变量，造成未定义行为。使用new分配内存时，才能使用auto_ptr和shared_ptr，使用new [ ]分配内存时，不能使用它 们。不使用new分配内存时，不能使用auto_ptr或shared_ptr；不使用new或new []分配内存时， 不能使用unique_ptr。

2. 有关智能指针的注意事项

   auto_ptr< string > ps (new string("hello"));

   auto_ptr< string > vocation;

   vocation = ps;

   这导致ps不再引用该字符串。在vocation 放弃对象的所有权 后，便可能使用它来访问该对象。当程序打印ps指向的字符串时，却发现这是一个空指针。另外，在程序块结束后，将试图删除同一个对象两次。

   1. 定义赋值运算符，使之执行深复制。这样两个指针将指向不同的对象，其中的一个对象是另一个对象的副本。 
   2. 建立所有权（ownership）概念，对于特定的对象，只能有一个智能指针可拥有它，这样只有拥有对象的智能指针的构造函数会删除该对象。然后，让赋值操作转让所有权。这就是用于auto_ptr和 unique_ptr的策略，但unique_ptr的策略更严格。 因此上述代码如使用unique_ptr将会在编译阶段出错，编译阶段错误比潜在的程序崩溃更安全。
   3. 创建智能更高的指针，跟踪引用特定对象的智能指针数。这称为引 用计数（reference counting）。例如，赋值时，计数将加1，而指针过期时，计数将减1。仅当最后一个指针过期时，才调用delete。这 是shared_ptr采用的策略。

3. unique_ptr优于auto_ptr

   unique_ptr< string > demo(const char * s)

   {

   ​	unique_ptr< string > temp(new string(s));

   ​	return temp;

   }

   unique_ptr< string > ps;

   ps = demo("uniquely special");

   demo( )返回一个临时unique_ptr，然后ps接管了原本归返回的 unique_ptr所有的对象，而返回的unique_ptr很快被销毁，没有机会使用它来访问无效的数据。换句话说，没有理由禁止这种赋值。

   unique_ptr < string > pu1(new string "Hi ho");

   unique_ptr < string > pu2;

   pu2 = pu1; //#1 invalid

   unique_ptr< string > pu3;

   pu3 = unique_ptr<  string >(new string "yo"); //#2 valid

   语句#1将留下悬挂的unique_ptr（pu1），这可能导致危害。语句#2 不会留下悬挂的unique_ptr，因为它调用unique_ptr的构造函数，该构造 函数创建的临时对象在其所有权转让给pu3后就会被销毁。这种随情况而 异的行为表明，unique_ptr优于允许两种赋值的auto_ptr。这也是禁止 （只是一种建议，编译器并不禁止）在容器对象中使用auto_ptr，但允 许使用unique_ptr的原因。如果容器算法试图对包含unique_ptr的容器执行类似于语句#1的操作，将导致编译错误；如果算法试图执行类似于语句#2的操作，则不会有任何问题。而对于auto_ptr，类似于语句#1的操作可能导致不确定的行为和神秘的崩溃。

   unique_ptr< string > ps1, ps2;

   ps1 = demo("hello");

   ps2 = move(ps1); //C++有一个标准库函数std::move( )， 让您能够将一个unique_ptr赋给另一个，该函数返回一个unique_ptr对象

   ps1 = demo("hi");

   相比于auto_ptr，unique_ptr还有另一个优点。它有一个可用于数组的变体：

   unique_ptr< double[] > pt(new double(5));

4.  选择智能指针

###  标准模板库

1. 模板类vector

2. 可对矢量(vector)执行的操作

   1. 迭代器iterator：vector< type >::iterator pd;
   2. begin( )——返回一个指向容器中第一个元素的迭代器；
   3. end( )——返回一个表示超过容器尾的迭代器；
   4. push_back( )将元素添加到矢量末尾；
   5. erase( )方法删除矢量中给定区间的元素，接受两个迭代器参数， 这些参数定义了要删除的区间；
   6. insert( )方法接受3个迭代器参数，第一个 参数指定了新元素的插入位置，第二个和第三个迭代器参数定义了被插 入区间，该区间通常是另一个容器对象的一部分

3. 对矢量可执行的其他操作

   1. for_each( )它接受3个参数。前两个 是定义容器中区间的迭代器，最后一个是指向函数的指针，函数将被指向的函数应用于容器区间中的各个元素，传递给函数的元素是通过值传递的，因此不能直接修改容器的内容：

      for_each(ob.begin(), ob.end(), func);

   2. Random_shuffle( )函数接受两个指定区间的迭代器参数，并随机排 列该区间中的元素，该函数要求容器类允许随机访问：

      Random_shuffle(ob.begin(), ob.end()）

   3. sort( )函数也要求容器支持随机访问。该函数有两个版本

      1. 第一个版本接受两个定义区间的迭代器参数，并使用为存储在容器中的类型元素定义的 < 运算符，对区间中的元素进行升序操作。如果容器元素是用户定义的对象，则要使用sort( )，必须定义能够 处理该类型对象的operator<( )函数；
      2. 如果想按降序或是其他排列方式，它接受3个参数，前两个参数也是指定区间的迭代器，最后一个参数是指向要使用的 函数的指针（函数对象）

4.  基于范围的for循环

   for(auto & x : ob) func(x); //按引用传递可修改容器的内容

###  泛型编程

1. 为何使用迭代器

2. 迭代器类型

   1.  输入迭代器
      1. 只能从起点向终点单向遍历；
      2. 从一次性的数据源读取数据，每个元素只能读取一次；
      3. 可以递增迭代器 ++ ，使其指向下一个元素；
      4. 可以解引用 * 以访问当前元素
   2. 输出迭代器
      1. 只能从起点向终点单向遍历；
      2. 可以写入元素，但不支持读取元素；
      3. 可以递增迭代器 ++ ，使其指向下一个位置；
      4. 可以解引用 * 并赋值以写入数据；
   3. 正向迭代器
      1. 只能从起点向终点单向遍历；
      2. 可以读取元素，也可以写入元素；
      3. 可以递增迭代器 ++，使其指向下一个元素；
      4. 解引用 * 以访问和修改当前元素；
      5. 正向迭代器可以被多次遍历，每次遍历都以相同的顺序访问元素；
   4. 双向迭代器
      1. 双向迭代器具有正向迭代器的所有特性；
      2. 同时支持两种（前缀和后缀）递减运算符；
   5. 随机访问迭代器
      1.  随机访问迭代器具有双向迭代器的所有特性；
      2. 同时添加了支持随机访问的操作和用于对元素进行排序的关系运算符；

3. 迭代器层次结构

   各种迭代器的类型并不是确定的，而只是一种概念性描述。每个容器类都定义了一个类级typedef名称—— iterator，因此vector类的迭代器类型为vector :: interator。然而，该类的文档将指出，矢量迭代器是随机访问迭代器，它允许使用基于任何迭代器类型的算法，因为随机访问迭代器具有所有迭代器的功能。同样，list类的迭代器类型为list :: iterator。STL实现了一个双向链表，它使用双向迭代器，因此不能使用基于随机访问迭代器的算法，但可以使用基于要求较低的迭代器的算法。

4.  概念、改进和模型

   1. 将指针用作迭代器

      迭代器是广义指针，而指针满足所有的迭代器要求。迭代器是STL 算法的接口，而指针是迭代器，因此STL算法可以使用指针来对基于指针的非STL容器进行操作，例如，可将STL算法用于数组。假设Receipts是一个double数组，并要按升序对它进行排序：

      const int SIZE = 100;

      double Receipts[SIZE];

      sort(Receipts, Receipts + SIZE);

      C++确保了表达式Receipts + n是被定义的，只要该表达式的结果位 于数组中。因此，C++支持将超尾概念用于数组，使得可以将STL算法 用于常规数组。由于指针是迭代器，而算法是基于迭代器的，这使得可 将STL算法用于常规数组。同样，可以将STL算法用于自己设计的数组 形式，只要提供适当的迭代器（可以是指针，也可以是对象）和超尾指 示器即可。

   2. 其他有用的迭代器

      头文件iterator还提供了其他一些专用的预定义迭代器类型。它们是reverse_iterator、 back_insert_iterator、front_insert_iterator和insert_iterator

      1. ostream_iterator

         #include < iterator >

         ...

         ostream_iterator< int, char > out_iter(cout, " ");

         第 一个模板参数（这里为int）指出了被发送给输出流的数据类型；第二个 模板参数（这里为char）指出了输出流使用的字符类型（另一个可能的 值是wchar_t）。构造函数的第一个参数（这里为cout）指出了要使用的 输出流，它也可以是用于文件输出的流，最后一个字符 串参数是在发送给输出流的每个数据项后显示的分隔符；

         *out_iter++ = 15; //等价于 cout << 15 << " ";

         将15和由空格组成的字符串发送 到cout管理的输出流中，并为下一个输出操作做好了准备；

         copy(dice.begin(), dice.end(), out_iter); //ob容器的整个区间复制到输出流中，即显示容器的内容。

      2. istream_iterator

         copy(istream_iterator< int, char>(cin), istream_iterator< int, char>(), dice.begin()); //不创建命名的迭代器，直接构建匿名迭代器，推荐使用

         第一 个参数指出要读取的数据类型，第二个参数指出输入流使用的字符类型。使用构造函数参数cin意味着使用由cin管理的输入流，省略构造函 数参数表示输入失败，因此上述代码从输入流中读取，直到文件结尾、 类型不匹配或出现其他输入故障为止。

      3. reverse_iterator

         reverse_iterator执行递增操作 将导致它被递减。vector类有一个名为rbegin( )的成员函数和一个名 为rend( )的成员函数，前者返回一个指向超尾的反向迭代器，后者返回 一个指向第一个元素的反向迭代器。因为对迭代器执行递增操作将导致 它被递减，所以可以使用下面的语句来反向显示内容：

         copy(dice.rbegin(), dice.rend(), out_iter)；

         rbegin( )和end( )返回相同的值（超尾），但类型不同（reverse_iterator和iterator）。同样， rend( )和begin( )也返回相同的值（指向第一个元素的迭代器），但类型不同。

         因为rbegin( )返回超尾，因此不能对该地址进行解除引用。同样，如果rend( )是第一个元素的位 置，则copy( )必须提早一个位置停止，因为区间的结尾处不包括在区间中。 反向指针通过先递减，再解除引用解决了这两个问题。即* rp将在* rp的当前值之前对迭代器执行解除引用。也就是说，如果rp指向位置 6，则* rp将是位置5的值，依次类推。

         ```c++
         #include <iostream>
         #include <iterator>
         #include <vector>
         
         using namespace std;
         
         int main()
         {
         	int casts[10] = {5, 92, 65, 2, 14, 20, 3, 61, 18, 9};
         	vector<int> dice(10);
         	copy(casts, casts + 10, dice.begin());
         	cout << "let the dice be cast\n";
         	ostream_iterator<int, char> out_iter(cout, " ");
         	copy(dice.begin(), dice.end(), out_iter);
         	cout << endl;
         	cout << "implicit use of reverse iterator\n";
         	copy(dice.rbegin(), dice.rend(), out_iter);
         	cout << endl;
         	cout << "explicit use of reverse iterator\n";
         	vector<int>::reverse_iterator ri;
         	for(ri = dice.rbegin(); ri != dice.rend(); ++ri)
         		cout << *ri << ' ';
         	cout << endl;
         
             return 0;
         }
         ```

      4. back_insert_iterator、front_insert_iterator和 insert_iterator

         三种插入迭代器将添 加新的元素，而不会覆盖已有的数据，并使用自动内存分配来确保能够 容纳新的信息。back_insert_iterator将元素插入到容器尾部，而 front_insert_iterator将元素插入到容器的前端。最后，insert_iterator将元素插入到insert_iterator构造函数的参数指定的位置前面。这三个插入迭代器都是输出容器概念的模型。这里存在一些限制。back_insert_iterator只能用于允许在尾部快速插入的容器（快速插入指的是一个时间固定的算法），vector类符合这种要求。 front_insert_iterator只能用于允许在起始位置做时间固定插入的容器类 型，vector类不能满足这种要求，但queue满足。insert_iterator没有这些 限制，因此可以用它把信息插入到矢量的前端。然而， front_insert_iterator对于那些支持它的容器来说，完成任务的速度更快。

      5. 这些迭代器将容器类型作为模板参数，将实际的容器标识符作为构造函数参数。

         back_insert_iterator< vector< int > > back_iter(dice)

         必须声明容器类型的原因是，迭代器必须使用合适的容器方法。 back_insert_iterator的构造函数将假设传递给它的类型有一个push_back( ) 方法。copy( )是一个独立的函数，没有重新调整容器大小的权限。但前面的声明让back_iter能够使用方法vector::push_back( )，该方法有这样的权限。

         ```c++
         #include <iostream>
         #include <string>
         #include <iterator>
         #include <vector>
         #include <algorithm>
         
         using namespace std;
         
         void output(const string & s) { cout << s << " "; }
         
         int main()
         {
         	string s1[4] = {"fine", "fish", "fashion", "fate"};
         	string s2[2] = {"busy", "bats"};
         	string s3[2] = {"silly", "singers"};
         	vector<string> words(4);
         	copy(s1, s1 + 4, words.begin());
         	for_each(words.begin(), words.end(), output);
         	cout << endl;
         	copy(s2, s2 + 2, back_insert_iterator<vector<string>>(words));
         	for_each(words.begin(), words.end(), output);
         	cout << endl;
         	copy(s3, s3 + 2, insert_iterator<vector<string>>(words, words.begin()));
         	for_each(words.begin(), words.end(), output);
         	cout << endl;
         
             return 0;
         }
         ```

5. 容器种类

   1. 容器概念

   2. C++11新增的容器要求

   3. 序列模型

      1. vector

         1. 可反转容器，提供rend()和rbegin()方法；
         2. 支持随机访问；
         3. 在尾部添加和删除元素的时间是固定的，但在头部或中间插入和删除元素的复杂度为线性时间；
         4. vector模板类是最简单的序列类型，除非其他类型的特殊优点能够更好地满足程序的要求，否则应默认使用这种类型；

      2. deque

         1. 可反转容器；
         2. 支持随机访问；
         3. 开始位置以及结束位置插入和删除元素的时间是固定的；
         4. 如果多数操作发生在序列的起始和结尾处，则应考虑使用deque数据结构。为实现在deque两端执行插入和删除操作的时间为固定的这一目的，deque对象的设计比vector对象更为复杂。因此，尽管二者都提供对元素的随机访问和在序列中部执行线性时间的插入和删除操作，但 vector容器执行这些操作时速度要快些。

      3. list(双向链表)

         1. 可反转容器；
         2. 不支持随机访问；
         3. 任一位置进行插入和删除的时间都是固定的；
         4. list也可使用C++11的初始化列表功能来简化初始化；
         5. 与矢量迭代器不同，从容器中插入或删除元素 之后，链表迭代器指向元素将不变。例如，假 设有一个指向vector容器第5个元素的迭代器，并在容器的起始处插入一 个元素。此时，必须移动其他所有元素，以便腾出位置，因此插入后， 第5个元素包含的值将是以前第4个元素的值。因此，迭代器指向的位置 不变，但数据不同。然后，在链表中插入新元素并不会移动已有的元 素，而只是修改链接信息。指向某个元素的迭代器仍然指向该元素，但它链接的元素可能与以前不同。

         ```c++
         #include <iostream>
         #include <list>
         #include <iterator>
         #include <algorithm>
         
         void outint(int n) { std::cout << n << " "; }
         
         int main()
         {
         	using namespace std;
         
         	list<int> one(5, 2);
         	int stuff[5] = {1, 2, 4, 8, 6};
         	list<int> two;
         	two.insert(two.begin(), stuff, stuff + 5);
         	int more[6] = {6, 4, 2, 4, 6, 5};
         	list<int> three(two);
         	three.insert(three.end(), more, more + 6);
         
         	cout << "List one: ";
         	for_each(one.begin(), one.end(), outint);
         	cout << endl << "List two: ";
         	for_each(two.begin(), two.end(), outint);
         	cout << endl << "List three: ";
         	for_each(three.begin(), three.end(), outint);
         	three.remove(2);
         	cout << endl << "List three minus 2s: ";
         	for_each(three.begin(), three.end(), outint);
         	three.splice(three.begin(), one);
         	cout << endl << "List three after splice: ";
         	for_each(three.begin(), three.end(), outint);
         	cout << endl << "List one: ";
         	for_each(one.begin(), one.end(), outint);
         	three.unique();
         	cout << endl << "List three after unique: ";
         	for_each(three.begin(), three.end(), outint);
         	three.sort();
         	three.unique();
         	cout << endl << "List three after sort and unique: ";
         	for_each(three.begin(), three.end(), outint);
         	two.sort();
         	three.merge(two);
         	cout << endl << "Sorted two merged into three: ";
         	for_each(three.begin(), three.end(), outint);
         	cout << endl;
         	
             return 0;
         }
         ```

      4. list工具箱

      5. forward_list（C++11）

         1. 单链表；
         2. 不可反转的容器；
         3. 不支持随机访问；
         4. 任一位置进行插入和删除的时间都是固定的;

      6. queue(适配器类)

         1. 不允许随机访问；
         2. 不允许遍历队列；
         3. 可以将元素添加到队尾、从队首删除元素、查看队首和队尾的值、检查元素数目和测试队列是否为空；

      7. priority_queue(适配器类)

         支持的操作与queue相同。两者之间的主要区别在于，在priority_queue中，最大的元素被移到队首。内部区别在于，默认的底层类是vector。可以修改用于确定哪 个元素放到队首的比较方式，方法是提供一个可选的构造函数参数；

         priority_queue< int > pq1;

         priority_queue< int > pq2(greater < int >); //greater< >( )函数是一个预定义的函数对象

      8. stack(适配器类)

         1. 它给底层类（默认情况下为vector）提供了典型的栈接口；
         2. 不允许随机访问；
         3. 不允许遍历；
         4. 可以将压入推到栈顶、从栈顶弹出元素、查看栈顶的值、检查元素数目和测试栈是 否为空；

      9. array（C++11）

         模板类array是头文件array中定义的，它并非STL容器，因为其长度是固定的。因此，array没有定义调整容器大小的操作，可将很多标准STL算法用于array对象，如copy( )和for_each( )。

6. 关联容器：

   关联容器（associative container）是对容器概念的另一个改进。关联容器将值与键关联在一起，并使用键来查找值，因此关键容器中的元素是按键有序排列的。对于容器X，表达式X::value_type通常指出了存储在容器中的值类型。对于关联容器来说，表达式 X::key_type指出了键的类型。关联容器通常是使用某种树实现的。树是一种数据结构，其根节点链接到一个或两个节点，而这些节点又链接到一个或两个节点，从而形 成分支结构。像链表一样，节点使得添加或删除数据项比较简单；但相对于链表，树的查找速度更快。因此，与序列相似， 关联容器也允许插入新元素，但不能指定元素的插入位置。

   STL提供了4种关联容器：set、multiset、map和multimap。

   1. set

      关联集合，可反转，可排序，集合中的每个元素既是键也是值，且键是唯一的，所以不能存储多个相同的值。

      set< Type > A; //a set of Type objects

      第二个模板参数是可选的，可用于指示用来对键进行排序的比较函数或对象。默认情况下，将使用模板less< Type >

      ```c++
      #include <iostream>
      #include <string>
      #include <set>
      #include <algorithm>
      #include <iterator>
      
      int main()
      {
      	using namespace std;
      	const int N = 6;
      	string s1[N] = {"bufoon", "thinker", "for", "heave", "can", "for"};
      	string s2[N] = {"metal", "any", "food", "elegant", "deliver", "for"};
      	
      	set<string> A(s1, s1 + N);
      	set<string> B(s2, s2 + N);
      
      	ostream_iterator<string, char> out(cout, " ");
      	cout << "Set A: ";
      	copy(A.begin(), A.end(), out);
      	cout << endl;
      	cout << "Set B: ";
      	copy(B.begin(), B.end(), out);
      	cout << endl;
      
      	cout << "Union of A and B: ";
      	set_union(A.begin(), A.end(), B.begin(), B.end(), out);
      	cout << endl;
      
      	cout << "Intersection of A and B: ";
      	set_intersection(A.begin(), A.end(), B.begin(), B.end(), out);
      	cout << endl;
      
      	cout << "Difference of A and B: ";
      	set_difference(A.begin(), A.end(), B.begin(), B.end(), out);
      	cout << endl;
      
      	set<string> C;
      	cout << "Set C:\n";
      	set_union(A.begin(), A.end(), B.begin(), B.end(), insert_iterator<set<string>>(C, C.begin()));
      	copy(C.begin(), C.end(), out);
      	cout << endl;
      
      	string s3("grungy");
      	C.insert(s3);
      	cout << "Set C after insertion:\n";
      	copy(C.begin(), C.end(), out);
      	cout << endl;
      
      	cout << "Showing a range:\n";
      	copy(C.lower_bound("grungy"), C.upper_bound("metal"), out);
      	cout << endl;
      
          return 0;
      }
      ```

   2. multimap

      与set相似，multimap也是可反转的、经过排序的关联容器，但键和值的类型不同，且同一个键可能与多个值相关联。

      multimap< keyType, valueType > codes; 

      实际的值类型将键类型和数据类型结合为一对。STL使用模板类pair< class T, class U >将这两种值存储到一个对象中。因此，实际的值类型为pair< const keyType, valueType >，对于pair对象，使用first和second成员来访问其两个部分

      第3个模板参数是可选的，指出用于对键进行排序的比较函数或对象。在默认情况下，将使用模板less< Type >

      ```c++
      #include <iostream>
      #include <string>
      #include <map>
      #include <algorithm>
      
      typedef int KeyType;
      typedef std::pair<const KeyType, std::string> Pair;
      typedef std::multimap<KeyType, std::string> MapCode;
      
      int main()
      {
      	using namespace std;
      	MapCode codes;
      
      	codes.insert(Pair(415, "San Francisco"));
      	codes.insert(pair(510, "Oakland"));
      	codes.insert(Pair(718, "Brooklyn"));
      	codes.insert(Pair(718, "Staten Island"));
      	codes.insert(pair(415, "San Rafael"));
      	codes.insert(pair(510, "Berkeley"));
      
      	cout << "Number of cities with area code 415: "
      		 << codes.count(415) << endl;
      	cout << "Number of cities with area code 718: "
      		 << codes.count(718) << endl;
      	cout << "Number of cities with area code 510: "
      		 << codes.count(510) << endl;
      	cout << "Area Code City\n";
      	MapCode::iterator it;
      	for(it = codes.begin(); it != codes.end(); ++it)
      		cout << "	" << (*it).first << "	   "
      			 << (*it).second << endl;
      
      	pair<MapCode::iterator, MapCode::iterator> range
      		= codes.equal_range(718);
      	cout << "Cities with area code 718:\n";
      	for(it = range.first; it != range.second; ++it)
      		cout << (*it).second << endl;
      
          return 0;
      }
      ```

7. 无序关联容器（C++11）

   与关联容器一样，无序关联容器也将值与键关联起来，并使用键来查找值。但底层的差别在 于，关联容器是基于树结构的，而无序关联容器是基于数据结构哈希表的，这旨在提高添加和删除元素的速度以及提高查找算法的效率。有4 种无序关联容器，它们是unordered_set、unordered_multiset、 unordered_map和unordered_multimap

###  函数对象

1. 函数符概念

   1. 函数符是可以以函数方式与( )结合使用的任意对象。这包括函数名、指向函数 的指针和重载了( )运算符的类对象（即定义了函数operator( )( )的类）；
   2. 生成器（generator）是不用参数就可以调用的函数符。 一元函数（unary function）是用一个参数可以调用的函数符。 二元函数（binary function）是用两个参数可以调用的函数符。返回bool值的一元函数是谓词（predicate）； 返回bool值的二元函数是二元谓词（binary predicate）。

   list模板有一个将谓词作为参数的remove_if( )成员，该函数将谓词 应用于区间中的每个元素，如果谓词返回true，则删除这些元素。下面的代码中，一个值（V）作为函数参数传递，而第二个参数（cutoff）是 由类构造函数设置的。有了该定义后，就可以将不同的TooBig对象初始 化为不同的取舍值，供调用remove_if( )时使用。

   ```c++
   #include <iostream>
   #include <list>
   #include <iterator>
   #include <algorithm>
   
   template<class T>
   class TooBig
   {
   	private:
   		T cutoff;
   	public:
   		TooBig(const T & t) { cutoff = t; }
   		bool operator()(T & val) { return val > cutoff; }
   };
   
   void outint(int n) { std::cout << n << " "; }
   
   int main()
   {
   	using std::list;
   	using std::cout;
   	using std::endl;
   
   	TooBig<int> f100(100);
   	int vals[10] = {50, 100, 90, 180, 60, 210, 415, 88, 188, 201};
   	list<int> yadayada = {50, 100, 90, 180, 60, 210, 415, 88, 188, 201}; //c++11列表初始化
   	list<int> etcetera{vals, vals + 10};
   	cout << "Original lists:\n";
   	for_each(yadayada.begin(), yadayada.end(), outint);
   	cout << endl;
   	for_each(etcetera.begin(), etcetera.end(), outint);
   	cout << endl;
   	yadayada.remove_if(f100);
   	etcetera.remove_if(TooBig<int>(200));
   	cout << "Trimmed lists;\n";
   	for_each(yadayada.begin(), yadayada.end(), outint);
   	cout << endl;
   	for_each(etcetera.begin(), etcetera.end(), outint);
   	cout << endl;
   
       return 0;
   }
   ```

   假设已经有了一个接受两个参数的函数模板：

   template < class T >

   bool tooBig(const T & val, const T & lim)

   {

   ​	return val  > lim;

   }

   则可以使用类将它转换为单个参数的函数对象：

   template < class T >

   class TooBig2

   {

   ​	private:

   ​		T cutoff;

   ​	public:

   ​		TooBig2(const T & t) : cutoff(t) {}

   ​		bool operator()(const T & v) { return tooBig< T >(v, cutoff); }

   }

2. 预定义的函数符

   对于所有内置的算术运算符、关系运算符和逻辑运算符，STL都提供了等价的函数符;它们可以用于处理C++内置类型或任何用户定义类型（如果重载了相应的运算符）。

3. 函数适配器

   预定义函数符都是自适应的。使函数符成为自适应的原因是通过定义一些 typedef 成员，使得它们能够与函数适配器一起工作。这些 typedef 成员包括 result_type、first_argument_type 和 second_argument_type，它们分别标识了函数对象的返回类型、第一参数类型和第二参数类型。函数适配器是用于修改或组合函数对象的工具，使用这些 typedef 成员，可以使得适配器更加通用和强大。例如，std::bind1st 和 std::bind2nd 适配器在绑定参数时使用这些 typedef 来确定参数和返回类型。

   1. binder1st

      binder1st(f2(), val) f1; 

      f2()为自适应二元函数， f1(x)等价于 f2(val, x)；

   2. bind1st

      auto f1= bind1st(f2(), val);

      简化 binder1st类的使用，对其提供用于构建binder1st对象的函数名称和值，它将返回一个这种类型的对象binder ；

   3. binder2nd

      与binder1st类似，只是将常数赋给第二个参数；

   4. bind2nd

      类似于 bind1st；

   ```c++
   #include <iostream>
   #include <vector>
   #include <iterator>
   #include <algorithm>
   #include <functional>
   
   using namespace std;
   
   void Show(double);
   const int LIM = 6;
   
   int main()
   {
   	double arr1[LIM] = {28, 29, 30, 35, 38, 59};
   	double arr2[LIM] = {63, 65, 69, 75, 80, 99};
   	vector<double> gr8(arr1, arr1 + LIM);
   	vector<double> m8(arr2, arr2 + LIM);
   	cout.setf(ios_base::fixed);
   	cout.precision(1);
   	cout << "gr8:\t";
   	for_each(gr8.begin(), gr8.end(), Show);
   	cout << endl;
   	cout << "m8:\t";
   	for_each(m8.begin(), m8.end(), Show);
   	cout << endl;
   
   	vector<double> sum(LIM);
   	transform(gr8.begin(), gr8.end(), m8.begin(), sum.begin(), plus<double>());
   	cout << "sum:\t";
   	for_each(sum.begin(), sum.end(), Show);
   	cout << endl;
   	
   	vector<double> prod(LIM);
   	transform(gr8.begin(), gr8.end(), prod.begin(), bind1st(multiplies<double>(), 2.5));
   	cout << "prod:\t";
   	for_each(prod.begin(), prod.end(), Show);
   	cout << endl;
   
       return 0;
   }
   
   void Show(double x)
   {
   	cout.width(6);
   	cout << x << ' ';
   }
   ```

### 算法

1. 算法组

   STL将算法库分成4组： 非修改式序列操作； 修改式序列操作； 排序和相关操作； 通用数字运算

2. 算法的通用特征

   对算法进行分类的方式之一是按结果放置的位置进行分类。有些算法就地完成工作，有些则创建拷贝。

   因此有些算法有两个版本：就地版本和复制版本。复制版本的名称将以_copy结尾。复制版本将接受一个额外的输出迭代器参数，该参数指定结果的放置位置。例如，函数replace( )的原型如下：

   template< class ForwardIterator, class T >

   void replace(ForwardIterator first, FordwardIterator last, const T& old_value, const T& new_value);

   它将所有的old_value替换为new_value，这是就地发生的。由于这 种算法同时读写容器元素，因此迭代器类型必须是ForwardIterator或更 高级别的。复制版本的原型如下：

   template< class ForwardIterator, class OutputIterator, class T >

   OutputIterator replace_copy(ForwardIterator first, FordwardOterator last, OutputIterator result, const T& old_value, const T& new_value);

   在这里，结果被复制到result指定的新位置，因此对于指定区间而 言，只读输入迭代器足够了对于复制算法， 统一的约定是：返回一个迭代器，该迭代器指向复制的最后一个值后面 的一个位置。

   另一个常见的变体是：有些函数有这样的版本，即根据将函数应用 于容器元素得到的结果来执行操作。这些版本的名称通常以_if结尾。例 如，如果将函数用于旧值时，返回的值为true，则replace_if( )将把旧值 替换为新的值。下面是该函数的原型：

   template< class ForwardIterator, class Predicate, class T >

   void replace_if(ForwardIterator first, FordwardOterator last, Predicate pred,  const T& new_value);

   STL选择用Predicate来提醒用户，实参应模拟Predicate概念。同 样，STL使用诸如Generator和BinaryPredicate等术语来指示必须模拟其 他函数对象概念的参数。

3.  STL和string类

   string类虽然不是STL的组成部分，但设计它时考虑到了STL。例 如，它包含begin( )、end( )、rbegin( )和rend( )等成员，因此可以使用 STL接口。

4. 函数和容器方法

   有时可以选择使用STL方法或STL函数。通常方法是更好的选择。 首先，它更适合于特定的容器；其次，作为成员函数，它可以使用模板类的内存管理工具，从而在需要时调整容器的长度。

   假设有一个由数字组成的链表，并要删除链表中某个特定值 （例如4）的所有实例。如果la是一个list对象：

   可以使用链表的 remove( )方法：

   la.remove(4); //值为4的元素都被删除，同时链表的 长度将被自动调整。

   还可以使用一个名为remove( )的STL算法：

   newend = remove(la.begin(), la.end(), 4); //值为4的元素都被删除，但不能调整链表的长度。返回一个指向新的超尾值的迭代器。

   earse(newend, la.end()); //删除一个区间来修改容器的长度。

5. 使用STL

   使用STL时应尽可能减少要编写的代码。STL通用、灵活的设计将节省大量工作。另外，STL设计者就是非常关心效率 的算法人员，算法是经过仔细选择的，并且是内联的。

### 其他库

1. vector

   vector模板类是一个容器类和算法系统的一部分，它支持面向容器的操作，如排序、 插入、重新排列、搜索、将数据转移到其他容器中等，支持自动调整大小。

2. valarray

   1. valarray类模 板是面向数值计算的，不是STL的一部分。例如，它没有push_back( )和 insert( )方法，但为很多数学运算提供了一个简单、直观的接口。

   2. valarray类有一个resize( )方法，但 不能像使用vector的push_back时那样自动调整大小。没有支持插入、排序、搜索等操作的方法。

   3. C++11提供了接受valarray对象作为参数的模板函 数begin()和end()，因此，您将使用begin(val)而不是val.begin。这些函数返回的值满足STL区间需求，例如：

      valarray< double > val(10);

      sort(begin(val), end(val))；

   4. slice类对 象可用作数组索引，在这种情况下，它表的不是一个值而是一组值。 slice对象被初始化为三个整数值：起始索引、索引数和跨距。起始索引 是第一个被选中的元素的索引，索引数指出要选择多少个元素，跨距表 示元素之间的间隔。例如，slice(1, 4, 3)创建的对象表示选择4个元素， 它们的索引分别是1、4、7和10。也就是说，从起始索引开始，加上跨 距得到下一个元素的索引，依此类推，直到选择了4个元素。如果varint 是一个valarray对象，则下面的语句将把第1、4、7、10个元素都设 置为10：

      varint[slice(1, 4, 3)] = 10;

      这种特殊的下标指定功能让您能够使用一个一维valarray对象来表 示二维数据。例如，假设要表示一个4行3列的数组，可以将信息存储在 一个包含12个元素的valarray对象中，然后使用一个slice(0, 3, 1)对象作 为下标，来表示元素0、1和2，即第1行。同样，下标slice(0, 4, 3)表示元 素0、3、6和9，即第一列。

3. array

   array是为替代内置数组而设计的，它通过提供更好、更安全的接口，让 数组更紧凑，效率更高。Array表示长度固定的数组，因此不支持 push_back( )和insert( )，但提供了多个STL方法，包括begin( )、end( )、 rbegin( )和rend( )，这使得很容易将STL算法用于array对象。

4. 模板initializer_list（C++11）

   可使用初始化列表语法将STL 容器初始化为一系列值：

   vector< double > vec {1.23, 51.2, 5.2, 36.7};

   这之所以可行，是因为容器类现在包含将initializer_list 作为参数的构造函数。例如，vector包含一个将 initializer_list作为参数的构造函数，因此上述声明与下面的代码等价：

   vector< double > vec ({1.23, 51.2, 5.2, 36.7});

   考虑到C++11新增的通用初始化语法，可使用表示法{}而不是()来调用类构造函数，即使没有接受initializer_list作为参数的构造函数；如果类有接受initializer_list作为参数的构造函数，则使用语法{}将调用该构造函数；

   ()初始化可以进行隐式的缩窄转换，但{}初始化禁止这种行为；

   ```c++
   #include <iostream>
   #include <initializer_list>
   
   double sum(std::initializer_list<double> il);
   double average(const std::initializer_list<double> & ril);
   
   using namespace std;
   
   int main()
   {
   	cout << "List 1 : sum = " << sum({2, 3, 4})
   		 << ", ave = " << average({2, 3, 4}) << "\n";
   	std::initializer_list<double> dl = {1.1, 2.2, 3.3, 4.4, 5.5};
   	cout << "List 2 : sum = " << sum(dl)
   		 << ", ave = " << average(dl) << "\n";
   	//dl = {3.2, 2.33, 5.2, 6.9, 11.3}; 不允许，无法直接将一个 std::initializer_list 赋值给另一个 std::initializer_list，因为它没有赋值运算符。
       std::initializer_list<double> dl2(dl); //用另一个initializer_list初始化: std::initializer_list<double> dl2(dl1); 使用拷贝初始化，通过另一个 initializer_list 对象来初始化新的 initializer_list 对象
   	cout << "List 3 : sum = " << sum(dl2)
   		 << ", ave = " << average(dl2) << "\n";
   
       return 0;
   }
   
   double sum(std::initializer_list<double> il)
   {
   	double tot = 0;
   	for(auto p = il.begin(); p != il.end(); p++)
   		tot += *p;
   	return tot;
   }
   
   double average(const std::initializer_list<double> & ril)
   {
   	double tot = sum(ril);
   	if(ril.size() > 0)
   		return tot / ril.size();
   	return 0;
   }
   ```

##  输入、输出和文件

###  C++输入和输出概述

### 输出方法

1. 重载的 << 运算符；

2.  其他ostream方法：

   1. put():

      输出字符；

      原型：ostream & put(char);

      调用：cout.put('w'); //返回cou引用，可链式调用；

      ​	   cout.put(65); //打印'A'；

   2. write()：

      输出字符串；

      原型：basic_ostream< charT, traits > & wrote(const char_type * s, streamsize n);

      调用：cout.write("li"，2); //返回cout引用，可链式调用；

      write( )方法并不会在遇到空字符时自动停止打印 字符，而只是打印指定数目的字符，即使超出了字符串的边界；

      write( )方法也可用于数值数据，可以将数字的地址强制转换为 char *，然后传递给它：

      long val = 560031841;
      cout.write((char *) &val, sizeof(long)); //打印"aha!",4字节的long值将作为4个独立的字节被传输。输出设备将把每个字节作为ASCII码进行解释。因此在屏幕上，560031841将被显示为4个字符的组合

3. 刷新输出缓冲区

   1. 多数C++实现都会在输入即将发生时刷新缓冲区；

   2. 控制符flush刷新缓冲区：

      可以直接调用flush( )来刷新cout缓冲区：flush(cout);

      重载<<运算符使用：cout << flush;

   3. 控制符endl刷新缓冲区；

4. 用cout进行格式化:

   1. 要控制整数以十进制、 十六进制还是八进制显示，可以使用dec、hex和oct控制符;

      调用：hex(cout);

      重载<<运算符使用：cout << hex;

   2. 调整字段宽度：

      下一次输出时，值在字段中右对齐，并用空格来填满整个字段，新的宽度设置只在随后的第一次输出中有效；

      原型：int width(int i); //将字段宽度设置 为i个空格，并返回以前的字段宽度值

      原型：int width(); //返回字段宽度的当前设置

      调用：cout.width(10);

   3. 填充字符：

      与字段宽度不同的是，新的填充字符将一直有效，直到更改它为止。

      cout.fill('*');

   4. 设置浮点数的显示精度：

      C++的默认精度为6位，但末尾的0将不显示。precision()成员函数使得能够选择其他值。新的精度设置将一直有效，直到被重新设置。

   5. seft():

      ios_base类有一个受保护的数据成员，其中的各位（这里叫作标记）分别控制着格式化的各个方面，例如计数系统、是否显示末尾的0等。打开一个标记称为设置标记（或位），并意味着相应的位被设置为1。例如，hex、dec和oct控制符调整控制计数系统的3个标记位。setf( )函数提供了另一种调整标记位的途径。

      1. 设置单个位控制的格式信息：fmtflags setf(fmtflags); 

         fmtflags是bitmask类型的typedef名，bitmask类型是一种用来存储各个位值的类型。它可以是整型、枚举，也可以是STL bitset容器。这里的主要思想是，每一位都是可以单独访问的，都有自己的含义。iostream软件包使用 bitmask来存储状态信息。

         返回值是类型为fmtflags的数字，指出所有标记以前的设置。

         1. ios_base ::boolalpha 输入和输出bool值，可以为true或false；
         2. ios_base ::showbase 对于输出，使用C++基数前缀（0，0x）；
         3. ios_base ::showpoint 显示末尾的小数点；
         4. ios_base ::uppercase 对于16进制输出，使用大写字母，科学计数法将使用E表示法；
         5. ios_base ::showpos 在正数前面加上+；

      2. 设置由多位控制的格式选项：fmtflags setf(fmtflags， fmtflags);

         第二参数指出要清除哪些位，用第一参数指出要设置哪些位，返回值是类型为fmtflags的数字，指出所有标记以前的设置；

   6. unsetf( )：

      void unself(fmtflags mask);

      其中，mask是位模式。mask中所有的位都设置为1，将使得对应的 位被复位。也就是说，setf( )将位设置为1，unsetf( )将位恢复为0。例如：

      cout.setf(ios:_base::showpoint); 

      cout.unsetf(ios_base::showpoint);

      启用默认模式的两种方法：

      cout.setf(0, ios_base::floatfield); //default mode

      cout.unsetf(ios::base::floatfield); //default mode

   7. 标准控制符：

      C++ 标准库提供了多种控制符来设置格式标志，它们调用 setf()并自动提供正确的参数。常见的控制符包括 dec、hex、oct、left、right、fixed、scientific 等。这些控制符的工作方式都与前面提到的hex相似。他们被定义在头文件iomanip中；

   8. 3个最常用的控制符对应于iostream提供的三种用来设置精度、填充字符和字段宽度的方法，分别是 setprecision( )、setfill( )和setw( )，不同于标准控制符的是，他们接受参数。

###  使用cin进行输入

istream类（在iostream头文件中定 义）重载了抽取运算符>>，使之能够识别基本类型，运算符函数的原型如下：istream & operator>>(int &); //返回调用对象的引用，这使得能够将输入拼接起来，就像拼接输出那样；

1. cin>>如何检查输入

   在单字符模式下，>>运算符将读取字符，将它放置到指定的位置。在其他模式 下，>>运算符将读取一个指定类型的数据。也就是说，它读取从非空白字符开始，到与目标类型不匹配的第一个字符之间的全部内容，并且连续输入空格间隔开的数据时，会跳过空白字符。

2.  流状态

   cin或cout对象包 含一个描述流状态（stream state）的数据成员（从ios_base类那里继承 的）。流状态（被定义为iostate类型，而iostate是一种bitmask类型）由3个ios_base元素组成：eofbit、badbit或failbit，其中每个元素都是一位，可以是1（设置）或0（清除）。当cin操作到达文件末尾时，它将设置 eofbit；当cin操作未能读取到预期的字符时（像前一个例子那样），它 将设置failbit。I/O失败（如试图读取不可访问的文件或试图写入写保护的磁盘），也可能将failbit设置为1。在一些无法诊断的失败破坏流时， badbit元素将被设置（实现没有必要就哪些情况下设置failbit，哪些情况 下设置badbit达成一致）。当全部3个状态位都设置为0时，说明一切顺利。

   1. 设置状态：

      1. clear(iostate state = goodbit)：将流的状态设置为state。如果没有提供参数，默认将状态设置为goodbit（表示没有错误）；
      2. setstate(iostate state)：将流的当前状态与state合并，这意味着流的现有状态将保留，并与新状态合并。例如，如果流当前处于 failbit状态，再设置 eofbit，则流将同时具有 failbit和 eofbit 状态。；

   2. I/O和异常：

      exceptions( )方法返回一个位字段，它包含3位，分别对应于eofbit、failbit和badbit，ios_base::failure异常类是从 std::exception类派生而来的，因此包含一个what( )方法，打印错误信息。

      1. 设置异常掩码：通过调用exceptions()方法设置流对象的异常掩码，指定哪些错误状态（如failbit、badbit）应导致抛出异常；
      2. 输入/输出操作：进行输入/输出操作，如读取整数。如果输入不符合预期类型（如用户输入了非整数），该操作会导致流对象的错误状态位被设置（如failbit）；
      3. 检测错误状态：在输入/输出操作失败后，流对象会根据其异常掩码检测当前的错误状态。如果错误状态与异常掩码匹配，则抛出ios_base::failure异常。

      ```c++
      #include <iostream>
      #include <exception>
      
      using namespace std;
      
      int main()
      {
      	cin.exceptions(ios_base::failbit);
      	cout << "enter numbers:" << endl;
      	int sum = 0;
      	int input;
      	try{
      		while(cin >> input)
      		{
      			sum += input;
      		}
      	}catch(ios_base::failure & bf)
      	{
      		cout << bf.what() << endl;
      		cout << "O!the horror!\n";
      	}
      
      	cout << "last value entered = " << input << endl;
      	cout << "sum = " << sum << endl;
      
          return 0;
      }
      ```

3. 其他istream方法：

   1. read( )：

      read( )不会在输入后加上空值字符， 因此不能将输入转换为字符串。read( )方法不是专为键盘输入设计的， 它最常与ostream write( )函数结合使用，来完成文件输入和输出。该方法的返回类型为istream &;

   2. peek( )：

      返回输入中的下一个字符，但不抽取输入流中的字符。 也就是说，它使得能够查看下一个字符；

   3. gcount( )：

      返回最后一个非格式化抽取方法读取的字符数。这意 味着字符是由get( )、getline( )、ignore( )或read( )方法读取的，不是由抽 取运算符（>>）读取的，抽取运算符对输入进行格式化，使之与特定的数据类型匹配；

   4. putback( )：

      将一个字符插入到输入字符串中，被插入的字符将是下一条输入语句读取的第一个字符。putback( )方法接受一个char参数 ——要插入的字符，其返回类型为istream &。并且它允许将字符放到不是刚才读取的位置；

   5. eatline( ):

      通常用于从输入缓冲区中清除剩余的字符，尤其是在处理输入操作时可能产生的换行符和其他多余的字符，等价于下述的代码：

      while(cin.get() != '\n') { continue; }

###  文件输入和输出

1. 简单的文件I/O:

   ```c++
   #include <iostream>
   #include <fstream>
   #include <string>
   
   using namespace std;
   
   int main()
   {
   	string filename;
   	cout << "enter name for new file: ";
   	cin >> filename;
   
   	ofstream fout(filename);
   	//ofstream fout;
   	//fout.open(filename);
   	fout << "for your eyes only!\n";
   	cout << "enter your secret number: ";
   	float secret;
   	cin >> secret;
   	fout << "your secret number is " << secret << endl;
   	fout.close();
   
   	ifstream fin(filename);
   	cout << "here are the contents of" << filename << ":\n";
   	char ch;
   	while(fin.get(ch))
   		cout << ch;
   	cout << "Done!\n";
   	fin.close();
   
       return 0;
   }
   ```

2. is_open( )：

   if(fin.fail( )) ... //failed to open

   if(!fin.good( )) ... //failed to open

   if(!fin) ... //failed to open

   上面三种方式等价，而is_open( )能够检测出以上方式不能检测出的微妙问题，如试图以不合适的文件模式打开文件时失败；

3. 命令行处理技术：

   存在命令行环境中运行的程序能够访问命令行参数的机制，方法是使用下面的main( )函数：

   int main(int argc, char *argv[])

   argc为命令行中的参数个数，其中包括命令名本身。argv变量为一个指针，它指向一个指向char的指针。即argv为指针数组，每个数组成员为每个命令行参数的字符串地址；

4. 文件模式：

   ios_base类定义了一个openmode类型，用于表示模式；与fmtflags和iostate类型一样，它也是一种bitmask类型（以前，其类型为int）。可以选择ios_base类中定义的多个常量来指定模式。ifstream open( )方法和构造函数用ios_base::in（打开文件以读取）作为模式参数的默认值，而ofstream open( )方法和构造函数用ios_base::out | ios_base::trunc（打开文件，以写入并截短文件）作为默认值。位运算符OR（|）用于将两个位值合并成一个可用于设置两个位的值。fstream类不提供默认的模式值，因此在创建这种类的对象时，必须显式地提供模式。

   1. 追加文件：

      ofstream fout(file, ios_base::out | ios_base::app);

   2. 二进制文件:

      ofstream fout(file, ios_base::out | ios_base::app | ios_base::binary);

      fout.write((char *) &pr, sizeof pr); //写入

      ifstream fin(file, ios_base::in | ios_base::binary);

      fin.read((char *) &pr, sizeof pr); //读取

      1. 同样的方法也适用于不使用虚函数的类。在这种情况下，只有数据成员被保存，而方法不会被保存。如果类有虚方法，则也将复制隐藏指针（该指针指向虚函数的指针表）。由于下一次运行程序时，虚函数表可能在不同的位置，因此将文件中的旧指针信息复制到对象中，将可能造成混乱。
      2. 将数据存储在文件中时，可以将其存储为文本格式或二进制格式。对于字符来说，二进制表示与文本表示是一样的，即字符的ASCII码的二进制表示。对于数字来说，二进制表示与文本表示有很大的差别。
      3. 文本格式便于读取，可以使用编辑器或字处理器来读取和编辑文本文件，可以很方便地将文本文件从一个计算机系统传输到另一个计算机系统。二进制格式对于数字来说比较精确，因为它存储的是值的内部表示，因此不会有转换误差或舍入误差。以二进制格式保存数据的速度更快，因为不需要转换，并可以大块地存储数据。二进制格式通常占用的空间较小，这取决于数据的特征。然而，如果另一个系统使用另一种内部表示，则可能无法将数据传输给该系统。同一系统上不同的编译器也可能使用不同的内部结构布局表示。在这种情况下，则必须编写一个将一种数据转换成另一种的程序。

   3.  随机存取

      1. fstream类继承了来自 std::basic_istream 和 std::basic_ostream 的两个方法：seekg( )和seekp( )，前者将输入指针移到指定的文件位置，后者将输出指针移到指定的文件位置（实际上，由于fstream类使用缓冲区来存储中间数据，因此指针指向的是缓冲区中的位置，而不是实际的文件）。也可以将seekg( )用于ifstream对象，将seekp( )用于oftream对象。

         seekg( )的char类型的模板具体化1：

         istream & seekg(streamoff, ios_base::seekdir);

         streamoff值被用来度量相对于文件特定位置的偏移量（单位为字节）。streamoff参数表示相对于三个位置之一的偏移量（以字节为单位）。seek_dir参数是ios_base类中定义的另一种整型，有3个可能的值。常量ios_base::beg指相对于文件开始处的偏移量。常量ios_base::cur指相对于当前位置的偏移量；常量ios_base::end指相对于文件尾的偏移量。

         seekg( )的char类型的模板具体化2：

         istream & seekg(streampos);

         streampos类型的值定位到文件中的一个位置。它可以是类，这个类将包含一个接受streamoff参数的构造函数和一个接受整数参数的构造函数，以便将两种类型转换为streampos值。streampos值表示文件中的绝对位置（从文件开始处算起）。

      2. 如果要检查文件指针的当前位置，则对于输入流，可以使用tellg( )方法，对于输出流，可以使用tellp( )方法。它们都返回一个表示当前位置的streampos值（以字节为单位，从文件开始处算起）。创建fstream对象时，输入指针和输出指针将一前一后地移动，因此tellg( )和tellp( )返回的值相同。

      3. char * tmpnam(char * pszName);

         tmpnam( )函数创建一个临时文件名，将它放在pszName指向的C-风格字符串中。常量L_tmpnam和TMP_MAX（二者都是在cstdio中定义的）限制了文件名包含的字符数以及在确保当前目录中不生成重复文件名的情况下tmpnam( )可被调用的最多次数。

5. 内核格式化：

   1. 读取string对象中的格式化信息或将格式化信息写入string对象中被称为内核格式化；

   2. 头文件sstream定义了一个从ostream类派生而来的ostringstream类，可以将可用于cout的方法用于ostringstream对象；

      ```c++
      #include <iostream>
      #include <iomanip> // For std::fixed and std::setprecision
      #include <sstream> // For std::ostringstream
      #include <string>  // For std::string
      
      int main() {
          // 创建一个字符串流对象
          std::ostringstream outstr;
      
          // 定义价格和附加信息
          double price = 380.0;
          const char * ps = " for a copy of the IOS/ETC C++ standard!";
      
          // 设置浮点数精度为2位小数
          outstr.precision(2);
          
          // 设置浮点数以定点形式输出
          outstr << std::fixed;
          
          // 将信息写入字符串流
          outstr << "Pay only CHF " << price << ps << std::endl;
      
          // 使用str()方法获取字符串内容
          std::string mesg = outstr.str();  // str()方法可以“冻结”该对象，便不能将信息写入该对象中。
          
          // 输出字符串内容
          std::cout << mesg;
      
          return 0;
      }
      
      ```
   
   3. istringstream类允许使用istream方法族读取istringstream对象中的数据，istringstream对象可以使用string对象进行初始化；
   
      ```c++
      #include <iostream>
      #include <sstream>
      #include <string>
      
      using namespace std;
      
      int main() {
          string facts = "10 20 30 40 50"; // 字符串，其中包含要读取的数字
          istringstream instr(facts); // 使用字符串初始化输入流对象
      
          int n; // 用于存储从流中读取的整数
          int sum = 0; // 用于存储数字的和
      
          // 从输入流中读取整数，并将它们相加
          while (instr >> n) {
              sum += n; // 将读取的整数加到 sum 中
          }
      
          cout << "Sum: " << sum << endl; // 输出结果
      
          return 0;
      }
      
      ```

##  探讨C++新标准

### 复习前面介绍过的C++11功能

1. 统一的初始化（列表初始化）：

   1. 初始化列表语法可防止缩窄，即禁止将数值赋给无法存储它的数值变量，但允许转换为更宽的类型。另外，只要值在较窄类型的取值范围内，将其转换为较窄的类型也是允许的；

   2. std::initializer_list

      如果类有接受initializer_list作为参数的构造函数，则初始化列表语法就只能用于该构造函数。列表中的元素必须是同一种类型或可转换为同一种类型。模板类initializer_list包含成员函数begin( )和end( )。还可将initializer_list用作常规函数的参数；

2. 声明：

   1. auto
   2. decltype
   3. 返回类型后置
   4. 模板别名：using =
   5. nullptr

3. 智能指针：unique_ptr、shared_ptr和weak_ptr

4. 异常规范方面的修改：

   以前，C++提供了一种语法，可用于指出函数可能引发哪些异常：

   void f501(int) throw(bad_dog);

   void f733(long long) throw();

   C++编程社区的集体经验表明，异常规范的效果没有预期的好。因此，C++11摒弃了异常规范。然而，标准委员会认为，指出函数不会引发异常有一定的价值，他们为此添加了关键字
   noexcept：

   void f875(short, short) noexcept;

5.  作用域内枚举:

   C++11新增了一种枚举。这种枚举使用class或struct定义：

   enum class New1 {never, sometimes, often, always};

   enum struct New2 {never, lever, sever};

   新枚举要求进行显式限定，以免发生名称冲突。因此，引用特定枚举时，需要使用New1::never和New2::never等。

6.  对类的修改:

   1. 显式转换运算符:

      C++引入了关键字explicit，以禁止单参数构造函数导致的自动转换：

   2. 类内成员初始化:

      可使用等号或大括号版本的初始化，但不能使用圆括号版本的初始化。

      class Session

      {

      ​	int mem1 = 10;

      ​	double mem2 {1966.54};

      ​	short mem3;

      ​	public:

      ​		Session() {}

      ​		Session(short s) : mem3(s) {}

      ​		Session(int n, double d, short s) : mem1(n), mem2(d), mem3(s) {}

      ​		...

      };

      其结果与给前两个构造函数提供成员初始化列表，并指定mem1和mem2的值相同。

7. 模板和STL方面的修改:

   1. 基于范围的for循环：针对于内置数组以及包含方法begin( ) 和end( ) 的类（如std::string）和STL容器:

      for(auto x : arr) { ... }

      如果要在循环中修改数组或容器的每个元素，可使用引用类型：

      for(auto & x : arr) { ... }

   2. 新的STL容器

      C++11新增了STL容器forward_list、unordered_map、unordered_multimap、unordered_set和unordered_multiset;

   3. 新的STL方法

      C++11新增了STL方法cbegin( )和cend( )。与begin( )和end( )一样，这些新方法也返回一个迭代器，指向容器的第一个元素和最后一个元素的后面，因此可用于指定包含全部元素的区间。另外，这些新方法将元素视为const。与此类似，crbegin( )和crend( )是rbegin( )和rend( )的const版本;

   4. valarray升级

      模板valarray独立于STL开发的，其最初的设计导致无法将基于范围的STL算法用于valarray对象。C++11添加了两个函数（begin( )和end()），它们都接受valarray作为参数，并返回迭代器，这些迭代器分别指向valarray对象的第一个元素和最后一个元素后面。这让您能够将基于范围的STL算法用于valarray;

   5. 摒弃export

      C++98新增了关键字export，旨在提供一种途径，让程序员能够将模板定义放在接口文件和实现文件中，其中前者包含原型和模板声明，而后者包含模板函数和方法的定义。实践证明这不现实，因此C++11终止了这种用法，但仍保留了关键字export，供以后使用;

   6. 尖括号

      为避免与运算符>>混淆，C++要求在声明嵌套模板时使用空格将尖括号分开：

      std::vector<std::list< int > > vl;

      C++11不再这样要求：:

      std::vector<std::list< int >> vl;

8.  右值引用

   右值是指不具名的临时对象，其生命周期短暂，左值是指可被定位的对象，具有持久的存储；

   左值引用（&）：只能绑定到左值，可以通过引用修改原对象。

   常量左值引用（const &）：可以绑定到左值或右值，不能修改原对象。

   右值引用（&&）：只能绑定到右值，允许修改绑定的右值，主要用于实现移动语义。

###  移动语义和右值引用

1. 复制构造函数：

   1. 执行深复制；

   2. 如果没有移动构造函数，且编译器未能通过优化（优化指的是编译器可能自动消除额外的复制工作）消除对复制构造函数的需求，下面的语句将调用复制构造函数：

      Useless four(one + three); 

      即形参被初始化为一个临时对象，而临时对象初始化为one + three的值；

2. 移动构造函数：

   1. 获取这些数据的所有权（新对象指向数据，并把原来的对象指向的数据设置为空，即不 能让多个指针指向相同的数据）；
   2. 使用右值引用与编译器优化不同的是程序员可指出何时该使用移动语义；
   3. 使用前提：编写移动构造函数，参数为右值引用，不能是const引用，实参为右值；

   ```c++
   #include <iostream>
   
   using namespace std;
   
   class Useless
   {
   	private:
   		int n;
   		char * pc;
   		static int ct;
   		void ShowObject() const;
   	public:
   		Useless();
   		explicit Useless(int k);
   		Useless(int k, char ch);
   		Useless(const Useless & f);
   		Useless(Useless && f);
   		~Useless();
   		Useless operator+(const Useless & f) const;
   		void ShowData() const;
   };
   
   int Useless::ct = 0;
   
   Useless::Useless()
   {
   	++ct;
   	n = 0;
   	pc = nullptr;
   	cout << "default constructor called; number of object: " << ct << endl;
   	ShowObject();
   }
   
   Useless::Useless(int k) : n(k)
   {
   	++ct;
   	cout << "int constructor called; number of object: " << ct << endl;
   	pc = new char[n];
   	ShowObject();
   }
   
   Useless::Useless(int k, char ch) : n(k)
   {
   	++ct;
   	cout << "int. char construtor called; number of objects: " << ct << endl;
   	pc = new char[n];
   	for(int i = 0; i < n; i++)
   		pc[i] = ch;
   	ShowObject();
   }
   
   Useless::Useless(const Useless & f) : n(f.n)
   {
   	++ct;
   	cout << "copy const called; number of objects: " << ct << endl;
   	pc = new char[n];
   	for(int i = 0; i < n; i++)
   		pc[i] = f.pc[i];
   	ShowObject();
   }
   
   Useless::Useless(Useless && f) : n(f.n)
   {
   	++ct;
   	cout << "move constructor called; number of objects: " << ct << endl;
   	pc = f.pc;
   	f.pc = nullptr;
   	f.n = 0;
   	ShowObject();
   }
   
   Useless::~Useless()
   {
   	cout << "destructor called; objects left: " << --ct << endl;
   	cout << "deleted object:\n";
   	ShowObject();
   	delete [] pc;
   }
   
   Useless Useless::operator+(const Useless & f) const
   {
   	cout << "Entering operator+()\n";
   	Useless temp = Useless(n + f.n);
   	for(int i = 0; i < n; i++)
   		temp.pc[i] = pc[i];
   	for(int i = n; i < temp.n; i++)
   		temp.pc[i] = f.pc[i - n];
   	cout << "temp object:\n";
   	cout << "leaving operator+()\n";
   	return temp;
   }
   
   void Useless::ShowObject() const
   {
   	cout << "Number of elements: " << n;
   	cout << " Data address: " << (void *) pc << endl; 
   }
   
   void Useless::ShowData() const
   {
   	if(n == 0)
   		cout << "(object empty)";
   	else
   		for(int i = 0; i < n; i++)
   			cout << pc[i];
   	cout << endl;
   }
   
   int main()
   {
   	Useless one(10, 'x');
   	Useless two = one;
   	Useless three(20, 'o');
   	Useless four(one + three);
   	cout << "object one :";
   	one.ShowData();
   	cout << "object two :";
   	two.ShowData();
   	cout << "object three :";
   	three.ShowData();
   	cout << "object four :";
   	four.ShowData();
       return 0;
   }
   ```

3. 赋值运算符:

   Useless & Useless::operator=(const Useless & f)
   {
   	if(this == &f)

   ​		return *this;

   ​	delete [] pc;

   ​	n = f.n;

   ​	pc = new char[n];

   ​	for(int i =0; i < n; i++)

   ​		pc[i] = f.pc[i];

   ​	return *this;
   }

4. 移动赋值运算符：

   1. 移动赋值运算符删除目标对象中的原始数据，并将源对象的所有权转让给目标，不能让多个指针指向相同的数据；
   2. 使用前提：编写移动赋值运算符，参数为右值引用，不能是const引用，实参为右值；

    Useless & Useless::operator=(Useless && f)
   {
   	if(this == &f)

   ​		return *this;

   ​	delete [] pc;

   ​	n = f.n;

   ​	pc = f.pc;

   ​	f.pc = nullptr;

   ​	f.n = 0;

   ​	return *this;
   }

5. 运算符static_cast<>：

   1. 将对象的类型通过运算符强制转换为右值类型后，即可使用移动操作；

   2. C++11对左值使用头文件utility中声明的函数std::move( )后，即可使用移动操作；

      对于下面的代码如果没有定义移动赋值运算符，编译器将使用复制赋值运算符。如果也没有定义复制赋值运算符，将根本不允许赋值；

      two = std::move(one);

   ```c++
   #include <iostream>
   #include <utility>
   
   using namespace std;
   
   class Useless
   {
   	private:
   		int n;
   		char * pc;
   		static int ct;
   		void ShowObject() const;
   	public:
   		Useless();
   		explicit Useless(int k);
   		Useless(int k, char ch);
   		Useless(const Useless & f);
   		Useless(Useless && f);
   		~Useless();
   		Useless operator+(const Useless & f) const;
   		Useless & operator=(const Useless & f);
   		Useless & operator=(Useless && f);
   		void ShowData() const;
   };
   
   int Useless::ct = 0;
   
   Useless::Useless()
   {
   	++ct;
   	n = 0;
   	pc = nullptr;
   	cout << "default constructor called; number of object: " << ct << endl;
   	ShowObject();
   }
   
   Useless::Useless(int k) : n(k)
   {
   	++ct;
   	cout << "int constructor called; number of object: " << ct << endl;
   	pc = new char[n];
   	ShowObject();
   }
   
   Useless::Useless(int k, char ch) : n(k)
   {
   	++ct;
   	cout << "int. char construtor called; number of objects: " << ct << endl;
   	pc = new char[n];
   	for(int i = 0; i < n; i++)
   		pc[i] = ch;
   	ShowObject();
   }
   
   Useless::Useless(const Useless & f) : n(f.n)
   {
   	++ct;
   	pc = new char[n];
   	for(int i = 0; i < n; i++)
   		pc[i] = f.pc[i];
   }
   
   Useless::Useless(Useless && f) : n(f.n)
   {
   	++ct;
   	pc = f.pc;
   	f.pc = nullptr;
   	f.n = 0;
   }
   
   Useless::~Useless()
   {
   	delete [] pc;
   }
   
   Useless Useless::operator+(const Useless & f) const
   {
   	Useless temp = Useless(n + f.n);
   	for(int i = 0; i < n; i++)
   		temp.pc[i] = pc[i];
   	for(int i = n; i < temp.n; i++)
   		temp.pc[i] = f.pc[i - n];
   	return temp;
   }
   
   Useless & Useless::operator=(const Useless & f)
   {
   	cout << "copy assignment operator called:\n";
   	if(this == &f)
   		return *this;
   	delete [] pc;
   	n = f.n;
   	pc = new char[n];
   	for(int i =0; i < n; i++)
   		pc[i] = f.pc[i];
   	return *this;
   }
   
   Useless & Useless::operator=(Useless && f)
   {
   	cout << "move assignment operator called:\n";
   	if(this == &f)
   		return *this;
   	delete [] pc;
   	n = f.n;
   	pc = f.pc;
   	f.pc = nullptr;
   	f.n = 0;
   	return *this;
   }
   
   void Useless::ShowObject() const
   {
   	cout << "Number of elements: " << n;
   	cout << " Data address: " << (void *) pc << endl; 
   }
   
   void Useless::ShowData() const
   {
   	if(n == 0)
   		cout << "(object empty)";
   	else
   		for(int i = 0; i < n; i++)
   			cout << pc[i];
   	cout << endl;
   }
   
   int main()
   {
   	Useless one(10, 'x');
   	Useless two = one + one;
   	cout << "object one: ";
   	one.ShowData();
   	cout << "object two: ";
   	two.ShowData();
   	Useless three;
   	cout << "three = one :\n";
   	three = one;
   	cout << "object three: ";
   	three.ShowData();
   	Useless four;
   	cout << " four = one + two\n";
   	four = one + two;
   	cout << "object four: ";
   	four.ShowData();
   	cout << "four = move(one)\n";
   	four = move(one);
   	cout << "new four: ";
   	four.ShowData();
   	cout << "object one: ";
   	one.ShowData();
       return 0;
   }
   ```

### 新的类功能

1. 默认的方法和禁用的方法：

   1. 假定要使用某个默认的函数，而这个函数由于某种原因不会自动创建，可使用关键字default显式地声明这些方法的默认版本：

      ClassName(const ClassName &) = default;

      ClassName() = default; //使用编译器生成的默认构造函数

   2. 关键字delete可用于禁止编译器使用特定方法，例如要禁止复制对象，可禁用复制构造函数和复制赋值运算符：

      ClassName(const ClassName &) = delete;

      ClassName & operator=(const ClassName &) = delete;

      在启用移动方法的同时如果禁用复制方法：

      ClassName three(one); //not allowed;

      ClassName four(one + two); //allowed

   3. default只能用于6个特殊成员函数，但delete可用于任何成员函数。delete的一种可能用法是禁止特定的转换：

      void redo(double);

      调用ob.redo(5)，int值5将被提升为5.0，进而执行redo();

      void redo(int) = delete;

      调用sc.redo(5)与原型redo(int) 匹配。编译器检测到这一点以及redo(int) 被禁用后，将这种调用视为编译错误。

   4. 禁用函数的重要一点：它们只用于查找匹配函数，使用禁用函数将导致编译错误。

2. 委托构造函数：

   C++11允许您在一个构造函数的定义中使用另一个构造函数。这被称为委托，因为构造函数暂时将创建对象的工作委托给另一个构造函数。委托使用成员初始化列表语法的变种：

   Class ClassName

   {

   ​	int k;

   ​	double x;

   ​	string st;

   public:

   ​	ClassName();

   ​	ClassName(int);

   ​	ClassName(int, double);

   ​	ClassName(int, double, string);

   }

   ClassName::Classname(int kk, double xx, string stt) : k(kk), x(xx), st(stt) {...}

   ClassName::Classname() : ClassName(0, 0.01, "Oh") {...}

   ClassName::Classname(int kk) : ClassName(kk, 0.01, "Ah") {...}

   ClassName::Classname(int kk, double xx) : ClassName(kk, xx, "Uh") {...}

3. 继承构造函数

   让派生类继承基类的所有构造函数（默认构造函数、复制构造函数和移动构造函数除外），但不会使用与派生类构造函数的特征标匹配的构造函数：

   ```c++
   #include <iostream>
   
   using namespace std;
   
   class BS
   {
   	private:
   		int q;
   		double w;
   	public:
   		BS() : q(0), w(0) {}
   		BS(int k) : q(k), w(100) {}
   		BS(double x) : q(-1), w(x) {}
   		BS(int k, double x) : q(k), w(x) {}
   		void Show() const {cout << q << ", " << w << '\n';} 
   };
   
   class DR : public BS
   {
   	short j;
   	public:
   		using BS::BS;
   		DR() : j(-100) {}
   		DR(double x) : BS(2 * x), j(int(x)) {}
   		DR(int i) : j(-2), BS(i, 0.5 * i) {}
   		void Show() const {cout << j << ", "; BS::Show();}
   };
   
   int main()
   {
   	DR o1;
   	DR o2(18.81);
   	DR o3(10, 1.8);
       o1.Show();
   	o2.Show();
   	o3.Show();
   
       return 0;
   }
   ```

   由于没有构造函数DR(int, double)，因此创建DR对象o3时，将使用 继承而来的BS(int, double)。请注意，继承的基类构造函数只初始化基类 成员；如果还要初始化派生类成员，则应使用成员列表初始化语法：

   DR(int i, int k, double x) : j(i), BS(k, x) {}

4. 管理虚方法：override和final

   class Action

   {

   ​	int a;

   ​	public:

   ​		Action(int i = 0) : a(i) {}

   ​		int val() const {return a;};

   ​		virtual void f(char ch) const { cout  << val() << ch << "\n"; }

   };

   class Bingo : public Action

   {

   ​	public:

   ​		Bingo(int i = 0) : Action(i) {}

   ​		virtual void f(char * ch) const { cout << val() << ch << "!\n"; }

   };

   1. 用虚说明符override指出您要覆盖一个虚函数：将其放在参数列表后面。如果声明与基类方法不匹配，编译器将视为错误。因此，下面的Bingo::f( )版本将生成一条编译错误消息：

      virtual void f(char * ch) const override { cout << val() << ch << "!\n"; }

   2. 说明符final禁止派生类覆盖特定的虚方法，为此可在参数列表后面加上final。例如，下面的代码禁止Action 的派生类重新定义函数f( )：

      virtual void f(char ch) const final  { cout << val() << ch << "\n"; }

### Lambda函数

1. 比较函数指针、函数符和Lambda函数

   这三种方法的相对效率取决于编译器内联那些东西。函数指针方法 阻止了内联，因为编译器传统上不会内联其地址被获取的函数，因为函 数地址的概念意味着非内联函数。而函数符和lambda通常不会阻止内联

   ```c++
   #include <iostream>
   #include <vector>
   #include <algorithm>
   #include <cmath>
   #include <ctime>
   
   const long Size1 = 39L;
   const long Size2 = 100 * Size1;
   const long Size3 = 100 * Size2;
   
   bool f3(int x) { return x % 3 == 0; }
   bool f13(int x) { return x % 13 == 0; }
   
   using namespace std;
   
   int main()
   {
   	vector<int> numbers(Size1);
   	srand(time(0));
   	generate(numbers.begin(), numbers.end(), rand);
   	cout << "Sample size = " << Size1 << endl;
   
   	//使用函数指针
   	int count3 = count_if(numbers.begin(), numbers.end(), f3);
   	cout << "Count of numbers divisible by 3: " << count3 << endl;
   	int count13 = count_if(numbers.begin(), numbers.end(), f13);
   	cout << "Count of numbers divisible by 13: " << count13 << endl;
   
   	numbers.resize(Size2);
   	generate(numbers.begin(), numbers.end(), rand);
   	cout << "Sample size = " << Size2 << endl;
   
   	//使用函数符
   	class f_mod
   	{
   		private:
   			int dv;
   		public:
   			f_mod(int d = 1) : dv(d) {}
   			bool operator()(int x) { return x % dv == 0; }
   	};
   	count3 = count_if(numbers.begin(), numbers.end(), f_mod(3));
   	cout << "Count of numbers divisible by 3: " << count3 << endl;
   	count13 = count_if(numbers.begin(), numbers.end(), f_mod(13));
   	cout << "Count of numbers divisible by 13: " << count13 << endl;
   
   	numbers.resize(Size3);
   	generate(numbers.begin(), numbers.end(), rand);
   	cout << "Sample size = " << Size3 << endl;
   
   	//使用lambdas
   	count3 = count_if(numbers.begin(), numbers.end(), [](int x) { return x % 3 == 0;});
   	cout << "Count of numbers divisble by 3: " << count3 << endl;
   	count3 = count_if(numbers.begin(), numbers.end(), [](int x) { return x % 13 == 0;});
   	cout << "Count of numbers divisble by 13: " << count3 << endl;
   
       return 0;
   }
   ```

2. 为何使用lambda

3. lambda函数注意事项：

   1. lambda表达式完全由一条返回语句组成时，返回类型相当于使用decltyp根据返回值推断得到的，因此无需声明返回类型；

      [] (int x) { return x % 3 == 0; }

      多条语句组成时，需要使用新增的返回类型后置语法；

      [] (double x) -> double{ int y = x; return x - y; }

   2. 可给lambda指定一个名称：

      auto mod3 = [] (int x) { return x % 3 == 0; }

      bool result = mod3(z);

   3. lambad可访问作用域内的任何动态变量；要捕获要使用的变量，可将其名称放在中括号内。 如果只指定了变量名，如[z]，将按值访问变量；如果在名称前加上&， 如[&count]，将按引用访问变量。[&]让您能够按引用访问所有动态变量，而[=]让您能够按值访问所有动态变量。还可混合使用这两种方式，例如，[ted, &ed]让您能够按值访问ted以及按引用访问ed，[&, ted] 让您能够按值访问ted以及按引用访问其他所有动态变量，[=, &ed]让您 能够按引用访问ed以及按值访问其他所有动态变量。

### 包装器

1. 包装器function及模板的低效性

   下面的代码输出有5个不同的地址，这表明模板use_f( )有5个不同的实例化。其中的dub是一个函数的名称，该函数接受一个double参数并返回一个double值。函数名是指针，因此参数F的类型为double( * ) (double)： 一个指向这样的函数的指针，即它接受一个double参数并返回一个 double值。第二个参数的类型也是double(*) (double)，因此该调用使用的use_f( )实例化与第一个调用相同。 在接下来的两个use_f( )调用中，第二个参数为对象，F的类型分别为Fp和Fq，因为将为这些F值实例化use_f( )模板两次。最后两个调用将F的类型设置为编译器为lambda表达式使用的类型(同样不同)；

   ```c++
   //somedefs.h
   
   #include <iostream>
   
   template <typename T, typename F>
   T use_f(T v, F f)
   {
   	static int count = 0;
   	count ++;
   	std::cout << " use_f count = " << count
   		<< ", &count = " << &count << std:: endl;
   	return f(v);
   }
   
   class Fp
   {
   	private:
   		double z_;
   	public:
   		Fp(double z = 1.0) : z_(z) {}
   		double operator() (double p) { return z_ * p; }
   };
   
   class Fq
   {
   	private:
   		double z_;
   	public:
   		Fq(double z = 1.0) : z_(z) {}
   		double operator()(double q) { return z_ + q; }
   };
   ```

   ```c++
   //callable.cpp
   
   #include "somedefs.h"
   #include <iostream>
   
   using namespace std;
   
   double dub(double x) { return 2.0 * x; }
   double square(double x) { return x * x; }
   
   int main()
   {
   	double y = 1.21;
   	cout << "pointer dub:\n";
   	cout << " " << use_f(y, dub) << endl;
   	cout << "pointer square:\n";
   	cout << " " << use_f(y, square) << endl;
   	cout << "object Fp:\n";
   	cout << " " << use_f(y, Fp(5.0)) << endl;
   	cout << "object Fq:\n";
   	cout << " " << use_f(y, Fq(5.0)) << endl;
   	cout << "lambda 1:\n";
   	cout << " " << use_f(y, [](double u) { return u * u; }) << endl;
   	cout << "lambda 2:\n";
   	cout << " " << use_f(y, [](double u) { return u + u / 2.0; }) << endl;
   
       return 0;
   }
   ```

2. 修复问题

   上述程序清单中的函数指针、函数对象和lambda 表达式有一个相同的地方，它们都接受一个double参数并返回一个double值，因此它们的调用特征标（call signature）相同。

   模板function是在头文件functional中声明的，它从调用特征标的角度定义了一个对象，可用于包装调用特征标相同的函数指针、函数对象 或lambda表达式。因此可使用 function创建六个包装器，用于表示6个函数、函数符和 lambda。这样，在对use_f( )的全部6次调用中，让F的类型都相同 （function），因此只实例化一次。据此修改后的callable.cpp如下：

   ```c++
   #include "somedefs.h"
   #include <iostream>
   #include <functional>
   
   using namespace std;
   
   double dub(double x) { return 2.0 * x; }
   double square(double x) { return x * x; }
   
   int main()
   {
   	double y = 1.21;
   	function<double(double)> ef1 = dub;
   	function<double(double)> ef2 = square;
   	function<double(double)> ef3 = Fq(10.0);
   	function<double(double)> ef4 = Fp(10.0);
   	function<double(double)> ef5 = [](double u) { return u * u; };
   	function<double(double)> ef6 = [](double u) { return u + u / 2.0; };
   	cout << "pointer dub:\n";
   	cout << " " << use_f(y, ef1) << endl;
   	cout << "pointer square:\n";
   	cout << " " << use_f(y, ef2) << endl;
   	cout << "object Fp:\n";
   	cout << " " << use_f(y, ef3) << endl;
   	cout << "object Fq:\n";
   	cout << " " << use_f(y, ef4) << endl;
   	cout << "lambda 1:\n";
   	cout << " " << use_f(y, ef5) << endl;
   	cout << "lambda 2:\n";
   	cout << " " << use_f(y, ef6) << endl;
   
       return 0;
   }
   ```

3. 其他方式

   1. 只使用一个临时 function对象，将其用作函数use_f( )的参数:

      typedef function< double(double) > fdd;

      cout << use_f(y, fdd(dub)) << endl;

      ...

   2. 修改后的程序清单让use_f( )的第二个实参与形参f匹配，但另一种 方法是让形参f的类型与原始实参匹配。为此，可在模板use_f( )的定义 中，将第二个参数声明为function包装器对象，如下所示：

      template < typename T >

      T use_f(T v, std::function< T(T) > f) { ... }

      这样函数的调用如下：

      cout << use_f< double >(y, dub) << endl;

      ...

      参数dub、Fp(5.0)等本身的类型并不是function，因 此在use_f后面使用了来指出所需的具体化。这样，T被设置为 double，而std::function< T(T) >变成了std::function< double(double) >。


### 可变参数模板

1. 模板和函数参数包：

   C++11提供了一个用省略号表示的元运算符（meta-operator），让您能够声明表示模板参数包的标识符，模板参数包基本上是一个类型列表。同样，它还让您能够声明表示函数参数包的标识符，而函数参数包 基本上是一个值列表。

   template< typename... Args>

   void show_list(Args... args) { ... }

   调用：

   show_list();

   show_list(888.5, "cat");

   show_list(12, 12.5, "aaa");

2. 展开参数包

   将省略号放在函数参 数包名的右边，将参数包展开

   ```c++
   #include <iostream>
   #include <string>
   
   using namespace std;
   
   void show_list() {}
   
   template<typename T>
   void show_list(const T& value)
   {
   	cout << value << endl;
   }
   
   template<typename T, typename... Args>
   void show_list(const T& value, const Args&... args)
   {
   	cout << value << ',';
   	show_list(args...);
   }
   
   int main()
   {
   	int n = 14;
   	double x = 2.71828;
   	string mr = "hello";
   	show_list(n, x);
   	show_list(x * x, '!', 7, mr);
   
       return 0;
   ```
   

