ES 实现的更新模式都是 Append 更新，即追加新文档数据，标记旧数据删除。不同于 MySQL 等 B 树实现的关系型数据库，直接操作数据页来更新，即原地更新。
# Index 与 Upsert

Index 与 Upsert 的更新模式都是 Append 更新。Index 是原文档覆盖更新，Upsert 是指定属性更新，即 copy 原文档数据，更新属性，Append data。

两者的性能考量：

- Index：全量替换文档模式，需要指定整个文档 data。此时需要考虑这个文档的大小，网络 IO，序列化，缓冲区这些都会影响到更新文档的性能。

- Upsert：追加字段更新模式，数据占用稍优于 Index，主要消耗在于 copy 文档的旧字段。通过指定属性 `doc_as_upsert = true` 实现。

推荐使用 Upsert，幂等性高，性能较优，并发更新时可指定 retry_on_conflict 属性。

# 原地更新与 Append 更新

Append 更新：追加写入，后台异步合并数据段。写入性能高，需要维护和读取多版本数据，读性能较差，短时间存储放大。

原地更新：直接在数据页上更新，存储占有稍优，写入可以按照磁盘特性顺序写入相邻数据，读性能高。写入时需要维护数据页的合并和分隔，写入性能较差。



