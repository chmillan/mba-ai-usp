# FFT — Transformada Rápida de Fourier




Uma forma intuitiva de compreender a análise de Fourier é imaginar a série temporal de tráfego como uma música composta por diferentes ritmos simultâneos. Ao observar o tráfego no domínio do tempo, todos esses comportamentos aparecem combinados em uma única série.

A análise de Fourier permite decompor essa série em componentes de diferentes frequências e investigar quais delas têm maior importância no sinal. Essas componentes podem revelar estruturas periódicas presentes nos dados.


Por exemplo, o tráfego pode apresentar um comportamento que se repete aproximadamente a cada 24 horas:

```
baixo → sobe → pico → desce → baixo
baixo → sobe → pico → desce → baixo
baixo → sobe → pico → desce → baixo
        DIA 1           DIA 2
```

NNesse caso, o espectro de Fourier tende a apresentar uma concentração de energia associada ao período de 24 horas, indicando uma componente diária importante.

Ao mesmo tempo, pode existir uma variação associada aos dias da semana — por exemplo, diferenças sistemáticas entre dias úteis e finais de semana. Nesse caso, também pode aparecer uma componente associada a um período próximo de 7 dias.

Uma série temporal pode, portanto, conter simultaneamente:
```
padrão de 24 horas
+
padrão de 7 dias
+
outros padrões
+
ruído
+
eventos excepcionais
```
nquanto o gráfico no domínio do tempo mostra como o tráfego evolui ao longo do tempo, o espectro de Fourier mostra quais frequências — e, consequentemente, quais períodos — apresentam maior contribuição para a variabilidade da série.

De forma intuitiva, a análise de Fourier permite pegar um sinal complexo observado ao longo do tempo e investigar quais ritmos estão escondidos dentro dele.

Em vez de observar apenas:
```
Gbps
 ↑
 |    /\      /\      /\      /\
 |   /  \    /  \    /  \    /  \
 |__/    \__/    \__/    \__/    \__
 +------------------------------------→ tempo
```

a análise permite observar algo conceitualmente semelhante a:

```
intensidade
 ↑
 |                     │
 |                     │  24 h
 |                     │
 |       │             │
 |       │             │        │
 +------------------------------------→ frequência
      semanal       1/dia    2/dia
```
## DFT e FFT

A Transformada Discreta de Fourier (DFT) é o procedimento matemático que permite obter o espectro de frequências de uma série discreta. A FFT (Fast Fourier Transform) é um algoritmo eficiente para calcular essa transformada.

Assim, quando aplicamos a FFT à série de tráfego, obtemos uma representação que permite identificar concentrações de energia em determinadas frequências ou períodos.

Tendência e componentes periódicas

De forma simplificada, podemos pensar que a série de tráfego contém diferentes componentes:

\text{tendência}(t)
+
\text{sazonalidade}(t)
+
\text{variação residual}(t)
]

A tendência representa mudanças de longo prazo no nível da série. Por exemplo, o volume médio de tráfego pode aumentar progressivamente ao longo dos anos.

A sazonalidade representa padrões que se repetem em determinados intervalos, como ciclos diários ou semanais.

Assim, é possível que exista uma oscilação diária ao mesmo tempo em que o nível em torno do qual essa oscilação ocorre esteja aumentando:

```
nível do tráfego
 ↑
 |                         /\    /\
 |                   /\   /  \__/  \
 |             /\   /  \_/
 |       /\   /  \_/
 |  /\__/  \_/
 +------------------------------------→ tempo
          tendência crescente
       + oscilações periódicas
```

A análise de Fourier realizada neste notebook está voltada principalmente à identificação dessas **componentes periódicas**. A tendência de longo prazo é uma característica diferente da série e deve ser considerada separadamente na análise e, posteriormente, na construção do modelo de previsão.

## Vazamento espectral e janela de Hann

Para identificar as componentes periódicas por meio da FFT, é necessário considerar que o cálculo é realizado sobre um trecho finito da série temporal.

Esse trecho pode começar e terminar em pontos diferentes de um mesmo ciclo. Quando os valores nas duas extremidades são muito diferentes, o recorte cria uma descontinuidade artificial. Na análise espectral, essa descontinuidade passa a ser interpretada como parte do sinal.

Como consequência, a energia de uma componente pode se espalhar para frequências vizinhas, introduzindo outras frequências no espectro. Esse fenômeno é chamado de vazamento espectral (spectral leakage).

Para reduzir esse efeito, antes do cálculo da FFT foi aplicada uma janela de Hann. A janela atribui pesos progressivamente menores às observações próximas às extremidades do trecho, fazendo com que o sinal se aproxime suavemente de zero no início e no fim. Dessa forma, reduz-se a descontinuidade provocada pelo recorte e, consequentemente, o vazamento espectral.

É importante observar que a janela de Hann não funciona como uma média móvel e não tem como objetivo suavizar a evolução do tráfego ou eliminar seus picos. Ela modifica os pesos das observações especificamente para tornar a análise espectral de um trecho finito mais adequada. Portanto, sua aplicação faz parte do procedimento de estimação do espectro, e não de um tratamento destinado a remover variações reais da série.

## No contexto do TCC

Neste trabalho, a análise espectral busca responder perguntas como:

Existe uma componente diária forte?

Existe uma componente semanal?

Aparecem componentes correspondentes a períodos de 12 ou 8 horas?

Essas componentes podem ser harmônicos do ciclo diário, em vez de ciclos independentes?

As componentes identificadas em 2023 também aparecem em 2024?

Essas periodicidades permanecem relativamente estáveis ao longo do tempo?

A análise de Fourier não tem como objetivo prever diretamente o valor futuro do tráfego. Ela é utilizada para identificar estruturas temporais periódicas potencialmente úteis para a construção de features de entrada para o modelo de previsão.

Por exemplo, uma estrutura diária e semanal forte e persistente fornece justificativa empírica para posteriormente testar atributos relacionados à hora do dia, ao dia da semana, lags de 24 h, lags de 168 h ou representações cíclicas equivalentes, em vez de simplesmente escolher essas features por intuição.

resumo documento:
há ritmos escondidos → Fourier permite enxergá-los → FFT é como calculamos → precisamos separar isso de tendência → existe um problema técnico ao calcular → usamos Hann → e é por isso que fazemos tudo isso no TCC.