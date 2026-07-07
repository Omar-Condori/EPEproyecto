
# Entregable 1: Diseño de Red

**Competencia:** CE0311 — Conectividad · Área de Infraestructura Tecnológica

---

## Resumen Ejecutivo

Finanzas es un sistema integral de gestión financiera y administrativa desarrollado para operaciones multi-sucursal. La plataforma está compuesta por 6 capas de software: SPA Angular, proxy inverso Nginx, backend Spring Boot 4.0.1, Apache Kafka, PostgreSQL 16 y un microservicio de notificaciones con WebSocket.

El diseño de red se alinea con el modelo de responsabilidad compartida de AWS y cumple con estándares internacionales: ISO/IEC 27001, SOC 1/2/3, IEEE 802.1Q y el marco AWS Well-Architected.

---

## 1. Levantamiento de Requerimientos

### 1.1 Contexto Organizacional

El sistema opera para organizaciones con múltiples sucursales y puntos de venta simultáneos. Cada sucursal opera con su propio contexto, pero la administración central requiere visibilidad consolidada en tiempo real.

**Perfiles de usuario y concurrencia estimada en hora pico:**

| Tipo de usuario | Cantidad estimada | Usuarios concurrentes en pico |
|-----------------|------------------|-------------------------------|
| Operadores por sucursal (ventas, caja, compras) | 15–25 | 12–18 |
| Supervisores y jefes de sucursal | 5–8 | 3–5 |
| Administradores del sistema | 2–3 | 1–2 |
| Gerencia / Reportes | 3–5 | 2–3 |
| **Total estimado en hora pico** | — | **~18–28 usuarios concurrentes** |

### 1.2 Requerimientos Técnicos

#### Componentes y puertos

| Capa | Componente | Tecnología | Puerto |
|------|------------|------------|--------|
| Presentación | SPA Angular | Angular 20 + TailwindCSS | 4200 (dev) / 80 (prod) |
| Proxy | Nginx | Nginx + Let's Encrypt TLS | 80 / 443 |
| Aplicación | Backend Finanzas | Spring Boot 4.0.1 / Java 21 | 8084 |
| Mensajería | Apache Kafka | Kafka Broker | 9092 |
| Tiempo real | Notificaciones Service | Spring Boot 3.5.9 + WebSocket STOMP | 8086 |
| Persistencia | Base de datos | PostgreSQL + Flyway | 5432 |

#### Ancho de banda estimado en hora pico

| Flujo de datos | Tamaño típico por operación | Frecuencia en hora pico | Ancho de banda estimado |
|----------------|-----------------------------|-------------------------|--------------------------|
| Peticiones REST (JSON) | 3–15 KB por petición | ~3,500–5,000 peticiones/hora | ~2.5–4.0 Mbps sostenidos |
| Autenticación JWT | 2–4 KB por petición | ~100 peticiones/hora | ~0.1 Mbps |
| WebSocket notificaciones | ~0.5–1 KB cada 5–10 s por usuario | ~360 msg/usuario/hora | ~0.3–0.5 Mbps |
| Carga inicial SPA Angular | ~3–5 MB por primera carga | ~5–10 primeras cargas/hora | Picos cortos (&lt; 1 min) |
| Subida de archivos | 0.1–3 MB por archivo | ~20–30 archivos/hora | ~0.5 Mbps (esporádico) |
| Consultas a BD (interno) | 10–50 KB por consulta | ~8,000–12,000 consultas/hora | ~3.0–5.0 Mbps (red interna) |
| Eventos Kafka (interno) | 0.5–2 KB por evento | ~1,200 eventos/hora | ~0.2 Mbps (red interna) |
| **Total estimado** | — | — | **&lt; 10 Mbps** |

#### Requerimientos de latencia y disponibilidad

| Componente | Latencia máxima aceptable | Disponibilidad esperada | Ventana de operación |
|------------|---------------------------|-------------------------|---------------------|
| API REST (operaciones transaccionales) | &lt; 500 ms (p95) | 99.5% | Lunes–sábado, 7:00 – 22:00 |
| WebSocket (notificaciones en vivo) | &lt; 200 ms | 99.0% | Mismo horario operativo |
| Carga de frontend (SPA) | &lt; 3 s (primera carga) | 99.5% | 24/7 (lectura) |
| Base de datos (PostgreSQL) | &lt; 100 ms por consulta simple | 99.9% | 24/7 |

### 1.3 Requerimientos de Negocio

1. **Continuidad operativa:** Disponibilidad continua durante horario laboral; graceful degradation en caso de contingencia.
2. **Consolidación multi-sucursal:** Sincronización bidireccional entre sucursales y servidor central.
3. **Seguridad de datos:** Segmentación por zonas de seguridad, cifrado en tránsito y en reposo, RBAC.
4. **Trazabilidad y auditoría regulatoria:** Preservación de IP cliente a través de proxies y balanceadores; logs protegidos.
5. **Escalabilidad progresiva:** Crecimiento sin rediseños costosos; migración a alta disponibilidad (Multi-AZ).
6. **Restricción presupuestaria:** Soluciones de bajo costo para piloto; ruta de migración documentada.
7. **Notificaciones en tiempo real:** Latencia predecible y baja para canales en vivo.

---

## 2. Diseño de Topología

### 2.1 Diagrama de Topología Lógica

La arquitectura se despliega sobre AWS usando una VPC segmentada en subredes que aíslan cada capa del sistema.

**Equivalencias cloud (conceptos tradicionales vs. AWS):**

| Concepto tradicional | Equivalente en AWS |
|----------------------|--------------------|
| VLAN (IEEE 802.1Q) | Subredes de VPC + Security Groups |
| DMZ | Subred pública |
| Switch de capa 2/3 | VPC Route Tables + NACLs |
| NAT Gateway | Internet Gateway (para subred pública; no NAT en piloto para ahorrar costos) |

### 2.2 Diagrama de Topología Física / Despliegue

#### Escenario 1: Piloto (bajo costo)

| Recurso | Especificación | Justificación |
|---------|----------------|---------------|
| Tipo de instancia | EC2 t3.medium (2 vCPU, 4 GiB RAM) | t3.micro insuficiente; t3.medium óptimo costo-rendimiento |
| Arquitectura | x86_64 | Compatibilidad probada con imágenes Docker; evita ARM issues |
| Almacenamiento | EBS gp3 — 30 GB | Base de datos + logs Kafka + aplicación; 3000 IOPS base |
| Sistema operativo | Ubuntu Server 24.04 LTS | Mayor compatibilidad Docker; soporte LTS hasta 2029 |
| IP elástica | 1 dirección IPv4 pública | Mantiene misma IP si se detiene/inicia la instancia |

#### Escenario 2: Arquitectura objetivo (producción con alta disponibilidad)

**Ruta de migración (piloto → producción):**

| Fase | Cambios | Costo estimado |
|------|---------|----------------|
| Fase 0 — Piloto | Una instancia EC2 t3.medium, todo en contenedores Docker | ~USD 25/mes |
| Fase 1 — Separación de BD | Migrar PostgreSQL a RDS Single-AZ, Kafka a MSK Serverless | ~USD 50/mes |
| Fase 2 — Alta disponibilidad | ALB + Auto Scaling Group (2 instancias) + RDS Multi-AZ | ~USD 120/mes |
| Fase 3 — Producción completa | 2 AZ, MSK Multi-AZ, WAF, CloudFront, ElastiCache Redis | ~USD 250/mes |

### 2.3 Segmentación: VLAN, DMZ y Subnetting (equivalentes cloud)

#### Plan de subnetting (VLSM sobre la VPC)

| Subred | CIDR | Rango de hosts utilizables | Uso | Equivalente funcional |
|--------|------|----------------------------|-----|-----------------------|
| Pública (DMZ) | 10.0.1.0/24 | 10.0.1.1 – 10.0.1.254 | Nginx proxy, Internet Gateway | VLAN de acceso público / DMZ |
| Aplicación | 10.0.2.0/24 | 10.0.2.1 – 10.0.2.254 | Backend Spring Boot, Notificaciones Service | VLAN de aplicación |
| Datos | 10.0.3.0/24 | 10.0.3.1 – 10.0.3.254 | PostgreSQL, Kafka | VLAN de datos / almacenamiento |
| Gestión | 10.0.4.0/24 | 10.0.4.1 – 10.0.4.254 | Bastion Host / AWS Systems Manager | VLAN de administración (OOB) |
| Reservada 1 | 10.0.5.0/24 | — | Crecimiento futuro (expansión capa aplicación) | — |
| Reservada 2 | 10.0.6.0/24 | — | Crecimiento futuro (segunda zona de disponibilidad) | — |
| Reservadas adicionales | 10.0.7.0/24 – 10.0.255.0/24 | — | Expansión a largo plazo | — |

#### Grupos de Seguridad (equivalente funcional a VLANs)

| Grupo de Seguridad | Puertos permitidos | Origen permitido | Protocolo | Descripción |
|--------------------|--------------------|-----------------|-----------|-------------|
| SG-Web | 80 (HTTP), 443 (HTTPS) | 0.0.0.0/0 (Internet) | TCP | Único grupo expuesto a internet. Nginx recibe tráfico y redirige internamente. |
| SG-App | 8084 (Backend API) | SG-Web | TCP | Solo Nginx puede acceder al backend. |
| SG-App | 8086 (Notificaciones WS) | SG-Web | TCP | Nginx puede redirigir WebSockets. |
| SG-App | 9092 (Kafka producer) | SG-App (self) | TCP | Comunicación interna entre backend y Kafka. |
| SG-Data | 5432 (PostgreSQL) | SG-App | TCP | Solo servicios de aplicación acceden a BD. |
| SG-Data | 9092 (Kafka broker) | SG-App | TCP | Solo capa de aplicación produce/consume de Kafka. |
| SG-Mgmt | 22 (SSH) | IP del administrador | TCP | Acceso SSH restringido. |
| SG-Mgmt | 443 (SSM) | AWS SSM | TCP | Acceso alternativo vía Systems Manager sin exponer SSH. |

#### DMZ (Zona Desmilitarizada)

Características de la subred pública (DMZ):

1. **Único punto de entrada:** Todo tráfico pasa por IGW solo a Nginx.
2. **Proxy inverso como barrera:** Nginx termina TLS, valida cabeceras HTTP, protege contra ataques comunes.
3. **Sin NAT Gateway:** Instancia en subred pública; ahorro ~USD 32/mes en piloto.
4. **VPC Gateway Endpoint para S3:** Acceso directo sin internet, sin costo.

#### Reglas de firewall (Network ACLs)

| Subred | Reglas de entrada | Reglas de salida |
|--------|-------------------|------------------|
| Pública (DMZ) | Permitir HTTP/HTTPS desde 0.0.0.0/0; denegar el resto | Permitir todo tráfico hacia subredes internas |
| Aplicación | Permitir tráfico desde 10.0.1.0/24 (DMZ) y 10.0.3.0/24 (datos); denegar el resto | Permitir tráfico hacia 10.0.3.0/24 (datos) y 0.0.0.0/0 para actualizaciones |
| Datos | Permitir tráfico solo desde 10.0.2.0/24 (aplicación); denegar el resto | Permitir tráfico solo hacia 10.0.2.0/24 |
| Gestión | Permitir SSH solo desde IP del administrador; denegar el resto | Permitir solo tráfico de salida esencial |

---

## 3. Incorporación de Redundancia

### 3.1 Alta disponibilidad de bajo costo (piloto)

Mecanismos HA sin costos elevados:

1. **Recuperación automática de instancia EC2:** Alarma CloudWatch en StatusCheckFailed; acción "Recover this instance". RTO ~15–20 min.
2. **Snapshots automáticos del volumen EBS:** DLM con snapshot diario y retención de 7 días.
3. **Durabilidad de Amazon S3 para documentos:** S3 Standard con 11 nueves de durabilidad; versioning y soft delete opcionales.
4. **IP elástica como failover rápido:** Reasignable en segundos a nueva instancia; misma URL para usuarios.
5. **Monitoreo proactivo con health checks:**
   - Nivel 1: CloudWatch StatusCheck (instancia EC2) → 60 s → Recuperación automática
   - Nivel 2: Endpoint /api/auth/health (Spring Boot Actuator) → 30 s → Notificación push

### 3.2 Enlaces redundantes

#### Redundancia física AWS (gestionada)

- Múltiples enlaces físicos entre racks dentro de AZ
- Múltiples enlaces de fibra hacia backbone regional
- Internet Gateway con HA nativa
- VPC Gateway Endpoint (S3) sin SPOF
- Energía y refrigeración con redundancia N+1 o 2N

#### Redundancia lógica (gestionada por el estudiante)

1. **Balanceo de carga en capa de aplicación (Nginx upstream):** Preparado para escalar horizontalmente.
2. **Redundancia de DNS:** Registro A con TTL bajo (60 s).
3. **Cola de mensajes Kafka como buffer de resiliencia:** Retención configurable (~7 días); mensajes se entregan al reconectar.
4. **Persistencia de datos en PostgreSQL con WAL:** WAL registra cada transacción antes de aplicarla; recuperación automática al reiniciar.

### 3.3 Análisis de puntos únicos de falla (SPOF)

| Componente | Riesgo si falla | Impacto | Mitigación actual (piloto) | Mitigación futura (producción) |
|------------|-----------------|---------|-----------------------------|--------------------------------|
| Instancia EC2 única | 100% del sistema inaccesible | Crítico | Recuperación automática + snapshot diario | Auto Scaling Group con 2+ instancias en subredes privadas, ALB |
| Volumen EBS gp3 | Pérdida de datos de BD y Kafka | Crítico | Snapshots automáticos DLM (7 días) | RDS Multi-AZ, MSK Multi-AZ |
| Servicio de notificaciones | Usuarios no reciben actualizaciones en tiempo real | Alto | Kafka retiene mensajes hasta reconectar; no pérdida de datos | 2+ instancias con sticky sessions en Nginx/ALB |
| Apache Kafka (single broker) | Topic de notificaciones no disponible | Alto | Operaciones transaccionales del backend funcionan sin Kafka (degradación controlada) | MSK con 3+ brokers; cluster multi-nodo |
| Nginx (proxy inverso) | Frontend y API inaccesibles desde internet | Crítico | Recuperación automática + IP elástica; frontend servible temporalmente desde S3+CloudFront | Nginx en 2+ instancias detrás de ALB, o ALB directamente con TLS gestionado |
| Certificado TLS (Let's Encrypt) | HTTPS no funciona; advertencia de seguridad | Alto | Renovación automática cada 60 días (certbot + cron); alerta por email | ACM (AWS Certificate Manager) con renovación automática asociado al ALB |
| Conexión a internet | Sin acceso a servicios externos (Gmail SMTP, Twilio, S3) | Medio | Funciones de negocio locales siguen funcionando; degradación controlada | NAT Gateway en HA; Direct Connect si necesario |
| BD PostgreSQL (single node) | Toda escritura/lectura falla | Crítico | Recuperación automática + WAL replay; snapshot diario | RDS Multi-AZ con réplica en espera síncrona (failover ~60–120 s automático) |

---

## 4. Cumplimiento de Estándares

### 4.1 Estándares de cableado estructurado — TIA/EIA

| Estándar | Ámbito | Aplicación en Finanzas |
|----------|--------|-----------------------|
| TIA/EIA-568.0 | Cableado genérico | **AWS responsabilidad:** Centros de datos con cableado estructurado certificado (CAT6A/OM4/OS2), SOC 1/2/3, ISO 27001. |
| TIA/EIA-569 | Espacios de telecomunicaciones | **AWS responsabilidad:** Cuartos de telecomunicaciones dedicados, bandejas portacables, separación cobre/fibra, entradas redundantes, hot/cold aisle containment TIA-942. |
| TIA/EIA-606-B | Administración de infraestructura | **AWS responsabilidad:** DCIM con etiquetado estandarizado, auditoría SOC 2. **Responsabilidad estudiante:** Etiquetar recursos virtuales (VPC-FINANZAS, SG-WEB-FIN, SUBNET-PUB-DMZ, EC2-BACKEND-FIN). |
| TIA/EIA-942 | Infraestructura de centro de datos | **AWS responsabilidad:** Tier III+ con redundancia N+1 energía/refrigeración, generadores de respaldo 72+h, múltiples proveedores telecom. |

### 4.2 Estándares de redes — IEEE 802.x

| Estándar | Ámbito | Aplicación en Finanzas |
|----------|--------|-----------------------|
| IEEE 802.1Q | VLANs | **Equivalente cloud:** VPC + Security Groups; aislamiento lógico de 4 capas; tráfico entre subredes solo por rutas explícitas. |
| IEEE 802.3 | Ethernet | **AWS responsabilidad:** ENI (Elastic Network Interface) con soporte hasta 100 GbE; para t3.medium: hasta 5 Gbps en ráfaga (suficiente para &lt;10 Mbps estimado). |
| IEEE 802.11 | Wi-Fi | **Aplicación:** Sucursales usan Wi-Fi según disponibilidad local; se recomienda WPA3 como mínimo. |
| IEEE 802.1X | Autenticación basada en puerto | **Equivalente cloud:** IAM + Security Groups; solo entidades autenticadas/autorizadas se comunican; principio de mínimo privilegio. |
| IEEE 802.1D / MSTP | Spanning Tree | **No aplica:** VPC de AWS sin loops por diseño; enrutamiento estático definido. |

### 4.3 Estándares de gestión de seguridad — ISO/IEC

| Estándar | Ámbito | Aplicación en Finanzas |
|----------|--------|-----------------------|
| ISO/IEC 27001:2022 | SGSI | **AWS certificado:** ISO 27001 para servicios. **Finanzas:** Controles Anexo A aplicables: A.9 (Control de acceso: JWT RSA-256, roles/permisos, sesiones concurrentes), A.12 (Seguridad operativa: auditoría, dependencias verificadas), A.16 (Gestión de incidentes: bloqueo de IP, detección de intentos fallidos), A.10 (Criptografía: TLS 1.2+, EBS/S3 cifrados). |
| ISO/IEC 27002:2022 | Buenas prácticas controles | Controles implementados: 5.15 (Control de acceso: RBAC + SoD), 5.17 (Autenticación: JWT + contexto sucursal), 8.9 (Gestión de configuración: Flyway versionado), 8.15 (Registro y supervisión: auditoría de todas las escrituras). |
| ISO/IEC 27017:2015 | Cloud | Modelo de responsabilidad compartida AWS; estudiante gestiona seguridad en la nube (VPC, SGs, IAM, cifrado). |
| ISO/IEC 27018:2019 | Protección de datos en cloud | Cifrado en reposo; acceso por roles; tabla de auditoría registra quién accedió y modificó cada registro. |

### 4.4 Estándares de centros de datos — Uptime Institute

| Clasificación | Descripción | Equivalente en Finanzas |
|---------------|-------------|------------------------|
| Tier III (N+1) | Redundancia concurrente; mantenimiento sin interrupción | Aplicado por AWS en sus centros de datos; diseño preparado para alcanzar equivalente Tier III en Fase 2. |
| Tier IV (2N+1) | Tolerante a fallos; cualquier fallo único no afecta operación | No aplica en piloto por costo; documentado como mejora a largo plazo (Fase 3 multi-región). |

### 4.5 Marco de buenas prácticas — AWS Well-Architected Framework

| Pilar | Aplicación en Finanzas |
|-------|------------------------|
| Excelencia operativa | Infraestructura definida como código mediante documentación detallada; migraciones Flyway versionadas; alertas CloudWatch. |
| Seguridad | 4 subredes con Security Groups por capa; JWT RSA-256; cifrado en tránsito y reposo; auditoría de escrituras; mínimo privilegio. |
| Confiabilidad | Recuperación automática EC2; snapshots EBS diarios; IP elástica; Kafka como buffer; ruta Multi-AZ documentada. |
| Eficiencia de rendimiento | Instancia dimensionada para &lt;10 Mbps y 28 usuarios; Nginx con compresión gzip y cache de Angular estático. |
| Optimización de costos | Se elimina NAT Gateway (~USD 32/mes ahorrado), ALB (~USD 20/mes ahorrado), RDS (~USD 15–30/mes ahorrado); VPC Gateway Endpoint para S3 sin costo; costo piloto ~USD 29/mes. |
| Sostenibilidad | Una sola instancia EC2 t3.medium consume menos recursos que múltiples instancias subutilizadas; EBS gp3 energéticamente eficiente; uso de infraestructura compartida AWS (mayor eficiencia que datacenter on-prem). |

### 4.6 Cumplimiento normativo local (Perú)

| Normativa | Ámbito | Aplicación en Finanzas |
|----------|--------|------------------------|
| Ley N.° 29733 | Protección de datos personales | Cifrado en tránsito y reposo; control de acceso granular por roles; registro de auditoría para trazabilidad; notificación al titular en caso de violación (procedimiento documentado). |
| Ley N.° 27269 | Firmas y certificados digitales | Diseño de red soporta comunicación segura con SUNAT (OSE/OPE) mediante mTLS (no implementado actualmente pero habilitado por la arquitectura). |

---

## Anexos

### Anexo A: Diagramas detallados

- A.1 Diagrama de topología lógica
- A.2 Diagrama de topología física
- A.3 Diagrama de segmentación

### Anexo B: Lista completa de requerimientos (trazabilidad al esquema de datos)

### Anexo C: Consideraciones éticas (ACM)

### Anexo D: Estimación de costos mensuales en AWS

---

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE03%20Infraestructura%20Tecnol%C3%B3gica/Entregable-1%20Dise%C3%B1o%20de%20Red.pdf)

