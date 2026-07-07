# Entregable 3: Plan de Gestión del Proyecto (PMBOK / Agile)

**Competencia:** CE012 — Gestión de Proyectos

---

## 1. Glosario de Términos Técnicos y Metodológicos

| Término | Definición |
|---------|------------|
| **PMBOK** | Guía de buenas prácticas de gestión de proyectos del PMI que define metodologías, áreas de conocimiento y procesos estándar para asegurar el cumplimiento de la línea base del proyecto (alcance, tiempo y costo). |
| **Scrum** | Marco de trabajo ágil iterativo e incremental utilizado para la fase de construcción de la plataforma web. Permite la entrega de valor constante mediante iteraciones denominadas Sprints. |
| **WBS / EDT** | Estructura de Desglose del Trabajo: descomposición jerárquica del alcance total del trabajo para ser ejecutado por el equipo del proyecto, facilitando la estimación y control de costos y tiempos. |
| **EVM** | Earned Value Management / Gestión del Valor Ganado: metodología estándar de control de proyectos que integra las mediciones de alcance, cronograma y presupuesto, evaluando el estado del proyecto frente a la línea base mediante varianzas e índices de rendimiento. |
| **DoD** | Definition of Done / Definición de Terminado: conjunto de criterios acordados que debe cumplir una Historia de Usuario para considerarse completamente lista y libre de errores en el incremento de producto. |
| **DoR** | Definition of Ready / Definición de Listo: criterios mínimos que debe cumplir una Historia de Usuario en el Product Backlog para poder ser seleccionada e integrada dentro de la planificación de un Sprint. |
| **Apache Kafka** | Plataforma de streaming de eventos distribuida utilizada en el sistema 'Finanzas-Clinic' para la mensajería asíncrona de alertas, autorizaciones y auditorías en tiempo real. |
| **JWT** | JSON Web Token: estándar abierto para la transmisión segura de información entre el frontend en Angular y el backend en Spring Boot mediante tokens cifrados, garantizando la seguridad en el acceso basado en roles. |

---

## 2. Acta de Constitución del Proyecto (Project Charter) - Ampliado

### 2.1. Sponsor del Proyecto y Autorizaciones

El proyecto es patrocinado por el Director de Administración y Finanzas de la Clínica Médica Unión, quien provee los recursos financieros y el respaldo administrativo. El Gestor del Proyecto está formalmente facultado para planificar, dirigir y reportar el avance, asignando tareas a los recursos de TI asignados.

### 2.2. Justificación Estratégica

El presente subproyecto se alinea directamente con el objetivo institucional de transformación digital. La desconexión actual de los sistemas contables y médicos genera descuadres recurrentes en cierres de caja chica y general (15% de error por transcripciones), y pérdidas contables por omisión de la depreciación fiscal de activos fijos. La solución 'Finanzas-Clinic' ataca estas brechas reduciendo costos de personal administrativo, eliminando mermas y garantizando el control interno inmutable para cumplir con los estándares de SUSALUD y SUNAT.

### 2.3. Objetivos SMART del Proyecto

| Criterio | Descripción |
|----------|-------------|
| **Específico (Alcance)** | Desarrollar y desplegar en producción el sistema web 'Finanzas-Clinic' con 5 módulos centrales: Caja Chica, Caja General/Arqueos, Activos Fijos con cálculo automático de depreciación, Convenios/Cuentas Corrientes, e Interceptor de Auditoría Activa con mensajería asíncrona Kafka en contenedores Docker. |
| **Medible (Calidad)** | Reducir el tiempo promedio del cuadre diario de cajas de sucursales de 120 minutos a menos de 15 minutos (87.5% de ahorro en tiempo), lograr el 100% de registro de operaciones mutables en base de datos de auditoría, y disminuir en un 80% las facturas vencidas de convenios con aseguradoras. |
| **Alcanzable (Recursos)** | El desarrollo se ejecutará con el equipo interno de desarrollo web de la clínica de 3 especialistas y un analista de aseguramiento de calidad (QA). |
| **Relevante (Estratégico)** | Alinear la solución de software con el plan estratégico de la clínica, apoyando el objetivo de reducción de un 15% en costos operativos administrativos y el aumento de satisfacción del paciente en recepción en un 25%. |
| **Temporal (Tiempo)** | Completar todo el ciclo del proyecto, desde el diagnóstico inicial hasta la capacitación y entrega del producto en producción, en un período límite de 16 semanas. |

### 2.4. Requerimientos de Alto Nivel

| Código | Tipo | Descripción |
|--------|------|-------------|
| RF-01 | Funcional | El sistema debe registrar movimientos de ingresos y egresos de la caja chica y general y emitir arqueos en tiempo real. |
| RF-02 | Funcional | El sistema debe registrar activos fijos y calcular mensualmente su depreciación lineal en base a los parámetros de la SUNAT. |
| RF-03 | Funcional | El sistema debe controlar las cuentas corrientes y los copagos asociados a convenios corporativos de seguros de pacientes. |
| RF-04 | Funcional | El sistema debe inyectar automáticamente un registro en la base de datos de auditoría ante cualquier cambio mutable (INSERT, UPDATE, DELETE). |
| RF-05 | Funcional | El sistema debe permitir la aprobación digital de descuentos mediante el envío de alertas web asíncronas en tiempo real. |
| RNF-01 | No Funcional | La interfaz de usuario debe desarrollarse en Angular y ser responsiva para pantallas de escritorio y tablets en cajas. |
| RNF-02 | No Funcional | La comunicación entre cliente y servidor debe estar protegida mediante tokens de autenticación JWT seguros con expiración. |
| RNF-03 | No Funcional | El sistema de notificaciones debe operar de forma asíncrona mediante Apache Kafka con un tiempo de entrega menor a 2 segundos. |

### 2.5. Supuestos y Restricciones del Proyecto

- **Restricción Presupuestal**: El costo total del proyecto no debe superar los $19,000 USD de inversión inicial asignados.
- **Restricción del Plazo**: La entrega final del software funcional debe concretarse al término de la semana 16, debido al cierre del ciclo académico.
- **Restricción de Infraestructura**: El sistema debe desplegarse obligatoriamente utilizando contenedores Docker para garantizar la portabilidad.
- **Supuesto de Integración**: El sistema de admisión médica legacy expone vistas de base de datos accesibles en red local para obtener los datos de la atención del paciente.
- **Supuesto de Recursos Humanos**: Los desarrolladores de TI mantendrán un nivel de dedicación del 100% durante la fase de desarrollo e implementación ágil.

### 2.6. Registro de Stakeholders y Plan de Interesados

| Stakeholder | Poder | Interés | Estrategia de Involucramiento | Frecuencia |
|-------------|-------|---------|-------------------------------|------------|
| Gerencia General | Alto | Alto | Gestionar de cerca. Reportes financieros consolidados de la Curva S y valor ganado. | Mensual |
| Jefe de Finanzas | Alto | Alto | Gestionar de cerca. Participación en el diseño de interfaces y control de arqueos. | Semanal |
| Cajeros Administrativos | Bajo | Alto | Mantener informados. Talleres prácticos e inducciones de usabilidad de interfaces. | Quincenal |
| Jefe de Logística | Medio | Alto | Mantener informados. Validación del módulo de depreciación y activos fijos. | Quincenal |
| SUNAT / SUSALUD | Alto | Medio | Mantener satisfechos. Garantizar cumplimiento tributario de facturación y logs. | Por hito |
| Equipo de Desarrollo | Medio | Alto | Colaborar activamente. Reuniones diarias (Daily Standup) y planeación Scrum. | Diario |

---

## 3. Gestión del Alcance del Proyecto

### 3.1. Declaración Detallada del Alcance

El proyecto abarca el ciclo de vida completo (CDIO) del subproyecto Finanzas-Clinic. A continuación, se detalla qué actividades y componentes específicos se encuentran dentro de los límites del proyecto:
- **Caja Chica y Caja General**: Incluye la creación de interfaces para apertura, movimientos (ingresos/egresos), arqueos automáticos, y alertas web de saldos bajos.
- **Activos Fijos y Depreciación**: Cálculo mensual automático de depreciación contable según el método de línea recta alineado a la SUNAT. Registro físico y transferencias de activos.
- **Cuentas Corrientes y Convenios**: Administración de saldos pendientes de copago de pacientes con pólizas de seguros corporativos y conciliación de cuentas.
- **Auditoría de Datos Activa**: Clase Java interceptora de la base de datos para inyectar logs inmutables con ID de usuario, fecha, hora, tabla, tipo de operación, valor antiguo y nuevo.
- **Mensajería Kafka**: Servicio independiente en contenedores Docker para la mensajería de notificaciones de seguridad y aprobaciones de descuentos en tiempo real.

### 3.2. Estructura de Desglose del Trabajo (WBS / EDT)

La WBS organiza el alcance total en entregables estructurados. Se definen 5 fases a tres niveles de detalle:
1.0 Gestión de Proyecto
   - 1.1 Project Charter
   - 1.2 Plan de Gestión
   - 1.3 Reuniones de Cierre
2.0 Análisis y Diseño
   - 2.1 Modelado BPMN
   - 2.2 Modelo Base de Datos
   - 2.3 Mockups Angular
3.0 Desarrollo de Software
   - 3.1 Módulo Caja Chica
   - 3.2 Módulo Caja General
   - 3.3 Módulo Activos Fijos
   - 3.4 Módulo Convenios
   - 3.5 Productor Kafka
   - 3.6 Consumidor Kafka
4.0 Seguridad y Auditoría
   - 4.1 Autenticación JWT
   - 4.2 Interceptor Auditoría
5.0 Pruebas y Despliegue
   - 5.1 Pruebas Unitarias
   - 5.2 Pruebas QA Funcionales
   - 5.3 Docker-Compose
   - 5.4 Carga de Datos
   - 5.5 Capacitación Cajas
   - 5.6 Soporte de Salida

### 3.3. Diccionario de la EDT (WBS Dictionary) - Completo

| Código WBS | Entregable / Paquete de Trabajo | Descripción Detallada del Trabajo | Criterios de Aceptación y Validación |
|------------|----------------------------------|-------------------------------------|--------------------------------------|
| 1.1 | Project Charter | Documento formal de inicio que define sponsor, objetivos, restricciones y firma de autorización. | Firma del Patrocinador de la clínica. |
| 1.2 | Plan de Gestión | Planificación de línea base de alcance, tiempo, costos, riesgos y control (EVM). | Aprobación del Directorio de la clínica. |
| 1.3 | Reuniones de Cierre | Reuniones de revisión final del proyecto y actas de aceptación final de los usuarios. | Cierre formal firmado sin pendientes. |
| 2.1 | Modelado BPMN | Modelado de procesos AS-IS (Excel manual) y TO-BE (Digitalizado en cajas) bajo estándar BPMN. | Aprobación del Jefe de Finanzas. |
| 2.2 | Modelo Base de Datos | Diseño lógico y físico de base de datos relacional SQLite/MySQL y modelado de tablas. | Esquema ER aprobado sin redundancias. |
| 2.3 | Mockups Angular | Diseño visual interactivo en wireframes de Angular para cajas y activos. | Aprobación de usabilidad por cajeros líderes. |
| 3.1 | Módulo Caja Chica | APIs y pantallas para la creación de cajas, transacciones, reembolsos y arqueos en local. | Cierres diarios exactos sin cuadres manuales. |
| 3.2 | Módulo Caja General | Lógica de transferencias de cajas de sucursales a tesorería de la clínica. | Consolidación de saldos sin pérdidas de datos. |
| 3.3 | Módulo Activos Fijos | Lógica para el registro de activos, traslados y depreciación contable según ley peruana. | Cálculo exacto validado por contabilidad. |
| 3.4 | Módulo Convenios | Lógica de control de copagos por aseguradora y control de cuentas corrientes corporativas. | Alertas de facturas vencidas correctas. |
| 3.5 | Productor Kafka | Desarrollo de servicios Spring Boot encargados de enviar eventos a tópicos de Kafka. | Mensajes JSON serializados sin fallas de red. |
| 3.6 | Consumidor Kafka | Desarrollo del microservicio notificaciones-service que lee la cola de Kafka. | Notificaciones en Angular visualizadas en real-time. |
| 4.1 | Autenticación JWT | Mecanismo de seguridad Spring Security con emisión de tokens JWT cifrados al loguearse. | Expiración de token a los 60 min, roles validados. |
| 4.2 | Interceptor Auditoría | Clase AuditInterceptor que hereda de JPA Hibernate y escribe logs en base de datos. | Log inmutable del 100% de operaciones de escritura. |
| 5.1 | Pruebas Unitarias | Desarrollo de JUnit en Spring Boot para lógica contable de depreciación y arqueos. | Cobertura mínima del 80% del código backend. |
| 5.2 | Pruebas QA Funcionales | Pruebas de estrés y pruebas funcionales de extremo a extremo en cajas. | Cero errores críticos activos en el backlog. |
| 5.3 | Docker-Compose | Configuración de docker-compose.yml que orquesta BD, Backend, Frontend y Kafka. | Contenedores corren con un solo comando docker. |
| 5.4 | Carga de Datos | Migración de activos fijos, saldos de cajas y cuentas de Excel a la base de datos. | Consistencia total de datos importados. |
| 5.5 | Capacitación Cajas | Capacitaciones guiadas en vivo con manuales de usuario del software. | Cajeros aprueban evaluación de usabilidad. |
| 5.6 | Soporte de Salida | Soporte presencial continuo a cajeros durante la primera semana en producción. | Cero incidentes sin atender más de 24 horas. |

### 3.4. Procedimiento de Control de Cambios del Alcance

Para mantener bajo control el alcance de la línea base, se establece el siguiente flujo operativo:
1. **Solicitud**: El interesado completa un Formulario de Solicitud de Cambio detallando la justificación y beneficio de la modificación.
2. **Evaluación Técnica**: El Gestor de Proyecto evalúa el impacto en tiempo (Gantt) y costo (Curva S).
3. **Decisión**: El Comité de Control de Cambios (Sponsor, PM y Jefe de Finanzas) aprueba, rechaza o aplaza la solicitud.
4. **Implementación**: De ser aprobada, se actualizan el Plan de Gestión, la WBS y el cronograma, comunicándolo a los involucrados.

---

## 4. Gestión del Cronograma del Proyecto

### 4.1. Lista de Actividades y Secuenciamiento (Línea de Base)

| Cód. Act. | Descripción de la Tarea | Duración | Predecesor | Recurso Asignado |
|-----------|--------------------------|----------|------------|------------------|
| A1 | Elaboración de Diagnóstico Organizacional | 10 días | Ninguno | PM / Analista de TI |
| A2 | Análisis Financiero y Caso de Negocio | 7 días | A1 | PM / Asesor Financiero |
| A3 | Elaboración y Firma del Project Charter | 3 días | A2 | Sponsor / PM |
| A4 | Modelado de procesos financieros BPMN AS-IS/TO-BE | 10 días | A3 | PM / Analista de TI |
| A5 | Diseño y modelado ER de base de datos relacional | 5 días | A4 | Arquitecto de TI |
| A6 | Diseño de Mockups e interfaces web en Angular | 5 días | A5 | Dev Frontend |
| A7 | Configuración de entorno Docker local y base de datos | 2 días | A6 | Dev Backend |
| A8 | Desarrollo de API REST Caja Chica y Cierres (Java) | 10 días | A7 | Dev Backend |
| A9 | Desarrollo de API REST Depreciación Activos Fijos | 5 días | A8 | Dev Backend |
| A10 | Desarrollo de API REST Cuentas de Convenios | 5 días | A9 | Dev Backend |
| A11 | Desarrollo de Interceptor de Auditoría Activa Java | 5 días | A10 | Dev Backend |
| A12 | Integración de Productor y Consumidor Apache Kafka | 5 días | A11 | Dev Backend |
| A13 | Construcción de interfaces Angular para Caja Chica/General | 8 días | A12 | Dev Frontend |
| A14 | Construcción de interfaces Angular para Activos y Depreciación | 4 días | A13 | Dev Frontend |
| A15 | Construcción de interfaces Angular para Cuentas/Convenios | 4 días | A14 | Dev Frontend |
| A16 | Desarrollo de JUnit unitarias y cobertura al 80% | 5 días | A15 | Dev Backend / QA |
| A17 | Pruebas funcionales integrales y de estrés de la app | 5 días | A16 | Analista QA |
| A18 | Configuración de docker-compose para despliegue | 3 días | A17 | Arquitecto / SysAdmin |
| A19 | Carga inicial de activos, cajas y saldos históricos | 2 días | A18 | PM / Cajero Líder |
| A20 | Capacitación práctica a cajeros y soporte en sitio | 5 días | A19 | PM / Dev Frontend |

### 4.2. Cronograma del Proyecto y Diagrama de Gantt

El cronograma estructurado define la ruta crítica enfocada en las fases de desarrollo (A8 al A15) debido a la complejidad de las integraciones de Kafka y la seguridad JWT.

### 4.3. Hitos del Proyecto

- **Hito 1 (Semana 4)**: Aprobación del Acta de Constitución (Project Charter) y Caso de Negocio.
- **Hito 2 (Semana 8)**: Cierre del Diseño de Arquitectura, Modelo de Base de Datos y Mockups Angular.
- **Hito 3 (Semana 11)**: Fin del desarrollo del Backend Spring Boot (APIs de Cajas, Activos, Auditoría).
- **Hito 4 (Semana 13)**: Fin del desarrollo del Frontend Angular (Pantallas y flujos reactivos).
- **Hito 5 (Semana 14)**: Aprobación de pruebas QA finales e integración de colas Apache Kafka.
- **Hito 6 (Semana 16)**: Cierre del Proyecto, Go-Live en producción Docker y Capacitación completada.

---

## 5. Plan de Gestión de Costos y Presupuesto

### 5.1. Presupuesto Detallado por Entregables WBS

| Código WBS | Entregable / Paquete de Trabajo | Costo Estimado (USD) |
|------------|----------------------------------|-----------------------|
| 1.0 | Gestión de Proyecto | $1,500 |
| 2.0 | Análisis y Diseño | $3,000 |
| 3.0 | Desarrollo de Software | $11,000 |
| 4.0 | Seguridad y Auditoría | $1,500 |
| 5.0 | Pruebas y Despliegue | $2,000 |
| | **Línea Base de Costos** | **$19,000** |
| | Reserva de Contingencia (10%) | $1,900 |
| | Reserva de Gestión (5%) | $950 |
| | **Presupuesto Total Autorizado** | **$21,850** |

### 5.2. Línea Base de Costos (Curva S)

La Curva S representa la acumulación planificada de costos (PV) a lo largo de las 16 semanas.

### 5.3. Mecanismo de Control Presupuestario (Valor Ganado - EVM)

Para realizar el control de desviaciones de costo y cronograma durante el ciclo de vida del proyecto, se establece una simulación mensual de métricas de Valor Ganado (EVM):

| Mes Evaluado | PV ($) | EV ($) | AC ($) | CV ($) | SV ($) | CPI | SPI | Estado General |
|--------------|--------|--------|--------|--------|--------|-----|-----|----------------|
| Mes 1 (Semana 4) | 4,500 | 4,500 | 4,300 | +200 | 0 | 1.05 | 1.00 | Bajo presupuesto, a tiempo. |
| Mes 2 (Semana 8) | 10,500 | 10,000 | 10,200 | -200 | -500 | 0.98 | 0.95 | Ligero retraso y sobrecosto. |
| Mes 3 (Semana 12) | 17,000 | 17,500 | 17,100 | +400 | +500 | 1.02 | 1.03 | Adelantado y bajo presupuesto. |

Proyección EAC (Mes 3): Con un CPI de 1.02, el costo final estimado es EAC = BAC / CPI = $19,000 / 1.02 ≈ $18,627 USD, lo que proyecta un ahorro neto de $373 USD frente a la línea base.

---

## 6. Plan de Gestión de Riesgos (Matriz Cualitativa y Calor) - Extendido

| Cód. | Riesgo de TI / Negocio | Prob (1-5) | Imp (1-5) | Severidad | Estrategia | Mitigación y Contingencia | Severidad Post | Responsable |
|------|-------------------------|------------|-----------|-----------|------------|-----------------------------|----------------|-------------|
| R-01 | Demoras en entrega de la API del facturador SUNAT. | 3 | 4 | 12 (Alto) | Mitigar | Coordinar sandbox temprano. Crear simulador local de respuestas de facturación. | 6 (Bajo) | Dev Backend |
| R-02 | Caídas en el bus asíncrono de Apache Kafka. | 2 | 4 | 8 (Med) | Mitigar | Implementar controlador de reintentos y persistencia local temporal en base de datos. | 4 (Bajo) | Líder Técnico |
| R-03 | Accesos no autorizados a la BD contable. | 2 | 5 | 10 (Alto) | Evitar | Autenticación JWT con expiración corta, cifrado SSL en red y desactivar root remoto. | 5 (Bajo) | SysAdmin |
| R-04 | Resistencia de cajeros a migrar de hojas de Excel. | 4 | 3 | 12 (Alto) | Mitigar | Talleres de capacitación práctica interactiva y soporte en sitio las primeras semanas. | 6 (Bajo) | Jefe Finanzas |
| R-05 | Sobrecosto en servidores de nube de producción AWS. | 2 | 3 | 6 (Bajo) | Mitigar | Configurar alertas de presupuesto máximo mensual y límites de cuota de recursos. | 3 (Bajo) | Project Manager |
| R-06 | Inconsistencias en el cálculo contable de activos fijos. | 2 | 4 | 8 (Med) | Evitar | Realizar auditoría y validación del cálculo con el equipo contable en la fase de diseño. | 4 (Bajo) | Contabilidad |
| R-07 | Falta de compatibilidad del frontend Angular en navegadores legacy. | 3 | 3 | 9 (Med) | Mitigar | Utilizar compilación polyfill estándar para navegadores antiguos en cajas de la clínica. | 3 (Bajo) | Dev Frontend |
| R-08 | Pérdida de datos históricos de caja chica durante migración. | 2 | 4 | 8 (Med) | Evitar | Generar scripts de validación de carga y copias de seguridad de Excel antes del proceso. | 4 (Bajo) | PM / Dev Backend |
| R-09 | Falta de dedicación del equipo técnico de desarrollo por tareas de soporte. | 3 | 4 | 12 (Alto) | Evitar | Congelar tareas de soporte externo al equipo durante la fase de sprints. | 4 (Bajo) | Sponsor / PM |
| R-10 | Modificación regulatoria imprevista por la SUNAT en facturación. | 3 | 4 | 12 (Alto) | Mitigar | Diseñar módulo de facturación parametrizado y desacoplado mediante microservicios. | 6 (Bajo) | Líder Técnico |

---

## 7. Gestión Ágil del Proyecto (SCRUM)

### 7.1. Ceremonias de Scrum y Flujo de Trabajo

Durante las 6 semanas de desarrollo, se implementan las ceremonias del marco Scrum para garantizar adaptabilidad:

- **Sprint Planning** (2 horas): Al inicio de cada Sprint, el equipo selecciona e integra las historias del Product Backlog prioritarias al Sprint Backlog.
- **Daily Standup** (15 minutos): Reuniones diarias en las que el equipo responde: ¿Qué hice ayer? ¿Qué haré hoy? ¿Qué impedimentos tengo?
- **Sprint Review** (1 hora): Al finalizar el Sprint, se presenta el incremento de producto al Sponsor y al Jefe de Finanzas para su aprobación.
- **Sprint Retrospective** (1 hora): Evaluación de procesos y dinámicas del equipo para identificar oportunidades de mejora continua.

### 7.2. Product Backlog Priorizado (12 Historias de Usuario)

| ID | Historia de Usuario | Prioridad | Criterios de Aceptación (Given-When-Then) |
|----|---------------------|-----------|-------------------------------------------|
| US-01 | Registro de Apertura y Cierre de Cajas | Alta | • Dado que el cajero inicia sesión, cuando ingresa el monto de apertura, entonces el sistema guarda el saldo inicial.<br>• Dado que se cierra la jornada, cuando se calcula el saldo final, entonces el sistema debe alertar si existe un descuadre mayor a $0. |
| US-02 | Registro de Movimientos de Caja Chica | Alta | • Dado que se realiza un egreso, cuando se carga la imagen del comprobante, entonces el sistema descuenta el saldo local.<br>• Dado que el saldo baja del 20%, cuando ocurre un egreso, entonces se genera una notificación Kafka automática. |
| US-03 | Búsqueda e Integración de Admisión | Alta | • Dado que se ingresa el DNI, cuando se busca, entonces se visualizan los procedimientos médicos pendientes de pago.<br>• Dado que se selecciona la atención, cuando se procesa, entonces se precargan los montos del copago automáticamente. |
| US-04 | Emisión de Comprobantes de Pago | Alta | • Dado que el cobro es exitoso, cuando se confirma, entonces se genera el XML firmado del comprobante electrónico.<br>• Dado que falla la conexión a SUNAT, cuando se emite, entonces se almacena en cola de reintentos y se alerta. |
| US-05 | Registro de Activos Fijos | Media | • Dado que se ingresa un nuevo equipo, cuando se guarda, entonces se genera un código de barras único automático.<br>• Dado que el activo cambia de ubicación, cuando se edita, entonces se registra la traza histórica del responsable. |
| US-06 | Cálculo de Depreciación de Activos | Media | • Dado que es fin de mes, cuando se ejecuta el proceso batch, entonces se calcula la depreciación lineal de cada activo.<br>• Dado que el activo llega a su valor residual, cuando se deprecia, entonces el cálculo de depreciación se detiene. |
| US-07 | Control de Cuentas de Convenios | Alta | • Dado que el paciente cuenta con seguro, cuando se emite la factura, entonces el copago se asocia a la aseguradora.<br>• Dado que la factura vence en 30 días, cuando se llega a la fecha, entonces se genera una alerta visual de cobranza. |
| US-08 | Aprobación de Descuentos Especiales | Alta | • Dado que se solicita un descuento, cuando el administrador aprueba en la web, entonces se envía una alerta Kafka al cajero.<br>• Dado que el descuento supera el 30%, cuando se solicita, entonces requiere doble aprobación del Jefe de Finanzas. |
| US-09 | Visualización de Logs de Auditoría | Media | • Dado que el auditor ingresa al módulo, cuando filtra por fecha, entonces se visualiza la traza inmutable del interceptor.<br>• Dado que ocurre una modificación sensible, cuando se registra, entonces se guarda la IP del cliente y el ID de usuario. |
| US-10 | Autenticación y Seguridad de Roles | Alta | • Dado que se ingresan credenciales, cuando se valida, entonces se emite un token JWT firmado por el backend.<br>• Dado que el token JWT es inválido, cuando se realiza una consulta, entonces el servidor retorna un error 401 Unauthorized. |
| US-11 | Monitoreo de Notificaciones Kafka | Baja | • Dado que la consola está activa, cuando se dispara una alerta, entonces se visualiza el payload JSON en pantalla.<br>• Dado que un broker de Kafka se desconecta, cuando se detecta, entonces se envía una alerta al log de auditoría. |
| US-12 | Reportes de Rendimiento de Cajas | Media | • Dado que se ingresa al dashboard, cuando se selecciona la sucursal, entonces se muestra el gráfico consolidado de ingresos.<br>• Dado que finaliza el mes, cuando se solicita, entonces el sistema genera un reporte PDF consolidado automático. |

---

## 8. Planes de Gestión Auxiliares (Calidad, Recursos y Comunicaciones)

### 8.1. Plan de Gestión de Calidad del Software

Para garantizar la robustez, mantenibilidad y seguridad del sistema web, se establecen las métricas mínimas de calidad que deben auditarse antes de la salida a producción:

- **Cobertura de Pruebas**: Las pruebas unitarias JUnit en Spring Boot deben cubrir como mínimo el 80% de las clases de servicio contable y financiero.
- **Tiempo de Respuesta de APIs**: El 95% de las peticiones HTTP REST del frontend al backend deben responder en un tiempo menor a 500 milisegundos en red local.
- **Tasa de Defectos**: Cero defectos críticos o de seguridad (bloqueantes) detectados por el analista QA durante la fase de Sprint Review.
- **Estándares de Codificación**: Uso de herramientas de análisis de código estático (SonarQube) para asegurar que el código no posea fallas críticas de mantenibilidad ni vulnerabilidades.

### 8.2. Plan de Gestión de Recursos y Matriz RACI

| Entregable WBS / Rol | Sponsor | Project Manager | Dev Backend | Dev Frontend | Analista QA |
|-----------------------|---------|-----------------|-------------|--------------|-------------|
| 1.1 Project Charter | A | R | I | I | I |
| 2.1 Modelado BPMN | I | A | R | R | I |
| 3.1 Módulo Cajas | I | A | R | R | C |
| 4.2 Interceptor de Auditoría | I | A | R | I | C |
| 5.1 Pruebas y Despliegue | I | A | R | R | R |

Nota de Roles: R = Responsable de la ejecución, A = Aprobador final, C = Consultado para asesoría, I = Informado del resultado.

### 8.3. Plan de Gestión de Comunicaciones

| Qué Comunicar | Quién Comunica | A Quién | Cómo / Medio | Frecuencia | Objetivo Clave |
|---------------|----------------|---------|--------------|------------|----------------|
| Avance de tareas diarias | Desarrolladores | Project Manager | Daily Standup | Diario | Identificar bloqueos de inmediato. |
| Métricas de Valor Ganado (EVM) | Project Manager | Gerencia y Sponsor | Reporte de Estado | Mensual | Evaluar salud y costos acumulados. |
| Demostración de Incrementos | Equipo de Scrum | Sponsor y Cajas | Sprint Review | Cada 2 semanas | Obtener feedback del avance web. |
| Solicitud de cambio de alcance | Interesados | Comité Cambios | Formulario / Correo | Por ocurrencia | Evaluar el impacto en tiempo/costo. |

---

## 9. Conclusiones y Recomendaciones

El Plan de Gestión de Proyecto para la plataforma 'Finanzas-Clinic' de la Clínica Médica Unión representa un instrumento metodológico exhaustivo que garantiza el éxito del subproyecto. Al combinar de forma estructurada los estándares del PMBOK con la agilidad e iteración de Scrum, el equipo de TI cuenta con directrices claras para ejecutar, controlar y monitorear el proyecto de software dentro del plazo de 16 semanas. Se recomienda estrictamente realizar el control mensual de las métricas de Valor Ganado (EVM) para anticipar desviaciones y aplicar los planes de mitigación de la matriz de riesgos ante cualquier retraso en la API del facturador electrónico.

---

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE01%20Gesti%C3%B3n%20de%20Tecnolog%C3%ADas%20de%20Informaci%C3%B3n/CE0121%20_%20CE0125%20_%20Entregable%203%20.%20Plan%20de%20Gesti%C3%B3n%20del%20Proyecto%20(PMBOK%20_AGILE).pdf)
