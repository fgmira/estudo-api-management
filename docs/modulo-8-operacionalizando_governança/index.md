# Módulo 8 — Operacionalizando a Governança de APIs

> **Série:** Gerenciamento e Governança de APIs
> **Nível:** Estratégico e técnico
> **Pré-requisito:** Módulos 1 a 7

---

## O que este módulo constrói

Os módulos anteriores construíram o vocabulário, os frameworks e os critérios de avaliação da governança de APIs. Este módulo trata de **colocar tudo isso em operação**: como transformar intenção em infraestrutura, políticas em código, diagnóstico em roadmap executado.

Governança que existe apenas como documento ou como processo manual não escala. Este módulo descreve o caminho da governança como capacidade operacional sustentável.

---

## Capítulos

### [Prefácio · A plataforma como produto](prefacio.md)

A distinção fundamental entre uma plataforma construída de dentro para fora — como sistema de controle — e uma construída de fora para dentro — como produto cujo sucesso se mede pela adoção voluntária. O argumento que atravessa todo o módulo.

---

### [8.1 · Da intenção à infraestrutura](cap_8_1_intensao_infra.md)

O capítulo de abertura do módulo. Trata do limite da governança manual, o que significa operacionalizar a governança, a plataforma como produto interno e o que este módulo cobre.

---

### [8.2 · Os contextos de uma plataforma de governança](cap_8_2_contextos_plataforma.md)

O mapa arquitetural da plataforma. Cobre as **interface layers** (as superfícies de acesso), os **bounded contexts** que compõem o núcleo da plataforma e o **context map** que define como esses contextos se relacionam.

---

### [8.3 · O catálogo como fonte de verdade](cap_8_3_catalogo.md)

O problema que o catálogo resolve, o que um registro precisa conter, o ciclo de vida do registro, como o catálogo se mantém fiel à realidade, o catálogo como **fundação das outras capacidades** e os desafios comuns de implementação.

---

### [8.4 · Políticas como código](cap_8_4_politicas.md)

O argumento por tratar políticas de governança como artefatos versionados. Cobre o que uma política contém, seu ciclo de vida, **enforcement por ambiente**, exceções como parte do modelo, **preview de impacto** antes de ativar uma política nova e os desafios comuns.

---

### [8.5 · O pipeline de governança](cap_8_5_pipeline.md)

O ponto onde governança se torna gate automatizado. Cobre o que é um pipeline de governança, os **gates** como unidades de enforcement, a distinção entre **CI e CD** como dois contextos de validação distintos, a tensão entre rigor e velocidade, o modelo de integração e execução e os desafios comuns.

---

### [8.6 · Inteligência de portfólio](cap_8_6_inteligencia.md)

Como dados se transformam em decisões. Cobre o modelo de **saúde do portfólio**, tendências versus instantâneos, os padrões que dados revelam e regras não capturam, o papel da IA na inteligência de portfólio, os **indicadores inegociáveis** e os desafios comuns.

---

### [8.7 · Descoberta e reconciliação](cap_8_7_descoberta.md)

Como manter o catálogo fiel à realidade da infraestrutura. Cobre o problema da divergência, o **modelo de agent distribuído**, as capabilities de cada agent, o orchestrator e as trigger rules, o catálogo de divergências e o princípio de que descoberta não decide — apenas reporta.

---

### [8.8 · Portal e a experiência do desenvolvedor](cap_8_8_portal.md)

O portal como superfície de acesso à plataforma para duas audiências distintas. Cobre **descoberta** (encontrar a API certa), **integração** (do contrato à primeira chamada), **self-service** (acesso sem burocracia), o painel do CoE e os desafios comuns.

---

### [8.9 · Conhecimento como contexto independente](cap_8_9_conhecimento.md)

Por que conhecimento requer gestão própria — não é uma feature do portal. Cobre o ciclo de vida do conteúdo, a organização para **consumo humano e agêntico**, como o conhecimento se conecta com os outros contextos, a governança do conhecimento e os desafios comuns.

---

### [8.10 · Assistência inteligente](cap_8_10_assistencia.md)

Os três ângulos de IA na plataforma: a plataforma como **serviço consumido por agentes**, a **IA usada pelos contextos internos** e os **agentes que assistem os usuários**. Cobre os princípios para uso de IA em governança e os desafios comuns.

---

### [8.11 · DX e inteligência de produto](cap_8_11_dx_inteligencia_produto.md)

Como medir e evoluir a experiência de usar a plataforma. Cobre o que DX significa em governança além do portal, como medir DX, a **inteligência de produto** como capacidade analítica da própria plataforma e o ciclo do dado à decisão de produto.

---

### [8.12 · Identidade e acesso](cap_8_12_identidade_acesso.md)

Quem pode fazer o quê na plataforma. Cobre os **três tipos de principal** (humano, sistema, agente), o **modelo de papéis**, a integração com sistemas externos de identidade, a autossuficiência sem IdP externo, a **identidade agêntica** e a autenticação por canal.

---

### [8.13 · Build, buy ou compor](cap_8_13_comprar_fazer.md)

Como decidir o que construir, adquirir ou compor. Cobre o falso binário entre build e buy, o que o mercado oferece, os critérios de avaliação, os riscos de acoplamento a vendors e a **abordagem de composição** como terceira via.

---

### [8.14 · Adoção — o problema mais difícil](cap_8_14_adocao.md)

O lado humano da operacionalização. Cobre por que boas plataformas falham na adoção, o **problema da narrativa**, as estratégias de introdução, o papel dos **champions e primeiros adotantes**, como medir adoção real e o jogo longo da mudança organizacional.

---
