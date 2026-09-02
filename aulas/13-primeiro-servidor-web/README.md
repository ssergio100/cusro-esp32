# Aula 13 — Primeiro servidor Web

## O que vamos fazer

Na Aula 12 o ESP32 entrou na rede e ganhou um IP. Agora vamos dar o passo
seguinte: fazer o ESP32 **servir uma página web** — ou seja, se comportar
como um "mini-servidor" que você acessa pelo **navegador** do celular ou
computador. Na página, um **botão** vai **ligar e desligar um LED** na placa.

Por que isso é útil? É o começo de algo muito poderoso: **controlar um
dispositivo pela internet/rede**, sem precisar tocar nele. Um sonho de
"casa inteligente" começa aqui.

## O que você vai aprender

No final desta aula, você vai entender:

- a diferença entre **cliente** e **servidor**;
- o que é **HTTP** (em uma frase simples);
- como **iniciar um servidor** no ESP32;
- como criar uma **rota `/`**;
- como servir **HTML simples**;
- como criar um **botão no navegador**;
- como criar uma **rota para ligar/desligar o LED**;
- o **fluxo** navegador → ESP32 → GPIO;
- como **exibir o IP** no Monitor Serial;
- como **diagnosticar** o acesso pela rede local.

## Cliente e servidor

Imagine um **restaurante**:

- O **cliente** é quem chega fazendo um **pedido** (o navegador pede uma
  página).
- O **servidor** é a cozinha que **prepara** o que foi pedido e **entrega**
  (o ESP32 responde com a página).

```
cliente (navegador)  ──pedido──>  servidor (ESP32)
        <────resposta────
```

Até aqui o ESP32 foi **cliente** (pediu conexão à rede). Agora ele vira
**servidor**: recebe pedidos e responde. O navegador é o cliente.

## O que é HTTP (em uma frase)

**HTTP** é a "linguagem" que cliente e servidor usam para conversar na web.
Quando você digita um endereço no navegador e ele responde com uma página,
é uma **conversa HTTP**. Não precisa decorar detalhes — só saber que o ESP32
"entende" essa linguagem por uma biblioteca.

## Ligação

Vamos usar o **LED embutido** (Aula 01) — nenhuma ligação externa é
necessária. Só a placa, o cabo USB e a rede Wi‑Fi da Aula 12.

Você também vai precisar do **IP** que o ESP32 recebeu (Aula 12) para
acessá-lo no navegador.

## Antes de ligar

- Reconecte à **mesma rede Wi‑Fi** do seu celular/computador. Para acessar a
  página, o navegador precisa estar na **mesma rede** que o ESP32.
- Isso é um **servidor na rede local** — **não** exponha o ESP32 à internet
  pública (por segurança, mantenha o acesso só pela sua rede Wi‑Fi).

## Primeiro teste: página simples

Vamos criar um servidor que responde com uma **página HTML** na rota `/`.

```cpp
// Aula 13 — Primeiro servidor Web

#include <WiFi.h>
#include <WebServer.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

const int LED = 2;

WebServer servidor(80);

void setup() {
  Serial.begin(115200);
  pinMode(LED, OUTPUT);

  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.print("Acesse: http://");
  Serial.println(WiFi.localIP());

  servidor.on("/", paginaInicial);
  servidor.begin();
}

void loop() {
  servidor.handleClient();
}

void paginaInicial() {
  servidor.send(200, "text/html",
    "<h1>Ola do ESP32!</h1>"
    "<p>Seu servidor esta funcionando.</p>");
}
```

### Entendendo o código

```cpp
#include <WebServer.h>
```

Inclui a biblioteca **WebServer** (já vem com o ESP32), que cuida de receber
as requisições HTTP.

```cpp
WebServer servidor(80);
```

Cria o "controle" do servidor na **porta 80** (a porta padrão da web — quando
você digita só um IP no navegador, ele usa a 80 por padrão).

```cpp
servidor.on("/", paginaInicial);
```

**`servidor.on(caminho, funcao)`** diz: "**quando** alguém acessar a rota
`/` (`/` é a página principal), **chame** a função `paginaInicial`".

```cpp
servidor.begin();
```

**Inicia** o servidor — a partir daqui ele aceita conexões.

```cpp
void loop() {
  servidor.handleClient();
}
```

No `loop()`, a linha **`servidor.handleClient()`** **escuta** se chegou
algum pedido e o trata (chamando a função certa). Sem essa linha, o servidor
não faz nada.

```cpp
void paginaInicial() {
  servidor.send(200, "text/html", "...");
}
```

A função que responde à rota `/`. **`servidor.send(codigo, tipo, conteudo)`**
**envia a resposta**:

- **`200`** — o código que significa "deu tudo certo" (a "resposta OK");
- **`"text/html"`** — o tipo de conteúdo (é uma página HTML);
- entre aspas, o **conteúdo** da página (o HTML).

## O que deve acontecer

Envie o programa, anote o **IP** mostrado no Serial, e abra no navegador:
`http://SEU_IP` (ex.: `http://192.168.1.42`). Deve aparecer:

> **Ola do ESP32!**
> Seu servidor esta funcionando.

Se vê essa página, **você criou um servidor web no ESP32!**

## Botão na página e rota para o LED

Agora o passo mais interessante: adicionar **botões** na página que **ligam e
desligam o LED**. Cada botão "chama" uma rota do servidor, que mexe no GPIO.

```cpp
// Aula 13 — Controlar o LED pelo navegador

#include <WiFi.h>
#include <WebServer.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

const int LED = 2;

WebServer servidor(80);

void setup() {
  Serial.begin(115200);
  pinMode(LED, OUTPUT);

  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.print("Acesse: http://");
  Serial.println(WiFi.localIP());

  servidor.on("/", paginaInicial);
  servidor.on("/liga", ligarLed);
  servidor.on("/desliga", desligarLed);
  servidor.begin();
}

void loop() {
  servidor.handleClient();
}

void paginaInicial() {
  servidor.send(200, "text/html",
    "<h1>Controlar LED</h1>"
    "<a href=\"/liga\"><button>Ligar LED</button></a>"
    "<a href=\"/desliga\"><button>Desligar LED</button></a>");
}

void ligarLed() {
  digitalWrite(LED, HIGH);
  servidor.send(200, "text/html", "<p>LED ligado</p><a href=\"/\">Voltar</a>");
}

void desligarLed() {
  digitalWrite(LED, LOW);
  servidor.send(200, "text/html", "<p>LED desligado</p><a href=\"/\">Voltar</a>");
}
```

### Entendendo o que há de novo

- **Duas rotas novas**: `servidor.on("/liga", ...)` e
  `servidor.on("/desliga", ...)`.
- **`<a href="/liga">`** — um **link** (a tag `a`). Quando você **clica**, o
  navegador pede a rota `/liga` ao servidor.
- **`<button>Ligar LED</button>`** — o **botão** visual. O link em volta
  faz o clique do botão ir para a rota.
- Em `ligarLed()` / `desligarLed()`: **`digitalWrite(LED, HIGH/LOW)`** liga
  ou desliga o LED (Aula 01), e depois respondemos com uma página de
  confirmação.

## O fluxo completo

Veja como tudo se encaixa:

```
1. Navegador pede "http://IP/"           → servidor envia a página com botões
2. Você clica "Ligar LED"                 → navegador pede "http://IP/liga"
3. Servidor chama ligarLed()              → digitalWrite(LED, HIGH)
4. LED acende na placa                    → servidor responde "LED ligado"
```

```
navegador → HTTP → ESP32 (WebServer) → digitalWrite → GPIO/LED
```

É exatamente esse o "fluxo navegador → ESP32 → GPIO" que você vai usar em
muitos projetos.

## Se não funcionar

- **Não abre a página**: confirme se está na **mesma rede** que o ESP32.
  Confira o **IP** (o endereço pode mudar se reconectar). Tente acessar
  digitando `http://` antes do IP.
- **Página abre, mas o botão não mexe no LED**: confira o pino do LED da sua
  placa (Aula 02) e se cliquei nos botões `/liga` e `/desliga`.
- **O servidor "trava"**: garanta que `servidor.handleClient()` está no
  `loop()`. Não coloque `delay()` longo no `loop()` que impeça de atender
  os pedidos.
- **Navegador mostra erro**: verifique se o IP foi digitado corretamente.

## Experimente você

Agora, os desafios:

1. **Mude o texto da página**: personalize o título e o texto das páginas.
2. **Mostre o estado**: crie uma função que diga se o LED está aceso ou
   apagado na página. (Dica: use `digitalRead(LED)` para saber o estado e
   monte a string de resposta de acordo.)
3. **Adicione um botão de "piscar"**: crie uma rota `/pisca` que faça o LED
   piscar algumas vezes (use `millis` da Aula 04 ou um pequeno `delay`).
4. **Compartilhe**: abra a página pelo **celular** (na mesma rede) e teste
   os botões por lá.

## O que aprendemos

Nesta aula:

- o **navegador** é o **cliente** e o **ESP32** virou o **servidor**;
- **HTTP** é a "linguagem" dessa conversa;
- usamos a biblioteca **WebServer** na **porta 80**;
- **`servidor.on(rota, funcao)`** liga um caminho a uma função;
- **`servidor.send(codigo, tipo, conteudo)`** envia a resposta (200 = OK);
- botões na página viram **links** para rotas que **controlam o GPIO**;
- o **fluxo é**: navegador → ESP32 → GPIO;
- servidor local **não** deve ser exposto à internet.

## Próximo passo

Agora o ESP32 **responde** a pedidos. O próximo passo é inverter o jogo: o
ESP32 **fazendo pedidos** para servidores externos — buscando informações de
**APIs** na internet.

Nos vemos na [Aula 14](../14-esp32-conversando-com-servidores/README.md).
