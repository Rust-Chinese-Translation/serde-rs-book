# 属性

[属性][Attributes] (attributes) 用来对 Serde derive 宏生成的
`Serialize` 和 `Deserialize` 实现进行自定义。
这里的属性要求 Rust 编译器版本至少为 1.15 。

[Attributes]: https://doc.rust-lang.org/book/attributes.html

有三种属性：

- [**container 属性**] — 用在结构体或枚举体声明上
- [**variant 属性**] — 用在枚举体的成员上
- [**field 属性**] — 用在结构体字段上，或者枚举体成员上

[**container 属性**]: container-attrs.md
[**variant 属性**]: variant-attrs.md
[**field 属性**]: field-attrs.md

```rust
# use serde::{Serialize, Deserialize};
#
#[derive(Serialize, Deserialize)]
#[serde(deny_unknown_fields)]  // <-- this is a container attribute
struct S {
    #[serde(default)]  // <-- this is a field attribute
    f: i32,
}

#[derive(Serialize, Deserialize)]
#[serde(rename = "e")]  // <-- this is also a container attribute
enum E {
    #[serde(rename = "a")]  // <-- this is a variant attribute
    A(String),
}
#
# fn main() {}
```

注意：单个结构体、枚举体、成员或者字段都可能有多个属性。
