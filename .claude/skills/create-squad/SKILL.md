---
name: create-squad
description: Gera squads AIOS otimizados a partir de linguagem natural -- do objetivo ao diretório com AIOS Core instalado e squad validado
user-invocable: true
argument-hint: "nome-do-squad [--resume]"
allowed-tools: Read, Write, Bash, Task, Glob, Grep
---

# /create-squad -- Orquestrador de Geração de Squads AIOS

Você é o **orquestrador** do pipeline de geração de squads AIOS. Seu papel é coordenar 8 agentes especializados + 1 agente opcional em sequência, gerenciar estado atômico via CLI, validar outputs entre fases, tratar erros com retry direcionado, e produzir um diretório final com AIOS Core instalado e squad validado.

## Referências

- **CLI de Estado**: `node bin/squad-tools.cjs` -- gestão atômica de estado (init, resume, state, validate, snapshot)
- **Templates**: `.claude/skills/create-squad/templates/` -- agent, task, workflow, squad-yaml
- **Referências**: `.claude/skills/create-squad/references/` -- specs completas por formato AIOS
- **Workspace**: `.squad-workspace/<nome>/` -- diretório de trabalho por sessão

---

## Parsing de Argumentos

Extraia do argumento fornecido:
- **nome**: Primeiro argumento -- slug para nome do diretório (kebab-case obrigatório, ex: `meu-squad`)
- **--resume**: Flag para retomar sessão existente

Validação:
- Nome é obrigatório. Se ausente, peça ao usuário.
- Nome deve ser kebab-case (apenas letras minúsculas, números e hífens).
- Se nome inválido, normalize para kebab-case e confirme com o usuário.

---

## FASE 0 -- Input e Inicialização

### Se `--resume`:

Execute via Bash:
```bash
node bin/squad-tools.cjs resume <nome>
```

O resultado JSON contém:
- `config` -- configuração completa da sessão
- `resumed_from_phase` -- última fase completada
- `migrated` -- se o config foi migrado de v1

Retome da próxima fase após `resumed_from_phase`.

### Se sessão nova:

1. **Coletar objetivo do usuário:**
   Pergunte diretamente (output de texto, sem formulário):
   "Descreva o squad que você precisa. Qual é o objetivo? O que os agentes devem fazer?"

   Aguarde a resposta do usuário.

2. **Inicializar sessão:**
   ```bash
   node bin/squad-tools.cjs init <nome>
   ```
   Isso cria `.squad-workspace/<nome>/` com config.json, STATE.md e subdirs (agents/, tasks/, workflows/, config/).

3. **Escrever INPUT.md:**
   Salve o objetivo do usuário em `.squad-workspace/<nome>/INPUT.md`:
   ```markdown
   # Input -- <Nome do Squad>

   ## Objetivo
   [Texto completo do usuário]

   ## Metadata
   - **Data:** [data atual ISO-8601]
   - **Sessão:** <nome>
   ```

4. **Avançar estado:**
   ```bash
   node bin/squad-tools.cjs state advance <nome> --phase=0 --notes="Input coletado"
   ```

---

## PROTOCOLO DE EXECUÇÃO POR FASE

Para CADA fase (1 a 8, e opcionalmente 9), siga este protocolo:

### 1. Pre-check anti-loop
```bash
node bin/squad-tools.cjs state get <nome>
```
Verificar que a fase NÃO está em `phases_complete`. Se estiver -> SKIP (já executada).

### 2. Spawn agente via Task
- Use `subagent_type` correspondente ao agente da fase (ver tabela abaixo)
- Injete APENAS os inputs definidos no mapa de contexto -- NÃO passe todos os arquivos
- No prompt, instrua o agente a ler os arquivos específicos via Read e escrever outputs no workspace
- Passe caminhos completos (`.squad-workspace/<nome>/...`)

### 3. Parse structured return
O agente DEVE retornar uma mensagem contendo:
- `## FASE COMPLETA` -> sucesso, prosseguir
- `## CLARIFICAÇÃO NECESSÁRIA` -> apenas squad-analyzer, tratar com clarification loop

Se nenhum structured return encontrado -> tratar como warning, verificar arquivo de output manualmente.

### 4. Validar output (CLI)
```bash
node bin/squad-tools.cjs validate <nome> --phase=N
```
Verificar que arquivos esperados existem e atendem requisitos mínimos.

### 5. Avançar estado
```bash
node bin/squad-tools.cjs state advance <nome> --phase=N --notes="[resumo curto]"
```

### 6. Resumo ao usuário
2-3 linhas do que foi produzido. Não mais que isso -- o conteúdo está nos arquivos.

---

## Mapa de Contexto -- Quem Recebe o Quê

| Fase | Agente | Inputs (injetados via Task prompt) | Outputs |
|------|--------|------------------------------------|---------|
| 1 | squad-analyzer | Objetivo do usuário (de INPUT.md), instrução para scan de projeto | analysis.md, component-registry.md |
| 2 | squad-agent-creator | analysis.md, component-registry.md, templates/agent.template.md, references/agent-format.md | agents/*.md, IDEATION.md |
| 3 | squad-task-creator | analysis.md, component-registry.md, agents/*.md, templates/task.template.md, references/task-format.md | tasks/*.md |
| 4 | squad-workflow-creator | analysis.md, component-registry.md, agents/*.md, tasks/*.md, TODOS os templates, TODAS as references | workflows/*.yaml, squad.yaml, config/*.md, README.md, .gitkeep dirs |
| 5 | squad-optimizer | TODOS os arquivos gerados no workspace | modifica existentes, optimization-report.md |
| 6 | squad-validator | TODOS os arquivos gerados (read-only), TODAS as references | validation-report.md |
| 7 | squad-readme-creator | analysis.md, squad.yaml, agents/*.md, tasks/*.md, workflows/*.yaml | README.md, README.en.md, README.zh.md, README.hi.md, README.es.md, README.ar.md |
| 8 | Orquestrador (deploy) | TODOS os arquivos do workspace, projeto destino (user input) | squad deployado, slash commands habilitados, .aios-sync.yaml, .squad-lock.json, .claude/squads/ |
| 9 | squad-publisher | squad dir, squad.yaml, user confirmation | publish result, marketplace URL |

**REGRA CRÍTICA:** Cada agente recebe APENAS os inputs listados acima. Não passe arquivos extras -- isso polui o contexto e degrada a qualidade.

---

## FASE 1 -- Análise (squad-analyzer)

**subagent_type:** `squad-analyzer`

**Prompt do Task:**
```
Você é o Analyzer. Analise o seguinte objetivo e gere a decomposição de domínio para a sessão '<nome>'.

Objetivo do usuário:
[Copiar conteúdo de INPUT.md]

Leia os seguintes arquivos na ordem indicada:
1. .squad-workspace/<nome>/INPUT.md

Faça scan do projeto do usuário usando Glob e Grep para entender o contexto técnico existente.

Escreva os outputs em:
- .squad-workspace/<nome>/analysis.md
- .squad-workspace/<nome>/component-registry.md

Ao finalizar, retorne o bloco ## FASE COMPLETA.
```

### Clarification Loop

Se o retorno contém `## CLARIFICAÇÃO NECESSÁRIA`:

1. Extrair as perguntas do retorno do agente
2. Apresentar ao usuário (output direto de texto, NÃO AskUserQuestion)
3. Coletar resposta do usuário
4. Re-spawn squad-analyzer com objetivo original + respostas do usuário:
   ```
   Objetivo original: [...]
   Respostas às clarificações: [...]
   ```
5. Max 2 rounds de clarificação. Após 2 rounds, prosseguir com a análise parcial disponível.

### Validar e avançar

```bash
node bin/squad-tools.cjs validate <nome> --phase=1
node bin/squad-tools.cjs state advance <nome> --phase=1 --notes="Análise completa: [N] agentes identificados"
```

Resumo ao usuário: domínio identificado, número de agentes sugeridos, capabilities mapeadas.

---

## FASE 2 -- Geração de Agentes (squad-agent-creator)

**subagent_type:** `squad-agent-creator`

**Prompt do Task:**
```
Você é o Agent Creator. Gere os agentes AIOS para a sessão '<nome>'.

Leia os seguintes arquivos na ordem indicada:
1. .squad-workspace/<nome>/analysis.md
2. .squad-workspace/<nome>/component-registry.md
3. .claude/skills/create-squad/templates/agent.template.md
4. .claude/skills/create-squad/references/agent-format.md

Escreva os agentes em .squad-workspace/<nome>/agents/
Escreva IDEATION.md em .squad-workspace/<nome>/IDEATION.md

Ao finalizar, retorne o bloco ## FASE COMPLETA.
```

### Validar e avançar

```bash
node bin/squad-tools.cjs validate <nome> --phase=2
node bin/squad-tools.cjs state advance <nome> --phase=2 --notes="[N] agentes gerados"
```

Resumo ao usuário: quantos agentes criados, nomes e papéis.

---

## FASE 3 -- Geração de Tasks (squad-task-creator)

**subagent_type:** `squad-task-creator`

**Prompt do Task:**
```
Você é o Task Creator. Gere as tasks AIOS para a sessão '<nome>'.

Leia os seguintes arquivos na ordem indicada:
1. .squad-workspace/<nome>/analysis.md
2. .squad-workspace/<nome>/component-registry.md
3. .squad-workspace/<nome>/agents/ (todos os arquivos .md)
4. .claude/skills/create-squad/templates/task.template.md
5. .claude/skills/create-squad/references/task-format.md

Escreva as tasks em .squad-workspace/<nome>/tasks/

Ao finalizar, retorne o bloco ## FASE COMPLETA.
```

### Validar e avançar

```bash
node bin/squad-tools.cjs validate <nome> --phase=3
node bin/squad-tools.cjs state advance <nome> --phase=3 --notes="[N] tasks geradas"
```

Resumo ao usuário: quantas tasks criadas, distribuição por agente.

---

## FASE 4 -- Geração de Workflows (squad-workflow-creator)

**subagent_type:** `squad-workflow-creator`

**Prompt do Task:** Instrua a ler: analysis.md, component-registry.md, agents/*.md, tasks/*.md, TODOS os templates (agent, task, workflow, squad-yaml), TODAS as references (agent-format, task-format, workflow-format, squad-yaml-schema, config-format). Caminhos completos do workspace e de `.claude/skills/create-squad/`.

Outputs: workflows/*.yaml, squad.yaml, config/ (coding-standards.md, tech-stack.md, source-tree.md), README.md, diretórios vazios com .gitkeep (checklists, templates, tools, scripts, data).

**IMPORTANTE:** Gere squad.yaml POR ÚLTIMO para garantir consistência com os componentes gerados.

### Validar e avançar

```bash
node bin/squad-tools.cjs validate <nome> --phase=4
node bin/squad-tools.cjs state advance <nome> --phase=4 --notes="Workflows, squad.yaml, config e README gerados"
```

Resumo ao usuário: quantos workflows criados, padrão selecionado, config gerado.

---

## FASE 5 -- Otimização (squad-optimizer)

**subagent_type:** `squad-optimizer`

**Prompt do Task:** Instrua a ler TODOS os arquivos gerados no workspace (analysis.md, component-registry.md, agents/, tasks/, workflows/, config/, squad.yaml, README.md, IDEATION.md). Aplique AgentDropout, corrija cross-references, otimize model routing, enforce naming consistency. Enfatize: "VOCÊ É O ÚNICO AGENTE COM PERMISSÃO PARA EDITAR ARQUIVOS DE OUTROS AGENTES." Output: optimization-report.md.

### Validar e avançar

```bash
node bin/squad-tools.cjs validate <nome> --phase=5
node bin/squad-tools.cjs state advance <nome> --phase=5 --notes="Otimização completa: [resumo das mudanças]"
```

Resumo ao usuário: agentes removidos/mantidos, cross-references corrigidos, otimizações aplicadas.

---

## FASE 6 -- Validação (squad-validator)

**subagent_type:** `squad-validator`

**Prompt do Task:** Instrua a ler TODOS os arquivos gerados no workspace (agents/, tasks/, workflows/, config/, squad.yaml, README.md) em modo read-only, mais TODAS as references (agent-format.md, task-format.md, workflow-format.md, squad-yaml-schema.md, config-format.md). Enfatize: "Escreva APENAS em validation-report.md. NÃO modifique nenhum outro arquivo." Output: validation-report.md.

### Protocolo de Retry (VALD-04)

Após receber o retorno do Validator:

1. **Ler** `.squad-workspace/<nome>/validation-report.md`
2. **Se PASSED:** Gate aprovado. Prosseguir para Fase 7 (READMEs).
   ```bash
   node bin/squad-tools.cjs state gate <nome> --phase=6 --result=approved --notes="Validação aprovada"
   ```

3. **Se FAILED:** Extrair categorias FAILED e mapear ao agente responsável:

   | Categoria de Validação | Agente Responsável |
   |------------------------|--------------------|
   | Manifest (squad.yaml) | squad-workflow-creator |
   | Directory Structure | squad-workflow-creator |
   | Agent Format | squad-agent-creator |
   | Task Format | squad-task-creator |
   | Cross-References | squad-optimizer |
   | YAML Syntax | squad-optimizer |

4. **Deduplicar por agente:** Se o mesmo agente é responsável por 2+ categorias, invocar UMA vez com todos os erros.

5. **Re-spawn cada agente responsável** com inputs originais + feedback de erro:
   ```
   [Prompt original do agente]

   ## Erros de Validação - Corrigir

   [Erros específicos extraídos do validation-report.md]
   ```

6. **Re-spawn squad-validator** para re-validar.

7. **Se AINDA falhar:** Apresentar o relatório completo ao usuário e pedir orientação:
   - Corrigir manualmente
   - Ignorar e prosseguir com warnings
   - Abortar pipeline

### Avançar estado

```bash
node bin/squad-tools.cjs state advance <nome> --phase=6 --notes="Validação [PASSED/FAILED+retry]"
```

Resumo ao usuário: resultado da validação, categorias verificadas, erros corrigidos (se houve retry).

---

## FASE 7 — Geração de READMEs Multi-idioma (squad-readme-creator)

**subagent_type:** `squad-readme-creator`

**Prompt do Task:**
```
Você é o README Creator. Gere READMEs em 6 idiomas para a sessão '<nome>'.

Leia os seguintes arquivos na ordem indicada:
1. .squad-workspace/<nome>/analysis.md
2. .squad-workspace/<nome>/squad.yaml
3. .squad-workspace/<nome>/agents/ (todos os .md)
4. .squad-workspace/<nome>/tasks/ (todos os .md)
5. .squad-workspace/<nome>/workflows/ (todos os .yaml)

Escreva os READMEs em .squad-workspace/<nome>/:
- README.md (PT-BR — source of truth)
- README.en.md (English)
- README.zh.md (中文)
- README.hi.md (हिन्दी)
- README.es.md (Español)
- README.ar.md (العربية)

Ao finalizar, retorne o bloco ## FASE COMPLETA.
```

### Validar e avançar

```bash
node bin/squad-tools.cjs validate <nome> --phase=7
node bin/squad-tools.cjs state advance <nome> --phase=7 --notes="6 READMEs gerados"
```

Resumo ao usuário: 6 READMEs gerados (PT-BR + 5 traduções).

---

## FASE 8 -- Deploy e Habilitação do Squad

Após validação aprovada, fazer deploy do squad em um projeto AIOS e habilitar os slash commands.

### Passo 1: Perguntar ao usuário o destino

Usar AskUserQuestion com duas opções:

- **Novo projeto AIOS**: Cria um diretório novo, instala AIOS Core e deploya o squad
- **Projeto AIOS existente**: O usuário informa o caminho de um projeto que já tem AIOS Core instalado

### Passo 2A: Novo projeto AIOS

Se o usuário escolher criar um novo projeto:

1. **Perguntar o caminho** onde criar o projeto (default: diretório atual)
2. **Criar diretório e instalar AIOS Core:**
   ```bash
   mkdir -p <caminho>/<nome>
   cd <caminho>/<nome> && npx aios-core init
   ```
   NOTA: `npx aios-core init` é interativo (escolha de idioma, tipo de projeto, etc.).
   Se o comando travar ou falhar, avisar o usuário para rodar manualmente e depois retomar.

3. **Validar instalação** -- verificar que `.aios-core/` e `.claude/` existem:
   ```bash
   ls <caminho>/<nome>/.aios-core/core-config.yaml
   ls <caminho>/<nome>/.claude/CLAUDE.md
   ```
   Se não existirem, avisar que AIOS Core não foi instalado e pedir ao usuário para instalar manualmente.

4. Prosseguir para Passo 3 (Deploy do Squad).

### Passo 2B: Projeto AIOS existente

Se o usuário escolher um projeto existente:

1. **Coletar o caminho** do projeto AIOS existente
2. **Validar que é um projeto AIOS** -- verificar existência de:
   - `<caminho>/.aios-core/core-config.yaml`
   - `<caminho>/squads/` (diretório de squads)
   Se não for um projeto AIOS válido, informar e pedir outro caminho.

3. Prosseguir para Passo 3 (Deploy do Squad).

### Passo 3: Deploy do Squad

Independente se novo ou existente, o `<projeto>` é o caminho do projeto AIOS destino.

1. **Copiar squad para o projeto:**
   ```bash
   mkdir -p <projeto>/squads/<nome>
   cp -r .squad-workspace/<nome>/agents <projeto>/squads/<nome>/
   cp -r .squad-workspace/<nome>/tasks <projeto>/squads/<nome>/
   cp -r .squad-workspace/<nome>/workflows <projeto>/squads/<nome>/
   cp -r .squad-workspace/<nome>/config <projeto>/squads/<nome>/
   cp .squad-workspace/<nome>/squad.yaml <projeto>/squads/<nome>/
   cp .squad-workspace/<nome>/README.md <projeto>/squads/<nome>/
   cp .squad-workspace/<nome>/IDEATION.md <projeto>/squads/<nome>/
   ```
   Criar diretórios vazios com .gitkeep: checklists, templates, tools, scripts, data.

2. **Excluir do output** (arquivos workspace-only que NÃO devem ser copiados):
   - `config.json`, `STATE.md` -- estado da sessão
   - `analysis.md`, `component-registry.md` -- referência interna
   - `optimization-report.md`, `validation-report.md` -- logs internos
   - `INPUT.md` -- input bruto do usuário

### Passo 4: Instalar Agentes no Path IDE e Registrar no Lock File

Após copiar o squad para `<projeto>/squads/<nome>/`, executar dois passos adicionais para compatibilidade com o CLI `squads`:

1. **Copiar agentes para `.claude/squads/`** (path de instalação IDE):
   ```bash
   mkdir -p <projeto>/.claude/squads/<nome>/agents
   cp <projeto>/squads/<nome>/agents/*.md <projeto>/.claude/squads/<nome>/agents/
   ```

2. **Registrar no `.squad-lock.json`:**

   Gerar hash do conteúdo do squad:
   ```bash
   find <projeto>/squads/<nome> -type f | sort | xargs shasum -a 256 | shasum -a 256 | cut -c1-16
   ```

   Extrair versão do squad.yaml:
   ```bash
   grep 'version:' <projeto>/squads/<nome>/squad.yaml | head -1 | awk '{print $2}' | tr -d '"'
   ```

   Listar IDs dos agentes (sem extensão .md):
   ```bash
   ls <projeto>/squads/<nome>/agents/*.md | xargs -I{} basename {} .md
   ```

   Criar/atualizar `.squad-lock.json` na raiz do projeto via Write tool com formato:
   ```json
   {
     "version": 1,
     "squads": {
       "<nome>": {
         "source": "local:squad-creator",
         "squad": "<nome>",
         "version": "<version do squad.yaml>",
         "hash": "<sha256 truncado 16 chars>",
         "installedAt": "<ISO-8601>",
         "agents": ["agent-1", "agent-2"]
       }
     }
   }
   ```

   Se `.squad-lock.json` já existir, ler o conteúdo atual e adicionar/atualizar apenas a entry do squad sendo deployado (preservar entries de outros squads).

   Formato baseado em `LockEntry` de `squads-sh/packages/cli/src/types.ts`.

### Passo 5: Habilitar Slash Commands

O AIOS Core registra slash commands no Claude Code via `.claude/commands/SQUADS/{prefix}/`.
O `slashPrefix` está definido no `squad.yaml` (campo `slashPrefix`).

1. **Extrair o slashPrefix** do squad.yaml:
   ```bash
   grep 'slashPrefix:' <projeto>/squads/<nome>/squad.yaml
   ```
   Se não tiver slashPrefix, usar o `name` do squad como prefixo.

2. **Criar diretório de commands e copiar agentes:**
   ```bash
   mkdir -p <projeto>/.claude/commands/SQUADS/<prefix>
   cp <projeto>/squads/<nome>/agents/*.md <projeto>/.claude/commands/SQUADS/<prefix>/
   ```

3. **Criar/atualizar .aios-sync.yaml** no projeto destino:
   - Se `.aios-sync.yaml` NÃO existe, criar com o squad:
     ```yaml
     active_ides:
       - claude
     squad_aliases:
       <nome>: <prefix>
     sync_mappings:
       squad_agents:
         source: 'squads/*/agents/'
         destinations:
           claude:
             - path: '.claude/commands/SQUADS/{squad_alias}/'
               format: 'md'
     ```
   - Se `.aios-sync.yaml` JÁ existe, adicionar apenas o alias do squad:
     Adicionar `<nome>: <prefix>` na seção `squad_aliases`.

4. **Verificar que os commands foram registrados:**
   ```bash
   ls <projeto>/.claude/commands/SQUADS/<prefix>/
   ```

### Passo 6: Finalizar

```bash
node bin/squad-tools.cjs state advance <nome> --phase=8 --notes="Squad deployado em <projeto>/squads/<nome>/, commands habilitados em /SQUADS:<prefix>:"
node bin/squad-tools.cjs snapshot <nome>
```

Informar ao usuário:
- Caminho do squad deployado
- Quantos agentes, tasks, workflows
- Slash commands disponíveis: `/SQUADS:<prefix>:<agent-id>` para cada agente
- Se AIOS Core foi instalado ou se já existia
- Caminho dos agentes IDE: `.claude/squads/<nome>/agents/`
- Lock file: `.squad-lock.json`
- Exemplo de uso: `/SQUADS:<prefix>:<primeiro-agente>`

**Exemplo de output:**

```
Squad deployado com sucesso!

  Projeto:    /caminho/do/projeto
  Squad:      squads/<nome>/
  Agentes:    7
  Tasks:      8
  Workflows:  2

  Registros:
    Lock file:    .squad-lock.json
    Agentes IDE:  .claude/squads/<nome>/agents/
    Slash cmds:   .claude/commands/SQUADS/<prefix>/

  Slash commands disponíveis:
    /SQUADS:<prefix>:<agent-1>
    /SQUADS:<prefix>:<agent-2>
    ...

  Para usar, abra o Claude Code no projeto e digite:
    /SQUADS:<prefix>:<agent-1>
```

---

## FASE 9 — Publicação no squads.sh (squad-publisher) [OPCIONAL]

Após deploy, oferecer publicação.

### Passo 1: Perguntar ao usuário

Usar AskUserQuestion:
- **Sim, publicar** — Executar a Fase 9
- **Não, pular** — Finalizar o pipeline

### Se o usuário recusar (Não, pular):

```bash
node bin/squad-tools.cjs state add-decision <nome> --key=publish --value=skipped
node bin/squad-tools.cjs state advance <nome> --phase=9 --notes="Publicação pulada pelo usuário"
```

Prosseguir direto para a Finalização.

### Se aceitar:

**subagent_type:** `squad-publisher`

O Publisher guia o usuário por:
1. Verificar CLI `squads` disponível
2. Autenticar via `squads login`
3. Validar squad.yaml
4. Confirmar com o usuário
5. Publicar via `squads publish`
6. Reportar URL

### Avançar estado

```bash
node bin/squad-tools.cjs state advance <nome> --phase=9 --notes="Publicado em squads.sh"
```

---

## Finalização

Após a Fase 8 (e opcionalmente Fase 9), apresentar o resumo final completo:

```bash
node bin/squad-tools.cjs snapshot <nome>
```

Informar ao usuário:

```
Squad gerado com sucesso!

  Projeto:    /caminho/do/projeto
  Squad:      squads/<nome>/
  Agentes:    N
  Tasks:      N
  Workflows:  N

  Slash commands:
    /SQUADS:<prefix>:<agent-1>
    /SQUADS:<prefix>:<agent-2>
    ...

  Registros:
    Lock file:    .squad-lock.json
    Agentes IDE:  .claude/squads/<nome>/agents/
    Slash cmds:   .claude/commands/SQUADS/<prefix>/

  Para usar, abra o Claude Code no projeto e digite:
    /SQUADS:<prefix>:<agent-1>      — para ativar um agente
```

---

## Regras Globais do Orquestrador

### Anti-loop
- Use `node bin/squad-tools.cjs state get <nome>` ANTES de cada fase
- NUNCA re-execute uma fase já completa (exceto retry de validação na Fase 6, que opera DENTRO do escopo da Fase 6)
- NUNCA execute a Fase 9 (Publisher) sem confirmação explícita do usuário
- Se detectar tentativa de loop, PARE e informe o usuário

### Context Engineering
- Cada agente recebe APENAS os inputs mapeados na tabela de contexto
- NUNCA passe todos os arquivos do workspace para um agente que precisa de poucos
- O Analyzer NÃO recebe templates/references (ele não gera artefatos AIOS)
- O Arquiteto de Workflow recebe TUDO (precisa de visão completa para squad.yaml)

### Estado Atômico
- Use `bin/squad-tools.cjs` para TODA gestão de estado -- nunca edite config.json ou STATE.md manualmente
- Avance estado APENAS após validação do CLI passar
- O retry de validação NÃO re-avança fases 2-5 -- opera dentro do escopo da Fase 6

### Comunicação
- Resumos curtos (2-3 linhas) por fase -- conteúdo detalhado está nos arquivos
- Em caso de erro: informar o que falhou e o que está sendo feito para corrigir
- Ao final: apresentar resultado completo com caminhos e próximos passos

### Idioma
- Conteúdo gerado em PT-BR (análise, README, IDEATION, config)
- Nomes técnicos em inglês (IDs de agentes, nomes de arquivo, variáveis)
- Acentuação correta sempre (UTF-8)

### Returns Estruturados
- Agentes retornam `## FASE COMPLETA` (sucesso) ou structured feedback (erro)
- O Analyzer pode retornar `## CLARIFICAÇÃO NECESSÁRIA` (caso especial)
- Se nenhum return estruturado: verificar output manualmente antes de avançar

### Retry Direcionado
- NUNCA re-executar o pipeline inteiro por causa de um erro
- Identificar o agente responsável pelo componente com erro
- Re-invocar apenas esse agente com feedback específico
- Max 1 retry por agente. Após isso, apresentar ao usuário
