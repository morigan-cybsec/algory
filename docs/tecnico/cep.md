# CEP: estado, região e cidade

Servidor: `app/cep.py`, chamado na carga de qualquer planilha (`main.py` `_prepara`).

| O que | Como | Custo |
|---|---|---|
| Estado (`uf`) e região (`regiao`) | Faixa do CEP (tabela fixa dos Correios, 30 faixas) | Na hora, sem rede, para qualquer número de linhas |
| Cidade (`cidade`) | Um CEP por **área** (5 primeiros dígitos) na BrasilAPI, em segundo plano, ~3 por segundo; no máximo 800 áreas novas por carga | O upload não espera; a cidade aparece em instantes |
| Área (`cep_area`) | Os 5 primeiros dígitos | — |

- A resposta "área → cidade" fica em `data/cep_areas.json` e vale para todos os projetos (não é dado de ninguém). A segunda planilha da mesma região não consulta nada.
- Se a BrasilAPI cair, a carga segue só com estado e região; a próxima carga tenta de novo. Área que não existe fica marcada para não ser perguntada outra vez.
- **Privacidade:** o CEP inteiro e o endereço não são gravados. Quando não há coluna de CEP, mas o endereço traz um ("Rua X, 10 - 09910-770"), ele é aproveitado antes de o endereço sair.
- Colunas `uf`, `regiao` ou `cidade` que a planilha já traz preenchidas são respeitadas.
- Sem CEP (só rua e bairro) não há região: exigiria geocodificação paga e mandar o endereço do cliente para fora.
