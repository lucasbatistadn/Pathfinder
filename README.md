# Projeto Micromouse "Pathfinder"

**Um robô autônomo (Micromouse) desenvolvido para o desafio Robô Challenge 2025 da PUC Minas.**

O "Pathfinder" é um robô compacto projetado para navegar, mapear e resolver um labirinto de forma 100% autônoma. Este repositório contém o firmware (código-fonte), os arquivos de design do chassi (STL) e a documentação do projeto.

## Contexto do Projeto

Este projeto foi desenvolvido como requisito para [Nome da Disciplina ou Matéria] da **PUC Minas**, com o objetivo de competir na categoria Micromouse do evento **Robô Challenge 2025**.

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

##Software, Firmware e Algoritmos

* **Linguagem:** C++
* **Ambiente/IDE:** [Arduino IDE / PlatformIO com VS Code]
* **Firmware:** O código-fonte principal está localizado em `src/main.cpp`.

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
