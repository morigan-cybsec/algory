# Histórico de mudanças

Cada entrada é uma correção instalada em produção (`alNNN`), com a bateria de checagens rodada no servidor depois.

## Novembro de 2026

| Versão | O que mudou |
|---|---|
| al066 | Texto do pagamento simplificado: problema com pagamento se resolve por chamado ou e-mail. README com a lista de limitações conhecidas, com a dívida do código da tela em primeiro lugar. |
| al065 | Texto do beta refeito na tela do plano, na árvore e na documentação: pode ter erro e há chamado direto com a Osmio; pagamento por Pix sem cobrança automática; o que foi pago é honrado, e problema com pagamento se resolve por chamado ou e-mail; o que é acompanhado para melhorar são números de uso, não o conteúdo dos dados. |
| al064 | Compromisso do beta escrito na tela do plano, na árvore e na documentação: o que foi pago é honrado pelo período pago, no valor pago; mudança só vale para os períodos seguintes e é avisada antes. |
| al063 | O chat geral (fora de projeto) passa a usar a árvore do servidor: a árvore antiga da tela casava por palavra solta e tinha texto velho. Teste que abre as 21 telas e abas em dois tamanhos e guarda uma base de comparação, para a consolidação da tela não ser feita às cegas. |
| al062 | **Planilhas bagunçadas:** a linha "TOTAL" de um CSV entrava como venda e o total do chat saía em dobro; título acima do cabeçalho, milhar com ponto ("12.345"), negativo entre parênteses, "-" e "N/A" e coluna repetida também eram lidos errado. Tudo é arrumado na leitura, e a tela avisa o que foi mexido. Excel com subtotal e linha em branco vira uma tabela só. **Árvore:** busca por palavras-chave depois da frase; quando dois ou três assuntos empatam, o chat pergunta qual deles em vez de chutar. **Segurança:** o que vem de um link do Google passa pela mesma portaria do envio direto. |
| al061 | **Modo sem IA:** quando os tickets acabam, o chat não trava: avisa o motivo e oferece botões com perguntas que funcionam sem IA, montadas com as métricas do próprio projeto (cada uma conferida pela regra antes de virar botão) e com assuntos de ajuda. **Árvore** refeita: de 5 para 13 tópicos, casados pela frase e não por palavra solta ("qual o gasto com fornecedor?" recebia a explicação do preço do ticket). Na IA, a consulta passa a receber a coluna de data com o intervalo e a métrica principal (13 de 16 na amostra da al060). |
| al060 | **O Algory se apresenta como beta:** selo na tela e aviso de que os valores são os da fase beta e podem mudar. **Chamados de ajuda:** o "Fale conosco" virou chamado com número, tipo e situação, e a resposta aparece no app. Teste com sete planilhas públicas: dois tipos de arquivo que davam erro no envio (fim de linha antigo, coluna sem título), cp1252, data americana, ano somado como medida e categorias largas, todos corrigidos. Na IA, projeto com tabela sempre tenta a consulta (as respostas erradas da rodada anterior saíram sem consulta). Aviso por e-mail quando a API cai ou o disco enche. **Segurança:** FastAPI e Starlette atualizados depois que a auditoria apontou duas falhas conhecidas. |
| al059 | **Memória da conversa** sem guardar a conversa: o contexto fica no navegador e o servidor confere antes de usar ("e por produto?", "e em setembro?", "por que mudou?"). Comparação de períodos por regra. Desempate entre tabelas do mesmo projeto. **A conta nas mãos do dono:** baixar os dados, sair dos outros aparelhos, excluir a conta. Alertas de gasto e de tentativas para quem cuida. Vazamento da instrução do sistema barrado e testado. Auditoria das bibliotecas na bateria. Resumo por e-mail com "o que mudou". |
| al058 | A regra das métricas testada em cinco ramos (clínica, delivery, escola, planilha em inglês, estoque) e corrigida: "quantos pedidos" conta linhas, taxa de entrega é em reais, verbos de dinheiro e de contagem, plural em filtros, vocabulário em inglês. **Respondedor de reserva desligado:** sem tickets, ele chutava a resposta. Cópia de segurança diária do banco e dos dados. |
| al057 | Importar tabela de um PostgreSQL do cliente: cópia única, só leitura, sem guardar a senha, rede interna recusada. Dashboard compara mês incompleto com os mesmos dias do mês anterior. Decimal pequeno não vira mais porcentagem no texto da IA. O chat lista as métricas do projeto. |
| al056 | Métricas com nome (a mesma pergunta dá sempre a mesma consulta, sem IA), investigação em passos com teto ("por que caiu?") e gabarito de 49 perguntas para medir a regra e a IA. Corrigido: a coluna "quantidade" era tratada como média. |
| al055 | **Segurança:** portaria dos arquivos. Só planilha entra como planilha: programa, script, página, PDF, compactado, nome com caminho e extensão dupla são recusados pelo conteúdo. |
| al054 | Folga fixa de uma pessoa só tira dias da jornada da empresa (antes criava trabalho em dia que a empresa fecha). |
| al053 | **Segurança:** o banco do projeto abre sem acesso a arquivo e rede; cabeçalho de planilha não vira comando. Limite de buscas de CEP ao mesmo tempo. |
| al052 | CEP vira estado, região e cidade. O CEP inteiro e o endereço não são gravados. |
| al051 | Calendário do projeto (jornada, feriados, folgas, escalas), ritmo da meta, cartões e modais no dashboard, 5 modelos e 8 cartões novos. Códigos com prefixo, tipo Tester e período com data de início. Aba "Contas em teste" no painel. |
| al050 | Integrações só ligam em projeto com dados. Estudo das próximas integrações. |
| al049 | Integrações reais de saída: link por e-mail, resumo por e-mail, alertas em chat por webhook, agenda .ics. Caixa de e-mail própria separada dos códigos de entrada. |
| al048 | Botão "Atualizar do Google", com intervalo por plano e tamanho da planilha. |
| al047 | Integrações que não existiam de verdade passam a aparecer como "Em breve". |
| al046 | Cabeçalhos de segurança na página (CSP, frame, nosniff). Consulta de rotina da conta fora do limite do login. |
| al045 | Perguntas sobre pessoas da equipe: o modelo recebe o ID com a coluna de origem. |
| al044 | Termos de uso com a identificação do responsável. |
| al043 | A tela mantém a rolagem ao redesenhar. Cartão do plano ativado por código. Botão Anexar. |
| al042 | Google Planilhas publicada com uma aba só. |
| al041 | Nomes da equipe viram ID. Link público com três níveis de privacidade. |
| al040 | Rota interna fechada para fora. |
| al039 | OCR em português, inglês e espanhol. Resposta conferida contra o idioma do cliente. Moderação nos três idiomas. |
| al037–al038 | Verificação de imagens (limpeza, OCR, moderação, visão). Imagens só em planos pagos. Bateria de checagens dentro da API. |
| al035–al036 | Ajustes do relatório pelo chat (renomear, trocar agregação, priorizar). |
| al032–al034 | Ligações automáticas entre tabelas, com origem e motivo. Assistente na tela de associar tabelas. |

Versões anteriores (al001–al031): base do produto, contas, pagamentos, leitor de planilhas, dashboard por regra, comparação, arrumação de dados, comunidade e painel de administração.
