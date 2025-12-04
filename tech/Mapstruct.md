### 文档概述
本文档详细介绍了在Mapstruct中如何为特定字段应用条件逻辑处理，特别是数值字段小于0时设为null的场景。文档涵盖三种不同情况的处理方式：数值型字段处理、字符串字段特殊值处理，以及使用@Named注解精确控制特定字段的转换逻辑。通过具体代码示例展示了如何实现精准的字段级条件验证。

### 使用场景
- **数值验证**：为特定数值字段（如age、score、count）添加小于0时设为null的验证
- **精确控制**：使用@Named注解只为某些特定字段应用条件逻辑，其他同类型字段不受影响
- **字符串处理**：处理字符串字段中的特殊值（如"-1"）转换为null
- **字段级验证**：在对象映射过程中对特定字段进行条件验证和转换
- **类型安全转换**：确保数值类型字段在映射过程中的数据有效性

---

### 情况一：你实际想处理的是**数值型字段**（比如 `age`、`score`、`count` 等），但误写成了 `name`

✅ 那你可以这样指定字段：

```java
@Mapper
public interface MyMapper {

    MyMapper INSTANCE = Mappers.getMapper(MyMapper.class);

    @Mapping(source = "score", target = "score") // 显式声明（其实可省略）
    Target map(Source source);

    // 仅对 score 字段生效
    default Integer mapScoreToNullIfNegative(Integer score) {
        return (score != null && score < 0) ? null : score;
    }
}
```

只要 `mapScoreToNullIfNegative` 的 **输入和输出类型** 与 `source.score` → `target.score` 匹配，MapStruct 就会自动用于该字段。

---

### 情况二：你真的有一个叫 `name` 的 **数值字段**（比如 `int name` —— 虽不常见，但可能）

那你同样可以：

```java
default Integer mapName(Integer name) {
    return (name != null && name < 0) ? null : name;
}
```

MapStruct 会根据**类型 + 字段名上下文**自动选用此方法处理 `name` 字段（前提是源和目标都有 `Integer name` 字段）。

---

### 情况三：`name` 是 `String`，但你想在值为某些内容时设为 `null`

比如：“如果 name 是 '-1'，则设为 null”，那逻辑要调整：

```java
default String mapName(String name) {
    return "-1".equals(name) ? null : name;
}
```

---

### 关键点总结：

- MapStruct **通过方法签名（参数+返回类型）自动绑定**到对应字段。
- 如果多个字段类型相同（如多个 `Integer`），MapStruct **默认会复用同一个方法**。
- 若你只想对**特定字段**应用逻辑（比如只对 `score` 而不是 `age`），建议：
 - 使用 `@Named` 注解 + `qualifiedByName` 显式指定。

#### 示例：用 `@Named` 精确控制字段

```java
@Mapper
public interface MyMapper {

    MyMapper INSTANCE = Mappers.getMapper(MyMapper.class);

    @Mapping(source = "score", target = "score", qualifiedByName = "toNullIfNegative")
    @Mapping(source = "age", target = "age") // 不处理 age
    Target map(Source source);

    @Named("toNullIfNegative")
    default Integer toNullIfNegative(Integer value) {
        return (value != null && value < 0) ? null : value;
    }
}
```

这样就**只对 `score` 字段**应用该逻辑，`age` 字段不受影响。

---

✅ 所以回答你的问题：**可以指定字段（如 name 或其他字段名）应用该逻辑**，根据字段类型选择合适方式，推荐使用 `@Named` + `qualifiedByName` 实现精准控制。

如果你能提供具体的字段名和类型（比如 `source.negativeValue` → `target.amount`，类型是 `Integer`），我可以给你更精确的代码。

---

[← 返回文档首页](../index.md)
