# 03 · Del archivo recibido al dato publicado

**Decisión central.** Si se interrumpe la carga justo después de recibir un pago, ¿qué puede ver administración y qué pasa al reintentar? Diseñá el recorrido de un lote con esa pregunta abierta. Cerralo cuando puedas demostrar que los totales siguen siendo correctos.

## Diseñá el recorrido

| Paso | Trabajo que vas a diseñar | Evidencia esperada |
| --- | --- | --- |
| Recepción | Python identifica fuente, contrato, fecha, huella y revisión. Conserva el original. | Manifiesto con conteo de origen, suma de control y referencia al archivo. |
| Extracción | pandas lee cada hoja o CSV con codificación, tipos y separadores explícitos. A cada fila se le asigna referencia al origen. | Perfil inicial con nulos, duplicados, formatos y claves distintas. |
| Preparación | Estandarizá estados, fechas y montos con reglas trazables. Conservá valor original y valor preparado. | Ejemplo de una transformación permitida y una ambigüedad que se apartó. |
| Validación | Contrastá esquema, claves, dominios y relaciones. Separá repetidos exactos de revisiones válidas y conflictos. | Conteos por motivo, incluido el total de filas apartadas. |
| Carga | SQLAlchemy administra la conexión y la transacción sobre PostgreSQL. Procesá maestros antes de hechos y pagos junto con sus aplicaciones. | Evidencia de confirmación o reversión completa y clave usada para reintentos. |
| Publicación | Habilitá las vistas analíticas con un lote validado. Registrá hora de corte y último intento exitoso. | Ficha de cierre, filas visibles y conciliación de importes. |

En la zona **original** preservás el archivo recibido. En **staging** guardás filas de origen y sus revisiones para inspeccionarlas. En el **modelo relacional** quedan los registros aceptados con integridad. En **analítica** publicás vistas con unidades y fechas útiles para el informe. Documentá dónde se conserva cada versión y qué vista consume Power BI.

## Política de aceptación propuesta

- Si faltan encabezados obligatorios o cambia el significado de una columna, detené el lote completo y registrá el motivo. No actualices el corte publicado con esa entrega.
- Para problemas aislados de agenda, podés publicar filas válidas y dejar el lote con estado *parcial*. Informá cuántas se apartaron y qué período cubren.
- En un cierre de caja, tratá el conjunto de pagos y aplicaciones como una unidad. Si un recibo tiene repartos incompletos o falla el total de control, retené ese cierre hasta conciliarlo. Las vistas conservan el cierre anterior para esa fuente.
- Definí una identidad de negocio estable por entidad. Usá ID y revisión para detectar un cambio. Compartí el ID del evento entre CSV y API para reconocer reenvíos. Una versión más vieja que la vigente se conserva como historial.
- Registrá por separado la hora de recepción de la fuente, el corte de negocio y la fecha de publicación. Si hay atrasos, que administración vea la antigüedad de caja y agenda por separado.

**Prueba de conteos.** En un ejemplo de 100 filas de agenda llegan 88 nuevas, 3 correcciones válidas, 4 duplicadas exactas y 5 con error. Las 100 filas tienen una disposición. El modelo incorpora 88 claves nuevas y modifica 3 existentes. Al reprocesar el mismo lote se esperan cero altas adicionales y los mismos totales de negocio. Un identificador de intento nuevo permite auditar el reintento.

**Prueba de atomicidad.** Simulá una interrupción después de preparar un recibo de ARS 18.000 y antes de cargar sus dos aplicaciones. El cierre anterior sigue visible, el intento figura como fallido y un nuevo intento incorpora un único recibo completo. Guardá el registro de ejecución fuera de la transacción que se revierte para conservar la evidencia del fallo. [Transacciones en SQLAlchemy](https://docs.sqlalchemy.org/en/20/tutorial/dbapi_transactions.html)

## Ejecución incremental y observabilidad

Usá la última actualización de origen junto con un ID estable para ordenar registros con la misma hora. Si el origen permite demoras, volvé a leer una ventana de solapamiento y resolvé las coincidencias por ID y revisión. Avanzá la marca de lectura al confirmar el procesamiento durable. Si publicás un lote parcial, guardá primero las filas apartadas con su causa y ruta de reproceso.

| Simulación | Qué deberías observar |
| --- | --- |
| Llega dos veces el mismo archivo | La huella identifica el reenvío. Conteos de negocio y caja estables. |
| Dos ejecuciones coinciden | Una obtiene el derecho a procesar ese lote. La otra espera o registra que ya está tomado. |
| Aparece un evento tardío | El historial lo conserva con ambos tiempos. La corrección recalcula solo el período afectado cuando corresponde. |
| La base se cae en mitad del cierre | Último cierre consistente y reintento verificable. |
| Un día no llega la fuente | Marca de antigüedad y aviso en el registro. Un archivo faltante se diferencia de uno válido con cero filas. |

Por corrida registrá ID, fuente, versión, inicio y fin, estado, conteos, suma esperada, suma aceptada, filas apartadas y última fecha de datos. Diseñá una ficha de incidente con síntoma, impacto, causa, reintento y evidencia de recuperación.

## Segunda entrega · API, Docker y programación

**FastAPI.** Diseñá un contrato de entrada para un evento de turno. Campos mínimos: ID del evento, ID del turno, nueva situación, hora efectiva, hora registrada, revisión y versión del contrato. Por ejemplo, EVT-082 confirma TUR-42. Un primer envío válido se acepta para procesar. Un reenvío idéntico devuelve la identidad ya recibida. La misma identidad con contenido incompatible produce un conflicto, y un estado desconocido genera un error de validación. Definí si la aceptación significa “recibido” o “publicado” y dónde se puede consultar el resultado posterior. Conservá una referencia al evento original como harías con un CSV.

**Docker y Compose.** Dibujá los servicios de base, ETL y, en la segunda etapa, API. Definí qué datos persisten en el volumen de PostgreSQL, cómo se conservan las fuentes, qué configuración necesita cada servicio y cómo comprueba el ETL que la base está lista. Registrá versiones y entregá ejemplos de configuración con valores ficticios. El entorno pasa la prueba cuando lo podés reconstruir, volver a procesar la muestra y conservar los resultados esperados después de reiniciar.

**cron en Ubuntu.** Como ejercicio, elegí un cierre diario a las 22:00 de Córdoba. Documentá qué pasa si tu notebook está apagada: al siguiente inicio, detectá los días faltantes y recuperá los lotes en orden. Reservá una única ejecución simultánea. Verificá con una corrida en horario y otra recuperada. El tablero de gestión debería mostrar qué fuente llegó y hasta qué fecha se puede confiar en cada cifra.

**Criterio de cierre.** Mostrá la ruta de un dato desde el archivo o evento hasta la vista, y repetí el recorrido después de una falla. Guardá evidencia de que se incorporó exactamente una vez a los totales.
