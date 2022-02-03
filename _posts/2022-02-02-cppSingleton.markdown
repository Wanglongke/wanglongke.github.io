---
layout: post
title:  cpp设计模式-单例模式
date:   2022-01-29 18:05:55 +0300
image:  10.jpg
tags:   设计模式
description: 单例设计模式的两种实现方式                                                                                                                                                            
---

单例模式是程序设计模式的一种，这种设计模式下程序对象在程序的整个生存周期内只存在一个，下面是两种不同的实现方式。

## 方法A
首先构造函数要私有化，防止从外界构造多余的对象。这里将单例对象放在 ```getInstance()``` 内部，只有当需要一个对象的时候调用该函数才会申请一块空间构造此对象。
```cpp
class A {

public:
    static A* getInstance() {
        static A* instance_ = nullptr;
        if (instance_ == nullptr) {
            instance_ = new A();
        }
        return instance_;
    }
    void init() {
        std::cout << "A init" << std::endl;
    }
    ~A() {
    }
private:
    A() {
        std::cout << "A Construct" << std::endl;
    }
    A(const A& a) {

    }
};
```
## 方法B
同样构造函数私有化，单例对象做为成员变量位于 ```class B``` 内部且私有化。虽然使用 ```private``` 关键字修饰该单例对象，但是由于其是静态变量，可以在外部进行初始化。
```cpp
class B {

public:
    static B* getInstance() {
        if (instance_m == nullptr) {
            instance_m = new B();
        }
        return instance_m;
    }
    void init() {
        std::cout << "B init" << std::endl;
    }
private:
    static B* instance_m;
    B() {
        std::cout << "B Construct" << std::endl;
    }
    B(const B& b) {

    }

};
```
静态成员变量不接受 ```private``` 关键字修饰，因此可用以下方式赋予初值空指针 ```nullptr```, 对象的申请构造将在调用函数 ```getInstance()``` 时进行。
```cpp
B* B::instance_m = nullptr;
```

## 主函数测试

```cpp
int main(int argc, char* argv[]) {
    A* a = A::getInstance();
    B* b = B::getInstance();
    a->init();
    b->init();
}
```
命令台输出结果如下：
```
A Construct
B Construct
A init
B init
```