# PrintDesk — Contexto completo do projecto

> Ficheiro de handoff para continuação em novo chat.
> Estado: **Fase 0 — Design de ecrãs**

---

## 1. O que é o projecto

Portal web para a empresa **Ricotecnica** (cliente: **Hélio Matsinhe**), que presta serviços de outsourcing de impressão — aluga e mantém impressoras em instalações de clientes de qualquer sector.

O problema actual: quando uma impressora avaria, o cliente envia email à Ricotecnica, um gestor de email cria um documento em papel e liga ao técnico. Ninguém sabe o estado em tempo real. Nada fica registado.

O portal substitui esse processo integralmente.

---

## 2. Stack

| Camada | Tecnologia |
|---|---|
| Backend | Java 21 + Spring Boot 4.1.1 + Maven |
| Frontend | React + Vite + TypeScript |
| Base de dados | MySQL 8 |
| ORM | Spring Data JPA (Hibernate) + Flyway |
| Autenticação | Spring Security + JWT |
| Estado remoto | TanStack Query |
| Estilos | Tailwind CSS |
| Notificações | Email + SMS (agregador local) |

**Nomes dos repositórios:**
- `printdesk-api` — backend
- `printdesk-web` — frontend

**Package name Java:** `com.printdesk.api`

---

## 3. Actores e papéis

Um único modelo de utilizador com 4 papéis. Cada utilizador tem exactamente um papel:

| Papel | O que faz |
|---|---|
| **Cliente** | Submete pedidos dos seus balcões, consulta estado |
| **Técnico** | Vê pedidos atribuídos, regista intervenção no local (móvel) |
| **HelpDesk** | Triagem, atribuição, actualização e fecho de pedidos |
| **Admin** | Gestão de utilizadores, clientes, balcões, equipamento |

---

## 4. Entidades principais (modelo de dados)

```
utilizador       → papel (enum), cliente_id, ativo
cliente          → nome, sla_horas (padrão: 24), horario_inicio, horario_fim
balcao           → cliente_id, nome, localizacao, contacto
impressora       → balcao_id, marca_modelo, nr_serie, nr_interno, estado
atribuicao       → balcao_id, tecnico_id, principal (bool)
pedido           → nr_pedido, impressora_id, aberto_por, tecnico_id, tipo, estado,
                   descricao_avaria, aberto_em, agendado_para, iniciado_em,
                   resolvido_em, fechado_em, resumo_trabalho, contador_copias,
                   referencia_antiga
pedido_evento    → pedido_id, autor_id, tipo, estado_anterior, estado_novo,
                   comentario, criado_em   [append-only]
checklist_item   → descricao, ordem, ativo
checklist_resp   → pedido_id, item_id, resultado, observacao
anexo            → pedido_id, carregado_por, caminho, tipo_mime, criado_em
consumivel_usado → pedido_id, designacao, quantidade
plano_manutencao → impressora_id, periodicidade_meses, proxima_data
```

---

## 5. Máquina de estados do pedido

```
submetido → atribuído → em reparação → fechado
               ↓              (qualquer estado antes de iniciar)
           cancelado
```

| Estado | Quem transita |
|---|---|
| submetido | Cliente / HelpDesk / Admin |
| atribuído | HelpDesk / Admin (atribuição automática ao técnico do balcão) |
| em reparação | Técnico (regista início no local) |
| fechado | Técnico / HelpDesk (com resumo + checklist + contador obrigatórios) |
| cancelado | Cliente (antes de iniciar) / HelpDesk / Admin |

**Regras importantes:**
- Pedido às 17h → prazo começa no dia útil seguinte
- SLA: 24 horas úteis para início da intervenção (não para conclusão)
- Falta de peça ou intervenção longa → pedido fica em `em reparação`, técnico acrescenta comentários
- Pedido fechado não reabre — se o problema voltar, abre novo pedido
- Uma avaria = um pedido, do início ao fim

---

## 6. Tipos de pedido

- **Avaria** — problema reportado pelo cliente
- **Manutenção preventiva** — gerada automaticamente pelo sistema a cada N meses
- **Visita de cortesia** — deslocação sem avaria

---

## 7. Funcionalidades por entrega (MVP = E0 a E6)

| Entrega | O que fica a funcionar | Semanas |
|---|---|---|
| E0 | Arranque: requisitos, wireframes, modelo de dados, ambiente | 1–2 |
| E1 | Autenticação, contas individuais e perfis de acesso | 3 |
| E2 | Cadastro de clientes, balcões e impressoras, importação do parque | 4–5 |
| E3 | Atribuição técnico–balcão e submissão de pedido pelo cliente | 6 |
| E4 | Ciclo completo do pedido com histórico de eventos | 7–8 |
| E5 | Ecrã móvel do técnico para registo no local | 9 |
| E6 | Notificações por email/SMS e consulta de estado pelo cliente | 10 |
| E7 | Manutenção preventiva automática com calendário | 11–12 |
| E8 | Checklist, contador de cópias e consumíveis | 13 |
| E9 | Alerta de reincidência e medição do SLA | 14 |
| E10 | Relatórios de prazos, produtividade por técnico | 15 |
| E11 | Exportação Excel e PDF | 16 |

**Prazo total: 16 semanas (4 meses)**
**Ponto de decisão na semana 10** — cliente decide se avança para E7–E11

---

## 8. Contrato da API (MVP)

Prefixo `/api`, JWT em `Authorization: Bearer`.

**Auth**
```
POST /auth/login
POST /auth/refresh
POST /auth/logout
POST /auth/convite/{token}
POST /auth/recuperar
GET  /me
```

**Parque**
```
GET  POST /clientes
GET  PUT  /clientes/{id}
GET  POST /balcoes          ?clienteId=
GET  PUT  /balcoes/{id}
GET  POST /impressoras       ?balcaoId= ?estado=
GET  PUT  /impressoras/{id}
POST      /impressoras/importar
GET  POST /balcoes/{id}/tecnicos
GET  POST /utilizadores
PUT       /utilizadores/{id}/estado
```

**Pedidos**
```
GET  /pedidos                ?estado= ?balcaoId= ?tecnicoId= ?de= ?ate=
POST /pedidos                (Idempotency-Key header)
GET  /pedidos/{id}
GET  /pedidos/{id}/eventos
POST /pedidos/{id}/atribuir
POST /pedidos/{id}/agendar
POST /pedidos/{id}/iniciar
POST /pedidos/{id}/fechar    (resumo + checklist + contador obrigatórios)
POST /pedidos/{id}/cancelar
POST /pedidos/{id}/comentarios
POST /pedidos/{id}/anexos
GET  /impressoras/{id}/pedidos
```

**Códigos de resposta relevantes:**
- `409` — transição de estado ilegal, ou pedido já aberto para a mesma máquina
- `422` — fecho sem checklist ou sem contador

---

## 9. Ecrãs a desenhar (fase actual)

Esta é a fase em curso. Precisamos de wireframes/mockups dos 8 ecrãs do MVP:

| # | Ecrã | Papel principal | Prioridade |
|---|---|---|---|
| 1 | **Login** | Todos | Alta |
| 2 | **Dashboard** | Todos (vista diferente por papel) | Alta |
| 3 | **Submeter pedido** | Cliente | Alta |
| 4 | **Lista de pedidos** | HelpDesk / Admin | Alta |
| 5 | **Detalhe do pedido** | Todos | Alta |
| 6 | **Registo de intervenção** | Técnico (móvel) | Alta |
| 7 | **Cadastro de equipamento** | Admin | Média |
| 8 | **Gestão de utilizadores** | Admin | Média |

**Próximo passo no novo chat:** começar pelo Login + Dashboard (já decidido).

---

## 10. Decisões tomadas

| # | Decisão |
|---|---|
| D1 | Submissão directa pelo cliente — sem intermediários |
| D2 | SLA de 24 horas (tempo de resposta, não de reparação) |
| D3 | Duração da reparação fora do SLA |
| D4 | Qualquer sector de actividade (não só bancos) |
| D5 | Uma intervenção por pedido |
| D6 | Pedido fora do horário → prazo começa no dia útil seguinte |
| D7 | Falta de peça → pedido fica em reparação, técnico acrescenta comentários |
| D8 | Contas individuais por pessoa (sem contas partilhadas por balcão) |
| D9 | Numeração nova (formato `PI-2026-00147`), referência antiga guardada opcionalmente |
| D10 | Contador de cópias: registo manual pelo técnico no fecho de cada intervenção |
| D11 | Checklist: lista padrão de fábrica, configurável pelo admin depois |

**Checklist padrão de fábrica:**
1. Impressão de teste OK?
2. Bandeja de papel sem encravamento?
3. Nível de toner suficiente?
4. Tambor em bom estado?
5. Cabos e ligações seguras?
6. Limpeza do interior feita?

---

## 11. Recolhas ainda pendentes (Fase 0)

| # | O quê | Responsável |
|---|---|---|
| R1 | Lista definitiva de itens da checklist de intervenção | Equipa técnica do Hélio |
| R2 | Como é feita hoje a leitura do contador e qual é a fonte oficial para facturação | Cliente |

---

## 12. Regras de negócio chave

- `RN03` — Número de pedido sequencial por ano, imutável (`PI-AAAA-NNNNN`)
- `RN04` — Fecho obrigatório: resumo + checklist + contador
- `RN05` — Nada é eliminado; tudo é desactivado
- `RN07` — Não é possível abrir segundo pedido de avaria para máquina com pedido em aberto
- `RN08` — SLA conta em horas úteis; pedido fora do horário → dia seguinte
- `RN10` — Contador novo ≥ contador anterior da mesma máquina
- `RN11` — Falta de peça → pedido fica em reparação (sem segundo pedido)
- `RN13` — Pedido fechado não reabre; problema recorrente = novo pedido

---

## 13. O que NÃO está no âmbito

- Facturação e documentos fiscais
- Gestão de stock de consumíveis
- Integração com sistemas dos clientes
- App móvel nativa (interface responsiva no browser)
- Leitura automática de contadores via SNMP
- Georreferenciação de técnicos

---

## 14. Estrutura de pastas prevista

**Backend (`printdesk-api`)**
```
src/main/java/com/printdesk/api/
├── auth/
├── parque/        ← clientes, balcões, impressoras, atribuições
├── pedido/        ← ciclo do pedido, estados, eventos, checklist, anexos
├── notificacao/   ← outbox, email, SMS
├── relatorio/
└── comum/         ← erros, auditoria, segurança, configuração
```

**Frontend (`printdesk-web`)**
```
src/
├── features/
│   ├── auth/
│   ├── pedidos/
│   ├── parque/
│   └── tecnico/   ← ecrã móvel
├── components/    ← componentes partilhados
└── lib/           ← api client, utils
```

---

*Gerado em: Setembro 2026 — PrintDesk handoff para fase de design*
