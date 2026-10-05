# Prueba técnica


API simple para administrar bares/eventos de Tucumán, con:
- CRUD de lugares  
- Carga automática desde una fuente mock (JSON)  
- Lógica tipo “IA” (heurística) para clasificar y detectar posibles duplicados  

---

## Stack

- Python 3.13  
- FastAPI + Uvicorn  
- SQLAlchemy  
- SQLite  

Elegí este stack porque es rápido de levantar y suficiente para la prueba.

---

## Cómo correr el proyecto

1. Clonar el repo:

```bash
git clone https://github.com/nacho942/zoco-prueba.git
cd zoco-prueba

2. Instalar dependencias 
py -m pip install --user fastapi "uvicorn[standard]" sqlalchemy "pydantic<3"

3. Levantar API
py -m uvicorn app.main:app --reload

4. Navegador 
API root: http://127.0.0.1:8000
Docs (Swagger): http://127.0.0.1:8000/docs


---


## Funcionalidades

1. CRUD
Modelo Place con:

id
name
location
category
source
obtained_at
is_active
created_at
updated_at

2. Endpoints:

POST /places – crear lugar
GET /places – listar lugares
PUT /places/{id} – editar
DELETE /places/{id} – desactivar (soft delete: is_active = false)

---

## Ingesta automatica (mock)

Fuente simulada: data/tucuman_bares.json
Lógica: app/ingest.py
Endpoint: POST /sync/mock

Flujo de /sync/mock:

1. Lee los bares/eventos del JSON.

2. Para cada item:

Si ya existe un lugar con el mismo name y location, no lo vuelve a crear (evita duplicados exactos).
Si no existe, lo inserta en la base de datos.
Asigna la categoría usando la lógica de IA basada en el nombre.

3. Devuelve un resumen, por ejemplo:
{ "inserted": 4, "updated": 0 }

En un entorno real, la función que hoy lee el JSON podría reemplazarse por scraping de una web pública o consumo de una API.
El resto del flujo (evitar duplicados, clasificar, guardar en DB) sería el mismo.

##Logica de IA (heuristica)

Archivo 'app/ai.py'

1.Clasificacion : clasificar_lugar_por_nombre(nombre: str) -> str

Clasifica el lugar según el texto del nombre:

Devuelve: bar, café, boliche, recital u otro.

Ejemplos:
"Irlanda Bar" → bar
"Cafe Las Heras" → café

2.Deteccion de posibles duplicados: detectar_posible_duplicado(nombre_candidato, lugares_existentes, umbral_similitud=5) -> dict

3. Heuristica simple:

Normaliza el nombre:
    pasa a minúsculas,
    elimina palabras genéricas como "bar", "café", "pub", etc.,
    deja solo letras y números.

Compara el nombre normalizado con los de los lugares ya existentes en la misma ubicación.

Calcula un puntaje de similitud por cantidad de caracteres en común.

Si el puntaje supera un umbral, lo considera posible duplicado.

##Escalabilidad y mejoras 

Cómo lo escalaría:

Cambiar SQLite por PostgreSQL.

Separar servicios:
    servicio de ingestión (scraping/APIs externas),
    servicio de API pública (CRUD),
    servicio de IA/deduplicación.

Procesar la ingestión en background con colas (RabbitMQ, Kafka, etc.).

Agregar índices y búsqueda difusa en la base (por ejemplo, trigram similarity) para mejorar la detección de duplicados.

Posibles mejoras:

    Reemplazar la heurística actual por IA “real” (embeddings de texto) para comparar nombres y direcciones.
    Mejorar la normalización de datos (nombres y direcciones).
    Agregar validaciones más estrictas en los endpoints.
    Agregar revisión/aprobación manual para casos dudosos.

s
