第1章 类型推导
拷贝：忽视引用和const
引用/指针：携带引用和const
万能引用：能区分左值引用和右值引用

数组：会被推导成指针

auto:和template的区别在于列表初始化，auto会将{a, b, c}推导为{initializer_list}，template不能
C++典型的错误设计，但是用auto作为返回值或lamba形参中使用时，实际上是template型推导，不会产生这样的问题

decltype:
返回值型别尾序语法，使用尾序返回值->decltype()

example:
template<typename T, typename Index>
auto Foo(T& container, Index i)->decltype(container[i])
{
    return c[i];
}
这个auto实际上不涉及类型推导，这样可以后推导返回值类型，避免编译器报错
(返回值类型依赖于形参，需要先推断形参类型)

不过现在感觉没什么卵用了，auto一样随便推导返回值

书上说的最合理的写法
template<typename T, typename Index>
decltype(auto) Foo(T& container, Index i)
{
    return c[i];
}

实际上这样也是随便玩，一回事
template<typename T, typename Index>
auto& Foo(T& container, Index i)
{
    return c[i];
}

这东西有什么卵用？？？

typeid typeinfo
并不完全准确 无法正确反应引用和const的推导