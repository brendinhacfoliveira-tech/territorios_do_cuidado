# ============================================================
# ETAPA 0 - INSTALAÇÃO E IMPORTAÇÃO DAS BIBLIOTECAS
# ============================================================

!pip install -q pandas numpy scipy statsmodels openpyxl plotly

from pathlib import Path
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import statsmodels.api as sm
import plotly.graph_objects as go
from google.colab import files


# ============================================================
# ANÁLISE ESTATÍSTICA DA HANSENÍASE POR ESTADO
# ============================================================

# ============================================================
# 1. CONFIGURAÇÕES
# ============================================================

PASTA = Path("/content/projeto_hanseniase")

URL_ARQUIVO = (
    "https://raw.githubusercontent.com/"
    "brendinhacfoliveira-tech/territorios_do_cuidado/main/base_projeto.xlsx"
)

ABA = "Casos_hanseniase"
HORIZONTE = 10

RESULTADOS = PASTA / "resultados"
GRAFICOS = RESULTADOS / "graficos"
GRAFICOS.mkdir(parents=True, exist_ok=True)


# ============================================================
# 2. IMPORTAÇÃO DA BASE
# ============================================================

df_original = pd.read_excel(
    URL_ARQUIVO,
    sheet_name=ABA,
    engine="openpyxl"
)

df_original.columns = df_original.columns.astype(str).str.strip()


# ------------------------------------------------------------
# ETAPA 3 - TRATAMENTO E ORGANIZAÇÃO DOS DADOS
# ------------------------------------------------------------

def converter_valor(valor):
    if pd.isna(valor):
        return np.nan
    if isinstance(valor, (int, float, np.number)):
        return float(valor)
    texto = str(valor).strip()
    if texto in ["", "-", "—", "nan", "None"]:
        return np.nan
    if "," in texto:
        texto = texto.replace(".", "").replace(",", ".")
    else:
        partes = texto.split(".")
        if len(partes) == 2 and len(partes[1]) == 3 and partes[0].isdigit():
            texto = texto.replace(".", "")
    return pd.to_numeric(texto, errors="coerce")


# Verifica se já existe coluna "Ano" ou se os anos estão nas colunas com estados na 1ª coluna
if "Ano" in df_original.columns:
    df = df_original.copy()
    df["Ano"] = pd.to_numeric(df["Ano"], errors="coerce")
    df = df.dropna(subset=["Ano"])
    df["Ano"] = df["Ano"].astype(int)
    estados = [col for col in df.columns if col != "Ano"]
    for estado in estados:
        df[estado] = df[estado].apply(converter_valor)
    df = df.sort_values("Ano").reset_index(drop=True)
else:
    col_uf = df_original.columns[0]
    base_temp = df_original.rename(columns={col_uf: "UF"}).copy()
    base_temp["UF"] = base_temp["UF"].astype(str).str.strip().str.upper()
    base_temp = base_temp[
        ~base_temp["UF"].isin(["", "NAN", "NONE", "UF/ANO", "UF", "TOTAL"])
    ]

    colunas_anos = {}
    for col in base_temp.columns:
        if col == "UF":
            continue
        try:
            ano = int(str(col).strip())
            if 1900 <= ano <= 2100:
                colunas_anos[col] = ano
        except ValueError:
            continue

    base_longa = base_temp.melt(
        id_vars="UF",
        value_vars=list(colunas_anos.keys()),
        var_name="Col_Ano",
        value_name="Valor_Bruto"
    )
    base_longa["Ano"] = base_longa["Col_Ano"].map(colunas_anos)
    base_longa["Casos"] = base_longa["Valor_Bruto"].apply(converter_valor)

    df = base_longa.pivot_table(
        index="Ano",
        columns="UF",
        values="Casos",
        aggfunc="first"
    ).sort_index().reset_index()
    df.columns.name = None
    estados = [col for col in df.columns if col != "Ano"]

print("\nEstrutura da base:")
display(df)
print("\nEstados analisados:", estados)
print("\nQuantidade de observações:", df.shape[0])


# ------------------------------------------------------------
# ETAPA 4 - ESTATÍSTICA DESCRITIVA POR ESTADO
# ------------------------------------------------------------

medidas = {}

for estado in estados:
    dados = df[estado].dropna()
    modas = dados.mode()
    moda = ", ".join(map(str, modas.tolist())) if len(modas) > 0 else np.nan

    medidas[estado] = {
        "N": dados.count(),
        "Média": dados.mean(),
        "Mediana": dados.median(),
        "Moda": moda,
        "Mínimo": dados.min(),
        "Máximo": dados.max(),
        "Amplitude": dados.max() - dados.min() if len(dados) > 0 else np.nan,
        "Variância": dados.var(ddof=1),
        "Desvio Padrão": dados.std(ddof=1),
        "Coeficiente de Variação (%)": (
            (dados.std(ddof=1) / dados.mean()) * 100
            if len(dados) > 0 and dados.mean() != 0 else np.nan
        ),
        "1º Quartil": dados.quantile(0.25),
        "3º Quartil": dados.quantile(0.75),
        "Assimetria": dados.skew(),
        "Curtose": dados.kurtosis()
    }

tabela_descritiva = pd.DataFrame(medidas)
print("\nTABELA DE ESTATÍSTICA DESCRITIVA")
display(tabela_descritiva.round(3))


# ------------------------------------------------------------
# ETAPA 5 - MATRIZ DE CORRELAÇÃO
# ------------------------------------------------------------

matriz_correlacao = df.corr(method="pearson")
print("\nMATRIZ DE CORRELAÇÃO")
display(matriz_correlacao.round(3))

plt.figure(figsize=(12, 8))
plt.imshow(matriz_correlacao, cmap="coolwarm", vmin=-1, vmax=1)
plt.colorbar(label="Correlação de Pearson")
plt.xticks(range(len(matriz_correlacao.columns)), matriz_correlacao.columns, rotation=45, ha="right")
plt.yticks(range(len(matriz_correlacao.index)), matriz_correlacao.index)

for i in range(len(matriz_correlacao.index)):
    for j in range(len(matriz_correlacao.columns)):
        valor = matriz_correlacao.iloc[i, j]
        if pd.notna(valor):
            plt.text(j, i, f"{valor:.2f}", ha="center", va="center", fontsize=8)

plt.title("Matriz de Correlação entre as Variáveis")
plt.tight_layout()
plt.savefig(GRAFICOS / "matriz_correlacao.png", dpi=150)
plt.show()


# ------------------------------------------------------------
# ETAPA 6 - ANÁLISE DAS MEDIDAS AO LONGO DOS ANOS
# ------------------------------------------------------------

medidas_por_ano = []

for ano in sorted(df["Ano"].unique()):
    dados_ano = df.loc[df["Ano"] == ano, estados].iloc[0]
    valores = dados_ano.dropna()

    medidas_por_ano.append({
        "Ano": ano,
        "Média": valores.mean(),
        "Mediana": valores.median(),
        "Moda": valores.mode().iloc[0] if len(valores.mode()) > 0 else np.nan,
        "Amplitude": valores.max() - valores.min() if len(valores) > 0 else np.nan,
        "Variância": valores.var(ddof=1),
        "Desvio Padrão": valores.std(ddof=1),
        "Mínimo": valores.min(),
        "Máximo": valores.max()
    })

tabela_ano = pd.DataFrame(medidas_por_ano)
print("\nMEDIDAS ESTATÍSTICAS POR ANO")
display(tabela_ano.round(3))


# ------------------------------------------------------------
# IDENTIFICAÇÃO DOS ANOS COM MAIORES E MENORES MEDIDAS
# ------------------------------------------------------------

medidas_comparar = ["Média", "Mediana", "Amplitude", "Variância", "Desvio Padrão"]
resultados_temporais = []

for medida in medidas_comparar:
    if tabela_ano[medida].notna().any():
        indice_max = tabela_ano[medida].idxmax()
        indice_min = tabela_ano[medida].idxmin()
        ano_max = tabela_ano.loc[indice_max, "Ano"]
        val_max = tabela_ano.loc[indice_max, medida]
        ano_min = tabela_ano.loc[indice_min, "Ano"]
        val_min = tabela_ano.loc[indice_min, medida]
    else:
        ano_max = val_max = ano_min = val_min = np.nan

    valor_2024 = tabela_ano.loc[tabela_ano["Ano"] == 2024, medida]

    resultados_temporais.append({
        "Medida": medida,
        "Ano Maior Valor": ano_max,
        "Maior Valor": val_max,
        "Ano Menor Valor": ano_min,
        "Menor Valor": val_min,
        "Valor em 2024": valor_2024.iloc[0] if len(valor_2024) > 0 else np.nan
    })

comparacao_temporal = pd.DataFrame(resultados_temporais)
print("\nCOMPARAÇÃO DAS MEDIDAS AO LONGO DO TEMPO")
display(comparacao_temporal.round(3))


# ------------------------------------------------------------
# ETAPA 7 - REGRESSÕES LINEARES POR ESTADO
# ------------------------------------------------------------

resultados_regressao = []
modelos = {}

for estado in estados:
    dados_estado = df[["Ano", estado]].dropna()
    if len(dados_estado) < 2:
        continue

    X = dados_estado["Ano"]
    Y = dados_estado[estado]
    X_modelo = sm.add_constant(X)
    modelo = sm.OLS(Y, X_modelo).fit()
    modelos[estado] = modelo

    intercepto = modelo.params["const"]
    coeficiente = modelo.params["Ano"]
    r2 = modelo.rsquared
    p_valor = modelo.pvalues["Ano"]

    if coeficiente > 0:
        tendencia = "Crescente"
    elif coeficiente < 0:
        tendencia = "Decrescente"
    else:
        tendencia = "Estável"

    resultados_regressao.append({
        "Estado": estado,
        "Intercepto": intercepto,
        "Coeficiente_Ano": coeficiente,
        "R²": r2,
        "P-valor": p_valor,
        "Tendência": tendencia,
        "Equação": f"Y = {intercepto:.2f} + ({coeficiente:.4f} × Ano)"
    })

tabela_regressoes = pd.DataFrame(resultados_regressao)
print("\nRESULTADOS DAS REGRESSÕES LINEARES")
display(tabela_regressoes.round(4))


# ------------------------------------------------------------
# ETAPA 8 - VISUALIZAÇÃO DAS REGRESSÕES HISTÓRICAS
# ------------------------------------------------------------

for estado in estados:
    if estado not in modelos:
        continue

    dados_estado = df[["Ano", estado]].dropna()
    modelo = modelos[estado]
    X = dados_estado["Ano"]
    Y = dados_estado[estado]
    Y_estimado = modelo.predict(sm.add_constant(X))

    plt.figure(figsize=(10, 5))
    plt.scatter(X, Y, label="Casos observados")
    plt.plot(X, Y_estimado, label="Regressão linear", linewidth=2)
    plt.title(f"Regressão Linear - {estado}")
    plt.xlabel("Ano")
    plt.ylabel("Número de casos")
    plt.legend()
    plt.grid(alpha=0.3)
    plt.tight_layout()
    plt.savefig(GRAFICOS / f"regressao_{estado}.png", dpi=150)
    plt.show()


# ------------------------------------------------------------
# ETAPA 9 - PROJEÇÃO DOS PRÓXIMOS 10 ANOS
# ------------------------------------------------------------

ultimo_ano = int(df["Ano"].max())
anos_futuros = np.arange(ultimo_ano + 1, ultimo_ano + HORIZONTE + 1)
projecoes = pd.DataFrame({"Ano": anos_futuros})

for estado in estados:
    if estado not in modelos:
        projecoes[estado] = np.nan
        continue

    modelo = modelos[estado]
    X_futuro = sm.add_constant(pd.DataFrame({"Ano": anos_futuros}), has_constant="add")
    estimativas = np.maximum(modelo.predict(X_futuro), 0)
    projecoes[estado] = estimativas

print("\nPROJEÇÃO DOS PRÓXIMOS 10 ANOS")
display(projecoes.round(2))


# ------------------------------------------------------------
# ETAPA 10 - ORGANIZAÇÃO DOS DADOS PARA A ANIMAÇÃO
# ------------------------------------------------------------

projecoes_long = projecoes.melt(
    id_vars="Ano",
    var_name="Estado",
    value_name="Casos_Estimados"
)
projecoes_long["Casos_Estimados"] = pd.to_numeric(
    projecoes_long["Casos_Estimados"],
    errors="coerce"
)

ano_corte_hist = max(int(df["Ano"].min()), ultimo_ano - 4)
historico_long = df[df["Ano"] >= ano_corte_hist].melt(
    id_vars="Ano",
    value_vars=estados,
    var_name="Estado",
    value_name="Casos"
)


# ------------------------------------------------------------
# ETAPA 11 - CONSTRUÇÃO DOS FRAMES DA ANIMAÇÃO
# ------------------------------------------------------------

frames = []

for ano_atual in anos_futuros:
    dados_frame = []

    for estado in estados:
        historico_estado = historico_long[
            (historico_long["Estado"] == estado) &
            (historico_long["Ano"] <= ultimo_ano)
        ]
        projecao_estado = projecoes_long[
            (projecoes_long["Estado"] == estado) &
            (projecoes_long["Ano"] <= ano_atual)
        ]

        dados_historicos = pd.DataFrame({
            "Ano": historico_estado["Ano"],
            "Valor": historico_estado["Casos"],
            "Tipo": "Observado"
        })
        dados_projetados = pd.DataFrame({
            "Ano": projecao_estado["Ano"],
            "Valor": projecao_estado["Casos_Estimados"],
            "Tipo": "Estimado"
        })

        dados_estado = pd.concat(
            [dados_historicos, dados_projetados],
            ignore_index=True
        ).sort_values("Ano")

        dados_frame.append({
            "Estado": estado,
            "Ano": dados_estado["Ano"].tolist(),
            "Valor": dados_estado["Valor"].tolist(),
            "Tipo": dados_estado["Tipo"].tolist()
        })

    frames.append({
        "Ano": int(ano_atual),
        "Dados": dados_frame
    })


# ------------------------------------------------------------
# ETAPA 12 - ANIMAÇÃO INTERATIVA
# ------------------------------------------------------------

fig = go.Figure()

for estado in estados:
    historico_estado = historico_long[historico_long["Estado"] == estado]
    projecao_inicial = projecoes_long[
        (projecoes_long["Estado"] == estado) &
        (projecoes_long["Ano"] <= ultimo_ano)
    ]

    dados_iniciais = pd.concat([
        pd.DataFrame({
            "Ano": historico_estado["Ano"],
            "Valor": historico_estado["Casos"],
            "Tipo": "Observado"
        }),
        pd.DataFrame({
            "Ano": projecao_inicial["Ano"],
            "Valor": projecao_inicial["Casos_Estimados"],
            "Tipo": "Estimado"
        })
    ], ignore_index=True).sort_values("Ano")

    fig.add_trace(
        go.Scatter(
            x=dados_iniciais["Ano"],
            y=dados_iniciais["Valor"],
            mode="lines+markers",
            name=estado,
            text=[
                f"{valor:.0f}" if tipo == "Estimado" and pd.notna(valor) else ""
                for valor, tipo in zip(dados_iniciais["Valor"], dados_iniciais["Tipo"])
            ],
            textposition="top center",
            hovertemplate="<b>%{fullData.name}</b><br>Ano: %{x}<br>Casos: %{y:.0f}<extra></extra>"
        )
    )

frames_plotly = []

for frame in frames:
    traces_frame = []
    for dados_estado in frame["Dados"]:
        traces_frame.append(
            go.Scatter(
                x=dados_estado["Ano"],
                y=dados_estado["Valor"],
                mode="lines+markers",
                name=dados_estado["Estado"],
                text=[
                    f"{valor:.0f}" if tipo == "Estimado" and pd.notna(valor) else ""
                    for valor, tipo in zip(dados_estado["Valor"], dados_estado["Tipo"])
                ],
                textposition="top center",
                hovertemplate="<b>%{fullData.name}</b><br>Ano: %{x}<br>Casos: %{y:.0f}<extra></extra>"
            )
        )

    frames_plotly.append(
        go.Frame(
            data=traces_frame,
            name=str(frame["Ano"]),
            layout=go.Layout(title_text=f"Projeção de Hanseníase por Estado - {frame['Ano']}")
        )
    )

fig.frames = frames_plotly


# ------------------------------------------------------------
# ETAPA 13 - CONFIGURAÇÃO DA ANIMAÇÃO
# ------------------------------------------------------------

fig.update_layout(
    title=f"Projeção da Hanseníase por Estado - {anos_futuros[0]}",
    xaxis=dict(
        title="Ano",
        range=[ano_corte_hist, int(anos_futuros[-1])],
        dtick=2
    ),
    yaxis_title="Número estimado de casos",
    template="plotly_white",
    width=1200,
    height=700,
    legend_title="Estados",
    updatemenus=[
        {
            "type": "buttons",
            "showactive": False,
            "buttons": [
                {
                    "label": "▶ Reproduzir",
                    "method": "animate",
                    "args": [
                        None,
                        {
                            "frame": {"duration": 800, "redraw": True},
                            "transition": {"duration": 400},
                            "fromcurrent": True
                        }
                    ]
                },
                {
                    "label": "⏸ Pausar",
                    "method": "animate",
                    "args": [
                        [None],
                        {
                            "frame": {"duration": 0, "redraw": False},
                            "mode": "immediate",
                            "transition": {"duration": 0}
                        }
                    ]
                }
            ]
        }
    ],
    sliders=[
        {
            "active": 0,
            "currentvalue": {"prefix": "Ano projetado: "},
            "steps": [
                {
                    "label": str(frame["Ano"]),
                    "method": "animate",
                    "args": [
                        [str(frame["Ano"])],
                        {
                            "frame": {"duration": 500, "redraw": True},
                            "mode": "immediate",
                            "transition": {"duration": 300}
                        }
                    ]
                }
                for frame in frames
            ]
        }
    ]
)

fig.write_html(RESULTADOS / "projecao_hanseniase.html", include_plotlyjs=True)
fig.show()


# ------------------------------------------------------------
# ETAPA 14 - EXPORTAÇÃO DOS RESULTADOS
# ------------------------------------------------------------

nome_saida = str(RESULTADOS / "Analise_Hanseniase.xlsx")

with pd.ExcelWriter(nome_saida, engine="openpyxl") as writer:
    df.to_excel(writer, sheet_name="Base Tratada", index=False)
    tabela_descritiva.to_excel(writer, sheet_name="Estatistica Descritiva")
    matriz_correlacao.to_excel(writer, sheet_name="Correlacao")
    tabela_ano.to_excel(writer, sheet_name="Medidas por Ano", index=False)
    comparacao_temporal.to_excel(writer, sheet_name="Comparacao Temporal", index=False)
    tabela_regressoes.to_excel(writer, sheet_name="Regressoes", index=False)
    projecoes.to_excel(writer, sheet_name="Projecoes", index=False)

files.download(nome_saida)

print("\nANÁLISE CONCLUÍDA!")
