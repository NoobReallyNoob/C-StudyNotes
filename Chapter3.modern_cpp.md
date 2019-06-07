第3章 modern c++
第3章中的大部分内容包括using/nullptr这样的，个人是有这个习惯，也知道更好的。
不过学习一下为什么更好也不错。

条款7:
条款7基本废话，我反正没有这么sb的习惯，大括号不乱用就行了。

条款8:
关于NULL，使用nullptr更好的原因在于类型，nullptr仅作为指针，
且能够在类型推导中被正确推断，但是NULL不行。
指针和整型间重载非常危险，所以我一直都不喜欢用!来判断空指针。

条款9:
using和typedef

在模板中，using非常强大
如下，随便来个
template <typename T>
using testList = std::list<T>;

template <typename T>
typedef std::list<T> testList;

typedef这个渣渣这就编不过了。

如果硬要用typedef:
template <typename T>
struct testLists {
	typedef std::list<T> testList;
};

用的时候别人testList 你要testLists::testList。
low爆了。

如果你要把这玩意拿到别的模板类里用，你还得加个typename，否则编译器不知道
这东西是给具体类型，编不过
template <typename T>
class test {
	typename testLists<T>::testList testL;
};

还是这样清爽
template <typename T>
class test {
	testList<T> testL;
};

条款10:优先使用强枚举

首先得说，我很讨厌强枚举。。。
因为很多时候用枚举，就是为了和各种整型转，然后你不要我转，
我往往还得强转。。。。

好处其实我早就知道了，一个是为了避免污染命名空间，一个是为了避免隐式转换带来的污染。

我想说:你让我用强枚举, 就是在用static_cast<color::red>之类污染代码。。。
我还不如constexpr red = 0;constexpr yellow = 0;这样写。。

当前还勉强有给好处，可以前置声明。。不过普通枚举指定了类型也一样啊。。

缺点举例：

using UserInfo = std::tuple<std::string, std::string, size_t>;

enum class InfoFileds {
	uiName,
	uiEmail
};

auto val = std::get<static_cast<size_t>(InfoFileds::uiName)>(info);

真的丑。。

用函数解决 还得靠constexpr,要不常量表达式编不过

constexpr size_t Foo(InfoFileds infoType)
{
	return static_cast<size_t>(infoType);
}

auto val = std::get<Foo(InfoFileds::uiName)>(info);

说实话。。除了需要限定多个输入类型且不用和整型打交道的时候，真不知道你有啥好处。。
纯粹就是污染代码！！！

条款11:delete替代private

好处是能够在编译时被识别，而private要到链接。

最好在public中写delete,好处是能够得到delete相关的编译报错信息而不是private这样的。

delete还有给用处是删除指定参数类型，用于特化。

比如

template <typename T>
bool handlePointer(T* ptr)
{
	return true;
}

template<>
bool handlePointer(char*) = delete;

char* ch = 0;
res = handlePointer(ch);

直接就编不过了，可以干掉一些不想要的类型。

条款12:override和final

重写:
1.基类中是虚函数
2.函数名一致
3.形参类型一致，且const性保持一致
4.如果涉及成员函数引用饰词,饰词完全一致才能override
void foo() &;    左值引用this
void foo() &&;   右值引用this

不加override 可能会引发问题，编译器可能不会进行是否覆盖相关的检查

final:
用于类时，类会被禁止作为基类

成员函数引用饰词，可以避免不必要的拷贝:
类自身是左值是，拷贝往往是合理的，
但有些时候，创建出来是临时对象，是一个右值，没有必要进行多余的拷贝，
通过移动进行处理更好：

比如:
int32_t g_copy_num = 0;

struct TestData {
};

class TestDataFactory {
public:
	TestData& data() &
	{
		g_copy_num++;
		return data_;
	}

	TestData& data() &&
	{
		return std::move(data_);
	}
private:
	TestData data_;
};

TestDataFactory makeDataFactory()
{
	return TestDataFactory();
}

TEST(ConstructTest, Test1) {
	TestDataFactory testDataFactory;

	auto valr = testDataFactory.data();

	EXPECT_EQ(g_copy_num, 1);
	g_copy_num = 0;
	auto vall = TestDataFactory().data();
	EXPECT_EQ(g_copy_num, 0);
}

以上。




