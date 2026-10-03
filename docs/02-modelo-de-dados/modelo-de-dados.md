# Modelo de dados

Transcrição do [DER actual](DER_Portal_Pedidos_Intervencao.svg) ([PNG](DER_Portal_Pedidos_Intervencao.png)). O racional de cada entidade está na §9 dos [Requisitos v2.5](../01-requisitos/Requisitos_v2.5.md) e as notas de mapeamento JPA estão na §13.4.

```mermaid
erDiagram
  CLIENTE ||--o{ BALCAO : possui
  CLIENTE ||--o{ UTILIZADOR : pertence
  BALCAO ||--o{ IMPRESSORA : aloja
  BALCAO ||--o{ ATRIBUICAO : tem
  UTILIZADOR ||--o{ ATRIBUICAO : cobre
  IMPRESSORA ||--o{ PEDIDO : gera
  IMPRESSORA ||--o{ PLANO_MANUTENCAO : segue
  UTILIZADOR ||--o{ PEDIDO : "abre / executa"
  PEDIDO ||--o{ PEDIDO_EVENTO : historico
  PEDIDO ||--o{ CHECKLIST_RESPOSTA : inclui
  CHECKLIST_ITEM ||--o{ CHECKLIST_RESPOSTA : responde
  PEDIDO ||--o{ ANEXO : tem
  PEDIDO ||--o{ CONSUMIVEL_USADO : consome

  UTILIZADOR {
    bigint id PK
    uuid uuid
    string nome
    string email
    string palavra_passe_hash
    string papel
    bigint cliente_id FK
    boolean ativo
  }
  CLIENTE {
    bigint id PK
    string nome
    int sla_horas
    time horario_inicio
    time horario_fim
    boolean ativo
  }
  BALCAO {
    bigint id PK
    bigint cliente_id FK
    string nome
    string localizacao
    string contacto
  }
  IMPRESSORA {
    bigint id PK
    bigint balcao_id FK
    string marca_modelo
    string nr_serie
    string nr_interno
    string estado
  }
  ATRIBUICAO {
    bigint id PK
    bigint balcao_id FK
    bigint tecnico_id FK
    boolean principal
  }
  PLANO_MANUTENCAO {
    bigint id PK
    bigint impressora_id FK
    int periodicidade_meses
    date proxima_data
  }
  PEDIDO {
    bigint id PK
    string nr_pedido
    string referencia_antiga
    bigint impressora_id FK
    bigint aberto_por FK
    bigint tecnico_id FK
    string tipo
    string estado
    text descricao_avaria
    timestamp aberto_em
    timestamp agendado_para
    timestamp iniciado_em
    timestamp resolvido_em
    timestamp fechado_em
    text resumo_trabalho
    int contador_copias
  }
  PEDIDO_EVENTO {
    bigint id PK
    bigint pedido_id FK
    bigint autor_id FK
    string tipo
    string estado_anterior
    string estado_novo
    text comentario
    timestamp criado_em
  }
  CHECKLIST_ITEM {
    bigint id PK
    string descricao
    int ordem
    boolean ativo
  }
  CHECKLIST_RESPOSTA {
    bigint id PK
    bigint pedido_id FK
    bigint item_id FK
    string resultado
    text observacao
  }
  ANEXO {
    bigint id PK
    bigint pedido_id FK
    bigint carregado_por FK
    string caminho
    string tipo_mime
    timestamp criado_em
  }
  CONSUMIVEL_USADO {
    bigint id PK
    bigint pedido_id FK
    string designacao
    int quantidade
  }
```

> Os tipos (`bigint`, `time`, etc.) seguem a §13.4 da v2.5. O DER original só indica nomes, PK e FK. Na imagem, só o `utilizador` mostra a coluna `uuid`, mas a v2.5 diz que todas as entidades expostas na API a devem ter.

## Lacunas conhecidas

Inconsistências entre o DER e os requisitos. Convém resolvê-las antes da primeira migração Flyway:

| # | Lacuna | Requisito afectado |
|---|---|---|
| L1 | Não existe uma tabela `utilizador_balcao`. O Utilizador Cliente está associado "a um ou mais balcões" e o RN02 limita a visibilidade por balcão, mas o DER só tem `cliente_id`. | Glossário, RN02, `GET /me` |
| L2 | `pedido.resolvido_em` existe, mas a máquina de estados não tem o estado "resolvido": o fecho é feito num só passo. Ou se remove a coluna, ou se define quando é preenchida. | §6 |
| L3 | A v2.5 diz que os anexos podem estar "associados a pedido ou evento", mas `anexo` só tem `pedido_id`. | §9, RF12 |
| L4 | O `uuid` público só aparece no `utilizador`. | §13.4 |
| L5 | Falta a tabela de outbox das notificações, que a §13.2 exige. | RF27–RF30 |
| L6 | Falta guardar o canal de notificação preferido de cada utilizador (email ou SMS) e o seu telefone. | RF30 |
