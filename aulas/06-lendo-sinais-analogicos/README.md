# Aula 06 — Lendo sinais analógicos

## O que vamos fazer

Até agora controlamos **saídas** e lemos uma **entrada digital** (o botão,
que só dizia ligado/desligado). Nesta aula vamos dar um passo além: vamos ler
um sinal que **pode variar entre 0 V e 3,3 V**, e transformá-lo em um número
que o programa entende.

Para isso, usaremos um **potenciômetro** — aquela peça com um eixo que você
gira (comum em controles de volume e dimmers). Ao girar, ele varia a tensão
em um GPIO, e o ESP32 mostra o número correspondente no Monitor Serial.

Por que isso é útil? Muitos sensores do mundo real **variam** em vez de só
ligar/desligar: temperatura, luz, distância, posição. Saber ler esse tipo de
sinal abre um mundo novo de projetos.

## O que você vai aprender

No final desta aula, você vai entender:

- a diferença entre um sinal **digital** (HIGH ou LOW) e um **analógico**
  (varia entre 0 e 3,3 V);
- que um **potenciômetro** gera valores intermediários;
- o que é o **ADC** e como ele "mede tensão e devolve um número";
- como ligar o potenciômetro de forma segura (3V3, GND e um GPIO);
- como ler com **`analogRead()`**;
- como ver os valores no **Monitor Serial**;
- que **pequenas oscilações** nos valores são normais;
- **nunca aplicar mais que 3,3 V** em um GPIO.

## O problema: nem tudo é ligado/desligado

Um botão é um sinal **digital**: ou está **HIGH** (pressionado) ou **LOW**
(solto). São apenas dois valores.

Mas imagine que você gire um controle de volume. Não dá para dizer "ligado"
ou "desligado" — existe uma **faixa inteira** entre o volume mínimo e o
máximo. É um sinal **analógico**, que pode assumir **muitos valores** entre
0 V e 3,3 V.

O potenciômetro é perfeito para demonstrar isso: ele é uma peça que, ao ser
girada, divide uma tensão e entrega valores intermediários.

## O que é o ADC

O ESP32 não entende "tensão" diretamente — ele entende **números**. Então
precisa de um tradutor.

O **ADC** (sigla de *Analog-to-Digital Converter*, "conversor analógico para
digital") é exatamente esse tradutor. Pense nele como um **medidor de tensão
que devolve um número**:

```
tensão (0 a 3,3 V)  →  ADC  →  número (0 a 4095 no ESP32 comum)
```

- Se o pino está em **0 V**, o ADC devolve um número **baixo**;
- se está em **3,3 V**, devolve um número **alto**;
- se está em um valor intermediário, devolve um número no meio.

Na prática, para ler, usamos a função **`analogRead(pino)`**, que faz essa
medição e nos devolve o número.

> Não vamos aprofundar agora a "resolução" do ADC (quantos números ele
> produz) — só precisamos saber que a leitura varia de um valor baixo a um
> valor alto conforme mudamos a tensão. O número exato não é o foco: o
> **foco** é perceber que o valor **muda** quando giramos.

## Ligação

Vamos ligar o potenciômetro. Ele tem **três pernas**:

- a **primeira** perna → **3V3** (energia);
- a **última** perna → **GND** (retorno);
- a **perna do meio** → um **GPIO** de leitura (o pino de leitura analógica).

```
ESP32
3V3  ─────────────────── perna 1 do potenciômetro
GPIO (ADC) ───────────── perna do meio do potenciômetro
GND  ─────────────────── última perna do potenciômetro
```

> O pino de leitura analógica **depende da sua placa**. No ESP32 clássico,
> por exemplo, o **GPIO 34** pode ler analógico; em outras variantes os pinos
> mudam. **Consulte o pinout da sua placa (Aula 02)** e escolha um pino que o
> pinout indique como leitura analógica (ADC). Vamos usar **GPIO 34** no
> código, mas ajuste ao seu modelo.

## Antes de ligar

- **Desligue o cabo USB** antes de montar o circuito, e reconecte só depois.
- **Nunca conecte mais de 3,3 V em um GPIO.** O potenciômetro aqui recebe
  apenas **3V3** da própria placa, então está seguro — mas não leve energia
  externa para os pinos.
- Confira no **pinout** se o GPIO escolhido é adequado para leitura
  analógica na sua placa.

## Primeiro teste

Nosso primeiro teste é o menor possível: girar o potenciômetro e **ver o
número mudar no Monitor Serial**. Assim confirmamos que o ADC e a ligação
estão funcionando.

## Código

Abra um sketch novo e digite:

```cpp
// Aula 06 — Lendo o potenciômetro com ADC

const int PINO_ADC = 34;

void setup() {
  Serial.begin(115200);
}

void loop() {
  int valor = analogRead(PINO_ADC);
  Serial.println(valor);
  delay(200);
}
```

Compile, envie e abra o Monitor Serial (na velocidade `115200`).

## Entendendo o código

```cpp
const int PINO_ADC = 34;
```

Damos um **nome** ao pino de leitura (Aula 03). Ajuste `34` para o GPIO de
ADC da sua placa.

```cpp
Serial.begin(115200);
```

Abre a **conversa** com o computador (Monitor Serial), como na Aula 03 e
como veremos mais a fundo na Aula 07.

```cpp
int valor = analogRead(PINO_ADC);
```

Aqui está a novidade: **`analogRead(pino)`** mede a tensão no pino e devolve
o número. Guardamos esse número na variável `valor`.

```cpp
Serial.println(valor);
```

Mostra o número no Monitor Serial (e pula de linha).

```cpp
delay(200);
```

Uma pausa de 0,2 segundo para não inundar a tela de mensagens.

## O que deve acontecer

Gire o eixo do potenciômetro **completamente** em um sentido, depois no
outro. No Monitor Serial, o número deve:

- **diminuir** até um valor baixo em um extremo;
- **aumentar** até um valor alto no outro extremo;
- **mudar suavemente** à medida que você gira pelo meio.

Ou seja: o valor acompanha o giro do potenciômetro. Isso prova que o ESP32
está **lendo uma tensão analógica** e a transformando em número.

> Você pode notar que, mesmo parado, o número **oscila levemente** (ex.:
> muda alguns dígitos para cima e para baixo). Isso é **normal** — a medição
> tem pequenas variações elétricas. Não é erro do seu circuito.

## Se não funcionar

- **O Monitor Serial não mostra nada**: confira a velocidade (`115200`) e a
  porta (Aula 00). Pode ser preciso apertar o botão EN da placa.
- **O valor não muda ao girar**: confira a ligação. As três pernas do
  potenciômetro devem ir para 3V3, GPIO e GND, nessa ordem (a do meio sempre
  no GPIO). Se trocar duas das externas, o giro continuará mudando o valor,
  mas invertido — o que é aceitável. Se **nada** muda, verifique os fios.
- **O valor fica sempre no máximo ou sempre no mínimo**: o pino pode estar
  mal conectado, ou você está lendo um pino que não é de ADC na sua placa.
  Confira o pinout.
- **Valores saltam muito**: confira se não há fio solto. Um pouquinho de
  oscilação é normal, mas saltos enormes indicam mau contato.

## Experimente você

Agora, dois desafios para fixar:

1. **Observe os extremos**: gire o potenciômetro até o máximo e anote o
   maior número que vê; gire até o mínimo e anote o menor. Compare com
   outro aluno/colega de kit (podem variar um pouco conforme a placa).
2. **Controle o brilho PWM com o potenciômetro**: use o `analogRead` da
   Aula 06 junto com o `analogWrite` da Aula 05. The potenciômetro define o
   brilho do LED. Monte o LED (GPIO + resistor + LED + GND, da Aula 05) e
   gire o potenciômetro para acender/apagar o LED gradualmente.

   Dica: os números do ADC e do `analogWrite` têm faixas diferentes. Para
   ligá-los, você pode usar `map()` — uma função que "converte" uma faixa de
   números em outra. Exemplo:

   ```cpp
   int leitura = analogRead(PINO_ADC);
   int brilho = map(leitura, 0, 4095, 0, 255);
   analogWrite(LED, brilho);
   ```

   Não se preocupe em dominar `map()` agora — só use o exemplo acima e gire
   o potenciômetro para ver o LED mudando de brilho.

## O que aprendemos

Nesta aula:

- sinais **digitais** têm dois valores (HIGH/LOW); sinais **analógicos**
  variam entre 0 V e 3,3 V;
- um **potenciômetro** gera valores intermediários conforme o giramos;
- o **ADC** é o "medidor de tensão que devolve um número";
- ligamos o potenciômetro em **3V3, um GPIO de leitura e GND**;
- **`analogRead(pino)`** lê a tensão e devolve um número;
- valores podem **oscilar levemente** — normal;
- **nunca** aplicar mais que 3,3 V em um GPIO.

## Próximo passo

Até aqui o Monitor Serial apareceu várias vezes para ver números. Vamos dar
um passo atrás e **entender essa ferramenta a fundo** — como usá-la para
diagnosticar problemas e "enxergar" o que o programa está pensando.

Nos vemos na Aula 07.
