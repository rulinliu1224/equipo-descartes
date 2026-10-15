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
 
### CU-07: Gestionar Cobros (pagos, recordatorios en disputa)
 
* **ID / Código:** `CU-07`
* **Nombre del Caso de Uso:** Gestionar cobros y pasarela de pagos
* **Descripción:** Registra los pagos de las facturas (automáticamente vía Stripe o manualmente por otros medios), actualiza el estado de la factura ("Pagada", "Parcial", "Fallida") y el saldo del cliente, y permite pausar los recordatorios de una factura en disputa.
* **Actor Principal:** Usuario operativo / Stripe (Sistema Externo).
* **Actores Secundarios:** Sistema.
* **Ámbito (Scope):** Módulo de Transacciones y Pagos.
* **Nivel (Level):** Objetivo de usuario / Subfunción (vía webhook).
* **Requisitos relacionados:** UR4, UR5, FR9, FR9.1, FR11.2, FR18, NFR19.
  
#### Interesados e Intereses
* **Empresa:** Registrar de inmediato los ingresos, conciliar los pagos con las facturas y detectar errores de cobro.
* **Cliente:** Obtener la confirmación del pago al abonar su factura.
  
#### Condiciones y Garantías
* **Precondiciones:**
  * Existe una factura en estado "Pendiente de pago" o "Mora".
* **Garantía de Éxito:** La factura pasa a "Pagada", se descuenta del saldo del cliente y se emite el comprobante de cobro.
* **Garantía Mínima:** Si el cobro es rechazado, la factura permanece en "Pendiente/Mora" y se registra el intento fallido. Un mismo pago nunca se aplica dos veces.
  
#### Flujos de Eventos
**Disparador:** Evento webhook de Stripe o registro manual del cobro por parte del usuario operativo.
 
**Escenario Principal de Éxito (automático vía Stripe):**
1. El cliente paga mediante el enlace de Stripe.
2. Stripe procesa la transacción y envía el webhook al sistema.
3. El sistema valida la firma del webhook y localiza la factura correspondiente.
4. El sistema comprueba que el evento no esté duplicado y que el importe coincida con el pendiente.
5. El sistema cambia el estado de la factura a "Pagada", descuenta el importe de la deuda del cliente y cancela sus recordatorios.
6. Si el cliente estaba en "Baja con deuda" y ya no tiene deuda, el sistema lo pasa a "Baja" y notifica al Usuario responsable.
   
**Extensiones (Flujos Alternativos):**
* **1a. Registro manual de un cobro (transferencia, efectivo u otro medio):**
  * 1a1. El usuario selecciona la factura e indica importe, fecha, medio y referencia.
  * 1a2. El sistema continúa en el paso 4.
* **2a. Pago fallido o rechazado por la pasarela:**
  * 2a1. Stripe notifica el fallo.
  * 2a2. El sistema marca el intento como fallido y envía el aviso de pago rechazado.
* **3a. Firma del webhook no válida:**
  * 3a1. El sistema descarta el evento y lo registra como incidencia de seguridad.
* **4a. Evento de pago duplicado:**
  * 4a1. El sistema no vuelve a aplicar el pago, lo registra y notifica al responsable.
* **4b. Importe inferior al pendiente:**
  * 4b1. El sistema registra el pago, deja la factura en "Parcial" con el saldo restante y notifica al responsable.
* **4c. Importe superior al pendiente:**
  * 4c1. El sistema registra el pago, marca la diferencia como incidencia y notifica al responsable para su revisión.
* **Pausar recordatorios por disputa:**
  * 1. El usuario selecciona una factura que el cliente está disputando y elige "Pausar recordatorios".
  * 2. El usuario indica el motivo y la fecha de revisión.
  * 3. El sistema marca la factura "En disputa", deja de enviar recordatorios y lo registra en el historial.
  * 4. El usuario puede reanudar los recordatorios en cualquier momento.
</details>

---
 
<details>
<summary><b>CU-08: Gestionar Usuarios y Permisos</b></summary>
 
### CU-08: Gestionar Usuarios y Permisos
 
* **ID / Código:** `CU-08`
* **Nombre del Caso de Uso:** Gestionar usuarios y permisos (control de acceso basado en roles)
* **Descripción:** Permite dar de alta, modificar y desactivar usuarios internos, asignarles roles (Administrador, Usuario operativo), clientes asignados y permisos.
* **Actor Principal:** Administrador.
* **Actores Secundarios:** Servicio de email (Sistema Externo).
* **Ámbito (Scope):** Módulo de Administración de Usuarios.
* **Nivel (Level):** Objetivo de usuario.
* **Requisitos relacionados:** UR12, FR2, FR3, FR3.1, NFR11.
  
#### Interesados e Intereses
* **Empresa:** Garantizar la seguridad de la información limitando las acciones según el perfil del empleado.

#### Condiciones y Garantías
* **Precondiciones:**
  * El Administrador debe estar autenticado y tener permiso de gestión de usuarios.
* **Garantía de Éxito:** Se actualizan los roles, clientes asignados o permisos y se aplican de inmediato a las sesiones de los usuarios.
* **Garantía Mínima:** No se permite eliminar ni desactivar al Superadministrador, ni que un Administrador se asigne a sí mismo un rol superior.
  
#### Flujos de Eventos
**Disparador:** El Administrador accede al panel de gestión de usuarios y permisos.
 
**Escenario Principal de Éxito:**
1. El Administrador visualiza la lista de usuarios del sistema.
2. Selecciona un usuario existente o la opción de crear un nuevo usuario.
3. Define los datos personales, el correo corporativo, el rol y los clientes asignados.
4. El sistema valida el dominio del correo, guarda la configuración y aplica la matriz de permisos del rol.
5. El sistema envía al nuevo usuario un enlace de activación para definir su contraseña.
   
**Extensiones (Flujos Alternativos):**
* **3a. Desactivación de un usuario:**
  * 3a1. El Administrador cambia el estado del usuario a "Inactivo".
  * 3a2. El sistema invalida inmediatamente sus sesiones activas.
  * 3a3. Si el usuario tenía clientes asignados, el sistema pide reasignarlos a otro responsable.
* **3b. Permiso temporal:**
  * 3b1. El Administrador concede un permiso adicional indicando fecha de inicio y de fin.
  * 3b2. Al llegar la fecha de fin, el sistema retira el permiso automáticamente y lo registra.
* **4a. Correo de dominio no corporativo o ya existente:**
  * 4a1. El sistema rechaza el alta e indica el motivo.
</details>

---

<details>
<summary><b>CU-09: Gestionar Clientes</b></summary>
 
### CU-09: Gestionar Clientes (alta, reactivación)
 
* **ID / Código:** `CU-09`
* **Nombre del Caso de Uso:** Gestionar clientes
* **Descripción:** Permite dar de alta, modificar, dar de baja y reactivar clientes, con sus datos comerciales y fiscales separados y varios contactos por empresa.
* **Actor Principal:** Usuario operativo.
* **Actores Secundarios:** Ninguno.
* **Ámbito (Scope):** Módulo de Clientes.
* **Nivel (Level):** Objetivo de usuario.
* **Requisitos relacionados:** BR3, BR4, UR2, FR4, FR4.1, FR18, NFR1, NFR7, NFR16.
  
#### Interesados e Intereses
* **Usuario operativo:** Dar de alta un cliente de forma rápida y sin errores.
* **Empresa:** Tener los datos fiscales correctos para facturar y un registro de los cambios de cada cliente.
  
#### Condiciones y Garantías
* **Precondiciones:** El usuario debe estar autenticado y tener permiso de crear clientes.
* **Garantía de Éxito:** El cliente queda registrado o actualizado con sus contactos, y el cambio queda en su historial.
* **Garantía Mínima:** No se guarda un cliente con datos fiscales obligatorios incompletos ni duplicado. Al dar de baja un cliente, sus facturas y PDFs siguen accesibles.

#### Flujos de Eventos
**Disparador:** El usuario selecciona para crear un nuevo cliente o abre la ficha de un cliente existente.
 
**Escenario Principal de Éxito (alta):**
1. El usuario introduce los datos comerciales (nombre, sector, ubicación, responsable interno).
2. El usuario introduce los datos fiscales (razón social, NIF/CIF, dirección fiscal, régimen de impuestos).
3. El usuario añade uno o varios contactos (nombre, cargo, email, teléfono, canal preferido, estado).
4. El sistema valida los datos obligatorios y comprueba que el NIF/CIF no esté ya registrado.
5. El sistema guarda el cliente, lo asigna al responsable y lo registra en el historial.
6. El sistema ofrece configurar la suscripción del cliente.
   
**Extensiones (Flujos Alternativos):**
* **4a. Datos obligatorios incompletos o inválidos:**
  * 4a1. El sistema marca los campos erróneos y no guarda hasta que se corrijan.
* **4b. Cliente duplicado:**
  * 4b1. El sistema avisa de que ya existe un cliente con ese NIF/CIF y ofrece abrir su ficha (o reactivarlo si está dado de baja).
* **Modificación de un cliente:**
  * El usuario edita los datos y guarda.
  * Si otro usuario guardó el mismo cliente entretanto, el sistema rechaza el guardado con aviso de conflicto.
* **Reactivación de un cliente dado de baja:**
  * El usuario selecciona un cliente en estado "Baja" y elige "Reactivar".
  * El sistema cambia el estado a "Activo", conserva su historial previo y ofrece configurar una nueva suscripción.
    
</details>

---
