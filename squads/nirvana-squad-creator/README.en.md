# Nirvana Squad Creator

> Generates optimized AIOS squads from natural language — 11-phase pipeline with analysis, generation, optimization, validation, multilingual READMEs, deployment, CC Skill and squads.sh publishing.

## Installation

```bash
npx squads add gutomec/nirvana-squad-creator
```

## What It Does

Nirvana Squad Creator is a **meta-tool**: an AIOS squad that generates other AIOS squads. From a natural language objective, it produces a complete, optimized squad with:

- **Agents** with personality, archetype and commands (AGENT-PERSONALIZATION-STANDARD-V1)
- **Tasks** with explicit Input/Output contracts (TASK-FORMAT-SPECIFICATION-V1)
- **Workflows** with automatic pattern selection and transitions
- **Config** tailored to the domain (coding-standards, tech-stack, source-tree)
- **READMEs** in 6 languages (PT-BR, en, zh, hi, es, ar)
- **CC Skill package** for direct use in Claude Code
- **Publishing** to the squads.sh marketplace

Zero redundant agents. Validation across 6 categories. Automatic deployment with slash command enablement.

## Pipeline — 11 Phases

| Phase | Agent | Role | Model |
|-------|-------|------|-------|
| 0 | Orchestrator | Collects input, initializes session | — |
| 1 | 🔍 Analyzer | Analyzes requirements, generates component-registry | Sonnet |
| 2 | 🏗️ AgentCreator | Generates AIOS agent definitions | Opus |
| 3 | 📋 TaskCreator | Generates tasks with Input/Output contracts | Opus |
| 4 | 🔄 WorkflowCreator | Generates workflows, squad.yaml, config | Opus |
| 5 | ⚡ Optimizer | AgentDropout, cross-references, naming | Opus |
| 6 | ✅ Validator | 6-category AIOS validation | Sonnet |
| 7 | 🌐 ReadmeCreator | READMEs in 6 languages | Opus |
| 8 | — Deploy | Deploys to AIOS project, enables commands | Orchestrator |
| 9 | 🎯 CCCreator | Transforms into CC Skill package | Opus |
| 10 | 🔭 SkillsScout | Searches for complementary skills (optional) | Sonnet |
| 11 | 🚀 Publisher | Publishes to squads.sh (optional) | Orchestrator |

## Agents

| Icon | Name | Archetype | Responsibility |
|------|------|-----------|----------------|
| 🔍 | Analyzer | Guardian | Decomposes objective into domain, capabilities and roles |
| 🏗️ | AgentCreator | Builder | Generates agent definitions with persona_profile |
| 📋 | TaskCreator | Builder | Generates tasks with chained Input/Output contracts |
| 🔄 | WorkflowCreator | Flow_Master | Generates workflows, squad.yaml, config and README |
| ⚡ | Optimizer | Balancer | Eliminates redundancies, fixes cross-references |
| ✅ | Validator | Guardian | Validates against 6 AIOS specification categories |
| 🌐 | ReadmeCreator | Builder | Generates READMEs in PT-BR + 5 translations |
| 🎯 | CCCreator | Builder | Transforms AIOS squad into CC Skill package |
| 🔭 | SkillsScout | Guardian | Searches for complementary community skills |
| 🚀 | Publisher | Flow_Master | Guides publishing to the squads.sh marketplace |

## Tasks

| Task | Owner | Atomic Layer |
|------|-------|-------------|
| `analyzeRequirements()` | Analyzer | Organism |
| `createAgents()` | AgentCreator | Organism |
| `createTasks()` | TaskCreator | Organism |
| `createWorkflows()` | WorkflowCreator | Organism |
| `optimizeSquad()` | Optimizer | Organism |
| `validateSquad()` | Validator | Organism |
| `createMultilingualReadme()` | ReadmeCreator | Organism |
| `deploySquad()` | Orchestrator | Organism |
| `createCcSkill()` | CCCreator | Organism |
| `discoverSkills()` | SkillsScout | Molecule |
| `publishSquad()` | Publisher | Molecule |
| `manageState()` | Orchestrator | Molecule |

## Workflows

### squad_generation_pipeline
Main 11-phase pipeline — from requirements analysis to publishing.
```
[Analyzer] → [AgentCreator] → [TaskCreator] → [WorkflowCreator] → [Optimizer] → [Validator] → [ReadmeCreator] → Deploy → [CCCreator] → [SkillsScout] → [Publisher]
```

### squad_publish_flow
Standalone flow to publish an existing squad to squads.sh.
```
[Validator] → [Publisher]
```

## Configuration

- `config/coding-standards.md` — Naming conventions, format rules, language
- `config/tech-stack.md` — Node.js, AIOS Core, Claude Code, YAML/Markdown
- `config/source-tree.md` — Squad directory structure

## Usage

### Full pipeline
```bash
/nsc:agents:squad-analyzer
```

### Individual agents
```
/nsc:agents:squad-analyzer          — Requirements analysis
/nsc:agents:squad-agent-creator     — Agent generation
/nsc:agents:squad-task-creator      — Task generation
/nsc:agents:squad-workflow-creator  — Workflows and squad.yaml
/nsc:agents:squad-optimizer         — Optimization
/nsc:agents:squad-validator         — Validation
/nsc:agents:squad-readme-creator    — Multilingual READMEs
/nsc:agents:squad-cc-creator        — CC Skill package
/nsc:agents:squad-skills-scout      — Skills discovery
/nsc:agents:squad-publisher         — Publishing
```

## Author

**Luiz Gustavo Vieira Rodrigues** ([@gutomec](https://github.com/gutomec))

## License

MIT
