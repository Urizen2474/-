# 1.Lua

## 1.环境配置

[(32条消息) Windows10下使用VS2019搭建Lua开发环境_看看你的头发的博客-CSDN博客_lua vs2019](https://blog.csdn.net/weixin_43603958/article/details/109015563)

## 1.变量

camel命名法:首个单词小写，后面的单词首字母大写(类名方法名首字母大写)

所有的变量初始化则默认创建,并且默认为全局变量

可以同时给多个变量赋值

```lua
a=1  --[[默认没有分号]]

local a=1 使其成为局部变量

a,b=1,2 a为1，b为2

a="asdasd"
b='qdqwd'都是字符串

连接字符串->a..b(两个点..)

tostring()
tonumber(string)  [[转换失败为nil]]

字符串前面加一个#就可以获得字符串长度->#a=6


```

所有未声明的变量的默认值为nil(相当于null)

```lua
c=[[....]]  双括号内的任意值的都会被原封不动保存，即使转义字符
```

## 2.函数

```lua
function funcname(...)
    --body
    end

function funcname(...)
    --body
    return ...
    end  ---lua支持同时返回多个值


```

## 3.Table（表）

### 3.1数字下标

```lua
a={2,"asd",a,function}   [[Lua下标从1开始，并且可以包含任意类型的值]]
    
    对数组名进行#也可以获得其长度
    
table.insert(数组名，插入点的下标(可省略，默认为数组末尾)，插入值)
    
table.remove(数组名，删除点下标) [[remove还会返回删除掉的数组元素的值]]
```

### 3.2字符串下标

```lua
a={a=231,b="ad",c=function()end,d={},[";ad'"]=123} [[相当于字典]]

a["sdfg"]=34234  [[直接加入，不需要数组内也存在]]

调用:
a.a [[a数组的a下标]]
a["a"] 同上
a[;ad']  [[也可以自定义字符串下标]]
```

### 3.3全局表(_G)

```lua
程序中所有的全局变量都存储在全局表_G中
```

## 4.逻辑运算符

```lua
除了不等于是~=(波浪+等于号)，其他都是相同的
```

```lua
and or not 可以直接用(要小写)    只用false和nil是假，其他都是真

逻辑运算得到不是只有true和false，而是对应的值

print(a or b)  [[a=3,b=4,---输出3]]\
print(a and b) 


or返回值在表达式为true的情况下，永远会输出为true的变量的值
or返回值在表达式为false的情况下，永远会输出右边的那个变量的值

and返回值在表达式为false的情况下，永远会输出左边的那个变量的值
and返回值在表达式为true的情况下，永远会输出为右边的变量的值




```

<font size=5>逻辑控制语句</font>

```lua
if else语句

if 表达式 then
    --body
    else
    --body
    else if 表达式 then
    --body
    end
```



```lua
for语句

for i = 1, 10, 3 do  [从1开始，到10结束，每次循环i加3，循环中作为循环条件的i不可以修改]
    --body
    end

Lua不支持自减自加 --，-=，+，+=

while语句

while 表达式 do
    --body
    end



```

## 5.Require

运行指定文件，不需要后缀名,顺便还可以使用文件内的全局变量

```lua
require("目标文件名")
```

![image-20220804153430851](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804153430851.png)

VSCode需要在文件前面加入文件路径层级，VS不用

只会运行一次（不论一个程序调用多少次）

lua文件可以返回一个值(不是和函数一样)

require有一个返回值，就是打开文件的返回值

### 5.1多次使用require文件内的内容

调用文件

![image-20220804155652639](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804155652639.png)

被调用文件

![image-20220804155738929](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804155738929.png)

```lua
table. 可以直接把变量加入进table里面
```

## 6.迭代器

### 6.1数字下标

```lua
a={"a","asd","as",231}
for i,j in ipairs(t) do   i->下标 j->数组值 
    ---body
    end
[[循环至下标无效的值循环就会停止]]

```

### 6.2字符下标

把ipairs改成pairs就行

<font color=red>但是pairs对于不连续的下标也会输出，而ipairs则会终止循环</font>

![image-20220804185934874](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804185934874.png)

只会输出一个cb，并且还是先输出

```lua
pairs其实是调用next函数
next(a)  ---ajw
next(a,"ajw")  ---cb
next会返回给定下标的下一个下标对应的值，如果没有下一个则返回nil
```

## 7.字符串

lua里面的字符串成员是一个字节一个字节的存的，并且\0也可以存储，还不会影响数组长度

![image-20220804191639566](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804191639566.png)

实际调用:  

```lua
a={}
a:byte(1)  [[表名:函数名(下标)]]
```

### 7.1正则

![image-20220804200353530](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804200353530.png)

转义用%

![image-20220804200443421](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804200443421.png)

#### 7.1.1正则混合

```lua
[%d%a] ---既可以匹配一个数字，也可以匹配一个字母,相当于是一个或的集合，只能匹配一个
```

<font color=green>注:正则表达式要用""包裹</font>

![image-20220804202721995](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804202721995.png)

所有修饰符只对其前面的那一个正则字符起作用(所以可以用[])    

* *尽可能长的匹配,可以为零

    +      +尽可能长的匹配，至少匹配到一个，不然匹配失败
    +     -尽可能短的匹配，可以为零，所以一般会在后面添加一个限制
    +      ？只匹配一个，和不加没什么区别

加了括号的正则可以被match直接获取，而忽略括号外的匹配到的字符串(但它仍参与匹配，只是不出现在最终的值里面)

## 8.元表

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220804205629439.png" alt="image-20220804205629439" style="zoom:80%;" />

metatable相当于给普通的表赋予了自定义的运算的功能，相当于 t.__add(t,temp)

也可以像异常一样使用，向__index

```lua
在给表的不存在的下标赋值或获取时(__index和   __newindex)

__index是针对不存在下标的获取
__newindex是针对不存在下标的赋值
作为表时相同，作为函数时，index是(table, key)，newindex(table, key, value)

__index当直接获取表不存在的下标的时候，系统就会就去找这个表的元表里面有没有__index
在查找__index有没有这个下标，如果没有仍是nil

__newindex当执行给不存在下标的值赋值时，将忽略原有操作，直接执行元表中的__newindex
，可以用rawset(table,key,value)在__newindex里面赋值(调用如:temp[xx]=xx,会递归调用__newindex,导致栈溢出)


```

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220805101831675.png" alt="image-20220805101831675" style="zoom:200%;" />

## 9.面向对象

实现

```lua
 local bag = {}
 local bagmt =   
 {
     put = function(t, item)
       table.insert(t.items, item)
     end,
     take = function(t, item)
        return table.remove(t.items, item)
     end,
     list = function(t)
       return table.concat(t.items, ", ")
     end,
 }
 bagmt["__index"] = bagmt   --自己就是自己的元表，当t被访问了不存在的下标时，会找bagmt里面的__index, 而bagmt的__index是他自己，则就是在他自己的成员里面找这些下标
 function bag.new()     --相当于构造函数
      local t =         --相当于是实例化的对象
      {
         items = {}      --负责存储对应值
      }
      setmetatable(t, bagmt)    --给他个元表，存储这个对象的函数
      return t          --构造结束获得的对象
 end
 local temp = bag.new()        
 temp:put("ajwcb")     
 temp:put("ajwcb")
 temp:put("ajwcb")
 temp:put("ajwcb")
 temp:put("ajwcb")
 print(temp:list())
```

## 10.协程coroutine(单线程)

Lua不存在多线程，协程并不是同时进行多个程序

coroutine.create(func)  只会把一个函数作为一个新协程创建，并返回一个thread对象



![image-20220805144041110](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220805144041110.png)

yield可以有参数，并传给它暂停掉的那个resume

resume也可以有参数，并传给它开启的那个yield

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220805145223394.png" alt="image-20220805145223394" style="zoom: 50%;" />

协程只是用户态的堆栈切换，让cpu切换堆栈，调用不同的函数而已，协程可以认为是一个任务（函数）。有一个调度器管理这些函数，函数可以yeild 这样调度器就让cpu切换到下一个任务，即函数。

![image-20220805154350191](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220805154350191.png)

create虽然功能和wrap相同，但是只有thread类型的变量才可以作为参数去调用其他的协程函数







## 11.C++嵌入Lua

**C++调用Lua文件**

首先，打开main.cpp，并包含以下头文件：

```Lua
#include "lua.hpp"
```



如果此时编译不报错，则说明你之前设置的search path是正确的，如果报错，请自行调整search path。

这里面的lua.hpp的内容如下：

```
extern "C" {

#include "lua.h"

#include "lualib.h"

#include "lauxlib.h"

}
```



这才是大多数Lua教程里的代码嘛。然后在main函数里面添加以下内容：



```lua
 复制代码
/1. 初始化Lua虚拟机

lua_State *lua_state;

lua_state = luaL_newstate();

//2.设置待注册的Lua标准库，这个库是给你的Lua脚本用的

//因为接下来我们只想在Lua脚本里面输出hello world，所以只引入基本库就可以了

static const luaL_Reg lualibs[] =

{

    { "base", luaopen_base },

    { NULL, NULL}

};

//3.注册Lua标准库并清空栈

const luaL_Reg *lib = lualibs;

for(; lib->func != NULL; lib++)

{

    luaL_requiref(lua_state, lib->name, lib->func, 1);

    lua_pop(lua_state, 1);

}

//4、运行hello.lua脚本

luaL_dofile(lua_state, "hello.lua");

//5. 关闭Lua虚拟机

lua_close(lua_state);
```

### 11.1C++调用Lua

​          

| C的栈的下标 | 对应的数据 | Lua的栈的下标 |
| ----------- | ---------- | ------------- |

![image-20220806150152686](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220806150152686.png)



在Lua和C语言之间进行数据交换时，由于两种语言之间有着较大的差异，比如Lua是动态类型，C语言是静态类型，Lua是自动内存管理，而C语言则是手动内存管理。为了解决这些问题，Lua的设计者使用了**虚拟栈作为二者之间数据交互的介质**。在C/C++程序中，**如果要获取Lua的值，只需调用Lua的C API函数，Lua就会将指定的值压入栈中。要将一个值传给Lua时，需要先将该值压入栈，然后调用Lua的C API，Lua就会获取该值并将其从栈中弹出**。为了可以将不同类型的值压入栈，以及从栈中取出不同类型的值，Lua为**每种类型**均设定了一个特定函数。



## 12.lua热更新

### 12.1简单描述

​     Lua的热更新流程简单地说就是：从服务器下载"资源校验表"，对比MD5码异同，MD5码不同的文件就是需要更新的文件

​     Tips：Application.persistentDataPath，热更的重要路径，该文件夹可读可写，在移动端唯一一个可读写操作的文件夹。

​     移动端可以将本地的资源（资源MD5值配置表）等一些文件放到StreamingAssets文件夹下，通过Copy到persistentDataPath下与服务器的版本文件配置表作比对，完成资源的热更。

​     为什么不在StreamingAsset文件夹下直接操作？因为该文件夹只读，不可写，资源无法更新进去。

​     为什么不在persistentDataPath文件夹操作，因为该文件夹是apk安装以后，才会形成的一个文件夹，无法提前创建


![img](https://img-blog.csdnimg.cn/d635779ce4d74d249956a473b33e2130.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Im65pyv5bCx5pivQ3RybEM=,size_20,color_FFFFFF,t_70,g_se,x_16)



