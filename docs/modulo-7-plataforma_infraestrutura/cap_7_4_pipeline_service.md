# Módulo 7 · A Plataforma de Governança
## Capítulo 7.4 · O Pipeline Service

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Técnico — especificação de serviço
> **Pré-requisito:** Cap 7.2 · Cap 7.3

---

## Sumário

- [7.4.1 · Responsabilidade e limites](#741--responsabilidade-e-limites)
- [7.4.2 · CI e CD — dois contextos distintos](#742--ci-e-cd--dois-contextos-distintos)
- [7.4.3 · A taxonomia de políticas do pipeline](#743--a-taxonomia-de-políticas-do-pipeline)
- [7.4.4 · O modelo assíncrono](#744--o-modelo-assíncrono)
- [7.4.5 · O spec bundle — resolver antes de enviar](#745--o-spec-bundle--resolver-antes-de-enviar)
- [7.4.6 · Gates platform-side vs. runner-side](#746--gates-platform-side-vs-runner-side)
- [7.4.7 · A interface de gate](#747--a-interface-de-gate)
- [7.4.8 · Environment profiles](#748--environment-profiles)
- [7.4.9 · O fluxo de exceções](#749--o-fluxo-de-exceções)
- [7.4.10 · agctl build — geração de configuração de gateway](#7410--agctl-build--geração-de-configuração-de-gateway)
- [7.4.11 · O ciclo de vida do deployment](#7411--o-ciclo-de-vida-do-deployment)
- [7.4.12 · Como os eventos alimentam o Analytics](#7412--como-os-eventos-alimentam-o-analytics)
- [Requisitos derivados](#requisitos-derivados)

---

## 7.4.1 · Responsabilidade e limites

O Pipeline Service tem uma responsabilidade única: **orquestrar a avaliação de qualidade e compliance de uma API ao longo do seu ciclo de vida**. Recebe specs, distribui avaliações, consolida resultados e publica eventos.

O que o Pipeline Service **não** faz:

- Não define políticas — isso é do Policy Service
- Não implementa lint, SAST ou DAST — ferramentas externas implementam a interface de gate
- Não faz deploy no gateway — o tooling de CD do time faz isso
- Não armazena o histórico analítico — isso é do Analytics Service
- Não acessa diretamente repositórios ou gateways — usa os adaptadores da camada de integração

```mermaid
graph TB
    subgraph pipeline_boundary["Pipeline Service — o que faz e o que não faz"]
        direction LR

        subgraph faz["Faz"]
            f1["Recebe spec bundle
            do agctl via API"]
            f2["Busca políticas aplicáveis
            do Policy Service"]
            f3["Orquestra gates em DAG
            com paralelismo e condições"]
            f4["Avalia políticas diretamente
            contra o conteúdo da spec"]
            f5["Delega capacidades especializadas
            a ferramentas via interface de gate"]
            f6["Consolida resultados
            aplica exceções ativas"]
            f7["Publica GateExecutado
            e RunConcluido no Event Bus"]
        end

        subgraph nao_faz["Não faz"]
            n1["Não define políticas
            Policy Service é o dono"]
            n2["Não implementa SAST · DAST
            ferramentas externas implementam"]
            n3["Não faz deploy
            tooling de CD do time faz"]
            n4["Não acessa repositórios
            nem gateways diretamente"]
        end
    end

    style faz fill:#e6ffe6,stroke:#009900
    style nao_faz fill:#ffe6e6,stroke:#cc0000
```

---

## 7.4.2 · CI e CD — dois contextos distintos

A mesma spec pode ser avaliada em dois contextos com propósitos e gates diferentes.

```mermaid
graph TB
    subgraph ci_context["Contexto CI — validação do contrato"]
        direction TB
        ci_trigger["Trigger: push · PR
        contexto: spec + documentação
        o deployment não existe ainda"]

        ci_gates["Gates disponíveis:
        estrutura e lint
        autenticação e segurança na spec
        server URL — formato e políticas
        quebra de contrato vs. versão anterior
        qualidade e AI Readiness
        validação semântica IA
        SAST — via runner · resultado enviado"]

        ci_result["Resultado:
        bloqueia ou libera merge
        feedback ao desenvolvedor
        expectativa: < 5 minutos"]

        ci_trigger --> ci_gates --> ci_result
    end

    subgraph cd_context["Contexto CD — validação do deployment"]
        direction TB
        cd_trigger["Trigger: deployment registrado
        contexto: API em execução
        o deployment existe"]

        cd_gates["Gates disponíveis:
        todos os gates CI +
        server-matches-deployment
        validação de segurança da infra
        DAST — via runner · resultado enviado
        contract testing
        post-deploy security scan"]

        cd_result["Resultado:
        bloqueia ou libera promoção de ambiente
        feedback ao operador
        expectativa: pode ser longo"]

        cd_trigger --> cd_gates --> cd_result
    end

    ci_context -->|"após merge e deploy"| cd_context

    style ci_context fill:#e6f0ff,stroke:#0044cc
    style cd_context fill:#fff0e6,stroke:#cc6600
```

**A regra fundamental:** validar server-matches-deployment ou segurança de infraestrutura em CI é impossível — o deployment não existe ainda. Esses gates são exclusivos do contexto CD.

---

## 7.4.3 · A taxonomia de políticas do pipeline

Políticas vivem no Policy Service. O Pipeline as busca e as avalia — cada categoria com sua lógica de avaliação própria.

```mermaid
mindmap
  root((Políticas
  avaliadas
  no Pipeline))
    Estrutura e lint
      Sintaxe válida da spec
      Campos obrigatórios presentes
      Convenções de nomeação
      Schemas com additionalProperties false
      Erros no formato RFC 7807
    Autenticação e segurança
      Security scheme declarado
      Toda operação com security requirement
      Escopos seguem taxonomia do portfólio
      Sem Implicit Grant declarado
      Rate limits declarados por operação
    Servidor e deployment
      HTTPS obrigatório em servers
      URL segue padrão do ambiente
      Server corresponde a deployment — apenas CD
      Certificado válido — apenas CD
    Quebra de contrato
      Comparação spec atual vs. anterior no Registry
      Regras por tipo — OpenAPI · gRPC · AsyncAPI · GraphQL
      Versão incrementada corretamente se breaking change
      Deprecation notice presente se removendo operação
    Qualidade e AI Readiness
      Descrições de operação presentes e não vazias
      Paginação em coleções
      Operações irreversíveis sinalizadas
      Idempotência declarada
      Edge cases documentados
    Semântica — via AI Service
      Coerência entre nome do recurso e atributos
      Descrições correspondem ao que a operação faz
      Consistência de vocabulário no domínio
      Completude contextual de edge cases
    Código — via runner
      SAST — análise estática de código-fonte
      Resultado enviado como GateOutput pelo runner
    Runtime — via runner · CD only
      DAST — API em execução
      Contract testing
      Post-deploy security scan
```

**O princípio que organiza essa taxonomia:** o Policy Service define as regras. O Pipeline decide como avaliá-las — diretamente contra a spec, delegando ao AI Service, ou aguardando resultado do runner.

---

## 7.4.4 · O modelo assíncrono

Gates têm latências muito diferentes — de centésimos de segundo (lint) a dezenas de minutos (DAST). Uma API síncrona geraria timeouts inevitáveis. O modelo correto é **job creation + polling**.

```mermaid
sequenceDiagram
    participant agctl as agctl no Runner
    participant PS as Pipeline Service
    participant Workers as Gate Workers

    agctl->>PS: POST /pipeline/runs
    Note over agctl,PS: {specBundle, context, environment,
    Note over agctl,PS:  apiId, commitRef, runnerGates}

    PS->>PS: cria Run · persiste estado
    PS->>Workers: distribui gates para execução
    PS->>agctl: 202 Accepted
    Note over PS,agctl: {runId, statusUrl,
    Note over PS,agctl:  estimatedDuration}

    loop polling (intervalo progressivo)
        agctl->>PS: GET /pipeline/runs/{runId}
        PS->>agctl: {status: running,
        Note over PS,agctl:  progress: {done: 3, total: 7,
        Note over PS,agctl:  current: "semantic-validation"}}
    end

    Workers->>PS: gates concluídos
    PS->>PS: consolida resultado

    agctl->>PS: GET /pipeline/runs/{runId}
    PS->>agctl: {status: completed,
    Note over PS,agctl:  result: fail,
    Note over PS,agctl:  gates: [...detalhes...]}

    agctl->>agctl: exit 0 ou 1
```

---

### Modos de execução do agctl

```bash
# Modo padrão — polling transparente com barra de progresso
agctl lint --spec api.yaml

✓ estrutura-lint          0.8s
✓ auth-policy             0.6s
✓ breaking-change         1.1s
⟳ semantic-validation     running...
✓ semantic-validation     8.3s
✗ doc-coherence           4.1s — 2 issues encontrados

Run: ❌ failed · run-id: run-abc-123
Ver detalhes: agctl run show run-abc-123

# Modo assíncrono — para pipelines com steps paralelos
agctl lint --spec api.yaml --async
# → RUN_ID=run-abc-123 (exit imediato)

# Em step posterior — aguarda resultado
agctl run wait --id $RUN_ID --timeout 15m

# Modo CI/CD — saída JSON para processamento
agctl lint --spec api.yaml --format json
# exit 0 = pass · exit 1 = fail
# stdout: JSON com detalhes completos
```

---

### Dois mecanismos de notificação — polling e webhook

O Pipeline Service suporta dois mecanismos para entregar o resultado de um run. A escolha depende de quem iniciou o run e se há alguém aguardando ativamente.

```mermaid
graph LR
    subgraph polling_model["Polling — alguém está esperando"]
        direction TB
        p1["Caso de uso:
        desenvolvedor aguardando no terminal
        pipeline de CI aguardando para decidir
        agctl rodando no runner"]
        p2["Como funciona:
        agctl faz GET /pipeline/runs/id
        em intervalos progressivos
        até receber status completed"]
        p3["Vantagem:
        progresso visível em tempo real
        controle total no cliente"]
    end

    subgraph webhook_model["Webhook — ninguém está esperando"]
        direction TB
        w1["Caso de uso:
        run disparado automaticamente pela plataforma
        check noturno de compliance em lote
        revalidação por novo CVE detectado
        notificação de sistema externo"]
        w2["Como funciona:
        cliente registra webhookUrl no POST
        pipeline processa em background
        plataforma faz POST no webhookUrl
        quando o run completa"]
        w3["Vantagem:
        cliente não precisa aguardar
        eficiente para runs em lote
        integra com Slack · Teams
        GitHub Status API · monitoring"]
    end

    style polling_model fill:#e6f0ff,stroke:#0044cc
    style webhook_model fill:#e6ffe6,stroke:#009900
```

**Polling — o fluxo do agctl:**
```bash
# agctl abstrai o polling — transparente para o desenvolvedor
agctl lint --spec api.yaml
# internamente: POST → runId → GET em loop → exibe progresso → exit
```

**Webhook — o fluxo de sistemas automatizados:**
```json
POST /pipeline/runs
{
  "specBundle": "...",
  "context": "ci",
  "webhookUrl": "https://ci.empresa.com/hooks/agctl/run-complete"
}
→ 202 Accepted {runId}
→ cliente segue com outras tarefas
→ plataforma faz POST no webhookUrl quando termina
```

**Segurança do webhook — o payload é assinado:**
```
POST https://ci.empresa.com/hooks/agctl/run-complete
X-AGCTL-Signature: sha256=<HMAC com secret compartilhado>
X-AGCTL-Run-Id: run-abc-123
X-AGCTL-Timestamp: 1700000000
```
O destinatário verifica o HMAC antes de processar. Mesmo padrão do GitHub, Stripe e Slack.

**Quando usar cada um:**

| Situação | Mecanismo |
|---|---|
| Desenvolvedor aguardando no terminal | Polling — agctl mostra progresso |
| Pipeline de CI decidindo se faz merge | Polling — agctl no runner |
| Run disparado automaticamente pela plataforma | Webhook |
| Notificação para Slack · Teams · monitoring | Webhook |
| Check noturno em lote de muitas APIs | Webhook |
| Revalidação por novo CVE em múltiplas APIs | Webhook |
| DAST longo com steps paralelos no CD | Ambos — `--async` + webhook de backup |

---

## 7.4.5 · O spec bundle — resolver antes de enviar

Specs OpenAPI frequentemente referem a outros arquivos via `$ref`. O agctl resolve essas referências antes de enviar qualquer coisa para a plataforma — gerando um documento único e auto-contido.

```mermaid
graph LR
    subgraph repo["Repositório — distribuído"]
        main["specs/pedidos-api.yaml
        $ref: ./schemas/order.yaml
        $ref: ./schemas/customer.yaml
        $ref: ./shared/errors.yaml"]
        s1["schemas/order.yaml"]
        s2["schemas/customer.yaml"]
        s3["shared/errors.yaml"]
        s4["shared/address.yaml
        ($ref aninhado em customer.yaml)"]
    end

    subgraph agctl_process["agctl — antes de enviar"]
        bundle["Resolve todos os refs
        gera documento único
        auto-contido
        tipicamente < 500KB"]
    end

    subgraph platform["Plataforma recebe"]
        result["spec bundle completo
        sem dependência do repositório
        avaliação completa possível"]
    end

    repo --> agctl_process --> platform
```

**Por tipo de spec:**
- **OpenAPI**: `$ref` para schemas, paths, components — bundling via `@apidevtools/swagger-parser` ou equivalente
- **gRPC proto**: `import` statements — bundling agrupa todos os .proto referenciados
- **AsyncAPI**: `$ref` para schemas e channels
- **GraphQL SDL**: arquivos de schema separados

O agctl detecta automaticamente o tipo pelo conteúdo ou pela extensão e usa o bundler adequado.

---

## 7.4.6 · Gates platform-side vs. runner-side

```mermaid
graph TB
    subgraph platform_gates["Gates platform-side
    Pipeline Service executa internamente"]
        direction LR
        pg1["Estrutura e lint
        avalia spec bundle"]
        pg2["Autenticação e segurança
        avalia spec bundle"]
        pg3["Quebra de contrato
        spec bundle vs. Registry"]
        pg4["Qualidade e AI Readiness
        spec bundle + AI Service"]
        pg5["Server — formato e políticas
        spec bundle"]
        pg6["Validação semântica
        spec bundle via AI Service"]
    end

    subgraph runner_gates["Gates runner-side
    runner executa · agctl envia resultado"]
        direction LR
        rg1["SAST
        runner tem acesso ao código
        executa Semgrep · SonarQube · Snyk
        envia GateOutput estruturado"]
        rg2["DAST — CD only
        runner tem acesso ao staging
        executa OWASP ZAP · Burp
        envia GateOutput estruturado"]
        rg3["Contract testing — CD only
        runner executa Pact
        envia GateOutput estruturado"]
        rg4["Post-deploy security — CD only
        agctl deploy-verify
        aciona Security Scanner
        via Discovery Service"]
    end

    subgraph submit["Como o runner envia resultados"]
        s1["agctl gate submit
        --gate sast
        --run-id $RUN_ID
        --results sast-results.json"]
    end

    runner_gates --> submit
    submit -->|"POST /pipeline/runs/{id}/gates/{gate}"| PS["Pipeline Service
    consolida resultado"]

    style platform_gates fill:#e6ffe6,stroke:#009900
    style runner_gates fill:#e6f0ff,stroke:#0044cc
```

O run fica em `waiting-for-runner-gates` até que todos os runner gates sejam submetidos ou o timeout configurado seja atingido.

---

## 7.4.7 · A interface de gate

Todo gate — seja executado internamente pela plataforma ou por uma ferramenta externa — implementa a mesma interface.

```
GateInput {
  gateId:      string          identificador único do gate
  gateName:    string          nome legível
  specBundle:  string          spec resolvida
  specType:    openapi | graphql | grpc | asyncapi | mcp
  context:     ci | cd
  environment: experimental | development | staging | production
  apiId:       UUID            referência ao Registry
  runId:       UUID            identificador do run
  policies:    [Policy]        políticas aplicáveis a este gate
}

GateOutput {
  gateId:      string
  status:      pass | fail | warn | skip | info
  issues:      [Issue] {
    severity:  block | warn | info
    code:      string          código da política violada
    message:   string          o que está errado
    suggestion:string          como corrigir
    path:      string          onde na spec — ex: paths./pedidos.get.security
  }
  duration:    milliseconds
  metadata:    {}              dados adicionais específicos do gate
}
```

A interface é o contrato. Qualquer ferramenta que produz um `GateOutput` válido pode ser usada como gate. O CoE adiciona um novo gate configurando a ferramenta e mapeando seu output para a interface — sem modificar o Pipeline Service.

---

## 7.4.8 · Environment profiles

O mesmo gate tem comportamentos diferentes dependendo do ambiente de destino. O CoE configura profiles no Policy Service — o Pipeline os aplica automaticamente baseado no ambiente do run.

```mermaid
graph LR
    subgraph profiles["Environment Profiles — enforcement por ambiente"]
        direction TB

        exp["🧪 experimental
        Padrão: SKIP
        Qualquer policy não listada: SKIP
        Objetivo: máxima liberdade
        para experimentação"]

        dev["🔧 development
        Padrão: WARN
        Qualquer policy não listada: WARN
        Objetivo: visibilidade sem bloqueio"]

        stg["🔍 staging
        Padrão: BLOCK
        Mesmo rigor que produção
        Objetivo: última chance antes
        de ir a produção"]

        prd["🚀 production
        Padrão: BLOCK
        Nenhuma policy pode ser SKIP
        Objetivo: compliance total"]
    end

    subgraph exemplo["Exemplo de configuração pelo CoE"]
        direction TB
        e1["auth-required:
        experimental: SKIP
        development:  WARN
        staging:      BLOCK
        production:   BLOCK"]

        e2["https-required:
        experimental: SKIP
        development:  SKIP
        staging:      BLOCK
        production:   BLOCK"]

        e3["breaking-changes:
        experimental: SKIP
        development:  WARN
        staging:      BLOCK
        production:   BLOCK"]

        e4["description-required:
        experimental: SKIP
        development:  INFO
        staging:      WARN
        production:   BLOCK"]
    end

    style exp fill:#f0f0f0,stroke:#999999
    style dev fill:#fff9e6,stroke:#cc9900
    style stg fill:#e6f0ff,stroke:#0044cc
    style prd fill:#e6ffe6,stroke:#009900
```

O branchMapping do Repository (Cap 7.3) conecta branches a ambientes — e ambientes a profiles. O desenvolvedor faz push para `feature/nova-ideia` e automaticamente recebe o profile `experimental`. Sem configuração adicional.

---

## 7.4.9 · O fluxo de exceções

Quando um gate falha, o desenvolvedor pode solicitar uma exceção formal. A exceção é **detectada pelo Pipeline, decidida pelo Policy Service**.

```mermaid
sequenceDiagram
    participant Dev as Desenvolvedor
    participant agctl as agctl
    participant PS as Pipeline Service
    participant Policy as Policy Service
    participant CoE as Arquiteto do CoE
    participant Analytics as Analytics Service

    Dev->>agctl: agctl lint --spec api.yaml
    PS->>Policy: avalia políticas
    Policy->>PS: auth-policy falhou
    PS->>agctl: ❌ auth-policy — security scheme ausente em 2 operações

    agctl->>Dev: Gate falhou. Solicitar exceção?
    Dev->>agctl: agctl exception request
    Note over Dev,agctl: --gate auth-policy
    Note over Dev,agctl: --api pedidos-api
    Note over Dev,agctl: --reason "prazo crítico · corrigir na sprint 42"
    Note over Dev,agctl: --expires 2025-07-15

    agctl->>Policy: POST /exceptions/requests
    Policy->>CoE: notifica — exceção pendente

    CoE->>Policy: aprova com prazo confirmado
    Policy->>Dev: notificado — exceção aprovada até 15/07

    Dev->>agctl: agctl lint --spec api.yaml
    PS->>Policy: avalia políticas + exceções ativas
    Policy->>PS: auth-policy — EXCEÇÃO ATIVA até 15/07
    PS->>agctl: ✅ pass-with-exception

    Policy->>Analytics: ExceçãoConcedida {gate, api, expires, approver}

    Note over Analytics: Analytics monitora padrões:
    Note over Analytics: auth-policy tem 8 exceções ativas
    Note over Analytics: → sinaliza ao CoE: política pode estar errada
    Note over Analytics: (o 20º aforisma em ação)
```

**O que torna a exceção governada:**
- Justificativa obrigatória — o desenvolvedor explica o porquê
- Prazo obrigatório — sem exceções eternas
- Aprovação explícita do CoE — não apenas notificação
- Auditável — aparece no resultado como `pass-with-exception`, não `pass`
- Expiração automática — quando o prazo vence, o gate volta a falhar
- Analytics detecta padrões — muitas exceções para o mesmo gate é sinal de política incorreta

---

## 7.4.10 · agctl build — geração de configuração de gateway

O `agctl build` gera configurações de gateway **derivadas da spec e das políticas do CoE**. A governança que a plataforma define se materializa na configuração que vai ao gateway.

```mermaid
graph TB
    subgraph build_inputs["Inputs para agctl build"]
        i1["Spec bundle
        o contrato da API"]
        i2["Políticas do CoE
        auth · rate limits · cors
        escopos · headers de segurança"]
        i3["Gateway e ambiente de destino
        kong · aws · azure · apigee
        production · staging"]
    end

    subgraph build_outputs["Outputs por gateway"]
        o1["Kong
        deck.yaml
        rotas · plugins JWT · rate limiting
        consumer mappings · CORS config"]

        o2["AWS API Gateway
        openapi-annotated.yaml
        x-amazon-apigateway-auth
        x-amazon-apigateway-integration
        ou: template.yaml (SAM)"]

        o3["Azure APIM
        api.json + policy.xml
        autenticação · throttling
        transformation policies"]

        o4["Apigee
        apiproxy bundle
        policies XML · target endpoints"]
    end

    subgraph validate["agctl validate-build — antes de aplicar"]
        v1["Valida artefato gerado
        IaC scanning:
        authorizer declarado?
        IAM com least privilege?
        resource policy restritiva?
        WAF associado?"]
    end

    build_inputs --> build_outputs
    build_outputs --> validate
```

```bash
# Gera artefato para Kong em staging
agctl build --spec api.yaml --gateway kong --environment staging
# output: deck-staging.yaml

# Valida segurança do artefato antes de aplicar
agctl validate-build --artifact deck-staging.yaml --gateway kong

✓ JWT plugin declarado
✓ Rate limiting configurado
✓ CORS restrito às origens aprovadas
✗ mTLS não está configurado — WARN (staging: WARN, production: BLOCK)

# O time aplica com o tooling que já usa
deck apply -s deck-staging.yaml
```

**Por que o agctl build, não o desenvolvedor manualmente:**
Um artefato gerado pelo agctl **garante que as políticas do CoE estão aplicadas**. Um artefato criado manualmente pode divergir — intencionalmente ou por descuido. O mesmo sistema que define as políticas gera a infraestrutura que as implementa.

---

## 7.4.11 · O ciclo de vida do deployment

O Pipeline Service coordena o registro do deployment no Registry via agctl — sem fazer o deploy em si.

```mermaid
sequenceDiagram
    participant CD as Pipeline de CD
    participant agctl as agctl
    participant PS as Pipeline Service
    participant Registry as Registry Service
    participant Tooling as Tooling de CD
    participant Bus as Event Bus

    Note over CD: Antes do deploy

    CD->>agctl: agctl deployment register
    Note over CD,agctl: --api pedidos-api --spec v2.1
    Note over CD,agctl: --server kong-prod --environment production

    agctl->>Registry: POST /deployments
    Registry->>agctl: deploymentId: dep-xyz · status: pending

    Note over CD: Deploy acontece no tooling do time

    CD->>Tooling: deck apply -s deck-production.yaml
    Tooling->>Tooling: aplica no Kong

    alt Deploy bem-sucedido
        CD->>agctl: agctl deployment confirm --id dep-xyz
        agctl->>Registry: PATCH /deployments/dep-xyz → active
        Registry->>Bus: DeploymentAtivado

        Note over CD: Verificação pós-deploy
        CD->>agctl: agctl deploy-verify --id dep-xyz
        agctl->>PS: aciona Security Scanner
        PS->>agctl: SecurityScanResult
        agctl->>CD: ✅ configuração de segurança validada

    else Deploy falhou
        CD->>agctl: agctl deployment cancel --id dep-xyz --reason "deck apply failed"
        agctl->>Registry: PATCH /deployments/dep-xyz → failed
        Registry->>Bus: DeploymentFalhou
    end

    alt Rollback necessário
        CD->>agctl: agctl deployment rollback
        Note over CD,agctl: --api pedidos-api
        Note over CD,agctl: --environment production
        Note over CD,agctl: --reason "instabilidade detectada"

        agctl->>Registry: POST /deployments/dep-xyz/rollback
        Registry->>Registry: dep-xyz: active → rolled-back
        Registry->>Registry: dep-anterior: deprecated → active
        Registry->>Bus: DeploymentRolledBack
    end
```

---

## 7.4.12 · Como os eventos alimentam o Analytics

Cada gate executado e cada run concluído produz eventos que o Analytics Service consome para construir inteligência de portfólio.

```mermaid
graph TB
    subgraph eventos_pipeline["Eventos do Pipeline Service"]
        e1["GateExecutado
        gateId · gateName · apiId
        status · issues · duration
        context · environment · runId"]

        e2["RunConcluido
        runId · apiId · context
        gatesTotal · gatesPassed
        hasExceptions · duration
        specType · specRef"]

        e3["ExceçãoConcedida
        gateId · apiId · reason
        approver · expires"]

        e4["ExceçãoExpirada
        gateId · apiId
        gate volta a falhar"]
    end

    subgraph analytics_value["O que o Analytics extrai ao longo do tempo"]
        direction LR

        av1["Taxa de falha por gate por domínio
        'security-policy falha 60% das vezes
        no domínio de pagamentos'"]

        av2["Friction real
        'doc-coherence tem a maior
        taxa de abandono — devs param de tentar'"]

        av3["Tendência por time
        'team-checkout melhorou de 45%
        para 82% de compliance em 6 semanas'"]

        av4["Sinal de política errada
        'auth-policy tem 12 exceções ativas
        — pode ser hora de revisar'
        o 20º aforisma detectado por dados"]

        av5["Tempo médio de iteração
        'devs rodam lint 3.2x em média
        antes de conseguir publish'"]

        av6["Correlação gate failure → incidente
        'APIs que falham em security-policy
        têm 3x mais incidentes'"]
    end

    eventos_pipeline -->|"via Event Bus"| analytics_value

    style av4 fill:#fff9e6,stroke:#cc9900
    style av6 fill:#ffe6e6,stroke:#cc0000
```

A correlação gate failure → incidente — o último item — é o insight que só aparece depois de meses de dados acumulados. É o que transforma a plataforma de ferramenta de enforcement em sistema de inteligência de governança.

---

## Requisitos derivados

| # | Requisito | Origem |
|---|---|---|
| R-7.4.1 | O Pipeline Service é orquestrador — não implementa lint, SAST, DAST nem define políticas | Responsabilidade única |
| R-7.4.2 | Toda chamada ao Pipeline Service é assíncrona — POST retorna 202 com runId | Modelo assíncrono |
| R-7.4.3 | O agctl executa polling progressivo internamente — transparente para o desenvolvedor | CLI como abstração |
| R-7.4.4 | O agctl bundleia a spec resolvendo todos os refs antes de enviar | Spec bundle |
| R-7.4.5 | O Pipeline Service suporta gates platform-side e runner-side via agctl gate submit | Conectividade |
| R-7.4.6 | Todo gate implementa GateInput/GateOutput — a plataforma não assume nada além dessa interface | Interface de gate |
| R-7.4.7 | Políticas têm enforcement level configurável por ambiente — BLOCK · WARN · SKIP · INFO | Environment profiles |
| R-7.4.8 | Exceções são aprovadas pelo CoE via Policy Service — o Pipeline apenas detecta a necessidade | Separação de responsabilidades |
| R-7.4.9 | Exceções têm prazo obrigatório e expiram automaticamente | Governança de exceções |
| R-7.4.10 | agctl build gera configuração de gateway derivada da spec e das políticas — sem intervenção manual | Build de gateway |
| R-7.4.11 | agctl validate-build valida o artefato gerado antes de qualquer deploy | IaC scanning |
| R-7.4.12 | O ciclo de deployment tem três passos via agctl: register → confirm/cancel → rollback opcional | Ciclo de deployment |
| R-7.4.13 | GateExecutado e RunConcluido são publicados no Event Bus para consumo pelo Analytics Service | Dados como produto |
| R-7.4.14 | O Analytics Service detecta padrões de exceções repetidas e sinaliza políticas candidatas a revisão | O 20º aforisma por dados |

---

## Próximo capítulo

**7.5 · O Policy Service** — o sistema que define, versiona e avalia políticas como código — com o ciclo de vida das políticas e o mecanismo de exceções em profundidade.

---

*Série: Gerenciamento e Governança de APIs · Módulo 7 · Capítulo 7.4*