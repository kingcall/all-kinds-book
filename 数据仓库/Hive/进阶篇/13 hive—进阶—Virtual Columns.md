## Virtual Columns

Hive 0.8.0 提供了对两个虚拟列的支持：

一个是`INPUT__FILE__NAME`，这是 Map 器任务的 Importing 文件名。

另一个是`BLOCK__OFFSET__INSIDE__FILE`，这是当前的全局文件位置。

对于块 zipfile，它是当前块的文件偏移量，这是当前块的第一个字节的文件偏移量。

从 Hive 0.8.0 开始，添加了以下虚拟列：

- ROW__OFFSET__INSIDE__BLOCK
- RAW__DATA__SIZE
- ROW__ID
- GROUPING__ID

重要的是要注意，此处列出的所有“虚拟列”都不能用于任何其他目的**(即使用具有虚拟列的列创建表将失败，并显示“ SemanticException 错误 10328：无效的列名.”)

### Simple Examples

从 src 中选择`INPUT__FILE__NAME`，密钥`BLOCK__OFFSET__INSIDE__FILE`；

选择密钥，按密钥 Sequences 从 src 组中计数(`INPUT__FILE__NAME`)；

从 src 中选择*，其中`BLOCK__OFFSET__INSIDE__FILE`> 12000 通过密钥排序；

