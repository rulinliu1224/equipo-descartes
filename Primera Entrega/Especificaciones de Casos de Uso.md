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
* **Descripción:** El usuario operativo administra los planes/tarifas de un cliente o procesa la solicitud de baja. Si existen facturas pendientes de pago, no se permite dar de baja al cliente ni detener la generación de sus facturas recurrentes hasta recibir la confirmación del pago pendiente.
* **Actor Principal:** Usuario operativo.
* **Ámbito (Scope):** Sistema de Facturación.
* **Nivel (Level):** Objetivo de usuario.

#### Interesados e Intereses
* **Cliente:** Solicitar la baja o el cambio de su suscripción.
* **Empresa:** Cobrar todas las deudas pendientes antes de dar la baja definitiva.

#### Condiciones y Garantías
* **Precondiciones:**
  * El usuario debe estar autenticado en el sistema.
  * El cliente debe existir en la base de datos.
* **Garantía de Éxito:** La suscripción es actualizada o el cliente se da de baja si no presenta facturas impagas.
* **Garantía Mínima:** No se permite la baja total si existen facturas vencidas sin saldar.

#### Flujos de Eventos
* **Disparador:** El usuario selecciona un cliente e intenta gestionar la suscripción o dar de baja.

**Escenario Principal de Éxito:**
1. El usuario busca y selecciona al cliente.
2. El sistema muestra el estado actual de la suscripción y del historial de pagos.
3. El usuario solicita tramitar la baja del cliente.
4. El sistema verifica que el cliente tenga todas sus facturas al día (cobros pendientes == 0).
5. El sistema registra la baja del cliente y cancela la suscripción en el sistema y en Stripe a través de la api.
6. El sistema emite la confirmación de la baja a los interesados (Cliente y Administrador)

**Extensiones (Flujos Alternativos):**
***4a. El cliente presenta facturas pendientes/impagas (Intento de dar de baja cliente con deudas):**
  * 4a1. El sistema rechaza la solicitud de baja e informa que hay deudas pendientes por saldar.
  * 4a2. El sistema mantiene al cliente en estado activo, **continuando con la generación habitual de facturas recurrentes**.
<!-- TODO: Definir si se hace un estado intermedio de cliente activo pero sin bloqueando la generación de facturas o cliente activo pero continuando con la generación habitual -->
  * 4a3. El sistema continúa enviando notificaciones y recordatorios automáticos de impago al cliente.
  * 4a4. Al recibir la confirmación de pago de la factura pendiente (vía Webhook de Stripe), el sistema habilita nuevamente la opción de procesar la baja definitiva.
<!-- TODO: Definir si se hace de forma automatica la baja del cliente en el sistema o se envia una notificación al Administrador y que este lo de de baja. -->
</details>

---

<details>
<summary><b>CU-03: Generar Facturas Recurrentes Automáticas</b></summary>

### CU-03: Generar Facturas Recurrentes Automáticas y Prorrrateo

* **ID / Código:** `CU-03`
* **Nombre del Caso de Uso:** Generar facturas recurrentes automático el día 1 con prorrateo
* **Descripción:** Procesar de forma automática al inicio de cada mes (día 1) la emisión de **exactamente una factura mensual** por cada cliente con suscripción activa, sin importar cuántas facturas anteriores permanezcan impagas (mientras el cliente no esté dado de baja). Además, al cumplirse un año desde la primera factura del cliente, el sistema evalúa si corresponde un cambio de plan o tarifa y aplica el **prorrateo** correspondiente sobre la nueva factura.
* * **Actor Principal:** Sistema (proceso programado) / Administrador.
* **Actores Secundarios:** Servicio de email / WhatsApp (`<<sistema externo>>`).
* **Ámbito (Scope):** Módulo de Facturación Automática.
* **Nivel (Level):** Subfunción / Objetivo de sistema.

#### Interesados e Intereses
* **Empresa:** Garantizar la emisión de una factura mensual por cliente activo y asegurar la actualización de tarifas/planes al cumplir el ciclo anual con su prorrateo correspondiente.
* **Cliente:** Recibir mensualmente su factura y ver reflejados de forma clara los ajustes proporcionales en caso de cambio de tarifa al cumplirse el año.

#### Condiciones y Garantías
* **Precondiciones:** El cliente debe tener una suscripción activa (no estar dado de baja).
* **Garantía de Éxito:** Se emite la factura mensual de cada cliente activo, aplicando el recálculo y prorrateo anual si corresponde, y se notifica al cliente
* **Garantía Mínima:** Si la generación o emisión de la factura de un cliente específico falla, el sistema registra la incidencia, notifica instantaneamente al administrador o usuario responsable del cliente, pero no se detiene el procesamiento general, permitiendo que el sistema continúe emitiendo las facturas del resto de los clientes activos sin interrupciones.

#### Flujos de Eventos
**Disparador:** Llega el día 1 de cada mes a las 00:00 h o ejecución manual por parte del Adminidtrador.
  
**Escenario Principal de Éxito:**
1. El sistema identifica a todos los clientes que mantienen su suscripción activa para el periodo (independientemente de si registran deudas o facturas impagas anteriores).
2. El sistema verifica si el cliente cumple **1 año desde la emisión de su primera factura**.
  * Si cumple el año: El sistema evalúa el cambio de plan/tarifa y calcula el **prorrateo proporcional** correspondiente para la nueva factura.
  * Si no cumple el año: El sistema aplica el cobro de la tarifa habitual contratada.
3. El sistema genera la factura del mes en curso y actualiza el saldo total deudor del cliente.
4. El sistema ejecuta el caso de uso `<<include>>` **Enviar facturas y recordatorios**.
5. El Servicio externo de Email/WhatsApp envía la nueva factura emitida al cliente.

**Extensiones (Flujos Alternativos):**
* **1a. El cliente acumula facturas de meses anteriores impagas:**
  * 1a1. El sistema continúa emitiendo la nueva factura individual del mes actual.
  * 1a2. El sistema suma el nuevo importe al saldo total adeudado por el cliente.
  * 1a3. El sistema intensifica el envío de recordatorios de cobro por la deuda acumulada.
* **2a. Detección de cambio de rango/tarifa en la revisión anual (`<<extend>>` Reorganizar/Revisar facturación anual):**
  * 2a1. El sistema detecta que el ajuste anual implica un cambio de tramo o valor de tarifa.
  * 2a2. El sistema aplica el cálculo prorrateo y notifica al Administrador sobre el ajuste realizado.
</details>

---

<details>
<summary><b>CU-04: Gestionar Facturas Puntuales</b></summary>

### CU-04: Gestionar Facturas Puntuales

* **ID / Código:** `CU-04`
* **Nombre del Caso de Uso:** Gestionar facturas puntuales (vista previa, PDF, emisión)
* **Descripción:** Permite a los usuarios administrativos crear y emitir cobros/facturas únicas bajo demanda para un cliente (por servicios extraordinarios o ventas aisladas no recurrentes).Aunque no son automáticas ni repetitivas, estas facturas se programan/emiten para efectuarse a principio del mes (día 1), consolidándose con el ciclo mensual de cobros. 
* **Actores Secundarios:** Servicio de email / WhatsApp (`<<sistema externo>>`).
* **Ámbito (Scope):** Módulo de Facturación Puntual.
* **Nivel (Level):** Objetivo de usuario.

#### Interesados e Intereses
* **Empresa:** Disponer de un mecanismo flexible para facturar conceptos únicos sin alterar el ciclo de suscripción recurrente del cliente.
* **Cliente:** Recibir el comprobante detallado de los servicios puntuales contratados con su correspondiente enlace de pago.

#### ⚙️ Condiciones y Garantías
* **Precondiciones:**
  * El usuario operativo debe estar autenticado (`CU-01`).
  * El cliente receptor debe existir registrado en la base de datos y estar como no recurrente.
* **Garantía de Éxito:** La factura puntual es emitida, guardada en la base de datos en estado "Pendiente de pago", sumada al historial/informes del cliente (`CU-06`) y enviada al cliente (`CU-05`).
* **Garantía Mínima:** Si la generación de la factura falla antes de la emisión, el sistema no actualiza los saldos ni dispara notificaciones al cliente, sino que avisa al Administrador.

#### 🔄 Flujos de Eventos
**Disparador:** El usuario selecciona un cliente y configura la generación de facturas como no recurrente (Factura puntual a principio de mes)

**Escenario Principal de Éxito:**
1. El usuario selecciona el cliente e ingresa los conceptos, importes, fecha de facturación y fecha de vencimiento del cobro puntual.
2. El sistema genera una **vista previa** del comprobante en formato PDF/pantalla.
3. El usuario confirma la aprobación y a principio de mes se genera la factura automaticamente.
4. Llegado el día de facturación, el sistema asigna la numeración oficial, establece el estado "Pendiente de pago" y actualiza los saldos deudores del cliente.
5. El sistema ejecuta el caso de uso `<<include>>` `CU-05` Enviar facturas y recordatorios.
6. El servicio externo notifica al cliente adjuntando la factura puntual y el enlace de pago.

**Extensiones (Flujos Alternativos):**
* **3a. Cancelación del proceso previo a la emisión:**
  * 3a1. El usuario decide cancelar el formulario antes de confirmar la emisión.
  * 3a2. El sistema descuarta los datos ingresados sin guardar ningún registro ni alterar la contabilidad del cliente.

</details>

---

<details>
<summary><b>CU-05: Enviar Facturas y Recordatorios de Pago</b></summary>

### CU-05: Enviar Facturas y Recordatorios de Pago

* **ID / Código:** `CU-05`
* **Nombre del Caso de Uso:** Enviar facturas y recordatorios de pago (`<<include>>`)
* **Descripción:** Subfunción encargada de enviar las notificaciones de emisión de facturas (tanto recurrentes como puntuales) y gestionar los recordatorios automáticos periódicos de cobro cuando una factura entra en periodo de pago o permanece impaga.
* **Actor Principal:** Sistema, proceso programado.
* **Actores Secundarios:** Servicio de email / WhatsApp (`<<sistema externo>>`).
* **Ámbito (Scope):** Módulo de Notificaciones.
* **Nivel (Level):** Subfunción / Proceso reutilizable.

#### Interesados e Intereses
* **Empresa:** Mantener una comunicación fluida de cobro y reducir envio de avisos manual mediante avisos automatizados.
* **Cliente:** Recibir sus comprobantes a tiempo y recordar los vencimientos pendientes de pago.

#### Condiciones y Garantías
* **Precondiciones:**
  * Existencia de una factura emitida (recurrente `CU-03` o puntual `CU-04`) en estado "Pendiente de pago" o "Mora"(Pasado de fecha de pago).
* **Garantía de Éxito:** Se envía la notificación o recordatorio correspondiente a través del canal configurado (Email/WhatsApp) y se registra el evento en el historial del cliente.
* **Garantía Mínima:** Si falla el envío por un canal (ej. WhatsApp), el sistema intenta el envío por el canal alternativo (Email) y registra el resultado en la bitácora, si ninguna de las dos funciona notificar al ADministrador.

#### Flujos de Eventos
**Disparador:** Invocación directa por la emisión de una factura o ejecución del programa diario de revisión de deudas.

**Escenario Principal de Éxito:**
1. El sistema recibe la orden de envío para una factura específica (por aviso de recordatorio programado).
2. El sistema genera el enlace de pago directo de Stripe y adjunta la factura en PDF.
3. El sistema envía el mensaje al cliente utilizando el Servicio externo de Email/WhatsApp.
4. El sistema registra la fecha y hora del envío en e historial de notificaciones.

**Extensiones (Flujos Alternativos):**
* **1a. Recordatorio periódico por facturas en mora:**
  * 1a1. El programa detecta facturas no pagadas que han superado la fecha de vencimiento.
  * 1a2. El sistema intensifica los avisos según la frecuencia configurada (ej. cada 5 días) incluyendo el detalle de la deuda total.

</details>

---

<details>
<summary><b>CU-06: Generar Informes y Reportes Financieros</b></summary>

### CU-06: Generar Informes y Reportes Financieros

* **ID / Código:** `CU-06`
* **Nombre del Caso de Uso:** Generar informes y reportes financieros
* **Descripción:** Permite a los administradores consultar y exportar métricas, reportes de facturación e informes de morosidad consolidando en una única fuente de datos tanto los ingresos/deudas procedentes de la **facturación recurrente** (`CU-03`) como de la **facturación puntual** (`CU-04`).
* **Actor Principal:** Administrador.
* **Actores Secundarios:** Ninguno.
* **Ámbito (Scope):** Módulo de Reportes e Inteligencia de Negocio.
* **Nivel (Level):** Objetivo de usuario.

#### Interesados e Intereses
* **Empresa / Administradores:** Disponer de una visión clara y unificada de balance de ingresos, proyección de cobros y cantidad pendiente a cobrar.

####  Condiciones y Garantías
* **Precondiciones:**
  * El Administrador debe estar autenticado con rol directivo/admin (`CU-01`).
  * El Administrador debe tener los permisos para visualizar y crear informes o reportes.
* **Garantía de Éxito:** El sistema consolida la información de ambas fuentes (recurrentes y puntuales) y muestra/exporta el reporte consolidado.
* **Garantía Mínima:** Si no hay datos en el rango de fechas seleccionado, el sistema muestra la pantalla sin errores indicando "Sin registros".

#### Flujos de Eventos
**Disparador:** El Administrador accede a la sección 'informes y reportes' y selecciona los parámetros de búsqueda.

**Escenario Principal de Éxito:**
1. El Administrador selecciona el tipo de informe deseado y el rango de fechas.
2. El sistema consulta las bases de datos agregando las cifras de facturas recurrentes y puntuales.
3. El sistema calcula los totales, saldos cobrados y saldos adeudados.
4. El sistema presenta en pantalla los gráficos y tablas explicativas.
5. El Administrador puede descargar el reporte en el formato deseado (PDF/Excel).

**Extensiones (Flujos Alternativos):**
* **1a. Rango de fechas no válido o inconsistente:**
  * 1a1. El Administrador ingresa una fecha de inicio posterior a la fecha de fin.
  * 1a2. El sistema notifica que el rango es inválido y solicita la corrección de las fechas.
* **2a. Inexistencia de datos para los filtros seleccionados:**
  * 2a1. El sistema realiza la consulta y no halla registros en el periodo indicado.
  * 2a2. El sistema despliega la interfaz con saldos en cero ($0) y muestra el mensaje *"Sin registros para los criterios seleccionados"*.
* **4a. Exceso en el volumen de datos para la vista en pantalla:**
  * 4a1. La consulta abarca un volumen masivo de datos que ralentiza la visualización web.
  * 4a2. El sistema procesa el reporte de forma asíncrona y envía el archivo generado al correo del Administrador.
* **5a. Error durante la exportación del documento:**
  * 5a1. Se produce una falla al compilar el archivo PDF o Excel.
  * 5a2. El sistema muestra una alerta de error de descarga y permite reintentar el proceso.

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
