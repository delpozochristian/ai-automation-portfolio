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

### Google Gemini

1. API key en [Google AI Studio](https://aistudio.google.com/apikey)
2. n8n → **Credentials → Google Gemini(PaLM) API**

---

## 2. Credenciales

| Credencial | Tipo | Workflows |
|---|---|---|
| Google Gemini(PaLM) Api account | `googlePalmApi` | 01–08 |
| Qdrant Local | `qdrantApi` | 02–08 (RAG) |

| Campo Qdrant local | Valor |
|---|---|
| URL | `http://localhost:6333` |
| API Key | vacío (sin auth) |

---

## 3. Colecciones Qdrant

| Colección | Rol | Escrita por | Leída por |
|---|---|---|---|
| `portfolio_knowledge` | CV personal (demos 01–06) | 02 | 03, 05, 06 |
| `jobs_knowledge` | Job descriptions | 04 | 05, 06, 07 |
| `candidates_knowledge` | Pool multi-candidato (producto) | **08** | **07** |

Arquitectura: **index once / retrieve many** — ver README.

---

## 4. Archivos

### PDFs personales (02 / 04)

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
| CV personal | `/home/node/.n8n-files/CV.pdf` | 02 |
| Job Description | `/home/node/.n8n-files/JobDescription.pdf` | 04 |

### Demo batch (08 + 07)

- **08** embebe 3 CVs ficticios y los indexa en `candidates_knowledge` (sin PDF).
- **07** no lee archivos: solo RAG retrieve + scoring.
- Textos de referencia: carpeta [`demo/`](../demo/).

Para producción en 08: reemplazar el Code node por Read Files + Extract PDF + metadata `candidate_id`.

---

## 5. Orden de ejecución (producto)

```
04 - Job Indexer                 → jobs_knowledge
08 - Candidates Knowledge Indexer → candidates_knowledge
07 - Screening Engine            → ranking ATS JSON   ★
05 - Match AI                    → deep-dive 1 candidato
06 - Interview Coach             → plan de entrevista
02/03                            → demos portfolio personal
01                               → baseline sin RAG
```

### Demo Screening (recomendado)

1. Importar **08** → Gemini + Qdrant → **Execute** (crea `candidates_knowledge`)
2. Importar **04** → indexar JD de demo en `jobs_knowledge` (ideal)
3. Importar **07** → **Execute** → salida de **Rank Candidates (ATS Payload)**
4. Comparar con `demo/sample_screening_result.json`

Si `jobs_knowledge` está vacío, el agente puede fallar o scorear mal: indexá la JD primero.

---

## 6. Importar

1. **Workflows → Import from File**
2. Elegir `workflow.json`
3. Reasignar Gemini / Qdrant si aparecen en rojo
4. Activar (chat) o Execute (manual / screening)

---

## 7. Prompts & contrato ATS

- Prompts anti-sesgo: [`docs/PROMPTS.md`](./PROMPTS.md)
- Output 07: `candidate_id`, `job_id`, `criteria_scores`, `evidence`, `recommendation`, `interview_priority`

---

## 8. Troubleshooting

| Problema | Solución |
|---|---|
| 07: no hay evidencia de candidato | Ejecutar **08** primero |
| 07: job retrieve vacío | Ejecutar **04** con la JD de demo |
| Credenciales en rojo | Reasignar en cada nodo |
| JSON inválido del agente | Revisar system prompt; el Code node busca `{...}` |
| Scores inconsistentes | Misma rúbrica en `PROMPTS.md` |

---

## 9. Seguridad

- No versionar `.env`, API keys ni PDFs personales
- Demos = perfiles ficticios
- Rotá keys si alguna exportación las incluyó embebidas
