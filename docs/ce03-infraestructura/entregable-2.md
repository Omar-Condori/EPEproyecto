# Entregable 2: Planificación de Seguridad

**Competencia:** CE0321 — Seguridad · Área de Infraestructura Tecnológica

## Descripción

Plan de seguridad integral para el Sistema Financiero Finanzas, incluyendo identificación de activos, análisis de riesgos, políticas de seguridad y plan de continuidad del negocio.

## Contenido

### 1. Identificación de Activos Críticos

| Activo | Tipo | Clasificación |
|---|---|---|
| Base de datos financiera | Datos | Crítico |
| Servidor de aplicaciones Spring Boot | Software | Crítico |
| Credenciales de acceso | Información | Crítico |
| Registros de auditoría | Datos | Alto |
| Infraestructura AWS | Hardware | Alto |

### 2. Análisis de Riesgos

- **Metodología:** MAGERIT / ISO 31000
- **Matriz de riesgos:** Probabilidad vs. Impacto
- **Mapa de calor de riesgos:** Visualización de riesgos críticos
- **Plan de tratamiento:**
    - Mitigación (controles técnicos)
    - Transferencia (seguros, AWS Shield)
    - Aceptación (riesgos residuales)

### 3. Políticas de Seguridad

#### Política de Control de Acceso
- Autenticación multifactor (MFA)
- Principio de mínimo privilegio
- Revisión trimestral de accesos

#### Política de Seguridad de Datos
- Encriptación en reposo (AES-256) y en tránsito (TLS 1.3)
- Clasificación de datos (público, interno, confidencial, restringido)
- Retención y destrucción segura

#### Política de Gestión de Credenciales y Secretos
- AWS Secrets Manager para credenciales
- Rotación automática cada 90 días
- Prohibición de hardcoding

#### Política de Seguridad de Red
- Security Groups con reglas de mínimo acceso
- NACLs a nivel de subred
- WAF para protección contra OWASP Top 10

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE03%20Infraestructura%20Tecnol%C3%B3gica/Entregable2%20Planificaci%C3%B3n%20de%20Seguridad.pdf)
