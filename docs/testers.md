# Para quem vai testar o Algory

Obrigado por topar. O Algory está em **beta**: funciona e está no ar, mas ainda está sendo testado com dados de verdade. O que você achar, bom ou ruim, muda o produto.

Esta página tem tudo o que você precisa: como ativar o código, o que testar e como me avisar de um problema.

## 1. Ativar o código

1. Abra [algory.osmio.tech](https://algory.osmio.tech) e crie a conta, com e-mail e senha. Chega um código de 6 dígitos no seu e-mail para confirmar.
2. Vá em **Perfil › Plano e pagamentos › Resgatar código**.
3. Cole o código que você recebeu (formato `XXXXXX-XXXX-XXXX-XXXX`). A tela mostra o que ele libera antes de você confirmar.
4. Confirme. O teste começa na hora. Se o código tiver data de início, a tela avisa o dia.

Não achou o e-mail de confirmação? Olhe a caixa de spam.

## 2. O que o código libera

- Vale por **5 dias** a partir da ativação (pode variar; a tela mostra).
- Tem um **total de tickets para o período todo**, sem limite por dia. Use como quiser.
- Quando acaba, a conta volta para o plano Grátis. **Nada é apagado.**
- Não é preciso pagar nada para testar. Se você assinar um plano durante o teste, o plano pago começa na hora.

Ticket é o que a IA consome. O dashboard, os filtros e as perguntas diretas (total, por categoria, por mês) não gastam nenhum.

## 3. Os primeiros dez minutos

1. **Suba uma planilha de verdade**, em Excel (.xlsx) ou CSV, do jeito que ela está, bagunçada mesmo. Também serve uma Planilha do Google publicada na web. Sem planilha, a tela inicial oferece um projeto de exemplo.
2. **Confira como as colunas foram lidas.** O Algory mostra o que entendeu (data, valor, categoria) antes de montar qualquer coisa. Se ele avisar que "arrumou" algo na leitura, veja se faz sentido.
3. **Olhe o dashboard.** Ele é montado sem IA, direto dos seus dados.
4. **Abra o chat do projeto e pergunte.** Por exemplo:
   - "qual o total de vendas?"
   - "vendas por loja" ou "top 5 produtos"
   - "vendas em outubro" e depois "e por produto?"
   - "compare com o mês anterior"
   - "por que caiu?"
5. **Clique em "Como foi calculado"** em qualquer resposta. Aparece a consulta usada. É a forma de conferir se o número é o que você esperava.

## 4. O que mais vale testar

- Perguntar do jeito que você perguntaria a alguém que entende do seu negócio.
- Tentar confundir o assistente. Se ele inventar um número, eu quero saber.
- Usar no celular.
- Gastar os tickets até acabar. O chat não trava: ele avisa que a IA chegou no limite e mostra perguntas que continuam funcionando.
- Compartilhar o dashboard por link, na aba Compartilhar.

## 5. Achou um problema?

O melhor caminho é dentro do próprio Algory: **Ajuda › Ajuda e chamados**. Você escolhe o tipo (dúvida, problema, reclamação ou sugestão) e escreve. O chamado chega direto em mim, e a resposta aparece na mesma tela e no seu e-mail.

Ajuda muito dizer:

- o que você perguntou ou fez;
- o que esperava;
- o que apareceu (um print ou o texto de "Como foi calculado").

Outros caminhos:

- **E-mail:** mail@osmio.tech
- **LinkedIn:** [Morigan Ventrici](https://www.linkedin.com/in/morigan-ventrici/)

## 6. O que eu vejo do seu uso

Vejo o **registro de uso** da conta: horário de cada pergunta, qual caminho respondeu (regra, árvore ou IA), quantos tickets gastou e quanto demorou. É com isso que eu melhoro o sistema.

**Não** vejo o texto das suas perguntas, as respostas nem o conteúdo das suas planilhas. O texto das perguntas nem é guardado. Colunas de identificação (nome, CPF, e-mail, endereço) são retiradas antes de qualquer coisa ser armazenada.

Em **Privacidade › Sua conta e seus dados** você baixa tudo o que o Algory guarda sobre você e exclui a conta quando quiser.

## 7. Depois do teste

A conta continua existindo no plano Grátis, com os seus projetos. Se quiser seguir usando com mais tickets, os planos estão em Perfil › Plano e pagamentos. O pagamento é por Pix, por 30 dias, sem mensalidade nem cobrança automática. Não é obrigação: o teste é para você conhecer e para eu aprender.

## 8. Quer conhecer o projeto?

Esta página é só a porta de entrada para quem vai testar. O resto está no repositório:

- **[Apresentação do Algory](../README.md):** o que é, o que funciona hoje, valores e limitações conhecidas. Comece por aqui.
- **[Como usar](uso.md):** a ferramenta inteira, tela por tela.
- **[Como a IA trabalha](como-a-llm-trabalha.md)** e **[segurança](seguranca.md):** o caminho de uma pergunta e o que é feito com os seus dados.
