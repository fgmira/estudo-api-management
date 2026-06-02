# Módulo 6 — IA & APIs

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Estratégico e técnico
> **Pré-requisito:** Módulos 1 a 5

---

## Por que IA merece um módulo próprio?

Os módulos anteriores construíram um programa de APIs projetado para um mundo específico: APIs são contratos entre sistemas, criados por humanos, consumidos por sistemas determinísticos que seguem integrações planejadas. Esse mundo não desapareceu — mas um novo tipo de consumidor emergiu que viola sistematicamente as premissas sobre as quais esse modelo foi construído.

**Sistemas de IA agênticos** — que percebem contexto, raciocinam sobre objetivos, planejam sequências de ações e executam operações autonomamente — estão consumindo APIs em produção hoje. Ao mesmo tempo, a própria IA se torna infraestrutura exposta via API: LLMs, embeddings, pipelines de geração. APIs são agora tanto *consumidoras* quanto *produtoras* de capacidades de IA.

Este módulo documenta o que muda — e o que permanece válido — quando IA entra no ecossistema de APIs.

---

## Capítulos

### [6.1 · IA e APIs — o novo paradigma](cap_6_1_ia_api_paradgma.md)

O capítulo de abertura do módulo. Documenta a mudança fundamental: o consumidor que não é humano, o que isso implica para o modelo de governança (descoberta autônoma, comportamento não-determinístico, autorização adaptativa) e a dupla face da IA nas APIs — IA como consumidora de APIs e IA como serviço exposto via API. Fecha estabelecendo o que dos módulos anteriores permanece válido e o que precisa ser revisitado.

---

### [6.2 · MCP e os protocolos agênticos](cap_6_2_mcp_protocolos_agenticos.md)

Antes do MCP, conectar sistemas de IA a ferramentas externas exigia uma integração customizada para cada par — um problema N×M. O **Model Context Protocol** resolve isso com um padrão aberto que permite a qualquer agente descobrir e invocar qualquer ferramenta sem código customizado. O capítulo cobre a arquitetura e funcionamento do MCP, como APIs existentes se tornam ferramentas para agentes, o protocolo **Agent-to-Agent (A2A)** para colaboração entre agentes, o conceito de **AI Readiness** do portfólio de APIs e o papel do catálogo como contexto para agentes.

---

### [6.3 · Identidade e autorização de agentes](cap_6_3_identidade_autorizacao_agentes.md)

IAM tradicional foi construído sobre premissas que sistemas agênticos violam: identidade humana estável, sessão com início e fim claros, ação intencional e auditável. O capítulo cobre os modelos de **identidade de agentes**, o problema central da **delegação em cascata** (quando um agente invoca outro em nome de um usuário), **Token Exchange (RFC 8693)** em cadeias agênticas, **least privilege** para agentes e **FGA em contexto agêntico**. Fecha com o padrão de **identidade efêmera** como abordagem recomendada.

---

### [6.4 · Segurança no ecossistema agêntico](cap_6_4_seguranca_agentica.md)

Agentes introduzem vetores de ataque que não existiam no modelo tradicional de APIs. O capítulo cobre o novo landscape de ameaças agênticas, **prompt injection** como vetor de ataque contra APIs, o **OWASP LLM Top 10 2025** e o **OWASP Top 10 para Aplicações Agênticas 2026** aplicados ao contexto de APIs, **tool poisoning** e **shadow agents** como ameaças específicas do ecossistema MCP, casos documentados de exploração e — criticamente — o que o programa de APIs pode fazer na camada de defesa.

---

### [6.5 · IA produzindo APIs — riscos e governança](cap_6_5_ia_produzindo_api.md)

*Vibe coding* tornou-se mainstream — mas IA gerando código de API introduz padrões de vulnerabilidade específicos que o programa de governança precisa endereçar. O capítulo examina a evidência empírica sobre qualidade e segurança de código gerado por IA, os **padrões de vulnerabilidade mais frequentes** em APIs geradas (autenticação fraca, exposição excessiva de dados, validação superficial), os riscos na **geração de specs OpenAPI por IA**, o fenômeno do **slopsquatting** (dependências alucinadas que podem ser sequestradas por atacantes) e como o programa de governança — gates de revisão, SAST, políticas de style guide — precisa se adaptar a esse novo contexto.

---

### [6.6 · AI Gateway — o novo plano de controle](cap_6_6_ai_gateway.md)

O API Gateway tradicional foi projetado para tráfego HTTP estruturado com schemas fixos e consumidores previsíveis — premissas que agentes violam. O capítulo apresenta o **AI Gateway** como plano de controle especializado para tráfego agêntico, cobrindo os três fluxos a governar (agente consumindo APIs, API exposta a agentes, agente-a-agente), as **capacidades específicas** do AI Gateway (inspeção de prompt, detecção de prompt injection, controle de token budget, rastreabilidade de cadeias de chamada) e o modelo de **coexistência e convergência** entre API Gateway e AI Gateway no portfólio.

---

### [6.7 · Governança de APIs na era agêntica — tensões e equilíbrios](cap_6_7_gov_api_era_agentica.md)

O capítulo de fechamento do módulo. Agentes exigem APIs mais abertas, descoberta autônoma e autorização dinâmica — tudo o que a governança tradicional foi construída para restringir. O capítulo mapeia onde a **governança tradicional atrapalha** (gates lentos, contratos rígidos, catálogos estáticos) e onde ela se torna **ainda mais crítica** (rastreabilidade de ações autônomas, controle de escopo, auditabilidade). Cobre como o **CoE precisa evoluir**, os novos objetos de governança que emergem com agentes (tool registries, agent policies, trace stores) e o novo equilíbrio entre agilidade agêntica e controle institucional.
