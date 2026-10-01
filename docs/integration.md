# Integración Jev: contrato y límites

## Lo verificado y lo pendiente

En un laboratorio Hermes independiente se ejecutaron peticiones reales a OpenRouter y llamadas MCP desde el modelo conversacional. El contrato HTTP no es supuesto. La futura aplicación de este repositorio todavía no tiene cliente implementado.

Pendiente antes de usar información real: revisar políticas actuales de OpenRouter y TypeSafe, conservación de datos, acceso, región si corresponde y autorización para el catálogo/casos. Una API funcional no demuestra que esté aprobado el tratamiento de datos.

Fuentes oficiales:

- https://openrouter.ai/docs/guides/community/jev
- https://openrouter.ai/docs/api/api-reference/alphadecisions/submit-a-decisions-questions-and-answers-request
- https://openrouter.ai/docs/cookbook/building-agents/gate-tool-calls-with-jev

El endpoint está en una superficie alpha: comprobar el contrato vigente antes de implementar y mantener tests de esquema.

## Transporte y autenticación

```text
POST https://openrouter.ai/api/alpha/decisions
Authorization: Bearer <clave resuelta del entorno del servidor>
Content-Type: application/json
```

Modelo solicitado: `typesafe/jev-1.13`.
Variable para este contrato: `OPENROUTER_API_KEY`.

No usar `/chat/completions`: Jev no genera texto. No poner el secreto en Git, navegador, prompts ni logs. No copiar auth.json o .env de Hermes al repositorio.

## Entrada mínima

Ejemplo FICTICIO de petición, no un resultado:

```json
{
  "model": "typesafe/jev-1.13",
  "state": {
    "declaraciones_prospecto": {
      "mensaje_anonimizado": "Busco un departamento en Tulum para inversion.",
      "normalizacion": {
        "operacion_declarada": "desconocido",
        "tipo_inmueble": "departamento"
      }
    },
    "contexto_catalogo": [],
    "politica": "Modo observacion. No inferir operacion a partir de inversion."
  },
  "questions": {
    "operacion": {
      "type": "choice",
      "instructions": "Clasifica solo la operacion explicitamente declarada por el prospecto, no la operacion del catalogo.",
      "criteria": {
        "compra": "Compra explicitamente declarada",
        "renta": "Renta explicitamente declarada",
        "indefinido": "Operacion ausente, ambigua o contradictoria"
      }
    }
  }
}
```

Los IDs internos no entran en state. Separar evidencia del catálogo y palabras del prospecto. Las cargas de comparación entre variantes deben documentar qué campo añade cada variante.

## Primitivas de respuesta

| Tipo | Campos documentados |
| --- | --- |
| noul | type, noul: probabilidad en [0,1]. |
| choice | type, choice, probabilities por opción, confidence. |
| score | type, score sobre índices de una rúbrica ordenada, legend, probabilities, confidence. |

Envelope: `model`, `answers`, `usage`; `id` y `provider` cuando se devuelvan. `usage.cost` cuando exista sirve para reconciliar gasto; no deducir costos ficticios.

En las pruebas reales el modelo solicitado `typesafe/jev-1.13` se resolvió como `typesafe/jev-1.13-20260917`. Conservar ambos. No reemplazar el token solicitado ni aceptar cualquier modelo por conveniencia: validar una lista explícita o el patrón fechado de esa misma versión, revisándolo frente a documentación actual.

No hay razonamiento ni explicación textual devuelta por Jev. Una explicación de GPT o plantilla debe etiquetarse como interpretación propia.

## Contrato local del adaptador

En éxito:

- ok=true.
- status=evaluado_solo_observacion.
- execution_authorized=false.
- Modelo solicitado/resuelto y versión local de criterios.
- Respuesta estructurada validada.
- Metadatos reales de uso, latencia, proveedor e ID cuando existan.

En fallo:

- ok=false.
- status=sin_evaluacion.
- execution_authorized=false.
- Mensaje de UI: “No hubo evaluación”.
- Error categorizado sin secreto, carga ni respuesta cruda sensible.

`ok`, `status`, `execution_authorized` y la versión de criterios son campos del adaptador local, no se debe afirmar que todos provienen de OpenRouter.

## Validación y fallos

- Límite de contexto antes de enviar; presupuesto configurable, inicialmente fragmentos cortos.
- Timeout finito; reintentos acotados solo donde proceda, sin bucles ilimitados ni duplicación silenciosa de gasto.
- Distinguir 401/403, 402, 429, 5xx y errores de red.
- Rechazar JSON inválido, respuestas incompletas, tipos erróneos, opciones desconocidas, NaN, infinito y probabilidades fuera de rango.
- No coercionar booleanos a números.
- Validar puntuación contra índices de la rúbrica y conjunto de opciones contra criterios solicitados.
- No interpretar `confidence` como permiso ni garantía.
- Fallos no producen puntuaciones sustitutivas ni autorizaciones.

## Privacidad y registros

Enviar solo información anonimizada mínima, con revisión manual en piloto. Evitar direcciones precisas, nombres de clientes, documentos originales, teléfonos, correos e identificadores. Los filtros no garantizan anonimización perfecta.

Audit general: ID local, versión de criterios, modelo, latencia, uso y resultado permitido. No guardar cargas completas en logs. Si la evaluación requiere conservar entrada/contexto, hacerlo solo en almacén local autorizado, no en Git, con retención y control de acceso.

## Integración con conversación

En Hermes puede exponerse por MCP stdio de solo lectura. Usar entorno propio y verificar `FastMCP`, descubrimiento y llamadas reales. Un MCP opcional no obliga al agente a consultarlo ni impide acciones por otros medios.

Este MVP no dispone de ejecutor comercial: nunca concede permisos. Si en el futuro se añaden operaciones externas, los controles deterministas y confirmaciones deben estar dentro del ejecutor, sin rutas que eludan la política.

`gpt-6.1-sol` se verificó en Hermes mediante `openai-codex`. Su uso fuera de Hermes requiere contrato separado; no se inventa una API ni se presupone compatibilidad con OpenRouter.
