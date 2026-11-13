# Aula 5 – Computação em Nuvem | Kubernetes

## Sobre a aula

Nesta aula, demos continuidade ao conteúdo de Computação em Nuvem. Na aula anterior, trabalhamos com o Docker, aprendendo a executar uma aplicação dentro de um contêiner. Dessa vez, avançamos um pouco mais e conhecemos o Kubernetes, uma ferramenta utilizada para gerenciar aplicações que trabalham com contêineres.

Durante a aula, aprendemos como criar uma aplicação, verificar se ela está funcionando, aumentar a quantidade de cópias e entender o que acontece quando uma delas é excluída.

Toda a atividade prática foi realizada pelo Killercoda, utilizando o Kubernetes Playground, diretamente pelo navegador.

## O que foi trabalhado na aula?

- Conhecendo o Kubernetes e sua função.
- Criando e gerenciando Pods.
- Utilizando Deployments para controlar aplicações.
- Aumentando a quantidade de réplicas.
- Entendendo como funciona a recuperação automática.
- Criando Services para permitir o acesso à aplicação.
- Testando a comunicação com o Nginx.
- Removendo os recursos utilizados.

---

# Atividade prática – Kubernetes

## Passo 1 – Acessando o Killercoda

Primeiramente, acessei o site Killercoda e fiz login na minha conta gratuita. Depois, entrei no Kubernetes Playground de dois nós e aguardei até que o terminal e o ambiente estivessem disponíveis para começar a atividade.

O ambiente utilizado é temporário, então foi necessário realizar a prática durante a aula.

**O que aprendi:** O Killercoda permite praticar Kubernetes diretamente pelo navegador, sem precisar instalar e configurar todo o ambiente no computador.

## Passo 2 – Verificando o ambiente

Antes de começar a criar a aplicação, precisei verificar se o Kubernetes estava funcionando corretamente.

Primeiro, consultei a versão instalada:

**Comando:**

```bash
kubectl version --client
```

**Resultado:**

```text
Client Version: v1.37.1
Kustomize Version: v5.8.1
```

Depois, verifiquei os nós disponíveis:

**Comando:**

```bash
kubectl get nodes
```

**Resultado:**

```text
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   4d6h   v1.37.1
node01         Ready    <none>          4d5h   v1.37.1
```

**O que aprendi:** Apareceram dois nós, chamados `controlplane` e `node01`, ambos com o status `Ready`. Isso mostrou que o ambiente estava preparado para receber nossa aplicação.

## Passo 3 – Criando a aplicação

Nessa etapa, criamos um Deployment chamado `web-aula5`, utilizando o Nginx como aplicação.

**Comando:**

```bash
kubectl create deployment web-aula5 --image=nginx:alpine
```

**Resultado:**

```text
deployment.apps/web-aula5 created
```

Depois, verificamos se a aplicação havia sido criada:

**Comando:**

```bash
kubectl get deployments
```

**Resultado:**

```text
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
web-aula5   1/1     1            1           109s
```

**O que aprendi:** Conseguimos criar nossa primeira aplicação utilizando o Kubernetes. O Deployment ficou responsável por controlar a aplicação e manter a quantidade de réplicas que configuramos.

## Passo 4 – Verificando os Pods

Depois de criar a aplicação, verificamos se o Kubernetes havia criado o Pod corretamente.

**Primeiro comando:**

```bash
kubectl get pods
```

**Resultado:**

```text
NAME                         READY   STATUS    RESTARTS   AGE
web-aula5-779c6f9566-bqttt   1/1     Running   0          4m24s
```

Em seguida, utilizamos outro comando para visualizar mais informações:

**Segundo comando:**

```bash
kubectl get pods -o wide
```

**Resultado:**

```text
NAME                         READY   STATUS    RESTARTS   AGE     IP              NODE     NOMINATED NODE   READINESS GATES
web-aula5-779c6f9566-bqttt   1/1     Running   0          5m35s   192.168.1.232   node01   <none>           <none>
```

**O que aprendi:** O Pod apareceu com o status `Running`, mostrando que estava funcionando. Também conseguimos visualizar seu endereço IP e identificar que estava sendo executado no nó `node01`.

Entendi que o Pod é o ambiente onde os contêineres da aplicação são executados.

## Passo 5 – Consultando os detalhes do Pod

Nessa etapa, utilizamos o comando `describe` para consultar informações mais completas sobre o Pod.

Na primeira tentativa, ocorreu um erro porque digitei o número zero no lugar da letra O.

**Primeira tentativa:**

```bash
kubectl describe pod $(kubectl get pods -0 jsonpath='{.items[0].metadata.name}')
```

**Resultado:**

```text
error: unknown shorthand flag: '0' in -0
See 'kubectl get --help' for usage.
```

Depois de identificar o erro, corrigimos o comando:

**Comando corrigido:**

```bash
kubectl describe pod $(kubectl get pods -o jsonpath='{.items[0].metadata.name}')
```

**Resultado da consulta:**

```text
Name:             web-aula5-779c6f9566-bqttt
Namespace:        default
Node:             node01/172.30.2.2
Labels:           app=web-aula5
Status:           Running
IP:               192.168.1.232
Controlled By:    ReplicaSet/web-aula5-779c6f9566
Containers:
  nginx:
    Image:        nginx:alpine
    State:        Running
    Ready:        True
    Restart Count: 0
Events:
  Normal  Scheduled  Successfully assigned default/web-aula5-779c6f9566-bqttt to node01
  Normal  Pulling    Pulling image "nginx:alpine"
  Normal  Pulled     Successfully pulled image "nginx:alpine"
  Normal  Created    Container created
  Normal  Started    Container started
```

**O que aprendi:** Conseguimos visualizar informações importantes, como o nome da imagem utilizada, o endereço IP, o nó onde o Pod estava funcionando e os eventos que aconteceram durante sua criação.

Também percebi que um pequeno erro de digitação pode impedir a execução de um comando, mas que é possível identificar e corrigir esses problemas.

## Passo 6 – Aumentando a quantidade de réplicas

Depois de verificar que a aplicação estava funcionando, fizemos um teste aumentando a quantidade de réplicas de uma para três.

**Comando:**

```bash
kubectl scale deployment web-aula5 --replicas=3
```

**Resultado:**

```text
deployment.apps/web-aula5 scaled
```

Em seguida, verificamos os Pods novamente:

**Comando:**

```bash
kubectl get pods -o wide
```

**Resultado:**

```text
NAME                         READY   STATUS    RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
web-aula5-779c6f9566-9mw7p   1/1     Running   0          53s   192.168.1.131   node01   <none>           <none>
web-aula5-779c6f9566-bqttt   1/1     Running   0          12m   192.168.1.232   node01   <none>           <none>
web-aula5-779c6f9566-pd274   1/1     Running   0          53s   192.168.1.8     node01   <none>           <none>
```

**O que aprendi:** Conseguimos aumentar a quantidade de cópias da aplicação sem precisar criar cada Pod manualmente. O Kubernetes criou os novos Pods e manteve os três em execução.

Essa parte mostrou como podemos aumentar a capacidade de uma aplicação de acordo com a necessidade.

## Passo 7 – Verificando Deployment, ReplicaSet e Pods

Nessa etapa, utilizamos um comando para visualizar três recursos do Kubernetes ao mesmo tempo.

**Comando:**

```bash
kubectl get deployment,replicaset,pods
```

**Resultado:**

```text
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-aula5   3/3     3            3           13m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/web-aula5-779c6f9566   3         3         3       13m

NAME                             READY   STATUS    RESTARTS   AGE
pod/web-aula5-779c6f9566-9mw7p   1/1     Running   0          2m2s
pod/web-aula5-779c6f9566-bqttt   1/1     Running   0          13m
pod/web-aula5-779c6f9566-pd274   1/1     Running   0          2m2s
```

**O que aprendi:** Nessa consulta, conseguimos entender melhor como os recursos trabalham juntos:

- Deployment: controla a aplicação e define quantas réplicas queremos.
- ReplicaSet: ajuda a manter a quantidade de Pods configurada.
- Pods: executam os contêineres da aplicação.

Foi interessante visualizar como essas partes trabalham juntas para manter a aplicação funcionando.

## Passo 8 – Excluindo um Pod e verificando sua recriação

Nessa etapa, fizemos um teste para observar o que aconteceria se um dos Pods fosse excluído.

Primeiro, verificamos os Pods existentes:

**Comando:**

```bash
kubectl get pods
```

**Resultado:**

```text
NAME                         READY   STATUS    RESTARTS   AGE
web-aula5-779c6f9566-9mw7p   1/1     Running   0          3m49s
web-aula5-779c6f9566-bqttt   1/1     Running   0          15m
web-aula5-779c6f9566-pd274   1/1     Running   0          3m49s
```

Depois, excluímos um dos Pods:

**Comando:**

```bash
kubectl delete pod $(kubectl get pods -o jsonpath='{.items[0].metadata.name}')
```

**Resultado:**

```text
pod "web-aula5-779c6f9566-9mw7p" deleted from default namespace
```

Em seguida, consultamos novamente:

**Comando:**

```bash
kubectl get pods
```

**Resultado:**

```text
NAME                         READY   STATUS    RESTARTS   AGE
web-aula5-779c6f9566-bqttt   1/1     Running   0          18m
web-aula5-779c6f9566-pd274   1/1     Running   0          7m
web-aula5-779c6f9566-pl2n7   1/1     Running   0          39s
```

**O que aprendi:** Mesmo depois de excluir um Pod, o Kubernetes percebeu que faltava uma réplica e criou outro automaticamente.

Essa foi uma das partes mais interessantes da atividade, pois conseguimos observar na prática como funciona a recuperação automática. Ou seja, não precisamos criar manualmente outro Pod para substituir o que foi removido.

## Passo 9 – Criando um Service

Depois de trabalhar com os Pods, criamos um Service chamado `web-service` para permitir o acesso à aplicação dentro do cluster.

**Comando:**

```bash
kubectl expose deployment web-aula5 --port=80 --target-port=80 --name=web-service
```

**Resultado:**

```text
service/web-service exposed
```

Depois, verificamos se o serviço havia sido criado:

**Comando:**

```bash
kubectl get service web-service
```

**Resultado:**

```text
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
web-service   ClusterIP   10.97.50.123   <none>        80/TCP    56s
```

**O que aprendi:** Conseguimos criar um ponto de acesso para nossa aplicação. Entendi que o Service facilita a comunicação com os Pods, sem depender diretamente do endereço IP individual de cada um.

## Passo 10 – Testando a comunicação com o Service

Na última parte da atividade, fizemos um teste para verificar se era possível acessar a aplicação por meio do Service.

Para isso, utilizamos o Curl, que permite fazer requisições e verificar as respostas de uma aplicação.

Na primeira tentativa, houve um erro porque foi utilizado apenas um hífen em `-restart=Never`.

**Primeira tentativa:**

```bash
ubectl run teste --image=curlimages/curl --rm -it -restart=Never -- \ curl http://web-service
```

**Resultado:**

```text
error: unknown shorthand flag: 'r' in -restart=Never
See 'kubectl run --help' for usage.
```

Na segunda tentativa, houve outro erro de digitação: foi escrito `rum` em vez de `run`.

**Segunda tentativa:**

```bash
kubectl rum teste --image=curlimages/curl --rm -it --restart=Never -- \ curl http://web-service
```

**Resultado:**

```text
error: unknown command "rum" for "kubectl"

Did you mean this?
        run
```

Depois, fizemos uma terceira tentativa, corrigindo a escrita:

**Terceira tentativa:**

```bash
kubectl run teste --image=curlimages/curl --rm -it --restart=Never -- \ curl http://web-service
```

**Resultado apresentado:**

```text
curl: (3) URL rejected: Bad hostname

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.</p>
</body>
</html>
```

**O que aprendi:** O teste apresentou a página padrão do Nginx, embora também tenha aparecido uma mensagem de erro do Curl. Com isso, conseguimos observar o retorno da aplicação e entender a finalidade do Service na comunicação dentro do cluster.

Também percebi a importância de escrever os comandos corretamente e conferir as mensagens apresentadas pelo terminal.

---

# Desafio prático – Aula 5

Depois de concluir a atividade principal, realizamos o desafio proposto pela professora. A proposta era aumentar a aplicação para cinco réplicas, excluir um Pod e verificar se o Kubernetes conseguiria manter novamente a quantidade configurada.

## Passo 1 – Aumentando para cinco réplicas

Primeiramente, utilizamos o comando abaixo para aumentar a quantidade de réplicas da aplicação `web-aula5` para cinco.

**Comando:**

```bash
kubectl scale deployment web-aula5 --replicas=5
```

**Resultado:**

```text
deployment.apps/web-aula5 scaled
```

Em seguida, verificamos se os cinco Pods estavam funcionando:

**Comando:**

```bash
kubectl get pods
```

**Resultado:**

```text
NAME                         READY   STATUS    RESTARTS   AGE
web-aula5-779c6f9566-q86vk   1/1     Running   0          13m
web-aula5-779c6f9566-rssm4   1/1     Running   0          13m
web-aula5-779c6f9566-rtjtb   1/1     Running   0          2m59s
web-aula5-779c6f9566-s7r5l   1/1     Running   0          9m37s
web-aula5-779c6f9566-w5c8s   1/1     Running   0          2m59s
```

Como podemos observar, os cinco Pods estavam com o status `Running`, mostrando que estavam funcionando corretamente.

Depois, realizamos um teste para verificar se o Kubernetes conseguiria substituir automaticamente um Pod excluído.

**Comando:**

```bash
kubectl delete pod $(kubectl get pods -o jsonpath='{.items[0].metadata.name}')
```

**Resultado:**

```text
pod "web-aula5-779c6f9566-q86vk" deleted from default namespace
```

Após a exclusão, verificamos novamente os Pods:

**Comando:**

```bash
kubectl get pods
```

**Resultado:**

```text
NAME                         READY   STATUS    RESTARTS   AGE
web-aula5-779c6f9566-fvkp6   1/1     Running   0          61s
web-aula5-779c6f9566-rssm4   1/1     Running   0          16m
web-aula5-779c6f9566-rtjtb   1/1     Running   0          5m55s
web-aula5-779c6f9566-s7r5l   1/1     Running   0          12m
web-aula5-779c6f9566-w5c8s   1/1     Running   0          5m55s
```

**O que aprendi:** Mesmo após a exclusão de um dos Pods, o Kubernetes criou outro automaticamente e manteve as cinco réplicas. Assim, conseguimos comprovar na prática o funcionamento da escalabilidade e da recuperação automática.

## Passo 2 – Removendo os recursos utilizados

Depois de concluir o teste, começamos a remover os recursos utilizados na atividade.

Primeiramente, tentamos excluir o serviço `web-service`.

**Comando:**

```bash
kubectl delete service web-service
```

**Resultado:**

```text
Error from server (NotFound): services "web-service" not found
```

Nessa tentativa, o Kubernetes informou que não encontrou um serviço com esse nome no namespace padrão.

Em seguida, excluímos o Deployment:

**Comando:**

```bash
kubectl delete deployment web-aula5
```

**Resultado:**

```text
deployment.apps "web-aula5" deleted from default namespace
```

Depois, verificamos se os Pods tinham sido removidos:

**Comando:**

```bash
kubectl get pods
```

**Resultado:**

```text
No resources found in default namespace.
```

Por último, verificamos os serviços que continuavam disponíveis:

**Comando:**

```bash
kubectl get services
```

**Resultado:**

```text
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7d12h
```

**O que aprendi:** Conseguimos remover o Deployment e confirmar que os Pods haviam sido excluídos. Na última consulta, apareceu somente o serviço padrão do Kubernetes.

Também entendi que, além de criar e utilizar os recursos, é importante saber removê-los quando não são mais necessários.

---

# O que aprendi na Aula 5?

Essa aula foi importante para entender melhor como funciona o gerenciamento de aplicações com Kubernetes.

Antes, tínhamos trabalhado com o Docker para executar contêineres. Agora, conseguimos perceber como o Kubernetes ajuda a organizar e controlar essas aplicações.

Os principais aprendizados foram:

- **Kubernetes:** entendi que é utilizado para gerenciar aplicações que funcionam por meio de contêineres.
- **Pod:** aprendi que é onde os contêineres são executados.
- **Deployment:** entendi que controla a aplicação e mantém a quantidade de réplicas desejada.
- **ReplicaSet:** aprendi que ajuda a manter a quantidade de Pods configurada.
- **Escalabilidade:** consegui aumentar a aplicação de uma para três e depois para cinco réplicas.
- **Recuperação automática:** observei que, quando um Pod é excluído, o Kubernetes cria outro para manter a quantidade configurada.
- **Service:** entendi que facilita o acesso e a comunicação com os Pods.

## Conclusão

A Aula 5 foi uma continuação importante do que aprendemos anteriormente sobre Docker. Dessa vez, conseguimos conhecer o Kubernetes e entender como ele facilita o gerenciamento de aplicações em ambientes de computação em nuvem.

O que mais chamou minha atenção foi o teste de excluir um Pod e perceber que o próprio Kubernetes criou outro automaticamente. Isso ajudou a entender melhor como ele trabalha para manter a aplicação funcionando conforme a quantidade configurada.

Também tivemos alguns erros de digitação durante a prática, mas conseguimos identificar os problemas e entender melhor os comandos utilizados.

No geral, foi uma aula em que conseguimos colocar a teoria em prática e compreender melhor como as aplicações podem ser organizadas, ampliadas e mantidas em funcionamento na nuvem.

