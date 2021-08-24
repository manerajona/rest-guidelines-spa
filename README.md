# Introducción

Una API REST se centra en las entidades de negocio (datos) expuestos como recursos que se identifican a través de URIs y que pueden ser manipuladas a través de métodos estandarizados tipo CRUD utilizando diferentes representaciones. Las APIs RESTful tienden a ser menos específicas para cada caso de uso y son adecuadas para un ecosistema de micro-servicios. Un principio importante para el diseño y el uso de las API es la Ley de Postel, también conocida como el Principio de Robustez (véase también el [RFC 1122](https://datatracker.ietf.org/doc/html/rfc1122)):

> "Sé liberal en lo que aceptas, pero conservador en lo que envías

## ¿Qué es REST o Representational State Transfer?

- Es un estilo de arquitectura para la creación de aplicaciones en un esquema cliente-servidor, NO UNA TECNOLOGÍA.
- Se centra en la Transferencia de recursos a través de peticiones (Request) y respuestas (Response) síncronos.
- Cada Transferencia devuelve un estado númerico (1XX, 2XX, 3XX, 4XX, 5XX).
- En REST un objeto de negocio es considerado un Recurso.
- Un Recurso puede tener múltiples representaciones.
- Por lo general los Recursos son representados en un body en formato JSON o XML.
- Utiliza una comunicación STATELESS.
- Es un modelo adecuado para operaciones CRUD (Create, Read, Update, Delete).
- Los recursos pueden ser almacenados en caché.

## Terminología en REST

- **Recurso** = entidad de negocio.
- **Verbs** = métodos HTTP (GET, POST, PUT, PATCH, DELETE, etc).
- **Body** = payloads JSON/XML.
- **URI** = Identificador Uniforme de Recursos.
- **URL** = Localizador Uniforme de Recursos (información del host + URI).
- **Idempotent** = Propiedad de una operación de ser aplicada múltiples veces sin cambiar el resultado.
- **Stateless** = El servicio no mantiene ningún estado o sesión del cliente.
- **HATEOAS** = Acrónimo de "Hypermedia As The Engine Of Application State".

## VERBOS DE CONTRATO UNIFORME (HTTP/S)

- **GET**: obtiene el recurso (Sólo lectura, Idempotente)

- **HEAD**: como GET pero sólo obtiene los metadatos (Sólo lectura, Idempotente)

- **POST**: crea un nuevo recurso (No-Idempotente)

- **PUT**: crea o actualiza (si no existe) un recurso existente (Idempotente)

- **PATCH**: modifica parcialmente un recurso existente (No-Idempotente)

- **DELETE**: elimina un recurso (Idempotente)

- **TRACE**: se hará eco de la petición recibida. Se utiliza para ver si la solicitud fue alterada por servidores intermedios (proxis).

- **OPTIONS**: devuelve los métodos soportados para una url especificada (Sólo lectura, Idempotente)

**Métodos idempotentes**: GET, HEAD, PUT, TRACE, OPTIONS.
	
**Métodos NO-idempotentes**: POST (cuando se invoca la misma solicitud POST N veces, tendrá N nuevos recursos en el servidor) y PATCH (cuando se invoca la misma solicitud PATCH N veces, se actualizará N veces el recurso).

## Fields/Parameters

Anatomía de una url
```
https://afiliaciones.sancorsalud.com.ar:8080/v1/afiliados/1001/personas?nombre=Juan&apellido=Perez&edad=41

protocolo://sub-dominio.dominio:puerto/api-version/recurso/id-recurso/sub-recurso?filtro1=arg1&filtro2=arg2&filtro3=arg3
```

Los fields son los valores que pueden variar entre peticiones. Existen tres tipos de fields:

1. **PATH FIELDS**: Son parámetros que ayudan a identificar un único recurso. Por ejemplo: ```/afiliados/1001```, ```/afiliados/1002```, ```/afiliados/1003```, etc.

2. **QUERY FIELDS**: Son parámetros que sirven para filtrar una colección de recursos. A diferencia de un PATH PARAM no se asegura que se recupere un recurso único, si no que se debe esperar de 0 a N Recursos. Por ejemplo: ```/personas?nombre=Juan&apellido=Perez&edad=41```, ```/personas?nombre=John&apellido=Smith```, ```/personas?edad=10```, etc.

3. **BODY FIELDS**: Son parámetros en formato JSON/XML que son enviados en al servidor para crear o actualizar los datos de un recurso (se detalla en lineamientos para JSON).

## Rangos de Status Code (HTTP/S)

- **100-199**: información personalizada

- **200-299**: solicitud exitosa (200: OK, 201: CREATED, 204: NO CONTENT)

- **300-399**: redirecciones (301: MOVED PERMANENTLY)

- **400-499**: errores del cliente (400: BAD REQUEST, 401: UNAUTHORIZED, 403 FORBIDDEN, 404: NOT FOUND)

- **500-599**: errores del servidor (500: INTERNAL SERVER ERROR, 503: SERVICE UNAVAILABLE)

## Modelo de madurez de Richardson (RMM):
	
- **Nivel 0** = El intercambio de POX (Plain old XML). Ej: RPC, SOAP.
- **Nivel 1** = Recursos identificados por URIs.
- **Nivel 2** = Verbos HTTP como operaciones Curl.
- **Nivel 3** = Conocido como "La gloria de REST" o bajo el acrónimo HATEOAS, en dónde los recursos se relacionan por medio de links (Hypertext o URIs) incluidos en el payload.

![rmm.png](/rmm.png)

Algunas lecturas interesantes sobre el estilo de diseño de la API RESTful y la arquitectura de los servicios:

- REST API Design - Resource Modeling: 
https://www.thoughtworks.com/de-de/insights/blog/rest-api-design-resource-modeling

- Richardson Maturity Model — Steps toward the glory of REST: 
https://martinfowler.com/articles/richardsonMaturityModel.html

- Fielding Dissertation: Architectural Styles and the Design of Network-Based Software Architectures: 
https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm

# Lineamientos generales

## 1. DEBE mantener las URI lo más representativas posible

Seguir como regla general el sigiente pattern: ```{api-version}/{nombre-aplicación}/{nombre-del-recurso-en-plural}/{id-recurso}```

> Una URI debe representar inequívocamente a un recurso, NO UNA ACCIÓN.

Por eso, evitar colocar en POST -/-> "/crear", en PATCH -/-> "/actualizar", en delete -/->"/borrar" o en GET -/-> "/buscar". Ésto debe quedar implícito por la URI y el tipo de método que se utiliza.

NOTA: Se agrega el **nombre de la aplicación** para identificar en el bff si se está llamando a Salesforce o los sistemas legados de Fluir.

## 2. DEBE existir una correlación uniforme entre Método y Status code

Como regla general tener en cuenta que:

- **POST** se utiliza para crear un nuevo recurso, en caso de éxito devuelve status code **CREATED (201)** y sin body.

- **POST** se utiliza para correr un proceso asíncrono (batch, bulk), en caso de éxito devuelve status code **ACCEPTED (202)**.

- **PUT** se utiliza para actualizar o crear (si no existiese) un recurso, en caso de éxito devuelve status code **NO_CONTENT (204)** si se actualizó el recurso o **CREATED (201)** si se creó uno nuevo y en ambos casos sin body.

- **PATCH** se utiliza para actualizar parcialmente un recurso, en caso de éxito devuelve status code **NO_CONTENT (204)** y sin body.

- **DELETE** se utiliza para eliminar un recurso, en caso de éxito devuelve status code **NO_CONTENT (204)** y sin body.

- **GET (sin parametros)** se utiliza para obtener la colección de recursos, en caso de éxito devuelve status code **OK (200)** y la collection en el body.

- **GET (con parametros)** se utiliza para obtener un recurso en particular, en caso de éxito devuelve status code **OK (200)** y el recurso en el body.

## 3. DEBE utilizar minúsculas tanto en las URLs y en JSON, evitar camelCase.

Como regla general si existe una mayúscula en la url o en los campos del body (json) es un error. Los lineamientos de nomenclatura específicos se desarrollan más adelante.

## 4. DEBE proporcionar la especificación de la API utilizando OpenAPI 3

Utilizamos la especificación OpenAPI como estándar para definir nuestras APIs. Los diseñadores de la API deberían proporcionar la especificación de sus API utilizando un único documento autocontenido (como Swagger). Animamos a utilizar la versión 3.0 de OpenAPI: [OpenApi Specification](https://swagger.io/specification/).

Sugerencia: Una buena manera de explorar OpenAPI es navegar a través del [mapa mental de la especificación OpenAPI](https://openapi-map.apihandyman.io/) y el plugin Swagger para IntelliJ IDEA para crear su primera API: [Swagger Zalando](https://plugins.jetbrains.com/search?search=swagger%20Monte).

## 5. DEBERÍA proporcionar el manual de usuario de la API

Además de la especificación de la API, es una buena práctica proporcionar un manual de usuario de la API para mejorar la experiencia del desarrollador cliente, especialmente de los ingenieros que tienen menos experiencia en el uso de esta API.

# Lineamientos sobre JSON

## 1. DEBERÍA pluralizar los nombres de la colecciones

Los nombres de las colecciones deben ser pluralizados para indicar que contienen múltiples valores. Esto implica a su vez que los nombres de los objetos deben ser singulares.

Incorrecto:
```json
{
    "telefono_fijo": [
        {
            "tipo": "T",
            "caracteristica": "1234 ",
            "descripcion": "123545",
            "factura_por_email": "N",
            "codigo": 3
        }
    ],
    "telefono_celular": [
        {
            "tipo": "C",
            "caracteristica": "3493 ",
            "descripcion": "499353",
            "factura_por_email": "X",
            "codigo": 1
        }
    ],
    "email": [
        {
            "tipo": "E",
            "caracteristica": "",
            "descripcion": "yamilamartino8@gmail.com",
            "factura_por_email": "S",
            "codigo": 2
        },{
            "tipo": "E",
            "caracteristica": "",
            "descripcion": "yamilamartino8@hotmail.com",
            "factura_por_email": "N",
            "codigo": 4
        }
    ]
}
```

Correcto:
```json
{
    "telefonos_fijos": [
        {
            "tipo": "T",
            "caracteristica": "1234 ",
            "descripcion": "123545",
            "factura_por_email": "N",
            "codigo": 3
        }
    ],
    "telefonos_celulares": [
        {
            "tipo": "C",
            "caracteristica": "3493 ",
            "descripcion": "499353",
            "factura_por_email": "X",
            "codigo": 1
        }
    ],
    "emails": [
        {
            "tipo": "E",
            "caracteristica": "",
            "descripcion": "yamilamartino8@gmail.com",
            "factura_por_email": "S",
            "codigo": 2
        },{
            "tipo": "E",
            "caracteristica": "",
            "descripcion": "yamilamartino8@hotmail.com",
            "factura_por_email": "N",
            "codigo": 4
        }
    ]
}
```

## 2. Los nombres de propiedades deberían estar en snake_case y nunca camelCase

El primer carácter debe ser una letra o un guión bajo, y los caracteres siguientes pueden ser una letra, un guión bajo o un número. (Se recomienda utilizar _ al principio de los nombres de las propiedades sólo para palabras clave como _links, _errors, etc).

Incorrecto:
```json
{
    "recordTypeId": "0124x000000q1xjAAA",
    "tipoDeAsociado": "Linea individual",
    "condicionIVA": "Consumidor final",
    "convenio": "Mutual",
    "numeroDeDocumento": "123",
    "condicionDeContratacion": "Gravado",
    "fechaNacimiento": "1990-01-22",
    "cuil": "20337873101",
    "fechaDeAlta": "2020-07-24",
    "fechaDeEstado": "2020-07-24",
    "nombres": "Pablo",
    "apellido": "Morgado",
    "email": "test@gmail.com",
    "telefono": "349215367043",
    "tipoDeDocumento": 96,
    "telefonoFijo": "3492433502",
    "horarioDeContacto": "8 hs a 12 hs",
    "medioDeContactoPreferido": "Email",
    "estado": "ACTIVO"
}
```

Correcto:
```json
{
    "record_type_id": "0124x000000q1xjAAA",
    "tipo_de_asociado": "Linea individual",
    "condicion_iva": "Consumidor final",
    "convenio": "Mutual",
    "numero_de_documento": "123",
    "condicion_de_contratacion": "Gravado",
    "fecha_nacimiento": "1990-01-22",
    "cuil": "20337873101",
    "fecha_de_alta": "2020-07-24",
    "fecha_de_estado": "2020-07-24",
    "nombres": "Pablo",
    "apellido": "Morgado",
    "email": "test@gmail.com",
    "telefono": "349215367043",
    "tipo_de_documento": 96,
    "telefono_fijo": "3492433502",
    "horario_de_contacto": "8 hs a 12 hs",
    "medio_de_contacto_preferido": "Email",
    "estado": "ACTIVO"
}
```

Justificación: No existe un estándar establecido en la industria, pero muchas empresas populares prefieren snake_case: por ejemplo, GitHub, Stack Exchange, Twitter. Otras, como Google y Amazon, utilizan ambas, pero no sólo camelCase. Es esencial establecer un aspecto consistente de manera que el JSON parezca salido de la misma mano.

## 3. No usar null para las propiedades booleanas

Las propiedades JSON booleanas no deben presentarse como nulos. Un booleano es esencialmente una enumeración cerrada de dos valores, verdadero y falso. Si el contenido tiene un valor nulo significativo, es preferible sustituir el booleano por una enumeración de valores o estados con nombre - por ejemplo, accepted_terms_and_conditions con true o false puede sustituirse por accepted_terms_and_conditions con valores yes, no y unknown.
 
## 4. No debería usar null para arrays vacíos

Los valores vacíos de los arrays pueden representarse inequívocamente como la lista vacía, [].

## 5. DEBE utilizar la misma semántica para las propiedades nulas y ausentes

OpenAPI 3.x permite marcar las propiedades como requeridas y como nulables para especificar si las propiedades pueden estar ausentes ({}) o ser nulas ({"ejemplo":null}). Siga las siguentes reglas para ver cuando un valor puede ser nulo o ausente:

|requerida|nulable|ausente ```{}```|null ```{"example":null}```
|---|---|---|---|
|TRUE|TRUE|NO|SI|
|FALSE|TRUE|SI|SI|
|TRUE|FALSE|NO|NO|
|FALSE|FALSE|SI|NO|

Como regla general tenga en cuenta que **todas las variables nulas deben ser enviadas como ausentes, excepto que sean requeridas (estén marcadas como @NotNull)**.

## 6. DEBERÍA nombrar las propiedades de fecha/hora con el sufijo _at

Las propiedades de fecha y hora deberían terminar con _at para distinguirlas de las propiedades booleanas que, de otro modo, tendrían nombres muy similares o incluso idénticos:

- created_at en lugar de created,
- modified_at en lugar de modified
- occurred_at en lugar de occurred
- returned_at en lugar de returned

## 7. DEBERÍA declarar los valores de las enumeraciones utilizando UPPER_SNAKE_CASE

Las enumeraciones deberían representarse como definiciones de tipo String. Los valores de las enumeraciones deben utilizar sistemáticamente el formato de mayúsculas, por ejemplo para monedas USD, EUR o ARS y para idiomas ES_SPA, ES_ARG, EN_US, EN_GB. Este enfoque permite distinguir claramente los valores de las propiedades u otros elementos.

Excepción: Esta regla no se aplica a los valores que distinguen entre mayúsculas y minúsculas procedentes de fuera del ámbito de definición de la API, por ejemplo, los códigos de idioma de la norma ISO 639-1 (El ejemplo anterior es incorrecto según esta norma).


# Lineamientos sobre URLs

## 1. DEBE seguir una convención de nomenclatura para nombres de host

Los nombres de host en las APIs deberían ajustarse a la nomenclatura funcional dependiendo del público como sigue:

```
<functional-hostname> = <functional-name>.sancorsalud.com.ar
```
Incorrecto:
```
http://serviciosjbosstest.ams.red:8080/Afiliaciones/api
```
Correcto:
```
https://afiliaciones.sancorsalud.com.ar
```

## 2. DEBE utilizar nombres en minúsculas con guiones para URI/URL

Como regla general jamás debería haber mayùsculas en una url, y los strings deberían ser separados por un guión medio si fuera necesario.

Incorrecto:
```
/SalesForce/Products/{product-id}

/Abacom/ClientesPotenciales/{cliente-potencial-id}
```

Correcto:
```
/salesforce/products/{product-id}

/abacom/clientes-potenciales/{cliente-potencial-id}
```
Esto se aplica a los segmentos de la ruta únicamente y no a los nombres de los parámetros de cosulta (utilizan snake_case).

## 3. DEBE utilizar snake_case (nunca camelCase) para los parámetros de consulta

Incorrecto:
```
/asociados/usuario-web?nroDeCliente=1111&idCuenta=0001&apellido=sosa
```

Correcto:
```
/asociados/usuario-web?nro_de_cliente=1111&id_cuenta=0001&apellido=sosa
```

## 4. DEBE pluralizar los nombres de los recursos

Normalmente, se proporciona una colección de recursos cuando cuando a un método GET no se le pasa parámetros. El caso especial de un recurso singleton debe ser modelado como una colección con cardinalidad 1 y debería incluirse además la definición de max_items = min_items = 1 para hacer explícita la restricción de cardinalidad.

Incorrecto:
```
/salesforce/asset

/salesforce/account
```

Correcto:
```
/salesforce/assets

/salesforce/accounts
```

## 5. No debería usar /api como ruta base

En la mayoría de los casos, todos los recursos proporcionados por un servicio forman parte de la API pública, y por lo tanto deberían estar disponibles bajo la ruta base "/".

Si el servicio también debe soportar APIs internas no públicas - para funciones específicas de apoyo operativo, le animamos a mantener dos especificaciones de API diferentes (Para ambas API, no debe utilizar /api como ruta base).

Consideramos que la ruta base de la API forma parte de la configuración nombre de host funcional (punto 1). Por lo tanto, esta información debe ser declarada como parte del host.

Incorrecto:
```
/api/v1/salesforce/products
```

Correcto:
```
/v1/salesforce/products
```

## 6. DEBERÍA utilizar rutas normalizadas sin segmentos de ruta vacíos ni barras finales

No debe especificar rutas con barras duplicadas o finales, por ejemplo, /clientes//direcciones o /clientes/. En consecuencia, tampoco debe especificar o utilizar variables de ruta con valores de cadena vacíos.

Razonamiento: Las rutas no estándar no tienen una semántica clara. Como resultado, el comportamiento de las rutas no estándar varía entre los diferentes componentes de la infraestructura HTTP. Esto puede llevar a resultados ambiguos e inesperados durante el manejo y monitoreo de las solicitudes.

Recomendamos implementar servicios robustos contra clientes que no sigan esta regla. Todos los servicios deberían normalizar las rutas de las peticiones antes de procesarlas, eliminando las barras duplicadas y las finales. Por lo tanto, las siguientes peticiones deberían referirse al mismo recurso:
```
GET /asociados/ordenes/{order-id}
GET /asociados/ordenes/{order-id}/
GET /asociados/ordenes//{id-orden}
```
Nota: la normalización de rutas no es compatible con todos los Frameworks de desarrollo de forma inmediata. Los servicios deberían admitir al menos la ruta normalizada y rechazar todas las rutas alternativas con un estado de error descriptivo.

## 7. DEBE ceñirse a los parámetros de consulta convencionales

Si proporcionas soporte de consulta para buscar, ordenar, filtrar y paginar, debes seguir las siguientes convenciones de nomenclatura:

- **q**: parámetro de consulta por defecto.

- **sort**: lista de campos separada por comas para definir el orden de clasificación. Para indicar la dirección de la ordenación, los campos pueden llevar el prefijo + (ascendente) o - (descendente), por ejemplo, /ordenes?sort=+id.

- **fields**: expresión de nombre de campo para recuperar sólo un subconjunto de campos de un recurso, por ejemplo, /ordenes?fields=id,total,productos.

- **embed**: expresión de nombre de campo para expandir o incrustar subentidades, por ejemplo, dentro de una entidad orden, expandir el código de producto en el objeto productos. Implementar embed correctamente es difícil.

- **offset**: desplazamiento numérico del primer elemento proporcionado en una página que representa una solicitud de colección (para paginación), por ejemplo, /ordenes?offset=2.

- **limit**: límite sugerido por el cliente para restringir el número de entradas en una página (para paginación), por ejemplo, /ordenes?limit=2&offset=10

- **cursor**: un puntero a una página, que nunca debe ser inspeccionado o construido por los clientes. Suele codificar (encriptado) la posición de la página, es decir, el identificador del primer o último elemento de la página, la dirección de la paginación y los filtros de consulta aplicados para recrear la colección (para paginación).

# Lineamientos sobre Recursos

## 1. DEBE evitar las acciones y mantener URIs sin verbos

REST tiene que ver con los recursos, así que trata de modelar una API en torno a RECURSOS utilizando los métodos HTTP estándar como indicadores de operación. Por ejemplo, si una aplicación tiene que bloquear productos explícitamente para que sólo un usuario pueda editarlos, cree un bloqueo de productos con PUT o POST en lugar de utilizar una acción de bloqueo.

Incorrecto:
```
PUT /salesforce/productos/bloquear/{id-producto}

GET /salesforce/assets/buscarporproducto

GET /salesforce/accounts/gettiporegistro
```
Correcto:
```
PUT /salesforce/productos-bloqueados/{id-producto}

GET /salesforce/assets?id_producto={id-producto}

GET /salesforce/accounts-tipos-registro
```
La ventaja añadida es que ya dispone de un servicio para navegar, filtrar y hacer operaciones CRUD sobre los nuevos recursos.

Justificación: La API describe recursos, por lo que el único lugar donde deben aparecer acciones es en los métodos HTTP. En tus URIs, en lugar de pensar en acciones (verbos) usa sólo sustantivos.

## 2. Una URI siempre DEBE identificar a la colección de recursos y no un recurso en particular

Cuando definimos definimos nuestras URIs debemos pensarlas como una colección de recursos que:
1. Pueden ser filtrados por medio de parámetros de consulta, por ejemplo, ```/recursos?param0=arg0&param1=arg1&param2=arg2```.
2. Cuyos elementos pueden ser identificados individualmente por medio de un id único, por ejemplo, ```/recursos/1, /recursos/2, /recursos/3```.

## 3. DEBE identificar los recursos y sub-recursos a través de segmentos de ruta

Algunos recursos de la API pueden contener o hacer referencia a sub-recursos. Los sub-recursos son partes de un recurso de nivel superior. Los sub-recursos deben ser referenciados por su nombre e identificador en los segmentos de la ruta como sigue:
```
/recursos/{recurso-id}/subrecursos/{subrecurso-id}
```
Para mejorar la experiencia del cliente, debe procurar que las URL sean intuitivas, y que cada sub-ruta sea una referencia válida a un recurso o a un conjunto de recursos. Por ejemplo, si /partners/{partner-id}/addresses/{address-id} es válido, entonces, en principio, también /partners/{partner-id}/addresses, /partners/{partner-id} y /partners deben ser válidos. 

Ejemplo de rutas:
```
/shopping-carts/de:1681e6b88ec1/items/1
/shopping-carts/de:1681e6b88ec1

/contenido/imágenes/9cacb4d8
/contenido
```

Excepción: En algunas situaciones el identificador de recurso no se pasa como un segmento de ruta sino a través de la información de autorización en un Header, por ejemplo, un token de autorización o una cookie de sesión.

## 4. DEBE utilizar identificadores de recursos amigables

Para simplificar el encoding de los identificadores de recursos en las URL, su representación debe consistir únicamente en ASCII que utilicen letras, números, guión bajo, guión medio, dos puntos, punto y -en raras ocasiones- barra invertida (/).

Incorrecto:
```
PATCH /salesforce/accounts?dni={dni} # está bien para filtrar pero NO PARA IDENTIFICAR UN RECURSO
PATCH /salesforce/accounts/dni|{dni}
PATCH /salesforce/accounts/dni-->{dni}
PATCH /salesforce/accounts/dni👉{dni} # algún día en UTF-8 porque no...
```
Correcto:
```
PATCH /salesforce/accounts/dni:{dni}
PATCH /salesforce/accounts/dni.{dni}
PATCH /salesforce/accounts/dni_{dni}
PATCH /salesforce/accounts/dni-{dni}
PATCH /salesforce/accounts/dni/{dni} # está mal pero no tan mal!!
```
Nota: las barras invertida sólo se permiten para identificar recursos subyacentes.

## 5. DEBE utilizar parámetros de consulta para filtrar y NO para identificar recursos

Los parámetros de query son útiles para filtrar elementos en una colección, pero **no debemos utilizarlos para identificar recursos**.

Los parámetros de consulta quedar reservados para el método GET pero no para POST, PUT, PATCH, DELETE.

Incorrecto:
```
PATCH /salesforce/accounts?dni={dni} 
PATCH /salesforce/accounts?id={id}
```
Correcto:
```
GET /salesforce/accounts?dni={dni} 
GET /salesforce/accounts/{id}

PATCH /salesforce/accounts/dni-{dni}
PATCH /salesforce/accounts/{id}
```

## 6. PUEDE exponer claves compuestas como identificadores de recursos

Si un recurso se identifica mejor por una clave compuesta formada por otros múltiples identificadores de recursos, se permite reutilizar la clave compuesta en su forma natural utiliando barras invertidas como separador. Sin violar la regla anterior DEBE identificar los recursos y sub-recursos a través de segmentos de ruta como sigue:
```
/recursos/{clave compuesta-1}[delim-1]...[delim-n-1]{clave compuesta-n}
```
Ejemplo de rutas:
```
/shopping-carts/{país}/{id-sesión}
/shopping-carts/{país}/{id-sesión}/artículos/{id-artículo}

/repositories/{repository-name}-SNAPSHOT/artifacts/{artifact-name}:{tag}
/repositories/{repository-name}-RELEASE/artifacts/{artifact-name}:{tag}
```
Advertencia: La exposición de una clave compuesta como la descrita anteriormente limita la capacidad de evolucionar la estructura del identificador del recurso.

Para compensar este inconveniente, las APIs deben aplicar una abstracción de clave compuesta de forma consistente en todos los parámetros y atributos de las peticiones y respuestas, permitiendo a los consumidores tratarlos como una sustitución del identificador de recurso técnico. El uso de componentes de clave compuesta independientes debe limitarse a las solicitudes de búsqueda y creación, de la siguiente manera.

Componentes de clave compuesta pasados como parámetros en una query de búsqueda independientes (sólo para GET):
```
GET /productos?id=id1,id2,id3
```

## 7. PUEDE considerar el uso de URLs no-anidadas

Si un sub-recurso sólo es accesible a través de su recurso principal y no puede existir sin el recurso principal, considere el uso de una estructura de URL anidada, por ejemplo
```
/shoping-carts/ar/1681e6b88ec1/cart-items/1
```
Sin embargo, si se puede acceder al recurso directamente a través de su id único, la API debería exponerlo como un recurso de nivel superior. Por ejemplo, el cliente tiene una colección para los pedidos de venta; sin embargo, los pedidos de venta tienen un id único global y algunos servicios pueden optar por acceder a los pedidos directamente, por ejemplo
```
/clientes/1637asikzec1
/sales-orders/5273gh3k525a
```

## 8. Debería utilizar UUIDs si es necesario

La generación de IDs puede ser un problema de escala en casos de uso de alta frecuencia y casi en tiempo real. Los UUIDs resuelven este problema, ya que pueden ser generados sin colisiones de forma distribuida y no coordinada y sin viajes de ida y vuelta adicionales del servidor.

Sin embargo, también presentan algunas desventajas

- Clave puramente técnica sin significado: no están preparados para las convenciones de denominación o alcance de nombres que podrían ser útiles por razones pragmáticas, por ejemplo, aprendimos a utilizar nombres para los atributos de los productos, en lugar de UUID.

- Son bastante largos: una representación legible requiere 36 caracteres y conlleva un mayor consumo de memoria y ancho de banda.

- No se ordenan a lo largo de su historial de creación y no se indica el volumen de identificadores utilizados

Se desaconseja especialmente el uso de UUIDs como claves primarias de los datos maestros y de configuración, que tienen un bajo volumen de ids pero una amplia funcionalidad.

**Hay que tener en cuenta que los identificadores numéricos secuenciales y estrictamente crecientes pueden revelar información comercial crítica y confidencial, como el volumen de pedidos, a clientes no privilegiados.**

En cualquier caso, deberíamos utilizar siempre cadenas de caracteres en lugar de números para los identificadores. Esto nos da más flexibilidad para evolucionar el esquema de nomenclatura de los identificadores. En consecuencia, si se utilizan como identificadores, los UUID no deben calificarse utilizando una propiedad de formato.

Sugerencia: Normalmente, se utiliza un UUID aleatorio. Aunque la versión 1 de UUID también contiene marcas de tiempo principales, no se refleja en su ordenación lexicográfica. Este déficit es abordado por ULID (Universally Unique Lexicographically Sortable Identifier). Se puede favorecer el ULID en lugar del UUID, por ejemplo, para casos de uso de paginación ordenados según la hora de creación.

## 9. DEBERÍA limitar el número de tipos de recursos

Para que el mantenimiento y la evolución del servicio sean manejables, deberíamos seguir los principios de diseño de "segmentación funcional" y "separación de intereses" y no mezclar diferentes funcionalidades de negocio en la misma definición de API. 

En la práctica, esto significa que el número de tipos de recursos expuestos a través de una API debe ser limitado. En este contexto, un tipo de recurso se define como un conjunto de recursos altamente relacionados, como una colección, sus miembros y cualquier sub-recursos directos.

Por ejemplo, los recursos siguientes se contarían como tres tipos de recursos, uno para los clientes, otro para las direcciones y otro para las direcciones relacionadas con los clientes:
```
/clientes
/clientes/{id}
/clientes/{id}/preferencias

/clientes/{id}/direcciones
/clientes/{id}/direcciones/{addr}

/direcciones
/direcciones/{addr}
```
Tenga en cuenta que:

- Consideramos que /clientes/id/preferencias forma parte del tipo de recurso /clientes porque tiene una relación uno a uno con el cliente sin un identificador adicional.

- Consideramos /clientes y /clientes/id/direcciones como tipos de recursos separados porque /clientes/id/direcciones/{addr} también existe con un identificador adicional para la dirección.

- Consideramos que /direcciones y /clientes/{id}/direcciones son tipos de recursos separados porque no hay una forma fiable de estar seguros de que son el mismo.

Teniendo en cuenta esta definición, nuestra experiencia es que las API bien definidas no implican más de 4 a 8 tipos de recursos. Puede haber excepciones con dominios de negocio más complejos que requieran más recursos, pero debería comprobar primero si puede dividirlos en subdominios separados con APIs distintas.

## 10. DEBERÍA limitar el número de niveles de sub-recursos

Existen recursos principales (con rutas url raíz) y sub-recursos (o recursos anidados con rutas url no-raíz). Debe utilizar máximo 3 niveles de sub-recursos (anidados) ya que más niveles aumentan la complejidad de la API y la longitud de la ruta url. (Recuerde que algunos navegadores web populares no admiten URLs de más de 2000 caracteres).

# Lineamientos sobre Errores

## 1. DEBE especificar una descripción granular de los errores

Las APIs deben definir la visión funcional y de negocio y abstraerse de los aspectos de implementación. Las respuestas de éxito y error son una parte vital para definir cómo se utiliza correctamente una API.

Por lo tanto, deberías definir todas las respuestas de error específicas del servicio en tu especificación de la API, con el fin de proporcionar información importante para que los clientes puedan manejar situaciones estándar y excepcionales.

Sugerencia: A menos que un código de respuesta transmita una semántica clara del origen del error, utiliza una forma no-estándar que desarrolle una explicación adicional, se pueden combinar múltiples especificaciones de respuesta de error utilizando, por ejemplo, el siguiente patrón:
```json
{
    "_errors": [
        {
            "code": "INVALID_FIELD_VALUE",
            "description": "Dni en path y body no coinciden",
            "field": "dni",
            "value": "foo",
            "location": "PATH"
        }
    ],
    "_status": "BAD_REQUEST"
}
```
Los diseñadores de APIs también deberían pensar en exponer una tabla con los errores como parte de la documentación asociada a la API. Esta tabla proporciona información y orientación sobre el manejo de los errores específicos de la aplicación y está referenciado a través de enlaces desde la especificación de la API.

## 2. DEBE utilizar códigos de estado HTTP estándar

Sólo debes utilizar códigos de estado HTTP estandarizados de forma coherente con su semántica. No debe inventar nuevos códigos de estado HTTP.

Los estándares RFC definen ~60 códigos de estado HTTP diferentes con una semántica específica (principalmente [RFC7231](https://tools.ietf.org/html/rfc7231#section-6) y [RFC 6585](https://tools.ietf.org/html/rfc7231#section-6)) - y hay otros nuevos "códigos no oficiales", por ejemplo, los utilizados por servidores web populares como Nginx. VER: https://httpstatuses.com/.

A continuación, enumeramos los códigos de estado HTTP más utilizados y mejor comprendidos, de acuerdo con su semántica en las RFCs. Las APIs deberían utilizar sólo estos códigos para evitar los malentendidos que surgen de los códigos de estado HTTP menos utilizados.

### **Success codes**

|Code|Detalle|Methods|
|---|---|---|
|200|OK - esta es la respuesta estándar de éxito|Todos|
|201|CREATED - Se devuelve cuando se crea una entidad con éxito. Es libre de devolver una respuesta vacía o el recurso creado en un link. |POST, PUT|
|202|ACCEPTED - La solicitud se ha realizado con éxito y se procesará de forma asíncrona. |POST, PUT, PATCH, DELETE|
|204|NO CONTENT - No hay mensaje de respuesta. |PUT, PATCH, DELETE|
|207|MULTI-STATUS - La respuesta contiene múltiples informaciones de estado para las diferentes partes de una solicitud que implique procesamiento masivo (batch, bulk). |POST, PUT, PATCH, DELETE|

### **Redirection codes**

|Code|Detalle|Methods|
|---|---|---|
|301|Moved Permanently -  Esta y todas las futuras solicitudes deben dirigirse a la URI indicada.|Todos|
|303|See Other - La respuesta a la solicitud puede encontrarse en otro URI utilizando un método GET.|POST, PUT, PATCH, DELETE|
|304|Not Modified -  indica que una solicitud GET o HEAD condicional habría dado lugar a una respuesta 200 si no fuera porque la condición se evaluó como falsa, es decir, el recurso no se ha modificado desde la fecha o la versión pasada a través de las cabeceras de solicitud If-Modified-Since o If-None-Match.|GET, HEAD|

### **Errores de lado del cliente**

|Code|Detalle|Methods|
|---|---|---|
|400|Bad request - Debería entregarse en caso de que algún campo falle en la validación de la lógica de negocio.|Todos|
|401|Unauthorized - los usuarios deben logearse (esto suele significar "sin autentificar").|Todos|
|403|Forbbiden - el usuario no está autorizado a utilizar este recurso.|Todos|
|404|Not Found -  el recurso no se encontró o no existe.|Todos|
|405|Method Not Allowed - el método (GET, POST, PUT, etc) no se encuentra implementado para ese recurso.|Todos|
|406|Not Acceptable - contenido no aceptable según las cabeceras Accept enviadas en el request.|Todos|
|408|Request timeout - el servidor arrojó time out en la operación.|Todos|
|409|Conflict - no puede completarse la operación debido a un conflicto, por ejemplo, cuando dos clientes intentan crear el mismo recurso o si hay actualizaciones concurrentes y conflictivas.|POST, PUT, PATCH, DELETE|
|410|Gone - el recurso se ha eliminado y no es posible accederlo.|Todos|
|415|Unsupported Media Type - contenido no soportado según la cabecera content-type enviada en el request.|POST, PUT, PATCH, DELETE|
|423|Locked - otra operación está siendo procesada sobre ese recurso.|PUT, PATCH, DELETE|
|429|Too many requests - se ha sobrepasado el limite de request sobre un recurso (esto suele ocurrir en cyberataques).|Todos|

### **Errores del lado del servidor**

|Code|Detalle|Methods|
|---|---|---|
|500|Internal Server Error - una indicación genérica de error para un problema inesperado de ejecución del servidor. Este tipo de errores deberían ser aleatorios y recuperarse en caso de fallas, de caso contrario estamos en presencia de un Bug.|Todos|
|501|Not Implemented - el servidor no puede satisfacer la solicitud (normalmente implica una disponibilidad futura, por ejemplo, una nueva función).|Todos|
|503|Service Unavailable - el servicio no está disponible (temporalmente) (por ejemplo, si hay dependencia con un servicio secundario que no está disponible), puede ser conveniente que el cliente haga un reintento. Si es posible, el servicio debe indicar cuánto tiempo debe esperar el cliente estableciendo la cabecera Retry-After.|Todos|


## 3. DEBE utilizar los códigos de estado más específicos que 200 y 500

Debe utilizar el código de estado HTTP más específico cuando devuelva información sobre el estado de procesamiento de su solicitud o situaciones de error.

## 4. No debe exponer stacktraces 

Los stack traces contienen detalles de implementación que no son parte de una API, y en los que los clientes nunca deberían confiar. Además, los stack traces pueden filtrar información sensible que los socios y terceras partes no están autorizados a recibir y pueden revelar información sobre vulnerabilidades a los atacantes.

## 5. DEBE utilizar el código 207 para las solicitudes por lotes o masivas

Algunas APIs deben proporcionar solicitudes por lotes o masivas utilizando POST por razones de rendimiento, es decir, para la eficiencia de la comunicación y el procesamiento. En este caso, los servicios pueden necesitar señalar múltiples códigos de respuesta para cada parte de una solicitud por lotes o masiva. Dado que HTTP no proporciona una guía adecuada para manejar las solicitudes y respuestas por lotes o masivas, definimos el siguiente enfoque:

Una solicitud por lotes o masiva puede devolver códigos de estado 4xx/5xx, si el fallo no es específico de un elemento y no puede restringirse a elementos individuales de la solicitud por lotes o masiva, por ejemplo, en caso de situaciones de sobrecarga o fallos generales del servicio.

Las reglas están pensadas para que los clientes puedan actuar sobre las respuestas de lote y masivas de forma coherente inspeccionando los resultados individuales. Rechazamos explícitamente la opción de aplicar 200 sin inspeccionar el resultado, ya que queremos evitar riesgos y esperamos que los clientes manejen los fallos parciales del lote de todos modos.

## 6. DEBE utilizar el código 429 con las cabeceras para los límites de peticiones.

Las APIs que deseen gestionar la tasa de peticiones de los clientes deben utilizar el código de respuesta 429 (Too Many Requests), si el cliente ha superado la tasa de peticiones (ver [RFC 6585](https://datatracker.ietf.org/doc/html/rfc6585)). Dichas respuestas también deben contener información de cabecera que proporcione más detalles al cliente. Hay dos enfoques que un servicio puede adoptar para la información de cabecera:

- Devolver una cabecera ```Retry-After``` que indique cuánto tiempo debe esperar el cliente antes de realizar una nueva solicitud. La cabecera Retry-After puede contener un valor de fecha HTTP para reintentar después o el número de segundos para retrasar. Cualquiera de los dos es aceptable, pero las APIs deberían preferir utilizar un retraso en segundos.

- Devuelve un trío de cabeceras X-RateLimit. Estas cabeceras permiten a un servidor expresar un nivel de servicio en forma de un número de peticiones permitidas dentro de una ventana de tiempo determinada y cuando la ventana se restablece. Las cabeceras X-RateLimit son:
	
  1. ```X-RateLimit-Limit```: El número máximo de peticiones que el cliente puede realizar en esta ventana.
	
  1. ```X-RateLimit-Remaining```: El número de peticiones permitidas en la ventana actual.
	
  1. ```X-RateLimit-Reset```: El tiempo relativo en segundos en el que se restablecerá la ventana de límite de velocidad. Tenga en cuenta que esto es diferente al uso que hacen Github y Twitter de una cabecera con el mismo nombre que utiliza segundos de época UTC en su lugar.
