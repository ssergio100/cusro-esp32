# Aula 09 — Usando um dispositivo I2C

## O que vamos fazer

Na Aula 08 aprendemos a **encontrar** dispositivos I2C e seus endereços.
Nesta aula vamos **usar um dispositivo I2C de verdade** para fazer algo útil:
vamos mostrar **texto em um pequeno display OLED** (SSD1306).

> O **display OLED SSD1306** é o nosso **exemplo** de dispositivo I2C — é
> barato e muito comum, por isso usamos ele. Mas a **ideia central** aqui é
> **usar um dispositivo I2C com uma biblioteca**, então **serve qualquer
> dispositivo I2C que você tiver** (o método é o mesmo: achar o endereço,
> instalar a biblioteca certa e usar). Não compre nada especial — use o que
> já tem ou o que for mais fácil para você encontrar.

Por que isso é útil? Um display dá ao ESP32 um jeito de **mostrar informações
para você** sem precisar do computador. É o primeiro passo para fazer um
"painel" ou um dispositivo compacto. E, de quebra, você vai usar uma
**biblioteca** pela primeira vez — um recurso essencial em todos os próximos
capítulos.

## O que você vai aprender

No final desta aula, você vai entender:

- como **identificar o endereço** de um dispositivo com o scanner (Aula 08);
- como **instalar uma biblioteca**;
- **o que é uma biblioteca**;
- como **inicializar** o dispositivo;
- como **escrever uma informação simples** (texto no display);
- como **mostrar o resultado**;
- que a **biblioteca esconde os detalhes** do protocolo;
- um **erro comum**: o **endereço errado**;
- um pequeno **desafio**.

## O que é uma biblioteca

Uma **biblioteca** (ou *library*) é um **conjunto de códigos prontos** que
outra pessoa escreveu, que você pode **reutilizar** para não precisar
reinventar tudo.

Pense nela como um **livro de receitas**: você não precisa saber fazer cada
ingrediente do zero; segue a receita pronta. No nosso caso, a biblioteca do
SSD1306 já sabe **todos os detalhes** de conversar com o display via I2C —
você só precisa "seguir a receita".

Isso é importante porque são **muitos** detalhes técnicos embaixo do pano
(sequências de comandos, tempos, registros). A biblioteca **esconde isso**,
oferecendo **funções simples** como "mostre este texto aqui".

## Ligação

Vamos usar o **display OLED SSD1306** (módulos pequenos, de 0,96", são os
mais comuns). A ligação é igual à da Aula 08 — é I2C:

```
ESP32             Display SSD1306
3V3   ──────────── VCC (energia)
GND   ──────────── GND (retorno)
GPIO SDA ──────── SDA (dados)
GPIO SCL ──────── SCL (relógio)
```

> **Verifique o pinout da sua placa** para os GPIOs de SDA e SCL (Aula 08).
> Em muitos ESP32 clássicos, SDA = 21 e SCL = 22; em outras variantes, mudam.

## Antes de ligar

- **Desligue o cabo USB** antes de montar o circuito; reconecte depois.
- **Não aplique mais de 3,3 V** em nenhum pino de sinal.
- Garanta o **GND comum** entre a placa e o display.
- Seu módulo SSD1306 pode ter variações de resolução (0,96" = **128×64**)
  ou de endereço (`0x3C` ou `0x3D`). Vamos tratar os dois no código.

## Primeiro teste: o endereço

Antes de mais nada, **descubra o endereço** do seu display rodando o
**scanner I2C da Aula 08** com o display conectado.

**Anote o endereço** que aparecer (ex.: `0x3C`). Vamos usá-lo no código.

Se não aparecer nada, revise a ligação (energia, GND comum, SDA, SCL) antes
de seguir.

## Instalando a biblioteca

Para usar o display, instalamos a biblioteca no Arduino IDE (uma única vez):

1. No menu, vá em **Gerenciador de Bibliotecas** (ou *Library Manager*).
2. Pesquise por **"Adafruit SSD1306"**.
3. Instale a biblioteca **Adafruit SSD1306**.
4. Ela vai pedir para instalar as **dependências** (outras bibliotecas que
   ela usa, como a **Adafruit GFX**). Aceite.

> A biblioteca **Adafruit GFX** contém as funções de desenho (texto, formas)
> que o SSD1306 usa. Instale-a também quando for oferecida.

## Código

Abra um sketch novo e digite:

```cpp
// Aula 09 — Texto no display OLED SSD1306

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define TELA_LARGURA 128
#define TELA_ALTURA 64

Adafruit_SSD1306 display(TELA_LARGURA, TELA_ALTURA, &Wire, -1);

void setup() {
  Serial.begin(115200);

  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("Falha ao iniciar o display. Verifique o endereco.");
    while (true) {
    }
  }

  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(10, 20);
  display.println("Ola!");
  display.display();
}

void loop() {
}
```

## Entendendo o código

Vamos ler as partes importantes, devagar.

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
```

Incluímos as bibliotecas. A Wire cuida do I2C; a GFX traz as funções de
desenho; a SSD1306 entende o display. É isso que "manda as receitas" para o
nosso uso.

```cpp
#define TELA_LARGURA 128
#define TELA_ALTURA 64
```

**`#define`** cria **nomes para números fixos** (a largura e a altura da tela
em pixels). O display 0,96" tem 128 colunas por 64 linhas.

```cpp
Adafruit_SSD1306 display(TELA_LARGURA, TELA_ALTURA, &Wire, -1);
```

**Criamos um "objeto"** chamado `display`, que representa o nosso display.
Não precisa entender objetos agora — basta saber que `display` é o "controle
remoto" que usamos para mandar comandos ao painel. Os argumentos dizem o
tamanho da tela e qual barramento usar (`&Wire` = I2C).

```cpp
if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
  Serial.println("Falha ao iniciar o display. Verifique o endereco.");
  while (true) {
  }
}
```

**`display.begin(...)`** tenta **inicializar** o display. O **`0x3C`** aqui é
o **endereço** que você anotou no scanner — **se estiver errado, falha**!

A ideia do `if (!display.begin(...))`: `!` significa "negação" (Aula 04). Se
a inicialização **falhar** (`!` inverte o resultado), entramos no `if`,
mostramos um aviso e **paramos** com o `while (true)` que não faz nada (fica
preso para sempre). Assim você sabe na hora que algo está errado.

> Se o seu endereço for outro (ex.: `0x3D`), **troque o `0x3C`** no código.

Agora vem o que o display mostra:

```cpp
display.clearDisplay();
```

**Limpa** a tela — apaga tudo o que estava antes.

```cpp
display.setTextSize(2);
```

Define o **tamanho do texto** (2 = o dobro do tamanho básico).

```cpp
display.setTextColor(SSD1306_WHITE);
```

Define a **cor** do texto. Este display é monocromático (um tom só), então
branco = "acender os pixels".

```cpp
display.setCursor(10, 20);
```

Define **onde** o texto começa (a posição na tela, em pixels: coluna 10,
linha 20).

```cpp
display.println("Ola!");
```

Coloca o texto **"na memória do display"** (ainda não aparece de fato).

```cpp
display.display();
```

**`display()`** **mostra** na tela o que foi desenhado. Este passo é
obrigatório — sem ele, nada aparece! Pense em desenhar num rascunho e só
depois imprimir: o `display.display()` é o botão "imprimir".

## O que deve acontecer

Depois de enviar, o display deve mostrar o texto **"Ola!"**. O texto fica
**fixo** na tela (é um desenho) — ele não se move nem pisca.

Se aparecer **"Falha ao iniciar o display"** no Monitor Serial, o problema é
quase sempre o **endereço** (reveja o scanner) ou a **ligação** (fios,
energia, GND comum).

## Se não funcionar

- **"Falha ao iniciar o display"**: confira o **endereço** (rode o scanner e
  use o valor exato). Verifique SDA/SCL corretos e o **GND comum**. Confirme
  se o display está alimentado (VCC em 3V3).
- **A tela fica em branco**: confirme se chamou **`display.display()`** no
  final. Sem isso, nada aparece.
- **Nada compila**: confira se as bibliotecas **Adafruit SSD1306** e
  **Adafruit GFX** estão instaladas.
- **O texto aparece pequeno ou mal posicionado**: ajuste `setTextSize` e
  `setCursor`.
- **Imagem "fantasma"/duplicada**: pode ser fiação com má qualidade ou falta
  de GND. Confira os fios.

## Experimente você

Agora, o desafio. Personalize seu display:

1. **Mude o texto**: troque "Ola!" por uma frase sua e reajuste o `setCursor`
   para centralizar melhor. (Dica: textos de tamanho 2, para caber bem em
   128×64, têm no máximo ~8 caracteres por linha.)
2. **Mostre a leitura de um sensor**: combine com a Aula 06 — leia o
   potenciômetro (`analogRead`) e mostre o valor **no display** em vez de só
   no Serial. Use `display.println(valor)` com o número em uma variável.
   Lembre de limpar a tela e chamar `display.display()` a cada atualização.
3. **Explore o endereço errado**: troque o `0x3C` por um número que não é o
   seu (ex.: `0x00`) e observe a mensagem de erro. Depois volte ao correto.
   Isso te ajuda a **reconhecer** esse erro no futuro.

## O que aprendemos

Nesta aula:

- uma **biblioteca** é código pronto que reutilizamos ("livro de receitas");
- para usar um dispositivo I2C, primeiro **descobrimos o endereço** com o
  scanner;
- **instalamos** a biblioteca Adafruit SSD1306 e suas dependências;
- **inicializamos** o display com `display.begin()` e o **endereço**;
- escrevemos texto com **`setTextSize`, `setTextColor`, `setCursor`,
  `println`** e **`display.display()`** (que de fato mostra na tela);
- a biblioteca **esconde os detalhes** do protocolo I2C;
- o **endereço errado** é o erro mais comum — o `if` de verificação nos
  avisa.

## Próximo passo

O I2C é ótimo para dispositivos simples e muitos na mesma linha. Mas,
para displays e periféricos que precisam de **velocidade**, existe outro
barramento: o **SPI**, que usa **mais fios** — e é o que vamos usar para o
grande display gráfico da Aula 11.

Nos vemos na [Aula 10](../10-spi/README.md).
