# Userflows del MVP — modelo definitivo de tres roles

**Versión:** 03/09/2026  
**Perfiles:** público, `EMPLOYEE`, `ADMIN` y `ORG`  
**Alcance:** historias H1–H31

Este documento sustituye cualquier flujo anterior basado en dos roles. La organización social dispone del perfil autenticado `ORG`, pero nunca accede a datos personales o a horas individuales de empleados.

## Fuentes de verdad

- [Contrato canónico de API y estados](https://github.com/Proyecto-Fundacion-Verisure/backend/issues/117)
- [Backlog funcional y técnico](https://github.com/orgs/Proyecto-Fundacion-Verisure/projects/2)
- [Tarea de actualización de userflows](https://github.com/Proyecto-Fundacion-Verisure/documentacion-po/issues/4)

## 1. Acceso, sesión y alta de una entidad

```mermaid
flowchart TD
  A[Landing pública · H3] --> B{¿Qué quiere hacer?}
  B -->|Entrar| C[Login · H1]
  C --> D{Credenciales válidas}
  D -->|No| E[Error genérico 401]
  E --> C
  D -->|Sí| F{Rol y estado}
  F -->|ADMIN activo| G[Dashboard de Fundación]
  F -->|EMPLOYEE activo| H[Catálogo de actividades]
  F -->|ORG activa| I[Panel de la entidad]
  F -->|ORG no verificada| J[Reenviar verificación]
  F -->|ORG pendiente| K[Estado: pendiente de aprobación]
  F -->|ORG rechazada| L[Estado: cuenta rechazada]
  B -->|Registrar entidad| M[Formulario público · H25]
  M --> N{CIF y datos válidos}
  N -->|No| M
  N -->|Sí| O[Correo de verificación]
  O --> P{Enlace vigente}
  P -->|No| J
  P -->|Sí| K
  K --> Q[Revisión ADMIN · H26]
  Q -->|Aprobar| R[Cuenta ORG activa + correo]
  Q -->|Rechazar| L
  G --> S[Cerrar sesión · H2]
  H --> S
  I --> S
  S --> A
```

Reglas clave:

- Una sesión válida se dirige según rol; una cuenta ORG no activa conserva la sesión solo para mostrar su estado.
- Si el CIF ya pertenece a una entidad activa, aprobar añade la nueva persona a esa entidad. Rechazarla no desactiva la entidad ni sus otras cuentas.
- El cierre de sesión limpia token, usuario y cabecera de autenticación en el cliente.

## 2. Propuestas y ciclo de una actividad

```mermaid
flowchart TD
  A[Público envía propuesta · H4] --> B[Propuesta NEW]
  O[ORG envía propuesta propia · H31] --> B
  B --> C[Bandeja ADMIN · H5]
  C -->|Rechazar| D[Propuesta REJECTED + correo]
  C -->|Aceptar| E[Borrador de actividad DRAFT]
  X[ADMIN crea actividad directa · H6] --> E
  E --> F[Editar, previsualizar y guardar]
  F -->|Publicar| G[Actividad PUBLISHED + correo]
  G --> H[Gestión ADMIN · H7]
  H -->|Editar| G
  H -->|Cancelar con confirmación| I[Actividad CANCELLED + avisos]

  J[ORG crea actividad · H28] --> K[Borrador editable DRAFT]
  K -->|Guardar| K
  K -->|Enviar a revisión| L[PENDING_APPROVAL y bloqueada]
  L --> M[Cola ADMIN · H29]
  M -->|Aprobar| G
  M -->|Devolver con nota| N[Devuelta con reviewNote]
  N -->|ORG corrige| K
```

Reglas clave:

- Solo ADMIN publica en el catálogo.
- Una actividad enviada por ORG deja de ser editable hasta que ADMIN la aprueba o devuelve.
- La nota de devolución permanece visible sobre el formulario de ORG.

## 3. Participación del empleado

```mermaid
flowchart TD
  A[EMPLOYEE abre catálogo · H10] --> B[Filtrar y buscar]
  B --> C[Detalle de actividad · H11]
  C --> D{Acción}
  D -->|Favorito| E[Guardar o quitar favorito · H13]
  D -->|Solicitar plaza| F[Confirmación informada · H12]
  F --> G[Inscripción WAITLISTED]
  G --> H[Tablero ADMIN · H8/H24]
  H -->|Rechazar| I[REJECTED + correo]
  H -->|Aceptar y hay plaza| J[CONFIRMED + correo]
  H -->|Aceptar sin plaza| K[WAITLISTED aceptada]
  J --> L[Mis voluntariados · H14]
  K --> L
  L -->|Antes del inicio| M[Desinscribirse · H15]
  M --> N[CANCELLED]
  N --> O{¿Hay espera aceptada?}
  O -->|Sí| P[Promover primera a CONFIRMED]
  O -->|No| Q[Plaza disponible]
  H -->|Dar de baja · H9| N
```

Reglas clave:

- Aceptar una inscripción sin aforo conserva la cola con `accepted=true`; no falla por actividad completa.
- La persona ve su posición en espera, pero no datos de otras personas.
- La baja deja de estar disponible cuando comienza la actividad.

## 4. Cierre, certificado y analítica

```mermaid
flowchart TD
  A[Actividad FINISHED] --> B[Inscripción pendiente de cierre]
  B --> C[EMPLOYEE informa horas, valoración y evidencia · H16]
  C --> D[Report SUBMITTED]
  D --> E[Cola de cierres ADMIN · H17]
  E -->|Devolver con nota| F[Report RETURNED]
  F --> C
  E -->|Validar horas| G[Report VALIDATED]
  G --> H[Participación CLOSED]
  H --> I[Certificado imprimible · H18]
  G --> J[Dashboard Fundación · H19]
  J --> K[Filtros compartibles · H20]
  J --> L[Exportaciones · H21]
  J --> M[Ranking agregado · H22]
  G --> N[Dashboard ORG anonimizado · H30]
```

Reglas clave:

- Solo existe un cierre por inscripción; reenviar uno devuelto actualiza el existente.
- La entidad ve métricas agregadas, nunca nombres, correos, departamentos ni horas individuales.
- Cada gráfico dispone de una tabla equivalente accesible.

## 5. Avisos por correo

```mermaid
flowchart LR
  A[Cuenta ORG registrada] --> A1[Verificar correo]
  B[Cuenta aprobada o rechazada] --> B1[Avisar a ORG]
  C[Actividad publicada o cancelada] --> C1[Avisar a personas afectadas]
  D[Inscripción aceptada o rechazada] --> D1[Avisar a EMPLOYEE]
  E[Plaza liberada] --> E1[Avisar a persona promovida]
  F[Actividad finalizada] --> F1[Pedir cierre a EMPLOYEE]
  G[Cierre validado o devuelto] --> G1[Avisar a EMPLOYEE]
```

Los avisos de H23 son efectos secundarios: no sustituyen el estado visible dentro de la plataforma ni bloquean la operación principal.

## Matriz de rastreabilidad

| Historia | Perfil principal | Paso cubierto |
|---|---|---|
| H1 | Todos | Login, sesión, rutas protegidas y redirección por rol |
| H2 | Todos | Menú de usuario y logout |
| H3 | Público | Landing, misión, cifras y líneas de acción |
| H4 | Público | Envío de propuesta sin cuenta |
| H5 | ADMIN | Revisar, aceptar o rechazar propuesta |
| H6 | ADMIN | Crear, guardar y publicar actividad |
| H7 | ADMIN | Listar, editar y cancelar actividad |
| H8 | ADMIN | Consultar inscripciones, aforo y contadores |
| H9 | ADMIN | Dar de baja y promover desde la lista de espera |
| H10 | EMPLOYEE | Explorar y filtrar catálogo |
| H11 | EMPLOYEE | Consultar detalle de actividad |
| H12 | EMPLOYEE | Solicitar plaza con confirmación |
| H13 | EMPLOYEE | Añadir o quitar favoritos |
| H14 | EMPLOYEE | Consultar voluntariados activos y cerrados |
| H15 | EMPLOYEE | Cancelar la propia inscripción antes del inicio |
| H16 | EMPLOYEE | Crear o corregir el cierre de participación |
| H17 | ADMIN | Validar o devolver cierres |
| H18 | EMPLOYEE | Consultar e imprimir certificado |
| H19 | ADMIN | Consultar KPI y gráficos agregados |
| H20 | ADMIN | Filtrar dashboard y compartir vista |
| H21 | ADMIN | Exportar datos e informes |
| H22 | ADMIN | Consultar ranking agregado |
| H23 | Todos | Recibir los siete grupos de avisos por correo |
| H24 | ADMIN | Aceptar o rechazar inscripciones |
| H25 | ORG | Registrar cuenta, verificar correo y consultar estado |
| H26 | ADMIN | Aprobar o rechazar cuentas ORG |
| H27 | ORG | Consultar panel de actividades y propuestas propias |
| H28 | ORG | Crear borrador y enviar actividad a revisión |
| H29 | ADMIN | Aprobar o devolver actividades de ORG |
| H30 | ORG | Consultar impacto agregado y anonimizado |
| H31 | ORG | Crear y consultar propuestas propias |

## Validación de consistencia

- Los flujos usan `Registration`, `Report`, `Proposal` y `Partner`, sin términos obsoletos como `Enrollment`.
- Se representan los estados de cuenta `PENDING_VERIFICATION`, `PENDING_APPROVAL`, `ACTIVE` y `REJECTED`.
- Se representa `PENDING_APPROVAL` dentro del ciclo de actividad de ORG.
- Las rutas de ORG obtienen la entidad desde la sesión; el flujo no solicita `partnerId` al usuario.
- La matriz cubre H1–H31 sin conservar el modelo anterior de dos roles.

## Revisión

El artefacto requiere aprobación de al menos otra persona del equipo antes de cerrar la tarea #4.

