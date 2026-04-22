================================================================================
                ANÁLISE AB TEST - CAMPANHA DE MARKETING FAST FOOD
================================================================================

OBJETIVO DO PROJETO
================================================================================
Analisar os resultados de um teste A/B de campanha de marketing em redes de 
fast food, identificando qual promoção gera melhor desempenho (vendas) e 
preenchendo lacunas de dados faltantes através de técnicas de síntese estatística.

ESTRUTURA DO PROJETO
================================================================================
O projeto é dividido em dois notebooks principais:

1. EDA.ipynb (Exploratory Data Analysis)
   - Carregamento de dados
   - Exploração de lacunas de dados
   - Geração de dados sintéticos
   - Preparação de dados para análise

2. AnaliseDeSResultados.ipynb (Análise de Resultados)
   - Análise estatística ANOVA + Tukey
   - Cálculo de lift entre promoções
   - Visualizações e boxplots
   - Conclusões e recomendações

DADOS
================================================================================
Arquivo: WA_Marketing-Campaign.csv
Registros originais: 548
Registros após síntese: 552 (4 registros sintéticos adicionados)
Registros ajustados: WA_Marketing-Campaign-Ajustado.csv

Colunas principais:
- MarketID: Identificador do mercado (1-10)
- MarketSize: Tamanho do mercado (Small, Medium, Large)
- LocationID: Identificador do local
- AgeOfStore: Idade da loja
- Promotion: Tipo de promoção (1, 2, 3)
- week: Semana da observação
- SalesInThousands: Vendas em milhares (métrica principal)

PROBLEMA IDENTIFICADO
================================================================================
MarketID = 2 não possui registros para Promotion = 2, criando uma lacuna
nos dados. Este mercado é pequeno (Small) e só possui dados para:
  - Promotion 1: 4 registros
  - Promotion 3: 20 registros
  - Promotion 2: FALTAM 4 REGISTROS

SOLUÇÃO APLICADA
================================================================================
Geração de dados sintéticos usando Regressão Linear com Normalização (z-score):

1. Filtrar dados de mercados com MarketSize = "Small"
2. Aplicar normalização StandardScaler às features numéricas:
   - week
   - Promotion
   - AgeOfStore
   - MarketID
3. Treinar modelo LinearRegression com dados normalizados
4. Predizer valores para cada semana faltante em MarketID = 2, Promotion = 2
5. Adicionar registros sintéticos ao dataset original

Resultados:
- Promotion 2, MarketID 2, week 1: 60.58 (SalesInThousands)
- Promotion 2, MarketID 2, week 2: 61.18 (SalesInThousands)
- Promotion 2, MarketID 2, week 3: 61.77 (SalesInThousands)
- Promotion 2, MarketID 2, week 4: 62.37 (SalesInThousands)

FUNÇÕES IMPLEMENTADAS
================================================================================

1. check_bias_and_stratification()
   ────────────────────────────────────────────────────────────────────────────
   Localização: EDA.ipynb (bloco 1)
   
   Propósito:
   Verifica se há viés prévio nos grupos do teste A/B ou se a estratificação
   dos grupos foi adequada.
   
   Parâmetros:
   - df: DataFrame com os dados
   - group_col: Coluna de agrupamento (padrão: 'Promotion')
   - covariates: Variáveis a testar (padrão: MarketSize, AgeOfStore, LocationID, MarketID)
   
   Retorno:
   Dicionário com:
     - 'summary': Estatísticas descritivas (média, std, contagem)
     - 'tests': Resultados de testes estatísticos (Chi-square, ANOVA, t-test)
     - 'bias_report': Relatório textual de possíveis vieses
   
   Lógica:
   - Separa covariáveis em numéricas e categóricas
   - Para categóricas: aplica Chi-square test
   - Para numéricas com >2 grupos: aplica ANOVA
   - Para numéricas com 2 grupos: aplica t-test
   - Identifica vieses com p-value < 0.05


2. calculate_lift()
   ────────────────────────────────────────────────────────────────────────────
   Localização: EDA.ipynb e AnaliseDeSResultados.ipynb
   
   Propósito:
   Calcula o lift (ganho percentual) de SalesInThousands entre as promoções,
   usando uma promoção como controle.
   
   Parâmetros:
   - df: DataFrame com os dados
   - control_promotion: Promoção de controle (padrão: 1)
   
   Retorno:
   Dicionário com lifts percentuais:
     Exemplo: {'Promotion 2 vs 1': -17.95, 'Promotion 3 vs 1': -4.77}
   
   Fórmula:
   lift = (média_promoção - média_controle) / média_controle * 100
   
   Interpretação:
   - Valor positivo: promoção supera o controle
   - Valor negativo: promoção performa pior que o controle


3. promotion_anova_tukey()
   ────────────────────────────────────────────────────────────────────────────
   Localização: EDA.ipynb e AnaliseDeSResultados.ipynb
   
   Propósito:
   Realiza ANOVA para testar diferenças globais entre promoções e, se 
   significativo, aplica o teste post-hoc de Tukey para comparações pareadas.
   
   Parâmetros:
   - df: DataFrame com os dados
   - metric: Métrica a analisar (padrão: 'SalesInThousands')
   - group_col: Coluna de agrupamento (padrão: 'Promotion')
   
   Retorno:
   Dicionário com:
     - 'anova': {'f_stat': F-estatístico, 'p_value': p-valor}
     - 'tukey_df': DataFrame com comparações pareadas de Tukey
     - 'means': Médias de vendas por promoção (ordenadas descendente)
     - 'best_promotion': Promoção com maior média
     - 'best_significant_vs_others': Booleano se melhor vs todas
     - 'comparisons_with_best': DataFrame com comparações da melhor promoção
   
   Testes Estatísticos:
   1. ANOVA: Testa H0 = "todas as promoções têm média igual"
      - Se p < 0.05: rejeita H0 (há diferença significativa)
   
   2. Tukey HSD: Comparação pareada com correção de múltiplos testes
      - Identifica quais pares diferem significativamente
      - Fornece intervalos de confiança para diferenças


4. plot_sales_boxplots()
   ────────────────────────────────────────────────────────────────────────────
   Localização: EDA.ipynb e AnaliseDeSResultados.ipynb
   
   Propósito:
   Gera visualizações em boxplot para identificar outliers e distribuição
   de SalesInThousands por promoção.
   
   Parâmetros:
   - df: DataFrame com os dados
   
   Saída:
   Gráfico Matplotlib com boxplots para cada promoção
   
   Interpretação do Boxplot:
   - Linha central: mediana
   - Caixa: intervalo interquartil (IQR)
   - Bigodes: limites de 1.5*IQR
   - Pontos: outliers além de 1.5*IQR


5. generate_synthetic_data_with_linear_regression()
   ────────────────────────────────────────────────────────────────────────────
   Localização: EDA.ipynb (bloco 7)
   
   Propósito:
   Gera dados sintéticos para combinações ausentes de MarketID e Promotion
   usando regressão linear com normalização StandardScaler (z-score).
   
   Parâmetros:
   - df: DataFrame original
   - target_market_id: ID do mercado alvo (padrão: 2)
   - target_promotion: ID da promoção alvo (padrão: 2)
   - market_size_filter: Tamanho de mercado para base de treino (padrão: 'Small')
   
   Retorno:
   DataFrame com registros sintéticos contendo todas as colunas originais
   
   Processo Detalhado:
   
   1. FILTRAGEM:
      - Seleciona apenas mercados com MarketSize = 'Small'
      - Copia atributos do mercado alvo (AgeOfStore, LocationID)
   
   2. NORMALIZAÇÃO (StandardScaler - z-score):
      - Aplica transformação Z = (X - μ) / σ
      - Colunas normalizadas: week, Promotion, AgeOfStore, MarketID
      - Garante que o modelo receba dados em escala padrão
   
   3. TREINAMENTO:
      - Treina LinearRegression com features normalizadas
      - Target: SalesInThousands
      - Modelo aprende relacionamento entre features e vendas
   
   4. PREDIÇÃO:
      - Para cada semana no mercado 2:
         a) Cria vetor com [week, promotion=2, AgeOfStore, MarketID=2]
         b) Normaliza usando o mesmo scaler do treino
         c) Prediz SalesInThousands com o modelo treinado
   
   5. CONSOLIDAÇÃO:
      - Retorna DataFrame com 4 registros sintéticos
      - Uma entrada para cada semana do mercado 2


PASSOS DO WORKFLOW
================================================================================

PASSO 1: PREPARAÇÃO DE DADOS (EDA.ipynb)
────────────────────────────────────────────────────────────────────────────
1.1 Carregar dados CSV original
1.2 Explorar distribuição de MarketID vs Promotion
1.3 Identificar lacunas (MarketID=2, Promotion=2 faltando)
1.4 Verificar MarketSize do mercado 2 (Small)

PASSO 2: SÍNTESE DE DADOS (EDA.ipynb)
────────────────────────────────────────────────────────────────────────────
2.1 Aplicar z-score às variáveis numéricas
2.2 Treinar modelo de regressão linear normalizado
2.3 Gerar 4 registros sintéticos para MarketID=2, Promotion=2
2.4 Concatenar com dados originais (df_augmented)
2.5 Salvar em WA_Marketing-Campaign-Ajustado.csv

PASSO 3: ANÁLISE DE VIÉS (EDA.ipynb)
────────────────────────────────────────────────────────────────────────────
3.1 Executar check_bias_and_stratification()
3.2 Verificar p-values de Chi-square/ANOVA/t-test
3.3 Confirmar se grupos estão bem estratificados (p > 0.05)

PASSO 4: ANÁLISE ESTATÍSTICA (AnaliseDeSResultados.ipynb)
────────────────────────────────────────────────────────────────────────────
4.1 Carregar dados ajustados
4.2 Calcular ANOVA e Tukey para promoções
4.3 Identificar melhor promoção por média
4.4 Verificar se melhor é significativa vs outras

PASSO 5: CÁLCULO DE LIFT (AnaliseDeSResultados.ipynb)
────────────────────────────────────────────────────────────────────────────
5.1 Calcular lift de cada promoção vs controle (Promotion 1)
5.2 Interpretar ganho/perda percentual

PASSO 6: VISUALIZAÇÃO (AnaliseDeSResultados.ipynb)
────────────────────────────────────────────────────────────────────────────
6.1 Gerar boxplots para cada promoção
6.2 Criar gráficos de média por dimensão (week, MarketID, etc)
6.3 Identificar padrões e outliers

PASSO 7: CONCLUSÕES
────────────────────────────────────────────────────────────────────────────
7.1 Resumir resultados de ANOVA e Tukey
7.2 Recomendar melhor promoção
7.3 Documentar limitações (dados sintéticos para MarketID=2)


RESULTADOS PRINCIPAIS
================================================================================

Teste ANOVA:
F = 20.6156
p-value = 2.328e-09 (altamente significativo)
Conclusão: Há diferença significativa entre as promoções

Médias de SalesInThousands:
- Promotion 1: 58.10 (maior - melhor desempenho)
- Promotion 3: 55.36
- Promotion 2: 47.69 (menor)

Lift vs Promotion 1:
- Promotion 2 vs 1: -17.95% (pior)
- Promotion 3 vs 1: -4.77% (melhor que 2, mas pior que 1)

Teste Tukey:
- Promotion 1 vs 2: diferença significativa (p < 0.05)
- Promotion 1 vs 3: NÃO significativa (p = 0.2448)
- Promotion 2 vs 3: diferença significativa (p < 0.05)

RECOMENDAÇÃO:
Adotar Promotion 1 como estratégia principal.
Promotion 3 é alternativa viável (diferença não significativa).
Evitar Promotion 2 (perda de ~18% em vendas).


CONSIDERAÇÕES TÉCNICAS
================================================================================

1. NORMALIZAÇÃO COM STANDARDSCALER
   Por que usar?
   - Regressão linear é sensível a escala de features
   - StandardScaler (z-score) garante média=0, std=1
   - Melhora convergência do modelo
   - Dados de predição devem usar mesmo scaler
   
   Implementação:
   - scaler.fit_transform(): normaliza dados de treino
   - scaler.transform(): normaliza dados de predição
   - Garante coerência entre treino e predição

2. DADOS SINTÉTICOS
   Limitações:
   - Baseia-se em padrão de mercados Small
   - MarketID=2 pode ter particularidades não capturadas
   - Use com cautela em decisões críticas
   
   Validação:
   - Valores preditos (60-62) dentro da faixa típica
   - Consistent com promoção 2 em outros mercados

3. TESTE ESTATÍSTICO TUKEY
   Por que pós-hoc?
   - ANOVA testa diferença global
   - Tukey identifica pares específicos
   - Corrige múltiplas comparações (controla erro tipo I)
   - Alpha = 0.05 (5% significância)


COMO USAR OS NOTEBOOKS
================================================================================

1. EXECUÇÃO SEQUENCIAL
   - Executar EDA.ipynb na ordem dos blocos (1 a 10)
   - Depois executar AnaliseDeSResultados.ipynb
   - Garante que dados sejam preparados antes da análise

2. MODIFICAÇÕES POSSÍVEIS
   - Trocar mercado: target_market_id = 3
   - Trocar promoção: target_promotion = 1
   - Trocar tamanho: market_size_filter = 'Medium'

3. REUTILIZAÇÃO DAS FUNÇÕES
   Exemplo em novo notebook:
   
   from sklearn.linear_model import LinearRegression
   from sklearn.preprocessing import StandardScaler
   from scipy import stats
   from statsmodels.stats.multicomp import pairwise_tukeyhsd
   
   # [copiar funções do EDA.ipynb]
   
   result = promotion_anova_tukey(seu_df)
   print(result['means'])


ARQUIVOS GERADOS
================================================================================
1. WA_Marketing-Campaign-Ajustado.csv
   - Dataset com dados sintéticos inclusos
   - 552 registros (548 originais + 4 sintéticos)
   - Pronto para análises futuras

2. Gráficos (salvos automaticamente)
   - Boxplots por promoção
   - Barplots por dimensão
   - Visualizações de distribuição


DEPENDÊNCIAS
================================================================================
Bibliotecas Python necessárias:
- pandas: manipulação de dados
- numpy: operações numéricas
- scipy.stats: testes estatísticos (ANOVA, Chi-square, t-test)
- sklearn: regressão linear e normalização
- matplotlib: visualizações
- seaborn: gráficos estatísticos
- statsmodels: teste post-hoc de Tukey

Instalação:
pip install pandas numpy scipy scikit-learn matplotlib seaborn statsmodels


CONCLUSÃO
================================================================================
Este projeto demonstra:
✓ Tratamento de dados faltantes com síntese estatística
✓ Aplicação de testes ANOVA e Tukey para comparação de grupos
✓ Importância da normalização em machine learning
✓ Análise completa de teste A/B de marketing
✓ Visualização e interpretação de resultados estatísticos

Próximos passos sugeridos:
1. Análise por segmento (MarketSize, AgeOfStore)
2. Modelos preditivos para vendas futuras
3. Análise de série temporal por week
4. Otimização de promoção por mercado

================================================================================
                            Fim do Readme
================================================================================
