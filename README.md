# IdentityMind
IdentityMind ofrece una Plataforma SaaS (software as a service), para la gestión de riesgo en línea y automatización de verificación de cumplimientos legales. Ayuda a empresas a reducir el fraude en la incorporación de clientes y transacciones, y mejora la utilización de AML (Anti-money laudering), KYC (know your custumer) y listas de sancionados. Esta plataforma construye valida y evalúa continuamente las identidades digitales a través de la tecnología eDNA ™ para garantizar la seguridad comercial global, y cumplimientos legislativos del cliente desde su incorporación y durante todo el ciclo de vida en la plataforma. Realizan un seguimiento seguro de las entidades involucradas en cada transacción (por ejemplo, consumidores, comerciantes, titulares de tarjetas, billeteras de pago, métodos alternativos de pago), para construir una reputación de pago y permitir que las empresas pueden reducir el potencial fraude y blanqueo de capitales.

Recibe información transaccional (pagos, transferencias, KYC, KYB) que comparan con su base de datos de identidad y la evalúa con la política de riesgo y cumplimiento que el cliente ha creado.

A continuación, se muestra una interacción típica de comerciante / IdentityMind para validar una transacción:

![](https://files.readme.io/bd2d2c0-v9qj1DP.png)

## EndPoints para validar datos del cliente
### Solicitud KYC del cliente

**URL**: `https://sandbox.identitymind.com/im/account/consumer`

**Method**: `POST`.

**Body params** (IDM ejecutara los test de seguridad correspondientes para validar la informacion):
- Parámetros requeridos para validar la **Geolocalización**:
   - **`Ip <string>(requerido)`**: dirección IP del usuario
      
      Utiliza al proveedor **MaxMind** para efectuar esta validación, mediante la ip puede detectar mediante la verificación de geolocalización países de alto riesgo y discrepancias entre los puntos específicos de la ubicación.

- Parametros requeridos para validar el **Correo Electrónico**:
   - **`tea <string> (requerido)**`: Email del usuario

- Parametros requeridos para validar el **Teléfono Celular**
   - **`pm <string>  (requerido)`**: Teléfono Celular del usuario

- Parametros requridos para validar la **Versión digital identificación oficial vigente**
   - **`scanData <Sting>  (requerido)`**: Imagen de la parte frontal de la identificación,  codificada en Base64 y con un tamaño máximo de 5MB
   
   - **`doctype <string>  (requerido)`**: Tipo del documento. Pasaporte (PP) | Licencia de conducir (DL) | Tarjeta de identidad emitida por el gobierno (ID) | Permiso de residencia (RP) | Factura de servicios públicos (UB)
   
   - **`docCountry <sring>  (requerido)`**: País en el que se emitió el documento
   
   - **`backsideImageData <string> (opcional)`**: Imagen de la parte de atrás del documento, codificada en Base64 y con un tamaño máximo de 5MB
   
   - **`bfn <string> (opcional)`**: Primer nombre de facturación
   
   - **`bln <string> (opcional)`**: Apellido de facturación
   
   - **`dob <string> (opcional)`**: fecha de nacimiento codificada como unix timestamp o un string en formato ISO8601
 
   Utiliza como proveedor de identificación de documentos a **Acuant** o **Mitek**

- Parámetros requeridos para la **validación biométrica de fotografía del rostro del cliente**

   -  **`faceImages <array of strings> (requerido)`**: Imagen de la cara del cliente, con la información de la imagen codificada en Base64 en un arreglo en formato JSON.
   
- Verificacion de **Blacklist**:
   
   - **`bfn <string> (requerido)`**: Primer nombre de facturación`
   
   - **`bln <string> (requerido)`**: Apellido de facturación`
   
   Este proceso se realiza mediante el servicio de sanction screening
   
- Verificación de **Cuenta Bancaria**:

   - **`patch <string> (requerido)`**: Identificador único de cuenta(hash)
   
   - **`ptoken <string> (requerido)`**: Una versión enmascarada o tokenizada del token de la cuenta.
   
   El proceso para obtener estos parámetros se especifican más adelante en este documento
   
- Validacion de **Clave Única de Registro de Población y clave de elector**
   
   - **`bfn <string> (requerido)`**: Primer nombre de facturación`
   
   - **`bln <string> (requerido)`**: Apellido de facturación
   
   - **`bmn <string> (requerido)`** : Segundo nombre
   
   - **`dob <string> (requerido)`** : Fecha de nacimiento
   
   - **`bsn <string> (requerido)`** : Dirección de residencia. Incluye numero de edificio, nombre de calle y numero de apartamento
   
   Para este proceso IDM utiliza el proveedor de Circulo de crédito, que es un servicio de verificación de identidad que permite realizar validaciones contra instituciones mexicanas

### Agregar un documento a una solicitud KYC

**URL**: `https://sandbox.identitymind.com/im/account/consumer/appId/files`

**Method**: `POST`

**Path Params**:
   
   - **`appId <string> (requerido)`**: Identificador de la solicitud KYC 
  
**Body Params**:
 
   - **`File <file> (requerido)`**: Documento a incluir en la solicitud
   
   - **`Description <string> (opcional)`**: Breve descripción   
   
## Proceso de validación de documentos
   La plataforma de indetityMind  utiliza una interfaz REST con datos codificados JSON. Al enviar los datos del formulario KYC mediante su Api, se analiza esta información tomando en cuenta datos de identidad, políticas de riesgo y cumplimientos legales que previamente se configuraron en la plataforma.  
Poseen distintos proveedores de servicios para la verificación de documentos, y utilizan un algoritmo para dirigir las peticiones a los mejores proveedores de pendiendo de la tarea a realizar.
Para explicar la forma en que la plataforma opera se tienen tres casos de usos:

### Identificación positiva
   Se asegura de que los documentos suministrados sean auténticos, válidos y que los datos que contienen coincidan con los datos suministrados por el cliente,  algunas de la pruebas de seguridad que se pueden realizar a través de la plataforma son las siguientes:

- `Security Test DV:3`. Autenticación de documento
- `Security Test DV:8`. Coincidencia de nombre
- `Security Test DV:9`. Fecha de Nacimiento
- `Security Test DV:10`. Coincidencia de dirección
- `Security Test DV: 11`. Documento expirado

La plataforma también ayuda a combatir la falsificación y robo de identidad; Esto se logra mediante el test de seguridad de  coincidencia de rostro `Security Test DV: 5`, el cual compara la foto del documento con una foto del cliente, la cual puede ser suministra por un archivo de imagen o como una “selfie”.

### Identificación de riesgo
   Una serie de fallos en la validación de un documento puede ser considerada un riesgo. El test de seguridad de Autenticación de documento puede fallar por varias razones; un caso común es que la imagen carece de suficiente resolución, a pesar de que el documento puede ser reconocido, tal vez los datos no se pueden leer, para este caso se utiliza el test de seguridad de documento decisivo `Security Test DV: 21`.
En caso de que no sea permitido la incorporación de usuarios de algunos países, se debe verificar que el  tipo de documento y la ubicación cumplan con las normativas del negocio, para lograr esto se tiene que tomar en cuenta lo siguiente:

   1- El país extraído del documento no coincide con el país proporcionado en la API (La API permite especificar cuales países son permitidos), para ello la plataforma pueden ejecutar los siguientes test de seguridad:
   
   - `Security Test DV:20`. Coincidencia de país en el documento
   - `Security Test DV:18`. País ID
   
   2- El tipo de documento no ofrece suficiente información para verificar la localidad del usuario (la API permite especificar el tipo de documento requerido), para este caso la plataforma puede ejecutar los siguientes tests de seguridad:
   
   - `Security Test DV:2`. Coincidencia de tipo de documento
   - `Security Test DV:19`. Tipo de documento
   
### Volver a suministrar imagen
   
   En caso de que un usuario suministre una imagen de mala resolución, se puede automatizar una respuesta para este escenario. Para este escenario la plataforma puede ejecutar el siguiente test de seguridad.
   
   - `Security Test DV:1`: Documento Procesado
   
   Si los tests de seguridad `DV3` y `DV21` fallan, no se puede declarar el documento como autentico o falsificación, para esta situación se tienen dos opciones; o la imagen se analiza manualmente o se le pide al usuario volver a suministrar la imagen.
   
##  Cosas que no puede validar IDM   
   Con respecto a la validación de la credencial para votar, la documentación de IDM no especifica si el proceso de verificación del código impreso en la credencial para votar y el Numero y año de emisión es posible mediante el uso de servicio de Circulo de crédito o con algún otro proveedor, por lo tanto es necesario realizar pruebas en este sentido o buscar alguna otra solución.   
   
## Obtener Hash y version tokenizada de la cuenta bancaria para su posterior validación

La API de IdentityMind no acepta identificadores textuales de cuentas bancarias sin cifrar. Acepta la siguiente información sobre la cuenta bancaria a utilizar:

- Hash de cuenta bancaria
- Token de cuenta bancaria

Para generar el hash de cuenta bancaria, se debe utilizar la **`SALT`**proporcionada por IdentityMind, para generar un hash **`SHA-1`** para el número de cuenta bancaria y convertir el arreglo de bytes del hash en una cadena hexadecimal. El hash debe incluirse en la cadena JSON de la solicitud.

Para este proceso se puede seguir el siguiente algoritmo:
~~~
import hashlib
import re

SALT = '\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'

def normalize_payment_identifier(identifier):
   return re.sub(r'[^a-zA-Z0-9]', '', identifier)

def generate_payment_hash(identifier):
   m = hashlib.sha1(SALT)
   m.update(normalize_payment_identifier(identifier))
   return m.hexdigest()
    
def generate_bank_account_hash(routing_num, account_num):
   """
   Generate a hash of the provided routing and account number using a well
   defined salt.  The value of the salt and a complete description of the
   hashing algorithm is available from IdentityMind support.
   """
   return generate_payment_hash(routing_num + account_num)

def generate_bank_account_token(routing_num, account_num):
   normalized = normalize_payment_identifier(routing_num + account_num)
   return normalized[0:6] + 'XXXXXXXX' + normalized[-4:]    
~~~

### Hash de cuenta bancaria

- Para un número de cuenta bancaria de EE. UU., concatene el **`SALT`**, el número de ruta y el número de cuenta y se pasa como argumento a la función para obtener el hash del número de cuenta.
- Para un número de cuenta IBAN internacional, concatene el **`SALT`** y el número de cuenta IBAN completo y pasa como argumento a la función para obtener el hash del número de cuenta.

>**Normalización**
>
> Todos los espacios y guiones deben ser removidos del numero de cuenta antes de realizar el hash

> **Valor de muestra**
>
>  Por ejemplo, el hash del número de cuenta bancaria para 321076479 74600015199010 previamente concatenado con el **`SALT`** es 3f57733f34b677294fed96efd440b8d9e7728fa5 y el hash para SN12K00100152000025690007542 previamente concatenado con el **`SALT`** es dd91898995dfef188eca122c5e0dd4550f3a. Se puede incluir esto en las pruebas unitarias para verificar que la implementación de hash sea correcta.

### Token de cuenta bancaria

Para el token de numero de cuenta bancaria IBM recomienda:

   - Para un número de cuenta bancaria de EE. UU., Los primeros 6 dígitos del número de ruta, seguidos de XXXXXXXX y los últimos 4 dígitos del número de cuenta
   - Para un número de cuenta IBAN internacional, los primeros 6 dígitos de la cuenta, seguidos de XXXXXXXX y los últimos 4 dígitos del número de cuenta
   
>**Token**
>
>El número enmascarado/tokenizado solo se usa con la interfaz de usuario como etiqueta para esta cuenta bancaria, y puede estar en cualquier formato que se ajuste a las limitaciones de seguridad de la información.

## Informacion de interes

### Acuant GO

Con Acuant Go podemos considerar una posible solución para la integración de la verificación KYC a nuestro sitio web. El plugin de Acuant GO es un formulario de solicitud en línea para el enrolamiento de clientes y para realizar la verificación debida de KYC.

El plugin está integrado con Acuant Compliance Platform, que tiene un ecosistema de más de 25 proveedores para validar la información del usuario. El complemento también está conectado a Acuant Digital Identity Network, que es una fuente poderosa para verificar si un usuario ha sido visto en la red y usar su reputación de eDNA para la evaluación de riesgos de identidad.

#### Ejemplo de flujo de usuario
   1- **Recolectar información**: El cliente proporciona la información necesaria para la verificación KYC. 
   
   <img src="https://downloads.intercomcdn.com/i/o/89065201/ec3f6e36eb0d57c4e650bafb/Screen+Shot+2018-12-04+at+11.12.47+AM.png" width="700" height="500" />
   
   2- Cada sección validará el formato de la información y el usuario podrá avanzar una vez que la información sea válida.
   
   <img src="https://downloads.intercomcdn.com/i/o/89065279/c0dd54d2bb096e382de73d2d/Screen+Shot+2018-12-04+at+11.12.59+AM.png" width="700" height="500" />
   
   3- Si se desea, se mostrará una sección de Verificación de ID, donde los usuarios deberán seleccionar el tipo de documento junto con el país emisor del documento:
   
   <img src="https://downloads.intercomcdn.com/i/o/89065472/f6d71685e508283da5a625e7/Screen+Shot+2018-12-04+at+11.17.24+AM.png" width="700" height="500" />
   
   4- Después de seleccionar el tipo de documento, el usuario podrá cargarlo a través del escritorio como un archivo o mediante el teléfono celular usando la cámara del               teléfono.
   
   <img src="https://downloads.intercomcdn.com/i/o/89065529/82396ae31888ad5139c1869d/Screen+Shot+2018-12-04+at+11.17.37+AM.png" width="700" height="500" />
   <img src="https://downloads.intercomcdn.com/i/o/89087123/277389ef8c41f8b5860dc040/Screen+Shot+2018-12-04+at+11.26.19+AM.png" width="700" height="500" />

Una vez que el usuario ha ingresado toda la información, se podría decidir mostrar un mensaje o redirigir a una nueva página según el resultado de KYC.

## Como acuant realiza la verificación de un documento

El proceso general consta de dos partes:

   1. Capturar el documento en la parte de frente y detrás, se extrae toda la información vía OCR de amazon web services y se realizan las pruebas forenses sobre el mismo (entre 10 y 50 pruebas), estas pruebas van permitir determinar si el documento es verdadero o falso. Acuant a creado una gran biblioteca de plantillas de documentos, que abarcan mas de 6000 documentos en aproximadamente 200 países, que permiten validar los candados de seguridad en el documento y determinar si el documento ha sido manipulado o si efectivamente es autentico sin necesidad de ninguna verificación manual 
   
   2. Se realiza una verificación facial mediante una selfie para determinar si el rostro de la persona coincide con la foto en el documento, y en paralelo a esto se ejecuta una prueba de vida para verificar que la persona no este tratando de burlar el sistema colocando una foto impresa, o un video sintético.
   
   Al finalizar este proceso se devuelve un puntaje del 0 al 100, que indica el porcentaje de similitud del rostro de la persona con la foto en el documento, y toda la información biométrica recabada es borrada de los servidores de acuant.
