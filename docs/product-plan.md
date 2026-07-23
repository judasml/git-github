# empat.IA Evaluaciones — Plan de producto y arquitectura

## 1. Decisiones confirmadas

- **Nombre de trabajo:** empat.IA Evaluaciones.
- **White-label desde día uno:** cada organización debe poder configurar `logo`, `wordmark`, `primary_color` y metadatos de marca sin cambiar código.
- **Color por defecto:** `#4A5DA8`, expuesto como token configurable, nunca hardcodeado en componentes.
- **Stack confirmado:** Next.js + Supabase + PostgreSQL.
- **Privacidad como arquitectura:** el modo Anónimo se garantiza a nivel de base de datos con separación de tablas, vistas/RPC seguras y Row Level Security (RLS), no con filtros de frontend.
- **IA en MVP:** agente de consulta sobre resultados y reporte narrativo con Claude API desde la primera versión.
- **Fase 2:** microacciones, conversación jefe-colaborador, checkpoints 30/60/90 y personaje visual KIM/KimOrb.

## 2. Objetivo del MVP

Construir una plataforma web multiusuario donde RR.HH. pueda crear evaluaciones de desempeño 90°/180°/270°/360°, distribuirlas mediante enlaces únicos, recolectar respuestas propias, analizar resultados, generar reporte narrativo y consultar un agente de IA que respete estrictamente el modo de confidencialidad de cada campaña.

## 3. Principios de producto

1. **La promesa al colaborador manda.** Si una campaña es Anónima, ningún usuario administrador ni agente puede ver identidad unida al contenido.
2. **Dueños de la recolección.** Google Forms/Excel quedan como compatibilidad secundaria; la fuente principal son formularios propios.
3. **Mobile-first para colaboradores.** El respondente entra por token, sin cuenta, generalmente desde WhatsApp.
4. **Operación clara para RR.HH.** El dashboard prioriza participación, cobertura, pendientes y recordatorios antes del análisis.
5. **Diseño white-label.** El sistema visual usa tokens por organización y hereda `#4A5DA8` como default.
6. **IA con permisos.** El agente solo consulta datasets autorizados según organización, rol y modo de campaña.

## 4. Arquitectura propuesta

### Frontend / full-stack

- Next.js App Router.
- Server Components para vistas autenticadas con datos agregados.
- Route Handlers / Server Actions para operaciones sensibles.
- Componentes reutilizados/adaptados del Design System en `docs/design-system`.
- CSS variables generadas por organización:
  - `--brand-primary`
  - `--brand-logo-url`
  - `--brand-wordmark`
  - tokens derivados para hover, focus, chart colors y estados.

### Backend y datos

- Supabase Auth para usuarios administradores.
- PostgreSQL como fuente de verdad.
- RLS habilitado en tablas expuestas.
- Funciones SQL/RPC para:
  - resolver tokens públicos de invitación,
  - insertar respuestas,
  - calcular agregados,
  - entregar contexto seguro al agente,
  - bloquear cambios de modo después de la primera invitación.

### IA

- Claude API para:
  - agente de consulta en lenguaje natural,
  - reporte narrativo ejecutivo.
- El agente no consulta tablas crudas directamente desde el frontend.
- El agente recibe contexto desde una capa segura de datos:
  - datasets agregados para campañas Anónimas,
  - datasets atribuidos solo para campañas Confidenciales y roles autorizados.
- Registrar cada consulta en `agent_query_logs` con organización, usuario, campaña, propósito, fuentes consultadas y timestamps.

## 5. Módulos del MVP

### 5.1 Administración

- Login/logout.
- Selector de organización si un usuario pertenece a varias.
- Dashboard operativo por campaña:
  - campañas activas,
  - porcentaje de avance,
  - pendientes,
  - cobertura por fuente,
  - candado de umbral mínimo de anonimato,
  - copiar enlaces o preparar mensajes manuales.

### 5.2 Constructor de evaluaciones

- Crear campaña.
- Elegir tipo: 90°, 180°, 270° o 360°.
- Elegir modo: Anónima por defecto o Confidencial.
- Modo bloqueado después de enviar/generar la primera invitación.
- Seleccionar plantilla por departamento.
- Editar competencias y preguntas.
- Definir fuentes aplicables.
- Configurar bandas de desempeño, inicialmente con defaults:
  - Destacado: ≥ 4.80
  - Competente: 4.50–4.79
  - En desarrollo: 4.00–4.49
  - Requiere atención: < 4.00

### 5.3 Invitaciones y recolección

- Token único por invitación.
- El respondente no necesita cuenta.
- Formulario responsive/mobile-first.
- Aviso de privacidad calculado por modo:
  - Anónimo: promete anonimato real del contenido.
  - Confidencial: informa transparencia de atribución; no usa la palabra “anónimo”.
- Prevención de doble envío.
- Participación visible para RR.HH. en ambos modos.

### 5.4 Análisis y reportes

- Ranking por promedio general.
- Ranking por competencia.
- Radar por competencia y fuente.
- Mapa de calor gerentes × competencias.
- Brechas por fuente.
- Comentarios:
  - mezclados/no atribuibles en modo Anónimo,
  - atribuibles en modo Confidencial.
- Reporte ejecutivo HTML/PDF.
- Reporte narrativo generado con Claude API.

### 5.5 Agente de IA

- Chat en lenguaje natural para RR.HH.
- Preguntas objetivo:
  - “¿Cuáles son las competencias más débiles de Operaciones?”
  - “Resume el feedback del gerente X.”
  - “¿Qué brechas son más urgentes?”
- Respuestas con alcance limitado por:
  - organización,
  - rol,
  - campaña,
  - modo de confidencialidad,
  - umbral mínimo de agregación.

## 6. Pantallas iniciales

1. Login.
2. Dashboard de campañas.
3. Crear evaluación.
4. Editor de plantilla/preguntas.
5. Gestión de evaluados e invitaciones.
6. Formulario público por token.
7. Gracias / enviado.
8. Resultados generales.
9. Reporte individual.
10. Chat de IA sobre resultados.
11. Configuración de organización y marca.

## 7. Uso del Design System

El Design System subido por el owner sirve como referencia para:

- tokens de color, tipografía, espaciado, radio y sombra;
- UI mobile-first para respondentes;
- dashboard operativo de RR.HH.;
- resultados individuales;
- tono de contenido en español;
- comportamiento white-label.

La implementación debe migrar esos conceptos a componentes Next.js mantenibles, evitando copiar valores visuales como literales cuando correspondan a tokens.

## 8. Riesgos principales

- **Privacidad mal implementada:** mitigación con separación física/lógica de tablas, RLS, vistas seguras y pruebas de privacidad.
- **IA filtrando datos indebidos:** mitigación con contexto generado por funciones seguras, no acceso directo del modelo a la BD.
- **Sobrecosto LLM:** mitigación con logs, límites por organización y caché de reportes narrativos.
- **Complejidad de constructor:** mitigación con plantillas base y edición incremental.
- **Promesas inconsistentes:** mitigación con textos automáticos por modo y bloqueo de modo tras invitaciones.

## 9. Orden recomendado de construcción

1. Base Next.js + theme tokens white-label.
2. Supabase schema inicial + RLS base.
3. Flujo público de colaborador por token.
4. Auth admin + organizaciones.
5. Constructor simple de campañas.
6. Invitaciones y participación.
7. Análisis agregado.
8. Reporte individual.
9. Agente IA con capa segura de contexto.
10. Exportación y polish visual.
