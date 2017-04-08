# aws-serverless-samfarm
Este repo contiene el código completo y las plantillas necesarias para crear una demostración de SamFarm sin servidores como se mostró en  la presentación de AWS Lambda en Re: Invent 2016.

Hay tres partes separadas de esta aplicación: la api, la tubería que detecta, construye y despliega los cambios, y el sitio web.
Comencemos por poner en funcionamiento el sitio web.

## Paso 1

### Sitio Web
En el [directorio website](website/) hay 4 archivos:

1. **[index.html](website/index.html):** Este es el archivo index que nuestro Contenedor S3 va a mostrar.
2. **[app.js](website/app.js):** El corazón de nuestro sitio web. Vamos a hacer algunos cambios aquí.
3. **[squirrel.png](website/squirrel.png):** Nuestro amigo! la ardilla SAM.
4. **[website.yaml](website/website.yaml):** La plantilla CloudFormation utilizada para crear el Contenedor de Amazon S3 para el sitio web.

Para crear el stack del sitio web.

[<img src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png">](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=myteststack&templateURL=https://awscomputeblogimages.s3-us-west-2.amazonaws.com/samfarm-website.yaml)

Una vez que el stack esté completo, tendremos que rastrear el nombre del contenedor S3 y la URL del sitio web.

Ahora tenemos todas las partes separadas de nuestro sitio web, pero vamos a ponerlo a SAM:

```bash
sh upload_website.sh <s3-bucket-name>
```

Y visita la url que guardaste antes. Deberías ver algo como esto:

![](https://tinybits.awsbryan.com/0l1Qd3oMc8.gif)

Ta-da, un sitio web funcionando! la ardilla SAM puede estar sola ahora mismo, pero vamos a arreglar eso en un momento.

## Paso 2
### API
La API Serverless que estamos construyendo! El [directorio api](api/) contiene 5 archivos. 

1. **[beta.json](api/beta.json):** El archivo de ensayo de CloudFormation. CloudFormation lo usará para pasar parámetros a nuestra plantilla CloudFormation.
2. **[buildspec.yml](api/buildspec.yml):** Esto es usado por CodeBuild en el paso de construcción de nuestro pipeline. Veremos eso más tarde.
3. **[index.js](api/index.js):** El código de la función Lambda!
4. **[package.json](api/package.json):** El package.json que define qué paquetes necesitamos para nuestra función Lambda.
5. **[saml.yaml](api/saml.yaml):** SAML mi YAML! Este es el archivo de plantilla SAM que se utiliza para crear el recurso de API Gateway y la función Lambda, y los conecta.

Crea un nuevo repo de github del [directorio api](api/) y coloca estos archivos allí. Este repo se usará para el pipeline automatizado CI / CD que construiremos a continuación.

## Paso 3

### Pipeline
El pipeline es una tubería completa sin servidor de CI / CD para construir y desplegar tu api. En este ejemplo, estamos utilizando CloudFormation para crear el pipeline, todos los recursos y los permisos necesarios.

Se crean los siguientes recursos:

- Un contenedor S3 para almacenar los artifacts del código.
- Una etapa de AWS CodeBuild para construir cualquier cambio hecho en el repo.
- El AWS CodePipeline que vigilará los cambios en tu repositorio e impulsará estos cambios hasta los pasos de creación y despliegue.
- Todos los roles IAM y politicas requeridas.

Las plantillas CloudFormation que se utilizan para crear estos recursos se pueden encontrar en el [directorio pipeline](pipeline/).

Para crear el stack del pipeline, hace clic en el botón de LaunchStack a continuación.

[<img src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png">](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=myteststack&templateURL=https://awscomputeblogimages.s3-us-west-2.amazonaws.com/samfarm-main.yaml)

## Paso 4
### Actualiza tu website
Si has mirado el código del sitio web (o si no lo has hecho, ahora es tu oportunidad!), puedes haber notado que el sitio web hace solicitudes http a una API periódicamente para actualizar el número de SAM en la pantalla. Ahora mismo está apuntando a nada, así que vamos a actualizarlo para apuntarlo a la nueva API. En el archivo [app.js](website/app.js), busca la linea

```javascript
var GET_SAM_COUNT_URL = 'INSERT API GATEWAY URL HERE';
```

y actualizalo a su endpoint de API Gateway. Debe estar en el formato:

```
https://<api-id>.execute-api.us-east-1.amazonaws.com/Prod/sam
```

Ahora vamos a actualizar el sitio web estático S3 con este cambio:

```bash
sh upload_website.sh <s3-bucket-name>
```


## Paso 5
### Iniciemos la fiesta
Ahora que tenemos nuestro sitio web, nuestro repositorio de código con nuestra función lambda y nuestro pipeline configurado, vamos a verlo en acción. Vamos a hacer dos cambios en nuestro repositorio, primero vamos a configurar nuestra API para CORS, y segundo vamos a actualizar nuestra función Lambda. Ambos cambios se realizarán en el repo que creamos en el Paso 2.


#### CORS
Abrimos al archivo beta.json y actualizamos la siguiente línea:

```json
"OriginUrl": "*"
```

a la url para el sitio estático S3 que creamos. Algo como:

```json
"OriginUrl": "http://<s3-bucket-name>.s3-website-us-east-1.amazonaws.com"
```


#### Función Lambda
Abrimos al archivo index.js en el repo que hicimos en el Paso 2 y actualizamos la línea:

```javascript
var samCount = 1;
```

a

```javascript
var samCount = 15;
```

Commit y push los cambios. 

Volvemos al pipeline que generamos en el Paso 3, veremos que AWS CodePipeline recogerá automáticamente los cambio e iniciará el proceso de creación e implementación. Voila! Una solución completamente controlada, sin servidor, CI / CD para dar a una ardilla unos cuantos amigos.


# AWSenEspañol
Te invitamos a participar de nuestro grupo en LinkedIn, [AWS en Español](https://www.linkedin.com/groups/7403992). Profesionales de habla hispana de todo el mundo nos estamos reuniendo allí para compartir y colaborar en nuestro idioma.


