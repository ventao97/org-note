
# Table of Contents

1.  [C语言形式可变长参数varargs](#org927172c)
    1.  [用法](#org88d5761)
    2.  [Default Argument Promotion（默认参数提升）](#orgdb3e8c2)
2.  [C++推荐使用的initializer\_list](#orgbf5dd86)
3.  [参考文档](#org1abb56b)



<a id="org927172c"></a>

# C语言形式可变长参数varargs


<a id="org88d5761"></a>

## 用法

比较常见的有printf系列的函数，其函数声明一般如下：

    int add(int count, ...);

在上面的声明中使用&#x2026;代表函数的未知可变参数，其使用方式如下：

    int add(int count, ...)
    {
        int result = 0;
        std::va_list args;
        va_start(args, count);
        for(int i = 0; i < count; i++)
        {
            result += va_arg(args, int);
        }
        va_end(args);
        return result;
    }

由上面的例子可知，其可变参数的展开是通过一组特殊的宏来实现，其原型如下：

    // <cstdarg>
    void va_start( std::va_list ap, parm_n );                // enables access to variadic function arguments 
    T va_arg( std::va_list ap, T );                          // accesses the next variadic function argument 
    void va_copy( std::va_list dest, std::va_list src );     // makes a copy of the variadic function arguments 
    void va_end( std::va_list ap );                          // ends traversal of the variadic function arguments 

这里面唯一有点陌生的就是va\_copy，其作用是复制一份va\_list，有了va\_copy，就能解决多次读参数的问题了，
因为va\_arg每次都会使参数指针后移，从而无法重新读取前面的参数，而va\_copy则是复制一份参数指针，从而使得能够重复读。

**需要注意的是，函数原型里面的&#x2026;符号前面一般都需要有一个先导参数，其作用是用于标识可变参数的起始位置， 并且其一般来说都是用来标识参数的个数时使用的** ，而且该先导参数的类型有一定的限制，根据标准库文档描述，其应当满足如下条件：

    Defined in header <cstdarg>
    void va_start( std::va_list ap, parm_n );
    va_start should be invoked with an instance to a valid va_list object ap before any calls to va_arg.
    If parm_n is declared with reference type or with a type not compatible with the type that results from default argument promotions, the behavior is undefined.

**即其不能为引用类型，并且其类型必须和Default Argument Promotion之后的类型项匹配** ，这里又诞生了一个新概念&#x2013;Default Argument Promotion，下面会讲到

另外，可变长参数函数在重载函数匹配时的优先级是最低的，因此， **除了printf这类用法之外它通常作为SFINAE匹配过程中的默认匹配。**


<a id="orgdb3e8c2"></a>

## Default Argument Promotion（默认参数提升）

所谓的默认参数提升指的是每个通过&#x2026;传递进来的可变参数会按照以下规则进行类型提升：

1.  std::nullptr\_t 转换成void\*
2.  float类型转换为double类型
3.  bool，char，short以及unscoped enum 都会通过整型提升转化为int或更大的整形

一句话来说就是解析的时候不要使用char/short/float等类型传入va\_arg来解析参数，
一个有意思的地方就是有些面试题会考察printf函数的实现，其实就是为了考察varargs的默认参数提升这个知识点。


<a id="orgbf5dd86"></a>

# C++推荐使用的initializer\_list

**和varargs的方式不同，initializer\_list要求参数的类型完全一致** ，并且initializer\_list只提供参数的常迭代器访问，其函数原型多如下：

    void func(std::initializer_list<int> l);

其使用方式如下：

    void func(std::initializer_list<int> l)
    {
        int size = l.size();
        for(auto& num : l)
        {
          //...
        }
    
        for(auto beg = l.begin(); begin != l.end(); begin++)
        {
          //...
        }
    }


<a id="org1abb56b"></a>

# 参考文档

-   [Variadic arguments - cppreference](https://zh.cppreference.com/w/cpp/language/variadic_arguments)

