# Módulo 5 · Segurança de APIs
## Capítulo 5.5 · Zero Trust para APIs

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Arquitetural e estratégico
> **Pré-requisito:** Cap 5.1 · Cap 5.2 · Cap 5.4

---

## Sumário

- [5.5.1 · O modelo de perímetro e por que ele falha para APIs](#551--o-modelo-de-perímetro-e-por-que-ele-falha-para-apis)
- [5.5.2 · Os sete tenets do NIST SP 800-207 aplicados a APIs](#552--os-sete-tenets-do-nist-sp-800-207-aplicados-a-apis)
- [5.5.3 · Identidade como o novo perímetro](#553--identidade-como-o-novo-perímetro)
- [5.5.4 · Verificação contínua — além do token na entrada](#554--verificação-contínua--além-do-token-na-entrada)
- [5.5.5 · Microsegmentação e lateral movement](#555--microsegmentação-e-lateral-movement)
- [5.5.6 · Como Zero Trust conecta os capítulos anteriores](#556--como-zero-trust-conecta-os-capítulos-anteriores)
- [5.5.7 · Maturidade de adoção — Zero Trust não é binário](#557--maturidade-de-adoção--zero-trust-não-é-binário)
- [Fontes e referências](#fontes-e-referências)

---

## 5.5.1 · O modelo de perímetro e por que ele falha para APIs

### O modelo de perímetro

O modelo de segurança dominante durante décadas foi baseado em perímetro de rede. A premissa era simples: defender a fronteira, e tudo dentro dela é confiável. Firewalls, DMZs e VPNs foram projetados para esse modelo — construir um castelo com muros altos e validar quem entra pelo portão.

```mermaid
graph TB
    subgraph internet["🌐 Internet — não confiável"]
        ext["Usuários externos
        Parceiros
        Atacantes"]
    end

    subgraph perimetro["🔒 Perímetro — firewall / VPN"]
        fw["Firewall
        DMZ"]
    end

    subgraph interna["🏰 Rede interna — implicitamente confiável"]
        direction LR
        app1["App A"]
        app2["App B"]
        app3["App C"]
        db1["DB"]
        db2["DB"]
        app1 <-->|"livre"| app2
        app2 <-->|"livre"| app3
        app1 <-->|"livre"| db1
        app3 <-->|"livre"| db2
    end

    ext -->|"bloqueado"| fw
    fw -->|"dentro = confiável"| app1

    style internet fill:#ffcccc,stroke:#cc0000
    style perimetro fill:#ffffcc,stroke:#cccc00
    style interna fill:#ccffcc,stroke:#00cc00
```

Dentro da rede, serviços se comunicavam livremente. Um serviço que passava pelo firewall tinha acesso a praticamente tudo. A confiança era baseada em localização — se você está dentro, você é confiável.

---

### Por que esse modelo falha para APIs

**APIs existem para romper o perímetro.** Uma API pública, por definição, é exposta à internet. Uma API de parceiro cruzando o perímetro via integração B2B. Uma API interna acessada por times remotos via VPN. O modelo de perímetro foi construído para um mundo onde o valor estava contido dentro da rede — e APIs invertem exatamente essa premissa.

**O perímetro já foi comprometido.** A questão não é se um atacante já está dentro da rede — é quando isso aconteceu. Credenciais comprometidas, phishing, supply chain attacks, insiders maliciosos — todos resultam em um ator dentro do perímetro operando como se fosse confiável. Se a rede interna é implicitamente confiável, um ator comprometido tem acesso lateral irrestrito.

**A rede interna não existe mais como entidade coesa.** Microserviços em múltiplos clusters Kubernetes, workloads em clouds diferentes, times remotos acessando ambientes de desenvolvimento — a "rede interna" é um conceito cada vez mais fictício. Tratar como confiável uma rede que não tem fronteiras claras é uma premissa que não se sustenta.

```mermaid
graph LR
    subgraph falhas["Por que o modelo de perímetro falha"]
        direction TB
        f1["❌ APIs públicas
        existem para romper o perímetro
        por design"]

        f2["❌ Perímetro já comprometido
        credenciais vazadas · phishing
        insiders · supply chain"]

        f3["❌ Rede interna fragmentada
        multi-cloud · remote work
        microserviços em múltiplos clusters"]

        f4["❌ Lateral movement irrestrito
        um serviço comprometido
        acessa tudo na rede interna"]
    end

    style f1 fill:#ffcccc,stroke:#cc0000
    style f2 fill:#ffcccc,stroke:#cc0000
    style f3 fill:#ffcccc,stroke:#cc0000
    style f4 fill:#ffcccc,stroke:#cc0000
```

---

### O princípio Zero Trust

O NIST SP 800-207 — publicado em agosto de 2020 — é a referência normativa central para Zero Trust Architecture. O documento define Zero Trust como um conjunto de paradigmas de segurança que move as defesas de amplos perímetros de rede para recursos individuais, com foco em proteger recursos e não segmentos de rede.

> *Rose, S., Borchert, O., Mitchell, S. & Connelly, S. Zero Trust Architecture. NIST SP 800-207, agosto 2020. Disponível em: [doi.org/10.6028/NIST.SP.800-207](https://doi.org/10.6028/NIST.SP.800-207)*

O princípio central: **nunca confiar, sempre verificar.** Nenhuma requisição é implicitamente confiável baseada em localização de rede. Toda requisição é autenticada, autorizada e continuamente validada — independente de onde origina.

```mermaid
graph TB
    subgraph zt["🛡️ Zero Trust — sem perímetro implícito"]
        direction TB

        subgraph identidades["Identidades verificadas"]
            u["👤 Usuário
            autenticado · contexto validado"]
            s["⚙️ Serviço
            certificado mTLS · token"]
            d["💻 Dispositivo
            postura · compliance"]
        end

        subgraph pdp_area["Decisão de acesso — por requisição"]
            pe["Policy Engine
            identidade + postura + contexto
            → decisão dinâmica"]
        end

        subgraph recursos["Recursos protegidos individualmente"]
            api1["API Pedidos
            🔐 verifica cada requisição"]
            api2["API Pagamentos
            🔐 verifica cada requisição"]
            api3["API Identidade
            🔐 verifica cada requisição"]
            db["Banco de Dados
            🔐 acesso por identidade"]
        end

        u -->|"toda requisição verificada"| pe
        s -->|"toda requisição verificada"| pe
        d -->|"postura validada"| pe
        pe -->|"allow"| api1
        pe -->|"allow"| api2
        pe -->|"allow"| api3
        pe -->|"allow"| db
    end

    style zt fill:#e6f3ff,stroke:#0066cc
    style pdp_area fill:#fff3e6,stroke:#cc6600
    style recursos fill:#e6ffe6,stroke:#006600
```

---

## 5.5.2 · Os sete tenets do NIST SP 800-207 aplicados a APIs

O NIST SP 800-207 define sete tenets que formam o núcleo filosófico de uma Zero Trust Architecture. Para cada tenet, a tradução direta para o contexto de APIs:

---

**Tenet 1 — Todos os dados e serviços são recursos**

No contexto de APIs: cada API é um recurso que requer proteção independente — não importa se está na rede interna, em um cloud provider ou exposta publicamente. Uma API de microserviço interno não é menos um recurso do que uma API pública.

*Implicação prática:* o catálogo do Cap 3.5 e o CMDB do Cap 4.3 precisam cobrir todas as APIs — incluindo as de comunicação interna entre serviços. Shadow APIs são um violação direta deste tenet.

---

**Tenet 2 — Toda comunicação é protegida independente de localização de rede**

No contexto de APIs: TLS 1.3 é obrigatório para toda comunicação — incluindo entre serviços internos. A localização na rede interna não é um substituto para criptografia em trânsito.

*Implicação prática:* comunicação entre microserviços sem TLS — comum em redes internas "confiáveis" — viola este tenet. Service mesh com mTLS automático é a implementação arquitetural.

---

**Tenet 3 — Acesso a recursos é concedido por sessão**

No contexto de APIs: tokens de vida curta por requisição ou sessão — não sessões permanentes com acesso irrevogável. Access tokens com expiração de minutos, não horas ou dias.

*Implicação prática:* o modelo de refresh token do Cap 5.4.2 é a implementação — vida curta para access tokens, renovação explícita via refresh token.

---

**Tenet 4 — Acesso é determinado por política dinâmica**

No contexto de APIs: a decisão de acesso considera não apenas o token e o escopo — considera o estado da identidade, do dispositivo, do comportamento e do contexto ambiental. Um token válido de um dispositivo com postura comprometida pode ser negado.

*Implicação prática:* o PDP do Cap 5.4.12 é o componente que implementa política dinâmica. A decisão não é estática — é reavaliada com base em contexto atual.

---

**Tenet 5 — Integridade e postura dos ativos são monitoradas continuamente**

No contexto de APIs: as APIs produzidas e consumidas são ativos cuja postura é monitorada — vulnerabilidades conhecidas, configurações, comportamento anômalo. O monitoramento do Cap 5.2.3 implementa este tenet.

*Implicação prática:* SCA no pipeline verifica dependências com CVEs. Monitoramento comportamental detecta desvios. Reconciliação periódica do portfólio real com o catálogo declarado.

---

**Tenet 6 — Autenticação e autorização são dinâmicas e estritamente aplicadas antes de qualquer acesso**

No contexto de APIs: a sequência completa de validação do Cap 5.4.3 — assinatura, expiração, issuer, audience, scope — é executada a cada requisição. Não há "já validei uma vez, pode passar".

*Implicação prática:* validação no Resource Server a cada requisição. Token introspection para tokens opacos a cada requisição (com cache de TTL curto). Sem confiança residual entre requisições.

---

**Tenet 7 — Coleta de informação e melhoria contínua do estado de segurança**

No contexto de APIs: logs de segurança, métricas de autorização, detecção de anomalias alimentam melhoria contínua da postura. O [Anexo F · SIEM e correlação de eventos de segurança](../anexos/f_siem.md) e o Continual Improvement do Cap 4.6.7 implementam este tenet.

*Implicação prática:* cada incidente de segurança é input para evolução das políticas. O CoE revisa periodicamente a postura Zero Trust do portfólio.

```mermaid
mindmap
  root((Zero Trust
  NIST SP 800-207))
    Tenet 1
      Toda API é um recurso
      Sem exceção por localização
      Catálogo e CMDB completos
    Tenet 2
      TLS obrigatório sempre
      Incluindo comunicação interna
      Service mesh com mTLS
    Tenet 3
      Tokens de vida curta
      Acesso por sessão
      Sem sessões permanentes
    Tenet 4
      Política dinâmica
      Contexto avaliado por requisição
      PDP centralizado
    Tenet 5
      Monitoramento contínuo
      SCA e CVEs
      Postura dos ativos
    Tenet 6
      Verificação a cada requisição
      Sem confiança residual
      Validação completa sempre
    Tenet 7
      Coleta e aprendizado
      Incidentes → políticas
      Melhoria contínua
```

---

## 5.5.3 · Identidade como o novo perímetro

No modelo de perímetro, a localização de rede era o proxy de confiança — estar dentro da rede interna era suficiente. No Zero Trust, **identidade substitui localização** como base de confiança.

Identidade no contexto de Zero Trust para APIs não é apenas o usuário humano — é qualquer entidade que faz uma requisição:

```mermaid
graph TB
    subgraph identidades["Identidades em um programa de APIs Zero Trust"]
        direction LR

        subgraph humanos["Identidades humanas"]
            dev["👤 Desenvolvedor
            autenticado via OIDC
            MFA obrigatório"]
            ops["👤 Operador
            acesso privilegiado
            just-in-time · auditado"]
            user["👤 Usuário final
            autenticado via app
            token delegado OAuth 2.0"]
        end

        subgraph servicos["Identidades de serviço"]
            svc_a["⚙️ Serviço A
            client credentials
            certificado mTLS"]
            svc_b["⚙️ Serviço B
            managed identity
            sem secret armazenado"]
            job["⚙️ Job automatizado
            service account
            escopo mínimo"]
        end

        subgraph dispositivos["Identidades de dispositivo"]
            corp["💻 Dispositivo corporativo
            certificado de device
            MDM enrollado"]
            ci["🔧 Pipeline CI/CD
            OIDC workload identity
            sem secrets estáticos"]
        end
    end

    subgraph verificacao["Verificação de identidade — a cada requisição"]
        strong_auth["Autenticação forte
        token válido · mTLS · certificado"]
        context["Contexto validado
        postura · localização · comportamento"]
        authz["Autorização aplicada
        scope · FGA · política dinâmica"]
    end

    humanos --> verificacao
    servicos --> verificacao
    dispositivos --> verificacao

    style humanos fill:#e6f0ff,stroke:#0044cc
    style servicos fill:#fff0e6,stroke:#cc6600
    style dispositivos fill:#e6ffe6,stroke:#006600
    style verificacao fill:#f0e6ff,stroke:#6600cc
```

---

### O problema das credenciais de longa vida

O maior vetor de comprometimento em arquiteturas de APIs não é a quebra de criptografia — é o roubo de credenciais de longa vida: secrets hardcoded, API keys que nunca expiram, service accounts com permissões acumuladas ao longo de anos.

Zero Trust combate isso com o princípio de **credenciais efêmeras** — identidades que existem apenas pelo tempo necessário:

```mermaid
sequenceDiagram
    participant CI as Pipeline CI/CD
    participant IdP as Identity Provider
    participant AS as Authorization Server
    participant API as API Produção

    Note over CI,API: Abordagem Zero Trust — sem secrets estáticos

    CI->>IdP: apresenta OIDC workload identity token
    Note over CI,IdP: token efêmero emitido pelo CI/CD provider
    Note over CI,IdP: válido por minutos — não armazenado

    IdP->>CI: valida identidade do pipeline
    CI->>AS: troca workload token por access token
    AS->>CI: access token (5 min) com escopo mínimo

    CI->>API: deploy com access token
    API->>CI: deploy autorizado

    Note over CI,API: Token expirou — próximo job obtém novo token
    Note over CI,API: Nenhum secret foi armazenado em nenhum momento
```

---

## 5.5.4 · Verificação contínua — além do token na entrada

A implementação mais comum de Zero Trust em APIs é a verificação do token na entrada — o gateway valida o token e libera a requisição. Isso é necessário mas não suficiente. Zero Trust pressupõe **verificação contínua** durante a sessão.

```mermaid
stateDiagram-v2
    [*] --> Autenticado : token válido na entrada

    Autenticado --> Monitorado : sessão iniciada

    Monitorado --> Suspeito : anomalia detectada
    Monitorado --> Autorizado : comportamento normal

    Suspeito --> StepUp : step-up authentication
    Suspeito --> Revogado : risco alto — revogar imediatamente

    StepUp --> Autorizado : verificação adicional aprovada
    StepUp --> Revogado : verificação falhou

    Autorizado --> Monitorado : continua sendo monitorado

    Revogado --> [*] : acesso encerrado · incidente registrado

    note right of Suspeito
        Sinais de anomalia:
        — volume acima do histórico
        — padrão sequencial de IDs
        — localização geográfica incomum
        — horário atípico para este consumer
    end note

    note right of StepUp
        Step-up authentication:
        — requer re-autenticação
        — ou segundo fator
        — para operações de alto risco
    end note
```

---

### O que aciona verificação adicional

**Mudança de comportamento durante a sessão** — o consumidor que normalmente faz 100 requisições por hora começa a fazer 10.000. O padrão de IDs acessados torna-se sequencial. O horário é incomum.

**Operação de alto risco** — uma operação que excede um threshold de valor, que afeta um objeto crítico ou que é irreversível pode exigir step-up authentication — confirmação adicional do usuário.

**Sinal externo de comprometimento** — o sistema de monitoramento recebeu alerta de que as credenciais deste consumidor aparecem em um dump de dados comprometidos. O acesso é suspenso preventivamente.

---

## 5.5.5 · Microsegmentação e lateral movement

### O problema do lateral movement

Em arquiteturas de microserviços sem Zero Trust, um serviço comprometido tem acesso lateral irrestrito a todos os outros serviços na rede interna. O atacante que comprometeu o `notificacao-service` pode alcançar o `pagamentos-service` — porque ambos estão na "rede interna confiável".

```mermaid
graph TB
    subgraph sem_zt["❌ Sem Zero Trust — lateral movement irrestrito"]
        direction LR
        atacante["🔴 Atacante
        comprometeu
        notificacao-service"]

        subgraph rede_interna["Rede interna — tudo confiável"]
            notif["notificacao-service
            ⚠️ comprometido"]
            pedidos["pedidos-service
            dados de clientes"]
            pagamentos["pagamentos-service
            dados financeiros"]
            auth["auth-service
            credenciais"]
            db_pag["💾 DB Pagamentos"]
            db_usr["💾 DB Usuários"]

            notif -->|"acesso livre"| pedidos
            notif -->|"acesso livre"| pagamentos
            notif -->|"acesso livre"| auth
            pagamentos --> db_pag
            auth --> db_usr
        end

        atacante -.->|"compromete"| notif
    end

    style sem_zt fill:#fff0f0,stroke:#cc0000
    style atacante fill:#ffcccc,stroke:#cc0000
    style notif fill:#ffaaaa,stroke:#cc0000
```

```mermaid
graph TB
    subgraph com_zt["✅ Com Zero Trust — microsegmentação"]
        direction LR
        atacante2["🔴 Atacante
        comprometeu
        notificacao-service"]

        subgraph segmentado["Serviços microsegmentados — identidade verificada"]
            notif2["notificacao-service
            ⚠️ comprometido
            identidade: notif-svc"]

            pedidos2["pedidos-service
            🔐 só aceita de: api-gateway, checkout-svc"]

            pagamentos2["pagamentos-service
            🔐 só aceita de: pedidos-svc, reembolso-svc"]

            auth2["auth-service
            🔐 só aceita de: api-gateway"]

            db_pag2["💾 DB Pagamentos
            🔐 só aceita de: pagamentos-svc"]

            db_usr2["💾 DB Usuários
            🔐 só aceita de: auth-svc, perfil-svc"]

            notif2 -->|"❌ identidade não autorizada"| pedidos2
            notif2 -->|"❌ identidade não autorizada"| pagamentos2
            notif2 -->|"❌ identidade não autorizada"| auth2
        end

        atacante2 -.->|"compromete"| notif2
    end

    style com_zt fill:#f0fff0,stroke:#00cc00
    style atacante2 fill:#ffcccc,stroke:#cc0000
    style notif2 fill:#ffaaaa,stroke:#cc0000
    style pedidos2 fill:#ccffcc,stroke:#00cc00
    style pagamentos2 fill:#ccffcc,stroke:#00cc00
    style auth2 fill:#ccffcc,stroke:#00cc00
```

---

### Service mesh como infraestrutura Zero Trust

O service mesh — Istio, Linkerd, Consul Connect — é a infraestrutura que torna microsegmentação operacional em ambientes Kubernetes. Sem service mesh, implementar mTLS entre todos os pares de serviços é manualmente inviável. Com service mesh, mTLS é automático e transparente para o código da aplicação.

```mermaid
graph LR
    subgraph service_mesh["Service Mesh — infraestrutura Zero Trust para microserviços"]
        direction TB

        subgraph svc_a_pod["Pod: pedidos-service"]
            svc_a_app["📦 App
            pedidos-service"]
            proxy_a["🔷 Sidecar Proxy
            Envoy / Linkerd"]
            svc_a_app <--> proxy_a
        end

        subgraph svc_b_pod["Pod: pagamentos-service"]
            svc_b_app["📦 App
            pagamentos-service"]
            proxy_b["🔷 Sidecar Proxy
            Envoy / Linkerd"]
            svc_b_app <--> proxy_b
        end

        subgraph control_plane["Control Plane"]
            cp["🎛️ Control Plane
            Istio / Consul
            políticas de autorização
            rotação automática de certificados
            telemetria"]
        end

        proxy_a <-->|"mTLS automático
        identidade verificada
        política aplicada"| proxy_b

        cp -->|"distribui certificados
        aplica políticas"| proxy_a
        cp -->|"distribui certificados
        aplica políticas"| proxy_b
    end

    style service_mesh fill:#e6f0ff,stroke:#0044cc
    style control_plane fill:#fff0e6,stroke:#cc6600
    style svc_a_pod fill:#e6ffe6,stroke:#006600
    style svc_b_pod fill:#e6ffe6,stroke:#006600
```

O service mesh implementa os Tenets 2 e 6 do NIST SP 800-207 de forma transparente: toda comunicação entre serviços é protegida via mTLS (Tenet 2), e autenticação e autorização são aplicadas a cada requisição (Tenet 6) — sem que o código da aplicação precise gerenciar certificados ou validar tokens entre serviços.

---

## 5.5.6 · Como Zero Trust conecta os capítulos anteriores

Zero Trust não é uma tecnologia nova a ser implementada do zero. É um **framework que organiza controles já existentes** em uma estratégia coerente. Cada capítulo anterior do Módulo 5 contribui para uma postura Zero Trust:

```mermaid
graph TB
    zt["🛡️ Postura Zero Trust
    para o programa de APIs"]

    subgraph design["Cap 5.1 — Security by Design"]
        d1["Superfícies de ataque
        minimizadas no design"]
        d2["Threat modeling
        por operação"]
    end

    subgraph arsenal["Cap 5.2 — Arsenal de segurança"]
        a1["Controles preventivos
        TLS · WAF · rate limiting"]
        a2["Controles detectivos
        monitoramento comportamental
        auditoria e não-repúdio"]
        a3["Controles corretivos
        revogação · isolamento"]
    end

    subgraph owasp["Cap 5.3 — OWASP Top 10"]
        o1["Vulnerabilidades conhecidas
        mitigadas por design"]
        o2["CVE/CVSS/EPSS
        priorização de remediação"]
    end

    subgraph authn["Cap 5.4 — Autenticação e Autorização"]
        au1["Identidade forte
        OAuth 2.0 · OIDC · mTLS"]
        au2["FGA
        autorização por objeto
        política dinâmica"]
        au3["Token propagation
        identidade preservada
        em microserviços"]
    end

    design -->|"assets protegidos
    individualmente"| zt
    arsenal -->|"defense in depth
    em cada camada"| zt
    owasp -->|"vulnerabilidades
    sistematicamente mitigadas"| zt
    authn -->|"identidade como
    novo perímetro"| zt

    style zt fill:#e6f3ff,stroke:#0066cc,stroke-width:3px
    style design fill:#fff9e6,stroke:#cc9900
    style arsenal fill:#ffe6e6,stroke:#cc0000
    style owasp fill:#e6ffe6,stroke:#009900
    style authn fill:#f0e6ff,stroke:#6600cc
```

---

### O que Zero Trust acrescenta além dos controles individuais

Cada controle dos capítulos anteriores resolve um problema específico. Zero Trust acrescenta três elementos que os controles individuais não fornecem:

**Coerência** — Zero Trust é o princípio organizador que garante que os controles são aplicados de forma consistente em todo o portfólio, não apenas onde alguém lembrou de configurar.

**Postura de assume breach** — Zero Trust presume que o perímetro já foi comprometido. Esse pressuposto muda o design dos controles: em vez de "como evitar que atacantes entrem", a pergunta é "o que um atacante pode fazer se já estiver dentro?". Microsegmentação, FGA e auditoria são as respostas a essa pergunta.

**Verificação contínua** — os controles individuais verificam na entrada. Zero Trust verifica continuamente — durante toda a sessão, para toda requisição.

---

## 5.5.7 · Maturidade de adoção — Zero Trust não é binário

Zero Trust não é um estado que se atinge — é uma direção que se percorre. Nenhuma organização implementa todos os tenets simultaneamente. O NIST SP 800-207A — extensão do SP 800-207 para ambientes cloud-native — reconhece explicitamente que a adoção é incremental.

```mermaid
graph LR
    subgraph nivel0["Nível 0 — Perímetro tradicional"]
        n0["Firewall como fronteira
        Rede interna implicitamente confiável
        Sem verificação de identidade de serviço
        TLS apenas para externo"]
    end

    subgraph nivel1["Nível 1 — Identidade básica"]
        n1["OAuth 2.0 e tokens em todas as APIs
        TLS obrigatório — incluindo interno
        Catálogo e inventário de APIs
        Autenticação MFA para humanos"]
    end

    subgraph nivel2["Nível 2 — Autorização granular"]
        n2["FGA com verificação por objeto
        mTLS em comunicação M2M
        Monitoramento comportamental
        Auditoria com não-repúdio"]
    end

    subgraph nivel3["Nível 3 — Zero Trust maduro"]
        n3["Service mesh com mTLS automático
        Microsegmentação por serviço
        Política dinâmica por requisição
        Credenciais efêmeras em todo o stack
        Verificação contínua de postura
        Assume breach como premissa de design"]
    end

    nivel0 -->|"adoção de
    identidade"| nivel1
    nivel1 -->|"autorização
    granular"| nivel2
    nivel2 -->|"infraestrutura
    Zero Trust"| nivel3

    style nivel0 fill:#ffcccc,stroke:#cc0000
    style nivel1 fill:#fff0cc,stroke:#cc9900
    style nivel2 fill:#ccffcc,stroke:#009900
    style nivel3 fill:#cce6ff,stroke:#0066cc
```

---

### A jornada incremental para um programa de APIs

**De onde partir:** a maioria dos programas de APIs começa no Nível 0 ou entre 0 e 1. O primeiro passo de maior impacto é a adoção consistente de identidade — OAuth 2.0 em todas as APIs, TLS obrigatório inclusive internamente, catálogo completo do portfólio.

**O que desbloqueia cada nível:**

Nível 1 → 2: o modelo de escopos bem definido (Cap 5.1.6), a implementação de FGA (Cap 5.4.12), e o plano de observabilidade ativo (Cap 3.8) são os habilitadores.

Nível 2 → 3: a maturidade operacional do Cap 4.7 (ITIL + SRE + DevOps), o service mapping do Cap 4.3 e a infraestrutura de service mesh são os habilitadores.

**O que o CoE governa nessa jornada:**

```mermaid
graph TB
    subgraph coe_zt["CoE — Governança da jornada Zero Trust"]
        direction LR

        politica["📋 Política Zero Trust
        nível mínimo obrigatório
        por criticidade da API"]

        roadmap["🗺️ Roadmap
        por onde começar
        dependências entre níveis"]

        metricas["📊 Métricas de postura
        percentual de APIs
        em cada nível"]

        gates["🔒 Gates de publicação
        novas APIs começam
        no nível 1 no mínimo"]

        revisao["🔄 Revisão periódica
        APIs legadas migradas
        progressivamente"]
    end

    politica --- roadmap
    roadmap --- metricas
    metricas --- gates
    gates --- revisao

    style coe_zt fill:#f0f0ff,stroke:#6666cc
```

O CoE define o nível mínimo de postura Zero Trust exigido por criticidade — APIs públicas com dados sensíveis no mínimo Nível 2, APIs de microserviços internos no mínimo Nível 1. Novas APIs publicadas precisam atender o nível mínimo como gate de publicação. APIs legadas são migradas progressivamente conforme roadmap.

---

## Pontos-chave do capítulo

- O modelo de perímetro falha para APIs por três razões estruturais: APIs existem para romper o perímetro, o perímetro já foi comprometido e a rede interna fragmentada tornou a localização um proxy inválido de confiança
- O NIST SP 800-207 define sete tenets de Zero Trust. Aplicados a APIs: toda API é um recurso, TLS é obrigatório inclusive internamente, tokens têm vida curta, política é dinâmica por requisição, ativos são monitorados continuamente, autenticação é aplicada a cada requisição e aprendizado contínuo evolui a postura
- Identidade substitui localização como base de confiança. Identidades incluem usuários humanos, serviços, dispositivos e pipelines. Credenciais efêmeras — workload identity, managed identity — eliminam secrets estáticos de longa vida
- Verificação contínua vai além do token na entrada. Anomalias durante a sessão acionam step-up authentication ou revogação. Um token válido não é garantia de comportamento legítimo ao longo de toda a sessão
- Microsegmentação limita lateral movement — um serviço comprometido não alcança outros serviços porque cada serviço verifica a identidade do chamador. Service mesh implementa mTLS automático e microsegmentação de forma transparente para o código da aplicação
- Zero Trust não é uma tecnologia nova — é um framework que organiza os controles dos capítulos anteriores em uma estratégia coerente, acrescentando coerência, assume breach como premissa e verificação contínua
- Zero Trust não é binário. A adoção é incremental em quatro níveis. O CoE governa a jornada definindo nível mínimo por criticidade, roadmap de migração e gates de publicação

---

## Fontes e referências

| Fonte | Referência completa |
|---|---|
| **NIST SP 800-207 (2020)** | Rose, S., Borchert, O., Mitchell, S. & Connelly, S. *Zero Trust Architecture*. NIST SP 800-207, agosto 2020. Disponível em: [doi.org/10.6028/NIST.SP.800-207](https://doi.org/10.6028/NIST.SP.800-207) |
| **NIST SP 800-207A** | *Zero Trust Architecture Model for Cloud-Native Applications*. NIST SP 800-207A. Disponível em: [nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-207A.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-207A.pdf) |

---

## Próximo capítulo

**5.6 · Segurança no ciclo de vida — shift left** — como a segurança é incorporada em cada fase do ciclo de vida de APIs: threat modeling no design, SAST/SCA no pipeline, revisão de segurança como gate de publicação.

---

*Série: Gerenciamento e Governança de APIs · Módulo 5 · Capítulo 5.5*