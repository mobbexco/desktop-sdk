# Windows POS SDK
En esta documentación se definen los parámetros y elementos necesarios para utilizar el SDK de Mobbex en la plataforma Windows, permitiendo su integración con otras aplicaciones.

#### Primeros Pasos
Antes de comenzar, debe asegurarse de tener instalada la plataforma .NET Framework versión 4.5.2
Todos los archivos descargados desde nuestro repositorio se deben copiar juntos en la ubicación que usted elija dentro de su aplicación. El archivo principal y con el cuál deberá interactuar vía consola es Mobbex.exe

#### Configuración de parámetros  
Para poder utilizar este SDK es necesario que usted posea las Private Key y Public Key asociadas a su comercio. Esto puede generarlo desde la sección "Mis Apps" dentro de su cuenta de usuario.
Estas credenciales deberán configurarse como una variable de entorno al ejecutar el SDK, dependiendo del tipo de clave necesaria en cada operación.
El nombre de la variable de entorno utilizada es ```APIKEY```

Por otro lado, es necesario configurar las opciones de salida del SDK, utilizadas para almacenar y/o imprimir los cupones correspondientes a las transacciones, una vez que son completadas (independientemente de su resultado). Al menos una de las dos opciones de salida debe estar configuradas, caso contrario se generará una alerta al iniciar la aplicación.
Estos parámetros se definen en el mismo archivo de configuraciones, dentro del nodo "output".
- ```folder```: indica la carpeta donde se almacenarán los cupones en formato PDF al finalizar la transacción
- ```printer```: este nodo contiene las propiedades de la impresora utilizada para imprimir los cupones al finalizar la transacción. La propiedad **name** hace referencia al nombre de la impresora tal como está definida en Windows. La propiedad **type** indica el tipo de impresora (valores admitidos: "normal" y "ticket").
```json
"output":{
	"folder": "..",
  "printer": {
  	"name": "HP Laserjet 1102W",
    "type": "normal"
  }
}
```
La definición de la carpeta y/o impresora de cupones también puede realizarse enviando estos parámetros como opciones al ejecutar Mobbex.exe, teniendo en cuenta la sintáxis de opciones por consola:
```bash
Mobbex.exe -f ".." -p "HP Laserjet 1102W:normal"
```
Donde el caracter **:** es el separador del nombre y tipo de la impresora.
Si se opta por esta opción, estas opciones deberán ser enviadas en toda llamada al SDK.

## Operaciones
### 1. Crear intención de Cobro
>```APIKEY```: Llave Privada  

El primer paso para realizar una operación es crear la intención de cobro, obteniendo el *Intent Token* necesario para las operaciones siguientes.
Para esto se debe ejecutar la opción ***-cit*** del programa, pasando como parámetros de la opción todos los atributos de la operación en formato JSON. Los parámetros son los mismos que los definidos para un Checkout tradicional (definidos en https://mobbex.dev/docs/checkout).
Para realizar generar una operación en modo test o modo de pruebas, simplemente se debe agregar al programa la opción ***-t***
Ejemplo:
```bash
Mobbex.exe -t -cit "{\"total\": 100, \"currency\": \"ARS\", \"description\": \"Venta de Prueba\", \"reference\": \"Referencia001\", \"customer\": {\"identification\": \"99999999\", \"email\": \"demo@mobbex.com\", \"name\": \"Demo User\"}}"
```
En este caso el nodo "customer" es **obligatorio**:
- ```customer```: Objeto de tipo JSON con los datos del cliente.
		
    - ```uid```:  Identificador único e interno del cliente. Obligatorio.
    - ```name```: Nombre del Cliente. Obligatorio.
    - ```identification```: DNI. Obligatorio.
    - ```phone```: Número de Celular.

La respuesta del SDK será similar a la siguiente:

```json
{
 "operation": "cit", 
 "type": "result", 
 "data": {
   "result":true,
   "data":{
     "id":"ZNLLBHDO0DK1E1FC3G",
     "url":"https://mobbex.com/p/checkout/v2/ZNLLBHDO0DK1E1FC3G",
     "description":"Venta de Prueba",
     "currency":"ARS",
     "total":100,
     "created":1606264299516,
     "customer":{
       "name":"Demo User"
     },
     "intent":{
       "token":"a143c8e2-46ae-4e61-8ddb-ec6f10f6d57b"
     }
   }
 }
}
```
Debemos almacenar el parámetro ***token*** para utilizarlo en las próximas operaciones.

### 2. Crear Token de Tarjeta
>```APIKEY```: Llave Pública  

En este paso crearemos el token perteneciente a la tarjeta con la cuál se realizará la operación.
Se realiza con la opción ***-ct***, pasando como parámetro el intent token generado en el paso anterior.
```bash
Mobbex.exe -ct "{\"it\": \"a143c8e2-46ae-4e61-8ddb-ec6f10f6d57b\"}"
```
Existen 2 formas de cargar los datos de la tarjeta para generar el token:
- **Ingreso manual**: con la opción ***-m*** luego de la opción -ct. Esta opción abre una ventana para que el usuario pueda cargar los datos de la tarjeta y, además, seleccionar el plan de cuotas de la operación. Ejemplo:
```bash
Mobbex.exe -ct "{\"it\": \"a143c8e2-46ae-4e61-8ddb-ec6f10f6d57b\"}" -m
```
- **Ingreso a través de lector de tarjetas Mox**: con la opción ***-r*** luego de la opción -ct. Se debe indicar como parámetro de esta opción el tipo de lector a utilizar para realizar la operación, siendo los valores actualmente soportados:
  - ```BBPOS_CHBT```: Permite utilizar el lector BBPOS modelo CHB10
  - ```INGENICO_5500```: Permite utilizar el lector Ingenico modelo Moby 5500
  - ```INGENICO_6500```: Permite utilizar el lector Ingenico modelo Moby 6500
```bash
Mobbex.exe -ct "{\"it\": \"a143c8e2-46ae-4e61-8ddb-ec6f10f6d57b\"}" -r "BBPOS_CHBT"
```

El resultado de ejecutar el comando tendrá una salida como la siguiente:
```json
{
  "operation": "ct",
  "type": "result",
  "data": {
    "result":true,
    "data":{
      "token":"T:abgtu~Olh",
      "description":"Visa Débito terminada en 0010",
      "source":{
        "references":["visa.debit"],
        "reference":"visa.debit",
        "generic":"visa.debit",
        "compatReference":"visa.debit",
        "name":"Visa Débito",
        "shortName":"Visa Débito",
        "currency":"ARS",
        "card":{
          "level":"classic",
          "product":{
            "name":"Visa Débito",
            "shortName":"Visa Débit",
            "variant":"debit",
            "lengths":[16],
            "gaps":[4,8,12],
            "code":{
              "name":"CVV",
              "length":3,
              "position":1
            },           "logo":"https://res.mobbex.com/images/sources/png/visa.png",
            "validation":["length","exp","cvv","luhn"]
          },
          "issuer":{
            "shortName":"Visa",
            "name":"Visa",
            "color":"#122d98",
            "logo":"https://res.mobbex.com/images/sources/png/visa.png"
          }
        },
        "type":"card",
        "priority":0
      }
    }
  }, 
  "options":{
    "bin":"450799",
    "installment":{
      "reference":"1"
    }
  }
}
```

En este paso debemos almacenar los siguientes valores:
- ```data.data.token```: Es el token que hace referencia a la tarjeta almacenada.
- ```options.bin```: Será utilizado en el próximo paso para identificar el tipo de tarjeta.
- ```options.installment.reference```: Hace referencia al plan de cuotas seleccionado. Aplica solo a carga manual de tarjeta.

### 3. Obtener Medios de Pago y Cuotas
>```APIKEY```: Llave Pública  

En el caso de que la operación de tokenización haya sido realizada con un lector, o simplemente requiera obtener los medios de pago (planes de cuotas) asociados a una tarjeta, debe utilizar esta opción del SDK.
Debe llamarse al archivo con la opción ***-gi*** pasando como parámetro de la opción los siguientes datos:
 - ```it```: Intent token de la operación. Este parámetro es Obligatorio.
 - ```bin```: Primeros 6 dígitos de la tarjeta sobre los cuales se quieren obtener los planes de cuotas. Es un valor retornado también de la operación anterior. Este parámetro es Obligatorio.
 - ```installments```: esta opción indica si calcular los valores de cuotas de las formas de pago. Valores true/false. Este parámetro es opcional y por defecto es true.

Ejemplo de utilización de esta opción:
```bash
Mobbex.exe -gi "{\"it\": \"a143c8e2-46ae-4e61-8ddb-ec6f10f6d57b\", \"bin\": \"450799\", \"installments\": true}"
```

El resultado de esta operación es la siguiente:
```json
{
  "operation": "gi", 
  "type": "result", 
  "data": {
    "result":true,
    "data":{
      "type":"card",
      "source":{
        "references":["visa.debit"],
        "reference":"visa.debit",
        "generic":"visa.debit",
        "compatReference":"visa.debit",
        "name":"Visa Débito",
        "shortName":"Visa Débito",
        "currency":"ARS",
        "card":{
          "level":"classic",
          "product":{
            "name":"Visa Débito",
            "shortName":"Visa Débit",
            "variant":"debit",
            "lengths":[16],
            "gaps":[4,8,12],
            "code":{
              "name":"CVV",
              "length":3,
              "position":1
            },
           "logo":"https://res.mobbex.com/images/sources/png/visa.png",
           "validation":["length","exp","cvv","luhn"]
          },
          "issuer":{
            "shortName":"Visa",
            "name":"Visa",
            "color":"#122d98",
            "logo":"https://res.mobbex.com/images/sources/png/visa.png"
          }
        },
        "type":"card",
        "priority":0
      },
      "installments":[{
        "name":"Débito",
        "description":"10% de descuento",
        "sourceReference":"",
        "count":1,
        "reference":"1",
        "percentage":-10,
        "entity_percentage":0,
        "discount":0,
        "promo_code":"",
        "modes":["CNP","CP"],
        "status":"active",
        "order":1,
        "updated":"2020-10-27T13:39:59.829Z",
        "created":"2020-03-25T22:37:02.155Z",
        "_id":"5e7bdd0e2b6c8c490948ad8d",
        "uid":"4EZZXHEe4",
        "type":"direct",
        "updatedBy":"5d682afc8d0b0318fd9691d7",
        "totals":{
          "currency":{
            "value":"TEST",
            "label":"Test Money",
            "symbol":"T$",
            "hidden":false
          },
          "installment":{
            "amount":90,
            "count":1
          },
          "total":90,
          "financial":{
            "percentage":-10,
            "amount":-10
          }
        }
      }]
    }
  }
}
```

El parámetro a almacenar en este caso es ```reference``` del plan de cuotas elegido.

### 4. Confirmar Operación
>```APIKEY```: Llave Pública  

Para confirmar la operación y ejecutar finalmente el cobro, unicamente debe llamar al SDK con la opción ***-co*** y los siguientes parámetros en formato JSON:
 - ```it```: Intent token de la operación.
 - ```source```: Nodo que contiene los datos referidos a la tarjeta a procesar.
 - ```source.token```: Token de la tarjeta.
 - ```source.bin```: Primeros 6 dígitos de la tarjeta (BIN) obtenido en la etapa de tokenización de la tarjeta.
 - ```installment```: referencia del plan de cuotas o forma de pago de la tarjeta.

Ejemplo:
```bash
Mobbex.exe -co "{\"it\": \"a143c8e2-46ae-4e61-8ddb-ec6f10f6d57b\", \"source\":{\"token\":\"T:abgtu~Olh\", \"bin\": \"450799\"}, \"installment\": \"1\"}"
```
Al ejecutar el comando se abrirá una ventana donde deberá ingresar el código de seguridad (CVV) de la tarjeta, para poder confirmar finalmente la operación.

Luego de ingresar el CVV y confirmar, el resultado del comando es similar al siguiente:
```json
{"operation": "co",
 "type": "result",
 "data": {
   "result":true,
   "data":{
     "view":"result",
     "options":{},
     "id":"Erne0MBjK",
     "status":{
       "code":"400",
       "text":"Rechazado",
       "message":"La tarjeta ingresada es inválida. Verifique la tarjeta ingresada. (Cod. 103)"
     },
     "total":90,
     "currency":{
       "value":"TEST",
       "label":"Test Money",
       "symbol":"T$",
       "hidden":false},
     "data":[
       {
         "type":"item",
        "key":"transactionId",
         "label":"ID de Transacción",
         "value":"Erne0MBjK",
         "priority":2,
         "visibility":"visible"
       },
       {
         "type":"item",
         "key":"cardNumber",
         "label":"Tarjeta",
         "value":"450799******0010",
         "priority":99,
         "visibility":"hidden"
       }
     ],
     "actions":[
       {
         "label":"Imprimir",
         "key":"print",
         "type":"default",
         "action":{
           "type":"print",
           "data":{
             "url":"https://ms.mobbex.com/prod/reports",
             "path":"/v2/coupon/",
             "id":"Erne0MBjK",
             "schema":"card"}
         }
       },
       {
         "label":"Reintentar",
         "key":"retry",
         "type":"primary",
         "action":{
           "type":"reload"}
       }]
   }
 }
}
```
Además de la salida por consola, el sistema almacenará el cupón de la operación en formato PDF en la carpeta indicada y/o realizará la impresión del mismo por la impresora configurada.

### 5. Recuperar Operación por Cupón
>```APIKEY```: Llave Privada

Si se desea recuperar los datos de una operación teniendo su identificador de cupón, se puede hacer utilizando la opción ***-go*** del SDK, pasando como parámetro el identificador de operación en formato JSON dentro de la propiedad ```uid```
Ejemplo:
```bash
Mobbex.exe -go "{\"uid\": \"Erne0MBjK\"}"
```
Si además de mostrar los datos por consola se desea obtener el comprobante de la operación (en formado PDF o por impresora) se puede agregar opcionalmente la propiedad ```save``` con valor *true*
```bash
Mobbex.exe -go "{\"uid\": \"Erne0MBjK\", \"save\": true}"
```
En este caso el comprobante se almacenará y/o imprimirá según los parámetros de configuración definidos.
