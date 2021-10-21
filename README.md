# Autoscaler de aplicação com kubernetes

O Horizontal Pod Autoscaler (HPA) escala automaticamente o número de Pods em um replication controller, deployment, replica set ou stateful set baseado na utilização de CPU.

>**NOTA**: O HPA não é aplicável a objetos que não podem ser escalados, por exemplo, DaemonSets.

O HPA é implementado como um recurso de API do kubernetes e um controlador. O recurso determina o comportamento do controlador. O controlador periodicamente ajusta o número de replicas em um *replicattion controller* ou deployment para combinar com as métricas tais como utilização de CPU, uso de memória ou qualquer outra métrica customizada especificada pelo usuário.

# Baixando o projeto

Acesse o projeto com os seguintes comandos:

    $ git clone https://github.com/eduardoocarneiro/k8s-autoscale-hpa.git
    $ cd k8s-autoscale-hpa

# Metrics Server
O primeiro passo para que possamos fazer o autoscaling é obter as métricas. O **HorizontalPodAutoscaler** normalmente busca as métricas de uma série de APIs (metrics.k8s.io, custom.metrics.k8s.io, e external.metrics.k8s.io). A API metrics.k8s.io geralmente é fornecida pelo metrics server, que precisa ser iniciado separadamente.

Veja [Support for metrics APIs](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-metrics-apis) para mais detalhes.

## Iniciando o Metrics Server

    $ kubectl apply -f metrics-server.yaml

# Subindo a aplicação
Para utilizarmos o HPA, precisamos antes de mais nada, conhecer nossa aplicação e os recursos (CPU, memória, etc.) que ela utiliza em média. Vamos subir nosso deployment com algumas configurações de recursos que serão utilizados pelo metrics server para que o autoscaler funcione.

Para esse tutorial utilizaremos uma aplicação hello-world da Google. Para implantá-la execute:

    $ kubectl apply -f deployment.yaml

# Configurando o serviço
Vamos agora fazer a configuração do nosso serviço. Utilizaremos o nodePort para expor uma porta externamente ao nosso cluster:

    $ kubectl apply -f service.yaml

# Estressando a aplicação
Existem inúmeras ferramentas para teste de estress em aplicações web. Para esse manual utilizaremos o **fortio**, sinta-se a vontade para utilizar a que você desejar.

Abra um terminal e execute o comando abaixo:

    kubectl run --image=fortio/fortio fortio -- load -qps 2000 -t 40s -c 150 "http://myapp-service:8080"

Isso vai executar um pod que fará 2000 queries por segundo, durante 40 segundos com 150 conexões concorrentes na miha aplicação **myapp-service**.

Em um outro terminal execute o comando abaixo para acompanhar o autoscaler em funcionamento:

    watch "kubectl get hpa"

Você notará que, ao iniciar o estress na aplicação, o kubernetes automaticamente incrementará o número de Pods de acordo com os padrões configurados no arquivo **hpa.yaml**.

Para finalizar o fortio execute:

    kubectl delete pod fortio

Depois de alguns minutos você poderá notar que os Pods automaticamente serão deletados, restando assim apenas as replicas configuradas no nosso deployment.
