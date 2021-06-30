# 使用 derive 宏

Serde 提供一个 derive 宏，给定义在你 crate 中的数据结构生成
`Serialize` 和 `Deserialize` traits 的方法实现，
让你的数据结构在所有 Serde 支持的数据格式中方便地映射。

**你只需要在代码中使用 `#[derive(Serialize, Deserialize)]` ，那么一些准备就绪。**

这句代码提供的功能是基于 Rust 的 `#[derive]` 机制，
就像你使用内置的 `Clone`、`Copy`、 `Debug` traits 那样，它会自动添加方法实现。
你可以给大多数带泛型、trait bound 的结构体、枚举体实现那两个 traits 。
只有在极少数的情况下，你才可能需要给一个尤为复杂的类型
[手动实现](custom-serialization.md) 那两个 traits 。

Serde 的 derive 宏要求 Rust 编译器的版本至少为 1.31 ，然后完成以下操作：

- 在 Cargo.toml 中添加 `serde = { version = "1.0", features = ["derive"] }` 依赖
- 确保所有基于 Serde 的依赖（比如 serde_json ）与 serde 1.0 兼容 
- 在你想序列化的结构体和枚举体所在的模块中，导入 derive 宏
  `use serde::Serialize;` ，然后给结构体或枚举体添加 `#[derive(Serialize)]` 
- 类似地，使用 `use serde::Deserialize;` 导入，然后给想要反序列化的结构体或枚举体添加
  `#[derive(Deserialize)]`

在 `Cargo.toml` 中写入以下内容：

```toml
[package]
name = "my-crate"
version = "0.1.0"
authors = ["Me <user@rust-lang.org>"]

[dependencies]
serde = { version = "1.0", features = ["derive"] }

# serde_json is just for the example, not required in general
serde_json = "1.0"
```

然后在 `src/main.rs` 文件中：

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let point = Point { x: 1, y: 2 };

    let serialized = serde_json::to_string(&point).unwrap();
    println!("serialized = {}", serialized);

    let deserialized: Point = serde_json::from_str(&serialized).unwrap();
    println!("deserialized = {:?}", deserialized);
}
```

输出结果：

```
$ cargo run
serialized = {"x":1,"y":2}
deserialized = Point { x: 1, y: 2 }
```

# 疑难解答

有时，及时你给结构体、枚举体加上了 `#[derive(Serialize)]`，依然遇到编译期报错：

```
the trait `serde::ser::Serialize` is not implemented for `...`
```

这说明你使用的库与 Serde 版本不兼容。
在你的 Cargo.toml 配置中依赖了 serde 1.0 版本，
但有的库依赖了 serde 0.9 。
所以你的数据结果实现了 serde 1.0 的 `Serialize` trait  ，
而那个库需要来自 serde 0.9 的 `Serialize` trait 。
Rust 编译器把它们视为不同的 traits 。

解决办法是更新或者降低版本，直到它们的 serde 版本一致。
`cargo tree -d` 命令帮助你找到下载的所有重复依赖。
