# Publicação do projeto no LinkedIn

## Versão coerente com o estágio atual

Estou desenvolvendo um novo projeto de portfólio em Power BI: um dashboard para análise comercial.

Antes de começar pelos gráficos, organizei o problema e a estrutura dos dados. O conjunto reúne 3.431 vendas, 39.960 itens, 701 clientes, 93 registros de produto e uma hierarquia comercial com vendedores, supervisores e gerentes. O histórico cobre o período de fevereiro de 2021 a outubro de 2023.

O diagrama do modelo já contém a fato de vendas, dimensões de cliente, produto, vendedor e calendário, além de uma tabela dedicada às medidas. A próxima etapa é validar os relacionamentos, documentar os cálculos e construir as análises de realizado versus meta.

Na validação inicial, as chaves verificadas não apresentaram registros órfãos e os valores de cabeçalho, itens, desconto e venda líquida conciliaram. Ao mesmo tempo, ficaram claros dois cuidados de modelagem: usar o ID para clientes com nomes repetidos e definir o rateio do desconto antes de analisar venda líquida por produto.

O aprendizado mais importante até aqui foi simples: um dashboard confiável não começa no visual. Ele começa na pergunta de negócio, na qualidade das fontes e em um modelo que permita explicar os números.

Vou compartilhar a evolução do projeto e, quando a camada visual estiver validada, publicar o case completo no GitHub.

Link do projeto: **[ADICIONAR URL DO GITHUB]**

Que análise você consideraria indispensável em um dashboard comercial?

## Segunda opção de gancho

> Um dashboard comercial não começa no gráfico. Começa na definição correta do que será medido.

O restante do texto pode permanecer igual à versão principal.

## Versão para usar depois da conclusão

Concluí meu projeto de dashboard comercial em Power BI.

O desafio foi transformar dados de vendas, metas, clientes, produtos e equipe comercial em uma visão orientada à decisão. O modelo foi estruturado com uma tabela fato de vendas, dimensões de apoio, calendário e medidas explícitas para manter os indicadores rastreáveis.

O dashboard permite analisar:

- **[INSERIR ANÁLISE VALIDADA 1]**;
- **[INSERIR ANÁLISE VALIDADA 2]**;
- **[INSERIR ANÁLISE VALIDADA 3]**.

Um dos principais insights encontrados foi **[INSERIR INSIGHT VALIDADO]**.

Além do desenvolvimento visual, documentei as fontes, a arquitetura, as limitações e o processo de validação. O projeto completo está disponível no GitHub:

**[ADICIONAR URL DO GITHUB]**

Qual indicador você usa para avaliar desempenho comercial além do faturamento?

## Checklist antes de publicar

- substituir todos os campos entre colchetes;
- adicionar a URL pública e testar o acesso sem estar autenticado;
- incluir uma imagem nítida do dashboard autoral;
- conferir se os números citados continuam válidos após a carga final;
- não usar screenshots ou entregas da pasta `Solução` como se fossem autorais;
- confirmar a autorização de redistribuição das bases e dos assets do curso;
- revisar ortografia e a visualização do post no celular;
- publicar somente depois de concluir e validar o PBIX quando usar a versão final.
