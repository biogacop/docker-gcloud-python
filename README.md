


# Créditos

Proyecto forkeado desde https://github.com/lvthillo/python-flask-docker.git.

Mantenemos las indicaciones originales en la parte final de este fichero. 

# Aportación

La aportación es la integración continua con google cloud usando el fichero GCP-Deploy.yml para desplegar con CI/CD desde Github a Google Cloud y las variables de entorno.


Instrucciones para automatizar el despliegue en:
https://cloud.google.com/community/tutorials/cicd-cloud-run-github-actions

```
$ . ./env.sh
$ gcloud config set project PROJECT_ID
$ google auth login

$ gcloud auth list
      Credentialed Accounts
ACTIVE  ACCOUNT
*       dockerum@gmail.com
        otheraccount@gmail.com

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

```


```
$ gcloud iam service-accounts create $ACCOUNT_NAME   --description="Cloud Run deploy account"   --display-name="Cloud-Run-Deploy"
$ gcloud projects add-iam-policy-binding $PROJECT_ID   --member=serviceAccount:$ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com   --role=roles/run.admin
$ gcloud projects add-iam-policy-binding $PROJECT_ID   --member=serviceAccount:$ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com   --role=roles/storage.admin
$ gcloud projects add-iam-policy-binding $PROJECT_ID   --member=serviceAccount:$ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com   --role=roles/iam.serviceAccountUser

```

```
$ gcloud iam service-accounts keys create key.json \
    --iam-account $ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com

```



# python-flask-docker
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


