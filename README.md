
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

## 4. Modelo de datos

La base de datos usa **un solo contenedor lógico** con tres tipos de documento, distinguidos por el campo `type`.

| Tipo | `_id` | Decisión de diseño | Justificación |
|---|---|---|---|
| `product` | `PROD-000001` | Documento propio con `atributos` **embebidos** | Cada categoría tiene atributos distintos; viven dentro del producto sin esquema fijo. |
| `customer` | `CLI-000001` | Documento propio | Un cliente puede tener muchos pedidos. Embeberlos haría crecer el documento sin límite. |
| `order` | `PED-000001` | `lineas_detalle` **embebidas**; `cliente_id` y `producto_id` **referenciados** | Las líneas de detalle forman parte del pedido y normalmente se consultan junto con este. Cada línea guarda un *snapshot* (nombre, categoría y precio) para reconstruir la compra aunque el producto cambie. |

**Ejemplo de producto** (categoría `Tecnologia`):

Los precios utilizados en los datos sintéticos se expresan en colones costarricenses y se generan dentro de rangos definidos para cada categoría.

```json
{
  "_id": "PROD-000123",
  "type": "product",
  "nombre": "Laptop 123",
  "categoria": "Tecnologia",
  "precio": 845000,
  "stock": 120,
  "atributos": { "marca": "TechBrand-7", "ram_gb": 16, "almacenamiento_gb": 512 }
}
```

**Ejemplo de pedido** (estructura anidada):

```json
{
  "_id": "PED-000001",
  "type": "order",
  "cliente_id": "CLI-004637",
  "fecha_pedido": "2026-07-14",
  "estado": "pendiente",
  "monto_total": 25000,
  "lineas_detalle": [
    {
      "producto_id": "PROD-009498",
      "nombre_snapshot": "Alimento para mascota 9498",
      "categoria_snapshot": "Mascotas",
      "cantidad": 2,
      "precio_unitario": 12500,
      "subtotal": 25000
    }
  ]
}
```

**Índices creados**

| Índice | Campos | Consulta que acelera |
|---|---|---|
| `idx_product_category` | `type`, `categoria` | Productos de una categoría |
| `idx_product_price` | `type`, `precio` | Productos en un rango de precios |
| `idx_technology_ram` | `type`, `categoria`, `atributos.ram_gb` | Atributo específico de una categoría |
| `idx_pedidos_cliente` | `type`, `cliente_id` | Historial de pedidos de un cliente |

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


## 10. Solución de problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| `No hay conexión con CouchDB` | El contenedor no está activo o el puerto es otro | Ejecute `docker compose ps` y revise `COUCHDB_PORT` o verifique que CouchDB esté corriendo en localhost:5984 |
| `Usuario o contraseña incorrectos` | La contraseña del `.env` no coincide con la configurada en CouchDB | Revise el archivo `.env` y las credenciales del servidor |
| `No se encontraron documentos 'product'` | Se ejecutó el segundo notebook antes que el primero | Ejecute primero `Simulacion.ipynb` |
| Las ventas por categoría aparecen vacías o con clave `null` | La base se cargó con una versión anterior de los datos | Reinicie desde cero (sección 9) y cargue de nuevo |
| Los notebooks piden los datos de conexión manualmente | Jupyter o VS Code no encontró el archivo `.env` | Abra los notebooks desde la carpeta raíz del repositorio |

## 11. Limitaciones

- **Un índice por consulta:** el motor de consultas Mango selecciona un índice por búsqueda; las condiciones que no puedan resolverse directamente con ese índice se evalúan posteriormente sobre los documentos candidatos.
- **Actualización de documentos:** CouchDB guarda una nueva revisión completa del documento en cada cambio. El *update handler* evita reenviar el pedido completo desde el cliente, pero el servidor sigue almacenando una nueva revisión.
- **Datos sintéticos:** los datos se generan con distribuciones simplificadas, por lo que no representan necesariamente patrones reales de compra.
- **Instalación de un solo nodo:** la configuración utilizada corresponde a un entorno local de demostración y no cubre replicación ni clúster.

## 12. Plan de contingencia para la demostración

- Se recomienda cargar los datos y ejecutar ambos notebooks completos antes de la presentación.
- Si se utiliza Docker, los datos persisten en el volumen mientras no se elimine con docker compose down -v.
- Si se utiliza una instalación local de CouchDB, conviene verificar previamente que la base mercadotico360 y los design documents estén disponibles.
- Es recomendable conservar capturas de pantalla o un video corto de la ejecución como respaldo.

# 13. Integrantes

| Nombre | Carnet |
|---|---|
| Carlos Armando Soto Monge | [B67061] | 
| Ericka Marisol Quesada Madrigal | [C26111] | 
| Evelio De Los Ángeles Chinchilla Rosales | [A91830] |
| Gustavo Jhosua Vargas Viales | [C38271] |
| Rodrigo Javier Gutiérrez Calderón | [C23599] |

## 14. Referencias

- Apache Software Foundation. *Apache CouchDB Documentation*. Disponible en: <https://docs.couchdb.org/en/stable/>

- Apache Software Foundation. *Mango: consultas declarativas (`_find`) e índices*. Disponible en: <https://docs.couchdb.org/en/stable/api/database/find.html>

- Apache Software Foundation. *Design Documents: vistas, update handlers y validación*. Disponible en: <https://docs.couchdb.org/en/stable/ddocs/ddocs.html>

- ICEX España Exportación e Inversiones. *Informe e-País: Comercio electrónico en Costa Rica 2025 — Resumen ejecutivo*. Oficina Económica y Comercial de España en San José, 2025. Disponible en: <https://www.icex.es/content/dam/icex/centros/costa-rica/documentos/2025/informe-epais-comercio-electronico-costa-rica-2025-resumen-ejecutivo.pdf>

- NIC Costa Rica. *NIC Costa Rica participó en el Ecommerce & Delivery Forum para acercar el comercio en línea a las PYMES*. 2 sep. 2026. Disponible en: <https://nic.cr/2026/09/02/nic-costa-rica-participo-en-el-ecommerce-delivery-forum-para-acercar-el-comercio-en-linea-a-las-pymes/>

- E. F. Codd. *A relational model of data for large shared data banks*. Communications of the ACM, vol. 13, no. 6, pp. 377–387, jun. 1970. DOI: 10.1145/362384.362685.

- R. Agrawal, A. Somani y Y. Xu. *Storage and querying of e-commerce data*. Proceedings of the 27th International Conference on Very Large Data Bases (VLDB), Roma, Italia, 2001, pp. 149–158.

- R. Cattell. *Scalable SQL and NoSQL data stores*. ACM SIGMOD Record, vol. 39, no. 4, pp. 12–27, 2011. DOI: 10.1145/1978915.1978919.

- M. Stonebraker. *SQL databases v. NoSQL databases*. Communications of the ACM, vol. 53, no. 4, pp. 10–11, abr. 2010. DOI: 10.1145/1721654.1721659.

- F. Gessert, W. Wingerath, S. Friedrich y N. Ritter. *NoSQL database systems: A survey and decision guidance*. Computer Science - Research and Development, vol. 32, no. 3–4, pp. 353–365, 2017. DOI: 10.1007/s00450-016-0334-3.
