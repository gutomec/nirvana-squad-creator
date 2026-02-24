# Nirvana Squad Creator

> Gera squads AIOS otimizados a partir de linguagem natural — pipeline de 9 fases com análise, geração, otimização, validação, README multi-idioma, deploy e publicação no squads.sh.

## Instalação

```bash
npx squads add gutomec/nirvana-squad-creator/nirvana-squad-creator
```

## Estrutura

```
nirvana-squad-creator/          # Squad AIOS publicável
├── agents/                     # 9 agentes especializados
├── tasks/                      # 10 tasks com contratos Entrada/Saída
├── workflows/                  # 2 workflows (pipeline + publish)
├── config/                     # Padrões, tech stack, source tree
├── references/                 # Specs completas dos formatos
├── templates/                  # Templates anotados
├── scripts/                    # CLI tools
├── squad.yaml                  # Manifesto do squad
├── README.md                   # Documentação PT-BR
└── README.*.md                 # Traduções (en, zh, hi, es, ar)
```

## Documentação

Veja [nirvana-squad-creator/README.md](nirvana-squad-creator/README.md) para documentação completa.

## Licença

MIT
