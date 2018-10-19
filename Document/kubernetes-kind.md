#### kubernetes支持多种kind，kind表示对象，代表系统的一个永久资源。每种kind有各自独特的功能，根据各自的特点，
#### 在实际使用中选择合适的kind。

-   Pod
指单个pod。
    -   Job
        example:
        ```
        apiVersion: batch/v1
          kind: Job
          metadata:
            # Unique key of the Job instance
            name: example-job
          spec:
            template:
              metadata:
                name: example-job
              spec:
                containers:
                - name: pi
                  image: perl
                  command: ["perl"]
                  args: ["-Mbignum=bpi", "-wle", "print bpi(2000)"]
                # Do not restart containers after they exit
                restartPolicy: Never
          ```
          -   Pod
              example:
              ```
              apiVersion: v1
              kind: Pod
              metadata:
                name: pod-example
              spec:
                containers:
                - image: ubuntu:trusty
                  command: ["echo"]
                  args: ["Hello World"]
              ```
                
-   Controller
    Controller可以创建和管理多个Pod，提供副本管理、滚动升级和集群级别的自愈能力。例如，如果一个Node故障，Controller就能自动将该节点上
    的Pod调度到其他健康的Node上。包含一个或者多个Pod的Controller: Deployment、StatefulSet、DaemonSet等
    -   Deployment
        参考来源：http://docs.kubernetes.org.cn/317.html
        Deployment为Pod和Replica Set（升级版的 Replication Controller）提供声明式更新
        example:
            ```
            apiVersion: apps/v1beta1
            kind: Deployment
            metadata:
              name: nginx-deployment
            spec:
              replicas: 3
              template:
                metadata:
                  labels:
                    app: nginx
                spec:
                  containers:
                  - name: nginx
                    image: nginx:1.7.9
                    ports:
                    - containerPort: 80
            ```
-   ReplicationController
     参考来源：http://docs.kubernetes.org.cn/437.html
     在用户定义范围内，如果pod增多，则ReplicationController会终止额外的pod，如果减少，RC会创建新的pod，始终保持在定义范围。
     例如，RC会在Pod维护（例如内核升级）后在节点上重新创建新Pod。
        example：
            ```
            apiVersion: v1
            kind: ReplicationController
            metadata:
              name: nginx
            spec:
              replicas: 3
              selector:
                app: nginx
              template:
                metadata:
                  name: nginx
                  labels:
                    app: nginx
                spec:
                  containers:
                  - name: nginx
                    image: nginx
                    ports:
                    - containerPort: 80
            ```
        查看nginx
            ```
            $ kubectl describe replicationcontrollers/nginx
            ```
-   StatefulSet
    参考来源：http://docs.kubernetes.org.cn/443.html
    有状态系统服务设计，
       example:
       ```
       apiVersion: apps/v1beta1
       kind: StatefulSet
       metadata:
         name: web
       spec:
         serviceName: "nginx"
         replicas: 3
         template:
           metadata:
           labels:
             app: nginx
           spec:
             terminationGracePeriodSeconds: 10
             containers:
             - name: nginx
               image: gcr.io/google_containers/nginx-slim:0.8
               ports:
               - containerPort: 80
                 name: web
       ```
-   DaemonSet
    DaemonSet能够让所有（或者一些特定）的Node节点运行同一个pod.
    example:
    ```
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluentd-elasticsearch
      namespace: kube-system
      labels:
        k8s-app: fluentd-logging
    spec:
      selector:
        matchLabels:
          name: fluentd-elasticsearch
        template:
          metadata:
            labels:
              name: fluentd-elasticsearch
            spec:
              tolerations:
              - key: node-role.kubernetes.io/master
                effect: NoSchedule
              containers:
              - name: fluentd-elasticsearch
                image: gcr.io/google-containers/fluentd-elasticsearch:1.20
                resources:
                limits:
                  memory: 200Mi
                  requests:
                    cpu: 100m
                    memory: 200Mi
                 volumeMounts:
                 - name: varlog
                   mountPath: /var/log
                 - name: varlibdockercontainers
                   mountPath: /var/lib/docker/containers
                   readOnly: true
                 terminationGracePeriodSeconds: 30
                volumes:
                - name: varlog
                  hostPath:
                    path: /var/log
                - name: varlibdockercontainers
                  hostPath:
                    path: /var/lib/docker/containers
            ```
