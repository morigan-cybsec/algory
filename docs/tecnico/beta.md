# Beta, chamados de ajuda, operação e planilhas reais (al060)

## O Algory se apresenta como beta

- Um selo **BETA** fica na barra do topo, em todas as telas.
- Em **Perfil › Plano e tickets** há um cartão "Algory em beta": o projeto é mantido com recursos próprios; assinar qualquer plano ajuda a pagar o servidor e a IA, mas não é obrigação. Os valores são os da fase beta e **podem mudar**, mas o que foi pago é honrado: pagou um mês, tem um mês, e problema com pagamento se resolve por chamado ou em mail@osmio.tech. Não há mensalidade nem cobrança automática (o pagamento é por Pix, por período). O cartão também diz o que é acompanhado para melhorar o sistema: números de uso, não o conteúdo dos dados. Quem testar e apoiar agora será reconhecido com vantagens se o projeto for adiante. Não prometo nenhuma vantagem específica, porque ainda não sei quais serão.

## Chamados de ajuda

O "Fale conosco" virou **Ajuda e chamados**. A diferença: antes a mensagem saía e a resposta só chegava por e-mail. Agora cada mensagem é um chamado com número, tipo e situação, e a resposta aparece no próprio app.

| Lado | O que faz |
|---|---|
| Pessoa | Abre um chamado (dúvida, problema, reclamação ou sugestão). Logada, vê em "Meus chamados" a situação (aberto, respondido, fechado) e a resposta |
| Eu, no painel | Aba Mensagens: leio, respondo ali mesmo e posso fechar. A resposta vai para o app e por e-mail |

- `POST /contato` aceita `tipo`. `GET /contato/meus` devolve só os chamados da conta que pergunta.
- `POST /adm/api/mensagens/{id}/responder` grava a resposta e avisa por e-mail.
- Continua valendo: sem login dá para escrever (com e-mail), no máximo 3 por hora por endereço, e o campo-isca contra robô.
- Migração `013_chamados.sql`: colunas `tipo`, `status`, `resposta`, `respondido_em`.

## Operação

| O que | Como |
|---|---|
| A API caiu | A cada 5 minutos o contêiner do painel chama `/health`. Duas falhas seguidas (10 minutos) viram e-mail "Algory FORA DO AR". Quando volta, outro e-mail |
| Disco enchendo | Menos de 15% livre no disco dos dados vira um e-mail por dia |
| Biblioteca com falha | A auditoria da al059 apontou o Starlette 0.46.2 com duas falhas conhecidas (negação de serviço). O FastAPI subiu de 0.115 para 0.162, que traz o Starlette 1.x. Os 251 testes passam nas versões novas |

**Ainda falta:** um ambiente de homologação (hoje cada correção vai direto para produção, com backup antes e bateria depois) e o destino da cópia de segurança externa.

## Planilhas reais

Até a al059 a regra só tinha visto dados que eu gerei. Baixei sete planilhas públicas (vendas de uma loja com 8.399 linhas, supermercado, corridas de táxi, anúncios de aluguel com 32.960 linhas, gorjetas, voos, vendas por vendedor) e subi pelo caminho normal.

| O que apareceu | Efeito | Correção |
|---|---|---|
| Arquivo com fim de linha antigo (só `\r`) | **Erro 500 no envio** | O texto é normalizado antes de ler |
| Primeira coluna sem título (índice exportado) | **Erro 500 no envio** | A coluna ganha o nome `coluna_1` |
| CSV em cp1252 (Excel no Windows) | "Região" virava "Regi�o" | Tenta UTF-8; se não for, lê como cp1252 |
| Datas no formato americano (mês/dia/ano) | A coluna não era reconhecida como data: sem período, sem série, sem investigação | O formato americano entra depois do brasileiro na detecção |
| Coluna `year` em número | Era **somada** como medida | Ano, mês, year, month são agrupamento |
| Categoria com muitos valores (162 bairros, 40 clientes) | Não dava para perguntar "qual bairro…" | Numa pergunta, texto que se repete vale como agrupamento e filtro (até 5.000 valores) |
| Tabela com colunas `total` e `fare` | "total de fare" ficava sem resposta | Se a métrica mais forte não fecha a frase, a regra tenta a seguinte |
| "qual o ticket médio?" sem conta guardada | Caía na resposta pronta sobre tickets do plano | Ticket médio = medida principal ÷ linhas, quando há uma medida principal |
| "qual pagamento é mais usado?" | Sem resposta | "mais usado / comum / frequente" conta linhas |

Antes das correções: 2 arquivos de 7 nem subiam e 41 de 60 perguntas tinham resposta. Depois: os 7 sobem e 52 de 60 têm resposta por regra. As 8 restantes são recusas (vão para a IA).

Os arquivos originais não estão no repositório. Cada problema virou um caso pequeno em `tests/test_reais.py`.

**O que isto não é:** planilha de cliente. Dado público é limpo demais. O teste que falta é com 10 a 20 planilhas de quem vai usar, e é para isso que servem o selo beta e os chamados.

## Acerto da IA

A rodada da al059 em produção deu 6 de 8, e as duas erradas saíram **sem consulta nenhuma**. A causa: quando a regra de classificação tinha pouca certeza, um modelo pequeno decidia as fontes da pergunta e às vezes tirava "tabela" da lista. A resposta era escrita sem dado. Agora projeto com tabela sempre tenta a consulta. A amostra da bateria subiu de 8 para 16 perguntas, e cada erro guarda a consulta e o começo da resposta.

## Planilhas bagunçadas (al062)

Depois das planilhas públicas, montei cinco arquivos do jeito que uma pequena empresa exporta do Excel e subi pelo caminho normal.

| O que o arquivo tinha | O que acontecia | Agora |
|---|---|---|
| Linha "TOTAL" no fim do CSV | Entrava como venda: **o total do chat saía em dobro** (R$ 276.750 em vez de R$ 138.375), e "por loja" dava o valor certo, então os dois números não batiam | A linha sai na leitura e a tela avisa |
| Título e data de emissão acima do cabeçalho | O título virava o nome da coluna e a tabela ficava com 2 colunas | O cabeçalho é a primeira linha larga; as de cima são puladas |
| Milhar com ponto ("12.345") | Lido como 12,345: total de 154,74 em vez de 154.740 | Em CSV com ponto e vírgula, coluna em que todo valor tem esse formato é milhar |
| Negativo entre parênteses, "12,5%" | Viravam texto | Viram número |
| "-" e "N/A" onde não há valor | A coluna deixava de ser medida | Viram vazio |
| Duas colunas "Valor" | A segunda apagava a primeira | A segunda vira "Valor 2" |
| Excel com subtotais e linha em branco no meio | Três quadros para a pessoa escolher, dois sem cabeçalho | Uma tabela só, sem os subtotais |

O que foi arrumado aparece num aviso depois do envio ("Arrumei na leitura: tirei 1 linha de total…"). CSV limpo não é mexido (há teste para isso). O código está em `app/csvreal.py`.

## Envios e links (al062)

- **Link do Google.** O endereço que a pessoa cola nunca é buscado. Dele sai só o código da planilha, e o endereço é montado por mim em docs.google.com. Redirecionamento só para domínio do Google, por HTTPS. A partir da al062 o conteúdo baixado passa pela mesma portaria do envio direto.
- **Conectores de API externa** ficam para depois do lançamento. Cada um abre uma superfície nova (credencial guardada, endereço escolhido pelo cliente), e o PostgreSQL já mostrou o trabalho que dá fazer direito.
