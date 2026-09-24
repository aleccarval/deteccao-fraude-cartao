Detecção de Fraude em Transações de Cartão de Crédito com Machine Learning
Projeto desenvolvido com o objetivo de construir, comparar e explicar modelos preditivos para identificar transações fraudulentas num fluxo altamente desbalanceado de pagamentos com cartão de crédito.


1. O Problema e o Impacto do Desbalanceamento
Em cenários reais de análise de transações financeiras e detecção de invasões/anomalias, os eventos maliciosos são extremamente raros. No conjunto de dados utilizado, apenas 0,17% das operações representam fraude, enquanto 99,83% são normais.
Por que a Acurácia Engana?
Se um modelo simplesmente prever que todas as transações são legítimas (classe 0), ele alcançará 99,83% de acurácia. No entanto, terá identificado zero fraudes, tornando-se completamente inútil para a segurança da operação.

Por essa razão, a nossa avaliação foca-se nas métricas da Classe 1 (Fraude):
Recall (Revocação): Mede a capacidade de interceptar fraudes reais. É a métrica mais importante para evitar prejuízos diretos e estornos bancários (chargeback).
Precisão: Proporção de alertas disparados que eram, de fato, fraudes. Uma precisão muito baixa satura as equipes de segurança e gera fricção desnecessária com clientes idôneos.
F1-Score: Média harmônica entre Precisão e Recall, servindo de critério de equilíbrio.
PR-AUC (Área sob a Curva Precision-Recall): Muito mais informativa do que a curva ROC para dados com forte assimetria.


2. Preparação e Pré-Processamento dos Dados
Transformação Logarítmica: A variável Amount possui assimetria acentuada; foi aplicada a transformação Amount_log = np.log1p(Amount) para estabilizar a escala. A coluna Time foi descartada.
Divisão Estratificada (Train/Test Split): Separação em 80% para treino e 20% para teste utilizando stratify=y, garantindo que o conjunto de teste preserva exatamente a proporção real de 0,17% de fraudes do ambiente produtivo.
Padronização: Aplicação do StandardScaler ajustado (fit) exclusivamente no conjunto de treino e replicado (transform) no teste para evitar fuga de informação (data leakage).
Tratamento do Desbalanceamento no Treino:
Aplicação de SMOTE (Oversampling) e RandomUnderSampler (Undersampling) restritos à partição de treino.
Utilização do parâmetro nativo scale_pos_weight no XGBoost para penalizar erros na classe minoritária.


3. Comparação de Modelos e Otimização de Limiares (Thresholds)
Os classificadores utilizam, por padrão, o limiar de decisão 0.50. Em problemas com classes raras, calibrar esse corte permite encontrar o melhor compromisso operacional.
Resultados Obtidos (Classe 1 - Fraude)
Modelo
Limiar Padrão
Recall (0.50)
Precisão (0.50)
F1 (0.50)
Limiar Ajustado
Recall Ajustado
Precisão Ajustada
F1 Ajustada
Regressão Logística (Baseline)
0.50
89.8%
8.2%
0.150
0.85
84.7%
76.5%
0.804
Random Forest
0.50
79.6%
88.6%
0.838
0.35
83.7%
85.4%
0.845
XGBoost (Campeão)
0.50
82.7%
89.0%
0.857
0.30
87.8%
88.7%
0.882

Justificação do Limiar Escolhido
Para o modelo vencedor (XGBoost), o limiar foi reduzido de 0.50 para 0.30. Esta decisão permitiu elevar o Recall de 82,7% para 87,8% com impacto residual mínimo na precisão, garantindo que mais incidentes fraudulentos sejam capturados sem inflacionar a taxa de falsos alarmes.


4. Explicabilidade do Modelo (SHAP)
Para compreender os fatores determinantes por trás das decisões do classificador, foi empregue a biblioteca SHAP com o TreeExplainer sobre o modelo XGBoost:

Principais Preditores: As componentes principais V14, V17, V12 e V10 apresentaram o maior impacto na pontuação de risco.
Sentido da Decisão: Valores negativos atípicos nessas variáveis aumentam significativamente os valores de SHAP, empurrando o modelo para a classificação de fraude.
Importância do Valor: A variável Amount_log teve peso secundário comparativamente aos desvios de padrão comportamental representados pelos componentes do PCA, demonstrando que operações fraudulentas ocorrem frequentemente em quantias de baixo a médio valor para contornar verificações simplistas de limite.


5. Diferenciais e Alterações em Relação à Abordagem Original
Eliminação de Data Leakage: Todo o rebalanceamento sintético (SMOTE/Undersampling) foi executado estritamente sobre a base de treino. O conjunto de teste manteve a sua assimetria original.
Calibração de Limiar Automatizada: Implementação de função para varredura contínua da curva Precisão-Recall identificando o ponto de máximo F1-Score.
Interpretabilidade Local e Global: Integração de gráficos Summary Plot e decomposição individual Waterfall via SHAP.


6. Como Executar o Projeto
Clone o repositório do seu projeto no GitHub.
Instale as dependências:
pip install pandas numpy scikit-learn xgboost imbalanced-learn matplotlib shap
Abra e execute o notebook detect.ipynb de ponta a ponta.

(O dataset é importado dinamicamente via URL durante a execução).

