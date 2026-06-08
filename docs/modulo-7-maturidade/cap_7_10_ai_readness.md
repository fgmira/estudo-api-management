# Módulo 7 · Maturidade em Governança de APIs
## Capítulo 7.10 · Dimensão: AI Readiness

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Diagnóstico e progressão
> **Pré-requisito:** Cap 7.3 · Módulo 6 completo

---

## Sumário

- [7.10.1 · O que esta dimensão mede](#7101--o-que-esta-dimensão-mede)
- [7.10.2 · Os cinco níveis na prática](#7102--os-cinco-níveis-na-prática)
- [7.10.3 · Padrões de falha comuns](#7103--padrões-de-falha-comuns)
- [7.10.4 · Os desafios de progressão](#7104--os-desafios-de-progressão)
- [7.10.5 · Como avaliar o nível atual](#7105--como-avaliar-o-nível-atual)
- [7.10.6 · Conexões com os módulos anteriores](#7106--conexões-com-os-módulos-anteriores)

---

## 7.10.1 · O que esta dimensão mede

AI Readiness mede a prontidão do portfólio de APIs para ser consumido por agentes de inteligência artificial — sistemas autônomos que descobrem, selecionam e invocam APIs para completar tarefas sem intervenção humana direta em cada passo.

Esta dimensão é deliberadamente restrita ao **portfólio governado** — as APIs que a organização publica e mantém. Como a governança em si usa IA é uma questão de Operacionalização, tratada na Dimensão 8. Aqui a pergunta é outra: as APIs do portfólio têm as características que permitem que um agente as use de forma confiável e autônoma?

A distinção entre Developer Experience e AI Readiness é importante de estabelecer. DX mede a qualidade da experiência para desenvolvedores humanos. AI Readiness mede a qualidade da experiência para agentes — e os requisitos são diferentes em aspectos fundamentais.

Um desenvolvedor humano tolera ambiguidade: lê uma descrição imprecisa, infere o significado pelo contexto, pergunta quando está em dúvida. Um agente não tem essas capacidades de forma confiável. Uma descrição que é "suficientemente clara" para um desenvolvedor humano pode ser insuficiente para um agente que precisa decidir autonomamente se deve invocar aquela operação, com quais parâmetros, e como interpretar o resultado.

A pergunta central desta dimensão: **um agente de IA consegue descobrir as APIs relevantes do portfólio, entender o que cada uma faz sem ambiguidade, invocá-las corretamente e interpretar os resultados — sem intervenção humana no processo?**

Três elementos compõem a maturidade em AI Readiness:

**Qualidade semântica** — descrições de operações, parâmetros e respostas com precisão suficiente para que um agente entenda o que fazer, quando fazer e o que esperar — incluindo edge cases e operações irreversíveis.

**Acessibilidade agêntica** — mecanismos que permitem que agentes descubram e acessem as APIs do portfólio: exposição via MCP, catálogo com metadados semânticos, taxonomia de capacidades pesquisável por intenção.

**Governança de identidade agêntica** — a organização sabe quais agentes estão acessando suas APIs, com quais permissões, em nome de quem, e tem mecanismos para auditar, controlar e revogar esse acesso.

---

## 7.10.2 · Os cinco níveis na prática

### Nível 1 — Caótico

O consumo agêntico não é considerado. APIs são projetadas exclusivamente para consumo humano direto — desenvolvedor lê a documentação, escreve o código, faz a integração. Descrições, quando existem, são escritas para comunicar intenção a humanos e pressupõem contexto de domínio que agentes não têm. Não há mecanismo de descoberta por intenção, não há exposição via MCP, não há consideração de identidade agêntica.

---

### Nível 2 — Reativo

A organização reconhece que agentes de IA vão consumir suas APIs — ou já estão consumindo, de forma não controlada. Algumas APIs têm descrições mais elaboradas, alguns times começam a considerar legibilidade por máquina ao escrever especificações. Não há critério formal de AI Readiness, não há inventário de quais APIs são adequadas para consumo agêntico, não há governança de identidade agêntica.

---

### Nível 3 — Proativo

AI Readiness é um critério formal de publicação: uma API não pode ser marcada como pronta para consumo agêntico sem satisfazer requisitos de qualidade semântica — descrições completas, edge cases documentados, operações irreversíveis explicitamente sinalizadas, exemplos que cobrem casos de uso típicos.

Os primeiros MCP Servers expõem APIs críticas do portfólio. O catálogo tem metadados semânticos suficientes para descoberta por intenção — não apenas por nome ou categoria. A organização começa a ter visibilidade sobre quais agentes acessam suas APIs, embora a governança ainda seja básica.

---

### Nível 4 — Preditivo

A maior parte do portfólio ativo satisfaz os critérios de AI Readiness. Agentes conseguem descobrir e usar a maioria das APIs sem orientação adicional. A identidade agêntica está estruturada: agentes têm identidades próprias com escopos específicos, o acesso é auditável, e há mecanismos para revogar permissões de agentes específicos.

A organização monitora como agentes estão usando suas APIs — quais operações são mais invocadas, quais geram mais erros de interpretação, quais precisam de melhoria. Dados de uso agêntico orientam evoluções do portfólio.

---

### Nível 5 — Transformador

APIs são projetadas com consumo agêntico como caso de uso primário — não como consideração posterior. O design considera explicitamente como um agente vai descobrir, selecionar, invocar e compor APIs para completar tarefas complexas. O portfólio é um ecossistema agêntico: APIs são componentizáveis, têm semântica precisa o suficiente para composição autônoma, e a governança de agentes está integrada ao modelo de governança do portfólio como um todo.

---

## 7.10.3 · Padrões de falha comuns

### Documentação para humanos, não para agentes

A documentação foi escrita para comunicar intenção a um desenvolvedor que vai ler e interpretar. Ela diz "retorna os pedidos do cliente" — o que é compreensível para um humano que entende o contexto de negócio, mas ambíguo para um agente que precisa saber: todos os pedidos? De qual período? Em qual estado? Ordenados como?

A diferença não é de detalhe — é de modelo de consumidor. Documentação para humanos pode deixar implícito o que o contexto torna óbvio. Documentação para agentes precisa tornar explícito o que seria implícito, porque agentes não têm o contexto que humanos acumulam pela experiência com o domínio.

### MCP como checkbox

O MCP Server existe. Cada API do portfólio tem uma ferramenta correspondente. O CoE marcou "MCP implementado" no seu roadmap de AI Readiness. Na prática, as descrições das ferramentas no MCP Server são cópias das descrições da spec original — que foram escritas para humanos, não para agentes. O servidor existe mas os agentes não conseguem usá-lo de forma confiável porque a qualidade semântica das descrições não foi revisada com um modelo de consumo agêntico em mente.

MCP como checkbox é o equivalente de AI Readiness do compliance de checkbox de segurança: a estrutura existe, a substância não.

### Agentes sem identidade

Agentes de IA estão consumindo APIs do portfólio com credenciais de desenvolvedores humanos — porque ninguém configurou credenciais específicas para agentes, ou porque o processo de obter credenciais para agentes é tão burocrático que times simplesmente reutilizaram o que tinham. O resultado é que a organização não sabe distinguir tráfego humano de tráfego agêntico, não consegue auditar o que agentes específicos estão fazendo, e não tem como revogar o acesso de um agente sem revogar também o acesso do humano cujas credenciais ele está usando.

---

## 7.10.4 · Os desafios de progressão

### Do Nível 1 para o Nível 2

O desafio é a conscientização: a maioria das organizações no Nível 1 não está nesse nível por negligência — está porque o consumo agêntico era hipotético quando as APIs foram projetadas. A transição começa com reconhecer que o modelo de consumo está mudando e que APIs projetadas exclusivamente para humanos vão progressivamente se tornar menos úteis num ecossistema onde agentes são consumidores de primeira classe.

### Do Nível 2 para o Nível 3

O salto requer definir e operacionalizar critérios de AI Readiness — o que significa que uma API está pronta para consumo agêntico. Esses critérios precisam ser verificáveis automaticamente ou por processo estruturado de revisão, não apenas por julgamento subjetivo de quem está publicando.

A exposição via MCP no Nível 3 requer um esforço de revisão das descrições de ferramentas com modelo mental de agente — não apenas mapear operações existentes para o formato MCP. Esse trabalho é mais sobre qualidade semântica do que sobre implementação técnica.

### Do Nível 3 para o Nível 4

A cobertura do portfólio é o desafio principal: elevar a maior parte das APIs ao nível de AI Readiness requer revisar especificações existentes com critérios que não existiam quando foram escritas. Isso é um trabalho de retrofit que compite com o desenvolvimento de novas capacidades.

A estruturação de identidade agêntica no Nível 4 requer que a organização trate agentes como uma classe de consumidor distinta — com seu próprio processo de credenciamento, seus próprios escopos e seu próprio modelo de auditoria. Isso tem implicações no modelo de acesso que pode não estar previsto na infraestrutura de autenticação existente.

### Do Nível 4 para o Nível 5

Design agêntico nativo requer um shift na forma como APIs são concebidas: começar pelo caso de uso agêntico — como um agente vai usar isso? — em vez de começar pelo caso de uso humano e adaptar. Isso implica que times de desenvolvimento desenvolvam intuição sobre como agentes raciocinam e quais características de design facilitam ou dificultam o uso autônomo confiável.

---

## 7.10.5 · Como avaliar o nível atual

| Pergunta | O que revela |
|---|---|
| O consumo agêntico é considerado no design de novas APIs? | Presença de consciência agêntica — Nível 2+ |
| Há critérios formais definindo o que é uma API "AI-ready"? | Diferença entre Nível 2 e Nível 3 |
| As descrições de operações documentam edge cases e operações irreversíveis explicitamente? | Qualidade semântica para agentes |
| APIs críticas do portfólio estão expostas via MCP com descrições revisadas para consumo agêntico? | Presença de acessibilidade agêntica — Nível 3+ |
| A organização consegue distinguir tráfego agêntico de tráfego humano nos logs? | Presença de identidade agêntica — Nível 3+ |
| Agentes têm identidades e escopos próprios, separados das credenciais humanas? | Governança de identidade agêntica — Nível 4 |

---

## 7.10.6 · Conexões com os módulos anteriores

**Módulo 6 — IA e APIs**

O Módulo 6 estabeleceu o fundamento teórico desta dimensão: o que é consumo agêntico, quais são as características que tornam uma API adequada para agentes, como MCP funciona como protocolo de acesso agêntico, e o que é AI Readiness como propriedade de design. Esta dimensão de maturidade mede o quanto essas práticas estão operando de forma consistente no portfólio — não apenas conhecidas ou adotadas pontualmente.

**Cap 6.8 — AI Readiness do portfólio**

O Cap 6.8 detalhou especificamente como avaliar e evoluir o portfólio em termos de prontidão agêntica. Os critérios descritos no Cap 6.8 são os critérios que, quando aplicados de forma consistente a todo o portfólio, definem a progressão entre os níveis desta dimensão.

---

## Pontos-chave do capítulo

- AI Readiness desta dimensão mede o portfólio governado — a prontidão das APIs para consumo agêntico. O uso de IA na governança em si é Operacionalização (Dimensão 8)
- A diferença entre DX e AI Readiness é o modelo de consumidor: humanos toleram ambiguidade e inferem pelo contexto; agentes precisam de precisão semântica explícita
- Os três elementos: qualidade semântica, acessibilidade agêntica e governança de identidade agêntica
- MCP como checkbox e agentes sem identidade são os padrões de falha mais comuns — ambos resultam de tratar AI Readiness como projeto técnico em vez de mudança de modelo de consumidor
- A transição para o Nível 4 requer tratar agentes como classe de consumidor distinta — com credenciamento, escopos e auditoria próprios
- Nível 5 é design agêntico nativo: começar pelo caso de uso agêntico, não adaptar o caso de uso humano

---

## Próximo capítulo

**7.11 · Dimensão: Operacionalização** — maturidade na automação e no suporte ferramental à governança, incluindo o uso de IA para aprimorar o próprio processo de governança.

---

*Série: Gerenciamento e Governança de APIs · Módulo 7 · Capítulo 7.10*