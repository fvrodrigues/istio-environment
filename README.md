# istio-environment
Configuração ambiente istio

Após rodar kubectl apply -f 'arquivo' , deve adicionar label ao namespace desejado conforme abaixo : 

kubectl label namespace default istio-injection=enabled

