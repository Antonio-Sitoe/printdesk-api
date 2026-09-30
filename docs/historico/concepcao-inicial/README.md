> ⚠️ **Documento histórico.** Substituído pelos [Requisitos v2.5](../../01-requisitos/Requisitos_v2.5.md) e pelo [Handoff](../../04-handoff/PrintDesk_Contexto_Handoff.md). Ver [HISTORICO.md](../../HISTORICO.md) para o que mudou.

# Concepção — portal de avarias de impressoras

Base de concepção a partir da revisão do enunciado. O enunciado observa bem o problema e lista requisitos ainda crus. O que falta é a fase de concepção: corrigir o modelo antes de desenhar, senão os erros contaminam o resto.

## Índice

1. [Correcções conceptuais](01-correcoes-conceptuais.md) — papéis, balcão e histórico
2. [Modelo de dados e estados](02-modelo-de-dados-e-estados.md) — atribuição, eventos e máquina de estados
3. [Requisitos em falta e classificação](03-requisitos.md) — o que o enunciado omite e o que está mal arrumado
4. [Faseamento](04-faseamento.md) — F1, F2 e F3
5. [Estado e prioridades](05-estado-e-prioridades.md) — o que está feito, o que bloqueia e a sequência até ao primeiro código

## O que este conjunto fixa

- `Administrador`, `Cliente`, `Tecnico Impressora` e `Tecnico HelpDesk` são papéis de um `Utilizador`, não classes.
- `Balcão` / `Agência` é a entidade que falta para saber quem é responsável por cada máquina.
- O histórico é uma tabela de eventos append-only. O estado do pedido é a projecção do último evento.
- O ciclo mínimo do pedido vai de `submetido` a `fechado`, com ramos `cancelado` e `reaberto`.
- Notificação, uso em campo, contadores, anexos e medição de SLA entram como requisitos, não como detalhes de interface.

## Em aberto

O desbloqueador real é a assinatura da proposta pelo Hélio. Até lá, o que se pode adiantar está em [Estado e prioridades](05-estado-e-prioridades.md): wireframes dos 5 ecrãs, repositório base Java/Spring Boot + React, ou contrato formal com cláusulas simples.

O diagrama entidade-relacionamento existe no projecto, mas não foi colado neste conjunto: em [Modelo de dados e estados](02-modelo-de-dados-e-estados.md) ficam só as entidades que o texto de concepção nomeia.
