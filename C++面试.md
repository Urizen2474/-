

![image-20220825181313288](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220825181313288.png)



# 1.C++

## 1.虚函数表和多态(*)

### 1.1虚函数和多态

虚函数（Virtual Function）：在基类中声明为 virtual 并在一个或多个派生类中被重新定义的成员函数。
纯虚函数（Pure Virtual Function）：基类中没有实现体的虚函数称为纯虚函数（有纯虚函数的基类称为虚基类）。
C++  “虚函数”的存在是为了实现面向对象中的“多态”，即父类类别的指针（或者引用）指向其子类的实例，然后通过父类的指针（或者引用）调用实际子类的成员函数。通过动态赋值，实现调用不同的子类的成员函数（动态绑定）。正是因为这种机制，<font color='red'>把析构函数声明为“虚函数”可以防止在内存泄露。</font>

#### 1.1.1虚函数的访问权限

```c++
#include <iostream>

using namespace std;

class A
{
public:
	virtual void Print() { cout << "A::Print" << endl; }
};
class E :public A
{
//public:	/* 即使是private，编译依然可以通过，多态时也能正确调用E::Print() */
private:
	virtual void Print() { cout << "E::Print" << endl; }
};

void PrintInfo(A& r)
{
	r.Print();		// 多态，调用哪个Print，取决于r引用了哪个类的对象
}

int main()
{
	E e;
	PrintInfo(e);		// 输出 E::Print
	return 0;
}

```

一般情况下，基类与派生类的同名同参数表的[虚函数](https://so.csdn.net/so/search?q=虚函数&spm=1001.2101.3001.7020)的访问范围说明符应保持一致。

- 如果基类为`private`，则编译会报错。
- 如果基类为`public`，派生类为`private`，编译依然可以通过，多态时也能正确调用派生类的虚函数。<font color='green'>//只有这种情况是特殊的</font>

![image-20220716190734243](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220716190734243.png)

#### 1.1.2静态绑定和动态绑定

​     <font size=10> 1.</font> 基类的指针不能调用派生类自己定义的函数，只能调用虚函数！因为调用普通函数时是绑定静态的，调用虚函数时才动态绑定。
​       为了支持c++的多态性，才用了动态绑定和静态绑定。理解他们的区别有助于更好的理解多态性，以及在编程的过程中避免犯错误。需要理解四个名词：
对象的静态类型：<font color='red'>对象在声明时采用的类型，是在编译期确定的。</font>
对象的动态类型：<font color='red'>目前所指对象的类型，是在运行期决定的。</font>对象的动态类型可以更改，但是静态类型无法更改。<font color='green'>//基类指针或引用</font>,但还是取决于是否是虚函数
​          静态绑定：绑定的是对象的静态类型，某特性（比如函数）依赖于对象的静态类型，发生在编译期。
​          动态绑定：绑定的是对象的动态类型，某特性（比如函数）依赖于对象的动态类型，发生在运行期。

```c++
#include 
using namespace std;
class Base
{
public:
    void FuncInBase(){cout << "Func in Base..." << endl;}
};
class Derived:public Base
{
public:
    void FuncInDerived() {cout << "Func in Derived..." << endl;}
}; 
int main(void)
{
    Base *p = new Derived();
    p->FuncInBase();   //Func in  Base
    p->FuncInDerived();  //编译不通过
    return 0;
}
```

<font size=10>2.</font>虚函数的访问控制（编译器根据对象的静态类型来决定访问控制权限，并且进行形参的默认参数的赋值！）

​    虚函数是实现多态的机制，也就是说在利用基类的指针或者引用调用虚函数时，调用的是该指针或者引用的**动态类型**的相应的函数，这里有几点需要注意的。

（1） 编译器在决定调用函数时，如果该函数是虚函数才会在运行时确定调用什么函数（动态绑定），如果不是虚函数，那么在编译阶段就已经确定了调用的函数类型（静态绑定）。
        如下面的代码，基类与派生类都声明了函数f。但是在main函数的调用中编译器调用的是静态类型对应的函数，因为f函数并不是虚函数，虽然在基类与派生类中都声明了该函数。

```c++
class Base
{
public:
    void f(int i=0) {cout << "f() in Base..." << i << endl;}
};
class Derived:public Base
{
private:
    void f(int i=1){cout << "f() in derived..." << i << endl;}
};
int main(void)
{
    Base *b = new Derived();
    b->f();//f() in Base...
    return 0;
}
```

​    (2）如下基类定义虚函数为public，派生类覆盖了该虚函数，但是将其声明为private，这样当基类的指针绑定到派生类的对象时，使用该基类指针调用该虚函数时，调用是否成功。如果二者的访问权限反过来呢。

```C++
class Base
{
public:
    virtual void f(int i=0) {cout << "f() in Base..." << i << endl;}
};
class Derived:public Base
{
private:
    void f(int i=1){cout << "f() in derived..." << i << endl;}
};
 
int main(void)
{
    Base *b = new Derived();
    b->f();//f() in derived...0
    return 0;
}

```

分析】首先分析为什么输出结果是f() in derived。 编译器在看到b对f进行调用时，此时编译器根据b的**静态类型**（也就是Base）来决定f函数是否可访问，并且进行形参的默认参数的赋值！！
        由于f是虚函数，那么具体调用哪个函数是在运行时确定的，于是在运行时查找Derived的虚函数表，得到虚函数f（此时的f已经被Derived类覆盖，于是调用的就是派生类的版本。）
       至于，为什么i的值为0，上述分析也已经说明。
       如果将两者的访问权限交换，那么访问控制这一关都过不了，其实很简单，既然你需要派生类继承f函数，将其在Base类中声明为private本身就是不对的。

​      

#### 1.1.3函数调用机制

例：Base*base=new Derived();同有一个f()函数

1.根据静态类型(Base)决定f函数的访问权和默认值的赋值(base的f()的默认值)

2.判断Base的f()是否是虚函数，是则查找Derived的虚函数表

3.编译器还会根据参数类型决定查谁的虚函数表，如果Base和Derived的f的参数列表不一样,则会调用Base的虚函数表

4.调用Derived的f()

##### 1.1.3.1虚函数和静态函数冲突

```c++
#include <iostream>
using namespace std;

class A                                                     
{
public :
    void virtual print(){ cout<<"A::print()"<< endl; }
};

class B :public A
{
public :
    void virtual print(){ cout<<"B::print()"<< endl; }
} ;
class C :public B   
{
public :
    static void print(){ cout<<"C::print()"<< endl; }
} ;

void print (A a)                   
{   
     a.print();                         
}

void main()
{                                                                                                           
     A a, *aa, *ab, * ac;                       
     B b;                                                                         
     C c;
     aa=& a;                                                                                                       
     ab=& b;                           

     ac=& c;   
                                           
    //c.print();                                                         
     ac-> print();  //B::print() (有static)                                       //C::print() (没static)
}
```

static和virtual相互冲突,声明时就不可以同时存在，子类中用static声明的函数不会加入虚函数表

### 1.2虚函数表

虚函数表(Virtual Table，V-Table)：使用 V-Table 实现 C++ 的多态。在这个表中，主要是一个类的虚函数的地址表，这张表解决了继承、覆盖的问题，保证其真实反应实际的函数。这样，在有虚函数的类的实例中分配了指向这个表的指针的内存，所以，当用父类的指针来操作一个子类的时候，这张虚函数表就显得尤为重要了，它就像一个地图一样，指明了实际所应该调用的函数。
<font color='red'>编译器应该保证虚函数表的指针存在于对象实例中最前面的位置</font>（这是为了保证取到虚函数表的有最高的性能——如果有多层继承或是多重继承的情况下）。 这意味着可以通过对象实例的地址得到这张虚函数表，然后就可以遍历其中函数指针，并调用相应的函数。

#### 1.2.1无继承的虚函数表

```c++
#include <iostream>
using namespace std;

class base_class
{
public:
    virtual void v_func1()
    {
        cout << "This is base_class's v_func1()" << endl;
    }
    virtual void v_func2()
    {
        cout << "This is base_class's v_func2()" << endl;
    }
    virtual void v_func3()
    {
        cout << "This is base_class's v_func3()" << endl;
    }
};

int main()
{
    // 查看 base_class 的虚函数表
    base_class bc;
    cout << "base_class 的虚函数表首地址为：" << (int*)&bc << endl; // 虚函数表地址存在对象的前四个字节
    cout << "base_class 的 第一个函数首地址：" << (int*)*(int*)&bc+0 << endl; // 指针运算看不懂？没关系，一会解释给你听
    cout << "base_class 的 第二个函数首地址：" << (int*)*(int*)&bc+1 << endl;
    cout << "base_class 的 第三个函数首地址：" << (int*)*(int*)&bc+2 << endl;
    cout << "base_class 的 结束标志: " << *((int*)*(int*)&bc+3) << endl;
    
    // 通过函数指针调用函数，验证正确性
    typedef void(*func_pointer)(void);
    func_pointer fp = NULL;
    fp = (func_pointer)*((int*)*(int*)&bc+0); // v_func1()
    fp();
    fp = (func_pointer)*((int*)*(int*)&bc+1); // v_func2()
    fp();
    fp = (func_pointer)*((int*)*(int*)&bc+2); // v_func3()
    fp();
    return 0;
}

```

#### 1.2.1.1指针转换

&bc：获得 bc 对象的地址
(int*)&bc: 类型转换，获得虚函数表的首地址。这里使用 int* 的原因是函数指针的大小的 4byte，使用 int* 可以使得他们每次的偏移量保持一致（sizeof(int*) = 4，32-bit机器）。
(int*)&bc：解指针引用，<font color='red'>获得虚函数表</font>。
(int*)*(int*)&bc+0：和上面相同的类型转换，获得虚函数表的第一个虚函数地址。
(int*)*(int*)&bc+1：同上，获得第二个函数地址。
(int*)*(int*)&bc+2：同上，获得第三个函数地址。
*((int*)*(int*)&bc+3：获得虚函数表的结束标志，所以这里我解引用了。和我们使用链表的情况是一样的，虚函数表当然也需要一个结束标志。
typedef void(*func_pointer)(void)：定义一个函数指针，参数和返回值都是 void。
*((int*)*(int*)&bc+0)：找到第一个函数，注意这里需要解引用。

#### 1.2.1.2单一继承的虚函数表

子类没有父类的虚函数（陈皓文章中用了“覆盖”一词，我觉得太合理，但是我又找不到更合理的词语，所以就用一个句子代替了。）

```c++
#include <iostream>
using namespace std;

class base_class
{
public:
    virtual void v_func1()
    {
        cout << "This is base_class's v_func1()" << endl;
    }
    virtual void v_func2()
    {
        cout << "This is base_class's v_func2()" << endl;
    }
    virtual void v_func3()
    {
        cout << "This is base_class's v_func3()" << endl;
    }
};
class dev_class : public base_class
{
public:
    virtual void v_func4()
    {
        cout << "This is dev_class's v_func4()" << endl;
    }
    virtual void v_func5()
    {
        cout << "This is dev_class's v_func5()" << endl;
    }
};

int main()
{
    // 查看 dev_class 的虚函数表
    dev_class dc;
    cout << "dev_class 的虚函数表首地址为：" << (int*)&dc << endl;
    cout << "dev_class 的 第一个函数首地址：" << (int*)*(int*)&dc+0 << endl;
    cout << "dev_class 的 第二个函数首地址：" << (int*)*(int*)&dc+1 << endl;
    cout << "dev_class 的 第三个函数首地址：" << (int*)*(int*)&dc+2 << endl;
    cout << "dev_class 的 第四个函数首地址：" << (int*)*(int*)&dc+3 << endl;
    cout << "dev_class 的 第五个函数首地址：" << (int*)*(int*)&dc+4 << endl;
    cout << "dev_class 的虚函数表结束标志: " << *((int*)*(int*)&dc+5) << endl;
    // 通过函数指针调用函数，验证正确性
    typedef void(*func_pointer)(void);
    func_pointer fp = NULL;
    for (int i=0; i<5; i++) {
        fp = (func_pointer)*((int*)*(int*)&dc+i);
        fp();
    }
    return 0;
}

输出结果: 
dev_class 的虚函数表首地址为：0x22ff0c
dev_class 的 第一个函数首地址：0x472d10
dev_class 的 第二个函数首地址：0x472d14
dev_class 的 第三个函数首地址：0x472d18
dev_class 的 第四个函数首地址：0x472d1c
dev_class 的 第五个函数首地址：0x472d20
dev_class 的虚函数表结束标志: 0
This is base_class's v_func1()
This is base_class's v_func2()
This is base_class's v_func3()
This is dev_class's v_func4()
This is dev_class's v_func5()
可以看出，v-table中虚函数是顺序存放的，先基类后派生类。
    
    
    
    
```

#### 1.2.1.3子类有重写父类的虚函数

输出结果: 
dev_class 的虚函数表首地址为：0x22ff0c
dev_class 的 第一个函数首地址：0x472d10
dev_class 的 第二个函数首地址：0x472d14
dev_class 的 第三个函数首地址：0x472d18
dev_class 的 第四个函数首地址：0x472d1c
dev_class 的 第五个函数首地址：0x472d20
dev_class 的虚函数表结束标志: 0
This is base_class's v_func1()
This is base_class's v_func2()
This is base_class's v_func3()
This is dev_class's v_func4()
This is dev_class's v_func5()

可以看出当派生类中 dev_class 中重写了父类 base_class 的前两个虚函数（v_func1，v_func2）之后，使用派生类的虚函数指针**代替**了父类的虚函数。未重写的父类虚函数位置没有发生变化。//代码块里面的任何操作都不会影响

虚函数表的指针指向(比如: 在重写的函数里面调用基类函数)

#### 1.2.1.4多重继承下的虚函数表

```c++
#include <iostream>
using namespace std;

class base_class1
{
public:
    virtual void bc1_func1()
    {
        cout << "This is bc1_func1's v_func1()" << endl;
    }
};

class base_class2
{
public:
    virtual void bc2_func1()
    {
        cout << "This is bc2_func1's v_func1()" << endl;
    }
};

class dev_class : public base_class1, public base_class2
{
public:
    virtual void dc_func1()
    {
        cout << "This is dc_func1's dc_func1()" << endl;
    }
};

int main()
{
    dev_class dc;
    cout << "dc 的虚函数表 bc1_vt 地址：" << (int*)&dc << endl;
    cout << "dc 的虚函数表 bc1_vt 第一个虚函数地址：" << (int*)*(int*)&dc+0 << endl;
    cout << "dc 的虚函数表 bc1_vt 第二个虚函数地址：" << (int*)*(int*)&dc+1 << endl;
    cout << "dc 的虚函数表 bc1_vt 结束标志：" << *((int*)*(int*)&dc+2) << endl;
    cout << "dc 的虚函数表 bc2_vt 地址：" << (int*)&dc+1 << endl;
    cout << "dc 的虚函数表 bc2_vt 第一个虚函数首地址：：" << (int*)*((int*)&dc+1)+0 << endl;
    cout << "dc 的虚函数表 bc2_vt 结束标志：" << *((int*)*((int*)&dc+1)+1) << endl;
    // 通过函数指针调用函数，验证正确性
    typedef void(*func_pointer)(void);
    func_pointer fp = NULL;
    // bc1_vt
    fp = (func_pointer)*((int*)*(int*)&dc+0);
    fp();
    fp = (func_pointer)*((int*)*(int*)&dc+1);
    fp();
    // bc2_vt
    fp = (func_pointer)*(((int*)*((int*)&dc+1)+0));
    fp();
    return 0;
}

输出结果：


dc 的虚函数表 bc1_vt 地址：0x22ff08
dc 的虚函数表 bc1_vt 第一个虚函数地址：0x472d38
dc 的虚函数表 bc1_vt 第二个虚函数地址：0x472d3c
dc 的虚函数表 bc1_vt 结束标志：-4
dc 的虚函数表 bc2_vt 地址：0x22ff0c
dc 的虚函数表 bc2_vt 第一个虚函数首地址：：0x472d48
dc 的虚函数表 bc2_vt 结束标志：0
This is bc1_func1's v_func1()
This is dc_func1's dc_func1()
This is bc2_func1's v_func1()

```

![img](https://imgconvert.csdnimg.cn/aHR0cDovL3BpYzAwMi5jbmJsb2dzLmNvbS9pbWFnZXMvMjAxMi8xNTMzNTcvMjAxMjA3MTExNjMxMzM4Mi5qcGc?x-oss-process=image/format,png)

多重继承的情况，会为每一个基类建一个虚函数表。派生类的虚函数放到第一个虚函数表的后面。

这个结束标志（虚函数表）的值在不同的编译器下是不同的。在WinXP+VS2003下，这个值是NULL。而在Ubuntu 7.10 + Linux 2.6.22 + GCC 4.1.3下，这个值是如果1，表示还有下一个虚函数表，如果值是0，表示是最后一个虚函数表。”。那么，我在 Windows 7 + Code::blocks 10.05 下尝试，这个值是如果是 **-4**，表示还有**下一个**虚函数表，如果是**0**，表示是**最后一个**虚函数表。

#### 



### 1.3内存的存储方式

即使类没有提供public接口操作private数据成员，我们一样可以通过获取类对象地址来操作私有数据成员

示例:  

```C++
class CTest
{
public:
    explicit CTest(int prii=111,int proi=222,int pubi=333)
    {
        m_prii=prii;
        m_proi=proi;
        m_pubi=pubi;
    }
    void show() const 
    {
        cout<<"m_prii="<<m_prii <<" m_proi="<<m_proi<<" m_pubi="<<m_pubi<<endl;
    }
private:
    int m_prii;
protected:
    int m_proi;
public:
    int m_pubi;
};


int main()
{
    CTest tobj;
    tobj.show();
    int *pi= (int *)(&tobj);//*****注意强制转换,如有疑问参读《inside C++ object model》
    cout<<"&tobj="<<&tobj<<" \n";//tobj对象本身的地址
    cout<<pi<<"="<<*pi<<" ";//pi的地址和指向的值
    pi++;
    cout<<pi<<"="<<*pi<<" ";//pi+1后的地址和指向的值
    pi++;
    cout<<pi<<"="<<*pi<<" "<<endl;//pi+1后的地址和指向的值
    cin.get();
}

   
1.类对象数据成员在内存中是连续存放的，和我们在类中数据成员的声明次序一样（以上结果只是MS的VS2010编译器结果）

2.从tobj对象的起始内存地址后，依次连续存放3个相同类型数据成员，中间没有什么其他数据存在（比如说用于访问权限设置的数据…）；//必须要数据类型相同才行，而且还和虚函数表不一样，数据成员没有表，子类不能用一个基类来表示
    
结果:
程序没有通过任何接口也访问了私有成员
```

![image-20220716173307042](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220716173307042.png)

  其实所谓的访问权限(public ,protected ,private)从来都只是编译器强加给我们的规定，从而使我们编写出来的程序更符合OO特性，

   实际一般程序在内存中数据读取没有任何限制，只要你能拿到数据地址就能读写数据（这也是很多程序外挂的原理，通过抓到的地址直接修改内存数据）。

   成员函数的访问权限的作用就是由编译器决定根据你定义的访问权限，给不给你该成员函数在内存中的具体地址

在真实内存中是不存在访问权限这种设置的

   类的访问权限设置仅在程序编译期起作用，一旦程序进入执行期（虚函数动态绑定就发生在这个时候，所以即使派生类为private访问权限），就没有访问权限控制了（这时候编译器不起作用了）；

C++程序的内存格局通常分为五个区：**全局数据区（data area）,代码区（code area）、栈区（stack area）、堆区（heap area）（即自由存储区）,文字常量区。**全局数据区存放全局变量和静态变量，初始化的全局变量和静态变量在一块区域，未初始化的全局变量和未初始化的静态变量在相邻的另一块区域，程序结束后由系统释放。；所有类成员函数和非成员函数代码存放在代码区；为运行函数而分配的局部变量、函数参数、返回数据、返回地址等存放在栈区；文字常量区存储常量字符串，程序结束后由系统释放，余下的空间都被称为堆区。









## 2.递归和循环的区别

递归是通过函数不断的在内部调用自身实现的

循环是通过设置计算的初始值及终止条件，在一个范围内重复计算

<font size=5>递归的优点:</font>

递归的方式比循环的代码简洁

此外，在树的前序、中序、后序遍历算法的代码中，递归的实现方式也比循环简单的多，而且更加容易实现。

**Tips：在面试的时候，如果面试官没有特别要求，则建议尽量多采用递归的方法编程。**

<font size=5>递归的缺点</font>

递归由于是调用自身，而函数调用是有时间和空间的消耗的：每一次函数调用，都需要在内存中分配空间以保存参数、返回地址及临时变量，而且往栈里压入数据和弹出数据都需要时间，更何况，每个进程的栈的容量是有限的。因此，递归实现的效率不如循环，而且，递归还有可能引起严重的调用栈溢出等问题。

## 3.delete this

<font size=5>**−1−**</font>

在类对象的内存空间中，只有数据成员和虚表指针，并不包含代码内容。

<font size=5>**− 2 −**</font>

类的成员函数单独放在代码段中

<font size=5>**−3−**</font>

在成员函数内部执行delete this之后，只要之后执行内容和this指向的内存块无关，则毫不影响。

但涉及this指针，则会出现不可预期后果。

<font size=5>**−4−**</font>

在当前成员函数未结束前，delete this语句并未马上执行，this指向内存块不是马上回收到系统，这时访问数据成员是随机数，访问虚表发生指针无效，系统崩溃。

<font size=8>**−5−**</font>

若析构函数执行delete this 会怎样？

<font color='red'>会 堆 或 栈 溢 出</font> 

进入析构函数后，执行delete this，delete this本身调用this指向类的析构函数，所以会无限递归下去。

## 4.顶层const和底层const

<font size=7>顶层const</font>

type* const value；

<font size=7>底层const</font>

const type* value;







## 5.内存对齐

内存对齐的目的是为了让CPU能一次获取到数据，从而提升性能
CPU**只能使用基本类型**，char, short, int, long, float, double 等，不能使用数组或结构体等复合类型（汇编中并没有一个指令能直接存取一个struct或数组）。所以：内存对齐的单位是基本类型，目标是让CPU能一次获取到基本类型的值。

### 5.1什么是内存对齐

为了提高程序的性能，[数据结构](https://so.csdn.net/so/search?q=数据结构&spm=1001.2101.3001.7020)应该尽可能地在自然边界上对齐。

### 5.2为什么需要内存对齐

1、`便于移植`：不是所有的硬件平台都能访问任意地址上的任意数据的；某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常。

2、`提高处理器访问速度`：对于未对齐的内存，处理器可能需要访问两次内存才能将数据完全读出，而对于**对齐的内存，处理器只需要一次即可**。

对于原因2具体解释一下：

尽管内存是以字节为单位，但是大部分CPU并不是按字节块来存取内存的。它一般会以2的n次方个字节为单位来存取内存，将上述这些存取单位称为**内存存取粒度**。

假设我们的内存存取粒度为4，即CPU只能从地址为4的倍数的内存开始读取数据。 对于int类型的数据，其占4个字节。


![image-20220719141438325](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719141438325.png)

如果没有内存对齐，该数据可能存放在从地址1开始的内存中，即【1-4】，那么处理器需要先读取从地址0开始的连续4个内存单元【0-3】，再读取从地址4开始的连续4个内存单元【4-7】，总共需要CPU访问两次内存才能完全获取该int类型的值。

如果有内存对齐，该数据**只能存放在地址为4**的倍数的内存单元中，比如其在【0-3】，那么只需要读取从地址0开始的连续4个内存单元【0-3】，即CPU访问一次内存就能完全获取该类型的值

### 5.3数据类型大小

数据类型大小
**在32位编译器中**

char:1个字节

short:2个字节

int:4个字节

指针:4个字节(在64位编译器中是8个字节)

float:4个字节

long:4个字节(在64位编译器中是8个字节)

double:8个字节

long long:8个字节


### 5.4内存对齐规则

可以通过预编译命令`#pragma pack(n)`来指定有效对齐值

```
注意：并不是说n就是有效对齐值，可以理解为建议以n为有效对齐值，实际有效对齐值还要根据结构体成员大小来决定，如果n的值比结构体数据成员的大小小才起作用。
```

`有效对齐值N`:表示地址对齐在N上，即数据的存放地址%N=0；

如果没有预编译命令`#pragma pack(n)`指定有效对齐值， 则每个特定平台上的编译器都有自己的默认值n，**通常Linux默认值为4，window默认值为8**

- `结构体数据成员对齐规则`：第一个成员放在offet(偏移量)为0的地方,以后每个数据成员的offset按照**该成员的大小和有效对齐值中较小的整数倍**，即

​                 有效对齐值=*m**i**n*(数据成员类型大小，*n*)

​                *o**f**f**s**e*t=有效对齐值∗整数倍

​       `结构体对齐规则`：在数据成员对齐后，结构体本身也要对齐，其总大小为有效对齐值的整数倍，即

​            *s**i**z**e**o*f(结构体)=有效对齐值∗整数倍

- `结构体作为数据成员`:对于数据成员是结构体的情况，则该结构体成员要从其内部最大数据成员大小和有效对齐值中较小的整数倍地址开始存储

#### 样例1最大类型int

例子1：最大类型int

**上面提到默认情况下，window的n为8，而最大类型为int，所以结构体按照min(n，4)=4字节对齐**

![image-20220719143546288](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719143546288.png)

![image-20220719143709606](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719143709606.png)

```c++
struct S1 {
	char c;//类型长度1<8按1字节对齐；offset为0，存放区间在[0,0]
	int i;//类型长度4<=8按4字节对齐；offset为4(比0大的4的倍数)，存放区间在[4,7]
	short s;//类型长度2<8按2字节对齐；offset为8(比7大的8的倍数)，存放区间在[8,9]
};//进行结构体对齐，将10(9-0+1)提升到4的倍数，最终该结构体大小为12

struct S2 {
	char c;//类型长度1<8按1字节对齐；offset为0，存放区间在[0,0]
	short s;//类型长度2<8按2字节对齐；offset为2(比0大的2的倍数)，存放区间在[2,3]
	int i;//类型长度4<=8按4字节对齐；offset为4(比3大的4的倍数)，存放区间在[4,7]
};//进行整体对齐，将8提升到4的倍数，最终该结构体大小为8


int main()
{
	S1 objectS1 = { 'a','1','2' };
	S2 objectS2 = { 'a','2','1' };
	cout << "sizeof S1：" << sizeof(objectS1) << endl;//输出12
	cout << "sizeof S2：" << sizeof(objectS2) << endl;//输出8
}

```

#### 样例2最大类型double

**默认情况下，最大类型为double，所以结构体按照min(n，8)=8字节对齐**

![image-20220719145151296](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719145151296.png)

```c++
struct S1 {
	char c;//类型长度1<8按1字节对齐；offset为0，存放区间在[0,0]
	double d;//类型长度8<=8按8字节对齐；offset为8，存放区间在[8,15]
	short s;//类型长度2<8按2字节对齐；offset为16，存放区间在[16,17]
};//进行整体对齐，将17提升到8的倍数，最终该结构体大小为24

struct S2 {
	char c;//类型长度1<8按1字节对齐；offset为0，存放区间在[0,0]
	short s;//类型长度2<8按2字节对齐；offset为2，存放区间在[2,3]
	double d;//类型长度8<=8按4字节对齐；offset为8，存放区间在[8,15]
};//进行整体对齐，将16提升到8的倍数，最终该结构体大小为16


int main()
{
	S1 objectS1 = { 'a','1','2' };
	S2 objectS2 = { 'a','2','1' };
	cout << "sizeof S1：" << sizeof(objectS1) << endl;//输出24
	cout << "sizeof S2：" << sizeof(objectS2) << endl;//输出16
}

```

#### 样例3指定一字节对齐值

![image-20220719145841650](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719145841650.png)

```c++
#pragma pack(1)
struct S1 {
	char c;//类型长度1<=1按1字节对齐；offset为0，存放区间在[0,0]
	int i;//类型长度4>1按1字节对齐；offset为1，存放区间在[1,4]
	short s;//类型长度2>1按1字节对齐；offset为5，存放区间在[5,6]
};//类型进行结构体对齐，将7提升到1的倍数，最终该结构体大小为7
int main()
{
	S1 objectS1 = { 'a','1','2' };
	cout << "sizeof S1：" << sizeof(objectS1) << endl;//输出7
}

```

#### 样例4指定二字节对齐值

![image-20220719150336364](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719150336364.png)

```c++
#pragma pack(2)
struct S1 {
	char c;//类型长度1<2按1字节对齐；offset为0，存放区间在[0,0]
	int i;//类型长度4>2按2字节对齐；offset为2，存放区间在[2,5]
	short s;//类型长度2>=2按2字节对齐；offset为6，存放区间在[6,7]
};//进行结构体对齐，将8提升到2的倍数，最终该结构体大小为8

int main()
{
	S1 objectS1 = { 'a','1','2' };
	cout << "sizeof S1：" << sizeof(objectS1) << endl;//输出8
}

```

#### 样例5结构体类型数据成员

![image-20220719150753206](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719150753206.png)

```c++
#pragma pack(4)
struct S1 {
	char c;//类型长度1<4按1字节对齐；offset为0，存放区间在[0,0]
	double d;//类型长度8>4按4字节对齐；offset为4，存放区间在[4,11]
	short s;//类型长度2<4按2字节对齐；offset为12，存放区间在[12,13]
};//进行整体对齐，将14提升到4的倍数，最终该结构体大小为16

struct S2 {
	char c;//类型长度1<4按1字节对齐；offset为0，存放区间在[0,0]
	double s;//类型长度8>4按4字节对齐；offset为4，存放区间在[4,11]
	S1 m_S1;//该结构体内最大数据成员类型是double,所以可以把S1看成double来计算，但8>4,所以按4字节对齐；offset为12，存放区间在[12,27]
	short arr[3];//类型长度2<4按2字节对齐；offset为28，存放区间在[28,33]
};//进行整体对齐，将34提升到4的倍数，最终该结构体大小为36


int main()
{
	S1 objectS1 = { 'a','1','2' };
	S2 objectS2 = { 'a','2',objectS1,{1,2,3} };
	cout << "sizeof S1：" << sizeof(objectS1) << endl;//输出16
	cout << "sizeof S2：" << sizeof(objectS2) << endl;//输出36
}

```

**注意**：结构体中的结构体类型的成员变量也要进行整体对齐

### 5.5什么情况下需要内存对齐

1. 该数据需要直接写入文件
2. 该数据需要通过网络传给其他程序

## 6.TCP

***\*TCP协议：三次握手，四次挥手原理\****

三次握手：两个人打电话
1:在吗
2：在哦
1：好的

四次挥手：挂电话
1：我要挂了
2：好的
2：我也挂了
1：好的（1挂电话，然后2也挂了电话）

## 7.序列化

### 一、[序列化](https://so.csdn.net/so/search?q=序列化&spm=1001.2101.3001.7020)

在编写应用程序的时候往往需要将程序的某些数据存储在内存中，然后将其写入某个文件或是将它传输到网络中的另一台计算机上以实现通讯。这个**将** **程序数据转化成能被存储并传输的格式**的过程被称为“序列化”（Serialization），而它的逆过程则可被称为“反序列化” （Deserialization）。
简单来说，序列化就是将对象实例的状态转换为可保持或传输的格式的过程。与序列化相对的是反序列化，它根据流重构对象。这两个过程结合起来，可以轻 松地**存储和传输数据**。例如，可以序列化一个对象，然后使用 HTTP 通过 Internet 在客户端和服务器之间传输该对象。
序列化：将对象变成字节流的形式传出去。
反序列化：从字节流恢复成原来的对象。
简单来说，对象序列化通常用于两个目的：
1）将对象存储于硬盘上 ，便于以后反序列化使用。
2）在网络上传送对象的字节序列。

### 二、C++对象序列化方法

#### 1、Boost.Serialization

Boost.Serialization可以创建或重建程序中的**等效结构**，并保存为二进制数据、文本数据、XML或者有用户自定义的其他文件。该库具有以下吸引人的特性：
1). 代码可移植（实现仅依赖于ANSI C++）
2). 深度指针保存与恢复
3). 可以序列化STL容器和其他常用模版库
4). 数据可移植
5). 非入侵性
boost::serialization 基于 boost::archive 来完成任意复杂数据结构的序列化，boost::archive提供两个实现类来完成序列化、反序列化操作：
boost::archive::text_oarchive     序列化数据，也称为：输出、保存（save）
boost::archive::text_iarchive      反序列化数据，也称为：输入、载入（load）
也可以使用二进制格式:binary_oarchive, binary_iarchive
&操作符 
序列化操作使用 << 或者 & 操作符将数据存入text_oarchive中：
ar << data;
ar & data;
反序列化操作使用 >> 或者 & 操作符从text_iarchive中读取数据：
ar >> data;
ar & data;
为什么要引入&操作符？很简单，&操作符可以同时用于序列化和反序列化操作，这样就只需要提供一个 serialize 模板函数就可以同时用于两种操作，具体执行哪种由ar的类型（模版参数类型）决定；如果ar是text_oarchive类型则是序列化，如果ar是text_iarchive类型则是反序列化。

#### 2、protobuf

Google Protocol Buffers (GPB)是Google内部是用的数据编码方式，旨在用来代替XML进行数据交换。可用于数据序列化与反序列化。主要特性有：
1). 高效
2). 语言中立（Cpp, Java, Python）
3). 可扩展

#### 3.MFC Serialization

 Windows平台下可使用MFC中的序列化方法。MFC 对 CObject 类中的序列化提供内置支持。因此，所有从 CObject 派生的类都可利用 CObject 的序列化协议。
   继承CObject的类，实现序列化方法Serialize(CArchive& ar)，添加序列化宏DECLARE_SERIAL。编写序列化与反序列化的对象。

几种方法对比：  
Google Protocol Buffers效率较高，但是数据对象必须预先定义，并使用protoc编译，适合要求效率，允许自定义类型的内部场合使用。Boost.Serialization 使用灵活简单，而且支持标准C++容器。相比而言，MFC的效率较低，但是结合VisualStdio平台使用最为方便。

参考资料
http://www.cnblogs.com/lanxuezaipiao/p/3703988.html
http://blog.csdn.net/liyongofdm/article/details/7650380

[(32条消息) C++之序列化_镇天雷帝的博客-CSDN博客_c++ 序列化](https://blog.csdn.net/qq_41980769/article/details/107250278)

## 8.多线程

### 1.什么是C++多线程？

线程：线程是**操作系统**能够进行运算调度的**最小单位**，它被包含在进程之中，进程包含一个或者多个线程。进程可以理解为完成一件事的完整解决方案，而线程可以理解为这个解决方案中的的一个步骤，可能这个解决方案就这只有一个步骤，也可能这个解决方案有多个步骤。
多线程：多线程是实现并发（并行）的手段，并发（并行）即**多个线程同时执行**，一般而言，多线程就是把执行一件事情的完整步骤拆分为多个子步骤，然后使得这**多个步骤同时执行**。
C++多线程：（简单情况下）C++多线程使用多个函数实现各自功能，然后将不同函数生成不同线程，并同时执行这些线程（不同线程可能存在一定程度的执行先后顺序，但总体上可以看做同时执行）。

### 2.C++多线程基础知识

#### 2.1创建线程

首先要引入头文件#include<thread>(C++11的标准库中提供了多线程库)，该头文件中定义了thread类，创建一个线程即实例化一个该类的对象，实例化对象时候调用的构造函数需要传递一个参数，**该参数就是函数名**，thread th1(proc1)；**如果传递进去的函数本身需要传递参数，实例化对象时将这些参数按序写到函数名后面，thread th1(proc1,a,b)**;//**只要创建了线程对象（传递“函数名/可调用对象”作为参数的情况下），线程就开始执行**（std::thread 有一个无参构造函数重载的版本，不会创建底层的线程）。
有两种**线程阻塞方法join()与detach()**，阻塞线程的目的是**调节各线程的先后执行顺序**，这里重点讲join()方法，不推荐使用detach()，detach()使用不当会发生引用对象失效的错误。当线程启动后，**一定要在和线程相关联的thread对象销毁前，对主线程运用join()或者detach()**。
join(), 当前线程暂停, **等待指定的线程执行结束后, 当前线程再继续**。th1.join()，即**该语句所在的线程**（该语句写在main（）函数里面，即主线程内部）暂停，等待指定线程（指定线程为th1）执行结束后，主线程再继续执行。**//等待调用者的线程执行结束**
整个过程就相当于你在做某件事情，中途你让老王帮你办一个任务（你办的时候他同时办）（创建线程1），又叫老李帮你办一件任务（创建线程2），现在你的这部分工作做完了，需要用到他们的结果，只需要等待老王和老李处理完（join()，阻塞主线程），等他们把任务做完（子线程运行结束），你又可以开始你手头的工作了（主线程不再阻塞）。

```c++
#include<iostream>
#include<thread>
using namespace std;
void proc(int a)
{
    cout << "我是子线程,传入参数为" << a << endl;
    cout << "子线程中显示子线程id为" << this_thread::get_id()<< endl;
}
int main()
{
    cout << "我是主线程" << endl;
    int a = 9;
    thread th2(proc,a);//第一个参数为函数名，第二个参数为该函数的第一个参数，如果该函数接收多个参数就依次写在后面。此时线程开始执行。
    cout << "主线程中显示子线程id为" << th2.get_id() << endl;
    th2.join()；//此时主线程被阻塞直至子线程执行结束。
    return 0;
}
```

#### 2.2 互斥量使用

什么是互斥量？

这样比喻，单位上有一台打印机（**共享数据**a），你要用打印机（线程1要操作数据a），同事老王也要用打印机(线程2也要操作数据a)，但是打印机同一时间只能给一个人用，此时，**规定不管是谁，在用打印机之前都要向领导申请许可证（lock）**，用完后再向领导归还许可证(unlock)，许可证总共只有一个,没有许可证的人就等着在用打印机的同事用完后才能申请许可证(阻塞，线程1lock互斥量后其他线程就无法lock,只能等线程1unlock后，其他线程才能lock)，那么，这个许可证就是互斥量。互斥量保证了使用打印机这一过程不被打断。

程序实例化mutex对象m,线程调用成员函数m.lock()会发生下面 3 种情况：
(1)如果该互斥量当前未上锁，则调用线程将该互斥量锁住，直到调用unlock()之前，该线程一直拥有该锁。
(2)如果该互斥量当前被锁住，则**调用线程被阻塞**,直至该互斥量被解锁。

<font color='green'>上锁不单是防止数据被同时占用，还能约束线程行为</font>

<font color='green'>同一个主线程的子线程之间没有先后执行之分，但是主线程必定是最先执行的，除非用了join和detach</font>

互斥量怎么使用？

首先需要\#include<mutex>

lock()与unlock():

```c++
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;//实例化m对象，不要理解为定义变量
void proc1(int a)
{
    m.lock();
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
    m.unlock();
}

void proc2(int a)
{
    m.lock();
    cout << "proc2函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 1 << endl;
    m.unlock();
}
int main()
{
    int a = 0;
    thread proc1(proc1, a);
    thread proc2(proc2, a);
    proc1.join();
    proc2.join();
    return 0;
}
```

不推荐实直接去调用成员函数lock()，因为如果忘记unlock()，将导致锁无法释放，**使用lock_guard或者unique_lock能避免忘记解锁**这种问题。

lock_guard():
其原理是：声明一个**局部的lock_guard对象**，在其构造函数中进行加锁，在其析构函数中进行解锁。最终的结果就是：**创建即加锁，作用域结束自动解锁**。从而使用lock_guard()就可以替代lock()与unlock()。
通过设定作用域，使得lock_guard在合适的地方被析构（在互斥量锁定到互斥量解锁之间的代码叫做临界区（需要互斥访问共享资源的那段代码称为临界区），临界区范围应该尽可能的小，即lock互斥量后应该尽早unlock），通过使用{}来调整作用域范围，可使得互斥量m在合适的地方被解锁：

```c++
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;//实例化m对象，不要理解为定义变量
void proc1(int a)
{
    lock_guard<mutex> g1(m);//用此语句替换了m.lock()；lock_guard传入一个参数时，该参数为互斥量，此时调用了lock_guard的构造函数，申请锁定m
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
}//此时不需要写m.unlock(),g1出了作用域被释放，自动调用析构函数，于是m被解锁

void proc2(int a)
{
    {
        lock_guard<mutex> g2(m);
        cout << "proc2函数正在改写a" << endl;
        cout << "原始a为" << a << endl;
        cout << "现在a为" << a + 1 << endl;
    }//通过使用{}来调整作用域范围，可使得m在合适的地方被解锁
    cout << "作用域外的内容3" << endl;
    cout << "作用域外的内容4" << endl;
    cout << "作用域外的内容5" << endl;
}
int main()
{
    int a = 0;
    thread proc1(proc1, a);
    thread proc2(proc2, a);
    proc1.join();
    proc2.join();
    return 0;
}
```

<font size=5>上锁只能确保数据不被同时占用，子线程之间的执行前后是随机的(大概率符合定义顺序)</font>

lock_gurad也可以传入两个参数，第一个参数为adopt_lock标识时，表示 构造函数中不再进行互斥量锁定，因此此时需要提前手动锁定。

```C++
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;//实例化m对象，不要理解为定义变量
void proc1(int a)
{
    m.lock();//手动锁定
    lock_guard<mutex> g1(m,adopt_lock);
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
}//自动解锁

void proc2(int a)
{
    lock_guard<mutex> g2(m);//自动锁定
    cout << "proc2函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 1 << endl;
}//自动解锁

int main()
{
    int a = 0;
    thread proc1(proc1, a);
    thread proc2(proc2, a);
    proc1.join();
    proc2.join();
    return 0;
}
```

unique_lock:
unique_lock类似于lock_guard,只是unique_lock用法更加丰富，同时支持lock_guard()的原有功能。
使用lock_guard后不能手动lock()与手动unlock();**使用unique_lock后可以手动lock()与手动unlock()**;
unique_lock的第二个参数，除了可以是adopt_lock,还可以是try_to_lock与defer_lock;
try_to_lock: **尝试去锁定，得保证锁处于unlock的状态**,然后尝试现在能不能获得锁；尝试用mutx的lock()去锁定这个mutex，但如果没有锁定成功，会立即返回，不会阻塞在那里  <font color='red'>//尝试获取共享值</font>
defer_lock: **始化了一个没有加锁的mutex;**<font color='red'>//使unique_lock变量拥有互斥量的权限(开关锁)，并且在作用域等同于第一个参数的mutex</font>

|                      | lock_guard     | unique_lock                           |
| -------------------- | -------------- | ------------------------------------- |
| 手动lock与手动unlock | 不支持         | 支持                                  |
| 参数                 | 支持adopt_lock | 支持adopt_lock/try_to_lock/defer_lock |

```c++
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;
void proc1(int a)
{
    unique_lock<mutex> g1(m, defer_lock);//始化了一个没有加锁的mutex
    cout << "不拉不拉不拉" << endl;
    g1.lock();//手动加锁，注意，不是m.lock();注意，不是m.lock();注意，不是m.lock()
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
    g1.unlock();//临时解锁
    cout << "不拉不拉不拉"  << endl;
    g1.lock();
    cout << "不拉不拉不拉" << endl;
}//自动解锁

void proc2(int a)
{
    unique_lock<mutex> g2(m,try_to_lock);//尝试加锁，但如果没有锁定成功，会立即返回，不会阻塞在那里；
    cout << "proc2函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 1 << endl;
}//自动解锁
int main()
{
    int a = 0;
    thread proc1(proc1, a);
    thread proc2(proc2, a);
    proc1.join();
    proc2.join();
    return 0;
}
//unique_lock所有权的转移

mutex m;
{  
    unique_lock<mutex> g2(m,defer_lock);
    unique_lock<mutex> g3(move(g2));//所有权转移，此时由g3来管理互斥量m
    g3.lock();
    g3.unlock();
    g3.lock();
}
```

#### 2.3condition_variable

需要#include<condition_variable>;
**wait(locker):**在线程被阻塞时，该函数会自动调用 locker.unlock() 释放锁，使得其他被阻塞在锁竞争上的线程得以继续执行。另外，一旦当前线程获得通知(通常是另外某个线程调用 notify_* 唤醒了当前线程)，wait() 函数此时再自动调用 locker.lock()。
notify_all():唤醒所有等待的线程
notify_once():随机唤醒一个等待的线程

condition_variable类似于[信号量](https://so.csdn.net/so/search?q=信号量&spm=1001.2101.3001.7020)机制，实现了线程的等待和唤醒。

```c++
#include<iostream>  
#include<thread>  
#include<mutex>  
#include<condition_variable>  
#include<chrono>
using namespace std;
mutex m;
condition_variable cond;
int LOOP = 10;
int flag = 0;

void fun(int id) {
	for (int i = 0; i < LOOP; i++) {
		unique_lock<mutex> lk(m);	//加锁
		//写法1，while循环比较，多次唤醒时，只要不满足条件就阻塞，if只判断一次会出错
		/*while (id != flag)
			cond.wait(lk);*/

			//写法2，实现原理和上面一样 ，id != flag时会阻塞，唤醒时继续判断，id == flag才会唤醒成功
		cond.wait(lk, [=]() {
			return id == flag;
			});
		cout << (char)('A' + id) << " ";
		flag = (flag + 1) % 3;
		cond.notify_all();
	}
}
int main() {
	thread A(fun, 0);
	thread B(fun, 1);
	thread C(fun, 2);

	A.join();
	B.join();
	C.join();
	cout << endl;
	cout << "main end" << endl;
	return 0;
}

```

##### 2.3.1semaphore源码

```c++
#pragma once
#include<mutex>
#include<condition_variable>
class semaphore {
public:
	semaphore(long count = 0) :count(count) {}
	void wait() {
		std::unique_lock<std::mutex>lock(mx);
		cond.wait(lock, [&]() {return count > 0; });
		--count;
	}
	void signal() {
		std::unique_lock<std::mutex>lock(mx);
		++count;
		cond.notify_one();
	}

private:
	std::mutex mx;
	std::condition_variable cond;
	long count;
};

```



#### 2.3异步线程

##### 2.3.1什么是异步线程(目的)

异步操作的目的是为了提高响应的并发量和控制访问的安全性以及健壮性。说的再直白一些，就是**把访问过程，处理过程和响应过程分离**。

异步是相对于同步来说，同步相当于一问一答，必须实现，假如你去银行办理业务，你问柜台的小姐姐一句话，半天才回复你，估计你就怒了。但是如果你去带着一些木料去定作家具，就不愿意等在那儿，而愿意等弄好了，给你打个电话，你再去取。其实好多人学了很多年计算机编程，并没有明白一个道理，计算机编程其实就是现实世界的一种映射，或者准确一点儿说，是现实世界一部分的可以数字逻辑化的映射。<font color='green'>交出任务，等任务完成再获取结果</font>

为了简化异步编程，提供std::async这个模板类。相应的，为了进行异步多线程间的交互提供了**std::future,std::promise,std::packaged_task**这几个模板。

简单来说**std::promise提供初步的多线程间的数据操作，std::future可以得到这些操作的结果**，而在实际应用中不仅有简单的数值操作，**有时候还需要对函数的运行进行处理（特别是函数运行结果）**，这时候就得需要std::packaged_task了。其实这两者可以从另外一个抽象层次上统一起来，就是这个操作的对象既可以是基础值也可以是一个函数，动态映射即可，但是，这不仅会消耗性能，还会增加应用的复杂度，这会不会是c++标准制定者有所考虑呢？不得而知。
而std::async又可以看作对上述三个的进一步的抽象，让他们应用起来更简单清晰。它的参数主要说明如下：
**std::launch::deferred延迟调用，延迟到future对象调用get()或者wait()的时候才执行函数，否则不会执行。**
**std::launch::async：强制这个异步任务在新线程上执行，即系统要创建一个新线程来执行相关函数;**
std::launch::async |std::launch::deferred “ |”符号代表二选一，都有可能 。
不带参数；只有函数名；默认 std::launch::async |std::launch::deferred，看具体平台的默认设置。

而实际情况中，线程的同步和异步，都是要针对具体的应用场景来说的。不过随着应用场景的越来越复杂，高并发和大数据量通信带的结果就是，异步编程越来越复杂，应用也越来越广泛。为了解决这个问题不同的语言和框架都提出了的解决方式，除最初的线程模拟，到封装，再到后来Go语言等提出的协程等，都朝着一个目标前进，那就是不断的减少异步编程的复杂度。


##### 2.3.2异步线程的实现

需要#include

**async与future：**
async是一个函数模板，用来**启动一个异步任务**，它返回一个future类模板对象，future对象起到了**占位的作用**，刚实例化的future是没有储存值的，但在调用future对象的get()成员函数时，**主线程会被阻塞直到异步线程执行结束**，并把返回结果传递给future，即通过FutureObject.get()获取函数返回值。

相当于你去办政府办业务（主线程），把资料交给了前台，前台安排了人员去给你办理（async创建子线程），前台给了你一个单据（future对象），说你的业务正在给你办（子线程正在运行），等段时间你再过来凭这个单据取结果。过了段时间，你去前台取结果，但是结果还没出来（子线程还没return），你就在前台等着（阻塞），直到你拿到结果（get()）你才离开（不再阻塞）。<font color='brown'>就是比join多了一个返回值,顺便创建一个线程</font>

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include<future>
#include<Windows.h>
using namespace std;
double t1(const double a, const double b)
{
	double c = a + b;
	Sleep(3000);//假设t1函数是个复杂的计算过程，需要消耗3秒
	return c;
}

int main() 
{
	double a = 2.3;
	double b = 6.7;
	future<double> fu = async(t1, a, b);//创建异步线程线程，并将线程的执行结果用fu占位；
	cout << "正在进行计算" << endl;
	cout << "计算结果马上就准备好，请您耐心等待" << endl;
	cout << "计算结果：" << fu.get() << endl;//阻塞主线程，直至异步线程return
        //cout << "计算结果：" << fu.get() << endl;//取消该语句注释后运行会报错，因为一个future对象的get()方法只能调用一次。
	return 0;
}
```

##### 2.3.2shared_future

future与shared_future的用途都是为了**占位**，但是两者有些许差别。
future的get()成员函数是转移数据所有权;shared_future的get()成员函数是复制数据。

```c++
  
get的源码
    
const _Ty& get() const {
        return this->_Get_value();
    }//shared_future的get函数
   
   _Ty get() {
        future _Local{_STD move(*this)};//直接用move转移了所有权
        return _STD move(_Local._Get_value());
    }//future的get函数
#define _STD ::std::


```

因此：
**future对象的get()只能调用一次**；无法实现多个线程等待同一个异步线程，一旦其中一个线程获取了异步线程的返回值，其他线程就无法再次获取。
**shared_future对象的get()可以调用多次**；可以实现多个线程等待同一个异步线程，每个线程都可以获取异步线程的返回值。

|              | **future** | **shared_future** |
| ------------ | ---------- | ----------------- |
| get语义      | 转移       | 赋值              |
| 可否多次调用 | 否         | 可以              |

##### 2.3.3应用

```C++
#include <iostream>
#include <thread>
#include <functional>
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
#include <future>
#include <string>
#include <mutex>
#include <chrono>
 
int func_async()
{
    std::cout << "run async" << std::endl;
                //延时为了模拟异步操作
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    return 100;
}
 
int TestAsync()
{
    //测试一下不同参数效果
    auto h = std::async(std::launch::async/*std::launch::deferred*/, func_async);
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    std::cout << "strat async get :" << h.get()<< std::endl;
 
    return 0;
}
 
void func_promise(std::promise<int>& p) {
    std::this_thread::sleep_for(std::chrono::milliseconds(3000));
    p.set_value(100);                         
}
 
int TestPromise()
{
    std::promise<int> p;
    std::thread t(func_promise, std::ref(p));
 
    std::future<int> f = p.get_future();
    std::cout << "get promis future is:" << f.get() << std::endl;
    t.join();
 
    return 0;
}
int func_task(int a, int b) {
    std::this_thread::sleep_for(std::chrono::seconds(3));
 
    return a<b?a:b;
}
 
int TestPackaged()
{
    std::packaged_task<int(int, int)> task(func_task);
    std::future<int> f = task.get_future();
    std::thread t(std::move(task), 20, 10);
    std::cout <<"get task future is:"<< f.get() << std::endl;          
    t.join();
    return 0;
    
}
 
int main()
{
    TestAsync();
    TestPromise();
    TestPackaged();
    return 0;
}
```

##### 2.3.4官网例子

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
#include <future>
#include <string>
#include <mutex>
 
std::mutex m;
struct X {
    void foo(int i, const std::string& str) {
        std::lock_guard<std::mutex> lk(m);
        std::cout << str << ' ' << i << '\n';
    }
    void bar(const std::string& str) {
        std::lock_guard<std::mutex> lk(m);
        std::cout << str << '\n';
    }
    int operator()(int i) {
        std::lock_guard<std::mutex> lk(m);
        std::cout << i << '\n';
        return i + 10;
    }
};
 
template <typename RandomIt>
int parallel_sum(RandomIt beg, RandomIt end)
{
    auto len = end - beg;
    if (len < 1000)
        return std::accumulate(beg, end, 0);
 
    RandomIt mid = beg + len/2;
    auto handle = std::async(std::launch::async,
                             parallel_sum<RandomIt>, mid, end);
    int sum = parallel_sum(beg, mid);
    return sum + handle.get();
}
 
int main()
{
    std::vector<int> v(10000, 1);
    std::cout << "The sum is " << parallel_sum(v.begin(), v.end()) << '\n';
 
    X x;
    // Calls (&x)->foo(42, "Hello") with default policy:
    // may print "Hello 42" concurrently or defer execution
    auto a1 = std::async(&X::foo, &x, 42, "Hello");
    // Calls x.bar("world!") with deferred policy
    // prints "world!" when a2.get() or a2.wait() is called
    auto a2 = std::async(std::launch::deferred, &X::bar, x, "world!");
    // Calls X()(43); with async policy
    // prints "43" concurrently
    auto a3 = std::async(std::launch::async, X(), 43);
    a2.wait();                     // prints "world!"
    std::cout << a3.get() << '\n'; // prints "53"
} // if a1 is not done at this point, destructor of a1 prints "Hello 42" here

```

#### 2.4原子类型automic和原子操作

**原子操作指“不可分割的操作”**；也就是说这种操作状态要么是完成的，要么是没完成的。**互斥量的加锁一般是针对一个代码段，而原子操作针对的一般都是一个变量**。<font color='red'>必须一次性完成</font>
automic是一个模板类，使用该模板类实例化的对象，提供了一些保证原子性的成员函数来实现共享数据的常用操作。

可以这样理解：
在以前，定义了一个共享的变量(int i=0)，多个线程会操作这个变量，那么每次操作这个变量时，都是用lock加锁，操作完毕使用unlock解锁，以保证线程之间不会冲突；
现在，实例化了一个类对象(automic I=0)来代替以前的那个变量，每次操作这个对象时，就不用lock与unlock，这个对象自身就具有原子性，以保证线程之间不会冲突。

automic对象提供了常见的原子操作（通过调用成员函数实现对数据的原子操作）：
**store是原子写操作，load是原子读操作。exchange是于两个数值进行交换的原子操作**。
**即使使用了automic，也要注意执行的操作是否支持原子性**。一般atomic原子操作，针对++，–，+=，-=，&=，|=，^=是支持的。
如果从对象读取值的加载操作是 原子 的，而且对这个对象的**所有修改操作**也是 原子 的，
那么加载操作得到的值要么是对象的初始值，要么是某次修改操作存入的值。

另一方面，非原子操作可能会被另一个线程观察到只完成一半。
如果这个操作是一个存储操作，那么其他线程看到的值，可能既不是存储前的值，也不是存储的值，而是别的什么值。
如果这个非原子操作是一个加载操作，它可能先取到对象的一部分，然后值被另一个线程修改，然后它再取到剩余的部分，
所以它取到的既不是第一个值，也不是第二个值，而是两个值的某种组合。
正如第三章所讲的，这一下成了一个容易出问题的竞争冒险，
但在这个层面上它可能就构成了 **数据竞争** （见5.1节），就成了未定义行为。

在C++中，多数时候你需要一个原子类型来得到原子的操作，我们来看一下这些类型。

##### 2.4.1标准原子类型

标准 原子类型 定义在头文件<atomic>中。
这些类型上的所有操作都是原子的，**在语言定义中只有这些类型的操作是原子的**，不过你可以用互斥锁来 模拟 原子操作。
实际上，标准原子类型自己的实现就可能是这样模拟出来的：
它们(几乎)都有一个is_lock_free()成员函数，//不能被锁
这个函数让用户可以查询某原子类型的操作是直接用的原子指令(x.is_lock_free()返回true)，
还是编译器和库内部用了一个锁(x.is_lock_free()返回false)。

只有std::atomic_flag类型不提供is_lock_free()成员函数。这个类型是一个简单的布尔标志，并且在这种类型上的操作都需要是无锁的；当你有一个简单无锁的布尔标志时，你可以使用其实现一个简单的锁，并且实现其他基础的原子类型。当你觉得“真的很简单”时，就说明：在std::atomic_flag对象明确初始化后，做查询和设置(使用test_and_set()成员函数)，或清除(使用clear()成员函数)都很容易。这就是：无赋值，无拷贝，没有测试和清除，没有其他任何操作。

剩下的原子类型都可以通过**特化std::atomic<>类型模板而访问到**，并且拥有更多的功能，但可能**不都是无锁的**(如之前解释的那样)。在最流行的平台上，期望原子变量都是**无锁的内置类型**(例如std::atomic<int>和std::atomic<void*>)，但这没有必要。你在后面将会看到，每个特化接口所反映出的类型特点；位操作(如&=)就没有为普通指针所定义，所以它也就不能为原子指针所定义。

除了直接使用std::atomic<>类型模板外，你可以使用在表5.1中所示的原子类型集。由于历史原因，原子类型已经添加入C++标准中，这些备选类型名可能参考相应的std::atomic<>特化类型，或是特化的基类。**在同一程序中混合使用备选名与std::atomic<>特化类名，会使代码的移植大打折扣**。


| **原子类型**    | **相关特化类**                  |
| --------------- | ------------------------------- |
| atomic_bool     | std::atomic<bool>               |
| atomic_char     | std::atomic<char>               |
| atomic_schar    | std::atomic<signed char>        |
| atomic_uchar    | std::atomic<unsigned char>      |
| atomic_int      | std::atomic<int>                |
| atomic_uint     | std::atomic<unsigned>           |
| atomic_short    | std::atomic<short>              |
| atomic_ushort   | std::atomic<unsigned short>     |
| atomic_long     | std::atomic<long>               |
| atomic_ulong    | std::atomic<unsigned long>      |
| atomic_llong    | std::atomic<long long>          |
| atomic_ullong   | std::atomic<unsigned long long> |
| atomic_char16_t | std::atomic<char16_t>           |
| atomic_char32_t | std::atomic<char32_t>           |
| atomic_wchar_t  | std::atomic<wchar_t>            |

C++标准库不仅提供基本原子类型，还定义了与原子类型对应的非原子类型，就如同标准库中的`std::size_t`。如表5.2所示这些类型:

<font size=5>std::size_t</font>

- std::size_t可以**存放下理论上可能存在的对象的最大大小**，该对象可以是任何类型，包括数组。自C++14起，大小无法以std::size_t表示的类型是非良构的。
- **std::size_t 通常被用于数组索引和循环计数**
  - 使用其它类型来进行数组索引操作的程序可能会在某些情况下出错，例如在 64 位系统中使用 unsigned int 进行索引时，如果索引号超过 UINT_MAX 或者依赖于 32 位取模运算的话，程序就会出错。
  - 在对诸如 std::string、std::vector 等 C++ 容器进行索引操作时，正确的类型是该容器的成员 typedef size_type，而该类型通常被定义为与 std::size_t 相同

```c++
#include <cstddef>
#include <iostream>

int main()
{
    const std::size_t  N  = 10;
    int *a = new int [N];

    for(std::size_t n = 0; n < N; n++){
        a[n] = n;
    }

    for(std::size_t n = N; n-- > 0;){
        std::cout << a[n] << " ";
    }

    delete [] a;
}
源码:
 typedef unsigned __int64 size_t;
```

表5.2 标准原子类型定义(typedefs)和对应的内置类型定义(typedefs)

| **原子类型定义**      | **标准库中相关类型定义** |
| --------------------- | ------------------------ |
| atomic_int_least8_t   | int_least8_t             |
| atomic_uint_least8_t  | uint_least8_t            |
| atomic_int_least16_t  | int_least16_t            |
| atomic_uint_least16_t | uint_least16_t           |
| atomic_int_least32_t  | int_least32_t            |
| atomic_uint_least32_t | uint_least32_t           |
| atomic_int_least64_t  | int_least64_t            |
| atomic_uint_least64_t | uint_least64_t           |
| atomic_int_fast8_t    | int_fast8_t              |
| atomic_uint_fast8_t   | uint_fast8_t             |
| atomic_int_fast16_t   | int_fast16_t             |
| atomic_uint_fast16_t  | uint_fast16_t            |
| atomic_int_fast32_t   | int_fast32_t             |
| atomic_uint_fast32_t  | uint_fast32_t            |
| atomic_int_fast64_t   | int_fast64_t             |
| atomic_uint_fast64_t  | uint_fast64_t            |
| atomic_intptr_t       | intptr_t                 |
| atomic_uintptr_t      | uintptr_t                |
| atomic_size_t         | size_t                   |
| atomic_ptrdiff_t      | ptrdiff_t                |
| atomic_intmax_t       | intmax_t                 |
| atomic_uintmax_t      | uintmax_t                |

对于`std::atomic<T>`模板，使用对应的T类型去特化模板的方式，要好于使用别名的方式

通常，标准原子类型是不能拷贝和赋值，他们没有拷贝构造函数和拷贝赋值操作。

但是，因为可以隐式转化成对应的内置类型，所以这些类型依旧支持赋值，可以使用load()和store()成员函数，exchange()、compare_exchange_weak()和compare_exchange_strong()。它们都支持复合赋值符：+=, -=, *=, |= 等等。并且使用整型和指针的特化类型还支持 ++ 和 --。当然，这些操作也有功能相同的成员函数所对应：fetch_add(), fetch_or() 等等。**赋值操作和成员函数的返回值要么是被存储的值(赋值操作)，要么是操作前的值(命名函数)**。这就能避免赋值操作符返回引用。为了获取存储在引用的值，代码需要执行单独的读操作，从而允许另一个线程在赋值和读取进行的同时修改这个值，这也就为条件竞争打开了大门。
std::atomic<>类模板不仅仅一套特化的类型，其作为一个原发模板也可以使用用户定义类型创建对应的原子变量。因为，它是一个通用类模板，操作被限制为**load(),store()(赋值和转换为用户类型), exchange(), compare_exchange_weak()和compare_exchange_strong()。**

<font color='green'>//用户自定义类型创建的原子变量只可以执行这些操作</font>

**每种函数类型的操作都有一个可选内存排序参数**，这个参数可以用来指定所需存储的顺序。在5.3节中，会对存储顺序选项进行详述。现在，只需要知道操作分为三类：

Store操作，可选如下顺序：memory_order_relaxed, memory_order_release, memory_order_seq_cst。

Load操作，可选如下顺序：memory_order_relaxed, memory_order_consume, memory_order_acquire, memory_order_seq_cst。

Read-modify-write(读-改-写)操作，可选如下顺序：memory_order_relaxed, memory_order_consume, memory_order_acquire, memory_order_release, memory_order_acq_rel, memory_order_seq_cst。

所有操作的默认顺序都是**memory_order_seq_cst**。

现在，让我们来看一下每个标准原子类型进行的操作，就从`std::atomic_flag`开始吧。

##### 2.4.2std::atomic_flag的相关操作

std::atomic_flag是最简单的标准原子类型，它表示了一个布尔标志。这个类型的对象可以在两个状态间切换：**设置和清除**。它就是那么的简单，只作为一个构建块存在。我从未期待这个类型被使用，除非在十分特别的情况下。正因如此，它将作为讨论其他原子类型的起点，因为它会展示一些原子类型使用的通用策略。

`std::atomic_flag`类型的对象必须被ATOMIC_FLAG_INIT初始化。初始化标志位是“清除”状态。这里没得选择；这个标志**总是初始化为“清除”**：

```c++
std::atomic_flag f = ATOMIC_FLAG_INIT;
```

这适用于任何对象的声明，并且可在任意范围内。它是**唯一**需要以如此特殊的方式初始化的原子类型，但它也是**唯一保证无锁**的类型。如果std::atomic_flag是静态存储的，那么就要保证其是静态初始化的，也就意味着没有初始化顺序问题；**在首次使用时，其都需要初始化**。

当你的标志对象已初始化，那么你只能做三件事情：销毁，清除或设置(查询之前的值)。这些事情对应的函数分别是：clear()成员函数，和test_and_set()成员函数。clear()和test_and_set()成员函数可以指定好内存顺序。clear()是一个存储操作，所以不能有memory_order_acquire或（acquire:获得）memory_order_acq_rel语义，但是test_and_set()是一个“读-改-写”操作，所有可以应用于任何内存顺序标签。每一个原子操作，默认的内存顺序都是memory_order_seq_cst。例如：

```c++
f.clear(std::memory_order_release);  // 1
bool x=f.test_and_set();  // 2
```





#### 2.5线程池

##### 2.5.1不采用线程池时:

创建线程 -> 由该线程执行任务 -> 任务执行完毕后销毁线程。即使需要使用到大量线程，每个线程都要按照这个流程来创建、执行与销毁。

**虽然创建与销毁线程消耗的时间 远小于 线程执行的时间，但是对于需要频繁创建大量线程的任务，创建与销毁线程 所占用的时间与CPU资源也会有很大占比。**

**为了减少创建与销毁线程所带来的时间消耗与资源消耗，因此采用线程池的策略**

程序启动后，**预先创建**一定数量的**线程**放入空闲队列中，这些线程都是处于阻塞状态，基本不消耗CPU，只占用较小的内存空间。

接收到任务后，线程池选择一个空闲线程来执行此任务。

任务执行完毕后，不销毁线程，线程继续保持在池中**等待下一次的任务**。

##### 2.5.2**线程池所解决的问题**

(1) 需要频繁创建与销毁大量线程的情况下，减少了创建与销毁线程带来的时间开销和CPU资源占用。**（省时省力）**

(2) 实时性要求较高的情况下，由于大量线程预先就创建好了，接到任务就能马上从线程池中调用线程来处理任务，略过了创建线程这一步骤，提高了实时性。**（实时）**

#### 2.6实例

<font size=5>生产者消费者问题</font>

```c++
/*
生产者消费者问题
*/
#include <iostream>
#include <deque>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <Windows.h>
using namespace std;

deque<int> q;
mutex mu;
condition_variable cond;
int c = 0;//缓冲区的产品个数

void producer() { 
	int data1;
	while (1) {//通过外层循环，能保证生成用不停止
		if(c < 3) {//限流
			{
				data1 = rand();
				unique_lock<mutex> locker(mu);//锁
				q.push_front(data1);
				cout << "存了" << data1 << endl;
				cond.notify_one();  // 通知取
				++c;
			}
			Sleep(500);
		}
	}
}

void consumer() {
	int data2;//data用来覆盖存放取的数据
	while (1) {
		{
			unique_lock<mutex> locker(mu);
			while(q.empty())
				cond.wait(locker); //wati()阻塞前先会解锁,解锁后生产者才能获得锁来放产品到缓冲区；生产者notify后，将不再阻塞，且自动又获得了锁。
			data2 = q.back();//取的第一步
			q.pop_back();//取的第二步
			cout << "取了" << data2<<endl;
			--c;
		}
		Sleep(1500);
	}
}
int main() {
	thread t1(producer);
	thread t2(consumer);
	t1.join();
	t2.join();
	return 0;
}
```

#### 2.7threadlocal

C++11中的thread_local是C++存储期的一种，属于**线程存储期**。存储期定义C++程序中变量/函数的范围(可见性)和生命周期。C++程序中可用的存储期包括auto、register、static、extern、mutable和thread_local。这些说明符**放置在它们所修饰的类型之前**。

线程局部存储(Thread Local Storage，TLS)是一种**存储期**(storage duration)，**对象的存储是在线程开始时分配**，线程结束时回收，每个线程有该对象自己的实例。这种对象的链接性(linkage)可以是静态的也可是外部的。TLS的一个例子是用全局变量errno表示错误号。这可能在多线程并发时产生同步错误。线程局部存储的errno是个解决办法。

对于Windows系统来说，全局变量或静态变量会被放到".data"或".bss"段中，但当使用declspec(thread)定义一个线程私有变量的时候，编译器会把这些变量放到PE文件的".tls"段中。当系统启动一个新的线程时，它会从进程的堆中分配一块足够大小的空间，然后把".tls"段中的内容复制到这块空间中，于是每个线程都有自己独立的一个".tls"副本。所以对于用declspec(thread)定义的同一个变量，它们在不同线程中的地址都是不一样的。对于一个TLS变量来说，它有可能是一个C++的全局对象，那么每个线程在启动时不仅仅是复制".tls"的内容那么简单，还需要把这些TLS对象初始化，必须逐个地调用它们的全局构造函数，而且当线程退出时，还要逐个地将它们析构，正如普通的全局对象在进程启动和退出时都要构造、析构一样。Windows PE文件的结构中有个叫数据目录的结构。它总共有16个元素，其中有一元素下标为IMAGE_DIRECT_ENTRY_TLS，这个元素中保存的地址和长度就是TLS表(IMAGE_TLS_DIRECTORY结构)的地址和长度。TLS表中保存了所有TLS变量的构造函数和析构函数的地址，Windows系统就是根据TLS表中的内容，在每次线程启动或退出时对TLS变量进行构造和析构。TLS表本身往往位于PE文件的".rdata"段中。

C++11引入了thread_local关键字用于下述情形：(1).命名空间(全局)变量；(2).文件静态变量；(3).函数静态变量；(4).静态成员变量。此外，不同编译器提供了各自的方法声明线程局部变量。thread_local作为类成员变量时必须是static的。

C++11中的thread_local关键字仅可允许使用在：命名空间范围内声明的对象；块范围内声明的对象；静态数据成员。它指示对象具有线程存储期(thread storage duration)。可以将其与static或extern组合以分别指定内部或外部链接(始终具有外部链接的静态数据成员除外)，但是附加的static不会影响存储期。具有不同范围的内部或外部链接的thread_local变量的名称可以引用相同或不同的实例，具体取决于代码是在同一线程中执行还是在不同线程中执行。

当你声明一个thread_local变量时，每个线程都有其自己的副本。当你通过名称引用它时，将使用与当前线程关联的副本。

如果类的成员函数内定义了thread_local变量，则对于同一个线程内的该类的多个对象都会共享一个变量实例，并且只会在第一次执行这个成员函数时初始化这个变量实例，这一点是跟类的静态成员变量类似的。


```c++
#include "funset.hpp"
#include <thread>
#include <mutex>
#include <iostream>
#include <string>
 
///
// reference: https://en.cppreference.com/w/cpp/language/storage_duration
namespace {
 
thread_local unsigned int rage = 1;
std::mutex cout_mutex;
 
void increase_rage(const std::string& thread_name)
{
	++rage; // modifying outside a lock is okay; this is a thread-local variable, 每个线程有一个副本
	std::lock_guard<std::mutex> lock(cout_mutex);
	fprintf(stdout, "Rage counter for: %s : %d\n", thread_name.c_str(), rage); // : 2, 子线程中rage的值为2
}
 
} // namespace
 
int test_thread_local_1()
{
	std::thread a(increase_rage, "a"), b(increase_rage, "b");
 
	{
		std::lock_guard<std::mutex> lock(cout_mutex);
		fprintf(stdout, "Rage counter for main: %d\n", rage); // 1, 主线程中rage的值始终为1
	}
 
	a.join();
	b.join();
	return 0;
}
```



## 9.decltype

### 9.1常规用法

decltype是C++11新增的关键字，主要用于提取变量和表达式的类型。
decltype的语法形式为：decltype(e)，这里e是一个表达式，而decltype(e)是一个类型指示符。decltype的结果不是值，而是一个类型。
decltype的语法规则主要有以下四条：
如果e是一个**没有用小括号括起来**的标识符表达式或类成员存取表达式(**表达式**)，那么decltype(e)的结果类型为**该表达式中标识符的声明类型**。
注：这里的小括号是指表达式e自身带的小括号，而不是decltype(e)中的小括号。
如果e是T类型的x值，那么decltype(e)的结果类型为**T&&**。
注：x值（xvalue）是C++11新引入的值的种类，介于传统的左值和右值之间。最常见的x值为**无名右值引用**。&&
如果e是T类型的左值，那么decltype(e)的结果类型为**T&**。
注：同时满足规则1和规则3的情况下，规则1优先。
如果e是T类型的纯右值，那么decltype(e)的结果类型为**T**。
注：纯右值（prvalue）即传统右值。**字面量以及临时对象都是纯右值。**

测试代码及出错信息

```c++
#include <iostream>
 
template<typename T>
T f();
 
struct S {int a;};
 
int main()
{
    int a = 0;
    S s;
    f<decltype(a)>();//int
    f<decltype(s.a)>();//int
    f<decltype(std::move(a))>();//&&
    f<decltype((a))>();//&
    f<decltype((s.a))>();//&
    f<decltype(0)>();//int
 
    decltype(a) b = a; // int b = a;
    decltype((a)) c = a; // int& c = a;
}

main.cpp:(.text.startup+0x5): undefined reference to `int f<int>()'
main.cpp:(.text.startup+0xa): undefined reference to `int f<int>()'
main.cpp:(.text.startup+0xf): undefined reference to `int&& f<int&&>()'
main.cpp:(.text.startup+0x14): undefined reference to `int& f<int&>()'
main.cpp:(.text.startup+0x19): undefined reference to `int& f<int&>()'
main.cpp:(.text.startup+0x1e): undefined reference to `int f<int>()'

```

decltype(a)和decltype(s.a)的结果类型为int。（适用规则1）
理由：a是未带外围小括号的标识符，s.a是未带外围小括号的类成员存取表达式。变量a的声明类型是int，S结构的成员变量a的声明类型也是int。
decltype(std::move(a))的结果类型为int&&。（适用规则2）
理由：std::move(a)是无名右值引用，是x值的一种。
decltype((a))和decltype((s.a))的结果类型为int&。（适用规则3）
理由：(a)和(s.a)都是带外围小括号的左值引用，是左值。
decltype(0)的结果类型为int。（适用规则4）
理由：0是纯右值（即传统右值）。

### 9.2decltype(auto)

decltype(auto)是C++14新增的类型指示符，可以用来声明变量以及指示函数返回类型。
当decltype(auto)被用于声明变量时，该变量必须立即初始化。假设该变量的初始化表达式为e，那么该变量的类型将被推导为decltype(e)。也就是说在推导变量类型时，先用初始化表达式替换decltype(auto)当中的auto，然后再根据decltype的语法规则来确定变量的类型。

```c++
decltype(auto) prop = value; ---> decltype(value)
```

decltype(auto)也可以被用于指示函数的返回值类型。假设函数返回表达式e，那么该函数的返回值类型将被推导为decltype(e)。也就是说在推导函数返回值类型时，先用返回值表达式替换decltype(auto)当中的auto，然后再根据decltype的语法规则来确定函数返回值的类型。

```c++
#include <iostream>
 
template<typename T>  
T f();
 
struct S {int a;};
 
int a = 0;
S s;
decltype(auto) g1() {return s.a;}
decltype(auto) g2() {return std::move(a);}
decltype(auto) g3() {return (a);}
decltype(auto) g4() {return (0);}
 
int main()
{
    decltype(auto) i1 = a;//int
    decltype(auto) i2 = std::move(a);//&&
    decltype(auto) i3 = (s.a);//int&
    decltype(auto) i4 = (0);//int
    f<decltype(i1)>();
    f<decltype(i2)>();
    f<decltype(i3)>();
    f<decltype(i4)>();
    f<decltype(g1())>();//int
    f<decltype(g2())>();//int&&
    f<decltype(g3())>();//int&
    f<decltype(g4())>();//int
}


main.cpp:(.text.startup+0x5): undefined reference to `int f<int>()'
main.cpp:(.text.startup+0xa): undefined reference to `int&& f<int&&>()'
main.cpp:(.text.startup+0xf): undefined reference to `int& f<int&>()'
main.cpp:(.text.startup+0x14): undefined reference to `int f<int>()'
main.cpp:(.text.startup+0x19): undefined reference to `int f<int>()'
main.cpp:(.text.startup+0x1e): undefined reference to `int&& f<int&&>()'
main.cpp:(.text.startup+0x23): undefined reference to `int& f<int&>()'
main.cpp:(.text.startup+0x28): undefined reference to `int f<int>()'
```

这段代码使用了同样的编程技巧来查看decltype(auto)的结果类型。
从出错信息中可以得知:

变量i1的类型等同于decltype(a)的结果类型，即int。（适用规则1）
函数g1的返回值类型等同于decltype(s.a)的结果类型，即int。（适用规则1）
变量i2的类型等同于decltype(std::move(a))的结果类型，即int&&。（适用规则2）
函数g2的返回值类型等同于decltype(std::move(a))的结果类型，即int&&。（适用规则2）
变量i3的类型等同于decltype((s.a))的结果类型，即int&。（适用规则3）
函数g3的返回值类型等同于decltype((a))的结果类型，即int&。（适用规则3）
变量i4的类型等同于decltype((0))的结果类型，即int。（适用规则4）
函数g4的返回值类型等同于decltype((0))的结果类型，即int。（适用规则4）

## 10.内存模型

### 1.1内存模型基础

这里从两方面来讲内存模型：一方面是基本结构，这与事物在内存中是怎样布局的有关；另一方面就是并发。对于并发基本结构很重要，特别是在低层[原子操作](https://so.csdn.net/so/search?q=原子操作&spm=1001.2101.3001.7020)。所以我将会从基本结构讲起。`C++`中它与所有的对象和内存位置有关。

### 1.2对象和内存位置

在一个C++程序中的所有数据都是由对象(objects)构成。这不是说你可以创建一个int的衍生类，或者是基本类型中存在有成员函数，或是像在Smalltalk和Ruby语言下讨论程序那样——“一切都是对象”。“对象”仅仅是对C++数据构建块的一个声明。**C++标准定义类对象为“存储区域”**，但对象还是可以将自己的特性赋予其他对象，比如，其类型和生命周期。

像int或float这样的对象就是简单基本类型；当然，也有用户定义类的实例。一些对象(比如，数组，衍生类的实例，特殊（具有非静态数据成员）类的实例)拥有子对象，但是其他对象就没有。

无论对象是怎么样的一个类型，一个对象都会存储在一个或多个内存位置上。每一个内存位置不是一个标量类型的对象，就是一个标量类型的子对象，比如，unsigned short、my_class*或序列中的相邻位域。当你使用位域，就需要注意**：虽然相邻位域中是不同的对象，但仍视其为相同的内存位置**。如图5.1所示，将一个struct分解为多个对象，并且展示了每个对象的内存位置。//一个对象内存储的对象不论怎么被存储的，都视为相同的内存位置
![image-20220720214836257](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220720214836257.png)

首先，完整的struct是一个有多个子对象(每一个成员变量)组成的对象。位域bf1和bf2共享同一个内存位置(int是4字节、32位类型)，并且std::string类型的对象s由内部多个内存位置组成，但是其他的每个成员都拥有自己的内存位置。注意，位域宽度为0的bf3是如何与bf4分离，并拥有各自的内存位置的。(译者注：**图中bf3是一个错误展示，在C++和C中规定，宽度为0的一个未命名位域强制下一位域对齐到其下一type边界**，其中type是该成员(bf3)的类型。这里使用命名变量为0的位域，可能只是想展示其与bf4是如何分离的。有关位域的更多可以参考wiki的页面)。
这里有四个需要牢记的原则：

   1.每一个变量都是一个对象，包括作为其成员变量的对象。

   2.每个对象至少占有一个内存位置。

   3.**基本类型都有确定的内存位置**(无论类型大小如何，即使他们是相邻的，或是数组的一部分)。

   4.相邻位域是相同内存中的一部分。

### 1.3对象、内存位置和并发

这部分对于C++的多线程应用来说是至关重要的：所有东西都在内存中。当两个线程访问不同的内存位置时，不会存在任何问题，一切都工作顺利。而另一种情况下，当两个线程访问同一个内存位置，你就要小心了。如果没有线程**更新内存位置上的数据**，那还好；只读数据不需要保护或同步。当有线程对内存位置上的数据进行修改，那就有可能会产生**条件竞争**，就如第3章所述的那样。

为了避免条件竞争，两个线程就需要一定的执行顺序。第一种方式，如第3章所述那样，使用互斥量来确定访问的顺序；当同一互斥量在两个线程同时访问前被锁住，那么在同一时间内就只有一个线程能够访问到对应的内存位置，所以后一个访问必须在前一个访问之后。另一种方式是**使用原子操作同步机制**(详见5.2节中对于原子操作的定义)，决定两个线程的访问顺序。使用原子操作来规定顺序在5.3节中会有介绍。当多于两个线程访问同一个内存地址时，**对每个访问这都需要定义一个顺序**。

如果不去规定两个不同线程对同一内存地址访问的顺序，那么访问就不是原子的；并且，当两个线程都是“作者”时，就会产生数据竞争和未定义行为。

以下的声明由为重要：未定义的行为是C++中最黑暗的角落。根据语言的标准，一旦应用中有任何未定义的行为，就很难预料会发生什么事情；因为，未定义行为是难以预料的。我就知道一个未定义行为的特定实例，让某人的显示器起火的案例。虽然，这种事情应该不会发生在你身上，但是数据竞争绝对是一个严重的错误，并且需要不惜一切代价避免它。

另一个重点是：当程序中的对同一内存地址中的数据访问存在竞争，你可以使用原子操作来避免未定义行为。当然，这不会影响竞争的产生——原子操作并没有指定访问顺序——但原子操作把程序拉回了定义行为的区域内。

在我们了解原子操作前，还有一个有关对象和内存地址的概念需要重点了解：修改顺序。

### 1.4修改顺序

每一个在C++程序中的对象，**都有(由程序中的所有线程对象)确定好的修改顺序**//自带的，在对象的初始化开始阶段确定。在大多数情况下，这个顺序不同于执行中的顺序，但是在给定的执行程序中，所有线程都需要遵守这顺序。**如果对象不是一个原子类型(将在5.2节详述)，你必要确保有足够的同步操作**，来确定每个线程都遵守了变量的修改顺序。当不同线程在不同序列中访问同一个值时，你可能就会遇到数据竞争或未定义行为(详见5.1.2节)。如果你使用原子操作，编译器就有责任去替你做必要的同步。

这一要求意味着：投机执行是不允许的，因为当线程按修改顺序访问一个特殊的输入，之后的读操作，必须由线程返回较新的值，并且之后的写操作必须发生在修改顺序之后。同样的，在同一线程上允许读取对象的操作，要不返回一个已写入的值，要不在对象的修改顺序后(也就是在读取后)再写入另一个值。虽然，所有线程都需要遵守程序中每个独立对象的修改顺序，但它们没有必要遵守在独立对象上的相对操作顺序。在5.3.3节中会有更多关于不同线程间操作顺序的内容。

## 11.类型转换

### 11.1static_cast

与dynamic_cast对应的是static_cast（静态强制）。static_cast关键字一般用来将[枚举类型](https://so.csdn.net/so/search?q=枚举类型&spm=1001.2101.3001.7020)转换成整型，或者短整形转换成长整形，又或者整型转换成浮点型。也可以用来将指向父类的指针转换成指向子类的指针。

#### 11.1.1static_cast使用注意事项

1）static_cast可以用于基本类型的转换，如short与int、int与float、enum与int之间；

2）static_cast也可以**用于类类型的转换**，但**目标类型必须含有相应的构造函数**；

3）static_cast还可以转换对象的指针类型，但它不进行运行时类型检查，所以是不安全的；

4）static_cast甚至可以把任何表达式都转换成void类型；

5）satic_cast**不能移除变量的const属性**，请参考const_cast操作符；

6）static_cast进行的是简单粗暴的转换（仅仅依靠尖括号中的类型），所以其正确性完全由程序员自己保证。

7）static_cast是在编译时进行的，这与dynamic_cast正好相反。

#### 11.1.2**static_cast的使用形式：**

static_cast< T >(exp)
dynamic_cast< T >(exp)
其中，T为目标数据类型，exp为原始数据类型变量或者表达式。

```c++
#include <iostream>
//#include <typeinfo>
using namespace std;
 
void test() {
    char ch = 'a';
    short sh = 10;
    int i1 = static_cast<char>(ch);//成功，将char型数据转换成int型数据
    int i2 = static_cast<short>(sh);
    
    cout<<i1<<endl;  //97
    cout<<i2<<endl;  //10
 
    double *d = new double;
    void *v = static_cast<void*>(d);//成功，将double指针转换成void指针
 
    cout<<v<<endl;  //0x7f7f9ec05960
 
 
    int i = 20;
    const int iConst = static_cast<const int>(i);//成功，将int型数据转换成const int型数据
 
    cout<<iConst<<endl;  //20
 
    const int jConst = 30;
    //int *p = static_cast<int*>(&jConst);//error: static_cast from 'const int *' to 'int *' is not allowed
}
 
int main()
{
    test();
 
    return 0;
}
/*
编译环境：mac os下用g++编译.
*/

```

```c++
const int jConst = 30;
    //int *p = static_cast<int*>(&jConst);
```

发生编译错误：error: static_cast from 'const int *' to 'int *' is not allowed。

根据提示也很明显，原因是static_cast不能将“const int *”转换成“int *”，即违反了static_cast使用注意事项的第5条：static_cast不能移除变量的const属性。
<font size=5>**用户定义类型使用static_cast转换:(继承)**</font>

```c++
#include <iostream>
#include <typeinfo>
using namespace std;
 
class Base {
public:
    int a;
    void fun1() {cout<<"Base::fun1"<<endl;}
    void fun2() {cout<<"Base::fun2"<<endl;}
};
 
class Derive : public Base{
public:
    int b;
    void fun2() {cout<<"Derive::fun2"<<endl;}
    void fun4() {cout<<"Derive::fun4"<<endl;}
};
 
void test() {
    Base b;
    Derive d;
 
    Base *pB = static_cast<Base*>(&d);    //派生类指针->父类指针
 
    Derive *pD = static_cast<Derive*>(&b); //父类指针->派生类指针
 
    pB->fun1(); // 调用父类的fun1
    pB->fun2(); // 调用父类的fun2
    //pB->fun4(); // 编译错误：error: no member named 'fun4' in 'Base'。 因为fun4是派生类的成员函数，只能通过派生类对象进行访问。
 
    pD->fun1(); //调用父类的fun1
    pD->fun2(); //调用派生类类的fun2
    pD->fun4(); //调用派生类类的fun4，fun4是派生类的成员函数，而不是父类的成员函数。
}
    
 
int main()
{
    test();
 
    return 0;
}
/*
编译环境：mac os下用g++编译：
*/
```

可见，使用static_cast能够进行派生类和父类的相互转换。

**再看一个例子：互不相关的类之间使用static_cast进行转换**

```c++
#include <iostream>
#include <typeinfo>
using namespace std;
 
class A {
public:
    int a;
    void fun1() {cout<<"A::fun1"<<endl;}
};
 
class B{
public:
    int b;
 
    B(A& a) {cout<<"B::constructor"<<endl;}
 
    void fun2() {cout<<"B::fun2"<<endl;}
};
 
void test() {
    A a;
 
    B b = static_cast<B>(a);    //A->B
 
    //b.fun1(); //编译错误，error: no member named 'fun1' in 'B'; did you mean 'fun2'?。 因为对于b，没有fun1成员函数。
    //实测并不会报错，一切正常，并且B相关的所有数据成员都设置为空
    b.fun2(); //B::fun2
}
    
 
int main()
{
    test();
 
    return 0;
}
/*
编译环境：mac os下用g++编译：
*/
```

但是，如果在B中，没有定义 B(A& a)的话，就会发生编译错误：

可见，如果A和B没有继承关系的两个互不相关的类，想要由A转换为B，则在B中必须定义“以A为参数的”构造函数。

[(32条消息) C++基础#22：C++中的静态强制static_cast_liranke的博客-CSDN博客_c++ static_cast](https://blog.csdn.net/liranke/article/details/5295133)

























## 12.malloc和new（*）

(1）malloc和new都是在堆上开辟内存的**malloc**只负责开辟内存，**没有初始化功能**，需要用户自己初始化；new不但开辟内存，还可以进行**初始化**，如new int(10)；表示在堆上开辟了一个4字节的int整形内存，初始值是10，再如new int[10] ()；表示在堆上开辟了一个包含10个整形元素的数组，初始值都为0。

(2）malloc是函数，**开辟内存需要传入字节数**，如malloc(100)；表示在堆上开辟了100个字节的内存，返回void*，表示分配的堆内存的起始地址，因此malloc的返回值需要强转成指定类型的地址；new是**运算符**，开辟内存需要指定类型，返回指定类型的地址，因此不需要进行强转。

```c++
#include<iostream>
using namespace std;
int main()
{
	unsigned char* p = new unsigned char('a');  //new不但开辟内存，还可以进行初始化，表示在堆上开辟了一个1字节的unsigned char整形内存，初始值是'a';
	unsigned char* p_arr = new unsigned char[10](); // 表示在堆上开辟了一个包含10个unsigned char元素的数组，初始值都为0。（加上小括号）
	cout << *p << endl;
	cout << int(*p) << endl;
	for (int i = 0; i < 10; i++) {
		cout << *p_arr++ << endl;
	}

	int *p1 = (int*)malloc(sizeof(int)); // = > 根据传入字节数开辟内存，没有初始化
	int *p2 = (int*)malloc(sizeof(int) * 100); //= > 开辟400个字节的内存，相当于包含100个整形元素的数组，没有初始化
	for (int i = 0; i < 100; i++) {
		cout << *p++ << endl;  //1 取数组当前位置的值*p; 2 p指向下一位置的数据
		//输出来里面是乱码，因为不知道之前内存里的是什么东西
	}
	
	delete p;
	delete []p_arr;
	free(p1);
	free(p2);
	
	return 0;
}

```

(3）malloc开辟内存失败返回NULL，new开辟内存失败抛出bad_alloc类型的异常，需要捕获异常才能判断内存开辟成功或失败，n**ew运算符其实是operator new函数的调用，它底层调用的也是malloc来开辟内存的，new它比malloc多的就是初始化功能，对于类类型来说，所谓初始化，就是调用相应的构造函数。**

(4）malloc开辟的内存永远是通过free来释放的；而new单个元素内存，用的是delete，如果new[]数组，用的是**delete[]**来释放内存的。
(5）malloc开辟内存只有一种方式，而new有四种**分别是普通的new（内存开辟失败抛出bad_alloc异常）, nothrow版本的new，const new以及定位new。**

1. **new是操作符，而malloc是函数**。
2. new在调用的时候先分配内存，在调用构造函数，释放的时候调用析构函数；而**malloc没有构造函数和析构函数**。
3. malloc需要给定**申请内存的大小**，返回的指针需要**强转**；new会调用构造函数，不用指定内存的大小，返回指针不用强转。
4. new可以被**重载**；malloc不行
5. new分配内存更**直接**和**安全**。
6. new发生错误**抛出异常**，malloc返回null

**malloc底层实现：**当开辟的空间小于 128K 时，调用 brk（）函数；当开辟的空间大于 128K 时，调用mmap（）。malloc采用的是内存池的管理方式，以减少内存碎片。先申请大块内存作为堆区，然后将堆区分为多个内存块。当用户申请内存时，直接从堆区分配一块合适的空闲快。采用隐式链表将所有空闲块，每一个空闲块记录了一个未分配的、连续的内存地址。

**new底层实现：**关键字new在调用构造函数的时候实际上进行了如下的几个步骤：

1. 创建一个新的对象
2. 将构造函数的作用域赋值给这个新的对象（因此this指向了这个新的对象）
3. 执行构造函数中的代码（为这个新对象添加属性）
4. 返回新对象





## 13.存储方式

![image-20220803172239748](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220803172239748.png)

![image-20220803172300836](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220803172300836.png)

## 14.extern

### 14.1翻译单元

#### 14.1.1简述

在C语言术语中， 翻译单元指C编译器产生目标文件（object file）的最终输入。在非正式使用情况下，**翻译单元也叫编译单元**。一个编译单元大致由一个<font color=red>**经过C预处理器处理过的源文件组成**</font>，意味着*由#include指令列出的头文件会被正确的包含进来，由#ifdef指令包含的代码会被包含进来，定义的宏会被展开*。

#### 14.1.2上下文

由单元组成的C程序叫源文件（或者叫预处理文件），源文件除了源代码以外，还包括C预处理指令(#include)。一个源文件经过预处理器处理后的输出叫做翻译单元。

预处理主要包括将一个源文件中由#include指令声明的文件（通常是头文件，也可能是其它源文件）递归地替换，产生的结果是一个预处理翻译单元。接下来包括**对#define指令进行宏展开**，对#ifdef指令进行条件编译等等; 这一步便将预处理翻译单元转换成一个**翻译单元**。编译器从翻译单元产生一个目标文件，目标文件经过后续处理后链接（可能需要其它目标文件）成一个可执行程序。

需要注意的是预处理器是语言无关的，只是一个词法处理器，只在词法分析级别，它并不做语法分析，所以它**不能处理具体的C语法**。编译单元做为编译器的输入，它将不会看到任何预处理指令，因为在编译之前预处理指令已经被预处理器处理了。**一个翻译单元根本上是基于一个文件，实际输入编译器的源代码可能和程序员所看到的大不一样，特别是递归包含的头文件**。



#### 14.1.3范围

翻译单元定义了一个范围，大致是文件范围，功能上类似于模块范围；在C术语中称为内部连接，内部连接是C语言中两种连接方式之一。在函数块外声明的名字（函数和变量）仅对该翻译单元可见，称为[内部连接](http://en.wikipedia.org/wiki/Linkage_(software))，内部连接对链接器不可见。如果名字对其它翻译单元可见，称为[外部连接](http://en.wikipedia.org/wiki/Linkage_(software))，外部连接对链接器可见。

C语言没有模块的概念。但是单独的目标文件（翻译单元产生的目标文件）功能也像一个独立的模块，如果一个源文件没有包含其它源文件，内部连接（翻译单元范围）可能被认为是包括所有头文件的文件范围。

#### 14.1.4代码组织

大部分工程的代码都是保存在以.c为后缀（c++用.cpp, .c++, 或 .cc，通常用.cpp）的文件中。被包含的文件一般以.h为后缀（c++用.hpp或.hh， 在c++中通常用.h比较多），为了避免多个源文件包含头文件产生的名字冲突，头文件中一般不包含函数或变量的定义。头文件可以被其它头文件包含。在项目中，.c文件至少包含一个头文件是标准做法。

### 14.2extern的含义

![image-20220806134017846](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220806134017846.png)



#### 14.2.1修饰全局变量

```c++
//file1
int i = 0; //默认拥有外部链接
extern int j; //j定义于其他翻译单元
extern int i = 10; //与 int i = 0 相同，这里extern被忽略，属于重复定义
```

### 14.3链接指示: extern "C"

C++向下兼容C，所以C++需要提供一种方式能和C代码互动。说起来应该是很简单的一件事，因为他们的语法都是一样的嘛

<font size=5>C++代码</font>

链接指示可以有两种形式:  单个的或复合的。链接指示(extern ' ')不能出现在**类定义或函数定义**的内部。同样的链接指示必须在函数的每个声明中都出现

cstring头文件的部分函数声明

```c++
单语句
    extern "C" size_t strlen(const char *);
复合语句
    extern "C" {
    int strcmp(const char*, const char *);
    char *strcat(char*, const char* );
}
```

#### 14.3.1链接指示和头文件

也可直接应用于预编译的头文件

```C++
extern "C" {
    #include<string.h>
}
```

头文件中的所有普通函数**声明**都认为是由链接指示的语言编写的，并且链接指示**可以嵌套**，链接指示的头文件内的链接指示的函数不受影响

#### 14.3.2指向 extern "C"函数的指针

编写函数的语言类型是函数类型的一部分

指向其他语言编写的函数的指针必须与函数本身使用相同的链接指示

```c++
extern "C" void (*pf)(int); //使用pf调用函数的时候，编译器默认该函数也是一个C函数

```

因为C不支持重载，所以C函数只能有一个，其他的重载版本都是C++函数



## 15.volatile

[volatile](https://so.csdn.net/so/search?q=volatile&spm=1001.2101.3001.7020)的本意是“易变的”,volatile关键字是一种类型修饰符，**用它声明的类型变量表示可以被某些编译器未知的因素更改**，比如操作系统、硬件或者其它线程等。遇到这个关键字声明的变量，**编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问**。

```C++
volatile int i=10;
 
int a = i;
 
。。。//其他代码，并未明确告诉编译器，对i进行过操作
 
int b = i;
```

volatile 指出 i是随时可能发生变化的，**每次使用它的时候必须从i的地址中读取**，因而[编译器](https://so.csdn.net/so/search?q=编译器&spm=1001.2101.3001.7020)生成的汇编代码会重新从i的地址读取数据放在b中。而优化做法是，*由于编译器发现两次从i读数据的代码之间的代码没有对i进行过操作*，它会自动把**上次读的数据**放在b中。而不是**重新从i里面读**。这样以来，如果i是一个寄存器变量或者表示一个端口数据就容易出错，所以说volatile可以保证对特殊地址的稳定访问。

volatile和const并不冲突，可同时用于修饰同一个变量，用法也相同，只有volatile修饰的对象可以调用被volatile修饰的成员函数，且有volatile xx,  xx volatile, volatile xx volatile



## 16.波兰式

### 16.1波兰式（前缀式）

波兰式是在通常的表达式中，二元运算符总是置于与之相关的两个运算对象之前，所以，这种表示法也称为前缀表达式。例如：3*（2-（5+1）），用波兰式来表示是： *3 - 2 + 5 1。

阅读这个表达式需要**从左至右读入表达式**，**如果一个操作符后面跟着两个操作数时，则计算**，然后将结果作为操作数替换这个操作符和两个操作数，重复此步骤，直至所有操作符处理完毕。从左往右依次读取，直到遇到+ 5 1，做计算后，将表达式替换为* 3 - 2 6，然后再次从左往右读取，直到遇到- 2 6，做计算后，将表达式替换为*3 （-4）（这里“-”为负号不是减号，-4为一个数负四），从而得到最终结果-12。

### 16.2逆波兰式（后缀式）

逆波兰式（Reverse Polish notation，RPN，或逆波兰记法），也叫后缀表达式（将运算符写在操作数之后）。例如：3*（2-（5+1）），用逆波兰式来表示是：3 2 5 1 + - *，也就是把操作运算符往操作数后面放。

阅读这个表达式需要**从左往右读入表达式**，当读到第一个操作符时，从左边取出两个操作数做计算，然后将这个结果作为操作数替换这个操作符和两个操作数，重复此步骤，直至所有操作符处理完毕。
3 2 5 1 + - *
→3 2 6 - *
→3 （-4） *
→ -12

## 17.条件编译（头文件用）

c++中条件编译相关的预编译指令，包括 #define、#undef、#ifdef、#ifndef、#if、#elif、#else、#endif、defined。

#define 定义一个预处理宏
#undef 取消宏的定义

#if 编译预处理中的条件命令，相当于C语法中的if语句
#ifdef 判断某个宏是否被定义，若已定义，执行随后的语句
#ifndef 与#ifdef相反，判断某个宏是否未被定义
#elif 若#if, #ifdef, #ifndef或前面的#elif条件不满足，则执行#elif之后的语句，相当于C语法中的else-if
#else 与#if, #ifdef, #ifndef对应, 若这些条件不满足，则执行#else之后的语句，相当于C语法中的else
#endif #if, #ifdef, #ifndef这些条件命令的结束标志.
defined 　与#if, #elif配合使用，判断某个宏是否被定义



C++中#ifdef、#else、#endif 都是预处理命令，称为条件编译命令。其中，#ifdef 后接一个标识符和程序段1，#else 后接程序段2和结束条件编译段的预处理命令 #endif。其中所有的预处理命令都必须换行写，且单独占一行。其意义为：如果标识符已经用宏定义命令进行过定义，编译程序段1，跳过程序段2；否则，跳过程序段1，编译程序段2。

## 18.::

全局作用域符号：当[全局变量](https://so.csdn.net/so/search?q=全局变量&spm=1001.2101.3001.7020)在局部函数中与其中某个变量重名，那么就可以用::来区分

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220806154055334.png" alt="image-20220806154055334" style="zoom: 50%;" />



## 19.类的大小

### 1 类的大小与什么有关系？

**与类大小有关的因素：普通成员变量，虚函数，继承（单一继承，多重继承，重复继承，虚拟继承）**

**与类大小无关的因素：静态成员变量，静态成员函数及普通成员函数**



### 2 空类

空类即什么都没有的类，按上面的说法，照理说大小应该是0，但是，**空类的大小为1**，因为空类可以实例化，**实例化必然在内存中占有一个位置**，因此，编译器为其优化为一个字节大小。

```c++
#include <iostream>
using namespace std;
 
class base
{
};
class derived:public base
{
 private:
    int a;
};
 
 
 
int main(int argc, char** argv) {
        cout << sizeof(base) << endl;
        cout << sizeof(derived) << endl;
        return 0;
}
```

此时，derived类的大小为4，derived类的大小是自身int成员变量的大小，至于为什么没有加上父类base的大小1是因为空白基优化的问题，**在空基类被继承后，子类会优化掉基类的1字节的大小**，节省了空间大小，提高了运行效率。

### 3 一般类的大小（注意内存对齐）

示例1

```C++
#include <iostream>
using namespace std;
 
 
class base1
{
private:
    char a;
    int b;
    double c;
};
class base2
{
private:
    char a;
    double b;
    int c;
};
 
 
int main(int argc, char** argv) {
        cout << sizeof(base1) << endl;//16
        cout << sizeof(base2) << endl;//24
        return 0;
}
```

### 4 含虚函数的单一继承

```c++
#include <iostream>
using namespace std;
 
class Base
{
private:
    char a;
public:
    virtual void f();
    virtual void g();
};
class Derived:public Base
{
private:
    int b;
public:
    void f();
};
class Derived1:public Base
{
private:
    double b;
public:
    void g();
    virtual void h();
};
 
 
int main(int argc, char** argv) {
        cout << sizeof(Base) << endl;//16
        cout << sizeof(Derived) << endl;//16
        cout << sizeof(Derived1) << endl;//24
        return 0;
}
```

## 20.Git





基础linux操作

![image-20220815223209055](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220815223209055.png)

![image-20220816122547880](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220816122547880.png)

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220816123119542.png" alt="image-20220816123119542" style="zoom: 50%;" />

git add .     git commit       git push

![image-20220816123525638](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220816123525638.png)

```
git clone ##git文件网址
git init    git init的作用主要是用于初始化一个空的仓库.后续在这个仓库下面的一切的操作,都会纳入到Git的版本控制当中

git add .   把当前目录下的所有文件放入暂存区

git status  查看远程仓库 暂存区 本地仓库 的状态

git commit -m   把暂存区文件提交到本地仓库，-m是提交的消息（注释差不多）



git pull --allow-unrelated-histories

git push 



```



![image-20220817184739080](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220817184739080.png)

创建一个公钥

```
ssh-keygen -t rsa  //在用户名/.ssh目录下
```

每台电脑可以生成一个自己唯一的一个公钥





![image-20220817174041095](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220817174041095.png)

## 21.P4（废弃）

P4虽然是epic官方推荐，但是教程少且质量差，纯英文界面，操作复杂难上手，不如用Git (虽然Git功能差一点)

![image-20220818185543796](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220818185543796.png)

新建一个工作区（本地仓库）

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220818185657058.png" alt="image-20220818185657058" style="zoom:50%;" />

用户名就是P4里面的用户名，修改一个文件，编辑器会询问是否迁出，迁出即代表你占用着这个文件，只有你把它提交给源码管理，才会释放控制权

右键文件，Get Revision 可以回到给定版本号的版本

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220818190428155.png" alt="image-20220818190428155" style="zoom: 67%;" />

## 22.短答

<font size =5>代码到可执行文件</font>

答:  一个C++程序从源码到执行文件，有四个过程，**预编译、编译、汇编、链接**。





<font size =5>说说数组和指针的区别</font>

数组:保存多个相同类型数据的集合

指针：一个存储其他变量在内存中的地址

赋值：同类型指针可以互相赋值，数组不行

存储方式：数组在内存中连续存放，指针没有确切存储位置

sizeof：数组: sizeof（数组名）/sizeof (数据类型)

指针：x32 = 4，x64 = 8





<font size =5>说说什么是函数指针，如何定义函数指针，有什么使用场景</font>

1. **概念：**函数指针就是**指向函数**的指针变量。每一个函数都有一个入口地址，该入口地址就是函数指针所指向的地址。

1. 函数指针的**应用场景**：**回调**（callback）。我们调用别人提供的 API函数(Application Programming Interface,应用程序编程接口)，称为Call；如果别人的库里面调用我们的函数，就叫Callback。



<font size =5>说说静态变量什么时候初始化？</font>

C++标准规定：全局或静态对象当且仅当对象首次用到时才进行构造。

**作用域**：C++里作用域可分为6种：全局，局部，类，语句，命名空间和文件作用域。





<font size =5> nullptr调用成员函数可以吗？为什么？</font>

能。

原因：因为在**编译时对象**就绑定了**函数地址**，和指针空不空没关系。





<font size =5>说说什么是野指针，怎么产生的，如何避免？</font>

1. **概念：**野指针就是指针指向的位置是不可知的（随机的、不正确的、没有明确限制的）

   2.**产生原因**：释放内存后指针不及时置空（野指针），依然指向了该内存，那么可能出现非法访问的错误。这些我们都要注意避免。

1. **避免办法：**

   （1）初始化成NULL

   （2）申请内存后判空

   （3）指针指向变量释放后指针设置成NULL

   （4）使用智能指针





<font size =5>说说静态局部变量，全局变量，局部变量的特点，以及使用场景</font>

1. **首先从作用域考虑**：C++里作用域可分为6种：全局，局部，类，语句，命名空间和文件作用域。

   全局变量：全局作用域，可以通过extern作用于其他非定义的源文件。

   静态全局变量 ：全局作用域+文件作用域，所以无法在其他文件中使用。

   局部变量：局部作用域，比如函数的参数，函数内的局部变量等等。

   静态局部变量 ：局部作用域，只被初始化一次，直到程序结束。

2. **从所在空间考虑**：除了局部变量在栈上外，其他都在静态存储区。因为静态变量都在静态存储区，所以下次调用函数的时候还是能取到原来的值。

3. **生命周期**： 局部变量在栈上，出了作用域就回收内存；而全局变量、静态全局变量、静态局部变量都在静态存储区，直到程序结束才会回收内存。

4. **使用场景**：从它们各自特点就可以看出各自的应用场景，不再赘述。





<font size =5>说说内联函数和宏函数的区别</font>

1. **宏定义不是函数**，但是使用起来像函数。预处理器用复制宏代码的方式代替函数的调用，省去了函数压栈退栈过程，提高了效率；**而内联函数本质上是一个函数**，内联函数一般用于函数体的代码比较简单的函数，不能包含复杂的控制语句，while、switch，并且内联函数本身不能直接调用自身。
2. **宏函数**是在预编译的时候把所有的宏名用宏体来替换，简单的说就是字符串替换 ；**而内联函数**则是在编译的时候进行代码插入，编译器会在每处调用内联函数的地方**直接把内联函数的内容展开**，这样可以省去函数的调用的开销，提高效率
3. **宏定义**是没有类型检查的，无论对还是错都是直接替换；**而内联函数**在编译的时候会进行类型的检查，内联函数满足函数的性质，比如有返回值、参数列表等

最好将inline函数定义在头文件





<font size =5>说说运算符i++和++i的区别</font>

1. **赋值顺序不同**：++ i 是先加后赋值；i ++ 是先赋值后加；++i和i++都是分两步完成的。
2. **效率不同**：后置++执行速度比前置的慢。
3. **i++ 不能作为左值，而++i 可以**

   4.两者都不是原子操作。





## 23.map和unordered_map的优缺点，适用于什么场景？(*)

<font size =7>map</font>
map的底层是基于**红黑树**实现的
**优点**：<font color='red' >有序性</font>，这是map最大的优点，其元素的有序性在很多应用中都会简化很多操作
map的查找，删除，增加等一系列操作的**时间复杂度稳定**，都为O(logn)

**缺点**：
查找 删除 增加等操作的**平均时间复杂度较慢**，与n相关

<font size = 7>unordered_map</font>
unordered_map的底层基于哈希表实现的

优点: 查找，删除 , 添加的速度快，时间复杂度为常数级 O（c)

缺点: 由于unordered_map的底层是基于哈希表实现的 以（key,value)的储存形式，因此空间占用率高

unordered_map的查找，删除，添加的时间复杂度不稳定 平均为常数级O（c) 取决于哈希函数，极端情况下为  O（n)



## 24.emplace_back和push_back区别

emplace_back()是c++11的新特性。

emplace_back(arg), arg-----构造函数的参数

区别在于：

push_back()方法**要调用构造函数和复制构造函数**，这也就代表着要先构造一个临时对象，然后把临时的copy构造函数拷贝或者移动到容器最后面。

而emplace_back()在实现时，则是**直接在容器的尾部创建这个元素，省去了拷贝或移动元素的过程。**

emplace_back() 函数在原理上⽐ push_back() 有了⼀定的改进，包括在内存优化⽅⾯和运⾏效率⽅⾯。内存优化主要体现在使⽤了就地构造（直接在容器内构造对象，不⽤拷⻉⼀个复制品再使⽤） + 强制类型转换 的⽅法来 实现，在运⾏效率⽅⾯，由于省去了拷⻉构造过程
结论：在C++11情况下，**果断用emplace_back代替push_back** 



## 25.数组和链表的区别

<font size=5>一、数组</font>

数组是最基本的数据结构，所开辟的**内存空间是连续的**，且内存大小一经确定之后便无法再更改；
<font color = 'blue'>优点</font>：**查找速度快**，因为开辟的内存空间是连续的，
为什么说是查找速度快？

1、因为**可以直接通过数组的索引得到对应的数据**，

2、因为存储数据的内存连续，就算不知道所需要的数据对应的索引，即便从头到尾顺序查找一遍也能快速得到想要的数据。

<font color='blue'>缺点</font>：
1、**浪费内存，缺乏弹性**（不能根据当前实际需求更改大小）。

2、**增添和删除的效率低**。因为数组的**大小在一开始就确定**，无法更改，在后续想要添加或者删除数据，不能直接往里面添加或者删除索引，取而代之的方法是：先复制原有的数组，根据添加或者删除的数据再增加或减小数组长度，再往新的数组中添加或删除数据。

<font size=5>二、链表</font>

链表，存储数据的内存**不需要连续**的，链表中的数据可以存储在内存的任何地方，这是因为**链表中的每个数据都存储了下一个链表的地址**，从而使**离散**的内存空间**联系在一起**，能**合理的利用内存**。每个链表包含多个节点，每个节点又包含数据域和引用域。

<font color='blue'>缺点</font>：
**查找元素麻烦**，如果要查找链表中的一个元素，需要从第一个元素开始，依次往下，直到找到需要的元素位置。

<font color = 'blue'>优点</font>：
添加和删除元素十分方便。只需要知道修改前一个元素指向的地址即可。

<font color = 'red'>插入</font>: 把插入位置的原元素的上一个元素的next指向插入元素，再把插入元素的next指向原元素

<font color = 'red'>删除</font>:  把删除元素的前一个元素的next指向删除元素的next



链表实现原理：

1.创建节点类，并在类中写两个部分，数据域和引用域。

2.创建链表类，需要包含头结点，尾节点和大小，并在其中加入所需要的方法



## 26.智能指针

<font size =5>一. 定义</font>

在C++中没有[垃圾回收](https://so.csdn.net/so/search?q=垃圾回收&spm=1001.2101.3001.7020)机制，必须自己释放分配的内存，否则就会造成内存泄露。解决这个问题最有效的方法是使用智能指针（smart pointer）。

```text
C++11中提供了三种智能指针，使用这些智能指针时需要引用头文件<memory>

std::shared_ptr：共享的智能指针
std::unique_ptr：独占的智能指针
std::weak_ptr：弱引用的智能指针，它不共享指针，不能操作资源，是用来监视shared_ptr的。

```

<font size=5>shared_ptr</font>

共享智能指针是指**多个智能指针可以同时管理同一块有效的内存，并通过一个User_Count的成员函数记录同时有几个共享指针指向同一个内存地址** 

**可以调用共享智能指针类提供的get()方法得到原始地址**

**共享指针的User_Count为0时，执行删除操作，这个删除操作对应的函数被称之为删除器**，**这个删除器函数本质是一个回调函数(不需要程序指定运行，编辑器自动运行)**



<font size=5>unique_ptr</font>

**std::unique_ptr是一个独占型的智能指针，它不允许<font  color = ' red'>其他的智能指针</font>共享其内部的指针，可以通过它的构造函数初始化一个独占智能指针对象，但是不允许通过赋值将一个unique_ptr赋值给另一个unique_ptr。**



unique_ptr指定删除器和shared_ptr指定删除器是有区别的，unique_ptr指定删除器的时候**需要确定删除器的类型**，所以不能像shared_ptr那样直接指定删除器



<font size=5>weak_ptr</font>

弱引用智能指针std::weak_ptr可以看做是**shared_ptr的助手**，它不管理shared_ptr内部的指针。std::weak_ptr没有重载操作符*和->，因为它**不共享指针，不能操作资源**，所以它的构造不会增加引用计数，析构也不会减少引用计数，它的主要作用就是**作为一个旁观者监视shared_ptr中管理的资源是否存在。**


```c++
#include <iostream>
#include <memory>
using namespace std;
 
int main() 
{
    shared_ptr<int> sp(new int);
 
    weak_ptr<int> wp1;//null
    weak_ptr<int> wp2(wp1);//null
    weak_ptr<int> wp3(sp);//sp
    weak_ptr<int> wp4;//null
    wp4 = sp;//sp
    weak_ptr<int> wp5;
    wp5 = wp3;
    
    return 0;
}
```

```tex
1.weak_ptr<int> wp1;构造了一个空weak_ptr对象
2.weak_ptr<int> wp2(wp1);通过一个空weak_ptr对象构造了另一个空weak_ptr对象
3.weak_ptr<int> wp3(sp);通过一个shared_ptr对象构造了一个可用的weak_ptr实例对象
4.wp4 = sp;通过一个shared_ptr对象构造了一个可用的weak_ptr实例对象（这是一个隐式类型转换）
5.wp5 = wp3;通过一个weak_ptr对象构造了一个可用的weak_ptr实例对象
 通过调用std::weak_ptr类提供的use_count()方法可以获得当前所观测资源的引用计数
```

**常用函数**

```tex
1. 通过调用std::weak_ptr类提供的expired()方法来判断观测的资源是否已经被释放

2. 通过调用std::weak_ptr类提供的lock()方法来获取管理所监测资源的shared_ptr对象

3. 通过调用std::weak_ptr类提供的reset()方法来清空对象，使其不监测任何资源

利用weak_ptr可以解决shared_ptr的一些问题
 1.返回管理this的shared_ptr
 2.解决循环引用问题
 
   在某些情况下，需要断开 shared_ptr 实例间的循环引用

   想要观察某个对象但不需要其保持活动状态，请使用该实例
```

<font size=5>shared_ptr的循环引用问题</font>

什么是循环引用，两个对象相互使用shared_ptr指向对方。造成的后果是：内存泄漏

```c++
class A;
class B;
class A {
public:
    std::shared_ptr<B> bptr;
    ~A() {
        cout << "A is deleted" << endl; // 析构函数后，才去释放成员变量
    }
};

class B {
public:
    std::shared_ptr<A> aptr;
    ~B() {
        cout << "B is deleted" << endl;  // 析构函数后，才去释放成员变量
    }
};

int main()
{
    std::shared_ptr<A> pa;
    {
        std::shared_ptr<A> ap(new A);
        std::shared_ptr<B> bp(new B);
        ap->bptr = bp;
        bp->aptr = ap;
    }
    return 0;
}


```







# 2.数据结构

![img](https://img-blog.csdnimg.cn/20210704171234592.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doZXJlSXNIZXJvRnJvbQ==,size_32,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20210704171728670.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doZXJlSXNIZXJvRnJvbQ==,size_32,color_FFFFFF,t_70)

## 2.1基础概念

- 对于实现某个算法，我们往往会用到一些数据结构。
- 因为我们通常不能一下子把数据处理完，更多的时候需要先把它们放在一个容器或者说缓存里面，等到一定的时刻再把它们拿出来。
- 这其实是一种 **「空间换时间」** 思想的体现， 恰当使用数据结构可以帮助我们高效地处理数据。
- 常用的一些数据结构如下：

| 数据结构 | **应用场景**                             |
| -------- | ---------------------------------------- |
| 数组     | 线性存储、元素为任意相同类型、随机访问   |
| 字符串   | 线性存储、元素为字符、结尾字符、随机访问 |
| 链表     | 链式存储、快速删除                       |
| 栈       | 先进后出                                 |
| 队列     | 先进先出                                 |
| 哈希表   | 随机存储、快速增删改查                   |
| 二叉树   | 对数时间增删改查，二叉查找树、线段树     |
| 多叉树   | B/B+树 硬盘树、字典树 字符串前缀匹配     |
| 森林     | 并查集 快速合并数据                      |
| 树状数组 | 单点更新，成段求和                       |

任何一种数据结构都不是 **完美的**。所以我们需要根据对应的场景，来采用对应的数据结构，具体用哪种数据结构，需要通过刷题不断刷新经验，才能总结出来。

### 2.1.1时间复杂度

常数时间的操作（操作与数据量无关），如数组，可以直接获得目标下标的值，反例如链表，必须要遍历对应数量元素才能得到目标的值,<font color='red'>时间复杂度是算法最差情况下的数值</font>

**常数操作的次数中以n为方程的最高阶项去掉系数**

![image-20220825160336959](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220825160336959.png)

例:选择排序  ---- O(n^2)  O---一个数据的上限(在数据量达到一定程度，系数及低阶项的大小可忽略，只能用最高阶项来表示一个数据的上限)

两个操作时间复杂度相同，只能用实际操作（运行代码）来比较 

<font size =5>额外空间复杂度</font>：

   **在执行代码过程中申请的额外存储空间**，比如变量，有限个变量则O(1)，分析额外空间复杂度和时间复杂度类似;如使用冒泡排序对一个数组排序，期间只需要一个临时变量temp，那么该算法的额外空间复杂度为O(1)。又如归并排序，在排序过程中需要创建一个与样本数组相同大小的辅助数组，尽管在排序过后该数组被销毁，但该算法的额外空间复杂度为O(n)。

  执行一个操作，是否需要额外开辟一个空间，不需要为 O (1), 需要则根据实际, 例如需要额外开辟一个数组则是O (n)

 









## 2.2二叉树

### 2.1.1相关术语

①节点：包含一个数据元素及若干指向子树分支的信息 。

②节点的度：一个节点拥有子树的数目称为节点的度。

③叶子节点：也称为终端节点，没有子树的节点或者度为零的节点。

④分支节点：也称为非终端节点，度不为零的节点称为非终端节点。

⑤树的度：树中所有节点的度的最大值 。

⑥节点的层次：从根节点开始，假设根节点为第1层，根节点的子节点为第2层，依此类推，如果某一个节点位于第L层，则其子节点位于第L+1层。

⑦树的深度：也称为树的高度，树中所有节点的层次最大值称为树的深度  。

⑧有序树：如果树中各棵子树的次序是有先后次序，则称该树为有序树 。

⑨无序树：如果树中各棵子树的次序没有先后次序，则称该树为无序树 。

⑩森林：由m（m≥0）棵互不相交的树构成一片森林。如果把一棵非空的树的根节点删除，则该树就变成了一片森林，森林中的树由原来根节点的各棵子树构成  。



#### 2.1.1.1满二叉树

除底层节点度数为0，其他所有节点度数为2

深度（最大层数）为k，节点个数为（2^k）- 1

1、满二叉树：如果一棵二叉树只有度为0的节点和度为2的节点，并且度为0的节点在同一层上，则这棵二叉树为满二叉树。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220824091746566.png" alt="image-20220824091746566" style="zoom:50%;" />







#### 2.1.1.2完全二叉树

2、完全二叉树：深度为k，有n个节点的二叉树当且仅当其每一个节点都与深度为k的满二叉树中编号从1到n的节点一一对应时，称为完全二叉树 。



3、完全二叉树的特点是叶子节点只可能出现在层序最大的两层上，并且某个节点的左分支下子孙的最大层序与右分支下子孙的最大层序相等或大1 。

```text
完全二叉树：完全二叉树的节点数是任意的，从形式上讲它是个缺失的的三角形，但所缺失的部分一定是右下角某个连续的部分，最后那一行可能不是完整的，对于k层的完全二叉树，节点数的范围
2^(k - 1) - 1 < N < 2^k - 1;

设二叉树的深度为h，除第 h 层外，其它各层 (1～h-1) 的结点数都达到最大个数，第 h 层所有的结点都连续集中在最左边，这就是完全二叉树。
```



<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220824091811253.png" alt="image-20220824091811253" style="zoom:50%;" />





​     满二叉树（full binary tree）中，每个深度级别的每一个可能的位置都有一个节点。

​     满二叉树最下面一层，所有的节点都是叶节点（也就是说，所有的叶节点都处于相同的深度，并且每个非叶节点都具有两个子节点）。

​     完全二叉树是在最深层之外的每一个可能位置都有一个节点，

​     完全二叉树在最深的那一层，节点按照从左到右的位置进行排列。

#### 2.1.1.3平衡二叉树

​     平衡二叉树又称为AVL树，是一种特殊的二叉排序树。其左右子树都是平衡二叉树，且**左右子树高度之差的绝对值不超过1**。一句话表述为：**以树中所有结点为根的树的左右子树高度之差的绝对值不超过1**。将二叉树上结点的左子树深度减去右子树深度的值称为**平衡因子**BF，那么平衡二叉树上的所有结点的平衡因子只可能是-1、0和1。只要二叉树上有一个结点的平衡因子的绝对值大于1，则该二叉树就是不平衡的。







#### 2.1.1.4红黑树

红黑树是一种二叉查找树，但在每个节点增加一个存储位表示节点的颜色，可以是红或黑（非红即黑）。通过对任何一条从根到叶子的路径上各个节点着色的方式的限制，红黑树确保没有一条路径会比其它路径长出两倍，因此，红黑树是一种弱平衡二叉树，相对于要求严格的AVL树来说，它的旋转次数少，所以对于搜索，插入，删除操作较多的情况下，通常使用红黑树。

性质：

\1. 每个节点非红即黑

\2. 根节点是黑的;

\3. 每个叶节点（叶节点即树尾端NULL指针或NULL节点）都是黑的;

\4. 如果一个节点是红色的，则它的子节点必须是黑色的。

\5. 对于任意节点而言，其到叶子点树NULL指针的每条路径都包含相同数目的黑节点;



区别：

AVL 树是高度平衡的，频繁的插入和删除，会引起频繁的rebalance，导致效率下降；红黑树不是高度平衡的，算是一种折中，插入最多两次旋转，删除最多三次旋转。









### 2.1.2二叉树的遍历

#### 2.1.2.1前序遍历 -递归

文中的[二叉树](https://so.csdn.net/so/search?q=二叉树&spm=1001.2101.3001.7020)结构如下：

```c++
struct TreeNode {
	int val;
	TreeNode* left;
	TreeNode* right;
	TreeNode() : val(0), left(nullptr), right(nullptr) {}
	TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
	TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
	
};
```

二叉树的前序遍历顺序为先访问根节点，然后对左子树进行前序遍历，再对右子树前序遍历，这是一个很明显的递归思想。访问当前节点的值，函数调用自身去访问左节点，然后访问右节点，那么递归函数的主体内容已经确定了

```c++
void helper(TreeNode* root) 
{
		//访问当前节点的值
		int res = root->val;
		
		//前序遍历左子树
		helper(root->left);
		
		//前序遍历右子树
		helper(root->right);
}

```

函数的主体已经确定，后面就是对细节问题的一些优化了，可以看到我们并没有考虑节点为空的情况，这一点也很简单，节点为空的时候直接返回就行，然后我们要把读取到的节点值存储起来，需要一个数组来存放数值，这样一来前序遍历的递归函数就完成了，下面是完整代码



```c++
vector<int> preorderTraversal(TreeNode* root)
{
	//二叉树前序遍历
	vector<int> res; //定义数组
	helper(root, res);
	return res;
}

void helper(TreeNode* root, vector<int>& res)
{
	//如果节点为空，则直接返回
	if (root == nullptr)
		return;
	//将节点的值存到数组中
	res.push_back(root->val);
	
	//前序遍历左子树
	helper(root->left, res);
	
	//前序遍历右子树
	helper(root->right, res);
}


```











### 2.1.3二叉树的增删改查

<font size =5>增-二叉树的建立</font>

建立一个 左子树 < 本节点 < 右子树的二叉树

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220824094344347.png" alt="image-20220824094344347" style="zoom:50%;" />

建立二叉树就是插入二叉树的过程，**本次插入的位置都是叶子节点**，将比较小的放到[叶子节点](https://so.csdn.net/so/search?q=叶子节点&spm=1001.2101.3001.7020)的左边，比较大的放到叶子节点的右边。

例如我们要将 **6** 添加到上面的二叉树的中，

- 比较 6 > 4 因此，接右子树
- 比较 6 < 8 ，接左子树
- 比较 6 < 7 接 ，左子树
- 比较 6 > 5 ，且 5的右子树为空，将6 作为 5 的右子树（结束）

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220824094502744.png" alt="image-20220824094502744" style="zoom:67%;" />

## 2.3哈希表

一张表只有一个哈希函数

哈希表：也叫做**散列表**。是根据**关键字和值**（Key-Value）直接进行访问的数据结构。也就是说，它**通过关键字 key 和一个映射函数 Hash(key) 计算出对应的哈希表上的地址，再得到这个地址上的值 value**，然后把**键值对映射到表中一个位置来访问记录**，以加快查找的速度。这个**映射函数叫做哈希函数**（散列函数），用于存放记录的数组叫做 哈希表（散列表）。 哈希表的关键思想是使用哈希函数，将键 key 和值 value 映射到对应表的某个区块中。可以将算法思想分为两个部分：

向哈希表中**插入一个关键字**：**哈希函数**决定该关键字的对应值应该存放到表中的哪个区块，并将对应值存放到该区块中
在哈希表中**搜索一个关键字**：使用**相同的哈希函数**从哈希表中查找对应的区块，并在特定的区块搜索该关键字对应的值
![image-20220826142310888](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220826142310888.png)



### 2.3.1哈希函数

哈希函数：**将哈希表中元素的关键字映射为元素存储位置的函数**。一般来说，哈希函数会满足以下几个条件：

​      哈希函数应该易于计算，并且尽量使计算出来的索引值均匀分布，这能减少哈希冲突
​      哈希函数计算得到的哈希值是**一个固定长度的输出值**
​      如果 Hash(key1) 不等于 Hash(key2)，那么 key1、key2 一定不相等
​      如果 Hash(key1) 等于 Hash(key2)，**那么 key1、key2 可能相等，也可能不相等（会发生哈希冲突），没有不发生哈希冲突的哈希函数，只能根据需求选择特定的哈希函数**

在哈希表的实际应用中，关键字的类型除了数字类型，还有可能是字符串类型、浮点数类型、大整数类型，甚至还有可能是几种类型的组合。**一般会将各种类型的关键字先转换为整数类型，再通过哈希函数，将其映射到哈希表中。** 而关于整数类型的关键字，通常用到的哈希函数方法有：直接定址法、除留余数法、平方取中法、基数转换法、数字分析法、折叠法、随机数法、乘积法、点积法等。


<font size =5>哈希冲突处理</font>

哈希冲突：不同的关键字通过同一个哈希函数可能得到同一哈希地址，即 key1 ≠ key2，而Hash(key1) = Hash(key2)，这种现象称为哈希冲突。

<font size=4>解决方法</font>

<font size=4>开放地址法</font>

开放地址法：**指的是将哈希表中的「空地址」向处理冲突开放(再找过一个地址给它)**。当哈希表未满时，处理冲突时需要**尝试另外的单元**，直到找到空的单元为止。**H(i) = (Hash(key) + F(i)) \% m，i = 1, 2, 3, ..., n (n ≤ m - 1)**

​     H(i) 是在处理冲突中得到的地址序列。即在第 1 次冲突（i = 1）时经过处理得到一个新地址 H(1)，如果在 H(1) 处仍然发生冲突（i = 2）时经过处理时得到另一个新地址 H(2) …… 如此下去，直到求得的 H(n) 不再发生冲突
​      Hash(key) 是**哈希函数**，m 是**哈希表表长**，取余目的是为了使得到的**下一个地址一定落在哈希表**中
​      F(i) 是**冲突解决方法**，取法可以有以下几种：
​           线性探测法：F(i) = 1, 2, 3, …, m - 1
​           二次探测法：F(i) = 1^2, -1^2, 2^2, -2^2, …, n^2(n ≤ m / 2)
​           伪随机数序列：F(i) = 伪随机数序列

<font size=4>链地址法</font>

链地址法：将**具有相同哈希地址的元素（或记录）存储在同一个线性链表中**。 链地址法是一种更加常用的哈希冲突解决方法。相比于开放地址法，链地址法更加简单。 假设哈希函数产生的哈希地址区间为 [0, m - 1]，哈希表的表长为 m。则可以将哈希表定义为一个有 m 个头节点组成的链表指针数组 T。

- 这样在插入关键字的时候，只需要通过哈希函数 Hash(key) 计算出对应的哈希地址 i，然后将其以链表节点的形式插入到以 T[i] 为头节点的**单链表**中。在链表中插入位置可以在表头或表尾，也可以在中间。如果每次插入位置为表头，则插入操作的时间复杂度为 O(1)。

+ 而在在查询关键字的时候，只需要通过哈希函数 Hash(key) 计算出对应的哈希地址 i，然后将对应位置上的链表**整个扫描一遍**，比较链表中每个链节点的键值与查询的键值是否一致。查询操作的时间复杂度跟链表的长度 k 成正比，也就是 O(k)。对于哈希地址比较均匀的哈希函数来说，理论上讲，k= n//m，其中 n 为关键字的个数，m 为哈希表的表长。

相对于开放地址法，采用链地址法处理冲突要**多占用一些存储空间**（主要是链节点占用空间）。但它可以减少在进行插入和查找具有相同哈希地址的关键字的操作过程中的平均查找长度。这是因为在链地址法中，待比较的关键字都是具有相同哈希地址的元素，而在开放地址法中，待比较的关键字不仅包含具有相同哈希地址的元素，而且还包含哈希地址不相同的元素。



# 3.算法

## 3.1算法简介

算法是什么东西？
它是一种方法，一种**解决问题的方案**。
举个例子，你现在要去上班，可以选择 走路、跑步、坐公交、坐地铁、自己开车 等等，这些都是**解决方案**。但是它们都会有一些衡量指标，让你有一个权衡，最后选择你认为最优的策略去做。
而衡量的指标诸如：时间消耗、金钱消耗、是否需要转车、是否可达 等等。

时间消耗就对应了：**时间复杂度**
金钱消耗就对应了：**空间复杂度**
是否可达就对应了：**算法可行性**

- 当然，是否需要转车，从某种程度上都会影响 **时间复杂度** 或者 **空间复杂度**。

## 3.2排序

### 3.2.1选择排序

O(n^2)

   选择和冒泡排序本质上都是把一个最值元素(最大或最小)在每轮检测中，和数组当前给定范围内的最右或最左的元素交换，只不过**选择**是**记录这个最值的下标**，**最后**再进行交换，**冒泡**是直接**不断的和相邻元素交换**，直到最右或最左

   冒泡看上去比选择麻烦，实际结构更加稳定，数值相同的元素的前后不会被改变，但是牺牲了时间复杂度

   选择排序：通过比较，选出每一轮中最值元素（最大或最小），然后把它和本轮中的第一个元素进行交换，所以这个算法的关键是要记录每次比较的结果，即每次比较后的最值位置（下标）。


整数数组长度----->length = sizeof(ar) / sizeof(ar[0])

```c++
//选择排序 小端
template<typename T>
void  Solution::selectSort(T ar, int length)
{
	int i = 0;
	int min_index = 0;
	array_length = length;
	for (i; i < length ; i++)//选择目标元素
	{
		min_index = i;
		for (int j = i + 1; j < length; j++)//记录当前范围(i---length)内的最小值的下标
		{
			if (ar[j] < ar[min_index])
			{
				min_index = j;
			}
		}
		swap(ar, i, min_index);
	}
	
	show_array(ar);
	return;
}
```

### 3.2.2冒泡排序 

O(n^2) 

冒泡排序：从数组的第一个元素开始（arr[0]）,两两比较（arr[n], arr[n+1]),如果前面的数大于后面的数，则交换两个元素的位置，把大的数往后移动。经过一轮比较后，最大的数会被交换到最后的位置（arr[n-1]）。

```c++
//冒泡排序  小端
template<typename T>
void Solution::BubbleSort(T ar, int length)
{
	array_length = length;
	for (int i = length - 1; i > 0; i--)
        //第二个循环的范围(0---i)
	{
		for (int j = 0; j < i; j++)
         //找出给定范围内的最小数,赋值给当前i下标的元素
		{
			if (ar[j] > ar[j + 1])
				swap(ar, j, j + 1);
		}
	}
    
	show_array(ar);
	return;
}

```

### 3.2.3插入排序

O(n^2)

```c++
template<typename T>
void Solution::InsertSort(T ar, int length)
{
	array_length = length;
	for (int i = 1; i < length; i++)
		for (int j = i - 1; j >= 0 && ar[j] > ar[j + 1]; j--)
			swap(ar, j, j + 1);
	show_array(ar);
	return;
}

```

求中间值 middle = L + (( R - L ) > > 1) 右移一位就相当于除以了2



### 3.2.4归并排序

把一个数组分割成两个，先把这两个数组都排序好，再比较这两个数组的第一个元素的下标，小的(小端)放进一个空数组，然后对应的下标后移，直到越界











## 3.3异或运算(*)

相同为0，不同为1, A^B, 没有进位的相加

满足交换律和结合律 

要求a b在内存里面的位置不同，对于数组来说就是下标不能相同，相同的就会把这块内存清零

加个判断就行

a = a ^ b  -- a ^ b

b = a ^ b  -- a ^ b ^ b -- a

a = a ^ b   -- a ^ b ^ a   -- b

```C++
//给定一个数组，有一个数在数组中出现了奇数次，其他数都是偶数次，要求时间复杂度为O(n)，空间复杂度为O(1)

void check(int ar[], int len = 8)
{
	int target = 0;

	for (int i = 0; i < len; i++)
	{
		target = target ^ ar[i];
	}
	printf("%d", target);

}


//给定一个数组，有两个数在数组中出现了奇数次，其他数都是偶数次

void check(int ar[], int len = 8)
{
	int target = 0;
	int OnlyOne = 0;
	for (int i = 0; i < len; i++)
	{
		target = target ^ ar[i];

	}
    //获得target的补码与target的与，就是target二进制的最右边的1对应的数(注意这个数不是1，它只是除了这个位上是1其他位为0)
	int rightOne = target & (~target + 1);
    
	for (int i = 0; i < len; i++)
	{
		if ((ar[i] & rightOne) == 0)
            //获得所有在这一位上面为0的数
            //获得为1的数就把0改成rightOne
			OnlyOne ^= ar[i];
	}
	printf("%d  %d", OnlyOne, target^OnlyOne);

}



```

![image-20220827205026037](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220827205026037.png)



## 3.4二分法

![image-20220827220401301](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220827220401301.png)

在一个**有序数组**里面，找一个对应数，直接查看中间的那个数，根据比较的大小可以省去查找另外一边的数

O(log2^N)  根据需要分割多少次 O(logN)=O(log2^N)



在一个**无序数组**中，相邻两个数必定不相等，求局部最小数

（一个数比它左右相邻两个数都要小）

因为在一个范围内，两边界都不满足局部最小，则这个范围内的数值会形成一个边界为类似二元一次方程曲线的数值曲线，又因为相邻数不相等，则必定这个曲线会有一个拐点(比左右都要小)



<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220828150943543.png" alt="image-20220828150943543" style="zoom:50%;" />

代码实现

```c++
template<typename T>
int Solution::Dichotomy(T ar, int length)
{
	int middle = 0;
	int left = 0;
	int right = length - 1;
	if (ar[0] < ar[1])
		return ar[0];
	else if (ar[length - 1] < ar[length - 2])
		return ar[length - 1];
	while(1)
	{
		middle = (right - left) / 2;
		if (ar[middle] < ar[middle + 1] && ar[middle] < ar[middle - 1])
			return ar[middle];
		if (ar[middle] < ar[middle + 1])
			right = middle;
		else if (ar[middle] < ar[middle - 1])
			left = middle;
	}

}
```

## 3.5对数器

假设有一个算法，要测试其可行性和性能优劣，可以把这个算法的最简单（暴力实现之类的，一定不会出错的最简单实现，不考虑任何复杂度）实现也写出来，然后用一个随机样本产生器，产生随机的样本数据交给两个算法，比对结果相同与否，从而得出算法的可行性



## 3.6递归

相当于把运算放进系统栈，有不确定的结果，就把它压入栈，算出结果了，就把结果弹出栈 ,每一个节点展开就是一个多叉树,

master公式

子问题的规模要一样

```tex
T(N) = a * T (N / b)  + O (N ^ d)  

T(N) 母问题的数据量 

a 子问题的执行次数 T(N /b)子问题的数据量
N/b 子问题的规模，子问题的规模要一样

O(N ^ d)  除子问题以外剩下的过程的时间复杂度


```

```C++
int process(int []ar , int L,int R)
{  
    int mid = L + ( (R - L) >> 1);
    int lef = process(ar,L,mid);
    int right =process(ar,mid+1,R);
    
    return lef>right?lef:right;
    
}
//master公式 T(N)=2*(T/2) + O(1 )
```

![image-20220829092656304](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220829092656304.png)











## 反转链表

递归

```c++
/*
struct ListNode {
    int val;
    struct ListNode *next;
    ListNode(int x) :
            val(x), next(NULL) {
    }
};*/
class Solution {
public:
    ListNode* ReverseList(ListNode* pHead) {
        //特判：注意不要漏掉pHead->next==NULL的情况
    if(pHead==NULL || pHead->next==NULL){
        return pHead;
    }
        //递归调用
        ListNode* ans = ReverseList(pHead->next);
        //让当前结点的下一个结点的 next 指针指向当前节点
        pHead->next->next=pHead;
       //同时让当前结点的 next 指针指向NULL ，从而实现从链表尾部开始的局部反转
        pHead->next=NULL;
        return ans;
    }
};
```

非递归

```c++
// c++
class Solution {
public:
 ListNode* ReverseList(ListNode* pHead) {
     ListNode* pre = nullptr;
     while (pHead) { // 判断链表是否已经到结尾
         ListNode* t = pHead->next; // 保留下一个即将翻转的表头
         pHead->next = pre; // 将现在要翻转的节点的后继指向前一个节点 pre
         pre = pHead; // 现在的 P 成为下一个前继节点
         pHead = t; // p 成为下一个要翻转的节点
     }
     return pre;
 }
};
```



# 4.设计模式

## 4.1单例模式

​        其意图是保证**一个类仅有一个实例**，并提供一个访问它的全局访问点，该实例被所有程序模块共享。

定义一个单例类：

1. **私有化它的构造函数**，以防止外界创建单例类的对象；
2. **使用类的私有静态指针变量指向类的唯一实例**；
3. 使用一个**公有的静态方法获取该实例**。

### 4.1.1懒汉模式

懒汉版：单例实例在第一次被使用时才进行初始化，这叫做延迟初始化。就是不到调用*getInstance*函数时，这个类的对象是一直不存在的。**懒汉本身是线程不安全的**。

```c++
// version 1.0
class Singleton
{
private:
	static Singleton* instance;//注意这是一个指针，饿汉的是一个实例
private://私有化所有构造函数，析构函数
	Singleton() {};
	~Singleton() {};
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
public:
	static Singleton* getInstance() 
        {
		if(instance == NULL) 
			instance = new Singleton();
		return instance;
	}
};
 
// init static member
Singleton* Singleton::instance = NULL;
```

**问题1：**懒汉模式存在内存泄露的问题，有两种解决方法：

1. 使用智能指针
2. 使用静态的嵌套类对象

```c++
// version 1.1，第二种解决方法
class Singleton
{
private:
	static Singleton* instance;
private:
	Singleton() { };
	~Singleton() { };
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
private:
	class Deletor {
	public:
		~Deletor() {
			if(Singleton::instance != NULL)
				delete Singleton::instance;
		}
	};
	static Deletor deletor;
public:
	static Singleton* getInstance() {
		if(instance == NULL) {
			instance = new Singleton();
		}
		return instance;
	}
};
 
// init static member
Singleton* Singleton::instance = NULL;
```

在程序运行结束时，系统会调用静态成员deletor的析构函数，该析构函数会删除单例的唯一实例。使用这种方法释放单例对象有以下特征：

​        **在单例类内部定义专有的嵌套类。**
​        **在单例类内定义私有的专门用于释放的静态成员。**
​        **利用程序在结束时析构全局变量的特性，选择最终的释放时机。**

在单例类内再定义一个嵌套类，总是感觉很麻烦。


问题2：这个代码在单线程环境下是正确无误的，但是当拿到多线程环境下时这份代码就会出现race condition，注意version 1.0与version 1.1都不是线程安全的。要使其**线程安全**，能在多线程环境下实现单例模式，我们首先想到的是利用同步机制来正确的保护我们的shared data。可以使用双检测锁模式（DCL: Double-Checked Locking Pattern）：


```c++
static Singleton* getInstance() {
	if(instance == NULL) {
		Lock lock;  // 基于作用域的加锁，超出作用域，自动调用析构函数解锁
        if(instance == NULL) {
        	instance = new Singleton();
        }
	}
	return instance;
}
```



注意，线程安全问题仅出现在第一次初始化（new）过程中，而在后面获取该实例的时候并不会遇到，也就没有必要再使用lock。双检测锁很好地解决了这个问题，它通过加锁前检测是否已经初始化，避免了每次获取实例时都要首先获取锁资源。

加入DCL后，其实还是有问题的，关于memory model。在某些内存模型中（虽然不常见）或者是由于编译器的优化以及运行时优化等等原因，使得instance虽然已经不是nullptr但是其所指对象还没有完成构造，这种情况下，另一个线程如果调用getInstance()就有可能使用到一个**不完全初始化的对象**。换句话说，就是代码中第2行：if(instance == NULL)和第六行instance = new Singleton();没有正确的同步，在某种情况下会出现new返回了地址赋值给instance变量而Singleton此时还没有构造完全，当另一个线程随后运行到第2行时将不会进入if从而返回了不完全的实例对象给用户使用，造成了严重的错误。在C++11没有出来的时候，只能靠插入两个memory barrier（内存屏障）来解决这个错误，但是C++11引进了memory model，提供了Atomic实现内存的同步访问，即不同线程总是获取对象修改前或修改后的值，无法在对象修改期间获得该对象。

因此，在有了C++11后就可以正确的跨平台的实现DCL模式了，利用atomic，代码如下：

```c++
atomic<Widget*> Widget::pInstance{ nullptr };
Widget* Widget::Instance() {
    if (pInstance == nullptr) { 
        lock_guard<mutex> lock{ mutW }; 
        if (pInstance == nullptr) { 
            pInstance = new Widget(); 
        }
    } 
    return pInstance;
}
```

C++11中的atomic类的默认memory_order_seq_cst保证了3、6行代码的正确同步，由于上面的atomic需要一些性能上的损失，因此我们可以写一个优化的版本：

```c++
atomic<Widget*> Widget::pInstance{ nullptr };
Widget* Widget::Instance() {
    Widget* p = pInstance;
    if (p == nullptr) { 
        lock_guard<mutex> lock{ mutW }; 
        if ((p = pInstance) == nullptr) { 
            pInstance = p = new Widget(); 
        }
    } 
    return p;
}
```

**Best of All:**

C++11规定了local static在多线程条件下的初始化行为，**要求编译器保证了内部静态变量的线程安全性**。在C++11标准下，《Effective C++》提出了一种更优雅的单例模式实现，使用函数内的 local static 对象。这样，只有当第一次访问getInstance()方法时才创建实例。这种方法也被称为Meyers' Singleton。C++0x之后该实现是线程安全的，C++0x之前仍需加锁。local static----静态局部

```c++
// version 1.2
class Singleton
{
private:
	Singleton() { };
	~Singleton() { };
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
public:
	static Singleton& getInstance() 
        {
		static Singleton instance;
		return instance;
	}
};
```

### 4.1.2饿汉模式

   饿汉模式在单例类定义的时候（即在main函数之前）就进行实例化。因为main函数执行之前，全局作用域的类成员静态变量m_Instance已经初始化，故没有多线程的问题。

```c++
// version 1.3
class Singleton
{
private:
	static Singleton instance;
private:
	Singleton();
	~Singleton();
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
public:
	static Singleton& getInstance() {
		return instance;
	}
}
 
// initialize defaultly
Singleton Singleton::instance;
```

​        由于在main函数之前初始化，所以没有线程安全的问题。但是潜在问题在于no-local static对象（函数外的static对象）在不同编译单元中的**初始化顺序**是未定义的。也即，static Singleton instance;和static Singleton& getInstance()二者的初始化顺序不确定，如果在初始化完成之前调用 getInstance() 方法会返回一个未定义的实例。


总结：

- Eager Singleton 虽然是线程安全的，但存在潜在问题；
- Lazy Singleton通常需要**加锁来保证线程安全**，但局部静态变量版本在C++11后是线程安全的；
- **局部静态变量版本**（Meyers Singleton）最优雅。







