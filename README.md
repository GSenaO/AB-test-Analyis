# 🍔 Fast Food Marketing: Análise de Teste A/B e Dados Sintéticos

![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Statsmodels](https://img.shields.io/badge/Statsmodels-blue?style=for-the-badge)

## 📌 Visão Geral do Projeto
Este projeto analisa os resultados de um teste A/B para uma rede de fast-food que planeja lançar um novo item no menu. O desafio central foi determinar qual de **três campanhas promocionais** obteve o melhor desempenho em vendas.

Um diferencial técnico deste repositório é o tratamento de **dados faltantes** através de **síntese estatística**, garantindo que a análise final fosse robusta e livre de vieses causados por remoção de amostras.

* **Base de Dados:** [Fast Food Marketing Campaign A/B Test (Kaggle)](https://www.kaggle.com/datasets/chebotinaa/fast-food-marketing-campaign-ab-test).

---

## 🏗️ Estrutura do Repositório
O projeto está organizado de forma modular em dois notebooks principais:

1.  **`EDA.ipynb` (Exploração e Síntese):** Focado em carregamento, limpeza, identificação de lacunas e geração de dados sintéticos via regressão.
2.  **`AnaliseDeResultados.ipynb` (Estatística):** Contém os testes de hipóteses (ANOVA + Tukey), cálculos de *lift* e visualizações finais.

---

## 🛠️ O Desafio Técnico: Dados Faltantes
Identificamos que o **MarketID = 2** não possuía registros para a **Promoção 2** (4 semanas ausentes). Este mercado é de tamanho pequeno ("Small").

### 💡 Solução Aplicada: Regressão Linear com Normalização
Para manter a integridade estatística, optei por não excluir o mercado, mas sim realizar uma **imputação avançada**:
* **Filtro:** Seleção apenas da base de treino de mercados de tamanho similar (`MarketSize = 'Small'`).
* **Normalização:** Aplicação de `StandardScaler` (Z-Score) nas features (`week`, `AgeOfStore`, `Promotion`, `MarketID`) para garantir que a regressão não sofra viés de escala.
* **Modelo:** Treinamento de uma `LinearRegression` para predizer os valores semanais de `SalesInThousands` ausentes. Os valores preditos (60.58 a 62.37) mantiveram a consistência da amostra.

---

## 🧪 Metodologia Estatística
A análise foi conduzida com rigor estatístico para evitar decisões baseadas apenas em médias superficiais.

* **Métrica Principal:** Vendas em milhares (`SalesInThousands`).
* **Hipótese Nula (H0):** Não há diferença significativa entre as promoções.
* **Testes Realizados:**
    1.  **ANOVA:** Para verificar se existe qualquer diferença global entre os grupos (Resultado: $F = 20.6156$, $p = 2.328e-09$).
    2.  **Pós-teste de Tukey:** Para identificar especificamente quais pares de promoções diferem entre si, corrigindo o erro de múltiplas comparações (Alpha = 0.05).

---

## 📊 Resultados e Insights
Abaixo, os resultados consolidados após a análise estatística:

| Promoção | Média de Vendas (k) | Lift vs. Promo 1 | Diferença Significativa? |
| :--- | :--- | :--- | :--- |
| **Promoção 1** | **58.10** | Controle | - |
| **Promoção 3** | 55.36 | -4.77% | **Não** (vs Promo 1, $p = 0.2448$) |
| **Promoção 2** | 47.69 | -17.95% | **Sim** (Pior desempenho) |

### 🚀 Conclusão de Negócio
* **Vencedora em Vendas Absolutas:** A **Promoção 1** apresentou a maior média absoluta.
* **Insight Crítico:** O teste de Tukey revelou que a Promoção 3 é estatisticamente equivalente à Promoção 1.
* **Recomendação:** A empresa pode optar tanto pela Promoção 1 quanto pela Promoção 3. A decisão final deve ser baseada em critérios de negócio secundários (CAC, custos de produção ou facilidade de implementação). **Evite a Promoção 2**.

---

## ⚙️ Como Utilizar
### Pré-requisitos
```bash
pip install pandas numpy scipy scikit-learn matplotlib seaborn statsmodels
