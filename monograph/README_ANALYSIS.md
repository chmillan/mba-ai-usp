
# Roteiro de análise e modelagem — Tráfego IX.br São Paulo

Este documento acompanha a preparação, a exploração e a modelagem da série temporal de tráfego do IX.br São Paulo para o TCC **Previsão de Tráfego de Internet em Pontos de Troca de Tráfego: Modelagem de Séries Temporais com Ênfase nos Picos de Demanda**.

**Pergunta de pesquisa:** em que medida modelos de séries temporais conseguem antecipar o **máximo diário** e o **P95 diário** de tráfego para o dia seguinte?

O objetivo é avaliar o desempenho preditivo, sobretudo em dias de demanda elevada e nas subestimações. O trabalho não pretende determinar diretamente a capacidade necessária da infraestrutura.

## Visão geral

| Notebook | Papel | Situação |
|---|---|---|
| `00` — Preparação | Reconstruir e disponibilizar a série original de cinco minutos | Implementado |
| `01` — Nível de tráfego | Explorar evolução, sazonalidade e diferenças entre dias | Análises principais realizadas |
| `02` — Picos | Caracterizar máximos diários, P95 e relação pico/média | Análises principais realizadas |
| `03` — Frequência | Investigar periodicidades por análise espectral | Análises realizadas |
| `03a` — Completude | Avaliar a sensibilidade dos indicadores a medições ausentes | Análises realizadas |
| `04` — Alvos | Consolidar uma tabela diária auditável para previsão | Em implementação/revisão |
| `05` — Baselines | Comparar previsões ingênuas e sazonais | Planejado |
| `06` — Modelos | Treinar e comparar modelos supervisionados | Planejado |

Os estados acima refletem o andamento conhecido do projeto, não uma auditoria de execução de todas as células.

## 00 — Preparação dos dados. Como obter uma série temporal consistente?

**Objetivo:** reconstruir a série temporal a partir do XML, combinar tráfego de entrada e saída, converter para Gbps e produzir arquivos processados.

**Saídas existentes:**

- `serie_temporal_filtrada_gbps.csv` — série processada de cinco minutos.
- `resumo_anual_filtrado_gbps.csv` — resumo anual.

**Atenção:** a completude anual calculada nessa etapa não substitui a auditoria de completude **diária** contra as 288 posições esperadas. A referência de fuso horário dos timestamps ainda precisa ser confirmada.

## 01 — Análise exploratória do nível de tráfego. Como o tráfego evolui e varia?

**Objetivo:** caracterizar a evolução histórica do nível de tráfego e padrões temporais.

**Realizado:** evolução e suavização da média diária, comparações intra-anuais, padrões por dia da semana e diferenças entre dias úteis e fins de semana.

**Cuidados de interpretação:** a faixa `média ± 1,96 × desvio/√n` utilizada na exploração não é um intervalo de previsão e não incorpora automaticamente a dependência temporal. Agregações de medições individuais por dia da semana não são equivalentes, necessariamente, a agregações com igual peso por dia.

## 02 — Análise dos picos. Quais foram os picos e como evoluíram?

**Objetivo:** descrever a magnitude, a evolução e os padrões dos picos de tráfego.

**Realizado:** série de máximos diários, P95, média e contagem de observações; resumos anuais; relação entre máximo, P95 e média; padrões semanais e perfis intradiários.

**Limitação para modelagem:** a série exploratória `picos_diarios` admite dias com pelo menos uma observação. Para os alvos preditivos, será necessário aplicar o critério de completude diária e preservar o calendário, tarefa do notebook 04.

## 03 — Análise de frequência. Quais periodicidades existem na série?

**Objetivo:** identificar escalas temporais relevantes e formular hipóteses para futuros atributos preditivos.

**Resultados principais:**

- Periodicidade diária de aproximadamente **24 horas**, dominante.
- Componente de **12 horas** e harmônicas menores de **8 e 6 horas**.
- Sinal semanal como candidato relevante na série de máximos diários.

Na análise horária, foram utilizados critérios próprios de completude e interpolação limitada para fins exploratórios. **Esses valores interpolados não devem ser usados automaticamente como alvos de previsão.** Periodicidade identificada não implica ganho preditivo comprovado.

## 03a — Sensibilidade à incompletude diária. Como as lacunas afetam os indicadores?

**Objetivo:** investigar quanto a ausência de medições de cinco minutos altera média, P95, P99 e máximo diário, sem modelos de previsão.

**Procedimento:** partir de dias completos (288 observações), remover artificialmente 14 ou 28 medições e comparar os indicadores recalculados com os valores do dia completo. Foram estudadas remoções aleatórias, blocos de baixa demanda, blocos de alta demanda e máscaras empíricas compostas.

**Conclusão metodológica:** adotar **provisoriamente 90% de completude diária** (pelo menos 260 de 288 observações). Os erros médios nos cenários aleatórios e empíricos foram relativamente pequenos, mas lacunas concentradas em períodos de alta demanda podem causar subestimações relevantes. O critério não garante confiabilidade dos extremos e não permite conhecer os valores verdadeiros de dias originalmente incompletos.

**Importante:** o erro desse experimento é **indicador incompleto − indicador do dia completo**, não erro de um modelo preditivo.

## 04 — Consolidação dos alvos de previsão. Quais valores de máximo e P95 diário são adequados para treinamento e avaliação, considerando a qualidade dos dados?

**Objetivo:** Construir e validar a série diária de máximo e P95 de tráfego, aplicando os critérios de qualidade definidos nas análises anteriores e preparando o conjunto de dados para a modelagem preditiva.

**Escopo mínimo:**

- [ ] Construir uma linha por dia em 2023–2024, mantendo o calendário completo.
- [ ] Calcular **máximo** e **P95** (alvos principais), **P99** e **média** (complementares), diretamente das observações de cinco minutos, sem interpolação.
- [ ] Usar quantis lineares (`numpy.quantile(method="linear")`).
- [ ] Registrar contagem, completude, lacunas, duplicatas e diagnósticos dos extremos.
- [ ] Distinguir **indicador observado** de **alvo elegível** (mínimo de 260 observações válidas).
- [ ] Documentar empates, limites do dia e referência temporal; não presumir fuso horário.
- [ ] Validar disponibilidade por mês/ano e coerência dos indicadores com poucas tabelas.
- [ ] Exportar o dataset de forma que o notebook 05 possa carregá-lo independentemente.

A investigação detalhada do **horário do máximo** é secundária e não deve bloquear os modelos de magnitude. Não é necessário refazer gráficos históricos, análise espectral ou simulações de incompletude.

## 05 — Baselines de previsão (planejado). Quão bem conseguimos prever com métodos simples?

**Objetivo:** estabelecer referências simples para prever o máximo e o P95 do **dia seguinte**.

Comparar, nas mesmas datas-alvo, o valor do dia anterior, o do mesmo dia da semana anterior e estatísticas móveis calculadas exclusivamente com informações disponíveis na origem da previsão. As defasagens devem respeitar o **calendário real**, inclusive dias sem alvo elegível.

## 06 — Modelos supervisionados (planejado) Modelos mais elaborados melhoram as previsões?

**Objetivo:** investigar se modelos com defasagens, estatísticas históricas e atributos de calendário superam os baselines.

Começar por modelos parcimoniosos, considerando o número limitado de exemplos diários. Comparar posteriormente, se justificável, diferentes extensões do histórico de treinamento e atributos derivados das periodicidades observadas.

## Protocolo temporal e avaliação

| Período | Uso previsto |
|---|---|
| **2023** | Desenvolvimento e investigação inicial |
| **2024** | Validação temporal e seleção de estratégias |
| **2025** | Avaliação final da estratégia selecionada |
| **Janeiro/2026** | Análise complementar, se pertinente |

Dados anteriores a 2023 podem ser investigados em um experimento adicional de ampliação do treinamento. Houve contato exploratório anterior com dados de 2025; isso deve ser declarado como limitação da avaliação final.

**Métricas:** MAE, RMSE, frequência e magnitude das subestimações e desempenho em dias de demanda elevada. Limiares estatísticos de alta demanda devem ser definidos com dados de treinamento, sem utilizar o período de teste.

**Tamanho amostral:** uma previsão diária gera no máximo 365 exemplos de treinamento em 2023, antes das exclusões por qualidade e das defasagens. A série de cinco minutos continua disponível para a construção de atributos históricos, mas não aumenta o número de datas-alvo independentes.

## Limites de aplicação operacional

O problema **preditivo** é estimar o máximo e o P95 de amanhã e quantificar os erros. O problema **operacional** é determinar margens de capacidade aceitáveis para a infraestrutura. Este segundo problema depende de dados e requisitos não disponíveis no estudo, como capacidade instalada, redundância, margens de segurança e tolerância ao risco.

Assim, o TCC **não definirá tolerâncias operacionais arbitrárias nem capacidade necessária**. A discussão final deverá explicar como o desempenho estatístico — especialmente as subestimações em dias de alta demanda — pode informar, mas não substituir, avaliações de engenharia.

## Pendências metodológicas prioritárias

- [ ] Confirmar a referência temporal/fuso dos timestamps e o significado dos limites de cada dia.
- [ ] Finalizar e validar o dataset diário do notebook 04.
- [ ] Documentar o tratamento operacional de duplicatas e empates.
- [ ] Verificar as datas elegíveis e a continuidade do calendário antes de construir defasagens.
- [ ] Decidir se vale comparar treinamento restrito a 2023 com histórico mais longo, após os primeiros baselines.

> **Princípio de organização:** notebooks 00–03a documentam preparação, exploração e qualidade; o 04 consolida os alvos; os notebooks seguintes avaliam a capacidade de prevê-los. Evitar duplicar análises já concluídas.




#########


# Possibilidade de pesquisa futura — Previsão antecipada de anomalias no tráfego de Internet

**Status:** ideia exploratória para investigação futura.  
**Escopo:** não faz parte dos objetivos principais do TCC atual.

## 1. Motivação

O TCC atual investiga modelos de séries temporais para prever o máximo diário e o P95 diário de tráfego no IX.br São Paulo, com atenção especial aos períodos de maior demanda e aos erros de subestimação.

Uma possibilidade de extensão é utilizar a resolução original dos dados, com medições a cada cinco minutos, para investigar a **previsão antecipada de comportamentos anômalos**.

Em vez de prever o tráfego agregado do dia seguinte, a proposta seria analisar o comportamento recente da série e estimar a probabilidade de ocorrência de uma anomalia em um horizonte curto.

**Pergunta de pesquisa:**

> É possível antecipar comportamentos anômalos no tráfego de Internet utilizando padrões observados na última hora de medições?

O interesse principal seria identificar sinais que antecedem alterações incomuns no tráfego, permitindo investigar a viabilidade de alertas antecipados.

## 2. Detecção versus previsão de anomalias

É importante distinguir dois problemas.

### Detecção de anomalias

Identificar se uma observação já registrada apresenta comportamento incomum em relação ao padrão esperado.

Exemplo:

- O tráfego normalmente varia entre 800 e 850 Gbps naquele horário.
- Uma nova medição registra 1.100 Gbps.
- O sistema identifica a observação como potencialmente anômala.

### Previsão antecipada de anomalias

Utilizar observações anteriores para estimar a possibilidade de ocorrência de uma anomalia futura.

Exemplo:

- O sistema analisa as últimas 12 medições, correspondentes a uma hora.
- Identifica características como aumento de volatilidade, aceleração ou mudança de tendência.
- Estima o risco de uma anomalia ocorrer nos próximos 15 minutos.

**Diferença fundamental:** detectar uma anomalia depois que ela ocorreu não significa conseguir antecipá-la. Algumas anomalias podem ocorrer sem sinais precursores observáveis.

## 3. Formulação inicial do problema

### Dados de entrada

Série temporal de tráfego do IX.br São Paulo, com resolução de cinco minutos.

Uma configuração experimental inicial poderia utilizar:

- Janela histórica: 12 observações (60 minutos).
- Horizonte de previsão: próximas 3 observações (15 minutos).
- Variável de entrada principal: tráfego em Gbps.
- Saída: probabilidade estimada de ocorrência de uma anomalia no horizonte futuro.

Esses valores são apenas uma configuração inicial para experimentação. Outras janelas e horizontes poderiam ser comparados.

### Exemplo conceitual

| Elemento | Exemplo |
|---|---|
| Histórico disponível | 10h00–10h55 |
| Quantidade de medições | 12 |
| Origem da previsão | 10h55 |
| Horizonte futuro | 11h00–11h10 |
| Saída do modelo | Probabilidade de anomalia nesse horizonte |

A definição exata dos intervalos deverá evitar sobreposição indevida entre entrada e período previsto.

## 4. O que caracteriza uma anomalia?

Esta é uma das principais decisões metodológicas.

**Um pico de tráfego não é necessariamente uma anomalia.**

O tráfego pode apresentar valores elevados durante períodos de maior utilização sem que exista qualquer comportamento irregular.

Por outro lado, quedas abruptas, mudanças inesperadas de tendência ou desvios expressivos em relação ao padrão histórico podem representar eventos relevantes.

Possíveis definições a investigar:

1. **Desvio em relação ao comportamento esperado:** observações muito diferentes do padrão previsto para determinado horário.
2. **Mudança abrupta:** crescimento ou queda incomum em um intervalo curto.
3. **Anomalia contextual:** valor que seria normal em determinado período, mas incomum em outro.
4. **Evento de interesse operacional:** comportamento associado a incidentes reais, caso existam registros externos que permitam essa identificação.

A escolha deverá considerar o objetivo do estudo e a disponibilidade de dados de referência.

Sem registros de incidentes, o modelo poderá estudar **anomalias estatísticas**, mas não será possível afirmar automaticamente que elas representam falhas de infraestrutura.

## 5. Abordagens de modelagem

### 5.1. Previsão do tráfego seguida de detecção

Uma abordagem inicial seria prever as próximas observações de tráfego e comparar os valores previstos com os efetivamente observados.

Modelos possíveis:

- ARIMA e variantes.
- Modelos de regressão com defasagens.
- Modelos de Machine Learning supervisionado.
- LSTM, GRU ou redes convolucionais temporais.

Desvios elevados entre previsão e observação podem indicar anomalias.

**Limitação:** essa estratégia identifica desvios quando os valores reais chegam. Não constitui, por si só, uma previsão antecipada da ocorrência de anomalias.

### 5.2. Classificação antecipada de anomalias

Outra abordagem seria transformar o problema em classificação supervisionada.

A partir de uma janela histórica, o modelo estimaria se ocorrerá pelo menos uma anomalia no horizonte futuro.

Modelos candidatos:

- Regressão logística.
- Random Forest.
- XGBoost.
- Redes neurais sequenciais, como LSTM, GRU e TCN.

A saída poderia ser uma probabilidade estimada de ocorrência do evento.

Essa abordagem exige uma definição consistente de anomalia e a construção de rótulos para os períodos futuros.

### 5.3. Previsão probabilística

Uma terceira possibilidade seria estimar a distribuição ou os intervalos de valores futuros de tráfego.

Exemplos:

- Regressão quantílica.
- Modelos probabilísticos de séries temporais.
- Redes neurais com saídas probabilísticas.

O objetivo seria estimar a incerteza das previsões e, dependendo da definição adotada, a probabilidade de ultrapassar determinados limites.

Essa estratégia pode ser especialmente útil quando a intensidade e a incerteza do tráfego futuro são relevantes.

## 6. Atributos potencialmente informativos

Além das próprias observações de tráfego, seria possível investigar atributos derivados da janela histórica.

| Categoria | Exemplos |
|---|---|
| Nível recente | Média, mediana, máximo e mínimo |
| Variabilidade | Desvio-padrão e amplitude |
| Tendência | Inclinação e diferenças sucessivas |
| Mudança de comportamento | Aceleração e variações abruptas |
| Contexto temporal | Hora do dia e dia da semana |
| Comparação histórica | Desvio em relação ao padrão típico daquele horário |

Os atributos devem ser calculados exclusivamente com informações disponíveis até a origem da previsão.

A utilidade de cada atributo precisaria ser demonstrada experimentalmente.

## 7. Desafios metodológicos

### 7.1. Raridade dos eventos

Anomalias relevantes podem ser pouco frequentes.

Um classificador que sempre prevê ausência de anomalia pode apresentar elevada acurácia sem ser útil.

Será necessário considerar o desbalanceamento das classes.

### 7.2. Dependência temporal

Janelas consecutivas compartilham muitas observações.

Embora seja possível construir milhares de exemplos, eles não representam amostras independentes.

A separação entre treinamento, validação e teste deve respeitar a ordem temporal e evitar vazamento de informações.

### 7.3. Qualidade das medições

Medições ausentes, duplicadas ou inconsistentes podem produzir falsas anomalias.

Será importante distinguir problemas de coleta de mudanças reais no comportamento do tráfego.

### 7.4. Ausência de sinais antecedentes

Nem toda anomalia é previsível.

Eventos abruptos podem não apresentar padrões identificáveis na hora anterior.

A pesquisa deverá investigar empiricamente se existe capacidade preditiva além de referências simples.

### 7.5. Definição e qualidade dos rótulos

Os resultados dependem diretamente do critério utilizado para identificar anomalias.

Se os rótulos forem produzidos por regras estatísticas, a avaliação medirá a capacidade de antecipar eventos definidos por essas regras, e não necessariamente incidentes reais.

## 8. Estratégia experimental possível

Uma investigação futura poderia seguir as seguintes etapas:

1. Definir o tipo de anomalia a ser estudado.
2. Auditar a qualidade da série de cinco minutos.
3. Construir rótulos de eventos anômalos sem utilizar informações futuras indevidamente.
4. Criar janelas históricas e horizontes de previsão.
5. Estabelecer baselines simples.
6. Treinar classificadores supervisionados.
7. Comparar modelos de maior complexidade, caso haja justificativa.
8. Avaliar desempenho em períodos futuros não utilizados no desenvolvimento.
9. Investigar os eventos corretamente antecipados, os não detectados e os falsos alarmes.

A comparação deverá utilizar os mesmos eventos e horizontes de previsão para todos os modelos.

## 9. Avaliação dos resultados

A acurácia isolada não seria uma métrica suficiente, especialmente diante de eventos raros.

Métricas relevantes incluem:

- **Precisão:** proporção de alertas emitidos que correspondem a eventos.
- **Recall:** proporção de eventos efetivamente antecipados.
- **F1-score:** equilíbrio entre precisão e recall.
- **PR-AUC:** desempenho da classificação em diferentes limiares, particularmente relevante em problemas desbalanceados.
- **Taxa de falsos alarmes:** frequência de alertas sem ocorrência do evento.
- **Antecedência do alerta:** tempo entre a previsão e a ocorrência da anomalia.

Também seria importante distinguir avaliações por janela de previsão e por evento, evitando contar repetidamente a mesma anomalia como vários acertos independentes.

O desempenho deverá ser comparado com baselines e acompanhado de análise de incerteza, sempre que viável.

## 10. Relação com o TCC atual

O projeto atual trabalha principalmente com indicadores agregados por dia:

- Máximo diário.
- P95 diário.
- Previsão para o dia seguinte.

A proposta de previsão de anomalias utilizaria outra escala temporal e outro objetivo:

| Aspecto | TCC atual | Pesquisa futura |
|---|---|---|
| Unidade temporal | Dia | Cinco minutos |
| Horizonte | Dia seguinte | Minutos à frente |
| Alvo | Máximo e P95 diário | Ocorrência de anomalia |
| Tipo de problema | Regressão | Classificação ou previsão probabilística |
| Principal preocupação | Erros nos períodos de alta demanda | Eventos não antecipados e falsos alarmes |
| Aplicação potencial | Apoio à análise de demanda | Apoio a sistemas de alerta antecipado |

A resolução de cinco minutos permitiria construir mais exemplos temporais do que a agregação diária, mas a forte dependência entre janelas e a possível raridade dos eventos continuariam sendo desafios.

## 11. Possíveis contribuições científicas

A investigação poderia contribuir para responder:

- Existem padrões mensuráveis que antecedem anomalias de tráfego?
- Qual é o horizonte máximo em que essas anomalias podem ser antecipadas com desempenho útil?
- Atributos estatísticos simples são suficientes ou modelos sequenciais oferecem ganhos?
- Como a qualidade e a completude dos dados afetam a antecipação de eventos?
- Qual é a relação entre antecedência, eventos não detectados e falsos alarmes?

Essas questões permitiriam avaliar não apenas a capacidade de identificar comportamentos incomuns, mas também a existência de sinais antecipatórios na série temporal.

## 12. Encaminhamento

**Decisão atual:** manter a previsão antecipada de anomalias como possibilidade de pesquisa futura, sem incorporá-la aos objetivos ou às implementações do TCC em andamento.

A prioridade permanece na construção e avaliação de modelos para previsão do máximo diário e do P95 diário.

A experiência adquirida com preparação dos dados, análise de periodicidade, critérios de completude, validação temporal e avaliação de erros poderá servir como base metodológica para essa investigação.

**Hipótese exploratória futura:**

> Padrões locais de variação, tendência e volatilidade do tráfego podem conter informações capazes de antecipar determinados comportamentos anômalos em horizontes curtos, embora essa capacidade dependa da natureza dos eventos e precise ser demonstrada empiricamente.



