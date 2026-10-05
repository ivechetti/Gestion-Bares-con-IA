# Gestión de Bares con IA

API REST para administrar bares y eventos de Tucumán, con:

- CRUD de lugares
- Carga automática desde una fuente mock (JSON)
- Lógica tipo "IA" (heurística) para clasificar y detectar posibles duplicados

## Stack

- Python 3.13
- FastAPI + Uvicorn
- SQLAlchemy
- SQLite

Elegí este stack porque es rápido de levantar y suficiente para la prueba.

## Cómo correr el proyecto

1. Clonar el repo:
```bash
git clone https://github.com/ivechetti/Gestion-Bares-con-IA.git
cd Gestion-Bares-con-IA
```

2. Instalar dependencias:
```bash
py -m pip install --user fastapi "uvicorn[standard]" sqlalchemy "pydantic<3"
```

3. Levantar la API:
```bash
py -m uvicorn app.main:app --reload
```

4. Abrir en el navegador:
- API root: http://127.0.0.1:8000
- Docs (Swagger): http://127.0.0.1:8000/docs

## Funcionalidades

### CRUD
Modelo `Place` con: `id`, `name`, `location`, `category`, `source`, `obtained_at`, `is_active`, `created_at`, `updated_at`.

Endpoints:
- `POST /places`: crear lugar
- `GET /places`: listar lugares
- `PUT /places/{id}`: editar
- `DELETE /places/{id}`: desactivar (soft delete: `is_active = false`)

### Ingesta automática (mock)
- Fuente simulada: `data/tucuman_bares.json`
- Lógica: `app/ingest.py`
- Endpoint: `POST /sync/mock`

Flujo de `/sync/mock`:
1. Lee los bares/eventos del JSON.
2. Para cada item:
   - Si ya existe un lugar con el mismo `name` y `location`, no lo vuelve a crear.
   - Si no existe, lo inserta en la base de datos.
   - Asigna la categoría con la lógica de IA basada en el nombre.
3. Devuelve un resumen, por ejemplo: `{ "inserted": 4, "updated": 0 }`

En un entorno real, la función que hoy lee el JSON podría reemplazarse por scraping de una web pública o consumo de una API. El resto del flujo sería el mismo.

## Lógica de IA (heurística)

Archivo `app/ai.py`

**1. Clasificación:** `clasificar_lugar_por_nombre(nombre: str) -> str`
Devuelve: bar, café, boliche, recital u otro.
Ejemplos: "Irlanda Bar" → bar, "Cafe Las Heras" → café.

**2. Detección de posibles duplicados:**
`detectar_posible_duplicado(nombre_candidato, lugares_existentes, umbral_similitud=5) -> dict`

Heurística:
- Normaliza el nombre (minúsculas, elimina palabras genéricas como "bar", "café", "pub", deja solo letras y números).
- Compara con los lugares existentes en la misma ubicación.
- Calcula un puntaje de similitud por cantidad de caracteres en común.
- Si supera el umbral, lo considera posible duplicado.

## Escalabilidad y mejoras

Cómo lo escalaría:
- Cambiar SQLite por PostgreSQL.
- Separar servicios: ingestión, API pública (CRUD) y deduplicación.
- Procesar la ingestión en background con colas (RabbitMQ, Kafka, etc.).
- Agregar índices y búsqueda difusa en la base (trigram similarity).

Posibles mejoras:
- Reemplazar la heurística por embeddings de texto para comparar nombres y direcciones.
- Mejorar la normalización de datos.
- Agregar validaciones más estrictas en los endpoints.
- Agregar revisión manual para casos dudosos.
