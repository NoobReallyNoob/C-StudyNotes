第3章 smart pointer

条款18:unique_ptr
1.没有捕获的情况下lamba表达式没有任何额外的存储字节，开销比函数指针还小
(这一节居然讲这个。。)
2.实际上析构器是unique_ptr型别的一部分
3.可以方便地转为shared_ptr
4.unique_ptr实际上有两个版本,单对象unique_ptr<T>,数组unique_ptr<T[]>两个版本
数组版本重载了下标
如:
std::unique_ptr<int32_t[]> m(new int32_t[10]);
m[6] = 6;
注意:数组版本无法通过make_unique创建
5.默认情况下自身的析构器不占内存，实际上可以认为和裸指针开销一致

条款19:shared_ptr
1.自身内存开销是裸指针两倍，还需要一个裸指针维护一个控制块
包括:引用计数,弱引用计数,析构器,分配器等
控制块同样被所有的shared_ptr共享
2.引用计数实际上是原子操作，有一定开销(比实际基本类型大的多)
3.移动构造最快，因为不用修改引用计数(比拷贝构造好)
4.析构器不是shared_ptr型别的一部分
e.g.
    auto p = std::make_unique<std::string>("666");
    auto q = std::make_shared<std::string>("666");

p的类型:std::unique_ptr<std::string, std::default_delete<std::string>>
q的类型:std::shared_ptr<std::string>

如果要自定义析构器:
    auto Foo = [](std::string * str) {
        *str = "666";
        delete str;
    };

    std::unique_ptr<std::string, decltype(Foo)> p(new std::string, Foo);
    std::shared_ptr<std::string> q(new std::string, Foo);

    顺便
    (1).std::unique_ptr<std::string, decltype(Foo)> p;
    不能通过编译，自定义析构器后没有默认函数，必须声明+定义;
    
    (2)自定义析构器也不能通过make_unique/shared来做，只能按老的方法来

5.控制块仅第一次构造是创建,即
(1)make_shared生成控制块
(2)通过unique_ptr/auto_ptr/裸指针 构造会生成控制块
使用同一个裸指针构造shared_ptr,会生成多个控制块，导致重复释放或者野指针等未定义行为
尽量make_shared，不得以比如需要自定义析构器，在构造函数里new裸指针，
别人给的裸指针，需要保存，最好直接拷贝内容。

实际上unique_ptr也一样会导致这样的问题，总之就是多个管家就死翘翘了。

6.enable_shared_from_this
如果一个类希望使用自己(this)通过shared_ptr使用，必须enable_shared_from_this,
因为this是裸指针,构造shared_ptr会产生一个控制块,这样如果你的对象用指针维护，
那么和5中写的一样，多个管家导致未定义行为。

enable_shared_from_this有一个成员函数shared_from_this，
即该函数会检查控制块是否创建过，没有的话直接抛异常，
所以enable_shared_from_this也导致此对象必须用shared_ptr维护

7.shared_ptr没有原始数组版本，只有单对象版本，即没有shared_ptr<T[]>

8.shared_ptr的额外开销
1.如果用默认析构器+分配器，控制块额外会消耗三个字节，主要的开销是分配控制块内存
但是如果用make_shared构造，分配内存的成本实际被并入对象的内存分配之中，
即可以认为只有额外三个字节的开销。
2.引用计数的加减，需要原子化操作，但实际上会映射到单机器指令，单指令操作成本较低。
3.控制块的虚函数开销，仅析构时。

条款20:weak_ptr
weak_ptr的expired不是原子的。
用weak_ptr构造shared_ptr或者.lock()是原子的，前者会抛异常，
最好用.lock()。

使用场合:
1.持有生命周期不属于自己控制的对象(缓存，observer模式)
2.防止循环引用(父子节点如果明确，可以父节点持有unique_ptr，子节点持有裸指针)

条款21:使用std::make_unique和std::make_shared
好处:
1.简洁
2.异常安全
e.g.
void processWidget(std::shared_ptr<Widget>(new Widget), computePriority());
编译器可能顺序:
1).new widget
2).computePriority()
3).std::shared_ptr构造
如果2中抛异常，则widget内存泄露，使用make_shared则可以避免。

不能使用make_unique和make_shared的场合，必须单独用一个表达式构造好智能指针，不可以
直接在参数中构造。
建议写法:
std::shared_ptr<Widget> spw(new Widget, cusDel)
void processWidget(std::move(spw), computePriority());
使用move避免引用计数加减。

3.shared_ptr有控制块，make_shared可以避免分配两次内存，有更好的性能

缺陷:
某些场合无法使用:
1.不支持列表初始化
2.自定义析构器
3.weak_ptr场合,在weak_ptr析构前，控制块都会存在，通过make_shared创建的情况下，
对象本身的内存和控制块生命周期一致，只要有weak_ptr存在，对象本身的内存无法单独释放，
而构造函数创建的则可以使对象先单独释放。

条款22:pimpl
好处:
1.编译速度
2.屏蔽接口细节
3.移动语义友好

缺陷:
1.需要使用std::experimental::propagate_const保证const传递给指针(需要编译器/编译选项支持)

由于unique_ptr要求完整型别(shared_ptr不要求)，析构函数必须显示定义，且在源文件中声明。



    
