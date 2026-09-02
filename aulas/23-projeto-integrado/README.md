# Aula 23 — Projeto integrado

## O que vamos fazer

Chegamos à última aula! Vamos **juntar tudo o que aprendemos** em um único
projeto real: um **painel conectado** que busca a **hora/uma informação da
internet**, mostra em um **display**, reage a um **botão** e **salva uma
configuração** — tudo isso usando o que você já viu em várias aulas.

O segredo deste projeto é **montá‑lo por etapas**: cada parte é pequena e já
funciona sozinha; depois vamos **somá‑las** uma a uma. Um projeto grande é
isso: a soma de várias partes pequenas que você já domina.

Por que isso é útil? É o "projeto de formatura" de alguém que quer fazer
coisas de verdade com o ESP32 — e mostra como **organizar** um código maior
em **funções simples** (algo que facilita muito a vida).

## O que você vai aprender

No final desta aula, você vai entender:

- como **planejar (desenhar)** um projeto antes de programar;
- o que é **NTP** e como obter a **hora da internet**;
- como **organizar o código em funções simples**;
- como **integrar gradualmente** display, botão, Wi‑Fi e internet;
- como usar **Preferences** para salvar uma configuração;
- como **diagnosticar** e lidar com **falhas** (sem Wi‑Fi, servidor fora);
- como **revisar segurança básica**;
- e perceber que **um projeto grande é só a soma de partes pequenas**.

## Definindo o projeto

**O que o painel vai fazer?**

1. Conectar ao **Wi‑Fi** (Aula 12).
2. Buscar a **hora certa da internet** (via **NTP**).
3. Mostrar a **hora** no **display ST7789** (Aula 11).
4. Um **botão** alterna entre mostrar a hora e mostrar outra informação
   (ex.: temperatura/uma API simples da Aula 14).
5. **Salvar** a última escolha (hora ou outra info) com **Preferences**
   (Aula 19), para que, ao religar, o painel volte ao modo que você usou.
6. Avisos no **Monitor Serial** para diagnóstico (Aula 07).

Isso usa quase tudo: SPI, botão, Wi‑Fi, HTTP, NTP, Preferences, display.

## Desenhando em blocos (antes do código)

Vamos desenhar os **blocos** do sistema num diagrama simples. Isso ajuda a ver
as partes antes de programar:

```
[ Botão ] --> [ ESP32 ] --> [ Display ST7789 ]
                |   ^
                |   +--- Preferences (salva o modo)
                |
                v
            [ Wi-Fi ] --> [ NTP: hora ] e [ API: outra info ]
```

Pense em cada bloco como uma **função** no código:

- `conectarWiFi()` — liga o Wi‑Fi;
- `buscarHora()` — pega a hora (NTP);
- `buscarInfo()` — pega uma informação (API);
- `mostrar(txt)` — escreve no display;
- `salvarModo()` / `carregarModo()` — Preferences;
- `diagnostico(...)` — mensagens no Serial.

Não precisamos codificar tudo de uma vez. **Uma parte de cada vez, testando
cada uma.**

> ⚠️ Use **placeholders** para SSID/senha (ex.: `SEU_SSID`) — nunca
> informações reais no código do repositório (lembre das Aulas 12 e 19).

## Material necessário

- **ESP32**;
- **display ST7789** (Aula 11);
- um **botão** (Aula 03);
- **Wi‑Fi** (Aula 12);
- Monitor Serial (115200);
- (opcional, se for usar a API) um endpoint público da Aula 14.

## Ligação

Use o mesmo esquema de antes: display via **SPI** (Aula 11), botão com
`INPUT_PULLUP` (Aula 03). Confira os pinos no seu pinout.

```
ESP32                  Display ST7789
SCLK (18) ──────────── SCL
MOSI (23) ──────────── SDA
GPIO CS (5) ────────── CS
GPIO DC (2) ────────── DC
3V3 / GND ──────────── VCC / GND
(backlight, se houver) → 3V3

ESP32                  Botão
GPIO 4 ─────────────── terminal 1
GND  ───────────────── terminal 2 (INPUT_PULLUP)
```

> ⚠️ **Atenção à 3,3 V**: mantenha tudo em **3,3 V** (Aula 01). O display e o
> botão trabalham com 3,3 V — não invente 5 V sem aviso. Se o **backlight**
> do display existir, ligue-o a **3V3** (ou a um GPIO para o brilho).

## Etapa 1 — Testar o botão e o display juntos (sem internet)

Antes de envolver a internet, vamos garantir que **botão** e **display**
funcionam. Este pequeno programa acende o fundo do display em duas cores
conforme o botão:

```cpp
// Aula 23 — Etapa 1: botão e display

#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>
#include <SPI.h>

#define PIN_CS   5
#define PIN_DC   2
#define PIN_RST  -1
#define BOTAO    4

#define LARGURA  240
#define ALTURA   240

Adafruit_ST7789 tela(PIN_CS, PIN_DC, PIN_RST);

void setup() {
  Serial.begin(115200);
  pinMode(BOTAO, INPUT_PULLUP);
  tela.init(LARGURA, ALTURA);
  Serial.println("Etapa 1 iniciada.");
}

void loop() {
  if (digitalRead(BOTAO) == LOW) {
    tela.fillScreen(ST77XX_GREEN);
    tela.setCursor(10, 10);
    tela.setTextColor(ST77XX_WHITE);
    tela.print("Botao OK (modo A)");
  } else {
    tela.fillScreen(ST77XX_BLUE);
    tela.setCursor(10, 10);
    tela.setTextColor(ST77XX_WHITE);
    tela.print("Botao solto (modo B)");
  }
}
```

**Teste:** pressione o botão — a cor do display muda. Se funcionar, as duas
peças estão prontas. (Relembre: `digitalRead == LOW` = pressionado, Aula 03.)

## Etapa 2 — Conectar ao Wi‑Fi (Aula 12)

Vamos garantir a conexão **antes** de juntar com o display:

```cpp
// Aula 23 — Etapa 2: só conectar ao Wi-Fi

#include <WiFi.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

void conectarWiFi() {
  Serial.println("Conectando ao Wi-Fi...");
  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("Conectado! IP: " + WiFi.localIP().toString());
}

void setup() {
  Serial.begin(115200);
  conectarWiFi();
}

void loop() {
}
```

**Teste:** o Serial mostra o **IP** quando conectar. Guardamos essa função —
vamos reaproveitá‑la.

## Etapa 3 — Buscar a hora e informação (NTP e API)

Agora o bloco de **internet**. Três ideias rápidas:

- **NTP** é um "serviço da hora": o ESP32 pergunta "que horas são?" e recebe
  a **hora certa** pela internet. Configuramos com `configTime(...)`.
- Uma **API** (Aula 14) pode trazer uma **outra informação** (ex.: uma
  temperatura). Deixamos como opcional para você ajustar.

Vamos configurar o NTP e, no Serial, mostrar a hora:

```cpp
// Aula 23 — Etapa 3: obter a hora por NTP

#include <WiFi.h>
#include <time.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

void conectarWiFi() {
  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Conectado!");
}

void configurarNTP() {
  configTime(-3 * 3600, 0, "pool.ntp.org");  // fuso -3h (Brasil), ajuste o seu
  Serial.println("Ajustando o relogio...");
}

String horaFormatada() {
  struct tm info;
  if (!getLocalTime(&info)) {
    return "--:--:--";
  }
  char buffer[9];
  strftime(buffer, sizeof(buffer), "%H:%M:%S", &info);
  return String(buffer);
}

void setup() {
  Serial.begin(115200);
  conectarWiFi();
  configurarNTP();
}

void loop() {
  Serial.println(horaFormatada());
  delay(1000);
}
```

### Entendendo as partes novas

```cpp
#include <time.h>
```

Inclui suporte a **hora/data**.

```cpp
configTime(-3 * 3600, 0, "pool.ntp.org");
```

**`configTime(fusoEmSeg, horarioDeVerao, servidor)`** ajusta o relógio:
- `-3 * 3600` — o fuso em **segundos** (‑3 horas = horário do Brasil;
  **ajuste para o seu fuso**); `0` = sem horário de verão por padrão;
- `"pool.ntp.org"` — um servidor de hora público.

```cpp
struct tm info;
if (!getLocalTime(&info)) {
```

**`getLocalTime(&info)`** busca a **hora atual** e preenche `info`. Se ainda
não sincronizou, devolve falso (volta `"--:--:--"`). (O `struct tm` é só um
"pacote" que guarda hora/data.)

```cpp
strftime(buffer, sizeof(buffer), "%H:%M:%S", &info);
```

**`strftime(...)`** transforma a hora em texto (`%H` hora, `%M` minuto, `%S`
segundo) num `buffer`. Esse é um jeito comum de formatar hora em C/C++.

**Teste:** o Serial mostra a hora correta a cada segundo. Se mostrar
`--:--:--`, aguarde alguns segundos para o NTP sincronizar.

> Para a **outra informação** (temperatura etc.), você pode **reaproveitar**
> o HTTP/JSON da Aula 14 numa função `buscarInfo()`. Fica opcional — o foco
> desta aula é a **integração**, não uma API específica.

## Etapa 4 — Juntar tudo em um painel (integração)

Agora o momento de **somar as partes**: Wi‑Fi + NTP + display + botão +
Preferences. Cada bloco virou uma **função simples** que usamos no `loop()`.

O painel mostra **hora** ou **outra info**, e o botão **alterna**. A escolha
fica **salva** (Preferences) para religar no mesmo modo.

```cpp
// Aula 23 — Projeto integrado: painel conectado

#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>
#include <SPI.h>
#include <WiFi.h>
#include <Preferences.h>
#include <time.h>

#define PIN_CS   5
#define PIN_DC   2
#define PIN_RST  -1
#define BOTAO    4
#define LARGURA  240
#define ALTURA   240

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

Adafruit_ST7789 tela(PIN_CS, PIN_DC, PIN_RST);
Preferences prefs;

const int MODO_HORA = 0;
const int MODO_INFO = 1;
int modoAtual = MODO_HORA;
unsigned long ultimoToque = 0;

// ---------- Bloco: Wi-Fi ----------
void conectarWiFi() {
  Serial.println("Conectando ao Wi-Fi...");
  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("Conectado!");
}

// ---------- Bloco: NTP (hora) ----------
void configurarNTP() {
  configTime(-3 * 3600, 0, "pool.ntp.org");
  Serial.println("Ajustando relogio (NTP)...");
}

String horaFormatada() {
  struct tm info;
  if (!getLocalTime(&info)) {
    return "--:--:--";
  }
  char buffer[9];
  strftime(buffer, sizeof(buffer), "%H:%M:%S", &info);
  return String(buffer);
}

// ---------- Bloco: Preferences (guarda a escolha) ----------
void salvarModo() {
  prefs.begin("painel", false);
  prefs.putInt("modo", modoAtual);
  prefs.end();
}

void carregarModo() {
  prefs.begin("painel", true);
  modoAtual = prefs.getInt("modo", MODO_HORA);
  prefs.end();
}

// ---------- Bloco: Display ----------
void mostrarTela(String titulo, String valor) {
  tela.fillScreen(ST77XX_BLACK);
  tela.setCursor(20, 30);
  tela.setTextColor(ST77XX_CYAN);
  tela.print(titulo);
  tela.setCursor(20, 90);
  tela.setTextColor(ST77XX_WHITE);
  tela.setTextSize(3);
  tela.print(valor);
  tela.setTextSize(1);
}

// ---------- Bloco: Diagnóstico (Serial) ----------
void diagnostico(String msg) {
  Serial.println("[" + String(millis()) + " ms] " + msg);
}

void setup() {
  Serial.begin(115200);
  pinMode(BOTAO, INPUT_PULLUP);
  tela.init(LARGURA, ALTURA);

  carregarModo();              // 1. escolha salva
  diagnostico("Iniciando painel, modo '" + String(modoAtual) + "'");

  conectarWiFi();              // 2. rede
  configurarNTP();             // 3. hora
}

void loop() {
  // Botao muda o modo (com debounce simples usando millis)
  if (digitalRead(BOTAO) == LOW && millis() - ultimoToque > 300) {
    ultimoToque = millis();
    modoAtual = (modoAtual == MODO_HORA) ? MODO_INFO : MODO_HORA;
    salvarModo();
    diagnostico("Modo alterado para '" + String(modoAtual) + "'");
  }

  // Mostra conforme o modo
  if (modoAtual == MODO_HORA) {
    mostrarTela("HORA (NTP)", horaFormatada());
  } else {
    // Exemplo de "outra info": (coloque aqui o resultado da sua API/sensor)
    mostrarTela("OUTRA INFO", "exemplo");
  }

  delay(500);
}
```

Leia com calma — **não é um monstrinho**: são as funções das aulas anteriores
uma do lado da outra.

### Entendendo a parte de organização

- **Cada bloco é uma função** (Wi‑Fi, NTP, Preferences, Display, Diagnóstico).
  Isso **organiza** o código: dá para ler e alterar cada parte isolada.
- **`#define` com nomes** (Aula 05) no topo: pinos e constantes fáceis de
  encontrar e trocar.
- **`setup()`** faz as tarefas de **uma vez**: carrega o modo, conecta,
  ajusta a hora.
- **`loop()`** cuida do que se repete: ler o **botão** (alternar) e
  **atualizar o display**.

### Entendendo as partes novas

```cpp
unsigned long ultimoToque = 0;
...
if (digitalRead(BOTAO) == LOW && millis() - ultimoToque > 300) {
  ultimoToque = millis();
  modoAtual = (modoAtual == MODO_HORA) ? MODO_INFO : MODO_HORA;
```

Um **debounce por tempo** (Aula 04/16): só aceita o toque se passaram **300 ms**
desde o último (`millis()`). `(condicao) ? a : b` é um atalho: se a condição
for verdadeira, usa `a`; senão, `b` — aqui alterna entre os dois modos.

```cpp
void salvarModo() { ... prefs.putInt("modo", modoAtual); ... }
void carregarModo() { ... modoAtual = prefs.getInt("modo", MODO_HORA); ... }
```

**Salva/recarrega** o modo com **Preferences** (Aula 19). Ao religar, o
painel volta ao modo que foi usado por último.

```cpp
void diagnostico(String msg) {
  Serial.println("[" + String(millis()) + " ms] " + msg);
}
```

Uma **função de diagnóstico** (Aula 07): registra no Serial com o **tempo** de
quando ocorreu — muito útil para depurar.

## Etapa 5 — Testar falhas (sem Wi‑Fi, servidor fora, religar)

Projetos de verdade precisam lidar com falhas. Teste estes cenários:

- **Sem Wi‑Fi**: desligue a rede. Veja o Serial travar no `while` da
  `conectarWiFi()`. Perceba o efeito — em versões melhores, você colocaria um
  **limite de tentativas** (contador com `millis()`) para não ficar preso
  para sempre.
- **Servidor/hora fora**: o NTP responde com `--:--:--` até sincronizar — o
  painel continua funcionando, mostrando o placeholder. Bom exemplo de
  **falha parcial** sem travar o resto.
- **Religar**: desligue e religue. O painel volta ao **modo salvo**
  (Preferences). Esse é o teste mais gratificante — a config se manteve.

**Dica para o `while` que trava:** experimente trocar o `while` da conexão
por uma **espera limitada** (loop de X tentativas com `delay`) e, se não
conectar, mostre um aviso no display e siga em frente. Assim o painel não
"congela" para sempre sem rede.

## Revisão de segurança básica

- **Sem senhas/chaves no código**: use placeholders; valores reais entram via
  runtime (Preferences/Aula 19) ou config segura — **nunca** no repositório.
- **Não exponha o ESP32 em internet aberta sem cuidado** (Aula 13): esta
  apresentação é didática; numa aplicação real, mantenha redes e senhas
  protegidas.
- **Sem dados sensíveis**: não envie informações pessoais desnecessárias.

## Organização final do código

Mesmo que este código seja pequeno, veja o padrão: **cada função faz uma
coisa** e tem nome claro. Se um dia o projeto crescer, você **separa as
funções em arquivos** (ex.: `wifi.h`, `tela.h`). Por ora, funções bem
nomeadas já garantem um código legível. Este é o **primeiro degrau** de uma
boa organização — nada de arquitetura complicada nesta etapa.

## Se não funcionar

- **Display escuro/errado**: revise CS/DC/SCK/MOSI e o backlight (Aula 11).
- **Botão não alterna**: confira o pino, o `INPUT_PULLUP` e o debounce
  (`300 ms`). Veja no Serial se o modo mudou (`diagnostico`).
- **Hora não sincroniza (`--:--:--`)**: aguarde mais (NTP leva alguns
  segundos) e confira **fuso** e a conexão. Teste com o Serial aberto.
- **Não salva o modo**: confira `prefs.begin("painel", ...)` e se o `end()`
  é chamado após gravar.
- **Trava por causa do Wi‑Fi**: o `while` sem limite "prende" o painel sem
  rede. Use a **espera limitada** da Etapa 5.
- **Compila erro em alguma API**: as bibliotecas (Adafruit, WiFi, Preferences,
  time) são padrão do ESP32; verifique se instalou o Adafruit GFX/ST7789
  (Aula 11). APIs de bibliotecas podem variar com a versão — adapte.

## Experimente você

Agora, os desafios:

1. **Limite a espera do Wi‑Fi**: troque o `while` por uma espera limitada
   (contador + `millis()`); se falhar, mostre `"Sem rede"` no display.
2. **Outra informação de verdade**: no modo "OUTRA INFO", chame a sua
   `buscarInfo()` da Aula 14 (HTTP/JSON) e mostre o valor (ex.: temperatura).
3. **Recenture o painel**: adicione um `tela.fillScreen` e textos maiores para
   deixar a hora bem visível.

> Relembre a regra de ouro deste curso: **partes pequenas, cada uma
> funcionando, somadas aos poucos.** É assim que se constrói qualquer
> projeto grande.

## O que aprendemos

Nesta aula, você **juntou** praticamente tudo:

- **planejou** o projeto com um diagrama de blocos antes do código;
- usou **display** (Aula 11), **botão** (Aula 03), **Wi‑Fi** (Aula 12),
  **HTTP/API** (Aula 14), **NTP** (novidade), **Preferences** (Aula 19) e
  **Serial** (Aula 07) juntos;
- **NTP** usa `configTime` e `getLocalTime` para a hora da internet;
- **organizou** em **funções simples e nomeadas**;
- **integrou por etapas**, testando cada parte antes de somar;
- trata **falhas** (sem Wi‑Fi, hora pendente) sem travar tudo;
- revisou **segurança básica** (sem segredos no código);
- e percebeu que **um projeto grande é só a soma de partes pequenas**.

## Próximo passo

Este é o fim da jornada! Você saiu do zero e chegou a um **projeto integrado
funcionando** — passando por entradas, saídas, tempo, comunicação e
organização. Agora é **praticar e criar os seus próprios projetos**, aplicando
as mesmas etapas aqui exercitadas. Parabéns — você já é uma pessoa que
constrói com o ESP32.

Boa bancada e bom voo!
