# Time Travel 实现逻辑
目前前，有两种实现 Time Travel 的路径：
1. 限制搜索使用的 vec_count，使用在 GrowingSegment 中
2. 生成 bitmask，和 DSL 计算结果合并后再 Merge 到一起

