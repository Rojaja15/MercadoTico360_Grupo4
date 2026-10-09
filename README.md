# MercadoTico 360: Solución orientada a documentos con Apache CouchDB

**Curso:** XS0131 – Gestión de Bases de Datos y Análisis de Información
**Actividad:** Investigación Grupal 1 – Desafío NoSQL
**Caso asignado:** Caso 2 – MercadoTico 360 (modelo de **documentos**)
**Tecnología:** Apache CouchDB 3.5.2

---

## 1. Descripción

MercadoTico 360 es un marketplace ficticio que agrupa comercios costarricenses. Su catálogo mezcla productos de naturaleza muy distinta (alimentos, ropa, tecnología, artesanías, entre otros), por lo que cada categoría necesita atributos diferentes. En un modelo relacional esto obliga a crear muchas tablas y columnas opcionales.

Este repositorio implementa una alternativa basada en **documentos JSON** sobre Apache CouchDB. Permite:

- almacenar un catálogo heterogéneo sin forzar un esquema idéntico para todos los productos;
- guardar pedidos con estructuras anidadas (líneas de detalle) que conservan lo que el cliente compró aunque el producto cambie después;
- consultar, agregar y evolucionar el esquema sin migraciones relacionales.

## 2. Contenido del repositorio

```
.
├── Simulacion.ipynb               # Genera los datos sintéticos y los carga en CouchDB
├── Creacion_de_requisitos.ipynb   # Ejecuta consultas, índices, actualizaciones, agregaciones y pruebas
├── docker-compose.yml             # Levanta CouchDB localmente (ver sección 5.2)
├── requirements.txt               # Dependencias de Python (ver sección 5.3)
├── .env.example                   # Plantilla de configuración, sin secretos
├── .gitignore                     # Debe incluir la línea ".env"
└── README.md
```

Los archivos docker-compose.yml, requirements.txt y .env.example se incluyen en el repositorio y su contenido se describe en la sección 5.

## 3. Requisitos del caso y dónde se cumplen

| N.º | Requisito del caso | Dónde se cumple |
|---|---|---|
| 1 | Diseñar documentos y justificar qué se embebe y qué se referencia | `Simulacion.ipynb` (tabla de diseño) y sección 4 de este documento |
| 2 | Atributos diferentes por categoría | `Simulacion.ipynb` (diccionario `CATALOGO`) |
| 3 | Índices para al menos tres patrones de consulta | `Creacion_de_requisitos.ipynb` (sección *Índices*) |
| 4 | Búsqueda por categoría, rango de precio y atributo específico | `Creacion_de_requisitos.ipynb` (sección *Búsquedas*) |
| 5 | Recuperar pedidos de un cliente y actualizar su estado | `Creacion_de_requisitos.ipynb` (sección *Pedidos*) |
| 6 | Al menos dos agregaciones | `Creacion_de_requisitos.ipynb` (sección *MapReduce*, tres vistas) |
| 7 | Evolución del esquema sin migración | `Creacion_de_requisitos.ipynb` (sección *Evolución*) |

**Escala de los datos generados:** 25 000 productos en 8 categorías, 10 000 clientes y 50 000 pedidos con 1 a 4 líneas de detalle (85 000 documentos en total).

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

## 6. Ejecución

Abra los notebooks desde la carpeta raíz del repositorio en VS Code o Jupyter, utilizando el entorno virtual configurado.

```bash
jupyter notebook
```

Ejecute los notebooks **en este orden**, corriendo todas las celdas de arriba hacia abajo.

### Paso 1 · `Simulacion.ipynb` (carga de datos)

Crea la base de datos si no existe, genera los datos con una semilla fija (`SEED = 42`, por lo que los resultados son reproducibles) y los carga por lotes.

**Resultado esperado al final:**

```
Documentos de datos generados en esta simulación: 85000
```

También se imprimen tres documentos de ejemplo (un producto, un cliente y un pedido).

Cada documento utiliza un _id definido, por lo que una nueva ejecución no crea documentos duplicados con identificadores distintos. Los documentos ya existentes se identifican durante la carga.

### Paso 2 · `Creacion_de_requisitos.ipynb` (consultas y pruebas)

Ejecuta las consultas, pruebas de índices, actualizaciones, agregaciones y pruebas de evolución del esquema. Al final muestra un resumen de los resultados obtenidos.

```
RESUMEN DE PRUEBAS - MERCADOTICO 360

Correcto: Diseño y atributos variables
Correcto: Índices creados - 4 índices
Correcto: Comparación antes/después de índices
Correcto: Búsqueda por categoría, precio y atributo
Correcto: Pedidos y actualización de estado
Correcto: Agregaciones MapReduce - 3 vistas consultadas
Correcto: Evolución del esquema

Pruebas correctas: 7/7
```

Los tiempos de ejecución dependen del equipo. La creación de índices y la primera consulta a las vistas MapReduce pueden requerir más tiempo, ya que CouchDB debe construir o actualizar las estructuras necesarias sobre los documentos almacenados.

## 7. Pruebas y evidencias

| Prueba o evidencia | Cómo se obtiene |
|---|---|
| Visualización de documentos | Sección de documentos y atributos variables del segundo notebook, o Fauxton |
| Comparación antes y después de índices | Función comparar_antes_despues(); elimina temporalmente los índices de productos, ejecuta la consulta, los recrea y vuelve a medir |
| Pedido anidado y productos con atributos variables | Sección de documentos y atributos variables |
| Actualización del estado de un pedido | Sección de pedidos y actualización de estado |
| Agregaciones | Sección de agregaciones con MapReduce |
| Evolución y validación del esquema | Sección de evolución del esquema |

## 8. Efectos que el segundo notebook deja en la base de datos

Conviene conocerlos antes de ejecutarlo:

- Crea tres *design documents*: `_design/pedidos` (cambio de estado), `_design/agregaciones` (vistas MapReduce) y `_design/validacion` (reglas de consistencia).
- Cambia el estado de un pedido del cliente `CLI-000001`.
- Inserta el producto `PROD-999999` (categoría nueva `Tecnologia_Agricola`) y agrega el atributo `certificacion_ecologica` a un producto de la categoría `Ropa`.
- Elimina y vuelve a crear los índices de productos durante la prueba de rendimiento.

La regla de validación rechaza cualquier documento cuyo campo `type` no sea válido (`product`, `customer` u `order`), por lo que otros scripts que escriban en la misma base deben respetarla.

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

## 13. Integrantes

| Nombre | Carnet |
|---|---|
| Carlos Armando Soto Monge | [Carnet] | 
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