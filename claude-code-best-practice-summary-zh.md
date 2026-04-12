# claude-code-best-practice 中文总结

来源仓库: https://github.com/shanraisshan/claude-code-best-practice

整理时间: 2026-04-12  
参考快照: `46d7e2b6eca0ec94d9e63792772ddf339945eaaf`，仓库最近提交时间为 2026-04-11  
说明: 这是一份中文概要，不是逐字翻译。重点提炼仓库里的实践框架、配置思路和落地建议。

## 一句话总结

这个仓库把 Claude Code 的使用方式从“让 AI 随手写代码”整理成一套更工程化的工作法: 用 `CLAUDE.md` 管长期记忆，用 commands 编排流程，用 skills 注入可复用能力，用 subagents 隔离复杂任务，再用 hooks、MCP、权限、计划模式、测试门禁和多模型复核来提高可靠性。

## 仓库主要内容

仓库不是一个应用代码项目，而是一份 Claude Code 最佳实践资料库和示例库。它覆盖这些主题:

- Claude Code 的核心概念: subagents、commands、skills、hooks、MCP、plugins、settings、memory、checkpointing、CLI flags。
- 实现示例: 一个天气工作流展示 `Command -> Agent -> Skill` 的协作方式。
- 使用技巧: 来自 Claude Code 团队、Boris Cherny、Thariq 等人的经验整理。
- 开发工作流: Research -> Plan -> Execute -> Review -> Ship，以及跨模型 Claude Code + Codex 复核流程。
- 报告: agents、commands、skills 的区别，大型 monorepo 中 skills 的发现机制，全局配置和项目配置的边界，浏览器自动化 MCP 对比，使用限制和上下文退化分析等。

## 核心架构: Commands、Agents、Skills

仓库最重要的观点是: 不要把所有事情都塞进一个提示词，而要按职责拆开。

### Command

Command 是用户显式触发的入口，通常以 `/xxx` 的形式运行。它适合做工作流编排，例如先问用户选择，再调用 agent，再调用 skill，最后输出结果。Command 默认在主对话上下文里执行，所以适合轻量、明确、用户主动触发的流程。

适合使用 command 的场景:

- 需要用户主动启动的标准流程。
- 需要串联多个步骤、agent 或 skill。
- 不希望每次会话都把大量说明加载进上下文，只在触发时读取。

### Agent / Subagent

Agent 是独立上下文里的自主执行者。它适合多步骤任务、探索代码库、后台执行、隔离上下文、限制工具权限、切换模型或预加载特定 skills。

适合使用 agent 的场景:

- 复杂、多步、自主性强的任务。
- 不想污染主上下文的调查或执行。
- 需要不同模型、权限、工具或 MCP server。
- 需要背景运行、工作树隔离、持久记忆或专门角色。

### Skill

Skill 是可复用能力包。它可以被 Claude 根据描述自动触发，也可以被 command 或 agent 使用。Skill 的描述会进入上下文用于匹配，完整内容通常按需加载，这对上下文管理很重要。

适合使用 skill 的场景:

- 可复用的步骤、规范、检查清单、领域知识。
- 希望 Claude 在相关任务中自动调用。
- 希望给 agent 预加载特定领域能力。
- 想把复杂知识做成渐进披露，而不是一次塞满上下文。

### 选择顺序

如果用户只是问一个简单问题，优先用轻量机制。仓库里的判断倾向是:

1. Skill: 轻量、可自动匹配、适合常见可复用动作。
2. Agent: 更重，但有独立上下文，适合复杂自治任务。
3. Command: 不会自动触发，适合作为用户主动启动的工作流入口。

## 典型编排: Command -> Agent -> Skill

仓库用天气示例说明一个清晰的分层模式:

1. 用户运行 `/weather-orchestrator`。
2. Command 负责询问温度单位，并协调后续步骤。
3. Command 调用 `weather-agent`。
4. `weather-agent` 预加载 `weather-fetcher` skill，用它的说明去获取天气数据。
5. Command 再调用 `weather-svg-creator` skill，把结果渲染成 SVG 和 Markdown。

这个例子强调两种 skill 用法:

- Agent skill: 预加载进 agent，作为该 agent 的领域知识。
- Direct skill: 被 command 或主会话直接调用，完成独立输出任务。

核心设计原则是职责清晰: command 管流程，agent 管自主数据获取，skill 管可复用能力或输出生成。

## CLAUDE.md 和长期记忆

仓库认为 `CLAUDE.md` 是提升 Claude Code 输出质量最重要的项目文件之一。它应该存放团队共享的约定、项目结构、测试命令、代码风格、常见陷阱和工作流规则。

关键建议:

- 根目录 `CLAUDE.md` 放全仓库共享规则。
- 子目录 `CLAUDE.md` 放组件或包级别规则。
- 个人偏好放本地忽略文件，不要提交给团队。
- 在大型 monorepo 中，Claude 启动时会向上读取祖先目录里的 `CLAUDE.md`，对子目录的 `CLAUDE.md` 则通常在接触对应文件后惰性加载。
- 不要把 `CLAUDE.md` 写成巨型百科。仓库自身建议保持精炼，避免让模型被过多规则稀释。

## Skills 在 monorepo 中的发现方式

Skills 和 `CLAUDE.md` 的加载机制不同:

- 根级 `.claude/skills/` 中的 project skills 会较早可见。
- 子包里的 `.claude/skills/` 会在处理对应目录文件时被发现。
- Skill description 会进入上下文用于匹配，完整内容一般只在调用时加载。
- 如果 skill 名称冲突，企业级、个人级、项目级之间有优先级；plugin skills 通常带命名空间，避免冲突。

实践建议:

- 共享流程放根级 `.claude/skills/`。
- 包级框架规范放各包自己的 `.claude/skills/`。
- 危险操作类 skill 设置为必须显式调用。
- 描述字段要短而准，因为描述会消耗上下文预算。

## Settings、权限和安全

仓库把 Claude Code 配置分成全局和项目两个层级:

- 全局目录 `~/.claude/`: 个人设置、keybindings、任务、agent teams、个人 agents、个人 skills、用户级 MCP、凭据等。
- 项目目录 `.claude/`: 团队共享 settings、commands、skills、agents、hooks、rules。
- 项目根目录 `.mcp.json`: 项目级 MCP servers。

设置优先级大致是:

1. 命令行参数。
2. `.claude/settings.local.json`。
3. `.claude/settings.json`。
4. `~/.claude/settings.local.json`。
5. `~/.claude/settings.json`。
6. 组织 managed settings 是策略层，不能被普通本地配置覆盖。

权限方面，仓库鼓励预先允许常用安全工具，减少重复确认，但不建议随意使用完全跳过权限的模式。`deny` 规则应该被视为安全底线。

## MCP 使用建议

仓库的态度不是“装越多 MCP 越好”，而是选择少数高价值 MCP。推荐方向包括:

- Context7: 获取最新库文档，减少过时 API 幻觉。
- Playwright: 做浏览器自动化、前端验证、截图、表单和页面测试。
- Claude in Chrome 或 Chrome DevTools 类 MCP: 让模型看到真实浏览器状态、控制台和网络信息。
- DeepWiki: 快速理解 GitHub 仓库结构。
- Excalidraw: 生成架构图和流程图。

建议路径是: 先用文档类 MCP 做 research，再用浏览器类 MCP debug，最后用图形或文档工具沉淀结果。

## 工作流建议

仓库反复强调复杂任务要走工程流程，而不是一次性让模型“直接写完”。

推荐主线:

1. Research: 先读代码、读文档、明确约束。
2. Plan: 进入计划模式，拆阶段，定义测试门禁。
3. Execute: 按阶段实现，每阶段尽量可验证。
4. Review: 用 code review、background agents 或跨模型检查找问题。
5. Ship: 小 PR、清晰提交、可回滚。

常见实践:

- 复杂任务先用 plan mode。
- 给 Claude 明确的验证方式，例如测试命令、截图检查、lint、日志检查。
- 使用 git worktrees 并行跑多个上下文。
- PR 尽量小，便于 review、revert 和 bisect。
- 经常提交，任务完成后及时落盘。
- 对长期任务使用 background tasks 或 scheduled tasks。
- 使用 `/context` 和 `/compact` 管理上下文，避免上下文污染。

## 跨模型工作流: Claude Code + Codex

仓库提供了一个跨模型 QA 模式:

1. Claude Code 用 plan mode 产出分阶段计划和测试门禁。
2. Codex CLI 在另一个终端审查计划，基于真实代码库补充问题和中间阶段。
3. Claude Code 按计划逐阶段实现。
4. Codex CLI 再审查实现是否符合计划。

这个流程的价值在于用不同模型的偏差互相制衡，尤其适合高风险改动、复杂架构调整和需要更严格复核的任务。

## 调试和验证

仓库给出的调试建议很务实:

- 用 `/doctor` 检查安装、认证和配置问题。
- 出现 UI 问题时让模型看截图或通过浏览器 MCP 观察真实页面。
- 长时间运行的命令放后台任务，便于持续看日志。
- 前端问题优先让模型看 console、network、DOM 和截图。
- 让另一个 agent 或另一个模型做独立验证。
- 对模型输出保持验证习惯，不把“看起来合理”当作完成。

## 自定义体验

仓库也整理了很多个性化能力:

- `/powerup`: 通过互动课程学习 Claude Code 功能。
- `/model` 和 `/effort`: 根据任务调模型和推理强度。
- status line: 显示上下文、模型、成本和会话信息。
- hooks: 在工具调用、停止、压缩、权限请求等事件上执行脚本。
- `/voice`: 语音输入。
- `/remote-control`、`/teleport`、Claude Code Web: 跨设备或云端继续任务。
- agent teams: 多个 Claude Code session 协作，适合复杂并行任务。

## 我会如何落地这套实践

如果要把这份仓库里的实践应用到自己的项目，可以按这个顺序来:

1. 第一天: 跑 `/init`，写一个短而清晰的 `CLAUDE.md`，记录项目结构、测试命令、代码风格和禁止事项。
2. 第一周: 固化 2 到 3 个高频 commands，例如 `/plan-feature`、`/fix-bug`、`/review-pr`。
3. 第二周: 把重复规范做成 skills，例如测试策略、前端检查清单、API 设计规则。
4. 第三周: 为复杂任务增加 subagents，例如 reviewer、researcher、frontend-debugger、migration-planner。
5. 第四周: 接入少数关键 MCP，例如 Context7 和 Playwright，并把权限写清楚。
6. 之后: 对高风险任务启用跨模型复核或后台 agent 验证，把发现的问题沉淀回 CLAUDE.md、skills 或 hooks。

## 注意事项

- 这个仓库更新很快，内容和 Claude Code 版本强相关。使用前最好确认当前 Claude Code 文档和 changelog。
- 仓库混合了官方文档、社区经验、视频整理和作者实践，不能全部当作官方规范。
- MCP、hooks、权限和自动化会带来安全风险，尤其是文件写入、shell、浏览器和凭据相关能力。
- 最有价值的不是复制所有配置，而是学会按职责拆分上下文、流程和工具。

## 推荐阅读顺序

1. `README.md`: 先建立全局地图。
2. `reports/claude-agent-command-skill.md`: 搞清楚 agent、command、skill 的边界。
3. `orchestration-workflow/orchestration-workflow.md`: 看完整编排例子。
4. `best-practice/claude-memory.md`: 设计自己的 `CLAUDE.md`。
5. `best-practice/claude-settings.md`: 配置权限、模型、hooks、MCP 和显示体验。
6. `reports/claude-skills-for-larger-mono-repos.md`: 如果你的项目是 monorepo，这篇很关键。
7. `development-workflows/cross-model-workflow/cross-model-workflow.md`: 学习 Claude Code + Codex 的互审流程。
