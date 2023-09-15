# Modulo de Kubenets Curso Fullcycle 3.0

1. [Introdução](#introducao) 
2. [Serviços](#service) 
3. [Objetos de configuração](#config) 
4. [Probes](#probes) 
5. [Recursos e escala](#hpa) 
6. [Stateful e volumes persistentes](#volumes) 
7. [Ingress](#ingress) 
8. [Cert-manager](#tls) 
9. [Namespaces e Service Accounts](#ns) 

## Introdução <a id="introducao"></a>

Por alguma incompatibilidade do docker desktop tive que utilizar o k3d invés do kind

---

Para criar multiplos nodes no k3d

```
k3d cluster create nome-node-cluster --agents n_nodes
```

Aplicar configuração no k8s

```
kubectl apply -f pod.yaml
```

pegar os pod's/recursos
```
kubectl get pods
```
Aplicar redirecionamento de portas
```
kubectl port-forward pod/goserver 8080:8080
```
Hierarquia
Deployment > ReplicaSet -> Pod

Criar um proxy para acesso direto via api
```
kubectl proxy --port=8080
```

_(**deplotment.yaml**)_

------ 

## Serviços <a id="service"></a>


Services => NodePort -> ClusterIP > LoadBalancer
_(**service.yaml**)_

## Objetos de configuração <a id="config"></a>

Objetos de configuração. Variaveis de ambiente: ConfigMap _(**configmap-env.yaml**)_
Como injetar dependencia no app _(**configmap-family.yaml**)_
Secrets _(**secret.yaml**)_


## Probes <a id="probes"></a>

Monitorar o status da aplicação através de Probes

K8s possui 3 principais probes 
- _livenessProbe_
Verifica se aplicação está viva. caso falhe renicia o container
- _readinessProbe_
Verifica se a aplicação está pronta para receber trafégo
- _startupProbe_
Verifica se o container está pronto.

## Recursos e escala <a id="hpa"></a>

Sobre monitoramento dos recursos é possivel avaliar via metrics-server.
é possivel limitar tanto a cpu quanto a memória disponível para cada pod. unidade basica de cpu é o vCPU. 1vCPU equivale 1000m (milicores)

Para escalar a aplicação com base a utilização do recurso é possivel utilizar o HorizontalPodAutoscaler
_(**hpa.yaml**)_


Stressar a aplicação utilizando fortio
```
kubectl run -it fortio --rm --image=fortio/fortio -- load -qps 800 -t 120s -c 70 "http://goserver-service/healthz"
```

## Stateful e volumes persistentes <a id="volumes"></a>

Caso precise ordenamento na subida/descida dos pos e importante utilizar o StatefulSet

Para ilustrar essa aplicação foi utilizado uma imagem mysql. 
_(**statefulset.yaml**)_

Sobre a montagem de volume é possivel fazer 2 maneiras. um volume estático total _(**pv.yaml**)_
Ou de maneira dinâmica através do PersistentVolumeClaim  _(**pvc.yaml**)_

é possivel fazer uma associação dinâmica dos volumes através do volumeClaimTemplates


Para conseguir direcionar o trafégo diretamente a um pod espécifico é utilizado outro tipo de serviço chamado headless service.
_(**mysql-service-h.yaml**)_


## Ingress <a id="ingress"></a>

Ingress serve como uma especie de roteiador para redirecionar os multiplos serviços a partir de um unico endereço fazendo o loadbalancer _(**ingress.yaml**)_

Ingress permite reduzir o custo de aplicação ao utilizar apenas um ponto acesso. ex: _**API Gateway**_


## Cert-manager <a id="tls"></a>
Para realizar a certificação TLS utiliza-se o Cert-manager para gerenciar as credenciais. A emissao é feita através do let's encrypt

_(**cluster-issuer.yaml**)_

## Namespaces e Service Accounts <a id="ns"></a>

Por padrão ao utilizar o k8s. é utilizado o context default.

É recomendavel ter pelo menos 2 namespaces por projeto min: Dev / Prod

```
kubectl create namespace dev
kubectl create namespace prod
```

é possivel criar contextos apartir do comando

```
kubectl config set-context dev --namespace=dev --cluster=clustername --user=username

kubectl config set-context prod --namespace=prod --cluster=clustername --user=username
```

Para trocar de contexto basta 

```
kubectl config use-context dev
```

Service Accounts permite configurar Roles para restrigir o acesso caso exista uma vulnerabilidade na aplicação.

Para ilustrar esses conceitos foi criado um deploy simples na pasta namespaces

_(**security.yaml**)_


