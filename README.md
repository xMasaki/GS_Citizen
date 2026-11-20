# AcessaCidade

> **FIAP · Global Solution 2026.2 · CitizenTech · Engenharia de Software 4º ano · ODS 11**
>
> Plataforma web para o cidadão **registrar barreiras de acessibilidade nas calçadas** (buracos, pisos
> quebrados, rampas irregulares, obstáculos), com **triagem automática inicial da foto**, protocolo para
> acompanhamento, assistente em linguagem simples e painel de indicadores por bairro.

> **Aviso:** o AcessaCidade é um projeto acadêmico. **Não é um serviço oficial do governo** nem tem
> parceria com órgão público. O status "Resolvida" é registro da equipe, não confirmação de órgão público.
> A triagem da foto é **apoio inicial** e sempre exige revisão humana; o sistema **não afirma** que uma
> calçada ou rota é acessível. Os dados de demonstração são **simulados** (cidade fictícia "Cidade Exemplo").

## Ficha Técnica do MVP (resumo)

Versão completa (estrutura da pág. 20 do PDF oficial): **[docs/FICHA_TECNICA_MVP.md](docs/FICHA_TECNICA_MVP.md)**.

| Campo | Resposta |
|---|---|
| Nome do produto | AcessaCidade |
| Equipe | [PREENCHER: nome da equipe] — [PREENCHER: nomes e RMs dos integrantes] |
| Cidadão beneficiado | Pessoas com deficiência (cadeirantes, pessoas cegas ou com baixa visão), pessoas idosas e pessoas com mobilidade reduzida que se deslocam a pé, e quem as acompanha |
| Território de atuação | [PREENCHER: cidade/bairros onde a equipe observou o problema] |
| Problema, em uma frase | Barreiras nas calçadas impedem ou tornam arriscado o deslocamento dessas pessoas, e não há um jeito simples de avisar quem pode resolver nem de acompanhar o que acontece depois |
| Proposta de valor | Registrar uma barreira em poucos minutos pelo celular, sem app e sem cadastro, e acompanhar pelo protocolo; para quem resolve, denúncias padronizadas, com foto triada e explicada, priorizáveis em painel |
| Arquitetura | SPA (HTML/CSS/JS) servida por backend FastAPI + SQLite; visão computacional OpenCV clássica; assistente com LLM via OpenRouter + modo por regras; pipeline Raw→Bronze→Silver→Gold (DuckDB, Airflow); robô UiPath; Docker + GitHub Actions + Terraform (Azure ACI) + Prometheus/Grafana; placa física com QR Code |
| Modelos de IA | OpenCV clássico (sem rede neural; pontuação heurística, não probabilidade) · LLM configurável via OpenRouter (padrão `openai/gpt-4o-mini`) · modo alternativo por regras |
| ODS | 11 (metas 11.7, 11.2, 11.3) · secundários 10 (10.2) e 9 (9.1) |
| URL do MVP | [PREENCHER: URL pública do MVP] |
| Repositório | [PREENCHER: URL do repositório público no GitHub] |
| Vídeo-pitch | [PREENCHER: link do vídeo] |

## O que é

Um **único produto** integrado: cada uma das 7 disciplinas entrega um componente do mesmo sistema
(nada de sete sistemas independentes). O cidadão abre o site (ou lê o QR Code da placa no ponto de
ônibus), registra a barreira com endereço, tipo, impacto, descrição e foto opcional, recebe um protocolo
`AC-AAAA-NNNNNN` e acompanha o status. A foto passa por uma triagem explicável em 7 etapas visíveis
(original, cinza, filtrada, contraste, bordas, máscara, contornos) com resultado "Possível buraco
identificado", "Nenhum indício suficiente de buraco" ou "Análise inconclusiva".

- Arquitetura: [docs/ARQUITETURA_GERAL.md](docs/ARQUITETURA_GERAL.md)
- Diagramas (Mermaid) e integrações testadas × manuais: [docs/DIAGRAMA_INTEGRACAO.md](docs/DIAGRAMA_INTEGRACAO.md)
- Contrato entre componentes: [docs/CONTRATOS_DE_INTEGRACAO.md](docs/CONTRATOS_DE_INTEGRACAO.md)
- Dicionário de dados consolidado (com classificação LGPD): [docs/DICIONARIO_DE_DADOS.md](docs/DICIONARIO_DE_DADOS.md)
- Inventário do que ainda falta preencher (678 marcadores agrupados por categoria e prioridade): [docs/INVENTARIO_PREENCHER.md](docs/INVENTARIO_PREENCHER.md)
- Preencher nome da equipe, integrantes e links de uma vez: edite [dados_entrega.yaml](dados_entrega.yaml) e rode `.\.venv\Scripts\python.exe tools\preencher_marcadores.py --dry-run` (preview + diff; sem `--dry-run` pede confirmação, faz backup e gera `docs/RELATORIO_PREENCHIMENTO.md`). Campo vazio não troca nada.

## Status honesto por disciplina

Verificação real em **15/11/2026**, nesta máquina (Windows 11, Python 3.11): `python -m pytest` → **492 passed**;
teste ponta a ponta com Playwright + Edge contra o backend real (com a Gold publicada pelo pipeline) → **201/201 verificações OK**; Gitleaks
(diretório) sem vazamentos; Semgrep 0 achados; Trivy (dependências, Dockerfile, Terraform) 0 HIGH/CRITICAL;
`terraform validate` Success e `terraform test` 3 passed. **Não** foram executados (sem ferramenta ou
credencial no ambiente): Docker, Airflow, Azure, GitHub Actions, UiPath Studio, Onshape, fabricação, LLM
real, vídeos, PDFs finais com links, MakerWorld.

Classificação: **CONCLUÍDA E TESTADA** · **IMPLEMENTADA, MAS NÃO TESTADA** · **PARCIALMENTE IMPLEMENTADA** · **PENDENTE DE AÇÃO MANUAL**.

| Disciplina | Entrega | Situação | Observação |
|---|---|---|---|
| Base do MVP | Frontend SPA + backend FastAPI + SQLite (todas as telas e endpoints mínimos) | CONCLUÍDA E TESTADA | 56 testes da API + E2E 199/199; capturas em [`evidencias/mvp_local/`](evidencias/mvp_local/LEIAME.md) |
| Base do MVP | MVP em URL pública | PENDENTE DE AÇÃO MANUAL | Depende do deploy na Azure (disciplina 06) |
| 01 · Generative AI | AcessaCidade Helper (prompts, contexto, guardrails, anti-alucinação, fallback) | CONCLUÍDA E TESTADA | 145 testes; integrado em `POST /api/assistente` e `#/assistente` (modo por regras) |
| 01 · Generative AI | Respostas com LLM real (OpenRouter) e exemplos reais | PENDENTE DE AÇÃO MANUAL | Exige `OPENROUTER_API_KEY` da equipe; script `examples/gerar_exemplos_llm.py` pronto |
| 01 · Generative AI | Requisitos da disciplina e PDF | PENDENTE DE AÇÃO MANUAL | O PDF oficial está "TBD": confirmar com o professor; PDF de rascunho é gerado |
| 02 · Big Data | Pipeline Raw→Bronze→Silver→Gold + runner local + notebook | CONCLUÍDA E TESTADA | 107 testes; `--sem-api`: RAW 271 → BRONZE 260 → SILVER 250, 10/10 checagens; execução real contra a API publicando a Gold no painel |
| 02 · Big Data | DAG Airflow (6 tarefas) | IMPLEMENTADA, MAS NÃO TESTADA | Estrutura testada com stub; Airflow real exige Docker/WSL2 |
| 02 · Big Data | Diagrama de arquitetura e dicionário de dados | CONCLUÍDA E TESTADA | Gerados no PDF de rascunho (diagramas Mermaid desenhados) |
| 02 · Big Data | Prints/vídeo do pipeline no Airflow | PENDENTE DE AÇÃO MANUAL | Ver `02-Big-Data-Architecture/docs/EVIDENCIAS_NECESSARIAS.md` |
| 02 · Big Data | PDF único + link do repositório | PARCIALMENTE IMPLEMENTADA | Gerador funciona; faltam nomes/RMs e links |
| 03 · BI & Startup | Lean Canvas (9 blocos) e modelo de negócio | PARCIALMENTE IMPLEMENTADA | Completos como **hipóteses não validadas**; a validação depende da equipe |
| 03 · BI & Startup | Roteiro do pitch, storyboard, cenas, roteiro de teste de usabilidade | IMPLEMENTADA, MAS NÃO TESTADA | Documentos prontos; sem teste com usuários reais (7 testes automáticos cobrem só os geradores do canvas e das planilhas) |
| 03 · BI & Startup | Vídeo-pitch (≤ 5 min, todos participam) e evidência de valor | PENDENTE DE AÇÃO MANUAL | Gravação e testes com cidadãos pela equipe |
| 03 · BI & Startup | PDF único com links clicáveis | PARCIALMENTE IMPLEMENTADA | Gerador funciona (15 páginas de rascunho); faltam links e dados da equipe |
| 04 · Visão | Triagem OpenCV (13 passos, 3 classes, 7 imagens intermediárias, privacidade) | CONCLUÍDA E TESTADA | 45 testes; avaliada **somente com imagens sintéticas** — nenhuma acurácia é declarada |
| 04 · Visão | Documento das transformações e dados extraídos | CONCLUÍDA E TESTADA | [`04-Applied-Computer-Vision/docs/DOCUMENTACAO_TECNICA_TRANSFORMACOES.md`](04-Applied-Computer-Vision/docs/DOCUMENTACAO_TECNICA_TRANSFORMACOES.md); PDF de rascunho gerado |
| 04 · Visão | Fotos reais + comparação com classificação manual | PENDENTE DE AÇÃO MANUAL | Coleta em campo com cuidado LGPD; script `04-Applied-Computer-Vision/scripts/avaliar_comparacao.py` pronto |
| 04 · Visão | Vídeo (≤ 5 min) e PDF final com links | PENDENTE DE AÇÃO MANUAL | Roteiro em [`04-Applied-Computer-Vision/docs/ROTEIRO_VIDEO_VISAO.md`](04-Applied-Computer-Vision/docs/ROTEIRO_VIDEO_VISAO.md) |
| 05 · RPA | Assessment, BPM AS-IS/TO-BE, RPA × API, business case, planilha simulada | CONCLUÍDA E TESTADA | 49 testes (artefatos, planilhas, BPMN); valores do business case são **premissas** |
| 05 · RPA | Projeto UiPath (`Main.xaml` e workflows) | IMPLEMENTADA, MAS NÃO TESTADA | XAML verificado estaticamente; **não** aberto nem executado no UiPath Studio |
| 05 · RPA | Prints, vídeo (≤ 3 min) e ZIP via Teams | PENDENTE DE AÇÃO MANUAL | `05-Robotic-Process-Automation/scripts/gerar_zip_entrega.ps1` pronto; instruções em [`05-Robotic-Process-Automation/docs/INSTRUCOES_ZIP_ENTREGA.md`](05-Robotic-Process-Automation/docs/INSTRUCOES_ZIP_ENTREGA.md) |
| 06 · DevOps | Dockerfile (não root, healthcheck) e Docker Compose | IMPLEMENTADA, MAS NÃO TESTADA | Verificação estática + Trivy config 0; sem `docker build` nesta máquina |
| 06 · DevOps | Workflow GitHub Actions (pytest, Gitleaks, Semgrep, Trivy, build, deploy OIDC) | IMPLEMENTADA, MAS NÃO TESTADA | Cada ferramenta rodada localmente com resultado limpo; o workflow só roda após o push |
| 06 · DevOps | Terraform (RG, ACR, ACI, identidade, logs) | IMPLEMENTADA, MAS NÃO TESTADA | `fmt`/`validate` OK e `terraform test` 3 passed (provider simulado); nunca aplicado na Azure |
| 06 · DevOps | Prometheus + Grafana (4 sinais de ouro) e alerta | IMPLEMENTADA, MAS NÃO TESTADA | `/metrics` testado; stack e disparo do alerta dependem de Docker |
| 06 · DevOps | URL pública, execução barrada/corrigida, prints, custo real, PDF final | PENDENTE DE AÇÃO MANUAL | Ver [`06-Secure-DevOps-Cloud/docs/PUBLICACAO_AZURE.md`](06-Secure-DevOps-Cloud/docs/PUBLICACAO_AZURE.md) e [`DEMONSTRACAO_EXECUCAO_BARRADA.md`](06-Secure-DevOps-Cloud/docs/DEMONSTRACAO_EXECUCAO_BARRADA.md) |
| 07 · Maker Lab | Arquivos de fabricação (DXF, SVG, STL, 3MF, STEP) e fontes CAD paramétricas | CONCLUÍDA E TESTADA | 45 testes; STL estanques; peças laser e 3D |
| 07 · Maker Lab | CAD do original e redesign lado a lado | PARCIALMENTE IMPLEMENTADA | O original usa **medidas estimadas** até a medição em campo; Onshape pendente |
| 07 · Maker Lab | Diagnóstico no local, fabricação, fotos antes/depois, QR definitivo | PENDENTE DE AÇÃO MANUAL | Exige visita, laboratório e a URL pública (o QR não foi gerado com URL inventada) |
| 07 · Maker Lab | Publicação no MakerWorld, bônus e PDF final | PENDENTE DE AÇÃO MANUAL | Ver [`07-Project-Based-Maker-Lab/docs/MAKERWORLD_PUBLICACAO.md`](07-Project-Based-Maker-Lab/docs/MAKERWORLD_PUBLICACAO.md) e [`BONUS_PECA_TERCEIROS.md`](07-Project-Based-Maker-Lab/docs/BONUS_PECA_TERCEIROS.md) |

O detalhe de cada pendência (programa, arquivo, menu, passo a passo, resultado esperado e como conferir)
está no `PENDENCIAS_E_ACOES_MANUAIS.txt` de cada pasta e no guia
**[PASSO_A_PASSO_PARA_CONCLUIR_O_PROJETO.txt](PASSO_A_PASSO_PARA_CONCLUIR_O_PROJETO.txt)**.

## Início rápido

Pré-requisito: **Python 3.11**. Não é preciso Docker para rodar localmente.

### Windows (PowerShell), na raiz do repositório

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt   # ou requirements.txt (só a aplicação)
Copy-Item .env.example .env                                         # opcional: ajuste as variáveis
.\.venv\Scripts\python.exe -m uvicorn acessacidade_api.main:app --app-dir app/backend --host 127.0.0.1 --port 8000
```

Alternativa: `.\scripts\run_local.ps1` (carrega o `.env` e define o `PYTHONPATH`; `-Instalar` cria o venv).
Se a execução de scripts estiver bloqueada: `powershell -ExecutionPolicy Bypass -File .\scripts\run_local.ps1`.

### Linux / macOS

```bash
python3.11 -m venv .venv
.venv/bin/python -m pip install -r requirements-dev.txt
cp .env.example .env
./scripts/run_local.sh            # ou: ./scripts/run_local.sh --instalar
```

Depois abra: `http://127.0.0.1:8000/` (aplicação) · `http://127.0.0.1:8000/docs` (API) · `http://127.0.0.1:8000/health`.
Com `SEED_DEMO_DATA=true` (padrão) o banco começa com denúncias **simuladas**. Sem `OPENROUTER_API_KEY`
o assistente responde em modo básico (`fonte: fallback`). Sem `ADMIN_TOKEN`/`PIPELINE_TOKEN`, a área da
equipe e as rotas do pipeline respondem 403 (desativadas).

Docker (preparado, não executado nesta máquina): `docker compose up -d --build` (aplicação em `:8000`) e
`docker compose --profile monitoring up -d --build` (com Prometheus, Alertmanager e Grafana).

Pipeline de dados local: `$env:PYTHONPATH = "02-Big-Data-Architecture/src"; .\.venv\Scripts\python.exe -m acessacidade_pipeline --sem-api`
(detalhes em `02-Big-Data-Architecture/README.md`).

## Testes

```powershell
.\.venv\Scripts\python.exe -m pytest          # na raiz: 492 testes (app/backend, 01, 02, 03, 04, 05, 06, 07)
```

Teste ponta a ponta da interface (Playwright com o Microsoft Edge instalado; o servidor precisa estar rodando):

```powershell
# Terminal 1 — servidor com um token de equipe gerado na hora (nunca grave o token em arquivo)
$env:ADMIN_TOKEN = .\.venv\Scripts\python.exe -c "import secrets; print(secrets.token_urlsafe(24))"
.\.venv\Scripts\python.exe -m uvicorn acessacidade_api.main:app --app-dir app/backend --host 127.0.0.1 --port 8000

# Terminal 2 — o mesmo valor, só em variável de ambiente (o script não aceita token por argumento)
$env:ACESSACIDADE_E2E_ADMIN_TOKEN = "<o mesmo valor de ADMIN_TOKEN do terminal 1>"
.\.venv\Scripts\python.exe app/frontend/tests_e2e/e2e_acessacidade.py --url http://127.0.0.1:8000 --shots evidencias/mvp_local
```

Sem `ACESSACIDADE_E2E_ADMIN_TOKEN`, o E2E apenas confere que um token inválido é recusado. Resultado
esperado: `RESULTADO: N/N verificações OK; 0 falha(s).` (código de saída 0). O E2E cria 2 denúncias de
teste identificadas no texto; rode-o num banco de demonstração.

## Estrutura de pastas

```text
.
├── app/
│   ├── backend/                     # API FastAPI (acessacidade_api) + testes test_api_*.py
│   └── frontend/                    # SPA (index.html, css/, js/) + tests_e2e/
├── 01-Generative-AI-Engineering/    # AcessaCidade Helper (src/acessacidade_helper, prompts/, examples/)
├── 02-Big-Data-Architecture/        # pipeline (src/acessacidade_pipeline), dags/, airflow/, notebooks/, data/ (aponta para data/ da raiz)
├── 03-Business-Intelligence-Startup/# Lean Canvas, modelo de negócio, pitch, dashboard (Excel)
├── 04-Applied-Computer-Vision/      # visão (src/acessacidade_vision), scripts/, samples/, results/
├── 05-Robotic-Process-Automation/   # projeto UiPath, planilhas, BPMN, relatório
├── 06-Secure-DevOps-Cloud/          # docker/, terraform/, monitoring/, github-actions/, docs/
├── 07-Project-Based-Maker-Lab/      # cad/, laser-cut/, 3d-print/, docs/
├── .github/workflows/               # CI/CD DevSecOps (ci-devsecops.yml)
├── .semgrep/                        # regras Semgrep customizadas + testes das regras
├── infra/                           # scripts Azure CLI e template YAML do ACI
├── data/                            # samples/ (simulados e sintéticos), processed/ (Gold), exports/, runtime/ (não versionado)
├── docs/                            # arquitetura, diagramas, contrato, dicionário, Ficha Técnica
├── evidencias/                      # provas reais da entrega (mvp_local/ já existe)
├── scripts/                         # run_local.ps1 / run_local.sh, gerador de tráfego
├── dados_entrega.yaml               # equipe, integrantes e links finais (vazio até a equipe preencher)
├── tools/                           # gerar_todos_pdfs.ps1, gerar_pdf_markdown.py, preencher_marcadores.py
├── docker-compose.yml   .dockerignore   .gitleaks.toml   pytest.ini
├── requirements.txt   requirements-dev.txt   .env.example   .gitignore
└── PASSO_A_PASSO_PARA_CONCLUIR_O_PROJETO.txt
```

## Documentação por disciplina

| Disciplina | Componente | README | Documento de entrega | Pendências |
|---|---|---|---|---|
| 01 · Generative AI for Engineering | Inteligência | [README](01-Generative-AI-Engineering/README.md) | [RELATORIO_ENTREGA_GENAI.md](01-Generative-AI-Engineering/docs/RELATORIO_ENTREGA_GENAI.md) | [PENDENCIAS](01-Generative-AI-Engineering/PENDENCIAS_E_ACOES_MANUAIS.txt) |
| 02 · Big Data Architecture & Data Integration | Dados | [README](02-Big-Data-Architecture/README.md) | [RELATORIO_ENTREGA_BIGDATA.md](02-Big-Data-Architecture/docs/RELATORIO_ENTREGA_BIGDATA.md) | [PENDENCIAS](02-Big-Data-Architecture/PENDENCIAS_E_ACOES_MANUAIS.txt) |
| 03 · Business Intelligence & Startup Development | Negócio e pitch | [README](03-Business-Intelligence-Startup/README.md) | [DOCUMENTO_DE_ENTREGA_BI.md](03-Business-Intelligence-Startup/docs/DOCUMENTO_DE_ENTREGA_BI.md) | [PENDENCIAS](03-Business-Intelligence-Startup/PENDENCIAS_E_ACOES_MANUAIS.txt) |
| 04 · Applied Computer Vision | Visão | [README](04-Applied-Computer-Vision/README.md) | [RELATORIO_ENTREGA_VISAO.md](04-Applied-Computer-Vision/docs/RELATORIO_ENTREGA_VISAO.md) | [PENDENCIAS](04-Applied-Computer-Vision/PENDENCIAS_E_ACOES_MANUAIS.txt) |
| 05 · Robotic Process Automation | Automação | [README](05-Robotic-Process-Automation/README.md) | [RELATORIO_RPA.md](05-Robotic-Process-Automation/docs/RELATORIO_RPA.md) | [PENDENCIAS](05-Robotic-Process-Automation/PENDENCIAS_E_ACOES_MANUAIS.txt) |
| 06 · Secure DevOps Tools & Cloud Computing | Produção | [README](06-Secure-DevOps-Cloud/README.md) | [RELATORIO_ENTREGA_DEVOPS.md](06-Secure-DevOps-Cloud/docs/RELATORIO_ENTREGA_DEVOPS.md) | [PENDENCIAS](06-Secure-DevOps-Cloud/PENDENCIAS_E_ACOES_MANUAIS.txt) |
| 07 · Project-Based Maker Lab | Físico | [README](07-Project-Based-Maker-Lab/README.md) | [RELATORIO_ENTREGA_MAKER.md](07-Project-Based-Maker-Lab/docs/RELATORIO_ENTREGA_MAKER.md) | [PENDENCIAS](07-Project-Based-Maker-Lab/PENDENCIAS_E_ACOES_MANUAIS.txt) |

Backend: [app/backend/README.md](app/backend/README.md) · Infraestrutura manual na Azure: [infra/README.md](infra/README.md) ·
Evidências: [evidencias/README.md](evidencias/README.md).

Entrega de todas as disciplinas até **19/11/2026 às 23h55**, no portal de cada disciplina (RPA: ZIP pela
atividade do Teams). Nenhum envio foi feito por este repositório.

## PDFs de entrega

```powershell
.\tools\gerar_todos_pdfs.ps1 -Saida "$env:TEMP\acessacidade_pdfs"     # rascunhos, fora do repositório
.\tools\gerar_todos_pdfs.ps1 -RepoUrl [PREENCHER: URL do repositório] -Estrito   # versão final em entregas\pdf
```

O script chama o gerador de cada disciplina (01, 02, 03, 04 e 05 têm o seu; 06 e 07 usam
`tools/gerar_pdf_markdown.py`, que desenha os diagramas Mermaid e mantém os links clicáveis) e grava tudo
numa pasta. Links de 01, 03 e 04 vêm do `docs/links_entrega.yaml` de cada uma dessas pastas (ex.: `01-Generative-AI-Engineering/docs/links_entrega.yaml`). Decisão de versionamento:
os **PDFs finais** em `entregas/pdf/` podem ser versionados (cópia de referência no repositório público);
**rascunhos** com `[PREENCHER]` não devem ir para o Git — gere-os fora do repositório com `-Saida`.
Testado em 15/11/2026: os 7 PDFs de rascunho foram gerados com páginas e texto (conferidos com `pypdf`).

## Segurança e LGPD

- Nenhum segredo no código ou no Git: tudo por variável de ambiente (`.env` local, ignorado) ou GitHub
  Secrets. `.env.example` lista as variáveis sem valores secretos.
- Nome do denunciante opcional, guardado só no SQLite e **nunca** retornado pela API nem exportado.
- Fotos validadas por decodificação, re-codificadas **sem EXIF/GPS**, redimensionadas e com **rostos borrados**;
  a imagem bruta não é guardada.
- Logs em JSON sem nome, chave de API ou conteúdo de fotos. Camada Gold sem dados pessoais nem texto livre.
- Cabeçalhos de segurança (CSP restritiva, `nosniff`, `no-referrer`, `DENY`), rate limit, limite de upload,
  tokens comparados em tempo constante, container não root.
- Detalhes: [docs/ARQUITETURA_GERAL.md §5](docs/ARQUITETURA_GERAL.md), `06-Secure-DevOps-Cloud/docs/LGPD_MAPEAMENTO_DADOS_PESSOAIS.md`
  e `06-Secure-DevOps-Cloud/docs/RESPONSABILIDADE_COMPARTILHADA.md`.

## Licença

[PREENCHER: licença escolhida pela equipe (ex.: MIT) e arquivo LICENSE]. Até a escolha, nenhuma licença
de uso é concedida. Peças de terceiros do Maker Lab mantêm a licença e o crédito de seus autores.

## Equipe

| Nome | RM | Papel |
|---|---|---|
| [PREENCHER] | [PREENCHER] | [PREENCHER] |
| [PREENCHER] | [PREENCHER] | [PREENCHER] |
| [PREENCHER] | [PREENCHER] | [PREENCHER] |
