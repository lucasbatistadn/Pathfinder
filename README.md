# Projeto Micromouse "Pathfinder"

Projeto desenvolvido na **PUC Minas - Coração Eucarístico**.

**Integrantes:**
* Arthur Victor Lopes de Melo Alves
* Leonardo de Souza Fernandes
* Lucas Batista Domingues Nogueira
* Sophia da Costa Fernandes

**Um robô autônomo (Micromouse) desenvolvido para o desafio Robô Challenge 2025 da PUC Minas.**

O "Pathfinder" é um robô compacto projetado para navegar, mapear e resolver um labirinto de forma 100% autônoma. Este repositório contém o firmware (código-fonte), os arquivos de design do chassi (STL) e a documentação do projeto.

## Contexto do Projeto

Este projeto foi desenvolvido por alunos da **PUC Minas**, com o objetivo de competir na categoria Micromouse do evento **Robô Challenge 2025**.

O desafio consiste em duas fases (embora este código inicial foque na exploração):
1.  **Fase de Exploração:** O robô entra no labirinto sem conhecimento prévio e deve mapear as paredes usando seus sensores.
2.  **Fase de "Speed Run":** Após encontrar o centro, o robô calcularia o caminho mais curto e o percorreria na maior velocidade possível.

## Funcionalidades Principais

* **Navegação Autônoma:** Capaz de tomar decisões em tempo real para navegar pelo labirinto.
* **Mapeamento de Labirinto:** Utiliza 3 sensores Sharp de distância (IR) para detectar paredes à frente e nas laterais.
* **Chassi Otimizado:** Estrutura 100% impressa em 3D, projetada para ser leve, ágil e acomodar todos os componentes.
* **Firmware em C++:** Código desenvolvido do zero para o microcontrolador ESP32.

---

## Hardware: Lista de Componentes

O design do hardware foi focado em baixo custo, agilidade e processamento eficiente.

| Componente | Modelo Específico | Propósito no Projeto |
| :--- | :--- | :--- |
| **Microcontrolador** | ESP32 (Placa/Módulo) | Cérebro do robô. Processa os sensores e o algoritmo de navegação. |
| **Sensor (Frente)** | 1x Sharp GP2Y0A21YK0F | Sensor analógico de distância (10-80cm) para detecção frontal. |
| **Sensores (Laterais)** | 2x Sharp GP2Y0A51SK0F | Sensores analógicos de distância (2-15cm) para alinhamento. |
| **Motores** | 2x Motor N20 (100:1 - 150RPM) | Fornecem a tração para as rodas. |
| **Driver de Motor** | 1x Ponte H L298N (Módulo) | Controla a direção e velocidade (PWM) dos dois motores N20. |
| **Alimentação** | 1x Bateria Li-Ion/LiPo 7.4V 2000mAh | Fornece energia principal para todo o sistema. |
| **Regulador de Tensão**| Módulo LM2596 (Step-Down) | Reduz os 7.4V da bateria para 5V/3.3V (para o ESP32 e sensores). |
| **Rodas** | 2x Roda de Borracha 42mm | Garantem boa aderência com o piso do labirinto. |
| **Apoio (Roda Boba)** | 1x Roda de Esfera Transferidora | Terceiro ponto de apoio para permitir curvas fáceis (caster). |
| **Chassi** | Design próprio (Impresso em 3D) | Estrutura principal que une todos os componentes. |

## Software, Firmware e Algoritmos

* **Linguagem:** C++
* **Ambiente/IDE:** [Arduino IDE / PlatformIO com VS Code]
* **Firmware:** O código-fonte principal está localizado em abaixo do arquivo README.md como codigofonte.

### Algoritmo de Navegação

Implementamos um algoritmo **Seguidor de Parede (Wall Follower)**, baseado na **"Regra da Mão Direita Adaptada"** (conforme a lógica na função `loop()`). Este método garante que o robô explore e encontre a saída de qualquer labirinto simples (sem "ilhas").

A lógica de decisão é a seguinte:
1.  O robô avança se o caminho estiver livre.
2.  Se a frente estiver bloqueada ( `distFrente < 10.0` ), o robô para e decide:
3.  **Prioridade 1:** Verifica se há um caminho livre à **direita**. Se sim, gira 90° à direita.
4.  **Prioridade 2:** Se não, verifica se há um caminho livre à **esquerda**. Se sim, gira 90° à esquerda.
5.  **Último Recurso:** Se estiver bloqueado na frente, direita e esquerda (beco sem saída), gira 180° (duas curvas de 90°).

### Controle de Movimento

O controle de movimento **não utiliza encoders** e é baseado em duas técnicas principais:

* **Controle de Alinhamento (Controlador P):** Durante o movimento para frente (função `alinharRobo()`), um **controlador Proporcional (P)** simples é usado. Ele lê os sensores laterais, calcula o erro (`distLatDir - distLatEsq`) em relação ao centro do corredor e ajusta a velocidade de cada motor (aumentando uma e diminuindo a outra) para manter o robô sempre centralizado.

* **Controle de Curvas (Temporizado):** As curvas de 90° e 180° (função `girar90()`) são executadas com base em **temporização (delays)**. O valor `const int TEMPO_GIRO` foi calibrado manualmente para que a rotação seja a mais precisa possível.

---

## Processo de Montagem e Construção

A montagem do Pathfinder foi um processo de prototipagem rápida, focada em integrar os componentes eletrônicos a um chassi personalizado.

### 1. Estrutura e Chassi
* O chassi foi modelado no software **Blender** e **impresso em 3D** com filamento **PLA**, garantindo uma base leve e sob medida.
* Os motores N20 e a roda boba de esfera foram fixados mecanicamente na base.

### 2. Montagem Eletrônica
* **Conexões Principais:** Utilizamos uma **protoboard** (breadboard) como ponto central para conectar o ESP32, os sensores e os módulos.
* **Soldagem:** Para garantir conexões elétricas robustas e que resistissem ao movimento, **utilizamos ferro de solda** para fixar os fios diretamente nos terminais dos **motores N20** e nos pinos da **Ponte H (L298N)**.
* **Fiação:** Jumpers de diversos tipos (macho-macho, macho-fêmea) foram usados para fazer as conexões entre os sensores, a protoboard e o ESP32.
* **Fixação:** **Fita dupla-face** e **cola quente** foram usadas estrategicamente para fixar componentes mais leves (como a própria protoboard e os sensores) no chassi, amortecendo vibrações e mantendo tudo no lugar.

### 3. Calibração Inicial (Hardware)
* Um passo crucial foi a **calibração do regulador de tensão** (Step-Down LM2596). Usando um multímetro, ajustamos o trimpot do módulo para garantir uma saída estável de **5V** para alimentar o ESP32 e os sensores a partir da bateria de 7.4V.
* O restante da calibração (sensores e movimento) foi feito via software.
