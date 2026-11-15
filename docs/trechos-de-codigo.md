# Trechos de código

O código completo do Algory não está neste repositório ([por quê](../README.md#por-que-o-código-não-está-aqui)). Aqui ficam os trechos que mostram as decisões mais importantes. São do servidor (Python, FastAPI), copiados como estão em produção.

Escolhi trechos que mostram a técnica sem servir de mapa: nenhuma lista de regras, assinaturas ou limites aparece aqui.

## 1. O modelo não vê o número e não escreve o número

O resultado da consulta vai para o modelo com cada valor numérico já trocado por um marcador. O modelo escreve a frase com os marcadores, e o servidor coloca os valores depois.

```python
def values_of(cols, rows, lim=40):
    """Tabela pequena para o modelo + lista de valores {{vN}} para o servidor substituir."""
    vals, lines = [], []
    for r in rows[:lim]:
        cells = []
        for c, v in zip(cols, r):
            if isinstance(v, (int, float)):
                vals.append(fmt_num(v))
                cells.append(f"{seguranca.dado(c, 40)}={{{{v{len(vals)}}}}}")
            else:
                cells.append(f"{seguranca.dado(c, 40)}={seguranca.dado(v)}")
        lines.append(", ".join(cells))
    return vals, "\n".join(lines)


def fill(text, vals):
    return re.sub(r"\{\{v(\d+)\}\}",
                  lambda m: vals[int(m.group(1)) - 1] if 0 < int(m.group(1)) <= len(vals) else "?", text)
```

Se mesmo assim o modelo digitar um número, a conferência pega. Ano não conta:

```python
def loose_numbers(text):
    """Números digitados fora dos marcadores (anos e ordinais pequenos não contam)."""
    t = re.sub(r"\{\{v\d+\}\}", "", text)
    t = re.sub(r"\[[^\]]*\]", "", t)
    return [n for n in re.findall(r"(?<![\w{])\d[\d.,]*\d|(?<![\w{])\d{2,}", t)
            if not re.fullmatch(r"(19|20)\d{2}", n)]
```

Resposta com número solto é refeita uma vez, sem cobrar. Se vier de novo, é barrada.

## 2. Célula de planilha é dado, nunca instrução

Todo texto que sai da planilha a caminho do modelo passa por aqui: uma linha só, curto, sem marcas que imitam papel de sistema. Se parecer uma ordem, a célula inteira some.

```python
def dado(v, lim=80):
    """Uma célula de texto ou nome de coluna a caminho do modelo: uma linha só, curta, sem marcas de papel.
    Se parecer uma ordem para o assistente, a célula inteira vira "[texto removido]"."""
    t = _PAPEL.sub(" ", _CTRL.sub(" ", str(v)))
    t = " ".join(t.split())[:lim]
    return REMOVIDO if suspeito(t) else t
```

## 3. O banco não consegue ler arquivo nem rede

Esta é a correção da falha mais grave que achei. Antes eu confiava só numa checagem do texto da consulta. Agora o próprio banco abre sem essa capacidade, somente leitura. Mesmo que uma consulta maliciosa passe pela checagem, o banco recusa.

```python
class conexao:
    def __init__(self, project_id: str, escrita: bool = False, externo: bool = False):
        # externo=False (padrão): o DuckDB fica proibido de ler arquivo do servidor ou a rede,
        # mesmo que um SQL passe pela checagem. Só a carga de tabela liga, para ler o próprio arquivo temporário.
        self.pid, self.escrita, self.externo = project_id, escrita, externo

    def __enter__(self):
        ...
        self.con = duckdb.connect(project_db(self.pid), read_only=not self.escrita,
                                  config={"enable_external_access": bool(self.externo)})
```

Cada projeto é um arquivo de banco separado. Uma consulta não tem como alcançar o projeto de outra conta, porque ele não está naquela conexão.

## 4. Provedor que treina com os dados não recebe dado de cliente

A fila de modelos de cada tarefa é filtrada antes de qualquer chamada. `app_ficticio` só é verdadeiro nas demos, que usam dados inventados.

```python
def fila(tarefa, app_ficticio, sem_pago=False):
    """Fila efetiva: tira quem não tem chave, quem treina (se o dado é real) e, se pedido, quem é pago."""
    out = []
    for mid in ROTAS[tarefa]:
        if PROV[M[mid]["prov"]]["treina"] and not app_ficticio:
            continue
        if sem_pago and not M[mid]["gratis"]:
            continue
        if tem_chave(mid):
            out.append(mid)
    return out
```

## 5. Nome de coluna nunca vira comando

O cabeçalho da planilha é texto do usuário. Na carga, todo nome de coluna entra entre aspas, com as aspas internas dobradas:

```python
q = lambda c: '"' + c.replace('"', '""') + '"'
```

Uma linha. A falta dela era a segunda falha grave: um cabeçalho montado de propósito virava comando no banco.

---

O contexto de cada trecho está em [como a LLM trabalha](como-a-llm-trabalha.md) e em [segurança](seguranca.md).
