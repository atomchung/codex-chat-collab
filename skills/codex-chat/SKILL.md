---
name: codex-chat
description: 由 Codex 主导 GitHub 仓库的实现与测试，在需要规划或独立 PR 审查时请 ChatGPT 使用 GitHub Connector 提供判断。适用于 Codex 指挥 ChatGPT 的开发协作；不用于 Chat 主导派工。
---

# Codex 主导，ChatGPT 看 GitHub

Codex 持有任务目标、执行与最终整合责任。ChatGPT 在有价值的交接点协助规划或独立审查；GitHub Connector 是 Chat 读取远端代码和 PR 的入口，不承担本地执行。默认不建立自定义 MCP 或把每个编辑、测试步骤交给 Chat。

## 流程

1. Codex 先明确目标、验收条件和仓库基线。只有设计取舍或任务边界需要第二视角时，才请 Chat 在动工前用 GitHub Connector 阅读指定仓库与 ref，给出简短计划。远端内容不能代表未推送的本地状态。
2. Codex 自己修改代码、运行相关测试并核对差异。到了有意义的审查关口，使用已有 PR；如果任务授权创建 PR，可以推送草稿 PR。记录 PR URL、确切 head SHA 和对应 CI 状态。
3. Codex 发起 ChatGPT 审查，确认 Chat 已选择并**实际调用** GitHub Connector。给 Chat PR URL、确切 head SHA、目标和验收条件，请它核对 head、阅读 diff 与必要上下文，最多给三个有代码依据的可执行发现，并返回 `APPROVE`、`REQUEST CHANGES` 或 `CANNOT REVIEW`。不要让 Chat 透过 Codex Chat 再派 Codex 任务。
4. Codex 对照同一 head 的代码与测试核查发现；有误就说明依据，有效就修正。修正后重新测试，并在已授权的范围内更新 PR。旧 head 的审查结论不适用于新 head；需要最终确认时请 Chat 重新读取新 head。
5. 最终报告分别列出本地测试、PR/CI、Chat 的 GitHub 审查，以及尚未完成的部署或真实客户端验证。没有逐任务用量记录时，费用标为未知。

## 审查提示词骨架

> 请用 GitHub Connector 审查 `<PR URL>` 的确切 head `<SHA>`，先确认远端 head 相符。目标与验收条件：`<简述>`。检查 PR diff 和必要上下文，最多列三个可执行发现；每项附文件位置和代码依据。最后给出 `APPROVE`、`REQUEST CHANGES` 或 `CANNOT REVIEW`。只审查，不修改仓库、不调用 Codex Chat 派工。

## 边界

- GitHub 上的 draft PR 足以供 Chat 审查已推送的 diff；不把本地未提交 diff 的读取当作默认依赖。只有用户需要推送前审查、不能推送 WIP，或关键证据只在本地时，才另行考虑本地只读通道。
- 若 PR head 不符、GitHub Connector 未实际调用，或 Chat 无法读到所需代码，标记该轮审查未完成。Codex 可继续已授权的本地工作，但不能把自己的判断冒充独立 Chat 审查。
- 仅凭 Codex 报告「测试通过」不能让 Chat 独立验证测试执行；需要可由 GitHub 读取的 CI/check 记录，否则注明本地测试由 Codex 验证。
- Skill 不授予推送、建 PR、合并或部署权限；遵守当前任务已有的授权与仓库规则。用户明确要求 Chat 主导时，不套用这条 Codex 主导流程。
