# Entregable 3: Diseño de Centro de Datos

**Competencia:** CE0331 — Infraestructura · Área de Infraestructura Tecnológica

## Resumen Ejecutivo

Este documento describe el diseño del centro de datos para el sistema FINANZAS-MED, implementado en AWS Cloud siguiendo el AWS Well-Architected Framework y principios de Uptime Institute. Se detalla el dimensionamiento de recursos, la arquitectura por fases de madurez (Tier II → Tier III → Tier III+), el diseño de almacenamiento con gp3 y respaldos automatizados, y el plan de continuidad operativa con RTO 4h y RPO 1h.

---

## 1. Contexto y Alcance

### 1.1 Objetivos
- Proporcionar una infraestructura cloud escalable y segura para el ERP Clínico-Financiero FINANZAS-MED
- Garantizar disponibilidad ≥ 99.9% (objetivo inicial, escalable a 99.95% en fases avanzadas)
- Cumplir con normativas: Ley 29733 (Perú), ISO/IEC 27001, AWS Well-Architected Framework
- Permitir evolución modular sin downtime significativo

### 1.2 Alcance
- Diseño lógico y físico de la infraestructura en AWS
- Dimensionamiento de recursos por servicio
- Plan de respaldo y recuperación ante desastres (DR Plan)
- Estrategia de escalabilidad por fases
- Monitoreo y observabilidad

---

## 2. Arquitectura General

### 2.1 Componentes Principales

| Componente | Tecnología | Propósito |
|------------|------------|-----------|
| Balanceador de Carga | Application Load Balancer (ALB) | Distribuir tráfico HTTPS a instancias EC2 |
| Servidor Web / Proxy | Nginx (Docker) | SSL termination, reverse proxy |
| Aplicación Backend | Spring Boot 4.0.1+ (Java 21, Docker) | Lógica de negocio, APIs REST |
| Notificaciones | Servicio de Notificaciones (Spring Boot, Docker) | Kafka consumer, WebSocket STOMP |
| Base de Datos | PostgreSQL 16 (Docker, luego RDS) | Almacenamiento transaccional y analítico |
| Mensajería Asincrónica | Apache Kafka 3.x (Docker, luego MSK) | Cola de eventos, desacoplamiento |
| Almacenamiento | AWS EBS gp3 | Volúmenes persistentes |
| Gestión de Acceso | AWS IAM + SSM Session Manager | Acceso seguro sin puertos SSH públicos |
| Monitoreo | AWS CloudWatch + CloudTrail | Logs, métricas, auditoría |
| Certificados | Let's Encrypt + Certbot (Docker) | TLS 1.2/1.3 |

---

## 3. Dimensionamiento de Recursos (Fase 1: Piloto)

### 3.1 Instancia EC2 Principal
- **Tipo:** t3.medium
- **vCPU:** 2
- **RAM:** 4 GiB
- **Almacenamiento EBS gp3:**
  - Volumen raíz: 10 GB, 3000 IOPS, 125 MB/s
  - Volumen datos: 50 GB, 3000 IOPS, 125 MB/s
  - Volumen logs: 20 GB, 3000 IOPS, 125 MB/s
- **Recuperación automática:** Habilitada
- **Monitoreo:** CloudWatch Detailed Monitoring

### 3.2 Asignación de Recursos por Contenedor (Docker Compose)

| Servicio | Imagen | vCPU | RAM | Almacenamiento | Puertos |
|----------|--------|------|-----|----------------|---------|
| Nginx Proxy | nginx:alpine | 0.5 | 512 MB | 2 GB | 80, 443 |
| Finanzas-Med API | finanzas-med-api:latest | 2.0 | 4096 MB | 10 GB | 8084, 8086 |
| Notification Service | finanzas-med-notifications:latest | 1.0 | 2048 MB | 5 GB | - |
| PostgreSQL | postgres:16-alpine | 2.0 | 4096 MB | 50 GB | 5432 |
| Kafka | confluentinc/cp-kafka:latest | 1.0 | 2048 MB | 20 GB | 9092 |
| Zookeeper | confluentinc/cp-zookeeper:latest | 0.5 | 512 MB | 5 GB | 2181 |
| Certbot | certbot/certbot:latest | 0.25 | 256 MB | 1 GB | - |

**Nota:** En Fase 1, los recursos se comparten en una sola instancia. Se identifica como cuello de botella potencial la RAM (4 GiB) para PostgreSQL + Kafka; se recomienda migrar a t3.large (8 GiB) o separar servicios en Fase 2.

---

## 4. Arquitectura por Fases de Madurez

### 4.1 Fase 1: Piloto (Tier II, Uptime Institute)
- **Objetivo:** Validar MVP en producción
- **Características:**
  - Single EC2 t3.medium
  - Snapshots diarios automatizados (DLM)
  - Recuperación automática de instancia
  - ALB público (HTTPS)
  - SSL/TLS con Let's Encrypt
  - RTO: 4h, RPO: 1h
  - Disponibilidad objetivo: 99.9%

### 4.2 Fase 2: Crecimiento (Tier III, Uptime Institute)
- **Objetivo:** Escalar a mayor volumen y redundancia básica
- **Cambios clave:**
  - Separar servicios de datos a servicios gestionados:
    - PostgreSQL → Amazon RDS Single-AZ
    - Kafka → Amazon MSK Serverless
  - Auto Scaling Group (ASG) con 2 instancias t3.large detrás de ALB
  - Mantenimiento concurrente sin interrupción del servicio
  - Mejora de RPO a 15 min (RDS Point-in-Time Recovery)
  - Disponibilidad objetivo: 99.95%

### 4.3 Fase 3: Producción Completa (Tier III+, Uptime Institute)
- **Objetivo:** Alta disponibilidad multi-AZ, seguridad enterprise
- **Cambios clave:**
  - RDS Multi-AZ (standby en otra AZ de us-east-1)
  - MSK Multi-AZ
  - Amazon ElastiCache Redis para caching
  - AWS WAF + AWS Shield Standard para protección DDoS
  - AWS CloudFront como CDN para assets estáticos
  - AWS Secrets Manager para credenciales
  - AWS Direct Connect (opcional) para conexión dedicada desde clínicas
  - Redundancia completa en todos los componentes críticos
  - RTO: &lt; 1h, RPO: &lt; 5 min
  - Disponibilidad objetivo: 99.99%

---

## 5. Almacenamiento y Respaldos

### 5.1 Volúmenes EBS gp3
- **Ventajas:** Predecible, escalable independientemente de IOPS y throughput
- **Configuración:**

| Volumen | Tamaño | IOPS Base | Throughput Base | Propósito |
|---------|--------|-----------|-----------------|-----------|
| /dev/xvda (root) | 10 GB | 3000 | 125 MB/s | Sistema operativo, binarios Docker |
| /dev/xvdb (data) | 50 GB | 3000 | 125 MB/s | PostgreSQL data dir, Kafka logs |
| /dev/xvdc (logs) | 20 GB | 3000 | 125 MB/s | Logs de aplicación, Nginx, CloudWatch agent |

### 5.2 Estrategia de Respaldos

| Tipo | Frecuencia | Retención | Método |
|------|------------|-----------|--------|
| Snapshot EBS (data) | Diario (03:00 UTC) | 30 días | AWS DLM Lifecycle Policy |
| Snapshot EBS (root) | Diario (03:30 UTC) | 7 días | AWS DLM Lifecycle Policy |
| Snapshot EBS (logs) | Semanal (domingo 04:00 UTC) | 90 días | AWS DLM Lifecycle Policy |
| WAL PostgreSQL | Continuo | 7 días | Archivo a S3 via `wal-g` |
| Dump lógico PostgreSQL | Diario (02:00 UTC) | 30 días | pg_dump → S3 |

### 5.3 Recuperación de Datos
- **RPO (Recovery Point Objective):** 1 hora (Fase 1) → 15 min (Fase 2) → &lt; 5 min (Fase 3)
- **RTO (Recovery Time Objective):** 4 horas (Fase 1) → &lt; 1h (Fase 3)
- **Procedimiento de DR (Fase 1):**
  1. Identificar AZ de fallo
  2. Restaurar último snapshot EBS a nuevo volumen en AZ diferente
  3. Lanzar nueva instancia EC2 t3.medium con volumen restaurado
  4. Recuperar WAL desde S3 hasta punto de fallo
  5. Reasignar Elastic IP o actualizar DNS Route 53
  6. Validar integridad de datos y servicio
  7. Notificar a stakeholders

---

## 6. Monitoreo y Observabilidad

### 6.1 AWS CloudWatch
- **Métricas:**
  - CPUUtilization, NetworkIn/NetworkOut, DiskReadOps/DiskWriteOps (EC2)
  - FreeableMemory, ReadLatency, WriteLatency, TransactionLogsDiskUsage (RDS en fases avanzadas)
  - Alarmas: CPU &gt; 80% por 5 min, RAM &lt; 512 MB, DiskSpace &lt; 10%
- **Logs:**
  - Log groups para:
    - `/ecs/finanzas-med/nginx`
    - `/ecs/finanzas-med/api`
    - `/ecs/finanzas-med/postgresql`
    - `/ecs/finanzas-med/kafka`
  - Retención: 30 días
  - Metric Filters para patrones de error (HTTP 5xx, SQL exceptions, Kafka lag)

### 6.2 AWS CloudTrail
- Habilitado para todas las regiones
- Logs entregados a bucket S3 con cifrado SSE-KMS
- Retención: 365 días
- Alarmas para eventos críticos:
  - `StopInstances`, `TerminateInstances`, `ModifyInstanceAttribute`
  - `CreateSecurityGroup`, `AuthorizeSecurityGroupIngress`
  - `DeleteBucket`, `PutBucketPolicy`

---

## 7. Costos Estimados (Fase 1: Piloto, us-east-1)

| Servicio | Configuración | Costo Mensual Estimado (USD) |
|----------|---------------|-------------------------------|
| EC2 t3.medium | On-Demand | $33.60 |
| EBS gp3 (80 GB total) | 3000 IOPS, 125 MB/s | $10.40 |
| ALB | 1 LCUs | $16.00 |
| S3 Standard | 10 GB | $0.23 |
| CloudWatch Detailed | 7 métricas custom | $2.10 |
| Data Transfer (estimado 100 GB/mes) | Ingratuito entrada, salida primeros 10 GB | ~$9.00 |
| **Total (estimado)** | | **~$71.33/mes** |

---

## 8. Cumplimiento y Estándares

- **AWS Well-Architected Framework:**
  - Excelencia Operativa: Infraestructura como Código (Docker Compose, futuro Terraform), runbooks documentados
  - Seguridad: IAM least privilege, Security Groups, SSM, TLS everywhere, cifrado en reposo y tránsito
  - Fiabilidad: Recuperación automática, snapshots, DR Plan
  - Eficiencia de Rendimiento: gp3, monitoreo, right-sizing
  - Optimización de Costos: t3 burstable, On-Demand → Savings Plans en fases avanzadas
  - Sostenibilidad: Instancias de tamaño adecuado, apagado en horarios no laborales en piloto
- **Uptime Institute Tier Classification:**
  - Fase 1: Tier II (Capacidad redundante básica)
  - Fase 2: Tier III (Mantenimiento concurrente)
  - Fase 3: Tier III+ (Multi-AZ, casi Tier IV)
- **Normativas:**
  - Ley 29733 de Protección de Datos Personales (Perú): Cifrado, RBAC, logs de auditoría
  - ISO/IEC 27001/27002: Controles de seguridad documentados

---

## Anexos

### Anexo A: Ejemplo docker-compose.yml (simplificado)
```yaml
version: '3.8'
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./certbot/conf:/etc/letsencrypt:ro
      - ./certbot/www:/var/www/certbot:ro
    depends_on:
      - api
    restart: unless-stopped

  api:
    image: finanzas-med-api:latest
    ports:
      - "8084:8084"
      - "8086:8086"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/finanzas_med
      - SPRING_DATASOURCE_USERNAME=fm_admin
      - SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
    depends_on:
      - postgres
      - kafka
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    environment:
      - POSTGRES_DB=finanzas_med
      - POSTGRES_USER=fm_admin
      - POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres/init:/docker-entrypoint-initdb.d:ro
    secrets:
      - postgres_password
    restart: unless-stopped

  kafka:
    image: confluentinc/cp-kafka:latest
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    volumes:
      - kafka_data:/var/lib/kafka/data
    depends_on:
      - zookeeper
    restart: unless-stopped

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    volumes:
      - zookeeper_data:/var/lib/zookeeper/data
    restart: unless-stopped

volumes:
  postgres_data:
  kafka_data:
  zookeeper_data:

secrets:
  postgres_password:
    file: ./secrets/postgres_password.txt
```

---

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE03%20Infraestructura%20Tecnol%C3%B3gica/Entregable%203-%20Dise%C3%B1o%20de%20Centro%20de%20Datos.pdf)
