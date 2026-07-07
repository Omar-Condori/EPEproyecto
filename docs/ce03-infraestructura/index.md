# Infraestructura Tecnológica

## Competencia CE03

Esta área comprende el **diseño de infraestructura tecnológica** sobre AWS, incluyendo conectividad de red, seguridad informática y arquitectura de centro de datos cloud híbrida para el sistema Finanzas.

## Entregables

### [Entregable 1: Diseño de Red](entregable-1.md)

Arquitectura de red de bajo costo sobre AWS, incluyendo topología lógica y física, segmentación VLAN/DMZ, redundancia con alta disponibilidad y cumplimiento de estándares.

### [Entregable 2: Planificación de Seguridad](entregable-2.md)

Identificación de activos críticos, análisis de riesgos con matriz y mapa de calor, políticas de seguridad (control de acceso, datos, credenciales, red) y plan de continuidad.

### [Entregable 3: Diseño de Centro de Datos](entregable-3.md)

Arquitectura física y lógica del centro de datos para 5 contenedores (Nginx, Spring Boot, Notificaciones, PostgreSQL 16, Kafka) sobre AWS, con dimensionamiento, layouts, virtualización, almacenamiento y plan de continuidad.

## Stack de Infraestructura

| Componente | Tecnología AWS |
|---|---|
| **Cómputo** | EC2 (t3.medium → t3.large), Auto Scaling Group |
| **Base de datos** | RDS PostgreSQL 16 (Single-AZ → Multi-AZ) |
| **Mensajería** | MSK Serverless / Kafka |
| **Almacenamiento** | EBS gp3, S3, DLM Snapshots |
| **Red** | VPC, Subnets, ALB, CloudFront, WAF |
| **Contenedores** | Docker Engine sobre EC2 |
| **Cache** | ElastiCache Redis |
