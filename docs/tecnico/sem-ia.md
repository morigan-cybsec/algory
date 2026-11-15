# Modo sem IA e árvore (al061)

## O problema

Quando os tickets acabavam, o chat respondia uma frase ("Acabaram os tickets do dia…") e a pessoa ficava sem saber o que ainda dava para fazer. Só que a maior parte das perguntas comuns nem usa IA: total, por categoria, por mês, top, comparação e "por que mudou?" são respondidas por regra, direto no banco do projeto, com 0 ticket. Bloquear o chat jogaria isso fora.

## O que acontece agora

Sem tickets (do dia, da semana ou do mês), com o teto de gasto do beta batido ou com a IA desligada:

1. Pergunta que a regra entende é respondida normalmente. Nada muda.
2. Pergunta que precisaria da IA recebe um aviso com o motivo ("A IA chegou no limite de tickets de hoje"), quando os tickets voltam, e **botões**:
   - até 4 perguntas montadas com as métricas e colunas **do próprio projeto** ("Valor por canal", "Top 5 loja por valor"), mais "Quais métricas você conhece?";
   - 3 assuntos de ajuda ("Acabaram os tickets. E agora?", "O que gasta ticket?", "Quais são os planos e os preços?").
3. Cada resposta dada por botão traz "Mais perguntas sem IA" para voltar à lista.

Garantia: uma pergunta só vira botão depois de passar pela mesma leitura que responde a pergunta (`metricas.entender`). Botão não leva a outra recusa. A bateria confere isso no servidor de verdade.

O servidor devolve em `/ask`: `sem_ia` (cota, teto ou ia_off) e `opcoes` (`[{t, tipo}]`). O código está em `app/semia.py`; a tela, em `ui102.js`.

## A árvore

A árvore responde perguntas sobre o próprio Algory com texto fixo, sem modelo.

| | Antes | Agora |
|---|---|---|
| Tópicos | 5 | 13 |
| Como casa | palavra solta em qualquer lugar da frase | a frase inteira; palavra solta só em pergunta de uma ou duas palavras ("tickets?") |
| Textos | alguns desatualizados | reescritos com o que o produto faz hoje |

O defeito que isso corrige apareceu em produção: **"qual o gasto com fornecedor?" recebia a explicação do preço do ticket**, porque a palavra "gasto" bastava. O mesmo valia para "teste", "beta", "exemplo" e "privacidade" dentro de uma pergunta sobre os dados. O teste novo tem 14 perguntas de dados com essas palavras, e nenhuma cai na árvore.

Tópicos: privacidade, como funciona, o que perguntar, tickets, limite, planos, beta, arquivos, sem dados, número errado, conta, compartilhar, integrações. O código está em `app/arvore.py`.

## Acerto da IA (o que a rodada de 16 mostrou)

13 de 16 na al060. As três erradas e o que mudou:

| Pergunta | O que a IA fez | Correção |
|---|---|---|
| "qual o faturamento em outubro?" | Não gerou consulta: disse que faltava coluna de data, e a tabela tinha | A instrução passa a dizer qual é a coluna de data e o intervalo que ela cobre |
| "qual produto vendeu mais?" | Consultou só o nome do produto, sem o valor, e ordenou por quantidade | Pergunta de categoria traz a categoria e o valor; sem medida dita, vale a métrica principal |
| "top 2 produtos por valor" | Acertou os números e escreveu "Tênis" onde o dado diz "Tenis" | A resposta copia o nome como está no dado; a conferência do gabarito ignora acento |

A hipótese da al060 (o classificador tirava "tabela" das fontes) se confirmou em parte: as respostas sem consulta caíram de 2 em 8 para 1 em 16, e a que sobrou tinha outra causa (a primeira linha da tabela acima).
