<img width="690" height="440" alt="acuracia_vs_snr" src="https://github.com/user-attachments/assets/09eb18b2-e1c1-4d91-bd74-02903ff34545" />
# Classificação Automática de Modulação (AMC) com Redes Neurais

Sou estudante do 3º período de Engenharia de Telecomunicações e criei este repositório para documentar meu projeto de estudo independente em Inteligência Artificial aplicada a redes sem fio, como **IoT** (Internet das Coisas, que conecta dispositivos simples com pouca bateria e baixo processamento) e 5G/6G. 

A ideia central é resolver um problema clássico da camada física: descobrir automaticamente qual **modulação** (regra que transforma bits em alterações de fase e amplitude de uma onda) foi usada na transmissão — entre **BPSK, QPSK, 8PSK e 16QAM** — olhando apenas para o sinal que chega sujo de ruído no receptor.

## O que estou testando

A partir de conversas e orientações acadêmicas sobre o alto custo computacional de redes profundas, decidi focar em abordagens mais leves que façam sentido para hardwares simples de IoT[cite: 1, 2]. 

Em vez de criar uma rede gigante, mantive exatamente a mesma **MLP** (perceptron multicamadas, uma rede neural densa tradicional com duas camadas ocultas de 64 neurônios) e mudei apenas a forma de entregar o sinal para ela[cite: 1, 2]:

1. **IQ Bruto:** passo os valores dos eixos I (em fase) e Q (em quadratura) da **constelação** (gráfico que representa os símbolos transmitidos como pontos no plano) na ordem exata em que chegam.
2. **IQ Ordenado:** organizo separadamente as partes I e Q do menor para o maior antes de enviar para a rede, tirando o efeito da ordem aleatória dos símbolos.
3. **Cumulantes:** resumo todo o bloco de símbolos em apenas 4 números estatísticos ($|C_{20}|$, $|C_{40}|$, $|C_{41}|$ e $C_{42}$) que descrevem o formato da nuvem de pontos[cite: 1, 2].

## Resultados da Semana 1

Simulei o envio dos sinais em um canal **AWGN** (ruído branco gaussiano aditivo, usado para imitar o ruído térmico dos circuitos) variando a **SNR** (relação sinal-ruído, medida em decibéis que compara a potência do sinal útil com a do ruído). Também variei a quantidade de símbolos observados por vez (32, 128 e 512) para ver como o tamanho da amostra afeta a **acurácia** (porcentagem de acertos do modelo, onde um chute aleatório entre 4 opções equivale a 25%)[cite: 1, 2]:

| Entrada | 32 símb. (8 / 20 dB) | 128 símb. (8 / 20 dB) | 512 símb. (8 / 20 dB) |
| :--- | :---: | :---: | :---: |
| **IQ bruto** | 47,8% / 50,3% | 40,8% / 41,8% | 33,8% / 34,2% |
| **IQ ordenado** | 76,8% / 87,0% | 88,8% / 99,9% | 95,3% / 100,0% |
| **Cumulantes** | 82,8% / 88,4% | 97,4% / 97,4% | 99,8% / 99,6% |

Na prática, os números acima mostram três comportamentos bem claros no experimento[cite: 1, 2]:

O **IQ bruto** vai mal em qualquer cenário (e até piora com 512 símbolos) porque a ordem dos símbolos sorteados é aleatória[cite: 1, 2]. Como embaralhar os pontos não muda a modulação, a rede densa se perde tentando decorar sequências que não existem[cite: 1, 2].

O **IQ ordenado** resolve essa bagunça da ordem e chega a 100% de acerto quando o sinal está limpo (20 dB), mas perde força em 8 dB porque o ruído borra os "degraus" que a ordenação cria e destrói o par original I/Q de cada ponto[cite: 1, 2].

Os **cumulantes** confirmaram minha hipótese inicial: com poucos símbolos (32), a conta estatística de 4ª ordem fica instável, mas com 512 símbolos o modelo bate **99,8% de acerto no ruído (8 dB)**[cite: 1, 2]. Como o método entrega só 4 números para a rede em vez de 1.024 valores brutos, o gasto de processamento cai drasticamente[cite: 1, 2].

<img width="690" height="440" alt="acuracia_vs_snr" src="https://github.com/user-attachments/assets/c39cdf1a-3ab8-4c93-a0bc-264429bab932" />


## Como rodar o código

O script foi feito para rodar direto no Google Colab, mas funciona localmente em Python instalando as bibliotecas básicas:

```bash
pip install numpy scikit-learn matplotlib
