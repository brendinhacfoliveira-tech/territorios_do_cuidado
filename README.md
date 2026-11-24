# Territórios do Cuidado — Análise Estatística da Hanseníase por Estado

Projeto acadêmico voltado à análise estatística exploratória, temporal e preditiva dos casos de hanseníase por unidade federativa (UF), a partir da base `base_projeto.xlsx` (aba `Casos_hanseniase`).

## Base de Dados
- **Arquivo:** `base_projeto.xlsx`
- **Aba analisada:** `Casos_hanseniase`
- **Organização:** dados anuais de casos por estado, tratados e convertidos para análise numérica e temporal.

## Etapas da Análise
1. Importação e padronização da base de dados.
2. Estatística descritiva por estado (média, mediana, moda, mínimo, máximo, amplitude, variância, desvio padrão, coeficiente de variação, quartis, assimetria e curtose).
3. Matriz de correlação de Pearson e visualização gráfica.
4. Comparação das medidas estatísticas ao longo dos anos.
5. Regressões lineares simples por estado (`Casos` em função do `Ano`) e gráficos históricos.
6. Projeção linear exploratória para os próximos 10 anos e animação interativa em Plotly.
7. Exportação consolidada dos resultados em planilha Excel (`Analise_Hanseniase.xlsx`).

## Como Executar
1. Abra o código no **Google Colab** (ou baixe o repositório para rodar em ambiente Python local).
2. Certifique-se de que as bibliotecas necessárias estejam instaladas (`pandas`, `numpy`, `scipy`, `statsmodels`, `openpyxl`, `plotly`, `matplotlib`).
3. Execute as células em sequência para gerar as tabelas, gráficos, animação e arquivo Excel de saída.

> **Observação:** as projeções futuras são extrapolações lineares exploratórias com fins acadêmicos e não representam previsões epidemiológicas oficiais.
