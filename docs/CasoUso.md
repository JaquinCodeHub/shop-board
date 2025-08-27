# 📋 Casos de Uso - Panel Administrativo E-commerce .NET Core

> **Comentarios:**  
> - Casos de uso específicos para panel administrativo de e-commerce con .NET Core MVC.
> - Roles soportados: `SuperAdmin`, `Admin`, `Manager`, `Employee` con permisos granulares.
> - Incluye flujos de gestión de productos, pedidos, inventario y reportes con auditoría completa.
> - Arquitectura monolítica con Entity Framework Core y Identity Framework.

---

## Caso de Uso: Autenticación y Autorización de Administrador

**Actor Principal:** Administrador  
**Controlador:** `AccountController`  
**Escenario Principal:**  
1. El administrador accede a `/Account/Login` desde el navegador.
2. El sistema muestra formulario de login con validación anti-CSRF.
3. Administrador ingresa email y contraseña, opcionalmente "Recordarme".
4. El sistema valida credenciales contra `Users` table con `PasswordHasher`.
5. Si es válido, genera JWT token con claims de usuario y permisos.
6. Establece cookie de autenticación y registra login en `user_sessions`.
7. Redirige a `/Dashboard` según el rol del usuario autenticado.
8. El dashboard carga widgets personalizados según permisos del usuario.

**Escenarios Alternativos:**  
- Si credenciales inválidas, muestra error y registra intento fallido.
- Si cuenta está desactivada, muestra mensaje específico.
- Si requiere cambio de contraseña, redirige a `/Account/ChangePassword`.
- Si sesión expira, redirige automáticamente al login.

**Restricciones:**  
- Rate limiting: máximo 5 intentos por IP en 15 minutos.
- Passwords deben cumplir política de seguridad configurable.
- Auditoría completa de todos los intentos de login.
- JWT expira en tiempo configurable (ej: 8 horas).

---

## Caso de Uso: Gestión CRUD de Productos

**Actor Principal:** Manager o Admin  
**Controlador:** `ProductsController` (Area: Products)  
**Escenario Principal:**  
1. Usuario accede a `/Products` con permiso `View.Products`.
2. Sistema muestra lista paginada con filtros (categoría, estado, stock).
3. Para crear producto, usuario hace clic en "Nuevo Producto".
4. Sistema valida permiso `Create.Products` y muestra formulario.
5. Usuario completa datos: nombre, descripción, SKU, precio, categorías.
6. Selecciona categorías desde dropdown jerárquico cargado vía AJAX.
7. Sube imágenes del producto con validación de tipos y tamaños.
8. Al enviar, sistema valida ModelState y unicidad de SKU.
9. Crea registro en `products` table y asocia categorías en `product_categories`.
10. Procesa imágenes, las guarda en storage y registra en `product_images`.
11. Registra acción en `audit_logs` con valores anteriores y nuevos.
12. Muestra mensaje de éxito y redirige a lista de productos.

**Escenarios Alternativos:**  
- Si SKU duplicado, muestra error específico con sugerencia.
- Si faltan campos requeridos, mantiene datos ingresados y muestra errores.
- Puede guardar como borrador para completar después.
- Puede clonar producto existente como base para nuevo.
- Importación masiva desde Excel con validación por lotes.

**Restricciones:**  
- Validación de permisos en cada acción del controlador.
- SKU debe ser único en toda la base de datos.
- Imágenes limitadas a 5MB cada una, máximo 10 por producto.
- Auditoría obligatoria de todas las operaciones CRUD.

---

## Caso de Uso: Gestión de Inventario y Stock

**Actor Principal:** Employee con permisos de inventario  
**Controlador:** `InventoryController` (Area: Products)  
**Escenario Principal:**  
1. Usuario accede a `/Products/Inventory` con permiso `View.Inventory`.
2. Sistema muestra dashboard de inventario con métricas clave.
3. Ve lista de productos con stock actual, mínimo y alertas.
4. Para ajustar stock, selecciona producto y hace clic "Ajustar Stock".
5. Sistema valida permiso `Update.Inventory` y muestra modal de ajuste.
6. Usuario especifica tipo de movimiento (Entrada, Salida, Ajuste).
7. Ingresa cantidad, razón del movimiento y referencia opcional.
8. Al confirmar, sistema valida que stock resultante no sea negativo.
9. Actualiza `stock_quantity` en `products` table atómicamente.
10. Registra movimiento en `inventory_movements` con stock antes/después.
11. Si stock queda bajo el mínimo, crea alerta en `stock_alerts`.
12. Envía notificación en tiempo real vía SignalR a usuarios autorizados.
13. Actualiza vista de inventario con nuevos valores vía AJAX.

**Escenarios Alternativos:**  
- Si movimiento causaría stock negativo, muestra error y sugiere alternativas.
- Puede hacer transferencias entre ubicaciones si están configuradas.
- Movimientos masivos desde archivo Excel con validación.
- Programar ajustes automáticos para fechas específicas.

**Restricciones:**  
- Transacciones atómicas para evitar inconsistencias de stock.
- Validación de permisos granular por tipo de movimiento.
- Auditoría completa con usuario, fecha, IP y razón.
- Notificaciones automáticas cuando stock < nivel mínimo.

---

## Caso de Uso: Procesamiento de Pedidos

**Actor Principal:** Admin o Manager  
**Controlador:** `OrdersController` (Area: Sales)  
**Escenario Principal:**  
1. Usuario accede a `/Sales/Orders` con permiso `View.Orders`.
2. Sistema muestra lista de pedidos con filtros por estado, fecha, cliente.
3. Para procesar pedido, usuario hace clic en pedido "Pendiente".
4. Sistema muestra detalles completos del pedido y timeline.
5. Usuario revisa items, verifica stock disponible en tiempo real.
6. Si hay stock suficiente, hace clic "Procesar Pedido".
7. Sistema valida permiso `Process.Orders` y stock de cada item.
8. Actualiza estado del pedido a "Procesando" en base de datos.
9. Reserva stock creando movimientos de inventario "OUT".
10. Actualiza stock disponible de cada producto automáticamente.
11. Genera número de factura y calcula totales con impuestos.
12. Registra usuario que procesó y timestamp en `processed_by_id`.
13. Envía notificación automática al cliente vía email.
14. Crea notificación interna para departamento de envíos.
15. Actualiza dashboard con métricas de ventas en tiempo real.

**Escenarios Alternativos:**  
- Si algún item no tiene stock, ofrece procesar parcialmente.
- Puede cambiar estado a "En espera" si falta información.
- Puede cancelar pedido con razón obligatoria si hay problemas.
- Procesar múltiples pedidos seleccionados en lote.

**Restricciones:**  
- Transacciones ACID para mantener consistencia orden-inventario.
- Solo usuarios con permisos específicos pueden cambiar estados.
- Auditoría completa de todos los cambios de estado.
- Validación de reglas de negocio antes de cada transición.

---

## Caso de Uso: Generación de Reportes Personalizados

**Actor Principal:** Manager o Admin  
**Controlador:** `ReportsController` (Area: Sales)  
**Escenario Principal:**  
1. Usuario accede a `/Sales/Reports` con permiso `View.Reports`.
2. Sistema muestra centro de reportes con plantillas disponibles.
3. Usuario selecciona "Reporte de Ventas" y hace clic "Configurar".
4. Sistema valida permiso `Generate.Reports` y muestra formulario.
5. Usuario configura parámetros: rango de fechas, productos, categorías.
6. Selecciona formato de salida (PDF, Excel, CSV) y opciones de gráficos.
7. Puede guardar configuración como plantilla para uso futuro.
8. Al generar, sistema valida fechas y crea registro en `reports` table.
9. Ejecuta consulta optimizada usando repositorio específico.
10. Procesa datos aplicando filtros y agregaciones necesarias.
11. Genera archivo usando servicio correspondiente (Excel/PDF).
12. Guarda archivo en storage seguro y actualiza ruta en BD.
13. Notifica al usuario cuando reporte está listo para descarga.
14. Proporciona enlace de descarga con expiración de 7 días.

**Escenarios Alternativos:**  
- Puede programar reportes automáticos (diario, semanal, mensual).
- Si reporte es muy grande, lo procesa en background job.
- Puede enviar reporte por email a lista de destinatarios.
- Permite exportar datos crudos para análisis en herramientas externas.

**Restricciones:**  
- Reportes grandes se procesan de forma asíncrona.
- Archivos generados se eliminan automáticamente después de 30 días.
- Límite de 10 reportes concurrentes por usuario.
- Permisos granulares para diferentes tipos de datos financieros.

---

## Caso de Uso: Gestión de Usuarios y Roles

**Actor Principal:** SuperAdmin  
**Controlador:** `UsersController` (Area: Admin)  
**Escenario Principal:**  
1. SuperAdmin accede a `/Admin/Users` con permiso `View.Users`.
2. Sistema muestra lista paginada de usuarios con roles asignados.
3. Para crear usuario, hace clic "Nuevo Usuario" con permiso `Create.Users`.
4. Completa formulario: email, nombre, teléfono, roles iniciales.
5. Sistema valida unicidad de email y genera contraseña temporal.
6. Crea usuario en `users` table con estado `Pending`.
7. Asigna roles especificados en `user_roles` table.
8. Envía email de bienvenida con instrucciones de primer login.
9. Registra acción de creación en `audit_logs` con detalles completos.
10. Nuevo usuario debe cambiar contraseña en primer acceso.
11. Al cambiar contraseña, estado cambia a `Active` automáticamente.

**Para gestión de roles:**  
1. SuperAdmin accede a `/Admin/Roles` con permiso `View.Roles`.
2. Puede crear roles personalizados con combinación de permisos.
3. Asigna permisos granulares por módulo (Products, Sales, etc.).
4. Los roles del sistema (SuperAdmin, Admin) no son editables.
5. Puede clonar rol existente como base para nuevo rol.

**Escenarios Alternativos:**  
- Puede importar usuarios masivamente desde Excel.
- Desactivar usuarios en lugar de eliminarlos (soft delete).
- Transferir ownership de datos entre usuarios.
- Programar desactivación automática de usuarios temporales.

**Restricciones:**  
- Solo SuperAdmin puede crear otros SuperAdmins.
- Usuarios no pueden modificar sus propios permisos.
- Auditoría obligatoria de todos los cambios de roles.
- Validación de integridad referencial antes de eliminar usuarios.

---

## Caso de Uso: Monitoreo de Alertas y Notificaciones

**Actor Principal:** Cualquier usuario autenticado  
**Controlador:** `NotificationsController` y SignalR Hub  
**Escenario Principal:**  
1. Sistema detecta evento que requiere notificación (stock bajo, nuevo pedido).
2. `NotificationService` evalúa reglas configuradas para el evento.
3. Determina usuarios que deben recibir notificación según permisos.
4. Crea registro en `notifications` table con tipo y datos del evento.
5. Envía notificación en tiempo real vía SignalR a usuarios conectados.
6. Actualiza contador de notificaciones no leídas en UI.
7. Si usuario no está conectado, queda pendiente para próximo login.
8. Usuario hace clic en campana de notificaciones para ver lista.
9. Sistema marca notificaciones como leídas al visualizarlas.
10. Usuario puede configurar preferencias de notificación.

**Tipos de notificaciones automáticas:**  
- Stock bajo: cuando producto < nivel mínimo configurado.
- Nuevo pedido: notifica a departamento de ventas.
- Pago recibido: actualiza estado y notifica contabilidad.
- Error del sistema: alerta a administradores técnicos.
- Reportes listos: notifica cuando generación termina.

**Escenarios Alternativos:**  
- Puede silenciar notificaciones por período específico.
- Envío por email para notificaciones críticas.
- Integración con sistemas externos (Slack, Teams).
- Escalamiento automático si notificaciones no se atienden.

**Restricciones:**  
- Rate limiting para evitar spam de notificaciones.
- Persistencia de notificaciones críticas por 30 días.
- Cifrado de notificaciones con datos sensibles.
- Opt-out obligatorio para notificaciones no esenciales.

---

## Caso de Uso: Auditoría y Trazabilidad de Acciones

**Actor Principal:** Admin o Auditor  
**Controlador:** `AuditController` (Area: Admin)  
**Escenario Principal:**  
1. Admin accede a `/Admin/System/AuditTrail` con permiso `View.AuditLogs`.
2. Sistema muestra interfaz de auditoría con filtros avanzados.
3. Puede filtrar por usuario, entidad, acción, rango de fechas.
4. Para cada acción registrada, muestra valores antes/después.
5. Incluye metadata: IP, user agent, timestamp, contexto.
6. Puede hacer drill-down en cambios específicos de entidades.
7. Exporta logs de auditoría en formato seguro para compliance.
8. Sistema mantiene integridad de logs (no modificables).

**Eventos auditados automáticamente:**  
- Todos los cambios CRUD en entidades principales.
- Cambios de permisos y roles de usuarios.
- Accesos y cambios en configuraciones del sistema.
- Operaciones de inventario y movimientos de stock.
- Generación y descarga de reportes financieros.

**Escenarios Alternativos:**  
- Puede configurar retención de logs por tipo de evento.
- Integración con sistemas SIEM externos.
- Alertas automáticas para patrones sospechosos.
- Backup automático de logs críticos a almacenamiento externo.

**Restricciones:**  
- Logs de auditoría son inmutables una vez creados.
- Encriptación de datos sensibles en logs.
- Retención mínima de 7 años para datos financieros.
- Acceso a auditoría requiere autenticación multifactor.

---

## Caso de Uso: Dashboard Personalizable y Métricas

**Actor Principal:** Cualquier usuario autenticado  
**Controlador:** `DashboardController`  
**Escenario Principal:**  
1. Usuario accede a `/Dashboard` después de login exitoso.
2. Sistema carga configuración de widgets guardada en `dashboard_widgets`.
3. Para cada widget, valida permisos antes de mostrar datos.
4. Carga métricas en tiempo real usando AJAX y caché Redis.
5. Muestra gráficos interactivos con Chart.js o similar.
6. Usuario puede arrastrar y redimensionar widgets.
7. Cambios se guardan automáticamente vía AJAX.
8. Puede agregar nuevos widgets desde galería disponible.
9. Configura parámetros de cada widget (período, filtros).

**Widgets disponibles según permisos:**  
- Ventas del día/mes (requiere `View.Sales`).
- Productos con stock bajo (requiere `View.Inventory`).
- Pedidos pendientes (requiere `View.Orders`).
- Top productos vendidos (requiere `View.Products`).
- Nuevos clientes registrados (requiere `View.Customers`).
- Alertas del sistema (requiere permisos específicos).

**Escenarios Alternativos:**  
- Puede crear dashboards temáticos (ventas, inventario, etc.).
- Compartir configuración de dashboard con otros usuarios.
- Exportar dashboard como PDF para reportes ejecutivos.
- Alertas automáticas cuando métricas superan umbrales.

**Restricciones:**  
- Widgets solo muestran datos según permisos del usuario.
- Actualización automática cada 30 segundos sin recargar página.
- Máximo 12 widgets por dashboard para mantener rendimiento.
- Caché inteligente para optimizar consultas frecuentes.

---

## Buenas Prácticas y Consideraciones Técnicas

- **Arquitectura MVC:** Separación clara de responsabilidades con Entity Framework Core.
- **Dependency Injection:** Servicios registrados en contenedor IoC de .NET Core.
- **Unit of Work Pattern:** Transacciones atómicas para operaciones complejas.
- **Repository Pattern:** Abstracción de acceso a datos con interfaces.
- **DTO/ViewModel Pattern:** Separación entre modelos de dominio y presentación.
- **Authorization Policies:** Autorización basada en políticas y claims.
- **Audit Trail:** Registro automático de cambios con Entity Framework interceptors.
- **Caching Strategy:** Redis para sesiones y caché, memoria para datos estáticos.
- **Background Jobs:** Hangfire para procesamiento asíncrono de reportes.
- **Validation:** Data Annotations + FluentValidation para reglas complejas.
- **Security:** HTTPS obligatorio, CSRF protection, input sanitization.
- **Performance:** Lazy loading, paginación, índices optimizados.
- **Monitoring:** Serilog para logs estructurados, Application Insights.
- **Error Handling:** Global exception handling con logging detallado.
- **Testing:** Unit tests con xUnit, integration tests con TestServer.

---