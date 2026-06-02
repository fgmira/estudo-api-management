# Anexo I · Recursos, injeção e gestão de ativos

> **Referência:** Capítulo 5.3 · OWASP API Security Top 10
> **Cobre:** API4:2023 · API7:2023 · API9:2023 · API10:2023
> **Série:** Gerenciamento e Governança de APIs

---

## I.1 · API4:2023 — Unrestricted Resource Consumption

### O que é consumo irrestrito de recursos

APIs sem limites de consumo de recursos são vetores naturais de negação de serviço, extração de dados e, em modelos de negócio baseados em consumo, abuso financeiro. A categoria abrange três dimensões distintas de consumo:

**Consumo de capacidade computacional** — operações sem limite de taxa que permitem que um único consumidor sature os recursos do servidor.

**Consumo de dados** — coleções sem paginação, filtros sem limite de intervalo, exportações sem aprovação que permitem extração da base completa em poucas requisições.

**Consumo de serviços externos** — APIs que chamam serviços de terceiros — envio de email, SMS, pagamento, serviços de IA — a cada requisição sem limites, multiplicando o custo de um ataque.

### Impacto além da disponibilidade

O impacto de API4 vai além da negação de serviço clássica:

**Impacto financeiro direto:** Em arquiteturas serverless ou em APIs que chamam serviços pagos por requisição, um atacante pode gerar custos significativos com poucas requisições bem posicionadas. Um endpoint de envio de SMS sem rate limiting pode custar milhares de reais em minutos.

**Extração de dados em escala:** Um endpoint de busca sem paginação ou com limite de page size muito alto permite extrair toda a base de dados de forma legítima — autenticado, dentro dos padrões técnicos.

**Degradação de qualidade para outros consumidores:** Mesmo sem intenção maliciosa, um consumidor sem rate limiting que faz uso intensivo pode degradar a experiência de todos os outros consumidores — o problema de vizinho barulhento em infraestrutura compartilhada.

### Mitigação em profundidade

**Rate limiting por camadas:**

```
Camada 1 — por IP: proteção contra ataques sem token
Camada 2 — por consumer (client_id): proteção contra abuso por consumidor específico
Camada 3 — por operação: limites diferenciados por risco e custo
Camada 4 — por conta de usuário: limites no nível do usuário final em APIs com delegação
```

**Paginação obrigatória com limite máximo de page size:**

```yaml
parameters:
  - name: limite
    in: query
    schema:
      type: integer
      minimum: 1
      maximum: 100   # nunca mais que isso, independente do que o consumidor solicite
      default: 20
```

**Limites de tamanho de payload:**

```yaml
requestBody:
  content:
    application/json:
      schema:
        maxLength: 1048576   # 1MB — nunca aceitar payloads sem limite
```

**Timeout em operações longas:**
Operações que consultam múltiplos backends, geram relatórios ou processam arquivos devem ter timeouts explícitos. Uma requisição sem timeout pode manter uma conexão aberta indefinidamente, consumindo recursos do servidor.

**Separação de operações síncronas e assíncronas:**
Operações longas não devem ser síncronas. Um endpoint que aceita uma requisição de processamento pesado, retorna imediatamente um ID de job e processa de forma assíncrona previne tanto timeout quanto abuso de recursos.

**CWEs associadas:**
- **CWE-770** — Allocation of Resources Without Limits or Throttling
- **CWE-400** — Uncontrolled Resource Consumption

---

## I.2 · API7:2023 — Server Side Request Forgery (SSRF)

### O mecanismo de SSRF em APIs

SSRF ocorre quando uma API aceita uma URL como input e realiza uma requisição HTTP para essa URL do lado do servidor, sem validar adequadamente o destino. O servidor se torna um proxy involuntário que o atacante usa para acessar recursos que de outra forma seriam inacessíveis.

Em APIs, SSRF é especialmente comum em funcionalidades como:
- Import de dados por URL
- Webhooks configuráveis pelo usuário
- Preview de links
- Conversão de documentos a partir de URLs
- Integração com serviços externos configurados pelo usuário

### Por que SSRF em cloud é especialmente crítico

Em ambientes cloud — AWS, GCP, Azure — cada instância tem acesso a um metadata service via endereço IP fixo (`169.254.169.254` em AWS e Azure, `169.254.169.254` e `metadata.google.internal` em GCP). Esse serviço expõe credenciais temporárias de IAM com os privilégios da role da instância.

Um SSRF que consegue fazer `GET http://169.254.169.254/latest/meta-data/iam/security-credentials/` pode obter credenciais temporárias de alta privilégio — permitindo acesso ao S3, RDS, Secrets Manager e outros serviços da conta AWS.

O impacto de um SSRF simples em cloud pode ser comprometimento completo da infraestrutura.

### Mitigação em profundidade

**Allowlist de destinos:**
A mitigação mais eficaz. Se a API aceita URLs, definir explicitamente quais domínios ou IPs são permitidos como destino. Qualquer URL fora da allowlist é rejeitada antes de qualquer requisição ser feita.

```
# Allowlist exemplo para um serviço de webhook
Permitido: *.empresa-parceira.com, api.servico-externo.com
Bloqueado: tudo o mais, incluindo:
  - 169.254.169.254 (metadata AWS/Azure)
  - 10.0.0.0/8 (rede interna)
  - 172.16.0.0/12 (rede interna)
  - 192.168.0.0/16 (rede interna)
  - 127.0.0.1 / localhost (loopback)
  - 0.0.0.0 (wildcard)
```

**Resolução DNS e verificação posterior:**
URLs com nomes de domínio podem apontar para IPs internos via DNS rebinding — o domínio resolve para um IP externo na validação inicial e para um IP interno na requisição real. Verificar o IP após resolução DNS, não apenas o hostname.

**Desabilitar redirecionamentos automáticos:**
Redirecionamentos HTTP podem ser usados para contornar validações de URL. A requisição inicial aponta para um destino válido que redireciona para o destino malicioso.

**Resposta mínima ao usuário:**
Não retornar o conteúdo completo da URL processada ao usuário quando não necessário. Limitar a informação de erro que poderia confirmar ao atacante se o destino interno foi alcançado.

**CWE associada:**
- **CWE-918** — Server-Side Request Forgery (SSRF)

---

## I.3 · API9:2023 — Improper Inventory Management

### Shadow APIs como risco estrutural

O Cap 4.8 tratou descoberta de APIs do prisma da governança em profundidade. Aqui a perspectiva é de segurança: APIs fora do inventário controlado são a superfície de ataque invisível — e por isso mais perigosa.

Shadow APIs tipicamente acumulam as vulnerabilidades das outras categorias por ausência dos controles que o processo de governança fornece:

- Sem revisão de segurança → nenhuma das vulnerabilidades das outras categorias foi avaliada
- Sem monitoramento → incidentes não são detectados
- Sem atualização → dependências com CVEs críticos não são atualizadas
- Sem rate limiting configurado → vulneráveis a API4

### APIs versões legadas

Além das shadow APIs genuínas — criadas fora do processo — o problema de inventário inclui versões de APIs que deveriam ter sido desativadas mas continuam ativas.

Uma API v1 mantida em produção "por precaução" após o lançamento da v2 é um vetor de ataque: pode ter vulnerabilidades corrigidas na v2, pode usar padrões de autenticação desatualizados, pode não receber atualizações de segurança porque ninguém percebeu que ainda tem tráfego.

O processo de sunset do Cap 2.6 existe precisamente para evitar esse acúmulo — mas apenas funciona quando o inventário está atualizado.

### APIs de terceiros sem avaliação

A organização consome APIs de terceiros — parceiros, fornecedores de dados, serviços de infraestrutura. Cada uma é um ponto de confiança que, se comprometida, pode propagar-se para a organização. A ausência de um inventário de APIs de terceiros consumidas é um problema análogo à ausência de inventário de APIs produzidas.

### Mitigação em profundidade

**Catálogo como condição de existência:**
Qualquer API que não está no catálogo deve ser tratada como não autorizada e sujeita a processo de regularização ou desligamento — não como exceção tolerada.

**Reconciliação periódica automatizada:**
Comparação regular entre o catálogo e o que o gateway, o service registry e os logs de tráfego conhecem. Discrepâncias são tratadas como incidentes de governança.

**Processo formal de sunset vinculado ao ciclo de vida:**
O processo de depreciação e sunset do Cap 2.6 deve ser iniciado com antecedência suficiente. Versões sem plano de sunset ativo representam dívida de segurança crescente.

**Inventário de APIs consumidas:**
O CMDB do Cap 4.3 deve registrar não apenas as APIs produzidas pela organização, mas as APIs de terceiros que os sistemas internos consomem — com avaliação de risco periódica.

**CWEs associadas:**
- **CWE-1059** — Insufficient Technical Documentation
- **CWE-710** — Improper Adherence to Coding Standards

---

## I.4 · API10:2023 — Unsafe Consumption of APIs

### A inversão do modelo de ameaça

As outras categorias do Top 10 tratam de ameaças externas que atacam as APIs que a organização produz. API10 inverte o modelo: trata das ameaças que entram pela cadeia de APIs que a organização consome.

A premissa incorreta mais comum: "essa é uma API do nosso parceiro de confiança, os dados que ela retorna podem ser tratados como confiáveis". Parceiros de confiança podem ser comprometidos. APIs de terceiros podem ter vulnerabilidades. Dados de fontes externas podem ser maliciosos — seja por comprometimento do fornecedor ou por injeção na cadeia.

### Como API10 se manifesta

**Injeção via dados de terceiros:**
Uma API consome dados de um fornecedor e os armazena ou exibe sem sanitização. Se o fornecedor for comprometido e injetar payloads de XSS, SQL injection ou command injection nos dados, esses payloads propagam-se para a organização consumidora.

**Dependência de disponibilidade sem circuit breaker:**
Uma API que chama um serviço externo de forma síncrona, sem timeout e sem circuit breaker, herda a instabilidade desse serviço. Uma degradação do parceiro se propaga para a API da organização.

**HTTPS não verificado para APIs internas:**
Comunicação entre microsserviços internos sem TLS, ou com verificação de certificado desabilitada "por comodidade", cria pontos de interceptação dentro da própria infraestrutura.

**Confiança excessiva em dados retornados:**
Usar diretamente em queries de banco de dados, comandos de sistema ou respostas a usuários dados retornados por APIs de terceiros sem sanitização.

### Mitigação em profundidade

**Tratar dados de APIs de terceiros como input não confiável:**
O mesmo tratamento de sanitização e validação aplicado a dados de usuários deve ser aplicado a dados de APIs de terceiros. A origem confiável não garante que o dado em si seja seguro.

**TLS obrigatório e verificação de certificado sempre ativada:**
Mesmo para comunicação interna entre microsserviços. Desabilitar verificação de certificado para "facilitar o desenvolvimento" cria um hábito que frequentemente chega à produção.

**Circuit breaker e timeout em todas as dependências externas:**

```
Para cada chamada a API externa:
- Timeout máximo: 5s (ou adequado ao caso de uso)
- Circuit breaker: abre após N falhas consecutivas
- Fallback: comportamento definido quando o circuit está aberto
- Retry com backoff exponencial: não sobrecarregar um serviço degradado
```

**Monitoramento de SLA de APIs consumidas:**
O service mapping do Cap 4.3 registra as dependências de APIs externas. O plano de observabilidade do Cap 3.8 deve monitorar a disponibilidade e latência dessas dependências — para detectar degradação antes que impacte os consumidores da organização.

**Avaliação de segurança de APIs de terceiros no onboarding:**
Antes de integrar uma nova API de terceiro, avaliação do nível de segurança do fornecedor: certificações, histórico de vulnerabilidades, processo de disclosure. Como discutido no Cap 4.2 na dimensão de Parceiros e Fornecedores.

**CWEs associadas:**
- **CWE-345** — Insufficient Verification of Data Authenticity
- **CWE-346** — Origin Validation Error

---

## Padrões comuns às quatro categorias

As quatro categorias deste anexo refletem um tema central: **o programa de APIs precisa gerenciar ativamente seus limites e dependências — tanto o que consome quanto o que produz, tanto o que é visível quanto o que está fora do inventário**.

- **API4** — limites de consumo de recursos não gerenciados
- **API7** — URLs e destinos de requisição não gerenciados
- **API9** — inventário de APIs não gerenciado
- **API10** — dependências de APIs externas não gerenciadas

A mitigação sistêmica passa por um programa de gestão de ativos e dependências — que o CMDB do Cap 4.3 e o processo de descoberta do Cap 4.8 habilita — e por controles de limite explícitos em cada operação.

---

## Referências

| Fonte | Referência completa |
|---|---|
| **OWASP API Security Top 10 (2023)** | OWASP Foundation. Disponível em: [owasp.org/www-project-api-security](https://owasp.org/www-project-api-security/) |
| **OWASP SSRF Prevention Cheat Sheet** | OWASP Foundation. Disponível em: [cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) |

---

*Série: Gerenciamento e Governança de APIs · Módulo 5 · Anexo I*