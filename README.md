# Jev Experimet — laboratorio inmobiliario en modo observación

Documentación para construir en Cursor un MVP que combine conversación con `gpt-6.1-sol`, decisiones estructuradas de `typesafe/jev-1.13` y contexto inmobiliario verificable.

> Este repositorio contiene documentación y especificaciones, no una aplicación implementada. El laboratorio Hermes se probó por separado; su código, configuración, credenciales y sesiones no se publican aquí.

## Empieza aquí

1. [Onboarding paso a paso en Cursor](docs/onboarding.md).
2. [Wiki del proyecto](docs/wiki.md).
3. [Diccionario de datos y normalización](docs/data_dictionary.md).
4. [Contrato de integración con Jev](docs/integration.md).
5. [Evaluación y pruebas de aceptación](docs/evaluation_protocol.md).

## Flujo del MVP

Mensaje anonimizado → normalización con incertidumbres → contexto vigente del catálogo → evaluación Jev → revisión humana.

`gpt-6.1-sol` conversa y puede sugerir preguntas; Jev devuelve decisiones tipadas y probabilidades, no texto ni razonamiento. SQLite resuelve el catálogo estructurado inicial. Streamlit es la propuesta para una interfaz local. Ninguno de esos componentes de aplicación está implementado aún en este repositorio.

## Límites no negociables

- `execution_authorized=false` en todos los resultados.
- Sin envío de mensajes, agendas, modificación de inventario o escritura en CRM.
- No confundir precio total, mensualidad, apartado y enganche.
- No inferir una operación de compra solo porque el mensaje dice “inversión”.
- No publicar catálogos reales, casos reales, logs, bases de datos ni secretos.
- Un error de Jev se presenta como “No hubo evaluación”, sin resultados inventados.
- Un perfil Hermes no es un sandbox del sistema operativo.

## Estado y evidencia

En un laboratorio Hermes separado se verificó conversación con `gpt-6.1-sol` mediante `openai-codex` y tres herramientas MCP: `clasificar_intencion`, `evaluar_lead` y `revisar_accion`. Se obtuvieron respuestas válidas en 11 casos ficticios; las 9 etiquetas inicialmente esperadas coincidieron y pasaron 7 pruebas unitarias. Son pruebas de integración pequeñas, no una validación comercial ni una garantía de exactitud. No prueban las funciones futuras de normalización, recuperación, UI o importación de catálogo.

La API real utilizada para Jev fue `POST https://openrouter.ai/api/alpha/decisions`, autenticada con `OPENROUTER_API_KEY`. La revisión de políticas de datos y la autorización para casos reales siguen pendientes.

## Desarrollo

El onboarding distingue comandos que pueden ejecutarse ahora de comandos condicionados a crear código y dependencias. No hay todavía `pyproject.toml`, `app/ui.py`, aplicación, datos reales ni suite ejecutable del MVP.

Esta wiki vive en `docs/` y se versiona con Git. No es la función GitHub Wiki, que utiliza un repositorio separado.
