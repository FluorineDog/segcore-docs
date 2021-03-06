# 基本类型

- `Collection`: 数据表, 包含多个 Segment
- `Segment`: 存放一段数据的内存结构, 支持可并发的插入, 删除, 查询, 索引加载, 监控统计等功能. 
- `Schema`: Collection 的数据格式定义, 包含
  - `vector<FieldMeta>`, 即 FieldMeta 的有序集合
  - `isAutoId`. 当其为 True 时, 默认 `RowId` 为主键列
  - `primaryKey` (when `isAutoId = False`), 指定主键列
- `FieldMeta`: 列的属性, 包含
  - `DataType` 数据类型, 包含 Int8...Int64, Float, Double, FloatVector, BinaryVector
  - `Dim` (when dataType is vector type): 向量维度
  - `metric_type` (when dataType is vector type, optional): 这个向量所对应的 MetricType, 和小批索引相关, 可以为空. 
  - `FieldName`: 列名
  - `FieldId`: 列的唯一编号
  - (隐藏) `FieldOffset`, 为 Schema 中 `vector<Field>` 的下标. Segcore 内部计算基本基于 FieldOffset
- `Span` 类似 std::span，支持 Vector 类型的数据，同时可以隐式转化为 `SpanBase` 以方便接口重载。