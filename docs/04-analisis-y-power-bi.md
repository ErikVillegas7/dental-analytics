# 04 · Indicadores, consultas y conciliación del informe

Un turno del 30 de junio puede cobrarse el 2 de julio. Si dirección pide “ingresos de junio”, acordá primero qué fecha y qué hecho representan la pregunta. Conservá un glosario compartido entre Excel, las vistas de PostgreSQL y las medidas de Power BI.

## Contrato de indicadores

| Indicador | Unidad, fecha y cálculo | Interpretación o caso límite |
| --- | --- | --- |
| Pacientes atendidos | IDs de paciente distintos con al menos un turno atendido en el período de inicio previsto | La misma persona puede tener dos visitas. Recalculá el valor para seis meses, porque sumar meses repite pacientes. |
| Visitas atendidas | Cantidad de turnos distintos con estado atendido por fecha prevista | Dos tratamientos en un turno siguen representando una visita. |
| Tasa de ausencia | Ausentes ÷ (atendidos + ausentes), por fecha prevista, con resultado conocido | Excluir cancelados y pendientes. Si el denominador es cero, mostrar “sin base”. |
| Tasa de cancelación | Cancelados ÷ (atendidos + ausentes + cancelados), por fecha prevista | Cohorte de turnos con estado final. Informar por separado pendientes y futuros. Una reprogramación incluye una cancelación. |
| Reservas por franja | Cantidad de IDs de turno por hora de inicio prevista, incluido el cancelado. Segmentar por día y hora | Mide demanda registrada. Para medir pedidos sin turno haría falta registrar esas solicitudes. |
| Ocupación atendida | Suma de minutos previstos de turnos atendidos ÷ minutos habilitados del profesional en el mismo período | Aproxima uso de capacidad con duración reservada. Calcular intersecciones por franja y unir intervalos de capacidad solapados. Sin minutos habilitados, mostrar “sin base”. |
| Frecuencia de un tratamiento | Número de prestaciones realizadas de ese código por fecha de prestación | Un servicio realizado dos veces cuenta dos unidades. |
| Valor producido | Suma del precio final de prestaciones realizadas por fecha de prestación | Mide valor de la actividad. Puede quedar pendiente de cobro. |
| Cobros de caja | Suma de recibos aceptados por fecha de cobro | Por tratamiento o profesional, sumar aplicaciones y seguir la prestación hasta su turno. El total repartido debe coincidir con caja. |
| Saldo pendiente al corte | Valor de prestaciones hasta el corte menos aplicaciones con cobro hasta ese mismo corte | Incluir historia anterior al inicio del filtro. Una tarjeta al 31 de julio incorpora lo que se debía en junio. |
| Gastos pagados y flujo neto | Egresos por fecha de pago. Flujo neto de caja = cobros del período menos egresos pagados del período | La política de ejemplo usa caja en ARS nominales. Los gastos generales se muestran a nivel clínica. |

En esta primera versión el precio final incluye los ajustes acordados para esa prestación. Todo recibo se imputa en su fecha de cobro a prestaciones ya realizadas. Anticipos, devoluciones, impuestos y gastos por profesional quedarían para una ampliación del modelo.

## Caso patrón · Cierre del 30 de junio

Un único profesional, PRO-01, tiene disponibilidad de **09:00 a 11:00** el 30 de junio de 2026. No hay saldos de apertura. Todas las fechas del caso son locales de Córdoba.

| Turno | Paciente | Horario | Resultado | Prestaciones |
| --- | --- | --- | --- | --- |
| TUR-01 | PAC-01 | 09:00–09:45 | Atendido | PRE-01 limpieza ARS 20.000 y PRE-02 control ARS 10.000 |
| TUR-02 | PAC-02 | 09:45–10:15 | Ausente | Ninguna |
| TUR-03 | PAC-03 | 10:15–10:45 | Cancelado antes de empezar | Ninguna |
| TUR-04 | PAC-01 | 10:15–10:45 | Atendido, ocupa el horario liberado | PRE-03 control ARS 10.000 |

| Fecha | Movimiento | Aplicación o categoría |
| --- | --- | --- |
| 30 de junio | Recibo PAG-01 por ARS 12.000 de PAC-01 | ARS 12.000 a PRE-01 |
| 30 de junio | Gasto GAS-01 por ARS 5.000 | Insumos |
| 2 de julio | Recibo PAG-02 por ARS 18.000 de PAC-01 | ARS 8.000 a PRE-01 y ARS 10.000 a PRE-02 |

**Solución para contrastar tus consultas.** Para el ejemplo completo no hay otras operaciones en julio.

| Control | Resultado esperado | Cómo se obtiene |
| --- | --- | --- |
| Pacientes y visitas de junio | 1 paciente, 2 visitas | PAC-01 asistió en dos turnos. |
| Asistencia, cancelaciones y demanda | 33,33 % de ausencia, 25 % de cancelación, 4 reservas | 1/3, 1/4 y cuatro IDs. Las franjas de las 09 y 10 h tienen dos reservas cada una. |
| Capacidad utilizada en junio | 62,5 % | (45 + 30) / 120 minutos. La cancelación libera su horario. |
| Tratamientos de junio | 2 controles, 1 limpieza | Por frecuencia gana control. Por valor producido empatan en ARS 20.000 cada uno. |
| Cierre de caja de junio | ARS 40.000 producidos, 12.000 cobrados, 5.000 gastados, 7.000 de flujo neto | El trabajo de junio queda registrado aunque el pago llegue en julio. |
| Saldos al cierre | ARS 28.000 al 30/06 y ARS 10.000 al 31/07 | En julio quedan por cobrar los ARS 10.000 de PRE-03. |
| Caja de julio | ARS 18.000 cobrados, cero producidos y cero gastados | Los ingresos de caja corresponden a trabajo realizado en junio. |
| Cobros por tratamiento hasta el 31/07 | Limpieza ARS 20.000 y control ARS 10.000 | Las tres aplicaciones suman ARS 30.000, igual que los dos recibos. |

**Trampa de los JOIN.** Si unís un recibo de ARS 18.000 a sus dos aplicaciones y sumás el importe del recibo, aparece ARS 36.000. Partí del recibo para calcular caja y del importe de aplicación para atribuir cobros. Un pago y otro del mismo importe pueden ser válidos: eliminar importes iguales tampoco resuelve el problema.

## Laboratorio de análisis SQL

| Pregunta o desafío | Herramienta a practicar | Comprobación |
| --- | --- | --- |
| ¿Qué pacientes tuvieron más de una visita en el mes? | JOIN, GROUP BY y HAVING. Distinguí filtrar filas antes de agrupar de filtrar un grupo calculado. | En el caso patrón aparece PAC-01 con dos visitas. |
| ¿Qué turnos carecen de prestaciones y por qué? | LEFT JOIN y tratamiento explícito de NULL | TUR-02 y TUR-03 siguen presentes y se puede explicar su estado. |
| ¿Qué tratamientos lideran en frecuencia, valor y cobros? | Agregación, orden y ranking con función de ventana | Conservá empates y verificá las tres métricas por separado. |
| ¿Cómo cambia el saldo entre cierres? | CTE para agregar hechos por separado y acumulado por fecha | Junio termina en 28.000 y julio en 10.000 aunque en julio no hay prestaciones. |
| ¿Qué horario conviene revisar? | Calendario, franjas e intersección de intervalos. Compará reservas, minutos y tamaño de muestra. | La franja de 10 h del ejemplo ocupa 30 de sus 60 minutos. |
| ¿Cómo evolucionaron cobros y egresos en seis meses? | Vista por mes con calendario completo. Diferenciá mes en cero de fuente faltante. | Al sumar meses completos, el total coincide con caja. |

## Diseño del informe en Power BI

| Página | Qué debería poder decidir quien la usa | Visuales y filtros que vas a construir |
| --- | --- | --- |
| Agenda | ¿Qué franja revisar para confirmaciones o capacidad? | Matriz día × hora con reservas, ocupación y tamaño de base. Tarjetas de pacientes únicos, visitas y ausencia. Filtros de fecha y profesional. |
| Tratamientos | ¿Qué servicio se hace más y cuál genera más valor o cobros? | Barras ordenables por unidades, valor producido y cobros aplicados. Detalle del período y fecha de cada medida. |
| Caja | ¿Cuánto entró, cuánto salió y cuánto se adeuda a la fecha elegida? | Líneas mensuales de cobros y gastos, tarjeta de saldo al corte y tabla por prestación pendiente. Incluir fecha de actualización de caja. |

En **Power Query**, conectá las vistas analíticas de PostgreSQL y revisá tipos y fechas. Para el primer informe podés elegir importación y documentar cuándo se actualiza el modelo. En Ubuntu podés trabajar con Python, SQL y PostgreSQL. Reservá una sesión de Windows compatible para **Power BI Desktop**, por ejemplo en una VM si tu equipo tiene recursos. En esa VM, el servidor de la conexión debe ser la dirección alcanzable del equipo que aloja PostgreSQL. [Requisitos de Power BI Desktop](https://learn.microsoft.com/en-us/power-bi/fundamentals/desktop-get-the-desktop) · [Conector PostgreSQL](https://learn.microsoft.com/en-us/power-query/connectors/postgresql)

**Modelo semántico.** Dibujá dimensiones de fecha, profesional, paciente y tratamiento. Relacionalas con hechos separados de turno, capacidad, prestación, aplicación, recibo y gasto según corresponda. Usá relaciones de una dimensión a muchas filas de hechos. Documentá el rol de cada fecha. Un filtro por tratamiento llega a prestaciones y aplicaciones. Los gastos generales siguen a nivel clínica, así que desactivá las interacciones que sugieran un reparto inexistente.

**Medidas DAX a diseñar.** Practicá recuento distinto para pacientes, razón de dos totales para ausentismo y ocupación, y saldo acumulado hasta la máxima fecha seleccionada. Las tasas de varios profesionales se calculan con los numeradores y denominadores combinados. Un promedio simple de porcentajes puede distorsionar la clínica.

**Aceptación del informe.** Elegí junio, julio y luego ambos meses. Contrastá SQL, una tabla dinámica de Excel y el informe. Probá con un profesional sin actividad, con un período sin cobertura y con dos visitas de un mismo paciente. Guardá capturas de los filtros y una tabla de conciliación con valor esperado, observado, diferencia y explicación.
