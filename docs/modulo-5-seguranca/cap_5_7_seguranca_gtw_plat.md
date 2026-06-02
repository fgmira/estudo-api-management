# Módulo 5 · Segurança de APIs
## Capítulo 5.7 · Segurança no gateway e na plataforma

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Técnico e operacional
> **Pré-requisito:** Cap 5.2 · Cap 5.4 · Cap 5.5

---

## Sumário

- [5.7.1 · O gateway como plano de controle de segurança](#571--o-gateway-como-plano-de-controle-de-segurança)
- [5.7.2 · Controles de autenticação e autorização no gateway](#572--controles-de-autenticação-e-autorização-no-gateway)
- [5.7.3 · Proteção contra abuso — rate limiting e throttling avançados](#573--proteção-contra-abuso--rate-limiting-e-throttling-avançados)
- [5.7.4 · Validação e filtragem de tráfego](#574--validação-e-filtragem-de-tráfego)
- [5.7.5 · TLS, certificados e gestão de segredos na plataforma](#575--tls-certificados-e-gestão-de-segredos-na-plataforma)
- [5.7.6 · O que o gateway não pode fazer — e o que a aplicação precisa resolver](#576--o-que-o-gateway-não-pode-fazer--e-o-que-a-aplicação-precisa-resolver)
- [Fontes e referências](#fontes-e-referências)

---

## 5.7.1 · O gateway como plano de controle de segurança

O gateway de APIs é o ponto de entrada centralizado do portfólio — e por isso é o lugar natural para aplicar controles de segurança que valem para todas as APIs, sem que cada time de produto precise reimplementá-los.

Essa centralização tem valor real: políticas de autenticação, rate limiting, validação de schema e WAF configurados uma vez no gateway se aplicam a todas as APIs que passam por ele. A alternativa — cada API implementando cada controle individualmente — é inconsistente por natureza.

```mermaid
graph TB
    subgraph externos["Consumidores externos"]
        ext1["📱 App mobile"]
        ext2["🌐 Aplicação web"]
        ext3["🤝 Parceiro B2B"]
        ext4["🤖 Sistema externo"]
    end

    subgraph gateway_layer["API Gateway — plano de controle de segurança"]
        direction TB

        subgraph authn_layer["Autenticação e autorização"]
            token_val["Validação de tokens JWT
            verificação de assinatura · exp · iss · aud"]
            oauth_proxy["OAuth 2.0 proxy
            introspection para tokens opacos"]
            api_key["Gestão de API Keys
            para consumidores legados"]
        end

        subgraph protection["Proteção contra abuso"]
            rl["Rate limiting
            por consumer · por IP · por operação"]
            waf_gw["WAF
            OWASP Core Rule Set
            injeção · scanning · exploits"]
            circuit["Circuit breaker
            proteção de backends"]
        end

        subgraph validation["Validação de tráfego"]
            schema_val["Validação de schema
            request · response"]
            tls_term["TLS termination
            TLS 1.3 obrigatório"]
            cors["CORS
            origens permitidas"]
        end

        subgraph observ["Observabilidade de segurança"]
            log_gw["Logging centralizado
            toda requisição com correlation ID"]
            metrics_gw["Métricas de segurança
            4xx · 5xx · rate limit violations"]
        end
    end

    subgraph backends["Backends / Microserviços"]
        api1["API Pedidos"]
        api2["API Pagamentos"]
        api3["API Identidade"]
    end

    ext1 & ext2 & ext3 & ext4 --> gateway_layer
    authn_layer --> protection --> validation
    validation --> api1 & api2 & api3
    observ -.-> log_gw

    style gateway_layer fill:#e6f0ff,stroke:#0044cc,stroke-width:2px
    style authn_layer fill:#e6ffe6,stroke:#009900
    style protection fill:#ffe6e6,stroke:#cc0000
    style validation fill:#fff9e6,stroke:#cc9900
    style observ fill:#f0e6ff,stroke:#6600cc
```

---

## 5.7.2 · Controles de autenticação e autorização no gateway

### Validação de tokens JWT no gateway

O gateway pode validar tokens JWT localmente usando as chaves públicas do Authorization Server — disponíveis no JWKS endpoint. Essa validação acontece antes que a requisição chegue ao backend, eliminando requisições com tokens inválidos sem consumir recursos dos serviços.

A validação no gateway verifica: assinatura, expiração (`exp`), issuer (`iss`) e audiência (`aud`). O que o gateway geralmente não pode verificar — e que pertence ao backend — é o escopo no nível de operação específica e a autorização por objeto (FGA).

```mermaid
sequenceDiagram
    participant C as Client
    participant GW as Gateway
    participant JWKS as JWKS Endpoint
    participant API as API Backend

    C->>GW: requisição com Bearer token

    GW->>GW: decodifica JWT header
    GW->>JWKS: busca chave pública (kid)
    Note over GW,JWKS: cache com TTL — não busca a cada requisição
    JWKS->>GW: chave pública

    GW->>GW: verifica assinatura
    GW->>GW: verifica exp · iss · aud

    alt Token inválido
        GW->>C: 401 Unauthorized
        Note over GW: requisição não chega ao backend
    else Token válido
        GW->>API: requisição com headers enriquecidos
        Note over GW,API: X-User-Id: sub do token
        Note over GW,API: X-Scopes: escopos do token
        Note over GW,API: X-Correlation-Id: gerado pelo gateway
        API->>GW: resposta
        GW->>C: resposta
    end
```

### Propagação de claims para os backends

O gateway enriquece as requisições com informações extraídas do token antes de encaminhar ao backend — eliminando a necessidade de cada backend validar e decodificar o token de forma independente.

Esse padrão de propagação de claims é conveniente mas tem um risco: se o backend confia cegamente nos headers propagados pelo gateway, um atacante que acessa o backend diretamente (contornando o gateway) pode forjar esses headers.

A mitigação: backends devem ser acessíveis apenas pelo gateway — via regras de rede, service mesh ou validação de certificado mTLS. Um backend que pode ser acessado diretamente sem passar pelo gateway é uma vulnerabilidade de design.

### Suporte a múltiplos provedores de identidade

Em portfólios com múltiplos Authorization Servers — Entra ID para usuários corporativos, Cognito para usuários externos, Keycloak para aplicações legadas — o gateway pode suportar múltiplos JWKS endpoints e validar tokens de diferentes issuers conforme configuração por rota.

---

## 5.7.3 · Proteção contra abuso — rate limiting e throttling avançados

### Granularidade de rate limiting

O Cap 5.2.2 introduziu rate limiting por consumer, operação e IP. No gateway, a implementação dessas camadas requer uma arquitetura de armazenamento distribuído para que os limites se apliquem corretamente em ambientes com múltiplas instâncias do gateway.

```mermaid
graph TB
    subgraph rl_arch["Arquitetura de rate limiting distribuído"]
        direction TB

        subgraph instancias["Instâncias do gateway"]
            gw1["Gateway
            instância 1"]
            gw2["Gateway
            instância 2"]
            gw3["Gateway
            instância 3"]
        end

        subgraph contadores["Contadores distribuídos"]
            redis["Redis / Memcached
            contadores compartilhados
            por consumer · por IP · por rota"]
        end

        subgraph politicas["Políticas de rate limit"]
            p1["Consumer A: 1000 req/min"]
            p2["Consumer B: 100 req/min (free tier)"]
            p3["POST /auth: 5 req/min por IP"]
            p4["GET /relatorios: 10 req/hora por consumer"]
        end

        gw1 & gw2 & gw3 <-->|"lê e incrementa
        contadores atômicos"| redis
        politicas --> redis
    end

    style rl_arch fill:#e6f0ff,stroke:#0044cc
    style contadores fill:#fff0e6,stroke:#cc6600
    style politicas fill:#e6ffe6,stroke:#009900
```

### Rate limiting adaptativo

Rate limiting estático — N requisições por minuto — é eficaz contra abuso de volume mas não detecta ataques que ficam abaixo dos thresholds. Rate limiting adaptativo ajusta os limites dinamicamente com base no comportamento observado:

- Um consumidor que nunca excedeu 100 req/min e de repente está em 900 req/min recebe throttling progressivo mesmo que ainda esteja abaixo do limite absoluto
- Um IP que fez 50 tentativas de autenticação falhadas em 5 minutos recebe rate limiting mais restritivo mesmo que esteja abaixo do limite por requisição

### Headers de rate limit na resposta

O gateway deve comunicar o status de rate limit ao consumidor via headers padronizados, permitindo que consumidores bem implementados se ajustem antes de atingir o limite:

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 247
X-RateLimit-Reset: 1700000060
Retry-After: 60  # incluído apenas quando 429 é retornado
```

---

## 5.7.4 · Validação e filtragem de tráfego

### Validação de schema no gateway

O gateway pode validar requests contra a spec OpenAPI antes de encaminhá-los ao backend. Requisições com payloads que violam o schema — campos faltando, tipos errados, campos extras quando `additionalProperties: false` — são rejeitadas no gateway com 400, sem consumir recursos do backend.

Essa validação é uma camada adicional de proteção — não substitui a validação na aplicação. Um atacante que controla a rede pode tentar contornar o gateway. A aplicação sempre valida o input que recebe, independente de onde veio.

### WAF no gateway

O WAF configurado no gateway — com o OWASP Core Rule Set como baseline — filtra padrões de ataque conhecidos antes que cheguem ao backend. O Cap 5.2.2 documentou as limitações estruturais do WAF para APIs: não detecta IDOR, mass assignment nem lógica de negócio abusada. No gateway, o WAF cobre o que consegue — injeção, scanning, exploits de CVEs conhecidos — sem a ilusão de que é suficiente sozinho.

### Headers de segurança na resposta

O gateway adiciona headers de segurança a todas as respostas — sem que cada backend precise fazê-lo individualmente:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Cache-Control: no-store
Content-Security-Policy: default-src 'none'
```

### CORS centralizado

Configuração de CORS no gateway — com allowlist explícita de origens — aplicada uniformemente a todas as APIs. Sem configuração de CORS no gateway, cada backend precisaria configurá-lo individualmente, com risco de `Access-Control-Allow-Origin: *` em APIs que retornam dados sensíveis.

---

## 5.7.5 · TLS, certificados e gestão de segredos na plataforma

### TLS termination e re-encryption

```mermaid
graph LR
    client["Cliente
    TLS 1.3"]

    subgraph gateway_tls["Gateway"]
        term["TLS Termination
        descriptografa
        inspeciona
        aplica políticas"]
        reencrypt["TLS Re-encryption
        nova conexão TLS
        para o backend"]
    end

    subgraph backend["Backend"]
        api["API
        recebe TLS do gateway
        não do cliente diretamente"]
    end

    client -->|"TLS 1.3
    certificado público"| term
    term --> reencrypt
    reencrypt -->|"TLS interno
    certificado interno / mTLS"| api

    style gateway_tls fill:#e6f0ff,stroke:#0044cc
```

O gateway termina a conexão TLS do cliente — descriptografa o tráfego para inspecionar e aplicar políticas — e estabelece uma nova conexão TLS para o backend. Essa arquitetura (TLS termination + re-encryption) é o padrão para gateways que precisam inspecionar tráfego.

A alternativa — TLS passthrough — não permite inspeção de tráfego no gateway mas garante criptografia end-to-end até o backend.

### Gestão de certificados na plataforma

Certificados TLS expirados são uma das causas mais comuns de indisponibilidade e de vulnerabilidades de segurança em APIs. A plataforma precisa de automação de gestão de certificados:

```mermaid
sequenceDiagram
    participant Monitor as Certificate Monitor
    participant CA as Certificate Authority
    participant Gateway as Gateway / Load Balancer
    participant Ops as Ops Team

    loop A cada 24h
        Monitor->>Gateway: verifica expiração dos certificados
        Gateway->>Monitor: lista certificados com datas de expiração
    end

    alt Certificado expira em < 30 dias
        Monitor->>CA: solicita renovação automática (ACME / Let's Encrypt)
        CA->>Gateway: novo certificado instalado
        Monitor->>Ops: notifica renovação bem-sucedida
    else Certificado expira em < 7 dias e renovação falhou
        Monitor->>Ops: alerta urgente — renovação manual necessária
    else Certificado expirado
        Monitor->>Ops: alerta crítico · incidente gerado
    end
```

### Secrets da plataforma

Credenciais da plataforma — client secrets de integrações com Authorization Servers, credenciais de banco de dados, API keys de serviços externos — devem ser gerenciadas via vault centralizado, nunca em arquivos de configuração ou variáveis de ambiente em texto plano.

O NIST SP 800-57 (Cap 5.2.2) define os princípios: ciclo de vida com rotação periódica, least privilege, e revogação imediata em caso de comprometimento.

---

## 5.7.6 · O que o gateway não pode fazer — e o que a aplicação precisa resolver

A centralização no gateway tem um limite fundamental: o gateway não conhece a semântica de negócio das APIs que protege. Há controles que só podem existir na aplicação:

```mermaid
graph TB
    subgraph gateway_can["✅ O que o gateway resolve"]
        gc1["Autenticação de token
        assinatura · exp · iss · aud"]
        gc2["Rate limiting
        por consumer · IP · operação"]
        gc3["WAF
        padrões de ataque conhecidos"]
        gc4["Validação de schema
        formato · tipos · campos extras"]
        gc5["TLS e headers de segurança
        para todas as APIs"]
        gc6["Logging centralizado
        toda requisição"]
    end

    subgraph app_must["❌ O que a aplicação precisa resolver"]
        am1["Autorização por objeto (BOLA/IDOR)
        pedido pertence a este usuário?"]
        am2["Autorização fina (FGA)
        este usuário pode esta operação
        neste contexto específico?"]
        am3["Lógica de negócio abusada
        cupom válido para este usuário?
        limite de cancelamentos atingido?"]
        am4["Injeção semântica
        o payload é válido para
        o estado atual do objeto?"]
        am5["Validação de estado
        o pedido pode ser cancelado
        neste momento?"]
    end

    subgraph design_must["🔐 O que o design resolve (Cap 5.1)"]
        dm1["Exposição mínima de dados
        campos necessários apenas"]
        dm2["Identificadores não-previsíveis
        UUIDs vs IDs sequenciais"]
        dm3["Escopos granulares
        least privilege por operação"]
    end

    style gateway_can fill:#e6ffe6,stroke:#009900
    style app_must fill:#ffe6e6,stroke:#cc0000
    style design_must fill:#e6f0ff,stroke:#0044cc
```

Essa divisão de responsabilidades é o argumento central do Módulo 5 como um todo: segurança robusta de APIs emerge da combinação de design seguro (Cap 5.1), controles na plataforma (Cap 5.7) e lógica de autorização na aplicação (Cap 5.4.12) — não de nenhum dos três isoladamente.

---

## Pontos-chave do capítulo

- O gateway é o plano de controle de segurança centralizado — autenticação de tokens, rate limiting, WAF, validação de schema, TLS e headers de segurança aplicados a todas as APIs sem que cada time precise reimplementar
- A validação JWT no gateway elimina requisições inválidas antes que consumam recursos dos backends. Os claims validados no gateway — assinatura, exp, iss, aud — não incluem FGA, que pertence à aplicação
- Backends devem ser acessíveis apenas via gateway. Um backend acessível diretamente invalida todos os controles de segurança do gateway
- Rate limiting distribuído exige armazenamento compartilhado (Redis) para que os limites se apliquem corretamente em múltiplas instâncias do gateway. Rate limiting adaptativo complementa o estático para ataques abaixo dos thresholds
- Gestão automatizada de certificados — renovação proativa antes da expiração — previne a causa mais evitável de incidentes de disponibilidade e vulnerabilidade
- O gateway não conhece a semântica de negócio: BOLA, FGA e lógica de negócio abusada são responsabilidade da aplicação. Segurança robusta combina gateway, aplicação e design — não qualquer um deles isoladamente

---

## Fontes e referências

| Fonte | Referência completa |
|---|---|
| **OWASP Core Rule Set** | OWASP Foundation. *OWASP ModSecurity Core Rule Set*. Disponível em: [coreruleset.org](https://coreruleset.org/) |
| **NIST SP 800-57 Pt1 Rev.5 (2020)** | Barker, E. *Recommendation for Key Management*. NIST SP 800-57 Part 1 Revision 5. Disponível em: [csrc.nist.gov/pubs/sp/800/57/pt1/r5/final](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) |

---

## Próximo capítulo

**5.8 · Segurança em APIs de parceiros e APIs públicas** — as dimensões específicas de segurança para APIs expostas além dos limites organizacionais.

---

*Série: Gerenciamento e Governança de APIs · Módulo 5 · Capítulo 5.7*