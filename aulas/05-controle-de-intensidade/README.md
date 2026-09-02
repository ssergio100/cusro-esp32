# Aula 05 — Controlando o brilho de um LED com PWM

## O que vamos fazer

Até agora o LED só fazia duas coisas: **aceso** ou **apagado** (HIGH ou LOW).
Nesta aula vamos fazer o LED **brilhar com intensidades diferentes** — apagado,
fracinho, médio, forte, e até "subir e descer" o brilho lentamente.

Por que isso é útil? Porque controlar **intensidade** é o que permite, por
exemplo, regular a luz de um ambiente, a cor de um LED RGB, a velocidade de
um motor ou o tom de um som. A ferramenta por trás disso se chama **PWM**.

E tem uma surpresa: o GPIO **não** consegue gerar "meia tensão". Veremos
como mesmo assim conseguimos dar a impressão de brilho variado.

## O que você vai aprender

No final desta aula, você vai entender:

- que um **GPIO digital** normalmente trabalha em **HIGH** ou **LOW**;
- que **PWM não é uma tensão analógica verdadeira**;
- que PWM **liga e desliga** o sinal muito rapidamente;
- que o **tempo ligado** determina o brilho percebido;
- o que é o **duty cycle**;
- como **alterar gradualmente** o brilho de um LED;
- a diferença básica entre **PWM** e uma **saída digital** comum.

## O problema: HIGH ou LOW

Lembra da Aula 01? Quando usamos `digitalWrite()`:

```
HIGH → LED ligado
LOW  → LED desligado
```

Só temos dois estados. Então surge uma pergunta natural:

**"Como fazemos o LED ficar com metade do brilho, se o GPIO só conhece HIGH
e LOW?"**

O GPIO **não** consegue dizer "3,3 / 2 = 1,65 volt". Ele só sabe ser ligado
ou desligado. Como resolver?

## O que é PWM

A solução é o **PWM** — sigla em inglês de *Pulse Width Modulation*
("modulação por largura de pulso"). Não precisa decorar o nome em inglês;
a ideia é o que importa.

O truque: em vez de definir **metade da tensão**, o ESP32 **liga e desliga o
sinal muito, muito rápido**. O olho humano não percebe a piscada tão rápida —
ele vê a **média**. Quanto mais tempo ligado, mais brilhante o LED parece.

Visualmente, imagine o tempo como barras. Cada barra é "o sinal ligado":

```
100%:  ████████████████   (sempre ligado → brilho máximo)

 50%:  ████____████____   (ligado metade do tempo → meio brilho)

 25%:  ██______██______   (ligado um quarto do tempo → brilho baixo)
```

O celular, o monitor e a luz LED da sua casa usam truques parecidos para
produzir tons diferentes de luz e de brilho.

> Importante desde já: **PWM não gera 1,5 V ou 2 V** de forma contínua. O
> GPIO continua alternando entre aproximadamente **0 V** (LOW) e **3,3 V**
> (HIGH) — só que, por estar ligando e desligando rápido, o *resultado
> aparente* é um brilho intermediário. É isso que chamamos de "PWM não é uma
> tensão analógica verdadeira".

## Duty cycle

A medida do "quanto tempo o sinal fica ligado" se chama **duty cycle**
("ciclo de trabalho"). É uma porcentagem do tempo em que o sinal fica
**ligado**:

```
100%  → ligado o tempo todo    (brilho máximo)
 50%  → ligado metade do tempo (brilho médio)
 25%  → ligado um quarto       (brilho baixo)
  0%  → desligado              (apagado)
```

Se você não está acostumado com porcentagem, pense assim: "por **cento**"
quer dizer "de cada 100 partes". Então **50%** = de cada 100 instantes
iguais, o sinal fica ligado em 50 deles. Quanto maior a porcentagem, mais
tempo ligado, mais brilho.

Não vamos aprofundar a **frequência** (quantas vezes por segundo isso se
repete) agora — só precisamos entender o *duty cycle* para controlar o
brilho.

## Ligação

Vamos usar um **LED externo com resistor**, como no fim da Aula 03:

```
ESP32
GPIO 2 ───── resistor ───── LED ───── GND
```

> Confirme o pino do seu LED no **pinout** (Aula 02). O valor do resistor
> depende do LED; um valor comum para 3,3 V fica na faixa de **220 a 330
> ohms**. O resistor protege o LED e o pino.

## Primeiro teste

Nosso primeiro teste é **controlar o brilho em níveis fixos**. O programa vai
mostrar, em sequência: apagado, brilho baixo, brilho médio e brilho máximo.

## Código

Abra um sketch novo e digite:

```cpp
// Aula 05 — Brilho do LED em níveis fixos

const int LED = 2;

void setup() {
  pinMode(LED, OUTPUT);
}

void loop() {
  analogWrite(LED, 0);     // apagado (0%)
  delay(1000);

  analogWrite(LED, 64);    // brilho baixo (25%)
  delay(1000);

  analogWrite(LED, 128);   // brilho médio (50%)
  delay(1000);

  analogWrite(LED, 255);   // brilho máximo (100%)
  delay(1000);
}
```

Depois, **compile e envie** (Aula 01) e observe o LED.

## Entendendo o código

A única novidade aqui é a função **`analogWrite()`**. Vamos entender o que
ela faz e seus números.

```cpp
pinMode(LED, OUTPUT);
```

Como antes, dizemos que o pino vai ser uma **saída**.

```cpp
analogWrite(LED, 0);
```

**`analogWrite(pino, valor)`** ativa o **PWM** naquele pino, com um certo
**duty cycle**. O **segundo número** é o "quanto" de brilho.

O intervalo vai de **0 a 255** (256 níveis, de 0 a 255). Nesse intervalo:

- **`0`** = **0%** = sempre desligado = LED **apagado**;
- **`64`** = cerca de **25%** = **brilho baixo**;
- **`128`** = cerca de **50%** = **brilho médio**;
- **`255`** = **100%** = sempre ligado = LED **aceso no máximo**.

> Por que 255? É o mesmo intervalo clássico do Arduino (que dá 256 níveis:
> 0 a 255). Apenas lembre: **0 é o mínimo, 255 é o máximo.** Quanto maior o
> número, mais o sinal fica ligado, mais brilho.

No `loop()`, cada linha define um nível e depois espera 1 segundo com
`delay()`, para você ter tempo de ver cada um. Assim você "enxerga" a
diferença entre os quatro níveis de brilho.

> Sobre a API: este código usa **`analogWrite()`, que é a forma atual e
> simplificada** nas versões recentes do pacote Arduino-ESP32 (Core 3.x).
> Quem ainda usa um Core antigo (2.x) ou PlatformIO com a versão antiga
> precisa usar as funções `ledcSetup()`/`ledcAttachPin()`/`ledcWrite()`.
> Nas versões novas, `analogWrite()` cuida disso sozinho — por isso usamos
> ela aqui. Se o seu código não compilar com `analogWrite()`, verifique qual
> versão do pacote ESP32 está instalada (Aula 00).

## Criando um fade

Agora vamos usar o que aprendemos para o efeito mais bonito: o **fade** — o
LED **aumenta o brilho aos poucos** até o máximo e depois **diminui** de
volta, suavemente.

Para isso, em vez de escrever níveis fixos, usamos um **laço `for`** que
passa por vários valores.

```cpp
// Aula 05 — Fade: brilho sobe e desce suavemente

const int LED = 2;

void setup() {
  pinMode(LED, OUTPUT);
}

void loop() {
  for (int brilho = 0; brilho <= 255; brilho++) {
    analogWrite(LED, brilho);
    delay(10);
  }

  for (int brilho = 255; brilho >= 0; brilho--) {
    analogWrite(LED, brilho);
    delay(10);
  }
}
```

### Entendendo o código novo

```cpp
for (int brilho = 0; brilho <= 255; brilho++) {
```

O **`for`** é um **laço** (uma repetição). É como dizer: "comece com uma
variável `brilho` valendo 0; **enquanto** ela for menor ou igual a 255;
**a cada volta**, aumente (`brilho++`, que significa `brilho = brilho + 1`)".

Ou seja, o bloco dentro do `for` vai rodar com `brilho` = 0, depois 1, 2,
3... até 255. A cada volta, fazemos o `analogWrite` com esse valor e
esperamos 10 ms.

```cpp
  analogWrite(LED, brilho);
  delay(10);
```

Define o brilho no valor atual da variável e espera um pouco. Como os valores
sobem devagar de 0 a 255, o LED **acende gradualmente**.

```cpp
for (int brilho = 255; brilho >= 0; brilho--) {
```

Agora o `for` ao **contrário**: começa em 255 e vai **diminuindo**
(`brilho--` = `brilho = brilho - 1`) até chegar em 0. Assim o LED **apaga
gradualmente**.

No fim do `loop()`, recomeçamos — o ciclo sobe e desce sem parar.

## O que deve acontecer

- **Primeiro teste**: o LED deve alternar a cada segundo entre **apagado →
  brilho baixo → brilho médio → brilho máximo**, e repetir.
- **Fade**: o LED deve **acender aos poucos**, chegar ao brilho máximo,
  depois **apagar aos poucos**, e repetir — um efeito suave e contínuo.

Se você vê diferenças claras de brilho (mesmo sem multímetro), o PWM está
funcionando.

## Se não funcionar

- **O LED não acende nem apaga**: confira o pino do LED no pinout (Aula 02)
  e a placa/porta corretas. Verifique o resistor em série.
- **O LED fica só "aceso" ou só "apagado", sem variar**: confira se está
  usando `analogWrite()` (e não `digitalWrite`). Se usou `analogWrite` e
  mesmo assim não variar, confira a versão do pacote ESP32 (a nota na seção
  "Entendendo o código").
- **Erro de compilação com `analogWrite`**: sua versão do Core ESP32 pode
  ser antiga. Atualize o pacote para a versão 3.x, ou use a API antiga
  (`ledcSetup`/`ledcAttachPin`/`ledcWrite`).
- **O fade sobe e desce rápido demais/lento demais**: ajuste o `delay(10)`
  dentro dos `for`. Menor = mais rápido; maior = mais lento.
- **A diferença entre 64 e 128 é pequena**: o olho percebe mudanças de brilho
  de forma não linear. É normal que os extremos pareçam mais "distantes".
  Isso não é um erro do programa.

## Experimente você

Agora, os desafios do capítulo (reaproveite o código do fade):

1. **Mudar a velocidade do fade**: altere o `delay(10)` para `delay(5)` e
   depois para `delay(30)`. Sinta como a velocidade do vai-e-vem muda.
2. **Criar três níveis de brilho**: escreva um programa que mostre
   **apagado → brilho baixo → brilho médio → brilho máximo**, mas com apenas
   **três** níveis intermediários de sua escolha entre 0 e 255.
3. **Botão alternando os níveis** (desafio mais completo): reutilize o botão
   da Aula 03. A cada **pressionada**, o brilho muda para o próximo nível:
   apagado (0) → baixo (64) → médio (128) → máximo (255) → de volta a 0.

   Dica: use `INPUT_PULLUP` para o botão e uma variável que guarde o **nível
   atual**. No `setup()`, coloque o pino do LED como `OUTPUT` e comece com
   `analogWrite(LED, 0)`.

   **Atenção**: como o LED agora precisa de PWM, **não** use `digitalWrite`
   para o LED; use sempre `analogWrite(LED, valor)`.

## O que aprendemos

Nesta aula:

- um GPIO digital normal tem dois estados: **HIGH** (aceso) e **LOW**
  (apagado);
- **PWM** resolve o problema ligando e desligando o sinal muito rápido — o
  olho vê a **média**;
- **PWM não é uma tensão analógica verdadeira**: o pino continua alternando
  entre **0 V e 3,3 V**;
- o **duty cycle** é a porcentagem de tempo em que o sinal fica ligado
  (100%, 50%, 25%, 0%);
- **`analogWrite(pino, valor)`** define o brilho com um valor de **0 a 255**;
- com um laço **`for`**, varremos vários valores e criamos um **fade**
  (brilho sobe e desce suavemente).

## Próximo passo

Agora que sabemos controlar o brilho de uma **saída**, vamos aprender a
**ler valores do mundo real**: como o ESP32 entende um sinal que não é
simplesmente ligado/desligado, mas pode ter muitos níveis.

Nos vemos na Aula 06.
