# Plan de Desarrollo del DEMO — Taller Mecánico SaaS

**Proyecto:** Sistema de Control de Servicios y Mantenimiento Preventivo de Vehículos  
**Etapa:** DEMO previo al MVP  
**Equipo:** 3 personas  
**Stack:** Next.js + NestJS + Prisma + PostgreSQL  
**Entorno:** Debian 13 local  
**Dominio:** `taller.gt`

## 1. Objetivo del DEMO

El DEMO debe validar el flujo principal antes de ampliar el sistema al MVP:

```text
Tenant
→ Cliente
→ Vehículo
→ Orden de servicio
→ Kilometraje real
→ Servicio preventivo
→ Próximo mantenimiento
→ Dashboard / alerta
→ Segunda visita
→ Nuevo kilometraje real
→ Recalibración
```

El objetivo no es completar todas las funciones finales, sino demostrar que la arquitectura, el modelo de datos y las reglas centrales funcionan de punta a punta.

## 2. Alcance incluido

El DEMO incluirá:

- 2 tenants de prueba;
- 2 sucursales por tenant;
- usuarios por rol;
- autenticación básica;
- aislamiento multi-tenant;
- RLS básico;
- clientes;
- empresas;
- vehículos;
- historial de propietarios;
- catálogo de servicios;
- órdenes de servicio;
- mecánico responsable;
- kilometraje real de ingreso;
- historial de lecturas;
- servicios realizados;
- cálculo del próximo mantenimiento;
- estados PENDIENTE, PROXIMO, VENCIDO, ATENDIDO y CANCELADO;
- recalibración 60/40;
- dashboard básico;
- datos seed;
- pruebas esenciales.

## 3. Fuera del DEMO

Se pospone para el MVP:

- WhatsApp real;
- Evolution API;
- SMTP real;
- pg-boss completo;
- reintentos de notificaciones;
- idempotencia avanzada;
- MFA;
- recuperación de contraseña;
- proceso comercial SaaS completo;
- planes y downgrade;
- cancelación y retención de tenants;
- reportes PDF/CSV;
- panel técnico de soporte;
- auditoría completa;
- backups automatizados;
- restore mensual;
- CI/CD completo;
- despliegues versionados y rollback;
- hardening final.

La arquitectura debe quedar preparada para agregar estas funciones sin rediseñar el núcleo.

## 4. Flujo que debe funcionar en la demostración

1. Abrir un tenant por subdominio.
2. Iniciar sesión.
3. Seleccionar sucursal.
4. Crear cliente.
5. Crear vehículo.
6. Crear orden.
7. Registrar kilometraje real.
8. Agregar servicio preventivo.
9. Asignar mecánico.
10. Cambiar a `EN_PROCESO`.
11. Marcar servicio `REALIZADO`.
12. Cambiar a `TERMINADO`.
13. Generar próximo mantenimiento.
14. Mostrarlo en dashboard.
15. Registrar una segunda visita.
16. Ingresar kilometraje real actualizado.
17. Calcular `km_dia_real`.
18. Recalibrar `km_dia_estimado`.
19. Recalcular próxima fecha.
20. Mostrar el nuevo estado/proyección.

# 5. División del trabajo entre 3 personas

## Persona 1 — Backend, autenticación y multi-tenancy

### Responsabilidad

Construir la base técnica del backend y garantizar el aislamiento entre tenants.

### Tareas

- crear `apps/api`;
- configurar NestJS;
- integrar Prisma;
- configurar manejo global de errores;
- generar `requestId`;
- estructurar `/api/v1`;
- resolver tenant por subdominio;
- fijar `tenant_id` desde sesión;
- implementar RLS básico;
- usar `SET LOCAL` por transacción;
- validar sucursal activa;
- implementar login;
- Argon2id;
- JWT Access Token;
- Refresh Token;
- sesiones persistentes;
- logout;
- RBAC;
- Guards y Decorators;
- permisos por rol;
- usuarios de tenant;
- asignación de sucursales;
- endpoints de auth, usuarios y sucursales;
- Swagger básico.

### Roles del DEMO

- ADMINISTRADOR;
- SUPERVISOR;
- RECEPCION;
- MECANICO.

### Entregables

- API ejecutable;
- login;
- sesión;
- resolución de tenant;
- aislamiento RLS;
- permisos;
- Swagger;
- pruebas de aislamiento.

---

## Persona 2 — Base de datos, dominio y mantenimiento

### Responsabilidad

Construir el modelo de datos y las reglas centrales.

### Tareas de datos

Crear entidades para:

- tenants;
- branches;
- tenant_users;
- user_branches;
- customers;
- vehicles;
- vehicle_owner_history;
- service_catalog;
- work_orders;
- work_order_services;
- odometer_readings;
- maintenance_projections.

### Clientes

- persona;
- empresa;
- documento opcional para persona;
- NIT obligatorio para empresa;
- teléfono/correo;
- deshabilitar sin borrar.

### Vehículos

- placa obligatoria;
- `placa_normalizada`;
- unicidad por tenant;
- tipo;
- marca;
- modelo;
- año;
- propietario actual;
- historial de propietarios.

### Órdenes

- UUID v7;
- correlativo por sucursal/año;
- sucursal fija;
- kilometraje obligatorio;
- mecánico;
- estados:
  - RECIBIDO;
  - EN_PROCESO;
  - TERMINADO;
  - ENTREGADO;
  - ANULADO.

### Servicios de orden

- PENDIENTE;
- REALIZADO;
- NO_REALIZADO;
- máximo una vez por orden.

### Kilometraje

- conservar todas las lecturas;
- impedir kilometraje menor;
- permitir varias lecturas el mismo día;
- comparar contra última lectura válida de una fecha anterior;
- calcular uso real.

### Mantenimiento

```text
km_objetivo =
km_actual + km_recomendado
```

```text
dias_por_km =
km_recomendado / km_dia_estimado
```

```text
fecha_por_km =
fecha_servicio + CEIL(dias_por_km)
```

```text
fecha_por_tiempo =
fecha_servicio + dias_recomendados
```

```text
fecha_estimada =
MIN(fecha_por_km, fecha_por_tiempo)
```

### Recalibración

```text
km_dia_estimado_nuevo =
0.60 × km_dia_real
+
0.40 × km_dia_estimado_anterior
```

### Estados de mantenimiento

- PENDIENTE;
- PROXIMO;
- VENCIDO;
- ATENDIDO;
- CANCELADO.

### Entregables

- `schema.prisma`;
- migraciones;
- constraints;
- lógica de dominio;
- cálculo de mantenimiento;
- recalibración;
- pruebas unitarias;
- seed inicial.

---

## Persona 3 — Frontend, UX e integración

### Responsabilidad

Construir el portal del taller y conectar el flujo completo con la API.

### Tareas

Crear `apps/web`.

### Base UI

- Next.js;
- Tailwind;
- shadcn/ui;
- React Hook Form;
- Zod;
- responsive desktop/tablet/mobile.

### Pantallas

#### Login

- acceso por tenant;
- manejo de errores;
- sesión.

#### Dashboard

Mostrar:

- órdenes recibidas;
- órdenes en proceso;
- mantenimientos próximos;
- vencidos;
- vehículos atendidos;
- últimos ingresos.

#### Clientes

- listado;
- búsqueda;
- crear;
- editar;
- deshabilitar;
- detalle.

#### Vehículos

- listado;
- búsqueda por placa;
- crear;
- editar;
- propietario;
- historial.

#### Servicios

- catálogo;
- crear;
- editar;
- activar/desactivar.

#### Órdenes

- crear;
- seleccionar cliente;
- seleccionar vehículo;
- ingresar kilometraje;
- seleccionar servicios;
- asignar mecánico;
- actualizar estado;
- marcar servicio realizado/no realizado;
- observaciones;
- terminar.

#### Mantenimiento

- próximos;
- vencidos;
- detalle;
- fecha estimada;
- km objetivo;
- datos usados para el cálculo;
- resultado de recalibración.

### Entregables

- portal funcional;
- flujo completo integrado;
- dashboard;
- CRUD necesarios;
- UI estable para presentación.

# 6. Trabajo compartido

Los tres deben participar en:

- revisión del modelo;
- contratos API;
- revisión de Pull Requests;
- integración;
- pruebas;
- resolución de errores;
- documentación;
- ensayo de la demostración.

Todo módulo crítico debe ser revisado por al menos una segunda persona.

# 7. Sprint DEMO 1 — Fundación

## Persona 1

- NestJS;
- auth;
- tenant resolution;
- RLS;
- usuarios;
- roles;
- sesiones.

## Persona 2

- Prisma;
- tenants;
- sucursales;
- usuarios;
- clientes;
- vehículos;
- migraciones.

## Persona 3

- Next.js;
- layout;
- login;
- navegación;
- dashboard base;
- clientes;
- vehículos.

## Resultado obligatorio

```text
Login
→ Tenant
→ Sucursal
→ Cliente
→ Vehículo
```

# 8. Sprint DEMO 2 — Órdenes y servicios

## Persona 1

- Guards;
- permisos;
- APIs de órdenes;
- validación de sucursal activa;
- estados autorizados.

## Persona 2

- órdenes;
- detalles;
- kilometraje;
- correlativos;
- mecánico;
- constraints;
- catálogo.

## Persona 3

- interfaz de órdenes;
- cliente/vehículo;
- kilometraje;
- servicios;
- mecánico;
- estados;
- observaciones.

## Resultado obligatorio

```text
Cliente
→ Vehículo
→ Orden
→ Kilometraje
→ Servicio
→ Mecánico
→ EN_PROCESO
→ TERMINADO
```

# 9. Sprint DEMO 3 — Mantenimiento preventivo

## Persona 1

- endpoints de mantenimiento;
- filtros;
- permisos;
- endpoint de dashboard.

## Persona 2

- cálculo;
- km objetivo;
- fecha estimada;
- recalibración;
- estados;
- único mantenimiento activo;
- pruebas.

## Persona 3

- dashboard;
- próximos;
- vencidos;
- historial;
- detalle del mantenimiento;
- visualización del recálculo.

## Resultado obligatorio

```text
Servicio terminado
→ Próximo mantenimiento
→ Dashboard
→ Nueva visita
→ Nuevo kilometraje
→ Recalibración
→ Nueva fecha
```

# 10. Dependencias

## Persona 1 → Persona 3

Debe entregar temprano:

- auth;
- contratos;
- permisos;
- formato de errores.

## Persona 2 → Persona 3

Debe entregar temprano:

- entidades;
- DTO;
- reglas;
- endpoints de dominio.

## Persona 1 ↔ Persona 2

Deben coordinar:

- RLS;
- tenant context;
- transacciones;
- constraints;
- estados.

# 11. Contratos a fijar primero

Definir DTO/esquemas para:

- Login;
- Usuario;
- Sucursal;
- Cliente;
- Vehículo;
- Servicio;
- Orden;
- DetalleOrden;
- LecturaKilometraje;
- Mantenimiento;
- DashboardSummary.

Ubicación recomendada:

```text
packages/shared
```

# 12. Git

`main` protegida.

Ramas cortas:

```text
feature/auth
feature/tenants
feature/customers
feature/vehicles
feature/orders
feature/maintenance
feature/dashboard
```

Reglas:

- Pull Request obligatorio;
- no merge directo a `main`;
- revisión de otra persona;
- commits pequeños;
- lint/test/build antes del merge.

# 13. Definition of Done

Una tarea está terminada cuando:

- compila;
- pasa lint;
- valida entrada;
- respeta tenant;
- respeta permisos;
- maneja errores;
- no rompe migraciones;
- tiene prueba cuando corresponde;
- fue revisada;
- está integrada en `main`.

# 14. Pruebas mínimas

## Unitarias

- cálculo de mantenimiento;
- recalibración 60/40;
- estados;
- kilometraje menor;
- permisos principales.

## Integración

- tenant A no ve tenant B;
- placa única por tenant;
- kilometraje menor rechazado;
- orden terminada genera mantenimiento;
- solo un mantenimiento activo por vehículo + servicio;
- nueva lectura recalcula.

## E2E

```text
Login
→ crear cliente
→ crear vehículo
→ crear orden
→ ingresar km
→ realizar servicio
→ terminar orden
→ ver mantenimiento
→ segunda visita
→ recalibrar
```

# 15. Datos seed

Crear:

```bash
pnpm db:seed
```

Debe generar:

## Tenant 1

```text
Taller Central
├── CENTRAL
└── NORTE
```

## Tenant 2

```text
Moto Servicio GT
├── ZONA1
└── ZONA10
```

Cada tenant debe incluir:

- ADMINISTRADOR;
- SUPERVISOR;
- RECEPCION;
- MECANICO;
- al menos 5 clientes;
- al menos 8 vehículos;
- catálogo de servicios;
- órdenes;
- próximos;
- vencidos.

# 16. Guion del DEMO

## Paso 1 — Login

Ingresar a:

```text
taller-central.taller.gt
```

## Paso 2 — Dashboard

Mostrar:

- órdenes;
- próximos;
- vencidos.

## Paso 3 — Cliente

Crear un cliente.

## Paso 4 — Vehículo

Registrar vehículo.

## Paso 5 — Orden

Crear orden e ingresar kilometraje real.

Agregar:

```text
Cambio de aceite
```

Asignar mecánico.

## Paso 6 — Proceso

```text
RECIBIDO
→ EN_PROCESO
```

Marcar:

```text
REALIZADO
```

## Paso 7 — Terminar

```text
EN_PROCESO
→ TERMINADO
```

Mostrar:

- km objetivo;
- fecha estimada;
- mantenimiento generado.

## Paso 8 — Segunda visita

Ingresar un kilometraje real mayor.

Mostrar:

```text
km_dia_real
km_dia_estimado_anterior
km_dia_estimado_nuevo
fecha_estimada_anterior
fecha_estimada_nueva
```

## Paso 9 — Aislamiento

Entrar al segundo tenant y demostrar que no puede consultar información del primero.

# 17. Checklist del DEMO

## Infraestructura

- [ ] Debian 13
- [ ] PostgreSQL
- [ ] Node.js
- [ ] pnpm
- [ ] Caddy
- [ ] DNS `taller.gt`
- [ ] wildcard DNS
- [ ] systemd básico

## Backend

- [ ] NestJS
- [ ] Prisma
- [ ] tenant resolution
- [ ] RLS
- [ ] auth
- [ ] roles
- [ ] sesiones
- [ ] clientes
- [ ] vehículos
- [ ] catálogo
- [ ] órdenes
- [ ] kilometraje
- [ ] mantenimiento

## Frontend

- [ ] login
- [ ] dashboard
- [ ] clientes
- [ ] vehículos
- [ ] servicios
- [ ] órdenes
- [ ] mantenimientos
- [ ] responsive

## Datos

- [ ] tenant 1
- [ ] tenant 2
- [ ] 4 sucursales
- [ ] usuarios
- [ ] clientes
- [ ] vehículos
- [ ] órdenes
- [ ] próximos
- [ ] vencidos

## Pruebas

- [ ] cálculo
- [ ] RLS
- [ ] kilometraje
- [ ] mantenimiento
- [ ] E2E principal

## Presentación

- [ ] guion probado
- [ ] datos controlados
- [ ] reset de demo
- [ ] los 3 integrantes conocen el flujo

# 18. Condición de cierre del DEMO

El DEMO está listo cuando:

1. existen dos tenants aislados;
2. cada tenant tiene sucursales;
3. usuarios pueden autenticarse;
4. roles restringen acciones;
5. clientes y vehículos pueden registrarse;
6. cada ingreso registra kilometraje real;
7. una orden contiene servicios;
8. existe mecánico responsable;
9. una orden puede finalizar;
10. el sistema genera próximo mantenimiento;
11. muestra próximos y vencidos;
12. una segunda visita recalibra la estimación;
13. conserva historial;
14. el dashboard refleja la operación;
15. el flujo puede demostrarse sin modificar datos manualmente.

# 19. Después del DEMO

La siguiente fase será el MVP:

```text
WhatsApp real
→ SMTP
→ pg-boss
→ recordatorios
→ retries
→ idempotencia
→ consentimiento
→ SaaS comercial
→ planes
→ soporte
→ reportes
→ auditoría completa
→ backups
→ hardening
→ release final
```

# 20. Regla principal de alcance

Durante el DEMO no se incorporará una funcionalidad del MVP si retrasa el flujo:

```text
Cliente
→ Vehículo
→ Orden
→ Servicio
→ Mantenimiento
→ Recalibración
```

Ese flujo es la prioridad absoluta.
