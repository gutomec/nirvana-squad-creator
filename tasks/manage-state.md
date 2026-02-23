---
task: manageState()
responsavel: "Orchestrator"
responsavel_type: Agente
atomic_layer: Molecule

Entrada:
  - nome: sessionName
    tipo: string
    descricao: "user input ou --resume flag"
    obrigatorio: true
  - nome: phaseNumber
    tipo: string
    descricao: "pipeline progress"
    obrigatorio: false
  - nome: notes
    tipo: string
    descricao: "phase completion notes"
    obrigatorio: false

Saida:
  - nome: stateJson
    tipo: object
    descricao: "all pipeline phases (state tracking)"
    obrigatorio: true
  - nome: stateMd
    tipo: file
    descricao: ".squad-workspace/<session>/STATE.md"
    obrigatorio: true

Checklist:
  pre-conditions:
    - "[ ] Sessão inicializada ou existente no .squad-workspace/"
    - "[ ] CLI squad-tools.cjs disponível e funcional"
    - "[ ] Parâmetros de entrada válidos (sessionName não-vazio)"
  post-conditions:
    - "[ ] Estado atualizado atomicamente via squad-tools.cjs"
    - "[ ] config.json reflete o estado correto da sessão"
    - "[ ] STATE.md gerado/atualizado com resumo legível"
    - "[ ] Fase anterior marcada como completed (se avançando)"
    - "[ ] Nenhuma perda de estado em caso de falha"

Tools:
  - tool_name: squad-tools.cjs
    version: "1.0.0"
    used_for: "Gerenciamento atômico de estado do pipeline"
    shared_with:
      - analyzeRequirements()
      - createAgents()
      - createTasks()
      - createWorkflows()
      - optimizeSquad()
      - validateSquad()
      - createMultilingualReadme()
      - deploySquad()
      - createCcSkill()
      - discoverSkills()
      - publishSquad()
    cost: free
    cacheable: false

Performance:
  duration_expected: "<5 segundos"
  cost_estimated: "~100 tokens (operação local)"
  cacheable: false
  parallelizable: false
  skippable_when: "Nunca — state management é infraestrutura obrigatória"

Error Handling:
  strategy: retry
  retry:
    max_attempts: 3
    delay: "1s"
  fallback: "Log erro e continuar pipeline — estado será reconciliado na próxima operação"
  notification: "orchestrator"

Metadata:
  story: "Como pipeline, preciso de gerenciamento atômico de estado para suportar resume e tracking"
  version: "1.0.0"
  dependencies: []
  author: "Squad Creator"
  created_at: "2026-02-22T00:00:00Z"
  updated_at: "2026-02-22T00:00:00Z"
---

# manageState()

## Pipeline Diagram

```
┌─────────────────┐  ┌──────────────┐  ┌──────────────┐
│ sessionName     │  │ phaseNumber  │  │ notes        │
│ (string)        │  │ (optional)   │  │ (optional)   │
└────────┬────────┘  └──────┬───────┘  └──────┬───────┘
         │                  │                 │
         └──────────┬───────┴─────────────────┘
                    │
                    ▼
           ┌─────────────────┐
           │  Orquestrador    │
           │  (state mgmt)    │
           └────────┬─────────┘
                    │
                    ▼
           ┌─────────────────┐
           │ squad-tools.cjs │
           │ (CLI)           │
           └────────┬─────────┘
                    │
           ┌────────┴────────┐
           │                 │
           ▼                 ▼
    ┌─────────────┐  ┌─────────────┐
    │ config.json │  │ STATE.md    │
    │ (state)     │  │ (readable)  │
    └─────────────┘  └─────────────┘
```

## Descrição

A task `manageState()` é uma task **transversal** usada por todas as fases do pipeline. Gerencia o estado da sessão de forma atômica usando o CLI `squad-tools.cjs`.

### Responsabilidades

1. **Inicialização de Sessão** — Quando o pipeline inicia:
   ```bash
   node bin/squad-tools.cjs init <session> [--preset=padrao]
   ```
   - Cria diretório `.squad-workspace/<session>/`
   - Gera `config.json` com estado inicial
   - Define metadata da sessão (timestamp, preset, etc.)

2. **Resumo de Sessão** — Quando o usuário retorna:
   ```bash
   node bin/squad-tools.cjs resume <session>
   ```
   - Carrega estado existente de `config.json`
   - Identifica última fase completa
   - Permite continuar de onde parou

3. **Avanço de Fase** — Quando uma fase é concluída:
   ```bash
   node bin/squad-tools.cjs state advance <session> --phase=N [--notes="..."]
   ```
   - Marca fase N como `completed`
   - Registra timestamp de conclusão
   - Adiciona notas opcionais
   - Avança current_phase para N+1

4. **Gate de Validação** — Quando o Validator produz resultado:
   ```bash
   node bin/squad-tools.cjs state gate <session> --phase=N --result=approved
   ```
   - Registra resultado do gate (approved/rejected)
   - Se rejected, marca para re-execução

5. **Consulta de Estado** — A qualquer momento:
   ```bash
   node bin/squad-tools.cjs state get <session>
   ```
   - Retorna estado atual em JSON

6. **Validação por Fase** — Verificar artefatos de uma fase:
   ```bash
   node bin/squad-tools.cjs validate <session> --phase=N
   ```
   - Verifica que artefatos esperados existem
   - Fases válidas: 1-9

7. **Snapshot** — Backup do estado atual:
   ```bash
   node bin/squad-tools.cjs snapshot <session>
   ```
   - Cria snapshot do workspace completo

### STATE.md

Além do `config.json` (machine-readable), gera um `STATE.md` (human-readable):

```markdown
# Pipeline State — <session>

| Fase | Nome | Status | Agente | Início | Fim | Notas |
|------|------|--------|--------|--------|-----|-------|
| 1 | Análise | completed | Analyzer | ... | ... | ... |
| 2 | Agentes | completed | AgentCreator | ... | ... | ... |
| 3 | Tasks | in_progress | TaskCreator | ... | - | ... |
| ... | ... | ... | ... | ... | ... | ... |

## Artefatos Gerados
- agents/: 5 arquivos
- tasks/: 8 arquivos
- workflows/: 2 arquivos
- ...
```

### Atomicidade

- Toda operação de estado é atômica — ou completa ou não faz efeito
- `config.json` é gravado de forma atômica (write-to-temp + rename)
- Em caso de falha, o estado anterior é preservado
- STATE.md é derivado de config.json e pode ser regenerado
