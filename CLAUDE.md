# Cartão de Remédios — gerador visual para pacientes analfabetos

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
6. Candidatos aparecem em `#candidatosList`, cada um com nome/qtd/turnos
   **editáveis** e botão próprio "+ adicionar ao cartão" — nenhum
   candidato entra na lista de medicamentos (`medicamentos[]`) sem esse
   clique explícito. Mesmo princípio de segurança do soaperando (nunca
   escrever automaticamente sem confirmação do médico): leitura de
   receita erra nome/dose/horário com frequência real, e um remédio
   errado nesse cartão é diretamente perigoso (o paciente não lê pra
   conferir).

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
Sem suíte de teste no projeto ainda; validação foi manual, reconstruindo o
texto linha-a-linha esperado da receita real e rodando `parseReceita` num
script Node descartável (`node -e "..."` fatiando o HTML pelos marcadores
`const RX_CABECALHO_MED` / `function adicionarMedicamentoDaLista`) — receita
com os 4 medicamentos bateu turno-a-turno e qtd-a-qtd contra o resultado
esperado antes de aplicar o fix no arquivo real.

**Quadrinho "cor da caixa" (2026-09-09).** Adicionado ao `.med-card`, ACIMA
do símbolo geométrico — dashed box em branco onde o cuidador pinta ou
escreve a cor real da caixa do remédio (ideia validada com o usuário a
partir de um mockup). Não inverte a prioridade símbolo-primeiro/cor-bônus
da seção "Restrições de design" acima: funciona igual sem impressora
colorida (é só um espaço pro cuidador desenhar/escrever à mão), o símbolo
continua sendo o código que o app garante ser único por medicamento.

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

## Possíveis próximos passos (não pedidos ainda, só ideias em aberto)
- Persistir múltiplos pacientes numa sessão (lista salva localmente).
- Exportar/importar lista de medicamentos comuns (evitar redigitar Losartana
  toda vez).
- Modo "várias posologias no mesmo dia" (ex.: 2x manhã + 1x tarde) já é
  suportado via múltiplos turnos, mas quantidade é fixa por tomada — avaliar
  se precisa quantidade diferente por turno no mesmo medicamento.
- Testar impressão real em impressora P&B de UBS pra validar legibilidade
  dos padrões de preenchimento (listras/pontos/xadrez) em baixa resolução.
