# SegmentGrowing 
SegmentGrowing 额外拥有以下接口:
1. `PreInsert(size) -> reseveredOffset`: 串行接口, 为未来的 `size` 行插入预留空间, 返回插入的起始点 `reseveredOffset`. 
2. `Insert(reseveredOffset, size, ...Data...)`: 将 `...Data...` 写入到 `[reseveredOffset, reseveredOffset + size)` 这个范围的空间中. 这个接口允许并发调用. 
   1. `...Data...` 包含 row_ids, timestamps 这两个系统属性列, 以及其他数据
      1. 其他数据的存放格式可能是行式的 RowBasedData, 或者列式的 columnBasedData
3. `PreDelete & Delete(reseveredOffset, row_ids, timestamps)` 删除接口, 和插入接口原理一致. 

SegmentGrowing 储存数据的格式为分段(`chunk`)的列式数据, 每一段的行数相等, 
由创建 Segment 的参数 `size_per_chunk` 控制. 
插入时, 先分配至足够数据段保证 `total_size <= num_chunk * size_per_chunk`, 
再将行式的原始数据转成列式写入. 

查询时, 会对每一个 `chunk` 进行搜索，搜索结果储存为 `SubQueryResult`，然后再进行合并操作。

SegmentGrowing 内部还实现了小批量索引的能力. 在 `SegcoreConfig` 中预设了小批量索引的参数. 
当 schema 中指定了 `metric_type` 时, Segment 中会使用预设的参数, 为每一个插入完毕的 `chunk` 
创建对应的小批量索引, 加速查询. 