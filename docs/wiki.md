# Wiki del MVP inmobiliario con Jev

## 1. Propósito

Evaluar consultas inmobiliarias anonimizadas con hechos declarados, contexto verificable y revisión humana. El caso inicial es la búsqueda de departamentos en Tulum para inversión; no limita el diseño a esa ciudad.

El sistema no decide automáticamente si una persona es un buen prospecto. Ayuda a identificar intención, datos pendientes, contradicciones y preguntas de seguimiento. El propósito “inversión” no confirma por sí mismo una compra.

## 2. Alcance

Incluye captura manual, normalización, consulta de catálogo, evaluación estructurada, trazabilidad, pregunta editable y revisión humana local.

Excluye mensajería automática, agendas, cambios de CRM, modificación de inventario, aprobación/descarte automático, decisiones crediticias y acciones sobre producción. Todos los resultados contienen `execution_authorized=false`.

## 3. Estado actual

Este repositorio comienza como una especificación documental. No contiene una aplicación desplegada ni un catálogo real.

Un laboratorio Hermes independiente verificó el acceso a Jev por HTTP y las llamadas MCP desde `gpt-6.1-sol`. Eso valida el transporte del laboratorio, no un adaptador futuro de esta aplicación ni la calidad comercial del sistema. No se publican sus secretos, sesiones o archivos de estado.

## 4. Arquitectura propuesta

1. Interfaz local recibe un fragmento ya anonimizado.
2. Normalizador representa hechos, su evidencia y sus ambigüedades.
3. Recuperador obtiene contexto pertinente, vigente y trazable.
4. Adaptador valida el contexto mínimo y llama a Jev.
5. Presentador separa declaraciones, catálogo, respuestas del modelo e interpretación.
6. Revisor humano confirma, corrige o marca desacuerdo.

Stack propuesto: Python, Streamlit local, SQLite y pytest, con dependencias fijadas después de una prueba mínima. No se añade una base vectorial ni un grafo en el primer incremento.

### Rol del modelo conversacional

`gpt-6.1-sol` sirve para conversar y proponer aclaraciones. En el laboratorio el proveedor verificado fue `openai-codex`; no debe suponerse que ese identificador está disponible en OpenRouter ni que la autenticación de Hermes funciona en una aplicación externa sin integración adicional.

La primera UI puede usar reglas y plantillas locales para preguntas, sin necesitar un segundo contrato LLM. Si se conecta al agente conversacional, se documentará su transporte y autenticación por separado. Las salidas del asistente no se atribuyen a Jev.

## 5. Organización objetivo

```text
app/                        # FUTURO: no implementado aún
  ui.py
  models.py
  normalization.py
  retrieval.py
  privacy.py
  evaluation.py
  jev_adapter.py
  storage.py
tests/                      # FUTURO: pruebas de cada módulo
  fixtures/                 # solo casos ficticios explícitos
docs/
  wiki.md
  onboarding.md
  data_dictionary.md
  integration.md
  evaluation_protocol.md
data/                       # FUTURO: archivos reales locales ignorados
README.md
.gitignore
.env.example
pyproject.toml              # FUTURO: declarar al implementar
```

## 6. Datos y normalización

El [diccionario](data_dictionary.md) define inmuebles, planes de pago, conocimiento y casos anonimizados. Los identificadores internos relacionan registros localmente, no se mandan a Jev.

Un dato pendiente usa `desconocido`, no cero ni una disponibilidad asumida. Los datos malformados se rechazan: no se corrigen silenciosamente.

El normalizador conserva el fragmento de evidencia y el contexto temporal. Una nueva declaración no reemplaza otra hasta confirmar qué modifica. No resuelve por su cuenta el significado de “fin de año”.

Aceptación central: ante “10 mil pesos y fin de año”, el sistema pregunta tipo de presupuesto y significado del plazo; no da por confirmado un presupuesto total ni una fecha de compra.

## 7. Recuperación estructurada

- Filtrar ciudad y tipo de inmueble cuando estén declarados.
- Filtrar operación únicamente cuando esté confirmada.
- Comparar moneda y tipo de importe equivalentes.
- Seleccionar información vigente a una fecha de referencia explícita.
- Recuperar campos pertinentes, no exportar el catálogo completo.
- Mostrar fuente, versión/actualización y límites de vigencia.

Una mensualidad se compara con la mensualidad de un plan, no con el precio total. Coincidir en mensualidad no prueba viabilidad: pueden faltar enganche, duración, intereses y condiciones. Sin moneda o tipo de presupuesto confirmado no existe comparación válida.

La vigencia del plan no prueba disponibilidad del inmueble. Si no puede confirmarse disponibilidad, mostrar “Disponibilidad desconocida”. Las fuentes vencidas pueden mostrarse como antecedentes, nunca como oferta vigente.

La búsqueda semántica solo se considera después para documentos libres, con recuperación de fuentes y sin reemplazar filtros exactos.

## 8. Adaptador Jev

Ver [contrato y seguridad](integration.md). Usar el endpoint real de decisiones, no chat completions. Separar declaraciones del prospecto y contexto del catálogo en campos diferentes; el catálogo no debe presentarse como declaración del prospecto.

Conservar respuesta estructurada, modelo solicitado y resuelto, versión de criterios y metadatos de uso cuando existan. No inventar campos que la API no entregue.

Errores de autenticación, crédito, red, JSON o esquema producen “No hubo evaluación”. No hay aprobación por defecto, evaluación alternativa atribuida a Jev ni ejecución externa.

## 9. Pantalla de revisión

Bloques visibles:

1. Hechos declarados: valor y evidencia literal.
2. Datos faltantes/contradictorios: qué falta y por qué no se resolvió.
3. Contexto recuperado: fuente, vigencia y disponibilidad verificada o desconocida.
4. Resultado Jev: primitiva, etiqueta/puntuación, probabilidades y versión real.
5. Interpretación del asistente/aplicación: explícitamente separada.
6. Pregunta sugerida: editable, sin botón de envío externo.
7. Revisión humana: confirmado, corregido o desacuerdo, sin decisión comercial automática.

### Estados de la aplicación, no etiquetas de Jev

Reglas propuestas con precedencia de arriba hacia abajo:

| Estado | Regla |
| --- | --- |
| Sin evaluación | La evaluación requerida no se ejecutó, no está configurada o falló. Mostrar causa. |
| Pendiente de aclaración | Hay contradicciones o faltan operación, moneda/tipo de presupuesto o significado de un plazo necesarios para la consulta. |
| Sin contexto vigente | La consulta requiere oferta y ninguna fuente cumple vigencia/procedencia; desconocimiento de vigencia no equivale a vigente. |
| Listo para revisión humana | Evaluación válida, contexto pertinente vigente cuando se requiera y sin ambigüedades esenciales pendientes. No equivale a aprobado. |

Además del estado principal, conservar indicadores independientes; un fallo no debe ocultar datos pendientes ni advertencias del catálogo. Una solicitud de simple clasificación no requiere catálogo y debe declararlo.

## 10. Evaluación y piloto

Ver [protocolo](evaluation_protocol.md). Etiquetar con una guía comercial compartida, conservar desacuerdos, separar ajuste y evaluación. Comparar mensaje, normalización y contexto verificado sobre los mismos casos.

Primero casos ficticios. Casos reales anonimizados solo después de autorización de uso y revisión del tratamiento de datos. Ejecutar piloto sin afectar el proceso comercial. No establecer un umbral universal de confianza.

## 11. Privacidad y trazabilidad

Nombres, teléfonos, correos, identificadores, secretos y originales no entran en Git ni en cargas externas. Los detectores automáticos ayudan, pero no garantizan anonimización. Revisar manualmente cada carga durante el piloto.

La revisión local de un caso real también es información protegida aunque esté anonimizada. Su almacenamiento debe ser autorizado, limitado, con acceso restringido y política de retención. No registrar cargas completas en logs generales; usar el almacén local autorizado si el protocolo necesita conservarlas.

## 12. Grafo: solo con necesidad demostrada

Considerar un grafo si simplifica consultas y trazabilidad que SQLite no resuelve bien:

- Inmueble → ubicado_en → Zona.
- Inmueble → ofrece → PlanPago.
- PlanPago → tiene_condicion → Condicion.
- Condicion → respaldada_por → Fuente.
- Declaracion → reemplaza → DeclaracionAnterior, solo con reemplazo confirmado.

Toda relación debe tener procedencia y vigencia. No migrar solo para “mejorar la IA”.

## 13. Entregas y definición de terminado

A. Modelos y normalización: desconocidos explícitos, evidencia, ambigüedad y pruebas.
B. Catálogo local: importación autorizada, vigencia, disponibilidad y recuperación.
C. UI: bloques separados y ausencia de controles externos.
D. Adaptador: llamadas reales, errores seguros, validación y uso verificable.
E. Evaluación: revisión comercial, métricas por caso y piloto observacional.

El MVP termina cuando la interfaz se ejecuta, las pruebas pasan con evidencia real y la procedencia se conserva; no cuando se crean archivos. Documentación publicada no significa aplicación implementada.

Decisiones pendientes: origen/autorización del catálogo, responsable comercial de etiquetas, política de retención, tratamiento por proveedores y si la UI necesita integrar conversación además del perfil Hermes existente.
