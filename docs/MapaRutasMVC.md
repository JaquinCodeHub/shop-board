# 🛣️ Mapa de rutas MVC .NET Core - Panel Administrativo E-commerce

> **Comentarios:**  
> - Este mapa describe las rutas MVC para el panel administrativo de e-commerce con .NET Core.
> - Arquitectura monolítica con separación por Areas, Controllers y Actions.
> - Roles soportados: `SuperAdmin`, `Admin`, `Manager`, `Employee` (usuarios autenticados).
> - Incluye rutas de vistas (GET) y acciones (POST/PUT/DELETE) con validación de permisos.

---

## Estructura de Areas y Controllers

```csharp
Areas/
├── Admin/
│   ├── Controllers/
│   │   ├── DashboardController.cs
│   │   ├── UsersController.cs
│   │   ├── RolesController.cs
│   │   └── SystemController.cs
│   └── Views/
├── Products/
│   ├── Controllers/
│   │   ├── ProductsController.cs
│   │   ├── CategoriesController.cs
│   │   └── InventoryController.cs
├── Sales/
│   ├── Controllers/
│   │   ├── OrdersController.cs
│   │   ├── CustomersController.cs
│   │   └── ReportsController.cs
└── Controllers/ (Root)
    ├── HomeController.cs
    ├── AccountController.cs
    └── DashboardController.cs
```

---

## Rutas de Autenticación y Cuenta

| Método | Ruta                           | Action                    | Descripción                                  | Permisos      |
|--------|--------------------------------|---------------------------|----------------------------------------------|---------------|
| GET    | `/Account/Login`               | Login()                   | Página de inicio de sesión                   | Público       |
| POST   | `/Account/Login`               | Login(LoginViewModel)     | Procesar login con JWT                       | Público       |
| GET    | `/Account/ForgotPassword`      | ForgotPassword()          | Formulario recuperar contraseña              | Público       |
| POST   | `/Account/ForgotPassword`      | ForgotPassword(model)     | Enviar email de recuperación                 | Público       |
| GET    | `/Account/ResetPassword`       | ResetPassword(token)      | Formulario restablecer contraseña            | Público       |
| POST   | `/Account/ResetPassword`       | ResetPassword(model)      | Procesar nueva contraseña                    | Público       |
| POST   | `/Account/Logout`              | Logout()                  | Cerrar sesión y limpiar JWT                  | Autenticado   |
| GET    | `/Account/Profile`             | Profile()                 | Ver perfil del usuario actual                | Autenticado   |
| POST   | `/Account/Profile`             | Profile(ProfileViewModel) | Actualizar perfil del usuario                | Autenticado   |
| GET    | `/Account/ChangePassword`      | ChangePassword()          | Formulario cambiar contraseña                | Autenticado   |
| POST   | `/Account/ChangePassword`      | ChangePassword(model)     | Procesar cambio de contraseña                | Autenticado   |
| GET    | `/Account/AccessDenied`        | AccessDenied()            | Página de acceso denegado                    | Público       |

---

## Rutas del Dashboard Principal

| Método | Ruta                           | Action                    | Descripción                                  | Permisos      |
|--------|--------------------------------|---------------------------|----------------------------------------------|---------------|
| GET    | `/`                            | Index()                   | Dashboard principal según rol                | Autenticado   |
| GET    | `/Dashboard`                   | Index()                   | Dashboard principal                          | Autenticado   |
| GET    | `/Dashboard/Stats`             | GetStats()                | Estadísticas del dashboard (AJAX)           | Autenticado   |
| GET    | `/Dashboard/SalesChart`        | GetSalesChart(period)     | Datos para gráfico de ventas (JSON)         | View.Sales    |
| GET    | `/Dashboard/TopProducts`       | GetTopProducts(count)     | Productos más vendidos (JSON)               | View.Products |
| GET    | `/Dashboard/RecentOrders`      | GetRecentOrders(count)    | Pedidos recientes (JSON)                     | View.Orders   |
| GET    | `/Dashboard/LowStockAlert`     | GetLowStockAlert()        | Alertas de stock bajo (JSON)                | View.Inventory|
| POST   | `/Dashboard/SaveWidget`        | SaveWidget(widgetConfig)  | Guardar configuración de widget             | Autenticado   |
| DELETE | `/Dashboard/RemoveWidget/{id}` | RemoveWidget(id)          | Eliminar widget del dashboard                | Autenticado   |

---

## Area: Products - Gestión de Productos

| Método | Ruta                                    | Action                         | Descripción                                  | Permisos         |
|--------|-----------------------------------------|--------------------------------|----------------------------------------------|------------------|
| GET    | `/Products`                             | Index(filter)                  | Lista paginada de productos                  | View.Products    |
| GET    | `/Products/Details/{id}`                | Details(id)                    | Ver detalles del producto                    | View.Products    |
| GET    | `/Products/Create`                      | Create()                       | Formulario crear producto                    | Create.Products  |
| POST   | `/Products/Create`                      | Create(CreateProductViewModel) | Procesar creación de producto                | Create.Products  |
| GET    | `/Products/Edit/{id}`                   | Edit(id)                       | Formulario editar producto                   | Update.Products  |
| POST   | `/Products/Edit/{id}`                   | Edit(id, UpdateProductViewModel)| Procesar edición de producto                 | Update.Products  |
| GET    | `/Products/Delete/{id}`                 | Delete(id)                     | Confirmar eliminación de producto           | Delete.Products  |
| POST   | `/Products/Delete/{id}`                 | DeleteConfirmed(id)            | Procesar eliminación de producto            | Delete.Products  |
| GET    | `/Products/BulkEdit`                    | BulkEdit()                     | Formulario edición masiva                    | Update.Products  |
| POST   | `/Products/BulkEdit`                    | BulkEdit(BulkEditViewModel)    | Procesar edición masiva                      | Update.Products  |
| GET    | `/Products/Import`                      | Import()                       | Formulario importar productos                | Create.Products  |
| POST   | `/Products/Import`                      | Import(ImportViewModel)        | Procesar importación desde Excel            | Create.Products  |
| GET    | `/Products/Export`                      | Export(filter)                 | Exportar productos a Excel                   | Export.Products  |
| POST   | `/Products/UpdateStock/{id}`            | UpdateStock(id, quantity)      | Actualizar stock del producto (AJAX)        | Update.Inventory |
| GET    | `/Products/Search`                      | Search(term)                   | Búsqueda de productos (AJAX)                 | View.Products    |
| POST   | `/Products/ToggleFeatured/{id}`         | ToggleFeatured(id)             | Marcar/desmarcar como destacado             | Update.Products  |
| POST   | `/Products/ToggleActive/{id}`           | ToggleActive(id)               | Activar/desactivar producto                  | Update.Products  |

### Subcontroller: ProductImages

| Método | Ruta                                    | Action                         | Descripción                                  | Permisos         |
|--------|-----------------------------------------|--------------------------------|----------------------------------------------|------------------|
| POST   | `/Products/{id}/Images/Upload`          | UploadImage(id, file)          | Subir imagen del producto                    | Update.Products  |
| DELETE | `/Products/{id}/Images/{imageId}`       | DeleteImage(id, imageId)       | Eliminar imagen del producto                 | Update.Products  |
| POST   | `/Products/{id}/Images/{imageId}/Primary`| SetPrimaryImage(id, imageId)   | Establecer imagen principal                  | Update.Products  |
| POST   | `/Products/{id}/Images/Reorder`         | ReorderImages(id, order)       | Reordenar imágenes                           | Update.Products  |

---

## Area: Products - Gestión de Categorías

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Products/Categories`                  | Index()                             | Lista jerárquica de categorías               | View.Categories  |
| GET    | `/Products/Categories/Details/{id}`     | Details(id)                         | Ver detalles de la categoría                 | View.Categories  |
| GET    | `/Products/Categories/Create`           | Create(parentId?)                   | Formulario crear categoría                   | Create.Categories|
| POST   | `/Products/Categories/Create`           | Create(CreateCategoryViewModel)     | Procesar creación de categoría               | Create.Categories|
| GET    | `/Products/Categories/Edit/{id}`        | Edit(id)                            | Formulario editar categoría                  | Update.Categories|
| POST   | `/Products/Categories/Edit/{id}`        | Edit(id, UpdateCategoryViewModel)   | Procesar edición de categoría                | Update.Categories|
| POST   | `/Products/Categories/Delete/{id}`      | Delete(id)                          | Eliminar categoría (AJAX)                    | Delete.Categories|
| POST   | `/Products/Categories/Reorder`          | Reorder(ReorderViewModel)           | Reordenar categorías                         | Update.Categories|
| GET    | `/Products/Categories/Tree`             | GetCategoryTree()                   | Árbol de categorías (JSON)                   | View.Categories  |

---

## Area: Products - Gestión de Inventario

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Products/Inventory`                   | Index(filter)                       | Lista de inventario con stock                | View.Inventory   |
| GET    | `/Products/Inventory/Movements`         | Movements(filter)                   | Historial de movimientos de inventario       | View.Inventory   |
| GET    | `/Products/Inventory/LowStock`          | LowStock()                          | Productos con stock bajo                     | View.Inventory   |
| GET    | `/Products/Inventory/Adjust/{id}`       | AdjustStock(id)                     | Formulario ajustar stock                     | Update.Inventory |
| POST   | `/Products/Inventory/Adjust/{id}`       | AdjustStock(id, AdjustStockViewModel)| Procesar ajuste de stock                     | Update.Inventory |
| GET    | `/Products/Inventory/Transfer`          | Transfer()                          | Formulario transferir stock                  | Update.Inventory |
| POST   | `/Products/Inventory/Transfer`          | Transfer(TransferViewModel)         | Procesar transferencia de stock              | Update.Inventory |
| GET    | `/Products/Inventory/Alerts`            | Alerts()                            | Configurar alertas de stock                  | Update.Inventory |
| POST   | `/Products/Inventory/Alerts`            | SaveAlerts(AlertsViewModel)         | Guardar configuración de alertas            | Update.Inventory |
| GET    | `/Products/Inventory/Report`            | InventoryReport(filter)             | Reporte de inventario                        | Export.Inventory |

---

## Area: Sales - Gestión de Pedidos

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Sales/Orders`                         | Index(filter)                       | Lista paginada de pedidos                    | View.Orders      |
| GET    | `/Sales/Orders/Details/{id}`            | Details(id)                         | Ver detalles del pedido                      | View.Orders      |
| GET    | `/Sales/Orders/Create`                  | Create()                            | Formulario crear pedido manual               | Create.Orders    |
| POST   | `/Sales/Orders/Create`                  | Create(CreateOrderViewModel)        | Procesar creación de pedido                  | Create.Orders    |
| GET    | `/Sales/Orders/Edit/{id}`               | Edit(id)                            | Formulario editar pedido                     | Update.Orders    |
| POST   | `/Sales/Orders/Edit/{id}`               | Edit(id, UpdateOrderViewModel)      | Procesar edición de pedido                   | Update.Orders    |
| POST   | `/Sales/Orders/UpdateStatus/{id}`       | UpdateStatus(id, status)            | Cambiar estado del pedido (AJAX)             | Update.Orders    |
| POST   | `/Sales/Orders/Process/{id}`            | ProcessOrder(id)                    | Procesar pedido                              | Process.Orders   |
| POST   | `/Sales/Orders/Ship/{id}`               | ShipOrder(id, ShipViewModel)        | Marcar como enviado                          | Ship.Orders      |
| POST   | `/Sales/Orders/Cancel/{id}`             | CancelOrder(id, reason)             | Cancelar pedido                              | Cancel.Orders    |
| GET    | `/Sales/Orders/Print/{id}`              | PrintOrder(id)                      | Imprimir factura/pedido                      | View.Orders      |
| GET    | `/Sales/Orders/Invoice/{id}`            | GenerateInvoice(id)                 | Generar factura PDF                          | View.Orders      |
| GET    | `/Sales/Orders/Export`                  | Export(filter)                      | Exportar pedidos a Excel                     | Export.Orders    |

---

## Area: Sales - Gestión de Clientes

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Sales/Customers`                      | Index(filter)                       | Lista paginada de clientes                   | View.Customers   |
| GET    | `/Sales/Customers/Details/{id}`         | Details(id)                         | Ver perfil completo del cliente              | View.Customers   |
| GET    | `/Sales/Customers/Create`               | Create()                            | Formulario crear cliente                     | Create.Customers |
| POST   | `/Sales/Customers/Create`               | Create(CreateCustomerViewModel)     | Procesar creación de cliente                 | Create.Customers |
| GET    | `/Sales/Customers/Edit/{id}`            | Edit(id)                            | Formulario editar cliente                    | Update.Customers |
| POST   | `/Sales/Customers/Edit/{id}`            | Edit(id, UpdateCustomerViewModel)   | Procesar edición de cliente                  | Update.Customers |
| POST   | `/Sales/Customers/Delete/{id}`          | Delete(id)                          | Eliminar cliente (soft delete)              | Delete.Customers |
| GET    | `/Sales/Customers/{id}/Orders`          | CustomerOrders(id, filter)          | Historial de pedidos del cliente            | View.Orders      |
| GET    | `/Sales/Customers/{id}/Addresses`       | CustomerAddresses(id)               | Direcciones del cliente                      | View.Customers   |
| POST   | `/Sales/Customers/{id}/Addresses`       | AddAddress(id, AddressViewModel)    | Añadir dirección al cliente                  | Update.Customers |
| POST   | `/Sales/Customers/Import`               | ImportCustomers(ImportViewModel)    | Importar clientes desde Excel                | Create.Customers |
| GET    | `/Sales/Customers/Export`               | Export(filter)                      | Exportar clientes a Excel                   | Export.Customers |

---

## Area: Sales - Reportes y Analytics

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Sales/Reports`                        | Index()                             | Centro de reportes                           | View.Reports     |
| GET    | `/Sales/Reports/Sales`                  | SalesReport()                       | Formulario reporte de ventas                 | View.Reports     |
| POST   | `/Sales/Reports/Sales/Generate`         | GenerateSalesReport(model)          | Generar reporte de ventas                    | Generate.Reports |
| GET    | `/Sales/Reports/Products`               | ProductsReport()                    | Formulario reporte de productos              | View.Reports     |
| POST   | `/Sales/Reports/Products/Generate`      | GenerateProductsReport(model)       | Generar reporte de productos                 | Generate.Reports |
| GET    | `/Sales/Reports/Customers`              | CustomersReport()                   | Formulario reporte de clientes               | View.Reports     |
| POST   | `/Sales/Reports/Customers/Generate`     | GenerateCustomersReport(model)      | Generar reporte de clientes                  | Generate.Reports |
| GET    | `/Sales/Reports/Inventory`              | InventoryReport()                   | Formulario reporte de inventario             | View.Reports     |
| POST   | `/Sales/Reports/Inventory/Generate`     | GenerateInventoryReport(model)      | Generar reporte de inventario                | Generate.Reports |
| GET    | `/Sales/Reports/Download/{id}`          | DownloadReport(id)                  | Descargar reporte generado                   | Download.Reports |
| GET    | `/Sales/Reports/History`                | ReportHistory()                     | Historial de reportes generados             | View.Reports     |
| POST   | `/Sales/Reports/Schedule`               | ScheduleReport(ScheduleViewModel)   | Programar reporte automático                 | Schedule.Reports |

---

## Area: Admin - Gestión de Usuarios

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Admin/Users`                          | Index(filter)                       | Lista paginada de usuarios                   | View.Users       |
| GET    | `/Admin/Users/Details/{id}`             | Details(id)                         | Ver detalles del usuario                     | View.Users       |
| GET    | `/Admin/Users/Create`                   | Create()                            | Formulario crear usuario                     | Create.Users     |
| POST   | `/Admin/Users/Create`                   | Create(CreateUserViewModel)         | Procesar creación de usuario                 | Create.Users     |
| GET    | `/Admin/Users/Edit/{id}`                | Edit(id)                            | Formulario editar usuario                    | Update.Users     |
| POST   | `/Admin/Users/Edit/{id}`                | Edit(id, UpdateUserViewModel)       | Procesar edición de usuario                  | Update.Users     |
| POST   | `/Admin/Users/Delete/{id}`              | Delete(id)                          | Eliminar usuario (soft delete)              | Delete.Users     |
| POST   | `/Admin/Users/ToggleActive/{id}`        | ToggleActive(id)                    | Activar/desactivar usuario                   | Update.Users     |
| GET    | `/Admin/Users/{id}/Roles`               | UserRoles(id)                       | Ver roles del usuario                        | View.Users       |
| POST   | `/Admin/Users/{id}/AssignRole`          | AssignRole(id, roleId)              | Asignar rol al usuario                       | Assign.Roles     |
| POST   | `/Admin/Users/{id}/RemoveRole`          | RemoveRole(id, roleId)              | Quitar rol del usuario                       | Assign.Roles     |
| GET    | `/Admin/Users/{id}/Activity`            | UserActivity(id, filter)            | Historial de actividad del usuario          | View.ActivityLogs|
| POST   | `/Admin/Users/ResetPassword/{id}`       | ResetPassword(id)                   | Resetear contraseña del usuario             | Reset.Passwords  |

---

## Area: Admin - Gestión de Roles y Permisos

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Admin/Roles`                          | Index()                             | Lista de roles del sistema                   | View.Roles       |
| GET    | `/Admin/Roles/Details/{id}`             | Details(id)                         | Ver detalles del rol                         | View.Roles       |
| GET    | `/Admin/Roles/Create`                   | Create()                            | Formulario crear rol                         | Create.Roles     |
| POST   | `/Admin/Roles/Create`                   | Create(CreateRoleViewModel)         | Procesar creación de rol                     | Create.Roles     |
| GET    | `/Admin/Roles/Edit/{id}`                | Edit(id)                            | Formulario editar rol                        | Update.Roles     |
| POST   | `/Admin/Roles/Edit/{id}`                | Edit(id, UpdateRoleViewModel)       | Procesar edición de rol                      | Update.Roles     |
| POST   | `/Admin/Roles/Delete/{id}`              | Delete(id)                          | Eliminar rol personalizado                   | Delete.Roles     |
| GET    | `/Admin/Roles/{id}/Permissions`         | RolePermissions(id)                 | Ver permisos del rol                         | View.Permissions |
| POST   | `/Admin/Roles/{id}/AddPermission`       | AddPermission(id, permissionId)     | Añadir permiso al rol                        | Update.Permissions|
| POST   | `/Admin/Roles/{id}/RemovePermission`    | RemovePermission(id, permissionId)  | Quitar permiso del rol                       | Update.Permissions|
| GET    | `/Admin/Permissions`                    | Permissions()                       | Lista de todos los permisos                  | View.Permissions |
| GET    | `/Admin/Permissions/Matrix`             | PermissionsMatrix()                 | Matriz rol-permisos                          | View.Permissions |

---

## Area: Admin - Configuración del Sistema

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Admin/System/Settings`                | Settings()                          | Configuraciones del sistema                  | View.Settings    |
| POST   | `/Admin/System/Settings`                | SaveSettings(SettingsViewModel)     | Guardar configuraciones                      | Update.Settings  |
| GET    | `/Admin/System/Maintenance`             | Maintenance()                       | Panel de mantenimiento                       | System.Maintenance|
| POST   | `/Admin/System/ClearCache`              | ClearCache()                        | Limpiar caché del sistema                    | System.Maintenance|
| POST   | `/Admin/System/OptimizeDatabase`        | OptimizeDatabase()                  | Optimizar base de datos                      | System.Maintenance|
| GET    | `/Admin/System/Backup`                  | Backup()                            | Panel de respaldos                           | System.Backup    |
| POST   | `/Admin/System/CreateBackup`            | CreateBackup()                      | Crear respaldo de BD                         | System.Backup    |
| GET    | `/Admin/System/Logs`                    | SystemLogs(filter)                  | Logs del sistema                             | View.SystemLogs  |
| GET    | `/Admin/System/AuditTrail`              | AuditTrail(filter)                  | Rastro de auditoría                          | View.AuditLogs   |
| GET    | `/Admin/System/Health`                  | SystemHealth()                      | Estado de salud del sistema                  | View.SystemHealth|

---

## Rutas API para AJAX y JavaScript

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/api/dashboard/stats`                  | GetDashboardStats()                 | Estadísticas para dashboard (JSON)          | Autenticado      |
| GET    | `/api/products/search`                  | SearchProducts(term)                | Búsqueda de productos (JSON)                | View.Products    |
| GET    | `/api/customers/search`                 | SearchCustomers(term)               | Búsqueda de clientes (JSON)                 | View.Customers   |
| POST   | `/api/notifications/markread/{id}`      | MarkNotificationRead(id)            | Marcar notificación como leída             | Autenticado      |
| GET    | `/api/notifications/unread`             | GetUnreadNotifications()            | Obtener notificaciones no leídas           | Autenticado      |
| POST   | `/api/products/{id}/stock`              | UpdateProductStock(id, quantity)    | Actualizar stock vía AJAX                   | Update.Inventory |
| GET    | `/api/orders/{id}/timeline`             | GetOrderTimeline(id)                | Timeline del pedido (JSON)                  | View.Orders      |
| POST   | `/api/upload/image`                     | UploadImage(file)                   | Subir imagen genérica                       | Upload.Files     |
| DELETE | `/api/files/{id}`                       | DeleteFile(id)                      | Eliminar archivo                            | Delete.Files     |

---

## Rutas de Notificaciones y Tiempo Real

| Método | Ruta                                    | Action                              | Descripción                                  | Permisos         |
|--------|-----------------------------------------|-------------------------------------|----------------------------------------------|------------------|
| GET    | `/Notifications`                        | Index(filter)                       | Centro de notificaciones                     | Autenticado      |
| POST   | `/Notifications/MarkAllRead`            | MarkAllAsRead()                     | Marcar todas como leídas                    | Autenticado      |
| POST   | `/Notifications/Delete/{id}`            | Delete(id)                          | Eliminar notificación                       | Autenticado      |
| GET    | `/Notifications/Settings`               | Settings()                          | Configurar preferencias                     | Autenticado      |
| POST   | `/Notifications/Settings`               | SaveSettings(NotificationSettings)  | Guardar preferencias                        | Autenticado      |

---

## Middleware y Filtros de Autorización

```csharp
// Atributos de autorización personalizados
[Authorize(Policy = "RequirePermission")]
[Permission("View.Products")]
public async Task<IActionResult> Index() { }

[Authorize(Policy = "RequireRole")]
[RequireRole("Admin", "Manager")]
public async Task<IActionResult> AdminAction() { }

// Filtros globales
[AuditLog] // Registra todas las acciones en audit_logs
[ValidateAntiForgeryToken] // Protección CSRF
[ResponseCache(NoStore = true)] // No cachear páginas admin
```

---

## Configuración de Rutas en Startup.cs

```csharp
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.UseEndpoints(endpoints =>
{
    // Rutas de Areas
    endpoints.MapControllerRoute(
        name: "areas",
        pattern: "{area:exists}/{controller=Dashboard}/{action=Index}/{id?}");
    
    // Rutas API
    endpoints.MapControllerRoute(
        name: "api",
        pattern: "api/{controller}/{action}/{id?}");
    
    // Ruta por defecto
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Dashboard}/{action=Index}/{id?}");
    
    // SignalR para notificaciones en tiempo real
    endpoints.MapHub<NotificationHub>("/notificationHub");
});
```

---

## Consideraciones de Seguridad y Rendimiento

- **Autorización basada en políticas:** Cada acción valida permisos específicos.
- **Audit Trail:** Todas las acciones CRUD se registran automáticamente.
- **CSRF Protection:** Tokens anti-falsificación en todos los formularios.
- **Rate Limiting:** Limitación de peticiones en endpoints sensibles.
- **Input Validation:** Validación del lado servidor con Data Annotations.
- **Output Encoding:** Prevención de XSS con Razor encoding.
- **File Upload Security:** Validación de tipos de archivo y tamaños.
- **Database Security:** Uso de Entity Framework con parámetros seguros.
- **Session Management:** JWT con expiración y renovación automática.
- **Logging:** Registro detallado de errores y actividad sospechosa.
- **Caching:** Caché de datos frecuentes para mejorar rendimiento.
- **Pagination:** Paginación obligatoria en listados grandes.
- **Bulk Operations:** Operaciones masivas optimizadas.
- **Real-time Updates:** SignalR para notificaciones instantáneas.

---