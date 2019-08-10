第5章 move & forward

条款23:std::move和std::forward
std::move强制转换为右值，std::forward根据自己的T类型来转换，多用于转发

条款24:万能引用和右值引用
类型推导才可能是万能引用，必须是T&&和auto&&。
T&&必须是模块函数，不是是模板类的函数，
模板类的函数同样是强类型，到了函数就不存在类型推导了。

条款25:右值引用move,万能引用forward

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

