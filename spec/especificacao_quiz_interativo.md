# Especificação: quiz interativo a partir de um documento

Use este documento como instrução fixa para transformar um arquivo (.docx, .pptx, .pdf) em um quiz interativo de página única (HTML + CSS + JS). Ele descreve o que já foi feito nos quizzes **Pensamento sequencial** e **Lógica de programação 1**.

## 1. Entrega

- **Um único arquivo `.html`**, com CSS e JS dentro dele, sem build e sem framework.
- Idioma da interface e dos textos: **português do Brasil**.
- Publicado como artefato hospedado (link do claude.ai). Atualizações pequenas são feitas editando o mesmo link, não criando um novo.
- Antes de gerar, ler o arquivo anexo por completo (texto e imagens).
- Nome do arquivo em `snake_case`, por exemplo `quiz_logica_programacao_1.html`.
- Favicon: um emoji que lembre o tema.

## 2. Restrições técnicas da página publicada

- Scripts externos só de `cdnjs.cloudflare.com`, `cdn.jsdelivr.net/npm/`, `cdn.tailwindcss.com` e `code.jquery.com`, com versão fixa. O padrão é **não usar nenhum**.
- Único CSS externo permitido: Google Fonts (`Nunito` 400/700/800), sempre com fallback `system-ui, sans-serif`.
- Sem imagens remotas e sem chamadas de rede. Emojis e CSS cobrem os ícones.
- Sem `<form>`. Usar `onclick`/`onchange` e `button`.
- Não depender de `localStorage`. O estado fica em variáveis JS.
- Meta viewport `width=device-width, initial-scale=1, viewport-fit=cover`, com `padding-top/bottom: env(safe-area-inset-*)` no `:root`.
- Texto digitado pelo usuário nunca é injetado como HTML sem escapar (`esc()`).
- Respeitar `prefers-reduced-motion` (animação de entrada do card só quando permitido).

## 3. Design

**Identidade:** limpa, didática, em cartão único centralizado (`max-width: 640px`), fonte Nunito, 17px, `line-height` 1.55.

**Tokens de cor (claro, padrão):**

| Token | Valor | Uso |
|---|---|---|
| `--bg` | `#f2f5fa` | fundo da página |
| `--card` | `#fff` | cartão e campos |
| `--ink` | `#162036` | texto |
| `--muted` | `#5a6781` | texto secundário |
| `--acc` | `#0e8f7e` | cor de destaque (botões, foco, progresso) |
| `--accink` | `#fff` | texto sobre o destaque |
| `--ok` / `--okbg` | `#17803d` / `#e3f6ea` | acerto |
| `--bad` / `--badbg` | `#c2371f` / `#fdeae6` | erro |
| `--line` | `#d5dcea` | bordas |

**Tokens no modo escuro** (`@media (prefers-color-scheme: dark)` com `:root:not([data-theme="light"])`, repetidos em `:root[data-theme="dark"]`): `--bg #0f1626`, `--card #18223a`, `--ink #eaf0fb`, `--muted #9fb0cf`, `--acc #38c9b4`, `--accink #06211c`, `--ok #4cd388`, `--okbg #12301f`, `--bad #ff8a73`, `--badbg #3a1d19`, `--line #2b3a5c`, com `color-scheme: dark`.

**Componentes fixos:**

- Cabeçalho: `h1` (1.5rem, peso 800) e subtítulo curto em `--muted`.
- **Trilha de progresso** (`.rail`): um círculo numerado por questão, ligados por uma linha. Concluída = preenchida com ✓; atual = borda de destaque com halo; futura = vazia.
- **Cartão** (`.card`): borda 1px, raio 14px, padding 20px, título `h2` (1.2rem) e enunciado.
- **Botão principal** com o texto "Conferir"; vira "Próxima" após o acerto e "Ver resultado" na última questão. Botão secundário (`.ghost`) com contorno.
- **Feedback** (`.fb`): caixa com barra lateral de 5px. Verde para acerto, vermelha para erro, neutra para informação.
- **Foco visível:** `outline: 3px solid var(--acc)` em campos e botões.
- **Chips informativos** (`.boxes`): quadrinhos com borda para valores de apoio, como "Caixa A = 10" ou "p: …".

## 4. Regras do jogo (valem para todos os quizzes)

1. **Só avança de fase quem acerta.** Resposta errada mostra "Ainda não", deixa tentar de novo e **não revela a resposta certa**.
2. **Explicação só depois do acerto.** Todo acerto mostra uma explicação curta de por que a resposta está certa.
3. **Contagem de tentativas:** cada clique em "Conferir" com resposta preenchida soma 1 tentativa na questão. Resposta vazia não conta e pede para preencher.
4. **Dica a partir da 3ª tentativa errada**, em um campo `h` por questão: orienta o raciocínio sem entregar a resposta.
5. **Tela de resultado:** total de tentativas em número grande, o mínimo possível (igual ao número de questões), "acertos de primeira: X/N", uma mensagem por faixa e a lista de revisão com as tentativas de cada questão (✓ se foi de primeira, ↻ se precisou repetir). Botão "Refazer o quiz" zera tudo.
6. **Alternativa errada em múltipla escolha:** fica marcada em vermelho e desmarcada. Quando a pessoa clica em **outra** alternativa, a marcação vermelha some e a mensagem de erro também. Nenhuma alternativa fica travada.
7. Ao errar, manter o que a pessoa já fez (ordem montada, palavras achadas, texto digitado selecionado para sobrescrever).
8. Enter confere (e avança, depois de acertar) nas questões de texto.

## 5. Tipos de questão

Todas ficam no array `Q`. Campos comuns: `k` (tipo), `title`, `text` (opcional), `exp` (explicação), `h` (dica).

| Tipo (`k`) | Campos próprios | Comportamento |
|---|---|---|
| `order` | `items` (lista na ordem certa) | A lista é embaralhada (nunca igual à ordem certa). Reordenar com ▲▼ (teclado e celular) ou arrastando. Ao errar, os itens fora do lugar ficam vermelhos e os certos, verdes. |
| `choice` | `opts` (4 alternativas, ou 2 em V/F), `a` (índice da correta), `fixed` (1 = não embaralhar), `defs` (opcional, chips com definições) | Botões de rádio. **Sempre 1 correta.** Usar `fixed` em V/F e P/NP, para a ordem não mudar. |
| `text` | `seq` (opcional, mostra a sequência com um `?`), `a` (lista de respostas aceitas, em minúsculas), `boxes` (opcional) | Campo de texto. Comparação sem diferença de maiúsculas, espaços, vírgula/ponto e traço longo/curto. |
| `words` | `words` (em maiúsculas, com acento) | Caça-palavras: toque na primeira e na última letra. Aceita horizontal, vertical e diagonal, nos dois sentidos. A palavra achada ganha uma cor de fundo e é riscada na lista. "Conferir" só acerta com todas achadas. |

**Chips de caixas** (`boxes`): `1` usa o conjunto padrão A=10, B=3, C=7. Uma lista `[["P",12],["Q",4]]` define um conjunto novo.

**Esquema mínimo de uma questão:**

```js
{k:"choice", title:"2) Reflexão: regar antes da terra",
 text:"O que acontece se você regar antes de encher o vaso com terra?",
 opts:["Resposta certa","Errada 1","Errada 2","Errada 3"], a:0,
 h:"dica opcional", exp:"Por que está certa, em 1 ou 2 frases."}
```

## 6. Do documento para as questões

1. **Mapear:** cada atividade do documento vira um grupo de questões, numerado na ordem em que aparece (`1)`, `2)`, …). O título de cada questão começa com o número do grupo.
2. **Contar:** apurar o total de questões antes de escrever. Se a pessoa pedir um total (por exemplo, 15), completar com questões do mesmo formato e atualizar o "mínimo possível de tentativas".
3. **Escolher o tipo:**
   - sequência de passos → `order`
   - pergunta "o que acontece se…" → `choice` com 4 alternativas
   - "complete a sequência" ou conta → `text`
   - classificar ou V/F → `choice` com `fixed`
   - caça-palavras → `words`
4. **Alternativas erradas** (distratores) devem ser plausíveis, curtas e de tamanho parecido com a certa. Uma delas pode ser "Nada muda, a ordem não importa" quando o tema é ordem.
5. **Explicações** em linguagem simples, mostrando o raciocínio (a conta, a regra do padrão), nunca só "porque sim".
6. **Corrigir sem mudar o sentido:** erros de digitação do documento (por exemplo "Proporsição") são corrigidos na tela e avisados na resposta.
7. **Ambiguidades:** quando o documento deixa duas leituras possíveis ou falta informação (por exemplo "Pedro é mais alto que o José"), escolher a mais defensável, acrescentar opção ou explicação que resolva e **dizer na resposta qual suposição foi feita**.
8. **Conferir os gabaritos** antes de publicar: refazer contas e padrões, e testar o caça-palavras procurando cada palavra na grade.
9. Exercício extra pedido "semelhante" a outro repete a estrutura (ordenar + 2 reflexões de múltipla escolha, ou o mesmo formato de conta), com tema novo e sem ambiguidade na ordem correta.

## 7. Acessibilidade

- `lang="pt-BR"`, `aria-live="polite"` no cartão, `aria-label` nos campos e nos botões ▲▼ ("Subir: …").
- Tudo funciona só com teclado. Rádios e botões são elementos nativos.
- Nunca depender só de cor: acerto e erro também têm texto ("Correto!", "Ainda não.") e os ícones ✓ e ↻ na revisão.
- Alvos de toque confortáveis no celular (células do caça-palavras com `aspect-ratio: 1`).

## 8. Processo de geração

1. Ler o documento anexo (texto, tabelas, imagens).
2. Montar o plano: grupos, tipos e total de questões, e as suposições a avisar.
3. Escrever o HTML único com CSS e JS.
4. Validar: sintaxe do JS, gabaritos e (se houver) grade do caça-palavras.
5. Publicar e responder de forma curta: link, estrutura do quiz e **só as suposições que o professor precisa conferir**.
6. Incluir o quiz novo no index (seção 9).
7. Mudanças seguintes (novas questões, ajustes de regra) são edições no mesmo link. Quando uma regra muda, aplicar também aos quizzes já publicados que usam a mesma base, se a pessoa pedir.

## 9. Página index dos quizzes

Todo quiz novo entra em uma **página index** única, que abre os quizzes por categoria. Link atual: https://claude.ai/artifact/AEMwyM2ZJkSvHgCR3etJcu

- **Arquivo:** `index_quizzes.html`, único, com os mesmos tokens de cor, fonte e modo escuro do quiz (seção 3).
- **Estrutura:** título "Quizzes", subtítulo curto e uma seção por categoria (nome + quantidade de quizzes). Cada quiz é um cartão-link numerado na sequência, com título, descrição curta (assunto e número de questões) e uma seta →.
- **Dados em um só lugar:** a página é montada a partir da lista `CATS` no início do `<script>`:

```js
const CATS=[
{nome:"Lógica de programação",quizzes:[
 {titulo:"Pensamento sequencial",desc:"Resumo curto · 15 questões",url:"https://claude.ai/artifact/..."},
 {titulo:"Lógica de programação 1",desc:"Resumo curto · 15 questões",url:"https://claude.ai/artifact/..."}
]}
];
```

- **Quiz novo na mesma categoria:** acrescentar um item em `quizzes`. **Categoria nova:** acrescentar um objeto em `CATS`. Nada mais muda.
- Os links abrem em outra aba (`target="_blank" rel="noopener"`).
- **Depois de publicar um quiz novo:** atualizar o index com uma edição no mesmo link (`edit`), sem criar outro index, e informar o link do index na resposta.

## 10. Histórico de decisões já tomadas

- Reflexão em texto livre foi trocada por **2 questões de múltipla escolha com 4 alternativas e 1 correta**.
- Questões numéricas "caixas" usam um conjunto de valores por grupo (A/B/C; P/Q/R).
- "Hoje está frio." foi classificada como **proposição** (afirma algo que pode ser julgado, o valor depende do dia).
- "Pedro é mais alto que o José." tem a terceira opção "Não dá para saber com os dados do enunciado" como correta.
- Quizzes publicados: **Pensamento sequencial** (15 questões) e **Lógica de programação 1** (15 questões).
- **Index** publicado com a categoria "Lógica de programação" (os dois quizzes acima). Novas categorias e quizzes entram pela lista `CATS`.
