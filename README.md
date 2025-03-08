Instalação das ferramentas
Instale o kind

curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.5.1/kind-$(uname)-amd64 && \
chmod +x ./kind && \
sudo mv ./kind /usr/local/bin/kind
Instale o auto-completar do kind:

echo 'source <(kind completion bash)' >>~/.bashrc &&\
source ~/.bashrc
Instale o kubectl:

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl &&\
chmod +x ./kubectl && \
sudo mv ./kubectl /usr/local/bin/kubectl && \
kubectl version
Instale o auto-completar do kubectl:

echo 'source <(kubectl completion bash)' >>~/.bashrc &&\
source ~/.bashrc
Instale o Docker:

sudo curl -fsSL https://get.docker.com | sh && \
sudo usermod -aG docker $USER && \
sudo newgrp docker && \
sudo systemctl enable docker && \
sudo systemctl start docker && \
sudo systemctl status docker && \
sudo docker --version
Hello World Kind
Crie o cluster:

kind create cluster --name kind
Exporte o arquivo kubeconfig:

export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
Informações do cluster:

kubectl cluster-info

Conecte no su cluster: 

kubectl cluster-info --context kind-kind

Visualize seus clusters:

kind get clusters
Execute a aplicação:

kubectl run demo --image=franciscojsc/myhello --port=9999 --labels app=demo
Redirecione a porta do host para o porta do pod com o port-forward:

kubectl port-forward deploy/demo 9999:8888
Visualize o status:

kubectl get pods --selector app=demo
Delete o cluster:

kind delete cluster --name kind
Kind com multi-nós
Crie o arquivo config.yaml:

kind: Cluster
apiVersion: kind.sigs.k8s.io/v1alpha3
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
Crie o cluster utilizando o arquivo config.yaml:

kind create cluster --config config.yaml 
Exporte o arquivo kubeconfig:

export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
Informações do cluster:

kubectl cluster-info
Visualize os nós do cluster:

kubectl get nodes
Para destruir o cluster:

kind delete cluster --name kind
Kind com WORDPRESS e MYSQL
Será utilizados os tutoriais disponíveis em https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume e https://itnext.io/starting-local-kubernetes-using-kind-and-docker-c6089acfc1c0 como referências, para executar o Wordpress com Mysql em um cluster kubernetes com a ferramenta Kind.

De acordo com o tutorial do site https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume, o Worpress e Mysql utilizam:

Both applications use PersistentVolumes and PersistentVolumeClaims to store data.

A PersistentVolume (PV) is a piece of storage in the cluster that has been manually provisioned by an administrator, or dynamically provisioned by Kubernetes using a StorageClass. A PersistentVolumeClaim (PVC) is a request for storage by a user that can be fulfilled by a PV. PersistentVolumes and PersistentVolumeClaims are independent from Pod lifecycles and preserve data through restarting, rescheduling, and even deleting Pods.

Crie o cluster:

kind create cluster --name kind
Exporte o arquivo kubeconfig:

export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
Crie um diretório para os arquivos que serão utilizados:

mkdir kind-wp && \
cd kind-wp
Crie o arquivo com o nome kustomization.yaml:

cat <<EOF >./kustomization.yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=YOUR_PASSWORD
resources:
    - mysql-deployment.yaml
    - wordpress-deployment.yaml
EOF
Realize o download do arquivo mysql-deployment.yaml:

curl -LO https://k8s.io/examples/application/wordpress/mysql-deployment.yaml
Realize o download do arquivo wordpress-deployment.yaml:

curl -LO https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml
Aplique as configurações dos arquivos ao cluster:

cd kind-wp
kubectl apply -k ./
Verifique o status do cluster:

kubectl get secrets
kubectl get pvc
kubectl get pods
kubectl get services wordpress
Redirecione a porta, para permitir acesso ao service:

kubectl port-forward svc/wordpress 8080:80
Acesse a url http://localhost:8080 e visualize a aplicação em execução.

Para destruir o cluster:

kind delete cluster --name kind
Kind com PHP
Será utilizado o tutorial disponível em https://king.host/blog/2018/05/como-usar-kubernetes-na-pratica como referência, para executar um container PHP e exibir informações do php em uma página web, utilizando a ferramenta Kind.

Crie o cluster:

kind create cluster --name kind
exporte o arquivo kubeconfig:

export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
Crie o arquivo para o objeto pod com o nome aplicacao.yaml:

apiVersion: v1   #versão da API do Kubernetes 
kind: Pod        #Tipo do objeto que será criado
metadata: 
 name: aplicacao #Nome de identificação para o objeto pod
spec:            #Características do objeto
 containers:
   - name: container-aplicacao
     image: php:5.6-apache
     ports:
       - containerPort: 80
Crie o objeto pod:

kubectl create -f aplicacao.yaml
Redirecione a porta 8080 do host para a porta 80 do container:

kubectl port-forward pod/aplicacao 8080:80
Apague o pod:

kubectl delete pods aplicacao
Visualize os pods:

kubectl get pods
Crie o arquivo para o objeto deployment com o nome deployment.yaml:

apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: aplicacao-deployment
spec:
  template:
    metadata:
      labels:
        name: aplicacao-pod  
    spec:
      containers:
        - name: container-aplicacao  
          image: php:5.6-apache  
          ports:  
            - containerPort: 80
Crie o objeto deployment:

kubectl create -f deployment.yaml 
Apague o pod:

kubectl delete pods aplicacao
Visualize os pods:

kubectl get pods
Crie o arquivo para o objeto service com o nome servico-aplicacao.yaml:

apiVersion:  v1
kind:  Service
metadata:
  name:  servico-aplicacao
spec:
  type: LoadBalancer
  ports:
    - port:  80
  selector:
    name:  aplicacao-pod
Crie o objeto service:

kubectl create -f servico-aplicacao.yaml 
Visualize os Services:

kubectl get services
Redirecione a porta 8080 do host para a porta 80 do container:

kubectl port-forward deploy/aplicacao-deployment  8080:80
Recupere a identificação do pod:

kubectl get pods
Entre no container do pod, subtituindo id-of-pod-here pelo id do pod:

kubectl exec -it id-of-pod-here bash
Adicione o conteúdo a seguir dentro do container:

echo "<?php phpinfo(); ?>" > index.php
Acesse a url http:localhost:8080 e visualize as informações do PHP.

Para destruir o cluster:

kind delete cluster --name kind


Para a realização do teste, execute o comando sem este parâmetro, ficando da seguinte maneira:
kubectl run -it fortio --rm --image=fortio/fortio -- load -qps 800 -t 120s -c 70 "http://goserver-service/healthz"