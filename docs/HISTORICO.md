# Histórico do projecto

Como a documentação evoluiu, do mais antigo para o mais recente. As versões substituídas estão em [historico/](historico/) e servem só de registo: **não as uses como referência**.

---

## 1. Enunciado original — 2021

📁 [historico/2021-enunciado-original/](historico/2021-enunciado-original/)

Trabalho académico de Hélio Da Costa Matsinhe para a disciplina de Desenvolvimento de Sistemas (Universidade Pedagógica de Maputo, docente MSc Cláudia Jovo Gune).

- Cenário bancário: BCI e agências do Standard Bank.
- Metodologia Scrum, sprint de 21 dias.
- 8 requisitos funcionais (RF01–RF08) e 5 "não-funcionais" (RNF01–RNF05).
- Diagrama de casos de uso e diagrama de classes (as imagens não passam para texto).
- Estado do pedido: só "Actualizado" ou "Pendente".
- Impacto referido: resposta demora mais de 2 horas.

## 2. Concepção inicial — revisão crítica do enunciado

📁 [historico/concepcao-inicial/](historico/concepcao-inicial/)

Primeira análise. Identificou os três erros de fundo que se mantêm até hoje:

1. Administrador, Cliente e Técnicos são **papéis** de um único `Utilizador`, não classes.
2. Falta a entidade **Balcão**, que é quem resolve "que técnico é responsável por que máquina".
3. O histórico tem de ser uma tabela de **eventos append-only**, não sobreposição de estado.

Também acrescentou notificações, uso em campo, contador de cópias, anexos e medição de SLA, e reclassificou os RNF originais como funcionais.

**O que ficou ultrapassado:**

| Concepção inicial | Substituído por (v2.5 / handoff) |
|---|---|
| SLA de 2 horas | SLA de 24 horas úteis até ao início da intervenção (D2) |
| 8 estados: submetido, atribuído, agendado, em curso, resolvido, fechado, cancelado, reaberto | 5 estados. "Agendado" passou a ser campo; não há reabertura (RN13) |
| Faseamento F1/F2/F3 | Entregas E0–E11 em 4 fases |
| 5 ecrãs do MVP | 8 ecrãs |
| SMS ou WhatsApp | Email + SMS |
| Contexto bancário | Qualquer sector (D4) |

O ficheiro `05-estado-e-prioridades.md` já é posterior à v2.5 e à proposta. As prioridades que lá estão passaram para o handoff e para o [README](README.md#pontos-em-aberto).

## 3. Modelo de dados v1

📁 [historico/modelo-de-dados-v1/](historico/modelo-de-dados-v1/)

Primeiro DER, num HTML com Mermaid. Tinha 9 tabelas e chaves primárias `uuid`.

**Substituído** pelo [DER actual](02-modelo-de-dados/modelo-de-dados.md), que acrescenta:
- as tabelas `anexo`, `consumivel_usado` e `checklist_item` (a checklist passou de texto livre a lista configurável);
- `resolvido_em` e `referencia_antiga` em `pedido`, e `estado_anterior`/`estado_novo` em `pedido_evento`;
- o horário de contagem do SLA em `cliente`;
- o `palavra_passe_hash` do utilizador;
- chave interna numérica mais `uuid` público, em vez de `uuid` como PK.

## 4. Requisitos v2 → v2.5

📁 [01-requisitos/](01-requisitos/)

Reescrita completa do enunciado: glossário, critérios de sucesso mensuráveis, matriz de permissões, RF01–RF36, RN01–RN13, RNF01–RNF17, arquitectura Java + React e contrato da API. As decisões D1–D9 foram fechadas com o cliente.

Só a versão 2.5 foi guardada (havia 8 cópias idênticas espalhadas pelos Downloads). Não há cópia das versões intermédias.

## 5. Proposta comercial

📁 [03-proposta-comercial/](03-proposta-comercial/)

11 entregas com valores (45 000 MZN no total, MVP por 30 000 MZN), pagamento por entrega aceite e custos recorrentes. Alinhada com a secção 11 da v2.5.

## 6. Handoff PrintDesk — Setembro 2026 (mais recente)

📁 [04-handoff/](04-handoff/)

Resumo operacional para continuar o trabalho noutro chat. Novidades face à v2.5:
- nome do produto: **PrintDesk**; repositórios `printdesk-api` e `printdesk-web`;
- Spring Boot 4.1.1 e package `com.printdesk.api`;
- decisões D10 (contador manual no fecho) e D11 (checklist padrão de fábrica com 6 itens);
- lista dos 8 ecrãs a desenhar e a ordem: Login + Dashboard primeiro.

---

## Limpeza feita em 30/09/2026

Ficheiros duplicados **eliminados** (conteúdo idêntico, verificado por hash MD5):

| Ficheiro | Cópias removidas |
|---|---|
| `Requisitos_Portal_Pedidos_Intervencao_Ricotecnica_v2.md` | 6 (3 na raiz com sufixo (1)(2)(3), mais as de `files (1)`, `files (2)`, `files (3)`) |
| `PrintDesk_Contexto_Handoff.md` | 1 (`(1)`) |
| `Proposta_Portal_Pedidos_Intervencao.docx` | 2 (`files (1)`, `files (3)`) |
| `DER_Portal_Pedidos_Intervencao.png` | 2 (`files (2)`, `files (3)`) |

As pastas `files`, `files (1)`, `files (2)` e `files (3)` ficaram vazias e foram removidas.
