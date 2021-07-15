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

在查询时, 允许部分列数据缺失, 只要查询时被涉及到的列数据都加载好了. 

# SegmentSealedImpl 内部数据定义
1. `row_count_opt_`: 
   1. 加载第一行数据时，写入 row_count 数据
   2. 其后加载的所有列都必须和这个数据相等
3. `xxx_ready_bitset_` `system_ready_count_`
   1. 用来记录对应列是否被加载, bitset 和 FieldOffset 对应
   2. 当且仅当以下条件满足是，可以进行查询
      1. system_ready_count_ == 2， 也就是两列系统列 RowId/Timestamp 都被加载
      2. 查询涉及常量列被加载过了
      3. 查询涉及到的向量列，要么原始数据列被加载，要么索引被加载
4. `scalar_indexings_`: 存放常量索引
   1. 使用 knowhere 中 StructuredSortedIndex
5. `primary_key_index_`: 存放 pk 列的索引 
   1. 使用全新的 ScalarIndexBase 格式
   2. **注意，这里的功能和常量索引可能有重叠，建议将 4. 替换掉后复用**
6. `field_datas_`: 存放原始数据
   1. `aligned_vector<char>` 格式保证 `int/float` 数据存放后是对齐的
7. `SealedIndexingRecord vecindexs_`: 存放向量索引数据
8. `row_ids_/timestamps_`: RowId/Timestamp 原始数据
9. `TimestampIndex`: 给 Timestamp 提供的索引
10. `schema`: schema

# SegmentSealedImpl 内部函数定义
1. 大多数函数是 SegmentInternalInterface 相应函数的实现，不再赘述
2. `update_row_count`: 用上文的逻辑更新 row_count
3. `mask_with_timestamps`: 使用 Timestamp 列的信息，对 mask 进行更新，以支持 Time Travel 功能