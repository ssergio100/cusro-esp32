# Aula 08 — I2C sem mistério

## O que vamos fazer

Até agora o ESP32 conversou com **um** componente por vez, geralmente usando
um GPIO por sinal. Mas e se quiséssemos conectar **vários dispositivos**
(sensor de temperatura, display, relógio) usando **poucos fios**?

Existe um jeito elegante de fazer isso: o **I2C**. Nesta aula vamos entender
o conceito de **barramento I2C** — como vários dispositivos dividem a mesma
"linha" com apenas dois fios de dados — e criar um **scanner** que encontra
os dispositivos conectados.

Por que isso é útil? É a base para usar displays pequenos (Aula 09), sensores
e relógios. Entender I2C agora torna o resto do curso muito mais fácil.

## O que você vai aprender

No final desta aula, você vai entender:

- a ideia de **"vários dispositivos na mesma linha"**;
- o que são **SDA** e **SCL**;
- o que é um **endereço** ("número da casa");
- como **ligar** um dispositivo I2C (3V3, GND, SDA, SCL);
- o conceito de **GND comum** (Aula 02);
- como criar um **scanner I2C**;
- como ler um resultado como **`0x3C`** (só o necessário sobre hexadecimal);
- o que acontece quando **nenhum dispositivo** é encontrado;
- um pequeno **desafio** para testar outro periférico.

## A ideia: vários dispositivos na mesma linha

Imagine um **ônibus** (em inglês, *bus*): várias pessoas embarcam e descem ao
longo do trajeto, mas o veículo é um só. Um **barramento** em eletrônica é
parecido: **vários dispositivos** compartilham **os mesmos fios** para falar
com o microcontrolador, um de cada vez.

O **I2C** é um tipo de barramento. Sua grande vantagem: precisa de **apenas
dois fios de dados** para conversar com **muitos dispositivos** ao mesmo
tempo.

```
ESP32 ──────┬───── dispositivo 1 (ex.: display)
            ├───── dispositivo 2 (ex.: sensor)
            └───── dispositivo 3 (ex.: relógio)
```

Todos usam os **mesmos dois fios**. Como o ESP32 sabe com qual deles está
falando? É para isso que serve o **endereço** — vamos ver em seguida.

## SDA e SCL

O I2C usa dois fios de dados, cada um com um papel:

- **SDA** (*Serial Data*): é o fio por onde os **dados** trafegam — as
  informações em si.
- **SCL** (*Serial Clock*): é o fio de **relógio/pulso** — o "ritmo" que
  mantém todo mundo sincronizado.

```
SCL = o "metrônomo" que marca o ritmo
SDA = o "pipe" por onde os dados fluem
```

Os nomes em inglês podem assustar, mas a ideia é simples: um fio marca o
ritmo (SCL), o outro carrega os dados (SDA).

## O endereço: o "número da casa"

Como vários dispositivos estão na mesma linha, o ESP32 precisa saber **com
qual deles falar**. Cada dispositivo I2C tem um **endereço** — pense como o
**número da casa em uma rua**.

Quando o ESP32 quer falar com um dispositivo, ele **chama pelo endereço**:

```
ESP32 → SDA → "Olá, casa 0x3C! Preciso que você..."
```

Só a "casa" com aquele endereço responde. Os outros ignoram. É assim que
vários dispositivos dividem a mesma linha sem se atrapalhar.

> **Um pouco sobre `0x3C`:** o `0x` indica que o número está em **hexadecimal**
> (base 16). Você **não precisa** saber converter — só precisa ler o número e
> usá-lo. No scanner, ele aparecerá como `0x3C`, `0x27` etc. Trate-os como
> "etiquetas" dos dispositivos.

## Ligação

Vamos ligar um dispositivo I2C. Conecte quatro fios:

```
ESP32             Dispositivo I2C
3V3   ──────────── VCC (energia)
GND   ──────────── GND (retorno)
GPIO SDA ──────── SDA (dados)
GPIO SCL ──────── SCL (relógio)
```

Atenção aos **GPIOs de SDA e SCL**, que variam conforme a placa:

- No **ESP32 clássico**, o padrão costuma ser **SDA = GPIO 21** e **SCL =
  GPIO 22**.
- Em **outras variantes** (como muitos ESP32-S3), os pinos I2C padrão
  **mudam**. **Consulte o pinout da sua placa (Aula 02)** e ajuste.

## Antes de ligar

- **Desligue o cabo USB** antes de montar, e reconecte só depois.
- **Não aplique mais de 3,3 V** em nenhum pino de sinal.
- Lembre do **GND comum** (Aula 02): todos os dispositivos no barramento
  devem compartilhar o **mesmo GND** da placa. Sem isso, a comunicação não
  funciona.
- Muitos **módulos prontos** já trazem os **resistores de pull-up** internos
  (lembra do conceito de pull-up da Aula 03?). Por isso, em geral, **não**
  precisamos adicionar resistores externos — apenas ligamos os quatro fios.

## Primeiro teste: o scanner I2C

Vamos conectar um dispositivo e **descobrir o endereço dele** automaticamente.
O **scanner I2C** é um programa que "varre" todos os endereços possíveis e
diz quais estão ocupados.

## Código

```cpp
// Aula 08 — Scanner I2C

#include <Wire.h>

void setup() {
  Serial.begin(115200);
  Wire.begin();

  Serial.println("Procurando dispositivos I2C...");

  int encontrados = 0;
  for (int endereco = 1; endereco < 127; endereco++) {
    Wire.beginTransmission(endereco);
    int erro = Wire.endTransmission();

    if (erro == 0) {
      Serial.print("Encontrado: 0x");
      if (endereco < 16) {
        Serial.print("0");
      }
      Serial.println(endereco, HEX);
      encontrados++;
    }
  }

  if (encontrados == 0) {
    Serial.println("Nenhum dispositivo encontrado.");
  } else {
    Serial.print("Total: ");
    Serial.print(encontrados);
    Serial.println(" dispositivo(s).");
  }
}

void loop() {
}
```

> O pino de SDA/SCL padrão é o usado pelo `Wire.begin()` na sua placa. Se o
> seu dispositivo estiver em outros pinos, use `Wire.begin(sda, scl)` com os
> GPIOs certos — consulte o pinout.

## Entendendo o código

```cpp
#include <Wire.h>
```

Incluímos a **biblioteca Wire**, que já vem com o Arduino e cuida da
comunicação I2C. (Vamos falar mais sobre bibliotecas na Aula 09.)

```cpp
Wire.begin();
```

**Inicia** o barramento I2C nos pinos padrão da placa.

```cpp
for (int endereco = 1; endereco < 127; endereco++) {
```

Um **laço `for`** (Aula 05) que **testa um por um** todos os endereços de 1 a
126.

```cpp
Wire.beginTransmission(endereco);
int erro = Wire.endTransmission();
```

Aqui está o "chamar a casa": **`beginTransmission(endereco)`** começa a falar
com o endereço, e **`endTransmission()`** termina e devolve um **código de
erro**. Se o `erro` for `0`, significa que **alguém respondeu** nesse
endereço.

```cpp
if (erro == 0) {
  Serial.print("Encontrado: 0x");
  if (endereco < 16) {
    Serial.print("0");
  }
  Serial.println(endereco, HEX);
  encontrados++;
}
```

Se deu certo (`erro == 0`), mostramos o endereço. O **`Serial.println(endereco,
HEX)`** imprime o número em **hexadecimal** (por isso o `0x` aparece). O `if`
extra só adiciona um `0` na frente para números pequenos ficarem com dois
dígitos (`0x04`, `0x0A` etc.).

```cpp
if (encontrados == 0) {
  Serial.println("Nenhum dispositivo encontrado.");
}
```

Se **nenhum** dispositivo respondeu, avisamos. Isso acontece quando há **fio
solto**, **GND não compartilhado**, **endereço fora da faixa** ou o
dispositivo **não está ligado/encontrado**.

## O que deve acontecer

Com um dispositivo I2C conectado e ligado (3V3, GND, SDA, SCL):

```
Procurando dispositivos I2C...
Encontrado: 0x3C
Total: 1 dispositivo(s).
```

O endereço `0x3C` é só um exemplo — o seu dispositivo mostrará o **endereço
dele** (pode ser `0x27`, `0x3C`, `0x68` etc.). **Anote esse número**: você
vai usá-lo na Aula 09.

Se você desconectar o dispositivo e rodar de novo, deve ver:

```
Nenhum dispositivo encontrado.
```

## Se não funcionar

- **"Nenhum dispositivo encontrado"**: confira os quatro fios (VCC, GND, SDA,
  SCL). Verifique se o **GND está compartilhado** com a placa. Confirme se os
  pinos SDA/SCL são os corretos da sua placa (pinout).
- **Resultado muda toda hora**: pode ser **mau contato** ou fio solto.
- **O programa não compila**: confirme que o Wire está instalado (vem por
  padrão no Arduino).
- **Endereço estranho ou `0xFF`**: pode indicar problema de alimentação ou de
  fio. Confira VCC e GND.

## Experimente você

Agora, o desafio:

1. **Descubra o endereço de outro dispositivo**: conecte um **segundo**
   dispositivo I2C que você tenha (ex.: outro sensor ou módulo) e rode o
   scanner. Repare que ele lista **dois** endereços — um para cada
   dispositivo na mesma linha.
2. **Teste sem dispositivo**: remova o dispositivo e rode o scanner de novo.
   Observe a mensagem "Nenhum dispositivo encontrado." — é importante
   **reconhecer esse erro** para os próximos capítulos.
3. Se seu módulo tiver **jumpers ou solda para escolher o endereço**, mude-o
   e veja o número diferente no scanner. Isso confirma o conceito de
   "número da casa".

## O que aprendemos

Nesta aula:

- **I2C** é um barramento em que **vários dispositivos** dividem **os mesmos
  dois fios**;
- **SDA** carrega os dados; **SCL** marca o ritmo;
- cada dispositivo tem um **endereço** (o "número da casa");
- a ligação usa **3V3, GND, SDA e SCL** (com **GND comum**);
- o scanner I2C **encontra os dispositivos** e mostra seus endereços em
  hexadecimal (ex.: `0x3C`);
- também mostra quando **nenhum** dispositivo é encontrado — e devemos
  reconhecer isso como um erro de ligação.

## Próximo passo

Agora que sabemos **encontrar** um dispositivo e seu endereço, vamos usar
um dispositivo I2C de verdade — como um display ou sensor — para **ler e
escrever** informações úteis.

Nos vemos na [Aula 09](../09-usando-um-dispositivo-i2c/README.md).
