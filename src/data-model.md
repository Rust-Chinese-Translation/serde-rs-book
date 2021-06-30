# Serde 数据模型

Serde 数据模型就是数据结构和数据格式交互的 API 。
你可以把这个 API 视为 Serde 的类型系统。 

代码中，Serde 序列化的部分由 [`Serializer`] trait 定义，
反序列化的部分由 [`Deserializer`] trait 定义。
这两个 traits 把每个 Rust 数据结构映射到 29 种可能的类型上。
[`Serializer`] trait 的每个方法对应了一种数据模型。

当把一种数据结构序列化到某种数据格式时，
这种数据结构所实现的 [`Serialize`] 就负责把数据结构映射到 Serde 的数据模型，
这种映射是通过调用 [`Serializer`] 的某个方法做到的；
而数据格式实现的 [`Serializer`] 则负责把 Serde 的数据模型映射到准备好的输出呈现上。

当从某种格式中反序列化成数据结构时，
这种数据结构所实现的 [`Deserialize`] 就负责把数据结构映射到 Serde 的数据模型，
这种映射把实现了 [`Visitor`] trait 的数据传给 [`Deserializer`] ，
[`Visitor`] trait 可以接收数据模型的多种类型；
而数据格式实现的 [`Deserializer`] 调用 [`Visitor`] 的某个方法，
负责把输入的数据映射成 Serde 数据模型。

[`Serializer`]: https://docs.serde.rs/serde/trait.Serializer.html
[`Deserializer`]: https://docs.serde.rs/serde/trait.Deserializer.html
[`Serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`Deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html
[`Visitor`]: https://docs.serde.rs/serde/de/trait.Visitor.html

## 类型

Serde 数据模型对 Rust 类型系统做了简化，由下面 29 种类型组成：

- **14 种原生类型 (primitive)**
  - bool
  - i8, i16, i32, i64, i128
  - u8, u16, u32, u64, u128
  - f32, f64
  - char
- **字符串 (string)**
  - UTF-8 字节有长度，但是没有 null 结束符，而且可能只有 0 字节。
  - 序列化时，所有字符串等同地被处理。
  - 反序列化时，有三种字符串：临时的、有所有权的、被借用的。
    它们的区别在 [deserializer lifetimes][Understanding deserializer lifetimes] 
    一章会介绍，而且这也是 Serde 能以零拷贝方式高效反序列化的关键。
- **字节数组 (byte array)** - [u8]
  - 类似于字符串，反序列化时，有三种：临时的、有所有权的、被借用的。
- **Option**
  - None 或者有某个值。
- **unit**
  - Rust 中 `()` 的类型，它表明没有数据的匿名值。
- **unit 结构体**
  - 比如 `struct Unit`、 `PhantomData<T>`，它表明没有数据的值，但有一个名字。
- **unit 成员 (variant)**
  - 比如 `enum E { A, B }` 中的 `E::A`、 `E::B`
- **newtype 结构体**
  - 比如 `struct Millimeters(u8)`
- **newtype 成员 (variant)**
  - 比如 `enum E { N(u8) }` 中的 `E::N`
- **序列 (seq)**
  - 变长的、可存放各种类型值的序列，比如 `Vec<T>`、 `HashSet<T>`。
  - 序列化时，迭代 (iterate) 所有数据之前可能知道也可能不知道长度，
    其长度取决于序列化之后的数据。
  - 注意，像 `vec![Value::Bool(true), Value::Char('c')]` 这样的 Rust 
    同类型 (homogeneous) 集合可能当作 Serde 的异类型 (heterogeneous) 序列处理，这种情况下 
    Serde 的 bool 值之后是 Serde 的 char 值。
- **元组 (tuple)**
  - 具有固定大小的、异类型的一列值，其长度在反序列化期间无需查看序列化的数据就可知，
    比如 `(u8,)`、 `(String, u64, Vec<T>)`、 `[u64; 10]` 。
- **元组结构体 (tuple_struct)**
  - 有名字的元组，比如 `struct Rgb(u8, u8, u8)`
- **元组成员 (tuple_variant)**
  - 比如 `enum E { T(u8, u8) }` 中的 `E::T`
- **map**
  - 可变大小的异类型键-值对 (key-value pairing) ，比如 `BTreeMap<K, V>` 。
  - 序列化时，迭代所有数据之前可能知道也可能不知道长度。
  - 反序列化时，其长度必须查看序列化之后的数据才知道。
- **结构体 (struct)**
  - 具有固定大小、多种键-值对的类型，其中键 (keys) 是编译期常量字符串。
  - 反序列时无需查看序列化数据就可以知道键，比如 `struct S { r: u8, g: u8, b: u8 }` 。
- **结构体成员 (struct_variant)**
  - 比如 `enum E { S { r: u8, g: u8, b: u8 } }` 中的 `E::S` 。

[Understanding deserializer lifetimes]: lifetimes.md

## 映射到数据模型

把大部分 Rust 类型映射到数据模型是直截了当的 (straightforward) 。
比如 Rust 的 `bool` 类型就对应了 Serde 的 bool 类型；
Rust 的元组结构体 `Rgb(u8, u8, u8)` 对应 Serde 的元组结构体。

但是，没有理由认为这种映射必须是直截了当的。
[`Serialize`] 和 [`Deserialize`] traits 可以在 Rust 类型和 Serde 数据模型中做 **任何** 
映射，只要这种映射关系针对使用场景来说是合适的。

打个比方，想想 Rust 的 [`std::ffi::OsString`] 类型。
这种类型表示原生平台的字符串。
在 Unix 系统上，它是任意非零字节，在 Windows 系统上，它是任意非零 16 位的值。
把 `OsString` 映射成 Serde 数据模型中的以下一种类型再自然不过：

- 映射为 Serde 的 **string**：
  不巧，序列化时处理起来会很麻烦，因为 `OsString` 不保证以 UTF-8 方式呈现；
  反序列化也会很麻烦，因为 Serde strings 运行是 0 字节大小。
- 映射为 Serde 的 **byte array**：
  这解决了上面讨论的两种麻烦，但如果在 Unix 上序列化一个 `OsString` ，
  在 Windows 上反序列化它，就会得到一个 [错误的字符串][the wrong string] 。

所以，不采用上述做法，而是给 `OsString` 实现 `Serialize` 和 `Deserialize` 时，
把它作为 Serde 的 **枚举体** 进行映射。
如下给出了这种做法，即使这与任何单个平台上的定义都不太一样。

```rust
# #![allow(dead_code)]
#
enum OsString {
    Unix(Vec<u8>),
    Windows(Vec<u16>),
    // and other platforms
}
#
# fn main() {}
```

灵活地映射到 Serde 数据模型是影响深远的。
当实现 `Serialize` 和 `Deserialize` 时，
请了解清楚类型的背景，直觉地映射它并非最佳做法。

[`std::ffi::OsString`]: https://doc.rust-lang.org/std/ffi/struct.OsString.html
[the wrong string]: https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/
