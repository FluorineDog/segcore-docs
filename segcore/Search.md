# Search
Search 目前支持两种模式: json DSL 模式, 以及 Boolean Expr 模式. 前者处于 deprecated 状态, 不再详解.

Boolean Expr 执行模式如下: 
1. 客户端将 expr 和 topK, 查询向量等数据编码进 proto 中, 发送到 proxynode
2. proxynode 对 proto 进行解析,生成 protobuf 格式的 plan, 并进行静态检查, 塞入上述的 proto 中
3. querynode 反序列化 protobuf 中的 plan 生成可执行的 PlanAST, 执行 Segcore 查询操作.

Expr 文法参见 [expr_grammar.md](expr_grammar.md)

# Segcore 查询流程
获得 PlanAST 后, 使用访问者模式解释执行整棵 AST 树:
1. VectorNode 节点包含向量查询的参数和可选的 Predicate.
   1. 当 Predicate 存在时, 访问执行 PredicateExpr 获得 bitset 作为向量查询的 bitmask. 
   2. 当 Predicate 不存在时, 向量查询的 bitmask 条件为空.
2. PredicateExpr 所对应的 AST 上有以下节点. 使用 Visitor 模式, 自顶向下解释执行, 下层结果将作为上层的输入, 得到最终的 Bitmask
   1. LogicalUnaryExpr: 非语句
   2. LogicalBinaryExpr: 与或语句
   3. TermExpr: 点查语句, `A in [1, 2, 3]`
   4. CompareExpr: 比较语句
3. TermExpr 和 CompareExpr 是执行的叶节点。

