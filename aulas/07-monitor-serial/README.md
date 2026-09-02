# Aula 07 — Monitor Serial: a janela do ESP32

## O que vamos fazer

Várias aulas usaram o **Monitor Serial** para ver números e mensagens. Nesta
aula vamos parar e entender essa ferramenta a fundo — porque ela é, na
prática, **os "olhos" e "ouvidos" do ESP32** quando algo não funciona.

Por que isso é útil? Quando o seu programa faz algo errado, o ESP32 **não tem
tela** para contar o que está pensando. O Monitor Serial é o jeito mais
simples de "fazer perguntas" ao programa e descobrir onde ele está falhando.
Aprender a usá-lo bem economiza muita frustração.

## O que você vai aprender

No final desta aula, você vai entender:

- que o **ESP32 não tem tela** própria para mostrar o que pensa;
- o que faz **`Serial.begin()`**;
- como mostrar texto e valores com **`Serial.print()`** e
  **`Serial.println()`**;
- como **exibir o valor de um botão** e de uma **leitura analógica**;
- como colocar **mensagens de diagnóstico** ("entrei aqui", "conectado",
  "erro") no programa;
- que **diagnóstico é uma prática normal** de programação;
- que **excesso de mensagens** também pode atrapalhar;
- um pequeno **exercício de depuração**.

## Problema: o ESP32 não tem tela

Um computador tem monitor. O ESP32, não. Quando o programa roda, você **não
vê** o que ele está pensando: os valores das variáveis, se uma condição
aconteceu, se entrou em determinada parte do código.

O **Monitor Serial** resolve isso: é uma **janela no computador** onde o
ESP32 pode **enviar mensagens**. É como um **fio de conversa** entre o
programa e você — ele "fala", e você lê.

```
ESP32 ─────USB─────> computador (Monitor Serial)
      "Olá, mundo"
      "botão = 1"
      "Wi-Fi conectado"
```

## Ligação

Nesta aula **não precisa de circuito**. Vamos usar apenas:

- a placa **ESP32**;
- o **cabo USB**;
- o **computador com o Arduino IDE**.

A "conexão" é só o cabo USB (como na Aula 00). Todo o trabalho é no código.

## Primeiro teste: "Olá"

Vamos começar com a mensagem mais clássica: o ESP32 dizendo "Olá".

```cpp
// Aula 07 — Primeira mensagem no Serial

void setup() {
  Serial.begin(115200);
  Serial.println("Ola, eu sou o ESP32!");
}

void loop() {
}
```

Compile, envie e abra o **Monitor Serial**. **Importante:** confira que a
velocidade no Monitor Serial está em **115200** (canto inferior direito).

### Entendendo o código

```cpp
Serial.begin(115200);
```

**`Serial.begin(número)`** **inicia** a comunicação com o computador. O
número é a **velocidade da conversa** (em "bits por segundo"). O código e o
Monitor Serial precisam usar **o mesmo número** — se um usa 115200 e o outro
9600, as mensagens aparecem como "garranchos".

Pense em duas pessoas combinando de falar no mesmo ritmo: se uma fala devagar
e a outra rápido, ninguém se entende.

```cpp
Serial.println("Ola, eu sou o ESP32!");
```

**`Serial.println(texto)`** envia o texto ao computador e **pula de linha**
(o "ln" vem de *line*). Vamos ver em detalhe a diferença entre `println` e
`print` logo abaixo.

## Exibindo valores

De nada adianta só dizer "Olá". O verdadeiro poder é **mostrar os valores das
variáveis** em tempo real. Vamos exibir o valor de um **botão** (Aula 03) e
de uma **leitura analógica** (Aula 06).

### Exibindo o valor de um botão

Monte o botão da Aula 03 (GPIO → botão → GND, com INPUT_PULLUP) e use:

```cpp
// Aula 07 — Exibindo o valor do botão

const int BOTAO = 4;

void setup() {
  Serial.begin(115200);
  pinMode(BOTAO, INPUT_PULLUP);
}

void loop() {
  int estado = digitalRead(BOTAO);
  Serial.print("Botao: ");
  Serial.println(estado);
  delay(200);
}
```

### O que muda aqui

- **`Serial.print("Botao: ")`** — envia o texto **sem** pular de linha.
- **`Serial.println(estado)`** — envia o **valor da variável** e pula de
  linha.

Juntando os dois, a saída fica em uma linha só, por exemplo:

```
Botao: 1
Botao: 0
Botao: 1
```

O `estado` é 1 (HIGH, solto) ou 0 (LOW, pressionado) — é o mesmo número que
lia­mos na Aula 03, agora visível para conferir.

> Regra rápida: **`print`** escreve e continua na mesma linha; **`println`**
> escreve e **pula de linha** (por isso "ln"). Use `println` para cada
> "registro" completo, e `print` quando quiser montar uma linha com várias
> partes.

### Exibindo uma leitura analógica

Monte o potenciômetro da Aula 06 e use:

```cpp
// Aula 07 — Exibindo a leitura analógica

const int PINO_ADC = 34;

void setup() {
  Serial.begin(115200);
}

void loop() {
  int valor = analogRead(PINO_ADC);
  Serial.print("ADC: ");
  Serial.println(valor);
  delay(200);
}
```

Gire o potenciômetro e veja o valor mudando, agora **rotulado** na tela:

```
ADC: 512
ADC: 1800
ADC: 3500
```

Isso é muito mais legível do que um número solto — já sabemos o que ele
significa.

## O que deve acontecer

- No primeiro teste: a mensagem "Ola, eu sou o ESP32!" aparece **uma vez**
  (porque está no `setup()`, que roda uma única vez).
- No teste do botão: uma linha nova a cada 0,2 segundo mostrando `Botao: 1`
  ou `Botao: 0`, que **muda quando você aperta o botão**.
- No teste do potenciômetro: linhas `ADC: <número>` que **acompanham o giro**.

## Mensagens de diagnóstico

Além dos valores, o truque mais útil é colocar **mensagens que explicam onde
o programa está**. São os "marcadores" que ajudam a achar o bug.

```cpp
Serial.println("Entrei no loop!");             // sei que cheguei aqui
Serial.println("Botao pressionado!");         // esta condicao disparou
Serial.println("Wi-Fi conectado!");            // uma etapa terminou
Serial.println("Erro: leitura falhou.");       // algo deu errado
```

A ideia: coloque um `Serial.println` em pontos-chave. Se uma mensagem **não
aparece**, você descobre **até onde o programa chegou** — e o problema está
ali. Se aparece uma mensagem de **erro**, você sabe o que falhou.

Isso não é "artimanha de amador": **diagnosticar assim é uma prática normal**
de programação. Programadores profissionais fazem a mesma coisa o tempo todo.

## Cuidado: excesso de mensagens

Mais mensagens **não é** sempre melhor. Se o `loop()` roda milhares de vezes
por segundo e você imprime em todas, o Monitor Serial **enche de texto** e
fica impossível de ler — e o despejo de mensagens ainda **atrasa o programa**
(transmitir pelo cabo custa tempo).

Por isso usamos **`delay()`** entre as impressões, ou imprimimos **só quando
algo muda**. Regra prática: mostre o necessário, com moderação.

## Primeiro teste de depuração

Vamos praticar. O exercício é **encontrar um problema**: um programa que
deveria acender um LED quando o botão for pressionado, mas não acende.

Monte o botão (GPIO 4) e o LED (GPIO 2 com resistor). Rode este programa:

```cpp
// Aula 07 — Programa com um problema proposital

const int BOTAO = 4;
const int LED = 2;

void setup() {
  Serial.begin(115200);
  pinMode(BOTAO, INPUT_PULLUP);
  pinMode(LED, OUTPUT);
  Serial.println("Inicio do setup.");
}

void loop() {
  int estado = digitalRead(BOTAO);
  Serial.print("Estado do botao: ");
  Serial.println(estado);

  if (estado == LOW) {
    Serial.println("Botao pressionado, ligando LED.");
    digitalWrite(LED, HIGH);
  } else {
    Serial.println("Botao solto.");
    digitalWrite(LED, LOW);
  }

  delay(300);
}
```

**O que você deve observar:** as mensagens de diagnóstico mostram **o que o
programa acredita** que está acontecendo. Quando você pressiona o botão, ele
diz "Botao pressionado, ligando LED". Se mesmo assim o LED **não acender**,
as mensagens revelam que o problema **não está no código** — está na
**ligação** (talvez o LED esteja virado, ou faltando resistor, ou no pino
errado). A mensagem te guia para o problema real.

> Depuração, na prática, é **comparar o que o programa pensa com o que você
> espera**. As mensagens são a ponte entre os dois.

## Experimente você

Agora, alguns desafios de diagnóstico:

1. **Rotule mais**: no teste do potenciômetro, além do valor, mostre também a
   **porcentagem** aproximada (ex.: `int pct = map(valor, 0, 4095, 0, 100);`
   e imprima `"Porcentagem: "` + `pct`). Veja como fica mais fácil entender.
2. **Mensagem de "entrei aqui"**: coloque um `Serial.println("Inicio do
   loop.")` como primeira linha do `loop()` de qualquer programa e observe: a
   mensagem aparece várias vezes por segundo. Isso te mostra a **velocidade**
   de repetição.
3. **Encontre o bug**: escreva um programa que deveria mostrar "X" no Serial
   mas não mostra. Adicione mensagens de diagnóstico em etapas (antes e
   depois do ponto provável do problema) até achar onde ele está
   "desaparecendo".

## O que aprendemos

Nesta aula:

- o ESP32 **não tem tela**; o **Monitor Serial** é a janela para o que ele
  "pensa";
- **`Serial.begin(velocidade)`** inicia a conversa (use a mesma velocidade
  no código e no Serial);
- **`Serial.print()`** escreve na linha; **`Serial.println()`** escreve e
  pula de linha;
- podemos exibir **valores de variáveis** (botão, ADC) em tempo real;
- **mensagens de diagnóstico** ("entrei aqui", "erro") ajudam a achar bugs;
- diagnóstico **é prática normal**, mas **excesso** de mensagens atrapalha.

## Próximo passo

Agora que sabemos conversar com o ESP32 e mostrar o que ele pensa, vamos
aprender a fazer o ESP32 conversar com **outros dispositivos** usando um
barramento chamado **I2C** — que permite conectar vários componentes com
poucos fios.

Nos vemos na Aula 08.
