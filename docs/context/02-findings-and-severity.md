# Modelo de achados e criticidade

## Severidade

| Nível | Significado |
|---|---|
| P0 | Crítico; impacto grave e confirmado ou fortemente evidenciado |
| P1 | Alto; risco relevante que normalmente exige resolução ou decisão formal |
| P2 | Médio; pode permitir ressalva/tratamento planejado conforme política |
| P3 | Baixo; informativo ou melhoria que normalmente não bloqueia |

A severidade não deve ser usada como substituto de confiança ou status.

## Estados de verificação

Estados como estes podem existir independentemente da severidade:

- `verified`
- `not_verified`
- `inconclusive`
- `planned_change`
- `not_relevant`
- `dismissed`

Os nomes finais podem ser adaptados aos nomes já usados na implementação.

## Status de resolução

Sugestão de dimensão separada:

- `open`
- `accepted`
- `fixed`
- `waived`
- `pending_review`

Uma exceção de P0/P1, quando permitida pela política, deve exigir justificativa e autorização explícita; não deve ser equivalente a um simples `dismissed`.

## Confiança

Registrar confiança separadamente, por exemplo:

- `high`
- `medium`
- `low`

Um achado potencialmente P0 com baixa confiança deve provocar investigação/revisão, não ser rebaixado automaticamente para P3.

## Estrutura sugerida

```yaml
finding:
  id: INTENT-001
  stage: intent
  category: requirement_contradiction
  severity: P1
  confidence: high
  verification_status: verified
  resolution_status: open
  root_cause_group: AUTHORIZATION_CHANGE
  requirement_refs:
    - SPEC-123
  evidence:
    - type: diff
      ref: path/to/file
    - type: document
      ref: SPEC-123
  summary: "..."
  impact: "..."
  recommendation: "..."
  operator_action_required: false
```
