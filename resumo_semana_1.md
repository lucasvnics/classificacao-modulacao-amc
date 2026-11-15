# Resumo da Semana 1: Classificação Automática de Modulação para IoT

Autor: Lucas Vinnicius Duarte de Oliveira Carlos (3º período de Engenharia de Telecomunicações — UFPE)

## 1. Objetivo do Estudo

Nesta primeira semana, o objetivo foi compreender os fundamentos da camada física e avaliar como diferentes formas de representar um sinal digital afetam a acurácia (porcentagem de acertos) de uma rede neural ao identificar quatro modulações (BPSK, QPSK, 8PSK e 16QAM) sob ruído. O foco prático é buscar soluções leves que funcionem bem em dispositivos IoT (Internet das Coisas, que possuem pouca bateria e baixo poder de processamento).

## 2. Metodologia

Para isolar o impacto da representação dos dados, mantive exatamente a mesma rede neural densa (MLP, perceptron multicamadas com duas camadas ocultas de 64 neurônios) e comparei três entradas diferentes:

- IQ Bruto: entrega os valores das componentes em fase (I) e em quadratura (Q) da constelação na ordem aleatória em que chegam.
- IQ Ordenado: organiza separadamente os valores de I e Q do menor para o maior antes de enviá-los à rede, eliminando o efeito da ordem aleatória dos símbolos.
- Cumulantes de Ordem Superior: resume todo o bloco de símbolos em apenas 4 estatísticas (|C20|, |C40|, |C41| e C42) que descrevem o formato geométrico da nuvem de pontos.

Os testes avaliaram o comportamento dos modelos em cenários de ruído AWGN (ruído branco gaussiano aditivo) variando a SNR (relação sinal-ruído em decibéis) e o tamanho da janela de amostra (32, 128 e 512 símbolos).

## 3. Resultados e Descobertas

A tabela reúne a acurácia obtida nos experimentos (onde o chute aleatório entre 4 classes equivale a 25%):

| Representação | 32 símb. (8 / 20 dB) | 128 símb. (8 / 20 dB) | 512 símb. (8 / 20 dB) |
| :--- | :---: | :---: | :---: |
| IQ bruto | 47,8% / 50,3% | 40,8% / 41,8% | 33,8% / 34,2% |
| IQ ordenado | 76,8% / 87,0% | 88,8% / 99,9% | 95,3% / 100,0% |
| Cumulantes | 82,8% / 88,4% | 97,4% / 97,4% | 99,8% / 99,6% |

Os números confirmaram três pontos principais:

- A ordem aleatória atrapalha o IQ bruto: o modelo fica entre 33,8% e 50,3% porque embaralhar os símbolos não muda a modulação, fazendo a rede densa tentar decorar padrões aleatórios.
- O cruzamento de desempenho com ruído: o IQ ordenado alcança 100% quando o sinal está limpo (20 dB com 512 símbolos), mas cai para 95,3% em cenário ruidoso (8 dB) porque o ruído borra os degraus criados pela ordenação e destrói o par original I e Q de cada ponto.
- Confirmação da hipótese dos cumulantes: com amostras curtas (32 símbolos), a estatística de 4ª ordem fica instável (82,8% a 8 dB), mas com 512 símbolos ela estabiliza, neutraliza o efeito do ruído gaussiano e vence todos os outros métodos em 8 dB (99,8% de acurácia) usando apenas 4 números como entrada.

## 4. Limitações desta Primeira Etapa

Para manter o rigor metodológico, registro as premissas simplificadas usadas nesta semana:

- Os experimentos utilizam sinais 100% simulados em Python, e não capturas reais pelo ar.
- O canal considera apenas ruído térmico (AWGN), sem efeitos de desvanecimento ou reflexões em obstáculos.
- O receptor assume sincronismo perfeito de símbolo, fase e frequência, e os resultados dependem dos hiperparâmetros escolhidos para a rede.
