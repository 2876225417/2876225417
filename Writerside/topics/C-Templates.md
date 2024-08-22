# C++ Templates

## 1. 函数模板 ##

### 1.1 初探函数模板 ###

#### 1.1.1 定义模板 ####
此处定义了一个返回两数中较大者的函数
```C++
template<typename T>
inline T const& max(T const& a, T const& b){
    return a > b ? a : b;
}
```
上述代码中的typename为模板参数，T则为模板参数名。  
T可以换成其他任意的合法字符，如：MyType。

模板参数typename可以替换为class，在当前语义环境下class与typename的效果是相同的。  
注意：引入类型参数时，typename不能替换为struct。
```C++
template<class T>
inline T const& max(T const& a, T const& b){
    return a > b ? a : b;
}
```

#### 1.1.2 使用模板 ####
此处为模板函数的使用：
```C++
#include <iostream>

template<typename T>
inline T const& max(T const& a, T const& b){
    using Type = decltype(a);
    std::cout << "Type:" << typeid(std::type_identity_t<T>).name() << '\n';
    return a > b ? a : b;
}

int main(){
    printf("Result: %lf\n", ::max(3.0, 4.4));
    printf("Result: %d\n", ::max(3, 5));
    printf("Result: %c\n", ::max('a', 'b'));
    printf("Result: %s\n", ::max("abcf", "abcd"));
    return 0;
}

/* output:
 * Type:d
 * Result: 4.400000
 * Type:i
 * Result: 5
 * Type:c
 * Result: b
 * Type:A5_c
 * Result: abcf
 */
```
注：使用域限定符::是为了保证调用的是全局命名空间内的max()(即自己定义的max函数)，
因为在标准库中也存在一个std::max()，为避免二义性所以添加域限定符。  

由输出结果可知，类型T被转换为了传入参数的类型。  
因为对于模板函数的每次调用，模板参数T都被实例化(instantiation)为了对应的类型。  
operator<(以及其他运算符) 适用于内置类型。如果需要对自定义类进行比较运算，则需要手动重载该运算符。  
```C++
    MyClass t1, t2;
    std::cout << "Result: " << ::max(t1, t2);
    /*
     * Compiling Error
     */
```
由此可知，模板实例化会经历两次编译：
1. 实例化之前，检查模板本身是否存在语法错误。
2. 实例化期间，检查模板函数调用是否有效(对指定的类型是否存在重载函数)。

### 1.2 实参的推导(deduction) ###
模板参数T的类型由调用函数的实参决定。   
当实参类型不同时，例如：
```C++
    template<typename T>    
    inline T const& max(T const& a, T const& b);
    max('a', 23);   // Error
    max(23, 43);    // Ok
```
这里的T不允许进行自动的类型转换，所以当接受的实参类型不同时就会导致编译器报错。  
处理上述问题的三种方法：
1. 对实参进行强制类型转换，使其类型相互匹配：
   ```C++
    max(static_cast<int>('a'), 32);
    max<double>(3, 3.4)
   ```
2. 显示指定或限定T的类型
3. 指定两个参数可以具有不同类型的参数