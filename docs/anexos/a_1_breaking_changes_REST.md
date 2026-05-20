# Anexo A.1 · Breaking changes em APIs REST

> **Referência:** Capítulo 2.5 · Versionamento e gestão de breaking changes
> **Série:** Gerenciamento e Governança de APIs

---

## Non-breaking changes (backward compatible)

| Mudança | Notas |
|---|---|
| Adicionar novo endpoint | Consumidores existentes não são afetados |
| Adicionar campo opcional no response | Consumidores que ignoram campos desconhecidos não são afetados |
| Adicionar campo opcional no request | Consumidores que não enviam o campo continuam funcionando |
| Adicionar novo código de status de sucesso | Verificar impacto em consumidores que não tratam o novo código |
| Melhorar mensagem de erro sem mudar código | O código de erro é o contrato — a mensagem é informativa |
| Adicionar novo header de resposta | Consumidores que ignoram headers desconhecidos não são afetados |
| Adicionar novo query parameter opcional | Consumidores que não enviam o parâmetro recebem comportamento padrão |
| Expandir limite de rate limiting | Consumidores nunca dependem de um limite mais restritivo |

---

## Breaking changes (backward incompatible)

| Mudança | Impacto |
|---|---|
| Remover endpoint existente | Consumidores que chamam o endpoint recebem 404 |
| Remover campo do response | Consumidores que dependem do campo recebem null ou erro |
| Tornar campo opcional do request em obrigatório | Consumidores que não enviam o campo recebem erro de validação |
| Mudar tipo de campo (string para integer) | Consumidores que assumem o tipo original quebram no parse |
| Mudar nome de campo | Consumidores que referenciam o nome antigo recebem null |
| Mudar código de status de resposta | Consumidores que tratam o código antigo quebram no switch/case |
| Mudar estrutura de URL | Consumidores com URLs hardcoded recebem 404 |
| Remover valor de enum | Consumidores que enviavam ou esperavam o valor removido quebram |
| Restringir validações aceitas | Consumidores que enviavam valores antes aceitos recebem erros |
| Mudar comportamento de autenticação | Consumidores com o esquema antigo perdem acesso |

---

## Potencialmente breaking (avaliar caso a caso)

| Mudança | Risco |
|---|---|
| Adicionar novo valor em enum | Quebra consumidores que tratam enum como fechado com switch/case sem default |
| Mudar formato de data/hora | Quebra consumidores com parsers que não suportam o novo formato |
| Mudar tamanho padrão de paginação | Quebra consumidores que assumem número fixo de itens por página |
| Mudar campo sempre presente para opcional | Quebra consumidores que não tratam ausência do campo |
| Adicionar validação mais restritiva em campo existente | Quebra consumidores que enviavam valores que agora são inválidos |
| Mudar ordenação padrão de resultados | Quebra consumidores que dependem da ordem sem especificar sort |

---

*Série: Gerenciamento e Governança de APIs · Anexo A.1*