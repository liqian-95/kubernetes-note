-   config
参考来源：https://kubernetes-v1-4.github.io/docs/user-guide/configmap/
https://k8smeetup.github.io/docs/tasks/configure-pod-container/configmap/
用于创建配置文件，可挂载到pod或container中。
    -   
        ```
        kind: ConfigMap
        apiVersion: v1
        metadata:
          creationTimestamp: 2016-02-18T19:14:38Z
          name: example-config
          namespace: default
        data:
          example.property.1: hello
          example.property.2: world
          example.property.file: |-
            property.1=value-1
            property.2=value-2
            property.3=value-3
        ```
    -   
        ```
        apiVersion: v1
        data:
          game.properties: |-
            enemies=aliens
            lives=3
            enemies.cheat=true
            enemies.cheat.level=noGoodRotten
            secret.code.passphrase=UUDDLRLRBABAS
            secret.code.allowed=true
            secret.code.lives=30
          ui.properties: |
            color.good=purple
            color.bad=yellow
            allow.textmode=true
            how.nice.to.look=fairlyNice
        kind: ConfigMap
        metadata:
          creationTimestamp: 2016-02-18T18:34:05Z
          name: game-config
          namespace: default
          resourceVersion: "407"-
          selfLink: /api/v1/namespaces/default/configmaps/game-config
          uid: 30944725-d66e-11e5-8cd0-68f728db1985
        ```
        创建并查看：
        ```
        $ kubectl create configmap game-config-3 --from-file=game-special-key=docs/user-guide/configmap/kubectl/game.properties
        $ kubectl get configmaps game-config-3 -o yaml
        ```
    -   环境变量使用config
        -   配置样例
            example:
            ```
            apiVersion: v1
            data:
              special.how: very
              special.type: charm
            kind: ConfigMap
            metadata:
              creationTimestamp: 2016-02-18T19:14:38Z
              name: special-config
              namespace: default
              resourceVersion: "651"
              selfLink: /api/v1/namespaces/default/configmaps/special-config
              uid: dadce046-d673-11e5-8cd0-68f728db1985
            ```
        -   使用实例
            example:
            ```
            apiVersion: v1
            kind: Pod
            metadata:
              name: dapi-test-pod
            spec:
              containers:
                - name: test-container
                  image: gcr.io/google_containers/busybox
                  command: [ "/bin/sh", "-c", "env" ]
                  env:
                    - name: SPECIAL_LEVEL_KEY
                      valueFrom:
                        configMapKeyRef:
                          name: special-config
                          key: special.how
                    - name: SPECIAL_TYPE_KEY
                      valueFrom:
                        configMapKeyRef:
                          name: special-config
                          key: special.type
              restartPolicy: Never
            ```
    -   设置command使用config
        -   配置样例
            example:
            ```
            apiVersion: v1
            kind: ConfigMap
            metadata:
              name: special-config
              namespace: default
            data:
              special.how: very
              special.type: charm
            ```
        -   使用实例
            example:
            ```
            apiVersion: v1
            kind: Pod
            metadata:
              name: dapi-test-pod
            spec:
              containers:
                - name: test-container
                  image: gcr.io/google_containers/busybox
                  command: [ "/bin/sh", "-c", "echo $(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)" ]
                  env:
                    - name: SPECIAL_LEVEL_KEY
                      valueFrom:
                        configMapKeyRef:
                          name: special-config
                          key: special.how
                    - name: SPECIAL_TYPE_KEY
                      valueFrom:
                        configMapKeyRef:
                          name: special-config
                          key: special.type
              restartPolicy: Never
            ```  
    -   volume插件使用config
        -   配置样例
            example:
            ```
            apiVersion: v1
            kind: ConfigMap
            metadata:
              name: special-config
              namespace: default
            data:
              special.how: very
              special.type: charm
            ```
        -   使用实例
            example:
            ```
            apiVersion: v1
            kind: Pod
            metadata:
              name: dapi-test-pod
            spec:
              containers:
                - name: test-container
                  image: gcr.io/google_containers/busybox
                  command: [ "/bin/sh", "cat /etc/config/special.how" ]
                  volumeMounts:
                  - name: config-volume
                    mountPath: /etc/config
              volumes:
                - name: config-volume
                  configMap:
                    name: special-config
              restartPolicy: Never
            ```
            或使用
            ```
            apiVersion: v1
            kind: Pod
            metadata:
              name: dapi-test-pod
            spec:
              containers:
                - name: test-container
                  image: gcr.io/google_containers/busybox
                  command: [ "/bin/sh", "cat /etc/config/path/to/special-key" ]
                  volumeMounts:
                  - name: config-volume
                    mountPath: /etc/config
              volumes:
                - name: config-volume
                  configMap:
                    name: special-config
                    items:
                    - key: special.how
                      path: path/to/special-key
              restartPolicy: Never
            ```
    -   实例，配置某一文件
    config example：
    ```
    apiVersion: v1
    data:
      redis-config: |
        maxmemory 2mb
        maxmemory-policy allkeys-lru
    kind: ConfigMap
    metadata:
      creationTimestamp: 2016-03-30T18:14:41Z
      name: example-redis-config
      namespace: default
      resourceVersion: "24686"
      selfLink: /api/v1/namespaces/default/configmaps/example-redis-config
      uid: 460a2b6e-f6a3-11e5-8ae5-42010af00002
    ```
    使用：
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: redis
    spec:
      containers:
      - name: redis
        image: kubernetes/redis:v1
        env:
        - name: MASTER
          value: "true"
        ports:
        - containerPort: 6379
        resources:
          limits:
            cpu: "0.1"
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: config
      volumes:
        - name: data
          emptyDir: {}
        - name: config
          configMap:
            name: example-redis-config
            items:
            - key: redis-config
              path: redis.conf
    ```
    