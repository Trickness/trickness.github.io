---
layout: post
category : C++11&14/std::thread_local
tagline: ""
tags : [C++,C++11&14]
---  
{% include JB/setup %}

# thread_local variables	    
在C++11标准中添加了一个名为thread_local的关键字，用来声明线程的局部变量，其使用方式如下

## 简单示例    

    #include <iostream>
    #include <memory>
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

我们使用`thread_local`修饰符在全局声明了一个`i`变量，这个`i`变量将被每个新建线程**拷贝**并作为其域内的变量，所以线程1中的`i`变量与main中的`i`变量指向不同地址，所以我们在main中修改了i的值，然而线程中打印出来的`i`的值仍然为0     
`thread_local`修饰后仍然是一个**变量**，我们依旧能够使用取地址操作或者通过引用的方法传递给其他线程修改     
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
`thread_local`变量如果并没有直接使用到，其是否会被初始化与编译器有关，如下面所给出的程序    

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
        // nothing to do
    }
    
    int main(){
        std::thread t1(do_nothing);
        t1.join();
    }
    
你编译运行后可能得到如下输出    

    initialized
    initialized
    deleted
    deleted
    
或者    

    initialized
    deleted
    
或者什么都不输出(于MinGW GCC 4.9测试为这种情况)

## 杂论
thread-local storage 和 `static`(或者说`global`) 存储很类似，每一个线程都将拥有一份这个数据的拷贝，`thread_local`对象的生命周期从线程开始时开始(对于全局变量)，或者首先分配空间。当线程退出的时候对象析构      


## 支持    
 - 在GCC4.9上[Thread-local storage](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2659.htm)的支持已经相当完善，`thread_local`也能好好工作
 - VS2013及更前的版本对[Thread-local storage](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2659.htm)的支持尚不完善[2]，但是据说VS2015已经完全支持[Thread-local storage](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2659.htm)，这点待考证

## 参考资料
[1][what-does-the-thread-local-mean-in-c11](http://stackoverflow.com/questions/11983875/what-does-the-thread-local-mean-in-c11)
[2][(MSVC)支持 C++11/14/17 功能（现代 C++）](https://msdn.microsoft.com/zh-cn/library/hh567368.aspx)
