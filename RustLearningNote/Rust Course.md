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

