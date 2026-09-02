# Aula 22 — Áudio digital com I2S

## O que vamos fazer

Nosso ESP32 já conversou por fios, pela internet e sem fio. Falta um tipo
especial de dado: **áudio**. Nesta aula o ESP32 vai **tocar um som** —
um tom simples — através de um pequeno **alto‑falante**, usando um módulo
chamado **I2S**.

Vamos entender a diferença entre **áudio analógico** e **digital**, conhecer
as linhas do **I2S** (BCLK, LRCLK/WS e DATA), montar um módulo simples
(como o **MAX98357A**) e fazer o ESP32 **tocar um tom**. É o primeiro passo
para projetos com música, alarmes ou voz.

## O que você vai aprender

No final desta aula, você vai entender:

- **áudio analógico** x **áudio digital**;
- o que é **I2S** e por que ele é próprio para áudio;
- os **sinais** do I2S: **BCLK, LRCLK/WS e DATA**;
- o papel do **amplificador** e do **alto‑falante**;
- como o **MAX98357A** simplifica tudo;
- como **gerar um tom** (um "bip") no ESP32;
- como **diagnosticar** quando não há som;
- o básico (só o básico!) de sample rate.

## Áudio analógico x digital

- **Analógico**: o som é um **sinal contínuo** de tensão, como a onda de um
  alto‑falante analógico. Varia "sem degraus".
- **Digital**: o som é transformado em **números** (amostras). O sinal fica
  em **degraus** — muitas amostras por segundo, tão próximas que os nossos
  ouvidos ouvem como som contínuo.

```
analógico:  ~ contínuo ~~~
digital:    _|-|_|-|_|- (degraus = amostras)
```

O ESP32 trabalha com **áudio digital**: envia **números** que representam o
som.

Toda essa conversão (som ↔ números) é chamada de **PCM**. Você não precisa
entender tudo — só saber que, para enviar som, o ESP32 envia uma sequência
de **amostras numéricas**.

## O que é I2S

**I2S** (*Inter‑IC Sound*) é um **padrão de comunicação para áudio digital**
entre chips. Ele é dedicado a som — diferente do I2C e do SPI que vimos
para dados em geral.

Pense no I2S como uma **"esteira de som"** entre o ESP32 e o
amplificador/fone: o ESP32 **empilha amostras** na esteira, e o receptor
**desempilha** e transforma em som.

```
ESP32 ──(amostras digitais)──> I2S ──> amplificador ──> alto-falante
```

## As linhas do I2S (o "trífio" de áudio)

O I2S usa (no mínimo) três linhas:

- **BCLK** — o **relógio de bits** (o "pulso" que marca cada pedaço de dado).
- **LRCLK / WS** — o **relógio de canal** (Word Select): diz se a amostra é
  do canal **esquerdo (L)** ou **direito (R)**.
- **DATA** — a linha **por onde viaja o som** (as amostras em si).

```
BCLK  = ritmo (marca os pedaços)
LRCLK = esquerda ou direita
DATA  = o som (amostras)
```

Não se preocupe com os detalhes elétricos. O importante: o ESP32 define
**quais pinos** usa para cada linha no código, e o **módulo** cuida do resto.

## O papel do amplificador e do alto‑falante

O sinal digital do ESP32 é **fraco e digital** — não dá para ligar direto num
alto‑falante. Por isso usamos:

- **Amplificador/DAC**: transforma o **digital** em **analógico** forte
  o bastante (é o que o **MAX98357A** faz).
- **Alto‑falante**: converte o sinal elétrico em **som**.

```
ESP32 (digital) → [MAX98357A: converte + amplifica] → alto-falante → som
```

O alto‑falante precisa de **energia** (o amplificador fornece) — lembre disso
na hora de ligar.

## Material necessário

- ESP32;
- módulo amplificador **MAX98357A** (I2S) ou similar;
- um **alto‑falante/fone** pequeno (8 Ω, com o módulo) ou o conector do
  módulo;
- cabos **jumper**;
- Monitor Serial (115200).

> O **MAX98357A** é um módulo bem comum e barato para I2S: já tem o
> amplificador e a conversão embutidos. Você só conecta 3 fios de dados
> (BCLK, LRC, DIN) + alimentação e liga um alto‑falante.

## Ligação (MAX98357A)

O MAX98357A tem esses pinos de dados: **BCLK**, **LRC** (que é o LRCLK/WS) e
**DIN** (DATA), além de **VIN** (alimentação), **GND** e terminais de
alto‑falante.

Escolha **3 pinos livres do ESP32** para o BCLK, LRC e DIN. Os valores abaixo
são um exemplo comum — **confira o pinout da sua placa** e use pinos livres.

```
ESP32                MAX98357A
GPIO 26 ───────────── BCLK
GPIO 25 ───────────── LRC   (LRCLK)
GPIO 27 ───────────── DIN   (DATA)
3V3  ──────────────── VIN
GND  ──────────────── GND
        alto-falante ── terminal + / - do módulo (ou conector)
```

> ⚠️ **Alimentação e alto‑falante:** os terminais do alto‑falante vão ao
> módulo, **não** direto ao ESP32. E lembre da regra de **3,3 V/5 V**: este
> módulo comum trabalha com **3,3 V** (conecte no pino 3V3). Se um dia usar
> versões/módulos que **exijam 5 V**, será outro esquema. **Não invente**
> ligações — siga a ficha do seu módulo.

## Primeiro teste: tocar um tom (bip)

Vamos fazer o ESP32 tocar um **tom de 440 Hz** (o "Lá" de afinação) por um
segundo. Para isso usamos a biblioteca moderna **`ESP_I2S`** (que já vem no
ESP32 Arduino Core 3.x).

Precisamos configurar o **sample rate** — o número de amostras por segundo.
Um valor comum é **44 100 Hz** (padrão de CD), mas para um tom simples podemos
usar **16 000** para economizar. Quanto maior o sample rate, mais amostras
por segundo (som de melhor qualidade), mas mais processamento.

Geramos a onda do tom calculando, para cada amostra, o seno do ângulo e
enviando (+ altos e baixos) ao I2S:

```cpp
// Aula 22 — Tocar um tom simples com I2S (MAX98357A)

#include <ESP_I2S.h>
#include <math.h>

I2SClass I2S;

const int BCLK = 26;
const int LRC  = 25;
const int DIN  = 27;

const int SAMPLE_RATE = 16000;   // amostras por segundo
const int DURACAO_MS  = 1000;    // duracao do tom, em milissegundos
const int FREQUENCIA  = 440;     // Hz (nota La)

void setup() {
  Serial.begin(115200);

  I2S.setPins(BCLK, LRC, DIN, -1);       // bclk, ws(LRCLK), dout(DATA)
  bool ok = I2S.begin(I2S_MODE_STD, SAMPLE_RATE,
                      I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_STEREO);
  if (!ok) {
    Serial.println("Falha ao iniciar o I2S!");
    return;
  }
  Serial.println("I2S iniciado. Tocando tom...");

  tocarTom(FREQUENCIA, DURACAO_MS);

  Serial.println("Tom tocado. Fim.");
}

void tocarTom(int frequencia, int duracaoMs) {
  int totalAmostras = (SAMPLE_RATE / 1000) * duracaoMs;
  float passo = 2.0f * PI * frequencia / SAMPLE_RATE;

  for (int i = 0; i < totalAmostras; i++) {
    float seno = sin(i * passo);              // entre -1 e 1
    int16_t amostra = (int16_t)(seno * 1000); // vira numero de 16 bits

    // I2S em modo stereo: envia a mesma amostra nos dois canais
    int16_t frame[2] = { amostra, amostra };
    I2S.write(frame, sizeof(frame));
  }
}

void loop() {
}
```

### Entendendo o código

```cpp
#include <ESP_I2S.h>
```

Inclui o suporte moderno a **I2S** do ESP32 Arduino Core 3.x (a biblioteca
`ESP_I2S`).

```cpp
I2SClass I2S;
```

Cria um objeto `I2S` (nossa "esteira de som").

```cpp
I2S.setPins(BCLK, LRC, DIN, -1);
```

**`setPins(bclk, ws, dout, din)`** diz **quais pinos** usar: `BCLK`, `LRC`
(que é o WS) e `DIN` (o `dout` de dados de saída). O último valor `-1` = não
usamos entrada de dados.

```cpp
bool ok = I2S.begin(I2S_MODE_STD, SAMPLE_RATE,
                    I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_STEREO);
```

**`begin(modo, sampleRate, larguraBits, canais)`** liga o I2S:
- `I2S_MODE_STD` — modo padrão (o que o MAX98357A usa);
- `SAMPLE_RATE` — quantas amostras por segundo;
- `I2S_DATA_BIT_WIDTH_16BIT` — cada amostra tem **16 bits**;
- `I2S_SLOT_MODE_STEREO` — temos os canais **esquerdo e direito**.

```cpp
float passo = 2.0f * PI * frequencia / SAMPLE_RATE;
```

Calcula o "passo" da onda (quanto o ângulo avança a cada amostra, para
produzir a frequência desejada). É um detalhe matemático para gerar o tom.

```cpp
float seno = sin(i * passo);
int16_t amostra = (int16_t)(seno * 1000);
```

Para cada amostra, calculamos um valor de seno (entre -1 e 1) e transformamos
em um **número de 16 bits** (o `int16_t`), que é como o áudio digital é
representado aqui. É isso que o módulo recebe e vira som.

```cpp
int16_t frame[2] = { amostra, amostra };
I2S.write(frame, sizeof(frame));
```

Como o I2S está em **stereo**, enviamos a amostra **dos dois lados** (um
"frame" de 2 valores: esquerda e direita) com **`I2S.write(...)`**. Repetimos
isso até a duração do tom acabar.

### Um aviso sobre a API do I2S

**A API de áudio também mudou** ao longo das versões do framework.
As funções acima (`ESP_I2S`, `setPins`, `begin` com estes argumentos)
correspondem à **API do Arduino ESP32 Core 3.x**.

Se, na sua IDE, **não compilar**, é provável que sua versão seja antiga (Core
1.x/2.x), onde o nome/caminho era diferente (ex.: `driver/i2s.h` com
`i2s_config_t`) ou onde se usava outra biblioteca. **Nesse caso**, use o
**exemplo de "I2S" que vem com a sua instalação** (Arquivo → Exemplos →
I2S) e **adapte**. O **conceito** — configurar pinos, sample rate e `write`
das amostras — é o mesmo; muda a sintaxe. Sempre **verifique a versão da sua
IDE**.

## Sobre o sample rate (só o básico)

O **sample rate** é quantas amostras o som tem por segundo (`16000` = 16 mil
amostras por segundo). Quanto maior, mais fiel o som — mas mais dados a
processar.

Para a maioria dos projetos, valores como **16 000** (chamado de 16 kHz) ou
**44 100** (qualidade de CD) bastam. Não precisa ir mais fundo nesta aula.

## Se não funcionar (sem som)

- **Nada toca**: confira as **ligações** (BCLK, LRC, DIN e VIN/GND) e os
  **pinos** no código — eles precisam bater com o que ligou.
- **"Falha ao iniciar o I2S"**: revise os pinos e se não estão em uso por
  outra coisa. Confira o Serial (115200).
- **Som baixíssimo ou nenhum**: ajuste o **volume do módulo** (o MAX98357A
  tem um pino GAIN; verifique a ficha) e confira o **alto‑falante** e sua
  ligação aos terminais do módulo.
- **Estala em vez de tocar**: o tom com seno e `1000` deve funcionar; se
  estalar muito, confira alimentação estável (`3,3 V`).
- **Compila com erro na sua versão**: adapte a API conforme o **aviso
  acima** e o exemplo da sua IDE.

## Experimente você

Agora, os desafios:

1. **Mude a nota**: troque `FREQUENCIA` por outros valores (ex.: `523` = Dó,
   `659` = Mi) e ouça a diferença.
2. **Sirene**: varie a frequência ao longo do tempo no `loop()` (aumentando
   o valor em cada amostra) para criar um som "subindo" — uma sirene simples.
3. **Piscar junto**: faça um **LED** piscar na mesma frequência do tom
   (Aulas 04/05), sincronizando som e luz.

> Dica: para a sirene (desafio 2), mude o cálculo do passo aos poucos dentro
> do loop de amostras, subindo de uma frequência baixa para uma alta.

## O que aprendemos

Nesta aula:

- **áudio analógico** é contínuo; **digital** é em amostras numéricas;
- **I2S** é o padrão de comunicação de áudio digital;
- as linhas **BCLK, LRCLK/WS e DATA** e o papel de cada uma;
- o **amplificador (MAX98357A)** converte digital → analógico e reforça;
- geramos um **tom** gerando amostras com `sin()` e `I2S.write()`;
- o **sample rate** é só quantas amostras por segundo;
- a **API do I2S** varia com a versão — sempre adaptar pela sua IDE;
- cuidados com **alimentação e alto‑falante** (3,3 V).

## Próximo passo

Você já conhece quase todos os blocos do ESP32: entradas, saídas, tempo,
comunicação (fios, internet, sem fio) e até áudio. Falta **juntar tudo** em
um projeto real. Essa é a nossa última grande parada: o **projeto integrado**.

Nos vemos na Aula 23.