# Guía de configuración — AI Talent Copilot

Cómo levantar los workflows en tu instancia de n8n.

---

## 1. Infraestructura

### n8n

- Docker (recomendado en local)
- n8n Cloud
- Self-hosted (npm / VPS)

Versión mínima: **n8n 1.x** con `@n8n/n8n-nodes-langchain`.

### Qdrant

```bash
docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant
```

Dashboard: `http://localhost:6333/dashboard`

Colecciones creadas por indexación: `portfolio_knowledge`, `jobs_knowledge`.

### Google Gemini

1. API key en [Google AI Studio](https://aistudio.google.com/apikey)
2. n8n → **Credentials → Google Gemini(PaLM) API**

---

## 2. Credenciales

| Credencial | Tipo | Workflows |
|---|---|---|
| Google Gemini(PaLM) Api account | `googlePalmApi` | 01–07 |
| Qdrant Local | `qdrantApi` | 02–07 (RAG) |

| Campo Qdrant local | Valor |
|---|---|
| URL | `http://localhost:6333` |
| API Key | vacío (sin auth) |

---

## 3. Archivos

### PDFs personales (indexers 02 / 04)

Montar volumen Docker:

```yaml
services:
  n8n:
    image: n8nio/n8n
    ports: ["5678:5678"]
    volumes:
      - n8n_data:/home/node/.n8n
      - ./files:/home/node/.n8n-files
    environment:
      - N8N_RESTRICT_FILE_ACCESS_TO=/home/node/.n8n-files
```

| Archivo | Ruta ejemplo | Workflow |
|---|---|---|
| CV | `/home/node/.n8n-files/CV.pdf` | 02 |
| Job Description | `/home/node/.n8n-files/JobDescription.pdf` | 04 |

### Demo batch (workflow 07)

No requiere PDF: el nodo **Load Demo Screening Batch** trae JD + 3 CVs ficticios embebidos.  
Textos de referencia en el repo: carpeta [`demo/`](../demo/).

---

## 4. Orden de ejecución (producto)

```
04 - Job Indexer         →  jobs_knowledge          (opcional para 07)
02 - CV Indexer          →  portfolio_knowledge     (para 03/05/06)
07 - Screening Engine    →  ranking JSON multi-CV   ★ demo principal
05 - Match AI            →  deep-dive 1 candidato
06 - Interview Coach     →  plan de entrevista
03 - Recruiter AI        →  chat RAG candidato
01 - Assistant           →  baseline sin RAG
```

### Demo rápida (LinkedIn / stakeholders)

1. Importar **07 - Recruiter Screening Engine**
2. Credencial Gemini
3. **Execute workflow**
4. Ver salida de **Rank Candidates** (JSON ordenado)
5. Comparar con `demo/sample_screening_result.json`

### RAG completo

1. Ejecutar **04** (y **02** si usás 03/05/06)
2. Activar chat en 03 / 05 / 06
3. Opcional: habilitar nodo **Optional JD RAG** en 07 tras indexar la JD

---

## 5. Importar

1. **Workflows → Import from File**
2. Elegir `workflow.json`
3. Reasignar Gemini / Qdrant si aparecen en rojo
4. Activar (chat) o Execute (manual)

---

## 6. Prompts

Fuente de verdad: [`docs/PROMPTS.md`](./PROMPTS.md)

- Anti-sesgo (solo skills / experiencia / requisitos)
- Salidas auditables
- JSON schema para screening (07)

---

## 7. Troubleshooting

| Problema | Solución |
|---|---|
| Credenciales en rojo | Reasignar en cada nodo |
| 07 no parsea JSON | Revisar salida cruda del LLM; el Code node busca `{...}` |
| RAG vacío en 03/05/06 | Ejecutar 02 y 04 primero |
| Scores inconsistentes | Misma JD + misma rúbrica en `PROMPTS.md` |
| Chat sin respuesta | Workflow activo + Gemini OK |

---

## 8. Seguridad

- No versionar `.env`, API keys ni PDFs personales
- Demos = perfiles ficticios
- Rotá keys si alguna exportación las incluyó embebidas
