# Aula 12 — Wi‑Fi

## O que vamos fazer

Até agora o ESP32 "falou" com o computador pelo cabo USB. Nesta aula vamos
dar ao ESP32 o seu grande superpoder: **conectar a uma rede Wi‑Fi** e receber
um **endereço IP** — ou seja, virar um dispositivo na rede, sem fio.

Por que isso é útil? É o primeiro passo para tudo que vem: o ESP32 como
**servidor web** (Aula 13), conversando com servidores (Aula 14), MQTT (Aula
15) e, no fim, um projeto conectado de verdade.

## O que você vai aprender

No final desta aula, você vai entender:

- o que é **SSID** e **senha**;
- como usar **`WiFi.begin()`**;
- como **acompanhar a conexão** pelo Monitor Serial;
- o que é **`WiFi.status()`**;
- como mostrar o **IP local** recebido;
- o que é um **IP** (de forma simples);
- como fazer uma **reconexão básica**;
- por que **não publicar senhas** no Git;
- como usar **placeholders ou arquivo separado** para credenciais;
- um pequeno **desafio**: mostrar a **intensidade do sinal**.

## O que é SSID e senha

O **SSID** é simplesmente o **nome da sua rede Wi‑Fi** (aquele nome que
aparece no celular quando você procura redes para conectar). A **senha** é a
chave da rede.

Para o ESP32 entrar na sua rede, você precisa dizer a ele o **SSID** e a
**senha**. Sem isso, ele não sabe para onde se conectar.

## Credenciais: segurança importante

As instruções desta aula pedem para você colocar o **seu** SSID e senha no
programa. **Atenção:** o código com a senha **não deve ir para o
repositório Git** (Aula 00 / AGENTS.md) — senão sua senha fica exposta.

A solução mais simples: crie um **arquivo separado** (por exemplo,
`credenciais.h`) que **não** seja commitado, e coloque as credenciais lá. Ou
use **placeholders** no código mostrado e preencha você mesmo com os seus
valores, sem publicar.

> 📌 **Regra de ouro:** nunca commite o **SSID e senha reais**. Use
> placeholders (ex.: `"SEU_SSID"` e `"SUA_SENHA"`) no repositório.

## Ligação

Nesta aula **não precisamos de circuito nem de fios** — a conexão Wi‑Fi é
**sem fio** (antena embutida na placa). Só precisamos:

- a placa **ESP32**;
- o **cabo USB** (apenas para enviar o programa e ver o Serial);
- uma **rede Wi‑Fi** a que você tem acesso.

## Antes de ligar

- Tenha o **SSID** e a **senha** da sua rede em mãos (não precisa digitar a
  senha real no repositório — use placeholder no código mostrado).
- O **Monitor Serial** deve estar na velocidade `115200`.
- Algumas redes (empresas, com certificados) são complicadas de conectar;
  para esta aula, use uma **rede doméstica comum** (ou o roteador do
  celular/`hotspot`).

## Primeiro teste: conectar e ver o IP

```cpp
// Aula 12 — Conexão Wi-Fi

#include <WiFi.h>

const char* SSID     = "SEU_SSID";     // troque pelo nome da rede
const char* SENHA    = "SUA_SENHA";   // troque pela senha

void setup() {
  Serial.begin(115200);
  delay(1000);

  Serial.println("Conectando ao Wi-Fi...");
  WiFi.begin(SSID, SENHA);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("Conectado!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());
}

void loop() {
}
```

### Entendendo o código

```cpp
#include <WiFi.h>
```

Inclui a biblioteca **WiFi**, que já vem com o ESP32 e cuida de toda a
comunicação sem fio.

```cpp
const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";
```

Guardamos o **SSID** e a **senha**. (`const char*` é só "como guardar texto
fixo" — não precisa decorar; o importante é que você coloca seus valores
entre as aspas.)

```cpp
WiFi.begin(SSID, SENHA);
```

**`WiFi.begin(ssid, senha)`** inicia a tentativa de conexão com aquela rede.

```cpp
while (WiFi.status() != WL_CONNECTED) {
  delay(500);
  Serial.print(".");
}
```

Um **laço `while`** (repete **enquanto** a condição for verdadeira). Aqui, ele
fica **esperando até conectar**: enquanto `WiFi.status()` **não for**
`WL_CONNECTED` (ou seja, ainda não conectado), mostra um ponto e espera meio
segundo.

- **`WiFi.status()`** devolve o **estado atual** da conexão.
- **`WL_CONNECTED`** é a constante que significa "conectado".
- Depois de conectar, o `while` para e o programa continua.

```cpp
Serial.println("Conectado!");
Serial.print("IP: ");
Serial.println(WiFi.localIP());
```

**`WiFi.localIP()`** devolve o **endereço IP** que o roteador deu ao ESP32.
Mostramos no Serial.

### O que é um IP?

Um **endereço IP** é como o **endereço da casa** do seu ESP32 na rede — um
número único (ex.: `192.168.1.42`) que identifica aquele dispositivo dentro
daquela rede. Assim como uma carta chega a uma casa pelo endereço, os dados
chegam ao ESP32 pelo **IP**.

## O que deve acontecer

No Monitor Serial, algo parecido com:

```
Conectando ao Wi-Fi...
........
Conectado!
IP: 192.168.1.42
```

O número exato do IP varia conforme a sua rede (costuma começar com
`192.168.`). **Anote o IP** — você vai usá-lo na Aula 13 para acessar a placa
pelo navegador.

Se aparecer uma longa sequência de pontos sem conectar, revise o SSID/senha
e a proximidade do roteador.

## Reconexão básica

O Wi‑Fi pode **cair** (o roteador reinicia, você sai de perto etc.). Sem
tratamento, o ESP32 continua "achando" que está conectado. Uma reconexão
básica e simples: **verificar de vez em quando** e, se perdeu a conexão,
tentar de novo. (Vamos usar o `millis` da Aula 04, sem bloquear.)

```cpp
// Aula 12 — Conexão com reconexão básica

#include <WiFi.h>

const char* SSID  = "SEU_SSID";
const char* SENHA = "SUA_SENHA";

unsigned long ultimaChecagem = 0;

void setup() {
  Serial.begin(115200);
  pinMode(2, OUTPUT);
  WiFi.begin(SSID, SENHA);
}

void loop() {
  if (millis() - ultimaChecagem >= 10000) {
    ultimaChecagem = millis();

    if (WiFi.status() != WL_CONNECTED) {
      Serial.println("Wi-Fi caiu. Reconectando...");
      WiFi.reconnect();
    }
  }

  digitalWrite(2, (WiFi.status() == WL_CONNECTED));
}
```

### Entendendo o que mudou

- **`millis() - ultimaChecagem >= 10000`** — a cada **10 segundos**, checa se
  ainda está conectado (reaproveita o conceito de tempo da Aula 04).
- **`WiFi.reconnect()`** — tenta **reconectar**.
- **`digitalWrite(2, (WiFi.status() == WL_CONNECTED))`** — uma linha esperta:
  o **LED interno** fica aceso quando conectado e apagado quando não. Assim
  você vê o estado **sem abrir o Serial**. (O `==` devolve `true`/`false`,
  que a placa trata como HIGH/LOW.)

## Se não funcionar

- **Nunca conecta (pontos para sempre)**: confira o SSID e a senha (detecta
  erros de digitação). Aproxime-se do roteador. Confirme se a rede é
  doméstica simples.
- **"Conectado", mas sem IP**: raramente, falta esperar um pouco. Ligue o
  cabo de novo e observe.
- **O LED não acende no exemplo de reconexão**: confira o pino do LED da sua
  placa (Aula 02) e se conectou antes.
- **Conectou, mas apaga depois de um tempo**: o roteador derrubou; a
  reconexão deve tratar isso (observe as mensagens).

## Experimente você

Agora, o desafio:

1. **Mostre a intensidade do sinal**: a função **`WiFi.RSSI()`** devolve a
   força do sinal (um número negativo; quanto mais perto de 0, melhor).
   Depois de conectar, mostre no Serial:

   ```cpp
   Serial.print("Sinal (RSSI): ");
   Serial.println(WiFi.RSSI());
   ```

   Mova-se para perto e depois para longe do roteador e observe o número
   mudar.
2. **Separe as credenciais**: crie um arquivo `credenciais.h` no mesmo sketch
   com o SSID/senha, inclua-o com `#include "credenciais.h"`, e nunca
   publique esse arquivo. Isso protege sua senha.
3. **LED de status**: no exemplo básico, faça o LED interno **piscar** quando
   ainda não estiver conectado (use o `millis` da Aula 04) e ficar **aceso**
   quando conectar. Assim o estado fica visível sem abrir o Serial.

## O que aprendemos

Nesta aula:

- o **SSID** é o nome da rede e a **senha** é a chave de acesso;
- **`WiFi.begin(ssid, senha)`** inicia a conexão;
- **`WiFi.status()`** mostra o estado e **`WL_CONNECTED`** é "conectado";
- **`WiFi.localIP()`** devolve o **endereço IP** na rede;
- usamos um **`while`** para esperar conectar e **`millis`** para a
  **reconexão** básica;
- **as credenciais nunca vão para o repositório** (usa-se placeholder ou
  arquivo separado).

## Próximo passo

Agora que o ESP32 está na rede com um IP, vamos transformá-lo em um
**servidor web**: uma página acessível pelo navegador que controla um LED.

Nos vemos na [Aula 13](../13-primeiro-servidor-web/README.md).
