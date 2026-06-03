# Módulo 7 — A Plataforma de Governança

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Técnico e estratégico
> **Pré-requisito:** Módulos 1 a 6

---

## O que este módulo constrói

Os módulos anteriores definiram *o que* governar — ciclo de vida, contratos, segurança, catálogo, identidade, maturidade. Este módulo trata de *como* governar em escala: a plataforma de software que materializa políticas em código, automatiza gates, centraliza observabilidade e torna a governança operacional em vez de apenas prescritiva.

O [Prefácio](prefacio.md) estabelece a filosofia de design da plataforma — os princípios derivados da PEP-20 do Python que funcionam como filtro para cada decisão de arquitetura, cada interface, cada mensagem de erro.

---

## Capítulos

### [Prefácio · A filosofia da plataforma](prefacio.md)

A PEP-20 — o Zen of Python — como fonte de princípios de design para uma plataforma de governança. Oito aforismos de Tim Peters aplicados ao problema de construir software que muitas pessoas diferentes vão usar, com necessidades frequentemente conflitantes. Inclui o 20º aforismo proposto: *"Unenforced rules breed contempt for all rules. Unless the rule was wrong."*

---

### [7.1 · Visão e posicionamento do produto](cap_7_1_visao_posicionamento.md)

O problema que a plataforma existe para resolver, para quem, onde estão as lacunas no landscape atual de ferramentas, o posicionamento como produto open source e os princípios de design derivados do prefácio.

---

### [7.2 · Arquitetura de referência](cap_7_2_arquitetura_referencia.md)

A arquitetura técnica da plataforma: o modelo de produto com **API, CLI, SDK e Console** como superfícies de acesso; os **cinco serviços** que compõem o núcleo; a separação entre camada **OLTP e OLAP** para análise do portfólio; o CLI em detalhe; o modelo de integração com sistemas externos e os fluxos principais de uso.

---

### [7.3 · O Registry Service — o coração da plataforma](cap_7_3_registry_service.md)

O serviço central que mantém o estado autoritativo de todo o portfólio de APIs. Cobre o **modelo de domínio** e suas entidades, o uso de **event sourcing** como mecanismo de persistência (estoque lógico com memória completa de mudanças), o **catálogo de eventos**, as **queries de governança** que o serviço expõe, o **context map** com os demais serviços e a especificação da API do Registry.
