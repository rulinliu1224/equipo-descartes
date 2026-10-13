# Requisitos funcionales

- Generar facturas recurrentes automáticamente el día 1 a partir de la suscripción, con descuento en altas a mitad de mes.
- Crear facturas puntuales (concepto, cantidad, precio unitario, descuento e impuesto).
- Gestionar las facturas: poder editar el borrador y ver una vista previa.
- Generar el PDF de la factura con la identidad corporativa y conservar el PDF original emitido.
- Gestionar clientes (alta, baja, reactivación…) con datos comerciales y fiscales separados y varios contactos por empresa.
- Importar clientes desde CSV con previsualización y resumen de creados, actualizados y rechazados.
- Gestionar la suscripción del cliente (plan, tarifa acordada, fecha de inicio, responsable interno, condiciones de pago y descuentos).
- Revisar anualmente la facturación de cada cliente. Los cambios de tarifa requieren que un usuario los apruebe; no se pueden hacer de forma automática.
- Registrar pagos (Stripe u otro sistema), detectando eventos duplicados o importes mal implementados en el sistema.
- Enviar facturas y recordatorios automáticos por email y/o WhatsApp según un calendario configurable, que puede ser modificado según lo pactado con el cliente.
- Pausar los recordatorios de una factura cuando está en disputa por algún problema del cliente. Para ello hay que indicar un motivo y una fecha de revisión.
- Permitir la baja de la recurrencia aunque el cliente no haya pagado facturas anteriores, pero manteniendo activo el recobro de las facturas pendientes.
- Iniciar sesión con email del dominio corporativo y gestionar usuarios internos (crear y desactivar), sin autorregistro.
- Recuperar la contraseña mediante un enlace (caduca al tiempo).
- Asignar roles (Superadministrador, Administrador, Usuario) y clientes a cada usuario, con posibilidad de dar permisos temporales.
- Aprobar o rechazar operaciones sensibles desde una bandeja de aprobaciones. Una operación se manda a la bandeja de aprobaciones después de cruzar un umbral de precio previamente configurado.
- Buscar globalmente datos en la aplicación y aplicar filtros (cliente, ubicación, sector, estado de factura, recurrente/puntual, fecha, responsable).
- Consultar informes (configurables) y un dashboard financiero (facturado, cobrado, pendiente, vencido) y tener la posibilidad de exportar a CSV y PDF.
- Mostrar en un dashboard o tabla el trabajo pendiente que tiene un usuario.
- Notificar internamente al responsable de cada cliente los eventos relevantes, con prioridad y estado.
- Abrir y seguir incidencias o tickets de facturación de un cliente.
- Que haya logs de lo que se le ha ido haciendo a cada cliente, para que haya un registro por si el cliente lo pide en un futuro.
- Ofrecer una API autenticada para consultar clientes, facturas, pagos o deuda pendiente, y para notificar solicitudes de baja.
- Configurar la mayoría de la interfaz (plazos, recordatorios, impuestos, moneda (euro, dólar…), tarifas, plantillas de email).
- Registrar lo que van haciendo los procesos automáticos y mostrar una pantalla de salud del sistema.

# Requisitos no funcionales

- Uso simple e interactivo: un usuario que no conoce la aplicación da de alta un cliente estándar con su suscripción en menos de 5 minutos o es capaz de encontrar una factura o cliente en menos de un minuto.
- Compatible con las últimas versiones de los navegadores más comunes (Chrome, Firefox, Safari y Edge) y también con la interfaz web en el móvil.
- Que use los estándares de diseño web para móviles, incluyendo que se pueda hacer zoom.
- Las búsquedas habituales responden en menos de 2 segundos con miles de clientes y cientos de miles de facturas.
- Las operaciones pesadas se ejecutan en segundo plano y avisan al terminar.
- El fallo de un cliente en el procesamiento no detiene el proceso del resto.
- Detectar conflictos cuando dos usuarios editan el mismo elemento y garantizar la consistencia.
- Acceso solo con email del dominio corporativo.
- Contraseñas cifradas con hash.
- Cierre de sesión por inactividad.
- Autorización comprobada en servidor.
- Los importes se calculan con una regla de redondeo fija.
- Fechas y zonas horarias se guardarán en UTC, pero también se muestran en la zona del cliente.
- Interfaz preparada para varios idiomas (español e inglés).
- Copias de seguridad automáticas diarias de datos y documentos, con posibilidad de restaurar una versión previa.
- Se conservan los datos de facturas de clientes aunque se den de baja.
- Documentación y mantenimiento: cada regla definida en un único sitio, documentación de cómo funciona el programa, de la API y de despliegue.
- Integraciones (Stripe, email, WhatsApp) sustituibles sin cambiar el modelo de factura.
- Nada de depender de inteligencia artificial.
