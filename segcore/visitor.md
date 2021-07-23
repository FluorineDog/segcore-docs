# Visitor 模式
Visitor Pattern 在 Core 中被用于 Execution Plan 的解释执行。

1. `${core}/src/query/PlanNode.h` 中，包含物理计划，其中
   1. `FloatVectorANNS` FloatVector 搜索的执行算子
   2. `BinaryVectorANNS` BinaryVector 搜索的执行算子
2. `${core}/src/query/Expr.h` 包含表达式的计算物理算子，包括
   1. `TermExpr` 支持 `col in [1, 2, 3]` 等操作
   2. `RangeExpr`支持 `a >= 5` `1 < b < 2` 等常量与数据列的比较
   3. `CompareExpr` 支持系统列间的比较，比如 `a < b`
   4. `LogicalBinaryExpr` 与/或 二元逻辑操作符
   5. `LogicalUnaryExpr` 非 一元逻辑操作符
注意到，TermExpr 和 RangeExpr 都有

目前支持以下 visitor，在 `${core/query/visitors}` 下
1. `ShowPlanNodeVisitor` 以 json 格式打印 PlanNode
2. `ShowExprVisitor` Expr -> json
3. `Verify...Visitor` 验证...合法性
4. `ExtractInfo...Visitor` 从 ... 中提取必要信息，包括 involved_fields 等
5. `ExecExprVisitor` 根据 Expr 计算表达式, 生成 bitmask
6. `ExecPlanNodeVistor` 物理计划执行器，目前仅支持 ANNS Node

