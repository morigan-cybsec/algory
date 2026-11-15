# Cópia de segurança (al058)

Até a al057 não existia cópia nenhuma: um `DROP` errado ou um disco corrompido levava as contas, os pagamentos e as planilhas de todo mundo.

## O que é copiado

| O quê | Como | Onde fica |
|---|---|---|
| Banco das contas (usuários, sessões, pagamentos, códigos, livro de tickets) | `pg_dump -Fc` de dentro do contêiner `db` | `~/algory_api/backups/pg_AAAAMMDD-HHMM.dump` |
| Pasta de dados (um banco DuckDB por projeto, calendários, ajustes de colunas, retratos das planilhas) | `tar.gz` da pasta `data` | `~/algory_api/backups/dados_AAAAMMDD-HHMM.tar.gz` |

- Todo dia às **04:17**, por `/etc/cron.d/algory_backup` (instalado pelo `instalar.sh`).
- Guarda os **últimos 7 dias**. A pasta e os arquivos são só do root (`700` / `600`).
- Não roda se o disco livre for menor que duas vezes a pasta de dados mais 512 MB.
- O resultado de cada rodada fica em `backups/ultimo.txt`, e a bateria confere que a última cópia tem menos de 26 horas.

## O que isto NÃO resolve

A cópia fica **no mesmo servidor**. Protege de erro meu, de correção ruim e de arquivo corrompido. Não protege de perder o servidor. Para isso, copie para fora de tempos em tempos:

```
scp -r osmio@SERVIDOR:algory_api/backups ./algory_backups_$(date +%F)
```

As cópias têm dado pessoal (e-mail, hash de senha). Guarde num disco cifrado.

### Envio automático para fora (al059)

Se existir o arquivo `~/algory_api/.env.backup`, o roteiro manda as duas cópias do dia por `scp` logo depois de gerar:

```
ALGORY_BACKUP_REMOTO=usuario@outro-servidor:/pasta/algory
ALGORY_BACKUP_CHAVE=/root/.ssh/algory_backup
```

A chave tem de ser sem senha e servir só para isto. No destino, vale restringir a chave a `scp` naquela pasta. O resultado aparece em `backups/ultimo.txt` (`fora=ok`, `fora=FALHOU` ou `fora=nao_configurada`) e a bateria mostra.

## Como restaurar

```
cd ~/algory_api
sudo docker compose stop api worker agenda adm

# banco das contas
sudo docker compose exec -T db pg_restore -U algory -d algory --clean --if-exists < backups/pg_AAAAMMDD-HHMM.dump

# pasta de dados (guarda a atual antes)
sudo mv data data.antes && sudo tar -xzf backups/dados_AAAAMMDD-HHMM.tar.gz && sudo chown -R 1000:1000 data

sudo docker compose up -d
```

Restaurar o banco e a pasta **do mesmo horário** evita conta sem projeto ou projeto sem conta.

## Rodar na mão

```
sudo ~/algory_api/deploy/backup.sh
sudo cat ~/algory_api/backups/ultimo.txt
```
