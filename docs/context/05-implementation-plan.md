# Plano de implementação para Codex

## Meta

Refatorar/evoluir o Quality Gate para reduzir ruído e perguntas sem perder rigor, mantendo uma única skill com três etapas.

## Fase 1 — Descoberta do repositório
1. Localizar implementação atual da skill.
2. Mapear prompts, schemas, tags e fluxo entre etapas.
3. Identificar onde perguntas ao operador são geradas.
4. Identificar integração atual com BRD/SPEC/TDD/ADRs e resultados externos.
5. Documentar comportamento atual antes de alterar.

**Entrega:** mapa dos componentes e pontos de mudança.

## Fase 2 — Schema comum de achados
1. Criar/normalizar schema compartilhado.
2. Separar:
   - severity
   - confidence
   - verification status
   - resolution status
3. Incluir evidências e referências.
4. Incluir agrupamento por causa raiz.
5. Incluir `operator_action_required`.

**Entrega:** schema + testes.

## Fase 3 — Redução de ruído no Intent Verification
1. Exigir evidência concreta para emitir achado.
2. Distinguir contradição real de divergência textual.
3. Agrupar achados relacionados.
4. Adicionar filtro de relevância/impacto.
5. Evitar pergunta automática para P2/P3 quando política já resolve.
6. Preservar rastreabilidade BRD/SPEC/TDD/ADR -> mudança.

**Entrega:** menos achados redundantes sem perder violações relevantes.

## Fase 4 — Security sem duplicação
1. Inventariar o que já vem de Checkmarx/SonarQube.
2. Remover/rebaixar verificações que apenas repetem SAST genérico.
3. Focar em contexto/requisitos/fluxos de segurança.
4. Permitir correlação com resultados técnicos externos.

**Entrega:** Security Gate complementar ao gate técnico.

## Fase 5 — Test Coverage orientado ao negócio
1. Mapear requisitos/comportamentos afetados.
2. Mapear testes correspondentes.
3. Avaliar fluxos principais, exceções e estados.
4. Não duplicar métricas genéricas de cobertura.
5. Exigir evidência de execução quando configurada como obrigatória.

**Entrega:** cobertura baseada em comportamento/requisito.

## Fase 6 — Política central
1. Implementar `PASS/WARN/REVIEW_REQUIRED/BLOCK`.
2. Tornar regras configuráveis e versionadas.
3. Garantir que ausência de evidência não vire `PASS`.
4. Integrar falhas obrigatórias do gate técnico externo.
5. Registrar motivo exato da decisão.

**Entrega:** engine determinística + testes de tabela.

## Fase 7 — Human review by exception
1. Substituir perguntas por achado por revisão agrupada.
2. Perguntar somente quando necessário para decisão/correção da análise.
3. Continuar análises diagnósticas após bloqueio quando possível.
4. Pausar apenas quando informação humana for indispensável.

**Entrega:** redução mensurável de interações.

## Fase 8 — Validação
Executar contra mudanças reais/representativas e medir:
- total de achados
- achados agrupados
- perguntas ao operador
- falsos positivos
- P0/P1 detectados
- decisões alteradas pelo operador
- duplicações com Checkmarx/SonarQube
- tempo de revisão

Comparar baseline antes/depois.

## Critérios de aceite iniciais

- as três etapas continuam sequenciais numa única skill
- nenhuma quarta etapa/agente decisor livre é criada
- política final é determinística
- P0 confirmado não pode ser descartado informalmente
- achados têm evidência rastreável
- perguntas ao operador são exceção, não padrão
- Security não reproduz SAST genérico já coberto
- Coverage não usa somente percentual genérico como evidência de negócio
- relatório final explica por que o gate passou, alertou, pediu revisão ou bloqueou

## Primeira instrução sugerida ao Codex

> Leia todos os arquivos Markdown deste diretório antes de modificar código. Depois, inspecione o repositório e produza um plano de implementação baseado no estado real da skill. Não altere código até identificar onde cada uma das três etapas, schemas, perguntas ao operador e política de decisão estão implementados. Preserve as fronteiras de responsabilidade descritas nestes documentos e sinalize qualquer conflito entre o desenho desejado e a implementação atual.
