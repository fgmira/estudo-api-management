# Anexo G · Autorização e controle de acesso em APIs

> **Referência:** Capítulo 5.3 · OWASP API Security Top 10
> **Cobre:** API1:2023 · API2:2023 · API5:2023
> **Série:** Gerenciamento e Governança de APIs

---

## G.1 · API1:2023 — Broken Object Level Authorization (BOLA)

### Por que é a vulnerabilidade mais prevalente

BOLA — também chamada de IDOR (Insecure Direct Object Reference) em contextos mais amplos — é consistentemente a categoria de maior impacto no OWASP API Security Top 10 porque combina alta prevalência com alta facilidade de exploração. Não requer ferramentas especializadas, não requer conhecimento técnico avançado e frequentemente não é detectada por nenhum controle automático.

A causa raiz é um erro conceitual no modelo de segurança: confundir autenticação com autorização. Um sistema que verifica "você está autenticado?" mas não verifica "você tem direito a este objeto específico?" está vulnerável a BOLA para qualquer objeto que use um identificador previsível ou obtido de uma resposta anterior.

### O mecanismo em detalhe

A exploração segue um padrão consistente:

**Passo 1 — Observação:** o atacante realiza uma ação legítima e observa o identificador do objeto retornado ou usado. `GET /pedidos/meus` retorna `{"id": "4521", ...}`.

**Passo 2 — Variação:** o atacante substitui o identificador pelo de outro objeto. `GET /pedidos/4520`, `GET /pedidos/4519`.

**Passo 3 — Extração:** se os objetos são retornados sem verificação de ownership, o atacante tem acesso a dados de outros usuários.

A variação pode ser incremental (IDs sequenciais), por enumeração (UUIDs obtidos de outras fontes) ou por referência cruzada (IDs obtidos de um endpoint para usar em outro).

### Casos documentados e impacto

O Anexo E cobriu casos específicos — Peloton, veículos conectados, plataforma acadêmica. O padrão é consistente: em todos os casos, autenticação estava implementada corretamente. A falha era exclusivamente na verificação de autorização por objeto.

O impacto varia de exposição de dados pessoais a comprometimento completo de contas e controle físico de dispositivos. A OWASP documenta BOLA como a categoria responsável pela maioria das notificações de violação de dados em APIs.

### Mitigação em profundidade

**No design (Cap 5.1):**
- Definir explicitamente no contrato que a operação retorna apenas objetos do usuário autenticado
- Usar UUIDs aleatórios em vez de IDs sequenciais para eliminar enumeração trivial
- Documentar no threat modeling qual é o mecanismo de verificação de ownership para cada operação

**Na implementação:**
```
Para cada operação que acessa um objeto por ID:
1. Extrair a identidade do consumidor do token (sub, client_id)
2. Buscar o objeto pelo ID fornecido
3. Verificar: objeto.owner == identidade do token
4. Se não corresponder: retornar 403 — nunca retornar o objeto
5. Se corresponder: retornar o objeto
```

A verificação deve acontecer na camada de aplicação — não pode ser delegada ao gateway. O gateway não conhece o modelo de dados da aplicação.

**Na governança:**
- Gate de revisão de segurança que verifica presença de verificação de ownership em todas as operações com objeto por ID
- Testes de autorização automatizados no pipeline: para cada endpoint, testar com token de usuário B tentando acessar objeto de usuário A

### CWEs associadas

- **CWE-639** — Authorization Bypass Through User-Controlled Key
- **CWE-284** — Improper Access Control

---

## G.2 · API2:2023 — Broken Authentication

### O que é autenticação quebrada em APIs

Autenticação quebrada em APIs é diferente de autenticação quebrada em aplicações web. Não é primariamente sobre senhas fracas ou sessões sem timeout — é sobre implementações incorretas de mecanismos de autenticação baseados em tokens.

As manifestações mais comuns:

**Tokens sem expiração ou com expiração muito longa** — um access token que nunca expira ou que expira em 24 horas é uma janela de ataque que persiste indefinidamente após um comprometimento.

**Secrets de JWT fracos** — JWT assinados com HS256 usando um secret previsível, curto ou default. Atacantes com o token podem tentar bruteforce do secret para assinar tokens arbitrários.

**Ausência de validação de claims** — verificar apenas a assinatura do JWT sem validar `exp`, `iss`, `aud` ou `sub` cria tokens válidos que não deveriam ser aceitos.

**Fluxos de autenticação com vulnerabilidades** — troca de código sem PKCE em flows de authorization code, permitindo interceptação; ausência de state parameter, permitindo CSRF em fluxos OAuth.

**Ausência de proteção contra credential stuffing** — endpoints de autenticação sem rate limiting por conta ou por IP que permitem testar listas de senhas comprometidas sem limitação.

### Por que padrões estabelecidos existem

A tentação de implementar autenticação customizada é real — OAuth 2.0 parece complexo, JWT parece simples de usar diretamente, e as necessidades específicas do sistema parecem justificar customizações. A história de vulnerabilidades de autenticação em APIs mostra que customizações quase sempre introduzem fraquezas que os padrões estabelecidos resolvem.

O RFC 9700 — Best Current Practice for OAuth 2.0 Security, publicado em janeiro 2025 — consolida anos de experiência em produção e vulnerabilidades descobertas para fornecer orientações precisas sobre como implementar OAuth 2.0 de forma segura. Seguir esse documento é mais seguro do que qualquer implementação customizada.

### Mitigação em profundidade

**Access tokens de vida curta:**
Access tokens com expiração de 5 a 15 minutos limitam drasticamente o impacto de um token comprometido. Refresh tokens de vida longa, armazenados de forma segura, permitem renovação sem impacto ao usuário.

**Validação completa de JWT:**
```
Para cada requisição com JWT:
1. Verificar assinatura (algoritmo declarado no header)
2. Verificar exp — token não expirou
3. Verificar iss — emissor esperado
4. Verificar aud — audience corresponde a esta API
5. Verificar sub — identidade do usuário presente
Rejeitar qualquer token que falhe em qualquer verificação
```

**PKCE obrigatório no Authorization Code Flow:**
PKCE — Proof Key for Code Exchange, RFC 7636 — protege o código de autorização contra interceptação. É obrigatório para clientes públicos e recomendado para todos os clientes segundo RFC 9700.

**Rate limiting em endpoints de autenticação:**
Limites muito mais restritivos do que endpoints regulares — 5 tentativas por minuto por conta, bloqueio progressivo após falhas repetidas.

### CWEs associadas

- **CWE-287** — Improper Authentication
- **CWE-798** — Use of Hard-coded Credentials
- **CWE-330** — Use of Insufficiently Random Values

---

## G.3 · API5:2023 — Broken Function Level Authorization (BFLA)

### A distinção entre BOLA e BFLA

BOLA é a falha de autorização horizontal — um usuário acessa objetos de outros usuários do mesmo nível. BFLA é a falha de autorização vertical — um usuário acessa funções de um nível de privilégio superior.

Em termos práticos: BOLA é "João acessa os pedidos de Maria". BFLA é "João acessa a função de administração que deveria ser acessível apenas por administradores".

### Como BFLA se manifesta em APIs

**Endpoints administrativos sem proteção adequada:**
Uma API com endpoints `/admin/usuarios` e `/admin/relatorios` que verificam apenas autenticação sem verificar se o token tem um escopo ou role administrativo.

**Diferenciação inadequada entre leitura e escrita:**
Um token com escopo `pedidos:read` que consegue chamar `DELETE /pedidos/{id}` porque o endpoint verifica apenas autenticação, não o escopo específico da operação.

**Endpoints "ocultos" sem proteção:**
A falsa sensação de segurança por obscuridade — endpoints administrativos que não aparecem na documentação mas existem em produção e não têm controles de acesso adequados.

**Escalada via encadeamento de APIs:**
Um usuário comum acessa uma API que internamente chama outra API com privilégios elevados, sem que a segunda API verifique a identidade original do usuário.

### Por que é difícil de detectar automaticamente

BFLA não tem um padrão técnico óbvio. Um atacante testando `DELETE /pedidos/4521` com um token de leitura está fazendo uma requisição tecnicamente válida — autenticação correta, endpoint existente. Apenas a verificação do escopo do token contra o escopo exigido pela operação detecta o problema.

Ferramentas de DAST podem tentar operações com tokens de baixo privilégio, mas dependem de conhecimento do modelo de autorização esperado.

### Mitigação em profundidade

**Autorização baseada em escopo para cada operação:**
```yaml
# No contrato OpenAPI
paths:
  /admin/usuarios:
    get:
      security:
        - oauth2: [admin:usuarios:read]   # escopo específico de admin
  /pedidos/{id}:
    delete:
      security:
        - oauth2: [pedidos:write]   # diferente de pedidos:read
```

**Princípio de deny-by-default:**
Qualquer operação sem verificação explícita de autorização deve ser rejeitada por padrão. A ausência de um controle de acesso não deve ser interpretada como permissão.

**Testes de autorização vertical automatizados:**
Para cada endpoint, testar com tokens de diferentes níveis de privilégio e verificar que operações privilegiadas são rejeitadas para tokens não privilegiados.

### CWEs associadas

- **CWE-285** — Improper Authorization
- **CWE-269** — Improper Privilege Management

---

## Padrões comuns às três categorias

As três categorias deste anexo compartilham uma causa raiz comum: **a separação inadequada entre autenticação e autorização**.

Autenticação responde à pergunta "quem é você?". Autorização responde à pergunta "o que você pode fazer?". Sistemas que implementam bem a autenticação e negligenciam a autorização são vulneráveis a todas as três categorias:

- **BOLA** — autorização por objeto ausente
- **BFLA** — autorização por função ausente
- **API2** — autenticação implementada de forma incorreta

A mitigação sistêmica passa por um modelo de autorização explícito — definido no design, declarado no contrato, verificado na implementação e testado no pipeline — que cobre tanto o nível de objeto quanto o nível de função para cada operação da API.

---

## Referências

| Fonte | Referência completa |
|---|---|
| **OWASP API Security Top 10 (2023)** | OWASP Foundation. Disponível em: [owasp.org/www-project-api-security](https://owasp.org/www-project-api-security/) |
| **RFC 9700 — OAuth 2.0 Security BCP (2025)** | Lodderstedt, T. et al. *Best Current Practice for OAuth 2.0 Security*. RFC 9700, janeiro 2025. Disponível em: [datatracker.ietf.org/doc/rfc9700](https://datatracker.ietf.org/doc/rfc9700/) |
| **RFC 7636 — PKCE** | Sakimura, N. et al. *Proof Key for Code Exchange by OAuth Public Clients*. RFC 7636, setembro 2015. Disponível em: [datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) |

---

*Série: Gerenciamento e Governança de APIs · Módulo 5 · Anexo G*