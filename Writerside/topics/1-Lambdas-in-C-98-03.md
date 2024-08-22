# 1. Lambdas in C++98/03

## 本章的主要内容
1. 如何将仿函数(functor)传给标准库中的算法
2. 仿函数和函数指针(function pointer)的限制
3. 为什么函数辅助工具不够好

### 在 C++98/03 中的可调用对象(callable object)
在C++标准库的设计理念中，像"std::sort"、"std::for_each"、"std::transform"等算法可以接受任何可调用对象(callable objects)，
并将其应用于输入容器的元素，然而，在C++98/03标准中，这些算法只能接受函数指针和仿函数(函数对象)。

此处使用一个函数输出vector中的元素。  
一般的实现：
```C++
#include <algorithm>
#include <iostream>
#include <vector>

void PrintFunc(int x){
    std::cout << x << '\n';
}

int main(){
    std::vector<int> v;
    v.push_back(1);
    v.push_back(2);
    for_each(v.begin(), v.end(), PrintFunc);
}
/* output：
 * 1
 * 2
 */
```
上面的代码使用std::for_each遍历整个vector容器(使用for_each的原因是C++98/03还不支持范围遍历)，
并且它将PrintFunc作为一个可调用对象进行传递。

接下来，将这个函数转换为一个仿函数(functor)：
```C++
#include <iostream>
#include <vector>
#include <algorithm>

struct PrintFunctor{
    void operator()(int x) const {
        std::cout << x << '\n';
    }
};

/* or this version
class PrintFunctor{
public:
    void operator()(int x) const {
        std::cout << x << '\n';
    }
};
*/

int main(){
    std::vector<int> v;
    v.push_back(1);
    v.push_back(2);
    for_each(v.begin(), v.end(), PrintFunctor());
}
/* output:
 * 1
 * 2
 */
```
上述例子使用了一个具有operator()的仿函数。

然而函数指针(function pointer)通常是无状态的，即它们只是指向一个函数，并不能存储任何额外的信息或状态，
相比之下仿函数可以拥有成员变量，从而允许它们在多个调用之间存储状态。

定义一个简单的仿函数，让其记录自己被调用的次数，其关键在于定义仿函数中的状态量：
```C++
#include <algorithm>
#include <iostream>
#include <vector>

class printFunctor{
public:
    printFunctor(const std::string str): strText(str), numCalls(0) { }
    void operator()(int x){
        std::cout << strText << x << '\n';
    }
    int getNumCalls() const{
        return numCalls;
    }
private:
    std::string strText;
    mutable int numCalls;
};

int main(){
    std::vector v = {1, 2, 3, 4, 5, 6};
    std::string preText = "Elem: ";
    printFunctor visitor = 
    std::for_each(v.begin(), v.end(), printFunctor(preText));
    std::cout << "numCalls: " << visitor.getNumCalls() << '\n';
}
/* output:
 * 1
 * 2
 * 3
 * 4
 * 5
 * 6
 * numCalls: 6
 */
```

### 仿函数(functor)的问题  
虽然可以使用一个单独的类去设计仿函数，
但是在与算法调用不同的地方编写函数和仿函数，
这导致函数代码在源文件中的位置与算法调用的位置相距很远，由此增加了代码的阅读和维护难度。

在C++98/03中，有一个限制是不能使用局部类型(在函数内部定义的类型)作为模板参数，例如：
```C++
int main(){
    /* define a type inside a function */
    struct PrintFunctor{
        void operator()(int x) const{
            std::cout << x << '\n';
        }
    };
    
    std::vector<int> v(10, 1);
    std::for_each(v.begin(), v.end(), PrintFunctor());
}
```

使用 GCC 的 -std=C++98 标准进行编译，会出现如下错误：
```C++
error: template argument for
'template<class _IIter, class _Funct> _Funct
std::for_each(_IIter, _IIter, _Funct)'
uses local type 'main()::PrintFunctor'
```

### 使用 辅助函数(functional helper) 解决

在标准库中 `<functional>` 头文件，有很多类型和函数可以与标准算法一起使用：  
`std::plus<T>()`: 接受两个参数并返回它们的和。  
`std::minus<T>()`: 接受两个参数并返回它们的差。 
```C++
#include <functional>
#include <iostream>
#include <vector>
#include <algorithm>

int main(){
    std::vector<int> vi1 = {1, 2, 3, 4, 5, 6};
    std::vector<int> vi2 = {2, 3, 4, 5, 6, 7};
    std::vector<int> vRes(vi1.size());
    
    std::transform(vi1.begin(), vi1.end(), vi2.begin(), 
                    vRes.begin(), std::plus<int>());
    std::cout << "Res for plus: ";
    for(int n: vRes) std::cout << n << ' ';
    
    std::transform(vi1.begin(), vi1.end(), vi2.begin(),
                    vRes.begin(), std::minus<int>());
    std::cout << "\nRes for minus: ";
    for(int n: vRes) std::cout << n << ' ';  
}
/* output:
 * Res for plus: 3 5 7 9 11 13
 * Res for minus: -1 -1 -1 -1 -1 -1
 */
```
`std::less<T>()`: 接受两个参数并返回第一个参数是否小于第二个参数。  
`std::greater_equal<T>()`: 接受两个参数并返回第一个参数是否大于或等于第二个参数。  
`std::bind1st`: 创建一个可调用对象，将第一个参数固定为给定值。  
`std::bind2nd`: 创建一个可调用对象，将第二个参数固定为给定值。  

使用辅助函数的好处：
```C++
#include <algorithm>
#include <functional>
#include <vector>

int main(){
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    const size_t smaller5 = std::count_if(
    v.begin(), v.end(), 
    std::bind2nd(std::less<int>(), 5));

    return smaller5;
}
```
上面的代码中，使用std::bind2nd将 5 绑定至std::less函数(x < y)的第二个参数(x < 5)，
即返回的值为 return x < 5。
```C++
    const size_t greater5 = std::count_if(
    v.begin(), v.end(), 
    std::bind1st(std::less<int>(), 5));
```
同理，此处使用std::bind1st将 5 绑定至std::less函数(x > y)的第一个参数(5 < x)，
即返回值的为 return 5 < x。

> 补充：std::bind的用法
> ```C++
> #include <iostream>
> #include <functional>
> 
> int add(int x, int y){ 
>   std::cout << "1st param: " << x << '\n';
>   return x + y; 
> }
> 
> int main(){
>   /* 创建一个绑定第一个参数为10的函数对象 */
>   auto res = std::bind(add, 10, std::placeholders::_1);
> 
>   std::cout << "12 + 10 = " << res(12) << '\n';
> }
> /* output:
>  * 12 + 10 = 1st param: 10 
>  * 22
>  */
> ```


然而，在很多情况下，一个功能实现会有很多函数组成，那么此处语法就会显得很复杂：
```C++
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};
    const size_t val = std::count_if(v.begin(), v.end(),
                               std::bind(std::logical_and<bool>(),
                               std::bind(std::greater<int>(), _1, 2),
                               std::bind(std::less_equal<int>(), _1, 6)));


```
上面这么复杂的语法其实实现的结果就是 return x > 2 && x <= 6。  
这些问题在后面的C++11中得到了改善。







