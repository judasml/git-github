# AGENTS.md — Reglas de trabajo para este repositorio

Proyecto: **empat.IA Evaluaciones** (MK Business Solutions).
Documentos maestros: `docs/product-plan.md`, `docs/mvp-backlog.md`, `docs/data-model.md`. Las decisiones ahí registradas ya están tomadas: no las reabras ni las vuelvas a explicar.

## Estilo de comunicación (obligatorio)

1. **Responde primero, explica después (y solo si hace falta).** La primera línea de tu respuesta debe contestar directamente la pregunta. Máximo 2–4 frases de contexto adicional.
2. **Cero menús de opciones no solicitados.** No ofrezcas "Opción A / Opción B" salvo que el owner pida alternativas explícitamente. Si hay más de un camino, **elige tú el mejor**, dilo en una línea con su razón, y procede.
3. **Una sola pregunta por respuesta, y solo si es bloqueante.** Si puedes avanzar con un supuesto razonable, avanza y declara el supuesto en una línea ("Asumí X; corrígeme si no."). No detengas el trabajo por decisiones menores.
4. **Prohibido repetir contexto del proyecto.** El owner conoce su propio proyecto. No resumas el brief, no listes lo que ya existe, no expliques qué es un PR, un commit o una tabla en cada respuesta.
5. **Sin tablas de estado ni listados de "lo que falta"** salvo petición explícita.
6. **Longitud máxima orientativa: 150 palabras** para preguntas simples; para tareas técnicas, el largo que exija el código o el diff, con explicación mínima.
7. **Cierra siempre con una sola acción siguiente**, no con un menú. Formato: "Siguiente paso: [acción]. ¿Procedo?" — o simplemente procede si ya fue autorizado.
8. Idioma: español. Tono: directo, de colega técnico. Sin frases de relleno ("En palabras simples", "Pero ojo", "Mi recomendación sería").

## Modo de trabajo

- **Sesgo a la acción.** Si la tarea está en `docs/mvp-backlog.md` y fue autorizada, ejecútala sin pedir confirmación intermedia. Confirma solo antes de: borrar datos, cambiar el modelo de datos ya aprobado, o alterar decisiones del product-plan.
- **Commits pequeños y descriptivos.** Un commit por unidad de trabajo. Menciona la ruta de los archivos tocados al reportar.
- **Al terminar una tarea reporta en este formato fijo:**
  - Hecho: [una línea]
  - Archivos: [rutas]
  - Verificado: [comando o prueba ejecutada]
  - Siguiente paso: [uno solo]

## Decisiones técnicas ya cerradas (no reabrir)

- Stack: **Next.js + Supabase + PostgreSQL**.
- Color default `#4A5DA8` como design token configurable por organización (white-label desde el día uno).
- **Modo Anónimo garantizado por arquitectura de datos** (separación de tablas + RLS), nunca solo por filtrado en frontend. Ninguna consulta de administrador puede unir identidad ↔ contenido en modo Anónimo. En modo Confidencial jamás se usa la palabra "anónimo" frente al colaborador.
- **IA en el MVP:** agente de consulta sobre resultados + reporte narrativo (Claude API). Microacciones, conversación sugerida y checkpoints 30/60/90 son fase 2. KIM/KimOrb (personaje visual) es fase 2.
- Orden de construcción: flujo del colaborador (formulario por token) → admin → reportes → IA.
- Escala 1–5, bandas: ≥4.80 Destacado · 4.50–4.79 Competente · 4.00–4.49 En desarrollo · <4.00 Requiere atención (configurables por cliente a futuro).

## Qué hacer ante ambigüedad

Elige la interpretación más simple compatible con `docs/product-plan.md`, decláralo en una línea y continúa. Preguntar es el último recurso, no el primero.
