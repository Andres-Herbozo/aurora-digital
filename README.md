# aurora-digital

**Ecosistema de Automatización IA — Aurora Digital**

Sistema autónomo de calificación de leads y generación de propuestas comerciales, con
validación humana antes del contacto con el cliente.

**Proyecto final — [Nombre del curso]**
Autor: Andrés Herbozo · [Mes] 2026

---

## El problema

Aurora Digital es una agencia de servicios digitales que recibe consultas por su
formulario web en lenguaje natural, sin estructura. Cada consulta requiere que alguien la
lea, determine qué servicio corresponde, evalúe la urgencia y redacte una propuesta.
El proceso es lento, inconsistente entre ejecutivos, y los leads de mayor valor se
diluyen entre los exploratorios.

## La solución

Un flujo que interpreta la consulta, la clasifica contra el catálogo real de servicios,
redacta una propuesta con los precios y plazos correctos, y **se detiene a esperar
aprobación humana** antes de enviar cualquier correo al prospecto.

```
Formulario web
      ↓
  Validación de datos ─────────────→ [Log de error]
      ↓
  Registro en Airtable
      ↓
  Clasificación IA ────────────────→ [Log de error]
      ↓
  Filtro de confianza ─────────────→ [Revisión manual]
      ↓
  Redacción de propuesta
      ↓
  ⏸  APROBACIÓN HUMANA (Slack)
      ↓                    ↘
  Envío por Gmail        [Descartado]
      ↓
  Cierre del ciclo
```

---

## Stack

| Capa | Tecnología | Rol |
|---|---|---|
| Orquestación | n8n (self-hosted, queue mode) | Flujo principal y manejo de errores |
| Base de datos | Airtable | Memoria del sistema y registro de estados |
| Procesamiento IA | Claude (Anthropic API) | Clasificación estructurada y redacción |
| Validación | Slack (app `Aurora Bot`) | Punto de human-in-the-loop |
| Salida | Gmail | Entrega de la propuesta al lead |

---

## Entregables

| # | Criterio de la rúbrica | Dónde está |
|---|---|---|
| 1 | Mapa de arquitectura | [`docs/documentacion-proyecto.pdf`](docs/) — sección 1 |
| 2 | Estructuras de datos documentadas | [`docs/documentacion-proyecto.pdf`](docs/) — sección 2 |
| 3 | Matriz de costos por modelo | [`docs/documentacion-proyecto.pdf`](docs/) — sección 3 |
| 4 | Seguridad y resiliencia | [`docs/documentacion-proyecto.pdf`](docs/) — sección 4 |
| 5 | Dashboard de control | [Vista pública →](https://airtable.com/appdkQ1zI3fufTsuH/shrdTWxVYHFyoRbcg) |

**Enlaces:**
- Base de datos en modo lectura: [Embudo de leads](https://airtable.com/appdkQ1zI3fufTsuH/shrdTWxVYHFyoRbcg)
- Dashboard · Pipeline (Kanban): https://airtable.com/appdkQ1zI3fufTsuH/shrLFhQ4dUVYqRUoA
- Dashboard · Errores y tasa de fallos: https://airtable.com/appdkQ1zI3fufTsuH/shrjZ9A6y6Cmlr7jn
- Video demo (3 min): PENDIENTE

---

## Contenido del repositorio

```
├── docs/
│   ├── documentacion-proyecto.pdf     Documento principal (4 secciones)
│   └── diagrama-arquitectura.png
├── workflows/
│   ├── 01-clasificacion-leads.json    Flujo principal
│   ├── 02-aprobacion-slack.json       Respuesta del HITL
│   └── 03-manejador-errores.json      Error workflow global
├── prompts/
│   ├── clasificacion.md
│   └── redaccion-propuesta.md
└── evidencias/                        Screenshots de los 5 tests
```

---

## Modelo de datos

Tres tablas relacionadas en Airtable:

- **Leads** — memoria del ciclo completo. El campo `Estado` recorre: Nuevo → Procesado IA
  → Esperando aprobación → Aprobado → Enviado, con ramas hacia Descartado, Revisión manual
  y Error.
- **Servicios** — catálogo con precios y plazos. Se inyecta dinámicamente en el prompt de
  clasificación, de modo que agregar un servicio no requiere modificar el flujo.
- **Logs de error** — registro de fallos con nodo de origen, tipo y payload.

Cardinalidad: un lead solicita un servicio, un servicio puede ser solicitado por muchos
leads. Un error se origina en un solo lead, un lead puede acumular varios errores.

---

## Resiliencia

Cuatro capas de manejo de errores:

1. **Validación de entrada** — filtro previo a cualquier llamada de IA
2. **Continue on Fail** — los nodos de IA no derriban el flujo al fallar
3. **Retry** — 2 reintentos ante fallos transitorios de API
4. **Error Workflow global** — captura cualquier fallo no previsto y lo registra

Todos los caminos de error escriben en la tabla `Logs de error`, que alimenta la tasa de
errores del dashboard.

---

## Instalación

1. Importar los archivos de `workflows/` en una instancia de n8n
2. Configurar credenciales: Airtable (PAT), Anthropic (API key), Slack (Bot token),
   Gmail (OAuth2)
3. Duplicar la estructura de Airtable según el esquema de la sección 2 del PDF
4. Configurar el Request URL de Interactivity en la app de Slack `Aurora Bot`, apuntando
   al webhook del flujo 02
5. Activar el Error Workflow global en la configuración de cada flujo

**Los archivos JSON no contienen credenciales.** Todas las claves se gestionan mediante el
almacén de credenciales de n8n y deben configurarse en la instancia de destino.

---

## Nota sobre seguridad

Este repositorio no contiene claves de API, tokens ni variables de entorno. Las capturas
de pantalla fueron revisadas para excluir credenciales visibles. El sistema aplica
minimización de datos: se capturan únicamente nombre, correo, empresa y consulta — los
campos necesarios para responder al lead.
