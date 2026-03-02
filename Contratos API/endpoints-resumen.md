# Resumen de Endpoints — API REST
## Sistema de Triage y Gestión de Solicitudes Académicas

**Base URL:** `http://localhost:8080`  
**Formato de respuesta:** `application/json`  
**Versión OpenAPI:** 3.0.3

---

## Solicitudes — `/api/solicitudes`

| Método | Endpoint                                    | Descripción                                              | RF         | Estado entrada  | Estado salida  |
|--------|---------------------------------------------|----------------------------------------------------------|------------|-----------------|----------------|
| `POST` | `/api/solicitudes`                          | Registrar nueva solicitud académica                      | RF-01      | —               | REGISTRADA     |
| `GET`  | `/api/solicitudes`                          | Consultar todas (con filtros opcionales)                 | RF-07      | —               | —              |
| `GET`  | `/api/solicitudes/{id}`                     | Obtener solicitud con historial completo                 | RF-06, RF-07| —              | —              |
| `PUT`  | `/api/solicitudes/{id}/clasificar`          | Clasificar tipo y calcular prioridad automática          | RF-02, RF-03| REGISTRADA     | CLASIFICADA    |
| `PUT`  | `/api/solicitudes/{id}/priorizar`           | Ajustar prioridad manualmente (con justificación)        | RF-03      | cualquiera      | sin cambio     |
| `PUT`  | `/api/solicitudes/{id}/estado`              | Cambiar estado (transición validada)                     | RF-04      | ver tabla abajo | siguiente      |
| `PUT`  | `/api/solicitudes/{id}/asignar`             | Asignar responsable activo                               | RF-05      | CLASIFICADA     | EN_ATENCION    |
| `PUT`  | `/api/solicitudes/{id}/cerrar`              | Cerrar solicitud con observación obligatoria             | RF-08      | ATENDIDA        | CERRADA        |
| `GET`  | `/api/solicitudes/estado/{estado}`          | Consultar por estado específico                          | RF-07      | —               | —              |
| `GET`  | `/api/solicitudes/solicitante/{id}`         | Consultar por solicitante                                | RF-07      | —               | —              |
| `GET`  | `/api/solicitudes/responsable/{id}`         | Consultar por responsable asignado                       | RF-07      | —               | —              |

---

## Usuarios — `/api/usuarios`

| Método | Endpoint                          | Descripción                                      | RF         |
|--------|-----------------------------------|--------------------------------------------------|------------|
| `POST` | `/api/usuarios`                   | Registrar nuevo usuario                          | RF-05, RF-13|
| `GET`  | `/api/usuarios`                   | Listar todos los usuarios                        | RF-13      |
| `GET`  | `/api/usuarios/{id}`              | Obtener usuario por ID                           | RF-13      |
| `GET`  | `/api/usuarios/rol/{rol}`         | Filtrar usuarios por rol                         | RF-13      |
| `GET`  | `/api/usuarios/responsables-activos` | Listar responsables disponibles para asignación| RF-05      |
| `PUT`  | `/api/usuarios/{id}/desactivar`   | Desactivar usuario                               | RF-05      |
| `PUT`  | `/api/usuarios/{id}/activar`      | Activar usuario                                  | RF-05      |

---

## Request Bodies — Referencia Rápida

### POST `/api/solicitudes` — RF-01
```json
{
  "descripcion": "Solicito homologación de Cálculo I cursada en otra universidad",
  "canalOrigen": "CORREO",
  "solicitanteId": 1,
  "fechaLimite": "2026-03-15"
}
```

### PUT `/{id}/clasificar` — RF-02, RF-03
```json
{
  "tipoSolicitud": "HOMOLOGACION",
  "usuarioResponsableId": 2
}
```

### PUT `/{id}/priorizar` — RF-03
```json
{
  "prioridad": "CRITICA",
  "justificacion": "Cierre de período en 3 días, impacto directo en matrícula",
  "usuarioResponsableId": 2
}
```

### PUT `/{id}/estado` — RF-04
```json
{
  "nuevoEstado": "ATENDIDA",
  "observaciones": "Solicitud procesada satisfactoriamente",
  "usuarioResponsableId": 2
}
```

### PUT `/{id}/asignar` — RF-05
```json
{
  "responsableId": 3,
  "usuarioResponsableId": 2,
  "observaciones": "Asignado al Dr. Martínez por especialidad"
}
```

### PUT `/{id}/cerrar` — RF-08
```json
{
  "observacionCierre": "Homologación aprobada. Resolución N° 2026-045 firmada.",
  "usuarioResponsableId": 2
}
```

### POST `/api/usuarios` — RF-05, RF-13
```json
{
  "identificacion": "1094123456",
  "nombre": "Juan",
  "apellido": "García",
  "email": "juan.garcia@uniquindio.edu.co",
  "rol": "ESTUDIANTE",
  "password": "password123"
}
```

---

## Respuesta Estándar (ApiResponseDTO)

```json
{
  "exitoso": true,
  "mensaje": "Solicitud registrada exitosamente",
  "datos": { ... }
}
```

### Respuesta de Error

```json
{
  "exitoso": false,
  "mensaje": "Transición inválida: la solicitud está en estado CERRADA",
  "datos": null
}
```

---

## Códigos HTTP utilizados

| Código | Uso                                                                    |
|--------|------------------------------------------------------------------------|
| `200`  | Operación exitosa (GET, PUT)                                           |
| `201`  | Recurso creado exitosamente (POST)                                     |
| `400`  | Datos de entrada inválidos, validaciones fallidas                      |
| `404`  | Recurso no encontrado (solicitud o usuario)                            |
| `409`  | Conflicto de negocio: transición inválida, responsable inactivo, etc. |
| `500`  | Error interno del servidor                                             |

---

## Parámetros de Consulta — GET `/api/solicitudes`

| Parámetro      | Tipo              | Requerido | Ejemplo          | Descripción                    |
|----------------|-------------------|-----------|------------------|--------------------------------|
| `estado`       | EstadoSolicitud   | No        | `REGISTRADA`     | Filtrar por estado             |
| `tipo`         | TipoSolicitud     | No        | `HOMOLOGACION`   | Filtrar por tipo               |
| `prioridad`    | Prioridad         | No        | `ALTA`           | Filtrar por prioridad          |
| `responsableId`| Long              | No        | `3`              | Filtrar por responsable        |

**Ejemplo:**
```
GET http://localhost:8080/api/solicitudes?estado=REGISTRADA&prioridad=ALTA
GET http://localhost:8080/api/solicitudes?tipo=HOMOLOGACION&responsableId=3
```

---

## Valores de Enumeraciones

### CanalOrigen
`CSU` | `CORREO` | `SAC` | `TELEFONICO` | `PRESENCIAL`

### TipoSolicitud
`REGISTRO_ASIGNATURAS` | `HOMOLOGACION` | `CANCELACION_ASIGNATURAS` | `SOLICITUD_CUPOS` | `CONSULTA_ACADEMICA`

### EstadoSolicitud
`REGISTRADA` → `CLASIFICADA` → `EN_ATENCION` → `ATENDIDA` → `CERRADA`

### Prioridad (ordenado de menor a mayor urgencia)
`BAJA` (1) | `MEDIA` (2) | `ALTA` (3) | `CRITICA` (4)

### Rol
`ESTUDIANTE` | `DOCENTE` | `ADMINISTRATIVO` | `RESPONSABLE`
