# Entregable 1: Diseño de Red

**Competencia:** CE0311 — Conectividad · Área de Infraestructura Tecnológica

## Descripción

Arquitectura de red de bajo costo sobre AWS para el Sistema Financiero Finanzas (Backend Spring Boot + Frontend Angular), incluyendo levantamiento de requerimientos, topología, redundancia y cumplimiento de estándares.

## Contenido

### 1. Levantamiento de Requerimientos

- Contexto organizacional
- Requerimientos técnicos (ancho de banda, latencia, concurrencia)
- Requerimientos de negocio (disponibilidad, crecimiento)
- Fuentes y evidencia

### 2. Diseño de Topología

#### Topología Lógica
- Diagrama de red lógica con componentes AWS
- Flujo de comunicación entre capas (Web → App → DB)

#### Topología Física / Despliegue
- Distribución de instancias EC2 en zonas de disponibilidad
- Ubicación de balanceadores y servicios

#### Segmentación
- **VLAN:** Separación lógica por función (web, aplicación, datos)
- **DMZ:** Zona desmilitarizada para acceso público
- **Subnetting:** Subredes públicas y privadas con CIDR definido

### 3. Redundancia y Alta Disponibilidad

- Alta disponibilidad de bajo costo (Multi-AZ)
- Enlaces redundantes
- Análisis de puntos únicos de falla (SPOF)
- Estrategia de failover

### 4. Cumplimiento de Estándares

- Estándares IEEE 802.3 (Ethernet)
- Modelo OSI / TCP-IP
- Buenas prácticas AWS Well-Architected Framework

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE03%20Infraestructura%20Tecnol%C3%B3gica/Entregable-1%20Dise%C3%B1o%20de%20Red.pdf)
