# 设计文档：打印当前时间并保留模型执行溯源

## 1. 概述与背景

本仓库当前仅包含一个最小化的 Python 3 程序 `hello_world.py`，其行为是向标准输出写入 `Hello, World!` 并换行，退出码为 0，标准错误为空（已在工作树中复现，stdout 字节为 `4865 6c6c 6f2c 2057 6f72 6c64 210a`，exit=0）。

issue #3 要求在「保留现有输出」的前提下增加打印当前时间的功能，并交付中文设计文档、保留测试报告、记录各阶段由哪些模型执行哪些任务，最后保留分支并合入 main。

本设计文档即满足需求 #2 所要求的中文设计文档，并取代现有英文版 `DESIGN.md`。

## 2. 需求

1. **打印当前时间**：在保留现有 `Hello, World!` 输出的情况下，额外打印当前时间。
2. **中文设计文档**：输出中文设计文档（即本文件，取代英文 `DESIGN.md`），同时保留测试报告 `TEST_PLAN.md`（不删除，仅更新断言）。
3. **保留分支并合入 main**：保留创建的特性分支 `agent/issue-3-20260624182301`，并将其合入 `main`（目标为 `origin/main` = `d13b3cb`）。
4. **保留模型执行溯源**：记录哪些模型执行了哪些任务，作为仓库级产物随合并进入 main。

## 3. 非目标

- 不引入命令行参数、环境变量或时区配置（使用本地时间）。
- 不引入打包（setup.py / pyproject）、不发布到 PyPI、不引入第三方依赖。
- 不修改 CI / 不新增 GitHub Actions 工作流。
- 不支持 ISO-8601 带时区偏移格式或多行时间输出。
- 不对现有 `Hello, World!` 行做任何字符级改动（保留逐字节一致）。
- 不翻译 `TEST_PLAN.md` 的代码与断言（保留代码可读性，仅更新断言语义并可选地增加中文小标题）。
- 不改动本地陈旧的 `main`（`928b5b8`）本身；合并目标始终对齐 `origin/main`。

## 4. 方案与设计

### 4.1 `hello_world.py` 变更

在现有 `print("Hello, World!")` 之后追加一行打印当前本地时间，采用 `datetime.now().strftime("%Y-%m-%d %H:%M:%S")`：

```python
from datetime import datetime

print("Hello, World!")
print(datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
```

- 第 1 行输出保持 `Hello, World!\n` 不变（满足「在现有输出的情况下」）。
- 第 2 行为当前时间，固定格式 `YYYY-MM-DD HH:MM:SS`，便于测试用正则校验。
- 退出码仍为 0，标准错误仍为空。
- 仅使用标准库 `datetime`，无新增依赖。

### 4.2 `DESIGN.md` 重写为中文

将现有英文 `DESIGN.md` 替换为本中文设计文档（即本文件内容），描述新增的时间打印行为、非目标、测试策略与回滚。原英文「逐字节 stdout 契约」描述更新为「结构化 stdout 契约」（见第 8 节）。

### 4.3 `TEST_PLAN.md` 更新（保留）

保留 `TEST_PLAN.md`（满足需求 #2「保留测试报告」），但更新其断言以适配新增的时间行：

- 将 `assert result.stdout == b"Hello, World!\n"` 改为结构化断言：
  - stdout 按行拆分后，第 1 行严格等于 `Hello, World!`；
  - 第 2 行匹配正则 `^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$`；
  - 总行数为 2 且以换行结尾。
- 保留 `result.returncode == 0` 与 `result.stderr == b""` 断言。
- 「Branch artifact check」的预期文件清单由 `DESIGN.md / TEST_PLAN.md / hello_world.py` 扩展为包含 `PROVENANCE.md`。
- 可选地为各小节增加中文标题，但代码块与断言保持原语言以保证可执行。

### 4.4 `PROVENANCE.md` 新增（仓库级溯源）

在仓库根目录新增 `PROVENANCE.md`，使用 Markdown 表格定义固定 schema：

| Stage | Agent | Model | Task |
| --- | --- | --- | --- |
| Analysis | issue-to-pr-analyst | glm-5.2 (inherited) | 根因分析与行为复现 |
| Architecture | issue-to-pr-architect | glm-5.2 (inherited) | 设计文档与验收标准 |
| Implementation | issue-to-pr-implementer | glm-5.2 (inherited) | 修改 hello_world.py 与文档 |
| Security | issue-to-pr-security | glm-5.2 (inherited) | 安全审查 |
| Verification | issue-to-pr-verifier | glm-5.2 (inherited) | 执行测试计划验证 |
| Documentation | issue-to-pr-documenter | glm-5.2 (inherited) | 终稿化溯源任务描述与文档 |

- Schema 列固定为 `Stage | Agent | Model | Task`，由架构阶段定义。
- 运行时配置对六个阶段均设置 `model: "inherit"`，继承的活动模型为 `glm-5.2`，需在文件中显式声明。
- `Task` 列的具体描述由 documentation 阶段终稿化；架构阶段负责定义 schema 与初始内容。
- 仓库级文件（而非运行目录产物）以确保合并入 main 后持久存在。

## 5. 备选方案

- **时间输出位置**：同行追加 vs. 新增一行。选择「新增一行」：对现有 `Hello, World!` 行零侵入，真正满足「在现有输出的情况下」。
- **时间格式**：ISO-8601 带时区 vs. `strftime("%Y-%m-%d %H:%M:%S")`。选择后者：无时区配置非目标，格式确定且易于正则验证。
- **溯源产物位置**：运行目录产物 vs. 仓库级 `PROVENANCE.md`。选择仓库级：运行目录产物不会随合并进入 main，无法满足需求 #4。
- **合并策略**：Squash merge vs. 真正 merge commit。选择真正 merge commit：保留特性分支 ref 以满足「保留分支」。
- **TEST_PLAN 处理**：翻译为中文 vs. 保留并更新断言。选择保留并更新：issue 措辞为「保留测试报告」，非「翻译」。

## 6. 回滚

回滚方式为 `git revert <feature-commit>`（撤销功能提交），由于无状态迁移、无外部依赖、无兼容性影响，回滚为纯操作。回滚后 `hello_world.py` 恢复为单行 `print("Hello, World!")`，`DESIGN.md`/`TEST_PLAN.md`/`PROVENANCE.md` 视回滚范围一并恢复或单独保留。无需数据库迁移或服务降级。

## 7. 交付与合并

- 特性分支：`agent/issue-3-20260624182301`（当前 HEAD = `d13b3cb` = `origin/main`，工作树干净）。
- 合并目标：`origin/main`（`d13b3cb`），而非本地陈旧的 `main`（`928b5b8`）。
- 合并方式：真正 merge commit（非 squash），以保留分支 ref。
- 合并为交付阶段动作，受 `require_push_approval` 与 `require_merge_approval` 双重审批门控，架构阶段不执行合并。

## 8. 测试策略

- **结构化 stdout 断言**：放弃逐字节 `b"Hello, World!\n"` 全量断言，改为按行结构化校验（第 1 行严格匹配，第 2 行正则匹配时间格式）。
- **不变量断言**：退出码 == 0、stderr == `b""` 保持不变。
- **制品完整性**：`git diff --name-only origin/main...HEAD` 须包含 `hello_world.py`、`DESIGN.md`、`TEST_PLAN.md`、`PROVENANCE.md`。
- **设计文档语言**：`DESIGN.md` 须为中文且描述新行为（人工/字典校验含中文字符且无原英文契约描述）。
- **溯源完整性**：`PROVENANCE.md` 存在于仓库根，表格含六个阶段且均映射到 agent + glm-5.2 + task。
