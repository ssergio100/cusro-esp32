# Aula 15 — MQTT

## O que vamos fazer

O HTTP (Aula 14) é bom para pedidos pontuais: você pede, o servidor responde,
acabou. Mas em muitos projetos queremos uma **conversa contínua** — vários
dispositivos **enviando e recebendo** mensagens o tempo todo, como um grupo
de WhatsApp. Para isso existe o **MQTT**.

Nesta aula vamos entender o conceito de **publish/subscribe** ("publicar e
assinar") e fazer o ESP32 **entrar nessa rede de mensagens**: publicar o
estado de um **botão** e receber comandos para **controlar um LED**.

Por que isso é útil? MQTT é a base de muita "casa inteligente" e Internet das
Coisas: vários sensores publicam seus dados, e vários controles assinam para
recebê-los — tudo conversando pelo mesmo protocolo leve.

## O que você vai aprender

No final desta aula, você vai entender:

- o **broker** como "central de mensagens";
- o que é um **tópico**;
- **publisher** (quem publica);
- **subscriber** (quem assina);
- como **conectar o ESP32 ao broker**;
- como **publicar uma mensagem**;
- como **receber uma mensagem**;
- como **controlar um LED via tópico**;
- como **publicar o estado de um botão/sensor**;
- o **fluxo completo**.

## O broker: a "central de mensagens"

No MQTT, os dispositivos **não falam diretamente** entre si. Todos conversam
com um **broker** — uma **central de mensagens** que fica no meio e repassa
tudo.

```
                     ┌── dispositivo A
ESP32 ─── conecta-se ao ───  broker ── dispositivo B
                     └── dispositivo C
```

O broker é como um **quadro de avisos**: alguém **cola** uma mensagem e
quem está **interessado** lê. Nenhum dispositivo precisa conhecer o outro —
só conhece o broker.

## Tópicos, publisher e subscriber

- **Tópico** é o "assunto" da mensagem — uma **etiqueta**, como
  `sala/luz` ou `cozinha/temperatura`. Pense nele como o **endereço temático**
  da mensagem.
- **Publisher** (publicador) é quem **publica** (envia) uma mensagem em um
  tópico.
- **Subscriber** (assinante) é quem **se inscreve** em um tópico para **ler**
  as mensagens que chegam ali.

```
publisher ──publica em─>  tópico "sala/luz"  <──assina── subscriber
```

Quem publica **não precisa saber** quem está ouvindo; quem assina **não
precisa saber** quem publicou. O **broker** repassa. Essa é a magia do
publish/subscribe.

## Ligação

Nesta aula usaremos:

- a placa **ESP32**;
- o **botão** (GPIO → botão → GND, da Aula 03);
- o **LED** (GPIO 2 ou o do seu LED interno, com resistor se externo);
- a **Wi-Fi** da Aula 12;
- um **broker MQTT público** de teste (não precisa de cadastro para
  aprender). Use um broker de teste que permita clientes anônimos, com um
  endereço padrão desse serviço.

> ⚠️ Brokers públicos de teste servem **só para aprender** — qualquer pessoa
> pode ver os tópicos. Não publique dados sensíveis ali.

## Antes de ligar

- Conecte à **mesma rede Wi‑Fi**.
- Tenha o **broker** de teste em mente (endereço e porta, normalmente a
  **1883**).
- Monte o **botão** e o **LED** conforme as Aulas 03/05.
- **Não** exponha o broker a dados sensíveis.

## Instalando a biblioteca

Para usar MQTT com o ESP32, instale a biblioteca **PubSubClient** (no
Gerenciador de Bibliotecas, pesquise por "PubSubClient" e instale). Ela cuida
de toda a conversa com o broker.

## Primeiro teste: conectar e publicar

Vamos conectar ao broker e **publicar** uma mensagem em um tópico.

```cpp
// Aula 15 — MQTT: conectar e publicar

#include <WiFi.h>
#include <PubSubClient.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

const char* BROKER = "ENDEREÇO_DO_BROKER";
const int PORTA = 1883;

WiFiClient wifi;
PubSubClient mqtt(wifi);

void setup() {
  Serial.begin(115200);

  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("Wi-Fi conectado!");

  mqtt.setServer(BROKER, PORTA);

  if (mqtt.connect("meu_esp32")) {
    Serial.println("Conectado ao broker!");
    mqtt.publish("curso/teste", "Ola do ESP32!");
  } else {
    Serial.print("Falha, estado: ");
    Serial.println(mqtt.state());
  }
}

void loop() {
  mqtt.loop();
}
```

### Entendendo o código

```cpp
#include <PubSubClient.h>
```

Inclui a biblioteca **PubSubClient** (MQTT).

```cpp
WiFiClient wifi;
PubSubClient mqtt(wifi);
```

Cria dois objetos: `wifi` (a conexão de rede) e `mqtt` (o cliente MQTT, que
"viaja" por cima do Wi‑Fi).

```cpp
mqtt.setServer(BROKER, PORTA);
```

**`setServer(endereco, porta)`** diz ao cliente **qual broker** usar.
`PORTA` normalmente é `1883`.

```cpp
if (mqtt.connect("meu_esp32")) {
```

**`connect(nome)`** tenta **se registrar** no broker com um **nome** (o
"apelido" do dispositivo). Devolve `true` se deu certo.

```cpp
mqtt.publish("curso/teste", "Ola do ESP32!");
```

**`publish(tópico, mensagem)`** **publica** a mensagem no tópico. Aqui, envia
o texto no tópico `curso/teste`.

```cpp
mqtt.state();
```

Se a conexão falhou, **`state()`** mostra o **código do erro** (para
diagnóstico).

```cpp
void loop() {
  mqtt.loop();
}
```

**`mqtt.loop()`** é **obrigatório** — é o que mantém viva a conversa: cuida de
enviar/receber mensagens e de reconectar se preciso. Sem ele, nada acontece.

### Como ver a mensagem?

Para **ver** a mensagem publicada, use um segundo assinante: um cliente MQTT
no seu **celular** ou computador (ex.: um app/cliente MQTT), conectado ao
**mesmo broker**, **assinando** o tópico `curso/teste`. Quando o ESP32
publicar, a mensagem aparece lá.

> Esse é o coração do MQTT: o **publisher** publica (ESP32) e o
> **subscriber** (seu app) recebe. Os dois só precisam compartilhar o mesmo
> broker e o mesmo tópico.

## Recebendo mensagens (subscriber)

Agora vamos fazer o **ESP32 assinar** um tópico e **reagir** a ele —
controlando o LED. Quando alguém publicar `on` ou `off` no tópico, o LED
liga ou desliga.

```cpp
// Aula 15 — MQTT: assinar e controlar LED

#include <WiFi.h>
#include <PubSubClient.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

const char* BROKER = "ENDEREÇO_DO_BROKER";
const int PORTA = 1883;

const int LED = 2;
const char* topicoLed = "curso/luz";

WiFiClient wifi;
PubSubClient mqtt(wifi);

void setup() {
  Serial.begin(115200);
  pinMode(LED, OUTPUT);

  WiFi.begin(SSID, SENHA);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("Wi-Fi conectado!");

  mqtt.setServer(BROKER, PORTA);
  mqtt.setCallback(receberMensagem);

  conectaBroker();
}

void loop() {
  if (!mqtt.connected()) {
    conectaBroker();
  }
  mqtt.loop();
}

void conectaBroker() {
  while (!mqtt.connected()) {
    Serial.println("Conectando ao broker...");
    if (mqtt.connect("meu_esp32")) {
      Serial.println("Conectado!");
      mqtt.subscribe(topicoLed);
    } else {
      Serial.print("Falha, estado: ");
      Serial.println(mqtt.state());
      delay(2000);
    }
  }
}

void receberMensagem(char* topico, byte* payload, unsigned int tamanho) {
  if (tamanho == 0) {
    return;
  }

  String mensagem = "";
  for (unsigned int i = 0; i < tamanho; i++) {
    mensagem += (char)payload[i];
  }

  Serial.print("Recebido em '");
  Serial.print(topico);
  Serial.print("': ");
  Serial.println(mensagem);

  if (mensagem == "on") {
    digitalWrite(LED, HIGH);
  } else if (mensagem == "off") {
    digitalWrite(LED, LOW);
  }
}
```

### Entendendo as partes novas

```cpp
mqtt.setCallback(receberMensagem);
```

**`setCallback(funcao)`** diz: "**quando** chegar uma mensagem em um tópico
que assinei, **chame** a função `receberMensagem`". É como cadastrar um
"alerta".

```cpp
mqtt.subscribe(topicoLed);
```

**`subscribe(tópico)`** **assina** o tópico: a partir daqui, o ESP32 vai
**receber** as mensagens publicadas nele.

```cpp
void loop() {
  if (!mqtt.connected()) {
    conectaBroker();
  }
  mqtt.loop();
}
```

A cada volta, **checa se ainda está conectado**; se não, **reconecta**. Só
então chama o `loop()` do MQTT. Isso dá uma **reconexão básica** sem travar.

```cpp
void receberMensagem(char* topico, byte* payload, unsigned int tamanho) {
```

Esta é a função de callback. O ESP32 chama ela sozinho com os **dados da
mensagem**:
- `topico` — em qual tópico chegou;
- `payload` — o **conteúdo** da mensagem (em bytes);
- `tamanho` — quantos bytes tem.

```cpp
String mensagem = "";
for (unsigned int i = 0; i < tamanho; i++) {
  mensagem += (char)payload[i];
}
```

**Junta os bytes** do payload em uma `String` (texto). O `for` percorre cada
byte e o `+=` vai montando o texto.

```cpp
if (mensagem == "on") {
  digitalWrite(LED, HIGH);
} else if (mensagem == "off") {
  digitalWrite(LED, LOW);
}
```

Se a mensagem for `"on"`, liga o LED; se `"off"`, desliga. (Reaproveita o
`if` da Aula 03.)

### Como testar?

Do seu app/cliente MQTT, **publique** no tópico `curso/luz` a mensagem `on`
e depois `off`. O LED deve **acender e apagar** de acordo.

## Publicando o estado de um botão

Para fechar o ciclo, vamos **publicar** o estado de um **botão**. Monte o
botão (Aula 03) e publique no broker que ele foi pressionado:

```cpp
// trecho: publicar estado do botão (use com a parte de conexão acima)

const int BOTAO = 4;

void setup() {
  // ... configuração anterior ...
  pinMode(BOTAO, INPUT_PULLUP);
}

void loop() {
  if (!mqtt.connected()) {
    conectaBroker();
  }
  mqtt.loop();

  if (digitalRead(BOTAO) == LOW) {
    mqtt.publish("curso/botao", "pressionado");
    delay(300);   // evita publicar milhares de vezes por segundo
  }
}
```

Aqui, sempre que o botão estiver **pressionado** (`== LOW`, lembra da Aula
03), o ESP32 **publica** `"pressionado"` no tópico `curso/botao`. Um assinante
do broker recebe na hora.

## O fluxo completo

Juntando tudo:

```
Botão (entrada) ──publica──> broker ──assina/entrega──> app (subscriber)

app (publisher) ──"on"──> tópico "curso/luz" ──assina──> ESP32 ──liga──> LED
```

```
sensor/botão → ESP32 → publish → broker → subscribe → outro dispositivo
comando (app) → broker → subscribe → ESP32 → GPIO → LED
```

O MQTT permite que **vários** publishers e subscribers conversem pelo mesmo
broker, cada um com seus tópicos.

## Se não funcionar

- **"Falha, estado"**: anote o código de `mqtt.state()` e confira o **broker**
  (endereço/porta) e a **Wi‑Fi**. Alguns brokers exigem que o nome do cliente
  seja único.
- **Conecta, mas não recebe**: confira o **tópico** — o assinante e o
  publisher precisam usar o **mesmo tópico exato** (maiúsculas contam).
- **LED não muda ao publicar**: confira se a mensagem é exatamente `on`/`off`
  (sem espaços) e se o pino do LED está certo.
- **Publica vários por segundo**: o `delay(300)` existe para evitar spam.
- **Não vejo no app**: confira se o app está no **mesmo broker** e assinou o
  mesmo tópico.

## Experimente você

Agora, os desafios:

1. **Publique um valor numérico**: use o potenciômetro (Aula 06) e publique a
   leitura como texto: `mqtt.publish("curso/sensor", String(valor).c_str());`
   (o `.c_str()` converte a `String` para o formato que o broker aceita).
2. **Dois tópicos**: faça o LED responder a `on`/`off` **e** publique o botão
   em outro tópico ao mesmo tempo.
3. **Teste com dois dispositivos**: se tiver dois ESP32, um publica (botão) e
   o outro assina (LED), ambos no mesmo broker e tópicos — sem conexão direta
   entre eles. Essa é a graça do MQTT.

## O que aprendemos

Nesta aula:

- o **broker** é a **central de mensagens** (ninguém fala direto com
  ninguém);
- um **tópico** é a "etiqueta/assunto" da mensagem;
- **publisher** publica; **subscriber** assina; o broker repassa;
- usamos **PubSubClient**: `setServer`, `connect`, `publish`, `subscribe`,
  `setCallback` e o `loop()` **obrigatório**;
- o **ESP32 controla um LED** recebendo mensagens (`on`/`off`) e **publica**
  o estado do botão;
- **brokers públicos de teste** são só para aprender — sem dados sensíveis.

## Próximo passo

Nosso ESP32 já "fica de olho" em botões checando a cada volta do `loop()`.
Mas existe um jeito de ele ser avisado **na hora** quando algo acontece — as
**interrupções**. Vamos ver isso na próxima aula.

Nos vemos na [Aula 16](../16-interrupcoes/README.md).
