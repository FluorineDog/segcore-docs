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

## SegmentGrowingImpl 内部参数
1. SegcoreConfig 包含 Segcore 的一些参数，需要在创建 Segment 对象时指定
2. InsertRecord 插入的数据在这里
3. DeleteRecord 已被废弃
4. IndexingRecord 包含小批量索引的数据
5. SealedIndexingRecord 已被废弃

### SegcoreConfig
1. 管理 chunk_size 和小批量索引的参数
2. `parse_from` 可以从 yaml 中解析 （此功能目前没有实装）
   1. 参考 `${milvus}/internal/core/unittest/test_utils/test_segcore.yaml`
3. `default_config` 提供默认的参数

### InsertRecord
用来管理插入的数据。可以并发插入，内部包含
1. 计算预留空间的 `atomic<int64_t> reserved`
2. 计算插入段，给出当前已完成插入数据的 `AckResponder` 
3. 储存数据列的 `ConcurrentVector`，每一列都有一个独立的对象存储数据

插入时，执行以下逻辑：
1. 上层串行调用 `PreInsert(size) -> reserved_offset`，分配一段地址空间
   1. 这一段地址空间为 `[reserved_offset, reserved_offset + size)`
2. 上层并发调用 `Insert(reserved_offset, size, ...Data...)` 接口，将数据写入上述地址空间中
    1. 首先，对每一列数据的 `ConcurrentVector `调用 `grow_to_at_least` 接口预留空间
    2. 对每一列调用 `set_data_raw` 接口，把数据填充到相应位置
    3. 完成后，对 `AckResponder` 调用 `AddSegment`，将 `[reserved_offset, reserved_offset + size)` 标记为已经插入
       1. AckResponder 的 GetAck 接口可以返回 `reserverd`

### ConcurrentVector
这是可以并发插入的列数据存储工具。由多段数据 chunk 构成。
1. `grow_to_at_least(size)` 调用后，预留不低于 `size` 的空间
2. `set_data_raw(element_offset, source, element_count)` 将 source 指向的一段连续数据
3. `get_span(chunk_id)` 获取指向对应 chunk 的 span