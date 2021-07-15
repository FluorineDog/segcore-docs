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

# SegmentSealedImpl 内部函数