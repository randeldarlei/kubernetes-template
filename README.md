# Kubernetes Guide

## Arquitetura de Containers

**alias k=kubectl**

- **Container =** Isolamento de Recursos (Processos, dados e etc).
- **Container Engine =** Docker ou Podman, são responsável pela criação do container (pontos de montagem, volumes e etc).
- **Container Runtime =** Se comunica com o Kernel do S.O para garantir o isolamento necessário para a execução do Container (low level RunCI) (high level Containerd).
- **OCI =** Open Container Initiative.
- **Kubernetes =** Orquestrador de Container.
- **Control Plane =** Um Node que controla o Cluster.
- **Workers =** Roda todos as Máquinas que rodam as aplicações do Kubernetes.
- **POD =** Menor unidade do Kubernetes (Contém mais de um Container).
- **Comunicação entre dois PODs** (Via IP).
- **Deployment =** Controller responsável pelo deploy dentro do Kubernetes.
- **Replica Set =** Controller responsável por todas as replicas do Deployment.
- **Service NodePort =** Responsável por expor os PODS para fora do Cluster.
- **API Server =** Disponibiliza uma API que utiliza JSON sobre HTTP para comunicação, sendo o principal enabler do comando **kubectl.**
- **emptydir =** Um Volume vazio a ser adicionado ao POD

### Portas Padrão

- **etcd =** Guarda todo o estado do Cluster (Informações do kube-api-server).
- **kube scheduler =** Responsável por realiar o agendamento ****e inclusão de novos containers.
- **kube controller manager =** Garante o estado do Cluster.
- **kubelet =** Agente do Node, verificando a integridade dos containers.
- **kube proxy =** Realiza a função de proxy dentro do node.
- **Porta Padrão Kube API Server =** 6443 TCP
- **Porta Padrão etcd =** 2379 -2380 - TCP
- **Porta Padrão kube scheduler =** 10251 -TCP
- **Porta Padrão kubelet =** 10250 - TCP
- **Porta padrão  kube controller =** 10252 - TCP
- **Porta padrão Nodeport =** 30000 - 32767 - TCP
- **Porta padrão Wave net =** 6783-6784 TCP/UDP

## Comandos Kubernetes ( Container = “Isolamento de recursos”):

- **kubectl run** (Executar).
- **kubectl run —image nginx —port 80 —name jiropops** (Executar um deployment).

- **kubectl exec** (Acessar).
- **kubectl exec -it** (Executa um comando em um Pod de forma interativa).
- **kubectl exec -ti giropops — bash** (Executa o bash no Pod “giropops”).

- **kubectl get** (Descrever).
- **kubectl get pods -A** (descrever todos PODS do Cluster).
- **kubectl get pods -n kube-system** (descreve todos os PODS do kube system).
- **kubectl get pods** watch (listar pods criados).
- **kubectl get secret** (busca um secret específico dentro do Kubernetes).
- **kubectl get service -n** (descreve o serviço dentro do cluster conforme parametro).
- **kubectl get namespace** (descreve valores de um namespace conforme parametro).
- **kubectl get deployment** ******-A****** (descreve todos os deployments existentes no Cluster).
- **kubectl get services -A** (descreve todos os services existentes no Cluster).
- **kubectl get replicaset -A** (descreve todos os replicasets existentes no Cluster).

- **kubectl config current-context** (valida o cluster atual).

- **kubectl describe** (descreve elementos indicados pod por exemplo).
- **kubectl describe {param} -o yaml** (É possível salvar a saída do recurso como yaml).
- **kubectl describe {param} -o wide** (Exibe opções como node Port e IP).

- **kubectl edit** (editar um pod).

- **kubectl apply** (Aplicar alguma configuração específica).
- **kubectl apply -f** (aplica um código declarativo para a criação do recurso).

- **kubectl expose** (criar serviço).
- **kubectl expose pods** giropops (criou um serviço para ou “expos” o Pod giropops).
- **kubectl expose pods —type NodePort** giropops (criou um serviço do tipo NodePort para ou “expos” o Pod giropops).

- **kubectl delete pod** (excluir um pod imperativo).
- **kubectl delete service** (excluir um serviço).
- **kubectl delete deployment** (excluir um deployment).
- **kubectl delete namespace** (excluir um namespace).
- **kubectl delete -f** (excluir um pod declarativo).
- **kubectl delete pods** (deletando Pods).
- **kubectl delete svc** (deletar Serviços).

- **kubectl attach** (Acessa um POD específico).
- **kubectl attach (pod name) -c (container name) -i -t** (Acessa um POD específico no modo interativo que já contenha um bash disponível)

- **kubectl logs** (Busca logs)
- **kubectl logs {pod name}**
- **kubectl logs {pod name} -c {container}**
- **kubectl logs {pod name} -f**

- **— options**
- — **dry-run=client** (Este parametro executa o comando de maneira simulada ou seja ele valida as configurações mas não cria de fato os recursos).
- **-o yaml** (Este parametro exporta o Output do comando para um arquivo yaml).
- free -m (Este comando valida o uso de CPU e Memória pelo container no linux)

## Manifesto de criação de um POD simples no Kubernetes

```yaml
apiVersion: v1 --> #Versão da API que controla os PODs.
kind: Pod      --> #Recurso a ser criado, neste caso um "POD".
metadata:      --> #Informações de metadata do recurso.
	creationTimestamp: null
	labels:
		run: giropops
	name: giropops
spec:          --> #Especifícações sobre o POD.
	containers:
	- image: nginx
		name: giropops
		ports:
		- containerPort: 80
		resources: {}
	dnsPolicy: ClusterFirst
	restartPolicy: Always
status {}
```

## Gerenciamento de Memória e CPU em um POD

```yaml
apiVersion: v1 --> #Versão da API que controla os PODs.
kind: Pod      --> #Recurso a ser criado, neste caso um "POD".
metadata:      --> #Informações de metadata do recurso.
	creationTimestamp: null
	labels:
		run: giropops
	name: giropops
spec:
	containers:
	- image: ubuntu
		name: giropops
		args:
	- sleep
	- "1800"
	resources:
		limits:
			cpu: "0.5" --> #Limite máximo a ser utilizado pelo POD.
			memory: "128Mi" --> #Limite máximo de memória a ser utilizado pelo POD.
    requests:
			cpu: "0.3" --> #Kubernetes garante essa quantidade de CPU para o POD.
			memory: "64Mi" --> #Kubernetes garante essa quantidade de Memoria para o POD.
  dnsPolicy: ClusterFirst
	restartPolicy: Aways
```

## Gerenciamento de Volume simples em um POD

```yaml
apiVersion: v1 --> #Versão da API que controla os PODs.
kind: Pod      --> #Recurso a ser criado, neste caso um "POD".
metadata:      --> #Informações de metadata do recurso.
	creationTimestamp: null
	labels:
		run: giropops
	name: giropops
spec:
	containers:
	- image: nginx
		name: webserver
		volumeMounts:  --> #Local ao qual o volume é utilizado no POD.
		- mountPath: /giropops
			name: primeiro-emptydir 
		args:
	- sleep
	- "1800"
	resources:
		limits:
			cpu: "1" --> #Limite máximo a ser utilizado pelo POD.
			memory: "128Mi" --> #Limite máximo de memória a ser utilizado pelo POD.
    requests:
			cpu: "0.5" --> #Kubernetes garante essa quantidade de CPU para o POD.
			memory: "64Mi" --> #Kubernetes garante essa quantidade de Memoria para o POD.
  dnsPolicy: ClusterFirst
	restartPolicy: Always
	volumes:        --> #Local ao qual o volume é declarado no manifesto.
	- name: primeiro-emptydir
		emptyDir:
			sizeLimit: 256Mi
```


