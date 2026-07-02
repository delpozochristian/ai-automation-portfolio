# Guía de configuración

Instrucciones para levantar los workflows de este portfolio en tu propia instancia de n8n.

---

## 1. Infraestructura

### n8n

Opciones habituales:

- **Docker** (recomendado para desarrollo local)
- **n8n Cloud**
- **Self-hosted** (npm / servidor propio)

Versión mínima recomendada: **n8n 1.x** con soporte para nodos `@n8n/n8n-nodes-langchain`.

### Qdrant

```bash
docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant
```

Dashboard: `http://localhost:6333/dashboard`

Las colecciones `portfolio_knowledge` y `jobs_knowledge` se crean automáticamente al ejecutar los workflows de indexación.

### Google Gemini

1. Crear API key en [Google AI Studio](https://aistudio.google.com/apikey)
2. En n8n: **Credentials → Add credential → Google Gemini(PaLM) API**
3. Pegar la API key

---

## 2. Credenciales en n8n

| Credencial | Tipo n8n | Usada en |
|---|---|---|
| Google Gemini(PaLM) Api account | `googlePalmApi` | Todos los workflows |
| Qdrant Local | `qdrantApi` | 02, 03, 04, 05 |

### Qdrant Local (ejemplo)

| Campo | Valor local |
|---|---|
| URL | `http://localhost:6333` |
| API Key | vacío (instancia local sin auth) |

Para Qdrant Cloud, usar la URL y API key de tu cluster.

---

## 3. Archivos PDF

Los workflows de indexación leen PDFs desde el filesystem de n8n.

### Docker

Montar un volumen y habilitar acceso a archivos:

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

Colocar los archivos:

| Archivo | Ruta en el contenedor | Workflow |
|---|---|---|
| CV | `/home/node/.n8n-files/CV_delPozoChristian_2026.pdf` | 02 |
| Job Description | `/home/node/.n8n-files/JobDescription.pdf` | 04 |

> Ajustá el nodo **Read/Write Files from Disk** si usás otra ruta o nombre de archivo.

---

## 4. Orden de ejecución

```
02 - CV RAG          →  crea/llena portfolio_knowledge
04 - Job Indexer     →  crea/llena jobs_knowledge
03 - Recruiter AI    →  chat con RAG del CV
05 - Recruiter Match →  comparación candidato vs. puesto
01 - Assistant       →  independiente (sin RAG)
```

### Indexación (manual)

1. Importar workflow **02** o **04**
2. Verificar ruta del PDF y credenciales
3. Clic en **Execute workflow**
4. Confirmar en Qdrant que la colección tiene puntos indexados

### Chat (interactivo)

1. Importar workflow **01**, **03** o **05**
2. Activar el workflow
3. Abrir el panel de chat del nodo **When chat message received**

---

## 5. Importar workflows

1. **Workflows → Import from File**
2. Seleccionar `workflow.json` de la carpeta del proyecto
3. Si n8n muestra nodos con credenciales en rojo, reasignar:
   - Google Gemini en nodos de LLM y Embeddings
   - Qdrant en nodos Vector Store
4. Guardar y activar (para workflows de chat)

---

## 6. Personalización

### Cambiar el perfil del candidato

- **Workflow 01:** editar el `systemMessage` del nodo AI Agent
- **Workflows 02–05:** reemplazar el PDF del CV y re-ejecutar la indexación

### Cambiar el modelo Gemini

En los nodos **Google Gemini Chat Model**, el campo `modelName` acepta valores como:

- `models/gemini-2.5-flash` (usado en 03 y 05)
- `models/gemini-2.0-flash`

### Prompt del matcher (workflow 05)

El system prompt del AI Agent define el formato de salida:

- Compatibilidad (%)
- Fortalezas / Debilidades
- Skills faltantes
- Riesgos
- Recomendación
- Preguntas para entrevista

---

## 7. Troubleshooting

| Problema | Solución |
|---|---|
| Credenciales en rojo tras importar | Reasignar credenciales manualmente en cada nodo |
| Error al leer PDF | Verificar ruta, permisos y `N8N_RESTRICT_FILE_ACCESS_TO` |
| Qdrant sin resultados | Ejecutar primero los workflows 02 y/o 04 |
| Colección no existe | Ejecutar el workflow de indexación; Qdrant la crea al insertar |
| Respuestas genéricas en 03/05 | Confirmar que `portfolio_knowledge` tiene datos indexados |
| Chat no responde | Verificar que el workflow está **activo** |

---

## 8. Notas de seguridad

- No commitear `.env`, API keys ni PDFs con datos personales.
- Los `workflow.json` exportados solo contienen **referencias** a credenciales (nombre e ID interno de n8n), nunca el secreto en sí.
- Al compartir workflows, rotá API keys si alguna vez se exportaron con credenciales embebidas (no es el caso de este repo).
