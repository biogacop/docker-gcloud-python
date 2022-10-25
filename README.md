

# Nuestra aportación

La aportación es la integración continua para desplegar con CI/CD desde Github al sistema de contenedores de Google Cloud.


Las instrucciones para automatizar el despliegue son una mejora de las que provee Google en:
https://cloud.google.com/community/tutorials/cicd-cloud-run-github-actions


# Conceptos


  * En google cloud creamos proyectos que son agregadores de recursos de todo tipo. Un proyecto puede tener varios almacenes de datos, bases de datos, contenedores desplegados, imágenes docker registradas, logs, etc.. Nosotros sobre todo estaremos interesados en que diferentes alumnos pueden desplegar cada uno su aplicación de forma separada pero dentro del mismo proyecto.
  * En Github, vamos a tener repositorios públicos o privados con diferentes ramas. De una rama o de todas podemos hacer un despliegue automático con lo que se llama Integración contínua.
  * La integración continua es un proceso con diferentes etapas en las que se utilizan algunos recursos de Github y otros remotos de nuestro proveedor de nube. Para ello hay que dar a github las credenciales y definir las etapas.
  * Lo mismo que hacemos con Github puede hacerse con Gitlab o con bitbucket pero los ficheros clave se llaman de diferente manera.
  * Aunque es posible un uso gratuíto, la cantidad de recursos disponibles está acotada y tarde o temprano hay que pagar según tarifa.
  * Como los despliegues están basados en contenedores docker, el resultado en la nube debe ser el mismo que en local. Todo puede probarse y subirse cuando está maduro.


## Secuencia de configuración en Google Cloud



Partimos de la base de que se tiene instalado el cliente SDK de google cloud. Es interesante probarlo pero no es realmente imprescindible puesto que casi todo se puede hacer desde la consola gráfica.


### Fijamos usuario y proyecto de trabajo


Hemos puesto algunas variables de entorno que deben ser personalizadas en otros proyectos.


```
$ . ./env.sh
$ gcloud config set project PROJECT_ID

```

Hacemos login si lo que se quiere es trabajar desde shell

```
$ gcloud auth login

$ gcloud auth list
      Credentialed Accounts
ACTIVE  ACCOUNT
*       dockerum@gmail.com
        otheraccount@gmail.com

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

```

Como se ve, tenemos la posibilidad de manejar varias identidades y vamos a poner conmutar entre varias de ellas.


### Creación de usuario instrumental para CI



Creamos una nueva cuenta circunscrita al espacio de trabajo de Google Cloud en nuestro proyecto. La usará el repositorio. Puede haber tantas como repositorios y aplicaciones queramos tener. La creación debe hacerla el propietario del espacio Google Cloud bien desde linea de comandos o desde consola.

```
$ gcloud iam service-accounts create $ACCOUNT_NAME   --description="Cloud Run deploy account"   --display-name="Cloud-Run-Deploy"

```

Necesitamos autenticar al usuario desde Github. Con este fin creamos una credencial para el usuario dockerumu-ci. El contenido íntegro de key.json debe ser incluida en un secreto de Github. IMPORTANTE: el fichero key.json no debe incluirse como fichero en el repositorio, sino como SECRET!!!

```
$ gcloud iam service-accounts keys create key.json \
    --iam-account $ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com

```

### Permisionado del usuario para CI


En las instrucciones de google se sobre permisiona al usuario para CI haciéndolo Admin de demasiadas cosas. Este conjunto de permisos es más ajustado:

```

App Engine Deployer
Container Registry Service Agent
Cloud Build Editor
Cloud Build Service Agent
Cloud Run Developer
Service Account User
Storage Object Creator
Storage Object Viewer

```


### Habilitación de APIs


Un usuario cualquiera solo podrá hacer uso de las APIs que estén habilitadas en el proyecto. Las que necesitamos habilitar son:


```

Cloud Run API
Cloud Build API
Cloud Logging API
Cloud Pub/Sub API
App Engine Admin API


```

## Configuración en Github


### Configuración de secretos


Vamos a crear 4 secrets, entrando por Settings -> Secrets -> New repository Secret:


  - GCP_APP_NAME, nombre de la aplicación una vez desplegada. Puede haber más de una en un proyecto Gcloud. En nuestro ejemplo "alumno01" (sin comillas).
  - GCP_CREDENTIALS, contenido completo del key.json. Es el json tal cual
  - GCP_EMAIL, el usuario instrumental para CI, en nuestro caso "dockerumu-ci@practml.iam.gserviceaccount.com"  (sin comillas)
  - GCP_PROJECT_ID, el nombre del proyecto en google cloud, en nuestro caso "practml" (sin comillas)
 


### Actions


Por el simple hecho de tener el fichero GCP-Deploy.yml se desencadenará una acción automática de CI. Si hemos concedido los permisos y habilitado todo lo anterior, el único error que cabe esperar en una aplicación sencilla es que no se pueda desplegar bien.


Un fallo  en el despliegue tras varios éxitos, siempre es señal de que el contenedor contiene un software que no arranca bien. Prueba a hacer el build en tu equipo y ejecútalo para depurar los errores antes de subirlo a la nube.


Una Action deja un log. Puede ser re-ejecutada de nuevo para ver si produce un efecto diferente pero eso solo va a ser así si el problema era de permisos o de secretos y no del contenido de repositorio.


## Resultado

Tras el despliegue tenemos una URL pública del tipo  https://alumno01-cf6czs3gma-ey.a.run.app/

Si no está accesible para todos los orígenes, Cloud Run -> <aplicación> -> Triggers -> "Allow unauthenticated invocatios"



# Sobre la gratuidad de los recursos empleados


## Github


Podemos usar una cuenta gratuita para:


  - Alojar el proyecto
  - Usar 2000 minutos/mes de acciones para integración continua
  - Compartir proyectos con otros compañeros
  - Configurar las credenciales como un secreto por parte del profesor y el alumno no puede ver la credencial


Más en detalle sobre el billing de Github: https://docs.github.com/en/github/setting-up-and-managing-billing-and-payments-on-github/managing-billing-for-github-actions/about-billing-for-github-actions#:~:text=GitHub%20Actions%20usage%20is%20free,is%20controlled%20by%20spending%20limits.



## Google cloud


Hay un acceso gratuito de 300$ o 90 días para usar los servicios de google cloud. Pasado ese periodo hay que empezar a pagar. Mirando el billing sobre el saldo negativo se ve que el precio es realmente bajo. Aquí vamos a usar:


  - IAM para dar de alta usuarios para cada proyecto usados en la integración continua
  - Habilitar APIs necesarias para que GITHUB invoque acciones
  - Usar el Container Registry para alojar nuestras imágenes Docker
  - Usar el CLoud Run para desplegar la aplicación



# README.md del proyecto python base

 
## Créditos


Proyecto forkeado desde https://github.com/lvthillo/python-flask-docker.git.

Mantenemos las indicaciones originales en la parte final de este fichero. 


## python-flask-docker
Basic Python Flask app in Docker which prints the hostname and IP of the container

### Build application
Build the Docker image manually by cloning the Git repo.
```
$ git clone https://github.com/lvthillo/python-flask-docker.git
$ docker build -t lvthillo/python-flask-docker .
```

### Download precreated image
You can also just download the existing image from [DockerHub](https://hub.docker.com/r/lvthillo/python-flask-docker/).
```
docker pull lvthillo/python-flask-docker
```

### Run the container
Create a container from the image.
```
$ docker run --name my-container -d -p 8080:8080 lvthillo/python-flask-docker
```

Now visit http://localhost:8080
```
 The hostname of the container is 6095273a4e9b and its IP is 172.17.0.2. 
```

### Verify the running container
Verify by checking the container ip and hostname (ID):
```
$ docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-container
172.17.0.2
$ docker inspect -f '{{ .Config.Hostname }}' my-container
6095273a4e9b
```


