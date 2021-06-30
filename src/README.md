  [![GitHub]][repo]
  [![rustdoc]][docs]
  [![Latest Version]][crates.io]

[GitHub]: ./img/github.svg
[repo]: https://github.com/serde-rs/serde
[rustdoc]: ./img/rustdoc.svg
[docs]: https://docs.serde.rs/serde/
[Latest Version]: https://img.shields.io/crates/v/serde.svg?style=social
[crates.io]: https://crates.io/crates/serde

- 项目地址：[serde](https://github.com/serde-rs/serde)
- 英文：
[repo](https://github.com/serde-rs/serde-rs.github.io) |
[渲染版](https://serde.rs)
- 中文：
[repo](https://github.com/zjp-CN/serde-rs-book) |
[渲染版](https://zjp-cn.github.io/serde-rs-book) |
[国内阅读站点](http://129.28.186.100/serde-rs-book)

# Serde

Serde 是对 Rust 数据结构进行高效、通用地 序列化
(***ser***ializing) 和反序列化 (***de***serializing) 的框架。

Serde 生态由数据结构和数据格式组成：
* 这些数据结构 (data structures) 能够让自身序列化、反序列化；
* 这些数据格式 (data formats) 能够让数据序列化、反序列化。

Serde 提供了数据结构与格式的交互层，让任何支持的数据结构，
使用任何支持的数据格式来序列化、反序列化。

## 设计阐述

很多其他编程语言依赖于运行时映射 (runtime reflection) 来对数据序列化，
但 Serde 依赖的是 Rust 强大的 trait 系统。

给数据结构实现 Serde 的 `Serialize` 和 `Deserialize` traits 就能让其自身序列化、反序列化，
或者使用 Serde 的 derive 属性，让数据结构在编译期自动实现这两个 traits 。
这种方式避免了映射或解析运行时类型信息造成的开销。

实际上，很多情况下，Rust 编译器可以完全优化掉数据结构和数据格式之间的交互开销，
使得 Serde 序列化就像针对特定的数据结构和格式手写 serializer 一样高效。

## 数据格式

以下是支持 Serde 的部分数据格式，由社区人员编写：

- [JSON] 广泛用于 HTTP APIs 的数据格式
- [Bincode] 在 Servo渲染引擎中用于 IPC 的一种压缩二进制格式
- [CBOR] 简明二进制对象展现 (Concise Binary Object Representation) ，
  一种不需要进行版本协商的小型二进制数据交换形式
- [YAML] 自称是一种人性化（可读性高）的配置语言，但不是标记语言
- [MessagePack] 类似于对 JSON 进行压缩的高效二进制格式
- [TOML] 最小配置格式，被 [Cargo] 使用
- [Pickle] Python 世界里常用的格式
- [RON] Rusty Object Notation 看起来像 Rust 语法的数据格式，解决 JSON 无法区分同类/异类结构的问题
- [BSON] 数据存储和网络传输的格式，被 MongoDB 使用
- [Avro] 在 Apache Hadoop 内使用的一种二进制格式，支持 schema 定义
- [JSON5] JSON 的超集，涵盖 ES5 的一些功能
- [Postcard] 一种 no\_std  Rust 、对嵌入式系统友好的压缩二进制格式
- [URL] 查询字符串，采用 x-www-form-urlencoded 格式
- [Envy] 把系统环境变量反序列化成 Rust 结构体的一种方式（只支持反序列化）
- [Envy Store] 把 [AWS Parameter Store] 的参数反序列化成 Rust 结构体的一种方式（只支持反序列化）
- [S-expressions] 被 Lisp 语言家族使用的展现代码和数据的文本格式
- [D-Bus] 二进制有线格式（用于 KDE 等桌面系统的低时延、低消耗的 IPC 通讯格式）
- [FlexBuffers] Google 开发的 FlatBuffers 序列化格式，无模式 (schemaless)、零拷贝 (zero-copy)
- [Bencode] BitTorrent 协议使用的一种简单二进制格式
- [DynamoDB Items] [rusoto_dynamodb] 使用的格式，把数据转换成 DynamoDB ，
  或者从 DynamoDB 转化数据
- [Hjson] 针对 JSON 所设计的一种语法拓展，目的是可供人们阅读和编辑（只支持反序列化）

[JSON]: https://github.com/serde-rs/json
[Bincode]: https://github.com/servo/bincode
[CBOR]: https://github.com/pyfisch/cbor
[YAML]: https://github.com/dtolnay/serde-yaml
[MessagePack]: https://github.com/3Hren/msgpack-rust
[TOML]: https://github.com/alexcrichton/toml-rs
[Pickle]: https://github.com/birkenfeld/serde-pickle
[RON]: https://github.com/ron-rs/ron
[BSON]: https://github.com/zonyitoo/bson-rs
[Avro]: https://github.com/flavray/avro-rs
[JSON5]: https://github.com/callum-oakley/json5-rs
[URL]: https://docs.rs/serde_qs
[Postcard]: https://github.com/jamesmunns/postcard
[Envy]: https://github.com/softprops/envy
[Envy Store]: https://github.com/softprops/envy-store
[Cargo]: http://doc.crates.io/manifest.html
[AWS Parameter Store]: https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html
[S-expressions]: https://github.com/rotty/lexpr-rs
[D-Bus]: https://docs.rs/zvariant
[FlexBuffers]: https://github.com/google/flatbuffers/tree/master/rust/flexbuffers
[Bencode]: https://github.com/P3KI/bendy
[DynamoDB Items]: https://docs.rs/serde_dynamo
[rusoto_dynamodb]: https://docs.rs/rusoto_dynamodb
[Hjson]: https://github.com/Canop/deser-hjson

## Data structures

Out of the box, Serde is able to serialize and deserialize common Rust data
types in any of the above formats. For example `String`, `&str`, `usize`,
`Vec<T>`, `HashMap<K,V>` are all supported. In addition, Serde provides a derive
macro to generate serialization implementations for structs in your own program.
Using the derive macro goes like this:

!PLAYGROUND 72755f28f99afc95e01d63174b28c1f5
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let point = Point { x: 1, y: 2 };

    // Convert the Point to a JSON string.
    let serialized = serde_json::to_string(&point).unwrap();

    // Prints serialized = {"x":1,"y":2}
    println!("serialized = {}", serialized);

    // Convert the JSON string back to a Point.
    let deserialized: Point = serde_json::from_str(&serialized).unwrap();

    // Prints deserialized = Point { x: 1, y: 2 }
    println!("deserialized = {:?}", deserialized);
}
```
