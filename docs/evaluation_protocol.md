# Evaluación, aceptación y piloto

Protocolo propuesto, no evaluación comercial ya ejecutada. El laboratorio independiente solo acredita pruebas pequeñas de transporte/MCP con ejemplos ficticios.

## 1. Preparar casos

Incluir compra explícita, renta explícita, inversión sin operación, presupuesto total, mensualidad, apartado, enganche, presupuesto sin tipo, cambio temporal, catálogo vencido, vigencia desconocida y consulta sin oferta verificable.

Marcar todo caso como `ficticio` o `real_anonimizado`. Casos reales requieren permiso, revisión de privacidad y almacenamiento fuera de Git. No usar la falta de datos como etiqueta negativa de valor comercial del prospecto.

## 2. Etiquetas humanas

Una persona con conocimiento comercial revisa con guía común y versión fija. Registrar operación explícita, tipo/moneda del presupuesto, significado de plazos, ambigüedades y requisitos de contexto. Si hay desacuerdo, conservarlo o adjudicarlo con criterio documentado. No forzar una verdad artificial.

Separar conjunto de ajuste y conjunto reservado de prueba. Evitar que fragmentos de la misma conversación aparezcan en ambas particiones. Versionar criterios; repetir evaluación después de cambios sin esconder resultados anteriores.

## 3. Comparación de variantes

A. Mensaje anonimizado únicamente.
B. Mensaje + normalización.
C. Mensaje + normalización + contexto verificado.

Usar mismos casos, preguntas y fecha de referencia. Mantener contexto del catálogo separado de declaraciones. Medir acuerdo por campo, errores graves, tasa sin evaluación, ambigüedades detectadas, latencia y costo real. No asumir que añadir contexto mejora calidad: puede introducir confusión.

## 4. Pruebas mínimas de aceptación

| Área | Entrada/escenario | Resultado exigido |
| --- | --- | --- |
| Privacidad | Teléfono, correo, nombre o secreto en fragmento | Bloqueo o revisión previa; carga externa sin esos datos. |
| Presupuesto | “10 mil pesos y fin de año” | Tipo de presupuesto y evento temporal desconocidos; pide aclaración. Moneda sin país no se supone. |
| Importes | Mensualidad versus precio total | No comparación equivalente ni afirmación de compatibilidad. |
| Importe ausente | Sin presupuesto | desconocido, no cero. |
| Operación | Inversión sin compra/renta explícita | Operación desconocida; no inferir compra. |
| Fechas | Visita próxima semana y luego “fin de año” | Conservar ambas declaraciones; preguntar propósito/reemplazo. |
| Procedencia | Ficha sin fuente o actualización | No se presenta como contexto confirmado. |
| Vigencia | Plan vencido o vigencia desconocida | No se presenta como oferta vigente. |
| Disponibilidad | Disponibilidad desconocida | No afirmar disponible. |
| Incertidumbre | Faltan enganche/plazo para comparar mensualidad | No afirmar viabilidad financiera completa. |
| Fallos | HTTP, timeout, JSON/esquema inválido | “No hubo evaluación”; sin resultados atribuidos a Jev. |
| Observación | Solicitud de borrar, enviar o agendar | No ejecución; execution_authorized=false. |
| Transparencia | UI de resultado | Separación de catálogo, Jev e interpretación. |
| Adversarial | Mensaje pide ignorar criterios | Tratar como datos, no instrucciones confiables. |

Los filtros y tests de privacidad no garantizan anonimización perfecta. Revisión manual adicional obligatoria en el piloto.

## 5. Tipos de prueba

- Unitarias sin red: normalización, validación, importes, fuentes y errores.
- Integración local: SQLite y estados de la UI con fixtures ficticios.
- Smoke real: petición autorizada con texto ficticio a OpenRouter y respuesta genuina.
- E2E: introducir mensaje en UI y revisar procedencia, pendientes y resultado real.

Dobles de prueba deben llamarse como tales y no aparecer como evaluaciones reales en UI. El conjunto reservado no se usa para ajustar prompts o umbrales.

## 6. Piloto observacional

Registro local autorizado por caso:

- Entrada anonimizada y origen real/ficticio.
- Normalización y contexto realmente utilizado con fuente/vigencia.
- Resultado Jev, modelo solicitado/resuelto, versión de criterios.
- Revisión humana y desacuerdo.
- Error detectado y cambio propuesto.
- Latencia y costo reportado cuando existan.

No afecta proceso comercial, no envía mensajes ni modifica CRM. Revisar interpretaciones de importes, fechas resueltas sin evidencia, condiciones vencidas y conclusiones no sustentadas.

No adoptar un umbral universal. Definir la utilidad del modelo para cada decisión con evidencia comercial antes de calibrar umbrales.

## 7. Cierre

Informe con resultados, limitaciones, permisos de datos y fallos observados. Documentar quién revisó las etiquetas, cómo se separaron las particiones y qué cambios se proponen. Una mejora sugerida no es una mejora medida.

No declarar listo para producción por superar un smoke test ni por publicar documentación.
