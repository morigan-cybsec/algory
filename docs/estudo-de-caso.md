# Estudo de caso

Como o Algory foi feito, o que deu certo, o que deu errado e onde mais essa estrutura serve.

## O ponto de partida

O Matheus Ventrici trouxe a ideia: um assistente que lê a planilha de uma pequena empresa e responde o que o dono quer saber. Dono de loja não abre ferramenta de BI. Ele tem uma planilha e uma dúvida.

Eu peguei a ideia como desculpa para estudar três coisas ao mesmo tempo:

1. até onde dá para ir desenvolvendo com IA como ferramenta de trabalho;
2. como estruturar uma LLM por API num produto que cobra pelo uso;
3. como um chatbot se comporta quando tem usuário de verdade do outro lado.

## A primeira semana

Em uma semana o Algory tinha as funções principais: conta com verificação em duas etapas, leitura de planilha, dashboard por regra, perguntas em português, tickets, pagamento por PIX, painel de administração e as barreiras de segurança da LLM.

Depois vieram as correções em produção. Cada uma tem número (`al001` a `al066`), entra por um instalador que faz backup antes e é seguida de uma bateria de checagens no próprio servidor. Hoje a bateria tem mais de 190 checagens e o projeto tem 270 testes automáticos. O histórico está no [CHANGELOG](../CHANGELOG.md).

## As ferramentas

| Para quê | Ferramenta |
|---|---|
| Escrever, revisar e testar código | Claude (Anthropic), em conversa e operando o navegador |
| Servidor | Python, FastAPI |
| Dados | DuckDB (um arquivo por projeto), PostgreSQL, Redis |
| LLM do produto | Claude Haiku por API; Groq, Cloudflare e Gemini em tarefas internas |
| Testes | pytest, Playwright, e a bateria que roda dentro da API em produção |
| Infraestrutura | VPS, Docker Compose, nginx |

A lista completa, com o motivo de cada uma, está em [tecnologias](tecnologias.md).

## Como foi trabalhar com IA

Vale contar com honestidade, porque era o objetivo do estudo.

**O que funcionou.**

- **Velocidade com rede de proteção.** A IA escreve rápido. O que tornou isso seguro foi o processo em volta: toda mudança vira um pacote numerado, com backup, teste e bateria. Sem isso, velocidade é só um jeito de quebrar mais rápido.
- **Varredura de segurança.** Pedi uma revisão completa do código procurando falha. Ela achou duas graves que estavam lá desde o começo, reproduziu as duas e corrigiu. Estão descritas em [segurança](seguranca.md). Eu não teria achado tão cedo sozinho.
- **Teste em produção pelo navegador.** A IA operou o site logado numa conta de teste e tentou quebrar: arquivo malicioso, endereço interno em webhook, pergunta pedindo arquivo do servidor, rajada de requisições. Cada correção foi verificada no ar, não só no meu computador.
- **Documentação junto com o código.** Cada função nova saiu com o documento técnico dela.

**O que deu errado.**

- **A IA afirma que testou o que não testou.** Aconteceu mais de uma vez de uma checagem estar conferindo a coisa errada, ou um teste ter ficado desatualizado depois de uma mudança. Aprendi a pedir a medição, não a conclusão.
- **Ela resolve o pedido, não o problema.** Pedi integrações e recebi telas bonitas de integrações que não existiam. Tive que voltar, marcar tudo como "Em breve" e fazer de verdade só as que dava para testar.
- **Código empilhado.** Cada correção virou um módulo por cima do anterior. Funciona e é fácil de reverter, mas a tela hoje tem 101 módulos e a ordem importa.
- **Erro meu:** demorei para exigir teste de tudo. As duas falhas graves passaram semanas no ar porque as primeiras versões só testavam o caminho feliz.
- **Erro meu:** em alguns momentos colei senha de conta de teste na conversa. Não deveria. A regra agora é: credencial nunca vai para o chat.

- **Erro meu:** testei a regra das perguntas só com a minha planilha de exemplo, de "loja" e "valor". Quando fui para uma clínica e um delivery, ela respondeu "quantos pedidos?" com um valor em reais. Cinco planilhas de ramos diferentes viraram teste fixo.
- **Erro meu:** o projeto ficou semanas no ar sem cópia de segurança. Entrou na al058.

**O que eu levo.**

- A IA é um ótimo par de programação e um péssimo chefe. Quem decide o que é "pronto" tem que ser eu.
- Pedir "o porquê com medição" muda a qualidade da resposta.
- Uma pergunta de cada vez. Pedido grande demais volta pela metade.
- O processo (pacote, backup, bateria) vale mais que qualquer instrução bem escrita.
- Em uma semana, uma pessoa entrega o que antes pedia uma equipe pequena. Manter depois continua sendo trabalho de gente.

## O que aprendi sobre a LLM dentro do produto

O detalhe está em [como a LLM trabalha](como-a-llm-trabalha.md). O resumo:

- **Tire da IA tudo que não é linguagem.** O dashboard, os filtros, os prazos e a meta são regra. A IA só traduz pergunta em consulta e resultado em frase.
- **Não deixe a IA escrever número.** Marcador no texto, valor colocado pelo servidor.
- **Mande o esquema, não os dados.** É mais barato e é mais seguro.
- **Trate o que vem do usuário como dado, não como instrução.** Vale para a pergunta, para a célula da planilha e para o texto dentro de uma imagem.
- **A defesa que conta é a que não depende do modelo.** O banco aberto só para leitura, sem acesso a arquivo e rede, segura o que nenhuma instrução segura.

## Onde mais dá para aplicar

A estrutura não é sobre planilha de vendas. É: **dado tabular + regra para o que é exato + LLM para o que é linguagem + barreiras em volta.** Isso serve em outros lugares.

| Contexto | O que mudaria |
|---|---|
| **Assistente de interface** | Já existe: é o Algory dentro do [Flux](https://demoflux.osmio.tech) e do [Quasar](https://demoquasar.osmio.tech). Em vez de consultar dados, ele guia a pessoa pelo sistema. |
| **Atendimento com base de documentos** | Trocar a consulta ao banco por busca em documentos. A regra de citar só o que existe já está no projeto. |
| **Relatório recorrente** | O resumo por e-mail já é isso em pequeno: regra monta, IA só redige quando precisa. |
| **Auditoria de planilha** | O leitor já acha fórmula quebrada, categoria duplicada e dado fora do padrão. Dá para virar produto sozinho. |

## Ideias: Algory especializado

O Algory de hoje é generalista. A parte de regra é onde dá para especializar, porque é lá que mora o vocabulário de cada área.

| Versão | O que saberia de fábrica |
|---|---|
| **Algory Financeiro** | Fluxo de caixa, DRE, contas a pagar e receber, conciliação. Já consulta Selic, IPCA e câmbio. |
| **Algory Comercial** | Funil, conversão por etapa, comissão, meta por vendedor. O calendário com escala já serve de base. |
| **Algory Estoque** | Giro, ruptura, curva ABC, ponto de pedido. |
| **Algory Pessoas** | Escala, banco de horas, absenteísmo. Exige mais cuidado com dado pessoal do que qualquer outro. |
| **Algory Clínica ou Escola** | Agenda, ocupação, faltas. Só faz sentido com uma camada de privacidade ainda mais rígida. |
| **Algory Segurança** | Ler registro de acesso e apontar o que foge do padrão. É a minha área, e é a que eu mais quero testar. |

Em todas, a receita é a mesma: mais regra e mais vocabulário da área, a mesma IA, as mesmas barreiras.

## Onde o Algory se encaixa

Existem três famílias de produto parecidas: o BI corporativo com IA acoplada (Power BI, Tableau), o analista por conversa (você sobe um arquivo e pergunta) e a IA dentro da própria planilha. O Algory está na segunda, mirando quem as outras não atendem: a pequena empresa brasileira com uma planilha bagunçada e nenhum analista.

| Prática que o mercado recomenda | No Algory |
|---|---|
| Dar significado aos dados antes de gerar consulta | Papéis e significado das colunas, contas guardadas e, desde a al056, métricas com nome |
| Defesa imposta pelo banco, não pelo modelo | Banco só de leitura, sem arquivo e sem rede |
| Camadas de checagem em vez de "instrução melhor" | Verificador, checagem da consulta, conferência da resposta |
| Perguntas de gabarito para medir regressão | Gabarito de 49 perguntas, rodado a cada instalação |

O que decidi **não** copiar:

- **Executar código gerado.** Pede uma área isolada por usuário. Num servidor único com pagamento ao lado, é o maior risco que eu poderia abrir. Consulta só de leitura é uma limitação escolhida.
- **Agente livre.** Cada passo é uma chamada paga. Troquei por uma investigação de passos fixos, no banco, com teto.
- **Conectores de data warehouse com ligação permanente.** Guardar credencial de cliente pede cofre de segredos. Fiz o caminho curto: importar uma tabela do PostgreSQL como cópia única, sem guardar a senha.

O que não tem como empatar: distribuição. Quem já está dentro do Excel ou do Google Planilhas chega ao usuário sem ele procurar.

## O que as planilhas reais mostraram

Até a al059 a regra só tinha visto dado que eu mesmo gerei. Na al060 baixei sete planilhas públicas e subi pelo caminho normal. Duas nem entraram: uma tinha fim de linha antigo e a outra tinha a primeira coluna sem título, e as duas davam erro 500. Das 60 perguntas que fiz, 41 tinham resposta por regra. Depois das correções as sete sobem e 52 têm resposta; as outras 8 são recusas, que vão para a IA.

Na al062 fiz o passo seguinte: cinco arquivos montados do jeito que uma pequena empresa exporta do Excel. O pior achado foi silencioso: a linha "TOTAL" no fim do CSV entrava como venda, e o total do chat saía em dobro. Nenhum teste acusava, porque nenhum dado meu tinha linha de total.

O que eu tiro disso: teste com dado gerado mede se a conta está certa, e não mede se o arquivo entra. E dado público ainda é limpo demais. O teste que falta é com planilha de quem vai usar, e é por isso que o Algory passou a se apresentar como beta, com chamados de ajuda dentro do app. Os detalhes estão em [docs/tecnico/beta.md](tecnico/beta.md).

## O que falta

- Testar com planilhas de clientes de verdade.
- Um ambiente de homologação (hoje cada correção vai direto para produção, com backup antes e bateria depois).

- Consolidar os módulos da tela.
- Configurar as colunas pelo chat.
- Uma tela para o cliente criar as próprias métricas (hoje o catálogo sai dos papéis, significados e contas).
- Atualização automática do que vem do PostgreSQL (pede cofre de credenciais) e conector de MySQL.

Se você testar e achar algo, me avise: [para quem vai testar](testers.md).
