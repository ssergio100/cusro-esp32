# Aula 19 — Salvar configurações (NVS / Preferences)

## O que vamos fazer

Até aqui, quando você **desligava** o ESP32, tudo o que as variáveis guardavam
**sumia** (a RAM perde tudo, como vimos na Aula 18). Mas e se quisermos que
um dado **continue lá mesmo depois de desligar** — por exemplo, o SSID da
rede, um valor calibrado, o estado de uma preferência?

O ESP32 tem um espaço permanente para isso: o **NVS** (*Non-Volatile
Storage*, "armazenamento não volátil"). Na prática, usamos a biblioteca
**Preferences**, que torna essa tarefa bem fácil — como guardar coisas numa
prateleira que **não esvazia ao desligar**.

Nesta aula vamos **salvar um valor**, **ler** depois do religar, **atualizá-lo**
e aprender os cuidados (inclusive com **SSID/senha**).

## O que você vai aprender

No final desta aula, você vai entender:

- o que é **NVS/Flash** para dados;
- a biblioteca **Preferences**;
- os conceitos de **namespace** e **chave**;
- como **salvar**, **ler**, **atualizar** e **apagar**;
- quando essa memória **mais ajuda** (e seus limites);
- um **exemplo com Wi‑Fi** mostrando uso real;
- por que **não salvar SSID/senha no repositório**.

## A "prateleira que não esvazia"

A RAM (mesa de trabalho) perde tudo ao desligar. O **NVS** é um espaço da
**Flash** reservado para **pequenos dados permanentes** — configurações,
contadores, estados.

Pense assim:

```
RAM      = mesa de trabalho (some ao desligar)
NVS/Flash = prateleira permanente (continua guardado)
```

A ideia é guardar pequenas informações, não arquivos grandes (arquivos
grandes ficam para a Aula 20).

## Namespace e chave: como o armazenamento se organiza

A `Preferences` usa duas ideias simples:

- **Namespace**: é o "nome da prateleira" — agrupa suas preferências (ex.:
  `"bancada"`, `"sensor"`). Como se fosse uma pasta.
- **Chave (key)**: é o "rótulo" de um valor **dentro** da prateleira (ex.:
  `"nome"`, `"valor"`, `"contador"`).

```
prateleira "bancada"
   ├── rótulo "nome"    → "Aluno"
   └── rótulo "contador" → 7
```

Para ler/escrever, você abre o **namespace**, então acessa pela **chave**.

## Primeiro teste: salvar e ler um contador

Vamos fazer um programa que **guarda um número** (um contador de quantas
vezes o ESP32 religou) e lê na próxima vez. É o exemplo clássico e
gratificante de NVS.

```cpp
// Aula 19 — Contador que sobrevive a desligar

#include <Preferences.h>

Preferences prefs;

void setup() {
  Serial.begin(115200);

  prefs.begin("bancada", false);

  int contador = prefs.getInt("contador", 0);
  contador++;
  prefs.putInt("contador", contador);

  Serial.print("Vezes que o ESP32 ligou: ");
  Serial.println(contador);

  prefs.end();
}

void loop() {
}
```

### Entendendo o código

```cpp
#include <Preferences.h>
```

Inclui a biblioteca **Preferences** (já vem no ESP32, não precisa instalar).

```cpp
Preferences prefs;
```

Cria um objeto `prefs` (nossa "prateleira").

```cpp
prefs.begin("bancada", false);
```

**`begin(nomeNamespace, somenteLeitura)`** **abre** o namespace; o segundo
argumento `false` significa **leitura e escrita** (permitimos salvar). Se
fosse `true`, só abriríamos para **ler**.

```cpp
int contador = prefs.getInt("contador", 0);
```

**`getInt(chave, valorPadrao)`** **lê** o valor da chave. Se ainda **não
existir** (primeira vez), usa o **valor padrão** `0`.

```cpp
contador++;
prefs.putInt("contador", contador);
```

**`putInt(chave, valor)`** **salva** o número atualizado na chave.

```cpp
Serial.println(contador);
```

Mostra quantas vezes o ESP32 já ligou.

```cpp
prefs.end();
```

**`end()`** **fecha** a prateleira (boa prática). Depois, o valor continua
guardado para a próxima vez que ligar.

> **Teste:** religue o ESP32 (desligue e ligue, ou aperte o reset). O número
> no Serial **continua aumentando** — o valor **sobreviveu** ao desligar!
> Se fosse uma variável normal na RAM (Aula 18), ele voltaria sempre a `0`.

## O básico da API

Veja os principais métodos (são bem intuitivos):

| Tipo | Grava | Lê |
|------|-------|----|
| inteiro | `prefs.putInt("chave", valor)` | `prefs.getInt("chave", padrao)` |
| decimal | `prefs.putFloat("chave", valor)` | `prefs.getFloat("chave", padrao)` |
| texto | `prefs.putString("chave", "texto")` | `prefs.getString("chave", "padrao")` |
| sim/não | `prefs.putBool("chave", true)` | `prefs.getBool("chave", false)` |

E para **apagar**:

- **`prefs.remove("chave")`** remove uma **chave**;
- **`prefs.clear()`** **esvazia todo o namespace**.

Escolha o tipo certo: usar `putInt` para inteiros, `putFloat` para decimais,
`putString` para texto etc.

## Exemplo real: salvar o SSID

Um uso muito comum: salvar o **SSID** da rede para que o ESP32 se reconecte
sem você alterar o código. Mas **atenção ao aviso abaixo**.

```cpp
// Aula 19 — Salvar e recuperar texto (ex.: SSID)

#include <Preferences.h>

Preferences prefs;

void definirSSID(String novoSSID) {
  prefs.begin("rede", false);
  prefs.putString("ssid", novoSSID);
  prefs.end();
  Serial.print("SSID salvo: ");
  Serial.println(novoSSID);
}

String carregarSSID() {
  prefs.begin("rede", true);
  String ssid = prefs.getString("ssid", "ainda_nao_definido");
  prefs.end();
  return ssid;
}

void setup() {
  Serial.begin(115200);

  definirSSID("MinhaRede");
  Serial.print("SSID gravado: ");
  Serial.println(carregarSSID());
}

void loop() {
}
```

### Entendendo

Aqui montamos duas **funções reutilizáveis**:

- **`definirSSID(novoSSID)`** abre o namespace `"rede"` para escrita e salva o
  SSID na chave `"ssid"`.
- **`carregarSSID()`** abre para **leitura** (`true`) e devolve o SSID
  salvo (ou um valor padrão se ainda não existir).

Assim, um lugar **central** grava e um lugar **central** lê — e o resto do
programa só chama `carregarSSID()` quando precisar da rede.

> ⚠️ **Nunca grave SSID/senha "de verdade" no código do repositório.**
> Use sempre **placeholder** (ex.: `"SEU_SSID"`). O **valor real** pode ser
> **digitado pelo usuário** e salvo no NVS em **tempo de execução** (por
> exemplo, vindo de um formulário web — como na Aula 13→14), sem nunca ir
> para o código. Assim, nenhum segredo entra no repositório.

## Cuidados com NVS

- **É para dados pequenos**: poucos bytes por valor. Para tudo grande
  (imagens, logs, arquivos), use a Flash de arquivos da **Aula 20**.
- **Tem limites de escrita**: a Flash se desgasta com gravações demais.
  Evite ficar salvando a cada milissegundo no `loop()`. Salve quando algo
  **mudar** ou em intervalos moderados.
- **`begin` com o segundo argumento**: use `false` para escrever, `true`
  para só ler.
- **Sempre verificar**: na dúvida sobre o espaço/tamanho da sua placa,
  consulte a ficha (lembre da Aula 18 — nunca presumir).

## Se não funcionar

- **Contador não aumenta**: confirmar que `prefs.end()` não é chamado antes
  de ler (nesta aula lemos e gravamos em sequência). E que o namespace/chave
  são os mesmos entre gravar e ler.
- **Valor sempre volta ao padrão**: se você **compila/reenvia** o sketch, o NVS
  pode ser preservado, mas **alguns** "apagar flash" zeram tudo. Confirme se
  está **só** religando/resetando o ESP32 (não apagando a flash).
- **Não compila `Preferences`**: a biblioteca já vem com o ESP32 Arduino
  Core. Em versões antigas, garanta que esteja incluída.
- **Apagar acidentalmente tudo**: cuidado com `prefs.clear()` — remove o
  namespace inteiro.

## Experimente você

Agora, os desafios:

1. **Salve um texto**: use `putString`/`getString` para guardar seu **nome** e
   mostre-o no Serial após religar.
2. **Guarde uma preferência**: salve um valor com `putFloat` (ex.: o limite
   de um sensor) e leia com `getFloat`.
3. **Some com Wi‑Fi**: combine com a Aula 12 — guarde a rede e a **reestabeleça**
   com esse valor, em vez de SSID fixo no código.

## O que aprendemos

Nesta aula:

- o **NVS/Preferences** guarda **pequenos dados permanentes** mesmo com o
  ESP32 desligado;
- usamos `begin`, `getXxx`, `putXxx`, `end`, `remove` e `clear`;
- **namespace** é a "prateleira" e **chave** o "rótulo";
- escolhemos o **tipo certo** (`int`, `float`, `String`, `bool`);
- **SSID/senha reais nunca** entram no repositório — podem ser gravados no
  NVS em tempo de execução com placeholder;
- NVS é para **dados pequenos** e tem **limites de gravação**;
- dados grandes vão para a Flash de arquivos (Aula 20).

## Próximo passo

Salvamos valores pequenos. Mas e se quisermos salvar **arquivos inteiros** —
como textos, imagens ou dados de configuração grandes? Para isso existe o
**LittleFS**: o ESP32 pode criar e ler **arquivos** como um computador.

Nos vemos na [Aula 20](../20-arquivos/README.md).