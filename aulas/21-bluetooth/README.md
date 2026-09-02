# Aula 21 — Bluetooth e BLE

## O que vamos fazer

Até aqui, a comunicação sem fio foi via **Wi‑Fi** (internet, Aulas 12 a 20).
Mas existe outra forma de o ESP32 conversar **sem rede**: o **Bluetooth**.
Ele conecta o ESP32 **diretamente** ao seu celular ou a outro dispositivo,
sem precisar de roteador ou internet.

Nesta aula vamos entender a diferença entre **Bluetooth clássico** e **BLE**
(Bluetooth *Low Energy*, "baixo consumo"), e fazer o ESP32:
1. virar um **dispositivo BLE** que o celular pode **encontrar** e **conectar**;
2. **enviar uma mensagem** para um app no celular;
3. e **receber** um "comando" do celular para **controlar um LED**.

Por que isso é útil? É como o nosso ESP32 conversa com **aplicativos de
celular** em projetos como controle de brinquedos, sensores vestíveis e
automação — tudo sem precisar configurar Wi‑Fi.

## O que você vai aprender

No final desta aula, você vai entender:

- as diferenças entre **Bluetooth clássico** e **BLE**;
- os termos básicos do BLE: **periférico**, **central**, **GATT**, **serviço**,
  **característica**;
- como fazer o ESP32 **anunciar** (ser descoberto);
- como **enviar** dados ao celular;
- como **receber** comandos e **ligar o LED**;
- por que **BLE economiza bateria**.

## Bluetooth clássico vs BLE

- **Bluetooth clássico**: usado para coisas com **muito dado**, como fones e
  transferência de arquivos. Consome mais energia.
- **BLE** (*Bluetooth Low Energy*): feito para **poucos dados** com **baixo
  consumo** de bateria — perfeito para sensores e pequenos comandos.

Para IoT e projetos com ESP32, o **BLE** é o mais comum. É o que vamos usar.

```
Bluetooth clássico:  muito dado, mais energia (fones, arquivos)
BLE:                 poucos dados, pouca energia (sensores, comandos)
         └─> usaremos BLE
```

## Termos básicos do BLE (analogia)

O BLE usa uma organização de "serviços". Uma analogia: um **boletim de notas**

- **Periférico**: quem **anuncia** e **disponibiliza** dados (o ESP32).
- **Central**: quem **procura** e **se conecta** (o celular).
- **GATT**: o "protocolo de organização" — as regras de como os dados são
  trocados. Não precisa decorar o nome, só saber que estrutura os dados.
- **Serviço**: um **bloco** de dados relacionados (ex.: "Comandos").
- **Característica**: um **dado específico** dentro do serviço (ex.: "Ligar
  LED"). Pode ser **lido** ou **escrito** pelo celular.

```
ESP32 (periférico)
 └── Serviço "Controle"
      └── Característica "LED" (o celular pode escrever aqui)
      └── Característica "Mensagem" (o celular pode ler aqui)
```

## A biblioteca e um aviso de versão

Para BLE no ESP32 usamos a biblioteca **BluetoothSerial** (já vem no ESP32)
para o caso simples, ou a **BLEDevice** para o BLE completo. Nesta aula vamos
mostrar os dois caminhos de forma introdutória.

**Aviso importante:** a API de BLE (`BLEDevice`, `BLEServer`, etc.) foi
**atualizada ao longo das versões** do ESP32 Arduino Core, e alguns trechos
mudam. **Não copie scripts de internet sem verificar** a versão da sua IDE.
Acompanhe os **exemplos oficiais** que vêm com a sua instalação (menu
Arquivo → Exemplos → BLE) e **adapte**. O **conceito** de serviço/característica
é o mesmo; a sintaxe pode variar.

## Primeiro teste: Bluetooth Serial (simples)

O caminho mais fácil para começar é o **BluetoothSerial**: transforma o
Bluetooth clássico em uma **porta serial sem fio** — o celular "fala" com o
ESP32 como se fosse o cabo USB.

```cpp
// Aula 21 — Bluetooth Serial (porta serial sem fio)

#include "BluetoothSerial.h"

BluetoothSerial SerialBT;

void setup() {
  Serial.begin(115200);
  SerialBT.begin("MeuESP32");
  Serial.println("Olha o Bluetooth no celular!");
}

void loop() {
  if (SerialBT.available()) {
    SerialBT.write(SerialBT.read());
  }
  delay(20);
}
```

### Entendendo o código

```cpp
#include "BluetoothSerial.h"
```

Inclui a biblioteca **BluetoothSerial**.

```cpp
BluetoothSerial SerialBT;
```

Cria um objeto `SerialBT` — uma segunda "porta serial", só que via
**Bluetooth**.

```cpp
SerialBT.begin("MeuESP32");
```

**`begin(nome)`** inicia o Bluetooth com o **nome** `"MeuESP32"` (o nome que
aparece para outros dispositivos).

```cpp
if (SerialBT.available()) {
  SerialBT.write(SerialBT.read());
}
```

Se chegar algo (`available()`), **lê** (`read()`) e **devolve** (`write()`).
No celular, se você usar um app de "terminal Bluetooth Serial", tudo que
digitar aparece no ESP32 e é ecoado de volta.

### Como testar?

1. Suba o código com o **cabo USB** (para programar).
2. No **celular**, ligue o **Bluetooth** e **emparelhe** com `"MeuESP32"`.
3. Use um app de **terminal Bluetooth Serial** (pesquise por "Serial Bluetooth
   Terminal") para se conectar e **trocar mensagens**.

O **cabo USB já pode ser tirado** — a energia vem do celular? Não: o ESP32
precisa ser **alimentado** (por bateria, power bank ou o cabo). Só a
**comunicação** passa a ser por Bluetooth.

> ⚠️ Alimentação: retire o cabo USB só se o ESP32 estiver **alimentado por
> outra fonte** (5 V), senão ele desliga. Relembrando a regra 3,3 V/5 V da
> Aula 01.

## Segundo teste: BLE — enviar e receber com serviço/característica

Agora o jeito "BLE de verdade": montamos um **serviço** com duas
**características**, uma que o celular **lê** (mensagem) e outra em que o
celular **escreve** para **controlar o LED**.

Como o código BLE completo é extenso e varia entre versões, vamos apresentar
a **estrutura essencial** e recomendar fortemente que você **comece dos
exemplos oficiais** da sua IDE (ex.: "BLE_server") e faça as adaptações
indicadas. Abaixo, um esqueleto comentado para você entender o fluxo:

```cpp
// Aula 21 — Esqueleto de BLE (use o exemplo oficial da sua IDE e adapte)

#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

// 1. Defina o endereço (UUID) do serviço e das características.
#define SERVICO_UUID   "12345678-1234-1234-1234-123456789abc"
#define CAR_LED_UUID   "12345678-1234-1234-1234-123456789abd"
#define CAR_MSG_UUID   "12345678-1234-1234-1234-123456789abe"

// 2. O callback recebe quando o celular ESCREVE na característica "LED".
class MeuCallback : public BLECharacteristicCallbacks {
  void onWrite(BLECharacteristic* c) {
    String valor = c->getValue().c_str();
    if (valor == "on") {
      digitalWrite(2, HIGH);
    } else if (valor == "off") {
      digitalWrite(2, LOW);
    }
  }
};

void setup() {
  Serial.begin(115200);
  pinMode(2, OUTPUT);

  // 3. Cria o servidor BLE e o servico.
  BLEDevice::init("MeuESP32BLE");
  BLEServer* servidor = BLEDevice::createServer();
  BLEService* servico = servidor->createService(SERVICO_UUID);

  // 4. Cria a caracteristica "LED" (escrita pelo celular) e registra o callback.
  BLECharacteristic* carLed =
      servico->createCharacteristic(CAR_LED_UUID, BLECharacteristic::PROPERTY_WRITE);
  carLed->setCallbacks(new MeuCallback());

  // 5. (Opcional) Cria a caracteristica "Mensagem" (lida pelo celular).
  //    BLECharacteristic* carMsg =
  //        servico->createCharacteristic(CAR_MSG_UUID, BLECharacteristic::PROPERTY_READ);

  servico->start();

  // 6. Começa a anunciar (ser descoberto).
  BLEAdvertising* anuncio = BLEDevice::getAdvertising();
  anuncio->addServiceUUID(SERVICO_UUID);
  anuncio->start();

  Serial.println("BLE ativo. Procure 'MeuESP32BLE' no celular.");
}

void loop() {
}
```

### Entendendo o essencial

- **`BLEDevice::init(nome)`** inicia o BLE com esse nome.
- **`createServer()`** cria o **periférico**.
- **`createService(UUID)`** cria o **serviço** (o bloco de dados).
- **`createCharacteristic(UUID, propriedade)`** cria a **característica**;
  `PROPERTY_WRITE` permite o celular **escrever**; `PROPERTY_READ` permite
  **ler**.
- **`setCallbacks(...)`** registra a função que roda quando o celular
  **escreve** (o nosso `onWrite`), onde ligamos/desligamos o LED.
- **`getAdvertising()->start()`** faz o ESP32 **anunciar** que está lá
  (ser encontrado pelo celular).
- O **UUID** é um "endereço" longo das características. Num projeto real com
  apps, você define/usa os seus; para aprender, os valores acima servem.

> Como o app que controla é **seu** (ou um app BLE genérico), o UUID combinado
> entre o ESP32 e o app precisa ser o **mesmo**. Por isso, recomenda-se
> começar pelos **exemplos oficiais**, que já vêm com UUIDs prontos.

### Como testar?

1. Suba o código (cabo USB) e abra o Serial (115200).
2. No celular, use um **app BLE** (ex.: "nRF Connect" ou "LightBlue"). Ele
   **descobre** o `"MeuESP32BLE"`.
3. Conecte, encontre o serviço e a característica "LED", e **escreva** `on`
   ou `off`. O **LED** deve acender/apagar.

Para **ler** a "Mensagem", use o app BLE para **ler** a característica
`PROPERTY_READ` — o app mostra o valor que você definir.

## BLE economiza bateria

O **BLE** envia **poucos dados** e pode "dormir" entre comunicações — por isso
**consome menos**. É ideal quando o ESP32 roda **na bateria** e fica
"adormecido", acordando só quando precisa se comunicar. (Essa otimização de
baixo consumo é um tópico avançado; aqui basta entender o motivo do BLE.)

```
Wi‑Fi:  mais energia, precisa de rede
BLE:    menos energia, conexão direta sem rede
     └─> ideal para bateria
```

## Se não funcionar

- **Não aparece no celular**: certifique-se de que o **Bluetooth** do celular
  está ligado; que o ESP32 está **alimentado**; e que o **código** subiu (veja
  o Serial).
- **No Bluetooth Serial, não conecta**: pode ser preciso **esquecer** o
  dispositivo no celular e **emparelhar de novo**.
- **No BLE, não encontra o serviço/característica**: o **UUID** do ESP32 e do
  app precisam **coincidir**. Use os dos **exemplos oficiais**.
- **Escreve `on` mas o LED não liga**: confira o **pino**, o callback
  registrado e se a mensagem é exatamente `on`/`off`.
- **Compila com erro na sua versão**: a **API BLE mudou entre versões**.
  Adapte conforme os **exemplos da sua IDE** (ver aviso acima).

## Experimente você

Agora, os desafios:

1. **Eco Bluetooth**: no Bluetooth Serial, envie seu nome do app e veja o
   ESP32 ecoar no Serial (e vice-versa).
2. **Dois comandos**: no BLE, além de `on`/`off` no LED, adicione uma
   característica para **piscar** (ligar/desligar a cada 500 ms) — use
   `millis()` (Aula 04) no `loop()`.
3. **Envie dados**: faça o ESP32 **publicar** periodicamente (via BLE) o valor
   de um sensor (Aula 06) para o app ler.

## O que aprendemos

Nesta aula:

- **Bluetooth clássico** (muito dado/energia) vs **BLE** (pouco dado/baixa
  energia);
- **periférico** (ESP32) e **central** (celular); **GATT** organiza os dados;
  **serviço** é um bloco, **característica** é um dado;
- com **BluetoothSerial**, o celular vira uma **porta serial sem fio**;
- com **BLE**, montamos **serviço/características** para o celular **ler**
  (mensagem) e **escrever** (LED);
- a **API BLE varia com a versão** — sempre adaptar pelos **exemplos oficiais**;
- **BLE economiza bateria** — bom para dispositivos que rodam em bateria.

## Próximo passo

O ESP32 já sabe conversar por fios (I2C, SPI), pela internet (Wi‑Fi, HTTP,
MQTT) e sem fio direto (Bluetooth). Falta um último tipo de comunicação:
**áudio digital**. Na próxima aula, veremos como o ESP32 pode **tocar som**
usando **I2S**.

Nos vemos na [Aula 22](../22-audio-digital/README.md).