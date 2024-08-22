# STL源码分析

## 第一章 STL概述

### 可能令你困惑的C++语法

1. 类里面的static成员

当使用模板类初始化多个实例时，这些实例共享这个模板（同类型）类的static成员。

```C++
#include <iostream>

template<typename T>
class myClass
{
public:
	static int i1_;
	static float f1_;
};

// int type template
int myClass<int>::i1_ = 1;
float myClass<int>::f1_ = 1.1f;

// float type template
int myClass<float>::i1_ = 2;
float myClass<float>::f1_ = 2.2f;

int main()
{
	myClass<int> ic1_, ic2_;
	myClass<float> fc1_, fc2_;

	std::cout 
	    << ic1_.i1_ << ' '
		<< ic1_.f1_ << ' '
		<< ic2_.i1_ << ' '
		<< ic2_.f1_ << ' '
		<< fc1_.i1_ << ' '
		<< fc1_.f1_ << ' '
		<< fc2_.i1_ << ' '
		<< fc2_.f1_ << ' ';
	return 0;
}
```

综上可知，当对ic1_实例的i1_成员进行操作时，i2_对应的成员也会发生变化，
因为它们使用的是同一个模板类`myClass<int>`，同理，fc1_和fc2_也如此。

2. 模板类的特殊设计

初始化实例时，编译器会根据提供的指针的类型进行匹配。
```C++
#include <iostream>

// generalized
template<class T, class O>
struct tc
{
	tc() { std::cout << "I, O" << '\n'; }
};

// specialized 
template<class T>
struct tc<T*, T*>
{
	tc() { std::cout << "T*, T*" << '\n'; }
};

// specialized
template<class T>
struct tc<const T*, T*>
{
	tc() { std::cout << "const T*, T*" << '\n'; }
};

int main()
{
	tc<int, char> obj1;
	tc<int*, int*> obj2;
	tc<const int*, int*> obj3;
	tc<const int*, const int*> obj4;
	tc<int*, const int*> obj5;
	return 0;
}
```
输出结果：

![image_10.png](image_10.png)

若匹配不到，则会自动选择默认模板类。

3. 模板类中可再存在模板成员
```C++
#include <iostream>

class alloc{};

template<class T, class Alloc = alloc>
class vector{                       // template class
public:
    typedef T value_type;           // T -> value_type
    typedef value_type* iterator;   // value_type* -> iterator
    
    template<class I>               // template member function
    void insert(iterator position, I first, I last){ std:: cout << "insert()" << '\n'; }
};

int main(){
    int ia[5] = {0, 1, 2, 3, 4};
    
    vector<int> x;
    vector<int>::iterator ite = nullptr;
    x.insert(ite, ia, ia + 5);

    return 0;
}

```

4. template 参数可以根据前一个 template 参数而设定默认值
```C++
#include <iostream>
#include <cstddef>

class alloc {};

template<class T, class Alloc = alloc, size_t BufSiz = 0>
class deque{
public:
    deque() { std::cout << "deque()" << '\n'; }
};

// 根据前一个参数值T， 设定下一个参数Sequence的默认值为deque<T>
template<class T, class Sequence = deque<T>>
class stack {
public: stack() { std::cout << "stack()" << '\n'; }
private: Sequence c;
};

int main() {
    stack<int> x;
    return 0;
}
```

输出结果：

![image_11.png](image_11.png)

由输出结果可以看出，在初始化stack实例时，stack内的Sequence会先根据模板内的参数初始化一个deque实例，再实例化stack。

综上可知，当初始化一个模板类时，模板类中的模板中若有其他模板类，会优先初始其他化模板类对应的实例。


示例1，如下所示：
```C++
#include <iostream>
#include <cstddef>

class alloc {};

template<class T, class Alloc = alloc, size_t BufSiz = 0>
class deque{
public:
    deque() { std::cout << "deque()" << '\n'; }
};

// 先初始化deque的实例
template<class T, class Sequence = deque<T>>
class stack {
public: stack() { std::cout << "stack()" << '\n'; }
private: Sequence c;
};

// 先初始化stack的实例
template<class T, class Test = stack<T>>
class test
{
public:
    test() { std::cout << "test()" << '\n'; }
private:
    Test t;
};

int main() {
    test<int> t_;
    return 0;
}
```

输出结果：

![image_12.png](image_12.png)

示例2，如下所示
```C++
#include <iostream>
#include <cstddef>

class alloc {};

template<class T, class Alloc = alloc, size_t BufSiz = 0>
class deque{
public:
    deque() { std::cout << "deque()" << '\n'; }
};

template<class T, class Test = deque<T>>
class test
{
public:
    test() { std::cout << "test()" << '\n'; }
private:
    Test t;
};

template<class T, class Sequence = deque<T>, class test_ = test<T>>
class stack {
public: stack() { std::cout << "stack()" << '\n'; }
private: Sequence c; test_ _t;
};

int main() {
    stack<int> t_;
    return 0;
}
```
结果如下：

![image_13.png](image_13.png)

过程推导：
* 实例化deque -> deque()
* 实例化test_, 但实例化test_前需实例化deque -> deque()
* 实例化test_完成 -> test()
* 实例化stack完成 -> stack()


5. 模板类可拥有Non-type的模板参数
```C++
#include <iostream>
#include <cstddef>

class alloc {};

inline size_t __deque_buf_size(size_t n, size_t sz) {
    return n != 0 ? n : (sz < 512) ? size_t(512 / sz) : size_t(1);
}

template<class T, class Ref, class Ptr, size_t BufSiz>
struct __deque_iterator {
    typedef __deque_iterator<T, T&, T*, BufSiz> iterator;
    typedef __deque_iterator<T, const T&, const T*, BufSiz> const_iterator;
    static size_t buffer_size() { return __deque_buf_size(BufSiz, sizeof(T)); }
};

template<class T, class Alloc = alloc, size_t BufSiz = 0>
class deque {
public: typedef __deque_iterator<T, T&, T*, BufSiz> iterator;
};

int main() {
    std::cout << deque<int>::iterator::buffer_size() << '\n';
    std::cout << deque<int, alloc, 64>::iterator::buffer_size() << '\n';
    return 0;
}
```

在类`deque`中的成员`iterator`初始化时利用模板中的参数`BufSiz`是一个`value`而不是一个`type`。

由此可知，模板接受非类型`non-type`的参数。

6. Bound Friend Template

类模板的某个具体实例(instantiation)与其友元函数模板的具体实例(instantiation)一一对应。
```C++
#include <iostream>
#include <cstddef>

class alloc {};

template<class T, class Alloc = alloc, size_t  BufSiz = 0>
class deque{
public: deque() { std::cout << "deque()" << '\n'; }
};

template<class T, class Sequence>
class stack;

template<class T, class Sequence>
bool operator==(const stack<T, Sequence>& x, const stack<T, Sequence>& y);

template<class T, class Sequence>
bool operator<(const stack<T, Sequence>& x, const stack<T, Sequence>& y);

template<class T, class Sequence = deque<T>>
class stack{
    // style 1
    friend bool operator== <T> (const stack<T>& , const stack<T>& );
    friend bool operator< <T> (const stack<T>& , const stack<T>& );
    // style 2
    friend bool operator== <T> (const stack&, const stack&);
    friend bool operator< <T> (const stack&, const stack&);
    // style 3
    friend bool operator== <> (const stack&, const stack&);
    friend bool operator< <> (const stack&, const stack&);
    // invalid style
    //friend bool operator== (const stack&, const stack&);
    //friend bool operator< (const stack&, const stack&);
public: stack() { std::cout << "stack()" << '\n'; }
private: Sequence c;
};

template<class T, class Sequence>
bool operator==(const stack<T, Sequence>& x, const stack<T, Sequence>& y){
    std::cout << "operator==()" << '\t';
    return true;
}

template<class T, class Sequence>
bool operator< (const stack<T, Sequence>& x, const stack<T, Sequence>& y){
    std::cout << "operator<()" << '\t';
    return true;
}

int main(){
    stack<int> x;
    stack<int> y;
    
    std::cout << (x == y) << '\n';
    std::cout << (x < y) << '\n';

    //stack<int> y_;
    // invalid
    // std::cout << (x == y_) < '\n';   // no match for this
    // std::cout << (x < y_) << '\n';   // no match for this

    return 0;
}
```

一般步骤：在模板类内用友元函数声明要重载的运算符，（必须）再对声明的重载友元函数进行实现。

输出结果：

![image_14.png](image_14.png)

7. 类模板显式声明
```C++
#include <iostream>

#define __STL_TEMPLATE_NULL template<>

template<class Key>
struct hash{
    void operator() () { std::cout << "hash<T>" << '\n'; }
};

// explicit specialization
__STL_TEMPLATE_NULL struct hash<char>{
    void operator() () { std::cout << "hash<char>" << '\n'; }
};

__STL_TEMPLATE_NULL struct hash<unsigned char>{
    void operator() () { std::cout << "hash<unsigned char>" << '\n'; }
};

int main(){
    hash<long> t1;
    hash<char> t2;
    hash<unsigned char> t3;
    
    t1();
    t2();
    t3();
    
    return 0;
}
```

输出结果：

![image_15.png](image_15.png)

```C++
hash<long> -> hash<T> ∵hash<long>并没有被显式声明，故调用默认模板

hash<char> -> hash<char> ∵hash<char>已经被显式声明，故调用特定模板

hash<unsigned char> -> hash<unsigned char> 原因同上
```

通过这样的做法，可以自定义不同类型模板类的行为。

8. 临时对象的产生与运用
```C++
#include <vector>
#include <iostream>
#include <algorithm>

template<typename T>
class print{
public: void operator() (const T& elem) { std::cout << elem << ' '; }
};

int main(){
    int ia[6] = {0, 1, 2, 3, 4, 5, 6};
    std::vector<int> iv(ia, ia + 5);
    // print<int> is an unnamed object but not a invoking operation
    std::for_each(iv.begin(), iv.end(), print<int>());

    return 0;
}
```

临时对象即无名对象(unnamed objects)。
实现方法

对()符号进行重载：`void operator() (const T& elem) { std::cout << elem << ' '; }`

不需声明指定对象名称，直接通过()进行调用`std::for_each(iv.begin(), iv.end(), print<int>());`

此临时对象的生命周期

|---|---|
|状态|动作|
|开始|`for_each()(或其他函数)第一次调用print<int>()(临时对象)`|
|结束|`for_each()结束时`|

9. 静态常量成员在class内部直接初始化
```C++
#include <iostream>

template<typename T>
class testClass{
    public: expedient
    static const int _datai = 5;
    static const long _datal = 3l;
    static const char _datac = 'c';
};

int main(){
    std::cout << testClass<int>::_datai << '\n';
    std::cout << testClass<long>::_datal << '\n';
    std::cout << testClass<char>::_datac << '\n;

    return 0;
}
```

10. increment/decrement/dereference 操作符

此类操作符可用于迭代器(iterator)的指针的移动或取值(dereference)。

移动操作可分为前置(prefix)和后置(postfix)。
```C++
#include <iostream>

class INT{
    friend std::ostream& operator<<(std::ostream& os, const INT& i);
public: INT(int i): m_i(i) {};
    
    // prefix: increment and then fetch
    INT& operator++(){
        ++(this->m_i);
        return *this;
    }
    
    // postfix: fetch and then increment
    const INT operator++(int){
        INT temp = *this;
        ++(*this);
        return temp;
    }
    
    // prefix: minus and then fetch
    INT& operator--(){
        --(this->m_i);
        return *this;
    }
    
    // postfix: fetch and then increment
    const INT operator--(int){
        INT temp = *this;
        --(*this);
        return temp;
    }
    
    // deference
    int& operator*() const{
        std::cout << "dereference" << '\n';
        return (int&)m_i;
    }
    
private: int m_i;
};

std::ostream& operator<<(std::ostream& os, const INT& i){
    os << '[' << i.m_i << ']';
    return os;
}

int main(){
    INT I(5);
    std::cout << I++ << '\n';
    std::cout << ++I << '\n';
    std::cout << I-- << '\n';
    std::cout << --I << '\n';
    std::cout << *I << '\n';
    return 0;
}
```

输出结果：

![image_16.png](image_16.png)

分析：
* **prefix increment**：即前置++，类似地像`++i`这样的，优先对i进行操作后再使用。
```C++
 // prefix: increment and then fetch
    INT& operator++(){
        ++(this->m_i);
        return *this;
    }
```
* **postfix increment**：即后置++，类似地像`i++`这样的，优先使用i后，再对其进行操作。
```C++
  // postfix: fetch and then increment
    const INT operator++(int){
        INT temp = *this;   // 声明一个临时对象temp用于存储要操作的目标对象，即还没被操作的目标对象
        ++(*this);
        return temp;        // 返回没有被操作的目标对象
    }
```

* **dereference**：即解引用，返回目标对象的对应类型的地址。
```C++
// deference
int& operator*() const{
    std::cout << "dereference" << '\n';
    return (int&)m_i;
}
```

11. 前闭后开区间表示法 **[ )**
```C++
template<class Iterator, class T>
InputIterator find(Iterator first, Iterator Last, const T& value){
    while(first != last & *first != value) ++first;
    return first;
}

template<class InputIterator, class Function>
Function for_each(InputIterator first, InputIterator last, Function f){
    for(; first != last; ++first) f(*first)
    return f;
}
```
前闭后开即 **[first, last)**, 让指针从first开始
而达不到last(last的前一个位置 => last - 1)。

![image_17.png](image_17.png)

12. function call操作符(operator())
```C++
#include <iostream>

template<class T>
struct plus{
    T operator() (const T& x, const T& y) const { return x + y; }
};

template<class T>
struct minus{
    T operator() (const T& x, const T& y) const { return x - y; }
};

int main(){
    plus<int> plusobj;      // functor object of plus
    minus<int> minusobj;    // functor object of minus
    
    // invoke the functor by the corresponding instances
    std::cout << plusobj(5, 10) << '\n';
    std::cout << minusobj(10, 5) << '\n';
    
    // create the temporary object as the functor
    std::cout << plus<int>()(30, 20) << '\n';
    std::cout << minus<int>()(210, 10) << '\n';
    
    return 0;
}
```
此处对各模板类的 **()** 进行了重载，使之成为了一个仿函数(functor)


## 第二章 空间配置器(allocator)

### 空间配置器的标准接口

一般接口(generalized)

`allocator::value_type  `  
`allocator::pointer  `  
`allocator::const_pointer  `  
`allocator::reference  `  
`allocator::const_reference  `  
`allocator::size_type  `  
`allocator::difference_type  `  

特殊化接口(specified)

|----|----|
|接口|作用|
|`allocator::rebind`|旨在帮助allocator为**不同类型**的元素分配空间，即使allocator已被指定为一个对象分配空间。|
|`allocator::allocator()`|默认构造函数| 
|`allocator::allocator(const allocator&)`|拷贝函数| 
|`template<class U>allocator::allocator(const allocator<U>&)`|泛型拷贝函数| 
|`allocator::~allocator()`|默认析构函数| 
|`pointer allocator::address(reference x) const`|返回指定对象的地址，例如：a.address(x) <=> &x| 
|`pointer allocator::allocate(size_type n, const void* = 0)`|配置空间，用来存储n个类型为T的对象，第二个参数主要用于优化内存优化和管理| 
|`allocator::deallocate(pointer p, size_type n)`|释放之前所分配的内存| 
|`allocator::allocator::max_size() const`|返回所成功分配的最大内存大小的值| 
|`allocator::destroy(pointer p)`|等价于 p->T()| 

用一个例子对`allocator::rebind`进行补充：
```C++
template<class T>
class alloc;

template<typename T, typename Alloc = alloc<T>>
class MyContainer{
public:
    template<typename U>
    struct rebind{
        typedef Alloc<U> other; // Allocator type for type U
        // other represents some other objects
    };
};
```


2. 设计一个简单的空间配置器(Compiler:G++)
```C++
#include <new>          // for placement new
#include <cstddef>      // for ptrdiff_t, size_t
#include <cstdlib>      // for exit()
#include <climits>      // for UINT_MAX
#include <iostream>     // for cerr
#include <vector>       // for test

namespace T_ {
    // allocate memory
    template<class T>
    inline T* _allocate(ptrdiff_t size, T*) {
        std::set_new_handler(0);    // invoke when out of memory
        T* tmp = (T*)(::operator new((size_t)(size * sizeof(T))));  // allocate memory 
        if (tmp == 0) {   // the allocated memory is 0, which means that allocating memory fails
            std::cerr << "out of memory" << '\n';
            exit(0);    // terminate processs
        }
        return tmp;     // return the pointer of the allocated memory block
    }

    // release memory
    template<class T>
    inline void _deallocate(T* buffer) {
        ::operator delete(buffer);  // release the allocated memory
    }

    // constructor
    template<class T1, class T2>
    inline void _construct(T1* p, const T2& value) {
        new(p) T1(value);           // assign value at pointer p
    }

    // destructor
    template<class T>
    inline void _destroy(T* ptr) {
        ptr->~T();                  // invoke the destructor of type class
    }

    template<class T>
    class allocator {
    public:
        typedef T			value_type;
        typedef T* pointer;
        typedef const T* const_pointer;
        typedef T& reference;
        typedef const T& const_reference;
        typedef size_t		size_type;
        typedef ptrdiff_t	difference_type;

        /*-----implement part of the specific API-----*/

        // allocator::rebind->rebind allocator of type U
        template<class U>
        struct rebind {
            typedef allocator<U> other;
        };

        // pointer allocator::allocate->hint used for locality
        pointer allocate(size_type n, const void* hint = 0) {
            return _allocate((difference_type)n, (pointer)0);
        }

        // allocator::deallocate
        void deallocate(pointer p, size_type n) { _deallocate(p); }

        // allocator::construct
        void construct(pointer p, const T& value) {
            _construct(p, value);
        }

        // allocator::destroy
        void destroy(pointer p) { _destroy(p); }

        // pointer allocator::address
        pointer address(reference x) { return (pointer)&x; }

        // const_pointer allocator::const_address
        const_pointer const_address(const_reference x) {
            return (const_pointer)&x;
        }

        // size_type allocator::max_size()
        size_type max_size() const {
            return size_type(UINT_MAX / sizeof(T));
        }
    };
}

int main() {
    int ia[5] = { 0, 1, 2, 3, 4 };
    unsigned int i;

    std::vector<int, T_::allocator<int>> iv(ia, ia + 5);
    for (i = 0; i < iv.size(); i++) std::cout << iv[i] << ' ';
    std::cout << '\n';

    return 0;
}
```
此处的STL设计只是一个简易的allocator。

### SGI空间配置器 {id="sgi_2"} ###
SGI配置器与一般的allocator有所不同

|---------|--------------------------------------|--------|---|
|         |名称|              语法                       |  是否接受参数 |编译器|
| GENERAL |allocator|`vector<int, std::allocator<int>> iv` | 是      |VC or VB|
| SGI     |alloc|`vector<int, sd::alloc> iv`          | 否      |GCC OR G++|

尽管SGI的alloc与标准的allocator有所不同，但是这并不影响使用，因为一般都是用缺省的方式对alloc进行声明。  
SGI的每一个容器都使用缺省的空间配置器alloc，如下所示

```C++
template<class T, class Alloc = alloc>  // 缺省使用alloc为配置器
class vector{ ... };
```

### SGI标准的空间配置器 `std::allocator` {id="sgi_1"} ###

在SGI内部也有一个名为`allocator`的配置器，但是相比较于标准的空间配置器，它的效率较低，两者差
异在于前者把`C++`的 `::operator new` 和 `::operator delete` 进行了简单的包装。  
以下是SGI_style的`allocator`实现。  
注：SGI_style的`allocator`只接受`void*`类型的参数。

```C++
#include <new>
#include <cstddef>
#include <cstdlib>
#include <limits>
#include <iostream>
#include <algorithm>
#include <vector>

/*-----generalized_style-----*/
template<class T>
inline T* allocate(ptrdiff_t size, T*)
{
    std::set_new_handler(0);
    T* tmp = (T*)(::operator new ((size_t)(size * sizeof(T))));
    if(tmp == 0)
    {
        std::cerr << "out of memory" << '\n';
        exit(1);
    }
    return tmp;
}

template<class T>
inline void deallocate(T* buffer)
{
    ::operator delete(buffer);
}

/*-----SGI_style-----*/
template<class T>
class allocator
{
public:
    typedef T			value_type;
    typedef T*			pointer;
    typedef const T*	const_pointer;
    typedef T&			reference;
    typedef const T*	const_reference;
    typedef size_t		size_type;
    typedef ptrdiff_t	difference_type;

    pointer allocate(size_type n)
    {
        return ::allocate((difference_type)n, (pointer)0);
    }

    /*-----in the book-----*/
    /*void deallocate(pointer p) { ::deallocate(p); }
      this deallocate is not runnable */
    /*-----and the below "deallocate" is runnable --------------*/

    void deallocate(pointer p, size_type n) { ::deallocate(p); }
    pointer address(reference x) { return (pointer)&x; }
    
    const_pointer const_address(const_reference x)
    {
        return (const_pointer)&x;
    }
    
    size_type init_page_size()
    {
        return std::max(size_type(1), size_type(4096 / sizeof(T)));
    }
    
    size_type max_size() const
    {
        return std::max(size_type(1), size_type(UINT_MAX / sizeof(T)));
    }
};

int main() {
    int ia[5] = { 0, 1, 2, 3, 4 };
    unsigned int i;

    std::vector<int, allocator<int>> iv(ia, ia + 5);
    for (i = 0; i < iv.size(); i++) std::cout << iv[i] << ' ';
    std::cout << '\n';

    return 0;
}
```

### SGI特殊的空间配置器 `std::alloc` ###

SGI_style的`std::alloc`相比较于SGI_style的std`::allocator`而
言，前者对基层内存配置/释放(`operator::new` & `operator::delete`)进行了效率优化。

一般在C++中关于对象对内存的配置和释放的操作
```
class Foo { ... };
Foo* pf = new Foo;  // allocate memory, then construct object
delete pf;          // destruct object, and release memory
```
关于内存配置和释放的流程如下

|---|---|---|
|   |第一阶段|第二阶段|
|new|调用`::operator new`配置内存|调用`Foo::Foo()`构造对象|
|delete|调用`Foo::~Foo()`析构对象|调用`::operator delete`释放内存|

简单来讲就是：
1. 造一个房子        `::operator new`
2. 人住进去          `Foo::Foo()`
3. 人走了            `Foo::~Foo()`
4. 再把房子给拆了     `::operator delete`

在STL allocator中对上面的步骤有精细的分工：
1. `alloc::allocate()`    配置空间
2. `alloc::deallocate()`  释放空间
3. `alloc::construct()`   构造对象
4. `alloc::destroy() `    析构对象

在STL标准中，allocator定义与`<memory>`中，SGI`<memory>`中包含如下两个文件
```
#include<stl_alloc.h>     // for allocating and deallocating of memory
#include<stl_construct.h> // for constructing and destructing of object
```

![image_18.png](image_18.png)

### 构造和析构基本工具: `construct()` & `destroy()`

![image_19.png](image_19.png)

对上图内容的代码化:

prefix             **specified**
```C++
#include <new>          // for placement new

template<class T1, class T2>
inline void construct(T1* p, const T2& value){
    new(p) T1(value);   // placement new -> invoke T1::T1(value);
}
```

第一版`destroy()`**specified**
```C++
template<class T>       
inline void destroy(T* pointer){
    pointer->~T();      // invoke ~T()
}
```
`pointer->~T();`一般析构。

第二版`destroy()`**generalized**
```C++
template<class T>       
inline void destroy(ForwardIterator first, ForwardIterator last){
    __destroy(first, last, value_type(first));
}
```
`__destroy(first, last, value_type(first));`根据元素的类型选择最合适的方式进行析构。

第二版`destroy()`**SE**
* SE1(char* ver.)  -> `inline void destroy(char*, char*) {}`  
* SE2(wchar_t* ver.)  -> `inline void destroy(wchar_t*, wchar_t*) {}`

判断元素类型是否有`trivial destructor`**generalized**
```C++
template<class ForwardIterator, class T>    
inline void __destroy(ForwardIterator first, ForwardIterator last, T*){
    typedef typename __type_traits<T>::has_trivial_destructor trivial_destructor;
    __destroy_aux(first, last, trivial_destructor());
}
```

* 元素类型有`non-trivial destructor`**specified**
```C++
templace<class ForwardIterator>             
inline void __destroy_aux(ForwardIterator first, ForwardIterator last, __false_type){
    for(; first < last; ++first) destroy(&*first);
}
```

* 元素类型有`trivial destructor`**specified**
```C++
template<class ForwardIterator>            
inline void __destroy_aux(ForwardIterator, ForwardIterator, __true_type){}
```

补充和解释：
1. 用来构造、析构的函数都为全局函数(*STL规范：配置器必须拥有名为**construct()** 和 **destroy()** 两个成员函数*)
2. 上面的`construct()`接受一个 *指针p* 和一个 *初值value* -> 将初值设定到指针所指的位置 => `placement new()`
3. 第一版`destroy()`接受一个指针，用于析构掉指针所指的对象。第二版`destroy()`接受`first`和`last`两个迭代器，用于析构掉*[first, last)*范围内的所有对象
4. 在对象析构之前需要考虑该对象所有的析构函数是否*trivial* -> `value_type()`(获取对象类型) -> `__type_traits<T>`(判断该类型对象的析构函数是否*trivial*)  
4.1 是`(__true_type)`，直接忽略   
4.2 否`(__false_type)`，对范围内的对象逐个进行*destroy()*(第一版)

### 空间的配置与释放 `std::alloc`
一般地，在对象构造前，要对其进行空间的配置，在对象析构后，要对配置的内存进行释放。
在SGI中，上述过程实现流程如下：
* 向 _system heap_ 申请空间
* 考虑多线程(_multi-threads_)状态
* 考虑**内存不足**时的应变措施
* 考虑内存分配时可能导致的**内存碎片**问题  

内存碎片：不同的对象构造需要申请不同的大小的内存空间，但可能由于分配得不够合理（有大有小），导致会有一些较小的内存区块（这部分内存区块无法满足目标对象构造的内存需求）残留，随着时间的推移这些小的内存区块会越来越多，即使总的剩余内存足够大，但是已无法为大型的对象分配空间，因此造成内存浪费。

内存碎片可能会导致的问题：
1. 碎片化的可用空间
2. 内存使用效率低下
3. 分配失败的概率增加
4. 性能下降

内存碎片问题的可能解决方法：
1. 使用固定大小的内存池：预分配固定大小的内存区块
2. 自定义内存配置器：根据实际需求进行内存分配
3. 内存压缩：手动对内存碎片进行清理
4. 智能内存管理：RAII和智能指针
5. 限制动态内存分配：对小型、生命周期短的对象使用基于堆栈的内存分配，对大型、生命周期长的对象使用动态内存分配

在C++中，对内存进行配置的操作是`::operator new()`,对内存进行释放的操作是`::operator delete()`。  
可类比C中的`malloc()`和`free()`函数。  
同时，在SGI中也是通过`malloc()`和`free()`两个函数来实现对内存的配置和释放。

考虑到上面提到的内存碎片问题，SGI设计了双层级的内存配置器：
```C++
#ifdef
typedef __malloc_alloc_template<0> malloc_alloc;
typedef malloc_alloc alloc;     // set alloc as primary allocator
#else
// set alloc as secondary allocator
typedef __default_alloc_template<__NODE_ALLOCATOR_THREADS, 0> alloc;
#endif
```
* 第一级配置器直接使用`malloc()`和`free()`
* 第二级配置器则根据情况采用不同的分配策略：
  * 当需要配置的内存区块**大于128bytes**时，使用一级配置器
  * 当需要配置的内存区块**小于128bytes**时，使用二级配置器 -> _memory pool_(内存池)

SGI会为双层及的配置器封装一个如下的接口（为了使接口符合STL的标准）：
```C++
template<class T, class Alloc>
class simple_alloc{
public:
    static T* allocate(size_t n){
        return 0 == n ? 0 : (T*) Alloc::allocate(n * sizeof(T));
    }
    static T* allocate(void){
        return (T*) Alloc::allocate(sizeof(T)); 
    }
    static void deallocate(T* p, size_t n){
        if(0 != n) Alloc::deallocate(p, n * sizeof(T)); 
    }
    static void deallocate(T* p){
        Alloc::deallocate(p, sizeof(T)); 
    }
};
```
综上可见，该接口内的成员函数都是调用配置器内的成员函数，其中的配置单位为目标元素的大小`sizeof(T)`。

SGI的STL容器全都使用`simple_alloc`这个接口，如：
```C++
template<class T, class Alloc = alloc> // default 
class vector{
protected:
    // specified allocator that allocates a size of element per time
    typedef simple_alloc<value_type, Alloc> data_allocator;
    void deallocate(){
        if(...) data_allocator(start, end_of_storage - start);
    }
    ...
};
```
第一级配置器与第二级配置器关系图

![image_20.png](image_20.png)

第一级配置器与第二级配置器的包装接口和运用方式关系图

![image_21.png](image_21.png)

### 第一级配置器 ___malloc_alloc_template_ 剖析 ###
```C++
#if 0
#   include <new>
#   define THROW_BAD_ALLOC throw bad_alloc
#elif #defined(__THROW_BAD_ALLOC)
#   include <iostream>
#   define __THROW_BAD_ALLOC cerr << "out of memory" << endl; exit(1);
#endif
```
特点：
* malloc-base allocator 比 default allocator速度慢
* 一般而言是thread-safe，内存空间利用率较高

注：
1. 无参模板用法  

`template<>`中定义内置类型与变量名，在实例化时直接给该变量赋值。
```C++
template<int inst>
class ex{
public: 
  void printf() { std::cout << "inst: " << inst; }
};

int main(){
  /* ex<para> e1; here para = inst */
  ex<3> e1;
  e1.printf();
  return 0;
}
```

2. `static`修饰符用法

|                  | static           | non-static       |
|------------------|------------------|------------------|
| with instance    | accessible       | accessible       |
| without instance | accessible       | **inaccessible** |
| *this            | **inaccessible** | accessible       |
```C++
#include <iostream>

class MyClass{
public:
    static void staticFunction(){
        std::cout << "Inside staticFunction" << '\n';
    }

    void nonstaticFunction(){
        std::cout << "Inside nonstaticFunction" << '\n';
    }
};

int main(){
    /* call static function without an instance */
    MyClass::staticFunction();

    /* class static function from an instance (this is uncommon but valid) */
    MyClass obj;
    obj.staticFunction();

    /* cannot call non-static function without an instance */
    MyClass::nonStaticFunction(); /* Error cannot call non-static function 
                                     without instance */

    /* call non-static function from an instance */
    obj.nonstaticFunction();

    return 0;
}
```
3. 函数指针用法
```C++
#include <iostream>

int add(int a, int b);
int subtract(int a, int b);

int main(){
    int(*operation)(int, int);  /* declaration of function pointer */

    operation = add;    /* assign address of add function to operation */
    printf("Addition result: %d\n", (*operation)(5, 3));    /* call function through function pointer */

    operation = subtract;   /* assign address of subtract function to operation */
    printf("Subtract result: %d\n", (*operation)(5, 3));    /* call function through function pointer */

    return 0;
}

int add(int a, int b){ return a + b; }
int subtract(int a, int b) { return a - b; }
```



一级配置器
```C++
template<int inst>
class __malloc_alloc_template {
private:
    /* the below are the function
     pointers which are used for cope with the out-of-memory(oom) */
    static void *oom_malloc(size_t);

    static void *oom_realloc(void *, size_t);

    static void (* __malloc_alloc_oom_handler)();

public:
    /* allocator */
    static void *allocate(size_t n) {
        void *result = malloc(n); // primary allocator employs malloc() directly
        /* when out of memory, invoke oom_malloc() */
        if (0 == result) result = oom_malloc(n);
        return result;
    }

    /* deallocator */
    static void deallocate(void *p, size_t /* n */) {
        free(p);  /* primary allocator employs free() directly */
    }

    /* reallocator */
    static void *reallocate(void *p, size_t /* old_sz */, size_t new_sz) {
        void *result = realloc(p, new_sz);
        /* primary allocator employs realloc() directly
 when out of memory, invoke oom_realloc()*/
        if (0 == result) result = oom_realloc(p, new_sz);
        return result;
    }

    /* emulate set_new_handler(), which means that
       out-of-memory handler is definable */
    static void (*set_malloc_handler(void (*f)()))() {
        void (* old)() = __malloc_alloc_oom_handler;
        __malloc_alloc_oom_handler = f;
        return (old);
    }
};

/* malloc_alloc out-of-memory banding */
template<int inst>
void (* __malloc_alloc_template<inst>::__malloc_alloc_oom_handler)() = 0;

template<int inst>
void* __malloc_alloc_template<inst>::oom_malloc(size_t n){
  void (* my_malloc_handler)();
  void* result;

  for(;;){ /* endlessly allocate and deallocate until allocate successfully */
    my_malloc_handler = __malloc_alloc_oom_handler;
    if(0 == my_malloc_handler) { __THROW_BAD_ALLOC; }
    (* my_malloc_handler)();
    result = malloc(n);
    if(result) return result;
   }
}
```
注：`typedef __malloc_alloc_template<0> malloc_alloc;`  
此处`inst`的值被指定为0

一级配置器主要采用 *C* 中的`malloc()`, `free()`, `realloc()`的函数实现内存的
配置、释放、重配置操作，并实现仿 *C++* 的`new-handler`机制(因为此处并没有直接使用
`::operator new`来配置内存)  

在 *C++* 中 `new handler` 的实现机制：当系统无法满足内存配置需求时，可调用一个自己指定的函数。  
换言之，当 `operator::new` 无法完成内存配置的操作时，在抛出异常前，会调用一个自己指定的
函数 _new-handler_ 来妥协解决内存不足的问题。

注：
1. *SGI*中用 `malloc` 而不是 `::operator new` 来进行内存配置，因此需要实现一个类似 _C++_ 中 `set_new_handler()` 的功能
`set_malloc_handler()`。
2. SGI的一级配置器的 `allocate()` 和 `realloc()` 都是在调用 `malloc()` 和 `realloc()` 不成功(发生内存不足等情况)
后才调用 `oom_malloc()` 和 `oom_realloc()`，然后一直循环尝试配置和释放内存，直到内存配置成功为止，若内存配置始终不成功
那么 `oom_malloc()` 和 `oom_realloc()` 就会抛出` __THROW_BAD_ALLOC` 的 *bad_alloc*异常信息，或直接 `exit(1)` 中止程序。

3. 内存不足的问题的处理函数由程序的设计者完成。


### 第二级配置器 *__default_alloc_template* 剖析

二级配置器相较于以一级配置器多了一些额外的机制，减少了小型内存分配时所带
来的内存浪费(内存碎片)，且分配的内存越小为后续带来的内存空间管理的难度就越大，
因为系统总要靠这多出来的空间进行内存管理。

![image_22.png](image_22.png)

SGI二级配置器的内存分配机制：
* 当分配的内存区块足够大时(大于128bytes)，就转交给一级配置器处理。
* 当分配的内存区块较小时(小于128bytes)，就通过内存池(memory pool)进行管理——层级管理法(sub-allocation)
> **层级管理法**  
> 每次配置一大块内存，并维护对应的自由链表(free-list)。下次弱再有相同大小的内存需求，直接从free-lists中进行
> 分配，如果程序释放了小型内存区块，那么就由配置器回收到free-lists中。

为了方便管理，SGI的二级配置器会主动将任何小额的内存需求量上调至
8 的倍数，并维护16个 free-lists，各自管理大小为 8, 16, 24, 32, 40, 48, 56, 64, 72, 80, 88, 
96, 104, 112, 120, 128 bytes 的小额内存区块。

`free-lists` 结构如下：
```
union obj{
  union obj* free_list_link;
  char client_data[1];
};
```
![image_23.png](image_23.png)

此处，`union obj* free_list_link` 可类比为链表中的 `LinkList* next`，
`char client_data[1]`则用指向实际的内存区块，此种做法可以节省内存的开销。

补充：
* `struct`中的各个成员变量都有自己的内存空间
* `union`中的各个成员变量共享着一块内存空间

二级配置器的部分实现
```C++
#include <cstddef>

enum {__ALIGN = 8};
enum {__MAX_BYTES = 128};
enum {__NFREELISTS = __MAX_BYTES / __ALIGN };

template<bool threads, int inst>
class __default_alloc_template{
private:
    static size_t ROUND_UP(size_t bytes){ return (((bytes) + __ALIGN - 1) & ~(__ALIGN - 1)); }
    /*  '(bytes + __ALIGN -1': This part adds an offset to the 'bytes' value. The '__ALIGN - 1'
     *  is used to ensure that we round up to the next multiple of '__ALIGN'.For example, if '__ALIGN'
     *  is 4, then '__ALIGN - 1' is 3, and adding 3 ensures that the result will be at least a multiple
     *  of 4 greater than 'bytes'.
     *  '& !(__ALIGN - 1)‘: This part performs a bitwise AND operation wit the complement of '(__ALIGN - 4)'.
     *  The complement operation '~' flips all the bits of '(__ALIGN - 1)', effective creating a bitmask where
     *  all bits are set to 1 except for the lower bits determined by '__ALIGN - 1'. By performing a bitwise AND
     *  with this bitmask, we effectively set the lower bits to O, thus rounding down the result to the nearest
     *  multiple of '__ALIGN'.
     */
private:
    union obj{
        union obj* free_list_link;
        char client_data[1];
    };
private:
    /* 16 free-lists */
    static obj* volatile free_list[__NFREELISTS];
    /* employ Nth free-list according to the size
     * of memory chunk (begin with 1st memory chunk)
    */
    static size_t FREELIST_INDEX(size_t bytes){ return (((bytes) + __ALIGN + 1) / __ALIGN - 1); }
    /* return a object in size of n, and add other memory
     * chunks in size of n into free-list if possible
    */
    static void* refill(size_t n);
    /* allocate a memory space that can hold n objs
     * its capacity is size
     * if inconvenient to allocate for n objs, n cloud decrease
    */
    static void* refill(size_t size, int& nobjs);

    // chunk allocation state
    static char* start_free;    // the start position in memory pool, only changes in chunk_alloc()
    static char* end_free;      // the end position in memory pool, only changes in chunk_alloc()
    static size_t heap_size;
public:
    static void* allocate(size_t n) { }
    static void deallocate(void* p, size_t n) { }
    static void* reallocate(void* p, size_t old_sz, size_t new_sz);
};

/* the original value and definition setting for static data member */
template <bool threads, int inst>
char* __default_alloc_template<threads, inst>::start_free = 0;

template <bool threads, int inst>
char* __default_alloc_template<threads, inst>::end_free = 0;

template <bool threads, int inst>
size_t __default_alloc_template<threads, inst>::heap_size = 0;

template <bool threads, int inst>
__default_alloc_template<threads, inst>::obj* volatile
__default_alloc_template<threads, inst>::free_list[__NFREELISTS] =
        {0, 0, 0, 0, 0, 0, 0, 0 , 0, 0, 0, 0, 0, 0, 0, 0};
```

### 空间配置函数 `allocate()`
配置器的标准接口函数allocate()，其工作方式如下：
* 判断区块大小
   * 大于128bytes调用一级配置器
   * 小于128bytes就检查对应的free list
     * 如果free list内有可用的区块，就直接用
     * 如果free list内没有可用的区块，就将区块大小上调至8倍数bytes，然后调用refill()，准备为free list重新填充空间

```C++
  // n must be > 0, to assure that 
  // there is the memory allocated
  static void* allocate(size_t n){
    obj* volatile* my_free_list;
    obj* result;
    
  // if allocating memory > 128 bytes
  if(n > (size_t) __MAX_BYTES){
    return (malloc_alloc:allocate(n));
  }
  
  // select a suitable chunk in 1 of 16 free lists
  my_free_list = free_list + FREELIST_INDEX(n);
  result = *my_free_list;
  if (result == 0){
    // no find a available free list
    // preparing to refill another
    void* r = refill(ROUND_UP(n));
    return r;
  }
  // adjust free list
  *my_free_list = result -> free_list_link;
  return (result);
  };
```

**内存区块从free list调出的操作**

![image_25.png](image_25.png)

### 空间释放函数 `deallocate()`
__default_alloc_template拥有一个标准接口函数deallocate()，其工作方式如下：
* 判断区块大小
  * 大于128bytes，直接调用一级配置器
  * 小于128bytes，找出对应的free list，将区块回收

```C++
    // p can not be 0
    static void deallocate(void* p, size_t n){
        obj* q = (obj *)p;
        obj* volatile* my_free_list;
        
        // if size > 128 bytes, call the primary allocator
        if(n > (size_t) __MAX_BYTES){
            malloc_alloc::deallocate(p, n);
            return;
        }
        
        // selecting the corresponding free list
        my_free_list = free_list + FREELIST_INDEX(n);
        
        // adjust free list, recycle the chunk
        q -> free_list_link = *my_free_list;
        *my_free_list = q;
    }
```

**内存区块回收至free list的操作**

![image_26.png](image_26.png)

### 重新填充`free lists`
在free lists中没有可用的区块时，调用refill()以为free list重新填充空间，
新的空间从内存池(经由chunk_alloc()完成)取出：
* 当内存池中剩余的内存足够时，缺省取得20个新区块
* 当内存池中剩余的内存不足时，取得的新区块数可能小于20

```C++
    // return a obj in size of n
    // and supply some chunks for free list properly sometimes
    // supposing that n is upregulated to a multiple of 8
    template <bool threads, int inst>
    void* __default_alloc_template<threads, inst>::refill(size_t n){
        int nobjs = 20;
        // call chunk_alloc(), try to get n(objs) chunks 
        // as new nodes for free list
        char* chunk = chunk_alloc(n, nobjs); // nobjs -> pass by reference
        obj* volatile* my_free_list;
        obj* result;
        obj* current_obj, * next_obj;
        int i;
        
        // if only get one chunk, then allocate this chunk to the target
        // so there is no new node for free list
        if(l == nobjs) return(chunk);
        // if not, adjust free list and prepare to link a new node
        my_free_list = free_list + FREELIST_INDEX(n);
        
        // create free list in chunk space
        result = (obj*)chunk;   // this piece of chunk will return to user
        // help free list point toward the new allocated space (from memory pool)
        *my_free_list = new_obj = (obj*)(chunk + n);
        // connect the each node in free list
        for(i = 1; ; i++){      // i begin with 1, because i[0] will be return to user
            current_obj = next_obj;
            if (nobjs - 1 == i){
                current_obj -> free_list_link = 0;
                break;
            }else{
                current_obj -> free_list_link = next_obj;
            }
        }
        return(result);
    }
```

### 内存池 `memory pool`
在谈及到`refill()`时，提到了关于`chunk_alloc()`这个函数，
它是用来从内存池中取空间给*free list*。
```C++

template <bool threads, int inst>
char* __default_alloc_template<threads, inst>::chunk_alloc(size_t size, int& nobjs){
    char* result;
    size_t total_bytes = size * nobjs;
    size_t bytes_left = end_free - start_free;  // the rest of memory pool

    if(bytes_left >= total_bytes){
        // the rest of memory pool could satisfy the need
        result = start_free;
        start_free += total_bytes;
        return(result);
    }else if( bytes_left >= size){
        // the rest of memory pool could not totally satisfy the need
        // but only one or more chunks
        nobjs = bytes_left / size;
        total_bytes = size * nobjs;
        result = start_free;
        start_free += total_bytes;
        return(result);
    }else{
        // memory pool can not supply the size o only one chunk
        size_t bytes_to_get = 2 * total_bytes + ROUND_UP(heap_size >> 4);
        // make full use of memory fragments
        if(bytes_left > 0){
            // allocate some surplus memory for free list primarily
            // select the proper free list
            obj* volatile* my_free_list = free_list + FREELIST_INDEX(bytes_left);
            // adjust free list, integrate the surplus memory in memory pool
            ((obj*) start_free) -> free_list_link = *my_free_list;
            *my_free_list = (obj*) start_free;
        }

        // configure heap to supply memory pool
        start_free = (char*) malloc(bytes_to_get);
        if(0 == start_free){
            // heap is not enough, malloc() fails
            int i;
            obj* volatile* my_free_list, *p;
            // check the memory that I currently have
            // allocate the small memory space is dangerous under multi-threads circumstance
            // select a proper free list
            // "proper" means that the unused memory is big enough to satisfy the need
            for(i = size; i <= __MAX_BYTES; i += __ALIGN){
                my_free_list = free_list + FREELIST_INDEX(i);
                p = *my_free_list;
                if(0 != p){
                    // chunk not in free list
                    // adjust free list to release the unused memory
                    *my_free_list = p -> free_list_link;
                    start_free = (char*) p;
                    end_free = start_free + i;
                    // revoke itself to fix nobjs
                    return(chunk_alloc(size, nobjs));
                    // any surplus memory will be integrated in free list as spare
                }
            }
            end_free = 0;   // if occur the accident, allocate the primary allocator
            // turning to out-of-memory
            start_free = (char*)malloc_alloc::allocate(bytes_to_get);
            // if failed, throw exception
            // if succeeded, memory lack will be fixed
        }
     heap_size += bytes_to_get;
        end_free = start_free + bytes_to_get;
        // revoke itself to fix nobjs
        return(chunk_alloc(size, nobjs));
    }
```

`chunk_alloc()`函数通过`end_free - start_free`来判断内存中的内存余量：
* 如果内存足够，就直接调出**20**个区块返回给*free list*
* 如果内存不足
  * 但够满足一个及以上区块，就调出(不足**20**个)剩余的区块
  * 甚至连一个区块都无法调出，此时尝试使用`malloc()`从*heap*中配置内存来增加内存池中的内存余量，配置的内存大小随着配置的次数逐渐增加

举例：
* 当调用`chunk_alloc(32, 20)`时，于是`malloc()`配置**40**个**32bytes**区块，
取出其中的第一个区块，剩下的**19**个放入`free_list[3]`进行维护，再将剩余的**20**个
留给内存池
* 当再调用`chunk_alloc(64, 20)`时，此时`free_list[7]`空空如也，
而内存池中的内存余量**20 * 32 bytes / 64 bytes = 10**个，就先把这**10**个区块返回，
取出其中的第一个区块，剩下的9个放入`free_list[7]`中进行维护。此时内存池空空如也
* 再继续调用`chunk_alloc(96, 20)`，此时`free_list[11]`空空如也，此时想要从内存池中找内存，
然而内存池中也是空空如也，于是以`malloc()`配置**40 + n**(附加量)个**96bytes**区块，取出其中的第一个区块，
再将**19**个交给`free_list[11]`进行维护，剩下的**20+n**(附加量)留给内存池……
* 当整个*system heap*空间都不够了，以至于无法再继续向内存池中添加内存，`malloc()`就不无法继续进行，
`chunk_alloc()`再在*free list*中寻找是否有可用且足够大的区块，有就用，没有就找一级配置器帮忙，
一级配置器使用`malloc()`进行内存配置，但它有*out-of-memory*处理机制(类似于`new handler`)：
  * 如果成功，释放出足够的内存以使用
  * 如果失败，抛出`bad_alloc`异常

![image_27.png](image_27.png)

提供标准配置接口的 simple_alloc
```C++
template<class T, class Alloc>
class simple_alloc{
public:
    static T* allocate(size_t n){
        return 0 == n ? 0 : (T*) Alloc::allocate(n * sizeof(T));
    }
    static T* allocate(void){
        return (T*) Alloc::allocate(sizeof(T)); 
    }
    static void deallocate(T* p, size_t n){
        if(0 != n) Alloc::deallocate(p, n * sizeof(T)); 
    }
    static void deallocate(T* p){
        Alloc::deallocate(p, sizeof(T)); 
    }
};
```

SGI通常使用这种方式来使用配置器：
```C++
template <class T, class Alloc = alloc> // 缺省使用 alloc 为配置器
class vector{
    public:
        typedef T value_type;
    protected:  // 专属配置器，每次配置一个元素大小
        typedef simple_alloc<value_type, Alloc> data_allocator;
};
```

### 内存基本处理工具
STL定义有5个全局函数，作用于未初始化空间上：
1. `construct()` -> **构造**
2. `destroy()` -> **析构**
3. `uninitialized_copy()` -> `copy()`
4. `uninitialized_fill()` -> `fill()`
5. `uninitialized_fill_n()` -> `fill_n()`  

其中 `copy()`, `fill()`, `fill_n()` 都为*STL*算法

### uninitialized_copy() ###
```C++
  template<class InputIterator, class ForwardIterator>
  ForwardIterator
  unitialized_copy(InputIterator first, InputIterator last, 
  ForwardIterator result);
```

`uninitialized_copy()`将内存的配置与对象的构造分离开来，其工作方式：在输入范围内，利用迭代器`i`，该迭代器会调用
`construct(&*(result + (i - first)), *i)`，以此来产生`*i`的拷贝对象，并将拷贝得到的对象放置到输出范围的相对位置上。  
当要实现一个容器时，`uninitialized_copy()`可以帮助容器的全区间构造函数在配置内存区块（以包含范围内的所有元素）后，在该内存区块上构造元素。  
C++对于此还有一个要求——_commit or rollback_：若能对每个元素都成功能进行`uninitialized_copy()`，则*commit*，否则*rollback*。

### uninitialized_fill ### 
```C++
  template<class ForwardIterator, class T>
  void unintialized_fill(ForwardIterator first, ForwardIterator last,
   const T&x);
```

`uninitialized_fill()`将内存配置与对象的构造分离开来，其工作方式：对于输入范围内的每个迭代器`i`，调用
`construct(&*i, x)`，在`i`所指之处放置x的拷贝对象。  
C++对此同样也有*commit or rollback*的要求：若能对每个元素都能成功进行`uninitialized_fill()`，则*commit*，如果有一个*copy constructor*抛出异常(操作失败)，
则将所有成功产生的元素全部析构掉。

### uninitialized_fill_n ###
```C++
  template<class ForwardIterator, class Size, class T>
  ForwardIterator
  uninitialized_fill_n(ForwardIterator first, Size n, const T& x);
```
`uninitialized_fill_n()`将内存配置与对象的构造分离开来，它可以为指定范围内的所有元素设定相同的初值。
对于`[first, first + n]`范围内的每个迭代器`i`，`uninitialized_fill_n()`会调用`construct(&*i, x)`，为其在对应的位置上产生`x`的拷贝对象。  
C++对此同样也有*commit or rollback*的要求：若能对每个元素都能成功`uninitialized_fill_n()`，则*commit*，如果有一个*copy constructor*抛出异常(操作失败)，
则将所有成功产生的元素全部析构掉。

### uninitialized_fill_n 实现方法 {id="uninitialized-fill-n_1"} ###

*uninitialized_fill_n()* 函数接受 3 个参数:
* 迭代器 first 指向欲初始化空间的起始处
* n 表示欲初始化空间的大小
* x 表示初值

```C++
template <class ForwardIterator, class Size, class T>
inline ForwardIterator uninitialized_fill_n(ForwardIterator first,  
                                            Size n,                 
                                            const T& x              
                                            ) {
    return __uninitialized_fill_n(first, n, x, value_type(first));
    // for the above, call value_type() fetch out value type of first
}
```

补充：萃取器(Extractor)
萃取器用于取出迭代器中特定信息，如元素类型、迭代器类型等
```C++
// define a extractor 定义一个萃取器
template <typename Iterator>
struct IteratorTraits{
    using ValueType = typename Iterator::value_type;
};

// polarize pointer 偏特化指针类型
template<typename T>
struct IteratorTraits<T*>{
    using ValueType = T;
};

int main(){
    using Iter = std::vector<int>::iterator;
    using ValueType = typename IteratorTraits<Iter>::ValueType;
    std::cout << "Value type of iterator: " << typeid(ValueType).name() << '\n';
    return 0;
}

输出:
Value type of interator: i -> (int)
```

uninitialized_fill_n()函数执行逻辑：
* 利用萃取器萃取出 first 的value type
* 判断此 value type 是否为POD型别
```C++
template <class ForwardIterator, class Size, class T, class T1>
inline ForwardIterator __uninitialized_fill_n(ForwardIterator first,
                                              Size n,
                                              const T& x,
                                              T1*){
    typedef typename __type_traits<T1>::is_POD_type is_POD;
    return __uninitialized_fill_n_aux(first, n, x, is_POD());
}
```

POD(Plain Old Data)指的是标量型别(scalar type)或传统的C struct型别。
因此POD型别内必然会有trivial ctor/dtor/copy/assignment函数(类比C++中的class)，
因此对POD型别采用效率最高的赋值方法，而对non-POD型别采用最安全的赋值方法：
* 前提条件1：如果 copy assignment 等同于 assignment
* 前提条件2：如果 destructor 等同于 trivial  

对于POD型别的处理：
```C++
template<class ForwardIterator, class Size, class T>
inline ForwardIterator __uninitialized_fill_n_aux(ForwardIterator first,
                                                  Size n,
                                                  const T& x,
                                                  __true_type){
    return fill_n(first, n, x);
}
```

对于non-POD型别的处理：
```C++
template <class ForwardIterator, class Size, class T>
ForwardIterator
__uninitialized_fill_n_aux(ForwardIterator first,
                           Size n,
                           const T& x,
                           __false_type){
    ForwardIterator cur = first;
    for(; n > 0; --n, ++cur) construct(&*cur, x);
    return cur;
}
```
补充：**POD型别**  
在C++中，**Plain Old Data (POD)** 是一种数据类型，具有与C语言中结构体和联合体相似的特性。POD类型可以用于在性能和内存布局上保证与C语言兼容的场景。POD类型的主要特征包括：

1. **标准布局类型（Standard Layout Type）**：
  - 该类型的所有非静态数据成员具有相同的访问权限（全部是`public`、`protected`或`private`）。
  - 类/结构体没有虚函数或虚基类。
  - 所有非静态数据成员具有相同的访问控制（如全部是`public`或全部是`private`）。
  - 派生类和基类必须具有相同的访问控制。

2. **聚合类型（Aggregate Type）**：
  - 没有用户定义的构造函数。
  - 没有私有或受保护的非静态数据成员。
  - 没有基类。
  - 没有虚函数。

POD类型的主要优点包括：
- **二进制兼容性**：POD类型与C语言的数据结构二进制兼容，因此可以直接用于C和C++之间的数据交换。
- **易于序列化**：由于POD类型的内存布局是线性的和固定的，可以直接将其写入文件或通过网络发送。
- **性能优化**：编译器可以对POD类型进行更多的优化，因为它们没有复杂的构造函数、析构函数或虚函数表。

### 示例

以下是一个POD类型的示例：

```C++
struct Point {
    int x;
    int y;
};

struct Rectangle {
    Point top_left;
    Point bottom_right;
};
```

以上两个结构体都是POD类型，因为它们符合POD类型的所有要求。它们没有用户定义的构造函数、析构函数、虚函数，并且数据成员的访问权限都是`public`。

### 非POD类型示例

以下是一个非POD类型的示例：

```C++
struct NonPOD {
    NonPOD() : x(0), y(0) {} // 用户定义的构造函数
    int x;
    int y;
};
```

由于`NonPOD`具有用户定义的构造函数，因此它不是POD类型。

### uninitialized_copy() 实现方法 {id="uninitialized-copy_1" id="uninitialized-copy_2"} ###
uninitialized_copy()函数接受三个参数：
* 迭代器 first 指向输入端的起始位置
* 迭代器 last 指向输入端的结束位置
* 迭代器 result 指向输出端的起始处

输入端区间表示为[first, last)的前闭后开区间

```C++
template<class InputIterator, class ForwardIterator>
inline ForwardIterator uninitialized_copy(InputIterator first,
                                          InputIterator last,
                                          ForwardIterator result){
    return __uninitialized_copy(first, last, result, value_type(result));
}
/* value_type 获取 first 的 value type */
```
这个函数的执行逻辑：
1. 先用萃取器萃取出迭代器result的value type
2. 判断该类型是否为POD类型

```C++
template <class InputIterator, class ForwardIterator, class T>
inline ForwardIterator __uninitialized_copy(InputIterator first,
                                            InputIterator last,
                                            ForwardIterator result,
                                            T*){
    typedef typename __type_traits<T>::is_POD_type is_POD;
    return __uninitialized_copy_aux(first, last, result, is_POD());
    /* apt to use the result of is_POD() to complement type inference */
}
```

POD(Plain Old Data)指的是标量型别(scalar type)或传统的C struct型别。
因此POD型别内必然会有trivial ctor/dtor/copy/assignment函数(类比C++中的class)，
因此对POD型别采用效率最高的赋值方法，而对non-POD型别采用最安全的赋值方法：
* 前提条件1：如果 copy assignment 等同于 assignment
* 前提条件2：如果 destructor 等同于 trivial

对于POD型别的处理：
```C++
template<class InputIterator, class ForwardIterator>
inline ForwardIterator __uninitialized_copy_aux(InputIterator first,
                                                InputIterator last,
                                                ForwardIterator result,
                                                __true_type){
    return copy(first, last, result);   // invoke copy() from STL
}
```

对于non-POD型别的处理：
```C++
template<class InputIterator, class ForwardIterator>
inline ForwardIterator __uninitialized_copy_aux(InputIterator first,
                                                InputIterator last,
                                                ForwardIterator result,
                                                __false_type){
    ForwardIterator cur = result;
    for(; first != last; ++first, ++cur) construct(&*cur, last);
    /* 此处只能够一个一个的元素进行构造，不能进行批量操作 */
    return cur;
}
```

对于 char* 和 wchar_t* 这两种类型，
采用最有效率的做法 memmove(直接对内存中的内容进行移动)：

* 针对 const char*
  ```C++
  /* a specialized version for const char* */
  inline char* uninitialized_copy(const char* first,
                                  const char* last,
                                  char* result){
      memmove(result, first, last - first);
      return result + (last - first);
  }
  ```
  
  * 针对 const wchar*
    ```C++
    /* a specialized version for const wchar* */
    inline wchar_t* uninitialized_copy(const wchar_t* first,
                                        const wchar_t* last,
                                        wchar_t* result){
      memmove(result, first, last - first);
      return result + (last - first);
    }
    ```
### uninitialized_fill 实现方法 {id="uninitialized-fill_1"} ###
uninitialized_fill()函数接受三个参数：
* 迭代器 first 指向输出端的起始位置
* 迭代器 last 指向输出端的结束处(前闭后开区间)
* x 表示初值
    
```C++
template <class ForwardIterator, class T>
inline void uninitialized_fill(ForwardIterator first,
                               ForwardIterator last,
                               const T& x){
    __uninitialized_fill(first, last, x, value_type(first));
}
```

这个函数的执行逻辑：
1. 先用萃取器萃取出迭代器result的value type
2. 判断该类型是否为POD类型

```C++
template<class ForwardIterator, class T, class T1>
inline void __uninitialized_fill(ForwardIterator first,
                                 ForwardIterator last,
                                 const T& x,
                                 T1*){
    typedef typename __type_trait<T>::is_POD_type is_POD;
    __uninitialized_fill_aux(first, last, x, is_POD());
}
```

POD(Plain Old Data)指的是标量型别(scalar type)或传统的C struct型别。
因此POD型别内必然会有trivial ctor/dtor/copy/assignment函数(类比C++中的class)，
因此对POD型别采用效率最高的赋值方法，而对non-POD型别采用最安全的赋值方法：
* 前提条件1：如果 copy assignment 等同于 assignment
* 前提条件2：如果 destructor 等同于 trivial

对于POD型别的处理：
```C++
template<class ForwardIterator, class T>
inline void __uninitialized_fill_aux(ForwardIterator first,
                                     ForwardIterator last,
                                     const T& x,
                                     __true_type){
    fill(first, last, x);
}
```

对于non-POD型别的处理：
```C++
template<class ForwardIterator, class T>
void __uninitialzied_fill_aux(ForwardIterator first,
                              ForwardIterator last,
                              const T& x,
                              __false_type){
    ForwardIterator cur = first;
    for(; cur != last; cur++) construct(&*cur, x);
}
```

对内存进行操作的函数的泛化版本和特化版本

![image_28.png](image_28.png)




