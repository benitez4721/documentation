
# IdentityMind
IdentityMind ofrece una Plataforma SaaS (software as a service), para la gestión de riesgo en linea y automatización de verificación de cumplimientos legales. Ayuda a empresas a reducir el fraude en la incoporacion de clientes y transacciones, y mejora la utilización de AML (Anti-money laudering), KYC (know your custumer) y listas de sancionados. Esta plataforma construye valida y evalua continuamente las identidades digitales atraves de la tecnología eDNA ™ para garantizar la seguridad comercial global, y cumplimentos legislativos del cliente desde su incoporacion y durante todo el ciclo de vida en la plataforma. Realizan un seguimiento seguro de las entidades involucradas en cada transacción( por ejemplo, consumidores, comerciantes, titulares de tarjetas, billeteras de pago, métodos alternativos de pago), para construir una reputación de pago y permitir que las empresas pueden reducir el potencial  fraude y blanqueo de capitales

## EndPoints para validar datos del cliente
### Evaluacion KYC del cliente
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
   - dob <string> (opcional)**:  de nacimiento codificada como unix timestamp o un string en formato ISO8601
 
   Utiliza como proveedor de identificación de documentos a **Acuant** o **Mitek**

