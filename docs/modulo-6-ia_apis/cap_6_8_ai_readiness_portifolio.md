# Módulo 6 · IA e APIs
## Capítulo 6.8 · AI Readiness do portfólio — avaliação e roadmap

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Estratégico e operacional
> **Pré-requisito:** Cap 6.1 a 6.7

---

## Sumário

- [6.8.1 · O que é AI Readiness de portfólio](#681--o-que-é-ai-readiness-de-portfólio)
- [6.8.2 · As cinco dimensões de avaliação](#682--as-cinco-dimensões-de-avaliação)
- [6.8.3 · O processo de avaliação](#683--o-processo-de-avaliação)
- [6.8.4 · Perfis de API e suas trajetórias](#684--perfis-de-api-e-suas-trajetórias)
- [6.8.5 · O roadmap de AI Readiness do portfólio](#685--o-roadmap-de-ai-readiness-do-portfólio)
- [6.8.6 · Módulo 6 — síntese e conexão com o Módulo 8](#686--módulo-6--síntese-e-conexão-com-o-módulo-8)
- [Fontes e referências](#fontes-e-referências)

---

## 6.8.1 · O que é AI Readiness de portfólio

AI Readiness não é uma propriedade binária — uma API não é simplesmente "pronta" ou "não pronta" para consumo por agentes. É um espectro de adequação ao longo de múltiplas dimensões, avaliado no contexto do tipo de agente que vai consumir a API e do tipo de tarefa que esse agente vai executar.

Uma API pode ser altamente adequada para um agente de leitura que consulta dados para gerar relatórios — e completamente inadequada para um agente de escrita que executa operações críticas de negócio. O mesmo contrato, a mesma implementação, avaliações diferentes dependendo do contexto de uso agêntico.

```mermaid
graph TB
    subgraph spectrum["Espectro de AI Readiness — não é binário"]
        direction LR

        nivel0["Nível 0 — Inacessível para agentes
        sem documentação semântica
        sem schema preciso
        sem autenticação adequada
        não deve ser exposta via MCP"]

        nivel1["Nível 1 — Leitura básica
        documentação básica presente
        autenticação OAuth 2.0
        schema válido
        adequada para agentes de consulta simples"]

        nivel2["Nível 2 — Leitura avançada
        descrições semânticas completas
        erros documentados
        rate limits claros
        adequada para agentes de análise e relatório"]

        nivel3["Nível 3 — Escrita controlada
        FGA implementada
        operações reversíveis separadas
        human-in-the-loop para alto risco
        auditoria com delegação completa
        adequada para agentes de execução supervisionada"]

        nivel4["Nível 4 — Escrita autônoma
        FGA granular por contexto
        idempotência garantida
        todos os edge cases documentados
        monitoramento comportamental ativo
        adequada para agentes de alta autonomia"]

        nivel0 --> nivel1 --> nivel2 --> nivel3 --> nivel4
    end

    style nivel0 fill:#ffcccc,stroke:#cc0000
    style nivel1 fill:#ffe6cc,stroke:#cc6600
    style nivel2 fill:#ffffcc,stroke:#cccc00
    style nivel3 fill:#ccffcc,stroke:#009900
    style nivel4 fill:#cce6ff,stroke:#0066cc
```

---

## 6.8.2 · As cinco dimensões de avaliação

O framework de AI Readiness avalia cada API em cinco dimensões, derivadas dos capítulos anteriores do módulo:

```mermaid
graph TB
    subgraph dimensoes["Cinco dimensões de AI Readiness"]
        direction LR

        subgraph d1["D1 — Documentação Semântica"]
            d1a["O que avalia:
            qualidade das descrições
            para consumo por LLMs"]
            d1b["Critérios:
            • descrições de operação claras e precisas
            • efeitos colaterais documentados
            • operações irreversíveis sinalizadas
            • exemplos de uso presentes
            • erros e edge cases cobertos"]
        end

        subgraph d2["D2 — Contrato"]
            d2a["O que avalia:
            precisão e completude
            do schema OpenAPI"]
            d2b["Critérios:
            • additionalProperties: false nos schemas de entrada
            • tipos e formatos precisos
            • idempotência declarada
            • rate limits documentados
            • paginação em coleções"]
        end

        subgraph d3["D3 — Segurança e Autorização"]
            d3a["O que avalia:
            adequação dos controles
            para consumo agêntico"]
            d3b["Critérios:
            • OAuth 2.0 com escopos granulares
            • FGA implementada por objeto
            • operações destrutivas com confirmação
            • rate limiting por consumidor
            • auditoria com identidade do agente"]
        end

        subgraph d4["D4 — Observabilidade Agêntica"]
            d4a["O que avalia:
            rastreabilidade de chamadas
            feitas por agentes"]
            d4b["Critérios:
            • logging identifica chamadas de agentes
            • cadeia de delegação registrada
            • correlation ID propagado
            • alertas para padrões anômalos
            • métricas separadas por tipo de consumidor"]
        end

        subgraph d5["D5 — Resiliência Agêntica"]
            d5a["O que avalia:
            comportamento sob padrões
            de uso agêntico"]
            d5b["Critérios:
            • retry-after em 429
            • erros estruturados e acionáveis
            • sem side effects em leitura
            • idempotência em escrita
            • degradação graciosa sob volume"]
        end
    end

    style d1 fill:#e6f0ff,stroke:#0044cc
    style d2 fill:#e6ffe6,stroke:#009900
    style d3 fill:#ffe6e6,stroke:#cc0000
    style d4 fill:#f0e6ff,stroke:#6600cc
    style d5 fill:#fff0e6,stroke:#cc6600
```

---

### Matriz de avaliação

Para cada API, cada dimensão recebe uma pontuação de 0 a 3:

| Pontuação | Significado |
|---|---|
| **0** | Ausente ou inadequado — bloqueia uso agêntico |
| **1** | Básico — permite uso agêntico limitado com riscos |
| **2** | Adequado — permite uso agêntico supervisionado |
| **3** | Maduro — permite uso agêntico autônomo |

O score total (0-15) mapeia para o nível de AI Readiness do 6.8.1. Um score de 0-5 é Nível 0-1, 6-9 é Nível 2, 10-12 é Nível 3, 13-15 é Nível 4.

---

## 6.8.3 · O processo de avaliação

A avaliação de AI Readiness do portfólio não precisa acontecer de uma vez — um portfólio grande pode ter centenas de APIs. O processo é incremental e priorizado.

```mermaid
flowchart TD
    subgraph processo["Processo de avaliação de AI Readiness"]

        step1["1. Mapeamento de uso agêntico atual e planejado
        Quais APIs já são usadas por agentes?
        Quais times planejam exposição via MCP?
        Quais APIs têm acesso a dados críticos?"]

        step2["2. Triagem por criticidade
        APIs de alto risco: dados sensíveis · operações críticas
        APIs de médio risco: leitura de dados operacionais
        APIs de baixo risco: dados públicos · não críticos"]

        step3["3. Avaliação das cinco dimensões
        Por API, por tipo de uso agêntico planejado
        Score em cada dimensão
        Lacunas identificadas"]

        step4["4. Classificação no espectro
        Score total → nível de AI Readiness
        Comparado com o nível necessário para
        o uso agêntico planejado"]

        step5["5. Gap analysis e priorização
        Lacunas que bloqueiam uso planejado
        Lacunas que criam risco inaceitável
        Lacunas que podem ser aceitas temporariamente"]

        step6["6. Roadmap de evolução
        Sprints de AI Readiness
        Integrado com roadmap de produto
        Gates de AI Readiness para novas APIs"]

        step1 --> step2 --> step3 --> step4 --> step5 --> step6
    end
```

---

### Automação da avaliação

Parte significativa da avaliação pode ser automatizada:

**D1 — Documentação Semântica** → análise do schema OpenAPI com LLM que avalia qualidade e completude das descrições. Ferramentas como 42Crunch e Spectral com regras customizadas cobrem parte da avaliação.

**D2 — Contrato** → lint automático com Spectral usando regras de AI Readiness — presença de `additionalProperties: false`, rate limits declarados, idempotência marcada.

**D3 — Segurança** → análise estática da spec — securitySchemes presentes, escopos declarados, operações destrutivas sinalizadas.

**D4 e D5** → requerem análise de implementação e comportamento em runtime — parcialmente automatizável via testes, parcialmente manual.

---

## 6.8.4 · Perfis de API e suas trajetórias

Diferentes APIs têm perfis de uso agêntico diferentes, e portanto trajetórias de AI Readiness diferentes. Não faz sentido elevar todas as APIs ao Nível 4 — o custo não justifica para APIs que nunca serão usadas por agentes autônomos.

```mermaid
graph LR
    subgraph perfis["Perfis de API e trajetórias de AI Readiness"]
        direction TB

        subgraph p1["APIs de consulta pública"]
            p1a["Tipo: dados abertos, preços, catálogos"]
            p1b["Agentes: leitura autônoma de baixo risco"]
            p1c["Trajetória: Nível 1 → 2
            foco em documentação semântica
            e resiliência agêntica"]
        end

        subgraph p2["APIs operacionais internas"]
            p2a["Tipo: pedidos, clientes, inventário"]
            p2b["Agentes: análise e relatório supervisionado"]
            p2c["Trajetória: Nível 2 → 3
            foco em FGA e observabilidade agêntica
            human-in-the-loop para escrita"]
        end

        subgraph p3["APIs de execução crítica"]
            p3a["Tipo: pagamentos, contratos, dados sensíveis"]
            p3b["Agentes: execução com aprovação explícita"]
            p3c["Trajetória: Nível 3 → 4
            foco em confirmação humana obrigatória
            auditoria forense completa
            FGA granular por contexto"]
        end

        subgraph p4["APIs administrativas"]
            p4a["Tipo: configuração, usuários, permissões"]
            p4b["Agentes: nunca autônomos — sempre humano no loop"]
            p4c["Trajetória: Nível 0 → 1 apenas para consulta
            escrita nunca exposta via MCP
            acesso de agentes não autorizado"]
        end
    end

    style p1 fill:#e6ffe6,stroke:#009900
    style p2 fill:#e6f0ff,stroke:#0044cc
    style p3 fill:#fff9e6,stroke:#cc9900
    style p4 fill:#ffe6e6,stroke:#cc0000
```

---

## 6.8.5 · O roadmap de AI Readiness do portfólio

O roadmap de AI Readiness não é um projeto separado do programa de APIs — é integrado ao ciclo de vida existente. Cada API que passa por manutenção, revisão ou evolução incorpora melhorias de AI Readiness nas dimensões com maior gap.

```mermaid
gantt
    title Roadmap de AI Readiness — exemplo de portfólio de 50 APIs
    dateFormat YYYY-MM
    axisFormat %b %Y

    section Fase 1 — Visibilidade
    Mapeamento de uso agêntico atual     :done, f1a, 2025-07, 1M
    Avaliação de todas as APIs (score)   :done, f1b, 2025-08, 2M
    Identificação de APIs de alto risco  :done, f1c, 2025-09, 1M

    section Fase 2 — APIs críticas (alto risco)
    FGA em APIs de dados sensíveis       :active, f2a, 2025-10, 3M
    Auditoria com delegação completa     :f2b, 2025-11, 2M
    Human-in-the-loop para operações críticas :f2c, 2026-01, 2M

    section Fase 3 — Documentação semântica
    Templates de descrição para MCP      :f3a, 2026-01, 2M
    Atualização das 30 APIs operacionais :f3b, 2026-02, 3M
    Gates de AI Readiness no pipeline    :f3c, 2026-03, 1M

    section Fase 4 — Maturidade agêntica
    MCP Servers para domínios principais :f4a, 2026-04, 3M
    AI Gateway em produção               :f4b, 2026-05, 2M
    Monitoring comportamental agêntico   :f4c, 2026-06, 2M
```

---

### Gates de AI Readiness como política obrigatória

A partir de um ponto de maturidade do programa, novas APIs que serão expostas via MCP devem atingir um score mínimo de AI Readiness antes da publicação — integrado ao gate de publicação do Cap 4.4:

```mermaid
flowchart LR
    nova_api["Nova API
    planejada para exposição via MCP"]

    avaliacao["Avaliação de AI Readiness
    cinco dimensões · score calculado"]

    nivel_min["Score ≥ mínimo
    para o nível requerido?"]

    publicacao["Publicação autorizada
    incluindo MCP Server"]

    remediation["Plano de remediação
    antes da publicação
    lacunas críticas bloqueiam
    lacunas menores: prazo acordado"]

    nova_api --> avaliacao --> nivel_min
    nivel_min -->|"sim"| publicacao
    nivel_min -->|"não"| remediation
    remediation -->|"lacunas corrigidas"| avaliacao
```

---

## 6.8.6 · Módulo 6 — síntese e conexão com o Módulo 8

O Módulo 6 documentou a transformação mais profunda no ecossistema de APIs desde a consolidação do REST como padrão dominante. Não é uma evolução incremental — é uma mudança de paradigma no perfil do consumidor, nas ameaças, nas ferramentas de governança e nos processos do CoE.

```mermaid
graph TB
    subgraph sintese["Módulo 6 — síntese do argumento"]
        direction TB

        c61["Cap 6.1 · O novo paradigma
        consumidor agêntico: autônomo · não determinístico
        escala imprevisível · delegação em cascata"]

        c62["Cap 6.2 · MCP e protocolos
        protocolo padrão de descoberta e uso de APIs
        AI Readiness como requisito de portfólio"]

        c63["Cap 6.3 · Identidade
        IAM tradicional inadequado
        Token Exchange · escopo efêmero · FGA crítica"]

        c64["Cap 6.4 · Segurança agêntica
        prompt injection · tool poisoning · shadow agents
        OWASP LLM e Agentic Top 10"]

        c65["Cap 6.5 · IA produzindo APIs
        2,74x mais vulnerabilidades
        slopsquatting · paradoxo da confiança"]

        c66["Cap 6.6 · AI Gateway
        Think · Act · Interact
        plano de controle unificado"]

        c67["Cap 6.7 · Tensões e equilíbrios
        onde a governança atrapalha
        onde se torna mais crítica"]

        c68["Cap 6.8 · AI Readiness
        avaliação e roadmap
        cinco dimensões · perfis · gates"]

        c61 --> c62 --> c63 --> c64 --> c65 --> c66 --> c67 --> c68
    end

    modulo8["Módulo 8 · Maturidade
    AI Readiness é uma das dimensões
    do diagnóstico holístico
    de maturidade do programa"]

    c68 -->|"alimenta"| modulo8

    style sintese fill:#e6f0ff,stroke:#0044cc
    style modulo8 fill:#e6ffe6,stroke:#009900
```

O Módulo 6 não fecha o tema de IA e APIs — o campo está evoluindo rápido demais para qualquer fechamento definitivo. O que fecha é o framework de raciocínio: os princípios para navegar o ecossistema agêntico, os riscos documentados empiricamente, as adaptações necessárias na governança, e o processo de avaliação e evolução do portfólio.

O Módulo 8 — Maturidade — retomará AI Readiness como uma das dimensões do diagnóstico holístico de maturidade do programa. Uma organização que implementou os fundamentos dos Módulos 1 a 5 e começou a Fase 1 do roadmap de AI Readiness está em um nível de maturidade diferente de uma que ainda trata IA como experimento. O Módulo 8 terá as ferramentas para fazer esse diagnóstico de forma contextual.

---

## Pontos-chave do capítulo

- AI Readiness não é binária — é um espectro de quatro níveis determinado por cinco dimensões: documentação semântica, contrato, segurança/autorização, observabilidade agêntica e resiliência agêntica
- O processo de avaliação é incremental e priorizado: mapeamento de uso atual e planejado, triagem por criticidade, avaliação das cinco dimensões, gap analysis e roadmap
- Diferentes perfis de API têm trajetórias diferentes: APIs de consulta pública vão ao Nível 2, operacionais ao Nível 3, críticas ao Nível 4, administrativas ficam no Nível 0-1 com escrita nunca exposta via MCP
- O roadmap de AI Readiness é integrado ao ciclo de vida existente — não um projeto separado. Gates de AI Readiness são incorporados ao processo de publicação para novas APIs expostas via MCP
- Parte da avaliação é automatizável via lint e análise estática. D4 e D5 requerem análise de comportamento em runtime
- O Módulo 6 documenta uma mudança de paradigma, não evolução incremental. AI Readiness alimenta o diagnóstico de maturidade do Módulo 8

---

## Fontes e referências

| Fonte | Referência completa |
|---|---|
| **MCP Specification** | Agentic AI Foundation. Disponível em: [modelcontextprotocol.io](https://modelcontextprotocol.io) |
| **NIST AI RMF (2023)** | NIST. *AI Risk Management Framework 1.0*. Disponível em: [doi.org/10.6028/NIST.AI.100-1](https://doi.org/10.6028/NIST.AI.100-1) |
| **Gartner Market Guide for AI Gateways (2025)** | Humphreys, A. et al. Gartner, outubro 2025. Disponível em: [gartner.com/en/documents/7051698](https://www.gartner.com/en/documents/7051698) |

---

> **Módulo 6 · IA e APIs — completo**
>
> Cap 6.1 · IA e APIs — o novo paradigma
> Cap 6.2 · MCP e os protocolos agênticos
> Cap 6.3 · Identidade e autorização de agentes
> Cap 6.4 · Segurança no ecossistema agêntico
> Cap 6.5 · IA produzindo APIs — riscos e governança
> Cap 6.6 · AI Gateway — o novo plano de controle
> Cap 6.7 · Governança de APIs na era agêntica — tensões e equilíbrios
> Cap 6.8 · AI Readiness do portfólio — avaliação e roadmap

---

*Série: Gerenciamento e Governança de APIs · Módulo 6 · Capítulo 6.8*