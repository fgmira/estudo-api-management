# Módulo 5 · Segurança de APIs
## Capítulo 5.8 · Segurança em APIs de parceiros e APIs públicas

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Estratégico e técnico
> **Pré-requisito:** Cap 3.6 · Cap 5.4 · Cap 5.7

---

## Sumário

- [5.8.1 · A ampliação da superfície de ataque além do perímetro](#581--a-ampliação-da-superfície-de-ataque-além-do-perímetro)
- [5.8.2 · Segurança em APIs de parceiros](#582--segurança-em-apis-de-parceiros)
- [5.8.3 · Segurança em APIs públicas](#583--segurança-em-apis-públicas)
- [5.8.4 · Credenciamento e gestão do ciclo de vida de consumidores externos](#584--credenciamento-e-gestão-do-ciclo-de-vida-de-consumidores-externos)
- [5.8.5 · Obrigações regulatórias em mercados específicos](#585--obrigações-regulatórias-em-mercados-específicos)
- [Fontes e referências](#fontes-e-referências)

---

## 5.8.1 · A ampliação da superfície de ataque além do perímetro

APIs internas são consumidas por atores conhecidos em contextos controlados — times de produto da mesma organização, com acesso a repositórios, pipelines e processos de suporte. Quando uma API é exposta a parceiros ou ao público, o modelo de risco muda fundamentalmente.

```mermaid
graph LR
    subgraph interna["APIs internas
    superfície de ataque controlada"]
        direction TB
        i1["Consumidores: times internos
        Identidades: conhecidas e gerenciadas
        Contexto: controlado
        Suporte: direto
        Incidentes: investigáveis"]
    end

    subgraph parceiros["APIs de parceiros
    superfície de ataque expandida"]
        direction TB
        p1["Consumidores: organizações externas
        Identidades: credenciadas via contrato
        Contexto: parcialmente controlado
        Suporte: via SLA contratual
        Incidentes: investigação cruzada"]
    end

    subgraph publicas["APIs públicas
    superfície de ataque máxima"]
        direction TB
        pu1["Consumidores: qualquer um
        Identidades: auto-registradas
        Contexto: não controlado
        Suporte: documentação e portal
        Incidentes: qualquer ator pode ser o atacante"]
    end

    interna -->|"exposição crescente
    risco crescente"| parceiros
    parceiros -->|"exposição máxima
    risco máximo"| publicas

    style interna fill:#e6ffe6,stroke:#009900
    style parceiros fill:#fff9e6,stroke:#cc9900
    style publicas fill:#ffe6e6,stroke:#cc0000
```

---

## 5.8.2 · Segurança em APIs de parceiros

O Cap 3.6 tratou governança de APIs de parceiros em profundidade — acordos, SLAs, credenciamento. Aqui o foco é nas dimensões específicas de segurança dessa relação.

### Modelo de confiança com parceiros

Parceiros não são consumidores internos mas também não são o público geral. Há uma relação contratual que estabelece obrigações mútuas de segurança — o que a organização garante ao parceiro e o que o parceiro precisa garantir para acessar as APIs.

```mermaid
graph TB
    subgraph obrigacoes["Obrigações de segurança — relação com parceiros"]
        direction LR

        subgraph org_garante["A organização garante ao parceiro"]
            og1["TLS 1.3 obrigatório
            confidencialidade em trânsito"]
            og2["Notificação de mudanças
            breaking changes com antecedência
            deprecação comunicada"]
            og3["Notificação de incidentes
            que afetam dados do parceiro
            com SLA contratual"]
            og4["Dados do parceiro isolados
            de dados de outros parceiros"]
        end

        subgraph parceiro_garante["O parceiro garante à organização"]
            pg1["Proteção das credenciais
            client_id e secrets
            não expostos em código público"]
            pg2["Uso dentro do escopo contratado
            sem repasse de credenciais
            sem uso para fins não autorizados"]
            pg3["Notificação de comprometimento
            se credenciais forem vazadas
            processo de resposta definido"]
            pg4["Compliance com políticas
            de dados recebidos via API
            conforme contrato de dados"]
        end
    end

    style org_garante fill:#e6f0ff,stroke:#0044cc
    style parceiro_garante fill:#f0e6ff,stroke:#6600cc
```

### Isolamento de credenciais por parceiro

Cada parceiro recebe credenciais exclusivas — client_id e client_secret ou certificado mTLS próprio. Credenciais compartilhadas entre parceiros são inaceitáveis: um incidente em um parceiro comprometeria todos os outros, e a trilha de auditoria não distinguiria as ações de parceiros diferentes.

### Escopos restritos ao caso de uso contratado

O princípio de least privilege aplicado a parceiros: o parceiro recebe apenas os escopos necessários para o caso de uso contratado — não um escopo amplo "por conveniência". Se o contrato prevê leitura de pedidos, o token do parceiro tem `pedidos:read` — não `api:full_access`.

O CoE mantém o registro de quais escopos cada parceiro tem autorizado (Cap 5.1.6) — auditável, revisável e revogável.

### Monitoramento diferenciado para parceiros

O comportamento esperado de um parceiro é mais previsível do que o de consumidores do público geral — há um contrato que define o caso de uso. Desvios desse padrão são sinais mais claros de problema:

- Volume muito acima do contratado pode indicar uso indevido ou comprometimento
- Acesso a endpoints não previstos no caso de uso contratado é um sinal imediato de investigação
- Horários incomuns para o fuso horário do parceiro merecem atenção

---

## 5.8.3 · Segurança em APIs públicas

APIs públicas têm o modelo de risco mais exigente: qualquer pessoa pode se registrar, qualquer pessoa pode tentar explorar vulnerabilidades, e o volume de tentativas de ataque é ordens de magnitude maior do que em APIs internas ou de parceiros.

### Registro e onboarding seguro de consumidores

O self-service de registro — onde qualquer desenvolvedor pode criar uma conta e obter credenciais — é o padrão para APIs públicas. Mas self-service não significa ausência de controles:

```mermaid
flowchart TD
    registro["Desenvolvedor solicita acesso
    via portal de desenvolvedores"]

    subgraph controles_registro["Controles no registro"]
        email["Verificação de email
        previne registros falsos em massa"]
        captcha["CAPTCHA
        previne automação de registro"]
        rate_reg["Rate limit de registro
        por IP e por domínio de email"]
        sandbox["Acesso inicial ao sandbox
        não a produção diretamente"]
    end

    subgraph controles_producao["Controles para acesso a produção"]
        review["Revisão do caso de uso
        para APIs com dados sensíveis"]
        termos["Aceite dos termos de uso
        com obrigações legais explícitas"]
        tier["Tier de acesso
        free → basic → premium
        com limites progressivos"]
    end

    credenciais["Credenciais de produção
    com escopos e limites
    conforme o tier"]

    registro --> controles_registro
    controles_registro --> controles_producao
    controles_producao --> credenciais

    style controles_registro fill:#e6ffe6,stroke:#009900
    style controles_producao fill:#e6f0ff,stroke:#0044cc
```

### Defesa em profundidade para APIs públicas

APIs públicas exigem todas as camadas de defesa discutidas no módulo — com ênfase especial em proteção contra abuso automatizado:

```mermaid
graph TB
    subgraph camadas["Camadas de defesa — API pública"]
        direction TB

        c1["Camada 1 — Borda / CDN
        DDoS mitigation · geo-blocking
        IP reputation · bot detection"]

        c2["Camada 2 — Gateway
        Rate limiting por consumer e IP
        WAF · CORS · headers de segurança
        Validação de token e schema"]

        c3["Camada 3 — Aplicação
        Autenticação OAuth 2.0 + PKCE
        FGA · verificação de ownership
        Lógica de negócio protegida"]

        c4["Camada 4 — Dados
        Criptografia em repouso
        Mascaramento de dados sensíveis
        Auditoria de acesso"]

        c5["Camada 5 — Observabilidade
        Detecção de anomalias
        Alertas de segurança
        SIEM e correlação"]

        c1 --> c2 --> c3 --> c4
        c5 -.->|"monitora todas as camadas"| c1
        c5 -.-> c2
        c5 -.-> c3
        c5 -.-> c4
    end

    style c1 fill:#ffe6e6,stroke:#cc0000
    style c2 fill:#fff9e6,stroke:#cc9900
    style c3 fill:#e6ffe6,stroke:#009900
    style c4 fill:#e6f0ff,stroke:#0044cc
    style c5 fill:#f0e6ff,stroke:#6600cc
```

### Programa de responsible disclosure

APIs públicas são alvo de pesquisadores de segurança que buscam vulnerabilidades — tanto de forma mal-intencionada quanto de boa-fé. Um programa de responsible disclosure (ou bug bounty) canaliza as descobertas de boa-fé para um processo estruturado, em vez de resultar em divulgação pública sem aviso.

Um programa mínimo inclui: política de disclosure publicada no portal, canal de comunicação seguro para reportar vulnerabilidades, SLA de resposta inicial (geralmente 48-72h), e reconhecimento público ou recompensa para descobertas válidas.

---

## 5.8.4 · Credenciamento e gestão do ciclo de vida de consumidores externos

### O ciclo de vida das credenciais de consumidores externos

```mermaid
stateDiagram-v2
    [*] --> Registro : desenvolvedor cria conta

    Registro --> Sandbox : credenciais de sandbox emitidas
    Sandbox --> EmRevisao : solicita acesso a produção
    EmRevisao --> Ativo : caso de uso aprovado
    EmRevisao --> Rejeitado : caso de uso recusado

    Ativo --> Suspenso : violação de termos detectada
    Ativo --> EmRotacao : renovação periódica de secrets
    EmRotacao --> Ativo : novo secret configurado

    Suspenso --> Ativo : investigação concluída
    Suspenso --> Revogado : violação confirmada

    Ativo --> Revogado : parceiro / app encerrado
    Rejeitado --> [*]
    Revogado --> [*]

    note right of Suspenso
        Suspensão automática:
        — abuso de rate limit
        — padrão de ataque detectado
        — secret potencialmente comprometido
    end note

    note right of EmRotacao
        Rotação periódica:
        — conforme política do CoE
        — recomendado a cada 90-180 dias
        — obrigatório em caso de incidente
    end note
```

### Revogação de acesso — processo e urgência

A capacidade de revogar rapidamente as credenciais de um consumidor externo é tão crítica quanto a de revogar um token. O processo precisa ser documentado e testado:

**Revogação imediata** — para incidentes de segurança confirmados, comprometimento de credenciais ou violações graves de termos. Executada em minutos, com comunicação ao consumidor em paralelo.

**Revogação planejada** — para encerramento de contratos, mudança de parceiro ou desativação de aplicações. Com período de notificação adequado (geralmente 30-90 dias).

**Suspensão temporária** — para investigação de comportamento suspeito sem confirmação de incidente. O acesso é bloqueado enquanto a investigação acontece — se o comportamento for legítimo, a suspensão é revertida.

---

## 5.8.5 · Obrigações regulatórias em mercados específicos

Em mercados regulados, a segurança de APIs não é apenas uma boa prática — é uma obrigação legal com consequências de não-conformidade.

### LGPD e APIs com dados pessoais

A Lei Geral de Proteção de Dados — LGPD — impõe obrigações específicas para APIs que processam dados pessoais:

**Minimização de dados** — APIs não devem coletar ou expor mais dados pessoais do que o necessário para a finalidade declarada. O princípio de exposição mínima do Cap 5.1.2 é também uma obrigação legal sob a LGPD.

**Notificação de incidentes** — a LGPD estabelece prazo de 72 horas para notificação à ANPD em casos de incidentes que possam acarretar risco ou dano relevante aos titulares. O processo de resposta a incidentes do Cap 5.9 deve incluir verificação da obrigação de notificação.

**Direitos dos titulares** — APIs que retornam dados pessoais podem precisar de endpoints específicos para exercício de direitos: acesso, correção, portabilidade, exclusão.

### Open Finance e APIs financeiras

O ecossistema de Open Finance no Brasil exige que APIs financeiras sigam especificações técnicas de segurança definidas pelo Banco Central. O framework FAPI — Financial-grade API — do OpenID Foundation é a base técnica:

```mermaid
graph TB
    subgraph fapi["Financial-grade API (FAPI)
    segurança elevada para APIs financeiras"]
        direction LR

        fapi1["FAPI 1.0 Baseline
        Authorization Code + PKCE
        PAR · JARM"]

        fapi2["FAPI 1.0 Advanced
        mTLS ou DPoP obrigatório
        sender-constrained tokens
        request objects assinados"]

        fapi2_adv["FAPI 2.0
        atualização com melhorias
        de segurança adicionais"]

        fapi1 --> fapi2 --> fapi2_adv
    end

    subgraph obrigacoes_fapi["Obrigações específicas"]
        of1["Certificados digitais ICP-Brasil
        para autenticação de instituições"]
        of2["mTLS obrigatório
        para comunicação entre instituições"]
        of3["Conformidade com especificações
        do Banco Central do Brasil"]
        of4["Testes de conformidade
        antes de produção"]
    end

    fapi --> obrigacoes_fapi

    style fapi fill:#e6f0ff,stroke:#0044cc
    style obrigacoes_fapi fill:#fff9e6,stroke:#cc9900
```

---

## Pontos-chave do capítulo

- APIs de parceiros e APIs públicas ampliam progressivamente a superfície de ataque. O modelo de risco muda fundamentalmente quando consumidores são externos — identidades são auto-registradas ou credenciadas via contrato, contexto não é controlado e qualquer ator pode ser um atacante
- Em APIs de parceiros: cada parceiro recebe credenciais exclusivas, escopos restritos ao caso de uso contratado e monitoramento diferenciado. As obrigações de segurança são mútuas e contratualmente definidas
- Em APIs públicas: defesa em profundidade em cinco camadas — borda/CDN, gateway, aplicação, dados e observabilidade. Registro com verificação de email, CAPTCHA e acesso inicial ao sandbox. Programa de responsible disclosure para pesquisadores de boa-fé
- O ciclo de vida das credenciais de consumidores externos inclui revogação imediata para incidentes, revogação planejada para encerramentos e suspensão temporária para investigação
- LGPD impõe minimização de dados, notificação à ANPD em 72h para incidentes relevantes e suporte a direitos dos titulares como obrigações legais para APIs com dados pessoais
- Open Finance exige conformidade com FAPI — Financial-grade API — incluindo mTLS obrigatório, sender-constrained tokens e certificados ICP-Brasil para APIs financeiras

---

## Fontes e referências

| Fonte | Referência completa |
|---|---|
| **LGPD — Lei nº 13.709/2018** | Brasil. *Lei Geral de Proteção de Dados Pessoais*. Lei nº 13.709, 14 de agosto de 2018. Disponível em: [planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm](http://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm) |
| **FAPI — Financial-grade API** | OpenID Foundation. *Financial-grade API Security Profile*. Disponível em: [openid.net/wg/fapi](https://openid.net/wg/fapi/) |
| **Open Finance Brasil** | Banco Central do Brasil. *Open Finance Brasil*. Disponível em: [openfinancebrasil.org.br](https://openfinancebrasil.org.br/) |

---

## Próximo capítulo

**5.9 · Resposta a incidentes de segurança** — o processo completo de contenção, investigação, recuperação e aprendizado para incidentes de segurança em APIs.

---

*Série: Gerenciamento e Governança de APIs · Módulo 5 · Capítulo 5.8*