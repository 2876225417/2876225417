# 第一章 让自己习惯 C++
## 条款1：视C++为一个语言联邦

关于C++的四种次语言(_sub-language_):
1. *C*，*C++*以*C*为基础的。
2. *Object-Oriented C++*，体现了`C++`面向对象的特性。  
   对于内置类型而言`pass-by-value`相比较于`pass-by-reference`会更高效。  
   但对于面对对象的`C++`而言，`pass-value-by-reference-to-const`往往更好。

3. *Template C++*，泛型编程(_generic programming_)是C++重要的一部分。
4. *STL*，C++中的标准模板库。  
   但在Template，STL中迭代器(iterator)和函数对象都是在C的指针上进行构建的。

总结：在C++中对于不同部分的实现根据其特性采用不同的方法。
## 条款2：尽量以 `const, enum, inline` 替换 `#define`
在声明常量时尽量使用 `const, enum`, 声明函数时尽量使用 `inline`, 而不是`define`(在头文件中)。  
对于 `define` 而言，在定义常量时，该及其名称会被覆盖，所以当 _debug_ 时，不易发现这些关于这些常量的错误。  
简而言之，在定义常量时：  
将`define _TIME 30`  
替换成`const int _Time = 30;`以避免上述情况。  
此外，在这种情况下，使用 define 也会对性能产生一定影响(可以忽略不计)。  
在声明常量时替换#define的两种情况：
1. 常量指针(_constant pointers_)   
   定义一个常量的 _char*-based_ 字符串：
   `const char* const Name = "You Wenfei"`
2. 类(class)的成员变量(_member_)  
   定义一个唯一的成员变量
```C++
class GamePlayer{
private:
    static const int NumTurns = 5;  // 常量声明式(赋值)
    static const int _NumTurns;     // 常量定义式(未赋值)
    int scores[NumTurns];           // 使用该常量，此处声明的是一个静态数组
    /* static const 类型的变量必须在声明时就被赋值 */ 
};
const int GamePlayer NumTurns;      // 此处NumTurns已被赋值
const int GamePlayer::_NumTurns = 4;// 此方法也可行
```
By the way，`#define`不能用来定义`class`成员常量，也不具备封装性，例如：`private #define` 是不可行的。

现在，谈谈`enum`，对于`int scores[NumTurns]`利用`enum`来替代`static const`来获得同样的效果。
```C++
class GamePlayer{
private:
    enum { NumTurns = 5 };           // enum hack
    int scores[NumTurns];            // 与上面的效果相同 
};
```
*enum hack*的特点更像`#define`而不是`const`，*enum hack*中的变量并没有地址，因此无法对
这些变量进行取指针和取引用的操作，这是`const`无法实现的，`enum`和`#define`也不会导致额外的内存分配(一般而言，编译器不会为内置类型的`const`
对象设定额外的存储空间，除非有一个*pointer*或*reference*指向该对象)。

关于#define的其他误用情况：  
利用#define实现宏(macro)，宏看起来像函数，但在使用时不会产生函数调用(function call)的开销，例：
```C++
/* #define a macro with parameters that call function f() */
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
void f(int a) { std::cout << a << '\n'; }
/* disadvantages list 
 * 1. 无法保证传入的参数a，b相同
 * 2. 传入的a，b可能是表达式，增加了不确定性
 * 3. namespace(命名空间)污染
 * 4. 可读性差，debug麻烦 
 */
```
关于#define实现macro的奇怪事情：
```C++
int a = 5, b = 0;   // define 核算=_=
CALL_WITH_MAX(++a, b);  // 输出： a：7 b：0 -> 累加两次
std::cout << "<-max a: " << a << ", b: " << b << '\n';
CALL_WITH_MAX(++a, b + 10); // 输出： a：8 b：10 -> 累加一次
std::cout << "<-max a: " << a << ", b: " << b << '\n';
```
为了避免这种情况发生，可以用*template **inline***函数进行替代：
```C++
template<typename T>    // 对任意类型作用
inline void max(const T& a, const T& b) { f(a > b ? a : b); }
void f(int a) { std::cout << a << '\n'; }
```
总结：
1. 对于单**纯常量**，用`const`和enum替换`#define`
2. 对于**类似函数形式的宏**，用`inline`函数替换`#define`
## 条款3：尽可能使用 const
常见const与变量组合：
```C++
    char greeting[] = "hello";
    char* p = "world";               // non-const pointer, non-const data
    const char* p = "greeting";      // non-const pointer, const data
    char* const p  = "greeting";     // const pointer, non-const data
    const char* const p = "greeting";// const pointer, const data
```
const的声明位置决定了它的作用：
1. `const`声明在`*`之**前**，用来修饰**值(data)**。
2. `const`声明在`*`之**后**，用来修饰**指针(pointer)**。

关于STL和const
声明`iterator`为`const`等价于声明`pointer`为`const`，使`iterator`指向一个固定的对象。  
如果想让迭代器指向一个固定的对象，则需要的是`const_iterator`：
```C++
    std::vector<int> vec;
    /* iter的作用：T* const */
    const std::vector<int>::iterator iter = vec.begin();
    *iter = 10; // 修改iter所指向的对象的值，可行
    ++iter;     // 修改iter所指向的对象，不可行
    /* cIter的作用：const T* */
    std::vector<int>::const_iterator cIter = vec.begin();
    *cIter = 10;// 修改cIter所指的对象的值，不可行
    ++cIter;    // 修改cIter所指的对象，可行
```

关于函数的返回值和`const`
```C++
    class Rational { ... };
    const Rational operator* (const Rational& lhs, const Rational& rhs);
```
将一个函数的返回值声明为`const`，可以避免如下的情况发生：
```C++
    Rational a, b, c;
    if(a * b = c);
    /* 除非此处重载了操作符 operator=
     * Rational operator(const Rational& a, const Rational& b) 
     */
```
让`a*b`的返回值为`const`，一个*non-const*类型的变量无法赋值给*const*类型的变量，
使`a*b = c`这个表达式无法执行。

`const`型成员变量
```C++
class TextBlock{
public: 
    const char& operator[] (std::size_t position) const {
        return text[position];  // operator for const 对象
    }
    char& operator[] (std::size_t position){
        return text[position];  // operator for non-const 对象
    }
private:
    std::string text;
};
```
示例1：
`TextBlock`类中针对`const`和`non-const`类型的对象进行了`[]`操作的重载。
```C++
    TextBlock tb("Hello");
    std::cout << tb[0]; // 调用 non-const TextBlock::operator[]
    const TextBlock ctb("World"); // 调用 function const TextBlock::operator[]
    std::cout << tb[0];
```
示例2：
```C++
    // 调用 const TextBlock::operator[]
    void print(const TextBlock& ctb){ std::cout << ctb[0]; } 
```
上述代码中对`operator[]`进行了重载，因此它可`const`和`non-const`的`TextBlock`进行不同的处理：
```C++
    std::cout << tb[0]; // 可行，读一个non-const TextBlock
    tb[0] = 'x';        // 可行，写一个non-const TextBlock
    std::cout << cbt[0];// 可行，读一个const TextBlock
    ctb[0] = 'x';       // 不可行，写一个const TextBlock
    
    /* 因为ctb实例是const类型，调用[]操作时，返回的是一个const char&类型
     * 因此对一个const类型的TextBlock实例进行赋值
     * ctb[0] = 'x'; 这个操作不可行
     */
```
注❗❗❗：
```C++
    /* 上面的operator[]的返回类型是reference to char */
    tb[0] = 'x'; // 这个语句才成立
    /* tb[0]本身是存储的一个地址，故返回一个引用值(地址)是合理的 */
```

对比一下返回值为*reference to char*和*pointer to char*的区别
1. 返回值为*reference to char*
```C++
class TextBlock {
public:
	TextBlock(const std::string& s) :text(s) {}
	const char& operator[] (std::size_t position) const {
		return text[position + 1];  // operator for const 对象
	}
	char& operator[] (std::size_t position) {
		return text[position];  // operator for non-const 对象
	}
private:
	std::string text;
};

    TextBlock t1("abcdef");
	const TextBlock t2("abcdef");
	std::cout << t1[1] << '\n';
	std::cout << t2[1] << '\n';

    输出：
    b
    c
```
2. 返回值为*pointer to char*
```C++
class TextBlock {
public:
	TextBlock(const std::string& s) :text(s) {}
	const char* operator[] (std::size_t position) const {
		return &text[position + 1];  // operator for const 对象
	}
	char* operator[] (std::size_t position) {
		return &text[position];  // operator for non-const 对象
	}
private:
	std::string text;
};

	TextBlock t1("abcdef");
	const TextBlock t2("abcdef");
	std::cout << t1[1] << '\n';
	std::cout << t2[1] << '\n';
	
	输出：
	bcdef
	cdef
```
综上可知，返回值为*reference to char*时，返回的是字符串`text`中的一个字符，
即*reference*仅指向`text`中的单一成员；返回值为*pointer to char*时，返回的
是`text`中的一个子串，即以该指针为首的子串。

再看如下示例：
```C++
class CTextBlock{
public:
    char& operator[](std::size_t position) const{
        return pText[position];
    }
private:
    char* pText;
};

    const CTextBlock cctb("Hello");
    char* pc = &cctb[0];            // 指针pc指向pText字符串的第一个字符
    *pc = 'J';                      // 对pText字符串的第一个字符进行修改
    /* pText = "Jello" */
```
此处，`operator[]`并不修改成员的值，但是`*pc = 'J'`;间接的修改了成员变量。
故此处对`operator[]`声明的`const`没有意义。在实际编写代码的过程中也是不现实的。

`mutable` 与 `const`  
无`mutable`
```C++
class CTextBlock
{
public:
    std::size_t length() const;
private:
    char* pText;
    std::size_t textLength;
    bool lengthIsValid;
};

std::size_t CTextBlock::length() const      // const 限定
{
    if(!lengthIsValid)
    {
        textLength = std::strlen(pText);    // 不可行
        lengthIsValid = true;               // 不可行
    }
    return textLength;
}
```
被`const`限定的函数，无法对成员变量进行修改。

有`mutable`
```C++
class CTextBlock
{
public:
	std::size_t length() const;
private:
	char* pText;
	mutable std::size_t textLength;
	mutable bool lengthIsValid;
};

std::size_t CTextBlock::length() const      // const 限定
{
	if(!lengthIsValid)
	{
		textLength = std::strlen(pText);    // 可行
		lengthIsValid = true;               // 可行
	}
	return textLength;
}
```
被`mutable`修饰的成员变量，即使在被`const`限定的函数内也可被修改。

常量性转除(_casting away constness_)  
在某些情况下可能需要对一些被`const`限定的(成员)变量进行修改，此时就需要对这些变量
进行**常量性转除**操作。
示例1：
```C++
    const int i_ = 23;
    const int * pi_ = &i_;
    
    int* non_pi_ = const_cast<int*>(pi_);   // 常量性转除
    *non_pi_ = 233;
```
示例2：
```C++
class TextBlock
{
public:
    const char& operator[] (std::size_t position) const
    {
        return Text[position];
    }

	char& operator[] (std::size_t position)
	{
		return const_cast<char&>        // 要被常量性转除的变量类型为char& 
		(static_cast<const TextBlock&>  // 将返回类型强制转换为const TextBlock&  
		(*this)[position]);             // 以调用 const operator[]
		/* 输出：
		       c
		       b
         */
		
		/* 关于此处常量性转除的另一种写法 
		 * return const_cast<char&>
		 * (static_cast<const TextBlock&>
		 * (*this).Text[position]);     // 错误写法
		 * 直接返回Text中索引为1处的字符
		 * 输出：
		 *     b
		 *     b
		 */
		 
		/* 错误写法 
		 * return const_cast<char&>
		 * (static_cast<const TextBlock&>
		 *  Text[position]);            // 另一种错误写法
		 */
	}
private:
    std::string Text;
};
```
在上面代码中，添加了一个`static_cast`的操作，作用是将`operator[]`转成`const operator[]`，
这样可以在执行到此处时调用`const operator[]`而不是`operator[]`，即它自身，因为这样会导致
它自己调用自己而产生**无限递归**。

所以，想要在`const`限定的成员函数内调用`non-const`成员函数对成员变量进行修改，
就需要用到常量性转除`const_cast`方法进行实现。在`non-const`成员函数内调用被`const`限定的成员函数是正确可行的。

总结：
1. 给变量添加*const*修饰符可以让编译器帮你找出错误。
2. 编译器对于*const*使用很严格，所以在使用时也要注意。
3. 当*const*和*non-const*成员函数实现的功能相同时，在*non-const*函数中调用*const*函数可以避免代码重复。

## 条款4：确定对象被使用前已先被初始化
在C++中，变量如果只被声明而未被赋初值，可能会带来很多不必要的麻烦。
即使这些未被赋初值的变量可能会被默认赋值，但不一定在所有的情况下都是这样。
```C++
    double x; x = 3.3f;
    double x = 3.3f;
    double x; std::cin >> x;
    /* and so on... */
```
当然，这些都只是都内置类型的初始化，对自定义的类中的变量也需要进行初始化，
就是类中常有的构造函数。

关于类成员的初始化的构造函数主要有如下两种：
1. 在构造函数的实现部分进行**赋值**
2. 利用成员初值列进行**初始化**

赋值：在构造函数内部进行赋值，这种操作不是初始化
```C++
class PhoneNumber { ... };
class ABEntry{
public: 
    ABEntry(const std::string& name, 
            const std::string& address, 
            const std::list<PhoneNumber>& phones);
private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
};

ABEntry::ABEntry(const std::string& name, 
                 const std::string& address, 
                 const std::list<PhoneNumber>& phones){
    theName = name;
    theAddress = address;                 
    thePhones = phone;
    numTimesConsulted = 0;
}
```
初始化：利用成员初值列对成员变量进行初始化
```C++
ABEntry::ABEntry(const std::string& name, 
                 const std::string& address, 
                 const std::list<PhoneNumber>& phones):
                 theName(name),         // 拷贝构造
                 theAddress(address),   // 拷贝构造
                 thePhones(phones),     // 拷贝构造
                 numTimeConsulted(0){}
```
利用成员初值列对成员变量进行初始化的效率较高，此种方法调用的是default(默认)构造函数。  
然而利用赋值的方法对成员变量进行赋值也会调用default构造函数，但这时的default构造函数没有发挥任何作用。

无参构造：
```C++
ABEntry::ABEntry()
                 theName(),         // theName的默认构造
                 theAddress(),      // theAddress的默认构造
                 thePhones(),       // thePhones的默认构造
                 numTimeConsulted(0)// 显示声明numTimeConsulted的值为0
                 {}
```

**注❗❗❗**：
在编写成员初值列时，总是列出所有的成员变量，以免给自己带来不必要的麻烦。
对于const或reference类型的变量，它们一定得出现在成员初值列中(初始化)，而不允许被赋值。

例如：  
这种赋值类型的构造就不可行
```C++
class test
{
public:
	test(const int m, const int* n);
private:
	const int t1;
	const int*  b;
};

test::test(const int m, const int* n)
{
	t1 = m;
	b = n;
}
```
只能用成员初值列的构造方式对成员变量进行赋值
```C++
class test
{
public:
	test(const int m, const int* n) :t1(m), b(n) {}
private:
	const int t1;
	const int* b;
};
```

static对象：
1. global对象
2. 定义与namespace作用域内的对象
3. 在class内
4. 在函数内
5. 在file作用域内被声明为static类型的对象

这些对象通过调用自身的析构函数来自动销毁。

例如对于一个在线文件管理系统：
```C++
class FileSystem{
public:
    std::size_t numDisks() const;
};
extern FileSystem tfs;  // declare a global instance

/* 在另一个文件定义的对象 */
class Directory{
public:
    Directory ( parameter... );
};
Directory::Directory( parameter... ){
    std::size_t disks = tfs.numDisks(); // 使用tfs对象
}

/* 在另一个文件内的对象 */
/* Directory对象的实例化 */
Directory tempDir( parameters... );     // 使用tfs对象
```
为确保上述功能执行顺利，就要在`tfs`被调用之前将`tfs`初始化，否则会给自己带来很多不必要的麻烦。

解决该方法的另一种思路，将别的文件内的对象复制到需要调用该函数的地方，并声明为`static`，
该函数返回一个指向该对象的一个*reference*，然后调用这个函数，这样保证了所使用的对象是被初始化了的。
例如：
```C++
class FileSystem ( ... );
FileSystem& tfs(){          // 这个函数用来替换 tfs 对象，
    static FileSystem fs;   // 定义并初始化一个local static 对象
    return fs;              // 返回一个 reference 指向上述对象
}
class Directory ( ... );
Directory::Directory ( parameters ){
    std::size_t disks = tfs().numDisks();   // 将原来的 reference to tfs 对象
}                                           // 替换为现在的 tfs()
Directory& tempDir(){       // 这个函数用来替换 tempDir 对象
    static Directory td;    // 定义并初始化一个local static 对象
    return td;              // 返回一个 reference 指向上述对象
}
```
这种处理方式中的函数简单易用，但也有不足的地方：
在多线程环境下，这种操作会带来不确定性，解决方法之一就是在单线程启动阶段(_single-threaded startup portion_)
手动调用所有*reference-returning*函数，这种做法可以避免与初始化相关的线程竞争(_race conditions_)的情况。

但最重要的一点是，在使用*local static*这种解决方法的过程中，应该有一个合理的对象初始化次序，简而言之：
1. 手动初始化内置类型对象
2. 使用成员初值列对各个成员变量进行初始化
3. 仔细考虑对象初始化的先后顺序

总结：
1. 为内置类型对象手动初始化。
2. 使用成员初值列对各个内置类型成员变量进行初始化而不要函数内对各个成员变量进行赋值。
3. 若想跨文件进行初始化，使用*local static*替换*non-local static*对象这种做法最为妥当。
