# Política de decisão final

## Saídas

- `PASS`
- `WARN`
- `REVIEW_REQUIRED`
- `BLOCK`

## Princípios

A decisão deve ser determinística para o mesmo conjunto de resultados e a mesma versão da política.

Não usar média de severidades. Um único bloqueador pode determinar `BLOCK`.

## Regras iniciais

### BLOCK
Retornar `BLOCK` quando ocorrer, conforme política:
- P0 confirmado e não resolvido/aceito formalmente
- falha em requisito obrigatório
- check obrigatório externo falhou
- requisito obrigatório permanece não verificado quando a política exige evidência
- P1 aberto em categoria configurada como impeditiva

### REVIEW_REQUIRED
Retornar `REVIEW_REQUIRED` quando:
- existe decisão humana obrigatória pendente
- análise crítica está inconclusiva
- uma exceção/waiver requer aprovação
- evidência conflitante impede decisão automática

### WARN
Retornar `WARN` quando:
- não existem bloqueadores
- existem P2/P3 ou riscos aceitos que a política permite liberar com ressalva
- há tratamento planejado válido e não impeditivo

### PASS
Retornar `PASS` quando:
- verificações obrigatórias concluíram satisfatoriamente
- não há bloqueadores
- não há revisão humana obrigatória pendente
- evidência mínima exigida está presente

## Pseudocódigo

```python
def evaluate_quality_gate(results, policy):
    if results.has_mandatory_check_failure(policy):
        return "BLOCK"

    if results.has_unresolved_blocker(policy):
        return "BLOCK"

    if results.has_mandatory_unverified_requirement(policy):
        return "BLOCK"

    if results.has_pending_human_decision(policy):
        return "REVIEW_REQUIRED"

    if results.has_inconclusive_critical_analysis(policy):
        return "REVIEW_REQUIRED"

    if results.has_accepted_warnings(policy):
        return "WARN"

    return "PASS"
```

## Auditoria

O relatório final deve registrar:
- versão da política
- resultado de cada etapa
- achados consolidados
- evidências
- decisões do operador
- exceções/waivers
- motivo determinístico da decisão final
