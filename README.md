# RBQscript

FF0Script 的改进版。效率大幅提升。[原 FF0Script 链接](https://github.com/WarfarinBloodanger/ff0-script)

其实这就是 FF0Script 4.0+ 的版本。

请注意：**与 4.0 之前的版本不兼容**。

## Part 1. RBQScript 启动方式：

编译器兼解释器：`fvm.exe`。

总命令格式：
```
fvm [<byte-file>] | [[-i <input-file>] [-o <output-file>] [<options>] [<switches>]] 
```

格式一：`fvm [<byte-file>]`，将会执行 `<byte-file>` 中的字节码。如果检测到 `<bytecode-file>` 没有拓展名，将会自动加上一个 `.rbq` 作为文件名，否则直接使用它作为文件名。

格式二：`fvm -i <input-file> -o <output-file> [<options>] {<switches>}`。

将会从 `<input-file>` 里面读入源代码文件，以 `<options>` 和 `<switches>` 为附加条件进行编译后输出字节码文件到 `<output-file>`。

如果有输入文件没有输出文件，将会默认将输入文件名的后缀改为 `.rbq`。

options 格式：`-{<op-symbol>}`。op-symbol 表示执行不同操作，包含如下几个选项：

- `c` 编译；
- `r` 运行，注意：必须要包含 `c` 选项才能运行；
- `d` 调试，会在编译结束之后输出所有函数的字节码和符号表。

如：`-crd` 表示编译、运行加调试；`-cr` 表示编译运行；而 `-c` 表示只编译不运行。

switches 包含如下几个选项：

- `-xnobig`：禁止程序自动将过大的实数转化为高精度；
- `-xfull`：禁止程序使用简单版字节码；
- `-xnocheck`：禁用访问检查器，允许访问内部变量。

另有一个调试用工具 `rbqup.exe` 调用格式如下：

`rbqup <bytecode-file> [en|ch|op]`

这将会把 `<bytecode-file>` 里面的字节码进行标记后输出到控制台并着色。第三个参数，en 表示使用英文标注，ch 表示使用中文标注，op 表示使用助记符标注，如果没有第三个参数则默认为 en 模式。

如果检测到 `<bytecode-file>` 没有拓展名，将会自动加上一个 `.rbq` 作为文件名，否则直接使用它作为文件名。

## Part 2. RBQScript 语法

RBQScript 的语法非常简单，有 JavaScript 开发经验的程序员将会更容易的上手 RBQScript。

- 1. 变量、赋值、表达式；

**普通**变量无需声明，可以直接赋值：`a = 1`。注意，所有语句**一行一条**。

RBQScript 内置以下几种值类型：
- 实数，为 IEEE 规范下 C/C++/Java 中的 double 类型；
- 高精度整数，没有小数部分；
- 字符串；
- 函数（闭包）；
- 数组；
- null，表示空；
- 范围，表示一个下标的范围。

变量可以定义成局部变量。格式是在变量前面加上 `local`。
```
a = 0
function f() {
    local a = 1
    print(a)
}
f()
print(a)
```
如果定义了一个局部变量但不赋初值，那么这个变量将被自动初始化成 null。另外：全局变量不会自动初始化，均为 undefined 未定义类型。

注意：这里的局部变量是以**函数**为作用域的，不存在块作用域。也就是说如下代码能够正常运行：
```
if(true) {
	a = 10
}
print(a)
```

实数可以有两种表示方式：十进制表示和十六进制表示，如：

```
num = 1241
x = 0xff0
```

注意：当计算过程中，实数的绝对值超过了 0x7FFFFFFF 时，将会被自动转换为高精度（损失小数部分精度）进行运算，**可以使用** `-xnobig` **禁止该行为**。

字符串字面量为双引号或单引号括起来的字符序列，如：
```
s = "hello"
name = 'rbq'
```

可以用反斜杠 `\\` 进行转义。另外有特殊的一个转义字符：`\\u0x????`，将会把十六进制的 `???` 转化为 Unicode 字符。例如：`"\\'\\uC4E3"` 是字符串 `'你`。

数组字面量用大括号包围的逗号分隔的一个或多个值组成，如 `a = {1, 2, "hello"}` 。

数组自带关联数组功能。以下代码可以正常运行：

```
a = {}
a["test"] = 1
print(a["test"])
```

另外支持点语法：`a["test"]` 完全等价于 `a.test`。

对于范围类型，字面量格式为 `((L) : (R))`。表示的区间为 $[L, R)$。可用于对字符串、数组进行切片操作。如：

- `"abcdef"[2:5]` 等于 `"cde"`；
- `{1, 2, 3, 4, 3, 2}[0:4]` 等于 `{1， 2， 3， 4}`；

**请注意， : 运算符的优先级很高。** 如果使用 `(1+2 : 3+4)`，将会被识别成 `1 + (2 : 3) + 4` 而产生错误，请加上括号：`((1 + 2) : (3 + 4))`。

- 2. 控制流程语句；

- if 语句：
```
if (<condition>) {
    ...
}
else {
    ...
}
```

允许 if-else-if 级联语句。

- while 语句：
```
while(<condition>) {
    ...
}
```

- for 语句：
```
for(<name>, <first-value>, <condition>, <step-value>) {
    ...
}
```

这等价于 JavaScript 中的：`for(var <name> = <first-value>; <condition>; <name> = <name> + <step-value>)`。 

例如：`for(i, 1, i <= 10, 1)` 等价于 JavaScript 中的 `for(var i = 1; i <= 10; i = i + 1)` 。

- foreach 语句：
```
foreach(<name>, <iterable-value>) {

}
```

这将会迭代 `<iterable-value>` 中所有的值，**关联数组中的字符串下标不会被迭代**。可迭代的值类型有：字符串、数组、范围。

例如：
```
foreach(i, {1, 2, 4, 4, 1}) print(i)
```

特殊：你可以使用 `<name>__iterator` 来访问到迭代器的下标，如：

```
foreach(i, "abcdef") print(i, i__iterator)
```

将会在输出给定字符串的每个字符的同时在后面跟上字符所在的下标（从 $0$ 到 $6$）。

- break / continue 语句：

将会跳出循环 / 跳回到循环的头部重新开始。

- 3. 函数定义；

基本格式为：
```
function <name>([<arg1> [, <arg2>, <arg3>, ...]]) {

}
```

这将定义一个名为 `<name>` 的函数。请注意：函数都是**当前作用域的局部变量**（可以是全局变量）。

RBQScript 还支持第二种函数定义：Lambda 匿名函数。格式为：`(lambda {<arg-list>} {<expression>})`。

比如：`(lambda {x, y} {x * x + y})` 将会得到一个二元函数，计算 `x * x + y`。

用 `return` 从函数中返回值。

- 4. 其他；

使用 include 语句可以从其他文件中导入**源代码**，必须包含**完整拓展名**。

- 5. 内置函数：

- `print` 输出函数，参数个数不固定，将会把所有参数转化为字符串后**用空格分隔** 输出在同一行；

- `len` 函数接受一个参数，取范围、字符串、数组的长度，如 `len("hello")` 得到 $5$；
- `sub` 函数接受两个参数，取范围中给定下标的值。如 `sub((0:4), 1)` 得到 $2$；
- `time` 和 `clock` 函数不接受参数，分别获得当前时间和程序运行时间；
- `system` 函数接受一个字符串参数，返回执行系统命令的结果；
- `memset` 函数接受参数范围在 $1$ 到 $4$ 之间。第一个参数必须是数组类型；
1. 当只有一个参数时，将会把这个数组全部填充为 $0$（包含关联数组内的所有内容）；
2. 当有两个参数时，将会把这个数组全部填充为第二个参数的值；
3. 当有三个参数时，将会把这个数组从 $0$ 到第三个参数之间的下标全部填充为第二个参数。如果第三个参数是字符串，那么所有数字下标都会被填充成第二个参数；
4. 当有四个参数时，将会把这个数组从第三个参数到第四个参数之间的下标全部填充为第二个参数。
- 数学函数：使用 `math_xxx` 调用，都只接受一个实数参数。其中 `xxx` 可以替换为对应功能：这些功能包含：`sin/cos/tan/asin/acos/atan/sqrt/floor/ceil/log/log10/abs`。如 `math_sin` 计算正弦值。

- read 系列函数：不接受参数，`read_number`、`read_string`、`read_line` 分别读入一个实数、字符串（空白符分割）和一整行（换行符分割）并返回；

- file 系列函数：
1. `file_open` 的第一个参数是文件名，如果有第二个参数，则将其作为打开文件的方式，返回一个整数表示文件的句柄；
2. `file_close` 接受一个整数参数，将会关闭该句柄指向的文件；
3. `file_eof` 接受一个整数参数，如果到达文件尾部则返回 $1$，否则返回 $0$；
4. `file_read_xxx` 系列函数，和 read 系列参数类似的功能，接受一个参数作为文件句柄，从文件中按要求读取给定数据并返回；
5. `file_write_xxx` 系列参数，功能和 read 相反，写入到给定的文件句柄当中；

- `str2ascii` 接受一个字符串参数，如果该字符串长度为 $1$，则返回该字符串的字符的 ASCII 码；否则返回一个数组，依次存储字符串各个字符的 ASCII；

- `ascii2str` 可接受多个参数。
1. 如果第一个参数是数组，那么将该数组中所有的实数值转换为 ASCII 字符后连在一起，作为字符串返回；
2. 否则，将所有参数依次转换为 ASCII 字符后连在一起，作为字符串返回；
3. 上面两条中对于无法转换的非数字值，将会以 NaN 代替。

- `num` 接受一个字符串参数，将其转换为一个实数后返回；

- `big` 接受一个字符串参数，将其转换为一个高精度整数后返回；

### update 6th Dec 2021:

加入了对象系统，语法如下：

```
struct ClassName [extends SuperName] (field1, field2, field3...)
```
这将会定义一个叫做 ClassName 的结构体。目前 RBQScript **不支持成员函数**，因此称作是 struct，不是 class。

构造对象的方法：使用类名作为函数名调用，**从父类开始，按声明的字段顺序写入参数**。如：

```
struct Class(a, b)
struct Class2 extends Class(c, d)
obj = Class("hello", 5 + 9)
obj2 = Class2("world", 3 + 1, 5.433, {1, 2, 3})
print("obj: a is", obj.a, ", b is", obj.b)
print("obj2: a is", obj2.a, ", b is", obj2.b, ", c is", obj2.c, ", d[1] is", obj2.d[1])
```

输出：
```
obj: a is hello , b is 14
obj2: a is world , b is 4 , c is 5.433 , d[1] is 2
```

**推荐自己定义构造函数，如下：**

```
struct Class(a, b)
function make_Class(a) {
    return Class(a, a * a)
}
```

Object 是所有类的基类，在定义的时候不需要显示地写上 `extends Object`，会自动继承。

特殊字段：`obj.class_name` 返回该对象的类的名称，`obj.super_name` 返回该对象的基类的名称。

```
struct Person(name)
struct OIer extends Person(age)
oier = OIer("ff0", 343)
print(oier.name, oier.age, oier.class_name, oier.super_name)
```

输出：

```
ff0 343 OIer Person
```
