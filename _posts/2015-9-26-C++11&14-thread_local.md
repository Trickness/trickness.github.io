---
layout: post
category : C++11&14/std::thread_local
tagline: ""
tags : [C++11&14, thread_local]
---  
{% include JB/setup %}

# thread_local variables	    
在C++11标准中添加了一个名为thread_local的关键字，用来声明线程的局部变量，其使用方式如下

## 简单示例

```
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
```    

运行输出为`34509`,`30549`之类    

我们使用`thread_local`修饰符在全局声明了一个`i`变量，这个`i`变量将被每个新建线程拷贝并作为其域内的变量，所以线程1中的`i`变量与main中的`i`变量指向不同地址，所以我们在main中修改了i的值，然而线程中打印出来的`i`的值仍然为0     
可是这个`thread_local`变量仍然是一个变量，我们依旧能够使用取地址操作或者通过引用的方法传递给其他线程修改     
如同    
```
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
```    
如果没有出错，程序将输出`42`
