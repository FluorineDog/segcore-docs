# Time Travel 实现逻辑 (Segment Level)
目前前，有两种实现 Time Travel 的路径：
1. 限制搜索使用的 vec_count, 使用在 GrowingSegment 中
2. 生成 bitmask，和 DSL 计算结果合并后再 Merge 到一起, 主要使用在 SealedSegment 中

## Growing Segment Time Travel 
1. 在插入时，保证插入的数据时按照时间升序的. 
2. 二分查找到 Timestamp 的位置，记录为 vec_count
   1. 这里调用了 get_active_count 接口，直接定位到具体一行
3. 调用 vector_search 接口计算即可，无需处理 DSL 生成的 MASK

## SealedSegment Time Travel
1. 在 Load 时，数据被放置在连续的内存区域中，但是有以下性质：
   1. 分为多段
   2. 段内的 Timestamp 是无序的
   3. 段与段直接 Timestamp 有序。也就是保证上一个段的任何行的 ts 一定小于下一个段的 Timestamp
2. 算法为
   1. 使用 get_active_count 接口，找到最后一个包含合法 Ts 的段，返回这个段的最后一个元素位置作为 vec_count
   2. 计算 Timestamp 的 Bitset mask. 由于上述性质，之前的段的行全部满足条件，之后的段全部不满足，只需要计算这个 “最后一个段” 即可。
   3. 计算出来的 Bitset 和 DSL 的结果合并后，调用 vector_search 接口.


