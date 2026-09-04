# Relatório de medidas DAX — Dashboard Comercial

**Projeto analisado:** `PROJETO 31 - Comercial.pbix`  
**Data da análise:** 03/09/2026  
**Escopo:** arquivo autoral aberto no Power BI Desktop; a pasta `Solução` não foi tratada como autoria do projeto.

## 1. Entendimento do problema

O objetivo deste relatório é:

1. inventariar todas as medidas existentes no modelo semântico atual;
2. registrar as expressões DAX exatamente como estão no PBIX;
3. verificar resultado, uso nos visuais e coerência com o grão dos dados;
4. entregar um catálogo DAX recomendado para completar a camada analítica do painel;
5. separar medidas prontas de fórmulas que dependem de uma decisão de negócio ou de correção do modelo.

Foram identificadas **5 medidas**, distribuídas entre as tabelas `Medidas` e `fVendas`. O relatório possui duas páginas, mas apenas a medida `[$ Faturamento]` está vinculada a um visual. As tabelas de metas ainda não fazem parte do modelo carregado.

## 2. Resposta principal

### 2.1 Resumo executivo

| Situação | Quantidade | Conclusão |
|---|---:|---|
| Medidas existentes | 5 | Todas foram extraídas e documentadas |
| Medidas usadas nos visuais | 1 | `[$ Faturamento]`, em um cartão da Página 1 |
| Medidas corretas no total geral | 3 | Faturamento bruto, notas e quantidade |
| Medidas semanticamente ambíguas | 1 | Preço unitário médio não é ponderado por quantidade |
| Medidas incorretas | 1 | `[Faturamento]` desconta o valor da nota uma vez por item |
| Inteligência temporal funcional | 0 | `fVendas[data]` está como texto e não encontra correspondência em `dCalendario[Data]` |
| Medidas de meta | 0 | Não existe tabela de metas no modelo atual |

O valor bruto confirmado é **R$ 9.034.817,00**. O desconto correto no grão da nota é **R$ 631.995,00**, resultando em receita líquida de **R$ 8.402.822,00**.

### 2.2 Inventário das medidas existentes

| Tabela | Medida | Resultado sem filtros | Uso atual | Avaliação |
|---|---|---:|---|---|
| `Medidas` | `$ Faturamento` | R$ 9.034.817,00 | Cartão da Página 1 | Correta como receita **bruta**, mas o nome é ambíguo |
| `Medidas` | `Notas Emitidas` | 3.431 | Não usada | Correta para o modelo atual |
| `Medidas` | `Preço Unitário Médio` | R$ 151,7058 | Não usada | Fórmula válida, mas é média simples das linhas, não preço médio ponderado |
| `Medidas` | `Quant.Vendida` | 67.929 | Não usada | Correta; recomenda-se padronizar o nome |
| `fVendas` | `Faturamento` | **-R$ 4.738.529,15** | Não usada | Incorreta; desconto de nota repetido no grão do item |

### 2.3 DAX atual, sem alterações

#### `$ Faturamento`

```DAX
$ Faturamento =
SUMX (
    fVendas,
    fVendas[Quantidade] * fVendas[Valor Unitario]
)
```

Interpretação: soma o valor bruto dos itens. O resultado concilia com `SUM(fVendas[Valor Bruto])`.

#### `Notas Emitidas`

```DAX
Notas Emitidas =
DISTINCTCOUNT ( fVendas[nfe] )
```

Interpretação: conta notas fiscais distintas. O resultado atual é 3.431.

#### `Preço Unitário Médio`

```DAX
Preço Unitário Médio =
AVERAGE ( fVendas[Valor Unitario] )
```

Interpretação: calcula a média simples do preço unitário das 39.960 linhas de item. Cada linha possui o mesmo peso, independentemente da quantidade vendida.

#### `Quant.Vendida`

```DAX
Quant.Vendida =
SUM ( fVendas[Quantidade] )
```

Interpretação: soma as unidades vendidas. O resultado atual é 67.929.

#### `Faturamento`

```DAX
Faturamento =
SUMX (
    fVendas,
    ( fVendas[Quantidade] * fVendas[Valor Unitario] )
        - fVendas[Valor Desconto]
)
```

Interpretação: a intenção aparente é calcular receita líquida. A medida está incorreta porque `Valor Desconto` pertence à nota, mas foi repetido em todas as linhas de item. O mesmo problema existe na coluna calculada `fVendas[Valor Líquido]`.

#### Evidência quantitativa do problema de grão

| Cálculo sobre as 39.960 linhas | Resultado | Resultado correto no grão da nota |
|---|---:|---:|
| `SUM(fVendas[Valor Desconto])` | R$ 13.773.346,15 | R$ 631.995,00 |
| `SUM(fVendas[Valor Venda])` | R$ 181.898.945,85 | R$ 8.402.822,00 |
| Medida `[Faturamento]` atual | -R$ 4.738.529,15 | Não aplicável; fórmula inválida |

### 2.4 Mapeamento atual dos visuais

| Página | Conteúdo confirmado |
|---|---|
| Página 1 | Quatro cartões, dos quais apenas um usa `[$ Faturamento]`; uma tabela usa `dimProduto[descricao]` |
| Página 2 | Um shape, sem medida vinculada |

Portanto, o painel visual ainda não consome quatro das cinco medidas existentes.

### 2.5 Catálogo DAX recomendado — medidas seguras

Estas medidas usam apenas colunas confirmadas e produzem resultados coerentes sob filtros de produto, cliente, vendedor e data — desde que o relacionamento de calendário seja corrigido para os filtros temporais. As contagens distintas não são aditivas entre grupos, por definição.

```DAX
Receita Bruta =
SUM ( fVendas[Valor Bruto] )


Quantidade Vendida =
SUM ( fVendas[Quantidade] )


Notas Emitidas =
DISTINCTCOUNT ( fVendas[nfe] )


Clientes Ativos =
DISTINCTCOUNT ( fVendas[cliente_id] )


Produtos Vendidos =
DISTINCTCOUNT ( fVendas[produto_id] )


Itens de Venda =
DISTINCTCOUNT ( fVendas[id] )


Ticket Médio Bruto =
DIVIDE ( [Receita Bruta], [Notas Emitidas] )


Preço Médio Ponderado =
DIVIDE ( [Receita Bruta], [Quantidade Vendida] )


Unidades por Nota =
DIVIDE ( [Quantidade Vendida], [Notas Emitidas] )
```

Resultados de controle no total geral:

| Medida recomendada | Resultado validado |
|---|---:|
| Receita Bruta | R$ 9.034.817,00 |
| Quantidade Vendida | 67.929 |
| Notas Emitidas | 3.431 |
| Clientes Ativos | 696 |
| Produtos Vendidos | 90 |
| Itens de Venda | 39.960 |
| Ticket Médio Bruto | R$ 2.633,29 |
| Preço Médio Ponderado | R$ 133,00 |
| Unidades por Nota | 19,80 |

### 2.6 Receita líquida no grão da nota

As fórmulas abaixo recuperam uma única ocorrência do desconto por venda e conciliam o total geral.

```DAX
Desconto NF =
SUMX (
    VALUES ( fVendas[vendas_id] ),
    CALCULATE ( MAX ( fVendas[Valor Desconto] ) )
)


Receita Líquida NF =
[Receita Bruta] - [Desconto NF]


% Desconto NF =
DIVIDE ( [Desconto NF], [Receita Bruta] )


Ticket Médio Líquido NF =
DIVIDE ( [Receita Líquida NF], [Notas Emitidas] )
```

Resultados de controle:

| Medida | Resultado validado |
|---|---:|
| Desconto NF | R$ 631.995,00 |
| Receita Líquida NF | R$ 8.402.822,00 |
| % Desconto NF | 7,00% |
| Ticket Médio Líquido NF | R$ 2.449,09 |

> [!CAUTION]
> `Desconto NF` e `Receita Líquida NF` são corretas no total e em dimensões que não dividem uma nota entre vários membros, como cliente ou vendedor. Elas **não podem ser usadas diretamente por produto ou categoria**, porque uma mesma nota contém vários itens e o desconto ainda não possui regra oficial de rateio.

O teste por categoria comprovou o risco: somar o desconto integral das notas dentro de cada categoria produz R$ 2.236.364,70, embora o desconto real seja R$ 631.995,00.

### 2.7 Medidas condicionais — somente após aprovar o rateio

A alternativa abaixo distribui o desconto proporcionalmente ao valor bruto de cada item. Esta é uma **hipótese técnica**, não uma regra de negócio confirmada.

```DAX
Desconto Rateado Proporcional =
ROUND (
    SUMX (
        fVendas,
        VAR VendaAtual = fVendas[vendas_id]
        VAR BrutoItem = fVendas[Valor Bruto]
        VAR BrutoVenda =
            CALCULATE (
                SUM ( fVendas[Valor Bruto] ),
                FILTER (
                    ALL ( fVendas ),
                    fVendas[vendas_id] = VendaAtual
                )
            )
        VAR DescontoVenda =
            CALCULATE (
                MAX ( fVendas[Valor Desconto] ),
                FILTER (
                    ALL ( fVendas ),
                    fVendas[vendas_id] = VendaAtual
                )
            )
        RETURN
            DIVIDE ( BrutoItem, BrutoVenda, 0 ) * DescontoVenda
    ),
    2
)


Receita Líquida Rateada =
[Receita Bruta] - [Desconto Rateado Proporcional]
```

O total foi validado em R$ 631.995,00 de desconto. A soma das categorias também reconcilia, mas o rateio deve ser aprovado antes de entrar no painel. Para produção, é preferível materializar o valor rateado no Power Query e definir uma regra para resíduos de centavos.

### 2.8 Inteligência temporal — bloqueada até corrigir a data

O calendário tem 1.095 datas contínuas, de 01/01/2021 a 31/12/2023. Porém, `dCalendario[Data]` é do tipo Data e `fVendas[data]` está como Texto. Por isso, todas as 39.960 linhas da fato aparecem no membro em branco quando o resultado é agrupado por ano.

Depois de converter `fVendas[data]` para Data no Power Query, validar o relacionamento e marcar `dCalendario` como tabela de datas, usar:

```DAX
Receita Bruta AA =
CALCULATE (
    [Receita Bruta],
    DATEADD ( dCalendario[Data], -1, YEAR )
)


Variação Receita Bruta R$ =
[Receita Bruta] - [Receita Bruta AA]


Variação Receita Bruta % =
DIVIDE (
    [Variação Receita Bruta R$],
    [Receita Bruta AA]
)


Receita Bruta YTD =
TOTALYTD (
    [Receita Bruta],
    dCalendario[Data]
)


Receita Bruta YTD AA =
CALCULATE (
    [Receita Bruta YTD],
    DATEADD ( dCalendario[Data], -1, YEAR )
)


Variação Receita Bruta YTD % =
DIVIDE (
    [Receita Bruta YTD] - [Receita Bruta YTD AA],
    [Receita Bruta YTD AA]
)
```

Essas expressões foram validadas sintaticamente no modelo, mas não devem ser classificadas como funcionais enquanto o relacionamento temporal não produzir resultados por ano.

### 2.9 Medidas que não podem ser fechadas agora

| Grupo | Motivo do bloqueio | Evidência necessária |
|---|---|---|
| Meta, atingimento e gap | Nenhuma tabela de metas está carregada no modelo | Tabela final, grão, colunas e relacionamentos de `Meta_2022` e `Meta_2023` |
| Margem e lucratividade histórica | O custo disponível não possui vigência histórica | Regra oficial de custo por data ou aceite explícito de margem estimada com custo atual |
| Receita líquida por produto/categoria | Desconto está no cabeçalho da nota | Regra oficial de rateio |
| Comparação 2023 contra período completo | O histórico de 2023 termina em 22/10/2023 | Regra de período comparável, projeção ou meta proporcional |

## 3. Raciocínio técnico

O grão de `fVendas` é o item da venda: 39.960 linhas para 3.431 notas. `Valor Bruto` é aditivo nesse grão, mas `Valor Desconto` e `Valor Venda` vieram do cabeçalho da nota e aparecem repetidos nos itens.

Por isso:

- `SUM(fVendas[Valor Bruto])` é válido;
- `SUM(fVendas[Valor Desconto])` não é válido;
- `SUM(fVendas[Valor Venda])` não é válido;
- subtrair o desconto em cada linha gera um total negativo;
- receita líquida por produto exige rateio ou remodelagem.

A arquitetura recomendada utiliza medidas base curtas e medidas derivadas por branching. Isso centraliza a regra de negócio, reduz duplicação e permite testar cada componente separadamente.

## 4. Possíveis erros na abordagem

1. **Manter duas medidas chamadas faturamento com conceitos diferentes.** Uma representa bruto e a outra tenta representar líquido.
2. **Usar a medida `[Faturamento]` atual.** O resultado total é negativo e altera qualquer decisão baseada no KPI.
3. **Somar `Valor Venda` ou `Valor Desconto`.** As colunas estão repetidas no grão do item.
4. **Interpretar `AVERAGE(Valor Unitario)` como preço médio por unidade.** A medida atual atribui peso igual a cada linha, não a cada unidade vendida.
5. **Criar medidas anuais antes de corrigir a data.** O calendário não filtra a fato no estado atual.
6. **Aplicar receita líquida por produto sem rateio aprovado.** Isso atribui descontos integrais a várias categorias.
7. **Comparar 2023 parcial com anos completos.** O corte em 22/10/2023 precisa aparecer no indicador ou ser tratado por período comparável.
8. **Calcular margem histórica com custo atual.** O resultado seria estimativo, não margem histórica observada.

## 5. Versão otimizada

### Organização sugerida da tabela `Medidas`

| Pasta de exibição | Medidas |
|---|---|
| `01 Base` | Receita Bruta, Quantidade Vendida, Notas Emitidas, Clientes Ativos, Produtos Vendidos, Itens de Venda |
| `02 Eficiência` | Ticket Médio Bruto, Preço Médio Ponderado, Unidades por Nota |
| `03 Líquido - Nota` | Desconto NF, Receita Líquida NF, % Desconto NF, Ticket Médio Líquido NF |
| `04 Tempo` | Receita Bruta AA, variações e YTD, após a correção da data |
| `90 Condicional` | Desconto Rateado Proporcional e Receita Líquida Rateada, somente após aprovação |

### Padrões recomendados

- manter todas as medidas na tabela `Medidas`;
- usar nomes de negócio sem prefixos como `$` ou abreviações como `Quant.`;
- preencher descrição, pasta de exibição e formato de cada medida;
- usar moeda com duas casas para valores financeiros e percentual com uma ou duas casas;
- ocultar chaves técnicas e colunas numéricas da fato para evitar agregações implícitas;
- desabilitar Data/Hora automática depois de validar e marcar `dCalendario` como tabela de datas;
- substituir ou ocultar a coluna calculada incorreta `fVendas[Valor Líquido]` após a correção.

## 6. Próximo passo recomendado

Executar nesta ordem:

1. converter `fVendas[data]` de Texto para Data no Power Query;
2. validar `dCalendario[Data]` → `fVendas[data]` e confirmar que 2021, 2022 e 2023 deixam de cair no membro em branco;
3. criar as nove medidas seguras da seção 2.5 na tabela `Medidas`;
4. substituir o cartão atual por `[Receita Bruta]` e preencher os três cartões vazios com indicadores definidos;
5. decidir se “faturamento” oficial significa bruto ou líquido;
6. aprovar ou rejeitar o rateio proporcional do desconto por item;
7. carregar e modelar as metas antes de criar atingimento e gap;
8. testar totais gerais, ano, gerente, vendedor, categoria e produto antes de liberar o painel.

## Validação executada

- extração das cinco expressões DAX diretamente do modelo aberto;
- consulta dos valores de todas as medidas sem filtros;
- verificação das referências usadas nos visuais do PBIX salvo;
- teste do desconto no grão da nota e por categoria;
- teste do rateio proporcional e reconciliação do total;
- teste das medidas temporais, que confirmou o bloqueio de tipo/relacionamento da data;
- nenhuma medida, relacionamento ou visual foi alterado dentro do PBIX.
