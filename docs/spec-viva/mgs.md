# Cuentas y acceso

## Purpose

Permitir que una persona cree una cuenta, inicie y cierre sesión y consulte sus datos, y mantener las pantallas y operaciones privadas fuera del alcance de quien no tiene una sesión válida, tanto en la API como en la interfaz web.

## Requirements

### Requirement: Registro de cuenta por API

La API SHALL crear una cuenta con `POST /api/v1/auth/signup` a partir de `fullName`, `email`, `password` y `passwordConfirmation`, y SHALL devolver en la misma respuesta los datos del usuario creado y un token de acceso ya utilizable.

#### Scenario: Registro con datos válidos

- **WHEN** se envía un email no registrado, una contraseña de entre 8 y 32 caracteres, la misma contraseña como confirmación y un nombre
- **THEN** la respuesta es 200 con `{ "data": { "user": {...}, "token": "<cadena>" } }` y la cuenta queda creada

#### Scenario: El token del registro sirve de inmediato

- **WHEN** se usa el token recibido en el registro como `Authorization: Bearer <token>` en la consulta del perfil
- **THEN** la respuesta es 200 con los datos de la cuenta recién creada, sin ningún paso intermedio de inicio de sesión

#### Scenario: Registro sin nombre

- **WHEN** se envía `fullName` con valor `null`, cadena vacía o solo espacios, junto con el resto de datos válidos
- **THEN** la cuenta se crea y el usuario devuelto tiene `fullName: null`

### Requirement: Validación de los datos de registro

La API SHALL rechazar con 422 un registro cuyos datos no cumplan las reglas, SHALL informar de todos los campos inválidos en una sola respuesta, con como mucho un error por campo, y SHALL NOT crear la cuenta.

#### Scenario: Falta la clave del nombre

- **WHEN** el cuerpo no incluye la clave `fullName` y el resto de datos es válido
- **THEN** la respuesta es 422 con un error `rule: "required"` en `field: "fullName"`

#### Scenario: Cuerpo vacío

- **WHEN** se envía `{}`
- **THEN** la respuesta es 422 con un error `rule: "required"` para cada uno de `fullName`, `email`, `password` y `passwordConfirmation`

#### Scenario: Email con formato inválido

- **WHEN** el email no tiene formato de dirección de correo, por ejemplo `nope`
- **THEN** la respuesta es 422 con un error `rule: "email"` en `field: "email"`

#### Scenario: Contraseña demasiado corta

- **WHEN** la contraseña y su confirmación tienen menos de 8 caracteres
- **THEN** la respuesta es 422 con un error `rule: "minLength"` y `meta: { "min": 8 }` en `password` y otro igual en `passwordConfirmation`

#### Scenario: Contraseña demasiado larga

- **WHEN** la contraseña y su confirmación tienen 33 caracteres o más
- **THEN** la respuesta es 422 con un error `rule: "maxLength"` y `meta: { "max": 32 }` en `password` y otro igual en `passwordConfirmation`

#### Scenario: Contraseña de exactamente 32 caracteres

- **WHEN** la contraseña y su confirmación tienen exactamente 32 caracteres
- **THEN** la cuenta se crea con normalidad

#### Scenario: La confirmación no coincide

- **WHEN** la contraseña y la confirmación tienen una longitud válida pero son distintas
- **THEN** la respuesta es 422 con un error `rule: "sameAs"` en `field: "passwordConfirmation"` y `meta: { "otherField": "password" }`

#### Scenario: La longitud se comprueba antes que la coincidencia

- **WHEN** la confirmación es más corta de 8 caracteres y además distinta de la contraseña
- **THEN** el error de `passwordConfirmation` es `rule: "minLength"` y no aparece ningún error `sameAs`

### Requirement: Unicidad del email

La API SHALL impedir que dos cuentas tengan exactamente el mismo email, y la comparación SHALL distinguir mayúsculas de minúsculas.

#### Scenario: Email ya registrado

- **WHEN** se intenta registrar un email idéntico al de una cuenta existente
- **THEN** la respuesta es 422 con un error `rule: "database.unique"` en `field: "email"` y no se crea ninguna cuenta

#### Scenario: Email que solo difiere en mayúsculas

- **WHEN** existe una cuenta con `ada@example.com` y se registra `ADA@example.com`
- **THEN** se crea una segunda cuenta, distinta de la primera, con el email tal como se envió

### Requirement: Recorte de espacios en los datos de entrada

La API SHALL quitar los espacios del principio y del final de cada texto recibido antes de validarlo, y SHALL tratar como nulo un texto que queda vacío tras el recorte.

#### Scenario: Email con espacios alrededor

- **WHEN** se registra o se inicia sesión con `"  ada@example.com "`
- **THEN** la operación se resuelve como si se hubiera enviado `ada@example.com`

#### Scenario: Contraseña con espacios alrededor

- **WHEN** una cuenta se registra con la contraseña `"  abcdefgh  "` y después se inicia sesión con `abcdefgh`
- **THEN** el inicio de sesión tiene éxito

#### Scenario: La longitud se mide después de recortar

- **WHEN** se registra la contraseña `" abcdef "`, de 8 caracteres contando los espacios
- **THEN** la respuesta es 422 con `rule: "minLength"`, porque tras el recorte quedan 6

#### Scenario: Texto vacío en un campo obligatorio

- **WHEN** se envía `email` o `password` como cadena vacía
- **THEN** la respuesta es 422 con `rule: "required"` en ese campo

### Requirement: Inicio de sesión por API

La API SHALL emitir un token de acceso nuevo con `POST /api/v1/auth/login` cuando el email y la contraseña correspondan a una cuenta, y SHALL devolver junto a él los datos del usuario.

#### Scenario: Credenciales correctas

- **WHEN** se envían el email y la contraseña de una cuenta existente
- **THEN** la respuesta es 200 con `{ "data": { "user": {...}, "token": "<cadena>" } }`

#### Scenario: Cada inicio de sesión emite un token distinto

- **WHEN** la misma cuenta inicia sesión dos veces seguidas
- **THEN** recibe dos tokens diferentes y los dos permiten consultar el perfil

### Requirement: Rechazo de credenciales incorrectas

La API SHALL responder 400 con `{ "errors": [{ "message": "Invalid user credentials" }] }` cuando el email y la contraseña no correspondan a ninguna cuenta, y SHALL NOT distinguir en la respuesta si lo que falla es el email o la contraseña.

#### Scenario: Contraseña incorrecta

- **WHEN** se envía el email de una cuenta existente con una contraseña que no es la suya
- **THEN** la respuesta es 400 con el mensaje `Invalid user credentials` y no se emite ningún token

#### Scenario: Email no registrado

- **WHEN** se envía un email con formato válido que no pertenece a ninguna cuenta
- **THEN** la respuesta es idéntica a la de la contraseña incorrecta

#### Scenario: Email con distinta capitalización

- **WHEN** existe la cuenta `ada@example.com` y se inicia sesión con `Ada@example.com` y la contraseña correcta
- **THEN** la respuesta es 400 con `Invalid user credentials`

#### Scenario: Contraseña más larga que el máximo del registro

- **WHEN** se inicia sesión con una contraseña de 40 caracteres
- **THEN** la respuesta es 400 por credenciales incorrectas, no 422: el inicio de sesión no limita la longitud de la contraseña

### Requirement: Validación de los datos de inicio de sesión

La API SHALL rechazar con 422 un inicio de sesión al que le falte el email o la contraseña, o cuyo email no tenga formato válido, antes de comprobar las credenciales.

#### Scenario: Cuerpo vacío

- **WHEN** se envía `{}` al inicio de sesión
- **THEN** la respuesta es 422 con un error `rule: "required"` en `email` y otro en `password`

#### Scenario: Email con formato inválido

- **WHEN** se envía `nope` como email y cualquier contraseña
- **THEN** la respuesta es 422 con un error `rule: "email"` en `field: "email"`

### Requirement: Consulta del perfil por API

La API SHALL devolver con `GET /api/v1/account/profile` los datos de la cuenta a la que pertenece el token enviado como `Authorization: Bearer <token>`.

#### Scenario: Token válido

- **WHEN** se consulta el perfil con un token emitido por el registro o por el inicio de sesión que no se ha revocado
- **THEN** la respuesta es 200 con `{ "data": {...} }` y los datos del dueño del token

### Requirement: Protección de las operaciones de cuenta

La API SHALL responder 401 con `{ "errors": [{ "message": "Unauthorized access" }] }` a la consulta del perfil y al cierre de sesión cuando la petición no lleve un token válido.

#### Scenario: Sin cabecera de autorización

- **WHEN** se consulta el perfil o se cierra sesión sin cabecera `Authorization`
- **THEN** la respuesta es 401 con `Unauthorized access`

#### Scenario: Token inventado

- **WHEN** se envía `Authorization: Bearer` con un valor que la API no ha emitido
- **THEN** la respuesta es 401 con `Unauthorized access`

#### Scenario: Token sin el esquema Bearer

- **WHEN** se envía un token válido en `Authorization` sin el prefijo `Bearer`
- **THEN** la respuesta es 401 con `Unauthorized access`

### Requirement: Cierre de sesión por API

La API SHALL revocar con `POST /api/v1/account/logout` únicamente el token con el que se hace la petición, y SHALL responder 200 con `{ "message": "Logged out successfully" }`, sin el envoltorio `data`.

#### Scenario: Cierre de sesión con un token válido

- **WHEN** se cierra sesión con un token válido
- **THEN** la respuesta es 200 con `{ "message": "Logged out successfully" }`, y a partir de ese momento ese token recibe 401 en cualquier operación protegida

#### Scenario: Otros tokens de la misma cuenta siguen vivos

- **WHEN** una cuenta tiene dos tokens activos y cierra sesión con uno de ellos
- **THEN** el otro sigue permitiendo consultar el perfil

#### Scenario: Cerrar sesión dos veces con el mismo token

- **WHEN** se repite el cierre de sesión con un token ya revocado
- **THEN** la respuesta es 401 con `Unauthorized access`

### Requirement: Representación del usuario en la API

La API SHALL representar al usuario, allí donde lo devuelva, con exactamente los campos `id`, `fullName`, `email`, `createdAt`, `updatedAt` e `initials`, y SHALL NOT incluir nunca la contraseña.

#### Scenario: Usuario devuelto por registro, inicio de sesión o perfil

- **WHEN** cualquiera de las tres operaciones devuelve un usuario
- **THEN** el objeto trae `id` numérico, `fullName` (texto o `null`), `email`, `createdAt` y `updatedAt` como fechas ISO 8601 con desfase horario, e `initials`, y no trae ningún campo de contraseña

### Requirement: Iniciales del usuario

La API SHALL calcular `initials` en mayúsculas a partir del nombre, si existe, o del email, si no.

#### Scenario: Nombre de dos o más palabras

- **WHEN** el nombre es `Ada Lovelace`
- **THEN** `initials` es `AL`

#### Scenario: Nombre de una sola palabra

- **WHEN** el nombre es `ada`
- **THEN** `initials` es `AD`, las dos primeras letras

#### Scenario: Palabras separadas por más de un espacio

- **WHEN** el nombre es `ada  lovelace`, con dos espacios entre las palabras
- **THEN** `initials` es `AD`, las dos primeras letras de la primera palabra

#### Scenario: Cuenta sin nombre

- **WHEN** la cuenta no tiene nombre y su email es `mike@gmail.com`
- **THEN** `initials` es `MG`: la primera letra de lo que va antes de la arroba y la primera de lo que va después

### Requirement: Formato común de las respuestas de la API

La API SHALL responder siempre en JSON, envolver en `data` los resultados del registro, del inicio de sesión y del perfil, y devolver los errores como una lista `errors` cuyos mensajes van en inglés.

#### Scenario: Error de validación

- **WHEN** una petición falla la validación
- **THEN** cada elemento de `errors` trae `message`, `rule` y `field`, y `meta` cuando la regla tiene parámetros (`min`, `max`, `otherField`)

#### Scenario: El cliente pide HTML

- **WHEN** se consulta el perfil sin token y con `Accept: text/html`
- **THEN** la respuesta sigue siendo JSON: 401 con `Unauthorized access`

#### Scenario: Cuerpo que no es JSON válido

- **WHEN** se envía al registro un cuerpo con `Content-Type: application/json` que no es JSON bien formado
- **THEN** la respuesta es 400

### Requirement: Pantalla de registro

La interfaz SHALL ofrecer en `/register` un formulario con los campos «Nombre completo (opcional)», «Email», «Contraseña», acompañado de la pista «Entre 8 y 32 caracteres.», y «Repite la contraseña», un botón «Crear cuenta» y un enlace «Inicia sesión» que lleva a la pantalla de inicio de sesión.

#### Scenario: Registro correcto

- **WHEN** una persona sin sesión rellena el formulario con datos válidos y pulsa «Crear cuenta»
- **THEN** queda con la sesión iniciada y ve su perfil, sin pasar por la pantalla de inicio de sesión

#### Scenario: Las contraseñas no coinciden

- **WHEN** la persona escribe una contraseña y una repetición distintas y pulsa «Crear cuenta»
- **THEN** ve «Las contraseñas no coinciden.» bajo «Repite la contraseña», la cuenta no se crea y no aparece ningún otro mensaje aunque otros campos también estén mal

#### Scenario: Nombre en blanco

- **WHEN** la persona deja «Nombre completo» vacío o con solo espacios y el resto es válido
- **THEN** la cuenta se crea y su perfil muestra «Sin nombre»

#### Scenario: Envío en curso

- **WHEN** la persona pulsa «Crear cuenta» y la respuesta aún no ha llegado
- **THEN** el botón muestra «Creando cuenta…» y no se puede volver a pulsar

### Requirement: Pantalla de inicio de sesión

La interfaz SHALL ofrecer en `/login` un formulario con «Email» y «Contraseña», un botón «Entrar» y un enlace «Crea una» que lleva a la pantalla de registro, bajo el título «Inicia sesión».

#### Scenario: Inicio de sesión correcto

- **WHEN** una persona sin sesión introduce el email y la contraseña de su cuenta y pulsa «Entrar»
- **THEN** queda con la sesión iniciada y ve su perfil

#### Scenario: Credenciales incorrectas

- **WHEN** la persona introduce un email o una contraseña que no corresponden a ninguna cuenta
- **THEN** ve un aviso destacado sobre el formulario con «El email o la contraseña no son correctos.» y sigue en la pantalla de inicio de sesión

#### Scenario: Envío en curso

- **WHEN** la persona pulsa «Entrar» y la respuesta aún no ha llegado
- **THEN** el botón muestra «Entrando…» y no se puede volver a pulsar

### Requirement: Mensajes de error junto a cada campo

La interfaz SHALL mostrar en castellano los errores de validación del registro y del inicio de sesión bajo el campo al que se refieren, uno por campo, y SHALL NOT mostrar además un aviso general cuando todos los errores tienen su campo en pantalla.

#### Scenario: Email ya registrado

- **WHEN** la persona intenta registrarse con un email que ya tiene cuenta
- **THEN** ve bajo «Email» el mensaje «Ese email ya está registrado. Inicia sesión en su lugar.»

#### Scenario: Email con formato inválido

- **WHEN** la persona envía cualquiera de los dos formularios con un email sin formato válido
- **THEN** ve bajo «Email» el mensaje «Introduce una dirección de email válida.»

#### Scenario: Campos obligatorios vacíos

- **WHEN** la persona envía un formulario con el email, la contraseña o la repetición vacíos
- **THEN** ve bajo cada uno «Falta rellenar el email.», «Falta rellenar la contraseña.» o «Falta rellenar la confirmación de la contraseña.», según corresponda

#### Scenario: Contraseña fuera de longitud en el registro

- **WHEN** la persona se registra con una contraseña, repetida igual, de menos de 8 caracteres o de más de 32
- **THEN** ve bajo «Contraseña» «la contraseña debe tener al menos 8 caracteres.» o «la contraseña no puede superar los 32 caracteres.», en minúscula inicial, y un mensaje equivalente sobre la confirmación bajo «Repite la contraseña»

### Requirement: Errores del servidor en los formularios

La interfaz SHALL explicar en castellano, en un aviso sobre el formulario, los fallos que no se deben a un campo concreto.

#### Scenario: Servidor inalcanzable

- **WHEN** la persona envía el registro o el inicio de sesión y no hay conexión con el servidor
- **THEN** ve «No se pudo conectar con el servidor. Comprueba que el backend está arrancado.»

#### Scenario: Error interno del servidor

- **WHEN** el servidor responde con un error no previsto
- **THEN** la persona ve «Algo ha ido mal en el servidor. Inténtalo de nuevo en un momento.»

### Requirement: Pantalla de perfil

La interfaz SHALL mostrar en `/profile`, a quien tiene sesión, sus iniciales en un círculo, su nombre, o «Sin nombre» si no tiene, su email, la fecha de alta como «Miembro desde» en formato largo en castellano y un botón «Cerrar sesión».

#### Scenario: Persona con nombre

- **WHEN** una persona con sesión llamada Ada Lovelace abre su perfil
- **THEN** ve «AL» en el círculo, «Ada Lovelace», su email y, por ejemplo, «Miembro desde 30 de septiembre de 2026»

#### Scenario: Persona sin nombre

- **WHEN** una persona registrada sin nombre abre su perfil
- **THEN** ve «Sin nombre» como título y en el círculo las iniciales derivadas de su email

### Requirement: Cierre de sesión en la interfaz

La interfaz SHALL cerrar la sesión en el navegador al pulsar «Cerrar sesión» y llevar a la persona a la pantalla de inicio de sesión, aunque el servidor no confirme el cierre.

#### Scenario: Cierre de sesión

- **WHEN** una persona con sesión pulsa «Cerrar sesión»
- **THEN** el botón muestra «Cerrando sesión…» y a continuación ve la pantalla de inicio de sesión, sin ningún aviso

#### Scenario: Volver atrás tras cerrar sesión

- **WHEN** tras cerrar sesión la persona usa el botón «atrás» del navegador
- **THEN** no vuelve a ver su perfil y acaba en la pantalla de inicio de sesión

#### Scenario: Servidor inalcanzable al cerrar sesión

- **WHEN** la persona pulsa «Cerrar sesión» sin conexión con el servidor
- **THEN** igualmente ve la pantalla de inicio de sesión, sin ningún mensaje de error

### Requirement: Acceso a las pantallas según la sesión

La interfaz SHALL mostrar el perfil solo a quien tiene sesión, y SHALL mostrar las pantallas de registro e inicio de sesión solo a quien no la tiene.

#### Scenario: Sin sesión, a una pantalla privada

- **WHEN** una persona sin sesión abre `/profile` o cualquier dirección que no sea `/login` ni `/register`
- **THEN** acaba en la pantalla de inicio de sesión

#### Scenario: Con sesión, a una pantalla de acceso

- **WHEN** una persona con sesión abre `/login` o `/register`
- **THEN** acaba en su perfil

#### Scenario: Con sesión, a una dirección desconocida

- **WHEN** una persona con sesión abre una dirección que la aplicación no conoce
- **THEN** acaba en su perfil

### Requirement: Persistencia de la sesión en el navegador

La interfaz SHALL conservar la sesión entre recargas y entre aperturas de la aplicación en el mismo navegador, y SHALL comprobarla con el servidor al cargar, antes de mostrar ninguna pantalla.

#### Scenario: Recarga con sesión

- **WHEN** una persona con sesión recarga la página, o cierra la pestaña y vuelve a abrir la aplicación
- **THEN** ve un indicador de carga mientras se comprueba la sesión y después su perfil, sin volver a introducir credenciales

### Requirement: Explicación de una sesión perdida

La interfaz SHALL explicar en la pantalla de inicio de sesión por qué no se ha podido recuperar una sesión guardada, y SHALL conservar esa sesión cuando el fallo no se deba a que el servidor la haya rechazado.

#### Scenario: El servidor rechaza la sesión guardada

- **WHEN** al abrir la aplicación el servidor responde que la sesión guardada no es válida, por ejemplo porque su token ya se revocó
- **THEN** la persona ve la pantalla de inicio de sesión con el aviso «Tu sesión ha caducado. Vuelve a iniciar sesión.» y la sesión guardada se descarta

#### Scenario: El servidor no responde al abrir la aplicación

- **WHEN** al abrir la aplicación con una sesión guardada no hay conexión con el servidor
- **THEN** la persona ve la pantalla de inicio de sesión con el aviso «No se pudo conectar con el servidor. Comprueba que el backend está arrancado.», y si recarga cuando el servidor vuelve, entra en su perfil sin introducir credenciales

#### Scenario: Un nuevo intento sustituye al aviso

- **WHEN** se muestra el aviso de sesión perdida y la persona hace un intento de inicio de sesión que falla
- **THEN** el aviso pasa a mostrar el motivo del nuevo fallo en lugar del anterior

## 🅱 Parte B: las tres listas

## 1. Cuántos requisitos escribió el agente, y cuántos comprobaste tú abriendo el código

- El agente escribió 22 requisitos
- Cantidad de requisitos comprobados: 18

## 2. Las incoherencias que aparecieron al escribirla

- No se normaliza los email, por lo que un mismo correo pero con letras minúsculas/mayúsculas diferentes, crearán cuentas distintas
- La API quita los espacios del principio y del final de todos los textos, también de la contraseña.
- logout y GET / responden sin { data }, y logout además en inglés, a pesar de la convención de pasar todo por serialize().
- Cualquier 400 se traduce como «email o contraseña incorrectos», aunque en el futuro haya otros 400 que no tengan nada que ver
- Contraseña: el registro limita a 32 caracteres; el login no pone límite.
- Tokens: cada login acumula un token nuevo y el logout solo revoca el actual, así que «cerrar sesión» no cierra las demás sesiones abiertas.
- fullName: no tiene límite de longitud ni se recorta en el backend; el recorte (trim) solo lo hace el frontend.
- updatedAt: puede ser null en el esquema, pero el tipo del frontend lo declara como string.

## 3. Lo que no supiste decidir si era un bug o el contrato

- El email no se normaliza: ni minúsculas ni trim. <Foo@x.com> y <foo@x.com> podrían ser dos cuentas distintas, y el login distingue mayúsculas. No queda claro si es una decisión o un descuido.
- Cada login crea un token nuevo y el logout solo invalida el actual.
