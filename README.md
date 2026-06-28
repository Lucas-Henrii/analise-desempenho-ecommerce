# Análise de Desempenho de E-commerce: Pipeline SQL & Power BI

Este projeto simula um cenário real de negócios onde os dados de vendas de um e-commerce foram estruturados, modelados e transformados utilizando conceitos avançados de engenharia de dados (Star Schema) em um banco de dados relacional, para posterior consumo analítico e tomada de decisão estratégica em um dashboard interativo.

## 📌 Problema de Negócio
A diretoria de uma empresa de e-commerce necessitava centralizar e analisar os dados de vendas que se encontravam dispersos. Havia a necessidade de responder rapidamente a perguntas críticas, como:
1. Qual o faturamento total acumulado?
2. Quais clientes possuem o maior valor de ciclo de vida (Lifetime Value - LTV)?
3. Qual é a taxa de crescimento/evolução das vendas mês a mês?

## 🛠️ Tecnologias Utilizadas
* **Banco de Dados:** PostgreSQL (Instância Local)
* **Linguagem de Consulta:** SQL (DDL, DML, Window Functions, CTEs e Views)
* **Visualização de Dados:** Power BI Desktop

## 🏗️ Modelagem dos Dados (Arquitetura)
Para garantir a performance das consultas e facilitar a conexão com ferramentas de BI, os dados foram modelados seguindo o conceito de **Star Schema (Esquema Estrela)**:
* **Tabelas Dimensão:** `dim_clientes`, `dim_produtos` (fornecem o contexto analítico).
* **Tabela Fato:** `fato_vendas` (contém as métricas quantificáveis e chaves estrangeiras).

---

## 💻 Desenvolvimento Técnico e Consultas SQL

Em vez de sobrecarregar a ferramenta de visualização com cálculos pesados, toda a lógica de negócio, junções e agregações foram processadas diretamente na camada do banco de dados através da criação de **Views otimizadas**:

### 1. Visão Geral de Vendas (Consolidação de Dados)
Junção da tabela fato com as dimensões para o cálculo do faturamento por item.
```sql
CREATE OR REPLACE VIEW vw_visao_geral_vendas AS
SELECT 
    f.id_venda, f.data_venda, c.nome_cliente, c.cidade, c.estado,
    p.nome_produto, p.categoria, f.quantidade, p.preco_unitario,
    (f.quantidade * p.preco_unitario) AS faturamento_total
FROM fato_vendas f
JOIN dim_clientes c ON f.id_cliente = c.id_cliente
JOIN dim_produtos p ON f.id_produto = p.id_produto;
