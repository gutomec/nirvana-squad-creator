# Nirvana Squad Creator

> 通过自然语言生成优化的 AIOS squad — 包含分析、生成、优化、验证、多语言 README、部署、CC Skill 和 squads.sh 发布的 11 阶段流水线。

## 安装

```bash
npx squads add gutomec/squads-sh-aios/nirvana-squad-creator
```

## 功能介绍

Nirvana Squad Creator 是一个**元工具**：一个能生成其他 AIOS squad 的 AIOS squad。通过自然语言描述目标，它能生成一个完整且优化的 squad，包括：

- **智能体** — 具备个性化特征、archetype 和 commands（AGENT-PERSONALIZATION-STANDARD-V1）
- **任务** — 具备明确的输入/输出契约（TASK-FORMAT-SPECIFICATION-V1）
- **工作流** — 自动选择模式和转换规则
- **配置** — 适配领域（coding-standards、tech-stack、source-tree）
- **README 文档** — 支持 6 种语言（PT-BR、en、zh、hi、es、ar）
- **CC Skill 包** — 可直接在 Claude Code 中使用
- **发布** — 发布到 squads.sh 市场

零冗余智能体。6 个类别的验证。自动部署并启用斜杠命令。

## 流水线 — 11 个阶段

| 阶段 | 智能体 | 职责 | 模型 |
|------|--------|------|------|
| 0 | 编排器 | 收集输入，初始化会话 | — |
| 1 | 🔍 Analyzer | 分析需求，生成 component-registry | Sonnet |
| 2 | 🏗️ AgentCreator | 生成 AIOS 智能体定义 | Opus |
| 3 | 📋 TaskCreator | 生成带有输入/输出契约的任务 | Opus |
| 4 | 🔄 WorkflowCreator | 生成工作流、squad.yaml、配置 | Opus |
| 5 | ⚡ Optimizer | AgentDropout、交叉引用、命名规范 | Opus |
| 6 | ✅ Validator | 6 类 AIOS 验证 | Sonnet |
| 7 | 🌐 ReadmeCreator | 6 种语言的 README | Opus |
| 8 | — Deploy | 部署到 AIOS 项目，启用命令 | 编排器 |
| 9 | 🎯 CCCreator | 转换为 CC Skill 包 | Opus |
| 10 | 🔭 SkillsScout | 搜索互补技能（可选） | Sonnet |
| 11 | 🚀 Publisher | 发布到 squads.sh（可选） | 编排器 |

## 智能体

| 图标 | 名称 | Archetype | 职责 |
|------|------|-----------|------|
| 🔍 | Analyzer | Guardian | 将目标分解为领域、能力和角色 |
| 🏗️ | AgentCreator | Builder | 生成带有 persona_profile 的智能体定义 |
| 📋 | TaskCreator | Builder | 生成带有链式输入/输出契约的任务 |
| 🔄 | WorkflowCreator | Flow_Master | 生成工作流、squad.yaml、配置和 README |
| ⚡ | Optimizer | Balancer | 消除冗余，修复交叉引用 |
| ✅ | Validator | Guardian | 对照 6 个 AIOS 规范类别进行验证 |
| 🌐 | ReadmeCreator | Builder | 生成 PT-BR 及 5 种翻译版本的 README |
| 🎯 | CCCreator | Builder | 将 AIOS squad 转换为 CC Skill 包 |
| 🔭 | SkillsScout | Guardian | 搜索社区互补技能 |
| 🚀 | Publisher | Flow_Master | 引导发布到 squads.sh 市场 |

## 任务

| 任务 | 负责人 | Atomic Layer |
|------|--------|-------------|
| `analyzeRequirements()` | Analyzer | Organism |
| `createAgents()` | AgentCreator | Organism |
| `createTasks()` | TaskCreator | Organism |
| `createWorkflows()` | WorkflowCreator | Organism |
| `optimizeSquad()` | Optimizer | Organism |
| `validateSquad()` | Validator | Organism |
| `createMultilingualReadme()` | ReadmeCreator | Organism |
| `deploySquad()` | 编排器 | Organism |
| `createCcSkill()` | CCCreator | Organism |
| `discoverSkills()` | SkillsScout | Molecule |
| `publishSquad()` | Publisher | Molecule |
| `manageState()` | 编排器 | Molecule |

## 工作流

### squad_generation_pipeline
11 阶段主流水线 — 从需求分析到发布。
```
[Analyzer] → [AgentCreator] → [TaskCreator] → [WorkflowCreator] → [Optimizer] → [Validator] → [ReadmeCreator] → Deploy → [CCCreator] → [SkillsScout] → [Publisher]
```

### squad_publish_flow
独立流程，用于将现有 squad 发布到 squads.sh。
```
[Validator] → [Publisher]
```

## 配置

- `config/coding-standards.md` — 命名规范、格式规则、语言
- `config/tech-stack.md` — Node.js、AIOS Core、Claude Code、YAML/Markdown
- `config/source-tree.md` — squad 目录结构

## 使用方法

### 完整流水线
```bash
/nsc:agents:squad-analyzer
```

### 单个智能体
```
/nsc:agents:squad-analyzer          — 需求分析
/nsc:agents:squad-agent-creator     — 智能体生成
/nsc:agents:squad-task-creator      — 任务生成
/nsc:agents:squad-workflow-creator  — 工作流和 squad.yaml
/nsc:agents:squad-optimizer         — 优化
/nsc:agents:squad-validator         — 验证
/nsc:agents:squad-readme-creator    — 多语言 README
/nsc:agents:squad-cc-creator        — CC Skill 包
/nsc:agents:squad-skills-scout      — 技能发现
/nsc:agents:squad-publisher         — 发布
```

## 作者

**Luiz Gustavo Vieira Rodrigues** ([@gutomec](https://github.com/gutomec))

## 许可证

MIT
