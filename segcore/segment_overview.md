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

原则上，Growing/Sealed 可复用的代码逻辑都尽量写入了 `SegmentInternalInterface` 中，
另外两个类包含了差异较大的部分
   
参见
1. [segment_interface.md](segment_interface.md)
2. [segment_growing.md](segment_growing.md)
3. [segment_sealed.md](segment_sealed.md)
