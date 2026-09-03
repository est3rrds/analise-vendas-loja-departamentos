# 📐 Documentação das Medidas DAX

Todas as medidas estão organizadas em uma tabela dedicada chamada `Medidas`, separada das tabelas de dados, seguindo boa prática de organização de modelos Power BI.

## Medidas básicas

```dax
Receita Total = SUMX(Vendas, Vendas[Quantidade] * Vendas[Preco_Unitario] * (1 - Vendas[Desconto (%)]/100))
```
Soma a receita transação a transação, aplicando quantidade, preço unitário e desconto.

```dax
Qtd Vendas = COUNTROWS(Vendas)
```
Conta o número de transações (não a quantidade de itens vendidos).

```dax
Ticket Medio = DIVIDE([Receita Total], [Qtd Vendas])
```
Receita média por transação. Usa `DIVIDE()` em vez de `/` para evitar erro de divisão por zero.

```dax
Custo Total = SUMX(Vendas, RELATED(Produtos[Custo]) * Vendas[Quantidade])
```
Traz o custo unitário de `Produtos` via `RELATED()` e multiplica pela quantidade vendida em cada transação.

```dax
Margem Total = [Receita Total] - [Custo Total]
```

```dax
Margem % = DIVIDE([Margem Total], [Receita Total])
```

## Inteligência de tempo

Dependem da tabela `Calendario`, relacionada a `Vendas[Data_Venda]`.

```dax
Receita Mês Anterior = CALCULATE([Receita Total], DATEADD(Calendario[Data], -1, MONTH))
```
Desloca o contexto de data em um mês para trás.

```dax
Var % MoM = DIVIDE([Receita Total] - [Receita Mês Anterior], [Receita Mês Anterior])
```
Variação percentual mês a mês ("Month over Month").

```dax
Receita YTD = TOTALYTD([Receita Total], Calendario[Data])
```
Receita acumulada do início do ano até a data filtrada.

```dax
Receita Acumulada Trimestre = CALCULATE([Receita Total], DATESQTD(Calendario[Data]))
```
Receita acumulada dentro do trimestre atual.

## Ranking e destaque

```dax
Ranking Produtos = RANKX(ALL(Produtos[Nome_Produto]), [Receita Total], , DESC)
```
Posição de cada produto por receita. O `ALL()` garante que o ranking sempre compare todos os produtos, independente de filtros de página aplicados.

```dax
Top 5 Produtos Flag = IF([Ranking Produtos] <= 5, "Top 5", "Outros")
```

```dax
Melhor Vendedor = CALCULATE(
    MAX(Vendedores[Nome_Vendedor]),
    TOPN(1, ALL(Vendedores[Nome_Vendedor]), [Receita Total])
)
```

```dax
Mes Destaque = 
"Melhor mês: " & 
CALCULATE(
    MAX(Calendario[NomeMes]),
    TOPN(1, ALL(Calendario[NomeMes]), [Receita Total])
)
```
Retorna um texto dinâmico com o mês de maior receita — usado em cartão de destaque na página Visão Geral.

## Comparativas e % de participação

```dax
% Receita E-commerce = DIVIDE(
    CALCULATE([Receita Total], Vendas[Canal_Venda] = "E-commerce"),
    [Receita Total]
)
```

```dax
% do Total por Categoria = DIVIDE(
    [Receita Total],
    CALCULATE([Receita Total], ALL(Produtos[Categoria]))
)
```
Padrão de "% do total": o denominador ignora o filtro de categoria (via `ALL`) para trazer o total geral, permitindo calcular a fatia de cada categoria.

## Colunas calculadas (Power Query / M)

Além das medidas, três colunas foram criadas na consulta de dados (não são DAX, mas complementam o modelo):

- **`Vendas[Valor_Total]`**: `Quantidade × Preço Unitário × (1 - Desconto)`
- **`Clientes[Idade]`**: calculada a partir de `Data_Nascimento`
- **`Clientes[Faixa_Etaria]`**: faixas de 10 em 10 anos (18-25, 26-35, 36-45, 46-55, 56+), com uma coluna auxiliar `Ordem_Faixa` (baseada em `Idade`) usada para ordenação correta via "Classificar por Coluna"
- **`Clientes[Estado_Nome]`**: conversão de sigla (SP, RJ...) para nome completo do estado, necessária para geocodificação correta em visuais de mapa
