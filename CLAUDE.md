# Nirvana Squad Creator

Projeto Claude Code que gera squads AIOS otimizados a partir de linguagem natural. O usuario descreve o que precisa, e o sistema produz um squad completo com agents, tasks, workflows e squad.yaml validado.

Dado um objetivo em linguagem natural, gerar o squad AIOS mais otimizado possivel -- zero agentes redundantes, workflows inteligentes, estrutura 100% valida para o AIOS Core.

## Pipeline

| Fase | Agente | Papel | Modelo |
|------|--------|-------|--------|
| 1 | Analyzer | Analisa requisitos, identifica domínio, gera component registry | Sonnet |
| 2 | Agent Creator | Gera definições de agents com persona_profile e commands | Opus |
| 3 | Task Creator | Gera tasks com contratos Entrada/Saída e checklists | Opus |
| 4 | Workflow Creator | Gera workflows com transitions e seleção de pattern | Opus |
| 5 | Optimizer | Elimina redundâncias (AgentDropout), otimiza model routing | Opus |
| 6 | Validator | Valida estrutura, cross-references, campos obrigatórios | Sonnet |
| 7 | README Creator | Gera READMEs em 6 idiomas (PT-BR + en, zh, hi, es, ar) | Opus |
| 8 | Deploy | Deploya squad em projeto AIOS (novo ou existente) e habilita slash commands | Orquestrador |
| 9 | Publisher | Guia publicação no squads.sh marketplace (opcional) | Orquestrador |

Fluxo: Input --> Analyzer --> Agent Creator --> Task Creator --> Workflow Creator --> Optimizer --> Validator --> README Creator --> Deploy + Habilitação --> Publisher (opcional)

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

## Deploy e Habilitação de Squads

Após a geração, validação e READMEs do squad (Fases 1-7), a Fase 8 faz o deploy em um projeto AIOS e habilita os slash commands no Claude Code.

### Opções de Deploy

| Opção | Descrição |
|-------|-----------|
| **Novo projeto AIOS** | Cria diretório, roda `npx aios-core init`, deploya squad e habilita commands |
| **Projeto AIOS existente** | Usuário informa caminho, valida `.aios-core/`, deploya squad e habilita commands |

### Mecanismo de Habilitação de Slash Commands

O AIOS Core registra slash commands no Claude Code via a estrutura `.claude/commands/SQUADS/{prefix}/`. O processo:

1. Ler `slashPrefix` do `squad.yaml` (ex: `ds` → commands ficam em `/SQUADS:ds:*`)
2. Copiar agents do squad para `.claude/commands/SQUADS/{prefix}/`
3. Criar/atualizar `.aios-sync.yaml` na raiz do projeto com o alias do squad

Após isso, o Claude Code reconhece os comandos automaticamente:
```
/SQUADS:{prefix}:{agent-id}   → Ativa o agente correspondente
```

### Estrutura de Deploy

```
<projeto-aios>/
├── .aios-core/                    # AIOS Core (já existente ou recém-instalado)
├── .claude/
│   ├── squads/
│   │   └── <nome>/agents/             # ← Agentes instalados (IDE path)
│   └── commands/
│       ├── AIOS/agents/           # Agentes core do AIOS
│       └── SQUADS/<prefix>/       # ← Slash commands habilitados
├── .squad-lock.json                   # ← Registro de squads instalados
├── .aios-sync.yaml                # ← Mapeamento squad → prefix
└── squads/
    └── <nome-do-squad>/           # ← Squad deployado aqui
        ├── agents/
        ├── tasks/
        ├── workflows/
        ├── config/
        ├── squad.yaml
        └── README.md
```

### .aios-sync.yaml

Arquivo de configuração que mapeia squads para prefixos de slash commands:

```yaml
active_ides:
  - claude
squad_aliases:
  meu-squad: ms        # /SQUADS:ms:*
  outro-squad: os       # /SQUADS:os:*
sync_mappings:
  squad_agents:
    source: 'squads/*/agents/'
    destinations:
      claude:
        - path: '.claude/commands/SQUADS/{squad_alias}/'
          format: 'md'
```

Para re-sincronizar manualmente após editar agentes:
```bash
# Copiar agents atualizados para .claude/commands/
cp squads/<nome>/agents/*.md .claude/commands/SQUADS/<prefix>/
```

## Auto-Melhoria do Squad Creator

O skill `/improve-squad-creator` opera **independente** do pipeline de geração. Ele busca skills que melhorem o próprio Squad Creator — não os squads gerados.

### Uso

```bash
/improve-squad-creator                    # Busca geral
/improve-squad-creator --domain=validation  # Foco em validação
```

### Fluxo

1. Escaneia capacidades atuais do Squad Creator (agentes, templates, references)
2. Identifica gaps e áreas de melhoria
3. Busca skills via API Skyll
4. Apresenta relatório com recomendações
5. Instala skills aprovados pelo usuário (via `npx skills add`)
6. Sugere integrações (sem auto-modificar arquivos)

### Agente

- **self-improvement-scout** — Busca e avalia skills para auto-melhoria
- Localização: `.claude/skills/improve-squad-creator/agents/self-improvement-scout.md`

## Convenções

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
node bin/squad-tools.cjs validate <session> --phase=N    # Fases válidas: 1-9
node bin/squad-tools.cjs snapshot <session>
```

```
nirvana-squad-creator/
├── CLAUDE.md
├── bin/squad-tools.cjs
├── .claude/skills/create-squad/
│   ├── SKILL.md            # Orquestrador (Fases 0-9)
│   ├── templates/          # Templates anotados (squad.yaml, agent, task, workflow)
│   └── references/         # Specs completas por formato (5 docs)
├── .claude/skills/improve-squad-creator/
│   ├── SKILL.md            # Orquestrador de auto-melhoria
│   └── agents/             # self-improvement-scout
├── .claude/agents/
│   ├── squad-analyzer.md
│   ├── squad-agent-creator.md
│   ├── squad-task-creator.md
│   ├── squad-workflow-creator.md
│   ├── squad-optimizer.md
│   ├── squad-validator.md
│   ├── squad-readme-creator.md # Fase 7
│   └── squad-publisher.md      # Fase 9
└── .squad-workspace/       # Runtime workspace (gitignored)
    └── <session>/
        ├── config.json     # Estado da sessão
        ├── analysis.md     # Output do Analyzer
        ├── agents/         # Agent definitions
        ├── tasks/          # Task definitions
        ├── workflows/      # Workflow definitions
        └── config/         # Config files

# Output (projeto AIOS destino -- novo ou existente):
<projeto-aios>/
├── .aios-core/             # AIOS Core framework
├── .claude/
│   ├── squads/<nome>/agents/   # Agentes instalados (IDE path)
│   └── commands/SQUADS/<prefix>/  # Slash commands habilitados
├── .squad-lock.json        # Registro de squads instalados
├── .aios-sync.yaml         # Mapeamento squad → prefix
├── squads/<nome>/          # Squad AIOS deployado
│   ├── agents/*.md
│   ├── tasks/*.md
│   ├── workflows/*.yaml
│   ├── config/*.md
│   ├── squad.yaml
│   └── README.md
```

## Links

- Templates: `.claude/skills/create-squad/templates/`
- References: `.claude/skills/create-squad/references/`
- CLI Tool: `bin/squad-tools.cjs`
- Workspace: `.squad-workspace/` (gitignored)

## AIOS Squad Package

O Nirvana Squad Creator também existe como squad AIOS publicável em `squads/nirvana-squad-creator/`.

```
squads/nirvana-squad-creator/
├── agents/          # 9 agentes AIOS
├── tasks/           # 10 tasks com contratos
├── workflows/       # 2 workflows
├── config/          # 3 config files
├── scripts/         # squad-tools.cjs
├── templates/       # 4 templates
├── references/      # 5 referências
├── squad.yaml       # Manifesto
├── README.md        # PT-BR (+ 5 traduções)
└── README.*.md      # en, zh, hi, es, ar
```
