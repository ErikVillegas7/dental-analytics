# 05 · De una pregunta exploratoria a una evaluación predictiva

**Pregunta de gestión.** Recepción puede revisar pocos turnos por día. Con la información disponible 24 horas antes, ¿podría priorizar confirmaciones mejor que una regla sencilla? Es un experimento operativo con datos ficticios. La clínica del ejercicio sirve para evaluar tu método y las decisiones de simulación.

## Análisis exploratorio en Jupyter

| Pregunta | Trabajo con pandas, NumPy y visualización | Qué escribir al pie del gráfico |
| --- | --- | --- |
| ¿Cómo cambia la ausencia según la antelación? | Agrupá por intervalos entre reserva y visita. Dibujá tasas, acompañadas de cantidad de turnos. | Período, base del porcentaje y si los grupos pequeños admiten comparación. |
| ¿Las franjas de menor ocupación tienen pocas reservas o muchas ausencias? | Compará reservas por hora, minutos ofrecidos, minutos atendidos y ausencias. Usá una matriz de día y hora con Matplotlib o Seaborn. | Dos explicaciones compatibles y qué dato adicional permitiría distinguirlas. |
| ¿Cuánto se tarda en cobrar? | Calculá días hasta el primer cobro y hasta completar el saldo. Describí mediana y percentiles. Contá aparte prestaciones abiertas al corte. | Advertí si muchas prestaciones recientes siguen abiertas y cómo eso afecta la comparación. |

Escribí la hipótesis antes de mirar el gráfico, perfilá valores faltantes y revisá cuántos pacientes repiten. Compará los patrones observados con los parámetros usados por el simulador. Para una tasa, registrá numerador, denominador y, si estimás incertidumbre, la unidad de remuestreo. Si una persona aporta varias visitas, podés remuestrear pacientes completos en vez de tratar cada visita como independiente.

## Diseño del conjunto de predicción

- **Una fila por turno elegible.** En el instante de decisión, 24 h antes de empezar, el turno ya existe y figura reservado o confirmado según el historial conocido a esa hora. Excluí reservas de último momento y cancelaciones ya conocidas. Registrá cuántos casos salen por cada regla.
- **Resultado posterior.** Para esta pregunta, la etiqueta vale 1 si termina ausente y 0 si asiste o cancela después del corte. Pendientes al cierre de evaluación quedan fuera hasta tener desenlace. Esta población tiene su propio denominador, diferente de la tasa descriptiva de ausencias del dashboard.
- **Variables disponibles.** Día, horario, antelación calculada al reservar, historial de asistencias o ausencias ya conocido y cantidad de turnos previos. Para un paciente nuevo distinguí “cero visitas previas” de “historial faltante”. Los IDs permiten unir registros y rastrear errores.
- **Fuga de información.** El estado final, el pago posterior, la prestación realizada y la cantidad total de faltas del paciente al final del año revelan futuro. Reconstruí los datos como estaban disponibles en el corte, incluyendo demoras de registro y recepción. Simulá un evento registrado tarde para comprobarlo. [Guía de scikit-learn sobre fuga de datos](https://scikit-learn.org/stable/common_pitfalls.html)

**Ejemplo ficticio.** Un turno empieza el miércoles 10 de junio a las 10:00. Para decidir el martes 9 a las 10:00, podés usar una reserva ingresada el lunes y una visita anterior cerrada el viernes. Una falta del lunes cargada recién el martes a las 14:00 queda fuera de las variables de ese turno. El desenlace del miércoles aporta la etiqueta más adelante.

## Experimento reproducible

1. **Separación temporal.** Con doce meses completos, una propuesta es entrenar en meses 1–8, validar en 9–10 y evaluar una vez en 11–12. Usá como eje la fecha de decisión. Al entrenar, utilizá solo turnos con desenlace ya conocido a ese corte. Las imputaciones, escalados y categorías se aprenden con entrenamiento.
2. **Línea de base.** Usá como referencia la tasa global de ausencia del entrenamiento. Para una comparación de ranking, probá además una tasa por franja aprendida en entrenamiento, con retorno a la media global si el grupo es pequeño. Definí de antemano cómo desempatar.
3. **Modelo candidato.** Construí una regresión logística con scikit-learn. Explicá por qué un modelo interpretable alcanza para la primera comparación. Elegí variables y ajustes con la validación cronológica.
4. **Métricas y capacidad.** Reportá proporción de ausencias, *average precision* y su referencia, precisión y cobertura de ausencias entre los primeros K de cada día. Elegí K por capacidad de recepción. Si usás probabilidades para decidir, revisá calibración por grupos de riesgo y guardá una matriz de confusión para el umbral elegido en validación.
5. **Evaluación final.** Conservá el corte de prueba hasta el final. Compará el modelo y las referencias sobre los mismos turnos, revisá errores por mes y pacientes nuevos/recurrentes, y repetí con otra semilla de simulación para observar cuánto depende de tus propios supuestos.

**Ejemplo para aprender a leer la métrica.** En un día con 20 turnos elegibles ocurren 3 ausencias. Recepción podría revisar 5 turnos. Si 2 ausencias están entre los 5 elegidos, la precisión en esos cinco es 2/5 = 40 % y la cobertura de ausencias es 2/3 ≈ 67 %. Eso mide capacidad de priorización. Para estimar cuántas ausencias evitaría un recordatorio haría falta otro diseño que compare intervenciones. Definí cómo tratar días con menos de cinco turnos elegibles.

## Qué dejar para la entrevista

| Pieza que vas a producir | Contenido para completar con tus resultados |
| --- | --- |
| Informe ejecutivo de una página | Decisión, indicador, período, hallazgo con tamaño de muestra, limitación y siguiente acción. |
| Ficha del experimento | Origen sintético, población, instante de decisión, etiqueta, variables, cortes temporales, referencia, métrica, resultado y límites. |
| Registro de decisiones | Fecha, pregunta, opciones consideradas, elección, motivo, consecuencia y evidencia. Ejemplo: separar imputaciones para poder atribuir un recibo a dos servicios. |
| Historial de Git y GitHub | Un issue con criterio de aceptación, cambios pequeños, una revisión y evidencia al cerrar. Redactá el README final con resultados comprobados. |
| Guía de reproducción | Herramientas y versiones elegidas, configuración de ejemplo, orden de preparación y carga, actualización del informe, prueba de reintento y resultados de control. |

**Guion de demostración.** Abrí con la pregunta de caja de junio. Mostrá el recibo en la fuente, cómo sobrevivió a un reintento y en qué vista aparece. Filtrá junio y julio en Power BI para explicar la deuda. Cerrá con la comparación del modelo contra la referencia y una limitación concreta del generador. Un resultado donde la referencia iguala o supera al modelo también es defendible si podés explicar el experimento.
