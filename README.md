# 里 Satori Caja

App de gestión de caja para el restaurante Satori (Costa Rica).  
Registra movimientos de efectivo y transferencias, pagos a proveedores, cierres de turno y saldos en tiempo real.

🔗 **Live:** [cachosatori.github.io/satori-caja](https://cachosatori.github.io/satori-caja)

> Misma base de datos que [Satori Dashboard](https://github.com/CachoSatori/satori-dashboard) y [Satori Propinas](https://github.com/CachoSatori/satori-propinas) — mismo Apps Script v4.1, mismo Google Sheet.

---

## 📁 Archivos del proyecto

```
satori-caja/
├── index.html     ← App completa (frontend + lógica offline)
└── README.md      ← Este archivo
```

El backend (`satori_apps_script_v4.1.js`) está en la carpeta `SATORI PROPINAS` del repo local — es el script unificado de todas las apps.

---

## 🔐 Modos de acceso

El modo se configura desde **Config → Modo de acceso** y queda guardado en el dispositivo. No hay login con contraseña.

| Modo | Pestañas disponibles | Para quién |
|---|---|---|
| **Operacion** | Caja Diaria · Cierre de Turno · Pendientes · Resumen | Encargado de turno |
| **Contador** | Movimientos · Proveedores · Pendientes · Resumen | Contador / revisión |
| **Owner** | Todas + Config | Dueño |

---

## 📊 Pestañas

### Caja Diaria *(Operacion)*
- Apertura de turno: empleado, turno automático (Mediodía / Noche), saldo inicial sugerido desde el cierre anterior
- Cards de estado: Asignado · Gastado efectivo · Disponible
- Ingresos adicionales del turno
- Registro de pagos a proveedores: proveedor, monto en ₡ y/o $, método (Efectivo / Transferencia), referencia
- Las transferencias quedan en estado **Pendiente** hasta confirmación manual

### Cierre de Turno *(Operacion)*
Proceso en **dos fases** con barra de progreso:

**Fase 1 — Mediodía:**
- Ventas POS, dólares físicos, efectivo real en caja, propinas, otros egresos
- Queda sellado e inamovible al confirmar

**Fase 2 — Noche:**
- Ventas POS noche, dólares, efectivo real, propinas
- Separación para el día siguiente (`sep_diaria`) → se sugiere automáticamente en la próxima apertura

### Movimientos *(Contador / Owner)*
- Vista completa de todos los movimientos con filtro por rango de fechas
- Tipos: Ingreso · Egreso-Mercadería · Egreso-Personal · Egreso-Operativo · Egreso-Socios · Traspaso
- Cajas origen: Caja Fuerte · Registradora · Caja Proveedores · Banco
- Edición inline y eliminación con confirmación
- Exportar CSV

### Proveedores *(Contador / Owner)*
- Lista de proveedores con categoría, IBAN, ciclo de pago
- Historial de pagos por proveedor
- Agregar / editar / desactivar

### Pendientes *(Operacion / Contador / Owner)*
- Todos los movimientos con `método = Transferencia` y `estado = Pendiente`
- Agrupados por proveedor con IBAN visible
- Botón **✓ Pagado** por pago individual
- Botón **✓ Marcar todos pagados** por proveedor
- Botón **📷 Descargar imagen** → genera PNG con el detalle del pago para enviar por WhatsApp

### Resumen *(Contador / Owner)*
Saldos calculados en tiempo real desde todos los movimientos:
- **Caja Fuerte:** saldo inicial (AJUSTE) + ingresos − egresos pagados
- **Registradora:** efectivo del día a día
- **Caja Proveedores:** fondo destinado a pagos
- **Banco:** traspasos confirmados

Los pendientes **no** se descuentan hasta ser confirmados como Pagado.

### Config *(Owner)*
| Campo | Descripción |
|---|---|
| Modo de acceso | Cambia entre Operacion / Contador / Owner |
| URL Apps Script | URL del backend |
| Probar conexión | Verifica que el Sheets responda |
| Saldo Inicial Caja Fuerte | Se ingresa una sola vez |
| Tipo de cambio ₡/$ | Default ₡530, usado en Cierre de Turno |

---

## 🗄 Google Sheets — Hojas utilizadas

| Hoja | Contenido |
|---|---|
| `movimientos` | Todos los movimientos (18 columnas) |
| `turnos` | Registros de apertura de turno |
| `cierres_turno` | Cierres diarios fase 1 y 2 |
| `proveedores_caja` | Lista de proveedores |
| `empleados` | Padrón de empleados (compartido con Dashboard y Propinas) |
| `categorias_caja` | Tipos/categorías de movimiento |

**Schema `movimientos`:**  
`id · fecha · turno · tipo · categoria · subcategoria · proveedor_id · proveedor_nombre · empleado_id · empleado_nombre · monto_crc · monto_usd · metodo · caja_origen · estado · referencia · notas · timestamp`

---

## 🏗 Arquitectura

```
Google Sheets (fuente de verdad)
    └── mismas hojas que Dashboard y Propinas

Apps Script v4.1 (API unificada)
    ├── getTurnos / saveTurno
    ├── getMovimientos / saveMovimiento / saveMovimientosBulk / deleteMovimiento
    ├── updateMovEstado
    ├── getProvCaja / saveProvCaja / deleteProvCaja
    └── getCatsCaja / saveCatCaja / deleteCatCaja

GitHub Pages (hosting)
    └── satori-caja/index.html
```

**Sincronización:**
- Al abrir la app → sincroniza automáticamente desde Sheets
- Badge ⟳ en el header → sincronización manual
- Google Sheets siempre manda — los datos de Sheets sobreescriben el caché local ante cualquier conflicto

---

## 🔧 Actualizar el Apps Script

1. Copiar el contenido de `satori_apps_script_v4.1.js` (carpeta SATORI PROPINAS)
2. Ir a [script.google.com](https://script.google.com) → proyecto Satori
3. `Ctrl+A` → pegar → `Ctrl+S`
4. **Implementar → Administrar implementaciones → lápiz → Nueva versión → Implementar**
5. La URL no cambia — no hay que actualizar nada en las apps

---

## 🚀 Setup en nuevo dispositivo

1. Abrir `https://cachosatori.github.io/satori-caja/`
2. Ir a **Config** → ingresar la URL del Apps Script → Guardar
3. Seleccionar el **Modo de acceso** correspondiente
4. La app sincroniza y está lista

---

## 🗺 Relacionado

- [satori-dashboard](https://github.com/CachoSatori/satori-dashboard) — Dashboard de ventas, análisis y métricas del equipo
- [satori-propinas](https://github.com/CachoSatori/satori-propinas) — Distribución de propinas por turno
