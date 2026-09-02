# Aula 01 — Nosso primeiro programa

## O que vamos fazer

Vamos escrever nosso **primeiro programa do zero**. Ele vai fazer a placa
ESP32 **acender e apagar um LED** — a famosa "luz que pisca" (o *Blink*).

Por que isso é útil? Porque o Blink é o "Olá, Mundo" da eletrônica: é a
primeira prova de que você conseguiu escrever código, enviar para a placa e
ver um resultado físico. Tudo que virá no curso — botões, sensores, Wi-Fi —
começa com essas mesmas duas ferramentas: enviar energia para um pino e
esperar um tempinho.

Na Aula 00 você já rodou o exemplo Blink pronto. Agora vamos **escrever o
nosso próprio**, linha por linha, e entender exatamente o que cada parte faz.

## O que você vai aprender

Nesta aula, quatro conceitos fundamentais:

- **setup()** e **loop()** — as duas partes de todo programa Arduino;
- **pinMode()** e **digitalWrite()** — como configurar e mandar energia para
  um pino;
- **HIGH** e **LOW** — os dois estados de uma saída digital;
- **delay()** — como fazer o programa esperar.

É muita coisa junta? Não se preocupe. Vamos devagar, e cada ideia é simples.

## Material necessário

- A placa **ESP32** (a mesma da Aula 00);
- o **cabo USB**;
- o **computador com o Arduino IDE** já configurado.

**Não** precisamos de LED externo nesta aula. A própria placa tem um LED
acoplado (normalmente em um pino chamado GPIO 2 em muitas placas comuns),
então dá para fazer o teste sem nenhum componente extra.

> ⚠️ O pino exato do LED interno varia conforme a placa. O **GPIO 2** é
> comum em muitas placas. **Confirme na documentação da sua placa** — e, se
> não tiver certeza, o código abaixo faz o teste em vários pinos possíveis.

## Antes de ligar

- Conecte a placa apenas pelo **cabo USB ao computador**. Ainda não vamos
  conectar nenhum componente externo.
- Se uma luz azul na placa piscar quando você conectar ou enviar código,
  **tudo bem** — algumas placas têm um LED de status que pisca durante a
  comunicação. Não é um erro.

## Ligação

A ligação desta aula é só o cabo USB, como na Aula 00:

```
[Computador]  —USB—>  [Placa ESP32]
```

O LED que vamos programar já está **dentro** da placa. Não precisa fazer
nenhuma ligação extra agora. O trabalho é todo no código!

## Primeiro teste

Nosso primeiro teste é abrir o programa e ver **a placa piscar**. Vamos
escrever o código na seção seguinte, enviá-lo à placa (botão de *Upload*,
mesma seta da Aula 00) e observar.

Se o LED da placa piscar no ritmo do programa, deu certo!

## Código

Vamos digitar este programa no Arduino IDE (num arquivo novo: *Arquivo >
Novo*, e apague o que vier pronto):

```cpp
// Aula 01 — Nosso primeiro programa
// Pisca um LED interno da placa ESP32

void setup() {
  pinMode(2, OUTPUT);
}

void loop() {
  digitalWrite(2, HIGH);
  delay(500);
  digitalWrite(2, LOW);
  delay(500);
}
```

> 💡 Se o seu LED interno **não** estiver no GPIO 2, troque os **quatro**
> números `2` pelo pino correto. Nas placas mais comuns, é 2 mesmo.

Depois de digitar, **verifique a placa e a porta** e clique em **enviar**
(*Upload*).

## Entendendo o código

Vamos ler o programa de cima para baixo, em linguagem simples.

```cpp
void setup() {
```

**`setup()`** é a primeira parte de todo programa Arduino. Pense nela como
**"preparação do palco"**: aqui você define como cada pino vai se comportar
antes de a ação começar. O que está entre as chaves `{` e `}` é o que
acontece nessa parte.

```cpp
  pinMode(2, OUTPUT);
```

**`pinMode`** significa "modo do pino". Ele diz à placa o que o pino vai
fazer: **`OUTPUT`** (saída) significa "este pino vai mandar energia para
fora". Neste caso, dizemos que o pino 2 vai ser uma saída — será usado para
acender algo.

```cpp
}
```

Fecha a função `setup()`. Cada todo programa Arduino **executa `setup()` uma
única vez** ao ligar.

```cpp
void loop() {
```

**`loop()`** é a segunda parte. Pense nela como o **"corpo da música"**: o
que está aqui se repete **sem parar**, o tempo todo, enquanto a placa estiver
ligada.

```cpp
  digitalWrite(2, HIGH);
```

**`digitalWrite`** significa "escrever digital": mandar um valor digital para
um pino. Aqui, mandamos **`HIGH`** para o pino 2. **`HIGH`** quer dizer
"ligado" (em inglês, *alto* — a tensão máxima que o pino fornece, 3,3 V).
Ou seja: acenda o LED.

```cpp
  delay(500);
```

**`delay`** significa "atraso". Ela pausa o programa pelo tempo indicado, em
**milissegundos** (milésimos de segundo). `delay(500)` espera **meio
segundo**. É como dizer "fique assim por um momento antes de continuar".

```cpp
  digitalWrite(2, LOW);
```

Agora mandamos **`LOW`** para o pino 2. **`LOW`** quer dizer "desligado"
(tensão mínima, 0 V). Ou seja: apague o LED.

```cpp
  delay(500);
}
```

Outra pausa de meio segundo, e fecha o `loop()`. Como o `loop()` se repete,
a sequência fica: **acende, espera, apaga, espera, acende, espera...** — o
LED pisca eternamente. 🎉

Resumindo o programa:

- `setup()`: prepara o pino 2 como saída (uma vez).
- `loop()`: repete para sempre — liga, espera meio segundo, desliga, espera
  meio segundo.

## O que deve acontecer

Depois do *Upload*, o LED da placa deve ficar **piscando** em um ritmo
constante: meio segundo aceso, meio segundo apagado.

Se você vê isso, programou com sucesso seu primeiro circuito — parabéns!

## Se não funcionar

- **O programa não envia (erro ao enviar)**: confirme a placa e a porta
  corretas (menu de placas e porta, como na Aula 00). Às vezes é preciso
  apertar o botão **BOOT** da placa durante o envio em algumas placas.
- **O LED não pisca**: o LED interno pode estar em outro pino. Verifique a
  documentação da sua placa e troque os números `2` pelo pino correto.
- **O LED pisca forte, mas muito rápido ou muito lento**: ajuste os números
  dentro de `delay()`. Lembre: estão em milissegundos, então `1000` é 1
  segundo.
- **Não vejo LED nenhum na placa**: algumas placas têm LEDs pequenos e
  discretos. Olhe de perto durante a piscada. Se mesmo assim nada acender,
  verifique se o LED interno realmente existe na sua variante.

## Experimente você

Agora mexa sozinho. Alguns desafios pequenos:

1. Faça o LED piscar **mais rápido** (por exemplo, `delay(100)` — um décimo
   de segundo).
2. Faça o LED piscar **mais devagar** (`delay(1000)` — 1 segundo).
3. Faça o LED ficar aceso **muito tempo** e apagado **rapidinho**, para criar
   um efeito de "piscada":

```cpp
void loop() {
  digitalWrite(2, HIGH);
  delay(2000);    // aceso por 2 segundos
  digitalWrite(2, LOW);
  delay(200);     // apagado por 0,2 segundo
}
```

Tente prever o que vai acontecer antes de enviar. Depois veja se acertou.

## O que aprendemos

Nesta aula:

- todo programa Arduino tem **`setup()`** (roda uma vez, prepara) e
  **`loop()`** (roda para sempre);
- **`pinMode(pino, OUTPUT)`** diz que o pino é uma saída;
- **`digitalWrite(pino, HIGH)`** liga o pino e **`LOW`** o desliga;
- **`delay(milissegundos)`** pausa o programa.

Com essas quatro ideias, você já controla a saída mais básica da placa.

## Próximo passo

Agora que sabemos ligar e desligar um pino, vamos **entender os pinos a
fundo** — quais usar, quais exigem cuidado e por que o ESP32 trabalha com
3,3 V.

Nos vemos na Aula 02.
