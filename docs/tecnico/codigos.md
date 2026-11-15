# Códigos de ativação

Painel de administração › Códigos de ativação. Servidor: `app/pagamentos.py`, migração `012_codigos_tester.sql`.

- **Formato:** `PREFIXO-XXXX-XXXX-XXXX`. Prefixo de 2 a 6 letras, escolhido ao gerar (padrão `ALGORY`; `BETA` e `FIEL` são reservados). Códigos antigos `ALG-…` continuam valendo.
- **Tipo Plano:** libera o plano por N dias a partir do resgate (soma ao que a conta já tem).
- **Tipo Tester:** teste curto, padrão 3 dias depois de ativar, com **teto de tickets próprio para o período todo** (sem cota por dia ou semana). Não entra em conta com plano pago ativo. Ao fim, a conta volta para o Grátis e nada é apagado. Se a pessoa comprar um plano durante o teste, o plano pago começa na hora e o teto some.
- **Período (opcional):** "vale a partir de" e/ou "até".
  - Sem data de início: vale no resgate.
  - Resgatado antes do início: fica guardado na conta e liga sozinho no dia (agenda das 3h, ou ao abrir Plano e pagamentos). A prévia avisa a data e que, se a conta tiver um plano pago ativo nesse dia, a ativação é cancelada.
  - Com data de fim: vale até lá, seja qual for o dia do resgate.
- **Contas em teste** (aba do painel): quem resgatou código Tester, tickets usados contra o teto, número de perguntas e o registro de uso de cada uma (horário, camada, modelo, tickets, tempo, consultas, bloqueios). O texto das perguntas e das respostas não é guardado em lugar nenhum; a prévia do código avisa o cliente desse registro.
