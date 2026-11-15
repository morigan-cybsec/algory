# O Algory em cada aba

O Algory é o assistente que acompanha o usuário pelo app. Em cada tela ele faz uma coisa diferente, e quase tudo roda **sem IA**:

- **regra:** expressões e vocabulário de negócio, no navegador ou no servidor;
- **consulta direta ao banco do projeto:** DuckDB, uma base por projeto.

A IA só entra quando a regra não resolve, e cada chamada é cobrada em tickets do plano.

| Símbolo | Significado |
|---|---|
| 🟢 | regra ou banco, não gasta ticket |
| 🟣 | IA, gasta os tickets da chamada |
| 🔒 | barreira de segurança |

Front: arquivos `ui*.js`, que o `build.py` junta num HTML só. Servidor: `algory_api/app`.

---

## 1. Chat do projeto

É a conversa ao lado do dashboard. Cada frase passa por uma cadeia de tratadores, em `pHandle`. O primeiro que reconhece o pedido responde:

| Ordem | Tratador | O que reconhece | Como responde |
|---|---|---|---|
| 1 | Ajustes da pasta, `ui88.js` 🟢 | fundo, cores, grade, animação, modo de exibição, celular, layout pronto | Chama as mesmas funções dos botões (`setRepTema`, `setPal`, `setLay`, `P.dmodes`…), aplica na hora e oferece **Desfazer** (guarda um retrato do estado antes) |
| 2 | Conteúdo do relatório, `ui72.js` → `POST /projetos/{pid}/relatorio/pedido` 🟢/🟣 | tirar ou mostrar bloco, top N, ordem, período, filtro, trocar medida, agregação, renomear | `relatorio.interpretar` converte a frase numa **especificação** (medidas, dimensões, filtros, títulos). O usuário vê a **proposta** e aprova; só então ela vira **versão** (quantas versões ficam guardadas depende do plano). A IA só entra se a regra não entender |
| 3 | Layout dos modelos, `sharedIntent` 🟢 | aumentar, diminuir ou esconder cartão, variação executiva ou analítica, apresentação 16:9, exportar | Mexe direto no layout do modelo |
| 4 | Fórmulas, `ui61.js` 🟢 | "=receita / pedidos" | Lê a fórmula, aponta erro (divisão por zero, "* 100" sobrando) e mostra o valor com os dados |
| 5 | Perguntas, `POST /ask` → `pipeline.py` | "qual loja vendeu mais?" | Segue o trâmite abaixo |

**Trâmite de uma pergunta** (`pipeline.py`):

1. 🔒 **Máscara** (`mask.py`): CPF, CNPJ, e-mail, telefone, cartão e nomes viram marcadores antes de qualquer modelo.
2. 🟢 **Árvore e cache** (`router.py`, camada T0): as perguntas frequentes têm SQL pronto e não chamam modelo.
3. **Cota de tickets e roteador** (camada T1): a regra decide a rota. O modelo só é chamado se a confiança da regra ficar abaixo de 0,6.
4. 🟣 **Geração de SQL**: o modelo recebe só o **esquema** (nomes de colunas e exemplos mascarados).
   - 🔒 `sqlguard.py` executa no DuckDB em modo somente leitura, só nas tabelas do projeto, com limite de 10 mil linhas e 5 s.
5. 🟣 **Resposta**: o modelo escreve os números só como marcadores `{{vN}}`. Quem põe o valor é o servidor, a partir do resultado da consulta, então o modelo não consegue inventar número.
6. 🔒 **Conferência**: citação de documento precisa existir (`rag.py`). Tudo o que vem da planilha vai entre `<dados>` e é tratado como dado, nunca como instrução.
7. **Cobrança e registro**: a cobrança usa o custo real em dólar convertido em tickets, com mínimo de 1. Cada passo fica registrado no traço.

🔒 **Provedores** (`modelos.py`, `fila()`):
- Com dado real de cliente, a fila **remove todo provedor que treina** com os dados.
- O texto que o cliente lê sai do Claude Haiku; os outros só entram se o Claude cair.

## 2. Tabelas (confirmar o envio)

Ao subir uma planilha, ela é lida no servidor (`planilha.py`) antes de o dashboard ser montado:

| O que faz | Como |
|---|---|
| Acha as tabelas dentro das abas | `quadros`, `analisar`: blocos contíguos com cabeçalho |
| Separa as informações soltas da planilha | `soltas`: metas, títulos e anotações fora dos quadros |
| Lê fórmulas, inclusive as compartilhadas | `formulas`, `_desloca` |
| Avisa quando uma aba puxa dados de outra por fórmula | `ligacoes` |
| Converte meses em colunas em linhas | `deitar` |
| Tira colunas de identificação 🔒 | Nome, CPF, e-mail e telefone saem **antes** de gravar, inclusive na prévia e no retrato |
| Sugere o papel de cada coluna | `colunas.py`: data, medida, meta, dimensão, chave |
| Calcula prazos em dias úteis | `prazos`, com os feriados públicos |
| Cria contas entre colunas | `contas`: soma, razão e diferença |
| Arruma categorias 🟢/🟣 | `normaliza.py`: por regra (acento, maiúscula, espaço) **e**, nos planos pagos, por IA, que recebe só os valores distintos das categorias, nunca linhas. O usuário vê a proposta antes de aplicar |

O dashboard só é gerado depois que o usuário confirma as colunas: a rota do painel devolve `pendente` enquanto isso.

## 3. Planilha (visualização)

Mostra a planilha original guardada, com os quadros destacados (`ui83.js`).

- No computador, ao trocar de tabela, a câmera faz zoom out, desloca e faz zoom in.
- No celular, a vista é leve: sem transformação, desenha só o trecho escolhido.
- Mostra no máximo 200 linhas por vez, nunca o arquivo inteiro.
- O arquivo fica em `arquivos/<sha12>.json.gz` já limpo de identificadores.

## 4. Associar (ligar tabelas)

O Algory fica **fixo na lateral** (`ui87.js`) e fala conforme o momento. Tudo aqui é 🟢:

- **Visão geral:**
  - sugestões de ligação, cada uma com o motivo;
  - tabelas que ainda não estão ligadas a nenhuma outra;
  - uma pergunta pronta para o chat quando tudo já está ligado.
- **Coluna escolhida:** as colunas de outras tabelas que mais batem com ela.
- **Ligação aberta:**
  - quem fez, com data;
  - o motivo, que dá para editar;
  - quantos valores batem hoje, com exemplos;
  - o que aparece só de um lado.

Como as ligações nascem:
- **Automáticas** (`comparar.sugestoes`): mesmo nome de coluna e pelo menos 80% dos valores do menor conjunto iguais. São criadas depois de cada gravação de tabela e registram o motivo.
- **Sugeridas:** nomes diferentes com a mesma sobreposição de valores. Aparecem tracejadas; se aceitas, ficam com a origem `aceita`.

Origens registradas: `voce`, `auto`, `ia` e `aceita`. 🔒 O cliente não pode gravar uma ligação como `auto`.

A comparação entre tabelas (`comparar.py`) usa as ligações primeiro e, na falta delas, colunas de mesmo nome.

Duas vistas: **Plano**, com as tabelas lado a lado, e **Na planilha**, com os pedaços reais de cada aba. Na vista da planilha as linhas são roteadas só pelos espaços vazios (`ui86.js`):
- elas sobem do título, correm em faixas acima das tabelas e descem pela lateral;
- as faixas são distribuídas em trilhas, com a ligação mais curta mais perto das tabelas, para as linhas se aninharem em vez de se cruzarem;
- passar o mouse num título destaca as ligações dele.

## 5. Comunidade

O Algory lê a busca e procura no **repertório inteiro** (`ui90.js`): modelos do Algory, modelos e cartões publicados, conjuntos de cartões.

- **Sinônimos de negócio:** faturamento ≈ receita, filial ≈ loja, churn ≈ cancelamento.
- **Radicais:** "vendedor" acha "vendedores".
- **Pontuação:** título vale 3, área 2, descrição 1; um sinônimo vale 70% de uma palavra igual.
- **Explicação:** mostra os 3 mais parecidos com o porquê ("bate em: faturamento ≈ vendas, filial ≈ loja") e o botão de abrir.

Exemplo: "faturamento por filial" dá zero na busca exata e acha 44 itens relacionados. Tudo 🟢, no navegador.

Publicar 🔒 (`comunidade.py`):
1. **Integridade, no envio:** só entra estrutura (tipos de cartão, tamanhos, títulos). Nada de linhas de dados, números soltos, URLs ou dados pessoais.
2. **Moderação, às 03:00:** palavrões, normalizados contra acento, leet e letras espaçadas, recusam o item; assunto pesado vai para revisão manual.
3. **Imagens:** nunca são publicadas; o repertório mostra só uma descrição fixa ("Logo da empresa").

## 6. Cartão de imagem no dashboard

Disponível nos planos pagos e para quem entra com chave de demo (`ui89.js`, `imagem.py`, `POST /imagens/verificar`). O processo:

1. 🔒 **Preparo:** o servidor abre a imagem de verdade (não confia na extensão).
   - Limita o tamanho em pixels, contra bomba de descompressão.
   - Reduz para 1568 px e regrava em JPEG, o que **remove o EXIF**, inclusive a localização GPS.
2. 🟢 **Texto dentro da imagem:** OCR local (Tesseract) e o mesmo filtro da comunidade.
3. 🟣 **Conteúdo:** o Claude Haiku com visão classifica em categorias fechadas: sexual, violência, ódio, drogas ou armas, documento pessoal, menor em destaque.
   - A chamada vai **só para a Anthropic**, nunca pela fila de provedores, porque é imagem de cliente.
   - O texto escrito na imagem é tratado como conteúdo a avaliar, nunca como instrução.
   - Custa cerca de 1 ticket.
4. **Veredito:**

| Resultado | Quando |
|---|---|
| `recusada` | sexual, violência ou ódio |
| `revisao` | outras categorias ou dúvida; só o dono vê a imagem |
| `pendente` | sem IA naquele momento; só o dono vê |
| `aprovada` | nenhuma categoria encontrada |

A imagem **não é guardada no servidor**: fica no navegador do usuário. O link público mostra só os dados do dashboard, sem imagens.

## 7. Integrações (Google Sheets)

Só Google Planilhas por link publicado é real hoje. As outras aparecem como **Em breve**; ver `docs/integracoes.md`.

Um tutorial passo a passo com prints reais (`ui77.js`). O Algory confere o link colado e avisa se ele não é do tipo esperado: CSV de uma aba, documento inteiro em xlsx ou link de compartilhamento.

🔒 No servidor, `gsheet.py`:
- só segue redirecionamentos dentro dos domínios do Google;
- atualiza no máximo a cada 10 minutos;
- busca de novo depois de 6 horas, quando o projeto é aberto.

## 8. Link público do dashboard

- 🔒 O token tem 192 bits e é aleatório; no banco fica só o hash dele.
- Pode ter prazo, senha e ser desligado.
- 🔒 A rota pública devolve só os dados do dashboard (KPIs, série, rankings), nunca o perfil das colunas nem o dono.
- O link não é indexado por buscadores.

**Nomes da equipe** (`pessoas.py`). São as colunas de vendedor, consultor, responsável e afins.
- Na entrada, cada nome vira um ID estável por projeto: `P-3F9A2C`, um HMAC do projeto e do nome.
  - O banco do projeto e a IA só veem o ID.
  - Rankings, filtros e comparações funcionam igual, porque o mesmo nome sempre dá o mesmo ID.
- O dono vê o nome completo. Quando ele escreve "quanto a Ana vendeu?", o nome vira ID antes de ir para a IA.
- Na hora de gerar o link, o dono escolhe o nível:

| Nível | Como o nome aparece |
|---|---|
| 1 · Básico | Abreviado: "Ana S." |
| 2 · Só o primeiro nome | "Ana" |
| 3 · Proteção total (padrão) | Só o ID |

Documentos (CPF, CNPJ, e-mail, telefone) nunca aparecem em nível nenhum: saem antes de gravar.

---

## Telas sem o Algory, de propósito

**Perfil, Plano e tickets, Pagamento.** São dados pessoais e de cobrança. 🔒 O preço é sempre recalculado no servidor, e o painel de administração fica em outro domínio e outro contêiner.

## Telas que ainda não têm o Algory

| Tela | O que falta |
|---|---|
| Tabelas | Configurar pelo chat: "a coluna X é a data da venda", "prazo entre pedido e entrega" |
| Planilha (visualização) | Comentário do Algory sobre o que foi encontrado em cada aba |
| Ajustes do app | Aparência e alertas pelo chat |
| Notificações | Resumo do que mudou |

## Segurança (testado em produção, 13/11/2026)

- Limites do nginx: login responde 429 na 6ª tentativa por minuto, perguntas na 13ª, link público por volta da 6ª.
- 150 chamadas paralelas a /api/health: todas 200.
- Cabeçalho de CSV com HTML/JS: não executa.
- Corrigido (al046/al047): a página principal não mandava os cabeçalhos de segurança (CSP, nosniff, frame) e /conta/eu caía no limite do login.

## Calendário do projeto (al051)

Jornada, folgas, feriados e escalas por projeto; entram nos prazos em dias úteis, no ritmo da meta e na carga horária. Ver `docs/calendario.md`.

## Códigos de ativação e contas em teste (al051)

Prefixo próprio, tipo Tester com teto de tickets e período com data de início. Ver `docs/codigos.md`.
