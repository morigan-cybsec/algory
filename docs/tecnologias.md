# Tecnologias

| Camada | O que usei | Por quê |
|---|---|---|
| Servidor | **Python 3.12 + FastAPI** | Rápido de escrever e de testar. Os modelos de entrada validam tudo que chega antes de o código rodar. |
| Dados do cliente | **DuckDB**, um arquivo por projeto | Banco analítico que roda dentro do processo. Cada projeto é um arquivo isolado: um cliente não tem como alcançar o dado de outro. |
| Contas, uso e pagamentos | **PostgreSQL 16 + pgvector** | Dado que precisa de transação. O pgvector guarda os trechos de documentos para busca. |
| Fila e cache | **Redis** | Tarefas em segundo plano e respostas já dadas. |
| Tela | **JavaScript puro, sem framework** | 101 módulos montados num único arquivo HTML. Sem etapa de compilação, sem dependência para atualizar. |
| LLM principal | **Claude Haiku (Anthropic), por API** | É quem escreve o texto que o cliente lê. Bom custo por resposta e segue instrução de formato. |
| LLMs de apoio | **Groq, Cloudflare Workers AI, Gemini** | Tarefas internas (rotear, gerar consulta, checar segurança) e reserva se o principal cair. Provedor que treina com os dados nunca recebe dado real de cliente. |
| Leitura de imagem | **Tesseract** (português, inglês, espanhol) | Roda no servidor, sem mandar a imagem para fora. |
| Planilhas | **openpyxl** | Lê XLSX sem executar macro. |
| Feriados | **holidays** | Nacionais e estaduais, com os móveis calculados, sem depender de rede. |
| Dados públicos | **Banco Central, IBGE, BrasilAPI** | Selic, IPCA, câmbio, municípios, CEP. |
| Pagamento | **Mercado Pago (PIX)** | O preço é sempre recalculado no servidor. |
| E-mail | **Brevo** | Códigos de entrada e resumos. |
| Infraestrutura | **Docker Compose, nginx, VPS** | Seis serviços: API, painel de administração, fila, agenda, banco e Redis. |
| Testes | **pytest** (270 testes), **Playwright** | Os testes do servidor rodam sem banco e sem chave de IA. O fluxo da tela é testado num navegador de verdade. |
| Desenvolvimento | **Claude (Anthropic)** como assistente de código | Ver o [estudo de caso](estudo-de-caso.md). |

## Decisões que eu repetiria

- **Um banco por projeto.** Custou um pouco de organização e resolveu de vez o isolamento entre clientes.
- **Regra antes de IA.** Tudo que dá para resolver com regra, eu resolvo com regra. A IA fica para o que é linguagem.
- **Tela sem framework.** Para um projeto de uma pessoa, não ter cadeia de ferramentas foi um alívio.

## Decisões que eu mudaria

- **Módulos da tela empilhados.** Cada correção virou um módulo que envolve o anterior. Foi ótimo para entregar rápido e reverter fácil, mas a ordem passou a importar e ler o código ficou mais difícil. Numa segunda versão eu consolidaria.
- **Checagem de consulta por texto.** Comecei barrando comandos perigosos com expressão regular. Uma varredura mostrou que dava para escapar. A defesa de verdade passou a ser o próprio banco aberto sem acesso a arquivo e rede. Detalhes em [segurança](seguranca.md).
