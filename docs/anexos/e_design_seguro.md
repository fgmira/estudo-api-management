# Anexo E · Falhas de design seguro em APIs — casos práticos

> **Referência:** Capítulo 5.1 · Segurança como propriedade do design
> **Série:** Gerenciamento e Governança de APIs

---

> **Sobre este anexo**
>
> Este anexo aprofunda os padrões de falha de design de segurança introduzidos no Cap 5.1. Para cada padrão, apresentamos o problema conceitual, o mecanismo pelo qual a vulnerabilidade se manifesta, casos documentados que ilustram o impacto real e o design corrigido que resolve a falha na origem. O objetivo é educativo: compreender como essas falhas surgem é o primeiro passo para não repeti-las.

---

## E.1 · IDOR — Insecure Direct Object Reference

### O problema

IDOR — também nomeado pelo OWASP como Broken Object Level Authorization (BOLA) — é a vulnerabilidade mais prevalente em APIs e uma das mais exploradas. Ela ocorre quando um endpoint usa um identificador fornecido pelo consumidor para acessar um objeto diretamente, sem verificar se o consumidor autenticado tem autorização para acessar aquele objeto específico.

A falha não está na ausência de autenticação — o consumidor está autenticado. A falha está na ausência de **autorização por objeto**: o sistema verifica quem você é, mas não verifica se você tem direito àquele objeto específico.

### O mecanismo

```
GET /pedidos/1234
Authorization: Bearer <token_do_usuario_A>
```

O backend recebe a requisição, valida o token e retorna os dados do pedido 1234. O problema: não verifica se o pedido 1234 pertence ao usuário A. Se pertencer ao usuário B, os dados do usuário B são entregues ao usuário A — sem qualquer erro, sem qualquer log de anomalia, sem qualquer sinal de que algo errado aconteceu.

O atacante simplesmente incrementa o identificador:

```
GET /pedidos/1233   → dados de outro usuário
GET /pedidos/1235   → dados de outro usuário
GET /pedidos/1236   → dados de outro usuário
```

### Casos documentados

**Peloton (2021)**
Pesquisadores de segurança descobriram que as APIs de backend da Peloton retornavam dados privados de usuários mesmo quando os perfis estavam configurados como privados. Os endpoints de perfil não verificavam se o solicitante tinha autorização para ver aquele perfil específico — qualquer token válido era suficiente para acessar dados de qualquer usuário. A vulnerabilidade foi reportada responsavelmente e corrigida após divulgação.

**Veículos conectados — APIs de alarmes (Pen Test Partners)**
A empresa Pen Test Partners identificou vulnerabilidades de IDOR em APIs de alarmes inteligentes para veículos que permitiam account takeover completo sobre frotas inteiras. Atacantes podiam alterar senhas e endereços de email de contas, ganhando controle físico sobre os veículos afetados. A estimativa foi de aproximadamente três milhões de veículos de alto valor expostos antes que os fornecedores corrigissem as falhas após divulgação responsável.

**Facebook — deleção de fotos alheias**
Múltiplas vulnerabilidades em APIs do Facebook foram reportadas por pesquisadores de segurança e recompensadas via bug bounty. Em casos documentados, endpoints privilegiados que realizavam operações de modificação não verificavam adequadamente se o usuário autenticado tinha autorização sobre o objeto alvo — permitindo que um usuário deletasse fotos de outro usuário manipulando parâmetros de identificação de objeto.

**Plataforma acadêmica — IDOR em submissões (Amissah & Bentil, IJERT 2024)**
Um estudo publicado no International Journal of Engineering Research & Technology documentou uma vulnerabilidade de IDOR em uma plataforma de publicação acadêmica online. Ao submeter um artigo, os autores recebiam um ID sequencial para acompanhamento. Alterando esse ID por incremento ou decremento, era possível acessar o status de submissão de outros autores, cartas de aceite com comentários de revisores, links para portais de pagamento e certificados de outros pesquisadores. A exploração não exigia conhecimento técnico — apenas a alteração manual de um número na URL.

> *Amissah, E. & Bentil, F. Exposing Insecure Direct Object Reference (IDOR) Vulnerabilities in Academic Publication Platforms: A Case Study. International Journal of Engineering Research & Technology (IJERT), Vol. 13, Issue 08, agosto 2024. Disponível em: [ijert.org/exposing-insecure-direct-object-reference](https://www.ijert.org/exposing-insecure-direct-object-reference-idor-vulnerabilities-in-academic-publication-platforms-a-case-study)*

### O design com a falha

```yaml
# spec OpenAPI com falha de IDOR
paths:
  /pedidos/{pedidoId}:
    get:
      summary: Retorna dados do pedido
      security:
        - oauth2: [pedidos:read]
      # FALHA: não há como declarar verificação de ownership no contrato
      # a verificação precisa existir na implementação — mas o design não a pressupõe
      responses:
        '200':
          description: Dados do pedido
```

O problema não está apenas no código — está no modelo mental: o design pressupõe que autenticação é suficiente para autorizar o acesso. A verificação de que o pedido pertence ao usuário autenticado é tratada como detalhe de implementação — e frequentemente esquecida.

### O design corrigido

A correção tem dois componentes que se reforçam:

**No contrato:** a operação deve ser documentada como dependente do contexto do usuário autenticado — e o schema de resposta deve refletir que o endpoint retorna apenas o pedido do usuário corrente:

```yaml
paths:
  /pedidos/{pedidoId}:
    get:
      summary: Retorna pedido do usuário autenticado
      description: >
        Retorna os dados do pedido identificado por pedidoId,
        apenas se o pedido pertencer ao usuário autenticado.
        Retorna 403 se o pedido existir mas não pertencer ao usuário.
        Retorna 404 se o pedido não existir.
      security:
        - oauth2: [pedidos:read]
```

**Na implementação:** antes de retornar qualquer dado, verificar explicitamente que o objeto pertence ao usuário autenticado:

```
1. Extrair o userId do token JWT
2. Buscar o pedido pelo pedidoId
3. Verificar se pedido.userId == userId do token
4. Se não: retornar 403 (nunca retornar os dados)
5. Se sim: retornar os dados do pedido
```

O uso de UUIDs em vez de IDs sequenciais não elimina a necessidade da verificação — mas elimina a capacidade de enumeração, aumentando a fricção para o atacante.

---

## E.2 · Mass Assignment — Atribuição em Massa

### O problema

Mass assignment ocorre quando um endpoint aceita um objeto de entrada e o aplica diretamente ao modelo de dados sem filtrar quais campos são permitidos. Um consumidor que conhece a estrutura interna do objeto pode enviar campos que não deveria poder alterar — e o sistema os aceita silenciosamente.

### O mecanismo

Considere uma API de atualização de perfil de usuário. O design intuitivo aceita o objeto de usuário e o salva:

```
PATCH /usuarios/eu
Authorization: Bearer <token>
Content-Type: application/json

{
  "nome": "João Silva",
  "email": "joao@exemplo.com",
  "role": "admin",
  "creditosGratuitos": 999999,
  "emailVerificado": true
}
```

Se o backend simplesmente atualiza todos os campos recebidos, um usuário comum pode promover a si mesmo para administrador, adicionar créditos gratuitos à sua conta ou marcar seu email como verificado sem passar pelo processo de verificação.

A falha não exige conhecimento especializado — apenas a inspeção de uma resposta anterior que retornou o objeto completo com todos os seus campos.

### Por que o design cria a falha

A falha surge da combinação de dois problemas de design:

**Exposição excessiva nos endpoints de leitura** — o endpoint `GET /usuarios/eu` retorna o objeto completo, incluindo campos como `role`, `emailVerificado` e `creditosGratuitos`. Isso informa ao atacante quais campos existem.

**Schema permissivo nos endpoints de escrita** — o endpoint `PATCH /usuarios/eu` aceita qualquer campo do objeto de usuário sem especificar quais são permitidos para atualização pelo próprio usuário.

### O design corrigido

**Schema de entrada explícito e restritivo:**

```yaml
components:
  schemas:
    AtualizacaoPerfilRequest:
      type: object
      additionalProperties: false   # rejeita qualquer campo não declarado
      properties:
        nome:
          type: string
          maxLength: 100
        email:
          type: string
          format: email
      # campos como role, emailVerificado, creditosGratuitos
      # simplesmente não existem neste schema
      # se enviados, são rejeitados pelo additionalProperties: false
```

**Separação de schemas de leitura e escrita:**

O schema de resposta (`GET /usuarios/eu`) e o schema de entrada (`PATCH /usuarios/eu`) devem ser schemas distintos. O schema de resposta pode incluir campos de leitura. O schema de entrada inclui apenas os campos que o usuário pode alterar.

**Campos sensíveis em endpoints dedicados:**

Operações privilegiadas — alterar `role`, verificar email, ajustar créditos — devem ter endpoints próprios com autenticação e autorização específicas, não fazem parte da atualização de perfil comum.

---

## E.3 · Exposição excessiva de dados

### O problema

Exposição excessiva de dados ocorre quando um endpoint retorna mais informação do que o consumidor precisa para o caso de uso proposto. A causa mais comum é a prática de retornar o objeto completo do modelo de dados em vez de uma projeção adequada ao caso de uso.

### O mecanismo

Uma API de listagem de usuários para exibição em uma interface retorna:

```json
{
  "id": "uuid-123",
  "nome": "João Silva",
  "email": "joao@exemplo.com",
  "hashSenha": "$2b$10$...",
  "tokenRedefinicaoSenha": "abc123",
  "ultimoIp": "192.168.1.1",
  "flagsInternas": ["beta_user", "admin_override"],
  "dadosCartao": {
    "ultimos4Digitos": "4321",
    "tokenProcessador": "tok_visa_abc"
  }
}
```

A interface de exibição precisa apenas do nome e do email. Os demais campos — hash de senha, tokens de redefinição, IPs, flags internas, dados de cartão — não têm uso legítimo na interface. Mas estão expostos para qualquer consumidor que chame o endpoint.

### Por que o design cria a falha

O padrão mais comum é a geração automática de contratos a partir do modelo de dados interno — o objeto de banco de dados ou o objeto de domínio é diretamente serializado como resposta da API. O que é conveniente para o desenvolvedor é perigoso para o consumidor: a API expõe o modelo de dados interno sem filtragem.

### O design corrigido

**DTOs específicos por caso de uso:**

Em vez de retornar o objeto completo, definir Data Transfer Objects específicos para cada contexto de uso:

```yaml
components:
  schemas:
    UsuarioListagem:         # para listas públicas
      properties:
        id: { type: string }
        nome: { type: string }

    UsuarioPerfil:           # para perfil do próprio usuário
      properties:
        id: { type: string }
        nome: { type: string }
        email: { type: string }
        emailVerificado: { type: boolean }

    UsuarioAdmin:            # apenas em endpoints administrativos com escopo específico
      properties:
        id: { type: string }
        nome: { type: string }
        email: { type: string }
        flagsInternas: { type: array }
```

**A regra geral:** o schema de resposta é projetado a partir das necessidades do consumidor — não a partir da estrutura do modelo de dados interno. Campos que o consumidor não precisa simplesmente não existem no schema de resposta.

---

## E.4 · Endpoints sem paginação — extração de base completa

### O problema

Endpoints que retornam coleções sem limitar o tamanho da resposta permitem que um consumidor — legítimo ou malicioso — extraia a base completa de dados em uma única requisição ou em poucas requisições sequenciais.

### O mecanismo

Um endpoint de listagem sem paginação:

```
GET /usuarios
Authorization: Bearer <token>

→ [{"id": 1, "nome": "..."}, {"id": 2, ...}, ..., {"id": 500000, ...}]
```

Um consumidor com acesso legítimo ao endpoint — um parceiro credenciado, por exemplo — pode extrair todos os 500.000 usuários em uma única chamada. Um consumidor malicioso que obteve um token pode fazer o mesmo.

O problema não é apenas de segurança — é também de estabilidade: um endpoint sem paginação é um vetor de ataque de negação de serviço, pois uma única requisição pode consumir recursos significativos do servidor.

### Variações do problema

**Paginação sem limite máximo de page size:** o endpoint aceita `?limit=100000`, efetivamente anulando a paginação.

**Filtros que retornam conjuntos grandes:** um endpoint filtrado por data que permite intervalos ilimitados — `?de=2000-01-01&ate=2030-12-31` — pode retornar toda a base.

**Endpoint de exportação sem controle:** funcionalidades de "exportar tudo" sem limite ou aprovação criam vetores de extração legítimos que podem ser explorados.

### O design corrigido

**Paginação obrigatória com limite máximo:**

```yaml
paths:
  /usuarios:
    get:
      parameters:
        - name: pagina
          in: query
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: tamanho
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 100    # limite máximo por página — nunca maior
            default: 20
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  dados:
                    type: array
                    items:
                      $ref: '#/components/schemas/UsuarioListagem'
                  paginacao:
                    type: object
                    properties:
                      pagina: { type: integer }
                      tamanho: { type: integer }
                      total: { type: integer }
                      temProxima: { type: boolean }
```

**Escopos diferenciados para exportação:** funcionalidades de exportação em volume devem exigir um escopo específico — `usuarios:export` — que é concedido apenas com justificativa e revisão, diferente do escopo de leitura regular `usuarios:read`.

---

## E.5 · Erros que vazam informação interna

### O problema

Mensagens de erro que incluem stack traces, nomes de tabelas de banco de dados, queries SQL, nomes de classes internas ou detalhes de configuração fornecem a um atacante informação valiosa sobre a estrutura interna do sistema — facilitando ataques direcionados.

### O mecanismo

Um endpoint que encontra um erro retorna:

```json
{
  "erro": "Internal Server Error",
  "detalhe": "java.sql.SQLSyntaxErrorException: You have an error in your SQL syntax near 'WHERE user_id = 'abc'' at line 1",
  "stack": "com.empresa.api.UsuarioRepository.findById(UsuarioRepository.java:45)\n  at com.empresa.api.UsuarioService.getUsuario...",
  "query": "SELECT * FROM tb_usuarios WHERE user_id = 'abc'"
}
```

Esse erro revela: a linguagem de implementação (Java), o nome da tabela de banco de dados (`tb_usuarios`), a estrutura da query SQL, e que o sistema pode ser vulnerável a SQL injection (o valor `'abc'` foi incluído diretamente na query).

### O design corrigido

**Schema de erro padronizado sem informação técnica:**

O RFC 7807 — *Problem Details for HTTP APIs* — define um formato padrão para erros que é informativo para o consumidor sem expor detalhes internos:

```yaml
components:
  schemas:
    ErroAPI:
      type: object
      required: [type, title, status]
      properties:
        type:
          type: string
          format: uri
          description: URI que identifica o tipo do erro
          example: "https://api.empresa.com/erros/recurso-nao-encontrado"
        title:
          type: string
          description: Descrição legível do tipo do erro
          example: "Recurso não encontrado"
        status:
          type: integer
          description: Código HTTP
          example: 404
        detail:
          type: string
          description: Explicação específica desta instância do erro
          example: "O usuário solicitado não existe ou não está acessível"
        instance:
          type: string
          format: uri
          description: URI que identifica esta instância específica do erro
          example: "/solicitacoes/xyz-123"
      additionalProperties: false   # nenhum campo adicional — especialmente sem stack trace
```

**Separação entre log interno e resposta externa:** o stack trace completo é registrado nos logs internos com um correlation ID — não na resposta. O consumidor recebe o correlation ID para referenciar ao suporte, sem que o detalhe técnico seja exposto.

---

## E.6 · Ausência de rate limiting por consumidor

### O problema

Endpoints sem rate limiting específico por consumidor — ou com rate limiting apenas global — permitem que um único consumidor consuma desproporcionalmente a capacidade do serviço, prejudicando outros consumidores ou possibilitando ataques de enumeração e credential stuffing.

### O mecanismo

**Cenário 1 — Enumeração**

Um endpoint de verificação de disponibilidade de email sem rate limiting:

```
GET /verificar-email?email=usuario1@gmail.com  → {"disponivel": false}
GET /verificar-email?email=usuario2@gmail.com  → {"disponivel": true}
GET /verificar-email?email=usuario3@gmail.com  → {"disponivel": false}
```

Um atacante pode enriquecer bases de dados de emails verificando a existência de milhões de endereços sem qualquer limitação.

**Cenário 2 — Credential stuffing**

Um endpoint de autenticação sem rate limiting por IP ou por conta permite que um atacante teste listas de senhas comprometidas contra qualquer conta sem limitação de velocidade.

**Cenário 3 — Consumo predatório de recursos**

Um consumidor que chama um endpoint computacionalmente caro — geração de relatórios, processamento de imagens, consultas complexas — sem limitação pode consumir toda a capacidade do serviço, negando serviço aos demais consumidores.

### O design corrigido

**Declaração de rate limits no contrato:**

```yaml
paths:
  /verificar-email:
    get:
      x-rate-limit:
        por-ip: 10/minuto
        por-token: 100/hora
      responses:
        '429':
          description: Rate limit excedido
          headers:
            Retry-After:
              schema:
                type: integer
              description: Segundos até o próximo request ser aceito
            X-RateLimit-Limit:
              schema:
                type: integer
            X-RateLimit-Remaining:
              schema:
                type: integer
```

**Rate limiting diferenciado por operação:**

Operações sensíveis — autenticação, verificação de email, recuperação de senha — devem ter limites mais restritivos do que operações de leitura regulares. O design deve especificar esses limites por operação, não apenas como configuração global do gateway.

---

## Resumo — padrões de falha e suas origens de design

| Padrão de falha | Origem no design | Princípio violado |
|---|---|---|
| IDOR / BOLA | Autenticação sem autorização por objeto | Least privilege · Fail secure |
| Mass assignment | Schema de entrada sem restrição de campos | Least privilege · Minimização de exposição |
| Exposição excessiva | Retornar objeto completo em vez de projeção | Minimização de exposição |
| Sem paginação | Coleções sem limite de tamanho | Defense in depth · DoS prevention |
| Erros com stack trace | Mensagens de erro sem schema definido | Minimização de exposição · Information disclosure |
| Sem rate limiting | Ausência de limites declarados por operação | Defense in depth · DoS prevention |

---

## Referências

| Fonte | Referência completa |
|---|---|
| **OWASP API Security Top 10 (2023)** | OWASP Foundation. *OWASP API Security Top 10 2023*. Disponível em: [owasp.org/www-project-api-security](https://owasp.org/www-project-api-security/) |
| **Amissah & Bentil (2024)** | Amissah, E. & Bentil, F. Exposing Insecure Direct Object Reference (IDOR) Vulnerabilities in Academic Publication Platforms: A Case Study. *International Journal of Engineering Research & Technology (IJERT)*, Vol. 13, Issue 08, agosto 2024. Disponível em: [ijert.org/exposing-insecure-direct-object-reference](https://www.ijert.org/exposing-insecure-direct-object-reference-idor-vulnerabilities-in-academic-publication-platforms-a-case-study) |
| **RFC 7807 — Problem Details** | Nottingham, M. & Wilde, E. *Problem Details for HTTP APIs*. RFC 7807, março 2016. Disponível em: [datatracker.ietf.org/doc/html/rfc7807](https://datatracker.ietf.org/doc/html/rfc7807) |
| **Pen Test Partners — Connected Vehicle Research** | Pen Test Partners. *Connected Vehicle Security Research*. Disponível em: [pentestpartners.com/security-blog](https://www.pentestpartners.com/security-blog/) |

---

*Série: Gerenciamento e Governança de APIs · Módulo 5 · Anexo E*