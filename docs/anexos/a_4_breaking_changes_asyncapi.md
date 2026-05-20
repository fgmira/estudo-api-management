# Anexo A.4 · Breaking changes em AsyncAPI

> **Referência:** Capítulo 2.5 · Versionamento e gestão de breaking changes
> **Série:** Gerenciamento e Governança de APIs

---

## Estratégias de compatibilidade de schema

Antes de classificar mudanças, a organização precisa definir sua estratégia de compatibilidade — pois ela determina o que é ou não é breaking:

| Estratégia | Definição | Quando usar |
|---|---|---|
| **Backward** | Consumidores antigos conseguem ler eventos novos | Adição de campos opcionais |
| **Forward** | Consumidores novos conseguem ler eventos antigos | Remoção de campos não essenciais |
| **Full** | Backward + Forward — compatibilidade nos dois sentidos | Máxima segurança — mais restritivo |
| **None** | Sem garantia de compatibilidade — nova versão do schema | Breaking change explícita |

---

## Non-breaking changes (estratégia Backward)

| Mudança | Notas |
|---|---|
| Adicionar campo opcional ao payload | Consumidores antigos ignoram o campo desconhecido |
| Adicionar novo canal | Consumidores que não consomem o canal não são afetados |
| Adicionar novo tipo de evento | Consumidores que não tratam o novo tipo não são afetados |
| Aumentar tamanho máximo de campo | Verificar consumidores com validação de tamanho máximo |

---

## Breaking changes

| Mudança | Impacto |
|---|---|
| Remover campo existente | Consumidores que dependem do campo falham no parse ou recebem null |
| Tornar campo opcional em obrigatório | Produtores que não enviam o campo falham na validação do schema registry |
| Mudar tipo de campo | Consumidores que assumem o tipo antigo falham no parse |
| Renomear campo | Consumidores que referenciam o nome antigo recebem null |
| Mudar nome do canal/tópico | Consumidores inscritos no canal antigo param de receber eventos |
| Mudar chave de particionamento | Consumidores que dependem da ordem por chave recebem eventos fora de ordem |
| Mudar formato de serialização (JSON para Avro) | Consumidores com desserialização diferente falham |

---

## Considerações específicas de sistemas event-driven

**Ordenamento**
Mudanças que afetam a chave de particionamento no Kafka podem reordenar eventos de forma que quebra consumidores que dependem de ordenamento por entidade.

**Idempotência**
Mudanças no identificador único do evento (`event_id`) afetam consumidores que usam esse campo para garantir idempotência no processamento.

**Dead Letter Queue**
Consumidores que falham ao processar eventos de novo schema podem gerar acúmulo em DLQ. Monitorar após qualquer mudança de schema.

**Replay de eventos**
Se o sistema suporta replay de eventos históricos, mudanças de schema podem fazer com que eventos antigos — com schema anterior — falhem ao ser processados por consumidores que só conhecem o schema novo.

---

## Ferramentas recomendadas

- **Confluent Schema Registry** — registry centralizado com enforcement de compatibilidade para Kafka
- **AWS Glue Schema Registry** — alternativa para ambientes AWS
- **AsyncAPI Studio** — validação e visualização de specs AsyncAPI
- **AsyncAPI CLI** — validação de specs AsyncAPI via linha de comando

---

*Série: Gerenciamento e Governança de APIs · Anexo A.4*