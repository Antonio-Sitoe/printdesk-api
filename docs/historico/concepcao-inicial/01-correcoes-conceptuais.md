# Correcções conceptuais

Três erros do enunciado têm de ser corrigidos antes do desenho. Se forem herdados, contaminam o modelo, as permissões e os relatórios.

## 1. O diagrama de classes está conceptualmente errado

`Administrador`, `Cliente`, `Tecnico Impressora` e `Tecnico HelpDesk` não são classes. São papéis de um mesmo `Utilizador`.

`Gerar Relatorio` e `PRActualizar` são casos de uso ou acções, não entidades.

Modelar os papéis como classes gera quatro tabelas de pessoas quase idênticas e um sistema de permissões impossível de manter.

## 2. Falta a entidade central do problema

O texto diz que as impressoras estão em balcões e agências e que não há organização sobre qual técnico é responsável por um determinado balcão. Não existe `Balcão` / `Agência` no modelo. É a entidade que resolve essa queixa.

`IDCliente` numa impressora não basta: o BCI tem dezenas de balcões.

## 3. Não há histórico, só sobreposição de estado

O RNF02 exige histórico de pedidos por ID de máquina. O RNF05 exige um alerta de reincidência em 15 dias. Nenhum dos dois é possível se `PRActualizar` reescreve o pedido.

É precisa uma tabela de eventos append-only. Cada mudança fica registada; o estado actual do pedido lê-se a partir do último evento, sem apagar o que aconteceu antes.
