# Segment Overview 
Segment 目前有两种类型
1. Growing 状态, 允许动态插入, 但是不可以加载索引. 
2. Sealed 状态, 不允许插入, 可以加载索引

两者拥有共同的接口, 都以 `SegmentInterface` 为基类. 外部调用者只需要关心 
1. `SegmentInterface` 
2. `SegmentGrowing` & `CreateGrowingSegment`
3. `SegmentSealed` & `CreateSealedSegment`
三个类的函数声明及对应的构造函数，其余内部函数与皆作为实现细节隐藏于
1. `SegmentInternalInterface`
2. `SegmentGrowingImpl`
3. `SegmentSealedImpl`

SegmentInterface 的能力有 参见`${root}/internal/core/src/segcore/SegmentInterface.h`
1. `get_row_count`: 在 Segment 中数据的行数；
2. `get_schema`: 获取 Segment 对应的 Schema
3. `GetMemoryUsageInBytes`: 获取 Segment 的内存占用量
4. `Search(plan, placeholderGroup, timestamp) -> QueryResult`: 根据包含查询参数和 Predicate 的 Plan 进行查询操作, 返回 QueryResult 结果. 保证查询的数据其时间都在 Timestamp 之前. 
5. `FillTargetEntry(plan, &queryResult)`: 根据 plan 中的 targetEntries 列, 补全 queryResult 中缺失的 rowData 数据作为 TargetEntries 的返回值. 

