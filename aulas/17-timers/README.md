# Aula 17 — Timers

## O que vamos fazer

Nosso ESP32 já sabe medir o tempo com `millis()` (Aula 04) e reagir
rapidamente com interrupções (Aula 16). Agora vamos conhecer os **timers**
("temporizadores"): relógios internos que podem **disparar algo sozinhos** a
cada intervalo fixo.

Pense no **timer** como um **despertador**: o ESP32 agenda "me acorde a cada
X segundos", e, quando o momento chega, ele **para o que está fazendo** e roda
uma função. Nesta aula vamos apagar **o LED piscando sozinho** usando um
timer — sem precisar de `delay()` nem ficar checando `millis()`.

Por que isso é útil? Para ações que devem acontecer **de tempos em tempos**
(medir um sensor a cada segundo, enviar um dado a cada minuto), o timer cuida
**sozinho** — você não precisa gerenciar o tempo no `loop()`.

## O que você vai aprender

No final desta aula, você vai entender:

- o **timer** como "despertador periódico";
- a diferença entre `delay`, `millis` e **timer**;
- como **criar** um timer;
- como **agendar** uma função para rodar a cada intervalo;
- como **ligar/desligar (arrancar/parar)** o timer;
- por que a **função do timer** deve ser curta;
- **quando usar** cada ferramenta de tempo.

## Relembrando as ferramentas de tempo

- **`delay()`** (Aula 01): **pare** o ESP32 por um tempo. Bloqueia tudo.
- **`millis()`** (Aula 04): o ESP32 **anota** que horas são e você verifica
  no `loop()` se passou o intervalo. Deixo o ESP32 livre no meio.
- **Timer** (esta aula): o ESP32 **agenda** e é **avisado sozinho** quando a
  hora chega. Não precisa nem ficar olhando o relógio.

```
delay:    [pausa total]
millis:   [verifica no loop se passou o tempo]
timer:    [o ESP32 é avisado sozinho a cada intervalo]
```

O **timer** combina com a **interrupção**: ele pode **disparar uma função**
(a ISR do timer) de tempos em tempos, sem `loop()` envolvido.

## Ligação

Só o **LED** (GPIO 2 ou LED interno). Nada externo necessário.

```
ESP32
GPIO 2 ───── resistor ───── LED ───── GND   (ou LED interno)
```

Abra o **Monitor Serial** (115200).

## Primeiro teste: timer que pisca o LED

Vamos criar um timer que, a cada **1 segundo (1.000.000 microssegundos)**,
apaga ou acende o LED sozinho:

```cpp
// Aula 17 — Timer periódico (despertador)

#include <esp32-hal-timer.h>

const int LED = 2;

hw_timer_t* meuTimer = NULL;
volatile bool ligarLED = true;

void IRAM_ATTR aoPiscar() {
  ligarLED = !ligarLED;
}

void setup() {
  Serial.begin(115200);
  pinMode(LED, OUTPUT);

  meuTimer = timerBegin(1000000);           // precisão: 1 microssegundo
  timerAttachInterrupt(meuTimer, aoPiscar);  // função a chamar
  timerAlarm(meuTimer, 1000000, true, 0);    // 1 s, repetir, sem alarme extra
  timerStart(meuTimer);                      // liga o despertador
}

void loop() {
  digitalWrite(LED, ligarLED);
}
```

### Um aviso importante sobre a API do timer

**O ESP32 mudou as funções de timer entre versões do framework Arduino.**
As funções que usamos aqui (`timerBegin`, `timerAttachInterrupt`, `timerAlarm`,
`timerStart` com estes argumentos) correspondem à **API mais recente do
ESP32 Arduino Core 3.x**.

Se, na sua versão da IDE, o código **não compilar**, é porque a sua versão
usa a **API antiga** (ex.: `timerBegin(0, 80, true)` com três argumentos).
**Nesse caso**, procure o exemplo "Timer" que veio com a sua instalação, ou
consulte a página da sua placa, e **adapte** as chamadas. O **conceito** —
pedir ao timer para chamar uma função a cada intervalo — é o mesmo em
qualquer versão; só muda a sintaxe. Este é um ótimo exemplo de por que
sempre devemos **verificar a API da nossa versão**.

### Entendendo o código

```cpp
#include <esp32-hal-timer.h>
```

Inclui o suporte a timers do ESP32.

```cpp
hw_timer_t* meuTimer = NULL;
```

Cria um **ponteiro** para o timer (`hw_timer_t*`). Não se assuste com o `*`:
pense em `meuTimer` como o "endereço" do nosso despertador. `NULL` significa
"ainda não criei". Isso é um detalhe técnico que os exemplos de timer usam —
você não precisa dominar ponteiros para seguir a aula.

```cpp
volatile bool ligarLED = true;
```

Uma **flag** (`volatile`, como na Aula 16) que o timer alterna. O `loop()` a
lê para decidir o LED.

```cpp
void IRAM_ATTR aoPiscar() {
  ligarLED = !ligarLED;
}
```

A **função do timer** (observe o `IRAM_ATTR`, como nas ISRs da Aula 16). Ela
**só inverte** a flag. **Curta** — o trabalho real (acender o LED) fica no
`loop()`.

```cpp
meuTimer = timerBegin(1000000);
```

**`timerBegin(frequencia)`** **cria** o timer. Aqui, `1000000` = **1 milhão
de contagens por segundo** (microssegundos). Ou seja, o "tic" do timer.

```cpp
timerAttachInterrupt(meuTimer, aoPiscar);
```

**`timerAttachInterrupt(timer, funcao)`** diz qual **função chamar** quando o
alarme soar (como o `setCallback` do MQTT ou o `attachInterrupt` da Aula 16).

```cpp
timerAlarm(meuTimer, 1000000, true, 0);
```

**`timerAlarm(timer, valor, repetir, semAlarme)`** **agenda** o alarme:
- `1000000` — depois de **1 milhão de tics** (1 segundo), toca;
- `true` — **repita** para sempre (se fosse `false`, tocaria uma vez só);
- `0` — sem recurso extra de alarme (deixe 0 neste exemplo).

```cpp
timerStart(meuTimer);
```

**`timerStart(timer)`** **liga** o despertador. A partir daqui, a cada 1
segundo, a função `aoPiscar` é chamada **sozinha**, mesmo sem nada no
`loop()` senão o `digitalWrite`.

```cpp
void loop() {
  digitalWrite(LED, ligarLED);
}
```

O `loop()` fica **simples**: só acende/apaga o LED conforme a flag. Quem
**muda** a flag é o **timer** (o despertador), a cada 1 segundo.

```
timer conta 1 s → dispara → aoPiscar() inverte flag → loop() apaga/acende
```

## Detalhes práticos do timer

- **Arrancar e parar:** além de `timerStart`, você pode **parar** com
  `timerStop(meuTimer)` e **recomeçar** com `timerStart(meuTimer)` de novo.
  Isso é útil para "pausar" um despertador quando não quiser que ele toque.
- **Mudar o intervalo:** para tocar a cada X unidade de tempo, ajuste o valor
  do `timerAlarm` (ex.: `500000` = meio segundo).
- **Lembrar:** assim como a ISR da Aula 16, a função do timer deve ser
  **curta** — sem `delay()`, sem `Serial` pesado. Faça o trabalho no `loop()`.

## Quando usar cada ferramenta de tempo

Para ajudá-lo a escolher:

- **`delay()`** — quando o ESP32 pode simplesmente "ficar parado" esperando,
  sem precisar fazer mais nada (o mais simples, porém bloqueia).
- **`millis()`** — quando o ESP32 precisa **fazer várias coisas ao mesmo
  tempo** e você mesmo controla os intervalos no `loop()` (muito comum e
  recomendado).
- **Timer** — quando você quer que algo aconteça **de tempos em tempos sem
  precisar gerenciar no `loop()`**, especialmente se precisar de precisão ou
  de rodar **enquanto o `loop()` faz outra coisa pesada** (usando a
  interrupção do timer).

**Regra prática:** comece sempre com `millis()` no `loop()` — é o mais simples
e suficiente na maioria dos casos. Use o **timer** quando quiser "delegar" a
contagem ao hardware.

## Se não funcionar

- **"timerBegin" não compila**: a sua versão usa a **API antiga** do timer.
  Adapte conforme o aviso acima (procure o exemplo "Timer" da sua IDE e
  ajuste as chamadas). O conceito continua o mesmo.
- **O LED fica parado** (não pisca): confira se `timerStart` foi chamado e se
  o pino do LED está certo. Abra o Serial e veja se há erro.
- **Pisca rápido demais / demais**: ajuste o valor do `timerAlarm`
  (o intervalo em tics; lembre que `1000000` = 1 segundo com a frequência
  que definimos).
- **Dá "reset"**: se a função do timer ficou pesada (com `delay` ou `Serial`
  grande), pode travar. Deixe-a **curta** e faça o resto no `loop()`.

## Experimente você

Agora, os desafios:

1. **Mude o intervalo**: faça o LED piscar a cada **meio segundo** (`500000`).
2. **Conte as batidas**: use uma flag `volatile long batidas` que o timer
   aumenta, e o `loop()` mostra no **Serial** quando mudar.
3. **Suporte a pausa**: adicione um **botão** (Aula 03) que, quando
   pressionado, faz `timerStop`; quando pressionado de novo, `timerStart`.
   Use uma flag na ISR do botão (Aula 16). Agora você tem um "despertador
   que pode ser silenciado".

> Dica: lembre-se da regra de ouro — a **função do timer** e a **ISR do
> botão** devem ser **curtas**; quem faz o trabalho (piscar, imprimir) é o
> `loop()`.

## O que aprendemos

Nesta aula:

- o **timer** é um "despertador periódico" que chama uma função sozinho;
- ele **combina com interrupção** (a função do timer é uma ISR curta);
- usamos `timerBegin`, `timerAttachInterrupt`, `timerAlarm` e `timerStart`
  (API nova; a versão antiga troca a sintaxe, não o conceito);
- `timerStop`/`timerStart` **pausam/recomeçam** o despertador;
- a **função do timer** deve ser **curta** — o trabalho fica no `loop()`;
- escolhemos entre `delay`, `millis` e **timer** conforme a necessidade.

## Próximo passo

Até aqui programamos em "nível alto", sem nos preocupar muito com os
bastidores. Na próxima aula vamos olhar **dentro do ESP32** e entender a
**memória**: existem diferentes tipos, e saber disso ajuda a escrever
programas que não travam com projetos grandes.

Nos vemos na Aula 18.
