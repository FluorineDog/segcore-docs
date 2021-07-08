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


