第5章 lambda

条款31:避免默认捕获
说白了还是引用的东东被释放造成的问题。
需要注意的是this的传递，如果类的生命周期超出lambda的生命周期，
考虑拷贝需要的成员变量甚至拷贝这个类的对象。

条款32:初始化捕获
auto func = [str = std::move(str)] { std::cout << str << std::endl; };
emmm,原来还能这么玩，又学到了。

跟在大括号内move的区别在于，上面的在创建的lambda的时候就move走了，放在大括号内
则会在lambda执行的时候move。好处不多说了，临时变量的克星。。

条款33：decltype+std::forward转发

auto foo = [] (auto&& param) {
    func(std::forward<decltype(param)>(param));
};

条款34:使用lambda替代bind
不用bind，占位符太蠢了。。。

