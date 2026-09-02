# Aula 11 — Display gráfico ST7789

## O que vamos fazer

Na Aula 10 entendemos o SPI. Agora vamos **usar um display gráfico de
verdade**: o **ST7789** — aqueles displays TFT coloridos (de 1,3", 1,14" ou
2,0" etc.) muito comuns em projetos. Vamos ligá-lo via SPI e fazer ele
mostrar **texto, formas e uma cor de fundo** (e, de quebra, uma "imagem"
simples).

Por que isso é útil? É um salto grande de visual: em vez de um display de
poucos caracteres (OLED da Aula 09), temos uma **tela colorida** cheia de
pixels, ideal para painéis, jogos pequenos e interfaces.

## O que você vai aprender

No final desta aula, você vai entender:

- o display e seu **controlador** (ST7789);
- como **identificar a resolução** do seu módulo;
- como **ligar via SPI** (SCLK, MOSI, CS, DC + energia/backlight);
- sobre **alimentação** e **backlight**;
- como **instalar a biblioteca** adequada;
- como **inicializar** o display;
- como **limpar** a tela;
- como **mostrar texto**;
- como **desenhar uma forma** simples;
- como **mostrar uma imagem** simples;
- sobre **orientação/rotação**;
- como **diagnosticar** tela branca ou preta.

## O display e seu controlador

O **ST7789** é o "cérebro" dentro de alguns displays TFT coloridos. Ele é o
**controlador** que transforma os comandos que enviamos em pixels acesos na
tela.

A placa de vidro em si (o "módulo") pode vir em **tamanhos e resoluções
diferentes**, mas o controlador é o mesmo. Por isso, o **primeiro passo** é
descobrir a **resolução do seu módulo** — o número de pixels (largura ×
altura) que a tela tem.

Resoluções comuns de módulos ST7789:

- **240 × 240** pixels (1,3" — muito comum);
- **240 × 135** pixels (1,14" — formato "redondo/telão");
- **240 × 320** pixels (2,0");
- etc.

> 📷 **Olhe o vendedor/descrição do seu display** para saber a resolução, ou
> leia o texto impresso atrás do módulo. Anote **largura × altura** — vai
> usar no código.

## Ligação

O ST7789 usa **SPI**. Os fios típicos de um módulo são:

```
ESP32                Display ST7789
3V3   ────────────── VCC
GND   ────────────── GND
GPIO SCLK ────────── SCL (clock)
GPIO MOSI ────────── SDA (dados)  ← no display, o pino de dados é o MOSI
GPIO CS  ─────────── CS
GPIO DC ──────────── DC (comando/dado)
GPIO RST ────────── RST (reset, às vezes opcional)
GPIO (pode ter) ──── BL (backlight)
```

Os pinos variam entre módulos. Os **fundamentais** são: **VCC, GND, SCL
(SCLK), SDA (MOSI), CS e DC**. Alguns módulos têm um pino **RES/RST** (reset)
e um pino **BL/backlight** (luz de fundo).

Sobre os GPIOs SPI do ESP32:

- O **MOSI** e o **SCLK** costumam ter **pinos padrão de hardware SPI**. Em
  muitos ESP32 clássicos: **SCLK = GPIO 18** e **MOSI = GPIO 23**. **Consulte
  o pinout da sua placa (Aula 02)** — variam conforme a variante.
- **CS** e **DC** podem ser **quaisquer GPIOs** livres que você escolher
  (vamos usar GPIO 5 e GPIO 2, mas ajuste ao que sua placa tiver livre).

> ⚠️ **Alimentação e backlight:** o módulo deve receber **3,3 V no VCC**, e
> o **GND compartilhado** com a placa. O pino **BL** (backlight) controla a
> luz de fundo; se existir, ligue-o a **3V3** (luz acesa) ou a um GPIO (para
> poder ligar/desligar). **Não** aplique 5 V nos sinais.

## Antes de ligar

- **Desligue o cabo USB** antes de montar; reconecte depois.
- **Não aplique mais de 3,3 V** em nenhum pino de sinal.
- Garanta o **GND comum**.
- Não confie em um único diagrama de internet: **confira os nomes dos pinos
  do SEU módulo** (SDA/SCL no display são, respectivamente, MOSI/SCLK no
  ESP32). Rotular errado é a causa mais comum de tela branca.

## Primeiro teste: instalar a biblioteca e inicializar

Vamos instalar a biblioteca **Adafruit ST7735 and ST7789** (no Gerenciador de
Bibliotecas, pesquise por **ST7789** e instale; aceite as dependências,
incluindo **Adafruit GFX**, se pedirem).

Depois, o primeiro programa: **inicializar o display e mostrar uma cor**.

```cpp
// Aula 11 — Inicializando o ST7789 e mostrando uma cor

#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>
#include <SPI.h>

#define PIN_CS   5
#define PIN_DC   2
#define PIN_RST  -1

#define LARGURA  240
#define ALTURA   240

Adafruit_ST7789 tela(PIN_CS, PIN_DC, PIN_RST);

void setup() {
  Serial.begin(115200);

  tela.init(LARGURA, ALTURA);
  tela.fillScreen(ST77XX_BLUE);
  Serial.println("Display inicializado!");
}

void loop() {
}
```

### Entendendo o código

```cpp
#include <Adafruit_ST7789.h>
```

Inclui a biblioteca do ST7789 (que também usa a GFX e a SPI por baixo).

```cpp
#define PIN_CS   5
#define PIN_DC   2
#define PIN_RST  -1
```

Nomeamos os pinos (Aula 05, `#define`). **CS** é o "seleciona este
dispositivo" (Aula 10); **DC** decide se o que enviamos é um **comando** ou
um **dado**; **RST** é o reset — use `-1` se o seu módulo não tiver esse
pino (ou conecte a um GPIO e coloque o número).

```cpp
Adafruit_ST7789 tela(PIN_CS, PIN_DC, PIN_RST);
```

Criamos o "controle remoto" do display, chamado `tela` (como fizemos com o
OLED na Aula 09, mas agora para este display).

```cpp
tela.init(LARGURA, ALTURA);
```

**`init(largura, altura)`** **inicializa** o display com a **resolução** do
seu módulo. Se a resolução estiver errada, a tela pode ficar com aparência
estranha ou deslocada.

```cpp
tela.fillScreen(ST77XX_BLUE);
```

**`fillScreen(cor)`** pinta a **tela toda** de uma cor. `ST77XX_BLUE` é uma
constante de cor pronta (azul). Se a tela **ficar azul**, tudo funcionou!

## Texto e formas

Agora vamos usar as funções de desenho (que vêm da GFX, igual ao OLED da
Aula 09):

```cpp
// Aula 11 — Texto e formas no ST7789

#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>
#include <SPI.h>

#define PIN_CS   5
#define PIN_DC   2
#define PIN_RST  -1

#define LARGURA  240
#define ALTURA   240

Adafruit_ST7789 tela(PIN_CS, PIN_DC, PIN_RST);

void setup() {
  Serial.begin(115200);

  tela.init(LARGURA, ALTURA);

  tela.fillScreen(ST77XX_BLACK);

  tela.setTextSize(3);
  tela.setTextColor(ST77XX_WHITE);
  tela.setCursor(20, 20);
  tela.println("Ola!");

  tela.fillRect(30, 80, 60, 60, ST77XX_RED);
  tela.fillCircle(180, 110, 30, ST77XX_GREEN);

  Serial.println("Desenhado!");
}

void loop() {
}
```

### Entendendo as partes novas

Conceitos repetidos da Aula 09 (GFX): `fillScreen` limpa/pinta o fundo,
`setTextSize` define o tamanho do texto, `setTextColor` a cor, `setCursor` a
posição do texto, `println` coloca o texto e `fillRect`/`fillCircle` desenham
formas.

- **`fillRect(x, y, largura, altura, cor)`** desenha um **retângulo
  preenchido**. Os dois primeiros números são a posição do canto superior
  esquerdo; os dois seguintes, o tamanho. Aqui: um quadrado vermelho perto do
  topo esquerdo.
- **`fillCircle(x, y, raio, cor)`** desenha um **círculo preenchido**: centro
  em (x, y), com um certo raio. Aqui: um círculo verde.

> **Nota importante:** no ST7789, **não** precisamos de um passo
> "`display.display()`" como no OLED — cada comando de desenho já **vai
> direto para a tela** assim que é chamado. Então basta as funções acima.

Um detalhe: no exemplo mostramos apenas "Ola!". O ST7789 tem lugar de sobra
para **conteúdo maior** — você pode aumentar o texto ou desenhar mais
elementos.

## Orientação e rotação

Dependendo de como você segura o display, os textos podem aparecer "de lado"
ou "de cabeça para baixo". Existe uma função para girar a tela:

```cpp
tela.setRotation(1);
```

**`setRotation(número)`** gira a tela em passos de 90°. Os valores vão de 0 a
3 (0, 90°, 180°, 270°). Experimente cada um até a orientação ficar como você
quer. Depois que gira, os eixos x e y **mudam** — é comum precisar ajustar as
coordenadas do `setCursor`/`fillRect`.

## Mostrando uma "imagem" simples

Uma imagem de verdade (foto) precisaria de muitos dados — assunto da Aula 20
(LittleFS) e da Aula 18 (memória). Aqui, vamos mostrar uma "imagem" simples
feita de **formas** — como um desenho:

```cpp
tela.fillScreen(ST77XX_BLACK);
tela.fillRect(60, 40, 120, 80, ST77XX_BROWN);   // casa
tela.fillTriangle(50, 40, 120, 10, 190, 40, ST77XX_RED); // telhado
tela.fillRect(105, 70, 30, 50, ST77XX_YELLOW);  // porta
```

- **`fillTriangle(x1,y1, x2,y2, x3,y3, cor)`** desenha um **triângulo**
  preenchido a partir de três pontos.

Essas formas juntas desenham uma **casinha na tela**. Com isso você já vê que
qualquer "imagem" na tela é só uma combinação de pontos e formas.

## O que deve acontecer

- **Primeiro teste**: a tela fica **inteira azul**.
- **Texto e formas**: fundo preto, o texto "Ola!", um **quadrado vermelho** e
  um **círculo verde**.
- **Imagem simples**: a **casinha** (quadrado marrom, telhado vermelho em
  triângulo, porta amarela).

Cada passo é visível e verificável — ótimo para confirmar que está tudo certo.

## Se não funcionar

- **Tela totalmente branca ou preta**: o mais comum é **fiação errada**.
  Confira SCL (SCLK), SDA (MOSI), CS, DC e o **GND comum**. Confira também o
  **backlight**: se o pino BL existir, ele precisa de energia (3V3 ou GPIO
  alto) — sem isso, a tela parece apagada mesmo ligada.
- **A tela liga, mas mostra imagem deslocada/estranha**: a **resolução**
  (`LARGURA`/`ALTURA`) do `init()` pode estar errada para o seu módulo, ou
  falta `setRotation`. Ajuste.
- **Texto aparece de lado**: use `setRotation(0..3)` até ficar certo.
- **Nada aparece e há um cheiro/calor**: desligue na hora. Provavelmente
  ligou alimentação errada (p.ex., 5 V). Nunca aplique 5 V nos sinais.
- **Compila, mas sem imagem**: confira se os pinos CS/DC não conflitam com
  outros usos da sua placa e se o MOSI/SCLK são os do hardware SPI correto.

## Experimente você

Agora, os desafios:

1. **Troque as cores**: experimente outras constantes de cor (ex.:
   `ST77XX_RED`, `ST77XX_GREEN`, `ST77XX_YELLOW`, `ST77XX_WHITE`) e veja como
   a tela muda.
2. **Gire a tela**: tente `setRotation(0)`, `1`, `2`, `3` e veja como o
   desenho gira. Ajuste as coordenadas quando girar.
3. **Desenhe algo seu**: use `fillRect`, `fillCircle` e `fillTriangle` para
   desenhar uma segunda figura (ex.: um rosto, um carrinho ou uma flor).
4. **Mostre um valor**: combine com a Aula 06 (potenciômetro) e mostre a
   leitura **na tela** em vez de só no Serial — assim você tem um "painel"
   de verdade.

## O que aprendemos

Nesta aula:

- o **ST7789** é o controlador de displays TFT coloridos, e o **módulo** pode
  ter **várias resoluções** (240×240 etc.);
- a ligação usa **SPI** (SCLK, MOSI, CS, DC) + **VCC/3V3, GND** e, se houver,
  **BL** (backlight) e **RST**;
- usamos a biblioteca **Adafruit ST7735 and ST7789** com **`tela.init(largura,
  altura)`** e **`fillScreen`**;
- mostramos **texto** (GFX) e **formas** (`fillRect`, `fillCircle`,
  `fillTriangle`);
- **`setRotation(0..3)`** gira a tela;
- no ST7789 cada desenho já vai **direto à tela** (sem `display()` extra);
- diagnosticamos **tela branca/preta** (fiação, backlight, resolução).

## Próximo passo

Agora que temos uma tela colorida, vamos dar ao ESP32 o **superpoder de se
comunicar sem fio**: vamos conectá-lo ao **Wi‑Fi** e a uma rede.

Nos vemos na Aula 12.
