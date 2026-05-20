# Anexo B · Template de plano de depreciação

> **Referência:** Capítulo 2.6 · Depreciação e sunset controlado
> **Série:** Gerenciamento e Governança de APIs

---

> **Nota sobre este template**
>
> Este documento é uma sugestão de ponto de partida — não um padrão prescritivo. Cada organização deve adaptá-lo à sua realidade, cultura, porte e estrutura de governança. Seções podem ser simplificadas, expandidas ou removidas conforme o contexto. O objetivo é garantir que os elementos essenciais sejam considerados antes de qualquer comunicação ser feita aos consumidores.

---

## Seção 1 · Identificação

| Campo | Valor |
|---|---|
| API / Versão sendo depreciada | |
| Versão alternativa disponível | |
| Data de elaboração do plano | |
| Responsável pelo plano (API Owner) | |
| Aprovado por | |
| Data de aprovação | |
| Change Record no CMDB | |

---

## Seção 2 · Escopo e justificativa

**Descrição do que está sendo depreciado:**

**Justificativa da depreciação** (sinais que sustentam a decisão — produto, técnicos, estratégicos):

**Alternativa disponível e resumo das diferenças principais:**

**Impacto estimado** (número de consumidores ativos, volume de chamadas na versão sendo depreciada):

---

## Seção 3 · Cronograma

| Marco | Data prevista | Responsável |
|---|---|---|
| Aprovação do plano | | |
| Verificação contratual concluída | | |
| Comunicação prévia a consumidores críticos | | |
| Anúncio formal público | | |
| Início do período de coexistência | | |
| Ativação de headers de depreciação (RFC 8594) | | |
| Bloqueio de novos onboardings na versão antiga | | |
| Primeiro lembrete periódico | | |
| Outreach proativo para não migrados (60-90 dias antes do sunset) | | |
| Sunset | | |
| Registro de encerramento no CMDB | | |

---

## Seção 4 · Consumidores identificados

### Consumidores críticos

| Consumidor | Contato | Status de migração | Plano individual | Contrato ativo? |
|---|---|---|---|---|
| | | | | |

### Consumidores relevantes

| Consumidor | Contato | Status de migração |
|---|---|---|
| | | |

### Consumidores padrão

Número total identificado via dados de uso do gateway: ___

Canal de comunicação utilizado: ___

---

## Seção 5 · Verificação contratual

| Pergunta | Resposta |
|---|---|
| Existem contratos comerciais ativos com consumidores que usam a versão sendo depreciada? | |
| Algum contrato inclui cláusulas de prazo mínimo de aviso superiores à política padrão? | |
| Os prazos do cronograma são compatíveis com todas as obrigações contratuais? | |
| Há consumidores que exigem tratamento diferenciado por obrigação contratual? | |
| Validação jurídica concluída em: | |
| Validado por: | |

> Se a deprecation policy foi previamente validada pela área jurídica e este plano segue integralmente a política, a verificação contratual se limita a confirmar que não há contratos com cláusulas mais restritivas do que a política padrão.

---

## Seção 6 · Plano de comunicação

| Fase | Canal | Conteúdo resumido | Data | Responsável |
|---|---|---|---|---|
| Comunicação prévia — críticos | | | | |
| Anúncio formal | | | | |
| Ativação de headers RFC 8594 | Gateway | Automático | | |
| Lembrete 1 | | | | |
| Lembrete 2 | | | | |
| Lembrete final (30 dias antes) | | | | |
| Outreach proativo não migrados | | | | |

---

## Seção 7 · Plano de suporte à migração

| Recurso | Disponível para | Data de disponibilização | Responsável |
|---|---|---|---|
| Guia de migração | Todos | | |
| Sandbox estendido | Críticos e relevantes | | |
| Office hours técnicas | Críticos e relevantes | | |
| SDKs / ferramentas de migração | Todos (se aplicável) | | |
| Suporte dedicado | Críticos | | |

---

## Seção 8 · Critérios e processo de exceção

**Critérios que justificam extensão de prazo:**

**Prazo máximo de extensão permitido pela política:**

**Quem pode solicitar extensão:**

**Quem aprova:**

**Como a extensão é registrada:**

---

## Seção 9 · Execução do sunset

**Modo de encerramento selecionado** (imediato / degradação progressiva / redirecionamento temporário):

**Justificativa da escolha:**

**Configuração técnica necessária no gateway:**

**Plano de rollback** (o que fazer se o encerramento gerar impacto inesperado):

**Threshold de uso abaixo do qual o sunset pode ser executado:**

---

## Seção 10 · Registro pós-sunset

| Campo | Valor |
|---|---|
| Data efetiva do encerramento | |
| Consumidores ativos no momento do encerramento | |
| Evidências de notificação preservadas em | |
| CMDB atualizado em | |
| Catálogo atualizado em | |
| Responsável pelo registro | |

---

## Checklist de aprovação do plano

- [ ] Alternativa disponível e estável confirmada
- [ ] Consumidores identificados via dados de uso do gateway
- [ ] Verificação contratual concluída
- [ ] Cronograma alinhado com a deprecation policy
- [ ] Plano de comunicação definido para todos os segmentos
- [ ] Recursos de suporte à migração planejados
- [ ] Processo de exceção documentado
- [ ] Plano de rollback para o sunset definido
- [ ] Change Record criado no CMDB
- [ ] Aprovação formal registrada

---

## Checklist de execução do sunset

- [ ] Todos os consumidores críticos confirmaram migração ou receberam extensão formal
- [ ] Uso da versão abaixo do threshold definido
- [ ] Área jurídica confirmou que não há impedimentos contratuais para o encerramento na data prevista
- [ ] Plano de rollback revisado e equipe de plantão definida
- [ ] Gateway configurado para execução do encerramento
- [ ] CMDB e catálogo prontos para atualização pós-encerramento

---

*Série: Gerenciamento e Governança de APIs · Anexo B*