# Classificação Automática de Modulação (AMC) Leve para Redes IoT e Ad Hoc

Projeto pessoal de pesquisa e estudo independente em Inteligência Artificial aplicada a redes sem fio (IoT, redes ad hoc e 5G/6G). O objetivo é identificar automaticamente qual modulação digital foi utilizada em uma transmissão (BPSK, QPSK, 8PSK ou 16QAM) a partir de sinais recebidos com ruído, priorizando modelos ultraleves capazes de rodar diretamente na borda (Edge AI / TinyML) em microcontroladores com severa restrição de memória e bateria.

## 1. Motivação e Problema de Pesquisa

Em redes de sensores sem fio, redes ad hoc e dispositivos de Internet das Coisas (IoT), a autonomia de bateria e a baixa capacidade computacional são os maiores gargalos. Abordagens tradicionais de aprendizado profundo (como CNNs operando diretamente sobre formas de onda brutas) exigem muita memória e processamento, tornando-se inviáveis para nós alimentados por bateria.

Pergunta central do estudo: como a forma de representar o sinal recebido afeta a acurácia, a latência e o consumo energético de uma rede neural embarcada na classificação de modulações sob ruído?

## 2. Metodologia em Duas Frentes

### Frente 1: Camada Física e Engenharia de Atributos (Python / Scikit-Learn)
Comparei três formas diferentes de entregar o sinal complexo (em fase I e quadratura Q) sob canal com Ruído Branco Gaussiano Aditivo (AWGN) para uma rede neural densa (MLP):

1. IQ Bruto: entrega as componentes I e Q na ordem aleatória em que chegam.
2. IQ Ordenado: ordena separadamente os valores de I e Q do menor para o maior antes da rede.
3. Cumulantes de Ordem Superior: resume toda a janela de símbolos em apenas 4 estatísticas matemáticas (|C20|, |C40|, |C41| e C42) que descrevem o formato geométrico da constelação.

### Frente 2: Simulação de Rede e Benchmark de Hardware (LoRaSimPlus + ESP32)
Para validar a viabilidade prática em um cenário de comunicação sem fio, acoplei o classificador leve (mlp_esp32.py) ao simulador de eventos discretos LoRaSimPlus em um enlace ponto-a-ponto limpo (1 nó transmissor e 1 receptor/gateway separados por 1.236 metros, durante 1 hora sem colisões), estimando o custo computacional e energético em um microcontrolador ESP32 (240 MHz, 520 KB de SRAM).

## 3. Principais Resultados Experimentais

### 3.1 Comparação de Representações e Tamanho de Janela (MLP 64x64)
Avaliação de acurácia em cenários ruidosos (8 dB) e limpos (20 dB) variando o número de símbolos por janela (onde o chute aleatório entre 4 classes equivale a 25%):

| Representação de Entrada | 8 dB (32 símb.) | 8 dB (128 símb.) | 8 dB (512 símb.) | 20 dB (32 símb.) | 20 dB (128 símb.) | 20 dB (512 símb.) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| IQ Bruto | 47,8% | 40,8% | 33,8% | 50,3% | 41,8% | 34,2% |
| IQ Ordenado | 76,8% | 88,8% | 95,3% | 87,0% | 99,9% | 100,0% |
| Cumulantes (4 atributos) | 82,8% | 97,4% | 99,8% | 88,4% | 97,4% | 99,6% |

Principais achados da camada física:
- O IQ Bruto falha porque embaralhar a ordem dos símbolos não altera a modulação (invariância à permutação), fazendo a rede densa decorar padrões aleatórios.
- Com janelas de 512 símbolos, os 4 Cumulantes atingem 99,8% de acurácia sob ruído forte (8 dB), pois estatísticas de 4ª ordem anulam teoricamente o ruído gaussiano enquanto comprimem 1.024 valores brutos em apenas 4 números.
- Testes adicionais de estresse mostraram que treinar a rede apenas em sinal limpo (20 dB) causa overfitting (a acurácia em 8 dB cai para 75,0%), comprovando a necessidade de treinar em múltiplas SNRs (-4 a 20 dB). A matriz de confusão também revelou 0% de erro no BPSK, concentrando as raras trocas entre QPSK e 16QAM.



### 3.2 Compressão da Rede e Benchmark no ESP32 (LoRaSimPlus)
Reduzi a arquitetura para uma MLP ultraleve de apenas uma camada oculta com 16 neurônios (4 -> 16 -> 4, totalizando 148 parâmetros) e avaliei sua execução no receptor durante 1 hora de tráfego LoRaWAN:

| Métrica Avaliada (Rede e Hardware ESP32 @ 240 MHz) | Resultado Obtido |
| :--- | :---: |
| Pacotes enviados / recebidos no enlace (1.236 m) | 618 enviados / 569 recebidos (DER: 92,07%) |
| Vazão efetiva da rede (Throughput) | 882,215 bps |
| Energia total gasta na transmissão via Rádio | 8,424 Joules (8.424 mJ) |
| Memória ocupada pela MLP (148 parâmetros em float32) | 592 bytes (< 0,12% da SRAM do ESP32) |
| Acurácia da MLP na recepção | 100,00% (569/569 pacotes) |
| Latência estimada por inferência no ESP32 | 25,0 microssegundos |
| Energia gasta pela CPU na IA (569 pacotes) | 3,6985 mJ acumulados (6,50 uJ/pacote) |

Conclusão energética: enquanto o transceptor de rádio consumiu 8,424 Joules para transmitir os pacotes, o processamento da rede neural no receptor gastou apenas 0,0037 Joules no total — mais de 2.200 vezes menos energia que a transmissão sem fio, validando a viabilidade de inteligência na borda para nós IoT com restrição severa de bateria.

## 4. Estrutura do Repositório

- README.md: visão geral do projeto, metodologia e consolidação dos resultados.
- classificado_modulacao_py.ipynb: notebook principal com simulação AWGN, extração de cumulantes, treino das MLPs e matrizes de confusão.
- mlp_esp32.py: módulo leve em Python/NumPy para emulação de inferência, memória e gasto energético no ESP32 acoplado ao LoRaSimPlus.
- simulacao_lorasimplus_esp32.md: relatório técnico detalhado dos experimentos de rede no LoRaSimPlus.
- resumo_semana_1.md: registro inicial das primeiras descobertas e hipóteses do projeto.
- acuracia_vs_snr.png: curvas comparativas de desempenho por nível de ruído.

## 5. Como Executar (Reprodutibilidade)

Os experimentos utilizam semente aleatória fixa (np.random.default_rng(42)) para garantir reprodutibilidade. Para rodar localmente:

```bash
pip install numpy scikit-learn matplotlib simpy
