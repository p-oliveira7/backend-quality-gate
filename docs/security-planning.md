# Security Gate de APIs — planejamento detalhado

## Objetivo e escopo

Complementar o gate técnico com análise de segurança que depende de identidade, regras do negócio, estados, isolamento e relações entre serviços. Evitar perguntas por candidato e achados que apenas repetem Checkmarx ou SonarQube.

Este documento propõe arquitetura e algoritmo. Não implementa scripts, não examina código corporativo e não comprova capacidades das ferramentas do pipeline. Python é uma opção para auxiliares, não uma exigência de tecnologia.

O desenho mantém uma única skill: Intent → Security → Coverage. Security recebe contexto e evidências de Intent, aplica seu próprio funil e avança sem aprovação do operador. Um bloqueio de aprovação não interrompe Coverage; uma dependência indispensável à execução pode exigir uma pergunta mínima.

## Referências e fronteiras

- [OWASP API Security Top 10 — edição 2023](https://owasp.org/API-Security/editions/2023/en/0x11-t10/): taxonomia adotada explicitamente neste planejamento, sem alegação de que é a edição mais recente.
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/): fonte complementar para selecionar controles verificáveis. Na adoção, fixar versão e IDs dos requisitos efetivamente escolhidos.
- Requisitos, ADRs, contrato de API e política corporativa: definem as obrigações e o impacto aplicáveis ao sistema.

O Top 10 não é uma especificação completa de controles nem um catálogo exaustivo de riscos. Não determina severidade. Não executar todas as categorias para toda mudança: selecionar verificações pelo fluxo afetado, mantendo explícitas as lacunas de inventário e de evidência.

## Catálogo de categorias OWASP API 2023

Os nomes abaixo seguem a edição indicada. A coluna de sobreposição identifica possibilidades, não comprova cobertura efetiva de nenhuma ferramenta ou produto.

| ID e categoria | Gatilho de aplicabilidade | Pergunta contextual e evidência esperada | Possível sobreposição técnica |
|---|---|---|---|
| API1 — Broken Object Level Authorization | Acesso a recurso identificado por ID, consultas, lote ou mudança de proprietário/tenant. | O sujeito pode agir sobre este objeto? Relacionar política de propriedade/tenant, seleção do objeto, controle aplicado e cenário permitido/negado. | Fluxos de dados e ausência de checks podem ser sinalizados; é preciso confirmar se a ferramenta conhece a regra de propriedade. |
| API2 — Broken Authentication | Mudança em login, tokens, identidade, sessões, delegação ou revogação. | A identidade e sua validade são preservadas no fluxo? Usar contrato de identidade, validação, expiração e comportamento após revogação. | Bibliotecas, criptografia e padrões de validação podem já estar cobertos. Analisar somente o contexto restante. |
| API3 — Broken Object Property Level Authorization | Alteração de campos aceitos, retornados ou atualizados, serialização e operações parciais. | O sujeito pode ler ou alterar cada propriedade sensível? Usar matriz de campos/papéis e comportamento de entrada/saída. | Mass assignment e exposição técnica podem ser detectados; permissões de negócio por campo exigem contexto. |
| API4 — Unrestricted Resource Consumption | Paginação, tamanho de payload, consultas, upload, fan-out, jobs ou chamadas com custo. | Os limites protegem a operação e seus efeitos financeiros/operacionais? Demonstrar orçamento, multiplicadores e limites em cada camada relevante. | Ausência de limites e operações caras podem ser sinalizadas; orçamento e custo do fluxo dependem do negócio. |
| API5 — Broken Function Level Authorization | Nova operação, rota administrativa, método alternativo ou alteração de papéis. | O papel pode executar essa função por todos os caminhos relevantes? Usar matriz de funções, rotas e controles efetivos. | Checks de autorização e configurações podem estar cobertos; fronteira entre funções exige política. |
| API6 — Unrestricted Access to Sensitive Business Flows | Operações de reserva, compra, benefício, emissão ou outra capacidade sujeita a abuso. | Uso autorizado pode abusar de um fluxo sensível? Usar limites por ator, regras de negócio, estados e consequências. | Algumas regras de proteção podem aparecer nos scanners; identificação de abuso legítimo é contextual. |
| API7 — Server Side Request Forgery | Mudança em destino de chamada, URL fornecida externamente ou redirecionamento. | O fluxo mantém a fronteira de destinos permitidos? Relacionar origem do destino, validações e política de egress. | SSRF genérico é candidato forte a cobertura externa; não duplicar a análise técnica quando comprovada. |
| API8 — Security Misconfiguration | Mudança em deployment, gateway, erros, ambiente, CORS, headers ou configuração. | A configuração efetiva preserva a fronteira de exposição prevista? Usar artefato de configuração e topologia aplicável. | Regras estáticas/IaC podem cobrir partes; não presumir que configuração em código é a configuração efetiva. |
| API9 — Improper Inventory Management | Nova rota, versão, alias, descontinuação ou exposição de endpoint legado. | O inventário e a política cobrem todos os caminhos expostos? Usar inventário, gateway, versões e estado de descontinuação. | Código pode revelar endpoints, mas a exposição real exige evidência de deployment/inventário. |
| API10 — Unsafe Consumption of APIs | Mudança em parceiro, webhook, resposta externa ou integração. | Dados e ações externos são tratados com confiança adequada? Usar contrato, identidade de origem, validação e transições permitidas. | Validação, desserialização e fluxo de dados podem ser cobertos; autorização do efeito de negócio exige contexto. |

## Quatro frentes contextuais complementares

Estas frentes são lentes de análise, não quatro novos achados obrigatórios. O mapeamento OWASP é muitos-para-muitos; uma causa pode receber vários rótulos sem duplicar o achado. Nem todo defeito de idempotência ou concorrência é uma vulnerabilidade: exigir consequência de segurança ou abuso demonstrável.

| Categoria interna | Relação possível com OWASP | Aplicabilidade e cenário | Evidência mínima para sustentar a análise |
|---|---|---|---|
| `business_flow_abuse` | API6, API4 | Operação legítima permite benefício indevido, esgotamento de disponibilidade ou custo fora do limite. | Fluxo sensível, ator autorizado, regra/limite esperado, mecanismo de controle e consequência concreta. |
| `replay_and_concurrency` | API6 e, dependendo do efeito, API4 | Reenvio ou simultaneidade viola uma invariante, duplica efeito sensível ou ultrapassa limite. | Invariante, fronteira transacional, tratamento de repetição/concorrência e evidência do efeito; não basta ausência de um header de idempotência. |
| `permission_lifecycle` | API1, API2, API3, API5 | Revogação, troca de tenant/papel, cache, delegação ou execução assíncrona preserva acesso indevido. | Evento de mudança, regra temporal de validade, identidade usada em cada passo e comportamento do controle após a mudança. |
| `cross_service_trust` | API10, API2, API1, API5; API7 quando houver destino externo controlável | Webhook, parceiro ou job cruza fronteira de confiança e produz ação não autorizada. | Origem e identidade, contrato, fronteira de confiança, validação e autorização do efeito no estado do negócio. |

## Seleção das verificações

1. Coletar a mudança, rotas e métodos afetados, chamadas internas/externas e configurações relevantes.
2. Relacionar atores, papéis, tenants, objetos, campos sensíveis, estados e efeitos.
3. Identificar obrigações aplicáveis em requisitos e controles corporativos.
4. Selecionar categorias pelas relações demonstradas; busca semântica recupera possibilidades, não prova aplicabilidade.
5. Consultar a matriz de responsabilidade dos checks externos para cada condição.
6. Analisar o risco contextual restante e avaliar as obrigações, mesmo sem candidato emitido pelo modelo.

Uma configuração explícita pode definir verificações de base obrigatórias para o fluxo. Inventário incompleto ou relação incerta não deve ser classificado automaticamente como não aplicável.

## Contrato de cobertura externa e deduplicação

Separar **responsabilidade prevista** de **resultado observado**. O fato de um produto poder detectar um problema não prova que a regra esteja habilitada ou que a execução tenha abrangido a mudança.

Para cada check externo, registrar:

- ferramenta, modalidade de análise e perfil/regra;
- referência da mudança e escopo analisado;
- estado: passou, falhou, não executado, inconclusivo ou erro;
- evidência e momento da execução, com janela de validade definida pela política;
- condições cobertas e limitações conhecidas.

| Situação | Encaminhamento |
|---|---|
| Mesma condição, mesmo fluxo/escopo e resultado externo suficiente e atual | Referenciar o resultado; não repetir o achado técnico. |
| Achado externo válido com efeito contextual relevante | Enriquecer o mesmo registro com requisito e impacto, preservando sua origem. |
| Mesma categoria OWASP, mas condição ou fluxo diferente | Não deduplicar apenas pelo rótulo; investigar relação. |
| Check atribuído externamente, mas ausente, incompatível com a mudança ou insuficiente | Registrar lacuna; aplicar política de obrigatoriedade, sem presumir aprovação ou substituir por varredura genérica. |
| Evidências externas e contextuais conflitantes | Investigar escopo e significado; manter conflito explícito se persistir. |

Chave sugerida para correlação: controle/regra + operação + recurso + fronteira de confiança + referência da mudança. A chave facilita busca; equivalência ainda precisa ser sustentada. Excluir varredura genérica não elimina a obrigação de registrar cobertura ausente.

## Scripts e artefatos propostos

```text
schemas/security-candidate.schema.json
schemas/external-check.schema.json
policies/security-v1.json
scripts/collect_api_context.py
scripts/normalize_external_checks.py
scripts/validate_security_candidates.py
scripts/triage_security.py
scripts/render_report.py
```

| Componente | Método | Limite |
|---|---|---|
| `collect_api_context.py` | Ler diff, contratos de API, inventário, requisitos e relações disponíveis; produzir referências verificáveis. | Não inferir configuração de produção somente pelo código. |
| `normalize_external_checks.py` | Adaptadores por formato preservam regra, resultado, origem, escopo e referência da mudança. | Zero achados não equivale a scan concluído ou cobertura suficiente. |
| Análise semântica na skill | Gerar candidatos por fluxo e obrigação: ator, recurso, ação, controle e consequência. | Rótulo OWASP e plausibilidade não confirmam vulnerabilidade. |
| `validate_security_candidates.py` | Validar schema, referências, trechos, escopo e evidências externas. | A validação mecânica não prova exploração ou impacto. |
| `triage_security.py` | Selecionar regras, correlacionar, classificar, avaliar obrigações e dependências de Coverage. | Não ocultar incerteza por score ou limite de itens. |
| `render_report.py` | Apresentar grupos, impactos, origens, motivos e lacunas. | Não reinterpretar resultados na renderização. |

## Modelo de candidato

```json
{
  "id": "SEC-001",
  "category": "permission_lifecycle",
  "owasp_refs": ["API1:2023", "API5:2023"],
  "operation_ref": "OP-EXPORT",
  "actor_ref": "ROLE-EXPORTER",
  "resource_ref": "RESOURCE-TENANT-REPORT",
  "requirement_refs": ["REQ-REVOKE-017"],
  "evidence_refs": ["DIFF-QUEUE-04", "CODE-WORKER-07"],
  "expected_control": "validar permissao vigente antes da execucao sensivel",
  "hypothesis": "job pode executar apos revogacao sem revalidar permissao",
  "impact": "exportacao de dados apos perda de autorizacao",
  "confidence": "medium",
  "verification_status": "inconclusive",
  "resolution_status": "open",
  "suggested_severity": "P1",
  "external_check_refs": [],
  "root_cause_ref": "JOB-AUTHORIZATION-BOUNDARY"
}
```

Exemplo hipotético: o requisito pode permitir concluir jobs já autorizados; isso precisa ser verificado. Não presumir que revogação sempre invalida toda operação pendente. A ausência de referências externas no exemplo não comprova que houve scan.

## Regras de triagem

| Regra | Condição | Resultado |
|---|---|---|
| `S-01` | Referência, schema ou escopo inválido | Não confirmar achado; registrar falha e recuperar evidência quando possível. |
| `S-02` | Categoria citada sem cenário, controle esperado ou consequência | Não emitir violação confirmada; investigar relevância. |
| `S-03` | Equivalência com check externo suficiente demonstrada | Referenciar/enriquecer o registro existente, sem duplicar. |
| `S-04` | Controle compensatório aplicável e demonstrado | Reavaliar hipótese e excluir se resolvida, com justificativa. |
| `S-05` | Violação sustentada de controle obrigatório | Classificar impacto e registrar bloqueio conforme política. |
| `S-06` | Impacto potencialmente grave com evidência insuficiente | Investigar; persistindo, registrar análise crítica inconclusiva, sem rebaixar severidade pela confiança. |
| `S-07` | Evidência mandatória ou check externo obrigatório ausente | Registrar não verificado e aplicar política; nunca converter em aprovação. |
| `S-08` | Mesma causa e mesma correção demonstradas | Agrupar preservando atores, fluxos, requisitos e impactos distintos. |
| `S-09` | Informação indispensável para Coverage ausente e não inferível | Registrar dependência concreta e formular pergunta mínima. |
| `S-10` | Pendência não impede Coverage | Avançar com a pendência registrada; sem aprovação por etapa. |

Severidade depende de consequência e alcance: classe dos dados, sujeitos afetados, fronteira de tenant, operação sensível, duração e reversibilidade quando relevantes. Configurar tabelas por categoria; não atribuir P0 por pertencer ao Top 10 nem multiplicar pontuações para ocultar bloqueadores.

## Algoritmo ilustrativo

Pseudocódigo, não executável ou testado. Funções semânticas continuam sujeitas a variação; regras sobre o mesmo conjunto estruturado devem ser reproduzíveis.

```python
def analyze_security(change, intent_result, sources, external_reports, policy):
    context = collect_api_context(change, intent_result, sources)
    checks = normalize_external_checks(external_reports)
    coverage = validate_check_scope_and_freshness(checks, change, policy)
    obligations = identify_security_obligations(context, policy)
    selected = select_applicable_checks(context, obligations, policy)
    responsibility = partition_external_and_contextual(selected, coverage)

    candidates = semantic_security_analysis(
        context, responsibility.contextual_checks, obligations
    )
    findings, pending, exclusions, correlations = [], [], [], []

    for candidate in candidates:
        validation = validate_candidate(candidate, context)
        if not validation.valid:
            pending.append(evidence_recovery_request(candidate, validation))
            continue

        match = correlate_external_evidence(candidate, coverage)
        if match.same_condition_and_sufficient:
            correlations.append(enrich_existing_record(match, candidate))
            continue

        assessment = investigate_controls_and_business_effect(candidate, context)
        if assessment.compensating_control_resolves:
            exclusions.append(exclude(candidate, rule="S-04"))
        elif not assessment.sufficient:
            pending.append(record_uncertainty(candidate, assessment))
        else:
            findings.append(classify_security_by_policy(candidate, assessment, policy))

    obligation_status = assess_obligations(
        obligations, context, coverage, findings, correlations, pending, policy
    )
    groups = consolidate_supported_root_causes(findings, correlations, policy)
    dependencies = identify_coverage_execution_dependencies(
        context, obligation_status, pending
    )
    return build_stage_result(
        findings=groups,
        obligations=obligation_status,
        pending=pending,
        exclusions=exclusions,
        execution_blockers=dependencies,
        can_continue=(len(dependencies) == 0),
        policy=policy,
    )
```

Os resultados externos permanecem no resultado consolidado, inclusive suas falhas obrigatórias. Correlação não resolve automaticamente uma violação e não converte falha externa em aprovação.

## Exemplo de ponta a ponta

Mudança hipotética: exportação de relatório passa a executar em fila. O requisito exige permissão vigente no momento da execução.

1. Selecionar `permission_lifecycle` e `cross_service_trust` pela relação entre identidade, job e acesso a dados.
2. Conferir checks externos e seu escopo. Um scan sem achados de injeção não comprova esse controle de autorização.
3. Coletar requisito, identidade transportada pelo job, uso de tenant, lógica do worker e controles equivalentes.
4. Investigar se a autorização é revalidada em outra camada e se o requisito realmente exige isso.
5. Se uma evidência suficiente demonstrar violação, classificar segundo a consequência e registrar bloqueio de aprovação. Se faltar sustentação, manter inconclusivo/não verificado conforme obrigação.
6. Entregar a Coverage o cenário de teste: job agendado, permissão revogada, execução posterior e resultado esperado segundo o requisito.
7. Continuar sem perguntar ao operador se o comportamento esperado está claro. Se ele estiver indeterminado e for indispensável para Coverage, tentar resolver pelas fontes; somente depois formular a pergunta mínima.

Um cenário de teste sugerido não equivale a teste existente ou executado. Segurança e cobertura usam a mesma evidência compartilhada, mas respondem perguntas diferentes e não geram dois bloqueios independentes para a mesma causa sem justificativa.

## Plano de adoção

1. Inventariar regras, modalidades, escopos e formatos reais de Checkmarx e Sonar; não assumir equivalência entre produtos ou perfis.
2. Mapear categorias internas a fluxos e controles corporativos, com requisitos verificáveis.
3. Definir tabelas de evidência mínima, impacto e dependências entre Security e Coverage.
4. Construir casos representativos e baseline de perguntas, duplicações e decisões.
5. Comparar novo funil com o atual sem alterar decisões de produção inicialmente.
6. Calibrar exclusões e agrupamento revisando falsos positivos e riscos omitidos.
7. Adotar gradualmente o contrato estruturado e eliminar aprovação humana por etapa.

## Validação necessária

- Check externo válido, ausente, antigo, de outra mudança, incompleto ou com erro.
- Mesmo rótulo OWASP com condições distintas; mesma condição com múltiplos rótulos.
- Controle compensatório suficiente e alegação sem evidência.
- Repetição permitida pelo negócio versus repetição que viola invariante sensível.
- Revogação com semântica explícita versus requisito ambíguo.
- Violação confirmada que permite Coverage versus dependência que impede sua execução.
- Evidência mandatória ausente; consequência grave ainda incerta; várias causas independentes.

Medir perguntas por mudança, causas distintas, falsos positivos, violações relevantes omitidas, P0/P1 detectados, sobreposição externa e variação semântica em execuções repetidas. Triagem e ordenação devem ser estáveis para entradas e política idênticas. Não impor limite de achados que elimine bloqueadores.

## Decisões ainda necessárias na implementação real

Versões das referências; controles corporativos selecionados; cobertura comprovada dos scanners; janela de validade dos resultados; critérios de impacto; evidência mínima; formato de inventário; adaptadores; regras de agrupamento e catálogo de dependências de execução. Nenhuma credencial ou acesso ao código corporativo é necessário para manter este planejamento com exemplos sintéticos.
