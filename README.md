## 6. Ejecución

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