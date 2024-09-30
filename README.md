# API REST de Idempiere

Este documento proporciona una guía simplificada para usar la API REST de Idempiere, centrándose en tareas comunes como la autenticación y la creación de pedidos. Para obtener información más detallada, consulte el wiki oficial: https://wiki.idempiere.org/en/REST_Web_Services.

 Notas importantes:

- Permisos: El acceso a la API está limitado por los permisos del usuario.
- Usuario del sistema: No se puede acceder al usuario del sistema a través de la API.
- Demo: Utilice la demo oficial de Idempiere en https://demo.globalqss.com/webui/ para realizar pruebas.

Para validar que la informacion es correcta y sigues los pasos correctamente, por favor utiliza esta demo para tus pruebas iniciales.
### Guias:
#### Autenticacion
El sistema de autentacion de la API esta basado en JSON Web Token, por lo que es importante primero autenticar nuestro entorno de desarrollo para las siguientes peticiones, para hacer un inicio de sesion correctamente, es necesario:
* username
* password
* clientId
* roleId
* organizationId
* warehouseId (opcional)
* language (opcional, pero importante al momento de recibir traducciones o informacion)

El Endpoint sobre que usaremos para nuestro login es /`api/v1/auth/tokens`, veamos los dos posibles casos:

#### Login un solo paso
Si poseemos todos los campos mencionados anteriormente, podemos autenticar a los usarios con una sola peticion (POST), con el siguiente **body**:

```json
{
    "userName": "example",
    "password": "password",
    "parameters": {
        "clientId": 1,
        "roleId": 1,
        "organizationId": 1,
        "warehouseId": 1,
        "language": "es_CO"
    }
}
```

Un pequeno ejemplo, con datos reales de la demo que vimos previamente.
```Javascript
// Función para hacer una solicitud POST
async function sendData(url, data) {
  try {
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const result = await response.json();
    console.log(result);
  } catch (error) {
    console.error('Hubo un error:', error);
  }
}

// Uso de la función
const apiUrl = "https://demo.globalqss.com/api/v1/auth/tokens";

const data = {
    "userName": "GardenUser",
    "password": "GardenUser",
    "parameters": {
        "clientId": 1,
        "roleId": 1,
        "organizationId": 1,
        "warehouseId": 1,
        "language": "es_CO"
    }
};

sendData(apiUrl, data);
```

#### Login normal

En caso de querer iniciar sesion, eligiendo un rol, almacen o cliente, etc..., necesitas hacer las siguientes peticiones.

`POST .../api/v1/auth/tokens`

```json
{
    "userName": "username",
    "password": "password"
}
```

Response Payload

```json
// Response Payload
{
    "clients": [
        {
            "id": 11,
            "name": "GardenWorld"
        }
    ],
    "token": "eyJraWQiOiJpZGVtcGllcmUiLCJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJHYXJkZW5BZG1pbiIsImlzcyI6ImlkZW1waWVyZS5vcmciLCJDbGllbnRzIjoiMTEiLCJleHAiOjE2NjY5NjQ5Mjh9.3t0MuK6ReF7xNmb36ITM36VSKI5QnK3n0ZF_LIgPQSrso4oRhDsL8Mudc0NqH4qjvvKDlYsPquYKtrHnB5UiZg"
}
```

Como puedes ver, el response que obtendremos nos devolvera dentro del payload:
* Un **token temporal,** con el cual podemos consultar el resto de informacion
* Lista de clientes con id y nombre.

Con ese token puedes solicitar información de inicio de sesión del usuario que se está autenticando. Necesitas agregarlo al header del request con la clave valor como:
`Authorization: Bearer {authToken}`

La información que puedes solicitar es la siguiente, y debe hacerse en este orden porque cada solicitud necesita información de la llamada anterior:

GET .../api/v1/auth/roles?client={clientId}
Devuelve un array con los roles a los que tiene acceso el usuario

GET .../api/v1/auth/organizations?client={clientId}&role={roleId}
Devuelve un array con las organizaciones a las que tiene acceso el usuario.

/api/v1/auth/warehouses?client={clientId}&role={roleId}&organization={organizationId}
Devuelve un array con los almacenes a los que tiene acceso el usuario.

GET .../api/v1/auth/language?client={clientId}
Devuelve un array con los idiomas con los que el usuario puede iniciar sesión.

Cuando hayas obtenido toda la información necesaria, debes hacer una solicitud PUT final como esta:
PUT .../api/v1/auth/tokens

Body:

```
{
  "clientId": {clientId},
  "roleId": {roleId},
  "organizationId": {organizationId},
  "warehouseId": {warehouseId},
  "language": "{language}"
}
```

Response Payload
```
{
  "userId":101,
  "language":"en_US",
  "token":"eyJraWQiOiJpZGVtcGllcmUiLCJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJHYXJkZW5BZG1pbiIsIkFEX0NsaWVudF9JRCI6MTEsIkFEX1VzZXJfSUQiOjEwMSwiQURfUm9sZV9JRCI6MjAwMDAxLCJBRF9PcmdfSUQiOjExLCJNX1dhcmVob3VzZV9JRCI6MTAzLCJBRF9MYW5ndWFnZSI6ImVuX1VTIiwiQURfU2Vzc2lvbl9JRCI6MTAwMDE2MiwiaXNzIjoiaWRlbXBpZXJlLm9yZyIsImV4cCI6MTcwMTc3NTUzN30.bAUEhPylAQhZjZquJhvLpO9zMZG3g6zlM_IqO9ifeXJpAJBOoJtDqd8CrYPU1PKURzoPRSUglbKqr1LsXdz38A",
  "refresh_token":"eyJraWQiOiJpZGVtcGllcmUiLCJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIwODk1OTIyNC1iYzBhLTRkNTQtOTlhZS1jNmRmZjNiOGEwMzUiLCJpc3MiOiJpZGVtcGllcmUub3JnIiwiZXhwIjoxNzAxODU4MzM3fQ.uSE5SOtWgPvReC4JtyV4alHd-ccU0L9QhIpP2TwT7C5TJFeCGVYTdyWc291DaIweyiIGCfWFgQlbe0oH1EEXXg"
}
```
#### Refresh Token
Como vez en el payload anterior, el **token** base expira cada hora, el **refresh_token **cada 24 horas por defecto, pero esto puede variar segun la configuracion del sistema

Cuando el Token expire, puedes generarlo de nuevo:

`POST .../api/v1/auth/refresh`
Body:
`{`**`"refresh_token"`****`:"{{refreshToken}}"}`**

El "token'" sera el utilizado dentro de la cabecera.
Tras haber iniciado sesion, el token obtenido lo debes agregar en el header de todas las peticiones:
`Authorization: Bearer {authToken}`
#### Cliente
El modelo de CB_Partner (Terceros dentro de idempiere, es con el cual consultaremos a la API para obtener toda la informacion de los clientes existentes:

Puedes consultar al siguiente endpoint:

GET -> `/api/v1/models/c_bpartner`

Esto devolvera un payload con la siguiente estructura:

```json
{
    "page-count": 1,
    "records-size": 100,
    "skip-records": 0,
    "row-count": 26,
    "array-count": 0,
    "records": [
      ...
    ]
}
```

Como puedes ver, podemos obtener la cantidad de registros, cuantos fueron contados, y en que pagina nos encontramos.

Dentro de la documentacion oficial, puedes ver ejemplos de como filtrar, cambiar de pagina, y obtener los registros de la manera que mejor se ajuste a tus necesidades.

Para crear un tercero, solo con agregar el cambo Name ya podremos crear un C_BPartner, pero para idemtificarlo de una forma mas sencilla, es ideal incluir lo siguiente:

```
{
    "Name": "Antonio",
    "Name2": "Banderas",
    "Description": "Optional Value",
    "SO_Description": "Informacion adicional a mostrar en ordenes de cliente",
    "TaxID": "Numero de identificacion",
    "C_BPartner_Location": [
        {
            "Name": "Nombre de direccion",
            "Phone": "011222333",
            "C_Location_ID": {
                "Address1": "Calle",
                "Address2": "Calle 2"
            }
        }
    ]
}
```

#### Orden de venta
Para crear una orden de venta, necesitamos saber que el nombre de esta tabla o modelo es: `c_order`
Por lo que para crear o consultar un registro debemos consultar al endpoint : /api/v1/models/c_order

Un ejemplo de como crear una orden:

```
{
    
      "isSOTrx": true,
      "POReference": "Referencia",
      "Description": "Descripcion",
      "C_Currency_ID": 100,
      "M_PriceList_ID": 101,
      "DateOrdered": "2024-09-30",
      "DatePromised": "2024-09-30",
      "C_BPartner_ID": 1000016,
    //   "C_BPartner_Location_ID": partner.partnerLocationId,
      "deliveryViaRule": "P",
      "C_OrderLine": [
        {
            "M_Product_ID": 134,
            "QtyOrdered": 10,
            "QtyEntered": 10
        }
      ]
}
```

Esta seria un body correcto, para crear la cabecera de nuestas ordenes de venta, expliquemos campo por campo:

* isSoTrx: Indica que es una orden de venta
* PoRefernce: Agregamos la referencia de donde se origina la orden, algun codigo, o secuencia que identifique dicho proceso (opcional)
* Description: Descripcion detallada de la orden
* PriceList Id, Opcional.
* C Currency Id: Indicamos el id de la moneda (Opcional)
* DateOrdered: Fecha de orden en formato: yyyy-MM-dd
* DatePromised: Fecha de entrega en formato: yyyy-MM-dd
* C BPartner Location Id: indicamos el id de la direccion del cliente en caso de que posea mas de una
* C OrderLine, Indicamos las lineas de la orden, ya explicaremos en detalle como generarlas.

Esto nos debe retornar un Payload parecido al siguiente:

##### Payload Order:

```
{
    "id": 1000078,
    "uid": "43785296-17f4-45c6-8b4d-df354fca71a0",
    "AD_Client_ID": {
        "propertyLabel": "Tenant",
        "id": 11,
        "identifier": "GardenWorld",
        "model-name": "ad_client"
    },
    "AD_Org_ID": {
        "propertyLabel": "Organization",
        "id": 11,
        "identifier": "HQ",
        "model-name": "ad_org"
    },
    "IsActive": true,
    "Created": "2024-09-30T13:39:39Z",
    "CreatedBy": {
        "propertyLabel": "Created By",
        "id": 101,
        "identifier": "GardenAdmin",
        "model-name": "ad_user"
    },
    "Updated": "2024-09-30T13:39:39Z",
    "UpdatedBy": {
        "propertyLabel": "Updated By",
        "id": 101,
        "identifier": "GardenAdmin",
        "model-name": "ad_user"
    },
    "DocumentNo": "50015",
    "DocStatus": {
        "propertyLabel": "Document Status",
        "id": "DR",
        "identifier": "Drafted",
        "model-name": "ad_ref_list"
    },
    "C_DocType_ID": {
        "propertyLabel": "Document Type",
        "id": 0,
        "identifier": "** New **",
        "model-name": "c_doctype"
    },
    "C_DocTypeTarget_ID": {
        "propertyLabel": "Target Document Type",
        "id": 132,
        "identifier": "Standard Order",
        "model-name": "c_doctype"
    },
    "Description": "Descripcion",
    "IsApproved": false,
    "IsCreditApproved": false,
    "IsDelivered": false,
    "IsInvoiced": false,
    "IsPrinted": false,
    "IsTransferred": false,
    "DateOrdered": "0036-03-16",
    "DatePromised": "0036-03-16",
    "DateAcct": "2024-09-30",
    "C_PaymentTerm_ID": {
        "propertyLabel": "Payment Term",
        "id": 105,
        "identifier": "Immediate",
        "model-name": "c_paymentterm"
    },
    "C_Currency_ID": {
        "propertyLabel": "Currency",
        "id": 100,
        "identifier": "USD",
        "model-name": "c_currency"
    },
    "InvoiceRule": {
        "propertyLabel": "Invoice Rule",
        "id": "I",
        "identifier": "Immediate",
        "model-name": "ad_ref_list"
    },
    "FreightAmt": 0.0,
    "DeliveryViaRule": {
        "propertyLabel": "Delivery Via",
        "id": "P",
        "identifier": "Pickup",
        "model-name": "ad_ref_list"
    },
    "PriorityRule": {
        "propertyLabel": "Priority",
        "id": "5",
        "identifier": "Medium",
        "model-name": "ad_ref_list"
    },
    "TotalLines": 0.0,
    "GrandTotal": 0.0,
    "M_Warehouse_ID": {
        "propertyLabel": "Warehouse",
        "id": 103,
        "identifier": "HQ Warehouse",
        "model-name": "m_warehouse"
    },
    "M_PriceList_ID": {
        "propertyLabel": "Price List",
        "id": 101,
        "identifier": "Standard",
        "model-name": "m_pricelist"
    },
    "C_BPartner_ID": {
        "propertyLabel": "Business Partner",
        "id": 1000016,
        "identifier": "Antonio",
        "model-name": "c_bpartner"
    },
    "POReference": "Referencia",
    "ChargeAmt": 0.0,
    "Processed": false,
    "C_BPartner_Location_ID": {
        "propertyLabel": "Partner Location",
        "id": 1000005,
        "identifier": "Nombre de direccion",
        "model-name": "c_bpartner_location"
    },
    "IsSOTrx": true,
    "DeliveryRule": {
        "propertyLabel": "Delivery Rule",
        "id": "F",
        "identifier": "Force",
        "model-name": "ad_ref_list"
    },
    "FreightCostRule": {
        "propertyLabel": "Freight Cost Rule",
        "id": "I",
        "identifier": "Freight included",
        "model-name": "ad_ref_list"
    },
    "PaymentRule": {
        "propertyLabel": "Payment Rule",
        "id": "B",
        "identifier": "Cash",
        "model-name": "ad_ref_list"
    },
    "IsDiscountPrinted": false,
    "IsTaxIncluded": false,
    "IsSelected": false,
    "SendEMail": false,
    "Bill_BPartner_ID": {
        "propertyLabel": "Invoice Partner",
        "id": 1000016,
        "identifier": "Antonio",
        "model-name": "c_bpartner"
    },
    "Bill_Location_ID": {
        "propertyLabel": "Invoice Location",
        "id": 1000005,
        "identifier": "Nombre de direccion",
        "model-name": "c_bpartner_location"
    },
    "IsSelfService": false,
    "IsDropShip": false,
    "IsPayScheduleValid": false,
    "IsPriviledgedRate": false,
    "model-name": "c_order"
}
```

### Lineas de la orden:
```Python
# Un pequeno ejemplo de como crear una linea dentro de la orden de venta

"C_OrderLine": [
  "M_Product_ID": 134,
  "QtyOrdered": 10,
  "QtyEntered": 10
]
```

En las lineas, simplemente debemos indicar:

* M Product ID, identificando al producto
* Qty Entered, cantidad ingresada
* Qty Ordered cantidad ordenada por cliente.
### Crear cliente desde la orden:
En caso de que el cliente no exista y queramos crear uno nuevo, podemos hacer una sola peticion generando al cliente de nuestra orden.
> Advertencia! Es importante evitar la duplicidad de la informacion, por lo cual es importante validar que realmente el tercero/cliente no exista.

Usando la llave C_BPartner_ID, haremos lo siguiente:


POST -> c_order body:

```
{
    
      "isSOTrx": true,
      "POReference": "Referencia",
      "Description": "Descripcion",
      "C_Currency_ID": 100,
      "M_PriceList_ID": 101,
      "DateOrdered": "2024-09-30",
      "DatePromised": "2024-09-30",
      "C_BPartner_ID": {
          "Name": "Antonio",
          "Name2": "Banderas",
          "Description": "Optional Value",
          "SO_Description": "Informacion adicional a mostrar en ordenes de cliente",
          "TaxID": "Numero de identificacion",
          "C_BPartner_Location": [
              {
                  "Name": "Nombre de direccion",
                  "Phone": "011222333",
                  "C_Location_ID": {
                      "Address1": "Calle",
                      "Address2": "Calle 2"
                  }
              }
          ]
      },
    //   "C_BPartner_Location_ID": partner.partnerLocationId,
      "deliveryViaRule": "P",
      "C_OrderLine": [
          {
              "M_Product_ID": 134,
              "QtyOrdered": 10,
              "QtyEntered": 10
          }
        ]
      }
```
En el ejemplo de body anterior, pueder ver como dentro de la llave **C_BPartner_ID**, creamos un diccionario con los valores del cliente de nuestra orden.
