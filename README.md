# projeto-residencia-fpga-medidor-de-nivel-liquido
Sistema automático de bombeamento d'água desenvolvido com FPGA e linguagem de descrição de hardware Verilog.

# 💧 Controlador Automático de Reservatório em FPGA

![Verilog](https://img.shields.io/badge/Language-Verilog-blue)
![FPGA](https://img.shields.io/badge/Hardware-Lattice%20ECP5-orange)
![License](https://img.shields.io/badge/License-MIT-green)

Este projeto implementa um sistema de controle automático para dois reservatórios de água (Cisterna e Caixa Superior) utilizando uma FPGA. O sistema monitora níveis de água através de sensores, exibe o status em displays de 7 segmentos e controla uma bomba d'água de forma autônoma utilizando uma Máquina de Estados Finitos (FSM) do tipo Moore.

## 📸 Demonstração

![Foto do Projeto](https://via.placeholder.com/600x400?text=Insira+Foto+ou+GIF+da+Placa+Aqui)

## 📋 Funcionalidades

* **Monitoramento de Nível:** Leitura de 5 níveis de água (0%, 25%, 50%, 75%, 100%) para dois reservatórios independentes.
* **Controle Inteligente (Histerese):** Evita o acionamento intermitente da bomba ("bouncing"). A bomba liga quando a caixa superior está vazia e a inferior está cheia e desliga quando a caixa superior atinge 75% ou quando a caixa inferior estiver seca.
* **Proteção:** Impede o acionamento da bomba se a cisterna (caixa inferior) não tiver água suficiente (proteção contra marcha a seco).
* **Feedback Visual:**
    * **Displays 7-Seg:** Mostram o nível numérico atual (0 a 4) de cada caixa.
    * **LEDs de Status:** Indicam se a bomba está ligada (Verde).
* **Lógica de Sensor Invertida:** O sistema é projetado para sensores que operam em nível baixo ativo (0 = Água Presente / 1 = Sem Água).

## 🛠️ Hardware e Ferramentas

* **Placa FPGA:** Colorlight-i9 (Lattice ECP5 - LFE5U-25F)
* **Sensores de Nível (Customizados):**
    * Circuitos desenvolvidos manualmente utilizando componentes discretos: **Transistores (BJT), Resistores e LEDs**.
    * **Funcionamento:** Utilizam a condutividade da água para saturar o transistor, enviando nível lógico '0' para a FPGA e acendendo o LED correspondente para verificação visual imediata.
* **Linguagem:** Verilog (IEEE 1364).
* **Toolchain (Open Source):**
    * Yosys (Síntese)
    * Nextpnr (Place & Route)
    * openFPGALoader (Gravação)

## ⚙️ Arquitetura da FSM

O núcleo do projeto é uma Máquina de Estados (Moore) com 3 estados principais:

1.  **S_VAZIA (Idle/Proteção):**
    * Estado inicial ou de espera.
    * Válvula aberta.
    * Aguardando que a Cisterna (Inferior) tenha nível suficiente (100%) para permitir operação.
    * **Transição:** Se a Caixa Superior ficar **totalmente cheia** (Nível 100%), vai para `CHEIA`.
2.  **S_CHEIA (Pronto/Monitorando):**
    * A Cisterna inferior está cheia.
    * A válvula de entrada d'água é fechada.
    * **Transição:** Se a Caixa Superior ficar **totalmente vazia** (Nível 0% seco), vai para `S_ESVAZIANDO`.
3.  **S_ESVAZIANDO (Bombeando):**
    * A bomba é ligada (LED Verde).
    * A válvula de entrada d'água é aberta.
    * **Transição:** A bomba permanece ligada até que a Caixa Superior atinja **75%** OU se a Cisterna ficar **vazia** (< 25%).

## 🔌 Pinagem (Exemplo Colorlight-i9)

| Sinal | Descrição | Pino (Exemplo) | IO Type |
| :--- | :--- | :--- | :--- |
| `clk` | Clock (25MHz) | P3 | LVCMOS33 |
| `sensores_inf[0..4]` | Sensores Cisterna | P2 Header | Pull-up |
| `sensores_sup[0..4]` | Sensores Superior | P3 Header | Pull-up |
| `bomba` | Relé da Bomba | H18 | Output |
| `led_verde` | Indicador Bomba ON | G16 | Output |
| `display_inf` | 7-Seg Cisterna | P5 Header | Output |

*(Consulte o arquivo `.lpf` para a pinagem completa e exata)*

## 🚀 Como Executar

### Pré-requisitos
Instale o [OSS CAD Suite](https://github.com/YosysHQ/oss-cad-suite-build) para ter acesso a todas as ferramentas necessárias.

### Compilação e Gravação

1.  **Sintetizar e Criar Bitstream:**
    ```bash
    yosys -p "synth_ecp5 -top controlador_caixa_dagua -json hardware.json" controlador_caixa_dagua.v decodificador_nivel.v
    nextpnr-ecp5 --25k --package CABGA381 --json hardware.json --textcfg hardware.config --lpf pinout.lpf
    ecppack hardware.config hardware.bit
    ```

2.  **Gravar na Memória Flash (Persistente):**
    ```bash
    openFPGALoader -b colorlight-i9 -f hardware.bit
    ```

## 📂 Estrutura do Repositório

* `controlador_caixa_dagua.v`: Módulo principal contendo a FSM.
* `decodificador_nivel.v`: Módulo auxiliar para conversão Sensor -> Display.
* `pinout.lpf`: Arquivo de restrições de pinos (Lattice Preference File).

---
*Desenvolvido por Manoel Felipe, Paulo Gabriel e Yuri Gomes.*
