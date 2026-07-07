# Entregable 3: Diseño de Centro de Datos

**Competencia:** CE0331 — Infraestructura · Área de Infraestructura Tecnológica

## Descripción

Arquitectura física y lógica del centro de datos que aloja el sistema Finanzas, abarcando dimensionamiento, layout de racks, virtualización, almacenamiento, energía, refrigeración y continuidad operativa.

## Contenido

### 1. Dimensionamiento de Recursos

| Servicio | Contenedor | CPU | RAM | Almacenamiento |
|---|---|---|---|---|
| Proxy | Nginx | 0.5 vCPU | 512 MB | 2 GB |
| API | Spring Boot 4.0 | 2 vCPU | 4 GB | 10 GB |
| Notificaciones | Notification Service | 1 vCPU | 2 GB | 5 GB |
| Base de datos | PostgreSQL 16 | 2 vCPU | 4 GB | 50 GB (gp3) |
| Mensajería | Kafka | 1 vCPU | 2 GB | 20 GB |
| **Total** | | **6.5 vCPU** | **12.5 GB** | **87 GB** |

**Cuello de botella identificado:** 4 GiB de RAM en t3.medium para PostgreSQL + Kafka.

### 2. Arquitectura por Fases

#### Fase 1 — Piloto (Tier II)
- Instancia EC2 t3.medium (2 vCPU, 4 GiB RAM, 30 GB gp3)
- Recuperación automática y snapshots diarios (DLM)
- Equivalente funcional a centro de datos Tier II

#### Fase 2 — Crecimiento (Tier III)
- Separación de servicios de datos:
    - PostgreSQL → RDS Single-AZ
    - Kafka → MSK Serverless
- Auto Scaling Group con 2 instancias t3.large detrás de ALB
- Mantenimiento concurrente sin interrupción

#### Fase 3 — Producción Completa (Tier III+)
- RDS Multi-AZ (standby en otra AZ)
- MSK Multi-AZ
- ElastiCache Redis
- WAF + CloudFront
- Redundancia completa en todos los componentes críticos

### 3. Almacenamiento y Respaldos

| Tipo | Volumen | IOPS | Respaldos |
|---|---|---|---|
| Sistema | 10 GB gp3 | 3000 | Snapshot diario (retención 7 días) |
| Datos | 50 GB gp3 | 3000 | Snapshot diario (retención 30 días) |
| Logs | 20 GB gp3 | 3000 | Snapshot semanal (retención 90 días) |

### 4. Continuidad Operativa

| Métrica | Objetivo |
|---|---|
| RTO (Recovery Time Objective) | 4 horas |
| RPO (Recovery Point Objective) | 1 hora |
| Disponibilidad objetivo | 99.9% |
| Estrategia | Backup & Restore → Pilot Light → Warm Standby |

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE03%20Infraestructura%20Tecnol%C3%B3gica/Entregable%203-%20Dise%C3%B1o%20de%20Centro%20de%20Datos.pdf)
