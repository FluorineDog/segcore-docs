# Scripts and Tools
Segcore 开发过程中可能使用到以下脚本与命令

## 代码格式化
- 在 milvus/internal/core 下
  - 执行 `./run_clang_format .` 格式化 cpp 代码
    - 调用 clang-format-10, 需要预先用 `apt install clang-format-10` 安装
    - 同时调用 `build-support/add_${lang}_license.sh` 为 cmake 和 cpp 文件添加 license 信息
- 在 milvus/ 下
  - 使用 `make cppcheck` 检查格式, 包括
    - 是否 clang-format
    - 是否添加 license 信息
    - 是否满足 `cpplint.py` 的标准, 可能需要手动修复
  - `make verifier` 也包含 `make cppcheck` 的功能

## 代码编译
- 在 milvus/internal/core 下
  - 使用 `./build.sh -u -t Debug -o cmake-build-debug` 
    - 编译 unittest (`-u`)
    - 使用 Debug 模式编译 (`-t Debug`)
    - 输出到 cmake-build-debug 文件夹下 (`-o cmake-build-debug`)
- 也可以使用 Clion 打开 core 这个文件夹，选择 cmake 项目
  - 需要修改参数 `CMake Options`, 保持和上面 `./build.sh` 的第一行输出的参数一致.
  
## Visitor 代码生成 (这一段可以忽略，手写代码也不是不行)
- 在 milvus/tools/core_gen 下
  - 调用 `./all_generate.py` 可以生成 visitor 模式的相关文件
  - 模版来自 templates 目录, 使用 `assmeble` 函数装配
    - 一段模版格式为 `@@@@<output_tag>[@<repeated_key>]...####` 
      - 如果有 `<repeated_key>` 将自动根据数组重复生成
      - 内部的所有 `@@<tag>@@` 都会被 `<tag>` 对应的文本替换掉
      - 生成的文本输出到 `<output_tag>` 中，`main` 为最终的文件输出
  - 参数和配置文件在 `all_generate.py` 中可以调整
  - `extract_extra_body` 将 cpp 中填写的类成员 `ctor_and_member` 和必要的头文件 `extra_inc` 塞入生成的 .h 中
  - `meta_gen` 从被访问数据结构的定义文件中，抽取类名列表 `<struct_name>`, 以供代码生成. 
  - 最后，生成好的文件都会放在 milvus/internal/core/src/query/generated 下.
    - 有部分 cpp 文件需要拷贝到 src/query/visitor 下做进一步修改
