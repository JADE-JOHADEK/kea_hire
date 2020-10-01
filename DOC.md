# kea_hire
## Mise en route de kea_hire avec GCP

Par défaut nous ignorions la configuration de l'environnement.
Pour des raisons de lisibilités nous renseignerons les commandes essentielles et
préciserons les valeurs à utiliser. Chaque environnement doit etre personnalisé.
Ici dans ce cas pratique nous installerons l'application sur GCP qui vient avec 
un environnement et les dependences suivantes:

Dependencies 

Google Cloud SDK 

Docker

Kubectl



Si vous voulez renseignez plus de commandes et d'outils créez votre environnement personnalisé et ajoutez les dependences ci -dessus.
Docker, git et gcloud vous permet de faire les operations suivantes:

1. Utiliser le Dockerfile qui est contenu dans le repertoire du developpeur.
2. Exposer le port 3000 tel que spécifié dans les sources.
3. Copier le repertoire git sur GCP
4. Démarrer l'interface avec la fonction Loopback qui est contenu dans kea_hire

Nous allons créer un load balancer de 3 pods et nous allons placer tous les fichiers dans une nouvelle branche sur le repository de l’application avec comme nom [NOM_prenom].

Nous n'allons pas nous attarder sur les differents outils ni sur des options de configurations de sécurité.

Nom du projet : kea-hire-node

Démarrer CLOUD SHELL et utiliser gloud sur GCP


```sh


gcloud config set project kea-hire-node
```

##Pour cloner l'application de kea_hire disponible sur https://github.com/keagcp/kea_hire.git



```sh
git clone https://github.com/keagcp/kea_hire.git
```

## Entrer dans le repertoire de l'application

```sh
cd kea_hire
```

Jeter un coup d'oeil sur la version de gcloud et les fichiers dans l'aborescence du repertoire de l'appli

## Effectuer un build du projet

Executer :

```
docker build -t gcr.io/kea-hire-node/kea-hire-node:v1
```
Attention le project id que nous avons utilisé est celui de gcr dans notre commande.
Voici le contenu du Dockerfile


```
# Check out https://hub.docker.com/_/node to select a new base image
FROM node:10-slim
# Set to a non-root built-in user `node`
USER node
# Create app directory (with user `node`)
RUN mkdir -p /home/node/app

```

## Essayez de tester une image en executant le code :

Si 3000 et 3000 sont les ports que vous voulez utilisez localement et sur l'hote utilisez la commande suivantes

Si vous rencontrez une erreur effectuez un mappage de port different tout en redemarrant le container et en choisissant des ports libres ou disponibles.

```sh
docker run -d -p 3000:3000 gcr.io/kea-hire-node/kea-hire-node:v1
```
Resolvez au fur et a mesure toute erreur que vous rencontrez, par exemple pour un mappage different de port vous pouvez:

```sh
docker run -d -p 8080:3000 gcr.io/kea-hire-node/kea-hire-node:v1
```

## Push de l'image et Autres commandes 



```sh
gcloud docker -- push gcr.io/kea-hire-node/kea-hire-node:v1

gcloud config set project kea-hire-node


``` 

## Tests

```sh
curl http://localhost:8080

curl http://localhost:3000
```

## COntenu d'autres files


docker-compose.yaml

```sh
# Check out https://hub.docker.com/_/node to select a new base image
FROM node:10-slim

# Set to a non-root built-in user `node`
USER node

# Create app directory (with user `node`)
RUN mkdir -p /home/node/app

WORKDIR /home/node/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY --chown=node package*.json ./

RUN npm install

# Bundle app source code
COPY --chown=node . .

RUN npm run build

# Bind to all network interfaces so that it can be mapped to the host OS
ENV HOST=0.0.0.0 PORT=3000

EXPOSE ${PORT}
CMD [ "node", "." ]
```

kea-hire-node-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kea-hire-node
  labels:
    name: kea-hire-node
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kea-hire-node
  template:
    metadata:
      labels:
        app: kea-hire-node
    spec:
      containers:
      - name: kea-hire-node
        image: gcr.io/kea-hire-node/kea-hire-node:v3
        env:
        - name: NODE_ENV
          value: "development"
        - name: PORT
          value: "3000"
      restartPolicy: Always


```


kea-hire-node-service.yaml


```

apiVersion: v1
kind: Service
metadata:
  name: kea-hire-node
  labels:
    service: kea-hire-node
spec:
  selector:
    app: kea-hire-node
  type: LoadBalancer
  ports:
    - port: 3000


```

## Creons un cluster avec des noeuds "n1-standard-1" 
```sh

gcloud container clusters create kea-hire-node     --num-nodes=3 --zone asia-east2-b --machine-type n1-standard-1 --no-enable-ip-alias --no-enable-autoupgrade


```

##Nous allons exécuter à présent les commandes avec l'outil kubectl


Assurez vous que vous quittez le repertoire kea_hire puis que vous avez éditer les fichiers
nod-kea-hire-deployment.yaml et  node-kea-hire-service.yaml comme mentionné ci-dessus



```sh
		cd ..
	
	
	kubectl create -f ./kea_hire/kea-hire-node-deployment.yaml
kubectl create -f ./kea_hire/kea-hire-node-service.yaml

```

##Verifiez deployment et le pod et d'autres commentaires sur la configuration



```sh
kubectl get deployments

kubectl get pods

kubectl cluster-info

kubectl config view

kubectl get events

kubectl get services

kubectl get service node-kea-hire


	kubectl get pods
	
	
	kubectl get deployments
	
	
	kubectl get services
	
	
	kubectl get pods

```

Pour exposer deployment et non pod

```sh

kubectl expose deployment kea-hire-node --type="LoadBalancer"
```

Pour supprimer deployment et pod et reexecuter la commande create

```sh
		cd ..

	
		kubectl delete -f ./kea_hire/kea-hire-node-deployment.yaml
kubectl delete -f ./kea_hire/kea-hire-node-service.yaml

```

Vous pouvez verifier votre GCP avec 

```sh
	curl http://localhost:3000

```


VOus pouvez recuperer EXTERNAL-IP et acceder à 
[Kea_hire](http://EXTERNAL-IP:3000)


