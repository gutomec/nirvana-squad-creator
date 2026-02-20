# Nirvana Squad Creator

Projeto Claude Code que gera squads AIOS otimizados a partir de linguagem natural. O usuario descreve o que precisa, e o sistema produz um squad completo com agents, tasks, workflows e squad.yaml validado.

Dado um objetivo em linguagem natural, gerar o squad AIOS mais otimizado possivel -- zero agentes redundantes, workflows inteligentes, estrutura 100% valida para o AIOS Core.

## Pipeline

| Fase | Agente | Papel | Modelo |
|------|--------|-------|--------|
| 1 | Analyzer | Analisa requisitos, identifica dominio, gera component registry | Sonnet |
| 2 | Agent Creator | Gera definicoes de agents com persona_profile e commands | Opus |
| 3 | Task Creator | Gera tasks com contratos Entrada/Saida e checklists | Opus |
| 4 | Workflow Creator | Gera workflows com transitions e selecao de pattern | Opus |
| 5 | Optimizer | Elimina redundancias (AgentDropout), otimiza model routing | Opus |
| 6 | Validator | Valida estrutura, cross-references, campos obrigatorios | Sonnet |

Fluxo: Input --> Analyzer --> Agent Creator --> Task Creator --> Workflow Creator --> Optimizer --> Validator --> Output

## Formatos AIOS -- Referencia Rapida

### squad.yaml

```yaml
name: my-squad          # kebab-case
version: 1.0.0          # semver
aios:
  minVersion: "2.1.0"
  type: squad
```

Required: `name`, `version`, `description`, `aios.minVersion`, `aios.type`, `components`

Referencia completa: `.claude/skills/create-squad/references/squad-yaml-schema.md`

### agent.md

```yaml
agent:
  name: AgentName       # nome legivel
  id: agent-id          # kebab-case
  title: Role Title
persona_profile:
  archetype: Builder     # Builder | Guardian | Balancer | Flow_Master
```

Required: `agent.name`, `agent.id`, `agent.title`, `agent.icon`, `agent.whenToUse`, `persona_profile.archetype`, `persona_profile.communication.tone`, `greeting_levels`

Referencia completa: `.claude/skills/create-squad/references/agent-format.md`

### task.md

```yaml
task: extractData()         # camelCase()
responsavel: "AgentName"
responsavel_type: Agente    # Agente | Worker | Humano | Clone
atomic_layer: Molecule      # Atom | Molecule | Organism | Template | Page
Entrada: [...]              # Input contracts
Saida: [...]                # Output contracts
```

Required: `task`, `responsavel`, `responsavel_type`, `atomic_layer`, `Entrada`, `Saida`, `Checklist`

Referencia completa: `.claude/skills/create-squad/references/task-format.md`

### workflow.yaml

```yaml
workflow_name: my_workflow    # snake_case
agent_sequence: [agent-a, agent-b]
success_indicators: [...]
transitions: {...}
```

Required: `workflow_name`, `description`, `agent_sequence`, `success_indicators`

Referencia completa: `.claude/skills/create-squad/references/workflow-format.md`

### config/

Tres arquivos Markdown freeform no diretorio `config/`:
- `coding-standards.md` -- Regras de estilo, naming, testes
- `tech-stack.md` -- Runtime, frameworks, dependencias
- `source-tree.md` -- Estrutura de diretorios do projeto-alvo

Referencia completa: `.claude/skills/create-squad/references/config-format.md`

## Convencoes

**Naming:** kebab-case para tudo (arquivos, IDs, nomes de squads, commands). Excecao: task identifiers usam camelCase().

**Workspace:** Cada sessao opera em `.squad-workspace/<session>/` com subdiretorios por fase (agents/, tasks/, workflows/, config/).

**File ownership:** Cada agente escreve em diretorio/arquivo proprio. O Optimizer e o unico que edita arquivos de outros agentes.

**Contexto:** Agentes recebem apenas inputs mapeados -- nem todos os outputs anteriores sao passados adiante. O Analyzer gera um `component-registry.md` com nomes canonicos que todos os agentes subsequentes recebem.

**Idioma:** Conteudo em PT-BR, nomes de variaveis em ingles. UTF-8 com acentuacao correta.

## Comandos & Estrutura

```bash
node bin/squad-tools.cjs init <session> [--preset=padrao]
node bin/squad-tools.cjs resume <session>
node bin/squad-tools.cjs state get <session>
node bin/squad-tools.cjs state advance <session> --phase=N [--notes="..."]
node bin/squad-tools.cjs state gate <session> --phase=N --result=approved
node bin/squad-tools.cjs validate <session> --phase=N
node bin/squad-tools.cjs snapshot <session>
```

```
nirvana-squad-creator/
├── CLAUDE.md
├── bin/squad-tools.cjs
├── .claude/skills/create-squad/
│   ├── templates/          # Templates anotados (squad.yaml, agent, task, workflow)
│   └── references/         # Specs completas por formato (5 docs)
└── .squad-workspace/       # Runtime workspace (gitignored)
    └── <session>/
        ├── config.json     # Estado da sessao
        ├── analysis.md     # Output do Analyzer
        ├── agents/         # Agent definitions
        ├── tasks/          # Task definitions
        ├── workflows/      # Workflow definitions
        └── config/         # Config files
```

## Links

- Templates: `.claude/skills/create-squad/templates/`
- References: `.claude/skills/create-squad/references/`
- CLI Tool: `bin/squad-tools.cjs`
- Workspace: `.squad-workspace/` (gitignored)
