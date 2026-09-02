# Aula 02 — Entendendo os pinos do ESP32

## O que vamos fazer

Na Aula 01 você aprendeu a ligar e desligar um pino com o programa Blink —
uma luz piscando. Mas a placa tem **vários pinos**, e eles não são todos
iguais. Alguns mandam energia, outros só recebem ordens, e alguns são
perigosos de mexer do jeito errado.

Nesta aula vamos **entender os pinos do ESP32**: o que cada tipo faz, sobre o
que devemos ter cuidado e como identificar qual é qual olhando para a placa.

Por que isso é útil? Porque **a maioria dos estragos e frustrações no curso
vêm daí**: ligar algo no pino errado, aplicar tensão demais, ou confundir um
GPIO com um simples ponto de energia. Entender isso agora evita quase todos
os problemas futuros.

## O que você vai aprender

No final desta aula, você vai entender:

- o que é um **GPIO**;
- a diferença entre **GPIO**, **3V3**, **5V** e **GND**;
- a diferença entre **INPUT** e **OUTPUT**;
- que os GPIOs do ESP32 trabalham com **3,3 V** (e não "aguentam" 5 V);
- que **HIGH** é aproximadamente **3,3 V** e **LOW** é aproximadamente **0 V**;
- que o **número do GPIO não é a posição física** do pino na placa;
- que alguns GPIOs têm **funções especiais** ou **restrições**;
- a importância de consultar o **pinout** da placa;
- o conceito básico de **GND comum**.

São muitos itens, mas vamos em um por vez, devagar.

## Material necessário

- A placa **ESP32** (a mesma de sempre);
- o **cabo USB**;
- o **computador com o Arduino IDE**;
- o programa **Blink** que já escrevemos na Aula 01;
- **opcional**: um multímetro (para o último passo, se você tiver um).

## Antes de ligar

Leia com atenção — esta é uma das aulas mais importantes para **não
danificar a placa**:

- **Nunca aplique 5 V em um GPIO.** Os pinos de sinal do ESP32 trabalham com
  no máximo 3,3 V. Aplicar 5 V pode **queimar** o pino (e às vezes a placa).
- **Não conecte os GPIOs a fontes de energia que você não conhece.** Quando
  não tiver certeza da origem de uma tensão, não ligue.
- Conecte a placa **apenas pelo cabo USB** nesta aula. Vamos mexer só no
  código e olhar a placa — nenhuma ligação externa ainda.
- O pino **5V** da placa **não** é um GPIO: é energia fornecida **pela**
  placa, e precisa de cuidado. Vamos explicar isso já já.

## Explicação passo a passo

Vamos construir o entendimento aos poucos. Uma ideia por vez.

### 1. O que é um GPIO

Você já viu essa sigla nas aulas anteriores. Aqui vamos aprofundar um
pouquinho, só o necessário.

**GPIO** = *General Purpose Input/Output* ("entrada e saída de uso geral").

Pense nos GPIOs como o **ponto de contato entre o programa e o mundo
físico**. É por eles que o seu código conversa com o mundo fora da placa:

```
        mundo físico        mundo do programa
  LED / botão / sensor   <->        GPIO        <->   seu código
```

- O GPIO é o **"canal"** que leva as instruções do programa para uma luz, um
  motor, um sensor.
- Neste momento, só nos importa saber que cada GPIO tem um **número**, e que
  esse número é como o programa chama esse pino (por exemplo, `GPIO 2`).

"Entrada e saída" se refere a duas direções — é o próximo ponto.

### 2. Alimentação não é sinal

Olhando para a placa, nem todos os pinos são "de conversa" (GPIO). Alguns
são de **alimentação** — são os "gasodutos" de energia da placa. Vamos
diferenciar quatro tipos:

```
NOME      O QUE É                                             USO
-----     --------------------------------------------------  ---------------
3V3       3,3 volts de energia SAINDO da placa                alimentar sensores
5V        5 volts de energia (entra ou sai, com cuidado)      casos especiais
GND       "terra" — o fio de RETORNO obrigatório              sempre precisa
GPIO      pino de SINAL — conversa com o programa             ligar/desligar etc.
```

Vamos entender cada um:

- **3V3**: a placa **fornece** 3,3 volts por esses pinos. São "fios de
  energia" para alimentar componentes como um sensor. **Não** é um GPIO —
  você **não** liga e desliga pelo programa.
- **5V**: também é energia, mas de 5 volts. Este pino tem duas vidas: em
  muitas placas ele **recebe** energia quando a placa é alimentada por uma
  entrada específica, mas quando a placa vem do USB ele também pode
  **fornecer** energia. É o pino mais fácil de causar confusão — por isso o
  tratamos como "casos especiais, com cuidado".
- **GND**: significa "ground" ("terra"). É o **fio de retorno** da
  eletricidade. Todo circuito **precisa** de um caminho de volta ao GND
  para a corrente circular. Vamos falar mais no item 9.
- **GPIO**: os pinos de **sinal**, de conversa com o programa. É deles que
  falamos o tempo todo.

A regra de ouro: **alimentação (3V3/5V) é para dar energia; GPIO é para
dar sinal.** São coisas diferentes — não confunda os pinos entre si.

### 3. Entrada e saída

Cada GPIO pode ser configurado para uma de duas direções. Isso é o coração
do "Input/Output" da sigla:

**Saída (OUTPUT)** — o ESP32 **manda** algo para fora:

```
   ESP32  ——sinal——>  dispositivo (ex.: LED)
```

**Entrada (INPUT)** — o ESP32 **recebe** algo de fora:

```
   ESP32  <——sinal——  dispositivo (ex.: botão)
```

- Na Aula 01, usamos **OUTPUT**: a placa mandou energia para acender o LED.
- Na Aula 03, vamos usar **INPUT**: a placa vai escutar se um botão foi
  pressionado.

A direção é escolhida no código, com `pinMode()`. Lembre do Aula 01:
`pinMode(2, OUTPUT)` prepara o pino 2 para **mandar** sinal.

### 4. O ESP32 trabalha com 3,3 V

Este é um ponto crucial e que poupa estragos:

Os GPIOs do ESP32 funcionam com **3,3 volts**. O "1" lógico (HIGH) é 3,3 V,
e quando um equipamento **não é tolerante a 5 V**, aplicar 5 V num pino de
sinal pode **danificá-lo permanentemente**.

Os GPIOs do ESP32 **não são tolerantes a 5 V**. Ou seja: **não jogue 5 V em
um GPIO.**

Traduzindo em conselho prático: **se o seu componente usa 5 V, ele não pode
ser ligado direto num GPIO do ESP32.** Você precisará de adaptação (coisa de
uma aula mais avançada). Por enquanto, a regra é: **mantenha os sinais em
3,3 V.**

> Compare com um computador: componentes internos têm uma tensão padrão.
> Aqui, o padrão do ESP32 é 3,3 V. Escolher um componente que trabalhe nessa
> mesma tensão evita problemas.

### 5. HIGH e LOW no mundo real

Quando programamos `digitalWrite(2, HIGH)` e `digitalWrite(2, LOW)`, o que
realmente acontece no pino?

- **HIGH** ≈ o pino vai para **cerca de 3,3 V** (energia máxima que ele
  fornece);
- **LOW** ≈ o pino vai para **cerca de 0 V** (desligado, próximo do GND).

```
HIGH  ≈  3,3 V   (aceso, ligado)
LOW   ≈  0 V     (apagado, desligado)
```

Não é exatamente 3,3 e 0 perfeitos (sempre há pequenas variações), mas é
assim que devemos pensar. É por isso que ligar um LED direto no GPIO
(com um resistor, vinculado em aula futura) funciona: o GPIO dá energia em
HIGH e tira em LOW.

**Importante:** como HIGH é 3,3 V e NÃO 5 V, você entende por que não se
pode esperar que um GPIO acione coisas que exigem 5 V.

### 6. Identificando os pinos da placa (serigrafia e pinout)

Olhando para a sua placa, você vai ver **letras e números impressos** perto
dos pinos. Isso se chama **serigrafia** (a marcação impressa no plástico).

Muitas placas têm uma **tabela de pinos** (o **pinout**) — um diagrama que
mostra o nome e o número de cada pino. Esse diagrama é indispensável porque:

- o **número GPIO não é a posição física** na placa. O "pino físico nº 1"
  pode ser o "GPIO 3", e por aí vai. O número GPIO é o **nome lógico** que o
  programa usa, não a ordem em que estão na fileira.
- cada modelo de placa pode organizar os pinos de um jeito diferente.

**Sempre consulte o pinout da sua placa.** Procure por "pinout" + o nome
exato do seu modelo na internet, e guarde esse diagrama. É o seu mapa.

### 7. Nem todo GPIO é igual

Só para introduzir a ideia (sem decorar listas, sem pânico):

alguns GPIOs têm **funções especiais** ou **restrições** — por exemplo, alguns
são usados no boot, outros em comunicações específicas, outros podem se
comportar de jeitos estranhos dependendo da placa.

A **única coisa** que você precisa levar daqui é: **nem todo pino que
parece GPIO se comporta igual.** Por isso, a boa prática é usar na prática
os pinos que o **pinout** da sua placa indica como GPIO de uso geral seguro,
e confirmar antes de usar um pino "especial".

Não se preocupe em decorar. Conforme avançarmos e encontrarmos um pino
especial, **avisaremos na hora**. Por enquanto, a atitude segura basta.

### 8. GND comum

**GND** ("terra") é a **referência** do circuito — o fio de "zero" em relação
ao qual todas as tensões são medidas. HIGH 3,3 V é "3,3 V *acima do GND*".

O conceito de **GND comum** é simples:

> Quando dois circuitos (ou duas placas) precisam conversar, eles devem
> compartilhar o **mesmo GND** — como se fosse o "chão" onde os dois
> pisam. Se um está no "chão A" e o outro no "chão B", o sinal pode não
> fazer sentido.

```
Placa 1             Placa 2
 GPIO ———sinal———> GPIO
 GND  ——————————> GND   (GND comum: os dois "pisam" no mesmo lugar)
```

Na prática, quando ligarmos um botão ou sensor ao ESP32 (próximas aulas),
vamos sempre conectar o GND do componente ao **GND da placa**. Sem esse
retorno, o circuito simplesmente **não funciona**.

### 9. Experiência prática

Agora vamos colocar a mão na massa, usando o **Blink da Aula 01**.

**Como descobrir o GPIO do seu LED embutido:**

Muitas placas têm o LED embutido no **GPIO 2**, mas **não é regra**. A
melhor forma de confirmar é consultar o **pinout** da sua placa e procurar
por "LED" ou "built-in LED". Em alguns modelos (como certos ESP32-S3), pode
estar em outro pino.

**Passos:**

1. Abra o programa **Blink** da Aula 01.
2. Confirme (pelo pinout) qual é o **GPIO do LED embutido** da sua placa.
3. Se o seu não for o 2, troque o número no código.
4. Compile e faça o upload.

Repare: você entendeu *por que* usar aquele número — não é mero acaso, é uma
decisão baseada no **pinout** da placa.

### 10. Multímetro (opcional e muito simples)

Se você tiver um multímetro, podemos **ver no mundo real** o HIGH e o LOW.
Esta parte é opcional.

Um multímetro é um aparelho que **mede tensão** (entre outras coisas). Para
medir os 3,3 V e 0 V de um GPIO:

1. Coloque as pontas do multímetro na escala de **tensão contínua (V⎓)**,
   num valor que cubra uns 5 V (por exemplo, 20 V).
2. A ponta **preta** (comum) vai no **GND** da placa.
3. A ponta **vermelha** (V) vai no **GPIO** do LED (no mesmo pino onde está
   ligado o LED, ou num ponto que você consiga tocar com segurança).
4. Com o Blink rodando, leia o número que aparece:

- quando o LED está **acesso**, o multímetro mostra **cerca de 3,3 V**
  (o pino está em **HIGH**);
- quando o LED está **apagado**, mostra **cerca de 0 V** (o pino está em
  **LOW**).

Isso confirma, na prática, o item 5: HIGH ≈ 3,3 V e LOW ≈ 0 V. Não precisa
ser exatíssimo — basta ver o valor mudar entre os dois extremos.

> Se não tiver multímetro, tudo bem: pule esta parte. Ela é só para quem
> quiser "ver" o conceito com um aparelho.

## O que deve acontecer

Depois da experiência prática, você deve:

- saber dizer, na sua placa, **qual GPIO** é o LED embutido;
- entender **por que** usamos o número certo (por causa do **pinout**);
- (opcional) ter **medido** HIGH e LOW no multímetro e visto os ~3,3 V e ~0 V.

E, mentalmente, deve conseguir explicar: GPIO é sinal, 3V3/5V/GND são
alimentação, e o ESP32 fala em 3,3 V — **sem jogar 5 V em GPIO.**

## Se não funcionar

- **O LED não pisca e o código está igual ao da Aula 01**: confirme que o
  número do GPIO está certo para a **sua** placa (consultando o pinout).
- **Não sei qual é o GPIO do meu LED**: procure "pinout" + modelo da placa.
  Se não achar o LED embutido, sua variante pode não ter um — nesse caso,
  use um LED externo (com resistor e GND comum, que veremos em aula futura).
- **Multímetro não mudou de valor**: confira se as pontas estão na escala
  certa (tensão contínua) e se a preta está no GND e a vermelha no GPIO.
  Alguns multímetros precisam de ajuste de escala.
- **Confuso sobre qual pino é qual**: desligue o cabo, olhe com calma a
  serigrafia e o pinout, compare com a placa real. Vá devagar — não há pressa.

## Experimente você

Agora, um desafio pequeno para fixar:

1. No **pinout** da sua placa, procure e **liste**:
   - quantos pinos estão marcados como **3V3**;
   - quantos como **5V**;
   - quantos como **GND**;
   - quantos como **GPIO**.
2. Diga, com suas palavras (em voz alta ou escrevendo): "por que não posso
   ligar 5 V num GPIO do ESP32?"
3. Se tiver multímetro, escolha outro GPIO que o pinout marque como livre e
   repita a medição do passo 10 — tente prever o valor antes de ver.

## O que aprendemos

Nesta aula:

- **GPIO** é o "canal" entre o programa e o mundo físico (entrada ou saída);
- **3V3/5V** são alimentação; **GND** é o retorno; **GPIO** é sinal — coisas
  diferentes;
- **INPUT** o ESP32 recebe; **OUTPUT** o ESP32 envia;
- o ESP32 fala em **3,3 V** — **HIGH ≈ 3,3 V** e **LOW ≈ 0 V** — e **não
  tolera 5 V** nos GPIOs;
- o **número GPIO não é a posição física**; precisamos do **pinout**;
- alguns GPIOs têm **funções especiais** (avisaremos quando aparecerem);
- todo circuito precisa de **GND comum** para conversar.

## Próximo passo

Agora que sabemos que um GPIO pode ser **entrada**, vamos fazer o ESP32
**escutar o mundo**: vamos ler um **botão** e entender como a placa percebe
quando algo foi pressionado.

Nos vemos na Aula 03.
