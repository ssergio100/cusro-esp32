# Aula 00 — Preparando a bancada

## O que vamos fazer

Nesta aula não vamos ligar nada ainda. Vamos **conhecer nossa ferramenta
principal**: a placa ESP32 e o programa que usamos para falar com ela.

Isso é o que todo mundo precisa fazer antes de dar os primeiros passos: saber
*o que estamos segurando na mão* e *como conversamos com ela*.

Por que isso é útil? Porque, se você souber reconhecer as partes da placa e
instalar a ferramenta certa desde o começo, vai economizar muita dor de
cabeça depois. A maioria dos problemas no curso não é erro de código — é
instalação, porta errada ou ligação errada.

## O que você vai aprender

Nesta aula, apenas alguns conceitos:

- o que é um microcontrolador (e como ele difere de um computador);
- as partes importantes da placa ESP32;
- o que são os pinos GPIO;
- como instalar e configurar o **Arduino IDE**;
- como ligar a placa e ver a primeira mensagem dela no **Monitor Serial**.

Não vamos programar nada de verdade ainda. Isso é a Aula 01.

## Material necessário

Para esta aula você vai precisar de:

- Uma **placa ESP32** (qualquer variante comum serve para começar);
- um **cabo USB** — de preferência o mesmo que carrega celular, com ponta
  USB-padrão no computador e uma ponta menor (micro USB ou USB-C) na placa;
- um **computador com Linux/Ubuntu**;
- conexão com a **internet** (apenas para baixar o programa).

Não precisa de nenhum componente eletrônico extra agora. Só a placa e o
cabo.

> 📷 Se possível, tenha uma foto da sua placa em mãos. Cada variante de
> ESP32 é um pouco diferente, e você vai querer olhar para ela enquanto
> lemos os nomes das partes.

## Antes de ligar

Leia isto antes de conectar qualquer coisa:

- **Não conecte a placa a nenhuma fonte de energia além do cabo USB por
  enquanto.** Nesta fase, ela só precisa de energia vinda do computador.
- **Não force o conector USB.** As portas das placas são delicadas. Se não
  encaixar de primeira, vire o cabo — não empurre com força.
- Se a placa **esquentar muito** (a ponto de incomodar ao toque), tire o
  cabo imediatamente. Uma placa saudável aquece pouco.

## Ligação

Desta vez, "ligação" é apenas **conectar o cabo USB**:

```
[Computador]  —USB—>  [Placa ESP32]
```

Uma ponta do cabo vai no computador, a outra na placa. Só isso. A placa deve
acender uma **pequena luz** (normalmente azul ou vermelha) quando receber
energia do computador.

> 💡 Essa luz que acende ao ligar costuma ser chamada de **LED de
> alimentação**. Ela só indica que a placa recebeu eletricidade — não é a
> mesma luz que vamos programar depois.

## Primeiro teste

O primeiro teste é o mais simples possível: **fazer a placa acender o LED
de energia ao conectar o cabo**.

Não há código neste teste. Se a luz acender, a placa está recebendo energia
e está pronta para o próximo passo.

## Código

Não há código nesta aula. Ainda. 😉

Na verdade, a única "configuração" que faremos agora é **instalar a
ferramenta** que vamos usar para escrever e enviar programas à placa: o
**Arduino IDE**.

Vamos instalar juntos. No Linux/Ubuntu:

1. Abra o navegador e procure por **"Arduino IDE download"** (o site
   oficial é `arduino.cc`).
2. Baixe a versão para **Linux** (formato `.zip` ou `.AppImage`).
3. Extraia o arquivo baixado em uma pasta de sua preferência.
4. Abra o programa. Se for um `.AppImage`, pode ser preciso dar permissão de
   execução a ele (clicar com o botão direito > Propriedades > Permitir
   execução) ou rodar o comando `chmod +x` no terminal.

Depois de abrir o Arduino IDE, precisamos dizer a ele **qual placa vamos
usar**. O ESP32 não vem configurado por padrão — precisamos adicioná-lo uma
única vez:

1. No menu, procure por **"Gerenciador de placas"** (ou *Boards Manager*).
2. Pesquise por **"esp32"**.
3. Instale o pacote oficial **"esp32 by Espressif Systems"**.
4. Aguarde a instalação terminar (o download é grande na primeira vez).

Pronto. Agora o Arduino IDE "conhece" a nossa placa.

## Entendendo o código

Como não escrevemos código, vamos entender o que é cada coisa da placa —
isso vale ouro antes de começar a programar.

### O que é um microcontrolador?

Imagine um computador comum (seu notebook, por exemplo). Ele tem muita
potência, mas também é grande, gasta muita energia e precisa de um sistema
operacional complexo.

O **microcontrolador** é como um **computador minúsculo**, feito para fazer
*poucas tarefas* muito bem, gastando quase nada de energia. Ele é um único
"chipsinho" que contém o cérebro (processador), a memória e as formas de
falar com o mundo externo.

O ESP32 é um microcontrolador muito popular. Ele é barato, tem Wi-Fi e
Bluetooth embutidos — é por isso que ele é ótimo para "coisas conectadas",
como luzes inteligentes, sensores caseiros e pequenos dispositivos IoT.

### Microcontrolador versus computador

Uma tabela simples ajuda:

| | Computador | Microcontrolador (ESP32) |
|---|---|---|
| Tamanho | Grande | Minúsculo |
| Energia | Muita | Pouquíssima |
| Sistema operacional | Sim (Windows, Linux...) | Mínimo ou nenhum |
| Tarefa | Muitas ao mesmo tempo | Poucas, específicas |
| Custo | Alto | Barato |

A ideia central: você **programa** o microcontrolador para fazer uma tarefa
específica, e ele fica ali fazendo aquilo, sozinho, por muito tempo.

### As partes importantes da placa ESP32

Olhando para sua placa, tente encontrar:

- **O chip ESP32**: um "quadradinho" preto, geralmente com uma antena
  metálica ao lado (para o Wi-Fi).
- **A porta USB**: por onde ligamos o cabo. Ela alimenta a placa e também é
  o "cano" por onde enviamos programas e recebemos mensagens.
- **Os pinos (GPIO)**: fileiras de furos/terminais metálicos nas laterais.
  É por eles que conectamos LEDs, botões, sensores e outros componentes.
- **O botão EN (ou RST)**: reinicia a placa, como "desligar e ligar".
- **O botão BOOT**: usado em situações especiais (vamos falar sobre ele na
  Aula 01, se precisarmos).

### O que são GPIOs?

**GPIO** é uma sigla em inglês: *General Purpose Input/Output* —
"entrada e saída de uso geral".

Pense nos GPIOs como os "braços" da placa. Cada pino GPIO pode fazer dois
tipos básicos de coisa:

- **Saída**: a placa *manda* um sinal para fora — por exemplo, acender um
  LED.
- **Entrada**: a placa *escuta* um sinal que vem de fora — por exemplo, saber
  se um botão foi pressionado.

Não precisa decorar isso agora. Apenas entenda: os GPIOs são a forma de a
placa conversar com o mundo físico.

> ⚠️ Os GPIOs do ESP32 trabalham com **3,3 volts** — mais sobre isso na
> Aula 02. Por enquanto, a regra de ouro: **não conecte nada que não saiba
> de onde veio**. Sempre ao lado de cada aula haverá um aviso quando for
> seguro ligar algo.

### O Arduino IDE

O **Arduino IDE** é um programa (uma "interface de desenvolvimento") que
usamos para:

1. **escrever** o programa (o "código" — as instruções para a placa);
2. **compilar** (traduzir o que escrevemos para uma linguagem que o chip
   entende);
3. **enviar** esse programa para a placa pelo cabo USB;
4. **conversar** com a placa (receber mensagens dela).

Você pode pensar nele como um "editor de texto inteligente" que também sabe
entregar suas instruções à placa.

### Selecionar a placa e a porta

Antes de poder enviar um programa, o Arduino IDE precisa saber **qual**
placa você está usando e **onde** ela está conectada:

- **Placa**: no menu de placas, escolha o modelo do seu ESP32. Se não tiver
  certeza, "ESP32 Dev Module" é uma escolha segura para a maioria das placas
  comuns.
- **Porta**: é o nome que o computador dá ao cabo conectado. No Linux,
  costuma aparecer como algo parecido com `/dev/ttyUSB0` ou `/dev/ttyACM0`.
  Selecione a porta que corresponde à sua placa.

Se a porta não aparecer, verifique se o cabo **transmite dados** (alguns
cabos só carregam energia) e se o ESP32 está instalado corretamente. Esse é
o erro mais comum de quem está começando, e vamos voltar a ele abaixo.

### Monitor Serial

O **Monitor Serial** é uma janela do Arduino IDE que mostra as mensagens que
a placa envia para o computador. É como uma **linha de conversa** entre você
e a placa: ela "fala" e você vê.

Vamos usá-lo muito ao longo do curso. Ele é uma ferramenta de diagnóstico
essencial: quando algo não funciona, é ele que costuma nos dizer o motivo.

Para abri-lo: no Arduino IDE, procure pelo ícone de **lupa** (ou "Monitor
Serial" no menu). Garanta que a **porta** selecionada seja a da sua placa.

No final desta aula, faremos nosso primeiro "oi" para a placa por aqui.

## O que deve acontecer

Ao final desta aula, você deve ter:

- a placa ESP32 conectada pelo cabo USB, com o LED de alimentação aceso;
- o **Arduino IDE** instalado e rodando;
- o pacote **esp32** adicionado ao Gerenciador de placas;
- a **placa** e a **porta** corretas selecionadas;
- o **Monitor Serial** aberto.

Se tudo isso estiver pronto, você está com a bancada preparada para o curso.

## Se não funcionar

- **A placa não acende**: confira o cabo. Muitos cabos só carregam e não
  transmitem dados. Teste com outro cabo, de preferência um que você sabe
  que funciona com celular para transferir arquivos.
- **A porta não aparece**: verifique se o pacote "esp32" foi instalado e se
  o cabo transmite dados. Reinsira o cabo e espere alguns segundos. Às vezes
  o comando `ls /dev/tty*` no terminal mostra a porta disponível.
- **O Monitor Serial fica vazio ou com erro**: confirme que selecionou a
  porta certa. Feche e reabra o Monitor Serial. Em algumas placas, é preciso
  apertar o botão **EN** (reset) para reiniciá-la e ver mensagens novas.
- **O Arduino IDE não "vê" a placa**: verifique as permissões do sistema. Em
  alguns Linux, é preciso adicionar o usuário ao grupo que acessa portas
  seriais (procure seu grupo específico — costuma ser `dialout` ou
  `uucp`). Se preferir, use um guia do seu sistema.

## Experimente você

Agora vamos fazer de verdade a nossa primeira conversa com a placa. Não é
um programa da nossa autoria — é um exemplo que já vem pronto — mas é o
primeiro passo para provar que tudo está funcionando.

1. No Arduino IDE, abra o exemplo **Blink** (menu *Arquivo > Exemplos > 01.
   Basics > Blink*).
2. Garanta que **placa** e **porta** estão corretas.
3. Clique no botão de **enviar** (a seta para a direita, ou *Upload*).
4. Observe: o LED da placa deve **piscar** em um ritmo constante.

Se o "Blink" funcionar, sua ferramenta está pronta — e você já rodou seu
primeiro programa no ESP32, mesmo sem entender cada detalhe. Nada disso se
perde: na Aula 01 vamos destrinchar esse programa linha por linha.

## O que aprendemos

Nesta aula preparamos a base:

- entendemos que o ESP32 é um **microcontrolador** — um computador minúsculo
  para tarefas específicas, com Wi-Fi e Bluetooth embutidos;
- conhecemos as **partes da placa**: chip, porta USB, GPIOs, e botões EN e
  BOOT;
- entendemos que os **GPIOs** são a forma de a placa conversar com o mundo
  (entrada e saída);
- instalamos e configuramos o **Arduino IDE** com o pacote do ESP32;
- aprendemos a selecionar **placa e porta**, e a abrir o **Monitor Serial**;
- rodamos nosso primeiro exemplo, o **Blink**.

## Próximo passo

Agora que a bancada está pronta, vamos finalmente escrever nosso **primeiro
programa do zero** — e entender, linha por linha, o que cada parte faz.

Nos vemos na [Aula 01](../01-primeiro-programa/README.md). 🚀
