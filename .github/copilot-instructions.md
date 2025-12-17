<!-- Copilot instructions tailored to this repo (generated/updated) -->
# Copilot / AI agent instructions — myproject1

Purpose
- Help contributors and AI agents understand this small ETL project layout and common patterns so changes are safe and predictable.

Big picture
- Simple local ETL: CSV files land in `landing/` -> processed by the ingest notebook `script/ingestao.ipynb` -> persisted into a DuckDB file under `script/` (`dados_duckdb.db` / WAL present).
- Data flow is explicit and file-local (no remote services). Modifications mostly touch CSV parsing, SQL schema in the notebook, or DuckDB usage.

Key files
- `landing/` — drop CSVs here. Files use `;` as field separator in examples.
- `script/ingestao.ipynb` — canonical ingestion logic (pandas + duckdb). See cells that: open `dados_duckdb.db`, read `../landing/{arquivo}` with `sep=';'`, create table `bronze_produtos`, and insert DataFrame into DuckDB.
- `script/dados_duckdb.db` (and `*.wal`) — DB file used by the notebook. Avoid concurrent writers.

Patterns & conventions (project-specific)
- CSVs are read with `pd.read_csv('../landing/{arquivo}', sep=';')` — use relative path from `script/`.
- Schema is defined in SQL inside the notebook: `CREATE TABLE IF NOT EXISTS bronze_produtos (...)` with columns: `NATBR, MAKTX, WERKS, MAINS, LABST, data_ingestao, nome_arquivo`.
- Inserts use DuckDB SQL over in-memory DataFrames, e.g. `con.execute("INSERT INTO bronze_produtos SELECT * FROM df")`.
- This repo uses notebooks for core logic instead of standalone scripts. If converting to a script, preserve the relative paths and the DuckDB connection call: `duckdb.connect(database='dados_duckdb.db', read_only=False)`.

Developer workflows (discoverable commands)
- Install runtime dependency used by the notebook: `pip install duckdb pandas`.
- Run the notebook interactively in VS Code / Jupyter to step through ingestion cells. The notebook opens with imports and a DuckDB connection.
- To run programmatically, convert the notebook to a script (e.g., `jupyter nbconvert --to script script/ingestao.ipynb`) then run `python script/ingestao.py` after verifying paths.

Integration cautions
- DuckDB database file is local: ensure no concurrent process truncates or corrupts `script/dados_duckdb.db` or its WAL file. Prefer single-process writes.
- Notebook writes are idempotent only if SQL and inserts are guarded; the current pattern unconditionally inserts — consider deduplication if re-running ingest.

Examples (from repo)
- CSV read: `df = pd.read_csv(f'../landing/{arquivo}', sep=';')`
- Create table SQL (in-notebook):
  CREATE TABLE IF NOT EXISTS bronze_produtos (
    NATBR VARCHAR,
    MAKTX VARCHAR,
    WERKS VARCHAR,
    MAINS VARCHAR,
    LABST VARCHAR,
    data_ingestao TIMESTAMP,
    nome_arquivo VARCHAR
  )
- Insert example: `con.execute("INSERT INTO bronze_produtos SELECT * FROM df")`

What to ask the agent
- When modifying ingestion, ask: "Will this change break relative paths or the DuckDB schema?" Provide the target CSV sample and expected schema.
- For schema changes, show example CSV header + 5 rows and indicate desired column types.

If you find outdated or missing conventions here, update this file and run the notebook to validate the change.

— End
