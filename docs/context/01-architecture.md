# Arquitetura do Quality Gate

## Fluxo principal

```text
Input da mudança
  |
  v
Intent Verification
  |
  v
Security Gate
  |
  v
Test Coverage Gate
  |
  v
Consolidação
  |
  v
Política determinística
  |
  +--> PASS
  +--> WARN
  +--> REVIEW_REQUIRED
  +--> BLOCK
```

As três etapas ficam na mesma skill/orquestrador e compartilham evidências e contexto.

## 1. Intent Verification

### Objetivo
Verificar se a implementação continua alinhada à intenção documentada do produto/sistema.

### Fontes
- diff/código alterado
- BRD
- SPEC
- TDD
- ADRs

### Responsabilidades
- extrair/normalizar claims relevantes
- criar rastreabilidade requisito -> implementação
- detectar contradições, omissões e mudanças de comportamento
- executar análise de impacto
- distinguir violação real de mera divergência documental
- agrupar divergências relacionadas por causa raiz

### Não fazer
- gerar um achado para toda diferença textual
- assumir que documentação diferente implica defeito
- bloquear sem evidência suficiente

## 2. Security Gate

### Objetivo
Avaliar riscos de segurança que dependem do contexto da mudança, dos requisitos e das regras do negócio.

### Referências
- requisitos de segurança do projeto
- OWASP Top 10 como taxonomia/referência
- recomendações AWS aplicáveis ao contexto

### Responsabilidades
- correlacionar mudanças com requisitos/controles esperados
- identificar enfraquecimento de autorização, isolamento, validações ou fluxos sensíveis quando sustentado por evidências
- interpretar impacto de achados técnicos externos no contexto do negócio

### Não fazer
- reproduzir SAST genérico
- duplicar Checkmarx/SonarQube
- classificar algo como crítico apenas por pertencer a uma categoria OWASP

## 3. Test Coverage Gate

### Objetivo
Verificar se existe evidência de teste adequada para os comportamentos de negócio afetados.

### Responsabilidades
- mapear requisito/comportamento -> teste
- verificar fluxos principais
- verificar exceções e estados relevantes
- verificar evidência de execução quando disponível
- sinalizar comportamento importante sem validação adequada

### Não fazer
- duplicar métricas genéricas de cobertura do SonarQube
- tratar porcentagem de cobertura como prova suficiente de cobertura de negócio
