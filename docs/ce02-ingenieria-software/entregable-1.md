
# Entregable 1: Requerimientos y Diseño del Sistema

**Competencia:** CE021 — Ingeniería de Software

---

## Contexto del Proyecto

### Definición del Problema
Las clínicas de escala intermedia operan bajo un modelo de gestión fragmentado:
- Los datos de pacientes se encuentran desvinculados de los sistemas de facturación
- Errores en el cálculo de copagos y coaseguros para pacientes asegurados
- Retrasos en la liquidación de cuentas por cobrar con aseguradoras
- Falta de control en los arqueos diarios de caja chica
- Nula trazabilidad del stock de medicamentos (Kardex)
- Riesgos elevados de fraude por ausencia de políticas automatizadas de segregación de funciones

### Solución Propuesta
El sistema **FINANZAS-MED** es un ERP Clínico-Financiero unificado que integra:
- Admisión de pacientes
- Facturación de convenios
- Control de inventario farmacéutico
- Conciliación de cajas
- Garantía de integridad transaccional
- Seguridad de la información

---

## Objetivos del Proyecto

### Objetivo General
Diseñar e implementar un sistema ERP Clínico-Financiero multi-sucursal que integre la admisión de pacientes, facturación de convenios, control de inventario farmacéutico y conciliación de cajas, garantizando la integridad de las transacciones y la seguridad de la información.

### Objetivos Específicos
| Objetivo | Descripción |
|---|---|
| 1 | Reducir el tiempo promedio de facturación en ventanilla para pacientes asegurados de 15 minutos a menos de 2 minutos |
| 2 | Garantizar el cuadre de caja al 100% mediante triggers transaccionales en PostgreSQL |
| 3 | Asegurar la consistencia del stock farmacéutico con Kardex permanente por lotes (FIFO) |
| 4 | Implementar módulo de seguridad por roles dinámicos y permisos temporales con bitácora de auditoría inalterable |

---

## Sección 1: Especificación de Requerimientos

### 1.1 Requerimientos Funcionales (RF)

#### Módulo 1: Admisión y Expedientes Clínicos
| Código | Descripción |
|---|---|
| RF-01-01 | Registro y Validación de Persona: Registro de datos básicos con validación de unicidad por documento |
| RF-01-02 | Expediente Clínico de Paciente: NHC autoincremental, grupo sanguíneo, alergias, antecedentes familiares |
| RF-01-03 | Gestión de Grupo Familiar: Agrupación de pacientes para facturación acumulada y pólizas corporativas |

#### Módulo 2: Caja Chica y Tesorería de Puntos de Venta
| Código | Descripción |
|---|---|
| RF-02-01 | Apertura de Caja Chica: Inicialización con saldo inicial, registro de usuario creador y fecha/hora |
| RF-02-02 | Registro Transaccional de Movimientos: Ingresos y egresos vinculados a métodos de pago, actualización de saldo |
| RF-02-03 | Arqueo y Cierre de Caja Chica: Comparación saldo físico vs teórico, justificación obligatoria si hay diferencias |
| RF-02-04 | Consolidación Automática en Caja General: Transferencia transaccional de fondos netos a caja general |

#### Módulo 3: Facturación, Ventas Asistenciales y Facturación Electrónica
| Código | Descripción |
|---|---|
| RF-03-01 | Motor de Liquidación de Convenios: Cálculo automático de cobertura y copago |
| RF-03-02 | Registro de Comprobante Mixto: División entre copago (caja chica) y cobertura (cuenta corriente de convenio) |
| RF-03-03 | Registro de Venta con Derivación Médica: Vinculación de líneas de detalle al médico prescriptor |
| RF-03-04 | Facturación Electrónica en Tiempo Real: Integración con API NubeFact, almacenamiento de Hash SUNAT, firma digital, PDF y XML |

#### Módulo 4: Logística, Compras e Inventarios (Kardex)
| Código | Descripción |
|---|---|
| RF-04-01 | Emisión y Aprobación de Órdenes de Compra: Requerimientos formales con firma digital de supervisor |
| RF-04-02 | Registro de Compras e Ingreso al Almacén: Actualización de stock, registro de IGV, detracciones y percepciones |
| RF-04-03 | Cronograma de Pagos a Proveedores: Programación de cuotas para compras a crédito |
| RF-04-04 | Registro del Kardex Permanente: Historial de entradas/salidas por lote con fecha, sucursal, costo y saldo |

#### Módulo 5: Seguridad y Control Operativo
| Código | Descripción |
|---|---|
| RF-05-01 | Menú Dinámico por Perfil y Sucursal: Renderizado condicional según rol de usuario |
| RF-05-02 | Permisos Temporales y Restricciones: Privilegios especiales con fecha de caducidad automática |
| RF-05-03 | Trazabilidad por Auditoría (AuditLog): Registro de solo lectura de todas las acciones de escritura |

### 1.2 Requerimientos No Funcionales (RNF)

| Código | Categoría | Descripción |
|---|---|---|
| RNF-01 | Seguridad | Autenticación JWT asimétrico RSA-2048; contraseñas BCrypt (factor ≥12) |
| RNF-02 | Disponibilidad | Notificaciones asíncronas vía Kafka; respuesta API ≤ 200ms |
| RNF-03 | Precisión | Campos monetarios DECIMAL(15,5) para evitar errores de redondeo |
| RNF-04 | Rendimiento UI | SPA Angular 17+ con lazy loading; carga inicial &lt; 1.5s |
| RNF-05 | Control de Acceso | Bloqueo 15 min tras 5 intentos fallidos; máx. 3 sesiones concurrentes |
| RNF-06 | Tolerancia a Fallos | Reintento automático si NubeFact no responde (estado PENDIENTE_SUNAT) |

### 1.3 Reglas de Negocio (RN)

| Código | Descripción |
|---|---|
| RN-01 | No se cierra caja chica con transacciones sin validar o sin monto físico contado |
| RN-02 | Bloqueo de coberturas si consumo supera límite del convenio y corte_límite está activo |
| RN-03 | Segregación de funciones: Cajero ≠ Aprobador/Auditor en misma sucursal |
| RN-04 | Dispensación de fármacos con vencimiento bajo regla FIFO |
| RN-05 | No se registra compra a crédito sin cronograma de cuotas |
| RN-06 | No se consumen servicios de paquete vencido o eliminado |
| RN-07 | SystemAuditLog es inmutable: ni superadministrador puede alterarlo |

### 1.4 Restricciones del Sistema

- **Tecnológica:** Toda persistencia en PostgreSQL 16+
- **Regulatoria:** Facturación electrónica conforme a SUNAT; catálogo de fármacos con códigos DIGEMID/SUSALUD
- **Operativa:** Frontend compatible con Chrome 100+, Firefox 100+, Safari 15+

### 1.5 Historias de Usuario (HU)

#### HU-01: Gestión de Venta Mixta (Particular/Asegurado)
**Como** Cajero de Ventanilla  
**Quiero** registrar una venta asistencial para un paciente con convenio de seguro médico  
**Para** que el sistema calcule el copago correspondiente y cargue el saldo a la aseguradora.

**Precondición:** Paciente registrado, convenio activo, carta de garantía vigente.

**Flujo Principal:**
1. Cajero selecciona paciente
2. Sistema carga datos del convenio y coaseguro
3. Cajero agrega servicios al detalle
4. Sistema calcula cobertura y copago
5. Cajero selecciona método de pago y procesa cobro
6. Sistema registra copago en caja chica, incrementa consumo del convenio, imprime comprobante

**Criterio de Aceptación (Gherkin):**
```gherkin
Dado que el paciente está afiliado al plan "Rímac Oro" con coaseguro del 80% y copago fijo de S/ 20.00
Cuando el cajero ingresa un examen de laboratorio con costo de S/ 100.00
Entonces el sistema debe calcular la Cobertura en S/ 80.00, el Copago del paciente en S/ 20.00, y el total a cobrar en S/ 40.00
```

#### HU-02: Cierre y Arqueo de Caja Chica
**Como** Cajero de Ventanilla  
**Quiero** realizar el arqueo y cierre de mi caja chica al final de mi turno  
**Para** consolidar mis movimientos y transferir fondos netos a caja general.

#### HU-03: Emisión de Orden de Compra
**Como** Personal de Logística  
**Quiero** registrar una orden de compra para reabastecer medicamentos  
**Para** cotizar y solicitar abastecimiento formal a un proveedor autorizado.

---

## Sección 2: Prototipo Navegable

### 2.1 Mapeo de Interfaces Gráficas

#### A. Pantalla de Acceso Seguro (Login)
- Contenedor centrado con fondo blanco y bordes redondeados
- Campos para usuario y contraseña (con botón para alternar visibilidad)
- Enlace para recuperación de credenciales

**Lógica Técnica:**
- Validaciones reactivas con Angular Forms
- POST al backend con credenciales; respuesta con JWT firmado RSA-2048 si coinciden
- Registro de intentos de acceso en tabla IntrusionAttempt

#### B. Panel de Control Financiero (Dashboard Principal)
- Menú de navegación lateral izquierdo dinámico
- Tarjetas de KPIs en tiempo real (ventas, egresos, valor de inventario)
- Gráficos estadísticos de rendimiento operativo
- Indicador de expiración de sesión

**Lógica Técnica:**
- Menú consulta tablas menu y modulo según rol y sucursal activa
- KPIs cargan datos mediante observables Angular sobre API REST
- Indicador de tiempo restante sincronizado con expiración del JWT

### 2.2 Validación con Stakeholders
El prototipo de alta fidelidad en Angular fue validado mediante metodologías ágiles de diseño centrado en el usuario (UCD).

- **Cuestionario:** System Usability Scale (SUS)
- **Muestra:** 5 cajeros + 2 administradores
- **Puntuación Promedio:** 82.5/100
- **Categoría:** "Excelente"

---

## Sección 3: Diseño Arquitectónico

### 3.1 Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Frontend | Angular 17+ · Tailwind CSS (SPA responsiva) |
| Backend | Spring Boot 4.0.1 sobre Java 21 |
| Base de Datos | PostgreSQL 16 (transaccional, ACID) |
| Mensajería | Apache Kafka (notificaciones asíncronas) |
| Web Server | Nginx (servir build Angular, proxy REST) |

### 3.2 Diagrama de Componentes
El sistema desacopla físicamente:
1. **Presentación:** Angular SPA
2. **Control y Seguridad:** Spring Security + JWT
3. **Servicios de Negocio:** Spring Boot
4. **Integración y Eventos:** Apache Kafka
5. **Notificaciones:** Microservicio consumidor independiente

### 3.3 Diagrama de Despliegue

```
[Cliente Web] → [Nginx] → [Spring Boot API] → [PostgreSQL 16]
                                    ↓
                          [Apache Kafka] → [Microservicio Notificaciones]
```

**Flujo:**
1. Nginx sirve el build de Angular 17 y hace proxy HTTP/REST hacia el backend
2. Spring Boot API (JVM 21) atiende la lógica de negocio y persiste vía JDBC
3. Backend publica eventos en Apache Kafka (topic: sistema.notificaciones)
4. Servicio consumidor independiente procesa eventos y envía notificaciones por SMTP/WhatsApp sin bloquear la venta

### 3.4 Registro de Decisiones Arquitectónicas (ADR)

#### ADR-001: Kafka para Notificaciones
**Contexto:** Envío de comprobantes electrónicos vía email/WhatsApp tiene latencia variable y es propenso a fallos de red.

**Decisión:** Desacoplar servicio de notificaciones del flujo de venta usando Apache Kafka.

**Consecuencias:**
- ✅ Menor latencia percibida por el cajero
- ✅ Tolerancia a fallos: si SMTP falla, consumidor reintenta sin afectar la venta
- ❌ Incremento de complejidad en infraestructura

#### ADR-002: PostgreSQL con Lógica en el Motor
**Contexto:** Se requiere alta consistencia transaccional (ACID) y garantizar que la lógica de dinero en caja sea incorruptible.

**Decisión:** Utilizar PostgreSQL 16+ con restricciones estrictas de claves foráneas y triggers a nivel de motor.

**Consecuencias:**
- ✅ Integridad de datos absoluta
- ✅ Bloqueos de fila (SELECT ... FOR UPDATE) ante condiciones de carrera
- ❌ Base de datos asume parte de la lógica de negocio, dificultando migración futura

#### ADR-003: JWT Asimétrico RSA-2048
**Contexto:** Delegar accesos sin exponer el backend a consultas repetitivas de verificación de sesión.

**Decisión:** Implementar Spring Security para emitir tokens JWT firmados con algoritmo asimétrico RSA-2048.

**Consecuencias:**
- ✅ Mayor seguridad: llave pública no permite generar tokens válidos
- ✅ Verificar sesión en microservicios sin consultar BD de usuarios
- ❌ Ligeramente mayor sobrecarga de procesamiento vs HS256

---

## Sección 4: Diseño Detallado (Diagramas UML)

### 4.1 Diagrama de Casos de Uso del Sistema

| Actor | Casos de Uso |
|---|---|
| **Cajero** | Apertura de caja chica, Registro de venta mixta, Arqueo y cierre de caja |
| **Administrador** | Aprobar solicitud de anulación, Gestionar permisos temporales, Ver trazabilidad de auditoría |
| **Logística** | Emitir orden de compra, Registrar compra y actualizar Kardex |

### 4.2 Diagrama de Secuencia: Flujo de Venta Mixta

1. Cajero solicita cobrar una orden en Frontend Angular
2. Spring Security valida token JWT con llave pública RSA
3. VentaService consulta en PostgreSQL la cuenta corriente de convenio
4. Servicio calcula cobertura y copago
5. Transacción PostgreSQL:
   - Inserta en venta_registro y venta_detalle
   - Actualiza consumo acumulado en cred_ctacte
   - Inserta movimiento en caja_chica_movimiento
   - Trigger actualiza saldo de caja_chica
6. Commit de la transacción
7. Publicación de evento en Kafka (venta.confirmada)
8. Retorno HTTP 201 y frontend muestra éxito e imprime boleta

### 4.3 Diagrama de Estados: Ciclo de Vida de la Caja Chica

| Estado | Descripción |
|---|---|
| **CERRADA** | Estado inicial; caja inactiva hasta que se ingresa saldo inicial |
| **ABIERTA** | Se registran ingresos y egresos continuamente, actualizando el saldo contable |
| **ARQUEO** | Se compara saldo físico contado contra saldo contable calculado |
| **DESCUADRE** | Si hay diferencias, se exige justificación antes de continuar |
| **CERRADA (final)** | Tras validar, fondos netos se transfieren a Caja General |

---

## Anexos

### Anexo A: Matriz de Trazabilidad de Requerimientos (MTR)

| ID Req | Requerimiento Funcional | Caso de Uso | Componente Backend | Tabla BD Principal | Caso de Prueba |
|---|---|---|---|---|---|
| RF-01-01 | Registro Único de Persona | Registrar Paciente | DatosPersonalesService | datos_personales | testRegistroPersonaExitoso |
| RF-01-02 | Expediente Clínico Paciente | Registrar Paciente | DatosPacienteService | datos_paciente | testCreacionExpedientePaciente |
| RF-02-01 | Apertura de Caja Chica | Apertura de Caja | CajaChicaService | caja_chica | testAperturaCajaChicaConSaldoInicial |
| RF-02-03 | Arqueo y Cierre de Caja | Arqueo y Cierre | CajaChicaService | caja_chica | testArqueoCerrarCajaChicaExitoso |
| RF-03-01 | Liquidación de Convenios | Registrar Venta | VentaService | cred_planes, cred_planes_descuento | testCalculoCopagoYCoaseguro |
| RF-03-02 | Emisión Comprobante Mixto | Registrar Venta | VentaRegistroService | venta_registro, cred_ctacte | testCobroMixtoVentaAsegurada |
| RF-04-01 | Emisión Orden de Compra | Emitir Orden | OrdenCompraService | ordenes_compra | testEmisionOrdenCompraAprobada |
| RF-04-04 | Control de Kardex Permanente | Registrar Venta/Compra | KardexService | kardex | testActualizacionKardexSalidaVenta |
| RF-05-03 | Bitácora de Auditoría | Todas las escrituras | AuditLogService | SystemAuditLog | testRegistroLogsAuditoriaInmutables |

---

## KPIs y Resultados Esperados

| Indicador | Valor Objetivo |
|---|---|
| Tiempo de facturación en ventanilla | &lt; 2 minutos |
| Cuadre de caja automático | 100% |
| Trazabilidad de Kardex | FIFO por lotes |
| Auditoría inalterable | 100% |

---

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE02%20Ingenier%C3%ADa%20de%20Software/CE021-Entregable%201-Requerimientos%20y%20Dise%C3%B1o%20del%20Sistema.pdf)
