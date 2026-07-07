# Entregable 1: Requerimientos y Diseño del Sistema

**Competencia:** CE021 — Ingeniería de Software

## Descripción

Documento completo de especificación de requerimientos y diseño del sistema Finanzas-Clinic, incluyendo prototipos navegables, diagramas UML y decisiones arquitectónicas.

## Contenido

### 1. Especificación de Requerimientos

#### Requerimientos Funcionales (RF)

| Código | Descripción | Prioridad |
|---|---|---|
| RF-01 | Gestión de usuarios con roles y permisos | Alta |
| RF-02 | Registro y control de caja chica | Alta |
| RF-03 | Facturación electrónica | Alta |
| RF-04 | Conciliación de convenios con aseguradoras | Alta |
| RF-05 | Kardex de productos y control de inventarios | Alta |
| RF-06 | Reportes financieros en tiempo real | Media |
| RF-07 | Gestión de compras a proveedores | Media |

#### Requerimientos No Funcionales (RNF)

- **Rendimiento:** Respuesta < 2 segundos en operaciones críticas
- **Disponibilidad:** 99.9% en horario comercial
- **Seguridad:** Encriptación TLS 1.3, autenticación JWT
- **Escalabilidad:** Soporte para 200+ usuarios concurrentes

#### Historias de Usuario

Ejemplo de historias de usuario definidas para el backlog ágil:

> **HU-01:** Como cajero, quiero registrar un cobro mixto (efectivo + convenio) para que el paciente pueda pagar con su seguro.
>
> **HU-02:** Como administrador, quiero visualizar el arqueo diario automatizado para cuadrar caja sin errores.
>
> **HU-03:** Como contador, quiero generar reportes de depreciación de activos para cumplir con obligaciones tributarias.

### 2. Prototipo Navegable

- Mapeo de interfaces gráficas
- Flujo de navegación del usuario (User Flow)
- Validación con stakeholders

### 3. Diseño Arquitectónico

- **Diagrama de Componentes:** Módulos del sistema y sus interacciones
- **Diagrama de Despliegue:** Nginx → Angular (S3/CDN) → Spring Boot (EC2) → PostgreSQL (RDS)
- **Registro de Decisiones Arquitectónicas (ADR):** Justificación de cada decisión técnica

### 4. Diagramas UML

- **Diagrama de Casos de Uso:** Actores y funcionalidades del sistema
- **Diagrama de Secuencia:** Flujo de venta mixta (convenio asegurado)
- **Diagrama de Estados:** Ciclo de vida de la caja chica

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE02%20Ingenier%C3%ADa%20de%20Software/CE021-Entregable%201-Requerimientos%20y%20Dise%C3%B1o%20del%20Sistema.pdf)
