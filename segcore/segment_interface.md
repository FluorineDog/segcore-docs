# SegmentInterface

## 对外接口
1. `get_row_count`: 在 Segment 中数据的行数
2. `get_schema`: 获取 Segment 对应的 Schema
3. `GetMemoryUsageInBytes`: 获取 Segment 的内存占用量
4. `Search(plan, placeholderGroup, timestamp) -> QueryResult`: 根据包含查询参数和 Predicate 的 Plan 进行查询操作, 返回 QueryResult 结果. 保证查询的数据其时间都在 Timestamp 之前. 
5. `FillTargetEntry(plan, &queryResult)`: 根据 plan 中的 targetEntries 列, 补全 queryResult 中缺失的 rowData 数据作为 TargetEntries 的返回值. 

具体细节可以参见 `${milvus_root}/internal/core/src/segcore/SegmentInterface.h`
## 内部函数 
1. 
