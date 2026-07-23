# Ejemplo de conversación — Recruiter Chat Interface (09)

Prerrequisitos: workflows **08** (candidatos) y **04** (JD) ya indexaron Qdrant.

---

## Pregunta

> ¿Por qué recomendarías a Alex Rivera sobre Jordan Lee para Senior Backend Java Engineer?

---

## Respuesta esperada (estructura)

### Verdict

Alex Rivera (cand_001) es la mejor opción para este rol: evidencia de Kafka en producción, microservicios Spring Boot en AWS y liderazgo técnico. Jordan Lee (cand_002) encaja mejor como perfil mid-level / pipeline futuro.

### Side-by-side

| Criterion | Alex Rivera | Jordan Lee |
|---|---|---|
| Technical | Java 17, Spring Boot, Kafka ~40M msg/day, AWS EKS | Spring Boot APIs, SQL, AWS ECS básico; sin Kafka prod |
| Experience | 8 años backend | 5 años (bajo el target 6+) |
| Leadership | Tech lead / mentoring / hiring loops | Sin evidencia de mentoring |

### Evidence

- Alex: “Kafka pipelines ~40M messages/day”, “12 Spring Boot microservices on AWS EKS”
- Jordan: APIs Spring Boot + PostgreSQL/Redis; gaps explícitos en Kafka y seniority

### Gaps / risks

- Alex: profundidad Terraform no evidenciada
- Jordan: no cumple hard requirements de Kafka ni seniority

### Suggested next step

- **Alex → ADVANCE** — validar ownership de incidentes y profundidad cloud infra
- **Jordan → HOLD/REJECT** para esta requisición senior

---

Otras preguntas útiles para demo:

- ¿Quién es más fuerte en liderazgo, Alex o Sam?
- ¿Qué riesgos tiene avanzar a Sam Okonkwo?
- Dame 5 preguntas de entrevista para validar el gap de Kafka de Jordan
- Compará cand_001 vs cand_003 solo en criterio technical
