# Slack Extractor + Enrichment

Autor: Alexandre Santana

Ferramenta para **extrair**, **tratar** e **enriquecer** conversas do Slack, incluindo anexos (áudio, vídeo, imagens, documentos) e geração de resumo executivo em JSON.

---

## Visão geral

1. **Exporta mensagens** de canais/DMs a partir de `DATA_INICIO`.
2. **Baixa todos os anexos** e os armazena em subpastas independentes.
3. **Processa anexos**:
   - Áudio/vídeo → transcrição com *Whisper* (OpenAI).
   - Imagens → descrição com *GPT‑4o‑Vision*, usando cache perceptual para evitar chamadas repetidas.
   - Documentos → extração de texto, imagens internas dos documentos e, para alguns formatos, formatação em Markdown via *GPT‑4o*.
4. **Gera JSON final** `TRATADO_PRA_USO_<canal>__<id>__YYYY-mm-dd---HH-MM-SS.json` contendo:
   - Mensagens tratadas.
   - Anexos transcritos / interpretados.
   - `resumo_executivo` estruturado conforme o prompt `PROMPT_ENRIQUECIMENTO`.
5. Todas as etapas são orquestradas pelo script único (`main()`).

---

## Enriquecimento Semântico

O processo de enriquecimento semântico adiciona camadas de significado aos dados extraídos:

- **Reconhecimento de Entidades:** Identifica pessoas, produtos, projetos, departamentos e sistemas.
- **Mapeamento de Relacionamentos:** Detecta como as entidades se relacionam.
- **Agrupamento de Tópicos:** Organiza mensagens relacionadas ao longo do tempo.
- **Detecção de Pendências:** Identifica tarefas pendentes e seu status.
- **Referências Temporais:** Destaca datas, prazos e compromissos.
- **Resumo Executivo:** Fornece uma visão geral concisa dos pontos principais.

---

## Casos de Uso

- **Gestão do Conhecimento:** Preserva e torna pesquisável o conhecimento organizacional.
- **Análise de Projetos:** Acompanhae discussões e decisões ao longo dos prazos dos projetos.
- **Monitoramento de Pendências:** Identificae e acompanha tarefas pendentes.
- **Mapeamento de Relacionamentos:** Compreender a rede de pessoas, projetos e departamentos.
- **etc**
---

### Prompts Personalizados

É possível modificar os prompts utilizados pela IA, ajustando as seguintes variáveis:
- `PROMPT_ENRIQUECIMENTO`: Controla a análise semântica.
- `PROMPT_INTERPRETACAO_IMAGEM`: Controla a análise de imagens.
- `PROMPT_INTERPRETACAO_IMAGEM_CONTEXTO`: Controla se as análise de imagens devem ser contextualizadas com o conteúdo de texto de onde foram extraídas.
---

## Estrutura de pastas

```text
.
├── README.md
├── src/                          # (opcional) onde você colocar o script .py
├── ffmpeg-master-latest-win64-gpl/  # binários do FFmpeg
├── .env
└── arquivos_salvos/
    ├── ARQUIVOS_INTERMEDIARIOS/
    │   ├── <nome>__<id>__...json          # exportações brutas
    │   ├── tratado_<nome>__<id>__...json  # processados (sem resumo)
    │   └── <nome>__<id>__..._anexos/      # anexos baixados
    └── PROCESSADOS/
        └── TRATADO_PRA_USO_<nome>__<id>__...json   # arquivo final com resumo
```

---

## Requisitos

- **Python ≥ 3.10**
- **FFmpeg** acessível em `ffmpeg-master-latest-win64-gpl/bin/ffmpeg.exe`
- Bibliotecas Python:

| Pacote                   | Uso principal                   |
| ------------------------ | ------------------------------- |
| `openai`                 | modelos GPT‑4o / Whisper        |
| `slack_sdk`              | API Slack                       |
| `python-dotenv`          | variáveis de ambiente           |
| `requests`               | download de anexos              |
| `pandas`                 | planilhas CSV/Excel/Parquet/ORC |
| `pymupdf` (`fitz`)       | PDF                             |
| `python-docx`            | DOCX                            |
| `python-pptx`            | PPTX                            |
| `odfpy`                  | ODT/ODS/ODP                     |
| `beautifulsoup4`, `lxml` | HTML/XML                        |
| `pyyaml`                 | YAML                            |
| `extract_msg`            | MSG                             |
| `striprtf`               | RTF                             |
| `pyarrow`                | Parquet/ORC                     |
| `fastavro`               | Avro                            |
| `sqlparse`               | SQL format                      |
| `pillow`, `imagehash`    | cache de imagens                |

### Instalação rápida

```bash
pip install openai slack_sdk python-dotenv requests pandas pymupdf \
            python-docx python-pptx odfpy beautifulsoup4 lxml pyyaml \
            extract_msg striprtf pyarrow fastavro sqlparse pillow imagehash
```

---

## Configuração

1. **Variáveis de ambiente** (`.env`\*)

   Crie um arquivo chamado ".env" na raiz do projeto com o conteúdo:

   ```env
   SLACK_USER_TOKEN=xoxp-...
   OPEN_AI_TOKEN=sk-...
   ```

2. **Parâmetros no script** (topo do arquivo)

   ```python
   DATA_INICIO = "2025-02-01"      # primeiro dia a considerar
   CHANNEL_IDS = [
       {"id": "xxxxxxxx", "name": "canal"},
       # Adicione mais canais conforme necessário
   ]
   ```

3. **Instale o FFmpeg**
   - Baixe a partir do [site oficial do FFmpeg](https://www.ffmpeg.org/download.html) ou das [builds do BtbN](https://github.com/BtbN/FFmpeg-Builds/releases).
   - Extraia-o para a pasta `ffmpeg-master-latest-win64-gpl` na raiz do projeto.

---

## Execução

- O script executa **três etapas por canal**: exportação, processamento e análise + enriquecimento.
- Entre canais há pausa de `CANAL_INTERVALO` segundos para evitar *rate‑limit* do Slack.

---

## Saída

- JSON final em `arquivos_salvos/PROCESSADOS/` com estrutura:
  ```json
  {
    "mensagens": [ ... ],
    "resumo_executivo": {
      "tags_tematicas": [...],
      "entidades": {...},
      "relacoes": [...],
      "referencias_temporais": {...},
      "pendencias": [...],
      "topicos": [...],
      "resumo_executivo": "..."
    }
  }
  ```

---

## Observações finais

- **Caching de imagens** usa *perceptual hash* (PIL + imagehash), pra economizar tokens em imagens iguais.
- O pipeline foi escrito para utilidade imediata — sem arquitetura rebuscada.

---
# Download

Projeto privado.

🔗[https://github.com/alexandrefortes/Slack-DataMinder/blob/main/README.md](https://github.com/alexandrefortes/Slack-DataMinder/blob/main/README.md)
