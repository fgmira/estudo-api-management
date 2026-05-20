# Anexo A.3 · Breaking changes em Protocol Buffers (gRPC)

> **Referência:** Capítulo 2.5 · Versionamento e gestão de breaking changes
> **Série:** Gerenciamento e Governança de APIs

---

## Non-breaking changes

| Mudança | Notas |
|---|---|
| Adicionar novo campo com número novo | Clientes antigos ignoram campos desconhecidos na desserialização |
| Adicionar novo RPC ao serviço | Clientes antigos simplesmente não chamam o novo RPC |
| Adicionar novo tipo de mensagem | Não afeta mensagens existentes |
| Adicionar novo valor em enum | Clientes antigos recebem UNKNOWN — tratar adequadamente no código |
| Tornar campo required em optional (proto2) | Clientes antigos continuam enviando o campo |

---

## Breaking changes

| Mudança | Impacto |
|---|---|
| Mudar número de campo existente | Serialização binária incompatível — dados corrompidos |
| Remover campo sem reservar o número | O número pode ser reutilizado — dados corrompidos em mensagens antigas |
| Mudar tipo de campo | Serialização binária incompatível |
| Remover RPC existente | Clientes que chamam o RPC recebem UNIMPLEMENTED |
| Mudar nome do package | Geração de código falha — nome do package faz parte da assinatura |
| Renomear mensagem referenciada em outros protos | Quebra a compilação de protos dependentes |
| Mudar stream direction (unary para streaming) | Interface incompatível |

---

## Boas práticas para evitar breaking changes em Protobuf

**Sempre usar `reserved` ao remover campos**

Evita reutilização acidental de números de campo em versões futuras:

```protobuf
message PagamentoRequest {
  reserved 2; // campo "moeda_legado" removido na v1.2
  reserved "moeda_legado";

  string destinatario_id = 1;
  string moeda = 3;
}
```

**Nunca reutilizar números de campo** — mesmo que o campo tenha sido removido. O número identifica o campo na serialização binária.

**Versionar o package ao introduzir breaking changes:**

```protobuf
// Versão anterior
package pagamentos.v1;

// Nova versão com breaking changes
package pagamentos.v2;
```

---

## Ferramentas recomendadas

- **Buf CLI** — lint e detecção automática de breaking changes em arquivos `.proto`
- **Buf Schema Registry** — registry centralizado de schemas Protobuf com versionamento
- **grpcurl** — teste de APIs gRPC via linha de comando

---

*Série: Gerenciamento e Governança de APIs · Anexo A.3*