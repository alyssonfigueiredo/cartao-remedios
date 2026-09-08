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
