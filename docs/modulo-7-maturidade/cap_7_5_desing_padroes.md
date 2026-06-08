# Módulo 7 · Maturidade em Governança de APIs
## Capítulo 7.5 · Dimensão: Design e Padrões

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Diagnóstico e progressão
> **Pré-requisito:** Cap 7.3 · Cap 2.2 · Cap 3.4

---

## Sumário

- [7.5.1 · O que esta dimensão mede](#751--o-que-esta-dimensão-mede)
- [7.5.2 · Os cinco níveis na prática](#752--os-cinco-níveis-na-prática)
- [7.5.3 · Padrões de falha comuns](#753--padrões-de-falha-comuns)
- [7.5.4 · Os desafios de progressão](#754--os-desafios-de-progressão)
- [7.5.5 · Como avaliar o nível atual](#755--como-avaliar-o-nível-atual)
- [7.5.6 · Conexões com os módulos anteriores](#756--conexões-com-os-módulos-anteriores)

---

## 7.5.1 · O que esta dimensão mede

Design e Padrões mede a consistência do contrato de APIs no portfólio — o quanto as APIs se parecem entre si em estrutura, nomenclatura, comportamento e evolução, independentemente de quem as construiu ou quando.

Consistência de design não é estética. É funcional. Um portfólio inconsistente impõe custo real a cada consumidor que precisa aprender as idiossincrasias de cada API. Um desenvolvedor que integrou dez APIs de uma organização consistente integra a décima-primeira com confiança — sabe o que esperar. Num portfólio inconsistente, cada nova API é um problema novo.

A pergunta central desta dimensão: **um desenvolvedor que nunca viu essa API consegue usá-la corretamente baseado apenas no contrato, sem precisar de ajuda adicional?**

Três elementos compõem a maturidade em Design e Padrões:

**Consistência estrutural** — nomenclatura de recursos, convenções de URL, estrutura de erros, paginação, versionamento. As APIs do portfólio seguem as mesmas convenções estruturais.

**Qualidade do contrato** — a especificação descreve com precisão o comportamento da API: o que cada operação faz, o que pode dar errado, quais são os edge cases. O contrato é suficiente para integração — não apenas para documentação.

**Evolução controlada** — mudanças no contrato seguem processos que protegem os consumidores existentes. Breaking changes são sinalizadas, versionadas e comunicadas com antecedência adequada.

---

## 7.5.2 · Os cinco níveis na prática

### Nível 1 — Caótico

Cada desenvolvedor usa suas próprias convenções. APIs criadas por times diferentes usam nomenclaturas incompatíveis, estruturas de erro distintas, estratégias de versionamento conflitantes. Quando padrões existem, são tácitos — vivem na cabeça de quem os criou.

O sintoma mais visível do Nível 1 é o custo de integração: consumidores precisam de suporte extensivo para integrar cada API porque o contrato não é suficientemente claro ou consistente para ser autoexplicativo.

---

### Nível 2 — Reativo

Um style guide existe — escrito em algum documento, wiki ou repositório. Times são "encorajados" a segui-lo. O problema: sem enforcement, o style guide é seguido por quem já acreditava nele e ignorado por quem tem pressão de prazo ou simplesmente não concorda com alguma regra.

O gap entre o style guide declarado e o portfólio real é o diagnóstico do Nível 2. Se o style guide diz "use snake_case nos campos" mas 40% das APIs usam camelCase, a organização está no Nível 2 independentemente da qualidade do documento.

---

### Nível 3 — Proativo

O style guide é enforçado automaticamente em CI/CD. Um lint de spec — Spectral ou equivalente — verifica as regras do style guide a cada push. APIs que violam as regras não são publicadas. O portfólio começa a ter consistência real, não apenas declarada.

No Nível 3, a qualidade do contrato começa a receber atenção sistemática: descrições de operação são obrigatórias, erros precisam ser documentados, schemas precisam ser precisos. A especificação passa de um artefato de documentação para um artefato de governança.

O versionamento tem processo definido: o que constitui uma breaking change, como versões são sinalizadas, qual é a política de suporte a versões anteriores.

---

### Nível 4 — Preditivo

Os padrões evoluem baseados em evidências — dados de uso, feedback de consumidores, análise de problemas de integração. O CoE não apenas enforça o style guide existente — revisa-o regularmente com base em aprendizados do portfólio.

APIs que consistentemente geram problemas de integração são analisadas para entender se o problema está na implementação ou no padrão. Quando a evidência sugere que o padrão está errado — não a implementação — o padrão é revisado.

A qualidade do contrato atinge um patamar onde consumidores externos conseguem integrar sem precisar de suporte. A especificação é suficiente.

---

### Nível 5 — Transformador

Design é um habilitador estratégico. As APIs do portfólio são interoperáveis por design — com sistemas da própria organização, com parceiros, com o ecossistema mais amplo. A consistência transcende o estilo e alcança a semântica: os mesmos conceitos de negócio são representados da mesma forma em todo o portfólio.

No Nível 5, APIs são projetadas pensando em consumo por agentes de IA — descrições semânticas que permitem descoberta e uso autônomo, não apenas documentação para humanos. O contrato é preciso o suficiente para ser processado por máquinas tanto quanto por desenvolvedores.

---

## 7.5.3 · Padrões de falha comuns

### Style guide com paralisia por análise

O style guide tem 200 regras. Cada regra tem exceções. Cada exceção tem sub-casos. O documento ficou tão detalhado que ninguém o lê até o fim — e quem lê fica inseguro sobre qual regra aplicar em casos reais.

A causa é uma tentativa de antecipar todos os casos possíveis antes de começar. O resultado é um guia que não guia — é um catálogo de decisões passadas que não resolve situações novas. Style guides maduros têm poucos princípios claros e muitos exemplos concretos, não o inverso.

### Retrofit impossível

A organização tem um portfólio de 150 APIs construídas ao longo de cinco anos. O novo style guide define padrões que 80% do portfólio não segue. A conclusão — implícita ou explícita — é que o style guide se aplica apenas a novas APIs. As antigas continuam como estão.

O resultado é um portfólio eternamente inconsistente: novas APIs seguem os padrões, antigas não. Consumidores que precisam integrar com APIs antigas e novas têm que lidar com dois mundos diferentes. A maturidade real fica represada pelo legado que ninguém se compromete a evoluir.

### Consistência sintática sem coerência semântica

O portfólio tem excelente consistência estrutural — todos os recursos usam os mesmos padrões de nomenclatura, paginação, erros. Mas o mesmo conceito de negócio é representado de formas diferentes em APIs diferentes: "cliente" em algumas, "user" em outras, "account" em outras. Campos semanticamente equivalentes têm nomes diferentes. Enumerações com o mesmo significado têm valores diferentes.

A consistência sintática sem coerência semântica cria um portfólio que parece consistente até o momento em que alguém tenta usar múltiplas APIs juntas — e descobre que precisam de camadas de mapeamento para reconciliar os conceitos.

---

## 7.5.4 · Os desafios de progressão

### Do Nível 1 para o Nível 2

O desafio é principalmente político: quem tem autoridade para definir o padrão? Times com APIs legadas resistem a padrões que implicam retrabalho. Times com estilos próprios resistem a padrões que parecem arbitrários.

A abordagem mais eficaz não é criar o style guide ideal do zero — é documentar os padrões que já funcionam bem no portfólio existente, torná-los explícitos, e construir consenso em torno deles. O style guide que emerge de casos reais tem mais aderência do que o que desce de princípios abstratos.

### Do Nível 2 para o Nível 3

O salto requer dois componentes técnicos: um lint de spec configurado com as regras do style guide, e a integração desse lint no pipeline de CI/CD com enforcement bloqueante.

O componente técnico é o mais simples. O desafio real é cultural: a primeira vez que o lint bloqueia uma publicação de um time importante, haverá pressão para criar uma exceção. Como o CoE responde a essa pressão determina se a organização está de fato no Nível 3 ou voltou ao governance theater.

### Do Nível 3 para o Nível 4

Requer construir a capacidade de aprender com o portfólio. Isso implica coletar dados sobre onde os consumidores têm dificuldades — chamadas de suporte, tickets, feedbacks — e correlacioná-los com características das APIs. Quais patterns de design geram mais confusão? Quais convenções de nomenclatura os consumidores consistentemente entendem errado?

Também requer um processo para revisar e evoluir o style guide baseado nesses dados — o que significa aceitar que algumas decisões passadas estavam erradas e comunicar mudanças sem quebrar o portfólio existente.

### Do Nível 4 para o Nível 5

A coerência semântica — garantir que os mesmos conceitos de negócio sejam representados da mesma forma em todo o portfólio — é o trabalho central desta transição. Requer um modelo de domínio compartilhado que transcende times e sistemas individuais. É um trabalho de arquitetura empresarial tanto quanto de governança de APIs.

---

## 7.5.5 · Como avaliar o nível atual

| Pergunta | O que revela |
|---|---|
| O style guide está publicado e acessível a todos os times? | Presença no Nível 2+ |
| Qual é a taxa de conformidade do portfólio com o style guide? | Diferença entre Nível 2 (baixa) e Nível 3 (alta) |
| O lint de spec roda automaticamente e bloqueia publicações em não-conformidade? | Diferença entre Nível 2 e Nível 3 |
| Quando foi a última vez que o style guide foi revisado? Com que base? | Diferença entre Nível 3 (revisão ad hoc) e Nível 4 (revisão baseada em dados) |
| Um consumidor externo consegue integrar uma API nova sem suporte adicional? | Qualidade real do contrato |
| O mesmo conceito de negócio é representado da mesma forma em todo o portfólio? | Coerência semântica — Nível 5 |

---

## 7.5.6 · Conexões com os módulos anteriores

**Cap 2.2 — Design de APIs**

O Cap 2.2 estabeleceu os princípios de design de APIs: clareza de intenção, consistência, previsibilidade, tolerância a falhas. Esta dimensão de maturidade mede o quanto esses princípios estão sendo aplicados de forma consistente em todo o portfólio — não apenas nas APIs mais cuidadas ou criadas pelos times mais experientes.

**Cap 3.4 — Style Guide**

O Cap 3.4 detalhou o que um style guide maduro contém: convenções de nomenclatura, padrões de versionamento, estrutura de erros, paginação, segurança. A maturidade em Design e Padrões mede o quanto esse style guide está sendo efetivamente seguido — a distância entre o que o documento diz e o que o portfólio mostra.

---

## Pontos-chave do capítulo

- Design e Padrões mede consistência funcional do portfólio — não estética. Portfólio consistente reduz custo de integração para cada novo consumidor
- Os três elementos: consistência estrutural, qualidade do contrato e evolução controlada
- A transição do Nível 2 para o 3 requer lint automático e bloqueante em CI/CD — não apenas um bom documento
- Style guide com paralisia por análise, retrofit impossível e consistência sintática sem coerência semântica são os três padrões de falha mais comuns
- O diagnóstico correto mede a taxa de conformidade real do portfólio, não a existência do style guide
- Nível 5 requer coerência semântica — mesmo conceito de negócio representado da mesma forma em todo o portfólio — que é um trabalho de arquitetura empresarial tanto quanto de governança

---

## Próximo capítulo

**7.6 · Dimensão: Ciclo de Vida** — maturidade na gestão de APIs desde a concepção até o sunset, gates de qualidade, depreciação planejada e portfolio management.

---

*Série: Gerenciamento e Governança de APIs · Módulo 7 · Capítulo 7.5*