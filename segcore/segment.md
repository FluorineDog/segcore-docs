# Segment overview 
Segment 目前有两种类型
1. Growing 状态, 允许动态插入, 但是不可以加载索引. 
2. Sealed 状态, 不允许插入, 可以加载索引

两者拥有共同的接口, 都以 SegmentInterface 为基类. 
SegmentInterface 的能力有 参见`${root}/internal/core/src/segcore/SegmentInterface.h`
1. `get_row_count`: 在 Segment 中数据的行数；
2. `get_schema`: 获取 Segment 对应的 Schema
3. `GetMemoryUsageInBytes`: 获取 Segment 的内存占用量
4. `Search(plan, placeholderGroup, timestamp) -> QueryResult`: 根据包含查询参数和 Predicate 的 Plan 进行查询操作, 返回 QueryResult 结果. 保证查询的数据其时间都在 Timestamp 之前. 
5. `FillTargetEntry(plan, &queryResult)`: 根据 plan 中的 targetEntries 列, 补全 queryResult 中缺失的 rowData 数据作为 TargetEntries 的返回值. 

# SegmentGrowing 
SegmentGrowing 额外拥有以下接口:
1. `PreInsert(size) -> reseveredOffset`: 串行接口, 为未来的 `size` 行插入预留空间, 返回插入的起始点 `reseveredOffset`. 
2. `Insert(reseveredOffset, size, ...rowData...)`: 将 `...rowData...` 写入到 `[reseveredOffset, reseveredOffset + size)` 这个范围的空间中. 这个接口允许多线程调用. 
   1. `...rowData...` 包含 row_ids, timestamps 这两个系统属性列, 以及行式存放的数据 rowBasedRawData. 
3. `PreDelete & Delete(reseveredOffset, row_ids, timestamps)` 删除接口, 和插入接口原理一致. 

SegmentGrowing 储存数据的格式为分段(`chunk`)的列式数据, 每一段的行数相等, 
由创建 Segment 的参数 `size_per_chunk` 控制. 
插入时, 先分配至足够数据段保证 `total_size <= num_chunk * size_per_chunk`, 
再将行式的原始数据转成列式写入. 

查询时, 会对每一个 `chunk` 

SegmentGrowing 内部还实现了小批量索引的能力. 在 `SegcoreConfig` 中预设了小批量索引的参数. 
当 schema 中指定了 `metric_type` 时, Segment 中会使用预设的参数, 为每一个插入完毕的 `chunk` 
创建对应的小批量索引, 加速查询. 

# SegmentSealed
SegmentSealed 额外拥有以下接口: 
   1. `LoadIndex(loadIndexInfo)`: 加载索引. indexInfo 中包含
      1. `FieldId`
      2. `IndexParams`: KV 结构的 Index 参数
      3. `VecIndex`: 向量索引
   2. `LoadFieldData(loadFieldDataInfo)`: 加载列数据, 可能是 scalar 列的数据, 也可能是向量列的数据. 
      1. 注意: 同一列的索引和向量数据是可能共存的, 查询时优先使用索引. 
   3. `DropIndex(fieldId)`: 移除并释放已有的列索引
   4. `DropFieldData(fieldId)`: 移除并释放已有的列数据

在查询时, 认为在加载好索引的时刻, timetick 已经走到索引内所有数据之后, 所以无需对 timestamp 进行判断. 
同时, 这个接口允许部分列数据缺失, 只要查询时被涉及到的列数据都加载好了. 
