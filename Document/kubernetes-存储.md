kubernetes支持多种存储方案，大致分为两种：临时存储，永久存储。临时存储如emptyDir，随着pod删除立即被删除；永久存储又可分为内部存储和外
部存储，内部存储指的是在集群中某一文件夹下存储，如hostPath，若该集群节点挂掉，存储也将无法使用；外部存储指的是使用第三方存储，存储地方
不在本地，如NFS。

详细参考：https://github.com/kubernetes/examples/tree/master/staging/volumes

##### 以下介绍常用存储和存储备份

###### 常用存储

-   创建并使用存储，即volume
参考来源：http://docs.kubernetes.org.cn/429.html
    -   emptyDir
    使用emptyDir，临时存储文件。当Pod分配到Node上时，将会创建emptyDir，并且只要Node上的Pod一直运行，Volume就会一直存，直至pod被删除。
    删除容器即emptyDir被删除。
        example：
        ```
        apiVersion: v1        
        kind: Pod
        metadata:
          name: test-pd
        spec:
          containers:
          - image: gcr.io/google_containers/test-webserver
            name: test-container
            volumeMounts:
            - mountPath: /cache
              name: cache-volume
          volumes:
          - name: cache-volume
            emptyDir: {}
        ```
    -   hostPath
    允许挂载Node上的文件系统到Pod里面去。如果Pod需要使用Node上的文件，可以使用hostPath。
        ```
        apiVersion: v1
        kind: Pod
        metadata:
          name: test-pd
        spec:
          containers:
          - image: gcr.io/google_containers/test-webserver
            name: test-container
            volumeMounts:
            - mountPath: /test-pd
              name: test-volume
          volumes:
          - name: test-volume
            hostPath:
              path: /data
        ```
    -   NFS
        NFS 是Network File System的缩写，即网络文件系统。Kubernetes中通过简单地配置就可以挂载NFS到Pod中，而NFS中的数据是可以永久保
        存的，同时NFS支持同时写操作。Pod被删除时，Volume被卸载，内容被保留。
        参考来源：https://github.com/kubernetes/examples/tree/master/staging/volumes/nfs
        -   创建NFS
            example:在192.168.0.56的主机上的/home/ubuntu/nfs创建1024M大小的nfs
            ```
            apiVersion: v1
            kind: PersistentVolume
            metadata:
              name: nfs
            spec:
              capacity:
                storage: 1024Mi
              accessModes:
                - ReadWriteMany                   # ReadWriteOnce – 被单个节点mount为读写rw模式
              nfs:                                # ReadOnlyMany – 被多个节点mount为只读ro模式
                server: 192.168.0.56              # ReadWriteMany – 被多个节点mount为读写rw模式
                path: "/home/ubuntu/nfs"
            ```
            注： 一个卷同一时刻可以有多种访问方式，但写方式只能有一个。
            参考：https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
        
        -   使用NFS
            参考来源：https://github.com/kubernetes/examples/tree/master/staging/volumes/nfs
            https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
            example1：
            ```
            apiVersion: v1
            kind: Pod
            metadata:
              name: elastic
              namespace: test
            spec:
              containers:
              - image: elasticsearch:5.6
                name: es-test
                ports:
                - name: es-port
                  containerPort: 9200
                volumeMounts:
                - mountPath: /usr/share/elasticsearch/data
                  name: data
              volumes:
              - name: data
                nfs:
                  server: 192.168.0.127     ### nfs服务器ip
                  path: /home/storage       ### nfs服务器其中的一个共享目录
            ```
            example2：  创建PersistentVolume-申请PersistentVolumeClaim-使用该PersistentVolumeClaim
            创建一块nfs
            ```
            kind: PersistentVolume
            apiVersion: v1
            metadata:
              name: task-pv-volume
            spec:
              storageClassName: manual
              capacity:
                storage: 10Gi
              accessModes:
                - ReadWriteMany
              nfs:
                server: 192.168.0.127
                path: "/home/storage"
            ```
            申请nfs：
            ```
            kind: PersistentVolumeClaim
            apiVersion: v1
            metadata:
              name: task-pv-claim
            spec:
              storageClassName: manual
              accessModes:
                - ReadWriteMany
              resources:
                requests:
                  storage: 3Gi
            ```
            使用该nfs：
            ```
            apiVersion: v1
            kind: Pod
            metadata:
              name: elastic
              namespace: test
            spec:
              containers:
              - image: elasticsearch:5.6
                name: es-test
                ports:
                  - name: es-port
                    containerPort: 9200
                volumeMounts:
                  - mountPath: /usr/share/elasticsearch/data
                    name: data
              volumes:
                - name: data
                  persistentVolumeClaim:
                    claimName: task-pv-claim
            ```
            
    -   secret
    参考来源：https://kubernetes.io/docs/concepts/configuration/secret/
    secret volume用于将敏感信息（如密码）传递给pod。可以将secrets存储在Kubernetes API中，使用的时候以文件的形式挂载到pod中，而不用连接api。
        -   创建secret
            example:
            ```
            # Create files needed for rest of example.
            $ echo -n "admin" > ./username.txt
            $ echo -n "1f2d1e2e67df" > ./password.txt
            $ kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
            # 查看secrets
            $ kubectl get secrets
            $ kubectl describe secrets/db-user-pass
            
            # 或使用yaml, 将用户名admin、密码1f2d1e2e67df
            echo -n "admin" | base64
            echo -n "1f2d1e2e67df" | base64
            apiVersion: v1
            kind: Secret
            metadata:
              name: mysecret
            type: Opaque
            data:
              username: YWRtaW4=
              password: MWYyZDFlMmU2N2Rm
            ```
        -   使用secret
            example：
            ```
            apiVersion: v1
            kind: Pod
            metadata:
              name: mypod
            spec:
              containers:
              - name: mypod
                image: redis
                volumeMounts:
                - name: foo
                  mountPath: "/etc/foo"
                  readOnly: true
              volumes:
              - name: foo
                secret:
                  secretName: mysecret
            ```
    注：权限设置参考来源

###### 存储备份


