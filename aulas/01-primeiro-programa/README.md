# Aula 01 — Nosso primeiro programa

## O que vamos fazer

Vamos escrever nosso **primeiro programa do zero**. Ele vai fazer a placa
ESP32 **acender e apagar um LED** — a famosa "luz que pisca" (o *Blink*).

Por que isso é útil? Porque o Blink é o "Olá, Mundo" da eletrônica: é a
primeira prova de que você conseguiu escrever código, enviar para a placa e
ver um resultado **físico** (uma luz piscando). Tudo que virá no curso —
botões, sensores, Wi-Fi — começa com essas mesmas ferramentas: mandar
energia para um pino e esperar um tempinho.

Na Aula 00 você rodou o exemplo Blink pronto. Agora vamos **escrever o
nosso**, linha por linha, e entender exatamente o que acontece em cada etapa
do processo — inclusive o que é "compilar" e que tipo de erro pode surgir.

## O que você vai aprender

Nesta aula você vai entender o **caminho completo** de um programa:

- o que é um **sketch** (o "rascunho" que escrevemos);
- a estrutura mínima: **setup()** e **loop()**;
- o que significa o programa "começar" e depois **ficar repetindo** o loop;
- como identificar o **LED** que vamos usar (embutido ou externo);
- o mínimo necessário sobre **GPIO**;
- **pinMode()** para dizer que o pino será uma saída;
- **digitalWrite()** para ligar e desligar;
- o significado de **HIGH** e **LOW**;
- **delay()** como "espere tantos milissegundos";
- **milissegundos** na prática: 1000 ms = 1 segundo;
- o que é **compilar** e por que pode aparecer erro;
- como fazer **upload** para o ESP32;
- observar o **resultado físico**;
- fazer a **primeira modificação** e ver a diferença;
- um **erro proposital** que não trava a compilação mas impede o resultado.

Parece muita coisa, mas cada item aparece em um momento só, devagar.

## Material necessário

- A placa **ESP32** (a mesma da Aula 00);
- o **cabo USB**;
- o **computador com o Arduino IDE** já configurado.

O **LED** que vamos usar pode vir de duas formas — e vamos descobrir qual é
o seu caso na seção seguinte:

- **LED embutido**: a maioria das placas ESP32 já traz um LED pequeno
  soldado, ligado a um pino específico (em muitas placas comuns, o **GPIO
  2**).
- **LED externo**: se a sua placa não tiver, usaremos um LED comum soldado
  num pino que você escolher.

No final desta aula, você terá decidido qual usar.

## Antes de ligar

- Conecte a placa apenas pelo **cabo USB ao computador**. Ainda não vamos
  montar nenhum circuito externo.
- Se uma luz azul piscar na placa quando você conectar ou enviar código,
  **tudo bem** — algumas placas têm um LED de status que pisca durante a
  comunicação. Não é um erro.
- Se você for usar um **LED externo** agora, espere: vamos montar isso
  **depois** de entender o código. Um passo de cada vez.

## Ligação

Nesta primeira parte, a "ligação" é só o cabo USB, como na Aula 00:

```
[Computador]  —USB—>  [Placa ESP32]
```

- Se usar **LED embutido**: não precisa conectar nada além do cabo. O LED já
  está lá, dentro da placa.
- Se usar **LED externo**: faremos a ligação na seção "Experimente você",
  porque queremos que você entenda o código antes.

Agora, o trabalho é todo no **código**.

## Primeiro teste

Nosso primeiro teste é simplesmente **criar o programa e enviá-lo**. Vamos:

1. escrever o código;
2. **compilar** (entenderemos o que isso significa);
3. fazer **upload**;
4. olhar para a placa e **ver o LED piscar**.

Cada etapa é explicada abaixo, no seu momento.

## Código

Vamos abrir um arquivo novo no Arduino IDE (*Arquivo > Novo*, e apague o
que vier pronto). Esse arquivo é o que chamamos de **sketch** — um termo em
inglês para "rascunho". É um arquivo de texto onde escrevemos as instruções
para a placa. Todo sketch Arduino tem essa mesma cara.

Digite o programa abaixo (comente `2` se o seu LED embutido estiver em
outro pino, conforme explicaremos):

```cpp
// Aula 01 — Nosso primeiro programa
// Pisca um LED da placa ESP32

void setup() {
  pinMode(2, OUTPUT);
}

void loop() {
  digitalWrite(2, HIGH);
  delay(1000);
  digitalWrite(2, LOW);
  delay(1000);
}
```

> 💡 Se o seu LED interno **não** estiver no GPIO 2, troque os **quatro**
> números `2` pelo pino correto. Em muitas placas comuns, é 2 mesmo.
> Vamos explicar por que usamos o 2 e como descobrir o seu a seguir.

Antes de enviar, precisamos **compilar**. Vamos entender isso agora.

## Entendendo o código

### O sketch e a estrutura mínima

Nosso sketch tem exatamente **duas funções** obrigatórias, e todo programa
Arduino precisa delas: **`setup()`** e **`loop()`**.

Pense no `setup()` como a **"preparação do palco"**, e no `loop()` como o
**"corpo da música"**.

```cpp
void setup() {
```

**`setup()`** é a primeira parte. O que está entre as chaves `{` e `}` roda
**uma única vez**, logo que a placa liga. Aqui a gente prepara as coisas —
como dizer ao pino o que ele vai fazer.

```cpp
  pinMode(2, OUTPUT);
```

**`pinMode`** significa "modo do pino". Ele diz à placa o que o pino vai
fazer: **`OUTPUT`** (saída) significa "pode mandar energia para fora". Com
isso, dizemos que o pino 2 será uma **saída** — servirá para acender algo.

```cpp
}
```

Fecha o `setup()`. Ele rodou uma vez e acabou, ao ligar a placa.

```cpp
void loop() {
```

**`loop()`** é a segunda parte. O que está aqui **se repete sem parar**,
enquanto a placa estiver ligada. Depois que o `setup()` "prepara o palco",
é o `loop()` que faz a música tocar, repetindo a sequência infinitamente.

```cpp
  digitalWrite(2, HIGH);
```

**`digitalWrite`** significa "escrever digital": mandar um valor digital para
um pino. Aqui, mandamos **`HIGH`** para o pino 2.

**`HIGH`** quer dizer "ligado". Em inglês, *alto* — é a tensão máxima que o
pino fornece (no ESP32, 3,3 V). Ou seja: **acenda o LED**.

```cpp
  delay(1000);
```

**`delay`** significa "atraso" — é como dizer **"espere tantos
milissegundos"**. O número entre parênteses é o tempo de espera, em
**milissegundos**, que são milésimos de segundo. `delay(1000)` espera **1000
milissegundos = 1 segundo**.

Na prática, isso é só conversão:
- `delay(1000)` → espera **1 segundo**;
- `delay(500)` → espera **meio segundo**;
- `delay(200)` → espera **0,2 segundo**.

```cpp
  digitalWrite(2, LOW);
```

Agora mandamos **`LOW`** para o pino 2. **`LOW`** quer dizer "desligado" (a
tensão mínima, 0 V). Ou seja: **apague o LED**.

```cpp
  delay(1000);
}
```

Outra espera de 1 segundo, e fecha o `loop()`. Como o `loop()` se repete, a
sequência fica: **acende, espera 1s, apaga, espera 1s, acende, espera...** —
o LED pisca eternamente. 🎉

### E o que é "GPIO"?

Aqui, só o necessário: **GPIO** é a sigla em inglês de *General Purpose
Input/Output* — "entrada e saída de uso geral". São os pinos metálicos da
placa, como os "braços" dela.

Nesta aula, só usamos uma saída: o pino 2 manda energia para fora, ligando e
desligando o LED. O aprofundamento sobre entrada fica para as Aulas 02 e 03.

### Compilar: o que é e por que pode dar erro

Antes de enviar para a placa, o Arduino IDE precisa **compilar** o sketch.
"Compilar" é **traduzir** o texto que escrevemos (que um humano entende)
para uma linguagem que o chip entende (os "zeros e uns" do microcontrolador).

É como traduzir um texto de português para outra língua: se houver um erro de
**escrita**, a tradução falha. No código, erros de escrita são as coisas
como nome errado de função, falta de ponto e vírgula `;` ou parênteses
incompletos. Quando isso acontece, o Arduino IDE mostra uma **mensagem de
erro** na parte de baixo — costuma ter um texto vermelho.

Erros de compilação aparecem **antes** de mandar para a placa. Se aparecer
um, leia a mensagem: o Arduino IDE costuma dizer até a linha do problema.
Corrija, e compile de novo.

Se **não** aparecer erro, a compilação deu certo e estamos prontos para o
próximo passo.

### Upload: enviar para o ESP32

Depois de compilar sem erros, fazemos o **upload** — é o botão da seta para
a direita, como na Aula 00. "Upload" significa **enviar o programa
traduzido** para a placa pelo cabo USB.

Para o upload funcionar, lembre-se de ter a **placa** e a **porta**
corretas selecionadas (como configuramos na Aula 00). Durante o envio, a
placa reinicia e grava o programa na memória dela. Quando termina, o
`setup()` roda uma vez e o `loop()` começa a repetir — ou seja, **o LED
começa a piscar**.

### Observar o resultado físico

É aqui que a mágica acontece: olhe para a placa. Deve haver um LED pequeno
piscando. Esse é o **resultado físico**: algo que acontece no mundo real
por causa das suas instruções.

Se você vê o LED piscando a cada segundo, o programa funcionou do começo ao
fim: escrever, compilar, enviar e ver o resultado.

## O que deve acontecer

Depois do *Upload*, o LED da placa deve ficar **piscando** em um ritmo
constante: **1 segundo aceso, 1 segundo apagado** (por causa do
`delay(1000)`).

Se você vê isso, fez seu primeiro programa funcionar de verdade.

## Se não funcionar

- **Aparece um erro de compilação (texto vermelho)**: o programa não foi
  traduzido. Leia a mensagem e a linha citada. Conferir ponto e vírgula,
  parênteses e nomes de função. Após corrigir, compile de novo.
- **O programa compila, mas o LED não pisca**: o código está "correto" para
  o compilador, mas o **circuito** não faz o esperado. A causa mais comum é
  o **pino errado**: o seu LED embutido pode estar em outro GPIO (não o 2).
  Verifique a documentação da sua placa e troque os números `2`.
- **Erro ao fazer upload**: confirme a placa e a porta corretas (menu de
  placas e porta, da Aula 00). Em algumas placas, aperte o botão **BOOT**
  durante o envio.
- **O LED pisca forte, mas rápido ou lento demais**: ajuste os números
  dentro de `delay()`. Lembre: `1000` = 1 segundo.
- **Não vejo LED nenhum**: algumas placas têm LEDs pequenos e discretos.
  Olhe de perto. Se nada acender mesmo, verifique se sua variante tem LED
  interno — talvez você precise de um LED externo (Aula seguinte).

## Experimente você

Agora vamos mexer sozinho, em três passos. O objetivo é sentir como uma
pequena mudança no código muda o comportamento físico.

### Passo 1 — Acelere o pisca

Troque os dois `delay(1000)` por `delay(200)`. Compile e envie.

O que acontece? O LED agora pisca **muito mais rápido** — 0,2 segundo aceso,
0,2 segundo apagado. Você sentiu na prática que **`delay` controla a
velocidade**.

### Passo 2 — Um erro proposital (que passa na compilação!)

Agora o mais importante desta aula. Troque **só o primeiro** pino `2` no
`setup()` (o `pinMode(2, ...)`) por um número alto que sua placa não usa,
por exemplo `33`:

```cpp
void setup() {
  pinMode(33, OUTPUT);   // PINO ERRADO DE PROPOSITO
}

void loop() {
  digitalWrite(2, HIGH);
  delay(200);
  digitalWrite(2, LOW);
  delay(200);
}
```

Compile e envie. Repare: **a compilação passa sem erro!** O Arduino IDE não
reclama, porque `33` é um número válido de pino para ele.

Mas o LED… **não pisca**. Por quê? Porque definimos a **saída** no pino 33,
mas mandamos o sinal (HIGH/LOW) para o **pino 2**. O pino 2 nunca foi
configurado, e o pino 33 nunca recebeu comando. Resultado: código "correto"
para o compilador, mas comportamento errado no mundo real.

Isso é um **erro de lógica**, não de sintaxe. Ele é o tipo de erro mais
comum e traiçoeiro do curso: o computador não o pega, **só nós**, olhando
com atenção, conseguimos encontrar. Este é exatamente o tipo de bug que o
Monitor Serial (Aula 07) ajuda a caçar.

Cole o `pinMode(2, OUTPUT)` de volta e veja o LED piscar de novo.

### Passo 3 — Desafio de ritmo

Faça o LED ficar aceso **muito tempo** e apagado **rapidinho**, criando uma
"piscada": tente um ritmo como aceso `delay(2000)` e apagado `delay(200)`.
Preveja o que vê antes de enviar, depois confirme.

## O que aprendemos

Nesta aula:

- o arquivo que escrevemos se chama **sketch**, e todo sketch tem **setup()**
  (roda uma vez) e **loop()** (repete para sempre);
- **GPIO** são os pinos da placa — aqui só precisamos de uma **saída**;
- **pinMode(pino, OUTPUT)** prepara o pino como saída;
- **digitalWrite(pino, HIGH/LOW)** liga e desliga o pino;
- **delay(ms)** espera o número de **milissegundos** (1000 ms = 1 s);
- **compilar** traduz o código e pode falhar por erros de escrita;
- **upload** envia o programa à placa;
- um erro pode **passar na compilação** e mesmo assim impedir o resultado —
  é o caso do **pino errado**.

## Próximo passo

Agora que ligamos e desligamos um pino, vamos **entender os pinos a fundo**
— quais usar, quais exigem cuidado e por que o ESP32 trabalha com 3,3 V.

Nos vemos na [Aula 02](../02-entendendo-os-pinos/README.md).
