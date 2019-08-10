第5章 move & forward

条款23:std::move和std::forward
std::move强制转换为右值，std::forward根据自己的T类型来转换，多用于转发

条款24:万能引用和右值引用
类型推导才可能是万能引用，必须是T&&和auto&&。
T&&必须是模块函数，不是是模板类的函数，
模板类的函数同样是强类型，到了函数就不存在类型推导了。

条款25:右值引用move,万能引用forward
1.
class Widget {
public:

void setName(const std::string& name)
{
    this->name = name;
}

void setName(std::string&& name)
{
    this->name = std::move(name);
}

private:
    std::string name;
};

可以优化为:

class Widget {
public:

template<typename T>
void setName(T&& name)
{
    this->name = std::forward<T>(name);
}

private:
    std::string name;
};

这样写还有一个好处，即如果用一个char* 
如"666"构造，实际上会进行转发，只进行调用一次构造函数。
而上面会调用一个构造，一次移动构造，一次析构(临时变量析构)。

2.
对右值引用的最后一次使用用move，万能引用的最后一次使用用forward

3.
RVO需要满足两个条件
1)局部变量类型和函数返回值类型完全一致,包括引用类型
2)返回局部变量本身

如果有RVO，不对临时变量使用move或者forward，
RVO即使不能避免复制，也会生成一个右值，即进行move；
没有RVO，可以使用move或者forward。

条款26 避免重载万能引用
万能引用作为完美转发构造函数甚至会劫持拷贝构造和移动构造

条款27
额 frtti关了，基本和我无缘

条款28 引用折叠
万能引用实际上是通过引用折叠来实现,
原始的引用有任一引用为左值结果则折叠为左值, 否则为右值

条款29
移动不一定更快，如
1.std::array 实际上是移出所有的成员再移入，而其它的容器实际上
往往只需要复制一个指针(内容在堆上，栈上持有一个指向堆内存的指针)
2.小型字符串优化，SSO，小的字符串实际存储在std::string对象内的某个缓冲区，
而不是在堆上分配char*，导致拷贝可能更快

条款30 万能引用失败
1.列表初始化
2.0/NULL作为空指针(推断为整型)
3.static const/constexpr 基本类型 直接声明 没有定义
可能失败，因为这种static成员实际上没有被分配内存，编译器直接塞到所有出现的地方。。
4.重载的函数，模板函数，作为万能引用的参数
无法找到具体是哪个实例，所以会失败
5.位域
不可能有到比特的指针，因此不可能







