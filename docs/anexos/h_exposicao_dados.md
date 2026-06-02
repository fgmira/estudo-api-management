# Anexo H · Exposição de dados e configuração

> **Referência:** Capítulo 5.3 · OWASP API Security Top 10
> **Cobre:** API3:2023 · API6:2023 · API8:2023
> **Série:** Gerenciamento e Governança de APIs

---

## H.1 · API3:2023 — Broken Object Property Level Authorization

### O que une exposição excessiva e mass assignment

A edição 2023 do OWASP Top 10 unificou duas vulnerabilidades que na edição 2019 eram separadas — Excessive Data Exposure (API3:2019) e Mass Assignment (API6:2019) — em uma única categoria: Broken Object Property Level Authorization.

A unificação é correta. As duas vulnerabilidades são faces opostas do mesmo problema de design: o schema de resposta e o schema de entrada de uma API não são projetados a partir das necessidades do consumidor — são reflexos do modelo de dados interno.

**Exposição excessiva** — o schema de resposta retorna todas as propriedades do objeto, incluindo as que o consumidor não precisa e não deveria ver.

**Mass assignment** — o schema de entrada aceita todas as propriedades do objeto, incluindo as que o consumidor não deveria poder alterar.

### Exposição excessiva em detalhe

A causa mais comum: serialização automática do objeto de domínio ou de banco de dados como resposta da API. Frameworks que mapeiam automaticamente entidades para JSON facilitam o desenvolvimento mas criam exposição sistemática.

O que fica exposto além do necessário:

- Campos de sistema — `createdAt`, `updatedAt`, `deletedAt`, `version`
- Flags internas — `isAdmin`, `emailVerified`, `fraudScore`
- Dados de outros contextos — `passwordHash`, `resetToken`, `internalNotes`
- Relacionamentos desnecessários — objetos completos quando apenas IDs são necessários
- Metadados de implementação — nomes de tabelas, IDs internos de banco de dados

A consequência vai além da privacidade. Campos expostos revelam o modelo de dados interno — informação valiosa para um atacante que planeja mass assignment ou injeção.

### Mass assignment em detalhe

Mass assignment acontece quando o backend aceita um objeto de entrada e o aplica diretamente ao modelo de dados sem filtrar quais campos são permitidos para aquele consumidor naquele contexto.

O padrão de exploração mais comum:

1. O atacante chama `GET /usuarios/eu` e recebe o objeto completo, incluindo campos como `role: "user"`, `creditos: 0`
2. O atacante chama `PATCH /usuarios/eu` com `{"role": "admin", "creditos": 999999}`
3. Se o backend aplica o objeto de entrada diretamente ao modelo, o atacante se tornou administrador com créditos ilimitados

### Mitigação em profundidade

**Separação explícita de schemas de leitura e escrita:**

Cada endpoint deve ter schemas de input e output distintos, projetados especificamente para o caso de uso — não derivados do modelo de dados interno.

```yaml
components:
  schemas:
    # Schema de resposta — apenas o que o consumidor precisa ver
    UsuarioPublico:
      type: object
      properties:
        id: { type: string, format: uuid }
        nome: { type: string }
        avatarUrl: { type: string }

    # Schema de entrada para atualização de perfil
    AtualizacaoPerfilRequest:
      type: object
      additionalProperties: false   # rejeita qualquer campo não listado
      properties:
        nome: { type: string, maxLength: 100 }
        avatarUrl: { type: string, format: uri }
      # role, creditos, emailVerificado não existem neste schema
```

**Projeções específicas por caso de uso:**

A mesma entidade pode ter múltiplos schemas de resposta para diferentes contextos: `UsuarioPublico` para listas, `UsuarioPerfil` para o perfil do próprio usuário, `UsuarioAdmin` apenas em endpoints administrativos com escopo específico.

**CWEs associadas:**
- **CWE-213** — Exposure of Sensitive Information Due to Incompatible Policies
- **CWE-915** — Improperly Controlled Modification of Dynamically-Determined Object Attributes

---

## H.2 · API6:2023 — Unrestricted Access to Sensitive Business Flows

### Por que é diferente das outras categorias

API6 é a categoria mais difícil de mitigar com controles técnicos tradicionais porque não representa uma vulnerabilidade técnica — representa o abuso de funcionalidade legítima em escala ou frequência inadequadas.

Um bot que compra todos os ingressos disponíveis de um show em milissegundos está usando a API de compra exatamente como foi projetada — autenticado, dentro do rate limit por requisição, com payload válido. A vulnerabilidade não está na implementação técnica; está na ausência de controles que detectem e limitem o padrão de uso abusivo.

### Fluxos de negócio sensíveis e seus riscos

**Compra de itens com desconto ou estoque limitado:**
Bots que monitoram e compram automaticamente, privando usuários legítimos de acesso. Impacto: dano reputacional, perda de receita de consumidores legítimos.

**Cadastro em massa:**
Criação automatizada de contas para uso em spam, fraude, abuso de promoções de boas-vindas ou manipulação de métricas. Impacto: custos operacionais, manipulação de dados, fraude.

**Votação ou rating repetido:**
Manipulação de métricas de popularidade, reviews ou rankings através de votos automatizados. Impacto: distorção de dados, perda de confiança na plataforma.

**Abuso de promoções:**
Utilização de códigos de desconto ou créditos em escala superior ao intencionado. Impacto: perda financeira direta.

**Enumeração de disponibilidade:**
Verificação sistemática de disponibilidade de produtos, preços ou usuários para scraping de dados competitivos.

### Mitigação em profundidade

A mitigação de API6 requer uma combinação de controles que nenhum controle isolado consegue fornecer:

**Identificação dos fluxos sensíveis:**
O primeiro passo é identificar quais fluxos de negócio têm valor suficiente para serem alvos de abuso automatizado. Nem todas as APIs têm fluxos sensíveis — mas as que têm precisam de controles específicos.

**Device fingerprinting e detecção de automação:**
Análise de características da requisição que distinguem usuários humanos de bots: timing entre requisições, ausência de headers típicos de browsers, padrões de uso inconsistentes com comportamento humano.

**CAPTCHA em fluxos de alto valor:**
Para fluxos onde a fricção é aceitável — cadastro, compra de itens de alta demanda — CAPTCHA ou desafios similares criam barreira para automação.

**Rate limiting específico por fluxo de negócio:**
Limites definidos pelo contexto de negócio, não apenas por volume de requisições. "Máximo de 2 compras por hora por conta" é um limite de negócio diferente de "máximo de 100 requisições por minuto".

**Análise comportamental e scoring:**
Sistemas que atribuem scores de confiança a consumidores baseados no histórico de comportamento, aplicando controles progressivos conforme o score cai.

**CWEs associadas:**
- **CWE-770** — Allocation of Resources Without Limits or Throttling
- **CWE-799** — Improper Control of Interaction Frequency

---

## H.3 · API8:2023 — Security Misconfiguration

### O que é misconfiguration em APIs

Security Misconfiguration é a categoria mais ampla do Top 10 — pode afetar qualquer componente do stack de APIs: o gateway, a aplicação, o servidor, o container, a rede, o provedor de identidade.

O OWASP documenta as manifestações mais comuns:

**CORS mal configurado:**
`Access-Control-Allow-Origin: *` em APIs que retornam dados sensíveis permite que qualquer site web faça requisições autenticadas em nome de um usuário. Em APIs que usam cookies de sessão, isso é crítico.

**Headers de segurança ausentes:**
`Strict-Transport-Security` ausente permite downgrade de HTTPS para HTTP. `X-Content-Type-Options` ausente permite ataques de MIME sniffing. `Content-Security-Policy` ausente em portais de desenvolvedores permite XSS.

**Mensagens de erro com informação técnica:**
Stack traces, nomes de classes, queries SQL e versões de frameworks em respostas de erro fornecem ao atacante um mapa do sistema interno — como discutido no Anexo E.

**HTTPS desabilitado ou TLS desatualizado:**
APIs acessíveis via HTTP, ou que aceitam TLS 1.0 e 1.1, expõem tráfego a interceptação.

**Defaults inseguros não modificados:**
Credenciais padrão de administração, portas de debug abertas, endpoints de health check que expõem informação sensível sobre o ambiente.

**Permissões excessivas em cloud:**
Roles de IAM com permissões mais amplas do que o necessário, buckets de storage acessíveis publicamente, grupos de segurança com regras excessivamente permissivas.

### Por que misconfiguration é persistente

Misconfiguration persiste porque é frequentemente invisível até ser explorada. Um header de segurança ausente não causa erros — a aplicação funciona normalmente. Uma configuração de CORS permissiva não quebra nenhuma funcionalidade — pelo contrário, facilita o desenvolvimento. Defaults inseguros são inseguros precisamente porque permitem que tudo funcione sem configuração.

O segundo fator é a pressão de prazo: configurações de segurança são frequentemente adiadas para "depois que o sistema estiver funcionando" — e o depois nunca chega.

### Mitigação em profundidade

**Hardening como código no pipeline:**

Configurações de segurança devem ser declaradas como código, versionadas e validadas automaticamente no pipeline — não verificadas manualmente de forma periódica.

Headers de segurança obrigatórios verificados por lint da spec:
```yaml
# Verificar que o gateway adiciona os headers obrigatórios
x-security-headers:
  required:
    - Strict-Transport-Security
    - X-Content-Type-Options
    - X-Frame-Options
    - Content-Security-Policy
```

**Configuração de CORS explícita e restritiva:**

```yaml
cors:
  allowedOrigins:
    - "https://app.empresa.com"
    - "https://portal.empresa.com"
  # Nunca: allowedOrigins: ["*"] em APIs com dados sensíveis
  allowedMethods: [GET, POST, PUT, DELETE]
  allowedHeaders: [Authorization, Content-Type]
  exposeHeaders: [X-Request-ID]
  maxAge: 3600
```

**Ambientes de staging com configuração idêntica à produção:**

Misconfiguration frequentemente é introduzida porque staging tem configurações diferentes de produção. Infraestrutura como código que usa os mesmos templates em todos os ambientes — com apenas parâmetros de escala e credenciais diferentes — elimina essa fonte de divergência.

**Rotação e revisão periódica de permissões:**

Permissões em cloud, roles de serviço e credenciais de integração crescem ao longo do tempo via acumulação — cada nova necessidade adiciona permissões sem remover as antigas. Revisões periódicas e princípio de least privilege aplicado sistematicamente são o controle adequado.

**CWEs associadas:**
- **CWE-16** — Configuration
- **CWE-614** — Sensitive Cookie in HTTPS Session Without 'Secure' Attribute

---

## Padrões comuns às três categorias

As três categorias deste anexo compartilham um padrão: **decisões de design e configuração que facilitam o desenvolvimento mas criam superfície de ataque**.

Retornar o objeto completo é mais simples do que projetar um schema. Aceitar todos os campos é mais simples do que declarar explicitamente os permitidos. Usar defaults é mais simples do que configurar explicitamente cada parâmetro de segurança.

A mitigação sistêmica passa por tornar o caminho seguro o caminho mais simples: templates de schema que incluem `additionalProperties: false` por padrão, pipelines que rejeitam specs sem security schemes declarados, infraestrutura como código que aplica hardening automaticamente.

---

## Referências

| Fonte | Referência completa |
|---|---|
| **OWASP API Security Top 10 (2023)** | OWASP Foundation. Disponível em: [owasp.org/www-project-api-security](https://owasp.org/www-project-api-security/) |
| **OWASP Cheat Sheet — CORS** | OWASP Foundation. *Cross-Origin Resource Sharing Cheat Sheet*. Disponível em: [cheatsheetseries.owasp.org/cheatsheets/CORS_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/CORS_Cheat_Sheet.html) |

---

*Série: Gerenciamento e Governança de APIs · Módulo 5 · Anexo H*