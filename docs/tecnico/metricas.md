# Métricas com nome, investigação em passos e gabarito (al056)

Três peças que fecham a distância para as ferramentas grandes de análise com IA sem aumentar o custo por pergunta. As três rodam por regra, no banco do projeto, sem chamar modelo.

## 1. Métricas com nome (`app/metricas.py`)

**O problema.** Quando o modelo escreve a consulta do zero a cada pergunta, a mesma pergunta pode dar consultas diferentes, e "ticket médio" pode virar três contas diferentes.

**O que faço.** Cada projeto tem um catálogo de métricas. Ele não é digitado em lugar novo: sai do que o cliente já confirmou.

| Origem | De onde vem | Exemplo |
|---|---|---|
| Coluna | Coluna de número com o papel confirmado em Tabelas (somar ou tirar média) | `valor` = `sum("valor")` |
| Conta | Conta guardada na tela de Tabelas | `ticket médio` = `sum("valor") / count(*)` |
| Contagem | Linhas da tabela | "quantas linhas", "quantos registros" |

Cada métrica aceita outros nomes: o **significado** que o cliente escreveu para a coluna e os sinônimos de negócio do `vocab.py` ("faturamento", "quanto vendemos" → coluna `valor`).

**Como uma pergunta é lida** (`entender`). A regra procura, nesta ordem: métrica, série no tempo, período, top N e ordem, filtros (valores de categoria citados), agrupamento. O que foi entendido sai da frase. **Se sobrar uma palavra que a regra não conhece, ela não responde** e a pergunta segue para a IA.

| Pergunta | Resultado |
|---|---|
| "qual o faturamento em outubro?" | regra: soma de `valor` em outubro de 2026 |
| "qual loja vendeu mais?" | regra: ranking por loja |
| "quanto vendemos este mês no Centro?" | regra: período + filtro |
| "valor total **sem contar** o Centro" | recusa ("sem", "contar") → IA |
| "**compare** outubro com setembro" | recusa → IA |
| "qual o custo **e** o valor?" | duas métricas com a mesma força: ambíguo → IA |

Regras de segurança da leitura:

- Empate entre duas métricas ou duas categorias: não escolhe.
- Valor de categoria com nome de coluna (um canal chamado "Loja") não vira filtro escondido.
- Mês sem ano: o ano vem da última data dos dados, e a resposta diz qual ano usou.
- Só responde em português. Pergunta em inglês ou espanhol vai para a IA, que confere o idioma.
- O texto do cliente nunca entra na consulta. Entram nomes de coluna entre aspas e valores que **existem no banco**, com aspas dobradas.

**Onde entra no trâmite.** Em `pipeline.ask`, antes da árvore de respostas prontas e antes de qualquer modelo. Custa 0 ticket e funciona com a IA desligada ou sem saldo. A resposta sai no formato que a tela já desenha (número, ranking, série), com o botão "Como foi calculado" mostrando a consulta e "IA usada: não".

**Para o modelo.** Quando a pergunta vai para a IA, o gerador de consulta recebe o catálogo (`MÉTRICA vendas: lucro = (sum("valor") - sum("custo"))`) e usa a expressão pronta em vez de inventar a conta.

**Rota:** `GET /projetos/{pid}/metricas` devolve o catálogo do projeto (só para o dono).

## 2. Investigação em passos (`app/investigar.py`)

**O problema.** "Por que as vendas caíram?" é a pergunta que mais custa para uma IA: um agente livre faz dez ou vinte chamadas, cada uma paga, e pode concluir o que os dados não mostram.

**O que faço.** Passos fixos, todos consultas ao banco:

1. Compara dois períodos iguais. Sem período na pergunta: o mês da última data, até o último dia com dados, contra os mesmos dias do mês anterior. Não compara mês pela metade com mês cheio.
2. Separa a diferença em volume (quantas linhas) e valor médio por linha.
3. Em cada categoria (até 4), mede quanto cada valor mudou. Escolhe a categoria em que um único valor explica mais perto de 100% da diferença.
4. Dentro desse valor, repete a comparação nas outras categorias (um nível abaixo).

Teto: `MAX_CONSULTAS = 12`. Em geral usa de 5 a 9. Custa 0 ticket.

A resposta termina com um aviso fixo: **isto mostra onde a mudança aconteceu, não a causa.** A regra não inventa motivo.

Gatilhos: "por que … caiu/subiu/mudou", "o que aconteceu com …", "investiga …". O resto da frase passa pela mesma leitura das métricas, com a mesma exigência de entender tudo.

## 3. Gabarito (`app/gabarito.py`)

**O problema.** Sem perguntas de resposta conhecida, não dá para saber se trocar de modelo ou de instrução melhorou ou piorou.

**O que faço.**

- **Dados fixos:** 833 vendas de agosto a 13 de novembro de 2026, geradas sem acaso. Há uma queda plantada (loja Norte, produto Tenis, em novembro) e uma armadilha (um canal chamado "Loja").
- **Resposta esperada calculada em Python puro**, sem passar pelo banco nem pelo código medido.
- **49 perguntas:** 39 que a regra tem de acertar (total, sinônimo, filtro, período, ranking, top, média, conta, série, investigação) e 10 que ela tem de recusar.

| Modo | O que mede | Custo | Onde roda |
|---|---|---|---|
| `regra` | A regra acerta tudo que é dela e recusa o resto | 0 | Testes e bateria |
| `ia` | A IA de verdade, com a regra desligada, numa amostra | Chamadas reais | Bateria (amostra de 8) ou à mão |

```
docker compose exec api python -m app.gabarito          # regra
docker compose exec api python -m app.gabarito ia 20    # 20 perguntas pela IA
```

Cada rodada fica em `data/gabarito.jsonl` (modo, acertos, total, tickets, tempo, modelos, o que errou). O painel de administração lê em `GET /adm/api/gabarito`. Antes de trocar um modelo: rode, anote, troque, rode de novo.

O gasto da rodada de IA na bateria não entra no livro de tickets (usa um registro em memória, porque a conta "gabarito" não existe no banco). É da ordem de centavos por rodada.

## O que o gabarito já achou

- **"quantidade" era tratada como média.** A regra que reconhece taxas tinha `idade` solto, e "quant**idade**" casava. O dashboard mostrava a média da coluna em vez da soma. Corrigido em `painel.py`.
- **"qual o ticket médio?" caía na resposta pronta sobre tickets do plano**, mesmo em projeto com a conta "ticket médio" guardada. As métricas agora vêm antes da árvore.

## Ajustes da al057, puxados pelo gabarito

- **A IA errou 3 de 8 na primeira rodada em produção**, todas em perguntas com "loja". O gabarito tem um canal chamado "Loja", e a suspeita é que o gerador de consulta filtrou `canal = 'Loja'`. Agora ele recebe um aviso quando um valor de categoria tem nome de coluna, e a rodada guarda a consulta que a IA gerou em cada erro, para não precisar adivinhar.
- **Mês pela metade no dashboard.** O cartão comparava novembro até o dia 13 com outubro inteiro. Agora compara com os mesmos dias do mês anterior e diz isso ("vs mês anterior até o dia 13").
- **Decimal pequeno virava porcentagem.** No texto da IA, média de 1,5 item saía como "150,0%". Agora só vira porcentagem quando o nome da coluna é de taxa.

## al058: a regra fora do exemplo "loja e valor"

Testei a regra em cinco planilhas de ramos diferentes (clínica, delivery, escola, uma em inglês e estoque), com cabeçalhos do jeito que o cliente escreve: `Data do Atendimento`, `dt_pedido`, `Vlr Total`, `Custo Unit.`, `Product Category`. Na primeira passada ela respondeu 40 de 62 perguntas e **errou duas com cara de certeza**. O que mudou:

| O que aparecia | Causa | Correção |
|---|---|---|
| "quantos pedidos?" → **R$ 18.241,50** | A coluna `total_pedido` herdava os sinônimos de "pedido" | Palavra solta só vale como sinônimo se for a primeira do nome da medida. Com "quantos", a contagem tem preferência, e coluna em dinheiro é recusada |
| "taxa de entrega total" → **7,0%** | Todo nome com "taxa" virava porcentagem e média | Taxa de entrega, serviço, frete, matrícula etc. é valor em reais que se soma |
| "qual procedimento rendeu mais?" → sem resposta | Só entendia "vendeu" e "faturou" | Verbos de dinheiro (rendeu, pagou, cobrou, recebeu) apontam para a medida principal em dinheiro da tabela, quando ela é uma só |
| "qual bairro pede mais?" → sem resposta | Não havia contagem por verbo | Verbos de contagem (pede, atende, compra) contam linhas. O nome do evento vem da coluna de data: `dt_pedido` → "quantidade de pedidos" |
| "mensalidade dos atrasados" → sem resposta | O valor era "Atrasado", no singular | Filtro aceita o plural do valor |
| "qtd de Saída" → sem resposta | "saída" é sinônimo de custo no vocabulário e bloqueava o filtro | Só o nome real da coluna bloqueia um valor, não os sinônimos |
| Planilha em inglês não entendia "faturamento" | Vocabulário só em português | `revenue`, `sales`, `cost`, `units`, `region`, `product`, `category`… |
| "avaliação" somada | Faltava no reconhecimento de notas | Avaliação, rating, satisfação e duração são médias |

Depois: 58 de 62 respondidas, e as 4 que sobram são recusas certas (ambíguas ou com palavra que a regra não conhece). As cinco planilhas viraram teste fixo (`gabarito.variantes`, 36 casos com a consulta esperada escrita à mão) e rodam na bateria do servidor.

### O respondedor de reserva foi desligado

O teste em produção mostrou o pior caso: com os tickets do dia esgotados, um respondedor antigo "de reserva" assumia. Quando não entendia a pergunta, ele pegava a maior tabela e a primeira coluna de número. "Qual a duração média?" saiu como "Média de total pedido: R$ 60,80". Ele foi desligado. Sem tickets, quem responde é a regra das métricas, que só responde quando entende a frase inteira; o resto recebe o aviso de que os tickets acabaram.

### Dois ajustes no caminho da IA

- **Número no rótulo não é número inventado.** "O Açaí 500ml faturou…" era marcado como resposta com número sem origem, por causa do "500". A conferência agora aceita números que já estão na pergunta ou nos rótulos do resultado.
- **Centavos.** Valor quebrado acima de mil perdia os centavos ("6.730" em vez de "6.730,50").

### Limite que continua

Projeto com duas tabelas em que a mesma pergunta serve para as duas ("quanto vendemos?") é ambíguo: a regra recusa e a IA assume.

## O que não foi feito, e por quê

| Item | Motivo |
|---|---|
| Executar código gerado (Python) | Pede uma área isolada por usuário, com limite de processador, memória e rede. Num servidor único com pagamento ao lado, é o maior risco que eu poderia abrir. |
| Conector de banco do cliente | **Feito na al057**, como cópia única sem guardar senha. Ver `docs/conector.md`. MySQL continua de fora. |
| Agente livre com IA | Sem teto de passos, o custo por pergunta deixa de ser previsível no plano de R$ 25. A investigação por regra cobre o caso mais pedido. |
| Tela para criar métrica | O catálogo usa o que já existe (papel, significado, contas). Desde a al057 dá para perguntar no chat "quais métricas você conhece?" e ver o catálogo. Uma tela de edição fica para depois do lançamento. |
