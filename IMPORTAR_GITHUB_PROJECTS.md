# Llevar el seguimiento a GitHub Projects

El archivo [GitHub_Projects_Import.csv](GitHub_Projects_Import.csv) contiene las 22 tareas del tablero de Excel, con sus IDs, etapas, dependencias y criterios de cierre. Todas empiezan en `Todo`; la evidencia queda vacía para completarla a medida que avances.

Podés crear un proyecto personal de GitHub aunque todavía no tengas un repositorio:

1. En GitHub, creá un **Project** personal y elegí la vista **Board**.
2. Desde el menú del proyecto, elegí la opción para importar desde CSV y seleccioná `GitHub_Projects_Import.csv`.
3. Agrupá las tarjetas por `Status`. Usá `Etapa` para filtrar el trabajo y registrá los enlaces o rutas de evidencia en `Evidencia propia`.
4. Cuando tengas un repositorio, vinculalo al proyecto para asociar tareas con issues, cambios y revisiones.

Las dependencias se importan como texto en `Depende de`; GitHub no las convierte automáticamente en relaciones entre issues. El Excel original permanece intacto.
