# Interação com o operador

## Problema observado

Os primeiros testes produziram levantamentos e perguntas em excesso, principalmente em Intent Verification.

## Estratégia

Aplicar **human review by exception**.

### 1. Classificação automática primeiro
Cada etapa deve tentar:
- reunir evidências
- classificar severidade
- estimar confiança
- sugerir status
- agrupar achados por causa raiz

### 2. Não perguntar achado por achado
Consolidar achados relacionados e permitir revisão em lote.

### 3. Perguntar somente quando a resposta altera o caminho
Exemplos:
- ambiguidade relevante que impede classificar um requisito crítico
- decisão formal de aceitar um risco quando a política permite
- confirmação de mudança planejada que altera a interpretação do diff
- evidência indispensável ausente e não inferível

### 4. Não interromper apenas porque existe um bloqueio
Se Intent encontrar um P0 confirmado:
- registrar bloqueio provisório
- continuar Security e Test Coverage para completar o diagnóstico, quando possível
- manter a decisão final bloqueada até resolução/aceite formal

Pausar imediatamente somente quando a resposta do operador for necessária para que as próximas análises sejam corretas.

## Filtro de relevância

Antes de emitir um achado, perguntar internamente:
1. Existe evidência concreta?
2. O achado representa uma causa raiz distinta?
3. Ele pode alterar PASS/WARN/REVIEW_REQUIRED/BLOCK ou exigir ação relevante?
4. Já existe ferramenta técnica responsável por esse mesmo tipo de detecção?

Se as respostas indicarem ruído/duplicação, não criar um novo achado bloqueante; no máximo consolidar como informação quando útil.

## Objetivo de UX

O operador deve revisar exceções e decisões importantes, e não funcionar como classificador manual de todos os resultados da IA.
