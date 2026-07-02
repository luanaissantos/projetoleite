# Predição da produtividade leiteira municipal com aprendizado de máquina

Este repositório contém um notebook em Python/Google Colab para preparar dados públicos do SIDRA/IBGE e comparar modelos de aprendizado de máquina supervisionado na previsão da produtividade leiteira dos municípios brasileiros no ano seguinte.

A produtividade leiteira é definida como a quantidade anual de leite produzida, em litros, dividida pelo número de vacas ordenhadas no município. A tarefa de modelagem utiliza informações do município no ano `t` para estimar a produtividade no ano `t + 1`.

## Objetivo

O objetivo principal é avaliar se variáveis históricas, pecuárias e agrícolas melhoram a previsão da produtividade leiteira municipal futura em relação a uma referência simples de persistência, na qual a produtividade do próximo ano é estimada como igual à produtividade observada no ano atual.

Além de comparar algoritmos, o notebook investiga três conjuntos progressivos de informações:

1. **Histórico**: produtividade atual e indicadores temporais derivados.
2. **Pecuário**: histórico acrescido de produção de leite, vacas ordenhadas, efetivo bovino e indicadores do rebanho.
3. **Integrado**: histórico e pecuário acrescidos de variáveis municipais de milho e soja.

As variáveis agrícolas são tratadas como indicadores do contexto produtivo municipal. Elas não devem ser interpretadas como medida direta da alimentação do rebanho, pois os dados agregados do SIDRA/IBGE não indicam a destinação efetiva da produção agrícola.

## Dados utilizados

O notebook utiliza arquivos JSON previamente baixados do SIDRA/IBGE e compactados no arquivo:

```text
jsons_ibge_baixados.zip
```

As bases consideradas incluem informações anuais de 2000 a 2024 sobre:

- produção de leite;
- número de vacas ordenhadas;
- efetivo do rebanho bovino;
- área, produção e rendimento médio de milho;
- área, produção e rendimento médio de soja.

O arquivo compactado deve estar na mesma pasta do notebook ou em alguma pasta acessível no Google Drive. O notebook procura automaticamente o arquivo em `/content`, no diretório corrente e no Google Drive montado.

## Modelos comparados

O notebook compara cinco abordagens:

- **Persistência**: modelo de referência baseado na repetição da produtividade atual;
- **Regressão Ridge**: modelo linear regularizado;
- **Random Forest**: ensemble de árvores por bagging;
- **XGBoost**: gradient boosting para dados tabulares;
- **Multilayer Perceptron (MLP)**: rede neural feedforward para regressão.

Os hiperparâmetros dos modelos são selecionados com base no conjunto de validação. O conjunto de teste é mantido separado até a avaliação final.

## Divisão temporal

A divisão dos dados é cronológica, evitando o uso de informações futuras durante o treinamento:

| Conjunto | Anos das características | Anos-alvo previstos |
|---|---:|---:|
| Treinamento | 2000–2013 | 2001–2014 |
| Validação | 2014–2018 | 2015–2019 |
| Teste | 2019–2023 | 2020–2024 |

Após a seleção dos hiperparâmetros, os modelos finais são reajustados com treino + validação e avaliados uma única vez no conjunto de teste.

## Métricas de avaliação

O desempenho é avaliado por:

- **MAE**: erro absoluto médio, em litros por vaca ao ano;
- **RMSE**: raiz do erro quadrático médio, em litros por vaca ao ano;
- **R²**: coeficiente de determinação;
- **melhoria percentual do RMSE** em relação ao modelo de persistência.

Também são geradas análises dos erros por ano-alvo, região brasileira e faixa inicial de produtividade.

## Estrutura sugerida do repositório

```text
.
├── README.md
├── produtividade_leiteira_ml_colab.ipynb
├── jsons_ibge_baixados.zip                  # não versionar se o arquivo for grande
└── resultados_produtividade_leiteira_ml/    # gerado após a execução
    ├── figuras/
    ├── modelos/
    ├── resultados_consolidados.xlsx
    ├── resultados_validacao.csv
    ├── resultados_teste.csv
    ├── predicoes_teste.csv
    ├── importancia_variaveis.csv
    └── configuracao_experimento.json
```

Arquivos de dados, modelos treinados e resultados pesados podem ser mantidos fora do Git ou adicionados ao `.gitignore`, dependendo do tamanho e das regras de compartilhamento do trabalho.

## Como executar no Google Colab

1. Faça upload do notebook `produtividade_leiteira_ml_colab.ipynb` para o Google Colab.
2. Coloque o arquivo `jsons_ibge_baixados.zip` na mesma pasta do notebook ou em uma pasta do Google Drive.
3. Abra o notebook no Colab.
4. Execute todas as células em ordem.
5. Ao final, verifique a pasta `resultados_produtividade_leiteira_ml/`, criada automaticamente ao lado do arquivo de dados.

O notebook instala apenas pacotes ausentes no ambiente e utiliza bibliotecas como `pandas`, `numpy`, `scikit-learn`, `xgboost`, `tensorflow`, `matplotlib`, `joblib`, `openpyxl` e `pyarrow`.

## Principais saídas geradas

Durante a execução, o notebook salva tabelas, figuras, modelos e configurações do experimento. Entre os principais arquivos estão:

- `resumo_qualidade_dados.csv`: verificações iniciais da base;
- `resumo_amostras.csv`: número de observações e municípios por período;
- `resumo_produtividade_alvo.csv`: estatísticas da variável-alvo;
- `resultados_validacao.csv`: desempenho na etapa de seleção de hiperparâmetros;
- `resultados_teste.csv`: desempenho final no conjunto de teste;
- `predicoes_teste.csv`: valores observados e previstos para o teste;
- `ganhos_por_conjunto_de_informacao.csv`: comparação dos conjuntos histórico, pecuário e integrado;
- `metricas_por_ano.csv`, `metricas_por_regiao.csv` e `metricas_por_faixa.csv`: análises estratificadas;
- `importancia_variaveis.csv`: interpretação das variáveis por modelo;
- `resultados_consolidados.xlsx`: planilha com as principais tabelas;
- `base_modelagem.parquet`: base final usada na modelagem;
- `configuracao_experimento.json`: parâmetros, versões de bibliotecas e configurações utilizadas;
- `resultados_produtividade_leiteira_ml.zip`: pacote compactado com os resultados.

As principais figuras são salvas na subpasta `figuras/`, incluindo curvas de treinamento/validação, comparação de RMSE, gráfico observado versus predito e gráficos de importância de variáveis.

## Reprodutibilidade

O experimento utiliza semente fixa (`42`) e divisão temporal explícita. As etapas de imputação, padronização, seleção de hiperparâmetros e avaliação foram organizadas para evitar vazamento de informação do teste para o treinamento.

Mesmo assim, pequenas diferenças podem ocorrer entre execuções por causa de versões de bibliotecas, aceleração por GPU/CPU e rotinas internas não determinísticas de alguns algoritmos.

## Observações metodológicas

- A análise é preditiva, não causal.
- Os dados de milho e soja representam o contexto agrícola municipal, não o consumo efetivo desses grãos pelo rebanho leiteiro local.
- A produtividade é calculada apenas quando o número de vacas ordenhadas é maior que zero.
- Registros com símbolos especiais do SIDRA são tratados distinguindo zeros reais de valores ausentes ou não informados.
- O desempenho final deve ser interpretado principalmente em comparação com o modelo de persistência.

## Autor

Projeto desenvolvido como trabalho de aprendizado de máquina supervisionado, com foco na previsão da produtividade leiteira municipal brasileira a partir de dados públicos agropecuários.
