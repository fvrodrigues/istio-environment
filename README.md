# istio-environment

Arquivos para a instalação do Istio no Minikube

## Como instalar

### Minikube

Criar um cluster com o minikube de forma padrão:

_Obs_: Caso você utilize alguma ferramenta de virtualização (ex.: virtualbox), passar via ``--vm-driver=`` como ultimo argumento do codigo abaixo.

````shell
minikube start --memory=3Gb --cpu=3
````

Habilitar os addons ingress e dashboard

````shell
minikube addons enable dashboard

minikube addons enable ingress
````

Criar um host para o dashboard
````shell
cat dashboard-ingress.yaml | sed s/{{CLUSTER_IP}}/$(minikube ip)/g dashboard-ingress.yaml | kubectl apply -f -
````

Agora você pode acessar o dashboard do seu cluster kubernetes via: ``http://dashboard.<IP DO SEU CLUSTER>.nip.io``

### Istio

Aplicar os recursos conforme o numero de cada um:

````shell
kubectl apply -f 1-istio-init.yaml

kubectl apply -f 2-istio-minikube.yaml

kubectl apply -f 3-istio-secret.yaml

cat 4-istio-ingress.yaml | sed s/{{CLUSTER_IP}}/$(minikube ip)/g 4-istio-ingress.yaml | kubectl apply -f -

````

## Como utilizar

Após aplicar os recursos acima, deve-se adicionar a label de injection ao namespace desejado conforme abaixo:

````shell
export NAMESPACE=seu-namespace
kubectl label namespace $NAMESPACE istio-injection=enabled
````
