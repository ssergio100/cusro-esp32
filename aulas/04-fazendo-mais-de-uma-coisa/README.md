# Aula 04 — Fazendo mais de uma coisa ao mesmo tempo

## O que vamos fazer

Na Aula 03, o botão controlava um LED. Mas o programa tinha um detalhe: ele
usava `delay()` para esperar. E `delay()` tem um problema — **enquanto o
programa espera, ele não faz mais nada**.

Nesta aula vamos entender esse problema e aprender uma forma melhor de
**controlar o tempo**: a função **`millis()`**. Com ela, o ESP32 continua
trabalhando o tempo todo e só "checa" quando cada coisa deve acontecer.

Por que isso é útil? Porque projetos reais precisam fazer **várias coisas ao
mesmo tempo** (ler botão, piscar LED, ler sensor, mandar dados). Com
`delay()`, a placa fica parada; com `millis()`, ela faz tudo sem travar.

## O que você vai aprender

No final desta aula, você vai entender:

- o que o `delay()` realmente faz;
- por que `delay()` pode ser um problema;
- o que é **`millis()`**;
- como **medir o tempo decorrido**;
- como executar uma ação **periodicamente sem bloquear** o programa;
- como fazer **duas tarefas simples** acontecerem de forma independente;
- os conceitos de **"tempo anterior"** e **"tempo atual"**.

## Por que delay() pode atrapalhar

Primeiro, vamos lembrar o que o `delay()` faz de verdade.

```cpp
delay(1000);
```

É simples: **durante esse tempo, o programa fica esperando.** Enquanto ele
espera, **nada mais acontece**. Não lê botão, não lê sensor, não faz nada.

Imagine que você quer fazer duas coisas juntas:

- um **LED** piscando (cada 1 segundo), e
- um **botão** sendo lido (para o LED ligar/desligar).

O problema: no tempo que o programa está parado no `delay()`, ele **não lê o
botão**. Se você apertar o botão **justamente nesse momento**, o ESP32 pode
**demorar** para perceber — ele só "acorda" quando o `delay()` termina.

É como se fosse um vigia que **fecha os olhos** por um segundo e, se algo
acontecer nesse segundo, ele **não vê**.

## O que é millis()

A ideia é não "fechar os olhos". Em vez de **esperar parado**, o ESP32
continua rodando e **verifica de vez em quando se o tempo já passou**.

```
COM delay():                    COM millis():
LED liga                        verifica LED
   ↓                                ↓
espera 1s (PARADO)              verifica botão
   ↓                                ↓
LED desliga                     verifica LED
   ↓                                ↓
espera 1s (PARADO)              verifica botão
   ↓                                ↓
...                             verifica LED
                                ...
```

No da esquerda, o programa fica **preso** esperando. No da direita, ele **não
para nunca** — só vai "checando" cada coisa.

A função **`millis()`** nos dá a ferramenta para isso. `millis()` mostra
**quantos milissegundos se passaram desde que o ESP32 começou a executar o
programa**. É como um **relógio interno** que só cresce.

- Quando o programa inicia, `millis()` vale ~0;
- depois de meio segundo, vale ~500;
- depois de 1 segundo, ~1000;
- e assim por diante, contando para sempre.

> Não é um relógio "mágico": é só um contador que a placa mantém e nós
> consultamos.

## Primeiro teste

Nosso primeiro teste será **piscar um LED sem usar `delay()`**, usando o
relógio do `millis()` para decidir quando acender e quando apagar. O resultado
físico é o mesmo de antes (LED piscando), mas o programa **nunca fica
parado**.

## Código

Abra um sketch novo e digite:

```cpp
// Aula 04 — LED piscando com millis(), sem delay()

const int LED = 2;

unsigned long anterior = 0;
const unsigned long intervalo = 1000;

void setup() {
  pinMode(LED, OUTPUT);
}

void loop() {
  unsigned long agora = millis();

  if (agora - anterior >= intervalo) {
    anterior = agora;
    digitalWrite(LED, !digitalRead(LED));
  }
}
```

Depois, **compile e envie** (Aula 01) e observe o LED.

## Entendendo o código

Vamos ler as partes novas, devagar.

```cpp
unsigned long anterior = 0;
const unsigned long intervalo = 1000;
```

Aqui já vão as variáveis que vamos usar.

- **`unsigned long`** é um tipo de caixinha que **guarda números grandes
  que não têm sinal negativo** (só números positivos). Por que "grande"?
  Porque o valor do `millis()` cresce muito com o tempo, e precisamos de uma
  caixinha que não "estoure". Não precisa decorar — só saber que é a caixinha
  certa para tempos.
- **`anterior`** vai guardar o **último momento** em que fizemos algo. Por
  enquanto, `0`.
- **`intervalo`** é o nosso intervalo fixo de **1000 ms = 1 segundo**. É o
  tempo que queremos esperar entre as ações. É `const` porque não muda.

```cpp
unsigned long agora = millis();
```

A cada volta do `loop()`, perguntamos ao relógio: "**que horas são agora?**"
Guardamos o valor atual na variável **`agora`**.

```cpp
if (agora - anterior >= intervalo) {
```

Essa é a pergunta-chave: **"já passou tempo suficiente desde a última
vez?"**

Pense assim:

- `anterior` = o momento em que fizemos a ação **da última vez**;
- `agora` = o momento **atual**;
- `agora - anterior` = **quanto tempo já se passou** desde a última ação.

Aí comparamos (`>=` significa "maior ou igual a"): se o tempo passado `>=` o
nosso `intervalo` (1000 ms), significa que **já deu o tempo** — agora podemos
agir. Essa é a ideia de **"tempo anterior" e "tempo atual"**.

```cpp
  anterior = agora;
  digitalWrite(LED, !digitalRead(LED));
}
```

Se o tempo chegou, fazemos duas coisas:

1. **`anterior = agora;`** — atualizamos o "último momento" para o momento
   atual. Assim, a próxima comparação conta **a partir de agora**. É como
   "reiniciar o cronômetro".
2. **`digitalWrite(LED, !digitalRead(LED))`** — **alterna** (inverte) o LED.
   Vamos ler essa linha com calma:
   - `digitalRead(LED)` lê o estado **atual** do LED (HIGH ou LOW);
   - `!` significa **"negação"** (inverte): `!HIGH` vira `LOW`, e `!LOW`
     vira `HIGH`;
   - `digitalWrite(LED, ...)` escreve esse valor invertido.

   Ou seja: se o LED está aceso, apaga; se está apagado, acende. É um "liga →
   desliga → liga" automático. **Não usamos `delay()` em lugar nenhum.**

## Fazendo duas coisas ao mesmo tempo

Vamos usar o que aprendemos para fazer **duas tarefas independentes**: um
LED piscando a cada 1 segundo **e** um botão sendo lido o tempo todo — sem
um atrapalhar o outro.

Reutilize a ligação da Aula 03:

```
ESP32
GPIO 4 ───── botão ───── GND
GPIO 2 ───── resistor ───── LED ───── GND
```

### Código

```cpp
// Aula 04 — LED pisca sozinho + botão é lido o tempo todo

const int LED = 2;
const int BOTAO = 4;

unsigned long anteriorLED = 0;
const unsigned long intervaloLED = 1000;

void setup() {
  Serial.begin(115200);
  pinMode(LED, OUTPUT);
  pinMode(BOTAO, INPUT_PULLUP);
}

void loop() {
  unsigned long agora = millis();

  if (agora - anteriorLED >= intervaloLED) {
    anteriorLED = agora;
    digitalWrite(LED, !digitalRead(LED));
  }

  int estado = digitalRead(BOTAO);
  if (estado == LOW) {
    Serial.println("Pressionado!");
  }
}
```

> Troque o pino do LED se o seu for diferente (Aula 02).

### Entendendo o que mudou

- Damos a cada tarefa **seu próprio** `anterior` e seu próprio `intervalo`
  (aqui, `anteriorLED` e `intervaloLED`).
- No `loop()`, o LED usa `millis()` para decidir **quando** piscar.
- **Na mesma volta do loop**, lemos o botão com `digitalRead(BOTAO)` e
  imprimimos se estiver pressionado.

Como o `loop()` roda **muito rápido** e nunca fica preso em um `delay()`, o
botão é checado **constantemente**, mesmo enquanto o LED pisca. É isso que
queremos: duas tarefas independentes, cada uma com seu tempo.

```
Com millis():

verifica LED (passou 1s? muda se sim)
   ↓
verifica botão (está pressionado?)
   ↓
verifica LED de novo
   ↓
verifica botão de novo
   ↓
...
```

O programa **não espera parado**; ele fica circulando e "checa" quando cada
coisa deve acontecer. Por isso ele continua **responsivo** — se você
pressionar o botão, o ESP32 percebe na hora.

> Importante: em vez de dizer que isso é "multitarefa", pense assim: o ESP32
> executa o `loop()` várias vezes por segundo e, a cada volta, **verifica** se
> é hora de cada tarefa. Não há mágica — é só um relógio e umas comparações.

## O que deve acontecer

- No primeiro teste: o **LED pisca** a cada 1 segundo, mesmo sem `delay()`.
- No segundo exemplo: o **LED continua piscando sozinho** e, **no Monitor
  Serial**, "Pressionado!" aparece **no momento** em que você aperta o
  botão — sem atraso perceptível. As duas coisas acontecem **ao mesmo tempo**.

## Se não funcionar

- **O LED não pisca**: confira o pino do LED (Aula 02) e se a placa/porta
  estão corretas. Verifique se o código **não tem `delay()`** que impeça o
  loop.
- **O LED pisca rápido demais ou não alterna**: confira se `intervalo` está
  em milissegundos (`1000` = 1 s) e se `anterior` está sendo atualizado
  dentro do `if` (se esquecer, o `if` roda toda hora).
- **O botão não aparece no Monitor Serial**: mesma checagem da Aula 03
  (velocidade `115200`, porta correta, ligação do botão).
- **As duas coisas não acontecem "juntas"**: lembre que o `loop()` roda
  rápido e a tarefa do LED só muda quando o intervalo passa. O botão é
  checado a cada volta — o comportamento esperado é esse.

## Experimente você

Agora, o desafio do capítulo. Vamos fazer **dois LEDs** com **velocidades
diferentes** e **sem usar `delay()`**:

1. **LED A** pisca a cada **1 segundo**;
2. **LED B** pisca a cada **300 ms**.

Para isso, adicione um segundo LED (resistor + LED, ligado a outro GPIO, por
exemplo o **GPIO 5**), e **duplique** a estrutura do LED usando um segundo par
de variáveis (`anteriorLEDB` e `intervaloLEDB = 300`).

```cpp
// Aula 04 — Dois LEDs, velocidades diferentes, sem delay()

const int LEDA = 2;
const int LEDB = 5;

unsigned long anteriorA = 0;
const unsigned long intervaloA = 1000;

unsigned long anteriorB = 0;
const unsigned long intervaloB = 300;

void setup() {
  pinMode(LEDA, OUTPUT);
  pinMode(LEDB, OUTPUT);
}

void loop() {
  unsigned long agora = millis();

  if (agora - anteriorA >= intervaloA) {
    anteriorA = agora;
    digitalWrite(LEDA, !digitalRead(LEDA));
  }

  if (agora - anteriorB >= intervaloB) {
    anteriorB = agora;
    digitalWrite(LEDB, !digitalRead(LEDB));
  }
}
```

Observe como os dois LEDs piscam em **ritmos diferentes**, independentes um do
outro, e o programa nunca trava. Essa é a base de praticamente todo projeto
com ESP32 que faz "várias coisas".

Depois, tente **mudar as velocidades** (`intervaloA` e `intervaloB`) e ver o
efeito.

## O que aprendemos

Nesta aula:

- **`delay()`** faz o programa **esperar parado** — e, nesse tempo, ele não
  faz mais nada (pode perder um evento, como um botão);
- **`millis()`** é o "relógio interno" que diz quantos milissegundos se
  passaram desde o início;
- usando **"tempo atual" e "tempo anterior"**, calculamos se **já passou o
  intervalo** ("tempo decorrido");
- assim, executamos ações **periodicamente sem bloquear** o programa;
- cada tarefa pode ter **seu próprio intervalo** e seu próprio "último
  momento", rodando independentemente;
- o ESP32 **não para**: fica rodando o `loop()` e **verifica** quando cada
  tarefa deve acontecer.

## Próximo passo

Agora sabemos ligar, desligar e piscar com ritmo. Mas até aqui só usamos dois
estados (aceso/apagado). Vamos aprender a **controlar a intensidade** — como
fazer um LED brilhar em vários níveis, não só ligar ou desligar.

Nos vemos na Aula 05.
