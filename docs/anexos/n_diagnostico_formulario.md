# Módulo 7 · Maturidade em Governança de APIs
## Anexo N · Questionário de autodiagnóstico

> **Tipo:** Referência — instrumento de diagnóstico
> **Relacionado a:** Cap 7.3 · Cap 7.12

---

## Como usar este questionário

Este questionário traduz o framework de maturidade do Cap 7.3 em perguntas com respostas baseadas em evidências. Ele complementa o Cap 7.12 — use-o como ponto de partida para a coleta de evidências, não como substituto do processo de diagnóstico.

**Princípio de uso:** para cada pergunta, selecione a resposta que descreve o que acontece na maior parte do portfólio, na maior parte do tempo. Se a resposta correta para APIs críticas é diferente da resposta para APIs menores ou internas, isso é um sinal de inconsistência — e a resposta correta é a que descreve o cenário mais comum, não o melhor caso.

**Como determinar o nível:** para cada dimensão, o nível é determinado pelo conjunto de respostas. Use a seguinte orientação:
- Se a maioria das respostas aponta para um nível, esse é o nível atual
- Se há respostas espalhadas entre dois níveis adjacentes, a organização está em transição
- Nunca arredonde para cima — use o nível mais baixo consistente

**Quem deve responder:** aplique o questionário com múltiplos grupos (Cap 7.12) e compare as respostas. Discrepâncias entre grupos são tão informativas quanto as respostas em si.

---

## D1 · Estratégia e Liderança

**D1.Q1 — Estrutura de governança**

Qual é a estrutura formal responsável pela governança de APIs na organização?

| Nível | Resposta |
|---|---|
| 1 | Não existe estrutura formal. A responsabilidade está distribuída entre os times que desenvolvem APIs |
| 2 | Existe um grupo informal ou uma pessoa que advoga por governança, sem mandato formal nem recursos dedicados |
| 3 | Existe um CoE formalmente constituído com escopo definido, responsabilidades claras e pelo menos uma pessoa dedicada em tempo integral |
| 4 | O CoE tem autoridade formal para enforçar políticas, recursos adequados e mecanismos de escalação quando times não estão em compliance |
| 5 | A governança de APIs está alinhada explicitamente com objetivos estratégicos de negócio e tem visibilidade e suporte de liderança executiva de primeiro escalão |

---

**D1.Q2 — Enforcement**

Quando uma API não está em conformidade com as políticas definidas, o que acontece?

| Nível | Resposta |
|---|---|
| 1 | Nada acontece — não há processo de verificação de conformidade |
| 2 | A não-conformidade pode ser reportada, mas não há consequências formais. Times decidem se corrigem ou não |
| 3 | A publicação é bloqueada até que a conformidade seja atingida, independentemente de qual time está envolvido |
| 4 | Além do bloqueio, há análise de padrões — APIs repetidamente não-conformes geram ação proativa do CoE com o time responsável |
| 5 | O enforcement é incorporado ao fluxo de trabalho de forma que a conformidade é o caminho natural, não um obstáculo adicional |

---

**D1.Q3 — Recursos**

Como a governança de APIs é financiada e staffed?

| Nível | Resposta |
|---|---|
| 1 | Não há orçamento nem pessoas dedicadas |
| 2 | Pessoas se dedicam à governança como responsabilidade parcial, junto com outras funções |
| 3 | Há pelo menos uma pessoa dedicada em tempo integral, com orçamento específico para ferramentas e iniciativas |
| 4 | O time de governança tem dimensionamento adequado ao tamanho do portfólio, com capacidade de atender demandas sem backlog crônico |
| 5 | O investimento em governança é revisado periodicamente em função do crescimento do portfólio e dos resultados entregues |

---

**D1.Q4 — Visibilidade executiva**

Como a liderança executiva se relaciona com o programa de governança de APIs?

| Nível | Resposta |
|---|---|
| 1 | A liderança não tem visibilidade sobre o portfólio de APIs nem sobre o programa de governança |
| 2 | Há um patrocinador executivo, mas o programa não tem visibilidade regular na agenda de liderança |
| 3 | O estado do portfólio é reportado regularmente à liderança com métricas definidas |
| 4 | Decisões estratégicas sobre o portfólio envolvem liderança executiva com base em dados de governança |
| 5 | A liderança articula ativamente o valor do portfólio de APIs como ativo estratégico em contextos de negócio |

---

## D2 · Design e Padrões

**D2.Q1 — Style guide**

Qual é o estado do style guide de APIs na organização?

| Nível | Resposta |
|---|---|
| 1 | Não existe style guide formal. Cada time usa suas próprias convenções |
| 2 | Existe um style guide documentado, mas a adesão é voluntária e inconsistente |
| 3 | O style guide é enforçado automaticamente via lint integrado ao CI/CD, com taxa de conformidade monitorada |
| 4 | O style guide é revisado periodicamente com base em dados de uso, feedback de consumidores e análise de problemas de integração |
| 5 | Os padrões promovem interoperabilidade com o ecossistema externo e são reconhecidos como diferencial pela comunidade de desenvolvedores |

---

**D2.Q2 — Consistência do portfólio**

Se um consumidor integrar dez APIs diferentes do portfólio, o que ele encontrará?

| Nível | Resposta |
|---|---|
| 1 | Cada API parece ter sido criada por uma organização diferente — nomenclatura, estrutura de erros e estratégia de versionamento variam significativamente |
| 2 | Há alguma consistência nas APIs mais recentes, mas APIs mais antigas seguem padrões diferentes |
| 3 | A maioria das APIs segue os mesmos padrões estruturais — nomenclatura, paginação, erros, autenticação |
| 4 | Além de consistência estrutural, os conceitos de negócio são representados de forma consistente — o mesmo recurso tem o mesmo nome e os mesmos atributos em APIs diferentes |
| 5 | As APIs são projetadas para composição — um consumidor consegue combinar múltiplas APIs com mínimo de mapeamento e reconciliação |

---

**D2.Q3 — Qualidade do contrato**

Um desenvolvedor externo consegue integrar uma API do portfólio usando apenas a especificação?

| Nível | Resposta |
|---|---|
| 1 | Não — a integração requer acesso direto ao time que criou a API ou leitura do código-fonte |
| 2 | Parcialmente — a especificação existe mas tem lacunas que requerem esclarecimentos frequentes |
| 3 | Na maioria dos casos — a especificação é suficiente para integração básica, mas casos de borda requerem suporte |
| 4 | Sim — a especificação documenta comportamento esperado, edge cases e comportamento de erro com precisão suficiente para integração sem suporte |
| 5 | A especificação é processável por máquina — agentes de IA conseguem entender e usar a API baseados exclusivamente no contrato |

---

## D3 · Ciclo de Vida

**D3.Q1 — Processo de publicação**

O que acontece antes de uma nova API ser publicada para consumo?

| Nível | Resposta |
|---|---|
| 1 | Não há processo formal — a API fica disponível quando está pronta do ponto de vista do time que a desenvolveu |
| 2 | Há um processo documentado, mas é contornado quando há pressão de prazo |
| 3 | Há gates automatizados que a API precisa passar antes de ser publicada, independentemente da pressão de prazo |
| 4 | Além dos gates técnicos, há revisão de impacto — como a API se posiciona no portfólio, se resolve um problema que já foi resolvido de outra forma, quem serão os consumidores |
| 5 | A publicação é o resultado de um processo de product discovery que inclui validação com consumidores pretendidos antes do desenvolvimento |

---

**D3.Q2 — Inventário do portfólio**

A organização sabe com precisão quantas APIs estão em produção e seu estado atual?

| Nível | Resposta |
|---|---|
| 1 | Não — o inventário é desconhecido ou significativamente desatualizado |
| 2 | Há um inventário, mas está frequentemente desatualizado e pode não refletir o estado real |
| 3 | O inventário é mantido atualizado automaticamente — APIs aparecem e mudam de estado sem intervenção manual |
| 4 | O inventário inclui dados de uso, dependências e métricas de qualidade por API |
| 5 | O portfólio é gerenciado como um ativo com visão estratégica — quais APIs têm roadmap ativo, quais são candidatas a consolidação |

---

**D3.Q3 — Depreciação e sunset**

Como a organização retira APIs de operação?

| Nível | Resposta |
|---|---|
| 1 | APIs param de funcionar sem comunicação prévia, ou nunca são desativadas mesmo sem uso |
| 2 | Há intenção de comunicar depreciações, mas o processo não é seguido consistentemente |
| 3 | Depreciações seguem um processo definido: anúncio com antecedência mínima, período de coexistência de versões, data de sunset comunicada |
| 4 | Decisões de deprecação são orientadas por dados de uso real — a organização sabe quantos consumidores ativos tem e os notifica diretamente |
| 5 | O sunset de uma API é tratado como evento de produto — com comunicação estruturada, suporte ativo à migração e acompanhamento até que todos os consumidores tenham migrado |

---

## D4 · Segurança

**D4.Q1 — Autenticação e autorização**

Qual é o padrão de autenticação no portfólio?

| Nível | Resposta |
|---|---|
| 1 | Varia por API — alguns usam API Keys, outros tokens proprietários, alguns não têm autenticação |
| 2 | Há padrões definidos, mas a verificação de conformidade é manual e periódica |
| 3 | OAuth 2.0 com escopos bem definidos é o padrão enforçado automaticamente. Exceções são documentadas e justificadas |
| 4 | Além do padrão de autenticação, há gestão de identidade agêntica — agentes têm identidades próprias com escopos específicos |
| 5 | Zero Trust é o modelo operacional — toda requisição é autenticada e autorizada explicitamente, independentemente da origem |

---

**D4.Q2 — Gestão de credenciais**

Como as credenciais de acesso às APIs são gerenciadas?

| Nível | Resposta |
|---|---|
| 1 | Credenciais de longa duração estão distribuídas em configurações de sistemas sem inventário centralizado |
| 2 | Há alguma centralização, mas credenciais antigas não são auditadas nem rotacionadas sistematicamente |
| 3 | Credenciais têm prazo de expiração, são rotacionadas automaticamente e há inventário centralizado |
| 4 | O inventário de credenciais inclui contexto — qual sistema usa, para qual propósito, quem é o responsável |
| 5 | Credenciais estáticas são exceção, não regra — o modelo padrão usa tokens efêmeros e identidades federadas |

---

**D4.Q3 — Descoberta de vulnerabilidades**

Como a organização descobre vulnerabilidades de segurança no portfólio?

| Nível | Resposta |
|---|---|
| 1 | Principalmente por incidentes — usuários reportam problemas ou ataques revelam vulnerabilidades |
| 2 | Por auditoria periódica — revisões de segurança são feitas em ciclos, mas o portfólio pode ter vulnerabilidades entre auditorias |
| 3 | Por verificação contínua no CI/CD — análise estática e verificações de segurança rodam automaticamente a cada mudança |
| 4 | Por monitoramento de comportamento — anomalias no uso das APIs são detectadas e investigadas proativamente |
| 5 | Por análise preditiva — padrões históricos de vulnerabilidade informam onde a organização concentra verificação proativa |

---

## D5 · Observabilidade

**D5.Q1 — Visibilidade operacional**

O que a organização sabe sobre o comportamento atual do portfólio?

| Nível | Resposta |
|---|---|
| 1 | Pouco — logs existem por serviço mas não há visão consolidada |
| 2 | Disponibilidade e latência básica — um dashboard centralizado mostra se as APIs estão funcionando |
| 3 | SLOs definidos e monitorados — a organização sabe não apenas se as APIs funcionam mas se estão dentro dos parâmetros de qualidade acordados |
| 4 | Tendências e correlações — a organização detecta degradações antes que violem SLOs e correlaciona dados operacionais com contexto de portfólio |
| 5 | Observabilidade preditiva — anomalias são detectadas antes de se tornarem incidentes |

---

**D5.Q2 — Visibilidade de portfólio**

O CoE consegue responder a estas perguntas sem pesquisa manual?

*Qual porcentagem das APIs está em compliance com as políticas atuais? · Quais APIs têm menor qualidade de especificação? · Quais domínios estão regredindo em maturidade?*

| Nível | Resposta |
|---|---|
| 1 | Não — essas perguntas requerem semanas de investigação manual |
| 2 | Parcialmente — algumas informações estão disponíveis mas com esforço de coleta |
| 3 | Sim, para as perguntas básicas — compliance e qualidade de especificação estão disponíveis |
| 4 | Sim, incluindo tendências — a organização consegue ver a evolução dessas métricas ao longo do tempo |
| 5 | Sim, com contexto de negócio — os dados de portfólio estão correlacionados com valor de negócio entregue |

---

**D5.Q3 — Uso dos dados**

Quando foi a última vez que dados de observabilidade geraram uma decisão sobre o portfólio?

| Nível | Resposta |
|---|---|
| 1 | Não acontece — decisões são baseadas em percepção e experiência |
| 2 | Raramente — dados são apresentados em reuniões mas raramente mudam decisões |
| 3 | Ocasionalmente — dados de incidentes e qualidade informam algumas decisões |
| 4 | Regularmente — revisões periódicas do portfólio têm dados como ponto de partida obrigatório |
| 5 | Continuamente — o CoE tem alertas configurados que acionam análise quando métricas saem do padrão |

---

## D6 · Developer Experience

**D6.Q1 — Tempo de primeira integração**

Quanto tempo leva para um desenvolvedor novo fazer a primeira chamada bem-sucedida a uma API crítica do portfólio?

| Nível | Resposta |
|---|---|
| 1 | Dias a semanas — requer encontrar a pessoa certa, obter acesso informal, entender por tentativa e erro |
| 2 | Horas a dias — há documentação mas está incompleta ou desatualizada |
| 3 | Horas — o portal tem documentação completa, ambiente de teste e processo estruturado de credenciais |
| 4 | Minutos a uma hora — self-service completo sem dependência de intervenção humana |
| 5 | Descoberta assistida — o desenvolvedor descreve o que precisa e o sistema recomenda as APIs relevantes |

---

**D6.Q2 — Qualidade da documentação**

Como a documentação das APIs é mantida atualizada?

| Nível | Resposta |
|---|---|
| 1 | Não é mantida — a documentação fica desatualizada logo após a publicação |
| 2 | Manualmente — times atualizam quando lembram ou quando recebem feedback de consumidores |
| 3 | Automaticamente a partir da especificação — a documentação é gerada da spec e está sempre sincronizada com o contrato |
| 4 | Automaticamente com enriquecimento — além da spec, há exemplos, guias de uso e casos de borda mantidos com processo estruturado |
| 5 | Continuamente melhorada — dados de uso e tickets de suporte identificam onde a documentação é insuficiente e orientam melhorias |

---

**D6.Q3 — Sustentação da integração**

Como os consumidores são informados sobre mudanças nas APIs que usam?

| Nível | Resposta |
|---|---|
| 1 | Não são — descobrem quando a integração para de funcionar |
| 2 | Por comunicação ad hoc — emails ou posts em canais internos sem estrutura consistente |
| 3 | Por processo definido — breaking changes têm período de aviso com antecedência mínima definida |
| 4 | Proativamente e direcionado — cada consumidor ativo recebe notificação específica sobre mudanças nas APIs que usa |
| 5 | Com suporte ativo à migração — guias de migração, ambientes de teste da nova versão e suporte dedicado durante a transição |

---

## D7 · AI Readiness

**D7.Q1 — Consideração agêntica no design**

O consumo por agentes de IA é considerado no design de novas APIs?

| Nível | Resposta |
|---|---|
| 1 | Não — APIs são projetadas exclusivamente para consumo por desenvolvedores humanos |
| 2 | Informalmente — alguns times consideram, outros não, sem critério formal |
| 3 | Formalmente — há critérios definidos de AI Readiness que precisam ser satisfeitos antes de marcar uma API como pronta para consumo agêntico |
| 4 | Como caso de uso primário em APIs agênticas — o design considera explicitamente como agentes vão descobrir, selecionar e invocar a API |
| 5 | Nativamente — todo o portfólio ativo é projetado com consumo agêntico como caso de uso de primeira classe |

---

**D7.Q2 — Qualidade semântica para agentes**

As descrições de operações são suficientes para que um agente entenda quando e como usar cada operação?

| Nível | Resposta |
|---|---|
| 1 | Não — descrições são mínimas ou assumem contexto de domínio que agentes não têm |
| 2 | Parcialmente — algumas APIs têm descrições elaboradas, mas não há critério formal de suficiência |
| 3 | Para APIs marcadas como AI-ready — edge cases são documentados, operações irreversíveis são sinalizadas, exemplos cobrem casos típicos |
| 4 | Para a maior parte do portfólio ativo — a qualidade semântica é verificada automaticamente como parte do processo de publicação |
| 5 | Para todo o portfólio — incluindo APIs legadas que foram retrospectivamente atualizadas |

---

**D7.Q3 — Acessibilidade agêntica e identidade**

Como agentes de IA acessam as APIs do portfólio?

| Nível | Resposta |
|---|---|
| 1 | Da mesma forma que desenvolvedores humanos — sem distinção de mecanismo ou identidade |
| 2 | Alguns times criaram acesso específico para agentes, mas sem padrão organizacional |
| 3 | APIs críticas estão expostas via MCP. Agentes têm identidades próprias separadas das credenciais humanas |
| 4 | A maior parte do portfólio é acessível agenticamente. O acesso agêntico é auditável — a organização sabe quais agentes acessam quais APIs |
| 5 | A governança de identidade agêntica está integrada ao modelo de governança do portfólio — com credenciamento, escopos, auditoria e revogação estruturados |

---

## D8 · Operacionalização

**D8.Q1 — Automação de enforcement**

Como as políticas de governança são verificadas?

| Nível | Resposta |
|---|---|
| 1 | Manualmente — por revisão humana quando alguém tem tempo e disposição para fazê-la |
| 2 | Parcialmente — algumas políticas têm verificação automática, outras dependem de revisão manual |
| 3 | Automaticamente para todas as políticas críticas — gates no CI/CD bloqueiam publicações não conformes |
| 4 | Automaticamente com análise semântica — além de regras determinísticas, verificações baseadas em IA detectam problemas que regras fixas não capturam |
| 5 | O enforcement é invisível — está incorporado ao fluxo de trabalho de forma que a conformidade é o caminho natural |

---

**D8.Q2 — Integração ao ciclo de desenvolvimento**

Como os desenvolvedores interagem com as ferramentas de governança?

| Nível | Resposta |
|---|---|
| 1 | Não interagem — governança é externa ao ciclo de desenvolvimento |
| 2 | Depois de terminar o desenvolvimento — a verificação de conformidade é um passo separado |
| 3 | Durante o desenvolvimento — ferramentas integradas ao ambiente de desenvolvimento dão feedback em tempo real |
| 4 | Com fricção mínima — as ferramentas de governança são as mesmas que os times já usam para outros propósitos |
| 5 | Os desenvolvedores descrevem o processo de governança como uma ajuda, não como um obstáculo |

---

**D8.Q3 — Escalabilidade da governança**

Se o portfólio dobrasse de tamanho nos próximos 12 meses, a qualidade da governança se manteria?

| Nível | Resposta |
|---|---|
| 1 | Não — a governança já está sobrecarregada no tamanho atual |
| 2 | Provavelmente não — partes significativas dependem de capacidade humana que não escala linearmente |
| 3 | Sim para as partes automatizadas, mas o CoE precisaria crescer para manter as partes que dependem de revisão humana |
| 4 | Sim — a maior parte do enforcement é automatizada e escala independentemente do volume |
| 5 | Sim, e com melhoria — um portfólio maior gera mais dados que aprimoram a inteligência de governança |

---

## Consolidação do diagnóstico

Após responder todas as perguntas, consolide os resultados na tabela abaixo. O nível de cada dimensão é determinado pela leitura holística das respostas — não pela média aritmética.

| Dimensão | Nível atual | Evidências que fundamentam |
|---|:---:|---|
| D1 · Estratégia e Liderança | | |
| D2 · Design e Padrões | | |
| D3 · Ciclo de Vida | | |
| D4 · Segurança | | |
| D5 · Observabilidade | | |
| D6 · Developer Experience | | |
| D7 · AI Readiness | | |
| D8 · Operacionalização | | |

**Próximos passos após o diagnóstico:**

1. Compare os resultados entre diferentes grupos que responderam o questionário — as discrepâncias são tão informativas quanto os níveis
2. Identifique as dimensões habilitadoras que estão mais defasadas (especialmente D1 e D8)
3. Construa o roadmap de evolução seguindo o Cap 7.12
4. Defina critérios de transição observáveis para cada evolução pretendida
5. Agende a próxima rodada de diagnóstico — maturidade é uma prática contínua

---

*Série: Gerenciamento e Governança de APIs · Módulo 7 · Anexo N*