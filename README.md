#Datos para el despliegue de PandoraFMS en Google Cloud

1. Cuenta de Gmail: ngeek.nvc@gmail.com *Pass: Qazplm123.

2. Ingresar a la cuenta y configurar una credit card

3. ingrese a GCP con la cuanta de gmail 

4. Crea un proyecto nuevo y activa el cloud shell 
- Project name: Ngeek-NVC
- Project number: 904729239195
- Proejct ID: ngeek-nvc

5. Estando en el shell siga los siguientes pasos

* Preparar el entorno para el funcionamiento de la aplicacion

- Setear y verificar el acceso al proyecto
$ gcloud config set project ngeek-nvc
$ gcloud auth list
$ gcloud config list project
$ gcloud compute project-info describe

- Clonar el codigo de muestra
$ git clone https://github.com/rafaelameijeiras/PandoraFMS

- Establesca la region y la zona predeterminada
$ gcloud config set compute/region us-east1
$ gcloud config set compute/zone us-east1-b 

- Crear un cluster y autenticarlo
$ gcloud container clusters create cluster-pandorafms
$ gcloud container clusters get-credentials cluster-pandorafms
 
- Ingresa al directorio correspondiente
$ cd PandoraFMS/pandorafms_community/

- Instala el binario para la conversion de manifisto de compose a manifiestos de kubernetes 
$ wget https://github.com/kubernetes/kompose/releases/download/v1.26.1/kompose_1.26.1_amd64.deb # Replace 1.26.1 with latest tag
$ sudo apt install ./kompose_1.26.1_amd64.deb

* Preparar los archivos de la aplicacion

- Edita el manifiesto de compose para que el conversor funcione 
$ nano docker-compose.yml 
# Pega esto
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

version: '2'

services:

  db:
    image: rameijeiras/pandorafms-percona-base
    restart: always
    command: ["mysqld", "--innodb-buffer-pool-size=300M"] 
    environment:
      MYSQL_ROOT_PASSWORD: pandora
      MYSQL_DATABASE: pandora
      MYSQL_USER: pandora
      MYSQL_PASSWORD: pandora
    networks:
     - pandora
    expose:
     - "3306"    
  pandora:
    image: rameijeiras/pandorafms-community
    restart: always
    depends_on:
      - db
    environment:
      MYSQL_ROOT_PASSWORD: pandora
      DBHOST: db
      DBNAME: pandora
      DBUSER: pandora
      DBPASS: pandora
      DBPORT: 3306
      INSTANCE_NAME: pandora01
      SLEEP: 10
      RETRIES: 5
    volumes:
      - mysql:/var/lib/mysql
    networks:
     - pandora
    ports:
      - "8080:80"
      - "41121:41121"
      - "162:162/udp"

networks:
  pandora:
volumes:
  mysql:

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
- ctrl + x 
- y + intro

- compila el kompose 
$ kompose convert -f docker-compose.yml

- recompila los manifiestos .yaml para hacer el despliegue
$ kubectl apply -f mysql-persistentvolumeclaim.yaml
$ kubectl apply -f db-deployment.yaml
$ kubectl apply -f db-service.yaml
$ kubectl apply -f pandora-deployment.yaml
$ kubectl apply -f pandora-service.yaml
$ kubectl apply -f pandorafmscommunity-pandora-networkpolicy.yaml

- verifica que todo este funcionando 
$ kubectl get all
$ kubectl get pvc
$ kubectl get nodes

- forward al servicio de pandora 
$ kubectl port-forward svc/pandora 8080:8080

- acceder al puesto 8080 del shell y añadir /pandora_console/ a la URL

///////////////////////////////////////////////////////////////////////////////////////////
RECURSOS DE PODS Y NODES
-> fg //para regresar a un proceso pausado con cntrl+z
-> minikube start --extra-config=kubelet.housekeeping-interval=10s // para que el top sirva con los pods
-> kubectl top pods //para ver el CPU y la Memoria consumida de cada pod
-> kubectl top nodes //para ver el CPU y la Memoria consumida de cada node



- Si se requiere crear el proyecto en un namespace especifico entonces: (v3)
$ kubectl apply -f manifiestos/
$ kubectl apply --namespace pandora -f manifiestos/k8s/


