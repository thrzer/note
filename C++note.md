# C++笔记   
by：thrzer
- [C++笔记](#c笔记)
  - [初始化列表(initialization list)](#初始化列表initialization-list)
  - [友元函数：关键字 friend](#友元函数关键字-friend)
  - [继承](#继承)
    - [公有继承](#公有继承)
    - [私有继承和保护继承](#私有继承和保护继承)
  - [可变参数的函数(default argument)](#可变参数的函数default-argument)
  - [函数重载](#函数重载)
  - [内联函数：关键字 inline](#内联函数关键字-inline)
  - [关键字 const](#关键字-const)
  - [引用](#引用)
  - [向上造型](#向上造型)
  - [多态性：关键字 virtual](#多态性关键字-virtual)
    - [虚函数](#虚函数)
    - [虚基类](#虚基类)
  - [拷贝构造](#拷贝构造)
  - [关键字 static](#关键字-static)
  - [运算符重载：关键字 operator](#运算符重载关键字-operator)
    - [二元运算符重载](#二元运算符重载)
    - [一元运算符重载](#一元运算符重载)
  - [类型转换：关键字 explicit](#类型转换关键字-explicit)
    - [借助构造函数](#借助构造函数)
    - [调用转换函数](#调用转换函数)
  - [模板：关键字 template](#模板关键字-template)
    - [函数模板](#函数模板)
    - [类模板](#类模板)
  - [异常：try-catch语句 \& 关键字 throw](#异常try-catch语句--关键字-throw)
  - [流](#流)
  - [其他](#其他)


<font size = 3 color = #ff7fff>**前言：有关类、头文件、访问限制、构造与析构函数、STL等内容未记录在本文内**</font>


## 初始化列表(initialization list)
```
class A
{
private:
    int a;
public:
    A(int b = 1):a(2){a = b;}   //a先被赋值2，在函数内再被赋值b
}
```
1. 初始化列表可以初始化const的成员变量，但不能对static成员变量初始化
## 友元函数：关键字 <font size = 4 color = #00ffff>friend</font>
在类中用friend修饰另一个对象，用以声明该对象的成员函数内可以调用本类的private成员
```
class A
{
    friend void fun();
    friend class B;
}
```
1. 友元的友元不是自己的友元
## 继承
### 公有继承
子类将按顺序继承父类所有成员和访问属性
```
class 子类名 ：public 父类名
｛
    ……
｝
```
### 私有继承和保护继承
私有继承中，父类的public和protected成员在子类中视作private，private成员在子类中无法被访问\
保护继承中，父类的public和protected成员在子类中视作protect，private成员在子类中无法被访问
```
class 子类名 ：private(或protected) 父类名
｛
    ……
｝
```

## 可变参数的函数(default argument)
写一个参数或两个参数都可以，但只能写在h文件的声明内\
<font size = 4 color = #00e7a7>test.h</font>
```
void f(int a,int b = 0);
```
<font size = 4 color = #00e7a7>test.cpp</font>
```
void f(int a,int b)
｛
    ……
｝
```
调用时可以不提供b参数，b默认为0\
<font size = 4 color = #00e7a7>main.cpp</font>
```
f(1);
f(1,2);
```

## 函数重载
同一类中出现同名函数，可以根据参数表区分，但不能根据返回类型区分
1. 子类出现与父类同名函数时，会覆盖所有父类同名函数，无论参数表是否不同
2. 若需要调用父类中的同名函数，需在在函数名前加上父类名，如：
```
ch.PARENT::pfun();  //父类的重载函数
ch.PARENT::pfun(1); //父类的重载函数
//ch.pfun();        //error! 父类的重载函数被覆盖
ch.pfun(1,2);       //正常调用子类的函数
```

## 内联函数：关键字 <font size = 4 color = #00ffff>inline</font>
相当于直接将对应函数代码移动到调用位置。以增加代码空间来减少调用函数造成的额外时空开销\
<font size = 4 color = #00e7a7>test.h</font>
```
inline int itest(int x, int y)
{
	return x * y;
}
```

1. 被inline修饰的函数，连同函数头和主体都属于声明，需写在h文件中，不能写在cpp文件中
2. 内联函数不能是递归，也不能过于复杂，否则无法通过编译
3. 若在类声明中出现函数主体，不论是否有inline关键字都是内联函数。同时也可以在文件后单独写内联函数主体，但此时必须加inline

## 关键字 <font size = 4 color = #00ffff>const</font>
const int ＊p，不能通过p修改对象，但可以修改p指向的位置\
int ＊const p，不能修改p指向的位置，但可以修改p指向的对象\
const做类或函数后缀，不能修改成员变量\
const修饰函数返回值，表示返回的对象不能做左值
1. 某一变量做运算的过程中，会被保存为const类型，若此时将其视作引用（或指针）传入函数，将发生错误 `（如果不是引用就可以，意为该函数保证不会去修改这个i，但实际上本就无法修改，因此不是通过指针或者引用传入函数，可以不用const）`
```
void f(const int& a);//若删去const将报错

int i = 3;
f(i * 3);
```
2. 类中函数后加const，表示不能通过该函数修改成员变量 `可以理解为const this`
```
class A
{
    int a;
    void f() const {……} //在f中不能修改a
}
```
## 引用
C++对指针进行了包装，创造了引用，用符号 "&" 标明
```
int＆ a = b;
```
相当于a是b的别名，引用类型可以作为参数也可以作为返回值
1. 引用和指针相关，并拥有指针的特性。类似指针作为另一函数的形参时，常常加上const保证不去修改对象，C++中推荐此用法 `传入引用时，不需要让对象进行拷贝构造，可以提高程序效率`
3. 引用做函数返回值，若返回值作为右值，则赋值给左值；若返回值作为左值，则所引用对象被赋值
4. 引用类型实际上和原类型是同类型，因此可以赋值给原类型，而不能参与函数重载
5. 引用作为类成员时，必须在构造函数的初始化列表中初始化
   ```
   class A
   {
    public:
        A(int& b)：a(b){}
    private:
        int&a ;
   }
   ```
6. 引用类型不能构造数组
7. 引用绑定对象，不能通过强制类型转化为非父类的引用类型
8. 没有引用的引用，但有指针的指针，没有引用的指针，但有指针的引用

## 向上造型
子类对象通过调用指针或引用，可以当做父类对象使用\
如果直接把子类对象传入需要父类对象的函数，将会丢掉多出那一部分\
通过调取指针，再将指针进行强制类型转化，可以强行访问成员，无论是否private

## 多态性：关键字 <font size = 4 color = #00ffff>virtual</font>
作为函数前缀，用以区分父类和子类的同名函数

### 虚函数
此时p可以是含同名函数的子类的指针，通过此方法，可以通过父类调用子类函数
```
virtual void f(A ＊p)//传入的是子类对象
｛
    p-＞f()；//调用子类的f函数，若无virtual，将调用父类的f函数
｝
```
```
class A
{
    virtual void g();
}
class B : public A
{
    void g();
}
int main()
{
    A *p = new B;
    p-＞g();    //调用子类的g函数，若无virtual，将调用父类的g函数
}
```

### 虚基类
假如B1，B2都是A的子类，而C又是B1和B2的子类，那么C类重复继承B1和B2中A的部分，导致二义性。此时需要在B1，B2继承A时都加上virtual
```
class A
{
    int a;
}
class B1 : virtual public A
{
    int b1;
}
class B2 : virtual public A
{
    int b2;
}
class C : public B1, public B2
{
    int c;
}
```

1. virtual会使类多包含一个指向该类虚函数表的指针，从而使类占用空间变大。动态绑定依靠虚函数表实现
2. virtual是动态绑定，假设new了一个子类并交给父类指针，那么delete时会调用父类的析构，导致子类没有被完全delete。因此析构函数最好也加上virtual

## 拷贝构造
在对象传入、返回或赋值时，会调用拷贝构造函数，将对象赋值给另一个对象，此时会调用拷贝构造函数。若无拷贝构造函数，编译器将自动调用默认的拷贝构造函数，将所有成员全部拷贝过去 `注意：不同于部分教材的说法，实际上并不是字节对字节式的拷贝`
```
class A
{
    A(){}           //构造函数
    A(const A& a){} //拷贝构造函数
    //A(A a){}
    //error!这样写也属于传入对象，会造成拷贝构造函数递归，陷入死循环
};
```
1. 构造函数有形参时，可以有如下两种方式传入参数：\
    <font size = 4 color = #00e7a7>test.h</font>
    ```
    class A
    {
    public:
        A(int a):b(a){}
        int b;
    };
    ```
    <font size = 4 color = #00e7a7>main.cpp</font>
    ```
    A a(1);
    A a = 1;    //这两种方式是等价的，但一般用第一种
    ```
2. 编译器的默认拷贝函数在拷贝对象时，也会调用该对象的拷贝构造函数；拷贝指针或引用时，拷贝结果会对应同一个对象 `这是不好的，因为当两个对象都进行析构时，可能会delete两次。正确处理是让两个指针（或引用）对应不同的对象`
3. 拷贝函数被声明为private时，编译器不会自动调用该拷贝函数，此时不能直接通过调用该类的对象 `Java中常这么做`
4. 当某一函数有形参没有被使用时，会被编译器优化掉，不进行拷贝构造 `这一过程只在编译中发生，不在链接时发生，所以实际上只有当该函数是inline时才会出现被优化的情况`

## 关键字 <font size = 4 color = #00ffff>static</font>
修饰本地变量时，表示该变量的生存期为整个程序的生命周期，在下一次调用该函数时，该变量能保持上一次调用后的值\
修饰类的成员变量时，所有该类的对象共享该成员变量，也就是说，所有该类的对象都共用这一个变量。此时需要在类的外部定义该变量，否则编译器会报错\
修饰类的成员函数时，该函数只能调用static成员变量，不能用非static成员变量\
<font size = 4 color = #00e7a7>test.cpp</font>
```
int A::a;   //注意这里不能再加static，否则只有该cpp可以调用a
```
<font size = 4 color = #00e7a7>test.h</font>
```
class A
{
public：
    static int a;   //这相当于extern int a所以还需要在别处定义a
    int b;
    static void f(int c){cout << a+c << endl;}
    //static void g(){cout << b << endl;}   
    //error!不能访问非static类型的b
}；
```
1. 在C语言中，static修饰全局变量或函数时，表示该变量或函数的作用域只在当前c文件 `在C++中仍有这一特性，但属于相对“过时”的用法，因为通过类的访问限制（public，private，protected）已经可以实现此效果`
2. static修饰本地变量时，实际上将该变量保存在了和全局变量相同的内存区域，可以理解为其作用域只有该变量所在的函数
3. 全局变量在编译生成时顺序是不定的，因而程序不能对全局变量的声明顺序有依赖
4. static成员变量不能用初始化列表初始化，只能在声明时初始化，或者由其他函数赋值
5. 当static成员变量或函数为public时，有如下两种调用方式：`这是一种不需要创建对象就可以直接访问类中成员的手段，正因为访问时对象不一定存在，所以也就不能用this指针，更不能访问非static类型的成员`
    ```
    A s;
    cout << s.a << endl;
    cout << A::a << endl;
    ```

## 运算符重载：关键字 <font size = 4 color = #00ffff>operator</font>
C++中可以重新定义已有的运输符，但必须满足如下条件：\
1、参与运算的对象是由程序员自己定义的类型，比如类、枚举\
2、不能改变操作数的个数，比如"+"只能重载为二元运算符\
3、不能改变运算符原先满足的优先级

同时，还需要对应一个函数来描述运算规则，写成如下形式：
### 二元运算符重载
写法如下：
```
class A
{
public:
    int a;
    int b;
    A(int a,int b):a(a),b(b){}
    const A operator*(const A& a)const{return A(a*a.b,b*a.a);}
    //这三个const根据需要加上，以确保返回值不能做左值且不去修改算子
};
```
在main中调用：
```
A a(1,2);
A b(2,3);
A c = a*b;  //按照重载函数，c.a = 3，c.b = 6
```
### 一元运算符重载
写法如下：
```
class A
{
public:
    int a;
    A(int a):a(a){}
    const A operator-()const{return A(-a);}
};
```

1. 几乎所有运算符都可以重载，除了 "." "::" "?:" `三元运算符不能重载` "sizeof" "typeid" 和一些以"_cast"结尾的运算符
2. 如果重载函数不写在类中，把操作对象也加入到参数中即可 `"=" "->" "()" "[]" 等运算符只能写在类中，单目运算符也建议写在类中。除此之外，一般不把重载函数写在类中，而是通过声明为friend让类可以调用`
3. 调用重载运算符时，优先根据运算符左边的对象类型，选择对应的重载函数 `编译器会自动检查能用哪个重载函数（个人理解：根据访问权限，只有左边这个类的重载函数和全局重载函数在检查范围内）`
4. 如果参与重载运算符对象与重载函数的参数类型不匹配，编译器会尝试将对象自动转换为与重载函数参数类型一致的类型，然后再调用重载函数 `实际上非重载的运算符也会发生这一过程，如果对象不能转换为对应参数类型，编译器会报错`\
（可以简单理解为，只有将要作为重载函数参数的对象，编译器才会尝试自动类型转化。即谁进参数表，谁可以转化）
    ```
    //若在h文件中，类A中某一成员函数重载了运算符 "+"
    A a(1);
    A b(1);
    A c = a + b;    //正确使用
    A d = a + 1;    //编译器会将1转换为A类型，再进行运算
    //A e = 1 + b;  
    //error!编译器会优先调用int类型的"+"运算，但是int的+需要两个int，而b不是int形，且A没有关于int的转化函数，故报错
    A f = 1 - 0;  
    //"=" 与 "()"意义相同，相当于A f(1-0)，用这个结果传入f的构造函数进行初始化

    //若在h文件中，某一非成员函数重载了运算符 "-"
    A g = a - b;    //正确使用
    A h = a - 1;    //编译器会将1转换为A类型，再进行运算
    A h = 1 - b;    //编译器会将1转换为A类型，再进行运算
    A i = 1 - 0;    //相当于初始化，而不是将1和0都转化为A类型
    ```
1. 关于定义重载运算符的规范，此处不多赘述 `可以参考浙大C++课程p31《运算符重载---原型》，包括对加减乘除、比较、数组的定义`\
这里以 "=" 为例：
    ```
    const A& operator=(const A& b)
    {
        //必须做这样的判断，否则可能出错，原因可见下面示例
        if(this != &b)  //引用是对指针的包装，本质还是指针，因此可以做判断
        {
            /*
            delete p;
            //假如A类中声明了一个成员是指针，在弃用p时应当delete
            p = b.p;    
            //在这一步赋值时，假如b和a是同一个对象，那么此时b.p已经不存在了，发生错误
            */
            //或其他自定义赋值逻辑
        }    
        return *this;
    }
    ```
1. "=" 默认是赋值。如果对象的类没有重载 "="，会将右边对象拷贝给左边对象，有重载函数则优先调用重载函数。

## 类型转换：关键字 <font size = 4 color = #00ffff>explicit</font>
### 借助构造函数
当一个类的对象可以传入构造函数以构造另一个类的对象时，若将该对象传入函数以另一个对象为形参的函数，编译器会自动调用另一个对象的构造函数，实现类型转换\
<font size = 4 color = #00e7a7>test.h</font>
```
class A{};
class B
{
    B(A a){}
};
void f(B b);
```
<font size = 4 color = #00e7a7>main.cpp</font>
```
A a;
f(a);   
//编译器会自动调用B的构造函数，将a传入，构造一个B类的对象，再将这个对象传入f函数
```
假如不希望编译器自动调用构造函数，可以将构造函数声明为explicit，此时编译器将不再自动调用构造函数，而是报错 `explicit只能修饰构造函数，表示该构造函数不能被自动调用`
```
class B
{
    explicit B(A a){}
};
```
如果又需要主动让编译器调用构造函数做类型转换，可以这么写：
```
f(B(a));
```

### 调用转换函数
<font size = 4 color = #00e7a7>test.h</font>
```
class A
{
    ……
    operator B()const;
};
class B{};
void f(B b);
```
<font size = 4 color = #00e7a7>main.cpp</font>
```
A a;
f(a);
```

1. 以上两种类型转换方式没有优先级的区分，因此当两种写法都出现时会报错。如果希望优先用某一种方式，同样可以在构造函数前加explicit：
    ```
    class A
    {
        ……
        operator B()const;
    };
    class B
    {
        explicit B(A a){}   //做转换时不选择调用构造函数
    };
    ```
2. 尽管有自动类型转换，但更推荐单独写一个函数，在需要做类型转换时直接调用这个函数，比如：
    ```
    class A
    {
        B toB(A a)const;
    };
    ```

## 模板：关键字 <font size = 4 color = #00ffff>template</font>
### 函数模板
当需要对不同类型的对象做相似的处理时，不必重写代码，可以选择使用模板，用法如下：\
<font size = 4 color = #00e7a7>test.h</font>
```
template <class T>      //T为模板参数，表示该模板可以用于任何类型
void swap(T& a, T& b)   //template的下一行将作为模板函数
{
    T temp = a;
    a = b;
    b = temp;
}
```
有了模板之后，可以将不同类型的对象当作T类型使用\
<font size = 4 color = #00e7a7>main.cpp</font>
```
int a,b;
A c,d;
B e,f;
swap(a,b);
swap(c,d);
swap(e,f);
```

1. 模板的实质是向编译器声明一个制造函数的方式，`因此要写在h文件中` 当不同类型的对象传入时，编译器会按照所给的模板，自动生成对应的函数 `这些函数构成重载关系`\
可以按如下方式主动用模板指定制造函数：
    ```
    template <class T>
    void f(T& a)
    {
        ……
    }

    f<int>(a);  //制造了一个以int类型为参数的函数f
    ```
2. 模板函数中所有的模板参数 "T" 在调用时必须传入同种类型，编译器不会做自动类型转换，也就是说必须严格按照给出的类型模板传入参数\
在传入的T都是同种类型，却和已经指定的T的类型不符时，编译器才会自动将传入的参数转换为指定的类型
    ```
    template <class T>  // "T" 是模板参数名称，可以更改，一般就用 "T"
    void g(T& a,T& b)
    {
        ……
    }

    int a,b;
    g<float>(a,b);
    //生成了一个以float类型为参数的函数f，这时候编译器会自动把a和b转换为float类型
    ```

### 类模板
和函数模板类似，可以用于生成类，用法如下：\
<font size = 4 color = #00e7a7>test.h</font>
```
template <class T>
class A
{
public:
    A(T a):a(a){}
    T a;
    void f(T& a);
}
```
<font size = 4 color = #00e7a7>main.cpp</font>
```
A<int> a;
A<float> b;
//生成了两个类，名称是A<int>和A<float>，对应成员是int类型和float类型
```

1. 模板类中的函数都是模板函数，如果函数中涉及对T的运算，例如 ">" "<" 的比较，那么指定T的类型时需要有该类型对应的运算符重载函数 `这是一个比较容易遗漏的地方`
2. 需要多个模板参数时这么写：
    ```
    template <class A,class B>
    ```
3. 调用多重模板时会这么写：
    ```
    A< B<int> >  //个别编译器对空格有要求，不过VS没有
    ```
4. 模板参数也可以是变量：
    ```
    template <class T,int a = 5>   //类比函数的参数
    class A
    {
    T b[a];
    };
    ```
   传入的时候就可以直接是一个值：
    ```
    A<int> a;
    A<int, 10> b;
    ```
5. 继承时，子类可以是模板类，但父类不能是模板类 `向父类模板传入模板参数，变成一个具体的类就行`

## 异常：<font size = 4 color = #00ffff>try-catch语句 & 关键字 throw</font>
在编译过程中，或程序运行过程中，可能会产生一些错误，称为异常。C++对代码的错误检查比C更全面\
为了让程序有好的健壮性，需要在代码中加入异常处理。但传统的if-else语句会导致大量嵌套，降低可读性。为了让程序将正确情况和异常情况区分开，同时提高可读性，可以使用try-catch语句：\
<font size = 4 color = #00e7a7>Tsak.h</font>
```
class Aerror    //建立异常类用来描述异常的情况
{
public:
    int a;
    Aerror(int x){……} //根据造成异常对象构造出异常类的对象
};
class Task
{
public:
    void taskA();
    void taskB();
}
```
<font size = 4 color = #00e7a7>Task.cpp</font>
```
void Task::taskA()
{
    ……  //正常代码，实现功能A
    if(/*检查到由对象x造成的异常*/)
    {
        throw Aerror(x);
    }
}
```
<font size = 4 color = #00e7a7>main.cpp</font>
```
Task M;
try
{
    M.taskA();
    //假如taskA()中抛出异常，那么Aerror会被一直传回到这里
    M.taskB();
}
catch(Aerror& e)
//只能捕捉Aerror类的异常，也就是从taskA()中抛出的异常
{
    ……  //进行异常处理
}
catch(Berror& e){}    //捕捉taskB()中抛出的异常
```

1. 当throw被执行时，后面的代码都不会执行，程序进程会不断**原路返回**，直到遇到第一个try语句，执行对应的catch语句，然后退出整个try-catch语句，再向下执行
2. 如果不需要对异常类型做区分，可以建立异常类，直接用catch(...)来捕捉所有异常 `这种办法不能捕捉到抛出的异常对象，往往作为最后的“兜底”手段使用`
    ```
    Task M;
    try
    {
        M.taskA();
        M.taskB();
        M.taskC();
    }
    catch(Aerror& e){}
    catch(...)
    //捕捉任意被抛出的异常，可以来自taskA(),taskB(),taskC()等
    {
        ……  //进行异常处理
    }
    ```
3. 异常处理可以嵌套，在catch中再次出现throw，交给更高一层的语句去处理异常，称为异常的传播。在catch里的throw后面可以不跟要抛出的对象，表示把当前catch捕捉到的异常继续抛出给上一层的catch
4. 捕捉时，按catch书写的顺序捕捉，每个catch都会依次检查是否有匹配的异常类、是否有匹配的父类异常类型
5. 可以在函数声明中对其可能抛出的异常类型做限定：
    ```
    void taskA() : throw(Aerror,Berror)
    //taskA()只能抛出Aerror或Berror异常
    //如果throw后的括号为空，表示该函数“承诺”不抛出异常
    ```
6. new申请内存不成功时，会抛出名为 "bad_alloc" 的异常，可以用于后续的异常捕捉
7. 在构造函数中的throw，容易造成内存泄漏，需要delete所有已经申请的内存
8. 一个小细节：throw抛出的对象最好是堆栈中的对象 `也就是说最好不是指针，容易在后面忘记delete`，这样在catch后，这块空间会更快的被覆盖掉，可以提高内存的利用率

## 流
C++自带的四个标准流：\
cin,cout,cerr(标准错误),clog(标准日志)

1. cin,cout不需要像C一样使用格式化输入输出，会自动根据对象的类型选择输入或输出方式
2. 对于自定义类型，需要对流进行运算符重载：
    ```
    istream& operator>>(istream& is,ClassA& a)
    {
        //user's code
        return is;
    }
    ostream& operator<<(ostream& os,const ClassA& a)
    {
        //user's code
        return os;
    }
    ```
    这样的重载函数应当作为全局函数
3. cin下有很多函数，比如cin.get()，进行单个输入；putback()，将字符放回输入流；ignore()，忽略输入流的若干字符；peek()，返回下一个字符，但不将其从输入流中删除……
4. 可以使用一些操纵子，比如endl换行，dec十进制，oct八进制，hex十六进制，setbase()任意进制，flush刷新缓冲区……，setprecision(2)精度为2，setw(5)最小宽度为5，setfill('0')空位补0，ws跳过空格……\
操纵子也可以重载：
    ```
    ostream& tab(ostream& os)   //自制一个对齐操纵子
    {
        return os << '\t';
    }
    ```
## 其他

<font size = 3>**auto 类型推导**</font>

1. auto类型推导，C++11新特性，可以自动推导变量的类型，但只能推导变量的声明，不能推导变量的赋值
2. 不能作为形参

<font size = 3>**更多指针**</font>

<font size = 4 color = #ff7f3f>**备注：STL标准库、sfml和easyx图形库另开一篇**</font>

