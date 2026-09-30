# Portal de Submissão de Pedidos de Intervenção — Ricotecnica

**Documento de Requisitos — versão 2.5**
Revisão e reestruturação do documento original (2021)
Stack alvo: Java (Spring Boot) + React + MySQL

---

## 1. Introdução

### 1.1. Contexto

A Ricotecnica presta serviços de outsourcing de impressão, alugando e mantendo impressoras multifuncionais em instalações de clientes de qualquer sector — bancos, seguradoras, escritórios, instituições públicas. O parque instalado é geograficamente disperso por vários locais de cada cliente, e a manutenção correctiva e preventiva desse parque é feita por uma equipa técnica de campo coordenada por um departamento de HelpDesk.

O processo actual de reporte e resolução de avarias assenta em email e chamadas telefónicas. Este documento especifica o sistema que o substitui.

### 1.2. Âmbito do documento

Este documento define **o que** o sistema deve fazer e sob que restrições. Não define arquitectura de código, desenho de interface nem plano de implementação — esses são objecto de documentos próprios.

### 1.3. Glossário

| Termo | Definição |
|---|---|
| **Cliente** | Organização com contrato de outsourcing, de qualquer sector de actividade. Não é uma pessoa. |
| **Balcão** | Instalação física de um cliente onde estão alocadas impressoras. Consoante o cliente, pode ser uma agência, uma dependência, um piso ou um escritório. Um cliente pequeno pode ter um único balcão. |
| **Utilizador Cliente** | Pessoa do cliente, associada a um ou mais balcões, com permissão para submeter pedidos. Submete directamente, sem intermediários. |
| **Pedido** | Registo de uma solicitação de intervenção sobre uma impressora. Substitui o documento "PEDIDO DE REPARAÇÃO" em papel/email. |
| **Intervenção** | Deslocação e trabalho efectivo do técnico sobre a máquina. Cada pedido corresponde a uma única intervenção. |
| **SLA** | Tempo máximo contratado entre a submissão de um pedido e o início da intervenção. Por omissão 24 horas. Não abrange a duração da reparação em si. |
| **Contador** | Leitura do número acumulado de cópias/impressões da máquina. Base da facturação por cópia. |

---

## 2. Objectivos e critérios de sucesso

### 2.1. Objectivos

1. Eliminar o email e o telefone como canais de reporte de avarias.
2. Dar ao técnico de campo acesso directo e autónomo aos pedidos que lhe estão atribuídos, sem depender do HelpDesk para saber onde tem de ir a seguir.
3. Dar ao cliente visibilidade do estado do seu pedido sem ter de telefonar.
4. Constituir um histórico consultável por máquina, por balcão e por técnico.
5. Passar a medir o cumprimento do SLA em vez de o estimar.

### 2.2. Critérios de sucesso (mensuráveis)

| Indicador | Situação actual | Meta |
|---|---|---|
| Tempo médio entre submissão e início da intervenção | Não medido | ≤ 24 horas em 90% dos pedidos |
| Tempo entre conclusão da reparação e conhecimento na sede | Depende do regresso ou de chamada telefónica | ≤ 5 minutos (registo no local) |
| Pedidos com técnico responsável identificado | Não registado | 100% |
| Máquinas com histórico completo de intervenções | 0% | 100% ao fim de 6 meses |

> **Nota:** os valores da coluna "situação actual" são estimativas herdadas do documento original. Antes do arranque, devem ser medidos durante 2 a 4 semanas sobre o processo de email, ou os ganhos do sistema não serão demonstráveis.

---

## 3. Descrição do problema

### 3.1. Processo actual

1. A impressora avaria num balcão do cliente.
2. O cliente envia email para a Ricotecnica.
3. O gestor de email da Ricotecnica cria o documento PEDIDO DE REPARAÇÃO.
4. O gestor de email telefona ao técnico responsável.
5. O técnico desloca-se ao balcão e repara.
6. A Ricotecnica só sabe que a avaria foi resolvida quando o técnico regressa à sede ou quando alguém lhe liga a perguntar.

Não existe nenhuma camada intermediária entre o balcão e a Ricotecnica: é o próprio cliente que reporta.

### 3.2. Falhas identificadas

| # | Falha | Consequência |
|---|---|---|
| P1 | Ponto único de passagem no gestor de email | O processo pára quando essa pessoa está ausente ou sobrecarregada |
| P2 | O técnico não tem acesso à fila de pedidos | Depende de chamada telefónica para saber a próxima intervenção |
| P3 | Não há técnico responsável por balcão | Vários técnicos intervêm na mesma máquina, sem continuidade de diagnóstico |
| P4 | Não existe base de dados de intervenções | Impossível saber quantas vezes uma máquina avariou nem porquê |
| P5 | Fecho da intervenção não é comunicado em tempo real | A sede opera sem informação e gasta tempo a perseguir técnicos |
| P6 | Servidor de email sobrecarregado | Risco de pedidos perdidos ou atrasados |
| P7 | Prazo de resposta não é medido | Incumprimentos do SLA só são detectados por queixa do cliente |

### 3.3. Síntese

| | |
|---|---|
| **Problema** | Ausência de plataforma para submissão e acompanhamento de pedidos de intervenção |
| **Afecta** | Utilizadores dos clientes de outsourcing, equipa técnica, HelpDesk, gestão |
| **Impacto** | Prazo de resposta não medido nem garantido, equipamento alugado indisponível, ausência de dados de gestão |
| **Solução** | Portal com submissão, atribuição, notificação, actualização em tempo real e histórico permanente |

---

## 4. Actores e permissões

### 4.1. Actores

| Actor | Descrição |
|---|---|
| **Utilizador Cliente** | Colaborador do cliente, associado a um ou mais balcões. Submete os pedidos directamente. |
| **Técnico HelpDesk** | Recebe, triagem e atribui pedidos; agenda intervenções |
| **Técnico de Impressora** | Técnico de campo; executa e regista intervenções |
| **Administrador** | Gere utilizadores, clientes, balcões, parque de equipamento e relatórios |

**Princípio de modelação:** existe **uma única entidade Utilizador**. Os actores acima são *papéis* atribuídos a um utilizador, não tipos de pessoa distintos. Um utilizador pode acumular papéis (um chefe de equipa pode ser Técnico e HelpDesk).

### 4.2. Matriz de permissões

| Acção | Cliente | Técnico | HelpDesk | Admin |
|---|:---:|:---:|:---:|:---:|
| Submeter pedido | ✓ | ✓ | ✓ | ✓ |
| Ver pedidos dos seus balcões | ✓ | — | — | — |
| Ver pedidos que lhe estão atribuídos | — | ✓ | — | — |
| Ver todos os pedidos | — | — | ✓ | ✓ |
| Atribuir pedido a técnico | — | — | ✓ | ✓ |
| Agendar hora de intervenção | — | ✓ | ✓ | ✓ |
| Registar início/fim da intervenção | — | ✓ | — | ✓ |
| Fechar pedido | — | ✓ | ✓ | ✓ |
| Cancelar pedido | ✓* | — | ✓ | ✓ |
| Criar/editar utilizadores | — | — | — | ✓ |
| Inserir/remover equipamento | — | — | ✓ | ✓ |
| Gerir clientes e balcões | — | — | — | ✓ |
| Consultar relatórios | ✓** | — | ✓ | ✓ |

`*` apenas o próprio autor do pedido, e apenas antes do início da intervenção.
`**` apenas relatórios respeitantes aos seus balcões.

---

## 5. Requisitos funcionais

### 5.1. Autenticação e utilizadores

| N.º | Requisito | Descrição | Prioridade |
|---|---|---|---|
| RF01 | Autenticação | Autenticação por email e palavra-passe, sessão única por conta. Bloqueio temporário após 5 tentativas falhadas. | Deve |
| RF02 | Criação de utilizadores por convite | O Administrador regista o utilizador (nome, email, papel, cliente/balcões associados) e o sistema envia convite. Cada pessoa tem conta individual; não há contas partilhadas por balcão. **A palavra-passe é definida pelo próprio utilizador**, nunca atribuída pelo administrador. | Deve |
| RF03 | Recuperação de palavra-passe | Fluxo autónomo de reposição por email, com token de validade limitada. | Deve |
| RF04 | Desactivação de utilizadores | Contas são desactivadas, nunca eliminadas, para preservar a integridade do histórico. | Deve |

> **Alteração face ao documento original:** o RF04 original previa que o administrador atribuísse senhas e IDs manualmente. É uma prática insegura (a senha circula em canal legível e é conhecida por terceiros) e insustentável à escala do parque. Substituído pelo mecanismo de convite.

### 5.2. Parque de equipamento

| N.º | Requisito | Descrição | Prioridade |
|---|---|---|---|
| RF05 | Gestão de clientes | Registo de clientes de outsourcing com o respectivo SLA contratado em horas (valor por omissão: 24) e horário de contagem. | Deve |
| RF06 | Gestão de balcões | Registo de balcões por cliente, com designação e localização. Um pedido é sempre imputável a um balcão. | Deve |
| RF07 | Gestão de impressoras | Registo de impressoras com marca/modelo, número de série, número interno, balcão de alocação e estado (activa, em reparação, retirada). | Deve |
| RF08 | Transferência de equipamento | Uma impressora pode ser movida entre balcões, mantendo o histórico associado à máquina. | Deve |
| RF09 | Atribuição técnico–balcão | Cada balcão tem um técnico principal atribuído e, opcionalmente, técnicos suplentes. Resolve a falha P3. | Deve |
| RF10 | Importação inicial do parque | Carregamento do parque existente por ficheiro (CSV/Excel), com validação e relatório de erros. | Deve |

### 5.3. Ciclo de vida do pedido

| N.º | Requisito | Descrição | Prioridade |
|---|---|---|---|
| RF11 | Submissão de pedido | O utilizador selecciona a impressora do seu balcão, escolhe o tipo de pedido (avaria, manutenção preventiva, visita de cortesia), descreve o sintoma e submete. O sistema gera um número de pedido sequencial. | Deve |
| RF12 | Anexos | Possibilidade de anexar fotografias (do painel de erro, do estado da máquina) na submissão e durante a intervenção. Máx. 5 ficheiros, 5 MB cada. | Deve |
| RF13 | Atribuição | Na submissão, o pedido é atribuído automaticamente ao técnico principal do balcão. O HelpDesk pode reatribuir a qualquer momento. | Deve |
| RF14 | Agendamento | O técnico ou o HelpDesk indica a data e hora previstas de intervenção. É um campo do pedido, alterável a qualquer momento, e não altera o estado. | Deve |
| RF15 | Actualização de pedido | Qualquer alteração (novo comentário, reagendamento, mudança de técnico, mudança de estado) gera um registo de evento. Nenhuma actualização substitui informação anterior. | Deve |
| RF16 | Registo de intervenção | No local, o técnico regista o início — o pedido passa a em reparação. Ao concluir, regista o fim e um resumo do trabalho realizado, e o pedido passa a fechado. O resumo é obrigatório. | Deve |
| RF17 | Checklist de intervenção | Preenchimento de uma checklist do estado da máquina no fecho da intervenção. Os itens da checklist são configuráveis pelo Administrador. | Deve |
| RF18 | Leitura de contador | Registo obrigatório do contador de cópias no fecho de cada intervenção. | Deve |
| RF19 | Consumíveis e peças | Registo dos consumíveis e peças substituídos na intervenção. | Deveria |
| RF20 | Fecho de pedido | Só o Técnico ou o HelpDesk podem fechar um pedido. O Cliente não tem essa permissão. | Deve |
| RF21 | Cancelamento | Um pedido pode ser cancelado com justificação obrigatória, antes do início da intervenção. | Deve |
| RF22 | Consulta de estado | O Cliente consulta em qualquer momento o estado e o histórico dos pedidos dos seus balcões. | Deve |

### 5.4. Manutenção preventiva

| N.º | Requisito | Descrição | Prioridade |
|---|---|---|---|
| RF24 | Plano de manutenção | Cada impressora tem uma periodicidade de manutenção preventiva configurável, com valor por omissão de 2 meses. | Deve |
| RF25 | Geração automática | O sistema gera automaticamente o pedido de manutenção preventiva na data prevista, atribuindo-o ao técnico do balcão. | Deve |
| RF26 | Calendário de manutenções | Vista de calendário das manutenções previstas por técnico e por balcão. | Deveria |

### 5.5. Notificações

> **Secção nova.** É a lacuna mais grave do documento original: um portal que exige que alguém se lembre de o consultar não resolve as falhas P2 e P5 — apenas muda o sítio onde a informação fica parada.

| N.º | Requisito | Descrição | Prioridade |
|---|---|---|---|
| RF27 | Notificação de atribuição | O técnico é notificado imediatamente quando um pedido lhe é atribuído, com balcão, máquina e sintoma. | Deve |
| RF28 | Notificação de fecho | O HelpDesk e o autor do pedido são notificados no fecho da intervenção. | Deve |
| RF29 | Alerta de SLA em risco | Alerta ao HelpDesk quando um pedido atinge 75% do SLA contratado sem intervenção iniciada (com SLA de 24 horas, ao fim de 18 horas úteis). | Deve |
| RF30 | Canais | Notificação por email e por SMS. O SMS é obrigatório para o técnico de campo — a fiabilidade do email em mobilidade não é suficiente. Canal configurável por utilizador. | Deve |
| RF31 | Alerta de reincidência | Alerta ao Administrador e ao HelpDesk quando a mesma máquina gera mais de 2 pedidos de avaria em 15 dias. Visível apenas para estes papéis. | Deve |

### 5.6. Relatórios

| N.º | Requisito | Descrição | Prioridade |
|---|---|---|---|
| RF32 | Histórico por máquina | Lista completa de pedidos de uma impressora, com datas, técnicos, sintomas, trabalho realizado e contadores. | Deve |
| RF33 | Cumprimento de SLA | Relatório por cliente e por período: número de pedidos, tempo médio de resposta, percentagem dentro do SLA. | Deve |
| RF34 | Produtividade por técnico | Relatório bimestral de pedidos executados e fechados por técnico, para efeitos de cálculo de bónus por eficiência. | Deve |
| RF35 | Máquinas problemáticas | Ranking de máquinas por número de avarias no período, com evolução dos contadores. | Deveria |
| RF36 | Exportação | Exportação de qualquer relatório em Excel e PDF. | Deveria |

---

## 6. Máquina de estados do pedido

Quatro estados no percurso normal, mais um terminal de excepção.

```
   ┌───────────┐      ┌────────────┐      ┌──────────────┐      ┌──────────┐
   │ submetido │ ───→ │ atribuído  │ ───→ │ em reparação │ ───→ │ fechado  │
   └─────┬─────┘      └─────┬──────┘      └──────────────┘      └──────────┘
         │                  │
         └────────┬─────────┘
                  ↓
           ┌─────────────┐
           │  cancelado  │
           └─────────────┘
```

| Estado | Significado | Transições permitidas |
|---|---|---|
| `submetido` | Registado, ainda sem técnico | → atribuído, cancelado |
| `atribuído` | Técnico responsável definido; pode ter data prevista | → em reparação, cancelado |
| `em reparação` | Técnico iniciou a intervenção | → fechado |
| `fechado` | Trabalho concluído e resumido | — (terminal) |
| `cancelado` | Anulado com justificação | — (terminal) |

**O estado `em reparação` absorve toda a duração do trabalho.** Se faltar uma peça, se for preciso encomendar material ou se o técnico tiver de voltar noutro dia, o pedido mantém-se em reparação e o técnico acrescenta comentários com o ponto de situação. Não há estados de suspensão nem pedidos duplicados: uma avaria é um pedido, do princípio ao fim.

**O agendamento é um campo, não um estado.** A data prevista de intervenção fica no campo `agendado_para` e pode ser alterada as vezes que forem necessárias sem mudar o estado do pedido.

**Marcos temporais medidos:**

- **Tempo de resposta** = `em reparação` − `submetido` → medido contra o SLA de 24 horas (RF33). É o único indicador com prazo contratual.
- **Tempo de reparação** = `fechado` − `em reparação` → registado e reportado, sem prazo associado. A reparação demora o que tiver de demorar.

> **Alteração face ao documento original:** o RNF03 original definia apenas dois estados ("Actualizado" ou "Pendente"). Com dois estados não é possível distinguir um pedido que aguarda técnico de um pedido cujo técnico já está a trabalhar na máquina, o que torna impossível medir o prazo de resposta.

---

## 7. Regras de negócio

| N.º | Regra |
|---|---|
| RN01 | Um pedido está sempre associado a exactamente uma impressora. Uma avaria em duas máquinas gera dois pedidos. |
| RN02 | Um Utilizador Cliente só vê pedidos e equipamento dos balcões a que está associado. |
| RN03 | O número de pedido é sequencial por ano, imutável e nunca reutilizado (formato `PI-2026-00147`). Pedidos migrados do processo anterior guardam a numeração antiga em `referencia_antiga`, sem interferir na sequência nova. |
| RN04 | Um pedido não pode ser fechado sem resumo do trabalho realizado, checklist preenchida e leitura de contador. |
| RN05 | Nenhum registo de pedido é eliminado. Cancelamento e desactivação são estados, não remoções. |
| RN06 | A hora de fim de uma intervenção não pode ser anterior à hora de início. |
| RN07 | Não é permitido submeter um novo pedido de avaria para uma impressora que já tem um pedido em aberto — o sistema encaminha para o pedido existente. |
| RN08 | O SLA é de 24 horas por omissão, contado em horas úteis definidas por contrato e não em horas de calendário. Pedidos submetidos fora do horário laboral começam a contar na abertura do dia útil seguinte: um pedido submetido às 17h00 conta a partir da manhã seguinte. |
| RN09 | O técnico principal do balcão tem prioridade na atribuição automática; na sua ausência, o pedido fica em `submetido` para triagem manual do HelpDesk. |
| RN10 | O contador de cópias registado numa intervenção não pode ser inferior ao registado na intervenção anterior da mesma máquina. |
| RN11 | Um pedido acompanha a avaria do princípio ao fim. Falta de peça, encomenda de material ou nova deslocação do técnico não geram novo pedido: o pedido permanece em reparação e o técnico regista o ponto de situação em comentário. |
| RN12 | O SLA mede apenas o tempo até ao início da intervenção. A duração da reparação é registada e reportada, mas não está sujeita a prazo contratual. |
| RN13 | Um pedido fechado não é reaberto. Se o problema voltar, abre-se novo pedido — que é precisamente o que alimenta o alerta de reincidência (RF31). |

---

## 8. Requisitos não-funcionais

> **Nota metodológica:** os RNF01 a RNF05 do documento original eram, na sua maioria, requisitos funcionais mal classificados (níveis de acesso, histórico, estado, relatórios, alertas). Foram reclassificados como RF acima. Os requisitos não-funcionais verdadeiros — os que descrevem *qualidades* do sistema e não *funções* — estavam ausentes e são estabelecidos aqui.

### 8.1. Disponibilidade e desempenho

| N.º | Requisito | Critério |
|---|---|---|
| RNF01 | Disponibilidade | 99% em horário laboral (07h00–19h00, dias úteis) |
| RNF02 | Tempo de resposta | Qualquer ecrã carrega em menos de 3 segundos numa ligação móvel de 3G |
| RNF03 | Capacidade | 200 utilizadores registados e 30 sessões concorrentes na fase 1, escalável a 1000/100 sem alteração de arquitectura |
| RNF04 | Volume | 500 pedidos/mês na fase 1; o modelo deve suportar 5 anos de histórico sem degradação de consulta |

### 8.2. Usabilidade e acesso

| N.º | Requisito | Critério |
|---|---|---|
| RNF05 | Utilização em campo | Interface responsiva, utilizável num telemóvel de gama média. O registo de início e fim de intervenção é a operação mais frequente e deve estar a um toque do ecrã inicial do técnico. |
| RNF06 | Tolerância a rede fraca | O formulário de registo de intervenção não pode perder dados por falha de ligação; submissão com reenvio automático. |
| RNF07 | Idioma | Interface integralmente em português. |
| RNF08 | Curva de aprendizagem | Um agente de balcão submete o primeiro pedido sem formação prévia, apenas com o guia de uma página. |

### 8.3. Segurança

| N.º | Requisito | Critério |
|---|---|---|
| RNF09 | Transporte | Todo o tráfego em HTTPS. |
| RNF10 | Palavras-passe | Armazenadas com hash (bcrypt/argon2). Mínimo 8 caracteres. |
| RNF11 | Isolamento de dados | Um cliente nunca acede a dados de outro cliente. Verificação obrigatória ao nível da consulta, não apenas da interface. |
| RNF12 | Registo de auditoria | Todas as acções sobre pedidos, utilizadores e equipamento ficam registadas com autor, data/hora e valores alterados. |
| RNF13 | Sessão | Terminação automática após 8 horas de inactividade. |

### 8.4. Dados e manutenção

| N.º | Requisito | Critério |
|---|---|---|
| RNF14 | Retenção | Histórico de pedidos retido no mínimo 5 anos, alinhado com a duração dos contratos de outsourcing. |
| RNF15 | Cópias de segurança | Backup diário automático da base de dados, com teste de restauro trimestral. |
| RNF16 | Fuso horário | Todos os registos temporais armazenados em UTC e apresentados em CAT (UTC+2). |
| RNF17 | Integração futura | O modelo deve permitir exportação de contadores e intervenções para o sistema de facturação, sem redesenho. |

---

## 9. Modelo de domínio

Entidades principais e razão de existir:

| Entidade | Papel |
|---|---|
| `utilizador` | Pessoa autenticável. Papel como atributo, não como subclasse. |
| `cliente` | Organização contratante, de qualquer sector. Define o SLA e o horário de contagem. |
| `balcao` | Instalação física. Unidade de atribuição de técnicos e de visibilidade do cliente. Um cliente com um único local tem um único balcão. |
| `impressora` | Equipamento alocado a um balcão. Detentor do histórico. |
| `atribuicao` | Ligação técnico–balcão, com indicação de principal ou suplente. |
| `pedido` | Registo da solicitação. Estado actual e marcos temporais. |
| `pedido_evento` | Registo append-only de tudo o que aconteceu ao pedido. Fonte de verdade do histórico. |
| `checklist_item` / `checklist_resposta` | Modelo configurável e respostas por intervenção. |
| `plano_manutencao` | Periodicidade e próxima data por máquina. |
| `anexo` | Ficheiros associados a pedido ou evento. |
| `consumivel_usado` | Peças e consumíveis por intervenção. |

**Princípios de modelação a respeitar:**

1. Papéis de utilizador são atributos, não entidades separadas — o modelo original criava quatro tabelas de pessoas quase idênticas.
2. `balcao` é entidade de primeira classe — sem ela, as falhas P3 e P4 não são resolúveis.
3. O estado do pedido é uma projecção do último evento, nunca a única fonte de informação.
4. Acções ("gerar relatório", "actualizar pedido") não são entidades — são operações. O diagrama de classes original modelava-as como tais.
5. O modelo é independente da tecnologia: mantém-se inalterado com Java/JPA. Muda apenas o mapeamento, tratado na secção 13.4.

---

## 10. Fora de âmbito

Explicitamente **não** incluído nesta especificação:

- Facturação e emissão de documentos fiscais.
- Gestão de stock de consumíveis e peças (regista-se o consumo, não se gere o armazém).
- Monitorização automática das impressoras por SNMP ou agente de rede.
- Aplicação móvel nativa (a interface responsiva serve a fase 1).
- Integração com o sistema de gestão do cliente.
- Georreferenciação e optimização de rotas dos técnicos.

---

## 11. Faseamento e entregas

O trabalho é dividido em entregas, cada uma correspondendo a um conjunto de funcionalidades que fica a funcionar por si. É esta divisão que serve de base ao cronograma e à facturação da proposta comercial.

### Fase 0 — Arranque (2 semanas)

| Entrega | Âmbito |
|---|---|
| E0 | Requisitos validados, protótipo dos ecrãs aprovado, modelo de dados fechado e ambiente preparado |

### Fase 1 — Núcleo operacional (8 semanas)

| Entrega | Âmbito | Requisitos |
|---|---|---|
| E1 | Autenticação, contas individuais e perfis de acesso | RF01–RF04 |
| E2 | Cadastro de clientes, balcões e impressoras, com importação do parque | RF05–RF08, RF10 |
| E3 | Atribuição de técnicos por balcão e submissão do pedido pelo cliente | RF09, RF11, RF12 |
| E4 | Ciclo do pedido: atribuição, agendamento, fecho e histórico de eventos | RF13–RF16, RF20, RF21 |
| E5 | Ecrã móvel do técnico para registo da intervenção no local | RF16, RNF05, RNF06 |
| E6 | Notificações por email e SMS, e consulta de estado pelo cliente | RF22, RF27, RF28, RF30, RF32 |

Objectivo: substituir integralmente o email como canal de pedidos.

### Fase 2 — Controlo e prevenção (4 semanas)

| Entrega | Âmbito | Requisitos |
|---|---|---|
| E7 | Manutenção preventiva automática com calendário | RF24–RF26 |
| E8 | Checklist de intervenção, contador de cópias e consumíveis | RF17–RF19 |
| E9 | Alerta de reincidência e medição do prazo de resposta | RF29, RF31, RF33 |

Objectivo: manutenção preventiva automática e medição do SLA de 24 horas.

### Fase 3 — Gestão (2 semanas)

| Entrega | Âmbito | Requisitos |
|---|---|---|
| E10 | Relatórios de prazos, produtividade por técnico e máquinas problemáticas | RF33–RF35 |
| E11 | Exportação em Excel e PDF | RF36, RNF17 |

Objectivo: relatórios de gestão, bónus por eficiência e preparação da integração com facturação.

---

## 12. Decisões em aberto

Nenhuma linha de código deve ser escrita antes de estas questões estarem fechadas. Cada uma delas altera o desenho do sistema, não apenas o seu conteúdo.

### 12.1. Pressupostos assumidos

- Os balcões têm acesso à internet com fiabilidade suficiente para submissão de pedidos.
- Os técnicos de campo dispõem de telemóvel com acesso a dados.
- O parque de equipamento existe em formato digital passível de importação.
- O envio de SMS será feito através de um agregador local, com custo por mensagem.

Se algum destes pressupostos for falso, deve ser tratado como uma questão adicional abaixo.

### 12.2. Decisões já fechadas com o cliente

**D1 — Quem submete o pedido.** *Fechado: submissão directa pelo cliente.*
Não existe nenhuma camada intermediária entre o balcão e a Ricotecnica. É o próprio utilizador do cliente que submete o pedido no portal. Não há papel de "central" nem submissão em nome de terceiros.
*Consequência:* o modelo de actores da secção 4 mantém-se com quatro papéis, sem acrescentos. O ponto único de falha no gestor de email desaparece por completo.

**D2 — Prazo de resposta contratado.** *Fechado: 24 horas.*
O SLA é de 24 horas entre a submissão do pedido e o início da intervenção. É este o único indicador sujeito a prazo contratual.
*Consequência:* alimenta o RF29 (alerta a 75%, ou seja, às 18 horas) e o RF33 (relatório de cumprimento). O valor fica configurável por cliente, com 24 horas como valor por omissão, para acomodar contratos futuros com prazos diferentes.

**D3 — Duração da reparação.** *Fechado: fora do SLA.*
A reparação em si demora mais do que o prazo de resposta e varia com a natureza da avaria. É registada e reportada, mas não está sujeita a prazo contratual.
*Consequência:* regra RN12. O relatório de cumprimento (RF33) mede o tempo de resposta; o tempo de reparação aparece no relatório de produtividade (RF34) como informação, não como incumprimento.

**D4 — Perfil de cliente.** *Fechado: qualquer sector de actividade.*
O sistema não é específico do sector bancário. Serve qualquer cliente com contrato de outsourcing de impressão.
*Consequência:* remoção de todos os pressupostos bancários do documento. O conceito de "balcão" passa a designar qualquer instalação física — agência, escritório, piso ou armazém — e um cliente pequeno pode ter um único balcão.

**D5 — Intervenções por pedido.** *Fechado: uma intervenção por pedido.*
Cada pedido corresponde a uma única deslocação e intervenção do técnico.
*Consequência:* não existe entidade `intervencao` separada — os marcos temporais ficam no próprio pedido, tal como no modelo da secção 9.

**D6 — Contagem do prazo fora do horário.** *Fechado: passa para o dia útil seguinte.*
Um pedido submetido às 17h00 não consome horas de encerramento: o prazo começa a contar na abertura do dia útil seguinte e vence no fim desse dia.
*Consequência:* regra RN08. Basta registar o horário laboral do cliente; não é preciso acumular horas úteis fraccionadas.

**D7 — Falta de peça ou intervenção prolongada.** *Fechado: o pedido fica em reparação.*
Não há estado de suspensão nem abertura de segundo pedido. O pedido permanece em reparação enquanto o trabalho durar, e o técnico regista comentários com o ponto de situação.
*Consequência:* regra RN11 e máquina de estados reduzida a cinco estados (secção 6).

**D8 — Contas de utilizador do cliente.** *Fechado: conta individual por pessoa.*
Cada colaborador do cliente que submete pedidos tem a sua própria conta, associada a um ou mais balcões. Não há contas partilhadas por balcão.
*Consequência:* cada pedido fica associado a quem o submeteu, o que permite esclarecer dúvidas com a pessoa certa. Implica um volume de utilizadores maior do que o de balcões, pelo que o convite por email (RF02) e a desactivação de contas (RF04) passam a ser operações frequentes e não excepcionais.

**D9 — Numeração de pedidos.** *Fechado: numeração nova, com referência antiga preservada.*
O portal usa numeração própria, sequencial por ano e imutável. Os pedidos importados do processo anterior guardam a numeração antiga num campo opcional.
*Consequência:* regra RN03. Acrescenta o campo `referencia_antiga` à entidade `pedido`, preenchido apenas nos registos migrados.

### 12.3. Recolhas a fazer na Fase 0

Não são decisões pendentes — o caminho está definido. São recolhas de informação que dependem de terceiros e que devem estar concluídas antes do Sprint 1.

**R1 — Itens da checklist de intervenção (RF17).**
A lista tem de vir da equipa técnica, não pode ser inventada. Pedir aos técnicos a lista de verificações que já fazem hoje na máquina; se não existir lista formal, recolher numa sessão de uma hora com dois ou três deles.
*Responsável:* cliente. *Prazo:* fim da Fase 0.
*Nota:* o modelo de dados suporta uma lista configurável (`checklist_item`), pelo que o desenvolvimento não fica bloqueado à espera do conteúdo definitivo — mas a formação e o arranque do piloto ficam.

**R2 — Processo actual de leitura do contador de cópias (RF18).**
Independentemente do que existir, o contador passa a ser registado em cada intervenção. O que falta apurar é se já existe outro processo de recolha e, havendo, qual dos dois é a fonte oficial para facturação.
*Responsável:* cliente. *Prazo:* fim da Fase 0.
*Nota:* se ambos ficarem activos sem definição de fonte oficial, os dois conjuntos de leituras vão divergir e a facturação passa a depender de qual deles alguém consultou.

### 12.4. Decisões técnicas do lado do prestador

Não dependem do cliente, mas devem estar fechadas antes do Sprint 1:

- Agregador de SMS a utilizar em Moçambique, com custo por mensagem e prazo de contratação.
- Alojamento do sistema: servidor próprio, VPS ou nuvem.
- Repositório único ou separado para o backend Java e o frontend React.
- Definição de Pronto (Definition of Done) acordada com a equipa.

---

## 13. Arquitectura técnica — Java + React

### 13.1. Componentes

| Camada | Tecnologia | Observações |
|---|---|---|
| Backend | Java 21 + Spring Boot 3.x | API REST, sem renderização de páginas |
| Segurança | Spring Security + JWT | Token de acesso curto e refresh token |
| Persistência | Spring Data JPA (Hibernate) + MySQL 8 | Migrações versionadas com Flyway |
| Tarefas agendadas | Spring Scheduler | Manutenção preventiva, alertas de SLA e reincidência |
| Frontend | React + Vite + TypeScript | Interface única, responsiva, servida como estáticos |
| Estado remoto | TanStack Query | Cache, revalidação e gestão de erros de rede |
| Testes | JUnit 5 + Testcontainers | Testes de integração contra MySQL real |
| Distribuição | JAR executável + Nginx | React compilado servido pelo Nginx, API por proxy |

### 13.2. Decisões que decorrem dos requisitos

**Permissões (secção 4.2).** A matriz de permissões é implementada com `@PreAuthorize` ao nível dos serviços, nunca apenas escondendo botões na interface. O React esconde o que o utilizador não pode fazer, mas a decisão é sempre validada no servidor.

**Isolamento por cliente (RNF11).** O requisito de um cliente nunca ver dados de outro não pode depender de o programador se lembrar de filtrar. Implementa-se com um filtro Hibernate activado por sessão, aplicado automaticamente às consultas de pedidos, balcões e impressoras.

**Máquina de estados (secção 6).** Todas as transições passam por um único serviço de domínio que valida se a transição é legal, quem a pode fazer e que dados são obrigatórios. Nenhum controlador altera o estado directamente.

**Eventos (RF15).** Cada transição publica um evento de domínio que grava a linha em `pedido_evento`. A tabela é append-only: sem `update` nem `delete`.

**Notificações (RF27–RF30).** Padrão *outbox*: a transição grava a notificação numa tabela na mesma transacção que o pedido, e um processo agendado envia e marca como enviada, com repetição em caso de falha. Isto garante duas coisas — a submissão nunca fica bloqueada à espera do SMS, e nenhuma notificação se perde se o agregador estiver indisponível.

**Tolerância a rede fraca (RNF06).** O formulário de registo de intervenção guarda rascunho no dispositivo antes de submeter, e a submissão envia uma chave de idempotência para que um reenvio não crie um pedido duplicado.

**Anexos (RF12).** Ficheiros em armazenamento de objectos ou sistema de ficheiros; na base de dados fica apenas o metadado. Nunca guardar imagens em colunas da base de dados.

### 13.3. Estrutura do backend

Organização por domínio, não por camada técnica:

```
com.ricotecnica.portal
├── auth          autenticação, tokens, reposição de palavra-passe
├── utilizador    utilizadores, papéis, associação a balcões
├── parque        clientes, balcões, impressoras, atribuições
├── pedido        pedido, máquina de estados, eventos, checklist, anexos
├── manutencao    planos preventivos e geração agendada
├── notificacao   outbox, envio por email e SMS
├── relatorio     consultas de SLA, produtividade e histórico
└── comum         erros, auditoria, configuração, segurança
```

### 13.4. Mapeamento do modelo de dados

O modelo de domínio da secção 9 mantém-se sem alterações. Notas de implementação:

- **Chaves primárias:** `BIGINT` auto-incremento como chave interna, e uma coluna `uuid` pública usada nas URLs e na API. Expor o identificador sequencial permite adivinhar o volume de pedidos da empresa.
- **Enumerados:** `estado`, `tipo` e `papel` guardados como `VARCHAR` com `@Enumerated(EnumType.STRING)`, nunca como ordinal — um ordinal quebra silenciosamente quando se acrescenta um valor.
- **Datas:** todas as colunas temporais em UTC (`TIMESTAMP`), convertidas para CAT apenas na apresentação (RNF16).
- **Auditoria (RNF12):** `@CreatedBy`, `@CreatedDate`, `@LastModifiedBy` e `@LastModifiedDate` via Spring Data Auditing em todas as entidades.
- **Índices obrigatórios:** `pedido(impressora_id, aberto_em)` para o alerta de reincidência; `pedido(estado, aberto_em)` para as listas de trabalho; `pedido_evento(pedido_id, criado_em)` para o histórico.
- **Eliminação:** nenhuma entidade é eliminada fisicamente (RN05); utilizadores e impressoras têm coluna de estado activo.

---

## 14. Contrato da API — MVP

Convenções aplicáveis a todos os recursos:

- Prefixo `/api`, autenticação por `Authorization: Bearer <token>`.
- Datas em ISO-8601 UTC (`2026-08-08T14:32:00Z`).
- Listas paginadas com `?page=&size=&sort=`, resposta com `content`, `page`, `totalElements`.
- Erros no formato *Problem Details* (RFC 7807), com `type`, `title`, `status`, `detail` e, para erros de validação, `errors` por campo.
- As transições de estado são recursos de acção (`POST /pedidos/{id}/iniciar`), não um `PATCH` ao campo `estado`. Isto impede que a interface force um estado ilegal.

### 14.1. Autenticação

| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/auth/login` | Autenticação; devolve token de acesso e refresh |
| POST | `/auth/refresh` | Renova o token de acesso |
| POST | `/auth/logout` | Invalida o refresh token |
| POST | `/auth/convite/{token}` | Define a palavra-passe a partir do convite (RF02) |
| POST | `/auth/recuperar` | Pedido de reposição de palavra-passe |
| GET | `/me` | Perfil, papéis e balcões do utilizador autenticado |

### 14.2. Parque

| Método | Endpoint | Descrição |
|---|---|---|
| GET/POST | `/clientes` | Listar e criar clientes |
| GET/PUT | `/clientes/{id}` | Detalhe e actualização |
| GET/POST | `/balcoes` | Listar (filtro `?clienteId=`) e criar |
| GET/PUT | `/balcoes/{id}` | Detalhe e actualização |
| GET/POST | `/impressoras` | Listar (filtros `?balcaoId=`, `?estado=`) e criar |
| GET/PUT | `/impressoras/{id}` | Detalhe e actualização |
| POST | `/impressoras/importar` | Importação por ficheiro; devolve linhas aceites e rejeitadas |
| GET/POST | `/balcoes/{id}/tecnicos` | Consultar e definir atribuições técnico–balcão |
| GET/POST | `/utilizadores` | Listar e convidar utilizadores |
| PUT | `/utilizadores/{id}/estado` | Activar ou desactivar |

### 14.3. Pedidos

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/pedidos` | Lista filtrada por papel; filtros `?estado=`, `?balcaoId=`, `?tecnicoId=`, `?de=`, `?ate=` |
| POST | `/pedidos` | Submissão; aceita cabeçalho `Idempotency-Key` |
| GET | `/pedidos/{id}` | Detalhe completo com estado actual |
| GET | `/pedidos/{id}/eventos` | Histórico cronológico de eventos |
| POST | `/pedidos/{id}/atribuir` | Atribuir ou reatribuir técnico |
| POST | `/pedidos/{id}/agendar` | Definir ou alterar a data e hora previstas |
| POST | `/pedidos/{id}/iniciar` | Registar início da intervenção (→ em reparação) |
| POST | `/pedidos/{id}/fechar` | Resumo do trabalho, checklist e contador (→ fechado) |
| POST | `/pedidos/{id}/cancelar` | Cancelamento com justificação obrigatória |
| POST | `/pedidos/{id}/comentarios` | Comentário livre, gravado como evento |
| POST | `/pedidos/{id}/anexos` | Upload de ficheiro (multipart) |
| GET | `/impressoras/{id}/pedidos` | Histórico de pedidos da máquina (RF32) |

### 14.4. Códigos de resposta relevantes

| Código | Situação |
|---|---|
| 400 | Dados inválidos (campo obrigatório em falta, contador inferior ao anterior) |
| 401 | Token ausente ou expirado |
| 403 | Papel sem permissão para a acção, ou tentativa de aceder a outro cliente |
| 409 | Transição de estado ilegal, ou pedido já aberto para a mesma máquina (RN07) |
| 422 | Regra de negócio violada (fecho sem checklist ou sem contador) |

---

*Documento de requisitos v2.5 — base de trabalho. As decisões D1 a D9 estão fechadas. Restam as duas recolhas de informação da secção 12.3, a concluir durante a Fase 0.*
