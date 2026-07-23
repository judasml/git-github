# empat.IA Evaluaciones — Modelo de datos MVP

## 1. Objetivo del modelo

El modelo debe soportar evaluaciones 90°/180°/270°/360°, multi-tenant, white-label, dos modos de confidencialidad y agente de IA desde el MVP. La regla central es que en campañas Anónimas ninguna consulta de administrador ni contexto de IA pueda unir identidad del respondente con contenido de respuesta.

## 2. Convenciones

- IDs UUID por defecto.
- `organization_id` en todas las entidades tenant-scoped.
- `created_at`, `updated_at` cuando aplique.
- RLS habilitado en tablas expuestas del esquema público.
- Operaciones sensibles mediante funciones SQL `security definer` cuidadosamente auditadas o API server-side con service role solo en servidor.
- El frontend nunca recibe service role key.

## 3. Entidades principales

### 3.1 organizations

Cuenta cliente / tenant.

Campos sugeridos:

- `id uuid primary key`
- `name text not null`
- `slug text unique not null`
- `default_locale text default 'es-HN'`
- `created_at timestamptz default now()`

### 3.2 organization_branding

Configuración white-label.

Campos sugeridos:

- `organization_id uuid primary key references organizations(id)`
- `wordmark text default 'empat.IA Evaluaciones'`
- `logo_url text null`
- `primary_color text not null default '#4A5DA8'`
- `accent_color text null`
- `font_family text null`
- `updated_at timestamptz default now()`

Regla: `primary_color` alimenta el token CSS `--brand-primary`.

### 3.3 profiles

Perfil extendido para usuarios autenticados de Supabase Auth.

Campos sugeridos:

- `id uuid primary key` — coincide con `auth.users.id`
- `full_name text not null`
- `email text not null`
- `created_at timestamptz default now()`

### 3.4 organization_memberships

Usuarios administradores por organización.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `user_id uuid references profiles(id)`
- `role text check (role in ('owner','admin','analyst','viewer'))`
- `created_at timestamptz default now()`

RLS: un usuario solo ve membresías de organizaciones donde pertenece.

## 4. Evaluaciones

### 4.1 templates

Plantillas reutilizables por organización/departamento.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `name text not null`
- `department text null`
- `description text null`
- `is_default boolean default false`
- `created_by uuid references profiles(id)`
- `created_at timestamptz default now()`

### 4.2 template_competencies

Competencias de una plantilla.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `template_id uuid references templates(id)`
- `name text not null`
- `description text null`
- `display_order int not null`

### 4.3 template_questions

Preguntas de una plantilla.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `template_id uuid references templates(id)`
- `competency_id uuid references template_competencies(id)`
- `text text not null`
- `question_type text check (question_type in ('scale_1_5','yes_no','multiple_choice','comment')) default 'scale_1_5'`
- `source_scope text[] not null` — fuentes aplicables.
- `display_order int not null`
- `required boolean default true`

### 4.4 campaigns

Evaluación/ciclo.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `name text not null`
- `evaluation_type text check (evaluation_type in ('90','180','270','360'))`
- `confidentiality_mode text check (confidentiality_mode in ('anonymous','confidential')) default 'anonymous'`
- `department text null`
- `status text check (status in ('draft','inviting','collecting','closed','archived')) default 'draft'`
- `template_id uuid references templates(id)`
- `starts_at timestamptz null`
- `ends_at timestamptz null`
- `created_by uuid references profiles(id)`
- `created_at timestamptz default now()`

Regla dura: `confidentiality_mode` no se puede cambiar si existe al menos una invitación asociada.

### 4.5 campaign_competencies

Snapshot de competencias para la campaña. Evita que cambios futuros en plantilla alteren resultados históricos.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id)`
- `name text not null`
- `description text null`
- `display_order int not null`

### 4.6 campaign_questions

Snapshot de preguntas para la campaña.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id)`
- `competency_id uuid references campaign_competencies(id)`
- `text text not null`
- `question_type text not null`
- `source_scope text[] not null`
- `display_order int not null`
- `required boolean default true`

### 4.7 subjects

Gerentes/personas evaluadas dentro de una campaña.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id)`
- `full_name text not null`
- `role_title text null`
- `department text null`
- `external_ref text null`
- `created_at timestamptz default now()`

## 5. Invitaciones y participación

### 5.1 respondents

Identidad operativa del respondente para invitación/participación. En modo Anónimo esta tabla no debe poder unirse al contenido de respuesta por consultas admin.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `full_name text not null`
- `email text null`
- `phone text null`
- `external_ref text null`
- `created_at timestamptz default now()`

### 5.2 invitations

Token único por respondente, sujeto y relación.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id)`
- `subject_id uuid references subjects(id)`
- `respondent_id uuid references respondents(id)`
- `relationship text check (relationship in ('boss','peer','team','self','other'))`
- `token_hash text unique not null`
- `status text check (status in ('pending','opened','completed','expired')) default 'pending'`
- `channel text check (channel in ('manual_link','email','whatsapp')) default 'manual_link'`
- `opened_at timestamptz null`
- `completed_at timestamptz null`
- `created_at timestamptz default now()`

Notas:

- Guardar hash del token, no el token plano.
- RR.HH. puede ver `status` para seguimiento en ambos modos.

## 6. Respuestas: separación Anónimo vs Confidencial

### 6.1 response_batches

Contenedor lógico de una respuesta enviada. No guarda identidad directamente.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id)`
- `subject_id uuid references subjects(id)`
- `relationship text not null`
- `submitted_at timestamptz default now()`
- `mode_at_submission text check (mode_at_submission in ('anonymous','confidential')) not null`

En modo Anónimo, esta tabla permite analizar por campaña/sujeto/fuente, pero no por respondente.

### 6.2 response_answers

Contenido de respuestas.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `response_batch_id uuid references response_batches(id)`
- `question_id uuid references campaign_questions(id)`
- `numeric_value numeric null`
- `text_value text null`
- `created_at timestamptz default now()`

RLS/vistas admin no deben exponer caminos hacia `respondent_id` para campañas Anónimas.

### 6.3 confidential_response_links

Tabla que conserva vínculo identidad ↔ contenido solo cuando la campaña es Confidencial.

Campos sugeridos:

- `response_batch_id uuid primary key references response_batches(id)`
- `invitation_id uuid unique references invitations(id)`
- `respondent_id uuid references respondents(id)`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id)`
- `created_at timestamptz default now()`

Reglas:

- Solo se inserta si `campaign.confidentiality_mode = 'confidential'`.
- No existe fila para campañas Anónimas.
- RLS permite lectura solo a roles autorizados de la organización.

### 6.4 anonymous_submission_receipts

Registro técnico para impedir doble envío sin unir identidad a contenido en vistas admin.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id)`
- `invitation_id uuid unique references invitations(id)`
- `submitted_at timestamptz default now()`

Uso:

- Permite marcar invitación completada y evitar doble envío.
- No debe exponerse junto con `response_batches`/`response_answers` a admins.
- Si se usa en función server-side, la función no debe devolver vínculo identidad ↔ contenido.

## 7. Resultados y agregados

### 7.1 scoring_bands

Bandas configurables por organización/campaña.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id) null`
- `label text not null`
- `min_value numeric not null`
- `max_value numeric null`
- `color text null`
- `display_order int not null`

Defaults:

- Destacado: `4.80+`
- Competente: `4.50–4.79`
- En desarrollo: `4.00–4.49`
- Requiere atención: `<4.00`

### 7.2 Vistas/RPC de agregación

Recomendado no exponer consultas crudas para resultados. Crear funciones/vistas como:

- `get_campaign_summary(campaign_id)`
- `get_subject_report(subject_id)`
- `get_heatmap(campaign_id)`
- `get_comments_for_report(subject_id)`
- `get_ai_context(campaign_id, scope)`

Reglas para modo Anónimo:

- Aplicar mínimo `N >= 3` por fuente/grupo antes de devolver desglose.
- Comentarios se devuelven mezclados, sin identidad.
- Nunca devolver `respondent_id`, `invitation_id`, email, teléfono o nombre del respondente con contenido.

Reglas para modo Confidencial:

- Puede devolver atribución solo a roles autorizados.
- El agente puede recibir atribución si el rol lo permite y queda auditado.

## 8. IA y auditoría

### 8.1 ai_reports

Reportes narrativos generados.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `campaign_id uuid references campaigns(id)`
- `subject_id uuid references subjects(id) null`
- `report_type text check (report_type in ('campaign_summary','subject_report'))`
- `model text not null`
- `prompt_version text not null`
- `input_context_hash text not null`
- `content jsonb not null`
- `created_by uuid references profiles(id)`
- `created_at timestamptz default now()`

### 8.2 agent_query_logs

Auditoría de consultas del agente.

Campos sugeridos:

- `id uuid primary key`
- `organization_id uuid references organizations(id)`
- `user_id uuid references profiles(id)`
- `campaign_id uuid references campaigns(id) null`
- `subject_id uuid references subjects(id) null`
- `confidentiality_mode text null`
- `question text not null`
- `answer_summary text null`
- `model text not null`
- `context_scope jsonb not null`
- `used_attributed_data boolean default false`
- `created_at timestamptz default now()`

Regla:

- `used_attributed_data` debe ser `false` para campañas Anónimas.

## 9. Seguimiento posterior al MVP

### 9.1 improvement_plans

Fase 2.

- Plan por sujeto/campaña.
- Generado por IA o manual.

### 9.2 micro_actions

Fase 2.

- Acciones semanales.
- Estado de cumplimiento.

### 9.3 checkpoints

Fase 2.

- 30/60/90 días.
- Reevaluación o registro ligero.

## 10. RLS mínimo esperado

Supabase recomienda combinar Auth con RLS para seguridad de extremo a extremo, y RLS funciona como defensa en profundidad a nivel de Postgres. En este proyecto, eso se traduce en:

1. Habilitar RLS en tablas del esquema público.
2. Crear políticas por organización para usuarios autenticados.
3. No exponer tablas crudas de respuestas cuando haya riesgo de correlación.
4. Usar RPC/vistas seguras para resultados y contexto IA.
5. Mantener service role solo en servidor.
6. Probar que usuarios admin no puedan consultar vínculos prohibidos en modo Anónimo.

## 11. Pruebas obligatorias de privacidad

- Admin de organización A no ve datos de organización B.
- Viewer no puede crear campañas.
- Campaña Anónima:
  - no hay fila en `confidential_response_links`,
  - admin no puede unir `respondents` con `response_answers`,
  - agente no recibe nombres de respondentes,
  - grupos con N < 3 no aparecen desglosados.
- Campaña Confidencial:
  - existe vínculo atribuido,
  - solo roles autorizados pueden verlo,
  - agente registra `used_attributed_data = true` si usa atribución.
- Token completado no permite doble envío.
