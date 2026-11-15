# Integrações: o que é real e o que vem

Situação em 13/11/2026. Na tela **Ajustes › API e integrações**, tudo que não é real aparece como **Em breve**, com o botão desligado (`ui93.js`). Nenhuma integração simulada grava "conectado" na conta. Conexões antigas guardadas no navegador são apagadas ao abrir a tela.

## Funciona hoje

| Fonte / saída | Como | Onde no código |
|---|---|---|
| Arquivo CSV / XLSX | Upload no projeto (ou botão Anexar) | `main.py` `subir_tabela`, `pessoas.py` |
| Imagem (print, foto de tabela) | Anexar; OCR por/eng/spa; só planos pagos e códigos de demo | `imagem.py`, `ocr.py` |
| Google Planilhas | Link publicado: projeto › Tabelas › "Ligar planilha do Google". Botão **Atualizar do Google** no projeto (intervalo por plano e tamanho) | `gsheet.py`, `ui77.js`, `ui94.js` |
| PostgreSQL | Cópia única de uma tabela ou visão do banco do cliente para um projeto (plano Médio em diante). A senha não fica guardada | `conector.py`, `ui99.js`; ver `docs/conector.md` |
| Dados públicos | BCB (Selic, IPCA, câmbio), IBGE, BrasilAPI (CNPJ, CEP) | `publico.py` |
| Link do dashboard por e-mail | Compartilhar › Enviar por e-mail. Corpo padrão (pt/en/es), até 5 destinatários, sem texto livre | `integra.py` `enviar_link`, `ui95.js` |
| Resumo por e-mail (`mailsum`) | Todo dia ou toda segunda, 8h, para o e-mail da conta. Por regra, sem IA e sem ticket. Não repete se os números não mudaram | `integra.py` `rodada`, `agenda.py` |
| Alertas em chat (`hook`) | Webhook de entrada do Slack, Discord ou Google Chat; recebe o mesmo resumo | `integra.py` `hook_enviar` |
| Agenda por link (`ics`) | Link secreto .ics: fechamento do mês, vencimento do plano e dos links públicos, dia do resumo semanal | `integra.py` `ics`, rota `/agenda/{token}.ics` |

### Segurança das integrações de saída

- **Webhook:** o servidor só chama endereços que casam com o formato oficial do Slack, Discord ou Google Chat (lista fechada, HTTPS, sem seguir redirecionamento). Não dá para apontar para a rede interna (SSRF). O endereço fica só no servidor (`integra.json` do projeto, permissão 600); a tela recebe mascarado.
- **Menções:** `@everyone` e `<!channel>` vindos dos dados não disparam.
- **E-mail:** corpo padrão; o título do projeto perde HTML, ponto e dois-pontos (não dá para montar link). O texto é escapado no HTML. Cota por conta por dia: Grátis 3, Beta 10, Básico 20, Médio 50, Avançado 100; e 3 por hora por destinatário.
- **Remetente separado:** com `SMTP_HOST/SMTP_USER/SMTP_PASS` no `.env`, links e resumos saem pela caixa própria (ex.: mail@osmio.tech) e o Brevo fica só para os códigos de entrada. Sem SMTP, usam o Brevo até 60% do teto diário.
- **Agenda:** só datas, nenhum número dos dados. Só o hash do token fica guardado; gerar outro derruba o anterior.
- **Nomes da equipe** saem abreviados (nível 1) no resumo.
- Resumo, webhook e agenda: a partir do Básico. Plano que volta para o Grátis para de receber.

## Em breve (WIP)

| id | Integração | Falta |
|---|---|---|
| gcal | Google Agenda (escrever eventos) | OAuth Google |
| csv | Arquivo por e-mail | Caixa de entrada própria (inbound) |
| gdrive | Google Drive | OAuth Google, escopo só arquivos escolhidos (auditoria do Google) |
| outlook | Outlook / OneDrive | OAuth Microsoft Graph |
| quasar | Quasar | Contrato entre os apps |
| pg / mysql | Banco de dados | Conector só leitura, usuário dedicado, IP fixo de saída |
| api | API privada | Desenho da estrutura (fica para depois) |
| — | Teams no webhook | Formato novo (Power Automate), sem conta para testar |

## Plano

Estudo de 13/11/2026: o que cada uma pede de fora e se dá para testar.

| Integração | Como autentica | O que precisa de fora | Esforço |
|---|---|---|---|
| **Omie** | Chave fixa (app_key + app_secret) que o cliente copia do painel dele | Nada nosso. Para testar: uma conta Omie (tem teste grátis) | 1 a 2 dias |
| **Google Drive/Planilhas por login** | OAuth com escopo `drive.file` + seletor de arquivos do Google (não sensível: sem auditoria de segurança) | Projeto no Google Cloud, tela de consentimento, política de privacidade publicada | 2 a 3 dias + verificação básica da marca |
| **Bling** | OAuth 2 (API v3); sem ambiente de teste | Aplicativo cadastrado numa conta Bling | 2 a 3 dias |
| **Tiny (Olist)** / **Conta Azul** | OAuth | Aplicativo cadastrado em cada um | 2 a 3 dias cada |
| **Excel/OneDrive** | OAuth Microsoft Graph | Aplicativo no Entra ID (Azure) | 2 a 3 dias |
| **PostgreSQL** | **Feito na al057** como cópia única, sem guardar a senha | Endereço público conferido, só leitura, SSL, limite de tempo e de linhas | Ver `docs/conector.md` |
| **MySQL** | Usuário só leitura do cliente | As mesmas do PostgreSQL, com outra biblioteca | 1 dia + testes |
| **Arquivo por e-mail** | Endereço próprio por projeto | Caixa de entrada (inbound) no provedor de e-mail | 1 a 2 dias |
| **API privada** | A definir | Desenho da estrutura | depois |

Ordem sugerida: Omie (não depende de cadastro nosso), Google por login, Bling, os demais.

## Regras que valem para toda integração nova

- Pedir só a permissão que usa, e só leitura quando der.
- Colunas que identificam pessoas (nome, CPF, e-mail, telefone) passam por `pessoas.py` / `mask.py` antes de gravar.
- Tokens de terceiros cifrados no servidor, nunca no navegador.
- Nenhum dado do cliente vai para provedor que treina com dados.
