size 返回的数据类型为 ==string::size_type ==，是一个无符号整数。

当把string对象和字符字面值及字符串字面值混在一条语句中使用时，必须确保每个加法运算符（+）的两侧的运算对象至少有一个是string：
```C++
string s5 = "hello" + ","; // 错误：两个运算对象都不是string
```

![81eG9sXxYr3pJCZ](https://s2.loli.net/2022/01/18/81eG9sXxYr3pJCZ.png)
```C++
string s5 = "hiya";      // 拷贝初始化
string s6("hiya");       // 直接初始化
string s7(10, ′c′);      // 直接初始化，s7的内容是cccccccccc
```
如果初始化的值要用到多个，一般只能用直接初始化，例如s4

**cctype头文件中的函数**
![WdRylgsfm7zhFO8](https://s2.loli.net/2022/01/18/WdRylgsfm7zhFO8.png)

string对象的下标必须大于等于0而小于s.size()。


vector 可以包含绝大多数类型，但是==不可以包含引用==，因为==引用不是对象==。
![pPCg7rzK6tZYNf9](https://s2.loli.net/2022/01/18/pPCg7rzK6tZYNf9.png)

vector 可以只提供元素的数量，而不提供初始值
```C++
vector<int> ivec(10);        // 10个元素，每个都初始化为0
vector<string> svec(10);     // 10个元素，每个都是空string对象
```

 vector 初始化时，花括号中的值必须与元素类型相同。
 
 vector==直接初始化==适用的情况：
 * 初始值已知且数量较少
 * 初始值是另一个vector对象的副本
 * 所有元素的初始值都一样

![uAYR9kb2Jrg1zP4](https://s2.loli.net/2022/01/18/uAYR9kb2Jrg1zP4.png)

`size` 返回的类型是 `vector<int>::size_type`

如果`vector`对象是一个空的，则不能通过下标访问元素，只能通过`push_back`进行添加元素。

**迭代器**
![843UZkL2W1sAjxa](https://s2.loli.net/2022/01/18/843UZkL2W1sAjxa.png)
`end`返回的迭代器并不实际指向某个元素，所以不能对其进行递增或解引用的操作。

迭代器的类型：`iterator`、`const_iterator`
`cbegin`、`cend`返回的类型是`const_iterator`

箭头运算符：`it->mem`作用与`(*it).mem`相同，都是解引用。

**vector的限制**
* 不能在 for 循环中向 vector 添加元素。
* 任何一种可能改变 vector 容量的操作，都会使该 vector 对象的迭代器失效。

![at91KS8vMOqwcC2](https://s2.loli.net/2022/01/18/at91KS8vMOqwcC2.png)

迭代器距离的类型为`difference_type`，是一个带符号整型数。

**数组**
数组的大小确定不变，不能向数组中随意添加元素。
数组的元素应为对象，==不可以存在引用的数组==。
定义数组的时候必须指定数组的类型，不允许用auto关键字由初始值的列表推断类型。

使用字符串字面值初始化数组时，因为字符串字面值==结尾有一个空字符==，所以也会被拷贝到数组当中去。
```C++
char a1[] = {′C′, ′+′, ′+′};         // 列表初始化，没有空字符
char a2[] = {′C′, ′+′, ′+′, ′\0′};   // 列表初始化，含有显式的空字符      
char a3[] = "C++";                    // 自动添加表示字符串结束的空字符  
const char a4[6] = "Daniel";         // 错误：没有空间可存放空字符！
```

不能将数组的内容拷贝给其他数组作为其初始值，也不能用数组为其他数组赋值.

```C++
int ＊ptrs[10];               // ptrs是含有10个整型指针的数组
int &refs[10] = /＊ ?＊/;      // 错误：不存在引用的数组        
int (＊Parray)[10] = &arr;    // Parray指向一个含有10个整数的数组        
int (&arrRef)[10] = arr;     // arrRef引用一个含有10个整数的数组
int ＊(&arry)[10] = ptrs; // arry是数组的引用，该数组含有10个指针
```

数组下标的类型为`size_t`，定义于`cstddef`头文件中，是一个无符号类型。

当使用数组作为一个==auto==变量的初始值时，推断得到的类型是指针而非数组。
当使用==decltype==关键字时，decltype(ia)返回的类型是由10个整数构成的数组。

**数组的首元素与尾后元素**
```C++
int ia[] = {0,1,2,3,4,5,6,7,8,9}; // ia是一个含有10个整数的数组
int ＊beg = begin(ia);        // 指向ia首元素的指针        
int ＊last = end(ia);         // 指向arr尾元素的下一位置的指针
```
指向首元素和尾后元素的指针相减之后，类型为`ptrdiff_t`定义于头文件`cstddef`中。