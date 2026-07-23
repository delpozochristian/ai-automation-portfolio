# Guía de configuración

Cómo levantar los workflows de este portfolio en tu instancia de n8n.

---

## 1. Infraestructura

### n8n

- **Docker** (recomendado en local)
- **n8n Cloud**
- **Self-hosted** (npm / VPS)

Versión mínima: **n8n 1.x** con `@n8n/n8n-nodes-langchain`.

### Qdrant

```bash
docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant
```

Dashboard: `http://localhost:6333/dashboard`

Las colecciones `portfolio_knowledge` y `jobs_knowledge` se crean al ejecutar los workflows de indexación (02 y 04).

### Google Gemini

1. Crear API key en [Google AI Studio](https://aistudio.google.com/apikey)
2. En n8n: **Credentials → Add credential → Google Gemini(PaLM) API**
3. Pegar la API key

---

## 2. Credenciales en n8n

| Credencial | Tipo n8n | Usada en |
|---|---|---|
| Google Gemini(PaLM) Api account | `googlePalmApi` | 01–06 |
| Qdrant Local | `qdrantApi` | 02–06 |

### Qdrant Local (ejemplo)

| Campo | Valor local |
|---|---|
| URL | `http://localhost:6333` |
| API Key | vacío (sin auth en local) |

En Qdrant Cloud: URL + API key del cluster.

---

## 3. Archivos PDF

Los indexadores leen PDFs desde el filesystem de n8n.

### Docker

```yaml
services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
      - ./files:/home/node/.n8n-files
    environment:
      - N8N_RESTRICT_FILE_ACCESS_TO=/home/node/.n8n-files
```

| Archivo | Ruta en el contenedor | Workflow |
|---|---|---|
| CV | `/home/node/.n8n-files/CV_delPozoChristian_2026.pdf` | 02 |
| Job Description | `/home/node/.n8n-files/JobDescription.pdf` | 04 |

> Si usás otra ruta o nombre, editá el nodo **Read/Write Files from Disk**.

---

## 4. Orden de ejecución

```
02 - CV RAG              →  portfolio_knowledge
04 - Job Indexer         →  jobs_knowledge
03 - Recruiter AI        →  chat RAG (solo CV)
05 - Recruiter Match     →  scoring candidato vs. puesto
06 - Interview Coach     →  prep de entrevista (preguntas + respuestas)
01 - Assistant           →  independiente (sin RAG)
```

### Indexación (manual)

1. Importar **02** y/o **04**
2. Verificar ruta del PDF y credenciales
3. **Execute workflow**
4. Confirmar puntos en Qdrant

### Chat (interactivo)

1. Importar **01**, **03**, **05** o **06**
2. Activar el workflow
3. Abrir el chat del nodo **When chat message received**

**Importante:** 03, 05 y 06 necesitan datos indexados. Sin 02/04, las respuestas serán pobres o vacías.

---

## 5. Importar workflows

1. **Workflows → Import from File**
2. Seleccionar `workflow.json` de la carpeta del proyecto
3. Si hay nodos en rojo, reasignar:
   - Google Gemini → LLM y Embeddings
   - Qdrant → Vector Store
4. Guardar y activar (chat) o ejecutar (indexación)

---

## 6. Personalización

### Cambiar el perfil del candidato

- **01:** editar el `systemMessage` del AI Agent
- **02–06:** reemplazar el PDF del CV, re-ejecutar **02**, y (si aplica) **04**

### Cambiar el modelo Gemini

En **Google Gemini Chat Model**, `modelName` puede ser:

- `models/gemini-2.5-flash` (03, 05)
- `models/gemini-2.0-flash`
- u otro modelo Gemini disponible en tu cuenta

### Prompt del matcher (05)

Formato de salida:

- Compatibilidad (%)
- Fortalezas / Debilidades
- Skills faltantes
- Riesgos
- Recomendación
- Preguntas para entrevista

### Prompt del coach (06)

Formato de salida:

1. Resumen del puesto
2. Match con la experiencia del candidato
3. 10 preguntas probables
4. Respuestas sugeridas (1ª persona)
5. Riesgos / puntos débiles
6. Historias concretas a usar
7. Preguntas inteligentes para el entrevistador

Los nodos Vector Store de 05 y 06 incluyen `toolDescription` para que el agente sepa cuál herramienta es CV y cuál es job description.

---

## 7. Troubleshooting

| Problema | Solución |
|---|---|
| Credenciales en rojo tras importar | Reasignar en cada nodo |
| Error al leer PDF | Ruta, permisos y `N8N_RESTRICT_FILE_ACCESS_TO` |
| Qdrant sin resultados | Ejecutar primero 02 y/o 04 |
| Colección no existe | Corre el indexador; Qdrant la crea al insertar |
| Respuestas genéricas en 03/05/06 | Verificar que las colecciones tienen puntos |
| 05/06 confunde CV con puesto | Revisar `toolDescription` en cada Vector Store |
| Chat no responde | Workflow **activo** + credenciales Gemini OK |

---

## 8. Seguridad

- No versionar `.env`, API keys ni PDFs personales.
- Los `workflow.json` solo llevan referencias a credenciales (nombre/ID de n8n), no el secreto.
- Si alguna vez exportaste con secretos embebidos, rotá las keys.
