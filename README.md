# Créer une cluster kubernates avec minkube

## Installation de minikube

- https://kubernetes.io/docs/tasks/tools/install-minikube/

### Windows (Professionnel / 64 bits)

- Activer la virtualisation dans le BIOS
- Hyper-V : https://docs.microsoft.com/fr-fr/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v
  Powershell (admin) `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All`
- `choco install minikube docker --confirm`
- `minikube start --vm-driver hyperv`
- Affichage du dashboard : `minikube dashboard`

## Déploiement du premier service : soap-server

Dans le réperpertoire soap-server

- Créer le fichier Dockerfile :
```Dockerfile

FROM node:11

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install

# Bundle app source
COPY . .

EXPOSE 8080

# Start app
CMD [ "npm", "start" ]
```

- Initalisation docker `@FOR /f "tokens=* delims=" %i IN ('minikube docker-env') DO %i`
- Génération de l'image docker : `docker build -t soap-server:v1 .`
- Exposition du service : 
    ```shell
    kubectl run soap-server-instance --image=soap-server:v1 --port=8080
    kubectl expose deployment soap-server-instance --type=NodePort
    kubectl get services
    minikube service soap-server-instance  --url
    ```
- Test : `http://xxx.xxx.xxx.xxx:xxxxx/wsdl?wsdl`
- Test soap avec postman

## Déploiement du deuxième service : rest-api-server

### Modifier le service "rest-api-server/nodejs-server-generated/controllers/helloworldControllerService.js"

```js
res.send({
    result: 'Hello ' + req.data.value.name,
    HOSTNAME: process.env.HOSTNAME,
});
```

### Générer l'image docker (rest-api-server:v1) & exposer le service (rest-api-server-instance)

### Test avec postman : vérifier le HOSTNAME

### Scaler le service "rest-api-server-instance" via le dashboard minikube (2 pods)

### Supprimer le pod original. Test avec postman : vérifier le HOSTNAME

### Vérifier que le pod se relance. Test avec postman : vérifier le HOSTNAME

### Arrêt /!\

```shell
kubectl delete services soap-server-instance
kubectl delete deployment soap-server-instance
minikube stop
minikube delete
```