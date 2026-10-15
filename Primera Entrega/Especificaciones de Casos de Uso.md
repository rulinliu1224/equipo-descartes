# Especificación de Casos de Uso - Sistema de Facturación

<details>
<summary><b>CU-01: Iniciar Sesión</b></summary>
 
### CU-01: Iniciar Sesión

* **ID / Código:** `CU-01`
* **Nombre del Caso de Uso:** Iniciar sesión
* **Descripción:** El usuario accede al sistema de facturación autenticándose con sus credenciales corporativas (@turbineh.com).
* **Actor Principal:** Usuario operativo (por herencia: Administrador, Superadministrador).
* **Actores Secundarios:** Servicio de email (sistema externo), para la recuperación de contraseña.
* **Ámbito (Scope):** Sistema de Facturación.
* **Nivel (Level):** Objetivo de usuario.
* **Requisitos relacionados:** UR1, FR1, FR1.1, NFR8, NFR9, NFR10.
  
#### Interesados e Intereses
* **Usuario operativo:** Acceder a las funciones del sistema según sus permisos.
* **Empresa:** Garantizar que solo el personal autorizado con correo corporativo ingrese al sistema.
  
#### Condiciones y Garantías
* **Precondiciones:** El usuario debe estar previamente dado de alta en el sistema por un Administrador, no existe el sign-up.
* **Garantía de Éxito:** El usuario inicia sesión correctamente y el sistema carga el dashboard adaptado a sus permisos.
* **Garantía Mínima:** El intento fallido queda registrado en el historial y se deniega el acceso.
  
#### Flujos de Eventos
**Disparador:** El usuario ingresa a la URL del sistema e introduce sus credenciales.
 
**Escenario Principal de Éxito:**
1. El usuario ingresa su correo corporativo y contraseña.
2. El sistema valida el formato del correo (@turbineh.com).
3. El sistema comprueba la existencia de la cuenta, que esté activa y la validez de la contraseña.
4. El sistema carga el rol, los clientes asignados y los permisos del usuario.
5. El sistema redirige al usuario a la pantalla principal.
   
**Extensiones (Flujos Alternativos):**
* 2a. **Dominio de correo no corporativo:** El sistema notifica que solo se admiten cuentas @turbineh.com y finaliza la operación.
* 3a. **Credenciales incorrectas:** El sistema muestra un mensaje de error genérico y permite reintentar.
* 3b. **Cuenta desactivada:** El sistema deniega el acceso e indica que contacte con un Administrador.
* 3c. **El usuario no recuerda su contraseña (`<<extend>>` Recuperar contraseña):**
  * 3c1. El usuario selecciona "Recuperar contraseña".
  * 3c2. El sistema envía al correo corporativo un enlace de restablecimiento con caducidad.
  * 3c3. El usuario abre el enlace dentro del plazo y define una nueva contraseña.
  * 3c4. Si el enlace ha caducado, el sistema lo invalida y permite solicitar uno nuevo.
 * **Inactividad prolongada:** En cualquier momento posterior al inicio de sesión, si el usuario supera el tiempo de inactividad configurado, el sistema cierra la sesión automáticamente.
</details>

---
<details>
<summary><b>CU-02: Gestionar Suscripción y Baja de Cliente</b></summary>
 
### CU-02: Gestionar Suscripción y Baja de Cliente
 
* **ID / Código:** `CU-02`
* **Nombre del Caso de Uso:** Gestionar suscripción (plan, tarifa, pausa, baja)
* **Descripción:** El usuario operativo administra la suscripción de un cliente (plan, tarifa acordada, fecha de inicio, responsable, condiciones de pago y descuentos), la pausa o procesa la baja de la recurrencia. La baja puede tramitarse aunque el cliente tenga facturas pendientes: en ese caso se deja de generar facturas recurrentes, pero el recobro de las facturas pendientes sigue activo.
* **Actor Principal:** Usuario operativo.
* **Actores Secundarios:** Stripe (Sistema externo), Servicio de email / WhatsApp (Sistema externo).
* **Ámbito (Scope):** Sistema de Facturación.
* **Nivel (Level):** Objetivo de usuario.
* **Requisitos relacionados:** UR2, FR5, FR5.2, FR5.3, FR12.1, FR18.
  
#### Interesados e Intereses
* **Cliente:** Solicitar la baja o el cambio de su suscripción y dejar de recibir facturas nuevas.
* **Empresa:** Cobrar todas las deudas pendientes aunque el cliente se dé de baja, y controlar los cambios de tarifa.
  
#### Condiciones y Garantías
* **Precondiciones:**
  * El usuario debe estar autenticado en el sistema.
  * El cliente debe existir en la base de datos y estar asignado al usuario (o el usuario tener permiso sobre él).
* **Garantía de Éxito:** La suscripción queda actualizada, pausada o dada de baja; el cambio queda registrado en el historial del cliente.
* **Garantía Mínima:** Las facturas ya emitidas y su recobro no se ven afectados por ningún cambio en la suscripción. Si la operación falla, la suscripción conserva su estado anterior.

#### Flujos de Eventos
* **Disparador:** El usuario selecciona un cliente y elige gestionar su suscripción, o llega una solicitud de baja por la API.

**Escenario Principal de Éxito (baja de la recurrencia):**
1. El usuario busca y selecciona al cliente.
2. El sistema muestra el estado actual de la suscripción y el resumen de deuda pendiente.
3. El usuario solicita tramitar la baja de la recurrencia indicando motivo y fecha efectiva.
4. El sistema verifica si el cliente tiene facturas pendientes de pago.
5. El sistema registra la baja de la recurrencia y cancela la suscripción en Stripe a través de la API.
6. El sistema deja de incluir al cliente en la generación automática de facturas recurrentes.
7. El sistema registra la operación en el historial del cliente y emite la confirmación de la baja al Cliente y a los Usuarios responsables.
   
**Extensiones (Flujos Alternativos):**
* **3a. Cambio de plan o tarifa:**
  * 3a1. El usuario modifica el plan o la tarifa acordada.
  * 3a2. El sistema envía el cambio a la bandeja de aprobaciones (`<<include>>` Solicitar aprobación); el cambio no se aplica hasta ser aprobado por un Administrador.
* **3b. Pausa de la suscripción:**
  * 3b1. El usuario indica fecha de inicio y de fin de la pausa y el motivo.
  * 3b2. El sistema no genera facturas recurrentes durante el periodo pausado y la reactiva automáticamente al finalizar.
* **4a. El cliente tiene facturas pendientes o en mora:**
  * 4a1. El sistema avisa al usuario del importe pendiente y le pide confirmación para continuar.
  * 4a2. El sistema registra la baja de la recurrencia, pero mantiene al cliente en estado "Baja con deuda".
  * 4a3. El sistema mantiene activos los recordatorios de cobro de las facturas pendientes.
  * 4a4. Cuando se confirma el pago de la última factura pendiente, el sistema pasa el cliente a estado "Baja" y notifica al Usuario/Administrador responsable.
* **5a. Fallo en la cancelación en Stripe:**
  * 5a1. El sistema registra la incidencia, notifica al Administrador y reintenta la cancelación más tarde. La baja queda registrada en el sistema.
</details>

---

<details>
<summary><b>CU-03: Generar Facturas Recurrentes Automáticas</b></summary>
 
### CU-03: Generar Facturas Recurrentes Automáticas y Prorrateo
 
* **ID / Código:** `CU-03`
* **Nombre del Caso de Uso:** Generar facturas recurrentes automáticas el día 1 con prorrateo
* **Descripción:** Procesar de forma automática al inicio de cada mes (día 1) la emisión de exactamente una factura mensual por cada cliente con suscripción activa, sin importar cuántas facturas anteriores permanezcan impagas. Si el cliente se dio de alta a mitad del mes anterior, la primera factura aplica el **prorrateo** (descuento proporcional a los días no disfrutados).
* **Actor Principal:** Sistema (proceso programado) / Administrador (ejecución manual).
* **Actores Secundarios:** Servicio de email / WhatsApp (Sistema externo).
* **Ámbito (Scope):** Módulo de Facturación Automática.
* **Nivel (Level):** Subfunción / Objetivo de sistema.
* **Requisitos relacionados:** BR1, FR6, FR6.1, FR5.1, FR21, NFR5, NFR6, NFR12.
  
#### Interesados e Intereses
* **Empresa:** Garantizar la emisión de una factura mensual por cliente activo sin intervención manual.
* **Cliente:** Recibir mensualmente su factura y ver reflejado claramente el ajuste proporcional si se dio de alta a mitad de mes.
  
#### Condiciones y Garantías
* **Precondiciones:** El cliente debe tener una suscripción activa (ni dada de baja ni pausada en el periodo).
* **Garantía de Éxito:** Se emite la factura mensual de cada cliente activo, aplicando el prorrateo si corresponde, y se notifica al cliente.
* **Garantía Mínima:** Si la generación de la factura de un cliente falla, el sistema registra la incidencia, notifica al responsable del cliente y continúa con el resto de clientes sin interrupciones. Nunca se emiten dos facturas recurrentes del mismo periodo para el mismo cliente.

#### Flujos de Eventos
**Disparador:** Llega el día 1 de cada mes a las 8:00 h o el Administrador lanza la ejecución manual.
 
**Escenario Principal de Éxito:**
1. El sistema identifica a todos los clientes con suscripción activa para el periodo (independientemente de si tienen facturas impagas anteriores).
2. Para cada cliente, el sistema comprueba que no exista ya una factura recurrente del mismo periodo (Por si se realizó una manualmente)
3. El sistema calcula el importe con la tarifa vigente aprobada, aplicando descuentos e impuestos con la regla de redondeo fija.
4. El sistema genera la factura del mes, le asigna numeración oficial, conserva el PDF emitido y actualiza el saldo deudor del cliente.
5. El sistema ejecuta el caso de uso `<<include>>` Enviar facturas y recordatorios.
6. El sistema registra en el log de procesos automáticos el resultado de la ejecución (facturas emitidas, fallidas) y avisa al terminar.

**Extensiones (Flujos Alternativos):**
* **1a. El cliente acumula facturas de meses anteriores impagas:**
  * 1a1. El sistema emite igualmente la factura del mes actual.
  * 1a2. El sistema suma el nuevo importe al saldo total adeudado por el cliente.
* **2a. Ya existe la factura del periodo (por ejemplo, ejecución manual repetida):**
  * 2a1. El sistema omite al cliente y lo registra en el log.
* **3a. Alta a mitad de mes (prorrateo):**
  * 3a1. El sistema detecta que la fecha de inicio de la suscripción es posterior al día 1 del periodo anterior.
  * 3a2. El sistema calcula el importe proporcional a los días de servicio y lo refleja como línea de ajuste en la factura.
* **3b. El cliente cumple un año desde su primera factura:**
  * 3b1. El sistema factura con la tarifa vigente (sin cambiarla automáticamente).
  * 3b2. El sistema calcula el ajuste de tarifa y notifica al Administrador para que apruebe el cambio.
* **4a. Fallo al generar la factura de un cliente:**
  * 4a1. El sistema registra el error, notifica al responsable y continúa con el siguiente cliente.
</details>

---

<details>
<summary><b>CU-04: Gestionar Facturas Puntuales</b></summary>
 
### CU-04: Gestionar Facturas Puntuales (borrador, vista previa, PDF)
 
* **ID / Código:** `CU-04`
* **Nombre del Caso de Uso:** Gestionar facturas puntuales (borrador, vista previa, PDF, emisión)
* **Descripción:** Permite a los usuarios crear, editar en borrador y emitir facturas únicas para un cliente (servicios puntuales o ventas aisladas no recurrentes). Una vez aprobadas, se emiten a principio de mes (día 1), consolidándose con el ciclo mensual de cobros.
* **Actor Principal:** Usuario operativo.
* **Actores Secundarios:** Servicio de email / WhatsApp (Sistema externo).
* **Ámbito (Scope):** Módulo de Facturación Puntual.
* **Nivel (Level):** Objetivo de usuario.
* **Requisitos relacionados:** UR3, FR7, FR8, FR8.1, FR8.2, FR8.3, FR12.1, NFR12.
  
#### Interesados e Intereses
* **Empresa:** Disponer de un mecanismo flexible para facturar conceptos únicos sin alterar el ciclo de suscripción recurrente del cliente.
* **Cliente:** Recibir el comprobante detallado de los servicios puntuales con su enlace de pago.
  
#### Condiciones y Garantías
* **Precondiciones:**
  * El usuario operativo debe estar autenticado.
  * El cliente receptor debe existir en la base de datos.
* **Garantía de Éxito:** La factura puntual es emitida, guardada en estado "Pendiente de pago" con su PDF original, sumada al historial e informes del cliente y enviada al cliente.
* **Garantía Mínima:** Si la generación falla antes de la emisión, el sistema no actualiza los saldos ni notifica al cliente, y avisa al Administrador. Una factura emitida no puede editarse.
  
#### Flujos de Eventos
**Disparador:** El usuario selecciona un cliente y elige crear una factura puntual.
 
**Escenario Principal de Éxito:**
1. El usuario selecciona el cliente e ingresa las líneas (concepto, cantidad, precio unitario, descuento e impuesto), la fecha de facturación y la fecha de vencimiento.
2. El sistema calcula los totales y guarda la factura en estado "Borrador".
3. El sistema genera una vista previa del comprobante con la identidad corporativa.
4. El usuario confirma la factura.
5. Llegado el día de facturación, el sistema asigna la numeración oficial, genera y conserva el PDF, establece el estado "Pendiente de pago" y actualiza el saldo del cliente.
6. El sistema ejecuta el caso de uso `<<include>>` Enviar facturas y recordatorios.
   
**Extensiones (Flujos Alternativos):**
* **2a. Edición del borrador:**
  * 2a1. El usuario reabre un borrador existente, modifica las líneas y guarda; el sistema recalcula los totales.
  * 2a2. Si otro usuario guardó el borrador entretanto, el sistema rechaza el guardado y avisa del conflicto.
* **4a. El importe supera el umbral configurado (`<<extend>>` Solicitar aprobación):**
  * 4a1. El sistema envía la factura a la bandeja de aprobaciones y la deja en estado "Pendiente de aprobación".
  * 4a2. Si se aprueba, continúa en el paso 5; si se rechaza, vuelve a "Borrador" con el motivo del rechazo.
* **4b. Cancelación del proceso previo a la emisión:**
  * 4b1. El usuario decide descartar el borrador.
  * 4b2. El sistema elimina el borrador sin alterar la contabilidad del cliente.
</details>

---

<details>
<summary><b>CU-05: Enviar Facturas y Recordatorios de Pago</b></summary>
 
### CU-05: Enviar Facturas y Recordatorios de Pago
 
* **ID / Código:** `CU-05`
* **Nombre del Caso de Uso:** Enviar facturas y recordatorios de pago 
* **Descripción:** Subfunción encargada de enviar las facturas emitidas (recurrentes y puntuales) y de gestionar los recordatorios automáticos de cobro según el calendario configurado, mientras la factura esté pendiente o en mora.
* **Actor Principal:** Sistema (proceso programado).
* **Actores Secundarios:** Servicio de email / WhatsApp (Sistema externo), Stripe (Sistema Externo).
* **Ámbito (Scope):** Módulo de Notificaciones.
* **Nivel (Level):** Subfunción / Proceso reutilizable.
* **Requisitos relacionados:** BR2, FR10, FR11, FR11.1, FR11.2, FR18, NFR19.
  
#### Interesados e Intereses
* **Empresa:** Reducir la deuda pendiente y los avisos manuales mediante recordatorios automáticos.
* **Cliente:** Recibir sus comprobantes a tiempo y recordar los vencimientos pendientes.
  
#### Condiciones y Garantías
* **Precondiciones:**
  * Existe una factura emitida (recurrente o puntual) en estado "Pendiente de pago" o "Mora" (pasada la fecha de vencimiento).
* **Garantía de Éxito:** Se envía la notificación o recordatorio por el canal configurado (Email/WhatsApp) y se registra el envío en el historial del cliente.
* **Garantía Mínima:** Si falla el envío por un canal, el sistema intenta por el canal alternativo y registra el resultado; si fallan ambos, notifica al Usuario responsable del cliente.
  
#### Flujos de Eventos
**Disparador:** Emisión de una factura o ejecución del proceso diario de revisión de deudas.
 
**Escenario Principal de Éxito:**
1. El sistema recibe la orden de envío de una factura (emisión o recordatorio programado).
2. El sistema comprueba el calendario de recordatorios aplicable (el general o el pactado con el cliente).
3. El sistema genera el enlace de pago de Stripe y adjunta el PDF original de la factura.
4. El sistema envía el mensaje al cliente usando la plantilla configurada a través del servicio de Email/WhatsApp.
5. El sistema registra la fecha, hora y canal del envío en el historial de notificaciones.
   
**Extensiones (Flujos Alternativos):**
* **1a. Recordatorio periódico por facturas en mora:**
  * 1a1. El proceso diario detecta facturas no pagadas que superaron la fecha de vencimiento y las pasa a "Mora".
  * 1a2. El sistema envía los avisos según la frecuencia configurada, incluyendo el detalle de la deuda total.
* **2a. La factura tiene los recordatorios pausados por disputa:**
  * 2a1. El sistema no envía el recordatorio y lo registra.
  * 2a2. Al llegar la fecha de revisión de la disputa, el sistema notifica al responsable para que la resuelva o prorrogue.
* **2b. La factura ya está pagada:**
  * 2b1. El sistema cancela los recordatorios pendientes de esa factura.
* **4a. Fallo del canal principal:**
  * 4a1. El sistema reintenta por el canal alternativo; si también falla, notifica al responsable.
</details>

---

<details>
<summary><b>CU-06: Consultar Informes y Exportar</b></summary>
 
### CU-06: Consultar Informes y Exportar
 
* **ID / Código:** `CU-06`
* **Nombre del Caso de Uso:** Consultar informes y exportar
* **Descripción:** Permite consultar y exportar informes configurables de facturación, cobros y deudas, consolidando en una única fuente de datos la facturación recurrente y la facturación puntual.
* **Actor Principal:** Usuario operativo con permiso de informes
* **Actores Secundarios:** Servicio de email (Sistema externo), si es necesario el envió de los mismos.
* **Ámbito (Scope):** Módulo de Informes.
* **Nivel (Level):** Objetivo de usuario.
* **Requisitos relacionados:** BR7, UR9, FR14, FR14.2, NFR5, NFR12.
  
#### Interesados e Intereses
* **Empresa / Administradores:** Disponer de una visión clara de lo facturado, cobrado, pendiente y vencido.
  
#### Condiciones y Garantías
* **Precondiciones:**
  * El usuario debe estar autenticado y tener permiso para consultar informes.
* **Garantía de Éxito:** El sistema consolida ambas fuentes y muestra o exporta el informe.
* **Garantía Mínima:** Si no hay datos en el rango seleccionado, el sistema muestra la pantalla sin errores indicando "Sin registros". El usuario solo ve datos de los clientes a los que tiene acceso. En el caso de administradores y el superadministrador pueden visualizar datos sobre toda la base de clientes.

#### Flujos de Eventos
**Disparador:** El usuario accede a la sección "Informes" y selecciona los parámetros.
 
**Escenario Principal de Éxito:**
1. El usuario selecciona el tipo de informe, el rango de fechas y los filtros.
2. El sistema consulta las facturas recurrentes y puntuales y los pagos del periodo.
3. El sistema calcula los totales facturados, cobrados, pendientes y vencidos.
4. El sistema presenta en pantalla los gráficos y tablas.
5. El usuario descarga el informe en CSV o PDF.
   
**Extensiones (Flujos Alternativos):**
* **1a. Rango de fechas no válido:**
  * 1a1. El usuario ingresa una fecha de inicio posterior a la de fin.
  * 1a2. El sistema indica que el rango es inválido y solicita corregirlo.
* **2a. Inexistencia de datos para los filtros seleccionados:**
  * 2a1. El sistema muestra saldos en cero y el mensaje "Sin registros para los criterios seleccionados".
* **4a. Volumen de datos excesivo:**
  * 4a1. El sistema procesa el informe en segundo plano y avisa al usuario (y envía el archivo a su correo) al terminar.
* **5a. Error durante la exportación:**
  * 5a1. El sistema muestra una alerta de error y permite reintentar.
</details>

---

<details>
<summary><b>CU-07: Gestionar Cobros y Pasarela de Pagos</b></summary>

### CU-07: Gestionar Cobros y Pasarela de Pagos

* **ID / Código:** `CU-07`
* **Nombre del Caso de Uso:** Gestionar cobros y pasarela de pagos
* **Descripción:** Administra el procesamiento de los pagos de facturas (recurrentes y puntuales) a través de la integración con Stripe. Gestiona la actualización automática del estado de la factura ("Pagada", "Parcial", "Fallida") e impacta directamente el saldo del cliente.
* **Actor Principal:** Stripe / Pasarela de Pagos (`<<sistema externo>>`) / Usuario operativo.
* **Actores Secundarios:** Sistema.
* **Ámbito (Scope):** Módulo de Transacciones y Pagos.
* **Nivel (Level):** Subfunción / Proceso de transacción.

#### Interesados e Intereses
* **Empresa:** Asegurar el registro inmediato de los ingresos y conciliar automáticamente los pagos recibidos con las facturas emitidas.
* **Cliente:** Obtener la confirmación y comprobante de pago de forma instantánea al abonar su factura.

#### Condiciones y Garantías
* **Precondiciones:**
  * Existencia de una factura en estado "Pendiente de pago" o "Mora".
* **Garantía de Éxito:** El estado de la factura se actualiza a "Pagada", se liquida el saldo deudor del cliente y se emite el recibo/comprobante de cobro.
* **Garantía Mínima:** Si el cobro es rechazado, la pasarela notifica el fallo, la factura permanece en "Mora/Pendiente"y se registra el intento fallido en el historial.

#### Flujos de Eventos
**Disparador:** Evento webhook recibido desde Stripe (Pago realizado / Pago pendiente) o registro manual de cobro por parte del usuario operativo.

**Escenario Principal de Éxito (Automático vía Stripe):**
1. El cliente efectúa el pago mediante el enlace provisto por Stripe.
2. Stripe procesa la transacción con éxito y envía la notificación webhook al sistema.
3. El sistema valida la firma de seguridad del webhook y localiza la factura correspondiente.
4. El sistema cambia el estado de la factura a "Pagada" y descuenta el importe de la deuda del cliente.
5. Si el cliente no registra deudas restantes y tenía trámite de baja retenido (`CU-02`), el sistema desbloquea la baja definitiva.

**Extensiones (Flujos Alternativos):**
* **2a. Pago fallido o rechazado por la pasarela:**
  * 2a1. Stripe notifica evento de fallo (`payment_intent.payment_failed`).
  * 2a2. El sistema marca el intento como fallido y desencadena el envío de aviso de pago rechazado (`CU-05`).

</details>

---

<details>
<summary><b>CU-08: Gestionar Usuarios y Permisos</b></summary>

### CU-08: Gestionar Usuarios y Permisos

* **ID / Código:** `CU-08`
* **Nombre del Caso de Uso:** Gestionar usuarios y permisos (Control de acceso basado en roles)
* **Descripción:** Permite al Administrador dar de alta, modificar, desactivar o asignar roles (Ej. Administrador, Usuario operativo) a los usuarios del sistema para controlar el acceso a los diferentes módulos e información sensible.
* **Actor Principal:** Administrador.
* **Actores Secundarios:** Ninguno.
* **Ámbito (Scope):** Módulo de Administración de Usuarios.
* **Nivel (Level):** Objetivo de usuario.

#### Interesados e Intereses
* **Empresa:** Garantizar la seguridad de la información limitando las acciones operativas según el perfil del empleado.

#### Condiciones y Garantías
* **Precondiciones:**
  * El Administrador debe estar autenticado (`CU-01`).
  * El Administradotr tiene que tener la opción de asignar permisos.(Dados por el Superadministrador)
* **Garantía de Éxito:** Se actualizan los roles, credenciales o permisos en el sistema y se aplican de inmediato a las sesiones de los usuarios.
* **Garantía Mínima:** No se permite eliminar o desactivar al Superadministrador del sistema.

#### Flujos de Eventos
**Disparador:** El Administrador accede al panel de 'Gestión de Usuarios y Permisos'.
**Escenario Principal de Éxito:**
1. El Administrador visualiza la lista de usuarios del sistema.
2. Selecciona un usuario existente o la opción para crear un nuevo usuario.
3. Define los datos personales, correo corporativo y asigna el rol correspondiente (Adminisrador/ Usuario operativo).
4. El sistema guarda la configuración y asigna la matriz de permisos correspondiente.
5. El sistema envía un correo al nuevo usuario con las credenciales o enlace de activación.

**Extensiones (Flujos Alternativos):**
* **3a. Desactivación/Revocación de acceso a un usuario:**
  * 3a1. El Administrador cambia el estado del usuario a "Inactivo".
  * 3a2. El sistema invalida inmediatamente todas las sesiones activas de dicho usuario.

</details>
