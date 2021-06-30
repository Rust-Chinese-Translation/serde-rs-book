# container attributes

<a id="rename"></a>
- ##### `#[serde(rename = "name")]` 

  序列化/反序列化这个结构体/枚举体时，使用给定的名称，而不是 Rust 里的名称。

  可以分别给序列化/反序列化设置数据结构的名称：

  - `#[serde(rename(serialize = "ser_name"))]`
  - `#[serde(rename(deserialize = "de_name"))]`
  - `#[serde(rename(serialize = "ser_name", deserialize = "de_name"))]`

<a id="rename_all"></a>
- ##### `#[serde(rename_all = "...")]` 

  根据给定的大小写转换规则，给所有结构体的字段或枚举体的成员重命名。

  可填入的值有：
  `"lowercase"`, `"UPPERCASE"`, `"PascalCase"`, `"camelCase"`, `"snake_case"`,
  `"SCREAMING_SNAKE_CASE"`, `"kebab-case"`, `"SCREAMING-KEBAB-CASE"`

  可以分别给序列化/反序列化设置名称大小写：

  - `#[serde(rename_all(serialize = "..."))]`
  - `#[serde(rename_all(deserialize = "..."))]`
  - `#[serde(rename_all(serialize = "...", deserialize = "..."))]`

<a id="deny_unknown_fields"></a>
- ##### `#[serde(deny_unknown_fields)]` 

  
  Always error during deserialization when encountering unknown fields. When
  this attribute is not present, by default unknown fields are ignored for
  self-describing formats like JSON.

  *Note:* this attribute is not supported in combination with [`flatten`],
  neither on the outer struct nor on the flattened field.

  [`flatten`]: field-attrs.md#flatten

<a id="tag"></a>
- ##### `#[serde(tag = "type")]` 

  Use the internally tagged enum representation for this enum, with the given
  tag. See [enum representations](enum-representations.md) for details on this
  representation.

<a id="tag--content"></a>
- ##### `#[serde(tag = "t", content = "c")]` 

  Use the adjacently tagged enum representation for this enum, with the given
  field names for the tag and content. See [enum
  representations](enum-representations.md) for details on this representation.

<a id="untagged"></a>
- ##### `#[serde(untagged)]` 

  Use the untagged enum representation for this enum. See [enum
  representations](enum-representations.md) for details on this representation.

<a id="bound"></a>
- ##### `#[serde(bound = "T: MyTrait")]` 

  Where-clause for the `Serialize` and `Deserialize` impls. This replaces any
  trait bounds inferred by Serde.

  Allows specifying independent bounds for serialization vs deserialization:

  - `#[serde(bound(serialize = "T: MySerTrait"))]`
  - `#[serde(bound(deserialize = "T: MyDeTrait"))]`
  - `#[serde(bound(serialize = "T: MySerTrait", deserialize = "T: MyDeTrait"))]`

<a id="default"></a>
- ##### `#[serde(default)]` 

  When deserializing, any missing fields should be filled in from the struct's
  implementation of `Default`. Only allowed on structs.

<a id="default--path"></a>
- ##### `#[serde(default = "path")]` 

  When deserializing, any missing fields should be filled in from the object
  returned by the given function or method. The function must be callable as
  `fn() -> T`. For example `default = "my_default"` would invoke `my_default()`
  and `default = "SomeTrait::some_default"` would invoke
  `SomeTrait::some_default()`. Only allowed on structs.

<a id="remote"></a>
- ##### `#[serde(remote = "...")]` 

  This is used for deriving `Serialize` and `Deserialize` for [remote
  types](remote-derive.md).

<a id="transparent"></a>
- ##### `#[serde(transparent)]` 

  Serialize and deserialize a newtype struct or a braced struct with one field
  exactly the same as if its one field were serialized and deserialized by
  itself. Analogous to `#[repr(transparent)]`.

<a id="from"></a>
- ##### `#[serde(from = "FromType")]` 

  Deserialize this type by deserializing into `FromType`, then converting. This
  type must implement `From<FromType>`, and `FromType` must implement
  `Deserialize`.

<a id="try_from"></a>
- ##### `#[serde(try_from = "FromType")]` 

  Deserialize this type by deserializing into `FromType`, then converting
  fallibly. This type must implement `TryFrom<FromType>` with an error type that
  implements `Display`, and `FromType` must implement `Deserialize`.

<a id="into"></a>
- ##### `#[serde(into = "IntoType")]` 

  Serialize this type by converting it into the specified `IntoType` and
  serializing that. This type must implement `Clone` and `Into<IntoType>`, and
  `IntoType` must implement `Serialize`.

<a id="crate"></a>
- ##### `#[serde(crate = "...")]` 

  Specify a path to the `serde` crate instance to use when referring to Serde
  APIs from generated code. This is normally only applicable when invoking
  re-exported Serde derives from a public macro in a different crate.
