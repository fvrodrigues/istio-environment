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

### Injeção automatica do sidecar

Após aplicar os recursos acima, deve-se adicionar a label de injection ao namespace desejado conforme abaixo:

````shell
export NAMESPACE=seu-namespace
kubectl label namespace $NAMESPACE istio-injection=enabled
````

### Utilizando o gateway e o ingress com wildcard

Na instalação Istio, no arquivo 4-istio-ingress.yaml um dos ingresses criados era um wildcard para o endereço: ``http://*.apps.{CLUSTER_IP}.nip.io``

Este wildcard possibilita definirmos Virtuais Services que respondam automaticamente para qualquer endereço no padrão citado acima, desta forma nosso ponto de entrada é único (istio-ingressgateway), mas nossos virtuais services podem expor qualquer URL.

Abaixo um exemplo de virtual service que responde a URL: ``http://my-site.apps.192.168.99.116.nip.io``

__Obs.:__ Assumimos que o IP do seu cluster seja ``192.168.99.116``

````yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-site-external-route
spec:
  hosts:
  - my-site.apps.192.168.99.116.nip.io #(1)
  gateways:
  - istio-system/ingressgateway #(2)
  http:
  - route:
    - destination:
        host: my-app-service #(3)

````

- (1) Endereço que o será exposto no ingressgateway
- (2) Namespace e Gateway default do Istio (opcionalmente pode-se criar um novo gateway)
- (3) Nome do service que será o destino (o Istio traduz para `my-app-service.namespace.svc.cluster.local`).
