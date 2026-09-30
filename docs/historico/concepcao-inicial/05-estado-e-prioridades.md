# Estado do projecto e prioridades

Estado real com base no que já foi fechado, e o que falta por ordem de prioridade. Nada disto é código: o desbloqueador é a assinatura da proposta.

## O que está feito

- Documento de requisitos v2.5 com 9 decisões fechadas
- Proposta comercial com 11 entregas granulares e valores
- Diagrama entidade-relacionamento

## O que falta antes de escrever uma linha de código

### 1. O Hélio assina a proposta

É o único desbloqueador real. Nada mais faz sentido fazer antes disso.

### 2. Duas recolhas da Fase 0

Dependem do Hélio e da equipa técnica dele.

- Lista de itens da checklist de intervenção — sessão de uma hora com dois técnicos
- Como é feita hoje a leitura do contador e qual passa a ser a fonte oficial para facturação

### 3. Decisões técnicas que não dependem do Hélio

| Decisão | Impacto |
| --- | --- |
| Onde o sistema vai correr (VPS, nuvem, servidor próprio) | Afecta custo recorrente e tempo de setup |
| Agregador de SMS em Moçambique | Tem prazo de contratação, trata cedo |
| Repositório e estrutura (monorepo ou dois repos — Java + React) | Define como os sprints funcionam |
| Vais desenvolver sozinho ou tens alguém? | Define o ritmo realista dos sprints de 2 semanas |

### 4. Wireframes dos 5 ecrãs do MVP

Fazem parte da Fase 0, mas podes levar um rascunho para a reunião com o Hélio. Os ecrãs chave são:

- Submissão de pedido
- Lista de pedidos
- Detalhe do pedido
- Registo de intervenção no telemóvel
- Cadastro de equipamento

### 5. Contrato formal

A proposta tem campo de assinatura mas não tem cláusulas de rescisão, propriedade intelectual nem confidencialidade. Para uma apresentação académica pode ficar assim. Para implantação real, não.

## Sequência prática

```
Agora                 → Reunião com o Hélio, apresentar a proposta
Semana 1              → Adjudicação E0 a E6
Semana 1–2            → Fase 0: wireframes, recolhas R1 e R2, setup do ambiente
Sprint 1 (sem. 3–4)   → E1: autenticação e perfis — primeiro código
```

## O que dá para fazer já, sem esperar pelo Hélio

- Wireframes dos 5 ecrãs
- Setup de um repositório base Java/Spring Boot + React
- Contrato formal com cláusulas simples
