本文件记录 issue-to-pr 工作流各阶段由哪些模型执行哪些任务（运行时配置 model: inherit，继承的活动模型为 glm-5.2）。

| Stage | Agent | Model | Task |
| --- | --- | --- | --- |
| Analysis | issue-to-pr-analyst | glm-5.2 (inherited) | 根因分析与行为复现 |
| Architecture | issue-to-pr-architect | glm-5.2 (inherited) | 设计文档与验收标准 |
| Implementation | issue-to-pr-implementer | glm-5.2 (inherited) | 修改 hello_world.py 与文档 |
| Security | issue-to-pr-security | glm-5.2 (inherited) | 安全审查 |
| Verification | issue-to-pr-verifier | glm-5.2 (inherited) | 执行测试计划验证 |
| Documentation | issue-to-pr-documenter | glm-5.2 (inherited) | 终稿化溯源任务描述与文档 |
