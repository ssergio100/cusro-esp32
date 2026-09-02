# Aula 18 — Memória do ESP32

## O que vamos fazer

Até aqui escrevemos programas sem pensar muito no que acontece "por dentro".
Mas, quando os projetos crescem, o ESP32 pode **travar** ou ficar **sem
memória**. Nesta aula vamos abrir a "caixa" e entender **os tipos de memória**
do ESP32: qual guarda o programa, qual guarda os dados temporários e quais
guardam dados mesmo com o aparelho desligado.

Vamos ver na prática como **medir a memória livre** com funções prontas, e
entender o que fazer quando começar a faltar espaço.

Por que isso é útil? Saber disso evita os erros mais temidos dos iniciantes:
o "travar sem explicação", o "reiniciar sozinho" e o "não está mais
funcionando". Entender a memória é entender os limites e as possibilidades da
sua placa.

## O que você vai aprender

No final desta aula, você vai entender:

- a diferença entre **Flash** e **RAM** (com analogia);
- o que são e para que servem **Flash, RAM e PSRAM**;
- como **medir a RAM livre**;
- estratégias quando a **memória acaba**;
- um alerta sobre **ponteiros** e **strings**;
- uma regra prática de **prevenção**;
- e por que **PSRAM** é um recurso específico do ESP32‑S3 e alguns outros.

## Flash e RAM: a "caderneta" e a "mesa de trabalho"

A melhor analogia é uma **cozinha/escritório**:

- **Flash**: é a **prateleira de livros** — guarda o **programa** e dados
  permanentes. Por ser permanente, ela **continua guardada mesmo sem energia**.
  É limitado, mas não some quando desliga.
- **RAM**: é a **mesa de trabalho** — espaço **temporário** onde o programa
  roda e guarda variáveis enquanto está ligado. É **rápida**, porém **some
  tudo** quando desliga a energia.

```
            Flash (prateleira)          RAM (mesa)
 guarda      programa + dados perm.      variáveis/execução
 permanente? sim (some energia?)         não (perde ao desligar)
 rápido?     mais lento                  mais rápido
```

## Os três "compartimentos" do ESP32

No ESP32, costumamos citar três:

1. **Flash** — guarda o **programa** (o que você subiu) e a **partição**
   configurada. É onde o código "mora". Para a maioria dos projetos, você não
   mexe nela diretamente, só sabe que ela **existe** e é limitada.
2. **RAM** — a **memória de trabalho** do processador. É onde `loop()`,
   variáveis e a pilha rodam. **É a que costuma acabar** e causar travamentos.
3. **PSRAM** — uma memória **externa extra** (está **em placas específicas**,
   como alguns modelos do **ESP32‑S3** e outros). Pense nela como uma **mesa
   de trabalho maior**: dá mais espaço RAM para dados grandes (ex.: buffers
   de imagem, áudio). **Não existe em todas as placas** — é preciso verificar
   a sua.

> ⚠️ **Verifique sua placa**: PSRAM e os valores exatos de Flash/RAM variam
> conforme o **modelo**. Consulte o *pinout/ficha* da sua placa para saber
> quanto ela tem. Nunca presumir.

## Medindo a RAM livre na prática

Vamos ver quanto de RAM o ESP32 tem e quanto está **livre**. Isso ajuda a
perceber quando o projeto está "chegando no limite". As funções
`ESP.getHeapSize()` e `ESP.getFreeHeap()` mostram isso no Serial:

```cpp
// Aula 18 — Quanto de memória temos?

void setup() {
  Serial.begin(115200);

  Serial.print("Tamanho total da heap (RAM): ");
  Serial.println(ESP.getHeapSize());

  Serial.print("Heap livre agora: ");
  Serial.println(ESP.getFreeHeap());
}

void loop() {
}
```

### Entendendo o código

```cpp
Serial.print("Tamanho total da heap (RAM): ");
Serial.println(ESP.getHeapSize());
```

**`ESP.getHeapSize()`** devolve o **tamanho total** da **heap**. "Heap" é o
nome técnico da **parte da RAM gerenciada** de onde vêm as variáveis e
objetos criados em tempo de execução (as "caixinhas" que você aloca). Não
precisa decorar o termo — saiba que é a RAM de trabalho principal.

```cpp
Serial.println(ESP.getFreeHeap());
```

**`ESP.getFreeHeap()`** mostra quanta dessa RAM está **livre agora**. Quando
esse número fica **muito baixo**, o programa está perto de travar por falta
de memória.

> Se você tiver um **ESP32‑S3 com PSRAM**, também pode ver a PSRAM com
> `ESP.getPsramSize()` (total) e `ESP.getFreePsram()` (livre). Só tente em
> placas que **tenham** PSRAM; em placas sem, esses valores dão zero.

## Quando a memória acaba: o que fazer

Se você começar a ver travamentos ou valores de heap muito baixos, tente:

1. **Reduza o uso no `loop()`**: não aloque variáveis grandes a cada volta.
2. **Cuidado com `String`**: criar e juntar muitas `String` (com `+` ou `+=`)
   a todo momento **fragmenta** e **consome** a RAM. Em projetos apertados,
   prefira arrays `char` fixos.
3. **Libere o que não usa**: alguns criadores de objetos precisam ser
   "fechados"/"deletados" quando não são mais necessários (ex.: `http.end()`).
4. **Cuidado com loops**: um `for`/`while` sem saída pode criar objetos sem
   parar até estourar a memória.
5. **Use a PSRAM, se houver**: para buffers **grandes**, assim você tira o
   peso da RAM principal. (Nas placas que têm PSRAM.)

## Alerta sobre ponteiros e `String`

- **Ponteiros** (`*`) são uma forma avançada de "apontar" para lugares na
  memória. Você **não precisa dominá-los** para este curso, mas saiba que é
  comum que erros de memória (travamentos) estejam ligados a ponteiros mal
  usados **quando se escreve código avançado**. Aqui, vamos evitá-los.
- **`String`** (o tipo de texto) é **cômoda**, mas, usada exageradamente,
  consome RAM. Para valores que mudam muito, um vetor de `char` fixo é mais
  econômico. **Regra prática:** se der para viver com `char`, use `char`;
  use `String` quando precisar de comodidade e a RAM permitir.

## Prevenção prática (regra de bolso)

- **Mantenha o `loop()` leve**: o que puder ser calculado antes (no
  `setup()`) ou com menos frequência, faça assim.
- **Monitore**: rode o exemplo acima de vez em quando e anote quanto de heap
  livre você tem. Se cair **ininterruptamente** enquanto o programa roda,
  há "vazamento de memória" (algo acumulando sem parar).
- **Teste em etapas**: adicionar uma biblioteca nova e o programa passa a
  travar? Desfaça a mudança e verifique a memória. A culpa é frequente de
  memória, não de lógica.

## Se não funcionar

- **Não aparece nada no Serial**: confira a velocidade `115200`.
- **`ESP.getHeapSize()` etc. não compilam**: a maioria das placas ESP32 tem
  essas funções. Se a sua não tiver, procure a função equivalente para a sua
  plataforma. O conceito de medir a RAM é o que importa.
- **Quero ver a PSRAM, mas dá zero**: sua placa pode não ter PSRAM. Confirme
  o **modelo** — PSRAM é recurso específico (ex.: ESP32‑S3 com PSRAM).
- **O programa trava**: veja o heap livre; se estiver baixíssimo, siga os
  passos de "quando a memória acaba".

## Experimente você

Agora, os desafios:

1. **Leia e anote**: rode o exemplo e anote o **heap total** e o **livre** da
   sua placa.
2. **Derrube a memória**: crie um `for` que adiciona letras a uma `String`
   milhares de vezes e veja o `ESP.getFreeHeap()` **caindo** aos poucos.
   Depois remova o `for` e veja voltar. Isso mostra **na prática** como a RAM
   funciona.
3. **Compare**: se você tem um ESP32‑S3 com PSRAM, rode o exemplo e adicione
   `ESP.getPsramSize()` / `ESP.getFreePsram()` para ver a PSRAM extra.

> Cuidado: no desafio 2, não adicione letras **sem limite** a ponto de
> travar. Pare antes e observe o número.

## O que aprendemos

Nesta aula:

- **Flash** guarda o programa (permanente); **RAM** é a mesa de trabalho
  (temporária) e costuma esgotar;
- **PSRAM** é memória extra, presente **só em placas específicas** (como
  alguns ESP32‑S3);
- medimos a RAM com **`ESP.getHeapSize()`** e **`ESP.getFreeHeap()`**;
- **heap baixo** = perto de travar; devemos manter o programa **leve**;
- `String` usada demais gasta memória — prefira `char` quando possível;
- **sempre verificar a ficha da placa** (nada de presumir valores).

## Próximo passo

E se quiséssemos **salvar uma configuração** (ex.: o SSID, um valor) para que
ela continue lá **depois de desligar e religar**? A **Flash** guarda isso, mas
existe um jeito fácil do ESP32 de guardar pequenos dados permanentes: o
**NVS/Preferences**. Vamos aprender na próxima aula.

Nos vemos na Aula 19.
