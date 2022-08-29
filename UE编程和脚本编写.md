# UE编程和脚本编写

## 1.C++编程

### 1.1虚幻引擎中的容器

#### 1.1.1TArray：虚幻引擎中的数组

虚幻引擎4（UE4）中最简单的容器类是 `TArray`。`TArray` 负责同类型其他对象（称为"元素"）序列的所有权和组织。由于 `TArray` 是一个序列，其元素的**排序定义明确**，其函数用于确定性地操纵此类对象及其顺序。

##### 1.1.1.1TArray

指数--->index

`TArray` 是UE4中最常用的容器类。其速度快、内存消耗小、安全性高。`TArray` 类型由两大属性定义：元素类型和可选分配器。

元素类型是存储在数组中的对象类型。`TArray` 被称为同质容器。换言之，其所有元素均完全为相同类型。单个 `TArray` 中不能存储不同类型的元素。

分配器常被省略，默认为最常用的分配器。其**定义对象在内存中的排列方式**；以及数组如何进行扩展，以容纳更多的元素。若默认行为不符合要求，可选取多种不同的分配器，或自行编写。此部分将稍后讨论。

`TArray` 为数值类型。意味其与其他**内置类型**（如 `int32` 或 `浮点`）的处理方式相同。其设计时未考虑扩展问题，因此建议在实际操作中勿使用 `新建（new）` 和 `删除（delete）` 创建或销毁 `TArray` 实例。元素也为数值类型，为容器所拥有。`TArray` 被销毁时其中的元素也将被销毁。若在另一TArray中创建TArray变量，其元素将复制到新变量中，且不会共享状态。

##### 1.1.1.2创建和填充数组

```C++
TArray<int32> IntArray;
```

此操作会创建用于存储整数序列的空白数组。元素类型是可根据普通C++值规则进行复制和销毁的数值类型，例如 `int32`、`FString`、`TSharedPtr` 等。由于无指定分配器，因此 `TArray` 将采用**基于堆**的默认分配器。此时尚未分配内存。

有多种填充 `Tarray` 的方法。一种方式是使用 `Init` 函数，将大量元素副本填入数组。

```c++
IntArray.Init(10, 5);
// IntArray == [10,10,10,10,10]
```

`Add` 和 `Emplace` 函数用于在数组末尾新建对象：

```c++
TArray<FString> StrArr;
StrArr.Add    (TEXT("Hello"));
StrArr.Emplace(TEXT("World"));
// StrArr == ["Hello","World"]
```

新元素添加到数组时，数组的分配器将根据需要分配内存。当前数组大小超出时，默认分配器将添加足够的内存，用于存储多个新元素。`Add` 和 `Emplace` 函数的多数效果相同，细微区别在于：

- `Add`（或 `Push`）将元素类型的**实例复制**（或移动）到数组中。
- `Emplace` 使用给定参数构建元素类型的**新实例**。

因此在 `TArray<FString>` 中，`Add` 将用字符串文字创建临时 `FString`，然后将该临时 `FString` 的内容移至容器内的新 `FString` 中；而 `Emplace` 将用字符串文字直接新建 `FString`。最终结果相同，但 `Emplace` 可避免创建临时文件。对于 `FString` 等非浅显数值类型而言，临时文件通常有害无益。

总体而言，`Emplace` 优于 `Add`，因此其可避免在调用点创建无需临时变量，并将此类变量复制或移动到容器中。根据经验，可将 `Add` 用于浅显类型，将 `Emplace` 用于其他类型。`Emplace` 的效率**始终高于** `Add`，但 `Add` 的可读性可能更好。//用Emplace就完事了

利用 `Append` 可一次性添加其他 `TArray` 中的多个元素，或者指向常规C数组的指针及该数组的大小：

```c++
FString Arr[] = { TEXT("of"), TEXT("Tomorrow") };
StrArr.Append(Arr, ARRAY_COUNT(Arr));
// StrArr == ["Hello","World","of","Tomorrow"]
```

仅在尚不存在等值元素时，`AddUnique` 才会向容器添加新元素。使用以下元素类型的运算符检查等值性：`运算符==`:

```c++
StrArr.AddUnique(TEXT("!"));
// StrArr == ["Hello","World","of","Tomorrow","!"]

StrArr.AddUnique(TEXT("!"));
// StrArr is unchanged as "!" is already an element
```

与 `Add`、`Emplace` 和 `Append` 相同，`Insert` 将在给定索引处添加单个元素或元素数组的副本：

```c++
StrArr.Insert(TEXT("Brave"), 1);
// StrArr == ["Hello","Brave","World","of","Tomorrow","!"]
```

`SetNum` 函数可直接设置数组元素的数量。如新数量大于当前数量，则使用元素类型的默认构造函数新建元素：

```c++
StrArr.SetNum(8);
// StrArr == ["Hello","Brave","World","of","Tomorrow","!","",""]
```

如新数量小于当前数量，`SetNum` 将移除元素。移除元素的更多相关详情将稍后讨论：

```c++
StrArr.SetNum(6);
// StrArr == ["Hello","Brave","World","of","Tomorrow","!"]
```

##### 1.1.1.3迭代

有多种方法可迭代数组的元素，建议使用C++的范围（ranged-for）功能：

```c++
FString JoinedStr;
for (auto& Str :StrArr)//TArray<FString>StrArr
{
    JoinedStr += Str;
    JoinedStr += TEXT(" ");
}
// JoinedStr == "Hello Brave World of Tomorrow !"
```

同时也可使用基于索引的常规迭代：

```c++
for (int32 Index = 0; Index != StrArr.Num(); ++Index)
{
    JoinedStr += StrArr[Index];
    JoinedStr += TEXT(" ");
}
```

最后，还可通过数组迭代器类型控制迭代。函数 `CreateIterator` 和 `CreateConstIterator` 可分别用于元素的读写和只读访问：

```c++
for (auto It = StrArr.CreateConstIterator(); It; ++It)
{
    JoinedStr += *It;
    JoinedStr += TEXT(" ");
}
```

##### 1.1.1.4排序

调用 `Sort` 函数即可对数组进行排序：

```c++
StrArr.Sort();
// StrArr == ["!","Brave","Hello","of","Tomorrow","World"]
```

在此，数值按元素类型的 `运算符<` 排序。在FString中，此为忽略大小写的词典编纂比较。二元谓词也在实现后提供不同的排序语义，例如：

```c++
StrArr.Sort([](const FString& A, const FString& B) 
{
    return A.Len() < B.Len();//lambda表达式
});
// StrArr == ["!","of","Hello","Brave","World","Tomorrow"]
```

字符串现在按长度排序。注意：与之前相比，数组中三个长度相同的字符串"Hello"、"Brave"和"World"的相对排序发生了变化。这是因为 `Sort` **不稳定**，等值元素（因为断言只比较长度，所以此处字符串为等值）的相对排序无法保证。`Sort` 作为quicksort实现。

`HeapSort` 函数，无论是否使用二元谓词，均可用于执行堆排序。使用HeapSort函数与否，**取决于特定数据与Sort函数相比时的排序效率**。与 `Sort` 一样，`HeapSort` **也不稳定**。若在上述范例中使用 `HeapSort` 而非 `Sort`，结果将如下所示（此例中结果相同）：

```c++
StrArr.HeapSort([](const FString& A, const FString& B) {
    return A.Len() < B.Len();
});
// StrArr == ["!","of","Hello","Brave","World","Tomorrow"]
```

最后，`StableSort` 用于在排序后**保证等值元素的相对顺序**。若在上述范例中调用 `StableSort` 而非 `Sort` 或 `HeapSort`，结果将如下所示：

```c++
StrArr.StableSort([](const FString& A, const FString& B) {
    return A.Len() < B.Len();
});
// StrArr == ["!","of","Brave","Hello","World","Tomorrow"]
```

即：在进行词典编纂排序后，"Brave"、"Hello"和"World"的相对排序不会改变。`StableSort` 作为归并排序实现。

##### 1.1.1.5查询

使用 `Num` 函数可查询数组保存的元素数量：

```c++
int32 Count = StrArr.Num();
// Count == 6
```

如需直接访问数组内存（如确定C类API的互操作性），可使用 `GetData` 函数将指针返回到数组中的元素。仅在数组存在且未执行更改数组的操作时，此指针方有效。仅 `StrPtr` 的首个 `Num` 指数才可被解除引用：

```c++
FString* StrPtr = StrArr.GetData();
// StrPtr[0] == "!"
// StrPtr[1] == "of"
// ...
// StrPtr[5] == "Tomorrow"
// StrPtr[6] - undefined behavior
```

如容器为常量，则返回的指针也为常量。

可针对元素大小对容器进行询问：

```c++
uint32 ElementSize = StrArr.GetTypeSize();
// ElementSize == sizeof(FString)
```

要获取元素，可使用索引 `运算符[]` 将从零开始的索引传递给要获取的元素：

```c++
FString Elem1 = StrArr[1];
// Elem1 == "of"
```

传递小于0或大于等于Num()的无效索引将导致运行时错误。使用 `IsValidIndex` 函数询问容器，可确定特定索引是否有效：

```c++
bool bValidM1 = StrArr.IsValidIndex(-1);
bool bValid0  = StrArr.IsValidIndex(0);
bool bValid5  = StrArr.IsValidIndex(5);
bool bValid6  = StrArr.IsValidIndex(6);
// bValidM1 == false
// bValid0  == true
// bValid5  == true
// bValid6  == false
```

`运算符[]` 将返回引用。因此其还可用于改变数组中的元素（假定数组不为常量）。

```c++
StrArr[3] = StrArr[3].ToUpper();
// StrArr == ["!","of","Brave","HELLO","World","Tomorrow"]
```

与GetData函数相同：如数组为常量，`运算符[]` 将返回常量引用。还可使用 `Last` 函数从数组末端**反向索引**。索引默认为零。`Top` 函数是 `Last` 的同义词，唯一区别是其**不接受索引**//永远都是最后一个：

```c++
FString ElemEnd  = StrArr.Last();
FString ElemEnd0 = StrArr.Last(0);
FString ElemEnd1 = StrArr.Last(1);
FString ElemTop  = StrArr.Top();
// ElemEnd  == "Tomorrow"
// ElemEnd0 == "Tomorrow"
// ElemEnd1 == "World"
// ElemTop  == "Tomorrow"
```

可询问数组是否包含特定元素：

```c++
bool bHello   = StrArr.Contains(TEXT("Hello"));
bool bGoodbye = StrArr.Contains(TEXT("Goodbye"));
// bHello   == true
// bGoodbye == false
```

或询问数组是否包含与特定谓词匹配的元素：

```c++
bool bLen5 = StrArr.ContainsByPredicate([](const FString& Str){
    return Str.Len() == 5;
});
bool bLen6 = StrArr.ContainsByPredicate([](const FString& Str){//函数符
    return Str.Len() == 6;
});
// bLen5 == true
// bLen6 == false
bool ContainsByPredicate(Func fun);
{
    return fun(str);
}

```

使用 `Find` 函数族可查找元素。可使用Find确定元素是否存在**并返回其索引**：

```c++
int32 Index;
if (StrArr.Find(TEXT("Hello"), Index))
{
    // Index == 3
}
```

此操作会将 `Index` 设为首个找到的元素的索引。如存在重复元素而希望找到最末元素的索引，则使用 `FindLast` 函数：

```c++
int32 IndexLast;
if (StrArr.FindLast(TEXT("Hello"), IndexLast))
{
    // IndexLast == 3, because there aren't any duplicates
}
```

两个函数均会返回布尔，指出是否已找到元素，同时在找到元素索引时将其写入变量。

`Find` 和 `FindLast` 也可直接返回元素索引。如不将索引作为显式参数传递，这两个函数便会执行此操作。此将比上述函数更简洁，使用的函数则取决于特定需求或风格。

如未找到元素，将返回特殊 `INDEX_NONE` 值：

```c++
int32 Index2     = StrArr.Find(TEXT("Hello"));
int32 IndexLast2 = StrArr.FindLast(TEXT("Hello"));
int32 IndexNone  = StrArr.Find(TEXT("None"));
// Index2     == 3
// IndexLast2 == 3
// IndexNone  == INDEX_NONE
```

`IndexOfByKey` 的工作方式类似，不同元素可与任意对象比较。开始搜索前，使用 `Find` 函数会将参数实际转换为元素类型（此本例中为 `FString`）。使用 `IndexOfByKey`，可直接对不"键"，因此**即使键类型无法直接转换为元素类型，也可进行搜索。**

`IndexOfByKey` 适用于存在 `运算符==(ElementType, KeyType)` 的键类型。`IndexOfByKey` 将返回找到的首个元素的索引；如未找到元素，则返回 `INDEX_NONE`：

```c++
int32 Index = StrArr.IndexOfByKey(TEXT("Hello"));
// Index == 3
```

`IndexOfByPredicate` 函数用于查找与特定谓词匹配的首个元素的索引；如未找到，同样返回特殊 `INDEX_NONE` 值：

```c++
int32 Index = StrArr.IndexOfByPredicate([](const FString& Str){
    return Str.Contains(TEXT("r"));
});
// Index == 2
```

可返回指针并指向找到的元素，而不返回指数。`FindByKey` 与 `IndexOfByKey` 相似，将元素和任意对象进行对比，但返回指向所找到元素的指针。如未找到元素，则返回`nullptr`。

```c++
auto* OfPtr  = StrArr.FindByKey(TEXT("of")));
auto* ThePtr = StrArr.FindByKey(TEXT("the")));
// OfPtr  == &StrArr[1]
// ThePtr == nullptr
```

`FindByPredicate` 的使用方式和 `IndexOfByPredicate` 相似，不同点是返回指针而非索引：

```c++
auto* Len5Ptr = StrArr.FindByPredicate([](const FString& Str){
    return Str.Len() == 5;
});
auto* Len6Ptr = StrArr.FindByPredicate([](const FString& Str){
    return Str.Len() == 6;
});
// Len5Ptr == &StrArr[2]
// Len6Ptr == nullptr
```

最后，使用 `FilterByPredicate` 函数可获取与特定谓词匹配的元素数组：

```c++
auto Filter = StrArray.FilterByPredicate([](const FString& Str){
    return !Str.IsEmpty() && Str[0] < TEXT('M');
});
```

##### 1.1.1.6移除

`Remove` 函数族用于移除数组中的元素。`Remove` 函数将根据元素类型的 `运算符==` 函数移除**所有与提供元素等值**的元素。例如：

```c++
TArray<int32> ValArr;
int32 Temp[] = { 10, 20, 30, 5, 10, 15, 20, 25, 30 };
ValArr.Append(Temp, ARRAY_COUNT(Temp));
// ValArr == [10,20,30,5,10,15,20,25,30]

ValArr.Remove(20);
// ValArr == [10,30,5,10,15,25,30]
```

`RemoveSingle` 也可用于擦除数组中的首个匹配元素。以下情况尤为实用——此函数在数组中可能存在重复，而只希望擦除一个时；或作为优化，数组只能包含一个匹配元素时：

```c++
ValArr.RemoveSingle(30);
// ValArr == [10,5,10,15,25,30]
```

`RemoveAt` 函数也可用于按照从指定位置开始的索引移除元素。可使用 `IsValidIndex` 确定数组中的元素是否使用计划提供的索引，将无效索引传递给此函数会导致运行时错误：

```c++
ValArr.RemoveAt(2); // Removes the element at index 2
// ValArr == [10,5,15,25,30]

ValArr.RemoveAt(99); // This will cause a runtime error as
                       // there is no element at index 99
```

`RemoveAll` 也可用于函数移除与谓词匹配的元素。例如，移除为3倍数的所有数值：

```c++
ValArr.RemoveAll([](int32 Val) {
    return Val % 3 == 0;
});
// ValArr == [10,5,25]
```

在所有这些情况中，由于数组中不能出现空位，因此移除元素时其后的元素将被下移到更低指数中。（下标）

移动过程存在开销。如不需要剩余元素排序，可使用 `RemoveSwap`、`RemoveAtSwap` 和 `RemoveAllSwap` 函数减少此开销。此类函数的工作方式与其非交换变种相似，不同之处在于其**不保证剩余元素的排序**，因此可更快地完成任务：

```c++
TArray<int32> ValArr2;
for (int32 i = 0; i != 10; ++i)
    ValArr2.Add(i % 5);
// ValArr2 == [0,1,2,3,4,0,1,2,3,4]

ValArr2.RemoveSwap(2);
// ValArr2 == [0,1,4,3,4,0,1,3]

ValArr2.RemoveAtSwap(1);
// ValArr2 == [0,3,4,3,4,0,1]

ValArr2.RemoveAllSwap([](int32 Val) {
    return Val % 3 == 0;
});
// ValArr2 == [1,4,4]
```

最后，可使用 `Empty` 函数移除数组中所有元素：

```c++
ValArr2.Empty();
// ValArr2 == []
```

##### 1.1.1.7运算符

数组是常规数值类型，可使用标准复制构造函数或赋值运算符进行复制。由于数组严格拥有其元素，复制数组的操作是深层的，因此新数组将拥有其自身的元素**副本**：//不影响源

```c++
TArray<int32> ValArr3;
ValArr3.Add(1);
ValArr3.Add(2);
ValArr3.Add(3);

auto ValArr4 = ValArr3;
// ValArr4 == [1,2,3];
ValArr4[0] = 5;
// ValArr3 == [1,2,3];
// ValArr4 == [5,2,3];
```

作为 `Append` 函数的替代，可使用 `运算符+=` 对数组进行串联：

```c++
ValArr4 += ValArr3;
// ValArr4 == [5,2,3,1,2,3]
```

`TArray` 还支持移动语义，使用 `MoveTemp` 函数可调用这些语义。移动后，源数组必定为空：

```c++
ValArr3 = MoveTemp(ValArr4);
// ValArr3 == [5,2,3,1,2,3]
// ValArr4 == []
```

使用 `运算符==` 和 `运算符!=` 可对数组进行比较。元素的排序很重要：只有元素的顺序和数量相同时，两个数组才被视为相同。元素通过其自身的 `运算符==` 进行比较：

```c++
TArray<FString> FlavorArr1;
FlavorArr1.Emplace(TEXT("Chocolate"));
FlavorArr1.Emplace(TEXT("Vanilla"));
// FlavorArr1 == ["Chocolate","Vanilla"]

auto FlavorArr2 = Str1Array;
// FlavorArr2 == ["Chocolate","Vanilla"]

bool bComparison1 = FlavorArr1 == FlavorArr2;
// bComparison1 == true

for (auto& Str :FlavorArr2)
{
    Str = Str.ToUpper();
}
// FlavorArr2 == ["CHOCOLATE","VANILLA"]

bool bComparison2 = FlavorArr1 == FlavorArr2;
// bComparison2 == true, because FString comparison ignores case，FString的比较不论大小写

Exchange(FlavorArr2[0], FlavorArr2[1]);//交换1和2下标的值
// FlavorArr2 == ["VANILLA","CHOCOLATE"]

bool bComparison3 = FlavorArr1 == FlavorArr2;
// bComparison3 == false, because the order has changed
```

##### 1.1.1.8堆

`TArray` 拥有支持二叉堆数据结构的函数。**堆是一种二叉树**，其中父节点的排序等于或高于其子节点。作为数组实现时，树的根节点位于元素0，索引N处节点的**左右子节点的指数分别为2N+1和2N+2**。子节点彼此间不存在特定排序。

调用 `Heapify` 函数可将现有数组转换为堆。此会重载为是否接受谓词，无谓词的版本将使用元素类型的 `运算符<` 确定排序：

```c++
TArray<int32> HeapArr;
for (int32 Val = 10; Val != 0; --Val)
{
    HeapArr.Add(Val);
}
// HeapArr == [10,9,8,7,6,5,4,3,2,1]
HeapArr.Heapify();
// HeapArr == [1,2,4,3,6,5,8,10,7,9]
```

![image-20220719192418100](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719192418100.png)

树中的节点按堆化数组中元素的排序从左至右、从上至下读取。注意：数组在转换为堆后无需排序。排序数组也是有效堆，但堆结构的定义较为宽松，同一组元素可存在多个有效堆。

通过HeapPush函数可将新元素添加到堆，对其他节点进行重新排序，以对堆进行维护：

```c++
HeapArr.HeapPush(4);
// HeapArr == [1,2,4,3,4,5,8,10,7,9,6]
```

![image-20220719192442566](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719192442566.png)

`HeapPop` 和 `HeapPopDiscard` 函数用于**移除堆的顶部节点**。这两个函数的区别在于前者引用元素的类型来**返回顶部元素的副本**，而后者只是简单地移除顶部节点，不进行任何形式的返回。两个函数得出的数组变更一致，重新正确排序其他元素可对堆进行维护：

```c++
int32 TopNode;
HeapArr.HeapPop(TopNode);
// TopNode == 1
// HeapArr == [2,3,4,6,4,5,8,10,7,9]
```

![image-20220719192500184](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719192500184.png)

`HeapRemoveAt` 将删除数组中给定索引处的元素，然后重新排列元素，对堆进行维护：

```c++
HeapArr.HeapRemoveAt(1);
// HeapArr == [2,4,4,6,9,5,8,10,7]
```

![image-20220719193718161](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719193718161.png)

```
Heapify` 调用、其他堆操作或手动将数组操作到堆中之后，才应调用 `HeapPush`、`HeapPop`、`HeapPopDiscard` 和 `HeapRemoveAt
```

此类函数（包括 `Heapify`）都可选择使用**二元谓词决定堆中节点元素的排序**。堆操作默认使用元素类型的 `运算符<` 确定排序。如使用自定义谓词，须在所有堆操作中使用**相同谓词**。

最后，可使用 `HeapTop` 检查堆的顶部节点，无需变更数组：

```c++
int32 Top = HeapArr.HeapTop();
// Top == 2
```

##### 1.1.1.9Slack

可调整数组的大小，因此使用的内存量不同。为避免每次添加元素时重新分配内存，分配器提供的内存通常会超过必要内存，使之后调用 `Add` 时不会因重新分配内存而降低性能。同样，删除元素通常不会释放内存.此操作会使**数组拥有Slack元素**，也就是当前**未使用的有效预分配元素储存槽**。数组中存储的元素量与数组使用分配内存可存储的元素数量间的差值即为数组中的Slack量。

由于默认构建的数组不分配内存，Slack初始为零。**使用 `GetSlack` 函数可找出数组中的Slack量**。通过 `Max` 函数可获取容器重新分配前数组可保存的最大元素数量。`GetSlack` 等同 `Max` 和 `Num` 间的差值：

```c++
TArray<int32> SlackArray;
// SlackArray.GetSlack() == 0
// SlackArray.Num()      == 0
// SlackArray.Max()      == 0

SlackArray.Add(1);
// SlackArray.GetSlack() == 3
// SlackArray.Num()      == 1
// SlackArray.Max()      == 4

SlackArray.Add(2);
SlackArray.Add(3);
SlackArray.Add(4);
SlackArray.Add(5);
// SlackArray.GetSlack() == 17
// SlackArray.Num()      == 5
// SlackArray.Max()      == 22
```

分配器确定重新分配后容器中的Slack量。因此，用户不应认为Slack是常量。

虽然无需管理Slack，但可管理Slack对数组进行优化，以满足需求。例如，如需要向数组添加大约100个新元素，则**可在添加前确保拥有可至少存储100个新元素的Slack**，以便添加新元素时无需分配内存。上文所述的 `Empty` 函数接受可选Slack参数：

```c++
SlackArray.Empty();
// SlackArray.GetSlack() == 0
// SlackArray.Num()      == 0
// SlackArray.Max()      == 0
SlackArray.Empty(3);
// SlackArray.GetSlack() == 3
// SlackArray.Num()      == 0
// SlackArray.Max()      == 3
SlackArray.Add(1);
SlackArray.Add(2);
SlackArray.Add(3);
// SlackArray.GetSlack() == 0
// SlackArray.Num()      == 3
// SlackArray.Max()      == 3
```

`Reset` 函数与Empty函数类似，不同之处是**若当前内存分配已提供请求的Slack，该函数将不释放内存**。但若请求的Slack较大，其将分配更多内存：

**Reset不会使原Slack值变小**

```c++
SlackArray.Reset(0);
// SlackArray.GetSlack() == 3
// SlackArray.Num()      == 0
// SlackArray.Max()      == 3
SlackArray.Reset(10);
// SlackArray.GetSlack() == 10
// SlackArray.Num()      == 0
// SlackArray.Max()      == 10
```

最后，使用 `Shrink` 函数可移除所有Slack。此才做将把内存分配调整为保存当前元素所需的最小内存。`Shrink` 不会对数组中的元素产生影响。

```c++
SlackArray.Add(5);
SlackArray.Add(10);
SlackArray.Add(15);
SlackArray.Add(20);
// SlackArray.GetSlack() == 6
// SlackArray.Num()      == 4
// SlackArray.Max()      == 10
SlackArray.Shrink();
// SlackArray.GetSlack() == 0
// SlackArray.Num()      == 4
// SlackArray.Max()      == 4
```

##### 1.1.1.10原始内存

本质上而言，`TArray` 只是分配内存周围的包装器。直接修改分配的字节和自行创建元素即可将其用作包装器，此操作十分实用。`TArray` 将尽量利用其拥有的信息进行执行，但有时需降低一个等级。

```
TArray
```

`AddUninitialized` 和 `InsertUninitialized` 函数可将未初始化的空间添加到数组。两者工作方式分别与 `Add` 和 `Insert` 函数相同，只是不调用元素类型的构造函数。若要避免调用构造函数，建议使用此类函数。类似以下范例的情况中建议使用此类函数，其中计划用 `Memcpy` 调用完全覆盖结构体：

```c++
int32 SrcInts[] = { 2, 3, 5, 7 };
TArray<int32> UninitInts;
UninitInts.AddUninitialized(4);
FMemory::Memcpy(UninitInts.GetData(), SrcInts, 4*sizeof(int32));
// UninitInts == [2,3,5,7]
```

也可使用此功能保留计划自行构建对象所需内存：

```c++
TArray<FString> UninitStrs;
UninitStrs.Emplace(TEXT("A"));
UninitStrs.Emplace(TEXT("D"));
UninitStrs.InsertUninitialized(1, 2);
new ((void*)(UninitStrs.GetData() + 1)) FString(TEXT("B"));
new ((void*)(UninitStrs.GetData() + 2)) FString(TEXT("C"));
// UninitStrs == ["A","B","C","D"]
//inline void* __CRTDECL operator new(size_t _Size, _Writable_bytes_(_Size) void* _Where)
```

`AddZeroed` 和 `InsertZeroed` 的工作方式相似，不同点是**会将添加/插入的空间字节清零**：

```c++
struct S
{
    S(int32 InInt, void* InPtr, float InFlt)
        :Int(InInt)
        , Ptr(InPtr)
        , Flt(InFlt)
    {
    }
    int32 Int;
    void* Ptr;
    float Flt;
};
TArray<S> SArr;
SArr.AddZeroed();
// SArr == [{ Int:0, Ptr: nullptr, Flt:0.0f }]
```

`SetNumUninitialized` 和 `SetNumZeroed` 函数的工作方式与 `SetNum` 类似，**不同之处在于新数量大于当前数量时，将保留新元素的空间为未初始化或按位归零**。与 `AddUninitialized` 和 `InsertUninitialized` 函数相同，必要时需将新元素正确构建到新空间中：

<font color='green'>//在设置数组元素数量的同时修改元素空间</font>

```c++
SArr.SetNumUninitialized(3);
new ((void*)(SArr.GetData() + 1)) S(5, (void*)0x12345678, 3.14);
new ((void*)(SArr.GetData() + 2)) S(2, (void*)0x87654321, 2.72);
// SArr == [
//   { Int:0, Ptr: nullptr,    Flt:0.0f  },
//   { Int:5, Ptr:0x12345678, Flt:3.14f },
//   { Int:2, Ptr:0x87654321, Flt:2.72f }
// ]

SArr.SetNumZeroed(5);
// SArr == [
//   { Int:0, Ptr: nullptr,    Flt:0.0f  },
//   { Int:5, Ptr:0x12345678, Flt:3.14f },
//   { Int:2, Ptr:0x87654321, Flt:2.72f },
//   { Int:0, Ptr: nullptr,    Flt:0.0f  },
//   { Int:0, Ptr: nullptr,    Flt:0.0f  }
// ]
```

应谨慎使用"Uninitialized"和"Zeroed"函数族。如函数类型包含要构建的成员或未处于有效按位清零状态的成员，可导致数组元素无效和未知行为。此类函数适用于固定的数组类型，例如FMatrix和FVector。

##### 1.1.1.11FMemory::Memory

```c++
static FORCEINLINE void* Memcpy(void* Dest, const void* Src, SIZE_T Count)
	{
		return memcpy( Dest, Src, Count );
	}

void* __cdecl memcpy(
    _Out_writes_bytes_all_(_Size) void* _Dst,
    _In_reads_bytes_(_Size)       void const* _Src,
    _In_                          size_t      _Size
    );



```

###### memory

**不能拷贝含有类类型的任何成员**

<font color='red'>a) memcpy 是按字节拷贝的，与 strlen 搭配使用容易出错</font>

```c++
//下述代码只拷贝了 abcde ，没有拷贝 ‘\0’，strlen遇到 '\0' 就结束
char c1[10] = "abcde";
char c2[10];
memcpy(c2, c1, strlen(c1));

//以下正确拷贝做法
memcpy(c2, c1, sizeof c1);

```

<font color='red'>b) 不能拷贝 string 类型、含指针的结构体、结构体数组、自定义类等</font>

原因：memcpy属于简单的[浅拷贝](https://so.csdn.net/so/search?q=浅拷贝&spm=1001.2101.3001.7020)，即对指针进行拷贝只是仅仅的获取其中指向的数据地址，类中的指针指向的内容无法进行memcpy直接拷贝





##### 1.1.1.12其他

`BulkSerialize` 函数是序列化函数，可用作替代 `运算符<<`，将数组作为原始字节块进行序列化，而非执行逐元素序列化。如使用内置类型或纯数据结构体等浅显元素，可改善性能。

`CountBytes` 和 `GetAllocatedSize` 函数用于估算数组当前内存占用量。`CountBytes` 接受 `FArchive`，可直接调用 `GetAllocatedSize`。此类函数常用于统计报告。

`Swap` 和 `SwapMemory` 函数均接受两个指数并交换此类指数上的元素值。这两个函数相同，不同点是 `Swap` 会对指数执行额外的错误检查，并断言索引是否超出范围。



#### 1.1.2TMap

TMap主要由两个类型定义（一个键类型和一个值类型），以关联对的形式存储在映射中。

继 `TArray` 之后，**虚幻引擎4**（UE4）中最常用的容器是 `TMap`。`TMap` 与 `TSet` 类似，它们的结构均基于对键进行散列运算。但与 `TSet` 不同的是，此容器将数据存储为键值对（`TPair<KeyType, ValueType>`），**只将键用于存储和获取**。

映射有两种类型：`TMap` 和 `TMultiMap`。两者之间的不同点是，`TMap` 中的键是唯一的，而`TMultiMap` 可存储多个相同的键。在 `TMap` 中添加新的键值时，若所用的键与原有的对相同，新对将替换原有的对。在 `TMultiMap` 中，容器可以同时存储新对和原有的对。

##### 1.1.2.1TMap

在 `TMap` 中，键值对被视为映射的元素类型，相当于每一对都是个体对象。在本文中，元素就意味着键值对，而各个组件就被称作元素的键或元素的值。元素类型实际上是 `TPair<KeyType, ElementType>`，但很少需要直接引用 `TPair` 类型。

和 `TArray` 一样，`TMap` 也是**同质容器**，就是说它所有元素的类型都应完全相同。`TMap` 也是值类型，支持通常的复制、赋值和析构函数运算，以及它的元素的强所有权。在映射被销毁时，它的元素都会被销毁。键和值也必须为值类型。

`TMap` 是散列容器，这意味着键类型必须支持 `GetTypeHash` 函数，并提供 `运算符==` 来比较各个键是否等值。稍后将详细介绍散列。

`TMap` 也可使用任选分配器来控制内存分配行为。但不同于 `TArray`，这些是集合分配器，而不是 `FHeapAllocator` 和 `TInlineAllocator` 之类的标准UE4分配器。**集合分配器（`TSetAllocator`类）定义映射应使用的散列桶数量，以及应使用哪个标准UE4分配器来存储散列和元素。**

`KeyFuncs` 是最后一个 `TMap` 模板参数，该参数告知映射如何从元素类型获取键，如何比较两个键是否相等，以及如何对键进行散列计算。这些参数有默认值，它们只会返回对键的引用，使用 `运算符==` 确定相等性，并调用非成员 `GetTypeHash` 函数进行散列计算。如果您的键类型支持这些函数，可使用它作为映射键，不需要提供自定义 `KeyFuncs`。

与 `TArray` 不同的是，内存中 `TMap` 元素的相对排序既不可靠也不稳定，对这些元素进行迭代很可能会使它们返回的顺序和它们添加的顺序有所不同。这些元素也不太可能在内存中连续排列。映射的支持数据结构是稀疏数组，这种数组可有效支持元素之间的空位。当元素从映射中被移除时，稀疏数组中就会出现空位。将新的元素添加到数组可填补这些空位。但是，即便 `TMap` 不会打乱元素来填补空位，指向映射元素的指针仍然可能失效，因为如果存储器被填满，又添加了新的元素，整个存储可能会重新分配。

## 2.蓝图编程

### 1.蓝图函数库和蓝图宏库

相同:

都可以在蓝图中直接调用

不可以被UObeject的直接子类调用

不同:

函数库可以在任何除UObeject类的直接子类里面被调用

宏库只能被对应父类的子类调用(哪怕是UObeject)

### 2.函数和宏

宏只能在类自身上使用，函数只要改变访问控制

宏可以执行时间相关的操作，函数不行

函数可以创建和访问局部变量

### 3.AI

#### 3.1基础需求



创建一个AI_Controller ,  行为树，黑板

![image-20220731212032572](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220731212032572.png)

##### 3.1.1AI_Controller

AI_Controller和PlayerController,只不过换成电脑操控角色

AI_Controller里面只需要做下面的事情

![image-20220731212433502](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220731212433502.png)

给character指定一个AI_Controller

![image-20220731211914314](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220731211914314.png)



##### 3.1.2行为树和黑板

行为树执行AI_Controller要处理的事情

黑板储存行为树执行行为所需的数据

#### 3.2行为树

##### 3.2.1装饰器和服务

右键节点

![image-20220731212829733](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220731212829733.png)

##### 3.2.1.1装饰器

为节点添加判断条件(增加功能)

##### 3.2.1.2服务

增加节点逻辑，节点需要执行服务里的事件

#### 3.3EQS

EQS不会把值设置为空，自己建一个服务，集成一下







### 4.Math

#### 4.1dot(点乘积)

![image-20220731222219858](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220731222219858.png)

![image-20220731222646067](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220731222646067.png)

#### 4.2差积

可用于判断目标位于角色左边还是右边，或者是否在内部（右手定则）

实际为两向量所围成的平面的法线

差积为正则在右，反之在左



#### ![image-20220812195252404](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220812195252404.png)

三维坐标的表示



![image-20220812201654834](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220812201654834.png)





### 遇到的问题

Actor类不可以用Construct object from class ,要用spawnActor ,推测是Actor创建时默认要放到世界里





## 3.资源获取

### 3.1在UE4中使用GIF

插件获取地址:

https://github.com/neil3d/UAnimatedTexture4/releases

功能:

可直接把GIF文件拖入，使用方法和Texture2D一样

## 4.虚幻BUG(4.26)

### 1.事件图表拉远，线会扭曲

### 2.多次切换输入法会导致注释不能继续用中文输入法

没得救

### 3.右键菜单闪一下就没了

下载NIVIDA显卡补丁

### 4.EQS写不了注释

## 5.UE快捷键

运行时:    按''(引号)打开AI观察器