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
1. `PreInsert(int64_t size) -> int64_t`: 串行接口, 为未来的 `size` 行插入分配空间, 返回插入的起始点 `reseveredOffset`. 
2. `Insert(reseveredOffset, size, ...rowData...)`: 将 `...rowData...` 写入到 `[reseveredOffset, reseveredOffset + size)` 这个范围的空间中. 这个接口允许多线程调用. 
   1. `...rowData...` 包含 row_ids, timestamps 这两个系统属性列, 以及行式存放的数据 rowBasedRawData. 
3. `PreDelete & Delete(reseveredOffset, row_ids, timestamps)` 删除接口, 和插入接口原理一致. 

SegmentGrowing 储存数据的格式为分段的列式数据, 每一段的行数由创建 Segment 的参数 `size_per_chunk` 控制. 插入时, 先分配至足够数据段保证 `total_size <= num_chunk * size_per_chunk`, 再将行式的原始数据转成列式写入.

Segment