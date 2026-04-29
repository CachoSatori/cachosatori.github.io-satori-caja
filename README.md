# 里 Satori · Módulo Caja

Control de caja, pagos a proveedores y flujos de efectivo para **Satori Restaurante**.

> App separada del [Dashboard de Ventas](https://github.com/CachoSatori/satori-dashboard) — misma base de datos (Google Sheets), interfaz optimizada para uso diario en el punto de venta.

🔗 **Live:** [cachosatori.github.io/satori-caja](https://cachosatori.github.io/satori-caja)

---

## ¿Qué hace esta app?

Reemplaza la planilla de Excel de caja. Permite registrar todos los movimientos de efectivo del restaurante — pagos a proveedores, adelantos de salario, propinas, retiros — y generar resúmenes diarios, semanales y mensuales para el contador.

---

## Módulos

### Cierre de Turno *(cajero / manager)*
- Empleado, turno (mediodía / noche) y fecha
- Caja asignada para pagos en ₡ y $
- Opción de agregar dinero adicional recibido en el turno
- Registro de pagos a proveedores: proveedor (con categoría automática), monto en ₡ y/o $, método de pago, nota/factura
- Efectivo al cierre en ₡ y $
- Todos los datos se sincronizan automáticamente a Google Sheets

### Movimientos *(manager / owner)*
- Tabla completa de todos los movimientos del período
- Tipos: Ingreso · Egreso-Mercadería · Egreso-Personal · Egreso-Operativo · Egreso-Socios · Traspaso
- Filtros por fecha, tipo y proveedor
- Saldos en tiempo real de cada caja física (Registradora / Caja Proveedores / Caja Fuerte / Banco)
- Exportar CSV

### Proveedores *(manager / owner)*
- Gestión dinámica: agregar, editar, desactivar
- Campos: categoría, moneda preferida (₡ / $), ciclo de pago (diario / semanal / quincenal / mensual), método, IBAN
- Deuda pendiente visible por proveedor

### Pendientes *(manager / owner)*
- Semáforo de vencimiento: 🔴 vencido · 🟡 por vencer · 🟢 al día
- Deuda total en ₡ y $
- Detalle de facturas pendientes por proveedor
- Botón "Pagar" por línea → marca como pagado y actualiza Sheets
- Exportar CSV para contador (con detalle de facturas por proveedor)

### Resumen *(manager / owner)*
- Vista por mes / quincena / semana
- Categorías separadas: Ingresos / Mercaderías / Sueldos / Operativos / Socios
- Desglose por proveedor en mercaderías
- Resultado del período (Ingresos − Egresos)
- Posición de efectivo al cierre
- Exportar Excel/CSV

### Config *(owner)*
- Modo de acceso: Cajero / Manager / Owner
- Gestión de categorías de movimientos (agregar / desactivar)
  - Incluye: Adelanto de salario, Salario pendiente, Propinas por tarjeta, Gastos médicos, etc.
- Conexión al backend (Google Sheets via Apps Script)
- Sincronización manual

---

## Niveles de acceso

| Modo | Secciones disponibles |
|---|---|
| **Cajero** | Solo Cierre de Turno |
| **Manager** | Turno + Movimientos + Pendientes + Resumen |
| **Owner** | Todo + Proveedores + Config |

El modo se configura desde la sección Config y se guarda en el navegador. La pantalla de cajero está diseñada para estar siempre abierta en la PC de caja.

---

## Cajas físicas

| Caja | Uso |
|---|---|
| Registradora | Cobro de ventas al cliente |
| Caja Proveedores | Efectivo destinado a pagos de proveedores |
| Caja Fuerte | Resguardo del excedente nocturno |
| Banco | Depósitos y transferencias |

---

## Arquitectura

```
Google Sheets (base de datos compartida)
    ├── dias, historico, anual, productos, comps, meta  ← Dashboard Ventas
    ├── turnos          ← cierres de turno diarios
    ├── movimientos     ← todos los movimientos de caja
    ├── proveedores_caja ← lista de proveedores
    └── categorias_caja  ← categorías dinámicas

Apps Script (API unificada v3.0)
    ├── Endpoints ventas  ← sin cambios
    └── Endpoints caja    ← getTurnos, getMovimientos, getProvCaja, getCatsCaja...

GitHub Repos
    ├── satori-dashboard  → ventas, mix, metas, análisis
    └── satori-caja       → esta app
```

Ambos repos apuntan al mismo Apps Script desplegado — mismo Google Sheet, misma URL de backend.

---

## Backend (Google Apps Script)

El archivo `satori_apps_script_v3.js` contiene el script completo (ventas + caja). Reemplaza el código existente en el proyecto Apps Script del restaurante.

**Hojas nuevas que crea automáticamente:**
- `turnos` — cierres de turno
- `movimientos` — todos los movimientos
- `proveedores_caja` — proveedores con ciclos de pago
- `categorias_caja` — categorías configurables (con defaults precargados)

**Endpoints nuevos (GET):**
- `getTurnos` — lista de turnos cerrados (filtrable por fecha)
- `getMovimientos` — movimientos (filtrable por fecha y proveedor)
- `getProvCaja` — proveedores activos e inactivos
- `getCatsCaja` — categorías de movimientos

**Endpoints nuevos (POST):**
- `saveTurno` — cierra un turno y sincroniza los pagos como movimientos
- `saveMovimiento` — guarda un movimiento manual
- `updateMovEstado` — marca como pagado / pendiente
- `deleteMovimiento` — elimina un movimiento
- `saveProvCaja` — crea o actualiza proveedor
- `deleteProvCaja` — desactiva proveedor (preserva historial)
- `saveCatCaja` — agrega categoría
- `deleteCatCaja` — desactiva categoría

---

## Cómo actualizar el Apps Script

1. Ir a [script.google.com](https://script.google.com) → abrir el proyecto Satori
2. Seleccionar todo el código → `Ctrl+A` → borrar
3. Pegar el contenido de `satori_apps_script_v3.js`
4. Guardar → `Ctrl+S`
5. **Deploy → Manage deployments → editar → New version → Deploy**
6. La misma URL sigue funcionando — no hay que cambiar nada en el dashboard de ventas

---

## Stack

- HTML + CSS + JavaScript vanilla (sin frameworks, sin dependencias)
- Google Apps Script (backend serverless)
- Google Sheets (base de datos)
- GitHub Pages (hosting estático)

---

## Relacionado

- [satori-dashboard](https://github.com/CachoSatori/satori-dashboard) — Dashboard de ventas, mix de productos, metas y análisis
