# Entregable 1: Diagnóstico Organizacional y Alineamiento Estratégico

**Competencia:** CE011 — Gestión e Innovación de TI

---

## 1. Glosario de Términos Técnicos y Estratégicos

| Término | Definición |
|---------|------------|
| **COBIT 2019** | Marco global de gobierno y gestión de TI de ISACA que ayuda a las organizaciones a estructurar sus objetivos estratégicos y evaluar la madurez de sus capacidades digitales. |
| **PESTEL** | Herramienta de análisis estratégico que permite identificar las fuerzas externas que afectan a la clínica en los ámbitos Político, Económico, Social, Tecnológico, Ecológico y Legal. |
| **FODA Cruzado** | Matriz que cruza factores internos (Fortalezas, Debilidades) con externos (Oportunidades, Amenazas) para generar estrategias específicas. |
| **Cadena de Valor** | Modelo que divide los procesos de la clínica en actividades de soporte y primarias para entender cómo se genera valor. |
| **Ishikawa** | Diagrama de causa-efecto para estructurar las raíces del problema financiero. |
| **5 Porqués** | Metodología de análisis de causa raíz que interroga iterativamente hasta llegar al problema original. |
| **SUSALUD** | Superintendencia Nacional de Salud en el Perú. |
| **Kardex** | Sistema de control y registro del flujo de inventarios. |

---

## 2. Marco de Antecedentes y Contextualización del Proyecto

### 2.1. Contextualización Internacional de la Digitalización en Salud
A nivel mundial, la digitalización de clínicas y centros de salud ha dejado de ser una ventaja competitiva para convertirse en un estándar operativo indispensable. Países en la región de Latinoamérica, como Chile, Colombia y México, han implementado políticas nacionales de interoperabilidad y expediente clínico digital. La integración de la admisión con los módulos contables, automatización de facturación y control inmutable de transacciones han demostrado una reducción de hasta el 30% en los costos operativos de administración de clínicas privadas y un aumento en la fidelización de pacientes por tiempos de espera mínimos en ventanilla.

### 2.2. Situación Nacional de las Clínicas de Salud Privada en el Perú
En el Perú, el marco normativo de SUSALUD exige que las Instituciones Prestadoras de Servicios de Salud (IPRESS) garanticen la calidad en la atención y la seguridad en la protección de datos personales de los pacientes (Ley N° 29733). Por otro lado, la SUNAT ha establecido la obligatoriedad de la emisión de comprobantes electrónicos (boletas y facturas) de manera síncrona. Las clínicas medianas en el Perú a menudo carecen de presupuestos millonarios para adquirir licencias ERP internacionales (como SAP o Oracle), lo que las empuja al uso ineficiente de hojas de cálculo descentralizadas de Excel. Esto genera 'silos de información' donde contabilidad, logística y cajas operan de forma aislada, propiciando errores humanos, descuadres financieros y mermas no rastreables.

---

## 3. Contexto Organizacional de la Clínica Médica

### 3.1. Descripción del Sector y Entorno Competitivo (Porter Ampliado)

| Fuerza Competitiva | Intensidad | Justificación e Impacto Estratégico | Estrategia de TI Propuesta |
|---------------------|------------|--------------------------------------|-----------------------------|
| **Rivalidad entre Competidores** | Alta | Competidores directos cuentan con sistemas ERP integrados, atrayendo más convenios de seguros. | Desarrollar 'Finanzas-Clinic' para agilizar copagos y mejorar la competitividad. |
| **Poder de los Clientes (Pacientes)** | Media-Alta | Pacientes exigen tiempos de espera cortos en cajas y facturación sin errores. | Interfaz Angular rápida y reactiva integrada a la admisión médica. |
| **Poder de los Proveedores** | Media | Proveedores de insumos y farmacéuticos controlan stocks. Requieren pagos ágiles. | Módulo de activos e inventarios que optimiza cuentas por pagar. |
| **Amenaza de Nuevos Entrantes** | Baja-Media | Nuevas clínicas de mediano formato requieren permisos complejos y alta inversión. | Consolidar barreras de entrada mediante tecnología propietaria eficiente. |
| **Amenaza de Productos Sustitutos** | Baja | Medicina alternativa local o consultorios pequeños periféricos. | Mantener calidad médica asistencial y administrativa diferenciada. |

### 3.2. Estructura Organizacional Detallada y Roles Operativos
- **Gerencia General**: Supervisa el cumplimiento de las metas del negocio. Sufre la falta de reportes financieros confiables y en tiempo real.
- **Dirección Administrativa y Finanzas**: Asegura el control interno y la rentabilidad. Afectado por mermas no identificadas y retraso en el cobro de copagos corporativos.
- **Jefatura de Contabilidad**: Responsable del balance fiscal. El cálculo de la depreciación de activos fijos se realiza en plantillas Excel propensas a errores.
- **Jefatura de Admisión y Cajas**: Controla los flujos de efectivo diarios. Afectado por descuadres recurrentes de hasta el 15% debido a transcripciones manuales.
- **Jefatura de Logística**: Gestiona el inventario de la clínica. Carece de un sistema integrado de activos fijos con códigos de barras.
- **Área de TI**: Soporta la infraestructura. Carece de herramientas de seguridad modernas para la protección de accesos contables.

### 3.3. Cadena de Valor Detallada y Análisis de Ineficiencias
- **Gestión de Tecnología (Soporte)**: La carencia de automatización web obliga a dedicar recursos a parchar hojas de Excel dañadas.
- **Admisión (Primaria)**: El registro inicial del paciente se realiza correctamente, pero los datos se quedan aislados en la base de datos SQL legacy.
- **Facturación y Cajas (Primaria)**: El cajero debe digitar nuevamente los datos de admisión. Esto eleva los tiempos de atención de 5 a 15 minutos por paciente.
- **Cuentas por Cobrar (Primaria)**: La conciliación con las aseguradoras se realiza manualmente, perdiendo la trazabilidad de los copagos.

### 3.4. Registro y Matriz Completa de Stakeholders

| ID | Stakeholder/Rol | Poder (1-5) | Int. (1-5) | Clasificación | Expectativa Clave | Información Requerida | Apoyo | Frecuencia |
|----|-----------------|-------------|------------|---------------|-------------------|------------------------|-------|------------|
| S01 | Gerente General | 5 | 5 | Key Player | Rentabilidad, ROI, reportes de valor ganado | Avance general de hitos y costos | Partidario | Mensual |
| S02 | Jefe de Finanzas | 5 | 5 | Key Player | Arqueo de cajas exactos, control de caja chica | Reportes de arqueos y diferencias | Partidario | Semanal |
| S03 | Jefe de Logística | 3 | 4 | Mantener Info | Registro rápido de activos y depreciación | Flujo de inventarios y activos fijos | Neutral | Quincenal |
| S04 | Contador General | 4 | 4 | Gestionar Cerca | Conciliación de cuentas de seguros | Reportes contables de depreciación | Neutral | Quincenal |
| S05 | Cajeros Líderes | 2 | 5 | Mantener Info | Sistema responsivo y amigable | Manuales de uso y flujos de cajas | Neutral | Quincenal |
| S06 | SUNAT/SUSALUD | 5 | 3 | Satisfechos | Cumplimiento normativo | Logs de transacciones y XML firmado | Neutral | Por Hito |

---

## 4. Análisis Estratégico de la Clínica Médica

### 4.1. Filosofía Estratégica e Institucional
La Clínica Médica, en línea con la filosofía institucional de la Universidad Peruana Unión (UPEU), fundamenta su gestión en valores de integridad, calidad en el servicio y responsabilidad social. Su dirección estratégica busca no solo el crecimiento económico sostenible, sino brindar un servicio que refleje calidez humana y respeto por la persona. La digitalización contable es vista como una herramienta ética para garantizar la transparencia en el cobro a los pacientes.

### 4.2. Objetivos Estratégicos Corporativos (Horizonte 2026-2028)
1. **OE1: Eficiencia de Costos Operativos**: Reducir en un 15% los costos operativos de administración general mediante la automatización de procesos financieros redundantes.
2. **OE2: Experiencia y Calidad del Servicio al Paciente**: Incrementar en un 25% la satisfacción de los pacientes en recepción, disminuyendo el tiempo de espera en colas de cajas de 15 a menos de 5 minutos.
3. **OE3: Seguridad de Datos y Cumplimiento Legal**: Garantizar el 100% de trazabilidad de cambios financieros contables y encriptación de datos médicos según SUSALUD y SUNAT.
4. **OE4: Crecimiento del Canal Corporativo de Seguros**: Optimizar en un 80% el cobro y conciliación de copagos corporativos con aseguradoras, reduciendo las cuentas incobrables de seguros.

### 4.3. Matriz FODA Cruzada (Estrategias FO, DO, FA, DA)

#### Fortalezas (Internas)
- F1. Infraestructura de TI moderna.
- F2. Personal calificado e íntegro.
- F3. Prestigio de marca en salud.
- F4. Respaldo institucional UPEU.
- F5. BD legacy estructurada en citas.

#### Debilidades (Internas)
- D1. Cajas y admisión desintegrados.
- D2. Cuadres manuales en Excel.
- D3. Falta de log de auditoría contable.
- D4. Depreciación de activos manual.
- D5. Sin alertas de facturas de seguros.

#### Oportunidades (Externas)
- O1. Demanda de digitalización de salud.
- O2. APIs Open Source y Docker estables.
- O3. Crecimiento de seguros de salud.
- O4. API sandbox del PSE facturador.
- O5. Automatización en la nube.

#### Amenazas (Externas)
- A1. Competidores con ERPs avanzados.
- A2. Regulaciones de SUSALUD/SUNAT.
- A3. Ciberataques a datos de pacientes.
- A4. Resistencia al cambio de usuarios.
- A5. Sobrecostos en servidores de nube.

#### Estrategias
- **FO (Ofensivas)**: Utilizar el prestigio (F3) y la infraestructura moderna (F1) para desplegar servicios en la nube (O5). Usar la BD de citas (F5) para conectar la admisión con las APIs de facturación del PSE (O4).
- **DO (Reorientación)**: Desarrollar la plataforma web Finanzas-Clinic en Spring Boot (O2) para integrar cajas (D1) y automatizar el arqueo diario (D2). Implementar alertas web (O5) de vencimiento de seguros.
- **FA (Defensivas)**: Aprovechar la capacitación del personal (F2) para mitigar la resistencia a nuevos sistemas (A4). Fortalecer la base de datos local (F5) mediante cifrado para proteger datos de ciberataques (A3).
- **DA (Supervivencia)**: Implementar AuditInterceptor en backend (A2, D3) para registrar cambios de forma inmutable. Diseñar módulo contable lineal (A2, D4) homologado a la SUNAT.

### 4.4. Análisis PESTEL de Negocios de Salud en el Perú
- **Político**: El gobierno peruano promueve planes de aseguramiento universal en salud y de interoperabilidad de bases de datos médicas.
- **Económico**: La inflación y la competencia exigen optimizar procesos para mantener márgenes operativos.
- **Social**: El paciente moderno valora la transparencia en la facturación y la rapidez en colas de recepción.
- **Tecnológico**: La consolidación de tecnologías web de código abierto (Angular, Spring Boot, Docker) permite desarrollo a bajo costo.
- **Ecológico**: Políticas de 'cero papel' mediante boletas y cierres digitales.
- **Legal**: SUSALUD, SUNAT y Ley N° 29733 tipifican multas ante fugas de datos.

### 4.5. Factores Críticos de Éxito (FCE) Asociados a TI
- Integración fluida entre admisión, cajas y contabilidad.
- Cumplimiento normativo contable y médico.
- Rapidez en la facturación y reducción de tiempos de espera.
- Trazabilidad total de todas las transacciones financieras.

---

## 5. Diagnóstico Digital / TI (Situación Actual)

### 5.1. Inventario de Sistemas de Información Existentes
1. **BD de Admisión Legacy**: SQL Server 2008 local. Registra citas y procedimientos, sin interfaz web ni cobros.
2. **Excel de Caja Chica/General**: Archivo individual por cajero; se daña con frecuencia y no guarda historial seguro.
3. **Control de Activos en Excel**: Depreciación mensual manual, sin integración con logística ni trazabilidad.
4. **Portal del Facturador (PSE)**: Los cajeros reescriben manualmente los datos para emitir el comprobante SUNAT.

### 5.2. Evaluación Detallada de Madurez Digital (COBIT 2019)
Los cinco objetivos evaluados se ubican en **Nivel 1 (Inicial)**: procesos informales y reactivos.
- **EDM02: Optimización de Beneficios**: Sin análisis de ROI/VAN; pérdidas recurrentes por descuadres.
- **APO02: Gestión de la Estrategia**: Plan de TI informal y reactivo, sin alineación con el negocio.
- **BAI02: Definición de Requisitos**: Sin EDT ni historias de usuario estructuradas; retrabajos frecuentes.
- **DSS05: Servicios de Seguridad**: Contraseñas de red compartidas, sin JWT ni logs de auditoría.
- **MEA01: Monitoreo de Rendimiento**: No existen KPIs técnicos; la operación avanza a ciegas.

### 5.3. Brechas Tecnológicas Identificadas
1. **Integración Admisión–Cajas**: Falta de APIs REST que eliminen la doble digitación manual.
2. **Trazabilidad y Seguridad**: Sin logs de auditoría activa ni autenticación robusta con JWT.
3. **Control Logístico y Depreciación**: No hay módulo integrado de activos fijos con cálculo lineal SUNAT.
4. **Estabilidad y Mensajería**: Ausencia de un bus de mensajería (Kafka) tolerante a fallas del PSE.

### 5.4. Cuantificación de Ineficiencias Operativas y Financieras
Impacto total estimado: ≈ **$23,600 USD anuales**.
- **Doble digitación en cajas**: $8,200 USD / año (1.5 h diarias perdidas por cajero).
- **Mermas contables**: $5,400 USD / año (descuadres mensuales de $500 por caja).
- **Cuentas incobrables**: $10,000 USD / año (facturas de copagos corporativos rechazadas).

---

## 6. Identificación del Problema

### 6.1. Definición Estructurada del Problema de Negocio
"Falta de integración, control y trazabilidad en la gestión financiera y administrativa, que genera ineficiencia operativa, riesgo de pérdidas económicas y demoras críticas en la atención al paciente."

### 6.2. Análisis de Causa Raíz (Ishikawa)
- **Procesos**: Arqueos y cierres manuales; depreciación calculada en Excel.
- **Sistemas**: Admisión y cajas desconectadas; dependencia del PSE externo.
- **Control/Auditoría**: Sin logs inmutables; autorizaciones en papel.
- **Tecnología**: Servidores legacy SQL Server 2008 sin encriptación.

### 6.3. Metodología de los 5 Porqués
1. Se generan descuadres y pérdidas en cajas.
2. Los cajeros cometen errores de transcripción.
3. Deben digitar manualmente desde admisión al Excel.
4. Admisión y facturación están desintegradas.
5. No existe arquitectura de BD integrada ni APIs REST.

**Causa raíz**: La organización opera en Nivel de Madurez Digital 1 (Inicial), sin un plan estratégico de TI que justifique una plataforma financiera unificada.

### 6.4. Mapa de Alineamiento Estratégico y Justificación de la Intervención
| Objetivo | Módulo de Finanzas-Clinic | KPI |
|----------|---------------------------|-----|
| OE1 | Módulo de Cajas y Activos Fijos | Cuadre diario de caja en minutos. |
| OE2 | Aprobaciones y notificaciones Kafka | Tiempo de espera en ventanilla. |
| OE3 | Auditoría Activa y seguridad JWT | 100% de operaciones auditadas. |

---

## 7. Conclusiones y Recomendaciones
- El diagnóstico confirma un Nivel de Madurez Digital 1 (Inicial) en los cinco dominios evaluados de COBIT 2019.
- La desintegración entre admisión, cajas y contabilidad genera pérdidas estimadas en ≈ $23,600 USD anuales.
- Se recomienda implementar 'Finanzas-Clinic' con módulos de Cajas, Activos Fijos, Auditoría Activa y mensajería Kafka.
- El alineamiento estratégico garantiza que la inversión tecnológica responda directamente a los objetivos institucionales OE1–OE4.

---

## Documento Original
[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE01%20Gesti%C3%B3n%20de%20Tecnolog%C3%ADas%20de%20Informaci%C3%B3n/CE0111%20_%20CE0115%20_%20Entregable%201%20.%20Diagn%C3%B3stico%20Organizacional%20y%20Alineamiento%20Estrat%C3%A9gico.pdf)
