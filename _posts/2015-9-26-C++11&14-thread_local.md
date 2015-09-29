---
layout: post
category : C++11&14/std::thread_local
tagline: ""
tags : [C++,C++11&14]
---  
{% include JB/setup %}

# thread_local variables	    

## 提出背景:    

在C和早期版本的C＋＋[C++98／03]中，编写多线程程序时，线程没有自己的全局变量，这是因为在POSIX线程模型下，线程没有自己的全局变量，所以全局变量只能共享父进程的。当多线程函数越来越复杂，功能越来越多的时候，就必须要将必要的资源不断的通过参数的形式传递到不同的函数中，相当麻烦（当然我承认贸然使用全局变量是个非常冒进且不合规范的行为）。所以在C＋＋11中就提出了thread_local这个变量修饰，用于解决线程没有自己全局变量的问题。

## 简单示例    

    #include <iostream>
    #include <thread>
    
    thread_local int i = 0;
    
    int func(int val){
        i = val;
        i = i + 2;
        std::cout << i;
    }
    
    int func2(){
        std::cout << i;
    }
    
    int main(){
        i = 9;
        std::thread t1(func,1);
        std::thread t2(func,2);
        std::thread t3(func,3);
        std::thread t4(func2);
    
        t1.join();
        t2.join();
        t3.join();
        t4.join();
    
        std::cout << i << std::endl;
        return 0;
    }

运行输出为`34509`,`30549`之类    

我们使用`thread_local`修饰符在全局声明了一个`i`变量
 - `i`变量将被每个新建线程**拷贝**并作为其**域内**的**全局变量**
 - 线程1中的`i`变量与main中的`i`变量指向不同地址。
 - `thread_local`修饰后仍然是一个**变量**，我们依旧能够使用**取地址操作**者通过**引用**的方法传递给其他线程修改     

如同:    

    thread_local int i=0;
    
    void thread_func(int*p){
        *p=42;
    }
    
    int main(){
        i=9;
        std::thread t(thread_func,&i);
        t.join();
        std::cout<<i<<std::endl;
    }

如果没有出错，程序将输出`42`

## 议题--类的使用    

`thread_local` 变量在第一次使用时初始化，如果变量(类)没有被使用。此变量(类)将不会被初始化[求证][4]    

    #include <iostream>
    #include <thread>
    
    struct my_class{
        my_class(){
            std::cout<< "initialized" << std::endl;
        }
        ~my_class(){
            std::cout << "deleted" << std::endl;
        }
        int i;
    };
    
    thread_local my_class ss;
    
    void do_nothing(){
        
    }
    
    int main(){
        std::thread t1(do_nothing);
        t1.join();
    }
    
我编译执行后没有得到任何输出，即`main`和`t1`中的`ss`都没有被初始化。[测试编译器GCC4.9 MinGW32 & Debian X86_64]

## 杂论
thread-local storage 和 `static`(或者说`global`) 存储很类似，每一个线程都将拥有一份这个数据的拷贝，`thread_local`对象的生命周期从线程开始时开始(对于全局变量)，或者首先分配空间。当线程退出的时候对象析构      


## 支持    

 - 在GCC4.9上[Thread-local storage](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2659.htm)的支持已经相当完善，`thread_local`也能好好工作
 - VS2013及更前的版本对[Thread-local storage](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2659.htm)的支持尚不完善[2]，但是据说VS2015已经完全支持[Thread-local storage](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2659.htm)，这点待考证    

## 结束语    

在通常情况下并不建议使用全局变量，或者说使用全局变量的时候就通常说明你的程序设计得有问题。少用全局变量能够增强你程序(函数)的可重入性。毕竟是用全局变量将可能导致一些难以预见的问题(通常在多线程同步中)，并且难以模块化。所以在程序(函数)设计初期就请人这思考构建方式，做到少用全局变量，多用局部变量。

## 参考资料
[1][what-does-the-thread-local-mean-in-c11](http://stackoverflow.com/questions/11983875/what-does-the-thread-local-mean-in-c11)
[2][(MSVC)支持 C++11/14/17 功能（现代 C++）](https://msdn.microsoft.com/zh-cn/library/hh567368.aspx)
