# 3. Lambdas in C++14

相较于 C++11， C++14 对 Lambda 表达式进行了两项重要的增强：
* 带初始化器的捕获
* 泛型 Lambda

此外，标准还更新了一些规则以解决 C++11 中存在的问题，例如：
* Lambda 默认参数
* 返回类型为 auto 

## Lambda 表达式的默认参数
在 C++14 中， Lambda 表达式可以使用默认参数，这一小特性使得 Lambda 表达式更像普通函数：
```C++
#include <iostream>

int main(){
    const auto lam = [](int x = 10){ std::cout << x << '\n'; }
    lam();
    lam(100);
}
/* output:
 * 10
 * 100
 */
```

## 返回类型(Return Type)
在 C++14 中，返回类型的推导功能得到了改进和扩展，使得 Lambda 表达式和普通函数都可以使用'auto'作为返回类型：
```C++
    auto myFunction(){
        int x = computeX(...);
        int y = computeY(...);
        return x + y;
    }
```
在上面的例子中，编译器会推导出'int'作为返回类型。

Lambda 表达式遵循和使用'auto'返回类型的函数相同的规则：
```C++
auto foo = [](int x){
    if(x < 0) return x * 1.1f;  // return type -> float
    else return x * 2.1;        // return type -> double
}
```
上面的代码段无法通过编译，因为第一个返回语句返回'float'，
而第二个返回语句返回‘double'， 编译器无法确定统一的返回类型，
因此需要确保所有返回语句推导出相同的类型。

### 返回 Lambda 表达式 {id="Lambda_1"}
```C++
#include <iostream>
#include <functional>

std::function<int(int)> CreateMulLambda(int x){
    return [x](int param) noexcept { return x * param; };
}

int main(){
    const auto lam = CreateMulLambda(10);
    std::cout << sizeof(lam);
    return lam(2);
}
/* output:
 * 64
 */
```
使用'std::function'需要指定函数签名(函数返回类型及参数类型)并包含额外的头文件`'<function>'`。
'std::function'是一个较大的对象，在 CLANG 18 中的大小为 64 字节！

这一缺点在 C++14 中得到了改进，使得代码可以大幅简化的同时性能更高：
```C++
#include <iostream>

auto CreateMulLambda(int x) noexcept{
    return [x](int param){ return x * param; };
}

int main(){
    const auto lam = CreateLambda(10);
    std::cout << sizeof(lam);
    return lam(2);
}
/* output:
 * 4
 * 20
 */
```
现在完全可以使用编译时类型推导来取代辅助类型，在 CLANG 18 中，lambda 表达式的大小仅为 4 字节，比使用'std::function'的内存开销要小得多，
同时赋予函数'createMulLambda' noexcept 属性，确保其不会抛出任何异常，但在使用'std::function'时不能这么做。

## 带初始化器的捕获
在 C++14 中， Lambda 表达式可以在捕获列表中创建新的成员变量并进行初始化，这一特性被称为带初始化器的捕获或者广义 Lambda 捕获：
```C++
#include <iostream>
int main(){
    int x = 30;
    int y = 12;
    const auto foo = [z = x + y]() { std::cout << z << '\n'; };
    x = 0;
    y = 0;
    foo();
}
/* output:
 * 42
 */
```
在上面的代码中，编译器生成了一个新成员变量'z'，并将其初始化为'x+y'，
新变量的类型通过自动类型推导确定，相当于：
```C++
auto z = x + y;
```
Lambda 表达式等价于以下简化的仿函数：
```C++
struct _unnamedLambda{
    void operator ()() const{
        std::cout << z << '\n';
    }
    int z;
} someInstanc;
```
新变量'z'在 Lambda 表达式定义时被初始化，而不是在调用时被初始化，因此在定义 Lambda 之后修改'x'或'y'的值，'z'的值也不会改变。

### 引用作为带初始化器的捕获
通过带初始化器的捕获，可以灵活地创建对外部作用域变量的引用：
```C++
#include <iostream>
int main(){
    int x = 30;
    const auto foo = [&z = x]() { std::cout << z << 'n\'; };
    foo();
    x = 0;
    foo();
}
/* output:
 * 30
 * 0
 */
```
在上面的例子中，'z'是对'x'的引用，相当于：
```C++
auto& z = x;
```
因此，当'x'改变时，'z'也会随之变化。

## 局限性
尽管 C++14 中可以使用带初始化器的捕获，但存在一些限制：
1. 不能使用右值捕获：
   * 不能使用右值引用'&&'进行捕获：
   ```C++
       [&&z = x] // invalid syntax
    ```
2. 不支持参数包：
    * 不能在初始化器的捕获中使用参数包。
   > 一个简单捕获后跟省略号是一个包扩展([temp.variadic])。带初始化器的捕获后根省略号是无效的。
    * 即在 C++14 中，这样的语法是错误的：
   ```C++
    template <class... Args>
    auto captureTest(Args... args){
        return [ ...capturedArgs = std::move(args) ] (){};
    }
   ```
   
## 改进现有问题
C++14 的新特性解决了一些现有问题，例如只能移动类型(only moveable type)的问题，以及一些额外的优化。

### 移动(move)
在 C++11 中，无法按值捕获'unique_ptr'，只能按引用捕获。C++14 允许将对象移动到闭包类型的成员中：
```C++
#include <iostream>
#include <memory>

int main(){
    std::unique_ptr<int> p(new int{10});
    const auto bar = [ptr = std::move(p)] {
        std::cout << "pointer in lambda: " << ptr.get() << '\n';
    };
    std::cout << "pointer in main(): " << p.get() << '\n';
    bar();
}
/* output:
 * pointer in main(): 0
 * pointer in lambda: 0x141420
 */
```
通过捕获初始化器，可以将指针的所有权移动到 Lambda 中，如上面的代码段，'unique_ptr'在闭包对象创建后立即被置为'nullptr'，
而在调用 Lambda 表达式时，指针地址仍然有效。

### 与'std::function'的兼容性问题
如果在 Lambda 表达式中捕获一个只能移动的变量，会使闭包对象不可复制，
这在需要将 Lambda 存储到'std::function'中时会产生问题，
因为'std::function'只接受可复制的可调用对象。
```C++
#include <functional>
#include <iostream>
.....
std::unique_ptr<int> p(new int{10});
std::function<void()> fn = [ptr = std::move(p)](){};    // compile error
```

### 优化
捕获初始化器也可以用作潜在的优化技术。通过在捕获初始化器中计算某些值，可以避免在每次调用 Lambda 表达式时重复计算：
```C++
#include <iostream>
#include <string>
#include <vector>

int main(){
    using namespace std::string_literals;
    const std::vector<std::string> vs = {"apple", "orange", "foobar", "lemon" };
    const auto prefix = "foo"s;
    
    auto result = std::find_if(vs.begin(), vs.end(),
        [&prefix](const std::string& s){
            return s == prefix + "bar"s;
        }
    );
    if(result != vs.end()) std::cout << prefix << "-something found!\n";
    
    result = std::find_if(vs.begin(), vs.end(),
        [savedString = prefix + "bar"s](const std::string& s){
            return s == savedString;
        }
    );
    if(result != vs.end()) std::cout << prefix << "-something found!\n";
}
```
在第一种调用中，每次调用 Lambda 表达式时都会计算字符串的和，在第二个调用中，
通过初始化器(声明即初始化)只计算一次字符串的和，从而优化了性能。

### 捕获成员变量
捕获初始化器还可以用于捕获成员变量的副本，避免空悬引用的问题：
```C++
#include <iostream>
#include <algorithm>

struct Baz{
    auto foo() const{
        return [s = s] { std::cout << s << '\n'; }
    }
    std::string s;
};

int main(){
    const auto f1 = Baz{"abc"}.foo();
    const auto f2 = Baz{"xyz"}.foo();
    f1();
    f2();
}
/* output:
 * abc
 * xyz
 */
```
在'foo()'中，通过复制成员变量到闭包类型来捕获它，此外，使用'autp'推导成员函数'foo()'的返回类型。  
注意：使用'[s = s]'语法时，捕获的变量在闭包类型的作用域中，因此没有冲突。

## 泛型 Lambda {id="Lambda_2"}

早期的 Lambda 规范允许创建匿名函数对象并将它们传递给标准库中的各种泛型算法，
然而，闭包本身并不是“泛型”的，因为不能将模板参数指定为 Lambda 表达式的参数。
但是，在 C++14 的标准中引入了泛型 Lambda 表达式：
```C++
    const auto foo = [](auto x, int y){ 
        std::cout << x << ", " << y << '\n';
    };
    
    foo(10, 1);
    foo(10.1234, 2);
    foo("hello world", 3);
```
Lambda 表达式中的'auto x'相当于在闭包类型的调用运算符中使用模板声明：
```C++
struct{
    template<typename T>
    void operator()(T x, int y) const{
        std::cout << x << ", " << y << '\n';
    }
} someInstances;
```
如果有更多'auto'参数，代码将扩展为单独的参数模板：
```C++
    const auto fooDouble = [](auto x, auto y) { /*...*/ };
```
展开为：
```C++
struct{
    template<typename T, typename U>
    void operator()(T x, U y) const { /*...*/ }
} someOtherInstance;
```

## 可变参数泛型

如果要使用更多的'auto'函数参数，还可以使用“可变参数”：
泛型可变参数 Lambda 表达式求和：
```C++
#include <iostream>

template<typename T>
auto sum(T x) { return x; }

template<typename T1, typename ... T>
auto sum(T1 s, T... ts) { return s + sum(ts...); }

int main(){
    const auto sumLambda = [](auto ... args){
        std::cout << "sum of: " << sizeof...(args) << " numbers\n";
        return sum(args...);   
    };
    std::cout << sumLambda(1.1, 2.2, 3.3, 4.4);
}
/* output:
 * sum of 4 numbers:
 * 11
 */
```
在上面的代码示例中，泛型 Lambda 表达式使用'auto...'来表示可变参数包，本质上，它被扩展为如下调用运算符：
```C++
    struct __anonymousLambda{
        template<typename ... T>
        void operator()(T... args) const { /*...*/ };
    
    };
```
在 C++17 中出现了折叠表达式(fold expression)，它可以进一步改进泛型可变参数 Lambda。

## 使用泛型 Lambda 实现完美转发 {id="Lambda_3"}

使用泛型 Lambda 表达式时，不仅可以使用'auto x'，
还可以像其他'auto'变量一样添加任何限定符，如'auto&'、'const auto&'或'auto&&'。
其中一个非常有用的用法时指定'auto&& x'，这会变成一个转发(通用)引用，从而实现完美转发输入参数。
```C++
#include <iostream>
#include <string>

void foo(const std::string& ) { std::cout << "foo(const string&)\n"; }
void foo(std::string&& ) { std::cout << "foo(const string&&)\n"; }

int main(){
   const auto callFoo = [](auto&& str){
      std::cout << "Calling foo() on: " << str << '\n';
      foo(std::forward<decltype(str)>(str));
   };
   
   const std::string str = "Hello World";
   callFoo(str);
   callFoo("Hello world Ref Ref");
}
/* output:
 * Calling foo() on: Hello World
 * foo(const string&)
 * Calling foo() on: Hello World Ref Ref
 * foo(string&&)
 */
```
上述代码定义了两个'foo'函数的重载版本：一个接受'const std::string&'参数，
另一个接受'std::string&&"参数，'callFoo'这个 Lambda 表达式使用了一个通用引用参数，
如果将这个 Lambda 表达式重写为一个常规函数模板，其构成大概如下所示：
```C++
   template<typename T>
   void callFooFunc(T&& str){
      std::cout << "Calling foo() on: " << str << '\n';
      foo(std::forward<T>(str));
   }
```

## 推导正确类型
当类型推导比较棘手时，泛型 Lambda 可能非常有用，例如：
```C++
#include <iostream>
#include <algorithm>
#include <map>
#include <string>

int main(){
   const std::map<std::string, int> numbers{
      {"one", 1},
      {"two", 2},
      {"three", 3}
   };
   // each time entry is copied from pair<const string, int>
   std::for_each(std::begin(numbers), std::end(numbers),
      [](const std::pair<std::string, int>& entry){
         std::cout << entry.first << " = " << entry.second << '\n';
      }
   );
}
/* output:
 * one = 1
 * three = 3
 * two = 2
 */
```
### 问题与解决方法
此处有个错误，'std::map'的值类型是'std::pair<const Key, T>'，而不是'const std::pair<Key, T>'。
代码因为'std::pair<const std::string, int>'和'const std::pair<std::string, int>&'之间的转换而导致了额外的复制，
此处可以用'auto'解决：
```C++
std::for_each(std::begin(numbers), std::end(numbers),
   [](const auto& entry){
      std::cout << entry.first << " = " << entry.second << '\n';
   }
);
```
完整示例：
```C++
#include <iostream>
#include <algorithm>
#include <map>
#include <string>

int main(){
   const std::map<std::string, int> numbers{
      {"one", 1},
      {"two", 2},
      {"three", 3}
   };
   
   // print address
   for(auto mid = numbers.cbegin(); mig != numbers.cend(); ++mit)
      std::cout << &mit->first << ", " << &mit->second << '\n';
      
      // each time entry is copied from pair<const string, int>
      std::for_each(std::begin(numbers), std::end(numbers),
         [](const std::pair<std::string, int>& entry){
            std::cout << &entry.first << ", " << &entry.second << ": "
                     << entry.first << " = " << entry.second << '\n';
         }
      );
      
      // this time entried are not copied from pair<const string, int>
      // they have the same addresses
      std::for_each(std::begin(numbers), std::end(numbers),
         [](const std::pair<const auto& entry){
            std::cout << &entry.first << ", " << &entry.second << ": "
                     << entry.first << ", " << entry.second << 'n';
         }
      );
}
/* output:
 * 000001E9FBD3B820, 000001E9FBD3B848           
 * 000001E9FBD3B1F0, 000001E9FBD3B218
 * 000001E9FBD3B5E0, 000001E9FBD3B608
 * 0000000E9315F8B0, 0000000E9315F8D8: one = 1
 * 0000000E9315F8B0, 0000000E9315F8D8: three = 3
 * 0000000E9315F8B0, 0000000E9315F8D8: two = 2
 * 000001E9FBD3B820, 000001E9FBD3B848: one = 1
 * 000001E9FBD3B1F0, 000001E9FBD3B218: three = 3
 * 000001E9FBD3B5E0, 000001E9FBD3B608: two = 2
 */
```
前三行显示了 map 中键和值的地址，
中间三行显示了三个新的相同的地址，可能是循环迭代中的临时副本，
最后三行展示了'const auto&'版本，地址与前三行i相同。

要注意的是，在键在被复制的同时，值也会被复制，如果值是一个较大的对象，那么产生的内存开销将会更大。

## 用 Lambda 替换 std::bind1st 和 std::bind2nd {id="Lambda_4"}

从 C++11 开始，std::bind1st 和 std::bind2nd 这些功能已经被弃用，并在 C++17 中被移除。

像 bind1st()、bind2nd()、mem_func()等函数是在 C++98时代引入的，现在可以使用 Lambda 或其他现代方案来实现其功能。
此外，当前的标准中尚未支持完美转发、可变参数模板、decltype等 C++11的技术，因此，在代码中尽量避免使用它们。

以下是已经弃用的功能列表：
* unary_function()/pointer_to_unary_function()
* binary_function()/pointer_to_binary_function()
* bind1st()/binder1st
* bind2nd()/binder2nd
* ptr_fun()
* mem_fun()
* mem_fun_ref()

可以使用 Lambda 或自 C++11 起可用的 std::bind，或自 C++20 起可用的 std::bind_front 来替换 bind1st 和 bind2nd。

使用 std::bind1st 和 std::bind2nd 的示例：
```C++
const auto onePlus = std::bind1st(std::plus<int>(), 1);
cosnt auto twoPlust = std::bind2nd(std::minus<int>(), 1);
std::cout << onePlus(10) << ", " << minusOne(10) << '\n';
```
在上面的代码中，onePlus 是一个由 std::plus 和固定第一个参数为 1 组成的可调用对象，
其中 onePlus(n) 被扩展为 std::plus(1, n)；
同样的， minusOne(n) 是一个由 std::minus 和固定第二个参数为 1 组成的可调用对象，
其中 minusOne(n) 被扩展为 std::minus(n, 1)。

对上述代码使用 Lambda 进行改进：
```C++
const auto onePlus = [](int x){ return std::plus<int>()(1, x); };
const auto minusOne = [](itn x) { return std::minus<int>()(x, 1); };
std::cout << onePlus(10) << ", " << minusOne(10) << '\n';
```
显而易见，使用 Lambda 表达式来绑定参数更加浅显易懂。

## 使用现代 C++ 技术

### 使用 std::bind

相比于'bind1st'或'bind2nd'，'std::bind'更具有灵活性：
```C++
#include <algorithm>
#include <functional>
#include <iostream>

int main(){
   using std::placeholders::_1;
   const auto onePlus = std::bind(std::plus<int>(), _1, 1);
   const auto minusOne = std::bind(std::minus<int>(), 1, _1);
   std::cout << onePlus(10) << ", " << minusOne(10) << '\n';
}
/* output:
 * 11, 9
 */
```
'std::bind'更加灵活，因为它可以支持多个参数，甚至可以它们进行重新排列。
要进行参数管理，需要使用“占位符”。在上面的例子中，使用'_1'来表示将传递给最终函数对象的第一个参数。

但与 Lambda 表达式相比较而言，Lambda 表达式仍显得更加自然：
```C++
auto lamOnePlus1 = [](int b) { return 1 + b; };
auto lamMinusOne1 = [](int b) { return b - 1; };
std::cout << lamOnePlus1(10) << ", " << lamMinusOne(10) << '\n';
/* output:
 * 11, 9
 */
```

在 C++14 中可以使用初始化捕获来提高灵活性：
```C++
auto lamOnePlus = [a = 1](int b) { return a + b; };
auto lamMinusOne = [a = 1](int b) { return b - a; };
std::cout << lamOnePlus(10) << ", " << lamMinusOne(10) << '\n';
/* output:
 * 11, 9
 */
```

## 函数组合

### 使用 std::bind 进行函数组合 {id="std::bind_1"}
函数组合的示例：
```C++
#include <algorithm>
#include <functional>
#include <vector>

int main(){
   using std::placeholders::_1;
   const std::vector<int> v{1, 2, 3, 4, 5, 6, 7, 8, 9};
   const auto val = std::count_if(v.begin(), v.end(),
                                 std::bind(std::logical_and<bool>(),
                                 std::bind(std::greater<int>(), _1, 2),
                                 std::bind(std::less<int>(), _1, 6)));
   return val;
} 
/* output:
 * 3
 */
```

### 使用 Lambda 表达式重写函数组合 {id="lambda_5"}
使用 Lambda 表达式示例：
```C++
   std::vector<int> v{1, 2, 3, 4, 5, 6, 7, 8, 9};
   const auto more2less6 = std::count_if(v.begin(), v.end(),
                           [](int x) reutrn { return x > 2 && x < 6; });
```

## Lambda 表达式提升 {id="lambda_6"}
在使用标准库中的算法时有一个难以解决的问题就是将函数重载传递给接受可调用对象的函数模板：
```C++
#include <algorithm>
#include <vector>

int main(){
   const std::vector<int> vi{1, 2, 3, 4, 5, 6, 7, 8, 9};
   std::for_each(vi.begin(), vi.end(), foo);
}
/* error:
 * No matching function for call to 'for_each' 
 * candidate template ignored: couldn't infer template argument '_Fn'
 */
```
此处的问题是，编译器将'foo'是为模板函数，因此需要解析其类型，但无法确定'foo'接受的类型。

使用 Lambda 表达式解决：
```C++
   std::for_each(vi.begin(), vi.end(),
      [](auto x) { return foo(x); });
```
使用一个包装器，让它处理重载解析并调用适当的'foo()'重载。

使用 完美转发 改进：
```C++
std::for_each(vi.begin(), vi.end(),
      [](auto&& x) { return foo(std::forward<decltype(x)>(x)); });
```

完整示例：
```C++
#include <iostream>
#include <algorithm>
#include <vector>

void foo(int i) { std::cout << "int: " << i << '\n'; }
void foo(float f) { std::cout << "float: " << f << '\n'; }

int main(){
   const std::vector<int> vi{1, 2, 3, 4, 5, 6, 7, 8, 9};
   std::for_each(vi.begin(), vi.end(), 
               [](auto&& x){
                  return foo(std::forward<decltype(x)>(x)); 
   });
}
```

### 更通用的解决方案
对于高级的使用场景，这种方法可能不太理想，因为它不支持可变参数和异常规范，
对于更通用的场景解决方案，可以使用如下宏定义：
```C++
#define LIFT(foo) \
[](auto&&... x) \
noexcept(noexcept(foo(std::forward<decltype(x)>(x)...))) \
-> decltype(foo(std::forward<decltype(x)>(x)...)) \
{ return foo(std::forward<decltype(x)>(x)...); }
```
对上述宏解释：
1. 'return foo(std::forward<decltype(x)>(x)...);'：这是完美转发，用于正确传递输入参数给'foo'函数并保留它们的类型。
2. 'noexcept(noexcept(foo(std::forward<decltype(x)>(x)...)))'：使用嵌套的'noexcept'操作符来检查'foo'的异常规范，根据结果返回'noexcept(true)'或'noexcept(false)'。
3. 'decltype(foo(std::forward<decltype(x)>(x)...))'：用于推导包装 Lambda 的返回类型。

使用'LIFT'宏定义，可以轻松地创建 Lambda 并传递给标准库中的算法进行使用。

## 递归 Lambda 表达式 {id=”lambda_6“}

### 使用常规函数实现递归
常规函数可以很容易地进行递归调用，例如，计算阶乘：
```C++
int factorial(int n) { return n > 1 ? n * factorial(n - 1) : 1; }
int main(){ return factorial(5);}
/* output:
 * 120
 */
```

递归 Lambda 表达式的问题
直接使用 Lambda 表达式实现递归是行不通的：
```C++
int main(){
   auto factorial = [](int n){
      return n > 1 ? n * factorial(n - 1) : 1;
   };
   return factorial(5);
}
/* error: variable 'factorial' declared with deduced type 'auto' cannot appear in its own initializer */
```
无法通过编译的原因是因为在 Lambda 的主体内不能访问尚未完全评估的'factorial'。
用一个仿函数来对 Lambda 表达式进行展开：
```C++
struct fact{
   int operator()(int n) const{
      return n > 1 ? factorial(n - 1) : 1;
   }
};

auto factorial = fact{};
```
在'operator()'内无法访问仿函数类型的变量。

解决方法：
1. 使用'std::function'并捕获它
2. 使用内部 Lambda 并将其作为泛型参数传递

#### 方法一：使用 std::function并捕获它 {id="std::function_2"}
可以将 Lambda 赋值给 'std::function'并捕获他自己，从而实现递归调用：
```C++
#include <functional>
#include <iostream>

int main(){
   std::function<int(int)> factorial = [&](int n) -> int{
      return n > 1 ? n * factorial(n - 1) : 1;
   };
   std::cout << factorial(5) << '\n';
}
/* output:
 * 120
 */
```
在这个方法中，使用'std::function'定义'factorial'并捕获他自己，从而实现递归调用。

### 方法二：使用内部 Lambda 并传递泛型参数 {id="lambda_8"}
使用 C++14 的泛型 Lambda，可以避免'std::function'产生的额外内存开销：
```C++
#include <iostream>

int main(){
   const auto factorial = [](int n) noexcept{
      const auto f_impl = [](int n, const auto& impl) noexcept -> int{
         return n > 1 ? n * impl(n - 1, impl) : 1;
      };
      return f_impl(n, f_impl);
   };
   std::cout << factorial(5) << '\n';
}
```
在这个示例中，创建一个内部 Lambda'f_impl'进行主要处理，并将其自身作为参数传递，
利用 C++14 的泛型 Lambda，可以避免'std::function'的内存开销。

### 更多关于 Lambda 递归调用的方法 {id="lambda_9"}

假设有一个简单的递归函数'sum'用来计算'a'到'b'的平方和：
```C++
#include <iostream>
#include <functional>

auto term = [](int a) -> int { return a * a; };

auto next = [](int a) -> int { return ++a; };

auto sum = [term, next, sum](int a, int b) mutable -> int{
   if(a > b) return 0;
   else return term(a) + sum(next(a), b);
};

int main(){ return sum(1, 10); }
```
上述代码无法通过编译，因为 Lambda 捕获自身在定义时未完全定义，解决方法：
* 使用Y组合子：

#### Y组合子介绍
Y组合子是一种在函数式编程中使用的高阶函数，用于实现递归函数，它允许在不使用显示自引用的情况下定义递归函数。
Y组合子的定义和实现是 Lambda 演算中的一个经典问题，它的核心思想是通过传递自身作为参数来实现递归。

#### Y组合子的定义 {id="Y_combinator"}
在 Lambda 演算中，Y组合子的定义如下：  
> Y=λf.(λx.f(xx))(λx.f(xx))

这个定义表明，Y组合子是一个接受函数f并返回其不动点（固定点）的函数，即：
> Y(f)=f(Y(f))

   ```C++
  #include <utility>
  
  template<class F>
  struct y_combinator{
      F f;
  
      template<class... Args>
      decltype(auto) operator()(Args... args) const{
         return f(*this, std::forward<Args>(args)...);
      }
  };
  
  template<class F>
  y_combinator<std::decay_t<F>> make_y_combinator(F&& f){
      return { std::forward<F>(f) };
  }
  
  int main(){
      auto term = [](int a) -> int { return a * a; };
      auto next = [](int a) -> int { return ++a; };
      auto sum = make_y_combinator([term, next](auto sum, int a, int b) -> int{
         if(a > b) return 0;
         else return term(a) + sum(next(a), b);
      });
      return sum(1, 10);
  }
  ```

> 补充1：在 C++14 中实现Y组合子
> ```C++
> #include <functional>
> 
> // 定义 Y 组合子模板
> template<class F>
> struct y_combinator{
>     F f;
>  
>     template<class... Args>
>     decltype(auto) operator()(Args&&... args) const{
>        return f(*this, std::forward<Args>(args)...);
>     }
>  };
> 
> // 辅助函数，用于创建 Y 组合子
> template<class F>
> y_combinator<std::decay_t<F>> make_y_combinator(F&& f){
>     return { std::forward<F>(f) };
> }
> 
> // 使用 Y 组合子定义一个递归函数计算阶乘
> int main(){
>     auto factorial = make_y_combination([](auto self, int n) -> int {
>        if (n <= 1) return 1;
>        return n * self(n - 1);
>     });
> 
>     return factorial(5);
> }
> /* output:
>  * 120
>  */
> ```

> 补充2：其他组合子
> K 组合子  
> K 组合子，又称恒等函数组合子，用于返回第一个参数，忽略第二个参数。其定义如下：  
> K=λx.λy.x  
> C++ 实现：
>  ```C++
>  auto K = [](auto x){
>        return [x](auto y) { return x; };
>  };
> 
>  int main(){
>     auto k_instance = K(42);
>     return k_instance(100);
>  }
>  /* output:
>   * 42
>   */
>  ```

## 递归 Lambda 表达式与 Y 组合子在 C++ 中的应用 {id="lambda_10"}

### 理想的递归 Lambda 解决方案 {id="lambda_11"}
```C++
   auto fib = [&fib](int64_t x) -> int64_t{
      if (x == 0 || x == 1) return 1;
      else return fib(x - 1) + fib(x - 2);
   };
```
这种方法由于无法在初始化时捕获自身而无法通过编译。

### 使用泛型 Lambda 的解决方案 {id="Lambda_12"}
为了绕过上述问题，可以使用 C++14 的泛型 Lambda：
```C++
   auto fib = [](int64_t x, const auto& lambda) -> int64_t {
      if(x == 0 || x == 1) return 1;
      else return lambda(x - 1, lambda) + lambda(x - 2, lambda);
   };
   
   fib(35, fib);
```
这种方法虽然可以通过编译，但是代码整体冗长且不够美观。

### 改进的 Lambda 方法 {id="Lambda_14"}
通过进一步封装，可以使代码更加简洁
```C++
auto fib = [](itn64_t x){
   auto lambda = [](int64_t x, const auto& lambda) -> int64_t{
      if(x == 0 || x == 1) return 1;
      else return lambda(x - 1, lambda) + lambda(x - 2, lambda);
   };
   return lambda(x, lambda);
};
```

### 性能比较
比较普通函数、std::function、泛型 Lambda 的性能
```C++
#include <iostream>
#include <functional>
#include <chrono>

int64_t f(int64_t x){
    if(x == 0 || x == 1) return 1;
    else return f(x - 1) + f(x - 2);
}

int main(){
    int var = 35;

    std::function<int64_t(int64_t)> f1 = [&f1](int64_t x) -> int64_t{
        if(x == 0 || x == 1) return 1;
        else return f1(x - 1) + f1(x - 2);
    };

    auto f2 = [](int64_t x){
        auto lambda = [](int64_t x, const auto& ff) -> int64_t{
            if(x == 0 || x == 1) return 1;
            else return ff(x - 1, ff) + ff(x - 2, ff);
        };
        return lambda(x, lambda);
    };

    std::cout << "Lambda in C++14 tests\n";

    using namespace std::chrono;
    auto start1 = steady_clock::now();
    auto res1 = f(var);
    auto end1 = steady_clock::now();
    auto diff1 = end1 - start1;

    auto start2 = steady_clock::now();
    auto res2 = f1(var);
    auto end2 = steady_clock::now();
    auto diff2 = end2 - start2;

    auto start3 = steady_clock::now();
    auto res3 = f2(var);
    auto end3 = steady_clock::now();
    auto diff3 = end3 - start3;

    std::cout << "duration (normal function): " <<
    duration_cast<milliseconds>(diff1).count() << " ms\n";

    std::cout << "duration (std::function): " <<
    duration_cast<milliseconds>(diff2).count() << " ms\n";

    std::cout << "duration(auto): " <<
    duration_cast<milliseconds>(diff3).count() << " ms\n";
}
/* output:
 * Lambda in C++14 tests
 * duration (normal function): 45 ms
 * duration (std::function): 244 ms
 * duration(auto): 43 ms
 */
```
性能：'std::function'的性能较差，泛型 Lambda 方法和普通递归函数差不多。
可读性：泛型 Lambda 方法虽然有效，但代码较为冗长且不够美观。