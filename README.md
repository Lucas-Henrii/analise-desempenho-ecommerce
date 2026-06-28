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
```
### 2. Ranking de Clientes (Uso de Window Functions)
Demonstração de manipulação avançada utilizando a função de janela RANK() para ordenar os clientes por volume financeiro gerado (LTV).
```sql
CREATE OR REPLACE VIEW vw_ranking_clientes AS
SELECT 
    c.nome_cliente,
    SUM(f.quantidade * p.preco_unitario) AS total_gasto,
    RANK() OVER (ORDER BY SUM(f.quantidade * p.preco_unitario) DESC) AS posicao_ranking
FROM fato_vendas f
JOIN dim_clientes c ON f.id_cliente = c.id_cliente
JOIN dim_produtos p ON f.id_produto = p.id_produto
GROUP BY c.nome_cliente;
```
### 3. Evolução Mensal e Crescimento (Uso de CTEs e LAG)
Utilização de Expressões de Tabela Comuns (CTEs) e a função analítica LAG para recuperar o faturamento do mês anterior e calcular o crescimento percentual mês a mês de forma dinâmica.
```sql
CREATE OR REPLACE VIEW vw_evolucao_mensal AS
WITH faturamento_por_mes AS (
    SELECT 
        DATE_TRUNC('month', f.data_venda) AS mes,
        SUM(f.quantidade * p.preco_unitario) AS faturamento_atual
    FROM fato_vendas f
    JOIN dim_produtos p ON f.id_produto = p.id_produto
    GROUP BY DATE_TRUNC('month', f.data_venda)
)
SELECT 
    TO_CHAR(mes, 'YYYY-MM') AS ano_mes,
    faturamento_atual,
    LAG(faturamento_atual, 1) OVER (ORDER BY mes) AS faturamento_mes_anterior,
    COALESCE(
        ((faturamento_atual - LAG(faturamento_atual, 1) OVER (ORDER BY mes)) / 
        LAG(faturamento_atual, 1) OVER (ORDER BY mes)) * 100, 0
    ) AS percentual_crescimento
FROM faturamento_por_mes;
```
📊 Camada de Visualização (Power BI)
A conexão com o PostgreSQL foi feita em modo de Importação apontando diretamente para as Views analíticas criadas. O dashboard foi estruturado com foco em UX/UI e respostas rápidas para o negócio:

Métricas Principais: Cartão dinâmico exibindo o faturamento consolidado.

Comportamento Temporal: Gráfico de linhas ordenado cronologicamente demonstrando a variação de receita mês a mês.

Análise de Clientes: Gráfico de barras horizontais detalhando a soma de gastos exatos por cliente, identificando os líderes de receita.

🚀 Conclusão e Resultados
O projeto demonstra a capacidade prática de unir regras de negócio do dia a dia e conceitos técnicos de engenharia para criar soluções robustas de Business Intelligence. A centralização de dados e tratamento prévio via banco de dados reduziu a carga de processamento do relatório e forneceu visões analíticas limpas prontas para uso executivo.
