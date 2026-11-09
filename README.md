# BRAN Database Template

Template canônico da **BRAN Org** para disponibilização, validação e preservação digital de bases de dados acadêmicas abertas (FAIR Data Packages).

Projetado especificamente para acervos científicos, anais de congressos e periódicos, este modelo substitui engines web duplicadas por um formato focado exclusivamente em dados, proveniência e ciência aberta.

---

## 1. Estrutura do Repositório

```text
├── .github/workflows/
│   └── data-ci.yml           # CI/CD: Auditoria CRONA no PR + Upload Zenodo no CalVer
├── data/
│   ├── sample_articles.json  # Matriz canônica em JSON estruturado
│   ├── sample_articles.csv   # Tabela RFC 4180 UTF-8 com BOM (Excel/R)
│   └── sample_articles.sqlite # Banco relacional com busca FTS5 integrada
├── schemas/
│   ├── article.v1.schema.json    # Schema canônico da BRAN Org
│   └── provenance.v1.schema.json # Schema de auditoria de proveniência
├── scripts/
│   ├── build_dataset.py      # Pipeline local: valida schema, compila CSV/SQLite e gera hashes
│   ├── crona_audit_engine.py # Motor de auditoria de metadados e DOIs da CRONA
│   └── crona_zenodo_sync.py  # Empacotador e sincronizador com a API do Zenodo
├── .gitignore
├── .zenodo.json              # Metadados de depósito no Zenodo
├── CITATION.cff              # Metadados de citação acadêmica
├── LICENSE                   # Licença dupla (Código MIT / Dados CC-BY-NC-SA 4.0)
└── provenance.json           # Manifesto criptográfico com hashes SHA-256
```

---

## 2. Como Usar Este Template para uma Nova Base

### Passo 1: Criar o Repositório
Crie o repositório na organização a partir deste template:
```bash
gh repo create BRAN-Org/<evento>-open-database --template BRAN-Org/bran-database-template --public
```

### Passo 2: Adicionar os Dados Coletados
Coloque o arquivo JSON dos artigos em `data/<evento>_articles.json` respeitando os campos mínimos de [`schemas/article.v1.schema.json`](schemas/article.v1.schema.json):
- `id` ou `doi`: Identificador persistente ou slug rastreável.
- `title`: Título do artigo.
- `authors`: Lista de autores.
- `year`: Ano da publicação.

### Passo 3: Compilar Formatos e Proveniência
Execute o script unificado de build:
```bash
python3 scripts/build_dataset.py
```
O script automaticamente:
1. Valida todos os registros contra o schema canônico.
2. Gera `data/<evento>_articles.csv` com UTF-8 BOM.
3. Compila `data/<evento>_articles.sqlite` com tabela `articles` indexada e tabela virtual `articles_fts` (Full-Text Search).
4. Recalcula os hashes SHA-256 e atualiza o manifesto `provenance.json`.

---

## 3. Como Consumir os Dados (Exemplos Rápidos)

### Python (pandas & sqlite3)
```python
import pandas as pd
import sqlite3

# Via CSV
df = pd.read_csv("data/sample_articles.csv")
print(df[["title", "year", "authors"]].head())

# Via SQLite3 (Busca Full-Text rápida)
conn = sqlite3.connect("data/sample_articles.sqlite")
res = conn.execute("SELECT title, year FROM articles_fts WHERE articles_fts MATCH 'preservacao'")
print(res.fetchall())
```

### R
```R
library(readr)
articles <- read_csv("data/sample_articles.csv")
head(articles)
```

### DuckDB
```sql
SELECT year, count(*) AS total
FROM 'data/sample_articles.csv'
GROUP BY year
ORDER BY year DESC;
```

---

## 4. Governança e Automação (CRONA Bot)

- **Em Pull Requests**: O workflow `data-ci.yml` executa a triagem da `@crona-bot`. Se houver inconsistências de schema ou sintaxe, o PR é reprovado com comentários de orientação.
- **Em Releases (`vYYYY.MM`)**: Quando uma tag no padrão **CalVer** (ex: `v2026.11`) é publicada, a CRONA empacota automaticamente os dados e realiza o upload para o Zenodo via API, preservando o DOI do acervo.

---

## 5. Licença

- **Código e Scripts**: GNU General Public License v3.0 (GPL-3.0).
- **Dados e Metadados**: Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0).
