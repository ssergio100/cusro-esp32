# Aula 10 — SPI

## O que vamos fazer

Na Aula 08 vimos o I2C, que conecta **muitos dispositivos com poucos fios**.
Mas existe outro barramento muito comum: o **SPI**. Ele usa **mais fios**, em
troca de **mais velocidade** — e é por isso que é usado em **displays** e
periféricos que precisam atualizar rápido.

Nesta aula vamos **entender o SPI conceitualmente**: seus fios e a ideia por
trás deles. **Não vamos ligar nada ainda** — a Aula 11 traz o display de
verdade (ST7789), que usa SPI. Aqui, o objetivo é **entender para não se
perder lá**.

Por que isso é útil? O SPI está em displays, cartões SD, sensores rápidos e
muitos módulos. Entender o "cenário" agora faz o próximo capítulo ser só
"ligar fios e usar a biblioteca".

## O que você vai aprender

No final desta aula, você vai entender:

- o que diferencia o **SPI** do **I2C**;
- que o **SPI usa mais fios** (por isso pode ser mais rápido);
- o que significam **MOSI, MISO, SCLK e CS**, em linguagem humana;
- que **alguns dispositivos não usam MISO**;
- o papel do **CS** — "qual dispositivo estou chamando agora";
- o conceito de **barramento compartilhado**;
- sem precisar **decorar siglas** (só entender o papel de cada fio);
- uma preparação visual para o **ST7789** (Aula 11).

## Relembrando o I2C

No I2C, tudo passa por **dois fios** (SDA e SCL), e cada dispositivo tem um
**endereço** ("número da casa"). O ESP32 "chama" um endereço de cada vez. É
**simples** e usa **poucos fios** — ótimo para vários dispositivos pequenos
na mesma linha.

O SPI é um caminho diferente: **mais fios, mais velocidade, e nenhum
endereço** — em vez disso, um fio **individual de seleção** para cada
dispositivo.

## SPI usa mais fios

Enquanto o I2C usa 2 fios de dados, o SPI usa **pelo menos 3 ou 4**, cada um
com um papel específico. Os nomes são siglas em inglês, mas o **papel** de
cada um é simples. Vamos entender um por um.

### SCLK — o "metrônomo" (sincroniza tudo)

**SCLK** (*Serial Clock*) é o fio de **relógio**, como o SCL do I2C. Marca o
**ritmo** da comunicação. Ambos os lados usam esse ritmo para saber quando
"ler" ou "escrever" um bit.

Pense na música: sem um ritmo combinado, os músicos não tocam juntos. O
SCLK é esse ritmo.

### MOSI — o ESP32 manda dados

**MOSI** (*Master Out, Slave In* — "o mestre emite, o escravo recebe"). Não
decorize o nome; basta lembrar o **papel**: é o fio por onde o **ESP32 envia
dados** para o dispositivo.

```
ESP32 ──(MOSI)──> dispositivo
```

### MISO — o dispositivo responde

**MISO** (*Master In, Slave Out*) é o caminho **inverso**: por onde o
**dispositivo envia dados de volta** ao ESP32.

```
dispositivo ──(MISO)──> ESP32
```

**Importante:** **nem todo dispositivo usa MISO.** Muitos displays, por
exemplo, só **recebem** comandos (o ESP32 desenha a tela) e **não enviam**
dados de volta. Nesse caso, o fio MISO simplesmente **não é usado**.

### CS — "qual dispositivo estou chamando agora"

**CS** (*Chip Select* — "seleção do chip") é o fio mais importante para
entender o SPI com vários dispositivos.

Pense assim: no I2C, o ESP32 "chama pelo número da casa". No SPI, **não há
número** — em vez disso, cada dispositivo tem seu **próprio fio CS**. Quando
o ESP32 quer falar com um dispositivo, **puxa o CS daquele dispositivo para
baixo** (ativa), "dizendo": **"é você que eu quero falar agora."** Os outros
ficam em silêncio.

```
ESP32 ──CS1──> dispositivo 1   (ativo = "falo com você")
ESP32 ──CS2──> dispositivo 2   (inativo = "fique quieto")
ESP32 ──CS3──> dispositivo 3   (inativo)
```

Por isso, no SPI:

- **todos** os dispositivos **compartilham** os fios SCLK, MOSI (e MISO);
- mas cada um tem seu **próprio fio CS**.

## Barramento compartilhado

Assim como no I2C, no SPI vários dispositivos dividem **os mesmos fios de
dados**:

```
                ┌── dispositivo 1 (CS1)
ESP32 ────SCLK─┼── dispositivo 2 (CS2)
        ────MOSI─┤── dispositivo 3 (CS3)
        ────MISO─┘
```

O **SCLK, MOSI e MISO** são **compartilhados** entre todos. O que **muda** é
o **CS** — um fio exclusivo por dispositivo, que decide **quem está sendo
chamado**.

É como um corredor (SCLK/MOSI/MISO) por onde passa a conversa, e cada
dispositivo tem uma **porta com sino próprio** (CS). O ESP32 toca **um** sino
por vez.

## Por que usamos SPI para displays?

Displays precisam **atualizar muitos pixels** rapidamente (cada pixel é um
ponto da tela). Isso exige **muitos dados em pouco tempo** — ou seja,
**velocidade**. O SPI, com seus fios dedicados e sem a "negociação" de
endereços do I2C, costuma ser **mais rápido**, o que o torna ideal para
displays.

Por isso o **ST7789** (nosso display da Aula 11) usa **SPI** — e não I2C.

## Antes de ligar

Nesta aula **não há ligação** — é conceitual. Mas já vale a regra de ouro
para a próxima: no SPI do display, vamos usar fios de **sinal** (nunca acima
de 3,3 V) e sempre compartilhar o **GND** com a placa.

## Primeiro teste

Como não há circuito, nosso "teste" é **mental e visual**: olhar para a
tabela abaixo e **identificar o papel** de cada fio. Reserve 1 minuto.

```
SPI (resumo rápido)

SCLK  → ritmo (sincroniza todos)
MOSI  → ESP32 → dispositivo (dados de saída)
MISO  → dispositivo → ESP32 (dados de volta; nem todo usa)
CS    → "estou chamando este dispositivo agora" (um por dispositivo)
```

## Entendendo o cenário completo

Vamos juntar tudo com um exemplo simples de um **display SPI** (como o
ST7789 da próxima aula):

```
ESP32                 Display ST7789
GPIO SCLK ─────────── SCK   (ritmo)
GPIO MOSI ─────────── SDA   (dados que o ESP32 envia)  [nota: no display esse
                                                        pino costuma ser o "SDA"
                                                        do SPI, que é o MOSI]
GPIO CS  ──────────── CS    (seleção do display)
GPIO DC ───────────── DC    (comando ou dado — detalhe do display)
GND ───────────────── GND
3V3 ───────────────── VCC   (alimentação)
```

Cada display tem **alguns fios a mais** (como o **DC**), mas **os
conceitos** de SCLK, MOSI e CS continuam os mesmos que aprendemos aqui. Na
Aula 11 vamos ligar cada um e explicar o extra. Não precisa decorar nada —
só reconhecer os papéis.

> Nota: você pode ver o pino de dados de um display chamado de "SDA", mas
> nesse contexto ele é o **MOSI** do SPI (o ESP32 envia dados para o
> display). Não deixe a nomenclatura te confundir: o **papel** (ESP32 envia
> dados) é o que importa.

## Se não funcionar

Como não ligamos nada, não há "falha" de circuito aqui. Mas já anote os
**erros comuns do SPI** que veremos na Aula 11:

- **Fios trocados** (MOSI no lugar de SCLK etc.) — o display não responde ou
  mostra lixo;
- **CS errado ou faltando** — o display "não sabe" que está sendo chamado;
- **GND não compartilhado** — a comunicação falha;
- **velocidade alta demais** — dados ficam corrompidos (veremos como ajustar).

## Experimente você

Sem hardware, o desafio é **fixar os papéis**:

1. **Explique com suas palavras** por que o SPI pode ser mais rápido que o
   I2C (dica: mais fios dedicados, sem "chamar número de casa").
2. **Conte os fios**: diga quantos fios de **dados** o SPI usa e por quê
   (SCLK de ritmo + MOSI/MISO de dados) em comparação com o I2C (SDA/SCL).
3. **Pense no CS**: se você tivesse **3** displays SPI, quantos fios **CS**
   precisaria? (Resposta: 3 — um para cada — enquanto os outros fios seriam
   compartilhados.)

## O que aprendemos

Nesta aula:

- o **SPI** usa **mais fios** e, em troca, **mais velocidade** que o I2C;
- **SCLK** é o ritmo; **MOSI** leva dados do ESP32; **MISO** traz dados de
  volta (mas **nem todo dispositivo usa**);
- **CS** decide **qual dispositivo está sendo chamado agora** (um fio por
  dispositivo);
- vários dispositivos **compartilham** SCLK/MOSI/MISO, mas cada um tem seu
  **CS**;
- displays usam SPI por causa da **velocidade**;
- preparamos o terreno para o **ST7789** da Aula 11.

## Próximo passo

Agora que entendemos o SPI, vamos **ligar um display gráfico ST7789 de
verdade** e mostrar **texto, formas e uma imagem** na tela.

Nos vemos na Aula 11.
