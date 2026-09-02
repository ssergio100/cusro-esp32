# Roteiro do curso

> **ESP32 do Zero — Eletrônica e Programação para Pessoas Comuns**

Roteiro completo proposto. Cada aula tem nota (`00` a `23`). As aulas serão
escritas uma a uma, seguindo as regras pedagógicas de `AGENTS.md`.

## Aulas

- **00 — Preparando a bancada**
  - O que é ESP32
  - Microcontrolador versus computador
  - USB, alimentação, 3,3 V e GND
  - O que são GPIOs
  - Cuidados para não danificar a placa
  - Arduino IDE
  - Selecionar placa e porta
  - Monitor serial

- **01 — Nosso primeiro programa**
  - setup()
  - loop()
  - LED
  - digitalWrite()
  - HIGH e LOW
  - delay()

- **02 — Entendendo os pinos**
  - GPIO
  - entrada e saída
  - pinos que exigem cuidado
  - níveis de 3,3 V
  - GND comum

- **03 — Lendo um botão**
  - digitalRead()
  - INPUT
  - INPUT_PULLUP
  - botão pressionado
  - primeiro contato com lógica

- **04 — Fazendo mais de uma coisa**
  - problema do delay()
  - millis()
  - tarefas sem bloquear o programa

- **05 — Controle de intensidade**
  - PWM
  - controlar brilho de LED
  - conceito de duty cycle

- **06 — Lendo sinais analógicos**
  - ADC
  - potenciômetro
  - valores analógicos
  - limitações práticas do ADC do ESP32

- **07 — Monitor Serial como ferramenta de diagnóstico**
  - Serial.begin()
  - Serial.print()
  - acompanhar variáveis
  - descobrir problemas no programa

- **08 — I2C sem mistério**
  - SDA
  - SCL
  - endereço
  - vários dispositivos no mesmo barramento
  - scanner I2C

- **09 — Usando um dispositivo I2C**
  - escolher um periférico simples
  - biblioteca
  - endereço
  - leitura ou escrita

- **10 — SPI**
  - diferença básica entre SPI e I2C
  - MOSI
  - MISO
  - SCLK
  - CS

- **11 — Display gráfico**
  - ST7789
  - SPI
  - inicialização
  - texto
  - formas
  - imagens

- **12 — Wi-Fi**
  - conectar o ESP32
  - SSID
  - endereço IP
  - verificar conexão
  - reconexão básica

- **13 — Primeiro servidor Web**
  - HTTP de maneira simples
  - página servida pelo ESP32
  - navegador controlando um LED

- **14 — ESP32 conversando com servidores**
  - HTTP GET
  - APIs
  - JSON básico

- **15 — MQTT**
  - ideia de publish/subscribe
  - broker
  - tópicos
  - primeiro projeto conectado

- **16 — Interrupções**
  - por que existem
  - quando usar
  - quando não usar
  - exemplo simples

- **17 — Timers**
  - diferença entre millis(), timer e interrupção

- **18 — Memória**
  - RAM
  - Flash
  - PSRAM
  - por que memória acaba

- **19 — Salvando configurações**
  - Preferences/NVS
  - guardar valores após desligar

- **20 — Arquivos**
  - LittleFS
  - salvar e ler arquivos
  - imagens e páginas HTML

- **21 — Bluetooth**
  - visão geral
  - BLE
  - exemplo simples

- **22 — Áudio digital**
  - conceito de I2S
  - BCLK
  - LRCLK
  - DATA
  - exemplo simples

- **23 — Projeto integrado**
  - combinar vários recursos aprendidos

## Projetos intermediários

Ao longo do curso, incluímos pequenos projetos, como:

- botão controlando LED
- dimmer com potenciômetro
- scanner I2C
- display mostrando informações
- relógio NTP
- controle via navegador
- sensor enviando informações via Wi-Fi
- painel ST7789
- projeto usando LittleFS
- pequeno dispositivo IoT

## Projeto final

O aluno deve construir algo que combine:

- ESP32
- display
- entrada física
- Wi-Fi
- armazenamento
- comunicação com outro sistema
