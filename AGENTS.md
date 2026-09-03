# Course Repo 工程约定

## 仓库资源模型

本仓库的生产资源由两部分共同构成：

1. 课程目录：每个课程目录中的 `manifest.json`、`textbook.md` 和关联内容文件。
2. 根 [README.md](README.md)：独立的、人工维护的资源索引（Weekly Scope catalog）。它与课程目录并列，不是某个课程目录的附属说明。

处理资源发现、规划范围或课程包变更时，必须先读取根 README，再读取目标课程的 manifest 与教材源。README 只用于发现和选择范围；manifest 是课程身份、subject/domain、source 与 activities 的权威，`textbook.md` 是教材事实的权威。

## 修改规则

- 新增、删除、移动、改名，或调整生产资源的 ID、name、subject、source 时，必须同步更新根 README 的 catalog 条目。
- 只变更教材内容时，只更新相应 `textbook.md`；不要把教材细节复制进 README。
- 根 README 的 catalog 必须完整覆盖生产 `manifest.json`，但永远排除顶层 `测试/`、`.git/` 和 `.tmp/`。
- 不得用 README 替代 manifest，也不得从 README 推断未注册 provider 的可执行能力。

## 验证与工作树保护

- 编辑资源目录后运行 `python3 scripts/validate_resource_catalog.py --repo-root .`；该脚本只校验，不会改写文件。
- 修改校验器时运行 `python3 -m unittest discover -s tests -p 'test_*.py' -v`。
- 这是一个可能包含用户未提交内容的嵌套仓库。除非用户明确要求，不执行 `git reset`、`checkout`、`clean`、`pull`、提交或暂存，也不删除未跟踪内容。
