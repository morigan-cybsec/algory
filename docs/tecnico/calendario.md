# Calendário do projeto

Aba **Calendário** de cada projeto com dados. É a base para dias úteis, ritmo da meta e carga horária. Tudo por regra, sem IA e sem ticket.
Servidor: `app/calendario.py` (guarda em `<pasta do projeto>/calendario.json`). Tela: `ui97.js` (aba) e `ui98.js` (cartões e modais do dashboard).

## O que o cliente configura

| Item | Como | Padrão |
|---|---|---|
| Jornada | Linha com os 7 dias da semana; cada toque já salva | Segunda a sexta |
| Horário | Entrada, saída e pausa (opcional) | Sem horário |
| Feriado é folga | Liga/desliga | Ligado |
| Pontos facultativos | Carnaval, Cinzas, Corpus Christi, vésperas | Desligado |
| CEP | Só para achar o estado (feriados estaduais) e o nome da cidade | Só nacionais |
| Feriados da cidade e folgas fixas | Dia/mês + nome; repetem todo ano | Nenhum |
| Um dia | Marcar folga, marcar dia de trabalho ou voltar ao padrão; motivo opcional | — |
| Compromissos | Título e hora por dia | — |
| Escala por pessoa | Folga fixa na semana (um dia a menos que a empresa) e/ou ciclo (6x1, 5x2, 12x36, N x M). Só quando a planilha tem coluna de nomes | Segue a empresa |

## Ordem de decisão de um dia

Empresa: marcado à mão → feriado (se "feriado é folga") → dia da semana da jornada.
Pessoa: marcado para ela → marcado para a empresa → feriado → escala em ciclo → folga fixa → jornada da empresa.

## Feriados

Nacionais e estaduais vêm da biblioteca `holidays` (sem rede, calcula os móveis). Municipais não têm fonte pública confiável: o cliente cadastra os da cidade dele.

## Onde o calendário entra

- **Prazos em dias úteis** (`painel.py`): com calendário salvo, contam os dias que a empresa trabalha de verdade (jornada, feriados, folgas). Sem calendário, continua segunda a sexta.
- **Ritmo da meta** (`painel.py` → `ritmo` e uma frase): meta do último mês com dados contra os dias úteis. Diz o esperado até hoje, o necessário por dia útil e a projeção de fechamento. Aparece no dashboard, no link público e no resumo por e-mail/chat.
- **Carga horária**: dias úteis × horas do dia, por mês e por pessoa.
- **Dashboard** (`ui98.js`): cartão *Ritmo da meta* e cartão *A semana* (3 dias antes, hoje, 3 depois). Tocar num dia abre o modal do dia; tocar no ritmo abre a conta inteira.

## Na biblioteca (Repertório)

Cartões novos no índice: Ritmo da meta, Dias úteis, Calendário do mês, Dia, Faixa de 7 dias, Aviso de calendário, Dias da jornada, Modal do dia.
Gráficos novos nos modelos (`dash_lib3/core.js`): `cal`, `faixa`, `ritmo`, `escala`.
Modelos novos (função "Calendário e jornada"): Calendário de trabalho do mês, Meta por dia útil, Escala e folgas da equipe, Carga horária prevista e realizada, Impacto de feriados e fins de semana.
Modelos existentes que ganharam cartão de calendário: 15 (metas, vendedores, orçado, absenteísmo, capacidade, entregas, projetos, contas a pagar e a receber, scorecard).

## Limites

1.500 dias marcados, 500 compromissos, 40 feriados locais e 200 pessoas com escala por projeto. Texto livre (motivo, título) perde `<` e `>`.
Nomes da equipe: a tela do dono mostra o nome; no arquivo fica só o ID (`P-XXXXXX`).
