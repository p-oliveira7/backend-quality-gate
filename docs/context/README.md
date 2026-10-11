# Quality Gate — Contexto para Codex

## Objetivo

Construir uma única skill/orquestrador de **Quality Gate de negócio**, executando três etapas sequenciais e compartilhando contexto/evidências:

1. Intent Verification
2. Security Gate
3. Test Coverage Gate

Não criar uma quarta etapa neste momento.

## Fronteira de responsabilidade

Este Quality Gate não deve duplicar o gate técnico já existente.

**Gate técnico externo**
- Checkmarx
- SonarQube
- mutation testing
- checks estáticos/pipeline e validações técnicas já existentes

**Quality Gate desta skill**
- interpretação da intenção e dos requisitos
- riscos de segurança ligados ao contexto/regras do negócio
- cobertura de testes orientada aos comportamentos/requisitos do negócio
- correlação com evidências técnicas externas quando disponíveis

## Princípios

- Ausência de evidência não significa aprovação.
- Severidade, confiança e status de resolução são dimensões separadas.
- Perguntar ao operador apenas quando a decisão humana for realmente necessária.
- Agrupar achados relacionados por causa raiz.
- Evitar achados que não possam influenciar a decisão do gate.
- Continuar as análises diagnósticas mesmo quando um bloqueio já tiver sido encontrado, salvo quando uma resposta humana for indispensável para analisar corretamente as etapas seguintes.
- A decisão final deve ser baseada em política explícita/determinística, não em julgamento livre de um quarto agente.

## Documentos relacionados

- `01-architecture.md` — arquitetura e responsabilidades
- `02-findings-and-severity.md` — modelo de achados, criticidade e estados
- `03-operator-interaction.md` — estratégia para reduzir perguntas
- `04-decision-policy.md` — política final do gate
- `05-implementation-plan.md` — plano incremental para o Codex
