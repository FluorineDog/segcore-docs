# SegmentInterface

## 对外接口
1. `get_row_count`: 在 Segment 中数据的行数
2. `get_schema`: 获取 Segment 对应的 Schema
3. `GetMemoryUsageInBytes`: 获取 Segment 的内存占用量
4. `Search(plan, placeholderGroup, timestamp) -> QueryResult`: 根据包含查询参数和 Predicate 的 Plan 进行查询操作, 返回 QueryResult 结果. 保证查询的数据其时间都在 Timestamp 之前. 
5. `FillTargetEntry(plan, &queryResult)`: 根据 plan 中的 targetEntries 列, 补全 queryResult 中缺失的 rowData 数据作为 TargetEntries 的返回值. 

具体细节可以参见 `${milvus_root}/internal/core/src/segcore/SegmentInterface.h`

## 基本概念：
1. Segment: 基于 Timestamp 划分的数据分片，分片逻辑在 Go 端控制
2. chunk: Segment 的进一步划分，每一个 chunk 的 column 都是连续的数据
   1. SealedSegment 中，只会有一个 chunk
   2. GrowingSegment 中，目前为固定行数划分 chunk，随着数据插入，chunk 数量会随之增加
3. Span: 类似 std::span, 指向一段连续的内存数据 
4. SystemField: 包含 RowID 和 Timestamp 两种，是额外储存的列
5. SegOffset: 数据在 Segment 中的行号. 
6. 
## 内部函数
1. `num_chunk()`: 整体 chunk 数量
2. `size_per_chunk()`: 每个 chunk 的长度
3. `get_active_count(Timestamp)`: Timestamp 筛选后，参与计算的行数
4. `chunk_data(FieldOffset, chunk_id) -> Span<T>`: 返回指定列和 chunk 的连续原始数据。
5. `chunk_scalar_index(FieldOffset, chunk_id) -> const StructuredIndex<T>&`: 返回指定常量列，指定 chunk 的倒排索引
6. `num_chunk_index`: 已经创建好索引（包括索引和向量索引）的数量
   1. 在 GrowingSegment 这里，这个数值为已经创建好常量索引的 chunk 数量，这一部分可以走索引加速计算
   2. SealedSegment 一定为 1
7. `debug()`: Debug 时可调用此函数打印额外信息
8. `vector_search(vec_count, query..., timestamp, bitset, output)`: 对向量列进行 Search 操作
   1. `vec_count`: 指定有多少行 Segment 内部向量参与 Search 计算，在上层由 Timestamp 列计算得出
      1. 此功能主要用于 GrowingSegment 中，保证时间序关系
   2. `query...` 多个变量共同指定 query 的参数和数据
   3. `timestamp`: 指定 Timestamp 时的，用于筛选，主要用于 SealedSegment
   4. `bitset`: DSL 计算得到的 mask 数值
   5. `output` 输出 QueryResult
9. `bulk_subscript(FieldOffset|SystemField, seg_offsets..., output)`: 
   - 给一列 seg_offsets, 根据列计算出 `results[i] = FieldData[seg_offsets[i]]`, 用于 GetEntityByIds
   - FieldData 由 FieldOffset 或者 SystemField 指定
10. `search_ids(IdArray, timestamp) -> pair<IdArray, SegOffsets>`: 
    1.  根据 IdArray 中的 Primary Key, 找到对应的 SegOffset
    2.  不保证返回的序关系，但是返回的两列一定一一对应。
    3.  不存在的 PK 不会返回.
11. `check_search(Plan)`: 检查 Plan 是否合法
    1.  主要检查 Plan 中使用到列是否已经被加载
