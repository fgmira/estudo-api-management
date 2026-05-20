# Anexo A.2 · Breaking changes em APIs GraphQL

> **Referência:** Capítulo 2.5 · Versionamento e gestão de breaking changes
> **Série:** Gerenciamento e Governança de APIs

---

## Non-breaking changes

| Mudança | Notas |
|---|---|
| Adicionar novo tipo ao schema | Consumidores que não consultam o novo tipo não são afetados |
| Adicionar novo campo opcional em tipo existente | Consumidores que não consultam o campo não são afetados |
| Adicionar novo argumento opcional em campo existente | Consumidores que não passam o argumento recebem o comportamento padrão |
| Adicionar novo valor em enum | Menos arriscado que REST — consumidores GraphQL tendem a tratar enums de forma mais flexível |
| Deprecar campo sem removê-lo | Consumidores continuam funcionando — apenas recebem aviso de deprecação |
| Adicionar nova interface ou union | Consumidores existentes não são afetados |

---

## Breaking changes

| Mudança | Impacto |
|---|---|
| Remover campo existente | Queries que incluem o campo falham com erro de schema |
| Remover tipo existente | Queries que referenciam o tipo falham |
| Remover valor de enum | Queries que filtram pelo valor removido falham |
| Tornar campo nullable em non-nullable | Consumidores que tratam null para o campo quebram |
| Tornar campo non-nullable em nullable | Consumidores que assumem valor sempre presente podem quebrar |
| Mudar tipo de campo | Queries que assumem o tipo antigo falham no parse |
| Remover argumento obrigatório | Queries que passam o argumento podem falhar dependendo da implementação |
| Tornar argumento opcional em obrigatório | Queries que não passam o argumento falham |
| Mudar comportamento de resolver | Quebra semântica — não detectada por validação de schema |

---

## Potencialmente breaking

| Mudança | Risco |
|---|---|
| Mudar tipo nullable para union | Consumidores que assumem tipo único podem quebrar |
| Adicionar argumento obrigatório com default | Comportamento muda para consumidores que não passam o argumento |
| Mudar profundidade máxima permitida | Consumidores com queries complexas começam a receber erros |

---

## Ferramentas recomendadas

- **GraphQL Inspector** — detecta breaking changes entre versões de schema automaticamente
- **Apollo Studio** — schema registry com alertas de breaking changes
- **Rover CLI** — validação de schema via linha de comando

---

*Série: Gerenciamento e Governança de APIs · Anexo A.2*