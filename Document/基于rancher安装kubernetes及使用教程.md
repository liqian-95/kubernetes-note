-   安装高可用的rancher
    详细参考：
    http://rancher.com/docs/rancher/v1.6/en/installing-rancher/installing-server/#multi-nodes
    http://www.dockerinfo.net/260.html
    -   本教程使用openstack 所用到的数据库
        ```
        CREATE DATABASE IF NOT EXISTS cattle COLLATE = 'utf8_general_ci' CHARACTER SET = 'utf8';
        GRANT ALL ON cattle.* TO 'cattle'@'%' IDENTIFIED BY 'cattle';
        GRANT ALL ON cattle.* TO 'cattle'@'localhost' IDENTIFIED BY 'cattle';
        ```
    其中：
        ```
        --db-host               IP or hostname of MySQL server
        --db-port               port of MySQL server (default: 3306)
        --db-user               username for MySQL login (default: cattle)
        --db-pass               password for MySQL login (default: cattle)
        --db-name               MySQL database name to use (default: cattle)
        --db-strict-enforcing   Ensures table has primary key (default: false), available as of v1.6.11
        ```
    -   运行rancher容器
        ```
        $ docker run -d --restart=unless-stopped -p 8080:8080 -p 9345:9345 rancher/server \
             --db-host myhost.example.com --db-port 3306 --db-user username --db-pass password --db-name cattle \
             --advertise-address <IP_of_the_Node>
        ```
    其中advertise-address可以为ip或网络接口名，eg:eth0。
    -   负载平衡,使用haproxy或nginx
        -   在openstack controller node 安装haproxy ，将以下内容修改、复制到/etc/haproxy/haproxy.cfg, 然后重启haproxy。
            参考来源：http://rancher.com/docs/rancher/v1.6/en/installing-rancher/installing-server/basic-ssl-config/#example-haproxy-configuration
            ```
            global
              maxconn 4096
              ssl-server-verify none
            
            defaults
              mode http
              balance roundrobin
              option redispatch
              option forwardfor
            
              timeout connect 5s
              timeout queue 5s
              timeout client 36000s
              timeout server 36000s
            
            frontend http-in
              mode http
              #bind *:443 ssl crt /etc/haproxy/certificate.pem
              #mode tcp
              bind *:8080
              default_backend rancher_servers
            
              # Add headers for SSL offloading
              #http-request set-header X-Forwarded-Proto https if { ssl_fc }
              #http-request set-header X-Forwarded-Ssl on if { ssl_fc }
            
              acl is_websocket hdr(Upgrade) -i WebSocket
              acl is_websocket hdr_beg(Host) -i ws
              use_backend rancher_servers if is_websocket
            
            backend rancher_servers
              server websrv1 <rancher_server_1_IP>:8080 weight 1 maxconn 1024
              server websrv2 <rancher_server_2_IP>:8080 weight 1 maxconn 1024
              server websrv3 <rancher_server_3_IP>:8080 weight 1 maxconn 1024
            ```
        -   在docker集群部署nginx
        参考来源：http://rancher.com/docs/rancher/v1.6/en/installing-rancher/installing-server/basic-ssl-config/#example-nginx-configuration
            ```
            upstream rancher {
                server rancher-server:8080;
            }
            
            map $http_upgrade $connection_upgrade {
                default Upgrade;
                ''      close;
            }
            
            server {
                listen 80;
                server_name <server>;
                
                location / {
                    proxy_set_header Host $host;
                    proxy_set_header X-Forwarded-Proto $scheme;
                    proxy_set_header X-Forwarded-Port $server_port;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_pass http://rancher;
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection $connection_upgrade;
                    # This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this parameter, the default is 1 minute and will automatically close.
                    proxy_read_timeout 900s;
                }
            }
            
            server {
                listen 80;
                server_name <server>;
                return 301 https://$server_name$request_uri;
            }
            ```
-   使用rancher安装kubernetes
    由于kubernetes默认设置的原因，致使部署kubernetes无法启动kubernetes-dashboard，所以需要进行修改，修改kubernetes默认镜像来源。
    参考来源：
    https://www.cnrancher.com/rancher-k8s-accelerate-installation-document/
    https://www.cnrancher.com/kubernetes-installation/
    https://www.kubernetes.org.cn/2955.html
    -   前提准备
        -   关闭主机swap
            ```
            使用命令关闭swap，但本地主机重启后无效
            $ sudo swapoff -a
            使用命令查看是否关闭
            $ free -m
            若swap为0，则关闭成功。
            永久关闭——编辑/etc/fstab，注释掉swap一行
            ```
    -   版本对应，即docker版本与rancher版本，本次实验rancher：stable
        docker-17.06.03-ce，rancher/ks：v1.7.7-rancher1
        
    -   进入rancher UI界面，——系统管理——系统设置，此步骤可省略，若省略部署的是k8s v1.8
        ```
        在添加应用商店一览添加如下信息
                名称：library
                地址：https://git.oschina.net/rancher/rancher-catalog.git
                分类：k8s-cn，据说已不再维护，可使用k8s-v1.6-release
                注：k8s-cn部署的K8S是v1.5版本，k8s-v1.6-release部署的K8S是K8S-v1.7版本
                来源：http://blog.csdn.net/csdn_duomaomao/article/details/78068178
                然后保存。
        ```
    -   查看kubernetes环境模板是否发生改变
        ```
        若无，可进行如下修改：
            Private Registry for Add-Ons and Pod Infra Container Image一览填写:
            registry.cn-shenzhen.aliyuncs.com
            Pod Infra Container Image一览填写:
            rancher_cn/pause-amd64:3.0
            image namespace for Add-Ons and Pod一栏填写：
            rancher_cn
            image namespace for kubernetes-helm image一栏填写：
            rancher_cn
        ```    
    -   重新添加kubetnetes环境，添加主机
    -   等待一段时间，dashboard可使用。
注：将rancher中的kubernetes cli证书添加到装有kubctl的主机.ssh/文件中，可使用kubctl创建services。
-   创建namespace，即project
    -   使用yaml创建
        example： my-namespace.yaml
        ```
            ---
            apiVersion: v1
            kind: Namespace
            metadata:
              name: new-namespace
            ---                  # 以下用于限制资源，可无
            apiVersion: v1
            kind: ResourceQuota
            metadata:
              name: resourcelimit
              namespace: new-namespace
            spec:
              hard:
                cpu: "20"
                memory: 1Gi
                pods: "10"
                replicationcontrollers: "20"
                resourcequotas: "1"
                services: "5"
        ```
    命令：
        ```
        创建namespace
        $ kubectl create -f ./my-namespace.yaml
        查看该namespace
        $ kubectl describe ns new-namespace
        ```
    -   使用kubctl创建
        example：
        ```
        $ kubectl create namespace test
        ```
    注：可以使用yaml对在namespace中创建的pod、container资源进行限制
    如：对namespace——test，
        example：limits.yaml
        ```
        apiVersion: v1
        kind: LimitRange
        metadata:
          name: mylimits
          namespace: test
        spec:
          limits:
          - max:
              cpu: "2"
              memory: 1Gi
            min:
              cpu: 200m
              memory: 6Mi
            type: Pod
        
          - default:
              cpu: 300m
              memory: 200Mi
            defaultRequest:
              cpu: 200m
              memory: 100Mi
            max:
              cpu: "2"
              memory: 1Gi
            min:
              cpu: 100m
              memory: 3Mi
            type: Container
        ```


    