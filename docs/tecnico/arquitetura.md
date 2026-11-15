> O código citado aqui não está neste repositório. Ver o [aviso no README](../../README.md#por-que-o-código-não-está-aqui).

# Arquitetura

## Serviços

| Serviço | O que é | Para quê |
|---|---|---|
| `api` | FastAPI (`app/main.py`) | Rotas do produto. Atrás do nginx, em `/api/`. |
| `adm` | FastAPI separado (`app/adm_app.py`) | Painel de administração, em outro domínio e outro contêiner. A API do produto não expõe rota de administração para fora. |
| `worker` | Fila (RQ) | Leitura de documentos e tarefas noturnas. |
| `agenda` | Processo simples (`app/agenda.py`) | 03:00: limpeza, moderação, renovações, códigos agendados. 08:00: resumos por e-mail e chat. |
| `db` | Postgres 16 + pgvector | Contas, sessões, uso, pagamentos, códigos, links, trechos de documentos. |
| `redis` | Redis | Fila e cache. |
| DuckDB | Um arquivo por projeto | Os dados da planilha do cliente. Cada projeto só enxerga o próprio arquivo. |

A tela é um arquivo HTML estático servido pelo nginx na raiz do mesmo domínio.

## O caminho de uma planilha

1. **Leitura** (`planilha.py`, `ocr.py`): acha os quadros dentro da aba, cabeçalhos, fórmulas e informações soltas. Planilha com meses nas colunas é "deitada" em linhas.
2. **Limpeza** (`mask.py`, `cep.py`, `pessoas.py`): colunas de identificação saem; telefone vira estado; CEP vira estado, região e cidade; nome de vendedor vira ID.
3. **Conferência**: quando a leitura deixa dúvida, a tela mostra o que foi lido e o cliente confirma antes de virar tabela.
4. **Carga** (`sqlguard.load_table`): a tabela entra no DuckDB do projeto.
5. **Significado das colunas** (`colunas.py`): data, medida, meta, grupo. Por regra, com ajuste do cliente.

## O dashboard (sem IA)

`painel.py` monta tudo por regra: período, indicadores, série, rankings, prazos, comparação e frases. O calendário do projeto (`calendario.py`) entra nos prazos em dias úteis e no ritmo da meta. Não chama modelo, não gasta ticket.

## O caminho de uma pergunta (`pipeline.py`)

```
máscara → árvore e cache (T0) → cota → roteador (T1) → teto de gasto
        → contexto (SQL e/ou trechos de documento) → geração → conferência → tickets → traço
```

| Etapa | O que faz |
|---|---|
| Máscara | CPF, CNPJ, e-mail, telefone e cartão no texto da pergunta viram marcadores antes de qualquer modelo. |
| T0 | Perguntas fixas do produto e respostas já dadas saem da árvore ou do cache. Sem modelo, sem ticket. |
| Cota | Tickets por mês, semana e dia conforme o plano. |
| T1 | Roteador por regra escolhe a camada do modelo. O modelo só é consultado para rotear quando a regra fica em dúvida. |
| Teto | Limite de gasto real por conta e por dia. Se estourar, desce de camada ou para. |
| SQL | O modelo recebe só o esquema (nomes de colunas e 3 linhas de exemplo) e devolve uma consulta. `sqlguard.py` confere: só SELECT, só tabelas do projeto, 10 mil linhas, 5 segundos, sem acesso a arquivo ou rede. |
| Geração | O modelo escreve o texto com marcadores no lugar dos números. O servidor troca pelos valores do resultado. |
| Conferência | Número sem origem, citação que não existe ou idioma errado: refaz uma vez por nossa conta; se persistir, avisa. |
| Traço | Fica registrado o que foi feito (camada, modelo, tickets, tempo). O texto da pergunta não é guardado. |

## Modelos e provedores (`llm.py`, `modelos.py`)

Porta única para todos os modelos, com custo real medido. O texto que o cliente lê vem do Claude. Provedores que treinam com os dados enviados só recebem dados fictícios.

## A tela (`front/`)

JavaScript sem framework. São 101 módulos carregados em ordem; cada um estende o anterior (guarda a função antiga e a envolve). Isso permitiu entregar correções pequenas e reversíveis, ao custo de a ordem importar. `front/build.py` junta tudo, o CSS, a biblioteca e os ícones num arquivo só.

## Biblioteca (`biblioteca/`)

`catalogo.json` descreve 76 modelos de dashboard: que pergunta respondem, que colunas exigem e que gráficos usam. `core.js` desenha os exemplos; `cards.js` é o índice dos 28 cartões, com quando usar e quando evitar cada um.
