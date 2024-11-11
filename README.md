- [The C++ Programming Language](#the-c-programming-language)
  - [工具](#工具)
  - [字节序 (_Endianness_)](#字节序-endianness)
  - [类型转换](#类型转换)
    - [隐式转换](#隐式转换)
      - [初始化和赋值时](#初始化和赋值时)
      - [使用 `{}` 方式初始化时](#使用--方式初始化时)
      - [表达式中的转换](#表达式中的转换)
        - [整型提升（Integral promotion）](#整型提升integral-promotion)
        - [效验表](#效验表)
      - [传递参数时](#传递参数时)
    - [显式转换](#显式转换)
      - [强制转换](#强制转换)
  - [STL 容器（_Container_）](#stl-容器container)
  - [迭代器（_`iterator`_）](#迭代器iterator)
    - [正向反向迭代器转换](#正向反向迭代器转换)
  - [模板编程](#模板编程)
    - [`template`](#template)
      - [`template function`](#template-function)
      - [`template variable`](#template-variable)
      - [`template class`](#template-class)
      - [`template typename...` and `sizeof...()`](#template-typename-and-sizeof)
    - [类型推导](#类型推导)
    - [SFINAE 机制（_Substitution Failure Is Not An Error_）](#sfinae-机制substitution-failure-is-not-an-error)
      - [成员探测技术（_Detecting Members_）](#成员探测技术detecting-members)
    - [`concept` and `requires` C++20](#concept-and-requires-c20)
      - [`concept`](#concept)
      - [`requires`](#requires)
      - [使用](#使用)
  - [万能引用 \& 完美转发](#万能引用--完美转发)
    - [移动语义](#移动语义)
    - [万能引用（_Universal Reference_）](#万能引用universal-reference)
  - [指针数组 和 数组指针](#指针数组-和-数组指针)
    - [指针数组（_Array of Pointers_）](#指针数组array-of-pointers)
    - [数组指针（_Pointer to an Array_）](#数组指针pointer-to-an-array)
  - [`new` and `delete`](#new-and-delete)
    - [`new`](#new)
    - [placement `new`](#placement-new)
    - [`delete`](#delete)
    - [一些猜测](#一些猜测)
  - [编译期求值](#编译期求值)
    - [`constexpr`](#constexpr)
    - [`consteval`C++20](#constevalc20)
  - [`class`, `struct` and `union`](#class-struct-and-union)
    - [`public`, `private` and `protected`](#public-private-and-protected)
      - [用于访问域](#用于访问域)
      - [用于继承](#用于继承)
    - [^内存对齐](#内存对齐)
      - [默认对齐系数](#默认对齐系数)
      - [修改对齐系数](#修改对齐系数)
    - [^位域 (_bit field_)](#位域-bit-field)
      - [位域对齐规则](#位域对齐规则)
      - [位域字节序](#位域字节序)
      - [成员函数修饰符](#成员函数修饰符)

# The C++ Programming Language
---

## 工具

|NAME|FUNCTION|PATH|
|:---:|---|---|
|objdump|反汇编||
|c++filt|解析 C++ 符号||
|dumpbin|遍历 DLL 库|C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\bin\Hostx64\x86\dumpbin.exe|

## 字节序 (_Endianness_)

* 大端序 (_big-endian_)：
  * 高位字节在低地址，低位字节在高地址；
  * 更符合一般阅读习惯；
  * 一般用于网络传输和文件储存；

$0\text{x}12345678$ 的大端序：
|||||
|-|-|-|-|
|$0\text{x}12$|$0\text{x}34$|$0\text{x}56$|$0\text{x}78$|

$Low\Longrightarrow{High}$

* 小端序 (_little-endian_)：
  * 高位字节在高地址，低位字节在低地址；
  * 更符合计算机处理顺序；
  * 一般用于计算机内部处理；

> 计算都是从低位开始的，电路先处理低位字节效率会比较高。那么如果计算机采用小端序，就可以从低地址先读取到低位字节。

$0\text{x}12345678$ 的小端序：
|||||
|-|-|-|-|
|$0\text{x}78$|$0\text{x}56$|$0\text{x}34$|$0\text{x}12$|

$Low\Longrightarrow{High}$

* 使用 `std::endian`<sup>C++20</sup> 判断是大端序还是小端序：
```cpp
if constexpr (std::endian::native == std::endian::big)
{
  // Big-endian
}
else if constexpr (std::endian::native == std::endian::little)
{
  // Little-endian
}
```

$\to$ [位域字节序](#位域字节序)

## 类型转换

### 隐式转换

C++ 自动类型转换发生在以下情况：

+ 将一种算术类型的值赋给另一种算术类型的变量时；
+ 表达式中包含不同的类型时；
+ 将参数传递给函数时；

#### 初始化和赋值时

值被转换为接收变量的类型。如下：
```cpp
short sVal = 520;
int nVal = sVal;
```
$sVal$ 赋值给 $nVal$ 前，$sVal$ 先被转换为 `int` 类型。

_注意：将一个值赋给取值范围更小的类型时，如果是整型，将会发生截断，只复制右边的字节，如果是浮点型，则结果将是不确定的。_

#### 使用 `{}` 方式初始化时

> 列表初始化 (list-initialization)

相比直接使用 `=` 赋值，对类型转换要求更严格，不允许**缩窄（narrowing）**。

#### 表达式中的转换

##### 整型提升（Integral promotion）

在计算表达式时，取值范围小于 `int` 的类型将自动转为 `int` 再进行计算。

`false` 转为 $0$；
`true` 转为 $1$；

`wchar_t` 将提升为下列第一个宽度足够 `wchar_t` 取值范围的类型，`int`，`unsigned int`，`long`，`unsigned long`；

##### 效验表

1. 如果有一个操作数的类型是 `long double`，则另一个操作数转换为 `long double`；
2. 否则，如果有一个操作数的类型时 `double`，则将另一个操作数转换为 `double`；
3. 否则，如果有一个操作数的类型时 `float`，则将另一个操作数转换为 `float`；
4. 否则，说明操作数都是整型，因此执行 整型提升；
5. 如果两个操作数都是有符号或无符号的，且其中一个操作数的级别比另一个低，则转换为级别高的类型；
6. 如果一个操作数是有符号的，一个是无符号的，且无符号操作数的级别比有符号操作数高，则将有符号操作数转换为无符号操作数所属的类型；
7. 否则，如果有符号类型可表示无符号类型的所有可能取值，则将无符号操作数转换为有符号操作数所属的类型；
8. 否则，将两个操作数都转换为无符号版本；

+ 有符号整型级别：`long long` $>$ `long` $>$ `int` $>$ `short` $>$ `signed char`。
+ 无符号整型级别：同上。
+ `char` $=$ `signed char` $=$ `unsigned char`。
+ `bool` 级别最低。
+ `wchar_t`、`wchar16_t`、`wchar32_t` 与其底层类型相同。

#### 传递参数时

传递参数时的类型转换通常由 C++ 函数原型控制。

### 显式转换

C 风格类型转换：
```cpp
(type-id)expression
```

C++ 类型转换（推荐），类似调用构造函数：
```cpp
type-id(expression)
```

#### 强制转换

* `static_cast` 比普通的类型拥有更强的可读性，不能去掉 `cv` 属性限定符。
* `const_cast` 去除指针或引用的 `const` 属性限定符。
* `reinterpret_cast` 改变指针或引用的类型、将指针或引用转换为一个足够长度的整形、将整型转换为指针或引用类型。
* `dynamic_cast` 用于类层次间下行转换，在运行时处理，进行类型检查。
  * 在类层次间进行上行转换时，`dynamic_cast` 和 `static_cast` 的效果是一样的，下行转换时额外进行检查。
  * 基类中一定要有虚函数，否则编译不通过。存在虚函数，就说明它有想要让基类指针或引用指向派生类对象的情况，此时转换才有意义。
  * 转换成功返回指向类的指针或引用，转换失败指针返回 `nullptr`，引用则抛出 `std::bad_cast`。
  * 不能用于内置的基本数据类型的强制转换。

类层次间的 **上行转换**：子类指针指向父类指针（一般不会出问题）。

类层次间的 **下行转换**：即将父类指针转化子类指针。

## STL 容器（_Container_）

|Container|Template parameters|Functions|
|---|---|---|
|`std::array`|`typename` $value\_type$ <br> `std::size_t` $size$|数组|
|`std::vector`|`typename` $value\_type$ <br> `typename` $allocator=$ `std::allocator<value_type>`|自动扩容的矢量数组|
|`std::valarray`|`typename` $value\_type$|值数组，用于高效的数学运算|
|`initializer_list` <sup>C++11</sup>|`typename` $element\_type$|初始化列表|
|`std::bitset`|`std::size_t` $size$|位集，二进制数组|
|`std::function`|`typename` $return\_type$ <br> `typename...` $params\_type$|封装一个函数|
|`std::stack`|`typename` $value\_type$ <br> `typename` $sequence=$ `std::deque<value_type>`|栈，后进先出|
|`std::queue`|`typename` $value\_type$ <br> `typename` $sequence=$ `std::deque<value_type>`|队列，先进先出|
|`std::deque`|`typename` $value\_type$ <br> `typename` $allocator=$ `std::allocator<value_type>`|双向队列|
|`std::priority_queue`|`typename` $value\_type$ <br> `typename` $sequence=$ `std::vector<value_type>` <br> `typename` $compare=$ `std::less<value_type>`|优先队列，大根堆|
|`std::list`|`typename` $value\_type$ <br> `typename` $allocator=$ `std::allocator<value_type>`|双向链表|
|`std::forward_list`|`typename` $value\_type$ <br> `typename` $allocator=$ `std::allocator<value_type>`|单向链表|
|`std::string`||字符串|
|`std::string_view` <sup>C++17</sup>||字符串只读视图|
|`std::stringstream`||字符串流|
|`std::istringstream`||字符串输入流|
|`std::ostringstream`||字符串输出流|
|`std::set`|`typename` $key\_type$ <br> `typename` $compare=$ `std::less<key_type>` <br> `typename` $allocator=$ `std::allocator<key_type>`|集合，红黑树|
|`std::multiset`|`typename` $key\_type$ <br> `typename` $compare=$ `std::less<key_type>` <br> `typename` $allocator=$ `std::allocator<key_type>`|多重集，红黑树|
|`std::unordered_set` <sup>C++11</sup>|`typename` $value\_type$ <br> `typename` $hash=$ `std::hash<value_type>` <br> `typename` $pred=$ `std::equal_to<value_type>` <br> `typename` $allocator=$ `std::allocator<value_type>`|无序集合，哈希表|
|`std::unordered_multiset`|`typename` $value\_type$ <br> `typename` $hash=$ `std::hash<value_type>` <br> `typename` $pred=$ `std::equal_to<value_type>` <br> `typename` $allocator=$ `std::allocator<value_type>`|无序多重集，哈希表|
|`std::map`|`typename` $key\_type$ <br> `typename` $value\_type$ <br> `typename` $compare=$ `std::less<key_type>` <br> `typename` $allocator=$ `std::allocator<std::pair<const key_type, value_type>>`|映射表，红黑树|
|`std::multimap`|`typename` $key\_type$ <br> `typename` $value\_type$ <br> `typename` $compare=$ `std::less<key_type>` <br> `typename` $allocator=$ `std::allocator<std::pair<const key_type, value_type>>`|多重映射表，红黑树|
|`std::unordered_map` <sup>C++11</sup>|`typename` $key\_type$ <br> `typename` $value\_type$ <br> `typename` $hash=$ `std::hash<key_type>` <br> `typename` $pred=$ `std::equal_to<key_type>` <br> `typename` $allocator=$ `std::allocator<std::pair<const key_type, value_type>>`|无序映射表，哈希表|
|`std::unordered_multimap`|`typename` $key_type$ <br> `typename` $value\_type$ <br> `typename` $hash=$ `std::hash<key_type>` <br> `typename` $pred=$ `std::equal_to<key_type>` <br> `typename` $allocator=$ `std::allocator<std::pair<const key_type, value_type>>`|无序多重映射表，哈希表|
|`std::pair`|`typename` $first\_type$ <br> `typename` $second\_type$|二元组|
|`std::tuple` <sup>C++11</sup>|`typename...` $elements\_type$|多元组|
|`std::variant` <sup>C++17</sup>|`typename...` $types$|共用体，变体|
|`std::optional` <sup>C++17</sup>|`typename` $value\_type$|带额外标识符判断是否有值|
|`span` <sup>C++20</sup>|`typename` $value\_type$ <br> `std::size_t` $extent$|连续对象序列的轻量级视图|
|`std::any` <sup>C++17</sup>||存储满足构造函数要求的类型的实例，或没有值|

## 迭代器（_`iterator`_）

### 正向反向迭代器转换
`iterator` $\to$ `reverse_iterator`
```cpp
iterator it = iterator();
auto rit = reverse_iterator(it + 1);
```

`reverse_iterator` $\leftarrow$ `iterator`
```cpp
auto it = rit.base() - 1;
```

## 模板编程

### `template`

#### `template function`
```cpp
template<typename T>
void print(const T& value)
{
    std::cout << value << std::endl;
}
```

#### `template variable`
```cpp
template<typename T>
T value = static_cast<T>(520.1314);
```
> Note: `value<int>` and `value<double>` is **different**.

#### `template class`
```cpp
template<typename T>
class A
{
public:

  using value_type = T;
  
  // ...
}
```

#### `template typename...` and `sizeof...()`
[Parameter pack (since C++11)](https://en.cppreference.com/w/cpp/language/parameter_pack)

`sizeof...()` 获取参数包的大小（数量）

### 类型推导

+ `decltype` 获取标识符类型：

_被括号包裹的标识符表达式或类成员访问表达式将被推导为左值。_

```cpp
int a;

// int b = a;
decltype(a) b = a;

// int& b = a;
decltype((a)) c = a;
```

+ `auto` 类型占位符：

_占位符 `auto` 本身会丢失类型限定符，`decltype(auto)`<sup>C++14</sup> 可以解决这个问题。_

_自 C++14 起，函数的返回值（如果可以自动推导）也可以是 `auto`。_

不能进行返回类型推导的几种常见情况：

1. 函数在形式上有多个 `return`，且这些 `return` 返回的类型不相同（即使可以隐式类型转换）；
2. 函数返回初始化列表；
3. 要求 `auto*`，但是返回值不是指针；
4. 虚函数不能使用返回类型推导；
5. 在递归前不存在可推导的返回语句，例如：

Error:
```cpp
auto func(int x)
{
  return x == 0 ? 0 : func(x - 1);
}
```

Accept:
```cpp
auto func(int x)
{
	if (x == 0) return 0;
	return func(x - 1);
}
```

### SFINAE 机制（_Substitution Failure Is Not An Error_）

> 模板实例化时类型推演失败不会报错，而是当成一个语言特性。利用模板通用定义和模板特化，对于不同的模板参数给予不同的实现。

[C++ 模板学习：一个例子搞懂 SFINAE](https://zhuanlan.zhihu.com/p/688452093)

* `std::enable_if`
```cpp
// Define a member typedef @c type only if a boolean constant is true.
template<bool, typename _Tp = void>
  struct enable_if
  { };

// Partial specialization for true.
template<typename _Tp>
  struct enable_if<true, _Tp>
  { typedef _Tp type; };

template< bool B, class T = void >
  using enable_if_t = typename enable_if<B,T>::type;
```
> 这个元函数是一个 **类型萃取** (_Type Trait_) ，在模板的第一个参数 `B` 为 `true` 的时候，它的 `type` 成员会返回一个类型：
>> 如果没有第二个模板参数，返回类型是默认的 `void`，  
>> 否则，返回的是其 **第二个** 参数的类型。

> 如果参数 `B` 为 `false` 的时候，其成员类型是未定义的，根据 **_SFINAE_** 机制，编译期会忽略包含该 `std::enable_if_t<>` 表达式的模板。  
> 意思就是我们可以设立一个条件，当条件满足的时候，就让编译器生成这个函数实例，否则忽略它。

* `std::void_t`
```cpp
// A metafunction that always yields void, used for detecting valid types.
template<typename ...>
  using void_t = void;
```
> 这个元函数的意思是它可以接收任意个类型作为模板参数，编译器会在实参替换阶段检查看你输入的每个类型能否被替换成功，如果替换失败，编译器会忽略这个模板。

#### 成员探测技术（_Detecting Members_）

判断两个类型是否都具有 `iterator` 类型，和 `being()`、`end()` 成员函数

```cpp
// 普通版本
template<typename, typename, typename = std::void_t<>>
  struct is_container_type : std::false_type{ };

// 偏特化版本
template<typename T1, typename T2>
  struct is_container_type<T1, T2, std::void_t<typename T1::iterator, typename T2::iterator, decltype( std::declval<T1>().begin(), std::declval<T2>().begin())>>
 : std::true_type{ };
```

### `concept` and `requires` <sup>C++20</sup>
> 用于简化替代 [SFINAE](#sfinae-机制substitution-failure-is-not-an-error)

#### `concept`

+ 语法：
```cpp
template<template-parameter-list>
concept concept-name = constraint-expression;
```
+ 示例：
```cpp
template <class T>
concept Integral = std::is_integral<T>::value;

template <class T>
concept SignedIntegral = Integral<T> && std::is_signed<T>::value;

template <class T>
concept UnsignedIntegral = Integral<T> && !SignedIntegral<T>;
```

#### `requires`
+ 示例：
```cpp
template<typename T>
concept has_iterator = requires(T v)
{
    T::iterator;
    v.begin();
    v.end();
};
```

#### 使用

* `requires` 限制 `typename T`；
```cpp
template<typaname T>
requires std::integral<T>
void print(T n)
{
    std::cout << n << std::endl;
}
// or
template<typaname T>
void print(T n) requires std::integral<T>
{
    std::cout << n << std::endl;
}
```

* `concept` 限制 `auto`；
```cpp
void print(const std::integral<T> auto& n)
{
    std::cout << n << std::endl;
}
```

* `concept` 替代 `typename`；
```cpp
template<std::integral T>
void print(T n)
{
    std::cout << n << std::endl;
}
```

* `concept` 替代 `typename...`；
```cpp
template<std::integral ...T>
T sum(T ...args)
{
    return (... + args);
}
```

## 万能引用 & 完美转发

> https://theonegis.github.io/cxx/C-%E4%B8%AD%E7%9A%84%E4%B8%87%E8%83%BD%E5%BC%95%E7%94%A8%E5%92%8C%E5%AE%8C%E7%BE%8E%E8%BD%AC%E5%8F%91/

* `std::ref(value)` 返回 $value$ 的引用，其实是一个 `reference_wrapper`；
* `std::cref(value)` 返回 `const` $value$ 的引用；
* `std::declval<T>()` 返回 `T` 类型的右值引用，不管是否有没有默认构造函数或该类型不可以创建对象（不会创建对象，可以用于抽象基类）;
* `std::as_const(value)` 返回 $value$ 的 `const` 版本的引用，确保对象不会被更改；

### 移动语义

- `std::move(t)`
  -  返回 $t$ 的右值引用。
- `std::move_if_noexcept(t)`
  -  如果 $t$ 的移动构造函数是 `noexcept`，返回 $t$ 的右值引用，否则，返回 $t$ 的左值引用。
  -    ```cpp
        class Test
        {
            Test(Test&&) noexcept;
            Test& operator=(Test&&) noexcept;
        };
       ```
  - [Why?](https://stackoverflow.com/questions/28598488/stdmove-if-noexcept-calls-copy-assignment-even-though-move-assignment-is-noexc)
  - 为什么需要这个函数？
    - 例如在 `std::vector` 内部有使用到这个函数。
    - 如果元素类型的移动构造函数是 `noexcept`，则进行 **移动**，否则进行 **复制**。
    - 如果调用移动构造函数时有可能发生异常，则移动操作将会 **终止**，原有数据被破坏，导致失去 _数据完整性_，为了避免这一情况，将会采用复制的方式，这会损失一定的性能。
    - 尽量让移动构造函数是 `noexcept`。

> `std::move()` 和 `std::move_if_noexcept()` 不负责移动资源，仅返回右值引用，右值在 C++ 中标识一个 **临时对象**，移动资源由对象本身自定义，_对于临时对象，一般采用 **浅拷贝**，否则，采用 **深拷贝**_。

```cpp
class IntVector
{
public:

  // 复制构造函数
  IntVector(const IntVector& __obj)
  {
    (*this) = __obj;
  }

  // 移动构造函数
  IntVector(IntVector&& __obj) noexcept
  {
    (*this) = std::forward<IntVector>(__obj);
  }

  // 析构函数，释放资源，注意要判空
  ~IntVector()
  {
    if (_M_data != nullptr)
    {
      delete[] _M_data;
    }
  }

  // 复制，采用深拷贝
  IntVector& operator=(const IntVector& __obj)
  {
    _M_size = __obj._M_size;
    _M_data = new int[_M_size];

    ::memcpy(_M_data, __obj._M_data, _M_size * sizeof(int));
    
    return *this;
  }

  // 移动，采用浅拷贝
  IntVector& operator=(IntVector&& __obj) noexcept
  {
    _M_size = __obj._M_size;
    _M_data = __obj._M_data;

    // 赋空旧对象
    __obj._M_size = 0;
    __obj._M_data = nullptr;
  }

private:
  int* _M_data;
  int  _M_size;
};
```

注意：浅拷贝后应当对旧对象赋空，临时对象也将调用析构函数，将会释放资源。

### 万能引用（_Universal Reference_）

- `T&`  左值引用；
- `T&&` 右值引用（万能引用）；
- 右值可以是右值也可以是左值，所以 `T&&` 可以引用右值也可以引用左值，所以也可以叫做 _万能引用_；
- 左值优先匹配 `T&` 的重载，没有则会匹配 `T&&` 的重载；
- 右值不能匹配 `T&` 的重载，可以匹配 `const T&` 的重载；
-  ```cpp
    /**
    * n 引用了一个右值，但是 n 是一个左值
    * 因为它有了名称，有了地址
    */
    int&& n = 520;
   ```

### 折叠引用（_Universal Collapse_）

C++中不允许对引用再进行引用。

所有的折叠引用最终都代表一个引用，要么是左值引用，要么是右值引用。

规则是：如果任一引用为左值引用，则结果为左值引用。否则（即两个都是右值引用），结果为右值引用。

### 完美转发（_Perfect Forwarding_）

```cpp
template<typename T>
void func(T&)
{
    std::cout << "func(T&) is called." << std::endl;
}

template<typename T>
void func(T&&)
{
    std::cout << "func(T&&) is called." << std::endl;
}

template<typename T>
void call(T&& __t)
{
    func(__t);
}

int main(void)
{
    int var = 520;
    call(var);

    call(1314);

    return 0;
}
```

```out
> func(T&) is called.
> func(T&) is called.
```

> 在 `call()` 函数中，$\_\_t$ 引用了一个 **左值** 或 **右值**，但是 $\_\_t$ 是一个 **左值**，所以 **_始终_** 调用 `func(T&)`
> 
> 为了解决如上问题，可以使用完美转发 `std::forward()`，完美转发可以保留参数的左值或右值属性

```cpp
template<typename _Tp>
constexpr _Tp&&
forward(typename std::remove_reference<_Tp>::type& __t) noexcept
{
    return static_cast<_Tp&&>(__t);
}

template<typename _Tp>
constexpr _Tp&&
forward(typename std::remove_reference<_Tp>::type&& __t) noexcept
{
    static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument substituting _Tp must not be an lvalue reference type");
    return static_cast<_Tp&&>(__t);
}
```
> 根据 [折叠引用](#折叠引用universal-collapse) 中的规则，两个右值则为右值，否则为左值

重写 `call()` 函数为：
```cpp
template<typename T>
void call(T&& __t)
{
    func(std::forward(__t));
}
```

## 指针数组 和 数组指针

### 指针数组（_Array of Pointers_）
- $p$ 是一个包含 `int` 类型的指针作为其元素的数组。
```cpp
int* p[];
```

### 数组指针（_Pointer to an Array_）
- $p$ 是一个指向 `int` 数组的指针。
```cpp
int (*p)[];
```

`()` 优先级 $>$ `[]` 优先级，所以 $p$ 首先是一个指针，然后是指向一个数组。

## `new` and `delete`

### `new`
> https://learn.microsoft.com/zh-cn/cpp/cpp/new-operator-cpp?view=msvc-170

以下示例先分配然后释放一个二维字符数组，数组的大小为 $dim\times{10}$。 分配多维数组时，除第一个维度外的所有维度都必须是计算结果为 _正值的常数表达式_。 最左侧的数组维度可以是任何计算结果为正值的表达式。 使用 `new` 运算符分配数组时，第一个维度可以为 $0$；`new` 运算符返回一个唯一指针。

```cpp
char (*pchar)[10] = new char[dim][10];
delete[] pchar;
```

* `type-id` 不能包含 `const`、`volatile`、类声明或枚举声明。
* `new` 运算符不分配引用类型，因为它们不是对象。
* `new` 运算符不能用于分配函数，但可用于分配指向函数的指针。 下面的示例为返回整数的函数分配然后释放一个包含 $7$ 个指针的数组。
  *    ```cpp
        int (**p)() = new(int(*[7])());
        delete p;
       ```

### placement `new`

https://www.geeksforgeeks.org/placement-new-operator-cpp/

在指定的首地址上调用构造函数。

注意：不会申请内存。

```cpp
type* ptr = new (addr) construct-type(args-list)
```
> `addr` 是一个预先分配的内存地址  。
> `ptr` 与 `addr` 指向相同的地址，但已做类型转换。

例如：
```cpp
int main(void)
{
    // buffer on stack
    unsigned char buf[sizeof(int)];

    // placement new in buf 
    int *pInt = new (buf) int(3); 

    std::cout << *pInt << std::endl;
    std::cout << buf << ' ' << pInt << std::endl;

    return 0;
}
```

Output:
```
3
0x69fed8 0x69fed8
```

### `delete`

- `delete` 释放 `new` 申请的内存，调用 **第一个** 析构函数；
- `delete[]` 释放 `new[]` 申请的内存，调用 **所有** 析构函数；


### 一些猜测
---

|字节数量|元素个数（可选）|...|数据段|
|---|---|---|---|
|$ByteSize$|$ElementCount$|...|$Data$|

`new` 返回 $Data$ 的首地址

`delete` 和 `delete[]` 都会释放 $ByteSize$ 大小的内存

- 非必须调用析构函数的类型
  - `new` 和 `new[]` 都不会保存 $ElementCount$
  - `delete` 和 `delete[]` 都不会调用 _析构_ 函数
  - `new` 后 进行 `delete[]` 不会发生 _偏移量越界_，因为 `delete[]` 知道是 **非必须调用析构函数的类型**，所以它知道没有保存 $ElementCount$

- 必须调用析构函数的类型
  - 调用 `new`
    - `new` 不保存 $ElementCount$
    - `delete` 仅调用 **第一个** 元素的析构函数
    - `delete[]` 发生 _偏移量越界_，访问到错误的 $ByteSize$
  
  - 调用 `new[]`
    - `new[]` 保存 $ElementCount$
    - `delete` 仅调用 **第一个** 元素的析构函数
    - `delete[]` 调用 $ElementCount$ 个元素的析构函数

参考：
1. [C++ 中 `delete` 和 `delete[]` 的深层区别](https://www.foxzzz.com/cpp-array-delete/)
2. [重新了解 `delete[]`](https://segmentfault.com/a/1190000016128411)
---

## 编译期求值

### `constexpr`

* `constexpr if`<sup>C++17</sup>

类似 `#if`，但 `constexpr if` 舍弃的语句会受到完整的检查。

* `constexpr function`

函数可以在编译期求值（不是一定）。

* `constexpr variable`

真正的常量。

### `consteval`<sup>C++20</sup>

指定函数是立即函数（_immediate function_），即每次调用该函数必须产生编译时常量。

* `consteval if`<sup>C++23</sup>

检查函数调用是否出现在常量求值的场合。

```cpp
if consteval
{
  // 在编译期执行
}
else
{
  // 在运行期执行
}
```
或
```cpp
if not consteval
{
  // 在运行期执行
}
else
{
  // 在编译期执行
}
```

* `std::is_constant_evaluated`

检查函数调用是否出现在常量求值的场合。若对调用的求值出现在明显常量求值的表达式或类型转换的求值中，则返回 `true`，否则返回 `false`。

可能的实现：
```cpp
if consteval { return true } else { return false; }
```

* `consteval function`

类似 [`constexpr function`](#constexpr)，但保证在编译期求值。

## `class`, `struct` and `union`

在 C++ 中，`class` 和 `struct` 唯一的区别就是：_`class` 默认 `private`，`struct` 默认 `public`_。

结构体内存地址增长方向：
$$
\begin{align*}
Lo&w\\
\Downarrow\\
Hi&gh
\end{align*}
$$

### `public`, `private` and `protected`

#### 用于访问域
||内部访问|外部访问|派生类访问|友元类或友元函数访问|
|-|:-:|:-:|:-:|:-:|
|`public`|YES|YES|YES|YES|
|`private`|YES|NO|NO|NO|
|`protected`|YES|NO|YES|YES|

#### 用于继承
不同的继承方式将影响基类的访问域:
||base `public`|base `private`|base `protected`|
|-|:-:|:-:|:-:|
|`public`|unchange|unchange|unchange|
|`private`|change to `private`|unchange|change to `private`|
|`protected`|change to `protected`|unchange|unchange|

### [^](https://zhxilin.github.io/post/tech_stack/1_programming_language/modern_cpp/language_base/memory_alignment/)内存对齐

对齐单位 = $\min($ 最大类型长度 $,$ 编译期对齐系数 $)$。

最大类型的长度是结构体中的所有结构体展开后计算的，以下示例代码中 `struct A` 的最大类型是 `long long`，而不是 `struct B`：
```cpp
struct A
{
    struct B
    {
        int a;
        long long b;
    };

    int c;
};
```

#### 默认对齐系数
- $32$ 位下通常默认对齐系数为 $4$；
- $64$ 位下通常默认对齐系数为 $8$；

#### 修改对齐系数
修改对齐系数为 $n\in{\left\{2^x\right\},x\geq{0}}$：
```cpp
#pragma pack(n)
```
取消自定义的对齐系数，恢复为默认值：
```cpp
#pragma pack()
```

C++ 不允许一个对象的大小为 $0$，不同对象的地址不能具有相同的地址。

这是因为 `new` 需要分配不同的内存地址，不能分配内存大小为 $0$ 的空间，避免除以 `sizeof(T)` 引发 **除零异常**。

所以一个没有数据成员的空类，编译器会为其分配 $1$ 字节的内存空间，即空类的大小为 $1$。

空基类被继承后，如果派生类有自己的数据成员，那么空基类这 $1$ 个字节 **不会** 添加到派生类中。

### [^](https://learn.microsoft.com/zh-cn/cpp/cpp/cpp-bit-fields?view=msvc-170)位域 (_bit field_)

可以限制成员变量（整型）使用的二进制位数。

格式：
```cpp
struct bit_filed
{
    type [field_name] : witdh;
};
```

位域相当于一个类型的变量拆分为若干个段，如下，$a$ 和 $b$ 使用同一个 `unsigned int`，前提是 $a$ 和 $b$ 类型一致且 $a$ 和 $b$ 的占位和不大于该类型的占位和。

```cpp
struct test
{
  unsigned int a : 2;
  unsigned int b : 30;
};
```

+ **未命名位域** 字段可以用来作填充或调整位置，但不能使用。

+ 宽度为 $0$ 的 **未命名位域** 强制将下一个位域与下一个类型边界对齐，其中类型是成员的类型。

+ 如果溢出当前字边界，将移动到下一对齐边界。

+ 如果是有符号类型（`signed`），该字段第一个二进制位同样作为符号位。

+ 保留溢出取模等正常整型的特性。

+ 位域是 _左值_，但是位域 **不能取地址**，也不能将一个非 `const` 引用绑定到位域上。但是 `const` 引用可以绑定到位域上，此时会产生一个 **临时变量**。

+ 位域的初始化与普通结构体的初始化方法相同。

+ 位域清空可以使用 _地址重映射_ 或 _`union` 共享内存_ 或 _`memset`_ 等操作。

#### 位域对齐规则

+ 同一个位域字段必须存储在同一个存储单元中，不可以跨多个存储单元，一个存储单元容纳不下时，应从下一单元开始存放；

+ 整个结构的大小是位域字段类型中最宽大小的整数倍；

+ 若相邻的位域字段类型相同，且位宽之和小于等于类型的大小，则后面的字段紧邻前一个字段排列，直到容纳不下；

+ 若相邻的位域字段类型相同，但位宽之和大于类型的大小，则后面的字段从下一个存储单元开始，其偏移量为类型大小的整数倍；

+ 若相邻的位域字段类型不同，`gcc` 编译器采用压缩排列，尽量让后面的字段紧邻前一个字段排列，直到容纳不下，而 `msvc` 编译器不压缩排列；

+ 位域字段之间如果穿插非位域字段，则穿插的非位域字段不参与压缩排列，连续的位域字段还是会压缩排列；

#### 位域字节序

计算机底层一般使用小端序，所以位域字节序一般也是采用小端序。

```cpp
struct Data
{
  unsigned char fin     : 1;
  unsigned char rsv     : 3;
  unsigned char opcode  : 4;

  unsigned char mask    : 1;
  unsigned char payload : 7;
} data; // sizeof(Data) == 2
```

内存布局，每一格表示一个二进制位：
|$opcode_3$|$opcode_2$|$opcode_1$|$opcode_0$|$rev_2$|$rev_1$|$rsv_0$|$fin_0$|$payload_6$|$payload_5$|$payload_4$|$payload_3$|$payload_2$|$payload_1$|$payload_0$|$mask_0$|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|||||||||||||||||
$$Low\Longrightarrow{High}$$

$fin$ 和 $rsv$ 和 $opcode$ 拼成一个 `unsigned char`，$mask$ 和 $payload$ 拼成一个 `unsigned char`，采用小端序，反转字节序。

$data:=0\text{x}9883(0\text{b}10011000\text{'}10000011)$

由于是小端序，$0\text{x}83(0\text{b}10000011)$ 赋值给 $opcode\text{,}rev\text{,}fin$ 段，$0\text{x}98(0\text{b}10011000)$ 赋值给 $payload\text{,}mask$ 段。

如下：

$$
\begin{align*}
opcode&:=0\text{b}1000\to{8}\\
rsv&:=0\text{b}001\to{1}\\
fin&:=0\text{b}1\to{1}\\
\\
payload&:=0\text{b}1001100\to{76}\\
mask&:=0\text{b}0\to{0}
\end{align*}
$$

|$opcode_3$|$opcode_2$|$opcode_1$|$opcode_0$|$rev_2$|$rev_1$|$rsv_0$|$fin_0$|$payload_6$|$payload_5$|$payload_4$|$payload_3$|$payload_2$|$payload_1$|$payload_0$|$mask_0$|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|$1$|$0$|$0$|$0$|$0$|$0$|$1$|$1$|$1$|$0$|$0$|$1$|$1$|$0$|$0$|$0$|
$$Low\Longrightarrow{High}$$

#### 成员函数修饰符

- `const`
- `volatile`
- `&`
- `&&`
- `noexcept`
- `final`
- `override`
- `try`