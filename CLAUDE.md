# Receita Fácil — cartão de remédios para pacientes analfabetos

**Nome da ferramenta (2026-09-09, 4ª e última rodada de nome no mesmo dia):
"Receita Fácil".** Nome de arquivo/repo (`cartao-remedios`,
`cartao_remedios_editavel.html`) não mudou — só o nome visível pro usuário
(título da aba, cabeçalho do painel). Histórico completo do dia: "Gerador
de cartão de remédios" → "Receita Visual" → "Receita Acessível" → "SOAdesão"
(trocadilho "SOA" + adesão ao tratamento — termo clínico real, pensado pra
incorporar como sub-ferramenta do soaperando, no mesmo padrão do próprio
"SOAPerando": pun que não se auto-explica sozinho, funciona com subtítulo
do lado) → **"Receita Fácil"**, decisão final. Motivo da troca de volta pra
um nome sem trocadilho: o **domínio** já reservado pra essa ferramenta é
`receitafacil.soaperando.com.br` (CNAME commitado antes de "SOAdesão"
existir) — usar "SOAdesão" como nome do produto faria o domínio ficar
`soadesao.soaperando.com.br`, com "soa" aparecendo 2x colado
(SOA-desão-SOA-perando), estranho de falar em voz alta pro paciente. Além
disso "Receita Fácil" é auto-explicativo sem precisar de subtítulo — mais
importante aqui do que manter a família de trocadilhos "SOA*" do
soaperando. "SOAcessível" também foi cogitado e descartado (redundante
com o título do documento impresso, "Receituário Acessível" — repetir
"acessível" nos dois soava eco).

**Título do CARTÃO IMPRESSO em si: "Receituário Acessível"** (2026-09-09,
antes "MEUS REMÉDIOS") — pedido explícito do usuário depois de ver o mock
mesclado (ver seção abaixo). É editável pelo médico por paciente
(`contenteditable`), então nada impede trocar de volta na hora se quiser
para um paciente específico. Comentários internos do código (`// cartão
pra paciente que não lê...`) não foram todos trocados — são jargão
interno, não texto que o usuário vê.

## Contexto
Ferramenta para médico da Estratégia de Saúde da Família (Agudos do Sul/PR)
gerar, por paciente, um cartão impresso que ensina a tomar medicação sem
depender de leitura. Uso em UBS com impressora preto e branco (prioridade),
cor é só reforço opcional.

## Arquivo atual
`cartao_remedios_editavel.html` — single-file HTML/CSS/JS, sem build, sem
dependência externa. Roda abrindo direto no navegador.

**`index.html` é cópia idêntica** de `cartao_remedios_editavel.html`, existe
só pra permitir acesso pela raiz do GitHub Pages
(`alyssonfigueiredo.github.io/cartao-remedios/`) sem precisar digitar o
nome do arquivo na URL. **Toda edição feita em um precisa ser replicada no
outro antes de commitar** — não há build/symlink automatizando isso.

## Como funciona
- Painel de edição à esquerda (`#controls`, classe `.no-print`): nome do
  paciente, form pra adicionar medicamento (nome+dose, símbolo, quantidade
  por tomada, cor opcional, posologia livre, turnos manhã/tarde/noite).
- Folha imprimível à direita (`#sheet`, tamanho A4): agrupada por turno
  (sol manhã / sol tarde / lua noite), um card por medicamento dentro do
  turno.
- Cada medicamento recebe um **símbolo geométrico** (círculo, quadrado,
  triângulo, estrela, losango, cruz, hexágono, pentágono) como código
  principal — não depende de cor. Se acabam os 8 símbolos, cicla por
  **padrões de preenchimento** (sólido, listras diagonais, pontos, xadrez,
  contorno) pra continuar distinguindo em P&B.
- Checkbox "unidade tem impressora colorida" liga cor como reforço extra
  (o símbolo continua sendo o código principal mesmo com cor ligada).
- **Faixa de turno sinaliza dia/noite por contraste, não por cor (2026-09-09).**
  `.turno-band.turno-noite` é sempre fundo escuro (`var(--ink)`) com lua
  branca (`svgTurno`) + 3 estrelinhas; `.turno-manha`/`.turno-tarde` são
  sempre fundo claro com sol de 12 raios grossos preto. Isso é layout/
  wayfinding da FOLHA, não identificação de medicamento — não é a mesma
  regra de "símbolo não depende de cor" (essa vale só pro código do
  remédio); o contraste claro/escuro funciona sozinho em P&B, sem precisar
  do toggle `modoCor`. Ícones grandes (`.turno-icon-wrap`, 19mm) e cards
  maiores (`.med-card` 48mm, `.glyph-wrap` 34mm) de propósito — pedido
  explícito do usuário pra preencher mais a folha A4 e ficar mais didático/
  visível (público-alvo inclui baixa visão, não só analfabetismo).
- **Economia de toner (2026-09-09).** `PATTERNS` reordenado pra `contorno`
  vir primeiro — o 1º ciclo de até 8 remédios (o caso comum) imprime só o
  contorno da forma, não preenchida de preto sólido; `solido` foi pro fim
  da lista, só usado se os símbolos ciclarem (>8 remédios distintos). Essa
  parte vale pros SÍMBOLOS dos remédios. **A faixa da noite, por pedido
  explícito do usuário, voltou a ser preenchida sólida** (`var(--ink)`,
  texto branco) — tentei um meio-termo (emblema circular escuro só atrás
  da lua, bem mais barato em tinta) mas o usuário preferiu a faixa cheia
  de antes; não reintroduzir o emblema sem pedido novo. "À TARDE" ganhou
  ícone próprio (antes reusava o sol da manhã): xícara de chá com
  fumacinha, em contorno — `svgTurno(turno,...)` tem 3 ramos
  (`noite`/`tarde`/manhã-padrão).
- **Bug real corrigido (2026-09-09): a lua não aparecia na impressão P&B.**
  Navegador OMITE `background-color` na impressão por padrão (economia de
  tinta) a menos que o usuário marque manualmente "imprimir gráficos de
  fundo" nas opções da impressora — a faixa escura da noite virava branca e
  a lua branca sumia em cima do branco. Corrigido forçando
  `print-color-adjust:exact` (+ prefixo `-webkit-`) em `*` dentro de
  `@media print` — o fundo escuro passa a sair sempre, sem depender de
  configuração nenhuma do usuário. Suportado em Chrome/Edge/Firefox/Safari
  atuais; qualquer elemento novo que dependa de `background` pra comunicar
  algo (não só cor de reforço opcional) tem que continuar coberto por essa
  regra — não reintroduzir cor-de-fundo-como-informação sem conferir a
  impressão de verdade, não só a tela.
- Instrução impressa em cada card: "desenhe este símbolo na caixa do
  remédio" — caregiver reproduz o símbolo com caneta na caixa real, paciente
  bate símbolo com símbolo.
- Contagem de comprimidos por tomada = bolinhas pretas repetidas (tally),
  separado do símbolo de identificação do remédio.
- Nome/título/rodapé da folha são editáveis direto no HTML via
  `contenteditable`.
- Botão imprimir = `window.print()` com CSS `@media print` escondendo o
  painel de controle.
- Sem persistência entre pacientes (proposital, é ferramenta de uso único
  por atendimento — abrir, preencher, imprimir, fechar).

## Ajustar para caber em 1 página (2026-09-09)

Checkbox `#fit1Pagina` (perto do botão imprimir). Quando ligado,
`ajustarParaCaberEmUmaPagina()` (chamada no fim de `render()` e no `change`
do checkbox) mede `#sheet.scrollHeight`, compara com a altura de uma A4
(`297mm` convertida em px a `96/25.4` px/mm — mesma conta que os artboards
de impressão usam) e, se estourar, aplica `sheet.style.zoom = fator`
(`fator = alturaPáginaPx / alturaAtualPx`, nunca abaixo de `FIT1_FATOR_MIN
= 0.55`). `zoom` foi escolhido em vez de `transform:scale()` de propósito:
`transform` só afeta o desenho, a paginação de impressão continua
calculando com o tamanho ORIGINAL (o navegador ache que ainda precisa de
2 páginas mesmo com tudo visualmente menor); `zoom` recalcula o layout de
verdade, então a paginação real também encolhe — confirmado gerando PDF de
verdade (Playwright, `page.pdf({printBackground:true})`): 8 medicamentos
em 2 turnos saíam em 2 páginas sem o ajuste e em 1 com ele ligado, fator
calculado automaticamente (0.5868 nesse caso).

**"Quando viável" é literal — nunca força abaixo do piso de legibilidade.**
Se o fator necessário for menor que `FIT1_FATOR_MIN`, aplica o piso mesmo
assim (melhor um pouco menor que nada) e mostra `#fit1PaginaAviso` avisando
que não coube nem no mínimo — vai sair em mais de uma folha de qualquer
jeito. Cartão pra paciente que não lê não pode virar cartão minúsculo
ilegível só pra caber numa folha; testado com um caso absurdo (9
medicamentos × 3 turnos = 27 cards) pra confirmar que o aviso aparece em
vez de forçar um zoom inútil.

Efeito colateral aceito: como `zoom` encolhe a caixa inteira (`#sheet` tem
`width:210mm`), a folha reduzida também fica mais ESTREITA que a A4 —
sobra margem lateral em vez de só cortar a altura. Mesmo trade-off que
qualquer "encaixar na página" de leitor de PDF; não vale a complexidade de
encolher só a altura mantendo a largura cheia.

`zoom` é suporte amplo hoje (Chrome/Edge/Safari sempre, Firefox 126+,
2024) — não teve fallback pensado pra Firefox mais antigo porque o
navegador do consultório é Chrome/Edge.

## Restrições de design que já foram validadas com o usuário
- Símbolo geométrico é o código PRINCIPAL, cor é sempre só bônus opcional
  — nunca inverter essa prioridade.
- Layout pensado pra impressão A4, preto e branco, sem depender de
  qualidade de impressora.
- Linguagem simples, sem gírias, texto mínimo (paciente não lê, quem lê é
  o cuidador/profissional).

## Importar de receita (foto/PDF) — 2026-09-08

Seção "Importar de receita" no painel de edição, acima do form manual de
medicamento. Fluxo:

1. Médico anexa foto (`accept="image/*"`) ou PDF da receita.
2. `extrairTextoArquivo`: PDF tenta extrair texto nativo via **PDF.js**
   (`pdfjsLib.getPage().getTextContent()`); se a camada de texto vier vazia
   (PDF escaneado, <15 chars), cai pra OCR página a página no canvas via
   **Tesseract.js** (idioma `por`). Foto vai direto pro Tesseract.
   Ambas as libs via CDN (`cdn.jsdelivr.net`), sem backend — processamento
   inteiro no navegador do médico, nada sobe pra servidor.
3. Texto bruto extraído aparece num `<textarea id="receitaTexto">` editável
   — se o OCR errar algo, o médico corrige o texto ali e clica
   "Reprocessar" em vez de reeditar cada campo depois.
4. `parseReceita` → `extrairBlocosMedicamento` corta o texto em blocos por
   linha que bate `RX_CABECALHO_MED` (nome + número + unidade `mg/mcg/g/
   ml/UI`) — cada bloco = 1 candidato a medicamento, texto seguinte até o
   próximo cabeçalho é a posologia daquele bloco.
5. `detectarQtdPorTomada` (comprimidos/cápsulas/gotas/ml) e `detectarTurnos`
   (palavras manhã/tarde/noite/jejum/deitar, ou frequência tipo "8/8h"/
   "3x ao dia" convertida em nº de tomadas → turnos) rodam por bloco.
6. **Entra direto no cartão, sem clique de confirmação (mudou 2026-09-09,
   pedido do usuário: "se eu prescrevi, já está pronta").** `autoImportarReceita`
   empurra cada candidato pra `medicamentos[]` imediatamente e chama `render()`
   — o argumento original de "nunca escrever sem confirmação" (mesmo princípio
   do soaperando) valia contra o risco de a RECEITA estar errada; aqui a receita
   já é do próprio médico, o risco real é só a LEITURA dela (OCR/posição de
   texto do PDF — ver os bugs reais abaixo). Rede de segurança que sobra:
   `importadosAtuais` guarda referência aos objetos importados na última
   leva, e `#candidatosList` renderiza cada um com nome/qtd/turno **editáveis
   in-place** (edita direto o objeto que já está em `medicamentos[]`, sem
   precisar remover e redigitar) e botão "🗑 remover do cartão". Reprocessar
   o texto (depois de corrigir algo à mão) remove a leva anterior antes de
   adicionar a nova — nunca duplica.

**Bug real corrigido (2026-09-09) — importação não gerava nada com PDF do e-SUS.**
Testado com receituário real (e-SUS, 4 medicamentos, "Atenolol 50mg 12/12h",
"Espironolactona 25mg 1x/dia", "Hidroclorotiazida 25mg pela manhã", "Losartana
50mg 12/12h"). Duas causas, as duas na extração/parsing — não no OCR:
1. `extrairTextoArquivo` juntava `content.items.map(it=>it.str).join(' ')` —
   o PDF.js devolve os itens de texto da página SOLTOS, sem quebra de linha
   própria; juntar tudo com espaço colapsava a página inteira numa única
   "linha", e `extrairBlocosMedicamento` (que assume 1 linha = 1 dado) nunca
   encontrava separação nenhuma → zero candidatos. Fix: `reconstruirLinhasPdf`
   quebra por `item.hasEOL` (quando o PDF.js sabe) e, de reforço, por salto
   de posição vertical (`transform[5]`) entre itens.
2. `detectarTurnos` não reconhecia "a cada 12 horas" (só "12/12h" e "de 12 em
   12 horas") — frequência real mais comum de receituário de e-SUS/PEC.
   Emenda: `a cada\s*(\d{1,2})\s*h(?:oras)?` antes do fallback de manhã-só.
Efeito colateral também corrigido: `RX_CABECALHO_MED` capturava o prefixo de
itemização do e-SUS ("Comprimido 1. Atenolol 50mg" em vez de "Atenolol
50mg") — limpo com strip de `^comprimido\s+` e `^\d+\.\s*` no nome. E
`detectarQtdPorTomada` podia confundir a quantidade TOTAL dispensada ("60
comprimidos", plural, avulsa) com a quantidade POR TOMADA ("1 comprimido,"
singular, colada à frequência) — agora prioriza singular+vírgula antes de
cair no fallback plural.
Validação inicial foi manual (texto linha-a-linha reconstruído à mão + script
Node descartável). Validação de verdade (2026-09-09, mesma sessão) veio
depois, com Playwright real (Chromium) anexando o PDF de verdade no
`#receitaInput` e clicando "Ler receita" de ponta a ponta — precisou baixar
`pdf.min.js`/`pdf.worker.min.js`/`tesseract.min.js` pra uma pasta local
porque o sandbox de teste bloqueia o Chromium headless de baixar CDN
externo (não é um problema do app; navegador real do médico não tem essa
restrição). Esse teste com o arquivo real achou um **2º bug real que o
teste manual não pegava**: `RX_CABECALHO_MED — duplicata de 2 vias`.

**Bug real corrigido (2026-09-09) — receita de 4 medicamentos virava 8 no cartão.**
Receituário do e-SUS/PEC sai com **2 vias na mesma página/PDF** ("1ª via —
retenção na farmácia" + "2ª via — orientação ao paciente"), lado a lado,
cada uma com a lista de medicamentos INTEIRA repetida. `extrairTextoArquivo`
lê a página inteira, então o texto extraído tinha os 4 medicamentos 2x cada
— sem dedup, cada um virava 2 cards idênticos (símbolo diferente cada um,
porque o índice do símbolo é por ORDEM de aparição, não por medicamento:
"Atenolol" saía como círculo E como losango). `dedupCandidatos` (chave =
nome normalizado sem acento + qtd + turnos ordenados) roda no fim de
`parseReceita`, antes de `autoImportarReceita`. Achado só porque o teste
usou o PDF real com Playwright de ponta a ponta — nenhuma reconstrução
manual de texto reproduz a duplicação de via, porque ninguém digitaria o
texto duas vezes por engano. **Lição:** teste de import de receita SEMPRE
com o arquivo PDF/foto de verdade num navegador real, nunca só com texto
reconstruído à mão — a extração de PDF real tem estrutura (colunas, vias,
posição) que reconstrução manual não reproduz.

**"Cor da caixa": 3 rodadas no mesmo dia até virar mancha esfumaçada
(2026-09-09).** Linha do tempo: (1) quadrado tracejado ao lado do símbolo,
onde o cuidador pintaria/escreveria a cor real da caixa à mão; (2) testado
lado a lado com "só a frase cinza, sem caixa" e "frase + linha pra
escrever" — venceu a frase sem caixa nenhuma (`.cor-caixa-label`), porque
o quadrado competia por espaço com o símbolo no cartão horizontal; (3)
usuário reconsiderou: queria o quadrado de volta, mas com receio real de
que uma forma de CONTORNO NÍTIDO (quadrado ou círculo) ao lado dos símbolos
geométricos do remédio (que são todos formas de contorno nítido) pudesse
ser lida pelo paciente como mais um símbolo pra decorar, confundindo com o
código real do medicamento. Uma variante intermediária (mesmo quadrado sem
o tracejado, borda sólida) tinha sido cogitada e descartada na rodada 2 —
borda sólida sem conteúdo lia como elemento incompleto/quebrado, pior que
o tracejado. Solução final: **mancha "esfumaçada"** — `.cor-caixa-box`
virou um `::before` com gradiente radial cinza + `blur(2px)`, SEM borda
nem contorno definido, propositalmente diferente de qualquer forma de
`SHAPES` (nenhuma tem borda difusa) — o texto "cor da caixa" fica por cima
num `<span>` separado do `::before` pra não borrar junto. Volta a ficar
DENTRO do `.top-row`, ao lado do `.glyph-wrap` (estrutura de HTML da
rodada 1, só a aparência do quadrado mudou). Não inverte a prioridade
símbolo-primeiro/cor-bônus da seção "Restrições de design" acima: funciona
igual sem impressora colorida, é só um lembrete visual de onde anotar a
cor à mão; o símbolo continua sendo o código que o app garante ser único
por medicamento. Validado com screenshot ampliado (zoom) lado a lado do
símbolo real, mostrando o contraste de forma (nítido vs. difuso) antes de
fechar.

**Limitações conhecidas / não resolvidas ainda:**
- `detectarTurnos` cobre só 3 turnos (manhã/tarde/noite); frequência >3x/dia
  (ex.: 6/6h = 4x) marca os 3 turnos existentes e liga `alertaExtra` (aviso
  visual pro médico conferir o horário que sobra) — não há 4º bloco
  "madrugada" no layout.
- Regex de cabeçalho de medicamento (`RX_CABECALHO_MED`) exige nome seguido
  de número+unidade na mesma linha ("Losartana 50mg") — receita com nome e
  dose em campos/linhas separados (formulário estruturado de algum PEC) não
  bate e o bloco não é detectado; nesses casos o candidato não aparece e o
  médico usa o form manual normal.
- `reconstruirLinhasPdf` é heurística (hasEOL + salto de Y) — não testada
  contra outros formatos de receituário além do e-SUS (o PEC municipal
  citado no soaperando). PDF de outra origem pode segmentar linha diferente;
  se vier errado, o texto bruto editável em `#receitaTexto` é a rede de
  segurança (corrigir manualmente e clicar "Reprocessar").
- Sem teste automatizado ainda (projeto não tem suíte) — validar heurística
  de `detectarTurnos`/`extrairBlocosMedicamento` contra receitas reais
  antes de confiar em uso clínico sem revisão atenta do texto bruto.

## Padrão pré-definido e texto sintético (2026-09-09)
- `#fit1Pagina` (ajustar para caber em 1 página) nasce **marcado por padrão**
  no HTML — pedido explícito pra já sair "pronto pra uma folha só" sem o
  médico precisar lembrar de ligar. Continua respeitando o piso de
  legibilidade (`FIT1_FATOR_MIN`) e o aviso quando não cabe.
- Textos do painel de edição e da folha impressa foram encurtados (queixa:
  "tem muito texto na página") — `.sub`, `.modo-box .desc`, `.import-box
  .desc`, `.hint`, `.obs` do cartão e `.marcar-caixa` de cada card. Cortou
  frase redundante, manteve a instrução essencial. Paleta/fontes seguem
  copiadas de `assets/tokens.css` do soaperando (confirmado direto no
  arquivo-fonte, não no site renderizado) — `--brand:#0f766e`, Plus Jakarta
  Sans; é a mesma paleta usada por `index.html` do soaperando (o teal
  `crystal-tokens.css` é só da tela de login/acesso, não usar como
  referência aqui).

## Logo (2026-09-09)
`logo.png` (256×256, redimensionado de `assets/logo-heart-crop-hires.png`
do soaperando — é o mesmo PNG usado no `.logo-sq` do cabeçalho do site
soaperando.com.br, não o favicon `logo.png` dele, que tem crop/gradiente
diferente; decisão do usuário: usar exatamente o que aparece no site) vive
na raiz, ao lado de `cartao_remedios_editavel.html`/`index.html`. Dois
lugares: `#controls .brand-row` (painel de edição, 34px ao lado do título,
não imprime) e `.head .brand-mark` (canto superior esquerdo do cabeçalho
impresso, 14mm, position:absolute sobre `.head{position:relative}` —
não desloca o título centralizado). Print-color-adjust já cobre `*`, então
sai colorido mesmo em impressora P&B convertendo pra cinza sozinha; é
decorativo (marca do consultório), não símbolo funcional do cartão — não
compete com a regra "símbolo é o identificador primário".

## Bug real corrigido: caixa de turno quebrando entre páginas (2026-09-09)
Usuário reportou com PDF real: quando o conteúdo passava de 1 página (sem
"ajustar para 1 página" ou mesmo com ela ligada, se excedesse o piso
`FIT1_FATOR_MIN`), a caixa de turno (`.turno-box`) era cortada NO MEIO
pela paginação de impressão — faixa colorida numa folha, símbolos na
outra, borda arredondada "reabrindo" sozinha no topo da 2ª folha
(`overflow:hidden` + `border-radius` fragmentados pelo motor de
impressão). Fix em 2 partes, dentro de `@media print`:
1. `.turno-box, .med-card{ break-inside:avoid; }` — cada caixa de turno e
   cada cartão vira unidade atômica: se não coube no resto da página,
   pula inteira, nunca corta no meio.
2. Isso sozinho expôs um 2º bug: com `#sheet`/`#turnosOut` em
   `display:flex; flex-direction:column`, quando uma caixa pulava inteira
   pra página seguinte, o Chromium não recalculava a posição do `.foot`
   (irmão seguinte no flex) pra nova página — o rodapé ficava sobreposto
   por cima do último cartão. Fix: `#sheet, #turnosOut{ display:block; }`
   só na impressão (a tela continua flex, pra esticar as caixas quando
   sobra espaço com poucos remédios) — bloco simples pagina certo no
   Chromium, flex/grid não. `.turno-box` continua flex por dentro (título
   + grade), mas como já é atômica (não quebra) isso não afeta paginação.
Validado com Playwright real (`page.pdf`) em 2 cenários: 6 remédios sem
"ajustar p/ 1 página" (2 páginas, caixa noite inteira e correta na 2ª,
rodapé sem sobrepor) e com a opção ligada (1 página só, tudo legível).
Lição: ao adicionar `break-inside:avoid` num item de flex/grid container
que pode precisar pular de página, testar também os IRMÃOS seguintes —
o motor de impressão do Chromium não gosta de flex/grid atravessando
quebra de página, mesmo com o item quebrado sendo atômico.

## Despoluir o cartão — instrução é pro paciente, não pra quem preenche (2026-09-09)
Usuário notou que boa parte do texto repetido em cada cartão de remédio
("desenhe na caixa") era instrução de PREENCHIMENTO (pra quem desenha o
símbolo/marca a cor, uma vez, com o cartão em mãos) — não informação que
o PACIENTE (analfabeto, é quem de fato lê/usa o cartão pronto) precisa
carregar consigo. A explicação de como preencher já existe uma vez só no
topo da folha (`.obs`); repetir por cartão era ruído.
- **Removido**: `.marcar-caixa` ("desenhe na caixa") de cada cartão —
  virou div morta, removida do JS (`render()`) e do CSS.
- **Nome do remédio em CAIXA ALTA** (`text-transform:uppercase` em
  `.med-name`) — mais fácil de reconhecer por quem já teve alguma noção
  de letras/embalagens (nome impresso na caixa geralmente também é
  maiúsculo), sem precisar mudar como o médico digita.
- **Total de comprimidos do turno com o MESMO peso visual do título**
  (`DE MANHÃ`/`À NOITE`) — antes era texto pequeno cinza (`.total`,
  15px/700), virou `.total-badge` com o número grande (23px/800, igual
  `.titulo`) e "COMPRIMIDOS" pequeno/maiúsculo embaixo como rótulo de
  unidade. É a 2ª informação mais importante do bloco pro paciente
  ("quantos eu tomo agora"), não deveria estar sub-hierarquizada.
- **Fileira-legenda de símbolos** (`.turno-legenda`) entre a faixa do
  turno e a grade de cartões: miniaturas (11mm) dos símbolos daquele
  turno, sem texto — resumo visual de relance antes de descer aos
  cartões com nome+cor. `svgGlyph(...)` é chamado 2x por remédio agora
  (legenda mini + cartão grande); cada chamada já gera `uid` aleatório
  próprio pro `<pattern>` SVG, sem colisão entre as duas.
Validado com screenshot real via Playwright (`#sheet` inteiro) — ver
hierarquia visual antes de fechar, não só o HTML gerado.

## Bolinha de contagem virou traço grosso (2026-09-09)
Usuário achou que a bolinha (`.pip`, redonda) ainda podia confundir o
paciente — parecida demais com "mais um símbolo geométrico" ao lado do
círculo/quadrado/triângulo do remédio. Mock comparando 4 alternativas
(traço vertical, tally de 5 com diagonal, tirinha de blister, risco
horizontal) mostrado lado a lado com os símbolos reais antes de decidir —
o tally só se justificaria se o teto de "vira numeral acima de 5" um dia
mudasse, e o blister competia visualmente com o quadrado/círculo do
remédio (borda dentro de borda). Vencedor: **traço grosso vertical**, sem
elemento novo (só a marca, sem contorno ao redor). `.contagem .pip` foi de
`width:3.8mm; height:3.8mm; border-radius:50%` (bolinha) pra
`width:3mm; height:11mm; border-radius:1mm` (traço), com `gap` de 1.6mm
pra 2.4mm (mais espaçado, pedido explícito). Nome da classe CSS/JS
(`pip`) não mudou — só a aparência; `renderContagem()` não precisou de
nenhuma alteração.

## 4 bugs reais achados testando com receitas reais no navegador (2026-09-09)
Usuário pediu pra testar com PDF real dele antes de confiar na extração —
pedido certeiro: 3 dos 4 bugs abaixo só existem com dado real (e-SUS de
Curitiba), nenhum sintético teria achado. Método: `npm install
pdfjs-dist@3.11.174` local (o CDN jsdelivr é bloqueado no proxy desta
sessão) servido junto de uma cópia do `index.html` com as 2 tags
`<script src=cdn...>` do pdf.js trocadas pra apontar pro arquivo local —
só pra rodar o teste; a extração roda IDÊNTICA (mesmo pdf.js, mesma
versão), só a origem do arquivo muda. Testado com 5 receitas reais
(Andressa, Carolina, Célia — cada uma com atestado + receituário de
verdade, incluindo criança em uso de líquidos).

1. **Nome do paciente nunca era extraído.** O e-SUS de Curitiba rotula
   `Usuário:`, não `Paciente:`/`Nome do paciente:` (só cogitados por mim,
   nunca confirmados contra dado real). `extrairNomePaciente` ganhou
   `usuário:` como padrão FORTE (1º da lista) — os outros ficam de
   fallback pra outros municípios/sistemas. Achado adicional: o pdf.js
   cola a coluna "Idade" direto depois do nome na mesma linha reconstruída
   (`"Usuário: CELIA PEREIRA DIAS 64"`) — `limpar()` ganhou
   `.replace(/\s+\d+\s*$/, '')` pra cortar esse número solto no fim, que
   nome nenhum tem de verdade.
2. **"Quantidade:" grudava no início do nome do remédio**
   (`"Quantidade: PARACETAMOL 500 MG COMPRIMIDO"`). A tabela da receita
   tem "Quantidade" numa coluna separada, mas o pdf.js reconstrói a linha
   por proximidade vertical (Y) e cola o rótulo antes do nome na mesma
   linha. Fix em `extrairBlocosMedicamento`: strip de um rótulo solto
   `palavra:` no início do nome capturado, antes dos strips que já
   existiam (`comprimido `, numeração).
3. **Linha de posologia virava medicamento fantasma.** `"DAR 10 ML, 1X AO
   DIA..."` bate no mesmo padrão "nome + número + unidade" que um
   cabeçalho de remédio de verdade — sem guarda, cada posologia de líquido
   criava um "remédio" a mais (achado com receituário pediátrico real: 3
   remédios reais + 2 fantasmas `"DAR 10ML"`/`"TOMAR 5ML"`).
   `RX_CABECALHO_MED` ganhou lookahead negativo pra linha que COMEÇA com
   verbo de administração (dar/tomar/aplicar/usar/administrar/ingerir/
   pingar/instilar/inalar).
4. **Líquido (gotas/ml) virava bolinha de comprimido — risco de dose.**
   Receita real: "PARACETAMOL 200MG/ML GOTAS — DAR 35 GOTAS de 8/8h". O
   app desenhava 35 bolinhas pretas, idênticas às de contagem de
   comprimido — um cuidador podia ler "35 comprimidos". E mais: a
   abreviação **"CP"** de comprimido (rotina em posologia real — "TOMAR 01
   CP DE 6/6H") não tinha prioridade sobre a quantidade TOTAL dispensada
   ("Quantidade: ... 10 comprimido(s)"), então "1 comprimido de 6/6h"
   saía como "10 COMPRIMIDOS" no cartão — dose errada por um fator de 10,
   achado nas receitas reais da Célia e da Carolina (ambas usam "CP").
   Duas mudanças: (a) `detectarQtdPorTomada` devolve `{qtd, unidade}` e
   tenta primeiro `verbo(tomar|dar|aplicar|usar|ingerir|pingar) + número +
   unidade` (a posologia de verdade), só caindo no padrão solto antigo
   (que pega o que aparecer primeiro no texto, geralmente o total) se
   nenhuma posologia com verbo for achada; (b) `renderContagem(m)` — nova
   função central que decide bolinha vs. numeral: unidade "gotas"/"ml"
   NUNCA vira bolinha (sempre numeral grande + rótulo da unidade,
   `CONTAGEM_MAX_BOLINHAS` não se aplica), e comprimido acima de 5 vira
   numeral também (bolinha só serve pra contagem de relance, não pra
   substituir número escrito). O total do CABEÇALHO do turno
   (`totalComprimidos`) também parou de somar unidade líquida junto —
   soma só itens `unidade==='comprimido'`, e o badge inteiro (divisor +
   número) some quando não sobra nenhum comprimido de verdade naquele
   turno, em vez de mostrar "0 COMPRIMIDOS" ou misturar unidades.
   Medicamento adicionado à mão pelo formulário sempre tem
   `unidade:'comprimido'` (o campo do form já se chama "Comprimidos por
   tomada", sem ambiguidade).
Validado com Playwright real (Chromium + pdf.js real) nas 3 receitas reais
— PDF final de cada uma conferido visualmente (screenshot) e como PDF de
impressão de verdade, sem quebra de página, sem medicamento fantasma, sem
dose inflada, sem bolinha de líquido.

## Bug real corrigido: "ajustar para 1 página" tinha pontos cegos de recálculo (2026-09-09)
Usuário reportou PDF real quebrando em 2 páginas com uma receita de 4
remédios de manhã + 2 à noite — geometricamente cabia numa página (2 boxes
de turno bem abaixo de 297mm de altura somados), mas saiu partido mesmo
assim. Investigando `ajustarParaCaberEmUmaPagina()`, achei que ela só era
chamada em 2 lugares: depois de `render()` (toda vez que a lista de
remédios muda) e no `change` do checkbox "ajustar para 1 página". **Ela
NUNCA era recalculada:**
- Ao editar `#pacienteInput`/`#rodapeInput` (só atualizavam o texto de
  saída, sem re-medir a altura).
- Ao editar direto os campos `contenteditable` da folha (nome do
  paciente, título "Receituário Acessível", texto de instrução, rodapé) —
  zero listener neles antes deste fix.
- **Antes de imprimir de verdade** — não havia rede de segurança
  recalculando no clique do botão, então qualquer mudança de layout entre
  a última medição e o clique em "Imprimir" (inclusive a fonte Google
  Fonts, que carrega via `@import` assíncrono e pode ainda não ter
  terminado no momento em que `render()` mediu a altura pela 1ª vez —
  clássica causa de "cabe na tela, estoura no PDF": a métrica do texto
  muda depois que a fonte troca do fallback do sistema pra Plus Jakarta
  Sans, e ninguém remedia depois disso) passava direto pro PDF sem chance
  de correção.
**Fix:** (1) `document.querySelectorAll('#sheet [contenteditable]')` +
listener `input` chamando `ajustarParaCaberEmUmaPagina` em cada um; (2)
mesma chamada adicionada dentro dos listeners de `pacienteInput`/
`rodapeInput`; (3) `document.fonts.ready.then(ajustarParaCaberEmUmaPagina)`
— recalcula assim que a fonte customizada termina de carregar, não importa
quando isso aconteça; (4) `printBtn.onclick` agora chama
`ajustarParaCaberEmUmaPagina()` antes de `window.print()` — rede de
segurança final. **Não consegui reproduzir a corrida exata de fonte
assíncrona neste ambiente** (o proxy da sessão bloqueia
`fonts.googleapis.com`/`fonts.gstatic.com`, então aqui a Plus Jakarta Sans
nunca carrega e todo teste roda 100% no fallback do sistema, sem swap de
fonte pra reproduzir a corrida) — mas validei com Playwright que os 4
pontos de recálculo novos DE FATO mudam o zoom aplicado quando disparados
(editar o `h2` contenteditable moveu o zoom de 0.999 pra 0.845 numa
receita de teste), então os gaps reais que existiam (edição sem
recálculo) estão fechados mesmo sem confirmar 100% que a corrida de fonte
era a causa exata deste caso específico.

## Nome do paciente extraído automaticamente da receita importada (2026-09-09)
Pedido do usuário: a receita de origem (e-SUS/PEC) já traz o nome do
paciente no texto — o app só usava o texto pra achar MEDICAMENTOS, o nome
ficava sempre em branco ("NOME DO PACIENTE") esperando o médico digitar de
novo algo que já estava no papel. `extrairNomePaciente(texto)`
(`cartao_remedios_editavel.html`) varre linha por linha com 2 níveis de
padrão: **fortes** primeiro (`Nome do paciente:`, `Paciente:`), só caindo
pro **fraco** (`Nome:`) se nenhum forte bater — e o fraco exige `:` colado
em "Nome" (sem espaço no meio) bem de propósito, senão "Nome do
medicamento: Losartana" seria lido como nome de paciente. `limpar()` corta
lixo colado na mesma linha (CPF, data de nascimento, número de cartão
SUS/CNS) que o e-SUS costuma grudar depois do nome sem quebra de linha.
Chamado em `lerReceitaSelecionada()` logo depois do parse de medicamentos,
**só preenche se `#pacienteInput` ainda estiver vazio** — nunca sobrescreve
nome que o médico já digitou ou corrigiu manualmente (mesmo comportamento
de "nunca sobrescrever o usuário" já usado em `_herdarContextoCalc` do
soaperando, adaptado aqui). Status da leitura ganha uma frase extra
("Nome do paciente identificado automaticamente — confira.") só quando
acha algo, pra não afirmar confiança que não existe quando o padrão não
bate. Testado com 5 casos via Node (nome + lixo na mesma linha, "Nome:"
fraco coexistindo com "Nome do medicamento:" sem colidir, receita sem
nome nenhum) — todos corretos.

## Layout mesclado a partir de mock gerado por IA (2026-09-09)
Usuário mandou uma imagem gerada por IA como referência de layout e pediu
pra avaliar o que valia aproveitar. Processo: montei um mock estático
separado (`scratchpad/mock-mesclado.html`, fora do app real) reutilizando
as funções reais de `svgGlyph`/`svgTurno` pra comparar com fidelidade,
mostrei screenshot, várias rodadas de ajuste conversando sobre o que
pegar/rejeitar, só então portei pro arquivo de produção. **O que foi
aproveitado do mock da IA:**
- **Divisor vertical** (`.turno-divisor`, 1px) entre o título do turno e o
  total de comprimidos, dentro de `.turno-band` — separa "o que é" de
  "quanto é" sem gastar espaço extra.
- **Cartão horizontal**: caixinha "cor da caixa" ao LADO do símbolo
  (`.top-row`, flex row), não empilhada em cima — cartão mais baixo.
  `.med-card` ganhou `border-left` (divisor fino entre cartões, tipo
  colunas de tabela) em vez do gap solto de antes.
- **Título do documento mais fino/discreto**: `.head h2` foi de
  `font-weight:800`/32px pra `font-weight:300`/26px com
  `letter-spacing:.14em` e uppercase — é rótulo de identificação, não
  informação que o paciente precisa decodificar (quem carrega peso visual
  continua sendo o título de turno e o nome do remédio, ambos 800).
  Exigiu adicionar o peso 300 ao `@import` do Google Fonts (só vinha
  400-800).

**O que foi avaliado e REJEITADO de propósito** (registrar pra não
reaparecer como "esqueceram disso"):
- **Fileira-legenda de símbolos pequenos** (`.turno-legenda`, existia
  numa versão anterior do mesmo dia) — avaliação do usuário: repete os
  mesmos símbolos que já aparecem grandes 2 linhas abaixo, sem
  acrescentar informação, e quebra o ritmo visual antes do cartão de
  verdade. Removida (HTML do `render()` + CSS).
- **Total do turno como única contagem** (o mock da IA não mostra pips
  de contagem por remédio, só o total agregado no cabeçalho) — mantive as
  bolinhas de contagem por cartão (`.contagem`): pra um paciente que toma
  2+ remédios no mesmo turno, o total sozinho não diz quantos comprimidos
  de CADA um, e é exatamente esse dado que evita erro de dose.
- **Cabeçalho com logo+tagline do soaperando + selo do SUS** — são
  elementos de marca específicos daquela referência; o cabeçalho atual
  (logo + nome da ferramenta) já cumpre o mesmo papel de identificação.

**Bug real achado NO PROCESSO (não fazia parte do pedido): flex-wrap
multi-linha esticava o divisor até o fundo da caixa.** Com `.turno-grid`
em `display:flex; flex-wrap:wrap`, quando o nº de remédios não enche a
última linha exatamente (ex.: 3 remédios, 2 numa linha + 1 sozinho na
linha de baixo), o comportamento padrão do `align-content`/`align-items`
("stretch") distribui o espaço vertical sobrando do `.turno-grid`
(`flex:1`) entre as linhas — e como `.med-card` tem `border-left`, esse
divisor esticava junto até o fim da caixa do turno inteiro, uma linha
vertical solta sem função. Fix: `align-content:flex-start;
align-items:flex-start;` no `.turno-grid` — cada linha (e cada card)
passa a ter só a altura do próprio conteúdo. Validado visualmente com
Playwright, com uma receita real de 3+2+2 remédios (o caso que reproduzia
o bug, 3 não é múltiplo do que cabe por linha).

## Bug real corrigido: preenchimento sólido/colorido não aparecia (2026-09-09)
Achado por acaso construindo um mock pra comparar layout (não fazia parte
do pedido original). `patternDefs()` (função de símbolo, `svgGlyph`)
define os padrões "sólido", "contorno" e o modo colorido (`usaCor`) com
`<pattern width="1" height="1"><rect width="1" height="1" .../></pattern>`
sem `patternContentUnits`. O padrão SVG desse atributo é
`userSpaceUse` — então o `rect width="1" height="1"` é 1 pixel real, não
100% do tile (que É 100% da forma, via `patternUnits` default
`objectBoundingBox`) — o preenchimento vira um pontinho de 1px escondido
no canto, e a forma renderiza como se fosse contorno vazio. `contorno`
(fill branco) mascarava o próprio bug — o resultado visual "errado" é
idêntico ao "certo" pra um fundo branco — então nunca foi notado. Mas
**"sólido" (2º ciclo do mesmo formato) e o modo colorido inteiro estavam
quebrados em produção**: qualquer receita com 9+ remédios do mesmo
formato, ou com "impressora colorida" marcado, saía com símbolos
indistinguíveis entre si — o oposto do que a ferramenta existe pra
garantir. Confirmado reproduzindo o bug isolado (SVG puro) e depois
chamando `svgGlyph(...)` direto no app real via Playwright antes do fix
(círculo "sólido" e círculo colorido saíam ambos como contorno vazio) e
depois (preenchimento preto sólido e teal aparecem certos). Fix: acrescentar
`patternContentUnits="objectBoundingBox"` nos 3 `<pattern>` afetados —
`listrasDiag`/`pontos`/`xadrez` já usam `patternUnits="userSpaceOnUse"`
com pixels reais de propósito e não tinham esse problema.

## Domínio próprio: receitafacil.soaperando.com.br (2026-09-09, pendente do lado DNS)
Usuário pediu esse domínio (subdomínio do soaperando.com.br, que já é do
consultório). Feito o que dava pra fazer daqui: arquivo `CNAME` na raiz
do repo com `receitafacil.soaperando.com.br` (é assim que o GitHub Pages
sabe servir esse domínio em vez de só `alyssonfigueiredo.github.io/cartao-remedios/`).
**Falta o lado DNS, fora do alcance desta sessão** (sem acesso à conta
Cloudflare do soaperando) — passos que o usuário precisa fazer manualmente:
1. Cloudflare → DNS do domínio `soaperando.com.br` → adicionar registro
   CNAME: nome `receitafacil`, destino `alyssonfigueiredo.github.io`,
   proxy **desligado** (DNS only/nuvem cinza) até o certificado do GitHub
   ser emitido — proxy ligado antes disso costuma travar a validação.
2. Esperar propagar (minutos a poucas horas).
3. GitHub → repo `cartao-remedios` → Settings → Pages → em "Custom domain"
   já deve aparecer `receitafacil.soaperando.com.br` (veio do `CNAME`
   commitado) — aguardar o check de DNS ficar verde e marcar "Enforce
   HTTPS" assim que disponível.
4. Depois de confirmado, pode ligar o proxy do Cloudflare (nuvem laranja)
   se quiser as proteções dele — não é obrigatório.
URL antiga (`alyssonfigueiredo.github.io/cartao-remedios/`) continua
funcionando em paralelo (GitHub Pages não desliga a URL padrão).

## Importar mais de uma receita/foto de uma vez (2026-09-09)
`#receitaInput` ganhou `multiple` — usuário pode selecionar 2+ arquivos
(ex.: 2 receitas separadas, ou várias fotos de embalagem) numa única
escolha. `lerReceitaSelecionada()` virou um laço: lê cada arquivo em
sequência com `extrairTextoArquivo` (status mostra "Lendo arquivo N/M..."
quando há mais de 1), concatena todo o texto extraído e processa como
receita única (`parseReceita` + `autoImportarReceita`, sem mudança nos
dois) — os medicamentos de todos os arquivos entram juntos na mesma folha.
`dedupCandidatos` (já existente, criado pro caso de 2 vias do e-SUS no
mesmo PDF) cobre de graça o usuário anexar o mesmo arquivo 2x sem querer.
Sequencial de propósito, não paralelo — mantém a ordem do status legível
e evita rodar todo o OCR ao mesmo tempo se o médico anexar várias fotos
grandes.

## Leitura de receita dispara sozinha ao anexar (2026-09-09)
Usuário tinha que escolher o arquivo E lembrar de clicar em "Ler receita"
— passo fácil de esquecer, e nada avisava se esquecesse (arquivo escolhido,
nada acontece). A lógica do clique virou a função nomeada
`lerReceitaSelecionada()`, chamada tanto pelo botão quanto por um listener
`change` no próprio `#receitaInput` — a leitura começa assim que o arquivo
é anexado, sem esperar clique nenhum. O botão continua existindo, mas
virou "↻ Ler de novo" (reprocessa o mesmo arquivo — útil se o 1º OCR saiu
ruim ou algo falhou). Validado com Playwright: `setInputFiles` sozinho
(sem `.click()` no botão) já dispara `#receitaStatus` mudando pra
"Processando...".

## Folha só aparece depois do 1º medicamento (2026-09-09, ajustado no mesmo dia)
Usuário notou que a folha A4 em branco (`#sheet`) sempre visível à direita,
mesmo antes de qualquer medicamento, confundia — parecia erro/vazio em vez
de "ainda não preenchi nada", e ocupava metade da tela sem função alguma
nesse momento. `#sheet` nasce com `display:none` (era `flex`) e ganha a
classe `.visivel` (toggle em `render()`, junto do resto da renderização)
só quando `medicamentos.length > 0` — com uma pequena animação de entrada
(`sheetIn`, fade+slide 6px) pra não ser um "pulo" seco na tela.

**1ª versão (revertida no mesmo dia): placeholder `#vazioInicial` na 2ª
coluna.** Mostrava uma mensagem curta ("O cartão aparece aqui...") no
lugar da folha enquanto vazio — mas o usuário pediu que a tela ficasse
"toda com a primeira coluna" nesse estado, não com uma 2ª coluna estreita
de aviso. `#vazioInicial` foi removido (HTML + CSS); no lugar, `#controls`
ganha a classe `.solo` (mesmo toggle de `render()`, `!temMed`) que aumenta
sua largura de 400px pra 640px — o formulário sozinho ocupa mais espaço
central em vez de ficar cercado de branco vazio dos dois lados. Ao
adicionar o 1º remédio, `.solo` sai e `#controls` volta a 400px enquanto
a folha (`.visivel`) surge ao lado — layout de 2 colunas volta ao normal.
`.empty-msg` (mensagem antiga DENTRO do `#sheet` vazio, de uma versão
ainda anterior) já não existe.

Validado com Playwright nos 2 estados: vazio (só a coluna do formulário,
mais larga, sem a folha) e com 1 medicamento (formulário volta a 400px,
folha aparece ao lado).

## Possíveis próximos passos (não pedidos ainda, só ideias em aberto)
- Persistir múltiplos pacientes numa sessão (lista salva localmente).
- Exportar/importar lista de medicamentos comuns (evitar redigitar Losartana
  toda vez).
- Modo "várias posologias no mesmo dia" (ex.: 2x manhã + 1x tarde) já é
  suportado via múltiplos turnos, mas quantidade é fixa por tomada — avaliar
  se precisa quantidade diferente por turno no mesmo medicamento.
- Testar impressão real em impressora P&B de UBS pra validar legibilidade
  dos padrões de preenchimento (listras/pontos/xadrez) em baixa resolução.
