---
layout: post
category : C/C++
tagline: "Biger than biger"
tags : [C/C++, Integer]
---
{% include JB/setup %}

## C/C++ 中的二进制量     

我们在C/C++中常常使用的二进制类型是 _int_ ，然而 _int_ 并不是一个准确的数据类型，在不同编译器上， _int_ __的大小可能不同__，_int_ __只保证有最少为16位，最大可能为64位__    

> *在网络编程以及其他对整数大小和上限要求严格的地方，请避免使用 _int_ 或者 _long_ 这类数据类型*    

## 实际编程中常用的二进制类型    

在实际编程中，我们常用的二进制数据类型有如下(当然也有与其配套的unsigned xxx_t)    

> int8_t     
> int16_t    
> int32_t   
> int64_t   

他们分别占用 1,2,4,8 Byte   

## int128    

偶然得知这个世界上竟然出现了 **__int128_t** 这种神奇的数据类型，不禁各处搜索了下，[GCC Docs 6.8](https://gcc.gnu.org/onlinedocs/gcc/_005f_005fint128.html) 中有如下说明:    

> As an extension the integer scalar type __int128 is supported for targets which have an integer mode wide enough to hold 128 bits. Simply write __int128 for a signed 128-bit integer, or unsigned __int128 for an unsigned 128-bit integer. There is no support in GCC for expressing an integer constant of type __int128 for targets with long long integer less than 128 bits wide. [1]


大概来说就是看上去gcc是支持 _int128_ 和 _unsigned int128_ 的，但是又没指明是否作为标准.    

我们便在不同平台做了如下测试:    

### Debian x86_64 with gcc version 4.9.3 (Debian 4.9.3-3) -std=c++11           

`$ cat test.cpp`       

    #include <iostream>
    #include <cstdint>
    using namespace std;
    
    int main(){
        cout << "intmax_t's size = " <<sizeof(intmax_t) << endl;
        __int128 s = 0xFFFFFFFFFFFFFFFF;
        s = s * s;
        cout << "__int128's size = " << sizeof(s) << endl;
    #ifdef _INT128_DEFINED
        cout << "INT128 has been defined" << endl;
    #else
        cout << "INT128 hasn't been defined" << endl;
    #endif
    }

使用 `$ g++ -o test test.cpp -std=c++11` 编译成功，运行得到的结果却让我有点迷茫    

`$ ./test`

    intmax_t's size = 8      
    __int128's size = 16      
    INT128 hasn't been defined      


奇怪的是    
**_INT128_DEFINED 这个宏并没有被定义，但是 *__int128* 这个数据类型却通过了编译，且却实占用了16Byte的内存空间**(哪位朋友知道的邮件我)      


我们要来确认下它**是否保存了16Byte大小的数字**(最高约为3.4*10^38)     

    #include <iostream>
    #include <cstdint>
    
    using namespace std;
    
    void print_uint128(unsigned __int128 n){
        if (n == 0) {
        return;
        }
        print_uint128(n/10);
        putchar(n%10+0x30);
    }
    
    int main(){
        unsigned __int128 s = 0xFFFFFFFFFFFFFFFF;
        s = s*s;
        print_uint128(s);
        cout << endl;
    } 

(部分代码引用自[[2]](http://stackoverflow.com/questions/11656241/how-to-print-uint128-t-number-using-gcc))    

得到结果    

`340282366920938463426481119284349108225`    

*__int128*确实保存了2的128次方大小的数据        


### Windows X86_64 with gcc 4.9.2 (i686-posix-dwarf-rev1, Built by MinGW-W64 project)      

在MinGW上，事情变得比较奇怪了，我这里实验发现 `_INT128_DEFINED` 这个宏是被定义了的，然而却找不到任何类似于 *__int128* *int128_t* *int128* 之类的数据类型的存在    

这里也就没法演示了     

## 其他    
1. 在网上找到的许多文章都是使用的 *int128_t* 这类，但是在我Debian系统上并没有这种数据类型的定义，只存在 *__int128* 和 *unsigned __int128* 这两种数据类型    
2. 值得注意的是，我并没有直接
`s = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF`, 而是 `s = 0xFFFFFFFFFFFFFFFF;s = s * s` 因为在我实验中，直接赋值 `0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF` 只能得到 __*uint64_t* 所能容纳的最大值__,这说明在编译器中对于 *__int128* 的  **支持尚不完全**  

## 结束语    

1. 据说这个支持在 GCC 44.4.6以后[[3]](http://4byte.cn/question/1000920/int128-on-linux-for-intel-compiler.html)（待考证）就已经存在，不过希望大家多下去考证
2. 如果能在 _NOIP_ 上能用 *__int128* 这种二进制量的话，老师再也不用担心我的高精度题目了    


## 参考资料
[1][how to print __uint128_t number using gcc?](http://stackoverflow.com/questions/11656241/how-to-print-uint128-t-number-using-gcc)      
[2][6.8 128-bit Integers](https://gcc.gnu.org/onlinedocs/gcc/_005f_005fint128.html)
[3][int128 on Linux for Intel compiler](http://4byte.cn/question/1000920/int128-on-linux-for-intel-compiler.html)
