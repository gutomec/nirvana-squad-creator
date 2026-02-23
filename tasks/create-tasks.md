---
task: createTasks()
responsavel: "TaskCreator"
responsavel_type: Agente
atomic_layer: Organism

Entrada:
  - campo: analysisMd
    tipo: file
    origen: "analyzeRequirements() task output"
    obrigatorio: true
  - campo: componentRegistryMd
    tipo: file
    origen: "analyzeRequirements() task output"
    obrigatorio: true
  - campo: agentFiles
    tipo: array<file>
    origen: "createAgents() task output"
    obrigatorio: true
  - campo: taskTemplateMd
    tipo: file
    origen: "templates/task.template.md"
    obrigatorio: true
  - campo: taskFormatMd
    tipo: file
    origen: "references/task-format.md"
    obrigatorio: true

Saida:
  - campo: taskFiles
    tipo: array<file>
    destino: "createWorkflows() task + optimizeSquad() task"
    persistido: true

Checklist:
  pre-conditions:
    - "[ ] analysis.md e component-registry.md existem e são válidos"
    - "[ ] Agentes gerados existem no diretório agents/"
    - "[ ] Template de task disponível em templates/task.template.md"
    - "[ ] Referência de formato disponível em references/task-format.md"
  post-conditions:
    - "[ ] Pelo menos 1 task criada no diretório tasks/"
    - "[ ] Cada task tem campos required: task, responsavel, responsavel_type, atomic_layer, Entrada, Saida, Checklist"
    - "[ ] Contratos Entrada/Saída são encadeados — output da task N referencia input da task N+1"
    - "[ ] Cada Entrada item tem: campo, tipo, origen, obrigatorio"
    - "[ ] Cada Saida item tem: campo, tipo, destino, persistido"
    - "[ ] responsavel de cada task corresponde a um agent.name existente"

Performance:
  duration_expected: "3-8 minutos"
  cost_estimated: "~5000 tokens (Opus)"
  cacheable: false
  parallelizable: false
  skippable_when: "Nunca — tasks são prerequisito para workflows"

Error Handling:
  strategy: retry
  retry:
    max_attempts: 2
    delay: "5s"
  fallback: "Gerar tasks com contratos mínimos e marcar para revisão"
  notification: "orchestrator"

Metadata:
  story: "Como pipeline, preciso gerar definições de tasks com contratos Entrada/Saída encadeados"
  version: "1.0.0"
  dependencies:
    - analyzeRequirements()
    - createAgents()
  author: "Squad Creator"
  created_at: "2026-02-22T00:00:00Z"
  updated_at: "2026-02-22T00:00:00Z"
---

# createTasks()

## Pipeline Diagram

```
┌──────────────┐  ┌────────────────────┐  ┌──────────────┐
│ analysis.md  │  │ component-registry │  │ agents/*.md  │
│              │  │ .md                │  │              │
└──────┬───────┘  └────────┬───────────┘  └──────┬───────┘
       │                   │                     │
       └───────────┬───────┴─────────────────────┘
                   │
                   ▼
          ┌─────────────────┐     ┌───────────────┐  ┌──────────────┐
          │  TaskCreator     │◀────│ task.template │  │ task-format  │
          │  (squad-task-    │     │ .md           │  │ .md          │
          │   creator)       │     └───────────────┘  └──────────────┘
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────┐
          │ tasks/*.md   │
          │ (encadeados) │
          └──────────────┘
              │
              ▼
     ┌────────────────────┐
     │ Contratos           │
     │ Entrada → Saída     │
     │ Task N → Task N+1   │
     └────────────────────┘
```

## Descrição

A task `createTasks()` é a **terceira fase** do pipeline. Gera definições de tasks AIOS com contratos de dados explícitos que conectam o pipeline de ponta a ponta.

### Responsabilidades

1. **Leitura de Inputs** — Consumir a análise, o registry e os agentes gerados para entender o escopo completo do squad.

2. **Geração de Tasks** — Para cada task prevista no `component-registry.md`, gerar um arquivo `.md` com:
   - Bloco YAML com `task` (camelCase()), `responsavel`, `responsavel_type`, `atomic_layer`
   - Array `Entrada` com contratos de input tipados
   - Array `Saida` com contratos de output tipados
   - `Checklist` com `pre-conditions` e `post-conditions`
   - Seções opcionais: Performance, Error Handling, Metadata

3. **Encadeamento de Contratos** — Garantir que:
   - O `destino` de cada `Saida` referencia a task consumidora
   - A `origen` de cada `Entrada` referencia a task produtora
   - Tipos são compatíveis entre produtor e consumidor
   - Campos obrigatórios têm produtores garantidos

4. **Classificação Atomic Layer** — Atribuir a camada correta:
   - `Atom` — operação indivisível simples
   - `Molecule` — combinação de atoms em unidade lógica
   - `Organism` — operação complexa multi-molecule
   - Camadas funcionais quando mais descritivo

### Regras de Geração

- `task` identifier: camelCase seguido de `()`
- `responsavel`: nome legível do agente (PascalCase), não o ID
- Cada task tem pelo menos 1 Entrada e 1 Saida
- Checklist tem pelo menos 1 pre-condition e 1 post-condition
- O diagrama ASCII de pipeline é obrigatório no corpo Markdown
- Descrição detalha responsabilidades, regras e critérios de qualidade

### Validação de Cross-References

| Campo | Validação |
|-------|-----------|
| `responsavel` | Deve corresponder a `agent.name` de um agente existente |
| `Entrada[].origen` | Deve referenciar task existente ou input externo válido |
| `Saida[].destino` | Deve referenciar task existente ou output externo válido |
| `task` identifier | Deve corresponder ao `component-registry.md` |
