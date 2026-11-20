# AcessaCidade

**FIAP · GS 2026.2 · 4ESPS · CitizenTech · ODS 11**

Plataforma web para o cidadão registrar barreiras de acessibilidade nas calçadas (buracos, pisos quebrados, rampas irregulares, obstáculos), com **triagem automática inicial da foto**, protocolo para acompanhamento, assistente em linguagem simples e painel de indicadores por bairro.

## Ficha Técnica do MVP (resumo)

| Campo | Resposta |
|---|---|
| Nome do produto | AcessaCidade |
| Equipe | Lucas Masaki Nagahama - RM553084 |
| Cidadão beneficiado | Pessoas com deficiência (cadeirantes, pessoas cegas ou com baixa visão), pessoas idosas e pessoas com mobilidade reduzida que se deslocam a pé, e quem as acompanha |
| Território de atuação | Inicialmente em alguns bairros metropolitanos |
| Problema, em uma frase | Barreiras nas calçadas impedem ou tornam arriscado o deslocamento dessas pessoas, e não há um jeito simples de avisar quem pode resolver nem de acompanhar o que acontece depois |
| Proposta de valor | Registrar uma barreira em poucos minutos pelo celular, sem app e sem cadastro, e acompanhar pelo protocolo; para quem resolve, denúncias padronizadas, com foto triada e explicada, priorizáveis em painel |
| Arquitetura | SPA (HTML/CSS/JS) servida por backend FastAPI + SQLite; visão computacional OpenCV clássica; assistente com LLM via OpenRouter + modo por regras; pipeline Raw→Bronze→Silver→Gold (DuckDB, Airflow); robô UiPath; Docker + GitHub Actions + Terraform (Azure ACI) + Prometheus/Grafana; placa física com QR Code |
| Modelos de IA | OpenCV clássico (sem rede neural; pontuação heurística, não probabilidade) · LLM configurável via OpenRouter (padrão `openai/gpt-4o-mini`) · modo alternativo por regras |
| ODS | 11 (metas 11.7, 11.2, 11.3) · secundários 10 (10.2) e 9 (9.1) |
| URL do MVP | http://102.133.197.125:8000/#/ |
| Repositório | https://github.com/xMasaki/GS_Citizen.git |


## O que é

Um único produto integrado: cada uma das 7 disciplinas entrega um componente do mesmo sistema. O cidadão abre o site ou lê o QR Code da placa no ponto de
ônibus, registra a barreira com endereço, tipo, impacto, descrição e foto opcional, recebe um protocolo e acompanha o status. 

A foto passa por uma triagem explicável em 7 etapas visíveis (original, cinza, filtrada, contraste, bordas, máscara, contornos) com resultado "Possível buraco identificado", "Nenhum indício suficiente de buraco" ou "Análise inconclusiva".
