# Airflow 2.10.5 para a DAG `acessacidade_elt_pipeline`

Esta pasta sobe um Apache Airflow 2.10.5 (Python 3.11) em **modo standalone** para orquestrar
o pipeline ELT do AcessaCidade (`../dags/acessacidade_elt_pipeline.py`).

> **Atenção — compatibilidade**
> - O Airflow 2.10.5 **não roda nativamente no Windows** (depende de recursos POSIX). No Windows,
>   use **Docker Desktop (backend WSL2)** ou instale dentro do **WSL2 (Ubuntu)**.
> - O Airflow 2.10.5 suporta Python **3.8 a 3.12**. **Não use Python 3.13/3.14** para instalar o
>   Airflow 2.10.5: a instalação falha ou fica sem constraints válidas.
> - O modo standalone (SQLite + SequentialExecutor) é para demonstração e desenvolvimento, não
>   para produção.

Sem Airflow, as mesmas tarefas rodam em sequência com o runner local:
`python -m acessacidade_pipeline --sem-api` (ver `../docs/INSTRUCOES_EXECUCAO.md`).

---

## Opção A — Docker Desktop (Windows, macOS ou Linux)

Pré-requisito: Docker Desktop instalado e em execução (no Windows, com WSL2 habilitado).

```powershell
cd C:\caminho\para\GS7-4ESPR\02-Big-Data-Architecture\airflow
# com o .env da raiz (traz PIPELINE_TOKEN):
docker compose --env-file ..\..\.env -f docker-compose.airflow.yml up -d
# sem .env (o pipeline roda só com a fonte simulada):
docker compose -f docker-compose.airflow.yml up -d

# acompanhar a inicialização (a 1ª vez instala o duckdb e cria o banco do Airflow; ~1–3 min)
docker compose -f docker-compose.airflow.yml logs -f airflow

# senha do usuário "admin" (gerada pelo próprio Airflow)
docker compose -f docker-compose.airflow.yml exec airflow cat /opt/airflow/standalone_admin_password.txt
```

Abra **http://127.0.0.1:8080** (a porta só é publicada em 127.0.0.1), entre com `admin` e a senha
acima, ative (unpause) a DAG `acessacidade_elt_pipeline` e clique em **Trigger DAG**.

O que o compose faz:

| Item | Valor |
|---|---|
| Imagem | `apache/airflow:2.10.5-python3.11` |
| Comando | `standalone` (webserver + scheduler + triggerer + usuário admin) |
| Volumes | `../dags → /opt/airflow/dags`, `../src → /opt/airflow/src`, `../../data → /opt/airflow/acessacidade_data` |
| `_PIP_ADDITIONAL_REQUIREMENTS` | `duckdb==1.1.3` (pandas 2.1.4, httpx 0.27.0 e numpy 1.26.4 já vêm na imagem — todas as versões são as das constraints 2.10.5/py3.11) |
| `PYTHONPATH` | `/opt/airflow/src` (o DAG também acrescenta `../src` ao `sys.path`) |
| `PIPELINE_DATA_DIR` | `/opt/airflow/acessacidade_data` (= pasta `data/` do repositório) |
| `ACESSACIDADE_API_URL` | `http://host.docker.internal:8000` (API rodando na máquina hospedeira); pode ser trocada pela variável `ACESSACIDADE_API_URL_AIRFLOW` no `.env` (ex.: URL pública da API) |
| `PIPELINE_TOKEN` | lido do `.env` (`--env-file`); vazio = extração/publicação via API desativadas, com aviso no log |
| Porta | `127.0.0.1:8080:8080` |

Executar sem a API (somente a fonte simulada): em **Trigger DAG w/ config**, use
`{"usar_api": false}`.

Linux: antes do `up`, rode `export AIRFLOW_UID=$(id -u)` para que os arquivos gerados em `data/`
pertençam ao seu usuário.

Parar: `docker compose -f docker-compose.airflow.yml down` (os resultados ficam em `data/processed/`).

## Opção B — pip com constraints (Linux nativo ou WSL2 Ubuntu)

```bash
cd /caminho/para/GS7-4ESPR
python3.11 -m venv .venv-airflow && source .venv-airflow/bin/activate
pip install "apache-airflow==2.10.5" \
  --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.10.5/constraints-3.11.txt"
pip install "duckdb==1.1.3" \
  --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.10.5/constraints-3.11.txt"

export AIRFLOW_HOME="$HOME/airflow-acessacidade"
export AIRFLOW__CORE__DAGS_FOLDER="$PWD/02-Big-Data-Architecture/dags"
export AIRFLOW__CORE__LOAD_EXAMPLES=false
export PYTHONPATH="$PWD/02-Big-Data-Architecture/src"
export PIPELINE_DATA_DIR="$PWD/data"
export ACESSACIDADE_API_URL="http://localhost:8000"      # opcional
export PIPELINE_TOKEN="<mesmo valor do .env da API>"     # opcional

airflow standalone        # UI em http://localhost:8080, senha em $AIRFLOW_HOME/standalone_admin_password.txt
# ou, sem UI, uma execução de teste completa da DAG:
airflow dags test acessacidade_elt_pipeline
```

## Verificação

- Na UI, a visão **Graph** deve mostrar 6 tarefas em linha:
  `extract_and_load_raw → transform_bronze → curate_silver → build_gold → quality_report → publish_gold`.
- Os logs de cada tarefa mostram as contagens (linhas raw/bronze/silver, agregados Gold, verificações de qualidade).
- Após a execução, `data/processed/gold/gold_snapshot.json` e `data/processed/relatorio_qualidade.md` são atualizados.

Este compose **não foi executado** no ambiente de desenvolvimento (sem Docker disponível); o
arquivo foi validado apenas sintaticamente (YAML). Ver `../PENDENCIAS_E_ACOES_MANUAIS.txt`.
