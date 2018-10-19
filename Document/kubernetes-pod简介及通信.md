##### kubernetes管理的最小单元为pod，而不是容器。一个pod可以包含多个容器，也只可包含一个容器。

##### Pod这个结构体中有个变量Status，通过这个变量可以得到每个Pod的状态信息，这个Status变量对应的结构体是PodStatus，
##### Pod状态信息同四个变量相关，分别是Phase、Conditions、InitContainerStatuses和ContainerStatuses，
##### 这四个变量分别表示Pod所在生命周期阶段、Pod生命周期需要满足的条件、Pod中所有初始化容器状态、Pod中所有应用容器状态。

-   变量Phase有五个可选的值，分别是Pending、Running、Succeeded、Failed和Unknown
    -   pending：kubernetes已经开始创建Pod，但是Pod中的一个或多个容器还没有被启动。
    -   running：kubernetes已经将Pod分配到节点上，并且Pod中的所有容器都启动了。还包括Pod中至少有一个容器仍然在运行状态，或者正在重新启动状态。
    -   succeeded：Pod中的所有容器都处在终止状态，并且这些容器是自主正常退出到终止状态的，也就是退出代码为0，而且kubernetes也没有重启任何容器。
    -   failed：Pod中的所有容器都处在终止状态，并且至少有一个容器不是正常终止的，也就是退出代码不为0，或者是由于系统强行终止的。
    -   unknown：由于一些特殊情况无法获取Pod状态，比如由于网络原因无法同Pod所在的主机通讯。

-   变量Phase的取值还取决于结构体PodSpec中的RestartPolicy变量，这个RestartPolicy变量是用来设置Pod中容器重启策略的,包括三个可选值，分别是Always、OnFailure和Never
    -   Never：表示表示在对容器不执行重启策略，运行pod仅一次；
    -   Always：表示对容器一直执行重启策略。如果不设置RestartPolicy，那么Always是默认值。
    -   OnFailure：表示在容器失败的时候重启容器。
-   变量Status表示每种Type对应的状态，有三个可选的值，分别是True、False和Unknown
    -   True：表示Pod处于某种类型的有效条件中。
    -   False：表示Pod不在某种类型的有效条件中。
    -   Unknown：表示kubernetes无法判断Pod是否在某种类型的有效条件中。
    
##### pod通信大致分为两种：同一pod不同容器通信；不同pod之间通信。
-   同一pod不同容器之间通信；

    因为kubernetes将一个pod放在kubernetes集群中的某一台服务器运行，即同一pod的不同容器运行会运行在同一服务器上；
    且同一pod不同容器共享同一网络、存储等，容器之间通信可通过localhost通信，当然需要使用不同的端口。
    
    example：
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-test
      namespace: test
    spec:
      containers:
      - name: elasticsearch
        image: elasticsearch:5.6
        ports:
        - name: es-port
          containerPort: 9200
       
      - name: kibana
        image: kibana:5.6
        ports:
        - name: kibana-port
          containerPort: 5601
        env:
        - name: "ELASTICSEARCH_URL"
          value: "http://localhost:9200"
    ```
    
-   不同pod之间通信

又可分为两种：不同pod同一服务器：不同的pod，但pod在同一机器之间通信，此种通信基于docker0等方式通信。不同pod不同服务器之间：此种通信需要
借助第三方服务完成，如fannel网络服务。

具体实现方式：
    -   通过pod IP
        创建pod后，会赋予pod一个ip，其他pod通过ip访问。此种方式局限性较大：首先需要知道ip，再者随着pod挂掉重启，ip会发生改变。
    -   使用services
        将pod的ip映射为一个服务，通过访问service来访问pod。不管pod的ip如何改变，service ip不会改变。
        services详解见——基于rancher安装kubernetes使用教程
    -   映射端口服务器
        将服务port映射到该pod所在的服务器，通过访问该服务器的ip：port访问该服务。此种方式部署需要指定固定节点，如果该节点挂掉，服务挂掉。
        example: 可以通过http：//es-2-IP:8888访问
        ```
        apiVersion: v1
        kind: Pod
        metadata:
          name: podport-test
          namespace: test
        spec:
          containers:
          - name: nginx
            image: nginx:stable
            ports:
            - name: nginx-port
              containerPort: 80
              hostPort: 8888
              protocol: TCP
          nodeSelector: 
            kubernetes.io/hostname: es-2
        ```
    
        
    
    
      