# Instruções para o Big Pickle — Capítulos 6 a 23

Curso: **ESP32 do Zero — Eletrônica e Programação para Pessoas Comuns**

Estas instruções seguem a mesma filosofia dos capítulos anteriores:

- linguagem extremamente simples;
- passos curtos;
- uma ideia nova por vez;
- não assumir conhecimento prévio;
- exemplos pequenos e funcionais;
- diagramas ASCII quando ajudarem;
- explicar cada comando novo;
- sempre terminar com algo visível, mensurável ou verificável;
- não alterar capítulos anteriores sem autorização;
- fazer um commit Git pequeno e específico ao final de cada capítulo;
- parar e aguardar autorização antes de criar o capítulo seguinte.

---

## Capítulo 6 — Lendo sinais analógicos

### OBJETIVO
Ensinar um iniciante a entender que alguns sinais podem variar entre 0 V e 3,3 V e que o ESP32 consegue transformar essa tensão em um número usando o ADC.

### MARCOS
1. Relembrar digital: HIGH ou LOW.
2. Mostrar que um potenciômetro pode gerar valores intermediários.
3. Explicar ADC como um “medidor de tensão que devolve um número”.
4. Ligar potenciômetro em 3V3, GND e um GPIO adequado.
5. Ler com `analogRead()`.
6. Mostrar valores no Monitor Serial.
7. Girar o potenciômetro e observar a mudança.
8. Explicar que pequenas oscilações são normais.
9. Não aplicar mais que 3,3 V no GPIO.
10. Pequeno desafio: usar a leitura para controlar brilho PWM.

### REGRAS
- Não aprofundar resolução, atenuação ou calibração nesta primeira aula.
- Não entrar em fórmulas complexas.
- Não prometer precisão de multímetro.
- Usar um GPIO adequado para ADC na placa escolhida.

---

## Capítulo 7 — Monitor Serial como ferramenta de diagnóstico

### OBJETIVO
Ensinar o aluno a usar o Monitor Serial como uma janela para enxergar o que está acontecendo dentro do programa.

### MARCOS
1. Explicar que o ESP32 não tem tela própria para mostrar o que pensa.
2. Introduzir `Serial.begin(115200)`.
3. Mostrar `Serial.println("Olá")`.
4. Exibir valor de botão.
5. Exibir leitura analógica.
6. Mostrar mensagens como “entrei aqui”, “Wi‑Fi conectado”, “erro”.
7. Explicar que diagnóstico é uma ferramenta normal de programação.
8. Mostrar que excesso de mensagens também pode atrapalhar.
9. Pequeno exercício de depuração.

### REGRAS
- Não entrar em níveis de log avançados.
- Não transformar a aula em depuração profissional.
- Usar exemplos dos capítulos anteriores.

---

## Capítulo 8 — I2C sem mistério

### OBJETIVO
Ensinar o conceito de barramento I2C com SDA, SCL e endereços.

### MARCOS
1. Explicar “vários dispositivos na mesma linha”.
2. Mostrar SDA e SCL.
3. Explicar endereço como “número da casa”.
4. Mostrar ligação com 3V3 e GND.
5. Explicar GND comum.
6. Criar scanner I2C.
7. Mostrar resultado como `0x3C`.
8. Explicar apenas o necessário sobre hexadecimal.
9. Mostrar o que acontece quando nenhum dispositivo é encontrado.
10. Pequeno desafio: testar outro periférico.

### REGRAS
- Não aprofundar ACK/NACK, clock stretching ou protocolo elétrico.
- Não exigir conhecimento de hexadecimal.
- Explicar que alguns módulos já têm resistores de pull-up.

---

## Capítulo 9 — Usando um dispositivo I2C

### OBJETIVO
Usar um periférico I2C real para transformar o conceito do capítulo anterior em algo útil.

### SUGESTÃO
Escolher apenas um dispositivo principal, por exemplo OLED SSD1306, BME280 ou DS3231.

### MARCOS
1. Identificar endereço com scanner.
2. Instalar biblioteca.
3. Explicar o que é uma biblioteca.
4. Inicializar o dispositivo.
5. Ler ou escrever uma informação simples.
6. Mostrar resultado.
7. Explicar que a biblioteca esconde detalhes do protocolo.
8. Mostrar erro comum: endereço errado.
9. Pequeno desafio.

### REGRAS
- Não usar mais de um periférico no exemplo principal.
- Não aprofundar a biblioteca internamente.
- Manter código curto.

---

## Capítulo 10 — SPI

### OBJETIVO
Introduzir SPI e mostrar por que ele é usado em displays e periféricos rápidos.

### MARCOS
1. Relembrar I2C.
2. Mostrar que SPI usa mais fios.
3. Explicar MOSI, MISO, SCLK e CS em linguagem humana.
4. Mostrar que alguns dispositivos não usam MISO.
5. Explicar CS como “qual dispositivo estou chamando agora”.
6. Mostrar barramento compartilhado.
7. Não exigir memorização de siglas.
8. Preparar o aluno para o ST7789.

### REGRAS
- Não aprofundar modos SPI, CPOL, CPHA ou registradores.
- Foco no entendimento visual.

---

## Capítulo 11 — Display gráfico ST7789

### OBJETIVO
Fazer o aluno ligar um ST7789 e mostrar texto, formas e uma imagem simples.

### MARCOS
1. Apresentar o display e seu controlador.
2. Identificar resolução do módulo.
3. Ligar SPI.
4. Explicar alimentação e backlight.
5. Instalar biblioteca adequada.
6. Inicializar display.
7. Limpar tela.
8. Mostrar texto.
9. Desenhar uma forma simples.
10. Mostrar uma imagem simples.
11. Explicar orientação/rotação.
12. Diagnosticar tela branca ou preta.

### REGRAS
- Verificar a resolução do módulo.
- Não assumir que todo ST7789 tem os mesmos pinos.
- Não entrar ainda em sprites, DMA ou otimização.

---

## Capítulo 12 — Wi‑Fi

### OBJETIVO
Conectar o ESP32 a uma rede Wi‑Fi e mostrar claramente o que acontece durante a conexão.

### MARCOS
1. Explicar SSID e senha.
2. Mostrar `WiFi.begin()`.
3. Acompanhar conexão pelo Monitor Serial.
4. Mostrar `WiFi.status()`.
5. Mostrar IP local recebido.
6. Explicar IP de forma simples.
7. Mostrar reconexão básica.
8. Alertar para não publicar senhas no Git.
9. Usar placeholders ou arquivo separado para credenciais.
10. Pequeno desafio: mostrar intensidade do sinal.

### REGRAS
- Nunca colocar credenciais reais no repositório.
- Não entrar ainda em redes corporativas ou certificados.

---

## Capítulo 13 — Primeiro servidor Web

### OBJETIVO
Fazer o ESP32 servir uma página simples acessível pelo navegador e controlar um LED.

### MARCOS
1. Explicar cliente e servidor.
2. Explicar HTTP em uma frase simples.
3. Iniciar servidor.
4. Criar rota `/`.
5. Mostrar HTML simples.
6. Criar botão no navegador.
7. Criar rota para ligar/desligar LED.
8. Mostrar fluxo navegador → ESP32 → GPIO.
9. Exibir IP no Monitor Serial.
10. Diagnosticar acesso pela rede local.

### REGRAS
- HTML mínimo.
- Não entrar ainda em autenticação ou internet pública.
- Não expor o ESP32 diretamente à internet.

---

## Capítulo 14 — ESP32 conversando com servidores

### OBJETIVO
Ensinar o ESP32 a fazer uma requisição HTTP para uma API.

### MARCOS
1. Explicar que agora o ESP32 será cliente.
2. Fazer HTTP GET simples.
3. Mostrar código de resposta.
4. Ler resposta como texto.
5. Introduzir JSON de forma visual.
6. Extrair um ou dois campos.
7. Mostrar erros de conexão.
8. Explicar timeout.
9. Pequeno desafio.

### REGRAS
- API simples e pública ou exemplo local.
- Não usar chave secreta no repositório.
- Não aprofundar REST.

---

## Capítulo 15 — MQTT

### OBJETIVO
Ensinar publish/subscribe de forma simples e prática.

### MARCOS
1. Explicar broker como “central de mensagens”.
2. Explicar tópico.
3. Publisher.
4. Subscriber.
5. Conectar ESP32 ao broker.
6. Publicar uma mensagem.
7. Receber uma mensagem.
8. Controlar LED via tópico.
9. Publicar estado de botão ou sensor.
10. Mostrar fluxo completo.

### REGRAS
- Um broker apenas.
- Poucos tópicos.
- Não aprofundar QoS, retained ou Last Will nesta primeira aula.

---

## Capítulo 16 — Interrupções

### OBJETIVO
Explicar que algumas coisas precisam chamar a atenção do ESP32 imediatamente, sem ficar sendo verificadas o tempo todo.

### MARCOS
1. Relembrar polling com `digitalRead()`.
2. Explicar interrupção como “campainha”.
3. Criar exemplo com botão.
4. Mostrar ISR muito curta.
5. Explicar por que não colocar código pesado dentro da interrupção.
6. Usar uma flag para avisar o `loop()`.
7. Mostrar `volatile` apenas no nível necessário.
8. Explicar quando usar e quando não usar.

### REGRAS
- Não usar `delay()` dentro da ISR.
- Não fazer Serial pesado dentro da ISR.
- Não aprofundar prioridades e detalhes internos.

---

## Capítulo 17 — Timers

### OBJETIVO
Diferenciar `millis()`, timers e interrupções de timer.

### MARCOS
1. Relembrar `millis()`.
2. Explicar timer como “relógio interno”.
3. Mostrar caso simples de execução periódica.
4. Comparar com `millis()`.
5. Mostrar que timer pode gerar interrupção.
6. Usar exemplo pequeno.
7. Explicar quando `millis()` já é suficiente.

### REGRAS
- Não transformar a aula em registradores de hardware.
- Usar API atual do Arduino-ESP32.
- Explicar diferenças entre versões apenas se necessário.

---

## Capítulo 18 — Memória: RAM, Flash e PSRAM

### OBJETIVO
Dar ao iniciante um modelo mental simples de onde o programa e os dados ficam.

### MARCOS
1. Flash = onde o programa fica guardado.
2. RAM = mesa de trabalho.
3. PSRAM = mesa extra, quando disponível.
4. Mostrar que variáveis ocupam memória.
5. Mostrar que imagens grandes ocupam muito espaço.
6. Mostrar tamanho do programa após compilação.
7. Explicar que nem todo ESP32 tem PSRAM.
8. Relacionar com displays e imagens.

### REGRAS
- Não aprofundar fragmentação de heap.
- Não entrar em linker script.
- Usar analogias simples.

---

## Capítulo 19 — Salvando configurações com Preferences/NVS

### OBJETIVO
Ensinar o ESP32 a lembrar configurações mesmo depois de desligado.

### MARCOS
1. Mostrar que variáveis normais somem ao desligar.
2. Explicar NVS como pequena área de armazenamento.
3. Abrir namespace.
4. Salvar valor.
5. Ler valor.
6. Alterar valor e reiniciar.
7. Mostrar persistência.
8. Exemplo: brilho, nome ou opção do usuário.

### REGRAS
- Não usar NVS para gravações contínuas em alta frequência.
- Explicar desgaste da flash de maneira breve.

---

## Capítulo 20 — Arquivos com LittleFS

### OBJETIVO
Ensinar a salvar e ler arquivos na flash do ESP32.

### MARCOS
1. Explicar sistema de arquivos.
2. Inicializar LittleFS.
3. Criar arquivo.
4. Escrever texto.
5. Ler texto.
6. Listar arquivos.
7. Mostrar uso para HTML, configurações e imagens.
8. Introduzir organização da pasta de dados.
9. Explicar diferença conceitual entre NVS e arquivo.

### REGRAS
- Não aprofundar partições.
- Usar arquivos pequenos.
- Mostrar tratamento básico de erro.

---

## Capítulo 21 — Bluetooth e BLE

### OBJETIVO
Dar uma visão prática de Bluetooth no ESP32, com foco em BLE.

### MARCOS
1. Explicar Bluetooth clássico x BLE.
2. Explicar que nem todos os ESP32 suportam os mesmos modos.
3. Explicar dispositivo, serviço e característica em linguagem simples.
4. Criar exemplo BLE mínimo.
5. Fazer o celular encontrar o ESP32.
6. Ler ou escrever um valor simples.
7. Mostrar possibilidade de controle local sem Wi‑Fi.

### REGRAS
- Verificar suporte da variante usada.
- Não aprofundar GATT em excesso.
- Não usar exemplos gigantes de biblioteca.

---

## Capítulo 22 — Áudio digital com I2S

### OBJETIVO
Introduzir áudio digital e mostrar como o ESP32 envia dados para um DAC ou amplificador I2S.

### MARCOS
1. Explicar áudio analógico x digital.
2. Apresentar I2S.
3. Explicar BCLK, LRCLK/WS e DATA.
4. Mostrar ligação com módulo simples, como MAX98357A.
5. Gerar um tom simples ou reproduzir um pequeno áudio.
6. Explicar sample rate apenas superficialmente.
7. Mostrar o papel do amplificador e do alto-falante.
8. Diagnosticar ausência de som.

### REGRAS
- Não aprofundar DSP.
- Não usar áudio complexo no primeiro exemplo.
- Cuidado com alimentação e ligação do alto-falante.

---

## Capítulo 23 — Projeto integrado

### OBJETIVO
Fazer o aluno juntar vários conceitos do curso em um único projeto, sem introduzir uma grande quantidade de conceitos novos.

### PROJETO SUGERIDO
Criar um pequeno painel conectado usando:

- ESP32;
- display ST7789;
- um botão;
- Wi‑Fi;
- NTP ou API;
- Preferences ou LittleFS;
- página web simples ou MQTT.

### MARCOS
1. Definir o que o projeto fará.
2. Desenhar blocos do sistema antes do código.
3. Testar cada parte separadamente.
4. Ligar display.
5. Testar botão.
6. Conectar Wi‑Fi.
7. Buscar informação.
8. Salvar uma configuração.
9. Integrar tudo em passos pequenos.
10. Adicionar diagnóstico Serial.
11. Testar falhas: sem Wi‑Fi, servidor indisponível e reinicialização.
12. Organizar o código.
13. Revisar segurança básica.
14. Fazer versão final.

### OBJETIVO PEDAGÓGICO
O aluno deve perceber que um projeto grande é apenas a soma de várias partes pequenas que ele já aprendeu.

### REGRAS
- Não entregar um programa enorme logo no início.
- Construir por etapas.
- Cada etapa deve funcionar antes da próxima.
- Reutilizar conceitos dos capítulos anteriores.
- Explicar organização em funções simples.
- Não introduzir arquitetura avançada desnecessariamente.

---

# Instrução final para todos os capítulos

Para cada capítulo:

1. Crie somente o capítulo solicitado.
2. Não altere capítulos anteriores sem autorização.
3. Revise a linguagem para um iniciante completo.
4. Confirme que o exemplo funciona com a versão atual do Arduino-ESP32.
5. Mostre quais arquivos foram alterados.
6. Faça um commit Git pequeno e específico.
7. Pare e aguarde autorização antes de continuar.

Mensagens de commit sugeridas:

```text
docs: adiciona capitulo 06 leitura analogica
docs: adiciona capitulo 07 monitor serial
docs: adiciona capitulo 08 introducao ao i2c
docs: adiciona capitulo 09 periferico i2c
docs: adiciona capitulo 10 introducao ao spi
docs: adiciona capitulo 11 display st7789
docs: adiciona capitulo 12 wifi
docs: adiciona capitulo 13 servidor web
docs: adiciona capitulo 14 cliente http
docs: adiciona capitulo 15 mqtt
docs: adiciona capitulo 16 interrupcoes
docs: adiciona capitulo 17 timers
docs: adiciona capitulo 18 memoria
docs: adiciona capitulo 19 preferences nvs
docs: adiciona capitulo 20 littlefs
docs: adiciona capitulo 21 bluetooth ble
docs: adiciona capitulo 22 audio i2s
docs: adiciona capitulo 23 projeto integrado
```
