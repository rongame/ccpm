# Claude Code PM

[![Automaze](https://img.shields.io/badge/By-automaze.io-4b3baf)](https://automaze.io)
&nbsp;
[![Claude Code](https://img.shields.io/badge/+-Claude%20Code-d97757)](https://github.com/automazeio/ccpm/blob/main/README.md)
[![GitHub Issues](https://img.shields.io/badge/+-GitHub%20Issues-1f2328)](https://github.com/automazeio/ccpm)
&nbsp;
[![MIT License](https://img.shields.io/badge/License-MIT-28a745)](https://github.com/automazeio/ccpm/blob/main/LICENSE)
&nbsp;
[![Follow on 𝕏](https://img.shields.io/badge/𝕏-@aroussi-1c9bf0)](http://x.com/intent/follow?screen_name=aroussi)
&nbsp;
[![Star this repo](https://img.shields.io/badge/★-Star%20this%20repo-e7b10b)](https://github.com/automazeio/ccpm)

### Claude Code workflow to ship ~~faster~~ _better_ using spec-driven development, GitHub issues, Git worktrees, and mutiple AI agents running in parallel.

Stop losing context. Stop blocking on tasks. Stop shipping bugs. This battle-tested system turns PRDs into epics, epics into GitHub issues, and issues into production code – with full traceability at every step.

![Claude Code PM](screenshot.webp)

# Claude 命令系统完整指南

本文档详细介绍了 `.claude/commands` 目录下所有命令的功能、用法和使用场景。

## 目录结构

```
.claude/commands/
├── code-rabbit.md          # CodeRabbit 代码审查处理
├── prompt.md               # 复杂提示词处理
├── re-init.md             # 重新初始化 CLAUDE.md
├── context/               # 项目上下文管理
│   ├── create.md          # 创建初始项目上下文
│   ├── prime.md           # 加载项目上下文
│   └── update.md          # 更新项目上下文
├── pm/                    # 项目管理命令
│   ├── help.md            # 显示帮助信息
│   ├── init.md            # 初始化项目管理
│   ├── status.md          # 查看项目状态
│   ├── epic-*.md          # Epic 相关命令
│   ├── issue-*.md         # Issue 相关命令
│   ├── prd-*.md           # PRD 相关命令
│   └── 其他管理命令...
└── testing/               # 测试相关命令
    ├── prime.md           # 配置测试环境
    └── run.md             # 运行测试
```

---

## 1. 根目录命令

### 1.1 `/code-rabbit` - CodeRabbit 代码审查处理器

**功能**: 智能处理 CodeRabbit 代码审查评论，根据项目上下文决定接受或忽略建议。

**使用方法**:
```bash
/code-rabbit
```
然后粘贴一个或多个 CodeRabbit 评论。

**主要特性**:
- **上下文感知**: 考虑完整代码库上下文，而不是孤立地处理建议
- **智能筛选**: 自动区分有效建议和不适用的通用建议
- **并行处理**: 对于多文件评论，启动并行子代理处理
- **批量应用**: 使用 MultiEdit 工具批量应用接受的更改

**处理流程**:
1. 分析评论是否涉及单个文件或多个文件
2. 读取相关文件获取上下文
3. 评估每个建议的有效性
4. 应用接受的更改并报告决策理由

**常见忽略的建议类型**:
- 与项目约定冲突的样式偏好
- 不适用于特定用例的通用最佳实践
- 非性能关键代码的性能优化
- 内部工具的可访问性建议

**常见接受的建议类型**:
- 实际的 bug（空值检查、错误处理）
- 安全漏洞
- 资源泄漏
- 类型安全问题
- 逻辑错误

### 1.2 `/prompt` - 复杂提示词处理

**功能**: 处理包含大量 @ 引用的复杂提示词，避免直接输入失败。

**使用方法**:
1. 将复杂的提示词写入此文件
2. 在提示命令中输入 `/prompt`

**适用场景**:
- 提示词包含多个文件引用
- 提示词过长或复杂
- 直接输入提示词时出现失败

### 1.3 `/re-init` - 重新初始化 CLAUDE.md

**功能**: 更新或创建 CLAUDE.md 文件，整合 `.claude/CLAUDE.md` 中的规则。

**使用方法**:
```bash
/re-init
```

**执行逻辑**:
- 如果 CLAUDE.md 存在，则更新其内容
- 如果不存在，则创建新文件并包含 `.claude/CLAUDE.md` 的规则

---

## 2. Context 上下文管理命令

### 2.1 `/context:create` - 创建初始项目上下文

**功能**: 分析当前项目状态，创建全面的基线文档。

**使用方法**:
```bash
/context:create
```

**预检查项**:
1. **上下文目录检查**: 检查是否存在现有上下文文件
2. **项目类型检测**: 自动识别 Node.js、Python、Rust、Go 等项目类型
3. **Git 仓库验证**: 确认这是一个 Git 仓库
4. **权限检查**: 验证写入权限

**创建的文件**:
- `progress.md` - 当前项目状态、已完成工作和下一步计划
- `project-structure.md` - 目录结构和文件组织映射
- `tech-context.md` - 技术栈、依赖和开发工具目录
- `system-patterns.md` - 架构模式和设计决策
- `product-context.md` - 产品需求、目标用户和核心功能
- `project-brief.md` - 项目范围、目标和关键指标
- `project-overview.md` - 功能和能力的高级概述
- `project-vision.md` - 长期愿景和战略方向
- `project-style-guide.md` - 编码标准、约定和样式偏好

**每个文件包含的前置元数据**:
```yaml
---
created: [真实日期时间]
last_updated: [真实日期时间]
version: 1.0
author: Claude Code PM System
---
```

### 2.2 `/context:prime` - 加载项目上下文

**功能**: 为新的代理会话加载基本上下文，理解代码库结构。

**使用方法**:
```bash
/context:prime
```

**加载优先级**:
1. **优先级 1 - 基本上下文**:
   - `project-overview.md` - 项目高级理解
   - `project-brief.md` - 核心目的和目标
   - `tech-context.md` - 技术栈和依赖

2. **优先级 2 - 当前状态**:
   - `progress.md` - 当前状态和最近工作
   - `project-structure.md` - 目录和文件组织

3. **优先级 3 - 深度上下文**:
   - `system-patterns.md` - 架构和设计模式
   - `product-context.md` - 用户需求和要求
   - `project-style-guide.md` - 编码约定
   - `project-vision.md` - 长期方向

**错误恢复**:
- 如果关键文件缺失，尝试从 README.md 理解
- 如果技术上下文缺失，直接分析 package.json/requirements.txt
- 提供部分上下文但注明限制

### 2.3 `/context:update` - 更新项目上下文

**功能**: 更新项目上下文文档以反映当前项目状态。建议在每个开发会话结束时运行。

**使用方法**:
```bash
/context:update
```

**更新策略**:
- **progress.md** - 总是更新（最近提交、当前分支、未提交更改）
- **project-structure.md** - 结构变化时更新
- **tech-context.md** - 依赖变化时更新
- **system-patterns.md** - 架构变化时更新
- **product-context.md** - 需求变化时更新
- **project-brief.md** - 很少更新（基本目标变化）
- **project-overview.md** - 重大里程碑时更新
- **project-vision.md** - 很少更新（战略方向变化）
- **project-style-guide.md** - 约定变化时更新

**智能更新**:
- 只更新有实际变化的文件
- 保留准确的时间戳
- 进行针对性更新，不重写整个文件
- 在重大更新时添加更新历史

---

## 3. PM 项目管理命令

### 3.1 基础管理命令

#### `/pm:help` - 显示帮助信息
**功能**: 运行帮助脚本显示完整的项目管理命令列表。

#### `/pm:init` - 初始化项目管理
**功能**: 初始化项目管理系统，设置必要的目录结构和配置。

#### `/pm:status` - 查看项目状态
**功能**: 显示当前项目的整体状态，包括进行中的 Epic、Issue 和任务。

#### `/pm:sync` - 全双向同步
**功能**: 在本地和 GitHub 之间进行完整的双向同步。

**使用方法**:
```bash
/pm:sync [epic_name]  # 同步特定 epic
/pm:sync              # 同步所有
```

**同步流程**:
1. 从 GitHub 拉取当前状态
2. 根据 GitHub 更新本地文件
3. 将本地更改推送到 GitHub
4. 处理冲突（显示两个版本供选择）

### 3.2 Epic 管理命令

#### `/pm:epic-start <epic_name>` - 启动 Epic 执行
**功能**: 在共享分支中启动并行代理来处理 epic 任务。

**使用方法**:
```bash
/pm:epic-start user-authentication
```

**执行流程**:
1. **验证 Epic 存在**: 检查 `.claude/epics/$ARGUMENTS/epic.md`
2. **检查 GitHub 同步**: 验证 epic 前置元数据中的 `github:` 字段
3. **分支管理**: 创建或切换到 `epic/$ARGUMENTS` 分支
4. **识别就绪任务**: 分析依赖关系，找出可以开始的任务
5. **启动并行代理**: 为每个就绪任务的工作流启动代理
6. **跟踪执行**: 创建执行状态文档监控进度

**并行代理配置**:
- 每个代理处理特定的工作流（如数据库、API、UI）
- 所有代理在同一分支工作
- 遵循 `/rules/agent-coordination.md` 协调规则
- 频繁提交，使用格式：`Issue #{issue}: {specific change}`

#### `/pm:epic-list` - 列出所有 Epic
**功能**: 显示所有可用的 Epic 及其状态。

#### `/pm:epic-status <epic_name>` - 查看 Epic 状态
**功能**: 显示特定 Epic 的详细执行状态。

**其他 Epic 命令**:
- `/pm:epic-close` - 关闭 Epic
- `/pm:epic-decompose` - 分解 Epic 为任务
- `/pm:epic-edit` - 编辑 Epic
- `/pm:epic-merge` - 合并 Epic 分支
- `/pm:epic-oneshot` - 一次性 Epic 执行
- `/pm:epic-refresh` - 刷新 Epic 状态
- `/pm:epic-show` - 显示 Epic 详情
- `/pm:epic-start-worktree` - 在工作树中启动 Epic
- `/pm:epic-sync` - 同步 Epic 到 GitHub

### 3.3 Issue 管理命令

#### `/pm:issue-start <issue_number>` - 开始处理 Issue
**功能**: 基于工作流分析，使用并行代理开始处理 GitHub Issue。

**使用方法**:
```bash
/pm:issue-start 1234
```

**预检查**:
1. 获取 Issue 详情（通过 `gh issue view`）
2. 查找本地任务文件
3. 检查分析文件是否存在
4. 确保 Epic 工作树存在

**其他 Issue 命令**:
- `/pm:issue-analyze` - 分析 Issue
- `/pm:issue-close` - 关闭 Issue
- `/pm:issue-edit` - 编辑 Issue
- `/pm:issue-reopen` - 重新打开 Issue
- `/pm:issue-show` - 显示 Issue 详情
- `/pm:issue-status` - 查看 Issue 状态
- `/pm:issue-sync` - 同步 Issue 到 GitHub

### 3.4 PRD 产品需求文档命令

#### `/pm:prd-new <feature_name>` - 创建新 PRD
**功能**: 启动新产品需求文档的头脑风暴。

**使用方法**:
```bash
/pm:prd-new user-authentication
```

**验证要求**:
- 功能名称必须是 kebab-case 格式（小写字母、数字、连字符）
- 必须以字母开头
- 不允许空格或特殊字符

**创建流程**:
1. 发现和上下文收集
2. 理解要解决的问题
3. 结构化 PRD 创建
4. 包含用户故事、技术要求、成功指标

#### `/pm:prd-parse <feature_name>` - 解析 PRD 为 Epic
**功能**: 将 PRD 转换为技术实现 Epic。

**使用方法**:
```bash
/pm:prd-parse user-authentication
```

**转换流程**:
1. 验证 PRD 存在且有效
2. 分析产品需求
3. 创建技术实现计划
4. 生成 Epic 和相关任务
5. 建立依赖关系

**其他 PRD 命令**:
- `/pm:prd-edit` - 编辑 PRD
- `/pm:prd-list` - 列出所有 PRD
- `/pm:prd-status` - 查看 PRD 状态

### 3.5 其他管理命令

- `/pm:blocked` - 查看被阻塞的任务
- `/pm:clean` - 清理项目管理文件
- `/pm:import` - 导入外部任务
- `/pm:in-progress` - 查看进行中的任务
- `/pm:next` - 查看下一个任务
- `/pm:search` - 搜索任务和 Epic
- `/pm:standup` - 生成站会报告
- `/pm:test-reference-update` - 更新测试引用
- `/pm:validate` - 验证项目管理数据

---

## 4. Testing 测试命令

### 4.1 `/testing:prime` - 配置测试环境

**功能**: 检测测试框架、验证依赖并配置测试运行器代理以优化测试执行。

**使用方法**:
```bash
/testing:prime
```

**支持的框架**:
- **JavaScript/Node.js**: Jest, Mocha
- **Python**: Pytest, unittest
- **Rust**: Cargo test
- **Go**: go test
- **Ruby**: RSpec
- **Java**: JUnit

**检测流程**:
1. **框架检测**: 扫描配置文件和依赖
2. **依赖验证**: 确认测试框架已安装
3. **测试发现**: 统计测试文件和模式
4. **配置创建**: 生成 `.claude/testing-config.md`

**代理配置原则**:
- 使用详细输出进行调试
- 顺序执行测试（无并行）
- 捕获完整堆栈跟踪
- 不使用模拟 - 使用真实实现
- 等待每个测试完成

### 4.2 `/testing:run [test_target]` - 运行测试

**功能**: 使用配置的测试运行器代理执行测试。

**使用方法**:
```bash
/testing:run              # 运行所有测试
/testing:run user.test.js # 运行特定测试文件
/testing:run auth         # 运行匹配模式的测试
```

**执行特性**:
- 使用 `.claude/agents/test-runner.md` 中的测试运行器代理
- 详细输出用于调试
- 不使用模拟服务
- 捕获完整输出包括堆栈跟踪
- 如果测试失败，在假设代码问题之前检查测试结构

**结果报告**:
- 成功：显示通过的测试数量和执行时间
- 失败：显示失败详情和建议修复
- 混合：显示通过、失败和跳过的统计

---

## 5. 使用最佳实践

### 5.1 项目初始化流程
1. 运行 `/context:create` 建立项目基线
2. 运行 `/pm:init` 初始化项目管理
3. 运行 `/testing:prime` 配置测试环境

### 5.2 日常开发流程
1. 会话开始时运行 `/context:prime` 加载上下文
2. 使用 `/pm:status` 查看当前状态
3. 使用相应的 PM 命令处理任务
4. 会话结束时运行 `/context:update` 更新上下文

### 5.3 Epic 和 Issue 管理
1. 使用 `/pm:prd-new` 创建产品需求
2. 使用 `/pm:prd-parse` 转换为技术 Epic
3. 使用 `/pm:epic-start` 启动并行开发
4. 使用 `/pm:epic-status` 监控进度
5. 使用 `/pm:sync` 保持与 GitHub 同步

### 5.4 测试管理
1. 项目初期运行 `/testing:prime` 一次
2. 使用 `/testing:run` 执行测试
3. 测试失败时检查测试结构而非立即假设代码问题

### 5.5 代码审查
1. 收到 CodeRabbit 评论时使用 `/code-rabbit`
2. 信任对代码库的理解胜过通用建议
3. 审查决策以维护审计跟踪

---

## 6. 注意事项

### 6.1 通用注意事项
- 所有命令都使用真实的系统时间，从不使用占位符
- 在覆盖现有文件之前总是请求确认
- 优雅地处理错误并提供具体指导
- 保持输出专注于失败和重要信息

### 6.2 并行代理协调
- 遵循 `/rules/agent-coordination.md` 中的协调规则
- 最大并行代理数应合理（例如 5-10 个）
- 如果启动多个代理，监控系统资源
- 代理在同一分支工作，不是单独的分支

### 6.3 Git 操作
- 遵循 `/rules/branch-operations.md` 进行 Git 操作
- 在启动 Epic 之前检查未提交的更改
- 频繁提交并使用描述性消息
- 保持分支整洁和有组织

### 6.4 测试执行
- 总是使用详细输出
- 检查测试结构如果测试失败
- 使用真实服务，不使用模拟
- 让每个测试完全完成

---

## 7. 故障排除

### 7.1 常见问题

**上下文相关**:
- "❌ 未找到上下文" → 运行 `/context:create`
- "⚠️ 上下文文件损坏" → 重新运行 `/context:create`

**项目管理相关**:
- "❌ Epic 未找到" → 运行 `/pm:prd-parse <epic_name>`
- "❌ Epic 未同步" → 运行 `/pm:epic-sync <epic_name>`
- "❌ 有未提交更改" → 提交或储藏更改

**测试相关**:
- "❌ 未配置测试" → 运行 `/testing:prime`
- "❌ 未安装测试依赖" → 安装依赖（npm install 等）

### 7.2 权限问题
- 确保对 `.claude/` 目录有写权限
- 检查 Git 仓库权限
- 验证 GitHub CLI 认证（`gh auth login`）

### 7.3 性能优化
- 对于大型项目，某些命令可能需要更长时间
- 并行代理启动时监控系统资源
- 定期清理旧的分支和工作树

---

这个命令系统提供了一个完整的项目管理和开发工作流，从项目初始化到代码审查，从测试执行到 Epic 管理。通过合理使用这些命令，可以显著提高开发效率和项目管理质量。
---

## Star History

![Star History Chart](https://api.star-history.com/svg?repos=automazeio/ccpm)
