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

## 有理数和复数

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

