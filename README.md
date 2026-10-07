
Inicie Jupyter **desde la carpeta raíz del repositorio**, para que se encuentre el archivo `.env`:

```bash
jupyter notebook
```

Ejecute los notebooks **en este orden**, corriendo todas las celdas de arriba hacia abajo.

### Paso 1 · `Simulacion.ipynb` (carga de datos)

Crea la base de datos si no existe, genera los datos con una semilla fija (`SEED = 42`, por lo que los resultados son reproducibles) y los carga por lotes.

**Resultado esperado al final:**

```
Documentos en la BD: 85000 (esperados de esta simulación: 85000)
```

También se imprimen tres documentos de ejemplo (un producto, un cliente y un pedido).

La carga es **idempotente**: cada documento tiene un `_id` propio, así que ejecutarla de nuevo no duplica datos, solo informa cuántos ya existían.

### Paso 2 · `Creacion_de_requisitos.ipynb` (requisitos del caso)

Ejecuta una sección por requisito y, al final, imprime un resumen de cumplimiento:

```
=== RESUMEN DE REQUISITOS · MERCADOTICO 360 ===
[OK] Req 1-2: diseño y atributos variables
[OK] Req 3: índices
[OK] Req 4: antes/después de índices
[OK] Req 4: categoría + precio + atributo
[OK] Req 5: pedidos y actualización
[OK] Req 6: agregaciones MapReduce
[OK] Req 7: evolución del esquema
7/7 requisitos verificados
```

Los tiempos de ejecución dependen del equipo. La creación de índices y la primera consulta a las vistas MapReduce pueden tardar algunos segundos o minutos, porque CouchDB construye esas estructuras sobre los 85 000 documentos.

## 5. Instalación

### 5.1 Prerrequisitos

| Herramienta | Versión sugerida | Para qué se usa |
|---|---|---|
| Python | 3.10 o superior | Ejecutar los notebooks |
| Apache CouchDB | 3.5.2 | Base de datos documental |
| Docker y Docker Compose | Versión reciente | Alternativa para ejecutar CouchDB en un contenedor |
| Git | Cualquiera | Clonar el repositorio |

El proyecto puede ejecutarse con una instalación local de Apache CouchDB o mediante Docker. En ambos casos, los notebooks se conectan a CouchDB mediante HTTP utilizando el puerto 5984.

Si ya cuenta con Apache CouchDB instalado localmente, puede omitir Docker y configurar los datos de conexión mediante el archivo `.env`.

### 5.2 Clonar el repositorio y configurar CouchDB

```bash
git clone [URL_DEL_REPOSITORIO]
cd [NOMBRE_DEL_REPOSITORIO]
```

El repositorio incluye un archivo docker-compose.yml que permite iniciar CouchDB mediante Docker.

```yaml
services:
  couchdb:
    image: couchdb:3.5.2
    container_name: mercadotico-couchdb
    ports:
      - "5984:5984"
    environment:
      COUCHDB_USER: ${COUCHDB_USER}
      COUCHDB_PASSWORD: ${COUCHDB_PASS}
    volumes:
      - couchdb_data:/opt/couchdb/data

volumes:
  couchdb_data:
```

### 5.3 Entorno de Python

El repositorio incluye el archivo requirements.txt con las dependencias necesarias:

```text
requests
python-dotenv
ipykernel
notebook
```

Cree y active un entorno virtual, e instale las dependencias:

```bash
python -m venv .venv

# Linux / macOS
source .venv/bin/activate
# Windows (PowerShell)
.venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt
```

### 5.4 Configuración de credenciales

El repositorio **no contiene contraseñas**. Copie la plantilla y complete sus propios valores:

```bash
cp .env.example .env        # En Windows: copy .env.example .env
```

Copie el archivo `.env.example` como `.env` y complete sus credenciales:

```text
COUCHDB_HOST=localhost
COUCHDB_PORT=5984
COUCHDB_USER=admin
COUCHDB_PASS=
DATABASE_NAME=mercadotico360
```

Edite `.env` y escriba una contraseña en `COUCHDB_PASS`. Verifique que `.gitignore` contenga la línea `.env` para no subir sus credenciales por accidente.

> Si no existe un archivo `.env`, los notebooks preguntan cada dato al ejecutarse (la contraseña se oculta al escribirla).

### 5.5 Iniciar CouchDB

Si utiliza Docker, inicie el servicio con:

```bash
docker compose up -d
```

Para comprobar que el servicio responde:

```bash
curl http://localhost:5984/
```

Debe devolver un JSON de bienvenida con la versión de CouchDB. Además, puede explorar los datos desde el panel web **Fauxton**: <http://localhost:5984/_utils>.

## 7. Pruebas y evidencias

| Evidencia solicitada | Cómo se obtiene |
|---|---|
| Visualización de documentos reales | Sección *Documentos reales* del segundo notebook, o Fauxton |
| Pruebas antes y después de crear índices | La función `comparar_antes_despues()` elimina los índices de productos, mide una consulta, los recrea y la mide otra vez. Muestra documentos examinados y tiempo (`execution_stats`) |
| Consulta de un pedido anidado y de un producto con atributos variables | Sección *Documentos reales* |
| Evolución del esquema y validación | Sección *Evolución*: categoría nueva, atributo nuevo y rechazo de un producto con precio negativo |

## 8. Efectos que el segundo notebook deja en la base de datos

Conviene conocerlos antes de ejecutarlo:

- Crea tres *design documents*: `_design/pedidos` (cambio de estado), `_design/agregaciones` (vistas MapReduce) y `_design/validacion` (reglas de consistencia).
- Cambia el estado de un pedido del cliente `CLI-000001`.
- Inserta el producto `PROD-999999` (categoría nueva `Tecnologia_Agricola`) y agrega el atributo `certificacion_ecologica` a un producto de la categoría `Ropa`.
- Elimina y vuelve a crear los índices de productos durante la prueba de rendimiento.

La regla de validación rechaza cualquier documento sin un `tipo_doc` válido (`producto`, `cliente` o `pedido`), por lo que otros scripts que escriban en la misma base deben respetarla.

## 9. Reiniciar desde cero

**Opción A:** si utiliza Docker

```bash
docker compose down -v      # elimina el contenedor y su volumen de datos
docker compose up -d
```

Luego vuelva a ejecutar los dos notebooks en orden.

**Opción B:** si utiliza una instalación local de CouchDB

Elimine la base de datos mercadotico360 desde Fauxton y créela nuevamente al ejecutar Simulacion.ipynb.

Luego ejecute otra vez los dos notebooks en orden.