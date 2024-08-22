# 2. Lambdas in C++11

关于C++11的最终草案[N3337](https://timsong-cpp.github.io/cppwp/n3337/expr.prim.lambda)中Lambda表达式的内容如下：  
Lambda表达式(Lambda Expressions):
1. Lambda 表达式的目的和语法
    * Lambda 表达式提供了一种简洁的方式来创建简单的函数对象。
    * 示例：
    * ```C++
      #include <algorithm>
      #include <cmath>
      void abssort(float* x, unsigned N){
        std::sort(x, x + N, [](float a, float b){ 
           return std::abs(a) < std::abs(b); 
         });
      }
      ```
    * Lambda 表达式的语法:
    * ```Python
      lambda-expression: 
            lambda-introducer lambda-declaratoropt compound-statement
      lambda-introducer:
            [ lambda-captureopt ]
      lambda-capture:
            capture-default
            capture-list
            capture-default, capture-list
      lambda-default:
            &
            =
      lambda-list:
            capture-opt
            capture-list, capture ...opt
      capture:
            identifier
            & identifier
            this
      lambda-declarator:
            ( parameter-declaration-clause ) mutableopt
            exception-specificationopt attribute-specifier-seqopt trailing-return-typeopt
      ```
2. Lambda表达式计算的结果
    * 计算Lambda 表达式会生成一个prvalue临时对象(closure object)。
    * Lambda 表达式不应出现在未计算的操作数中。
    * 闭包对象的行为类似于函数对象。
3. Lambda 表达式类型
    * Lambda 表达式的类型是唯一的、未命名的的非联合类型，成为闭包类型(closure type)。
    * 闭包类型在包含相应 Lambda 表达式的最小块作用域、类作用域或命名空间作用域中声明。
4. Lambda 表达式的默认行为
    * 如果 Lambda 表达式不包含 Lambda 声明符，则默认为'()'。
    * 如果 Lambda 表达式不包含尾返回类型，则尾返回类型默认为以下类型：
      * 如果复合语句的形式为'{ return expression }'， 则返回表达式的类型。
      * 否则，返回'void'。
5. 闭包类型的函数调用运算符
    * 闭包类型具有一个公共的内联函数调用运算符，其参数和返回类型由 Lambda 表达式的参数声明子句和尾返回类型描述。
    * 如果参数声明子句后没有'mutable'，则函数调用运算符被声明为'const'。
    * 该运算符既不是虚拟的也不是声明为'volatile'。
    * Lambda 声明符中的任何异常规范适用于相应的函数调用运算符。
6. 无捕获的 Lambda 表达式的函数指针转换
    * 无捕获的 Lambda 表达式的闭包类型具有一个公共的非虚拟的'const'转换函数，该函数转换为具有相同参数和返回类型的函数指针。
    * 该转换函数返回的值是一个函数的地址，调用该函数具有与调用闭包类型的函数调用运算符相同的效果。
7. Lambda 表达式的复合语句
    * Lambda 表达式的复合语句生成函数调用运算符的函数体，但在名称查找和类型确定方面，复合语句被视为 Lambda 表达式的一部分。
8. 捕获规则
    * 如果 Lambda 捕获包含默认捕获'&'，则捕获列表中的标识符前不应有'&'。
    * 如果 Lambda 捕获包含默认捕获'='，则捕获列表中的标识符前不应有‘this',且每个标识符前应有'&'。
    * 捕获列表中的标识符或'this'不能重复。
9. 本地 Lambda 表达式
    * 如果 Lambda 表达式的最小封闭作用域是块作用域，则称为本地 Lambda 表达式；否则，不应在 Lambda 引入器中有捕获列表。
10. 捕获列表中的标识符查找
    * 捕获列表中的标识符使用无资格名称查找规则进行查找，每个查找应找到在本地 Lambda 表达式的到达作用域中声明的具有自动存储持续时间的变量。
11. 隐式捕获
    * 如果 Lambda 表达式具有捕获默认值且复合语句 odr-使用'this'或具有自动存储持续时间的变量，则这些实体被隐式捕获。
12. 实体的捕获
    * 如果实体被显式或隐式捕获，则称其捕获。捕获的实体在包含 Lambda 表达式的作用域中被 odr-使用。
13. 默认参数中的 Lambda表达式
    * 出现在默认参数中的 Lambda 表达式不应显式或隐式捕获任何实体。
14. 按值捕获
    * 实体按值捕获，如果它被隐式捕获且捕获默认值为'='，或被显式捕获且捕获不包括'&'。
15. 按引用捕获
    * 实体按引用捕获，如果它被隐式或显式捕获，但不是按值捕获。
16. 嵌套 Lambda 表达式的捕获转换
    * 如果一个 Lambda 表达式 m2 捕获一个实体，而该实体被其直接封闭的 Lambda 表达式 m1 捕获，则 m2 的捕获根据 m1 的捕获方式进行转换。
17. 按值捕获的 id 表达式
    * 每个按值捕获的实体的 id 表达式被转换为对闭包类型中相应未命名数据成员的访问。
18. decltype 操作符中的捕获
    * decltype((x)) 中的 x 被视为对闭包类型中相应数据成员的访问。
19. 闭包类型的构造函数和赋值运算符
    * 与 Lambda 表达式关联的闭包类型具有被删除的默认构造函数和被删除的复制赋值运算符。它具有隐式声明的复制构造函数，并且可能具有隐式声明的移动构造函数。
20. 闭包类型的析构函数
    * 与 Lambda 表达式关联的闭包类型具有隐式声明的析构函数。
21. 捕获的实体的初始化
    * 在评估 Lambda 表达式时，按值捕获的实体用于直接初始化生成的闭包对象的每个相应非静态数据成员。
22. 按引用捕获的生命周期
    * 如果按引用捕获的实体在其生命周期结束后调用函数调用运算符，可能会导致未定义行为。
23. 捕获的包展开
    * 捕获后跟随省略号表示包展开。

>补充：移动构造函数和移动赋值运算符
> * 移动构造函数的实现示例：
> ```C++
> class MyClass{
>   int* data;
> 
>   MyClass(int size): data(new int[size]){ }
>   
>   MyClass(MyClass&& other/* 此处接受一个右值 */) noexcept: data(other.data){
>     // 将其他对象的数据指针置为空，表示资源所有权已经转移  
>     other.data = nullptr;
>   }
>   
>   ~MyClass(){
>       delete[] data;
>   }
>   
>   MyClass(const MyClass&) = delete;
>   MyClass& operator=(const MyClass&) = delete;
> };
> 
> int main(){
>   MyClass obj1(10);   // 调用普通构造函数
>   MyClass obj2 = std::move(obj1); // 调用移动构造函数
>   
>   // 此时 ojb1.data 为 nullptr，资源所有权已经转移给 obj2
> }
> ```
> * 移动构造函数的用途：
>   * 提高性能：在处理大型数据结构或资源密集型对象时，移动构造函数可以显著减少不必要的深度拷贝操作，从而提高程序性能。
>   * 资源管理：在实现资源管理类（如智能指针、容器类）时，移动构造函数可以更高效地管理资源转移。
> * 什么时候会调用移动构造函数：
>   * 当一个临时对象(右值)被用来初始化另一个对象时。
>   * 当返回一个局部对象时，如果启用了返回值优化(ROV)，则可能会调用移动构造函数。
>   * 当使用标准库函数如'std::move'将一个对象转换为右值引用时。
> * 注意事项：
>   * 移动构造函数通常与移动赋值运算符一起实现，以确保对象在移动语义下的正确行为。
>   * 实现移动构造函数时，通常需要禁用复制构造函数和复制赋值运算符，以避免不必要的拷贝。
> 
> * 移动赋值运算符
>   * ```C++
>     #include <iostream>
>     #include <utility>
>   
>     MyClass{
>     public:
>        int* data;
>      
>        MyClass(int size): data(new int[size]) { }
>        MyClass(MyClass&& other) noexcept: data(other.data) {
>           other.data = nullptr;
>        }
>        MyClass& operator=(MyClass&& other) noexcept{
>           if(this != other){
>              // 释放当前对象持有的资源
>              delete[] data;
>              // 获取源对象的资源
>              data = other.data;
>              // 将源对象的资源指针置为空
>              other.data = nullptr;
>           }
>           return *this;
>        }
>        
>        ~MyClass(){
>           delete[] data;   
>        }
>        
>        MyClass(const MyClass&) = delete;
>        MyClass& operator=(const MyClass&) = delete;
>     };
>     
>     int main(){
>       MyClass obj1(10);
>       MyClass obj2(20);
>       
>       obj2 = std::move(obj1);
>       
>       /* 此时obj1.data为nullptr，
>              obj2.data为obj1.data的地址
>        */
>     }
>     ```


## 本章的主要内容
* Lambda 的基本语法
* 如何捕获变量
* 如何捕获成员变量
* Lambda 的返回类型
* 什么是闭包对象
* 如何将 Lambda 转换为函数指针并在 C 风格的 API 中使用
* 什么是 IIFE(立即调用函数表达式)
* 如何从 Lambda 表达式继承以及为什么这会有用


### Lambda 表达式的语法
Lambda表达式的语法结构如下所示：
```C++
[]() specifiers exception attr -> ret { /* code */ }
^ ^  ^                            ^
| |  |                            |
| |  |                            optional: trailing return type
| |  |
| |  optional: mutable, exception specification or noexcept, attributes
| |
| parameter list (optional when no specifiers added)
|
Lambda introducer with an optional capture list
```

关于Lambda 表达式在C++中的明确定义：
1. Lambda 表达式计算的结果
    * 计算Lambda 表达式会生成一个prvalue(pure rvalue，纯右值)临时对象(closure object)。
    * Lambda 表达式不应出现在未计算的操作数中。
    * 闭包对象的行为类似于函数对象。
2. Lambda 表达式类型
    * Lambda 表达式的类型是唯一的、未命名的的非联合类型，成为闭包类型(closure type)。
    * 闭包类型在包含相应 Lambda 表达式的最小块作用域、类作用域或命名空间作用域中声明。

### Lambda 表达式的一些示例 {id="lambda_1"}

1. 最简单的 Lambda 表达式：
    ```C++
    []{};
    ```
    此 Lambda 表达式只需要'[]'和空的'{}'作为函数体。参数列表'()'是可选的。

2. 带有两个参数的 Lambda 表达式：
    ```C++
    [](float f, int a) { return a * f; }
    [](int a, int b) { return a < b; }
    ```
    这是 Lambda 表达式最常见的类型之一，参数通过'()'部分进行传递，这和常规的函数是一样的，不需要指定返回类型，编译器会自动推断。

3. 带尾返回类型的 Lambda 表达式：
    ```C++
    [](MyClass t) -> int { auto a = t.compute(); print(a); return a; };
    ```
    此 Lambda 表达式显式定义了返回值类型，尾返回类型从 C++11 开始也适用于常规函数声明。
   > 补充：尾返回类型
   > * 尾返回类型语法：尾返回类型使用关键字'auto‘和'->'符号。
   > * ```C++
   >    auto functionName(parameters) -> returnType 
   >   ```
   > 示例:
   > 1. 简单函数的尾返回类型
   >   ```C++
   >   auto add(int a, int b) -> int{
   >      return a + b;
   >   }
   >   ```
   > 2. 模板函数的尾返回类型
   >   ```C++
   >   template<typename T, typename U>
   >   auto add(T a, U b) -> decltype(a + b){
   >      return a + b;
   >   }
   >   ```
   > 3. Lambda 表达式的尾返回类型
   >   ```C++
   >   auto lambda = [](int a, int b) -> int {
   >      return a + b;
   >   }
   >   ```

4. 额外的修饰符：
    ```C++
    [x](int a, int b) mutable { ++x; return a < b; };
    [](float param) noexcept { return param * param; };
    [x](int a, int b) mutable noexcept { ++x; return a < b; }; 
    ```
    在此示例中， Lambda 函数体前添加了修饰符：'mutable'(以便可以更改捕获的变量)和'noexcept'，
    第三个 Lambda 表达式中的 'mutable noexcept' 是固定的顺序，若写成 'noexcept mutable' 则不能通过编译，
    当使用了'mutable'和'noexcept‘，则需要在表达式中添加'()'。

5. 关于可选的'()'：
```C++
[x] { std::cout << x; } // 不需要'()'
[x] mutable { ++x; }; // 编译错误，因为mutable存在，故需要'()'
[x]() mutable { ++x; }; // 编译正常
[] noexcept { };    // 编译错误，因为noexcept存在，故需要'()'
[]() noexcept { }; // 编译正常
```
对于后面的C++17和C++20中的'constexpr'和'consteval'也适用。

#### 属性
Lambda 表达式的语法还允许使用以'[[attr_name]]'形式引入的属性。
然而，如果将属性应用于Lambda，那么它应用于调用运算符的类型，而不是运算符本身。
尝试以下表达式：
```C++
auto myLambda = [](int a) [[nodiscard]] { return a * a; };
```
Clang会生成如下错误信息：
```
error: 'nodiscard' attribute cannot be applied to types
```

### 编译展开
Lambda 表达式传给std::for_each的示例：
```C++
#include <iostream>
#include <algortihm>
#include <vector>

int main(){
    // 定义一个functor与下面一般的 Lambda 表达式做对比
    struct{ /* anonymous */
        void operator()(int x) const{
            std::cout << x << 'x';
        }
    } someInstances;
    
    const std::vector<int> v{1, 2, 3, 4, 5};
    std::for_each(v.cbegin(), v.cend(), someInstance);
    std::for_each(v.cbegin(), v.cend(), [](int x){
        std::cout << x << '\n'; 
    });
}
```
在这个示例中，编译器将以下 Lambda 表达式：
```C++
[](int x) { std::cout << x << '\n'; }
```
转换为一个匿名仿函数，简化形式如下：
```C++
struct{
    void operator()(int x){
        std::cout << x << '\n';
    }
}someInstances;
```
编译器具体的展开结果如下：
```C++
#include <iostream>
#include <vector>
#include <algorithm>

int main()
{
  std::vector<int, std::allocator<int> > v = std::vector<int, std::allocator<int> >{std::initializer_list<int>{1, 2, 3, 4, 5}, std::allocator<int>()};
    
  class __lambda_7_39
  {
    public: 
    inline /*constexpr */ void operator()(int x) const
    {
      std::operator<<(std::cout.operator<<(x), '\n');
    }
    
    using retType_7_39 = void (*)(int);
    inline constexpr operator retType_7_39 () const noexcept
    {
      return __invoke;
    };
    
    private: 
    static inline /*constexpr */ void __invoke(int x)
    {
      __lambda_7_39{}.operator()(x);
    }
    
    public: 
    // inline /*constexpr */ __lambda_7_39(__lambda_7_39 &&) noexcept = default;
    // /*constexpr */ __lambda_7_39() = default;
    
  };
  
  std::for_each(v.cbegin(), v.cend(), __lambda_7_39{});
  return 0;
}
```

### Lambda 表达式的类型 {id="lambda_2"}

#### 编译器生成闭包类型
* 编译器为每个 Lambda 表达式生成一个唯一的闭包类型(closure type)，无法预测这个类型名。
* 因此，需要使用'auto'(或'decltype')来推断类型
    ```C++
    auto myLambda = [](int a) -> double { return 2.0 * a; };
    ```

#### 不同的闭包类型
* 即使两个 Lambda 表达式完全相同，它们的类型也是不同的：
    ```C++
    auto firstLam = [](int x) { return x * 2; };
    auto secondLam = [](int x) { return x * 2; };
    ```
  * 编译器必须为每个 Lambda 声明两个独特的未命名类型：
    ```C++
    #include <type_traits>
    int main(){
        const auto firstLam = [](int x) { return x * 2; };
        const auto secondLam = [](int x) { return x * 2; };
        static_assert(!std::is_same(decltype(firstLam), 
                        decltype(secondType)>::value,
                         "must be different!");
    }
    ```
    在编译器眼中，可知这样两个 Lambda 表达式生成的是两个不同的闭包类型：
    ```C++
    #include <type_traits>
    
    int main()
    {
        
      class __lambda_4_25
      {
        public: 
        inline /*constexpr */ int operator()(int x) const
        {
          return x * 2;
        }
        
        using retType_4_25 = int (*)(int);
        inline constexpr operator retType_4_25 () const noexcept
        {
          return __invoke;
        };
        
        private: 
        static inline /*constexpr */ int __invoke(int x)
        {
          return __lambda_4_25{}.operator()(x);
        }
        
        
        public:
        // /*constexpr */ __lambda_4_25() = default;
        
      };
      
      const __lambda_4_25 firstLam = __lambda_4_25{};
        
      class __lambda_5_27
      {
        public: 
        inline /*constexpr */ int operator()(int x) const
        {
          return x * 2;
        }
        
        using retType_5_27 = int (*)(int);
        inline constexpr operator retType_5_27 () const noexcept
        {
          return __invoke;
        };
        
        private: 
        static inline /*constexpr */ int __invoke(int x)
        {
          return __lambda_5_27{}.operator()(x);
        }
        
        
        public:
        // /*constexpr */ __lambda_5_27() = default;
        
      };
      
      const __lambda_5_27 secondLam = __lambda_5_27{};
      /* PASSED: static_assert(!std::integral_constant<bool, false>::value, "must be different"); */
      return 0;
    }
    ```
    > C++17中的改进：  
    > C++17中可以使用没有消息的'static_assert'和辅助变量模板'is_same_v'：
    > ```C++
    > static_assert(std::is_same_v(double, decltype(func(10))>);
    > ```
  * 使用'std::function'
    * 尽管不能确切知道 Lambda 的类型，但可以指定 Lambda 的签名，并将其存储在'std::function'中：
       ```C++
       /* std::function<返回值类型(接受参数类型)> */
       std::function<double(int)> myFunc = [](int a) -> double { return 2.0 * a; };
       ```
    * 需要注意的是，'std::function'是一个重量级对象，因为它需要处理所有可调用对象，其内部机制较复杂，涉及类型转换或内存动态分配，现检查其大小：
       ```C++
       #include <functional>
       #include <iostream>
      
       int main(){
            const auto myLambda = [](int a) noexcept -> double { return 2.0 * a; };
            
            const std::function<double(int)> myFunc = [](int a) noexcept -> double { return 2.0 * a; };
            
            std::cout << "sizeof(myLambda) is " << sizeof(myLambda) << '\n';
            std::cout << "sizeof(myFunc) is " << sizeof(myFunc) << '\n';
            
            return myLambda(10) == myFunc(10);      
       }
       /* output:
        * sizeof(myLambda) is 1
        * sizeof(myFunc) is 64
        */
       ```
        * 由于'myLambda'只是一个无状态的 Lambda，它也是一个空类，没有任何数据成员字段，所以它的大小只有一个字节。
        * 而'std::function'版本要大得多，为64字节(不同的编译器及编译器版本和当前操作系统版本会导致此值不同)，如果可能，依赖‘auto'推断以获得最小的闭包对象。

### 构造函数与复制
1. Lambda 表达式的闭包类型
   * 根据C++规范：
     * Lambda 表达式关联的闭包类型有一个被删除的默认构造函数(default constructor)。
     * 闭包类型还有一个被删除的复制赋值运算符(copy assignment operator)。
2. 不能默认构造和赋值，即进行一般的copying操作  
    由于默认构造函数和复制赋值运算符被禁用，如下代码进行编译会报错：
    ```C++
    auto foo = [&x, &y]() { ++x; ++y; };
    decltype(foo) fooCopy;
    ```
    编译结果：
    ```
    error: no matching constructor for initialization of 'decltype(foo)'
    ```
3. 可以复制 Lambda  
    虽然不能默认构造和赋值 Lambda，但可以复制 Lambda：
    ```C++
    #include <type_traits>
    
    int main(){
        const auto firstLam = [](int x) noexcept { return x * 2; };
        const auto secondLam = firstLam;
        static_assert(std::is_same<decltype<firstLam), decltype(secondLam)>::value, "must be the same!");
    }
   /* verify the same type of firstLam and secondLam */
    ```
4. 捕获变量的复制  
当复制 Lambda 时，其状态也会被复制。这在涉及捕获变量时尤为重要。闭包类型将捕获的变量存储为成员字段，复制 Lambda 会复制这些数据成员字段。

5. C++20 的改进  
在 C+20 中，无状态的 Lambda 将具有默认构造函数和赋值运算符，使其更加灵活和易用。

### Lambda 表达式的调用运算符 {id="lambda_3"}
1. Lambda 表达式的内部实现
   * 在 Lambda 表达式的函数体中编写的代码，会被编译成对应闭包类型的'operator()'函数中的代码。
2. 默认行为
   * 在 C++11 中，'operator()'默认是一个'const'的内联成员函数。
   * Lambda 表达式：
   * ```C++
     auto lam = [](double param) { /* do something 8*/ };
     ```
   * 编译展开后：
   * ```C++
     struct __anonymousLambda{
        inline void operator()(double param) const { /* do something */ }
     };
     ```
     
### 重载
1. Lambda 表达式不支持重载
    * Lambda 表达式无法定义"重载"版本，无法接受不同的参数类型：
    * ```C++
      auto lam = [](double param) { /* do something */ };
      auto lam = [](int param) { /* do something */ };
      ```
      上述代码无法通过编译，因为编译器无法将这两个 Lambda 转换为单个functor，而且不能重定义相同的变量。
2. 使用仿函数实现重载
    * 使用functor实现重载
    * ```C++
      struct MyFunctor{
        inline void operator()(double param) const { /* do something */ };
        inline void operator()(int param) const { /* do something */ };
      };
      ```
    * 'MyFunctor'现在可以处理'double'和'int'类型的参数。

### Lambda 表达式的修饰符和捕获 {id="lambda_4"}

##### 修饰符(modifier)
1. 默认声明：在默认情况下，Lambda 表达式生成的调用运算符('operator()')是'const'内联成员函数。
2. 其他修饰符：在C++11中，可以使用'mutable'和异常规范('noexcept')来修饰调用运算符：  
    Lambda 表达式：
    ```C++
    auto myLambda = [](int a) mutable noexcept { /* do something */ };
    ```
    编译展开后：
    ```C++
    struct __anonymousLambda{
        inline void operator()(int a) noexcept { /* do something */ };
    };
    ```
   
#### 捕获(capture)
* 捕获子句：'[]'不仅引入 Lambda 表达式，还包含捕获的变量列表，称为"捕获子句"。
* 捕获变量：捕获变量会在闭包类型中作为成员变量(非静态static数据成员)存储，可以在 Lambda 体内访问。

#### 捕获方式
* '[&]': 按引用捕获所有在作用域中的自动存储变量。
* '[=]': 按值捕获所有在作用域中的自动存储变量。
* '[x, &y]': 显式按值捕获'x'和按引用捕获'y'。
* '[args...]': 按值捕获模板参数包。
* '[&args...]': 按引用捕获模板参数包。 

捕获示例：
```C++
int x = 2, y = 3;
const auto l1 = []() { return 1; };         // 无捕获
const auto l2 = [=]() { return x; };        // 全部按值捕获
const auto l3 = [&]() { return y; };        // 全部按引用捕获
const auto l4 = [x]() { return x; };        // 仅按值捕获 x
const auto l5 = [&y]() { return y; };       // 仅按引用捕获 y
const auto l6 = [x, &y]() { return x * y; };// x 按值捕获， y 按引用捕获
const auto l7 = [=, &x]() { return x + y; };// 全部按值捕获， x 按引用捕获
const auto l8 = [&, y]() { return x - y; };// 全部按引用捕获， y 按值捕获
```

#### 捕获变量行为
* 按值捕获：变量在 Lambda 定义时被复制。
  * Lambda 表达式：
    ```C++
    std::string str{"Hello Lambda"};
    auto foo = [str]() { std::cout << str << '\n'; }
    foo();
    ```
  * 编译展开后：
    ```C++
    struct _unnamedLambda{
        _unnamedLambda(std::string s): str(s) { }
        void operator()() const {
            std::cout << str << '\n';
        }
        std::string str;
    };
    ```
* 按引用捕获：变量在 Lambda 调用时使用当前值。
    * Lambda 表达式
      ```C++
      int x, y = 1;
      const auto foo = [&x, &y]() noexcept { ++x; ++y; };
      foo()
      ```
    * 编译展开后：
      ```C++
      struct _unnamedLambda{
        _unnamedLambda(int& a, int& b): x(a), y(b) { }
        void operator()() const noexcept{
            ++x; ++y;
        }
        int& x;
        int& y;
      };
      ```
* 注意事项：
  * 捕获模式：虽然'[=]'或'[&]'捕获所有变量很方便，但显式捕获变量更安全，避免意外副作用。
  * 生命周期：C++ 闭包不会延长捕获引用的生命周期，确保在 Lambda 调用时捕获的变量仍然存在。

#### mutable 关键字
在默认情况下，Lambda 表达式的闭包类型的'operator()'被标记为'const'，因此不能在 Lambda 体内修改捕获的变量。
但如果要改变这种行为，就需要在参数列表后添加'mutable‘关键字，这种用法实际上从闭包类型的调用操作符声明中移除了'const'：
Lambda 表达式：
```C++
int x = 1;
auto foo = [x]() mutable { ++x; };
```
编译展开：
```C++
struct __lambda_x1{
    void operator()(){ ++x; }
    int x;
};
```

#### 使用 mutable 拷贝捕获两个变量 {id='mutable_1'}
```C++
#include <iostream>

int main(){
    const auto print = [](const char* str, int x, int y){
        std::cout << str << ": " << x << " " << y << '\n';
    };
    
    int x = 1, y = 1;
    print("in main()", x, y);
    
    auto foo = [x, y, &print]() mutable {
        ++x;
        ++y;
        print("in foo()", x, y);
    };
    
    foo();
    print("in main()", x, y);
}
/* output:
 * in main(): 1 1
 * in foo(): 2 2
 * in main(): 1 1
 */
```
上述代码中，Lambda 表达式通过拷贝捕获了'x'和'y'，并通过引用捕获了'print'。
在'foo'内部，'x'和'y'的值被修改，但这些修改并不影响外部作用域中的原始变量'x'和'y'。

##### 通过引用捕获变量
当通过引用捕获时，Lambda可以在不使用'mutable'的情况下修改引用的值：
```C++
int x = 1;
std::cout << x << '\n';
const auto foo = [&x]() noexcept { ++x; };
foo();
std::cout << x << '\n';

/* output:
 * 1
 * 2
 */
```

##### 关于 mutable 和 const

使用'mutable'时，不能将生成的闭包对象标记为'const'，因为这会阻止调用 Lambda：
```C++
int x = 10;
const auto lam = [x]() mutable { ++x; };
// lam(); 将导致编译出错
```
导致编译出错的原因是不能在'const'对象上调用非'const'成员函数。

#### 捕获变量的实例-调用计数器
例子背景：
Lambda 表达式在需要使用标准库中的算法并改变其默认行为时很有用。在'std::sort'中，通常可以自定义比较函数，
现在，可以在其中引入一个计数器来增强比较器的功能。
代码示例：
```C++
#include <algorithm>
#include <iostream>
#include <vector>

int main(){
    std::vector<int> vec = {0, 5, 2, 9, 7, 6, 1, 3, 4, 8};
    size_t compCounter = 0;
    
    std::sort(vec.begin(), vec.end(), [&compCounter](){
        ++compCounter;
        return a < b;
    });
    
    std::cout << "Number of comparisons: " << compCounter << '\n';
    for(const auto& v: vec) std::cout << v << ',';
}
/* output:
 * Number of comparisons: 54
 * 0,1,2,3,4,5,6,7,8,9,
 */
```

#### 捕获全局变量
在 Lambda 表达式中使用'[=]'按值捕获所有变量，但对于全局变量而言，并不如此：
```C++
#include <iostream>

int global = 10;

int main(){
    std::cout << global << '\n';
    
    auto foo [=]() mutable noexcept { ++global; };
    foo();
    std::cout << global << '\n';
    
    const auto increaseGlobal = []() noexcept { ++global; };
    increaseGlobal();
    std::cout << global << '\n';
    
 /* compile error
  * const auto moreIncreaseGlobal = [global]() noexcept { ++global; };
  * moreIncreaseGlobal();
  * std::cout << global << '\n';
  */
}
/* output:
 * 10
 * 11
 * 12
 */
```
无论使用什么方式捕获，Lambda 表达式始终引用全局对象，而不会创建局部副本。  
最后一个moreIncreaseGlobal()使用Clang会编译失败，说明不能捕获全局变量。

#### 捕获静态变量
与捕获全局变量类似，捕获静态对象时也会遇到同样的问题：
```C++
#include <iostream>

void bar(){
    static int static_int = 10;
    std::cout << static_int << '\n';
    
    auto foo = [=]() mutable noexcept { ++static_int; };
    foo();
    std::cout << static_int << '\n';
    
    const auto increase = []() noexcept { ++static_int; };
    increase();
    std::cout << static_int << '\n';
    
 /* compile error
  * const auto moreIncrease = [static_int]() { ++static_int; };
  * moreIncrease();
  * std::cout << static_int << '\n';
  */
}
/* output:
 * 10
 * 11
 * 12
 */
```
与全局变量相同，静态变量不能按值捕获，使用Clang进行编译会报错，因为不能捕获具有非自动存储持续时间的变量。

#### 捕获类成员变量和`'this'`指针
在类成员函数中捕获成员变量会更加复杂，因为所有数据成员都与'this'指针相关联。  
一个错误示例：
```C++
#include <iostream>

struct Baz{
    void foo(){
        const auto lam = [s]() { std::cout << s; };
        lam();
    }
    std::string s;
};

int main(){
    Baz b;
    b.foo();
}
```
错误原因：不能捕获'Baz::s'并且'this'指针没有捕获。
```C++
struct Baz{
    void foo(){
        const auto lam = [this]() { std::cout << s; };
        lam();
    };
    std::string s;
};
```
通过使用‘this'指针，可以捕获成员变量。

#### 从方法返回 Lambda {id="lambda_5"}
```C++
#include <iostream>

struct Baz{
    std::function<void()> foo(){
        return [=, this] { std::cout << s << '\n'; }
    }
    std::string s;
};

int main(){
    auto f1 = Baz{"abc"}.foo(); /* temporary object */
    auto f2 = Baz{"xyz"}.foo(); /* temporary object */
    f1();
    f2();
    Baz b("ex");
    auto func = b.foo();
    func();
}
/* output:
 *
 *
 * ex
 */
```
'foo()'方法返回一个 Lambda ，该 Lambda 捕获类的成员变量。  
~~以下类似~~：
```C++
struct Bar{
    std::string const& foo() const { return s; };
    std::string s;
};

auto&& f1 = Bar{"abc"}.foo();   // dangling reference
```
~~或者~~：
```C++
std::function<void()> foo(){
    return[s] { std::cout << s << '\n'; };
}
```

上面的代码中'f1'和'f2‘使用的都是临时对象，可能会出现空悬引用(dangling reference)的问题，导致未定义行为。  
捕获'this'在 Lambda 的生命周期可能超过对象本事时可能会出现其他问题，特别是在异步调用(async)和多线程(multithreading)中。

#### 只能移动对象(moveable-only object)
对于一个只能移动的对象(例如‘unique_ptr’)，那么不能将其作为捕获变量按值捕获到 Lambda 表达式中，只能够按引用捕获，但是这并不会转移对象的所有权：
```C++
#include <iostream>
#include <memory>

int main(){
    std::unique_ptr<int> p(new int{10});
    
    // 按值捕获 - 编译错误
    // auto foo = [p]() {};
    
    // 按引用捕获 - 可通过编译，但不转移所有权
    auto foo_ref = [&p]() { std::cout << *p << '\n'; };
    foo_ref();
}
/* output:
 * 10
 */
```
在上面这种情况中，捕获'std::unique_ptr'的唯一方法时按引用捕获，然后，这种方法不能转移指针的所有权。  
解决方法：使用 C++14 中的初始化捕获：通过初始化捕获，可以在 Lambda 表达式中捕获一个移动的对象，从而转移其所有权。
```C++
#include <iostream>
#include <memory>

int main(){
    std::unique_ptr<int> p(new int{10});
    
    // 使用初始化捕获 - 转移所有权
    auto foo = [p = std::move(p)](){
        std::cout << *p << '\n';
    };
    
    foo();
    
    if(!p) std::cout << "p is nullptr after being moved" << '\n';
}
/* output:
 * 10
 * p is nullptr after being moved
 */
```

#### 保持常量性(const preserving)
如果捕获了一个常量变量，其常量性会被保留：
```C++
#include <iostream>
#include <type_traits>

int main(){
    const int x = 10;
    auto foo = [x]() mutable{
        std::cout << std::is_const<decltype(x)>::value << '\n';
        // x = 11; 编译错误
    }
    foo();
}
/* output:
 * 1
 */
```
由上面的代码可知，即使在 Lambda 表达式中使用 'mutable' 关键字，'x'的常量性依然保留，不能被修改。

####  参数包捕获
在捕获子句中，也可以利用可变参数模板(variadic templates)来捕获参数包：
```C++
#include <iostream>
#include <tuple>

template<class... args>
void captureTest(Args... args){
    const auto lambda = [args...]{
        const auto tup = std::make_tuple(args...);
        std::cout << "tuple size: " << std::tuple_size<decltype(tup)>::value << '\n';
        std::cout << "tuple 1st: " << std::get<0>(tup) << '\n';
    };
    lambda();
}

int main(){
    captureTest(1, 2, 3, 4);
    captureTest("Hello Lambda", 10.0f);
}
/* output:
 * tuple size: 4
 * tuple 1st: 1
 * tuple size: 2
 * tuple 1st: Hello Lambda
```
编译展开：
```C++
#include <iostream>
#include <tuple>

template<class ... Args>
void captureTest(Args... args)
{
    
  class __lambda_6_22
  {
    public: 
    inline auto operator()() const
    {
      const auto tup = std::make_tuple(args... );
      (std::operator<<(std::cout, "tuple size:  ") << std::tuple_size<decltype(tup)>::value) << '\n';
      (std::operator<<(std::cout, "tuple 1st: ") << std::get<0>(tup)) << '\n';
    }
    
    private: 
    Args... args;
    
    public:
    __lambda_6_22(const type_parameter_0_0... & _args)
    : args{_args...}
    {}
    
  };
  
  const auto lambda = __lambda_6_22{args};
  lambda();
}

/* First instantiated from: insights.cpp:15 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
void captureTest<int, int, int, int>(int __args0, int __args1, int __args2, int __args3)
{
    
  class __lambda_6_22
  {
    public: 
    inline /*constexpr */ void operator()() const
    {
      const std::tuple<int, int, int, int> tup = std::make_tuple(__args0, __args1, __args2, __args3);
      std::operator<<(std::operator<<(std::cout, "tuple size:  ").operator<<(std::integral_constant<unsigned long, 4>::value), '\n');
      std::operator<<(std::operator<<(std::cout, "tuple 1st: ").operator<<(std::get<0>(tup)), '\n');
    }
    
    private: 
    int __args0;
    int __args1;
    int __args2;
    int __args3;
    
    public:
    __lambda_6_22(int & ___args0, int & ___args1, int & ___args2, int & ___args3)
    : __args0{___args0}
    , __args1{___args1}
    , __args2{___args2}
    , __args3{___args3}
    {}
    
  };
  
  const __lambda_6_22 lambda = __lambda_6_22{__args0, __args1, __args2, __args3};
  lambda.operator()();
}
#endif


/* First instantiated from: insights.cpp:16 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
void captureTest<const char *, float>(const char * __args0, float __args1)
{
    
  class __lambda_6_22
  {
    public: 
    inline /*constexpr */ void operator()() const
    {
      const std::tuple<const char *, float> tup = std::make_tuple(__args0, __args1);
      std::operator<<(std::operator<<(std::cout, "tuple size:  ").operator<<(std::integral_constant<unsigned long, 2>::value), '\n');
      std::operator<<(std::operator<<(std::operator<<(std::cout, "tuple 1st: "), std::get<0>(tup)), '\n');
    }
    
    private: 
    const char * __args0;
    float __args1;
    
    public:
    __lambda_6_22(const char * ___args0, float & ___args1)
    : __args0{___args0}
    , __args1{___args1}
    {}
    
  };
  
  const __lambda_6_22 lambda = __lambda_6_22{__args0, __args1};
  lambda.operator()();
}
#endif

int main()
{
  captureTest(1, 2, 3, 4);
  captureTest("Hello Lambda", 10.0F);
  return 0;
}
```
可以通过可变参数模板在 Lambda 表达式中捕获参数包，捕获的参数包可以存储在'tuple'对象中，便于访问和操作。

#### 返回类型推断
在很多情况下，可以省略 Lambda 表达式的返回类型，从 C++11 开始，编译器能够推断返回类型，只要所有的 return 语句返回的表达式类型相同：
```C++
#include <type_traits>

int main(){
    const auto baz = [](int x) noexcept{
        if(x < 20) return x * 1.1;  // return double
        else return x * 2.1;        // return double
    };
    static_assert(std::is_same(dobule, decltype(baz(10))>::value, "has to be the same);
}
```
在上面的 Lambda 表达式中，两个返回语句的返回类型都为 double，因此编译器可以推断出返回类型。

####  尾置返回类型语法
使用尾置返回类型语法可以显式地指定返回类型：
```C++
#include <iostream>

int main(){
    const auto testSpeedString = [](int speed) noexcept{
        if(speed > 100) return "you're a super fast";
        else return "you're a regular";
    };
    auto str = testSpeedString(100);
    str += " driver";
    std::cout << str;
}
```
上述代码会出现编译错误，因为 const char* 没有 += 操作符，调整后：
```C++
    auto testSpeedString = [](int speed) -> std::string {
        if(speed > 100) return "you're a super fast";
        else return "you're a regular";
    };
    auto str = testSpeedString(100);
    str += " driver";
/* output:
 * you're a regular driver
 */
```
注意，此处在显式设置了返回类型为'std::string'后，需要移除'noexcept'，因为创建了'std::string'可能会抛出异常。  
或者使用'std::string_literals'，然后返回'"you're a regular"s'来表示'std::string'类型。

此处也可实现一个'std::string'的继承类speedString，并实现operator+=的重载：
```C++
#include <iostream>

class speedString: public std::string{
public:
    using std::string::string;  // 继承 std::string 的构造函数
    
    speedString& operator+=(const std::string& rhs){
        std::string::operator+=(rhs);
        return *this;
    }
    
    // 重载 operator+= 以支持 const char* 类型
    speedString& operator+=(const char* rhs){
        std::string::operator+=(rhs);
        return *this;
    }
};

int main(){
    const auto testSpeedString = [](int speed) noexcept -> speedString{
        if(speed > 100) return "you're a super fase";
        else return "you're a regular";
    };
    
    speedString str = testSpeedString(100);
    str += " driver";
    std::cout < str;
}
/* output:
 * you're a regular driver
 */
```

### 函数指针转换(Conversion Function Pointer)
如果 Lambda 表达式没有捕获任何变量，编译器可以将其转换为常规函数指针，标准中描述如下：
> 对于没有捕获的 Lambda 表达式，其闭包类型具有一个公共的、非虚的、非显式的 const 转换函数，
> 该函数转化为为具有与闭包类型的函数调用运算符相同参数和返回类型的函数指针。该转换函数返回的值应该是一个函数的地址，
> 当调用该函数时，其效果与调用闭包类型的函数调用运算符相同。

例如：
```C++
#include <iostream>
void callWith10(void (*bar)(int)){
    bar(10);
}

int main(){
    struct{
        using f_ptr = void(*)(int);
        void operator()(int s) const { return call(s); }
        operator f_ptr() const { return &call; }
    private:
        static void call(int s) { std::cout << s << '\n'; };
    } baz;
    
    callWith10(baz);
    callWith10([](int x) { std::cout << x << '\n'; };
}
```
> 解释：
> 1. 'callWith10()'：'void(*bar)(int)'是一个函数指针，指向返回为类型为'void',参数类型为'int'的函数，'callWith10()'这个函数接受一个这样的函数指针作为参数，然后调用该函数并传入参数‘10’。
> 2. 'using f_ptr = void(*)(int)(等价于 typedef void(*f_ptr)(int));'定义了一个函数指针类型'f_ptr'。
> 3. 'void operator()(int s) const { return call(s); }'重载了'operator()'，使得对象'baz'对象可以像函数一样被调用，并且会调用私有的静态成员函数'call'。
> 4. 'operator f_ptr() const { return & call; }'定义了从结构体类型到函数指针类型的隐式转换操作符，也就是说，这个结构体实例'baz'可以被隐式转换为指向静态成员函数'call'的函数指针。

示例：使用Lambda 调用C库中的'std::qsort'进行反向排序：
```C++
#include <iostream>
#include <cstdlib>

int main(){
    int values[] = {8, 9, 2, 5, 1, 4, 7, 3, 6};
    constexpr size_t numElements = sizeof(values) / sizeof(values[0]);
    
    std::qsort(values, numElements, sizeof(int), 
                [](const void* a, const void* b) noexcept {
        return (*(int*)b - *(int*)a);
    }
    );
    
    for(const auto& val: values) std::cout << val << ", ";
}
/* output:
 * 9, 8, 7, 6, 5, 4, 3, 2, 1
 */
```
上面的代码中，'std::qsort'只接受函数指针作为比较器，编译器可以隐式地将传递地无状态 Lambda 表达式转换为函数指针。
> 总结：
> 1. 无捕获 Lambda 转换为函数指针：
>       * 无捕获地 Lambda 表达式可以转换为与其函数调用运算符具有相同参数和返回类型地函数指针。
>       * 这种转换由编译器自动完成，方便在需要C风格回调地情况下使用。
> 2. 仿函数(functor)显式转换：
>       * 通过定义一个转换操作符，仿函数可以显式地转换为函数指针。
>       * 这在需要传递复杂对象(如仿函数)到需要函数指针地接口时非常有用。

### 一个棘手的案例
案例如下：
```C++
#include <type_traits>

int main(){
    auto funcPtr = +[]{};
    static_assert(std::is_same(decltype(funcPtr), void(*)()>::value);
}
```
编译展开：
```C++
#include <type_traits>

int main()
{
      
  class __lambda_8_19
  {
    public: 
    inline /*constexpr */ void operator()() const
    {
    }
    
    using retType_8_19 = auto (*)() -> void;
    inline constexpr operator retType_8_19 () const noexcept
    {
      return __invoke;
    }
    
    private: 
    static inline /*constexpr */ void __invoke()
    {
      __lambda_8_19{}.operator()();
    }
    
    
    public:
    // /*constexpr */ __lambda_8_19() = default;
    
  };
  
  using FuncPtr_8 = auto (*)() -> void;
  FuncPtr_8 funcPtr = +__lambda_8_19{}.operator __lambda_8_19::retType_8_19();
  /* PASSED: static_assert(std::integral_constant<bool, true>::value); */
  return 0;
}
```
源代码使用了'+'，这是一个一元运算符，这个运算符可以用于指针，因此编译器将无状态的 Lambda 转换为函数指针，然后赋值给'funcPtr'，
相反，如果没有一元运算符'+'，'funcPtr'就只是一个常规的闭包对象，同时'static_assert'也会失效。

在这种情况下，一元操作符'+'和'static_cast‘的作用效果相同，如果不希望编译器创建太多函数实例化时，可以进行如下操作：
```C++
template<typename F>
void call_function(F f){
    f(10);
}

int main(){
    call_function(static_cast<int(*)(int)>([](int x) {
        return x + 2;
    }));
    call_function(static_cast<int(*)(int)>([](int x) {
        return x * 2;
    }));
}
```
在上面的代码中，编译器只需要创建一个'call_function'的实例，因为它只接受一个函数指针'int(*)(int)'，如果去掉了'static_cast'，那么编译器就会为每个 Lambda 创建两个不同类型的'call_function'实例。

### IIEF(Immediately Invoked Expression Function) - 立即调用的函数表达式
直接调用 Lambda 表达式示例：
```C++
#include <iostream>

int main(){
    int x = 1, y = 1;
    [&]() noexcept { ++x; ++y; }();
    std::cout << x << ',' << y;
}
/* output:
 * 2, 2
 */
```
此时， Lambda 表达式创建后没有分配给任何闭包对象，而是直接通过'()'调用。

这样的 Lambda 表达式，在初始化一个复杂的'const'对象时比较有用。
```C++
    const auto val = [](){
    /* do something */
    }();
```
此时，'val'是一个由 Lambda 表达式返回的类型常量值：
```C++
    /* val1 是 int */
    const auto val1 = []() { return 10; }();
    /* val2 是 std::string */
    const auto val2 = []() -> std::string { return "ABC"; }();
```

一个更具体的示例：
使用IIFE作为助手 Lambda 来在函数内部创建一个常量值——IIFE 与 HTML 生成示例：
```C++
#include <iostream>

void Valiate(const std::string&) {}

std::string BuildHred(const std::string& link, 
                        const std::string& text){
    const std::string html = [&link, &text] {
        const std::string inText = text.empty() ? link : text;
        return "<a href=\"" + link + "\">" + inText + "</a>";
    }();
    Validate(html);
    return html;
}

int main(){
    try{
        const auto ahref = BuildHref("ppqwqqq.space", "ppQwQqq");
        std::cout << ahref;
    }
    catch (...) {
        std::cout << "bad format...";
    }
}
```
上面的代码中，'BuildHref'函数，接受两个参数，然后生成一个`'<a></a>'`HTML标签，
基于输入参数，构建'html‘变量，如果'text'不为空，则将其用作内部HTML值，否则使用'link'。
通过使用 IIEF 可以在对多输入参数的条件下使表达式更加简洁：编写一个独立的 Lambda 表达式， 然后将其变量标记为'const'，之后即可将'const'变量传递给'ValidateHTML'。



### 提高 IIEF 代码可读性的方法

1. 避免使用'auto'
    * 明确地指定类型，以便更清楚地看到变量的类型：
    * ```C++
        const bool EnableErrorReporting = [&]() {
            if(HighLevelWarningEnabled()) return true;
            if(HighLevelWarningEnabled()) return UsersWantReporting();
            return false;
        }();
      ```
2. 添加注释：
    * 在'}'后面添加一个注释，指明这是‘IIEF'：
      * ```C++
        const bool EnableErrorReporting = [&]() {
            if (HighLevelWarningEnabled()) return true;
            if (HighLevelWarningEnabled()) return UserWantReporting();
            return false;
        }();    // call it now
        ```

### Lambda 表达式的继承与多态 {id="Lambda_7"}

Lambda 表达式的继承：
由于编译器会将 Lambda 表达式展开为带有'operator()'的仿函数对象，因此可以从这种类型继承：
```C++
#include <iostream>

template<typename Callable>
class ComplexFunctor: public Callable{
public: explicit ComplexFunctor(Callable f): Callable(f) { }
}

template<typename Callable>
ComplexFunctor<Callable> MakeComplexFunctor(Callable&& cal){
    return ComplexFunctor<Callable>(cal);
}

int main(){
    const auto func = MakeComplexFunctor([]() {
        std::cout << "Hello Functor\n";
    });
    func();
}
```
在这个例子中，'ComplexFunctor'类从模板参数'Callable'继承，如果想从 Lambda 继承，
必须添加一些额外的操作，因为无法明确知道闭包类型的确切类型(除非将其封装在'std::function'中)，
因此需要'MakeComplexFunctor'函数来执行模板参数推导并获取 Lambda 闭包类型。

多重 Lambda 继承：
示例： 从两个 Lambda 继承并创建一个重载集：
```C++
#include <iostream>

template<typename TCall, typename UCall>
class SimpleOverLoaded: public TCall, UCall{
public:
    SimpleOverLoaded(TCall tf, UCall uf): TCall(tf), UCall(uf){}
    using TCall::operaotr();
    using UCall::operator();
};

template<typename TCall, typename UCall>
SimpleOverLoaded<TCall, UCall> MakeOverloaded(TCall&& tf, UCall&& uf){
    return SimpleOverLoaded<TCall, UCall>(tf, uf);
}

int main(){
    const auto func = MakeOverloaded(
        [](int) { std::cout << "Int!\n"; },
        [](float) { std::cout << "Float!\n"; }
    );
    func(10);
    func(10.0f);
}
/* output:
 * Int!
 * Float!
 */
```
此处从两个模板进行继承，并显示暴露它们的'operator()'。

#### 为什么需要显式暴露

编译器在寻找正确的重载函数时，要求它们得在同一个作用域中：
```C++
#include<iostream>

struct BaseInt{
    void Func(int) { std::cout << "BaseInt...\n"; };
};

struct BaseDobule{
    void Func(double) { std::cout << "BaseDouble...\n"; }
};

struct Derived: public BaseInt, BaseDouble{
    using BaseInt::Func;
    using BaseFunc::Func;
};

int main(){
    Derived d;
    d.Func(10.0);
}
/* output:
 * BaseDouble...
 */
```
如果没有'using'语句，编译器就会报错，因为'Func()'可以来自'BaseInt'或'BaseDouble'的作用域，编译器无法决定使用哪个。


### 在容器中存储 Lambda 表达式 {id="Lambda_11"}

使用函数指针存储 Lambda:  
Lambda 表达式不能默认创建和赋值，然而利用无状态 Lambda 表达式转换为函数指针的特性，虽然无法直接存储闭包对象，但可以保存从 Lambda 表达式转换出来的函数指针：
```C++
#include <iostream>
#include <vector>

int main(){
    using Func = void(*)(int&);
    std::vector<TFunc> ptrFuncVec;
    
    ptrFuncVec.push_back([](int& x) { std::cout << x << '\n'; });
    prtFuncVec.push_back([](int& x) { x *= 2; });
    ptrFuncVec.push_back(ptrFuncVec[0]);
    
    int x = 10;
    for(const auto& entry: ptrFuncVec) entry(x);
}
/* output:
 * 10
 * 20
 */
```
在'ptrFuncVec'中有三个变量：
1. 输出输入参数的值。
2. 修改该值
3. 是第一个的副本，再次输出该值。

这种方法虽然有效，但仅限于无状态的 Lambda 表达式。

使用std::function封装Lambda：  
为了能够在容器中能够使用其他的状态的 Lambda 表达式，可以使用'std::function'处理，
这样，使其不仅可以处理整数，还可以处理字符串对象：
```C++
#include <iostream>
#include <functional>
#include <algorithm>
#include <vector>

int main(){
    std::vector<std::function<std::string(const std::string&)>> vecFilters;
    
    size_t removedSpaceCounter = 0;
    const auto removeSpaces = [&removedSpaceCounter](const std::string& str){
        std::string tmp;
        std::copy_if(str.begin(), str.end(), std::back_inserter(tmp),
                    [](char ch) { return !isspace(ch); });
        removedSpaceCounter += str.length() - tmp.length();
        return tmp;
    }
    
    const auto makeUpperCase = [](const std::string& str){
        std::string tmp = str;
        std::transform(tmp.begin(), tmp.end(), tmp.begin(),
                        [](unsigned char c) { return std::toupper(c); });
        return tmp;
    };
    
    vecFilters.emplace_back(removeSpaces);
    vecFilters.emplace_back([](const std::string& x){
        return x + " Amazing";
    });
    vecFilters.emplace_back([](const std::string& x){
        return x + " Modern";
    });
    vecFilters.emplace_back([](const std::string& x){
        return x + " C++";
    });
    vecFilters.emplace_back([](const std::string& x){
        return x + " World!";
    });
    vecFilters.emplace_back(makeUpperCase);

    const std::string str = "   H e l l o     ";
    auto temp = str;
    for(const auto& entryFunc: vecFilters) temp = entryFunc(temp);
    std::cout << temp << '\n';
    std::cout << "Removed spaces: " << removedSpaceCounter << '\n';
}
/* output:
 * HELLO AMAZING MODERN C++ WORLD!
 * Removed spaces: 12
 */
```
此代码，在容器中存储'std::function<std::string(const std::string&)>'允许使用任何类型的函数对象，包括捕获变量的 Lambda 表达式。

