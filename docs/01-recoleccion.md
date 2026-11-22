# 01 · Cómo recolectar los datos de una clínica ficticia

**Primer entregable propio:** un inventario de fuentes, un contrato por entrega y una muestra escrita a mano que recorra todo el circuito. La clínica es inventada. Podés empezar con lápiz y Excel, sin tener una base instalada.

## Hacé un relevamiento antes de elegir columnas

| Preguntale a este rol ficticio | Pregunta de relevamiento | Qué decisión habilita |
| --- | --- | --- |
| Recepción | Cuando alguien reprograma, ¿qué queda del horario anterior? | Relacionar la reserva cancelada con la nueva sin borrar el historial. |
| Profesional | ¿Puede hacer dos prestaciones en una visita? ¿Puede cambiar el precio entre visitas? | Registrar una fila por prestación y guardar el precio de ese día. |
| Caja | ¿Una transferencia puede cubrir dos prestaciones? ¿Se cobra en cuotas? | Separar el recibo de la distribución de ese dinero. |
| Administración | ¿Qué cuenta como hora disponible y cómo se registra un cierre del consultorio? | Calcular la capacidad con intervalos habilitados. |
| Dirección | Para hablar de ingresos, ¿se refiere al trabajo realizado o a los cobros recibidos? | Etiquetar las dos medidas y elegir la fecha de cada una. |

Anotá la respuesta que asumas, la fecha y un ejemplo donde esa decisión cambie una cifra. Usá alias como PAC-01 y PRO-01. Inventá los registros desde cero, sin nombres, DNI, teléfonos, historias clínicas, imágenes ni texto libre sobre pacientes.

## Inventario y contrato mínimo

Los nombres de archivo y las frecuencias son una propuesta de práctica. En cada fuente indicá responsable ficticio, frecuencia, granularidad (qué significa una fila), identificador estable, campos obligatorios, unidades, versión del contrato, zona horaria y política de correcciones.

| Fuente simulada y entrega | Qué se recolecta por fila | Frecuencia propuesta | Control de recepción |
| --- | --- | --- | --- |
| Recepción · Excel, hojas Pacientes y Profesionales | Un ID ficticio por persona, fecha de alta y, para profesionales, alias de agenda | Foto completa semanal | ID único por hoja. Una ausencia en la foto requiere revisión antes de dar de baja. |
| Administración · Excel, hoja Catálogo | Código, descripción de servicio y precio de referencia | Foto semanal | Código único. Una nueva tarifa afecta futuras prestaciones. |
| Recepción · Excel, hoja Disponibilidad | Bloque disponible, profesional, inicio y fin | Semanal y ante cambios | Fin posterior a inicio. Descontar cierres y descansos antes de entregar. |
| Agenda · CSV de turnos | ID, paciente, profesional, creación, inicio y fin previstos, estado, ID anterior al reprogramar, última actualización | Exportación diaria de cambios | Tiempos con desplazamiento UTC o zona declarada. Validar estados y claves. |
| Agenda · CSV de eventos | ID de evento, turno, estado nuevo, instante efectivo y de registro | Cada día desde el primer mes | Sirve para reconstruir qué se sabía en una fecha. Conservar también reservas y confirmaciones iniciales. |
| Gabinete · CSV de prestaciones | ID de prestación, turno, código de tratamiento, fecha de prestación y precio final en ARS | Al cierre diario | El turno debe estar atendido. Una fila representa una unidad efectivamente realizada. |
| Caja · Excel, hojas Pagos y Aplicaciones | Un recibo con ID, fecha e importe en una hoja y un reparto recibo–prestación con importe en otra | Cierre diario | Las aplicaciones del recibo suman exactamente su importe. |
| Administración · CSV de egresos | ID, categoría, fecha de pago y monto en ARS | Semanal | Controlar cantidad e importe contra el total de la entrega. |
| Integración · API de agenda con FastAPI | Un evento de agenda con la misma identidad que en el CSV | Etapa 2, a medida que ocurre | Reenviar el mismo evento una vez por cada canal debe producir una sola novedad. |

**Ejemplo de contrato para agenda.** Nombre de lote ficticio `agenda_2026-06-30_r01`. Versión de formato 1, texto UTF-8, separador punto y coma, identificadores leídos como texto, fechas con formato año-mes-día y hora local de Córdoba con desplazamiento. Usá coma decimal solo si lo declara esa fuente. Una celda vacía en fin previsto es un dato faltante. Un identificador con ceros iniciales conserva esos ceros.

Acompañá cada entrega con una ficha de control. Un ejemplo de caja podría declarar 2 recibos y ARS 24.000 en total. Al recibirla, registrá nombre, momento, versión, conteo, total de control y huella del contenido. El mismo nombre con contenido distinto requiere una revisión nueva. Conservá la entrega original para poder volver a procesarla.

## Diseñá la simulación en tres escalas

1. **Caso manual.** Empezá con una jornada, cuatro reservas y un cobro parcial. Calculá los resultados esperados en Excel antes de simular volumen. El [caso de conciliación](04-analisis-y-power-bi.md) sirve como referencia.
2. **Semana piloto.** Inventá una cancelación seguida de otra reserva para el mismo horario, un paciente que regresa y un pago recibido el mes siguiente. Comprobá que cada situación se puede expresar con los campos del inventario.
3. **Año de práctica.** Como meta orientativa, simulá 12 meses, 4 profesionales, alrededor de 800 pacientes y 4.000 turnos. Partí de la disponibilidad real de tu escenario, asigná duración y recién después ubicá reservas. Si no entran, revisá capacidad o volumen. Algunos pacientes repiten y un turno atendido puede generar varias prestaciones. Los importes son supuestos del ejercicio.

Usá **NumPy** para fijar una semilla y sortear hora, duración o demora de cobro con distribuciones documentadas. Usá **Python** para respetar dependencias entre eventos y **pandas** para comparar el resultado con los supuestos. Una hipótesis simulada puede ser que la antelación influya modestamente en las ausencias. Dejá variación dentro de cada grupo para que la respuesta no quede codificada por una sola columna.

## Inyectá errores que puedas reconocer después

Generá primero una versión consistente. Luego agregá errores en una copia de entrega y guardá una hoja de control privada del ejercicio con ID, anomalía y resultado esperado. Esa hoja sirve para evaluar la limpieza y queda fuera de los atributos predictivos.

| Anomalía intencional | Ejemplo | Resultado esperado |
| --- | --- | --- |
| Duplicado exacto | El mismo recibo y revisión aparece dos veces | Un cobro confirmado, repetición registrada. |
| Corrección legítima | TUR-13 pasa de reservado a atendido en una revisión posterior | Historial completo, estado vigente actualizado. |
| Conflicto de versión | Mismo ID y revisión con dos importes diferentes | Revisión manual sin elegir un monto por azar. |
| Campo obligatorio vacío | Prestación sin precio final | Cuarentena con causa, sin inventar un cero. |
| Clave inexistente | TUR-22 refiere a PRO-99 ausente del maestro | Apartar y reintentar cuando llegue el maestro. |
| Imposibilidad temporal | Un turno termina antes de empezar | Rechazar el intervalo y conservar la fila original. |
| Fecha ambigua o monto mal formateado | Un origen sin formato declarado mezcla “1.200” y “1,200” | Apartar el dato hasta poder interpretarlo con el contrato. |

**Evidencia para cerrar.** Entregá el contrato, una ficha de lote y el conteo esperado por clase de error. Agregá la semilla, versión del generador, período y parámetros, y comprobá que una segunda generación reproduce las fuentes bajo la misma configuración.
