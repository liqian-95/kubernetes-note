-   services
参考来源：https://kubernetes.io/docs/concepts/services-networking/service/
    kubernetes提供的服务模式主要有ClusterIP（默认）、NodePort、LoadBalancer、ExternalName。
    一般前一种用于内部访问，后面的用于外部访问。
    -   clusterip  
        暴露服务使用集群的ip，即kubernetes分配的ip。访问该类型的服务：clusterip：port的形式。
        example：
        ```
        kind: Service
        apiVersion: v1
        metadata:
          name: my-service
        spec:
          selector:
            app: MyApp
          ports:
          - protocol: TCP
            port: 80
            targetPort: 9376
        ```
        也可以在clusterip下提供外部服务，但此服务仅限于kubernetes集群。如集群ip：192.168.0.2,192.168.0.3
        example：
        ```
        kind: Service
        apiVersion: v1
        metadata:
          name: my-service
        spec:
          selector:
            app: MyApp
          externalIPs: 
            - 192.168.0.2   #或192.168.0.3 
          ports:
          - protocol: TCP
            port: 80
            targetPort: 9376
        ```
        通过192.168.0.2:9376来访问。
    -   NodePort
        暴露服务使用的是集群节点的ip，且每个node都开放了该port。访问方式：（任意节点ip）nodeip：NodePort
        ```
        apiVersion: v1
        kind: Service
        metadata:
          name: my-nginx
          labels:
            run: my-nginx
        spec:
          type: NodePort
          ports:
          - port: 8080         # services cluster ip使用的端口，即services-clusterip:8886也可访问该服务
            targetPort: 80     # podip使用的端口，即pod-clusterip:80也可访问该服务
            protocol: TCP
            name: http
            nodePort: 30001    #  kubernetes要求的节点端口范围30000-
          - port: 443
            protocol: TCP
            name: https
          selector:          ##表示labels包含以下内用的pod可以使用该服务名，此service对应一类pod，并非仅只一个
            run: my-nginx
        ```
    -   LoadBalancer
        使用云服务提供商提供的负载均衡暴露服务。
        example：
        ```
        kind: Service
        apiVersion: v1
        metadata:
          name: my-service
        spec:
          selector:
            app: MyApp
          ports:
          - protocol: TCP
            port: 80
            targetPort: 9376
          clusterIP: 10.0.171.239
          loadBalancerIP: 78.11.24.19
          type: LoadBalancer
        status:
          loadBalancer:
            ingress:
            - ip: 146.148.47.155
        ```
        -   ExternalName
        
