# Como a árvore trabalha (al062)

A árvore é a parte do chat que responde sobre o próprio Algory (planos, tickets, privacidade, arquivos, conta) com texto fixo. Não usa modelo de IA e não gasta ticket. O código está em `app/arvore.py`.

## A ordem de uma pergunta no chat

1. **Regra dos dados.** Num projeto com tabela, tenta primeiro responder com os dados ("valor por loja"). Se entender a frase inteira, responde e acabou.
2. **Árvore.** Três tentativas, da mais certa para a mais solta:
   - **Título:** a pergunta é igual ao título de um tópico (é o que os botões mandam).
   - **Frase:** a pergunta contém uma das frases do tópico ("quanto custa um ticket", "como excluo minha conta").
   - **Palavras-chave:** cada tópico tem uma lista de radicais ("cancel", "senha", "assinatura", "webhook"). Cada palavra da pergunta que bate vale um ponto para aquele tópico.
3. **IA**, se ninguém acima respondeu e há tickets. Sem tickets, entra o [modo sem IA](sem-ia.md).

## A busca por palavras-chave

| Situação | O que a árvore faz |
|---|---|
| Um tópico fica na frente sozinho | Responde com ele |
| Dois ou três tópicos empatam ("tickets plano", "problema no upload") | **Pergunta:** "Encontrei mais de um assunto. Qual deles?", com um botão por tópico |
| Mais de três empatam, ou as chaves cobrem menos de 3/4 das palavras da pergunta | Não responde: segue para a IA |
| Alguma palavra da pergunta é nome de coluna do projeto | Não responde: é pergunta de dados |

As duas últimas linhas são a trava. Sem elas, palavra-chave vira o defeito que a al061 corrigiu: "qual o gasto com fornecedor?" recebia a explicação do preço do ticket. Com a regra dos 3/4, "vendas por cliente", "problema por loja" e "total de erros" continuam fora (12 casos no teste). E num projeto que tem uma coluna "erros", a palavra "erros" sozinha não vai para o tópico de suporte.

Palavras sem conteúdo ("como", "quero", "posso", "meu") não contam na cobertura.

## Como acrescentar um tópico

Em `TOPICOS`: id, título (vira botão), frases, texto. Em `CHAVES`: os radicais. O teste `test_todo_botao_da_arvore_tem_resposta` garante que todo título tem resposta.

## O que ainda fica com a IA

Pergunta sobre o Algory escrita de um jeito que nenhuma frase ou chave cobre, e toda pergunta de dados que a regra não entende. O painel mostra as perguntas que foram para a IA sem o texto delas (o texto não é guardado), então a lista de frases cresce pelos chamados e pelos testes, não por espionagem do chat.

## Chat geral (al063)

Fora de projeto, o chat tinha uma árvore própria, na tela, mais antiga: casava por palavra solta ("vendas", "plano", "teste") e tinha texto velho ("a tabela sai em 24 h", "300 tickets do beta"). Quem escrevia "minhas vendas caíram" recebia o convite para o projeto de exemplo.

Agora a pergunta do chat geral vai primeiro para a árvore do servidor, pela rota `POST /arvore`. Ela devolve só texto fixo sobre o produto: não pede conta, não chama IA e não gasta ticket. Se responde, a tela mostra a resposta com o botão da tela certa (planos, uso, Ajuda e chamados). Se fica em dúvida, pergunta. Se não é com ela, segue o caminho de antes. Da árvore antiga ficaram só as entradas que abrem coisas da tela (projeto de exemplo, fontes de dados).

## Rede de segurança da tela (al063)

Antes de mexer no esqueleto da tela há agora um teste que abre as 21 telas e abas (início, conversas, repertório, integrações, notificações, uso, planos, perfil e suas abas, ajustes, ajuda, sobre, e dentro do projeto: dashboard, dados, calendário, compartilhar, versões, tabelas, anexar) em 1280 e 390 px. Para cada uma ele guarda o tamanho do texto, o número de botões, se houve erro de script e se a página estourou a largura. O resultado fica como base: qualquer mudança na tela é comparada com ela.
