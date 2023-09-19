# serveless-todo-app

Para implementar este proyecto, debe implementar una aplicación TODO simple utilizando AWS Lambda y el marco Serverless. Busque todos los comentarios que comiencen con `TODO:` en el código para encontrar los marcadores de posición que necesita implementar.

# Functionality of the application

Esta aplicación permitirá crear/eliminar/actualizar/obtener elementos TODO. Cada elemento TODO puede tener opcionalmente una imagen adjunta. Cada usuario sólo tiene acceso a los elementos TODO que ha creado.

# TODO items
La aplicación debe almacenar elementos TODO y cada elemento TODO contiene los siguientes campos:

* `todoId` (string) - a unique id for an item
* `createdAt` (string) - date and time when an item was created
* `name` (string) - name of a TODO item (e.g. "Change a light bulb")
* `dueDate` (string) - date and time by which an item should be completed
* `done` (boolean) - true if an item was completed, false otherwise
* `attachmentUrl` (string) (optional) - a URL pointing to an image attached to a TODO item

También puedes almacenar una identificación de un usuario que creó un elemento TODO.

## Prerequisites

* <a href="https://manage.auth0.com/" target="_blank">Auth0 account</a>
* <a href="https://github.com" target="_blank">GitHub account</a>
* <a href="https://nodejs.org/en/download/package-manager/" target="_blank">NodeJS</a> version up to 12.xx 
* Serverless 
   * Create a <a href="https://dashboard.serverless.com/" target="_blank">Serverless account</a> user
   * Install the Serverless Framework’s CLI  (up to VERSION=2.21.1). Refer to the <a href="https://www.serverless.com/framework/docs/getting-started/" target="_blank">official documentation</a> for more help.
   ```bash
   npm install -g serverless@2.21.1
   serverless --version
   ```
   * Login and configure serverless to use the AWS credentials 
   ```bash
  # Inicie sesión en su panel desde la CLI. Le pedirá que abra su navegador y finalice el proceso.
    inicio de sesión sin servidor
    # Configurar sin servidor para usar las credenciales de AWS para implementar la aplicación
    # Debe tener un par de claves de acceso (YOUR_ACCESS_KEY_ID y YOUR_SECRET_KEY) de un usuario de IAM con permisos de acceso de administrador
    credenciales de configuración de sls --provider aws --key YOUR_ACCESS_KEY_ID --secret YOUR_SECRET_KEY --profile serverless
   ```
   
# Functions to be implemented

Para implementar este proyecto, necesita implementar las siguientes funciones y configurarlas en el archivo `serverless.yml`:

* `Auth`: esta función debe implementar un autorizador personalizado para API Gateway que debe agregarse a todas las demás funciones.

* `GetTodos`: debería devolver todos los TODO para un usuario actual. Se puede extraer una identificación de usuario de un token JWT que envía la interfaz

Debería devolver datos similares a estos:

```json
{
  "items": [
    {
      "todoId": "123",
      "createdAt": "2019-07-27T20:01:45.424Z",
      "name": "Buy milk",
      "dueDate": "2019-07-29T20:01:45.424Z",
      "done": false,
      "attachmentUrl": "http://example.com/image.png"
    },
    {
      "todoId": "456",
      "createdAt": "2019-07-27T20:01:45.424Z",
      "name": "Send a letter",
      "dueDate": "2019-07-29T20:01:45.424Z",
      "done": true,
      "attachmentUrl": "http://example.com/image.png"
    },
  ]
}
```

* `CreateTodo`: debería crear un nuevo TODO para un usuario actual. Se puede encontrar una forma de datos enviada por una aplicación cliente a esta función en el archivo `CreateTodoRequest.ts`

It receives a new TODO item to be created in JSON format that looks like this:

```json
{
  "createdAt": "2019-07-27T20:01:45.424Z",
  "name": "Buy milk",
  "dueDate": "2019-07-29T20:01:45.424Z",
  "done": false,
  "attachmentUrl": "http://example.com/image.png"
}
```

Debería devolver un nuevo elemento TODO similar a este:

```json
{
  "item": {
    "todoId": "123",
    "createdAt": "2019-07-27T20:01:45.424Z",
    "name": "Buy milk",
    "dueDate": "2019-07-29T20:01:45.424Z",
    "done": false,
    "attachmentUrl": "http://example.com/image.png"
  }
}
```

* `UpdateTodo`: debe actualizar un elemento TODO creado por un usuario actual. Se puede encontrar una forma de datos enviada por una aplicación cliente a esta función en el archivo `UpdateTodoRequest.ts`

Recibe un objeto que contiene tres campos que se pueden actualizar en un elemento TODO:

```json
{
  "name": "Buy bread",
  "dueDate": "2019-07-29T20:01:45.424Z",
  "done": true
}
```

La identificación de un elemento que debe actualizarse se pasa como parámetro de URL.

Debería devolver un cuerpo vacío.

* `DeleteTodo`: debe eliminar un elemento TODO creado por un usuario actual. Espera una identificación de un elemento TODO para eliminar.

Debería devolver un cuerpo vacío.

* `GenerateUploadUrl`: devuelve una URL prefirmada que se puede utilizar para cargar un archivo adjunto para un elemento TODO.

Debería devolver un objeto JSON similar a este:

```json
{
  "uploadUrl": "https://s3-bucket-name.s3.eu-west-2.amazonaws.com/image.png"
}
```

Todas las funciones ya están conectadas a los eventos apropiados de API Gateway.

Se puede extraer una identificación de un usuario de un token JWT pasado por un cliente.

También debe agregar los recursos necesarios a la sección `recursos` del archivo `serverless.yml`, como la tabla DynamoDB y el depósito S3.


# Frontend

La carpeta `cliente` contiene una aplicación web que puede utilizar la API que debe desarrollarse en el proyecto.

Esta interfaz debería funcionar con su aplicación sin servidor una vez desarrollada, no necesita realizar ningún cambio en el código. El único archivo que necesita editar es el archivo `config.ts` en la carpeta `client`. Este archivo configura su aplicación cliente tal como se hizo en el curso y contiene un punto final API y una configuración Auth0:

```ts
const apiId = '...' API Gateway id
export const apiEndpoint = `https://${apiId}.execute-api.us-east-1.amazonaws.com/dev`

export const authConfig = {
  domain: '...',    // Domain from Auth0
  clientId: '...',  // Client id from an Auth0 application
  callbackUrl: 'http://localhost:3000/callback'
}
```

## Authentication

Para implementar la autenticación en su aplicación, deberá crear una aplicación Auth0 y copiar "dominio" e "id de cliente" al archivo `config.ts` en la carpeta `client`. Recomendamos utilizar tokens JWT cifrados asimétricamente.

# Best practices

Para completar este ejercicio, siga las mejores prácticas de la sexta lección de este curso.

## Logging

El código inicial viene con un registrador [Winston](https://github.com/winstonjs/winston) configurado que crea [formato JSON](https://stackify.com/what-is-structured-logging-and-why -los desarrolladores lo necesitan/) declaraciones de registro. Puedes usarlo para escribir mensajes de registro como este:

```ts
import { createLogger } from '../../utils/logger'
const logger = createLogger('auth')

// You can provide additional information with every log statement
// This information can then be used to search for log statements in a log storage system
logger.info('User was authorized', {
  // Additional information stored with a log statement
  key: 'value'
})
```


# Calificar el envío

Una vez que haya terminado de desarrollar su aplicación, configure los parámetros `apiId` y Auth0 en el archivo `config.ts` en la carpeta `client`. Un revisor iniciaría el servidor de desarrollo React para ejecutar la interfaz que debe configurarse para interactuar con su aplicación sin servidor.

**IMPORTANT**

*Deje su solicitud en ejecución hasta que se revise el envío. Si se implementa correctamente, no costará casi nada cuando su aplicación esté inactiva.*

# Suggestions

Para almacenar elementos TODO, es posible que desee utilizar una tabla de DynamoDB con índices secundarios locales. Para crear un índice secundario local, necesita crear un recurso de DynamoDB como este:

```yml

TodosTable:
  Type: AWS::DynamoDB::Table
  Properties:
    AttributeDefinitions:
      - AttributeName: partitionKey
        AttributeType: S
      - AttributeName: sortKey
        AttributeType: S
      - AttributeName: indexKey
        AttributeType: S
    KeySchema:
      - AttributeName: partitionKey
        KeyType: HASH
      - AttributeName: sortKey
        KeyType: RANGE
    BillingMode: PAY_PER_REQUEST
    TableName: ${self:provider.environment.TODOS_TABLE}
    LocalSecondaryIndexes:
      - IndexName: ${self:provider.environment.INDEX_NAME}
        KeySchema:
          - AttributeName: partitionKey
            KeyType: HASH
          - AttributeName: indexKey
            KeyType: RANGE
        Projection:
          ProjectionType: ALL # What attributes will be copied to an index

```

Para consultar un índice necesita usar el método `query()` como:

```ts
await this.dynamoDBClient
  .query({
    TableName: 'table-name',
    IndexName: 'index-name',
    KeyConditionExpression: 'paritionKey = :paritionKey',
    ExpressionAttributeValues: {
      ':paritionKey': partitionKeyValue
    }
  })
  .promise()
```

# Cómo ejecutar la aplicación

## Backend

Para implementar una aplicación, ejecute los siguientes comandos:

```
cd backend
npm install
sls deploy -v
```

## Frontend

To run a client application first edit the `client/src/config.ts` file to set correct parameters. And then run the following commands:

```
cd client
npm install
npm run start
```

Esto debería iniciar un servidor de desarrollo con la aplicación React que interactuará con la aplicación TODO sin servidor.

# Colección cartero

Una forma alternativa de probar su API es utilizar la colección Postman que contiene solicitudes de muestra. Puedes encontrar una colección Postman en este proyecto. Para importar esta colección, haga lo siguiente.
Click on the import button:

![Alt text](images/import-collection-1.png?raw=true "Image 1")


Haga clic en "Elegir archivos":

![Alt text](images/import-collection-2.png?raw=true "Image 2")


Seleccione un archivo para importar:

![Alt text](images/import-collection-3.png?raw=true "Image 3")

Right click on the imported collection to set variables for the collection:

![Alt text](images/import-collection-4.png?raw=true "Image 4")

Provide variables for the collection (similarly to how this was done in the course):

![Alt text](images/import-collection-5.png?raw=true "Image 5")

Right click on the imported collection to set variables for the collection:

![Alt text](images/import-collection-4.png?raw=true "Image 4")

Provide variables for the collection (similarly to how this was done in the course):

![Alt text](images/import-collection-5.png?raw=true "Image 5")
