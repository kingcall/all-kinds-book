[TOC]

> 翻译自：https://developers.google.com/protocol-buffers/docs/proto3



本指导描述了如何使用 protocol buffer 语言来构建 protocol buffer 数据，包括 `.proto` 文件语法和如何基于该 `.proto` 文件生成数据访问类。本文是涵盖 protocol buffer 语言 proto3 版本的内容，若需要 proto2 版本的信息，请参考 [Proto2 Language Guide](https://developers.google.com/protocol-buffers/docs/proto) 。

本文是语言指导——关于文中描述内容的分步示例，请参考所选编程语言的对应 [tutorial](https://developers.google.com/protocol-buffers/docs/tutorials) （当前仅提供了 proto2，更多 proto3 的内容会持续更新）。

## [#](http://www.hellokang.net/encoding/proto3.html#定义一个消息类型)定义一个消息类型

我们先看一个简单示例。比如说我们想定义个关于搜索请求的消息，每个搜索请求包含一个查询字符串，一个特定的页码，和每页的结果数量。下面是用于定义消息类型的 `.proto` 文件：

```protobuf
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```



- 文件的第一行指明了我们使用的是 proto3 语法：若不指定该行 protocol buffer 编译器会认为是 proto2 。该行必须是文件的第一个非空或非注释行。
- `SearchRequest` 消息定义了三个字段（名称/值对），字段就是每个要包含在该类型消息中的部分数据。每个字段都具有名称和类型 。

### [#](http://www.hellokang.net/encoding/proto3.html#指定字段类型)指定字段类型

上面的例子中，全部字段都是标量类型：两个整型（`page_number` 和 `result_per_page`）和一个字符串型（`query`）。同样，也可以指定复合类型的字段，包括枚举型和其他消息类型。

### [#](http://www.hellokang.net/encoding/proto3.html#分配字段编号)分配字段编号

正如你所见，消息中定义的每个字段都有一个**唯一编号**。字段编号用于在消息二进制格式中标识字段，同时要求消息一旦使用字段编号就不应该改变。注意一点 1 到 15 的字段编号需要用 1 个字节来编码，编码同时包括字段编号和字段类型（ 获取更多信息请参考 [Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding.html#structure) ）。16 到 2047 的字段变化使用 2 个字节。因此应将 1 到 15 的编号用在消息的常用字段上。注意应该为将来可能添加的常用字段预留字段编号。

最小的字段编号为 1，最大的为 2^29 - 1，或 536,870,911。注意不能使用 19000 到 19999 （`FieldDescriptor::kFirstReservedNumber` 到 `FieldDescriptor::kLastReservedNumber`）的字段编号，因为是 protocol buffer 内部保留的——若在 .proto 文件中使用了这些预留的编号 protocol buffer 编译器会发出警告。同样也不能使用之前[预留](http://www.hellokang.net/encoding/proto3.html)的字段编号。

### [#](http://www.hellokang.net/encoding/proto3.html#指定字段规则)指定字段规则

消息的字段可以是一下规则之一：

- singular ， 格式良好的消息可以有 0 个或 1 个该字段(但不能多于 1 个)。这是 proto3 语法的默认字段规则。
- repeated ，格式良好的消息中该字段可以重复任意次数（包括 0 次）。重复值的顺序将被保留。

在 proto3 中，标量数值类型的重复字段默认会使用 packed 压缩编码。

更多关于 packed 压缩编码的信息请参考 [Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding.html#packed) 。

### [#](http://www.hellokang.net/encoding/proto3.html#增加更多消息类型)增加更多消息类型

单个 .proto 文件中可以定义多个消息类型。这在定义相关联的多个消息中很有用——例如要定义与搜索消息`SearchRequest` 相对应的回复消息 `SearchResponse`，则可以在同一个 .proto 文件中增加它的定义：

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```



### [#](http://www.hellokang.net/encoding/proto3.html#增加注释)增加注释

使用 C/C++ 风格的 `//` 和 `/* ... */` 语法在 .proto 文件添加注释。

```protobuf
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```


[#](http://www.hellokang.net/encoding/proto3.html#保留字段)保留字段

在采取彻底删除或注释掉某个字段的方式来更新消息类型时，将来其他用户再更新该消息类型时可能会重用这个字段编号。后面再加载该 .ptoto 的旧版本时会引发好多问题，例如数据损坏，隐私漏洞等。一个防止该问题发生的办法是将删除字段的编号（或字段名称，字段名称会导致在 JSON 序列化时产生问题）设置为保留项 `reserved`。protocol buffer 编译器在用户使用这些保留字段时会发出警告。

```protobuf
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```



注意，不能在同一条 `reserved` 语句中同时使用字段编号和名称。

### [#](http://www.hellokang.net/encoding/proto3.html#proto-文件会生成什么？).proto 文件会生成什么？

当 protocol buffer 编译器作用于一个 .proto 文件时，编辑器会生成基于所选编程语言的关于 .proto 文件中描述消息类型的相关代码 ，包括对字段值的获取和设置，序列化消息用于输出流，和从输入流解析消息。

- 对于 **C++**， 编辑器会针对于每个 `.proto` 文件生成`.h` 和 `.cc` 文件，对于每个消息类型会生成一个类。
- 对于 **Java**, 编译器会生成一个 `.java` 文件和每个消息类型对应的类，同时包含一个特定的 `Builder`类用于构建消息实例。
- **Python** 有些不同 – Python 编译器会对于 .proto 文件中每个消息类型生成一个带有静态描述符的模块，以便于在运行时使用 metaclass 来创建必要的 Python 数据访问类。
- 对于 **Go**， 编译器会生成带有每种消息类型的特定数据类型的定义在`.pb.go` 文件中。
- 对于 **Ruby**，编译器会生成带有消息类型的 Ruby 模块的 `.rb` 文件。
- 对于**Objective-C**，编辑器会针对于每个 `.proto` 文件生成`pbobjc.h` 和 `pbobjc.m.` 文件，对于每个消息类型会生成一个类。
- 对于 **C#**，编辑器会针对于每个 `.proto` 文件生成`.cs` 文件，对于每个消息类型会生成一个类。
- 对于 **Dart**，编辑器会针对于每个 `.proto` 文件生成`.pb.dart` 文件，对于每个消息类型会生成一个类。

可以参考所选编程语言的教程了解更多 API 的信息。更多 API 详细信息，请参阅相关的 [API reference](https://developers.google.com/protocol-buffers/docs/reference/overview) 。

## [#](http://www.hellokang.net/encoding/proto3.html#标量数据类型)标量数据类型

消息标量字段可以是以下类型之一——下表列出了可以用在 .proto 文件中使用的类型，以及在生成代码中的相关类型：

| .proto Type | Notes                                                        | C++ Type | Java Type  | Python Type[2] | Go Type | Ruby Type                      | C# Type    | PHP Type          | Dart Type |
| ----------- | ------------------------------------------------------------ | -------- | ---------- | -------------- | ------- | ------------------------------ | ---------- | ----------------- | --------- |
| double      |                                                              | double   | double     | float          | float64 | Float                          | double     | float             | double    |
| float       |                                                              | float    | float      | float          | float32 | Float                          | float      | float             | double    |
| int32       | 使用变长编码。负数的编码效率较低——若字段可能为负值，应使用 sint32 代替。 | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| int64       | 使用变长编码。负数的编码效率较低——若字段可能为负值，应使用 sint64 代替。 | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| uint32      | 使用变长编码。                                               | uint32   | int[1]     | int/long[3]    | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       |
| uint64      | 使用变长编码。                                               | uint64   | long[1]    | int/long[3]    | uint64  | Bignum                         | ulong      | integer/string[5] | Int64     |
| sint32      | 使用变长编码。符号整型。负值的编码效率高于常规的 int32 类型。 | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| sint64      | 使用变长编码。符号整型。负值的编码效率高于常规的 int64 类型。 | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| fixed32     | 定长 4 字节。若值常大于2^28 则会比 uint32 更高效。           | uint32   | int[1]     | int/long[3]    | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       |
| fixed64     | 定长 8 字节。若值常大于2^56 则会比 uint64 更高效。           | uint64   | long[1]    | int/long[3]    | uint64  | Bignum                         | ulong      | integer/string[5] | Int64     |
| sfixed32    | 定长 4 字节。                                                | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| sfixed64    | 定长 8 字节。                                                | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| bool        |                                                              | bool     | boolean    | bool           | bool    | TrueClass/FalseClass           | bool       | boolean           | bool      |
| string      | 包含 UTF-8 和 ASCII 编码的字符串，长度不能超过 2^32 。       | string   | String     | str/unicode[4] | string  | String (UTF-8)                 | string     | string            | String    |
| bytes       | 可包含任意的字节序列但长度不能超过 2^32 。                   | string   | ByteString | str            | []byte  | String (ASCII-8BIT)            | ByteString | string            | List<int> |

可以在 [Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding) 中获取更多关于消息序列化时类型编码的相关信息。

> [1] Java 中，无符号 32 位和 64 位整数使用它们对应的符号整数表示，第一个 bit 位仅是简单地存储在符号位中。

> [2] 所有情况下，设置字段的值将执行类型检查以确保其有效。

> [3] 64 位或无符号 32 位整数在解码时始终表示为 long，但如果在设置字段时给出 int，则可以为 int。在所有情况下，该值必须适合设置时的类型。见 [2]。

> [4] Python 字符串在解码时表示为 unicode，但如果给出了 ASCII 字符串，则可以是 str（这条可能会发生变化）。

> [5] Integer 用于 64 位机器，string 用于 32 位机器。

## [#](http://www.hellokang.net/encoding/proto3.html#默认值)默认值

当解析消息时，若消息编码中没有包含某个元素，则相应的会使用该字段的默认值。默认值依据类型而不同：

- 字符串类型，空字符串
- 字节类型，空字节
- 布尔类型，false
- 数值类型，0
- 枚举类型，第一个枚举元素
- 内嵌消息类型，依赖于所使用的编程语言。参考 [generated code guide](https://developers.google.com/protocol-buffers/docs/reference/overview) 获取详细信息。

对于可重复类型字段的默认值是空的（ 通常是相应语言的一个空列表 ）。

注意一下标量字段，在消息被解析后是不能区分字段是使用默认值（例如一个布尔型字段是否被设置为 false ）赋值还是被设置为某个值的。例如你不能通过对布尔值等于 false 的判断来执行一个不希望在默认情况下执行的行为。同时还要注意若一个标量字段设置为默认的值，那么是不会被序列化以用于传输的。

查看 [generated code guide](https://developers.google.com/protocol-buffers/docs/reference/overview) 来获得更多关于编程语言生成代码的内容。

## [#](http://www.hellokang.net/encoding/proto3.html#枚举)枚举

定义消息类型时，可能需要某字段值是一些预设值之一。例如当需要在 `SearchRequest` 消息类型中增加一个 `corpus` 字段， `corpus` 字段的值可以是 `UNIVERSAL`， `WEB`，`IMAGES`， `LOCAL`， `NEWS`， `PRODUCTS` 或 `VIDEO`。仅仅需要在消息类型中定义带有预设值常量的 `enum` 类型即可完成上面的定义。

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```



如你所见，`Corpus` 枚举类型的第一个常量映射到 0 ：每个枚举的定义必须包含一个映射到 0 的常量作为第一个元素。原因是：

- 必须有一个 0 值，才可以作为数值类型的默认值。
- 0 值常量必须作为第一个元素，是为了与 proto2 的语义兼容就是第一个元素作为默认值。

将相同的枚举值分配给不同的枚举选项常量可以定义别名。要定义别名需要将 `allow_alisa` 选项设置为 `true`，否则 protocol 编译器当发现别名定义时会报错。

```protobuf
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```



枚举的常量值必须在 32 位整数的范围内。因为枚举值在传输时采用的是 varint 编码，同时负值无效因而不建议使用。可以如上面例子所示，将枚举定义在消息类型内，也可以将其定义外边——这样该枚举可以用在 .proto 文件中定义的任意的消息类型中以便重用。还可以使用 *MessageType*.*EnumType* 语法将枚举定义为消息字段的某一数据类型。

使用 protocol buffer 编译器编译 .proto 中的枚举时，对于 Java 或 C 会生成相应的枚举类型，对于 Python 会生成特定的 `EnumDescriptor` 类用于在运行时创建一组整型值符号常量即可。

反序列化时，未识别的枚举值会被保留在消息内，但如何表示取决于编程语言。若语言支持开放枚举类型允许范围外的值时，这些未识别的枚举值简单的以底层整型进行存储，就像 C++ 和 Go。若语言支持封闭枚举类型例如 Java，一种情况是使用特殊的访问器（译注：accessors）来访问底层的整型。无论哪种语言，序列化时的未识别枚举值都会被保留在序列化结果中。

更多所选语言中关于枚举的处理，请参考 [generated code guide](https://developers.google.com/protocol-buffers/docs/reference/overview) 。

### [#](http://www.hellokang.net/encoding/proto3.html#保留值)保留值

在采取彻底删除或注释掉某个枚举值的方式来更新枚举类型时，将来其他用户再更新该枚举类型时可能会重用这个枚举数值。后面再加载该 .ptoto 的旧版本时会引发好多问题，例如数据损坏，隐私漏洞等。一个防止该问题发生的办法是将删除的枚举数值（或名称，名称会导致在 JSON 序列化时产生问题）设置为保留项 `reserved`。protocol buffer 编译器在用户使用这些特定数值时会发出警告。可以使用 `max` 关键字来指定保留值的范围到最大可能值。

```protobuf
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```



注意不能在 `reserved` 语句中混用字段名称和数值。

## [#](http://www.hellokang.net/encoding/proto3.html#使用其他消息类型)使用其他消息类型

消息类型也可作为字段类型。例如，我们需要在 `SearchResponse` 消息中包含 `Result` 消息——想要做到这一点，可以将 `Result` 消息类型的定义放在同一个 .proto 文件中同时在 `SearchResponse` 消息中指定一个 `Result` 类型的字段：

```protobuf
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}	
```



### [#](http://www.hellokang.net/encoding/proto3.html#导入定义)导入定义

前面的例子中，我们将 `Result` 消息定义在了与 `SearchResponse` 相同的文件中——但若我们需要作为字段类型使用的消息类型已经定义在其他的 .proto 文件中了呢？

可以通过导入操作来使用定义在其他 .proto 文件中的消息定义。在文件的顶部使用 import 语句完成导入其他 .proto 文件中的定义：

```protobuf
import "myproject/other_protos.proto";
```



默认情况下仅可以通过直接导入 .proto 文件来使用这些定义。然而有时会需要将 .proto 文件移动位置。可以通过在原始位置放置一个伪 .proto 文件使用 `import public` 概念来转发对新位置的导入，而不是在发生一点更改时就去更新全部对旧文件的导入位置。任何导入包含 `import public` 语句的 proto 文件就会对其中的 `import public` 依赖产生传递依赖。例如：

```protobuf
// new.proto
// 全部定义移动到该文件
```



```protobuf
// old.proto
// 这是在客户端中导入的伪文件
import public "new.proto";
import "other.proto";
```



```protobuf
// client.proto
import "old.proto";
// 可使用 old.proto 和 new.proto 中的定义，但不能使用 other.proto 中的定义
```



protocol 编译器会使用命令行参数 `-I`/`--proto_path` 所指定的目录集合中检索需要导入的文件。若没有指定，会在调用编译器的目录中检索。通常应该将 `--proto_path` 设置为项目的根目录同时在 import 语句中使用全限定名。

### [#](http://www.hellokang.net/encoding/proto3.html#使用-proto2-类型)使用 proto2 类型

可以在 proto3 中导入 proto2 定义的消息类型，反之亦然。然而，proto2 中的枚举不能直接用在 proto3 语法中（但导入到 proto2 中 proto3 定义的枚举是可用的）。

## [#](http://www.hellokang.net/encoding/proto3.html#嵌套类型)嵌套类型

可以在一个消息类型中定义和使用另一个消息类型，如下例所示—— `Result` 消息类型定义在了 `SearchResponse` 消息类型中：

```protobuf
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```



使用 `Parent.Type` 语法可以在父级消息类型外重用内部定义消息类型：

```protobuf
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```



支持任意深度的嵌套：

```protobuf
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```



## [#](http://www.hellokang.net/encoding/proto3.html#消息类型的更新)消息类型的更新

如果现有的消息类型不再满足您的所有需求——例如，需要扩展一个字段——同时还要继续使用已有代码，别慌！ 在不破坏任何现有代码的情况下更新消息类型非常简单。仅仅遵循如下规则即可：

- 不要修改任何已有字段的字段编号
- 若是添加新字段，旧代码序列化的消息仍然可以被新代码所解析。应该牢记新元素的默认值以便于新代码与旧代码序列化的消息进行交互。类似的，新代码序列化的消息同样可以被旧代码解析：旧代码解析时会简单的略过新字段。参考未知字段获取详细信息。
- 字段可被移除，只要不再使用移除字段的字段编号即可。可能还会对字段进行重命名，或许是增加前缀 `OBSOLETE_` ，或保留字段编号以保证后续不能重用该编号。
- `int32`， `uint32`， `int64`， `uint64`， 和 `bool` 是完全兼容的——意味着可以从这些字段其中的一个更改为另一个而不破坏前后兼容性。若解析出来的数值与相应的类型不匹配，会采用与 C++ 一致的处理方案（例如，若将 64 位整数当做 32 位进行读取，则会被转换为 32 位）。
- `sint32` 和`sint64` 相互兼容但不与其他的整型兼容。
- `string` and `bytes` 在合法 UTF-8 字节前提下也是兼容的。
- 嵌套消息与 `bytes` 在 bytes 包含消息编码版本的情况下也是兼容的。
- `fixed32` 与`sfixed32` 兼容， `fixed64` 与 `sfixed64`兼容。
- `enum` 与 `int32`，`uint32`， `int64`，和 `uint64` 兼容（注意若值不匹配会被截断）。但要注意当客户端反序列化消息时会采用不同的处理方案：例如，未识别的 proto3 枚举类型会被保存在消息中，但是当消息反序列化时如何表示是依赖于编程语言的。整型字段总是会保持其的值。
- 将一个单独值更改为新 `oneof` 类型成员之一是安全和二进制兼容的。 若确定没有代码一次性设置多个值那么将多个字段移入一个新 `oneof` 类型也是可行的。将任何字段移入已存在的 `oneof` 类型是不安全的。

## [#](http://www.hellokang.net/encoding/proto3.html#未知字段)未知字段

未知字段是解析结构良好的 protocol buffer 已序列化数据中的未识别字段的表示方式。例如，当旧程序解析带有新字段的数据时，这些新字段就会成为旧程序的未知字段。

本来，proto3 在解析消息时总是会丢弃未知字段，但在 3.5 版本中重新引入了对未知字段的保留机制以用来兼容 proto2 的行为。在 3.5 或更高版本中，未知字段在解析时会被保留同时也会包含在序列化结果中。

## [#](http://www.hellokang.net/encoding/proto3.html#any-类型)Any 类型

Any 类型允许我们将没有 .proto 定义的消息作为内嵌类型来使用。一个 `Any` 包含一个类似 `bytes` 的任意序列化消息，以及一个 URL 来作为消息类型的全局唯一标识符。要使用 `Any` 类型，需要导入 `google/protobuf/any.proto`。

```protobuf
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```



对于给定的消息类型的默认 URL 为 `type.googleapis.com/packagename.messagename` 。

不同的语言实现会支持运行时的助手函数来完成类型安全地 `Any` 值的打包和拆包工作——例如，Java 中，`Any` 类型会存在特定的 `pack()` 和 `unpack()` 访问器，而 C++ 中会是 `PackFrom()` 和 `UnpackTo()` 方法：

```protobuf
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```



**当前处理 Any 类型的运行库正在开发中**

若你已经熟悉了 proto2 语法，Any 类型的位于 [extensions](https://developers.google.com/protocol-buffers/docs/proto#extensions) 部分。

## [#](http://www.hellokang.net/encoding/proto3.html#oneof)Oneof

若一个含有多个字段的消息同时大多数情况下一次仅会设置一个字段，就可以使用 oneof 特性来强制该行为同时节约内存。

Oneof 字段除了全部字段位于 oneof 共享内存以及大多数情况下一次仅会设置一个字段外与常规字段类似。对任何oneof 成员的设置会自动清除其他成员。可以通过 `case()` 或 `WhichOneof()` 方法来检测 oneof 中的哪个值被设置了，这个需要基于所选的编程语言。

### [#](http://www.hellokang.net/encoding/proto3.html#使用-oneof)使用 oneof

使用 `oneof` 关键字在 .proto 文件中定义 oneof，同时需要跟随一个 oneof 的名字，就像本例中的 `test_oneof`：

```protobuf
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```



然后将字段添加到 oneof 的定义中。可以增加任意类型的字段，但不能使用 **repeated** 字段。

在生成的代码中，oneof 字段和常规字段一致具有 getters 和 setters 。同时也会获得一个方法以用于检测哪个值被设置了。更多所选编程语言中关于 oneof 的 API 可以参考 [API reference](https://developers.google.com/protocol-buffers/docs/reference/overview) 。

### [#](http://www.hellokang.net/encoding/proto3.html#oneof-特性)Oneof 特性

- 设置 oneof 的一个字段会清除其他字段。因此入设置了多次 oneof 字段，仅最后设置的字段生效。

```protobuf
SampleMessage message;
message.set_name("name");
CHECK(message.has_name());
message.mutable_sub_message();   // 会清理 name 字段
CHECK(!message.has_name());
```

1
2
3
4
5

- 若解析器在解析得到的数据时碰到了多个 oneof 的成员，最后一个碰到的是最终结果。
- oneof 不能是 `repeated`。
- 反射 API 可作用于 oneof 字段。
- 若将一个 oneof 字段设为了默认值（就像为 int32 类型设置了 0 ），那么 oneof 字段会被设置为 "case"，同时在序列化编码时使用。
- 若使用 C++ ，确认代码不会造成内存崩溃。以下的示例代码就会导致崩溃，因为 `sub_message` 在调用 `set_name()` 时已经被删除了。

```protobuf
SampleMessage message;
SubMessage* sub_message = message.mutable_sub_message();
message.set_name("name");      // 会删除 sub_message
sub_message->set_...     		// 此处会崩溃
```



- 同样在 C++ 中，若 `Swap()` 两个 oneof 消息，那么消息会以另一个消息的 oneof 的情况：下例中，`msg1`会是 `sub_message1` 而 `msg2` 中会是 `name`。

```protobuf
SampleMessage msg1;
msg1.set_name("name");
SampleMessage msg2;
msg2.mutable_sub_message();
msg1.swap(&msg2);
CHECK(msg1.has_sub_message());
CHECK(msg2.has_name());
```

### [#](http://www.hellokang.net/encoding/proto3.html#向后兼容问题)向后兼容问题

在添加或删除 oneof 字段时要当心。若检测到 oneof 的值是 `None`/`NOT_SET`，这意味着 oneof 未被设置或被设置为一个不同版本的 oneof 字段。没有方法可以区分，因为无法确定一个未知字段是否是 oneof 的成员。

**标记重用问题**

- **移入或移出 oneof 字段**： 消息序列化或解析后，可能会丢失一些信息（某些字段将被清除）。然而，可以安全地将单个字段移入新的 oneof 中，同样若确定每次操作只有一个字段被设置则可以移动多个字段。
- **删除一个 oneof 字段并又将其加回**： 消息序列化和解析后，可能会清除当前设置的 oneof 字段。
- **拆分或合并 oneof**：这与移动常规字段有类似的问题。

## [#](http://www.hellokang.net/encoding/proto3.html#map-映射表)Map 映射表

若需要创建关联映射表作为定义的数据的一部分，protocol buffers 提供了方便的快捷语法：

```protobuf
map<key_type, value_type> map_field = N;
```

`key_type` 处可以是整型或字符串类型（其实是除了 float 和 bytes 类型外任意的标量类型）。注意枚举不是合法的 `key_type` 。`value_type` 是除了 map 外的任意类型。

例如，若需要创建每个项目与一个字符串 key 相关联的映射表，可以采用下面的定义：

```protobuf
map<string, Project> projects = 3;
```

- 映射表字段不能为 `repeated`
- 映射表的编码和迭代顺序是未定义的，因此不能依赖映射表元素的顺序来操作。
- 当基于 .proto 生成文本格式时，映射表的元素基于 key 来排序。数值型的 key 基于数值排序。
- 当解析或合并时，若出现冲突的 key 以最后一个 key 为准。当从文本格式解析时，若 key 冲突则会解析失败。
- 若仅仅指定了映射表中某个元素的 key 而没有指定 value，当序列化时的行为是依赖于编程语言。在 C++，Java，和 Python 中使用类型的默认值来序列化，但在有些其他语言中可能不会序列化任何东西。

生成的映射表 API 当前可用于全部支持 proto3 的编程语言。在 [API reference](https://developers.google.com/protocol-buffers/docs/reference/overview) 中可以获取更多关于映射表 API 的内容。

### [#](http://www.hellokang.net/encoding/proto3.html#向后兼容问题-2)向后兼容问题

映射表语法与以下代码是对等的，因此 protocol buffers 的实现即使不支持映射表也可以正常处理数据：

```protobuf
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```



任何支持映射表的 protocol buffers 实现都必须同时处理和接收上面代码的数据定义。

## [#](http://www.hellokang.net/encoding/proto3.html#包)包

可以在 .proto 文件中使用 `package` 指示符来避免 protocol 消息类型间的命名冲突。

```protobuf
package foo.bar;
message Open { ... }
```



这样在定义消息的字段类型时就可以使用包指示符来完成：

```protobuf
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```



包指示符的处理方式是基于编程语言的：

- C++ 中生成的类位于命名空间中。例如，`Open` 会位于命名空间 `foo::bar` 中。
- Java 中，使用 Java 的包，除非在 .proto 文件中使用 `option java_pacakge` 做成明确的指定。
- Python 中，package 指示符被忽略，这是因为 Python 的模块是基于文件系统的位置来组织的。
- Go 中，作为 Go 的包名来使用，除非在 .proto 文件中使用 `option java_pacakge` 做成明确的指定。
- Ruby 中，生成的类包裹于 Ruby 的命名空间中，还要转换为 Ruby 所需的大小写风格（首字母大写；若首字符不是字母，则使用 `PB_` 前缀）。例如，`Open` 会位于命名空间 `Foo::Bar` 中。
- C# 中作为命名空间来使用，同时需要转换为 PascalCase 风格，除非在 .proto 使用 `option csharp_namespace` 中明确的指定。例如，`Open` 会位于命名空间 `Foo.Bar` 中。

### [#](http://www.hellokang.net/encoding/proto3.html#包和名称解析)包和名称解析

protocol buffer 中类型名称解析的工作机制类似于 C++ ：先搜索最内层作用域，然后是次内层，以此类推，每个包被认为是其外部包的内层。前导点（例如，`.foo.bar.Baz`）表示从最外层作用域开始。

protocol buffer 编译器会解析导入的 .proto 文件中的全部类型名称。基于编程语言生成的代码也知道如何去引用每种类型，即使编程语言有不同的作用域规则。

## [#](http://www.hellokang.net/encoding/proto3.html#定义服务)定义服务

若要在 RPC （Remote Procedure Call，远程过程调用）系统中使用我们定义的消息类型，则可在 .proto 文件中定义这个 RPC 服务接口，同时 protocol buffer 编译器会基于所选编程语言生成该服务接口代码。例如，若需要定义一个含有可以接收 `SearchRequest` 消息并返回 `SearchResponse` 消息方法的 RPC 服务，可以在 .proto 文件中使用如下代码定义：

```protobuf
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

1
2
3

最直接使用 protocal buffer 的 RPC 系统是 [gRPC](https://grpc.io/) ：一款 Google 开源，语言和平台无关的 RPC 系统。gRPC 对 protocol buffer 的支持非常好同时允许使用特定的 protocol buffer 编译器插件来基于 .proto 文件生成相关的代码。

若不想使用 gRPC，同样可以在自己的 RPC 实现上使用 protocol buffer。可以在 [Proto2 Language Guide](https://developers.google.com/protocol-buffers/docs/proto#services) 处获得更多关于这方面的信息。

同样也有大量可用的第三方使用 protocol buffer 的项目。对于我们了解的相关项目列表，请参考 [third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md) 。

## [#](http://www.hellokang.net/encoding/proto3.html#json-映射)JSON 映射

Proto3 支持 JSON 的规范编码，这使得系统间共享数据变得更加容易。下表中，将逐类型地描述这些编码。

若 JSON 编码中不存在某个值或者值为 null，当将其解析为 protocol buffer 时会解析为合适的默认值。若 procol buffer 中使用的是字段的默认值，则默认情况下 JSON 编码会忽略该字段以便于节省空间。实现上应该提供一个选项以用来将具有默认值的字段生成在 JSON 编码中。

| proto3                 | JSON          | JSON 示例                                | 说明                                                         |
| ---------------------- | ------------- | ---------------------------------------- | ------------------------------------------------------------ |
| message                | object        | `{"fooBar": v, "g": null,…}`             | 生成 JSON 对象。消息字段名映射为对象的 lowerCamelCase（译著：小驼峰） 的 key。若指定了 `json_name` 选项，则使用该选项值作为 key。解析器同时支持 lowerCamelCase 名称（或 `json_name` 指定名称）和原始 proto 字段名称。全部类型都支持 `null` 值，是当做对应类型的默认值来对待的。 |
| enum                   | string        | `"FOO_BAR"`                              | 使用 proto 中指定的枚举值的名称。解析器同时接受枚举名称和整数值。 |
| map<K,V>               | object        | `{"k": v, …}                             | 所有的 key 被转换为字符串类型。                              |
| repeated V             | array         | `[v, …]`                                 | `null` 被解释为空列表 []。                                   |
| bool                   | true, false   | `true, false`                            |                                                              |
| string                 | string        | `"Hello World!"`                         |                                                              |
| bytes                  | base64 string | `"YWJjMTIzIT8kKiYoKSctPUB+"`             | JSON 值是使用标准边界 base64 编码的字符串。不论标准或 URL 安全还是携带边界与否的 base64 编码都支持。 |
| int32, fixed32, uint32 | number        | `1, -10, 0`                              | JSON 值是 10 进制数值。数值或字符串都可以支持。              |
| int64, fixed64, uint64 | string        | `"1", "-10"`                             | JSON 值是 10 进制字符串。数值或字符串都支持。                |
| float, double          | number        | `1.1, -10.0, 0, "NaN","Infinity"`        | JSON 值是数值或特定的字符串之一："NaN"，"Infinity" 和 "-Infinity" 。数值和字符串都支持。指数表示法同样支持。 |
| Any                    | `object`      | `{"@type": "url", "f": v, … }`           | 若 Any 类型包含特定的 JSON 映射值，则会被转换为下面的形式： `{"@type": xxx, "value": yyy}`。否则，会被转换到一个对象中，同时会插入一个 `"@type"` 元素用以指明实际的类型。 |
| Timestamp              | string        | `"1972-01-01T10:00:20.021Z"`             | 采用 RFC 3339 格式，其中生成的输出总是 Z规范的，并使用 0、3、6 或 9 位小数。除 `“Z”` 以外的偏移量也可以。 |
| Duration               | string        | `"1.000340012s", "1s"`                   | 根据所需的精度，生成的输出可能会包含 0、3、6 或 9 位小数，以 “s” 为后缀。只要满足纳秒精度和后缀 “s” 的要求，任何小数（包括没有）都可以接受。 |
| Struct                 | `object`      | `{ … }`                                  | 任意 JSON 对象。参见 `struct.proto`.                         |
| Wrapper types          | various types | `2, "2", "foo", true,"true", null, 0, …` | 包装器使用与包装的原始类型相同的 JSON 表示，但在数据转换和传输期间允许并保留 null。 |
| FieldMask              | string        | `"f.fooBar,h"`                           | 参见`field_mask.proto`。                                     |
| ListValue              | array         | `[foo, bar, …]`                          |                                                              |
| Value                  | value         |                                          | Any JSON value                                               |
| NullValue              | null          |                                          | JSON null                                                    |
| Empty                  | object        | {}                                       | 空 JSON 对象                                                 |

### [#](http://www.hellokang.net/encoding/proto3.html#json-选项)JSON 选项

proto3 的 JSON 实现可以包含如下的选项：

- **省略使用默认值的字段**：默认情况下，在 proto3 的 JSON 输出中省略具有默认值的字段。该实现可以使用选项来覆盖此行为，来在输出中保留默认值字段。
- **忽略未知字段**：默认情况下，proto3 的 JSON 解析器会拒绝未知字段，同时提供选项来指示在解析时忽略未知字段。
- **使用 proto 字段名称代替 lowerCamelCase 名称**： 默认情况下，proto3 的 JSON 编码会将字段名称转换为 lowerCamelCase（译著：小驼峰）形式。该实现提供选项可以使用 proto 字段名代替。Proto3 的 JSON 解析器可同时接受 lowerCamelCase 形式 和 proto 字段名称。
- **枚举值使用整数而不是字符串表示**： 在 JSON 编码中枚举值是使用枚举值名称的。提供了可以使用枚举值数值形式来代替的选项。

## [#](http://www.hellokang.net/encoding/proto3.html#选项)选项

.proto 文件中的单个声明可以被一组选项来设置。选项不是用来更改声明的含义，但会影响在特定上下文下的处理方式。完整的选项列表定义在 `google/protobuf/descriptor.proto` 中。

有些选项是文件级的，意味着可以卸载顶级作用域，而不是在消息、枚举、或服务的定义中。有些选项是消息级的，意味着需写在消息的定义中。有些选项是字段级的，意味着需要写在字段的定义内。选项还可以写在枚举类型，枚举值，服务类型，和服务方法上；然而，目前还没有任何可用于以上位置的选项。

下面是几个最常用的选项：

- `java_package` （文件选项）：要用在生成 Java 代码中的包。若没有在 .proto 文件中对 `java_package` 选项做设置，则会使用 proto 作为默认包（在 .proto 文件中使用 "package" 关键字设置）。 然而，proto 包通常不是合适的 Java 包，因为 proto 包通常不以反续域名开始。若不生成 Java 代码，则此选项无效。

```protobuf
option java_package = "com.example.foo";
```

- java_multiple_files （文件选项）：导致将顶级消息、枚举、和服务定义在包级，而不是在以 .proto 文件命名的外部类中。

```protobuf
option java_multiple_files = true;
```

- java_outer_classname（文件选项）：想生成的最外层 Java 类（也就是文件名）。若没有在 .proto 文件中明确指定 `java_outer_classname` 选项，类名将由 .proto 文件名转为 camel-case 来构造（因此 `foo_bar.proto` 会变为 `FooBar.java`）。若不生成 Java 代码，则此选项无效。

```protobuf
option java_outer_classname = "Ponycopter";
```

- optimize_for （文件选项）： 可被设为 `SPEED`， `CODE_SIZE`，或 `LITE_RUNTIME`。这会影响 C++ 和 Java 代码生成器（可能包含第三方生成器） 的以下几个方面：
- SPEED （默认）： protocol buffer 编译器将生成用于序列化、解析和消息类型常用操作的代码。生成的代码是高度优化的。
- CODE_SIZE ：protocol buffer 编译器将生成最小化的类，并依赖于共享的、基于反射的代码来实现序列化、解析和各种其他操作。因此，生成的代码将比 SPEED 模式小的多，但操作将变慢。类仍将实现与 SPEED 模式相同的公共 API。这种模式在处理包含大量 .proto 文件同时不需要所有操作都要求速度的应用程序中最有用。
- LITE_RUNTIME ：protocol buffer 编译器将生成仅依赖于 “lite” 运行库的类（libprotobuf-lite 而不是libprotobuf）。lite 运行时比完整的库小得多（大约小一个数量级），但会忽略某些特性，比如描述符和反射。这对于在受限平台（如移动电话）上运行的应用程序尤其有用。编译器仍然会像在 SPEED 模式下那样生成所有方法的快速实现。生成的类将仅用每种语言实现 MessageLite 接口，该接口只提供 `Message` 接口的一个子集。

```protobuf
option optimize_for = CODE_SIZE;	
```

- `cc_enable_arenas`（文件选项）：为生成的 C++ 代码启用 [arena allocation](https://developers.google.com/protocol-buffers/docs/reference/arenas) 。
- `objc_class_prefix` （文件选项）： 设置当前 .proto 文件生成的 Objective-C 类和枚举的前缀。没有默认值。你应该使用 [recommended by Apple](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4) 的 3-5 个大写字母作为前缀。注意所有 2 个字母前缀都由 Apple 保留。
- `deprecated` （字段选项）：若设置为 `true`， 指示该字段已被废弃，新代码不应使用该字段。在大多数语言中，这没有实际效果。在 Java 中，这变成了一个 `@Deprecated` 注释。将来，其他语言的代码生成器可能会在字段的访问器上生成弃用注释，这将导致在编译试图使用该字段的代码时发出警告。如果任何人都不使用该字段，并且您希望阻止新用户使用它，那么可以考虑使用保留语句替换字段声明。

```text
int32 old_field = 6 [deprecated=true];
```

### [#](http://www.hellokang.net/encoding/proto3.html#自定义选项)自定义选项

protocol buffer 还允许使用自定义选项。大多数人都不需要此高级功能。若确认要使用自定义选项，请参阅 [Proto2 Language Guide](https://developers.google.com/protocol-buffers/docs/proto.html#customoptions) 了解详细信息。注意使用 [extensions](https://developers.google.com/protocol-buffers/docs/proto.html#extensions) 来创建自定义选项，只允许用于 proto3 中。

## [#](http://www.hellokang.net/encoding/proto3.html#生成自定义类)生成自定义类

若要生成操作 .proto 文件中定义的消息类型的 Java、Python、C++、Go、Ruby、Objective-C 或 C# 代码，需要对 .proto 文件运行 protocol buffer 编译器 `protoc`。若还没有安装编译器，请 [download the package](https://developers.google.com/protocol-buffers/docs/downloads.html) 并依据 README 完成安装。对于 Go ，还需要为编译器安装特定的代码生成器插件：可使用 GitHub 上的 [golang/protobuf](https://github.com/golang/protobuf/) 库。

Protocol buffer 编译器的调用方式如下:

```text
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

1

- `IMPORT_PATH` 为`import` 指令检索 .proto 文件的目录。若未指定，使用当前目录。多个导入目录可以通过多次传递 `--proto_path` 选项实现；这些目录会依顺序检索。 `-I=*IMPORT_PATH*` 可作为 `--proto_path` 的简易格式使用。
- 可以提供一个或多个输出指令：
- `--cpp_out` 在 `DST_DIR`目录 生成 C++ 代码。参阅 [C++ generated code reference](https://developers.google.com/protocol-buffers/docs/reference/cpp-generated) 获取更多信息。
- `--java_out` 在 `DST_DIR`目录 生成 Java 代码。参阅 [Java generated code reference](https://developers.google.com/protocol-buffers/docs/reference/java-generated) 获取更多信息。
- `--python_out`在 `DST_DIR`目录 生成 Python代码。参阅 [Python generated code reference](https://developers.google.com/protocol-buffers/docs/reference/python-generated) 获取更多信息。
- `--go_out` 在 `DST_DIR`目录 生成 Go 代码。参阅 [Go generated code reference](https://developers.google.com/protocol-buffers/docs/reference/go-generated) 获取更多信息。
- `--ruby_out` 在 `DST_DIR`目录 生成 Ruby 代码。 coming soon!
- `--objc_out` 在 `DST_DIR`目录 生成 Objective-C 代码。参阅 [Objective-C generated code reference](https://developers.google.com/protocol-buffers/docs/reference/objective-c-generated) 获取更多信息。
- `--csharp_out` 在 `DST_DIR`目录 生成 C# 代码。参阅 [C# generated code reference](https://developers.google.com/protocol-buffers/docs/reference/csharp-generated) 获取更多信息。
- `--php_out` 在 `DST_DIR`目录 生成 PHP代码。参阅 [PHP generated code reference](https://developers.google.com/protocol-buffers/docs/reference/php-generated) 获取更多信息。

作为额外的便利，若 DST_DIR 以 `.zip` 或 `.jar` 结尾，编译器将会写入给定名称的 ZIP 格式压缩文件，`.jar` 还将根据 Java JAR 的要求提供一个 manifest 文件。请注意，若输出文件已经存在，它将被覆盖；编译器还不够智能，无法将文件添加到现有的存档中。

- 必须提供一个或多个 .proto 文件作为输入。可以一次指定多个 .proto 文件。虽然这些文件是相对于当前目录命名的，但是每个文件必须驻留在 `IMPORT_PATHs` 中，以便编译器可以确定它的规范名称。