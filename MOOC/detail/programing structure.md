## 全局变量 & 宏 & 大程序结构

<br>

### 全局变量
#### 认识 全局变量
- 定义在函数外的变量就是全局变量
- 全局变量具有全局的生存期和作用域
   - 它们与任何函数无关
   - 任何函数（定义在全局变量后的的函数）内部都可以使用它们

例如：
```c
int f(void);

int gAll = 12;

int main(void){
	
	//__func__ 可以打印出当前函数的函数名，下划线一边是两个
	printf("in %s gAll = %d\n", __func__, gAll);
	//全局变量可以直接使用，不需要再声明
	f();
	printf("again in %s gAll = %d\n", __func__, gAll);
	//函数内对全局变量值的改变在 main函数中依然存在
	return 0;
}

int f(void) {
	
	printf("in %s gAll = %d\n", __func__, gAll);
	gAll += 2;
	printf("again in %s gAll = %d\n", __func__, gAll);

	return gAll;
}
```
输出：
```c
in main gAll = 12
in f gAll = 12
again in f gAll = 14
again in main gAll = 14
```
###### 全局变量的初始化
- 没有初始化的全局变量**默认值为 0**
    - 指针默认为 **NULL**
- 只能用**编译时刻已知**[^1]的值来初始化全局变量
- 全局变量的初始化发生在main函数之前

注释1：
```c
int gAll = 12;
int g = gAll;//报错

int main(void) {

	return 0;
}
```
下面这段代码在某些编译器（dev c++）上是可以编译的，但是在 vs 上是不能编译的
```c
const int gAll = 12;
int g = gAll;

int main(void) {

	return 0;
}
```
**但是，这种方式是不推荐的**

###### 被隐藏的全局变量
- 如果函数内部存在与全局变量同名的变量，则全局变量被隐藏。

```c
int f(void);

int gAll = 12;

int main(void) {

	printf("in %s gAll = %d\n", __func__, gAll);

	f();

	printf("again in %s gAll = %d\n", __func__, gAll);

	return 0;
}

int f(void) {
	
	int gAll = 2;//仅在这个范围内适用
	printf("in %s gAll = %d\n", __func__, gAll);
	gAll += 2;
	printf("again in %s gAll = %d\n", __func__, gAll);

	return gAll;
}
```
输出：
```c
in main gAll = 12
in f gAll = 2
again in f gAll = 4
again in main gAll = 12
```
即使 gAll 在 main 函数中被覆盖，f 函数中的 gAll 也是不会被该改变的

为什么会这样？自己思考一下。

#### 静态本地变量
- 在本地变量定义时加上 static 修饰符就成为静态本地变量
- 当离开函数的生存期后，静态本地变量会继续存在并保持其值
- 静态本地变量的初始化只会在第一次进入这个函数时进行，以后进入函数时会保持上次离开时的值。

例：
**不用static**的情况
```c
int f(void);

int main(void) {

	f();
	f();
	f();

	return 0;
}

int f(void) {

	int All = 1;
	printf("in %s All = %d\n", __func__, All);
	All += 2;
	printf("again in %s All = %d\n", __func__, All);

	return All;
}
```
输出：
```c
in f All = 1
again in f All = 3
in f All = 1
again in f All = 3
in f All = 1
again in f All = 3
```

**使用static**：
```c
int f(void);

int main(void) {

	f();
	f();
	f();

	return 0;
}

int f(void) {

	static int All = 1;//只添加 static
	printf("in %s All = %d\n", __func__, All);
	All += 2;
	printf("again in %s All = %d\n", __func__, All);

	return All;
}
```
输出：
```c
in f All = 1
again in f All = 3
in f All = 3
again in f All = 5
in f All = 5
again in f All = 7
```
**看看地址**
```c
int f(void);

int gAll = 12;

int main(void) {

	printf("1 st\n");
	f();
	printf("2 nd\n");
	f();

	return 0;
}

int f(void) {
	
	int a = 0;
	int b = 0;
	static int All = 1;

	printf("&All :  %p\n", &All);
	printf("&gAll:  %p\n", &gAll);
	printf("&a :    %p\n", &a);
	printf("&b :    %p\n", &b);

	return All;
}
```
输出：
```c
1 st
&All :  00007FF6A9ECC054
&gAll:  00007FF6A9ECC050
&a :    000000E8815CF8B4
&b :    000000E8815CF8D4
2 nd
&All :  00007FF6A9ECC054
&gAll:  00007FF6A9ECC050
&a :    000000E8815CF8B4
&b :    000000E8815CF8D4
```
全局变量 gAll 与 静态局部变量 All 在内存中相邻

**总结**
- 静态本地变量实际上是特殊的全局变量
- 它们位于相同的内存区域
- 静态本地变量具有全局的生存期，函数内的局部作用域

#### 返回指针的函数 
请同学们先看一下下面这个程序：

```c
int* f(void);
void g(void);

int main(void) {

	int* p = f();
	printf("*p = %d\n", *p);
	g();
	printf("*p = %d\n", *p);

	return 0;
}

int* f(void) {

	int i = 12;

	return &i;
}
void g(void) {

	int k = 24;
	
	printf("k = %d\n", k);

	return k;
}
```
输出：
```c
*p = 12
 k = 24
*p = 24
```
**i 和 k 的内存其实是同一块空间**

**总结**

- 返回 **本地变量** 的地址是危险的
- 返回 **全局变量** 或 **静态局部变量** 的地址是安全的
- 返回函数内 malloc 的内存是安全的，但是容易造成问题
- 最好的做法是**返回传入的指针**

说了这么多，总结一句话

###### 尽量避免使用 全局变量 和 静态本地变量

<div align = "center">

![？？？](https://img-blog.csdnimg.cn/20200211000341388.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>？？？</p>

</div>

为什么这里就不深讲了，有兴趣的朋友可以下来自己查查。

***
### 编译预处理 与 宏

#### 编译预处理指令
- `#` 开头的是编译预处理指令
- 它们不是 C语言的一部分，但是 C语言离不开他们
- `#define` 用来定义一个宏

#### define 关键字
回想我们刚学 double 的时候，是不是计算过圆的面积。当时我们可能是这样写的：
```c
#include<stdio.h>

const double PI = 3.14159;

int main(void) {

	printf("%f\n", 2 * PI * 3.0);
	return 0;
}
```
现在我们用 宏 就不需要用 const 修饰的全局变量了，我们也说过，全局变量最好不用。
```c
#include<stdio.h>

#define PI 3.14159
//注意：不写分号 不写等于号

int main(void) {

	printf("%f\n", 2 * PI * 3.0);
	return 0;
}
```
现在，我们打开我们的虚拟机，进入 Linux 系统。

<div align = "center">

![](https://img-blog.csdnimg.cn/20200211003555492.png)

<p>1.创建一个 c文件</p>

</div>

<div align = "center">

![](https://img-blog.csdnimg.cn/20200211005048362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>2.写一个简单的带宏的 c程序</p>

</div>


<div align = "center">

![](https://img-blog.csdnimg.cn/20200211003635685.png)

<p>3.这时后我们成功创建了一个 c文件</p>

</div>


<div align = "center">

![](https://img-blog.csdnimg.cn/20200211003754529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>4.编译c文件，并保留中间文件</p>

</div>


现在多出来了 4 个文件，蓝色的是文件夹，我们不去管它，绿色的是可执行文件，类似 windows 的 .exe 文件
现在我们主要关注这 3 个中间文件

<div align = "center">

![](https://img-blog.csdnimg.cn/20200211003955771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>文件详细情况</p>

</div>

一个 c文件编译的过程文件变化是这样的：

>`.c `（处理编译预处理指令）-> `.i `（产生汇编代码）-> `.s`（汇编生成目标文件） -> `.o`（链接等） -> `a.out `
>

可以看到 .i 文件时很大

<div align = "center">

![](https://img-blog.csdnimg.cn/20200211005244118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>我们看看 .i 文件的 结尾部分</p>

</div>


<div align = "center">

![](https://img-blog.csdnimg.cn/20200211005330453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>对比 .c 文件</p>

</div>

**我们发现程序中的宏 PI 被换成了它所表示的 数字**

这种替换是**简单的文本替换**，我们再试试其他的替换方式：

<div align = "center">

![](https://img-blog.csdnimg.cn/20200211010337202.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>替换字符串</p>

</div>

我们再试试这样，定义宏的时候 不带双引号：

<div align = "center">

![](https://img-blog.csdnimg.cn/20200211010437762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>FRORMAT 同样被替换了</p>

</div>

<div align = "center">

![](https://img-blog.csdnimg.cn/20200211010618834.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>编译器给了 warning</p>

</div>


<div align = "center">

![](https://img-blog.csdnimg.cn/20200211010833810.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>FORMAT并没有被替换</p>

</div>

因此可知，**被 `" "`扩起来的字符串 宏 是不会替换的**

###### 总结
- 格式： `#define <名字> <值>`
- 注意结尾没有分号，因为不是 C 的语句
- 名字必须是一个单词，值可以是任何（注意字符串替换定义时需要带引号）
- 在 **C语言的编译器开始编译之前**，编译预处理程序（cpp）会把程序中的宏的名字替换为值
- linux/unix 
    - 编译并保留中间文件指令：`gcc --save-temps`
    - 查看文件结尾：`tail` 

###### 宏
- 如果在一个宏的值中有其他宏的名字，这些宏也是会被替换的
- 如果一个宏的值超过一行，最后一行之前的行末需要加 \ 
- 宏的值后面出现的注释不会被当作宏的值的一部分

###### 没有值的宏
- `#define _DEBUG`
- ` #define _CRT_SECURE_NO_WARNINGS` 用 VS 的应该都知道这个吧，加上这个你就可以直接用`scanf`而不是`scanf_s`了
>这类宏是用来做条件编译的，后面有其他编译预处理指令来检查这个宏是否已经被定义过了。
>比如有这个宏执行这部分代码，没有则执行另外一部分

###### 预定义的宏
- `__LINE__`
- `__FILE__`
- `__DATE__`
- `__TIME__`
- `__STDC__`

我们来试着用一下：
```c
int main(void) {

	printf("%s : %d\n", __FILE__, __LINE__);
	printf("%s %s\n", __DATE__, __TIME__);

	return 0;
}
```
输出：
```c
D:\vscode\练习\12-31\Project1\oj.c : 174
Feb 11 2020 04:12:57
```
值得注意的是，`__LINE__`表示的是它自己所在的行数

你们在熟睡，而我还在给你们写教学，关注我/点个赞/转发 不过分吧~

![](https://img-blog.csdnimg.cn/2020021104154880.gif)

#### 带参数的宏

- `#define cube(x) ( (x) * (x) * (x) )`

例如：
```c
#define cube(x) ((x) * (x) * (x))

int main(void) {

	printf("%d\n", cube(5));
	return 0;
}
```
输出：
```c
125
```
**容易犯的错误**

一下这两种写法在程序中会不会有问题？
- `#define ERROR(1x) (x * 57)`
- `#define ERROR2(x) (x) * 57`

思考一下这个程序会的到你想要的结果吗？
```c
#define ERROR1(x) (x * 57)
#define ERROR2(x) (x) * 57

int main(void) {
	
	printf("%d\n", ERROR(1 + 2));
	printf("%d\n", 300 / ERROR(1));

	return 0;
}
```
输出：
```c
115
17100
```
为什么会这样呢？我们不妨来看一下，`.i`文件内部：

<div align = "center">

![](https://img-blog.csdnimg.cn/20200211043539513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)

<p>注意运算顺序</p>

</div>

**定义带参数的宏的原则**
- 一切都要有括号
   - 整个值有括号
   - 每个参数都有括号

所以，上面错误的例子的正确的写法就是：
`#define ERROR  ( (x) * 57 )`

**带参数的宏的更多用法**：
- `#define MIN(a, b)  ((a) > (b) ? (b) : (a))`

**定义宏切记不要加分号**

错误示范：
```c
#define PRETTY_PRINT(msg) printf(msg);

int main(void) {

	int n = 0;

	printf("Input an number\n");
	scanf("%d", &n);

	if (n < 10)
		PRETTY_PRINT("less than 10\n");
	else
		PRETTY_PRINT("more than 10\n");

	return 0;
}
```
VS 会报错 ：没有匹配 if 的非法 else，为什么呢？

因为如果你在宏后面加了 `;`,你又在 if 内的语句后加了`;`
这样在`.i`的阶段，if 后的语句有了两个 `;`,即：
`PRETTY_PRINT("less than 10\n");;`
第二个`;`表示 一个空语句，这样 else 前面就没有对象可以匹配了

###### 总结
- **#开头的预处理指令并不是 C语言独有的内容**
- **宏的参数时没有类型的**
- 大型程序中宏的使用很常见
- 宏可以很复杂，可以产生函数
   - 使用运算符 `#` 和 `##`
- 部分宏会被 `inline`函数取代
- 中西方差异（国人少用）

***
Quiz：
请看下面的代码片段，判断这段程序会输出什么？
```c

#define TOUPPER(c) ('a' <= (c) && (c) <= 'z' ? (c) - 'a' + 'A' : (c))

	int i = 0;
	char s[1000];
	
	strcpy(s, "abcd");
	
	putchar(TOUPPER(s[++i]));

```
A: B 
B: C
C: D
D: E

*这道题是需要都脑子的呦!*
公众号后台回复：**0211 1** 查看答案和解析
***
### 大程序结构
#### 多个源代码文件
###### 多个源文件`.c` 
**引入**
回想我们学习的过程，开始是 main（）里的代码太长了，我们学习了函数，将其分开
现在如果 一个源文件太长了，我们就可以将其分成几个源文件

**怎么让多个源文件联系起来？**
在编译器上创建一个项目，将你想操作的 .c 文件放到同一个项目中

#### 头文件 `.h`
###### **`" " ` 还是 `< >` ?**
- `#include`有两种形式来指出要插入的文件
   - `" "`要求编译器首先在当前目录（.c 文件所在目录）寻找这个文件；如果没有，再去编译器指定的目录寻找。**自己的头文件用**
   - `< >`让编译器只在指定位置寻找 **。系统的头文件用**
- 编译器知道自己的标准库的头文件在哪里
- 环境变量 和 编译器命令行参数也可以指定寻找头文件的目录

###### **`#include`的误区**
- `#include`不是用来引入库的
- `stdio.h ` 中只有函数的声明，函数的定义在其他的地方
- C语言编译器默认会引入所有标准库
- `#include<stdio.h>`的作用其实就是将 这个头文件的所有内容 插入到这个文件中来。目的是让编译器知道你使用的函数时所给的参数是否正确。（类似函数的声明）

**为什么不引用 `stdlib.h` 依然可以使用 `malloc ` ?**
这时因为在你调用函数前没有声明函数（引入头文件），编译器回去猜测 参数 和 函数返回类型都为 `int`型
恰好 `malloc` 的参数 `size_t` 是 `long int` ，返回值是个指针，也可以看作是 16进制的 整型。

*[为什么？可以参考我的另一篇文章，点击跳转](https://mp.weixin.qq.com/s/JEalmGOwNXp9IM0W7B7YJw
)*

###### 头文件 
- 使用和定义函数的地方都应该包含这个头文件
- 将 函数声明 全局变量 放入 `.h`文件

###### 不对外公开的 函数&变量
函数&全局变量前加上 `static`就使得这个 函数/变量 只能在当前文件中被使用

### 声明
###### extern
当一个c 文件想调用另一个 c文件中定义的全局变量时
需要在头文件中加上 `extern <类型> <变量名>` 来声明这个变量

例如：
```c
<1.c>
int gAll = 12;

<2.c>
printf("%d\n", gAll);

<1.h>
extern int gAll;
```
**声明不产生代码**

###### 避免重复声明
请看下例：
```c
<1.h>
int a ;

<2.h>
#include"1.h"

<3.h>
#include"1.h" 
//相当于
//int a 

#include"2.h" 
//相当于
//#include"1.h"
//相当于
//int a ;

//可以看到， int a 被重定义的
```
如何避免上述这种重定义情况？

###### 条件编译和宏
- 运用条件编译和宏，保证这个头文件在一个编译单元中只会被 include 一次
- `#pragma once`也能起到相同作用，但不是所有的编译器都支持
```c
#ifdef _MAIN_H_//先看 这个宏是否定义过，是：继续 不是：跳过这个结构。跳过的意思是编译时将不再向 .i 文件中插入这段代码
#define _MAIN_H_//没有定义，则定义

#endif
```
这就是我们前面说的预定义的宏的一种使用方法。
应用这种方法我们再看上例

```c
<1.h>
#ifdef _FIRST_H_
#define _FIRST_H_

int a ;

#endif
<2.h>
#include"1.h"

<3.h>
#include"1.h" 
//相当于：
//#ifdef _FIRST_H
//#define _FIRST_H
//int a ;
//#endif
#include"2.h" 
//相当于：
//#include"1.h"
//这时，_FIRST_H 已经被定义，则跳过

```

***
以上就是本次的内容，欢迎各位指出我的错误，谢谢！

这篇教程的Github地址：

https://github.com/hairrrrr/C-CrashCourse

Github 大概在 微信公众号更新 1 ~ 2 天后更新，欢迎加入我，让我们一起完成全世界最全的 C 语言教学！

关注我的微信公众号
获取第一时间更新 👇👇👇

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200211164411282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0OTU0MDEw,size_16,color_FFFFFF,t_70)
