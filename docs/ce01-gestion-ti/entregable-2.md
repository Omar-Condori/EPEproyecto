# Entregable 2: Business Case del Proyecto

**Competencia:** CE012 — Gestión de Proyectos

---

## 1. Glosario de Términos Económicos y Financieros

| Término | Definición |
|---------|------------|
| **CAPEX (Capital Expenditure)** | Presupuesto destinado a la adquisición de activos fijos, desarrollo del software a medida y licencias iniciales necesarias para poner en marcha el proyecto. |
| **OPEX (Operational Expenditure)** | Costos recurrentes necesarios para mantener operativo el sistema en producción (servidores de nube, soporte correctivo, conectividad, copias de seguridad). |
| **VAN (Valor Actual Neto / NPV)** | Indicador financiero que mide el valor presente de los flujos de caja futuros generados por el proyecto, descontados a una tasa de interés (COK), restando la inversión inicial. |
| **TIR (Tasa Interna de Retorno / IRR)** | Tasa de descuento que hace que el Valor Actual Neto (VAN) del proyecto sea exactamente igual a cero. Representa la rentabilidad intrínseca de la iniciativa. |
| **PRI (Periodo de Recupero de Inversión / Payback Period)** | Tiempo necesario para que los flujos netos acumulados igualen la inversión inicial (CAPEX), indicando el momento exacto de retorno de la inversión. |
| **COK (Costo de Oportunidad del Capital)** | Tasa de descuento de referencia que representa la rentabilidad mínima exigida por la clínica para viabilizar el uso de sus fondos en este subproyecto (establecida en 12% anual). |
| **B/C Ratio (Relación Beneficio / Costo)** | Indicador que evalúa la cantidad de dinero retornado por cada dólar invertido, dividiendo el valor actual de los beneficios entre el valor actual de los costos acumulados. |

---

## 2. Marco de Antecedentes y Justificación de Inversiones de TI

### 2.1. Análisis de Costos de Transacción en Salud
Las ineficiencias en los procesos administrativos clínicos no solo causan molestias a los pacientes, sino que representan costos de transacción invisibles que erosionan el margen operativo de las IPRESS. La desconexión entre el registro médico (admisión) y el cobro (cajas) obliga a una digitación redundante que eleva el riesgo operativo. En economía de la salud, se ha demostrado que cada minuto que un cajero dedica a transcribir datos manuales incrementa el costo de facturación unitario y propicia la pérdida de ingresos por mermas en los arqueos. Automatizar estos flujos mediante una plataforma integrada web elimina la ineficiencia de raíz y protege el flujo de caja diario.

### 2.2. Justificación del Desarrollo Propio vs. Compra de Software
Al evaluar la viabilidad de modernización, el Directorio de la Clínica Médica Unión debe decidir entre adquirir un ERP comercial o construir una solución web a medida. Los ERP comerciales de salud poseen altos costos de licenciamiento por usuario y requieren costos de personalización complejos para adaptarse al sistema de admisión legacy existente. Por otro lado, parchar la situación actual con Excel arriesga la seguridad de los datos. El desarrollo web a medida ('Finanzas-Clinic') representa la opción óptima: elimina los pagos de licencias recurrentes a terceros, brinda propiedad intelectual completa a la clínica, e integra de forma nativa las reglas de copagos y facturación de la SUNAT con un costo total de propiedad (TCO) altamente competitivo.

---

## 3. Justificación del Proyecto

### 3.1. Definición Detallada del Problema y Pérdidas del Negocio
La Clínica Médica Unión opera actualmente con procesos fragmentados. El flujo de información financiera se gestiona mediante hojas de cálculo de Excel locales compartidas de forma inestable en red local. Las ineficiencias identificadas son:
- **Descuadres en Arqueos de Cajas:** Las cajeras transcriben los datos y montos de los procedimientos del paciente en el portal del facturador electrónico del PSE de forma manual, generando errores en el 15% de los cobros, traduciéndose en pérdidas directas por mermas contables de $500 USD mensuales por sucursal.
- **Inoperancia en el Control de Activos Fijos:** La depreciación lineal fiscal contable exigida por SUNAT se realiza manualmente en Excel sin un inventario enlazado a la base de datos de logística. Esto provoca multas regulatorias y descuadres contables en la declaración anual de la clínica.
- **Pérdidas en el Canal Corporativo de Seguros:** Los cobros de convenios y copagos con aseguradoras médicas se procesan de forma manual y tardía. Al no existir alertas de vencimiento, el 5% de las facturas no se cobran a tiempo, perdiendo un estimado de $10,000 USD anuales.
- **Vulnerabilidad en Auditoría Contable:** La ausencia de registros de logs de auditoría activa expone la base de datos a modificaciones de montos y anulaciones de cobros fraudulentos sin rastro de usuario, vulnerando el control interno exigido por SUSALUD.

### 3.2. Objetivos SMART del Proyecto
- **Específico (Specific):** Implementar el sistema web 'Finanzas-Clinic' con módulos de Caja Chica/General, Activos Fijos con cálculo automático de depreciación lineal, Cuentas Corrientes/Convenios, e interceptor de auditoría activa inmutable con alertas Kafka.
- **Medible (Measurable):** Disminuir el tiempo de arqueo diario de cajas de 120 minutos a menos de 15 minutos (87.5% de ahorro temporal), registrar el 100% de operaciones mutables en BD de auditoría, y recuperar el 80% de las facturas de copagos corporativos en mora.
- **Alcanzable (Achievable):** Desarrollar la plataforma a medida mediante un equipo interno de 3 especialistas y 1 QA utilizando Spring Boot, Angular y Docker.
- **Relevante (Relevant):** Alinear el software con el plan estratégico de la clínica, apoyando el objetivo de reducción de un 15% en costos operativos administrativos y elevando la satisfacción del paciente en ventanilla en un 25%.
- **Temporal (Time-bound):** Completar el ciclo de vida del subproyecto, desde el diagnóstico hasta la capacitación final y Go-Live, en un plazo límite de 16 semanas.

### 3.3. Beneficios Esperados del Subproyecto
Se clasifican los retornos y aportes de valor del software a la organización:
- **Beneficios Estratégicos:** Facilitar la toma de decisiones gerenciales mediante dashboards financieros integrados y reportes de rentabilidad en tiempo real. Cumplir con las auditorías regulatorias de SUSALUD al poseer logs inmutables.
- **Beneficios Tácticos:** Reducir la mora del canal corporativo de aseguradoras de salud mediante alertas visuales de facturación. Eliminar las autorizaciones físicas en papel para descuentos a pacientes mediante flujos web asíncronos con Kafka.
- **Beneficios Operativos:** Automatizar los cierres de cajas y el cálculo mensual lineal de depreciación fiscal. Eliminar la digitación redundante y reducir el estrés del personal de cajas.

---

## 4. Análisis de Alternativas Tecnológicas

### 4.1. Descripción Pormenorizada de Alternativas
Se detallan las características y costos estimados de las tres alternativas consideradas:
- **Alternativa A: Adquisición de ERP Comercial de Salud (SAP Business One o similar):** Esta opción propone la compra e implementación de un software comercial cerrado. Si bien el sistema es robusto, requiere un costo de adquisición muy alto (CAPEX estimado de $45,000 USD en licencias y consultorías) más soporte anual costoso. Posee poca flexibilidad para integrarse con el sistema de admisión legacy y adaptaciones regulatorias complejas.
- **Alternativa B: Optimización Legacy mediante Macros Excel y Base Access local:** Mantener el entorno actual y desarrollar macros VBA y una base Access compartida en red local. Aunque tiene costo de adquisición mínimo, carece de concurrencia real, no posee logs de seguridad, las macros se corrompen y es altamente inestable frente al volumen de transacciones de la clínica.
- **Alternativa C (Solución Propuesta): Desarrollo a Medida 'Finanzas-Clinic' (Spring Boot + Angular):** Desarrollo de aplicación web desacoplada en Spring Boot y Angular, dockerizado y con bus de mensajería Kafka. Esta alternativa ofrece personalización completa de flujos de copagos, integración nativa con admisión legacy, inyección automática de logs de auditoría activa y control JWT. Costo de implementación moderado ($19,000 USD) sin licencias recurrentes por usuario a terceros.

### 4.2. Definición y Justificación de Criterios de Evaluación
- **Costo Total de Propiedad (TCO - 25%):** Suma del CAPEX inicial y OPEX acumulado a 3 años. Se priorizan soluciones eficientes sin licenciamientos de uso abusivos.
- **Adaptabilidad y Personalización (20%):** Facilidad para programar las reglas contables de seguros peruanos y copagos específicos de la clínica.
- **Seguridad e Integración (20%):** Capacidad de integrar la admisión legacy y asegurar datos contables mediante tokens JWT cifrados e interceptores.
- **Escalabilidad y Rendimiento (15%):** Tolerancia al volumen de pacientes y concurrencia de cajas en red local mediante colas asíncronas Kafka.
- **Tiempo de Salida al Mercado (Time-to-Market - 20%):** Velocidad de desarrollo e implementación del software (plazo estricto de 16 semanas).

### 4.3. Matriz Comparativa Ponderada de Alternativas
| Criterio de Evaluación | Peso | Alt. A (ERP Comercial) | Alt. B (Macros/Legacy) | Alt. C (Desarrollo Web Propio) |
|-------------------------|------|-------------------------|-------------------------|---------------------------------|
| Costo Total (TCO a 3 años) | 25% | 2 (Costo muy alto) | 5 (Costo mínimo) | 4 (Costo moderado) |
| Adaptabilidad y Personalización | 20% | 3 (Configuración limitada) | 2 (Inestable/Rígido) | 5 (Personalización total) |
| Seguridad e Integración Legacy | 20% | 4 (Estándar/Robusto) | 1 (Altamente vulnerable) | 5 (API REST + JWT + Interceptor) |
| Escalabilidad y Rendimiento | 15% | 4 (Robusto) | 1 (Fallas con volumen) | 5 (Mensajería Kafka Docker) |
| Tiempo de Implementación (Time-to-Market) | 20% | 3 (6 meses de consultoría) | 4 (2 meses) | 5 (Despliegue en 4 meses) |
| **Puntuación Total Ponderada** | **100%** | **3.00** | **2.95** | **4.75** |

---

## 5. Evaluación de Beneficios Financieros

### 5.1. Detalle de Beneficios Cuantificables (Tangibles)
Se calculan matemáticamente los beneficios económicos anuales estimados generados por el software web:
- **Ahorro por Eficiencia en Arqueo de Cajas:** El tiempo promedio de arqueo de caja se reduce de 120 a 15 minutos en las 5 cajas de la clínica. Ahorro de 1.75 horas diarias por caja, sumando 3,193.7 horas recuperadas al año. Valorizado en base al sueldo promedio por hora ($2.56 USD/hora): $8,200 USD anuales.
- **Prevención de Descuadres y Mermas:** La sincronización automática de admisión y cajas reduce los descuadres promedio de $500 USD a $50 USD mensuales en las sucursales. Ahorro neto anual: $5,400 USD.
- **Recuperación de Copagos Corporativos:** Las alertas de facturación y conciliación del módulo de convenios evitan el vencimiento y rechazo de facturas por parte de las aseguradoras corporativas. Ahorro estimado anual: $10,000 USD.

**Beneficios Brutos Anuales: $23,600 USD**

### 5.2. Flujo de Caja Proyectado a 3 Años (Horizonte del Proyecto)
Se modela el flujo neto contable. Costo de descuento (COK) = 12% anual. OPEX anual = $3,500 USD:

| Concepto Financiero | Año 0 (Inversión) | Año 1 | Año 2 | Año 3 |
|----------------------|-------------------|-------|-------|-------|
| Beneficios / Ahorros ($) | 0 | 23,600 | 23,600 | 23,600 |
| Costos de Inversión (CAPEX) ($) | -19,000 | 0 | 0 | 0 |
| Costos Operativos (OPEX) ($) | 0 | -3,500 | -3,500 | -3,500 |
| **Flujo de Caja Neto (FCN) ($)** | **-19,000** | **20,100** | **20,100** | **20,100** |
| **Flujo Neto Acumulado ($)** | **-19,000** | **1,100** | **21,200** | **41,300** |

### 5.3. Cálculo Matemático de Indicadores Financieros (Paso a Paso)

#### 5.3.1. Valor Actual Neto (VAN)
- **Fórmula:** VAN = -I₀ + [ F₁ / (1 + COK)¹ ] + [ F₂ / (1 + COK)² ] + [ F₃ / (1 + COK)³ ]
- **Reemplazando:** COK = 12% = 0.12, I₀ = $19,000, F = $20,100
- **Cálculo:**
  VAN = -19,000 + [ 20,100 / (1.12)¹ ] + [ 20,100 / (1.12)² ] + [ 20,100 / (1.12)³ ]
  VAN = -19,000 + 17,946.43 + 16,023.60 + 14,306.78
  VAN = -19,000 + 48,276.81 = **$29,276.81 USD**
- **Interpretación:** El VAN es positivo ($29,276.81 USD > 0), lo que demuestra que el subproyecto recupera la inversión inicial de capital de la clínica, cubre el costo de oportunidad del dinero del 12% y genera un beneficio económico neto adicional.

#### 5.3.2. Tasa Interna de Retorno (TIR)
- **Ecuación:** 0 = -19,000 + [ 20,100 / (1 + TIR)¹ ] + [ 20,100 / (1 + TIR)² ] + [ 20,100 / (1 + TIR)³ ]
- **Resultado:** TIR = **87.2%**
- **Interpretación:** Al ser la TIR del 87.2% muy superior al COK del 12%, el proyecto posee un margen de seguridad extremadamente alto ante posibles incrementos en la tasa de descuento o variaciones de OPEX.

#### 5.3.3. Periodo de Recupero de Inversión (PRI)
En el Año 1, el flujo neto acumulado es de $1,100 USD, lo que indica que la recuperación se realiza en el Año 1.
- **Cálculo:**
  PRI = (Inversión restante por recuperar al inicio del Año 1 / Flujo de Caja Neto del Año 1) * 12 meses
  PRI = (19,000 / 20,100) * 12 meses = **11.3 meses**
- **Interpretación:** La inversión se recupera en su totalidad en el mes 11.3, antes de finalizar el primer año de operaciones del sistema en producción.

#### 5.3.4. Relación Beneficio / Costo (B/C Ratio)
- **Fórmula:** B/C = Valor Actual de los Beneficios (VAB) / Valor Actual de los Costos (VAC)
- **Cálculo:**
  VAB = [ 23,600 / (1.12)¹ ] + [ 23,600 / (1.12)² ] + [ 23,600 / (1.12)³ ] = $56,682.90 USD
  VAC = 19,000 + [ 3,500 / (1.12)¹ ] + [ 3,500 / (1.12)² ] + [ 3,500 / (1.12)³ ]
  VAC = 19,000 + 3,125.00 + 2,790.18 + 2,491.23 = $27,406.41 USD
  B/C = 56,682.90 / 27,406.41 = **2.07**
- **Interpretación:** Por cada $1.00 USD invertido en CAPEX y OPEX acumulados, la clínica obtiene $2.07 USD de retorno por ahorros de eficiencia contable.

---

## 6. Estimación de Costos (CAPEX / OPEX)

### 6.1. Presupuesto Detallado de Inversión Inicial (CAPEX)
| Cód. Presup. | Concepto de Gasto | Recurso / Rol Asignado | Esfuerzo Estimado | Costo (USD) |
|--------------|--------------------|-------------------------|-------------------|-------------|
| C-01 | Servicios de Dirección de TI | Líder Técnico / Arquitecto de Software | 160 horas ($25/h) | $4,000 |
| C-02 | Desarrollo Backend Java | Desarrollador Spring Boot Experto | 300 horas ($20/h) | $6,000 |
| C-03 | Desarrollo Frontend Angular | Desarrollador Angular / Tailwind | 250 horas ($20/h) | $5,000 |
| C-04 | Aseguramiento de Calidad | Analista QA Funcional y de Seguridad | 166 horas ($15/h) | $2,500 |
| C-05 | Infraestructura de Desarrollo | Servidor de base de datos local SQLite y red | Monto fijo | $500 |
| C-06 | Licencia e Integraciones | API Homologada del Facturador Electrónico PSE | Licencia inicial | $1,000 |

**Inversión Total Autorizada (CAPEX): $19,000 USD**

### 6.2. Presupuesto Operativo y Mantenimiento Anual (OPEX)
| Cód. Gasto | Categoría del Costo | Detalle de los Servicios en Producción | Costo Anual (USD) |
|------------|----------------------|-----------------------------------------|-------------------|
| O-01 | Infraestructura Nube | Servidores EC2 AWS, RDS base de datos y Cloud Kafka | $1,200 |
| O-02 | Soporte Técnico Correctivo | Soporte ante incidentes en cajas (10% del CAPEX) | $1,900 |
| O-03 | Conectividad y Backups | Copias de seguridad automáticas diarias S3 y red local | $400 |
| O-04 | Actualización Regulatoria | Ajustes anuales a parámetros tributarios SUNAT | Variable (incluido en soporte) |

**Gasto Operativo Total Anual (OPEX): $3,500 USD**

---

## 7. Plan de Gestión de Riesgos Iniciales
Se presenta la matriz de riesgos iniciales evaluando probabilidad (1-5) e impacto (1-5) pre y post mitigación:

| Cód. | Riesgo de Negocio / TI | Prob (1-5) | Imp (1-5) | Severidad | Estrategia | Acción de Control Mitigante | Severidad Post | Responsable |
|------|-------------------------|------------|-----------|-----------|------------|-------------------------------|----------------|-------------|
| R-01 | Desviación del alcance del proyecto contable | 3 | 3 | 9 (Med) | Mitigar | Crear Acta de Constitución estricta y control de cambios formal | 3 (Bajo) | Project Manager |
| R-02 | Retraso en la API de facturas SUNAT externa | 3 | 4 | 12 (Alto) | Mitigar | Coordinar sandbox temprano y desarrollar simulación local de API | 6 (Bajo) | Líder Técnico |
| R-03 | Acceso no autorizado a la base de datos de cajas | 2 | 5 | 10 (Alto) | Evitar | Autenticación JWT cifrada con firma y expiración | 5 (Bajo) | SysAdmin |
| R-04 | Resistencia de cajeros a migrar de Excel al sistema | 4 | 3 | 12 (Alto) | Mitigar | Talleres prácticos guiados de usabilidad de interfaces | 6 (Bajo) | Jefe Finanzas |
| R-05 | Sobrecostos de servidores de nube AWS en producción | 2 | 3 | 6 (Bajo) | Mitigar | Configurar alertas de presupuesto máximo mensual AWS | 3 (Bajo) | Project Manager |
| R-06 | Errores en el cálculo de depreciación lineal | 2 | 4 | 8 (Med) | Evitar | Auditar fórmulas matemáticas con contabilidad en diseño | 4 (Bajo) | Contabilidad |
| R-07 | Incompatibilidad de Angular con navegadores legacy | 3 | 3 | 9 (Med) | Mitigar | Utilizar emulación polyfill estándar en Angular para cajas | 3 (Bajo) | Dev Frontend |
| R-08 | Pérdida de datos históricos al migrar Excel a BD | 2 | 4 | 8 (Med) | Evitar | Scripts de carga previa y copias de seguridad de los archivos | 4 (Bajo) | Líder Técnico |
| R-09 | Falta de soporte continuo post Go-Live en cajas | 2 | 4 | 8 (Med) | Mitigar | Asignar soporte técnico presencial en cajas la primera semana | 4 (Bajo) | Jefe de TI |
| R-10 | Cambios regulatorios de SUNAT imprevistos | 2 | 4 | 8 (Med) | Mitigar | Diseñar base de datos parametrizada para tasas tributarias | 4 (Bajo) | Contabilidad |

---

## 8. Conclusiones y Recomendaciones
El análisis formal de este Business Case ratifica la viabilidad y conveniencia de implementar la plataforma web Finanzas-Clinic en la Clínica Médica Unión. La evaluación de alternativas demuestra que el desarrollo a medida es la solución óptima frente a software comercial cerrado, ofreciendo personalización a costo sostenible. Asimismo, los indicadores económicos muestran un retorno financiero contundente (VAN de $29,276.81 USD y TIR de 87.2%), con una recuperación total de la inversión en el mes 11.3 de Go-Live. Se recomienda iniciar la fase de ejecución del proyecto y establecer un comité de seguimiento mensual de costos y riesgos.

---

## Documento Original
[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE01%20Gesti%C3%B3n%20de%20Tecnolog%C3%ADas%20de%20Informaci%C3%B3n/CE0113%20_%20Entregable%202%20.%20Business%20Case%20del%20Proyecto.pdf)
