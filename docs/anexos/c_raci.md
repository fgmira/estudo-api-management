# Anexo C · Modelo RACI sugerido para governança de APIs

> **Referência:** Capítulo 3.2 · Papéis e responsabilidades
> **Série:** Gerenciamento e Governança de APIs

---

> **Nota sobre este modelo**
>
> Este RACI é uma sugestão de ponto de partida — não uma prescrição. A pesquisa de Duhamel et al. (2022) confirmou empiricamente que a alocação correta de papéis depende de fatores de contingência organizacional específicos: formalização de procedimentos, centralização, complexidade e tamanho do departamento de TI. Adapte este modelo à realidade da sua organização antes de adotá-lo. Em organizações menores, múltiplos papéis são frequentemente exercidos pela mesma pessoa — o que é normal e esperado.

---

## Legenda

| Sigla | Significado |
|---|---|
| **R** | Responsible — executa a atividade |
| **A** | Accountable — responsável último pelo resultado · aprova |
| **C** | Consulted — consultado antes da decisão |
| **I** | Informed — informado após a decisão |

**Regra fundamental:** cada atividade deve ter exatamente um **A** — nunca zero, nunca dois.

---

## Decisões de ciclo de vida de APIs

| Decisão | API Owner | API PM | API Steward | CoE | Time Plataforma | Time Produto | Consumidor |
|---|---|---|---|---|---|---|---|
| Iniciar concepção de nova API | A | C | I | C | I | R | I |
| Verificar duplicação no catálogo | C | C | R | A | I | C | — |
| Aprovar charter de nova API | A | C | C | A | — | I | — |
| Aprovar spec OpenAPI para desenvolvimento | A | C | R | A | I | C | C |
| Autorizar publicação em produção | A | C | R | A | I | R | — |
| Aprovar breaking change | A | C | C | A | I | R | C |
| Autorizar nova versão major | A | A | C | A | I | C | C |
| Iniciar processo de depreciação | A | C | C | A | I | I | I |
| Conceder extensão de prazo de depreciação | A | C | — | A | — | — | C |
| Executar sunset | A | I | I | A | R | I | I |

---

## Decisões de políticas e padrões

| Decisão | API Owner | API PM | API Steward | CoE | Time Plataforma | Time Produto | Consumidor |
|---|---|---|---|---|---|---|---|
| Definir e publicar style guide | I | C | C | A | C | C | — |
| Atualizar style guide | C | C | C | A | C | C | — |
| Definir critérios de qualidade de gates | I | C | C | A | C | C | — |
| Definir política de depreciação | C | C | C | A | I | C | C |
| Conceder exceção a política | C | C | R | A | I | C | — |
| Revisar política após padrão de exceções | C | C | C | A | C | C | — |

---

## Decisões de plataforma e infraestrutura

| Decisão | API Owner | API PM | API Steward | CoE | Time Plataforma | Time Produto | Consumidor |
|---|---|---|---|---|---|---|---|
| Escolher e evoluir API gateway | C | C | C | C | A | I | — |
| Configurar políticas no gateway em produção | I | — | C | A | R | I | — |
| Manter e evoluir catálogo de APIs | I | I | C | A | R | I | C |
| Operar portal de desenvolvedores | I | C | C | C | A | I | I |
| Definir padrões de observabilidade | C | — | C | A | R | C | — |

---

## Decisões de conflito e escalação

| Decisão | API Owner | API PM | API Steward | CoE | Time Plataforma | Time Produto | Consumidor |
|---|---|---|---|---|---|---|---|
| Resolver conflito técnico entre times | C | C | R | A | C | C | — |
| Escalar conflito não resolvido | R | R | R | A | C | C | — |
| Arbitrar exceção de alto impacto | C | C | C | R | — | C | — |
| Revisar decisão de bloqueio de lançamento | A | C | C | A | I | C | — |

---

## Notas de aplicação

**Sobre papéis duplos no A:**
Algumas decisões têm dois **A** na tabela — API Owner e CoE. Isso reflete situações onde a accountability é genuinamente compartilhada: o API Owner é accountable pelo valor da API, o CoE é accountable pela conformidade com as políticas. Na prática, quando há conflito entre os dois, o CoE tem autoridade final em questões de política e o API Owner tem autoridade final em questões de produto.

**Sobre o consumidor como ator:**
O consumidor aparece com **C** ou **I** em várias decisões — nunca com **R** ou **A** para atividades internas. Isso reflete sua posição como ator externo que deve ser consultado e informado, mas não tem responsabilidade de execução nas decisões internas de governança.

**Sobre organizações em estágio inicial:**
Em organizações onde CoE e API Steward não existem formalmente, as responsabilidades **A** do CoE recaem sobre o arquiteto ou tech lead responsável pelo programa de APIs. As responsabilidades **R** do API Steward recaem sobre o API Owner ou um engenheiro sênior designado.

---

*Série: Gerenciamento e Governança de APIs · Anexo C*