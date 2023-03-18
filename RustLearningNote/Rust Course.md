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
- 高阶函数变成：函数作为参数也能作为返回值，例如`.map(|field| field.trim())`，这里的`map`方法使用闭包函数作为参数(也叫做匿名函数，lambda函数)
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

- 常量不允许使用`mut`。**常量不仅仅默认不可变，而且自始至终 不可变**，因为常量在编译完成后，已经确定了它的值。
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

- 使用 `wrapping_*` 方法在所有模式下都按照补码循环溢出规则处理，例如 `wrapping_add`
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

