# PrintDesk — Portal de Pedidos de Intervenção (Ricotecnica)

Documentação do projecto, organizada. Começa aqui.

**Estado actual:** Fase 0 — design de ecrãs (Login + Dashboard primeiro).
**Última actualização desta organização:** 30/09/2026.

---

## O projecto em 30 segundos

A **Ricotecnica** faz outsourcing de impressão: aluga e mantém impressoras em instalações de clientes de qualquer sector. Hoje, quando uma impressora avaria, o cliente envia email, um gestor cria um "PEDIDO DE REPARAÇÃO" em papel e liga ao técnico. Ninguém sabe o estado em tempo real e nada fica registado.

O **PrintDesk** é o portal web que substitui esse processo: o cliente submete o pedido, o sistema atribui-o ao técnico responsável pelo balcão, notifica-o por SMS/email, o técnico regista a intervenção no telemóvel e tudo fica num histórico por máquina.

- **Cliente do projecto:** Hélio Matsinhe (autor do enunciado académico de 2021, Universidade Pedagógica de Maputo)
- **Stack:** Java 21 + Spring Boot 4.1.1 + MySQL 8 · React + Vite + TypeScript · JWT · Flyway
- **Repositórios:** `printdesk-api` (backend, package `com.printdesk.api`) e `printdesk-web` (frontend)
- **Prazo:** 16 semanas, 11 entregas. MVP = E0 a E6 (10 semanas, 30 000 MZN). Total 45 000 MZN.

---

## Estrutura da pasta

```
docs/
├── README.md                     ← este ficheiro: índice e contexto
├── HISTORICO.md                  ← como o projecto evoluiu e o que foi substituído
├── 01-requisitos/
│   └── Requisitos_v2.5.md        ← especificação completa (RF, RN, RNF, API, arquitectura)
├── 02-modelo-de-dados/
│   ├── modelo-de-dados.md        ← DER em texto (Mermaid) + lacunas conhecidas
│   ├── DER_Portal_Pedidos_Intervencao.svg
│   └── DER_Portal_Pedidos_Intervencao.png
├── 03-proposta-comercial/
│   ├── Proposta_Portal_Pedidos_Intervencao.docx   ← original para enviar/assinar
│   └── proposta.md                                 ← transcrição em texto
├── 04-handoff/
│   └── PrintDesk_Contexto_Handoff.md   ← resumo operacional mais recente
└── historico/                    ← versões antigas, mantidas só como registo
    ├── 2021-enunciado-original/  ← PDF do trabalho académico + transcrição
    ├── concepcao-inicial/        ← primeira revisão crítica do enunciado
    └── modelo-de-dados-v1/       ← primeiro DER (HTML/Mermaid)
```

---

## Fonte de verdade por tema

Quando dois documentos divergem, vale o da coluna "Fonte".

| Tema | Fonte | Notas |
|---|---|---|
| Requisitos funcionais e não-funcionais | [Requisitos v2.5](01-requisitos/Requisitos_v2.5.md) §5, §8 | RF01–RF36, RNF01–RNF17 |
| Regras de negócio | [Requisitos v2.5](01-requisitos/Requisitos_v2.5.md) §7 | RN01–RN13 |
| Máquina de estados | [Requisitos v2.5](01-requisitos/Requisitos_v2.5.md) §6 | 5 estados: submetido → atribuído → em reparação → fechado, + cancelado |
| Decisões com o cliente | [Handoff](04-handoff/PrintDesk_Contexto_Handoff.md) §10 | D1–D11 (a v2.5 só tem D1–D9) |
| Stack, nomes de repositórios e packages | [Handoff](04-handoff/PrintDesk_Contexto_Handoff.md) §2, §14 | Substitui a §13 da v2.5 |
| Modelo de dados | [DER](02-modelo-de-dados/modelo-de-dados.md) | 12 tabelas |
| Contrato da API | [Requisitos v2.5](01-requisitos/Requisitos_v2.5.md) §14 | O handoff tem a mesma lista, resumida |
| Entregas, prazos e valores | [Proposta](03-proposta-comercial/proposta.md) | E0–E11 |
| Ecrãs do MVP | [Handoff](04-handoff/PrintDesk_Contexto_Handoff.md) §9 | 8 ecrãs (a concepção inicial falava de 5) |
| Recolhas pendentes | [Handoff](04-handoff/PrintDesk_Contexto_Handoff.md) §11 | R1 checklist, R2 contador |

---

## Divergências entre os documentos actuais

A v2.5 e o handoff concordam em quase tudo. Onde não concordam, o handoff é mais recente:

| Ponto | Requisitos v2.5 | Handoff (vale este) |
|---|---|---|
| Spring Boot | 3.x | 4.1.1 |
| Package Java | `com.ricotecnica.portal` | `com.printdesk.api` |
| Módulos backend | inclui `utilizador` e `manutencao` | não os lista (provavelmente dentro de `auth`/`parque`/`pedido`, a confirmar) |
| Decisões | D1–D9 | D1–D11 (D10 contador manual, D11 checklist padrão de fábrica) |
| Nome do produto | "Portal de Pedidos de Intervenção" | PrintDesk |

## Pontos em aberto

**Recolhas com o cliente (Fase 0)**
- R1: lista definitiva da checklist (o D11 permite arrancar com a lista padrão de 6 itens)
- R2: como se lê hoje o contador e qual é a fonte oficial para facturação

**Decisões técnicas do lado do prestador**
- Alojamento (VPS, nuvem ou servidor próprio)
- Agregador de SMS em Moçambique (tem prazo de contratação)
- Definition of Done
- Desenvolvimento sozinho ou em equipa

**Outros**
- A proposta não diz se já foi assinada. O handoff indica que se está a trabalhar na Fase 0, o que sugere adjudicação; confirmar.
- A proposta não tem cláusulas de rescisão, propriedade intelectual nem confidencialidade. Bastam para contexto académico; para implantação real, não.
- Lacunas no modelo de dados: ver [modelo-de-dados.md](02-modelo-de-dados/modelo-de-dados.md#lacunas-conhecidas).
