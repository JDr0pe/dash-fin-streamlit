import streamlit as st
import pandas as pd
import plotly.express as px

st.set_page_config(page_title="Dashboard Financeiro & RH", layout="wide")
st.title("Dashboard Financeiro & RH (Excel)")

st.sidebar.header("Configuração dos dados")
uploaded_file = st.sidebar.file_uploader(
    "Envie o arquivo Excel (com abas Financeiro e RH)",
    type=["xlsx", "xls"]
)

@st.cache_data
def carregar_dados(arquivo):
    # Lê as duas abas específicas
    df_fin = pd.read_excel(arquivo, sheet_name="Financeiro")
    df_rh = pd.read_excel(arquivo, sheet_name="RH")
    return df_fin, df_rh

if uploaded_file is None:
    st.info("Envie um arquivo Excel na barra lateral para começar.")
else:
    df_fin, df_rh = carregar_dados(uploaded_file)

    # Filtros
    anos_disponiveis = sorted(df_fin["ano"].dropna().unique())
    ano_sel = st.sidebar.selectbox("Ano", anos_disponiveis)

    df_fin_filtrado = df_fin[df_fin["ano"] == ano_sel]
    df_rh_filtrado = df_rh[df_rh["ano"] == ano_sel]

    # KPIs principais
    receita_total = df_fin_filtrado["receita"].sum()
    despesa_total = df_fin_filtrado["despesa"].sum()
    lucro = receita_total - despesa_total
    headcount = df_rh_filtrado[df_rh_filtrado["status"] == "Ativo"]["id_func"].nunique()

    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Receita total", f"R$ {receita_total:,.2f}")
    col2.metric("Despesas", f"R$ {despesa_total:,.2f}")
    col3.metric("Lucro", f"R$ {lucro:,.2f}")
    col4.metric("Headcount ativo", int(headcount))

    st.markdown("### Visão Financeira")
    col_f1, col_f2 = st.columns(2)

    # Receita x Despesa por mês
    df_mes = (
        df_fin_filtrado
        .groupby("mes", as_index=False)[["receita", "despesa"]]
        .sum()
    )
    fig_fin_mes = px.bar(
        df_mes,
        x="mes",
        y=["receita", "despesa"],
        barmode="group",
        title="Receita x Despesa por mês"
    )
    col_f1.plotly_chart(fig_fin_mes, use_container_width=True)

    # Despesa por centro de custo
    df_cc = (
        df_fin_filtrado
        .groupby("centro_custo", as_index=False)["despesa"]
        .sum()
    )
    fig_cc = px.bar(
        df_cc,
        x="centro_custo",
        y="despesa",
        title="Despesas por centro de custo"
    )
    col_f2.plotly_chart(fig_cc, use_container_width=True)

    st.markdown("### Visão de RH")
    col_r1, col_r2 = st.columns(2)

    # Headcount por departamento
    df_head_dep = (
        df_rh_filtrado[df_rh_filtrado["status"] == "Ativo"]
        .groupby("departamento", as_index=False)["id_func"]
        .nunique()
    )
    fig_head_dep = px.bar(
        df_head_dep,
        x="departamento",
        y="id_func",
        title="Headcount por departamento"
    )
    col_r1.plotly_chart(fig_head_dep, use_container_width=True)

    # Custo de pessoal (soma salário) por departamento
    df_sal_dep = (
        df_rh_filtrado[df_rh_filtrado["status"] == "Ativo"]
        .groupby("departamento", as_index=False)["salario"]
        .sum()
    )
    fig_sal_dep = px.bar(
        df_sal_dep,
        x="departamento",
        y="salario",
        title="Custo de pessoal por departamento"
    )
    col_r2.plotly_chart(fig_sal_dep, use_container_width=True)

    # Opcional: mostrar dados brutos
    with st.expander("Ver dados brutos Financeiro"):
        st.dataframe(df_fin_filtrado)
    with st.expander("Ver dados brutos RH"):
        st.dataframe(df_rh_filtrado)
