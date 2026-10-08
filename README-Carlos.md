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