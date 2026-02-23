---
task: discoverSkills()
responsavel: "SkillsScout"
responsavel_type: Agente
atomic_layer: Molecule

Entrada:
  - nome: analysisMd
    tipo: file
    descricao: "analyzeRequirements() task output"
    obrigatorio: true
  - nome: componentRegistryMd
    tipo: file
    descricao: "analyzeRequirements() task output"
    obrigatorio: true
  - nome: squadYaml
    tipo: file
    descricao: "createWorkflows() task output"
    obrigatorio: true
  - nome: ccSkillReportMd
    tipo: file
    descricao: "createCcSkill() task output"
    obrigatorio: false

Saida:
  - nome: skillsDiscoveryReportMd
    tipo: file
    descricao: "orquestrador (para instalação interativa)"
    obrigatorio: true

Checklist:
  pre-conditions:
    - "[ ] CC Skill gerado (fase 9 completa)"
    - "[ ] analysis.md e component-registry.md disponíveis para contexto"
    - "[ ] squad.yaml disponível com nome e domínio do squad"
    - "[ ] Conexão de rede disponível para acesso à API Skyll"
  post-conditions:
    - "[ ] skills-discovery-report.md gerado com ranking de skills"
    - "[ ] Cada skill rankeada com score 0-100 baseado nos 5 critérios"
    - "[ ] Recomendações de instalação com comando npx skills add"
    - "[ ] Análise de gap coverage vs capacidades existentes"
    - "[ ] Nenhuma skill duplicada com agente existente no squad"

Performance:
  duration_expected: "1-3 minutos"
  cost_estimated: "~1500 tokens (Sonnet) + API calls (Skyll)"
  cacheable: true
  parallelizable: false
  skippable_when: "Usuário recusar skills discovery quando perguntado"

Error Handling:
  strategy: fallback
  fallback: "Se API Skyll indisponível, gerar relatório com nota 'API unavailable' e skip da fase"
  notification: "orchestrator"

Metadata:
  story: "Como squad, posso ser complementado com skills da comunidade que cobrem gaps"
  version: "1.0.0"
  dependencies:
    - createCcSkill()
  author: "Squad Creator"
  created_at: "2026-02-22T00:00:00Z"
  updated_at: "2026-02-22T00:00:00Z"
---

# discoverSkills()

## Pipeline Diagram

```
┌──────────────┐  ┌────────────────────┐  ┌──────────┐  ┌──────────────────┐
│ analysis.md  │  │ component-registry │  │ squad    │  │ cc-skill-report  │
│              │  │ .md                │  │ .yaml    │  │ .md (opcional)   │
└──────┬───────┘  └────────┬───────────┘  └────┬─────┘  └────────┬─────────┘
       │                   │                   │                  │
       └───────────┬───────┴───────────────────┴──────────────────┘
                   │
                   ▼
          ┌─────────────────┐        ┌─────────────────┐
          │  SkillsScout     │───────▶│  API Skyll       │
          │  (squad-skills-  │◀───────│  (WebFetch GET)  │
          │   scout)         │        │  api.skyll.app   │
          └────────┬─────────┘        └─────────────────┘
                   │
                   ▼
          ┌──────────────────────┐
          │ skills-discovery-    │
          │ report.md            │
          │                      │
          │ - Ranking (0-100)    │
          │ - Gap analysis       │
          │ - Install commands   │
          └──────────────────────┘
                   │
                   ▼
          ┌──────────────────────┐
          │ Orchestrator         │
          │ (instalação          │
          │  interativa com      │
          │  aprovação do        │
          │  usuário)            │
          └──────────────────────┘
```

## Descrição

A task `discoverSkills()` é a **décima fase** do pipeline e é **opcional**. Busca skills complementares da comunidade via API Skyll para enriquecer o squad gerado.

### Responsabilidades

1. **Construção de Queries** — A partir do contexto do squad:
   - Extrair keywords do domínio (analysis.md)
   - Identificar capacidades do squad (component-registry.md)
   - Mapear gaps — áreas que nenhum agente cobre adequadamente
   - Formular queries de busca relevantes

2. **Busca via API Skyll** — Para cada query:
   - Endpoint: `https://api.skyll.app/search?q=<query>&limit=10`
   - Método: GET via WebFetch
   - Sem autenticação (API pública e gratuita)
   - Parsear resultados JSON

3. **Ranking de Skills** — Avaliar cada skill encontrada com score 0-100:

   | Critério | Peso | Descrição |
   |----------|------|-----------|
   | Relevância API | 30% | Score retornado pela Skyll |
   | Match de domínio | 25% | Keywords do squad vs descrição da skill |
   | Gap fill | 25% | Cobre área que nenhum agente cobre |
   | Popularidade | 10% | `install_count` normalizado |
   | Não-overlap | 10% | Penalidade se duplica agente existente |

4. **Geração do Relatório** — `skills-discovery-report.md`:
   - Ranking das top skills encontradas
   - Para cada skill: nome, descrição, score, critérios individuais
   - Análise de gap coverage
   - Comandos de instalação: `npx skills add <repo> --skill <name> -y -a claude-code`
   - Recomendação final (install / skip / investigate)

5. **Instalação Interativa** — Após gerar o relatório:
   - Apresentar ranking ao usuário
   - Para cada skill recomendada, perguntar: instalar? (sim/não)
   - Se sim, executar comando de instalação
   - Se não, registrar como "skipped" no relatório

### Fase Opcional

- O pipeline pergunta ao usuário se deseja executar skills discovery
- Se recusar, a fase é marcada como "skipped" no state tracking
- O pipeline finaliza normalmente sem esta fase
