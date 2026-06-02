# Módulo 6 · IA e APIs
## Capítulo 6.6 · AI Gateway — o novo plano de controle

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Arquitetural e operacional
> **Pré-requisito:** Cap 5.7 · Cap 6.2 · Cap 6.3 · Cap 6.4

---

## Sumário

- [6.6.1 · Por que o API Gateway tradicional não é suficiente](#661--por-que-o-api-gateway-tradicional-não-é-suficiente)
- [6.6.2 · O que é um AI Gateway](#662--o-que-é-um-ai-gateway)
- [6.6.3 · Os três fluxos agênticos a governar](#663--os-três-fluxos-agênticos-a-governar)
- [6.6.4 · Capacidades específicas do AI Gateway](#664--capacidades-específicas-do-ai-gateway)
- [6.6.5 · API Gateway e AI Gateway — convergência e coexistência](#665--api-gateway-e-ai-gateway--convergência-e-coexistência)
- [6.6.6 · Governança unificada — o modelo de controle](#666--governança-unificada--o-modelo-de-controle)
- [Fontes e referências](#fontes-e-referências)

---

## 6.6.1 · Por que o API Gateway tradicional não é suficiente

O API Gateway do Cap 5.7 foi projetado para um modelo específico de tráfego: requisições HTTP estruturadas com schemas definidos, consumidores com identidades estáveis e comportamento previsível, volume mensurável e padrões estabelecidos.

O tráfego agêntico tem características que o API Gateway tradicional não consegue governar adequadamente:

```mermaid
graph TB
    subgraph lacunas["Lacunas do API Gateway tradicional para tráfego agêntico"]
        direction LR

        l1["Tráfego para LLMs
        não é HTTP REST convencional
        é stream de tokens com
        contagem de tokens e custo variável
        Gateway não entende nem mede"]

        l2["MCP sobre HTTP
        protocolo diferente de REST
        descoberta dinâmica de tools
        contexto persistente entre chamadas
        Gateway não conhece a semântica"]

        l3["A2A — agente para agente
        comunicação entre agentes
        pode não passar pelo Gateway
        visibilidade zero sobre delegação"]

        l4["Guardrails de conteúdo
        inspecionar prompts e respostas
        detectar injeções e dados sensíveis
        Gateway inspeciona estrutura não semântica"]

        l5["Custo de tokens
        cada chamada a LLM tem custo variável
        baseado em tokens consumidos
        Gateway não tem essa dimensão"]
    end

    style l1 fill:#ffe6e6,stroke:#cc0000
    style l2 fill:#ffe6e6,stroke:#cc0000
    style l3 fill:#ffe6e6,stroke:#cc0000
    style l4 fill:#ffe6e6,stroke:#cc0000
    style l5 fill:#ffe6e6,stroke:#cc0000
```

O Gartner Market Guide for AI Gateways, publicado em outubro de 2025, identifica esse gap e projeta que até 2028, 70% dos times de engenharia construindo aplicações multimodelo usarão AI Gateways para melhorar confiabilidade e otimizar custos — ante 25% em 2025.

> *Humphreys, A. et al. Market Guide for AI Gateways. Gartner, outubro 2025. Disponível em: [gartner.com/en/documents/7051698](https://www.gartner.com/en/documents/7051698)*

---

## 6.6.2 · O que é um AI Gateway

O Gartner define AI Gateway como uma tecnologia ou plataforma que **atua como intermediário entre aplicações e vários serviços ou modelos de IA**, provendo um plano de controle centralizado para proteger, governar e observar workloads de IA.

```mermaid
graph TB
    subgraph ai_gw["AI Gateway — plano de controle para tráfego agêntico"]
        direction TB

        subgraph think["THINK — Acesso a modelos"]
            t1["Roteamento inteligente
            entre múltiplos LLMs
            baseado em custo · latência · capability"]
            t2["Controle de tokens
            limites por aplicação · por usuário
            custo rastreado e limitado"]
            t3["Guardrails de prompt
            detecção de PII
            detecção de injeção
            filtros de conteúdo"]
            t4["Cache semântico
            respostas similares reutilizadas
            redução de custo e latência"]
        end

        subgraph act["ACT — Invocação de ferramentas"]
            a1["Governança de MCP Servers
            registro e aprovação
            de servidores MCP em produção"]
            a2["Controle de tool calls
            autorização por tool
            logging de cada invocação"]
            a3["Rate limiting agêntico
            por tarefa · por agente · por usuário
            calibrado para padrões agênticos"]
            a4["Sanitização de outputs
            detecção de exfiltração
            conteúdo suspeito em respostas"]
        end

        subgraph interact["INTERACT — Comunicação A2A"]
            i1["Visibilidade A2A
            registra comunicação entre agentes
            mapeamento de cadeia de delegação"]
            i2["Autorização A2A
            verifica que delegação é válida
            token chain íntegra"]
            i3["Limite de recursão
            profundidade máxima de
            cadeia agente-agente"]
        end
    end

    agentes["🤖 Agentes
    e aplicações de IA"]

    llms["LLMs e modelos
    OpenAI · Anthropic · Google · Azure"]

    apis["APIs de negócio
    via MCP Servers"]

    agentes --> ai_gw
    ai_gw --> llms
    ai_gw --> apis

    style ai_gw fill:#e6f0ff,stroke:#0044cc,stroke-width:2px
    style think fill:#e6ffe6,stroke:#009900
    style act fill:#fff0e6,stroke:#cc6600
    style interact fill:#f0e6ff,stroke:#6600cc
```

---

## 6.6.3 · Os três fluxos agênticos a governar

O framework Think-Act-Interact — usado pela Gravitee e outros players do mercado para descrever os fluxos agênticos — é uma forma útil de organizar o que precisa ser governado:

### Think — Acesso a modelos de linguagem

O fluxo onde agentes consultam LLMs. As dimensões específicas de governança:

**Custo de tokens** — cada chamada a um LLM tem custo variável baseado no número de tokens no prompt e na resposta. Sem controle centralizado, o custo de uma aplicação agêntica pode escalar de forma imprevisível. O AI Gateway mede e limita o consumo de tokens por aplicação, por equipe e por usuário.

**Roteamento entre modelos** — organizações usam múltiplos modelos com diferentes capacidades e custos. Uma tarefa simples pode ser roteada para um modelo menor e mais barato; uma tarefa complexa para um modelo mais capaz. O AI Gateway implementa essa lógica de roteamento de forma centralizada.

**Guardrails de conteúdo** — inspecionar prompts antes de enviá-los ao modelo (detecção de PII, detecção de tentativas de jailbreak) e inspecionar respostas antes de entregá-las ao agente (detecção de alucinações, dados sensíveis, conteúdo inadequado).

### Act — Invocação de ferramentas via MCP

O fluxo onde agentes chamam tools MCP — que por baixo chamam APIs de negócio. As dimensões específicas:

**Registro e aprovação de MCP Servers** — análogo ao catálogo de APIs para consumidores humanos. Cada MCP Server que agentes podem acessar deve estar registrado, ter um owner responsável e ter passado por revisão de segurança. MCP Servers não registrados são bloqueados.

**Logging granular de tool calls** — cada chamada a uma tool MCP é registrada com: qual agente chamou, qual tool foi invocada, com quais parâmetros, qual foi o resultado, e qual é a cadeia de delegação completa.

**Rate limiting calibrado para agentes** — diferente de rate limiting para humanos, o rate limiting para agentes precisa considerar o contexto da tarefa. Um agente processando um relatório complexo legitimamente precisa de volume muito maior do que um humano fazendo consultas manuais.

### Interact — Comunicação A2A

O fluxo onde agentes se comunicam com outros agentes. O menos maduro dos três — o protocolo A2A ainda está sendo adotado — mas com as dimensões de governança mais críticas:

**Visibilidade** — apenas 24,4% das organizações têm visibilidade completa sobre comunicação A2A, segundo o relatório State of AI Agent Security. O AI Gateway é o ponto onde essa visibilidade pode ser centralizada.

**Integridade da cadeia de delegação** — verificar que os tokens trocados entre agentes são válidos, que a cadeia `act` é íntegra, e que nenhum agente está assumindo permissões que não foram delegadas.

**Controle de recursão** — agentes que chamam agentes que chamam agentes podem criar loops involuntários ou cadeias de profundidade excessiva que consomem recursos sem limites. O AI Gateway pode impor um limite máximo de profundidade.

---

## 6.6.4 · Capacidades específicas do AI Gateway

```mermaid
graph LR
    subgraph cap_especificas["Capacidades específicas do AI Gateway
    não presentes no API Gateway tradicional"]
        direction TB

        subgraph observ["Observabilidade de IA"]
            o1["Token accounting
            prompt tokens · completion tokens
            custo por chamada · por usuário · por modelo"]
            o2["Latency analytics
            time to first token · total latency
            específico para streaming de LLM"]
            o3["Model usage dashboard
            qual modelo · qual versão
            distribuição de uso"]
        end

        subgraph guardrails["Guardrails"]
            g1["PII detection
            números de cartão · CPF · email
            removidos antes de enviar ao LLM"]
            g2["Prompt injection detection
            padrões conhecidos de injeção
            instrução de override · jailbreak"]
            g3["Output filtering
            dados sensíveis em respostas
            conteúdo não permitido"]
        end

        subgraph mcp_gov["Governança MCP"]
            m1["MCP Server registry
            inventário de servidores aprovados
            equivalente ao catálogo de APIs"]
            m2["Tool call authorization
            política por tool · por agente
            confirmação humana para alto risco"]
            m3["MCP traffic inspection
            conteúdo de tool calls logado
            para investigação forense"]
        end

        subgraph cost["Gestão de custo"]
            c1["Budget limits
            por aplicação · por equipe
            alerta e bloqueio ao atingir limite"]
            c2["Semantic caching
            respostas similares reutilizadas
            redução de 30-60% de custos típica"]
            c3["Model routing por custo
            tarefa simples → modelo barato
            tarefa complexa → modelo capaz"]
        end
    end

    style observ fill:#e6f0ff,stroke:#0044cc
    style guardrails fill:#ffe6e6,stroke:#cc0000
    style mcp_gov fill:#fff0e6,stroke:#cc6600
    style cost fill:#e6ffe6,stroke:#009900
```

---

## 6.6.5 · API Gateway e AI Gateway — convergência e coexistência

O Gartner nota uma convergência rápida: fornecedores tradicionais de API management estão adicionando extensões específicas para IA, enquanto plataformas de IA nativas estão construindo capacidades de gateway.

```mermaid
graph TB
    subgraph convergencia["Convergência — API Gateway e AI Gateway"]
        direction LR

        subgraph api_gw["API Gateway — evoluindo"]
            ag1["Capacidades originais:
            auth · rate limiting · routing
            WAF · TLS · logging"]

            ag2["Extensões de IA sendo adicionadas:
            MCP traffic support
            token-aware rate limiting
            basic guardrails
            AI agent identity"]
        end

        subgraph ai_gw_ev["AI Gateway — madurecendo"]
            ai1["Capacidades nativas:
            LLM routing · token accounting
            guardrails · MCP governance
            semantic caching"]

            ai2["Convergindo para API governance:
            full HTTP API support
            integration com API catalogs
            unified identity model"]
        end

        convergido["Plataforma convergida
        governa todo o tráfego:
        APIs REST · GraphQL · gRPC
        MCP · A2A · LLM calls
        identidade unificada
        observabilidade consolidada"]

        api_gw --> convergido
        ai_gw_ev --> convergido
    end

    style convergencia fill:#e6f0ff,stroke:#0044cc
    style convergido fill:#e6ffe6,stroke:#009900
```

Para organizações que já têm um API Gateway maduro, a decisão prática é:

**Opção 1 — Estender o API Gateway existente** com plugins ou extensões de IA. Menor disrupção, reutiliza políticas e identidade existentes. Capacidades mais limitadas no curto prazo.

**Opção 2 — Adicionar um AI Gateway dedicado** em camada separada. Mais capacidade imediata para tráfego agêntico. Requer gestão de duas plataformas e integração entre elas.

**Opção 3 — Aguardar a convergência** usando o API Gateway existente com extensões mínimas até que a convergência do mercado produza uma plataforma unificada madura. Menor risco de aposta em tecnologia que pode ser absorvida.

A escolha depende da velocidade de adoção agêntica na organização — quanto mais rápida a adoção, mais urgente a necessidade de capacidades dedicadas.

---

## 6.6.6 · Governança unificada — o modelo de controle

A visão de destino — independente de como se chega lá — é uma governança unificada que aplica os mesmos princípios a todo o tráfego: APIs REST chamadas por humanos, APIs chamadas por agentes via MCP, chamadas A2A entre agentes e chamadas de agentes a LLMs.

```mermaid
graph TB
    subgraph modelo_unificado["Modelo de governança unificada"]
        direction TB

        subgraph identidade["Identidade unificada"]
            id1["Humanos · serviços · agentes
            o mesmo modelo de identidade
            OAuth 2.0 + workload identity
            Token Exchange para delegação"]
        end

        subgraph politica["Política como código"]
            pol1["Mesmas ferramentas de política
            (OPA / Cedar / OpenFGA)
            para todos os tipos de tráfego
            humano · agente · LLM"]
        end

        subgraph observ_u["Observabilidade consolidada"]
            ob1["Um painel
            API metrics + AI metrics
            token costs + latency + errors
            agent activity + delegation chains"]
        end

        subgraph auditoria_u["Auditoria unificada"]
            au1["Um log de auditoria
            humano fez X · agente fez Y
            em nome de Z · via W
            rastreabilidade completa"]
        end

        subgraph catalog_u["Catálogo unificado"]
            ca1["APIs REST + MCP Servers
            mesmo processo de aprovação
            mesmo ciclo de vida
            mesma descoberta"]
        end

        identidade --- politica
        politica --- observ_u
        observ_u --- auditoria_u
        auditoria_u --- catalog_u
    end

    style identidade fill:#e6f0ff,stroke:#0044cc
    style politica fill:#fff0e6,stroke:#cc6600
    style observ_u fill:#e6ffe6,stroke:#009900
    style auditoria_u fill:#f0e6ff,stroke:#6600cc
    style catalog_u fill:#fff9e6,stroke:#cc9900
```

O CoE que governa APIs REST é o mesmo CoE que governa MCP Servers e AI Gateways. Os princípios de governança — catálogo como fonte de verdade, policy as code, auditoria com rastreabilidade, least privilege — são os mesmos. O que muda são as ferramentas e as dimensões específicas a monitorar.

---

## Pontos-chave do capítulo

- O API Gateway tradicional não é suficiente para tráfego agêntico: não entende tokens de LLM, não governa MCP semanticamente, não tem visibilidade A2A e não inspeciona conteúdo de prompts e respostas
- O Gartner projeta que 70% dos times de engenharia usarão AI Gateways até 2028. AI Gateway é intermediário entre aplicações e serviços de IA, provendo controle centralizado de segurança, governança e observabilidade
- Três fluxos a governar: Think (acesso a modelos — custo, roteamento, guardrails), Act (invocação de tools MCP — registro, logging, rate limiting agêntico), Interact (A2A — visibilidade, integridade de delegação, controle de recursão)
- Capacidades específicas do AI Gateway: token accounting, semantic caching, PII detection, prompt injection detection, MCP Server registry, budget limits por aplicação
- API Gateway e AI Gateway estão convergindo. Três opções práticas: estender o existente, adicionar dedicado, ou aguardar convergência — dependendo da velocidade de adoção agêntica
- A visão de destino é governança unificada: mesma identidade, mesma política, mesma observabilidade e mesmo catálogo para humanos, agentes e LLMs

---

## Fontes e referências

| Fonte | Referência completa |
|---|---|
| **Gartner Market Guide for AI Gateways (2025)** | Humphreys, A. et al. *Market Guide for AI Gateways*. Gartner, outubro 2025. Disponível em: [gartner.com/en/documents/7051698](https://www.gartner.com/en/documents/7051698) |
| **State of AI Agent Security (2025)** | Gravitee. *State of AI Agent Security Report*. 2025. Disponível em: [gravitee.io/state-of-ai-agent-security](https://www.gravitee.io/state-of-ai-agent-security) |

---

## Próximo capítulo

**6.7 · Governança de APIs na era agêntica — tensões e equilíbrios** — onde a governança tradicional atrapalha, onde se torna mais crítica, e como o CoE precisa evoluir.

---

*Série: Gerenciamento e Governança de APIs · Módulo 6 · Capítulo 6.6*