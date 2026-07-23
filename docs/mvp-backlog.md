# empat.IA Evaluaciones — Backlog MVP

## Épica 1 — Fundaciones técnicas

### MVP-001 — Crear aplicación Next.js
- Inicializar Next.js con TypeScript.
- Configurar estructura `app/`, `components/`, `lib/`, `styles/`.
- Agregar lint/format básicos.
- Criterio de aceptación: la app corre localmente y despliega una página inicial.

### MVP-002 — Integrar Supabase
- Configurar cliente browser/server.
- Variables de entorno.
- Conexión a Supabase Auth y PostgreSQL.
- Criterio de aceptación: la app puede consultar una tabla de prueba bajo RLS.

### MVP-003 — Tokens white-label
- Crear tabla de configuración de marca por organización.
- Implementar default `#4A5DA8`.
- Exponer CSS variables por organización.
- Criterio de aceptación: cambiar `primary_color` en BD cambia botones, links y acentos sin tocar componentes.

## Épica 2 — Seguridad, organizaciones y usuarios

### MVP-004 — Modelo multi-tenant
- Crear `organizations`, `organization_memberships` y roles.
- RLS por organización.
- Criterio de aceptación: un admin solo ve datos de su organización.

### MVP-005 — Autenticación admin
- Login/logout con Supabase Auth.
- Protección de rutas admin.
- Criterio de aceptación: rutas `/admin/*` requieren sesión.

### MVP-006 — Configuración de organización
- Nombre comercial.
- Wordmark/logo placeholder.
- Color primario.
- Criterio de aceptación: el dashboard carga branding desde la organización.

## Épica 3 — Constructor de evaluaciones

### MVP-007 — Crear campaña
- Nombre.
- Tipo 90/180/270/360.
- Modo Anónima/Confidencial.
- Departamento.
- Fechas.
- Criterio de aceptación: una campaña queda guardada con modo Anónimo por defecto.

### MVP-008 — Bloqueo de modo
- Bloquear cambio de modo después de crear/enviar primera invitación.
- Criterio de aceptación: BD rechaza cambio de modo cuando existen invitaciones.

### MVP-009 — Plantillas, competencias y preguntas
- CRUD de plantillas.
- CRUD de competencias.
- CRUD de preguntas.
- Tipos iniciales: escala 1–5 y comentario.
- Criterio de aceptación: una campaña puede generarse desde plantilla editable.

### MVP-010 — Evaluados/sujetos
- Crear sujetos evaluados por campaña.
- Relacionar departamento/cargo opcional.
- Criterio de aceptación: RR.HH. puede agregar gerentes evaluados.

## Épica 4 — Invitaciones y formulario público

### MVP-011 — Invitaciones por token
- Crear invitación por sujeto, respondente y relación.
- Token único seguro.
- Estados: pendiente, abierta, completada, expirada.
- Criterio de aceptación: cada token abre solo su formulario.

### MVP-012 — Pantalla de entrada del colaborador
- Mostrar campaña, evaluado(s), relación y aviso correcto.
- Sin login.
- Mobile-first.
- Criterio de aceptación: en modo Confidencial no aparece la palabra “anónimo”.

### MVP-013 — Formulario de respuestas
- Escala 1–5 táctil.
- Progreso por pregunta/evaluación.
- Comentario final opcional.
- Criterio de aceptación: las respuestas se guardan correctamente.

### MVP-014 — Evitar doble envío
- Marcar invitación como completada.
- Bloquear reenvío del mismo token.
- Criterio de aceptación: un token completado muestra pantalla de ya enviado.

### MVP-015 — Separación Anónimo/Confidencial al guardar
- En modo Anónimo, guardar contenido sin vínculo consultable por admin hacia identidad.
- En modo Confidencial, guardar vínculo atribuido.
- Criterio de aceptación: pruebas SQL demuestran que admin no puede unir identidad ↔ contenido en campañas Anónimas.

## Épica 5 — Dashboard operativo

### MVP-016 — Dashboard de campaña
- Total invitaciones.
- Completadas.
- Pendientes.
- Cobertura por fuente.
- Umbral mínimo de anonimato.
- Criterio de aceptación: RR.HH. ve participación sin contenido individual en modo Anónimo.

### MVP-017 — Copiar enlaces y mensajes
- Copiar enlace por invitación.
- Copiar mensaje sugerido WhatsApp/correo.
- Criterio de aceptación: RR.HH. puede distribuir manualmente sin API WhatsApp.

### MVP-018 — Recordatorios manuales
- Filtrar pendientes.
- Copiar mensaje de recordatorio.
- Criterio de aceptación: dashboard muestra quién falta sin exponer respuestas.

## Épica 6 — Análisis y reportes

### MVP-019 — Cálculos de resultados
- Promedio por sujeto.
- Promedio por competencia.
- Promedio por fuente.
- Brechas.
- Bandas.
- Criterio de aceptación: resultados coinciden con datos de prueba conocidos.

### MVP-020 — Umbral de anonimato
- Ocultar desglose fuente/grupo con menos de 3 respuestas en modo Anónimo.
- Criterio de aceptación: la UI y RPC no devuelven grupos bajo umbral.

### MVP-021 — Resultados generales
- Ranking.
- Heatmap.
- Filtros por departamento/campaña.
- Criterio de aceptación: RR.HH. puede identificar fortalezas y riesgos por campaña.

### MVP-022 — Reporte individual
- Radar por competencia/fuente.
- Fortalezas.
- Áreas de mejora.
- Comentarios.
- Criterio de aceptación: comentarios anónimos no muestran identidad en modo Anónimo.

### MVP-023 — Exportación HTML/PDF
- Generar reporte presentable.
- Criterio de aceptación: reporte descargable con branding de organización.

## Épica 7 — IA en MVP

### MVP-024 — Capa segura de contexto para IA
- RPC/vistas para datasets permitidos.
- Dataset agregado para modo Anónimo.
- Dataset atribuido solo para modo Confidencial y roles autorizados.
- Criterio de aceptación: el agente nunca recibe identidad unida a contenido en modo Anónimo.

### MVP-025 — Reporte narrativo con Claude API
- Prompt de reporte ejecutivo.
- Entrada estructurada desde resultados seguros.
- Guardar generación/costo estimado.
- Criterio de aceptación: genera narrativa consistente sin exponer datos prohibidos.

### MVP-026 — Chat de IA sobre resultados
- UI de chat para RR.HH.
- Preguntas sobre campaña/resultados.
- Respuestas con citas internas de datos agregados.
- Criterio de aceptación: el chat responde solo dentro del alcance de organización/campaña/modo.

### MVP-027 — Auditoría de agente
- Log de consultas.
- Usuario.
- Organización.
- Campaña.
- Modo.
- Contexto usado.
- Criterio de aceptación: cada consulta queda auditable.

## Épica 8 — Importación secundaria de Excel

### MVP-028 — Plantilla fija de Excel
- Definir formato aceptado.
- Validar columnas.
- Mostrar errores claros.
- Criterio de aceptación: importaciones inválidas no contaminan datos.

### MVP-029 — Importar respuestas legado
- Mapear a campañas/sujetos/preguntas.
- Mantener limitaciones visibles.
- Criterio de aceptación: importación funciona solo con plantilla fija.

## Épica 9 — QA, privacidad y salida MVP

### MVP-030 — Pruebas de privacidad
- Tests de RLS.
- Tests de funciones de contexto IA.
- Tests de anonimato bajo N.
- Criterio de aceptación: suite falla si aparece identidad ↔ contenido en modo Anónimo.

### MVP-031 — Datos demo
- Organización demo MK.
- Campaña demo 270°.
- Resultados ficticios.
- Criterio de aceptación: demo permite recorrer admin, colaborador, reportes e IA.

### MVP-032 — Checklist de lanzamiento
- Variables de entorno.
- Políticas RLS revisadas.
- Avisos legales por modo.
- Límites IA.
- Criterio de aceptación: MVP listo para piloto cerrado.
