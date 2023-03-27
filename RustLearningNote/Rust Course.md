# 寻找牛刀，以便小试

## 安装Rust环境

安装rust命令：

```shell
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
#Rustup metadata and toolchains will be installed into the Rustup
#home directory, located at:

#  /home/yuwei/.rustup

#This can be modified with the RUSTUP_HOME environment variable.

#The Cargo home directory is located at:

#  /home/yuwei/.cargo

#This can be modified with the CARGO_HOME environment variable.

#The cargo, rustc, rustup and other commands will be added to
#Cargo's bin directory, located at:

#  /home/yuwei/.cargo/bin

#This path will then be added to your PATH environment variable by
#modifying the profile files located at:

#  /home/yuwei/.profile
#  /home/yuwei/.bashrc
```

让浏览器打开本地文档

```shell
rustup doc
```

## 认识Cargo

`cargo` 提供了一系列的工具，从项目的建立、构建到测试、运行直至部署，为 Rust 项目的管理提供尽可能完整的手段。同时，与 Rust 语言及其编译器 `rustc` 紧密结合。

```shell
cargo new world_hello
cd ./world_hello
```

使用`cargo new`命令创建一个项目，项目名位world_hello，该项目的结构和配置文件都是由cargo生成，**意味着项目被cargo管理**

Rust项目主要分为两个类型：`bin`和`lib`，前者是一个**可运行的项目**，后者是一个**依赖库项目**。**默认创建的是bin项目。**

通过tree命令查看项目的结构

```shell
$ tree
.
├── .git
├── .gitignore
├── Cargo.toml
└── src
    └── main.rs
```

**运行rust项目：**

```shell
cargo run#前提是在项目的文件夹中
```

该命令首先对项目进行编译，然后再运行，相当于运行了两个命令

```shell
cargo build
./target/debug/world_hello
#target文件为编译后生成的文件夹，debug则代表运行是debug模式
```

在`debug`模式下，代码的**编译速度非常快**，但是**运行速度变慢**，`debug`模式下，**Rust编译器不会做任何的优化**，只为了尽快的编译完成。

可以通过增加`--release`来避免使用`debug`模式

```shell
cargo run --release
cargo build --release
#这时就可以使用./target/release/world_hello来执行了
```

`cargo check`**可以快速检查代码是否能编译通过**

```shell
cargo check
    Checking world_hello v0.1.0 (/Users/sunfei/development/rust/world_hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s
```

## Cargo.toml和Cargo.lock

cargo的活动都基于这两个文件

- Cargo.html是cargo特有的**项目数据描述文件**，存储了项目的所有元配置信息

  ```toml
  [package]
  name = "world_hello"
  version = "0.1.0"
  edition = "2021"
  
  # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
  
  [dependencies]
  ```

  package中记录了项目的描述信息。

  这里的`name`字段定义了项目名称，`version`定义了当前版本，新项目默认的是`0.1.0`，`edition`字段定义了使用的Rust大版本。

  **主要通过各种依赖段落来描述该项目的各种依赖项**

  - 基于Rust官方仓库`crates.io`，通过版本说明来描述
  - 基于项目源代码的git仓库地址，通过URL来描述
  - 基于本地项目的绝对路径或者相对路径，通过类Unix模式的路径来描述

  三种形式的具体写法如下：

  ```toml
  [dependencies]
  rand = "0.3"
  hammer = { version = "0.5.0"}
  color = { git = "https://github.com/bjz/color-rs" }
  geometry = { path = "crates/geometry" }
  ```

- Cargo.lock是cargo根据同一项目的toml文件生成的**项目依赖详细清单**，一般不需要修改

## Rust语言初印象

```rust
fn greet_world() {
    let southern_germany = "Grüß Gott!";
    let chinese = "世界，你好";
    let english = "World, hello";
    let regions = [southern_germany, chinese, english];
    for region in regions.iter() {
        println!("{}", &region);
    }
}
fn main() {
    greet_world();
}
```

Rust原生支持`UTF-8`编码的字符串。println后面的`!`，为Rust中的`宏`操作符。

对于`println`来说，没有使用`%s`,`%d`，而是使用了`{}`，**因为RUST在底层会自动识别输出数据的类型**

**和其它语言不同，Rust的集合类型不能直接进行循环，需要变成迭代器(.iter())，才能用于迭代循环（在2021 Edition之后，可以直接使用for region in regions，Rust会隐式转换为regions.iter()）**

```rust
fn main(){
    let penguin_data = "\
    common name,length (cm)
    Little penguin,33
    Yellow-eyed penguin,65
    Fiordland penguin,60
    Invalid,data
    ";
    let records = penguin_data.lines();
    for (i,record) in records.enumerate(){
        if i==0 || record.trim().len() == 0{
            continue;
        }
        //声明一个fields变量，类型是Vec，Vec是vector的缩写，可伸缩的集合类型，动态数组
        //vec<_>代表元素类型由编译器自行判断
        let fields: Vec<_> = record.split(',').map(|field| field.trim()).collect();
        if cfg!(debug_assertions){
            // 输出到标准错误输出
            eprintln!("debug：{:?} -> {:?}",record,fields);
        }
        let name = fields[0];
        /*
        1.尝试把fields[1]的值转换为f32类型的浮点数，如果成功，则把f32值赋给length变量
        2.if let是一个匹配表达式，用于从=右边的结果，匹配出length的值
            1.当=右边的表达式执行成功，则会返回一个Ok(f32)的类型，如果失败，则会返回一个Err(e)类型
            2.同时 if let 还会做一次解构匹配，通过 Ok(length)去匹配右边的Ok(f32)，最终把相应的f32值赋给length

        3.也可以使用if let Err(length) = fields[1].parse::<f32>()，把错误匹配的结果给length
        
         */
        if let Ok(length) = fields[1].parse::<f32>(){
            //输出到标准输出
            println!("{},{}cm",name,length);
        }
    }
}
```

上面代码中，值得注意的`Rust`特性：

- Rust没有继承，因此Rust不是传统意义上的面向对象语言，但是它却有方法的使用，比如`record.trim()`和`record.split(',')`
- 高阶函数编程：函数作为参数也能作为返回值，例如`.map(|field| field.trim())`，这里的`map`方法使用闭包函数作为参数(也叫做匿名函数，lambda函数)
- 类型标注：通过::\<f32\>的使用，告诉编译器length是一个f32类型的浮点数。
- 条件编译：if cfg!(debug_assertions)，说明后面的内容只在debug模式下生效
- 隐式返回：Rust提供了`return`关键字用于函数返回，但是在很多时候，可以省略它，因为Rust是基于表达式的语言。

# Rust基础入门

```rust
//main函数无返回值
fn main(){
    /*
    使用let声明变量，进行绑定,a是不可变的（如果要更改，必须加上mut修饰）
    编译器会根据a的值自动推断a的类型：i32 (int32位)
     */
    let a = 10;
    //可以为b指定类型为i32
    let b: i32 = 10;
    //mut是mutable的缩写，也可以在数字后加上i32指定类型
    let mut c = 30i32;
    //可以使用下划线增加可读性。
    let d = 30_i32;
    // let b = 10i32;
    // println!("{}",a);
    let e = add(add(a,b),add(c,d));
    println!("{}",e);
}
fn add(i:i32,j:i32) -> i32{
    i + j//不能增加;，这会改变语法导致函数返回()
}
```

## 变量绑定与解构

type-level使用驼峰命名法，value-level使用蛇形命名法。[Rust命名规则](https://course.rs/practice/naming.html)

### 变量绑定

```rust
let a = "hello world";
```

这个过程称之为：**变量绑定**，而非以往的赋值，这涉及到Rust的核心原则：**所有权。**

**所有权：**任何内存对象都是有主人的，一般情况下完全属于它的主人，绑定就是把这个对象绑定给一个变量，让这个变量成为它的主人（**该对象之前的主人就会丧失对这个对象的所有权**）

### 变量可变性

Rust的变量在默认情况下是**不可变的**，使得代码更安全，性能更好。可以使用`mut`关键字让变量变为**可变的（只能改变其值，而不能改变其值的类型）。**避免无法预期的错误发生。

### 使用下划线开头忽略未使用的变量

创建变量后，如果不使用，会出现Warning，可以使用下划线作为变量名的开头，Rust则不会警告。

也可以使用：`#[allow(unused_variables)]`

### 变量解构

 `let`表达式不仅仅可以用于变量的绑定，还能进行复杂变量的解构：从一个相对复杂的变量中，匹配出该变量的一部分内容。

变量解构（Destructuring）是一种提取结构化数据（如元组、数组、结构体等）中的数据并将其分配给独立变量的技术。

```rust
fn main(){
    let (a,mut b): (bool,bool) = (true,false);
    //a = true,不可变，b = false，可变。这里使用的是元组解构的方法
    println!("a : {},b : {}",a,b);
    b = true;
    assert_eq!(a,b);
}
```

在Rust中，变量解构可以通过模式匹配来实现。模式匹配是一种匹配结构化数据的方式，它可以将一个给定的值与一个或多个模式进行比较，并根据匹配结果执行相应的操作。

#### 解构式赋值

可以在赋值语句的左式中使用元组、切片或者结构体模式。

```rust
struct Struct{
    e: i32
}
fn main(){
    let (a,b,c,d,e);
    (a,b) = (1,2);
    //_ 代表匹配一个值，但是不用，类似与python
    [c,..,d,_] = [1,2,3,4,5];
    //c = 1,中间应该应该代表省略，d从倒数第二个开始匹配，因为后面有一个 _ 占位
    Struct {e,..} = Struct {e:5};
    assert_eq!([1,2,1,4,5],[a,b,c,d,e]);
}
```

这里只是对之前绑定的变量进行了再赋值。

### 变量与常量之间的差异

变量和常量之间存在一些差异：

- 常量不允许使用`mut`。**常量不仅仅默认不可变，而且自始至终不可变**，因为常量在编译完成后，已经确定了它的值。
- 常量使用`const`关键字而不是`let`声明。

Rust常量命名规则是**字母全部大写，并且使用下划线分隔单词，对于数字字面量，可以使用下划线提高可读性。**

```rust
const MAX_POINTS: u32 = 100_000;
```

常量可以在任意作用域内声明，包括全局作用域，**在声明的作用域中，常量在`程序运行`的整个过程中都有效**

### 变量遮蔽(shadowing)

Rust允许声明相同的变量名，在后面声明的变量会遮蔽掉前面声明的。（前提在作用域允许范围内有效）。

```rust
fn main(){
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        println!("{}",x);
    }
    println!("{}",x);
}
```

这和`mut`变量的使用是不同的，第二个`let`生成了完全不同的新变量，两个变量只是刚好名字相同，**涉及到一次内存对象的再分配**，而`mut`声明的变量，可以**修改同一内存地址的值**，并不会发生内存对象的再分配，性能要更好。

变量遮蔽的用处在于，如果你在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字。

**这里就可以避免mut的不能改变变量类型(因为是重新生成对象)**

`usize`是一种与CPU相关的整数类型。

### 课后练习

1. 变量只能在被初始化后才能使用

2. 新创建的变量无法使用+=

   ```rust
   fn main() {
       let x = 1;
       let x += 2; //错误的
       
       println!("x = {}", x); 
   }
   ```

3. 作用域是一个变量能保持合法的范围

4. 修复错误

   ```rust
   // 修复错误
   fn main() {
       println!("{}, world", x); 
   }
   
   fn define_x() {
       let x = "hello";
   }
   ```

   修改后：

   ```rust
   fn main(){
       println!("{},world",define_x());
   }
   //"hello"是一个静态字符串，即&str类型，函数要求返回String,所以需要先使用to_string方法
   fn define_x() -> String(){
       let x = "hello".to_string();
       x
   }
   //另外一种方法
   fn define_x()->&'static str {
       let x = "hello";
       x
   }
   ```

   (1){:?}是实现Debug Trait后的打印格式，比一般的打印信息更丰富。

   (2)&'static str是Rust的另一个特性生命周期的产物，'static代表全局生命周期。

## 基本类型

Rust的每个值都有确定的数据类型，总的来说分为两类：基本类型和复合类型。**基本类型往往是一个最小化原子类型，无法解构为其它类型，由以下组成：**

- 数值类型：有符号整数 (`i8`, `i16`, `i32`, `i64`, `isize`)、 无符号整数 (`u8`, `u16`, `u32`, `u64`, `usize`) 、浮点数 (`f32`, `f64`)、以及有理数、复数
- 字符串：字符串字面量和**字符串切片 `&str`**
- 布尔类型： `true`和`false`
- 字符类型: 表示单个 Unicode 字符，存储为 4 个字节
- **单元类型**: 即 `()` ，其唯一的值也是 `()`

**Rust 编译器很聪明，它可以根据变量的值和上下文中的使用方式来自动推导出变量的类型**，在某些情况下，它无法推导出变量类型，需要手动去给予一个类型标注。

```rust
let guess = "42".parse().expect("Not a number!");
```

编译器会报错。

### 整数类型

有符号类型规定的数字范围是 -(2n - 1) ~ 2n - 1 - 1，无符号类型可以存储的数字范围是 0 ~ 2n - 1。

`isize` 和 `usize` 类型取决于程序运行的计算机 CPU 类型： 若 CPU 是 32 位的，则这两个类型是 32 位的，同理，若 CPU 是 64 位，那么它们则是 64 位。

| 数字字面量         | 示例          |
| ------------------ | ------------- |
| 十进制             | `98_222`      |
| 十六进制           | `0xff`        |
| 八进制             | `0o77`        |
| 二进制             | `0b1111_0000` |
| 字节 (仅限于 `u8`) | `b'A'`        |

Rust 整型默认使用 `i32`。

#### 整形溢出

- debug：Rust 会检查整型溢出，若存在这些问题，则使程序在编译时 *panic*(崩溃,Rust 使用这个术语来表明程序因错误而退出)。
- release：Rust **不**检测溢出。相反，当检测到整型溢出时，Rust 会按照补码循环溢出（*two’s complement wrapping*）的规则处理。

要显式处理可能的溢出，可以使用标准库针对原始数字类型提供的这些方法：

- 使用 `wrapping_*` 方法在所有模式下都按照补码循环溢出规则处理，例如`wrapping_add`
- 如果使用 `checked_*` 方法时发生溢出，则返回 `None` 值
- 使用 `overflowing_*` 方法返回该值(补码循环溢出)和一个指示是否存在溢出的布尔值
- 使用 `saturating_*` 方法使值达到最小值或最大值

### 浮点类型

默认浮点类型是 `f64`，在现代的 CPU 中它的速度与 `f32` 几乎相同，但精度更高。

浮点数造成的危险：

- **浮点数往往是你想要数字的近似表达**（不确定尾数）
- **浮点数在某些特性上是反直觉的**。因为 `f32` ， `f64` 上的比较运算实现的是 `std::cmp::PartialEq` 特征(类似其他语言的接口)，但是并没有实现 `std::cmp::Eq` 特征。Rust 的 `HashMap` 数据结构，KV结构，其中K必须是实现了 `std::cmp::Eq` 特征。

对于数学上未定义的结果，会产生一个特殊的结果：Rust 的浮点数类型使用 `NaN` (not a number)来处理这些情况。

**所有跟 `NaN` 交互的操作，都会返回一个 `NaN`，`NaN`不能用于比较。**

可以使用is_nan来判断是否为NaN。

### 数字运算

Rust 支持所有数字类型的基本数学运算：加法、减法、乘法、除法和取模运算。

**但是不同类型的数字类型不能直接进行运算，比如i32和i64**

### 位运算

Rust的运算基本上和其他语言一样

| 运算符  | 说明                                                   |
| ------- | ------------------------------------------------------ |
| & 位与  | 相同位置均为1时则为1，否则为0                          |
| \| 位或 | 相同位置只要有1时则为1，否则为0                        |
| ^ 异或  | 相同位置不相同则为1，相同则为0                         |
| ! 位非  | 把位中的0和1相互取反，即0置为1，1置为0                 |
| << 左移 | 所有位向左移动指定位数，右位补0                        |
| >> 右移 | 所有位向右移动指定位数，带符号移动（正数补0，负数补1） |

### 序列(Range)

Rust 提供了一个非常简洁的方式，用来生成连续的数值，例如 `1..5`，生成从 1 到 4 的连续数字，不包含 5 ；`1..=5`，生成从 1 到 5 的连续数字，包含 5。

序列只允许用于数字或**字符类型**，原因是：它们可以连续。

```rust
for i in 'a'..='z'{
    println!({},i);
}
```

### 有理数和复数

并未在标准库中，还包括如下：

- 有理数和复数
- 任意大小的整数和任意精度的浮点数
- 固定精度的十进制小数，常用于货币相关的场景

可以直接使用Rust数值库：num

按照以下步骤引用`num`库

1. 在 `Cargo.toml` 中的 `[dependencies]` 下添加一行 `num = "0.4.0"`
2. 使用`use num::complex::Complex`引入Complex包
3. 运行cargo run

## 所有权和借用

在以往，内存安全几乎都是通过GC的方式实现，但是GC会引来性能、内存占用以及Stop the world等问题，在高性能和系统编程上是不可接受的，因此Rust采用了新的方式：**所有权系统**

三种流派：

- **垃圾回收机制(GC)**，在程序运行时不断寻找不再使用的内存，典型代表：Java、Go
- **手动管理内存的分配和释放**, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
- **通过所有权来管理内存**，编译器在编译时会根据一系列规则进行检查

**这种检查只发生在编译期，因此对于程序运行期，不会有任何性能的损失。**

接下来，将从`字符串`来引导所有权的相关知识。

### 堆(heap)和栈(stack)

栈和堆的核心目标就是为程序在运行时提供可供使用的内存空间。

栈中的**所有数据都必须占用已知且固定大小的内存空间**，假设数据大小是未知的，那么在取出数据时，你将无法取到你想要的数据。

**对于大小未知或者可能变化的数据，我们需要将它存储在堆上。**

当向堆上放入数据时，需要请求一定大小的内存空间。操作系统在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的**指针**, 该过程被称为**在堆上分配内存。**接着**该指针将被推入到栈中，之后使用这个指针来访问数据在堆上的实际地址，进而访问该数据。**

性能区别：

1. 写入

   入栈要比在堆上分配内存要快，因为入栈时操作系统无需分配新的空间，只需要将新数据放入栈顶即可。而堆则需操作系统寻找连续的内存空间。

2. 读取

   栈数据往往可以直接存储在CPU高速缓存中，而堆数据只能存储在内存中。

   访问堆上的数据比访问栈上的数据慢，因为必须先访问栈再通过栈上的指针来访问内存。

所有权与堆栈：

当代码调用函数时，传递给函数的参数（包括可能指向堆上数据的指针和局部变量）依次压入栈中，当函数调用结束时，这些值将被从栈中按照相反的顺序依次移除。

因为堆上的数据缺乏组织，**因此跟踪这些数据何时分配和释放是非常重要的**，否则堆上的数据将产生**内存泄漏** —— **这些数据将永远无法被回收。**

### 所有权原则

**关于所有权的规则：**

1. Rust中每个值都被一个变量所拥有，该变量被成为值的所有者；
2. 一个值只能被一个变量所拥有，或者一个值只能拥有一个所有者；
3. **当所有者(变量)离开作用域范围内时，这个值将被丢弃(drop)**

```rust
let s = "hello";
```

这里的`s`时被**硬编码**进程序里的字符串值(类型为`&str`，也叫其字符串字面量)，它不适用于所有场景：

- **字符串字面量时不可变的**，因此被硬编码到程序代码中；
- 并非**所有字符串的值都能在编写代码时得知**

为此，Rust提供了动态字符串类型`String`，该类型被分配到堆上，因此可以动态伸缩，也就能存储在编译时大小未知的文本。

可以使用下面的方法基于字符串字面量来创建String类型：

```rust
let s = String::from("hello");
```

`::`是一种调用操作符，这里表示调用`String`中的`from`方法。

可以修改String类型

```rust
#![allow(unused)]
fn main() {
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() 在字符串后追加字面值

println!("{}", s); // 将打印 `hello, world!`
}
```

### 变量绑定背后的数据交互

```rust
fn main() {
    let x = 5;
    let y = x;
}
```

背后的逻辑：将`5`绑定到变量`x`；接着拷贝`x`的值赋给`y`，最终`x`和`y`都等于`5`。这两个都是通过自动拷贝的方式来赋值，都被存在栈中，无需在堆上分配内存。（拷贝非常快，这个速度远比在堆上创建内存来的多）

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
}
```

这里的`String`是堆上的，不能自动拷贝。实际上`String`类型是一个复杂类型，由**存储在栈中的堆指针、字符串长度、字符串容量共同组成**，因此**堆指针**是最重要的。容量是堆内存分配空间的大小，长度是目前已经使用的大小。

Rust解决二次释放和深拷贝带来的影响的方法就是转移所有权：**当 `s1` 赋予 `s2` 后，Rust 认为 `s1` 不再有效，因此也无需在 `s1` 离开作用域后 `drop` 任何东西，这就是把所有权从 `s1` 转移给了 `s2`，`s1` 在被赋予 `s2` 后就马上失效了**。

例如：

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

由于 Rust 禁止你使用无效的引用，你会看到以下的错误：

```shell
error[E0382]: use of moved value: `s1`
 --> src/main.rs:5:28
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`, which does
  not implement the `Copy` trait
```

拷贝指针、长度和容量而不拷贝数据听起来就像浅拷贝，但是又因为 Rust 同时使第一个变量 `s1` 无效了，因此这个操作被称为 **移动(move)**，而不是浅拷贝。

再看一段代码：

```rust
fn main() {
    let x: &str = "hello, world";
    let y = x;
    println!("{},{}",x,y);
}
```

这段代码和之前的 `String` 有一个本质上的区别：在 `String` 的例子中 `s1` 持有了通过`String::from("hello")` 创建的值的所有权，而这个例子中，**`x` 只是引用了存储在二进制中的字符串 `"hello, world"`，并没有持有所有权。**

因此 `let y = x` 中，仅仅是对该引用进行了拷贝，此时 `y` 和 `x` 都引用了同一个字符串。

**Rust永远不会自动创建数据的“深拷贝”，因此，任何自动的复制都不是深拷贝，运行时性能影响较小。可以使用`clone`的方法深度拷贝**

1. 深拷贝

   ```Rust
   fn main() {
       let s1 = String::from("hello");
       let s2 = s1.clone();
       println!("s1 = {}, s2 = {}", s1, s2);
   }
   ```

2. 浅拷贝

   ```rust
   fn main() {
       let x = 5;
       let y = x;
       println!("x = {}, y = {}", x, y);
   }
   ```

   这种没有报所有权的错误。原因是像**整型这样的基本类型在编译时是已知大小的**，会被存储在栈上，所以拷贝其实际的值是快速的。**这意味着没有理由在创建变量 `y` 后使 `x` 无效（`x`、`y` 都仍然有效）。**

**Rust有一个叫做`Copy`的特征，可以用在类似于整形这样在栈中存储的类型，如果一个类型拥有`Copy`特征，那么旧的变量在赋值给其它变量后仍然可用。**

**任何基本类型的组合可以 `Copy` ，不需要分配内存或某种形式资源的类型是可以 `Copy` 的**。如下是一些 `Copy` 的类型：

- 所有整数类型，比如 `u32`
- 布尔类型，`bool`，它的值是 `true` 和 `false`
- 所有浮点数类型，比如 `f64`
- 字符类型，`char`
- **元组，**当且仅当其**包含的类型也都是 `Copy` 的时候**。比如，`(i32, i32)` 是 `Copy` 的，但 `(i32, String)` 就不是
- 不可变引用 `&T` ，例如[转移所有权](https://course.rs/basic/ownership/ownership.html#转移所有权)中的最后一个例子，**但是注意: 可变引用 `&mut T` 是不可以 Copy的**

### 函数传值与返回

**将值传递给函数，一样会发生 `移动` 或者 `复制`，就跟 `let` 语句一样**

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效,这里无法使用s字符串了

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```

同样的，函数返回值也有所有权，例如:

```Rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 移给 s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中,
                                        // 它也将返回值移给 s3
} // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃

fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
                                             // 调用它的函数

    let some_string = String::from("hello"); // some_string 进入作用域.

    some_string                              // 返回 some_string 并移出给调用的函数
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}
```

**当所有权转移时，可变性也可以随之改变**

```rust
fn main() {
    let s = String::from("hello, ");
    let mut s1 = s;//原有的s是不可修改的，这里增加了mut之后可以修改

    s1.push_str("world")
}
```

```rust
fn main() {
    let x = Box::new(5);
    // println!("{}",x);
    // let ...      // 完成该行代码，不要修改其它行！
    let mut y = x.clone();
    *y = 4;
    
    assert_eq!(*x, 5);
}
```

### 部分 move

当解构一个变量时，可以同时使用 `move` 和**引用模式(即栈中的浅拷贝)**绑定的方式。当这么做时，`部分move`就会发生：变量中一部分的所有权被转移给其它变量，而另一部分我们获取了它的引用。

在这种情况下，原变量将无法再被使用，但是它没有转移所有权的那一部分依然可以使用，也就是之前被引用的那部分。

```rust
fn main() {
    #[derive(Debug)]
    struct Person {
        name: String,
        age: Box<u8>,
    }

    let person = Person {
        name: String::from("Alice"),
        age: Box::new(20),
    };

    // 通过这种解构式模式匹配，person.name 的所有权被转移给新的变量 `name`
    // 但是，这里 `age` 变量却是对 person.age 的引用, 这里 ref 的使用相当于: let age = &person.age 
    let Person { name, ref age } = person;

    println!("The person's age is {}", age);

    println!("The person's name is {}", name);

    // Error! 原因是 person 的一部分已经被转移了所有权，因此我们无法再使用它
    //println!("The person struct is {:?}", person);

    // 虽然 `person` 作为一个整体无法再被使用，但是 `person.age` 依然可以使用
    println!("The person's age from person struct is {}", person.age);
}
```

### 引用与借用

**引用的作用域从创建开始，到它最后一次使用的地方。而不是变量的花括号。对于这种编译器优化行为，Rust叫：Non-Lexical Lifetimes(NLL)**

Rust通过`借用(Borrowing)`使用某个变量的指针或者引用，**获取变量的引用，称之为借用**。

常规引用是一个**指针类型**，指向了对象存储的内存地址，与C一样的使用方式（使用\*进行解引用）;

```rust
fn main(){
    let x = 5;
    let y = &x;
    //*y使用这个值
}
```

**不允许比较整数与引用，因为是不同的类型**

在使用时，编译器会自动进行解引用，所以有时可以不加\*

#### 不可变引用

`&`符号就是引用，**允许你使用值，但是不能获取所有权，即不能对其内容进行更改。**

```rust
fn main() {
    let s = String::from("hello");
    change(&s);
}
fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

#### 可变引用

```rust
fn main(){
    let mut s = String::from("hello");
    change(&mut s);
}
fn change(s:&mut String){
    s.push_str(",world");
}
```

首先声明`s`是可变类型，其次**创建一个`可变的引用` &mut s和`接受可变引用参数`s: &mut String**

**同一作用域，特定数据只能有一个可变引用（重点是可变引用）**

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &mut s;
    let r2 = &mut s;
    println!("{}, {}", r1, r2);
}
```

**会报错**

这段代码出错的原因在于，第一个可变借用`r1`必须要持续到最后一次使用的位置println!，**在`r1`创建和最后一次使用之间，尝试创建第二个可变借用`r2`。**

```rust
fn main(){
    let mut y = String::from("kill");
    let z = &mut y;
    println!("{}",z);
    let u = &mut y;
    println!("{}",u);
}
```

**不会报错**

这个特性即是：`borrow checker`特性。好处是Rust在编译器就避免了数据竞争。

数据竞争可由以下行为造成：

- 两个或更多的指针同时访问同一数据
- 至少有一个指针被用来写入数据
- 没有同步数据访问的机制

可以通过**手动限制变量**的作用域避免这个问题

```rust
fn main() {
let mut s = String::from("hello");
    {
        let r1 = &mut s;
    } // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用
    let r2 = &mut s;
}
```

#### 可变引用与不可变引用不能同时存在

```rust
let mut s = String::from("hello");
let r1 = &s; // 没问题
let r2 = &s; // 没问题
let r3 = &mut s; // 大问题
println!("{}, {}, and {}", r1, r2, r3);
```

会出现："无法借用可变`s`，因为它已经被借用了不可变"

理解：正在借用不可变引用的用户，肯定不希望他借用的东西，被另外一个人莫名其妙改变了。**而多个不可变引用被允许，是因为没有人能修改数据，数据不会被污染。**

#### 悬垂引用

也叫做悬垂指针，意思是：**指针指向某个值后，这个值被释放了，而指针仍然存在，其指向的内存可能不存在任何值或者已被其他变量重新使用。**

**Rust中编译器可以确保引用永远不会变为悬垂状态：当你获取数据的引用后，编译器可以确保数据不会在引用结束前被释放，想要释放数据，必须先停止其引用。**

```rust
fn main() {
    let reference_to_nothing = dangle();
}
fn dangle() -> &String {
    let s = String::from("hello");
    &s
}
```

错误：this function's return type contains a borrowed value, but there is no value for it to be borrowed from. 该函数返回了一个借用的值，但是已经找不到它所借用值的来源

可以直接返回s，**这样所有权将被移动给接收者！**

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s
}
```

**使用{:p}可以输出引用指向的地址。**

**ref与&类似，但是其使用规则不同，ref是用于指定接收变量，这时就可以不使用&，也可以代表是一个引用。**

```rust
fn main() {
    let c = '中';
    let r1 = &c;
    // 填写空白处，但是不要修改其它行的代码
    let ref r2 = c;

    assert_eq!(*r1, *r2);
    
    // 判断两个内存地址的字符串是否相等
    assert_eq!(get_addr(r1),get_addr(r2));
}
// 获取传入引用的内存地址的字符串形式
fn get_addr(r: &char) -> String {
    format!("{:p}", r)
}
```

错误: 从不可变对象借用可变

```rust
fn main() {
    // 通过修改下面一行代码来修复错误
    let s = String::from("hello, ");
    borrow_object(&mut s)
}
fn borrow_object(s: &mut String) {}

```

正确：从可变对象借用不可变

```rust
// 下面的代码没有任何错误
fn main() {
    let mut s = String::from("hello, ");

    borrow_object(&s);
    
    s.push_str("world");
}

fn borrow_object(s: &String) {}
```

## 复合类型

### 字符串与切片

String 与 &str是没办法直接相互传递的，必须使用对应的方法才能进行传递。

例：

```rust
fn main() {
  let my_name = "Pascal";
  greet(my_name);
}
fn greet(name: String) {
  println!("Hello, {}!", name);
}
```

**会报错**

#### 切片

允许引用集合中部分连续的元素序列。

对于字符串而言，切片就是对`String`类型中某一部分的引用。

```rust
fn main(){
    let s = String::from("hello,world");
    let hello = &s[0..5];
    let world = &s[6..11];
}
```

这就是创建切片的语法，使用方括号包括的一个序列：\[开始索引..终止索引\]，其中长度是`终止索引`-`开始索引`，元素包括终止索引-1，不包括终止索引所在的位置。**这里的切片基本单位是字节，而不是内部的元素**

可以直接使用`&s[4..]`，这样就会从4一直到最后一个字节。

`let slice = &s[..]`，全部切片。

**在对字符串使用切片语法时需要格外小心，切片的索引必须落在字符之间的边界位置，也就是UTF-8字符的边界，例如，中文在UTF-8中占用3个字节，下面的代码就会崩溃：**

```rust
let s = String::from("中国人");
let a = &s[0..2];
println!("{}",a);
```

这里只取了`s`字符串的前两个字节，但是连`中`都无法完整。

**字符串切片的类型是`&str`**

例如：

```rust
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);
    s.clear(); // error!
    println!("the first word is: {}", word);
}
fn first_word(s: &String) -> &str {
    &s[..1]
}
```

**会报错，因为不能同时存在可变借用和不可变借用，s.clear()的作用是清除当前String中的所有内容，需要传入s的可变引用。在s.clear()处，可变借用和不可变借用同时生效。**

**为什么字符串字面量可以同时出现呢？因为它们都是不可变借用**

其它集合类型也可以使用切片，例如数组：

```rust
let a = [1,2,3,4,5];
let b = &a[0..3];
assert_eq!(b,&[1,2,3]);
```

返回的类型是`&[i32]`，工作方式是相同的。

#### 字符串字面量是切片

```rust
let s = "Hello,world!";
```

实际上，`s`的类型是`&str`，因此可以这样声明：

```rust
let s:&str = "Hello,world!";
```

这也是为什么字符串字面量是不可变的，因为`&str`是一个不可变引用。

#### 什么是字符串？

Rust中的字符是Unicode类型，因此每个字符占据**4个字节内存空间**，但是在字符串中不一样，字符串是**UTF-8**编码，字符串中的字符所占的字节数是变化的(1-4)，这样有助于大幅降低字符串所占用的内存空间。

Rust在语言级别，只有一种字符串类型：`str`，它通常是以引用类型出现`&str`，也就是字符串切片。但是在标准库中还有`String`类型。

`str`类型是硬编码进可执行文件，也无法被修改，**但是`String`则是一个可增长、可改变且具有所有权的UTF-8字符串。当Rust用户提到字符串时，往往指的是`String`类型和`&str`字符串切片类型，这两个都是UTF-8编码。**

#### String与&str的转换

从&str转为String：

```rust
let s = String::from("hello,world!");
let s = "hello,world!".to_string();
```

`String`转为`&str`

```rust
fn main(){
    let s = String::from("hello,world!");
    say_hello(&s);
    say_hello(&s[..]);
    say_hello(s.as_str());
}
fn say_hello(s:&str){
    println!("{}",s);
}
```

**这种灵活用法是因为`deref`隐式强制转换。**

#### 字符串索引

**Rust无法使用索引的方式访问某个字符或者子串**

```rust
let s1= String::from("hello");
let h = s1[0];
```

会产生如下错误：String` cannot be indexed by `{integer}

**字符串的底层的数据存储格式实际上是`[u8]`，一个字节数组。**对于`"Hola"`来说，长度是`4`字节，但是对于`"中国人"`来说，占用的是`9`的长度，如果这时候对于这个字符串进行索引，访问&s1[0]，没有任何意义，不是字符`中`而是这个字符的三个字节的第一个字节（这样会报错，因此在通过操作索引区间来访问字符串时，需要格外小心，一不小心就得崩溃。）

#### 操作字符串

1. 追加(Push)

   可以使用`push()`方法追加字符`char`，也可以使用`push_str`方法追加字符串字面量。

   **这两个方法都是在原有的字符串上追加，并不会返回新的字符串，因此字符串必须是mut修饰**

   ```rust
   fn main(){
       let mut s = String::from("Hello ");
       s.push_str("rust");
       s.push('!');
   }
   ```

2. 插入(Insert)

   可以使用`insert()`方法插入单个字符`char`，也可以使用`insert_str()`方法插入字符串字面量，需要传入两个参数，第一个是索引，第二个是要插入的字符(串)，索引从0开始计数。插入位置是**当前索引位置**

   ```rust
   fn main(){
       let mut s = String::from("Hello rust!");
       s.insert(5,',');//Hello, rust!
       s.insert_str(6," I like");//Hello,I like rust!
   }
   ```

3. 替换(replace)

   替换有关的方法有3个

   1. replace

      可适用于`String`和`&str`，`replace`接收两个参数，第一个参数是`要被替换的字符串`，第二个参数是`新的字符串`。会匹配所有的匹配的字符串，返回一个**新的字符串**

      ```rust
      fn main(){
          let string_place = String::from("I like rust.Learning rust is my favorite!")
          let new_string_replace = string_place.replace("rust","Rust");
      }
      ```

   2. replacen

      可适用于`String`和`&str`。`replacen()`方法接收三个参数，前两个参数与`replace()`方法一样，第三个参数则表示替换的个数。**返回新的字符串。**

      ```rust
      fn main(){
          let string_place = String::from("I like rust.Learning rust is my favorite!")
          let new_string_replace = string_place.replacen("rust","Rust",1);
      }
      ```

   3. replace_range

      仅适用于`String`类型，接收两个参数，第一个参数是要替换字符串的范围，第二个参数是新的字符串。**操作原有的字符串，因此要加mut修饰。**

      ```rust
      fn main(){
          let mut string_replace_range = String::from("I like rust!");
          string_replace_range.replace_range(7..8,"R");
      }
      ```

4. 删除(Delete)

   删除相关的方法有四个

   1. pop--删除并返回字符串的**最后一个字符**

      直接操作原有的字符串。

      ```rust
      fn main(){
          let mut string_pop = String::from("rust pop 中文!");
          let p1 = string_pop.pop();//"!"
          let p2 = string_pop.pop();//"文"
          //string_pop=rust pop 中
      }
      ```

   2. remove——删除并返回字符串中指定位置的字符

      **操作原有的字符串**，但是存在返回值，其返回值是删除位置的**字符串**，只有一个参数，表示字符串起始索引。

      **remove()方法是按照字节来处理字符串的，如果参数给的位置不是合法的字符边界，则会报错**

      ```rust
      fn main(){
          let mut string_remove = String::from("测试remove方法");
          println!("string_remove 占{}个字节",std::mem::size_of_val(string_remove.as_str()));
          string_remove.remove(0);
          println!("{}",string_remove);//输出：试remove方法
          //string_remove.remove(1);这里会报错
      }
      ```

   3. truncate——删除字符串中从**指定位置开始到结尾的全部字符**

      **直接操作原来的字符串。**无返回值，**按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。**

      ```rust
      fn main(){
          let mut string_truncate = String::from("测试truncate");
          string_truncate.truncate(3);//结果：测
      }
      ```

   4. clear——清空字符串

      方法是直接操作原有的字符串。调用后，删除字符串的所有字符，相当于`truncate()`参数为0的情况。

5. 连接(Concatencate)

   1. 使用`+`或者`+=`连接字符串

      要求右边的参数必须为字符串的切片引用(Slice)类型。使用`+`时，相当于调用了`std::string`标准库的Add 方法。`+`会返回一个新的字符串，因此原有变量声明无需使用`mut`修饰，而`+=`不返回新的字符串(使用的是AddAssign函数)，在原有的基础上进行操作，所以需要mut修饰。

      **当所有权转移时，可变性也可以随之改变**

      Add函数：

      ```rust
      impl Add<&str> for String {
          type Output = String;
          #[inline]
          fn add(mut self, other: &str) -> String {
              self.push_str(other);
              //这里之所有可以进行修改，是因为一个很重要的特性
              //当所有权转移时，可变性也可以随之改变
              self
          }
      }
      ```

      AddAssign

      ```rust
      impl AddAssign<&str> for String {
          #[inline]
          fn add_assign(&mut self, other: &str) {
              self.push_str(other);
              //这里传的是自身的引用，所以原有的必须为mut的变量
          }
      }
      ```

      ```rust
      fn main() {
          let string_append = String::from("hello ");
          let string_rust = String::from("rust");
          // &string_rust会自动解引用为&str
          let result = string_append + &string_rust;
          let mut result = result + "!";
          result += "!!!";
          println!("连接字符串 + -> {}", result);
      }
      ```

      **这里注意看Add的源码，由于所有权已经被mut self拿走，并且返回给新的字符串，所以传入的string_append已经不能再次使用了。**

      以下代码也是合法的：

      ```rust
      fn main() {
          let s1 = String::from("tic");
          let s2 = String::from("tac");
          let s3 = String::from("toe");
          // String = String + &str + &str + &str + &str
          let s = s1 + "-" + &s2 + "-" + &s3;
      }
      ```

   2. 使用format!连接

      `format!`适用于`String`和`&str`。

      使用 `format!` 宏来连接两个字符串时，不会丧失任何一个字符串的所有权。相反，它会返回一个新的字符串，该字符串包含了所有连接的子字符串。

      ```rust
      let a = "Hello".to_string();
      let b = "world".to_string();
      let c = format!("{} {}", a, b);
      ```

#### 字符串转移

通过`\字符的十六进制`的表示，转移输出1个字符。例如`\x52`代表`r`

使用`\u{十六进制}`，例如`\u{211D}`代表双写R。

换行了也会保持之前之前的字符串格式，但可以使用`\`中止。

```rust
fn main() {
    // 通过 \ + 字符的十六进制表示，转义输出一个字符
    let byte_escape = "I'm writing \x52\x75\x73\x74!";
    println!("What are you doing\x3F (\\x3F means ?) {}", byte_escape);

    // \u 可以输出一个 unicode 字符
    let unicode_codepoint = "\u{211D}";
    let character_name = "\"DOUBLE-STRUCK CAPITAL R\"";

    println!(
        "Unicode character {} (U+211D) is called {}",
        unicode_codepoint, character_name
    );

    // 换行了也会保持之前的字符串格式
    let long_string = "String literals
                        can span multiple lines.
                        The linebreak and indentation here ->\
                        <- can be escaped too!";
    println!("{}", long_string);
}
```

如果不需要进行转义，则可以通过增加r前缀。

`r"Escapes don't work here:\3F \u{211D}`

如果内部包含双引号，可以在开头和结尾增加`#`（如果仍然有歧义，则可以继续追加。）

`r#"And then I said: "There is no escape!""#;`

#### 操作UTF-8字符串

如果想以Unicode字符的方式遍历字符串，最好的办法是使用`chars`方法，例如：

```rust
for c in "中国人".chars(){
    println!("{}",c);
}
```

`bytes()`可以返回字符串的底层字节数组表示形式。

```rust
for b in "中国人".bytes(){
    println!("{}",b);
}
```

Rust提供了一个释放内存的函数`drop`，Rust在变量离开作用域时，自动调用drop函数。

#### 课后习题

正常情况下，无法使用`str`类型，但是可以通过`&str`来代替，如果实在需要使用`str`类型，只能配合`Box`，此时可以使用`&`将`Box<str>`转换为`&str`。

```rust
// 使用至少两种方法来修复错误
fn main() {
    let s: Box<str> = "hello, world".into();
    //greetings(&s);方案1
    //greetings(&s[..]);方案2
    //greetings(s);并且fn greetings(s:Box<str>)，此为方案3
}
fn greetings(s: &str) {
    println!("{}",s)
}
```

```rust
fn main() {
    let s: Box<&str> = "hello, world".into();
    greetings(*s)
}
fn greetings(s: &str) {
    println!("{}", s);
}
```

方案4

如果raw string中想使用双引号，那么可以使用r#"字符串"#的形式，如果还想使用#，那么可以使用r###""###的形式。

可以使用`String::new()`生成一个空的字符串

**字节字符串：**

想要一个非UTF-8形式的字符串可以使用字节字符串或者字节数组

```rust
fn main(){
    //长度为21的字节数组
    let bytestring:&[u8;21] = b"this is a byte string";
    //字节数组也可以使用转移
    let escaped = b"\x52\75\73\x74";
    //但是不支持unicode转移。
    //raw_string
    let raw_bytestring = br"\u{221D}is not escaped here";
    //将字节数组转为`str`可能会失败
    if let Ok(my_str) = str::from_utf8(raw_bytestring){
        ...;
    }
}
```

可以utf8_slice库访问UTF-8字符串的某个子串，它索引的是字符而不是字节。

一个切片引用占用了2个**字（而非字节）**大小的内存空间，该切片的第一个字是指向数据的指针，第二个字是切片的长度。字的大小取决于处理器架构，如果是64位，则一个切片引用是16个字节大小。

**切片引用可以用来借用数组的某个连续部分，对应的签名是`&[T]，数组的签名为：[T; length]`**

`&String`可以被隐式转换为`&str`类型。通过Deref。

`String`的底层是`Vec<u8>`，分配在堆上、可增长且不是以`null`结尾。

`&str`是切片引用类型`&[u8]`，指向一个合法的UTF-8序列。

二者的关系类似于`&[T]`和`Vec<T>`

```rust
// 填空
fn main() {  
   let mut s = String::from("hello, world");

   let slice1: &str = &s; // 使用两种方法
//   let slice1:&str = s.as_str();
   assert_eq!(slice1, "hello, world");
   let slice2 = &s[0..5];
   assert_eq!(slice2, "hello");
   let slice3: &mut String = &mut s; 
   slice3.push('!');
   assert_eq!(slice3, "hello, world!");
   println!("Success!")
}
```

`String`是一个智能指针，它作为一个结构体存储在栈上，然后指向存储在栈上的字符串底层数据。

存储在栈上的智能指针结构体由三部分组成：一个只指向堆上的字节数组，已使用的长度以及已分配的容量capacity（已使用的长度小于等于已分配的容量，如果容量不够时，会重新分配空间）。如果当前String的当前容量足够，那么添加字符将不会导致新的内存分配。

```rust
// 修改下面的代码以打印如下内容: 
// 25
// 25
// 25
// 循环中不会发生任何内存分配
fn main() {
    let mut s = String::with_capacity(25);
    println!("{}", s.capacity());
    for _ in 0..=2 {
        s.push_str("hello");
        println!("{}", s.capacity());
    }
    println!("Success!")=
}
```

可以通过capacity()方法查看当前容量，通过with_capacity()方法设置容量。

可以通过as_mut_ptr()获得当前String类型指向堆数组的指针，通过len()获得字符串的长度，通过capacity()获得当前的容量。

通过from_raw_parts(ptr,len,capacity)可以通过指针，长度，容量重构String类型。

# 寻找牛刀，以便小试

## 安装Rust环境

安装rust命令：

```shell
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
#Rustup metadata and toolchains will be installed into the Rustup
#home directory, located at:

#  /home/yuwei/.rustup

#This can be modified with the RUSTUP_HOME environment variable.

#The Cargo home directory is located at:

#  /home/yuwei/.cargo

#This can be modified with the CARGO_HOME environment variable.

#The cargo, rustc, rustup and other commands will be added to
#Cargo's bin directory, located at:

#  /home/yuwei/.cargo/bin

#This path will then be added to your PATH environment variable by
#modifying the profile files located at:

#  /home/yuwei/.profile
#  /home/yuwei/.bashrc
```

让浏览器打开本地文档

```shell
rustup doc
```

## 认识Cargo

`cargo` 提供了一系列的工具，从项目的建立、构建到测试、运行直至部署，为 Rust 项目的管理提供尽可能完整的手段。同时，与 Rust 语言及其编译器 `rustc` 紧密结合。

```shell
cargo new world_hello
cd ./world_hello
```

使用`cargo new`命令创建一个项目，项目名位world_hello，该项目的结构和配置文件都是由cargo生成，**意味着项目被cargo管理**

Rust项目主要分为两个类型：`bin`和`lib`，前者是一个**可运行的项目**，后者是一个**依赖库项目**。**默认创建的是bin项目。**

通过tree命令查看项目的结构

```shell
$ tree
.
├── .git
├── .gitignore
├── Cargo.toml
└── src
    └── main.rs
```

**运行rust项目：**

```shell
cargo run#前提是在项目的文件夹中
```

该命令首先对项目进行编译，然后再运行，相当于运行了两个命令

```shell
cargo build
./target/debug/world_hello
#target文件为编译后生成的文件夹，debug则代表运行是debug模式
```

在`debug`模式下，代码的**编译速度非常快**，但是**运行速度变慢**，`debug`模式下，**Rust编译器不会做任何的优化**，只为了尽快的编译完成。

可以通过增加`--release`来避免使用`debug`模式

```shell
cargo run --release
cargo build --release
#这时就可以使用./target/release/world_hello来执行了
```

`cargo check`**可以快速检查代码是否能编译通过**

```shell
cargo check
    Checking world_hello v0.1.0 (/Users/sunfei/development/rust/world_hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s
```

## Cargo.toml和Cargo.lock

cargo的活动都基于这两个文件

- Cargo.html是cargo特有的**项目数据描述文件**，存储了项目的所有元配置信息

  ```toml
  [package]
  name = "world_hello"
  version = "0.1.0"
  edition = "2021"
  
  # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
  
  [dependencies]
  ```

  package中记录了项目的描述信息。

  这里的`name`字段定义了项目名称，`version`定义了当前版本，新项目默认的是`0.1.0`，`edition`字段定义了使用的Rust大版本。

  **主要通过各种依赖段落来描述该项目的各种依赖项**

  - 基于Rust官方仓库`crates.io`，通过版本说明来描述
  - 基于项目源代码的git仓库地址，通过URL来描述
  - 基于本地项目的绝对路径或者相对路径，通过类Unix模式的路径来描述

  三种形式的具体写法如下：

  ```toml
  [dependencies]
  rand = "0.3"
  hammer = { version = "0.5.0"}
  color = { git = "https://github.com/bjz/color-rs" }
  geometry = { path = "crates/geometry" }
  ```

- Cargo.lock是cargo根据同一项目的toml文件生成的**项目依赖详细清单**，一般不需要修改

## Rust语言初印象

```rust
fn greet_world() {
    let southern_germany = "Grüß Gott!";
    let chinese = "世界，你好";
    let english = "World, hello";
    let regions = [southern_germany, chinese, english];
    for region in regions.iter() {
        println!("{}", &region);
    }
}
fn main() {
    greet_world();
}
```

Rust原生支持`UTF-8`编码的字符串。println后面的`!`，为Rust中的`宏`操作符。

对于`println`来说，没有使用`%s`,`%d`，而是使用了`{}`，**因为RUST在底层会自动识别输出数据的类型**

**和其它语言不同，Rust的集合类型不能直接进行循环，需要变成迭代器(.iter())，才能用于迭代循环（在2021 Edition之后，可以直接使用for region in regions，Rust会隐式转换为regions.iter()）**

```rust
fn main(){
    let penguin_data = "\
    common name,length (cm)
    Little penguin,33
    Yellow-eyed penguin,65
    Fiordland penguin,60
    Invalid,data
    ";
    let records = penguin_data.lines();
    for (i,record) in records.enumerate(){
        if i==0 || record.trim().len() == 0{
            continue;
        }
        //声明一个fields变量，类型是Vec，Vec是vector的缩写，可伸缩的集合类型，动态数组
        //vec<_>代表元素类型由编译器自行判断
        let fields: Vec<_> = record.split(',').map(|field| field.trim()).collect();
        if cfg!(debug_assertions){
            // 输出到标准错误输出
            eprintln!("debug：{:?} -> {:?}",record,fields);
        }
        let name = fields[0];
        /*
        1.尝试把fields[1]的值转换为f32类型的浮点数，如果成功，则把f32值赋给length变量
        2.if let是一个匹配表达式，用于从=右边的结果，匹配出length的值
            1.当=右边的表达式执行成功，则会返回一个Ok(f32)的类型，如果失败，则会返回一个Err(e)类型
            2.同时 if let 还会做一次解构匹配，通过 Ok(length)去匹配右边的Ok(f32)，最终把相应的f32值赋给length

        3.也可以使用if let Err(length) = fields[1].parse::<f32>()，把错误匹配的结果给length
        
         */
        if let Ok(length) = fields[1].parse::<f32>(){
            //输出到标准输出
            println!("{},{}cm",name,length);
        }
    }
}
```

上面代码中，值得注意的`Rust`特性：

- Rust没有继承，因此Rust不是传统意义上的面向对象语言，但是它却有方法的使用，比如`record.trim()`和`record.split(',')`
- 高阶函数编程：函数作为参数也能作为返回值，例如`.map(|field| field.trim())`，这里的`map`方法使用闭包函数作为参数(也叫做匿名函数，lambda函数)
- 类型标注：通过::\<f32\>的使用，告诉编译器length是一个f32类型的浮点数。
- 条件编译：if cfg!(debug_assertions)，说明后面的内容只在debug模式下生效
- 隐式返回：Rust提供了`return`关键字用于函数返回，但是在很多时候，可以省略它，因为Rust是基于表达式的语言。

# Rust基础入门

```rust
//main函数无返回值
fn main(){
    /*
    使用let声明变量，进行绑定,a是不可变的（如果要更改，必须加上mut修饰）
    编译器会根据a的值自动推断a的类型：i32 (int32位)
     */
    let a = 10;
    //可以为b指定类型为i32
    let b: i32 = 10;
    //mut是mutable的缩写，也可以在数字后加上i32指定类型
    let mut c = 30i32;
    //可以使用下划线增加可读性。
    let d = 30_i32;
    // let b = 10i32;
    // println!("{}",a);
    let e = add(add(a,b),add(c,d));
    println!("{}",e);
}
fn add(i:i32,j:i32) -> i32{
    i + j//不能增加;，这会改变语法导致函数返回()
}
```

## 变量绑定与解构

type-level使用驼峰命名法，value-level使用蛇形命名法。[Rust命名规则](https://course.rs/practice/naming.html)

### 变量绑定

```rust
let a = "hello world";
```

这个过程称之为：**变量绑定**，而非以往的赋值，这涉及到Rust的核心原则：**所有权。**

**所有权：**任何内存对象都是有主人的，一般情况下完全属于它的主人，绑定就是把这个对象绑定给一个变量，让这个变量成为它的主人（**该对象之前的主人就会丧失对这个对象的所有权**）

### 变量可变性

Rust的变量在默认情况下是**不可变的**，使得代码更安全，性能更好。可以使用`mut`关键字让变量变为**可变的（只能改变其值，而不能改变其值的类型）。**避免无法预期的错误发生。

### 使用下划线开头忽略未使用的变量

创建变量后，如果不使用，会出现Warning，可以使用下划线作为变量名的开头，Rust则不会警告。

也可以使用：`#[allow(unused_variables)]`

### 变量解构

 `let`表达式不仅仅可以用于变量的绑定，还能进行复杂变量的解构：从一个相对复杂的变量中，匹配出该变量的一部分内容。

变量解构（Destructuring）是一种提取结构化数据（如元组、数组、结构体等）中的数据并将其分配给独立变量的技术。

```rust
fn main(){
    let (a,mut b): (bool,bool) = (true,false);
    //a = true,不可变，b = false，可变。这里使用的是元组解构的方法
    println!("a : {},b : {}",a,b);
    b = true;
    assert_eq!(a,b);
}
```

在Rust中，变量解构可以通过模式匹配来实现。模式匹配是一种匹配结构化数据的方式，它可以将一个给定的值与一个或多个模式进行比较，并根据匹配结果执行相应的操作。

#### 解构式赋值

可以在赋值语句的左式中使用元组、切片或者结构体模式。

```rust
struct Struct{
    e: i32
}
fn main(){
    let (a,b,c,d,e);
    (a,b) = (1,2);
    //_ 代表匹配一个值，但是不用，类似与python
    [c,..,d,_] = [1,2,3,4,5];
    //c = 1,中间应该应该代表省略，d从倒数第二个开始匹配，因为后面有一个 _ 占位
    Struct {e,..} = Struct {e:5};
    assert_eq!([1,2,1,4,5],[a,b,c,d,e]);
}
```

这里只是对之前绑定的变量进行了再赋值。

### 变量与常量之间的差异

变量和常量之间存在一些差异：

- 常量不允许使用`mut`。**常量不仅仅默认不可变，而且自始至终不可变**，因为常量在编译完成后，已经确定了它的值。
- 常量使用`const`关键字而不是`let`声明。

Rust常量命名规则是**字母全部大写，并且使用下划线分隔单词，对于数字字面量，可以使用下划线提高可读性。**

```rust
const MAX_POINTS: u32 = 100_000;
```

常量可以在任意作用域内声明，包括全局作用域，**在声明的作用域中，常量在`程序运行`的整个过程中都有效**

### 变量遮蔽(shadowing)

Rust允许声明相同的变量名，在后面声明的变量会遮蔽掉前面声明的。（前提在作用域允许范围内有效）。

```rust
fn main(){
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        println!("{}",x);
    }
    println!("{}",x);
}
```

这和`mut`变量的使用是不同的，第二个`let`生成了完全不同的新变量，两个变量只是刚好名字相同，**涉及到一次内存对象的再分配**，而`mut`声明的变量，可以**修改同一内存地址的值**，并不会发生内存对象的再分配，性能要更好。

变量遮蔽的用处在于，如果你在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字。

**这里就可以避免mut的不能改变变量类型(因为是重新生成对象)**

`usize`是一种与CPU相关的整数类型。

### 课后练习

1. 变量只能在被初始化后才能使用

2. 新创建的变量无法使用+=

   ```rust
   fn main() {
       let x = 1;
       let x += 2; //错误的
       
       println!("x = {}", x); 
   }
   ```

3. 作用域是一个变量能保持合法的范围

4. 修复错误

   ```rust
   // 修复错误
   fn main() {
       println!("{}, world", x); 
   }
   
   fn define_x() {
       let x = "hello";
   }
   ```

   修改后：

   ```rust
   fn main(){
       println!("{},world",define_x());
   }
   //"hello"是一个静态字符串，即&str类型，函数要求返回String,所以需要先使用to_string方法
   fn define_x() -> String(){
       let x = "hello".to_string();
       x
   }
   //另外一种方法
   fn define_x()->&'static str {
       let x = "hello";
       x
   }
   ```

   (1){:?}是实现Debug Trait后的打印格式，比一般的打印信息更丰富。

   (2)&'static str是Rust的另一个特性生命周期的产物，'static代表全局生命周期。

## 基本类型

Rust的每个值都有确定的数据类型，总的来说分为两类：基本类型和复合类型。**基本类型往往是一个最小化原子类型，无法解构为其它类型，由以下组成：**

- 数值类型：有符号整数 (`i8`, `i16`, `i32`, `i64`, `isize`)、 无符号整数 (`u8`, `u16`, `u32`, `u64`, `usize`) 、浮点数 (`f32`, `f64`)、以及有理数、复数
- 字符串：字符串字面量和**字符串切片 `&str`**
- 布尔类型： `true`和`false`
- 字符类型: 表示单个 Unicode 字符，存储为 4 个字节
- **单元类型**: 即 `()` ，其唯一的值也是 `()`

**Rust 编译器很聪明，它可以根据变量的值和上下文中的使用方式来自动推导出变量的类型**，在某些情况下，它无法推导出变量类型，需要手动去给予一个类型标注。

```rust
let guess = "42".parse().expect("Not a number!");
```

编译器会报错。

### 数值类型

#### 整数类型

有符号类型规定的数字范围是 -(2n - 1) ~ 2n - 1 - 1，无符号类型可以存储的数字范围是 0 ~ 2n - 1。

`isize` 和 `usize` 类型取决于程序运行的计算机 CPU 类型： 若 CPU 是 32 位的，则这两个类型是 32 位的，同理，若 CPU 是 64 位，那么它们则是 64 位。

| 数字字面量         | 示例          |
| ------------------ | ------------- |
| 十进制             | `98_222`      |
| 十六进制           | `0xff`        |
| 八进制             | `0o77`        |
| 二进制             | `0b1111_0000` |
| 字节 (仅限于 `u8`) | `b'A'`        |

Rust 整型默认使用 `i32`。

##### 整形溢出

- debug：Rust 会检查整型溢出，若存在这些问题，则使程序在编译时 *panic*(崩溃,Rust 使用这个术语来表明程序因错误而退出)。
- release：Rust **不**检测溢出。相反，当检测到整型溢出时，Rust 会按照补码循环溢出（*two’s complement wrapping*）的规则处理。

要显式处理可能的溢出，可以使用标准库针对原始数字类型提供的这些方法：

- 使用 `wrapping_*` 方法在所有模式下都按照补码循环溢出规则处理，例如`wrapping_add`
- 如果使用 `checked_*` 方法时发生溢出，则返回 `None` 值
- 使用 `overflowing_*` 方法返回该值(补码循环溢出)和一个指示是否存在溢出的布尔值
- 使用 `saturating_*` 方法使值达到最小值或最大值

#### 浮点类型

默认浮点类型是 `f64`，在现代的 CPU 中它的速度与 `f32` 几乎相同，但精度更高。

浮点数造成的危险：

- **浮点数往往是你想要数字的近似表达**（不确定尾数）
- **浮点数在某些特性上是反直觉的**。因为 `f32` ， `f64` 上的比较运算实现的是 `std::cmp::PartialEq` 特征(类似其他语言的接口)，但是并没有实现 `std::cmp::Eq` 特征。Rust 的 `HashMap` 数据结构，KV结构，其中K必须是实现了 `std::cmp::Eq` 特征。

对于数学上未定义的结果，会产生一个特殊的结果：Rust 的浮点数类型使用 `NaN` (not a number)来处理这些情况。

**所有跟 `NaN` 交互的操作，都会返回一个 `NaN`，`NaN`不能用于比较。**

可以使用is_nan来判断是否为NaN。

#### 数字运算

Rust 支持所有数字类型的基本数学运算：加法、减法、乘法、除法和取模运算。

**但是不同类型的数字类型不能直接进行运算，比如i32和i64**

#### 位运算

Rust的运算基本上和其他语言一样

| 运算符  | 说明                                                   |
| ------- | ------------------------------------------------------ |
| & 位与  | 相同位置均为1时则为1，否则为0                          |
| \| 位或 | 相同位置只要有1时则为1，否则为0                        |
| ^ 异或  | 相同位置不相同则为1，相同则为0                         |
| ! 位非  | 把位中的0和1相互取反，即0置为1，1置为0                 |
| << 左移 | 所有位向左移动指定位数，右位补0                        |
| >> 右移 | 所有位向右移动指定位数，带符号移动（正数补0，负数补1） |

#### 序列(Range)

Rust 提供了一个非常简洁的方式，用来生成连续的数值，例如 `1..5`，生成从 1 到 4 的连续数字，不包含 5 ；`1..=5`，生成从 1 到 5 的连续数字，包含 5。

序列只允许用于数字或**字符类型**，原因是：它们可以连续。

```rust
for i in 'a'..='z'{
    println!({},i);
}
```

#### 有理数和复数

并未在标准库中，还包括如下：

- 有理数和复数
- 任意大小的整数和任意精度的浮点数
- 固定精度的十进制小数，常用于货币相关的场景

可以直接使用Rust数值库：num

按照以下步骤引用`num`库

1. 在 `Cargo.toml` 中的 `[dependencies]` 下添加一行 `num = "0.4.0"`
2. 使用`use num::complex::Complex`引入Complex包
3. 运行cargo run

### 字符、布尔、单元类型

#### 字符类型(Char)

Rust 的字符不仅仅是 `ASCII`，所有的 `Unicode` 值都可以作为 Rust 字符，包括单个的中文、日文、韩文、emoji 表情符号等等，都是合法的字符类型。`Unicode` 值的范围从 `U+0000 ~ U+D7FF` 和 `U+E000 ~ U+10FFFF`,字符类型也是占用 4 个字节：

#### 布尔bool

Rust 中的布尔类型有两个可能的值：`true` 和 `false`，布尔值占用内存的大小为 `1` 个字节

### 单元类型

单元类型就是 `()`，唯一的值也是 `()`。

 `main` 函数返回单元类型 `()`，而不是无返回值，无返回值的函数在Rust中有单独的定义：发散函数。

可以用 `()` 作为 `map` 的值（value），表示不关注具体的值，只关注key。

可以作为一个值用来占位，但是完全**不占用**任何内存。`()`不占用内存

### 语句和表达式

Rust 的函数体是由一系列语句组成，最后由一个表达式来返回值

**语句会执行一些操作但是不会返回一个值**，**而表达式会在求值后返回一个值**

由于 `let` 是语句，因此不能将 `let` 语句赋值给其它值

#### 表达式

表达式会进行求值，然后返回一个值。表达式可以成为语句的一部分，例如 `let y = 6` 中，`6` 就是一个表达式，它在求值后返回一个值 `6`

调用一个函数是表达式，因为会返回一个值，调用宏也是表达式，用花括号包裹最终返回一个值的语句块也是表达式，总之，能返回值，它就是表达式:

```rust
{
    let x = 3;
    x + 1
}
```

该语句块是表达式的原因是：它的最后一行是表达式，返回了 `x + 1` 的值，注意 `x + 1` 不能以分号结尾，否则就会从表达式变成语句， **表达式不能包含分号**。

最后，表达式如果不返回任何值，会隐式地返回一个 [`()`](https://course.rs/basic/base-type/char-bool.html#单元类型) 。

`x += 2`不是一个表达式，它是一个语句`()`

`x = x + 2`也不是一个表达式，它是一个语句，返回`()`

### 函数



## 所有权和借用

在以往，内存安全几乎都是通过GC的方式实现，但是GC会引来性能、内存占用以及Stop the world等问题，在高性能和系统编程上是不可接受的，因此Rust采用了新的方式：**所有权系统**

三种流派：

- **垃圾回收机制(GC)**，在程序运行时不断寻找不再使用的内存，典型代表：Java、Go
- **手动管理内存的分配和释放**, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
- **通过所有权来管理内存**，编译器在编译时会根据一系列规则进行检查

**这种检查只发生在编译期，因此对于程序运行期，不会有任何性能的损失。**

接下来，将从`字符串`来引导所有权的相关知识。

### 堆(heap)和栈(stack)

栈和堆的核心目标就是为程序在运行时提供可供使用的内存空间。

栈中的**所有数据都必须占用已知且固定大小的内存空间**，假设数据大小是未知的，那么在取出数据时，你将无法取到你想要的数据。

**对于大小未知或者可能变化的数据，我们需要将它存储在堆上。**

当向堆上放入数据时，需要请求一定大小的内存空间。操作系统在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的**指针**, 该过程被称为**在堆上分配内存。**接着**该指针将被推入到栈中，之后使用这个指针来访问数据在堆上的实际地址，进而访问该数据。**

性能区别：

1. 写入

   入栈要比在堆上分配内存要快，因为入栈时操作系统无需分配新的空间，只需要将新数据放入栈顶即可。而堆则需操作系统寻找连续的内存空间。

2. 读取

   栈数据往往可以直接存储在CPU高速缓存中，而堆数据只能存储在内存中。

   访问堆上的数据比访问栈上的数据慢，因为必须先访问栈再通过栈上的指针来访问内存。

所有权与堆栈：

当代码调用函数时，传递给函数的参数（包括可能指向堆上数据的指针和局部变量）依次压入栈中，当函数调用结束时，这些值将被从栈中按照相反的顺序依次移除。

因为堆上的数据缺乏组织，**因此跟踪这些数据何时分配和释放是非常重要的**，否则堆上的数据将产生**内存泄漏** —— **这些数据将永远无法被回收。**

### 所有权原则

**关于所有权的规则：**

1. Rust中每个值都被一个变量所拥有，该变量被成为值的所有者；
2. 一个值只能被一个变量所拥有，或者一个值只能拥有一个所有者；
3. **当所有者(变量)离开作用域范围内时，这个值将被丢弃(drop)**

```rust
let s = "hello";
```

这里的`s`时被**硬编码**进程序里的字符串值(类型为`&str`，也叫其字符串字面量)，它不适用于所有场景：

- **字符串字面量时不可变的**，因此被硬编码到程序代码中；
- 并非**所有字符串的值都能在编写代码时得知**

为此，Rust提供了动态字符串类型`String`，该类型被分配到堆上，因此可以动态伸缩，也就能存储在编译时大小未知的文本。

可以使用下面的方法基于字符串字面量来创建String类型：

```rust
let s = String::from("hello");
```

`::`是一种调用操作符，这里表示调用`String`中的`from`方法。

可以修改String类型

```rust
#![allow(unused)]
fn main() {
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() 在字符串后追加字面值

println!("{}", s); // 将打印 `hello, world!`
}
```

### 变量绑定背后的数据交互

```rust
fn main() {
    let x = 5;
    let y = x;
}
```

背后的逻辑：将`5`绑定到变量`x`；接着拷贝`x`的值赋给`y`，最终`x`和`y`都等于`5`。这两个都是通过自动拷贝的方式来赋值，都被存在栈中，无需在堆上分配内存。（拷贝非常快，这个速度远比在堆上创建内存来的多）

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
}
```

这里的`String`是堆上的，不能自动拷贝。实际上`String`类型是一个复杂类型，由**存储在栈中的堆指针、字符串长度、字符串容量共同组成**，因此**堆指针**是最重要的。容量是堆内存分配空间的大小，长度是目前已经使用的大小。

Rust解决二次释放和深拷贝带来的影响的方法就是转移所有权：**当 `s1` 赋予 `s2` 后，Rust 认为 `s1` 不再有效，因此也无需在 `s1` 离开作用域后 `drop` 任何东西，这就是把所有权从 `s1` 转移给了 `s2`，`s1` 在被赋予 `s2` 后就马上失效了**。

例如：

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

由于 Rust 禁止你使用无效的引用，你会看到以下的错误：

```shell
error[E0382]: use of moved value: `s1`
 --> src/main.rs:5:28
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`, which does
  not implement the `Copy` trait
```

拷贝指针、长度和容量而不拷贝数据听起来就像浅拷贝，但是又因为 Rust 同时使第一个变量 `s1` 无效了，因此这个操作被称为 **移动(move)**，而不是浅拷贝。

再看一段代码：

```rust
fn main() {
    let x: &str = "hello, world";
    let y = x;
    println!("{},{}",x,y);
}
```

这段代码和之前的 `String` 有一个本质上的区别：在 `String` 的例子中 `s1` 持有了通过`String::from("hello")` 创建的值的所有权，而这个例子中，**`x` 只是引用了存储在二进制中的字符串 `"hello, world"`，并没有持有所有权。**

因此 `let y = x` 中，仅仅是对该引用进行了拷贝，此时 `y` 和 `x` 都引用了同一个字符串。

**Rust永远不会自动创建数据的“深拷贝”，因此，任何自动的复制都不是深拷贝，运行时性能影响较小。可以使用`clone`的方法深度拷贝**

1. 深拷贝

   ```Rust
   fn main() {
       let s1 = String::from("hello");
       let s2 = s1.clone();
       println!("s1 = {}, s2 = {}", s1, s2);
   }
   ```

2. 浅拷贝

   ```rust
   fn main() {
       let x = 5;
       let y = x;
       println!("x = {}, y = {}", x, y);
   }
   ```

   这种没有报所有权的错误。原因是像**整型这样的基本类型在编译时是已知大小的**，会被存储在栈上，所以拷贝其实际的值是快速的。**这意味着没有理由在创建变量 `y` 后使 `x` 无效（`x`、`y` 都仍然有效）。**

**Rust有一个叫做`Copy`的特征，可以用在类似于整形这样在栈中存储的类型，如果一个类型拥有`Copy`特征，那么旧的变量在赋值给其它变量后仍然可用。**

**任何基本类型的组合可以 `Copy` ，不需要分配内存或某种形式资源的类型是可以 `Copy` 的**。如下是一些 `Copy` 的类型：

- 所有整数类型，比如 `u32`
- 布尔类型，`bool`，它的值是 `true` 和 `false`
- 所有浮点数类型，比如 `f64`
- 字符类型，`char`
- **元组，**当且仅当其**包含的类型也都是 `Copy` 的时候**。比如，`(i32, i32)` 是 `Copy` 的，但 `(i32, String)` 就不是
- 不可变引用 `&T` ，例如[转移所有权](https://course.rs/basic/ownership/ownership.html#转移所有权)中的最后一个例子，**但是注意: 可变引用 `&mut T` 是不可以 Copy的**

### 函数传值与返回

**将值传递给函数，一样会发生 `移动` 或者 `复制`，就跟 `let` 语句一样**

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效,这里无法使用s字符串了

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```

同样的，函数返回值也有所有权，例如:

```Rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 移给 s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中,
                                        // 它也将返回值移给 s3
} // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃

fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
                                             // 调用它的函数

    let some_string = String::from("hello"); // some_string 进入作用域.

    some_string                              // 返回 some_string 并移出给调用的函数
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}
```

**当所有权转移时，可变性也可以随之改变**

```rust
fn main() {
    let s = String::from("hello, ");
    let mut s1 = s;//原有的s是不可修改的，这里增加了mut之后可以修改

    s1.push_str("world")
}
```

```rust
fn main() {
    let x = Box::new(5);
    // println!("{}",x);
    // let ...      // 完成该行代码，不要修改其它行！
    let mut y = x.clone();
    *y = 4;
    
    assert_eq!(*x, 5);
}
```

### 部分 move

当解构一个变量时，可以同时使用 `move` 和**引用模式(即栈中的浅拷贝)**绑定的方式。当这么做时，`部分move`就会发生：变量中一部分的所有权被转移给其它变量，而另一部分我们获取了它的引用。

在这种情况下，原变量将无法再被使用，但是它没有转移所有权的那一部分依然可以使用，也就是之前被引用的那部分。

```rust
fn main() {
    #[derive(Debug)]
    struct Person {
        name: String,
        age: Box<u8>,
    }

    let person = Person {
        name: String::from("Alice"),
        age: Box::new(20),
    };

    // 通过这种解构式模式匹配，person.name 的所有权被转移给新的变量 `name`
    // 但是，这里 `age` 变量却是对 person.age 的引用, 这里 ref 的使用相当于: let age = &person.age 
    let Person { name, ref age } = person;

    println!("The person's age is {}", age);

    println!("The person's name is {}", name);

    // Error! 原因是 person 的一部分已经被转移了所有权，因此我们无法再使用它
    //println!("The person struct is {:?}", person);

    // 虽然 `person` 作为一个整体无法再被使用，但是 `person.age` 依然可以使用
    println!("The person's age from person struct is {}", person.age);
}
```

### 引用与借用

**引用的作用域从创建开始，到它最后一次使用的地方。而不是变量的花括号。对于这种编译器优化行为，Rust叫：Non-Lexical Lifetimes(NLL)**

Rust通过`借用(Borrowing)`使用某个变量的指针或者引用，**获取变量的引用，称之为借用**。

常规引用是一个**指针类型**，指向了对象存储的内存地址，与C一样的使用方式（使用\*进行解引用）;

```rust
fn main(){
    let x = 5;
    let y = &x;
    //*y使用这个值
}
```

**不允许比较整数与引用，因为是不同的类型**

在使用时，编译器会自动进行解引用，所以有时可以不加\*

#### 不可变引用

`&`符号就是引用，**允许你使用值，但是不能获取所有权，即不能对其内容进行更改。**

```rust
fn main() {
    let s = String::from("hello");
    change(&s);
}
fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

#### 可变引用

```rust
fn main(){
    let mut s = String::from("hello");
    change(&mut s);
}
fn change(s:&mut String){
    s.push_str(",world");
}
```

首先声明`s`是可变类型，其次**创建一个`可变的引用` &mut s和`接受可变引用参数`s: &mut String**

**同一作用域，特定数据只能有一个可变引用（重点是可变引用）**

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &mut s;
    let r2 = &mut s;
    println!("{}, {}", r1, r2);
}
```

**会报错**

这段代码出错的原因在于，第一个可变借用`r1`必须要持续到最后一次使用的位置println!，**在`r1`创建和最后一次使用之间，尝试创建第二个可变借用`r2`。**

```rust
fn main(){
    let mut y = String::from("kill");
    let z = &mut y;
    println!("{}",z);
    let u = &mut y;
    println!("{}",u);
}
```

**不会报错**

这个特性即是：`borrow checker`特性。好处是Rust在编译器就避免了数据竞争。

数据竞争可由以下行为造成：

- 两个或更多的指针同时访问同一数据
- 至少有一个指针被用来写入数据
- 没有同步数据访问的机制

可以通过**手动限制变量**的作用域避免这个问题

```rust
fn main() {
let mut s = String::from("hello");
    {
        let r1 = &mut s;
    } // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用
    let r2 = &mut s;
}
```

#### 可变引用与不可变引用不能同时存在

```rust
let mut s = String::from("hello");
let r1 = &s; // 没问题
let r2 = &s; // 没问题
let r3 = &mut s; // 大问题
println!("{}, {}, and {}", r1, r2, r3);
```

会出现："无法借用可变`s`，因为它已经被借用了不可变"

理解：正在借用不可变引用的用户，肯定不希望他借用的东西，被另外一个人莫名其妙改变了。**而多个不可变引用被允许，是因为没有人能修改数据，数据不会被污染。**

#### 悬垂引用

也叫做悬垂指针，意思是：**指针指向某个值后，这个值被释放了，而指针仍然存在，其指向的内存可能不存在任何值或者已被其他变量重新使用。**

**Rust中编译器可以确保引用永远不会变为悬垂状态：当你获取数据的引用后，编译器可以确保数据不会在引用结束前被释放，想要释放数据，必须先停止其引用。**

```rust
fn main() {
    let reference_to_nothing = dangle();
}
fn dangle() -> &String {
    let s = String::from("hello");
    &s
}
```

错误：this function's return type contains a borrowed value, but there is no value for it to be borrowed from. 该函数返回了一个借用的值，但是已经找不到它所借用值的来源

可以直接返回s，**这样所有权将被移动给接收者！**

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s
}
```

**使用{:p}可以输出引用指向的地址。**

**ref与&类似，但是其使用规则不同，ref是用于指定接收变量，这时就可以不使用&，也可以代表是一个引用。**

```rust
fn main() {
    let c = '中';
    let r1 = &c;
    // 填写空白处，但是不要修改其它行的代码
    let ref r2 = c;

    assert_eq!(*r1, *r2);
    
    // 判断两个内存地址的字符串是否相等
    assert_eq!(get_addr(r1),get_addr(r2));
}
// 获取传入引用的内存地址的字符串形式
fn get_addr(r: &char) -> String {
    format!("{:p}", r)
}
```

错误: 从不可变对象借用可变

```rust
fn main() {
    // 通过修改下面一行代码来修复错误
    let s = String::from("hello, ");
    borrow_object(&mut s)
}
fn borrow_object(s: &mut String) {}

```

正确：从可变对象借用不可变

```rust
// 下面的代码没有任何错误
fn main() {
    let mut s = String::from("hello, ");

    borrow_object(&s);
    
    s.push_str("world");
}

fn borrow_object(s: &String) {}
```

## 复合类型

### 字符串与切片

String 与 &str是没办法直接相互传递的，必须使用对应的方法才能进行传递。

例：

```rust
fn main() {
  let my_name = "Pascal";
  greet(my_name);
}
fn greet(name: String) {
  println!("Hello, {}!", name);
}
```

**会报错**

#### 切片

允许引用集合中部分连续的元素序列。

对于字符串而言，切片就是对`String`类型中某一部分的引用。

```rust
fn main(){
    let s = String::from("hello,world");
    let hello = &s[0..5];
    let world = &s[6..11];
}
```

这就是创建切片的语法，使用方括号包括的一个序列：\[开始索引..终止索引\]，其中长度是`终止索引`-`开始索引`，元素包括终止索引-1，不包括终止索引所在的位置。**这里的切片基本单位是字节，而不是内部的元素**

可以直接使用`&s[4..]`，这样就会从4一直到最后一个字节。

`let slice = &s[..]`，全部切片。

**在对字符串使用切片语法时需要格外小心，切片的索引必须落在字符之间的边界位置，也就是UTF-8字符的边界，例如，中文在UTF-8中占用3个字节，下面的代码就会崩溃：**

```rust
let s = String::from("中国人");
let a = &s[0..2];
println!("{}",a);
```

这里只取了`s`字符串的前两个字节，但是连`中`都无法完整。

**字符串切片的类型是`&str`**

例如：

```rust
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);
    s.clear(); // error!
    println!("the first word is: {}", word);
}
fn first_word(s: &String) -> &str {
    &s[..1]
}
```

**会报错，因为不能同时存在可变借用和不可变借用，s.clear()的作用是清除当前String中的所有内容，需要传入s的可变引用。在s.clear()处，可变借用和不可变借用同时生效。**

**为什么字符串字面量可以同时出现呢？因为它们都是不可变借用**

其它集合类型也可以使用切片，例如数组：

```rust
let a = [1,2,3,4,5];
let b = &a[0..3];
assert_eq!(b,&[1,2,3]);
```

返回的类型是`&[i32]`，工作方式是相同的。

#### 字符串字面量是切片

```rust
let s = "Hello,world!";
```

实际上，`s`的类型是`&str`，因此可以这样声明：

```rust
let s:&str = "Hello,world!";
```

这也是为什么字符串字面量是不可变的，因为`&str`是一个不可变引用。

#### 什么是字符串？

Rust中的字符是Unicode类型，因此每个字符占据**4个字节内存空间**，但是在字符串中不一样，字符串是**UTF-8**编码，字符串中的字符所占的字节数是变化的(1-4)，这样有助于大幅降低字符串所占用的内存空间。

Rust在语言级别，只有一种字符串类型：`str`，它通常是以引用类型出现`&str`，也就是字符串切片。但是在标准库中还有`String`类型。

`str`类型是硬编码进可执行文件，也无法被修改，**但是`String`则是一个可增长、可改变且具有所有权的UTF-8字符串。当Rust用户提到字符串时，往往指的是`String`类型和`&str`字符串切片类型，这两个都是UTF-8编码。**

#### String与&str的转换

从&str转为String：

```rust
let s = String::from("hello,world!");
let s = "hello,world!".to_string();
```

`String`转为`&str`

```rust
fn main(){
    let s = String::from("hello,world!");
    say_hello(&s);
    say_hello(&s[..]);
    say_hello(s.as_str());
}
fn say_hello(s:&str){
    println!("{}",s);
}
```

**这种灵活用法是因为`deref`隐式强制转换。**

#### 字符串索引

**Rust无法使用索引的方式访问某个字符或者子串**

```rust
let s1= String::from("hello");
let h = s1[0];
```

会产生如下错误：String` cannot be indexed by `{integer}

**字符串的底层的数据存储格式实际上是`[u8]`，一个字节数组。**对于`"Hola"`来说，长度是`4`字节，但是对于`"中国人"`来说，占用的是`9`的长度，如果这时候对于这个字符串进行索引，访问&s1[0]，没有任何意义，不是字符`中`而是这个字符的三个字节的第一个字节（这样会报错，因此在通过操作索引区间来访问字符串时，需要格外小心，一不小心就得崩溃。）

#### 操作字符串

1. 追加(Push)

   可以使用`push()`方法追加字符`char`，也可以使用`push_str`方法追加字符串字面量。

   **这两个方法都是在原有的字符串上追加，并不会返回新的字符串，因此字符串必须是mut修饰**

   ```rust
   fn main(){
       let mut s = String::from("Hello ");
       s.push_str("rust");
       s.push('!');
   }
   ```

2. 插入(Insert)

   可以使用`insert()`方法插入单个字符`char`，也可以使用`insert_str()`方法插入字符串字面量，需要传入两个参数，第一个是索引，第二个是要插入的字符(串)，索引从0开始计数。插入位置是**当前索引位置**

   ```rust
   fn main(){
       let mut s = String::from("Hello rust!");
       s.insert(5,',');//Hello, rust!
       s.insert_str(6," I like");//Hello,I like rust!
   }
   ```

3. 替换(replace)

   替换有关的方法有3个

   1. replace

      可适用于`String`和`&str`，`replace`接收两个参数，第一个参数是`要被替换的字符串`，第二个参数是`新的字符串`。会匹配所有的匹配的字符串，返回一个**新的字符串**

      ```rust
      fn main(){
          let string_place = String::from("I like rust.Learning rust is my favorite!")
          let new_string_replace = string_place.replace("rust","Rust");
      }
      ```

   2. replacen

      可适用于`String`和`&str`。`replacen()`方法接收三个参数，前两个参数与`replace()`方法一样，第三个参数则表示替换的个数。**返回新的字符串。**

      ```rust
      fn main(){
          let string_place = String::from("I like rust.Learning rust is my favorite!")
          let new_string_replace = string_place.replacen("rust","Rust",1);
      }
      ```

   3. replace_range

      仅适用于`String`类型，接收两个参数，第一个参数是要替换字符串的范围，第二个参数是新的字符串。**操作原有的字符串，因此要加mut修饰。**

      ```rust
      fn main(){
          let mut string_replace_range = String::from("I like rust!");
          string_replace_range.replace_range(7..8,"R");
      }
      ```

4. 删除(Delete)

   删除相关的方法有四个

   1. pop--删除并返回字符串的**最后一个字符**

      直接操作原有的字符串。

      ```rust
      fn main(){
          let mut string_pop = String::from("rust pop 中文!");
          let p1 = string_pop.pop();//"!"
          let p2 = string_pop.pop();//"文"
          //string_pop=rust pop 中
      }
      ```

   2. remove——删除并返回字符串中指定位置的字符

      **操作原有的字符串**，但是存在返回值，其返回值是删除位置的**字符串**，只有一个参数，表示字符串起始索引。

      **remove()方法是按照字节来处理字符串的，如果参数给的位置不是合法的字符边界，则会报错**

      ```rust
      fn main(){
          let mut string_remove = String::from("测试remove方法");
          println!("string_remove 占{}个字节",std::mem::size_of_val(string_remove.as_str()));
          string_remove.remove(0);
          println!("{}",string_remove);//输出：试remove方法
          //string_remove.remove(1);这里会报错
      }
      ```

   3. truncate——删除字符串中从**指定位置开始到结尾的全部字符**

      **直接操作原来的字符串。**无返回值，**按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。**

      ```rust
      fn main(){
          let mut string_truncate = String::from("测试truncate");
          string_truncate.truncate(3);//结果：测
      }
      ```

   4. clear——清空字符串

      方法是直接操作原有的字符串。调用后，删除字符串的所有字符，相当于`truncate()`参数为0的情况。

5. 连接(Concatencate)

   1. 使用`+`或者`+=`连接字符串

      要求右边的参数必须为字符串的切片引用(Slice)类型。使用`+`时，相当于调用了`std::string`标准库的Add 方法。`+`会返回一个新的字符串，因此原有变量声明无需使用`mut`修饰，而`+=`不返回新的字符串(使用的是AddAssign函数)，在原有的基础上进行操作，所以需要mut修饰。

      **当所有权转移时，可变性也可以随之改变**

      Add函数：

      ```rust
      impl Add<&str> for String {
          type Output = String;
          #[inline]
          fn add(mut self, other: &str) -> String {
              self.push_str(other);
              //这里之所有可以进行修改，是因为一个很重要的特性
              //当所有权转移时，可变性也可以随之改变
              self
          }
      }
      ```

      AddAssign

      ```rust
      impl AddAssign<&str> for String {
          #[inline]
          fn add_assign(&mut self, other: &str) {
              self.push_str(other);
              //这里传的是自身的引用，所以原有的必须为mut的变量
          }
      }
      ```

      ```rust
      fn main() {
          let string_append = String::from("hello ");
          let string_rust = String::from("rust");
          // &string_rust会自动解引用为&str
          let result = string_append + &string_rust;
          let mut result = result + "!";
          result += "!!!";
          println!("连接字符串 + -> {}", result);
      }
      ```

      **这里注意看Add的源码，由于所有权已经被mut self拿走，并且返回给新的字符串，所以传入的string_append已经不能再次使用了。**

      以下代码也是合法的：

      ```rust
      fn main() {
          let s1 = String::from("tic");
          let s2 = String::from("tac");
          let s3 = String::from("toe");
          // String = String + &str + &str + &str + &str
          let s = s1 + "-" + &s2 + "-" + &s3;
      }
      ```

   2. 使用format!连接

      `format!`适用于`String`和`&str`。

      使用 `format!` 宏来连接两个字符串时，不会丧失任何一个字符串的所有权。相反，它会返回一个新的字符串，该字符串包含了所有连接的子字符串。

      ```rust
      let a = "Hello".to_string();
      let b = "world".to_string();
      let c = format!("{} {}", a, b);
      ```

#### 字符串转移

通过`\字符的十六进制`的表示，转移输出1个字符。例如`\x52`代表`r`

使用`\u{十六进制}`，例如`\u{211D}`代表双写R。

换行了也会保持之前之前的字符串格式，但可以使用`\`中止。

```rust
fn main() {
    // 通过 \ + 字符的十六进制表示，转义输出一个字符
    let byte_escape = "I'm writing \x52\x75\x73\x74!";
    println!("What are you doing\x3F (\\x3F means ?) {}", byte_escape);

    // \u 可以输出一个 unicode 字符
    let unicode_codepoint = "\u{211D}";
    let character_name = "\"DOUBLE-STRUCK CAPITAL R\"";

    println!(
        "Unicode character {} (U+211D) is called {}",
        unicode_codepoint, character_name
    );

    // 换行了也会保持之前的字符串格式
    let long_string = "String literals
                        can span multiple lines.
                        The linebreak and indentation here ->\
                        <- can be escaped too!";
    println!("{}", long_string);
}
```

如果不需要进行转义，则可以通过增加r前缀。

`r"Escapes don't work here:\3F \u{211D}`

如果内部包含双引号，可以在开头和结尾增加`#`（如果仍然有歧义，则可以继续追加。）

`r#"And then I said: "There is no escape!""#;`

#### 操作UTF-8字符串

如果想以Unicode字符的方式遍历字符串，最好的办法是使用`chars`方法，例如：

```rust
for c in "中国人".chars(){
    println!("{}",c);
}
```

`bytes()`可以返回字符串的底层字节数组表示形式。

```rust
for b in "中国人".bytes(){
    println!("{}",b);
}
```

Rust提供了一个释放内存的函数`drop`，Rust在变量离开作用域时，自动调用drop函数。

#### 课后习题

正常情况下，无法使用`str`类型，但是可以通过`&str`来代替，如果实在需要使用`str`类型，只能配合`Box`，此时可以使用`&`将`Box<str>`转换为`&str`。

```rust
// 使用至少两种方法来修复错误
fn main() {
    let s: Box<str> = "hello, world".into();
    //greetings(&s);方案1
    //greetings(&s[..]);方案2
    //greetings(s);并且fn greetings(s:Box<str>)，此为方案3
}
fn greetings(s: &str) {
    println!("{}",s)
}
```

```rust
fn main() {
    let s: Box<&str> = "hello, world".into();
    greetings(*s)
}
fn greetings(s: &str) {
    println!("{}", s);
}
```

方案4

如果raw string中想使用双引号，那么可以使用r#"字符串"#的形式，如果还想使用#，那么可以使用r###""###的形式。

可以使用`String::new()`生成一个空的字符串

**字节字符串：**

想要一个非UTF-8形式的字符串可以使用字节字符串或者字节数组

```rust
fn main(){
    //长度为21的字节数组
    let bytestring:&[u8;21] = b"this is a byte string";
    //字节数组也可以使用转移
    let escaped = b"\x52\75\73\x74";
    //但是不支持unicode转移。
    //raw_string
    let raw_bytestring = br"\u{221D}is not escaped here";
    //将字节数组转为`str`可能会失败
    if let Ok(my_str) = str::from_utf8(raw_bytestring){
        ...;
    }
}
```

可以utf8_slice库访问UTF-8字符串的某个子串，它索引的是字符而不是字节。

一个切片引用占用了2个**字（而非字节）**大小的内存空间，该切片的第一个字是指向数据的指针，第二个字是切片的长度。字的大小取决于处理器架构，如果是64位，则一个切片引用是16个字节大小。

**切片引用可以用来借用数组的某个连续部分，对应的签名是`&[T]，数组的签名为：[T; length]`**

`&String`可以被隐式转换为`&str`类型。通过Deref。

`String`的底层是`Vec<u8>`，分配在堆上、可增长且不是以`null`结尾。

`&str`是切片引用类型`&[u8]`，指向一个合法的UTF-8序列。

二者的关系类似于`&[T]`和`Vec<T>`

```rust
// 填空
fn main() {  
   let mut s = String::from("hello, world");

   let slice1: &str = &s; // 使用两种方法
//   let slice1:&str = s.as_str();
   assert_eq!(slice1, "hello, world");
   let slice2 = &s[0..5];
   assert_eq!(slice2, "hello");
   let slice3: &mut String = &mut s; 
   slice3.push('!');
   assert_eq!(slice3, "hello, world!");
   println!("Success!")
}
```

`String`是一个智能指针，它作为一个结构体存储在栈上，然后指向存储在栈上的字符串底层数据。

存储在栈上的智能指针结构体由三部分组成：一个只指向堆上的字节数组，已使用的长度以及已分配的容量capacity（已使用的长度小于等于已分配的容量，如果容量不够时，会重新分配空间）。如果当前String的当前容量足够，那么添加字符将不会导致新的内存分配。

```rust
// 修改下面的代码以打印如下内容: 
// 25
// 25
// 25
// 循环中不会发生任何内存分配
fn main() {
    let mut s = String::with_capacity(25);
    println!("{}", s.capacity());
    for _ in 0..=2 {
        s.push_str("hello");
        println!("{}", s.capacity());
    }
    println!("Success!")=
}
```

可以通过capacity()方法查看当前容量，通过with_capacity()方法设置容量。

可以通过as_mut_ptr()获得当前String类型指向堆数组的指针，通过len()获得字符串的长度，通过capacity()获得当前的容量。

通过from_raw_parts(ptr,len,capacity)可以通过指针，长度，容量重构String类型。
