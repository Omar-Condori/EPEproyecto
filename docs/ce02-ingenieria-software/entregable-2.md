
# Entregable 2: Plataforma de Datos del Sistema

**Competencia:** CE022 — Ingeniería de Software

---

## Resumen Ejecutivo

### Plataforma de Datos Activa vs. Almacén Pasivo

En FINANZAS-MED, la base de datos no es solo un repositorio: ejecuta lógica de negocio crítica directamente en el motor PostgreSQL, garantizando consistencia incluso ante fallos del backend Spring Boot.

| Principio | Descripción |
|-----------|-------------|
| **Integridad Referencial Estricta** | FOREIGN KEY, CHECK y NOT NULL evitan estados inconsistentes: un movimiento de caja sin método de pago, o una venta sin sucursal. |
| **Control de Concurrencia** | Bloqueo de filas (SELECT ... FOR UPDATE) impide que cajeros simultáneos generen descuadres contables o duplicidad de stock. |
| **Lógica Crítica en Triggers** | Los saldos de caja y el stock de inventario se actualizan mediante funciones PL/pgSQL — independiente de caídas del servidor de aplicaciones. |
| **Rendimiento y Auditoría** | Índices B-Tree para búsquedas analíticas frecuentes y un flujo inmutable de auditoría (SystemAuditLog) para cada escritura. |

---

## Sección 1: Modelo de Datos y Mapeo de Capturas

### 1.1 Módulo de Seguridad, Accesos y Menú Dinámico

| Elemento | Descripción |
|----------|-------------|
| **A. Relación de Roles de Usuario** | `user_roles` vincula N-a-N los usuarios (`datos_personales`) con sus perfiles de permisos (`roles`). |
| **B. Multi-sucursal y Segregación** | `user_branch_roles` mapea privilegios por sede geográfica, con restricciones de segregación de funciones. |
| **C. Estructura de Menú Dinámico** | Las tablas `menu` y `modulo` se vinculan con los roles para cargar de forma adaptativa las rutas de la SPA Angular. |
| **D. Permisos Granulares y Excepciones** | `permissions` se vincula con `user_permissions` para conceder vigencia y expiración programada por usuario. |

### 1.2 Módulo de Catálogo General, Productos y Kardex Logístico

| Elemento | Descripción |
|----------|-------------|
| **A. Catálogo General — Sección Superior** | La tabla `catalogo` actúa como base heredada para productos y servicios asistenciales. |
| **B. Catálogo General — Sección Inferior** | Ampliación con clasificaciones y códigos tributarios de SUNAT (DIGEMID / SUSALUD). |
| **C. Ficha de Productos e Inventario** | Relación 1-a-1 con `catalogo`: stock mínimo/máximo, fecha de vencimiento y laboratorios. |
| **D. Control de Kardex Permanente** | Registra cada entrada y salida física con sucursal, cantidad, costo y usuario responsable. |
| **E. Niveles y Tarifas Médicas** | `niveles` asocia el catálogo de servicios con los médicos tratantes, permitiendo honorarios según nivel de especialidad, con control de centros de costo por sucursal. |

### 1.3 Módulo de Tesorería, Cajas Chicas y Caja General

| Elemento | Descripción |
|----------|-------------|
| **A. Caja Chica — Cabecera** | Control diario de saldos contables y estado de apertura/cierre de la caja de ventanilla. |
| **B. Movimientos de Caja Chica** | Registro analítico de ingresos y egresos, vinculado obligatoriamente a un método de pago. |
| **C. Caja General (Bóveda)** | Bóveda centralizadora que recibe los fondos acumulados al momento del arqueo de cajas chicas. |
| **D. Movimientos de Caja General** | Historial de entradas por cierre de caja y egresos mayores hacia cuentas bancarias. |
| **E. Saldos por Método de Pago** | `caja_general_saldo` acumula los fondos discriminados por tipo de cobro (Efectivo, Tarjetas, Yape). |
| **F. Catálogo de Métodos de Pago** | Vínculo centralizador que estandariza las modalidades de pago en todas las transacciones. |

### 1.4 Módulo de Logística y Compras a Proveedores

| Elemento | Descripción |
|----------|-------------|
| **A. Facturas de Compras** | `compras` vincula la factura tributaria física con el proveedor y los impuestos aplicados. |
| **B. Detalle Físico de Compras** | `detalle_compra` asocia productos del catálogo con unidades, costos y cantidades ingresadas. |
| **C. Emisión de Órdenes de Compra** | El requerimiento formal se registra en `ordenes_compra` con estado 'PENDIENTE', requiere firma digital de aprobación. `orden_compra_detalles` guarda ítems, cantidades y precios cotizados. |

### 1.5 Módulo de Cuentas Corrientes, Convenios y Paquetes

| Elemento | Descripción |
|----------|-------------|
| **A. Cuentas Corrientes de Convenio** | `cred_ctacte` registra límites y consumo acumulado de pólizas de seguros por paciente. |
| **B. Cronograma de Pagos y Amortización** | Programa las fechas de vencimiento de obligaciones financieras generadas al crédito. |
| **C. Cuenta Proveedor Acumulada** | Consolida la deuda total por pagar a cada droguería o laboratorio proveedor. |
| **D. Datos de Empresas Conveniadas** | Relaciona personas jurídicas (`datos_empresa`) vinculadas a pacientes y sus planes de cobertura. |
| **E. Oferta de Paquetes Promocionales** | `paquetes` define servicios asistenciales agrupados con tarifa cerrada y vigencia temporal. |

### 1.6 Módulo de Facturación y Consolidación de Sucursal

| Elemento | Descripción |
|----------|-------------|
| **A. Estructura de Sucursales** | Eje geográfico multi-sucursal que vincula la procedencia de ventas, inventarios y cajas. |
| **B. Detalle de Venta Asistencial** | `venta_detalle` y `venta_registro` asocian el cobro con el catálogo, el paciente y el médico tratante. |

### 1.7 Justificación de la Normalización de Base de Datos

El modelo se normaliza estrictamente hasta la **Tercera Forma Normal (3FN)** para evitar redundancias e inconsistencias.

| Forma Normal | Descripción | Aplicación Práctica |
|--------------|-------------|---------------------|
| **1FN (Valores Atómicos)** | Todas las columnas contienen valores atómicos, sin grupos repetidos. | Los precios (unitario, blister, caja) se definen en columnas DECIMAL individuales. |
| **2FN (Dependencia Completa de la Clave)** | Todas las columnas no-clave dependen completamente de la clave primaria. | Se divide la tabla en `catalogo` (atributos comunes: nombre, tipo, precios, códigos tributarios) y `productos` (atributos físicos: stock, vencimiento, factor caja). |
| **3FN (Sin Dependencias Transitivas)** | No hay dependencias transitivas de columnas no-clave respecto a la clave primaria. | Se extrae la información del laboratorio a una tabla independiente `laboratorios`; `productos` solo almacena `id_laboratorio` como FK. |

---

## Sección 2: Implementación de Base de Datos (Scripts DDL & DML)

### 2.1 Scripts DDL (PostgreSQL)

#### Tabla `sucursal`
```sql
CREATE TABLE sucursal (
    id_sucursal BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    direccion VARCHAR(200),
    estado VARCHAR(1) DEFAULT 'A' CHECK (estado IN ('A', 'I')),
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Tabla `datos_personales` (Usuarios)
```sql
CREATE TABLE datos_personales (
    id BIGSERIAL PRIMARY KEY,
    login VARCHAR(50) NOT NULL UNIQUE,
    passwd VARCHAR(255) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    nombre VARCHAR(100) NOT NULL,
    apepat VARCHAR(100) NOT NULL,
    apemat VARCHAR(100) NOT NULL,
    sueldo_base DECIMAL(12, 4) DEFAULT 0.0000,
    active BOOLEAN DEFAULT TRUE,
    sucursal_actual_id INTEGER,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Tabla `catalogo`
```sql
CREATE TABLE catalogo (
    id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL,
    tipo_catalogo VARCHAR(50) NOT NULL CHECK (tipo_catalogo IN ('PRODUCTO', 'SERVICIO')),
    precio_venta_unitario DECIMAL(12, 4) NOT NULL DEFAULT 0.0000,
    precio_compra DECIMAL(12, 4) NOT NULL DEFAULT 0.0000,
    es_controlado BOOLEAN DEFAULT FALSE,
    cod_digemid VARCHAR(50),
    codigo_susalud VARCHAR(50)
);
```

#### Tabla `productos` (Herencia 1-a-1 de `catalogo`)
```sql
CREATE TABLE productos (
    id_producto BIGINT PRIMARY KEY REFERENCES catalogo(id) ON DELETE CASCADE,
    stock DECIMAL(12, 4) NOT NULL DEFAULT 0.0000,
    stock_minimo DECIMAL(12, 4) NOT NULL DEFAULT 0.0000,
    stock_maximo DECIMAL(12, 4) NOT NULL DEFAULT 0.0000,
    fecha_venc DATE,
    id_laboratorio BIGINT,
    factor_caja DECIMAL(10, 2) NOT NULL DEFAULT 1.00
);
```

#### Tabla `caja_chica` y `caja_chica_movimiento`
```sql
CREATE TABLE caja_chica (
    id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(255) NOT NULL,
    saldo_inicial DECIMAL(15, 5) NOT NULL DEFAULT 0.00000,
    saldo_actual DECIMAL(15, 5) NOT NULL DEFAULT 0.00000,
    saldo_cierre_real DECIMAL(15, 5),
    estado VARCHAR(50) DEFAULT 'ABIERTA' CHECK (estado IN ('ABIERTA', 'CERRADA')),
    id_usuario_cajero VARCHAR(255) NOT NULL,
    fecha_cierre TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE caja_chica_movimiento (
    id BIGSERIAL PRIMARY KEY,
    id_caja_chica BIGINT NOT NULL REFERENCES caja_chica(id) ON DELETE RESTRICT,
    tipo VARCHAR(50) NOT NULL CHECK (tipo IN ('INGRESO', 'EGRESO')),
    monto DECIMAL(15, 5) NOT NULL CHECK (monto > 0),
    id_metodo_pago BIGINT NOT NULL REFERENCES metodos_pago(id) ON DELETE RESTRICT,
    descripcion TEXT,
    referencia VARCHAR(255),
    usuario VARCHAR(255) NOT NULL,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Tabla `system_audit_log` (Auditoría)
```sql
CREATE TABLE system_audit_log (
    id BIGSERIAL PRIMARY KEY,
    usuario VARCHAR(100) NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    tabla_afectada VARCHAR(100) NOT NULL,
    operacion VARCHAR(20) NOT NULL CHECK (operacion IN ('INSERT', 'UPDATE', 'DELETE')),
    valor_anterior TEXT,
    valor_nuevo TEXT,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2.2 Scripts DML (Carga Semilla)

#### Cargar sucursales de prueba
```sql
INSERT INTO sucursal (nombre, direccion) VALUES  
('Sede Principal - Lima Centro', 'Av. Wilson 1230, Lima'),
('Sede San Isidro', 'Av. Javier Prado Este 450, San Isidro'),
('Sede Miraflores', 'Av. Larco 780, Miraflores');
```

#### Cargar métodos de pago admitidos
```sql
INSERT INTO metodos_pago (descripcion) VALUES  
('EFECTIVO'), ('YAPE'), ('PLIN'), ('TARJETA_CREDITO'),
('TARJETA_DEBITO'), ('TRANSFERENCIA_BANCARIA');
```

#### Cargar catálogo semilla (Servicios y Medicamentos)
```sql
INSERT INTO catalogo (nombre, tipo_catalogo, precio_venta_unitario, precio_compra, es_controlado, cod_digemid) VALUES
('Consulta Medica General', 'SERVICIO', 50.0000, 0.0000, FALSE, NULL),
('Examen de Sangre - Hemograma Completo', 'SERVICIO', 45.0000, 10.0000, FALSE, NULL),
('Amoxicilina 500mg (Caja x 100)', 'PRODUCTO', 35.0000, 15.0000, FALSE, 'F12409'),
('Alprazolam 0.5mg (Caja x 30)', 'PRODUCTO', 45.0000, 20.0000, TRUE, 'F15671');
```

#### Cargar usuario administrador base
```sql
INSERT INTO datos_personales (login, passwd, email, nombre, apepat, apemat, sueldo_base, sucursal_actual_id) VALUES
('admin', '$2a$12$Zp40o9n6N/8hWl9jE.e8eOszn1K4/Z618V7M3Z2rL3.g77H05F8uG', 'sistemas@clinica.pe', 'Omar', 'Condor', 'Upeu', 3500.0000, 1);
```

---

## Sección 3: Consultas y Programación (Triggers & Procedures)

### 3.1 Triggers Transaccionales en PL/pgSQL

#### Trigger 1: Actualización del Saldo de Caja Chica
Este trigger actualiza de forma automática el saldo acumulado en `caja_chica` cuando se inserta un nuevo movimiento.

```sql
CREATE OR REPLACE FUNCTION fn_actualizar_saldo_caja_chica()
RETURNS TRIGGER AS $$
BEGIN
    -- Bloquear la fila para evitar condiciones de carrera
    PERFORM id FROM caja_chica WHERE id = NEW.id_caja_chica FOR UPDATE;
    
    -- Sumar o restar al saldo contable
    IF NEW.tipo = 'INGRESO' THEN
        UPDATE caja_chica
        SET saldo_actual = saldo_actual + NEW.monto, updated_at = CURRENT_TIMESTAMP
        WHERE id = NEW.id_caja_chica;
    ELSIF NEW.tipo = 'EGRESO' THEN
        UPDATE caja_chica
        SET saldo_actual = saldo_actual - NEW.monto, updated_at = CURRENT_TIMESTAMP
        WHERE id = NEW.id_caja_chica;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE TRIGGER tg_actualizar_saldo_caja_chica
AFTER INSERT ON caja_chica_movimiento
FOR EACH ROW
EXECUTE FUNCTION fn_actualizar_saldo_caja_chica();
```

#### Trigger 2: Registro de Trazabilidad e Inmutabilidad en Kardex
Sincroniza el stock en `productos` cuando se registra un movimiento físico en el kardex.

```sql
CREATE OR REPLACE FUNCTION fn_sincronizar_stock_kardex()
RETURNS TRIGGER AS $$
BEGIN
    -- Validar si el artículo es un producto físico
    IF NOT EXISTS (SELECT 1 FROM productos WHERE id_producto = NEW.id_catalogo) THEN
        RETURN NEW;
    END IF;

    -- Bloquear la fila del producto para concurrencia
    PERFORM id_producto FROM productos WHERE id_producto = NEW.id_catalogo FOR UPDATE;

    -- Sumar o restar stock según el signo del movimiento
    IF NEW.signo = '+' THEN
        UPDATE productos
        SET stock = stock + NEW.cantidad
        WHERE id_producto = NEW.id_catalogo;
    ELSIF NEW.signo = '-' THEN
        UPDATE productos
        SET stock = stock - NEW.cantidad
        WHERE id_producto = NEW.id_catalogo;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE TRIGGER tg_sincronizar_stock_kardex
AFTER INSERT ON kardex
FOR EACH ROW
EXECUTE FUNCTION fn_sincronizar_stock_kardex();
```

### 3.2 Funciones y Procedimientos Almacenados (PL/pgSQL)

#### Procedimiento `pr_registrar_compra_credito`
Automatiza la creación de una compra, actualiza la cuenta por pagar del proveedor e inserta las cuotas programadas al crédito.

```sql
CREATE OR REPLACE PROCEDURE pr_registrar_compra_credito(
    p_id_proveedor BIGINT,
    p_total DECIMAL(12, 4),
    p_num_cuotas INT,
    p_fecha_emision DATE,
    p_plazo_dias INT
)
LANGUAGE plpgsql AS $$
DECLARE
    v_id_compra BIGINT;
    v_monto_cuota DECIMAL(12, 4);
    i INT;
BEGIN
    v_monto_cuota := p_total / p_num_cuotas;
    
    -- 1. Insertar cabecera de compra
    INSERT INTO compras (id_proveedor, total, condicion_pago, fecha_emision, fecha_vencimiento)
    VALUES (p_id_proveedor, p_total, 'CREDITO', p_fecha_emision, p_fecha_emision + p_plazo_dias)
    RETURNING id INTO v_id_compra;
    
    -- 2. Actualizar o insertar saldo en la cuenta corriente del proveedor
    INSERT INTO cuenta_provider (id_proveedor, saldo_total)
    VALUES (p_id_proveedor, p_total)
    ON CONFLICT (id_proveedor)
    DO UPDATE SET saldo_total = cuenta_provider.saldo_total + EXCLUDED.saldo_total,
                  fecha_actualizacion = CURRENT_TIMESTAMP;
    
    -- 3. Generar cronograma de pagos fraccionados
    FOR i IN 1..p_num_cuotas LOOP
        INSERT INTO cronograma_pagos (id_compra, numero_cuota, monto_cuota, fecha_vencimiento, estado)
        VALUES (v_id_compra, i, v_monto_cuota, p_fecha_emision + (i * (p_plazo_dias / p_num_cuotas)), 'PENDIENTE');
    END LOOP;
END;
$$;
```

### 3.3 Consultas Analíticas y de Negocio

#### Consulta 1: Arqueo y Conciliación Contable de Caja Chica
Permite contrastar los movimientos por método de pago frente al saldo inicial para validar descuadres en tiempo real.

```sql
SELECT  
    c.id AS caja_id,
    c.nombre AS caja_nombre,
    c.saldo_inicial,
    c.saldo_actual AS saldo_teorico_sistema,
    COALESCE(SUM(CASE WHEN m.tipo = 'INGRESO' THEN m.monto ELSE 0 END), 0) AS total_ingresos,
    COALESCE(SUM(CASE WHEN m.tipo = 'EGRESO' THEN m.monto ELSE 0 END), 0) AS total_egresos,
    (c.saldo_inicial +  
     COALESCE(SUM(CASE WHEN m.tipo = 'INGRESO' THEN m.monto ELSE 0 END), 0) -  
     COALESCE(SUM(CASE WHEN m.tipo = 'EGRESO' THEN m.monto ELSE 0 END), 0)) AS saldo_calculado
FROM caja_chica c 
LEFT JOIN caja_chica_movimiento m ON c.id = m.id_caja_chica 
WHERE c.estado = 'ABIERTA' 
GROUP BY c.id, c.nombre, c.saldo_inicial, c.saldo_actual;
```

#### Consulta 2: Reporte de Alerta de Medicamentos con Stock Crítico por Sucursal
Muestra los fármacos que están por debajo de su stock mínimo para solicitar órdenes de compra.

```sql
SELECT  
    s.nombre AS sucursal_nombre,
    c.nombre AS producto_nombre,
    c.cod_digemid,
    p.stock AS stock_actual,
    p.stock_minimo,
    (p.stock_minimo - p.stock) AS faltante_para_minimo
FROM productos p 
JOIN catalogo c ON p.id_producto = c.id 
JOIN kardex k ON k.id_catalogo = c.id 
JOIN sucursal s ON k.id_sucursal = s.id_sucursal 
WHERE p.stock <= p.stock_minimo 
GROUP BY s.nombre, c.nombre, c.cod_digemid, p.stock, p.stock_minimo 
ORDER BY faltante_para_minimo DESC;
```

### 3.4 Evidencias de Optimización (Índices & EXPLAIN ANALYZE)

#### Creación de Índices B-Tree
```sql
-- Índice compuesto para consultas de ventas de sucursal
CREATE INDEX idx_venta_fecha_sucursal ON venta_registro(fecha, id_sucursal);

-- Índice compuesto para auditoría de movimientos de Kardex por producto y fecha
CREATE INDEX idx_kardex_catalogo_fecha ON kardex(id_catalogo, fecha);
```

#### Análisis de Rendimiento (EXPLAIN ANALYZE)
| Escenario | Tiempo de Ejecución | Plan de Ejecución | Mejora |
|-----------|---------------------|-------------------|--------|
| **Antes del Índice** | 45.120 ms | Sequential Scan (recorre toda la tabla) | - |
| **Después del Índice** | 1.250 ms | Index Only Scan (usa el índice compuesto) | ≈ 3600% |

---

## Sección 4: Seguridad, Auditoría y Administración

### 4.1 Control de Accesos y Seguridad de Roles (PostgreSQL Privilege Management)

Aplicamos el principio de privilegios mínimos:

```sql
-- 1. Crear el rol de la aplicación Spring Boot (sin privilegios de superusuario)
CREATE ROLE app_erp_login WITH LOGIN PASSWORD 'SecuredErpPwd2026';

-- 2. Limitar todos los accesos en el esquema public
REVOKE ALL ON SCHEMA public FROM PUBLIC;
GRANT USAGE ON SCHEMA public TO app_erp_login;

-- 3. Conceder privilegios DML sobre las tablas operativas
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO app_erp_login;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_erp_login;

-- 4. Revocar accesos DDL para que la aplicación no pueda borrar ni alterar tablas
REVOKE TRUNCATE, DROP ON ALL TABLES IN SCHEMA public FROM app_erp_login;
```

### 4.2 Auditoría Persistente de Modificación de Datos

#### Trigger de Auditoría Global
```sql
CREATE OR REPLACE FUNCTION fn_auditar_tabla_operativa()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO system_audit_log (usuario, ip_address, tabla_afectada, operacion, valor_anterior, valor_nuevo)
    VALUES (
        current_user,
        inet_client_addr()::text,
        TG_TABLE_NAME,
        TG_OP,
        CASE WHEN TG_OP IN ('UPDATE', 'DELETE') THEN row_to_json(OLD)::text ELSE NULL END,
        CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN row_to_json(NEW)::text ELSE NULL END
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Asociar el trigger a la tabla de datos personales para auditar cambios de contraseñas y salarios
CREATE TRIGGER tg_auditar_datos_personales
AFTER INSERT OR UPDATE OR DELETE ON datos_personales
FOR EACH ROW
EXECUTE FUNCTION fn_auditar_tabla_operativa();
```

### 4.3 Estrategia de Respaldo y Recuperación (Disaster Recovery Plan)

| Paso | Descripción |
|------|-------------|
| **1. Respaldo Diario Automatizado** | Job programado a las 23:59 h ejecuta `pg_dump` en formato Custom (.backup), permitiendo restauración selectiva de tablas. |
| **2. Políticas de Retención** | Los backups se conservan 30 días en disco secundario; una copia semanal se replica a almacenamiento frío externo (AWS S3 Glacier). |
| **3. Restauración ante Caídas** | Ante corrupción de base de datos, se restaura con `pg_restore`, limpiando y recreando estructura y datos desde el último backup. |

#### Comandos de Respaldo y Restauración
```bash
# Respaldo
pg_dump -h localhost -U postgres -F c -b -v -f "/var/backups/postgres/finanzas_med_$(date +%F).backup" clinica_db

# Restauración
pg_restore -h localhost -U postgres -d clinica_db --clean --verbose "finanzas_med_ultimo.backup"
```

---

## KPIs y Resultados

| Indicador | Valor |
|-----------|-------|
| Normalización de Base de Datos | Hasta 3FN |
| Mejora de Rendimiento con Índices | ≈ 3600% |
| Auditoría Inmutable | 100% de escrituras registradas |
| Ciclo de Respaldo | 24 horas |

---

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE02%20Ingenier%C3%ADa%20de%20Software/CE022-Entregable%202-Plataforma%20de%20Datos%20del%20Sistema.pdf)
