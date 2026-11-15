# Importar tabela de um PostgreSQL (al057)

O cliente copia uma tabela ou visão do banco dele para um projeto do Algory. É uma **cópia única**, não uma ligação permanente.

## Por que cópia e não ligação

Uma ligação permanente obriga a guardar a senha do banco do cliente. Guardar credencial de terceiros pede cofre de segredos, rotação e um plano para vazamento. Preferi não ter o problema: **a senha é usada uma vez e descartada.** Não vai para arquivo, banco nem registro. Para atualizar os dados, o cliente importa de novo e a tabela é substituída.

## Como funciona

1. Em **Integrações › PostgreSQL** o cliente escolhe o projeto e informa servidor, porta, banco, esquema, usuário e senha.
2. **Ver tabelas:** o servidor conecta e lista as tabelas e visões que aquele usuário enxerga.
3. **Importar:** o servidor lê a tabela escolhida (até o limite do plano), fecha a conexão e passa as linhas pela mesma limpeza de qualquer planilha.

Rota: `POST /projetos/{pid}/importar/postgres`. Sem `tabela` no corpo, devolve a lista; com `tabela`, importa.

| Plano | Linhas por importação |
|---|---|
| Grátis, Básico | não disponível |
| Médio, Beta | 50.000 |
| Avançado | 100.000 |

## Proteções (`app/conector.py`)

| Risco | O que segura |
|---|---|
| Usar o Algory para alcançar a rede interna do servidor | O endereço tem de ser um nome público. Todos os IPs que o nome resolve são conferidos; se **um** for interno, loopback, link-local ou reservado, recusa. |
| Trocar o DNS depois da conferência | A conexão vai para o IP conferido (`hostaddr`), não resolve o nome de novo. |
| Usar o Algory como varredor de portas | Só as portas 5432, 5433 e 6543. No máximo 8 conexões por conta a cada 10 minutos. |
| Senha trafegar aberta | Conexão cifrada obrigatória (`sslmode=require`). Banco sem SSL é recusado com a explicação. |
| Escrever no banco do cliente | A transação é somente leitura, mesmo que o usuário informado possa escrever. |
| Injeção pelo nome da tabela | Esquema e tabela entram como identificador (`psycopg.sql.Identifier`) e a tabela tem de existir em `information_schema`. |
| Consulta que não termina | 6 s para conectar, 15 s para ler, teto de linhas e de 200 colunas. |
| Dado pessoal do banco do cliente | Nome, CPF, e-mail, telefone e endereço saem antes de gravar; vendedor vira código. Igual a planilha. |
| Mensagem de erro vazar detalhe | As mensagens são fixas. Nenhuma devolve a senha, a cadeia de conexão nem o erro cru do banco. |

Na tela, a senha fica só na memória enquanto o modal está aberto (são dois passos) e é apagada ao importar ou fechar. Não vai para `localStorage`.

## O que recomendar ao cliente

- Criar no banco um **usuário só de leitura**, com acesso só ao que vai importar.
- Criar uma **visão por assunto** (`vendas_2026`) em vez de liberar a tabela inteira.
- Liberar no firewall do banco a conexão vinda do servidor do Algory.

## Limites conhecidos

- Só PostgreSQL. MySQL pede outra biblioteca e os mesmos testes; ficou para depois.
- Sem atualização automática: por decisão, já que não guardo a senha.
- `sslmode=require` cifra a conexão, mas não confere o certificado do servidor.
- A bateria testa a conexão de verdade contra o banco do próprio servidor (só ela tem esse caminho). Um banco externo de cliente ainda não foi testado em produção.
