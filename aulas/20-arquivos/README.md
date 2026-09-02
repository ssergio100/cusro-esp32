# Aula 20 — Arquivos com LittleFS

## O que vamos fazer

Na Aula 19, guardamos **pequenos valores** no NVS/Preferences. Mas e se
quisermos salvar **arquivos inteiros** — um texto longo, dados de um sensor
acumulados, ou mesmo uma **página web** pronta para ser servida ao navegador?

O ESP32 pode criar um **mini disco** dentro da sua Flash, chamado
**LittleFS**. Com ele, o ESP32 trabalha com **arquivos e pastas** como um
computador. Nesta aula vamos **criar um arquivo**, **escrever** nele,
**ler** de volta e **listar** o conteúdo — tudo no Monitor Serial.

Por que isso é útil? Arquivos dão **flexibilidade**: servir uma página HTML
sem montá-la no código, guardar registros (logs), ou armazenar configurações
grandes e organizadas.

## O que você vai aprender

No final desta aula, você vai entender:

- o **LittleFS** como um "mini disco" na Flash;
- como **montar** (inicializar) o LittleFS;
- como **abrir**, **escrever** e **ler** um arquivo;
- como **listar** arquivos/pastas;
- a diferença de uso entre **NVS** (pequenos) e **arquivos** (grandes);
- um exemplo prático com **página web**;
- cuidados com **espaço e limites**.

## O "mini disco" LittleFS

O LittleFS é um sistema de arquivos leve pensado para dispositivos pequenos.
Ele cria uma **estrutura de pastas e arquivos** dentro da Flash do ESP32.
Como se o ESP32 tivesse um **pendrive** embutido:

```
Flash
 └── LittleFS ("pendrive")
      ├── config.txt
      ├── logs/
      │    └── diario.txt
      └── pagina.html
```

Assim como no seu computador, você cria, lê, grava e apaga arquivos. O
conteúdo **continua** quando o ESP32 desliga (é Flash, Aula 18).

## Antes de ligar

- O **LittleFS** precisa de espaço reservado na partição de Flash. Algumas
  placas já vêm prontas; outras precisam de configuração. **Verifique a
  documentação da sua placa** para saber onde fica a pasta de "data" e como
  o LittleFS é habilitado. Este é um caso em que **nunca presumir** — cada
  ambiente difere.
- A maioria dos projetos usa o LittleFS habilitado por padrão no ESP32.
  Nesta primeira aula, vamos trabalhar apenas com **código** (criar/ler
  arquivos pelo programa), sem enviar arquivos externos ainda.

## Primeiro teste: criar, escrever e ler um arquivo

Vamos criar um arquivo `notas.txt`, escrever algumas linhas e ler de volta.
Depois, faremos também um contador permanente usando um arquivo.

```cpp
// Aula 20 — Criar, escrever e ler com LittleFS

#include <LittleFS.h>

void setup() {
  Serial.begin(115200);

  if (!LittleFS.begin(true)) {
    Serial.println("Falha ao montar o LittleFS.");
    return;
  }
  Serial.println("LittleFS montado!");

  escreverArquivo("/notas.txt", "Linha um\nLinha dois\nAula 20!");
  lerArquivo("/notas.txt");

  LittleFS.end();
}

void escreverArquivo(const char* caminho, const char* conteudo) {
  File arquivo = LittleFS.open(caminho, "w");
  if (!arquivo) {
    Serial.print("Erro ao abrir para escrita: ");
    Serial.println(caminho);
    return;
  }
  arquivo.print(conteudo);
  arquivo.close();
  Serial.print("Escrito: ");
  Serial.println(caminho);
}

void lerArquivo(const char* caminho) {
  File arquivo = LittleFS.open(caminho, "r");
  if (!arquivo) {
    Serial.print("Erro ao abrir para leitura: ");
    Serial.println(caminho);
    return;
  }
  Serial.println("--- Conteudo do arquivo ---");
  while (arquivo.available()) {
    Serial.write(arquivo.read());
  }
  Serial.println();
  arquivo.close();
}

void loop() {
}
```

### Entendendo o código

```cpp
#include <LittleFS.h>
```

Inclui a biblioteca **LittleFS**.

```cpp
if (!LittleFS.begin(true)) {
  Serial.println("Falha ao montar o LittleFS.");
  return;
}
```

**`LittleFS.begin(...)`** **monta o disco**. Se falhar (pouco espaço, placa
sem suporte), avisamos e **retornamos** (não seguimos). O argumento opcional
costuma permitir formatar automaticamente se necessário — confira o padrão da
sua instalação.

```cpp
File arquivo = LittleFS.open(caminho, "w");
```

**`LittleFS.open(caminho, modo)`** **abre** (ou **cria**) um arquivo. O modo
`"w"` = **write** (escrita, cria se não existir); `"r"` = **read** (leitura).
Devolve um objeto `File` — o arquivo aberto.

```cpp
if (!arquivo) {
```

Se `arquivo` for "falso", significa que **não abriu** (deu erro). Sempre
verificamos antes de usar.

```cpp
arquivo.print(conteudo);
```

**`arquivo.print(...)`** **escreve** o conteúdo no arquivo (como o
`Serial.print`, mas para o arquivo).

```cpp
arquivo.close();
```

**`arquivo.close()`** **fecha** o arquivo — importante para garantir que os
dados sejam salvos de verdade.

```cpp
while (arquivo.available()) {
  Serial.write(arquivo.read());
}
```

Para **ler**: enquanto houver dado disponível (`available()`), **lê um byte**
(`read()`) e **envia ao Serial** (`Serial.write`). Assim imprimimos o conteúdo
inteiro no Monitor.

## Um contador permanente em arquivo

Podemos guardar um número (ex.: quantas vezes ligou) em um **arquivo** — um
caminho diferente do NVS, mas igualmente permanente:

```cpp
// trecho: contador em arquivo (use com as funções acima)

void setup() {
  Serial.begin(115200);
  LittleFS.begin(true);

  int contador = 0;
  File arq = LittleFS.open("/contador.txt", "r");
  if (arq) {
    String texto = arq.readString();
    contador = texto.toInt();
    arq.close();
  }

  contador++;

  arq = LittleFS.open("/contador.txt", "w");
  if (arq) {
    arq.print(contador);
    arq.close();
  }

  Serial.print("Contador: ");
  Serial.println(contador);

  LittleFS.end();
}

void loop() {
}
```

### Entendendo

- **`arq.readString()`** lê **todo o texto** do arquivo como uma `String`;
- **`texto.toInt()`** converte a `String` em **número**;
- incrementamos, **reabrimos para escrita** e salvamos o novo valor.

O resultado é o mesmo "contador que sobrevive a desligar" da Aula 19 — mas
agora usando um **arquivo**. Note a diferença: aqui **guardamos um número em
um arquivo**, enquanto o NVS é mais simples para esse tipo de dado pequeno.
Escolha conforme a necessidade (veja abaixo).

## Listando arquivos e pastas

Para **listar** o que existe no "mini disco", podemos **abrir o diretório**:

```cpp
// trecho: listar arquivos

void listarArquivos(const char* pasta) {
  File dir = LittleFS.open(pasta);
  while (File f = dir.openNextFile()) {
    Serial.print(f.path());
    Serial.print("  (");
    Serial.print(f.size());
    Serial.println(" bytes)");
    f.close();
  }
}
```

**`LittleFS.open(pasta)`** abre o diretório; **`openNextFile()`** traz cada
arquivo seguinte; **`f.size()`** mostra o tamanho em bytes; **`f.path()`**
mostra o caminho. Assim você "anda" pelo disco.

## NVS ou LittleFS? Quando usar cada um

| Precisa... | Use |
|------------|-----|
| guardar **poucos valores** (config, contador) | **NVS/Preferences** (Aula 19) |
| guardar **arquivos grandes / textos / dados** | **LittleFS** (esta aula) |
| servir **página web / assets** | **LittleFS** (ótimo com Aulas 13–14) |

**Regra prática:** valores pequenos e frequentes → NVS; conteúdo grande,
organizado em arquivos → LittleFS.

## Cuidados com LittleFS

- **Espaço é limitado**: é uma partição da Flash. Não tente guardar mais do
  que o espaço reservado permite — verifique a ficha da placa.
- **Fechar os arquivos**: sempre `arquivo.close()` após usar (evita
  problemas de gravação).
- **Não gravar a cada instante**: a Flash se desgasta com muitas escritas.
  Grave quando necessário.
- **Partição**: habilitar/formatar o LittleFS pode depender de configuração
  da placa — verifique o ambiente (nada de presumir).

## Se não funcionar

- **"Falha ao montar"**: pode faltar espaço/partição para LittleFS na sua
  placa. Verifique a configuração de partição da sua placa.
- **"Erro ao abrir"**: confira o **caminho** (começa com `/`) e se o modo
  (`"w"`/`"r"`) está certo.
- **Arquivo aparece vazio**: não esqueceu do `arquivo.close()`? Os dados só
  são garantidos após fechar.
- **Não lista nada**: confira se o arquivo realmente foi criado (rode o
  primeiro exemplo completo) e o caminho da pasta.
- **Não compila `LittleFS`**: confira se a biblioteca está disponível na sua
  versão; em algumas, usa-se o nome da biblioteca do fabricante da placa.

## Experimente você

Agora, os desafios:

1. **Nome do usuário**: crie um arquivo `usuario.txt`, escreva seu nome e leia
   de volta no Serial.
2. **Registro (log)**: a cada `loop()` (com `millis()`, Aula 04, a cada 5
   segundos), **adicione** uma linha ao arquivo `log.txt` com a hora.
   (Pesquise `FILE_APPEND` / modo de anexar da sua versão para não apagar o
   que já tem.)
3. **Sirva uma página**: coloque um pequeno `pagina.html` no LittleFS e, no
   servidor web (Aulas 13–14), leia o arquivo e **entregue** o conteúdo ao
   navegador em vez de montar o HTML no código.

## O que aprendemos

Nesta aula:

- **LittleFS** é um "mini disco" na Flash do ESP32 (arquivos e pastas);
- usamos `LittleFS.begin`, `LittleFS.open`, `File.print`, `File.read`,
  `File.close`, `available`, `readString` e `toInt`;
- abrimos arquivos nos modos `"w"` (escrita) e `"r"` (leitura);
- sempre **fechamos** os arquivos e **verificamos** se abriram;
- **NVS** serve para valores pequenos; **LittleFS** para arquivos/conteúdo;
- lembramos dos limites de **espaço e gravação** da Flash.

## Próximo passo

Até aqui, toda a comunicação foi **sem fio via Wi‑Fi**. Mas existe outro
caminho sem fio que **não precisa de rede**: o **Bluetooth** — feito para
conectar o ESP32 **diretamente** ao celular ou a outros dispositivos, sem
roteador. Vamos conhecer isso na próxima aula.

Nos vemos na Aula 21.