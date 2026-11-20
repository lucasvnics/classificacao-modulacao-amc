# Simulação de Rede LoRaWAN e Benchmark da MLP Ultraleve no ESP32

Esta etapa estende o estudo de Classificação Automática de Modulação (AMC) da camada física para o nível de enlace e rede, avaliando a viabilidade computacional e energética de embarcar o classificador leve (Cumulantes + MLP) em um microcontrolador com restrição severa de hardware (ESP32) operando em uma rede IoT sem fio.

## 1. Ambiente e Configuração da Simulação

Para simular o tráfego de pacotes sem fio, utilizei o simulador de eventos discretos LoRaSimPlus em Python, configurando uma topologia ponto-a-ponto limpa (sem colisões de múltiplos usuários) para isolar o desempenho de um único enlace:

- Topologia: 1 nó transmissor (coordenadas x = 1204 m, y = 280 m) e 1 receptor/gateway central (0, 0), separados por aproximadamente 1.236 metros.
- Duração simulada: 1 hora (3.600.000 ms).
- Modelo de Rede Neural Embarcada: MLP (4 entradas -> 16 neurônios ocultos -> 4 saídas), totalizando 148 parâmetros treináveis.
- Referência de Hardware (Receptor): Microcontrolador ESP32 operando a 240 MHz, 520 KB de SRAM e potência ativa média de 260 mW.

## 2. Resultados de Rede (LoRaSimPlus)

Durante 1 hora de operação do enlace ponto-a-ponto, o simulador registrou os seguintes indicadores de desempenho:

| Métrica de Rede | Valor Obtido |
| :--- | :---: |
| Total de pacotes enviados | 618 pacotes |
| Pacotes recebidos com sucesso | 569 pacotes |
| Pacotes perdidos por colisão | 0 pacotes |
| Pacotes perdidos por atenuação/ruído | 49 pacotes |
| Taxa de Entrega de Dados (DER) | 92,07% |
| Volume de dados úteis entregues | 36.985 bytes (de 40.170 bytes) |
| Vazão efetiva da rede (Throughput) | 882,215 bps |
| Energia total gasta na transmissão (Rádio) | 8,424 Joules (8.424 mJ) |
| Energia efetiva por byte entregue | 0,2278 mJ/byte |

## 3. Benchmark da MLP Ultraleve no Receptor (Emulação ESP32)

A cada pacote recebido intacto no gateway (569 pacotes), o módulo mlp_esp32.py executou a inferência da rede neural utilizando multiplicação matricial pura em float32:

| Métrica Computacional e Energética | Valor Obtido |
| :--- | :---: |
| Arquitetura da Rede Neural | MLP (4 -> 16 -> 4) |
| Total de parâmetros | 148 parâmetros |
| Ocupação de memória RAM (pesos em float32) | 592 bytes (< 0,12% da SRAM do ESP32) |
| Acurácia na recepção | 100,00% (569/569 pacotes) |
| Latência estimada por pacote (@ 240 MHz) | 25,0 microssegundos |
| Energia gasta pela CPU por inferência | 6,50 microjoules |
| Energia total acumulada pela IA (569 pacotes) | 3,6985 mJ (0,0037 Joules) |

## 4. Conclusão Principal (Rádio vs. Processamento na Borda)

A comparação energética demonstra a viabilidade de executar inteligência artificial diretamente na borda (Edge AI / TinyML) em sensores alimentados por bateria:

- Enquanto o transceptor de rádio consumiu 8,424 Joules para transmitir os 618 pacotes, o processamento da MLP ultraleve no receptor consumiu apenas 0,0037 Joules no total — uma economia superior a 2.200 vezes em relação ao gasto de comunicação sem fio.
- A pegada de memória de apenas 592 bytes permite que o modelo coexista facilmente com a pilha de protocolos de rede sem exigir hardware dedicado.

## 5. Limitações Metodológicas Desta Etapa

- O LoRaSimPlus modela eventos de pacote, atenuação de percurso e relação sinal-ruído (SNR) no nível de enlace, não gerando formas de onda I/Q brutas. Assim, a SNR calculada pelo simulador para cada pacote foi utilizada para perturbar os cumulantes teóricos e medir o custo real da inferência embarcada.
- A latência (25 us) e a energia de CPU (6,5 uJ) foram estimadas analiticamente com base na contagem de operações em ponto flutuante (FLOPs) e nas especificações elétricas do ESP32 a 240 MHz, devendo ser validadas futuramente em bancada física.
