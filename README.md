# Deteccao-de-Fraude-em-Cartoes-de-Credito
Desafio Prático: Detecção de Fraude em Transações Reais com Machine Learning e Explicabilidade[cite: 1]

# Projeto Final: Detecção de Fraude em Transações de Cartão de Crédito com Machine Learning[cite: 1, 2]

## 🎯 Objetivo do Projeto
Desenvolver, comparar e explicar modelos de Machine Learning capazes de identificar transações fraudulentas em fluxos de pagamentos de cartões de crédito altamente desbalanceados, onde fraudes representam apenas 0,17% das ocorrências e a acurácia convencional é enganosa[cite: 1, 2].

### ⚠️ O Problema e o Impacto do Desbalanceamento
Em ambientes de pagamento digital e cibersegurança, eventos maliciosos são extremamente raros[cite: 2]:
- **99,83%** das transações são legítimas (classe `0`)[cite: 1, 2].
- **0,17%** das transações são fraudulentas (classe `1`)[cite: 1, 2].

#### Por que a Acurácia Engana?
Um modelo ingênuo que classifique todas as transações como normais alcançará 99,83% de acurácia, mas deixará passar 100% das fraudes, gerando prejuízos financeiros severos (*chargebacks*) e risco operacional[cite: 1, 2].

#### Métricas Avaliadas (Foco na Classe 1 - Fraude)
- `Recall (Revocação)`: Capacidade do modelo de interceptar o máximo de fraudes reais (minimiza falsos negativos)[cite: 1, 2].
- `Precisão`: Confiabilidade dos alertas disparados (minimiza bloqueios indevidos a clientes idôneos)[cite: 1, 2].
- `F1-Score`: Média harmônica entre Precisão e Recall, servindo de critério de equilíbrio[cite: 1, 2].
- `PR-AUC`: Área sob a curva Precision-Recall, métrica ideal para conjuntos com desbalanceamento severo[cite: 1, 2].

### 🛠️ Pipeline de Preparação dos Dados
A base de dados conta com atributos anonimizados via PCA (`V1` a `V28`), além de `Time` e `Amount`[cite: 1, 2]:
- `Amount_log`: Aplicação de transformação logarítmica (`np.log1p`) para estabilizar a escala e mitigar assimetria[cite: 1, 2].
- `Time`: Remoção da coluna de tempo bruto para priorizar componentes de padrão de consumo[cite: 1, 2].
- `train_test_split`: Divisão estratificada (80% treino / 20% teste) com `stratify=y` para manter a proporção real de 0,17% no conjunto de teste[cite: 1, 2].
- `StandardScaler`: Padronização ajustada (*fit*) estritamente no treino e replicada (*transform*) no teste, evitando vazamento de dados (*data leakage*)[cite: 1, 2].
- `Tratamento no Treino`: Aplicação de técnicas de amostragem (SMOTE e RandomUnderSampler) restritas à partição de treino[cite: 1, 2].

### 📊 Comparativo de Modelos e Otimização de Limiares (Thresholds)
Por padrão, classificadores utilizam o limiar de decisão `0.50`[cite: 1, 2]. Em problemas com classes raras, calibrar esse corte permite encontrar o melhor compromisso operacional[cite: 1, 2]:

| Modelo | Limiar Padrão | Recall (0.50) | Precisão (0.50) | F1 (0.50) | Limiar Ajustado | Recall Ajustado | Precisão Ajustada | F1 Ajustado |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Regressão Logística (Baseline)** | 0.50 | 89.8% | 8.2% | 0.150 | 0.85 | 84.7% | 76.5% | 0.804 |
| **Random Forest** | 0.50 | 79.6% | 88.6% | 0.838 | 0.35 | 83.7% | 85.4% | 0.845 |
| **XGBoost (Campeão)** | 0.50 | 82.7% | 89.0% | 0.857 | 0.30 | 87.8% | 88.7% | 0.882 |

#### Justificativa do Limiar Escolhido
Para o modelo vencedor (**XGBoost**), o limiar foi reduzido de `0.50` para `0.30`[cite: 2]. Essa decisão permitiu elevar o Recall para **87,8%** com impacto residual mínimo na precisão (**88,7%**), assegurando a interceptação da maioria das fraudes sem saturar as equipes de análise com falsos positivos[cite: 2].

### 🔍 Explicabilidade do Modelo (SHAP)
Para interpretar os fatores determinantes por trás das decisões do algoritmo, foi aplicada a biblioteca **SHAP** com o explicador `TreeExplainer` sobre o XGBoost[cite: 1, 2]:
1. **Principais Preditores:** As componentes principais `V14`, `V17`, `V12` e `V10` apresentaram o maior impacto no score de risco de fraude[cite: 2].
2. **Sentido da Decisão:** Valores negativos atípicos nessas variáveis aumentam significativamente os valores SHAP, empurrando a probabilidade para a classe de fraude[cite: 2].
3. **Importância do Valor:** A variável `Amount_log` teve peso secundário comparada às componentes de PCA, confirmando que invasores costumam fragmentar transações em valores medianos/baixos para driblar regras estáticas[cite: 2].

### 📋 Diferenciais em Relação à Abordagem Inicial
- **Eliminação de Data Leakage:** Amostragem sintética (SMOTE/Undersampling) aplicada exclusivamente sobre os dados de treino, mantendo a integridade estatística da base de teste[cite: 1, 2].
- **Calibração Automatizada:** Função de busca contínua na curva Precision-Recall para encontrar o limiar ótimo via F1-Score[cite: 1, 2].
- **Explicabilidade Prática:** Decomposição individual (*Waterfall*) e global (*Summary Plot*) com SHAP para auditoria e suporte a analistas de segurança[cite: 1, 2].
