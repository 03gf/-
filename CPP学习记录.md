# 形参带默认值的函数

``` cpp
int sum(int a = 10,int b) //从右向左压栈
```

# 内联函数
当调用函数的开销大于函数本身时，推荐使用内联函数，一般都会在函数调用处直接展开函数，不会生成函数符号，没有函数调用开销，一般函数内容都十分简单
``` cpp
inline int sum(int a,int b) return a + b; //像这种简单函数就推荐使用内联函数
```

# 函数重载
c++代码在产生函数符号的时候，函数符号是由函数名和参数列表类型决定的，而c只是由函数名决定，所以c不支持函数重载
一组函数称得上重载，要求在一个作用域上
```cpp
bool compare(int a,int b) {
    std::cout << "compare int int" <<std::endl;
    return a > b;
}
bool compare(double a,double b){
    std::cout << "compare double double" <<std::endl;
    return a > b;
}
int main()
{
    bool compare(double a, double b);//由于优先调用main函数内部的，而不会去全局域里面去找，导致此时会报错，参数不匹配，
    compare(1,2);
    return 0;
}
```

当c++调用c的代码的时候
``` cpp
extern "C"{
    //此处放c函数声明
    int sum(int a,int b) {
        return a + b;
    }
}   
```
c调用cpp代码时，将exter"c"写到cpp函数定义哪里，将其包裹住，c编译器不认识extern"c"

# const
在cpp中const修饰的是离他最近的类型
例如
```cpp
const int* p; //const 修饰的是int，不能修改指针p所指向的值，可以修改p所指向的对象
int* const p; //const 修饰的是int*，不能修改指针p所指向的内存，p只能指向之前的地址，但是可以修改那片地址的值
int* = const int *; //错
int* = int* const; //对 
```
# cpp面向对象
1. 类中实现的成员方法，会被自动处理成inline内联函数。
2. 定义的class本身没有大小，只有实例化的对象才有内存大小，大小取决于里面的成员变量。
3. 堆上的对象需要我们自己分配内存，自己主动调用delete函数进行内存回收，否则会造成内存泄露。
   .data上的对象：程序开始时调用构造，结束时析构
   heap上的对象：new时构造，delete时析构
   stack上的对象：定义时构造，离开作用域时析构。 
## 浅拷贝和深拷贝
浅拷贝是进行内存数据的拷贝，如果对象占用了外部资源，那么浅拷贝就会出现问题，那么此时就需要我们自己写一个拷贝构造函数

## 四种类型转换
```cpp
const_cast //  转类型，只能去掉const指针或引用转换
static_cast //  转类型，只能转换编译器认为是安全的类型转换，（没有如何联系的转换会被认为是不安全的）
dynamic_cast //  转类型，主要用于类的继承结构，能够在运行时检查类型转换的安全性，可以支持rtti类型的上下转换
reinterpret_cast //  转类型，类型转换c语言的强制类型转换
```
## 三条对象优化准则
1. 函数调用时，对象优先考虑引用传递，不要按照值传递。
2. 函数返回时，应该优先返回一个临时对象，而不用返回一个定义过的对象。
3. 接受返回值是对象的函数调用时，优先按初始化的方式去接受， 而不是赋值的方式去接受。
## explicit关键字
explicit关键字可以防止隐式类型转换，只能用于构造函数。
## 智能指针
智能指针是一种特殊的指针，它可以自动管理对象的生命周期，避免内存泄漏和悬空指针的问题。
智能指针的实现原理是使用引用计数，当引用计数为0时，自动释放对象的内存。
智能指针分为两种：
1. unique_ptr：独占式智能指针，不能被复制，只能被移动。
2. shared_ptr：共享式智能指针，多个智能指针可以指向同一个对象，当引用计数为0时，自动释放对象的内存。
3. weak_ptr：弱引用智能指针，不增加对象的引用计数，主要用于解决shared_ptr的循环引用问题。
我们一般定义对象时，使用shared_ptr智能指针,在引用对象时,使用weak_ptr智能指针。
注意：weak_ptr智能指针不能直接访问对象的成员，需要先转换为shared_ptr智能指针。
```cpp
weak_ptr<int> wp;
shared_ptr<int> sp = wp.lock();//这样将weak_ptr智能指针转换为shared_ptr智能指针
if(sp){
    // 访问对象的成员
}
```
## 锁
锁是一种用于保护共享资源的机制，防止多个线程同时访问导致数据不一致的问题。
锁的实现原理是使用互斥量（mutex），当一个线程获取到锁时，其他线程就需要等待，直到该线程释放锁。
锁的类型分为两种：
1. 互斥锁（mutex）：独占式锁，一次只能有一个线程获取到锁，其他线程需要等待。
2. 读写锁（read-write lock）：共享式锁，多个线程可以同时获取到锁，但是在写操作时，其他线程需要等待。
```cpp
std::condition_variable cv;
std::mutex mtx;

std::lock_guard<std::mutex> lck(mtx); //不可以用在函数参数传递或者返回过程中，只能用在简单的临界区代码段的互斥操作中

std::unique_lock<std::mutex> lck(mtx); //可以用在函数参数传递或者返回过程中，比lock_guard更灵活
cv.wait(lck); //使线程进入等待状态，等待其他线程通知 2 lck.unlock(); //可以手动释放锁

cv.notify_all(); //通知所有等待的线程,条件成立了，起来干活，其他在cv上等待的线程，收到通知，从等待状态转换为阻塞态，获取到互斥锁后，继续执行
```
## override
override 是 C++11 引入的一个 关键字 ，它的作用是用来标识一个函数是重写（override）了基类的虚函数。如果函数签名与基类的虚函数不匹配，编译器会报错
