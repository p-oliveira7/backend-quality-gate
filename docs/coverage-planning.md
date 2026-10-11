# Test Coverage Gate — planejamento detalhado

## Objetivo

Verificar evidências de teste para os comportamentos de negócio afetados pela mudança. Receber de Intent requisitos e comportamentos, e de Security controles e cenários de risco. Não repetir a interpretação dos requisitos nem uma varredura de segurança; verificar se os testes demonstram as expectativas compartilhadas.

Este documento é planejamento: scripts, schemas, pseudocódigo e exemplos não constituem implementação executável nem testes corporativos executados. Python é uma opção para auxiliares. Coverage é a terceira etapa da mesma skill, seguida por consolidação e política determinística, sem quarto agente decisor.

Percentual de cobertura, mutation testing e resultados do pipeline são evidências externas. Não recalcular nem duplicar esses gates técnicos. Um teste presente, um teste aprovado e uma linha executada são evidências distintas; nenhum deles, isoladamente, demonstra todo o comportamento esperado.

## Entradas e unidade de análise

Entradas: identidade da mudança; requisitos aplicáveis e suas versões; comportamentos afetados; cenários de Security; arquivos de teste e fixtures; configuração de execução; relatórios existentes e política versionada.

A unidade de análise é o **cenário obrigatório ou relevante**, com requisito, precondição, ação e resultado observável. O conjunto de cenários esperados é construído independentemente dos testes encontrados: não permitir que os testes existentes definam sozinhos o que precisa ser coberto.

```json
{
  "id": "SCENARIO-EXPORT-REVOKED",
  "requirement_refs": ["REQ-REVOKE-017"],
  "security_refs": ["SEC-001"],
  "behavior_ref": "REPORT-EXPORT-JOB",
  "precondition": "job pendente e permissao revogada antes da execucao",
  "action": "executar o job",
  "expected_observations": [
    "operacao sensivel negada",
    "nenhum artefato exportado disponibilizado"
  ],
  "mandatory": true,
  "execution_required": true,
  "scope": "worker e controle de autorizacao efetivo"
}
```

Exemplo sintético, válido apenas se o requisito realmente exigir autorização vigente na execução. Não inferir essa obrigação apenas do risco sugerido em Security.

## Catálogo de verificações

| Categoria | Pergunta e gatilho | Evidência esperada | Limite / possível sobreposição externa |
|---|---|---|---|
| `requirement_traceability` | Cada obrigação afetada tem cenários e testes relacionados? | IDs, matriz de rastreabilidade e relação sustentada entre requisito, cenário e teste. | Nome de teste ou similaridade semântica não prova cobertura. |
| `success_path` | O fluxo principal alterado tem resultado verificado? | Precondição, ação e asserções do resultado relevante. | Status de sucesso sozinho pode não provar o efeito de negócio. |
| `rejection_and_error` | Rejeições e falhas relevantes preservam os efeitos esperados? | Entradas/contextos inválidos e asserções sobre erro e ausência de efeitos indevidos. | Não exigir todos os erros imagináveis, somente os aplicáveis. |
| `boundary_and_variation` | Limites, combinações ou variações relevantes foram cobertos? | Valores de fronteira, papéis, tenants, campos ou estados selecionados por requisito. | Evitar produto cartesiano indiscriminado. |
| `state_transition` | A transição de estado preserva invariantes? | Estado anterior, ação, estado posterior e efeitos persistidos. | Não deduzir estado real de retorno HTTP apenas. |
| `security_scenario` | Os cenários aplicáveis de Security possuem evidência de teste? | Atores, recursos, permissões e observações necessárias à expectativa. | Não gerar novamente a vulnerabilidade já registrada em Security. |
| `replay_and_concurrency` | Cenário sensível de repetição/simultaneidade tem teste adequado? | Reenvio/concorrência controlada e verificação da invariante afetada. | Aplicar somente quando a obrigação e o risco justificam. |
| `integration_and_contract` | A fronteira alterada é exercitada com fidelidade suficiente? | Contrato, interação e nível de teste que preserva a propriedade avaliada. | Não exigir end-to-end para tudo; contratos/unitários podem ser suficientes conforme propriedade. |
| `assertion_adequacy` | As asserções verificam a propriedade relevante? | Comparações, efeitos, invariantes e condições negativas pertinentes. | Mutation testing pode ajudar, mas score sozinho não comprova adequação de negócio. |
| `fixture_and_mock_fidelity` | Fixtures/mocks representam o cenário sem substituir o controle sob análise? | Preparação, comportamento dos doubles e ligação com dependências reais/contratos. | Mockar o controle principal pode invalidar a evidência para aquele controle. |
| `execution_evidence` | O teste foi coletado e concluído para a mudança e alvo corretos? | ID do teste, ref da mudança, comando, ambiente, resultado e escopo. | Zero testes, relatório antigo ou teste excluído não validam execução. |
| `reliability_and_isolation` | A evidência depende de estado externo, ordem ou instabilidade conhecida? | Falhas/retries, configuração de isolamento e resultados relacionados ao cenário. | Não inferir flakiness de uma falha isolada; registrar incerteza. |

Essas categorias são dimensões de avaliação, não achados obrigatórios. Um único cenário pode envolver várias dimensões e uma só causa de lacuna. Selecionar pela mudança e política, mantendo as obrigações explícitas.

## Estados separados

Evitar um enum único que mistura presença, adequação e execução. Proposta:

| Dimensão | Valores sugeridos |
|---|---|
| Descoberta | `found`, `not_found_in_scope`, `search_incomplete` |
| Adequação | `adequate`, `insufficient`, `inconclusive`, `not_assessed` |
| Execução | `passed`, `failed`, `not_run`, `skipped`, `disabled`, `inconclusive`, `runner_error`, `expected_failure` |
| Verificação da obrigação | `verified`, `not_verified`, `inconclusive`, `not_relevant` |
| Resolução do achado | `open`, `fixed`, `accepted`, `waived`, `pending_review` |

“Não encontrado” é relativo ao escopo pesquisado. Busca incompleta não prova que o teste não exista. `expected_failure` precisa de interpretação conforme runner e política: não é sucesso automático da obrigação. Uma asserção que espera rejeição e passa é diferente de um teste marcado como falha esperada.

Se a execução é mandatória, um teste adequado mas não executado não verifica a obrigação. Se vários testes sustentam o mesmo cenário, usar uma regra explícita de composição, por exemplo todos os testes designados obrigatórios, em vez de um sucesso arbitrário.

## Algoritmo do funil

1. Normalizar expectativas de Intent/Security preservando origem, versão e ambiguidades.
2. Construir catálogo de cenários esperados por requisito e consequência, sem depender da existência de testes.
3. Indexar testes no escopo: IDs, localização, parametrizações, fixtures, mocks e asserções relevantes.
4. Mapear primeiro relações explícitas; usar busca estrutural e semântica para recuperar candidatos; confirmar a ligação.
5. Avaliar precondição, ação, observações e fidelidade do teste para cada cenário.
6. Correlacionar relatórios externos com os testes, parâmetros, alvo e referência da mudança.
7. Classificar presença, adequação e execução separadamente e avaliar obrigações pela política.
8. Agrupar lacunas pela causa/correção, sem perder cenários ou evidências.
9. Emitir resultados e pendências para a consolidação final, sem aprovação individual do operador.

Ambiguidade já presente em Intent/Security permanece referenciada. Se não impede uma verificação parcial útil, continuar essa verificação e registrar o limite. Se for indispensável, tentar resolver pelas fontes antes de uma pergunta mínima. Não inventar expectativas para obter uma decisão.

## Scripts propostos

```text
schemas/behavior-scenario.schema.json
schemas/test-evidence.schema.json
schemas/coverage-result.schema.json
policies/coverage-v1.json
scripts/build_behavior_catalog.py
scripts/index_tests.py
scripts/normalize_test_results.py
scripts/validate_test_evidence.py
scripts/triage_coverage.py
scripts/render_report.py
```

| Componente | Método | Limite |
|---|---|---|
| `build_behavior_catalog.py` | Normalizar cenários estruturados, IDs e proveniência; validar duplicatas e vínculos. | Interpretação de texto livre pertence à análise semântica, não a heurística de palavras. |
| `index_tests.py` | Adaptadores por framework, AST quando disponível e metadados de coleta. | Busca por nome não prova execução; macros e geração exigem adaptadores específicos. |
| `normalize_test_results.py` | Adaptadores de runner preservam IDs, parâmetros, estados, ref e escopo. | Contagem agregada ou XML sem proveniência pode ser insuficiente para ligar à mudança. |
| Análise semântica na skill | Relacionar cenário ao corpo do teste e avaliar observações e mocks. | A interpretação pode variar e precisa de validação por casos. |
| `validate_test_evidence.py` | Conferir schema, referências, arquivos, IDs e compatibilidade dos relatórios. | Relatório aprovado não prova qualidade das asserções. |
| `triage_coverage.py` | Aplicar regras por cenário, composição de evidências e agrupamento estável. | Não substituir ausência de prova por percentual de cobertura. |
| `render_report.py` | Mostrar matriz requisito → cenário → teste → resultado e motivos. | Não alterar a decisão ao redigir o relatório. |

Reutilizar relatórios existentes. Se a política autorizar execução, usar comandos e alvos do repositório com isolamento apropriado, preservar códigos de saída e confirmar testes realmente coletados. Não executar integrações destrutivas ou com dados de produção como consequência automática deste planejamento. A execução proposta não exige criar testes nem alterar código durante a análise.

## Regras de triagem

| Regra | Condição | Encaminhamento |
|---|---|---|
| `C-01` | Referência inválida ou busca incompleta | Recuperar evidência; registrar limite; não declarar ausência global. |
| `C-02` | Cenário mandatário sem teste correspondente no escopo suficiente | Registrar lacuna e obrigação não verificada. |
| `C-03` | Teste relacionado, mas não verifica observação necessária | Registrar inadequação indicando exatamente a observação ausente. |
| `C-04` | Mock substitui o controle que deveria ser verificado | Considerar evidência insuficiente para esse controle, preservando o que ela comprova. |
| `C-05` | Execução exigida e teste não executado, excluído ou desabilitado | Registrar não verificado; aplicar política mandatória. |
| `C-06` | Resultado falhou | Referenciar falha; investigar natureza antes de atribuir defeito de negócio. |
| `C-07` | Relatório de outra mudança, alvo ou parâmetros | Não admitir como execução suficiente; conservar referência com motivo. |
| `C-08` | Teste adequado e evidência válida satisfazem o cenário | Registrar obrigação verificada. |
| `C-09` | Falha técnica obrigatória já registrada externamente | Correlacionar ao cenário; não duplicar a causa nem ocultar o bloqueio. |
| `C-10` | Expectativa crítica inconclusiva após investigação | Manter pendência com origem; aplicar política final, sem descarte. |
| `C-11` | Mesma causa/correção afeta vários cenários | Agrupar preservando todos os vínculos e impactos. |

Severidade da lacuna segue o impacto do comportamento sem validação, não a contagem de testes ausentes. Falha de execução, por si só, não determina P0. Confiança não substitui evidência ou resolução.

## Pseudocódigo ilustrativo

Funções a especificar na implementação corporativa, não código executável/testado:

```python
def analyze_coverage(change, intent, security, sources, reports, policy):
    scenarios = build_expected_scenarios(intent, security, sources, policy)
    tests, search_scope = index_tests(sources, policy)
    runs = normalize_and_validate_runs(reports, change, policy)
    rows, pending = [], []

    for scenario in scenarios:
        matches = find_and_validate_test_links(scenario, tests, sources)
        discovery = assess_discovery(matches, search_scope)
        adequacy = assess_assertions_and_fidelity(scenario, matches)
        execution = correlate_execution(scenario, matches, runs)
        outcome = apply_scenario_rules(
            scenario, discovery, adequacy, execution, policy
        )
        rows.append(outcome)
        pending.extend(outcome.pending)

    findings = derive_supported_gaps(rows, policy)
    groups = consolidate_supported_root_causes(findings, policy)
    return {
        "scenario_matrix": rows,
        "findings": groups,
        "pending": pending,
        "external_check_refs": preserve_external_results(runs),
        "obligation_status": evaluate_obligations(rows, policy),
        "policy_version": policy.version,
    }
```

Derivar lacunas de uma matriz por cenário permite preservar obrigações mesmo quando o modelo não gera candidatos espontaneamente. Scripts podem tornar o encaminhamento determinístico; a adequação semântica continua sujeita a avaliação própria.

## Exemplo completo

Security entrega o risco de job exportar dados após revogação; Intent fornece o requisito que exige permissão vigente no momento da execução.

1. O catálogo inclui sucesso com permissão vigente e rejeição após revogação.
2. Um teste existente verifica apenas que a API responde que o job foi aceito.
3. Ele pode comprovar agendamento, mas não execução autorizada nem ausência de exportação após revogação.
4. Mesmo se esse teste passou e o percentual externo é alto, o cenário de revogação continua não verificado.
5. Se outro teste executa o worker com controle real, revoga a permissão e verifica rejeição e ausência do artefato, avaliar sua adequação e execução.
6. Se adequado, mas não executado para a mudança quando obrigatório, manter `not_verified`.
7. Se adequado e executado com evidência válida satisfatória, registrar `verified` para esse cenário. Isso não resolve automaticamente a vulnerabilidade: as evidências de Security devem ser reconciliadas, e conflitos permanecem explícitos.

Saída ilustrativa do teste de agendamento insuficiente:

```json
{
  "scenario_ref": "SCENARIO-EXPORT-REVOKED",
  "test_refs": ["TEST-EXPORT-ACCEPTED"],
  "discovery_status": "found",
  "adequacy_status": "insufficient",
  "execution_status": "passed",
  "verification_status": "not_verified",
  "rule_ids": ["C-03"],
  "missing_observations": [
    "rejeicao pelo worker apos revogacao",
    "ausencia de artefato exportado disponibilizado"
  ],
  "operator_action_required": false
}
```

O resultado aprovado pertence ao teste relacionado, não comprova o cenário inteiro. `operator_action_required: false` evita pergunta de classificação; não dispensa corrigir a lacuna mandatória.

## Validação e adoção

1. Inventariar runners, testes, parametrizações e formatos reais do pipeline.
2. Definir regras de composição por cenário e evidências mínimas.
3. Criar baseline de matrizes e decisões com casos representativos.
4. Comparar novo funil ao atual sem alterar produção inicialmente.
5. Revisar falsos vínculos, cenários omitidos e classificação de asserções/mocks.
6. Adotar o contrato estruturado e medir perguntas e consistência.

Casos necessários: teste presente mas sem asserção relevante; mock do controle; relatório antigo; zero testes; parametrização excluída; skipped/disabled; erro de runner; falha esperada; teste com rejeição esperada que passa; repetição/concorrência; busca incompleta; controles equivalentes; cenário comprovado por conjunto de testes; conflito entre teste e achado de Security.

Medir rastreabilidade, cenários mandatórios verificados, falsos positivos, vínculos incorretos, riscos omitidos, perguntas, tempo e variação semântica. Aumentar percentuais não substitui validar comportamentos.

## Decisões ainda necessárias

Formatos de execução, composição de testes por cenário, referências de mudança, validade temporal, testes obrigatórios, ambientes representativos, regras de mocks, tabelas de impacto, critérios de busca suficiente e responsabilidades dos checks externos. O planejamento pode evoluir com exemplos sintéticos sem compartilhar código corporativo.
