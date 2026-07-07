# Entregable 2: Plataforma de Datos del Sistema

**Competencia:** CE022 — Ingeniería de Software

## Descripción

Diseño e implementación de la plataforma de datos del sistema Finanzas-MED, un ERP Clínico y Financiero que procesa cobros inmediatos, deudas diferidas, inventarios críticos y arqueos contables.

## Contenido

### 1. Modelo de Datos

#### Módulo de Seguridad, Accesos y Menú Dinámico
- Tablas: `usuarios`, `roles`, `permisos`, `menus`, `auditoria_accesos`
- Control de acceso basado en roles (RBAC)

#### Módulo de Catálogo General, Productos y Kardex Logístico
- Tablas: `categorias`, `productos`, `kardex`, `lotes`, `almacenes`
- Control de inventario permanente con costo promedio

#### Módulo de Tesorería, Cajas Chicas y Caja General
- Tablas: `cajas`, `aperturas`, `arqueos`, `movimientos_caja`, `conciliaciones`
- Flujo de efectivo con cuadratura automática

#### Módulo de Logística y Compras a Proveedores
- Tablas: `proveedores`, `ordenes_compra`, `recepciones`, `devoluciones`
- Ciclo completo de compras con aprobaciones

#### Módulo de Cuentas Corrientes, Convenios y Paquetes
- Tablas: `convenios`, `aseguradoras`, `paquetes`, `cuentas_corrientes`, `liquidaciones`
- Gestión de convenios corporativos con aseguradoras

#### Módulo de Facturación y Consolidación de Sucursal
- Tablas: `facturas`, `comprobantes`, `consolidacion_sucursal`, `cierre_contable`
- Facturación electrónica SUNAT

### 2. Implementación (PostgreSQL)

- **DDL:** Scripts completos de creación de tablas con constraints, índices y relaciones
- **DML:** Carga de datos semilla para catálogos iniciales
- **Triggers:** Validaciones transaccionales en PL/pgSQL
- **Procedimientos Almacenados:** Funciones de negocio (cierre de caja, liquidación de convenios)
- **Consultas Analíticas:** Reportes de morosidad, rentabilidad por sucursal, rotación de inventarios

### 3. Seguridad y Auditoría

- Control de accesos y seguridad de roles (PostgreSQL Privilege Management)
- Auditoría persistente de modificación de datos
- Estrategia de respaldo y recuperación (Disaster Recovery Plan)
- Índices y plan de ejecución (EXPLAIN ANALYZE)

## Documento Original

[Descargar PDF](https://github.com/Omar-Condori/EPEproyecto/raw/main/CE02%20Ingenier%C3%ADa%20de%20Software/CE022-Entregable%202-Plataforma%20de%20Datos%20del%20Sistema.pdf)
