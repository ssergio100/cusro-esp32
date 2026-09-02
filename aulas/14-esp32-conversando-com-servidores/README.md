# Aula 14 — ESP32 conversando com servidores

## O que vamos fazer

Na Aula 13, o ESP32 foi o **servidor** (respondia a pedidos do navegador).
Nesta aula vamos **inverter os papéis**: o ESP32 vai ser o **cliente**, fazendo
um **pedido HTTP** para um servidor na internet e **lendo a resposta** — como
se fosse um "navegador automático".

Vamos usar uma **API pública simples** que devolve dados em **JSON** (um
formato de texto estruturado) e extrair uma ou duas informações.

Por que isso é útil? É a porta de entrada para a "internet das coisas":
pedir a **previsão do tempo**, cotação, horário (NTP) ou enviar dados a um
painel. O ESP32 passa a **buscar informação do mundo** em vez de só mostrar
o que ele mesmo calcula.

## O que você vai aprender

No final desta aula, você vai entender:

- que agora o **ESP32 será o cliente**;
- como fazer um **HTTP GET** simples;
- o que é o **código de resposta** HTTP;
- como **ler a resposta** como texto;
- o básico de **JSON** de forma visual;
- como **extrair um ou dois campos**;
- como reconhecer **erros de conexão**;
- o que é **timeout**;
- um pequeno **desafio**.

## ESP32 como cliente

Lembre do restaurante da Aula 13: **cliente** é quem faz o **pedido**;
**servidor** é quem **entrega**. Agora:

```
servidor na internet (ex.: uma API)  ←──pedido HTTP──  ESP32 (cliente)
        ───────────────resposta/JSON──────>
```

O ESP32 **pede** e mostra o que recebe. Isso é um **HTTP GET**: uma
"pergunta simples" ao servidor, que responde com os dados.

## O que é uma API

**API** é um "serviço pronto" na internet com o qual programas podem
conversar por HTTP. Pense nela como um **garçom de aplicativos**: você pede
um dado e ele te traz, já no formato certo para o programa entender.

Para esta aula, vamos usar uma **API pública e gratuita** que não exige
chave secreta. Um exemplo comum é uma API de **citações** ou de **hora/cotação**.
Use um endpoint público simples e estável.

> ⚠️ **Importante:** se a API exigir uma **chave/token**, **não coloque a
> chave no repositório** — use placeholder e um arquivo separado (Aula 12).
> Aqui, vamos usar uma API **sem chave** para simplificar.

## Ligação

Nenhuma ligação externa. Só a placa, o cabo USB e a **rede Wi‑Fi** (Aula 12).

Pode ser útil ter o **Monitor Serial** aberto na velocidade `115200`.

## Antes de ligar

- Conecte à **mesma rede Wi‑Fi** de antes.
- A API usada precisa ser **pública e simples**. Evite APIs que exijam
  cadastro ou chave nesta primeira aula.
- Lembre: **nunca** publique chaves no repositório.

## Primeiro teste: HTTP GET simples

Vamos fazer uma requisição GET e mostra `código de resposta` + o `texto`
recebido.

```cpp
// Aula 14 — HTTP GET simples

#include <WiFi.h>
#include <HTTPClient.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

const char* URL = "COLOQUE_AQUI_UMA_URL_PUBLICA";

void setup() {
  Serial.begin(115200);

  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("Conectado!");

  HTTPClient http;

  Serial.print("Pedindo: ");
  Serial.println(URL);

  http.begin(URL);
  int codigo = http.GET();

  if (codigo > 0) {
    Serial.print("Codigo HTTP: ");
    Serial.println(codigo);
    String resposta = http.getString();
    Serial.println("Resposta:");
    Serial.println(resposta);
  } else {
    Serial.print("Erro de conexao, codigo: ");
    Serial.println(codigo);
  }

  http.end();
}

void loop() {
}
```

### Entendendo o código

```cpp
#include <HTTPClient.h>
```

Inclui a biblioteca **HTTPClient**, que cuida de fazer os pedidos HTTP.

```cpp
HTTPClient http;
```

Cria um objeto `http` (um "cliente" pronto para pedir).

```cpp
http.begin(URL);
```

**`begin(URL)`** **prepara** o cliente para aquele endereço.

```cpp
int codigo = http.GET();
```

**`GET()`** faz o pedido e devolve o **código de resposta**. Esse é o lugar
do timeout: se o servidor demorar demais, `GET()` **desiste** e devolve um
valor negativo. O tempo máximo de espera é o **timeout** (quanto tempo o
cliente aguenta esperar antes de desistir). Você pode ajustá-lo com
`http.setTimeout(...)` se quiser.

```cpp
if (codigo > 0) {
```

Se o `codigo` **for maior que 0**, significa que a comunicação funcionou (o
código HTTP típico de sucesso é **200**). Se for **negativo ou zero**, houve
**erro de conexão** (rede caiu, servidor fora do ar, timeout).

```cpp
String resposta = http.getString();
```

**`getString()`** baixa o **corpo da resposta** (os dados em si) e guarda
numa **`String`** (um tipo que guarda texto).

```cpp
http.end();
```

Encerra a conexão (libera os recursos).

## O que deve acontecer

No Serial, você deve ver o **código de resposta** (geralmente `200` = tudo
certo) e, abaixo, o **texto** que a API devolveu. O formato desse texto
depende da API — pode ser **JSON** (vamos ver já já).

Se aparecer **erro de conexão** (código negativo), confira a rede e a URL.

## Introdução ao JSON (de forma visual)

Muitas APIs devolvem os dados em **JSON** — um formato de texto que organiza
informações em pares `"nome": valor`, separados por vírgulas e dentro de
chaves `{ }`.

Veja um exemplo (não é da nossa API, só para visualizar):

```json
{
  "cidade": "Sao Paulo",
  "temperatura": 24
}
```

Pense no JSON como uma **ficha organizada**:

- `"cidade"` é um **rótulo** (nome do campo);
- `Sao Paulo` é o **valor** desse campo;
- `"temperatura"` é outro rótulo, com o valor `24`.

Quando o ESP32 recebe um JSON, ele quer **extrair** o valor de um campo
específico — por exemplo, pegar o número que está em `"temperatura"`. Para
isso, usamos a **biblioteca ArduinoJson**.

## Extraindo um campo do JSON

Vamos instalar a biblioteca **ArduinoJson** (no Gerenciador de Bibliotecas,
pesquise por **ArduinoJson** e instale a oficial). Depois, este código pega a
resposta e extrai um campo:

```cpp
// Aula 14 — Extraindo um campo do JSON

#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

const char* URL = "COLOQUE_AQUI_UMA_URL_PUBLICA";

void setup() {
  Serial.begin(115200);

  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("Conectado!");

  HTTPClient http;
  http.begin(URL);
  int codigo = http.GET();

  if (codigo > 0) {
    String resposta = http.getString();

    JsonDocument doc;
    DeserializationError erro = deserializeJson(doc, resposta);

    if (erro) {
      Serial.print("Falha ao ler JSON: ");
      Serial.println(erro.c_str());
    } else {
      const char* campo = doc["CAMPO"];
      Serial.print("Valor do campo: ");
      Serial.println(campo);
    }
  } else {
    Serial.println("Erro de conexao.");
  }

  http.end();
}

void loop() {
}
```

### Entendendo as partes novas

```cpp
#include <ArduinoJson.h>
```

Inclui a biblioteca **ArduinoJson**.

```cpp
JsonDocument doc;
```

Cria um "documento" que vai guardar o JSON **parsed** (interpretado). O
`deserializeJson` transforma o **texto** JSON em algo com que o programa
consegue trabalhar.

```cpp
DeserializationError erro = deserializeJson(doc, resposta);
```

**`deserializeJson(doc, texto)`** **interpreta** o texto JSON e preenche o
`doc`. Se der erro (a resposta não é um JSON válido), devolve um erro.

```cpp
if (erro) {
  ... mostra erro ...
}
```

Se a leitura falhou (JSON malformado ou resposta não é JSON), avisamos.

```cpp
const char* campo = doc["CAMPO"];
Serial.println(campo);
```

**`doc["CAMPO"]`** acessa o valor do campo chamado `CAMPO`. Troque `"CAMPO"`
pelo **rótulo real** da sua API (ex.: `doc["temperatura"]`). Assim extraímos
o valor e mostramos.

> Se o valor for um **número** (não texto), use `int valor = doc["campo"];`
> ou `float` — o ArduinoJson converte o tipo automaticamente.

## Erros de conexão e timeout

Dois pontos para reconhecer:

- **Erro de conexão** (código negativo): a rede caiu, o endereço está
  errado, ou o servidor não respondeu a tempo. Verifique SSID/senha e a URL.
- **Timeout**: o cliente espera um tempo máximo pela resposta. Se o servidor
  estiver lento, `GET()` devolve erro **mesmo a rede estando OK**. Você pode
  aumentar o tempo com `http.setTimeout(15000)` (15 segundos) se precisar.

## Se não funcionar

- **código negativo / erro de conexão**: confira a Wi-Fi e se a URL está
  correta e acessível (teste pelo navegador).
- **"Falha ao ler JSON"**: a resposta pode não ser JSON válido. Imprima a
  `resposta` **crua** no Serial para inspecionar o que veio.
- **Campo vazio/nulo**: o nome do campo no `doc["..."]` não corresponde ao
  da API. Copie o rótulo **exato** da resposta (maiúsculas/minúsculas
  contam).
- **Compila, mas nada no Serial**: confira velocidade `115200` e a conexão.

## Experimente você

Agora, o desafio:

1. **Extraia um número**: se a sua API tem um campo numérico, mude o código
   para ler com `int valor = doc["campo"];` e mostre.
2. **Mostre em um display**: combine com a Aula 11 (ST7789) e mostre o valor
   extraído **na tela** em vez de só no Serial. Assim você tem um "painel"
   que busca dados do mundo.
3. **Mude a API**: use outra API pública e ajuste os nomes dos campos para
   extrair uma informação diferente.

## O que aprendemos

Nesta aula:

- o **ESP32** agora é o **cliente** (faz pedidos), usando **HTTP GET**;
- a resposta tem um **código** (200 = sucesso) e um **corpo** (texto/JSON);
- usamos **HTTPClient** com `begin`, `GET`, `getString` e `end`;
- **JSON** é texto estruturado em campos `"nome": valor`;
- usamos **ArduinoJson** para **extrair um campo**;
- reconhecemos **erros de conexão** e **timeout**;
- **chaves/tokens nunca vão para o repositório**.

## Próximo passo

HTTP serve para pedidos pontuais. Mas, em IoT, muitas vezes queremos uma
conversa contínua entre dispositivos. É aí que entra o **MQTT** — um
protocolo feito para isso, com "publicar" e "assinar" mensagens.

Nos vemos na Aula 15.
