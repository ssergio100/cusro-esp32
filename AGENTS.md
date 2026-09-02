# AGENTS.md — Regras do projeto

Este arquivo documenta as regras pedagógicas e técnicas deste curso para
futuros agentes de IA. Antes de escrever ou modificar qualquer conteúdo,
leia este arquivo e siga as regras à risca.

## O que é este projeto

Um curso didático de ESP32 para pessoas leigas, sem conhecimento prévio de
eletrônica, microcontroladores ou programação. A prioridade é o
**entendimento**, não a quantidade de conteúdo.

## Regras pedagógicas (obrigatórias)

1. Uma ideia nova por vez.
2. Passos curtos.
3. Sempre explicar primeiro "o que vamos fazer" e "por que isso é útil".
4. Evitar jargão técnico sem explicar.
5. Ao introduzir um termo novo, explicar em linguagem comum.
6. Todo código deve ser explicado.
7. Não entregar programas enormes sem necessidade.
8. Cada aula deve produzir algo visível ou verificável.
9. Mostrar erros comuns e como reconhecê-los.
10. Nunca presumir que o aluno sabe eletrônica ou C++.
11. Usar analogias simples quando ajudarem.
12. Preferir diagramas ASCII simples para ligações elétricas.
13. Ensinar primeiro o necessário e aprofundar depois.
14. Não transformar o curso em uma apostila acadêmica.
15. Escrever como alguém experiente ensinando outra pessoa na bancada.

## Estrutura obrigatória de cada aula

Toda aula deve seguir esta estrutura de seções, na ordem:

- `# Título`
- `## O que vamos fazer` — explicação curta.
- `## O que você vai aprender` — no máximo alguns conceitos.
- `## Material necessário`
- `## Antes de ligar` — alertas de alimentação, tensão ou ligação.
- `## Ligação` — diagrama ASCII simples.
- `## Primeiro teste` — o menor teste possível.
- `## Código` — código curto e funcional.
- `## Entendendo o código` — explicação em linguagem simples.
- `## O que deve acontecer` — resultado esperado.
- `## Se não funcionar` — checklist curto de diagnóstico.
- `## Experimente você` — um exercício pequeno.
- `## O que aprendemos` — resumo curto.
- `## Próximo passo` — uma frase preparando a próxima aula.

## Plataforma

- ESP32 (use ESP32-S3 apenas quando algum recurso específico exigir).
- Arduino IDE e framework Arduino / C++.
- Linux/Ubuntu para ferramentas de desenvolvimento.
- Sempre que possível, o conteúdo deve continuar útil para outras variantes
  de ESP32.

## Regras técnicas de escrita

- Pular de 3,3 V para 5 V ou vice-versa exige aviso explícito em
  `## Antes de ligar`.
- Nunca presumir nomes de pinos, GPIOs ou bibliotecas — sempre verificar a
  placa específica quando relevante.
- Código deve ser curto, comentado apenas onde ajuda, e sempre seguido de
  explicação.
- Diagramas ASCII devem vir entre blocos de código (delimitados por ```).

## Versionamento (Git)

1. Verifique o estado atual do repositório antes de modificar arquivos.
2. Não apague trabalho existente sem autorização.
3. Não reescreva histórico; não faça force push.
4. Nunca coloque senhas, SSIDs reais, tokens ou chaves de API no repositório.
5. Use commits pequenos e compreensíveis, com mensagens no estilo:

   - `docs: cria estrutura inicial do curso`
   - `docs: adiciona aula 00 preparação da bancada`
   - `docs: adiciona exercício sobre GPIO`

6. Não faça um único commit gigantesco contendo o curso inteiro.
