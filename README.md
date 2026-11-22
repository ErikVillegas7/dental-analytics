    # Dental Analytics

**Pregunta inicial.** ¿Qué debería revisar la encargada del consultorio si la agenda estuvo llena el martes, entró poca plata y quedaron sillones libres? Para responder necesitás distinguir reservas, atenciones, tratamientos realizados y cobros. Cada uno ocurre en un momento y tiene una unidad de registro distinta.

Proyecto de portafolio de **xhito_nk** · Diseño inicial · Una clínica ficticia en Córdoba · Importes ilustrativos en ARS · Datos 100 % sintéticos.

## Cómo empezar hoy

1. Abrí el [tablero de GitHub Projects](https://github.com/users/ErikVillegas7/projects/1/views/1) y empezá por **DA-01** en *Todo*.
2. Elegí una decisión de recepción y otra de administración. Escribí qué pregunta haría cada persona y qué haría distinto si conociera la respuesta.
3. Dibujá en papel un turno con dos prestaciones y un pago parcial. Marcá qué representa una fila de cada registro. En la [guía de métricas](docs/04-analisis-y-power-bi.md) hay un caso con resultados para contrastar después.
4. Guardá tu dibujo o nota con una fecha y agregá su ruta en el issue correspondiente. Pasá la tarjeta a *In Progress* o *Done* según avances.

## Qué vas a construir

| Persona que consulta | Decisión | Entrega que construís |
| --- | --- | --- |
| Recepción | Revisar franjas con muchas reservas y ausencias | Informe de demanda observada, capacidad y asistencia |
| Administración | Priorizar saldos y entender diferencias entre actividad y caja | Conciliación de prestaciones, cobros y gastos con fecha de corte |
| Dirección | Evaluar qué servicios sostienen el consultorio | Informe de tratamientos por frecuencia, valor producido y cobros aplicados |
| Responsable de datos | Detectar entregas incompletas o procesadas dos veces | Carga trazable y repetible, con pruebas y registro de rechazos |
| Responsable de agenda | Estudiar a qué turnos ofrecer confirmación adicional | Experimento de predicción de ausencias, evaluado sobre meses posteriores |

La primera entrega termina con un informe de Power BI que coincide con el caso manual. La segunda incorpora la API, la ejecución programada y el experimento predictivo. El tablero cubre las dos. Los resultados del caso manual son ejemplos didácticos. Los resultados de tu proyecto van a surgir de tus propias ejecuciones.

## Guía de trabajo

| Abrí | Cuando necesites… |
| --- | --- |
| [01 · Recolección y simulación](docs/01-recoleccion.md) | Decidir quién generaría cada archivo, qué campos pedir y cómo simular miles de filas con errores conocidos. |
| [02 · Diseño de la base](docs/02-base-de-datos.md) | Pasar de eventos de negocio a tablas, claves, cardinalidades y restricciones de PostgreSQL. |
| [03 · ETL y operación](docs/03-etl-y-operacion.md) | Diseñar validaciones, cargas incrementales, reintentos, API y ejecución programada. |
| [04 · Análisis y Power BI](docs/04-analisis-y-power-bi.md) | Acordar indicadores, practicar SQL y comprobar el tablero con un cierre manual. |
| [05 · Ciencia de datos y portafolio](docs/05-ciencia-de-datos-y-portafolio.md) | Explorar hipótesis, plantear una predicción sin fuga temporal y explicar resultados en una entrevista. |
| [Tablero editable](Tablero_Dental_Analytics.xlsx) | Elegir una tarea, revisar dependencias y guardar evidencia. El porcentaje mide tarjetas terminadas. |
| [Tablero GitHub Projects](https://github.com/users/ErikVillegas7/projects/1/views/1) | Seguir las 22 tareas en issues y moverlas entre estados. |
| [Usar GitHub Projects](IMPORTAR_GITHUB_PROJECTS.md) | Consultar el flujo de trabajo y cómo registrar evidencia. |
| [CSV de respaldo](GitHub_Projects_Import.csv) | Conservar una copia tabular de las tareas, etapas, dependencias y criterios de cierre. |

## Dónde entra cada herramienta

| Herramienta | Trabajo que hacés vos | Evidencia para tu portafolio |
| --- | --- | --- |
| Excel | Diseñás planillas de captura con listas válidas y conciliás un cierre con filtros y tabla dinámica. | Plantilla de recepción y cierre independiente. |
| CSV | Definís un formato de intercambio estable y controlás encabezados, codificación y separador. | Contrato del archivo y ejemplo de entrega rechazada. |
| Python | Organizás la lectura de entregas, las reglas y los resultados de cada corrida. | Descripción de una corrida y su recuperación. |
| NumPy | Generás fechas, duraciones y escenarios reproducibles con semilla. | Ficha del simulador con supuestos y comprobación de distribuciones. |
| pandas | Perfilás columnas, tipificás fechas, cruzás catálogos y aislás inconsistencias. | Informe antes y después de limpiar un lote. |
| PostgreSQL | Implementás el esquema, protegés relaciones y separás datos operativos de vistas de análisis. | Diagrama y evidencia de una restricción que detecta un error. |
| SQL | Usás JOIN, GROUP BY, HAVING, CTE, funciones de ventana y vistas para responder preguntas de gestión. | Consultas comentadas y resultados verificados. |
| SQLAlchemy | Definís cómo Python abre una transacción y confirma o revierte un lote en PostgreSQL. | Prueba de fallo parcial y reintento. |
| Power BI · Power Query · DAX | Conectás vistas, modelás las relaciones y calculás medidas sensibles a los filtros. | Informe, captura filtrada y comparación con Excel y SQL. |
| Git | Versionás documentación, decisiones y luego tu implementación por hitos. | Historial de cambios pequeños con propósito. |
| GitHub | Organizás issues y revisiones, y preparás una presentación reproducible del proyecto. | Issue enlazado a una decisión y a su evidencia de cierre. |
| Docker · Docker Compose | Diseñás el entorno reproducible de base y ETL, sus dependencias y la persistencia. | Prueba desde un entorno limpio y tras reiniciar los servicios. |
| API · FastAPI | Definís y después construís la recepción incremental de novedades de agenda. | Contrato, respuesta esperada y evidencia de un evento repetido. |
| Programador de tareas de Ubuntu (cron) | Elegís horario, control de doble ejecución y recuperación de una corrida perdida. | Dos ejecuciones registradas y una simulación de interrupción. |
| Jupyter · Matplotlib o Seaborn | Explorás cobertura, distribuciones e hipótesis antes de elegir variables. | Cuaderno reproducible con dos gráficos interpretados. |
| scikit-learn | Comparás una regresión logística con una predicción base, respetando fechas. | Ficha de evaluación con métricas, límites y errores típicos. |

## Cómo usar el tablero

El tablero de GitHub Projects ya contiene los issues **DA-01–DA-22**, todos en *Todo*. Cada issue incluye la etapa, sus dependencias y un criterio de cierre. Abrí el issue para agregar evidencia y mové la tarjeta entre *Todo*, *In Progress* y *Done*.

Las dependencias están documentadas como IDs en el cuerpo de cada issue; todavía no son relaciones automáticas entre issues. Usá GitHub Projects como fuente principal del estado y conservá el Excel como copia editable. [Documentación de GitHub Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects)

## Organización en tu máquina

El ZIP contiene un README, cinco guías y una planilla. Podés extraerlo en una carpeta temporal y llevar esos elementos a `/home/xhito/Desarrollo/DataScience/odontologia`. Si ya tenés un README, conservá el tuyo y fusioná las secciones que te sirvan.

A medida que desarrolles, reservá espacios para entregas originales, datos preparados, cuadernos de exploración, consultas, informe y evidencias. Registrá las versiones de las herramientas cuando las elijas. Antes de publicar, verificá que otra persona pueda recorrer desde una fuente sintética hasta un indicador y reproducir la cifra.
