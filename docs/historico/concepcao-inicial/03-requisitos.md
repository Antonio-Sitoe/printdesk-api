# Requisitos em falta e classificação

## Requisitos que faltam

O enunciado descreve a submissão do pedido. O problema real é que a informação não circula a tempo: o gestor tem de ligar ao técnico a perguntar se acabou. Um portal onde é preciso ir lá ver não resolve isso.

### Notificações

- Notificar o técnico quando o pedido lhe é atribuído.
- Notificar o HelpDesk quando o pedido fecha.
- Em Moçambique, SMS ou WhatsApp é mais fiável para o técnico em campo do que email ou push.

### Uso em campo

O técnico trabalha no balcão, com telemóvel e rede irregular. Isto é um requisito de arquitectura: mobile-first e tolerância a falhas de rede. Não é um detalhe de interface.

### Contador de cópias e consumíveis

Sendo outsourcing de impressão, a facturação é por cópia. Registar o contador em cada intervenção e as peças ou toners consumidos transforma o portal de gestor de tickets em activo comercial. É a maior omissão do documento.

### Fotografias e anexos

Fotografias ou anexos da avaria e do painel da máquina.

### Medição de SLA por contrato

O impacto das 2 horas está referido, mas não há nenhum requisito que o meça. O SLA mede-se entre `submetido` e `em curso` (ver [modelo e estados](02-modelo-de-dados-e-estados.md)).

## Classificação

RNF01 a RNF05 são quase todos requisitos funcionais mal arrumados: níveis de acesso, histórico, alertas, relatórios de bónus.

Os não-funcionais verdadeiros estão ausentes:

- disponibilidade
- utilizadores concorrentes
- retenção de dados
- autenticação e cifragem
- tempo de resposta

Vale a pena reorganizar esta lista antes de sair para código.

O RF04 também deve mudar: o administrador convida o utilizador. Não atribui senhas manualmente.
