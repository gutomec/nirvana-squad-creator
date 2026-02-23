---
task: createCcSkill()
responsavel: "CCCreator"
responsavel_type: Agente
atomic_layer: Organism

Entrada:
  - nome: allWorkspaceFiles
    tipo: array<file>
    descricao: ".squad-workspace/<session>/ (todos os artefatos gerados)"
    obrigatorio: true
  - nome: ccSkillFormatMd
    tipo: file
    descricao: "references/cc-skill-format.md"
    obrigatorio: true
  - nome: targetProject
    tipo: string
    descricao: "deploySquad() task output (caminho do projeto destino)"
    obrigatorio: true

Saida:
  - nome: ccSkillDir
    tipo: file
    descricao: "cc-skills/<nome>/ (diretório completo do CC Skill)"
    obrigatorio: true
  - nome: ccSkillReportMd
    tipo: file
    descricao: "discoverSkills() task"
    obrigatorio: true

Checklist:
  pre-conditions:
    - "[ ] Squad deployado no projeto destino (fase 8 completa)"
    - "[ ] Referência cc-skill-format.md disponível"
    - "[ ] Todos os artefatos AIOS disponíveis no workspace"
  post-conditions:
    - "[ ] cc-skills/<nome>/ criado com estrutura completa"
    - "[ ] CLAUDE.md gerado com documentação do projeto"
    - "[ ] .claude/settings.json configurado"
    - "[ ] .claude/commands/<prefix>/agents/ com personas interativas"
    - "[ ] .claude/skills/<nome>/SKILL.md com pipeline de fases"
    - "[ ] .claude/skills/<nome>/agents/ com skill agents"
    - "[ ] .claude/skills/<nome>/references/ com docs de referência"
    - "[ ] .claude/rules/ com regras globais (code-standards.md obrigatório)"
    - "[ ] cc-skill-report.md gerado com mapeamento AIOS → CC Skill"

Performance:
  duration_expected: "3-8 minutos"
  cost_estimated: "~6000 tokens (Opus)"
  cacheable: false
  parallelizable: false
  skippable_when: "Usuário solicitar explicitamente skip de CC Skill"

Error Handling:
  strategy: retry
  retry:
    max_attempts: 2
    delay: "5s"
  fallback: "Gerar CC Skill com estrutura mínima (SKILL.md + agents) e marcar como incompleto"
  notification: "orchestrator"

Metadata:
  story: "Como projeto, preciso de um CC Skill package para habilitar slash commands e pipeline no Claude Code"
  version: "1.0.0"
  dependencies:
    - deploySquad()
  author: "Squad Creator"
  created_at: "2026-02-22T00:00:00Z"
  updated_at: "2026-02-22T00:00:00Z"
---

# createCcSkill()

## Pipeline Diagram

```
┌──────────────────────────┐  ┌──────────────────┐  ┌──────────────┐
│ .squad-workspace/<session>│  │ cc-skill-format  │  │ targetProject│
│ (todos os artefatos AIOS) │  │ .md              │  │ (path)       │
└────────────┬─────────────┘  └────────┬─────────┘  └──────┬───────┘
             │                         │                    │
             └─────────────┬───────────┘                    │
                           │                                │
                           ▼                                │
                  ┌─────────────────┐                       │
                  │   CCCreator      │◀─────────────────────┘
                  │  (squad-cc-      │
                  │   creator)       │
                  └────────┬─────────┘
                           │
         ┌─────────────────┼─────────────────────────┐
         │                 │                          │
         ▼                 ▼                          ▼
  ┌─────────────┐  ┌──────────────────────┐  ┌──────────────────┐
  │ CLAUDE.md   │  │ .claude/             │  │ cc-skill-report  │
  │ settings    │  │ ├── commands/        │  │ .md              │
  │ .json       │  │ │   └── <prefix>/   │  │                  │
  │             │  │ ├── skills/<nome>/   │  │ Mapeamento       │
  │             │  │ │   ├── SKILL.md    │  │ AIOS → CC Skill  │
  │             │  │ │   ├── agents/     │  │                  │
  │             │  │ │   └── references/ │  └──────────────────┘
  │             │  │ └── rules/          │
  └─────────────┘  └──────────────────────┘
```

## Descrição

A task `createCcSkill()` é a **nona fase** do pipeline. Transforma o squad AIOS em um CC Skill package — o formato nativo do Claude Code para skills autocontidos.

### Responsabilidades

1. **Mapeamento AIOS → CC Skill** — Transformar artefatos AIOS no formato CC:

   | AIOS Source | CC Skill Target | Lógica |
   |---|---|---|
   | `agents/*.md` | `.claude/skills/<nome>/agents/*.md` | Skill agents com YAML frontmatter CC |
   | `agents/*.md` | `.claude/commands/<prefix>/agents/*.md` | Personas interativas com saudação |
   | `workflows/*.yaml` | `.claude/skills/<nome>/SKILL.md` | Pipeline de fases com Task spawn |
   | `config/coding-standards.md` | `.claude/rules/code-standards.md` | Bullets acionáveis |
   | `config/tech-stack.md` + domínio | `.claude/rules/*.md` | Rules específicas do domínio |
   | `squad.yaml` + `README.md` | `CLAUDE.md` + `.claude/settings.json` | Documentação + config |

2. **Geração do SKILL.md** — Orchestrator do CC Skill:
   - Converter workflows AIOS em pipeline de fases
   - Cada fase referencia um skill agent via `Task` spawn
   - Definir inputs/outputs por fase
   - Incluir gate conditions entre fases

3. **Geração de Skill Agents** — Para cada agente AIOS:
   - Converter formato AIOS para formato CC (YAML frontmatter simplificado)
   - Incluir `persona`, `instructions`, `tools` no formato CC
   - Preservar `responsibility_boundaries` e `core_principles`

4. **Geração de Command Personas** — Versão interativa dos agentes:
   - Incluir `greeting_levels` para interação direta
   - Configurar visibilidade como slash commands
   - Formato: `.claude/commands/<prefix>/agents/<agent-id>.md`

5. **Geração de Rules** — Regras globais do CC Skill:
   - `code-standards.md` obrigatório (extraído de `config/coding-standards.md`)
   - Rules de domínio adicionais conforme tech-stack e domínio

6. **Relatório de Conversão** — `cc-skill-report.md`:
   - Tabela de mapeamento completa
   - Arquivos gerados com caminhos
   - Diferenças entre formato AIOS e CC
   - Recomendações de ajuste manual (se houver)

### Estrutura Final do CC Skill

```
cc-skills/<nome>/
├── CLAUDE.md
└── .claude/
    ├── settings.json
    ├── commands/<prefix>/agents/
    │   ├── <agent-1>.md
    │   └── <agent-N>.md
    ├── skills/<nome>/
    │   ├── SKILL.md
    │   ├── agents/
    │   │   ├── <agent-1>.md
    │   │   └── <agent-N>.md
    │   └── references/
    │       └── *.md
    └── rules/
        ├── code-standards.md
        └── *.md
```
