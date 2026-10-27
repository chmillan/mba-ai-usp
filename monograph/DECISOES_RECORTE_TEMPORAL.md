# Decisões sobre o recorte temporal

## Objetivo

O objetivo principal do TCC é desenvolver e avaliar um **modelo de previsão da série temporal de tráfego**, com atenção ao desempenho nos picos e à aplicação no planejamento de capacidade do PTT/IX.

A investigação de anomalias é complementar: se viável, identificar observações inadequadas para o treinamento e avaliar seu tratamento. Construir um detector de anomalias não é requisito para concluir o trabalho.

## Recorte inicial adotado

Manter **1º de janeiro de 2023** como início do treinamento na estratégia principal, com a seguinte divisão cronológica:

| Etapa | Período | Uso |
|---|---|---|
| Treino inicial | 2023 | Aprender os padrões da série. |
| Validação | 2024 | Comparar modelos, atributos, recortes e tratamentos; selecionar hiperparâmetros. |
| Teste final | 2025 completo | Avaliar a estratégia escolhida, sem usar os resultados para ajustar o modelo. |
| Avaliação complementar | Janeiro de 2026 disponível | Verificar o desempenho nos dados mais recentes, sem substituir o teste anual. |

Existe também a estratégia em que usamos o histórico mais longo.

| Etapa | Período | Uso |
|---|---|---|
| Treino inicial | 2021-2023 | Aprender os padrões da série. |
| Validação | 2024 | Comparar modelos, atributos, recortes e tratamentos; selecionar hiperparâmetros. |

Se o histórico mais longo produzir previsões melhores em 2024, escolheremos essa alternativa.

Depois de escolher, treinaremos novamente incluindo também 2024, pois ele já terá cumprido seu papel de validação. (Preciso buscar referências para isso) Nesse caso, o treinamento final abrangeria 2021-2024 e o teste continuaria sendo 2025.

E “2025 completo” não significa prever o ano inteiro de uma vez: é possível, por exemplo, fazer previsões para o dia seguinte ao longo do ano, sempre usando apenas informações disponíveis até cada previsão.



## Justificativa e limites

A análise do [notebook 02](notebooks/02_peak_analysis.ipynb) indica forte expansão entre 2020 e 2022: a média dos máximos diários cresceu aproximadamente 75,3%, 76,1% e 37,0%, respectivamente. O P95 dos máximos diários atingiu cerca de 12.511 Gbps em 2022.

O período 2023–2025 representa uma escala de tráfego mais próxima da atual e oferece três anos completos para estudar padrões recorrentes. Ainda inclui variações relevantes, como os extremos de 2024. Começar apenas em 2025 reduziria a diversidade de situações e deixaria somente um ciclo anual completo.

O corte em 2023 é uma hipótese metodológica inicial, não uma ruptura estatisticamente comprovada. As análises mostram mudanças no tráfego, mas não estabelecem que tenham sido causadas pela entrada ou expansão dos serviços de streaming. Essa atribuição exigiria evidências adicionais.

O treino inicial de apenas um ano limita a diversidade observada e o estudo de padrões anuais. Por isso, comparar o treino em 2021–2023 com o treino somente em 2023, sempre usando 2024 como validação, antes da avaliação final em 2025.

Os dados de 2025 já foram examinados na análise exploratória e em Fourier; portanto, não constituem um período totalmente inédito. Registrar essa limitação e, a partir desta decisão, não usá-los para selecionar atributos, frequências, tratamentos ou modelos. Recalcular as escolhas dependentes dos dados apenas no treino e avaliá-las na validação.

Eventos de 2021–2022 não devem ser classificados automaticamente como erros ou descartados como impossíveis de ocorrer novamente. Podem apoiar uma avaliação histórica complementar, sem substituir a avaliação em dados futuros.

## O que vale comparar posteriormente

| Comparação | Finalidade |
|---|---|
| Treino em 2021–2023 versus somente 2023 | Verificar se o histórico adicional melhora a previsão ou introduz padrões menos representativos. Validar ambas as opções em 2024. |
| Janela de treinamento crescente versus janela móvel recente | Avaliar a adaptação a mudanças de comportamento e o efeito de manter observações antigas. |
| Baselines sazonais versus modelos de IA | Verificar o ganho sobre referências simples, como o mesmo horário do dia anterior ou da semana anterior. |
| Modelos com e sem atributos derivados dos ciclos de Fourier | Testar se os ciclos identificados contribuem para o desempenho fora da amostra. Selecionar frequências apenas no treino. |
| Previsão do tráfego versus previsão dos máximos diários | Avaliar tanto o comportamento habitual quanto o risco de subestimar a demanda máxima. |
| Treino com e sem tratamento de anomalias, se viável | Verificar se um tratamento justificado melhora a previsão na mesma validação, sem excluir automaticamente picos legítimos. |
| Avaliação futura versus investigação dos extremos históricos | Medir generalização em períodos posteriores e examinar separadamente a resposta a eventos como os de 2021–2022. |

As comparações devem usar divisões temporais, sem embaralhamento, e horizontes de previsão compatíveis com a decisão operacional. Priorizar métricas de previsão, como MAE e RMSE, acompanhadas de erros nos picos e subestimação. Se houver previsões probabilísticas, avaliar também a cobertura dos intervalos. Selecionar a estratégia em 2024 e manter 2025 reservado para o teste final, sem ciclos de ajuste guiados por seus resultados.

## Anomalias como etapa complementar

Investigar possíveis erros de medição, falhas de coleta e eventos atípicos, caso haja tempo e evidências suficientes. Um pico alto pode representar demanda legítima e importante para o objetivo do trabalho; sua intensidade, por si só, não justifica exclusão.

Qualquer remoção ou tratamento deve ter critério documentado, preservar os dados originais e ser ajustado apenas no conjunto de treino. Preservar o índice temporal: excluir uma observação não deve juntar artificialmente instantes separados. Comparar o efeito do tratamento em 2024 antes de adotá-lo.

Não remover eventos reais da validação ou do teste para melhorar as métricas. Eventuais erros de medição comprovados nesses períodos devem ser registrados e tratados por um protocolo de qualidade explícito. Se a investigação não produzir critérios confiáveis, prosseguir com a previsão sem uma etapa específica de exclusão de anomalias.


## Critério de completude diária para os máximos

Na revisão da qualidade dos dados, foi reavaliado o critério utilizado para considerar um máximo diário representativo. A regra inicial exigia pelo menos 274 das 288 observações de cinco minutos esperadas em um dia, correspondendo a aproximadamente 95% de cobertura. Esse critério mostrou-se particularmente restritivo em 2024: 82 dias não atingiam o limite estabelecido.

Foi então investigada a possibilidade de reduzir o limiar de completude para 90%, correspondente a pelo menos 260 das 288 observações diárias. Em 2024, 284 dias satisfazem o critério de 95%, 63 dias adicionais são recuperados pelo critério de 90% e 19 dias permanecem abaixo de 90%.

A decisão não foi baseada apenas no aumento da quantidade de dias disponíveis. Foram examinadas a quantidade e a distribuição temporal das observações ausentes nos dias afetados pela mudança de limiar. Nos 63 dias recuperados pelo critério de 90%, a mediana é de 18 observações ausentes, equivalentes a 90 minutos distribuídos ao longo do dia. O maior bloco contínuo de ausência apresenta mediana de 15 minutos e máximo de 25 minutos. Para comparação, entre os dias que já satisfazem o critério de 95%, o maior bloco apresenta mediana de 10 minutos e chega a 30 minutos. Assim, a flexibilização para 90% não introduz, nesse conjunto, dias caracterizados por longos períodos contínuos sem dados.

Também foi investigado se as ausências poderiam comprometer especificamente a estimativa do máximo diário. Parte dos valores ausentes ocorre próxima ao horário do máximo observado, de modo que a completude global do dia, isoladamente, não garante que o verdadeiro pico tenha sido observado. Por esse motivo, foi examinada também a robustez dos máximos.

Nos dias recuperados pelo limiar de 90%, a razão mediana entre o segundo maior valor observado e o máximo diário é de 99,45%. Nos dias que já satisfazem 95% de cobertura, essa razão é de 99,43%. Portanto, o comportamento típico dos máximos é muito semelhante nos dois grupos. Foram encontrados, entretanto, alguns máximos isolados, com grande diferença em relação às observações vizinhas. Esses casos também ocorrem entre os dias com pelo menos 95% de cobertura, inclusive com valores relativos mais extremos. Isso indica que a presença de máximos isolados constitui uma questão distinta da completude diária e não é resolvida pela manutenção do limiar de 95%.

Com base nesses diagnósticos, o limiar de 90% foi considerado adequado como critério de completude diária para a análise dos máximos. A escolha procura evitar a exclusão de dias com pequenas lacunas distribuídas ao longo do dia sem confundir completude com qualidade do próprio valor máximo.

A definição do pico diário permanece como uma decisão metodológica separada. A utilização do máximo absoluto (`max`) será comparada posteriormente com medidas mais robustas, como P95 e P99, antes da definição final dos atributos e alvos relacionados aos picos.

Esta revisão de 95% para 90% afeta a série de máximos diários. Ela não altera diretamente a série horária utilizada na FFT e no Welch do tráfego horário, cuja disponibilidade depende do critério de completude horária e da regra de interpolação de lacunas curtas.

## Relação com capacidade (totalmente opcional, é necessário avaliar a viabilidade de conseguir esses dados junto à empresa)

Prever tráfego elevado não basta para concluir se o PTT está preparado. Será necessário obter capacidade utilizável, margens operacionais e informações sobre gargalos e redundância. A série analisada soma entrada e saída; qualquer comparação com capacidade deve respeitar a mesma abrangência, direção e unidade.




####
o passar das medições de cinco minutos para indicadores diários, reduzimos bastante o número de observações disponíveis para treinar os modelos. E isso é uma questão metodológica importante para o TCC.
Quantas observações teremos?
Série de 5 minutos — 2023 e 2024
210.528
posições esperadas


Série diária — 2023 e 2024
731
dias possíveis



Como adotamos o critério mínimo de 90% de completude, o número efetivo de dias elegíveis será menor que 731.

E, respeitando a divisão temporal:
- 2023: até 365 observações para desenvolvimento e treinamento.
- 2024: até 366 observações para validação.
- 2025: até 365 observações para teste final.
Isso significa que, inicialmente, teremos no máximo 365 exemplos de treinamento, antes de descontar dias inválidos e defasagens.
Isso é um problema?
É uma limitação relevante, especialmente para modelos complexos. Mas não significa que nossa abordagem esteja errada.

Há duas estratégias possíveis:
Estratégia	Vantagem	Limitação
Prever diretamente o máximo e o P95 do dia seguinte	Alinhada aos objetivos do TCC; problema mais simples	Poucos exemplos de treinamento
Prever o tráfego em intervalos de 5 minutos e agregar as previsões para obter máximo e P95	Aproveita a resolução temporal original	Problema mais complexo, com 288 previsões para cada dia

A segunda abordagem não garante melhores resultados: erros nas previsões de cinco minutos podem se propagar para o máximo e o P95 diário.
O que eu sugiro para o TCC
Manteria a previsão direta dos indicadores diários como estratégia inicial, mas trataria o tamanho do conjunto de treinamento como uma questão a investigar.

Isso também reforça uma decisão que já havíamos considerado: comparar o treinamento utilizando apenas 2023 com um histórico mais longo, por exemplo, 2021–2023.

Além disso, podemos começar com baselines sazonais e modelos relativamente simples, como regressão regularizada e modelos baseados em árvores, antes de considerar arquiteturas mais complexas.

Um detalhe importante: mesmo prevendo apenas um valor por dia, não estamos obrigados a utilizar somente informações diárias como entrada. Podemos construir atributos com base nas 288 medições de cada dia anterior, desde que essas informações estejam disponíveis no momento da previsão.

Por exemplo, para prever o máximo de amanhã, podemos utilizar o máximo de hoje, o P95 de hoje, a variabilidade intradiária e indicadores dos últimos sete dias.

Portanto, teremos poucos exemplos supervisionados, mas ainda podemos extrair bastante informação da série original.

Eu manteria a abordagem atual até os primeiros baselines do notebook 05. Os resultados nos ajudarão a decidir se vale explorar uma estratégia de previsão em resolução mais alta.



####### Ideias do que incluir na discussao 


Seu trabalho não precisa determinar qual é a capacidade ideal da infraestrutura do IX.br. O objetivo é investigar até que ponto modelos de séries temporais conseguem antecipar a demanda elevada e os picos de tráfego, produzindo informações potencialmente úteis ao planejamento.

São duas questões distintas:
- Problema preditivo: qual será o máximo ou P95 de tráfego amanhã? Qual é o erro dessa previsão?
- Problema operacional: qual margem de capacidade é necessária para suportar essa demanda com um nível aceitável de risco?
Seu TCC aborda principalmente a primeira questão. A segunda depende de informações operacionais que você não terá, como capacidade efetivamente disponível, critérios de redundância, margens de segurança e tolerância à saturação.
Como tratar isso na monografia
Eu distribuiria essa discussão em três momentos.
Seção	O que registrar
Metodologia	As métricas de avaliação serão estatísticas, sem estabelecer um limite operacional de erro
Resultados	Quantificar os erros, especialmente a frequência e a magnitude das subestimações
Discussão e limitações	Explicar que a tradução dos erros preditivos em decisões de capacidade depende de critérios operacionais externos ao estudo

Isso não impede que você investigue o comportamento dos modelos nos dias de maior demanda.

Por exemplo, podemos definir um grupo de dias de alta demanda usando o P90 ou P95 da distribuição dos máximos diários do conjunto de treinamento. Esse limiar teria finalidade analítica, não representaria um limite de capacidade da infraestrutura.

Assim, conseguiríamos responder perguntas relevantes: os modelos apresentam erros maiores nos dias de tráfego elevado? Subestimam sistematicamente os máximos? Existe diferença entre prever P95 e máximo?
Sugestão de texto para a discussão do TCC
Limitações quanto à aplicação operacional das previsões
Embora a previsão de demanda elevada e de picos de tráfego possa fornecer informações relevantes para o planejamento de capacidade de infraestruturas de interconexão, a tradução dessas previsões em decisões operacionais depende de fatores que não são contemplados neste estudo.

Entre esses fatores estão a capacidade efetivamente disponível, os requisitos de redundância, as margens de segurança e os níveis de risco considerados aceitáveis pelos operadores da infraestrutura.

Dessa forma, o trabalho não pretende estabelecer limites operacionais de utilização nem determinar diretamente a capacidade necessária para o IX.br São Paulo. A avaliação será concentrada na precisão dos modelos e em seu comportamento nos períodos de maior demanda, com atenção especial à frequência e à magnitude das subestimações.

Os resultados poderão, portanto, contribuir para a compreensão do potencial e das limitações das técnicas de previsão como instrumentos de apoio ao planejamento, sem substituir análises específicas de engenharia e dimensionamento de capacidade.


Uma recomendação para os próximos experimentos
Eu manteria MAE, RMSE e métricas específicas de subestimação, sem introduzir uma tolerância operacional arbitrária.

Também podemos investigar erros relativos e o desempenho nos dias de maior demanda. Isso permitirá uma discussão mais rica do que simplesmente identificar qual modelo tem o menor MAE.

Portanto, não precisamos adiar o notebook 04 nem criar um experimento adicional para definir tolerâncias operacionais. Basta registrar essa delimitação de escopo e retomá-la na discussão final.

Aliás, essa distinção fortalece a coerência metodológica do trabalho: você estará avaliando a qualidade das previsões sem atribuir a elas uma capacidade de orientar diretamente o dimensionamento que os dados disponíveis não permitem demonstrar.