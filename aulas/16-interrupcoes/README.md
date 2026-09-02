# Aula 16 — Interrupções

## O que vamos fazer

Até aqui, para saber se um botão foi pressionado, o ESP32 **perguntava** o
tempo todo no `loop()` (`digitalRead`). Isso se chama **polling** ("ficar
perguntando"). Funciona, mas há uma forma diferente: **interrupção**.

Uma **interrupção** é como uma **campainha**: em vez de o ESP32 ficar
olhando para a porta, ele faz seu trabalho e, quando a "campainha toca"
(algum evento acontece), ele **para na hora** e atende. Nesta aula vamos
aprender a usar isso com um botão, de forma simples e segura.

Por que isso é útil? Para eventos que precisam de resposta **imediatíssima**,
ou que acontecem raramente, a interrupção evita que você fique "perguntando"
sem parar. Mas ela tem regras — vamos ver quando usar e quando **não** usar.

## O que você vai aprender

No final desta aula, você vai entender:

- o **polling** (ficar perguntando) com `digitalRead`;
- a **interrupção** como "campainha";
- como criar um **exemplo com botão**;
- o que é uma **ISR** (e por que ela deve ser **muito curta**);
- por que **não colocar código pesado** dentro da interrupção;
- como usar uma **flag** para avisar o `loop()`;
- o básico de **`volatile`** (só o necessário);
- **quando usar e quando não usar**.

## Relembrando o polling

No `loop()` da Aula 03:

```cpp
int estado = digitalRead(BOTAO);
if (estado == LOW) {
  // botão pressionado
}
```

Esse jeito — **checar repetidamente** — chama-se **polling**. O ESP32 fica
"perguntando" a cada volta: "pressionado? e agora? e agora?".

Problema: se o `loop()` estiver ocupado com outra coisa (ou `delay()` longo),
o ESP32 pode **demorar** para notar o botão.

## A interrupção: a "campainha"

A **interrupção** resolve isso. O ESP32 **se inscreve** para ser avisado
quando o pino **mudar** e, enquanto isso, continua fazendo suas outras
tarefas. Quando a mudança acontece, o hardware **interrompe** o que estava
fazendo e chama uma função especial: a **ISR**.

> **ISR** = *Interrupt Service Routine* ("rotina de serviço de interrupção").
> É a função que roda **na hora** quando a interrupção dispara.

Pense assim:

- **Polling**: você fica encostado na campainha, apertando o botão para ver
  se chamaram. Ineficiente.
- **Interrupção**: você deixa a campainha **te chamar** quando tocar. Você é
  avisado na hora, sem ficar olhando.

```
SEM interrupção (polling):
loop: verifica pino? verifica pino? verifica pino? ...

COM interrupção:
[ESP32 faz outras coisas]  →PINOA muda→  interrupt! → ISR roda na hora
```

## Ligação

Monte o **botão** (GPIO → botão → GND, com `INPUT_PULLUP`, da Aula 03) e o
**LED** (Aula 01/05). Nada além disso.

```
ESP32
GPIO 4 ───── botão ───── GND
GPIO 2 ───── resistor ───── LED ───── GND   (ou LED interno)
```

## Primeiro teste: interrupção com botão

Vamos fazer o ESP32 **ser avisado** quando o botão for pressionado e ligar o
LED. Mas atenção: **não colocaremos código pesado na ISR** — só vamos
**marcar** que algo aconteceu (a **flag**) e deixar o `loop()` fazer o resto.

```cpp
// Aula 16 — Interrupção de botão com flag

const int BOTAO = 4;
const int LED = 2;

volatile bool botaoPressionado = false;

void IRAM_ATTR aoPressionar() {
  botaoPressionado = true;
}

void setup() {
  Serial.begin(115200);
  pinMode(BOTAO, INPUT_PULLUP);
  pinMode(LED, OUTPUT);

  attachInterrupt(digitalPinToInterrupt(BOTAO), aoPressionar, FALLING);
}

void loop() {
  if (botaoPressionado) {
    botaoPressionado = false;
    digitalWrite(LED, !digitalRead(LED));
    Serial.println("Interrupcao disparada!");
  }
}
```

### Entendendo o código

```cpp
volatile bool botaoPressionado = false;
```

A variável **`botaoPressionado`** é a nossa **flag** (uma "bandeirinha" que
marca se algo aconteceu). O **`volatile`** avisa o compilador que essa
variável pode mudar **a qualquer momento** (por causa da interrupção), para
ele não "guardar uma cópia" desatualizada. É **necessário** para variáveis
compartilhadas com a ISR. Só isso que você precisa saber por enquanto.

```cpp
void IRAM_ATTR aoPressionar() {
  botaoPressionado = true;
}
```

Esta é a nossa **ISR**. O **`IRAM_ATTR`** é um detalhe próprio do ESP32 que
faz a ISR rodar de memória adequada (não precisa entender os detalhes — o
padrão é usar `IRAM_ATTR` em ISRs no ESP32). A função **não** faz o trabalho
pesado: ela só **marca** que o botão foi pressionado (`botaoPressionado =
true`). Isso mantém a ISR **muito curta**.

```cpp
attachInterrupt(digitalPinToInterrupt(BOTAO), aoPressionar, FALLING);
```

**`attachInterrupt(pino, funcao, modo)`** **ativa** a interrupção:
- `digitalPinToInterrupt(BOTAO)` — converte o número GPIO em "número de
  interrupção" (padrão do Arduino);
- `aoPressionar` — a função a chamar;
- `FALLING` — o **modo**: significa "interrompa quando o sinal **cair**" (de
  HIGH para LOW). Lembra da Aula 03? Com `INPUT_PULLUP`, pressionar derruba
  o pino para LOW — uma "queda" (FALLING). Perfeito.

```cpp
void loop() {
  if (botaoPressionado) {
    botaoPressionado = false;
    digitalWrite(LED, !digitalRead(LED));
    Serial.println("Interrupcao disparada!");
  }
}
```

No `loop()`, **verificamos a flag**. Se estiver `true` (alguém pressionou),
**limpamos** (voltamos a `false`) e fazemos o trabalho: alternamos o LED
(`!digitalRead`, da Aula 04). O **trabalho pesado fica no `loop()`**, nunca
na ISR.

```
evento (botão) → ISR marca flag → loop() vê a flag e age (LED/Serial)
```

## Por que a ISR deve ser curta?

Quando a interrupção dispara, ela **congela** o que o ESP32 estava fazendo e
começa a rodar a ISR. Se a ISR **demorar** (ex.: com `delay()`, `Serial` pesado
ou operações longas), o ESP32 fica **preso** nela e **perde** o resto.

Regras simples para a ISR:

- **NÃO use `delay()`** dentro da ISR (não faça sentido esperar ali);
- **NÃO faça `Serial` pesado** dentro da ISR (transmitir custa tempo);
- **NÃO** faça tarefas longas — a ISR deve ser **quase instantânea**.

O padrão saudável: a ISR **marca** (muda uma flag/variável) e vai embora. O
`loop()` faz o trabalho depois. É **exatamente** o que fizemos acima.

## Sobre o "tremor" do botão

Lembra do "*bounce*" citado na Aula 03? O contato do botão "tremelicar"
pode disparar a interrupção **várias vezes** em um clique. Neste exemplo
simples, isso pode fazer o LED piscar mais de uma vez por clique. É normal
e aceitável aqui — o tratamento completo do debounce fica para uma aula
mais avançada. O **conceito** da interrupção é o foco.

## Quando usar e quando não usar

**Use interrupção quando:**

- o evento precisa de resposta **muito rápida**;
- o evento acontece **raramente** (ex.: um botão, um pulso) e você não quer
  ficar perguntando;
- você precisa "acordar" o ESP32 de um estado de espera.

**NÃO use (prefira polling/`millis`) quando:**

- a tarefa de resposta é **complexa ou demorada** — deixe no `loop()`;
- você precisa de várias ISRs com lógica compartilhada complicada;
- o evento é **frequente e contínuo** (ex.: ler um sensor o tempo todo) — aí
  o polling no `loop()` é mais simples.

**Regra de ouro:** a **interrupção avisa**; quem faz o trabalho é o `loop()`.

## Se não funcionar

- **O LED nem liga**: confira a ligação do botão (GPIO, GND) e do LED, e o
  pino correto.
- **Pisca mais de uma vez por clique**: é o **bounce** (tremor) do botão.
  Nesta aula, é esperado. (O debounce é um assunto futuro.)
- **"Double reset"/comportamento estranho**: confirme se não há `delay()` ou
  `Serial` dentro da sua ISR.
- **Não funciona em todas as placas**: alguns GPIOs do ESP32 não podem ser
  interrompidos. Use os pinos que o pinout da sua placa indica como
  interrompíveis (a maioria dos GPIOs comuns funciona).

## Experimente você

Agora, os desafios:

1. **Mude o modo de interrupção**: teste também `RISING` (quando o sinal
   **sobe**, ou seja, ao soltar o botão). Veja a diferença de quando "dispara".
2. **Acumule cliques**: em vez de só alternar o LED, faça uma variável
   `volatile int cliques` que **aumenta** na ISR, e mostre o total no
   `loop()` quando mudar.
3. **Reconheça o limite**: rode o exemplo da Aula 03 (polling) e depois este
   (interrupção) e note como este reage **sem precisar ficar perguntando**.

## O que aprendemos

Nesta aula:

- **polling** = ficar perguntando; **interrupção** = ser avisado na hora;
- a **ISR** é a função que roda quando a interrupção dispara e deve ser
  **muito curta**;
- usamos **`attachInterrupt`** com o modo **`FALLING`** (queda do sinal);
- a ISR **marca uma flag** (`volatile`) e o **`loop()` faz o trabalho**;
- **não** se usa `delay()` nem `Serial` pesado **dentro da ISR**;
- interrupção **não** é para qualquer situação — há casos em que o **polling**
  é mais simples e adequado.

## Próximo passo

Vimos o `millis()` (Aula 04) e agora as interrupções. Existe ainda outra
ferramenta de tempo: os **timers** — que podem até **gerar interrupções**
periodicamente. Vamos diferenciar tudo isso na próxima aula.

Nos vemos na Aula 17.
