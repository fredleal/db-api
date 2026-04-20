# BR: [Nome curto da regra]

> Template do database "Business Rules". Properties: Domínio, Confiança (alta/média/baixa), Fonte, Última validação.

> **Domínio**: [Pedidos | Estoque | Faturamento | Logística | ...]
> **Confiança**: alta | média | baixa (quanto mais fontes confirmarem, mais alta)

## Regra
[Descrição clara e concisa da regra de negócio. 2-5 linhas.]

## Contexto / Por quê
[Por que essa regra existe? Qual problema ela resolve? Histórico se souber.]

## Exceções conhecidas
- [Caso em que a regra NÃO se aplica]

## Exemplo prático
[Cenário real que ilustra a regra em ação]

## Fonte
- Quem contou: [nome]
- Quando: [data]
- Confirmado por: [nome, se houver segunda fonte]

## Impacto no código
[Onde no sistema essa regra se manifesta? Módulo, endpoint, tabela, campo.]
