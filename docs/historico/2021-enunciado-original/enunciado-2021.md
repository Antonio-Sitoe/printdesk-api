# Enunciado original (2021) — transcrição

> ⚠️ **Documento histórico.** Todos os requisitos abaixo foram revistos e substituídos pelos [Requisitos v2.5](../../01-requisitos/Requisitos_v2.5.md). Transcrito de [Portal para submissão de pedidos de reparação.pdf](<Portal para submissão de pedidos de reparação.pdf>). Os diagramas de casos de uso e de classes são imagens e não foram transcritos.

**Autor:** Hélio Da Costa Matsinhe · Licenciatura em Informática, Universidade Pedagógica de Maputo, 2021
**Disciplina:** Desenvolvimento de Sistemas · **Docente:** MSc Cláudia Jovo Gune

## Problema

A RICOTECNICA aluga fotocopiadoras ao BCI e a algumas agências do Standard Bank. Quando uma máquina avaria:
1. o agente do banco comunica à central do banco, que envia email à Ricotecnica;
2. o gestor de email cria o documento PEDIDO DE REPARAÇÃO e comunica a um técnico.

Falhas descritas: o servidor de email fica sobrecarregado e só o gestor de email lê a caixa. O técnico depende do HelpDesk. Não há um técnico responsável por cada balcão. Não há base de dados de avarias. A sede só sabe que uma avaria foi resolvida quando o técnico regressa ou quando alguém lhe liga.

**Impacto:** a resposta demora mais de 2 horas.

## Metodologia

Scrum, com ciclos semanais e reuniões semanais com o HelpDesk. Prazo previsto: 21 dias.

## Requisitos funcionais originais

| N.º | Requisito | Descrição |
|---|---|---|
| RF01 | Login | Ecrã de login para os vários tipos de utilizador |
| RF02 | Submissão de pedido de reparação | Quando se detecta uma avaria na fotocopiadora |
| RF03 | Manutenção preventiva | A cada 2 meses, com calendário e marcação automática |
| RF04 | Criação de utilizadores | O administrador atribui senhas e IDs e define o tipo de utilizador |
| RF05 | Fecho do pedido | O técnico regista a hora de início e de fim e um resumo. Só o técnico ou o HelpDesk fecham. |
| RF06 | Actualização do pedido | Se não for atendido em 2 h, o pedido é actualizado; na preventiva, indica-se a data prevista |
| RF07 | Fotocopiadoras | Registo com o cliente e a localização |
| RF08 | Checklist de reparação | Campo onde o técnico regista o estado da máquina |

## "Requisitos não-funcionais" originais

| N.º | Requisito | Descrição |
|---|---|---|
| RNF01 | Níveis de acesso | Administrador, Cliente, Técnico |
| RNF02 | Histórico por ID da máquina | Guardar o histórico de todos os pedidos |
| RNF03 | Estado do pedido | "Actualizado" ou "Pendente" |
| RNF04 | Horas individuais do técnico | A cada 2 meses, pedidos fechados por técnico, para o bónus |
| RNF05 | Alerta de reincidência | Mais de 2 pedidos para a mesma máquina em 15 dias, visível só para o gestor ou administrador |

## Actores (casos de uso)

- **Cliente:** submete o pedido e marca hora (opcional).
- **Administrador:** cria utilizadores, gera relatórios, actualiza ou fecha pedidos, introduz ou remove equipamento.
- **Técnico de impressora:** actualiza o estado, agenda, reporta a avaria e fecha o pedido.
- **HelpDesk:** actualiza o pedido (agenda, número do pedido) e o equipamento.
