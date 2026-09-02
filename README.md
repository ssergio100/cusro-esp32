# ESP32 do Zero

**Eletrônica e Programação para Pessoas Comuns**

Um curso passo a passo para quem está começando do zero absoluto em
eletrônica, microcontroladores e programação.

## Para quem é este curso

Este curso foi feito para **pessoas comuns**:

- que nunca mexeram com eletrônica;
- que nunca programaram;
- que não sabem o que é um microcontrolador;
- que só querem entender o que estão fazendo, em vez de copiar código sem
  pensar.

Nada aqui presume que você sabe o que é um resistor, um pino GPIO ou uma
função em C++. A cada aula, um conceito novo por vez, explicado em
linguagem simples.

## Nossa filosofia

A prioridade deste curso **não é quantidade de conteúdo**. É o
**entendimento**.

Queremos que você:

- saiba *por que* cada coisa funciona;
- consiga ligar circuitos simples sem medo de estragar a placa;
- leia e entenda cada linha do código que usa;
- reconheça erros comuns e saiba corrigi-los.

Escrevemos como quem está ensinando você na bancada, ao lado, devagar.

## O que você vai precisar

- Um **ESP32** (uma placa comum de desenvolvimento)
- Um **cabo USB** para ligar a placa no computador
- Um **computador com Linux/Ubuntu**
- Algumas peças simples (LEDs, botões, resistores, fios) — vamos apresentar
  cada uma no momento certo

## Como usar este curso

Comece pela **Aula 00**. Cada aula termina com o "Próximo passo",
preparando você para a seguinte. Siga na ordem — as aulas constroem uma em
cima da outra.

## Índice — pule para o que te interessa

As aulas se constroem uma sobre a outra, então a **ordem** continua sendo a
melhor forma de aprender do zero. Mas se você já conhece alguma parte e quer
**ir direto ao assunto**, aqui está o caminho rápido, agrupado por tema.

**Começando do zero absoluto**

- [Aula 00 — Preparando a bancada](aulas/00-preparando-a-bancada/README.md) —
  conhecer a placa, instalar o Arduino IDE e rodar o primeiro teste.
- [Aula 01 — Nosso primeiro programa](aulas/01-primeiro-programa/README.md) —
  o "Blink" explicado linha por linha desde o começo.

**Entradas e saídas (os "braços" da placa)**

- [Aula 02 — Entendendo os pinos do ESP32](aulas/02-entendendo-os-pinos/README.md)
- [Aula 03 — Lendo um botão](aulas/03-lendo-um-botao/README.md)
- [Aula 04 — Fazendo mais de uma coisa ao mesmo tempo](aulas/04-fazendo-mais-de-uma-coisa/README.md)
- [Aula 05 — Controlando o brilho de um LED com PWM](aulas/05-controle-de-intensidade/README.md)
- [Aula 06 — Lendo sinais analógicos](aulas/06-lendo-sinais-analogicos/README.md)

**Vendo o que o ESP32 "pensa"**

- [Aula 07 — Monitor Serial: a janela do ESP32](aulas/07-monitor-serial/README.md)

**Conectando componentes (barramentos)**

- [Aula 08 — I2C sem mistério](aulas/08-i2c-sem-misterio/README.md)
- [Aula 09 — Usando um dispositivo I2C](aulas/09-usando-um-dispositivo-i2c/README.md)
- [Aula 10 — SPI](aulas/10-spi/README.md)
- [Aula 11 — Display gráfico ST7789](aulas/11-display-grafico/README.md)

**Internet e comunicação (as "coisas conectadas" — IoT)**

- [Aula 12 — Wi‑Fi](aulas/12-wi-fi/README.md)
- [Aula 13 — Primeiro servidor Web](aulas/13-primeiro-servidor-web/README.md)
- [Aula 14 — ESP32 conversando com servidores](aulas/14-esp32-conversando-com-servidores/README.md)
- [Aula 15 — MQTT](aulas/15-mqtt/README.md)
- [Aula 21 — Bluetooth e BLE](aulas/21-bluetooth/README.md)

**Tempo, precisão e organização do código**

- [Aula 16 — Interrupções](aulas/16-interrupcoes/README.md)
- [Aula 17 — Timers](aulas/17-timers/README.md)

**Guardando dados e áudio**

- [Aula 18 — Memória do ESP32](aulas/18-memoria/README.md)
- [Aula 19 — Salvar configurações (NVS / Preferences)](aulas/19-salvando-configuracoes/README.md)
- [Aula 20 — Arquivos com LittleFS](aulas/20-arquivos/README.md)
- [Aula 22 — Áudio digital com I2S](aulas/22-audio-digital/README.md)

**Juntando tudo**

- [Aula 23 — Projeto integrado](aulas/23-projeto-integrado/README.md) — uma
  aplicação final que usa várias aulas juntas.

## Estrutura do repositório

```
curso-esp32/
├── README.md          (este arquivo)
├── AGENTS.md          (regras do projeto para agentes de IA)
├── roteiro.md         (roteiro completo do curso)
├── aulas/             (uma pasta por aula, cada uma com seu README)
├── exemplos/          (códigos de exemplo)
├── imagens/           (imagens e diagramas)
├── projetos/          (projetos intermediários e final)
└── recursos/          (materiais de apoio)
```

## Comece agora

Vá para [Aula 00 — Preparando a bancada](aulas/00-preparando-a-bancada/README.md).
