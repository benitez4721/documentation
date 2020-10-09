
# IdentityMind
IdentityMind ofrece una Plataforma SaaS (software as a service), para la gestión de riesgo en linea y automatización de verificación de cumplimientos legales. Ayuda a empresas a reducir el fraude en la incoporacion de clientes y transacciones, y mejora la utilización de AML (Anti-money laudering), KYC (know your custumer) y listas de sancionados. Esta plataforma construye valida y evalua continuamente las identidades digitales atraves de la tecnología eDNA ™ para garantizar la seguridad comercial global, y cumplimentos legislativos del cliente desde su incoporacion y durante todo el ciclo de vida en la plataforma. Realizan un seguimiento seguro de las entidades involucradas en cada transacción( por ejemplo, consumidores, comerciantes, titulares de tarjetas, billeteras de pago, métodos alternativos de pago), para construir una reputación de pago y permitir que las empresas pueden reducir el potencial  fraude y blanqueo de capitales.

Recibe información transaccional (pagos, transferencias, KYC, KYB) que comparan con su base de datos de identidad y la evalua con la política de riesgo y cumplimiento que el cliente ha creado.

A continuación se muestra una interacción típica de comerciante / IdentityMind para validar una transacción:

![](https://files.readme.io/bd2d2c0-v9qj1DP.png)

## EndPoints para validar datos del cliente
### Solicitud KYC del cliente

**URL**: `https://sandbox.identitymind.com/im/account/consumer`

**Method**: `POST`.

**Body params** (según los parámetros que sean enviados con la petición, identityMind ejecutara los test de seguridad correspondientes para validar la informacion):
- Parametros requeridos para validar la **Geolocalizacion**:
   - **`Ip <string>(requerido)`** : dirección IP del usuario
      
      Utiliza al proveedor **MaxMind** para efectuar esta validación, mediante la ip puede detectar mediante la verificación de geolocalización países de alto riesgo y discrepancias entre los puntos específicos de la ubicación.

- Parametros requeridos para validar el **Correo Electronico**:
   - **`tea <string> (requerido)**`: Email del usuario

- Parametros requeridos para validar el **Telefono Celular**
   - **`pm <string>  (requerido)`**: Telefono Celular del usuario

- Parametros requridos para validar la **Versión digital identificación oficial vigente**
   - **`scanData <Sting>  (requerido)`**: Imagen de la parte frontal de la identificación,  codificada en Base64 y con un tamaño máximo de 5MB
   
   - **`doctype <string>  (requerido)`**: Tipo del documento. Pasaporte(PP) | Licencia de conducir(DL)  | Tarjeta de identidad emitida por el gobierno(ID) | Permiso de residencia(RP) | Factura de servicios públicos(UB)
   
   - **`docCountry <sring>  (requerido)`**: Pais en el que se emitio el documento
   
   - **`backsideImageData <string> (opcional)`**: Imagen de la parte de atrás del documento, codificada en Base64 y con un tamano máximo de 5MB
   
   - **`bfn <string> (opcional)`**: Primer nombre de facturación
   
   - **`bln <string> (opcional)`**: Apellido de facturación
   
   - **`dob <string> (opcional)`**: fecha de nacimiento codificada como unix timestamp o un string en formato ISO8601
 
   Utiliza como proveedor de identificación de documentos a **Acuant** o **Mitek**

- Parametros requeridos para la **validacion biométrica de fotografía del rostro del cliente**

   -  **`faceImages <array of strings> (requerido)`**: Imagen de la cara del cliente, con la información de la imagen codificada en Base64 en un arreglo en formato JSON.
   
- Verificacion de **Blacklist**:
   
   - **`bfn <string> (requerido): Primer nombre de facturación`**
   
   - **`bln <string> (requerido): Apellido de facturación`**
   
   Este proceso se realiza mediante el servicio de sanction screening

- Validacion de **Clave Unica de Registro de Poblacion y clave de elector**
   
   - **`bfn <string> (requerido)`**: Primer nombre de facturación`
   
   - **`bln <string> (requerido)`**: Apellido de facturación
   
   - **`bmn <string> (requerido)`** : Segundo nombre
   
   - **`dob <string> (requerido)`** : Fecha de nacimiento
   
   - **`bsn <string> (requerido)`** : Direccion de residencia. Incluye numero de edificio, nombre de calle y numero de apartamento
   
   Para este proceso IDM utiliza el proveedor de Circulo de crédito, que es un servicio de verificacion de identidad que permite realizar validaciones contra instituciones mexicanas

### Agregar un documento a una solicitud KYC

**URL**: `https://sandbox.identitymind.com/im/account/consumer/appId/files`

**Method**: `POST`

**Path Params**:
   
   - **`appId <string> (requerido)`**: Identificador de la solicitud KYC 
  
**Body Params**:
 
   - **`File <file> (requerido)`**: Documento a incluir en la solicitud
   
   - **`Description <string> (opcional)`**: Breve descripcion   
   
## Proceso de validacion de documentos
   La plataforma de indetityMind  utiliza una interfaz REST con datos codificados JSON. Al enviar los datos del formulario KYC mediante su Api, se analiza esta información tomando en cuenta datos de identidad, políticas de riesgo y cumplimientos legales que previamente se configuraron en la plataforma.  
Poseen distintos proveedores de servicios para la verificacion de documentos, y utilizan un algoritmo para dirigir las peticiones a los mejores proveedores de pendiendo de la tarea a realizar.
Para explicar la forma en que la plataforma opera se tienen tres casos de usos:

### Identificacion positiva
   Se asegura de que los documentos suminstrados sean autenticos, validos y que los datos que contienen coincidan con los datos suminstrados por el cliente,  algunas de la pruebas de seguridad que se pueden realizar a través de la plataforma son las siguientes:

- `Security Test DV:3`. Autenticacion de documento
- `Security Test DV:8`. Coincidencia de nombre
- `Security Test DV:9`. Fecah de nacimiento
- `Security Test DV:10`. Coincidencia de direccion
- `Security Test DV: 11`. Documento expirado

La plataforma también ayuda a combatir la falsificación y robo de identidad; Esto se logra mediante el test de seguridad de  coincidencia de rostro `Security Test DV: 5`, el cual compara la foto del documento con una foto del cliente, la cual puede ser suministra por un archivo de imagen o como una “selfie”.

### Identificacion de riesgo
   Una serie de fallos en la validación de un documento puede ser considerada un riesgo. El test de seguridad de Autenticacion de documento puede fallar por varias razones; un caso común es que la imagen carece de suficiente resolución, a pesar de que el documento puede ser reconocido, tal vez los datos no se pueden leer, para este caso se utiliza el test de seguridad de documento desicivo `Security Test DV: 21`.
En caso de que no sea permitido la incorporación de usuarios de algunos países, se debe verificar que el  tipo de documento y la ubicación cumplan con las normativas del negocio, para lograr esto se tiene que tomar en cuenta lo siguiente:

   1- El país extraído del documento no coincide con el país proporcionado en la API (La API permite especificar cuales países son permitidos), para ello la plataforma pueden ejecutar los siguientes test de seguridad:
   
   - `Security Test DV:20`. Coincidencia de pais en el documento
   - `Security Test DV:18`. Pais ID
   
   2- El tipo de documento no ofrece suficiente información para verificar la localidad del usuario (la API permite especificar el tipo de documento requerido), para este caso la plataforma puede ejecutar los siguientes tests de seguridad:
   
   - `Security Test DV:2`. Coincidencia de tipo de documento
   - `Security Test DV:19`. Tipo de documento
   
### Volver a suministrar imagen
   
   En caso de que un usuario suministre una imagen de mala resolución, se puede automatizar una respuesta para este escenario. Para este escenario la plataforma puede ejecutar el siguiente test de seguridad.
   
   - `Security Test DV:1`: Documento Procesado
   
   Si los tests de seguridad `DV3` y `DV21` fallan, no se puede declarar el documento como autentico o falsificación, para esta situación se tienen dos opciones; o la imagen se analiza manualmente o se le pide al usuario volver a suministrar la imagen.
   
##  Cosas que no puede validar IDM   
   Con respecto a la validación de la credencial para votar, la documentación de IDM no epecifica si el proceso de verificación del código impreso en la credencial para votar y el Numero y año de emision es posible mediante el uso de servicio de Circulo de crédito o con algun otro proveedor, por lo tanto es necesario realizar pruebas en este sentido o buscar alguna otra solución.   
   
   
