# Claude Code Skill — Formato de Referência

Especificação completa do formato de um **CC Skill package** — o output nativo do Claude Code gerado a partir de um squad AIOS. Este documento é a fonte de verdade para o agente `squad-cc-creator` (Fase 8).

---

## Estrutura de Diretórios

```
<projeto>/cc-skills/<nome>/
├── CLAUDE.md                              # Documentação do projeto (obrigatório)
├── .claude/
│   ├── settings.json                      # Configurações do Claude Code (obrigatório)
│   ├── commands/
│   │   └── <prefix>/
│   │       └── agents/                    # Slash commands — personas interativas (obrigatório)
│   │           ├── <agent-1>.md
│   │           ├── <agent-2>.md
│   │           └── ...
│   ├── skills/
│   │   └── <nome>/
│   │       ├── SKILL.md                   # Orchestrator — pipeline de fases (obrigatório)
│   │       ├── agents/                    # Skill agents — spawnáveis via Task tool (obrigatório)
│   │       │   ├── <agent-1>.md
│   │       │   ├── <agent-2>.md
│   │       │   └── ...
│   │       └── references/                # Docs de referência do domínio (opcional)
│   │           └── *.md
│   └── rules/                             # Regras globais do projeto (obrigatório, mínimo 1)
│       ├── code-standards.md              # Padrões de código (obrigatório)
│       └── *.md                           # Rules adicionais específicas do domínio
└── (output gerado pelo pipeline no projeto do usuário)
```

### Campos do Diagrama

| Diretório/Arquivo | Obrigatório | Origem AIOS | Descrição |
|---|---|---|---|
| `CLAUDE.md` | Sim | `squad.yaml` + `README.md` + análise | Documentação completa do projeto CC |
| `.claude/settings.json` | Sim | — | Configurações mínimas do Claude Code |
| `.claude/commands/<prefix>/agents/` | Sim | `agents/*.md` | Personas interativas (slash commands) |
| `.claude/skills/<nome>/SKILL.md` | Sim | `workflows/*.yaml` | Orquestrador do pipeline |
| `.claude/skills/<nome>/agents/` | Sim | `agents/*.md` | Agents técnicos para spawn via Task |
| `.claude/skills/<nome>/references/` | Não | Análise de domínio | Conhecimento especializado |
| `.claude/rules/` | Sim | `config/coding-standards.md` | Regras acionáveis do projeto |

---

## Formato: Skill Agent (`.claude/skills/<nome>/agents/*.md`)

Arquivo Markdown com **YAML frontmatter** seguido de **corpo Markdown**. O agent é spawnado via `Task(subagent_type="general-purpose", model="<modelo>")` pelo orchestrator.

### YAML Frontmatter (obrigatório)

```yaml
---
name: Nome do Agente
description: Descrição concisa do que o agente faz (1-2 linhas)
tools: Read, Write, Glob, Grep       # Tools que o agente precisa
model: opus                           # opus | sonnet
maxTurns: 25                          # Máximo de turnos
---
```

| Campo | Obrigatório | Tipo | Descrição |
|---|---|---|---|
| `name` | Sim | string | Nome legível do agente |
| `description` | Sim | string | O que faz em 1-2 linhas |
| `tools` | Sim | string (CSV) | Tools separados por vírgula |
| `model` | Sim | enum | `opus` (criação/construção) ou `sonnet` (pesquisa/auditoria) |
| `maxTurns` | Sim | number | Limite de turnos (15-40) |

### Regras de Inferência para Tools

| Tipo de Trabalho | Tools Recomendados |
|---|---|
| Pesquisa na web | `Read, Write, Glob, Grep, WebSearch, WebFetch` |
| Edição de código | `Read, Write, Edit, Bash, Glob, Grep` |
| Validação/auditoria | `Read, Glob, Grep` (mínimo para read-only) |
| Criação de artefatos | `Read, Write, Glob, Grep` |
| Orquestração | `Read, Write, Bash, Task, Glob, Grep` |

### Regras de Inferência para Model

| Tipo de Trabalho | Model |
|---|---|
| Pesquisa, análise, auditoria | `sonnet` |
| Criação, construção, síntese | `opus` |

### Corpo Markdown

```markdown
# Nome do Agente

Você é um **[Papel]** especialista em [domínio]. [Descrição do papel em 1-2 frases].

## Expertise

- [Área de expertise 1]
- [Área de expertise 2]
- [Área de expertise 3]

## Responsabilidades

1. **[Responsabilidade 1]**: [Descrição detalhada]
2. **[Responsabilidade 2]**: [Descrição detalhada]

## Regras

1. [Regra acionável 1]
2. [Regra acionável 2]

## Formato de Output

[Descrição do output esperado com exemplo de estrutura]
```

**Seções obrigatórias:** Título (H1), descrição do papel, Expertise, Responsabilidades, Regras, Formato de Output.

---

## Formato: Command/Persona (`.claude/commands/<prefix>/agents/*.md`)

Markdown puro (sem YAML frontmatter). Define a **persona interativa** do agente — é o que o usuário experimenta ao invocar `/<prefix>:agents:<agent-id>`.

### Estrutura

```markdown
# <Nome da Persona> — <Título>

Você é o **<Nome da Persona>**, [descrição do papel].

## Identidade
- **Nome:** <Nome curto e memorável>
- **Arquétipo:** <Builder | Guardian | Balancer | Flow_Master>
- **Tom:** <Descrição do tom>
- **Especialidade:** <Lista de especialidades>

## Saudação
Ao ser ativado, apresente-se:
> <icon> **<Nome>** | <Título>
> <Descrição de expertise em 1 linha>
> <Call-to-action>

## Capacidades

### <Categoria 1>
- [Capacidade específica]
- [Capacidade específica]

### <Categoria 2>
- [Capacidade específica]

## Comandos

- `*<command-1> [args]` — <Descrição>
- `*<command-2> [args]` — <Descrição>

## Colaboração

- **Entrega para:** <Agente destino> — <O que entrega>
- **Recebe de:** <Agente origem> — <O que recebe>

## Regras

- [Regra 1]
- [Regra 2]
```

### Regras de Geração de Nomes de Persona

Gerar nomes **curtos e memoráveis** baseados no papel:

| Papel | Exemplos de Nome |
|---|---|
| Pesquisador / Analista | Aurora, Scout, Lens |
| Construtor / Engenheiro | Forge, Atlas, Mason |
| Designer / Criativo | Prism, Canvas, Bloom |
| Guardião / Validador | Shield, Sentinel, Vigil |
| Otimizador / Performance | Catalyst, Turbo, Flux |
| Estrategista / Líder | Compass, Sage, Oracle |
| Escritor / Copywriter | Quill, Echo, Voice |

---

## Formato: SKILL.md Orchestrator (`.claude/skills/<nome>/SKILL.md`)

Arquivo com **YAML frontmatter** seguido de **Markdown** que define o pipeline de fases.

### YAML Frontmatter

```yaml
---
name: <nome-da-skill>
description: <Descrição completa do que a skill faz — 1-3 linhas>
user-invocable: true
argument-hint: "[contexto]"
allowed-tools: Read, Write, Edit, Bash, Task, Glob, Grep, WebSearch, WebFetch
---
```

| Campo | Obrigatório | Descrição |
|---|---|---|
| `name` | Sim | Identificador kebab-case |
| `description` | Sim | O que faz (usado pelo Claude para matching) |
| `user-invocable` | Sim | `true` para skills chamadas pelo usuário |
| `argument-hint` | Sim | Dica de argumentos |
| `allowed-tools` | Sim | Tools que o orchestrator pode usar |

### Corpo — Padrão de Fases

```markdown
# /<nome> — <Título>

Você é o **orquestrador** de um pipeline de N fases para [objetivo].

## Agents Disponíveis

| Agent | Arquivo | Modelo | Fase |
|-------|---------|--------|------|
| <Nome> | `agents/<id>.md` | <modelo> | <N> |

**Padrão de spawn:**
Task(subagent_type = "general-purpose",
     model = "<modelo>",
     prompt = "Leia suas instruções em: .claude/skills/<nome>/agents/<id>.md\n\n[contexto]")

## FASE 0 — Coleta de Contexto
[Parsing de argumentos, coleta de input]

## FASE N — <Nome da Fase>
**Subagente**: `<agent-id>`
1. Spawn Task com instruções
2. Verificar output
3. Resumir ao usuário

## Finalização
[Resumo final com estrutura criada]

## Regras Gerais
[Regras do orchestrator]
```

**Regra crítica**: Cada fase DEVE usar `Task(subagent_type="general-purpose")` com `model` explícito. O prompt DEVE instruir o agente a ler suas instruções do arquivo em `.claude/skills/<nome>/agents/`.

---

## Formato: Rules (`.claude/rules/*.md`)

Bullets acionáveis e concisos. Carregados automaticamente pelo Claude Code em toda interação.

### Regras de Formatação

- Um arquivo por tema (ex: `code-standards.md`, `mobile-first.md`)
- H1 com título descritivo
- Bullets concisos — uma regra por linha
- Sem parágrafos longos — regras devem ser skimmable
- Usar exemplos inline quando necessário (com backticks)

### Exemplo

```markdown
# Padrões de Código

- TypeScript strict mode obrigatório
- Named exports (não default)
- Componentes: PascalCase (Button.tsx, HeroSection.tsx)
- Variáveis CSS: kebab-case (--color-primary)
- Conteúdo textual em PT-BR
- Variáveis e código em inglês
- UTF-8 com acentuação correta
```

### Arquivo Obrigatório: `code-standards.md`

Todo CC Skill DEVE ter `code-standards.md` derivado de `config/coding-standards.md` do squad AIOS. Condensar em bullets acionáveis.

### Rules Adicionais por Domínio

Gerar rules adicionais com base no domínio detectado na análise:

| Domínio | Rules Sugeridas |
|---|---|
| Frontend/UI | `atomic-design-hierarchy.md`, `design-tokens.md`, `tailwind-gotchas.md` |
| Landing Page | `hormozi-copy.md`, `motion-patterns.md`, `mobile-first.md` |
| API/Backend | `api-patterns.md`, `error-handling.md`, `security.md` |
| Data/ETL | `data-quality.md`, `pipeline-patterns.md` |
| DevOps | `infrastructure.md`, `monitoring.md` |

---

## Formato: References (`.claude/skills/<nome>/references/*.md`)

Documentos de conhecimento especializado do domínio. Lidos pelos skill agents durante execução.

### Regras

- Conteúdo técnico e aprofundado
- Foco em "como fazer" — padrões, exemplos, anti-patterns
- Derivados da expertise dos agents AIOS (sintetizar conhecimento dos persona_profiles)
- Nomeados por tópico em kebab-case (ex: `token-system.md`, `hormozi-frameworks.md`)

---

## Formato: CLAUDE.md

Documentação principal do projeto CC Skill. Carregada automaticamente pelo Claude Code.

### Seções Obrigatórias

```markdown
# <Nome do Projeto>

<Descrição concisa: o que faz, como funciona>

## O que este projeto faz

<Descrição expandida com lista de capacidades>

## Tech Stack

| Tecnologia | Versão | Papel |
|---|---|---|
| ... | ... | ... |

## Pipeline — N Fases

| Fase | Agente | Papel | Modelo |
|---|---|---|---|
| 1 | ... | ... | ... |

## Como Invocar

### Skill Orchestrator (pipeline completo)
/<nome> [argumentos]

### Agents individuais (slash commands)
/<prefix>:agents:<agent-id> — <Persona> (<Papel>)

## Lista de Agents

| Agent | Persona | Arquétipo | Papel |
|---|---|---|---|
| ... | ... | ... | ... |

## Naming Conventions

| Contexto | Convenção | Exemplo |
|---|---|---|
| ... | ... | ... |

## Quality Gates

- [Gate 1]
- [Gate 2]

## Idioma e Encoding

- Conteúdo textual: PT-BR com acentuação correta
- Variáveis e código: Inglês (padrão internacional)
- Encoding: UTF-8

## Estrutura do Projeto

[Diagrama de diretórios]
```

---

## Formato: settings.json

Configurações mínimas do Claude Code.

```json
{
  "language": "portuguese"
}
```

---

## Tabela de Mapeamento Completa: AIOS → CC Skill

| # | AIOS Source | CC Skill Target | Lógica de Transformação |
|---|---|---|---|
| 1 | `agents/*.md` (persona_profile + expertise) | `.claude/skills/<nome>/agents/*.md` | Extrair YAML frontmatter CC (name, description, tools, model, maxTurns). Body: Expertise, Responsabilidades, Regras, Formato de Output |
| 2 | `agents/*.md` (identity + commands) | `.claude/commands/<prefix>/agents/*.md` | Gerar persona interativa: Identidade, Saudação, Capacidades, Comandos, Colaboração |
| 3 | `workflows/*.yaml` (agent_sequence + transitions) | `.claude/skills/<nome>/SKILL.md` | Converter agent_sequence em fases ordenadas. Cada fase: spawn Task com subagent_type="general-purpose" |
| 4 | `config/coding-standards.md` | `.claude/rules/code-standards.md` | Condensar em bullets acionáveis |
| 5 | `config/tech-stack.md` + domínio detectado | `.claude/rules/*.md` | Gerar rules específicas por domínio |
| 6 | `squad.yaml` + `README.md` + análise | `CLAUDE.md` | Compor documentação: O que faz, Tech Stack, Pipeline, Agents, Naming, Quality Gates |
| 7 | — | `.claude/settings.json` | Gerar com `{ "language": "portuguese" }` |
| 8 | Expertise dos agents (persona_profile) | `.claude/skills/<nome>/references/*.md` | Sintetizar conhecimento em docs de referência do domínio |

---

## Convenções Gerais

- **Naming**: kebab-case para arquivos, IDs e nomes de skills
- **Idioma**: Conteúdo em PT-BR, variáveis em inglês
- **Encoding**: UTF-8 com acentuação correta
- **Prefix**: Usar o `slashPrefix` do `squad.yaml` original
- **Nome da skill**: Usar o `name` do `squad.yaml` original
