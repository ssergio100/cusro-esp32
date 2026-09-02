# Aula 03 — Lendo um botão

## O que vamos fazer

Até aqui o ESP32 **mandou** sinais para fora (acendeu um LED, na Aula 01).
Nesta aula vamos fazer o contrário: o ESP32 vai **escutar** um sinal que vem
de fora. Vamos ligar um **botão** à placa e fazer o ESP32 descobrir quando
ele é **pressionado** e quando está **solto**.

Por que isso é útil? Porque um botão é a forma mais comum de dar uma "ordem"
a uma placa: ligar/desligar algo, escolher uma opção, confirmar uma ação.
Saber *ler* um botão abre as portas para quase todos os projetos interativos.

No fim da aula, você vai usar o botão para **controlar um LED**.

## O que você vai aprender

No final desta aula, você vai entender:

- a **diferença prática** entre entrada e saída;
- o que faz a função **digitalRead()**;
- como configurar um pino com **pinMode()**;
- a diferença entre **INPUT** e **INPUT_PULLUP**;
- por que uma **entrada não deve ficar "solta"**;
- como **ligar um botão** corretamente;
- como detectar botão **pressionado** e **solto**;
- como usar uma **condição simples com `if`**;
- como **controlar um LED com um botão**.

## Material necessário

- A placa **ESP32**;
- o **cabo USB**;
- o **computador com o Arduino IDE**;
- um **botão** de pressão simples (um "botãozinho" de 4 pernas, comum em kits);
- **fios** de ligação (jumpers);
- um **LED** e um **resistor** (para o projeto prático de controlar o LED).

> Vamos usar o **resistor interno** da placa (INPUT_PULLUP) para o botão,
> então **não** precisamos de resistor externo ali. O resistor externo que
> você verá é só para proteger o **LED**, no projeto final.

## Antes de ligar

- **Desligue o cabo USB** da placa **antes** de fazer qualquer ligação. Monte
  o circuito com a placa sem energia e só conecte o cabo depois.
- **Nunca** aplique 5 V nos GPIOs (lembre da Aula 02). Aqui só usamos 3,3 V
  e GND com o botão e o LED (com resistor).
- O **LED** precisa de um resistor em série (você vai ver qual usar no passo
  do LED). Ligá-lo direto, sem resistor, pode queimar o LED e forçar o pino.
- Confira sempre o **pinout** da sua placa (Aula 02) para escolher GPIOs.
  Nesta aula vamos usar **GPIO 4** para o botão e **GPIO 2** (ou o do seu
  LED embutido) para o LED.

## Ligação

Nosso "caminho" de sinais:

```
 botão → ESP32 → LED
```

### Ligação do botão

A ligação do botão é muito simples — dois pontos:

```
ESP32
GPIO 4 ───── botão ───── GND
```

Ou seja: **um fio** vai do GPIO 4 até uma perna do botão, e **outro fio**
vai de outra perna do botão até o **GND** da placa.

> Sobre os botões de 4 pernas: eles internamente conectam as pernas de dois
> em dois. Se ficar em dúvida, conecte um fio em **uma** perna de um lado e
> o outro fio em **uma** perna do lado oposto. Teste com o monitor — se não
> funcionar, vire o botão em 90° e tente de novo.

Vamos deixar o LED para depois. Primeiro, só o botão.

## Primeiro teste

Vamos fazer o **menor teste possível**: ler o botão e mostrar **na tela do
computador** se ele está pressionado ou solto. Para isso usamos o **Monitor
Serial** (Aula 00) e a função **digitalRead()** (que vamos explicar já já).

## Código

Abra um sketch novo e digite:

```cpp
// Aula 03 — Lendo um botão

const int BOTAO = 4;

void setup() {
  Serial.begin(115200);
  pinMode(BOTAO, INPUT_PULLUP);
}

void loop() {
  int estado = digitalRead(BOTAO);

  if (estado == LOW) {
    Serial.println("Pressionado!");
  } else {
    Serial.println("Solto");
  }

  delay(200);
}
```

Depois de digitar, **compile e envie** (Aula 01). Em seguida, **abra o
Monitor Serial** (Aula 00) para ver as mensagens.

## Entendendo o código

Vamos ler o programa devagar, reaproveitando o que já sabemos.

```cpp
const int BOTAO = 4;
```

Criamos um **nome fácil para o número do pino**. Em vez de espalhar o número
`4` pelo código, damos a ele o nome `BOTAO`. O `const int` diz que é um
**número inteiro fixo** (que não vai mudar). Isso deixa o código mais legível
e fácil de ajustar depois.

```cpp
void setup() {
  Serial.begin(115200);
  pinMode(BOTAO, INPUT_PULLUP);
}
```

No `setup()`, duas coisas:

- **`Serial.begin(115200)`** — abre a "linha de conversa" com o computador
  (o Monitor Serial). O número `115200` é a **velocidade** dessa conversa; o
  Monitor Serial deve estar com a mesma velocidade. (Detalhes na Aula 07.)
- **`pinMode(BOTAO, INPUT_PULLUP)`** — aqui está o pulo do gato. Preparamos o
  GPIO 4 como **entrada** e pedimos a **PULLUP** (vamos entender já).

```cpp
void loop() {
  int estado = digitalRead(BOTAO);
```

Dentro do `loop()`:

- **`digitalRead(BOTAO)`** lê o estado do pino e devolve **`HIGH`** ou
  **`LOW`** — ou seja, diz se aquele pino está recebendo "ligado" ou
  "desligado". É o oposto do `digitalWrite` da Aula 01: aqui a placa **lê**
  em vez de **escrever**.
- Guardamos esse resultado numa **variável** chamada `estado`. Uma variável
  é uma "caixinha" onde o programa guarda um valor para usar depois. `int`
  diz que ela guarda um número inteiro.

```cpp
  if (estado == LOW) {
    Serial.println("Pressionado!");
  } else {
    Serial.println("Solto");
  }
```

Aqui está o **`if`** — uma condição. Antes da sintaxe, a ideia humana:

> "**Se** o botão estiver pressionado, então faça isto; **senão**, faça
> aquilo outro."

Agora sim, a sintaxe:

- **`if (estado == LOW)`** — "`if`" é "se". Os parênteses contêm a condição.
  O `==` (dois sinais de igual!) significa **"é igual a?"**. Cuidado: `==` é
  comparação; `=` (um só) é *atribuição* (guardar valor). Aqui perguntamos:
  "o estado **é igual a** LOW?"
- **`Serial.println("...")`** — imprime (mostra) um texto no Monitor Serial
  e **pula para a próxima linha** (por isso "ln", de *line*).
- O bloco `{ ... }` do `if` executa se a condição for verdadeira.
- **`else`** — "senão". O bloco dele executa se a condição for **falsa**.

Resumindo: se o estado é LOW, mostra "Pressionado!"; senão, mostra "Solto".

```cpp
  delay(200);
```

Uma pausa de 0,2 segundo para não inundar o Monitor Serial de mensagens.

Mas espere: **por que "pressionado" aparece quando o estado é LOW?** Isso
parece ao contrário, e é exatamente o que vamos desvendar agora.

### A inversão importante (INPUT_PULLUP)

Com **INPUT_PULLUP**, o comportamento é invertido em relação ao que você
imagina:

```
botão SOLTO       =  HIGH  (lê "ligado")
botão PRESSIONADO =  LOW   (lê "desligado")
```

Vamos entender por quê, sem aprofundar eletrônica.

### Por que a entrada não deve ficar "solta"

Imagine um pino de entrada sem nada ligado nele. Esse pino fica "solta" —
sem uma referência clara. Nesse caso, ele pode **puxar sinais aleatórios do
ambiente** e ler valores que **mudam sozinhos** (interferência elétrica que
existe em todo lugar). O programa ficaria "maluco", sem lógica.

Uma **entrada solta** é um problema: o valor lido é **imprevisível**.

### O que o INPUT_PULLUP faz

Para resolver, ativamos um **resistor interno** da própria placa — o
**PULLUP**. Esse resistor "puxa" o pino para cima (para HIGH / 3,3 V) quando
nada está forçando o contrário.

Então, com o botão **solto**:

```
GPIO ─── pullup interno ─── HIGH (3,3 V)
GPIO ─── botão ABERTO (não conecta nada)     → pino lê HIGH
```

Quando você **pressiona o botão**, ele **fecha o caminho** entre o GPIO e o
**GND**:

```
GPIO ─── botão FECHADO ─── GND (0 V)
         (o GND "puxa" o pino para baixo)    → pino lê LOW
```

Por isso a inversão: o pino "começa" em HIGH (via pullup), e **pressionar**
liga o GPIO ao GND, derrubando para LOW. É por isso que:

```
botão SOLTO       = HIGH   = "Pressionado!" não; é "Solto"
botão PRESSIONADO = LOW    = "Pressionado!"
```

Esse é o ponto que **confunde todo iniciante** — mas agora você entende o
motivo, então não precisa decorar: basta lembrar que o pullup segura HIGH, e
o botão, ao fechar para GND, derruba para LOW.

## O que deve acontecer

Com o **Monitor Serial** aberto (na velocidade `115200`), você deve ver:

- com o botão **solto**: mensagens "Solto";
- ao **pressionar** o botão: mensagens "Pressionado!".

Quando soltar, volta a "Solto". Se vê isso alternando, você conseguiu **ler
um botão** com o ESP32 — sua primeira *entrada* funcionando.

> 💡 Pequena nota sobre "tremedeira": fisicamente, ao pressionar/soltar, os
> contatos do botão podem "tremelicar" por milissegundos, fazendo o valor
> pular algumas vezes antes de estabilizar (chamamos isso de **bounce**).
> Neste exemplo é imperceptível. Aprenderemos a tratá-lo numa aula futura.

## Controlando um LED

Agora o projeto prático: o botão vai **controlar um LED**. Quando estiver
**pressionado**, o LED acende; quando **solto**, o LED apaga.

### Ligação do LED

Adicione o LED ao circuito. Lembre-se da Aula 02: o LED precisa de um
**resistor em série** para limitar a corrente e não queimar.

```
ESP32
GPIO 2 ───── resistor ───── LED ───── GND
```

> Use o resistor adequado para o seu LED (um valor comum para LEDs em 3,3 V
> é algo em torno de **220 a 330 ohms**). Se não souber, comece com um valor
> maior e segure os 3,3 V — o importante é **usar um resistor**.

### Código do LED

```cpp
// Aula 03 — Botão controlando um LED

const int BOTAO = 4;
const int LED = 2;

void setup() {
  pinMode(BOTAO, INPUT_PULLUP);
  pinMode(LED, OUTPUT);
}

void loop() {
  int estado = digitalRead(BOTAO);

  if (estado == LOW) {
    digitalWrite(LED, HIGH);
  } else {
    digitalWrite(LED, LOW);
  }
}
```

> Troque `LED` para o pino do seu LED embutido se necessário (Aula 02).

### Entendendo a parte nova

- **`const int LED = 2;`** — outro nome de pino: o LED.
- **`pinMode(LED, OUTPUT)`** — o LED é **saída** (lembra da Aula 01?).
- No `loop()`, a mesma leitura do botão (entrada). Depois:
  - se **pressionado** (`estado == LOW`): `digitalWrite(LED, HIGH)` — **liga**
    o LED;
  - se **solto** (`else`): `digitalWrite(LED, LOW)` — **desliga** o LED.

Repare que agora temos **entrada e saída juntas** no mesmo programa: o GPIO 4
escuta o botão, e o GPIO 2 comanda o LED.

## Se não funcionar

- **O Monitor Serial não mostra nada**: confira a velocidade (`115200` no
  canto) e se a **porta** certa está selecionada (Aula 00). Pode ser preciso
  apertar o botão **EN** da placa para reiniciar e ver mensagens novas.
- **Mostra "Solto" ou "Pressionado!" o tempo todo, sem mudar**: provavelmente
  o botão não está fechando o circuito. Confira a **ligação** (GPIO 4 → botão
  → GND) e tente **virar o botão** em 90° (nos botões de 4 pernas, a
  conexão interna depende da posição).
- **A leitura fica "pulando" ao pressionar**: isso é normal (o *bounce*
  citado acima). Não é erro — trataremos depois.
- **O LED não acende no projeto**: confira se o LED está com o resistor e a
  polaridade certa (a perna longa do LED vai para o lado do pino de energia;
  a curta, para o GND), e se o pino do LED está correto.
- **Nada funciona**: volte ao pinout (Aula 02) e confira os GPIOs escolhidos.

## Experimente você

Agora, pequenos desafios para fixar:

1. **Ritmo de leitura**: mude o `delay(200)` para `delay(50)` no primeiro
   teste e veja como as mensagens no Monitor Serial ficam mais rápidas.
2. **Inversão mental**: explique com as suas palavras por que, com
   INPUT_PULLUP, pressionar o botão deixa o pino em LOW (dica: o botão fecha
   o caminho para o GND).
3. **LED sem pressionar**: inverta o comportamento: o LED deve ficar **aceso
   quando o botão está SOLTO** e apagar quando está pressionado. (Dica: troque
   o `HIGH` e o `LOW` dentro do `if`/`else`.)

## O que aprendemos

Nesta aula:

- **entrada** é quando o ESP32 *escuta*; **saída** é quando ele *manda*;
- **digitalRead()** lê um pino e devolve **HIGH** ou **LOW**;
- um pino de entrada **não deve ficar solto** (valor imprevisível);
- **INPUT_PULLUP** liga o resistor interno que segura o pino em **HIGH**;
- com pullup: **solto = HIGH**, **pressionado = LOW** — a inversão faz
  sentido porque o botão fecha o caminho para o GND;
- **if/else** executa código conforme uma condição (`==` compara valores);
- com entrada (botão) e saída (LED) juntas, controlamos o LED pelo botão.

## Próximo passo

O botão funciona — mas o nosso programa fica "preso" esperando enquanto usa
`delay()`, o que atrapalha quando queremos fazer **várias coisas ao mesmo
tempo**. Vamos resolver isso na próxima aula.

Nos vemos na [Aula 04](../04-fazendo-mais-de-uma-coisa/README.md).
