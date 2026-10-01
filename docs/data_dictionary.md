# Diccionario de datos y normalización

Esquema propuesto, aún no implementado. Referencias internas se quedan localmente. No se envían IDs al proveedor.

## Convenciones

- Dato pendiente: valor literal `desconocido` en la representación de dominio.
- Números: decimal exacto o `desconocido`; no usar float binario para dinero.
- En SQLite, importes se pueden guardar como decimal canónico en TEXT, incluido el marcador; validar al escribir. No usar cero como ausencia. Al comparar, parsear solo valores válidos con Decimal.
- Moneda: código explícito como MXN o USD; “pesos” sin país/contexto no confirma una moneda.
- Fechas: ISO cuando hay evidencia; si no, `desconocido` y conservar expresión literal en la declaración. No inventar día/año.
- Fuente: referencia verificable, con acceso local protegido si incluye datos privados.
- Disponibilidad: `disponible`, `no_disponible`, `desconocido`; debe estar respaldada y actualizada.
- Un dato malformado se rechaza. `desconocido` no oculta errores de importación.

## inmuebles

| Campo | Contrato |
| --- | --- |
| clave_interna | Identificador local único; no se envía a Jev. |
| ciudad | Ciudad declarada por la fuente o desconocido. |
| zona | Zona o desconocido. |
| tipo | Departamento, casa, terreno, otro o desconocido; vocabulario controlado. |
| operacion | Compra, renta, venta, otro o desconocido según semántica del catálogo documentada. |
| precio_total | Decimal no negativo o desconocido; no sustituir con mensualidad. |
| moneda | Moneda del precio total o desconocido. |
| disponibilidad | Estado respaldado, nunca inferido por tener una ficha. |
| fuente | Procedencia verificable o desconocido; sin fuente no es contexto confirmado. |
| actualizado_en | Fecha de actualización o desconocido. No demuestra por sí sola vigencia. |

Si la oferta tiene vigencia, añadir campos explícitos `vigente_desde`/`vigente_hasta` y su evidencia; no asumir permanencia a partir de actualizado_en.

## planes_pago

| Campo | Contrato |
| --- | --- |
| inmueble_referencia | Relación local con clave_interna. |
| tipo_plan | Contado, financiamiento u opción documentada; desconocido si falta. |
| apartado | Decimal o desconocido. |
| enganche | Decimal o desconocido. No mezclar importe con porcentaje. |
| mensualidad | Decimal o desconocido. |
| condiciones | Texto documental; aclarar duración, intereses, requisitos y restricciones. |
| vigente_desde | Fecha explícita o desconocido. |
| vigente_hasta | Fecha explícita o desconocido. |
| fuente | Fuente de ese plan, no una atribución inventada. |

Añadir `moneda` explícita para los importes del plan. Si la fuente usa porcentajes, preservar en campos distintos, por ejemplo enganche_porcentaje; no asignarlos al importe. Ausencia de fecha final no significa vigencia ilimitada. Si la fuente confirma vigencia abierta, registrarlo explícitamente con evidencia.

## conocimiento

`tema`, `contenido`, `fuente`, `version`, `vigente_desde`, `vigente_hasta`.

Cada regla/comunicado tiene versión local y procedencia. No presentar una regla interna como evaluación de Jev. Contenido vencido o de vigencia desconocida no se usa como condición comercial confirmada.

## casos_anonimizados

`referencia_local`, `mensaje_anonimizado`, `origen_real_o_ficticio`, `etiqueta_humana`, `version_criterios`.

- origen_real_o_ficticio: `real_anonimizado` o `ficticio`, nunca implícito.
- etiqueta_humana: etiqueta revisada o desconocido. Conservar desacuerdos por separado.
- Agregar partición `ajuste`/`prueba`, fecha de revisión y versión de guía.
- No versionar casos reales, incluso anonimizados, sin autorización expresa; política inicial: fuera de Git.

## Contrato de normalización

| Campo | Significado |
| --- | --- |
| operacion_declarada | Solo operación explícita, o desconocido. |
| proposito | Inversión, vivienda u otro propósito declarado. |
| ubicacion | Ciudad/zona con evidencia; ausentes: desconocido. |
| tipo_inmueble | Tipo explícito o desconocido. |
| presupuesto_importe | Decimal exacto o desconocido. |
| presupuesto_moneda | Moneda confirmada o desconocido. |
| presupuesto_tipo | total, mensualidad, apartado, enganche u otro; desconocido si ambiguo. |
| plazo_visita | Expresión literal y referencia temporal; desconocido si no asignable. |
| plazo_compra | Igual, sin inferir que una visita es una compra. |
| contradicciones | Lista de declaraciones incompatibles y evidencia; vacía si no se detectan, no si no se evaluaron. |
| datos_pendientes | Preguntas/campos necesarios pendientes. |

Agregar `declaraciones_temporales` y evidencia por campo para conservar expresiones no asignables a visita/compra. Conservar orden y relación de reemplazo solo cuando explícita. La fecha de referencia de la conversación es obligatoria antes de resolver fechas relativas.

## Ejemplo ficticio, no evaluación real

Primer mensaje: “Busco departamento en Tulum para inversión; quiero visitar la próxima semana.”
Segundo mensaje: “Tengo 10 mil pesos mexicanos y sería a fin de año.”

Normalización esperada:

```json
{
  "operacion_declarada": "desconocido",
  "proposito": "inversion",
  "ubicacion": {"ciudad": "Tulum", "zona": "desconocido"},
  "tipo_inmueble": "departamento",
  "presupuesto_importe": "10000",
  "presupuesto_moneda": "MXN",
  "presupuesto_tipo": "desconocido",
  "plazo_visita": "proxima semana: declaracion inicial sin fecha resuelta",
  "plazo_compra": "desconocido",
  "declaraciones_temporales": [
    {"texto": "a fin de año", "proposito": "desconocido", "reemplaza": "desconocido"}
  ],
  "contradicciones": [],
  "datos_pendientes": [
    "Confirmar compra o renta",
    "Confirmar si 10 mil MXN es total, mensualidad, apartado o enganche",
    "Confirmar a que evento corresponde fin de año",
    "Confirmar si ese plazo reemplaza la visita inicialmente mencionada"
  ]
}
```

No se declara una contradicción definitiva mientras el segundo plazo no tenga propósito confirmado. La aplicación sí marca posible cambio temporal pendiente de aclaración. Si el texto solo dice “pesos”, no usar automáticamente MXN.

Pregunta sugerida de plantilla, no salida de Jev:

“¿Buscas comprar o rentar? ¿Los 10 mil MXN corresponden al presupuesto total, una mensualidad, apartado o enganche? ¿Fin de año se refiere a la visita, a la compra o a otro momento?”
