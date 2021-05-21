


# Créditos

Proyecto forkeado desde https://github.com/lvthillo/python-flask-docker.git.

Mantenemos las indicaciones originales en la parte final de este fichero. 

# Nuestra aportación

La aportación es la integración continua con google cloud para desplegar con CI/CD desde Github al sistema de contenedores de Google Cloud.


Las instrucciones para automatizar el despliegue son una mejora de las que provee Google en:
https://cloud.google.com/community/tutorials/cicd-cloud-run-github-actions


## Secuencia de configuración desde linea de comandos

Partimos de la base de que se tiene instalado el cliente SDK de google cloud. Es interesante probarlo pero no es realmente imprescindible puesto que casi todo se puede hacer desde la consola gráfica.


### Fijamos usuario y proyecto de trabajo


Hemos puesto algunas variables de entorno que deben ser personalizadas en otros proyectos.


```
$ . ./env.sh
$ gcloud config set project PROJECT_ID

```

Hacemos login si lo que se quiere es trabajar desde shell

```
$ google auth login

$ gcloud auth list
      Credentialed Accounts
ACTIVE  ACCOUNT
*       dockerum@gmail.com
        otheraccount@gmail.com

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

```

Como se ve, tenemos la posibilidad de manejar varias identidades y vamos a poner conmutar entre varias de ellas.


### Creación de usuario instrumental para CI


Creamos una nueva cuenta circunscrita al espacio de trabajo de Google Cloud en nuestro proyecto. La usará el repositorio. Puede haber tantas como repositorios y aplicaciones queramos tener. La creación debe hacerla el propietario del espacio Google Cloud bien desde linea de comandos o desde consola.

```
$ gcloud iam service-accounts create $ACCOUNT_NAME   --description="Cloud Run deploy account"   --display-name="Cloud-Run-Deploy"

```

Necesitamos autenticar al usuario desde Github. Con este fin creamos una credencial para el usuario dockerumu-ci. El contenido íntegro de key.json debe ser incluida en un secreto de Github. IMPORTANTE: el fichero key.json no debe incluirse como fichero en el repositorio, sino como SECRET!!!

```
$ gcloud iam service-accounts keys create key.json \
    --iam-account $ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com

```

### Permisionado del usuario para CI


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


# Sobre la gratuidad de los recursos empleados


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


