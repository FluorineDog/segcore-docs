# Search

Search 目前支持两种模式: json DSl 模式, 以及 Boolean Expr 模式. 前者处于 deprecated 状态, 不再详解.

Boolean Expr 执行模式如下: 
1. 客户端将 expr 和 topK, 查询向量等数据编码进 
