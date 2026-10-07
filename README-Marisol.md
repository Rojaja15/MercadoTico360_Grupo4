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
