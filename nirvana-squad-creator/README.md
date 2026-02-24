# Nirvana Squad Creator

> Gera squads AIOS otimizados a partir de linguagem natural — pipeline de 11 fases com análise, geração, otimização, validação, README multi-idioma, deploy, CC Skill e publicação no squads.sh.

## Instalação

```bash
npx squads add gutomec/squads-sh-aios/nirvana-squad-creator
```

## O que Faz

O Nirvana Squad Creator é uma **meta-ferramenta**: um squad AIOS que gera outros squads AIOS. A partir de um objetivo em linguagem natural, ele produz um squad completo e otimizado com:

- **Agentes** com personalidade, archetype e commands (AGENT-PERSONALIZATION-STANDARD-V1)
- **Tasks** com contratos explícitos de Entrada/Saída (TASK-FORMAT-SPECIFICATION-V1)
- **Workflows** com seleção automática de pattern e transitions
- **Config** adaptado ao domínio (coding-standards, tech-stack, source-tree)
- **READMEs** em 6 idiomas (PT-BR, en, zh, hi, es, ar)
- **CC Skill package** para uso direto no Claude Code
- **Publicação** no marketplace squads.sh

Zero agentes redundantes. Validação em 6 categorias. Deploy automático com habilitação de slash commands.

## Pipeline — 11 Fases

| Fase | Agente | Papel | Modelo |
|------|--------|-------|--------|
| 0 | Orquestrador | Coleta input, inicializa sessão | — |
| 1 | 🔍 Analyzer | Analisa requisitos, gera component-registry | Sonnet |
| 2 | 🏗️ AgentCreator | Gera definições de agents AIOS | Opus |
| 3 | 📋 TaskCreator | Gera tasks com contratos Entrada/Saída | Opus |
| 4 | 🔄 WorkflowCreator | Gera workflows, squad.yaml, config | Opus |
| 5 | ⚡ Optimizer | AgentDropout, cross-references, naming | Opus |
| 6 | ✅ Validator | Validação 6 categorias AIOS | Sonnet |
| 7 | 🌐 ReadmeCreator | READMEs em 6 idiomas | Opus |
| 8 | — Deploy | Deploya em projeto AIOS, habilita commands | Orquestrador |
| 9 | 🎯 CCCreator | Transforma em CC Skill package | Opus |
| 10 | 🔭 SkillsScout | Busca skills complementares (opcional) | Sonnet |
| 11 | 🚀 Publisher | Publica no squads.sh (opcional) | Orquestrador |

## Agentes

| Icon | Nome | Archetype | Responsabilidade |
|------|------|-----------|------------------|
| 🔍 | Analyzer | Guardian | Decompõe objetivo em domínio, capacidades e roles |
| 🏗️ | AgentCreator | Builder | Gera definições de agentes com persona_profile |
| 📋 | TaskCreator | Builder | Gera tasks com contratos Entrada/Saída encadeados |
| 🔄 | WorkflowCreator | Flow_Master | Gera workflows, squad.yaml, config e README |
| ⚡ | Optimizer | Balancer | Elimina redundâncias, corrige cross-references |
| ✅ | Validator | Guardian | Valida contra 6 categorias de especificação AIOS |
| 🌐 | ReadmeCreator | Builder | Gera READMEs em PT-BR + 5 traduções |
| 🎯 | CCCreator | Builder | Transforma squad AIOS em CC Skill package |
| 🔭 | SkillsScout | Guardian | Busca skills complementares da comunidade |
| 🚀 | Publisher | Flow_Master | Guia publicação no squads.sh marketplace |

## Tasks

| Task | Responsável | Atomic Layer |
|------|-------------|-------------|
| `analyzeRequirements()` | Analyzer | Organism |
| `createAgents()` | AgentCreator | Organism |
| `createTasks()` | TaskCreator | Organism |
| `createWorkflows()` | WorkflowCreator | Organism |
| `optimizeSquad()` | Optimizer | Organism |
| `validateSquad()` | Validator | Organism |
| `createMultilingualReadme()` | ReadmeCreator | Organism |
| `deploySquad()` | Orquestrador | Organism |
| `createCcSkill()` | CCCreator | Organism |
| `discoverSkills()` | SkillsScout | Molecule |
| `publishSquad()` | Publisher | Molecule |
| `manageState()` | Orquestrador | Molecule |

## Workflows

### squad_generation_pipeline
Pipeline principal de 11 fases — da análise de requisitos à publicação.
```
[Analyzer] → [AgentCreator] → [TaskCreator] → [WorkflowCreator] → [Optimizer] → [Validator] → [ReadmeCreator] → Deploy → [CCCreator] → [SkillsScout] → [Publisher]
```

### squad_publish_flow
Fluxo standalone para publicar um squad existente no squads.sh.
```
[Validator] → [Publisher]
```

## Configuração

- `config/coding-standards.md` — Naming conventions, regras de formato, linguagem
- `config/tech-stack.md` — Node.js, AIOS Core, Claude Code, YAML/Markdown
- `config/source-tree.md` — Estrutura de diretórios do squad

## Uso

### Pipeline completo
```bash
/nsc:agents:squad-analyzer
```

### Agentes individuais
```
/nsc:agents:squad-analyzer          — Análise de requisitos
/nsc:agents:squad-agent-creator     — Geração de agentes
/nsc:agents:squad-task-creator      — Geração de tasks
/nsc:agents:squad-workflow-creator  — Workflows e squad.yaml
/nsc:agents:squad-optimizer         — Otimização
/nsc:agents:squad-validator         — Validação
/nsc:agents:squad-readme-creator    — READMEs multi-idioma
/nsc:agents:squad-cc-creator        — CC Skill package
/nsc:agents:squad-skills-scout      — Skills discovery
/nsc:agents:squad-publisher         — Publicação
```

## Autor

**Luiz Gustavo Vieira Rodrigues** ([@gutomec](https://github.com/gutomec))

## Licença

MIT
