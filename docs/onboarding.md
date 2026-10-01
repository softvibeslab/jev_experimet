# Onboarding: crear el proyecto en Cursor

Este documento guía la implementación. Hoy el repositorio contiene documentación; no asumas que ya hay aplicación, dependencias o tests.

## 1. Preparar acceso y herramientas

Instala Cursor, Git, Python compatible y uv siguiendo sus fuentes oficiales. Verifica en tu terminal:

```bash
git --version
python3 --version
uv --version
```

Usa tu autenticación GitHub autorizada; no pegues tokens en el chat de Cursor. Para la evaluación necesitas una clave de OpenRouter con acceso y crédito; introducirla solo en un archivo local excluido de Git o un gestor de secretos.

El MVP no requiere conectarse al CRM, al catálogo de producción ni a servicios de mensajería.

## 2. Clonar y abrir

```bash
git clone https://github.com/softvibeslab/jev_experimet.git
cd jev_experimet
```

En Cursor: Open Folder → seleccionar `jev_experimet`. Si ya tienes un clon, abre ese clon y revisa su estado antes de hacer pull o cambiar ramas.

```bash
git status --short --branch
git branch --show-current
git remote -v
```

No descartes cambios ajenos ni ejecutes force push. Este repositorio conserva el nombre literal `jev_experimet`.

## 3. Leer el contexto

Lee README y los documentos de `docs/`. Luego pide a Cursor:

> Inspecciona este repositorio sin modificarlo. Identifica qué existe y qué es una propuesta. Usa docs/wiki.md como especificación. No leas .env ni datos locales; no contactes servicios externos. Describe dependencias, contratos y riesgos antes de implementar.

Confirma que distingue el laboratorio Hermes existente de la futura aplicación Streamlit. No copies toda la carpeta de un perfil Hermes al repositorio.

## 4. Crear rama de implementación

Con árbol limpio y rama base revisada:

```bash
git fetch origin
git switch -c feature/normalizacion-mvp origin/main
```

Si hay cambios propios, conserva o termina esos cambios primero. Si la rama ya existe, inspeccionarla antes de reutilizarla.

## 5. Entrega A: modelos y normalización

Prompt para Cursor:

> Implementa únicamente modelos y normalización según docs/data_dictionary.md. Crea pyproject.toml con dependencias justificadas y una versión de Python explícita. Usa uv y pytest. Primero escribe pruebas que fallen: desconocidos, presupuesto sin tipo, inversión sin operación explícita y fechas ambiguas. No conectes APIs ni importes datos reales. Conserva evidencia de cada hecho y devuelve preguntas pendientes. Ejecuta las pruebas y reporta los resultados reales.

Comprobar especialmente:

- Un importe ausente no es cero.
- “10 mil pesos” no confirma precio total ni MXN sin contexto explícito.
- “Fin de año” no confirma fecha de compra.
- “Inversión” no confirma compra.
- Una actualización temporal ambigua no borra una declaración anterior.

## 6. Crear y verificar el entorno

Solo después de que exista un `pyproject.toml` válido con las dependencias de esta entrega:

```bash
uv sync
uv run pytest
```

Versionar `uv.lock` cuando sea generado y revisado. `.venv/` no se versiona. Si el sistema bloquea instalaciones globales, utiliza este entorno; no fuerces pip en el Python del sistema.

Los comandos anteriores son la ruta propuesta para el futuro código. No constituyen evidencia de que la aplicación documental actual ya funcione.

## 7. Entrega B: catálogo y recuperación

Prompt:

> Implementa SQLite, validación e importación manual con fuentes/vigencia. Usa únicamente fixtures ficticios marcados como tales. No accedas a Rovi ni a bases reales. Filtra ciudad, tipo y operación confirmada. Compara solo monedas y categorías de importes equivalentes. Añade pruebas de fuentes vencidas, vigencia desconocida, disponibilidad desconocida y mensualidad versus precio total. No declares compatibilidad por coincidencia parcial.

El catálogo real se incorpora después por un responsable autorizado. Guardarlo localmente fuera del historial Git; nunca presentar un fixture como inventario real.

## 8. Entrega C: UI local

Prompt:

> Implementa Streamlit con los bloques y reglas de estado de docs/wiki.md. Separa hechos, pendientes, catálogo, resultado Jev e interpretación. La pregunta sugerida es editable sin envío automático. Añade revisión humana local, sin botones de agenda o escritura CRM. Mantén execution_authorized=false. No produzcas evaluaciones falsas cuando Jev no esté configurado.

Después de crear `app/ui.py` y declarar Streamlit:

```bash
uv run streamlit run app/ui.py --server.address 127.0.0.1
```

Verificar la UI en el navegador local y sus estados vacíos/errores. En un VPS, elegir un puerto libre comprobado y acceso mediante túnel autorizado; no abrirlo públicamente ni modificar Nginx o servicios existentes por defecto.

## 9. Entrega D: integración real con Jev

Lee [integration.md](integration.md). Hay un contrato HTTP verificado en un laboratorio independiente, pero la aplicación aún debe implementar y probar su propio cliente.

Copia manualmente `.env.example` a un `.env` local con permisos restringidos. Introduce tu clave mediante editor seguro; no la pongas en argumentos de comandos, capturas ni prompts. La UI debe cargarla del lado servidor, nunca en el navegador.

Prompt:

> Implementa el adaptador según docs/integration.md. Endpoint fijo Decisions, contexto mínimo anonimizado, timeout, validación de respuesta y errores sin payloads sensibles. Distingue modelo solicitado y resuelto. Prueba primero sin red mediante dobles de prueba explícitos. Después realiza una prueba real autorizada con un caso ficticio; muestra respuesta real y uso, no una simulación. No integres autenticación Codex en esta entrega ni inventes un endpoint para GPT.

Las pruebas de red gastan crédito y deben marcarse separadamente de tests unitarios. No ejecutarlas automáticamente cada vez que se abre la UI.

## 10. Entrega E: evaluación y piloto

Seguir [evaluation_protocol.md](evaluation_protocol.md). Responsable comercial revisa etiquetas y desacuerdos. Mantener separados ajuste y prueba; no retocar criterios utilizando el conjunto reservado sin volver a separar la evaluación.

Realizar piloto solo con autorización, anonimización revisada y condiciones de tratamiento de datos documentadas. No conectarlo todavía a WhatsApp, Telegram, agenda o CRM.

## 11. Revisión antes del commit

```bash
git status --short
git diff
git diff --check
git diff --cached
```

Ejecutar las pruebas reales del incremento. Revisar que no se incluyen `.env`, originales, bases, logs, sesiones ni cambios ajenos. Si se filtró un secreto, rotarlo y seguir el procedimiento de historial; borrarlo del archivo no elimina exposición previa.

Añadir archivos explícitos revisados; ejemplo para un incremento DOCUMENTAL:

```bash
git add README.md docs/ .gitignore .env.example
git diff --cached
git commit -m "docs: update Jev MVP specification"
git push -u origin HEAD
```

Para código, seleccionar solo los archivos del incremento, no copiar este `git add` a ciegas. No hacer merge ni sobrescribir historial sin autorización.

## 12. Verificar GitHub

- Comparar SHA local y rama remota con `git ls-remote origin`.
- Abrir la rama explícita en GitHub y revisar archivos y enlaces.
- Crear PR si el flujo del equipo lo requiere.
- No declarar CI verde si no existe pipeline o no se ejecutó.

La documentación inicial se publica en main porque el repositorio estaba vacío. Las implementaciones posteriores deben ir en ramas revisables.

## 13. Definición de terminado

Documentación: wiki y onboarding publicados con enlaces válidos y sin datos sensibles.

MVP: aplicación ejecutada, tests de aceptación superados, adaptador probado o estado no configurado claro, datos con procedencia, revisión humana y ausencia de acciones comerciales externas. Las tareas pendientes no se presentan como implementadas.
