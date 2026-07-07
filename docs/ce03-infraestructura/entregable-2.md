
# Entregable 2: Planificación de Seguridad

**Competencia:** CE0321 — Seguridad · Área de Infraestructura Tecnológica

---

## Resumen Ejecutivo

Finanzas es un sistema de gestión financiera que procesa operaciones sensibles (ventas, compras, caja, cuentas corrientes, pago de personal, liquidaciones) para múltiples sucursales. La seguridad de la información no es opcional: una brecha podría causar pérdidas económicas directas, fuga de datos personales protegidos por la Ley N.° 29733 y daño reputacional irreversible.

El sistema ya cuenta con una base de seguridad:
- Autenticación JWT con firma asimétrica RSA-256 (pares de llaves PEM)
- Codificación de contraseñas con BCrypt
- Control de sesiones concurrentes (máx. 3 por usuario)
- Bloqueo temporal de IP tras 5 intentos fallidos (15 min)
- Detección de IP sospechosa por cambio en ventana de 1 hora
- Registro de auditoría granular en tabla lab_auditoria para todas las escrituras
- Control de acceso por roles y permisos con @PreAuthorize
- Monitoreo de intrusiones con tabla intrusion_attempts
- Guards de ruta en frontend Angular e interceptores HTTP con token JWT

Este plan de seguridad se construye sobre la base de la topología definida en el Entregable 1 (VPC segmentada en 4 subredes, Security Groups por capa, Nginx como único punto de entrada) y agrega controles técnicos y organizativos alineados con ISO/IEC 27001, Ley 29733 y OWASP Top 10.

---

## 1. Identificación de Activos Críticos

### 1.1 Lista y clasificación de activos

| ID | Activo | Tipo | Descripción | Criticidad | Dependencias |
|----|--------|------|-------------|------------|--------------|
| A-01 | Base de datos PostgreSQL (laboratorio_db) | Información | Almacena todas las transacciones financieras: ventas, compras, caja, cuentas corrientes, pagos, kardex, auditoría. ~17 tablas migradas + ~60 entidades JPA. | Crítico | A-04 (Backend), A-11 (Volumen EBS) |
| A-02 | Esquema y migraciones Flyway (V1–V11) | Información | Definición del esquema de BD, integridad referencial, triggers y datos semilla. | Crítico | A-01 |
| A-03 | Llaves criptográficas RSA (pares PEM) | Información | Llave privada y pública para firma y verificación de tokens JWT. Almacenadas en keys/private_key.pem y keys/public_key.pem. | Crítico | A-05 (JWT) |
| A-04 | Backend Spring Boot (código fuente) | Software | 494 archivos Java, 64 controladores REST, 108 servicios de negocio, lógica transaccional financiera. | Crítico | A-12 (Repositorio Git) |
| A-05 | Servicio de autenticación JWT (AuthService + JwtService) | Software | Módulo de login, registro, refresh token, generación y validación de tokens JWT con RSA-256. | Crítico | A-03, A-04, A-01 |
| A-06 | Módulo de seguridad perimetral (SecurityMonitor) | Software | Bloqueo de IP, detección de intrusiones, control de sesiones concurrentes, registro en intrusion_attempts. | Alto | A-04, A-01 |
| A-07 | Registro de auditoría (AuditLog / lab_auditoria) | Información | Trazabilidad completa de todas las operaciones de escritura: usuario, IP, acción, datos antes/después, timestamp. | Alto | A-04, A-01 |
| A-08 | Microservicio de notificaciones (Spring Boot 3.5.9) | Software | Consume eventos Kafka y distribuye vía WebSocket STOMP a clientes frontend. | Medio | A-09 |
| A-09 | Apache Kafka (topic sistema.notificaciones) | Infraestructura | Broker de mensajería asincrónica para flujo de notificaciones en tiempo real. | Medio | A-04, A-08 |
| A-10 | Frontend Angular 20 (48 módulos funcionales) | Software | Interfaz de usuario con dashboards, reportes, CRUDs, cliente WebSocket. Compilado como SPA estático. | Alto | A-21 (Nginx) |
| A-11 | Volumen EBS gp3 (30 GB) | Hardware virtual | Almacenamiento persistente de PostgreSQL, logs Kafka y archivos de aplicación. Respaldado por snapshots diarios. | Crítico | A-04, A-01, A-09 |
| A-12 | Repositorio Git del proyecto | Información | Código fuente completo de backend, frontend y notificaciones, incluyendo configuración, migraciones y llaves. | Alto | Todos |
| A-13 | Credenciales de base de datos (PostgreSQL) | Información | Usuario y contraseña de conexión a BD. Actualmente en application.properties. | Crítico | A-01 |
| A-14 | Credenciales de Gmail SMTP | Información | Usuario y contraseña para envío de correos electrónicos. Actualmente en application.properties. | Alto | A-04 |
| A-15 | Credenciales de Twilio (API) | Información | Account SID y Auth Token para integración WhatsApp/SMS. Actualmente en application.properties. | Medio | A-04 |
| A-16 | Datos personales de usuarios y empleados | Información | Nombres, DNI, direcciones, roles, asignación de sucursales. Protegidos por Ley 29733. | Crítico | A-01, A-05 |
| A-17 | Datos financieros transaccionales | Información | Ventas, compras, movimientos de caja, cuentas corrientes, pagos, liquidaciones, kardex. | Crítico | A-04 |
| A-18 | Documentos y archivos subidos (S3 / uploads) | Información | Comprobantes de pago (PDF), reportes exportados (PDF/Excel), evidencias. | Medio | A-04 |
| A-19 | Certificados TLS (Let's Encrypt) | Información | Certificados para HTTPS (Nginx). Renovación automática cada 60 días. | Alto | A-21 |
| A-20 | Configuración de la aplicación (application.properties) | Información | Configuración de BD, Kafka, JWT, seguridad, mail, Twilio, multipart, logging. Contiene credenciales en texto plano. | Crítico | A-04 |
| A-21 | Nginx (proxy inverso + TLS) | Software | Único punto de entrada desde internet. Terminación SSL, proxy reverso a backend, servidor de archivos estáticos. | Crítico | A-04, A-10, A-19 |
| A-22 | Instancia EC2 t3.medium | Hardware virtual | Servidor que aloja todos los contenedores Docker (Nginx, Backend, Notificaciones, PostgreSQL, Kafka). | Crítico | Todos los activos de infraestructura |
| A-23 | VPC y configuración de red (subredes, SGs, NACLs) | Infraestructura | Segmentación de red: 4 subredes, 4 Security Groups, tablas de rutas, VPC Gateway Endpoint. | Crítico | A-22 |
| A-24 | Administradores del sistema | Personas | Usuarios con privilegios de administración: gestión de usuarios, roles, permisos, configuración global. | Crítico | A-05 |
| A-25 | Usuarios operativos por sucursal | Personas | Personal que opera en una sucursal/punto de venta específico: registra ventas, compras, caja, kardex, atención al cliente. | Alto | A-05 |
| A-26 | Arquitectura y documentación del sistema | Información | Documentación técnica del proyecto, diagramas, diseño de red, plan de seguridad. | Medio | Todos |

### 1.2 Matriz de clasificación por tipo de activo

| Tipo de activo | Cantidad | Activos IDs |
|----------------|----------|-------------|
| Información (datos, documentos, configuración, llaves) | 13 | A-01, A-02, A-03, A-07, A-12, A-13, A-14, A-15, A-16, A-17, A-18, A-19, A-20, A-26 |
| Software (aplicaciones, módulos, servicios) | 7 | A-04, A-05, A-06, A-08, A-10, A-21 |
| Infraestructura (servidores, redes, almacenamiento) | 4 | A-09, A-11, A-22, A-23 |
| Personas (roles, usuarios, administradores) | 2 | A-24, A-25 |

### 1.3 Distribución de criticidad

| Nivel | Cantidad | Porcentaje |
|-------|----------|------------|
| Crítico | 14 | 54% |
| Alto | 8 | 31% |
| Medio | 4 | 15% |
| Bajo | 0 | 0% |
| **Total** | **26** | **100%** |

---

## 2. Análisis de Riesgos

### 2.1 Metodología

El análisis de riesgos sigue las directrices de:
- **ISO/IEC 27005:2022 (Gestión de Riesgos de Seguridad de la Información)**
- **NIST SP 800-30 (Guide for Conducting Risk Assessments)**

**Escala de valoración:**
- Probabilidad: 1 (Bajo, raro) → 4 (Muy Alto, semanal/casi seguro)
- Impacto: 1 (Bajo, pérdida < USD 100) → 4 (Muy Alto, pérdida > USD 10,000, daño legal, detención total del negocio)
- Nivel de riesgo = Probabilidad × Impacto

**Acción según nivel de riesgo:**
- 12–16: Crítico → Mitigación inmediata (prioridad 1)
- 8–9: Alto → Mitigación planificada (prioridad 2)
- 4–6: Medio → Monitoreo y mejora continua
- 1–3: Bajo → Aceptar o monitorear

### 2.2 Matriz de riesgos

| ID | Amenaza | Vulnerabilidad | Activo afectado | Prob. | Imp. | Nivel de riesgo | Control actual | Control propuesto |
|----|---------|----------------|-----------------|-------|-----|----------------|----------------|------------------|
| R-01 | Filtración de llaves RSA privadas | Llaves JWT almacenadas en archivos PEM dentro del classpath del backend, accesibles desde el repositorio Git, sin cifrado adicional. | A-03 (Llaves RSA), A-05 (JWT) | 2 | 4 | 8 (Alto) | Ninguno (las llaves están en texto plano dentro del proyecto) | Migrar llaves a AWS Secrets Manager o HSM (CloudHSM). Rotación automática cada 90 días. Eliminar llaves del repositorio Git y del classpath. |
| R-02 | Exposición de credenciales en texto plano | application.properties contiene usuario/contraseña de PostgreSQL, credenciales de Gmail SMTP y tokens de Twilio en texto plano sin cifrar. | A-13, A-14, A-15, A-20 | 3 | 4 | 12 (Crítico) | Ninguno (credenciales hardcodeadas) | Migrar todas las credenciales a variables de entorno o AWS Secrets Manager. Implementar policies de IAM para acceso a secretos. Rotación periódica. |
| R-03 | Ataque de fuerza bruta al login | El endpoint /api/auth/login no tiene rate limiting configurado. Aunque existe bloqueo por IP tras 5 intentos, un ataque distribuido (DDoS de autenticación) podría eludirlo. | A-05 (JWT), A-24 (Admin), A-25 (Usuarios) | 3 | 3 | 9 (Alto) | Bloqueo de IP tras 5 intentos fallidos (15 min). Control de sesiones concurrentes (máx. 3). | Implementar rate limiting en Nginx (límite por IP, por usuario, por endpoint). Agregar Cloudflare o AWS WAF con reglas de rate-based blocking. |
| R-04 | Inyección SQL | El sistema usa JPA/Hibernate que previene inyección en consultas parametrizadas, pero las consultas nativas y procedimientos almacenados no están auditados. | A-01 (PostgreSQL) | 2 | 4 | 8 (Alto) | JPA/Hibernate con parámetros tipados | Auditoría de todas las consultas nativas (JPQL nativo, @Query con SQL puro). Implementar WAF con reglas de SQL injection. Pruebas de penetración trimestrales. |
| R-05 | Ataque XSS (Cross-Site Scripting) | El frontend Angular tiene protección integrada contra XSS (sanitización automática de templates), pero los reportes exportados en PDF/HTML podrían inyectar scripts si los datos no se sanitizan al generarlos. | A-10 (Frontend), A-18 (Documentos) | 2 | 3 | 6 (Medio) | Sanitización automática de Angular en templates | Implementar Content Security Policy (CSP) estricta en Nginx. Sanitizar datos de entrada antes de generar PDFs/Excel. Validar datos en backend antes de devolverlos. |
| R-06 | Ataque de denegación de servicio (DoS/DDoS) | La instancia EC2 t3.medium tiene un solo punto de entrada (Nginx) sin protección contra ataques volumétricos ni escalado automático. | A-21 (Nginx), A-22 (EC2) | 3 | 3 | 9 (Alto) | Ninguno (la instancia está directamente expuesta a internet con protección mínima) | Implementar AWS WAF + AWS Shield Standard (gratuito). Configurar rate limiting en Nginx. Usar CloudFront como CDN con protección DDoS. |
| R-07 | Acceso no autorizado a Kafka | Kafka se ejecuta sin autenticación ni cifrado (puerto 9092 abierto dentro de la VPC). Cualquier servicio en la misma red podría producir/consumir mensajes del topic sistema.notificaciones. | A-09 (Kafka) | 2 | 3 | 6 (Medio) | Aislamiento por Security Group (solo SG-App accede al puerto 9092) | Habilitar SSL/TLS en Kafka para cifrado en tránsito. Configurar SASL/SCRAM para autenticación. Migrar a Amazon MSK con cifrado y autenticación gestionados. |
| R-08 | Intercepción de tráfico interno VPC | El tráfico entre servicios (backend ↔ PostgreSQL, backend ↔ Kafka) viaja en texto plano dentro de la VPC. Aunque la VPC está aislada, un atacante con acceso a la instancia podría capturar tráfico. | A-01, A-04, A-08, A-09 | 2 | 3 | 6 (Medio) | Aislamiento de VPC | Habilitar cifrado SSL/TLS en conexiones PostgreSQL (ssl=true). Cifrar comunicación con Kafka. Considerar VPC Traffic Mirroring para monitoreo. |
| R-09 | Fuga de datos por error en el registro de auditoría | La tabla lab_auditoria registra datos sensibles (oldData, newData) sin cifrado. Si un atacante accede a la BD, puede leer el historial completo de cambios de datos financieros. | A-07 (AuditLog), A-01 | 2 | 4 | 8 (Alto) | Control de acceso a la BD por Security Group | Cifrar columnas sensibles en lab_auditoria (cifrado a nivel de aplicación o PostgreSQL pgcrypto). Implementar política de retención y purge automático de auditoría > 1 año. |
| R-10 | Suplantación de identidad (token theft) | El token JWT se almacena en localStorage del navegador. Un ataque XSS (aunque mitigado) o un script malicioso de extensión podría robar el token y suplantar al usuario. | A-05 (JWT), A-24, A-25 | 3 | 4 | 12 (Crítico) | Token con expiración de 30 minutos. Refresh token con rotación. | Migrar almacenamiento de token de localStorage a HttpOnly cookies (con Secure y SameSite=Strict). Implementar boundary tokens (vincular token a características de la sesión: IP, User-Agent). |
| R-11 | Acceso root a la instancia EC2 por SSH expuesto | El puerto SSH (22) está abierto a internet (SG-Mgmt permite acceso desde IP del administrador). Si la IP del administrador se ve comprometida o cambia, la instancia queda accesible. | A-22 (EC2) | 2 | 4 | 8 (Alto) | Security Group restringe SSH a IP del administrador | Deshabilitar SSH por internet completamente. Usar solo AWS Systems Manager (SSM) Session Manager para acceso administrativo (sin puertos abiertos, sin bastion host, con auditoría integrada). |
| R-12 | Corrupción de base de datos por migración fallida | Las migraciones Flyway se ejecutan automáticamente al iniciar el backend. Una migración fallida o un script mal escrito podría corromper datos existentes. | A-01, A-02 | 2 | 4 | 8 (Alto) | Flyway con out-of-order=false. Snapshots diarios EBS. | Implementar entorno de staging para validar migraciones antes de producción. Agregar checksums de integridad a las migraciones. Probar rollback de migraciones (Flyway undo no es nativo, requiere scripts manuales). |
| R-13 | Violación de datos personales (Ley 29733) | La base de datos almacena datos personales (nombres, DNI, direcciones) sin cifrado en reposo a nivel de columna. Un acceso no autorizado a la BD expone estos datos. | A-16 (Datos personales) | 2 | 4 | 8 (Alto) | Cifrado en reposo del volumen EBS. Control de acceso por roles. | Implementar cifrado a nivel de columna en PostgreSQL (pgcrypto) para DNI, direcciones y datos sensibles. Enmascarar datos personales en logs y auditoría. |
| R-14 | Suplantación de sesión por fijación de sesión (session fixation) | El sistema no renueva el token JWT después de operaciones sensibles (cambio de sucursal, cambio de punto de venta). Un atacante podría fijar un token antes del cambio de contexto. | A-05 (JWT), A-24, A-25 | 1 | 3 | 3 (Bajo) | El token incluye idSucursal y idPunto como claims. | Renovar el token JWT completo después de cada cambio de sucursal o punto de venta (ya implementado en AuthSucursalService pero debe verificarse). |
| R-15 | Falta de segregación de funciones en operaciones financieras | Un mismo usuario podría tener permisos para crear y aprobar una misma operación (ej. crear un descuento y autorizarlo) si tiene los roles combinados. Esto viola el principio de SoD. | A-04 (Backend), A-24, A-25 | 2 | 3 | 6 (Medio) | Control de permisos por rol (@PreAuthorize) | Implementar matriz SOD (Segregation of Duties): definir reglas de conflicto (ej. "quien crea no puede aprobar"). Validar SOD antes de asignar roles a un usuario. Registrar violaciones SOD en auditoría. |

### 2.3 Resumen de riesgos por nivel

| Nivel de riesgo | Cantidad | ID de riesgos |
|------------------|----------|---------------|
| Crítico (12–16) | 2 | R-02, R-10 |
| Alto (8–9) | 8 | R-01, R-03, R-04, R-06, R-09, R-11, R-12, R-13 |
| Medio (4–6) | 4 | R-05, R-07, R-08, R-15 |
| Bajo (1–3) | 1 | R-14 |
| **Total** | **15** | |

### 2.4 Mapa de calor de riesgos

| Probabilidad \\ Impacto | Bajo (1) | Medio (2) | Alto (3) | Muy Alto (4) |
|-------------------------|----------|-----------|---------|-------------|
| Muy Alto (4) | — | — | — | — |
| Alto (3) | — | — | R-03, R-06 | R-02, R-10 |
| Medio (2) | — | — | R-05, R-07, R-08, R-15 | R-01, R-04, R-09, R-11, R-12, R-13 |
| Bajo (1) | — | — | R-14 | — |

### 2.5 Plan de tratamiento de riesgos

| Prioridad | ID | Medida | Tipo de tratamiento | Responsable | Plazo | Costo estimado |
|-----------|----|--------|---------------------|-------------|-------|----------------|
| 1 | R-02 | Migrar credenciales a Secrets Manager + variables de entorno | Reducción | Administrador del sistema | 1 semana | USD 0 – 4/mes (Secrets Manager) |
| 1 | R-10 | Migrar token JWT de localStorage a HttpOnly cookie | Reducción | Desarrollador backend/frontend | 2 semanas | USD 0 |
| 2 | R-03 | Rate limiting en Nginx + AWS WAF | Reducción | Administrador de infraestructura | 1 semana | USD 0 (Nginx) – 20/mes (WAF) |
| 2 | R-06 | AWS Shield + CloudFront + rate limiting | Reducción | Administrador de infraestructura | 2 semanas | USD 0 (Shield Std) – 30/mes (WAF) |
| 2 | R-01 | Migrar llaves RSA a Secrets Manager + rotación automática | Reducción | Administrador del sistema | 1 semana | USD 0 – 4/mes |
| 2 | R-11 | Deshabilitar SSH público, migrar a SSM Session Manager | Reducción | Administrador de infraestructura | 2 días | USD 0 |
| 3 | R-04 | Auditoría de consultas nativas + WAF SQL injection | Reducción | Desarrollador backend | 1 mes | USD 0 (auditoría) – 20/mes (WAF) |
| 3 | R-09 | Cifrar columnas sensibles en auditoría + política de retención | Reducción | Desarrollador backend | 2 semanas | USD 0 |
| 3 | R-12 | Entorno staging + rollback de migraciones | Reducción | Desarrollador backend | 1 mes | USD 0 (staging en misma cuenta) |
| 3 | R-13 | Cifrado a nivel de columna (pgcrypto) + enmascaramiento | Reducción | Desarrollador backend | 3 semanas | USD 0 |
| 4 | R-07 | SSL/TLS + SASL en Kafka | Reducción | Administrador de infraestructura | 2 semanas | USD 0 (configuración) |
| 4 | R-08 | SSL en PostgreSQL + cifrado en comunicación Kafka | Reducción | Administrador de infraestructura | 1 semana | USD 0 |
| 4 | R-15 | Implementar matriz SOD en asignación de roles | Reducción | Desarrollador backend | 3 semanas | USD 0 |
| 5 | R-05 | CSP estricta + sanitización en generación de reportes | Reducción | Desarrollador frontend | 2 semanas | USD 0 |
| 5 | R-14 | Verificar renovación de token en cambio de sucursal | Reducción | Desarrollador backend | 3 días | USD 0 |

---

## 3. Políticas de Seguridad

### 3.1 Política de Control de Acceso

| ID | Política | Descripción |
|----|----------|-------------|
| P-AC-01 | Principio de mínimo privilegio | Todo usuario del sistema recibirá únicamente los permisos estrictamente necesarios para cumplir sus funciones laborales. Ningún usuario podrá tener permisos que excedan su rol definido. |
| P-AC-02 | Autenticación multifactor (MFA) | El acceso a la consola de AWS y al panel de administración del sistema requerirá autenticación multifactor. Para usuarios operativos, se considera MFA basado en conocimiento (contraseña + token JWT con contexto de sucursal). |
| P-AC-03 | Sesiones concurrentes | Se mantiene el límite de máximo 3 sesiones concurrentes por usuario. Al exceder este límite, la sesión más antigua será invalidada automáticamente. |
| P-AC-04 | Bloqueo por intentos fallidos | Tras 5 intentos fallidos de inicio de sesión en un período de 15 minutos, la IP origen será bloqueada temporalmente por 15 minutos. Tras 10 intentos fallidos acumulados en 1 hora, el usuario será bloqueado hasta que un administrador lo desbloquee manualmente. |
| P-AC-05 | Caducidad de contraseñas | Las contraseñas de los usuarios expirarán cada 90 días. El sistema obligará al cambio de contraseña al primer inicio de sesión y después de cada restablecimiento de contraseña por parte del administrador. |
| P-AC-06 | Complejidad de contraseñas | Las contraseñas deberán tener un mínimo de 8 caracteres, incluyendo al menos una letra mayúscula, una letra minúscula, un dígito y un carácter especial. Se verificará contra una lista de contraseñas comunes (rockyou top 1000) para evitar su uso. |
| P-AC-07 | Cierre de sesión por inactividad | Las sesiones web expirarán automáticamente después de 30 minutos de inactividad. El usuario deberá autenticarse nuevamente para continuar. Se mostrará una advertencia 5 minutos antes de la expiración. |
| P-AC-08 | Control de acceso por sucursal | Un usuario autenticado solo podrá acceder a los datos y operaciones de la sucursal a la que está asignado. El cambio de sucursal requerirá una nueva autenticación con confirmación de contraseña (ya implementado en AuthSucursalService). |
| P-AC-09 | Registro de accesos | Todo inicio de sesión exitoso y fallido será registrado en la tabla sessions y en la auditoría (lab_auditoria), incluyendo: usuario, IP origen, User-Agent, timestamp, sucursal y resultado (éxito/fallo). |

### 3.2 Política de Seguridad de Datos

| ID | Política | Descripción |
|----|----------|-------------|
| P-DT-01 | Clasificación de datos | Todos los datos del sistema se clasificarán en cuatro niveles: Público (reportes agregados sin datos sensibles), Interno (procedimientos, documentación técnica), Confidencial (datos financieros transaccionales, datos de contacto) y Restringido (llaves criptográficas, credenciales, datos personales con DNI). Cada nivel tendrá controles de protección específicos. |
| P-DT-02 | Cifrado en reposo | Todos los datos clasificados como Confidenciales o Restringidos deberán almacenarse cifrados en reposo. Se implementará: (a) cifrado del volumen EBS (por defecto en AWS), (b) cifrado de buckets S3 (SSE-S3 o SSE-KMS), (c) cifrado a nivel de columna en PostgreSQL (pgcrypto) para DNI, direcciones y datos biométricos. |
| P-DT-03 | Cifrado en tránsito | Todas las comunicaciones entre componentes del sistema deberán viajar cifradas: (a) HTTPS (TLS 1.2+) entre cliente y servidor, (b) SSL/TLS en conexiones PostgreSQL, (c) SSL/TLS + SASL en Kafka, (d) STARTTLS en conexiones SMTP. |
| P-DT-04 | Retención de datos | Los datos transaccionales se conservarán por un mínimo de 5 años (exigencia contable y fiscal peruana). El registro de auditoría se conservará por 3 años. Los logs de aplicación se conservarán por 90 días. Los datos personales se eliminarán 1 año después de que el usuario cese su relación con el sistema, salvo obligación legal de retención. |
| P-DT-05 | Minimización de datos | Solo se recolectarán los datos estrictamente necesarios para la operación del sistema financiero. No se almacenarán datos sensibles no requeridos (ej. número de tarjeta de crédito, preguntas secretas) a menos que la legislación lo exija explícitamente. |
| P-DT-06 | Enmascaramiento de datos | Los datos personales (DNI, teléfono, correo) deberán mostrarse enmascarados en las interfaces de usuario que no requieran el valor completo para su función. Ejemplo: ***1234 para DNI, o ***@gmail.com para correo. Los datos completos solo se mostrarán en módulos autorizados explícitamente (ej. detalle de empleado para RRHH). |
| P-DT-07 | Respaldo y recuperación | Se realizarán respaldos diarios automáticos de la base de datos (snapshot EBS vía DLM) con retención de 7 días. Semanalmente se generará un respaldo adicional con retención mensual. Los respaldos se almacenarán en una región AWS secundaria (us-west-2) para protección ante desastre regional. |

### 3.3 Política de Gestión de Credenciales y Secretos

| ID | Política | Descripción |
|----|----------|-------------|
| P-CR-01 | Prohibición de secretos en código fuente | Queda terminantemente prohibido almacenar contraseñas, tokens, llaves API, certificados o cualquier secreto en el código fuente, archivos de configuración versionados o archivos dentro del classpath de la aplicación. Todos los secretos deberán gestionarse a través de AWS Secrets Manager o variables de entorno inyectadas en tiempo de ejecución. |
| P-CR-02 | Rotación periódica de secretos | Las credenciales de base de datos se rotarán cada 90 días. Los tokens de API (Twilio) se rotarán cada 180 días. Las llaves RSA de JWT se rotarán cada 90 días. Las credenciales SMTP se rotarán inmediatamente si hay indicios de compromiso. |
| P-CR-03 | Gestión de llaves criptográficas | Las llaves RSA utilizadas para firma JWT se generarán con una longitud mínima de 2048 bits. La llave privada se almacenará exclusivamente en AWS Secrets Manager con cifrado KMS gestionado por AWS. La llave pública podrá distribuirse libremente. Se mantendrá un historial de llaves anteriores para permitir la validación de tokens emitidos antes de una rotación. |
| P-CR-04 | Acceso bajo demanda a secretos | El acceso a los secretos almacenados en Secrets Manager se concederá únicamente a las instancias EC2 y servicios que los requieran, mediante políticas de IAM con permisos de mínimo privilegio. Ningún desarrollador o administrador podrá leer secretos de producción sin autorización explícita y registro en auditoría. |

### 3.4 Política de Seguridad de Red

| ID | Política | Descripción |
|----|----------|-------------|
| P-RD-01 | Segmentación por zonas de seguridad | La red se segmentará en cuatro zonas con niveles de confianza decrecientes: (1) DMZ — público, solo Nginx, (2) Aplicación — backend y microservicios, (3) Datos — bases de datos y colas de mensajería, (4) Gestión — acceso administrativo. El tráfico entre zonas será explícitamente autorizado por Security Groups y denegado por defecto. |
| P-RD-02 | Denegación por defecto | Todo el tráfico entrante y saliente está denegado por defecto. Solo se permitirán explícitamente las conexiones necesarias para la operación del sistema, documentadas en la matriz de puertos de la Sección 2.3 del CE0311. |
| P-RD-03 | Monitoreo de tráfico de red | Se habilitará VPC Flow Logs para registrar todo el tráfico de red de la VPC. Los logs se enviarán a CloudWatch Logs y se conservarán por 90 días. Se configurarán alarmas para detectar patrones anómalos: tráfico saliente a IPs sospechosas, múltiples conexiones desde una misma IP, escaneo de puertos. |
| P-RD-04 | Acceso administrativo sin SSH público | El acceso SSH directo a la instancia EC2 desde internet queda prohibido. Todo acceso administrativo se realizará exclusivamente a través de AWS Systems Manager Session Manager (SSM), que proporciona: (a) túneles cifrados sin exponer puertos, (b) registro completo de sesiones en CloudTrail, (c) control de acceso mediante políticas IAM, (d) posibilidad de restringir comandos específicos. |
| P-RD-05 | Protección contra escaneo de puertos | Se configurarán reglas en el Network ACL de la subred pública para limitar la tasa de conexiones entrantes. Se implementará un sistema de fail2ban en Nginx para bloquear IPs que realicen escaneo de puertos o múltiples intentos de conexión a puertos no expuestos. |

### 3.5 Política de Seguridad de Aplicaciones

| ID | Política | Descripción |
|----|----------|-------------|
| P-AP-01 | Validación de entrada | Toda entrada de datos proveniente del cliente (formularios, parámetros de URL, cabeceras HTTP) será validada tanto en el frontend (validación de formularios) como en el backend (validación con Jakarta Bean Validation + sanitización adicional). Se rechazarán entradas que no cumplan el formato esperado sin revelar información interna. |
| P-AP-02 | Protección contra OWASP Top 10 | El sistema implementará controles específicos para las 10 vulnerabilidades más críticas según OWASP: Inyección (JPA parametrizado), Autenticación rota (JWT + rate limiting), Exposición de datos sensibles (cifrado), XXE (deshabilitado por defecto en Jackson), Control de acceso roto (@PreAuthorize), Configuración insegura (checklist de hardening), XSS (sanitización Angular + CSP), Deserialización insegura (validación en Jackson), Componentes vulnerables (escaneo de dependencias), Logging y monitoreo insuficientes (auditoría + CloudWatch). |
| P-AP-03 | Cabeceras de seguridad HTTP | Nginx incluirá las siguientes cabeceras de seguridad en todas las respuestas HTTP: Strict-Transport-Security: max-age=31536000; includeSubDomains, X-Content-Type-Options: nosniff, X-Frame-Options: DENY, X-XSS-Protection: 1; mode=block, Content-Security-Policy: default-src 'self', Referrer-Policy: strict-origin-when-cross-origin, Permissions-Policy: geolocation=(), camera=(), microphone=(). |
| P-AP-04 | Gestión de dependencias vulnerables | Se implementará un proceso de escaneo automático de dependencias utilizando OWASP Dependency-Check integrado en el pipeline de Maven (pom.xml). Las vulnerabilidades críticas (CVSS ≥ 7.0) deberán corregirse en un plazo máximo de 7 días. Se mantendrá un inventario actualizado de todas las dependencias y sus versiones. |
| P-AP-05 | Pruebas de seguridad | Se realizarán pruebas de penetración (pentest) trimestrales sobre el backend y frontend. Se ejecutarán escaneos de vulnerabilidades automatizados semanalmente (ZAP, Nikto). Las vulnerabilidades encontradas se clasificarán por severidad y se asignarán para corrección según el SLA definido: Crítico (48 horas), Alto (7 días), Medio (30 días), Bajo (90 días). |

### 3.6 Política de Gestión de Incidentes

| ID | Política | Descripción |
|----|----------|-------------|
| P-IN-01 | Clasificación de incidentes | Los incidentes de seguridad se clasificarán en cuatro niveles: Crítico (compromiso de datos financieros, acceso no autorizado a base de datos, exposición de llaves privadas), Alto (denegación de servicio, compromiso de cuenta de administrador), Medio (intentos de intrusión, escaneo de puertos), Bajo (phishing a usuarios, spam). |
| P-IN-02 | Tiempos de respuesta | Los incidentes deberán ser reportados al administrador del sistema dentro de los siguientes plazos desde su detección: Crítico (inmediato, < 15 minutos), Alto (< 1 hora), Medio (< 24 horas), Bajo (< 72 horas). La resolución del incidente tendrá los siguientes SLA: Crítico (4 horas), Alto (24 horas), Medio (7 días), Bajo (30 días). |
| P-IN-03 | Procedimiento de respuesta | Todo incidente seguirá el ciclo PDR (Prepare, Detect, Respond, Recover): (1) preparación — runbooks documentados, contactos de emergencia, (2) detección — alertas CloudWatch, logs de auditoría, reportes de usuarios, (3) respuesta — contención, análisis forense, erradicación, (4) recuperación — restauración desde respaldo, verificación de integridad, lecciones aprendidas. |
| P-IN-04 | Notificación a autoridades | En caso de violación de datos personales, se notificará a la Autoridad Nacional de Protección de Datos Personales (Perú) dentro de las 72 horas siguientes a la confirmación del incidente, según lo establecido en la Ley N.° 29733 y su reglamento. Los titulares de los datos afectados serán notificados individualmente. |
| P-IN-05 | Evidencia forense | Ante un incidente crítico, se preservará la evidencia digital para fines forenses: (a) snapshot del volumen EBS de la instancia afectada, (b) exportación de logs de CloudWatch y VPC Flow Logs, (c) registro de sesiones de SSM, (d) dump de la tabla intrusion_attempts y sessions. Todo el manejo de evidencia seguirá la cadena de custodia documentada. |

### 3.7 Política de Seguridad Física (Cloud)

| ID | Política | Descripción |
|----|----------|-------------|
| P-FI-01 | Responsabilidad compartida AWS | Se reconoce que AWS es responsable de la seguridad física de los centros de datos (acceso a instalaciones, vigilancia, control de temperatura, energía redundante). El equipo del proyecto es responsable de la seguridad lógica de los recursos desplegados en AWS (configuración de VPC, IAM, Security Groups, cifrado). |
| P-FI-02 | Bloqueo de cuenta AWS | Se configurarán alertas de presupuesto (AWS Budgets) con umbrales al 50%, 80% y 100% del presupuesto mensual. Se habilitará AWS CloudTrail para auditoría de todas las acciones de la API de AWS. Se configurará una alarma de root activity para detectar cualquier uso de la cuenta root. |

### 3.8 Matriz de políticas vs. activos críticos

| Activo | Políticas aplicables |
|--------|-----------------------|
| A-01 (PostgreSQL) | P-DT-01, P-DT-02, P-DT-03, P-DT-04, P-DT-07, P-AP-01 |
| A-03 (Llaves RSA) | P-CR-01, P-CR-02, P-CR-03, P-CR-04 |
| A-04 (Backend Spring Boot) | P-AC-01 a P-AC-09, P-AP-01, P-AP-02, P-AP-04, P-AP-05 |
| A-05 (JWT) | P-AC-01 a P-AC-09, P-CR-03 |
| A-07 (Auditoría) | P-DT-04, P-DT-06, P-IN-01, P-IN-05 |
| A-10 (Frontend Angular) | P-AC-07, P-AP-01, P-AP-02, P-AP-05 |
| A-21 (Nginx) | P-AP-03, P-RD-05 |
| A-22 (EC2) | P-AC-02, P-RD-04, P-IN-03, P-FI-01 |
| A-23 (VPC y red) | P-RD-01, P-RD-02, P-RD-03 |

---

## 4. Roles y Responsabilidades

### 4.1 Estructura organizacional de seguridad

La administración de seguridad del sistema Finanzas se organiza en tres niveles, siguiendo el modelo de defensa en profundidad aplicado a la gestión de roles:
1. **Comité de Seguridad de la Información (CSI)** — Nivel estratégico
2. **Administrador del Sistema / CISO funcional** — Nivel táctico
3. **Administrador de Infraestructura (Cloud/Seguridad)**, **Oficial de Protección de Datos (DPO)**, **Desarrolladores Backend/Frontend**, **Usuarios Operativos**, **Auditor Externo** — Nivel operativo

### 4.2 Descripción de roles

| Rol | Responsabilidades clave | Habilidades requeridas | Reporta a |
|-----|--------------------------|------------------------|-----------|
| **Comité de Seguridad de la Información (CSI)** | - Aprobar políticas y normas de seguridad del sistema<br>- Revisar y aprobar el plan anual de seguridad<br>- Autorizar inversiones en controles de seguridad<br>- Revisar incidentes críticos y lecciones aprendidas<br>- Definir apetito y tolerancia al riesgo | Visión estratégica, conocimiento de negocio financiero, conocimiento de marco normativo (Ley 29733, ISO 27001) | Dirección General |
| **Administrador del Sistema (CISO funcional)** | - Gestionar usuarios, roles, permisos y perfiles de acceso<br>- Configurar y supervisar políticas de seguridad del backend<br>- Monitorear la tabla lab_auditoria y detectar anomalías<br>- Responder a incidentes de seguridad nivel 1 y 2<br>- Ejecutar el plan de respuesta a incidentes<br>- Gestionar rotación de llaves RSA y credenciales | Spring Security, JWT, RBAC, PostgreSQL, Flyway, conocimiento de OWASP Top 10, gestión de incidentes | CSI |
| **Administrador de Infraestructura (Cloud/Seguridad)** | - Configurar y mantener la VPC, subredes, SGs y NACLs<br>- Administrar la instancia EC2, volúmenes EBS y snapshots<br>- Monitorear CloudWatch, VPC Flow Logs y alarmas<br>- Aplicar parches de seguridad al sistema operativo y contenedores<br>- Gestionar backups y pruebas de recuperación<br>- Implementar controles de red (WAF, rate limiting, fail2ban) | AWS (VPC, EC2, S3, IAM, CloudWatch), Linux (Ubuntu), Docker, Nginx, Redes (TCP/IP, DNS, TLS), Seguridad cloud (AWS Well-Architected) | CSI |
| **Oficial de Protección de Datos (DPO)** | - Velar por el cumplimiento de la Ley N.° 29733 y su reglamento<br>- Atender reclamos y solicitudes de titulares de datos (ARCO)<br>- Reportar violaciones de datos a la Autoridad Nacional<br>- Realizar evaluaciones de impacto a la privacidad (EIP) | Ley 29733, normativa peruana de protección de datos, ISO/IEC 27701, conocimiento del sistema y sus datos | CSI / Dirección Legal |
| **Desarrollador Backend** | - Implementar controles de seguridad en el código Spring Boot<br>- Realizar code review con enfoque de seguridad<br>- Corregir vulnerabilidades identificadas en pentests<br>- Escribir migraciones Flyway seguras y auditables | Java 21, Spring Security, JPA, JWT, RSA, BCrypt, OWASP Top 10, pruebas unitarias y de integración | Administrador del Sistema |
| **Desarrollador Frontend** | - Implementar guards de ruta, interceptores y manejo seguro de tokens<br>- Sanitizar datos de entrada y salida en la interfaz<br>- Implementar CSP y cabeceras de seguridad en el frontend | Angular 20, TypeScript, RxJS, OWASP Top 10 (XSS, CSRF), WebSockets seguros | Administrador del Sistema |
| **Usuario Operativo** | - Usar el sistema de acuerdo con las políticas de seguridad establecidas<br>- No compartir credenciales con otros usuarios<br>- Reportar actividades sospechosas al administrador<br>- Cerrar sesión al finalizar su turno | Manejo del sistema Finanzas, conocimiento básico de seguridad informática | Jefe de Sucursal |
| **Auditor Externo** | - Realizar pruebas de penetración (pentest) trimestrales<br>- Verificar el cumplimiento de políticas y controles<br>- Emitir informes de hallazgos y recomendaciones | Pentest, OWASP Testing Guide, ISO 27001, SOC 2, auditoría de sistemas financieros | CSI |

### 4.3 Matriz RACI

La matriz RACI define quién es **Responsable** (ejecuta la actividad), **Responsable final** (rinde cuentas y es el dueño de la actividad), **Consultado** (debe ser consultado antes de ejecutar) y **Informado** (debe ser informado después de ejecutar) para cada tarea o control.

| Actividad / Control | CSI | Adm. Sistema | Adm. Infraestructura | DPO | Des. Backend | Des. Frontend | Usu. Operativo | Aud. Externo |
|---------------------|-----|--------------|-----------------------|-----|-------------|--------------|---------------|-------------|
| **Políticas de seguridad** | | | | | | | | |
| Aprobar políticas de seguridad | A | R | C | C | I | I | I | C |
| Redactar y mantener políticas | I | R/A | C | C | C | C | I | C |
| Difundir políticas a usuarios | I | R/A | I | C | I | I | I | I |
| Revisar políticas anualmente | A | R | C | C | C | C | I | C |
| **Control de acceso** | | | | | | | | |
| Crear y eliminar usuarios | I | R/A | I | I | I | I | I | I |
| Asignar roles y permisos | I | R/A | I | C | I | I | C | I |
| Revisar accesos trimestralmente | I | R | I | C | I | I | I | A |
| Bloquear/desbloquear IPs | I | R/A | C | I | I | I | I | I |
| **Gestión de secretos** | | | | | | | | |
| Rotar llaves RSA (JWT) | I | R/A | C | I | I | I | I | I |
| Rotar credenciales BD | I | R | A | I | I | I | I | I |
| Auditar acceso a secretos | I | R | R | I | I | I | I | A |
| **Seguridad de red** | | | | | | | | |
| Configurar VPC/SGs/NACLs | I | C | R/A | I | I | I | I | C |
| Monitorear VPC Flow Logs | I | C | R/A | I | I | I | I | I |
| Implementar WAF / rate limiting | I | C | R/A | I | I | I | I | I |
| Gestionar certificados TLS | I | I | R/A | I | I | I | I | I |
| **Seguridad de aplicaciones** | | | | | | | | |
| Implementar validación de entrada | I | C | I | I | R/A | R/A | I | C |
| Implementar cabeceras HTTP seguras | I | C | A | I | I | R | I | I |
| Escanear dependencias vulnerables | I | A | I | I | R | R | I | C |
| Corregir vulnerabilidades | I | A | I | I | R | R | I | C |
| **Monitoreo y detección** | | | | | | | | |
| Monitorear logs de auditoría | I | R/A | C | C | I | I | I | C |
| Configurar alarmas CloudWatch | I | C | R/A | I | I | I | I | I |
| Revisar intentos de intrusión | I | R/A | C | I | I | I | I | I |
| **Respuesta a incidentes** | | | | | | | | |
| Reportar incidente | I | A | A | C | R | R | R | I |
| Clasificar incidente | A | R | R | C | I | I | I | C |
| Contener y erradicar | I | R | R/A | I | C | C | I | I |
| Recuperar servicio | I | I | R/A | I | C | C | I | I |
| Notificar a autoridades (violación datos) | A | C | I | R | I | I | I | C |
| Documentar lecciones aprendidas | A | R | R | C | C | C | I | C |
| **Cumplimiento y auditoría** | | | | | | | | |
| Realizar pentest trimestral | A | C | C | C | I | I | I | R |
| Realizar EIP (privacidad) | I | C | I | R/A | I | I | I | C |
| Reportar cumplimiento normativo | A | C | I | R | I | I | I | C |
| **Capacitación** | | | | | | | | |
| Capacitar usuarios en seguridad | I | R/A | I | C | I | I | I | I |
| Capacitar desarrolladores (seguro) | I | R/A | I | I | I | I | I | I |
| Simulacro de incidentes anual | A | R | R | C | C | C | C | C |

### 4.4 Matriz de segregación de funciones (SOD)

Para evitar conflictos de interés y cumplir con el principio de SoD (Segregation of Duties), las siguientes combinaciones de roles están prohibidas en un mismo usuario:

| Combinación prohibida | Riesgo | Mitigación |
|------------------------|--------|------------|
| Administrador del Sistema + Usuario Operativo | El administrador podría otorgarse permisos para realizar y ocultar operaciones financieras fraudulentas | Separación de cuentas: el administrador usa una cuenta administrativa y una cuenta operativa distinta |
| Crear Usuario + Asignar Roles + Auditor | Podría crear un usuario con privilegios, asignarle roles y luego eliminar las trazas de auditoría | La auditoría se almacena en tabla inmutable con acceso de solo lectura para el auditor |
| Desarrollador Backend + Administrador de Infraestructura | Podría desplegar código malicioso directamente en producción sin revisión | Pipeline CI/CD con code review obligatorio; el desarrollador no tiene acceso directo a producción |
| Operador de Ventas + Autorizador de Descuentos | Podría crear una venta con descuento irregular y autoaprobarla | Flujo de autorización requiere dos usuarios distintos (ya implementado en autorizar-descuento) |

### 4.5 Plan de capacitación en seguridad

| Rol | Frecuencia | Temas | Modalidad |
|-----|------------|-------|-----------|
| Usuarios operativos | Anual (al ingreso y refuerzo anual) | Política de contraseñas, cómo reportar incidentes, phishing, no compartir credenciales, cierre de sesión | Presencial / Virtual + evaluación |
| Administradores (Sistema + Infraestructura) | Semestral | OWASP Top 10, hardening de servidores, respuesta a incidentes, gestión de secretos, AWS Well-Architected | Taller práctico |
| Desarrolladores | Semestral | Codificación segura, OWASP Top 10, revisión de dependencias, pruebas de seguridad | Taller práctico + code review guiado |
| Comité de Seguridad | Anual | Tendencias de amenazas, actualización normativa, lecciones aprendidas de incidentes | Sesión ejecutiva |

---

## Anexos

### Anexo A: Matriz de riesgos completa

[Incluye la misma matriz de la Sección 2.2 con mayor detalle de análisis de causa, impacto cualitativo y justificación de controles]

### Anexo B: Referencias a estándares

- ISO/IEC 27001:2022 — Sistemas de Gestión de Seguridad de la Información
- ISO/IEC 27002:2022 — Controles de Seguridad de la Información
- ISO/IEC 27005:2022 — Gestión de Riesgos de Seguridad de la Información
- ISO/IEC 27017:2015 — Controles de Seguridad para Servicios Cloud
- ISO/IEC 27018:2019 — Protección de Datos Personales en Cloud
- NIST SP 800-30 — Guide for Conducting Risk Assessments
- OWASP Top 10 2021 — Las 10 Vulnerabilidades Más Críticas
- Ley N.° 29733 — Ley de Protección de Datos Personales (Perú)
- Ley N.° 27269 — Ley de Firmas y Certificados Digitales (Perú)
- AWS Well-Architected Framework — Pilar de Seguridad

---

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE03%20Infraestructura%20Tecnol%C3%B3gica/Entregable2%20Planificaci%C3%B3n%20de%20Seguridad.pdf)
