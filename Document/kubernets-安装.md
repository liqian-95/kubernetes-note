### 架构可参考 https://jimmysong.io/kubernetes-handbook/concepts/pod-overview.html

-   快速安装
    -   配置代理
        -   配置docker代理
            教程见：[docker安装及配置](https://github.com/langtodu/docker/blob/master/Document/docker%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE.md)
        -   配置系统代理
            ```
            ## 临时代理
            export HTTP_PROXY=http://proxy_ip:port/
            export HTTPS_PROXY=https://proxy_ip:port/
            ## 如果使用临时代理无法apt-get update ，设置系统代理
                ### 将以下内容添加到/etc/apt/apt.conf（若无该文件可创建）
                Acquire::http::proxy "http://proxy_username:password@proxy_ip:port";
                Acquire::https::proxy "https://proxy_username:password@proxy_ip:port";
                ### 将以下内容添加到~/bashrc
                export HTTP_PROXY=http://proxy_ip:port/
                export HTTPS_PROXY=https://proxy_ip:port/
                ### 然后执行
                source .bashrc
            ```
    -   安装kubelet kubeadm kubectl
        [参考来源](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
        ```
        apt-get update && apt-get install -y apt-transport-https
        curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
        cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
        deb http://apt.kubernetes.io/ kubernetes-xenial main
        EOF
        apt-get update
        apt-get install -y kubelet kubeadm kubectl
        ```
    -   初始化
        ```
        kubeadm init
        ```
    -   配置kubectl
        ```
        $ mkdir -p $HOME/.kube
        $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        $ sudo chown $(id -u):$(id -g) $HOME/.kube/config
        ```
    -   添加节点
        
-   [基于rancher安装](./基于rancher安装kubernetes及使用教程.md)

-   全手动安装安装
安装参考https://github.com/rootsongjc/kubernetes-handbook/blob/master/cloud-native/kubernetes-and-cloud-native-app-overview.md
### 安装
-   环境准备
    单节点安装，ip：192.168.99.107
    关闭节点swap
    ```
    使用命令关闭swap，但本地主机重启后无效
    $ sudo swapoff -a
    使用命令查看是否关闭
    $ free -m
    若swap为0，则关闭成功。
    永久关闭——编辑/etc/fstab，注释掉swap一行
    ```
-   创建TLS证书和密钥

    [详细步骤地址](https://jimmysong.io/kubernetes-handbook/practice/create-tls-and-secret-key.html)
-   部署etcd，此为但节点安装

    [详细步骤地址](https://jimmysong.io/kubernetes-handbook/practice/etcd-cluster-installation.html)
    
    注：此过程如果etcd无法启动，可能由于目录权限问题。
-   安装kubectl命令工具

    [详细步骤地址](https://jimmysong.io/kubernetes-handbook/practice/kubectl-installation.html)
-   master节点安装

    [详细步骤地址](https://jimmysong.io/kubernetes-handbook/practice/master-installation.html)
-   node节点安装，此过程未操作

    [详细步骤地址](https://jimmysong.io/kubernetes-handbook/practice/node-installation.html)
-   dashboard安装，安装失败
    
    [详细步骤地址](https://jimmysong.io/kubernetes-handbook/practice/dashboard-addon-installation.html)
    
