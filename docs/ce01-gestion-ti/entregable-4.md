# Entregable 4: Modelado de Procesos AS-IS / TO-BE

**Competencia:** CE011 — Gobierno e Innovación de TI

---

## 1. Marco Teórico de Reingeniería de Procesos (BPR) en Salud

La Reingeniería de Procesos de Negocio (Business Process Reengineering - BPR), propuesta originalmente por Michael Hammer y James Champy, plantea el rediseño radical y la reconceptualización fundamental de los procesos de negocio para lograr mejoras espectaculares en medidas críticas y contemporáneas de rendimiento, tales como costos, calidad, servicio y rapidez.

En el contexto de organizaciones de salud como la Clínica Médica Unión, la reingeniería no consiste en realizar mejoras incrementales o parchar sistemas existentes con macros Excel, sino en rediseñar por completo el flujo informático. Al unificar la admisión con el cobro, se elimina la doble digitación y se introduce control interno mediante la automatización y la mensajería en tiempo real.

La Gestión de Procesos de Negocio (BPM) complementa la BPR al proveer un ciclo de vida estructurado de mejora continua. El estándar BPMN 2.0 (Business Process Model and Notation) se adopta para graficar los procesos AS-IS y TO-BE de forma unificada.

---

## 2. Glosario Ampliado de Notación BPMN 2.0

| Término | Definición |
|---------|------------|
| **Pool / Piscina** | Delimita la frontera del proceso para una organización completa (Clínica Médica Unión). |
| **Lane / Carril** | Subdivisión del Pool que asigna las tareas específicas a un departamento o actor (Cajeros, Contabilidad, TI). |
| **Evento de Inicio** | Define el punto de partida del flujo de secuencia, disparado por un evento (arribo de paciente o fin de mes). |
| **Evento de Fin** | Representa la conclusión del proceso y la inyección o guardado definitivo de los estados financieros en base de datos. |
| **Tarea de Usuario** | Actividad manual interactiva que requiere el uso de una interfaz de software (pantalla Angular de Finanzas-Clinic). |
| **Tarea de Servicio** | Operación automática ejecutada por un servicio de backend (APIs Spring Boot, interceptor de auditoría JPA). |
| **Tarea de Recepción** | Tarea diseñada para esperar un mensaje externo asíncrono (consumo de eventos del tópico de Kafka). |
| **Tarea de Envío** | Tarea encargada de enviar un mensaje de evento (productor Kafka notificando aprobación de descuento). |
| **Compuerta Exclusiva (X)** | Bifurcación que obliga al flujo a seguir un único camino basado en condiciones excluyentes. |
| **Compuerta Paralela (+)** | Bifurcación que inicia múltiples caminos de flujo que se ejecutan de manera simultánea. |
| **Flujo de Secuencia** | Flecha sólida que muestra el orden exacto en que se ejecutan las actividades dentro del carril. |
| **Flujo de Mensaje** | Línea punteada que muestra el intercambio de información entre dos pools o participantes independientes. |
| **Objeto de Datos** | Representa información física o digital que el proceso requiere o genera (por ejemplo, boleta XML firmada). |
| **Almacén de Datos** | Lugar físico o virtual donde el proceso persiste información de forma inmutable (base de datos SQLite/MySQL). |
| **Subproceso Colapsado** | Actividad compleja que contiene un flujo detallado interno representado mediante el símbolo [+] en el diagrama. |

---

## 3. Modelado del Proceso Actual (AS-IS) - Detalle Minucioso

### 3.1. Subproceso 1: Apertura, Cobro y Arqueo de Cajas (AS-IS)

**Narrativa del Proceso:** Actualmente, el cajero inicia el día abriendo una hoja de Excel en blanco en su red local y digitando manualmente el saldo inicial de caja chica. A lo largo del día, los pacientes llegan de admisión médica con su boleta de atención física. El cajero lee los montos de la hoja de papel y los transcribe manualmente en una hoja Excel consolidada y, por separado, en el portal web del facturador del PSE externo. Al cierre del día, el cajero debe contar físicamente los billetes y monedas, calcular el saldo final en Excel y comparar ambos montos. Si existe un descuadre, debe revisar manualmente cada comprobante físico, retrasando el cierre hasta por 2 horas.

| Paso | Actor / Rol | Acción Ejecutada en el Flujo AS-IS | Duración | Ineficiencia o Brecha Identificada |
|------|-------------|-------------------------------------|----------|-------------------------------------|
| 1 | Cajero | Abre archivo Excel local de cuadre de cajas. | 5 minutos | Archivo desprotegido, sin control de versión. |
| 2 | Cajero | Digita el saldo de apertura física de caja. | 3 minutos | Vulnerable ante alteración del saldo de inicio. |
| 3 | Paciente | Arriba a caja con orden de admisión médica impresa. | 2 minutos | Paso físico con riesgo de pérdida de papeles. |
| 4 | Cajero | Revisa manualmente la boleta y código de servicio. | 3 minutos | Posible error en la lectura visual del código. |
| 5 | Cajero | Digita datos del paciente y montos en Excel local. | 5 minutos | Doble digitación manual, tasa de error del 15%. |
| 6 | Cajero | Abre navegador web y se loguea en portal del PSE. | 3 minutos | Login manual sin tokens centralizados. |
| 7 | Cajero | Re-digita datos del paciente y servicio en portal PSE. | 5 minutos | Silos de información desintegrados. |
| 8 | Cajero | Confirma emisión y espera firma del comprobante XML. | 3 minutos | Lentitud de conexión externa del PSE. |
| 9 | Cajero | Imprime comprobante físico y cobra efectivo. | 2 minutos | Consumo excesivo de papel e insumos. |
| 10 | Cajero | Registra número de boleta y cobro en Excel local. | 3 minutos | Digitación redundante propensa a errores. |
| 11 | Cajero | Inicia recuento físico de billetes y monedas. | 15 minutos | Proceso manual lento expuesto a errores de conteo. |
| 12 | Cajero | Calcula sumatoria total en hoja de Excel. | 10 minutos | Errores en fórmulas o celdas desalineadas. |
| 13 | Cajero | Compara saldo físico vs saldo en Excel. | 5 minutos | Sin alertas automáticas de descuadres. |
| 14 | Cajero | Busca y valida boletas físicas una a una ante descuadres. | 60 minutos | Cuello de botella de auditoría manual. |
| 15 | Cajero | Guarda el Excel local y envía reporte por email. | 10 minutos | Falta de base de datos contable centralizada. |

### 3.2. Subproceso 2: Depreciación de Activos Fijos y Control Logístico (AS-IS)

**Narrativa del Proceso:** Logística registra los activos fijos comprados (camas clínicas, ventiladores, PCs) en cuadernos o archivos Excel locales sin etiquetado de códigos de barras. Al cierre de mes, el contador pide la lista de activos e ingresa manualmente las fechas de compra e importes en un Excel contable para calcular la depreciación lineal. Este cálculo manual suele omitir activos retirados o dados de baja por mantenimiento, generando descuadres fiscales en la declaración ante SUNAT.

| Paso | Actor / Rol | Acción Ejecutada en el Flujo AS-IS | Duración | Ineficiencia o Brecha Identificada |
|------|-------------|-------------------------------------|----------|-------------------------------------|
| 1 | Logístico | Registra compra de equipo en Excel de almacén. | 10 minutos | Sin códigos de barras, descontrol físico. |
| 2 | Logístico | Asigna responsable de activo mediante correo. | 15 minutos | Sin firma digital ni traza histórica. |
| 3 | Logístico | Archiva factura física de compra de activo. | 10 minutos | Riesgo de pérdida de sustento contable. |
| 4 | Contador | Solicita por correo la lista actualizada de activos. | 1 día | Espera por comunicación ineficiente. |
| 5 | Logístico | Consolida y envía archivo Excel de activos. | 2 horas | Falta de actualización de ubicación en tiempo real. |
| 6 | Contador | Abre plantilla Excel contable de depreciación. | 5 minutos | Proceso manual desvinculado de logística. |
| 7 | Contador | Digita nuevos activos y montos en Excel. | 30 minutos | Riesgo de error de transcripción de montos. |
| 8 | Contador | Calcula depreciación lineal aplicando fórmulas manuales. | 2 horas | Error en cálculo por activos dados de baja. |
| 9 | Contador | Registra asiento de depreciación manual en balance. | 30 minutos | Sin auditoría ni trazabilidad de cambios. |
| 10 | Contador | Archiva reporte Excel de depreciación del mes. | 10 minutos | Sin base de datos histórica inmutable. |

### 3.3. Subproceso 3: Cobro y Conciliación de Convenios Corporativos (AS-IS)

**Narrativa del Proceso:** Cuando un paciente de un convenio de seguros corporativo es atendido, admisión registra el copago en papel. El paciente va a caja, paga su copago y la boleta se archiva físicamente. Mensualmente, el asistente contable revisa las boletas físicas archivadas, arma un reporte Excel manual y lo envía por correo al seguro para el cobro. Al no existir alertas de facturación, el seguro rechaza facturas vencidas, generando pérdidas de $10,000 USD anuales por cuentas corrientes incobrables.

| Paso | Actor / Rol | Acción Ejecutada en el Flujo AS-IS | Duración | Ineficiencia o Brecha Identificada |
|------|-------------|-------------------------------------|----------|-------------------------------------|
| 1 | Admisión | Recibe datos del paciente asegurado. | 2 minutos | Paso físico con riesgo de error. |
| 2 | Admisión | Valida cobertura en portal externo de la aseguradora. | 10 minutos | Lentitud por accesos individuales. |
| 3 | Admisión | Escribe copago a cobrar en la orden física. | 2 minutos | Escritura manual, montos ilegibles. |
| 4 | Cajero | Registra cobro de copago y entrega boleta física. | 5 minutos | Sin sincronización con la aseguradora. |
| 5 | Cajero | Archiva copia física de la boleta de seguros. | 2 minutos | Riesgo de pérdida física de comprobantes. |
| 6 | Asistente | Revisa archivador físico al final de mes. | 2 días | Ineficiencia extrema por búsqueda manual. |
| 7 | Asistente | Digita facturas de copagos acumulados en Excel. | 3 horas | Doble digitación manual de montos. |
| 8 | Asistente | Envía planilla Excel al seguro por correo electrónico. | 15 minutos | Falta de canal EDI o API de cobros. |
| 9 | Seguro | Audita y concilia planilla Excel enviada. | 5 días | Retraso en el procesamiento del pago. |
| 10 | Seguro | Rechaza facturas en mora de cobro (>30 días). | 1 día | Pérdida financiera neta de copagos vencidos. |

---

## 4. Modelado de Procesos Propuestos (TO-BE)

Se rediseñan los procesos contables optimizándolos mediante la implementación de la plataforma web 'Finanzas-Clinic' con Spring Boot, Angular, autenticación JWT e integración asíncrona de colas Kafka.

### 4.1. Subproceso 1: Cobro y Arqueo Digitalizado (TO-BE)

**Rediseño:** Con 'Finanzas-Clinic', al abrir el sistema, se carga de forma síncrona el saldo anterior de la base de datos mediante validación JWT. Al cobrar, el cajero consulta en tiempo real los datos del paciente desde el sistema de admisión legacy sin digitar de nuevo. Al confirmar el pago, las APIs REST de Spring Boot emiten la boleta electrónica sincronizada al PSE. El cierre diario y arqueo se realiza en un clic, comparando saldos automáticos del sistema con el dinero en efectivo ingresado en Angular. Se genera un registro de auditoría activa inmutable del 100% de operaciones.

| Paso | Actor / Rol | Acción Ejecutada en Finanzas-Clinic | Duración | Mejora y Automatización de TI |
|------|-------------|-------------------------------------|----------|--------------------------------|
| 1 | Cajero | Inicia sesión con autenticación JWT segura. | 1 minuto | Autenticación basada en roles, firma encriptada. |
| 2 | Sistema | Abre caja importando saldo anterior de BD. | 1 segundo | Automatización del estado de caja en base de datos. |
| 3 | Cajero | Busca DNI de admisión legacy en la pantalla web. | 1 minuto | Integración directa de datos en base de datos local. |
| 4 | Sistema | Carga datos médicos del paciente y copago. | 1 segundo | Cero digitación de servicios, datos integrados. |
| 5 | Sistema | Envía comprobante síncrono al PSE y SUNAT. | 2 segundos | API REST de facturación síncrona, cero digitación. |
| 6 | Cajero | Ingresa saldo físico final y ejecuta cierre de caja. | 5 minutos | Cálculo automático de diferencias de arqueos. |
| 7 | Sistema | Valida cierre y registra en base de datos local. | 2 segundos | Cierre instantáneo con reporte en tiempo real. |
| 8 | Sistema | Inyecta logs de auditoría de cierre en BD. | 1 segundo | Log de auditoría inmutable activa con AuditInterceptor. |

### 4.2. Subproceso 2: Gestión de Activos Fijos Automatizado (TO-BE)

**Rediseño:** El logístico registra los activos en la pantalla web de Angular, generándose un código de barras único. Al cierre de mes, el sistema ejecuta un proceso batch programado en Java (Spring Boot) que calcula automáticamente la depreciación contable lineal de todos los activos activos, registrándolo en base de datos en 5 segundos. Esto elimina los descuadres contables y las hojas de Excel manuales.

| Paso | Actor / Rol | Acción Ejecutada en Finanzas-Clinic | Duración | Mejora y Automatización de TI |
|------|-------------|-------------------------------------|----------|--------------------------------|
| 1 | Logístico | Registra activo en Angular y emite código. | 2 minutos | Código de barras único e inventariado en BD. |
| 2 | Logístico | Asigna responsable y ubicación en el sistema. | 1 minuto | Control e histórico de traslados inmutable. |
| 3 | Contador | Inicia proceso batch de depreciación mensual. | 5 segundos | Proceso batch Spring Boot parametrizado SUNAT. |
| 4 | Sistema | Calcula e inyecta depreciación lineal contable. | 1 segundo | Cálculo en BD, cero errores de fórmulas. |
| 5 | Sistema | Genera log de auditoría del proceso de depreciación. | 1 segundo | Trazabilidad inmutable de cálculos fiscales. |
| 6 | Contador | Descarga reporte consolidado en formato PDF/Excel. | 1 minuto | Visualización instantánea en pantalla contable. |

### 4.3. Subproceso 3: Cobro y Conciliación de Convenios Digitalizado (TO-BE)

**Rediseño:** Al ingresar un paciente con seguro, admisión y cobro de copagos se sincronizan de forma digital. El copago pendiente de cobro corporativo se guarda en la cuenta corriente de la aseguradora en el sistema. Apache Kafka dispara alertas automáticas en tiempo real en la pantalla web del contador 10 días antes del vencimiento, asegurando el cobro oportuno y reduciendo a cero las facturas vencidas.

| Paso | Actor / Rol | Acción Ejecutada en Finanzas-Clinic | Duración | Mejora y Automatización de TI |
|------|-------------|-------------------------------------|----------|--------------------------------|
| 1 | Sistema | Sincroniza copago de admisión a la aseguradora. | 1 segundo | Registro contable en base de datos al instante. |
| 2 | Contador | Visualiza saldo corporativo y concilia en Angular. | 2 minutos | Interfaz web consolidada de cuentas. |
| 3 | Sistema | Emite alertas Kafka automáticas de vencimiento. | 2 segundos | Mensajería basada en eventos Apache Kafka. |
| 4 | Contador | Exporta reporte de conciliación en formato XML. | 1 minuto | Formato digital estándar para las aseguradoras. |
| 5 | Aseguradora | Procesa conciliación electrónica de cobros. | 1 día | Aprobación síncrona de facturas sin glosas. |
| 6 | Sistema | Registra confirmación de cobro e inyecta log. | 1 segundo | Actualización automática de cuenta corriente. |

---

## 5. Análisis Comparativo y Eficiencia Cuantificada

Se cuantifica de forma detallada la mejora en tiempos, costos y calidad logrados con el rediseño de procesos.

### 5.2. Reducción de Tiempos de Ejecución

| Proceso de Negocio Clínico | Tiempo AS-IS (Manual) | Tiempo TO-BE (Finanzas-Clinic) | Ahorro Neto (Tiempo) | Eficiencia (%) |
|-----------------------------|-----------------------|----------------------------------|------------------------|----------------|
| Apertura, Cobro y Cierre diario de Cajas | 120 minutos | 15 minutos | 105 minutos por sucursal | 87.5% de ahorro |
| Cálculo de depreciación lineal de Activos | 3 días (espera/procesamiento) | 5 segundos | Prácticamente 100% tiempo | 99.9% de ahorro |
| Conciliación y Cobro de Convenios | 3 días (revisión física) | 5 minutos | Prácticamente 3 días | 98.5% de ahorro |
| Aprobación de descuentos especiales | 24 horas (espera firmas) | 2 segundos | Prácticamente 24 horas | 99.9% de ahorro |

### 5.3. Reducción de Costos y Mermas Financieras

Al optimizar los procesos, se eliminan las fugas de dinero por errores en arqueos e ineficiencias de cobros de seguros:

- **Eliminación de Mermas Contables en Arqueos:** Se reduce la pérdida por descuadres manuales de $500 USD a $50 USD mensuales por sucursal, lo que ahorra $5,400 USD anuales a la clínica.
- **Ahorro de Horas Hombre en Cajas:** La eficiencia temporal de los cajeros al evitar arqueos y digitación redundante se traduce en $8,200 USD anuales recuperados.
- **Recuperación del Canal de Convenios:** Las alertas automáticas del sistema evitan la mora y rechazo de facturas de seguros por cobrar corporativas, recuperando $10,000 USD anuales.

**Ahorro Neto Anual Total para la Clínica Médica Unión:** $23,600 USD

### 5.4. Mejora en la Calidad del Servicio y Seguridad

Se detallan los beneficios en calidad y seguridad de información logrados:

- **Cero Papel y Cuidado Ecológico:** Digitalización completa del flujo de cobros, arqueos y reportes contables, eliminando la necesidad de impresión física.
- **Trazabilidad y Seguridad de Información:** Log inmutable del 100% de operaciones sensibles y seguridad JWT que previene alteraciones malintencionadas en base de datos.
- **Reducción de Colas en Cajas:** El tiempo de espera física del paciente en ventanilla para cobros y copagos corporativos se reduce drásticamente, elevando la experiencia del paciente.

### 5.5. Indicadores de Desempeño (KPIs) Operativos TO-BE

Se definen los indicadores clave de rendimiento técnico del software y base de datos para monitoreo del éxito:

- **TMA (Tiempo Medio de Arqueo):** Fórmula: Suma de tiempo de arqueos del día / Número total de cajas. Meta: TMA < 15 minutos (AS-IS: 120 minutos).
- **TET (Tasa de Error en Transcripción):** Fórmula: (Número de boletas con montos erróneos / Número total de boletas emitidas) * 100%. Meta: TET = 0.0% (AS-IS: 15.0%).
- **TCA (Tasa de Conciliación Automática):** Fórmula: (Monto de copagos conciliados digitalmente / Monto total de facturación de convenios) * 100%. Meta: TCA > 98.0% (AS-IS: 0.0%).

---

## 6. Conclusiones y Recomendaciones

El modelado y comparación de los procesos AS-IS y TO-BE demuestra que la falta de integración digital actual limita la competitividad de la clínica, genera sobrecostos operativos y expone los datos a vulnerabilidades de seguridad. La implementación de la plataforma web 'Finanzas-Clinic' permite rediseñar y optimizar los flujos de cajas, activos contables y convenios corporativos, logrando ahorros netos de tiempo (de hasta 99% de eficiencia en depreciación y aprobaciones) y costos anuales de $23,600 USD.

Se recomienda capacitar a los cajeros en las interfaces web propuestas y adoptar el estándar BPMN para el monitoreo y mejora continua de procesos de la organización.

---

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE01%20Gesti%C3%B3n%20de%20Tecnolog%C3%ADas%20de%20Informaci%C3%B3n/CE0131_CE0135%20%20Entregable%204%20.%20Modelado%20de%20Procesos%20AS_IS%20%20%20%20TO_BE.pdf)
