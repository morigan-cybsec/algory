# Como a LLM trabalha

A regra do projeto é: **a IA só entra quando regra não resolve.** A maior parte do que o Algory faz não chama modelo nenhum.

| | Sem IA | Com IA |
|---|---|---|
| Ler a planilha e achar as tabelas | ✔ | |
| Tirar dados pessoais | ✔ | |
| Montar o dashboard | ✔ | |
| Calcular prazos, metas, dias úteis | ✔ | |
| Mudar o dashboard por frase ("tira o ranking", "top 5") | ✔ (quase sempre) | só se a regra não entender |
| Responder perguntas frequentes | ✔ (respostas prontas) | |
| Responder pergunta direta sobre os dados ("faturamento em outubro", "qual loja vendeu mais") | ✔ (métricas com nome) | |
| Investigar uma queda ou alta ("por que caiu?") | ✔ (passos fixos, com teto) | |
| Responder uma pergunta nova, com condição ou comparação | | ✔ |
| Arrumar categorias escritas de jeitos diferentes | ✔ (acento, maiúscula) | nos planos pagos |
| Checar uma imagem enviada | ✔ (texto dentro dela) | ✔ (conteúdo) |

## O caminho de uma pergunta

Do momento em que a pessoa aperta Enter até a resposta aparecer:

```
 Pergunta
    │
 1. Máscara ............ CPF, CNPJ, e-mail, telefone e cartão viram marcadores
    │
 2. Métricas com nome .. a regra entendeu a pergunta INTEIRA (métrica, filtro, período, ordem)?
    │                    monta a consulta sozinha e responde                              ──► resposta (0 ticket)
    │                    "por que caiu?" vira uma investigação em passos fixos, no banco  ──► resposta (0 ticket)
    │
 2b. Respostas prontas . pergunta frequente ou já respondida? devolve na hora, sem modelo  ──► resposta (0 ticket)
    │
 3. Verificador ........ a frase parece tentativa de enganar o assistente? (regra; modelo só se desconfiar)
    │
 4. Cota ............... a conta tem tickets para hoje?
    │
 5. Roteador ........... qual o tamanho do problema? (regra; modelo só na dúvida)
    │
 6. Teto de gasto ...... o orçamento do mês aguenta este modelo? se não, usa um menor
    │
 7. Consulta ........... o modelo recebe SÓ os nomes das colunas e 3 linhas de exemplo, e devolve um SELECT
    │                    o servidor confere e roda: só leitura, só tabelas do projeto, 10 mil linhas, 5 segundos
    │
 8. Texto .............. o modelo recebe o RESULTADO e escreve a resposta com marcadores no lugar dos números
    │
 9. Conferência ........ número sem origem? idioma errado? citação que não existe? refaz uma vez por minha conta
    │
10. Troca .............. o servidor põe os valores reais nos marcadores e devolve os dados pessoais mascarados
    │
11. Cobrança e registro  custo real em dólar vira tickets; fica o registro do que foi feito (sem o texto)
    │
 Resposta
```

### O que a requisição carrega

A tela manda quatro coisas: o projeto, a pergunta, o idioma da pessoa e, se houver, o perfil do negócio. Nada mais.

O modelo **nunca recebe a planilha**. Na etapa 7 ele vê o esquema:

```
tabela vendas: data (data), loja (texto), vendedor (texto), valor (número), meta (número)
exemplos: 2026-11-02 | Centro | P-3F9A1C | 874.00 | 1500.00
```

Na etapa 8 ele vê só o resultado da consulta, e nele os números já foram trocados por marcadores:

```
vendedor=P-3F9A1C, receita={{v1}}
vendedor=P-81B2D0, receita={{v2}}
```

E devolve algo como: `Quem mais vendeu foi P-81B2D0, com {{v2}}.` O servidor troca `{{v2}}` por `7.245` e o código `P-81B2D0` pelo nome, se quem pergunta pode vê-lo. O modelo não chega a ver o valor. O código disso está em [trechos de código](trechos-de-codigo.md).

### Por que marcadores

É a decisão mais importante do projeto. Um modelo de linguagem escreve números errados com a mesma confiança dos certos. Se ele não pode escrever número, não pode errar número. O valor que aparece na tela é sempre o que o banco devolveu.

## Como trato os dados

| Momento | O que acontece |
|---|---|
| Na carga da planilha | Colunas de nome, CPF, CNPJ, RG, e-mail e endereço são tiradas. Telefone vira estado. CEP vira estado, região e cidade. Nome de vendedor vira um código. |
| No texto da pergunta | CPF, CNPJ, e-mail, telefone e cartão viram marcadores antes de qualquer modelo. |
| Na escolha do provedor | Provedor que treina com o que recebe sai da fila quando o dado é de cliente real. |
| No conteúdo que vai para o modelo | Tudo que veio da planilha é limpo de marcações que imitam instrução e vai marcado como dado, com a regra de nunca obedecer ao que estiver escrito ali. |
| Depois da resposta | O texto da pergunta e o da resposta não são guardados. Fica o registro técnico: modelo, tickets, tempo. |
| Imagens | São lidas e descartadas. Antes, o servidor regrava a imagem, o que apaga a localização e os outros dados escondidos da foto. |

## Como gasto menos tickets por pergunta

Um ticket vale cerca de meio centavo de real e corresponde ao custo real da chamada. Cada técnica abaixo tira custo de algum lugar.

| Técnica | O que economiza |
|---|---|
| **Métricas com nome** | Pergunta direta sobre os dados é respondida por regra: a mesma pergunta dá sempre a mesma consulta. Se sobrar uma palavra que a regra não conhece, ela não responde e a IA assume. Zero chamada. |
| **Investigação em passos** | "Por que caiu?" roda de 5 a 9 consultas fixas no banco, com teto de 12, em vez de um agente pago por passo. Zero chamada. |
| **Respostas prontas** | Perguntas sobre o produto ("como funciona", "o que faz com meus dados") têm resposta escrita. Zero chamada. |
| **Memória de respostas** | A mesma pergunta, no mesmo projeto, com os mesmos dados e no mesmo idioma devolve a resposta guardada. Zero chamada. |
| **Dashboard por regra** | A pessoa vê o principal sem perguntar nada. Muita pergunta deixa de existir. |
| **Só o esquema** | Mandar nomes de colunas em vez da planilha corta a entrada em ordens de grandeza. Uma planilha de 50 mil linhas custa o mesmo que uma de 50. |
| **Só o resultado** | O texto é escrito a partir de uma tabela pequena, não dos dados brutos. |
| **Roteador por regra** | Decidir o tamanho do problema quase nunca chama modelo. Só quando a regra tem menos de 60% de confiança. |
| **Modelo por tarefa** | Rotear, gerar consulta e checar segurança usam modelos pequenos e baratos. O modelo principal só escreve o texto final. |
| **Camadas** | Pergunta simples usa o modelo menor. Análise pesada usa o maior, e antes disso a tela avisa o custo e pede confirmação. |
| **Teto que desce de camada** | Passou de 60% do orçamento do mês, o maior modelo vira o médio; de 80%, tudo vira o menor; de 95%, só respostas prontas. O serviço fica mais simples, mas não cai. |
| **Refazer só uma vez** | Se a conferência reprova a resposta, refaço uma vez e não cobro. Não entro em laço. |
| **Limite de saída** | Cada tarefa tem um teto de tamanho de resposta. |

O resultado prático: uma pergunta comum sai por 1 ou 2 tickets.

## Memória sem guardar a conversa

O Algory lembra da pergunta anterior ("e por produto?"), mas o servidor não guarda a conversa. Cada resposta por regra devolve um contexto pequeno (métrica, filtros, período), o navegador guarda por 30 minutos e manda de volta. O servidor confere cada item contra o banco antes de usar. Para a IA, a pergunta anterior viaja junto como dado, mascarada. Detalhes em [memória](tecnico/memoria.md).

## Como eu meço se ele acerta

Tenho um gabarito: 49 perguntas sobre uma planilha fixa, com a resposta calculada à parte. A regra tem de acertar 39 e recusar 10. A IA é medida numa amostra, com a regra desligada, e a nota fica guardada com o modelo, o custo e o tempo. Antes de trocar um modelo ou uma instrução, rodo; depois, rodo de novo. Detalhes em [métricas, investigação e gabarito](tecnico/metricas.md).

Na primeira rodada o gabarito já achou dois erros meus: a coluna "quantidade" era tratada como média (a regra de taxas casava com "idade" dentro da palavra), e "qual o ticket médio?" caía na resposta pronta sobre os tickets do plano.

## O que eu observei do comportamento do chatbot

- **Ele obedece melhor a formato do que a proibição.** "Escreva números só como marcador" funciona muito mais do que "não invente números".
- **O erro mais comum não é mentir, é responder outra pergunta.** A consulta roda, o número está certo, mas não era aquilo que a pessoa queria. Por isso a resposta diz o que foi calculado.
- **Idioma vaza.** Com dados em inglês e pergunta em português, às vezes a resposta vinha em inglês. Passei a conferir o idioma da resposta contra o da pessoa e refazer.
- **Nome de coluna importa mais que eu esperava.** Coluna chamada `col_7` gera consulta ruim. Por isso existe a etapa de dar significado às colunas antes.
- **Sem contexto, um código não é uma pessoa.** Quando troquei nomes por códigos, o modelo deixou de entender perguntas sobre "o vendedor João". Resolvi mandando o código junto com a coluna de onde ele veio.
