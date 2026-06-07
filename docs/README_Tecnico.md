cat > /home/claude/README_final.md << 'EOF'
# 📊 Análise Logística & Supply Chain — Documentação Técnica

> **Arquivo Power BI:** BI para Portifólio
> **Documentação gerada via:** Power BI MCP — conexão direta ao modelo semântico
> **Data de extração:** Junho 2026
> **Autor:** Ruan Estevão Barbosa Lopes

---

## Índice

1. [Visão Geral do Projeto](#1-visão-geral-do-projeto)
2. [Fontes de Dados](#2-fontes-de-dados)
3. [Modelagem de Dados](#3-modelagem-de-dados)
4. [Colunas Calculadas](#4-colunas-calculadas)
5. [Medidas DAX](#5-medidas-dax)
6. [KPIs e Métricas](#6-kpis-e-métricas)
7. [Páginas do Dashboard](#7-páginas-do-dashboard)
8. [Filtros e Segmentações](#8-filtros-e-segmentações)
9. [Regras de Negócio](#9-regras-de-negócio)
10. [Insights Disponibilizados](#10-insights-disponibilizados)
11. [Estrutura Técnica](#11-estrutura-técnica)

---

## 1. Visão Geral do Projeto

**Nome:** Análise Logística & Supply Chain

**Objetivo de Negócio:** Prover visibilidade completa sobre a performance comercial e logística de uma distribuidora, permitindo que gestores tomem decisões baseadas em dados sobre faturamento, margens, custos de frete, eficiência de transportadoras e estratificação da base de clientes.

**Problema Resolvido:** A ausência de um painel centralizado impedia que a organização respondesse a perguntas críticas: Qual é a rentabilidade real após o custo de frete? Quais transportadoras geram mais custo com menor desempenho? Como está distribuída a base de clientes em valor (Curva ABC)? Como evolui o faturamento e lucro vs. ano anterior?

**Público-Alvo:** Diretores comerciais e de operações, gerentes de logística e supply chain, analistas de dados e planejamento estratégico.

**Período de Abrangência:** Janeiro 2021 a Dezembro 2024 (4 anos completos, ~50.000 transações).

---

## 2. Fontes de Dados

**Origem:** Base simulada a partir de ERP Protheus, gerada via Python/Pandas, exportada em .xlsx e importada diretamente no Power BI Desktop.

### 2.1 fVendas_Logistica — Tabela Fato (~50.000 linhas)

| Campo | Tipo | Descrição |
|---|---|---|
| ID_Pedido | Texto | Identificador único do pedido |
| Data_Venda | Data | Data de realização do pedido |
| Data_Entrega | Data | Data efetiva/prevista de entrega |
| ID_Cliente | Texto | FK para dClientes |
| ID_Produto | Texto | FK para dProdutos |
| ID_Transportadora | Texto | FK para dTransportadoras |
| Quantidade_Vendida | Inteiro | Unidades vendidas |
| Valor_Venda | Decimal | Receita bruta em R$ |
| Custo_Produto | Decimal | Custo de aquisição em R$ |
| Valor_Frete | Decimal | Custo de frete CIF em R$ |
| Status_Entrega | Texto | Entregue / Atrasado / Em Trânsito |

### 2.2 dClientes — Dimensão (300 registros)

| Campo | Tipo | Descrição |
|---|---|---|
| ID_Cliente | Texto | PK do cliente |
| Nome_Empresa | Texto | Razão social |
| Estado | Texto | UF |
| Cidade | Texto | Município |
| Regiao | Texto | Norte/Nordeste/Centro-Oeste/Sudeste/Sul |
| Segmento | Texto | Varejo/Atacado/Indústria/Serviços/E-commerce/Distribuidor |

### 2.3 dProdutos — Dimensão (45 registros)

| Campo | Tipo | Descrição |
|---|---|---|
| ID_Produto | Texto | PK do produto |
| Nome_Produto | Texto | Descrição |
| Categoria | Texto | Eletrônicos/Móveis/Alimentos/Vestuário/Insumos Industriais |
| Subcategoria | Texto | Subcategoria dentro da categoria |
| Peso_Kg | Decimal | Peso unitário em kg |

### 2.4 dTransportadoras — Dimensão (30 registros)

| Campo | Tipo | Descrição |
|---|---|---|
| ID_Transportadora | Texto | PK da transportadora |
| Nome_Transportadora | Texto | Nome comercial |
| Tipo_Veiculo | Texto | Van/Caminhão/Avião/Moto |
| Avaliacao_Transportadora | Decimal | Avaliação de 1 a 5 |

### 2.5 dCalendario — Dimensão de Data (1.461 dias: 01/01/2021–31/12/2024)

| Campo | Tipo | Descrição |
|---|---|---|
| Data | Data | PK — data completa |
| Ano | Inteiro | 2021–2024 |
| Trimestre | Inteiro | 1–4 |
| Nome_Trimestre | Texto | T1/T2/T3/T4 |
| Mes | Inteiro | 1–12 |
| Nome_Mes | Texto | Nome em português |
| Semana_Ano | Inteiro | Semana ISO |
| Dia | Inteiro | Dia do mês |
| Dia_Semana_Num | Inteiro | 1=Segunda a 7=Domingo |
| Nome_Dia_Semana | Texto | Nome em português |
| Eh_Fim_Semana | Booleano | Sábado ou domingo |
| Eh_Feriado_BR | Booleano | Feriado nacional |
| Mes/Ano | Data | Primeiro dia do mês |
| Semestre | Texto | S1 ou S2 |
| Eh_Dia_Util | Booleano | Não é fim de semana nem feriado |

---

## 3. Modelagem de Dados

**Tipo:** Star Schema (Esquema Estrela)

### Diagrama Lógico

```
                      dCalendario
                      [Data] (PK)
                           |
             Data_Venda    | 1:N
                           |
dClientes ─────────────────┼─────────────── dProdutos
[ID_Cliente](PK)    fVendas_Logistica    [ID_Produto](PK)
      |              (Tabela Fato)              |
      | N:1          [ID_Pedido]           N:1  |
      |              [Data_Venda]               |
      └──────────────[ID_Cliente]    ───────────┘
                     [ID_Produto]
                     [ID_Transportadora]
                           |
                      N:1  |
                           |
                  dTransportadoras
                  [ID_Transportadora](PK)
```

### Relacionamentos

| Tabela Origem | Campo | Tabela Destino | Campo | Cardinalidade | Filtro |
|---|---|---|---|---|---|
| fVendas_Logistica | Data_Venda | dCalendario | Data | N:1 | Unidirecional |
| fVendas_Logistica | Data_Entrega | LocalDateTable | Date | N:1 | Unidirecional |
| fVendas_Logistica | ID_Cliente | dClientes | ID_Cliente | N:1 | Unidirecional |
| fVendas_Logistica | ID_Produto | dProdutos | ID_Produto | N:1 | Unidirecional |
| fVendas_Logistica | ID_Transportadora | dTransportadoras | ID_Transportadora | N:1 | Unidirecional |
| dCalendario | Data | LocalDateTable | Date | N:1 | Unidirecional |
| dCalendario | Trim/ano | LocalDateTable | Date | N:1 | Unidirecional |

---

## 4. Colunas Calculadas

### fVendas_Logistica[Tempo de Entrega]
```dax
DATEDIFF(fVendas_Logistica[Data_Venda], fVendas_Logistica[Data_Entrega], HOUR)
```
Calcula a diferença em horas entre a data da venda e a data de entrega. Base para os KPIs de tempo médio.

### dClientes[Status de Cliente]
```dax
SWITCH(TRUE(),
    [% Acumulado Clientes] <= 0.80, "A - Alto Valor",
    [% Acumulado Clientes] <= 0.95, "B - Médio Valor",
    "C - Baixo Valor"
)
```
Classifica cada cliente na Curva ABC com base no faturamento acumulado. Campo central da página Clientes.

### dClientes[Cod e nome]
```dax
dClientes[ID_Cliente] & "- " & dClientes[Nome_Empresa]
```
Concatena ID e nome para garantir unicidade nas tabelas do dashboard, resolvendo duplicidades de nome fantasia.

### dCalendario[É Vigente?]
```dax
IF(dCalendario[Data] < MAX(fVendas_Logistica[Data_Venda]), "Sim", "Não")
```
Identifica datas com dados de venda associados, evitando que períodos futuros distorçam médias.

### dCalendario[Trim/ano]
```dax
STARTOFQUARTER(dCalendario[Data])
```
Retorna o primeiro dia de cada trimestre para agrupamentos trimestrais.

---

## 5. Medidas DAX

O projeto contém **59 medidas DAX** na tabela `Medidas`.

### 5.1 Faturamento e Volume

**Faturamento Total**
```dax
SUM(fVendas_Logistica[Valor_Venda])
```
Receita bruta total. Medida base para cálculos derivados.

**Total Vendas**
```dax
COUNTROWS(fVendas_Logistica)
```
Número total de pedidos.

**Média faturamento**
```dax
AVERAGE(fVendas_Logistica[Valor_Venda])
```
Ticket médio por pedido.

**Total clientes**
```dax
DISTINCTCOUNT(fVendas_Logistica[ID_Cliente])
```
Clientes únicos com pedidos no contexto ativo.

---

### 5.2 Custo e Lucro

**Custo frete**
```dax
SUM(fVendas_Logistica[Valor_Frete])
```
Custo total de frete CIF. Base para % Frete CIF.

**Total Pago Frete**
```dax
SUM(fVendas_Logistica[Valor_Frete])
```
Variante de custo de frete para contextos específicos.

**Custo Total**
```dax
SUM(fVendas_Logistica[Custo_Produto]) + [Custo frete]
```
Custo total da operação: produto + frete.

**Lucro Total**
```dax
[Faturamento Total] - [Custo Total]
```
Lucro bruto do período.

**Lucro Total - Frete**
```dax
[Faturamento Total] - ([Custo Total] + [Custo frete])
```
Lucro líquido após desconto explícito do frete — rentabilidade real.

---

### 5.3 Margem e Participação

**% Margem de Lucro**
```dax
[Lucro Total] / [Faturamento Total]
```
Margem bruta percentual.

**% Margem de Lucro Com frete**
```dax
[Lucro Total - Frete] / [Faturamento Total]
```
Margem líquida real após frete. Eixo Y do scatter plot na página ABC.

**% Frete CIF**
```dax
[Custo frete] / [Faturamento Total]
```
Percentual do custo de frete sobre o faturamento. Critério de ranking de transportadoras.

**% Acumulado Clientes**
```dax
VAR FatClienteAtual = CALCULATE(SUM(fVendas_Logistica[Valor_Venda]))
VAR FatTotal = CALCULATE(SUM(fVendas_Logistica[Valor_Venda]), ALL(dClientes))
VAR FatAcumulado = CALCULATE(
    SUM(fVendas_Logistica[Valor_Venda]),
    FILTER(ALL(dClientes),
        CALCULATE(SUM(fVendas_Logistica[Valor_Venda])) >= FatClienteAtual))
RETURN DIVIDE(FatAcumulado, FatTotal, 0)
```
Percentual acumulado de faturamento por cliente (maior para menor). Base do Pareto ABC. ALL(dClientes) garante denominador sempre global.

**Clientes ABC**
```dax
SWITCH(TRUE(),
    [% Acumulado Clientes] <= 0.80, "A - Alto Valor",
    [% Acumulado Clientes] <= 0.95, "B - Médio Valor",
    "C - Baixo Valor")
```
Classificação textual do cliente no contexto atual.

**Clientes ABC Por Cor**
```dax
IF(HASONEVALUE(dClientes[ID_Cliente]),
    SWITCH(TRUE(),
        [% Acumulado Clientes] <= 0.80, "🟢",
        [% Acumulado Clientes] <= 0.95, "🟡",
        "⚪"),
    BLANK())
```
Emoji de cor por classe. Funciona apenas com cliente único no contexto.

---

### 5.4 Contagem e Lucro por Classe ABC

**Clientes Classe A**
```dax
COUNTROWS(FILTER(VALUES(dClientes),
    dClientes[Status de Cliente] = "A - Alto Valor"))
```

**Clientes Classe B**
```dax
COUNTROWS(FILTER(VALUES(dClientes),
    dClientes[Status de Cliente] = "B - Médio Valor"))
```

**Clientes Classe C**
```dax
COUNTROWS(FILTER(VALUES(dClientes),
    dClientes[Status de Cliente] = "C - Baixo Valor"))
```

**Fat Clientes Classe A**
```dax
CALCULATE([Faturamento Total],
    FILTER(VALUES(dClientes[Status de Cliente]),
        dClientes[Status de Cliente] = "A - Alto Valor"))
```

**Lucro Clintes A**
```dax
CALCULATE([Lucro Total - Frete],
    dClientes[Status de Cliente] = "A - Alto Valor")
```

**Lucro Clientes B**
```dax
CALCULATE([Lucro Total - Frete],
    dClientes[Status de Cliente] = "B - Médio Valor")
```

**lucro Clintes C**
```dax
CALCULATE([Lucro Total - Frete],
    dClientes[Status de Cliente] = "C - Baixo Valor")
```

---

### 5.5 Logística e Entrega

**% Entregas no Prazo**
```dax
VAR Quantiida_Entregues = COUNTX(
    FILTER(fVendas_Logistica, fVendas_Logistica[Status_Entrega] = "Entregue"), "")
RETURN Quantiida_Entregues / [Total Vendas]
```
Taxa OTIF simplificada: pedidos "Entregue" / total de pedidos.

**% Pedidos em Trânsito**
```dax
VAR Pedidos_Transito = COUNTROWS(
    FILTER(fVendas_Logistica, fVendas_Logistica[Status_Entrega] = "Em Trânsito"))
RETURN DIVIDE(Pedidos_Transito, COUNTROWS(fVendas_Logistica), 0)
```

**% Pedidos Atrasados**
```dax
VAR Pedidos_Transito = COUNTROWS(
    FILTER(fVendas_Logistica, fVendas_Logistica[Status_Entrega] = "Atrasado"))
RETURN DIVIDE(Pedidos_Transito, COUNTROWS(fVendas_Logistica), 0)
```

**Tempo de entrega (Dias)**
```dax
AVERAGEX(fVendas_Logistica,
    fVendas_Logistica[Data_Entrega] - fVendas_Logistica[Data_Venda])
```

**Média Tempo de Entrega(Horas)**
```dax
AVERAGE(fVendas_Logistica[Tempo de Entrega])
```

**Média Tempo de Entrega Hrs Formatado**
```dax
FORMAT([Média Tempo de Entrega(Horas)], "000.0 Horas")
```

**Média Custo de frete por pedido**
```dax
DIVIDE(SUM(fVendas_Logistica[Valor_Frete]), [Total Vendas], 0)
```

---

### 5.6 Identificação de Transportadoras

**Melhor Transportadora**
```dax
CALCULATE(MAX(dTransportadoras[Nome_Transportadora]),
    TOPN(1, ALL(dTransportadoras[Nome_Transportadora]), [% Frete CIF], ASC))
```
Transportadora com menor % Frete CIF — dinâmica, atualiza com filtros.

**% Frete Melhor Transp**
```dax
VAR Melhor_Transp = [Melhor Transportadora]
RETURN CALCULATE([% Frete CIF],
    dTransportadoras[Nome_Transportadora] = Melhor_Transp)
```

**Média frete melhor Transp**
```dax
VAR Melhor_Transp = [Melhor Transportadora]
RETURN CALCULATE(Medidas[Média Custo de frete por pedido],
    dTransportadoras[Nome_Transportadora] = Melhor_Transp)
```

**Pior Transportadora**
```dax
CALCULATE(MAX(dTransportadoras[Nome_Transportadora]),
    TOPN(1, ALL(dTransportadoras[Nome_Transportadora]), [% Frete CIF]))
```
Transportadora com maior % Frete CIF.

**% Frete CIF pior Transp**
```dax
VAR Pior_Transp = [Pior Transportadora]
RETURN CALCULATE([% Frete CIF],
    dTransportadoras[Nome_Transportadora] = Pior_Transp)
```

**Média Frete Pior Transp**
```dax
VAR Pior_Transp = [Pior Transportadora]
RETURN CALCULATE(Medidas[Média Custo de frete por pedido],
    dTransportadoras[Nome_Transportadora] = Pior_Transp)
```

---

### 5.7 Medidas de Tooltip

**Transportadora selecionada**
```dax
"🚚 " & SELECTEDVALUE(dTransportadoras[Nome_Transportadora])
```

**Tipo de Veiculo Selecionado**
```dax
SELECTEDVALUE(dTransportadoras[Tipo_Veiculo])
```

**Região mais cara**
```dax
CALCULATE(MAX(dClientes[Regiao]),
    TOPN(1, ALL(dClientes[Regiao]), Medidas[Média Custo de frete por pedido]))
```

**Regiao mais barata**
```dax
CALCULATE(MAX(dClientes[Regiao]),
    TOPN(1, ALL(dClientes[Regiao]), [Média Custo de frete por pedido], ASC))
```

**Média de frete por pedido geral**
```dax
CALCULATE([Média Custo de frete por pedido], ALL(dTransportadoras))
```
Benchmark geral ignorando filtro de transportadora.

**% Acima ou abaixo da média**
```dax
VAR FreteGeral = [Média de frete por pedido geral]
VAR FreteSelecionado = [Média Custo de frete por pedido]
RETURN DIVIDE(FreteSelecionado - FreteGeral, FreteGeral)
```

**Texto variaçao media de frete**
```dax
VAR Variacao = [% Acima ou abaixo da média]
VAR Seta = IF(Variacao > 0, "Acima ▲", "Abaixo ▼")
RETURN FORMAT(Variacao, "0.00%") & " " & Seta
```

---

### 5.8 Inteligência Temporal (Year-over-Year)

Todas as medidas YoY utilizam DATEADD e são protegidas por HASONEVALUE para evitar retornos inválidos com múltiplos anos selecionados.

**Faturamento YoY**
```dax
IF(HASONEVALUE(dCalendario[Ano]),
    CALCULATE([Faturamento Total], DATEADD(dCalendario[Data], -1, YEAR)), "")
```

**% Faturamento YoY**
```dax
VAR Faturamento_Atual = [Faturamento Total]
VAR Faturamento_YoY = CALCULATE([Faturamento Total],
    DATEADD(dCalendario[Data], -1, YEAR))
RETURN IF(HASONEVALUE(dCalendario[Ano]),
    DIVIDE(Faturamento_Atual - Faturamento_YoY, Faturamento_YoY, "N/A"), "N/A")
```

**Indicador Fat Atual VS anterior**
```dax
VAR seta = IF([% Faturamento YoY] > 0, "▲ ", "▼ ")
RETURN seta & FORMAT([% Faturamento YoY], "0.00%")
```

**% Lucro YoY**
```dax
VAR Lucro_Atual = [Lucro Total]
VAR Lucro_YoY = CALCULATE([Lucro Total], DATEADD(dCalendario[Data], -1, YEAR))
RETURN IF(HASONEVALUE(dCalendario[Ano]),
    DIVIDE(Lucro_Atual - Lucro_YoY, Lucro_YoY, "N/A"), "N/A")
```

**Indicador Lucro Atual vs Anterior**
```dax
VAR seta = IF([% Lucro YoY] > 0, "▲ ", "▼ ")
RETURN seta & FORMAT([% Lucro YoY], "0.00%")
```

**% Total de Vendas YoY**
```dax
VAR Vendas_AnoAnterior = CALCULATE([Total Vendas],
    DATEADD(dCalendario[Data], -1, YEAR))
RETURN IF(HASONEVALUE(dCalendario[Ano]),
    DIVIDE([Total Vendas] - Vendas_AnoAnterior, Vendas_AnoAnterior, "N/A"), "N/A")
```

**Indicador Vendas Atual VS Anterior**
```dax
VAR variacao = [% Total de Vendas YoY]
RETURN IF(variacao > 0, "▲ ", "▼ ") & FORMAT(variacao, "0.00%")
```

**% Total Frete YoY**
```dax
VAR Custo_Atual = [Custo frete]
VAR Custo_Anterior = CALCULATE([Custo frete], DATEADD(dCalendario[Data], -1, YEAR))
RETURN IF(HASONEVALUE(dCalendario[Ano]),
    DIVIDE(Custo_Atual - Custo_Anterior, Custo_Anterior, "N/A"), "N/A")
```

**Indicador % Frete Total YoY**
```dax
VAR Custo = [% Total Frete YoY]
RETURN IF(Custo > 0, "▲ ", "▼ ") & FORMAT(Custo, "0.00%")
```

**% Custo de frete YoY**
```dax
VAR Custo_YoY = CALCULATE([Custo frete], DATEADD(dCalendario[Data], -1, YEAR))
RETURN IF(HASONEVALUE(dCalendario[Ano]),
    DIVIDE([Custo frete] - Custo_YoY, Custo_YoY, "N/A"), "N/A")
```

**Indicador % Custo de frete YoY**
```dax
VAR Custo_YoY = CALCULATE([Custo frete], DATEADD(dCalendario[Data], -1, YEAR))
VAR Percentual_YoY = IF(HASONEVALUE(dCalendario[Ano]),
    DIVIDE([Custo frete] - Custo_YoY, Custo_YoY, "N/A"), "N/A")
VAR SETA = IF(Percentual_YoY > 0, "▲ ", "▼ ")
RETURN SETA & FORMAT(Percentual_YoY, "0.00%")
```

**% Tempo Médio YoY**
```dax
VAR TempoAtual = [Média Tempo de Entrega(Horas)]
VAR TempoAnterior = CALCULATE([Média Tempo de Entrega(Horas)],
    DATEADD(dCalendario[Data], -1, YEAR))
RETURN IF(HASONEVALUE(dCalendario[Ano]),
    DIVIDE(TempoAtual - TempoAnterior, TempoAnterior, "N/A"), "N/A")
```

**Indicador % Tempo médio de Entrega**
```dax
VAR Tempo = [% Tempo Médio YoY]
RETURN IF(Tempo > 0, "▲ ", "▼ ") & FORMAT(Tempo, "0.00%")
```

**% Média Vlr frete YoY**
```dax
VAR Media_atual = Medidas[Média Custo de frete por pedido]
VAR Media_Anterior = CALCULATE(Medidas[Média Custo de frete por pedido],
    DATEADD(dCalendario[Data], -1, YEAR))
RETURN IF(HASONEVALUE(dCalendario[Ano]),
    DIVIDE(Media_atual - Media_Anterior, Media_Anterior, "N/A"), "N/A")
```

**Indicador % Média Vlr frete YoY**
```dax
VAR MediaValor = Medidas[% Média Vlr frete YoY]
VAR Seta = IF(Medidas[% Média Vlr frete YoY] > 0, "▲ ", "▼ ")
RETURN Seta & FORMAT(MediaValor, "0.00%")
```

**% Pedidos Entregues YoY**
```dax
IF(HASONEVALUE(dCalendario[Ano]),
    CALCULATE([% Entregas no Prazo], DATEADD(dCalendario[Data], -1, YEAR)), "N/A")
```

---

### 5.9 Insight e Contexto

**Isight automático Clientes**
```dax
"💡 Os " & [Clientes Classe A] &
" clientes Classe A representam apenas " &
FORMAT(
    DIVIDE([Clientes Classe A],
        DISTINCTCOUNT(fVendas_Logistica[ID_Cliente])), "0%") &
" da base de clientes e concentram 80% da receita — risco de concentração."
```
Gera narrativa dinâmica sobre a concentração da Classe A. Atualiza conforme filtros de período.

---

## 6. KPIs e Métricas

### Página Comercial

| KPI | Medida Principal | Medida de Variação |
|---|---|---|
| Faturamento Total | Faturamento Total | Indicador Fat Atual VS anterior |
| Lucro Total | Lucro Total | Indicador Lucro Atual vs Anterior |
| Total de Vendas | Total Vendas | Indicador Vendas Atual VS Anterior |
| % Margem de Lucro | % Margem de Lucro | — |

### Página Logística

| KPI | Medida Principal | Medida de Variação |
|---|---|---|
| Total Custo de Frete | Custo frete | Indicador % Custo de frete YoY |
| Tempo Médio Entrega | Média Tempo de Entrega Hrs Formatado | Indicador % Tempo médio de Entrega |
| Frete Médio/Pedido | Média Custo de frete por pedido | Indicador % Média Vlr frete YoY |
| % Pedidos Entregues | % Entregas no Prazo | — |

### Página Clientes ABC

| KPI | Medida | Descrição |
|---|---|---|
| Total Clientes | Total clientes | Clientes únicos com pedidos |
| Clientes Classe A | Clientes Classe A | Clientes que formam 80% da receita |
| Lucro Clientes A | Lucro Clintes A | Lucro pós-frete dos clientes Classe A |
| Clientes Classe C | Clientes Classe C | Clientes no último 5% da receita |

---

## 7. Páginas do Dashboard

### Página 0 — Capa

Tela de entrada com título, subtítulo, 3 botões de navegação (Comercial / Logística / Clientes), faixa de 4 KPIs resumo (Faturamento Total · Clientes Classe A · % Entregas no Prazo · Tempo de entrega em Dias) e assinatura do autor.

---

### Página 1 — Visão Executiva — Performance Comercial

**Objetivo:** Visão estratégica do desempenho comercial.

| Visual | Tipo | Campos | Objetivo |
|---|---|---|---|
| KPI Faturamento Total | Cartão | Faturamento Total + badge YoY | Receita com variação |
| KPI Lucro Total | Cartão | Lucro Total + badge YoY | Lucro com variação |
| KPI Total de Vendas | Cartão | Total Vendas + badge YoY | Volume de pedidos |
| KPI % Margem de Lucro | Cartão | % Margem de Lucro | Margem bruta |
| Faturamento por Período | Área + linha | Eixo X: Data, Y: Faturamento Total | Evolução temporal e sazonalidade |
| Faturamento Atual vs Ano Anterior | Barras agrupadas | Categoria, Fat. Total + Fat. YoY | Comparativo por categoria YoY |
| Faturamento Total por Região | Mapa de bolhas | Estado, Faturamento Total | Distribuição geográfica |
| Volume Vendas por Segmento | Treemap | Segmento, Total Vendas | Participação por segmento |

**Filtros:** Slicer Ano, Slicer Data (range), Subcategoria, Região.

---

### Página 2 — Visão Executiva — Performance Logística

**Objetivo:** Análise operacional de frete, transportadoras e prazo de entrega.

| Visual | Tipo | Campos | Objetivo |
|---|---|---|---|
| KPI Total Custo de Frete | Cartão | Custo frete + badge YoY | Custo logístico com variação |
| KPI Tempo Médio Entrega | Cartão | Horas formatado + badge YoY | Eficiência de prazo |
| KPI Frete Médio/Pedido | Cartão | Média frete/pedido + badge YoY | Custo unitário de frete |
| KPI % Pedidos Entregues | Cartão | % Entregas no Prazo + barra status | Taxa de entrega |
| % Frete CIF por Região | Barras verticais | Regiao, % Frete CIF | Custo relativo por região |
| Top 8 Média Frete por Transportadora | Barras horizontais c/ escala cor | Nome_Transportadora, Média Custo | Ranking de custo com semáforo |
| Tabela Estado × Transportadora | Tabela com drill | Estado, % Frete CIF, Custo frete, Tempo entrega | Detalhamento por estado |
| Card Melhor Transportadora | Cartão texto dinâmico | Melhor Transportadora, % Frete, Média | Destaque da mais eficiente |
| Card Pior Transportadora | Cartão texto dinâmico | Pior Transportadora, % Frete, Média | Destaque da menos eficiente |

**Tooltip personalizado (Top 8):** Exibe nome, média por pedido, % acima/abaixo da média, tipo de veículo, região mais cara e mais barata ao passar o mouse sobre cada barra.

**Filtros:** Slicer Ano, Slicer Data (range), Subcategoria, Região.

---

### Página 3 — Visão Geral de Clientes — Classificação ABC

**Objetivo:** Estratificação da base de clientes pela Curva ABC.

| Visual | Tipo | Campos | Objetivo |
|---|---|---|---|
| KPI Total Clientes | Cartão | Total clientes | Base total |
| KPI Clientes Classe A | Cartão | Clientes Classe A | Qtd. alto valor |
| KPI Lucro Clientes A | Cartão | Lucro Clintes A | Rentabilidade Classe A |
| KPI Clientes Classe C | Cartão | Clientes Classe C | Qtd. baixo volume |
| Card Classe A | Caixa texto | Texto + métricas Classe A | Descrição e números |
| Card Classe B | Caixa texto | Texto + métricas Classe B | Descrição e números |
| Card Classe C | Caixa texto | Texto + métricas Classe C | Descrição e números |
| % Faturamento por Classes | Rosca | Status de Cliente, Faturamento Total | Distribuição de receita |
| Tabela de Clientes | Tabela | Cod e nome, Segmento, Faturamento, % Acumulado, Classe, Cor | Lista completa classificada |
| Insight Automático (ⓘ) | Tooltip de página | Isight automático Clientes | Narrativa dinâmica de concentração |

**Filtros:** Slicer Ano, Slicer Data (range), Subcategoria, Região.

---

## 8. Filtros e Segmentações

| Filtro | Campo | Tipo | Impacto |
|---|---|---|---|
| Ano | dCalendario[Ano] | Botões 2021–2024 | Filtra todos os visuais da página |
| Período | dCalendario[Data] | Slider de intervalo | Filtra por datas personalizadas |
| Pesquisar Cliente | dClientes[ID_Cliente] | Caixa de busca | Isola cliente específico |
| Subcategoria | dProdutos[Subcategoria] | Dropdown | Restringe por subcategoria de produto |

O botão "Aplicar Filtros" abre o painel lateral de segmentações em todas as páginas analíticas.

---

## 9. Regras de Negócio

**Classificação ABC:** Implementada em duas camadas. A medida `% Acumulado Clientes` usa `ALL(dClientes)` para garantir que o denominador seja sempre o total global, independente de filtros externos. A coluna `Status de Cliente` aplica o SWITCH sobre essa medida com limiares em 80% e 95%.

**Melhor/Pior Transportadora:** O critério é o `% Frete CIF`. `TOPN(1, ALL(...))` garante que a identificação ignore filtros externos, sempre retornando a transportadora global melhor/pior no período selecionado.

**Tempo de Entrega:** Calculado em horas via `DATEDIFF(HOUR)` na coluna calculada. As medidas derivadas apresentam o valor em horas (média) ou dias (subtração direta de datas) conforme o KPI.

**OTIF Simplificado:** Considera entregue apenas pedidos com `Status_Entrega = "Entregue"`. Pedidos "Atrasado" e "Em Trânsito" não são contados.

**Comparativos YoY:** Todos usam `DATEADD(dCalendario[Data], -1, YEAR)` protegido por `HASONEVALUE(dCalendario[Ano])`. Com múltiplos anos selecionados, retornam "N/A".

**Unicidade de Clientes:** A coluna `Cod e nome` (ID + Nome) resolve duplicidades de razão social na base, garantindo identificação única em tabelas.

**Distribuição Pareto da base:** ~16% dos clientes (Classe A) concentram ~80% do faturamento; ~13% (Classe B) concentram ~15%; ~71% (Classe C) concentram ~5%.

---

## 10. Insights Disponibilizados

**Comercial:** Evolução mensal e anual de faturamento, lucro e volume de pedidos. Comparativo YoY por KPI. Categorias mais rentáveis. Distribuição geográfica da receita. Participação por segmento. Sazonalidade ao longo do ano.

**Logístico:** Custo total e unitário de frete com evolução temporal. % do frete sobre o faturamento por região e transportadora. Ranking de transportadoras por custo médio com escala visual de cor. Identificação dinâmica de melhor e pior transportadora. Tempo médio de entrega com variação YoY. Taxa de pedidos entregues, em trânsito e atrasados. Desempenho logístico detalhado por estado com drill-down por transportadora.

**Clientes:** Estratificação ABC: quantidade e lucro por classe. Percentual de concentração de receita nos clientes Classe A. Lista ordenada de clientes com classificação e indicador visual. Narrativa automática sobre grau de concentração da carteira. Distribuição percentual de receita entre as três classes.

---

## 11. Estrutura Técnica

| Item | Quantidade |
|---|---|
| Tabelas de dados (visíveis) | 5 |
| Tabelas auxiliares (LocalDateTable — ocultas) | 5 |
| Tabela de medidas (Medidas) | 1 |
| **Total de tabelas no modelo** | **11** |
| **Medidas DAX** | **59** |
| **Colunas calculadas** | **5** |
| **Páginas do dashboard** | **4** |
| Relacionamentos ativos | 7 |
| Nível de compatibilidade | 1600 |
| Idioma do modelo | Português Brasileiro (1046) |
| Volume da fato | ~50.000 linhas |
| Tamanho estimado do modelo | 8,3 MB |
| Data de criação do modelo | 03/04/2026 |
| Última atualização | 07/06/2026 |

### Stack Tecnológico

| Ferramenta | Finalidade |
|---|---|
| Power BI Desktop | Desenvolvimento do modelo semântico e dashboard |
| Power BI Service | Publicação e compartilhamento via iframe |
| Python / Pandas | Geração e exportação da base simulada |
| Excel (.xlsx) | Formato de importação dos dados |
| Star Schema | Padrão de modelagem dimensional |
| DAX | Linguagem de medidas e colunas calculadas |

---

*Documentação gerada via conexão direta ao modelo semântico utilizando Power BI MCP.*
*Ruan Estevão Barbosa Lopes — ruanestevao51@gmail.com — linkedin.com/in/ruan-lopes-8556573b0*
EOF
echo "README gerado: $(wc -l < /home/claude/README_final.md) linhas"