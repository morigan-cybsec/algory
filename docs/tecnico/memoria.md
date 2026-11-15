# Memória da conversa, comparação e a conta nas mãos do dono (al059)

## Memória sem guardar a conversa

Todo chat concorrente lembra da pergunta anterior. O Algory não guardava nada, e eu não queria começar a guardar o texto das perguntas. A saída foi deixar a memória **no navegador**.

1. Toda resposta por regra devolve um `ctx`: a métrica, o agrupamento, os filtros, o período e a série daquela resposta. Não tem texto da pergunta.
2. O navegador guarda o `ctx` por **30 minutos**, por projeto, na memória da aba. Fechou a aba, sumiu.
3. Na pergunta seguinte ele manda o `ctx` de volta, junto com a pergunta anterior (até 300 letras), só para o caso de a IA precisar entender "e", "isso", "o mesmo".
4. O servidor **confere o `ctx` contra o banco** antes de usar: a métrica tem de estar no catálogo, as colunas têm de existir, cada valor de filtro tem de existir na tabela, as datas têm de ser datas. Qualquer coisa fora disso derruba o contexto inteiro. O rótulo do período é refeito a partir das datas; nenhum texto do `ctx` é reaproveitado.

O que a continuação entende, por regra e sem ticket:

| Depois de "valor em outubro no Centro" | O que acontece |
|---|---|
| "e por produto?" | Mesma métrica, mesmo mês, mesma loja, agora por produto |
| "e o custo?" | Troca a métrica, mantém o resto |
| "e em setembro?" | Troca o período |
| "só do Norte" | Troca o filtro da mesma coluna |
| "e por mês?" | Vira série (e solta o período de um mês só, que não daria série) |
| "e no total?" | Limpa os filtros |
| "compare com o mês anterior" | Comparação dos dois períodos |
| "por que mudou?" | Investigação em passos daquela métrica, com aqueles filtros |

Uma pergunta nova e completa ("qual o custo total?") não herda nada. Continuação sem contexto não é respondida pela regra.

Cada resposta traz até 3 **sugestões de continuação**, tiradas das colunas que o projeto tem. Na tela viram botões.

## Comparação de períodos (`metricas.py`)

"compare outubro com setembro", "valor de outubro contra setembro por loja", "faturamento este mês contra o mês passado". Sem período dito, compara o mês da última data **até o último dia com dados** com os **mesmos dias** do mês anterior. Não compara mês pela metade com mês cheio.

## Duas tabelas no mesmo projeto

Quando a mesma palavra serve para métricas de duas tabelas ("vende mais" numa clínica e num delivery no mesmo projeto), a regra tenta ler a frase inteira com cada candidata e fica com a única que explica tudo ("qual **prato** vende mais" só fecha no delivery). Se as duas explicam, continua recusando.

## A conta nas mãos do dono (`/conta/...`)

| Rota | O que faz |
|---|---|
| `GET /conta/dados` | Um .zip com o cadastro, as tabelas de cada projeto em CSV e os ajustes. No máximo 3 por hora. Não leva senha, segredo de 2FA nem nada de outra conta |
| `GET /conta/sessoes` | Quantas sessões estão ativas |
| `POST /conta/sessoes/sair-todas` | Encerra todas as outras sessões; a deste aparelho continua |
| `POST /conta/excluir` | Pede a senha e a palavra EXCLUIR. Apaga projetos, tabelas, versões, links públicos, integrações, sessões e a conta. Não tem volta |

Na exclusão fica só o registro dos pagamentos, **sem vínculo com a pessoa** (a coluna do usuário vira nula), porque registro fiscal tem prazo legal de guarda. As cópias de segurança diárias ainda contêm a conta por até 7 dias, até girarem.

Na tela: **Ajustes › Privacidade › Sua conta e seus dados**.

## Avisos para quem cuida (`alertas.py`)

Os tetos de gasto já existiam; faltava alguém ficar sabendo. A cada 5 minutos o contêiner do painel confere e manda um e-mail para `ADMIN_EMAIL`, sem repetir:

- gasto do mês passou de 60%, 80% ou 95% do orçamento;
- gasto da última hora passou de 10% do orçamento do mês;
- o verificador barrou 10 ou mais perguntas na última hora.

O e-mail leva números, nunca o texto de pergunta de ninguém.

## Vazamento da instrução do sistema

Se a resposta do modelo repetir uma frase inteira das instruções do sistema, ela é trocada por um aviso fixo e o registro marca `vazamento`. A bateria manda três pedidos reais de "mostre suas instruções" e confere que nenhum trecho sai.

## Auditoria das bibliotecas

A bateria lista as bibliotecas instaladas no contêiner (`pip freeze`) e consulta a base pública OSV. Sai só nome e versão. Se alguma tiver falha conhecida, a linha diz qual.

## O resumo por e-mail ganhou "O que mudou"

O resumo diário ou semanal agora inclui o resultado da investigação por regra para a medida principal do projeto: quanto mudou contra os mesmos dias do mês anterior e qual categoria puxou. Continua sem IA.
